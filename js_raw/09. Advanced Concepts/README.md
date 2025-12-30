## Advanced Concepts

---

### 79. What is the `this` keyword?

**Answer:**

`this` is a special keyword in JavaScript that refers to the context in which a function is executed. Its value is determined by **how a function is called**, not where it's defined. `this` provides access to the object that is currently executing the code, enabling methods to access their parent object's properties and functions to be reusable across different objects.

#### **Basic Concept**

```javascript
// this refers to the object calling the method
const person = {
  name: 'John',
  greet: function() {
    console.log('Hello, ' + this.name);
  }
};

person.greet();  // 'Hello, John' - this refers to person

// Same function, different context
const anotherPerson = {
  name: 'Jane',
  greet: person.greet  // Reuse the same function
};

anotherPerson.greet();  // 'Hello, Jane' - this refers to anotherPerson
```

#### **1. Global Context**

```javascript
// In browsers (non-strict mode)
console.log(this);  // Window object

// In Node.js (top level)
console.log(this);  // {} (empty object or global)

function showThis() {
  console.log(this);
}

showThis();  // Window (browser, non-strict) or global (Node.js)

// In strict mode
'use strict';
function showThisStrict() {
  console.log(this);
}

showThisStrict();  // undefined
```

#### **2. Object Method Context**

```javascript
const user = {
  name: 'Alice',
  age: 25,
  getName: function() {
    return this.name;  // this = user
  },
  getInfo: function() {
    return `${this.name} is ${this.age} years old`;
  }
};

console.log(user.getName());   // 'Alice'
console.log(user.getInfo());   // 'Alice is 25 years old'

// Nested objects
const company = {
  name: 'TechCorp',
  employee: {
    name: 'Bob',
    getName: function() {
      return this.name;  // this = employee (immediate parent)
    }
  }
};

console.log(company.employee.getName());  // 'Bob' (not 'TechCorp')
```

#### **3. Constructor Function Context**

```javascript
function Person(name, age) {
  // this refers to the new object being created
  this.name = name;
  this.age = age;
  this.greet = function() {
    console.log(`Hi, I'm ${this.name}`);
  };
}

const person1 = new Person('John', 30);
const person2 = new Person('Jane', 25);

person1.greet();  // 'Hi, I'm John'
person2.greet();  // 'Hi, I'm Jane'

// Without 'new', this refers to global object (bad!)
const person3 = Person('Bob', 35);  // Forgot 'new'
console.log(person3);      // undefined
console.log(window.name);  // 'Bob' (polluted global!)
```

#### **4. Class Context**

```javascript
class Animal {
  constructor(name) {
    this.name = name;  // this = new instance
  }
  
  speak() {
    console.log(`${this.name} makes a sound`);
  }
  
  static info() {
    console.log(this);  // this = Animal class itself
  }
}

const dog = new Animal('Dog');
dog.speak();      // 'Dog makes a sound'
Animal.info();    // [class Animal]

// Instance vs Class
console.log(dog.name);     // 'Dog' (instance property)
console.log(Animal.name);  // 'Animal' (class name, not instance)
```

#### **5. Arrow Functions and this**

```javascript
// Arrow functions don't have their own 'this'
// They inherit 'this' from parent scope (lexical this)

const obj = {
  name: 'Object',
  
  // Regular function
  regularMethod: function() {
    console.log(this.name);  // 'Object'
    
    // Nested regular function loses context
    setTimeout(function() {
      console.log(this.name);  // undefined (this = Window or global)
    }, 100);
  },
  
  // Arrow function
  arrowMethod: function() {
    console.log(this.name);  // 'Object'
    
    // Nested arrow function keeps context
    setTimeout(() => {
      console.log(this.name);  // 'Object' (inherits from parent)
    }, 100);
  }
};

obj.regularMethod();
obj.arrowMethod();

// Arrow function as method (problem!)
const obj2 = {
  name: 'Object2',
  arrowAsMethod: () => {
    console.log(this.name);  // undefined (inherits from global scope)
  }
};

obj2.arrowAsMethod();  // 'this' is NOT obj2
```

#### **6. Event Handler Context**

```javascript
// In event handlers, 'this' refers to the element that received the event

const button = document.querySelector('button');

// Regular function
button.addEventListener('click', function() {
  console.log(this);  // <button> element
  this.classList.add('clicked');
  this.textContent = 'Clicked!';
});

// Arrow function (doesn't bind 'this')
button.addEventListener('click', () => {
  console.log(this);  // Window or parent scope, NOT button
  // this.classList.add('clicked');  // ❌ Won't work as expected
});

// Using event parameter
button.addEventListener('click', (event) => {
  console.log(event.target);  // <button> element
  event.target.classList.add('clicked');
});
```

#### **7. Explicit Binding (call, apply, bind)**

```javascript
function greet(greeting, punctuation) {
  console.log(`${greeting}, ${this.name}${punctuation}`);
}

const person = { name: 'John' };

// call() - invoke immediately with specific 'this'
greet.call(person, 'Hello', '!');  // 'Hello, John!'

// apply() - same as call but args as array
greet.apply(person, ['Hi', '.']);  // 'Hi, John.'

// bind() - create new function with bound 'this'
const greetPerson = greet.bind(person);
greetPerson('Hey', '?');  // 'Hey, John?'

// Partial application with bind
const greetJohn = greet.bind(person, 'Greetings');
greetJohn('!');  // 'Greetings, John!'

// Real-world example: method borrowing
const user1 = {
  name: 'Alice',
  greet: function() {
    console.log(`Hello, ${this.name}`);
  }
};

const user2 = { name: 'Bob' };

user1.greet.call(user2);  // 'Hello, Bob' (borrowed method)
```

#### **8. Common 'this' Pitfalls**

```javascript
// Problem 1: Method reference loses context
const person = {
  name: 'John',
  greet: function() {
    console.log(`Hi, ${this.name}`);
  }
};

person.greet();           // 'Hi, John' ✅
const greetFunc = person.greet;
greetFunc();              // 'Hi, undefined' ❌ (lost context)

// Solution 1: Use bind
const boundGreet = person.greet.bind(person);
boundGreet();             // 'Hi, John' ✅

// Solution 2: Arrow function wrapper
const wrappedGreet = () => person.greet();
wrappedGreet();           // 'Hi, John' ✅

// Problem 2: Callback functions lose context
const counter = {
  count: 0,
  increment: function() {
    this.count++;
    console.log(this.count);
  }
};

counter.increment();      // 1 ✅

setTimeout(counter.increment, 1000);  // NaN ❌ (lost context)

// Solution: Bind or arrow function
setTimeout(counter.increment.bind(counter), 1000);  // 2 ✅
setTimeout(() => counter.increment(), 1000);        // 2 ✅

// Problem 3: Nested functions
const obj = {
  name: 'Object',
  method: function() {
    console.log(this.name);  // 'Object' ✅
    
    function nested() {
      console.log(this.name);  // undefined ❌
    }
    nested();
  }
};

obj.method();

// Solution 1: Save 'this' reference
const obj2 = {
  name: 'Object2',
  method: function() {
    const self = this;  // Save reference
    function nested() {
      console.log(self.name);  // 'Object2' ✅
    }
    nested();
  }
};

// Solution 2: Use arrow function
const obj3 = {
  name: 'Object3',
  method: function() {
    const nested = () => {
      console.log(this.name);  // 'Object3' ✅ (lexical this)
    };
    nested();
  }
};
```

#### **Practical Examples**

**1. Object Methods:**
```javascript
const calculator = {
  value: 0,
  
  add: function(num) {
    this.value += num;
    return this;  // Return this for chaining
  },
  
  subtract: function(num) {
    this.value -= num;
    return this;
  },
  
  multiply: function(num) {
    this.value *= num;
    return this;
  },
  
  getValue: function() {
    return this.value;
  }
};

// Method chaining
const result = calculator
  .add(10)
  .multiply(2)
  .subtract(5)
  .getValue();

console.log(result);  // 15
```

**2. Event Handling:**
```javascript
class ToggleButton {
  constructor(element) {
    this.element = element;
    this.isOn = false;
    
    // Bind event handler to maintain 'this' context
    this.element.addEventListener('click', this.toggle.bind(this));
    
    // Alternative: arrow function (if not needing to remove listener)
    // this.element.addEventListener('click', () => this.toggle());
  }
  
  toggle() {
    this.isOn = !this.isOn;
    this.element.textContent = this.isOn ? 'ON' : 'OFF';
    this.element.classList.toggle('active', this.isOn);
    console.log(`Button is now ${this.isOn ? 'on' : 'off'}`);
  }
}

const button = document.querySelector('.toggle-btn');
const toggleBtn = new ToggleButton(button);
```

**3. Timer with Context:**
```javascript
class Timer {
  constructor(duration) {
    this.duration = duration;
    this.timeLeft = duration;
    this.intervalId = null;
  }
  
  start() {
    console.log('Timer started');
    
    // Use arrow function to preserve 'this'
    this.intervalId = setInterval(() => {
      this.timeLeft--;
      console.log(`Time left: ${this.timeLeft}`);
      
      if (this.timeLeft <= 0) {
        this.stop();
      }
    }, 1000);
  }
  
  stop() {
    clearInterval(this.intervalId);
    console.log('Timer stopped');
  }
  
  reset() {
    this.stop();
    this.timeLeft = this.duration;
  }
}

const timer = new Timer(5);
timer.start();
```

**4. DOM Manipulation:**
```javascript
class TodoList {
  constructor(containerElement) {
    this.container = containerElement;
    this.todos = [];
    this.setupEventListeners();
  }
  
  setupEventListeners() {
    // Use arrow functions to preserve 'this'
    this.container.addEventListener('click', (e) => {
      if (e.target.classList.contains('delete-btn')) {
        this.deleteTodo(e.target.dataset.id);
      }
      if (e.target.classList.contains('complete-btn')) {
        this.toggleComplete(e.target.dataset.id);
      }
    });
  }
  
  addTodo(text) {
    const todo = {
      id: Date.now(),
      text: text,
      completed: false
    };
    this.todos.push(todo);
    this.render();
  }
  
  deleteTodo(id) {
    this.todos = this.todos.filter(t => t.id !== Number(id));
    this.render();
  }
  
  toggleComplete(id) {
    const todo = this.todos.find(t => t.id === Number(id));
    if (todo) {
      todo.completed = !todo.completed;
      this.render();
    }
  }
  
  render() {
    this.container.innerHTML = this.todos.map(todo => `
      <div class="todo ${todo.completed ? 'completed' : ''}">
        <span>${todo.text}</span>
        <button class="complete-btn" data-id="${todo.id}">✓</button>
        <button class="delete-btn" data-id="${todo.id}">×</button>
      </div>
    `).join('');
  }
}

const todoList = new TodoList(document.getElementById('todo-container'));
todoList.addTodo('Learn JavaScript');
todoList.addTodo('Master this keyword');
```

**5. API Service:**
```javascript
class ApiService {
  constructor(baseUrl) {
    this.baseUrl = baseUrl;
    this.defaultHeaders = {
      'Content-Type': 'application/json'
    };
  }
  
  async get(endpoint) {
    const url = `${this.baseUrl}${endpoint}`;
    const response = await fetch(url, {
      headers: this.defaultHeaders
    });
    return response.json();
  }
  
  async post(endpoint, data) {
    const url = `${this.baseUrl}${endpoint}`;
    const response = await fetch(url, {
      method: 'POST',
      headers: this.defaultHeaders,
      body: JSON.stringify(data)
    });
    return response.json();
  }
  
  setAuthToken(token) {
    this.defaultHeaders['Authorization'] = `Bearer ${token}`;
  }
}

const api = new ApiService('https://api.example.com');
api.setAuthToken('my-token');

// Methods maintain 'this' context
api.get('/users')
  .then(users => console.log(users));
```

**6. React-like Component:**
```javascript
class Component {
  constructor(props) {
    this.props = props;
    this.state = {};
    
    // Bind methods to preserve 'this'
    this.handleClick = this.handleClick.bind(this);
    this.handleChange = this.handleChange.bind(this);
  }
  
  setState(newState) {
    this.state = { ...this.state, ...newState };
    this.render();
  }
  
  handleClick() {
    console.log('Clicked:', this.state.value);
  }
  
  handleChange(e) {
    this.setState({ value: e.target.value });
  }
  
  render() {
    // Render logic using this.state and this.props
    console.log('Rendering with state:', this.state);
  }
}

// Alternative: Use arrow functions (class fields)
class ModernComponent {
  constructor(props) {
    this.props = props;
    this.state = {};
  }
  
  // Arrow functions automatically bind 'this'
  handleClick = () => {
    console.log('Clicked:', this.state.value);
  }
  
  handleChange = (e) => {
    this.setState({ value: e.target.value });
  }
}
```

#### **'this' Determination Rules (Priority Order)**

```javascript
// 1. new binding (highest priority)
function Person(name) {
  this.name = name;  // this = new object
}
const p = new Person('John');

// 2. Explicit binding (call, apply, bind)
function greet() {
  console.log(this.name);
}
const obj = { name: 'Alice' };
greet.call(obj);  // this = obj

// 3. Implicit binding (method call)
const person = {
  name: 'Bob',
  greet: function() {
    console.log(this.name);  // this = person
  }
};
person.greet();

// 4. Default binding (lowest priority)
function defaultFunc() {
  console.log(this);  // this = global object (or undefined in strict mode)
}
defaultFunc();

// 5. Arrow functions (lexical, not dynamic)
// Arrow functions ignore all rules above and use lexical scope
const obj2 = {
  name: 'Charlie',
  greet: () => {
    console.log(this);  // this = outer scope, NOT obj2
  }
};
```

#### **Summary Table**

| Context | 'this' Value | Example |
|---------|-------------|---------|
| **Global** | Window/global | `console.log(this)` |
| **Function** | Window/undefined (strict) | `function f() { this }` |
| **Method** | Parent object | `obj.method()` |
| **Constructor** | New instance | `new Func()` |
| **Arrow function** | Lexical (outer scope) | `() => this` |
| **call/apply** | First argument | `func.call(obj)` |
| **bind** | Bound object | `func.bind(obj)` |
| **Event handler** | Event target | `element.addEventListener()` |
| **Class method** | Instance | `class { method() }` |
| **Static method** | Class itself | `class { static method() }` |

#### **Common Mistakes**

```javascript
// 1. Forgetting to bind
class MyClass {
  constructor() {
    this.value = 42;
  }
  
  method() {
    console.log(this.value);
  }
}

const obj = new MyClass();
setTimeout(obj.method, 1000);  // ❌ undefined

// Fix:
setTimeout(obj.method.bind(obj), 1000);  // ✅ 42

// 2. Arrow function as object method
const obj2 = {
  value: 42,
  method: () => {
    console.log(this.value);  // ❌ undefined (wrong scope)
  }
};

// Fix: Use regular function
const obj3 = {
  value: 42,
  method: function() {
    console.log(this.value);  // ✅ 42
  }
};

// 3. Reassigning methods
const calculator2 = {
  value: 10,
  getValue: function() {
    return this.value;
  }
};

const getValue = calculator2.getValue;
console.log(getValue());  // ❌ undefined (lost context)

// Fix:
const boundGetValue = calculator2.getValue.bind(calculator2);
console.log(boundGetValue());  // ✅ 10
```

#### **Key Takeaways**

- **'this' is determined by how a function is called**
- **Method call**: 'this' = object before the dot
- **Constructor**: 'this' = newly created object
- **Arrow functions**: 'this' = lexical scope (inherited)
- **Regular functions**: 'this' = calling context
- **Global context**: 'this' = window/global (or undefined in strict)
- **Use bind()** to permanently set 'this'
- **Use call/apply()** to set 'this' for one invocation
- **Event handlers**: 'this' = element (regular functions)
- **Arrow functions** don't have their own 'this'
- **Save reference** with `const self = this` pattern
- **Method chaining**: return 'this' from methods
- **Class methods**: usually need binding for callbacks
- Understanding **'this' is crucial** for OOP in JavaScript
- **Context loss** is a common bug source

---

### 80. How does `this` work in different contexts?

**Answer:**

The value of `this` in JavaScript depends entirely on the **execution context** - how and where a function is called. It follows specific binding rules with a priority hierarchy: `new` binding > explicit binding > implicit binding > default binding. Arrow functions are special and use lexical scoping instead.

#### **1. Global Context**

```javascript
// Browser (non-strict mode)
console.log(this);  // Window object
console.log(this === window);  // true

// Browser (strict mode)
'use strict';
console.log(this);  // Window (still Window at global level)

// Node.js
console.log(this);  // {} or global object

// Global function call
function showThis() {
  console.log(this);
}

showThis();  // Window (non-strict) or undefined (strict)

// With strict mode
'use strict';
function showThisStrict() {
  console.log(this);
}

showThisStrict();  // undefined
```

#### **2. Object Method Context (Implicit Binding)**

```javascript
const user = {
  name: 'Alice',
  age: 25,
  
  // Regular method
  greet: function() {
    console.log(`Hello, I'm ${this.name}`);
  },
  
  // Shorthand method
  getAge() {
    return this.age;
  },
  
  // Nested object
  address: {
    city: 'New York',
    getCity: function() {
      return this.city;  // 'this' refers to 'address', not 'user'
    }
  }
};

user.greet();         // 'Hello, I'm Alice' (this = user)
console.log(user.getAge());        // 25 (this = user)
console.log(user.address.getCity()); // 'New York' (this = address)

// Context is determined by call site
const greetFunc = user.greet;
greetFunc();  // 'Hello, I'm undefined' (lost context, this = global/undefined)
```

#### **3. Constructor Function Context**

```javascript
function Car(make, model) {
  // 'this' refers to the newly created object
  this.make = make;
  this.model = model;
  
  this.getInfo = function() {
    return `${this.make} ${this.model}`;
  };
}

const car1 = new Car('Toyota', 'Camry');
const car2 = new Car('Honda', 'Civic');

console.log(car1.getInfo());  // 'Toyota Camry' (this = car1)
console.log(car2.getInfo());  // 'Honda Civic' (this = car2)

// What 'new' does:
// 1. Creates a new empty object
// 2. Sets the object's prototype
// 3. Binds 'this' to the new object
// 4. Executes the constructor function
// 5. Returns the new object (unless constructor returns an object)

// Without 'new' - BAD!
const car3 = Car('Ford', 'Focus');  // Forgot 'new'
console.log(car3);         // undefined
console.log(window.make);  // 'Ford' (polluted global!)
```

#### **4. Class Context**

```javascript
class Person {
  constructor(name, age) {
    this.name = name;  // 'this' = new instance
    this.age = age;
  }
  
  // Instance method
  introduce() {
    console.log(`I'm ${this.name}, ${this.age} years old`);
  }
  
  // Static method
  static species() {
    console.log(this);  // 'this' = Person class itself
    return 'Homo sapiens';
  }
  
  // Getter
  get info() {
    return `${this.name} (${this.age})`;
  }
  
  // Setter
  set info(value) {
    const [name, age] = value.split(',');
    this.name = name.trim();
    this.age = parseInt(age);
  }
}

const person = new Person('John', 30);
person.introduce();  // 'this' = person instance

console.log(Person.species());  // 'this' = Person class
```

#### **5. Arrow Function Context (Lexical Binding)**

```javascript
// Arrow functions DON'T have their own 'this'
// They inherit 'this' from the enclosing scope

const obj = {
  name: 'Object',
  
  // Regular function
  regularMethod: function() {
    console.log('Regular:', this.name);  // 'Object'
    
    // Problem: nested function loses context
    function inner() {
      console.log('Inner regular:', this.name);  // undefined
    }
    inner();
    
    // Solution: arrow function
    const innerArrow = () => {
      console.log('Inner arrow:', this.name);  // 'Object' (lexical this)
    };
    innerArrow();
  },
  
  // Arrow function as method - PROBLEM!
  arrowMethod: () => {
    console.log('Arrow method:', this.name);  // undefined (lexical from global)
  }
};

obj.regularMethod();
obj.arrowMethod();  // ❌ 'this' is NOT obj

// Practical example: setTimeout
const counter = {
  count: 0,
  
  // Regular function
  incrementRegular: function() {
    setTimeout(function() {
      this.count++;  // ❌ 'this' is global/undefined, not counter
      console.log(this.count);
    }, 100);
  },
  
  // Arrow function solution
  incrementArrow: function() {
    setTimeout(() => {
      this.count++;  // ✅ 'this' is counter (lexical)
      console.log(this.count);
    }, 100);
  }
};

counter.incrementRegular();  // NaN
counter.incrementArrow();    // 1
```

#### **6. Explicit Binding (call, apply, bind)**

```javascript
function greet(greeting, punctuation) {
  return `${greeting}, ${this.name}${punctuation}`;
}

const person1 = { name: 'Alice' };
const person2 = { name: 'Bob' };

// call() - invoke with explicit 'this' and individual args
console.log(greet.call(person1, 'Hello', '!'));  // 'Hello, Alice!'
console.log(greet.call(person2, 'Hi', '.'));     // 'Hi, Bob.'

