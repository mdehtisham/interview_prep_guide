## ES6+ Features
61. What are template literals?

**Answer:**
Template literals (template strings) are string literals enclosed by backticks (`` ` ``) that support string interpolation, multi-line strings, and embedded expressions using `${}` syntax. They provide a cleaner and more powerful way to work with strings compared to regular string concatenation.

**Basic Syntax:**

```javascript
// Regular string
const regular = 'Hello, World!';

// Template literal
const template = `Hello, World!`;

// Both are strings
console.log(typeof regular);  // 'string'
console.log(typeof template); // 'string'
```

**String Interpolation:**

```javascript
// Old way - concatenation
const name = 'John';
const age = 30;
const message = 'Hello, my name is ' + name + ' and I am ' + age + ' years old.';

// ✅ Template literal - cleaner
const message = `Hello, my name is ${name} and I am ${age} years old.`;
console.log(message); // 'Hello, my name is John and I am 30 years old.'

// Any expression works
const a = 5;
const b = 10;
console.log(`Sum: ${a + b}`);          // 'Sum: 15'
console.log(`Product: ${a * b}`);      // 'Product: 50'
console.log(`Is greater? ${a > b}`);   // 'Is greater? false'
```

**Embedded Expressions:**

```javascript
// Function calls
function greet(name) {
  return `Hello, ${name.toUpperCase()}!`;
}
console.log(`Message: ${greet('john')}`); // 'Message: Hello, JOHN!'

// Object properties
const user = { name: 'Alice', age: 25 };
console.log(`User: ${user.name}, Age: ${user.age}`);

// Array methods
const numbers = [1, 2, 3, 4, 5];
console.log(`Total: ${numbers.reduce((a, b) => a + b)}`); // 'Total: 15'

// Conditional (ternary) operator
const age = 18;
console.log(`You are ${age >= 18 ? 'an adult' : 'a minor'}`);

// Complex expressions
const price = 29.99;
const quantity = 3;
const tax = 0.08;
console.log(`Total: $${(price * quantity * (1 + tax)).toFixed(2)}`);
// 'Total: $97.17'
```

**Multi-line Strings:**

```javascript
// Old way - concatenation with \n
const oldMultiline = 'Line 1\n' +
                     'Line 2\n' +
                     'Line 3';

// ✅ Template literal - natural multi-line
const newMultiline = `Line 1
Line 2
Line 3`;

console.log(newMultiline);
// Line 1
// Line 2
// Line 3

// HTML templates
const html = `
  <div class="card">
    <h2>${user.name}</h2>
    <p>${user.email}</p>
  </div>
`;

// SQL queries
const query = `
  SELECT *
  FROM users
  WHERE age > ${minAge}
    AND active = true
  ORDER BY created_at DESC
`;
```

**Preserves Whitespace:**

```javascript
// Indentation is preserved
const code = `
  function example() {
    console.log('Hello');
  }
`;

console.log(code);
// (includes leading spaces and newlines)

// Be careful with indentation
const html = `
    <div>
      <p>Content</p>
    </div>
`; // Includes the indentation!

// Use trim() if needed
const trimmedHtml = `
    <div>
      <p>Content</p>
    </div>
`.trim();
```

**Escaping Backticks:**

```javascript
// Escape backticks with backslash
const message = `This is a backtick: \``;
console.log(message); // 'This is a backtick: `'

// Nested template literals (no escape needed)
const outer = `Outer ${`Inner`}`;
console.log(outer); // 'Outer Inner'

// Code example in template
const codeExample = `Use \`console.log()\` to print`;
console.log(codeExample); // 'Use `console.log()` to print'
```

**Tagged Template Literals:**

```javascript
// Custom processing with tag functions
function highlight(strings, ...values) {
  return strings.reduce((result, str, i) => {
    return result + str + (values[i] ? `<mark>${values[i]}</mark>` : '');
  }, '');
}

const name = 'John';
const age = 30;
const result = highlight`Name: ${name}, Age: ${age}`;
console.log(result);
// 'Name: <mark>John</mark>, Age: <mark>30</mark>'

// strings: ['Name: ', ', Age: ', '']
// values: ['John', 30]
```

**Tagged Template Use Cases:**

**1. Sanitization/Escaping:**

```javascript
function html(strings, ...values) {
  return strings.reduce((result, str, i) => {
    const value = values[i];
    if (value == null) {
      return result + str;
    }
    
    // Escape HTML
    const escaped = String(value)
      .replace(/&/g, '&amp;')
      .replace(/</g, '&lt;')
      .replace(/>/g, '&gt;')
      .replace(/"/g, '&quot;')
      .replace(/'/g, '&#039;');
    
    return result + str + escaped;
  }, '');
}

const userInput = '<script>alert("XSS")</script>';
const safe = html`<div>${userInput}</div>`;
console.log(safe);
// '<div>&lt;script&gt;alert(&quot;XSS&quot;)&lt;/script&gt;</div>'
```

**2. Internationalization (i18n):**

```javascript
function i18n(strings, ...values) {
  const translations = {
    'Hello, ': 'Bonjour, ',
    '! You have ': '! Vous avez ',
    ' messages.': ' messages.'
  };
  
  return strings.reduce((result, str, i) => {
    const translated = translations[str] || str;
    return result + translated + (values[i] || '');
  }, '');
}

const name = 'Marie';
const count = 5;
const message = i18n`Hello, ${name}! You have ${count} messages.`;
console.log(message);
// 'Bonjour, Marie! Vous avez 5 messages.'
```

**3. SQL Parameterization:**

```javascript
function sql(strings, ...values) {
  const query = strings.reduce((result, str, i) => {
    return result + str + (values[i] !== undefined ? `$${i + 1}` : '');
  }, '');
  
  return {
    text: query,
    values: values
  };
}

const userId = 123;
const minAge = 18;
const query = sql`
  SELECT * FROM users
  WHERE id = ${userId}
    AND age > ${minAge}
`;

console.log(query);
// {
//   text: 'SELECT * FROM users WHERE id = $1 AND age > $2',
//   values: [123, 18]
// }
```

**4. Styling (CSS-in-JS):**

```javascript
function css(strings, ...values) {
  return strings.reduce((result, str, i) => {
    return result + str + (values[i] || '');
  }, '');
}

const primaryColor = '#007bff';
const padding = '20px';

const styles = css`
  .button {
    background-color: ${primaryColor};
    padding: ${padding};
    border: none;
    border-radius: 4px;
  }
`;

console.log(styles);
// CSS string with interpolated values
```

**Practical Examples:**

**1. Building URLs:**

```javascript
const baseUrl = 'https://api.example.com';
const endpoint = 'users';
const userId = 123;
const query = 'active=true&sort=name';

const url = `${baseUrl}/${endpoint}/${userId}?${query}`;
console.log(url);
// 'https://api.example.com/users/123?active=true&sort=name'

// Dynamic API request
async function fetchUser(id) {
  const response = await fetch(`${baseUrl}/users/${id}`);
  return response.json();
}
```

**2. Generating HTML:**

```javascript
function createCard(user) {
  return `
    <div class="card">
      <img src="${user.avatar}" alt="${user.name}" />
      <h2>${user.name}</h2>
      <p>${user.bio}</p>
      <button data-id="${user.id}">Follow</button>
    </div>
  `;
}

const user = {
  id: 1,
  name: 'John Doe',
  avatar: '/avatars/john.jpg',
  bio: 'Software developer'
};

document.body.innerHTML = createCard(user);
```

**3. Logging and Debugging:**

```javascript
function log(message, data) {
  const timestamp = new Date().toISOString();
  console.log(`[${timestamp}] ${message}:`, data);
}

log('User logged in', { userId: 123, ip: '192.168.1.1' });
// '[2024-01-15T10:30:00.000Z] User logged in: { userId: 123, ip: '192.168.1.1' }'

// Error messages
function validateAge(age) {
  if (age < 18) {
    throw new Error(`Invalid age: ${age}. Must be 18 or older.`);
  }
}
```

**4. Dynamic Content:**

```javascript
function generateGreeting(name, time) {
  const hour = new Date().getHours();
  const greeting = hour < 12 ? 'Good morning' : 
                   hour < 18 ? 'Good afternoon' : 
                   'Good evening';
  
  return `${greeting}, ${name}! The current time is ${time}.`;
}

console.log(generateGreeting('Alice', '14:30'));
// 'Good afternoon, Alice! The current time is 14:30.'
```

**5. Configuration:**

```javascript
const config = {
  api: {
    url: 'https://api.example.com',
    version: 'v2',
    key: 'abc123'
  }
};

const endpoint = `${config.api.url}/${config.api.version}/users`;
const headers = {
  'Authorization': `Bearer ${config.api.key}`,
  'Content-Type': 'application/json'
};
```

**Comparison with String Concatenation:**

```javascript
const name = 'John';
const age = 30;
const city = 'New York';

// Concatenation - hard to read
const concat = 'My name is ' + name + ', I am ' + age + ' years old, and I live in ' + city + '.';

// Template literal - clean and readable
const template = `My name is ${name}, I am ${age} years old, and I live in ${city}.`;

// Performance: Nearly identical for simple cases
// Readability: Template literals win
```

**Nesting Template Literals:**

```javascript
// Template literals can be nested
const name = 'John';
const items = ['apple', 'banana', 'orange'];

const message = `Hello ${name}, you have ${items.length} items: ${
  items.map(item => `"${item}"`).join(', ')
}`;

console.log(message);
// 'Hello John, you have 3 items: "apple", "banana", "orange"'

// Complex nesting
const users = [
  { name: 'Alice', age: 25 },
  { name: 'Bob', age: 30 }
];

const html = `
  <ul>
    ${users.map(user => `
      <li>${user.name} (${user.age})</li>
    `).join('')}
  </ul>
`;
```

**Common Patterns:**

```javascript
// Conditional content
const status = 'active';
const message = `User is ${status === 'active' ? 'currently active' : 'inactive'}`;

// Optional content
const middleName = 'James';
const fullName = `John${middleName ? ` ${middleName}` : ''} Doe`;

// Pluralization
const count = 5;
const text = `You have ${count} ${count === 1 ? 'item' : 'items'}`;

// Number formatting
const price = 1234.56;
const formatted = `Price: $${price.toLocaleString('en-US', { minimumFractionDigits: 2 })}`;
// 'Price: $1,234.56'
```

**Best Practices:**
- Use template literals for **string interpolation**
- Prefer over concatenation for **readability**
- Great for **multi-line strings** (HTML, SQL, etc.)
- Use **tagged templates** for custom processing
- **Sanitize user input** to prevent XSS attacks
- Be aware of **whitespace preservation**
- Use `trim()` for **cleaner multi-line strings**
- Expressions in `${}` are **evaluated immediately**

**Key Takeaways:**
- Template literals use backticks (`` ` ``)
- Support **string interpolation** with `${expression}`
- Enable **multi-line strings** without \n
- Cleaner than string concatenation
- Support **any JavaScript expression** in ${}
- **Tagged templates** for custom processing
- Whitespace is preserved (including indentation)
- Part of ES6, widely supported in modern browsers

---

### 62. What is destructuring in JavaScript?

**Answer:**

Destructuring is an ES6 feature that allows extracting values from arrays or properties from objects into distinct variables using a convenient syntax. It provides a concise way to unpack values from data structures.

#### **Array Destructuring**

```javascript
// Basic array destructuring
const colors = ['red', 'green', 'blue'];
const [first, second, third] = colors;
console.log(first);   // 'red'
console.log(second);  // 'green'
console.log(third);   // 'blue'

// Skipping elements
const [primary, , tertiary] = colors;
console.log(primary);   // 'red'
console.log(tertiary);  // 'blue'

// Default values
const [a, b, c, d = 'yellow'] = colors;
console.log(d);  // 'yellow' (default when undefined)

// Rest pattern
const numbers = [1, 2, 3, 4, 5];
const [head, ...tail] = numbers;
console.log(head);  // 1
console.log(tail);  // [2, 3, 4, 5]

// Swapping variables
let x = 10, y = 20;
[x, y] = [y, x];
console.log(x, y);  // 20, 10

// Nested array destructuring
const nested = [1, [2, 3], 4];
const [num1, [num2, num3], num4] = nested;
console.log(num2);  // 2
```

#### **Object Destructuring**

```javascript
// Basic object destructuring
const user = {
  name: 'John',
  age: 30,
  email: 'john@example.com'
};

const { name, age, email } = user;
console.log(name);   // 'John'
console.log(age);    // 30
console.log(email);  // 'john@example.com'

// Renaming variables
const { name: userName, age: userAge } = user;
console.log(userName);  // 'John'
console.log(userAge);   // 30

// Default values
const { name, country = 'USA' } = user;
console.log(country);  // 'USA' (default)

// Nested object destructuring
const employee = {
  id: 101,
  info: {
    name: 'Alice',
    department: {
      name: 'Engineering',
      location: 'Building A'
    }
  }
};

const { 
  info: { 
    name, 
    department: { name: deptName, location } 
  } 
} = employee;
console.log(name);      // 'Alice'
console.log(deptName);  // 'Engineering'
console.log(location);  // 'Building A'

// Rest properties
const { id, ...otherInfo } = employee;
console.log(id);         // 101
console.log(otherInfo);  // { info: {...} }

// Computed property names
const key = 'name';
const { [key]: value } = user;
console.log(value);  // 'John'
```

#### **Function Parameter Destructuring**

```javascript
// Object parameters
function displayUser({ name, age, email = 'N/A' }) {
  console.log(`${name}, ${age} years old, ${email}`);
}

displayUser({ name: 'Bob', age: 25 });
// Output: "Bob, 25 years old, N/A"

// Array parameters
function sum([a, b]) {
  return a + b;
}
console.log(sum([10, 20]));  // 30

// Nested destructuring in parameters
function processOrder({ 
  orderId, 
  customer: { name, address: { city } } 
}) {
  console.log(`Order ${orderId} for ${name} in ${city}`);
}

processOrder({
  orderId: 'A123',
  customer: {
    name: 'Charlie',
    address: { city: 'New York', zip: '10001' }
  }
});
// Output: "Order A123 for Charlie in New York"

// With default values
function createUser({ 
  name = 'Guest', 
  role = 'user', 
  active = true 
} = {}) {
  return { name, role, active };
}

console.log(createUser());  // { name: 'Guest', role: 'user', active: true }
console.log(createUser({ name: 'Admin' }));  // { name: 'Admin', role: 'user', active: true }
```

#### **Practical Use Cases**

**1. API Response Handling:**
```javascript
// Extract only needed data from API response
async function fetchUser(userId) {
  const response = await fetch(`/api/users/${userId}`);
  const { data: { name, email, profile: { avatar } } } = await response.json();
  return { name, email, avatar };
}

// Multiple return values
function getCoordinates() {
  return { latitude: 40.7128, longitude: -74.0060 };
}
const { latitude, longitude } = getCoordinates();
```

**2. React Component Props:**
```javascript
// Clean component syntax
function UserCard({ name, age, email, avatar = '/default-avatar.png' }) {
  return (
    <div className="user-card">
      <img src={avatar} alt={name} />
      <h3>{name}</h3>
      <p>{age} years old</p>
      <p>{email}</p>
    </div>
  );
}

// Props with nested destructuring
function Profile({ user: { name, settings: { theme, language } } }) {
  return <div data-theme={theme} data-lang={language}>{name}</div>;
}
```

**3. Configuration Objects:**
```javascript
function initializeApp({
  apiUrl = 'https://api.example.com',
  timeout = 5000,
  retries = 3,
  headers = {},
  ...options
} = {}) {
  console.log(`API: ${apiUrl}, Timeout: ${timeout}ms`);
  console.log('Additional options:', options);
}

initializeApp({ apiUrl: 'https://custom-api.com', debug: true });
```

**4. Array Operations:**
```javascript
// Extracting from regex matches
const url = 'https://example.com:8080/path';
const [, protocol, domain, port, path] = 
  url.match(/^(https?):\/\/([^:\/]+):(\d+)(.*)$/) || [];

console.log(protocol, domain, port, path);
// 'https', 'example.com', '8080', '/path'

// Working with iterators
const map = new Map([['a', 1], ['b', 2]]);
for (const [key, value] of map) {
  console.log(`${key}: ${value}`);
}
```

**5. Module Imports:**
```javascript
// Import specific functions
import { useState, useEffect, useMemo } from 'react';

// Destructure from default import
import React, { Component, Fragment } from 'react';

// Rename imports
import { default as lodash, map as lodashMap } from 'lodash';
```

#### **Common Patterns and Best Practices**

```javascript
// 1. Handling optional chaining with destructuring
const user = null;
const { name = 'Guest' } = user || {};  // Safe destructuring
console.log(name);  // 'Guest'

// 2. Mixed array and object destructuring
const response = {
  status: 200,
  data: ['item1', 'item2', 'item3']
};
const { status, data: [first, ...rest] } = response;

// 3. Destructuring in loops
const users = [
  { id: 1, name: 'Alice' },
  { id: 2, name: 'Bob' }
];

users.forEach(({ id, name }) => {
  console.log(`${id}: ${name}`);
});

// 4. Conditional destructuring
const config = { mode: 'production' };
const { 
  mode, 
  debug = mode === 'development'  // Computed default
} = config;

// 5. Aliasing with defaults
const { 
  name: userName = 'Anonymous',
  age: userAge = 0 
} = {};
console.log(userName, userAge);  // 'Anonymous', 0
```

#### **Common Pitfalls**

```javascript
// 1. Destructuring null or undefined throws error
const obj = null;
// const { prop } = obj;  // TypeError!
const { prop } = obj || {};  // Safe with fallback

// 2. Cannot redeclare with let/const
let x = 10;
// let { x } = { x: 20 };  // SyntaxError: Identifier 'x' has already been declared
({ x } = { x: 20 });  // OK - assignment without declaration (note parentheses)

// 3. Mixing default values with renaming
const { name: userName = 'Guest' } = {};  // Correct
// const { name = 'Guest': userName } = {};  // Syntax error

// 4. Deep destructuring can fail if intermediate is undefined
const data = { user: null };
// const { user: { name } } = data;  // TypeError!
const { user: { name } = {} } = data || {};  // Safe

// 5. Rest pattern must be last
// const [...rest, last] = [1, 2, 3];  // SyntaxError
const [first, ...rest] = [1, 2, 3];  // Correct
```

#### **Performance Considerations**

```javascript
// Destructuring has minimal overhead
// Before (multiple property accesses)
function process(config) {
  const url = config.url;
  const method = config.method;
  const headers = config.headers;
  // ... use url, method, headers
}

// After (single destructuring)
function process(config) {
  const { url, method, headers } = config;
  // ... use url, method, headers
}

// Both have similar performance, but destructuring is more readable
```

#### **Comparison with Traditional Approach**

| Feature | Traditional | Destructuring |
|---------|-------------|---------------|
| **Syntax** | `const name = user.name;` | `const { name } = user;` |
| **Multiple vars** | Multiple lines | Single line |
| **Default values** | `\|\|` operator | Built-in syntax |
| **Renaming** | New variable needed | `{ name: newName }` |
| **Nested access** | Multiple dots | Nested pattern |
| **Array swapping** | Temp variable needed | `[a, b] = [b, a]` |
| **Readability** | Verbose | Concise |
| **Safety** | Manual checks | Pattern-based |

#### **Key Takeaways**

- **Concise syntax** for extracting values from arrays and objects
- Works with **arrays, objects, nested structures**, and **function parameters**
- Supports **default values, renaming, rest patterns**, and **computed properties**
- Enables **variable swapping** without temporary variables
- Common in **modern JavaScript** (React, ES modules, API handling)
- **Fails on null/undefined** - use fallbacks (`|| {}`) for safety
- **Rest pattern must be last** in destructuring assignment
- Greatly improves **code readability** and reduces boilerplate
- Works with **iterables** (arrays, Maps, Sets, strings)
- Combining destructuring with default parameters creates flexible APIs

---

### 63. What are classes in JavaScript?

**Answer:**

Classes in JavaScript are syntactic sugar over the existing prototype-based inheritance. Introduced in ES6, they provide a cleaner and more intuitive syntax for creating objects and implementing inheritance, similar to class-based languages like Java or C++.

#### **Basic Class Syntax**

```javascript
// Class declaration
class Person {
  // Constructor method
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
  
  // Instance method
  greet() {
    return `Hello, my name is ${this.name}`;
  }
  
  // Instance method
  getDetails() {
    return `${this.name} is ${this.age} years old`;
  }
}

// Creating instances
const person1 = new Person('Alice', 30);
const person2 = new Person('Bob', 25);

console.log(person1.greet());        // "Hello, my name is Alice"
console.log(person2.getDetails());   // "Bob is 25 years old"
console.log(person1 instanceof Person);  // true

// Class expression (less common)
const Animal = class {
  constructor(name) {
    this.name = name;
  }
};

// Named class expression
const Cat = class Feline {
  constructor(name) {
    this.name = name;
  }
};
```

#### **Constructor Method**

```javascript
class User {
  constructor(username, email) {
    // Initialize instance properties
    this.username = username;
    this.email = email;
    this.createdAt = new Date();
    
    // Validation in constructor
    if (!email.includes('@')) {
      throw new Error('Invalid email address');
    }
  }
}

const user = new User('john_doe', 'john@example.com');
console.log(user.username);   // 'john_doe'
console.log(user.createdAt);  // Current date

// Constructor is optional
class EmptyClass {
  // No constructor - default empty constructor is used
}

const instance = new EmptyClass();
```

#### **Instance Methods**

```javascript
class Calculator {
  constructor() {
    this.result = 0;
  }
  
  add(num) {
    this.result += num;
    return this;  // Method chaining
  }
  
  subtract(num) {
    this.result -= num;
    return this;
  }
  
  multiply(num) {
    this.result *= num;
    return this;
  }
  
  getResult() {
    return this.result;
  }
  
  reset() {
    this.result = 0;
    return this;
  }
}

// Method chaining
const calc = new Calculator();
const result = calc.add(10).multiply(2).subtract(5).getResult();
console.log(result);  // 15
```

#### **Static Methods and Properties**

```javascript
class MathHelper {
  // Static property
  static PI = 3.14159;
  
  // Static method - called on class, not instances
  static add(a, b) {
    return a + b;
  }
  
  static multiply(a, b) {
    return a * b;
  }
  
  static circleArea(radius) {
    return MathHelper.PI * radius * radius;
  }
}

// Call static methods on class itself
console.log(MathHelper.add(5, 3));           // 8
console.log(MathHelper.circleArea(5));       // 78.53975
console.log(MathHelper.PI);                  // 3.14159

const helper = new MathHelper();
// console.log(helper.add(5, 3));  // TypeError - not available on instance

// Common use case: Factory methods
class User {
  constructor(name, role) {
    this.name = name;
    this.role = role;
  }
  
  static createAdmin(name) {
    return new User(name, 'admin');
  }
  
  static createGuest(name) {
    return new User(name, 'guest');
  }
}

const admin = User.createAdmin('Alice');
const guest = User.createGuest('Bob');
console.log(admin.role);  // 'admin'
console.log(guest.role);  // 'guest'
```

#### **Getters and Setters**

```javascript
class Rectangle {
  constructor(width, height) {
    this._width = width;    // Convention: underscore for private-like
    this._height = height;
  }
  
  // Getter - accessed like a property
  get width() {
    return this._width;
  }
  
  // Setter - assigned like a property
  set width(value) {
    if (value > 0) {
      this._width = value;
    } else {
      throw new Error('Width must be positive');
    }
  }
  
  get height() {
    return this._height;
  }
  
  set height(value) {
    if (value > 0) {
      this._height = value;
    } else {
      throw new Error('Height must be positive');
    }
  }
  
  // Computed property using getter
  get area() {
    return this._width * this._height;
  }
  
  get perimeter() {
    return 2 * (this._width + this._height);
  }
}

const rect = new Rectangle(10, 5);
console.log(rect.width);      // 10 (getter)
console.log(rect.area);       // 50 (computed)

rect.width = 20;              // setter
console.log(rect.area);       // 100 (recomputed)

// rect.width = -5;  // Error: Width must be positive
```

#### **Class Inheritance with `extends`**

```javascript
// Parent class
class Animal {
  constructor(name, species) {
    this.name = name;
    this.species = species;
  }
  
  makeSound() {
    return `${this.name} makes a sound`;
  }
  
  describe() {
    return `${this.name} is a ${this.species}`;
  }
}

// Child class extends parent
class Dog extends Animal {
  constructor(name, breed) {
    // Must call super() before using 'this'
    super(name, 'Dog');  // Calls parent constructor
    this.breed = breed;
  }
  
  // Override parent method
  makeSound() {
    return `${this.name} barks: Woof! Woof!`;
  }
  
  // New method specific to Dog
  fetch() {
    return `${this.name} is fetching the ball`;
  }
  
  // Call parent method with super
  getFullDescription() {
    return `${super.describe()} of breed ${this.breed}`;
  }
}

const dog = new Dog('Buddy', 'Golden Retriever');
console.log(dog.makeSound());           // "Buddy barks: Woof! Woof!"
console.log(dog.describe());            // "Buddy is a Dog"
console.log(dog.fetch());               // "Buddy is fetching the ball"
console.log(dog.getFullDescription());  // "Buddy is a Dog of breed Golden Retriever"

// Inheritance chain
console.log(dog instanceof Dog);      // true
console.log(dog instanceof Animal);   // true
console.log(dog instanceof Object);   // true
```

#### **The `super` Keyword**

```javascript
class Vehicle {
  constructor(brand, model) {
    this.brand = brand;
    this.model = model;
  }
  
  start() {
    return `${this.brand} ${this.model} is starting`;
  }
  
  static info() {
    return 'This is a vehicle class';
  }
}

class Car extends Vehicle {
  constructor(brand, model, doors) {
    // super() calls parent constructor
    super(brand, model);
    this.doors = doors;
  }
  
  start() {
    // super.method() calls parent method
    const parentStart = super.start();
    return `${parentStart} with ${this.doors} doors`;
  }
  
  static info() {
    // super in static method calls parent static method
    return `${super.info()} - Extended to Car`;
  }
}

const car = new Car('Toyota', 'Camry', 4);
console.log(car.start());   // "Toyota Camry is starting with 4 doors"
console.log(Car.info());    // "This is a vehicle class - Extended to Car"
```

#### **Private Fields and Methods (ES2022)**

```javascript
class BankAccount {
  // Private fields (start with #)
  #balance = 0;
  #accountNumber;
  
  constructor(accountNumber, initialBalance = 0) {
    this.#accountNumber = accountNumber;
    this.#balance = initialBalance;
  }
  
  // Private method
  #validateAmount(amount) {
    return amount > 0 && Number.isFinite(amount);
  }
  
  // Public method accessing private members
  deposit(amount) {
    if (this.#validateAmount(amount)) {
      this.#balance += amount;
      return `Deposited $${amount}. New balance: $${this.#balance}`;
    }
    throw new Error('Invalid amount');
  }
  
  withdraw(amount) {
    if (this.#validateAmount(amount) && amount <= this.#balance) {
      this.#balance -= amount;
      return `Withdrew $${amount}. New balance: $${this.#balance}`;
    }
    throw new Error('Invalid amount or insufficient funds');
  }
  
  getBalance() {
    return this.#balance;
  }
}

