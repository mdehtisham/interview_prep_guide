## Design Patterns

### 90. What are design patterns in JavaScript?

**Answer:**

Design patterns are reusable, proven solutions to common software design problems. They provide templates for writing code that is maintainable, scalable, and easier to understand. In JavaScript, design patterns help structure applications, manage complexity, and promote best practices. They're categorized into three main types: **Creational** (object creation), **Structural** (object composition), and **Behavioral** (object interaction and communication).

#### **Pattern Categories Overview**

```javascript
// CREATIONAL PATTERNS - Object creation mechanisms
// - Constructor Pattern
// - Factory Pattern
// - Singleton Pattern
// - Prototype Pattern
// - Builder Pattern

// STRUCTURAL PATTERNS - Object composition
// - Module Pattern
// - Revealing Module Pattern
// - Decorator Pattern
// - Facade Pattern
// - Proxy Pattern
// - Adapter Pattern
// - Composite Pattern

// BEHAVIORAL PATTERNS - Object communication
// - Observer Pattern (Pub/Sub)
// - Strategy Pattern
// - Command Pattern
// - Iterator Pattern
// - Mediator Pattern
// - Chain of Responsibility
// - State Pattern
// - Template Method Pattern
```

---

## **CREATIONAL PATTERNS**

### **1. Module Pattern**

The Module Pattern encapsulates private and public members, creating a clean API while hiding implementation details. It uses closures to maintain private state.

```javascript
// Basic Module Pattern
const ShoppingCart = (function() {
  // Private variables and functions
  let items = [];
  let total = 0;
  
  function calculateTotal() {
    total = items.reduce((sum, item) => sum + item.price * item.quantity, 0);
    return total;
  }
  
  function validateItem(item) {
    return item && item.id && item.price > 0 && item.quantity > 0;
  }
  
  // Public API
  return {
    addItem(item) {
      if (!validateItem(item)) {
        throw new Error('Invalid item');
      }
      
      const existingItem = items.find(i => i.id === item.id);
      
      if (existingItem) {
        existingItem.quantity += item.quantity;
      } else {
        items.push({ ...item });
      }
      
      calculateTotal();
      console.log(`Added ${item.name} to cart`);
    },
    
    removeItem(itemId) {
      const index = items.findIndex(i => i.id === itemId);
      
      if (index !== -1) {
        const removed = items.splice(index, 1)[0];
        calculateTotal();
        console.log(`Removed ${removed.name} from cart`);
        return true;
      }
      
      return false;
    },
    
    updateQuantity(itemId, quantity) {
      const item = items.find(i => i.id === itemId);
      
      if (item && quantity > 0) {
        item.quantity = quantity;
        calculateTotal();
        return true;
      }
      
      return false;
    },
    
    getItems() {
      // Return copy to prevent external modification
      return items.map(item => ({ ...item }));
    },
    
    getTotal() {
      return total;
    },
    
    clear() {
      items = [];
      total = 0;
      console.log('Cart cleared');
    },
    
    getItemCount() {
      return items.reduce((count, item) => count + item.quantity, 0);
    }
  };
})();

// Usage
ShoppingCart.addItem({ id: 1, name: 'Laptop', price: 999.99, quantity: 1 });
ShoppingCart.addItem({ id: 2, name: 'Mouse', price: 29.99, quantity: 2 });

console.log('Items:', ShoppingCart.getItems());
console.log('Total:', ShoppingCart.getTotal()); // 1059.97
console.log('Count:', ShoppingCart.getItemCount()); // 3

// Private variables are inaccessible
// console.log(items); // ReferenceError: items is not defined
```

**Advanced Module Pattern with Configuration:**

```javascript
const DatabaseConnection = (function() {
  // Private state
  let connection = null;
  let config = {
    host: 'localhost',
    port: 5432,
    database: 'mydb',
    maxRetries: 3
  };
  
  let isConnected = false;
  let queryLog = [];
  
  // Private methods
  function log(message, type = 'INFO') {
    const timestamp = new Date().toISOString();
    const logEntry = `[${timestamp}] [${type}] ${message}`;
    queryLog.push(logEntry);
    console.log(logEntry);
  }
  
  function validateQuery(query) {
    if (typeof query !== 'string' || query.trim().length === 0) {
      throw new Error('Invalid query');
    }
  }
  
  async function executeQuery(query) {
    // Simulated database query
    return new Promise((resolve) => {
      setTimeout(() => {
        resolve({ success: true, data: [], query });
      }, 100);
    });
  }
  
  // Public API
  return {
    configure(options) {
      config = { ...config, ...options };
      log(`Configuration updated: ${JSON.stringify(options)}`);
      return this;
    },
    
    async connect() {
      if (isConnected) {
        log('Already connected', 'WARN');
        return true;
      }
      
      log(`Connecting to ${config.host}:${config.port}/${config.database}`);
      
      // Simulated connection
      await new Promise(resolve => setTimeout(resolve, 500));
      
      connection = { id: Date.now() };
      isConnected = true;
      log('Connected successfully');
      
      return true;
    },
    
    async disconnect() {
      if (!isConnected) {
        log('Not connected', 'WARN');
        return true;
      }
      
      log('Disconnecting...');
      await new Promise(resolve => setTimeout(resolve, 200));
      
      connection = null;
      isConnected = false;
      log('Disconnected');
      
      return true;
    },
    
    async query(sql) {
      if (!isConnected) {
        throw new Error('Not connected to database');
      }
      
      validateQuery(sql);
      log(`Executing query: ${sql}`);
      
      try {
        const result = await executeQuery(sql);
        log(`Query executed successfully`, 'SUCCESS');
        return result;
      } catch (error) {
        log(`Query failed: ${error.message}`, 'ERROR');
        throw error;
      }
    },
    
    getConnectionStatus() {
      return {
        connected: isConnected,
        connectionId: connection?.id,
        config: { ...config } // Return copy
      };
    },
    
    getQueryLog() {
      return [...queryLog]; // Return copy
    },
    
    clearLog() {
      queryLog = [];
      log('Query log cleared');
    }
  };
})();

// Usage
await DatabaseConnection
  .configure({ host: 'db.example.com', port: 3306 })
  .connect();

await DatabaseConnection.query('SELECT * FROM users');
await DatabaseConnection.query('INSERT INTO logs VALUES (...)');

console.log(DatabaseConnection.getConnectionStatus());
console.log(DatabaseConnection.getQueryLog());

await DatabaseConnection.disconnect();
```

---

### **2. Revealing Module Pattern**

Similar to Module Pattern but explicitly defines what to expose, making the public API more obvious.

```javascript
const UserManager = (function() {
  // Private variables
  let users = [];
  let currentUserId = 1;
  
  // Private functions
  function generateId() {
    return currentUserId++;
  }
  
  function validateUser(user) {
    if (!user.name || !user.email) {
      throw new Error('Name and email are required');
    }
    
    if (!isValidEmail(user.email)) {
      throw new Error('Invalid email format');
    }
    
    return true;
  }
  
  function isValidEmail(email) {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
  }
  
  function findUserById(id) {
    return users.find(u => u.id === id);
  }
  
  function findUserByEmail(email) {
    return users.find(u => u.email === email);
  }
  
  // Public functions
  function createUser(userData) {
    validateUser(userData);
    
    if (findUserByEmail(userData.email)) {
      throw new Error('User with this email already exists');
    }
    
    const user = {
      id: generateId(),
      name: userData.name,
      email: userData.email,
      role: userData.role || 'user',
      createdAt: new Date(),
      active: true
    };
    
    users.push(user);
    console.log(`User created: ${user.name} (${user.email})`);
    
    return { ...user };
  }
  
  function getUserById(id) {
    const user = findUserById(id);
    return user ? { ...user } : null;
  }
  
  function getAllUsers() {
    return users.map(u => ({ ...u }));
  }
  
  function updateUser(id, updates) {
    const user = findUserById(id);
    
    if (!user) {
      throw new Error('User not found');
    }
    
    if (updates.email && updates.email !== user.email) {
      if (findUserByEmail(updates.email)) {
        throw new Error('Email already in use');
      }
    }
    
    Object.assign(user, updates, { updatedAt: new Date() });
    console.log(`User updated: ${user.name}`);
    
    return { ...user };
  }
  
  function deleteUser(id) {
    const index = users.findIndex(u => u.id === id);
    
    if (index === -1) {
      throw new Error('User not found');
    }
    
    const deleted = users.splice(index, 1)[0];
    console.log(`User deleted: ${deleted.name}`);
    
    return true;
  }
  
  function deactivateUser(id) {
    const user = findUserById(id);
    
    if (!user) {
      throw new Error('User not found');
    }
    
    user.active = false;
    user.deactivatedAt = new Date();
    console.log(`User deactivated: ${user.name}`);
    
    return true;
  }
  
  function searchUsers(query) {
    const lowercaseQuery = query.toLowerCase();
    
    return users
      .filter(u => 
        u.name.toLowerCase().includes(lowercaseQuery) ||
        u.email.toLowerCase().includes(lowercaseQuery)
      )
      .map(u => ({ ...u }));
  }
  
  function getUserCount() {
    return {
      total: users.length,
      active: users.filter(u => u.active).length,
      inactive: users.filter(u => !u.active).length
    };
  }
  
  // Reveal public API (explicitly defined)
  return {
    create: createUser,
    getById: getUserById,
    getAll: getAllUsers,
    update: updateUser,
    delete: deleteUser,
    deactivate: deactivateUser,
    search: searchUsers,
    count: getUserCount
  };
})();

// Usage
const user1 = UserManager.create({
  name: 'John Doe',
  email: 'john@example.com',
  role: 'admin'
});

const user2 = UserManager.create({
  name: 'Jane Smith',
  email: 'jane@example.com'
});

console.log('User by ID:', UserManager.getById(1));
console.log('All users:', UserManager.getAll());

UserManager.update(1, { role: 'superadmin' });
UserManager.deactivate(2);

console.log('Search results:', UserManager.search('john'));
console.log('User count:', UserManager.count());
```

---

### **3. Singleton Pattern**

Ensures only one instance of a class exists and provides global access to it.

```javascript
// ES6 Class Singleton
class Database {
  constructor() {
    if (Database.instance) {
      return Database.instance;
    }
    
    this.connection = null;
    this.queries = [];
    this.connected = false;
    
    Database.instance = this;
  }
  
  connect(connectionString) {
    if (this.connected) {
      console.log('Already connected');
      return this;
    }
    
    console.log(`Connecting to: ${connectionString}`);
    this.connection = { url: connectionString, id: Date.now() };
    this.connected = true;
    
    return this;
  }
  
  disconnect() {
    if (!this.connected) {
      console.log('Not connected');
      return this;
    }
    
    console.log('Disconnecting...');
    this.connection = null;
    this.connected = false;
    
    return this;
  }
  
  query(sql) {
    if (!this.connected) {
      throw new Error('Not connected to database');
    }
    
    console.log(`Executing: ${sql}`);
    this.queries.push({ sql, timestamp: new Date() });
    
    return { success: true, data: [] };
  }
  
  getQueryHistory() {
    return [...this.queries];
  }
  
  static getInstance() {
    if (!Database.instance) {
      Database.instance = new Database();
    }
    return Database.instance;
  }
}

// Usage
const db1 = new Database();
db1.connect('mongodb://localhost:27017');
db1.query('SELECT * FROM users');

const db2 = new Database(); // Returns same instance
db2.query('SELECT * FROM posts');

console.log(db1 === db2); // true
console.log(db1.getQueryHistory()); // Contains queries from both db1 and db2

// Alternative: Using static getInstance
const db3 = Database.getInstance();
console.log(db1 === db3); // true
```

**Singleton with Lazy Initialization:**

```javascript
const ConfigManager = (function() {
  let instance;
  
  function createInstance() {
    // Private configuration
    let config = {
      apiUrl: 'https://api.example.com',
      timeout: 5000,
      retryAttempts: 3,
      environment: 'development'
    };
    
    let listeners = [];
    
    // Private methods
    function notifyListeners(key, value) {
      listeners.forEach(callback => callback(key, value));
    }
    
    // Public methods
    return {
      get(key) {
        return key ? config[key] : { ...config };
      },
      
      set(key, value) {
        const oldValue = config[key];
        config[key] = value;
        
        console.log(`Config updated: ${key} = ${value}`);
        notifyListeners(key, value);
        
        return this;
      },
      
      setMultiple(updates) {
        Object.keys(updates).forEach(key => {
          this.set(key, updates[key]);
        });
        return this;
      },
      
      reset() {
        config = {
          apiUrl: 'https://api.example.com',
          timeout: 5000,
          retryAttempts: 3,
          environment: 'development'
        };
        console.log('Config reset to defaults');
        return this;
      },
      
      onChange(callback) {
        listeners.push(callback);
        return () => {
          listeners = listeners.filter(cb => cb !== callback);
        };
      },
      
      getEnvironment() {
        return config.environment;
      },
      
      isProduction() {
        return config.environment === 'production';
      }
    };
  }
  
  return {
    getInstance() {
      if (!instance) {
        instance = createInstance();
        console.log('ConfigManager instance created');
      }
      return instance;
    }
  };
})();

// Usage
const config1 = ConfigManager.getInstance();
config1.set('apiUrl', 'https://api.production.com');
config1.set('environment', 'production');

const config2 = ConfigManager.getInstance(); // Same instance
console.log(config2.get('apiUrl')); // 'https://api.production.com'

console.log(config1 === config2); // true

// Listen to config changes
const unsubscribe = config1.onChange((key, value) => {
  console.log(`Config changed: ${key} = ${value}`);
});

config1.set('timeout', 10000); // Triggers listener
```

**Application-wide Singleton:**

```javascript
class AppState {
  constructor() {
    if (AppState.instance) {
      return AppState.instance;
    }
    
    this.state = {
      user: null,
      theme: 'light',
      language: 'en',
      notifications: [],
      isLoading: false
    };
    
    this.subscribers = new Map();
    this.history = [];
    
    AppState.instance = this;
  }
  
  getState() {
    return { ...this.state };
  }
  
  setState(updates) {
    const oldState = { ...this.state };
    this.state = { ...this.state, ...updates };
    
    this.history.push({
      timestamp: new Date(),
      oldState,
      newState: { ...this.state }
    });
    
    // Notify subscribers
    Object.keys(updates).forEach(key => {
      if (this.subscribers.has(key)) {
        this.subscribers.get(key).forEach(callback => {
          callback(this.state[key], oldState[key]);
        });
      }
    });
    
    return this;
  }
  
  subscribe(key, callback) {
    if (!this.subscribers.has(key)) {
      this.subscribers.set(key, []);
    }
    
    this.subscribers.get(key).push(callback);
    
    // Return unsubscribe function
    return () => {
      const callbacks = this.subscribers.get(key);
      const index = callbacks.indexOf(callback);
      if (index > -1) {
        callbacks.splice(index, 1);
      }
    };
  }
  
  login(user) {
    this.setState({ user, isLoading: false });
    console.log(`User logged in: ${user.name}`);
  }
  
  logout() {
    this.setState({ user: null });
    console.log('User logged out');
  }
  
  setTheme(theme) {
    this.setState({ theme });
  }
  
  addNotification(notification) {
    const notifications = [...this.state.notifications, {
      id: Date.now(),
      ...notification,
      timestamp: new Date()
    }];
    this.setState({ notifications });
  }
  
  clearNotifications() {
    this.setState({ notifications: [] });
  }
  
  getHistory() {
    return [...this.history];
  }
  
  static getInstance() {
    if (!AppState.instance) {
      AppState.instance = new AppState();
    }
    return AppState.instance;
  }
}

// Usage across application
const appState = AppState.getInstance();

// Subscribe to user changes
appState.subscribe('user', (newUser, oldUser) => {
  console.log('User changed:', newUser);
});

// Subscribe to theme changes
appState.subscribe('theme', (newTheme) => {
  document.body.className = `theme-${newTheme}`;
});

// Login
appState.login({ id: 1, name: 'John Doe', email: 'john@example.com' });

// Change theme
appState.setTheme('dark');

// Add notification
appState.addNotification({
  type: 'success',
  message: 'Profile updated successfully'
});

// Access from anywhere in app
const sameInstance = AppState.getInstance();
console.log(sameInstance.getState().user); // Same user
```

---

### **4. Factory Pattern**

Creates objects without specifying their exact classes, providing an interface for object creation.

```javascript
// Simple Factory
class UserFactory {
  static createUser(type, data) {
    switch (type) {
      case 'admin':
        return new AdminUser(data);
      case 'moderator':
        return new ModeratorUser(data);
      case 'customer':
        return new CustomerUser(data);
      case 'guest':
        return new GuestUser(data);
      default:
        throw new Error(`Unknown user type: ${type}`);
    }
  }
}

// Base User class
class User {
  constructor(data) {
    this.id = data.id;
    this.name = data.name;
    this.email = data.email;
    this.createdAt = new Date();
  }
  
  getInfo() {
    return {
      id: this.id,
      name: this.name,
      email: this.email,
      role: this.constructor.name.replace('User', '').toLowerCase()
    };
  }
}

// Specific user types
class AdminUser extends User {
  constructor(data) {
    super(data);
    this.permissions = ['read', 'write', 'delete', 'manage_users'];
  }
  
  canManageUsers() {
    return true;
  }
  
  canAccessAdminPanel() {
    return true;
  }
  
  canDeleteAnyContent() {
    return true;
  }
}

class ModeratorUser extends User {
  constructor(data) {
    super(data);
    this.permissions = ['read', 'write', 'delete'];
  }
  
  canManageUsers() {
    return false;
  }
  
  canAccessAdminPanel() {
    return false;
  }
  
  canDeleteAnyContent() {
    return true;
  }
}

class CustomerUser extends User {
  constructor(data) {
    super(data);
    this.permissions = ['read', 'write'];
    this.purchaseHistory = [];
  }
  
  canManageUsers() {
    return false;
  }
  
  canAccessAdminPanel() {
    return false;
  }
  
  canDeleteAnyContent() {
    return false;
  }
  
  addPurchase(purchase) {
    this.purchaseHistory.push(purchase);
  }
}

class GuestUser extends User {
  constructor(data) {
    super(data);
    this.permissions = ['read'];
  }
  
  canManageUsers() {
    return false;
  }
  
  canAccessAdminPanel() {
    return false;
  }
  
  canDeleteAnyContent() {
    return false;
  }
}

// Usage
const admin = UserFactory.createUser('admin', {
  id: 1,
  name: 'Admin User',
  email: 'admin@example.com'
});

const customer = UserFactory.createUser('customer', {
  id: 2,
  name: 'John Doe',
  email: 'john@example.com'
});

console.log(admin.getInfo());
console.log(admin.canManageUsers()); // true
console.log(customer.canManageUsers()); // false
```

**Advanced Factory with Registration:**

```javascript
class VehicleFactory {
  constructor() {
    this.vehicles = new Map();
  }
  
  // Register vehicle types
  register(type, vehicleClass) {
    if (this.vehicles.has(type)) {
      console.warn(`Vehicle type ${type} is already registered`);
    }
    this.vehicles.set(type, vehicleClass);
    return this;
  }
  
  // Create vehicle instance
  create(type, options) {
    const VehicleClass = this.vehicles.get(type);
    
    if (!VehicleClass) {
      throw new Error(`Vehicle type ${type} is not registered`);
    }
    
    return new VehicleClass(options);
  }
  
  // Check if type is registered
  isRegistered(type) {
    return this.vehicles.has(type);
  }
  
  // Get all registered types
  getRegisteredTypes() {
    return Array.from(this.vehicles.keys());
  }
}

// Base Vehicle class
class Vehicle {
  constructor(options) {
    this.brand = options.brand;
    this.model = options.model;
    this.year = options.year;
    this.price = options.price;
  }
  
  getInfo() {
    return `${this.year} ${this.brand} ${this.model} - $${this.price}`;
  }
}

// Specific vehicle types
class Car extends Vehicle {
  constructor(options) {
    super(options);
    this.type = 'car';
    this.doors = options.doors || 4;
    this.fuelType = options.fuelType || 'gasoline';
  }
  
  getDetails() {
    return `${this.getInfo()} | ${this.doors} doors, ${this.fuelType}`;
  }
}

class Motorcycle extends Vehicle {
  constructor(options) {
    super(options);
    this.type = 'motorcycle';
    this.engineCC = options.engineCC;
    this.hasABS = options.hasABS || false;
  }
  
  getDetails() {
    return `${this.getInfo()} | ${this.engineCC}cc, ABS: ${this.hasABS}`;
  }
}

class Truck extends Vehicle {
  constructor(options) {
    super(options);
    this.type = 'truck';
    this.cargoCapacity = options.cargoCapacity;
    this.axles = options.axles || 2;
  }
  
  getDetails() {
    return `${this.getInfo()} | Capacity: ${this.cargoCapacity}t, ${this.axles} axles`;
  }
}

class ElectricCar extends Car {
  constructor(options) {
    super({ ...options, fuelType: 'electric' });
    this.batteryCapacity = options.batteryCapacity;
    this.range = options.range;
  }
  
  getDetails() {
    return `${super.getDetails()} | Battery: ${this.batteryCapacity}kWh, Range: ${this.range}km`;
  }
}

// Create factory and register vehicle types
const factory = new VehicleFactory();

factory
  .register('car', Car)
  .register('motorcycle', Motorcycle)
  .register('truck', Truck)
  .register('electric', ElectricCar);

// Usage
const car = factory.create('car', {
  brand: 'Toyota',
  model: 'Camry',
  year: 2024,
  price: 28000,
  doors: 4,
  fuelType: 'hybrid'
});

const motorcycle = factory.create('motorcycle', {
  brand: 'Harley-Davidson',
  model: 'Street 750',
  year: 2024,
  price: 8000,
  engineCC: 750,
  hasABS: true
});

const electric = factory.create('electric', {
  brand: 'Tesla',
  model: 'Model 3',
  year: 2024,
  price: 45000,
  batteryCapacity: 75,
  range: 500
});

console.log(car.getDetails());
console.log(motorcycle.getDetails());
console.log(electric.getDetails());

console.log('Registered types:', factory.getRegisteredTypes());
```

**Real-World Factory: Logger Factory:**

