## Functions

11. What is a function declaration vs function expression?

**Answer:**

**Function Declaration:**
```javascript
// Named function that is hoisted
function greet(name) {
  return `Hello, ${name}!`;
}
```

**Function Expression:**
```javascript
// Function assigned to a variable (not hoisted)
const greet = function(name) {
  return `Hello, ${name}!`;
};

// Named function expression (useful for recursion/debugging)
const factorial = function fact(n) {
  return n <= 1 ? 1 : n * fact(n - 1);
};
```

**Key Differences:**

| Aspect | Function Declaration | Function Expression |
|--------|---------------------|---------------------|
| **Hoisting** | Fully hoisted | Only variable hoisted (not function) |
| **When Available** | Before declaration in code | Only after assignment |
| **Name Required** | Yes | Optional (can be anonymous) |
| **Usage Context** | Statement | Can be part of expression |

**Hoisting Behavior:**
```javascript
// ✅ Works - function declaration hoisted
console.log(declared()); // "I'm declared"
function declared() {
  return "I'm declared";
}

// ❌ Error - function expression not hoisted
console.log(expressed()); // TypeError: expressed is not a function
const expressed = function() {
  return "I'm expressed";
};
```

**Use Cases:**

**Function Declaration - When you need hoisting:**
```javascript
// Mutual recursion (functions call each other)
function isEven(n) {
  return n === 0 ? true : isOdd(n - 1);
}

function isOdd(n) {
  return n === 0 ? false : isEven(n - 1);
}
```

**Function Expression - More flexible:**
```javascript
// Conditional function definition
const operation = condition ? 
  function(a, b) { return a + b; } : 
  function(a, b) { return a - b; };

// Callback functions
setTimeout(function() {
  console.log("Timeout");
}, 1000);

// IIFE (Immediately Invoked)
(function() {
  console.log("Running immediately");
})();
```

**Production Best Practices:**
```javascript
// ✅ Use function declarations for top-level functions
function processPayment(amount) {
  // Public API functions
}

// ✅ Use function expressions for callbacks/handlers
button.addEventListener('click', function handleClick(e) {
  // Named for better stack traces
});

// ✅ Arrow functions for short callbacks (covered next)
array.map(item => item * 2);

// ✅ Named function expressions for recursion
const traverse = function walk(node) {
  if (!node) return;
  walk(node.left);
  walk(node.right);
};
```

12. What are arrow functions and how are they different from regular functions?

**Answer:**
Arrow functions are a concise syntax for writing function expressions introduced in ES6. They have significant differences from regular functions, particularly regarding `this` binding.

**Syntax Variations:**
```javascript
// Traditional function
const add = function(a, b) {
  return a + b;
};

// Arrow function - full syntax
const add = (a, b) => {
  return a + b;
};

// Implicit return (single expression)
const add = (a, b) => a + b;

// Single parameter (no parentheses needed)
const square = x => x * x;

// No parameters
const greet = () => "Hello!";

// Returning object literal (wrap in parentheses)
const makePerson = (name, age) => ({ name, age });
```

**Key Differences:**

| Feature | Regular Function | Arrow Function |
|---------|-----------------|----------------|
| **`this` binding** | Dynamic (depends on call) | Lexical (from enclosing scope) |
| **`arguments` object** | Yes | No (use rest parameters) |
| **Constructor** | Can use `new` | Cannot use `new` |
| **`prototype` property** | Yes | No |
| **Method syntax** | Works well | Not recommended |
| **Hoisting** | Declaration: yes | No (expression) |

**1. `this` Binding (Most Important Difference):**
```javascript
// Regular function - 'this' depends on how it's called
const person = {
  name: "John",
  regularGreet: function() {
    console.log(`Hello, ${this.name}`);
  },
  arrowGreet: () => {
    console.log(`Hello, ${this.name}`);
  }
};

person.regularGreet(); // "Hello, John" (this = person)
person.arrowGreet();   // "Hello, undefined" (this = global/window)

// Common use case: Event handlers
class Button {
  constructor() {
    this.clicks = 0;
  }
  
  // ❌ Regular function loses context
  handleClickBad() {
    setTimeout(function() {
      this.clicks++; // 'this' is undefined or window
    }, 100);
  }
  
  // ✅ Arrow function preserves context
  handleClickGood() {
    setTimeout(() => {
      this.clicks++; // 'this' is Button instance
    }, 100);
  }
}
```

**2. No `arguments` Object:**
```javascript
// Regular function
function regularSum() {
  return Array.from(arguments).reduce((a, b) => a + b, 0);
}
regularSum(1, 2, 3); // 6

// Arrow function - use rest parameters
const arrowSum = (...args) => args.reduce((a, b) => a + b, 0);
arrowSum(1, 2, 3); // 6
```

**3. Cannot be Used as Constructors:**
```javascript
// Regular function
function Person(name) {
  this.name = name;
}
const john = new Person("John"); // ✅ Works

// Arrow function
const PersonArrow = (name) => {
  this.name = name;
};
// const jane = new PersonArrow("Jane"); // ❌ TypeError: not a constructor
```

**Production Use Cases:**

**✅ When to Use Arrow Functions:**
```javascript
// 1. Array methods
const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map(n => n * 2);
const evens = numbers.filter(n => n % 2 === 0);
const sum = numbers.reduce((acc, n) => acc + n, 0);

// 2. Callbacks with context preservation
class DataFetcher {
  constructor() {
    this.data = [];
  }
  
  fetchData() {
    fetch('/api/data')
      .then(response => response.json())
      .then(data => {
        this.data = data; // 'this' correctly refers to DataFetcher
      });
  }
}

// 3. Short, simple functions
const isEven = n => n % 2 === 0;
const double = x => x * 2;

// 4. Event listeners in classes
class Component {
  mount() {
    button.addEventListener('click', () => {
      this.handleClick(); // 'this' is Component instance
    });
  }
}
```

**❌ When NOT to Use Arrow Functions:**
```javascript
// 1. Object methods
const person = {
  name: "John",
  // ❌ Bad - 'this' doesn't refer to person
  greet: () => {
    console.log(`Hello, ${this.name}`); // undefined
  },
  // ✅ Good - use method shorthand or regular function
  greet() {
    console.log(`Hello, ${this.name}`); // "John"
  }
};

// 2. Prototype methods
// ❌ Bad
Person.prototype.getName = () => this.name; // Won't work

// ✅ Good
Person.prototype.getName = function() { return this.name; };

// 3. Event handlers needing 'this' to be the element
// ❌ Bad
button.addEventListener('click', () => {
  this.classList.toggle('active'); // 'this' is not the button
});

// ✅ Good
button.addEventListener('click', function() {
  this.classList.toggle('active'); // 'this' is the button
});

// 4. Functions needing 'arguments'
// ❌ Bad - no arguments object
const logAll = () => console.log(arguments); // ReferenceError

// ✅ Good - use rest parameters
const logAll = (...args) => console.log(args);
```

13. What is the difference between call, apply, and bind?

**Answer:**
`call`, `apply`, and `bind` are methods on function objects that allow you to explicitly set the `this` context and pass arguments. They are essential for controlling function execution context.

**call() - Immediate invocation with individual arguments:**
```javascript
function.call(thisArg, arg1, arg2, ...)
```

**apply() - Immediate invocation with array of arguments:**
```javascript
function.apply(thisArg, [arg1, arg2, ...])
```