// apply() - invoke with explicit 'this' and array of args
console.log(greet.apply(person1, ['Hey', '?']));  // 'Hey, Alice?'
console.log(greet.apply(person2, ['Yo', '!']));   // 'Yo, Bob!'

// bind() - create new function with bound 'this'
const greetAlice = greet.bind(person1);
console.log(greetAlice('Greetings', '.'));  // 'Greetings, Alice.'

const greetBob = greet.bind(person2, 'Welcome');  // Partial application
console.log(greetBob('!'));  // 'Welcome, Bob!'

// Bind multiple times (first bind wins)
const bound1 = greet.bind(person1);
const bound2 = bound1.bind(person2);  // Doesn't change binding
console.log(bound2('Hi', '!'));  // 'Hi, Alice!' (still person1)
```

#### **7. Event Handler Context**

```javascript
const button = document.querySelector('#myButton');

// Regular function - 'this' is the element
button.addEventListener('click', function(event) {
  console.log(this);  // <button#myButton>
  console.log(this === event.target);  // true
  
  this.textContent = 'Clicked!';
  this.classList.add('active');
});

// Arrow function - 'this' is lexical (NOT the element)
button.addEventListener('click', (event) => {
  console.log(this);  // Window or outer scope, NOT button
  // this.textContent = 'Clicked!';  // ❌ Won't work
  
  // Use event.target instead
  event.target.textContent = 'Clicked!';  // ✅
});

// Class method as handler
class ButtonHandler {
  constructor(element) {
    this.element = element;
    this.clickCount = 0;
    
    // Need to bind to preserve class instance context
    this.element.addEventListener('click', this.handleClick.bind(this));
    
    // Or use arrow function property
    // this.element.addEventListener('click', this.handleClickArrow);
  }
  
  handleClick() {
    this.clickCount++;
    console.log(`Clicked ${this.clickCount} times`);
    this.element.textContent = `Clicks: ${this.clickCount}`;
  }
  
  // Arrow function property (automatically bound)
  handleClickArrow = () => {
    this.clickCount++;
    console.log(`Clicked ${this.clickCount} times`);
  }
}

const handler = new ButtonHandler(button);
```

#### **8. Array Methods Context**

```javascript
const numbers = [1, 2, 3, 4, 5];

// Array methods accept optional 'this' argument

const multiplier = {
  factor: 10,
  
  multiplyArray: function(arr) {
    // Without thisArg - 'this' is undefined in callback
    const result1 = arr.map(function(num) {
      return num * this.factor;  // ❌ this is undefined
    });
    
    // With thisArg - specify 'this' for callback
    const result2 = arr.map(function(num) {
      return num * this.factor;  // ✅ this is multiplier
    }, this);  // Pass 'this' as second argument
    
    // Or use arrow function (lexical this)
    const result3 = arr.map(num => num * this.factor);  // ✅
    
    return result3;
  }
};

console.log(multiplier.multiplyArray(numbers));  // [10, 20, 30, 40, 50]

// Another example
const person = {
  name: 'John',
  friends: ['Alice', 'Bob', 'Charlie'],
  
  greetFriends: function() {
    // Using arrow function
    this.friends.forEach(friend => {
      console.log(`${this.name} says hi to ${friend}`);
    });
    
    // Using thisArg
    this.friends.forEach(function(friend) {
      console.log(`${this.name} says hi to ${friend}`);
    }, this);
  }
};

person.greetFriends();
```

#### **9. Context in Callbacks**

```javascript
// Problem: callbacks lose context
const user = {
  name: 'Alice',
  friends: ['Bob', 'Charlie'],
  
  printFriends: function() {
    this.friends.forEach(function(friend) {
      // 'this' is undefined (strict) or global (non-strict)
      console.log(this.name + ' knows ' + friend);  // ❌
    });
  }
};

// Solutions:

// Solution 1: Save 'this' reference
const user1 = {
  name: 'Alice',
  friends: ['Bob', 'Charlie'],
  
  printFriends: function() {
    const self = this;  // Save reference
    this.friends.forEach(function(friend) {
      console.log(self.name + ' knows ' + friend);  // ✅
    });
  }
};

// Solution 2: Use bind
const user2 = {
  name: 'Alice',
  friends: ['Bob', 'Charlie'],
  
  printFriends: function() {
    this.friends.forEach(function(friend) {
      console.log(this.name + ' knows ' + friend);  // ✅
    }.bind(this));
  }
};

// Solution 3: Use arrow function
const user3 = {
  name: 'Alice',
  friends: ['Bob', 'Charlie'],
  
  printFriends: function() {
    this.friends.forEach(friend => {
      console.log(this.name + ' knows ' + friend);  // ✅
    });
  }
};

// Solution 4: Use thisArg parameter
const user4 = {
  name: 'Alice',
  friends: ['Bob', 'Charlie'],
  
  printFriends: function() {
    this.friends.forEach(function(friend) {
      console.log(this.name + ' knows ' + friend);  // ✅
    }, this);  // Pass 'this' as second argument
  }
};
```

#### **10. Context in Async Code**

```javascript
const asyncObject = {
  value: 42,
  
  // setTimeout loses context with regular function
  delayedLog1: function() {
    setTimeout(function() {
      console.log(this.value);  // ❌ undefined
    }, 1000);
  },
  
  // Fix with arrow function
  delayedLog2: function() {
    setTimeout(() => {
      console.log(this.value);  // ✅ 42
    }, 1000);
  },
  
  // Fix with bind
  delayedLog3: function() {
    setTimeout(function() {
      console.log(this.value);  // ✅ 42
    }.bind(this), 1000);
  },
  
  // async/await preserves 'this' in method
  async fetchData() {
    console.log(this.value);  // 42
    const data = await fetch('/api/data');
    console.log(this.value);  // Still 42 (async doesn't affect this)
    return data;
  },
  
  // Promise chain
  processData: function() {
    return fetch('/api/data')
      .then(response => response.json())
      .then(data => {
        // Arrow function preserves 'this'
        console.log(this.value, data);  // ✅
        return data;
      });
  }
};
```

#### **Binding Priority Rules**

```javascript
// Priority (highest to lowest):
// 1. new binding
// 2. Explicit binding (call, apply, bind)
// 3. Implicit binding (method call)
// 4. Default binding (global/undefined)

function test() {
  console.log(this.value);
}

const obj1 = { value: 1 };
const obj2 = { value: 2 };

// 1. New binding (highest)
function Constructor() {
  console.log(this);  // New object
}
new Constructor();  // 'this' is new instance

// 2. Explicit binding
test.call(obj1);  // 1 (explicit binding)

// 3. Implicit binding
obj2.test = test;
obj2.test();  // 2 (implicit binding)

// Priority test: explicit > implicit
const boundTest = test.bind(obj1);
obj2.boundTest = boundTest;
obj2.boundTest();  // 1 (explicit wins over implicit)

// new > bind
function Func() {
  this.value = 3;
}
const boundFunc = Func.bind(obj1);
const instance = new boundFunc();
console.log(instance.value);  // 3 (new wins, not obj1.value)
```

#### **Comparison Table**

| Context | 'this' Binding | Can be changed? | Example |
|---------|---------------|-----------------|---------|
| **Global function** | Global/undefined | Yes (call/apply/bind) | `function() { this }` |
| **Object method** | Parent object | Yes | `obj.method()` |
| **Constructor** | New instance | No | `new Func()` |
| **Arrow function** | Lexical scope | No | `() => this` |
| **call/apply** | First argument | N/A | `func.call(obj)` |
| **bind** | Bound object | No (permanent) | `func.bind(obj)` |
| **Event handler** | Target element | Yes | `element.addEventListener()` |
| **Strict mode** | undefined | Yes | `'use strict'; func()` |
| **Array method** | thisArg or undefined | Yes (thisArg) | `arr.forEach(fn, thisArg)` |

#### **Key Takeaways**

- **'this' depends on call site**, not definition site
- **Priority**: new > explicit > implicit > default
- **Arrow functions** use lexical 'this' (inherit from scope)
- **Regular functions** have dynamic 'this' (depends on caller)
- **Methods**: 'this' = object before dot
- **Constructors**: 'this' = new instance
- **Global**: 'this' = window/global (or undefined in strict)
- **call/apply**: set 'this' for one call
- **bind**: create permanently bound function
- **Event handlers**: 'this' = target element (regular functions)
- **Callbacks** often lose context (use arrow/bind)
- **Arrow functions** can't be bound with call/apply/bind
- **Save reference** with `self = this` pattern (old style)
- **Modern preference**: arrow functions for callbacks
- **Understanding context** prevents common bugs

---

### 81. What is strict mode?

**Answer:**

Strict mode is a way to opt into a restricted variant of JavaScript that eliminates some silent errors, fixes mistakes that make it difficult for JavaScript engines to optimize, and prohibits some syntax likely to be defined in future versions of ECMAScript. It's enabled by adding `'use strict';` at the beginning of a script or function.

#### **Enabling Strict Mode**

```javascript
// 1. Global strict mode (entire script)
'use strict';

let x = 10;
function myFunc() {
  // All code in this file is strict
}

// 2. Function strict mode (function scope only)
function strictFunc() {
  'use strict';
  // Only this function is strict
  let y = 20;
}

function normalFunc() {
  // This function is not strict
  z = 30;  // Creates global variable (sloppy mode)
}

// 3. Module strict mode (automatic in ES6 modules)
// In .mjs files or <script type="module">
// 'use strict' is implicit, no need to declare
export function moduleFunc() {
  // Automatically in strict mode
}

// 4. Class bodies (automatic)
class MyClass {
  // Class bodies are always in strict mode
  constructor() {
    // No need for 'use strict'
  }
}
```

#### **1. Prevents Accidental Globals**

```javascript
// Non-strict mode (sloppy mode)
function createGlobal() {
  accidentalGlobal = 'oops';  // Creates global variable
}
createGlobal();
console.log(window.accidentalGlobal);  // 'oops'

// Strict mode
'use strict';
function strictFunction() {
  accidentalGlobal = 'oops';  // ❌ ReferenceError: accidentalGlobal is not defined
}

// Must use var, let, or const
function correctWay() {
  'use strict';
  let properVariable = 'correct';  // ✅
  const another = 'also correct';  // ✅
}
```

#### **2. Assignments to Non-Writable Properties**

```javascript
'use strict';

// 1. Read-only properties
const obj = {};
Object.defineProperty(obj, 'readOnly', {
  value: 42,
  writable: false
});

obj.readOnly = 100;  // ❌ TypeError: Cannot assign to read only property

// 2. Non-extensible objects
const sealed = {};
Object.preventExtensions(sealed);
sealed.newProp = 'value';  // ❌ TypeError: Cannot add property

// 3. Getter-only properties
const obj2 = {
  get prop() {
    return 'value';
  }
};
obj2.prop = 'new value';  // ❌ TypeError: Cannot set property

// Non-strict mode: All above fail silently (no error)
```

#### **3. Deleting Variables, Functions, and Non-Configurable Properties**

```javascript
'use strict';

// Can't delete variables
let x = 10;
delete x;  // ❌ SyntaxError: Delete of an unqualified identifier

// Can't delete functions
function myFunc() {}
delete myFunc;  // ❌ SyntaxError

// Can't delete non-configurable properties
delete Object.prototype;  // ❌ TypeError

// CAN delete object properties (if configurable)
const obj = { prop: 'value' };
delete obj.prop;  // ✅ Works fine

// Non-strict mode: delete returns false silently
```

#### **4. Duplicate Parameter Names**

```javascript
// Non-strict mode: Last parameter wins
function sum(a, a, c) {
  return a + a + c;  // Uses last 'a'
}
console.log(sum(1, 2, 3));  // 7 (2 + 2 + 3)

// Strict mode: Syntax error
'use strict';
function strictSum(a, a, c) {  // ❌ SyntaxError: Duplicate parameter name
  return a + a + c;
}
```

#### **5. Octal Syntax**

```javascript
// Non-strict mode: Octal literals allowed
const octal1 = 010;  // 8 in decimal
const octal2 = 077;  // 63 in decimal

// Strict mode: Octal literals forbidden
'use strict';
const strictOctal = 010;  // ❌ SyntaxError: Octal literals are not allowed

// Use explicit octal notation
const modernOctal = 0o10;  // ✅ 8 in decimal (ES6+)
const binary = 0b1010;     // ✅ 10 in decimal
const hex = 0xFF;          // ✅ 255 in decimal
```

#### **6. 'this' in Functions**

```javascript
// Non-strict mode: 'this' is global object
function showThis() {
  console.log(this);
}
showThis();  // Window (browser) or global (Node.js)

// Strict mode: 'this' is undefined
'use strict';
function strictThis() {
  console.log(this);
}
strictThis();  // undefined

// Method calls still work
const obj = {
  method: function() {
    'use strict';
    console.log(this);  // obj (not affected)
  }
};
obj.method();  // obj

// Important for callbacks
function callback() {
  'use strict';
  console.log(this);  // undefined (not global)
}
setTimeout(callback, 100);
```

#### **7. Reserved Words**

```javascript
'use strict';

// These are reserved for future use
// ❌ SyntaxError in strict mode:
const implements = 10;
const interface = 20;
const let = 30;  // 'let' is now a keyword anyway
const package = 40;
const private = 50;
const protected = 60;
const public = 70;
const static = 80;  // 'static' is now a keyword
const yield = 90;   // 'yield' is now a keyword

// Also reserved:
// arguments, eval (can't be variable names or parameters)
```

#### **8. Arguments Object Restrictions**

```javascript
// Non-strict mode: arguments reflects parameter changes
function nonStrict(a) {
  a = 100;
  console.log(arguments[0]);  // 100 (linked to parameter)
}
nonStrict(1);

// Strict mode: arguments is independent
'use strict';
function strictFunc(a) {
  a = 100;
  console.log(arguments[0]);  // 1 (not linked)
}
strictFunc(1);

// Can't assign to arguments
'use strict';
function test() {
  arguments = [1, 2, 3];  // ❌ SyntaxError: Unexpected eval or arguments
}

// Can't use arguments.caller or arguments.callee
'use strict';
function factorial(n) {
  return n <= 1 ? 1 : n * arguments.callee(n - 1);  // ❌ TypeError
}
```

#### **9. eval() Restrictions**

```javascript
// Non-strict mode: eval can create variables in scope
function nonStrictEval() {
  eval('var x = 10');
  console.log(x);  // 10 (x created in function scope)
}

// Strict mode: eval has its own scope
'use strict';
function strictEval() {
  eval('var x = 10');
  console.log(x);  // ❌ ReferenceError: x is not defined
}

// Can't use 'eval' as variable name
'use strict';
const eval = 10;  // ❌ SyntaxError
function test(eval) {}  // ❌ SyntaxError
```

#### **10. with Statement Forbidden**

```javascript
// Non-strict mode: 'with' is allowed (but discouraged)
const obj = { x: 10, y: 20 };
with (obj) {
  console.log(x);  // 10
  console.log(y);  // 20
}

// Strict mode: 'with' is forbidden
'use strict';
with (obj) {  // ❌ SyntaxError: Strict mode code may not include a with statement
  console.log(x);
}

// Alternative: destructuring
const { x, y } = obj;
console.log(x, y);  // 10, 20
```

#### **Practical Examples**

**1. Preventing Typos:**
```javascript
'use strict';

const user = {
  firstName: 'John',
  lastName: 'Doe'
};

// Typo in property name
user.fistName = 'Jane';  // Creates new property (no error)

// But typo in variable name
fistName = 'Jane';  // ❌ ReferenceError (caught by strict mode)

// Should be:
user.firstName = 'Jane';  // ✅
```

**2. Safer this Binding:**
```javascript
'use strict';

class Button {
  constructor(element) {
    this.element = element;
    this.clickCount = 0;
    
    // Without bind, 'this' would be undefined in strict mode
    this.element.addEventListener('click', this.handleClick.bind(this));
  }
  
  handleClick() {
    // In strict mode, 'this' is undefined if not bound
    // Forces proper binding practices
    this.clickCount++;
    console.log(`Clicks: ${this.clickCount}`);
  }
}

const btn = new Button(document.querySelector('button'));
```

**3. Module Pattern:**
```javascript
'use strict';

const Module = (function() {
  // Private variables
  let privateVar = 0;
  
  function privateFunction() {
    privateVar++;
  }
  
  // Public API
  return {
    publicMethod: function() {
      privateFunction();
      return privateVar;
    },
    
    reset: function() {
      privateVar = 0;
    }
  };
})();

console.log(Module.publicMethod());  // 1
console.log(Module.privateVar);      // undefined (truly private)
```

**4. Constructor Safety:**
```javascript
'use strict';

function Person(name) {
  // In strict mode, forgetting 'new' doesn't pollute global
  this.name = name;  // TypeError if called without 'new'
}

const person1 = new Person('John');  // ✅ Works
const person2 = Person('Jane');      // ❌ TypeError: Cannot set property 'name' of undefined

// ES6 classes are always strict (safe by default)
class SafePerson {
  constructor(name) {
    this.name = name;
  }
}

const person3 = new SafePerson('Bob');  // ✅
const person4 = SafePerson('Alice');    // ❌ TypeError: Class constructor cannot be invoked without 'new'
```

#### **Benefits of Strict Mode**

```javascript
'use strict';

// 1. Catches common coding mistakes
function benefits() {
  // Typo creates global (non-strict)
  // ReferenceError in strict ✅
  
  // Duplicate params (non-strict ignores)
  // SyntaxError in strict ✅
  
  // Delete non-deletable (non-strict fails silently)
  // TypeError in strict ✅
}

// 2. Prevents unsafe actions
function safetyFeatures() {
  // Can't mess with eval scope
  // Can't use 'with' statement
  // Can't access caller/callee
}

// 3. Enables optimizations
function performance() {
  // Engines can optimize better
  // 'arguments' doesn't track parameters
  // No need to check for 'with' statement
  // 'this' boxing not needed
}

// 4. Future-proofs code
function futureProof() {
  // Reserved words enforced
  // Syntax that may change is forbidden
  // Prepares for future ECMAScript versions
}
```

#### **Mixing Strict and Non-Strict**

```javascript
// Global non-strict
function nonStrictFunc() {
  // This function is non-strict
  implicitGlobal = 'allowed';
}

function strictFunc() {
  'use strict';
  // This function is strict
  // implicitGlobal = 'error';  // ❌ ReferenceError
}

// Can use both in same file
nonStrictFunc();  // Creates global
strictFunc();     // Would error if uncommented

// Concatenation warning
// File 1 (strict): 'use strict'; ...
// File 2 (non-strict): ...
// If concatenated, File 2 becomes strict!
```

#### **Common Pitfalls**

```javascript
// 1. Forgetting 'use strict' in new code
function oldCode() {
  // Sloppy mode
  mistake = 'oops';  // Creates global
}

// 2. Using strict mode with old libraries
'use strict';
// oldLibrary.js might not work in strict mode

// 3. Concatenated files
// Strict mode in one file affects concatenated files

// 4. Function inside non-strict
function outer() {
  // Non-strict
  function inner() {
    'use strict';
    // Only inner is strict
  }
}
```

#### **Best Practices**

```javascript
// 1. Always use strict mode in new code
'use strict';

// 2. Use ES6 modules (automatically strict)
// module.js
export function myFunc() {
  // Automatically strict
}

// 3. Use modern classes (automatically strict)
class MyClass {
  // Automatically strict
}

// 4. Enable in build tools
// ESLint: "strict": ["error", "global"]
// Babel: transforms to strict mode

// 5. Use linters to catch issues
// ESLint will warn about non-strict issues
```

#### **Key Takeaways**

- **'use strict'** enables strict mode
- **Prevents accidental globals** (undeclared variables error)
- **Makes 'this' undefined** in functions (not global)
- **Catches silent errors** (assignments to read-only, deletes)
- **Forbids duplicate** parameter names
- **Restricts eval()** to its own scope
- **Bans 'with' statement** completely
- **Reserves future keywords** (implements, interface, etc.)
- **Makes arguments independent** of parameters
- **Can't delete** variables, functions, non-configurable properties
- **ES6 modules** are automatically strict
- **Classes** are automatically strict
- **Better optimization** by engines
- **Use globally** in new projects
- **Modern JavaScript** assumes strict mode

79. What is the `this` keyword?
80. How does `this` work in different contexts?
81. What is strict mode?

---

### 82. What is memoization?

**Answer:**

Memoization is an optimization technique that caches the results of expensive function calls and returns the cached result when the same inputs occur again. It trades memory for speed by storing previously computed values, avoiding redundant calculations. Memoization is particularly effective for pure functions with deterministic outputs and recursive algorithms.

#### **Basic Concept**

```javascript
// Without memoization - slow
function slowFibonacci(n) {
  if (n <= 1) return n;
  return slowFibonacci(n - 1) + slowFibonacci(n - 2);
}

console.time('slow');
console.log(slowFibonacci(40));  // Takes several seconds
console.timeEnd('slow');

