## Fundamentals & Basics

1. What is JavaScript?

**Answer:**
JavaScript is a high-level, interpreted, dynamically-typed programming language that is one of the core technologies of the web, alongside HTML and CSS. Originally created for client-side scripting in web browsers, it has evolved into a full-stack language with Node.js enabling server-side development.

**Key Characteristics:**
- **Multi-paradigm:** Supports procedural, object-oriented, and functional programming styles
- **Single-threaded:** Executes code in a single call stack, but achieves concurrency through the event loop and asynchronous operations
- **Just-in-Time (JIT) compiled:** Modern JavaScript engines compile code to machine code at runtime for better performance
- **Prototype-based:** Objects inherit directly from other objects, rather than from classes (though ES6+ added class syntax as syntactic sugar)

**Production Use Cases:**
- Frontend web applications (React, Angular, Vue.js)
- Backend services (Node.js, Express, NestJS)
- Mobile applications (React Native, Ionic)
- Desktop applications (Electron)
- IoT and embedded systems

2. What are the different data types in JavaScript?

**Answer:**
JavaScript has **8 data types** divided into two categories:

**Primitive Types (7):**
1. **String:** Textual data (`"hello"`, `'world'`, `` `template` ``)
2. **Number:** Integers and floating-point numbers (`42`, `3.14`, `Infinity`, `NaN`)
3. **BigInt:** Integers larger than 2^53 - 1 (`123n`, `BigInt(123)`)
4. **Boolean:** Logical values (`true`, `false`)
5. **Undefined:** Variable declared but not assigned a value
6. **Null:** Intentional absence of any value
7. **Symbol:** Unique and immutable identifier, often used for object property keys (`Symbol('id')`)

**Non-Primitive Type (1):**
8. **Object:** Collections of key-value pairs, including arrays, functions, dates, maps, sets, etc.

**Type Checking:**
```javascript
typeof "hello"        // "string"
typeof 42             // "number"
typeof true           // "boolean"
typeof undefined      // "undefined"
typeof null           // "object" (historical bug in JavaScript)
typeof Symbol()       // "symbol"
typeof {}             // "object"
typeof []             // "object" (arrays are objects)
typeof function(){}   // "function" (special case)
```

**Production Note:** Always use proper type checking. For arrays, use `Array.isArray()`. For null, check explicitly with `value === null`.

3. What is the difference between `var`, `let`, and `const`?

**Answer:**

| Feature | var | let | const |
|---------|-----|-----|-------|
| **Scope** | Function-scoped | Block-scoped | Block-scoped |
| **Hoisting** | Yes (initialized as undefined) | Yes (not initialized - TDZ) | Yes (not initialized - TDZ) |
| **Re-declaration** | Allowed | Not allowed | Not allowed |
| **Re-assignment** | Allowed | Allowed | Not allowed (for primitives) |
| **Global Object Property** | Yes (in browser) | No | No |

**Detailed Explanation:**

**var (legacy):**
```javascript
function varExample() {
  console.log(x); // undefined (hoisted)
  var x = 10;
  
  if (true) {
    var x = 20; // Same variable!
    console.log(x); // 20
  }
  console.log(x); // 20 (not block-scoped)
}
```

**let (modern, reassignable):**
```javascript
function letExample() {
  // console.log(y); // ReferenceError: Cannot access before initialization
  let y = 10;
  
  if (true) {
    let y = 20; // Different variable (block-scoped)
    console.log(y); // 20
  }
  console.log(y); // 10
}
```

**const (modern, constant):**
```javascript
const PI = 3.14159;
// PI = 3.14; // TypeError: Assignment to constant variable

const user = { name: "John" };
user.name = "Jane"; // Allowed! Object properties can change
user = {}; // TypeError: Assignment to constant variable
```

**Production Best Practice:**
- Use `const` by default
- Use `let` only when you need to reassign
- Never use `var` in modern code (causes bugs in large applications)

4. What is hoisting in JavaScript?

**Answer:**
Hoisting is JavaScript's default behavior of moving **declarations** (not initializations) to the top of their scope during the compilation phase, before code execution.