**bind() - Returns new function with bound context (delayed invocation):**
```javascript
const boundFunction = function.bind(thisArg, arg1, arg2, ...)
```

**Detailed Examples:**

**call() - Execute immediately with comma-separated args:**
```javascript
function greet(greeting, punctuation) {
  return `${greeting}, ${this.name}${punctuation}`;
}

const person = { name: "John" };

greet.call(person, "Hello", "!");  // "Hello, John!"
greet.call(person, "Hi", ".");     // "Hi, John."
```

**apply() - Execute immediately with array of args:**
```javascript
function greet(greeting, punctuation) {
  return `${greeting}, ${this.name}${punctuation}`;
}

const person = { name: "John" };
const args = ["Hello", "!"];

greet.apply(person, args);  // "Hello, John!"
greet.apply(person, ["Hi", "."]); // "Hi, John."
```

**bind() - Create bound function for later:**
```javascript
function greet(greeting, punctuation) {
  return `${greeting}, ${this.name}${punctuation}`;
}

const person = { name: "John" };

// Create bound function
const greetJohn = greet.bind(person, "Hello");
greetJohn("!");  // "Hello, John!"
greetJohn(".");  // "Hello, John."

// Fully bound
const sayHelloJohn = greet.bind(person, "Hello", "!");
sayHelloJohn();  // "Hello, John!"
```

**Comparison Table:**

| Method | Invocation | Arguments Format | Returns | Use Case |
|--------|-----------|------------------|---------|----------|
| **call** | Immediate | Individual (comma-separated) | Function result | One-time execution |
| **apply** | Immediate | Array | Function result | Dynamic arg count |
| **bind** | Delayed | Individual (comma-separated) | New function | Event handlers, callbacks |

**Production Use Cases:**

**1. Borrowing Methods:**
```javascript
// Array methods on array-like objects
const arrayLike = { 0: 'a', 1: 'b', 2: 'c', length: 3 };

// Using call
const arr = Array.prototype.slice.call(arrayLike);
console.log(arr); // ['a', 'b', 'c']

// Modern alternative
const arr2 = Array.from(arrayLike);

// NodeList to Array
const divs = document.querySelectorAll('div');
const divArray = Array.prototype.map.call(divs, div => div.textContent);
```

**2. Finding Min/Max with apply:**
```javascript
const numbers = [5, 6, 2, 3, 7];

// apply is useful when you have an array
const max = Math.max.apply(null, numbers); // 7
const min = Math.min.apply(null, numbers); // 2

// Modern alternative with spread
const max2 = Math.max(...numbers);
```

**3. Function Composition with bind:**
```javascript
function multiply(a, b) {
  return a * b;
}

// Create specialized functions
const double = multiply.bind(null, 2);
const triple = multiply.bind(null, 3);

console.log(double(5));  // 10
console.log(triple(5));  // 15
```

**4. Event Handlers (Most Common use of bind):**
```javascript
class Counter {
  constructor() {
    this.count = 0;
    // ❌ Without bind, 'this' would be the button
    // button.onclick = this.increment; 
    
    // ✅ bind preserves 'this' context
    button.onclick = this.increment.bind(this);
  }
  
  increment() {
    this.count++;
    console.log(this.count);
  }
}

// Modern alternative: Arrow functions in constructor
class CounterModern {
  constructor() {
    this.count = 0;
    button.onclick = () => this.increment();
  }
  
  increment() {
    this.count++;
  }
}
```

**5. Partial Application (Currying):**
```javascript
function log(level, message, timestamp) {
  console.log(`[${timestamp}] ${level}: ${message}`);
}

// Create specialized loggers
const errorLog = log.bind(null, 'ERROR');
const infoLog = log.bind(null, 'INFO');

errorLog('Database connection failed', Date.now());
infoLog('User logged in', Date.now());
```

**6. Inheritance and Super Calls:**
```javascript
function Animal(name) {
  this.name = name;
}

function Dog(name, breed) {
  // Call parent constructor with current context
  Animal.call(this, name);
  this.breed = breed;
}

const myDog = new Dog('Buddy', 'Golden Retriever');
console.log(myDog.name); // 'Buddy'
```

**Performance & Modern Considerations:**
```javascript
// bind creates a new function each time (memory overhead)
// ❌ Bad in React - creates new function on every render
class Component {
  render() {
    return <button onClick={this.handleClick.bind(this)}>Click</button>;
  }
}

// ✅ Good - bind once in constructor
class Component {
  constructor() {
    this.handleClick = this.handleClick.bind(this);
  }
  render() {
    return <button onClick={this.handleClick}>Click</button>;
  }
}

// ✅ Better - use arrow function (lexical this)
class Component {
  handleClick = () => {
    // 'this' is automatically bound
  }
}
```

**Key Takeaways:**
- Use `call` when you know the exact arguments to pass
- Use `apply` when arguments are in an array (less common with ES6 spread)
- Use `bind` when you need a function with fixed context for later use
- Arrow functions often replace `bind` in modern code
- Remember: You can't rebind arrow functions (they ignore call/apply/bind for `this`)

14. What are higher-order functions?

**Answer:**
A higher-order function is a function that either:
1. **Takes one or more functions as arguments** (callbacks), OR
2. **Returns a function as its result**

This concept is fundamental to functional programming and makes JavaScript code more modular, reusable, and expressive.

**Type 1: Functions that Accept Functions (Callbacks):**

```javascript
// Built-in higher-order functions
const numbers = [1, 2, 3, 4, 5];

// map: transforms each element
numbers.map(n => n * 2);  // [2, 4, 6, 8, 10]

// filter: selects elements
numbers.filter(n => n > 3);  // [4, 5]

// reduce: accumulates values
numbers.reduce((sum, n) => sum + n, 0);  // 15

// sort: orders elements
numbers.sort((a, b) => b - a);  // [5, 4, 3, 2, 1]

// forEach: iterates
numbers.forEach(n => console.log(n));
```

**Type 2: Functions that Return Functions:**

```javascript
// Function factory
function createMultiplier(multiplier) {
  return function(number) {
    return number * multiplier;
  };
}

const double = createMultiplier(2);
const triple = createMultiplier(3);

double(5);  // 10
triple(5);  // 15

// Arrow function version
const createMultiplier = (multiplier) => (number) => number * multiplier;
```

**Production Use Cases:**

**1. Data Transformation Pipelines:**
```javascript
// Complex data processing
const users = [
  { name: 'John', age: 25, active: true },
  { name: 'Jane', age: 30, active: false },
  { name: 'Bob', age: 35, active: true }
];

const activeUserNames = users
  .filter(user => user.active)           // Select active users
  .map(user => user.name)                // Extract names
  .map(name => name.toUpperCase());      // Transform to uppercase

// ['JOHN', 'BOB']
```

**2. Custom Higher-Order Functions:**
```javascript
// Retry mechanism
function retry(fn, maxAttempts = 3, delay = 1000) {
  return async function(...args) {
    for (let i = 0; i < maxAttempts; i++) {
      try {
        return await fn(...args);
      } catch (error) {
        if (i === maxAttempts - 1) throw error;
        await new Promise(resolve => setTimeout(resolve, delay));
      }
    }
  };
}

const fetchWithRetry = retry(fetch, 3, 2000);
await fetchWithRetry('/api/data');

// Logging decorator
function withLogging(fn) {
  return function(...args) {
    console.log(`Calling ${fn.name} with`, args);
    const result = fn(...args);
    console.log(`${fn.name} returned`, result);
    return result;
  };
}

const add = (a, b) => a + b;
const addWithLog = withLogging(add);
addWithLog(2, 3); // Logs and returns 5
```

