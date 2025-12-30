## Scope & Closures

21. What is scope in JavaScript?

**Answer:**
Scope determines the accessibility (visibility) of variables, functions, and objects in different parts of your code during runtime. It defines where variables can be accessed and helps prevent naming conflicts.

**Types of Scope in JavaScript:**

1. **Global Scope** - Accessible everywhere in the code
2. **Function Scope** - Accessible only within the function
3. **Block Scope** - Accessible only within the block `{ }` (ES6+)
4. **Module Scope** - Accessible only within the module (ES6 modules)

**Global Scope:**
```javascript
// Variables declared outside any function or block
var globalVar = 'Global';
let globalLet = 'Also Global';
const globalConst = 'Global too';

function test() {
  console.log(globalVar); // Accessible
}

console.log(globalVar); // Accessible
```

**Function Scope:**
```javascript
function outer() {
  var functionScoped = 'Inside function';
  let alsoFunctionScoped = 'Also inside';
  
  console.log(functionScoped); // ✅ Accessible
  
  function inner() {
    console.log(functionScoped); // ✅ Accessible (parent scope)
  }
  
  inner();
}

outer();
// console.log(functionScoped); // ❌ ReferenceError: not accessible outside
```

**Block Scope (ES6+):**
```javascript
{
  let blockScoped = 'Inside block';
  const alsoBlockScoped = 'Block scoped';
  var notBlockScoped = 'Function scoped!';
  
  console.log(blockScoped); // ✅ Accessible
}

// console.log(blockScoped); // ❌ ReferenceError
console.log(notBlockScoped); // ✅ var ignores block scope!

// if statements create block scope
if (true) {
  let x = 10;
  const y = 20;
  var z = 30;
}
// console.log(x); // ❌ ReferenceError
// console.log(y); // ❌ ReferenceError
console.log(z); // ✅ 30 (var leaks out)

// for loops
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100); // 0, 1, 2 (each iteration has own 'i')
}

for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100); // 3, 3, 3 (shared 'i')
}
```

**Scope Hierarchy:**
```javascript
let global = 'Global';

function outer() {
  let outerVar = 'Outer';
  
  function middle() {
    let middleVar = 'Middle';
    
    function inner() {
      let innerVar = 'Inner';
      
      // All are accessible from inner scope
      console.log(innerVar);   // ✅ Own scope
      console.log(middleVar);  // ✅ Parent scope
      console.log(outerVar);   // ✅ Grandparent scope
      console.log(global);     // ✅ Global scope
    }
    
    inner();
    // console.log(innerVar); // ❌ Not accessible
  }
  
  middle();
}

outer();
```

**Production Implications:**

**1. Variable Collision Prevention:**
```javascript
// Without proper scoping
var name = 'Global Name';

function setName() {
  name = 'Changed'; // Modifies global!
}

setName();
console.log(name); // 'Changed' - unintended side effect

// With proper scoping
let name = 'Global Name';

function setName() {
  let name = 'Local'; // New variable, doesn't affect global
  console.log(name); // 'Local'
}

setName();
console.log(name); // 'Global Name' - safe
```

**2. Memory Management:**
```javascript
// Variables stay in memory while in scope
function processLargeData() {
  let largeArray = new Array(1000000).fill('data');
  
  // largeArray is in memory here
  doSomething(largeArray);
  
  // After function ends, largeArray can be garbage collected
}

// vs keeping in global scope
let largeArray = new Array(1000000).fill('data'); // Stays in memory forever!
```

**3. Module Pattern (Pre-ES6):**
```javascript
const counterModule = (function() {
  // Private variables (function scope)
  let count = 0;
  
  // Private function
  function log() {
    console.log(`Count: ${count}`);
  }
  
  // Public API
  return {
    increment() {
      count++;
      log();
    },
    decrement() {
      count--;
      log();
    },
    getCount() {
      return count;
    }
  };
})();

counterModule.increment(); // Count: 1
counterModule.getCount();  // 1
// counterModule.count;    // undefined - private!
```

**4. Avoiding Global Pollution:**
```javascript
// ❌ Bad - pollutes global scope
var helper1 = function() { /* ... */ };
var helper2 = function() { /* ... */ };
var helper3 = function() { /* ... */ };

// ✅ Good - namespace pattern
const MyApp = {
  helpers: {
    helper1: function() { /* ... */ },
    helper2: function() { /* ... */ },
    helper3: function() { /* ... */ }
  }
};

// ✅ Better - ES6 modules
// helpers.js
export function helper1() { /* ... */ }
export function helper2() { /* ... */ }
```

**Scope and `this`:**
```javascript
const obj = {
  name: 'Object',
  regularMethod: function() {
    console.log(this.name); // 'Object' - this refers to obj
    
    function inner() {
      console.log(this.name); // undefined - this is global/undefined
    }
    inner();
  },
  arrowMethod: function() {
    console.log(this.name); // 'Object'
    
    const inner = () => {
      console.log(this.name); // 'Object' - arrow function inherits this
    };
    inner();
  }
};
```

**Best Practices:**
- Use `let` and `const` (block-scoped) instead of `var`
- Keep variables in the smallest scope possible
- Avoid global variables whenever possible
- Use modules to encapsulate code
- Declare variables at the top of their scope for clarity
- Use IIFE or modules for private variables/functions

22. What is the difference between global scope and local scope?

**Answer:**
Global scope and local scope define the accessibility boundaries of variables in JavaScript. Understanding the difference is crucial for writing maintainable and bug-free code.

**Global Scope:**
Variables declared outside any function or block are in the global scope. They are accessible from anywhere in the code.

```javascript
// Global scope
var globalVar = 'I am global';
let globalLet = 'Also global';
const globalConst = 'Global constant';

function test() {
  console.log(globalVar); // ✅ Accessible
}

if (true) {
  console.log(globalLet); // ✅ Accessible
}

console.log(globalConst); // ✅ Accessible
```

**Local Scope:**
Variables declared inside a function or block are in local scope. They are only accessible within that function or block.

```javascript
function myFunction() {
  // Local scope (function)
  var localVar = 'I am local';
  let localLet = 'Also local';
  const localConst = 'Local constant';
  
  console.log(localVar); // ✅ Accessible inside function
}

myFunction();
// console.log(localVar); // ❌ ReferenceError: not defined

// Block scope
{
  let blockVar = 'Block scoped';
  console.log(blockVar); // ✅ Accessible inside block
}
// console.log(blockVar); // ❌ ReferenceError
```

**Detailed Comparison:**

| Aspect | Global Scope | Local Scope |
|--------|-------------|-------------|
| **Accessibility** | Everywhere in code | Only within function/block |
| **Lifetime** | Entire application runtime | Until function/block exits |
| **Memory** | Persists forever | Released after scope ends |
| **Naming Conflicts** | High risk | Low risk (isolated) |
| **Browser Global** | Attached to `window` (var) | Not attached to window |
| **Testing** | Harder (side effects) | Easier (isolated) |

**Global Variables in Browser:**
```javascript
// In browser
var globalVar = 'test';
let globalLet = 'test2';

console.log(window.globalVar);  // 'test' (var creates window property)
console.log(window.globalLet);  // undefined (let doesn't)

// Global object properties
window.myGlobal = 'value';
console.log(myGlobal); // 'value' - accessible globally
```

**Shadowing (Local Overrides Global):**
```javascript
let name = 'Global';

function test() {
  let name = 'Local'; // Shadows global variable
  console.log(name);  // 'Local'
}

test();
console.log(name); // 'Global' - unchanged
```

**Accidental Globals:**
```javascript
function createGlobal() {
  // ❌ Forgot 'let', 'const', or 'var' - creates global!
  accidentalGlobal = 'Oops';
}

createGlobal();
console.log(accidentalGlobal); // 'Oops' - now global!

// In strict mode, this throws an error
'use strict';
function strictFunction() {
  undeclared = 'Error'; // ReferenceError: undeclared is not defined
}
```

**Production Problems with Global Scope:**

**1. Naming Conflicts:**
```javascript
// library1.js
var userId = 123;

// library2.js
var userId = 456; // Overwrites library1's userId!

// yourCode.js
console.log(userId); // 456 - unexpected!
```