```javascript
class LoggerFactory {
  static createLogger(type, config = {}) {
    const loggers = {
      console: ConsoleLogger,
      file: FileLogger,
      remote: RemoteLogger,
      multi: MultiLogger
    };
    
    const LoggerClass = loggers[type];
    
    if (!LoggerClass) {
      throw new Error(`Unknown logger type: ${type}`);
    }
    
    return new LoggerClass(config);
  }
}

// Base Logger
class Logger {
  constructor(config) {
    this.level = config.level || 'info';
    this.levels = {
      debug: 0,
      info: 1,
      warn: 2,
      error: 3
    };
  }
  
  shouldLog(level) {
    return this.levels[level] >= this.levels[this.level];
  }
  
  formatMessage(level, message, data) {
    const timestamp = new Date().toISOString();
    const dataStr = data ? ` | ${JSON.stringify(data)}` : '';
    return `[${timestamp}] [${level.toUpperCase()}] ${message}${dataStr}`;
  }
}

// Console Logger
class ConsoleLogger extends Logger {
  log(level, message, data) {
    if (!this.shouldLog(level)) return;
    
    const formatted = this.formatMessage(level, message, data);
    
    switch (level) {
      case 'debug':
        console.debug(formatted);
        break;
      case 'info':
        console.info(formatted);
        break;
      case 'warn':
        console.warn(formatted);
        break;
      case 'error':
        console.error(formatted);
        break;
    }
  }
  
  debug(message, data) { this.log('debug', message, data); }
  info(message, data) { this.log('info', message, data); }
  warn(message, data) { this.log('warn', message, data); }
  error(message, data) { this.log('error', message, data); }
}

// File Logger (simulated)
class FileLogger extends Logger {
  constructor(config) {
    super(config);
    this.filePath = config.filePath || 'app.log';
    this.logs = [];
  }
  
  log(level, message, data) {
    if (!this.shouldLog(level)) return;
    
    const formatted = this.formatMessage(level, message, data);
    this.logs.push(formatted);
    
    // In real implementation, write to file
    console.log(`[FILE: ${this.filePath}] ${formatted}`);
  }
  
  debug(message, data) { this.log('debug', message, data); }
  info(message, data) { this.log('info', message, data); }
  warn(message, data) { this.log('warn', message, data); }
  error(message, data) { this.log('error', message, data); }
  
  getLogs() {
    return [...this.logs];
  }
}

// Remote Logger (simulated)
class RemoteLogger extends Logger {
  constructor(config) {
    super(config);
    this.endpoint = config.endpoint || 'https://logs.example.com/api';
    this.buffer = [];
    this.batchSize = config.batchSize || 10;
  }
  
  async log(level, message, data) {
    if (!this.shouldLog(level)) return;
    
    const logEntry = {
      timestamp: new Date().toISOString(),
      level,
      message,
      data
    };
    
    this.buffer.push(logEntry);
    
    if (this.buffer.length >= this.batchSize) {
      await this.flush();
    }
  }
  
  async flush() {
    if (this.buffer.length === 0) return;
    
    const logsToSend = [...this.buffer];
    this.buffer = [];
    
    console.log(`[REMOTE: ${this.endpoint}] Sending ${logsToSend.length} logs`);
    
    // Simulated API call
    // await fetch(this.endpoint, {
    //   method: 'POST',
    //   body: JSON.stringify(logsToSend)
    // });
  }
  
  debug(message, data) { return this.log('debug', message, data); }
  info(message, data) { return this.log('info', message, data); }
  warn(message, data) { return this.log('warn', message, data); }
  error(message, data) { return this.log('error', message, data); }
}

// Multi Logger (logs to multiple destinations)
class MultiLogger extends Logger {
  constructor(config) {
    super(config);
    this.loggers = config.loggers || [];
  }
  
  log(level, message, data) {
    this.loggers.forEach(logger => {
      logger.log(level, message, data);
    });
  }
  
  debug(message, data) { this.log('debug', message, data); }
  info(message, data) { this.log('info', message, data); }
  warn(message, data) { this.log('warn', message, data); }
  error(message, data) { this.log('error', message, data); }
}

// Usage
const consoleLogger = LoggerFactory.createLogger('console', { level: 'debug' });
consoleLogger.info('Application started');
consoleLogger.debug('Debug information', { userId: 123 });

const fileLogger = LoggerFactory.createLogger('file', {
  level: 'warn',
  filePath: 'app.log'
});
fileLogger.warn('Warning message');
fileLogger.error('Error occurred', { code: 500 });

const remoteLogger = LoggerFactory.createLogger('remote', {
  level: 'info',
  endpoint: 'https://logs.example.com/api',
  batchSize: 5
});
remoteLogger.info('User logged in', { userId: 456 });

// Multi-logger (logs to console and file)
const multiLogger = LoggerFactory.createLogger('multi', {
  level: 'info',
  loggers: [
    LoggerFactory.createLogger('console'),
    LoggerFactory.createLogger('file', { filePath: 'combined.log' })
  ]
});
multiLogger.info('This goes to both console and file');
```

---

## **BEHAVIORAL PATTERNS**

### **5. Observer Pattern (Pub/Sub)**

Defines a one-to-many dependency where changes in one object notify multiple dependent objects.

```javascript
// Event Emitter / Observer Pattern
class EventEmitter {
  constructor() {
    this.events = new Map();
  }
  
  // Subscribe to event
  on(event, callback) {
    if (!this.events.has(event)) {
      this.events.set(event, []);
    }
    
    this.events.get(event).push(callback);
    
    // Return unsubscribe function
    return () => this.off(event, callback);
  }
  
  // Subscribe once (auto-unsubscribe after first call)
  once(event, callback) {
    const onceCallback = (...args) => {
      callback(...args);
      this.off(event, onceCallback);
    };
    
    return this.on(event, onceCallback);
  }
  
  // Unsubscribe from event
  off(event, callback) {
    if (!this.events.has(event)) return;
    
    if (callback) {
      const callbacks = this.events.get(event);
      const index = callbacks.indexOf(callback);
      if (index > -1) {
        callbacks.splice(index, 1);
      }
    } else {
      // Remove all listeners for this event
      this.events.delete(event);
    }
  }
  
  // Emit event
  emit(event, ...args) {
    if (!this.events.has(event)) return;
    
    const callbacks = this.events.get(event);
    callbacks.forEach(callback => {
      try {
        callback(...args);
      } catch (error) {
        console.error(`Error in event listener for ${event}:`, error);
      }
    });
  }
  
  // Get listener count
  listenerCount(event) {
    return this.events.has(event) ? this.events.get(event).length : 0;
  }
  
  // Remove all listeners
  removeAllListeners(event) {
    if (event) {
      this.events.delete(event);
    } else {
      this.events.clear();
    }
  }
  
  // Get all event names
  eventNames() {
    return Array.from(this.events.keys());
  }
}

// Usage
const emitter = new EventEmitter();

// Subscribe to events
emitter.on('user:login', (user) => {
  console.log(`User logged in: ${user.name}`);
});

emitter.on('user:login', (user) => {
  console.log(`Welcome back, ${user.name}!`);
});

emitter.on('user:logout', (user) => {
  console.log(`User logged out: ${user.name}`);
});

// Subscribe once
emitter.once('app:ready', () => {
  console.log('Application is ready (this will only log once)');
});

// Emit events
emitter.emit('user:login', { name: 'John Doe', id: 1 });
// Output:
// User logged in: John Doe
// Welcome back, John Doe!

emitter.emit('app:ready'); // Logs
emitter.emit('app:ready'); // Doesn't log (once)

emitter.emit('user:logout', { name: 'John Doe', id: 1 });

console.log('Listener count for user:login:', emitter.listenerCount('user:login')); // 2
```

**Advanced Observer: Store with State Management:**

```javascript
class Store extends EventEmitter {
  constructor(initialState = {}) {
    super();
    this.state = initialState;
    this.history = [{ ...initialState }];
    this.maxHistory = 50;
  }
  
  // Get current state
  getState() {
    return { ...this.state };
  }
  
  // Set state and notify observers
  setState(updates, silent = false) {
    const oldState = { ...this.state };
    this.state = { ...this.state, ...updates };
    
    // Add to history
    this.history.push({ ...this.state });
    if (this.history.length > this.maxHistory) {
      this.history.shift();
    }
    
    if (!silent) {
      // Emit specific change events
      Object.keys(updates).forEach(key => {
        this.emit(`change:${key}`, this.state[key], oldState[key]);
      });
      
      // Emit general change event
      this.emit('change', this.state, oldState);
    }
    
    return this;
  }
  
  // Subscribe to specific state property
  subscribe(key, callback) {
    return this.on(`change:${key}`, callback);
  }
  
  // Subscribe to all changes
  subscribeAll(callback) {
    return this.on('change', callback);
  }
  
  // Reset state
  reset(silent = false) {
    const initialState = this.history[0] || {};
    this.setState(initialState, silent);
    return this;
  }
  
  // Undo last state change
  undo() {
    if (this.history.length < 2) return this;
    
    this.history.pop(); // Remove current
    const previousState = this.history[this.history.length - 1];
    this.state = { ...previousState };
    
    this.emit('change', this.state, this.state);
    return this;
  }
  
  // Get state history
  getHistory() {
    return [...this.history];
  }
}

// Usage
const store = new Store({
  user: null,
  theme: 'light',
  notifications: []
});

// Subscribe to specific property
store.subscribe('user', (newUser, oldUser) => {
  console.log('User changed:', newUser);
});

store.subscribe('theme', (newTheme, oldTheme) => {
  console.log(`Theme changed from ${oldTheme} to ${newTheme}`);
  document.body.className = `theme-${newTheme}`;
});

// Subscribe to all changes
store.subscribeAll((newState, oldState) => {
  console.log('State updated:', newState);
});

// Update state
store.setState({ user: { name: 'John', id: 1 } });
store.setState({ theme: 'dark' });
store.setState({
  notifications: [
    { id: 1, message: 'Welcome!', type: 'info' }
  ]
});

console.log('Current state:', store.getState());
console.log('State history:', store.getHistory());
```

**Real-World Observer: Chat Application:**

```javascript
class ChatRoom extends EventEmitter {
  constructor(name) {
    super();
    this.name = name;
    this.users = new Map();
    this.messages = [];
    this.typingUsers = new Set();
  }
  
  // User joins room
  join(user) {
    if (this.users.has(user.id)) {
      console.log(`${user.name} is already in the room`);
      return;
    }
    
    this.users.set(user.id, user);
    
    this.emit('user:join', {
      user,
      room: this.name,
      userCount: this.users.size
    });
    
    console.log(`${user.name} joined ${this.name}`);
  }
  
  // User leaves room
  leave(userId) {
    const user = this.users.get(userId);
    
    if (!user) return;
    
    this.users.delete(userId);
    this.typingUsers.delete(userId);
    
    this.emit('user:leave', {
      user,
      room: this.name,
      userCount: this.users.size
    });
    
    console.log(`${user.name} left ${this.name}`);
  }
  
  // Send message
  sendMessage(userId, text) {
    const user = this.users.get(userId);
    
    if (!user) {
      throw new Error('User not in room');
    }
    
    const message = {
      id: Date.now(),
      userId,
      userName: user.name,
      text,
      timestamp: new Date()
    };
    
    this.messages.push(message);
    this.typingUsers.delete(userId);
    
    this.emit('message', message);
    this.emit('typing:stop', { userId, userName: user.name });
    
    return message;
  }
  
  // User typing indicator
  setTyping(userId, isTyping) {
    const user = this.users.get(userId);
    
    if (!user) return;
    
    if (isTyping) {
      this.typingUsers.add(userId);
      this.emit('typing:start', { userId, userName: user.name });
    } else {
      this.typingUsers.delete(userId);
      this.emit('typing:stop', { userId, userName: user.name });
    }
  }
  
  // Get room info
  getInfo() {
    return {
      name: this.name,
      userCount: this.users.size,
      messageCount: this.messages.length,
      users: Array.from(this.users.values()),
      typingUsers: Array.from(this.typingUsers)
    };
  }
  
  // Get messages
  getMessages(limit = 50) {
    return this.messages.slice(-limit);
  }
}

// Usage
const generalRoom = new ChatRoom('General');

// Setup listeners
generalRoom.on('user:join', ({ user, userCount }) => {
  console.log(`📢 ${user.name} joined the room (${userCount} users online)`);
});

generalRoom.on('user:leave', ({ user, userCount }) => {
  console.log(`📢 ${user.name} left the room (${userCount} users online)`);
});

generalRoom.on('message', (message) => {
  console.log(`💬 ${message.userName}: ${message.text}`);
});

generalRoom.on('typing:start', ({ userName }) => {
  console.log(`✏️  ${userName} is typing...`);
});

generalRoom.on('typing:stop', ({ userName }) => {
  console.log(`✏️  ${userName} stopped typing`);
});

// Simulate users
const john = { id: 1, name: 'John' };
const jane = { id: 2, name: 'Jane' };

generalRoom.join(john);
generalRoom.join(jane);

generalRoom.setTyping(john.id, true);
setTimeout(() => {
  generalRoom.sendMessage(john.id, 'Hello everyone!');
}, 1000);

setTimeout(() => {
  generalRoom.setTyping(jane.id, true);
}, 1500);

setTimeout(() => {
  generalRoom.sendMessage(jane.id, 'Hi John!');
}, 2500);

setTimeout(() => {
  generalRoom.leave(john.id);
}, 3000);

console.log('Room info:', generalRoom.getInfo());
```

---

### **6. Strategy Pattern**

Defines a family of algorithms, encapsulates each one, and makes them interchangeable. Strategy lets the algorithm vary independently from clients that use it.

```javascript
// Payment strategies
class PaymentStrategy {
  pay(amount) {
    throw new Error('pay() must be implemented');
  }
  
  validate() {
    throw new Error('validate() must be implemented');
  }
}

// Concrete strategies
class CreditCardStrategy extends PaymentStrategy {
  constructor(cardNumber, cvv, expiryDate) {
    super();
    this.cardNumber = cardNumber;
    this.cvv = cvv;
    this.expiryDate = expiryDate;
  }
  
  validate() {
    if (this.cardNumber.length !== 16) {
      throw new Error('Invalid card number');
    }
    
    if (this.cvv.length !== 3) {
      throw new Error('Invalid CVV');
    }
    
    const [month, year] = this.expiryDate.split('/');
    const expiry = new Date(2000 + parseInt(year), parseInt(month) - 1);
    
    if (expiry < new Date()) {
      throw new Error('Card expired');
    }
    
    return true;
  }
  
  pay(amount) {
    this.validate();
    console.log(`Processing credit card payment of $${amount}`);
    console.log(`Card: **** **** **** ${this.cardNumber.slice(-4)}`);
    
    // Simulate payment processing
    return {
      success: true,
      transactionId: `CC-${Date.now()}`,
      amount,
      method: 'Credit Card'
    };
  }
}

class PayPalStrategy extends PaymentStrategy {
  constructor(email, password) {
    super();
    this.email = email;
    this.password = password;
  }
  
  validate() {
    if (!this.email.includes('@')) {
      throw new Error('Invalid email');
    }
    
    if (this.password.length < 6) {
      throw new Error('Invalid password');
    }
    
    return true;
  }
  
  pay(amount) {
    this.validate();
    console.log(`Processing PayPal payment of $${amount}`);
    console.log(`PayPal account: ${this.email}`);
    
    return {
      success: true,
      transactionId: `PP-${Date.now()}`,
      amount,
      method: 'PayPal'
    };
  }
}

class CryptoStrategy extends PaymentStrategy {
  constructor(walletAddress, currency = 'BTC') {
    super();
    this.walletAddress = walletAddress;
    this.currency = currency;
  }
  
  validate() {
    if (this.walletAddress.length < 26) {
      throw new Error('Invalid wallet address');
    }
    
    return true;
  }
  
  pay(amount) {
    this.validate();
    console.log(`Processing ${this.currency} payment of $${amount}`);
    console.log(`Wallet: ${this.walletAddress.slice(0, 10)}...${this.walletAddress.slice(-10)}`);
    
    return {
      success: true,
      transactionId: `CRYPTO-${Date.now()}`,
      amount,
      method: `Cryptocurrency (${this.currency})`
    };
  }
}

// Context class that uses strategies
class ShoppingCart {
  constructor() {
    this.items = [];
    this.paymentStrategy = null;
  }
  
  addItem(item) {
    this.items.push(item);
    console.log(`Added: ${item.name} - $${item.price}`);
  }
  
  getTotal() {
    return this.items.reduce((sum, item) => sum + item.price, 0);
  }
  
  setPaymentStrategy(strategy) {
    this.paymentStrategy = strategy;
    console.log(`Payment method set: ${strategy.constructor.name}`);
  }
  
  checkout() {
    if (!this.paymentStrategy) {
      throw new Error('No payment method selected');
    }
    
    const total = this.getTotal();
    console.log(`\nCheckout Summary:`);
    console.log(`Items: ${this.items.length}`);
    console.log(`Total: $${total}\n`);
    
    const result = this.paymentStrategy.pay(total);
    
    if (result.success) {
      console.log(`\n✓ Payment successful!`);
      console.log(`Transaction ID: ${result.transactionId}`);
      this.items = []; // Clear cart
    }
    
    return result;
  }
}

// Usage
const cart = new ShoppingCart();

cart.addItem({ name: 'Laptop', price: 999.99 });
cart.addItem({ name: 'Mouse', price: 29.99 });
cart.addItem({ name: 'Keyboard', price: 79.99 });

// Pay with credit card
cart.setPaymentStrategy(
  new CreditCardStrategy('1234567890123456', '123', '12/25')
);
cart.checkout();

// New cart with PayPal
const cart2 = new ShoppingCart();
cart2.addItem({ name: 'Monitor', price: 299.99 });
cart2.setPaymentStrategy(
  new PayPalStrategy('user@example.com', 'password123')
);
cart2.checkout();

// New cart with Crypto
const cart3 = new ShoppingCart();
cart3.addItem({ name: 'Headphones', price: 149.99 });
cart3.setPaymentStrategy(
  new CryptoStrategy('1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa', 'BTC')
);
cart3.checkout();
```

**Advanced Strategy: Sorting Algorithms:**

```javascript
class SortStrategy {
  sort(array) {
    throw new Error('sort() must be implemented');
  }
}

class BubbleSortStrategy extends SortStrategy {
  sort(array) {
    console.log('Using Bubble Sort');
    const arr = [...array];
    const n = arr.length;
    
    for (let i = 0; i < n - 1; i++) {
      for (let j = 0; j < n - i - 1; j++) {
        if (arr[j] > arr[j + 1]) {
          [arr[j], arr[j + 1]] = [arr[j + 1], arr[j]];
        }
      }
    }
    
    return arr;
  }
}

class QuickSortStrategy extends SortStrategy {
  sort(array) {
    console.log('Using Quick Sort');
    return this.quickSort([...array]);
  }
  
  quickSort(arr) {
    if (arr.length <= 1) return arr;
    
    const pivot = arr[Math.floor(arr.length / 2)];
    const left = arr.filter(x => x < pivot);
    const middle = arr.filter(x => x === pivot);
    const right = arr.filter(x => x > pivot);
    
    return [...this.quickSort(left), ...middle, ...this.quickSort(right)];
  }
}

class MergeSortStrategy extends SortStrategy {
  sort(array) {
    console.log('Using Merge Sort');
    return this.mergeSort([...array]);
  }
  
  mergeSort(arr) {
    if (arr.length <= 1) return arr;
    
    const mid = Math.floor(arr.length / 2);
    const left = this.mergeSort(arr.slice(0, mid));
    const right = this.mergeSort(arr.slice(mid));
    
    return this.merge(left, right);
  }
  
  merge(left, right) {
    const result = [];
    let i = 0, j = 0;
    
    while (i < left.length && j < right.length) {
      if (left[i] <= right[j]) {
        result.push(left[i++]);
      } else {
        result.push(right[j++]);
      }
    }
    
    return result.concat(left.slice(i)).concat(right.slice(j));
  }
}

class Sorter {
  constructor(strategy) {
    this.strategy = strategy;
  }
  
  setStrategy(strategy) {
    this.strategy = strategy;
  }
  
  sort(array) {
    console.time('Sort time');
    const result = this.strategy.sort(array);
    console.timeEnd('Sort time');
    return result;
  }
}

// Usage
const data = [64, 34, 25, 12, 22, 11, 90, 88, 45, 50];

const sorter = new Sorter(new BubbleSortStrategy());
console.log('Bubble:', sorter.sort(data));

sorter.setStrategy(new QuickSortStrategy());
console.log('Quick:', sorter.sort(data));

sorter.setStrategy(new MergeSortStrategy());
console.log('Merge:', sorter.sort(data));
```

**Real-World Strategy: Compression:**

```javascript
class CompressionStrategy {
  compress(data) {
    throw new Error('compress() must be implemented');
  }
  
  decompress(data) {
    throw new Error('decompress() must be implemented');
  }
}

class ZipCompression extends CompressionStrategy {
  compress(data) {
    console.log('Compressing with ZIP...');
    // Simulated compression
    const compressed = {
      algorithm: 'ZIP',
      originalSize: data.length,
      compressedSize: Math.floor(data.length * 0.6),
      data: `ZIP:${data}`
    };
    console.log(`Compressed: ${compressed.originalSize} → ${compressed.compressedSize} bytes`);
    return compressed;
  }
  
  decompress(compressed) {
    console.log('Decompressing ZIP...');
    return compressed.data.replace('ZIP:', '');
  }
}

class GzipCompression extends CompressionStrategy {
  compress(data) {
    console.log('Compressing with GZIP...');
    const compressed = {
      algorithm: 'GZIP',
      originalSize: data.length,
      compressedSize: Math.floor(data.length * 0.5),
      data: `GZIP:${data}`
    };
    console.log(`Compressed: ${compressed.originalSize} → ${compressed.compressedSize} bytes`);
    return compressed;
  }
  
  decompress(compressed) {
    console.log('Decompressing GZIP...');
    return compressed.data.replace('GZIP:', '');
  }
}

class Brotli Compression extends CompressionStrategy {
  compress(data) {
    console.log('Compressing with Brotli...');
    const compressed = {
      algorithm: 'Brotli',
      originalSize: data.length,
      compressedSize: Math.floor(data.length * 0.4),
      data: `BROTLI:${data}`
    };
    console.log(`Compressed: ${compressed.originalSize} → ${compressed.compressedSize} bytes`);
    return compressed;
  }
  
  decompress(compressed) {
    console.log('Decompressing Brotli...');
    return compressed.data.replace('BROTLI:', '');
  }
}

class FileCompressor {
  constructor(strategy) {
    this.strategy = strategy;
  }
  
  setStrategy(strategy) {
    this.strategy = strategy;
  }
  
  compressFile(file) {
    console.log(`\nCompressing file: ${file.name}`);
    const compressed = this.strategy.compress(file.data);
    return {
      name: `${file.name}.${compressed.algorithm.toLowerCase()}`,
      ...compressed
    };
  }
  
  decompressFile(compressedFile) {
    console.log(`\nDecompressing file: ${compressedFile.name}`);
    const data = this.strategy.decompress(compressedFile);
    return {
      name: compressedFile.name.split('.').slice(0, -1).join('.'),
      data
    };
  }
}

// Usage
const file = {
  name: 'document.txt',
  data: 'This is some sample data that needs to be compressed'
};

const compressor = new FileCompressor(new ZipCompression());
let compressed = compressor.compressFile(file);

// Switch to GZIP
compressor.setStrategy(new GzipCompression());
compressed = compressor.compressFile(file);

// Switch to Brotli (best compression)
compressor.setStrategy(new BrotliCompression());
compressed = compressor.compressFile(file);

// Decompress
const decompressed = compressor.decompressFile(compressed);
console.log('\nDecompressed:', decompressed);
```

---

### **7. Decorator Pattern**

Adds new functionality to objects dynamically without modifying their structure. Wraps the original object to add new behavior.