// With memoization - fast
function memoizedFibonacci() {
  const cache = {};
  
  return function fib(n) {
    if (n in cache) {
      return cache[n];  // Return cached result
    }
    
    if (n <= 1) {
      return n;
    }
    
    // Calculate and cache
    cache[n] = fib(n - 1) + fib(n - 2);
    return cache[n];
  };
}

const fastFib = memoizedFibonacci();
console.time('fast');
console.log(fastFib(40));  // Nearly instant
console.timeEnd('fast');
```

#### **1. Simple Memoization Function**

```javascript
// Generic memoization function
function memoize(fn) {
  const cache = {};
  
  return function(...args) {
    const key = JSON.stringify(args);
    
    if (key in cache) {
      console.log('Returning from cache:', key);
      return cache[key];
    }
    
    console.log('Computing result for:', key);
    const result = fn.apply(this, args);
    cache[key] = result;
    return result;
  };
}

// Example: expensive calculation
function expensiveCalculation(n) {
  let sum = 0;
  for (let i = 0; i < 1000000; i++) {
    sum += n * i;
  }
  return sum;
}

const memoizedCalc = memoize(expensiveCalculation);

console.log(memoizedCalc(5));  // Computing... (slow)
console.log(memoizedCalc(5));  // From cache (instant)
console.log(memoizedCalc(10)); // Computing... (slow)
console.log(memoizedCalc(5));  // From cache (instant)
```

#### **2. Fibonacci with Memoization**

```javascript
// Recursive approach with memoization
function fibonacciMemo() {
  const cache = {};
  
  function fib(n) {
    if (n in cache) return cache[n];
    
    if (n <= 1) {
      return n;
    }
    
    cache[n] = fib(n - 1) + fib(n - 2);
    return cache[n];
  }
  
  return fib;
}

const fibonacci = fibonacciMemo();

console.log(fibonacci(10));  // 55
console.log(fibonacci(50));  // 12586269025 (instant)
console.log(fibonacci(100)); // Very large number (still instant)

// Alternative: using closure directly
const fib = (function() {
  const cache = {};
  
  return function fibonacci(n) {
    if (n in cache) return cache[n];
    if (n <= 1) return n;
    
    return cache[n] = fibonacci(n - 1) + fibonacci(n - 2);
  };
})();
```

#### **3. Factorial with Memoization**

```javascript
const factorial = (function() {
  const cache = { 0: 1, 1: 1 };
  
  return function fact(n) {
    if (n in cache) return cache[n];
    
    return cache[n] = n * fact(n - 1);
  };
})();

console.log(factorial(5));   // 120
console.log(factorial(10));  // 3628800
console.log(factorial(20));  // 2432902008176640000
console.log(factorial(15));  // Uses cached values from 20!
```

#### **4. Advanced Memoization with Custom Key**

```javascript
// Memoization with custom key generation
function memoizeWithKey(fn, keyGenerator) {
  const cache = new Map();
  
  return function(...args) {
    const key = keyGenerator ? keyGenerator(...args) : args[0];
    
    if (cache.has(key)) {
      return cache.get(key);
    }
    
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}

// Example: Sum of array
function sumArray(arr) {
  return arr.reduce((sum, num) => sum + num, 0);
}

const memoizedSum = memoizeWithKey(sumArray, arr => arr.join(','));

console.log(memoizedSum([1, 2, 3]));  // Computing: 6
console.log(memoizedSum([1, 2, 3]));  // From cache: 6
console.log(memoizedSum([4, 5, 6]));  // Computing: 15
```

#### **5. Memoization with Expiration**

```javascript
// Memoization with TTL (Time To Live)
function memoizeWithTTL(fn, ttl = 5000) {
  const cache = new Map();
  
  return function(...args) {
    const key = JSON.stringify(args);
    const now = Date.now();
    
    if (cache.has(key)) {
      const { value, timestamp } = cache.get(key);
      
      if (now - timestamp < ttl) {
        console.log('Cache hit (fresh)');
        return value;
      } else {
        console.log('Cache expired');
        cache.delete(key);
      }
    }
    
    console.log('Computing result');
    const result = fn.apply(this, args);
    cache.set(key, { value: result, timestamp: now });
    return result;
  };
}

function fetchUserData(userId) {
  // Simulate API call
  return { id: userId, name: `User ${userId}`, timestamp: Date.now() };
}

const memoizedFetch = memoizeWithTTL(fetchUserData, 3000);

console.log(memoizedFetch(1));  // Computing
console.log(memoizedFetch(1));  // Cache hit
setTimeout(() => {
  console.log(memoizedFetch(1));  // Cache expired, recompute
}, 3500);
```

#### **6. Memoization with Size Limit (LRU Cache)**

```javascript
// LRU (Least Recently Used) cache
function memoizeWithLimit(fn, limit = 100) {
  const cache = new Map();
  
  return function(...args) {
    const key = JSON.stringify(args);
    
    // Move to end if exists (mark as recently used)
    if (cache.has(key)) {
      const value = cache.get(key);
      cache.delete(key);
      cache.set(key, value);
      return value;
    }
    
    // Compute result
    const result = fn.apply(this, args);
    
    // Remove oldest if at limit
    if (cache.size >= limit) {
      const firstKey = cache.keys().next().value;
      cache.delete(firstKey);
    }
    
    cache.set(key, result);
    return result;
  };
}

const memoizedCalc = memoizeWithLimit((x, y) => x * y, 3);

console.log(memoizedCalc(2, 3));  // 6
console.log(memoizedCalc(4, 5));  // 20
console.log(memoizedCalc(6, 7));  // 42
console.log(memoizedCalc(8, 9));  // 72 (evicts first entry)
console.log(memoizedCalc(2, 3));  // Recomputes (was evicted)
```

#### **7. Class Method Memoization**

```javascript
class Calculator {
  constructor() {
    // Memoize instance method
    this.expensiveOperation = this.memoize(this.expensiveOperation);
  }
  
  memoize(fn) {
    const cache = new Map();
    
    return function(...args) {
      const key = JSON.stringify(args);
      
      if (cache.has(key)) {
        return cache.get(key);
      }
      
      const result = fn.apply(this, args);
      cache.set(key, result);
      return result;
    };
  }
  
  expensiveOperation(n) {
    console.log(`Computing for ${n}...`);
    let result = 0;
    for (let i = 0; i < n * 1000000; i++) {
      result += i;
    }
    return result;
  }
}

const calc = new Calculator();
console.log(calc.expensiveOperation(10));  // Computing...
console.log(calc.expensiveOperation(10));  // From cache
```

#### **8. Decorator Pattern for Memoization**

```javascript
// Memoization decorator (ES7+ proposal)
function memoized(target, name, descriptor) {
  const original = descriptor.value;
  const cache = new Map();
  
  descriptor.value = function(...args) {
    const key = JSON.stringify(args);
    
    if (cache.has(key)) {
      return cache.get(key);
    }
    
    const result = original.apply(this, args);
    cache.set(key, result);
    return result;
  };
  
  return descriptor;
}

class MathOperations {
  @memoized
  factorial(n) {
    if (n <= 1) return 1;
    return n * this.factorial(n - 1);
  }
  
  @memoized
  fibonacci(n) {
    if (n <= 1) return n;
    return this.fibonacci(n - 1) + this.fibonacci(n - 2);
  }
}

// Without decorator support, use manual approach:
class MathOps {
  constructor() {
    this.factorialCache = new Map();
  }
  
  factorial(n) {
    if (this.factorialCache.has(n)) {
      return this.factorialCache.get(n);
    }
    
    const result = n <= 1 ? 1 : n * this.factorial(n - 1);
    this.factorialCache.set(n, result);
    return result;
  }
}
```

#### **Practical Examples**

**1. API Call Memoization:**
```javascript
function memoizeAsync(fn, ttl = 60000) {
  const cache = new Map();
  
  return async function(...args) {
    const key = JSON.stringify(args);
    const now = Date.now();
    
    if (cache.has(key)) {
      const { value, timestamp } = cache.get(key);
      if (now - timestamp < ttl) {
        console.log('Returning cached data');
        return value;
      }
      cache.delete(key);
    }
    
    console.log('Fetching fresh data');
    const result = await fn.apply(this, args);
    cache.set(key, { value: result, timestamp: now });
    return result;
  };
}

async function fetchUserProfile(userId) {
  const response = await fetch(`/api/users/${userId}`);
  return response.json();
}

const memoizedFetchUser = memoizeAsync(fetchUserProfile, 30000);

// First call - fetches from API
await memoizedFetchUser(123);

// Second call within 30s - returns cached
await memoizedFetchUser(123);

// After 30s - fetches again
```

**2. Complex Calculations:**
```javascript
// Memoized prime number checker
const isPrime = (function() {
  const cache = new Map();
  
  return function(num) {
    if (cache.has(num)) return cache.get(num);
    
    if (num <= 1) return cache.set(num, false).get(num);
    if (num === 2) return cache.set(num, true).get(num);
    if (num % 2 === 0) return cache.set(num, false).get(num);
    
    for (let i = 3; i <= Math.sqrt(num); i += 2) {
      if (num % i === 0) {
        return cache.set(num, false).get(num);
      }
    }
    
    return cache.set(num, true).get(num);
  };
})();

console.log(isPrime(17));    // true (computed)
console.log(isPrime(17));    // true (cached)
console.log(isPrime(97));    // true (computed)
console.log(isPrime(100));   // false (computed)
```

**3. React-like Component Optimization:**
```javascript
// Memoize expensive render calculations
function createComponent() {
  const renderCache = new Map();
  
  return {
    render(props) {
      const key = JSON.stringify(props);
      
      if (renderCache.has(key)) {
        console.log('Using cached render');
        return renderCache.get(key);
      }
      
      console.log('Computing render');
      // Expensive render calculation
      const rendered = this.expensiveRender(props);
      renderCache.set(key, rendered);
      return rendered;
    },
    
    expensiveRender(props) {
      // Complex calculations
      let result = '';
      for (let i = 0; i < 10000; i++) {
        result += props.text;
      }
      return result;
    }
  };
}

const component = createComponent();
component.render({ text: 'Hello' });  // Computing
component.render({ text: 'Hello' });  // Cached
component.render({ text: 'World' });  // Computing (different props)
```

**4. Search/Filter Memoization:**
```javascript
const searchService = {
  cache: new Map(),
  
  search(query, data) {
    const key = `${query}:${data.length}`;
    
    if (this.cache.has(key)) {
      console.log('Cache hit for:', query);
      return this.cache.get(key);
    }
    
    console.log('Searching for:', query);
    const results = data.filter(item => 
      item.toLowerCase().includes(query.toLowerCase())
    );
    
    this.cache.set(key, results);
    return results;
  },
  
  clearCache() {
    this.cache.clear();
  }
};

const data = ['Apple', 'Banana', 'Cherry', 'Date', 'Elderberry'];

console.log(searchService.search('a', data));  // Searching...
console.log(searchService.search('a', data));  // Cache hit
console.log(searchService.search('e', data));  // Searching...
```

**5. Route Path Matching:**
```javascript
function createRouter() {
  const routeCache = new Map();
  
  return {
    matchRoute(path, routes) {
      if (routeCache.has(path)) {
        return routeCache.get(path);
      }
      
      for (const route of routes) {
        const regex = new RegExp(route.pattern);
        if (regex.test(path)) {
          routeCache.set(path, route);
          return route;
        }
      }
      
      return null;
    }
  };
}

const router = createRouter();
const routes = [
  { pattern: '^/users/\\d+$', handler: 'userHandler' },
  { pattern: '^/posts/\\d+$', handler: 'postHandler' }
];

console.log(router.matchRoute('/users/123', routes));  // Computed
console.log(router.matchRoute('/users/123', routes));  // Cached
```

#### **When to Use Memoization**

```javascript
// ✅ GOOD USE CASES:

// 1. Pure functions (deterministic)
const multiply = memoize((a, b) => a * b);

// 2. Expensive computations
const fibonacci = memoize(n => {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
});

// 3. Repeated API calls
const getUser = memoizeAsync(async id => {
  return fetch(`/api/users/${id}`).then(r => r.json());
});

// 4. Recursive algorithms
const factorialMemo = memoize(n => {
  if (n <= 1) return 1;
  return n * factorialMemo(n - 1);
});

// ❌ BAD USE CASES:

// 1. Impure functions (side effects)
const badExample1 = memoize(() => {
  console.log('Side effect');  // Side effect!
  return Math.random();        // Non-deterministic!
});

// 2. Functions with different results for same input
const badExample2 = memoize(() => new Date());  // Always different

// 3. Functions that modify arguments
const badExample3 = memoize(arr => {
  arr.push(1);  // Mutates input!
  return arr;
});

// 4. Fast, simple operations
const badExample4 = memoize((a, b) => a + b);  // Too simple, overhead not worth it
```

#### **Performance Considerations**

```javascript
// Benchmark: With vs Without Memoization
function benchmark() {
  // Without memoization
  function slowFib(n) {
    if (n <= 1) return n;
    return slowFib(n - 1) + slowFib(n - 2);
  }
  
  // With memoization
  const fastFib = (function() {
    const cache = {};
    return function fib(n) {
      if (n in cache) return cache[n];
      if (n <= 1) return n;
      return cache[n] = fib(n - 1) + fib(n - 2);
    };
  })();
  
  // Test
  console.time('Without memoization');
  console.log(slowFib(35));  // ~5-10 seconds
  console.timeEnd('Without memoization');
  
  console.time('With memoization');
  console.log(fastFib(35));  // ~1ms
  console.timeEnd('With memoization');
  
  // Fibonacci(35) without memo: ~59 million recursive calls
  // Fibonacci(35) with memo: 35 calls
}
```

#### **Common Patterns**

```javascript
// Pattern 1: IIFE with closure
const memoFunc1 = (function() {
  const cache = {};
  return function(n) {
    return cache[n] || (cache[n] = expensiveCalc(n));
  };
})();

// Pattern 2: Higher-order function
const memoFunc2 = memoize(expensiveCalc);

// Pattern 3: Class with private cache
class MemoService {
  #cache = new Map();
  
  compute(input) {
    if (this.#cache.has(input)) {
      return this.#cache.get(input);
    }
    const result = this.expensiveCalc(input);
    this.#cache.set(input, result);
    return result;
  }
  
  expensiveCalc(input) {
    // Heavy computation
  }
}

// Pattern 4: WeakMap for object keys
const memoFunc4 = (function() {
  const cache = new WeakMap();
  
  return function(obj) {
    if (cache.has(obj)) return cache.get(obj);
    
    const result = processObject(obj);
    cache.set(obj, result);
    return result;
  };
})();
```

#### **Key Takeaways**

- **Memoization** caches function results based on inputs
- **Trade memory for speed** - stores results to avoid recalculation
- **Best for pure functions** with deterministic outputs
- **Excellent for recursive** algorithms (Fibonacci, factorial)
- **Use Map/Object** for simple caching
- **Add TTL** for time-sensitive data
- **Implement LRU** to limit memory usage
- **Key generation** is critical for complex arguments
- **Avoid memoizing** impure functions or side effects
- **Performance gain** is significant for expensive operations
- **Not suitable** for simple, fast operations
- **Clear cache** when data changes
- **Async memoization** works for API calls
- **WeakMap** for object keys (allows garbage collection)
- **Understand trade-offs** between memory and computation

---

### 83. What is debouncing?

**Answer:**

Debouncing is a programming technique that limits the rate at which a function can execute. It delays function execution until after a certain amount of time has passed since the last time it was invoked. If the function is called again before the delay expires, the timer resets. This is particularly useful for handling events that fire rapidly, like scroll, resize, or input events.

#### **Basic Concept**

```javascript
// Without debouncing - function fires on every keystroke
function handleSearch(query) {
  console.log('Searching for:', query);
  // API call or expensive operation
}

// input.addEventListener('keyup', (e) => {
//   handleSearch(e.target.value);  // Fires hundreds of times!
// });

// With debouncing - function fires once user stops typing
function debounce(func, delay) {
  let timeoutId;
  
  return function(...args) {
    // Clear previous timer
    clearTimeout(timeoutId);
    
    // Set new timer
    timeoutId = setTimeout(() => {
      func.apply(this, args);
    }, delay);
  };
}

const debouncedSearch = debounce(handleSearch, 500);

// input.addEventListener('keyup', (e) => {
//   debouncedSearch(e.target.value);  // Fires once after 500ms of inactivity
// });
```

#### **1. Simple Debounce Implementation**

```javascript
function debounce(func, delay) {
  let timeoutId;
  
  return function(...args) {
    const context = this;
    
    // Clear existing timer
    clearTimeout(timeoutId);
    
    // Set new timer
    timeoutId = setTimeout(() => {
      func.apply(context, args);
    }, delay);
  };
}

// Usage
function logMessage(message) {
  console.log('Message:', message);
}

const debouncedLog = debounce(logMessage, 1000);

debouncedLog('Hello');   // Timer starts
debouncedLog('World');   // Timer resets
debouncedLog('!');       // Timer resets
// After 1 second of no calls: "Message: !"
```

#### **2. Debounce with Immediate Execution**

```javascript
// Execute immediately on first call, then debounce
function debounce(func, delay, immediate = false) {
  let timeoutId;
  
  return function(...args) {
    const context = this;
    const callNow = immediate && !timeoutId;
    
    clearTimeout(timeoutId);
    
    timeoutId = setTimeout(() => {
      timeoutId = null;
      if (!immediate) {
        func.apply(context, args);
      }
    }, delay);
    
    if (callNow) {
      func.apply(context, args);
    }
  };
}

// Usage
const debouncedClick = debounce(() => {
  console.log('Button clicked');
}, 1000, true);

// First click executes immediately
// Subsequent clicks within 1s are ignored
// After 1s, next click executes immediately again
```

#### **3. Debounce with Cancel**

```javascript
function debounce(func, delay) {
  let timeoutId;
  
  const debounced = function(...args) {
    clearTimeout(timeoutId);
    
    timeoutId = setTimeout(() => {
      func.apply(this, args);
    }, delay);
  };
  
  // Add cancel method
  debounced.cancel = function() {
    clearTimeout(timeoutId);
    timeoutId = null;
  };
  
  // Add flush method (execute immediately)
  debounced.flush = function() {
    if (timeoutId) {
      clearTimeout(timeoutId);
      func.apply(this);
    }
  };
  
  return debounced;
}

// Usage
const debouncedSave = debounce(() => {
  console.log('Saving...');
}, 2000);

debouncedSave();  // Starts timer
debouncedSave();  // Resets timer

// Cancel if needed
debouncedSave.cancel();  // Cancels pending execution

// Or execute immediately
debouncedSave.flush();   // Executes right now
```

#### **4. Promise-based Debounce**

```javascript
function debouncePromise(func, delay) {
  let timeoutId;
  let rejectPrevious = null;
  
  return function(...args) {
    return new Promise((resolve, reject) => {
      // Reject previous promise
      if (rejectPrevious) {
        rejectPrevious('Debounced');
      }
      
      rejectPrevious = reject;
      clearTimeout(timeoutId);
      
      timeoutId = setTimeout(async () => {
        rejectPrevious = null;
        try {
          const result = await func.apply(this, args);
          resolve(result);
        } catch (error) {
          reject(error);
        }
      }, delay);
    });
  };
}

// Usage with async function
async function searchAPI(query) {
  const response = await fetch(`/api/search?q=${query}`);
  return response.json();
}

const debouncedSearch = debouncePromise(searchAPI, 500);

// Only the last call resolves
debouncedSearch('abc').catch(err => console.log(err));  // "Debounced"
debouncedSearch('abcd').catch(err => console.log(err)); // "Debounced"
debouncedSearch('abcde').then(data => console.log(data)); // Resolves
```

#### **Practical Examples**

**1. Search Input:**
```javascript
// Search as user types
const searchInput = document.getElementById('search');
const resultsDiv = document.getElementById('results');

const performSearch = debounce(async function(query) {
  if (!query.trim()) {
    resultsDiv.innerHTML = '';
    return;
  }
  
  try {
    resultsDiv.innerHTML = 'Searching...';
    const response = await fetch(`/api/search?q=${encodeURIComponent(query)}`);
    const results = await response.json();
    
    resultsDiv.innerHTML = results.map(item => 
      `<div class="result">${item.title}</div>`
    ).join('');
  } catch (error) {
    resultsDiv.innerHTML = 'Error occurred';
  }
}, 300);

searchInput.addEventListener('input', (e) => {
  performSearch(e.target.value);
});

// User types "JavaScript"
// J - timer starts (300ms)
// a - timer resets (300ms)
// v - timer resets (300ms)
// ... continues for each letter
// After 300ms of no typing: API call executes once
```

**2. Window Resize:**
```javascript
// Recalculate layout on window resize
function recalculateLayout() {
  const width = window.innerWidth;
  const height = window.innerHeight;
  
  console.log('Recalculating layout:', width, height);
  
  // Expensive layout calculations
  document.querySelectorAll('.card').forEach(card => {
    card.style.width = width < 768 ? '100%' : '33.33%';
  });
}

const debouncedResize = debounce(recalculateLayout, 250);

window.addEventListener('resize', debouncedResize);

// Without debouncing: fires 100+ times during resize
// With debouncing: fires once after user stops resizing
```

**3. Auto-save Feature:**
```javascript
class Editor {
  constructor() {
    this.content = '';
    this.saveStatus = document.getElementById('save-status');
    
    // Debounced save function
    this.debouncedSave = debounce(() => this.saveContent(), 2000);
  }
  
  handleInput(value) {
    this.content = value;
    this.saveStatus.textContent = 'Unsaved changes...';
    this.debouncedSave();
  }
  
  async saveContent() {
    this.saveStatus.textContent = 'Saving...';
    
    try {
      await fetch('/api/save', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ content: this.content })
      });
      
      this.saveStatus.textContent = 'Saved ✓';
    } catch (error) {
      this.saveStatus.textContent = 'Save failed';
    }
  }
}