**2. Memory Leaks:**
```javascript
// ❌ Global variables never garbage collected
var cache = [];

function addToCache(data) {
  cache.push(data); // Grows indefinitely
}

// ✅ Use local scope or proper cleanup
function processData() {
  let localCache = [];
  // ... use cache ...
  // localCache is garbage collected after function ends
}
```

**3. Tight Coupling:**
```javascript
// ❌ Functions depend on global state
var config = { theme: 'dark' };

function renderUI() {
  if (config.theme === 'dark') { /* ... */ }
}

// ✅ Pass dependencies explicitly
function renderUI(config) {
  if (config.theme === 'dark') { /* ... */ }
}
```

**Best Practices:**

**1. Minimize Global Variables:**
```javascript
// ❌ Multiple globals
var app = {};
var config = {};
var utils = {};

// ✅ Single global namespace
var MyApp = {
  app: {},
  config: {},
  utils: {}
};

// ✅✅ ES6 Modules (no globals needed)
// app.js
export const app = {};
export const config = {};
export const utils = {};
```

**2. Use Modules:**
```javascript
// config.js
const API_URL = 'https://api.example.com';
const TIMEOUT = 5000;

export { API_URL, TIMEOUT };

// main.js
import { API_URL, TIMEOUT } from './config.js';
// Only imported when needed, not global
```

**3. IIFE for Isolation:**
```javascript
(function() {
  // All variables here are local
  var privateVar = 'Not global';
  
  function privateFunction() {
    return privateVar;
  }
  
  // Only expose what's needed
  window.MyLibrary = {
    publicMethod: function() {
      return privateFunction();
    }
  };
})();
```

**4. Use 'use strict':**
```javascript
'use strict';

function test() {
  undeclaredVar = 'value'; // ReferenceError - prevents accidental globals
}
```

**Local Scope Benefits:**

```javascript
// ✅ Encapsulation
function calculatePrice(quantity, price) {
  let tax = 0.1;           // Local, can't interfere with other code
  let total = quantity * price * (1 + tax);
  return total;
}

// ✅ Reusability (no side effects)
function processUser(user) {
  let tempData = transform(user); // Local, function is pure
  return tempData;
}

// ✅ Testing (isolated)
function add(a, b) {
  return a + b; // No global dependencies
}
```

**When Global Scope is Acceptable:**
- Application configuration constants
- Third-party library exports
- Polyfills
- Single-page app root instance (like React root)
- Feature flags (sparingly)

```javascript
// Acceptable globals
const APP_VERSION = '1.0.0';
const IS_PRODUCTION = true;

// Or better, as module exports
export const APP_VERSION = '1.0.0';
export const IS_PRODUCTION = true;
```

23. What is lexical scoping?

**Answer:**
Lexical scoping (also called static scoping) means that the scope of a variable is determined by its position in the source code at the time it's written, not where it's called from. JavaScript uses lexical scoping to resolve variable access.

**Core Concept:**
```javascript
const name = 'Global';

function outer() {
  const name = 'Outer';
  
  function inner() {
    // 'name' is resolved by looking at WHERE inner() was DEFINED,
    // not where it's CALLED from
    console.log(name); // 'Outer' - uses parent scope
  }
  
  return inner;
}

const name = 'Another Global';
const innerFunc = outer();
innerFunc(); // 'Outer' - not 'Another Global' (lexical scope!)
```

**How Lexical Scope Works:**

JavaScript looks for variables by walking up the scope chain based on the code structure:

```javascript
const global = 'Global level';

function level1() {
  const level1Var = 'Level 1';
  
  function level2() {
    const level2Var = 'Level 2';
    
    function level3() {
      const level3Var = 'Level 3';
      
      // Lookup chain: level3 -> level2 -> level1 -> global
      console.log(level3Var); // ✅ Found in current scope
      console.log(level2Var); // ✅ Found in parent scope
      console.log(level1Var); // ✅ Found in grandparent scope
      console.log(global);    // ✅ Found in global scope
    }
    
    level3();
  }
  
  level2();
}

level1();
```

**Lexical vs Dynamic Scoping:**

```javascript
const value = 'Global';

function showValue() {
  console.log(value);
}

function outer() {
  const value = 'Outer';
  showValue();
}

outer(); // 'Global' - lexical scoping (where defined)
// If JavaScript used dynamic scoping, it would print 'Outer' (where called)
```

**Closure and Lexical Scope:**

Closures work because of lexical scoping - inner functions remember their lexical environment:

```javascript
function createCounter() {
  let count = 0; // Lexically scoped to createCounter
  
  return function() {
    count++; // Has access to 'count' due to lexical scope
    return count;
  };
}

const counter = createCounter();
console.log(counter()); // 1
console.log(counter()); // 2
console.log(counter()); // 3

// 'count' is accessed based on WHERE the inner function was DEFINED,
// not where it's CALLED
```

**Nested Function Example:**

```javascript
function outer(x) {
  function middle(y) {
    function inner(z) {
      // All three parameters accessible due to lexical scope
      console.log(x + y + z);
    }
    return inner;
  }
  return middle;
}

const fn = outer(1)(2);
fn(3); // 6 - all three variables still accessible
```

**Production Use Cases:**

**1. Data Privacy / Encapsulation:**
```javascript
function createBankAccount(initialBalance) {
  // 'balance' is private - only accessible via lexical scope
  let balance = initialBalance;
  
  return {
    deposit(amount) {
      balance += amount; // Accesses parent scope
      return balance;
    },
    withdraw(amount) {
      if (amount <= balance) {
        balance -= amount;
        return balance;
      }
      throw new Error('Insufficient funds');
    },
    getBalance() {
      return balance;
    }
  };
}

const account = createBankAccount(1000);
account.deposit(500);  // 1500
account.withdraw(200); // 1300
// account.balance;    // undefined - truly private!
```

**2. Event Handlers with State:**
```javascript
function setupButton(buttonId, initialCount) {
  let clickCount = initialCount; // Lexically scoped
  const button = document.getElementById(buttonId);
  
  button.addEventListener('click', function() {
    // Event handler accesses clickCount via lexical scope
    clickCount++;
    button.textContent = `Clicked ${clickCount} times`;
  });
}

setupButton('btn1', 0); // Each button has its own clickCount
setupButton('btn2', 0);
```

**3. Factory Functions:**
```javascript
function createLogger(prefix) {
  // prefix is captured lexically
  return {
    info: function(message) {
      console.log(`[${prefix}] INFO: ${message}`);
    },
    error: function(message) {
      console.log(`[${prefix}] ERROR: ${message}`);
    }
  };
}

const userLogger = createLogger('USER');
const adminLogger = createLogger('ADMIN');

userLogger.info('Logged in');   // [USER] INFO: Logged in
adminLogger.error('Access denied'); // [ADMIN] ERROR: Access denied
```

**4. Configuration Builders:**
```javascript
function createAPIClient(baseURL) {
  // baseURL captured in lexical scope
  return {
    get(endpoint) {
      return fetch(`${baseURL}${endpoint}`); // Uses lexical baseURL
    },
    post(endpoint, data) {
      return fetch(`${baseURL}${endpoint}`, {
        method: 'POST',
        body: JSON.stringify(data)
      });
    }
  };
}

const api1 = createAPIClient('https://api1.example.com');
const api2 = createAPIClient('https://api2.example.com');

api1.get('/users'); // https://api1.example.com/users
api2.get('/users'); // https://api2.example.com/users
```

**5. Memoization:**
```javascript
function memoize(fn) {
  const cache = {}; // Lexically scoped cache
  
  return function(...args) {
    const key = JSON.stringify(args);
    // Inner function accesses cache via lexical scope
    if (key in cache) {
      return cache[key];
    }
    const result = fn(...args);
    cache[key] = result;
    return result;
  };
}

const expensiveCalc = memoize(function(n) {
  console.log('Calculating...');
  return n * n;
});

expensiveCalc(5); // 'Calculating...' -> 25
expensiveCalc(5); // 25 (cached, no log)
```

**Loop Variable Capture (Classic Problem):**