**3. Event Handler Composition:**
```javascript
// Composing middleware-like handlers
function compose(...functions) {
  return function(value) {
    return functions.reduceRight((acc, fn) => fn(acc), value);
  };
}

const addTax = price => price * 1.1;
const addShipping = price => price + 5;
const round = price => Math.round(price * 100) / 100;

const calculateTotal = compose(round, addShipping, addTax);
calculateTotal(100); // 115
```

**4. Authorization & Validation:**
```javascript
// Higher-order function for access control
function requireAuth(handler) {
  return function(req, res) {
    if (!req.user) {
      return res.status(401).json({ error: 'Unauthorized' });
    }
    return handler(req, res);
  };
}

function requireRole(role) {
  return function(handler) {
    return function(req, res) {
      if (req.user.role !== role) {
        return res.status(403).json({ error: 'Forbidden' });
      }
      return handler(req, res);
    };
  };
}

// Usage
const getAdminData = requireRole('admin')(requireAuth(handler));
```

**5. Memoization (Caching):**
```javascript
function memoize(fn) {
  const cache = new Map();
  return function(...args) {
    const key = JSON.stringify(args);
    if (cache.has(key)) {
      console.log('Cache hit');
      return cache.get(key);
    }
    const result = fn(...args);
    cache.set(key, result);
    return result;
  };
}

// Expensive calculation
function fibonacci(n) {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
}

const fastFib = memoize(fibonacci);
fastFib(40); // Slow first time
fastFib(40); // Instant second time
```

**6. Debounce & Throttle (Covered in detail later):**
```javascript
// Debounce - delay execution until quiet period
function debounce(fn, delay) {
  let timeoutId;
  return function(...args) {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => fn(...args), delay);
  };
}

const searchAPI = debounce(query => {
  fetch(`/api/search?q=${query}`);
}, 300);

input.addEventListener('input', e => searchAPI(e.target.value));
```

**Benefits in Production:**
- **Code Reusability:** Write generic behavior once, apply anywhere
- **Separation of Concerns:** Business logic separate from cross-cutting concerns (logging, auth, caching)
- **Composition:** Build complex behavior from simple functions
- **Testability:** Each function can be tested in isolation
- **Declarative Code:** Express "what" not "how"

**Common Patterns:**
```javascript
// Pipe (left to right composition)
const pipe = (...fns) => x => fns.reduce((v, f) => f(v), x);

// Curry (partial application)
const curry = (fn) => {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn.apply(this, args);
    }
    return (...moreArgs) => curried(...args, ...moreArgs);
  };
};

// Once (execute only once)
function once(fn) {
  let called = false;
  let result;
  return function(...args) {
    if (!called) {
      called = true;
      result = fn(...args);
    }
    return result;
  };
}

const initializeApp = once(() => {
  console.log('App initialized');
  return { initialized: true };
});
```

15. What is a callback function?

**Answer:**
A callback function is a function passed as an argument to another function, to be executed at a later time. Callbacks are the foundation of asynchronous programming in JavaScript and enable event-driven architecture.

**Basic Concept:**
```javascript
// Simple callback example
function greet(name, callback) {
  const message = `Hello, ${name}!`;
  callback(message);
}

greet('John', (msg) => {
  console.log(msg);  // "Hello, John!"
});
```

**Types of Callbacks:**

**1. Synchronous Callbacks:**
Executed immediately in the current call stack.

```javascript
// Array methods use synchronous callbacks
const numbers = [1, 2, 3, 4, 5];

numbers.forEach((num) => {
  console.log(num);  // Executes immediately
});

const doubled = numbers.map((num) => num * 2);

// Custom synchronous callback
function processArray(arr, callback) {
  const result = [];
  for (let item of arr) {
    result.push(callback(item));
  }
  return result;
}

processArray([1, 2, 3], x => x * 2);  // [2, 4, 6]
```

**2. Asynchronous Callbacks:**
Executed later, outside the current call stack (via event loop).

```javascript
// setTimeout - executes callback after delay
setTimeout(() => {
  console.log('Delayed message');
}, 1000);

// Event listeners
button.addEventListener('click', (event) => {
  console.log('Button clicked');
});

// File operations (Node.js)
const fs = require('fs');
fs.readFile('file.txt', 'utf8', (error, data) => {
  if (error) {
    console.error(error);
    return;
  }
  console.log(data);
});

// HTTP requests
fetch('/api/data')
  .then(response => response.json())
  .then(data => console.log(data))
  .catch(error => console.error(error));
```

**Error-First Callback Pattern (Node.js Convention):**
```javascript
// Convention: First parameter is error, second is result
function readUserData(userId, callback) {
  database.query('SELECT * FROM users WHERE id = ?', [userId], (error, results) => {
    if (error) {
      return callback(error, null);  // Pass error, null data
    }
    callback(null, results);  // Pass null error, actual data
  });
}

// Usage
readUserData(123, (error, user) => {
  if (error) {
    console.error('Failed to fetch user:', error);
    return;
  }
  console.log('User:', user);
});
```

**Callback Hell (Pyramid of Doom):**
The main problem with callbacks is nesting, which leads to hard-to-read code.

```javascript
// ❌ Bad - Deeply nested callbacks
getData((error, data) => {
  if (error) {
    handleError(error);
  } else {
    processData(data, (error, processed) => {
      if (error) {
        handleError(error);
      } else {
        saveData(processed, (error, result) => {
          if (error) {
            handleError(error);
          } else {
            sendNotification(result, (error, sent) => {
              if (error) {
                handleError(error);
              } else {
                console.log('All done!');
              }
            });
          }
        });
      }
    });
  }
});
```

**Solutions to Callback Hell:**

**1. Named Functions:**
```javascript
// ✅ Better - Extract functions
function handleGetData(error, data) {
  if (error) return handleError(error);
  processData(data, handleProcessData);
}

function handleProcessData(error, processed) {
  if (error) return handleError(error);
  saveData(processed, handleSaveData);
}

function handleSaveData(error, result) {
  if (error) return handleError(error);
  sendNotification(result, handleNotification);
}

getData(handleGetData);
```

**2. Promises (Modern Solution):**
```javascript
// ✅ Best - Use Promises
getData()
  .then(data => processData(data))
  .then(processed => saveData(processed))
  .then(result => sendNotification(result))
  .then(() => console.log('All done!'))
  .catch(error => handleError(error));

// Even better - async/await
async function performOperations() {
  try {
    const data = await getData();
    const processed = await processData(data);
    const result = await saveData(processed);
    await sendNotification(result);
    console.log('All done!');
  } catch (error) {
    handleError(error);
  }
}
```

**Production Use Cases:**

**1. Event Handling:**
```javascript
// DOM events
document.getElementById('btn').addEventListener('click', function(e) {
  e.preventDefault();
  console.log('Clicked at:', e.clientX, e.clientY);
});

// Custom events
class EventEmitter {
  constructor() {
    this.events = {};
  }
  
  on(event, callback) {
    if (!this.events[event]) {
      this.events[event] = [];
    }
    this.events[event].push(callback);
  }
  
  emit(event, data) {
    if (this.events[event]) {
      this.events[event].forEach(callback => callback(data));
    }
  }
}

const emitter = new EventEmitter();
emitter.on('user:login', (user) => {
  console.log(`${user.name} logged in`);
});
```