const editor = new Editor();
const textarea = document.getElementById('editor');

textarea.addEventListener('input', (e) => {
  editor.handleInput(e.target.value);
});
```

**4. Form Validation:**
```javascript
// Validate email as user types
const emailInput = document.getElementById('email');
const errorDiv = document.getElementById('email-error');

const validateEmail = debounce(function(email) {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  
  if (!email) {
    errorDiv.textContent = '';
    emailInput.classList.remove('valid', 'invalid');
    return;
  }
  
  if (emailRegex.test(email)) {
    errorDiv.textContent = '';
    emailInput.classList.remove('invalid');
    emailInput.classList.add('valid');
  } else {
    errorDiv.textContent = 'Invalid email format';
    emailInput.classList.remove('valid');
    emailInput.classList.add('invalid');
  }
}, 500);

emailInput.addEventListener('input', (e) => {
  validateEmail(e.target.value);
});
```

**5. Scroll Event:**
```javascript
// Show/hide header on scroll
const header = document.querySelector('.header');
let lastScrollY = window.scrollY;

const handleScroll = debounce(function() {
  const currentScrollY = window.scrollY;
  
  if (currentScrollY > lastScrollY && currentScrollY > 100) {
    // Scrolling down
    header.classList.add('hidden');
  } else {
    // Scrolling up
    header.classList.remove('hidden');
  }
  
  lastScrollY = currentScrollY;
}, 100);

window.addEventListener('scroll', handleScroll);
```

**6. API Rate Limiting:**
```javascript
// Prevent excessive API calls
class APIClient {
  constructor() {
    this.debouncedGet = debounce((url) => this.get(url), 1000);
  }
  
  async get(url) {
    console.log('Fetching:', url);
    const response = await fetch(url);
    return response.json();
  }
  
  // Public method uses debounced version
  async fetchData(endpoint) {
    return this.debouncedGet(endpoint);
  }
}

const client = new APIClient();

// Multiple rapid calls
client.fetchData('/api/users');  // Queued
client.fetchData('/api/users');  // Queued (cancels previous)
client.fetchData('/api/users');  // Queued (cancels previous)
// Only last call executes after 1 second
```

**7. Autocomplete:**
```javascript
// Autocomplete with debouncing
class Autocomplete {
  constructor(inputElement, suggestionsElement) {
    this.input = inputElement;
    this.suggestions = suggestionsElement;
    
    this.debouncedFetch = debounce(
      (query) => this.fetchSuggestions(query),
      300
    );
    
    this.input.addEventListener('input', (e) => {
      this.handleInput(e.target.value);
    });
  }
  
  handleInput(value) {
    if (value.length < 2) {
      this.suggestions.innerHTML = '';
      return;
    }
    
    this.debouncedFetch(value);
  }
  
  async fetchSuggestions(query) {
    this.suggestions.innerHTML = 'Loading...';
    
    try {
      const response = await fetch(`/api/autocomplete?q=${query}`);
      const data = await response.json();
      
      this.suggestions.innerHTML = data.map(item => 
        `<div class="suggestion">${item}</div>`
      ).join('');
    } catch (error) {
      this.suggestions.innerHTML = 'Error loading suggestions';
    }
  }
}

const autocomplete = new Autocomplete(
  document.getElementById('search-input'),
  document.getElementById('suggestions')
);
```

#### **Debounce vs Throttle Comparison**

```javascript
// Debounce: Execute after delay of inactivity
const debounced = debounce(() => {
  console.log('Debounced');
}, 1000);

// Rapid calls:
debounced(); // Timer starts
debounced(); // Timer resets
debounced(); // Timer resets
debounced(); // Timer resets
// Executes once after 1 second of no calls

// Throttle: Execute at most once per interval (covered in next question)
```

#### **Common Use Cases**

```javascript
// ✅ GOOD USE CASES for debouncing:

// 1. Search/autocomplete (wait for user to finish typing)
searchInput.addEventListener('input', debounce(search, 300));

// 2. Form validation (validate after user stops typing)
emailInput.addEventListener('input', debounce(validateEmail, 500));

// 3. Auto-save (save after user stops editing)
textarea.addEventListener('input', debounce(autoSave, 2000));

// 4. Window resize (recalculate after resize finishes)
window.addEventListener('resize', debounce(recalculate, 250));

// 5. API calls (prevent excessive requests)
button.addEventListener('click', debounce(fetchData, 1000));

// ❌ BAD USE CASES for debouncing:

// 1. Infinite scroll (use throttle instead)
// Need regular updates, not waiting for pause

// 2. Game controls (use direct handling)
// Need immediate response

// 3. Real-time updates (use direct handling or throttle)
// Need consistent updates

// 4. Analytics tracking (use throttle)
// Need regular tracking, not just at end
```

#### **Key Takeaways**

- **Debouncing** delays execution until after period of inactivity
- **Timer resets** on each function call
- **Execute once** after user stops triggering events
- **Perfect for search** and autocomplete
- **Reduces API calls** significantly
- **Improves performance** by avoiding excessive operations
- **Add immediate option** for first-call execution
- **Include cancel/flush** methods for control
- **Use for resize events** to avoid layout thrashing
- **Great for auto-save** features
- **Different from throttling** (throttle limits rate, debounce waits)
- **Delay should match** user behavior expectations
- **300-500ms common** for search inputs
- **1000-2000ms common** for auto-save
- **Understanding debouncing** is essential for performance optimization

---

### 84. What is throttling?

**Answer:**

Throttling is a technique that limits how often a function can execute over time. Unlike debouncing (which delays until inactivity), throttling ensures a function executes at most once per specified time interval, regardless of how many times it's called. The first call executes immediately, then subsequent calls are ignored until the interval passes. This is ideal for events that fire continuously, like scroll or mousemove.

#### **Basic Concept**

```javascript
// Without throttling - function fires constantly
function handleScroll() {
  console.log('Scroll position:', window.scrollY);
  // Can fire 100+ times per second!
}

// window.addEventListener('scroll', handleScroll);

// With throttling - function fires at most once per interval
function throttle(func, limit) {
  let inThrottle;
  
  return function(...args) {
    if (!inThrottle) {
      func.apply(this, args);
      inThrottle = true;
      
      setTimeout(() => {
        inThrottle = false;
      }, limit);
    }
  };
}

const throttledScroll = throttle(handleScroll, 1000);

// window.addEventListener('scroll', throttledScroll);
// Fires once per second, regardless of scroll speed
```

#### **1. Simple Throttle Implementation**

```javascript
function throttle(func, limit) {
  let inThrottle;
  
  return function(...args) {
    const context = this;
    
    if (!inThrottle) {
      func.apply(context, args);
      inThrottle = true;
      
      setTimeout(() => {
        inThrottle = false;
      }, limit);
    }
  };
}

// Usage
function logMessage(message) {
  console.log('Message:', message, Date.now());
}

const throttledLog = throttle(logMessage, 2000);

throttledLog('First');   // Executes immediately
throttledLog('Second');  // Ignored (within 2s)
throttledLog('Third');   // Ignored (within 2s)
// After 2 seconds, next call will execute
```

#### **2. Throttle with Trailing Call**

```javascript
// Execute first call immediately, last call after interval
function throttle(func, limit) {
  let inThrottle;
  let lastFunc;
  let lastRan;
  
  return function(...args) {
    const context = this;
    
    if (!inThrottle) {
      func.apply(context, args);
      lastRan = Date.now();
      inThrottle = true;
      
      setTimeout(() => {
        inThrottle = false;
      }, limit);
    } else {
      clearTimeout(lastFunc);
      lastFunc = setTimeout(() => {
        if (Date.now() - lastRan >= limit) {
          func.apply(context, args);
          lastRan = Date.now();
        }
      }, limit - (Date.now() - lastRan));
    }
  };
}

// First call: executes immediately
// Intermediate calls: ignored
// Last call: executes after interval
```

#### **3. Advanced Throttle with Options**

```javascript
function throttle(func, limit, options = {}) {
  let timeout;
  let previous = 0;
  
  const throttled = function(...args) {
    const now = Date.now();
    const context = this;
    
    // If leading is false, set previous on first call
    if (!previous && options.leading === false) {
      previous = now;
    }
    
    const remaining = limit - (now - previous);
    
    if (remaining <= 0 || remaining > limit) {
      if (timeout) {
        clearTimeout(timeout);
        timeout = null;
      }
      
      previous = now;
      func.apply(context, args);
    } else if (!timeout && options.trailing !== false) {
      timeout = setTimeout(() => {
        previous = options.leading === false ? 0 : Date.now();
        timeout = null;
        func.apply(context, args);
      }, remaining);
    }
  };
  
  throttled.cancel = function() {
    clearTimeout(timeout);
    previous = 0;
    timeout = null;
  };
  
  return throttled;
}

// Usage with options
const throttledFn = throttle(myFunc, 1000, {
  leading: true,   // Execute on first call
  trailing: true   // Execute after interval with last args
});
```

#### **4. RequestAnimationFrame Throttle**

```javascript
// Throttle to browser's paint cycle (~60fps)
function throttleRAF(func) {
  let rafId = null;
  let lastArgs = null;
  
  return function(...args) {
    lastArgs = args;
    
    if (rafId === null) {
      rafId = requestAnimationFrame(() => {
        func.apply(this, lastArgs);
        rafId = null;
      });
    }
  };
}

// Usage for animations/visual updates
const throttledAnimation = throttleRAF(function(scrollY) {
  // Update element position
  element.style.transform = `translateY(${scrollY}px)`;
});

window.addEventListener('scroll', () => {
  throttledAnimation(window.scrollY);
});
```

#### **Practical Examples**

**1. Infinite Scroll:**
```javascript
class InfiniteScroll {
  constructor(containerElement) {
    this.container = containerElement;
    this.loading = false;
    this.page = 1;
    
    // Throttle scroll handler to check at most every 200ms
    this.throttledScroll = throttle(() => this.checkScroll(), 200);
    
    window.addEventListener('scroll', this.throttledScroll);
  }
  
  checkScroll() {
    const scrollTop = window.scrollY;
    const windowHeight = window.innerHeight;
    const documentHeight = document.documentElement.scrollHeight;
    
    // Check if near bottom
    if (scrollTop + windowHeight >= documentHeight - 500) {
      this.loadMore();
    }
  }
  
  async loadMore() {
    if (this.loading) return;
    
    this.loading = true;
    console.log('Loading page:', this.page + 1);
    
    try {
      const response = await fetch(`/api/posts?page=${++this.page}`);
      const posts = await response.json();
      
      posts.forEach(post => {
        const div = document.createElement('div');
        div.className = 'post';
        div.textContent = post.title;
        this.container.appendChild(div);
      });
    } catch (error) {
      console.error('Error loading posts:', error);
    } finally {
      this.loading = false;
    }
  }
}

const infiniteScroll = new InfiniteScroll(document.getElementById('posts'));
```

**2. Mouse Movement Tracking:**
```javascript
// Track mouse position for parallax effect
const parallaxLayer = document.querySelector('.parallax-layer');

const updateParallax = throttle(function(e) {
  const mouseX = e.clientX;
  const mouseY = e.clientY;
  const windowWidth = window.innerWidth;
  const windowHeight = window.innerHeight;
  
  // Calculate offset (-50 to 50 pixels)
  const offsetX = ((mouseX / windowWidth) - 0.5) * 100;
  const offsetY = ((mouseY / windowHeight) - 0.5) * 100;
  
  parallaxLayer.style.transform = `translate(${offsetX}px, ${offsetY}px)`;
}, 50);  // Update at most 20 times per second

document.addEventListener('mousemove', updateParallax);

// Without throttling: fires 100+ times per second
// With throttling: fires 20 times per second (smooth, efficient)
```

**3. Scroll Progress Indicator:**
```javascript
const progressBar = document.querySelector('.progress-bar');

const updateProgress = throttle(function() {
  const windowHeight = window.innerHeight;
  const documentHeight = document.documentElement.scrollHeight;
  const scrollTop = window.scrollY;
  
  // Calculate progress percentage
  const maxScroll = documentHeight - windowHeight;
  const progress = (scrollTop / maxScroll) * 100;
  
  progressBar.style.width = `${Math.min(progress, 100)}%`;
}, 100);  // Update every 100ms

window.addEventListener('scroll', updateProgress);
```

**4. Button Click Protection:**
```javascript
// Prevent rapid clicking (e.g., submit button)
const submitButton = document.getElementById('submit-btn');

const handleSubmit = throttle(async function() {
  console.log('Submitting form...');
  
  const formData = new FormData(document.getElementById('myForm'));
  
  try {
    const response = await fetch('/api/submit', {
      method: 'POST',
      body: formData
    });
    
    const result = await response.json();
    console.log('Success:', result);
  } catch (error) {
    console.error('Error:', error);
  }
}, 2000);  // Allow submit once every 2 seconds

submitButton.addEventListener('click', (e) => {
  e.preventDefault();
  handleSubmit();
});
```

**5. Window Resize:**
```javascript
// Update layout during resize (not just after)
const updateLayout = throttle(function() {
  const width = window.innerWidth;
  
  console.log('Updating layout for width:', width);
  
  // Responsive adjustments
  const cards = document.querySelectorAll('.card');
  const columns = width < 768 ? 1 : width < 1024 ? 2 : 3;
  const cardWidth = `${100 / columns}%`;
  
  cards.forEach(card => {
    card.style.width = cardWidth;
  });
}, 250);  // Update every 250ms during resize

window.addEventListener('resize', updateLayout);

// Throttling provides continuous updates during resize
// Debouncing would only update after resize stops
```

**6. Game Loop / Animation:**
```javascript
class Game {
  constructor(canvas) {
    this.canvas = canvas;
    this.ctx = canvas.getContext('2d');
    this.mouseX = 0;
    this.mouseY = 0;
    
    // Throttle mouse tracking to game's update rate
    this.throttledMouseMove = throttle((x, y) => {
      this.mouseX = x;
      this.mouseY = y;
    }, 16);  // ~60fps
    
    canvas.addEventListener('mousemove', (e) => {
      this.throttledMouseMove(e.clientX, e.clientY);
    });
    
    this.startGameLoop();
  }
  
  startGameLoop() {
    const loop = () => {
      this.update();
      this.render();
      requestAnimationFrame(loop);
    };
    loop();
  }
  
  update() {
    // Game logic using this.mouseX, this.mouseY
  }
  
  render() {
    // Draw game state
    this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
    this.ctx.fillStyle = 'blue';
    this.ctx.fillRect(this.mouseX - 25, this.mouseY - 25, 50, 50);
  }
}

const game = new Game(document.getElementById('game-canvas'));
```

**7. Analytics Tracking:**
```javascript
// Track scroll depth at intervals
class AnalyticsTracker {
  constructor() {
    this.maxScrollDepth = 0;
    this.milestones = [25, 50, 75, 100];
    this.tracked = new Set();
    
    this.throttledTrack = throttle(() => this.trackScrollDepth(), 1000);
    
    window.addEventListener('scroll', this.throttledTrack);
  }
  
  trackScrollDepth() {
    const windowHeight = window.innerHeight;
    const documentHeight = document.documentElement.scrollHeight;
    const scrollTop = window.scrollY;
    
    const scrollPercentage = Math.round(
      ((scrollTop + windowHeight) / documentHeight) * 100
    );
    
    this.maxScrollDepth = Math.max(this.maxScrollDepth, scrollPercentage);
    
    // Track milestones
    this.milestones.forEach(milestone => {
      if (this.maxScrollDepth >= milestone && !this.tracked.has(milestone)) {
        this.tracked.add(milestone);
        this.sendAnalytics('scroll_depth', milestone);
      }
    });
  }
  
  sendAnalytics(event, value) {
    console.log(`Analytics: ${event} = ${value}%`);
    // Send to analytics service
  }
}

const tracker = new AnalyticsTracker();
```

#### **Debounce vs Throttle Visual**

```javascript
// DEBOUNCE: Waits for silence
// Events:  | | | | |     | |         |
// Execute:                    ✓           ✓
// (Executes after events stop)

// THROTTLE: Regular intervals
// Events:  | | | | | | | | | | | | | | | |
// Execute: ✓       ✓       ✓       ✓
// (Executes at regular intervals)

// Example showing the difference
let debounceCount = 0;
let throttleCount = 0;

const debouncedFn = debounce(() => {
  console.log('Debounced:', ++debounceCount);
}, 1000);

const throttledFn = throttle(() => {
  console.log('Throttled:', ++throttleCount);
}, 1000);

// Simulate 100 rapid calls over 2 seconds
for (let i = 0; i < 100; i++) {
  setTimeout(() => {
    debouncedFn();  // Executes once after all calls
    throttledFn();  // Executes ~2 times (once per second)
  }, i * 20);
}
```

#### **When to Use What**

```javascript
// USE DEBOUNCING when you want to wait for user to finish:
// ✅ Search input (wait for typing to stop)
// ✅ Form validation (validate after user finishes)
// ✅ Auto-save (save after editing stops)
// ✅ Window resize (recalculate after resize finishes)

// USE THROTTLING when you want regular updates during activity:
// ✅ Scroll events (infinite scroll, progress bars)
// ✅ Mouse movement (parallax, tooltips)
// ✅ Window resize (live updates during resize)
// ✅ Game controls (regular input sampling)
// ✅ Analytics (periodic tracking)
// ✅ Button clicks (prevent rapid clicking)

// Example comparison
const searchInput = document.getElementById('search');

// Debounce: Search after user stops typing
searchInput.addEventListener('input', debounce((e) => {
  performSearch(e.target.value);
}, 500));

// Throttle: Show character count while typing
searchInput.addEventListener('input', throttle((e) => {
  updateCharCount(e.target.value.length);
}, 100));
```

#### **Performance Comparison**

```javascript
// Benchmark: Performance improvement
function benchmark() {
  let callCount = 0;
  
  // Normal function (no throttling)
  function normalFn() {
    callCount++;
  }
  
  // Throttled function
  let throttledCount = 0;
  const throttledFn = throttle(() => {
    throttledCount++;
  }, 100);
  
  // Simulate 1000 calls over 1 second
  const interval = setInterval(() => {
    normalFn();
    throttledFn();
  }, 1);
  
  setTimeout(() => {
    clearInterval(interval);
    console.log('Normal calls:', callCount);      // ~1000
    console.log('Throttled calls:', throttledCount); // ~10
    console.log('Reduction:', Math.round((1 - throttledCount / callCount) * 100) + '%');
  }, 1000);
}

// Result: 99% reduction in function executions
```

#### **Key Takeaways**

- **Throttling** limits execution rate over time
- **First call executes** immediately
- **Subsequent calls ignored** until interval passes
- **Regular intervals** regardless of call frequency
- **Perfect for scroll** and mousemove events
- **Different from debouncing** (throttle=regular, debounce=waits)
- **Use for infinite scroll** and progress tracking
- **Improves performance** by reducing executions
- **Add trailing option** to execute last call
- **Include cancel method** for cleanup
- **RequestAnimationFrame** good for visual updates
- **100-250ms common** for scroll events
- **50-100ms common** for mouse tracking
- **16ms (~60fps)** for animations
- **Understanding throttling** essential for smooth UX

82. What is memoization?
83. What is debouncing?
84. What is throttling?

---

### 85. What are Web Workers?

**Answer:**

Web Workers are a browser API that allows JavaScript code to run in background threads, separate from the main UI thread. They enable parallel processing and prevent heavy computations from blocking the user interface. Workers run in an isolated context without access to the DOM, communicating with the main thread through message passing. This is essential for performing CPU-intensive tasks without freezing the page.

#### **Basic Concept**

```javascript
// Main thread (main.js)
// Check if Web Workers are supported
if (window.Worker) {
  // Create a new worker
  const worker = new Worker('worker.js');
  
  // Send message to worker
  worker.postMessage({ task: 'calculate', value: 1000000 });
  
  // Receive message from worker
  worker.onmessage = function(e) {
    console.log('Result from worker:', e.data);
  };
  
  // Handle errors
  worker.onerror = function(error) {
    console.error('Worker error:', error.message);
  };
  
  // Terminate worker when done
  // worker.terminate();
} else {
  console.log('Web Workers not supported');
}