const account = new BankAccount('ACC123', 1000);
console.log(account.deposit(500));      // "Deposited $500. New balance: $1500"
console.log(account.getBalance());      // 1500

// console.log(account.#balance);       // SyntaxError: Private field
// account.#validateAmount(100);        // SyntaxError: Private method
```

#### **Practical Use Cases**

**1. UI Components:**
```javascript
class Modal {
  constructor(title, content, options = {}) {
    this.title = title;
    this.content = content;
    this.isOpen = false;
    this.options = { dismissible: true, ...options };
  }
  
  open() {
    this.isOpen = true;
    this.render();
    this.attachEvents();
  }
  
  close() {
    this.isOpen = false;
    this.cleanup();
  }
  
  render() {
    console.log(`Rendering modal: ${this.title}`);
  }
  
  attachEvents() {
    if (this.options.dismissible) {
      console.log('Attaching close event handlers');
    }
  }
  
  cleanup() {
    console.log('Cleaning up modal resources');
  }
}

const confirmModal = new Modal('Confirm Action', 'Are you sure?');
confirmModal.open();
```

**2. Data Models:**
```javascript
class Product {
  #price;
  
  constructor(name, price, category) {
    this.name = name;
    this.#price = price;
    this.category = category;
    this.createdAt = new Date();
  }
  
  get price() {
    return this.#price;
  }
  
  set price(value) {
    if (value < 0) throw new Error('Price cannot be negative');
    this.#price = value;
  }
  
  get priceWithTax() {
    return this.#price * 1.1;  // 10% tax
  }
  
  applyDiscount(percentage) {
    this.#price *= (1 - percentage / 100);
  }
  
  toJSON() {
    return {
      name: this.name,
      price: this.#price,
      category: this.category,
      createdAt: this.createdAt
    };
  }
}

const laptop = new Product('Laptop', 1000, 'Electronics');
console.log(laptop.price);           // 1000
console.log(laptop.priceWithTax);    // 1100
laptop.applyDiscount(10);
console.log(laptop.price);           // 900
```

**3. Service Classes:**
```javascript
class ApiService {
  constructor(baseURL) {
    this.baseURL = baseURL;
    this.headers = {
      'Content-Type': 'application/json'
    };
  }
  
  setAuthToken(token) {
    this.headers['Authorization'] = `Bearer ${token}`;
  }
  
  async get(endpoint) {
    const response = await fetch(`${this.baseURL}${endpoint}`, {
      headers: this.headers
    });
    return response.json();
  }
  
  async post(endpoint, data) {
    const response = await fetch(`${this.baseURL}${endpoint}`, {
      method: 'POST',
      headers: this.headers,
      body: JSON.stringify(data)
    });
    return response.json();
  }
}

