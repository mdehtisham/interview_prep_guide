## Arrays
39. What are arrays in JavaScript?

**Answer:**
Arrays are ordered, indexed collections that can hold multiple values of any type. In JavaScript, arrays are actually special objects with numeric keys and a length property, providing powerful methods for data manipulation.

**Array Basics:**

```javascript
// Creating arrays
const empty = [];
const numbers = [1, 2, 3, 4, 5];
const mixed = [1, 'two', true, null, { key: 'value' }, [1, 2]];

// Accessing elements (zero-indexed)
console.log(numbers[0]);  // 1 (first element)
console.log(numbers[4]);  // 5 (last element)
console.log(numbers[-1]); // undefined (no negative indexing like Python)

// Length property
console.log(numbers.length); // 5

// Modifying elements
numbers[0] = 10;
console.log(numbers); // [10, 2, 3, 4, 5]

// Arrays are objects
console.log(typeof numbers); // 'object'
console.log(Array.isArray(numbers)); // true (proper check)
```

**Array Creation Methods:**

```javascript
// 1. Array literal (most common)
const arr1 = [1, 2, 3];

// 2. Array constructor
const arr2 = new Array(1, 2, 3);
console.log(arr2); // [1, 2, 3]

// ⚠️ Single number creates empty array with that length
const arr3 = new Array(5);
console.log(arr3); // [ <5 empty items> ]
console.log(arr3.length); // 5

// 3. Array.of() - creates array from arguments
const arr4 = Array.of(1, 2, 3);
console.log(arr4); // [1, 2, 3]

const arr5 = Array.of(5); // Creates [5], not empty array
console.log(arr5); // [5]

// 4. Array.from() - creates from iterable or array-like
const arr6 = Array.from('hello');
console.log(arr6); // ['h', 'e', 'l', 'l', 'o']

const arr7 = Array.from({ length: 3 }, (_, i) => i * 2);
console.log(arr7); // [0, 2, 4]

// 5. Spread operator with iterable
const arr8 = [...'hello'];
console.log(arr8); // ['h', 'e', 'l', 'l', 'o']

const arr9 = [...new Set([1, 2, 2, 3])];
console.log(arr9); // [1, 2, 3] (duplicates removed)
```

**Array Characteristics:**

```javascript
// 1. Dynamic size
const arr = [1, 2, 3];
arr.push(4);  // Can grow
console.log(arr); // [1, 2, 3, 4]

arr.length = 2; // Can shrink
console.log(arr); // [1, 2]

// 2. Heterogeneous (mixed types)
const mixed = [
  42,                    // number
  'hello',              // string
  true,                 // boolean
  null,                 // null
  undefined,            // undefined
  { key: 'value' },    // object
  [1, 2, 3],           // array
  () => console.log('hi') // function
];

// 3. Sparse arrays (gaps in indices)
const sparse = [];
sparse[0] = 'first';
sparse[10] = 'eleventh';
console.log(sparse.length); // 11 (not 2!)
console.log(sparse); // ['first', <9 empty items>, 'eleventh']

// Empty slots are different from undefined
console.log(sparse[5]); // undefined
console.log(5 in sparse); // false (no property at index 5)

const explicit = [1, undefined, 3];
console.log(1 in explicit); // true (undefined is explicitly set)

// 4. Arrays are objects with numeric keys
const arr2 = ['a', 'b', 'c'];
console.log(arr2['0']); // 'a' (string key works)
console.log(arr2[0]);   // 'a' (numeric key)

// Can have non-numeric properties (but shouldn't)
arr2.customProp = 'value';
console.log(arr2.customProp); // 'value'
console.log(arr2.length); // 3 (non-numeric props don't affect length)
```

**Common Array Operations:**

**1. Adding Elements:**
```javascript
const arr = [1, 2, 3];

// Add to end
arr.push(4, 5);
console.log(arr); // [1, 2, 3, 4, 5]

// Add to beginning
arr.unshift(0);
console.log(arr); // [0, 1, 2, 3, 4, 5]

// Add at specific position
arr.splice(3, 0, 2.5); // At index 3, delete 0, insert 2.5
console.log(arr); // [0, 1, 2, 2.5, 3, 4, 5]

// Using length property
arr[arr.length] = 6;
console.log(arr); // [0, 1, 2, 2.5, 3, 4, 5, 6]
```

**2. Removing Elements:**
```javascript
const arr = [1, 2, 3, 4, 5];

// Remove from end
const last = arr.pop();
console.log(last); // 5
console.log(arr); // [1, 2, 3, 4]

// Remove from beginning
const first = arr.shift();
console.log(first); // 1
console.log(arr); // [2, 3, 4]

// Remove at specific position
arr.splice(1, 1); // At index 1, delete 1 element
console.log(arr); // [2, 4]

// Remove using delete (creates hole - avoid)
const arr2 = [1, 2, 3];
delete arr2[1];
console.log(arr2); // [1, <1 empty item>, 3]
console.log(arr2.length); // 3 (length unchanged!)
```

**3. Searching:**
```javascript
const fruits = ['apple', 'banana', 'orange', 'banana'];

// indexOf - first occurrence
console.log(fruits.indexOf('banana')); // 1

// lastIndexOf - last occurrence
console.log(fruits.lastIndexOf('banana')); // 3

// includes - boolean check
console.log(fruits.includes('apple')); // true
console.log(fruits.includes('grape')); // false

// find - first element matching condition
const numbers = [1, 5, 10, 15, 20];
const found = numbers.find(n => n > 10);
console.log(found); // 15

// findIndex - index of first match
const foundIndex = numbers.findIndex(n => n > 10);
console.log(foundIndex); // 3

// some - at least one matches
console.log(numbers.some(n => n > 15)); // true

// every - all match
console.log(numbers.every(n => n > 0)); // true
console.log(numbers.every(n => n > 10)); // false
```

**4. Transforming:**
```javascript
const numbers = [1, 2, 3, 4, 5];

// map - transform each element
const doubled = numbers.map(n => n * 2);
console.log(doubled); // [2, 4, 6, 8, 10]

// filter - keep elements matching condition
const evens = numbers.filter(n => n % 2 === 0);
console.log(evens); // [2, 4]

// reduce - accumulate to single value
const sum = numbers.reduce((acc, n) => acc + n, 0);
console.log(sum); // 15

const product = numbers.reduce((acc, n) => acc * n, 1);
console.log(product); // 120

// flatMap - map then flatten
const nested = [1, 2, 3];
const result = nested.flatMap(n => [n, n * 2]);
console.log(result); // [1, 2, 2, 4, 3, 6]
```

**5. Sorting and Reversing:**
```javascript
const numbers = [3, 1, 4, 1, 5, 9, 2, 6];

// sort - sorts in place (⚠️ lexicographic by default)
numbers.sort();
console.log(numbers); // [1, 1, 2, 3, 4, 5, 6, 9]

const nums = [1, 5, 10, 25, 100];
nums.sort(); // ⚠️ Problem!
console.log(nums); // [1, 10, 100, 25, 5] (lexicographic!)

// Proper numeric sort
nums.sort((a, b) => a - b);
console.log(nums); // [1, 5, 10, 25, 100]

// reverse - reverses in place
nums.reverse();
console.log(nums); // [100, 25, 10, 5, 1]

// Sort objects
const people = [
  { name: 'John', age: 30 },
  { name: 'Jane', age: 25 },
  { name: 'Bob', age: 35 }
];

people.sort((a, b) => a.age - b.age);
console.log(people);
// [{ name: 'Jane', age: 25 }, { name: 'John', age: 30 }, { name: 'Bob', age: 35 }]
```

**6. Extracting Portions:**
```javascript
const arr = [1, 2, 3, 4, 5];

// slice - extract portion (doesn't modify original)
const portion = arr.slice(1, 4);
console.log(portion); // [2, 3, 4]
console.log(arr); // [1, 2, 3, 4, 5] (unchanged)

// Negative indices
console.log(arr.slice(-2)); // [4, 5] (last 2)
console.log(arr.slice(0, -1)); // [1, 2, 3, 4] (all except last)

// Copy array
const copy = arr.slice();
console.log(copy); // [1, 2, 3, 4, 5]
```

**7. Joining and Splitting:**
```javascript
// join - array to string
const arr = ['Hello', 'World'];
console.log(arr.join(' ')); // 'Hello World'
console.log(arr.join('-')); // 'Hello-World'
console.log(arr.join('')); // 'HelloWorld'

// String split - string to array
const str = 'Hello World';
console.log(str.split(' ')); // ['Hello', 'World']
console.log(str.split('')); // ['H', 'e', 'l', 'l', 'o', ' ', 'W', 'o', 'r', 'l', 'd']

// CSV parsing
const csv = 'name,age,city';
console.log(csv.split(',')); // ['name', 'age', 'city']
```

**8. Flattening:**
```javascript
// flat - flatten nested arrays
const nested = [1, [2, 3], [4, [5, 6]]];

console.log(nested.flat());    // [1, 2, 3, 4, [5, 6]] (1 level)
console.log(nested.flat(2));   // [1, 2, 3, 4, 5, 6] (2 levels)
console.log(nested.flat(Infinity)); // [1, 2, 3, 4, 5, 6] (all levels)

// Remove empty slots
const sparse = [1, , 3, , 5];
console.log(sparse.flat()); // [1, 3, 5] (removes empty slots)
```

**Multi-dimensional Arrays:**

```javascript
// 2D array (matrix)
const matrix = [
  [1, 2, 3],
  [4, 5, 6],
  [7, 8, 9]
];

console.log(matrix[0][0]); // 1
console.log(matrix[1][2]); // 6
console.log(matrix[2][1]); // 8

// Iterate 2D array
for (let i = 0; i < matrix.length; i++) {
  for (let j = 0; j < matrix[i].length; j++) {
    console.log(matrix[i][j]);
  }
}

// Using forEach
matrix.forEach(row => {
  row.forEach(cell => {
    console.log(cell);
  });
});

// Create 3x3 matrix of zeros
const zeros = Array.from({ length: 3 }, () => Array(3).fill(0));
console.log(zeros);
// [[0, 0, 0], [0, 0, 0], [0, 0, 0]]
```

**Array Iteration:**

```javascript
const arr = ['a', 'b', 'c'];

// 1. for loop
for (let i = 0; i < arr.length; i++) {
  console.log(i, arr[i]);
}

// 2. for...of (values)
for (const value of arr) {
  console.log(value);
}

// 3. for...in (indices - avoid for arrays)
for (const index in arr) {
  console.log(index, arr[index]);
}

// 4. forEach
arr.forEach((value, index, array) => {
  console.log(index, value);
});

// 5. map (when transforming)
const upper = arr.map(s => s.toUpperCase());

// 6. entries() - [index, value] pairs
for (const [index, value] of arr.entries()) {
  console.log(index, value);
}

// 7. keys() - indices
for (const index of arr.keys()) {
  console.log(index);
}

// 8. values() - values
for (const value of arr.values()) {
  console.log(value);
}
```

**Performance Considerations:**

```javascript
// Fast operations (O(1))
arr.push(item);      // Add to end
arr.pop();           // Remove from end
arr[index];          // Access by index
arr.length;          // Get length

// Slow operations (O(n))
arr.unshift(item);   // Add to beginning (shifts all elements)
arr.shift();         // Remove from beginning (shifts all elements)
arr.splice(i, 0, x); // Insert in middle (shifts elements)

// Very slow operations (O(n) or worse)
arr.indexOf(item);   // Linear search
arr.includes(item);  // Linear search
arr.sort();          // O(n log n)
```

**Production Use Cases:**

**1. Data Processing Pipeline:**
```javascript
const users = [
  { name: 'John', age: 25, active: true },
  { name: 'Jane', age: 30, active: false },
  { name: 'Bob', age: 35, active: true }
];

// Chain operations
const result = users
  .filter(user => user.active)
  .map(user => user.name)
  .sort();

console.log(result); // ['Bob', 'John']
```