```javascript
// ❌ Problem with var (not block-scoped)
for (var i = 0; i < 3; i++) {
  setTimeout(function() {
    console.log(i); // 3, 3, 3 (all callbacks share same 'i')
  }, 100);
}

// ✅ Solution 1: IIFE creates new lexical scope
for (var i = 0; i < 3; i++) {
  (function(j) { // j is lexically scoped to this IIFE
    setTimeout(function() {
      console.log(j); // 0, 1, 2
    }, 100);
  })(i);
}

// ✅ Solution 2: let is block-scoped (each iteration = new scope)
for (let i = 0; i < 3; i++) {
  setTimeout(function() {
    console.log(i); // 0, 1, 2
  }, 100);
}
```

**Scope Chain Walking:**

```javascript
let a = 'global a';
let b = 'global b';

function outer() {
  let a = 'outer a';
  let c = 'outer c';
  
  function inner() {
    let a = 'inner a';
    let d = 'inner d';
    
    console.log(a); // 'inner a' - found in current scope
    console.log(b); // 'global b' - walks up to global
    console.log(c); // 'outer c' - walks up to outer
    console.log(d); // 'inner d' - found in current scope
    // Variable lookup stops at first match while walking up
  }
  
  inner();
}

outer();
```

**Lexical Scope is Determined at Parse Time:**

```javascript
function outer() {
  let x = 10;
  
  function inner() {
    console.log(x); // Scope is determined HERE (parse time)
  }
  
  return inner;
}

let x = 20;
const fn = outer();
fn(); // 10 - not 20! Uses lexical scope from definition
```

**Benefits of Lexical Scoping:**
- **Predictable:** Scope is clear from reading code structure
- **Tooling:** IDEs can statically analyze variable usage
- **Closures:** Enables powerful closure patterns
- **Optimization:** Engines can optimize scope lookups
- **Debugging:** Easier to trace variable origins

**Gotchas:**

```javascript
// Modifying parent scope variables
function outer() {
  let count = 0;
  
  function increment() {
    count++; // Modifies outer's count!
  }
  
  increment();
  console.log(count); // 1 - modified
}

// Reference vs Value
function outer() {
  let obj = { value: 0 };
  
  function modify() {
    obj.value = 10; // Modifies the object
    obj = { value: 20 }; // Only changes local reference
  }
  
  modify();
  console.log(obj.value); // 10 - object was modified
}
```

24. What is a closure?

**Answer:**
A closure is a function that has access to variables from its outer (enclosing) lexical scope, even after the outer function has finished executing. Closures are created every time a function is created in JavaScript.

**Simple Definition:**
> A closure is the combination of a function and the lexical environment within which that function was declared.

**Basic Example:**
```javascript
function outer() {
  let count = 0; // Outer scope variable
  
  function inner() {
    count++; // Inner function accesses outer variable
    return count;
  }
  
  return inner;
}

const counter = outer(); // outer() has finished executing
console.log(counter());  // 1 - but 'count' is still accessible!
console.log(counter());  // 2 - 'count' persists between calls
console.log(counter());  // 3 - this is a closure
```

**How Closures Work:**

When a function is created, it maintains a reference to its lexical environment (the scope in which it was created). This reference persists even after the outer function returns.

```javascript
function createGreeter(greeting) {
  // 'greeting' is in the closure
  return function(name) {
    return `${greeting}, ${name}!`;
  };
}

const sayHello = createGreeter('Hello');
const sayHi = createGreeter('Hi');

console.log(sayHello('John')); // "Hello, John!"
console.log(sayHi('Jane'));    // "Hi, Jane!"

// Each closure maintains its own copy of 'greeting'
```

**Multiple Closures Sharing Same Scope:**

```javascript
function createCounter() {
  let count = 0;
  
  return {
    increment: function() {
      count++;
      return count;
    },
    decrement: function() {
      count--;
      return count;
    },
    getCount: function() {
      return count;
    }
  };
}

const counter = createCounter();
counter.increment(); // 1
counter.increment(); // 2
counter.decrement(); // 1
console.log(counter.getCount()); // 1
// All three methods share the same 'count' via closure
```

**Closure Scope Chain:**

```javascript
const global = 'Global';

function outer() {
  const outerVar = 'Outer';
  
  function middle() {
    const middleVar = 'Middle';
    
    function inner() {
      const innerVar = 'Inner';
      
      // inner() has closure over all outer scopes
      console.log(innerVar);   // 'Inner'
      console.log(middleVar);  // 'Middle'
      console.log(outerVar);   // 'Outer'
      console.log(global);     // 'Global'
    }
    
    return inner;
  }
  
  return middle();
}

const fn = outer();
fn(); // Accesses all variables via closure chain
```

**Production Use Cases:**

**1. Data Privacy / Encapsulation:**
```javascript
function createUser(name, email) {
  // Private variables
  let password = '';
  let loginAttempts = 0;
  const MAX_ATTEMPTS = 3;
  
  // Private function
  function hashPassword(pass) {
    return btoa(pass); // Simple hash (use crypto in production)
  }
  
  // Public API (methods use closure to access private data)
  return {
    getName() {
      return name;
    },
    getEmail() {
      return email;
    },
    setPassword(newPassword) {
      password = hashPassword(newPassword);
    },
    login(inputPassword) {
      if (loginAttempts >= MAX_ATTEMPTS) {
        throw new Error('Account locked');
      }
      
      if (hashPassword(inputPassword) === password) {
        loginAttempts = 0;
        return true;
      }
      
      loginAttempts++;
      return false;
    }
  };
}

const user = createUser('John', 'john@example.com');
user.setPassword('secret123');
user.login('secret123'); // true
// user.password; // undefined - truly private!
```

**2. Event Handlers with State:**
```javascript
function setupButtons() {
  const buttons = document.querySelectorAll('.counter-btn');
  
  buttons.forEach((button, index) => {
    let count = 0; // Each button has its own count via closure
    
    button.addEventListener('click', function() {
      count++; // Closure captures 'count' and 'button'
      button.textContent = `Clicked ${count} times`;
    });
  });
}
```

**3. Module Pattern:**
```javascript
const ShoppingCart = (function() {
  // Private state
  let items = [];
  let total = 0;
  
  // Private functions
  function calculateTotal() {
    total = items.reduce((sum, item) => sum + item.price, 0);
  }
  
  // Public API
  return {
    addItem(item) {
      items.push(item);
      calculateTotal();
    },
    removeItem(itemId) {
      items = items.filter(item => item.id !== itemId);
      calculateTotal();
    },
    getItems() {
      return [...items]; // Return copy to prevent external modification
    },
    getTotal() {
      return total;
    },
    clear() {
      items = [];
      total = 0;
    }
  };
})();

ShoppingCart.addItem({ id: 1, name: 'Book', price: 20 });
ShoppingCart.addItem({ id: 2, name: 'Pen', price: 5 });
console.log(ShoppingCart.getTotal()); // 25
// ShoppingCart.items; // undefined - private!
```

**4. Function Factories:**
```javascript
function createMultiplier(multiplier) {
  return function(number) {
    return number * multiplier; // Closure over 'multiplier'
  };
}

const double = createMultiplier(2);
const triple = createMultiplier(3);
const quadruple = createMultiplier(4);

console.log(double(5));     // 10
console.log(triple(5));     // 15
console.log(quadruple(5));  // 20

// API client factory
function createAPIClient(baseURL, apiKey) {
  return {
    async get(endpoint) {
      const response = await fetch(`${baseURL}${endpoint}`, {
        headers: { 'Authorization': `Bearer ${apiKey}` }
      });
      return response.json();
    },
    async post(endpoint, data) {
      const response = await fetch(`${baseURL}${endpoint}`, {
        method: 'POST',
        headers: { 
          'Authorization': `Bearer ${apiKey}`,
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(data)
      });
      return response.json();
    }
  };
}

const apiClient = createAPIClient('https://api.example.com', 'secret-key-123');
apiClient.get('/users');
```