**2. Array Iteration:**
```javascript
// Data transformation
const users = [
  { name: 'John', age: 25 },
  { name: 'Jane', age: 30 }
];

// Callback for filtering
const adults = users.filter(user => user.age >= 18);

// Callback for mapping
const names = users.map(user => user.name);

// Callback for sorting
const sorted = users.sort((a, b) => a.age - b.age);
```

**3. Middleware Pattern:**
```javascript
// Express.js style middleware
function logger(req, res, next) {
  console.log(`${req.method} ${req.url}`);
  next();  // Call next callback in chain
}

function authenticate(req, res, next) {
  if (req.headers.authorization) {
    next();  // Continue
  } else {
    res.status(401).send('Unauthorized');
  }
}

app.use(logger);
app.use(authenticate);
app.get('/api/data', (req, res) => {
  res.json({ data: 'Protected' });
});
```

**Best Practices:**
- Always handle errors in async callbacks
- Avoid deep nesting (use Promises/async-await)
- Use named functions for better stack traces
- Follow error-first callback convention in Node.js
- Consider using Promises or async/await for complex async flows
- Be mindful of `this` binding in callbacks (use arrow functions when needed)

**Memory Considerations:**
```javascript
// ⚠️ Memory leak - callback holds reference to large data
function processLargeData() {
  const largeArray = new Array(1000000).fill('data');
  
  setTimeout(() => {
    // largeArray is still in memory due to closure
    console.log(largeArray[0]);
  }, 5000);
}

// ✅ Better - clear reference when done
function processLargeData() {
  let largeArray = new Array(1000000).fill('data');
  
  setTimeout(() => {
    console.log(largeArray[0]);
    largeArray = null;  // Allow garbage collection
  }, 5000);
}
```

16. What is function currying?

**Answer:**
Currying is a functional programming technique where a function with multiple arguments is transformed into a sequence of functions, each taking a single argument. Instead of calling `f(a, b, c)`, you call `f(a)(b)(c)`.

**Basic Example:**
```javascript
// Regular function
function add(a, b, c) {
  return a + b + c;
}
add(1, 2, 3); // 6

// Curried version
function curriedAdd(a) {
  return function(b) {
    return function(c) {
      return a + b + c;
    };
  };
}
curriedAdd(1)(2)(3); // 6

// Arrow function syntax (more concise)
const curriedAdd = a => b => c => a + b + c;
curriedAdd(1)(2)(3); // 6
```

**How It Works:**
Each function returns another function until all arguments are collected, then the final computation is performed.

```javascript
const step1 = curriedAdd(1);    // Returns function expecting b
const step2 = step1(2);         // Returns function expecting c
const result = step2(3);        // Returns final result: 6
```

**Generic Curry Function:**
```javascript
// Converts any function into a curried version
function curry(fn) {
  return function curried(...args) {
    // If we have all arguments, call the function
    if (args.length >= fn.length) {
      return fn.apply(this, args);
    }
    // Otherwise, return a function that collects more arguments
    return function(...nextArgs) {
      return curried(...args, ...nextArgs);
    };
  };
}

// Usage
function multiply(a, b, c) {
  return a * b * c;
}

const curriedMultiply = curry(multiply);
curriedMultiply(2)(3)(4);        // 24
curriedMultiply(2, 3)(4);        // 24 (can pass multiple args)
curriedMultiply(2)(3, 4);        // 24
```

**Production Use Cases:**

**1. Configuration Functions:**
```javascript
// API request builder
const makeRequest = method => url => data => headers => {
  return fetch(url, {
    method,
    body: JSON.stringify(data),
    headers: {
      'Content-Type': 'application/json',
      ...headers
    }
  });
};

// Create specialized request functions
const get = makeRequest('GET');
const post = makeRequest('POST');
const put = makeRequest('PUT');

// Create API endpoint functions
const getUsersAPI = get('/api/users');
const createUserAPI = post('/api/users');

// Use with specific data
getUsersAPI(null)({ 'Authorization': 'Bearer token' });
createUserAPI({ name: 'John' })({ 'Authorization': 'Bearer token' });
```

**2. Event Handlers:**
```javascript
// Generic event handler creator
const handleEvent = eventType => element => callback => {
  element.addEventListener(eventType, callback);
};

const onClick = handleEvent('click');
const onSubmit = handleEvent('submit');

// Create specific handlers
const handleButtonClick = onClick(document.getElementById('btn'));
const handleFormSubmit = onSubmit(document.getElementById('form'));

// Apply callbacks
handleButtonClick(e => console.log('Button clicked'));
handleFormSubmit(e => e.preventDefault());
```

**3. Validation Pipeline:**
```javascript
// Validator factory
const validate = rules => data => {
  const errors = [];
  rules.forEach(rule => {
    const error = rule(data);
    if (error) errors.push(error);
  });
  return errors.length > 0 ? errors : null;
};

// Validation rules
const required = field => data => 
  !data[field] ? `${field} is required` : null;

const minLength = field => length => data =>
  data[field] && data[field].length < length 
    ? `${field} must be at least ${length} characters` 
    : null;

const email = field => data =>
  data[field] && !/@/.test(data[field])
    ? `${field} must be a valid email`
    : null;

// Create validator
const validateUser = validate([
  required('username'),
  minLength('username')(3),
  required('email'),
  email('email')
]);

const errors = validateUser({ username: 'ab', email: 'invalid' });
// ['username must be at least 3 characters', 'email must be a valid email']
```

**4. Partial Application (Related Pattern):**
```javascript
// Logging with different levels
const log = level => message => timestamp => {
  console.log(`[${timestamp}] ${level}: ${message}`);
};

const info = log('INFO');
const error = log('ERROR');
const warn = log('WARN');

// Use throughout application
info('User logged in')(Date.now());
error('Database connection failed')(Date.now());

// With automatic timestamp
const logNow = level => message => log(level)(message)(Date.now());
const infoNow = logNow('INFO');
infoNow('Application started');
```

**5. Functional Composition:**
```javascript
// Composing transformations
const compose = (...fns) => x => fns.reduceRight((v, f) => f(v), x);

const addTax = rate => price => price * (1 + rate);
const addShipping = cost => price => price + cost;
const round = decimals => value => 
  Math.round(value * Math.pow(10, decimals)) / Math.pow(10, decimals);

// Create price calculators for different regions
const calculateUSPrice = compose(
  round(2),
  addShipping(10),
  addTax(0.08)
);

const calculateEUPrice = compose(
  round(2),
  addShipping(15),
  addTax(0.20)
);

calculateUSPrice(100);  // 118.00
calculateEUPrice(100);  // 135.00
```

**6. React/Redux Pattern:**
```javascript
// Redux connect-style HOC
const connect = mapStateToProps => mapDispatchToProps => Component => {
  return function ConnectedComponent(props) {
    const stateProps = mapStateToProps(store.getState());
    const dispatchProps = mapDispatchToProps(store.dispatch);
    return Component({ ...props, ...stateProps, ...dispatchProps });
  };
};

// Usage
const mapState = state => ({ user: state.user });
const mapDispatch = dispatch => ({ login: () => dispatch(loginAction()) });
const ConnectedUserProfile = connect(mapState)(mapDispatch)(UserProfile);
```