**How It Works:**
JavaScript engine processes code in two phases:
1. **Creation Phase:** Memory is allocated for variables and functions
2. **Execution Phase:** Code runs line by line

**Function Hoisting (Full Hoisting):**
```javascript
// This works because function declarations are fully hoisted
greet(); // "Hello!"

function greet() {
  console.log("Hello!");
}
```

**var Hoisting (Declaration Only):**
```javascript
console.log(name); // undefined (not ReferenceError)
var name = "John";
console.log(name); // "John"

// Behind the scenes:
// var name; // declaration hoisted
// console.log(name); // undefined
// name = "John"; // initialization stays in place
```

**let/const Hoisting (Temporal Dead Zone):**
```javascript
// console.log(age); // ReferenceError: Cannot access before initialization
let age = 25;

// Behind the scenes:
// let age; // hoisted but in TDZ
// console.log(age); // ReferenceError
// age = 25; // initialization
```

**Function Expressions vs Declarations:**
```javascript
// Function declaration - fully hoisted
foo(); // Works
function foo() { console.log("foo"); }

// Function expression - only variable is hoisted
// bar(); // TypeError: bar is not a function
var bar = function() { console.log("bar"); };
```

**Production Implications:**
- Always declare variables at the top of their scope for clarity
- Use function declarations when you need hoisting (e.g., mutual recursion)
- Prefer `let`/`const` to avoid hoisting confusion

5. What is the difference between `==` and `===`?

**Answer:**

| Operator | Name | Type Coercion | Use Case |
|----------|------|---------------|----------|
| `==` | Loose/Abstract Equality | Yes | Rarely used in production |
| `===` | Strict Equality | No | Default choice |

**`==` (Loose Equality) - Performs Type Coercion:**
```javascript
5 == "5"        // true (string "5" converted to number)
null == undefined  // true (special case)
0 == false      // true (false converted to 0)
"" == false     // true (both converted to 0)
[] == false     // true (array converted to 0)
[1] == 1        // true (array converted to number)
```

**`===` (Strict Equality) - No Type Coercion:**
```javascript
5 === "5"       // false (different types)
null === undefined  // false (different types)
0 === false     // false
"" === false    // false
[] === []       // false (different object references)
NaN === NaN     // false (NaN is not equal to itself)
```

**Type Coercion Rules (for `==`):**
1. If types are the same, compare like `===`
2. `null == undefined` (and vice versa)
3. String and Number: convert string to number
4. Boolean: convert to number (true→1, false→0)
5. Object and primitive: call `valueOf()` or `toString()` on object

**Production Best Practice:**
```javascript
// ❌ Bad - unpredictable
if (value == null) { } // checks both null and undefined

// ✅ Good - explicit
if (value === null || value === undefined) { }
// or
if (value == null) { } // Only acceptable use case

// ✅ Better - modern approach
if (value ?? defaultValue) { } // nullish coalescing
```

**ESLint Rule:** Most style guides enforce `eqeqeq` rule requiring `===` always.

6. What is the difference between `null` and `undefined`?

**Answer:**

| Aspect | undefined | null |
|--------|-----------|------|
| **Type** | `undefined` | `object` (historical bug) |
| **Meaning** | Variable declared but not assigned | Intentional absence of value |
| **Assignment** | JavaScript sets it automatically | Developer explicitly assigns it |
| **Function Return** | Default return value | Must be explicitly returned |

**undefined - Absence of Assignment:**
```javascript
let x;
console.log(x); // undefined

function test(param) {
  console.log(param); // undefined if not passed
}
test();

const obj = { name: "John" };
console.log(obj.age); // undefined (property doesn't exist)

function noReturn() {}
console.log(noReturn()); // undefined
```

**null - Intentional Absence:**
```javascript
let user = null; // Explicitly saying "no user"

function findUser(id) {
  const user = database.find(id);
  return user || null; // Explicitly return null if not found
}

// Clearing references
let data = { large: "object" };
data = null; // Help garbage collector
```

**Comparison:**
```javascript
null == undefined   // true (loose equality)
null === undefined  // false (different types)

typeof null         // "object" (JavaScript bug)
typeof undefined    // "undefined"

Boolean(null)       // false
Boolean(undefined)  // false
```