**5. Callback Functions:**
```javascript
function fetchUser(userId) {
  const requestTime = Date.now();
  
  fetch(`/api/users/${userId}`)
    .then(response => response.json())
    .then(user => {
      // Closure captures requestTime and userId
      const duration = Date.now() - requestTime;
      console.log(`Fetched user ${userId} in ${duration}ms`);
      return user;
    });
}
```

**6. Iterators and Generators:**
```javascript
function createIterator(array) {
  let index = 0; // Closure maintains state between calls
  
  return {
    next() {
      if (index < array.length) {
        return { value: array[index++], done: false };
      }
      return { done: true };
    },
    reset() {
      index = 0;
    }
  };
}

const iterator = createIterator([1, 2, 3]);
console.log(iterator.next()); // { value: 1, done: false }
console.log(iterator.next()); // { value: 2, done: false }
iterator.reset();
console.log(iterator.next()); // { value: 1, done: false }
```

**Common Pitfalls:**

**1. Loop Closure Problem:**
```javascript
// ❌ Problem: All callbacks share same 'i'
function createButtons() {
  for (var i = 0; i < 5; i++) {
    document.getElementById(`btn${i}`).onclick = function() {
      alert(i); // Always alerts 5!
    };
  }
}

// ✅ Solution 1: IIFE creates new scope
for (var i = 0; i < 5; i++) {
  (function(j) {
    document.getElementById(`btn${j}`).onclick = function() {
      alert(j); // Alerts correct value
    };
  })(i);
}

// ✅ Solution 2: Use let (block-scoped)
for (let i = 0; i < 5; i++) {
  document.getElementById(`btn${i}`).onclick = function() {
    alert(i); // Alerts correct value
  };
}
```

**2. Memory Leaks:**
```javascript
// ⚠️ Potential memory leak
function createHeavyClosures() {
  const largeData = new Array(1000000).fill('data');
  
  return function() {
    // Even if not using largeData, it's kept in memory
    console.log('Hello');
  };
}

// ✅ Better: Only close over what you need
function createHeavyClosures() {
  const largeData = new Array(1000000).fill('data');
  const summary = largeData.length; // Extract only what's needed
  
  return function() {
    console.log(`Array size: ${summary}`);
    // largeData can be garbage collected
  };
}
```

**3. Accidental Sharing:**
```javascript
function createFunctions() {
  const functions = [];
  
  for (var i = 0; i < 3; i++) {
    functions.push(function() {
      return i; // Shares same 'i'
    });
  }
  
  return functions;
}

const fns = createFunctions();
console.log(fns[0]()); // 3
console.log(fns[1]()); // 3
console.log(fns[2]()); // 3 (all return 3!)
```

**Performance Considerations:**
```javascript
// ⚠️ Creating closures in constructors (memory overhead)
function SlowObject() {
  this.value = 0;
  
  // Each instance creates new function in memory
  this.increment = function() {
    this.value++;
  };
}

// ✅ Better: Use prototype (shared across instances)
function FastObject() {
  this.value = 0;
}

FastObject.prototype.increment = function() {
  this.value++;
};

// When you need closure, balance is key
function OptimizedObject(initialValue) {
  this.value = initialValue;
  
  // Only create closure if needed
  if (initialValue > 100) {
    this.increment = function() {
      this.value++;
    };
  }
}
OptimizedObject.prototype.increment = function() {
  this.value++;
};
```

**Key Takeaways:**
- Closures give functions access to outer scope variables
- They persist even after outer function returns
- Essential for data privacy and encapsulation
- Enable powerful patterns (modules, factories, callbacks)
- Be mindful of memory implications
- Every function creates a closure in JavaScript

25. What are the practical uses of closures?

**Answer:**
Closures are one of the most powerful features in JavaScript. They enable numerous design patterns and solve common programming problems elegantly. Here are the most practical uses in production applications:

**1. Data Privacy and Encapsulation:**

Creating truly private variables that cannot be accessed from outside:

```javascript
function createWallet(initialBalance) {
  // Private variables - only accessible via closure
  let balance = initialBalance;
  const transactions = [];
  
  // Private function
  function recordTransaction(type, amount) {
    transactions.push({
      type,
      amount,
      timestamp: new Date(),
      balance: balance
    });
  }
  
  // Public API
  return {
    deposit(amount) {
      if (amount <= 0) throw new Error('Invalid amount');
      balance += amount;
      recordTransaction('deposit', amount);
      return balance;
    },
    
    withdraw(amount) {
      if (amount <= 0) throw new Error('Invalid amount');
      if (amount > balance) throw new Error('Insufficient funds');
      balance -= amount;
      recordTransaction('withdraw', amount);
      return balance;
    },
    
    getBalance() {
      return balance;
    },
    
    getTransactions() {
      return [...transactions]; // Return copy
    }
  };
}

const myWallet = createWallet(1000);
myWallet.deposit(500);     // 1500
myWallet.withdraw(200);    // 1300
console.log(myWallet.getBalance()); // 1300
// myWallet.balance = 99999; // ❌ Can't access private balance!
```

**2. Function Factories:**

Creating customized functions on demand:

```javascript
// HTTP request builder
function createHTTPClient(baseURL, defaultHeaders = {}) {
  return {
    async request(endpoint, options = {}) {
      const url = `${baseURL}${endpoint}`;
      const config = {
        ...options,
        headers: {
          ...defaultHeaders,
          ...options.headers
        }
      };
      
      const response = await fetch(url, config);
      return response.json();
    },
    
    get(endpoint) {
      return this.request(endpoint, { method: 'GET' });
    },
    
    post(endpoint, data) {
      return this.request(endpoint, {
        method: 'POST',
        body: JSON.stringify(data)
      });
    }
  };
}

// Create clients for different APIs
const githubAPI = createHTTPClient('https://api.github.com', {
  'Authorization': 'token GITHUB_TOKEN'
});

const internalAPI = createHTTPClient('https://internal.company.com/api', {
  'X-API-Key': 'internal-key'
});

githubAPI.get('/users/octocat');
internalAPI.post('/users', { name: 'John' });
```

**3. Memoization / Caching:**

Remembering computed results to avoid redundant calculations:

```javascript
function memoize(fn) {
  const cache = new Map(); // Closure maintains cache
  
  return function(...args) {
    const key = JSON.stringify(args);
    
    if (cache.has(key)) {
      console.log('Cache hit!');
      return cache.get(key);
    }
    
    console.log('Computing...');
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}

// Expensive Fibonacci calculation
function fibonacci(n) {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
}

const fastFib = memoize(fibonacci);

console.log(fastFib(40)); // Computing... (slow first time)
console.log(fastFib(40)); // Cache hit! (instant)

// Memoized API calls
const memoizedFetch = memoize(async (url) => {
  const response = await fetch(url);
  return response.json();
});

await memoizedFetch('/api/users'); // First call - hits server
await memoizedFetch('/api/users'); // Cache hit - instant
```

**4. Event Handlers with Private State:**

Managing state in event handlers without global variables:

```javascript
function setupCounter(buttonId, displayId) {
  let count = 0; // Private to this closure
  const button = document.getElementById(buttonId);
  const display = document.getElementById(displayId);
  
  button.addEventListener('click', function() {
    count++; // Accesses private count
    display.textContent = count;
  });
  
  return {
    reset() {
      count = 0;
      display.textContent = count;
    },
    getCount() {
      return count;
    }
  };
}

const counter1 = setupCounter('btn1', 'display1');
const counter2 = setupCounter('btn2', 'display2');
// Each counter has independent state via closures
```

**5. Partial Application and Currying:**

Creating specialized functions from general ones:

```javascript
// Partial application
function partial(fn, ...fixedArgs) {
  return function(...remainingArgs) {
    return fn(...fixedArgs, ...remainingArgs);
  };
}

function log(level, module, message) {
  console.log(`[${level}] [${module}] ${message}`);
}

const logError = partial(log, 'ERROR');
const logUserError = partial(log, 'ERROR', 'USER');

logError('AUTH', 'Login failed'); // [ERROR] [AUTH] Login failed
logUserError('Invalid email');    // [ERROR] [USER] Invalid email

// Discount calculator
function calculatePrice(basePrice, taxRate, discount) {
  return basePrice * (1 + taxRate) * (1 - discount);
}

const calculateWithTax = partial(calculatePrice, undefined, 0.1);
const calculateWithDiscount = partial(calculatePrice, undefined, 0.1, 0.2);

// More elegant currying
const multiply = a => b => c => a * b * c;
const multiplyBy2 = multiply(2);
const multiplyBy2And3 = multiplyBy2(3);
console.log(multiplyBy2And3(4)); // 24
```