**Benefits:**
- **Reusability:** Create specialized functions from generic ones
- **Composability:** Combine functions easily
- **Partial Application:** Fix some arguments, vary others
- **Delayed Execution:** Build up function gradually
- **Testability:** Test each curry stage independently

**When to Use vs When Not to Use:**

**✅ Use Currying When:**
```javascript
// Building configuration functions
const createAPIClient = baseURL => timeout => headers => { /* ... */ };

// Creating specialized variations
const buildQueryString = params => url => `${url}?${new URLSearchParams(params)}`;

// Function composition pipelines
const processData = compose(transform, validate, parse);
```

**❌ Avoid Currying When:**
```javascript
// Simple, straightforward operations (overengineering)
// ❌ Unnecessary
const add = x => y => x + y;
// ✅ Better
const add = (x, y) => x + y;

// Frequently changing all parameters together
// ❌ Awkward
calculateArea(width)(height);
// ✅ Better
calculateArea(width, height);

// Callback functions (confusing)
// ❌ Confusing
array.map(transform(config)(options));
// ✅ Better
array.map(item => transform(item, config, options));
```

**Performance Considerations:**
```javascript
// Currying creates additional function calls (slight overhead)
// For performance-critical code, measure before optimizing

// ❌ In tight loops (potential overhead)
for (let i = 0; i < 1000000; i++) {
  curriedFunction(a)(b)(c);
}

// ✅ Pre-bind if possible
const boundFunction = curriedFunction(a)(b);
for (let i = 0; i < 1000000; i++) {
  boundFunction(c);
}
```

17. What is an IIFE (Immediately Invoked Function Expression)?

**Answer:**
An IIFE (Immediately Invoked Function Expression) is a function that is executed immediately after it's created. It's a design pattern that creates a new scope to avoid polluting the global namespace.

**Syntax:**
```javascript
// Basic IIFE
(function() {
  console.log('Executed immediately!');
})();

// Arrow function IIFE
(() => {
  console.log('Arrow IIFE!');
})();

// With parameters
(function(name) {
  console.log(`Hello, ${name}!`);
})('John');

// Alternative syntax (less common)
(function() {
  console.log('Alternative syntax');
}());
```

**Why the Parentheses?**
```javascript
// ❌ Syntax Error - function declaration can't be immediately invoked
function() {
  console.log('Error');
}();

// ✅ Parentheses make it a function expression
(function() {
  console.log('Works!');
})();

// Alternative ways to force expression context
!function() { console.log('Works'); }();
+function() { console.log('Works'); }();
-function() { console.log('Works'); }();
~function() { console.log('Works'); }();
void function() { console.log('Works'); }();
```

**Classic Use Cases (Pre-ES6):**

**1. Module Pattern - Private Variables:**
```javascript
// Before ES6 modules, this was the standard pattern
const calculator = (function() {
  // Private variables
  let result = 0;
  
  // Private function
  function log(message) {
    console.log(`Calculator: ${message}`);
  }
  
  // Public API
  return {
    add(n) {
      result += n;
      log(`Added ${n}`);
      return this;
    },
    subtract(n) {
      result -= n;
      log(`Subtracted ${n}`);
      return this;
    },
    getResult() {
      return result;
    }
  };
})();

calculator.add(5).subtract(2).getResult(); // 3
// calculator.result // undefined - private!
// calculator.log('test') // undefined - private!
```

**2. Avoiding Global Pollution:**
```javascript
// Without IIFE - pollutes global scope
var counter = 0;
var increment = function() { counter++; };
var getCount = function() { return counter; };

// With IIFE - clean global scope
(function() {
  var counter = 0;
  window.myApp = {
    increment: function() { counter++; },
    getCount: function() { return counter; }
  };
})();

// Only myApp is in global scope
```

**3. Loop Variable Problem (Classic Interview Question):**
```javascript
// ❌ Problem with var (pre-ES6)
for (var i = 0; i < 5; i++) {
  setTimeout(function() {
    console.log(i); // 5, 5, 5, 5, 5 (all same reference)
  }, i * 100);
}

// ✅ Solution 1: IIFE creates new scope for each iteration
for (var i = 0; i < 5; i++) {
  (function(j) {
    setTimeout(function() {
      console.log(j); // 0, 1, 2, 3, 4 (captured value)
    }, j * 100);
  })(i);
}

// ✅ Solution 2: Modern approach with let (ES6+)
for (let i = 0; i < 5; i++) {
  setTimeout(function() {
    console.log(i); // 0, 1, 2, 3, 4 (block scoped)
  }, i * 100);
}
```

**Modern Use Cases (Still Relevant):**

**1. Isolated Initialization Code:**
```javascript
// Run setup code without polluting scope
(async function() {
  const config = await fetch('/api/config').then(r => r.json());
  const app = initializeApp(config);
  app.start();
})();

// config and app don't leak to global scope
```

**2. Library/Plugin Initialization:**
```javascript
// jQuery plugin pattern
(function($) {
  $.fn.myPlugin = function(options) {
    // Plugin code with access to private variables
    const settings = $.extend({
      color: 'blue',
      size: 'medium'
    }, options);
    
    return this.each(function() {
      // Plugin logic
    });
  };
})(jQuery);
```

**3. Configuration Objects:**
```javascript
const config = (function() {
  const environment = process.env.NODE_ENV || 'development';
  
  const configs = {
    development: {
      apiUrl: 'http://localhost:3000',
      debug: true
    },
    production: {
      apiUrl: 'https://api.example.com',
      debug: false
    }
  };
  
  return configs[environment];
})();

// config contains only the relevant environment settings
```

**4. Async IIFE (Top-level await alternative):**
```javascript
// Before top-level await, used for async initialization
(async function() {
  try {
    const user = await fetchUser();
    const posts = await fetchPosts(user.id);
    renderUI(user, posts);
  } catch (error) {
    console.error('Initialization failed:', error);
  }
})();

// Modern alternative (ES2022+)
// await fetchUser(); // Top-level await
```

**5. Polyfill Pattern:**
```javascript
// Adding missing browser features safely
(function() {
  if (!Array.prototype.includes) {
    Array.prototype.includes = function(element) {
      return this.indexOf(element) !== -1;
    };
  }
})();
```

**With Return Values:**
```javascript
// IIFE can return values
const result = (function(a, b) {
  return a + b;
})(5, 10);

console.log(result); // 15

// Factory pattern
const createCounter = (function() {
  let id = 0;
  return function() {
    return {
      id: ++id,
      count: 0,
      increment() { this.count++; }
    };
  };
})();

const counter1 = createCounter(); // { id: 1, count: 0, ... }
const counter2 = createCounter(); // { id: 2, count: 0, ... }
```

**Modern Alternatives:**

**ES6 Modules (Preferred for Encapsulation):**
```javascript
// module.js
let privateVar = 0;

export function increment() {
  privateVar++;
}

export function getValue() {
  return privateVar;
}

// main.js
import { increment, getValue } from './module.js';
// privateVar is not accessible
```

**Block Scope with let/const:**
```javascript
{
  const temp = expensiveOperation();
  doSomething(temp);
}
// temp is not accessible here
```

