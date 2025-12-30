## Objects & Prototypes

29. What are objects in JavaScript?

**Answer:**
Objects are JavaScript's fundamental data structure for storing collections of key-value pairs. They are the building blocks of JavaScript and nearly everything in JavaScript is an object or behaves like one (except primitives).

**What is an Object?**

An object is a collection of **properties**, where each property is an association between a **key** (string or Symbol) and a **value** (any type).

```javascript
// Simple object
const person = {
  name: 'John',
  age: 30,
  email: 'john@example.com'
};

// Accessing properties
console.log(person.name);       // 'John' (dot notation)
console.log(person['age']);     // 30 (bracket notation)
```

**Object Characteristics:**

**1. Key-Value Pairs:**
```javascript
const user = {
  // String keys
  firstName: 'John',
  'last-name': 'Doe',  // Keys with special chars need quotes
  
  // Number keys (converted to strings)
  1: 'one',
  2: 'two',
  
  // Values can be any type
  age: 30,              // Number
  isActive: true,       // Boolean
  hobbies: ['coding'],  // Array
  address: {            // Nested object
    city: 'New York'
  },
  greet: function() {   // Function (method)
    return `Hello, ${this.firstName}`;
  }
};

console.log(user[1]);           // 'one'
console.log(user['last-name']); // 'Doe'
console.log(user.greet());      // 'Hello, John'
```

**2. Dynamic Nature:**
```javascript
const obj = {};

// Add properties dynamically
obj.name = 'John';
obj['age'] = 30;
obj.greet = function() { return 'Hello'; };

// Modify properties
obj.name = 'Jane';

// Delete properties
delete obj.age;

console.log(obj); // { name: 'Jane', greet: [Function] }
```

**3. Reference Type:**
```javascript
// Objects are passed by reference
const original = { count: 0 };
const reference = original;

reference.count = 10;
console.log(original.count); // 10 (both point to same object)

// Comparison by reference
const a = { value: 1 };
const b = { value: 1 };
console.log(a === b);        // false (different objects)

const c = a;
console.log(a === c);        // true (same reference)
```

**Property Access Methods:**

```javascript
const person = {
  name: 'John',
  age: 30
};

// Dot notation (preferred for valid identifiers)
console.log(person.name);

// Bracket notation (required for special cases)
console.log(person['name']);

// Dynamic property access
const prop = 'age';
console.log(person[prop]); // 30

// Computed property names (ES6)
const dynamicKey = 'email';
const user = {
  [dynamicKey]: 'john@example.com',
  [`${dynamicKey}Verified`]: true
};
console.log(user.email);         // 'john@example.com'
console.log(user.emailVerified); // true
```

**Methods in Objects:**

```javascript
const calculator = {
  value: 0,
  
  // Method: function property
  add: function(n) {
    this.value += n;
    return this;
  },
  
  // ES6 method shorthand
  subtract(n) {
    this.value -= n;
    return this;
  },
  
  // Arrow function (doesn't have own 'this')
  multiply: (n) => {
    // this.value *= n; // ❌ 'this' doesn't refer to calculator
  },
  
  getValue() {
    return this.value;
  }
};

calculator.add(5).subtract(2); // Method chaining
console.log(calculator.getValue()); // 3
```

**Everything is an Object (Almost):**

```javascript
// Primitives have object wrappers
const str = 'hello';
console.log(str.toUpperCase()); // 'HELLO' (temporary String object)
console.log(str.length);        // 5

// Arrays are objects
const arr = [1, 2, 3];
console.log(typeof arr);        // 'object'
console.log(Array.isArray(arr)); // true

// Functions are objects
function greet() {}
greet.customProperty = 'value';
console.log(typeof greet);      // 'function' (special object)
console.log(greet.customProperty); // 'value'

// null is special
console.log(typeof null);       // 'object' (historical bug)
```

**Built-in Object Types:**

```javascript
// Object literal
const obj = {};

// Array
const arr = [1, 2, 3];

// Function
const func = function() {};

// Date
const date = new Date();

// RegExp
const regex = /ab+c/;

// Map (ES6)
const map = new Map();

// Set (ES6)
const set = new Set();

// Error
const error = new Error('Something went wrong');
```

**Production Use Cases:**

**1. Data Modeling:**
```javascript
const product = {
  id: 1001,
  name: 'Laptop',
  price: 999.99,
  category: 'Electronics',
  inStock: true,
  specifications: {
    cpu: 'Intel i7',
    ram: '16GB',
    storage: '512GB SSD'
  },
  tags: ['computer', 'portable', 'work'],
  
  getDisplayPrice() {
    return `$${this.price.toFixed(2)}`;
  },
  
  applyDiscount(percent) {
    this.price *= (1 - percent / 100);
    return this;
  }
};
```

**2. Configuration Objects:**
```javascript
const apiConfig = {
  baseURL: 'https://api.example.com',
  timeout: 5000,
  headers: {
    'Content-Type': 'application/json',
    'Authorization': 'Bearer token'
  },
  retries: 3,
  
  request(endpoint, options = {}) {
    const url = `${this.baseURL}${endpoint}`;
    return fetch(url, {
      ...options,
      timeout: this.timeout,
      headers: { ...this.headers, ...options.headers }
    });
  }
};
```

**3. State Management:**
```javascript
const appState = {
  user: null,
  isAuthenticated: false,
  theme: 'light',
  notifications: [],
  
  login(user) {
    this.user = user;
    this.isAuthenticated = true;
    this.notify('Login successful');
  },
  
  logout() {
    this.user = null;
    this.isAuthenticated = false;
    this.notify('Logged out');
  },
  
  notify(message) {
    this.notifications.push({
      message,
      timestamp: Date.now()
    });
  }
};
```

**4. Namespacing:**
```javascript
// Avoid global pollution
const MyApp = {
  config: {
    version: '1.0.0',
    apiUrl: 'https://api.example.com'
  },
  
  utils: {
    formatDate(date) { /* ... */ },
    validateEmail(email) { /* ... */ }
  },
  
  services: {
    userService: {
      getUser() { /* ... */ },
      updateUser() { /* ... */ }
    },
    authService: {
      login() { /* ... */ },
      logout() { /* ... */ }
    }
  },
  
  init() {
    console.log(`App ${this.config.version} initialized`);
  }
};

MyApp.init();
```

**5. Options/Settings Pattern:**
```javascript
function createSlider(element, options = {}) {
  // Merge with defaults
  const settings = {
    autoplay: false,
    speed: 300,
    infinite: true,
    arrows: true,
    dots: true,
    ...options  // Override defaults
  };
  
  return {
    element,
    settings,
    
    play() {
      if (this.settings.autoplay) {
        // Start autoplay
      }
    },
    
    next() {
      // Go to next slide
    }
  };
}

const slider = createSlider('.slider', {
  autoplay: true,
  speed: 500
});
```

**Object Literal Enhancements (ES6+):**

```javascript
const name = 'John';
const age = 30;

// Property shorthand
const person = {
  name,    // Same as name: name
  age      // Same as age: age
};

// Method shorthand
const obj = {
  // Old way
  greet: function() {
    return 'Hello';
  },
  
  // New way
  greet() {
    return 'Hello';
  }
};

// Computed property names
const propName = 'score';
const game = {
  [propName]: 100,
  [`${propName}Multiplier`]: 2
};

console.log(game.score);           // 100
console.log(game.scoreMultiplier); // 2
```

**Checking Properties:**

```javascript
const user = {
  name: 'John',
  age: 30
};

// Check if property exists
console.log('name' in user);          // true
console.log('email' in user);         // false

// hasOwnProperty (checks own properties, not inherited)
console.log(user.hasOwnProperty('name'));     // true
console.log(user.hasOwnProperty('toString')); // false (inherited)

// Check if property value is undefined
console.log(user.name !== undefined);  // true
console.log(user.email !== undefined); // false

// Optional chaining (ES2020)
console.log(user?.email?.toLowerCase()); // undefined (no error)
```

**Key Takeaways:**
- Objects are collections of key-value pairs (properties)
- Keys are strings or Symbols, values can be any type
- Objects are reference types (passed by reference)
- Nearly everything in JavaScript is an object
- Objects are mutable and dynamic
- Essential for data modeling, configuration, and state management
- Foundation of object-oriented programming in JavaScript
- Understanding objects is crucial for mastering JavaScript

30. What are the different ways to create objects?

**Answer:**
JavaScript provides multiple ways to create objects, each with its own use cases and advantages. Understanding these methods is essential for effective object-oriented programming in JavaScript.

**1. Object Literal (Most Common):**

The simplest and most direct way to create objects.

```javascript
// Empty object
const emptyObj = {};

// Object with properties
const person = {
  name: 'John',
  age: 30,
  greet() {
    return `Hello, I'm ${this.name}`;
  }
};

console.log(person.name);  // 'John'
console.log(person.greet()); // "Hello, I'm John"

// Nested objects
const user = {
  id: 1,
  name: 'John',
  address: {
    street: '123 Main St',
    city: 'New York',
    country: 'USA'
  },
  settings: {
    theme: 'dark',
    notifications: true
  }
};
```

**When to use:** Quick object creation, configuration objects, single instances, JSON-like data structures.

**2. Object Constructor (new Object()):**

Less common, but useful to understand.

```javascript
// Using Object constructor
const person = new Object();
person.name = 'John';
person.age = 30;
person.greet = function() {
  return `Hello, I'm ${this.name}`;
};

// Equivalent to object literal
const person2 = {
  name: 'John',
  age: 30,
  greet() {
    return `Hello, I'm ${this.name}`;
  }
};

console.log(person.name); // 'John'
```

**When to use:** Rarely used; object literals are preferred. Sometimes useful when the type is determined dynamically.

**3. Constructor Function (Pre-ES6 Classes):**

Pattern for creating multiple objects with the same structure.

```javascript
// Constructor function (capitalize first letter by convention)
function Person(name, age) {
  this.name = name;
  this.age = age;
  this.greet = function() {
    return `Hello, I'm ${this.name}`;
  };
}

// Create instances with 'new' keyword
const john = new Person('John', 30);
const jane = new Person('Jane', 25);

console.log(john.name);   // 'John'
console.log(jane.greet()); // "Hello, I'm Jane"

// Adding methods to prototype (more efficient)
Person.prototype.sayAge = function() {
  return `I am ${this.age} years old`;
};

console.log(john.sayAge()); // "I am 30 years old"

// Check instance
console.log(john instanceof Person); // true
```

**When to use:** Creating multiple instances with shared behavior, before ES6 classes, understanding JavaScript's prototypal inheritance.

**4. ES6 Classes (Modern Approach):**

Syntactic sugar over constructor functions, providing cleaner syntax.

```javascript
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
  
  // Methods are added to prototype automatically
  greet() {
    return `Hello, I'm ${this.name}`;
  }
  
  sayAge() {
    return `I am ${this.age} years old`;
  }
  
  // Static method
  static species() {
    return 'Homo sapiens';
  }
  
  // Getter
  get info() {
    return `${this.name}, ${this.age}`;
  }
  
  // Setter
  set age(value) {
    if (value < 0) throw new Error('Age must be positive');
    this._age = value;
  }
}

const john = new Person('John', 30);
console.log(john.greet());        // "Hello, I'm John"
console.log(Person.species());    // 'Homo sapiens'

// Inheritance
class Employee extends Person {
  constructor(name, age, jobTitle) {
    super(name, age); // Call parent constructor
    this.jobTitle = jobTitle;
  }
  
  // Override method
  greet() {
    return `${super.greet()}, I'm a ${this.jobTitle}`;
  }
}

const emp = new Employee('Alice', 28, 'Developer');
console.log(emp.greet()); // "Hello, I'm Alice, I'm a Developer"
```

**When to use:** Modern object-oriented programming, inheritance, cleaner syntax, production applications.

**5. Object.create() (Prototypal Inheritance):**

Creates a new object with specified prototype.

```javascript
// Create object with specific prototype
const personPrototype = {
  greet() {
    return `Hello, I'm ${this.name}`;
  },
  sayAge() {
    return `I am ${this.age} years old`;
  }
};

// Create object inheriting from personPrototype
const john = Object.create(personPrototype);
john.name = 'John';
john.age = 30;

console.log(john.greet()); // "Hello, I'm John"

// With properties
const jane = Object.create(personPrototype, {
  name: {
    value: 'Jane',
    writable: true,
    enumerable: true,
    configurable: true
  },
  age: {
    value: 25,
    writable: true,
    enumerable: true,
    configurable: true
  }
});

console.log(jane.sayAge()); // "I am 25 years old"

// Create object with null prototype (no inherited properties)
const pureObj = Object.create(null);
pureObj.name = 'Pure';
console.log(pureObj.toString); // undefined (no inherited methods)
```

**When to use:** Prototypal inheritance, avoiding constructor overhead, creating objects without `Object.prototype` methods.

**6. Factory Functions:**

Functions that return new objects without using `new`.

```javascript
// Simple factory
function createPerson(name, age) {
  return {
    name,
    age,
    greet() {
      return `Hello, I'm ${this.name}`;
    }
  };
}

const john = createPerson('John', 30);
const jane = createPerson('Jane', 25);

console.log(john.greet()); // "Hello, I'm John"

// Factory with private variables (closure)
function createCounter(initialValue = 0) {
  let count = initialValue; // Private variable
  
  return {
    increment() {
      count++;
      return count;
    },
    decrement() {
      count--;
      return count;
    },
    getCount() {
      return count;
    }
  };
}

const counter = createCounter(10);
console.log(counter.increment()); // 11
console.log(counter.getCount());  // 11
// counter.count; // undefined (truly private)

// Factory with validation
function createUser(name, email) {
  if (!email.includes('@')) {
    throw new Error('Invalid email');
  }
  
  return {
    name,
    email,
    toString() {
      return `${this.name} <${this.email}>`;
    }
  };
}

const user = createUser('John', 'john@example.com');
```

**When to use:** Encapsulation with closures, avoiding `new` keyword, flexible object creation, private variables.

**7. Object.assign() (Cloning/Merging):**

Creates a new object by copying properties.

```javascript
// Clone object
const original = { name: 'John', age: 30 };
const clone = Object.assign({}, original);

clone.age = 31;
console.log(original.age); // 30 (not affected)

// Merge objects
const defaults = { theme: 'light', language: 'en' };
const userPrefs = { theme: 'dark' };
const settings = Object.assign({}, defaults, userPrefs);