```javascript
// Base component
class Coffee {
  cost() {
    return 5;
  }
  
  description() {
    return 'Simple Coffee';
  }
}

// Decorator base
class CoffeeDecorator {
  constructor(coffee) {
    this.coffee = coffee;
  }
  
  cost() {
    return this.coffee.cost();
  }
  
  description() {
    return this.coffee.description();
  }
}

// Concrete decorators
class MilkDecorator extends CoffeeDecorator {
  cost() {
    return this.coffee.cost() + 1;
  }
  
  description() {
    return `${this.coffee.description()} + Milk`;
  }
}

class SugarDecorator extends CoffeeDecorator {
  cost() {
    return this.coffee.cost() + 0.5;
  }
  
  description() {
    return `${this.coffee.description()} + Sugar`;
  }
}

class WhipCreamDecorator extends CoffeeDecorator {
  cost() {
    return this.coffee.cost() + 1.5;
  }
  
  description() {
    return `${this.coffee.description()} + Whip Cream`;
  }
}

class VanillaDecorator extends CoffeeDecorator {
  cost() {
    return this.coffee.cost() + 2;
  }
  
  description() {
    return `${this.coffee.description()} + Vanilla`;
  }
}

// Usage
let myCoffee = new Coffee();
console.log(`${myCoffee.description()} - $${myCoffee.cost()}`);
// Simple Coffee - $5

myCoffee = new MilkDecorator(myCoffee);
console.log(`${myCoffee.description()} - $${myCoffee.cost()}`);
// Simple Coffee + Milk - $6

myCoffee = new SugarDecorator(myCoffee);
console.log(`${myCoffee.description()} - $${myCoffee.cost()}`);
// Simple Coffee + Milk + Sugar - $6.5

myCoffee = new WhipCreamDecorator(myCoffee);
console.log(`${myCoffee.description()} - $${myCoffee.cost()}`);
// Simple Coffee + Milk + Sugar + Whip Cream - $8

// Create fancy coffee in one go
let fancyCoffee = new Coffee();
fancyCoffee = new MilkDecorator(fancyCoffee);
fancyCoffee = new VanillaDecorator(fancyCoffee);
fancyCoffee = new WhipCreamDecorator(fancyCoffee);
console.log(`${fancyCoffee.description()} - $${fancyCoffee.cost()}`);
// Simple Coffee + Milk + Vanilla + Whip Cream - $9.5
```

**Advanced Decorator: Logger Decorator:**

```javascript
// Function decorator
function logExecutionTime(target, propertyKey, descriptor) {
  const originalMethod = descriptor.value;
  
  descriptor.value = async function(...args) {
    console.log(`[${propertyKey}] Starting...`);
    const start = performance.now();
    
    try {
      const result = await originalMethod.apply(this, args);
      const end = performance.now();
      console.log(`[${propertyKey}] Completed in ${(end - start).toFixed(2)}ms`);
      return result;
    } catch (error) {
      const end = performance.now();
      console.log(`[${propertyKey}] Failed in ${(end - start).toFixed(2)}ms`);
      throw error;
    }
  };
  
  return descriptor;
}

function validateArgs(target, propertyKey, descriptor) {
  const originalMethod = descriptor.value;
  
  descriptor.value = function(...args) {
    console.log(`[${propertyKey}] Validating arguments:`, args);
    
    if (args.some(arg => arg === null || arg === undefined)) {
      throw new Error('Invalid arguments: null or undefined not allowed');
    }
    
    return originalMethod.apply(this, args);
  };
  
  return descriptor;
}

function memoize(target, propertyKey, descriptor) {
  const originalMethod = descriptor.value;
  const cache = new Map();
  
  descriptor.value = function(...args) {
    const key = JSON.stringify(args);
    
    if (cache.has(key)) {
      console.log(`[${propertyKey}] Cache hit for:`, args);
      return cache.get(key);
    }
    
    console.log(`[${propertyKey}] Cache miss, computing...`);
    const result = originalMethod.apply(this, args);
    cache.set(key, result);
    
    return result;
  };
  
  return descriptor;
}

// Without decorator support, use wrapper functions
function createLoggingDecorator(fn) {
  return function(...args) {
    console.log(`Calling ${fn.name} with:`, args);
    const result = fn.apply(this, args);
    console.log(`${fn.name} returned:`, result);
    return result;
  };
}

function createCachingDecorator(fn) {
  const cache = new Map();
  
  return function(...args) {
    const key = JSON.stringify(args);
    
    if (cache.has(key)) {
      console.log('Cache hit');
      return cache.get(key);
    }
    
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}

// Usage
class Calculator {
  @validateArgs
  @memoize
  @logExecutionTime
  fibonacci(n) {
    if (n <= 1) return n;
    return this.fibonacci(n - 1) + this.fibonacci(n - 2);
  }
  
  @logExecutionTime
  async fetchData(url) {
    // Simulated API call
    await new Promise(resolve => setTimeout(resolve, 1000));
    return { data: `Data from ${url}` };
  }
}

// Without decorator support
function fibonacci(n) {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
}

const decoratedFib = createCachingDecorator(
  createLoggingDecorator(fibonacci)
);

console.log(decoratedFib(5));
console.log(decoratedFib(5)); // Cached
```

**Real-World Decorator: API Client:**

```javascript
class APIClient {
  async request(endpoint, options = {}) {
    const response = await fetch(endpoint, options);
    return response.json();
  }
}

// Decorator: Add authentication
class AuthDecorator {
  constructor(client, token) {
    this.client = client;
    this.token = token;
  }
  
  async request(endpoint, options = {}) {
    console.log('Adding authentication header');
    
    options.headers = {
      ...options.headers,
      'Authorization': `Bearer ${this.token}`
    };
    
    return this.client.request(endpoint, options);
  }
}

// Decorator: Add logging
class LoggingDecorator {
  constructor(client) {
    this.client = client;
  }
  
  async request(endpoint, options = {}) {
    console.log(`[API] ${options.method || 'GET'} ${endpoint}`);
    const start = Date.now();
    
    try {
      const result = await this.client.request(endpoint, options);
      const duration = Date.now() - start;
      console.log(`[API] Success (${duration}ms)`);
      return result;
    } catch (error) {
      const duration = Date.now() - start;
      console.log(`[API] Failed (${duration}ms):`, error.message);
      throw error;
    }
  }
}

// Decorator: Add retry logic
class RetryDecorator {
  constructor(client, maxRetries = 3, delay = 1000) {
    this.client = client;
    this.maxRetries = maxRetries;
    this.delay = delay;
  }
  
  async request(endpoint, options = {}) {
    let lastError;
    
    for (let attempt = 1; attempt <= this.maxRetries; attempt++) {
      try {
        return await this.client.request(endpoint, options);
      } catch (error) {
        lastError = error;
        console.log(`Attempt ${attempt} failed, retrying...`);
        
        if (attempt < this.maxRetries) {
          await new Promise(resolve => setTimeout(resolve, this.delay));
        }
      }
    }
    
    throw lastError;
  }
}

// Decorator: Add caching
class CachingDecorator {
  constructor(client, ttl = 60000) {
    this.client = client;
    this.cache = new Map();
    this.ttl = ttl;
  }
  
  async request(endpoint, options = {}) {
    // Only cache GET requests
    if (options.method && options.method !== 'GET') {
      return this.client.request(endpoint, options);
    }
    
    const cacheKey = `${endpoint}:${JSON.stringify(options)}`;
    const cached = this.cache.get(cacheKey);
    
    if (cached && Date.now() - cached.timestamp < this.ttl) {
      console.log('Cache hit');
      return cached.data;
    }
    
    console.log('Cache miss');
    const data = await this.client.request(endpoint, options);
    
    this.cache.set(cacheKey, {
      data,
      timestamp: Date.now()
    });
    
    return data;
  }
}

// Decorator: Rate limiting
class RateLimitDecorator {
  constructor(client, maxRequests = 10, timeWindow = 60000) {
    this.client = client;
    this.maxRequests = maxRequests;
    this.timeWindow = timeWindow;
    this.requests = [];
  }
  
  async request(endpoint, options = {}) {
    const now = Date.now();
    
    // Remove old requests outside time window
    this.requests = this.requests.filter(time => now - time < this.timeWindow);
    
    if (this.requests.length >= this.maxRequests) {
      const oldestRequest = this.requests[0];
      const waitTime = this.timeWindow - (now - oldestRequest);
      console.log(`Rate limit exceeded, waiting ${waitTime}ms`);
      await new Promise(resolve => setTimeout(resolve, waitTime));
    }
    
    this.requests.push(now);
    return this.client.request(endpoint, options);
  }
}

// Usage: Stack decorators
let client = new APIClient();

// Add authentication
client = new AuthDecorator(client, 'my-secret-token');

// Add logging
client = new LoggingDecorator(client);

// Add retry logic
client = new RetryDecorator(client, 3, 1000);

// Add caching
client = new CachingDecorator(client, 60000);

// Add rate limiting
client = new RateLimitDecorator(client, 10, 60000);

// Now client has all features
// await client.request('/api/users');
// await client.request('/api/posts');
```

---

### **8. Proxy Pattern**

Provides a surrogate or placeholder for another object to control access to it.

```javascript
// Virtual Proxy - Lazy loading
class RealImage {
  constructor(filename) {
    this.filename = filename;
    this.loadFromDisk();
  }
  
  loadFromDisk() {
    console.log(`Loading image from disk: ${this.filename}`);
    // Simulate expensive operation
  }
  
  display() {
    console.log(`Displaying image: ${this.filename}`);
  }
}

class ImageProxy {
  constructor(filename) {
    this.filename = filename;
    this.realImage = null;
  }
  
  display() {
    // Lazy load: only create real image when needed
    if (!this.realImage) {
      this.realImage = new RealImage(this.filename);
    }
    this.realImage.display();
  }
}

// Usage
console.log('Creating proxy images...');
const image1 = new ImageProxy('photo1.jpg');
const image2 = new ImageProxy('photo2.jpg');
const image3 = new ImageProxy('photo3.jpg');

console.log('\nImages created, but not loaded yet\n');

console.log('Displaying first image:');
image1.display(); // Loads and displays

console.log('\nDisplaying first image again:');
image1.display(); // Just displays (already loaded)

console.log('\nDisplaying second image:');
image2.display(); // Loads and displays
```

**Protection Proxy - Access Control:**

```javascript
class BankAccount {
  constructor(owner, balance = 0) {
    this.owner = owner;
    this.balance = balance;
  }
  
  deposit(amount) {
    this.balance += amount;
    console.log(`Deposited $${amount}. New balance: $${this.balance}`);
  }
  
  withdraw(amount) {
    if (amount > this.balance) {
      console.log('Insufficient funds');
      return false;
    }
    
    this.balance -= amount;
    console.log(`Withdrew $${amount}. New balance: $${this.balance}`);
    return true;
  }
  
  getBalance() {
    return this.balance;
  }
}

class BankAccountProxy {
  constructor(account, currentUser) {
    this.account = account;
    this.currentUser = currentUser;
  }
  
  checkAccess(operation) {
    if (this.currentUser !== this.account.owner) {
      console.log(`Access denied: ${this.currentUser} cannot ${operation} ${this.account.owner}'s account`);
      return false;
    }
    return true;
  }
  
  deposit(amount) {
    if (!this.checkAccess('deposit to')) return false;
    return this.account.deposit(amount);
  }
  
  withdraw(amount) {
    if (!this.checkAccess('withdraw from')) return false;
    return this.account.withdraw(amount);
  }
  
  getBalance() {
    if (!this.checkAccess('view balance of')) return null;
    return this.account.getBalance();
  }
}

// Usage
const johnAccount = new BankAccount('John', 1000);
const johnProxy = new BankAccountProxy(johnAccount, 'John');
const hackerProxy = new BankAccountProxy(johnAccount, 'Hacker');

// John can access his account
johnProxy.deposit(500);    // ✓ Works
johnProxy.withdraw(200);   // ✓ Works
console.log('Balance:', johnProxy.getBalance()); // ✓ Works

// Hacker cannot access John's account
hackerProxy.withdraw(100); // ✗ Denied
hackerProxy.getBalance();  // ✗ Denied
```

**Logging Proxy - Track Access:**

```javascript
class DataStore {
  constructor() {
    this.data = new Map();
  }
  
  set(key, value) {
    this.data.set(key, value);
  }
  
  get(key) {
    return this.data.get(key);
  }
  
  delete(key) {
    return this.data.delete(key);
  }
  
  has(key) {
    return this.data.has(key);
  }
}

class DataStoreProxy {
  constructor(dataStore) {
    this.dataStore = dataStore;
    this.accessLog = [];
  }
  
  log(operation, key, value = null) {
    const logEntry = {
      timestamp: new Date(),
      operation,
      key,
      value
    };
    
    this.accessLog.push(logEntry);
    console.log(`[LOG] ${operation} - key: ${key}${value !== null ? `, value: ${value}` : ''}`);
  }
  
  set(key, value) {
    this.log('SET', key, value);
    return this.dataStore.set(key, value);
  }
  
  get(key) {
    this.log('GET', key);
    return this.dataStore.get(key);
  }
  
  delete(key) {
    this.log('DELETE', key);
    return this.dataStore.delete(key);
  }
  
  has(key) {
    this.log('HAS', key);
    return this.dataStore.has(key);
  }
  
  getAccessLog() {
    return [...this.accessLog];
  }
}

// Usage
const store = new DataStore();
const proxy = new DataStoreProxy(store);

proxy.set('user1', { name: 'John', age: 30 });
proxy.set('user2', { name: 'Jane', age: 25 });

console.log(proxy.get('user1'));
console.log(proxy.has('user2'));
proxy.delete('user1');

console.log('\nAccess Log:');
console.log(proxy.getAccessLog());
```

**Smart Proxy with ES6 Proxy:**

```javascript
// Modern JavaScript Proxy
const user = {
  name: 'John',
  age: 30,
  email: 'john@example.com'
};

// Validation proxy
const validationHandler = {
  set(target, property, value) {
    console.log(`Setting ${property} = ${value}`);
    
    // Validate age
    if (property === 'age') {
      if (typeof value !== 'number' || value < 0 || value > 150) {
        throw new Error('Invalid age');
      }
    }
    
    // Validate email
    if (property === 'email') {
      if (!value.includes('@')) {
        throw new Error('Invalid email');
      }
    }
    
    target[property] = value;
    return true;
  },
  
  get(target, property) {
    console.log(`Getting ${property}`);
    
    // Hide sensitive properties
    if (property === 'password') {
      return '***';
    }
    
    return target[property];
  }
};

const userProxy = new Proxy(user, validationHandler);

// Valid operations
userProxy.name = 'Jane'; // ✓ Works
userProxy.age = 25;      // ✓ Works

// Invalid operations
try {
  userProxy.age = -5; // ✗ Throws error
} catch (error) {
  console.error(error.message);
}

try {
  userProxy.email = 'invalid'; // ✗ Throws error
} catch (error) {
  console.error(error.message);
}

console.log(userProxy.name); // Gets value
```

**Caching Proxy:**

```javascript
class ExpensiveCalculator {
  calculate(n) {
    console.log(`Performing expensive calculation for ${n}...`);
    
    // Simulate expensive operation
    let result = 0;
    for (let i = 0; i <= n; i++) {
      result += i;
    }
    
    return result;
  }
}

const calculatorHandler = {
  cache: new Map(),
  
  get(target, property) {
    if (property === 'calculate') {
      return (n) => {
        // Check cache
        if (this.cache.has(n)) {
          console.log(`Cache hit for ${n}`);
          return this.cache.get(n);
        }
        
        // Calculate and cache
        console.log(`Cache miss for ${n}`);
        const result = target[property](n);
        this.cache.set(n, result);
        
        return result;
      };
    }
    
    return target[property];
  }
};

const calculator = new ExpensiveCalculator();
const cachedCalculator = new Proxy(calculator, calculatorHandler);

console.log(cachedCalculator.calculate(1000)); // Calculates
console.log(cachedCalculator.calculate(1000)); // Returns cached
console.log(cachedCalculator.calculate(2000)); // Calculates
console.log(cachedCalculator.calculate(2000)); // Returns cached
```

---

### **9. Facade Pattern**

Provides a simplified interface to a complex subsystem, hiding its complexity from clients.

```javascript
// Complex subsystems
class CPU {
  freeze() {
    console.log('CPU: Freezing processor');
  }
  
  jump(position) {
    console.log(`CPU: Jumping to position ${position}`);
  }
  
  execute() {
    console.log('CPU: Executing instructions');
  }
}

class Memory {
  load(position, data) {
    console.log(`Memory: Loading data at position ${position}`);
  }
}

class HardDrive {
  read(lba, size) {
    console.log(`HardDrive: Reading ${size} bytes from LBA ${lba}`);
    return 'boot data';
  }
}

// Facade - Simple interface
class ComputerFacade {
  constructor() {
    this.cpu = new CPU();
    this.memory = new Memory();
    this.hardDrive = new HardDrive();
  }
  
  start() {
    console.log('Starting computer...\n');
    
    this.cpu.freeze();
    this.memory.load(0, this.hardDrive.read(0, 1024));
    this.cpu.jump(0);
    this.cpu.execute();
    
    console.log('\nComputer started successfully!');
  }
}

// Usage - Simple for client
const computer = new ComputerFacade();
computer.start();

// Without facade, client would need to:
// const cpu = new CPU();
// const memory = new Memory();
// const hardDrive = new HardDrive();
// cpu.freeze();
// memory.load(0, hardDrive.read(0, 1024));
// cpu.jump(0);
// cpu.execute();
```

**Advanced Facade: Home Theater System:**

```javascript
// Complex subsystems
class Amplifier {
  on() {
    console.log('Amplifier: Powering on');
  }
  
  off() {
    console.log('Amplifier: Powering off');
  }
  
  setVolume(level) {
    console.log(`Amplifier: Setting volume to ${level}`);
  }
  
  setSurroundSound() {
    console.log('Amplifier: Setting surround sound mode');
  }
}

class DVDPlayer {
  on() {
    console.log('DVD Player: Powering on');
  }
  
  off() {
    console.log('DVD Player: Powering off');
  }
  
  play(movie) {
    console.log(`DVD Player: Playing "${movie}"`);
  }
  
  stop() {
    console.log('DVD Player: Stopping playback');
  }
  
  eject() {
    console.log('DVD Player: Ejecting disc');
  }
}

class Projector {
  on() {
    console.log('Projector: Powering on');
  }
  
  off() {
    console.log('Projector: Powering off');
  }
  
  wideScreenMode() {
    console.log('Projector: Setting widescreen mode');
  }
}

class Lights {
  dim(level) {
    console.log(`Lights: Dimming to ${level}%`);
  }
  
  on() {
    console.log('Lights: Turning on');
  }
}

class Screen {
  down() {
    console.log('Screen: Lowering screen');
  }
  
  up() {
    console.log('Screen: Raising screen');
  }
}

// Facade - Simplified interface
class HomeTheaterFacade {
  constructor() {
    this.amplifier = new Amplifier();
    this.dvdPlayer = new DVDPlayer();
    this.projector = new Projector();
    this.lights = new Lights();
    this.screen = new Screen();
  }
  
  watchMovie(movie) {
    console.log(`\n=== Starting Movie: "${movie}" ===\n`);
    
    this.lights.dim(10);
    this.screen.down();
    this.projector.on();
    this.projector.wideScreenMode();
    this.amplifier.on();
    this.amplifier.setSurroundSound();
    this.amplifier.setVolume(5);
    this.dvdPlayer.on();
    this.dvdPlayer.play(movie);
    
    console.log('\n=== Enjoy your movie! ===\n');
  }
  
  endMovie() {
    console.log('\n=== Ending Movie ===\n');
    
    this.dvdPlayer.stop();
    this.dvdPlayer.eject();
    this.dvdPlayer.off();
    this.amplifier.off();
    this.projector.off();
    this.screen.up();
    this.lights.on();
    
    console.log('\n=== Movie ended ===\n');
  }
  
  listenToMusic(source) {
    console.log(`\n=== Playing Music from ${source} ===\n`);
    
    this.lights.dim(30);
    this.amplifier.on();
    this.amplifier.setVolume(3);
    
    console.log('\n=== Enjoy your music! ===\n');
  }
  
  endMusic() {
    console.log('\n=== Stopping Music ===\n');
    
    this.amplifier.off();
    this.lights.on();
    
    console.log('\n=== Music stopped ===\n');
  }
}

// Usage
const homeTheater = new HomeTheaterFacade();

// Watch a movie
homeTheater.watchMovie('The Matrix');

// Later...
homeTheater.endMovie();

// Listen to music
homeTheater.listenToMusic('Spotify');

// Later...
homeTheater.endMusic();
```

**Real-World Facade: API Client:**

```javascript
// Complex API operations
class AuthService {
  async login(credentials) {
    console.log('AuthService: Logging in...');
    // API call
    return { token: 'jwt-token-123', userId: 1 };
  }
  
  async logout(token) {
    console.log('AuthService: Logging out...');
    // API call
  }
  
  async refreshToken(token) {
    console.log('AuthService: Refreshing token...');
    return { token: 'new-jwt-token-456' };
  }
}

class UserService {
  async getProfile(userId, token) {
    console.log(`UserService: Getting profile for user ${userId}...`);
    return { id: userId, name: 'John Doe', email: 'john@example.com' };
  }
  
  async updateProfile(userId, data, token) {
    console.log(`UserService: Updating profile for user ${userId}...`);
    return { ...data, id: userId };
  }
}

class DataService {
  async fetchData(endpoint, token) {
    console.log(`DataService: Fetching from ${endpoint}...`);
    return { data: [] };
  }
  
  async postData(endpoint, data, token) {
    console.log(`DataService: Posting to ${endpoint}...`);
    return { success: true };
  }
}

class StorageService {
  saveToken(token) {
    console.log('StorageService: Saving token to localStorage');
    localStorage.setItem('token', token);
  }
  
  getToken() {
    console.log('StorageService: Retrieving token from localStorage');
    return localStorage.getItem('token');
  }
  
  clearToken() {
    console.log('StorageService: Clearing token from localStorage');
    localStorage.removeItem('token');
  }
}

// Facade - Simplified API
class APIFacade {
  constructor() {
    this.authService = new AuthService();
    this.userService = new UserService();
    this.dataService = new DataService();
    this.storageService = new StorageService();
    this.currentToken = null;
    this.currentUserId = null;
  }
  
  async login(email, password) {
    try {
      const { token, userId } = await this.authService.login({ email, password });
      
      this.currentToken = token;
      this.currentUserId = userId;
      this.storageService.saveToken(token);
      
      // Automatically fetch user profile
      const profile = await this.userService.getProfile(userId, token);
      
      console.log('✓ Login successful');
      return { success: true, profile };
    } catch (error) {
      console.error('✗ Login failed:', error);
      return { success: false, error };
    }
  }
  
  async logout() {
    try {
      await this.authService.logout(this.currentToken);
      
      this.storageService.clearToken();
      this.currentToken = null;
      this.currentUserId = null;
      
      console.log('✓ Logout successful');
      return { success: true };
    } catch (error) {
      console.error('✗ Logout failed:', error);
      return { success: false, error };
    }
  }
  
  async updateProfile(data) {
    try {
      // Auto-refresh token if needed
      await this.ensureValidToken();
      
      const updated = await this.userService.updateProfile(
        this.currentUserId,
        data,
        this.currentToken
      );
      
      console.log('✓ Profile updated');
      return { success: true, profile: updated };
    } catch (error) {
      console.error('✗ Profile update failed:', error);
      return { success: false, error };
    }
  }
  