**Production Best Practice:**
```javascript
// ✅ Use null for intentional "empty" state
let selectedUser = null; // No user selected yet

// ✅ Check for both when dealing with optional values
function processData(data) {
  if (data === null || data === undefined) {
    return defaultData;
  }
  // or using nullish coalescing
  const result = data ?? defaultData;
}

// ✅ Destructuring with defaults
const { name = "Guest" } = user ?? {};
```

7. What are falsy values in JavaScript?

**Answer:**
Falsy values are values that are coerced to `false` when evaluated in a Boolean context. JavaScript has **exactly 8 falsy values**:

1. `false` - The boolean false
2. `0` - Zero (number)
3. `-0` - Negative zero
4. `0n` - BigInt zero
5. `""` - Empty string
6. `null` - Absence of value
7. `undefined` - Uninitialized value
8. `NaN` - Not a Number

**Everything else is truthy**, including:
```javascript
// Truthy values (often confusing)
true            // boolean true
{}              // empty object
[]              // empty array
"false"         // non-empty string
"0"             // non-empty string
42              // non-zero number
-42             // negative number
Infinity        // infinity
new Date()      // date object
function(){}    // function
```

**Common Pitfalls:**
```javascript
// ❌ These are truthy but might seem falsy
if ([]) {
  console.log("Empty array is truthy!"); // This runs
}

if ({}) {
  console.log("Empty object is truthy!"); // This runs
}

// ❌ String "false" is truthy
const value = "false";
if (value) {
  console.log("This runs!"); // Truthy
}

// ❌ Zero checks
const count = 0;
if (count) {
  // Won't run, but 0 might be a valid value
}
```

**Production Best Practices:**
```javascript
// ✅ Explicit checks for meaningful values
if (array.length > 0) { } // Instead of if (array)
if (Object.keys(obj).length > 0) { } // Instead of if (obj)
if (value !== null && value !== undefined) { } // Be explicit

// ✅ Use nullish coalescing for defaults
const count = userInput ?? 0; // Only replaces null/undefined
const count = userInput || 0; // Replaces all falsy (might replace valid 0!)

// ✅ Explicit boolean conversion when needed
const hasData = Boolean(data); // or !!data
```

**Type Coercion Examples:**
```javascript
Boolean(false)       // false
Boolean(0)           // false
Boolean("")          // false
Boolean(null)        // false
Boolean(undefined)   // false
Boolean(NaN)         // false

!!false              // false (double NOT operator)
!!0                  // false
!!"hello"            // true
```

8. What is type coercion?

**Answer:**
Type coercion is the automatic or implicit conversion of values from one data type to another by JavaScript. It happens when operators or functions expect a specific type but receive a different type.

**Types of Coercion:**
1. **Implicit Coercion:** JavaScript automatically converts types
2. **Explicit Coercion:** Developer intentionally converts types

**String Coercion:**
```javascript
// Implicit - using + operator
"5" + 3          // "53" (number to string)
"Hello" + true   // "Hellotrue"
"Value: " + null // "Value: null"

// Explicit
String(123)      // "123"
(123).toString() // "123"
"" + 123         // "123" (trick)
```

**Number Coercion:**
```javascript
// Implicit - using -, *, /, % operators
"5" - 2          // 3 (string to number)
"5" * "2"        // 10
"10" / "2"       // 5
"5" - true       // 4 (true becomes 1)

// + doesn't coerce to number if any operand is a string
"5" + 2          // "52" (number to string instead!)

// Explicit
Number("123")    // 123
Number("12.5")   // 12.5
Number("abc")    // NaN
parseInt("123")  // 123
parseFloat("12.5") // 12.5
+"123"           // 123 (unary plus trick)
```

**Boolean Coercion:**
```javascript
// Implicit - in conditions
if (value) { }
value ? "yes" : "no"
!value
value && otherValue
value || defaultValue

// Explicit
Boolean(1)       // true
Boolean(0)       // false
!!value          // double NOT trick
```