**When to Still Use IIFE:**
- Quick scripts without module system
- Browser console experimentation
- Legacy codebases
- Avoiding global scope in inline scripts
- Creating immediate scopes in non-module contexts

18. What is the arguments object?

**Answer:**
The `arguments` object is an array-like object available inside all non-arrow functions that contains the values of the arguments passed to that function. It allows functions to accept any number of arguments.

**Basic Usage:**
```javascript
function sum() {
  console.log(arguments); // [Arguments] { '0': 1, '1': 2, '2': 3 }
  console.log(arguments.length); // 3
  console.log(arguments[0]); // 1
  console.log(arguments[1]); // 2
  
  let total = 0;
  for (let i = 0; i < arguments.length; i++) {
    total += arguments[i];
  }
  return total;
}

sum(1, 2, 3); // 6
sum(1, 2, 3, 4, 5); // 15
```

**Array-Like, Not Array:**
```javascript
function test() {
  console.log(typeof arguments); // "object"
  console.log(Array.isArray(arguments)); // false
  
  // Has length property
  console.log(arguments.length);
  
  // Can access by index
  console.log(arguments[0]);
  
  // ❌ No array methods
  // arguments.forEach(x => console.log(x)); // TypeError
  // arguments.map(x => x * 2); // TypeError
  
  // ✅ Convert to array first
  const args = Array.from(arguments);
  args.forEach(x => console.log(x));
  
  // Or use spread
  const args2 = [...arguments];
  
  // Or Array.prototype.slice
  const args3 = Array.prototype.slice.call(arguments);
}

test(1, 2, 3);
```

**Properties:**

**1. Length Property:**
```javascript
function checkArgs() {
  console.log(`Received ${arguments.length} arguments`);
  console.log(`Function expects ${checkArgs.length} parameters`);
}

checkArgs(1, 2, 3); // Received 3 arguments, Function expects 0 parameters
```

**2. Callee Property (Deprecated in Strict Mode):**
```javascript
// Reference to the currently executing function
function factorial(n) {
  if (n <= 1) return 1;
  return n * arguments.callee(n - 1); // Recursive call
}

// ⚠️ Deprecated - use named function instead
function factorial(n) {
  if (n <= 1) return 1;
  return n * factorial(n - 1);
}
```

**Accessing Extra Arguments:**
```javascript
function greet(firstName, lastName) {
  console.log(`Parameters: ${firstName} ${lastName}`);
  
  // Access additional arguments
  if (arguments.length > 2) {
    console.log('Extra arguments:');
    for (let i = 2; i < arguments.length; i++) {
      console.log(arguments[i]);
    }
  }
}

greet('John', 'Doe', 'Engineer', 30);
// Parameters: John Doe
// Extra arguments:
// Engineer
// 30
```

**Modifying Arguments:**
```javascript
function modify(x) {
  console.log('Initial:', x, arguments[0]); // 10, 10
  
  x = 20;
  console.log('After x change:', x, arguments[0]); // 20, 20 (linked in non-strict)
  
  arguments[0] = 30;
  console.log('After arguments change:', x, arguments[0]); // 30, 30
}

modify(10);

// In strict mode, they're not linked
'use strict';
function modifyStrict(x) {
  x = 20;
  console.log(x, arguments[0]); // 20, 10 (independent)
}
```

**Classic Use Cases:**

**1. Variable Argument Functions:**
```javascript
function max() {
  if (arguments.length === 0) return -Infinity;
  
  let maxValue = arguments[0];
  for (let i = 1; i < arguments.length; i++) {
    if (arguments[i] > maxValue) {
      maxValue = arguments[i];
    }
  }
  return maxValue;
}

max(1, 5, 3, 9, 2); // 9

// Modern alternative
const max = (...nums) => nums.length ? Math.max(...nums) : -Infinity;
```

**2. Function Overloading (Simulation):**
```javascript
function createPerson() {
  if (arguments.length === 1) {
    // Single object parameter
    return { name: arguments[0].name, age: arguments[0].age };
  } else if (arguments.length === 2) {
    // Two separate parameters
    return { name: arguments[0], age: arguments[1] };
  }
}

createPerson({ name: 'John', age: 30 });
createPerson('John', 30);
```

**3. Wrapper Functions:**
```javascript
// Logging wrapper
function logAndCall(fn) {
  return function() {
    console.log('Arguments:', arguments);
    return fn.apply(this, arguments);
  };
}

function add(a, b) {
  return a + b;
}

const loggedAdd = logAndCall(add);
loggedAdd(2, 3); // Logs arguments, returns 5
```

**Arrow Functions Don't Have arguments:**
```javascript
// ❌ Arrow functions don't have arguments object
const arrowSum = () => {
  console.log(arguments); // ReferenceError (or outer scope arguments)
};

// ✅ Use rest parameters instead
const arrowSum = (...args) => {
  return args.reduce((sum, n) => sum + n, 0);
};

// Nested arrow function accesses outer arguments
function outer() {
  const inner = () => {
    console.log(arguments); // Refers to outer's arguments
  };
  inner();
}

outer(1, 2, 3); // Logs outer's arguments
```

**Modern Alternative: Rest Parameters (Preferred):**
```javascript
// ❌ Old way with arguments
function sum() {
  return Array.from(arguments).reduce((a, b) => a + b, 0);
}

// ✅ Modern way with rest parameters
function sum(...numbers) {
  return numbers.reduce((a, b) => a + b, 0);
}

// Benefits of rest parameters:
// 1. Real array (has all array methods)
// 2. Works with arrow functions
// 3. More explicit and readable
// 4. Can mix with named parameters

function multiply(multiplier, ...numbers) {
  return numbers.map(n => n * multiplier);
}

multiply(2, 1, 2, 3); // [2, 4, 6]
```

**Performance Considerations:**
```javascript
// ⚠️ arguments object prevents optimizations in V8
function slow() {
  return arguments[0] + arguments[1]; // Harder to optimize
}

// ✅ Named parameters allow better optimization
function fast(a, b) {
  return a + b;
}

// ✅ Rest parameters also well-optimized
function fastest(...args) {
  return args[0] + args[1];
}
```

**Best Practices:**
- Prefer rest parameters (`...args`) over `arguments` in modern code
- Use `arguments` only when maintaining legacy code
- Convert to array immediately if you need array methods
- Be aware `arguments` doesn't exist in arrow functions
- Avoid `arguments.callee` (deprecated, forbidden in strict mode)

19. What are default parameters in JavaScript?

**Answer:**
Default parameters allow you to specify default values for function parameters if no argument is passed or if `undefined` is passed. Introduced in ES6, they provide a clean way to handle optional parameters.

**Basic Syntax:**
```javascript
// ES6 default parameters
function greet(name = 'Guest') {
  return `Hello, ${name}!`;
}

greet('John');    // "Hello, John!"
greet();          // "Hello, Guest!"
greet(undefined); // "Hello, Guest!" (undefined triggers default)
greet(null);      // "Hello, null!" (null doesn't trigger default)
```

**Multiple Default Parameters:**
```javascript
function createUser(name = 'Anonymous', age = 0, role = 'user') {
  return { name, age, role };
}

createUser();                           // { name: 'Anonymous', age: 0, role: 'user' }
createUser('John');                     // { name: 'John', age: 0, role: 'user' }
createUser('John', 25);                 // { name: 'John', age: 25, role: 'user' }
createUser('John', 25, 'admin');        // { name: 'John', age: 25, role: 'admin' }
createUser('John', undefined, 'admin'); // { name: 'John', age: 0, role: 'admin' }
```