// Worker thread (worker.js)
// Listen for messages from main thread
self.onmessage = function(e) {
  const { task, value } = e.data;
  
  if (task === 'calculate') {
    // Perform heavy calculation
    let result = 0;
    for (let i = 0; i < value; i++) {
      result += i;
    }
    
    // Send result back to main thread
    self.postMessage({ result: result });
  }
};
```

#### **1. Creating and Using Workers**

```javascript
// main.js
class WorkerManager {
  constructor(workerPath) {
    this.worker = new Worker(workerPath);
    this.setupListeners();
  }
  
  setupListeners() {
    this.worker.onmessage = (e) => {
      console.log('Message from worker:', e.data);
    };
    
    this.worker.onerror = (error) => {
      console.error('Worker error:', error.message, error.filename, error.lineno);
    };
  }
  
  send(data) {
    this.worker.postMessage(data);
  }
  
  terminate() {
    this.worker.terminate();
    console.log('Worker terminated');
  }
}

// Usage
const manager = new WorkerManager('worker.js');
manager.send({ command: 'start', data: [1, 2, 3, 4, 5] });

// Later: cleanup
// manager.terminate();

// worker.js
self.onmessage = function(e) {
  const { command, data } = e.data;
  
  switch(command) {
    case 'start':
      const result = processData(data);
      self.postMessage({ status: 'complete', result });
      break;
    case 'stop':
      self.close(); // Terminate from inside worker
      break;
  }
};

function processData(data) {
  return data.map(x => x * 2);
}
```

#### **2. Dedicated Workers**

```javascript
// Main thread
const worker = new Worker('dedicated-worker.js');

worker.postMessage({
  action: 'process',
  data: { numbers: [1, 2, 3, 4, 5] }
});

worker.onmessage = (e) => {
  console.log('Processed:', e.data);
  // { action: 'process', result: [2, 4, 6, 8, 10] }
};

// dedicated-worker.js
self.addEventListener('message', (e) => {
  const { action, data } = e.data;
  
  if (action === 'process') {
    const result = data.numbers.map(n => n * 2);
    
    self.postMessage({
      action: action,
      result: result
    });
  }
});

// Error handling in worker
self.addEventListener('error', (e) => {
  console.error('Worker error:', e);
});
```

#### **3. Shared Workers**

```javascript
// Main thread (can be from multiple pages)
const sharedWorker = new SharedWorker('shared-worker.js');

// Start the port
sharedWorker.port.start();

// Send message
sharedWorker.port.postMessage({
  type: 'subscribe',
  id: 'page-1'
});

// Receive messages
sharedWorker.port.onmessage = (e) => {
  console.log('Message from shared worker:', e.data);
};

// shared-worker.js
const connections = [];

self.onconnect = function(e) {
  const port = e.ports[0];
  connections.push(port);
  
  port.onmessage = function(e) {
    const { type, id } = e.data;
    
    if (type === 'subscribe') {
      console.log('New connection:', id);
      
      // Broadcast to all connections
      connections.forEach(conn => {
        conn.postMessage({
          type: 'notification',
          message: `${id} joined`
        });
      });
    }
  };
  
  port.start();
};
```

#### **4. Inline Workers (Blob Workers)**

```javascript
// Create worker from inline code
function createInlineWorker(workerFunction) {
  const blob = new Blob(
    ['(' + workerFunction.toString() + ')()'],
    { type: 'application/javascript' }
  );
  
  const blobURL = URL.createObjectURL(blob);
  return new Worker(blobURL);
}

// Define worker logic as function
const workerFunction = function() {
  self.onmessage = function(e) {
    const result = e.data * e.data;
    self.postMessage(result);
  };
};

// Create and use inline worker
const inlineWorker = createInlineWorker(workerFunction);

inlineWorker.postMessage(5);

inlineWorker.onmessage = function(e) {
  console.log('Result:', e.data); // 25
};

// Cleanup
// URL.revokeObjectURL(worker);
// inlineWorker.terminate();
```

#### **5. Worker with Promise Wrapper**

```javascript
// Utility to wrap worker in Promise
class WorkerPromise {
  constructor(workerPath) {
    this.worker = new Worker(workerPath);
    this.taskId = 0;
    this.pendingTasks = new Map();
    
    this.worker.onmessage = (e) => {
      const { taskId, result, error } = e.data;
      const task = this.pendingTasks.get(taskId);
      
      if (task) {
        if (error) {
          task.reject(new Error(error));
        } else {
          task.resolve(result);
        }
        this.pendingTasks.delete(taskId);
      }
    };
  }
  
  execute(data) {
    return new Promise((resolve, reject) => {
      const taskId = this.taskId++;
      
      this.pendingTasks.set(taskId, { resolve, reject });
      this.worker.postMessage({ taskId, data });
    });
  }
  
  terminate() {
    this.worker.terminate();
    this.pendingTasks.forEach(task => {
      task.reject(new Error('Worker terminated'));
    });
    this.pendingTasks.clear();
  }
}

// Usage
const workerPromise = new WorkerPromise('promise-worker.js');

async function processData() {
  try {
    const result1 = await workerPromise.execute({ value: 10 });
    console.log('Result 1:', result1);
    
    const result2 = await workerPromise.execute({ value: 20 });
    console.log('Result 2:', result2);
  } catch (error) {
    console.error('Error:', error);
  }
}

processData();

// promise-worker.js
self.onmessage = function(e) {
  const { taskId, data } = e.data;
  
  try {
    // Perform calculation
    const result = data.value * 2;
    
    // Send success
    self.postMessage({ taskId, result });
  } catch (error) {
    // Send error
    self.postMessage({ taskId, error: error.message });
  }
};
```

#### **Practical Examples**

**1. Image Processing:**
```javascript
// Main thread
class ImageProcessor {
  constructor() {
    this.worker = new Worker('image-worker.js');
  }
  
  async processImage(imageData) {
    return new Promise((resolve, reject) => {
      this.worker.onmessage = (e) => {
        resolve(e.data.processedImage);
      };
      
      this.worker.onerror = (error) => {
        reject(error);
      };
      
      this.worker.postMessage({
        imageData: imageData,
        filter: 'grayscale'
      });
    });
  }
}

// Usage
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');
const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);

const processor = new ImageProcessor();
const processed = await processor.processImage(imageData);

ctx.putImageData(processed, 0, 0);

// image-worker.js
self.onmessage = function(e) {
  const { imageData, filter } = e.data;
  
  if (filter === 'grayscale') {
    const data = imageData.data;
    
    for (let i = 0; i < data.length; i += 4) {
      const avg = (data[i] + data[i + 1] + data[i + 2]) / 3;
      data[i] = avg;     // Red
      data[i + 1] = avg; // Green
      data[i + 2] = avg; // Blue
      // data[i + 3] is alpha, leave unchanged
    }
  }
  
  self.postMessage({ processedImage: imageData });
};
```

**2. Data Processing:**
```javascript
// Main thread - processing large dataset
class DataWorker {
  constructor() {
    this.worker = new Worker('data-worker.js');
  }
  
  async processLargeDataset(data) {
    return new Promise((resolve) => {
      this.worker.onmessage = (e) => {
        resolve(e.data);
      };
      
      this.worker.postMessage({ data, operation: 'analyze' });
    });
  }
}

// Usage
const largeDataset = Array.from({ length: 1000000 }, (_, i) => ({
  id: i,
  value: Math.random() * 100
}));

const dataWorker = new DataWorker();

console.log('Processing started...');
const results = await dataWorker.processLargeDataset(largeDataset);
console.log('Results:', results);

// data-worker.js
self.onmessage = function(e) {
  const { data, operation } = e.data;
  
  if (operation === 'analyze') {
    // Heavy computation
    const sum = data.reduce((acc, item) => acc + item.value, 0);
    const avg = sum / data.length;
    const max = Math.max(...data.map(item => item.value));
    const min = Math.min(...data.map(item => item.value));
    
    self.postMessage({
      count: data.length,
      sum: sum,
      average: avg,
      max: max,
      min: min
    });
  }
};
```

**3. Prime Number Calculator:**
```javascript
// Main thread
const primeWorker = new Worker('prime-worker.js');

document.getElementById('calculate').addEventListener('click', () => {
  const max = parseInt(document.getElementById('max').value);
  const resultDiv = document.getElementById('result');
  
  resultDiv.textContent = 'Calculating...';
  
  primeWorker.postMessage({ max });
});

primeWorker.onmessage = function(e) {
  const { primes, count, time } = e.data;
  document.getElementById('result').textContent = 
    `Found ${count} primes in ${time}ms`;
  console.log('Primes:', primes);
};

// prime-worker.js
self.onmessage = function(e) {
  const { max } = e.data;
  const startTime = Date.now();
  
  const primes = findPrimes(max);
  const endTime = Date.now();
  
  self.postMessage({
    primes: primes,
    count: primes.length,
    time: endTime - startTime
  });
};

function findPrimes(max) {
  const primes = [];
  
  for (let num = 2; num <= max; num++) {
    let isPrime = true;
    
    for (let i = 2; i <= Math.sqrt(num); i++) {
      if (num % i === 0) {
        isPrime = false;
        break;
      }
    }
    
    if (isPrime) primes.push(num);
  }
  
  return primes;
}
```

**4. Progress Reporting:**
```javascript
// Main thread
const progressWorker = new Worker('progress-worker.js');
const progressBar = document.getElementById('progress');

progressWorker.onmessage = function(e) {
  const { type, progress, result } = e.data;
  
  if (type === 'progress') {
    progressBar.style.width = progress + '%';
    progressBar.textContent = progress + '%';
  } else if (type === 'complete') {
    console.log('Complete! Result:', result);
    progressBar.textContent = 'Done!';
  }
};

progressWorker.postMessage({ start: 0, end: 100000 });

// progress-worker.js
self.onmessage = function(e) {
  const { start, end } = e.data;
  let result = 0;
  
  for (let i = start; i < end; i++) {
    result += Math.sqrt(i);
    
    // Report progress every 1000 iterations
    if (i % 1000 === 0) {
      const progress = Math.round((i / end) * 100);
      self.postMessage({ type: 'progress', progress });
    }
  }
  
  self.postMessage({ type: 'complete', result });
};
```

**5. Worker Pool:**
```javascript
// Worker pool for parallel processing
class WorkerPool {
  constructor(workerPath, poolSize = 4) {
    this.workers = [];
    this.taskQueue = [];
    this.activeWorkers = 0;
    
    // Create worker pool
    for (let i = 0; i < poolSize; i++) {
      const worker = new Worker(workerPath);
      worker.onmessage = (e) => this.handleWorkerMessage(worker, e);
      this.workers.push({ worker, busy: false });
    }
  }
  
  execute(data) {
    return new Promise((resolve, reject) => {
      const task = { data, resolve, reject };
      
      const availableWorker = this.workers.find(w => !w.busy);
      
      if (availableWorker) {
        this.runTask(availableWorker, task);
      } else {
        this.taskQueue.push(task);
      }
    });
  }
  
  runTask(workerObj, task) {
    workerObj.busy = true;
    workerObj.currentTask = task;
    this.activeWorkers++;
    
    workerObj.worker.postMessage(task.data);
  }
  
  handleWorkerMessage(worker, e) {
    const workerObj = this.workers.find(w => w.worker === worker);
    
    if (workerObj.currentTask) {
      workerObj.currentTask.resolve(e.data);
      workerObj.busy = false;
      workerObj.currentTask = null;
      this.activeWorkers--;
      
      // Process next task in queue
      if (this.taskQueue.length > 0) {
        const nextTask = this.taskQueue.shift();
        this.runTask(workerObj, nextTask);
      }
    }
  }
  
  terminate() {
    this.workers.forEach(w => w.worker.terminate());
    this.workers = [];
  }
}

// Usage
const pool = new WorkerPool('calc-worker.js', 4);

// Process multiple tasks in parallel
const tasks = Array.from({ length: 20 }, (_, i) => i);
const promises = tasks.map(i => pool.execute({ value: i * 1000 }));

Promise.all(promises).then(results => {
  console.log('All tasks complete:', results);
  pool.terminate();
});
```

**6. File Processing:**
```javascript
// Process large file in chunks
class FileProcessor {
  constructor() {
    this.worker = new Worker('file-worker.js');
  }
  
  async processFile(file) {
    const chunkSize = 1024 * 1024; // 1MB chunks
    const chunks = Math.ceil(file.size / chunkSize);
    const results = [];
    
    for (let i = 0; i < chunks; i++) {
      const start = i * chunkSize;
      const end = Math.min(start + chunkSize, file.size);
      const chunk = file.slice(start, end);
      
      const arrayBuffer = await chunk.arrayBuffer();
      const result = await this.processChunk(arrayBuffer, i, chunks);
      results.push(result);
      
      // Update progress
      const progress = Math.round(((i + 1) / chunks) * 100);
      this.onProgress?.(progress);
    }
    
    return results;
  }
  
  processChunk(arrayBuffer, index, total) {
    return new Promise((resolve) => {
      this.worker.onmessage = (e) => resolve(e.data);
      this.worker.postMessage({
        chunk: arrayBuffer,
        index,
        total
      }, [arrayBuffer]); // Transfer ownership
    });
  }
}

// file-worker.js
self.onmessage = function(e) {
  const { chunk, index, total } = e.data;
  
  // Process the chunk
  const view = new Uint8Array(chunk);
  let checksum = 0;
  
  for (let i = 0; i < view.length; i++) {
    checksum += view[i];
  }
  
  self.postMessage({
    index,
    checksum,
    size: view.length
  });
};
```

#### **Worker Capabilities and Limitations**

```javascript
// WORKERS CAN:
// ✅ Access these APIs:
self.onmessage = function(e) {
  // - XMLHttpRequest / fetch
  fetch('/api/data');
  
  // - setTimeout / setInterval
  setTimeout(() => {}, 1000);
  
  // - IndexedDB
  // indexedDB.open('myDB');
  
  // - WebSockets
  // const ws = new WebSocket('ws://example.com');
  
  // - navigator object (subset)
  console.log(self.navigator.userAgent);
  
  // - importScripts (load other scripts)
  importScripts('util.js', 'helper.js');
  
  // - console methods
  console.log('Worker log');
};

// WORKERS CANNOT:
// ❌ Access these:
// - window object
// - document object
// - parent object
// - DOM manipulation
// - Most HTML5 APIs (Geolocation, etc.)
// - alert(), confirm(), prompt()
// - localStorage (use IndexedDB instead)

// Example: Import additional scripts in worker
// worker.js
importScripts('lib1.js', 'lib2.js');

self.onmessage = function(e) {
  // Can now use functions from lib1.js and lib2.js
  const result = someLibraryFunction(e.data);
  self.postMessage(result);
};
```

#### **Performance Considerations**

```javascript
// Benchmark: With vs Without Worker
function benchmark() {
  const iterations = 1000000000;
  
  // Without worker (blocks UI)
  console.time('Main thread (blocking)');
  let sum = 0;
  for (let i = 0; i < iterations; i++) {
    sum += i;
  }
  console.timeEnd('Main thread (blocking)');
  // UI is frozen during calculation
  
  // With worker (non-blocking)
  const worker = new Worker('calc-worker.js');
  
  console.time('Worker (non-blocking)');
  worker.postMessage({ iterations });
  worker.onmessage = function(e) {
    console.timeEnd('Worker (non-blocking)');
    console.log('Result:', e.data);
    // UI remains responsive
  };
}

// Consider overhead:
// - Worker creation time
// - Message passing overhead
// - Memory copying for large data

// Use Transferable Objects for large data
const largeArray = new Uint8Array(1024 * 1024 * 10); // 10MB
worker.postMessage({ data: largeArray }, [largeArray.buffer]);
// Array is transferred (moved), not copied
// largeArray is now unusable in main thread
```

#### **Key Takeaways**

- **Web Workers** run JavaScript in background threads
- **Prevent UI blocking** during heavy computations
- **No DOM access** - workers can't manipulate page
- **Message passing** via postMessage/onmessage
- **Dedicated workers** tied to single page
- **Shared workers** accessible from multiple pages/tabs
- **Transferable objects** for efficient large data transfer
- **importScripts** to load libraries in workers
- **Worker pool** for parallel task processing
- **Good for**: image processing, data analysis, cryptography
- **Overhead exists** - message passing has cost
- **Use for expensive** operations (>50ms)
- **Check support** with `if (window.Worker)`
- **Always terminate** workers when done
- **Essential for** maintaining smooth user experience

---

### 86. What is the difference between localStorage and sessionStorage?

**Answer:**

Both `localStorage` and `sessionStorage` are Web Storage APIs that store key-value pairs in the browser. The main difference is **persistence**: `localStorage` data persists indefinitely until explicitly cleared, while `sessionStorage` data is cleared when the page session ends (browser tab closes). Both have the same API and 5-10MB storage limit per origin, but different use cases and lifetimes.

#### **Basic Comparison**

```javascript
// localStorage - persists forever
localStorage.setItem('user', 'John');
console.log(localStorage.getItem('user')); // 'John'
// Still available after browser restart

// sessionStorage - cleared when tab closes
sessionStorage.setItem('tempUser', 'Jane');
console.log(sessionStorage.getItem('tempUser')); // 'Jane'
// Lost when tab/window closes

// Both use the same API
// setItem(key, value)
// getItem(key)
// removeItem(key)
// clear()
// key(index)
// length property
```

#### **1. Persistence Differences**

```javascript
// localStorage - survives:
// ✅ Browser restart
// ✅ Tab close/reopen
// ✅ Computer restart
// ✅ Indefinitely (until manually cleared)

localStorage.setItem('permanent', 'I will survive');
// Close browser, reopen, still there

// sessionStorage - cleared when:
// ❌ Tab is closed
// ❌ Browser is closed
// ❌ Page session ends
// ✅ Survives page refresh (F5)
// ✅ Survives navigation within same tab

sessionStorage.setItem('temporary', 'I will disappear');
// Close tab - data is gone

// Page refresh example
sessionStorage.setItem('counter', '0');

// After refresh:
let counter = parseInt(sessionStorage.getItem('counter') || '0');
counter++;
sessionStorage.setItem('counter', counter.toString());
// Counter persists across refreshes within same session
```

#### **2. Scope Differences**

```javascript
// localStorage - shared across:
// ✅ All tabs/windows from same origin
// ✅ All browser instances

// Tab 1
localStorage.setItem('shared', 'visible everywhere');

// Tab 2 (same origin)
console.log(localStorage.getItem('shared')); // 'visible everywhere'

// sessionStorage - isolated to:
// ❌ Only the specific tab/window
// ❌ Not shared between tabs

// Tab 1
sessionStorage.setItem('isolated', 'only in this tab');

// Tab 2 (same origin)
console.log(sessionStorage.getItem('isolated')); // null (not accessible)

// Duplicate tab (Ctrl+Shift+T or right-click > Duplicate)
// sessionStorage IS copied to the duplicate tab
```

#### **3. Complete API Usage**

```javascript
// SET ITEM
localStorage.setItem('name', 'John');
sessionStorage.setItem('name', 'Jane');

// GET ITEM
const localName = localStorage.getItem('name');    // 'John'
const sessionName = sessionStorage.getItem('name'); // 'Jane'
const missing = localStorage.getItem('nonexistent'); // null

// REMOVE ITEM
localStorage.removeItem('name');
sessionStorage.removeItem('name');

// CLEAR ALL
localStorage.clear();   // Removes all localStorage items
sessionStorage.clear(); // Removes all sessionStorage items

// KEY ACCESS (by index)
localStorage.setItem('a', '1');
localStorage.setItem('b', '2');
console.log(localStorage.key(0));  // 'a' or 'b' (order not guaranteed)
console.log(localStorage.key(1));  // 'b' or 'a'

// LENGTH
console.log(localStorage.length);   // Number of items
console.log(sessionStorage.length);

// ITERATE
for (let i = 0; i < localStorage.length; i++) {
  const key = localStorage.key(i);
  const value = localStorage.getItem(key);
  console.log(key, value);
}

// Object-like access (not recommended, but works)
localStorage.myKey = 'value';          // Same as setItem
console.log(localStorage.myKey);       // Same as getItem
delete localStorage.myKey;             // Same as removeItem
```

#### **4. Storing Complex Data**

```javascript
// localStorage/sessionStorage only store strings
// Must serialize objects

// STORING OBJECTS
const user = {
  id: 123,
  name: 'John Doe',
  email: 'john@example.com',
  preferences: {
    theme: 'dark',
    notifications: true
  }
};

// Convert to JSON string
localStorage.setItem('user', JSON.stringify(user));

// Retrieve and parse
const storedUser = JSON.parse(localStorage.getItem('user'));
console.log(storedUser.name);        // 'John Doe'
console.log(storedUser.preferences.theme); // 'dark'

// STORING ARRAYS
const todos = ['Task 1', 'Task 2', 'Task 3'];
localStorage.setItem('todos', JSON.stringify(todos));

const storedTodos = JSON.parse(localStorage.getItem('todos'));
console.log(storedTodos[0]); // 'Task 1'