const api = new ApiService('https://api.example.com');
api.setAuthToken('abc123');
// api.get('/users/1').then(data => console.log(data));
```

#### **Key Takeaways**

- **Syntactic sugar** over prototype-based inheritance
- Provides **cleaner syntax** for constructor functions and inheritance
- **Constructor method** initializes instance properties
- **Instance methods** shared across all instances (on prototype)
- **Static methods/properties** belong to class itself, not instances
- **Getters/setters** provide controlled property access
- **`extends`** keyword enables inheritance from parent class
- **`super`** keyword accesses parent constructor and methods
- **Private fields (#)** enforce true encapsulation (ES2022)
- **Not hoisted** - must be declared before use
- Classes are **first-class citizens** (can be passed, returned, assigned)
- **`new` keyword required** to create instances
- **Method chaining** possible by returning `this`
- Common in **modern frameworks** (React class components, Angular services)
- Prefer **classes for complex objects** with shared behavior

---

### 64. What is the difference between class and function constructor?

**Answer:**

While JavaScript classes and function constructors both create objects and implement inheritance, they differ in syntax, behavior, and features. Classes (ES6+) provide cleaner syntax but are essentially syntactic sugar over function constructors.

#### **Syntax Comparison**

```javascript
// FUNCTION CONSTRUCTOR (ES5)
function Person(name, age) {
  this.name = name;
  this.age = age;
}

// Adding methods to prototype
Person.prototype.greet = function() {
  return `Hello, I'm ${this.name}`;
};

// Static method
Person.createGuest = function() {
  return new Person('Guest', 0);
};

const person1 = new Person('Alice', 30);
console.log(person1.greet());  // "Hello, I'm Alice"

// CLASS (ES6+)
class PersonClass {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
  
  // Method automatically added to prototype
  greet() {
    return `Hello, I'm ${this.name}`;
  }
  
  // Static method
  static createGuest() {
    return new PersonClass('Guest', 0);
  }
}

const person2 = new PersonClass('Bob', 25);
console.log(person2.greet());  // "Hello, I'm Bob"
```

#### **Detailed Comparison Table**

| Feature | Function Constructor | Class |
|---------|---------------------|-------|
| **Syntax** | Function-based | Class keyword |
| **Hoisting** | Hoisted (can use before declaration) | Not hoisted (TDZ applies) |
| **Strict mode** | Non-strict by default | Always strict mode |
| **Methods** | Added to prototype manually | Defined inside class body |
| **`new` required** | Can be called without `new` | Must use `new` or throws error |
| **Inheritance** | Manual prototype manipulation | `extends` keyword |
| **Super** | Manual parent call | `super()` keyword |
| **Static methods** | Properties on function | `static` keyword |
| **Getters/Setters** | `Object.defineProperty` | Built-in syntax |
| **Private fields** | Closures or conventions | Native `#` syntax (ES2022) |
| **`typeof`** | 'function' | 'function' (classes are functions) |
| **Enumerable** | Methods enumerable by default | Methods non-enumerable |
| **Constructor property** | Yes | Yes |

#### **Hoisting Behavior**

```javascript
// FUNCTION CONSTRUCTOR - Hoisted
const p1 = new PersonFunc('Alice', 30);  // Works!

function PersonFunc(name, age) {
  this.name = name;
  this.age = age;
}

// CLASS - Not Hoisted (Temporal Dead Zone)
// const p2 = new PersonClass('Bob', 25);  // ReferenceError!

class PersonClass {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
}

const p2 = new PersonClass('Bob', 25);  // Must declare first
```

#### **Calling Without `new`**

```javascript
// FUNCTION CONSTRUCTOR - Can be called without 'new' (but shouldn't)
function Animal(name) {
  this.name = name;
}

const animal1 = Animal('Dog');  // No error, but 'this' is global/undefined
console.log(animal1);           // undefined
console.log(window.name);       // 'Dog' (in non-strict mode)

// Better: Check for 'new' usage
function SafeAnimal(name) {
  if (!(this instanceof SafeAnimal)) {
    return new SafeAnimal(name);
  }
  this.name = name;
}

// CLASS - Must use 'new'
class AnimalClass {
  constructor(name) {
    this.name = name;
  }
}

// const animal2 = AnimalClass('Cat');  // TypeError: Class constructor cannot be invoked without 'new'
const animal2 = new AnimalClass('Cat');  // Correct
```

#### **Inheritance Comparison**

```javascript
// FUNCTION CONSTRUCTOR INHERITANCE
function Vehicle(brand) {
  this.brand = brand;
}

Vehicle.prototype.start = function() {
  return `${this.brand} is starting`;
};

function Car(brand, model) {
  Vehicle.call(this, brand);  // Call parent constructor
  this.model = model;
}

// Set up inheritance chain manually
Car.prototype = Object.create(Vehicle.prototype);
Car.prototype.constructor = Car;

// Override method
Car.prototype.start = function() {
  return `${this.brand} ${this.model} is starting`;
};

const car1 = new Car('Toyota', 'Camry');
console.log(car1.start());  // "Toyota Camry is starting"

// CLASS INHERITANCE - Much cleaner
class VehicleClass {
  constructor(brand) {
    this.brand = brand;
  }
  
  start() {
    return `${this.brand} is starting`;
  }
}

class CarClass extends VehicleClass {
  constructor(brand, model) {
    super(brand);  // Call parent constructor
    this.model = model;
  }
  
  start() {
    return `${this.brand} ${this.model} is starting`;
  }
}

const car2 = new CarClass('Honda', 'Accord');
console.log(car2.start());  // "Honda Accord is starting"
```

#### **Accessing Parent Methods**

```javascript
// FUNCTION CONSTRUCTOR
function ParentFunc() {}
ParentFunc.prototype.show = function() {
  return 'Parent method';
};

function ChildFunc() {}
ChildFunc.prototype = Object.create(ParentFunc.prototype);
ChildFunc.prototype.constructor = ChildFunc;

ChildFunc.prototype.show = function() {
  // Call parent method - verbose
  const parentResult = ParentFunc.prototype.show.call(this);
  return parentResult + ' + Child method';
};

// CLASS - Super keyword
class ParentClass {
  show() {
    return 'Parent method';
  }
}

class ChildClass extends ParentClass {
  show() {
    // Call parent method - clean
    return super.show() + ' + Child method';
  }
}

const child = new ChildClass();
console.log(child.show());  // "Parent method + Child method"
```

#### **Getters and Setters**

```javascript
// FUNCTION CONSTRUCTOR - Verbose
function Circle(radius) {
  let _radius = radius;  // Private via closure
  
  Object.defineProperty(this, 'radius', {
    get: function() {
      return _radius;
    },
    set: function(value) {
      if (value > 0) {
        _radius = value;
      }
    }
  });
  
  Object.defineProperty(this, 'area', {
    get: function() {
      return Math.PI * _radius * _radius;
    }
  });
}

const circle1 = new Circle(5);
console.log(circle1.area);  // 78.54

// CLASS - Clean syntax
class CircleClass {
  constructor(radius) {
    this._radius = radius;
  }
  
  get radius() {
    return this._radius;
  }
  
  set radius(value) {
    if (value > 0) {
      this._radius = value;
    }
  }
  
  get area() {
    return Math.PI * this._radius * this._radius;
  }
}

const circle2 = new CircleClass(5);
console.log(circle2.area);  // 78.54
```

#### **Static Methods**

```javascript
// FUNCTION CONSTRUCTOR
function MathUtil() {}

MathUtil.add = function(a, b) {
  return a + b;
};

MathUtil.PI = 3.14159;

console.log(MathUtil.add(5, 3));  // 8
console.log(MathUtil.PI);         // 3.14159

// CLASS
class MathUtilClass {
  static PI = 3.14159;
  
  static add(a, b) {
    return a + b;
  }
}

console.log(MathUtilClass.add(5, 3));  // 8
console.log(MathUtilClass.PI);         // 3.14159
```

#### **Private Members**

```javascript
// FUNCTION CONSTRUCTOR - Closures for privacy
function BankAccount(initialBalance) {
  // Private variable via closure
  let balance = initialBalance;
  
  // Public methods with closure access
  this.deposit = function(amount) {
    balance += amount;
    return balance;
  };
  
  this.getBalance = function() {
    return balance;
  };
}

const account1 = new BankAccount(1000);
console.log(account1.getBalance());  // 1000
account1.deposit(500);
console.log(account1.getBalance());  // 1500
// console.log(account1.balance);    // undefined (private)

// CLASS - Native private fields (ES2022)
class BankAccountClass {
  #balance;  // Private field
  
  constructor(initialBalance) {
    this.#balance = initialBalance;
  }
  
  deposit(amount) {
    this.#balance += amount;
    return this.#balance;
  }
  
  getBalance() {
    return this.#balance;
  }
}

const account2 = new BankAccountClass(1000);
console.log(account2.getBalance());  // 1000
account2.deposit(500);
console.log(account2.getBalance());  // 1500
// console.log(account2.#balance);   // SyntaxError (truly private)
```

#### **Method Enumerability**

```javascript
// FUNCTION CONSTRUCTOR - Methods enumerable
function UserFunc(name) {
  this.name = name;
}

UserFunc.prototype.greet = function() {
  return 'Hello';
};

const user1 = new UserFunc('Alice');

for (let key in user1) {
  console.log(key);  // 'name', 'greet' (method is enumerable)
}

// CLASS - Methods non-enumerable
class UserClass {
  constructor(name) {
    this.name = name;
  }
  
  greet() {
    return 'Hello';
  }
}

const user2 = new UserClass('Bob');

for (let key in user2) {
  console.log(key);  // Only 'name' (greet is non-enumerable)
}

console.log(Object.keys(user2));  // ['name']
```

#### **Performance Considerations**

```javascript
// Both have similar performance characteristics
// Methods on prototype (shared) vs instance methods (per object)

// FUNCTION CONSTRUCTOR - Prototype methods (good)
function PersonA(name) {
  this.name = name;
}
PersonA.prototype.greet = function() {  // Shared across instances
  return `Hello ${this.name}`;
};

// FUNCTION CONSTRUCTOR - Instance methods (memory inefficient)
function PersonB(name) {
  this.name = name;
  this.greet = function() {  // New function for each instance!
    return `Hello ${this.name}`;
  };
}

// CLASS - Methods on prototype (good, automatic)
class PersonC {
  constructor(name) {
    this.name = name;
  }
  greet() {  // Automatically on prototype (shared)
    return `Hello ${this.name}`;
  }
}

// Creating 1000 instances
const persons = [];
for (let i = 0; i < 1000; i++) {
  persons.push(new PersonC(`Person${i}`));
}
// All 1000 instances share the same greet method (efficient)
```

#### **When to Use Each**

**Use Function Constructors when:**
- Working with legacy ES5 codebases
- Need maximum browser compatibility (pre-ES6)
- Explicitly need hoisting behavior
- Building simple objects without inheritance

**Use Classes when:**
- Building modern applications (ES6+ environment)
- Need inheritance hierarchies
- Want cleaner, more readable syntax
- Using frameworks that prefer classes (React class components, Angular)
- Need private fields (ES2022+)
- Working with TypeScript (better type support for classes)

```javascript
// Modern recommendation: Use classes
class ModernAPI {
  #apiKey;
  
  constructor(apiKey) {
    this.#apiKey = apiKey;
  }
  
  async fetchData(endpoint) {
    const response = await fetch(endpoint, {
      headers: { 'Authorization': `Bearer ${this.#apiKey}` }
    });
    return response.json();
  }
}

// Legacy style: Function constructor
function LegacyAPI(apiKey) {
  let _apiKey = apiKey;  // Closure for privacy
  
  this.fetchData = function(endpoint) {
    return fetch(endpoint, {
      headers: { 'Authorization': 'Bearer ' + _apiKey }
    }).then(function(response) {
      return response.json();
    });
  };
}
```

#### **Key Takeaways**

- **Classes are syntactic sugar** over function constructors
- **Classes provide cleaner syntax** especially for inheritance
- Function constructors are **hoisted**, classes are **not**
- Classes **enforce `new`** keyword, function constructors don't
- Class methods are **non-enumerable** by default
- Classes enable **native private fields** (`#field`)
- **`super` keyword** in classes simplifies parent access
- Both create objects with **same prototype chain**
- Classes are **always in strict mode**
- **Modern codebases prefer classes** for readability and features
- Function constructors still relevant for **legacy support**
- Understanding both helps with **legacy code maintenance**
- Classes work better with **TypeScript and modern tooling**
- Performance is **essentially the same** for both approaches

63. What are classes in JavaScript?
64. What is the difference between class and function constructor?

---

### 65. What are modules in JavaScript?

**Answer:**

JavaScript modules are reusable pieces of code that can be exported from one file and imported into another. Introduced in ES6 (ES2015), modules provide a way to organize, encapsulate, and share code across different files, promoting better code organization, maintainability, and avoiding global namespace pollution.

#### **Module Basics**

```javascript
// FILE: math.js (Module exporting functionality)
export function add(a, b) {
  return a + b;
}

export function subtract(a, b) {
  return a - b;
}

export const PI = 3.14159;

// FILE: app.js (Module importing functionality)
import { add, subtract, PI } from './math.js';

console.log(add(5, 3));        // 8
console.log(subtract(10, 4));  // 6
console.log(PI);               // 3.14159
```

#### **Export Syntax**

**1. Named Exports:**
```javascript
// math-utils.js

// Export during declaration
export function multiply(a, b) {
  return a * b;
}

export const MAX_VALUE = 1000;

export class Calculator {
  add(a, b) {
    return a + b;
  }
}

// Export separately
function divide(a, b) {
  if (b === 0) throw new Error('Division by zero');
  return a / b;
}

const MIN_VALUE = 0;

export { divide, MIN_VALUE };

// Export with rename
function internalHelper() {
  return 'helper';
}

export { internalHelper as helper };
```

**2. Default Export:**
```javascript
// user.js

// Default export (one per module)
export default class User {
  constructor(name, email) {
    this.name = name;
    this.email = email;
  }
  
  greet() {
    return `Hello, ${this.name}`;
  }
}

// Alternative syntax
class Product {
  constructor(name, price) {
    this.name = name;
    this.price = price;
  }
}

export default Product;

// Default export with function
export default function authenticate(username, password) {
  // Authentication logic
  return true;
}

// Default export with value
export default 42;
```

**3. Mixed Exports:**
```javascript
// api.js

// Default export
export default class ApiClient {
  constructor(baseUrl) {
    this.baseUrl = baseUrl;
  }
  
  async fetch(endpoint) {
    return fetch(`${this.baseUrl}${endpoint}`);
  }
}

// Named exports alongside default
export const API_VERSION = 'v1';
export const TIMEOUT = 5000;

export function formatUrl(endpoint) {
  return `/api/${API_VERSION}${endpoint}`;
}
```

#### **Import Syntax**

**1. Named Imports:**
```javascript
// Import specific exports
import { add, subtract } from './math.js';

// Import with rename (aliasing)
import { add as sum, subtract as diff } from './math.js';

console.log(sum(5, 3));   // 8
console.log(diff(10, 4)); // 6

// Import all named exports as namespace
import * as MathUtils from './math.js';

console.log(MathUtils.add(5, 3));      // 8
console.log(MathUtils.PI);             // 3.14159
console.log(MathUtils.subtract(10, 4)); // 6

// Import only for side effects (no bindings)
import './init-polyfills.js';
```

**2. Default Imports:**
```javascript
// Import default export (can use any name)
import User from './user.js';
import MyUser from './user.js';  // Same thing, different name

const user = new User('Alice', 'alice@example.com');
console.log(user.greet());  // "Hello, Alice"

// Import default with named exports
import ApiClient, { API_VERSION, TIMEOUT } from './api.js';

const client = new ApiClient('https://api.example.com');
console.log(API_VERSION);  // 'v1'
```