**2. Grouping Data:**
```javascript
const transactions = [
  { category: 'food', amount: 50 },
  { category: 'transport', amount: 30 },
  { category: 'food', amount: 20 },
  { category: 'transport', amount: 40 }
];

const grouped = transactions.reduce((acc, t) => {
  if (!acc[t.category]) {
    acc[t.category] = [];
  }
  acc[t.category].push(t);
  return acc;
}, {});

console.log(grouped);
// { food: [...], transport: [...] }
```

**3. Pagination:**
```javascript
function paginate(array, pageSize, pageNumber) {
  const start = (pageNumber - 1) * pageSize;
  const end = start + pageSize;
  return array.slice(start, end);
}

const items = Array.from({ length: 100 }, (_, i) => i + 1);
console.log(paginate(items, 10, 1)); // [1, 2, ..., 10]
console.log(paginate(items, 10, 2)); // [11, 12, ..., 20]
```

**Best Practices:**
- Use array literals `[]` instead of `new Array()`
- Use `Array.isArray()` to check for arrays
- Prefer immutable methods (`map`, `filter`, `slice`) over mutating ones
- Use `for...of` for iteration, not `for...in`
- Avoid sparse arrays when possible
- Use appropriate methods for performance
- Consider `Set` for unique values
- Use typed arrays for binary data

**Key Takeaways:**
- Arrays are ordered, indexed collections of any type
- Actually objects with numeric keys and special length property
- Dynamic size, can grow or shrink
- Rich set of built-in methods for manipulation
- Zero-indexed (first element at index 0)
- Essential data structure for most JavaScript programs
- Understanding array methods is crucial for effective programming

40. What is the difference between array methods: map, filter, and reduce?

**Answer:**
`map()`, `filter()`, and `reduce()` are fundamental array methods for functional programming in JavaScript. Each serves a different purpose: `map()` transforms elements, `filter()` selects elements, and `reduce()` accumulates elements into a single value.

**Quick Comparison:**

| Method | Purpose | Returns | Use When |
|--------|---------|---------|----------|
| **map()** | Transform each element | New array (same length) | Converting/modifying all elements |
| **filter()** | Select elements | New array (≤ length) | Keeping elements that match criteria |
| **reduce()** | Accumulate to single value | Any type | Calculating sum, building object, etc. |

**map() - Transform Elements:**

`map()` creates a new array by applying a function to each element. The resulting array has the **same length** as the original.

```javascript
const numbers = [1, 2, 3, 4, 5];

// Transform each element
const doubled = numbers.map(n => n * 2);
console.log(doubled); // [2, 4, 6, 8, 10]

// Original unchanged
console.log(numbers); // [1, 2, 3, 4, 5]

// Map with objects
const users = [
  { firstName: 'John', lastName: 'Doe' },
  { firstName: 'Jane', lastName: 'Smith' }
];

const fullNames = users.map(user => `${user.firstName} ${user.lastName}`);
console.log(fullNames); // ['John Doe', 'Jane Smith']

// Map with index
const indexed = numbers.map((num, index) => ({
  index,
  value: num,
  doubled: num * 2
}));
console.log(indexed);
// [
//   { index: 0, value: 1, doubled: 2 },
//   { index: 1, value: 2, doubled: 4 },
//   ...
// ]
```

**filter() - Select Elements:**

`filter()` creates a new array with only elements that pass a test. The resulting array has **equal or smaller length**.

```javascript
const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// Keep only even numbers
const evens = numbers.filter(n => n % 2 === 0);
console.log(evens); // [2, 4, 6, 8, 10]

// Keep only numbers greater than 5
const greaterThanFive = numbers.filter(n => n > 5);
console.log(greaterThanFive); // [6, 7, 8, 9, 10]

// Filter with objects
const users = [
  { name: 'John', age: 25, active: true },
  { name: 'Jane', age: 30, active: false },
  { name: 'Bob', age: 35, active: true }
];

const activeUsers = users.filter(user => user.active);
console.log(activeUsers);
// [{ name: 'John', age: 25, active: true }, { name: 'Bob', age: 35, active: true }]

// Filter with multiple conditions
const youngActiveUsers = users.filter(user => user.active && user.age < 30);
console.log(youngActiveUsers);
// [{ name: 'John', age: 25, active: true }]

// Filter falsy values
const mixed = [0, 1, false, 2, '', 3, null, undefined, 4];
const truthy = mixed.filter(Boolean);
console.log(truthy); // [1, 2, 3, 4]
```

**reduce() - Accumulate to Single Value:**

`reduce()` executes a reducer function on each element, resulting in a **single output value**.

```javascript
const numbers = [1, 2, 3, 4, 5];

// Sum all numbers
const sum = numbers.reduce((accumulator, current) => {
  return accumulator + current;
}, 0); // 0 is initial value
console.log(sum); // 15

// Product of all numbers
const product = numbers.reduce((acc, curr) => acc * curr, 1);
console.log(product); // 120

// Find maximum
const max = numbers.reduce((max, curr) => curr > max ? curr : max);
console.log(max); // 5

// Count occurrences
const fruits = ['apple', 'banana', 'apple', 'orange', 'banana', 'apple'];
const count = fruits.reduce((acc, fruit) => {
  acc[fruit] = (acc[fruit] || 0) + 1;
  return acc;
}, {});
console.log(count); // { apple: 3, banana: 2, orange: 1 }

// Group by property
const people = [
  { name: 'John', age: 25 },
  { name: 'Jane', age: 30 },
  { name: 'Bob', age: 25 }
];

const groupedByAge = people.reduce((acc, person) => {
  const age = person.age;
  if (!acc[age]) {
    acc[age] = [];
  }
  acc[age].push(person);
  return acc;
}, {});
console.log(groupedByAge);
// {
//   25: [{ name: 'John', age: 25 }, { name: 'Bob', age: 25 }],
//   30: [{ name: 'Jane', age: 30 }]
// }

// Flatten array
const nested = [[1, 2], [3, 4], [5, 6]];
const flattened = nested.reduce((acc, curr) => acc.concat(curr), []);
console.log(flattened); // [1, 2, 3, 4, 5, 6]
```

**Detailed Comparison:**

**1. Return Value:**
```javascript
const numbers = [1, 2, 3, 4, 5];

// map: always returns array with same length
const mapped = numbers.map(n => n * 2);
console.log(mapped.length); // 5 (same as original)

// filter: returns array with ≤ length
const filtered = numbers.filter(n => n > 3);
console.log(filtered.length); // 2 (smaller)

// reduce: returns single value (any type)
const reduced = numbers.reduce((sum, n) => sum + n, 0);
console.log(typeof reduced); // 'number'

// reduce can return array, object, etc.
const reducedArray = numbers.reduce((acc, n) => [...acc, n * 2], []);
console.log(reducedArray); // [2, 4, 6, 8, 10] (like map)
```

**2. Callback Parameters:**
```javascript
const arr = [10, 20, 30];

// map(element, index, array)
arr.map((element, index, array) => {
  console.log('Element:', element);
  console.log('Index:', index);
  console.log('Array:', array);
  return element * 2;
});

// filter(element, index, array)
arr.filter((element, index, array) => {
  console.log('Element:', element);
  console.log('Index:', index);
  console.log('Array:', array);
  return element > 15;
});

// reduce(accumulator, element, index, array)
arr.reduce((accumulator, element, index, array) => {
  console.log('Accumulator:', accumulator);
  console.log('Element:', element);
  console.log('Index:', index);
  console.log('Array:', array);
  return accumulator + element;
}, 0); // Initial value
```

**3. Initial Value (reduce only):**
```javascript
const numbers = [1, 2, 3, 4, 5];

// With initial value
const sum1 = numbers.reduce((acc, n) => acc + n, 0);
console.log(sum1); // 15

// Without initial value (uses first element)
const sum2 = numbers.reduce((acc, n) => acc + n);
console.log(sum2); // 15

// ⚠️ Empty array without initial value throws error
const empty = [];
// empty.reduce((acc, n) => acc + n); // TypeError!
const safeSum = empty.reduce((acc, n) => acc + n, 0); // 0 (safe)

// Initial value type can differ from array elements
const strings = ['1', '2', '3'];
const numberSum = strings.reduce((acc, str) => acc + Number(str), 0);
console.log(numberSum); // 6 (number, not string)
```

**Combining Methods (Chaining):**

```javascript
const users = [
  { name: 'John', age: 25, salary: 50000, active: true },
  { name: 'Jane', age: 30, salary: 60000, active: false },
  { name: 'Bob', age: 35, salary: 70000, active: true },
  { name: 'Alice', age: 28, salary: 55000, active: true }
];

// Complex transformation pipeline
const result = users
  .filter(user => user.active)              // Keep active users
  .map(user => user.salary)                 // Extract salaries
  .reduce((sum, salary) => sum + salary, 0); // Sum salaries

console.log(result); // 175000

// Get names of active users over 25, sorted
const activeNames = users
  .filter(user => user.active && user.age > 25)
  .map(user => user.name)
  .sort();

console.log(activeNames); // ['Alice', 'Bob']

// Average salary of active users
const avgSalary = users
  .filter(user => user.active)
  .reduce((acc, user, idx, arr) => {
    acc += user.salary;
    if (idx === arr.length - 1) {
      return acc / arr.length;
    }
    return acc;
  }, 0);

console.log(avgSalary); // 58333.33
```

**When to Use Each:**

**Use map() when:**
```javascript
// Converting data types
const strings = [1, 2, 3];
const numbers = strings.map(String); // ['1', '2', '3']

// Extracting properties
const users = [{ name: 'John' }, { name: 'Jane' }];
const names = users.map(u => u.name); // ['John', 'Jane']

// Transforming objects
const products = [{ price: 10 }, { price: 20 }];
const withTax = products.map(p => ({ ...p, total: p.price * 1.1 }));

// Adding computed properties
const items = [1, 2, 3];
const enriched = items.map((n, i) => ({ id: i, value: n, squared: n ** 2 }));
```

**Use filter() when:**
```javascript
// Removing unwanted elements
const numbers = [1, 2, 3, 4, 5];
const evens = numbers.filter(n => n % 2 === 0);

// Finding matches
const users = [{ age: 25 }, { age: 30 }, { age: 35 }];
const adults = users.filter(u => u.age >= 18);

// Removing duplicates (with indexOf)
const arr = [1, 2, 2, 3, 3, 4];
const unique = arr.filter((n, i, self) => self.indexOf(n) === i);

// Removing falsy values
const mixed = [0, 1, false, 2, '', 3];
const truthy = mixed.filter(Boolean);
```

**Use reduce() when:**
```javascript
// Calculating single value
const numbers = [1, 2, 3, 4, 5];
const sum = numbers.reduce((a, b) => a + b, 0);

// Building object/map
const pairs = [['a', 1], ['b', 2]];
const obj = pairs.reduce((acc, [k, v]) => ({ ...acc, [k]: v }), {});

// Grouping/categorizing
const items = [{ type: 'A' }, { type: 'B' }, { type: 'A' }];
const grouped = items.reduce((acc, item) => {
  (acc[item.type] = acc[item.type] || []).push(item);
  return acc;
}, {});

// Complex aggregations
const sales = [{ amount: 100 }, { amount: 200 }];
const stats = sales.reduce((acc, sale) => ({
  total: acc.total + sale.amount,
  count: acc.count + 1,
  average: (acc.total + sale.amount) / (acc.count + 1)
}), { total: 0, count: 0, average: 0 });
```

**Performance Considerations:**

```javascript
const large = Array.from({ length: 1000000 }, (_, i) => i);

// Single operation is faster
console.time('map then filter');
const result1 = large
  .map(n => n * 2)
  .filter(n => n > 100);
console.timeEnd('map then filter'); // ~50ms (processes array twice)

// Combined operation is more efficient
console.time('single reduce');
const result2 = large.reduce((acc, n) => {
  const doubled = n * 2;
  if (doubled > 100) {
    acc.push(doubled);
  }
  return acc;
}, []);
console.timeEnd('single reduce'); // ~25ms (processes array once)

// But readability often matters more than micro-optimizations
// Use chaining for clarity unless performance is critical
```