// STORING DATES (special handling)
const now = new Date();
localStorage.setItem('timestamp', now.toISOString());

const storedDate = new Date(localStorage.getItem('timestamp'));
console.log(storedDate);

// HELPER FUNCTIONS
const storage = {
  set(key, value) {
    try {
      localStorage.setItem(key, JSON.stringify(value));
      return true;
    } catch (e) {
      console.error('Storage error:', e);
      return false;
    }
  },
  
  get(key, defaultValue = null) {
    try {
      const item = localStorage.getItem(key);
      return item ? JSON.parse(item) : defaultValue;
    } catch (e) {
      console.error('Parse error:', e);
      return defaultValue;
    }
  },
  
  remove(key) {
    localStorage.removeItem(key);
  },
  
  clear() {
    localStorage.clear();
  }
};

// Usage
storage.set('user', { name: 'John', age: 30 });
const user = storage.get('user');
```

#### **5. Storage Events**

```javascript
// Listen for storage changes (only fires in OTHER tabs/windows)
window.addEventListener('storage', (e) => {
  console.log('Storage changed:');
  console.log('Key:', e.key);           // Key that changed
  console.log('Old value:', e.oldValue); // Previous value
  console.log('New value:', e.newValue); // New value
  console.log('URL:', e.url);           // Page URL where change occurred
  console.log('Storage:', e.storageArea); // localStorage or sessionStorage
});

// Tab 1 - make change
localStorage.setItem('username', 'John');

// Tab 2 - receives event
// Event fires with details about the change

// Note: Event does NOT fire in the tab that made the change
// Only fires in other tabs/windows from same origin

// Practical example: Sync logout across tabs
window.addEventListener('storage', (e) => {
  if (e.key === 'logout') {
    console.log('User logged out in another tab');
    // Redirect to login page
    window.location.href = '/login';
  }
});

// In any tab, trigger logout
localStorage.setItem('logout', Date.now().toString());
localStorage.removeItem('logout'); // Clean up
```

#### **Practical Examples**

**1. User Preferences:**
```javascript
// localStorage for persistent preferences
class Preferences {
  static save(key, value) {
    const prefs = this.getAll();
    prefs[key] = value;
    localStorage.setItem('preferences', JSON.stringify(prefs));
  }
  
  static get(key, defaultValue = null) {
    const prefs = this.getAll();
    return prefs[key] ?? defaultValue;
  }
  
  static getAll() {
    const prefs = localStorage.getItem('preferences');
    return prefs ? JSON.parse(prefs) : {};
  }
  
  static clear() {
    localStorage.removeItem('preferences');
  }
}

// Usage
Preferences.save('theme', 'dark');
Preferences.save('fontSize', 16);
Preferences.save('language', 'en');

console.log(Preferences.get('theme'));     // 'dark'
console.log(Preferences.getAll());         // { theme: 'dark', fontSize: 16, ... }

// Apply theme on page load
document.addEventListener('DOMContentLoaded', () => {
  const theme = Preferences.get('theme', 'light');
  document.body.classList.add(`theme-${theme}`);
});
```

**2. Shopping Cart:**
```javascript
// localStorage for persistent cart
class ShoppingCart {
  constructor() {
    this.storageKey = 'shopping-cart';
  }
  
  getItems() {
    const cart = localStorage.getItem(this.storageKey);
    return cart ? JSON.parse(cart) : [];
  }
  
  addItem(product) {
    const items = this.getItems();
    const existing = items.find(item => item.id === product.id);
    
    if (existing) {
      existing.quantity++;
    } else {
      items.push({ ...product, quantity: 1 });
    }
    
    localStorage.setItem(this.storageKey, JSON.stringify(items));
  }
  
  removeItem(productId) {
    let items = this.getItems();
    items = items.filter(item => item.id !== productId);
    localStorage.setItem(this.storageKey, JSON.stringify(items));
  }
  
  getTotal() {
    const items = this.getItems();
    return items.reduce((sum, item) => sum + (item.price * item.quantity), 0);
  }
  
  clear() {
    localStorage.removeItem(this.storageKey);
  }
}

const cart = new ShoppingCart();
cart.addItem({ id: 1, name: 'Product 1', price: 29.99 });
console.log(cart.getTotal()); // 29.99
```

**3. Form Data Backup:**
```javascript
// sessionStorage for temporary form data
class FormBackup {
  constructor(formId) {
    this.formId = formId;
    this.storageKey = `form-backup-${formId}`;
  }
  
  save() {
    const form = document.getElementById(this.formId);
    const formData = new FormData(form);
    const data = Object.fromEntries(formData);
    
    sessionStorage.setItem(this.storageKey, JSON.stringify(data));
  }
  
  restore() {
    const saved = sessionStorage.getItem(this.storageKey);
    if (!saved) return false;
    
    const data = JSON.parse(saved);
    const form = document.getElementById(this.formId);
    
    Object.entries(data).forEach(([name, value]) => {
      const field = form.elements[name];
      if (field) field.value = value;
    });
    
    return true;
  }
  
  clear() {
    sessionStorage.removeItem(this.storageKey);
  }
}

// Usage
const formBackup = new FormBackup('contact-form');

// Auto-save on input
document.getElementById('contact-form').addEventListener('input', () => {
  formBackup.save();
});

// Restore on page load
window.addEventListener('DOMContentLoaded', () => {
  if (formBackup.restore()) {
    console.log('Form data restored');
  }
});

// Clear after successful submit
document.getElementById('contact-form').addEventListener('submit', (e) => {
  e.preventDefault();
  // Submit form...
  formBackup.clear();
});
```

**4. Authentication Token:**
```javascript
// localStorage for persistent login, sessionStorage for temporary
class Auth {
  static saveToken(token, remember = false) {
    if (remember) {
      localStorage.setItem('authToken', token);
      localStorage.setItem('tokenExpiry', Date.now() + (30 * 24 * 60 * 60 * 1000)); // 30 days
    } else {
      sessionStorage.setItem('authToken', token);
    }
  }
  
  static getToken() {
    // Check localStorage first
    const localToken = localStorage.getItem('authToken');
    if (localToken) {
      const expiry = localStorage.getItem('tokenExpiry');
      if (Date.now() < parseInt(expiry)) {
        return localToken;
      } else {
        this.clearToken(); // Expired
      }
    }
    
    // Check sessionStorage
    return sessionStorage.getItem('authToken');
  }
  
  static isAuthenticated() {
    return !!this.getToken();
  }
  
  static clearToken() {
    localStorage.removeItem('authToken');
    localStorage.removeItem('tokenExpiry');
    sessionStorage.removeItem('authToken');
  }
}

// Login
function login(username, password, rememberMe) {
  // API call...
  const token = 'abc123';
  Auth.saveToken(token, rememberMe);
}

// Check auth
if (Auth.isAuthenticated()) {
  // User is logged in
  const token = Auth.getToken();
  // Use token for API calls
}

// Logout
Auth.clearToken();
```

**5. Multi-Tab Synchronization:**
```javascript
// Sync state across tabs using localStorage + storage event
class TabSync {
  constructor(key) {
    this.key = key;
    this.listeners = [];
    
    window.addEventListener('storage', (e) => {
      if (e.key === this.key && e.newValue) {
        const data = JSON.parse(e.newValue);
        this.notifyListeners(data);
      }
    });
  }
  
  broadcast(data) {
    localStorage.setItem(this.key, JSON.stringify(data));
  }
  
  onChange(callback) {
    this.listeners.push(callback);
  }
  
  notifyListeners(data) {
    this.listeners.forEach(callback => callback(data));
  }
}

// Usage
const sync = new TabSync('app-state');

sync.onChange((data) => {
  console.log('State updated in another tab:', data);
  updateUI(data);
});

// Broadcast change
sync.broadcast({ user: 'John', status: 'online' });
```

#### **Comparison Table**

| Feature | localStorage | sessionStorage |
|---------|-------------|----------------|
| **Lifetime** | Permanent (until cleared) | Page session (tab closes) |
| **Scope** | All tabs/windows (same origin) | Single tab/window |
| **Survives refresh** | Yes | Yes |
| **Survives browser close** | Yes | No |
| **Survives tab close** | Yes | No |
| **Shared across tabs** | Yes | No |
| **Storage limit** | ~5-10MB | ~5-10MB |
| **API** | Same | Same |
| **Storage event** | Fires in other tabs | Fires in other tabs |
| **Use case** | Persistent data | Temporary data |

#### **Best Practices**

```javascript
// 1. Always handle quota exceeded errors
try {
  localStorage.setItem('key', 'value');
} catch (e) {
  if (e.name === 'QuotaExceededError') {
    console.error('Storage quota exceeded');
    // Clear old data or notify user
  }
}

// 2. Check for storage support
function storageAvailable(type) {
  try {
    const storage = window[type];
    const test = '__storage_test__';
    storage.setItem(test, test);
    storage.removeItem(test);
    return true;
  } catch (e) {
    return false;
  }
}

if (storageAvailable('localStorage')) {
  // localStorage is available
}

// 3. Namespace your keys
const APP_PREFIX = 'myapp_';

localStorage.setItem(APP_PREFIX + 'user', 'John');
localStorage.setItem(APP_PREFIX + 'settings', '{}');

// 4. Validate stored data
function getSafeItem(key, defaultValue) {
  try {
    const item = localStorage.getItem(key);
    return item ? JSON.parse(item) : defaultValue;
  } catch (e) {
    console.error('Invalid JSON:', e);
    return defaultValue;
  }
}

// 5. Clean up old data
function cleanupOldData() {
  const keys = [];
  for (let i = 0; i < localStorage.length; i++) {
    keys.push(localStorage.key(i));
  }
  
  keys.forEach(key => {
    if (key.startsWith('temp_')) {
      const timestamp = localStorage.getItem(key + '_timestamp');
      if (Date.now() - parseInt(timestamp) > 24 * 60 * 60 * 1000) {
        localStorage.removeItem(key);
        localStorage.removeItem(key + '_timestamp');
      }
    }
  });
}
```

#### **Key Takeaways**

- **localStorage** persists indefinitely
- **sessionStorage** cleared when tab closes
- **Same API** for both (setItem, getItem, removeItem, clear)
- **localStorage shared** across all tabs (same origin)
- **sessionStorage isolated** to single tab
- **5-10MB limit** for each (per origin)
- **Only strings** stored (use JSON for objects)
- **Storage events** fire in other tabs only
- **Use localStorage** for persistent preferences, cart
- **Use sessionStorage** for temporary form data, session state
- **Always try/catch** for quota errors
- **Validate data** when retrieving
- **Namespace keys** to avoid conflicts
- **Not secure** - don't store sensitive data
- **Synchronous API** - consider IndexedDB for large data

---

### 87. What are cookies?

**Answer:**

Cookies are small pieces of data (max 4KB) stored by the browser and sent with every HTTP request to the same domain. They're primarily used for session management, personalization, and tracking. Unlike localStorage/sessionStorage, cookies are automatically included in HTTP requests, making them essential for server-side authentication. They have configurable expiration, domain/path scope, and security attributes (HttpOnly, Secure, SameSite).

#### **Basic Cookie Operations**

```javascript
// SET COOKIE (basic)
document.cookie = "username=John";

// SET COOKIE with expiration
const expires = new Date();
expires.setDate(expires.getDate() + 7); // 7 days from now
document.cookie = `username=John; expires=${expires.toUTCString()}`;

// SET COOKIE with max-age (seconds)
document.cookie = "sessionId=abc123; max-age=3600"; // 1 hour

// SET COOKIE with path
document.cookie = "theme=dark; path=/";

// SET COOKIE with domain
document.cookie = "user=John; domain=.example.com";

// SET COOKIE with secure flag (HTTPS only)
document.cookie = "token=xyz; secure";

// SET COOKIE with HttpOnly (not accessible via JavaScript - must be set by server)
// document.cookie = "authToken=xyz; httpOnly"; // Doesn't work from JavaScript

// SET COOKIE with SameSite
document.cookie = "csrf=token; SameSite=Strict";

// GET ALL COOKIES
console.log(document.cookie); // "username=John; sessionId=abc123; theme=dark"

// DELETE COOKIE (set expiration to past)
document.cookie = "username=; expires=Thu, 01 Jan 1970 00:00:00 UTC; path=/";
```

#### **1. Cookie Utility Functions**

```javascript
// Complete cookie utility
const Cookies = {
  // Set cookie
  set(name, value, options = {}) {
    let cookieString = `${encodeURIComponent(name)}=${encodeURIComponent(value)}`;
    
    // Expiration
    if (options.days) {
      const expires = new Date();
      expires.setDate(expires.getDate() + options.days);
      cookieString += `; expires=${expires.toUTCString()}`;
    }
    
    if (options.maxAge) {
      cookieString += `; max-age=${options.maxAge}`;
    }
    
    // Path
    cookieString += `; path=${options.path || '/'}`;
    
    // Domain
    if (options.domain) {
      cookieString += `; domain=${options.domain}`;
    }
    
    // Secure
    if (options.secure) {
      cookieString += '; secure';
    }
    
    // SameSite
    if (options.sameSite) {
      cookieString += `; SameSite=${options.sameSite}`;
    }
    
    document.cookie = cookieString;
  },
  
  // Get cookie
  get(name) {
    const cookies = document.cookie.split('; ');
    
    for (const cookie of cookies) {
      const [key, value] = cookie.split('=');
      if (decodeURIComponent(key) === name) {
        return decodeURIComponent(value);
      }
    }
    
    return null;
  },
  
  // Get all cookies as object
  getAll() {
    const cookies = {};
    const cookieArray = document.cookie.split('; ');
    
    for (const cookie of cookieArray) {
      const [key, value] = cookie.split('=');
      if (key) {
        cookies[decodeURIComponent(key)] = decodeURIComponent(value);
      }
    }
    
    return cookies;
  },
  
  // Remove cookie
  remove(name, options = {}) {
    this.set(name, '', {
      ...options,
      maxAge: -1
    });
  },
  
  // Check if cookie exists
  has(name) {
    return this.get(name) !== null;
  }
};

// Usage
Cookies.set('username', 'John Doe', { days: 7 });
Cookies.set('theme', 'dark', { days: 365, path: '/' });
Cookies.set('sessionId', 'abc123', { maxAge: 3600 });

console.log(Cookies.get('username'));  // 'John Doe'
console.log(Cookies.getAll());         // { username: 'John Doe', theme: 'dark', ... }
console.log(Cookies.has('theme'));     // true

Cookies.remove('username');
```

#### **2. Cookie Attributes**

```javascript
// EXPIRES - specific date/time
const expires = new Date('2025-12-31');
Cookies.set('limited', 'value', { expires: expires.toUTCString() });

// MAX-AGE - seconds from now (preferred over expires)
Cookies.set('temp', 'value', { maxAge: 60 }); // 60 seconds

// PATH - cookie available only on specific path
Cookies.set('adminToken', 'xyz', { path: '/admin' });
// Available on /admin, /admin/users, etc.
// NOT available on / or /products

// DOMAIN - cookie available on domain and subdomains
Cookies.set('user', 'John', { domain: '.example.com' });
// Available on example.com, www.example.com, api.example.com

// SECURE - only sent over HTTPS
Cookies.set('authToken', 'secret', { secure: true });
// Won't be sent over HTTP connections

// HTTPONLY - not accessible via JavaScript (server-side only)
// Set-Cookie: sessionId=abc123; HttpOnly
// Cannot be set from JavaScript, only from server

// SAMESITE - CSRF protection
// Strict: Cookie only sent in first-party context
Cookies.set('csrf', 'token', { sameSite: 'Strict' });

// Lax: Cookie sent on top-level navigation
Cookies.set('session', 'xyz', { sameSite: 'Lax' });

// None: Cookie sent in all contexts (requires Secure)
Cookies.set('tracking', 'abc', { sameSite: 'None', secure: true });
```

#### **3. Storing Complex Data**

```javascript
// Cookies store strings only - serialize objects
const user = {
  id: 123,
  name: 'John Doe',
  role: 'admin'
};

// Store as JSON
Cookies.set('user', JSON.stringify(user), { days: 7 });

// Retrieve and parse
const storedUser = JSON.parse(Cookies.get('user'));
console.log(storedUser.name); // 'John Doe'

// Enhanced cookie utility with JSON support
const CookiesJSON = {
  set(name, value, options = {}) {
    const stringValue = typeof value === 'object' 
      ? JSON.stringify(value)
      : String(value);
    Cookies.set(name, stringValue, options);
  },
  
  get(name) {
    const value = Cookies.get(name);
    if (!value) return null;
    
    try {
      return JSON.parse(value);
    } catch (e) {
      return value; // Return as string if not JSON
    }
  }
};

// Usage
CookiesJSON.set('settings', { theme: 'dark', fontSize: 16 }, { days: 30 });
const settings = CookiesJSON.get('settings');
console.log(settings.theme); // 'dark'
```

#### **Practical Examples**

**1. Authentication Cookie:**
```javascript
class AuthCookies {
  static login(token, rememberMe = false) {
    const options = {
      path: '/',
      secure: true,
      sameSite: 'Strict'
    };
    
    if (rememberMe) {
      options.days = 30;
    } else {
      options.maxAge = 3600; // 1 hour
    }
    
    Cookies.set('authToken', token, options);
  }
  
  static getToken() {
    return Cookies.get('authToken');
  }
  
  static isAuthenticated() {
    return Cookies.has('authToken');
  }
  
  static logout() {
    Cookies.remove('authToken', { path: '/' });
  }
}

// Usage
AuthCookies.login('abc123', true); // Remember me
if (AuthCookies.isAuthenticated()) {
  const token = AuthCookies.getToken();
  // Use token for API calls
}
AuthCookies.logout();
```

**2. Consent Banner:**
```javascript
class CookieConsent {
  static COOKIE_NAME = 'cookie-consent';
  
  static hasConsent() {
    return Cookies.has(this.COOKIE_NAME);
  }
  
  static setConsent(categories) {
    Cookies.set(this.COOKIE_NAME, JSON.stringify(categories), {
      days: 365,
      path: '/',
      sameSite: 'Lax'
    });
  }
  
  static getConsent() {
    const consent = Cookies.get(this.COOKIE_NAME);
    return consent ? JSON.parse(consent) : null;
  }
  
  static showBanner() {
    if (this.hasConsent()) return;
    
    const banner = document.createElement('div');
    banner.className = 'cookie-banner';
    banner.innerHTML = `
      <p>We use cookies to improve your experience.</p>
      <button id="accept-all">Accept All</button>
      <button id="accept-necessary">Necessary Only</button>
    `;
    document.body.appendChild(banner);
    
    document.getElementById('accept-all').addEventListener('click', () => {
      this.setConsent({ necessary: true, analytics: true, marketing: true });
      banner.remove();
    });
    
    document.getElementById('accept-necessary').addEventListener('click', () => {
      this.setConsent({ necessary: true, analytics: false, marketing: false });
      banner.remove();
    });
  }
}

// On page load
CookieConsent.showBanner();
```

**3. Shopping Cart:**
```javascript
class CartCookies {
  static COOKIE_NAME = 'shopping-cart';
  static MAX_AGE = 7 * 24 * 60 * 60; // 7 days in seconds
  
  static getCart() {
    const cart = Cookies.get(this.COOKIE_NAME);
    return cart ? JSON.parse(cart) : [];
  }
  
  static saveCart(cart) {
    // Keep under 4KB limit
    const cartString = JSON.stringify(cart);
    if (cartString.length > 4000) {
      console.warn('Cart too large for cookie');
      // Consider using localStorage instead
      return false;
    }
    
    Cookies.set(this.COOKIE_NAME, cartString, {
      maxAge: this.MAX_AGE,
      path: '/',
      sameSite: 'Lax'
    });
    return true;
  }
  
  static addItem(item) {
    const cart = this.getCart();
    const existing = cart.find(i => i.id === item.id);
    
    if (existing) {
      existing.quantity++;
    } else {
      cart.push({ ...item, quantity: 1 });
    }
    
    this.saveCart(cart);
  }
  
  static removeItem(itemId) {
    let cart = this.getCart();
    cart = cart.filter(item => item.id !== itemId);
    this.saveCart(cart);
  }
  
  static clear() {
    Cookies.remove(this.COOKIE_NAME, { path: '/' });
  }
}

// Usage
CartCookies.addItem({ id: 1, name: 'Product 1', price: 29.99 });
console.log(CartCookies.getCart());
```

**4. Language Preference:**
```javascript
class LanguagePreference {
  static COOKIE_NAME = 'lang';
  
  static set(language) {
    Cookies.set(this.COOKIE_NAME, language, {
      days: 365,
      path: '/',
      sameSite: 'Lax'
    });
    
    // Apply language
    document.documentElement.lang = language;
    this.loadTranslations(language);
  }
  
  static get() {
    return Cookies.get(this.COOKIE_NAME) || 'en';
  }
  
  static loadTranslations(lang) {
    // Load language-specific content
    console.log(`Loading ${lang} translations`);
  }
}