**3. Dynamic Imports:**
```javascript
// Dynamic import returns a Promise
async function loadModule() {
  const module = await import('./heavy-module.js');
  module.default();  // Call default export
  module.namedFunction();  // Call named export
}

// Conditional loading
if (userWantsFeature) {
  import('./optional-feature.js')
    .then(module => {
      module.initialize();
    })
    .catch(err => {
      console.error('Failed to load module:', err);
    });
}

// Load based on user action
button.addEventListener('click', async () => {
  const { showModal } = await import('./modal.js');
  showModal('Hello World');
});

// Dynamic import with destructuring
const { add, multiply } = await import('./math.js');
console.log(add(5, 3));  // 8
```

**4. Re-exporting:**
```javascript
// utils/index.js - Barrel file

// Re-export everything from another module
export * from './string-utils.js';
export * from './array-utils.js';

// Re-export with rename
export { default as DateFormatter } from './date-formatter.js';

// Re-export specific items
export { validateEmail, validatePhone } from './validators.js';

// Re-export default as named
export { default as Logger } from './logger.js';

// Usage
import { formatString, sortArray, Logger } from './utils/index.js';
```

#### **Module Features and Characteristics**

```javascript
// 1. Modules are automatically in strict mode
// No need for 'use strict'

// 2. Modules have their own scope
let privateVariable = 'private';  // Not accessible outside module

export function getPrivate() {
  return privateVariable;  // Only accessible through exported function
}

// 3. Modules are singletons (executed once)
// FILE: counter.js
let count = 0;

export function increment() {
  count++;
}

export function getCount() {
  return count;
}

// FILE: app.js
import { increment, getCount } from './counter.js';
import { increment as inc2 } from './counter.js';

increment();   // count = 1
inc2();        // count = 2 (same module instance)
console.log(getCount());  // 2

// 4. Imports are hoisted
console.log(add(5, 3));  // Works! Import is hoisted
import { add } from './math.js';

// 5. Imports are read-only (live bindings)
// FILE: module.js
export let counter = 0;

export function increment() {
  counter++;
}

// FILE: app.js
import { counter, increment } from './module.js';

console.log(counter);  // 0
increment();
console.log(counter);  // 1 (live binding updated!)

// counter = 5;  // TypeError: Assignment to constant variable
```

#### **Module Patterns**

**1. Configuration Module:**
```javascript
// config.js
const config = {
  api: {
    baseUrl: process.env.API_URL || 'https://api.example.com',
    timeout: 5000,
    retries: 3
  },
  features: {
    darkMode: true,
    analytics: true
  }
};

export default config;

// Optional: Named exports for specific sections
export const apiConfig = config.api;
export const featureFlags = config.features;
```

**2. Service Module:**
```javascript
// user-service.js
class UserService {
  constructor() {
    this.users = [];
  }
  
  async fetchUsers() {
    const response = await fetch('/api/users');
    this.users = await response.json();
    return this.users;
  }
  
  getUserById(id) {
    return this.users.find(user => user.id === id);
  }
}

// Export singleton instance
export default new UserService();

// Usage
import userService from './user-service.js';
await userService.fetchUsers();
```

**3. Utility Module:**
```javascript
// string-utils.js
export function capitalize(str) {
  return str.charAt(0).toUpperCase() + str.slice(1);
}

export function truncate(str, length) {
  return str.length > length ? str.slice(0, length) + '...' : str;
}

export function slugify(str) {
  return str.toLowerCase().replace(/\s+/g, '-');
}

// Usage
import { capitalize, slugify } from './string-utils.js';
```

**4. Factory Module:**
```javascript
// logger-factory.js
export function createLogger(namespace) {
  return {
    log: (message) => console.log(`[${namespace}] ${message}`),
    error: (message) => console.error(`[${namespace}] ERROR: ${message}`),
    warn: (message) => console.warn(`[${namespace}] WARN: ${message}`)
  };
}

// Usage
import { createLogger } from './logger-factory.js';

const authLogger = createLogger('AUTH');
authLogger.log('User logged in');
```

#### **Using Modules in HTML**

```html
<!DOCTYPE html>
<html>
<head>
  <title>ES6 Modules</title>
</head>
<body>
  <!-- Type="module" enables ES6 module syntax -->
  <script type="module">
    import { add, multiply } from './math.js';
    
    console.log(add(5, 3));       // 8
    console.log(multiply(4, 7));  // 28
  </script>
  
  <!-- External module file -->
  <script type="module" src="./app.js"></script>
  
  <!-- Modules are deferred by default -->
  <!-- No need for defer attribute -->
</body>
</html>
```

#### **Module Loading Options**

```javascript
// 1. Preloading modules
// HTML: <link rel="modulepreload" href="./critical-module.js">

// 2. Lazy loading for performance
async function loadFeature() {
  if (!featureLoaded) {
    const module = await import('./feature.js');
    module.init();
    featureLoaded = true;
  }
}

// 3. Conditional loading based on environment
const logger = 
  process.env.NODE_ENV === 'production'
    ? await import('./prod-logger.js')
    : await import('./dev-logger.js');

// 4. Loading with fallback
let mathLib;
try {
  mathLib = await import('./advanced-math.js');
} catch (error) {
  mathLib = await import('./basic-math.js');
}
```

#### **Module vs Script Differences**

| Feature | Script | Module |
|---------|--------|--------|
| **Scope** | Global | Module (own scope) |
| **Strict mode** | Optional | Always enabled |
| **Top-level `this`** | `window` | `undefined` |
| **Import/Export** | Not supported | Supported |
| **Loading** | Synchronous | Asynchronous |
| **Execution** | Immediate | Deferred |
| **Multiple loads** | Multiple executions | Single execution (singleton) |
| **CORS** | No restriction | Must respect CORS |
| **File extension** | Any | Usually `.js` or `.mjs` |

#### **CommonJS vs ES Modules**

```javascript
// COMMONJS (Node.js traditional)
// export
module.exports = {
  add: function(a, b) { return a + b; },
  PI: 3.14159
};

// import
const math = require('./math');
console.log(math.add(5, 3));

// ES MODULES (Standard JavaScript)
// export
export function add(a, b) {
  return a + b;
}
export const PI = 3.14159;

// import
import { add, PI } from './math.js';
console.log(add(5, 3));

// Node.js supports both:
// - Use .mjs extension for ES modules
// - Or set "type": "module" in package.json
```

#### **Practical Use Cases**

**1. Code Organization:**
```javascript
// Project structure
// src/
//   components/
//     Button.js
//     Modal.js
//   services/
//     api.js
//     auth.js
//   utils/
//     validators.js
//     formatters.js
//   app.js

// app.js
import { Button, Modal } from './components/index.js';
import { apiService } from './services/api.js';
import { validateEmail } from './utils/validators.js';
```

**2. Tree Shaking (Dead Code Elimination):**
```javascript
// utils.js - Large utility library
export function usedFunction() {
  return 'I am used';
}

export function unusedFunction() {
  return 'I am never imported';
}

// app.js - Only import what you need
import { usedFunction } from './utils.js';

// Build tools (webpack, rollup) will remove unusedFunction
// from the final bundle (tree shaking)
```

**3. Dependency Management:**
```javascript
// Clear dependency declaration
import React from 'react';
import { BrowserRouter, Route } from 'react-router-dom';
import axios from 'axios';

// vs older approach with global variables
// <script src="react.js"></script>
// <script src="react-router.js"></script>
// <script src="axios.js"></script>
```

#### **Best Practices**

```javascript
// 1. One module = One responsibility
// ❌ Bad: Everything in one file
// utils.js with string, array, date, math utilities

// ✅ Good: Separate concerns
// string-utils.js
// array-utils.js
// date-utils.js

// 2. Use index.js for barrel exports
// utils/index.js
export * from './string-utils.js';
export * from './array-utils.js';

// Usage
import { capitalize, sortArray } from './utils/index.js';

// 3. Prefer named exports for better refactoring
export function fetchUser() { }  // Easy to find/rename
// vs
export default function() { }    // Can be imported with any name

// 4. Keep modules focused and small
// ✅ Good: Small, focused modules
export function validateEmail(email) {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
}

// 5. Use absolute imports (with build tools)
import Button from '@/components/Button';  // Clean
// vs
import Button from '../../../components/Button';  // Fragile
```

#### **Common Pitfalls**

```javascript
// 1. Circular dependencies
// a.js
import { b } from './b.js';
export const a = 'a' + b;

// b.js
import { a } from './a.js';
export const b = 'b' + a;  // Can cause issues!

// 2. Missing file extensions in browser
import { add } from './math';  // ❌ May not work in browser
import { add } from './math.js';  // ✅ Always include .js

// 3. Mixing default and named exports inconsistently
// Confusing:
export default User;
export { User };  // Different things!

// 4. Modifying imported values
import { config } from './config.js';
config.apiUrl = 'new-url';  // ❌ Don't mutate imported objects
```

#### **Key Takeaways**

- **Modules encapsulate code** in their own scope, avoiding global pollution
- **Export/import syntax** enables code sharing between files
- **Named exports** allow multiple exports per module
- **Default exports** provide a single main export per module
- **Dynamic imports** enable code splitting and lazy loading
- **Modules are singletons** - executed once and cached
- **Imports are hoisted** and create live bindings
- **Always in strict mode** - no need for 'use strict'
- **Deferred execution** in browsers (like defer attribute)
- **Tree shaking** removes unused exports from bundles
- **Better than scripts** for organizing large applications
- **Use `.mjs`** extension or `"type": "module"` in Node.js
- **CORS applies** to module loading in browsers
- Standard way to organize **modern JavaScript** applications

---

### 66. What is the difference between named export and default export?

**Answer:**

Named exports and default exports are two ways to export values from a JavaScript module. Named exports allow multiple exports per module with specific names, while default exports provide a single main export per module that can be imported with any name.

#### **Named Exports**

```javascript
// FILE: math.js

// Multiple named exports in one module
export function add(a, b) {
  return a + b;
}

export function subtract(a, b) {
  return a - b;
}

export const PI = 3.14159;

export class Calculator {
  multiply(a, b) {
    return a * b;
  }
}

// Alternative: Export at the end
function divide(a, b) {
  return a / b;
}

const E = 2.71828;

export { divide, E };
```

**Importing Named Exports:**
```javascript
// Must use exact names with curly braces
import { add, subtract, PI } from './math.js';

console.log(add(5, 3));    // 8
console.log(PI);           // 3.14159

// Import with aliasing (rename)
import { add as sum, subtract as diff } from './math.js';

console.log(sum(5, 3));    // 8
console.log(diff(10, 4));  // 6

// Import all named exports as namespace
import * as Math from './math.js';

console.log(Math.add(5, 3));       // 8
console.log(Math.PI);              // 3.14159
console.log(Math.Calculator);      // [class Calculator]

// Import specific items only
import { PI } from './math.js';  // Only PI, ignore others
```

#### **Default Exports**

```javascript
// FILE: user.js

// Only ONE default export per module
export default class User {
  constructor(name, email) {
    this.name = name;
    this.email = email;
  }
  
  greet() {
    return `Hello, ${this.name}`;
  }
}

// Alternative syntaxes for default export
class Product {
  constructor(name) {
    this.name = name;
  }
}
export default Product;

// Default export with function
export default function authenticate(username, password) {
  return true;
}

// Default export with anonymous function
export default function(data) {
  return data.map(item => item.id);
}

// Default export with value
export default 42;

// Default export with object
export default {
  version: '1.0.0',
  author: 'John Doe'
};
```

**Importing Default Exports:**
```javascript
// Can use ANY name (no curly braces)
import User from './user.js';
import MyUser from './user.js';      // Same import, different name
import UserClass from './user.js';   // Still works

const user1 = new User('Alice', 'alice@example.com');
const user2 = new MyUser('Bob', 'bob@example.com');

// Both create instances of the same class
console.log(user1.greet());  // "Hello, Alice"
console.log(user2.greet());  // "Hello, Bob"
```

#### **Combining Named and Default Exports**

```javascript
// FILE: api.js

// Default export
export default class ApiClient {
  constructor(baseUrl) {
    this.baseUrl = baseUrl;
  }
  
  async fetch(endpoint) {
    return fetch(`${this.baseUrl}${endpoint}`);
  }
}

// Named exports alongside default
export const API_VERSION = 'v1';
export const TIMEOUT = 5000;
export const MAX_RETRIES = 3;

export function formatEndpoint(path) {
  return `/api/${API_VERSION}${path}`;
}

export class ApiError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
  }
}
```

**Importing Mixed Exports:**
```javascript
// Import default and named together
import ApiClient, { API_VERSION, TIMEOUT, formatEndpoint } from './api.js';

// Use both
const client = new ApiClient('https://api.example.com');
console.log(API_VERSION);  // 'v1'
console.log(formatEndpoint('/users'));  // '/api/v1/users'

// Alternative: Separate imports
import ApiClient from './api.js';
import { API_VERSION, TIMEOUT } from './api.js';

// Import default with alias
import { default as Client, API_VERSION } from './api.js';
```

#### **Detailed Comparison**

| Feature | Named Export | Default Export |
|---------|-------------|----------------|
| **Quantity** | Multiple per module | One per module |
| **Syntax** | `export function foo()` | `export default function foo()` |
| **Import syntax** | `import { foo } from './module'` | `import foo from './module'` |
| **Import name** | Must match export name | Any name allowed |
| **Curly braces** | Required in import | Not used |
| **Renaming** | `import { foo as bar }` | Import with any name directly |
| **Namespace import** | `import * as NS` | Not directly (use `NS.default`) |
| **Anonymous exports** | Not allowed | Allowed |
| **Tree shaking** | Better (unused exports removed) | All or nothing |
| **Refactoring** | Easier (name changes detected) | Harder (any name valid) |
| **Discoverability** | Better (IDE autocomplete) | Depends on documentation |
| **Use case** | Utility libraries, multiple functions | Main class, single purpose |

#### **Export Syntax Variations**

```javascript
// NAMED EXPORTS

// 1. Inline exports
export const VERSION = '1.0.0';
export function helper() { }
export class MyClass { }

// 2. Separate export statement
const A = 1;
const B = 2;
function foo() { }

export { A, B, foo };

// 3. Export with rename
const internalName = 'value';
export { internalName as publicName };

// 4. Re-exporting from another module
export { foo, bar } from './other-module.js';
export * from './another-module.js';

// DEFAULT EXPORTS

// 1. Inline default export (anonymous)
export default function() { }
export default class { }

// 2. Inline default export (named)
export default function myFunc() { }
export default class MyClass { }

// 3. Separate default export
function myFunction() { }
export default myFunction;

// 4. Default export with object literal
export default {
  prop1: 'value',
  method() { }
};
```

#### **Practical Examples**

**1. Utility Library (Prefer Named Exports):**
```javascript
// string-utils.js
export function capitalize(str) {
  return str.charAt(0).toUpperCase() + str.slice(1);
}

export function truncate(str, length) {
  return str.length > length ? str.slice(0, length) + '...' : str;
}

export function slugify(str) {
  return str.toLowerCase().replace(/\s+/g, '-');
}

// Import only what you need
import { capitalize, slugify } from './string-utils.js';

// Tree shaking removes unused 'truncate' from bundle
```