**Common Patterns:**

**1. map + filter (could be reduce):**
```javascript
const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// Using map + filter
const result1 = numbers
  .map(n => n * 2)
  .filter(n => n > 10);

// Using reduce (more efficient but less readable)
const result2 = numbers.reduce((acc, n) => {
  const doubled = n * 2;
  if (doubled > 10) {
    acc.push(doubled);
  }
  return acc;
}, []);

console.log(result1); // [12, 14, 16, 18, 20]
console.log(result2); // [12, 14, 16, 18, 20]
```

**2. Counting with reduce:**
```javascript
const votes = ['yes', 'no', 'yes', 'yes', 'no'];

const tally = votes.reduce((acc, vote) => {
  acc[vote] = (acc[vote] || 0) + 1;
  return acc;
}, {});

console.log(tally); // { yes: 3, no: 2 }
```

**3. Creating lookup map:**
```javascript
const users = [
  { id: 1, name: 'John' },
  { id: 2, name: 'Jane' },
  { id: 3, name: 'Bob' }
];

const userMap = users.reduce((acc, user) => {
  acc[user.id] = user;
  return acc;
}, {});

console.log(userMap[2]); // { id: 2, name: 'Jane' }
```

**Production Use Cases:**

**1. Data Transformation Pipeline:**
```javascript
const rawData = [
  { date: '2024-01-01', amount: 100, status: 'completed' },
  { date: '2024-01-02', amount: 200, status: 'pending' },
  { date: '2024-01-03', amount: 150, status: 'completed' }
];

const totalCompleted = rawData
  .filter(tx => tx.status === 'completed')
  .map(tx => tx.amount)
  .reduce((sum, amount) => sum + amount, 0);

console.log(totalCompleted); // 250
```

**2. React Component Rendering:**
```javascript
const TodoList = ({ todos }) => {
  // Map items to React elements
  const todoItems = todos
    .filter(todo => !todo.deleted)
    .map(todo => (
      <TodoItem key={todo.id} todo={todo} />
    ));
  
  return <ul>{todoItems}</ul>;
};
```

**3. API Response Processing:**
```javascript
async function getUserStats(userIds) {
  const users = await fetchUsers(userIds);
  
  return users
    .filter(user => user.active)
    .reduce((stats, user) => ({
      totalAge: stats.totalAge + user.age,
      totalSalary: stats.totalSalary + user.salary,
      count: stats.count + 1
    }), { totalAge: 0, totalSalary: 0, count: 0 });
}
```

**Best Practices:**
- Use **map()** for transformations (1-to-1 mapping)
- Use **filter()** for selections (keeping subset)
- Use **reduce()** for aggregations (many-to-one)
- Chain methods for readability, use reduce for performance
- Always provide initial value to reduce for safety
- Consider readability over micro-optimizations
- Use arrow functions for concise callbacks
- Don't mutate original array (all three return new arrays/values)