// On page load
const lang = LanguagePreference.get();
LanguagePreference.set(lang);

// Language selector
document.getElementById('lang-select').addEventListener('change', (e) => {
  LanguagePreference.set(e.target.value);
});
```

**5. Session Tracking:**
```javascript
class SessionTracker {
  static init() {
    if (!this.getSessionId()) {
      this.createSession();
    }
    this.trackPageView();
  }
  
  static createSession() {
    const sessionId = this.generateId();
    Cookies.set('sessionId', sessionId, {
      maxAge: 30 * 60, // 30 minutes
      path: '/',
      sameSite: 'Lax'
    });
  }
  
  static getSessionId() {
    return Cookies.get('sessionId');
  }
  
  static trackPageView() {
    const sessionId = this.getSessionId();
    const pageViews = parseInt(Cookies.get('pageViews') || '0');
    
    Cookies.set('pageViews', String(pageViews + 1), {
      maxAge: 30 * 60,
      path: '/'
    });
    
    // Send to analytics
    console.log(`Session ${sessionId}: Page view ${pageViews + 1}`);
  }
  
  static generateId() {
    return Math.random().toString(36).substr(2, 9);
  }
}

// Initialize
SessionTracker.init();
```

#### **Cookies vs localStorage vs sessionStorage**

```javascript
// COOKIES:
// ✅ Sent with every HTTP request
// ✅ Can be set by server (Set-Cookie header)
// ✅ Configurable expiration
// ✅ HttpOnly flag (secure from JavaScript)
// ✅ Domain/path scoping
// ❌ 4KB size limit
// ❌ Increase request size
// Use for: Authentication, session management

// localStorage:
// ✅ 5-10MB storage
// ✅ Simple API
// ✅ Persists forever
// ✅ Not sent with requests
// ❌ No expiration
// ❌ Same origin only
// ❌ Synchronous
// Use for: Persistent user data, preferences

// sessionStorage:
// ✅ 5-10MB storage
// ✅ Tab-isolated
// ✅ Cleared on tab close
// ✅ Not sent with requests
// ❌ Not shared across tabs
// ❌ No expiration control
// Use for: Temporary session data
```

#### **Security Considerations**

```javascript
// 1. Always use Secure flag for sensitive cookies (HTTPS only)
Cookies.set('authToken', 'secret', {
  secure: true,
  sameSite: 'Strict'
});

// 2. Use HttpOnly for authentication (set from server)
// Set-Cookie: sessionId=abc123; HttpOnly; Secure; SameSite=Strict

// 3. Use SameSite to prevent CSRF
Cookies.set('csrf-token', 'xyz', {
  sameSite: 'Strict',  // or 'Lax'
  secure: true
});

// 4. Never store sensitive data in cookies
// ❌ Bad
Cookies.set('creditCard', '1234-5678-9012-3456');
Cookies.set('password', 'mypassword');

// ✅ Good - store only tokens/IDs
Cookies.set('sessionId', 'abc123', {
  httpOnly: true,  // Set from server
  secure: true,
  sameSite: 'Strict'
});

// 5. Validate and sanitize cookie values
function sanitizeCookieValue(value) {
  // Remove potentially harmful characters
  return value.replace(/[;\s]/g, '');
}

// 6. Set appropriate expiration
Cookies.set('temp', 'value', { maxAge: 3600 }); // 1 hour only
```

#### **Key Takeaways**

- **Cookies** are small data (max 4KB) stored by browser
- **Sent with every** HTTP request to same domain
- **Essential for** server-side authentication
- **Configurable expiration** (expires, max-age)
- **Domain/path scoping** controls where cookie is sent
- **Secure flag** requires HTTPS
- **HttpOnly flag** prevents JavaScript access (server-only)
- **SameSite attribute** prevents CSRF attacks
- **Use for** authentication, session management, tracking
- **Not for** large data (use localStorage)
- **Always encode** values (special characters)
- **Security critical** - use Secure, HttpOnly, SameSite
- **Increase request size** - consider impact
- **Check size limit** (4KB per cookie, ~180 cookies per domain)
- **Modern preference**: localStorage for client-side data, cookies for auth

82. What is memoization?
83. What is debouncing?
84. What is throttling?
85. What are Web Workers?
86. What is the difference between localStorage and sessionStorage?
87. What are cookies?

---

### 88. What is JSON and how do you parse it?

**Answer:**

JSON (JavaScript Object Notation) is a lightweight, text-based data interchange format that's easy for humans to read and write, and easy for machines to parse and generate. It uses JavaScript object syntax but is language-independent. JSON supports strings, numbers, booleans, null, arrays, and objects. JavaScript provides `JSON.parse()` to convert JSON strings to JavaScript objects and `JSON.stringify()` to convert JavaScript objects to JSON strings.

#### **Basic JSON Syntax**

```javascript
// Valid JSON
const jsonString = `{
  "name": "John Doe",
  "age": 30,
  "isActive": true,
  "email": null,
  "hobbies": ["reading", "gaming", "coding"],
  "address": {
    "street": "123 Main St",
    "city": "New York",
    "zip": "10001"
  }
}`;

// JSON Rules:
// ✅ Property names MUST be in double quotes
// ✅ String values MUST use double quotes
// ✅ Supports: string, number, boolean, null, array, object
// ❌ No functions, undefined, Date objects, RegExp, or comments
// ❌ No trailing commas
// ❌ Single quotes not allowed for strings

// Invalid JSON examples:
// { name: "John" }              // ❌ Missing quotes on key
// { "name": 'John' }            // ❌ Single quotes
// { "age": undefined }          // ❌ undefined not allowed
// { "greet": function() {} }    // ❌ Functions not allowed
// { "items": [1, 2, 3,] }       // ❌ Trailing comma
```

#### **1. JSON.parse() - String to Object**

```javascript
// Basic parsing
const jsonString = '{"name":"John","age":30}';
const obj = JSON.parse(jsonString);

console.log(obj.name);  // 'John'
console.log(obj.age);   // 30
console.log(typeof obj); // 'object'

// Parsing arrays
const arrayString = '[1, 2, 3, 4, 5]';
const arr = JSON.parse(arrayString);
console.log(arr[0]);  // 1
console.log(Array.isArray(arr));  // true

// Parsing nested structures
const complexString = `{
  "user": {
    "id": 123,
    "profile": {
      "name": "John",
      "email": "john@example.com"
    }
  },
  "posts": [
    {"id": 1, "title": "First Post"},
    {"id": 2, "title": "Second Post"}
  ]
}`;

const data = JSON.parse(complexString);
console.log(data.user.profile.name);  // 'John'
console.log(data.posts[0].title);     // 'First Post'

// Parsing primitives
console.log(JSON.parse('"hello"'));   // 'hello'
console.log(JSON.parse('123'));       // 123
console.log(JSON.parse('true'));      // true
console.log(JSON.parse('null'));      // null
```

#### **2. JSON.stringify() - Object to String**

```javascript
// Basic stringification
const obj = {
  name: 'John',
  age: 30,
  isActive: true
};

const jsonString = JSON.stringify(obj);
console.log(jsonString);  // '{"name":"John","age":30,"isActive":true}'

// Stringify with indentation (pretty print)
const prettyJson = JSON.stringify(obj, null, 2);
console.log(prettyJson);
// {
//   "name": "John",
//   "age": 30,
//   "isActive": true
// }

// Stringify arrays
const arr = [1, 2, 3, { name: 'test' }];
console.log(JSON.stringify(arr));  // '[1,2,3,{"name":"test"}]'

// What gets stringified
const testObj = {
  string: 'hello',
  number: 42,
  boolean: true,
  null: null,
  array: [1, 2, 3],
  object: { nested: 'value' },
  
  // These are ignored or converted:
  undefined: undefined,        // Ignored in objects
  function: function() {},     // Ignored
  symbol: Symbol('test'),      // Ignored
  date: new Date(),            // Converted to ISO string
  regexp: /test/               // Converted to {}
};

console.log(JSON.stringify(testObj));
// {"string":"hello","number":42,"boolean":true,"null":null,
//  "array":[1,2,3],"object":{"nested":"value"},
//  "date":"2025-12-20T..."}

// Array stringification (undefined becomes null)
console.log(JSON.stringify([1, undefined, 3]));  // '[1,null,3]'
```

#### **3. JSON.parse() with Reviver Function**

```javascript
// Reviver function transforms parsed values
const jsonString = `{
  "name": "John",
  "age": 30,
  "birthdate": "1995-06-15",
  "salary": "50000",
  "metadata": "skip"
}`;

const obj = JSON.parse(jsonString, (key, value) => {
  // Convert date strings to Date objects
  if (key === 'birthdate') {
    return new Date(value);
  }
  
  // Convert string numbers to actual numbers
  if (key === 'salary') {
    return parseFloat(value);
  }
  
  // Skip certain properties
  if (key === 'metadata') {
    return undefined;  // Property will be omitted
  }
  
  // Return value unchanged for other keys
  return value;
});

console.log(obj.birthdate instanceof Date);  // true
console.log(typeof obj.salary);              // 'number'
console.log('metadata' in obj);              // false

// Real-world example: Parse dates
const apiResponse = `{
  "id": 1,
  "createdAt": "2025-01-01T12:00:00.000Z",
  "updatedAt": "2025-01-15T08:30:00.000Z"
}`;

const parsed = JSON.parse(apiResponse, (key, value) => {
  // Convert ISO date strings to Date objects
  if (typeof value === 'string' && /^\d{4}-\d{2}-\d{2}T/.test(value)) {
    return new Date(value);
  }
  return value;
});

console.log(parsed.createdAt instanceof Date);  // true
console.log(parsed.createdAt.getFullYear());    // 2025
```

#### **4. JSON.stringify() with Replacer Function**

```javascript
// Replacer function filters/transforms values
const user = {
  id: 123,
  name: 'John',
  password: 'secret123',
  email: 'john@example.com',
  creditCard: '1234-5678-9012-3456',
  age: 30
};

// Filter out sensitive data
const safeJson = JSON.stringify(user, (key, value) => {
  // Remove sensitive fields
  if (key === 'password' || key === 'creditCard') {
    return undefined;  // Omit from output
  }
  return value;
});

console.log(safeJson);
// {"id":123,"name":"John","email":"john@example.com","age":30}

// Transform values
const transformed = JSON.stringify(user, (key, value) => {
  // Mask credit card numbers
  if (key === 'creditCard') {
    return '****-****-****-' + value.slice(-4);
  }
  // Convert strings to uppercase
  if (typeof value === 'string') {
    return value.toUpperCase();
  }
  return value;
});

// Replacer with array (whitelist properties)
const selected = JSON.stringify(user, ['id', 'name', 'email']);
console.log(selected);  // {"id":123,"name":"John","email":"john@example.com"}
```

#### **5. Error Handling**

```javascript
// JSON.parse() throws SyntaxError for invalid JSON
function safeParse(jsonString, defaultValue = null) {
  try {
    return JSON.parse(jsonString);
  } catch (error) {
    console.error('JSON parse error:', error.message);
    return defaultValue;
  }
}

// Invalid JSON examples
console.log(safeParse('invalid json'));           // null
console.log(safeParse("{name: 'John'}"));         // null (single quotes)
console.log(safeParse('{"age": undefined}'));     // null (undefined)
console.log(safeParse('{"items": [1,2,3,]}'));    // null (trailing comma)

// JSON.stringify() errors
const circular = { name: 'John' };
circular.self = circular;  // Circular reference

try {
  JSON.stringify(circular);
} catch (error) {
  console.error('Stringify error:', error.message);
  // TypeError: Converting circular structure to JSON
}

// Safe stringify with error handling
function safeStringify(obj, fallback = '{}') {
  try {
    return JSON.stringify(obj);
  } catch (error) {
    console.error('Stringify error:', error.message);
    return fallback;
  }
}
```

#### **6. Working with Dates**

```javascript
// Date serialization/deserialization
const data = {
  name: 'Event',
  date: new Date('2025-12-31'),
  timestamp: Date.now()
};

// Dates are converted to ISO strings
const jsonString = JSON.stringify(data);
console.log(jsonString);
// {"name":"Event","date":"2025-12-31T00:00:00.000Z","timestamp":1735689600000}

// Parse back with date conversion
const parsed = JSON.parse(jsonString);
console.log(typeof parsed.date);  // 'string' (not Date!)

// Convert date strings back to Date objects
const restored = JSON.parse(jsonString, (key, value) => {
  if (key === 'date' && typeof value === 'string') {
    return new Date(value);
  }
  return value;
});

console.log(restored.date instanceof Date);  // true

// Custom date handling
class DateAwareJSON {
  static stringify(obj) {
    return JSON.stringify(obj, (key, value) => {
      if (value instanceof Date) {
        return { __type: 'Date', value: value.toISOString() };
      }
      return value;
    });
  }
  
  static parse(str) {
    return JSON.parse(str, (key, value) => {
      if (value && value.__type === 'Date') {
        return new Date(value.value);
      }
      return value;
    });
  }
}

const obj = { name: 'Test', created: new Date() };
const json = DateAwareJSON.stringify(obj);
const parsed2 = DateAwareJSON.parse(json);
console.log(parsed2.created instanceof Date);  // true
```

#### **Practical Examples**

**1. API Data Handling:**
```javascript
// Fetch and parse JSON from API
async function fetchUserData(userId) {
  try {
    const response = await fetch(`/api/users/${userId}`);
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    // response.json() internally uses JSON.parse()
    const userData = await response.json();
    
    return userData;
  } catch (error) {
    console.error('Error fetching user data:', error);
    return null;
  }
}

// Send JSON to API
async function updateUserData(userId, data) {
  try {
    const response = await fetch(`/api/users/${userId}`, {
      method: 'PUT',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(data)  // Convert object to JSON string
    });
    
    const result = await response.json();
    return result;
  } catch (error) {
    console.error('Error updating user:', error);
    return null;
  }
}

// Usage
const user = await fetchUserData(123);
console.log(user.name);

await updateUserData(123, { name: 'John Updated' });
```

**2. Local Storage with JSON:**
```javascript
// Store complex objects in localStorage
class StorageHelper {
  static set(key, value) {
    try {
      const jsonString = JSON.stringify(value);
      localStorage.setItem(key, jsonString);
      return true;
    } catch (error) {
      console.error('Storage error:', error);
      return false;
    }
  }
  
  static get(key, defaultValue = null) {
    try {
      const jsonString = localStorage.getItem(key);
      return jsonString ? JSON.parse(jsonString) : defaultValue;
    } catch (error) {
      console.error('Parse error:', error);
      return defaultValue;
    }
  }
  
  static remove(key) {
    localStorage.removeItem(key);
  }
  
  static clear() {
    localStorage.clear();
  }
}

// Usage
const settings = {
  theme: 'dark',
  fontSize: 16,
  notifications: {
    email: true,
    push: false
  }
};

StorageHelper.set('userSettings', settings);
const retrieved = StorageHelper.get('userSettings');
console.log(retrieved.theme);  // 'dark'
```

**3. Deep Cloning Objects:**
```javascript
// Quick deep clone using JSON (with limitations)
const original = {
  name: 'John',
  age: 30,
  hobbies: ['reading', 'gaming'],
  address: {
    street: '123 Main St',
    city: 'New York'
  }
};

// Deep clone (simple objects only)
const clone = JSON.parse(JSON.stringify(original));

clone.address.city = 'Los Angeles';
console.log(original.address.city);  // 'New York' (not affected)
console.log(clone.address.city);     // 'Los Angeles'

// Limitations of JSON cloning
const complexObj = {
  date: new Date(),
  func: function() {},
  regex: /test/,
  undefined: undefined,
  symbol: Symbol('test')
};

const cloned = JSON.parse(JSON.stringify(complexObj));
console.log(cloned);
// {
//   date: "2025-12-20T..." (converted to string)
//   regex: {} (lost pattern)
//   // func, undefined, symbol are omitted
// }

// For complex objects, use structuredClone() or libraries
const betterClone = structuredClone(original);
```

**4. Configuration Files:**
```javascript
// Load and parse configuration
async function loadConfig(configPath) {
  try {
    const response = await fetch(configPath);
    const configText = await response.text();
    
    // Parse with defaults
    const config = JSON.parse(configText, (key, value) => {
      // Convert environment-specific values
      if (key === 'debug' && value === 'auto') {
        return process.env.NODE_ENV !== 'production';
      }
      return value;
    });
    
    return config;
  } catch (error) {
    console.error('Config load error:', error);
    return getDefaultConfig();
  }
}

function getDefaultConfig() {
  return {
    apiUrl: 'http://localhost:3000',
    timeout: 5000,
    debug: false
  };
}

// Usage
const config = await loadConfig('/config.json');
console.log(config.apiUrl);
```

**5. Form Data Serialization:**
```javascript
// Convert form data to JSON
class FormSerializer {
  static serialize(form) {
    const formData = new FormData(form);
    const obj = {};
    
    for (const [key, value] of formData.entries()) {
      // Handle multiple values (checkboxes)
      if (obj[key]) {
        if (!Array.isArray(obj[key])) {
          obj[key] = [obj[key]];
        }
        obj[key].push(value);
      } else {
        obj[key] = value;
      }
    }
    
    return JSON.stringify(obj);
  }
  
  static deserialize(jsonString, form) {
    try {
      const obj = JSON.parse(jsonString);
      
      Object.entries(obj).forEach(([name, value]) => {
        const field = form.elements[name];
        if (field) {
          if (field.type === 'checkbox') {
            field.checked = value;
          } else if (field.type === 'radio') {
            const radio = form.querySelector(`input[name="${name}"][value="${value}"]`);
            if (radio) radio.checked = true;
          } else {
            field.value = value;
          }
        }
      });
      
      return true;
    } catch (error) {
      console.error('Deserialize error:', error);
      return false;
    }
  }
}

// Usage
const form = document.getElementById('myForm');
const json = FormSerializer.serialize(form);

// Save to localStorage
localStorage.setItem('formBackup', json);

// Restore later
const saved = localStorage.getItem('formBackup');
FormSerializer.deserialize(saved, form);
```

**6. Data Transformation Pipeline:**
```javascript
// Transform API data with JSON
class DataTransformer {
  static transform(jsonString, transformations) {
    let data = JSON.parse(jsonString);
    
    transformations.forEach(transform => {
      data = transform(data);
    });
    
    return data;
  }
  
  static addTimestamps(data) {
    return {
      ...data,
      processedAt: new Date().toISOString()
    };
  }
  
  static filterSensitive(data) {
    const { password, ssn, creditCard, ...safe } = data;
    return safe;
  }
  
  static normalizeKeys(data) {
    const normalized = {};
    for (const [key, value] of Object.entries(data)) {
      const normalizedKey = key.toLowerCase().replace(/\s+/g, '_');
      normalized[normalizedKey] = value;
    }
    return normalized;
  }
}

// Usage
const apiResponse = '{"Name":"John","Age":30,"Password":"secret"}';

const transformed = DataTransformer.transform(apiResponse, [
  DataTransformer.normalizeKeys,
  DataTransformer.filterSensitive,
  DataTransformer.addTimestamps
]);

console.log(transformed);
// {
//   name: "John",
//   age: 30,
//   processedAt: "2025-12-20T..."
// }
```

**7. JSON Schema Validation:**
```javascript
// Basic JSON validation
class JSONValidator {
  static validate(jsonString, schema) {
    try {
      const data = JSON.parse(jsonString);
      return this.validateObject(data, schema);
    } catch (error) {
      return { valid: false, errors: ['Invalid JSON'] };
    }
  }
  
  static validateObject(data, schema) {
    const errors = [];
    
    for (const [key, rules] of Object.entries(schema)) {
      const value = data[key];
      
      // Required check
      if (rules.required && value === undefined) {
        errors.push(`${key} is required`);
        continue;
      }
      
      if (value !== undefined) {
        // Type check
        if (rules.type && typeof value !== rules.type) {
          errors.push(`${key} must be ${rules.type}`);
        }
        
        // Min/Max for numbers
        if (rules.type === 'number') {
          if (rules.min !== undefined && value < rules.min) {
            errors.push(`${key} must be at least ${rules.min}`);
          }
          if (rules.max !== undefined && value > rules.max) {
            errors.push(`${key} must be at most ${rules.max}`);
          }
        }
        
        // String length
        if (rules.type === 'string' && rules.minLength) {
          if (value.length < rules.minLength) {
            errors.push(`${key} must be at least ${rules.minLength} characters`);
          }
        }
      }
    }
    
    return {
      valid: errors.length === 0,
      errors
    };
  }
}

// Usage
const userSchema = {
  name: { type: 'string', required: true, minLength: 2 },
  age: { type: 'number', required: true, min: 0, max: 150 },
  email: { type: 'string', required: true }
};

const jsonData = '{"name":"Jo","age":30,"email":"john@example.com"}';
const result = JSONValidator.validate(jsonData, userSchema);