  async fetchPosts() {
    try {
      await this.ensureValidToken();
      
      const posts = await this.dataService.fetchData('/posts', this.currentToken);
      
      return { success: true, posts };
    } catch (error) {
      console.error('✗ Fetch posts failed:', error);
      return { success: false, error };
    }
  }
  
  async createPost(postData) {
    try {
      await this.ensureValidToken();
      
      const result = await this.dataService.postData('/posts', postData, this.currentToken);
      
      console.log('✓ Post created');
      return { success: true, result };
    } catch (error) {
      console.error('✗ Create post failed:', error);
      return { success: false, error };
    }
  }
  
  async ensureValidToken() {
    // Check if token needs refresh
    // For simplicity, just log
    console.log('Checking token validity...');
  }
  
  isAuthenticated() {
    return this.currentToken !== null;
  }
}

// Usage - Much simpler for client
const api = new APIFacade();

// Login
await api.login('user@example.com', 'password');

// Update profile
await api.updateProfile({ name: 'Jane Doe', bio: 'Developer' });

// Fetch posts
await api.fetchPosts();

// Create post
await api.createPost({ title: 'Hello World', content: 'My first post' });

// Logout
await api.logout();
```

---

### **10. Command Pattern**

Encapsulates a request as an object, allowing you to parameterize clients with different requests, queue requests, and support undoable operations.

```javascript
// Receiver - the actual object that performs actions
class Light {
  constructor(location) {
    this.location = location;
    this.isOn = false;
  }
  
  on() {
    this.isOn = true;
    console.log(`${this.location} light is ON`);
  }
  
  off() {
    this.isOn = false;
    console.log(`${this.location} light is OFF`);
  }
}

// Command interface
class Command {
  execute() {
    throw new Error('execute() must be implemented');
  }
  
  undo() {
    throw new Error('undo() must be implemented');
  }
}

// Concrete commands
class LightOnCommand extends Command {
  constructor(light) {
    super();
    this.light = light;
  }
  
  execute() {
    this.light.on();
  }
  
  undo() {
    this.light.off();
  }
}

class LightOffCommand extends Command {
  constructor(light) {
    super();
    this.light = light;
  }
  
  execute() {
    this.light.off();
  }
  
  undo() {
    this.light.on();
  }
}

// Invoker - triggers commands
class RemoteControl {
  constructor() {
    this.history = [];
  }
  
  submit(command) {
    command.execute();
    this.history.push(command);
  }
  
  undo() {
    if (this.history.length === 0) {
      console.log('Nothing to undo');
      return;
    }
    
    const command = this.history.pop();
    command.undo();
  }
}

// Usage
const livingRoomLight = new Light('Living Room');
const kitchenLight = new Light('Kitchen');

const livingRoomOn = new LightOnCommand(livingRoomLight);
const livingRoomOff = new LightOffCommand(livingRoomLight);
const kitchenOn = new LightOnCommand(kitchenLight);
const kitchenOff = new LightOffCommand(kitchenLight);

const remote = new RemoteControl();

remote.submit(livingRoomOn);  // Living Room light is ON
remote.submit(kitchenOn);      // Kitchen light is ON
remote.submit(livingRoomOff);  // Living Room light is OFF

remote.undo();  // Living Room light is ON (undoes last command)
remote.undo();  // Kitchen light is OFF
remote.undo();  // Living Room light is OFF
```

**Advanced Command: Text Editor:**

```javascript
class TextEditor {
  constructor() {
    this.content = '';
  }
  
  getContent() {
    return this.content;
  }
  
  setContent(content) {
    this.content = content;
  }
  
  insert(text, position) {
    this.content = 
      this.content.slice(0, position) + 
      text + 
      this.content.slice(position);
  }
  
  delete(position, length) {
    return this.content.slice(position, position + length);
  }
}

class EditorCommand {
  execute() {}
  undo() {}
  redo() {}
}

class InsertCommand extends EditorCommand {
  constructor(editor, text, position) {
    super();
    this.editor = editor;
    this.text = text;
    this.position = position;
  }
  
  execute() {
    this.editor.insert(this.text, this.position);
    console.log(`Inserted "${this.text}" at position ${this.position}`);
  }
  
  undo() {
    const before = this.editor.getContent();
    this.editor.setContent(
      before.slice(0, this.position) + 
      before.slice(this.position + this.text.length)
    );
    console.log(`Undid insert of "${this.text}"`);
  }
  
  redo() {
    this.execute();
  }
}

class DeleteCommand extends EditorCommand {
  constructor(editor, position, length) {
    super();
    this.editor = editor;
    this.position = position;
    this.length = length;
    this.deletedText = '';
  }
  
  execute() {
    const content = this.editor.getContent();
    this.deletedText = content.slice(this.position, this.position + this.length);
    this.editor.setContent(
      content.slice(0, this.position) + 
      content.slice(this.position + this.length)
    );
    console.log(`Deleted "${this.deletedText}" at position ${this.position}`);
  }
  
  undo() {
    this.editor.insert(this.deletedText, this.position);
    console.log(`Undid delete of "${this.deletedText}"`);
  }
  
  redo() {
    this.execute();
  }
}

class ReplaceCommand extends EditorCommand {
  constructor(editor, oldText, newText) {
    super();
    this.editor = editor;
    this.oldText = oldText;
    this.newText = newText;
  }
  
  execute() {
    const content = this.editor.getContent();
    this.editor.setContent(content.replace(this.oldText, this.newText));
    console.log(`Replaced "${this.oldText}" with "${this.newText}"`);
  }
  
  undo() {
    const content = this.editor.getContent();
    this.editor.setContent(content.replace(this.newText, this.oldText));
    console.log(`Undid replace`);
  }
  
  redo() {
    this.execute();
  }
}

class CommandManager {
  constructor() {
    this.history = [];
    this.currentIndex = -1;
  }
  
  execute(command) {
    // Remove any commands after current index (when editing after undo)
    this.history = this.history.slice(0, this.currentIndex + 1);
    
    command.execute();
    this.history.push(command);
    this.currentIndex++;
  }
  
  undo() {
    if (this.currentIndex < 0) {
      console.log('Nothing to undo');
      return false;
    }
    
    this.history[this.currentIndex].undo();
    this.currentIndex--;
    return true;
  }
  
  redo() {
    if (this.currentIndex >= this.history.length - 1) {
      console.log('Nothing to redo');
      return false;
    }
    
    this.currentIndex++;
    this.history[this.currentIndex].redo();
    return true;
  }
  
  getHistory() {
    return this.history.map((cmd, index) => ({
      index,
      command: cmd.constructor.name,
      current: index === this.currentIndex
    }));
  }
}

// Usage
const editor = new TextEditor();
const manager = new CommandManager();

manager.execute(new InsertCommand(editor, 'Hello', 0));
console.log('Content:', editor.getContent()); // "Hello"

manager.execute(new InsertCommand(editor, ' World', 5));
console.log('Content:', editor.getContent()); // "Hello World"

manager.execute(new ReplaceCommand(editor, 'World', 'JavaScript'));
console.log('Content:', editor.getContent()); // "Hello JavaScript"

manager.undo(); // Undo replace
console.log('Content:', editor.getContent()); // "Hello World"

manager.undo(); // Undo insert " World"
console.log('Content:', editor.getContent()); // "Hello"

manager.redo(); // Redo insert " World"
console.log('Content:', editor.getContent()); // "Hello World"

console.log('\nHistory:', manager.getHistory());
```

**Real-World Command: Task Queue:**

```javascript
class Task {
  constructor(name, action) {
    this.name = name;
    this.action = action;
    this.status = 'pending'; // pending, running, completed, failed
    this.result = null;
    this.error = null;
  }
  
  async execute() {
    this.status = 'running';
    console.log(`[TASK] Running: ${this.name}`);
    
    try {
      this.result = await this.action();
      this.status = 'completed';
      console.log(`[TASK] Completed: ${this.name}`);
      return this.result;
    } catch (error) {
      this.status = 'failed';
      this.error = error;
      console.error(`[TASK] Failed: ${this.name}`, error.message);
      throw error;
    }
  }
  
  getInfo() {
    return {
      name: this.name,
      status: this.status,
      result: this.result,
      error: this.error?.message
    };
  }
}

class TaskQueue {
  constructor(concurrency = 1) {
    this.queue = [];
    this.running = [];
    this.completed = [];
    this.failed = [];
    this.concurrency = concurrency;
    this.isPaused = false;
  }
  
  add(task) {
    this.queue.push(task);
    console.log(`[QUEUE] Added task: ${task.name} (${this.queue.length} in queue)`);
    
    this.processNext();
    return this;
  }
  
  async processNext() {
    if (this.isPaused) return;
    if (this.running.length >= this.concurrency) return;
    if (this.queue.length === 0) return;
    
    const task = this.queue.shift();
    this.running.push(task);
    
    try {
      await task.execute();
      this.completed.push(task);
    } catch (error) {
      this.failed.push(task);
    } finally {
      const index = this.running.indexOf(task);
      if (index > -1) {
        this.running.splice(index, 1);
      }
      
      // Process next task
      this.processNext();
    }
  }
  
  pause() {
    this.isPaused = true;
    console.log('[QUEUE] Paused');
  }
  
  resume() {
    this.isPaused = false;
    console.log('[QUEUE] Resumed');
    
    // Process pending tasks
    while (this.running.length < this.concurrency && this.queue.length > 0) {
      this.processNext();
    }
  }
  
  clear() {
    this.queue = [];
    console.log('[QUEUE] Cleared pending tasks');
  }
  
  getStatus() {
    return {
      queued: this.queue.length,
      running: this.running.length,
      completed: this.completed.length,
      failed: this.failed.length,
      total: this.queue.length + this.running.length + this.completed.length + this.failed.length
    };
  }
  
  getResults() {
    return {
      completed: this.completed.map(t => t.getInfo()),
      failed: this.failed.map(t => t.getInfo())
    };
  }
}

// Usage
const queue = new TaskQueue(2); // Process 2 tasks concurrently

// Add tasks
queue.add(new Task('Download File 1', async () => {
  await new Promise(resolve => setTimeout(resolve, 1000));
  return 'file1.jpg downloaded';
}));

queue.add(new Task('Download File 2', async () => {
  await new Promise(resolve => setTimeout(resolve, 1500));
  return 'file2.jpg downloaded';
}));

queue.add(new Task('Process Image', async () => {
  await new Promise(resolve => setTimeout(resolve, 800));
  return 'image processed';
}));

queue.add(new Task('Upload to Server', async () => {
  await new Promise(resolve => setTimeout(resolve, 1200));
  return 'uploaded successfully';
}));

queue.add(new Task('Failing Task', async () => {
  await new Promise(resolve => setTimeout(resolve, 500));
  throw new Error('Simulated failure');
}));

// Wait a bit then check status
setTimeout(() => {
  console.log('\n[STATUS]', queue.getStatus());
}, 2000);

setTimeout(() => {
  console.log('\n[FINAL STATUS]', queue.getStatus());
  console.log('\n[RESULTS]', queue.getResults());
}, 5000);
```

---

### **11. Iterator Pattern**

Provides a way to access elements of a collection sequentially without exposing its underlying representation.

```javascript
// Custom Iterator
class ArrayIterator {
  constructor(array) {
    this.array = array;
    this.index = 0;
  }
  
  hasNext() {
    return this.index < this.array.length;
  }
  
  next() {
    if (!this.hasNext()) {
      throw new Error('No more elements');
    }
    
    return this.array[this.index++];
  }
  
  reset() {
    this.index = 0;
  }
  
  current() {
    return this.array[this.index];
  }
}

// Usage
const iterator = new ArrayIterator([1, 2, 3, 4, 5]);

while (iterator.hasNext()) {
  console.log(iterator.next());
}
// Output: 1, 2, 3, 4, 5

iterator.reset();
console.log(iterator.next()); // 1
```

**ES6 Iterator Protocol:**

```javascript
// Custom iterable object
class Range {
  constructor(start, end, step = 1) {
    this.start = start;
    this.end = end;
    this.step = step;
  }
  
  // Make it iterable by implementing [Symbol.iterator]
  [Symbol.iterator]() {
    let current = this.start;
    const end = this.end;
    const step = this.step;
    
    return {
      next() {
        if (current <= end) {
          const value = current;
          current += step;
          return { value, done: false };
        }
        return { done: true };
      }
    };
  }
}

// Usage with for...of
const range = new Range(1, 10, 2);

for (const num of range) {
  console.log(num); // 1, 3, 5, 7, 9
}

// Works with spread operator
console.log([...range]); // [1, 3, 5, 7, 9]

// Works with destructuring
const [first, second, ...rest] = range;
console.log(first, second, rest); // 1, 3, [5, 7, 9]
```

**Advanced Iterator: Tree Traversal:**

```javascript
class TreeNode {
  constructor(value) {
    this.value = value;
    this.children = [];
  }
  
  addChild(node) {
    this.children.push(node);
  }
}

class TreeIterator {
  constructor(root, strategy = 'breadth') {
    this.root = root;
    this.strategy = strategy;
    this.queue = [];
    this.stack = [];
    this.visited = new Set();
    this.initialize();
  }
  
  initialize() {
    if (this.strategy === 'breadth') {
      this.queue.push(this.root);
    } else if (this.strategy === 'depth') {
      this.stack.push(this.root);
    }
  }
  
  hasNext() {
    if (this.strategy === 'breadth') {
      return this.queue.length > 0;
    } else if (this.strategy === 'depth') {
      return this.stack.length > 0;
    }
  }
  
  next() {
    if (!this.hasNext()) {
      throw new Error('No more elements');
    }
    
    let node;
    
    if (this.strategy === 'breadth') {
      // Breadth-first (BFS)
      node = this.queue.shift();
      this.queue.push(...node.children);
    } else if (this.strategy === 'depth') {
      // Depth-first (DFS)
      node = this.stack.pop();
      this.stack.push(...node.children.reverse());
    }
    
    return node;
  }
  
  [Symbol.iterator]() {
    return {
      hasNext: () => this.hasNext(),
      next: () => {
        if (!this.hasNext()) {
          return { done: true };
        }
        return { value: this.next(), done: false };
      }
    };
  }
}

// Build tree
const root = new TreeNode('Root');
const child1 = new TreeNode('Child 1');
const child2 = new TreeNode('Child 2');
const child3 = new TreeNode('Child 3');
const grandchild1 = new TreeNode('Grandchild 1');
const grandchild2 = new TreeNode('Grandchild 2');

root.addChild(child1);
root.addChild(child2);
root.addChild(child3);
child1.addChild(grandchild1);
child1.addChild(grandchild2);

// Breadth-first traversal
console.log('Breadth-first:');
const bfsIterator = new TreeIterator(root, 'breadth');
while (bfsIterator.hasNext()) {
  const node = bfsIterator.next();
  console.log(node.value);
}
// Output: Root, Child 1, Child 2, Child 3, Grandchild 1, Grandchild 2

// Depth-first traversal
console.log('\nDepth-first:');
const dfsIterator = new TreeIterator(root, 'depth');
while (dfsIterator.hasNext()) {
  const node = dfsIterator.next();
  console.log(node.value);
}
// Output: Root, Child 1, Grandchild 1, Grandchild 2, Child 2, Child 3
```

**Generator-based Iterator:**

```javascript
class Playlist {
  constructor() {
    this.songs = [];
    this.currentIndex = 0;
  }
  
  addSong(song) {
    this.songs.push(song);
  }
  
  // Generator for sequential iteration
  *[Symbol.iterator]() {
    for (const song of this.songs) {
      yield song;
    }
  }
  
  // Generator for shuffle mode
  *shuffle() {
    const shuffled = [...this.songs].sort(() => Math.random() - 0.5);
    for (const song of shuffled) {
      yield song;
    }
  }
  
  // Generator for repeat mode
  *repeat(times = Infinity) {
    let count = 0;
    while (count < times) {
      for (const song of this.songs) {
        yield song;
      }
      count++;
    }
  }
  
  // Generator with filtering
  *filterByArtist(artist) {
    for (const song of this.songs) {
      if (song.artist === artist) {
        yield song;
      }
    }
  }
  
  // Async generator for streaming
  async *stream() {
    for (const song of this.songs) {
      // Simulate loading/buffering
      await new Promise(resolve => setTimeout(resolve, 100));
      console.log(`Streaming: ${song.title}`);
      yield song;
    }
  }
}

// Usage
const playlist = new Playlist();
playlist.addSong({ title: 'Song 1', artist: 'Artist A', duration: 180 });
playlist.addSong({ title: 'Song 2', artist: 'Artist B', duration: 200 });
playlist.addSong({ title: 'Song 3', artist: 'Artist A', duration: 220 });
playlist.addSong({ title: 'Song 4', artist: 'Artist C', duration: 190 });

// Normal iteration
console.log('Normal play:');
for (const song of playlist) {
  console.log(song.title);
}

// Shuffle
console.log('\nShuffle mode:');
for (const song of playlist.shuffle()) {
  console.log(song.title);
}

// Repeat 2 times
console.log('\nRepeat 2 times:');
for (const song of playlist.repeat(2)) {
  console.log(song.title);
}

// Filter by artist
console.log('\nArtist A songs:');
for (const song of playlist.filterByArtist('Artist A')) {
  console.log(song.title);
}

// Async streaming
console.log('\nStreaming:');
(async () => {
  for await (const song of playlist.stream()) {
    // Song is being streamed
  }
})();
```

**Real-World Iterator: Paginated API:**

```javascript
class PaginatedAPI {
  constructor(baseUrl, pageSize = 10) {
    this.baseUrl = baseUrl;
    this.pageSize = pageSize;
    this.currentPage = 1;
    this.totalPages = null;
    this.cache = new Map();
  }
  
  async fetchPage(page) {
    // Check cache
    if (this.cache.has(page)) {
      console.log(`Cache hit for page ${page}`);
      return this.cache.get(page);
    }
    
    console.log(`Fetching page ${page}...`);
    
    // Simulated API call
    const response = {
      page,
      pageSize: this.pageSize,
      totalPages: 5,
      totalItems: 47,
      data: Array.from({ length: this.pageSize }, (_, i) => ({
        id: (page - 1) * this.pageSize + i + 1,
        name: `Item ${(page - 1) * this.pageSize + i + 1}`
      }))
    };
    
    // Simulate network delay
    await new Promise(resolve => setTimeout(resolve, 500));
    
    this.totalPages = response.totalPages;
    this.cache.set(page, response.data);
    
    return response.data;
  }
  
  async *[Symbol.asyncIterator]() {
    while (this.totalPages === null || this.currentPage <= this.totalPages) {
      const data = await this.fetchPage(this.currentPage);
      
      for (const item of data) {
        yield item;
      }
      
      this.currentPage++;
      
      if (this.totalPages !== null && this.currentPage > this.totalPages) {
        break;
      }
    }
  }
  
  async *pages() {
    let page = 1;
    
    while (this.totalPages === null || page <= this.totalPages) {
      const data = await this.fetchPage(page);
      yield { page, data };
      
      page++;
      
      if (this.totalPages !== null && page > this.totalPages) {
        break;
      }
    }
  }
  
  reset() {
    this.currentPage = 1;
    this.totalPages = null;
  }
}

// Usage
const api = new PaginatedAPI('/api/items', 10);

// Iterate through all items
console.log('All items:');
(async () => {
  for await (const item of api) {
    console.log(item);
    
    // Early exit if needed
    if (item.id === 25) {
      console.log('Stopping at item 25');
      break;
    }
  }
})();

// Iterate by pages
console.log('\nBy pages:');
(async () => {
  for await (const { page, data } of api.pages()) {
    console.log(`Page ${page}:`, data.length, 'items');
  }
})();
```

---

### **12. Chain of Responsibility Pattern**

Passes requests along a chain of handlers. Each handler decides either to process the request or pass it to the next handler.

```javascript
// Base Handler
class Handler {
  constructor() {
    this.nextHandler = null;
  }
  
  setNext(handler) {
    this.nextHandler = handler;
    return handler; // Allow chaining
  }
  
  handle(request) {
    if (this.nextHandler) {
      return this.nextHandler.handle(request);
    }
    return null;
  }
}

// Concrete Handlers
class AuthenticationHandler extends Handler {
  handle(request) {
    console.log('AuthenticationHandler: Checking authentication...');
    
    if (!request.user) {
      console.log('❌ Authentication failed: No user');
      return { success: false, error: 'Not authenticated' };
    }
    
    console.log('✓ Authentication passed');
    return super.handle(request);
  }
}

class AuthorizationHandler extends Handler {
  handle(request) {
    console.log('AuthorizationHandler: Checking authorization...');
    
    if (!request.user.role || request.user.role !== 'admin') {
      console.log('❌ Authorization failed: Not admin');
      return { success: false, error: 'Not authorized' };
    }
    
    console.log('✓ Authorization passed');
    return super.handle(request);
  }
}

class ValidationHandler extends Handler {
  handle(request) {
    console.log('ValidationHandler: Validating request data...');
    
    if (!request.data || Object.keys(request.data).length === 0) {
      console.log('❌ Validation failed: No data');
      return { success: false, error: 'Invalid data' };
    }
    
    console.log('✓ Validation passed');
    return super.handle(request);
  }
}

class LoggingHandler extends Handler {
  handle(request) {
    console.log('LoggingHandler: Logging request...');
    console.log(`[LOG] ${request.method} ${request.path} by ${request.user?.name || 'anonymous'}`);
    
    return super.handle(request);
  }
}

class ProcessHandler extends Handler {
  handle(request) {
    console.log('ProcessHandler: Processing request...');
    console.log('✓ Request processed successfully');
    
    return {
      success: true,
      data: { message: 'Request processed', request }
    };
  }
}

// Usage
const authHandler = new AuthenticationHandler();
const authzHandler = new AuthorizationHandler();
const validationHandler = new ValidationHandler();
const loggingHandler = new LoggingHandler();
const processHandler = new ProcessHandler();

// Build chain
authHandler
  .setNext(authzHandler)
  .setNext(validationHandler)
  .setNext(loggingHandler)
  .setNext(processHandler);

// Test 1: Valid request
console.log('\n=== Test 1: Valid Request ===');
const request1 = {
  method: 'POST',
  path: '/api/users',
  user: { name: 'John', role: 'admin' },
  data: { name: 'New User' }
};
const result1 = authHandler.handle(request1);
console.log('Result:', result1);

// Test 2: No authentication
console.log('\n=== Test 2: No Authentication ===');
const request2 = {
  method: 'POST',
  path: '/api/users',
  data: { name: 'New User' }
};
const result2 = authHandler.handle(request2);
console.log('Result:', result2);

// Test 3: No authorization
console.log('\n=== Test 3: No Authorization ===');
const request3 = {
  method: 'POST',
  path: '/api/users',
  user: { name: 'Jane', role: 'user' },
  data: { name: 'New User' }
};
const result3 = authHandler.handle(request3);
console.log('Result:', result3);
```

**Advanced Chain: Support Ticket System:**

```javascript
class SupportHandler {
  constructor(name, level) {
    this.name = name;
    this.level = level;
    this.nextHandler = null;
  }
  
  setNext(handler) {
    this.nextHandler = handler;
    return handler;
  }
  