console.log(settings); // { theme: 'dark', language: 'en' }

// Modern alternative: Spread operator
const settings2 = { ...defaults, ...userPrefs };
console.log(settings2); // { theme: 'dark', language: 'en' }
```

**When to use:** Cloning objects, merging configurations, creating objects from existing ones.

**8. Spread Operator (ES6+):**

Modern way to clone and merge objects.

```javascript
// Clone object
const original = { name: 'John', age: 30 };
const clone = { ...original };

// Merge objects
const user = { name: 'John' };
const details = { age: 30, email: 'john@example.com' };
const complete = { ...user, ...details };

console.log(complete); // { name: 'John', age: 30, email: 'john@example.com' }

// Override properties
const defaults = { theme: 'light', size: 'medium' };
const custom = { ...defaults, theme: 'dark' };

console.log(custom); // { theme: 'dark', size: 'medium' }

// Add properties
const enhanced = {
  ...original,
  city: 'New York',
  getInfo() {
    return `${this.name} from ${this.city}`;
  }
};
```

**When to use:** Modern codebases, cloning, merging, adding properties to existing objects.

**Comparison Table:**

| Method | Use Case | Prototype | Private Data | Performance |
|--------|----------|-----------|--------------|-------------|
| **Object Literal** | Simple objects, configs | Object.prototype | No | Fast |
| **new Object()** | Rarely used | Object.prototype | No | Fast |
| **Constructor Function** | Multiple instances | Custom | No | Fast |
| **ES6 Class** | OOP, inheritance | Custom | No | Fast |
| **Object.create()** | Custom prototype | Custom | No | Fast |
| **Factory Function** | Encapsulation | Object.prototype | Yes (closure) | Slower |
| **Object.assign()** | Cloning, merging | Source's prototype | No | Medium |
| **Spread Operator** | Modern cloning | Source's prototype | No | Medium |

**Production Examples:**

**Use Case 1: API Response Objects**
```javascript
// Object literal for simple data
const apiResponse = {
  status: 200,
  data: {
    users: [],
    total: 0
  },
  error: null
};
```

**Use Case 2: Creating Multiple Similar Objects**
```javascript
// Class for multiple instances
class Product {
  constructor(name, price, category) {
    this.name = name;
    this.price = price;
    this.category = category;
  }
  
  getDisplayPrice() {
    return `$${this.price.toFixed(2)}`;
  }
}

const products = [
  new Product('Laptop', 999.99, 'Electronics'),
  new Product('Mouse', 29.99, 'Accessories'),
  new Product('Keyboard', 79.99, 'Accessories')
];
```

**Use Case 3: Private State Management**
```javascript
// Factory function with closures
function createWallet(initialBalance) {
  let balance = initialBalance;
  const transactions = [];
  
  return {
    deposit(amount) {
      balance += amount;
      transactions.push({ type: 'deposit', amount, date: new Date() });
    },
    withdraw(amount) {
      if (amount <= balance) {
        balance -= amount;
        transactions.push({ type: 'withdraw', amount, date: new Date() });
      }
    },
    getBalance() {
      return balance;
    }
  };
}
```

**Use Case 4: Configuration with Defaults**
```javascript
// Spread operator for merging
function initializeApp(userConfig = {}) {
  const defaultConfig = {
    apiUrl: 'https://api.example.com',
    timeout: 5000,
    retries: 3,
    debug: false
  };
  
  const config = { ...defaultConfig, ...userConfig };
  return config;
}
```

**Best Practices:**
- Use **object literals** for simple, one-off objects
- Use **classes** for OOP and multiple instances in modern code
- Use **factory functions** when you need private data via closures
- Use **Object.create()** for prototypal inheritance without constructors
- Use **spread operator** for cloning and merging in modern code
- Avoid `new Object()` - use object literals instead
- Choose based on needs: simplicity, inheritance, encapsulation, or performance

31. What is the prototype chain?

**Answer:**
The prototype chain is a mechanism that allows objects to inherit properties and methods from other objects. When you try to access a property on an object, JavaScript first looks on the object itself, then on its prototype, then on the prototype's prototype, and so on until it reaches `null`.

**How It Works:**

```javascript
const animal = {
  eats: true,
  walk() {
    console.log('Animal walks');
  }
};

const rabbit = {
  jumps: true
};

// Set animal as prototype of rabbit
rabbit.__proto__ = animal; // Don't use this in production! Use Object.create()

console.log(rabbit.jumps);  // true (own property)
console.log(rabbit.eats);   // true (inherited from animal)
rabbit.walk();              // 'Animal walks' (inherited method)

console.log(rabbit.toString()); // [object Object] (inherited from Object.prototype)
```

**The Chain Lookup:**

```javascript
// When accessing rabbit.walk():
// 1. Check rabbit object itself - not found
// 2. Check rabbit.__proto__ (animal) - found! Execute it
// 3. If not found, check animal.__proto__ (Object.prototype) - would check here
// 4. If not found, check Object.prototype.__proto__ (null) - undefined

const obj = {
  a: 1
};

// Prototype chain: obj → Object.prototype → null
console.log(obj.toString()); // From Object.prototype
console.log(obj.hasOwnProperty('a')); // From Object.prototype
```

**Visual Representation:**

```
rabbit object
  |
  | [[Prototype]] / __proto__
  ↓
animal object
  |
  | [[Prototype]] / __proto__
  ↓
Object.prototype
  |
  | [[Prototype]]
  ↓
null (end of chain)
```

**Constructor Functions and Prototype Chain:**

```javascript
function Person(name) {
  this.name = name;
}

Person.prototype.greet = function() {
  return `Hello, I'm ${this.name}`;
};

const john = new Person('John');

// Prototype chain: john → Person.prototype → Object.prototype → null
console.log(john.name);          // 'John' (own property)
console.log(john.greet());       // 'Hello, I'm John' (from Person.prototype)
console.log(john.toString());    // [object Object] (from Object.prototype)
console.log(john.hasOwnProperty('name')); // true (from Object.prototype)

// Checking the chain
console.log(john.__proto__ === Person.prototype);              // true
console.log(Person.prototype.__proto__ === Object.prototype);  // true
console.log(Object.prototype.__proto__);                       // null
```

**ES6 Classes and Prototype Chain:**

```javascript
class Animal {
  constructor(name) {
    this.name = name;
  }
  
  speak() {
    return `${this.name} makes a sound`;
  }
}

class Dog extends Animal {
  constructor(name, breed) {
    super(name);
    this.breed = breed;
  }
  
  speak() {
    return `${this.name} barks`;
  }
  
  fetch() {
    return `${this.name} fetches the ball`;
  }
}

const myDog = new Dog('Rex', 'Labrador');

// Prototype chain: myDog → Dog.prototype → Animal.prototype → Object.prototype → null
console.log(myDog.name);         // 'Rex' (own property)
console.log(myDog.breed);        // 'Labrador' (own property)
console.log(myDog.fetch());      // 'Rex fetches the ball' (from Dog.prototype)
console.log(myDog.speak());      // 'Rex barks' (from Dog.prototype, overridden)
console.log(myDog.toString());   // [object Object] (from Object.prototype)

// Verify the chain
console.log(myDog.__proto__ === Dog.prototype);                    // true
console.log(Dog.prototype.__proto__ === Animal.prototype);         // true
console.log(Animal.prototype.__proto__ === Object.prototype);      // true
console.log(Object.prototype.__proto__);                           // null
```

**Checking Prototype Chain:**

```javascript
function Person(name) {
  this.name = name;
}

const john = new Person('John');

// instanceof checks prototype chain
console.log(john instanceof Person);     // true
console.log(john instanceof Object);     // true
console.log(john instanceof Array);      // false

// isPrototypeOf() checks if object is in prototype chain
console.log(Person.prototype.isPrototypeOf(john));    // true
console.log(Object.prototype.isPrototypeOf(john));    // true

// getPrototypeOf() returns the prototype
console.log(Object.getPrototypeOf(john) === Person.prototype); // true
```

**Property Lookup Performance:**

```javascript
const obj = {
  prop: 'own property'
};

// Fast - found immediately
console.log(obj.prop);

// Slower - must traverse prototype chain
console.log(obj.toString());

// Even slower - must check multiple levels
class A {}
class B extends A {}
class C extends B {}
class D extends C {}

const instance = new D();
// Deep prototype chain = slower lookups
```

**Shadowing Properties:**

```javascript
const animal = {
  eats: true
};

const rabbit = Object.create(animal);
rabbit.eats = false; // Shadows animal.eats

console.log(rabbit.eats);      // false (own property)
console.log(animal.eats);      // true (unchanged)

delete rabbit.eats;
console.log(rabbit.eats);      // true (now uses inherited property)
```

**Production Use Cases:**

**1. Method Sharing (Memory Efficiency):**
```javascript
// ❌ Bad - each instance has its own method copy
function Person(name) {
  this.name = name;
  this.greet = function() { // New function for each instance!
    return `Hello, ${this.name}`;
  };
}

const person1 = new Person('John');
const person2 = new Person('Jane');
console.log(person1.greet === person2.greet); // false (different functions)

// ✅ Good - methods shared via prototype
function PersonOptimized(name) {
  this.name = name;
}

PersonOptimized.prototype.greet = function() {
  return `Hello, ${this.name}`;
};

const person3 = new PersonOptimized('John');
const person4 = new PersonOptimized('Jane');
console.log(person3.greet === person4.greet); // true (same function)
```

**2. Extending Built-in Objects (Use Caution):**
```javascript
// Adding method to all arrays (generally not recommended)
Array.prototype.last = function() {
  return this[this.length - 1];
};

const arr = [1, 2, 3];
console.log(arr.last()); // 3

// Better: Use utility functions or extend in controlled way
class MyArray extends Array {
  last() {
    return this[this.length - 1];
  }
}

const myArr = new MyArray(1, 2, 3);
console.log(myArr.last()); // 3
```

**3. Polyfills:**
```javascript
// Add missing methods to older browsers via prototype chain
if (!Array.prototype.includes) {
  Array.prototype.includes = function(element) {
    return this.indexOf(element) !== -1;
  };
}

// Now all arrays have .includes()
[1, 2, 3].includes(2); // true
```

**4. Creating Object Hierarchies:**
```javascript
// Base class
class Vehicle {
  constructor(brand) {
    this.brand = brand;
  }
  
  start() {
    return `${this.brand} vehicle started`;
  }
}

// Intermediate class
class Car extends Vehicle {
  constructor(brand, model) {
    super(brand);
    this.model = model;
  }
  
  drive() {
    return `Driving ${this.brand} ${this.model}`;
  }
}

// Specific class
class ElectricCar extends Car {
  constructor(brand, model, batteryLife) {
    super(brand, model);
    this.batteryLife = batteryLife;
  }
  
  charge() {
    return `Charging ${this.brand} ${this.model}`;
  }
}

const tesla = new ElectricCar('Tesla', 'Model 3', 100);

// Prototype chain: tesla → ElectricCar.prototype → Car.prototype → Vehicle.prototype → Object.prototype → null
console.log(tesla.charge());  // From ElectricCar.prototype
console.log(tesla.drive());   // From Car.prototype
console.log(tesla.start());   // From Vehicle.prototype
console.log(tesla.toString()); // From Object.prototype
```

**Modifying Prototype Chain:**

```javascript
// Get prototype
const proto = Object.getPrototypeOf(obj);

// Set prototype (prefer Object.create over this)
Object.setPrototypeOf(obj, prototype);

// Create with specific prototype
const obj = Object.create(prototype);

// Check if object has own property (not inherited)
obj.hasOwnProperty('propertyName');

// Get all own properties (not inherited)
Object.keys(obj);
Object.getOwnPropertyNames(obj);
```

**Common Pitfalls:**

**1. Modifying Object.prototype (Never Do This):**
```javascript
// ❌ VERY BAD - affects ALL objects
Object.prototype.myMethod = function() {
  return 'bad idea';
};

const obj = {};
console.log(obj.myMethod()); // 'bad idea' - unintended side effect

// Breaks for...in loops
for (let key in obj) {
  console.log(key); // Logs 'myMethod' unexpectedly
}
```

**2. Circular Prototype Chain:**
```javascript
const obj1 = {};
const obj2 = {};

// ❌ This would create circular reference (throws error)
// Object.setPrototypeOf(obj1, obj2);
// Object.setPrototypeOf(obj2, obj1); // TypeError
```

**3. Performance Issues with Long Chains:**
```javascript
// ⚠️ Deep inheritance can be slow
class Level1 {}
class Level2 extends Level1 {}
class Level3 extends Level2 {}
class Level4 extends Level3 {}
class Level5 extends Level4 {}

const instance = new Level5();
// Property lookups traverse entire chain
```

**Debugging Prototype Chain:**

```javascript
function printPrototypeChain(obj) {
  let current = obj;
  let level = 0;
  
  while (current !== null) {
    console.log(`Level ${level}:`, current.constructor?.name || 'Object');
    current = Object.getPrototypeOf(current);
    level++;
  }
  console.log('End of chain (null)');
}

class Animal {}
class Dog extends Animal {}
const myDog = new Dog();

printPrototypeChain(myDog);
// Level 0: Dog
// Level 1: Animal
// Level 2: Object
// End of chain (null)
```

**Best Practices:**
- Use prototype for shared methods (memory efficiency)
- Keep prototype chains shallow for performance
- Never modify `Object.prototype`
- Use `hasOwnProperty()` to check own properties vs inherited
- Prefer `Object.create()` over direct `__proto__` manipulation
- Use classes for cleaner syntax (they use prototypes under the hood)
- Understand that nearly all JavaScript inheritance is prototypal
- Use `instanceof` carefully (breaks across different execution contexts)

**Key Takeaways:**
- The prototype chain is how JavaScript implements inheritance
- Property lookup traverses up the chain until found or reaching `null`
- Every object has a prototype (except the root)
- Prototypes enable method sharing and memory efficiency
- Understanding the chain is essential for OOP in JavaScript
- Modern classes are syntactic sugar over the prototype chain

32. What is prototypal inheritance?

**Answer:**
Prototypal inheritance is JavaScript's mechanism where objects inherit properties and methods directly from other objects. Unlike classical inheritance (found in Java, C++), JavaScript objects inherit from other objects, not from classes (although ES6 classes are syntactic sugar over prototypal inheritance).

**Core Concept:**

In prototypal inheritance, objects can be created from other objects directly, and they "inherit" properties by linking to the parent object through the prototype chain.

```javascript
// Parent object (prototype)
const animal = {
  eats: true,
  sleep() {
    console.log('Sleeping...');
  }
};