**Default Values Can Be Expressions:**
```javascript
// Function calls as defaults
function getDefaultName() {
  console.log('Getting default name...');
  return 'Guest';
}

function greet(name = getDefaultName()) {
  return `Hello, ${name}!`;
}

greet('John'); // "Hello, John!" (getDefaultName not called)
greet();       // "Hello, Guest!" (getDefaultName called)

// Using other parameters
function createRectangle(width, height = width) {
  return { width, height };
}

createRectangle(10);      // { width: 10, height: 10 }
createRectangle(10, 20);  // { width: 10, height: 20 }

// Calculations
function discount(price, rate = 0.1) {
  return price * (1 - rate);
}

discount(100);      // 90 (10% off)
discount(100, 0.2); // 80 (20% off)
```

**Referencing Earlier Parameters:**
```javascript
// Later defaults can reference earlier parameters
function buildUrl(protocol = 'https', domain, path = `/${domain}`) {
  return `${protocol}://${domain}${path}`;
}

buildUrl(undefined, 'example.com');           // "https://example.com/example.com"
buildUrl('http', 'example.com', '/about');    // "http://example.com/about"

// ❌ Cannot reference later parameters (TDZ)
function invalid(a = b, b = 1) { // ReferenceError
  return a + b;
}
```

**undefined vs null:**
```javascript
function test(a = 5) {
  console.log(a);
}

test();          // 5 (no argument = undefined)
test(undefined); // 5 (undefined triggers default)
test(null);      // null (null does NOT trigger default)
test(0);         // 0 (0 is a valid value)
test('');        // '' (empty string is valid)
test(false);     // false (false is valid)
```

**Pre-ES6 Pattern (Legacy):**
```javascript
// Old way - manual checking
function greet(name) {
  name = name || 'Guest'; // ⚠️ Bug: 0, '', false also trigger default!
  return `Hello, ${name}!`;
}

// Better old way
function greet(name) {
  name = name !== undefined ? name : 'Guest';
  return `Hello, ${name}!`;
}

// Or
function greet(name) {
  if (name === undefined) {
    name = 'Guest';
  }
  return `Hello, ${name}!`;
}

// Modern way (ES6+)
function greet(name = 'Guest') {
  return `Hello, ${name}!`;
}
```

**Production Use Cases:**

**1. Configuration Objects:**
```javascript
function fetchData(url, options = {}) {
  const {
    method = 'GET',
    headers = {},
    timeout = 5000,
    retries = 3
  } = options;
  
  return fetch(url, {
    method,
    headers: {
      'Content-Type': 'application/json',
      ...headers
    },
    timeout,
    retries
  });
}

// Usage
fetchData('/api/users');
fetchData('/api/users', { method: 'POST' });
fetchData('/api/users', { timeout: 10000, retries: 5 });
```

**2. Pagination:**
```javascript
function getUsers(page = 1, limit = 10, sortBy = 'created_at') {
  const offset = (page - 1) * limit;
  return database.query(
    `SELECT * FROM users ORDER BY ${sortBy} LIMIT ${limit} OFFSET ${offset}`
  );
}

getUsers();         // page 1, 10 items, sorted by created_at
getUsers(2);        // page 2, 10 items, sorted by created_at
getUsers(1, 20);    // page 1, 20 items, sorted by created_at
```

**3. Logging Utility:**
```javascript
function log(message, level = 'info', timestamp = new Date()) {
  const formatted = `[${timestamp.toISOString()}] ${level.toUpperCase()}: ${message}`;
  console.log(formatted);
}

log('Application started');                    // Uses defaults
log('Error occurred', 'error');                // Custom level
log('Custom time', 'warn', new Date('2024')); // All custom
```

**4. Array/String Utilities:**
```javascript
function chunk(array, size = 1) {
  const chunks = [];
  for (let i = 0; i < array.length; i += size) {
    chunks.push(array.slice(i, i + size));
  }
  return chunks;
}

chunk([1, 2, 3, 4, 5]);      // [[1], [2], [3], [4], [5]]
chunk([1, 2, 3, 4, 5], 2);   // [[1, 2], [3, 4], [5]]

function truncate(str, maxLength = 50, suffix = '...') {
  return str.length > maxLength 
    ? str.slice(0, maxLength - suffix.length) + suffix 
    : str;
}

truncate('Long text here');                    // Truncates to 50 chars
truncate('Long text here', 10);                // Truncates to 10 chars
truncate('Long text here', 10, '...');         // Custom suffix
```

**5. API Client:**
```javascript
class APIClient {
  constructor(baseURL, timeout = 5000, retries = 3) {
    this.baseURL = baseURL;
    this.timeout = timeout;
    this.retries = retries;
  }
  
  async request(endpoint, method = 'GET', body = null, headers = {}) {
    const url = `${this.baseURL}${endpoint}`;
    const options = {
      method,
      headers: {
        'Content-Type': 'application/json',
        ...headers
      }
    };
    
    if (body) {
      options.body = JSON.stringify(body);
    }
    
    return fetch(url, options);
  }
}

const api = new APIClient('https://api.example.com');
api.request('/users');                    // GET with defaults
api.request('/users', 'POST', { name: 'John' });
```

**With Destructuring:**
```javascript
// Combining default parameters with destructuring
function createProduct({
  name,
  price = 0,
  category = 'General',
  inStock = true,
  tags = []
} = {}) {
  return { name, price, category, inStock, tags };
}

createProduct({ name: 'Laptop', price: 999 });
// { name: 'Laptop', price: 999, category: 'General', inStock: true, tags: [] }

createProduct();
// { name: undefined, price: 0, category: 'General', inStock: true, tags: [] }

// Better - required parameter
function createProduct({
  name,  // Required (no default)
  price = 0,
  category = 'General'
} = {}) {
  if (!name) throw new Error('name is required');
  return { name, price, category };
}
```

**Arrow Functions:**
```javascript
// Default parameters work with arrow functions
const greet = (name = 'Guest') => `Hello, ${name}!`;

const multiply = (a, b = 1) => a * b;

const createArray = (length = 0, fill = 0) => new Array(length).fill(fill);

createArray();         // []
createArray(3);        // [0, 0, 0]
createArray(3, 5);     // [5, 5, 5]
```

**Best Practices:**
- Use default parameters instead of manual undefined checks
- Remember: only `undefined` triggers defaults, not `null`, `0`, `''`, or `false`
- Place required parameters before optional ones
- Don't overuse - sometimes an options object is clearer
- Default values are evaluated at call time (fresh each call)
- Can reference earlier parameters but not later ones

20. What is the rest parameter?

**Answer:**
The rest parameter syntax (`...`) allows a function to accept an indefinite number of arguments as an array. It collects all remaining arguments into a real array, unlike the `arguments` object.

**Basic Syntax:**
```javascript
function sum(...numbers) {
  console.log(numbers); // Real array: [1, 2, 3, 4, 5]
  console.log(Array.isArray(numbers)); // true
  return numbers.reduce((total, n) => total + n, 0);
}