  handle(ticket) {
    if (ticket.priority <= this.level) {
      return this.process(ticket);
    }
    
    if (this.nextHandler) {
      console.log(`${this.name}: Escalating ticket #${ticket.id} to next level`);
      return this.nextHandler.handle(ticket);
    }
    
    console.log(`${this.name}: No one can handle ticket #${ticket.id}`);
    return { handled: false, message: 'No handler available' };
  }
  
  process(ticket) {
    console.log(`${this.name}: Handling ticket #${ticket.id} - ${ticket.issue}`);
    return {
      handled: true,
      handledBy: this.name,
      ticket
    };
  }
}

class Ticket {
  constructor(id, issue, priority) {
    this.id = id;
    this.issue = issue;
    this.priority = priority; // 1 = low, 2 = medium, 3 = high, 4 = critical
    this.createdAt = new Date();
  }
}

// Create support hierarchy
const juniorSupport = new SupportHandler('Junior Support', 1);
const seniorSupport = new SupportHandler('Senior Support', 2);
const teamLead = new SupportHandler('Team Lead', 3);
const manager = new SupportHandler('Manager', 4);

// Build chain
juniorSupport
  .setNext(seniorSupport)
  .setNext(teamLead)
  .setNext(manager);

// Create tickets
const tickets = [
  new Ticket(1, 'Password reset', 1),
  new Ticket(2, 'Account locked', 2),
  new Ticket(3, 'Data corruption', 3),
  new Ticket(4, 'Security breach', 4),
  new Ticket(5, 'General inquiry', 1)
];

// Process tickets
console.log('Processing Support Tickets:\n');
tickets.forEach(ticket => {
  console.log(`\nTicket #${ticket.id} (Priority: ${ticket.priority})`);
  const result = juniorSupport.handle(ticket);
  console.log('Result:', result.handled ? '✓ Handled' : '✗ Not handled');
});
```

**Real-World Chain: Middleware Pipeline:**

```javascript
class Middleware {
  constructor() {
    this.middlewares = [];
  }
  
  use(fn) {
    this.middlewares.push(fn);
    return this;
  }
  
  async execute(context) {
    let index = 0;
    
    const next = async () => {
      if (index >= this.middlewares.length) {
        return;
      }
      
      const middleware = this.middlewares[index++];
      await middleware(context, next);
    };
    
    await next();
  }
}

// Middleware functions
const authMiddleware = async (ctx, next) => {
  console.log('[Auth] Checking authentication...');
  
  if (!ctx.headers.authorization) {
    ctx.status = 401;
    ctx.body = { error: 'Unauthorized' };
    return; // Stop chain
  }
  
  ctx.user = { id: 1, name: 'John Doe' };
  console.log('[Auth] ✓ Authenticated');
  await next();
};

const loggerMiddleware = async (ctx, next) => {
  const start = Date.now();
  console.log(`[Logger] ${ctx.method} ${ctx.path}`);
  
  await next();
  
  const duration = Date.now() - start;
  console.log(`[Logger] Completed in ${duration}ms with status ${ctx.status}`);
};

const rateLimitMiddleware = async (ctx, next) => {
  console.log('[RateLimit] Checking rate limit...');
  
  // Simulated rate limit check
  const isRateLimited = false;
  
  if (isRateLimited) {
    ctx.status = 429;
    ctx.body = { error: 'Too many requests' };
    return;
  }
  
  console.log('[RateLimit] ✓ Within limits');
  await next();
};

const validationMiddleware = async (ctx, next) => {
  console.log('[Validation] Validating request...');
  
  if (ctx.method === 'POST' && !ctx.body) {
    ctx.status = 400;
    ctx.body = { error: 'Body required' };
    return;
  }
  
  console.log('[Validation] ✓ Valid');
  await next();
};

const corsMiddleware = async (ctx, next) => {
  console.log('[CORS] Setting CORS headers...');
  
  ctx.headers = {
    ...ctx.headers,
    'Access-Control-Allow-Origin': '*',
    'Access-Control-Allow-Methods': 'GET, POST, PUT, DELETE'
  };
  
  await next();
};

const handlerMiddleware = async (ctx, next) => {
  console.log('[Handler] Processing request...');
  
  ctx.status = 200;
  ctx.body = {
    message: 'Success',
    user: ctx.user,
    path: ctx.path
  };
  
  await next();
};

// Usage
const app = new Middleware();

app
  .use(loggerMiddleware)
  .use(corsMiddleware)
  .use(authMiddleware)
  .use(rateLimitMiddleware)
  .use(validationMiddleware)
  .use(handlerMiddleware);

// Test request 1: Valid
console.log('\n=== Request 1: Valid ===');
const context1 = {
  method: 'GET',
  path: '/api/users',
  headers: { authorization: 'Bearer token123' },
  status: null,
  body: null
};
await app.execute(context1);
console.log('Response:', context1.body);

// Test request 2: No auth
console.log('\n=== Request 2: No Auth ===');
const context2 = {
  method: 'GET',
  path: '/api/users',
  headers: {},
  status: null,
  body: null
};
await app.execute(context2);
console.log('Response:', context2.body);
```

---

### **13. Mediator Pattern**

Reduces coupling between components by making them communicate through a mediator object instead of directly with each other.

```javascript
// Mediator
class ChatRoom {
  constructor() {
    this.users = new Map();
  }
  
  register(user) {
    this.users.set(user.name, user);
    user.chatRoom = this;
    console.log(`${user.name} joined the chat room`);
  }
  
  sendMessage(message, from, to) {
    if (to) {
      // Private message
      const recipient = this.users.get(to);
      if (recipient) {
        recipient.receive(message, from);
      } else {
        console.log(`User ${to} not found`);
      }
    } else {
      // Broadcast to all
      this.users.forEach((user, name) => {
        if (name !== from) {
          user.receive(message, from);
        }
      });
    }
  }
  
  getUsers() {
    return Array.from(this.users.keys());
  }
}

// Colleague
class User {
  constructor(name) {
    this.name = name;
    this.chatRoom = null;
  }
  
  send(message, to = null) {
    console.log(`${this.name} sends: "${message}"${to ? ` to ${to}` : ' (broadcast)'}`);
    this.chatRoom.sendMessage(message, this.name, to);
  }
  
  receive(message, from) {
    console.log(`${this.name} received from ${from}: "${message}"`);
  }
}

// Usage
const chatRoom = new ChatRoom();

const john = new User('John');
const jane = new User('Jane');
const bob = new User('Bob');

chatRoom.register(john);
chatRoom.register(jane);
chatRoom.register(bob);

// Broadcast message
john.send('Hello everyone!');

// Private message
jane.send('Hi John!', 'John');

console.log('\nUsers in chat:', chatRoom.getUsers());
```

**Advanced Mediator: Air Traffic Control:**

```javascript
class AirTrafficControl {
  constructor() {
    this.aircrafts = new Map();
    this.runways = new Set(['Runway 1', 'Runway 2', 'Runway 3']);
    this.occupiedRunways = new Map();
  }
  
  registerAircraft(aircraft) {
    this.aircrafts.set(aircraft.id, aircraft);
    aircraft.setMediator(this);
    console.log(`✈️  ${aircraft.id} registered with ATC`);
  }
  
  requestLanding(aircraftId) {
    const aircraft = this.aircrafts.get(aircraftId);
    
    if (!aircraft) {
      console.log(`❌ Aircraft ${aircraftId} not registered`);
      return false;
    }
    
    // Find available runway
    for (const runway of this.runways) {
      if (!this.occupiedRunways.has(runway)) {
        this.occupiedRunways.set(runway, aircraftId);
        aircraft.grantLanding(runway);
        
        // Auto-release runway after landing
        setTimeout(() => {
          this.releaseRunway(runway, aircraftId);
        }, 3000);
        
        return true;
      }
    }
    
    console.log(`⏳ ${aircraftId} is holding - no runways available`);
    aircraft.hold();
    return false;
  }
  
  requestTakeoff(aircraftId) {
    const aircraft = this.aircrafts.get(aircraftId);
    
    if (!aircraft) {
      console.log(`❌ Aircraft ${aircraftId} not registered`);
      return false;
    }
    
    // Find available runway
    for (const runway of this.runways) {
      if (!this.occupiedRunways.has(runway)) {
        this.occupiedRunways.set(runway, aircraftId);
        aircraft.grantTakeoff(runway);
        
        // Auto-release runway after takeoff
        setTimeout(() => {
          this.releaseRunway(runway, aircraftId);
        }, 2000);
        
        return true;
      }
    }
    
    console.log(`⏳ ${aircraftId} is waiting for takeoff`);
    return false;
  }
  
  releaseRunway(runway, aircraftId) {
    if (this.occupiedRunways.get(runway) === aircraftId) {
      this.occupiedRunways.delete(runway);
      console.log(`🟢 ${runway} is now available`);
    }
  }
  
  getStatus() {
    return {
      totalAircrafts: this.aircrafts.size,
      totalRunways: this.runways.size,
      availableRunways: this.runways.size - this.occupiedRunways.size,
      occupiedRunways: Array.from(this.occupiedRunways.entries())
    };
  }
}

class Aircraft {
  constructor(id, type) {
    this.id = id;
    this.type = type;
    this.mediator = null;
    this.status = 'airborne';
  }
  
  setMediator(mediator) {
    this.mediator = mediator;
  }
  
  requestLanding() {
    console.log(`\n${this.id}: Requesting landing clearance`);
    this.mediator.requestLanding(this.id);
  }
  
  grantLanding(runway) {
    console.log(`✓ ${this.id}: Landing cleared on ${runway}`);
    this.status = 'landing';
    
    setTimeout(() => {
      this.status = 'landed';
      console.log(`✓ ${this.id}: Successfully landed on ${runway}`);
    }, 1000);
  }
  
  hold() {
    console.log(`⏳ ${this.id}: Holding pattern - awaiting clearance`);
    this.status = 'holding';
    
    // Retry after delay
    setTimeout(() => {
      if (this.status === 'holding') {
        this.requestLanding();
      }
    }, 2000);
  }
  
  requestTakeoff() {
    console.log(`\n${this.id}: Requesting takeoff clearance`);
    this.mediator.requestTakeoff(this.id);
  }
  
  grantTakeoff(runway) {
    console.log(`✓ ${this.id}: Takeoff cleared from ${runway}`);
    this.status = 'taking-off';
    
    setTimeout(() => {
      this.status = 'airborne';
      console.log(`✓ ${this.id}: Airborne`);
    }, 1000);
  }
}

// Usage
const atc = new AirTrafficControl();

const flight1 = new Aircraft('AA101', 'Boeing 737');
const flight2 = new Aircraft('UA202', 'Airbus A320');
const flight3 = new Aircraft('DL303', 'Boeing 777');
const flight4 = new Aircraft('SW404', 'Boeing 737');

atc.registerAircraft(flight1);
atc.registerAircraft(flight2);
atc.registerAircraft(flight3);
atc.registerAircraft(flight4);

// Simulate landing requests
flight1.requestLanding();
flight2.requestLanding();
flight3.requestLanding();
flight4.requestLanding(); // Will hold

setTimeout(() => {
  console.log('\n=== ATC Status ===');
  console.log(atc.getStatus());
}, 5000);
```

**Real-World Mediator: Form Validation:**

```javascript
class FormMediator {
  constructor() {
    this.fields = new Map();
    this.validators = new Map();
    this.errors = new Map();
  }
  
  registerField(name, field) {
    this.fields.set(name, field);
    field.setMediator(this);
  }
  
  registerValidator(name, validatorFn) {
    this.validators.set(name, validatorFn);
  }
  
  notify(fieldName, value) {
    // Validate field
    const validator = this.validators.get(fieldName);
    
    if (validator) {
      const error = validator(value, this.getAllValues());
      
      if (error) {
        this.errors.set(fieldName, error);
        this.fields.get(fieldName).showError(error);
      } else {
        this.errors.delete(fieldName);
        this.fields.get(fieldName).clearError();
      }
    }
    
    // Cross-field validation
    this.validateDependentFields(fieldName);
  }
  
  validateDependentFields(changedField) {
    // Re-validate related fields
    const dependencies = {
      password: ['confirmPassword'],
      email: ['confirmEmail']
    };
    
    const dependent = dependencies[changedField];
    if (dependent) {
      dependent.forEach(fieldName => {
        const field = this.fields.get(fieldName);
        if (field) {
          this.notify(fieldName, field.value);
        }
      });
    }
  }
  
  getAllValues() {
    const values = {};
    this.fields.forEach((field, name) => {
      values[name] = field.value;
    });
    return values;
  }
  
  isValid() {
    // Validate all fields
    this.fields.forEach((field, name) => {
      this.notify(name, field.value);
    });
    
    return this.errors.size === 0;
  }
  
  getErrors() {
    return Object.fromEntries(this.errors);
  }
}

class FormField {
  constructor(name, value = '') {
    this.name = name;
    this.value = value;
    this.mediator = null;
    this.error = null;
  }
  
  setMediator(mediator) {
    this.mediator = mediator;
  }
  
  setValue(value) {
    this.value = value;
    if (this.mediator) {
      this.mediator.notify(this.name, value);
    }
  }
  
  showError(error) {
    this.error = error;
    console.log(`❌ ${this.name}: ${error}`);
  }
  
  clearError() {
    this.error = null;
    console.log(`✓ ${this.name}: Valid`);
  }
}

// Usage
const formMediator = new FormMediator();

// Create fields
const emailField = new FormField('email');
const passwordField = new FormField('password');
const confirmPasswordField = new FormField('confirmPassword');
const usernameField = new FormField('username');

// Register fields
formMediator.registerField('email', emailField);
formMediator.registerField('password', passwordField);
formMediator.registerField('confirmPassword', confirmPasswordField);
formMediator.registerField('username', usernameField);

// Register validators
formMediator.registerValidator('email', (value) => {
  if (!value) return 'Email is required';
  if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value)) return 'Invalid email format';
  return null;
});

formMediator.registerValidator('password', (value) => {
  if (!value) return 'Password is required';
  if (value.length < 8) return 'Password must be at least 8 characters';
  return null;
});

formMediator.registerValidator('confirmPassword', (value, allValues) => {
  if (!value) return 'Please confirm password';
  if (value !== allValues.password) return 'Passwords do not match';
  return null;
});

formMediator.registerValidator('username', (value) => {
  if (!value) return 'Username is required';
  if (value.length < 3) return 'Username must be at least 3 characters';
  if (!/^[a-zA-Z0-9_]+$/.test(value)) return 'Username can only contain letters, numbers, and underscores';
  return null;
});

// Simulate user input
console.log('\n=== Form Validation ===\n');

emailField.setValue('invalid-email'); // Invalid
emailField.setValue('user@example.com'); // Valid

passwordField.setValue('123'); // Too short
passwordField.setValue('password123'); // Valid

confirmPasswordField.setValue('password'); // Doesn't match
confirmPasswordField.setValue('password123'); // Matches

usernameField.setValue('ab'); // Too short
usernameField.setValue('john_doe'); // Valid

// Validate entire form
console.log('\n=== Final Validation ===');
const isValid = formMediator.isValid();
console.log('Form valid:', isValid);

if (!isValid) {
  console.log('Errors:', formMediator.getErrors());
}
```

---

### **14. Adapter Pattern**

Converts the interface of a class into another interface that clients expect. Allows incompatible interfaces to work together.

```javascript
// Old API (incompatible interface)
class OldPaymentAPI {
  processPayment(cardNumber, amount) {
    console.log(`Old API: Processing $${amount} with card ${cardNumber}`);
    return {
      statusCode: 200,
      message: 'Success',
      transactionId: `OLD-${Date.now()}`
    };
  }
}

// New API (different interface)
class NewPaymentAPI {
  charge(paymentInfo) {
    console.log(`New API: Charging $${paymentInfo.amount}`);
    return {
      success: true,
      id: `NEW-${Date.now()}`,
      details: paymentInfo
    };
  }
}

// Adapter - makes NewPaymentAPI compatible with OldPaymentAPI interface
class PaymentAdapter {
  constructor(newApi) {
    this.newApi = newApi;
  }
  
  processPayment(cardNumber, amount) {
    // Convert old interface to new interface
    const paymentInfo = {
      card: cardNumber,
      amount: amount,
      currency: 'USD',
      timestamp: new Date().toISOString()
    };
    
    const result = this.newApi.charge(paymentInfo);
    
    // Convert new response to old response format
    return {
      statusCode: result.success ? 200 : 400,
      message: result.success ? 'Success' : 'Failed',
      transactionId: result.id
    };
  }
}

// Usage
const oldApi = new OldPaymentAPI();
const newApi = new NewPaymentAPI();
const adapter = new PaymentAdapter(newApi);

// Client code expects old interface
function processTransaction(api, cardNumber, amount) {
  const result = api.processPayment(cardNumber, amount);
  console.log('Transaction result:', result);
}

// Works with old API
processTransaction(oldApi, '1234-5678-9012-3456', 100);

// Also works with new API through adapter
processTransaction(adapter, '1234-5678-9012-3456', 100);
```

**Advanced Adapter: Data Format Converter:**

```javascript
// XML Data Source
class XMLDataSource {
  getData() {
    return `
      <users>
        <user>
          <id>1</id>
          <name>John Doe</name>
          <email>john@example.com</email>
        </user>
        <user>
          <id>2</id>
          <name>Jane Smith</name>
          <email>jane@example.com</email>
        </user>
      </users>
    `;
  }
}

// CSV Data Source
class CSVDataSource {
  getData() {
    return `id,name,email
1,John Doe,john@example.com
2,Jane Smith,jane@example.com`;
  }
}

// Application expects JSON format
class DataConsumer {
  processData(jsonData) {
    const users = JSON.parse(jsonData);
    console.log('Processing users:');
    users.forEach(user => {
      console.log(`- ${user.name} (${user.email})`);
    });
  }
}

// Adapter for XML
class XMLToJSONAdapter {
  constructor(xmlSource) {
    this.xmlSource = xmlSource;
  }
  
  getData() {
    const xmlString = this.xmlSource.getData();
    
    // Simple XML parsing (in production, use DOMParser or library)
    const users = [];
    const userMatches = xmlString.matchAll(/<user>[\s\S]*?<\/user>/g);
    
    for (const match of userMatches) {
      const userXml = match[0];
      const id = userXml.match(/<id>(\d+)<\/id>/)[1];
      const name = userXml.match(/<name>(.+?)<\/name>/)[1];
      const email = userXml.match(/<email>(.+?)<\/email>/)[1];
      
      users.push({
        id: parseInt(id),
        name,
        email
      });
    }
    
    return JSON.stringify(users);
  }
}

// Adapter for CSV
class CSVToJSONAdapter {
  constructor(csvSource) {
    this.csvSource = csvSource;
  }
  
  getData() {
    const csvString = this.csvSource.getData();
    const lines = csvString.trim().split('\n');
    const headers = lines[0].split(',');
    
    const users = lines.slice(1).map(line => {
      const values = line.split(',');
      const user = {};
      
      headers.forEach((header, index) => {
        user[header] = header === 'id' ? parseInt(values[index]) : values[index];
      });
      
      return user;
    });
    
    return JSON.stringify(users);
  }
}

// Usage
const consumer = new DataConsumer();

// XML data source
const xmlSource = new XMLDataSource();
const xmlAdapter = new XMLToJSONAdapter(xmlSource);
console.log('XML Data:');
consumer.processData(xmlAdapter.getData());

// CSV data source
console.log('\nCSV Data:');
const csvSource = new CSVDataSource();
const csvAdapter = new CSVToJSONAdapter(csvSource);
consumer.processData(csvAdapter.getData());
```

**Real-World Adapter: Third-Party API Integration:**

```javascript
// Third-party weather API (incompatible interface)
class WeatherAPIv1 {
  async fetchWeather(city) {
    console.log(`WeatherAPIv1: Fetching for ${city}`);
    
    // Simulated API response
    return {
      location: city,
      temp_celsius: 25,
      conditions: 'sunny',
      wind_speed_kmh: 15,
      humidity_percent: 60
    };
  }
}

class WeatherAPIv2 {
  async getTemperature(location) {
    console.log(`WeatherAPIv2: Fetching for ${location}`);
    
    // Different response format
    return {
      place: location,
      temperature: { value: 77, unit: 'F' },
      weather: 'clear',
      wind: { speed: 9.3, unit: 'mph' },
      humidity: 0.6
    };
  }
}

// Application interface
class WeatherService {
  constructor(adapter) {
    this.adapter = adapter;
  }
  
  async displayWeather(city) {
    const weather = await this.adapter.getWeather(city);
    
    console.log('\n=== Weather Report ===');
    console.log(`Location: ${weather.location}`);
    console.log(`Temperature: ${weather.temperature}°C`);
    console.log(`Conditions: ${weather.conditions}`);
    console.log(`Wind Speed: ${weather.windSpeed} km/h`);
    console.log(`Humidity: ${weather.humidity}%`);
  }
}

// Adapter for APIv1
class WeatherAPIv1Adapter {
  constructor(api) {
    this.api = api;
  }
  
  async getWeather(city) {
    const data = await this.api.fetchWeather(city);
    
    // Normalize to standard format
    return {
      location: data.location,
      temperature: data.temp_celsius,
      conditions: data.conditions,
      windSpeed: data.wind_speed_kmh,
      humidity: data.humidity_percent
    };
  }
}

// Adapter for APIv2
class WeatherAPIv2Adapter {
  constructor(api) {
    this.api = api;
  }
  
  async getWeather(city) {
    const data = await this.api.getTemperature(city);
    
    // Convert Fahrenheit to Celsius
    const tempCelsius = ((data.temperature.value - 32) * 5) / 9;
    
    // Convert mph to km/h
    const windKmh = data.wind.speed * 1.60934;
    
    // Normalize to standard format
    return {
      location: data.place,
      temperature: Math.round(tempCelsius),
      conditions: data.weather,
      windSpeed: Math.round(windKmh),
      humidity: Math.round(data.humidity * 100)
    };
  }
}

// Usage
(async () => {
  // Using APIv1
  const apiv1 = new WeatherAPIv1();
  const adapterv1 = new WeatherAPIv1Adapter(apiv1);
  const servicev1 = new WeatherService(adapterv1);
  await servicev1.displayWeather('London');
  
  // Using APIv2 (different API, same interface through adapter)
  const apiv2 = new WeatherAPIv2();
  const adapterv2 = new WeatherAPIv2Adapter(apiv2);
  const servicev2 = new WeatherService(adapterv2);
  await servicev2.displayWeather('New York');
})();
```

---

### **15. Composite Pattern**

Composes objects into tree structures to represent part-whole hierarchies. Allows clients to treat individual objects and compositions uniformly.

```javascript
// Component interface
class FileSystemItem {
  constructor(name) {
    this.name = name;
  }
  
  getSize() {
    throw new Error('getSize() must be implemented');
  }
  
  print(indent = '') {
    throw new Error('print() must be implemented');
  }
}

// Leaf - File
class File extends FileSystemItem {
  constructor(name, size) {
    super(name);
    this.size = size;
  }
  
  getSize() {
    return this.size;
  }
  
  print(indent = '') {
    console.log(`${indent}📄 ${this.name} (${this.size} KB)`);
  }
}

// Composite - Directory
class Directory extends FileSystemItem {
  constructor(name) {
    super(name);
    this.children = [];
  }
  