// Child object inheriting from animal
const rabbit = Object.create(animal);
rabbit.jumps = true;

console.log(rabbit.eats);   // true (inherited)
console.log(rabbit.jumps);  // true (own property)
rabbit.sleep();             // 'Sleeping...' (inherited method)

// Rabbit inherits from animal
console.log(Object.getPrototypeOf(rabbit) === animal); // true
```

**Methods of Implementing Prototypal Inheritance:**

**1. Object.create() (Most Direct):**
```javascript
const parent = {
  greet() {
    return `Hello from ${this.name}`;
  }
};

const child = Object.create(parent);
child.name = 'Child';

console.log(child.greet()); // 'Hello from Child'

// With initial properties
const child2 = Object.create(parent, {
  name: {
    value: 'Child2',
    writable: true,
    enumerable: true
  }
});
```

**2. Constructor Functions (Pre-ES6):**
```javascript
// Parent constructor
function Animal(name) {
  this.name = name;
}

Animal.prototype.eat = function() {
  return `${this.name} is eating`;
};

// Child constructor
function Dog(name, breed) {
  Animal.call(this, name); // Call parent constructor
  this.breed = breed;
}

// Set up inheritance
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;

// Add child-specific method
Dog.prototype.bark = function() {
  return `${this.name} barks!`;
};

const myDog = new Dog('Rex', 'Labrador');
console.log(myDog.eat());  // 'Rex is eating' (inherited)
console.log(myDog.bark()); // 'Rex barks!' (own method)

console.log(myDog instanceof Dog);    // true
console.log(myDog instanceof Animal); // true
```

**3. ES6 Classes (Modern Syntax):**
```javascript
// Parent class
class Animal {
  constructor(name) {
    this.name = name;
  }
  
  eat() {
    return `${this.name} is eating`;
  }
  
  sleep() {
    return `${this.name} is sleeping`;
  }
}

// Child class
class Dog extends Animal {
  constructor(name, breed) {
    super(name); // Call parent constructor
    this.breed = breed;
  }
  
  bark() {
    return `${this.name} barks!`;
  }
  
  // Override parent method
  eat() {
    return `${super.eat()} dog food`;
  }
}

const myDog = new Dog('Rex', 'Labrador');
console.log(myDog.eat());   // 'Rex is eating dog food'
console.log(myDog.bark());  // 'Rex barks!'
console.log(myDog.sleep()); // 'Rex is sleeping' (inherited)
```

**Prototypal vs Classical Inheritance:**

| Aspect | Prototypal (JavaScript) | Classical (Java, C++) |
|--------|------------------------|---------------------|
| **Inherit From** | Objects | Classes |
| **Flexibility** | Very flexible, dynamic | More rigid, compile-time |
| **Syntax** | Object.create(), prototype | class, extends |
| **Multiple Inheritance** | Via mixins | Not supported (usually) |
| **Property Lookup** | Prototype chain | Class hierarchy |
| **Memory** | Methods shared via prototype | Methods in class definition |

**Advanced Prototypal Patterns:**

**1. Multiple Inheritance (Mixins):**
```javascript
// Mixin pattern (JavaScript's way of multiple inheritance)
const canEat = {
  eat() {
    return `${this.name} is eating`;
  }
};

const canWalk = {
  walk() {
    return `${this.name} is walking`;
  }
};

const canSwim = {
  swim() {
    return `${this.name} is swimming`;
  }
};

// Combine multiple behaviors
class Animal {
  constructor(name) {
    this.name = name;
  }
}

// Mix in multiple behaviors
Object.assign(Animal.prototype, canEat, canWalk);

class Fish extends Animal {
  constructor(name) {
    super(name);
    Object.assign(Fish.prototype, canSwim);
  }
}

const salmon = new Fish('Salmon');
console.log(salmon.eat());  // 'Salmon is eating'
console.log(salmon.swim()); // 'Salmon is swimming'

// Modern mixin pattern
function mixin(...mixins) {
  return function(BaseClass) {
    mixins.forEach(mixin => {
      Object.assign(BaseClass.prototype, mixin);
    });
    return BaseClass;
  };
}

@mixin(canEat, canWalk, canSwim)
class Duck extends Animal {
  constructor(name) {
    super(name);
  }
}
```

**2. Delegation Pattern:**
```javascript
// Instead of copying, delegate to another object
const behavior = {
  speak() {
    return `${this.name} speaks`;
  },
  move() {
    return `${this.name} moves`;
  }
};

function createEntity(name) {
  return {
    name,
    // Delegate method calls to behavior
    speak: behavior.speak,
    move: behavior.move
  };
}

const entity = createEntity('Entity');
console.log(entity.speak()); // 'Entity speaks'
```

**3. OLOO (Objects Linking to Other Objects):**
```javascript
// Kyle Simpson's OLOO pattern - pure prototypal inheritance
const Vehicle = {
  init(brand) {
    this.brand = brand;
    return this;
  },
  drive() {
    return `Driving ${this.brand}`;
  }
};

const Car = Object.create(Vehicle);
Car.setup = function(brand, model) {
  this.init(brand);
  this.model = model;
  return this;
};
Car.honk = function() {
  return `${this.brand} ${this.model} honks`;
};

const myCar = Object.create(Car).setup('Toyota', 'Camry');
console.log(myCar.drive()); // 'Driving Toyota'
console.log(myCar.honk());  // 'Toyota Camry honks'
```

**Production Use Cases:**

**1. Component Inheritance:**
```javascript
// Base UI Component
class Component {
  constructor(selector) {
    this.element = document.querySelector(selector);
    this.state = {};
  }
  
  setState(newState) {
    this.state = { ...this.state, ...newState };
    this.render();
  }
  
  render() {
    // Override in child classes
  }
}

// Button component inheriting from Component
class Button extends Component {
  constructor(selector, onClick) {
    super(selector);
    this.onClick = onClick;
    this.setup();
  }
  
  setup() {
    this.element.addEventListener('click', () => {
      this.onClick();
    });
  }
  
  render() {
    this.element.textContent = this.state.text || 'Click me';
  }
}

// Modal component
class Modal extends Component {
  constructor(selector) {
    super(selector);
    this.state = { isOpen: false };
  }
  
  open() {
    this.setState({ isOpen: true });
  }
  
  close() {
    this.setState({ isOpen: false });
  }
  
  render() {
    this.element.style.display = this.state.isOpen ? 'block' : 'none';
  }
}
```

**2. API Client Inheritance:**
```javascript
// Base HTTP client
class HTTPClient {
  constructor(baseURL) {
    this.baseURL = baseURL;
  }
  
  async request(endpoint, options = {}) {
    const url = `${this.baseURL}${endpoint}`;
    const response = await fetch(url, options);
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    return response.json();
  }
  
  async get(endpoint) {
    return this.request(endpoint, { method: 'GET' });
  }
  
  async post(endpoint, data) {
    return this.request(endpoint, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data)
    });
  }
}

// Authenticated client
class AuthenticatedClient extends HTTPClient {
  constructor(baseURL, token) {
    super(baseURL);
    this.token = token;
  }
  
  async request(endpoint, options = {}) {
    const authOptions = {
      ...options,
      headers: {
        ...options.headers,
        'Authorization': `Bearer ${this.token}`
      }
    };
    
    return super.request(endpoint, authOptions);
  }
}

// User-specific client
class UserClient extends AuthenticatedClient {
  async getProfile() {
    return this.get('/user/profile');
  }
  
  async updateProfile(data) {
    return this.post('/user/profile', data);
  }
}

const userClient = new UserClient('https://api.example.com', 'token123');
await userClient.getProfile(); // Uses inherited request method with auth
```

**3. Error Handling Hierarchy:**
```javascript
// Base error
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
    this.isOperational = true;
    Error.captureStackTrace(this, this.constructor);
  }
}

// Specific errors
class ValidationError extends AppError {
  constructor(message) {
    super(message, 400);
    this.name = 'ValidationError';
  }
}

class AuthenticationError extends AppError {
  constructor(message) {
    super(message, 401);
    this.name = 'AuthenticationError';
  }
}

class NotFoundError extends AppError {
  constructor(resource) {
    super(`${resource} not found`, 404);
    this.name = 'NotFoundError';
  }
}

// Usage
try {
  throw new ValidationError('Invalid email format');
} catch (error) {
  if (error instanceof ValidationError) {
    console.log('Validation failed:', error.message);
  }
  if (error instanceof AppError) {
    console.log('Status code:', error.statusCode);
  }
}
```

**4. Model Inheritance (ORM-style):**
```javascript
// Base model
class Model {
  static tableName = '';
  
  static async find(id) {
    const result = await db.query(
      `SELECT * FROM ${this.tableName} WHERE id = ?`,
      [id]
    );
    return new this(result[0]);
  }
  
  async save() {
    const fields = Object.keys(this);
    const values = Object.values(this);
    
    return await db.query(
      `INSERT INTO ${this.constructor.tableName} SET ?`,
      [this]
    );
  }
  
  async delete() {
    return await db.query(
      `DELETE FROM ${this.constructor.tableName} WHERE id = ?`,
      [this.id]
    );
  }
}

// User model
class User extends Model {
  static tableName = 'users';
  
  constructor(data) {
    super();
    this.id = data.id;
    this.name = data.name;
    this.email = data.email;
  }
  
  async getPosts() {
    return await Post.findByUserId(this.id);
  }
}

// Post model
class Post extends Model {
  static tableName = 'posts';
  
  constructor(data) {
    super();
    this.id = data.id;
    this.userId = data.userId;
    this.title = data.title;
    this.content = data.content;
  }
  
  static async findByUserId(userId) {
    const results = await db.query(
      `SELECT * FROM ${this.tableName} WHERE userId = ?`,
      [userId]
    );
    return results.map(row => new this(row));
  }
}

// Usage
const user = await User.find(1);
await user.save();
const posts = await user.getPosts();
```

**Benefits of Prototypal Inheritance:**
- **Memory Efficiency:** Methods shared via prototype
- **Dynamic:** Can modify inheritance at runtime
- **Simple:** Objects inherit from objects directly
- **Flexible:** Easy to compose behaviors
- **Natural to JavaScript:** Language's native inheritance model

**Drawbacks:**
- **Can be confusing:** Different from classical OOP
- **Performance:** Deep prototype chains can be slower
- **No true privacy:** All properties are accessible
- **Complexity:** Mixing patterns can be confusing

**Best Practices:**
- Use ES6 classes for cleaner syntax (they use prototypes underneath)
- Prefer composition over deep inheritance hierarchies
- Keep prototype chains shallow for performance
- Use `Object.create()` for prototypal inheritance without constructors
- Don't modify built-in prototypes (Array.prototype, Object.prototype, etc.)
- Use `super` to call parent methods in classes
- Understand that classes are syntactic sugar over prototypes
- Consider mixins for multiple inheritance scenarios

**Key Takeaways:**
- Prototypal inheritance is JavaScript's native inheritance mechanism
- Objects inherit directly from other objects
- ES6 classes are syntactic sugar over prototypal inheritance
- Methods are shared via prototype for memory efficiency
- More flexible and dynamic than classical inheritance
- Understanding it is essential for mastering JavaScript OOP

33. What is the difference between `__proto__` and `prototype`?

**Answer:**
`__proto__` and `prototype` are two different properties that are often confused, but they serve different purposes in JavaScript's prototypal inheritance system.

**Key Differences:**

| Aspect | `__proto__` | `prototype` |
|--------|------------|------------|
| **What it is** | Actual object used in lookup chain | Property on constructor functions |
| **Exists on** | Every object (instances) | Only on constructor functions |
| **Points to** | The prototype of the object | Object that will become `__proto__` of instances |
| **Used for** | Prototype chain lookup | Creating new instances with `new` |
| **Standard** | Non-standard (deprecated) | Standard part of JavaScript |
| **Modern alternative** | `Object.getPrototypeOf()` / `Object.setPrototypeOf()` | Still used with constructors |

**`prototype` Property:**

The `prototype` property exists **only on constructor functions** (and classes). It's an object that will become the `__proto__` of instances created with that constructor.

```javascript
// Constructor function
function Person(name) {
  this.name = name;
}

// prototype is a property on the constructor
Person.prototype.greet = function() {
  return `Hello, I'm ${this.name}`;
};

console.log(typeof Person.prototype);        // 'object'
console.log(Person.prototype.constructor === Person); // true

// Create instance
const john = new Person('John');

// john doesn't have a prototype property
console.log(john.prototype); // undefined

// But john's __proto__ points to Person.prototype
console.log(john.__proto__ === Person.prototype); // true
```

**`__proto__` Property:**

The `__proto__` property exists on **every object** (except `null`). It's the actual object used in the prototype chain for property lookups.

```javascript
const john = new Person('John');

// __proto__ is the actual prototype used for lookups
console.log(john.__proto__);                    // Person.prototype
console.log(john.__proto__ === Person.prototype); // true

// The chain continues
console.log(john.__proto__.__proto__ === Object.prototype); // true
console.log(Object.prototype.__proto__); // null (end of chain)
```

**Visual Representation:**

```
Constructor Function: Person
    |
    | .prototype property
    ↓
Person.prototype object
    ↑
    | [[Prototype]] / __proto__
    |
Instance: john object
```

**Complete Example:**

```javascript
function Animal(name) {
  this.name = name;
}

Animal.prototype.eat = function() {
  return `${this.name} is eating`;
};

const dog = new Animal('Dog');

// Animal.prototype - the prototype object for instances
console.log(Animal.prototype);                    // { eat: [Function], constructor: Animal }

// dog.__proto__ - dog's actual prototype
console.log(dog.__proto__);                       // Same as Animal.prototype
console.log(dog.__proto__ === Animal.prototype);  // true

// dog doesn't have .prototype property
console.log(dog.prototype);                       // undefined

// Animal.prototype has __proto__ too
console.log(Animal.prototype.__proto__ === Object.prototype); // true

// The constructor property links back
console.log(Animal.prototype.constructor === Animal);         // true
console.log(dog.constructor === Animal);                      // true (via prototype chain)
```

**Why Two Different Properties?**

```javascript
function Person(name) {
  this.name = name;
}

// 1. Person.prototype: Template for instances
Person.prototype.greet = function() {
  return `Hello, ${this.name}`;
};

// 2. Create instance with 'new'
const john = new Person('John');