sum(1, 2, 3, 4, 5); // 15
```

**Key Differences from arguments Object:**

| Feature | arguments | Rest Parameter |
|---------|-----------|----------------|
| **Type** | Array-like object | Real array |
| **Array methods** | No (must convert) | Yes (all methods) |
| **Arrow functions** | No | Yes |
| **Named** | Always `arguments` | Any name you choose |
| **Partial collection** | No | Yes (after named params) |

**Combining with Named Parameters:**
```javascript
// Rest parameter must be last
function multiply(multiplier, ...numbers) {
  return numbers.map(n => n * multiplier);
}

multiply(2, 1, 2, 3, 4); // [2, 4, 6, 8]

// Multiple named + rest
function formatMessage(template, level, ...values) {
  console.log(`[${level}] ${template}`, ...values);
}

formatMessage('User %s logged in at %s', 'INFO', 'John', '10:30 AM');
// [INFO] User %s logged in at %s John 10:30 AM
```

**Rules and Limitations:**
```javascript
// ✅ Rest parameter must be last
function valid(a, b, ...rest) { }

// ❌ Cannot have parameters after rest
// function invalid(a, ...rest, b) { } // SyntaxError

// ❌ Only one rest parameter allowed
// function invalid(...rest1, ...rest2) { } // SyntaxError

// ✅ Can have no other parameters
function allArgs(...args) { }
```

**With Arrow Functions:**
```javascript
// Works perfectly with arrow functions (unlike arguments)
const sum = (...numbers) => numbers.reduce((a, b) => a + b, 0);

const max = (...numbers) => Math.max(...numbers);

const concat = (...arrays) => [].concat(...arrays);

// Example usage
sum(1, 2, 3);              // 6
max(5, 2, 9, 1);          // 9
concat([1, 2], [3, 4]);   // [1, 2, 3, 4]
```

**Production Use Cases:**

**1. Flexible API Functions:**
```javascript
// Logger accepting any number of messages
function log(level, ...messages) {
  const timestamp = new Date().toISOString();
  console.log(`[${timestamp}] ${level}:`, ...messages);
}

log('INFO', 'Application started');
log('ERROR', 'Connection failed', 'Retrying...', 'Attempt 3');
log('DEBUG', 'User data:', { id: 1, name: 'John' }, 'Role:', 'admin');

// Database query builder
function query(table, ...conditions) {
  const where = conditions
    .map(([field, value]) => `${field} = '${value}'`)
    .join(' AND ');
  return `SELECT * FROM ${table} WHERE ${where}`;
}

query('users', ['age', 18], ['status', 'active']);
// "SELECT * FROM users WHERE age = '18' AND status = 'active'"
```

**2. Math Operations:**
```javascript
function average(...numbers) {
  if (numbers.length === 0) return 0;
  return numbers.reduce((sum, n) => sum + n, 0) / numbers.length;
}

average(10, 20, 30, 40); // 25

function range(start, end, ...values) {
  return values.filter(v => v >= start && v <= end);
}

range(5, 15, 1, 7, 10, 20, 12, 3); // [7, 10, 12]
```

**3. Event Emitter Pattern:**
```javascript
class EventEmitter {
  constructor() {
    this.events = {};
  }
  
  on(event, callback) {
    if (!this.events[event]) {
      this.events[event] = [];
    }
    this.events[event].push(callback);
  }
  
  emit(event, ...args) {
    if (this.events[event]) {
      this.events[event].forEach(callback => callback(...args));
    }
  }
}

const emitter = new EventEmitter();

emitter.on('user:login', (username, timestamp, ip) => {
  console.log(`${username} logged in at ${timestamp} from ${ip}`);
});

emitter.emit('user:login', 'john_doe', Date.now(), '192.168.1.1');
```

**4. Function Composition:**
```javascript
// Compose multiple functions
function compose(...functions) {
  return function(input) {
    return functions.reduceRight((value, fn) => fn(value), input);
  };
}

const addTen = x => x + 10;
const multiplyByTwo = x => x * 2;
const square = x => x * x;

const calculate = compose(square, multiplyByTwo, addTen);
calculate(5); // ((5 + 10) * 2) ^ 2 = 900

// Pipe (left to right)
function pipe(...functions) {
  return function(input) {
    return functions.reduce((value, fn) => fn(value), input);
  };
}

const process = pipe(addTen, multiplyByTwo, square);
process(5); // ((5 + 10) * 2) ^ 2 = 900
```

**5. Middleware Pattern:**
```javascript
class RequestHandler {
  constructor() {
    this.middlewares = [];
  }
  
  use(...fns) {
    this.middlewares.push(...fns);
    return this;
  }
  
  async handle(req, res) {
    for (const middleware of this.middlewares) {
      await middleware(req, res);
    }
  }
}

const handler = new RequestHandler();
handler
  .use(logger, authenticate, authorize)
  .use(validateInput, sanitize);
```

**6. Array Operations:**
```javascript
// Merge multiple arrays
function merge(...arrays) {
  return [].concat(...arrays);
}

merge([1, 2], [3, 4], [5, 6]); // [1, 2, 3, 4, 5, 6]

// Find common elements
function intersection(...arrays) {
  if (arrays.length === 0) return [];
  return arrays.reduce((acc, arr) => 
    acc.filter(item => arr.includes(item))
  );
}

intersection([1, 2, 3], [2, 3, 4], [3, 4, 5]); // [3]

// Unique values from multiple arrays
function uniqueValues(...arrays) {
  return [...new Set([].concat(...arrays))];
}

uniqueValues([1, 2], [2, 3], [3, 4]); // [1, 2, 3, 4]
```

**7. Partial Application / Currying:**
```javascript
function partial(fn, ...args) {
  return function(...remainingArgs) {
    return fn(...args, ...remainingArgs);
  };
}

function greet(greeting, name, punctuation) {
  return `${greeting}, ${name}${punctuation}`;
}

const sayHello = partial(greet, 'Hello');
sayHello('John', '!'); // "Hello, John!"

const sayHelloJohn = partial(greet, 'Hello', 'John');
sayHelloJohn('!'); // "Hello, John!"
```

**With Destructuring:**
```javascript
// Destructure + rest for objects
const user = {
  id: 1,
  name: 'John',
  email: 'john@example.com',
  age: 30,
  role: 'admin'
};

const { id, name, ...rest } = user;
console.log(rest); // { email: '...', age: 30, role: 'admin' }

// In function parameters
function updateUser({ id, ...updates }) {
  return database.update(id, updates);
}

updateUser({ id: 1, name: 'Jane', age: 31 });
// Updates only name and age, not id

// Array destructuring with rest
const [first, second, ...remaining] = [1, 2, 3, 4, 5];
console.log(first);     // 1
console.log(second);    // 2
console.log(remaining); // [3, 4, 5]
```

**Performance Considerations:**
```javascript
// Rest parameters are efficient (no conversion needed)
function processItems(...items) {
  return items.filter(x => x > 0); // Direct array operations
}

// With arguments object (old way, less efficient)
function processItems() {
  const items = Array.from(arguments); // Conversion overhead
  return items.filter(x => x > 0);
}
```

**Best Practices:**
- Use rest parameters instead of the `arguments` object
- Name rest parameters descriptively (`...numbers`, `...items`, `...args`)
- Always place rest parameter last
- Works seamlessly with arrow functions
- Perfect for variable-length argument lists
- Use with spread operator for powerful array/object manipulations
- Ideal for wrapper/decorator functions that forward arguments