**2. Main Class (Prefer Default Export):**
```javascript
// database.js
export default class Database {
  constructor(config) {
    this.config = config;
  }
  
  async connect() {
    // Connection logic
  }
  
  async query(sql) {
    // Query logic
  }
}

// Clear, single-purpose module
import Database from './database.js';

const db = new Database({ host: 'localhost' });
```

**3. Configuration (Named Exports Recommended):**
```javascript
// config.js
export const API_URL = 'https://api.example.com';
export const API_KEY = process.env.API_KEY;
export const TIMEOUT = 5000;
export const MAX_RETRIES = 3;

export const featureFlags = {
  darkMode: true,
  analytics: false
};

// Import specific configs
import { API_URL, TIMEOUT } from './config.js';
```

**4. React Components (Varies by Convention):**
```javascript
// Button.js - Default export (common in React)
export default function Button({ label, onClick }) {
  return <button onClick={onClick}>{label}</button>;
}

// Or with named export
export function Button({ label, onClick }) {
  return <button onClick={onClick}>{label}</button>;
}

// Mixed approach for components with helpers
export default function Modal({ children }) {
  return <div className="modal">{children}</div>;
}

export function ModalHeader({ title }) {
  return <div className="modal-header">{title}</div>;
}

export function ModalBody({ children }) {
  return <div className="modal-body">{children}</div>;
}

// Usage
import Modal, { ModalHeader, ModalBody } from './Modal';
```

#### **Common Patterns**

**1. Default Export with Named Constants:**
```javascript
// logger.js
class Logger {
  log(message) {
    console.log(`[LOG] ${message}`);
  }
  
  error(message) {
    console.error(`[ERROR] ${message}`);
  }
}

export default new Logger();  // Singleton instance

export const LOG_LEVELS = {
  DEBUG: 0,
  INFO: 1,
  WARN: 2,
  ERROR: 3
};

// Usage
import logger, { LOG_LEVELS } from './logger.js';
logger.log('Application started');
```

**2. Barrel Exports (Index Files):**
```javascript
// components/index.js
export { default as Button } from './Button.js';
export { default as Modal } from './Modal.js';
export { default as Input } from './Input.js';

// Or with named exports
export { Button } from './Button.js';
export { Modal } from './Modal.js';
export { Input } from './Input.js';

// Usage - Clean imports
import { Button, Modal, Input } from './components';
```

**3. Factory Pattern with Named Exports:**
```javascript
// factories.js
export function createUser(name, role) {
  return {
    name,
    role,
    createdAt: new Date()
  };
}

export function createAdmin(name) {
  return createUser(name, 'admin');
}

export function createGuest() {
  return createUser('Guest', 'guest');
}

// Usage
import { createAdmin, createGuest } from './factories.js';
```

#### **When to Use Each**

**Use Named Exports when:**
- Module exports multiple values
- Building utility libraries
- Want better tree shaking
- Need explicit names for refactoring
- Exporting configuration constants
- Creating barrel exports (index files)

```javascript
// ✅ Good use of named exports
// utils.js
export function formatDate(date) { }
export function parseDate(str) { }
export function isValidDate(date) { }
```

**Use Default Exports when:**
- Module has a single main purpose
- Exporting a class as the primary export
- One main function/component per file
- Following framework conventions (React components)
- Creating singleton instances

```javascript
// ✅ Good use of default export
// UserService.js
export default class UserService {
  async fetchUsers() { }
  async createUser(data) { }
}
```

**Avoid mixing unnecessarily:**
```javascript
// ❌ Confusing: Too many exports with unclear purpose
export default function main() { }
export function helper1() { }
export function helper2() { }
export const config = { };

// ✅ Better: Clear separation
// main.js (default)
export default function main() { }

// helpers.js (named)
export function helper1() { }
export function helper2() { }

// config.js (named)
export const config = { };
```

#### **Common Pitfalls**

```javascript
// 1. Can't have multiple default exports
export default class User { }
// export default class Admin { }  // ❌ SyntaxError

// 2. Default export can't be directly destructured in export
const config = { api: 'url' };
// export default { config };  // Works but not ideal
export { config as default };  // ✅ Explicit

// 3. Confusing when importing default with named
import User from './user.js';  // Default
import { User } from './user.js';  // Named - different import!

// 4. Namespace import includes default as 'default'
import * as Utils from './utils.js';
Utils.default();  // Access default export
Utils.namedExport();  // Access named export

// 5. Can't use destructuring directly with default
// import { greet } from './user.js';  // Only for named exports
import User from './user.js';
const { greet } = new User();  // Destructure after import
```

#### **Migration Between Export Types**

```javascript
// Converting named to default
// BEFORE (named)
export function authenticate() { }

import { authenticate } from './auth.js';

// AFTER (default)
export default function authenticate() { }

import authenticate from './auth.js';

// Converting default to named
// BEFORE (default)
export default class User { }

import User from './user.js';

// AFTER (named)
export class User { }

import { User } from './user.js';
```

#### **Key Takeaways**

- **Named exports** allow multiple exports with specific names
- **Default exports** provide one main export per module
- **Named** requires `{ }` and matching names in imports
- **Default** can be imported with any name without `{ }`
- Can **combine both** in a single module
- **Named exports** better for tree shaking and refactoring
- **Default exports** better for single-purpose modules
- **Named preferred** for utility libraries and configurations
- **Default preferred** for main classes and React components
- **Import syntax** differs significantly between the two
- **Named exports** improve IDE autocomplete and discoverability
- Choose based on **module purpose** and team conventions
- **Consistency** within a project is important
- Both are **statically analyzable** (no runtime evaluation)
- Modern best practice: **prefer named exports** when possible

---

### 67. What are symbols in JavaScript?

**Answer:**

Symbols are a primitive data type introduced in ES6 that represent unique, immutable identifiers. Every symbol value is unique, even if they have the same description. Symbols are primarily used as property keys for objects to avoid name collisions and to create non-enumerable properties.

#### **Creating Symbols**

```javascript
// Creating symbols
const sym1 = Symbol();
const sym2 = Symbol();

console.log(sym1 === sym2);  // false (each symbol is unique)

// Symbols with descriptions (for debugging)
const sym3 = Symbol('mySymbol');
const sym4 = Symbol('mySymbol');

console.log(sym3 === sym4);  // false (still unique, despite same description)
console.log(sym3.toString());  // "Symbol(mySymbol)"
console.log(sym3.description);  // "mySymbol"

// Symbols are primitives, not objects
console.log(typeof Symbol());  // "symbol"

// Cannot use 'new' with Symbol
// const sym = new Symbol();  // TypeError: Symbol is not a constructor
```

#### **Using Symbols as Object Keys**

```javascript
// Symbols as property keys
const id = Symbol('id');
const user = {
  name: 'Alice',
  [id]: 12345  // Symbol property using computed property name
};

console.log(user.name);  // 'Alice'
console.log(user[id]);   // 12345
console.log(user.id);    // undefined (not a string property)

// Adding symbol properties after creation
const email = Symbol('email');
user[email] = 'alice@example.com';

console.log(user[email]);  // 'alice@example.com'

// Symbol properties don't show up in normal iteration
for (let key in user) {
  console.log(key);  // Only logs 'name' (symbol properties skipped)
}

console.log(Object.keys(user));  // ['name']
console.log(Object.getOwnPropertyNames(user));  // ['name']

// To access symbol properties
console.log(Object.getOwnPropertySymbols(user));  // [Symbol(id), Symbol(email)]
console.log(Reflect.ownKeys(user));  // ['name', Symbol(id), Symbol(email)]
```

#### **Symbol Registry (Global Symbols)**

```javascript
// Symbol.for() creates/retrieves global symbols
const globalSym1 = Symbol.for('app.id');
const globalSym2 = Symbol.for('app.id');

console.log(globalSym1 === globalSym2);  // true (same global symbol)

// Regular symbols vs global symbols
const localSym = Symbol('app.id');
const globalSym = Symbol.for('app.id');

console.log(localSym === globalSym);  // false (different types)

// Symbol.keyFor() retrieves the key for global symbols
console.log(Symbol.keyFor(globalSym1));  // 'app.id'
console.log(Symbol.keyFor(localSym));    // undefined (not global)

// Use case: Cross-realm/cross-file unique identifiers
// FILE: module1.js
export const CACHE_KEY = Symbol.for('app.cache');

// FILE: module2.js
import { CACHE_KEY } from './module1.js';
const key = Symbol.for('app.cache');
console.log(CACHE_KEY === key);  // true
```

#### **Well-Known Symbols**

JavaScript provides built-in symbols that customize language behavior:

```javascript
// 1. Symbol.iterator - Make objects iterable
const collection = {
  items: ['a', 'b', 'c'],
  [Symbol.iterator]() {
    let index = 0;
    return {
      next: () => {
        if (index < this.items.length) {
          return { value: this.items[index++], done: false };
        }
        return { done: true };
      }
    };
  }
};

for (let item of collection) {
  console.log(item);  // 'a', 'b', 'c'
}

// 2. Symbol.toStringTag - Customize Object.prototype.toString()
class CustomClass {
  get [Symbol.toStringTag]() {
    return 'CustomClass';
  }
}

const instance = new CustomClass();
console.log(Object.prototype.toString.call(instance));  // '[object CustomClass]'

// 3. Symbol.hasInstance - Customize instanceof behavior
class MyArray {
  static [Symbol.hasInstance](instance) {
    return Array.isArray(instance);
  }
}

console.log([] instanceof MyArray);  // true
console.log({} instanceof MyArray);  // false

// 4. Symbol.toPrimitive - Control type conversion
const obj = {
  [Symbol.toPrimitive](hint) {
    if (hint === 'number') {
      return 42;
    }
    if (hint === 'string') {
      return 'Hello';
    }
    return true;  // default
  }
};

console.log(+obj);      // 42 (number hint)
console.log(`${obj}`);  // 'Hello' (string hint)
console.log(obj + '');  // 'true' (default hint)

// 5. Symbol.species - Customize derived object creation
class MyArray extends Array {
  static get [Symbol.species]() {
    return Array;  // map(), filter() return Array, not MyArray
  }
}

const myArr = new MyArray(1, 2, 3);
const mapped = myArr.map(x => x * 2);
console.log(mapped instanceof MyArray);  // false
console.log(mapped instanceof Array);    // true
```

#### **Practical Use Cases**

**1. Avoiding Property Name Collisions:**
```javascript
// Library A defines a property
const libraryA = {
  version: '1.0.0',
  _internal: 'Library A data'  // Might conflict with other code
};

// Library B might override it
const libraryB = {
  _internal: 'Library B data'  // Collision!
};

// Using symbols prevents conflicts
const internalA = Symbol('internal');
const internalB = Symbol('internal');

const safeLibraryA = {
  version: '1.0.0',
  [internalA]: 'Library A data'
};

const safeLibraryB = {
  version: '2.0.0',
  [internalB]: 'Library B data'
};

// No collision - different symbols
console.log(safeLibraryA[internalA]);  // 'Library A data'
console.log(safeLibraryB[internalB]);  // 'Library B data'
```

**2. Creating Private-Like Properties:**
```javascript
// Private-like properties using symbols
const _balance = Symbol('balance');
const _validateAmount = Symbol('validateAmount');

class BankAccount {
  constructor(initialBalance) {
    this[_balance] = initialBalance;
  }
  
  [_validateAmount](amount) {
    return amount > 0 && Number.isFinite(amount);
  }
  
  deposit(amount) {
    if (this[_validateAmount](amount)) {
      this[_balance] += amount;
      return this[_balance];
    }
    throw new Error('Invalid amount');
  }
  
  getBalance() {
    return this[_balance];
  }
}

const account = new BankAccount(1000);
console.log(account.getBalance());  // 1000

// Symbol properties not visible in normal access
console.log(account.balance);  // undefined
console.log(Object.keys(account));  // []

// But accessible if symbol is known
console.log(account[_balance]);  // 1000 (not truly private)
```

**3. Metadata and Configuration:**
```javascript
// Adding metadata without affecting normal properties
const CONFIG = Symbol('config');
const METADATA = Symbol('metadata');

class Component {
  constructor(name) {
    this.name = name;
    this[CONFIG] = {
      version: '1.0.0',
      internal: true
    };
    this[METADATA] = {
      createdAt: new Date(),
      author: 'System'
    };
  }
  
  toJSON() {
    // Symbol properties automatically excluded from JSON
    return {
      name: this.name
      // config and metadata not included
    };
  }
}

const comp = new Component('MyComponent');
console.log(JSON.stringify(comp));  // {"name":"MyComponent"}
console.log(comp[CONFIG]);  // { version: '1.0.0', internal: true }
```

**4. Implementing Custom Behaviors:**
```javascript
// Custom iterable range
class Range {
  constructor(start, end) {
    this.start = start;
    this.end = end;
  }
  
  [Symbol.iterator]() {
    let current = this.start;
    const end = this.end;
    
    return {
      next() {
        if (current <= end) {
          return { value: current++, done: false };
        }
        return { done: true };
      }
    };
  }
}

const range = new Range(1, 5);
console.log([...range]);  // [1, 2, 3, 4, 5]

for (let num of range) {
  console.log(num);  // 1, 2, 3, 4, 5
}
```

**5. Extending Built-in Objects:**
```javascript
// Safely extending Array without naming conflicts
const shuffle = Symbol('shuffle');

Array.prototype[shuffle] = function() {
  const arr = [...this];
  for (let i = arr.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [arr[i], arr[j]] = [arr[j], arr[i]];
  }
  return arr;
};

const numbers = [1, 2, 3, 4, 5];
console.log(numbers[shuffle]());  // Randomly shuffled array

// No conflict with other array properties/methods
```

#### **Symbol Properties and JSON**

```javascript
const id = Symbol('id');
const user = {
  name: 'Alice',
  age: 30,
  [id]: 12345
};

// Symbol properties ignored in JSON serialization
console.log(JSON.stringify(user));  // {"name":"Alice","age":30}

// Symbol properties not in Object.assign() by default
const copy = Object.assign({}, user);
console.log(copy[id]);  // undefined

// To copy symbol properties
const fullCopy = Object.assign({}, user, 
  Object.getOwnPropertySymbols(user).reduce((acc, sym) => {
    acc[sym] = user[sym];
    return acc;
  }, {})
);
console.log(fullCopy[id]);  // 12345
```

#### **Well-Known Symbols Reference**

| Symbol | Purpose |
|--------|---------|
| **Symbol.iterator** | Make object iterable (for...of) |
| **Symbol.asyncIterator** | Make object async iterable (for await...of) |
| **Symbol.hasInstance** | Customize `instanceof` behavior |
| **Symbol.toStringTag** | Customize `Object.prototype.toString()` |
| **Symbol.toPrimitive** | Control type coercion |
| **Symbol.species** | Specify constructor for derived objects |
| **Symbol.match** | Used by `String.prototype.match()` |
| **Symbol.replace** | Used by `String.prototype.replace()` |
| **Symbol.search** | Used by `String.prototype.search()` |
| **Symbol.split** | Used by `String.prototype.split()` |
| **Symbol.isConcatSpreadable** | Control array flattening in `concat()` |
| **Symbol.unscopables** | Exclude properties from `with` statement |

#### **Common Patterns**