// 3. Behind the scenes, new does this:
// const john = {};
// john.__proto__ = Person.prototype; // Link instance to constructor's prototype
// Person.call(john, 'John');         // Call constructor with instance as 'this'
// return john;

// Now john can access greet via __proto__
console.log(john.greet()); // Works because john.__proto__.greet exists
```

**Using `__proto__` (Deprecated - Don't Use):**

```javascript
const animal = {
  eats: true
};

const rabbit = {
  jumps: true
};

// ❌ Don't use __proto__ directly (deprecated)
rabbit.__proto__ = animal;

console.log(rabbit.eats); // true (inherited)

// ✅ Use Object.getPrototypeOf() / Object.setPrototypeOf() instead
Object.setPrototypeOf(rabbit, animal);
console.log(Object.getPrototypeOf(rabbit) === animal); // true

// ✅ Or better, use Object.create()
const rabbit2 = Object.create(animal);
rabbit2.jumps = true;
```

**Modern Alternatives to `__proto__`:**

```javascript
const animal = { eats: true };
const rabbit = { jumps: true };

// ❌ Old way (deprecated)
rabbit.__proto__ = animal;

// ✅ Get prototype
const proto = Object.getPrototypeOf(rabbit);

// ✅ Set prototype
Object.setPrototypeOf(rabbit, animal);

// ✅ Best: Create with prototype from the start
const rabbit2 = Object.create(animal);
rabbit2.jumps = true;

// Check prototype
console.log(Object.getPrototypeOf(rabbit2) === animal); // true
```

**ES6 Classes and prototype:**

```javascript
class Animal {
  constructor(name) {
    this.name = name;
  }
  
  speak() {
    return `${this.name} makes a sound`;
  }
}

// Even with classes, prototype still exists
console.log(typeof Animal.prototype); // 'object'

Animal.prototype.sleep = function() {
  return `${this.name} is sleeping`;
};

const cat = new Animal('Cat');

// Instance has __proto__ pointing to class prototype
console.log(cat.__proto__ === Animal.prototype); // true

// Can still access prototype methods
console.log(cat.speak());  // 'Cat makes a sound'
console.log(cat.sleep());  // 'Cat is sleeping'

// Class methods are added to prototype
console.log(Animal.prototype.speak); // [Function: speak]
```

**Practical Examples:**

**1. Adding Methods to Instances vs Prototype:**
```javascript
function Person(name) {
  this.name = name;
  
  // ❌ Bad - method created for each instance
  this.greet = function() {
    return `Hello, ${this.name}`;
  };
}

// ✅ Good - method shared via prototype
Person.prototype.greet = function() {
  return `Hello, ${this.name}`;
};

const person1 = new Person('John');
const person2 = new Person('Jane');

// With instance methods - different functions
console.log(person1.greet === person2.greet); // false (if defined in constructor)

// With prototype methods - same function
console.log(person1.greet === person2.greet); // true (if on prototype)
```

**2. Checking the Prototype Chain:**
```javascript
function Animal() {}
function Dog() {}

Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;

const myDog = new Dog();

// Check with __proto__ (don't use in production)
console.log(myDog.__proto__ === Dog.prototype);                    // true
console.log(myDog.__proto__.__proto__ === Animal.prototype);       // true

// ✅ Better: Use Object.getPrototypeOf()
console.log(Object.getPrototypeOf(myDog) === Dog.prototype);       // true
console.log(Object.getPrototypeOf(Dog.prototype) === Animal.prototype); // true

// ✅ Or use instanceof
console.log(myDog instanceof Dog);    // true
console.log(myDog instanceof Animal); // true
```

**3. Dynamic Prototype Modification:**
```javascript
function Counter() {
  this.count = 0;
}

const counter1 = new Counter();

// Add method to prototype dynamically
Counter.prototype.increment = function() {
  this.count++;
};

// Existing instances immediately have access
counter1.increment();
console.log(counter1.count); // 1

// New instances also have it
const counter2 = new Counter();
counter2.increment();
console.log(counter2.count); // 1

// Both instances share the same method
console.log(counter1.increment === counter2.increment); // true
```

**4. Object Literals Don't Have prototype Property:**
```javascript
const obj = {
  name: 'Object'
};

// Object literals don't have .prototype
console.log(obj.prototype); // undefined

// But they have __proto__
console.log(obj.__proto__ === Object.prototype); // true

// Functions have both
function func() {}
console.log(func.prototype);                     // {} (exists)
console.log(func.__proto__ === Function.prototype); // true
```

**Common Pitfalls:**

**1. Confusing Which to Use:**
```javascript
function Person() {}

// ❌ Wrong - trying to add to instance's __proto__
const john = new Person();
john.__proto__.greet = function() { /* ... */ }; // Don't do this

// ✅ Right - add to constructor's prototype
Person.prototype.greet = function() { /* ... */ };
```

**2. Losing Constructor Reference:**
```javascript
function Parent() {}
function Child() {}

// ❌ Wrong - loses constructor reference
Child.prototype = Parent.prototype; // Don't do this

// ✅ Right - creates new object with Parent.prototype as prototype
Child.prototype = Object.create(Parent.prototype);
Child.prototype.constructor = Child; // Restore constructor
```

**3. Modifying Built-in Prototypes:**
```javascript
// ❌ Bad practice - affects all arrays globally
Array.prototype.last = function() {
  return this[this.length - 1];
};

// ✅ Better - extend in a controlled way
class MyArray extends Array {
  last() {
    return this[this.length - 1];
  }
}
```

**Memory Implications:**

```javascript
function Person(name) {
  this.name = name;
  
  // ❌ Each instance gets its own copy (wastes memory)
  this.greet = function() {
    return `Hello, ${this.name}`;
  };
}

// Creating 1000 instances = 1000 copies of greet function
const people = Array.from({ length: 1000 }, (_, i) => new Person(`Person${i}`));

// ✅ All instances share one copy via prototype (efficient)
Person.prototype.greet = function() {
  return `Hello, ${this.name}`;
};

// Creating 1000 instances = 1 copy of greet function
```

**Best Practices:**
- Never use `__proto__` directly in production code (deprecated)
- Use `Object.getPrototypeOf()` to read prototypes
- Use `Object.setPrototypeOf()` to set prototypes (or better, `Object.create()`)
- Use `Object.create()` when creating objects with specific prototypes
- Add shared methods to `.prototype` for memory efficiency
- Understand `prototype` is only on constructor functions
- Remember `__proto__` exists on all objects
- Use ES6 classes for cleaner syntax (they handle prototypes for you)

**Key Takeaways:**
- `prototype`: Property on constructor functions, template for instances
- `__proto__`: Property on instances, actual prototype used in lookups
- `__proto__` is deprecated, use `Object.getPrototypeOf()` / `Object.setPrototypeOf()`
- Methods on `prototype` are shared across instances (memory efficient)
- Understanding both is crucial for JavaScript inheritance
- ES6 classes hide this complexity but use prototypes underneath

34. What is Object.create()?

**Answer:**
`Object.create()` is a method that creates a new object with a specified prototype object and optional property descriptors. It's the most direct way to implement prototypal inheritance in JavaScript without using constructor functions or classes.

**Syntax:**

```javascript
Object.create(proto, [propertiesObject])
```

- `proto`: The object which should be the prototype of the newly-created object
- `propertiesObject` (optional): An object whose properties define property descriptors to be added to the newly-created object

**Basic Usage:**

```javascript
// Create object with specific prototype
const animal = {
  eats: true,
  walk() {
    console.log('Animal walks');
  }
};

// Create rabbit with animal as prototype
const rabbit = Object.create(animal);
rabbit.jumps = true;

console.log(rabbit.eats);   // true (inherited from animal)
console.log(rabbit.jumps);  // true (own property)
rabbit.walk();              // 'Animal walks' (inherited method)

// Verify prototype
console.log(Object.getPrototypeOf(rabbit) === animal); // true
```

**Creating Object with No Prototype:**

```javascript
// Create truly empty object (no prototype)
const pureObject = Object.create(null);

console.log(pureObject.toString); // undefined (no inherited methods)
console.log(Object.getPrototypeOf(pureObject)); // null

// Useful for creating dictionaries/hash maps
pureObject.key = 'value';
console.log('toString' in pureObject); // false (no inherited properties)

// Regular object has Object.prototype
const regularObject = {};
console.log('toString' in regularObject); // true (inherited from Object.prototype)
```

**With Property Descriptors:**

```javascript
const person = {
  isHuman: true
};

const john = Object.create(person, {
  name: {
    value: 'John',
    writable: true,
    enumerable: true,
    configurable: true
  },
  age: {
    value: 30,
    writable: true,
    enumerable: true,
    configurable: true
  },
  id: {
    value: 123,
    writable: false,     // Read-only
    enumerable: false,   // Won't show in for...in
    configurable: false  // Can't be deleted or reconfigured
  }
});

console.log(john.name);    // 'John'
console.log(john.age);     // 30
console.log(john.isHuman); // true (inherited)

john.name = 'Jane';        // Works (writable: true)
john.id = 999;             // Fails silently (writable: false)
console.log(john.id);      // 123 (unchanged)

delete john.id;            // Fails silently (configurable: false)
console.log(john.id);      // 123 (still there)

for (let key in john) {
  console.log(key);        // 'name', 'age' (id is not enumerable)
}
```

**Comparison with Other Object Creation Methods:**

```javascript
// 1. Object literal
const obj1 = { a: 1 };
// Prototype: Object.prototype

// 2. Constructor function
function MyClass(a) {
  this.a = a;
}
const obj2 = new MyClass(1);
// Prototype: MyClass.prototype

// 3. Object.create()
const proto = { b: 2 };
const obj3 = Object.create(proto);
obj3.a = 1;
// Prototype: proto (custom)

// 4. Object.create(null) - truly empty
const obj4 = Object.create(null);
obj4.a = 1;
// Prototype: null (no prototype)
```

**Implementing Inheritance:**

```javascript
// Parent object
const Animal = {
  init(name) {
    this.name = name;
    return this;
  },
  eat() {
    return `${this.name} is eating`;
  }
};

// Child object
const Dog = Object.create(Animal);
Dog.bark = function() {
  return `${this.name} barks`;
};
Dog.init = function(name, breed) {
  Animal.init.call(this, name);
  this.breed = breed;
  return this;
};

// Create instance
const myDog = Object.create(Dog).init('Rex', 'Labrador');

console.log(myDog.eat());   // 'Rex is eating' (from Animal)
console.log(myDog.bark());  // 'Rex barks' (from Dog)
console.log(myDog.name);    // 'Rex'
console.log(myDog.breed);   // 'Labrador'

// Prototype chain: myDog → Dog → Animal → Object.prototype → null
```

**Polyfill (Understanding How It Works):**

```javascript
if (!Object.create) {
  Object.create = function(proto, propertiesObject) {
    if (typeof proto !== 'object' && typeof proto !== 'function') {
      throw new TypeError('Object prototype may only be an Object or null');
    }
    if (propertiesObject !== undefined) {
      throw new Error('Second argument not supported');
    }
    
    function F() {}
    F.prototype = proto;
    return new F();
  };
}

// This shows that Object.create() essentially:
// 1. Creates a temporary constructor
// 2. Sets its prototype to the provided object
// 3. Returns a new instance
```

**Production Use Cases:**

**1. Creating Dictionary/Hash Map:**
```javascript
// Problem: Regular objects inherit from Object.prototype
const cache = {};
cache.toString = 'some value';
console.log(typeof cache.toString); // 'string' (overwrote inherited method)

// Solution: Use Object.create(null)
const pureCache = Object.create(null);
pureCache.toString = 'some value';
console.log(pureCache.toString); // 'some value' (no conflict)
console.log(pureCache.hasOwnProperty); // undefined (no inherited methods)

// Safe to use any key without conflicts
pureCache.constructor = 'value';
pureCache.__proto__ = 'value';
pureCache.hasOwnProperty = 'value';
// All work without issues
```

**2. Creating Clones with Specific Prototype:**
```javascript
function cloneWithPrototype(obj) {
  // Clone object preserving its prototype
  const clone = Object.create(Object.getPrototypeOf(obj));
  
  // Copy own properties
  Object.keys(obj).forEach(key => {
    clone[key] = obj[key];
  });
  
  return clone;
}

function Person(name) {
  this.name = name;
}
Person.prototype.greet = function() {
  return `Hello, ${this.name}`;
};

const john = new Person('John');
const johnClone = cloneWithPrototype(john);

console.log(johnClone.name);         // 'John'
console.log(johnClone.greet());      // 'Hello, John'
console.log(johnClone instanceof Person); // true
```

**3. Implementing Object Inheritance Pattern:**
```javascript
// Base configuration
const baseConfig = {
  apiUrl: 'https://api.example.com',
  timeout: 5000,
  retries: 3,
  headers: {
    'Content-Type': 'application/json'
  }
};

// Environment-specific configs
const devConfig = Object.create(baseConfig);
devConfig.apiUrl = 'http://localhost:3000';
devConfig.debug = true;

const prodConfig = Object.create(baseConfig);
prodConfig.apiUrl = 'https://api.production.com';
prodConfig.timeout = 10000;

console.log(devConfig.timeout);   // 5000 (inherited)
console.log(devConfig.apiUrl);    // 'http://localhost:3000' (overridden)
console.log(prodConfig.retries);  // 3 (inherited)

// Changes to base affect all children
baseConfig.retries = 5;
console.log(devConfig.retries);   // 5 (inherited)
console.log(prodConfig.retries);  // 5 (inherited)
```

**4. Factory Pattern with Inheritance:**
```javascript
// Base prototype
const Vehicle = {
  init(brand, model) {
    this.brand = brand;
    this.model = model;
    return this;
  },
  getInfo() {
    return `${this.brand} ${this.model}`;
  }
};

// Specialized prototypes
const Car = Object.create(Vehicle);
Car.init = function(brand, model, doors) {
  Vehicle.init.call(this, brand, model);
  this.doors = doors;
  return this;
};
Car.drive = function() {
  return `Driving ${this.getInfo()}`;
};

const Motorcycle = Object.create(Vehicle);
Motorcycle.init = function(brand, model, engineCC) {
  Vehicle.init.call(this, brand, model);
  this.engineCC = engineCC;
  return this;
};
Motorcycle.ride = function() {
  return `Riding ${this.getInfo()}`;
};

// Create instances
const myCar = Object.create(Car).init('Toyota', 'Camry', 4);
const myBike = Object.create(Motorcycle).init('Harley', 'Street 750', 750);