**Complex Coercion Examples:**
```javascript
// Array coercion
[1, 2] + [3, 4]  // "1,23,4" (arrays to strings, then concat)
[] + []          // "" (empty strings)
[] + {}          // "[object Object]"
{} + []          // 0 (depends on context!)

// Object coercion
const obj = {
  valueOf() { return 42; },
  toString() { return "hello"; }
};
obj + 1          // 43 (valueOf called)
String(obj)      // "hello" (toString called)

// Comparison coercion
"2" > "12"       // true (string comparison, not numeric)
"2" > 1          // true (coerced to numbers: 2 > 1)
```

**Production Pitfalls & Solutions:**
```javascript
// ❌ Problem: Unexpected concatenation
const total = "5" + 10;  // "510"

// ✅ Solution: Explicit conversion
const total = Number("5") + 10;  // 15
const total = parseInt("5") + 10; // 15

// ❌ Problem: Falsy number 0
const count = 0;
const result = count || 10;  // 10 (0 is falsy!)

// ✅ Solution: Nullish coalescing
const result = count ?? 10;  // 0 (only null/undefined replaced)

// ❌ Problem: Array/Object in conditions
if ([]) { /* always runs */ }

// ✅ Solution: Explicit checks
if (array.length > 0) { /* correct */ }

// ❌ Problem: NaN comparisons
NaN == NaN       // false
NaN === NaN      // false

// ✅ Solution: Use proper check
Number.isNaN(value)  // true if NaN
```

**Best Practices:**
- Always use explicit type conversion in production code for clarity
- Use `===` instead of `==` to avoid coercion issues
- Be cautious with `+` operator (use template literals instead)
- Validate and sanitize user input explicitly

9. What is the difference between primitive and non-primitive data types?

**Answer:**

**Primitive Types (Immutable, Pass by Value):**
- String, Number, BigInt, Boolean, Undefined, Null, Symbol
- Stored directly in the **stack**
- Immutable (cannot be changed, only replaced)
- Compared by **value**
- Each variable holds its own copy

**Non-Primitive Types (Mutable, Pass by Reference):**
- Object, Array, Function, Date, RegExp, Map, Set, etc.
- Stored in the **heap**, reference stored in stack
- Mutable (can be modified)
- Compared by **reference** (memory address)
- Variables hold references to the same object

**Memory & Mutability:**
```javascript
// Primitives - Immutable
let x = 10;
let y = x;     // Copy of value
y = 20;
console.log(x); // 10 (unchanged)

let str = "hello";
str[0] = "H";  // No effect (strings are immutable)
console.log(str); // "hello"
str = "Hello"; // Must create new string

// Objects - Mutable
let obj1 = { name: "John" };
let obj2 = obj1;  // Copy of reference
obj2.name = "Jane";
console.log(obj1.name); // "Jane" (both point to same object)

let arr1 = [1, 2, 3];
let arr2 = arr1;
arr2.push(4);
console.log(arr1); // [1, 2, 3, 4]
```

**Comparison:**
```javascript
// Primitives - Value comparison
10 === 10        // true
"hello" === "hello"  // true

// Objects - Reference comparison
{} === {}        // false (different references)
[] === []        // false
const a = { x: 1 };
const b = { x: 1 };
a === b          // false (different objects)

const c = a;
c === a          // true (same reference)
```

**Function Parameters:**
```javascript
// Primitive - Pass by Value
function changePrimitive(num) {
  num = 100; // Local copy changed
}
let x = 10;
changePrimitive(x);
console.log(x); // 10 (unchanged)

// Object - Pass by Reference
function changeObject(obj) {
  obj.name = "Changed"; // Original object modified
}
let user = { name: "John" };
changeObject(user);
console.log(user.name); // "Changed"

// Reassignment doesn't affect original
function reassignObject(obj) {
  obj = { name: "New" }; // Local reference changed
}
let person = { name: "John" };
reassignObject(person);
console.log(person.name); // "John" (unchanged)
```