```javascript
// 1. Constant values (enum-like)
const Status = {
  PENDING: Symbol('pending'),
  ACTIVE: Symbol('active'),
  INACTIVE: Symbol('inactive')
};

function setStatus(status) {
  if (status === Status.PENDING) {
    console.log('Status is pending');
  }
}

setStatus(Status.PENDING);

// 2. Private static properties
class Service {
  static #instances = new Map();
  static instanceKey = Symbol('instance');
  
  static getInstance(id) {
    if (!this.#instances.has(id)) {
      this.#instances.set(id, new Service(id));
    }
    return this.#instances.get(id);
  }
}

// 3. Hooks and lifecycle methods
const onInit = Symbol('onInit');
const onDestroy = Symbol('onDestroy');

class Component {
  constructor() {
    this[onInit]();
  }
  
  [onInit]() {
    console.log('Component initialized');
  }
  
  [onDestroy]() {
    console.log('Component destroyed');
  }
  
  destroy() {
    this[onDestroy]();
  }
}
```

#### **Key Takeaways**

- **Symbols are unique primitives** - each Symbol() creates a new unique value
- **Used as property keys** to avoid name collisions
- **Not enumerable** in for...in, Object.keys(), JSON.stringify()
- **Symbol.for()** creates global symbols shared across realms
- **Symbol.keyFor()** retrieves key for global symbols
- **Well-known symbols** customize built-in JavaScript behavior
- **Cannot use `new`** - Symbol is not a constructor
- **Description is optional** and only for debugging
- **Useful for metadata** that shouldn't appear in normal iteration
- **Not truly private** - accessible via Object.getOwnPropertySymbols()
- **ES6+ feature** - not available in older browsers without polyfill
- **Common use cases**: private-like properties, avoiding collisions, custom iterators, metadata storage
- **Symbol properties** excluded from most serialization/enumeration
- Prefer **true private fields (#)** for actual encapsulation (ES2022+)

65. What are modules in JavaScript?
66. What is the difference between named export and default export?
67. What are symbols in JavaScript?

---

### 68. What is the Set data structure?

**Answer:**

A Set is a built-in JavaScript collection that stores unique values of any type. Unlike arrays, Sets automatically remove duplicate values and provide efficient methods for adding, deleting, and checking for the existence of values. Sets maintain insertion order and are optimized for membership testing.

#### **Creating and Basic Operations**

```javascript
// Creating a Set
const set1 = new Set();
const set2 = new Set([1, 2, 3, 4, 5]);
const set3 = new Set('hello');  // Creates Set with unique characters

console.log(set2);  // Set(5) { 1, 2, 3, 4, 5 }
console.log(set3);  // Set(4) { 'h', 'e', 'l', 'o' } (duplicate 'l' removed)

// Adding values
const mySet = new Set();
mySet.add(1);
mySet.add(2);
mySet.add(3);
mySet.add(2);  // Duplicate - won't be added
console.log(mySet);  // Set(3) { 1, 2, 3 }

// Method chaining
mySet.add(4).add(5).add(6);
console.log(mySet.size);  // 6

// Checking existence
console.log(mySet.has(3));  // true
console.log(mySet.has(10)); // false

// Deleting values
mySet.delete(3);
console.log(mySet.has(3));  // false

// Size property
console.log(mySet.size);  // 5

// Clearing all values
mySet.clear();
console.log(mySet.size);  // 0
```

#### **Uniqueness and Value Equality**

```javascript
// Primitive values - compared by value
const numbers = new Set();
numbers.add(1);
numbers.add(1);
numbers.add('1');
console.log(numbers);  // Set(2) { 1, '1' } (different types)

// NaN is treated as equal to NaN
const set = new Set();
set.add(NaN);
set.add(NaN);
console.log(set.size);  // 1 (only one NaN)

// Objects - compared by reference
const obj1 = { id: 1 };
const obj2 = { id: 1 };
const objSet = new Set();
objSet.add(obj1);
objSet.add(obj2);
objSet.add(obj1);  // Duplicate reference
console.log(objSet.size);  // 2 (obj1 and obj2 are different objects)

// Same reference is duplicate
const user = { name: 'Alice' };
const users = new Set();
users.add(user);
users.add(user);
console.log(users.size);  // 1 (same reference)

// +0 and -0 are considered equal
const zeroSet = new Set();
zeroSet.add(0);
zeroSet.add(-0);
console.log(zeroSet.size);  // 1 (treated as same)
```

#### **Iterating Over Sets**

```javascript
const colors = new Set(['red', 'green', 'blue']);

// 1. for...of loop
for (let color of colors) {
  console.log(color);
}
// Output: 'red', 'green', 'blue'

// 2. forEach method
colors.forEach((value, valueAgain, set) => {
  console.log(value);
  // Note: value and valueAgain are the same (for API consistency with Map)
});

// 3. Iterator methods
console.log(colors.values());  // SetIterator { 'red', 'green', 'blue' }
console.log(colors.keys());    // Same as values() (for Map compatibility)
console.log(colors.entries()); // SetIterator { ['red', 'red'], ['green', 'green'], ['blue', 'blue'] }

// 4. Spread operator
const colorArray = [...colors];
console.log(colorArray);  // ['red', 'green', 'blue']

// 5. Array.from()
const colorArray2 = Array.from(colors);
console.log(colorArray2);  // ['red', 'green', 'blue']
```

#### **Common Use Cases**

**1. Removing Duplicates from Array:**
```javascript
// Most common Set use case
const numbers = [1, 2, 3, 2, 4, 3, 5, 1, 6];
const uniqueNumbers = [...new Set(numbers)];
console.log(uniqueNumbers);  // [1, 2, 3, 4, 5, 6]

// With strings
const words = ['apple', 'banana', 'apple', 'cherry', 'banana'];
const uniqueWords = [...new Set(words)];
console.log(uniqueWords);  // ['apple', 'banana', 'cherry']

// Chaining with other operations
const doubled = [...new Set(numbers)].map(n => n * 2);
console.log(doubled);  // [2, 4, 6, 8, 10, 12]
```

**2. Set Operations (Union, Intersection, Difference):**
```javascript
const setA = new Set([1, 2, 3, 4]);
const setB = new Set([3, 4, 5, 6]);

// Union - all unique values from both sets
const union = new Set([...setA, ...setB]);
console.log([...union]);  // [1, 2, 3, 4, 5, 6]

// Intersection - values in both sets
const intersection = new Set([...setA].filter(x => setB.has(x)));
console.log([...intersection]);  // [3, 4]

// Difference - values in A but not in B
const difference = new Set([...setA].filter(x => !setB.has(x)));
console.log([...difference]);  // [1, 2]

// Symmetric difference - values in A or B but not both
const symmetricDiff = new Set(
  [...setA].filter(x => !setB.has(x))
    .concat([...setB].filter(x => !setA.has(x)))
);
console.log([...symmetricDiff]);  // [1, 2, 5, 6]

// Subset check - all values in A are in B
const isSubset = [...setA].every(x => setB.has(x));
console.log(isSubset);  // false

// Superset check - all values in B are in A
const isSuperset = [...setB].every(x => setA.has(x));
console.log(isSuperset);  // false
```

**3. Tracking Unique Visitors/IDs:**
```javascript
// Track unique users
class UserTracker {
  constructor() {
    this.visitors = new Set();
  }
  
  trackVisit(userId) {
    this.visitors.add(userId);
  }
  
  hasVisited(userId) {
    return this.visitors.has(userId);
  }
  
  getUniqueVisitorCount() {
    return this.visitors.size;
  }
  
  getVisitors() {
    return [...this.visitors];
  }
}

const tracker = new UserTracker();
tracker.trackVisit('user1');
tracker.trackVisit('user2');
tracker.trackVisit('user1');  // Duplicate, not added
console.log(tracker.getUniqueVisitorCount());  // 2
console.log(tracker.hasVisited('user1'));      // true
```

**4. Efficient Membership Testing:**
```javascript
// Array lookup - O(n) time complexity
const allowedIds = [101, 102, 103, 104, 105, /* ... thousands more */];
console.log(allowedIds.includes(103));  // Slow for large arrays

// Set lookup - O(1) time complexity
const allowedIdsSet = new Set(allowedIds);
console.log(allowedIdsSet.has(103));  // Fast, constant time

// Practical example: Permission checking
class PermissionManager {
  constructor(permissions) {
    this.permissions = new Set(permissions);
  }
  
  hasPermission(permission) {
    return this.permissions.has(permission);  // O(1) lookup
  }
  
  grantPermission(permission) {
    this.permissions.add(permission);
  }
  
  revokePermission(permission) {
    this.permissions.delete(permission);
  }
}

const manager = new PermissionManager(['read', 'write', 'delete']);
console.log(manager.hasPermission('write'));  // true (fast)
```

**5. Filtering Unique Objects by Property:**
```javascript
// Remove duplicate objects by specific property
const users = [
  { id: 1, name: 'Alice' },
  { id: 2, name: 'Bob' },
  { id: 1, name: 'Alice Clone' },
  { id: 3, name: 'Charlie' }
];

// Track seen IDs
const seenIds = new Set();
const uniqueUsers = users.filter(user => {
  if (seenIds.has(user.id)) {
    return false;  // Duplicate ID
  }
  seenIds.add(user.id);
  return true;
});

console.log(uniqueUsers);
// [{ id: 1, name: 'Alice' }, { id: 2, name: 'Bob' }, { id: 3, name: 'Charlie' }]

// Alternative: Using Map for more control
const uniqueByIdMap = new Map(users.map(user => [user.id, user]));
const uniqueUsersAlt = [...uniqueByIdMap.values()];
```

**6. Tag/Category Management:**
```javascript
class Article {
  constructor(title, content) {
    this.title = title;
    this.content = content;
    this.tags = new Set();
  }
  
  addTag(tag) {
    this.tags.add(tag.toLowerCase());
    return this;
  }
  
  addTags(...tags) {
    tags.forEach(tag => this.addTag(tag));
    return this;
  }
  
  removeTag(tag) {
    this.tags.delete(tag.toLowerCase());
    return this;
  }
  
  hasTag(tag) {
    return this.tags.has(tag.toLowerCase());
  }
  
  getTags() {
    return [...this.tags];
  }
}

const article = new Article('JavaScript Guide', 'Content here');
article.addTags('javascript', 'es6', 'programming', 'JavaScript');  // Duplicate ignored
console.log(article.getTags());  // ['javascript', 'es6', 'programming']
console.log(article.hasTag('ES6'));  // true (case-insensitive)
```

#### **WeakSet - For Object-Only Collections**

```javascript
// WeakSet only holds objects (not primitives)
const weakSet = new WeakSet();

const obj1 = { id: 1 };
const obj2 = { id: 2 };

weakSet.add(obj1);
weakSet.add(obj2);
// weakSet.add(1);  // TypeError: Invalid value used in weak set

console.log(weakSet.has(obj1));  // true

// WeakSet allows garbage collection
// If obj1 has no other references, it can be GC'd
// weakSet.size is not available (no .size property)

// Use case: Tracking objects without preventing GC
class ObjectTracker {
  constructor() {
    this.tracked = new WeakSet();
  }
  
  track(obj) {
    this.tracked.add(obj);
  }
  
  isTracked(obj) {
    return this.tracked.has(obj);
  }
}

const tracker2 = new ObjectTracker();
let tempObj = { data: 'temporary' };
tracker2.track(tempObj);
console.log(tracker2.isTracked(tempObj));  // true

// When tempObj is no longer referenced elsewhere,
// it can be garbage collected (WeakSet doesn't prevent this)
tempObj = null;
```

#### **Performance Characteristics**

```javascript
// Set operations are generally O(1) for add, delete, has
const largeSet = new Set();

// Adding 1 million items - very fast
console.time('Set add');
for (let i = 0; i < 1000000; i++) {
  largeSet.add(i);
}
console.timeEnd('Set add');  // Fast

// Checking existence - O(1)
console.time('Set has');
console.log(largeSet.has(999999));
console.timeEnd('Set has');  // Very fast (constant time)

// Compare with Array
const largeArray = Array.from({ length: 1000000 }, (_, i) => i);

console.time('Array includes');
console.log(largeArray.includes(999999));
console.timeEnd('Array includes');  // Much slower (linear time)

// Iteration is O(n) for both
console.time('Set iteration');
for (let item of largeSet) {
  // Do nothing, just iterate
}
console.timeEnd('Set iteration');
```

#### **Set vs Array Comparison**

| Feature | Set | Array |
|---------|-----|-------|
| **Duplicates** | Automatically removed | Allowed |
| **Order** | Insertion order maintained | Index-based order |
| **Add operation** | `add()` - O(1) | `push()` - O(1) |
| **Search** | `has()` - O(1) | `includes()` - O(n) |
| **Delete** | `delete()` - O(1) | `splice()` - O(n) |
| **Index access** | Not supported | `arr[i]` - O(1) |
| **Size** | `size` property | `length` property |
| **Iteration** | for...of, forEach | for, for...of, forEach, map, filter |
| **Use case** | Unique values, membership tests | Ordered collections, index access |
| **Memory** | More efficient for unique values | Can waste memory with duplicates |
| **Methods** | add, delete, has, clear | push, pop, shift, unshift, splice, etc. |

#### **Converting Between Set and Array**

```javascript
// Array to Set
const arr = [1, 2, 3, 2, 1];
const set = new Set(arr);

// Set to Array - Method 1: Spread operator
const backToArray1 = [...set];
console.log(backToArray1);  // [1, 2, 3]

// Set to Array - Method 2: Array.from()
const backToArray2 = Array.from(set);
console.log(backToArray2);  // [1, 2, 3]

// Set to Array - Method 3: Manual iteration
const backToArray3 = [];
set.forEach(val => backToArray3.push(val));
console.log(backToArray3);  // [1, 2, 3]

// Using Set methods then converting back
const numbers2 = [1, 2, 3, 4, 5];
const doubled = [...new Set(numbers2)].map(n => n * 2);
const filtered = [...new Set(numbers2)].filter(n => n > 2);
```

#### **Advanced Patterns**

```javascript
// 1. Implementing a custom Set-like class with additional features
class EnhancedSet extends Set {
  union(otherSet) {
    return new EnhancedSet([...this, ...otherSet]);
  }
  
  intersection(otherSet) {
    return new EnhancedSet([...this].filter(x => otherSet.has(x)));
  }
  
  difference(otherSet) {
    return new EnhancedSet([...this].filter(x => !otherSet.has(x)));
  }
  
  isSubsetOf(otherSet) {
    return [...this].every(x => otherSet.has(x));
  }
  
  isSupersetOf(otherSet) {
    return [...otherSet].every(x => this.has(x));
  }
  
  toArray() {
    return [...this];
  }
}

const setX = new EnhancedSet([1, 2, 3]);
const setY = new EnhancedSet([3, 4, 5]);

console.log([...setX.union(setY)]);         // [1, 2, 3, 4, 5]
console.log([...setX.intersection(setY)]);  // [3]
console.log([...setX.difference(setY)]);    // [1, 2]

// 2. Memoization with Set for tracking computed values
const computed = new Set();

function expensiveOperation(id) {
  if (computed.has(id)) {
    console.log('Already computed:', id);
    return;
  }
  
  console.log('Computing:', id);
  // Expensive operation here
  computed.add(id);
}

expensiveOperation(1);  // "Computing: 1"
expensiveOperation(2);  // "Computing: 2"
expensiveOperation(1);  // "Already computed: 1"
```

#### **Common Pitfalls**

```javascript
// 1. Objects are compared by reference, not value
const set1 = new Set();
set1.add({ id: 1 });
set1.add({ id: 1 });
console.log(set1.size);  // 2 (different object references)

// Solution: Use primitive identifiers
const set2 = new Set();
set2.add(1);
set2.add(1);
console.log(set2.size);  // 1

// 2. Cannot access by index
const mySet2 = new Set([1, 2, 3]);
// console.log(mySet2[0]);  // undefined (no index access)
console.log([...mySet2][0]);  // 1 (convert to array first)

// 3. No direct way to map/filter Sets (must convert)
const nums = new Set([1, 2, 3, 4, 5]);
// nums.map(n => n * 2);  // TypeError: nums.map is not a function
const doubled2 = new Set([...nums].map(n => n * 2));

// 4. forEach valueAgain parameter is confusing
mySet2.forEach((value, valueAgain, set) => {
  console.log(value === valueAgain);  // true (same value twice for compatibility)
});

// 5. Adding arrays to Set doesn't flatten
const arrSet = new Set();
arrSet.add([1, 2]);
arrSet.add([1, 2]);
console.log(arrSet.size);  // 2 (different array references)
```

#### **When to Use Set**

**Use Set when:**
- Need to store unique values only
- Frequently checking for value existence
- Order of insertion matters but not index access
- Need efficient add/delete operations
- Working with primitive values
- Implementing mathematical set operations

**Use Array when:**
- Need duplicates
- Require index-based access
- Need array methods (map, filter, reduce)
- Order matters and you need sorting
- Working with complex transformations

```javascript
// ✅ Good use case for Set
function countUniqueWords(text) {
  const words = text.toLowerCase().match(/\b\w+\b/g);
  return new Set(words).size;
}

// ✅ Good use case for Array
function getTopScores(scores, count) {
  return scores.sort((a, b) => b - a).slice(0, count);
}
```

#### **Key Takeaways**

- **Set stores unique values** - duplicates automatically removed
- **O(1) time complexity** for add, delete, and has operations
- **Maintains insertion order** but no index access
- **Values compared by SameValueZero** (NaN === NaN, +0 === -0)
- **Objects compared by reference**, not by value
- **Iterable** with for...of, forEach, values(), keys(), entries()
- **Perfect for removing duplicates** from arrays
- **Efficient membership testing** compared to arrays
- **WeakSet** allows garbage collection of objects
- **No built-in map/filter** - convert to array first
- **size property** for number of elements (not .length)
- **Use for unique collections** where order matters but not index
- **Mathematical set operations** (union, intersection) require custom implementation
- Part of **ES6** - widely supported in modern environments
- Excellent for **performance-critical** membership checks

---

### 69. What is the Map data structure?

**Answer:**

A Map is a built-in JavaScript collection that stores key-value pairs where keys can be of any type (objects, functions, primitives). Unlike plain objects, Maps maintain insertion order, allow any value as a key, and provide better performance for frequent additions and deletions.

#### **Creating and Basic Operations**

```javascript
// Creating a Map
const map1 = new Map();
const map2 = new Map([
  ['name', 'Alice'],
  ['age', 30],
  ['city', 'New York']
]);

console.log(map2);  // Map(3) { 'name' => 'Alice', 'age' => 30, 'city' => 'New York' }

// Setting values
const myMap = new Map();
myMap.set('key1', 'value1');
myMap.set('key2', 'value2');
myMap.set('key3', 'value3');

// Method chaining
myMap.set('key4', 'value4').set('key5', 'value5');

// Getting values
console.log(myMap.get('key1'));  // 'value1'
console.log(myMap.get('unknown'));  // undefined

// Checking existence
console.log(myMap.has('key1'));  // true
console.log(myMap.has('key10')); // false

// Deleting entries
myMap.delete('key2');
console.log(myMap.has('key2'));  // false

// Size property
console.log(myMap.size);  // 4

// Clearing all entries
myMap.clear();
console.log(myMap.size);  // 0
```

#### **Any Type as Key**

```javascript
// Maps accept ANY type as key (major advantage over objects)

// 1. Object keys
const objKey1 = { id: 1 };
const objKey2 = { id: 2 };
const objectMap = new Map();

objectMap.set(objKey1, 'value for obj1');
objectMap.set(objKey2, 'value for obj2');

console.log(objectMap.get(objKey1));  // 'value for obj1'
console.log(objectMap.get({ id: 1 }));  // undefined (different object reference)

// 2. Function keys
function myFunc() {}
const funcMap = new Map();
funcMap.set(myFunc, 'function metadata');
console.log(funcMap.get(myFunc));  // 'function metadata'

// 3. Primitive keys (like objects)
const mixedMap = new Map();
mixedMap.set('string', 'string value');
mixedMap.set(42, 'number value');
mixedMap.set(true, 'boolean value');
mixedMap.set(undefined, 'undefined value');
mixedMap.set(null, 'null value');

console.log(mixedMap.get(42));  // 'number value'
console.log(mixedMap.get(true));  // 'boolean value'

// 4. NaN as key (works correctly in Map)
const nanMap = new Map();
nanMap.set(NaN, 'NaN value');
console.log(nanMap.get(NaN));  // 'NaN value' (works unlike in objects)

// Compare with regular objects
const obj = {};
obj[objKey1] = 'value';  // Converted to '[object Object]' string
obj[objKey2] = 'value2'; // Same string key, overwrites!
console.log(obj);  // { '[object Object]': 'value2' }
```

#### **Iterating Over Maps**

```javascript
const userMap = new Map([
  ['name', 'Alice'],
  ['age', 30],
  ['email', 'alice@example.com']
]);

// 1. for...of with entries (default)
for (let [key, value] of userMap) {
  console.log(`${key}: ${value}`);
}
// Output:
// name: Alice
// age: 30
// email: alice@example.com

// 2. for...of with entries() explicitly
for (let [key, value] of userMap.entries()) {
  console.log(key, value);
}

// 3. for...of with keys()
for (let key of userMap.keys()) {
  console.log(key);  // 'name', 'age', 'email'
}

// 4. for...of with values()
for (let value of userMap.values()) {
  console.log(value);  // 'Alice', 30, 'alice@example.com'
}

// 5. forEach method
userMap.forEach((value, key, map) => {
  console.log(`${key} => ${value}`);
});

// 6. Spread operator
const entries = [...userMap];
console.log(entries);  // [['name', 'Alice'], ['age', 30], ['email', 'alice@example.com']]

const keys = [...userMap.keys()];
console.log(keys);  // ['name', 'age', 'email']

const values = [...userMap.values()];
console.log(values);  // ['Alice', 30, 'alice@example.com']
```

#### **Common Use Cases**

**1. Caching/Memoization:**
```javascript
// Cache function results
class FunctionCache {
  constructor() {
    this.cache = new Map();
  }
  
  memoize(fn) {
    return (...args) => {
      const key = JSON.stringify(args);
      
      if (this.cache.has(key)) {
        console.log('Cache hit!');
        return this.cache.get(key);
      }
      
      console.log('Computing...');
      const result = fn(...args);
      this.cache.set(key, result);
      return result;
    };
  }
  
  clear() {
    this.cache.clear();
  }
}

const cache = new FunctionCache();
const expensiveCalculation = cache.memoize((a, b) => {
  return a + b;
});

console.log(expensiveCalculation(5, 3));  // "Computing..." then 8
console.log(expensiveCalculation(5, 3));  // "Cache hit!" then 8
```

**2. Associating Metadata with Objects:**
```javascript
// Store metadata for DOM elements or objects without modifying them
const elementMetadata = new Map();

function trackElement(element, data) {
  elementMetadata.set(element, {
    clicks: 0,
    lastInteraction: null,
    ...data
  });
}

function recordClick(element) {
  const metadata = elementMetadata.get(element);
  if (metadata) {
    metadata.clicks++;
    metadata.lastInteraction = new Date();
  }
}

// Usage (pseudo-code for DOM)
const button = document.createElement('button');
trackElement(button, { id: 'submit-btn' });
recordClick(button);
console.log(elementMetadata.get(button));  // { clicks: 1, lastInteraction: ..., id: 'submit-btn' }
```

**3. Counting Occurrences (Frequency Counter):**
```javascript
function countOccurrences(items) {
  const counter = new Map();
  
  for (let item of items) {
    counter.set(item, (counter.get(item) || 0) + 1);
  }
  
  return counter;
}

const fruits = ['apple', 'banana', 'apple', 'cherry', 'banana', 'apple'];
const fruitCount = countOccurrences(fruits);

console.log(fruitCount);  // Map(3) { 'apple' => 3, 'banana' => 2, 'cherry' => 1 }
console.log(fruitCount.get('apple'));  // 3

// Get most frequent item
const mostFrequent = [...fruitCount.entries()].reduce((max, entry) => 
  entry[1] > max[1] ? entry : max
);
console.log(mostFrequent);  // ['apple', 3]
```

**4. Grouping Data:**
```javascript
// Group objects by a property
function groupBy(array, keyFn) {
  const groups = new Map();
  
  for (let item of array) {
    const key = keyFn(item);
    if (!groups.has(key)) {
      groups.set(key, []);
    }
    groups.get(key).push(item);
  }
  
  return groups;
}

const users = [
  { name: 'Alice', role: 'admin' },
  { name: 'Bob', role: 'user' },
  { name: 'Charlie', role: 'admin' },
  { name: 'David', role: 'user' }
];

const byRole = groupBy(users, user => user.role);
console.log(byRole);
// Map(2) {
//   'admin' => [{ name: 'Alice', role: 'admin' }, { name: 'Charlie', role: 'admin' }],
//   'user' => [{ name: 'Bob', role: 'user' }, { name: 'David', role: 'user' }]
// }
```

**5. LRU Cache Implementation:**
```javascript
class LRUCache {
  constructor(capacity) {
    this.capacity = capacity;
    this.cache = new Map();
  }
  
  get(key) {
    if (!this.cache.has(key)) {
      return -1;
    }
    
    // Move to end (most recently used)
    const value = this.cache.get(key);
    this.cache.delete(key);
    this.cache.set(key, value);
    return value;
  }
  
  put(key, value) {
    // Remove if exists (to update position)
    if (this.cache.has(key)) {
      this.cache.delete(key);
    }
    
    // Add to end
    this.cache.set(key, value);
    
    // Remove oldest if over capacity
    if (this.cache.size > this.capacity) {
      const firstKey = this.cache.keys().next().value;
      this.cache.delete(firstKey);
    }
  }
}

const lru = new LRUCache(3);
lru.put(1, 'a');
lru.put(2, 'b');
lru.put(3, 'c');
console.log([...lru.cache]);  // [[1, 'a'], [2, 'b'], [3, 'c']]

lru.put(4, 'd');  // Removes 1 (oldest)
console.log([...lru.cache]);  // [[2, 'b'], [3, 'c'], [4, 'd']]

lru.get(2);  // Access 2, moves it to end
console.log([...lru.cache]);  // [[3, 'c'], [4, 'd'], [2, 'b']]
```

**6. Bidirectional Mapping:**
```javascript
class BiMap {
  constructor() {
    this.keyToValue = new Map();
    this.valueToKey = new Map();
  }
  
  set(key, value) {
    // Remove old mappings if they exist
    if (this.keyToValue.has(key)) {
      this.valueToKey.delete(this.keyToValue.get(key));
    }
    if (this.valueToKey.has(value)) {
      this.keyToValue.delete(this.valueToKey.get(value));
    }
    
    this.keyToValue.set(key, value);
    this.valueToKey.set(value, key);
  }
  
  getByKey(key) {
    return this.keyToValue.get(key);
  }
  
  getByValue(value) {
    return this.valueToKey.get(value);
  }
}

const bimap = new BiMap();
bimap.set('en', 'English');
bimap.set('es', 'Spanish');

console.log(bimap.getByKey('en'));       // 'English'
console.log(bimap.getByValue('Spanish')); // 'es'
```

#### **WeakMap - For Object-Only Keys**

```javascript
// WeakMap only accepts objects as keys
const weakMap = new WeakMap();

const obj1 = { id: 1 };
const obj2 = { id: 2 };

weakMap.set(obj1, 'data for obj1');
weakMap.set(obj2, 'data for obj2');
// weakMap.set('string', 'value');  // TypeError: Invalid value used as weak map key

console.log(weakMap.get(obj1));  // 'data for obj1'

// WeakMap allows garbage collection
// No .size, .clear(), or iteration methods

// Use case: Private data for objects
const privateData = new WeakMap();

class SecureUser {
  constructor(name, password) {
    this.name = name;  // Public
    privateData.set(this, { password });  // Private
  }
  
  authenticate(password) {
    return privateData.get(this).password === password;
  }
}

const user = new SecureUser('Alice', 'secret123');
console.log(user.name);  // 'Alice'
console.log(user.password);  // undefined (not accessible)
console.log(user.authenticate('secret123'));  // true

// When user is garbage collected, its private data is too
```

#### **Map vs Object Comparison**

| Feature | Map | Object |
|---------|-----|--------|
| **Key types** | Any type (objects, functions, primitives) | Strings and Symbols only |
| **Key order** | Insertion order guaranteed | Not guaranteed (numeric keys sorted) |
| **Size** | `map.size` (O(1)) | `Object.keys(obj).length` (O(n)) |
| **Iteration** | Directly iterable (for...of) | Need Object.keys/values/entries |
| **Performance** | Better for frequent add/delete | Better for small fixed structures |
| **Serialization** | No native JSON support | `JSON.stringify()` works |
| **Prototype** | Clean, no inherited keys | Has prototype chain |
| **Accidental keys** | No | Yes (`toString`, `constructor`, etc.) |
| **Default values** | None | Inherited from prototype |
| **Use case** | Dynamic key-value pairs | Fixed structure, property access |

```javascript
// Object quirks that Map avoids

// 1. Object keys are always strings (or symbols)
const obj = {};
obj[1] = 'number key';
obj['1'] = 'string key';  // Overwrites above!
console.log(obj);  // { '1': 'string key' }

const map = new Map();
map.set(1, 'number key');
map.set('1', 'string key');  // Different keys!
console.log(map);  // Map(2) { 1 => 'number key', '1' => 'string key' }

// 2. Object prototype pollution
const obj2 = {};
console.log(obj2.toString);  // [Function: toString] (inherited)

const map2 = new Map();
console.log(map2.get('toString'));  // undefined (clean)

// 3. Checking key existence
const obj3 = { hasOwnProperty: true };  // Shadows method!
// obj3.hasOwnProperty('key');  // TypeError

const map3 = new Map();
map3.set('hasOwnProperty', true);
map3.has('hasOwnProperty');  // Works fine
```

#### **Converting Between Map and Object**

```javascript
// Object to Map
const obj = { name: 'Alice', age: 30, city: 'NYC' };
const mapFromObj = new Map(Object.entries(obj));
console.log(mapFromObj);  // Map(3) { 'name' => 'Alice', 'age' => 30, 'city' => 'NYC' }

// Map to Object
const map = new Map([
  ['name', 'Bob'],
  ['age', 25]
]);

const objFromMap = Object.fromEntries(map);
console.log(objFromMap);  // { name: 'Bob', age: 25 }

// Alternative: Manual conversion
const objManual = {};
map.forEach((value, key) => {
  objManual[key] = value;
});
```

#### **Performance Characteristics**

```javascript
// Map is optimized for frequent additions/deletions
const iterations = 100000;

// Map performance
const map = new Map();
console.time('Map set');
for (let i = 0; i < iterations; i++) {
  map.set(i, i);
}
console.timeEnd('Map set');  // Fast

console.time('Map get');
for (let i = 0; i < iterations; i++) {
  map.get(i);
}
console.timeEnd('Map get');  // Fast

console.time('Map delete');
for (let i = 0; i < iterations; i++) {
  map.delete(i);
}
console.timeEnd('Map delete');  // Fast

// Object performance (comparable for small data)
const obj = {};
console.time('Object set');
for (let i = 0; i < iterations; i++) {
  obj[i] = i;
}
console.timeEnd('Object set');

// Map.size is O(1), Object.keys().length is O(n)
console.log(map.size);  // Instant
console.log(Object.keys(obj).length);  // Slower for large objects
```

#### **When to Use Map vs Object**

**Use Map when:**
- Keys are not strings/symbols
- Keys are unknown until runtime
- Need to iterate in insertion order
- Frequently adding/removing key-value pairs
- Need to know size efficiently
- Storing arbitrary key-value pairs

**Use Object when:**
- Keys are fixed and known
- Need JSON serialization
- Using as a record/struct with specific properties
- Need property access syntax (obj.prop)
- Small fixed structure

```javascript
// ✅ Good use case for Map
const userSessions = new Map();
function createSession(user) {
  userSessions.set(user, {
    startTime: Date.now(),
    lastActivity: Date.now()
  });
}

// ✅ Good use case for Object
const config = {
  apiUrl: 'https://api.example.com',
  timeout: 5000,
  retries: 3
};
```

#### **Key Takeaways**

- **Map stores key-value pairs** with any type as key
- **Maintains insertion order** for all keys
- **O(1) operations** for set, get, has, delete
- **size property** returns count in constant time
- **Any value as key** - objects, functions, primitives
- **Iterable by default** - works with for...of directly
- **No prototype pollution** - clean namespace
- **Better performance** for frequent add/delete operations
- **WeakMap** for object-only keys with garbage collection
- **No native JSON** serialization (use Object.fromEntries)
- **Preserves key types** unlike objects (1 vs '1')
- **Perfect for caching**, metadata, frequency counting
- **Use over objects** when keys are dynamic or non-string
- Part of **ES6** - widely supported
- **Predictable iteration order** unlike objects

---

### 70. What are iterators and generators?

**Answer:**

Iterators and generators are ES6 features that enable custom iteration behavior. An iterator is an object that provides a `next()` method for sequential access to values. A generator is a special function (using `function*` syntax) that can pause execution and yield multiple values over time, automatically creating an iterator.

#### **Iterators**

**Basic Iterator Concept:**
```javascript
// Iterator protocol: object with a next() method
// next() returns: { value: any, done: boolean }

const iterator = {
  current: 0,
  last: 5,
  
  next() {
    if (this.current <= this.last) {
      return { value: this.current++, done: false };
    } else {
      return { value: undefined, done: true };
    }
  }
};

console.log(iterator.next());  // { value: 0, done: false }
console.log(iterator.next());  // { value: 1, done: false }
console.log(iterator.next());  // { value: 2, done: false }
// ... continues until
console.log(iterator.next());  // { value: undefined, done: true }
```

**Iterable Protocol:**
```javascript
// Iterable: object with [Symbol.iterator]() method that returns an iterator

const iterable = {
  data: [1, 2, 3, 4, 5],
  
  [Symbol.iterator]() {
    let index = 0;
    const data = this.data;
    
    return {
      next() {
        if (index < data.length) {
          return { value: data[index++], done: false };
        } else {
          return { done: true };
        }
      }
    };
  }
};

// Now iterable works with for...of
for (let value of iterable) {
  console.log(value);  // 1, 2, 3, 4, 5
}

// Also works with spread operator
console.log([...iterable]);  // [1, 2, 3, 4, 5]

// And destructuring
const [first, second, ...rest] = iterable;
console.log(first, second, rest);  // 1, 2, [3, 4, 5]
```

**Custom Range Iterator:**
```javascript
class Range {
  constructor(start, end, step = 1) {
    this.start = start;
    this.end = end;
    this.step = step;
  }
  
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

const range = new Range(1, 10, 2);
console.log([...range]);  // [1, 3, 5, 7, 9]

for (let num of range) {
  console.log(num);  // 1, 3, 5, 7, 9
}
```

#### **Generators**

**Basic Generator Syntax:**
```javascript
// Generator function uses function* syntax
function* simpleGenerator() {
  yield 1;
  yield 2;
  yield 3;
}

const gen = simpleGenerator();

console.log(gen.next());  // { value: 1, done: false }
console.log(gen.next());  // { value: 2, done: false }
console.log(gen.next());  // { value: 3, done: false }
console.log(gen.next());  // { value: undefined, done: true }

// Generators are iterable
for (let value of simpleGenerator()) {
  console.log(value);  // 1, 2, 3
}

console.log([...simpleGenerator()]);  // [1, 2, 3]
```

**Generator with Logic:**
```javascript
function* numberGenerator(max) {
  let num = 0;
  while (num < max) {
    yield num++;
  }
  return 'Done!';  // Return value not included in iteration
}

const nums = numberGenerator(5);
console.log([...nums]);  // [0, 1, 2, 3, 4]

// Return value accessible only via next()
const nums2 = numberGenerator(3);
console.log(nums2.next());  // { value: 0, done: false }
console.log(nums2.next());  // { value: 1, done: false }
console.log(nums2.next());  // { value: 2, done: false }
console.log(nums2.next());  // { value: 'Done!', done: true }
```

**Infinite Generators:**
```javascript
function* infiniteSequence() {
  let num = 0;
  while (true) {
    yield num++;
  }
}

const sequence = infiniteSequence();
console.log(sequence.next().value);  // 0
console.log(sequence.next().value);  // 1
console.log(sequence.next().value);  // 2

// Use with care - infinite loop if fully consumed!
// Take only what you need
function* take(iterable, count) {
  let taken = 0;
  for (let item of iterable) {
    if (taken++ >= count) return;
    yield item;
  }
}

console.log([...take(infiniteSequence(), 5)]);  // [0, 1, 2, 3, 4]
```

**Passing Values to Generators:**
```javascript
function* generatorWithInput() {
  const first = yield 'Ready';
  console.log('Received:', first);
  
  const second = yield 'Next';
  console.log('Received:', second);
  
  return 'Complete';
}

const gen2 = generatorWithInput();
console.log(gen2.next());         // { value: 'Ready', done: false }
console.log(gen2.next('Hello'));  // Logs "Received: Hello", returns { value: 'Next', done: false }
console.log(gen2.next('World'));  // Logs "Received: World", returns { value: 'Complete', done: true }
```

**Generator Delegation (yield*):**
```javascript
function* generator1() {
  yield 1;
  yield 2;
}

function* generator2() {
  yield 'a';
  yield 'b';
}

function* combinedGenerator() {
  yield* generator1();  // Delegate to generator1
  yield* generator2();  // Delegate to generator2
  yield 'final';
}

console.log([...combinedGenerator()]);  // [1, 2, 'a', 'b', 'final']

// yield* also works with any iterable
function* arrayGenerator() {
  yield* [1, 2, 3];
  yield* 'hello';  // Strings are iterable
  yield* new Set([4, 5, 6]);
}

console.log([...arrayGenerator()]);  // [1, 2, 3, 'h', 'e', 'l', 'l', 'o', 4, 5, 6]
```

#### **Practical Use Cases**

**1. Lazy Evaluation:**
```javascript
// Generate values on-demand without storing entire sequence
function* fibonacci() {
  let [prev, curr] = [0, 1];
  while (true) {
    yield curr;
    [prev, curr] = [curr, prev + curr];
  }
}

// Only compute what we need
const fib = fibonacci();
console.log(fib.next().value);  // 1
console.log(fib.next().value);  // 1
console.log(fib.next().value);  // 2
console.log(fib.next().value);  // 3
console.log(fib.next().value);  // 5

// Get first 10 Fibonacci numbers
console.log([...take(fibonacci(), 10)]);
// [1, 1, 2, 3, 5, 8, 13, 21, 34, 55]
```

**2. Pagination/Chunking:**
```javascript
function* paginate(array, pageSize) {
  for (let i = 0; i < array.length; i += pageSize) {
    yield array.slice(i, i + pageSize);
  }
}

const items = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
for (let page of paginate(items, 3)) {
  console.log(page);
}
// [1, 2, 3]
// [4, 5, 6]
// [7, 8, 9]
// [10]
```

**3. Async Operations (with async generators):**
```javascript
async function* fetchPages(url, maxPages) {
  for (let page = 1; page <= maxPages; page++) {
    const response = await fetch(`${url}?page=${page}`);
    const data = await response.json();
    yield data;
  }
}

// Usage (in async context)
async function processPages() {
  for await (let data of fetchPages('/api/users', 5)) {
    console.log('Processing page:', data);
    // Process each page as it arrives
  }
}
```

**4. State Machines:**
```javascript
function* trafficLight() {
  while (true) {
    yield 'RED';
    yield 'GREEN';
    yield 'YELLOW';
  }
}

const light = trafficLight();
console.log(light.next().value);  // 'RED'
console.log(light.next().value);  // 'GREEN'
console.log(light.next().value);  // 'YELLOW'
console.log(light.next().value);  // 'RED' (cycles)
```

**5. Tree Traversal:**
```javascript
class TreeNode {
  constructor(value, left = null, right = null) {
    this.value = value;
    this.left = left;
    this.right = right;
  }
  
  * inOrder() {
    if (this.left) yield* this.left.inOrder();
    yield this.value;
    if (this.right) yield* this.right.inOrder();
  }
  
  * preOrder() {
    yield this.value;
    if (this.left) yield* this.left.preOrder();
    if (this.right) yield* this.right.preOrder();
  }
  
  * postOrder() {
    if (this.left) yield* this.left.postOrder();
    if (this.right) yield* this.right.postOrder();
    yield this.value;
  }
}

const tree = new TreeNode(
  4,
  new TreeNode(2, new TreeNode(1), new TreeNode(3)),
  new TreeNode(6, new TreeNode(5), new TreeNode(7))
);

console.log([...tree.inOrder()]);    // [1, 2, 3, 4, 5, 6, 7]
console.log([...tree.preOrder()]);   // [4, 2, 1, 3, 6, 5, 7]
console.log([...tree.postOrder()]);  // [1, 3, 2, 5, 7, 6, 4]
```

**6. Data Streaming:**
```javascript
function* lineReader(text) {
  const lines = text.split('\n');
  for (let line of lines) {
    yield line.trim();
  }
}

const logData = `
  2024-01-01: User logged in
  2024-01-02: File uploaded
  2024-01-03: Error occurred
`;

for (let line of lineReader(logData)) {
  if (line) {
    console.log('Processing:', line);
  }
}
```

**7. Custom Iteration Logic:**
```javascript
class Collection {
  constructor(items) {
    this.items = items;
  }
  
  * [Symbol.iterator]() {
    for (let item of this.items) {
      yield item;
    }
  }
  
  * reversed() {
    for (let i = this.items.length - 1; i >= 0; i--) {
      yield this.items[i];
    }
  }
  
  * filtered(predicate) {
    for (let item of this.items) {
      if (predicate(item)) {
        yield item;
      }
    }
  }
  
  * mapped(fn) {
    for (let item of this.items) {
      yield fn(item);
    }
  }
}

const coll = new Collection([1, 2, 3, 4, 5]);

console.log([...coll]);                          // [1, 2, 3, 4, 5]
console.log([...coll.reversed()]);               // [5, 4, 3, 2, 1]
console.log([...coll.filtered(n => n > 2)]);     // [3, 4, 5]
console.log([...coll.mapped(n => n * 2)]);       // [2, 4, 6, 8, 10]
```

#### **Generator Methods**

```javascript
function* generatorWithMethods() {
  try {
    yield 1;
    yield 2;
    yield 3;
  } catch (error) {
    console.log('Caught:', error.message);
  } finally {
    console.log('Cleanup');
  }
}

const gen3 = generatorWithMethods();

// next() - Get next value
console.log(gen3.next());  // { value: 1, done: false }

// return() - Force generator to finish
console.log(gen3.return('Forced end'));  // Logs "Cleanup", returns { value: 'Forced end', done: true }
console.log(gen3.next());  // { value: undefined, done: true }

// throw() - Throw error into generator
const gen4 = generatorWithMethods();
console.log(gen4.next());  // { value: 1, done: false }
console.log(gen4.throw(new Error('Test error')));  // Logs "Caught: Test error" and "Cleanup"
```

#### **Iterator vs Generator Comparison**

| Feature | Iterator | Generator |
|---------|----------|-----------|
| **Syntax** | Object with `next()` method | `function*` with `yield` |
| **Creation** | Manual implementation | Automatic iterator creation |
| **State** | Manual state management | Automatic state preservation |
| **Complexity** | More verbose | Concise and readable |
| **Pause/Resume** | Manual control needed | Built-in with `yield` |
| **Return value** | In `next()` method | With `return` statement |
| **Error handling** | Manual | Try-catch works naturally |
| **Use case** | Low-level control | Most common use cases |

```javascript
// Same functionality - Iterator vs Generator

// ITERATOR (manual)
const manualIterator = {
  [Symbol.iterator]() {
    let count = 0;
    return {
      next() {
        if (count < 3) {
          return { value: count++, done: false };
        }
        return { done: true };
      }
    };
  }
};

// GENERATOR (automatic)
function* autoGenerator() {
  let count = 0;
  while (count < 3) {
    yield count++;
  }
}

console.log([...manualIterator]);  // [0, 1, 2]
console.log([...autoGenerator()]);  // [0, 1, 2]
```

#### **Built-in Iterables**

```javascript
// Many built-in types are iterable

// Arrays
for (let item of [1, 2, 3]) { }

// Strings
for (let char of 'hello') { }

// Maps
for (let [key, value] of new Map()) { }

// Sets
for (let value of new Set()) { }

// TypedArrays
for (let byte of new Uint8Array([1, 2, 3])) { }

// Arguments object
function test() {
  for (let arg of arguments) { }
}

// NodeList (DOM)
// for (let element of document.querySelectorAll('div')) { }
```

#### **Key Takeaways**

- **Iterators** provide sequential access via `next()` method
- **Generators** are functions that can pause and resume execution
- **`function*` syntax** defines a generator function
- **`yield` keyword** pauses generator and returns value
- **Generators automatically** implement iterator protocol
- **Lazy evaluation** - values computed on-demand
- **Memory efficient** for large/infinite sequences
- **State preservation** between yield points
- **`yield*` delegates** to another generator or iterable
- **Async generators** (`async function*`) for async iteration
- **Perfect for** pagination, streaming, infinite sequences
- **`return()` method** forces generator completion
- **`throw()` method** injects error into generator
- **Works with** for...of, spread operator, destructuring
- Part of **ES6** - widely supported
- **Simplifies** complex iteration logic significantly

68. What is the Set data structure?
69. What is the Map data structure?
70. What are iterators and generators?