  add(item) {
    this.children.push(item);
    return this;
  }
  
  remove(item) {
    const index = this.children.indexOf(item);
    if (index > -1) {
      this.children.splice(index, 1);
    }
    return this;
  }
  
  getSize() {
    return this.children.reduce((total, child) => total + child.getSize(), 0);
  }
  
  print(indent = '') {
    console.log(`${indent}📁 ${this.name} (${this.getSize()} KB)`);
    this.children.forEach(child => child.print(indent + '  '));
  }
  
  find(name) {
    if (this.name === name) return this;
    
    for (const child of this.children) {
      if (child.name === name) return child;
      if (child instanceof Directory) {
        const found = child.find(name);
        if (found) return found;
      }
    }
    
    return null;
  }
}

// Usage
const root = new Directory('root');

const documents = new Directory('documents');
documents.add(new File('resume.pdf', 150));
documents.add(new File('cover-letter.docx', 50));

const photos = new Directory('photos');
photos.add(new File('vacation.jpg', 2000));
photos.add(new File('family.jpg', 1500));

const work = new Directory('work');
work.add(new File('project.pptx', 500));
work.add(new File('budget.xlsx', 100));
documents.add(work);

root.add(documents);
root.add(photos);
root.add(new File('readme.txt', 5));

// Print entire structure
root.print();

// Get total size
console.log(`\nTotal size: ${root.getSize()} KB`);

// Find specific directory
const workDir = root.find('work');
if (workDir) {
  console.log(`\nWork directory size: ${workDir.getSize()} KB`);
}
```

**Advanced Composite: UI Component Tree:**

```javascript
// Component base
class UIComponent {
  constructor(name) {
    this.name = name;
    this.visible = true;
  }
  
  render() {
    throw new Error('render() must be implemented');
  }
  
  show() {
    this.visible = true;
  }
  
  hide() {
    this.visible = false;
  }
  
  onClick(handler) {
    this.clickHandler = handler;
  }
}

// Leaf components
class Button extends UIComponent {
  constructor(name, label) {
    super(name);
    this.label = label;
  }
  
  render(indent = '') {
    if (!this.visible) return;
    console.log(`${indent}<button>${this.label}</button>`);
  }
  
  click() {
    console.log(`Button "${this.label}" clicked`);
    if (this.clickHandler) {
      this.clickHandler();
    }
  }
}

class TextInput extends UIComponent {
  constructor(name, placeholder) {
    super(name);
    this.placeholder = placeholder;
    this.value = '';
  }
  
  render(indent = '') {
    if (!this.visible) return;
    console.log(`${indent}<input type="text" placeholder="${this.placeholder}" />`);
  }
  
  setValue(value) {
    this.value = value;
    console.log(`Input "${this.name}" value set to: ${value}`);
  }
}

class Label extends UIComponent {
  constructor(name, text) {
    super(name);
    this.text = text;
  }
  
  render(indent = '') {
    if (!this.visible) return;
    console.log(`${indent}<label>${this.text}</label>`);
  }
}

// Composite components
class Panel extends UIComponent {
  constructor(name) {
    super(name);
    this.children = [];
  }
  
  add(component) {
    this.children.push(component);
    return this;
  }
  
  remove(component) {
    const index = this.children.indexOf(component);
    if (index > -1) {
      this.children.splice(index, 1);
    }
    return this;
  }
  
  render(indent = '') {
    if (!this.visible) return;
    console.log(`${indent}<div class="panel">`);
    this.children.forEach(child => child.render(indent + '  '));
    console.log(`${indent}</div>`);
  }
  
  find(name) {
    if (this.name === name) return this;
    
    for (const child of this.children) {
      if (child.name === name) return child;
      if (child instanceof Panel) {
        const found = child.find(name);
        if (found) return found;
      }
    }
    
    return null;
  }
}

class Form extends Panel {
  constructor(name) {
    super(name);
    this.data = {};
  }
  
  render(indent = '') {
    if (!this.visible) return;
    console.log(`${indent}<form>`);
    this.children.forEach(child => child.render(indent + '  '));
    console.log(`${indent}</form>`);
  }
  
  submit() {
    console.log('Form submitted with data:', this.data);
  }
}

// Usage
const loginForm = new Form('loginForm');

const usernamePanel = new Panel('usernamePanel');
usernamePanel.add(new Label('usernameLabel', 'Username:'));
usernamePanel.add(new TextInput('usernameInput', 'Enter username'));

const passwordPanel = new Panel('passwordPanel');
passwordPanel.add(new Label('passwordLabel', 'Password:'));
passwordPanel.add(new TextInput('passwordInput', 'Enter password'));

const buttonPanel = new Panel('buttonPanel');
const loginButton = new Button('loginButton', 'Login');
const cancelButton = new Button('cancelButton', 'Cancel');

loginButton.onClick(() => {
  console.log('Logging in...');
  loginForm.submit();
});

cancelButton.onClick(() => {
  console.log('Login cancelled');
});

buttonPanel.add(loginButton);
buttonPanel.add(cancelButton);

loginForm.add(usernamePanel);
loginForm.add(passwordPanel);
loginForm.add(buttonPanel);

// Render entire form
console.log('=== Login Form ===');
loginForm.render();

// Interact with components
console.log('\n=== Interactions ===');
const usernameInput = loginForm.find('usernameInput');
usernameInput.setValue('john_doe');

loginButton.click();

// Hide password panel
console.log('\n=== After hiding password ===');
passwordPanel.hide();
loginForm.render();
```

**Real-World Composite: Organization Structure:**

```javascript
class OrganizationUnit {
  constructor(name) {
    this.name = name;
  }
  
  getSalaryTotal() {
    throw new Error('getSalaryTotal() must be implemented');
  }
  
  getEmployeeCount() {
    throw new Error('getEmployeeCount() must be implemented');
  }
  
  print(indent = '') {
    throw new Error('print() must be implemented');
  }
}

// Leaf - Individual Employee
class Employee extends OrganizationUnit {
  constructor(name, position, salary) {
    super(name);
    this.position = position;
    this.salary = salary;
  }
  
  getSalaryTotal() {
    return this.salary;
  }
  
  getEmployeeCount() {
    return 1;
  }
  
  print(indent = '') {
    console.log(`${indent}👤 ${this.name} - ${this.position} ($${this.salary.toLocaleString()})`);
  }
}

// Composite - Department
class Department extends OrganizationUnit {
  constructor(name, manager) {
    super(name);
    this.manager = manager;
    this.members = [];
  }
  
  add(unit) {
    this.members.push(unit);
    return this;
  }
  
  remove(unit) {
    const index = this.members.indexOf(unit);
    if (index > -1) {
      this.members.splice(index, 1);
    }
    return this;
  }
  
  getSalaryTotal() {
    const managerSalary = this.manager.getSalaryTotal();
    const membersSalary = this.members.reduce(
      (total, member) => total + member.getSalaryTotal(),
      0
    );
    return managerSalary + membersSalary;
  }
  
  getEmployeeCount() {
    const managerCount = this.manager.getEmployeeCount();
    const membersCount = this.members.reduce(
      (total, member) => total + member.getEmployeeCount(),
      0
    );
    return managerCount + membersCount;
  }
  
  print(indent = '') {
    console.log(`${indent}🏢 ${this.name} Department`);
    console.log(`${indent}  Manager:`);
    this.manager.print(indent + '    ');
    
    if (this.members.length > 0) {
      console.log(`${indent}  Team Members:`);
      this.members.forEach(member => member.print(indent + '    '));
    }
    
    console.log(`${indent}  Total: ${this.getEmployeeCount()} employees, $${this.getSalaryTotal().toLocaleString()} salary`);
  }
}

// Usage
const ceo = new Employee('Alice Johnson', 'CEO', 250000);

const engineeringManager = new Employee('Bob Smith', 'Engineering Manager', 150000);
const engineeringDept = new Department('Engineering', engineeringManager);
engineeringDept.add(new Employee('Charlie Brown', 'Senior Engineer', 120000));
engineeringDept.add(new Employee('Diana Prince', 'Senior Engineer', 120000));
engineeringDept.add(new Employee('Eve Wilson', 'Junior Engineer', 80000));

const frontendTeamLead = new Employee('Frank Miller', 'Team Lead', 110000);
const frontendTeam = new Department('Frontend', frontendTeamLead);
frontendTeam.add(new Employee('Grace Lee', 'Developer', 90000));
frontendTeam.add(new Employee('Henry Ford', 'Developer', 90000));

engineeringDept.add(frontendTeam);

const salesManager = new Employee('Ivy Chen', 'Sales Manager', 130000);
const salesDept = new Department('Sales', salesManager);
salesDept.add(new Employee('Jack Black', 'Sales Rep', 70000));
salesDept.add(new Employee('Kate White', 'Sales Rep', 70000));

const company = new Department('TechCorp', ceo);
company.add(engineeringDept);
company.add(salesDept);

// Print organization structure
console.log('=== Organization Structure ===\n');
company.print();

console.log(`\n=== Company Summary ===`);
console.log(`Total Employees: ${company.getEmployeeCount()}`);
console.log(`Total Payroll: $${company.getSalaryTotal().toLocaleString()}`);
console.log(`Average Salary: $${Math.round(company.getSalaryTotal() / company.getEmployeeCount()).toLocaleString()}`);
```

---

### **16. Builder Pattern**

Separates the construction of a complex object from its representation, allowing the same construction process to create different representations.

```javascript
// Product
class Computer {
  constructor() {
    this.cpu = null;
    this.ram = null;
    this.storage = null;
    this.gpu = null;
    this.monitor = null;
    this.keyboard = null;
    this.mouse = null;
  }
  
  getSpecs() {
    return {
      cpu: this.cpu,
      ram: this.ram,
      storage: this.storage,
      gpu: this.gpu,
      monitor: this.monitor,
      keyboard: this.keyboard,
      mouse: this.mouse
    };
  }
  
  displaySpecs() {
    console.log('=== Computer Specifications ===');
    console.log(`CPU: ${this.cpu || 'Not specified'}`);
    console.log(`RAM: ${this.ram || 'Not specified'}`);
    console.log(`Storage: ${this.storage || 'Not specified'}`);
    console.log(`GPU: ${this.gpu || 'Not specified'}`);
    console.log(`Monitor: ${this.monitor || 'Not specified'}`);
    console.log(`Keyboard: ${this.keyboard || 'Not specified'}`);
    console.log(`Mouse: ${this.mouse || 'Not specified'}`);
  }
}

// Builder
class ComputerBuilder {
  constructor() {
    this.computer = new Computer();
  }
  
  setCPU(cpu) {
    this.computer.cpu = cpu;
    return this; // Enable method chaining
  }
  
  setRAM(ram) {
    this.computer.ram = ram;
    return this;
  }
  
  setStorage(storage) {
    this.computer.storage = storage;
    return this;
  }
  
  setGPU(gpu) {
    this.computer.gpu = gpu;
    return this;
  }
  
  setMonitor(monitor) {
    this.computer.monitor = monitor;
    return this;
  }
  
  setKeyboard(keyboard) {
    this.computer.keyboard = keyboard;
    return this;
  }
  
  setMouse(mouse) {
    this.computer.mouse = mouse;
    return this;
  }
  
  build() {
    return this.computer;
  }
}

// Director - knows how to build specific configurations
class ComputerDirector {
  static buildGamingPC() {
    return new ComputerBuilder()
      .setCPU('Intel Core i9-13900K')
      .setRAM('32GB DDR5')
      .setStorage('2TB NVMe SSD')
      .setGPU('NVIDIA RTX 4090')
      .setMonitor('34" 4K 144Hz')
      .setKeyboard('Mechanical RGB')
      .setMouse('Gaming Mouse 16000 DPI')
      .build();
  }
  
  static buildOfficePC() {
    return new ComputerBuilder()
      .setCPU('Intel Core i5-13400')
      .setRAM('16GB DDR4')
      .setStorage('512GB SSD')
      .setMonitor('24" 1080p')
      .setKeyboard('Standard Keyboard')
      .setMouse('Standard Mouse')
      .build();
  }
  
  static buildWorkstation() {
    return new ComputerBuilder()
      .setCPU('AMD Ryzen 9 7950X')
      .setRAM('64GB DDR5')
      .setStorage('4TB NVMe SSD')
      .setGPU('NVIDIA RTX 4080')
      .setMonitor('32" 4K IPS')
      .setKeyboard('Mechanical Keyboard')
      .setMouse('Precision Mouse')
      .build();
  }
}

// Usage
const gamingPC = ComputerDirector.buildGamingPC();
gamingPC.displaySpecs();

console.log('\n');

const officePC = ComputerDirector.buildOfficePC();
officePC.displaySpecs();

console.log('\n');

// Custom build
const customPC = new ComputerBuilder()
  .setCPU('AMD Ryzen 7 5800X')
  .setRAM('32GB DDR4')
  .setStorage('1TB SSD')
  .setGPU('AMD RX 6800 XT')
  .build();
customPC.displaySpecs();
```

**Advanced Builder: Query Builder:**

```javascript
class SQLQuery {
  constructor() {
    this.selectFields = [];
    this.fromTable = '';
    this.joins = [];
    this.whereClauses = [];
    this.orderByFields = [];
    this.limitValue = null;
    this.offsetValue = null;
  }
  
  toString() {
    let query = '';
    
    // SELECT
    if (this.selectFields.length > 0) {
      query += `SELECT ${this.selectFields.join(', ')}`;
    } else {
      query += 'SELECT *';
    }
    
    // FROM
    if (this.fromTable) {
      query += `\nFROM ${this.fromTable}`;
    }
    
    // JOINS
    if (this.joins.length > 0) {
      query += '\n' + this.joins.join('\n');
    }
    
    // WHERE
    if (this.whereClauses.length > 0) {
      query += `\nWHERE ${this.whereClauses.join(' AND ')}`;
    }
    
    // ORDER BY
    if (this.orderByFields.length > 0) {
      query += `\nORDER BY ${this.orderByFields.join(', ')}`;
    }
    
    // LIMIT
    if (this.limitValue !== null) {
      query += `\nLIMIT ${this.limitValue}`;
    }
    
    // OFFSET
    if (this.offsetValue !== null) {
      query += `\nOFFSET ${this.offsetValue}`;
    }
    
    return query + ';';
  }
}

class QueryBuilder {
  constructor() {
    this.query = new SQLQuery();
  }
  
  select(...fields) {
    this.query.selectFields = fields;
    return this;
  }
  
  from(table) {
    this.query.fromTable = table;
    return this;
  }
  
  join(table, condition, type = 'INNER') {
    this.query.joins.push(`${type} JOIN ${table} ON ${condition}`);
    return this;
  }
  
  leftJoin(table, condition) {
    return this.join(table, condition, 'LEFT');
  }
  
  rightJoin(table, condition) {
    return this.join(table, condition, 'RIGHT');
  }
  
  where(condition) {
    this.query.whereClauses.push(condition);
    return this;
  }
  
  whereEquals(field, value) {
    const formattedValue = typeof value === 'string' ? `'${value}'` : value;
    this.query.whereClauses.push(`${field} = ${formattedValue}`);
    return this;
  }
  
  whereIn(field, values) {
    const formattedValues = values
      .map(v => typeof v === 'string' ? `'${v}'` : v)
      .join(', ');
    this.query.whereClauses.push(`${field} IN (${formattedValues})`);
    return this;
  }
  
  orderBy(field, direction = 'ASC') {
    this.query.orderByFields.push(`${field} ${direction}`);
    return this;
  }
  
  limit(value) {
    this.query.limitValue = value;
    return this;
  }
  
  offset(value) {
    this.query.offsetValue = value;
    return this;
  }
  
  build() {
    return this.query.toString();
  }
  
  // Convenience method
  toString() {
    return this.build();
  }
}

// Usage
console.log('=== Simple Query ===');
const query1 = new QueryBuilder()
  .select('id', 'name', 'email')
  .from('users')
  .whereEquals('status', 'active')
  .orderBy('name', 'ASC')
  .limit(10)
  .build();
console.log(query1);

console.log('\n=== Complex Query with Joins ===');
const query2 = new QueryBuilder()
  .select('u.id', 'u.name', 'o.order_date', 'o.total')
  .from('users u')
  .leftJoin('orders o', 'u.id = o.user_id')
  .leftJoin('order_items oi', 'o.id = oi.order_id')
  .whereEquals('u.status', 'active')
  .whereIn('o.status', ['completed', 'shipped'])
  .orderBy('o.order_date', 'DESC')
  .limit(20)
  .offset(10)
  .build();
console.log(query2);

console.log('\n=== Filter Query ===');
const query3 = new QueryBuilder()
  .select('*')
  .from('products')
  .where('price > 100')
  .where('stock > 0')
  .whereIn('category', ['electronics', 'computers'])
  .orderBy('price', 'DESC')
  .build();
console.log(query3);
```

**Real-World Builder: HTTP Request Builder:**

```javascript
class HTTPRequest {
  constructor() {
    this.method = 'GET';
    this.url = '';
    this.headers = {};
    this.queryParams = {};
    this.body = null;
    this.timeout = 30000;
    this.retries = 0;
  }
  
  async execute() {
    const url = new URL(this.url);
    
    // Add query parameters
    Object.keys(this.queryParams).forEach(key => {
      url.searchParams.append(key, this.queryParams[key]);
    });
    
    const options = {
      method: this.method,
      headers: this.headers
    };
    
    if (this.body) {
      options.body = typeof this.body === 'string' 
        ? this.body 
        : JSON.stringify(this.body);
    }
    
    console.log(`\n[HTTP] ${this.method} ${url}`);
    console.log('[Headers]', this.headers);
    if (this.body) console.log('[Body]', this.body);
    
    // Simulated response
    return {
      status: 200,
      data: { success: true, message: 'Request successful' }
    };
  }
}

class HTTPRequestBuilder {
  constructor() {
    this.request = new HTTPRequest();
  }
  
  setMethod(method) {
    this.request.method = method.toUpperCase();
    return this;
  }
  
  get(url) {
    this.request.method = 'GET';
    this.request.url = url;
    return this;
  }
  
  post(url) {
    this.request.method = 'POST';
    this.request.url = url;
    return this;
  }
  
  put(url) {
    this.request.method = 'PUT';
    this.request.url = url;
    return this;
  }
  
  delete(url) {
    this.request.method = 'DELETE';
    this.request.url = url;
    return this;
  }
  
  setHeader(key, value) {
    this.request.headers[key] = value;
    return this;
  }
  
  setHeaders(headers) {
    this.request.headers = { ...this.request.headers, ...headers };
    return this;
  }
  
  setAuth(token, type = 'Bearer') {
    this.request.headers['Authorization'] = `${type} ${token}`;
    return this;
  }
  
  setContentType(contentType) {
    this.request.headers['Content-Type'] = contentType;
    return this;
  }
  
  json() {
    this.setContentType('application/json');
    return this;
  }
  
  setQueryParam(key, value) {
    this.request.queryParams[key] = value;
    return this;
  }
  
  setQueryParams(params) {
    this.request.queryParams = { ...this.request.queryParams, ...params };
    return this;
  }
  
  setBody(body) {
    this.request.body = body;
    return this;
  }
  
  setTimeout(timeout) {
    this.request.timeout = timeout;
    return this;
  }
  
  setRetries(retries) {
    this.request.retries = retries;
    return this;
  }
  
  build() {
    return this.request;
  }
  
  async send() {
    return this.request.execute();
  }
}

// Usage
console.log('=== GET Request ===');
const getRequest = new HTTPRequestBuilder()
  .get('https://api.example.com/users')
  .setAuth('my-token-123')
  .setQueryParams({
    page: 1,
    limit: 10,
    sort: 'name'
  })
  .send();

console.log('\n=== POST Request ===');
const postRequest = new HTTPRequestBuilder()
  .post('https://api.example.com/users')
  .json()
  .setAuth('my-token-123')
  .setBody({
    name: 'John Doe',
    email: 'john@example.com',
    role: 'admin'
  })
  .setTimeout(5000)
  .send();

console.log('\n=== PUT Request with Custom Headers ===');
const putRequest = new HTTPRequestBuilder()
  .put('https://api.example.com/users/123')
  .setHeaders({
    'Content-Type': 'application/json',
    'Authorization': 'Bearer token-xyz',
    'X-Custom-Header': 'custom-value'
  })
  .setBody({
    name: 'Jane Doe',
    email: 'jane@example.com'
  })
  .setRetries(3)
  .send();
```

---

### **17. Prototype Pattern**

Creates objects based on a template of an existing object through cloning. Avoids expensive initialization by copying existing instances.

```javascript
// Prototype with Object.create()
const carPrototype = {
  drive() {
    console.log(`${this.make} ${this.model} is driving`);
  },
  
  brake() {
    console.log(`${this.make} ${this.model} is braking`);
  },
  
  getInfo() {
    return `${this.year} ${this.make} ${this.model} - ${this.color}`;
  }
};

function createCar(make, model, year, color) {
  const car = Object.create(carPrototype);
  car.make = make;
  car.model = model;
  car.year = year;
  car.color = color;
  return car;
}

// Usage
const car1 = createCar('Toyota', 'Camry', 2024, 'Silver');
const car2 = createCar('Honda', 'Accord', 2024, 'Blue');

console.log(car1.getInfo()); // 2024 Toyota Camry - Silver
car1.drive(); // Toyota Camry is driving

console.log(car2.getInfo()); // 2024 Honda Accord - Blue
car2.drive(); // Honda Accord is driving
```

**Advanced Prototype with Cloning:**

```javascript
class Character {
  constructor(name, health, strength, skills = []) {
    this.name = name;
    this.health = health;
    this.strength = strength;
    this.skills = skills;
    this.inventory = [];
    this.position = { x: 0, y: 0 };
  }
  
  // Deep clone method
  clone() {
    const cloned = Object.create(Object.getPrototypeOf(this));
    
    // Deep copy properties
    cloned.name = this.name + ' (Clone)';
    cloned.health = this.health;
    cloned.strength = this.strength;
    cloned.skills = [...this.skills]; // Copy array
    cloned.inventory = this.inventory.map(item => ({ ...item })); // Deep copy
    cloned.position = { ...this.position }; // Copy object
    
    return cloned;
  }
  
  attack() {
    console.log(`${this.name} attacks with ${this.strength} strength!`);
  }
  
  useSkill(skillIndex) {
    if (this.skills[skillIndex]) {
      console.log(`${this.name} uses ${this.skills[skillIndex]}!`);
    }
  }
  
  addItem(item) {
    this.inventory.push(item);
  }
  
  move(x, y) {
    this.position = { x, y };
    console.log(`${this.name} moved to (${x}, ${y})`);
  }
  
  getInfo() {
    return {
      name: this.name,
      health: this.health,
      strength: this.strength,
      skills: this.skills.length,
      inventory: this.inventory.length,
      position: this.position
    };
  }
}

// Create prototype characters
const warriorPrototype = new Character('Warrior', 100, 20, ['Slash', 'Shield Block', 'Power Strike']);
const magePrototype = new Character('Mage', 70, 10, ['Fireball', 'Ice Bolt', 'Teleport']);

// Clone characters
const player1 = warriorPrototype.clone();
player1.name = 'Player 1 Warrior';
player1.addItem({ name: 'Sword', damage: 15 });
player1.addItem({ name: 'Shield', defense: 10 });
player1.move(10, 20);