**Production Implications:**
```javascript
// ❌ Problem: Unintended mutations
function addUser(users, user) {
  users.push(user); // Mutates original array!
  return users;
}

// ✅ Solution: Create copies
function addUser(users, user) {
  return [...users, user]; // New array
}

// ✅ Deep cloning objects
const original = { user: { name: "John" } };
const shallow = { ...original }; // Shallow copy
shallow.user.name = "Jane"; // Mutates original!

const deep = JSON.parse(JSON.stringify(original)); // Deep copy
// or use structuredClone() (modern)
const deep = structuredClone(original);

// ✅ Freezing objects (immutability)
const config = Object.freeze({ apiUrl: "..." });
// config.apiUrl = "new"; // TypeError in strict mode
```

**Memory Efficiency:**
```javascript
// Primitives - Small, fast
let a = 1;
let b = 1; // New value in stack

// Objects - Reference sharing
let user1 = { name: "John", age: 30 };
let user2 = user1; // No copy, just reference
// More memory efficient for large data
```

10. What is the temporal dead zone?

**Answer:**
The Temporal Dead Zone (TDZ) is the period between entering a scope and the actual declaration of a variable, during which the variable cannot be accessed. It applies to `let`, `const`, and `class` declarations.

**How It Works:**
```javascript
// TDZ starts here for 'x'
console.log(x); // ReferenceError: Cannot access 'x' before initialization
let x = 10;     // TDZ ends here
console.log(x); // 10
```

**Behind the Scenes:**
```javascript
{
  // TDZ for 'name' starts at block entry
  // Hoisted but not initialized
  
  // console.log(name); // ReferenceError
  // const temp = name; // ReferenceError
  // name = "value";    // ReferenceError
  
  let name = "John"; // TDZ ends - initialization occurs
  console.log(name); // "John" - now accessible
}
```

**Comparison with var:**
```javascript
// var - No TDZ (initialized as undefined)
console.log(varVariable); // undefined
var varVariable = "value";

// let - TDZ exists
console.log(letVariable); // ReferenceError
let letVariable = "value";

// const - TDZ exists (and must be initialized)
// const constVariable; // SyntaxError: Missing initializer
const constVariable = "value";
```

**TDZ in Different Scopes:**
```javascript
// Function parameters TDZ
function example(a = b, b = 2) {
  // 'b' is in TDZ when 'a' is initialized
  return [a, b];
}
// example(); // ReferenceError: Cannot access 'b' before initialization

function correct(a = 2, b = a) {
  return [a, b]; // Works: 'a' is initialized when 'b' needs it
}
correct(); // [2, 2]

// Block scope TDZ
{
  // TDZ for x starts
  const func = () => console.log(x); // Defined but not called
  // func(); // ReferenceError if called here
  let x = 10; // TDZ ends
  func(); // 10 - now works
}
```

**typeof and TDZ:**
```javascript
// var - typeof works
console.log(typeof varVar); // "undefined"
var varVar = 10;

// let - typeof causes ReferenceError in TDZ
console.log(typeof letVar); // ReferenceError (not "undefined")
let letVar = 10;

// Variable that doesn't exist - typeof returns "undefined"
console.log(typeof nonExistent); // "undefined" (no error)
```

**Production Benefits:**
```javascript
// ✅ Catches typos and wrong order bugs
function calculatePrice(quantity) {
  const total = quantity * price; // ReferenceError: good!
  // ... 100 lines of code ...
  let price = 10; // Caught immediately
}

// ❌ With var, this silently fails
function calculatePriceVar(quantity) {
  var total = quantity * price; // undefined * quantity = NaN
  // ... 100 lines of code ...
  var price = 10; // Bug found much later
}

// ✅ Prevents use before declaration
class Dog {
  constructor() {
    this.bark(); // ReferenceError if bark is let/const below
  }
}

// ✅ Enforces initialization order
const config = {
  apiUrl: API_BASE_URL, // Must be declared before
  timeout: 5000
};
const API_BASE_URL = "https://api.example.com";
```

**Best Practices:**
- Declare variables at the top of their scope for clarity
- Initialize `const` immediately when declared
- Use linters (ESLint) to catch TDZ issues: `no-use-before-define` rule
- The TDZ is a feature that helps catch bugs early in development