**6. Module Pattern:**

Creating modules with public and private members:

```javascript
const DatabaseModule = (function() {
  // Private variables
  let connection = null;
  const queryCache = new Map();
  const MAX_RETRIES = 3;
  
  // Private functions
  function validateQuery(query) {
    if (!query || typeof query !== 'string') {
      throw new Error('Invalid query');
    }
  }
  
  async function retryOperation(operation, retries = MAX_RETRIES) {
    for (let i = 0; i < retries; i++) {
      try {
        return await operation();
      } catch (error) {
        if (i === retries - 1) throw error;
        await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1)));
      }
    }
  }
  
  // Public API
  return {
    async connect(connectionString) {
      connection = await retryOperation(() => 
        establishConnection(connectionString)
      );
      console.log('Connected to database');
    },
    
    async query(sql) {
      validateQuery(sql);
      
      if (queryCache.has(sql)) {
        return queryCache.get(sql);
      }
      
      const result = await retryOperation(() => 
        connection.execute(sql)
      );
      
      queryCache.set(sql, result);
      return result;
    },
    
    clearCache() {
      queryCache.clear();
    },
    
    async disconnect() {
      if (connection) {
        await connection.close();
        connection = null;
        queryCache.clear();
      }
    }
  };
})();

// Usage
await DatabaseModule.connect('mongodb://localhost');
const users = await DatabaseModule.query('SELECT * FROM users');
```

**7. Debouncing and Throttling:**

Rate limiting function execution:

```javascript
function debounce(fn, delay) {
  let timeoutId; // Closure maintains timeout ID
  
  return function(...args) {
    clearTimeout(timeoutId);
    
    timeoutId = setTimeout(() => {
      fn.apply(this, args);
    }, delay);
  };
}

function throttle(fn, limit) {
  let inThrottle;
  let lastResult;
  
  return function(...args) {
    if (!inThrottle) {
      lastResult = fn.apply(this, args);
      inThrottle = true;
      
      setTimeout(() => {
        inThrottle = false;
      }, limit);
    }
    
    return lastResult;
  };
}

// Search as user types (debounced)
const searchAPI = debounce(function(query) {
  fetch(`/api/search?q=${query}`)
    .then(response => response.json())
    .then(results => displayResults(results));
}, 300);

document.getElementById('search').addEventListener('input', (e) => {
  searchAPI(e.target.value);
});

// Scroll handler (throttled)
const handleScroll = throttle(function() {
  const scrollPercent = (window.scrollY / document.body.scrollHeight) * 100;
  updateScrollIndicator(scrollPercent);
}, 100);

window.addEventListener('scroll', handleScroll);
```

**8. Iterator Pattern:**

Creating custom iterators with internal state:

```javascript
function createRangeIterator(start, end, step = 1) {
  let current = start; // Closure maintains state
  
  return {
    next() {
      if (current <= end) {
        const value = current;
        current += step;
        return { value, done: false };
      }
      return { done: true };
    },
    
    [Symbol.iterator]() {
      return this;
    }
  };
}

const range = createRangeIterator(1, 10, 2);
for (const num of range) {
  console.log(num); // 1, 3, 5, 7, 9
}

// Infinite sequence
function createInfiniteSequence(start = 0) {
  let current = start;
  
  return {
    next() {
      return { value: current++, done: false };
    }
  };
}

const sequence = createInfiniteSequence();
console.log(sequence.next().value); // 0
console.log(sequence.next().value); // 1
console.log(sequence.next().value); // 2
```

**9. Once Function:**

Ensuring a function runs only once:

```javascript
function once(fn) {
  let called = false;
  let result;
  
  return function(...args) {
    if (!called) {
      called = true;
      result = fn.apply(this, args);
    }
    return result;
  };
}

const initializeApp = once(function() {
  console.log('Initializing application...');
  // Expensive setup
  return { initialized: true, timestamp: Date.now() };
});

initializeApp(); // Runs initialization
initializeApp(); // Returns cached result
initializeApp(); // Returns cached result

// Singleton pattern
const DatabaseConnection = (function() {
  let instance;
  
  function createInstance() {
    return {
      connect() { console.log('Connected'); },
      query() { /* ... */ }
    };
  }
  
  return {
    getInstance() {
      if (!instance) {
        instance = createInstance();
      }
      return instance;
    }
  };
})();

const db1 = DatabaseConnection.getInstance();
const db2 = DatabaseConnection.getInstance();
console.log(db1 === db2); // true - same instance
```

**10. React Hooks Pattern:**

State management in functional components (similar to how React Hooks work internally):

```javascript
function createStatefulComponent() {
  let state = {}; // Closure maintains state
  const hooks = [];
  let hookIndex = 0;
  
  function useState(initialValue) {
    const currentIndex = hookIndex;
    hookIndex++;
    
    if (hooks[currentIndex] === undefined) {
      hooks[currentIndex] = initialValue;
    }
    
    const setState = (newValue) => {
      hooks[currentIndex] = newValue;
      render(); // Re-render component
    };
    
    return [hooks[currentIndex], setState];
  }
  
  function useEffect(callback, deps) {
    const currentIndex = hookIndex;
    hookIndex++;
    
    const oldDeps = hooks[currentIndex];
    const hasChanged = !oldDeps || deps.some((dep, i) => dep !== oldDeps[i]);
    
    if (hasChanged) {
      callback();
      hooks[currentIndex] = deps;
    }
  }
  
  function render() {
    hookIndex = 0; // Reset for next render
    component();
  }
  
  function component() {
    const [count, setCount] = useState(0);
    const [name, setName] = useState('Guest');
    
    useEffect(() => {
      console.log(`Count changed to ${count}`);
    }, [count]);
    
    return { count, setCount, name, setName };
  }
  
  return { render, component };
}
```