console.log(myCar.drive());      // 'Driving Toyota Camry'
console.log(myBike.ride());      // 'Riding Harley Street 750'
console.log(myCar.getInfo());    // 'Toyota Camry' (inherited from Vehicle)
```

**5. Implementing Method Chaining:**
```javascript
const Calculator = {
  init(value = 0) {
    this.value = value;
    return this;
  },
  add(n) {
    this.value += n;
    return this;
  },
  subtract(n) {
    this.value -= n;
    return this;
  },
  multiply(n) {
    this.value *= n;
    return this;
  },
  divide(n) {
    this.value /= n;
    return this;
  },
  getResult() {
    return this.value;
  }
};

// Create calculator instance
const calc = Object.create(Calculator).init(10);

const result = calc
  .add(5)
  .multiply(2)
  .subtract(10)
  .divide(2)
  .getResult();

console.log(result); // 10
```

**6. Creating Private Data Store:**
```javascript
function createPerson(name, age) {
  // Private data
  const privateData = { name, age };
  
  // Public interface
  return Object.create(null, {
    getName: {
      value: function() {
        return privateData.name;
      },
      enumerable: true
    },
    getAge: {
      value: function() {
        return privateData.age;
      },
      enumerable: true
    },
    setAge: {
      value: function(newAge) {
        if (newAge > 0) {
          privateData.age = newAge;
        }
      },
      enumerable: true
    }
  });
}

const person = createPerson('John', 30);
console.log(person.getName()); // 'John'
console.log(person.getAge());  // 30
person.setAge(31);
console.log(person.getAge());  // 31

// Can't access private data directly
console.log(person.name); // undefined
console.log(person.age);  // undefined
```

**Common Patterns:**

**1. OLOO (Objects Linking to Other Objects):**
```javascript
// Kyle Simpson's OLOO pattern
const Parent = {
  init(name) {
    this.name = name;
    return this;
  },
  greet() {
    return `Hello, I'm ${this.name}`;
  }
};

const Child = Object.create(Parent);
Child.setup = function(name, age) {
  this.init(name);
  this.age = age;
  return this;
};
Child.introduce = function() {
  return `${this.greet()} and I'm ${this.age} years old`;
};

const child = Object.create(Child).setup('John', 10);
console.log(child.introduce()); // "Hello, I'm John and I'm 10 years old"
```

**2. Extending Built-in Types:**
```javascript
// Create enhanced array with custom prototype
const arrayMethods = Object.create(Array.prototype);

arrayMethods.first = function() {
  return this[0];
};

arrayMethods.last = function() {
  return this[this.length - 1];
};

function createArray(...items) {
  const arr = Object.create(arrayMethods);
  arr.push(...items);
  return arr;
}

const myArray = createArray(1, 2, 3, 4, 5);
console.log(myArray.first());  // 1
console.log(myArray.last());   // 5
console.log(myArray.length);   // 5
```

**Advantages of Object.create():**
- **Pure prototypal inheritance:** No constructors needed
- **Flexible:** Can create objects with any prototype (including `null`)
- **Property descriptors:** Fine control over property characteristics
- **Memory efficient:** Methods shared via prototype
- **Clean syntax:** Direct and explicit
- **No `new` keyword:** Simpler mental model

**Disadvantages:**
- **Verbose:** Requires more code than classes
- **No initialization:** Need separate init method
- **Less familiar:** Classes are more common pattern
- **Property descriptors:** Second parameter is complex
- **Performance:** Slightly slower than literal notation in some engines

**Object.create() vs Constructor vs Class:**

```javascript
// 1. Object.create()
const proto1 = { greet() { return 'Hello'; } };
const obj1 = Object.create(proto1);
obj1.name = 'John';

// 2. Constructor function
function Person(name) {
  this.name = name;
}
Person.prototype.greet = function() { return 'Hello'; };
const obj2 = new Person('John');

// 3. ES6 Class
class PersonClass {
  constructor(name) {
    this.name = name;
  }
  greet() { return 'Hello'; }
}
const obj3 = new PersonClass('John');

// All achieve similar results, different syntax
console.log(obj1.greet()); // 'Hello'
console.log(obj2.greet()); // 'Hello'
console.log(obj3.greet()); // 'Hello'
```

**Best Practices:**
- Use `Object.create(null)` for dictionaries/hash maps to avoid inherited properties
- Prefer classes for constructor-based patterns (cleaner syntax)
- Use `Object.create()` for pure prototypal inheritance without constructors
- Always check for `null` prototype when using `Object.create(null)`
- Use property descriptors when you need fine control over properties
- Consider OLOO pattern for functional programming style
- Don't mix too many patterns in same codebase (consistency is key)

**Key Takeaways:**
- `Object.create()` creates new object with specified prototype
- Most direct way to implement prototypal inheritance
- Can create objects with `null` prototype (no inherited properties)
- Supports property descriptors for fine-grained control
- Useful for pure prototypal patterns and dictionaries
- Classes are syntactic sugar, `Object.create()` is the underlying mechanism
- Essential for understanding JavaScript's inheritance model

35. What are getters and setters?

**Answer:**
Getters and setters are special methods that provide access control to object properties. They allow you to define custom behavior when reading (getting) or writing (setting) a property value, enabling validation, computed properties, and encapsulation.

**Syntax:**

```javascript
// Object literal syntax
const obj = {
  get propertyName() {
    // Return value
  },
  set propertyName(value) {
    // Set value
  }
};

// Class syntax
class MyClass {
  get propertyName() {
    // Return value
  }
  set propertyName(value) {
    // Set value
  }
}

// Object.defineProperty syntax
Object.defineProperty(obj, 'propertyName', {
  get() {
    // Return value
  },
  set(value) {
    // Set value
  }
});
```

**Basic Example:**

```javascript
const person = {
  firstName: 'John',
  lastName: 'Doe',
  
  // Getter - called when reading property
  get fullName() {
    return `${this.firstName} ${this.lastName}`;
  },
  
  // Setter - called when writing property
  set fullName(value) {
    const parts = value.split(' ');
    this.firstName = parts[0];
    this.lastName = parts[1];
  }
};

// Using getter (looks like property access)
console.log(person.fullName); // 'John Doe' (no parentheses!)

// Using setter (looks like assignment)
person.fullName = 'Jane Smith';
console.log(person.firstName); // 'Jane'
console.log(person.lastName);  // 'Smith'
console.log(person.fullName);  // 'Jane Smith'
```

**Getters (Reading Values):**

```javascript
const circle = {
  radius: 5,
  
  // Computed property
  get diameter() {
    return this.radius * 2;
  },
  
  get circumference() {
    return 2 * Math.PI * this.radius;
  },
  
  get area() {
    return Math.PI * this.radius ** 2;
  }
};

console.log(circle.diameter);      // 10
console.log(circle.circumference); // 31.41592653589793
console.log(circle.area);          // 78.53981633974483

// Update radius
circle.radius = 10;
console.log(circle.diameter);      // 20 (automatically updates)
console.log(circle.area);          // 314.1592653589793
```

**Setters (Writing Values with Validation):**

```javascript
const user = {
  _age: 0, // Convention: underscore indicates "private"
  
  get age() {
    return this._age;
  },
  
  set age(value) {
    if (typeof value !== 'number') {
      throw new TypeError('Age must be a number');
    }
    if (value < 0 || value > 150) {
      throw new RangeError('Age must be between 0 and 150');
    }
    this._age = value;
  }
};

user.age = 25;
console.log(user.age); // 25

// Validation in action
try {
  user.age = -5; // Throws RangeError
} catch (e) {
  console.error(e.message); // 'Age must be between 0 and 150'
}

try {
  user.age = 'twenty'; // Throws TypeError
} catch (e) {
  console.error(e.message); // 'Age must be a number'
}
```

**Class Example:**

```javascript
class Temperature {
  constructor(celsius) {
    this._celsius = celsius;
  }
  
  get celsius() {
    return this._celsius;
  }
  
  set celsius(value) {
    if (value < -273.15) {
      throw new Error('Temperature below absolute zero!');
    }
    this._celsius = value;
  }
  
  get fahrenheit() {
    return (this._celsius * 9/5) + 32;
  }
  
  set fahrenheit(value) {
    this.celsius = (value - 32) * 5/9;
  }
  
  get kelvin() {
    return this._celsius + 273.15;
  }
  
  set kelvin(value) {
    this.celsius = value - 273.15;
  }
}

const temp = new Temperature(25);
console.log(temp.celsius);    // 25
console.log(temp.fahrenheit); // 77
console.log(temp.kelvin);     // 298.15

// Set using different scale
temp.fahrenheit = 68;
console.log(temp.celsius);    // 20

temp.kelvin = 300;
console.log(temp.celsius);    // 26.850000000000023
```

**Using Object.defineProperty:**

```javascript
const bankAccount = {
  _balance: 1000
};

Object.defineProperty(bankAccount, 'balance', {
  get() {
    console.log('Getting balance...');
    return this._balance;
  },
  set(value) {
    console.log('Setting balance...');
    if (value < 0) {
      throw new Error('Balance cannot be negative');
    }
    this._balance = value;
  },
  enumerable: true,
  configurable: true
});

console.log(bankAccount.balance); // Getting balance... 1000
bankAccount.balance = 2000;       // Setting balance...
console.log(bankAccount.balance); // Getting balance... 2000
```

**Read-Only Properties (Getter without Setter):**

```javascript
const config = {
  _apiKey: 'secret-key-12345',
  
  // Only getter, no setter - effectively read-only
  get apiKey() {
    return this._apiKey;
  }
};

console.log(config.apiKey); // 'secret-key-12345'
config.apiKey = 'new-key';  // Silently fails (or throws in strict mode)
console.log(config.apiKey); // Still 'secret-key-12345'

// In strict mode
'use strict';
const strictConfig = {
  _value: 42,
  get value() {
    return this._value;
  }
};

strictConfig.value = 100; // TypeError: Cannot set property value of #<Object> which has only a getter
```

**Write-Only Properties (Setter without Getter):**

```javascript
const logger = {
  _logs: [],
  
  // Only setter, no getter
  set log(message) {
    this._logs.push({
      message,
      timestamp: new Date()
    });
    console.log(`[${new Date().toISOString()}] ${message}`);
  }
};

logger.log = 'User logged in';  // Works
logger.log = 'User logged out'; // Works

// Can't read the property
console.log(logger.log); // undefined
console.log(logger._logs); // Can still access underlying data
```

**Production Use Cases:**

**1. Data Validation:**
```javascript
class User {
  constructor(email, password) {
    this.email = email;
    this.password = password;
  }
  
  get email() {
    return this._email;
  }
  
  set email(value) {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!emailRegex.test(value)) {
      throw new Error('Invalid email format');
    }
    this._email = value.toLowerCase();
  }
  
  get password() {
    return '********'; // Never expose password
  }
  
  set password(value) {
    if (value.length < 8) {
      throw new Error('Password must be at least 8 characters');
    }
    if (!/[A-Z]/.test(value)) {
      throw new Error('Password must contain uppercase letter');
    }
    if (!/[0-9]/.test(value)) {
      throw new Error('Password must contain number');
    }
    this._password = value; // In real app, hash this!
  }
}

const user = new User('JOHN@EXAMPLE.COM', 'Password123');
console.log(user.email);    // 'john@example.com' (normalized)
console.log(user.password); // '********' (hidden)
```

**2. Lazy Evaluation/Computed Properties:**
```javascript
class DataSet {
  constructor(data) {
    this.data = data;
    this._sum = null;
    this._average = null;
  }
  
  // Lazy calculation - computed only when first accessed
  get sum() {
    if (this._sum === null) {
      console.log('Computing sum...');
      this._sum = this.data.reduce((a, b) => a + b, 0);
    }
    return this._sum;
  }
  
  get average() {
    if (this._average === null) {
      console.log('Computing average...');
      this._average = this.sum / this.data.length;
    }
    return this._average;
  }
  
  addValue(value) {
    this.data.push(value);
    // Invalidate cached values
    this._sum = null;
    this._average = null;
  }
}

const dataset = new DataSet([1, 2, 3, 4, 5]);
console.log(dataset.average); // Computing sum... Computing average... 3
console.log(dataset.average); // 3 (cached, no recomputation)

dataset.addValue(6);
console.log(dataset.average); // Computing sum... Computing average... 3.5 (recalculated)
```

**3. Property Change Tracking:**
```javascript
class Observable {
  constructor() {
    this._listeners = {};
    this._data = {};
  }
  
  defineProperty(propName, initialValue) {
    this._data[propName] = initialValue;
    
    Object.defineProperty(this, propName, {
      get() {
        return this._data[propName];
      },
      set(value) {
        const oldValue = this._data[propName];
        this._data[propName] = value;
        this._notify(propName, oldValue, value);
      }
    });
  }
  
  onChange(propName, callback) {
    if (!this._listeners[propName]) {
      this._listeners[propName] = [];
    }
    this._listeners[propName].push(callback);
  }
  
  _notify(propName, oldValue, newValue) {
    if (this._listeners[propName]) {
      this._listeners[propName].forEach(callback => {
        callback(newValue, oldValue);
      });
    }
  }
}

const model = new Observable();
model.defineProperty('name', 'John');
model.defineProperty('age', 25);

model.onChange('name', (newVal, oldVal) => {
  console.log(`Name changed from ${oldVal} to ${newVal}`);
});

model.name = 'Jane'; // Name changed from John to Jane
model.age = 26;      // No listener, no notification
```

**4. API Response Formatting:**
```javascript
class APIResponse {
  constructor(data) {
    this._data = data;
  }
  
  get data() {
    return this._data;
  }
  
  // Format data differently for different contexts
  get jsonAPI() {
    return {
      data: {
        type: this._data.type,
        id: this._data.id,
        attributes: { ...this._data }
      }
    };
  }
  
  get restAPI() {
    return {
      status: 'success',
      result: this._data,
      timestamp: Date.now()
    };
  }
  
  get graphQL() {
    return {
      data: this._data,
      errors: null
    };
  }
}

const response = new APIResponse({ id: 1, name: 'John', type: 'user' });

console.log(response.jsonAPI);
// { data: { type: 'user', id: 1, attributes: {...} } }

console.log(response.restAPI);
// { status: 'success', result: {...}, timestamp: ... }
```

**5. Dependency Injection:**
```javascript
class ServiceContainer {
  constructor() {
    this._services = new Map();
  }
  