**Key Takeaways:**
- **map()**: Transform each element → new array (same length)
- **filter()**: Select elements → new array (≤ length)
- **reduce()**: Accumulate to single value → any type
- All are pure functions (don't modify original)
- Can be chained for complex data transformations
- Fundamental to functional programming in JavaScript
- Choose based on desired output: array transformation, subset, or single value

41. What is the difference between forEach and map?

**Answer:**
`forEach()` and `map()` are both array iteration methods, but they have fundamentally different purposes: `forEach()` executes a function for each element (side effects), while `map()` transforms elements and returns a new array.

**Quick Comparison:**

| Feature | forEach() | map() |
|---------|-----------|-------|
| **Returns** | `undefined` | New array |
| **Purpose** | Execute side effects | Transform elements |
| **Creates new array** | No | Yes |
| **Chainable** | No | Yes |
| **Use for** | Logging, updating external state | Data transformation |
| **Return value matters** | No | Yes |

**forEach() - Execute Side Effects:**

`forEach()` executes a function for each element but **returns `undefined`**. Used for side effects like logging, updating external state, or DOM manipulation.

```javascript
const numbers = [1, 2, 3, 4, 5];

// forEach returns undefined
const result = numbers.forEach(n => {
  console.log(n * 2);
});

console.log(result); // undefined (no return value!)

// forEach is for side effects
let sum = 0;
numbers.forEach(n => {
  sum += n; // Modifying external variable
});
console.log(sum); // 15

// Common use: logging
numbers.forEach((num, index) => {
  console.log(`Index ${index}: ${num}`);
});

// DOM manipulation
const elements = document.querySelectorAll('.item');
elements.forEach(el => {
  el.classList.add('active'); // Side effect: modifying DOM
});

// Updating external object
const users = [{ name: 'John' }, { name: 'Jane' }];
const userMap = {};
users.forEach(user => {
  userMap[user.name] = user; // Side effect: modifying external object
});
```

**map() - Transform Elements:**

`map()` creates a **new array** with transformed elements. Used for data transformation.

```javascript
const numbers = [1, 2, 3, 4, 5];

// map returns new array
const doubled = numbers.map(n => n * 2);
console.log(doubled); // [2, 4, 6, 8, 10]

// Original array unchanged
console.log(numbers); // [1, 2, 3, 4, 5]

// Transform objects
const users = [
  { firstName: 'John', lastName: 'Doe' },
  { firstName: 'Jane', lastName: 'Smith' }
];

const fullNames = users.map(user => 
  `${user.firstName} ${user.lastName}`
);
console.log(fullNames); // ['John Doe', 'Jane Smith']

// Extract properties
const ids = users.map(user => user.id);

// Add computed properties
const enriched = numbers.map((n, i) => ({
  index: i,
  value: n,
  squared: n ** 2
}));
```

**Side-by-Side Comparison:**

```javascript
const numbers = [1, 2, 3, 4, 5];

// ❌ Wrong: forEach doesn't return new array
const doubled1 = numbers.forEach(n => n * 2);
console.log(doubled1); // undefined

// ✅ Correct: map returns new array
const doubled2 = numbers.map(n => n * 2);
console.log(doubled2); // [2, 4, 6, 8, 10]

// ❌ Wrong: map for side effects (wasteful)
numbers.map(n => {
  console.log(n); // Works but creates unnecessary array
});

// ✅ Correct: forEach for side effects
numbers.forEach(n => {
  console.log(n); // Appropriate use
});
```

**Return Value Behavior:**

```javascript
const numbers = [1, 2, 3];

// forEach: return value in callback is ignored
const forEachResult = numbers.forEach(n => {
  return n * 2; // This return value is ignored!
});
console.log(forEachResult); // undefined

// map: return value in callback is used
const mapResult = numbers.map(n => {
  return n * 2; // This return value creates new array
});
console.log(mapResult); // [2, 4, 6]

// Explicit return required in map (with block body)
const doubled = numbers.map(n => {
  const result = n * 2;
  return result; // Must return!
});

// Implicit return with arrow function
const doubled2 = numbers.map(n => n * 2); // Implicit return
```

**Chaining:**

```javascript
const numbers = [1, 2, 3, 4, 5];

// ✅ map is chainable
const result = numbers
  .map(n => n * 2)
  .filter(n => n > 5)
  .map(n => n + 1);
console.log(result); // [7, 9, 11]

// ❌ forEach is not chainable
// numbers
//   .forEach(n => n * 2)
//   .filter(n => n > 5); // TypeError: Cannot read property 'filter' of undefined

// forEach breaks the chain
numbers
  .map(n => n * 2)
  .forEach(n => console.log(n)); // Chain ends here (returns undefined)
  // .map(n => n + 1); // Can't continue
```

**Performance:**

```javascript
const large = Array.from({ length: 1000000 }, (_, i) => i);

// forEach: slightly faster (no array creation)
console.time('forEach');
let sum1 = 0;
large.forEach(n => {
  sum1 += n;
});
console.timeEnd('forEach'); // ~10ms

// map: creates new array (slower + memory overhead)
console.time('map');
const result = large.map(n => n * 2);
console.timeEnd('map'); // ~30ms

// For side effects only, forEach is more efficient
// For transformations, map is the right choice
```

**Use Cases:**

**Use forEach() when:**

```javascript
// 1. Logging/debugging
const users = [{ name: 'John' }, { name: 'Jane' }];
users.forEach((user, index) => {
  console.log(`${index}: ${user.name}`);
});

// 2. DOM manipulation
const buttons = document.querySelectorAll('button');
buttons.forEach(button => {
  button.addEventListener('click', handleClick);
});

// 3. Updating external state
let total = 0;
const prices = [10, 20, 30];
prices.forEach(price => {
  total += price;
});

// 4. Calling functions with side effects
const callbacks = [fn1, fn2, fn3];
callbacks.forEach(fn => fn());

// 5. Database operations
const ids = [1, 2, 3];
ids.forEach(async id => {
  await database.update(id, { status: 'processed' });
});

// 6. Mutating array elements (careful!)
const objects = [{ count: 0 }, { count: 0 }];
objects.forEach(obj => {
  obj.count++; // Mutating objects
});
```

**Use map() when:**

```javascript
// 1. Transforming data
const numbers = [1, 2, 3];
const doubled = numbers.map(n => n * 2);

// 2. Extracting properties
const users = [{ name: 'John', age: 25 }, { name: 'Jane', age: 30 }];
const names = users.map(u => u.name);

// 3. Converting types
const strings = ['1', '2', '3'];
const numbers = strings.map(Number);

// 4. React component rendering
const TodoList = ({ todos }) => (
  <ul>
    {todos.map(todo => (
      <TodoItem key={todo.id} todo={todo} />
    ))}
  </ul>
);

// 5. API response formatting
const rawData = [{ id: 1, value: 'a' }, { id: 2, value: 'b' }];
const formatted = rawData.map(item => ({
  identifier: item.id,
  content: item.value.toUpperCase()
}));

// 6. Creating derived data
const products = [{ price: 10 }, { price: 20 }];
const withTax = products.map(p => ({
  ...p,
  totalPrice: p.price * 1.1
}));
```

**Common Mistakes:**

**1. Using forEach when map is needed:**
```javascript
const numbers = [1, 2, 3];

// ❌ Wrong: trying to collect results with forEach
const doubled = [];
numbers.forEach(n => {
  doubled.push(n * 2);
});

// ✅ Right: use map
const doubled = numbers.map(n => n * 2);
```

**2. Using map when forEach is needed:**
```javascript
const users = [{ name: 'John' }, { name: 'Jane' }];

// ❌ Wrong: map creates unnecessary array
users.map(user => {
  console.log(user.name); // Just logging, don't need new array
});

// ✅ Right: use forEach for side effects
users.forEach(user => {
  console.log(user.name);
});
```

**3. Forgetting to return in map:**
```javascript
const numbers = [1, 2, 3];

// ❌ Wrong: no return statement
const doubled = numbers.map(n => {
  n * 2; // Missing return!
});
console.log(doubled); // [undefined, undefined, undefined]

// ✅ Right: explicit return
const doubled = numbers.map(n => {
  return n * 2;
});

// ✅ Or use implicit return
const doubled = numbers.map(n => n * 2);
```

**4. Using map result incorrectly:**
```javascript
const numbers = [1, 2, 3];

// ❌ Wrong: ignoring map's return value
numbers.map(n => n * 2); // Creates array but doesn't use it

// ✅ Right: use or store the result
const doubled = numbers.map(n => n * 2);
console.log(doubled);
```

**Breaking Out of Loop:**

```javascript
const numbers = [1, 2, 3, 4, 5];

// ❌ Can't break out of forEach
numbers.forEach(n => {
  if (n === 3) {
    // break; // SyntaxError: Illegal break statement
    return; // Only skips current iteration, doesn't stop loop
  }
  console.log(n); // Logs: 1, 2, 4, 5
});

// ❌ Can't break out of map either
numbers.map(n => {
  if (n === 3) {
    // break; // SyntaxError
  }
  return n * 2;
});

// ✅ Use for loop if you need to break
for (let i = 0; i < numbers.length; i++) {
  if (numbers[i] === 3) {
    break; // Stops loop
  }
  console.log(numbers[i]); // Logs: 1, 2
}

// ✅ Or use some/every to short-circuit
numbers.some(n => {
  console.log(n);
  return n === 3; // Stops when true
}); // Logs: 1, 2, 3
```

**Async Operations:**

```javascript
const ids = [1, 2, 3, 4, 5];

// ❌ forEach doesn't wait for promises
ids.forEach(async id => {
  const data = await fetchData(id);
  console.log(data);
});
// Continues immediately, doesn't wait for fetches

// ✅ Use Promise.all with map for parallel async
const promises = ids.map(id => fetchData(id));
const results = await Promise.all(promises);
console.log(results); // All data

// ✅ Or for...of for sequential async
for (const id of ids) {
  const data = await fetchData(id);
  console.log(data);
}
```

**Production Examples:**

**1. React Component:**
```javascript
// ✅ Use map for rendering
const UserList = ({ users }) => (
  <div>
    {users.map(user => (
      <UserCard key={user.id} user={user} />
    ))}
  </div>
);

// ❌ Don't use forEach for rendering
const BadUserList = ({ users }) => {
  const cards = [];
  users.forEach(user => {
    cards.push(<UserCard key={user.id} user={user} />);
  });
  return <div>{cards}</div>; // Awkward
};
```

**2. Data Processing:**
```javascript
// ✅ Use map for transformation
const processData = (rawData) => {
  return rawData.map(item => ({
    id: item.id,
    name: item.name.toUpperCase(),
    timestamp: Date.now()
  }));
};

// ✅ Use forEach for side effects
const logData = (data) => {
  data.forEach(item => {
    logger.info(`Processing item ${item.id}`);
  });
};
```

**3. Event Handling:**
```javascript
// ✅ Use forEach for adding listeners
const buttons = document.querySelectorAll('.btn');
buttons.forEach(button => {
  button.addEventListener('click', handleClick);
});

// ❌ Don't use map (creates unnecessary array)
const result = buttons.map(button => {
  button.addEventListener('click', handleClick);
}); // result is array of undefined values
```

**Best Practices:**
- Use **map()** when you need transformed data (new array)
- Use **forEach()** when you need side effects (logging, DOM, state updates)
- Don't ignore map's return value
- Don't use map when forEach is sufficient (performance + clarity)
- Always return a value in map callback
- Consider for...of loop if you need to break out
- Use map with Promise.all for async operations
- Chain map for multiple transformations

**Key Takeaways:**
- **forEach()**: Executes function for each element, returns `undefined`
- **map()**: Transforms elements, returns new array
- forEach for side effects, map for transformations
- map is chainable, forEach is not
- Both don't modify original array
- Can't break out of either (use for loop if needed)
- Choose based on whether you need the return value

42. What are some common array methods?

**Answer:**
JavaScript arrays have a rich set of built-in methods for manipulation, transformation, and querying. Understanding these methods is essential for effective array handling and functional programming.

**Adding/Removing Elements:**

```javascript
const arr = [1, 2, 3];

// push() - add to end (mutates, returns new length)
const newLength = arr.push(4, 5);
console.log(arr); // [1, 2, 3, 4, 5]
console.log(newLength); // 5

// pop() - remove from end (mutates, returns removed element)
const removed = arr.pop();
console.log(removed); // 5
console.log(arr); // [1, 2, 3, 4]

// unshift() - add to beginning (mutates, returns new length)
arr.unshift(0);
console.log(arr); // [0, 1, 2, 3, 4]

// shift() - remove from beginning (mutates, returns removed element)
const first = arr.shift();
console.log(first); // 0
console.log(arr); // [1, 2, 3, 4]

// splice() - add/remove at any position (mutates, returns removed elements)
arr.splice(2, 1, 'a', 'b'); // At index 2, remove 1, insert 'a', 'b'
console.log(arr); // [1, 2, 'a', 'b', 4]

const removed = arr.splice(1, 2); // Remove 2 elements starting at index 1
console.log(removed); // [2, 'a']
console.log(arr); // [1, 'b', 4]
```

**Searching and Finding:**

```javascript
const numbers = [1, 2, 3, 4, 5, 3];

// indexOf() - first index of element (-1 if not found)
console.log(numbers.indexOf(3)); // 2
console.log(numbers.indexOf(10)); // -1

// lastIndexOf() - last index of element
console.log(numbers.lastIndexOf(3)); // 5

// includes() - boolean check
console.log(numbers.includes(3)); // true
console.log(numbers.includes(10)); // false

// find() - first element matching condition
const found = numbers.find(n => n > 3);
console.log(found); // 4

// findIndex() - index of first match
const index = numbers.findIndex(n => n > 3);
console.log(index); // 3

// findLast() - last element matching condition (ES2023)
const lastFound = numbers.findLast(n => n === 3);
console.log(lastFound); // 3

// findLastIndex() - index of last match (ES2023)
const lastIndex = numbers.findLastIndex(n => n === 3);
console.log(lastIndex); // 5
```

**Testing Arrays:**

```javascript
const numbers = [1, 2, 3, 4, 5];

// some() - at least one element matches
console.log(numbers.some(n => n > 4)); // true
console.log(numbers.some(n => n > 10)); // false

// every() - all elements match
console.log(numbers.every(n => n > 0)); // true
console.log(numbers.every(n => n > 3)); // false

// Array.isArray() - check if value is array
console.log(Array.isArray(numbers)); // true
console.log(Array.isArray('string')); // false
console.log(Array.isArray({ length: 0 })); // false
```

**Transforming Arrays:**

```javascript
const numbers = [1, 2, 3, 4, 5];

// map() - transform each element
const doubled = numbers.map(n => n * 2);
console.log(doubled); // [2, 4, 6, 8, 10]

// filter() - keep elements matching condition
const evens = numbers.filter(n => n % 2 === 0);
console.log(evens); // [2, 4]

// reduce() - accumulate to single value
const sum = numbers.reduce((acc, n) => acc + n, 0);
console.log(sum); // 15

// reduceRight() - reduce from right to left
const arr = [1, 2, 3, 4];
const result = arr.reduceRight((acc, n) => acc + n, 0);
console.log(result); // 10 (same sum, but processed right to left)

// flat() - flatten nested arrays
const nested = [1, [2, 3], [4, [5, 6]]];
console.log(nested.flat()); // [1, 2, 3, 4, [5, 6]] (1 level)
console.log(nested.flat(2)); // [1, 2, 3, 4, 5, 6] (2 levels)
console.log(nested.flat(Infinity)); // [1, 2, 3, 4, 5, 6] (all levels)

// flatMap() - map then flatten one level
const words = ['hello world', 'foo bar'];
const letters = words.flatMap(word => word.split(' '));
console.log(letters); // ['hello', 'world', 'foo', 'bar']
```

**Extracting Portions:**

```javascript
const arr = [1, 2, 3, 4, 5];

// slice() - extract portion (doesn't mutate)
console.log(arr.slice(1, 4)); // [2, 3, 4]
console.log(arr.slice(2)); // [3, 4, 5] (from index to end)
console.log(arr.slice(-2)); // [4, 5] (last 2 elements)
console.log(arr.slice()); // [1, 2, 3, 4, 5] (shallow copy)

// at() - get element at index (supports negative)
console.log(arr.at(0)); // 1
console.log(arr.at(-1)); // 5 (last element)
console.log(arr.at(-2)); // 4 (second to last)
```

**Joining and Concatenating:**

```javascript
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];

// concat() - merge arrays (doesn't mutate)
const merged = arr1.concat(arr2);
console.log(merged); // [1, 2, 3, 4, 5, 6]

// Can concatenate multiple arrays
const arr3 = [7, 8];
const merged2 = arr1.concat(arr2, arr3);
console.log(merged2); // [1, 2, 3, 4, 5, 6, 7, 8]

// join() - array to string
const words = ['Hello', 'World'];
console.log(words.join(' ')); // 'Hello World'
console.log(words.join('-')); // 'Hello-World'
console.log(words.join('')); // 'HelloWorld'
console.log(words.join()); // 'Hello,World' (default separator is comma)
```

**Sorting and Reversing:**

```javascript
const numbers = [3, 1, 4, 1, 5, 9, 2, 6];

// sort() - sorts in place (⚠️ converts to strings by default)
numbers.sort();
console.log(numbers); // [1, 1, 2, 3, 4, 5, 6, 9]

// Numeric sort
const nums = [1, 10, 2, 20, 3, 30];
nums.sort(); // ⚠️ [1, 10, 2, 20, 3, 30] (lexicographic!)
nums.sort((a, b) => a - b); // ✅ [1, 2, 3, 10, 20, 30]

// Descending sort
nums.sort((a, b) => b - a);

// Sort objects
const people = [
  { name: 'John', age: 30 },
  { name: 'Jane', age: 25 },
  { name: 'Bob', age: 35 }
];
people.sort((a, b) => a.age - b.age);

// reverse() - reverses in place
const arr = [1, 2, 3, 4, 5];
arr.reverse();
console.log(arr); // [5, 4, 3, 2, 1]

// toSorted() - returns sorted copy (ES2023, doesn't mutate)
const original = [3, 1, 2];
const sorted = original.toSorted();
console.log(original); // [3, 1, 2] (unchanged)
console.log(sorted); // [1, 2, 3]

// toReversed() - returns reversed copy (ES2023)
const reversed = original.toReversed();
console.log(reversed); // [2, 1, 3]
```

**Iterating:**

```javascript
const arr = ['a', 'b', 'c'];

// forEach() - execute function for each element
arr.forEach((value, index) => {
  console.log(`${index}: ${value}`);
});

// entries() - iterator of [index, value] pairs
for (const [index, value] of arr.entries()) {
  console.log(index, value);
}

// keys() - iterator of indices
for (const index of arr.keys()) {
  console.log(index); // 0, 1, 2
}

// values() - iterator of values
for (const value of arr.values()) {
  console.log(value); // 'a', 'b', 'c'
}
```

**Filling and Copying:**

```javascript
// fill() - fill with static value (mutates)
const arr1 = [1, 2, 3, 4, 5];
arr1.fill(0);
console.log(arr1); // [0, 0, 0, 0, 0]

const arr2 = [1, 2, 3, 4, 5];
arr2.fill(0, 2, 4); // Fill from index 2 to 4 (exclusive)
console.log(arr2); // [1, 2, 0, 0, 5]

// copyWithin() - copy part of array to another location (mutates)
const arr3 = [1, 2, 3, 4, 5];
arr3.copyWithin(0, 3); // Copy from index 3 to index 0
console.log(arr3); // [4, 5, 3, 4, 5]

const arr4 = [1, 2, 3, 4, 5];
arr4.copyWithin(0, 3, 4); // Copy index 3-4 to index 0
console.log(arr4); // [4, 2, 3, 4, 5]

// with() - returns copy with element replaced (ES2023)
const arr5 = [1, 2, 3, 4, 5];
const modified = arr5.with(2, 99);
console.log(arr5); // [1, 2, 3, 4, 5] (unchanged)
console.log(modified); // [1, 2, 99, 4, 5]
```

**Static Methods:**

```javascript
// Array.isArray() - check if value is array
console.log(Array.isArray([1, 2, 3])); // true
console.log(Array.isArray('string')); // false

// Array.from() - create array from iterable/array-like
console.log(Array.from('hello')); // ['h', 'e', 'l', 'l', 'o']
console.log(Array.from({ length: 3 }, (_, i) => i)); // [0, 1, 2]
console.log(Array.from(new Set([1, 2, 2, 3]))); // [1, 2, 3]

// Array.of() - create array from arguments
console.log(Array.of(1, 2, 3)); // [1, 2, 3]
console.log(Array.of(5)); // [5] (not empty array with length 5)
console.log(new Array(5)); // [ <5 empty items> ]
```

**Converting to String:**

```javascript
const arr = [1, 2, 3];

// toString() - convert to comma-separated string
console.log(arr.toString()); // '1,2,3'

// toLocaleString() - localized string representation
const numbers = [1000, 2000, 3000];
console.log(numbers.toLocaleString('en-US')); // '1,000, 2,000, 3,000'

const dates = [new Date('2024-01-01'), new Date('2024-12-31')];
console.log(dates.toLocaleString('en-US'));
```

**Method Categories:**

**Mutating Methods (modify original):**
```javascript
const arr = [1, 2, 3];

arr.push(4);        // [1, 2, 3, 4]
arr.pop();          // [1, 2, 3]
arr.shift();        // [2, 3]
arr.unshift(1);     // [1, 2, 3]
arr.splice(1, 1);   // [1, 3]
arr.sort();         // [1, 3]
arr.reverse();      // [3, 1]
arr.fill(0);        // [0, 0]
arr.copyWithin(0, 1); // [0, 0]
```

**Non-Mutating Methods (return new array/value):**
```javascript
const arr = [1, 2, 3];

arr.slice(1);       // [2, 3]
arr.concat([4]);    // [1, 2, 3, 4]
arr.map(n => n * 2); // [2, 4, 6]
arr.filter(n => n > 1); // [2, 3]
arr.flat();         // [1, 2, 3]
arr.flatMap(n => [n, n]); // [1, 1, 2, 2, 3, 3]
arr.toSorted();     // [1, 2, 3]
arr.toReversed();   // [3, 2, 1]
arr.with(0, 99);    // [99, 2, 3]

console.log(arr);   // [1, 2, 3] (unchanged)
```

**Common Patterns:**

**1. Remove duplicates:**
```javascript
const arr = [1, 2, 2, 3, 3, 4];

// Using Set
const unique1 = [...new Set(arr)];

// Using filter
const unique2 = arr.filter((item, index) => arr.indexOf(item) === index);
```

**2. Flatten deeply nested array:**
```javascript
const nested = [1, [2, [3, [4]]]];
const flat = nested.flat(Infinity);
```

**3. Create range of numbers:**
```javascript
const range = Array.from({ length: 10 }, (_, i) => i + 1);
// [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```

**4. Group by property:**
```javascript
const people = [
  { name: 'John', age: 25 },
  { name: 'Jane', age: 30 },
  { name: 'Bob', age: 25 }
];

const grouped = people.reduce((acc, person) => {
  const age = person.age;
  if (!acc[age]) acc[age] = [];
  acc[age].push(person);
  return acc;
}, {});
```

**5. Chunk array:**
```javascript
function chunk(arr, size) {
  return Array.from(
    { length: Math.ceil(arr.length / size) },
    (_, i) => arr.slice(i * size, i * size + size)
  );
}

console.log(chunk([1, 2, 3, 4, 5, 6, 7], 3));
// [[1, 2, 3], [4, 5, 6], [7]]
```

**Best Practices:**
- Prefer non-mutating methods for immutability
- Use appropriate method for the task (don't use map for side effects)
- Always provide comparator function to sort() for numbers
- Use Array.isArray() to check for arrays, not typeof
- Chain methods for readable data transformations
- Consider performance with large arrays
- Use ES2023 methods (toSorted, with) for immutability

**Key Takeaways:**
- JavaScript arrays have 30+ built-in methods
- Methods fall into categories: mutating vs non-mutating
- Common operations: add/remove, search, transform, test, sort
- Understanding methods enables functional programming patterns
- Choose based on whether you want to mutate or create new array
- Method chaining creates readable data pipelines
- Essential for effective JavaScript development

43. How do you check if a variable is an array?

**Answer:**
The recommended way to check if a variable is an array is using `Array.isArray()`. While other methods exist, they have limitations and edge cases that make them unreliable.

**Best Method: Array.isArray():**

```javascript
const arr = [1, 2, 3];
const obj = { 0: 1, 1: 2, 2: 3, length: 3 };
const str = 'hello';

// ✅ Most reliable method
console.log(Array.isArray(arr));  // true
console.log(Array.isArray(obj));  // false
console.log(Array.isArray(str));  // false

// Works across different execution contexts (iframes)
// Works with all array types
console.log(Array.isArray([]));                // true
console.log(Array.isArray(new Array(5)));      // true
console.log(Array.isArray(Array.from('hi')));  // true

// Typed arrays are NOT regular arrays
console.log(Array.isArray(new Int8Array()));   // false
```

**Why Array.isArray() is Best:**

```javascript
// 1. Works across different execution contexts
const iframe = document.createElement('iframe');
document.body.appendChild(iframe);
const arrFromIframe = new iframe.contentWindow.Array(1, 2, 3);

console.log(Array.isArray(arrFromIframe));      // true ✅
console.log(arrFromIframe instanceof Array);    // false ❌

// 2. Handles edge cases correctly
console.log(Array.isArray(Array.prototype));    // true
console.log(Array.isArray({ __proto__: Array.prototype })); // false
```

**Alternative Methods (Not Recommended):**

**1. typeof (Doesn't Work):**
```javascript
const arr = [1, 2, 3];
console.log(typeof arr); // 'object' (not helpful!)
console.log(typeof {});  // 'object' (same!)
```

**2. instanceof Array (Has Issues):**
```javascript
const arr = [1, 2, 3];
console.log(arr instanceof Array); // true

// ❌ Fails across different contexts
// ❌ Can be fooled by prototype manipulation
const fakeArray = { __proto__: Array.prototype };
console.log(fakeArray instanceof Array); // true (but not an array!)
console.log(Array.isArray(fakeArray));   // false ✅
```

**Array-Like Objects (Not Arrays):**

```javascript
// Arguments object
function test() {
  console.log(Array.isArray(arguments)); // false
  const argsArray = Array.from(arguments);
  console.log(Array.isArray(argsArray)); // true
}

// NodeList
const divs = document.querySelectorAll('div');
console.log(Array.isArray(divs)); // false

// Convert to array
const divsArray = Array.from(divs);
console.log(Array.isArray(divsArray)); // true
```

**Practical Use Cases:**

```javascript
// 1. Function parameter validation
function processItems(items) {
  if (!Array.isArray(items)) {
    throw new TypeError('Expected an array');
  }
  return items.map(item => item * 2);
}

// 2. Normalize input
function toArray(input) {
  return Array.isArray(input) ? input : [input];
}

console.log(toArray([1, 2, 3])); // [1, 2, 3]
console.log(toArray(5));         // [5]

// 3. Safe array operations
function safeMap(array, fn) {
  if (!Array.isArray(array)) {
    console.warn('Expected array');
    return [];
  }
  return array.map(fn);
}
```

**Best Practices:**
- **Always use `Array.isArray()`** for checking arrays
- Don't rely on `typeof`, `instanceof`, or `constructor`
- Be aware of array-like objects (arguments, NodeList, etc.)
- Convert array-like objects with `Array.from()` or spread operator
- Validate array inputs in functions

**Key Takeaways:**
- `Array.isArray()` is the only reliable method
- `typeof` returns 'object' (not helpful)
- `instanceof` fails across execution contexts
- Array-like objects are not arrays
- Essential for safe array operations and validation

44. What is the difference between slice() and splice()?

**Answer:**
`slice()` and `splice()` both extract portions of arrays, but have fundamentally different behaviors: `slice()` creates a shallow copy without modifying the original, while `splice()` modifies the original array in place.

**Quick Comparison:**

| Feature | slice() | splice() |
|---------|---------|----------|
| **Modifies original** | No (immutable) | Yes (mutates) |
| **Returns** | New array | Array of removed elements |
| **Purpose** | Extract/copy | Add/remove/replace |
| **Parameters** | (start, end) | (start, deleteCount, ...items) |
| **Can add items** | No | Yes |

**slice() - Extract Without Modifying:**

**Syntax:** `array.slice(start, end)`

```javascript
const arr = [1, 2, 3, 4, 5];

// Extract from index 1 to 4 (end is exclusive)
const portion = arr.slice(1, 4);
console.log(portion); // [2, 3, 4]
console.log(arr);     // [1, 2, 3, 4, 5] (unchanged!)

// From index to end
console.log(arr.slice(2)); // [3, 4, 5]

// Negative indices (from end)
console.log(arr.slice(-2));     // [4, 5]
console.log(arr.slice(-3, -1)); // [3, 4]

// Copy entire array
const copy = arr.slice();
console.log(copy === arr); // false (different reference)
```

**splice() - Modify Original Array:**

**Syntax:** `array.splice(start, deleteCount, item1, item2, ...)`

```javascript
const arr = [1, 2, 3, 4, 5];

// Remove 2 elements starting at index 1
const removed = arr.splice(1, 2);
console.log(removed); // [2, 3]
console.log(arr);     // [1, 4, 5] (modified!)

// Remove and insert
const arr2 = [1, 2, 3, 4, 5];
arr2.splice(2, 1, 'a', 'b');
console.log(arr2); // [1, 2, 'a', 'b', 4, 5]

// Insert without removing
const arr3 = [1, 2, 3];
arr3.splice(1, 0, 'x');
console.log(arr3); // [1, 'x', 2, 3]
```

**Detailed Comparison:**

```javascript
const original = [1, 2, 3, 4, 5];

// slice - doesn't modify
const sliced = original.slice(1, 3);
console.log(original); // [1, 2, 3, 4, 5] (unchanged)
console.log(sliced);   // [2, 3]

// splice - modifies
const arr2 = [1, 2, 3, 4, 5];
const spliced = arr2.splice(1, 2);
console.log(arr2);     // [1, 4, 5] (modified!)
console.log(spliced);  // [2, 3]
```

**Common Use Cases:**

**slice() Use Cases:**
```javascript
// 1. Copy array
const copy = arr.slice();

// 2. Get last N elements
const lastThree = arr.slice(-3);

// 3. Pagination
function paginate(array, pageSize, pageNumber) {
  const start = (pageNumber - 1) * pageSize;
  return array.slice(start, start + pageSize);
}
```

**splice() Use Cases:**
```javascript
// 1. Remove elements
arr.splice(2, 1); // Remove 1 at index 2

// 2. Insert elements
arr.splice(2, 0, 'x', 'y'); // Insert at index 2

// 3. Replace elements
arr.splice(1, 2, 'a', 'b'); // Replace 2 elements
```

**Immutability:**

```javascript
// slice is safe for immutability
const state = [1, 2, 3, 4, 5];
const newState = state.slice(1, 4);
// state unchanged

// splice breaks immutability
state.splice(1, 2); // Mutates state!

// Immutable alternative
const immutableSplice = (arr, start, del, ...items) => [
  ...arr.slice(0, start),
  ...items,
  ...arr.slice(start + del)
];
```

**Best Practices:**
- Use **slice()** for immutability (React, Redux)
- Use **splice()** for in-place modifications
- Remember: **slice** = **s**afe, **splice** = **sp**ecial
- Always assign slice() result
- Consider toSpliced() (ES2023) for immutable splice

**Key Takeaways:**
- **slice()**: Extracts, returns new array, immutable
- **splice()**: Modifies in place, can add/remove/replace
- slice is safe, splice mutates
- Critical for frameworks relying on immutability

45. What is array destructuring?

**Answer:**
Array destructuring is a JavaScript expression that unpacks values from arrays into distinct variables using convenient shorthand syntax, introduced in ES6.

**Basic Syntax:**

```javascript
// Traditional way
const arr = [1, 2, 3];
const first = arr[0];
const second = arr[1];

// ✅ Destructuring (cleaner)
const [first, second, third] = [1, 2, 3];
console.log(first);  // 1
console.log(second); // 2
console.log(third);  // 3
```

**Basic Destructuring:**

```javascript
const colors = ['red', 'green', 'blue'];

// Extract values
const [primary, secondary] = colors;
console.log(primary);   // 'red'
console.log(secondary); // 'green'

// More variables than array → undefined
const [a, b, c, d] = ['x', 'y', 'z'];
console.log(d); // undefined

// Fewer variables → extras ignored
const [first] = [1, 2, 3, 4, 5];
console.log(first); // 1
```

**Skipping Elements:**

```javascript
const arr = [1, 2, 3, 4, 5];

// Skip with empty slots
const [first, , third] = arr;
console.log(first); // 1
console.log(third); // 3

// Get first and last
const [head, , , , tail] = arr;
console.log(head); // 1
console.log(tail); // 5
```

**Rest Pattern (...):**

```javascript
const numbers = [1, 2, 3, 4, 5];

// Collect remaining elements
const [first, ...rest] = numbers;
console.log(first); // 1
console.log(rest);  // [2, 3, 4, 5]

// Get first two and rest
const [a, b, ...remaining] = numbers;
console.log(remaining); // [3, 4, 5]

// Rest must be last
// const [...rest, last] = arr; // ❌ SyntaxError
```

**Default Values:**

```javascript
// Without defaults
const [a, b, c] = [1, 2];
console.log(c); // undefined

// With defaults
const [x, y, z = 3] = [1, 2];
console.log(z); // 3 (default used)

// Default only for undefined
const [m, n = 10] = [5, null];
console.log(n); // null (not undefined, default not used)
```

**Nested Destructuring:**

```javascript
const nested = [1, [2, 3], 4];
const [a, [b, c], d] = nested;
console.log(a); // 1
console.log(b); // 2
console.log(c); // 3
console.log(d); // 4

// Deeply nested
const deep = [1, [2, [3, 4]]];
const [x, [y, [z, w]]] = deep;
console.log(z); // 3
```

**Swapping Variables:**

```javascript
// Traditional swap
let a = 1, b = 2;
let temp = a;
a = b;
b = temp;

// ✅ Destructuring swap (elegant)
let x = 1, y = 2;
[x, y] = [y, x];
console.log(x, y); // 2, 1
```

**Function Return Values:**

```javascript
function getCoordinates() {
  return [10, 20];
}

const [x, y] = getCoordinates();
console.log(x); // 10
console.log(y); // 20

// Ignore some values
function getStats() {
  return [10, 20, 30, 40];
}

const [count, , total] = getStats();
console.log(count); // 10
console.log(total); // 30
```

**Function Parameters:**

```javascript
// Destructure parameters
function sum([a, b]) {
  return a + b;
}

console.log(sum([3, 5])); // 8

// With defaults
function process([x, y = 0]) {
  return x + y;
}

console.log(process([1]));    // 1
console.log(process([1, 2])); // 3
```

**Iterating with Destructuring:**

```javascript
const pairs = [[1, 'one'], [2, 'two']];

// Destructure in for...of
for (const [num, word] of pairs) {
  console.log(`${num}: ${word}`);
}

// Object.entries()
const obj = { a: 1, b: 2 };
for (const [key, value] of Object.entries(obj)) {
  console.log(`${key}: ${value}`);
}
```

**Practical Use Cases:**

**1. React Hooks:**
```javascript
const [count, setCount] = useState(0);
const [state, dispatch] = useReducer(reducer, initialState);
```

**2. Promise.all:**
```javascript
const [users, posts] = await Promise.all([
  fetchUsers(),
  fetchPosts()
]);
```

**3. CSV Parsing:**
```javascript
const csv = 'John,Doe,30';
const [firstName, lastName, age] = csv.split(',');
```

**4. Regular Expressions:**
```javascript
const dateStr = '2024-01-15';
const [, year, month, day] = dateStr.match(/(\d{4})-(\d{2})-(\d{2})/);
console.log(year); // '2024'
```

**Common Pitfalls:**

```javascript
// 1. Undefined values
const [a, b, c] = [1, 2]; // c is undefined
// Use defaults: const [a, b, c = 0] = [1, 2];

// 2. Rest must be last
// const [...rest, last] = [1, 2, 3]; // ❌ SyntaxError

// 3. Can't destructure null/undefined
// const [a] = null; // ❌ TypeError
const [a] = arr || []; // ✅ Safe
```

**Best Practices:**
- Use for cleaner code when extracting array values
- Provide defaults for potentially undefined values
- Use rest pattern for remaining elements
- Skip unwanted elements with empty slots
- Great for React hooks, Promise.all, multi-returns
- Don't over-nest for readability

**Key Takeaways:**
- Unpacks array values into variables
- Syntax: `const [a, b] = array`
- Can skip elements, use defaults, rest pattern
- Works with any iterable
- Enables elegant variable swapping
- Essential in modern JavaScript and React
- Part of ES6, widely supported

46. What is the spread operator?

**Answer:**
The spread operator (`...`) is an ES6 feature that expands (spreads) an iterable (like an array, string, or object) into individual elements. It's used for copying, merging, and passing elements as function arguments.

**Syntax:** `...iterable`

**Array Spreading:**

```javascript
const arr = [1, 2, 3];

// Spread array elements
console.log(...arr); // 1 2 3 (individual arguments)
console.log([...arr]); // [1, 2, 3] (new array)

// Copy array (shallow)
const original = [1, 2, 3];
const copy = [...original];
console.log(copy); // [1, 2, 3]
console.log(copy === original); // false (different reference)

// Concatenate arrays
const arr1 = [1, 2];
const arr2 = [3, 4];
const merged = [...arr1, ...arr2];
console.log(merged); // [1, 2, 3, 4]

// Add elements while spreading
const numbers = [2, 3, 4];
const extended = [1, ...numbers, 5];
console.log(extended); // [1, 2, 3, 4, 5]

// Spread in any position
const middle = [0, ...numbers, 5, 6];
console.log(middle); // [0, 2, 3, 4, 5, 6]
```

**Object Spreading (ES2018):**

```javascript
const obj = { a: 1, b: 2 };

// Copy object (shallow)
const copy = { ...obj };
console.log(copy); // { a: 1, b: 2 }
console.log(copy === obj); // false

// Merge objects
const obj1 = { a: 1, b: 2 };
const obj2 = { c: 3, d: 4 };
const merged = { ...obj1, ...obj2 };
console.log(merged); // { a: 1, b: 2, c: 3, d: 4 }

// Override properties (last wins)
const defaults = { theme: 'light', lang: 'en' };
const userSettings = { theme: 'dark' };
const settings = { ...defaults, ...userSettings };
console.log(settings); // { theme: 'dark', lang: 'en' }

// Add new properties
const user = { name: 'John' };
const extendedUser = { ...user, age: 30, email: 'john@example.com' };
console.log(extendedUser); // { name: 'John', age: 30, email: 'john@example.com' }
```

**String Spreading:**

```javascript
const str = 'hello';

// Spread into array
const chars = [...str];
console.log(chars); // ['h', 'e', 'l', 'l', 'o']

// Combine with other elements
const letters = ['a', ...str, 'z'];
console.log(letters); // ['a', 'h', 'e', 'l', 'l', 'o', 'z']

// Create array from string
const word = [...'JavaScript'];
console.log(word); // ['J', 'a', 'v', 'a', 'S', 'c', 'r', 'i', 'p', 't']
```

**Function Arguments:**

```javascript
function sum(a, b, c) {
  return a + b + c;
}

const numbers = [1, 2, 3];

// Without spread (old way)
sum.apply(null, numbers); // 6

// ✅ With spread (clean)
sum(...numbers); // 6

// Math functions
const nums = [5, 2, 8, 1, 9];
console.log(Math.max(...nums)); // 9
console.log(Math.min(...nums)); // 1

// Variable number of arguments
function greet(greeting, ...names) {
  return `${greeting} ${names.join(', ')}!`;
}

console.log(greet('Hello', 'John', 'Jane', 'Bob')); // 'Hello John, Jane, Bob!'
```

**Set and Map Spreading:**

```javascript
// Set to array
const set = new Set([1, 2, 3, 2, 1]);
const arr = [...set];
console.log(arr); // [1, 2, 3] (duplicates removed)

// Map to array
const map = new Map([['a', 1], ['b', 2]]);
const entries = [...map];
console.log(entries); // [['a', 1], ['b', 2]]

const keys = [...map.keys()];
console.log(keys); // ['a', 'b']

const values = [...map.values()];
console.log(values); // [1, 2]
```

**Copying vs Reference:**

```javascript
// Primitives - copied
const arr1 = [1, 2, 3];
const arr2 = [...arr1];
arr2[0] = 99;
console.log(arr1[0]); // 1 (unchanged)

// ⚠️ Shallow copy - nested objects are references
const nested = [1, [2, 3]];
const copy = [...nested];
copy[1][0] = 99;
console.log(nested[1][0]); // 99 (changed! shared reference)

// Deep copy needed for nested structures
const deepCopy = JSON.parse(JSON.stringify(nested));
// or use structuredClone()
const deepCopy2 = structuredClone(nested);
```

**Practical Use Cases:**

**1. Cloning Arrays/Objects:**
```javascript
// Array clone
const original = [1, 2, 3, 4, 5];
const clone = [...original];

// Object clone
const user = { name: 'John', age: 30 };
const userCopy = { ...user };

// Prevent mutation
function processArray(arr) {
  const safeCopy = [...arr];
  safeCopy.sort();
  return safeCopy;
}

const numbers = [3, 1, 2];
processArray(numbers);
console.log(numbers); // [3, 1, 2] (original unchanged)
```

**2. Merging Arrays:**
```javascript
const fruits = ['apple', 'banana'];
const vegetables = ['carrot', 'spinach'];
const food = [...fruits, ...vegetables];
console.log(food); // ['apple', 'banana', 'carrot', 'spinach']

// Multiple arrays
const arr1 = [1, 2];
const arr2 = [3, 4];
const arr3 = [5, 6];
const merged = [...arr1, ...arr2, ...arr3];
console.log(merged); // [1, 2, 3, 4, 5, 6]

// Alternative to concat
// Old: arr1.concat(arr2)
// New: [...arr1, ...arr2]
```

**3. Merging Objects:**
```javascript
const defaults = {
  theme: 'light',
  lang: 'en',
  notifications: true
};

const userPrefs = {
  theme: 'dark',
  fontSize: 14
};

const config = { ...defaults, ...userPrefs };
console.log(config);
// {
//   theme: 'dark',      // overridden
//   lang: 'en',         // from defaults
//   notifications: true, // from defaults
//   fontSize: 14        // from userPrefs
// }
```

**4. Adding/Updating Object Properties:**
```javascript
const user = { name: 'John', age: 30 };

// Add property
const withEmail = { ...user, email: 'john@example.com' };

// Update property
const olderUser = { ...user, age: 31 };

// Conditional property
const isAdmin = true;
const userWithRole = {
  ...user,
  ...(isAdmin && { role: 'admin' })
};
console.log(userWithRole); // { name: 'John', age: 30, role: 'admin' }
```

**5. React State Updates:**
```javascript
// Array state
const [items, setItems] = useState([1, 2, 3]);

// Add item immutably
setItems([...items, 4]);

// Remove item immutably
setItems(items.filter(item => item !== 2));

// Object state
const [user, setUser] = useState({ name: 'John', age: 30 });

// Update property immutably
setUser({ ...user, age: 31 });

// Nested state update
const [data, setData] = useState({
  user: { name: 'John', address: { city: 'NY' } }
});

setData({
  ...data,
  user: {
    ...data.user,
    address: {
      ...data.user.address,
      city: 'LA'
    }
  }
});
```

**6. Function Arguments:**
```javascript
// Pass array elements as arguments
const point = [10, 20];
const canvas = { moveTo: (x, y) => console.log(`Moving to ${x}, ${y}`) };
canvas.moveTo(...point); // Moving to 10, 20

// Creating Date with array
const dateComponents = [2024, 0, 15]; // Year, Month (0-indexed), Day
const date = new Date(...dateComponents);
console.log(date); // Mon Jan 15 2024
```

**7. Removing Duplicates:**
```javascript
const numbers = [1, 2, 2, 3, 3, 4, 5, 5];
const unique = [...new Set(numbers)];
console.log(unique); // [1, 2, 3, 4, 5]

// With objects (by property)
const users = [
  { id: 1, name: 'John' },
  { id: 2, name: 'Jane' },
  { id: 1, name: 'John' }
];

const uniqueUsers = [...new Map(users.map(u => [u.id, u])).values()];
console.log(uniqueUsers); // [{ id: 1, name: 'John' }, { id: 2, name: 'Jane' }]
```

**8. Converting Iterables to Arrays:**
```javascript
// NodeList to Array
const divs = document.querySelectorAll('div');
const divsArray = [...divs];
divsArray.forEach(div => div.classList.add('active'));

// Arguments to Array
function myFunction() {
  const args = [...arguments];
  return args.map(arg => arg * 2);
}

// String to Array
const chars = [...'hello'];
console.log(chars); // ['h', 'e', 'l', 'l', 'o']
```

**Spread vs Rest:**

```javascript
// Spread - expands array/object
const arr = [1, 2, 3];
console.log(...arr); // 1 2 3 (spread)
const copy = [...arr]; // [1, 2, 3]

// Rest - collects into array
function sum(...numbers) { // rest parameter
  return numbers.reduce((a, b) => a + b, 0);
}
console.log(sum(1, 2, 3, 4)); // 10

// Array destructuring with rest
const [first, ...rest] = [1, 2, 3, 4];
console.log(first); // 1
console.log(rest); // [2, 3, 4]

// Object destructuring with rest
const { name, ...others } = { name: 'John', age: 30, city: 'NY' };
console.log(name); // 'John'
console.log(others); // { age: 30, city: 'NY' }
```

**Comparison with Other Methods:**

```javascript
// Concatenating arrays
const arr1 = [1, 2];
const arr2 = [3, 4];

// concat()
const merged1 = arr1.concat(arr2);

// Spread (cleaner)
const merged2 = [...arr1, ...arr2];

// Copying arrays
const original = [1, 2, 3];

// slice()
const copy1 = original.slice();

// Spread (more intuitive)
const copy2 = [...original];

// Copying objects
const obj = { a: 1, b: 2 };

// Object.assign()
const copy1 = Object.assign({}, obj);

// Spread (cleaner)
const copy2 = { ...obj };
```

**Performance Considerations:**

```javascript
const large = Array.from({ length: 100000 }, (_, i) => i);

// Spread is fast but creates new array
console.time('spread');
const copy = [...large];
console.timeEnd('spread'); // ~2ms

// For very large arrays, consider alternatives
console.time('slice');
const copy2 = large.slice();
console.timeEnd('slice'); // ~1ms (slightly faster)

// For simple cases, spread is fine and more readable
```

**Common Mistakes:**

**1. Assuming deep copy:**
```javascript
const nested = { a: 1, b: { c: 2 } };
const copy = { ...nested };

copy.b.c = 99;
console.log(nested.b.c); // 99 (shared reference!)

// Need deep copy
const deepCopy = structuredClone(nested);
```

**2. Order matters in object spread:**
```javascript
const obj1 = { a: 1, b: 2 };
const obj2 = { b: 3, c: 4 };

const merged1 = { ...obj1, ...obj2 };
console.log(merged1); // { a: 1, b: 3, c: 4 } (obj2.b wins)

const merged2 = { ...obj2, ...obj1 };
console.log(merged2); // { b: 2, c: 4, a: 1 } (obj1.b wins)
```

**3. Spreading non-iterables:**
```javascript
const num = 42;
// [...num]; // ❌ TypeError: num is not iterable

const obj = { a: 1 };
// [...obj]; // ❌ TypeError: obj is not iterable

// Objects need object spread
const objCopy = { ...obj }; // ✅ Works
```

**Browser Support:**

```javascript
// Array/String spread - ES6 (2015), widely supported
const arr = [...[1, 2, 3]];

// Object spread - ES2018, modern browsers
const obj = { ...{ a: 1, b: 2 } };

// Polyfills available for older environments
```

**Best Practices:**
- Use spread for **immutable operations** (don't mutate originals)
- Prefer spread over `.concat()` and `Object.assign()` for readability
- Remember spread creates **shallow copies**
- Use `structuredClone()` or libraries for deep copying
- Leverage spread for **cleaner React state updates**
- Combine with destructuring for powerful patterns
- Be aware of performance with very large arrays
- Order matters when spreading objects

**Key Takeaways:**
- Spread operator (`...`) expands iterables into individual elements
- Works with arrays, objects, strings, Sets, Maps, etc.
- Creates shallow copies (nested objects are references)
- Cleaner than `concat()`, `Object.assign()`, `apply()`
- Essential for immutable operations in React/Redux
- Different from rest operator (same syntax, opposite purpose)
- Introduced in ES6 (arrays) and ES2018 (objects)
- Fundamental tool for modern JavaScript development

47. How do you remove duplicates from an array?

**Answer:**
There are multiple ways to remove duplicates from an array in JavaScript, with the Set approach being the most common and efficient for simple cases.

**1. Using Set (Most Common):**

```javascript
const numbers = [1, 2, 2, 3, 3, 4, 5, 5];

// ✅ Simplest and most efficient
const unique = [...new Set(numbers)];
console.log(unique); // [1, 2, 3, 4, 5]

// Alternative: Array.from()
const unique2 = Array.from(new Set(numbers));
console.log(unique2); // [1, 2, 3, 4, 5]

// Works with any type
const strings = ['a', 'b', 'a', 'c', 'b'];
const uniqueStrings = [...new Set(strings)];
console.log(uniqueStrings); // ['a', 'b', 'c']
```

**How Set Works:**

```javascript
// Set only keeps unique values
const set = new Set([1, 2, 2, 3, 3, 4]);
console.log(set); // Set(4) { 1, 2, 3, 4 }

// Convert back to array
const array = [...set];
console.log(array); // [1, 2, 3, 4]

// Size property
console.log(set.size); // 4

// Check for value
console.log(set.has(2)); // true
```

**2. Using filter() with indexOf():**

```javascript
const numbers = [1, 2, 2, 3, 3, 4, 5, 5];

const unique = numbers.filter((item, index) => {
  return numbers.indexOf(item) === index;
});

console.log(unique); // [1, 2, 3, 4, 5]

// One-liner
const unique2 = numbers.filter((item, i) => numbers.indexOf(item) === i);
```

**How indexOf() Method Works:**

```javascript
// indexOf returns first occurrence
const arr = [1, 2, 2, 3];

console.log(arr.indexOf(1)); // 0 (first occurrence at index 0)
console.log(arr.indexOf(2)); // 1 (first occurrence at index 1)

// For duplicates, indexOf always returns first position
// At index 1: arr.indexOf(2) === 1 → true (keep)
// At index 2: arr.indexOf(2) === 1 → false (remove)
```

**3. Using reduce():**

```javascript
const numbers = [1, 2, 2, 3, 3, 4, 5, 5];

const unique = numbers.reduce((acc, current) => {
  if (!acc.includes(current)) {
    acc.push(current);
  }
  return acc;
}, []);

console.log(unique); // [1, 2, 3, 4, 5]

// More concise
const unique2 = numbers.reduce((acc, curr) => 
  acc.includes(curr) ? acc : [...acc, curr], []
);
```

**4. Using forEach() with Object/Set:**

```javascript
const numbers = [1, 2, 2, 3, 3, 4, 5, 5];

// Using Set
const unique = [];
const seen = new Set();

numbers.forEach(num => {
  if (!seen.has(num)) {
    seen.add(num);
    unique.push(num);
  }
});

console.log(unique); // [1, 2, 3, 4, 5]

// Using Object
const unique2 = [];
const obj = {};

numbers.forEach(num => {
  if (!obj[num]) {
    obj[num] = true;
    unique2.push(num);
  }
});
```

**Removing Duplicates from Arrays of Objects:**

**By Property Value:**

```javascript
const users = [
  { id: 1, name: 'John' },
  { id: 2, name: 'Jane' },
  { id: 1, name: 'John' }, // duplicate
  { id: 3, name: 'Bob' }
];

// Using Map
const uniqueById = [...new Map(users.map(user => [user.id, user])).values()];
console.log(uniqueById);
// [{ id: 1, name: 'John' }, { id: 2, name: 'Jane' }, { id: 3, name: 'Bob' }]

// Using filter
const unique = users.filter((user, index, self) =>
  index === self.findIndex(u => u.id === user.id)
);

// Using reduce
const uniqueUsers = users.reduce((acc, user) => {
  const exists = acc.find(u => u.id === user.id);
  if (!exists) {
    acc.push(user);
  }
  return acc;
}, []);
```

**By Multiple Properties:**

```javascript
const data = [
  { id: 1, type: 'A' },
  { id: 2, type: 'B' },
  { id: 1, type: 'A' }, // duplicate
  { id: 1, type: 'B' }, // different type
  { id: 2, type: 'B' }  // duplicate
];

// Create unique key from multiple properties
const unique = [...new Map(
  data.map(item => [`${item.id}-${item.type}`, item])
).values()];

console.log(unique);
// [
//   { id: 1, type: 'A' },
//   { id: 2, type: 'B' },
//   { id: 1, type: 'B' }
// ]

// Alternative with JSON.stringify (works but slower)
const seen = new Set();
const unique2 = data.filter(item => {
  const key = JSON.stringify(item);
  if (seen.has(key)) {
    return false;
  }
  seen.add(key);
  return true;
});
```

**Case-Insensitive Deduplication:**

```javascript
const words = ['Apple', 'banana', 'APPLE', 'Banana', 'cherry'];

// Convert to lowercase for comparison
const uniqueLower = [...new Set(words.map(w => w.toLowerCase()))];
console.log(uniqueLower); // ['apple', 'banana', 'cherry']

// Keep original case (first occurrence)
const unique = words.filter((word, index, self) =>
  self.findIndex(w => w.toLowerCase() === word.toLowerCase()) === index
);
console.log(unique); // ['Apple', 'banana', 'cherry']

// Using Map to preserve first occurrence
const map = new Map();
words.forEach(word => {
  const key = word.toLowerCase();
  if (!map.has(key)) {
    map.set(key, word);
  }
});
const unique2 = [...map.values()];
console.log(unique2); // ['Apple', 'banana', 'cherry']
```

**Performance Comparison:**

```javascript
const large = Array.from({ length: 10000 }, () => 
  Math.floor(Math.random() * 1000)
);

// 1. Set - Fastest (O(n))
console.time('Set');
const unique1 = [...new Set(large)];
console.timeEnd('Set'); // ~1ms

// 2. filter + indexOf - Slow (O(n²))
console.time('filter + indexOf');
const unique2 = large.filter((item, i) => large.indexOf(item) === i);
console.timeEnd('filter + indexOf'); // ~50ms

// 3. reduce + includes - Slow (O(n²))
console.time('reduce');
const unique3 = large.reduce((acc, curr) => 
  acc.includes(curr) ? acc : [...acc, curr], []
);
console.timeEnd('reduce'); // ~40ms

// 4. forEach + Set - Fast (O(n))
console.time('forEach + Set');
const unique4 = [];
const seen = new Set();
large.forEach(num => {
  if (!seen.has(num)) {
    seen.add(num);
    unique4.push(num);
  }
});
console.timeEnd('forEach + Set'); // ~2ms

// Winner: Set-based methods are fastest
```

**Practical Use Cases:**

**1. Unique Tags/Categories:**
```javascript
const posts = [
  { title: 'Post 1', tags: ['js', 'react'] },
  { title: 'Post 2', tags: ['js', 'vue'] },
  { title: 'Post 3', tags: ['react', 'node'] }
];

// Get all unique tags
const allTags = posts.flatMap(post => post.tags);
const uniqueTags = [...new Set(allTags)];
console.log(uniqueTags); // ['js', 'react', 'vue', 'node']
```

**2. Merging Arrays Without Duplicates:**
```javascript
const arr1 = [1, 2, 3, 4];
const arr2 = [3, 4, 5, 6];

const merged = [...new Set([...arr1, ...arr2])];
console.log(merged); // [1, 2, 3, 4, 5, 6]
```

**3. User Input Validation:**
```javascript
function getUniqueEmails(emails) {
  // Normalize and deduplicate
  return [...new Set(emails.map(e => e.trim().toLowerCase()))];
}

const emails = ['john@example.com', 'JOHN@EXAMPLE.COM', 'jane@example.com'];
console.log(getUniqueEmails(emails)); // ['john@example.com', 'jane@example.com']
```

**4. Form Data Processing:**
```javascript
const formData = {
  interests: ['coding', 'music', 'coding', 'sports', 'music']
};

// Remove duplicates from interests
formData.interests = [...new Set(formData.interests)];
console.log(formData.interests); // ['coding', 'music', 'sports']
```

**Maintaining Order:**

```javascript
// Set maintains insertion order
const numbers = [3, 1, 2, 3, 2, 4, 1];
const unique = [...new Set(numbers)];
console.log(unique); // [3, 1, 2, 4] (order preserved)

// If you need sorted unique values
const sortedUnique = [...new Set(numbers)].sort((a, b) => a - b);
console.log(sortedUnique); // [1, 2, 3, 4]
```

**Edge Cases:**

```javascript
// Empty array
console.log([...new Set([])]); // []

// Array with NaN
const withNaN = [1, NaN, 2, NaN, 3];
console.log([...new Set(withNaN)]); // [1, NaN, 2, 3]
// Note: Set treats NaN === NaN (different from ===)

// Array with null/undefined
const mixed = [1, null, 2, undefined, null, 3];
console.log([...new Set(mixed)]); // [1, null, 2, undefined, 3]

// Objects (by reference)
const obj1 = { a: 1 };
const obj2 = { a: 1 };
const objs = [obj1, obj2, obj1];
console.log([...new Set(objs)]); // [obj1, obj2] (by reference)
console.log(objs[0] === objs[2]); // true
```

**Best Practices:**
- Use **Set** for simple primitive deduplication (fastest and cleanest)
- Use **Map** for object deduplication by property
- Avoid `filter + indexOf` for large arrays (O(n²) complexity)
- Consider case sensitivity for strings
- For objects, decide deduplication criteria (by reference or by value)
- Use `structuredClone()` or deep copy if objects should be truly unique
- Document whether first or last occurrence is kept
- Test with edge cases (NaN, null, undefined)

**Key Takeaways:**
- **Set** is the best method for primitive arrays (fast, clean)
- `[...new Set(array)]` is the standard pattern
- `filter + indexOf` works but is slow for large arrays
- Objects need special handling (deduplicate by property)
- Set maintains insertion order
- Performance varies significantly (Set is O(n), filter+indexOf is O(n²))
- Essential for data cleaning and normalization
- Consider case sensitivity and comparison criteria

48. What is Array.from()?

**Answer:**
`Array.from()` is a static method that creates a new array from an array-like or iterable object. It's particularly useful for converting non-array objects (like NodeList, arguments, strings) into proper arrays.

**Syntax:** `Array.from(arrayLike, mapFn, thisArg)`

**Basic Usage:**

```javascript
// From string
const str = 'hello';
const chars = Array.from(str);
console.log(chars); // ['h', 'e', 'l', 'l', 'o']

// From Set
const set = new Set([1, 2, 3]);
const arr = Array.from(set);
console.log(arr); // [1, 2, 3]

// From Map
const map = new Map([['a', 1], ['b', 2]]);
const entries = Array.from(map);
console.log(entries); // [['a', 1], ['b', 2]]

// From array-like object
const arrayLike = { 0: 'a', 1: 'b', 2: 'c', length: 3 };
const array = Array.from(arrayLike);
console.log(array); // ['a', 'b', 'c']
```

**With Mapping Function (Second Parameter):**

```javascript
// Transform while converting
const numbers = [1, 2, 3];
const doubled = Array.from(numbers, x => x * 2);
console.log(doubled); // [2, 4, 6]

// From string with transformation
const str = 'hello';
const upperChars = Array.from(str, char => char.toUpperCase());
console.log(upperChars); // ['H', 'E', 'L', 'L', 'O']

// With index
const indexed = Array.from([10, 20, 30], (value, index) => ({
  index,
  value,
  squared: value ** 2
}));
console.log(indexed);
// [
//   { index: 0, value: 10, squared: 100 },
//   { index: 1, value: 20, squared: 400 },
//   { index: 2, value: 30, squared: 900 }
// ]
```

**Creating Ranges:**

```javascript
// Create array of numbers 0-9
const range = Array.from({ length: 10 }, (_, i) => i);
console.log(range); // [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

// Create array 1-10
const oneToTen = Array.from({ length: 10 }, (_, i) => i + 1);
console.log(oneToTen); // [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

// Create range with step
const evens = Array.from({ length: 5 }, (_, i) => i * 2);
console.log(evens); // [0, 2, 4, 6, 8]

// Descending range
const countdown = Array.from({ length: 5 }, (_, i) => 5 - i);
console.log(countdown); // [5, 4, 3, 2, 1]

// Character range
const alphabet = Array.from({ length: 26 }, (_, i) => 
  String.fromCharCode(97 + i)
);
console.log(alphabet); // ['a', 'b', 'c', ..., 'z']
```

**Converting Array-Like Objects:**

```javascript
// NodeList to Array
const divs = document.querySelectorAll('div');
const divsArray = Array.from(divs);
divsArray.forEach(div => div.classList.add('active'));

// HTMLCollection to Array
const links = document.getElementsByTagName('a');
const linksArray = Array.from(links);
const hrefs = linksArray.map(link => link.href);

// Arguments object to Array
function sum() {
  const args = Array.from(arguments);
  return args.reduce((a, b) => a + b, 0);
}
console.log(sum(1, 2, 3, 4)); // 10

// Modern alternative for arguments
function sum2(...args) {
  return args.reduce((a, b) => a + b, 0);
}
```

**Working with Strings:**

```javascript
// String to character array
const word = 'JavaScript';
const letters = Array.from(word);
console.log(letters); // ['J', 'a', 'v', 'a', 'S', 'c', 'r', 'i', 'p', 't']

// Handle Unicode properly (unlike split)
const emoji = '😀😃😄';
console.log(emoji.split('')); // Wrong: splits surrogate pairs
console.log(Array.from(emoji)); // Correct: ['😀', '😃', '😄']

// Count emoji correctly
const text = 'Hello 😀 World 😃';
console.log(text.length); // 17 (wrong, counts surrogate pairs)
console.log(Array.from(text).length); // 15 (correct)
```

**Creating Initialized Arrays:**

```javascript
// Array of same values
const zeros = Array.from({ length: 5 }, () => 0);
console.log(zeros); // [0, 0, 0, 0, 0]

// Array of objects (each unique)
const users = Array.from({ length: 3 }, (_, i) => ({
  id: i + 1,
  name: `User ${i + 1}`
}));
console.log(users);
// [
//   { id: 1, name: 'User 1' },
//   { id: 2, name: 'User 2' },
//   { id: 3, name: 'User 3' }
// ]

// Array of arrays (matrix)
const matrix = Array.from({ length: 3 }, () => 
  Array.from({ length: 3 }, () => 0)
);
console.log(matrix);
// [[0, 0, 0], [0, 0, 0], [0, 0, 0]]
```

**Practical Use Cases:**

**1. Paginate Array:**
```javascript
function paginate(array, pageSize) {
  return Array.from(
    { length: Math.ceil(array.length / pageSize) },
    (_, i) => array.slice(i * pageSize, (i + 1) * pageSize)
  );
}

const data = [1, 2, 3, 4, 5, 6, 7, 8, 9];
const pages = paginate(data, 3);
console.log(pages); // [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
```

**2. Generate Test Data:**
```javascript
const mockUsers = Array.from({ length: 10 }, (_, i) => ({
  id: i + 1,
  name: `User ${i + 1}`,
  email: `user${i + 1}@example.com`,
  active: Math.random() > 0.5
}));
```

**3. Convert Map Keys/Values:**
```javascript
const map = new Map([
  ['name', 'John'],
  ['age', 30],
  ['city', 'New York']
]);

const keys = Array.from(map.keys());
console.log(keys); // ['name', 'age', 'city']

const values = Array.from(map.values());
console.log(values); // ['John', 30, 'New York']

const entries = Array.from(map.entries());
console.log(entries); // [['name', 'John'], ['age', 30], ['city', 'New York']]
```

**4. Deduplicate While Transforming:**
```javascript
const numbers = [1, 2, 2, 3, 3, 4];
const uniqueDoubled = Array.from(
  new Set(numbers),
  num => num * 2
);
console.log(uniqueDoubled); // [2, 4, 6, 8]
```

**5. Chunk Array:**
```javascript
function chunk(array, size) {
  return Array.from(
    { length: Math.ceil(array.length / size) },
    (_, i) => array.slice(i * size, i * size + size)
  );
}

const items = [1, 2, 3, 4, 5, 6, 7];
console.log(chunk(items, 3)); // [[1, 2, 3], [4, 5, 6], [7]]
```

**Array.from() vs Spread Operator:**

```javascript
const set = new Set([1, 2, 3]);

// Both work for iterables
const arr1 = Array.from(set);
const arr2 = [...set];
console.log(arr1); // [1, 2, 3]
console.log(arr2); // [1, 2, 3]

// Array.from() works with array-like (spread doesn't)
const arrayLike = { 0: 'a', 1: 'b', length: 2 };
const arr3 = Array.from(arrayLike);
console.log(arr3); // ['a', 'b']
// const arr4 = [...arrayLike]; // ❌ TypeError: not iterable

// Array.from() allows mapping
const doubled = Array.from([1, 2, 3], x => x * 2);
const doubled2 = [...[1, 2, 3]].map(x => x * 2); // Spread needs separate map
```

**With thisArg (Third Parameter):**

```javascript
// Rarely used - sets 'this' context in mapping function
const multiplier = {
  factor: 2,
  multiply(x) {
    return x * this.factor;
  }
};

const numbers = [1, 2, 3];
const result = Array.from(numbers, multiplier.multiply, multiplier);
console.log(result); // [2, 4, 6]

// Modern alternative: arrow function (captures this automatically)
const result2 = Array.from(numbers, x => multiplier.multiply(x));
```

**Edge Cases:**

```javascript
// Empty array-like
console.log(Array.from({ length: 0 })); // []

// Missing length property
console.log(Array.from({ 0: 'a', 1: 'b' })); // [] (no length)

// Non-numeric length
console.log(Array.from({ length: '5' })); // [undefined × 5] (converts to number)

// Negative length
console.log(Array.from({ length: -5 })); // [] (treated as 0)

// null/undefined
console.log(Array.from(null)); // TypeError
console.log(Array.from(undefined)); // TypeError
console.log(Array.from(null) || []); // [] (safe default)
```

**Performance Considerations:**

```javascript
// Creating large arrays
const size = 1000000;

// Array.from with mapping
console.time('Array.from');
const arr1 = Array.from({ length: size }, (_, i) => i);
console.timeEnd('Array.from'); // ~50ms

// Traditional loop
console.time('for loop');
const arr2 = [];
for (let i = 0; i < size; i++) {
  arr2.push(i);
}
console.timeEnd('for loop'); // ~30ms

// For small arrays, Array.from is fine and more readable
// For very large arrays, traditional loops may be faster
```

**Best Practices:**
- Use **Array.from()** to convert array-like and iterable objects
- Leverage second parameter for transformation (cleaner than separate map)
- Prefer spread operator for simple iterable conversion
- Use for creating ranges and initialized arrays
- Handle Unicode strings properly with Array.from()
- Validate input (check for null/undefined)
- Consider performance for very large arrays
- Great for functional programming patterns

**Key Takeaways:**
- `Array.from()` creates arrays from array-like or iterable objects
- Accepts optional mapping function as second parameter
- Works with NodeList, arguments, strings, Sets, Maps, etc.
- Useful for creating ranges: `Array.from({ length: n }, (_, i) => i)`
- Handles Unicode correctly (unlike split)
- More versatile than spread operator (works with array-like objects)
- Essential for DOM manipulation and functional programming
- Part of ES6, widely supported