const player2 = warriorPrototype.clone();
player2.name = 'Player 2 Warrior';
player2.addItem({ name: 'Axe', damage: 18 });
player2.move(15, 25);

const player3 = magePrototype.clone();
player3.name = 'Player 1 Mage';
player3.addItem({ name: 'Staff', magic: 20 });
player3.move(5, 10);

// Use cloned characters
console.log('\n=== Character Info ===');
console.log('Player 1:', player1.getInfo());
console.log('Player 2:', player2.getInfo());
console.log('Player 3:', player3.getInfo());

console.log('\n=== Actions ===');
player1.attack();
player1.useSkill(0);

player3.attack();
player3.useSkill(2);

// Verify independence
console.log('\n=== Verifying Independence ===');
console.log('Player 1 inventory:', player1.inventory);
console.log('Player 2 inventory:', player2.inventory);
console.log('Prototypes unchanged:', warriorPrototype.inventory.length === 0);
```

**Real-World Prototype: Document Templates:**

```javascript
class DocumentPrototype {
  constructor() {
    this.title = '';
    this.content = '';
    this.metadata = {};
    this.sections = [];
    this.formatting = {};
  }
  
  clone() {
    const cloned = Object.create(Object.getPrototypeOf(this));
    
    // Deep copy all properties
    cloned.title = this.title;
    cloned.content = this.content;
    cloned.metadata = JSON.parse(JSON.stringify(this.metadata));
    cloned.sections = JSON.parse(JSON.stringify(this.sections));
    cloned.formatting = JSON.parse(JSON.stringify(this.formatting));
    
    return cloned;
  }
  
  setTitle(title) {
    this.title = title;
    return this;
  }
  
  setContent(content) {
    this.content = content;
    return this;
  }
  
  addSection(section) {
    this.sections.push(section);
    return this;
  }
  
  setMetadata(key, value) {
    this.metadata[key] = value;
    return this;
  }
  
  setFormatting(formatting) {
    this.formatting = { ...this.formatting, ...formatting };
    return this;
  }
  
  render() {
    console.log('='.repeat(50));
    console.log(`TITLE: ${this.title}`);
    console.log(`Author: ${this.metadata.author || 'Unknown'}`);
    console.log(`Date: ${this.metadata.date || 'N/A'}`);
    console.log('='.repeat(50));
    
    if (this.content) {
      console.log(`\n${this.content}\n`);
    }
    
    this.sections.forEach((section, index) => {
      console.log(`\n${index + 1}. ${section.title}`);
      console.log(section.content);
    });
    
    console.log('\n' + '='.repeat(50));
  }
}

// Create template documents
const reportTemplate = new DocumentPrototype()
  .setMetadata('type', 'report')
  .setMetadata('author', 'System')
  .setFormatting({
    font: 'Arial',
    fontSize: 12,
    margins: { top: 1, bottom: 1, left: 1, right: 1 }
  })
  .addSection({
    title: 'Executive Summary',
    content: '[Summary content goes here]'
  })
  .addSection({
    title: 'Findings',
    content: '[Findings content goes here]'
  })
  .addSection({
    title: 'Recommendations',
    content: '[Recommendations content goes here]'
  });

const letterTemplate = new DocumentPrototype()
  .setMetadata('type', 'letter')
  .setFormatting({
    font: 'Times New Roman',
    fontSize: 11,
    margins: { top: 1.5, bottom: 1.5, left: 1, right: 1 }
  })
  .addSection({
    title: 'Opening',
    content: 'Dear [Recipient],'
  })
  .addSection({
    title: 'Body',
    content: '[Letter content goes here]'
  })
  .addSection({
    title: 'Closing',
    content: 'Sincerely,\n[Your Name]'
  });

// Clone and customize documents
console.log('=== Monthly Report ===');
const monthlyReport = reportTemplate.clone()
  .setTitle('Monthly Sales Report - December 2024')
  .setMetadata('author', 'John Doe')
  .setMetadata('date', '2024-12-20')
  .setContent('This report summarizes sales performance for December 2024.');

monthlyReport.render();

console.log('\n=== Welcome Letter ===');
const welcomeLetter = letterTemplate.clone()
  .setTitle('Welcome Letter')
  .setMetadata('author', 'HR Department')
  .setMetadata('date', '2024-12-20');

welcomeLetter.sections[0].content = 'Dear New Employee,';
welcomeLetter.sections[1].content = 'Welcome to our company! We are excited to have you join our team.';
welcomeLetter.sections[2].content = 'Best regards,\nHR Team';

welcomeLetter.render();
```

---

### **18. State Pattern**

Allows an object to alter its behavior when its internal state changes. The object will appear to change its class.

```javascript
// State interface
class State {
  handle(context) {
    throw new Error('handle() must be implemented');
  }
}

// Concrete States
class RedLight extends State {
  handle(context) {
    console.log('🔴 RED - Stop');
    setTimeout(() => {
      context.setState(new GreenLight());
    }, 3000);
  }
}

class YellowLight extends State {
  handle(context) {
    console.log('🟡 YELLOW - Slow down');
    setTimeout(() => {
      context.setState(new RedLight());
    }, 1000);
  }
}

class GreenLight extends State {
  handle(context) {
    console.log('🟢 GREEN - Go');
    setTimeout(() => {
      context.setState(new YellowLight());
    }, 5000);
  }
}

// Context
class TrafficLight {
  constructor() {
    this.state = new RedLight();
  }
  
  setState(state) {
    this.state = state;
    this.state.handle(this);
  }
  
  start() {
    this.state.handle(this);
  }
}

// Usage
const trafficLight = new TrafficLight();
trafficLight.start();
```

**Advanced State: Order Processing:**

```javascript
class OrderState {
  constructor(name) {
    this.name = name;
  }
  
  process(order) {
    throw new Error('process() must be implemented');
  }
  
  cancel(order) {
    throw new Error('cancel() must be implemented');
  }
  
  ship(order) {
    throw new Error('ship() must be implemented');
  }
  
  deliver(order) {
    throw new Error('deliver() must be implemented');
  }
}

class PendingState extends OrderState {
  constructor() {
    super('Pending');
  }
  
  process(order) {
    console.log(`✓ Order #${order.id}: Pending → Processing`);
    order.setState(new ProcessingState());
    return true;
  }
  
  cancel(order) {
    console.log(`✓ Order #${order.id}: Pending → Cancelled`);
    order.setState(new CancelledState());
    return true;
  }
  
  ship(order) {
    console.log(`✗ Cannot ship order #${order.id} - must be processed first`);
    return false;
  }
  
  deliver(order) {
    console.log(`✗ Cannot deliver order #${order.id} - must be shipped first`);
    return false;
  }
}

class ProcessingState extends OrderState {
  constructor() {
    super('Processing');
  }
  
  process(order) {
    console.log(`✗ Order #${order.id} is already being processed`);
    return false;
  }
  
  cancel(order) {
    console.log(`✓ Order #${order.id}: Processing → Cancelled`);
    order.setState(new CancelledState());
    return true;
  }
  
  ship(order) {
    console.log(`✓ Order #${order.id}: Processing → Shipped`);
    order.setState(new ShippedState());
    return true;
  }
  
  deliver(order) {
    console.log(`✗ Cannot deliver order #${order.id} - must be shipped first`);
    return false;
  }
}

class ShippedState extends OrderState {
  constructor() {
    super('Shipped');
  }
  
  process(order) {
    console.log(`✗ Order #${order.id} has already been shipped`);
    return false;
  }
  
  cancel(order) {
    console.log(`✗ Cannot cancel order #${order.id} - already shipped`);
    return false;
  }
  
  ship(order) {
    console.log(`✗ Order #${order.id} has already been shipped`);
    return false;
  }
  
  deliver(order) {
    console.log(`✓ Order #${order.id}: Shipped → Delivered`);
    order.setState(new DeliveredState());
    return true;
  }
}

class DeliveredState extends OrderState {
  constructor() {
    super('Delivered');
  }
  
  process(order) {
    console.log(`✗ Order #${order.id} has already been delivered`);
    return false;
  }
  
  cancel(order) {
    console.log(`✗ Cannot cancel order #${order.id} - already delivered`);
    return false;
  }
  
  ship(order) {
    console.log(`✗ Order #${order.id} has already been delivered`);
    return false;
  }
  
  deliver(order) {
    console.log(`✗ Order #${order.id} has already been delivered`);
    return false;
  }
}

class CancelledState extends OrderState {
  constructor() {
    super('Cancelled');
  }
  
  process(order) {
    console.log(`✗ Cannot process cancelled order #${order.id}`);
    return false;
  }
  
  cancel(order) {
    console.log(`✗ Order #${order.id} is already cancelled`);
    return false;
  }
  
  ship(order) {
    console.log(`✗ Cannot ship cancelled order #${order.id}`);
    return false;
  }
  
  deliver(order) {
    console.log(`✗ Cannot deliver cancelled order #${order.id}`);
    return false;
  }
}

class Order {
  constructor(id, items) {
    this.id = id;
    this.items = items;
    this.state = new PendingState();
    this.history = [{ state: 'Pending', timestamp: new Date() }];
  }
  
  setState(state) {
    this.state = state;
    this.history.push({
      state: state.name,
      timestamp: new Date()
    });
  }
  
  getState() {
    return this.state.name;
  }
  
  process() {
    return this.state.process(this);
  }
  
  cancel() {
    return this.state.cancel(this);
  }
  
  ship() {
    return this.state.ship(this);
  }
  
  deliver() {
    return this.state.deliver(this);
  }
  
  getHistory() {
    return this.history.map(h => 
      `${h.state} at ${h.timestamp.toLocaleString()}`
    ).join('\n');
  }
}

// Usage
console.log('=== Order State Management ===\n');

const order1 = new Order(1001, ['Laptop', 'Mouse']);
console.log(`Order #${order1.id} - Current state: ${order1.getState()}\n`);

order1.process();  // Pending → Processing
order1.ship();     // Processing → Shipped
order1.deliver();  // Shipped → Delivered

console.log(`\nOrder #${order1.id} - Final state: ${order1.getState()}`);
console.log('\nOrder History:');
console.log(order1.getHistory());

// Test invalid transitions
console.log('\n=== Testing Invalid Transitions ===\n');
const order2 = new Order(1002, ['Phone']);
order2.ship();     // ✗ Cannot ship - not processed
order2.cancel();   // ✓ Pending → Cancelled
order2.process();  // ✗ Cannot process - cancelled
```

**Real-World State: Media Player:**

```javascript
class PlayerState {
  play(player) {
    console.log('Not implemented');
  }
  
  pause(player) {
    console.log('Not implemented');
  }
  
  stop(player) {
    console.log('Not implemented');
  }
  
  next(player) {
    console.log('Not implemented');
  }
  
  previous(player) {
    console.log('Not implemented');
  }
}

class StoppedState extends PlayerState {
  play(player) {
    console.log('▶️  Starting playback...');
    player.currentTime = 0;
    player.setState(new PlayingState());
  }
  
  pause(player) {
    console.log('⏸️  Already stopped');
  }
  
  stop(player) {
    console.log('⏹️  Already stopped');
  }
  
  next(player) {
    console.log('⏭️  Skipping to next track...');
    player.nextTrack();
    player.setState(new PlayingState());
  }
  
  previous(player) {
    console.log('⏮️  Going to previous track...');
    player.previousTrack();
    player.setState(new PlayingState());
  }
}

class PlayingState extends PlayerState {
  play(player) {
    console.log('▶️  Already playing');
  }
  
  pause(player) {
    console.log('⏸️  Pausing playback...');
    player.setState(new PausedState());
  }
  
  stop(player) {
    console.log('⏹️  Stopping playback...');
    player.currentTime = 0;
    player.setState(new StoppedState());
  }
  
  next(player) {
    console.log('⏭️  Skipping to next track...');
    player.nextTrack();
  }
  
  previous(player) {
    console.log('⏮️  Going to previous track...');
    player.previousTrack();
  }
}

class PausedState extends PlayerState {
  play(player) {
    console.log('▶️  Resuming playback...');
    player.setState(new PlayingState());
  }
  
  pause(player) {
    console.log('⏸️  Already paused');
  }
  
  stop(player) {
    console.log('⏹️  Stopping playback...');
    player.currentTime = 0;
    player.setState(new StoppedState());
  }
  
  next(player) {
    console.log('⏭️  Skipping to next track...');
    player.nextTrack();
    player.setState(new PlayingState());
  }
  
  previous(player) {
    console.log('⏮️  Going to previous track...');
    player.previousTrack();
    player.setState(new PlayingState());
  }
}

class MediaPlayer {
  constructor(playlist) {
    this.playlist = playlist;
    this.currentTrackIndex = 0;
    this.currentTime = 0;
    this.state = new StoppedState();
  }
  
  setState(state) {
    this.state = state;
  }
  
  play() {
    console.log(`\n[${this.getCurrentTrack()}]`);
    this.state.play(this);
  }
  
  pause() {
    console.log(`\n[${this.getCurrentTrack()}]`);
    this.state.pause(this);
  }
  
  stop() {
    console.log(`\n[${this.getCurrentTrack()}]`);
    this.state.stop(this);
  }
  
  next() {
    console.log(`\n[${this.getCurrentTrack()}]`);
    this.state.next(this);
  }
  
  previous() {
    console.log(`\n[${this.getCurrentTrack()}]`);
    this.state.previous(this);
  }
  
  nextTrack() {
    this.currentTrackIndex = (this.currentTrackIndex + 1) % this.playlist.length;
    this.currentTime = 0;
  }
  
  previousTrack() {
    this.currentTrackIndex = this.currentTrackIndex === 0 
      ? this.playlist.length - 1 
      : this.currentTrackIndex - 1;
    this.currentTime = 0;
  }
  
  getCurrentTrack() {
    return this.playlist[this.currentTrackIndex];
  }
  
  getStatus() {
    return {
      track: this.getCurrentTrack(),
      state: this.state.constructor.name.replace('State', ''),
      time: this.currentTime
    };
  }
}

// Usage
const player = new MediaPlayer([
  'Song 1.mp3',
  'Song 2.mp3',
  'Song 3.mp3'
]);

console.log('=== Media Player State Management ===');

player.play();      // Stopped → Playing
player.pause();     // Playing → Paused
player.play();      // Paused → Playing
player.next();      // Skip to next track
player.previous();  // Go back
player.stop();      // Playing → Stopped
player.pause();     // Already stopped

console.log('\n=== Current Status ===');
console.log(player.getStatus());
```

---

### **19. Template Method Pattern**

Defines the skeleton of an algorithm in a base class, letting subclasses override specific steps without changing the algorithm's structure.

```javascript
// Abstract class
class DataProcessor {
  // Template method - defines algorithm structure
  process(data) {
    console.log('=== Starting Data Processing ===\n');
    
    const validated = this.validate(data);
    if (!validated) {
      console.log('❌ Validation failed');
      return null;
    }
    
    const extracted = this.extract(validated);
    const transformed = this.transform(extracted);
    const loaded = this.load(transformed);
    
    this.cleanup();
    
    console.log('\n=== Processing Complete ===');
    return loaded;
  }
  
  // Abstract methods - must be implemented by subclasses
  validate(data) {
    throw new Error('validate() must be implemented');
  }
  
  extract(data) {
    throw new Error('extract() must be implemented');
  }
  
  transform(data) {
    throw new Error('transform() must be implemented');
  }
  
  load(data) {
    throw new Error('load() must be implemented');
  }
  
  // Hook method - optional override
  cleanup() {
    console.log('\n🧹 Cleanup completed');
  }
}

// Concrete implementation 1
class CSVProcessor extends DataProcessor {
  validate(data) {
    console.log('✓ Validating CSV format...');
    return typeof data === 'string' && data.includes(',');
  }
  
  extract(data) {
    console.log('✓ Extracting CSV data...');
    const lines = data.split('\n');
    const headers = lines[0].split(',');
    const rows = lines.slice(1).map(line => line.split(','));
    return { headers, rows };
  }
  
  transform(data) {
    console.log('✓ Transforming CSV to objects...');
    return data.rows.map(row => {
      const obj = {};
      data.headers.forEach((header, index) => {
        obj[header.trim()] = row[index]?.trim();
      });
      return obj;
    });
  }
  
  load(data) {
    console.log(`✓ Loading ${data.length} records...`);
    return data;
  }
  
  cleanup() {
    console.log('\n🧹 CSV processor cleanup: Closing file handles');
  }
}

// Concrete implementation 2
class JSONProcessor extends DataProcessor {
  validate(data) {
    console.log('✓ Validating JSON format...');
    try {
      JSON.parse(data);
      return true;
    } catch {
      return false;
    }
  }
  
  extract(data) {
    console.log('✓ Extracting JSON data...');
    return JSON.parse(data);
  }
  
  transform(data) {
    console.log('✓ Transforming JSON data...');
    // Normalize field names to lowercase
    if (Array.isArray(data)) {
      return data.map(item => {
        const normalized = {};
        Object.keys(item).forEach(key => {
          normalized[key.toLowerCase()] = item[key];
        });
        return normalized;
      });
    }
    return data;
  }
  
  load(data) {
    console.log(`✓ Loading ${Array.isArray(data) ? data.length : 1} record(s)...`);
    return data;
  }
  
  cleanup() {
    console.log('\n🧹 JSON processor cleanup: Clearing cache');
  }
}

// Concrete implementation 3
class XMLProcessor extends DataProcessor {
  validate(data) {
    console.log('✓ Validating XML format...');
    return typeof data === 'string' && data.includes('<') && data.includes('>');
  }
  
  extract(data) {
    console.log('✓ Extracting XML data...');
    // Simplified XML parsing
    const items = [];
    const itemMatches = data.matchAll(/<item>([\s\S]*?)<\/item>/g);
    
    for (const match of itemMatches) {
      items.push(match[1]);
    }
    
    return items;
  }
  
  transform(data) {
    console.log('✓ Transforming XML to objects...');
    return data.map(item => {
      const obj = {};
      const fieldMatches = item.matchAll(/<(\w+)>(.*?)<\/\1>/g);
      
      for (const match of fieldMatches) {
        obj[match[1]] = match[2];
      }
      
      return obj;
    });
  }
  
  load(data) {
    console.log(`✓ Loading ${data.length} records...`);
    return data;
  }
}

// Usage
const csvData = `name,age,city
John Doe,30,New York
Jane Smith,25,Los Angeles`;

const jsonData = `[
  {"Name": "John Doe", "Age": 30, "City": "New York"},
  {"Name": "Jane Smith", "Age": 25, "City": "Los Angeles"}
]`;

const xmlData = `
<items>
  <item><name>John Doe</name><age>30</age><city>New York</city></item>
  <item><name>Jane Smith</name><age>25</age><city>Los Angeles</city></item>
</items>`;

console.log('CSV Processing:');
const csvProcessor = new CSVProcessor();
const csvResult = csvProcessor.process(csvData);
console.log('Result:', csvResult);

console.log('\n' + '='.repeat(50) + '\n');

console.log('JSON Processing:');
const jsonProcessor = new JSONProcessor();
const jsonResult = jsonProcessor.process(jsonData);
console.log('Result:', jsonResult);

console.log('\n' + '='.repeat(50) + '\n');

console.log('XML Processing:');
const xmlProcessor = new XMLProcessor();
const xmlResult = xmlProcessor.process(xmlData);
console.log('Result:', xmlResult);
```

**Real-World Template: Report Generator:**

```javascript
class ReportGenerator {
  // Template method
  generate(data, options = {}) {
    console.log(`\n=== Generating ${this.getReportType()} Report ===\n`);
    
    this.initialize(options);
    
    const processedData = this.processData(data);
    
    this.renderHeader(options);
    this.renderBody(processedData);
    this.renderFooter(options);
    
    const output = this.finalize();
    
    console.log('\n✓ Report generation complete\n');
    return output;
  }
  
  // Abstract methods
  getReportType() {
    throw new Error('getReportType() must be implemented');
  }
  
  renderHeader(options) {
    throw new Error('renderHeader() must be implemented');
  }
  
  renderBody(data) {
    throw new Error('renderBody() must be implemented');
  }
  
  renderFooter(options) {
    throw new Error('renderFooter() must be implemented');
  }
  
  // Concrete methods with default implementation
  initialize(options) {
    this.output = [];
    this.options = options;
  }
  
  processData(data) {
    return data;
  }
  
  finalize() {
    return this.output.join('\n');
  }
}

class PDFReport extends ReportGenerator {
  getReportType() {
    return 'PDF';
  }
  
  renderHeader(options) {
    this.output.push('PDF HEADER');
    this.output.push('='.repeat(50));
    this.output.push(`Title: ${options.title || 'Untitled Report'}`);
    this.output.push(`Date: ${new Date().toLocaleDateString()}`);
    this.output.push(`Author: ${options.author || 'Unknown'}`);
    this.output.push('='.repeat(50));
  }
  
  renderBody(data) {
    this.output.push('\nCONTENT:');
    data.forEach((item, index) => {
      this.output.push(`\n${index + 1}. ${item.title}`);
      this.output.push(`   ${item.description}`);
      this.output.push(`   Value: ${item.value}`);
    });
  }
  
  renderFooter(options) {
    this.output.push('\n' + '='.repeat(50));
    this.output.push('PDF FOOTER');
    this.output.push(`Page 1 of 1`);
    this.output.push(`© ${options.company || 'Company'} ${new Date().getFullYear()}`);
  }
}

class HTMLReport extends ReportGenerator {
  getReportType() {
    return 'HTML';
  }
  
  renderHeader(options) {
    this.output.push('<!DOCTYPE html>');
    this.output.push('<html>');
    this.output.push('<head>');
    this.output.push(`  <title>${options.title || 'Report'}</title>`);
    this.output.push('  <style>');
    this.output.push('    body { font-family: Arial, sans-serif; margin: 20px; }');
    this.output.push('    h1 { color: #333; }');
    this.output.push('    .item { margin: 10px 0; padding: 10px; border: 1px solid #ddd; }');
    this.output.push('  </style>');
    this.output.push('</head>');
    this.output.push('<body>');
    this.output.push(`  <h1>${options.title || 'Report'}</h1>`);
    this.output.push(`  <p>Date: ${new Date().toLocaleDateString()}</p>`);
  }
  
  renderBody(data) {
    this.output.push('  <div class="content">');
    data.forEach(item => {
      this.output.push('    <div class="item">');
      this.output.push(`      <h3>${item.title}</h3>`);
      this.output.push(`      <p>${item.description}</p>`);
      this.output.push(`      <strong>Value: ${item.value}</strong>`);
      this.output.push('    </div>');
    });
    this.output.push('  </div>');
  }
  
  renderFooter(options) {
    this.output.push('  <footer>');
    this.output.push(`    <p>© ${options.company || 'Company'} ${new Date().getFullYear()}</p>`);
    this.output.push('  </footer>');
    this.output.push('</body>');
    this.output.push('</html>');
  }
}

class MarkdownReport extends ReportGenerator {
  getReportType() {
    return 'Markdown';
  }
  
  renderHeader(options) {
    this.output.push(`# ${options.title || 'Report'}`);
    this.output.push('');
    this.output.push(`**Date:** ${new Date().toLocaleDateString()}`);
    this.output.push(`**Author:** ${options.author || 'Unknown'}`);
    this.output.push('');
    this.output.push('---');
  }
  