  register(name, factory) {
    this._services.set(name, { factory, instance: null });
  }
  
  // Lazy singleton instantiation
  getService(name) {
    if (!this._services.has(name)) {
      throw new Error(`Service ${name} not registered`);
    }
    
    const service = this._services.get(name);
    
    if (!service.instance) {
      console.log(`Instantiating ${name}...`);
      service.instance = service.factory();
    }
    
    return service.instance;
  }
}

// Create dynamic getters
class App {
  constructor() {
    this.container = new ServiceContainer();
    
    // Register services
    this.container.register('logger', () => new Logger());
    this.container.register('database', () => new Database());
    
    // Create getters for each service
    ['logger', 'database'].forEach(serviceName => {
      Object.defineProperty(this, serviceName, {
        get() {
          return this.container.getService(serviceName);
        }
      });
    });
  }
}

const app = new App();
console.log(app.logger);   // Instantiating logger... [Logger instance]
console.log(app.logger);   // [Logger instance] (same instance, no reinstantiation)
console.log(app.database); // Instantiating database... [Database instance]
```

**Common Patterns:**

**1. Private Properties Pattern:**
```javascript
class Counter {
  #count = 0; // Private field (ES2022)
  
  get value() {
    return this.#count;
  }
  
  increment() {
    this.#count++;
  }
  
  decrement() {
    this.#count--;
  }
}

const counter = new Counter();
console.log(counter.value);  // 0
counter.increment();
console.log(counter.value);  // 1
// counter.#count; // SyntaxError: Private field '#count' must be declared in an enclosing class
```

**2. Proxy-like Behavior:**
```javascript
const target = {
  _name: 'Original'
};

Object.defineProperty(target, 'name', {
  get() {
    console.log('GET name');
    return this._name;
  },
  set(value) {
    console.log('SET name to', value);
    this._name = value;
  }
});

target.name;           // GET name
target.name = 'New';   // SET name to New
```

**Advantages:**
- **Validation:** Check values before setting
- **Computed properties:** Calculate values dynamically
- **Encapsulation:** Hide internal implementation
- **Side effects:** Trigger actions on get/set
- **Backwards compatibility:** Add validation without breaking API
- **Lazy evaluation:** Compute expensive values only when needed

**Disadvantages:**
- **Performance:** Slightly slower than direct property access
- **Debugging:** Harder to debug (hidden behavior)
- **Confusion:** Can be unclear that logic is executing
- **Inheritance:** Can be tricky with prototypes
- **Serialization:** JSON.stringify doesn't call getters

**Best Practices:**
- Use underscore prefix convention for "private" backing properties (`_property`)
- Keep getter/setter logic simple and fast
- Don't perform heavy computations in getters
- Avoid side effects in getters when possible
- Use meaningful names that indicate behavior
- Document getter/setter behavior in comments
- Consider using private fields (#field) instead of underscore convention
- Be consistent with getter/setter usage across codebase

**Key Takeaways:**
- Getters/setters provide controlled access to properties
- Enable validation, computed properties, and encapsulation
- Look like regular property access (no parentheses)
- Can be defined in object literals, classes, or Object.defineProperty
- Useful for data validation, lazy evaluation, and change tracking
- Slightly slower than direct property access
- Essential for clean APIs and data integrity

36. What is the difference between Object.freeze() and Object.seal()?

**Answer:**
`Object.freeze()` and `Object.seal()` are methods that prevent modifications to objects, but they have different levels of restriction. Both make objects immutable to some degree, but `freeze()` is more restrictive than `seal()`.

**Quick Comparison:**

| Operation | Normal Object | Object.seal() | Object.freeze() |
|-----------|--------------|---------------|-----------------|
| Add new properties | ✅ Yes | ❌ No | ❌ No |
| Delete properties | ✅ Yes | ❌ No | ❌ No |
| Modify existing properties | ✅ Yes | ✅ Yes | ❌ No |
| Reconfigure properties | ✅ Yes | ❌ No | ❌ No |
| Change prototype | ✅ Yes | ❌ No | ❌ No |

**Object.seal():**

`Object.seal()` prevents adding or removing properties, but allows modifying existing property values.

```javascript
const person = {
  name: 'John',
  age: 30
};

Object.seal(person);

// ✅ Can modify existing properties
person.name = 'Jane';
person.age = 31;
console.log(person); // { name: 'Jane', age: 31 }

// ❌ Cannot add new properties
person.email = 'jane@example.com';
console.log(person.email); // undefined (silently fails)

// ❌ Cannot delete properties
delete person.age;
console.log(person.age); // 31 (still there)

// Check if sealed
console.log(Object.isSealed(person)); // true
```

**Object.freeze():**

`Object.freeze()` prevents adding, removing, AND modifying properties. The object becomes completely immutable.

```javascript
const config = {
  apiUrl: 'https://api.example.com',
  timeout: 5000
};

Object.freeze(config);

// ❌ Cannot modify existing properties
config.apiUrl = 'https://api.malicious.com';
console.log(config.apiUrl); // 'https://api.example.com' (unchanged)

config.timeout = 10000;
console.log(config.timeout); // 5000 (unchanged)

// ❌ Cannot add new properties
config.retries = 3;
console.log(config.retries); // undefined

// ❌ Cannot delete properties
delete config.timeout;
console.log(config.timeout); // 5000 (still there)

// Check if frozen
console.log(Object.isFrozen(config)); // true
```

**Detailed Comparison:**

```javascript
// Normal object
const normal = { a: 1, b: 2 };
normal.a = 10;        // ✅ Works
normal.c = 3;         // ✅ Works
delete normal.b;      // ✅ Works
console.log(normal);  // { a: 10, c: 3 }

// Sealed object
const sealed = { a: 1, b: 2 };
Object.seal(sealed);
sealed.a = 10;        // ✅ Works (can modify)
sealed.c = 3;         // ❌ Fails (cannot add)
delete sealed.b;      // ❌ Fails (cannot delete)
console.log(sealed);  // { a: 10, b: 2 }

// Frozen object
const frozen = { a: 1, b: 2 };
Object.freeze(frozen);
frozen.a = 10;        // ❌ Fails (cannot modify)
frozen.c = 3;         // ❌ Fails (cannot add)
delete frozen.b;      // ❌ Fails (cannot delete)
console.log(frozen);  // { a: 1, b: 2 } (completely unchanged)
```

**Strict Mode Behavior:**

In strict mode, attempts to violate freeze/seal restrictions throw errors instead of failing silently.

```javascript
'use strict';

const sealed = Object.seal({ a: 1 });
sealed.b = 2; // TypeError: Cannot add property b, object is not extensible

const frozen = Object.freeze({ a: 1 });
frozen.a = 2; // TypeError: Cannot assign to read only property 'a'
```

**Nested Objects (Shallow Operation):**

Both `freeze()` and `seal()` are **shallow** - they don't affect nested objects.

```javascript
const user = {
  name: 'John',
  address: {
    city: 'New York',
    zip: '10001'
  }
};

Object.freeze(user);

// ❌ Cannot modify top-level properties
user.name = 'Jane';
console.log(user.name); // 'John' (unchanged)

// ✅ CAN modify nested object properties!
user.address.city = 'Los Angeles';
console.log(user.address.city); // 'Los Angeles' (changed!)

// Check frozen status
console.log(Object.isFrozen(user));         // true (top level)
console.log(Object.isFrozen(user.address)); // false (nested object)
```

**Deep Freeze Implementation:**

```javascript
function deepFreeze(obj) {
  // Freeze the object itself
  Object.freeze(obj);
  
  // Recursively freeze all properties
  Object.keys(obj).forEach(key => {
    const value = obj[key];
    if (value && typeof value === 'object') {
      deepFreeze(value);
    }
  });
  
  return obj;
}

const data = {
  user: {
    name: 'John',
    address: {
      city: 'New York'
    }
  }
};

deepFreeze(data);

// Now nested objects are also frozen
data.user.address.city = 'LA';
console.log(data.user.address.city); // 'New York' (unchanged)
```

**Deep Seal Implementation:**

```javascript
function deepSeal(obj) {
  Object.seal(obj);
  
  Object.keys(obj).forEach(key => {
    const value = obj[key];
    if (value && typeof value === 'object') {
      deepSeal(value);
    }
  });
  
  return obj;
}
```

**Checking Status:**

```javascript
const obj = { a: 1 };

// Check if extensible (can add properties)
console.log(Object.isExtensible(obj)); // true

Object.seal(obj);
console.log(Object.isExtensible(obj)); // false
console.log(Object.isSealed(obj));     // true
console.log(Object.isFrozen(obj));     // false (can still modify)

Object.freeze(obj);
console.log(Object.isSealed(obj));     // true (frozen implies sealed)
console.log(Object.isFrozen(obj));     // true
```

**Relationship Between Methods:**

```javascript
// Hierarchy: freeze > seal > preventExtensions

Object.preventExtensions(obj); // Cannot add properties
// Can still: modify, delete, reconfigure

Object.seal(obj); // Cannot add OR delete properties
// Can still: modify existing values

Object.freeze(obj); // Cannot add, delete, OR modify
// Completely immutable (shallow)

// freeze implies seal implies preventExtensions
if (Object.isFrozen(obj)) {
  console.log(Object.isSealed(obj));        // Always true
  console.log(Object.isExtensible(obj));    // Always false
}
```

**Production Use Cases:**

**1. Configuration Objects:**
```javascript
// Prevent accidental modification of config
const CONFIG = Object.freeze({
  API_URL: 'https://api.example.com',
  API_KEY: 'secret-key',
  TIMEOUT: 5000,
  MAX_RETRIES: 3
});

// Typo won't create new property
CONFIG.TIMEOOT = 10000; // Silently fails
console.log(CONFIG.TIMEOUT); // 5000 (correct value)

// Cannot be overridden
CONFIG.API_KEY = 'hacked'; // Fails
console.log(CONFIG.API_KEY); // 'secret-key' (safe)
```

**2. Enum-like Constants:**
```javascript
const Status = Object.freeze({
  PENDING: 'pending',
  IN_PROGRESS: 'in_progress',
  COMPLETED: 'completed',
  FAILED: 'failed'
});

// Safe to use throughout application
function updateStatus(newStatus) {
  if (!Object.values(Status).includes(newStatus)) {
    throw new Error('Invalid status');
  }
  // Update logic...
}

// Cannot be tampered with
Status.PENDING = 'hacked'; // Fails
delete Status.COMPLETED;   // Fails
```

**3. Data Integrity in Classes:**
```javascript
class Transaction {
  constructor(id, amount, timestamp) {
    // Make transaction immutable after creation
    Object.defineProperties(this, {
      id: { value: id, writable: false, enumerable: true },
      amount: { value: amount, writable: false, enumerable: true },
      timestamp: { value: timestamp, writable: false, enumerable: true }
    });
    
    Object.freeze(this);
  }
}

const transaction = new Transaction(1, 100, Date.now());
transaction.amount = 999999; // Fails silently (or throws in strict mode)
console.log(transaction.amount); // 100 (safe)
```

**4. Preventing Prototype Pollution:**
```javascript
// Seal prototypes to prevent pollution attacks
Object.seal(Object.prototype);
Object.seal(Array.prototype);
Object.seal(Function.prototype);

// Now these attacks fail
try {
  Object.prototype.isAdmin = true; // Fails
} catch (e) {
  console.error('Prototype pollution prevented');
}
```

**5. State Management:**
```javascript
class Store {
  constructor(initialState) {
    this._state = initialState;
    Object.freeze(this._state); // Immutable state
  }
  
  getState() {
    return this._state;
  }
  
  setState(newState) {
    // Create new frozen object instead of mutating
    this._state = Object.freeze({
      ...this._state,
      ...newState
    });
  }
}

const store = new Store({ count: 0 });

// Cannot mutate state directly
const state = store.getState();
state.count++; // Fails
console.log(store.getState().count); // 0 (unchanged)

// Must use setState
store.setState({ count: 1 });
console.log(store.getState().count); // 1
```

**Performance Considerations:**

```javascript
// Freezing has performance implications
const obj = { a: 1, b: 2, c: 3 };

// Normal object - fast property access
console.time('normal');
for (let i = 0; i < 1000000; i++) {
  obj.a;
}
console.timeEnd('normal');

// Frozen object - slightly slower
Object.freeze(obj);
console.time('frozen');
for (let i = 0; i < 1000000; i++) {
  obj.a;
}
console.timeEnd('frozen');

// Difference is usually negligible for normal use
```

**Common Pitfalls:**

**1. Forgetting Shallow Nature:**
```javascript
const config = Object.freeze({
  database: {
    host: 'localhost',
    port: 5432
  }
});

// ⚠️ Nested objects are NOT frozen
config.database.host = 'hacker.com'; // Works!
console.log(config.database.host); // 'hacker.com' (modified)

// Need deep freeze for nested objects
```

**2. Arrays Are Objects:**
```javascript
const arr = Object.freeze([1, 2, 3]);

// ❌ Cannot modify
arr[0] = 999;
console.log(arr[0]); // 1 (unchanged)

// ❌ Cannot use mutating methods
arr.push(4);    // TypeError
arr.pop();      // TypeError
arr.sort();     // TypeError

// ✅ Can use non-mutating methods
const newArr = arr.map(x => x * 2);
console.log(newArr); // [2, 4, 6]
```

**3. Not Checking Before Operations:**
```javascript
function addProperty(obj, key, value) {
  // ❌ Bad - doesn't check if frozen
  obj[key] = value;
  
  // ✅ Good - check first
  if (Object.isFrozen(obj)) {
    throw new Error('Cannot modify frozen object');
  }
  obj[key] = value;
}
```

**Advantages:**
- **Immutability:** Prevent accidental mutations
- **Security:** Protect critical data
- **Debugging:** Easier to track bugs (state doesn't change unexpectedly)
- **Thread safety:** Safe to share between workers
- **Predictability:** Object behavior is guaranteed

**Disadvantages:**
- **Performance:** Slight overhead
- **Shallow only:** Must deep freeze manually
- **Inflexibility:** Cannot undo freeze/seal
- **Complexity:** Adds conceptual overhead
- **Silent failures:** May hide bugs in non-strict mode

**Best Practices:**
- Use `freeze()` for constants and configuration objects
- Use `seal()` when you need to modify values but not structure
- Always use strict mode to catch violations
- Implement deep freeze for nested objects when needed
- Document frozen/sealed objects in code
- Consider immutability libraries (Immutable.js, Immer) for complex cases
- Don't overuse - only for data that truly needs protection
- Check frozen status before attempting modifications

**Key Takeaways:**
- `Object.seal()`: Prevents adding/removing properties, allows modifications
- `Object.freeze()`: Prevents adding/removing/modifying properties (fully immutable)
- Both are shallow operations - don't affect nested objects
- Use for configuration, constants, and data integrity
- Frozen objects are sealed, but sealed objects aren't necessarily frozen
- Silent failures in non-strict mode, throws errors in strict mode
- Essential for functional programming and immutable data patterns

37. What is the difference between deep copy and shallow copy?

**Answer:**
A **shallow copy** duplicates only the top-level properties of an object, while a **deep copy** recursively duplicates all nested objects and arrays, creating completely independent copies.

**Shallow Copy:**

A shallow copy creates a new object, but nested objects/arrays are still references to the original.

```javascript
const original = {
  name: 'John',
  age: 30,
  address: {
    city: 'New York',
    zip: '10001'
  },
  hobbies: ['reading', 'gaming']
};