**Best Practices:**
- Use closures for data privacy (better than private symbols)
- Perfect for factory functions and configuration builders
- Essential for event handlers with state
- Enable functional programming patterns (memoization, currying)
- Be mindful of memory leaks (don't capture unnecessary data)
- Modern alternative to class-based encapsulation
- Combine with module systems for clean architecture

26. What is the scope chain?

**Answer:**
The scope chain is the mechanism JavaScript uses to resolve variable names when they're referenced in code. When a variable is accessed, JavaScript searches for it starting from the current scope and moving up through parent scopes until it finds the variable or reaches the global scope.

**How It Works:**

Each execution context (function or block) has a reference to its outer lexical environment, creating a chain of scopes.

```javascript
const globalVar = 'Global';

function outer() {
  const outerVar = 'Outer';
  
  function middle() {
    const middleVar = 'Middle';
    
    function inner() {
      const innerVar = 'Inner';
      
      // Scope chain lookup order:
      // 1. inner scope (innerVar)
      // 2. middle scope (middleVar)
      // 3. outer scope (outerVar)
      // 4. global scope (globalVar)
      
      console.log(innerVar);   // Found in step 1
      console.log(middleVar);  // Found in step 2
      console.log(outerVar);   // Found in step 3
      console.log(globalVar);  // Found in step 4
      
      // console.log(nonExistent); // ReferenceError: Searched entire chain
    }
    
    inner();
  }
  
  middle();
}

outer();
```

**Visual Representation:**

```
Global Scope
  ↓ (contains)
outer() Scope
  ↓ (contains)
middle() Scope
  ↓ (contains)
inner() Scope

When inner() looks for a variable:
inner → middle → outer → global → ReferenceError (if not found)
```

**Scope Chain Creation:**

```javascript
// Each function "remembers" where it was defined
const x = 'global x';

function first() {
  const x = 'first x';
  
  return function second() {
    const x = 'second x';
    
    return function third() {
      console.log(x); // 'second x' - stops at first match
    };
  };
}

const fn = first()();
fn(); // Scope chain: third → second → first → global
```

**Variable Shadowing:**

When a variable in an inner scope has the same name as one in outer scope:

```javascript
let name = 'Global';

function outer() {
  let name = 'Outer'; // Shadows global 'name'
  
  function inner() {
    let name = 'Inner'; // Shadows outer 'name'
    console.log(name);  // 'Inner' - stops at first match
  }
  
  inner();
  console.log(name); // 'Outer'
}

outer();
console.log(name); // 'Global'
```

**Scope Chain and Closures:**

Closures work because functions maintain references to their scope chain:

```javascript
function createCounter() {
  let count = 0; // In createCounter's scope
  
  return function increment() {
    count++; // Looks up scope chain: increment → createCounter
    return count;
  };
}

const counter = createCounter();
counter(); // 1 - still has access to 'count' via scope chain
counter(); // 2
```

**Production Example - Nested Configuration:**

```javascript
const appConfig = {
  apiUrl: 'https://api.example.com',
  timeout: 5000
};

function createService(serviceName) {
  const serviceUrl = `${appConfig.apiUrl}/${serviceName}`;
  
  return {
    get(endpoint) {
      const fullUrl = `${serviceUrl}${endpoint}`;
      
      return fetch(fullUrl, {
        timeout: appConfig.timeout // Scope chain: method → service → global
      });
    },
    
    post(endpoint, data) {
      const fullUrl = `${serviceUrl}${endpoint}`;
      
      return fetch(fullUrl, {
        method: 'POST',
        body: JSON.stringify(data),
        timeout: appConfig.timeout
      });
    }
  };
}

const userService = createService('users');
userService.get('/profile'); // Access variables from multiple scopes
```

**Scope Chain with `let`, `const`, and `var`:**

```javascript
function testScope() {
  console.log(varVariable);   // undefined (hoisted)
  // console.log(letVariable); // ReferenceError (TDZ)
  
  var varVariable = 'var';
  let letVariable = 'let';
  
  {
    var blockVar = 'not block scoped';
    let blockLet = 'block scoped';
    
    // Scope chain for this block:
    // Current block → testScope → global
  }
  
  console.log(blockVar);     // 'not block scoped' (var ignores blocks)
  // console.log(blockLet);  // ReferenceError (block scoped)
}
```

**Performance Implications:**

```javascript
// Longer scope chains = slower lookups
const global1 = 'value';

function level1() {
  const level1Var = 'value';
  
  function level2() {
    const level2Var = 'value';
    
    function level3() {
      const level3Var = 'value';
      
      function level4() {
        // ⚠️ Deep nesting - slower to access global1
        console.log(global1); // Must traverse entire chain
        
        // ✅ Faster - local variable
        console.log(level3Var);
      }
      
      level4();
    }
    
    level3();
  }
  
  level2();
}

// Optimization: Store frequently accessed variables locally
function optimized() {
  const localGlobal = global1; // Cache in local scope
  
  function inner() {
    console.log(localGlobal); // Faster - shorter lookup
  }
  
  inner();
}
```

**Scope Chain vs Prototype Chain:**

Don't confuse these two concepts:

```javascript
// Scope chain - for variable lookup
function outer() {
  const x = 10;
  
  function inner() {
    console.log(x); // Scope chain lookup
  }
  
  inner();
}

// Prototype chain - for property lookup on objects
const obj = { a: 1 };
console.log(obj.toString()); // Prototype chain: obj → Object.prototype
```

**Common Pitfalls:**

**1. Accidental Global Creation:**
```javascript
function oops() {
  // Forgot 'let' - creates global after scope chain search fails
  someVar = 'value'; // Searches entire scope chain, then creates global
}

'use strict'; // Prevents this
function safe() {
  // someVar = 'value'; // ReferenceError in strict mode
}
```

**2. Loop Variable Capture:**
```javascript
// ❌ All closures share same 'i' in scope chain
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100); // 3, 3, 3
}

// ✅ Each iteration has own scope
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100); // 0, 1, 2
}
```

**3. Memory Leaks from Long Scope Chains:**
```javascript
function createLeak() {
  const largeData = new Array(1000000).fill('data');
  
  return function() {
    // Even if not using largeData, it stays in scope chain
    console.log('hello');
  };
}

// ✅ Better - only keep what you need
function noLeak() {
  const largeData = new Array(1000000).fill('data');
  const summary = largeData.length;
  
  return function() {
    console.log(summary); // Only summary in scope chain, not largeData
  };
}
```

**Debugging Scope Chain:**

```javascript
function debugScopes() {
  const localVar = 'local';
  
  debugger; // Pause here
  
  function inner() {
    const innerVar = 'inner';
    debugger; // Check scope chain in dev tools
    console.log(localVar); // Inspect how it's resolved
  }
  
  inner();
}

// In Chrome DevTools:
// 1. Open Sources tab
// 2. Hit debugger breakpoint
// 3. View "Scope" panel showing scope chain
```

**Best Practices:**

- Minimize scope chain depth for performance
- Use local variables for frequently accessed outer scope values
- Understand that closures extend scope chain lifetime
- Be aware of memory implications with large scope chains
- Use `const` and `let` to create proper block scopes
- Use strict mode to catch accidental global creation
- Keep functions shallow when possible

**Key Takeaways:**
- Scope chain is how JavaScript resolves variable references
- Search goes from inner to outer scopes
- Stops at first match (variable shadowing)
- Created when function is defined (lexical scoping)
- Essential to understanding closures and variable access
- Longer chains can impact performance slightly

27. What is block scope?

**Answer:**
Block scope is a scope created by a pair of curly braces `{}`. Variables declared with `let` and `const` inside a block are only accessible within that block and not outside it. This was introduced in ES6 (2015).

**Block Scope Basics:**

```javascript
{
  // This is a block
  let blockScoped = 'I am block scoped';
  const alsoBlockScoped = 'Me too';
  var notBlockScoped = 'I am NOT block scoped';
  
  console.log(blockScoped);      // ✅ Accessible
  console.log(alsoBlockScoped);  // ✅ Accessible
  console.log(notBlockScoped);   // ✅ Accessible
}

// console.log(blockScoped);      // ❌ ReferenceError
// console.log(alsoBlockScoped);  // ❌ ReferenceError
console.log(notBlockScoped);   // ✅ 'I am NOT block scoped' (var ignores blocks!)
```

**Block Scope in Different Contexts:**

**1. If Statements:**
```javascript
const age = 20;

if (age >= 18) {
  let status = 'adult';
  const canVote = true;
  var message = 'Welcome';
  
  console.log(status);   // ✅ 'adult'
  console.log(canVote);  // ✅ true
}

// console.log(status);   // ❌ ReferenceError
// console.log(canVote);  // ❌ ReferenceError
console.log(message);     // ✅ 'Welcome' (var leaks out)
```

**2. For Loops:**
```javascript
// Classic problem with var
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// Output: 3, 3, 3 (all share same 'i')

// Fixed with let (each iteration has own block scope)
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// Output: 0, 1, 2 (each closure captures its own 'i')

// let creates a new binding for each iteration
for (let i = 0; i < 3; i++) {
  let i = 'inner'; // Different variable!
  console.log(i);  // 'inner', 'inner', 'inner'
}
```

**3. Switch Statements:**
```javascript
const action = 'start';

switch (action) {
  case 'start': {
    // Block scope prevents variable collision
    let message = 'Starting...';
    console.log(message);
    break;
  }
  case 'stop': {
    let message = 'Stopping...'; // Different variable
    console.log(message);
    break;
  }
  default: {
    let message = 'Unknown action';
    console.log(message);
  }
}

// Without blocks - error!
switch (action) {
  case 'start':
    let message = 'Starting...';
    break;
  case 'stop':
    // let message = 'Stopping...'; // ❌ SyntaxError: Identifier 'message' already declared
    break;
}
```

**4. Try-Catch Blocks:**
```javascript
try {
  let result = riskyOperation();
  const data = processData(result);
  console.log(data);
} catch (error) {
  // 'error' is block-scoped to catch block
  console.error(error);
  let retryCount = 0;
  // Handle error
}

// console.log(result); // ❌ ReferenceError
// console.log(error);  // ❌ ReferenceError
```

**5. Standalone Blocks:**
```javascript
// You can create blocks anywhere
{
  let temp = expensiveCalculation();
  let result = process(temp);
  saveResult(result);
}
// temp and result are garbage collected here

// Useful for limiting variable lifetime
{
  const tempData = fetchLargeData();
  processData(tempData);
  // tempData can be garbage collected now
}
```

**Block Scope vs Function Scope:**

```javascript
function testScopes() {
  // Function scope - entire function
  var functionScoped = 'Available throughout function';
  
  if (true) {
    // Block scope - only in this block
    let blockScoped = 'Only in this block';
    var alsoFunctionScoped = 'Also available throughout function';
    
    console.log(blockScoped);      // ✅ Works
    console.log(functionScoped);   // ✅ Works
  }
  
  console.log(functionScoped);       // ✅ Works
  console.log(alsoFunctionScoped);   // ✅ Works (var)
  // console.log(blockScoped);       // ❌ ReferenceError
}
```

**Temporal Dead Zone (TDZ) in Block Scope:**

```javascript
{
  // TDZ for x starts here
  console.log(x); // ❌ ReferenceError: Cannot access before initialization
  let x = 10;     // TDZ ends here
  console.log(x); // ✅ 10
}

{
  console.log(y); // undefined (var is hoisted)
  var y = 20;
  console.log(y); // 20
}
```

**Production Use Cases:**

**1. Limiting Variable Lifetime:**
```javascript
function processUsers(users) {
  const results = [];
  
  for (const user of users) {
    // Each iteration has its own block scope
    let isValid = validateUser(user);
    let processedData = null;
    
    if (isValid) {
      processedData = transformUser(user);
      results.push(processedData);
    }
    
    // isValid and processedData are cleaned up here
  }
  
  return results;
}

// vs using var (all variables stay in memory longer)
function processUsersOld(users) {
  var results = [];
  
  for (var i = 0; i < users.length; i++) {
    var isValid = validateUser(users[i]);
    var processedData = null;
    
    if (isValid) {
      processedData = transformUser(users[i]);
      results.push(processedData);
    }
    
    // isValid and processedData still in memory
  }
  
  return results;
}
```

**2. Event Handlers in Loops:**
```javascript
// ✅ Correct - each handler has its own 'index'
const buttons = document.querySelectorAll('.button');

buttons.forEach((button, index) => {
  // let creates block scope for each iteration
  button.addEventListener('click', () => {
    console.log(`Button ${index} clicked`);
  });
});

// ❌ Wrong - all handlers share same 'i'
for (var i = 0; i < buttons.length; i++) {
  buttons[i].addEventListener('click', () => {
    console.log(`Button ${i} clicked`); // Always logs final value of i
  });
}

// ✅ Fixed with let
for (let i = 0; i < buttons.length; i++) {
  buttons[i].addEventListener('click', () => {
    console.log(`Button ${i} clicked`); // Correct index
  });
}
```

**3. Avoiding Variable Pollution:**
```javascript
function calculateTotal(items) {
  let total = 0;
  
  // Temporary calculation block
  {
    let tax = 0;
    let discount = 0;
    
    for (const item of items) {
      if (item.taxable) {
        tax += item.price * 0.1;
      }
      if (item.onSale) {
        discount += item.price * 0.2;
      }
    }
    
    total = items.reduce((sum, item) => sum + item.price, 0);
    total = total + tax - discount;
  }
  
  // tax and discount are no longer accessible
  return total;
}
```

**4. Switch Statement Safety:**
```javascript
function handleAction(action, data) {
  switch (action) {
    case 'CREATE': {
      let newItem = createItem(data);
      return saveToDatabase(newItem);
    }
    case 'UPDATE': {
      let newItem = updateItem(data); // Different variable, no conflict
      return saveToDatabase(newItem);
    }
    case 'DELETE': {
      return deleteFromDatabase(data);
    }
  }
}
```

**5. Resource Cleanup:**
```javascript
async function processFile(filename) {
  let fileHandle;
  
  try {
    fileHandle = await openFile(filename);
    
    {
      // Processing in its own block
      let data = await fileHandle.read();
      let processed = transformData(data);
      await saveProcessed(processed);
      
      // data and processed can be garbage collected here
    }
    
  } finally {
    if (fileHandle) {
      await fileHandle.close();
    }
  }
}
```

**Block Scope and Hoisting:**

```javascript
console.log(x); // undefined (var is hoisted)
var x = 10;

// console.log(y); // ReferenceError (let is hoisted but in TDZ)
let y = 20;

// The difference:
// var: hoisted and initialized to undefined
// let/const: hoisted but not initialized (TDZ)

{
  // var is hoisted to function/global scope
  var a = 1;
  
  // let is hoisted to block scope only
  let b = 2;
}

console.log(a); // ✅ 1
// console.log(b); // ❌ ReferenceError
```

**Nested Block Scopes:**

```javascript
let outer = 'outer';

{
  let outer = 'first block';
  console.log(outer); // 'first block'
  
  {
    let outer = 'second block';
    console.log(outer); // 'second block'
    
    {
      let outer = 'third block';
      console.log(outer); // 'third block'
    }
    
    console.log(outer); // 'second block'
  }
  
  console.log(outer); // 'first block'
}

console.log(outer); // 'outer'
```

**Best Practices:**

```javascript
// ✅ Use const by default
const API_URL = 'https://api.example.com';

// ✅ Use let when you need to reassign
let counter = 0;
counter++;

// ❌ Avoid var in modern code
// var oldStyle = 'avoid this';

// ✅ Create blocks to limit scope
{
  const tempData = getLargeData();
  processData(tempData);
  // tempData released here
}

// ✅ Use block scope in loops
for (let i = 0; i < 10; i++) {
  // Each iteration has its own 'i'
}

// ✅ Block scope in switch statements
switch (type) {
  case 'A': {
    const result = handleA();
    return result;
  }
  case 'B': {
    const result = handleB();
    return result;
  }
}
```

**Key Takeaways:**
- Block scope applies to `let` and `const` (not `var`)
- Created by `{}` in if, for, while, switch, try-catch, or standalone blocks
- Variables are only accessible within their block
- Each loop iteration with `let` creates a new block scope
- Helps prevent variable collisions and memory leaks
- Part of ES6, essential for modern JavaScript
- Use `const` by default, `let` when reassignment needed, avoid `var`

28. What is function scope?

**Answer:**
Function scope means that variables declared inside a function are only accessible within that function and its nested functions. This is the traditional scoping mechanism in JavaScript and applies to variables declared with `var`, as well as function parameters.

**Basic Function Scope:**

```javascript
function myFunction() {
  var functionScoped = 'I am function scoped';
  let alsoFunctionScoped = 'Me too';
  const andMe = 'And me';
  
  console.log(functionScoped);      // ✅ Accessible
  console.log(alsoFunctionScoped);  // ✅ Accessible
  console.log(andMe);               // ✅ Accessible
}

myFunction();
// console.log(functionScoped);      // ❌ ReferenceError
// console.log(alsoFunctionScoped);  // ❌ ReferenceError
// console.log(andMe);               // ❌ ReferenceError
```

**Function Scope vs Block Scope:**

```javascript
function demonstrateScopes() {
  // Function-scoped with var
  var functionVar = 'Available throughout function';
  
  if (true) {
    var stillFunctionScoped = 'Also throughout function';
    let blockScoped = 'Only in this block';
    
    console.log(functionVar);         // ✅ Works
    console.log(stillFunctionScoped); // ✅ Works
    console.log(blockScoped);         // ✅ Works
  }
  
  console.log(functionVar);           // ✅ Works
  console.log(stillFunctionScoped);   // ✅ Works (var ignores block)
  // console.log(blockScoped);        // ❌ ReferenceError (block-scoped)
}
```

**Function Parameters are Function-Scoped:**

```javascript
function greet(name, age) {
  // name and age are function-scoped
  console.log(name); // Accessible
  console.log(age);  // Accessible
  
  function inner() {
    // Parameters accessible in nested functions
    console.log(name); // ✅ Accessible via scope chain
  }
  
  inner();
}

greet('John', 30);
// console.log(name); // ❌ ReferenceError
```

**Nested Functions and Function Scope:**

```javascript
function outer() {
  var outerVar = 'Outer';
  
  function inner() {
    var innerVar = 'Inner';
    
    console.log(outerVar); // ✅ Accessible (parent function scope)
    console.log(innerVar); // ✅ Accessible (own scope)
  }
  
  inner();
  console.log(outerVar);   // ✅ Accessible
  // console.log(innerVar); // ❌ ReferenceError (child scope not accessible)
}

outer();
```

**var is Function-Scoped (Key Difference from let/const):**

```javascript
function testVar() {
  console.log(x); // undefined (hoisted but not initialized)
  
  if (true) {
    var x = 10; // Function-scoped, not block-scoped
  }
  
  console.log(x); // ✅ 10 (accessible outside if block)
  
  for (var i = 0; i < 3; i++) {
    // i is function-scoped
  }
  
  console.log(i); // ✅ 3 (accessible outside loop)
}

function testLet() {
  // console.log(y); // ReferenceError (TDZ)
  
  if (true) {
    let y = 10; // Block-scoped
  }
  
  // console.log(y); // ❌ ReferenceError
  
  for (let j = 0; j < 3; j++) {
    // j is block-scoped to loop
  }
  
  // console.log(j); // ❌ ReferenceError
}
```

**Function Scope and Hoisting:**

```javascript
function hoistingDemo() {
  console.log(a); // undefined (hoisted)
  console.log(b); // undefined (hoisted)
  
  var a = 10;
  
  if (true) {
    var b = 20; // Still hoisted to function scope
  }
  
  console.log(a); // 10
  console.log(b); // 20
}

// Behind the scenes (conceptually):
function hoistingDemo() {
  var a; // Hoisted to top of function
  var b; // Hoisted to top of function
  
  console.log(a); // undefined
  console.log(b); // undefined
  
  a = 10;
  
  if (true) {
    b = 20;
  }
  
  console.log(a); // 10
  console.log(b); // 20
}
```

**Function Declarations are Function-Scoped:**

```javascript
function outer() {
  // Function declaration is hoisted in function scope
  console.log(inner()); // ✅ Works (hoisted)
  
  function inner() {
    return 'Inner function';
  }
  
  if (true) {
    // Function declaration in block (behavior varies by mode)
    function blockFunction() {
      return 'Block function';
    }
  }
  
  // In non-strict mode, blockFunction might be accessible
  // In strict mode, it's block-scoped
}
```

**Production Use Cases:**

**1. Module Pattern (Pre-ES6 Modules):**
```javascript
const counterModule = (function() {
  // Function scope creates private variables
  var count = 0;
  var maxCount = 100;
  
  function validateCount(newCount) {
    return newCount >= 0 && newCount <= maxCount;
  }
  
  // Return public API
  return {
    increment: function() {
      if (validateCount(count + 1)) {
        count++;
      }
      return count;
    },
    
    decrement: function() {
      if (validateCount(count - 1)) {
        count--;
      }
      return count;
    },
    
    getCount: function() {
      return count;
    },
    
    reset: function() {
      count = 0;
    }
  };
})();

counterModule.increment(); // 1
counterModule.increment(); // 2
// counterModule.count;    // undefined (private!)
// counterModule.maxCount; // undefined (private!)
```

**2. Immediately Invoked Function Expression (IIFE):**
```javascript
// Create isolated function scope
(function() {
  var privateVar = 'Private';
  var helper = function() { /* ... */ };
  
  // Only expose what's needed
  window.MyLibrary = {
    publicMethod: function() {
      return privateVar;
    }
  };
})();

// privateVar and helper are not accessible globally
```

**3. Closure with Function Scope:**
```javascript
function createCounter(initialCount) {
  var count = initialCount; // Function-scoped, private
  
  return {
    increment: function() {
      count++;
      return count;
    },
    decrement: function() {
      count--;
      return count;
    },
    getCount: function() {
      return count;
    }
  };
}

const counter1 = createCounter(0);
const counter2 = createCounter(10);

counter1.increment(); // 1
counter2.increment(); // 11
// Each has its own function scope
```

**4. Loop Variable Problem with Function Scope:**
```javascript
// ❌ Classic problem with var (function-scoped)
var callbacks = [];

for (var i = 0; i < 3; i++) {
  callbacks.push(function() {
    console.log(i); // All closures share same 'i'
  });
}

callbacks[0](); // 3
callbacks[1](); // 3
callbacks[2](); // 3

// ✅ Solution 1: IIFE creates new function scope
var callbacks = [];

for (var i = 0; i < 3; i++) {
  callbacks.push((function(index) {
    return function() {
      console.log(index); // Each has its own 'index'
    };
  })(i));
}

callbacks[0](); // 0
callbacks[1](); // 1
callbacks[2](); // 2

// ✅ Solution 2: Use let (block-scoped)
const callbacks = [];

for (let i = 0; i < 3; i++) {
  callbacks.push(function() {
    console.log(i); // Each iteration has its own 'i'
  });
}

callbacks[0](); // 0
callbacks[1](); // 1
callbacks[2](); // 2
```

**5. Avoiding Global Pollution:**
```javascript
// ❌ Bad - pollutes global scope
var config = { /* ... */ };
var helper1 = function() { /* ... */ };
var helper2 = function() { /* ... */ };

// ✅ Good - use function scope
(function() {
  var config = { /* ... */ };
  var helper1 = function() { /* ... */ };
  var helper2 = function() { /* ... */ };
  
  // Initialize app
  init();
})();

// ✅ Modern - use modules
// config.js
export const config = { /* ... */ };

// helpers.js
export function helper1() { /* ... */ }
export function helper2() { /* ... */ }
```

**Function Scope and `this`:**

```javascript
function Person(name) {
  var self = this; // Store reference in function scope
  
  this.name = name;
  this.age = 0;
  
  // setTimeout uses different 'this'
  setTimeout(function() {
    // this.age++; // ❌ 'this' is not Person instance
    self.age++;    // ✅ Uses stored reference
    console.log(self.name, self.age);
  }, 1000);
  
  // Modern solution: arrow function (lexical this)
  setTimeout(() => {
    this.age++; // ✅ Arrow function inherits 'this'
    console.log(this.name, this.age);
  }, 1000);
}
```

**Function Scope in Classes:**

```javascript
class Counter {
  constructor() {
    this.count = 0;
    
    // Function expression - can access class scope
    this.increment = function() {
      this.count++; // 'this' refers to instance
    };
  }
  
  // Method - on prototype
  decrement() {
    this.count--;
  }
}

const counter = new Counter();
counter.increment(); // Works
counter.decrement(); // Works
```

**Best Practices:**

```javascript
// ✅ Use let/const instead of var in modern code
function modern() {
  const data = fetchData();
  let count = 0;
  
  for (let i = 0; i < data.length; i++) {
    count += data[i];
  }
  
  return count;
}

// ❌ Avoid var (function-scoped, confusing)
function legacy() {
  var data = fetchData();
  var count = 0;
  
  for (var i = 0; i < data.length; i++) {
    count += data[i];
  }
  
  return count;
}

// ✅ Use IIFE for isolation when needed
(function() {
  // Isolated function scope
  const privateData = { /* ... */ };
  
  // Expose only what's needed
  window.MyAPI = {
    getData() { return { ...privateData }; }
  };
})();

// ✅ Use modules (ES6+) instead of IIFE
// myModule.js
const privateData = { /* ... */ };

export function getData() {
  return { ...privateData };
}
```

**Key Takeaways:**
- Function scope means variables are accessible throughout the entire function
- Applies to `var`, function parameters, and function declarations
- Variables are hoisted to the top of the function
- `var` is function-scoped (ignores blocks like if, for, while)
- `let` and `const` are block-scoped (more restrictive)
- Function scope enables closures and private data
- Modern code prefers block-scoped `let`/`const` over function-scoped `var`
- IIFE pattern uses function scope for encapsulation
- Understanding function scope is essential for understanding closures