  renderBody(data) {
    this.output.push('');
    this.output.push('## Content');
    this.output.push('');
    data.forEach((item, index) => {
      this.output.push(`### ${index + 1}. ${item.title}`);
      this.output.push('');
      this.output.push(item.description);
      this.output.push('');
      this.output.push(`**Value:** ${item.value}`);
      this.output.push('');
    });
  }
  
  renderFooter(options) {
    this.output.push('---');
    this.output.push('');
    this.output.push(`*© ${options.company || 'Company'} ${new Date().getFullYear()}*`);
  }
}

// Usage
const reportData = [
  {
    title: 'Q4 Revenue',
    description: 'Total revenue for Q4 2024',
    value: '$1,250,000'
  },
  {
    title: 'Customer Growth',
    description: 'New customers acquired',
    value: '450 customers'
  },
  {
    title: 'Product Sales',
    description: 'Top-selling products',
    value: 'Product A, Product B'
  }
];

const options = {
  title: 'Q4 2024 Business Report',
  author: 'John Doe',
  company: 'TechCorp'
};

// Generate PDF report
const pdfGenerator = new PDFReport();
const pdfOutput = pdfGenerator.generate(reportData, options);
console.log(pdfOutput);

// Generate HTML report
const htmlGenerator = new HTMLReport();
const htmlOutput = htmlGenerator.generate(reportData, options);
console.log(htmlOutput);

// Generate Markdown report
const mdGenerator = new MarkdownReport();
const mdOutput = mdGenerator.generate(reportData, options);
console.log(mdOutput);
```

---

## **Pattern Comparison & Selection Guide**

### **When to Use Each Pattern**

| Pattern | Use When | Don't Use When | Complexity |
|---------|----------|----------------|------------|
| **Module** | Need encapsulation, private state | ES6 modules available | Low |
| **Singleton** | Need exactly one instance (DB, config) | Testing is priority, need multiple instances | Low |
| **Factory** | Object creation is complex, many types | Simple object creation | Medium |
| **Observer** | Need event-driven architecture, pub/sub | Simple one-to-one communication | Medium |
| **Strategy** | Multiple algorithms, runtime selection | Only one algorithm needed | Low |
| **Decorator** | Add behavior without subclassing | Core functionality changes | Medium |
| **Proxy** | Control access, lazy loading, caching | Direct access is sufficient | Medium |
| **Facade** | Simplify complex subsystems | System is already simple | Low |
| **Command** | Need undo/redo, queuing operations | Simple direct method calls | Medium |
| **Iterator** | Custom traversal logic needed | Built-in iteration sufficient | Low |
| **Chain of Responsibility** | Multiple handlers, request processing | Single handler sufficient | Medium |
| **Mediator** | Complex object communication | Simple direct communication | High |
| **Adapter** | Integrate incompatible interfaces | Interfaces already compatible | Low |
| **Composite** | Tree structures, part-whole hierarchies | Flat structures | Medium |
| **Builder** | Complex object construction, many options | Simple object creation | Medium |
| **Prototype** | Object cloning, avoid expensive creation | Objects are cheap to create | Low |
| **State** | Behavior changes based on state | Simple if-else sufficient | Medium |
| **Template Method** | Algorithm skeleton, variable steps | No common algorithm structure | Low |

### **Pattern Categories & Relationships**

**Creational Patterns** (Object Creation):
```
Singleton ──────► One instance globally
Factory ─────────► Create objects by type
Builder ─────────► Step-by-step construction
Prototype ───────► Clone existing objects
```

**Structural Patterns** (Object Composition):
```
Adapter ─────────► Make interfaces compatible
Decorator ───────► Add behavior dynamically
Facade ──────────► Simplify complex systems
Composite ───────► Tree structures
Proxy ───────────► Control access
```

**Behavioral Patterns** (Object Interaction):
```
Observer ────────► Event notification
Strategy ────────► Algorithm selection
Command ─────────► Encapsulate requests
State ───────────► Behavior by state
Chain ───────────► Pass requests along chain
Iterator ────────► Sequential access
Mediator ────────► Centralized communication
Template Method ─► Algorithm skeleton
```

### **Pattern Combinations**

Patterns often work together:

```javascript
// Singleton + Factory
class DatabaseFactory {
  static instance = null;
  
  static getInstance() {
    if (!DatabaseFactory.instance) {
      DatabaseFactory.instance = new DatabaseFactory();
    }
    return DatabaseFactory.instance;
  }
  
  createConnection(type) {
    switch (type) {
      case 'mysql':
        return new MySQLConnection();
      case 'postgres':
        return new PostgresConnection();
      default:
        throw new Error('Unknown database type');
    }
  }
}

// Strategy + Factory
class CompressionFactory {
  static create(type) {
    const strategies = {
      'zip': new ZipCompressionStrategy(),
      'gzip': new GzipCompressionStrategy(),
      'bzip2': new Bzip2CompressionStrategy()
    };
    return strategies[type] || strategies['zip'];
  }
}

// Observer + Mediator
class ChatMediator {
  constructor() {
    this.users = [];
    this.observers = []; // Log observers, analytics observers, etc.
  }
  
  subscribe(observer) {
    this.observers.push(observer);
  }
  
  notify(message) {
    this.observers.forEach(observer => observer.update(message));
  }
  
  sendMessage(message, from, to) {
    this.notify({ type: 'message', message, from, to });
    to.receive(message, from);
  }
}

// Decorator + Proxy
class CachedAPIProxy {
  constructor(api) {
    this.api = api;
    this.cache = new Map();
  }
  
  async get(endpoint) {
    if (this.cache.has(endpoint)) {
      return this.cache.get(endpoint);
    }
    
    // Decorated API with logging
    const result = await this.api.get(endpoint);
    this.cache.set(endpoint, result);
    return result;
  }
}
```

---

## **Anti-Patterns to Avoid**

### **1. God Object / Monster Class**

**Problem:** One class that does everything.

```javascript
// ❌ BAD - God Object
class Application {
  constructor() {
    this.users = [];
    this.products = [];
    this.orders = [];
    this.database = null;
    this.server = null;
  }
  
  // User management
  createUser(data) { /* ... */ }
  updateUser(id, data) { /* ... */ }
  deleteUser(id) { /* ... */ }
  
  // Product management
  createProduct(data) { /* ... */ }
  updateProduct(id, data) { /* ... */ }
  deleteProduct(id) { /* ... */ }
  
  // Order management
  createOrder(data) { /* ... */ }
  processOrder(id) { /* ... */ }
  shipOrder(id) { /* ... */ }
  
  // Database operations
  connectDB() { /* ... */ }
  queryDB() { /* ... */ }
  
  // Server operations
  startServer() { /* ... */ }
  handleRequest() { /* ... */ }
  
  // ... hundreds more methods
}

// ✅ GOOD - Separation of Concerns
class UserService {
  constructor(repository) {
    this.repository = repository;
  }
  
  create(data) { return this.repository.create(data); }
  update(id, data) { return this.repository.update(id, data); }
  delete(id) { return this.repository.delete(id); }
}

class ProductService {
  constructor(repository) {
    this.repository = repository;
  }
  
  create(data) { return this.repository.create(data); }
  update(id, data) { return this.repository.update(id, data); }
  delete(id) { return this.repository.delete(id); }
}

class OrderService {
  constructor(repository, userService, productService) {
    this.repository = repository;
    this.userService = userService;
    this.productService = productService;
  }
  
  create(orderData) { /* ... */ }
  process(orderId) { /* ... */ }
  ship(orderId) { /* ... */ }
}
```

### **2. Spaghetti Code**

**Problem:** Tangled, unstructured code with no clear flow.

```javascript
// ❌ BAD - Spaghetti Code
let total = 0;
for (let i = 0; i < items.length; i++) {
  if (items[i].price) {
    if (items[i].discount) {
      if (items[i].discount > 0) {
        total += items[i].price - (items[i].price * items[i].discount / 100);
      } else {
        total += items[i].price;
      }
    } else {
      total += items[i].price;
    }
  }
}
if (total > 100) {
  total = total - 10;
}

// ✅ GOOD - Clean Structure
class ShoppingCart {
  calculateTotal(items) {
    const subtotal = items.reduce((sum, item) => 
      sum + this.calculateItemPrice(item), 0
    );
    
    return this.applyCartDiscount(subtotal);
  }
  
  calculateItemPrice(item) {
    if (!item.price) return 0;
    
    const discount = item.discount || 0;
    return item.price * (1 - discount / 100);
  }
  
  applyCartDiscount(subtotal) {
    const DISCOUNT_THRESHOLD = 100;
    const DISCOUNT_AMOUNT = 10;
    
    return subtotal > DISCOUNT_THRESHOLD 
      ? subtotal - DISCOUNT_AMOUNT 
      : subtotal;
  }
}
```

### **3. Premature Optimization**

**Problem:** Optimizing before measuring or understanding bottlenecks.

```javascript
// ❌ BAD - Premature Optimization
class DataCache {
  constructor() {
    // Complex caching with LRU, compression, serialization
    this.cache = new Map();
    this.lruList = [];
    this.compressionAlgo = new ComplexCompressor();
    this.serializer = new CustomSerializer();
  }
  
  set(key, value) {
    const compressed = this.compressionAlgo.compress(value);
    const serialized = this.serializer.serialize(compressed);
    this.cache.set(key, serialized);
    this.updateLRU(key);
  }
  
  // ... complex implementation for 10 items
}

// ✅ GOOD - Simple First, Optimize Later
class DataCache {
  constructor() {
    this.cache = new Map();
  }
  
  set(key, value) {
    this.cache.set(key, value);
  }
  
  get(key) {
    return this.cache.get(key);
  }
  
  // Measure performance first, then optimize if needed
}
```

### **4. Overusing Patterns**

**Problem:** Using patterns where simple code suffices.

```javascript
// ❌ BAD - Pattern Overkill
class AdditionStrategy {
  execute(a, b) { return a + b; }
}

class SubtractionStrategy {
  execute(a, b) { return a - b; }
}

class CalculatorContext {
  setStrategy(strategy) {
    this.strategy = strategy;
  }
  
  calculate(a, b) {
    return this.strategy.execute(a, b);
  }
}

const calc = new CalculatorContext();
calc.setStrategy(new AdditionStrategy());
const result = calc.calculate(2, 3);

// ✅ GOOD - Simple Solution
function add(a, b) { return a + b; }
function subtract(a, b) { return a - b; }

const result = add(2, 3);
```

### **5. Singleton Abuse**

**Problem:** Using Singleton when not needed, making testing difficult.

```javascript
// ❌ BAD - Singleton Abuse
class Logger {
  static instance = null;
  
  static getInstance() {
    if (!Logger.instance) {
      Logger.instance = new Logger();
    }
    return Logger.instance;
  }
  
  log(message) {
    console.log(message);
  }
}

// Hard to test, can't mock easily
class UserService {
  createUser(data) {
    Logger.getInstance().log('Creating user');
    // ... create user
  }
}

// ✅ GOOD - Dependency Injection
class Logger {
  log(message) {
    console.log(message);
  }
}

class UserService {
  constructor(logger) {
    this.logger = logger;
  }
  
  createUser(data) {
    this.logger.log('Creating user');
    // ... create user
  }
}

// Easy to test with mock logger
const mockLogger = { log: jest.fn() };
const service = new UserService(mockLogger);
```

---

## **Modern JavaScript Patterns**

### **1. React Component Patterns**

```javascript
// Higher-Order Component (HOC)
function withLoading(Component) {
  return function WithLoadingComponent({ isLoading, ...props }) {
    if (isLoading) {
      return <div>Loading...</div>;
    }
    return <Component {...props} />;
  };
}

const UserListWithLoading = withLoading(UserList);

// Render Props
class DataProvider extends React.Component {
  state = { data: null, loading: true };
  
  componentDidMount() {
    fetch(this.props.url)
      .then(res => res.json())
      .then(data => this.setState({ data, loading: false }));
  }
  
  render() {
    return this.props.render(this.state);
  }
}

// Usage
<DataProvider
  url="/api/users"
  render={({ data, loading }) => (
    loading ? <Spinner /> : <UserList users={data} />
  )}
/>

// Custom Hooks (Modern React)
function useData(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    fetch(url)
      .then(res => res.json())
      .then(data => {
        setData(data);
        setLoading(false);
      })
      .catch(err => {
        setError(err);
        setLoading(false);
      });
  }, [url]);
  
  return { data, loading, error };
}

// Usage
function UserList() {
  const { data, loading, error } = useData('/api/users');
  
  if (loading) return <Spinner />;
  if (error) return <Error message={error.message} />;
  return <List items={data} />;
}

// Compound Components
function Tabs({ children }) {
  const [activeIndex, setActiveIndex] = useState(0);
  
  return (
    <TabsContext.Provider value={{ activeIndex, setActiveIndex }}>
      {children}
    </TabsContext.Provider>
  );
}

Tabs.List = function TabsList({ children }) {
  return <div className="tabs-list">{children}</div>;
};

Tabs.Tab = function Tab({ index, children }) {
  const { activeIndex, setActiveIndex } = useContext(TabsContext);
  return (
    <button
      className={activeIndex === index ? 'active' : ''}
      onClick={() => setActiveIndex(index)}
    >
      {children}
    </button>
  );
};

Tabs.Panel = function TabsPanel({ index, children }) {
  const { activeIndex } = useContext(TabsContext);
  return activeIndex === index ? <div>{children}</div> : null;
};

// Usage
<Tabs>
  <Tabs.List>
    <Tabs.Tab index={0}>Tab 1</Tabs.Tab>
    <Tabs.Tab index={1}>Tab 2</Tabs.Tab>
  </Tabs.List>
  <Tabs.Panel index={0}>Content 1</Tabs.Panel>
  <Tabs.Panel index={1}>Content 2</Tabs.Panel>
</Tabs>
```

### **2. Async Patterns**

```javascript
// Async Iterator Pattern
async function* fetchPages(url, totalPages) {
  for (let page = 1; page <= totalPages; page++) {
    const response = await fetch(`${url}?page=${page}`);
    const data = await response.json();
    yield data;
  }
}

// Usage
for await (const pageData of fetchPages('/api/items', 5)) {
  console.log('Processing page:', pageData);
}

// Promise Pool (Concurrency Control)
class PromisePool {
  constructor(concurrency) {
    this.concurrency = concurrency;
    this.running = 0;
    this.queue = [];
  }
  
  async add(promiseFactory) {
    while (this.running >= this.concurrency) {
      await Promise.race(this.queue);
    }
    
    this.running++;
    const promise = promiseFactory()
      .then(result => {
        this.running--;
        this.queue.splice(this.queue.indexOf(promise), 1);
        return result;
      })
      .catch(error => {
        this.running--;
        this.queue.splice(this.queue.indexOf(promise), 1);
        throw error;
      });
    
    this.queue.push(promise);
    return promise;
  }
  
  async all(promiseFactories) {
    return Promise.all(
      promiseFactories.map(factory => this.add(factory))
    );
  }
}

// Usage
const pool = new PromisePool(3); // Max 3 concurrent requests

const urls = Array.from({ length: 10 }, (_, i) => `/api/item/${i}`);
const results = await pool.all(
  urls.map(url => () => fetch(url).then(r => r.json()))
);

// Retry Pattern with Exponential Backoff
async function retryWithBackoff(fn, maxRetries = 3, baseDelay = 1000) {
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      return await fn();
    } catch (error) {
      if (attempt === maxRetries - 1) {
        throw error;
      }
      
      const delay = baseDelay * Math.pow(2, attempt);
      console.log(`Retry attempt ${attempt + 1} after ${delay}ms`);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
}

// Usage
const data = await retryWithBackoff(
  () => fetch('/api/unstable-endpoint').then(r => r.json()),
  5,
  1000
);

// Circuit Breaker Pattern
class CircuitBreaker {
  constructor(threshold = 5, timeout = 60000) {
    this.threshold = threshold;
    this.timeout = timeout;
    this.failures = 0;
    this.state = 'CLOSED'; // CLOSED, OPEN, HALF_OPEN
    this.nextAttempt = Date.now();
  }
  
  async execute(fn) {
    if (this.state === 'OPEN') {
      if (Date.now() < this.nextAttempt) {
        throw new Error('Circuit breaker is OPEN');
      }
      this.state = 'HALF_OPEN';
    }
    
    try {
      const result = await fn();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }
  
  onSuccess() {
    this.failures = 0;
    this.state = 'CLOSED';
  }
  
  onFailure() {
    this.failures++;
    if (this.failures >= this.threshold) {
      this.state = 'OPEN';
      this.nextAttempt = Date.now() + this.timeout;
      console.log(`Circuit breaker opened. Retry after ${this.timeout}ms`);
    }
  }
}

// Usage
const breaker = new CircuitBreaker(3, 30000);

async function fetchData() {
  return breaker.execute(() =>
    fetch('/api/data').then(r => r.json())
  );
}
```

### **3. Module Patterns (ES6+)**

```javascript
// Named Exports
// math.js
export const PI = 3.14159;
export function add(a, b) { return a + b; }
export function multiply(a, b) { return a * b; }

// app.js
import { PI, add, multiply } from './math.js';

// Default Export
// logger.js
export default class Logger {
  log(message) {
    console.log(`[${new Date().toISOString()}] ${message}`);
  }
}

// app.js
import Logger from './logger.js';

// Re-exports (Barrel Pattern)
// components/index.js
export { Button } from './Button.js';
export { Input } from './Input.js';
export { Card } from './Card.js';

// app.js
import { Button, Input, Card } from './components';

// Dynamic Imports
async function loadModule(moduleName) {
  if (moduleName === 'heavy-module') {
    const module = await import('./heavy-module.js');
    return module.default;
  }
}

// Namespace Pattern
const App = {
  config: {
    apiUrl: 'https://api.example.com',
    timeout: 5000
  },
  
  utils: {
    formatDate(date) { /* ... */ },
    parseJSON(str) { /* ... */ }
  },
  
  services: {
    api: {
      get(endpoint) { /* ... */ },
      post(endpoint, data) { /* ... */ }
    },
    
    auth: {
      login(credentials) { /* ... */ },
      logout() { /* ... */ }
    }
  }
};
```

---

## **Best Practices**

### **1. SOLID Principles Applied to Patterns**

- **Single Responsibility**: Each pattern solves one specific problem
- **Open/Closed**: Patterns allow extension without modification
- **Liskov Substitution**: Strategy and State patterns ensure interchangeable components
- **Interface Segregation**: Use small, focused interfaces
- **Dependency Inversion**: Depend on abstractions, not concretions

### **2. When to Use Patterns**

✅ **DO use patterns when:**
- Code is becoming complex and hard to maintain
- You recognize a recurring problem
- You need flexibility for future changes
- Multiple team members need to understand the code
- You're building a framework or library

❌ **DON'T use patterns when:**
- A simple solution exists
- The problem is unlikely to change
- It's a one-time script
- The pattern adds unnecessary complexity
- You're just learning patterns

### **3. Pattern Selection Checklist**

1. **Identify the problem**: What exactly are you trying to solve?
2. **Check simpler solutions**: Can vanilla JS solve it?
3. **Consider maintenance**: Will others understand this?
4. **Think about testing**: Does the pattern make testing easier or harder?
5. **Evaluate performance**: Does the abstraction add significant overhead?

### **4. Code Quality Guidelines**

```javascript
// ✅ Good Pattern Implementation
class PaymentProcessor {
  constructor(strategy) {
    this.strategy = strategy; // Dependency injection
  }
  
  // Clear, single purpose
  process(amount) {
    this.validateAmount(amount);
    return this.strategy.execute(amount);
  }
  
  // Easy to test
  validateAmount(amount) {
    if (amount <= 0) {
      throw new Error('Amount must be positive');
    }
  }
  
  // Allows strategy changes
  setStrategy(strategy) {
    this.strategy = strategy;
  }
}

// ✅ Good usage - Clear and testable
const processor = new PaymentProcessor(new CreditCardStrategy());
processor.process(100);

// Testing is straightforward
const mockStrategy = { execute: jest.fn() };
const testProcessor = new PaymentProcessor(mockStrategy);
```

---

## **Comprehensive Key Takeaways**

### **Pattern Categories Summary**

**Creational Patterns** - Focus on object creation:
- **Module**: Encapsulation and privacy
- **Singleton**: Single instance control
- **Factory**: Flexible object creation
- **Builder**: Step-by-step construction
- **Prototype**: Clone existing objects

**Structural Patterns** - Focus on object composition:
- **Adapter**: Interface compatibility
- **Decorator**: Add behavior dynamically
- **Facade**: Simplify complex systems
- **Composite**: Tree structures
- **Proxy**: Access control and lazy loading

**Behavioral Patterns** - Focus on object interaction:
- **Observer**: Event-driven architecture
- **Strategy**: Algorithm selection
- **Command**: Encapsulate operations
- **State**: Behavior based on state
- **Chain of Responsibility**: Request processing pipeline
- **Iterator**: Sequential access
- **Mediator**: Centralized communication
- **Template Method**: Algorithm skeleton

### **Essential Principles**

1. **Favor Composition Over Inheritance**
   - Use patterns like Decorator, Strategy, and Composite
   - More flexible and maintainable

2. **Program to Interfaces, Not Implementations**
   - Patterns like Strategy and Factory promote this
   - Easier to swap implementations

3. **Encapsulate What Varies**
   - Identify aspects that change and separate them
   - Strategy, State, and Builder patterns excel here

4. **Depend on Abstractions**
   - High-level modules shouldn't depend on low-level modules
   - Use Factory, Adapter, and Proxy patterns

5. **Keep It Simple**
   - Don't use patterns just because you know them
   - Choose simplicity over cleverness

### **Pattern Selection Quick Reference**

**Need to create objects?** → Creational Patterns
- One instance? → **Singleton**
- Many types? → **Factory**
- Complex setup? → **Builder**
- Copy existing? → **Prototype**

**Need to structure objects?** → Structural Patterns
- Incompatible interfaces? → **Adapter**
- Add features? → **Decorator**
- Simplify complexity? → **Facade**
- Tree structures? → **Composite**
- Control access? → **Proxy**

**Need objects to interact?** → Behavioral Patterns
- Event system? → **Observer**
- Multiple algorithms? → **Strategy**
- Undo/redo? → **Command**
- State-dependent behavior? → **State**
- Request pipeline? → **Chain of Responsibility**
- Custom iteration? → **Iterator**
- Complex communication? → **Mediator**
- Algorithm steps? → **Template Method**

### **Modern JavaScript Context**

- ES6+ provides native solutions: classes, modules, promises, async/await
- React promotes: HOCs, Render Props, Custom Hooks, Compound Components
- Node.js favors: Middleware pattern, Event Emitters, Streams
- Always consider framework-specific patterns before reinventing

### **Final Wisdom**

> "Design patterns are not a silver bullet. They are tools in your toolbox. Use them when they solve a real problem, not because they look elegant." 

**Remember:**
- Patterns make code more maintainable, not more clever
- Simple code is better than pattern-heavy code
- Understand the problem before applying a solution
- Patterns should emerge from needs, not be forced into designs
- Testing and clarity always trump cleverness

---