// Shallow copy using spread operator
const shallowCopy = { ...original };

// Top-level properties are independent
shallowCopy.name = 'Jane';
console.log(original.name);    // 'John' (unchanged)
console.log(shallowCopy.name); // 'Jane'

// ⚠️ Nested objects are still SHARED (references)
shallowCopy.address.city = 'Los Angeles';
console.log(original.address.city);    // 'Los Angeles' (changed!)
console.log(shallowCopy.address.city); // 'Los Angeles'

// ⚠️ Arrays are also SHARED
shallowCopy.hobbies.push('cooking');
console.log(original.hobbies);    // ['reading', 'gaming', 'cooking']
console.log(shallowCopy.hobbies); // ['reading', 'gaming', 'cooking']

// They share the same reference
console.log(original.address === shallowCopy.address); // true
console.log(original.hobbies === shallowCopy.hobbies); // true
```

**Deep Copy:**

A deep copy creates completely independent copies of all nested structures.

```javascript
const original = {
  name: 'John',
  age: 30,
  address: {
    city: 'New York',
    zip: '10001'
  },
  hobbies: ['reading', 'gaming']
};

// Deep copy using JSON (has limitations)
const deepCopy = JSON.parse(JSON.stringify(original));

// All levels are independent
deepCopy.name = 'Jane';
deepCopy.address.city = 'Los Angeles';
deepCopy.hobbies.push('cooking');

console.log(original.name);           // 'John' (unchanged)
console.log(original.address.city);   // 'New York' (unchanged)
console.log(original.hobbies);        // ['reading', 'gaming'] (unchanged)

console.log(deepCopy.name);           // 'Jane'
console.log(deepCopy.address.city);   // 'Los Angeles'
console.log(deepCopy.hobbies);        // ['reading', 'gaming', 'cooking']

// Different references
console.log(original.address === deepCopy.address); // false
console.log(original.hobbies === deepCopy.hobbies); // false
```

**Visual Representation:**

```
Shallow Copy:
original     →  { name: 'John', address: ──→ { city: 'NY' } }
                                              ↑
shallowCopy  →  { name: 'Jane', address: ────┘ }
                (Different object, but address points to same object)

Deep Copy:
original     →  { name: 'John', address: ──→ { city: 'NY' } }

deepCopy     →  { name: 'Jane', address: ──→ { city: 'LA' } }
                (Completely independent objects)
```

**Shallow Copy Methods:**

**1. Spread Operator (...):**
```javascript
const original = { a: 1, b: { c: 2 } };
const copy = { ...original };

copy.a = 10;
console.log(original.a); // 1 (independent)

copy.b.c = 20;
console.log(original.b.c); // 20 (shared!)
```

**2. Object.assign():**
```javascript
const original = { a: 1, b: { c: 2 } };
const copy = Object.assign({}, original);

copy.b.c = 20;
console.log(original.b.c); // 20 (shared!)
```

**3. Array.slice() / Array.concat():**
```javascript
const original = [1, 2, { a: 3 }];
const copy1 = original.slice();
const copy2 = [].concat(original);

copy1[0] = 10;
console.log(original[0]); // 1 (independent)

copy1[2].a = 30;
console.log(original[2].a); // 30 (shared!)
```

**4. Array.from():**
```javascript
const original = [1, 2, [3, 4]];
const copy = Array.from(original);

copy[0] = 10;
console.log(original[0]); // 1 (independent)

copy[2][0] = 30;
console.log(original[2][0]); // 30 (shared!)
```

**Deep Copy Methods:**

**1. JSON.parse(JSON.stringify()) - Simple but Limited:**
```javascript
const original = {
  name: 'John',
  age: 30,
  address: {
    city: 'New York'
  }
};

const deepCopy = JSON.parse(JSON.stringify(original));

// Completely independent
deepCopy.address.city = 'LA';
console.log(original.address.city); // 'New York'

// ⚠️ Limitations:
// - Loses functions
const obj1 = { 
  func: () => console.log('hello')
};
const copy1 = JSON.parse(JSON.stringify(obj1));
console.log(copy1.func); // undefined (lost!)

// - Loses undefined values
const obj2 = { a: undefined, b: 1 };
const copy2 = JSON.parse(JSON.stringify(obj2));
console.log(copy2); // { b: 1 } (a is missing)

// - Loses Dates (converts to string)
const obj3 = { date: new Date() };
const copy3 = JSON.parse(JSON.stringify(obj3));
console.log(typeof copy3.date); // 'string' (not Date object)

// - Loses RegExp, Map, Set
const obj4 = { 
  regex: /test/,
  map: new Map([['key', 'value']]),
  set: new Set([1, 2, 3])
};
const copy4 = JSON.parse(JSON.stringify(obj4));
console.log(copy4.regex); // {} (lost pattern)
console.log(copy4.map);   // {} (lost entries)
console.log(copy4.set);   // {} (lost items)

// - Cannot handle circular references
const obj5 = { a: 1 };
obj5.self = obj5;
// JSON.parse(JSON.stringify(obj5)); // TypeError: Converting circular structure
```

**2. structuredClone() - Modern Standard (Node 17+, Modern Browsers):**
```javascript
const original = {
  name: 'John',
  date: new Date(),
  regex: /test/i,
  map: new Map([['key', 'value']]),
  set: new Set([1, 2, 3]),
  nested: {
    deep: {
      value: 'deeply nested'
    }
  }
};

const deepCopy = structuredClone(original);

// Handles most types correctly
console.log(deepCopy.date instanceof Date);     // true
console.log(deepCopy.regex instanceof RegExp);  // true
console.log(deepCopy.map instanceof Map);       // true
console.log(deepCopy.set instanceof Set);       // true

// Completely independent
deepCopy.nested.deep.value = 'changed';
console.log(original.nested.deep.value); // 'deeply nested' (unchanged)

// ⚠️ Still cannot clone functions
const obj = { func: () => {} };
// structuredClone(obj); // DataCloneError: could not be cloned
```

**3. Custom Deep Clone Function:**
```javascript
function deepClone(obj, hash = new WeakMap()) {
  // Handle null or undefined
  if (obj === null || obj === undefined) return obj;
  
  // Handle primitive types
  if (typeof obj !== 'object') return obj;
  
  // Handle circular references
  if (hash.has(obj)) return hash.get(obj);
  
  // Handle Date
  if (obj instanceof Date) return new Date(obj);
  
  // Handle RegExp
  if (obj instanceof RegExp) return new RegExp(obj);
  
  // Handle Map
  if (obj instanceof Map) {
    const clonedMap = new Map();
    hash.set(obj, clonedMap);
    obj.forEach((value, key) => {
      clonedMap.set(key, deepClone(value, hash));
    });
    return clonedMap;
  }
  
  // Handle Set
  if (obj instanceof Set) {
    const clonedSet = new Set();
    hash.set(obj, clonedSet);
    obj.forEach(value => {
      clonedSet.add(deepClone(value, hash));
    });
    return clonedSet;
  }
  
  // Handle Array
  if (Array.isArray(obj)) {
    const clonedArr = [];
    hash.set(obj, clonedArr);
    obj.forEach((item, index) => {
      clonedArr[index] = deepClone(item, hash);
    });
    return clonedArr;
  }
  
  // Handle Object
  const clonedObj = Object.create(Object.getPrototypeOf(obj));
  hash.set(obj, clonedObj);
  
  Object.keys(obj).forEach(key => {
    clonedObj[key] = deepClone(obj[key], hash);
  });
  
  return clonedObj;
}

// Test with complex object
const complex = {
  name: 'John',
  date: new Date(),
  regex: /test/i,
  map: new Map([['key', 'value']]),
  set: new Set([1, 2, 3]),
  nested: {
    deep: {
      value: 'very deep'
    }
  }
};

// Handle circular reference
complex.self = complex;

const cloned = deepClone(complex);
console.log(cloned.date instanceof Date);      // true
console.log(cloned.regex instanceof RegExp);   // true
console.log(cloned.self === cloned);           // true (circular maintained)
console.log(cloned.self === complex);          // false (different object)

// Independent
cloned.nested.deep.value = 'changed';
console.log(complex.nested.deep.value);        // 'very deep' (unchanged)
```

**4. Using Lodash (Library Solution):**
```javascript
const _ = require('lodash');

const original = {
  name: 'John',
  nested: {
    deep: {
      value: 'test'
    }
  }
};

const deepCopy = _.cloneDeep(original);

deepCopy.nested.deep.value = 'changed';
console.log(original.nested.deep.value); // 'test' (unchanged)
```

**Comparison Table:**

| Method | Deep/Shallow | Functions | Dates | RegExp | Map/Set | Circular Refs | Speed |
|--------|-------------|-----------|-------|--------|---------|---------------|-------|
| Spread `{...obj}` | Shallow | ✅ | ✅ | ✅ | ✅ | ✅ | Fast |
| `Object.assign()` | Shallow | ✅ | ✅ | ✅ | ✅ | ✅ | Fast |
| `JSON` methods | Deep | ❌ | ⚠️ String | ❌ | ❌ | ❌ | Medium |
| `structuredClone()` | Deep | ❌ | ✅ | ✅ | ✅ | ✅ | Fast |
| Custom `deepClone()` | Deep | ⚠️ | ✅ | ✅ | ✅ | ✅ | Slow |
| Lodash `cloneDeep()` | Deep | ✅ | ✅ | ✅ | ✅ | ✅ | Medium |

**Production Use Cases:**

**1. State Management (Immutability):**
```javascript
// Redux-style state updates require copies
const state = {
  user: {
    name: 'John',
    preferences: {
      theme: 'dark'
    }
  }
};

// ❌ Wrong - mutates original state
state.user.preferences.theme = 'light';

// ✅ Shallow copy - works for one level
const newState1 = {
  ...state,
  user: {
    ...state.user,
    name: 'Jane'
  }
};

// ✅ Deep copy - for nested updates
const newState2 = JSON.parse(JSON.stringify(state));
newState2.user.preferences.theme = 'light';
```

**2. Caching Responses:**
```javascript
class APICache {
  constructor() {
    this.cache = new Map();
  }
  
  set(key, value) {
    // Deep clone to prevent external mutations
    this.cache.set(key, structuredClone(value));
  }
  
  get(key) {
    const cached = this.cache.get(key);
    // Return deep clone to prevent mutations
    return cached ? structuredClone(cached) : undefined;
  }
}

const cache = new APICache();
const data = { users: [{ name: 'John' }] };

cache.set('users', data);

const retrieved = cache.get('users');
retrieved.users[0].name = 'Jane';

// Original cache is unchanged
console.log(cache.get('users').users[0].name); // 'John'
```

**3. Undo/Redo Functionality:**
```javascript
class History {
  constructor(initialState) {
    this.states = [structuredClone(initialState)];
    this.currentIndex = 0;
  }
  
  saveState(state) {
    // Remove any future states if we're not at the end
    this.states.splice(this.currentIndex + 1);
    // Deep clone to preserve state
    this.states.push(structuredClone(state));
    this.currentIndex++;
  }
  
  undo() {
    if (this.currentIndex > 0) {
      this.currentIndex--;
      return structuredClone(this.states[this.currentIndex]);
    }
    return null;
  }
  
  redo() {
    if (this.currentIndex < this.states.length - 1) {
      this.currentIndex++;
      return structuredClone(this.states[this.currentIndex]);
    }
    return null;
  }
}

const editor = new History({ text: 'Hello' });
editor.saveState({ text: 'Hello World' });
editor.saveState({ text: 'Hello World!' });

console.log(editor.undo()); // { text: 'Hello World' }
console.log(editor.undo()); // { text: 'Hello' }
console.log(editor.redo()); // { text: 'Hello World' }
```

**Common Pitfalls:**

**1. Assuming Spread is Deep Copy:**
```javascript
const original = { a: { b: 1 } };
const copy = { ...original };

copy.a.b = 2;
console.log(original.a.b); // 2 (shared!) - Common mistake
```

**2. JSON Method Limitations:**
```javascript
const obj = {
  date: new Date(),
  func: () => console.log('hi'),
  undefined: undefined
};

const copy = JSON.parse(JSON.stringify(obj));
console.log(copy.date instanceof Date);  // false (it's a string)
console.log(copy.func);                  // undefined (lost)
console.log('undefined' in copy);        // false (property lost)
```

**3. Performance with Large Objects:**
```javascript
const huge = { /* very large object */ };

// Slow for large objects
console.time('JSON');
const copy1 = JSON.parse(JSON.stringify(huge));
console.timeEnd('JSON');

// Faster
console.time('structuredClone');
const copy2 = structuredClone(huge);
console.timeEnd('structuredClone');
```

**Best Practices:**
- Use **shallow copy** when you only need to modify top-level properties
- Use **deep copy** when you need complete independence from original
- Prefer `structuredClone()` for modern environments (best performance and features)
- Use JSON method only for simple objects (no functions, dates, etc.)
- Consider libraries (Lodash) for complex cloning needs
- Be aware of performance implications with large objects
- Test edge cases (circular refs, special types, functions)
- Document whether functions expect shallow or deep copies

**Key Takeaways:**
- **Shallow copy**: Duplicates top level only, nested objects are references
- **Deep copy**: Recursively duplicates everything, completely independent
- Spread operator and `Object.assign()` create shallow copies
- `structuredClone()` is the modern standard for deep cloning
- JSON method is simple but has many limitations
- Choose based on data structure complexity and performance needs
- Understanding the difference prevents subtle bugs with nested data

38. How do you clone an object in JavaScript?

**Answer:**
There are multiple ways to clone objects in JavaScript, each with different characteristics regarding shallow vs deep copying, performance, and support for various data types.

**Shallow Cloning Methods:**

**1. Spread Operator (ES6+) - Most Common:**
```javascript
const original = { a: 1, b: 2, c: 3 };
const clone = { ...original };