if (!result.valid) {
  console.log('Validation errors:', result.errors);
  // ["name must be at least 2 characters"]
}
```

#### **JSON Best Practices**

```javascript
// 1. Always handle parse errors
function parseJSON(jsonString) {
  try {
    return { success: true, data: JSON.parse(jsonString) };
  } catch (error) {
    return { success: false, error: error.message };
  }
}

// 2. Validate before parsing (optional)
function isValidJSON(str) {
  try {
    JSON.parse(str);
    return true;
  } catch (e) {
    return false;
  }
}

if (isValidJSON(userInput)) {
  const data = JSON.parse(userInput);
}

// 3. Use replacer to control serialization
const sensitiveData = {
  username: 'john',
  password: 'secret',
  token: 'abc123'
};

const safe = JSON.stringify(sensitiveData, ['username']);
// Only includes username

// 4. Pretty print for debugging
const debugJson = JSON.stringify(data, null, 2);
console.log(debugJson);

// 5. Handle circular references
function stringifyWithCircular(obj) {
  const seen = new WeakSet();
  
  return JSON.stringify(obj, (key, value) => {
    if (typeof value === 'object' && value !== null) {
      if (seen.has(value)) {
        return '[Circular]';
      }
      seen.add(value);
    }
    return value;
  });
}

// 6. Use Content-Type header
fetch('/api/data', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'  // Important!
  },
  body: JSON.stringify({ key: 'value' })
});
```

#### **Common Mistakes**

```javascript
// 1. Forgetting to parse/stringify
// ❌ Bad
localStorage.setItem('user', { name: 'John' });  // Stores "[object Object]"

// ✅ Good
localStorage.setItem('user', JSON.stringify({ name: 'John' }));

// 2. Not handling errors
// ❌ Bad
const data = JSON.parse(userInput);  // Throws on invalid JSON

// ✅ Good
try {
  const data = JSON.parse(userInput);
} catch (error) {
  console.error('Invalid JSON:', error);
}

// 3. Assuming JSON preserves types
const obj = { date: new Date(), func: () => {} };
const clone = JSON.parse(JSON.stringify(obj));
// date is now a string, func is missing!

// 4. Not using Content-Type header
// ❌ Bad
fetch('/api', {
  method: 'POST',
  body: JSON.stringify(data)  // Server might not parse correctly
});

// ✅ Good
fetch('/api', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify(data)
});

// 5. Circular references
const obj = { name: 'test' };
obj.self = obj;
// JSON.stringify(obj);  // ❌ Throws error
```

#### **Key Takeaways**

- **JSON** is text-based data interchange format
- **Language-independent** but uses JavaScript syntax
- **JSON.parse()** converts string to object
- **JSON.stringify()** converts object to string
- **Supports**: string, number, boolean, null, array, object
- **Does NOT support**: functions, undefined, Date, RegExp, Symbol
- **Property names** must be double-quoted
- **String values** must use double quotes
- **No trailing commas** allowed
- **Reviver function** transforms parsed values
- **Replacer function** filters/transforms stringified values
- **Always handle** parse errors (try/catch)
- **Dates become strings** - need manual conversion
- **Circular references** throw errors
- **Deep clone trick** works for simple objects only
- **Essential for APIs** and data storage

88. What is JSON and how do you parse it?

---

### 89. What is the difference between synchronous and asynchronous code?

**Answer:**

Synchronous code executes sequentially, blocking each line until the current operation completes before moving to the next. Asynchronous code allows operations to run in the background without blocking the main thread, enabling other code to execute while waiting for long-running tasks (like API calls, file I/O, or timers) to complete. JavaScript is single-threaded but uses the event loop, callback queue, and Web APIs to handle asynchronous operations efficiently.

#### **Basic Concept**

```javascript
// SYNCHRONOUS - Blocks execution
console.log('1. First');
console.log('2. Second');
console.log('3. Third');
// Output (in order): 1. First, 2. Second, 3. Third

// ASYNCHRONOUS - Non-blocking
console.log('1. First');

setTimeout(() => {
  console.log('2. Second (async)');
}, 0);

console.log('3. Third');
// Output: 1. First, 3. Third, 2. Second (async)
// Even with 0ms delay, setTimeout is asynchronous
```

#### **1. Synchronous Code Execution**

```javascript
// Synchronous execution - each line waits for previous to complete
function synchronousExample() {
  console.log('Start');
  
  // Blocking operation
  const result = heavyCalculation(); // Must complete before next line
  console.log('Result:', result);
  
  console.log('End');
}

function heavyCalculation() {
  let sum = 0;
  for (let i = 0; i < 1000000000; i++) {
    sum += i;
  }
  return sum;
}

synchronousExample();
// Output (in order):
// Start
// Result: 499999999500000000 (after waiting)
// End

// Problem: UI freezes during heavy calculation
// User cannot interact with page until calculation completes
```

#### **2. Asynchronous Code with Callbacks**

```javascript
// Asynchronous execution with callbacks
function asyncExample() {
  console.log('Start');
  
  // Non-blocking - continues immediately
  setTimeout(() => {
    console.log('Async operation completed');
  }, 2000);
  
  console.log('End');
}

asyncExample();
// Output:
// Start
// End
// Async operation completed (after 2 seconds)

// Multiple async operations
console.log('1');

setTimeout(() => console.log('2 - timeout 0ms'), 0);
setTimeout(() => console.log('3 - timeout 100ms'), 100);

Promise.resolve().then(() => console.log('4 - promise'));

console.log('5');

// Output:
// 1
// 5
// 4 - promise (microtask - higher priority)
// 2 - timeout 0ms (macrotask)
// 3 - timeout 100ms (macrotask)
```

#### **3. Asynchronous Code with Promises**

```javascript
// Promise-based asynchronous code
function fetchUserData(userId) {
  return new Promise((resolve, reject) => {
    // Simulate API call
    setTimeout(() => {
      if (userId > 0) {
        resolve({ id: userId, name: 'John Doe', email: 'john@example.com' });
      } else {
        reject(new Error('Invalid user ID'));
      }
    }, 1000);
  });
}

// Using promises
console.log('Fetching user...');

fetchUserData(123)
  .then(user => {
    console.log('User:', user);
    return fetchUserData(456); // Chain another async operation
  })
  .then(user2 => {
    console.log('User 2:', user2);
  })
  .catch(error => {
    console.error('Error:', error);
  });

console.log('Request sent, continuing...');

// Output:
// Fetching user...
// Request sent, continuing...
// User: { id: 123, name: 'John Doe', ... } (after 1 second)
// User 2: { id: 456, name: 'John Doe', ... } (after 2 seconds total)
```

#### **4. Asynchronous Code with Async/Await**

```javascript
// Modern async/await syntax (built on promises)
async function getUserData(userId) {
  // Simulate API call
  return new Promise(resolve => {
    setTimeout(() => {
      resolve({ id: userId, name: 'John Doe' });
    }, 1000);
  });
}

async function displayUserInfo() {
  console.log('Start fetching...');
  
  try {
    // Looks synchronous but is asynchronous
    const user = await getUserData(123);
    console.log('User:', user);
    
    const user2 = await getUserData(456);
    console.log('User 2:', user2);
    
    console.log('All data fetched');
  } catch (error) {
    console.error('Error:', error);
  }
}

displayUserInfo();
console.log('Function called, continuing...');

// Output:
// Start fetching...
// Function called, continuing...
// User: { id: 123, name: 'John Doe' } (after 1 second)
// User 2: { id: 456, name: 'John Doe' } (after 2 seconds total)
// All data fetched
```

#### **5. Event Loop and Execution Model**

```javascript
// Understanding the event loop
console.log('1 - Synchronous');

setTimeout(() => {
  console.log('2 - Macrotask (setTimeout)');
}, 0);

Promise.resolve().then(() => {
  console.log('3 - Microtask (Promise)');
});

queueMicrotask(() => {
  console.log('4 - Microtask (queueMicrotask)');
});

console.log('5 - Synchronous');

// Output:
// 1 - Synchronous
// 5 - Synchronous
// 3 - Microtask (Promise)
// 4 - Microtask (queueMicrotask)
// 2 - Macrotask (setTimeout)

// Execution order:
// 1. Synchronous code (call stack)
// 2. Microtasks (Promise callbacks, queueMicrotask)
// 3. Macrotasks (setTimeout, setInterval, I/O)
// 4. Render (if needed)
// 5. Repeat

// Event Loop Phases:
// ┌───────────────────────────┐
// │        Call Stack         │ ← Synchronous code executes here
// └───────────────────────────┘
//              ↓
// ┌───────────────────────────┐
// │    Microtask Queue        │ ← Promises, queueMicrotask
// │  (higher priority)        │
// └───────────────────────────┘
//              ↓
// ┌───────────────────────────┐
// │    Macrotask Queue        │ ← setTimeout, setInterval, I/O
// │  (lower priority)         │
// └───────────────────────────┘
```

#### **Practical Examples**

**1. Synchronous vs Asynchronous File Reading:**

```javascript
// SYNCHRONOUS (Node.js example - blocks execution)
function readFileSync() {
  console.log('Start reading file...');
  
  // Blocks until file is read (if this were real fs.readFileSync)
  const data = '/* file content */'; // Simulated blocking operation
  
  console.log('File content:', data);
  console.log('Done reading file');
}

readFileSync();
console.log('After file read'); // Must wait for file read to complete

// Output:
// Start reading file...
// File content: /* file content */
// Done reading file
// After file read

// ASYNCHRONOUS (Node.js example - non-blocking)
function readFileAsync() {
  console.log('Start reading file...');
  
  // Simulated async file read
  setTimeout(() => {
    const data = '/* file content */';
    console.log('File content:', data);
    console.log('Done reading file');
  }, 100);
}

readFileAsync();
console.log('After initiating file read'); // Doesn't wait

// Output:
// Start reading file...
// After initiating file read
// File content: /* file content */ (after delay)
// Done reading file
```

**2. API Requests Comparison:**

```javascript
// SYNCHRONOUS-STYLE (not recommended - for comparison)
function fetchUserSync(userId) {
  console.log(`Fetching user ${userId}...`);
  
  // Blocks until complete
  let result;
  const xhr = new XMLHttpRequest();
  xhr.open('GET', `/api/users/${userId}`, false); // false = synchronous
  xhr.send();
  
  if (xhr.status === 200) {
    result = JSON.parse(xhr.responseText);
  }
  
  console.log(`User ${userId} fetched`);
  return result;
}

// This blocks the browser - BAD!
// const user1 = fetchUserSync(1);
// const user2 = fetchUserSync(2);

// ASYNCHRONOUS with async/await (recommended)
async function fetchUserAsync(userId) {
  console.log(`Fetching user ${userId}...`);
  
  const response = await fetch(`/api/users/${userId}`);
  const user = await response.json();
  
  console.log(`User ${userId} fetched`);
  return user;
}

// Non-blocking - can fetch in parallel
async function fetchMultipleUsers() {
  console.log('Start fetching users...');
  
  // Parallel execution
  const [user1, user2, user3] = await Promise.all([
    fetchUserAsync(1),
    fetchUserAsync(2),
    fetchUserAsync(3)
  ]);
  
  console.log('All users fetched:', user1, user2, user3);
}

fetchMultipleUsers();
console.log('Fetching in progress...'); // Doesn't wait
```

**3. Sequential vs Parallel Async Operations:**

```javascript
// SEQUENTIAL (slower - waits for each)
async function sequentialFetch() {
  console.time('Sequential');
  
  const user1 = await fetch('/api/users/1').then(r => r.json());
  const user2 = await fetch('/api/users/2').then(r => r.json());
  const user3 = await fetch('/api/users/3').then(r => r.json());
  
  console.timeEnd('Sequential'); // ~3 seconds (1s + 1s + 1s)
  return [user1, user2, user3];
}

// PARALLEL (faster - runs simultaneously)
async function parallelFetch() {
  console.time('Parallel');
  
  const [user1, user2, user3] = await Promise.all([
    fetch('/api/users/1').then(r => r.json()),
    fetch('/api/users/2').then(r => r.json()),
    fetch('/api/users/3').then(r => r.json())
  ]);
  
  console.timeEnd('Parallel'); // ~1 second (all at once)
  return [user1, user2, user3];
}

// Sequential: 3 seconds total
// Parallel: 1 second total (3x faster!)
```

**4. Error Handling:**

```javascript
// SYNCHRONOUS error handling
function syncOperation() {
  try {
    console.log('Start');
    
    if (Math.random() > 0.5) {
      throw new Error('Sync error');
    }
    
    console.log('Success');
  } catch (error) {
    console.error('Caught:', error.message);
  }
}

syncOperation();

// ASYNCHRONOUS error handling with Promises
function asyncOperation() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (Math.random() > 0.5) {
        reject(new Error('Async error'));
      } else {
        resolve('Success');
      }
    }, 1000);
  });
}

asyncOperation()
  .then(result => console.log(result))
  .catch(error => console.error('Caught:', error.message));

// ASYNCHRONOUS error handling with async/await
async function handleAsyncOperation() {
  try {
    const result = await asyncOperation();
    console.log(result);
  } catch (error) {
    console.error('Caught:', error.message);
  }
}

handleAsyncOperation();
```

**5. Real-World Example: Data Processing Pipeline:**

```javascript
// Process data synchronously (blocks UI)
function processDataSync(data) {
  console.log('Processing started...');
  
  // Heavy computation - blocks everything
  const filtered = data.filter(item => item.value > 50);
  const mapped = filtered.map(item => ({ ...item, processed: true }));
  const sorted = mapped.sort((a, b) => b.value - a.value);
  
  console.log('Processing complete');
  return sorted;
}

// Process data asynchronously (non-blocking)
async function processDataAsync(data) {
  console.log('Processing started...');
  
  // Break into chunks, process with delays
  const result = [];
  const chunkSize = 1000;
  
  for (let i = 0; i < data.length; i += chunkSize) {
    const chunk = data.slice(i, i + chunkSize);
    
    // Process chunk
    const processed = chunk
      .filter(item => item.value > 50)
      .map(item => ({ ...item, processed: true }));
    
    result.push(...processed);
    
    // Yield to browser (allow UI updates)
    await new Promise(resolve => setTimeout(resolve, 0));
  }
  
  // Final sort
  result.sort((a, b) => b.value - a.value);
  
  console.log('Processing complete');
  return result;
}

// Large dataset
const largeData = Array.from({ length: 100000 }, (_, i) => ({
  id: i,
  value: Math.random() * 100
}));

// Synchronous: blocks UI for entire duration
// const result1 = processDataSync(largeData); // UI frozen

// Asynchronous: allows UI updates during processing
processDataAsync(largeData).then(result => {
  console.log('Result ready:', result.length, 'items');
});
console.log('UI remains responsive...'); // This logs immediately
```

**6. Callback Hell vs Modern Async:**

```javascript
// CALLBACK HELL (pyramid of doom)
function callbackHell() {
  getUser(123, (user, error) => {
    if (error) {
      console.error(error);
      return;
    }
    
    getPosts(user.id, (posts, error) => {
      if (error) {
        console.error(error);
        return;
      }
      
      getComments(posts[0].id, (comments, error) => {
        if (error) {
          console.error(error);
          return;
        }
        
        console.log('Comments:', comments);
      });
    });
  });
}

// MODERN ASYNC/AWAIT (clean and readable)
async function modernAsync() {
  try {
    const user = await getUser(123);
    const posts = await getPosts(user.id);
    const comments = await getComments(posts[0].id);
    
    console.log('Comments:', comments);
  } catch (error) {
    console.error(error);
  }
}

// Helper functions returning promises
function getUser(id) {
  return fetch(`/api/users/${id}`).then(r => r.json());
}

function getPosts(userId) {
  return fetch(`/api/posts?userId=${userId}`).then(r => r.json());
}

function getComments(postId) {
  return fetch(`/api/comments?postId=${postId}`).then(r => r.json());
}
```

**7. Combining Sync and Async:**

```javascript
class DataManager {
  constructor() {
    this.cache = new Map();
  }
  
  // Synchronous cache check
  getCached(key) {
    return this.cache.get(key); // Immediate return
  }
  
  // Asynchronous data fetching
  async fetchData(key) {
    // Check cache synchronously first
    const cached = this.getCached(key);
    if (cached) {
      console.log('Cache hit:', key);
      return cached;
    }
    
    // Fetch asynchronously if not cached
    console.log('Cache miss, fetching:', key);
    const data = await fetch(`/api/data/${key}`).then(r => r.json());
    
    // Update cache synchronously
    this.cache.set(key, data);
    
    return data;
  }
  
  // Synchronous cache invalidation
  clearCache(key) {
    if (key) {
      this.cache.delete(key);
    } else {
      this.cache.clear();
    }
  }
}

// Usage
const manager = new DataManager();

// First call - async fetch
const data1 = await manager.fetchData('users');
console.log(data1);

// Second call - sync cache hit
const data2 = await manager.fetchData('users');
console.log(data2); // Returns immediately from cache
```

#### **Performance Comparison**

```javascript
// Benchmark: Synchronous vs Asynchronous
async function benchmark() {
  // Simulate API calls (1 second each)
  function apiCall() {
    return new Promise(resolve => {
      setTimeout(() => resolve('data'), 1000);
    });
  }
  
  // SYNCHRONOUS-STYLE (sequential)
  console.time('Sequential');
  await apiCall();
  await apiCall();
  await apiCall();
  console.timeEnd('Sequential'); // ~3000ms
  
  // ASYNCHRONOUS (parallel)
  console.time('Parallel');
  await Promise.all([
    apiCall(),
    apiCall(),
    apiCall()
  ]);
  console.timeEnd('Parallel'); // ~1000ms
  
  console.log('Parallel is 3x faster!');
}

benchmark();
```

#### **When to Use Each**

```javascript
// USE SYNCHRONOUS when:
// ✅ Operations are fast (<16ms)
// ✅ Operations must complete before next step
// ✅ Working with in-memory data
// ✅ Simple calculations

// Examples:
const sum = numbers.reduce((a, b) => a + b, 0); // Sync - fast
const filtered = array.filter(x => x > 10);      // Sync - fast
const validated = validateForm(data);             // Sync - immediate

// USE ASYNCHRONOUS when:
// ✅ Network requests (API calls)
// ✅ File I/O operations
// ✅ Database queries
// ✅ Heavy computations (>50ms)
// ✅ Timers/delays
// ✅ User interactions that shouldn't block UI

// Examples:
const data = await fetch('/api/data');           // Async - network
const file = await readFile('data.txt');         // Async - I/O
const result = await heavyComputation(bigData);  // Async - CPU-intensive
await new Promise(resolve => setTimeout(resolve, 1000)); // Async - timer
```

#### **Common Patterns**

```javascript
// 1. Promise.all for parallel execution
const [users, posts, comments] = await Promise.all([
  fetch('/api/users').then(r => r.json()),
  fetch('/api/posts').then(r => r.json()),
  fetch('/api/comments').then(r => r.json())
]);

// 2. Promise.race for timeout
const dataWithTimeout = await Promise.race([
  fetch('/api/data').then(r => r.json()),
  new Promise((_, reject) => 
    setTimeout(() => reject(new Error('Timeout')), 5000)
  )
]);

// 3. Promise.allSettled for handling failures
const results = await Promise.allSettled([
  fetch('/api/endpoint1'),
  fetch('/api/endpoint2'),
  fetch('/api/endpoint3')
]);

results.forEach((result, index) => {
  if (result.status === 'fulfilled') {
    console.log(`Request ${index} succeeded:`, result.value);
  } else {
    console.log(`Request ${index} failed:`, result.reason);
  }
});

// 4. Async iteration
async function* asyncGenerator() {
  yield await Promise.resolve(1);
  yield await Promise.resolve(2);
  yield await Promise.resolve(3);
}

for await (const value of asyncGenerator()) {
  console.log(value); // 1, 2, 3
}

// 5. Async queue processing
class AsyncQueue {
  constructor() {
    this.queue = [];
    this.processing = false;
  }
  
  async add(asyncTask) {
    this.queue.push(asyncTask);
    if (!this.processing) {
      await this.process();
    }
  }
  
  async process() {
    this.processing = true;
    
    while (this.queue.length > 0) {
      const task = this.queue.shift();
      await task();
    }
    
    this.processing = false;
  }
}

const queue = new AsyncQueue();
queue.add(async () => await fetch('/api/1'));
queue.add(async () => await fetch('/api/2'));
```

#### **Key Takeaways**

- **Synchronous** code executes sequentially and blocks
- **Asynchronous** code runs in background without blocking
- **JavaScript is single-threaded** but uses event loop
- **Event loop** manages async operations efficiently
- **Microtasks** (Promises) execute before macrotasks (setTimeout)
- **Callbacks** were first async solution (callback hell)
- **Promises** improved async code readability
- **Async/await** is syntactic sugar over Promises
- **Always use async** for network requests and I/O
- **Parallel execution** with Promise.all for performance
- **Sequential execution** when operations depend on each other
- **Error handling** differs: try/catch vs .catch()
- **Non-blocking code** keeps UI responsive
- **Use synchronous** for fast, in-memory operations
- **Understanding async** is essential for modern JavaScript

---