clone.a = 10;
console.log(original.a); // 1 (independent)
console.log(clone.a);    // 10

// ⚠️ Shallow only
const nested = { a: 1, b: { c: 2 } };
const shallowClone = { ...nested };

shallowClone.b.c = 20;
console.log(nested.b.c); // 20 (shared!)
```

**2. Object.assign():**
```javascript
const original = { a: 1, b: 2, c: 3 };
const clone = Object.assign({}, original);

// Can also merge multiple objects
const obj1 = { a: 1 };
const obj2 = { b: 2 };
const obj3 = { c: 3 };
const merged = Object.assign({}, obj1, obj2, obj3);
console.log(merged); // { a: 1, b: 2, c: 3 }

// Later properties overwrite earlier ones
const result = Object.assign({}, { a: 1 }, { a: 2 });
console.log(result.a); // 2
```

**3. Object.create() with Property Descriptors:**
```javascript
const original = { a: 1, b: 2 };

// Clone with same prototype
const clone = Object.create(
  Object.getPrototypeOf(original),
  Object.getOwnPropertyDescriptors(original)
);

console.log(clone.a); // 1
console.log(Object.getPrototypeOf(clone) === Object.getPrototypeOf(original)); // true

// Preserves property descriptors
const obj = {};
Object.defineProperty(obj, 'readOnly', {
  value: 42,
  writable: false,
  enumerable: true,
  configurable: false
});

const cloned = Object.create(
  Object.getPrototypeOf(obj),
  Object.getOwnPropertyDescriptors(obj)
);

cloned.readOnly = 100; // Fails silently (non-writable)
console.log(cloned.readOnly); // 42
```

**4. Array Cloning:**
```javascript
const original = [1, 2, 3, 4, 5];

// Method 1: slice()
const clone1 = original.slice();

// Method 2: spread
const clone2 = [...original];

// Method 3: Array.from()
const clone3 = Array.from(original);

// Method 4: concat()
const clone4 = [].concat(original);

// Method 5: map()
const clone5 = original.map(x => x);

// All create shallow copies
clone1[0] = 10;
console.log(original[0]); // 1 (independent)

// ⚠️ Nested arrays are still shared
const nested = [1, 2, [3, 4]];
const shallowClone = [...nested];

shallowClone[2][0] = 30;
console.log(nested[2][0]); // 30 (shared!)
```

**Deep Cloning Methods:**

**1. structuredClone() - Modern Standard (Recommended):**
```javascript
const original = {
  name: 'John',
  age: 30,
  date: new Date(),
  regex: /test/i,
  map: new Map([['key', 'value']]),
  set: new Set([1, 2, 3]),
  buffer: new ArrayBuffer(8),
  nested: {
    deep: {
      value: [1, 2, 3]
    }
  }
};

// ✅ Handles most types correctly
const deepClone = structuredClone(original);

deepClone.nested.deep.value.push(4);
console.log(original.nested.deep.value); // [1, 2, 3] (unchanged)

// Preserves types
console.log(deepClone.date instanceof Date);      // true
console.log(deepClone.regex instanceof RegExp);   // true
console.log(deepClone.map instanceof Map);        // true
console.log(deepClone.set instanceof Set);        // true

// Handles circular references
const circular = { a: 1 };
circular.self = circular;
const clonedCircular = structuredClone(circular);
console.log(clonedCircular.self === clonedCircular); // true

// ⚠️ Cannot clone functions
const withFunc = { func: () => {} };
// structuredClone(withFunc); // DataCloneError
```

**2. JSON.parse(JSON.stringify()) - Simple but Limited:**
```javascript
const original = {
  name: 'John',
  age: 30,
  address: {
    city: 'New York',
    coordinates: {
      lat: 40.7128,
      lng: -74.0060
    }
  }
};

const deepClone = JSON.parse(JSON.stringify(original));

deepClone.address.coordinates.lat = 50;
console.log(original.address.coordinates.lat); // 40.7128 (unchanged)

// ⚠️ Limitations:
const problematic = {
  func: () => console.log('hi'),    // Lost
  date: new Date(),                 // Becomes string
  undefined: undefined,             // Lost
  infinity: Infinity,               // Becomes null
  nan: NaN,                        // Becomes null
  regex: /test/,                   // Becomes {}
  symbol: Symbol('test'),          // Lost
  map: new Map([['a', 1]]),        // Becomes {}
  set: new Set([1, 2, 3])         // Becomes {}
};

const cloned = JSON.parse(JSON.stringify(problematic));
console.log(cloned.func);      // undefined
console.log(typeof cloned.date); // 'string'
console.log('undefined' in cloned); // false
```

**3. Custom Deep Clone Function:**
```javascript
function deepClone(obj, hash = new WeakMap()) {
  // Primitives and null
  if (obj === null || typeof obj !== 'object') {
    return obj;
  }
  
  // Handle circular references
  if (hash.has(obj)) {
    return hash.get(obj);
  }
  
  // Handle specific object types
  if (obj instanceof Date) {
    return new Date(obj.getTime());
  }
  
  if (obj instanceof RegExp) {
    return new RegExp(obj.source, obj.flags);
  }
  
  if (obj instanceof Map) {
    const clonedMap = new Map();
    hash.set(obj, clonedMap);
    for (const [key, value] of obj) {
      clonedMap.set(key, deepClone(value, hash));
    }
    return clonedMap;
  }
  
  if (obj instanceof Set) {
    const clonedSet = new Set();
    hash.set(obj, clonedSet);
    for (const value of obj) {
      clonedSet.add(deepClone(value, hash));
    }
    return clonedSet;
  }
  
  // Handle Arrays
  if (Array.isArray(obj)) {
    const clonedArr = [];
    hash.set(obj, clonedArr);
    for (let i = 0; i < obj.length; i++) {
      clonedArr[i] = deepClone(obj[i], hash);
    }
    return clonedArr;
  }
  
  // Handle plain objects
  const clonedObj = Object.create(Object.getPrototypeOf(obj));
  hash.set(obj, clonedObj);
  
  for (const key of Object.keys(obj)) {
    clonedObj[key] = deepClone(obj[key], hash);
  }
  
  return clonedObj;
}

// Usage
const complex = {
  name: 'John',
  date: new Date(),
  map: new Map([['key', 'value']]),
  nested: {
    arr: [1, 2, { deep: 3 }]
  }
};

complex.circular = complex; // Circular reference

const cloned = deepClone(complex);
cloned.nested.arr[2].deep = 99;

console.log(complex.nested.arr[2].deep); // 3 (unchanged)
console.log(cloned.circular === cloned); // true (maintains circular ref)
console.log(cloned.date instanceof Date); // true
console.log(cloned.map instanceof Map);   // true
```

**4. Library Solutions:**

**Lodash cloneDeep:**
```javascript
const _ = require('lodash');

const original = {
  name: 'John',
  nested: {
    deep: {
      value: [1, 2, 3]
    }
  },
  func: () => console.log('hi')
};

const deepClone = _.cloneDeep(original);

deepClone.nested.deep.value.push(4);
console.log(original.nested.deep.value); // [1, 2, 3] (unchanged)

// ✅ Preserves functions
console.log(typeof deepClone.func); // 'function'
```

**Ramda clone:**
```javascript
const R = require('ramda');

const original = { a: { b: { c: 1 } } };
const cloned = R.clone(original);

cloned.a.b.c = 2;
console.log(original.a.b.c); // 1 (unchanged)
```

**Special Cases:**

**1. Cloning with Getters/Setters:**
```javascript
const original = {
  _value: 42,
  get value() {
    return this._value;
  },
  set value(val) {
    this._value = val;
  }
};

// Spread operator loses getters/setters
const spreadClone = { ...original };
console.log(Object.getOwnPropertyDescriptor(spreadClone, 'value'));
// { value: 42, writable: true, enumerable: true, configurable: true }
// (Converted to regular property!)

// ✅ Preserve getters/setters
const properClone = Object.defineProperties(
  {},
  Object.getOwnPropertyDescriptors(original)
);

console.log(Object.getOwnPropertyDescriptor(properClone, 'value'));
// { get: [Function: get], set: [Function: set], enumerable: true, configurable: true }
// (Getters/setters preserved!)
```

**2. Cloning Class Instances:**
```javascript
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
  
  greet() {
    return `Hello, I'm ${this.name}`;
  }
  
  clone() {
    return new Person(this.name, this.age);
  }
}

const john = new Person('John', 30);

// ❌ Loses class methods
const spreadClone = { ...john };
console.log(spreadClone.greet); // undefined

// ❌ Still loses methods
const jsonClone = JSON.parse(JSON.stringify(john));
console.log(jsonClone.greet); // undefined

// ✅ Use custom clone method
const properClone = john.clone();
console.log(properClone.greet()); // 'Hello, I'm John'
console.log(properClone instanceof Person); // true

// ✅ Or preserve prototype
const protoClone = Object.assign(
  Object.create(Object.getPrototypeOf(john)),
  john
);
console.log(protoClone.greet()); // 'Hello, I'm John'
console.log(protoClone instanceof Person); // true
```

**3. Cloning with Symbols:**
```javascript
const sym = Symbol('key');
const original = {
  [sym]: 'symbol value',
  regular: 'regular value'
};

// Spread doesn't copy symbol properties
const spreadClone = { ...original };
console.log(spreadClone[sym]); // undefined

// Object.assign doesn't copy symbol properties
const assignClone = Object.assign({}, original);
console.log(assignClone[sym]); // undefined

// ✅ Copy symbols manually
const properClone = {};
Object.assign(properClone, original);
Object.getOwnPropertySymbols(original).forEach(s => {
  properClone[s] = original[s];
});
console.log(properClone[sym]); // 'symbol value'

// ✅ Or use this helper
function cloneWithSymbols(obj) {
  const clone = { ...obj };
  Object.getOwnPropertySymbols(obj).forEach(sym => {
    clone[sym] = obj[sym];
  });
  return clone;
}
```

**Performance Comparison:**

```javascript
const large = { /* large nested object */ };

// Fastest - shallow copy
console.time('spread');
const clone1 = { ...large };
console.timeEnd('spread'); // ~0.1ms

// Fast - shallow copy
console.time('Object.assign');
const clone2 = Object.assign({}, large);
console.timeEnd('Object.assign'); // ~0.15ms

// Medium - deep copy
console.time('JSON');
const clone3 = JSON.parse(JSON.stringify(large));
console.timeEnd('JSON'); // ~2ms

// Fast - deep copy (modern)
console.time('structuredClone');
const clone4 = structuredClone(large);
console.timeEnd('structuredClone'); // ~1ms

// Slow - deep copy (comprehensive)
console.time('deepClone');
const clone5 = deepClone(large);
console.timeEnd('deepClone'); // ~5ms
```

**Method Selection Guide:**

```javascript
// Simple object, top-level only → Spread operator
const simple = { a: 1, b: 2 };
const clone1 = { ...simple };

// Nested object, no special types → JSON
const nested = { a: { b: { c: 1 } } };
const clone2 = JSON.parse(JSON.stringify(nested));

// Nested with Dates, Maps, Sets → structuredClone
const complex = { date: new Date(), map: new Map() };
const clone3 = structuredClone(complex);

// With functions, class instances → Custom or library
const withMethods = new MyClass();
const clone4 = _.cloneDeep(withMethods);

// Array of primitives → Spread or slice
const arr = [1, 2, 3];
const clone5 = [...arr]; // or arr.slice()

// Array of objects → structuredClone
const arrOfObj = [{ a: 1 }, { b: 2 }];
const clone6 = structuredClone(arrOfObj);
```

**Production Use Cases:**

**1. Redux State Updates:**
```javascript
// Immutable state updates
const state = {
  user: {
    name: 'John',
    preferences: {
      theme: 'dark'
    }
  }
};

// ✅ Correct way - create new objects at each level
const newState = {
  ...state,
  user: {
    ...state.user,
    preferences: {
      ...state.user.preferences,
      theme: 'light'
    }
  }
};
```

**2. Form Data Backup:**
```javascript
class FormManager {
  constructor(initialData) {
    this.currentData = initialData;
    this.backup = structuredClone(initialData);
  }
  
  updateField(field, value) {
    this.currentData[field] = value;
  }
  
  reset() {
    this.currentData = structuredClone(this.backup);
  }
  
  hasChanges() {
    return JSON.stringify(this.currentData) !== JSON.stringify(this.backup);
  }
}

const form = new FormManager({ name: 'John', email: 'john@example.com' });
form.updateField('name', 'Jane');
console.log(form.hasChanges()); // true
form.reset();
console.log(form.currentData.name); // 'John' (restored)
```

**3. Default Configuration:**
```javascript
const defaultConfig = {
  timeout: 5000,
  retries: 3,
  headers: {
    'Content-Type': 'application/json'
  }
};

function createClient(userConfig = {}) {
  // Merge with defaults, ensuring original defaults unchanged
  return {
    ...structuredClone(defaultConfig),
    ...userConfig
  };
}

const client1 = createClient({ timeout: 10000 });
const client2 = createClient({ retries: 5 });

console.log(client1.timeout); // 10000
console.log(client2.timeout); // 5000 (default)
```

**Best Practices:**
- Use **spread operator** for simple shallow copies (most common case)
- Use **structuredClone()** for deep copies in modern environments
- Use **JSON method** only for plain objects without special types
- Implement **custom clone method** in classes for proper cloning
- Consider **libraries** (Lodash) for complex cloning requirements
- Always test cloning with your actual data structures
- Document whether functions expect original or cloned data
- Be aware of performance implications with large objects

**Key Takeaways:**
- Multiple cloning methods exist for different use cases
- Spread operator and `Object.assign()` create shallow copies
- `structuredClone()` is the modern standard for deep cloning
- JSON method is simple but has many limitations (functions, dates, etc.)
- Choose method based on data complexity and required features
- Custom deep clone functions offer maximum control but complexity
- Libraries provide battle-tested solutions for edge cases
- Understanding cloning is essential for immutability patterns
