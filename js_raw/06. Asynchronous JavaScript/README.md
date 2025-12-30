## Asynchronous JavaScript
49. What is asynchronous programming?

**Answer:**
Asynchronous programming is a programming paradigm that allows code to execute without blocking the main thread, enabling operations (like network requests, file I/O, or timers) to run in the background while the program continues executing other code. This is crucial for maintaining responsive user interfaces and efficient resource utilization.

**Synchronous vs Asynchronous:**

```javascript
// ❌ Synchronous - blocks execution
console.log('Start');
// Imagine a 3-second blocking operation
for (let i = 0; i < 3000000000; i++) {} // Blocks for ~3 seconds
console.log('End');
// Output: Start -> (3 second freeze) -> End

// ✅ Asynchronous - doesn't block
console.log('Start');
setTimeout(() => {
  console.log('After 3 seconds');
}, 3000);
console.log('End');
// Output: Start -> End -> (3 seconds later) -> After 3 seconds
```

**Real-World Example:**

```javascript
// Synchronous (blocking) - BAD for web
function fetchDataSync() {
  // Blocks for 2 seconds
  const data = makeNetworkRequestSync(); // Freezes browser!
  console.log(data);
  console.log('Next operation');
}

// Asynchronous (non-blocking) - GOOD
function fetchDataAsync() {
  makeNetworkRequestAsync((data) => {
    console.log(data);
  });
  console.log('Next operation'); // Runs immediately
}

fetchDataAsync();
// Output: 'Next operation' -> (later) -> data
```

**Why Asynchronous Programming?**

```javascript
// Without async - UI freezes
function processLargeData() {
  showLoadingSpinner();
  const result = processMillionsOfRecords(); // 5 seconds - UI FROZEN!
  hideLoadingSpinner();
  displayResults(result);
}

// With async - UI stays responsive
async function processLargeDataAsync() {
  showLoadingSpinner();
  // Processing happens in chunks, UI can update
  const result = await processInChunks();
  hideLoadingSpinner();
  displayResults(result);
}
```

**Asynchronous Patterns in JavaScript:**

**1. Callbacks (Original Pattern):**

```javascript
function fetchUser(id, callback) {
  setTimeout(() => {
    const user = { id, name: 'John' };
    callback(user);
  }, 1000);
}

fetchUser(1, (user) => {
  console.log('User:', user);
});
console.log('Fetching user...'); // Runs first

// Output:
// Fetching user...
// User: { id: 1, name: 'John' }
```

**2. Promises (ES6):**

```javascript
function fetchUser(id) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      const user = { id, name: 'John' };
      resolve(user);
    }, 1000);
  });
}

fetchUser(1)
  .then(user => console.log('User:', user))
  .catch(error => console.error('Error:', error));

console.log('Fetching user...'); // Runs first
```

**3. Async/Await (ES2017):**

```javascript
async function getUser(id) {
  try {
    const user = await fetchUser(id);
    console.log('User:', user);
  } catch (error) {
    console.error('Error:', error);
  }
}

getUser(1);
console.log('Fetching user...'); // Runs first
```

**Common Asynchronous Operations:**

**Network Requests:**

```javascript
// Fetch API (returns Promise)
async function loadData() {
  try {
    const response = await fetch('https://api.example.com/data');
    const data = await response.json();
    console.log('Data:', data);
  } catch (error) {
    console.error('Failed to load:', error);
  }
}

// XMLHttpRequest (callback-based)
function loadDataXHR(callback) {
  const xhr = new XMLHttpRequest();
  xhr.open('GET', 'https://api.example.com/data');
  xhr.onload = () => callback(xhr.responseText);
  xhr.send();
}
```

**Timers:**

```javascript
// setTimeout - runs once after delay
setTimeout(() => {
  console.log('Executed after 2 seconds');
}, 2000);

// setInterval - runs repeatedly
const intervalId = setInterval(() => {
  console.log('Runs every second');
}, 1000);

// Clear interval after 5 seconds
setTimeout(() => {
  clearInterval(intervalId);
  console.log('Interval stopped');
}, 5000);
```

**File Operations (Node.js):**

```javascript
const fs = require('fs').promises;

// Asynchronous file read
async function readConfig() {
  try {
    const data = await fs.readFile('config.json', 'utf8');
    const config = JSON.parse(data);
    return config;
  } catch (error) {
    console.error('Failed to read config:', error);
  }
}

// Multiple file operations
async function processFiles() {
  const [file1, file2, file3] = await Promise.all([
    fs.readFile('file1.txt', 'utf8'),
    fs.readFile('file2.txt', 'utf8'),
    fs.readFile('file3.txt', 'utf8')
  ]);
  
  return file1 + file2 + file3;
}
```

**DOM Events:**

```javascript
// Event listeners are asynchronous
document.getElementById('button').addEventListener('click', () => {
  console.log('Button clicked');
  // Event handler runs when event occurs
});

console.log('Event listener registered'); // Runs immediately

// async event handler
document.getElementById('submit').addEventListener('click', async (e) => {
  e.preventDefault();
  const data = await submitForm();
  displaySuccess(data);
});
```

**Practical Examples:**

**1. Sequential Async Operations:**

```javascript
// Load user, then their posts, then comments
async function loadUserData(userId) {
  try {
    console.log('Loading user...');
    const user = await fetchUser(userId);
    
    console.log('Loading posts...');
    const posts = await fetchPosts(user.id);
    
    console.log('Loading comments...');
    const comments = await fetchComments(posts[0].id);
    
    return { user, posts, comments };
  } catch (error) {
    console.error('Error loading data:', error);
  }
}

// Each operation waits for previous to complete
```

**2. Parallel Async Operations:**

```javascript
// Load multiple resources simultaneously
async function loadDashboardData() {
  try {
    console.log('Loading dashboard...');
    
    // All requests start simultaneously
    const [user, posts, notifications, stats] = await Promise.all([
      fetchUser(1),
      fetchPosts(),
      fetchNotifications(),
      fetchStats()
    ]);
    
    return { user, posts, notifications, stats };
  } catch (error) {
    console.error('Error loading dashboard:', error);
  }
}

// Much faster than sequential loading
```

**3. Race Condition Handling:**

```javascript
// Use first response, timeout the rest
async function fetchWithTimeout(url, timeout = 5000) {
  const controller = new AbortController();
  
  const timeoutPromise = new Promise((_, reject) => {
    setTimeout(() => {
      controller.abort();
      reject(new Error('Request timeout'));
    }, timeout);
  });
  
  const fetchPromise = fetch(url, { signal: controller.signal });
  
  return Promise.race([fetchPromise, timeoutPromise]);
}

// Usage
try {
  const response = await fetchWithTimeout('https://api.example.com/data', 3000);
  const data = await response.json();
} catch (error) {
  console.error('Request failed or timed out:', error);
}
```

**4. Retry Logic:**

```javascript
async function fetchWithRetry(url, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      console.log(`Attempt ${i + 1}/${maxRetries}`);
      const response = await fetch(url);
      
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}`);
      }
      
      return await response.json();
    } catch (error) {
      console.error(`Attempt ${i + 1} failed:`, error.message);
      
      if (i === maxRetries - 1) {
        throw new Error('Max retries reached');
      }
      
      // Exponential backoff
      await new Promise(resolve => 
        setTimeout(resolve, Math.pow(2, i) * 1000)
      );
    }
  }
}

// Usage
try {
  const data = await fetchWithRetry('https://api.example.com/data');
  console.log('Data:', data);
} catch (error) {
  console.error('Failed after retries:', error);
}
```

**5. Debouncing Async Operations:**

```javascript
function debounce(fn, delay) {
  let timeoutId;
  
  return function(...args) {
    clearTimeout(timeoutId);
    
    return new Promise((resolve, reject) => {
      timeoutId = setTimeout(async () => {
        try {
          const result = await fn(...args);
          resolve(result);
        } catch (error) {
          reject(error);
        }
      }, delay);
    });
  };
}

// Debounced search
const debouncedSearch = debounce(async (query) => {
  const response = await fetch(`/api/search?q=${query}`);
  return await response.json();
}, 300);

// Usage in input handler
searchInput.addEventListener('input', async (e) => {
  try {
    const results = await debouncedSearch(e.target.value);
    displayResults(results);
  } catch (error) {
    console.error('Search failed:', error);
  }
});
```

**Benefits of Asynchronous Programming:**

| Benefit | Description | Example |
|---------|-------------|---------|
| **Non-blocking** | Code continues executing | UI remains responsive during network calls |
| **Better Performance** | Multiple operations in parallel | Load user data and posts simultaneously |
| **Resource Efficiency** | CPU used while waiting | Handle multiple requests without extra threads |
| **Improved UX** | No frozen interfaces | Show loading spinners, update progressively |
| **Scalability** | Handle more concurrent operations | Server can handle thousands of requests |

**Common Pitfalls:**

**1. Not Handling Errors:**

```javascript
// ❌ No error handling
async function loadData() {
  const data = await fetch('/api/data'); // If this fails, unhandled rejection
  return data;
}

// ✅ Proper error handling
async function loadData() {
  try {
    const response = await fetch('/api/data');
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }
    return await response.json();
  } catch (error) {
    console.error('Failed to load data:', error);
    return null; // or throw, or return default value
  }
}
```

**2. Forgetting await:**

```javascript
// ❌ Forgot await - returns Promise, not data
async function getUser() {
  const user = fetchUser(1); // Missing await!
  console.log(user); // Promise { <pending> }
  return user.name; // undefined or error
}

// ✅ Correct
async function getUser() {
  const user = await fetchUser(1);
  console.log(user); // { id: 1, name: 'John' }
  return user.name; // 'John'
}
```

**3. Sequential When Could Be Parallel:**

```javascript
// ❌ Slow - runs sequentially (5 seconds total)
async function loadData() {
  const user = await fetchUser(1);      // 2 seconds
  const posts = await fetchPosts();     // 2 seconds
  const comments = await fetchComments(); // 1 second
  return { user, posts, comments };
}

// ✅ Fast - runs in parallel (2 seconds total)
async function loadData() {
  const [user, posts, comments] = await Promise.all([
    fetchUser(1),      // 2 seconds
    fetchPosts(),      // 2 seconds
    fetchComments()    // 1 second
  ]);
  return { user, posts, comments };
}
```

**4. Creating Promises in Loops:**

```javascript
// ❌ Creates all promises at once (may overwhelm server)
async function processUsers(userIds) {
  const promises = userIds.map(id => fetchUser(id));
  const users = await Promise.all(promises); // 100 concurrent requests!
  return users;
}

// ✅ Process in batches
async function processUsers(userIds, batchSize = 5) {
  const results = [];
  
  for (let i = 0; i < userIds.length; i += batchSize) {
    const batch = userIds.slice(i, i + batchSize);
    const batchResults = await Promise.all(
      batch.map(id => fetchUser(id))
    );
    results.push(...batchResults);
  }
  
  return results;
}
```

**Best Practices:**
- Always handle errors (try/catch or .catch())
- Use async/await for cleaner code
- Run independent operations in parallel (Promise.all)
- Implement timeouts for network requests
- Add retry logic for critical operations
- Debounce/throttle frequent async calls
- Avoid blocking the main thread
- Test error scenarios thoroughly
- Use AbortController for cancellable requests
- Consider loading states and user feedback

**Key Takeaways:**
- Asynchronous programming allows non-blocking code execution
- Essential for network requests, timers, file I/O, events
- JavaScript uses callbacks, Promises, and async/await
- Enables responsive UIs and better performance
- Run independent operations in parallel for speed
- Always handle errors properly
- Main thread remains free for user interactions
- Critical for modern web development and Node.js

50. What is the event loop?

**Answer:**
The event loop is JavaScript's mechanism for handling asynchronous operations in a single-threaded environment. It continuously checks the call stack and task queues, executing code in a specific order, which allows JavaScript to perform non-blocking operations despite being single-threaded.

**JavaScript is Single-Threaded:**

```javascript
// JavaScript runs one piece of code at a time
console.log('First');
console.log('Second');
console.log('Third');

// Output (predictable, sequential):
// First
// Second
// Third
```

**How the Event Loop Works:**

**Core Components:**

1. **Call Stack** - Tracks function execution
2. **Web APIs** - Browser/Node.js APIs (setTimeout, fetch, etc.)
3. **Callback Queue (Task Queue)** - Holds callbacks from async operations
4. **Microtask Queue** - Holds Promise callbacks (higher priority)
5. **Event Loop** - Monitors and coordinates everything

```
┌───────────────────────────┐
│      Call Stack           │ ← JavaScript execution
└───────────────────────────┘
            ↑
            │
┌───────────────────────────┐
│      Event Loop           │ ← Checks if stack is empty
└───────────────────────────┘
            ↑
     ┌──────┴──────┐
     │             │
┌─────────┐  ┌─────────────┐
│Microtask│  │ Callback    │
│ Queue   │  │ Queue       │
│(Promise)│  │(setTimeout) │
└─────────┘  └─────────────┘
     ↑             ↑
     │             │
┌────────────────────────────┐
│     Web APIs / Node APIs    │
│  (setTimeout, fetch, etc.)  │
└────────────────────────────┘
```

**Event Loop Process:**

```javascript
// Step-by-step execution
console.log('1: Start'); // → Call stack

setTimeout(() => {
  console.log('2: Timeout'); // → Web API → Callback Queue
}, 0);

Promise.resolve().then(() => {
  console.log('3: Promise'); // → Microtask Queue
});

console.log('4: End'); // → Call stack

// Output order:
// 1: Start
// 4: End
// 3: Promise  (microtask - higher priority)
// 2: Timeout  (macrotask)
```

**Detailed Example with Explanation:**

```javascript
function main() {
  console.log('A'); // 1. Goes to call stack, executes
  
  setTimeout(() => {
    console.log('B'); // 4. Moved to callback queue after 0ms
  }, 0);
  
  Promise.resolve().then(() => {
    console.log('C'); // 3. Moved to microtask queue
  });
  
  console.log('D'); // 2. Goes to call stack, executes
}

main();

// Execution flow:
// 1. main() called → added to call stack
// 2. console.log('A') → executes → 'A' printed
// 3. setTimeout() → registered with Web API → removed from stack
// 4. Promise.resolve().then() → callback added to microtask queue
// 5. console.log('D') → executes → 'D' printed
// 6. main() finishes → removed from call stack
// 7. Call stack empty → event loop checks microtask queue
// 8. Promise callback executes → 'C' printed
// 9. Microtask queue empty → event loop checks callback queue
// 10. setTimeout callback executes → 'B' printed

// Output: A → D → C → B
```

**Call Stack Visualization:**

```javascript
function first() {
  console.log('First function');
  second();
  console.log('First function end');
}

function second() {
  console.log('Second function');
  third();
  console.log('Second function end');
}

function third() {
  console.log('Third function');
}

first();

// Call stack changes:
// Step 1: [first]
// Step 2: [first, second]
// Step 3: [first, second, third]
// Step 4: [first, second]        (third done)
// Step 5: [first]                (second done)
// Step 6: []                     (first done)

// Output:
// First function
// Second function
// Third function
// Second function end
// First function end
```

**Blocking the Event Loop:**

```javascript
// ❌ BAD - Blocks event loop
console.log('Start');

// Synchronous loop blocks for ~3 seconds
for (let i = 0; i < 3000000000; i++) {
  // Heavy computation - nothing else can run!
}

console.log('End');
// UI frozen for 3 seconds!

// ✅ GOOD - Non-blocking
console.log('Start');

setTimeout(() => {
  console.log('Async work done');
}, 0);

console.log('End');
// UI remains responsive
// Output: Start → End → Async work done
```

**Microtasks vs Macrotasks:**

```javascript
// Demonstration of queue priorities
console.log('Script start');

// Macrotask (lower priority)
setTimeout(() => {
  console.log('setTimeout 1');
}, 0);

// Microtask (higher priority)
Promise.resolve()
  .then(() => {
    console.log('Promise 1');
    return Promise.resolve();
  })
  .then(() => {
    console.log('Promise 2');
  });

// Another macrotask
setTimeout(() => {
  console.log('setTimeout 2');
}, 0);

// Microtask
Promise.resolve().then(() => {
  console.log('Promise 3');
});

console.log('Script end');

// Output:
// Script start
// Script end
// Promise 1
// Promise 3
// Promise 2
// setTimeout 1
// setTimeout 2

// Explanation:
// 1. Synchronous code runs first (Script start, Script end)
// 2. Call stack empties
// 3. ALL microtasks run (Promise 1, 3, 2)
// 4. One macrotask runs (setTimeout 1)
// 5. Check microtasks again (none)
// 6. Next macrotask runs (setTimeout 2)
```

**Microtasks (Higher Priority):**

- `Promise.then/catch/finally`
- `queueMicrotask()`
- `MutationObserver`
- `process.nextTick()` (Node.js - even higher priority)

**Macrotasks (Lower Priority):**

- `setTimeout`
- `setInterval`
- `setImmediate` (Node.js)
- I/O operations
- UI rendering

**Complex Example:**

```javascript
console.log('1');

setTimeout(() => {
  console.log('2');
  Promise.resolve().then(() => {
    console.log('3');
  });
}, 0);

Promise.resolve().then(() => {
  console.log('4');
  setTimeout(() => {
    console.log('5');
  }, 0);
});

Promise.resolve().then(() => {
  console.log('6');
});

setTimeout(() => {
  console.log('7');
  Promise.resolve().then(() => {
    console.log('8');
  });
}, 0);

console.log('9');

// Output: 1 → 9 → 4 → 6 → 2 → 3 → 7 → 8 → 5

// Step-by-step:
// 1. console.log('1') - executes
// 2. setTimeout(2) - queued as macrotask
// 3. Promise(4) - queued as microtask
// 4. Promise(6) - queued as microtask
// 5. setTimeout(7) - queued as macrotask
// 6. console.log('9') - executes
// 7. Call stack empty → run microtasks
// 8. Promise(4) executes → queues setTimeout(5) as macrotask
// 9. Promise(6) executes
// 10. Microtasks done → run macrotask setTimeout(2)
// 11. console.log('2') → queues Promise(3) as microtask
// 12. Run microtask Promise(3)
// 13. Run macrotask setTimeout(7)
// 14. console.log('7') → queues Promise(8) as microtask
// 15. Run microtask Promise(8)
// 16. Run macrotask setTimeout(5)
```

**Practical Implications:**

**1. Responsive UI:**

```javascript
// ❌ Blocks UI
function processLargeArray(arr) {
  for (let item of arr) {
    processItem(item); // 10,000 items - UI freezes!
  }
}

// ✅ Chunks work, keeps UI responsive
async function processLargeArrayAsync(arr, chunkSize = 100) {
  for (let i = 0; i < arr.length; i += chunkSize) {
    const chunk = arr.slice(i, i + chunkSize);
    
    // Process chunk
    chunk.forEach(item => processItem(item));
    
    // Yield to event loop (let UI update)
    await new Promise(resolve => setTimeout(resolve, 0));
  }
}
```

**2. Promise Resolution Order:**

```javascript
// Understanding when Promises resolve
const promise1 = Promise.resolve('A');
const promise2 = new Promise(resolve => {
  setTimeout(() => resolve('B'), 0);
});

promise1.then(console.log);
promise2.then(console.log);
console.log('C');

// Output: C → A → B
// Explanation:
// - console.log('C') is synchronous (runs first)
// - promise1.then goes to microtask queue
// - promise2.then waits for setTimeout (macrotask)
// - Microtasks run before macrotasks
```

**3. Event Handler Execution:**

```javascript
button.addEventListener('click', () => {
  console.log('Click 1');
  
  Promise.resolve().then(() => {
    console.log('Promise 1');
  });
  
  console.log('Click 2');
});

button.addEventListener('click', () => {
  console.log('Click 3');
  
  Promise.resolve().then(() => {
    console.log('Promise 2');
  });
  
  console.log('Click 4');
});

// When button clicked:
// Click 1
// Click 2
// Click 3
// Click 4
// Promise 1
// Promise 2

// Both handlers run synchronously, then microtasks
```

**4. Preventing Stack Overflow:**

```javascript
// ❌ Synchronous recursion - stack overflow
function recursiveSync(n) {
  if (n === 0) return;
  console.log(n);
  recursiveSync(n - 1); // Direct recursion - builds stack
}
// recursiveSync(100000); // RangeError: Maximum call stack size exceeded

// ✅ Asynchronous recursion - no stack overflow
function recursiveAsync(n) {
  if (n === 0) return;
  console.log(n);
  setTimeout(() => recursiveAsync(n - 1), 0); // Stack clears between calls
}
recursiveAsync(100000); // Works fine!
```

**Node.js Event Loop (More Complex):**

```javascript
// Node.js has additional queues
const fs = require('fs');

console.log('1: Start');

// Timer queue (macrotask)
setTimeout(() => console.log('2: setTimeout'), 0);

// I/O queue (macrotask)
fs.readFile(__filename, () => {
  console.log('3: readFile');
});

// Immediate queue (macrotask, after I/O)
setImmediate(() => console.log('4: setImmediate'));

// Microtask queue (highest priority)
Promise.resolve().then(() => console.log('5: Promise'));

// Next tick queue (even higher than microtasks in Node)
process.nextTick(() => console.log('6: nextTick'));

console.log('7: End');

// Output (Node.js):
// 1: Start
// 7: End
// 6: nextTick      (nextTick queue)
// 5: Promise       (microtask queue)
// 2: setTimeout    (timer queue)
// 4: setImmediate  (check queue)
// 3: readFile      (I/O queue, varies by file size)
```

**Common Misconceptions:**

**1. setTimeout(fn, 0) runs immediately:**

```javascript
console.log('A');
setTimeout(() => console.log('B'), 0);
console.log('C');

// Output: A → C → B (NOT A → B → C)
// setTimeout always defers to next macrotask
```

**2. Promises are synchronous:**

```javascript
console.log('A');

new Promise((resolve) => {
  console.log('B'); // Constructor callback is SYNCHRONOUS
  resolve();
}).then(() => {
  console.log('C'); // .then is ASYNCHRONOUS
});

console.log('D');

// Output: A → B → D → C
```

**3. All async code runs later:**

```javascript
async function test() {
  console.log('A'); // Runs synchronously until first await
  await Promise.resolve();
  console.log('B'); // Runs asynchronously after await
}

console.log('C');
test();
console.log('D');

// Output: C → A → D → B
```

**Visualizing Event Loop Cycle:**

```
1. Execute all synchronous code (call stack)
2. Call stack empty?
   YES → Go to step 3
   NO → Continue executing
3. Process ALL microtasks
   - Run one microtask
   - If it creates more microtasks, add to queue
   - Repeat until microtask queue is empty
4. Render UI (if browser)
5. Process ONE macrotask
   - Run oldest task from callback queue
6. Go back to step 2

Key: Microtasks can starve macrotasks if infinite!
```

**Best Practices:**
- Don't block the event loop with heavy synchronous operations
- Break large computations into chunks with setTimeout/setImmediate
- Understand microtask priority for Promise timing
- Use Web Workers for CPU-intensive tasks
- Monitor event loop lag in production (Node.js)
- Avoid infinite microtask loops
- Use async/await for readable async code
- Profile and optimize slow synchronous code

**Performance Monitoring:**

```javascript
// Measure event loop delay (Node.js)
const { performance } = require('perf_hooks');

let lastTime = performance.now();

setInterval(() => {
  const currentTime = performance.now();
  const delay = currentTime - lastTime - 1000; // Expected 1000ms
  
  if (delay > 100) {
    console.warn(`Event loop delay: ${delay}ms`);
  }
  
  lastTime = currentTime;
}, 1000);
```

**Key Takeaways:**
- Event loop enables async behavior in single-threaded JavaScript
- Components: call stack, queues (micro/macro), Web APIs, event loop
- Microtasks (Promises) run before macrotasks (setTimeout)
- ALL microtasks run before ANY macrotask
- Synchronous code always runs first
- Don't block the event loop with heavy operations
- Understanding the event loop is crucial for debugging async code
- Foundation of JavaScript's concurrency model

51. What is a Promise?

**Answer:**
A Promise is a JavaScript object representing the eventual completion (or failure) of an asynchronous operation and its resulting value. It's a cleaner alternative to callbacks, providing better error handling and avoiding callback hell.

**Basic Concept:**

```javascript
// A Promise is a placeholder for a future value
const promise = new Promise((resolve, reject) => {
  // Async operation
  setTimeout(() => {
    const success = true;
    
    if (success) {
      resolve('Operation succeeded!'); // Fulfill the promise
    } else {
      reject('Operation failed!'); // Reject the promise
    }
  }, 1000);
});

// Consuming the promise
promise
  .then(result => console.log(result)) // 'Operation succeeded!'
  .catch(error => console.error(error))
  .finally(() => console.log('Done'));
```

**Creating Promises:**

**1. Promise Constructor:**

```javascript
const myPromise = new Promise((resolve, reject) => {
  // resolve(value) - when operation succeeds
  // reject(error) - when operation fails
  
  const randomNum = Math.random();
  
  if (randomNum > 0.5) {
    resolve({ data: 'Success!', value: randomNum });
  } else {
    reject(new Error('Number too low'));
  }
});

myPromise
  .then(result => console.log('✓', result))
  .catch(error => console.error('✗', error.message));
```

**2. Promise.resolve() and Promise.reject():**

```javascript
// Create immediately resolved promise
const resolvedPromise = Promise.resolve('Immediate success');
resolvedPromise.then(console.log); // 'Immediate success'

// Create immediately rejected promise
const rejectedPromise = Promise.reject(new Error('Immediate failure'));
rejectedPromise.catch(console.error); // Error: Immediate failure

// Wrap value in promise
const value = 42;
const wrappedValue = Promise.resolve(value);
wrappedValue.then(val => console.log(val)); // 42

// If value is already a promise, returns it
const existingPromise = Promise.resolve('test');
const samePromise = Promise.resolve(existingPromise);
console.log(existingPromise === samePromise); // true
```

**Promise States:**

A Promise is in one of three states:

1. **Pending** - Initial state, operation not complete
2. **Fulfilled** - Operation completed successfully
3. **Rejected** - Operation failed

```javascript
// Pending → Fulfilled
const fulfilled = new Promise((resolve) => {
  setTimeout(() => resolve('Done'), 1000);
});

// Pending → Rejected
const rejected = new Promise((_, reject) => {
  setTimeout(() => reject(new Error('Failed')), 1000);
});

// Check state (not directly accessible, but observable)
const promise = Promise.resolve('value');
console.log(promise); // Promise { 'value' } - fulfilled

const pendingPromise = new Promise(() => {});
console.log(pendingPromise); // Promise { <pending> }
```

**Consuming Promises:**

**1. .then() - Handle Success:**

```javascript
fetch('https://api.example.com/user/1')
  .then(response => response.json())
  .then(user => {
    console.log('User:', user);
    return user.id; // Return value for next .then()
  })
  .then(userId => {
    console.log('User ID:', userId);
  });

// .then() returns a new promise
const promise1 = Promise.resolve(1);
const promise2 = promise1.then(x => x + 1);
const promise3 = promise2.then(x => x * 2);

promise3.then(console.log); // 4
// Chain: 1 → (1+1) → (2*2) → 4
```

**2. .catch() - Handle Errors:**

```javascript
fetch('https://api.example.com/user/1')
  .then(response => {
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }
    return response.json();
  })
  .then(user => console.log(user))
  .catch(error => {
    console.error('Error:', error.message);
    // Handle error (show message, retry, etc.)
  });

// .catch() catches errors from any previous .then()
Promise.resolve()
  .then(() => {
    throw new Error('Step 1 failed');
  })
  .then(() => {
    console.log('Step 2'); // Skipped
  })
  .catch(error => {
    console.error('Caught:', error.message); // Caught: Step 1 failed
  });
```

**3. .finally() - Cleanup:**

```javascript
showLoadingSpinner();

fetchUserData()
  .then(data => {
    displayData(data);
  })
  .catch(error => {
    showError(error);
  })
  .finally(() => {
    hideLoadingSpinner(); // Runs regardless of success/failure
  });

// .finally() doesn't receive arguments
Promise.resolve('success')
  .finally(() => {
    console.log('Cleanup'); // No access to 'success'
  })
  .then(value => {
    console.log(value); // 'success' - passed through
  });
```

**Promise Chaining:**

```javascript
// Sequential async operations
getUserById(1)
  .then(user => {
    console.log('User:', user.name);
    return getPostsByUser(user.id); // Return promise
  })
  .then(posts => {
    console.log('Posts:', posts.length);
    return getComments(posts[0].id); // Return promise
  })
  .then(comments => {
    console.log('Comments:', comments.length);
  })
  .catch(error => {
    console.error('Error in chain:', error);
  });

// Each .then() returns a new promise
// Errors propagate down to nearest .catch()
```

**Practical Examples:**

**1. Fetch Data:**

```javascript
function fetchUser(id) {
  return fetch(`https://api.example.com/users/${id}`)
    .then(response => {
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}`);
      }
      return response.json();
    })
    .then(user => {
      console.log('Fetched user:', user);
      return user;
    })
    .catch(error => {
      console.error('Failed to fetch user:', error);
      throw error; // Re-throw to let caller handle
    });
}

// Usage
fetchUser(1)
  .then(user => displayUser(user))
  .catch(error => showError(error));
```

**2. Retry Logic:**

```javascript
function fetchWithRetry(url, retries = 3) {
  return fetch(url)
    .then(response => {
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}`);
      }
      return response.json();
    })
    .catch(error => {
      if (retries > 0) {
        console.log(`Retrying... (${retries} attempts left)`);
        return fetchWithRetry(url, retries - 1);
      }
      throw error;
    });
}

fetchWithRetry('https://api.example.com/data')
  .then(data => console.log('Data:', data))
  .catch(error => console.error('Failed after retries:', error));
```

**3. Timeout Wrapper:**

```javascript
function withTimeout(promise, ms) {
  const timeout = new Promise((_, reject) => {
    setTimeout(() => reject(new Error('Timeout')), ms);
  });
  
  return Promise.race([promise, timeout]);
}

// Usage
const slowFetch = fetch('https://slow-api.example.com/data');

withTimeout(slowFetch, 5000)
  .then(response => response.json())
  .then(data => console.log('Data:', data))
  .catch(error => {
    if (error.message === 'Timeout') {
      console.error('Request timed out');
    } else {
      console.error('Request failed:', error);
    }
  });
```

**4. Promise Composition:**

```javascript
// Parallel operations
function loadDashboard(userId) {
  return Promise.all([
    fetchUser(userId),
    fetchPosts(userId),
    fetchNotifications(userId)
  ]).then(([user, posts, notifications]) => {
    return {
      user,
      posts,
      notifications,
      timestamp: Date.now()
    };
  });
}

loadDashboard(1)
  .then(dashboard => console.log('Dashboard:', dashboard))
  .catch(error => console.error('Failed to load dashboard:', error));
```

**Promise vs Callback:**

```javascript
// ❌ Callback hell
getUserById(1, (error, user) => {
  if (error) {
    console.error(error);
    return;
  }
  
  getPosts(user.id, (error, posts) => {
    if (error) {
      console.error(error);
      return;
    }
    
    getComments(posts[0].id, (error, comments) => {
      if (error) {
        console.error(error);
        return;
      }
      
      console.log(comments);
    });
  });
});

// ✅ Promise chain (cleaner)
getUserById(1)
  .then(user => getPosts(user.id))
  .then(posts => getComments(posts[0].id))
  .then(comments => console.log(comments))
  .catch(error => console.error(error)); // Single error handler
```

**Common Promise Methods:**

**1. Promise.all() - All must succeed:**

```javascript
const promises = [
  fetch('/api/user'),
  fetch('/api/posts'),
  fetch('/api/comments')
];

Promise.all(promises)
  .then(responses => Promise.all(responses.map(r => r.json())))
  .then(([user, posts, comments]) => {
    console.log('All data loaded:', { user, posts, comments });
  })
  .catch(error => {
    console.error('At least one failed:', error);
  });

// If ANY promise rejects, Promise.all() rejects immediately
```

**2. Promise.race() - First to finish:**

```javascript
const fast = new Promise(resolve => setTimeout(() => resolve('fast'), 100));
const slow = new Promise(resolve => setTimeout(() => resolve('slow'), 500));

Promise.race([fast, slow])
  .then(result => console.log('Winner:', result)); // 'fast'

// Use for timeouts
Promise.race([
  fetch('/api/data'),
  new Promise((_, reject) => 
    setTimeout(() => reject(new Error('Timeout')), 5000)
  )
]);
```

**3. Promise.allSettled() - Wait for all, regardless:**

```javascript
const promises = [
  Promise.resolve('success'),
  Promise.reject('error'),
  Promise.resolve('another success')
];

Promise.allSettled(promises).then(results => {
  results.forEach(result => {
    if (result.status === 'fulfilled') {
      console.log('✓', result.value);
    } else {
      console.error('✗', result.reason);
    }
  });
});

// Output:
// ✓ success
// ✗ error
// ✓ another success
```

**4. Promise.any() - First fulfilled:**

```javascript
const promises = [
  fetch('https://api1.example.com/data'),
  fetch('https://api2.example.com/data'),
  fetch('https://api3.example.com/data')
];

Promise.any(promises)
  .then(response => {
    console.log('First successful response:', response);
  })
  .catch(error => {
    console.error('All failed:', error); // AggregateError
  });
```

**Error Handling:**

```javascript
// Errors propagate to nearest .catch()
Promise.resolve()
  .then(() => {
    throw new Error('Error in step 1');
  })
  .then(() => {
    console.log('Skipped'); // Skipped
  })
  .catch(error => {
    console.error('Caught:', error.message);
    return 'recovered'; // Recover from error
  })
  .then(value => {
    console.log('Continued:', value); // 'recovered'
  });

// Multiple catch handlers
fetchData()
  .then(processData)
  .catch(error => {
    // Handle specific error
    if (error.name === 'NetworkError') {
      return retryFetch();
    }
    throw error; // Re-throw if can't handle
  })
  .then(displayData)
  .catch(error => {
    // Final error handler
    console.error('Unhandled error:', error);
  });
```

**Common Mistakes:**

**1. Forgetting to return:**

```javascript
// ❌ Doesn't wait for inner promise
getUserById(1)
  .then(user => {
    getPosts(user.id).then(posts => {
      console.log(posts); // Wrong nesting!
    });
  })
  .then(() => {
    // Runs immediately, doesn't wait for posts
  });

// ✅ Return promise to chain properly
getUserById(1)
  .then(user => {
    return getPosts(user.id); // Return!
  })
  .then(posts => {
    console.log(posts); // Runs after posts loaded
  });
```

**2. Not handling rejections:**

```javascript
// ❌ Unhandled promise rejection
fetchData(); // If this fails, unhandled rejection warning

// ✅ Always add .catch()
fetchData().catch(error => console.error(error));

// Or use try/catch with async/await
async function loadData() {
  try {
    await fetchData();
  } catch (error) {
    console.error(error);
  }
}
```

**3. Creating unnecessary promises:**

```javascript
// ❌ Unnecessary wrapping (anti-pattern)
function getUser(id) {
  return new Promise((resolve, reject) => {
    fetchUser(id) // Already returns a promise!
      .then(user => resolve(user))
      .catch(error => reject(error));
  });
}

// ✅ Just return the promise
function getUser(id) {
  return fetchUser(id);
}
```

**Best Practices:**
- Always handle errors with .catch() or try/catch
- Return promises from .then() for proper chaining
- Use Promise.all() for parallel operations
- Avoid creating unnecessary promises (promise constructor anti-pattern)
- Prefer async/await for more readable code
- Don't nest promises (use chaining)
- Use .finally() for cleanup logic
- Add timeout wrappers for network requests

**Key Takeaways:**
- Promise represents eventual completion of async operation
- Three states: pending, fulfilled, rejected
- Created with `new Promise(executor)` or Promise.resolve/reject
- Consumed with .then(), .catch(), .finally()
- Chaining creates sequential async flows
- Cleaner than callbacks, avoids callback hell
- Foundation for async/await syntax
- Essential for modern JavaScript async programming

52. What are Promise states?

**Answer:**
A Promise can be in one of three states: **pending** (initial state), **fulfilled** (operation succeeded), or **rejected** (operation failed). Once a Promise is fulfilled or rejected, it becomes **settled** and its state cannot change (immutable).

**Three Promise States:**

| State | Description | Can Transition To |
|-------|-------------|-------------------|
| **Pending** | Initial state, neither fulfilled nor rejected | Fulfilled or Rejected |
| **Fulfilled** | Operation completed successfully, has a value | None (final state) |
| **Rejected** | Operation failed, has a reason (error) | None (final state) |

**State Transitions:**

```javascript
// Pending → Fulfilled
const fulfilled = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve('Success!'); // Transitions to fulfilled
  }, 1000);
});

// Pending → Rejected
const rejected = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject(new Error('Failed!')); // Transitions to rejected
  }, 1000);
});

// Pending (never settles)
const pending = new Promise(() => {
  // Never calls resolve() or reject()
});
```

**1. Pending State:**

```javascript
// Promise starts in pending state
const promise = new Promise((resolve, reject) => {
  console.log('Promise created'); // Executes immediately
  
  // Async operation
  setTimeout(() => {
    resolve('Done');
  }, 2000);
});

console.log(promise); // Promise { <pending> }

// After 2 seconds
setTimeout(() => {
  console.log(promise); // Promise { 'Done' } - fulfilled
}, 2500);
```

**Characteristics of Pending:**
- Initial state when Promise is created
- Operation is still in progress
- No value or error yet
- Can transition to fulfilled or rejected
- Handlers (.then/.catch) are queued but not executed

```javascript
const pendingPromise = new Promise((resolve) => {
  // Long-running operation
  setTimeout(() => resolve('result'), 5000);
});

// These handlers are queued, waiting for settlement
pendingPromise.then(value => console.log('Value:', value));
pendingPromise.catch(error => console.error('Error:', error));

console.log('Promise is pending...');
// After 5 seconds: 'Value: result'
```

**2. Fulfilled State:**

```javascript
// Promise becomes fulfilled when resolve() is called
const fulfilledPromise = new Promise((resolve, reject) => {
  const data = { id: 1, name: 'John' };
  resolve(data); // Transition to fulfilled with value
});

fulfilledPromise.then(value => {
  console.log('Fulfilled with:', value);
  // { id: 1, name: 'John' }
});

// Immediately fulfilled promise
const immediate = Promise.resolve('Instant success');
console.log(immediate); // Promise { 'Instant success' }
```

**Characteristics of Fulfilled:**
- Operation completed successfully
- Has a resulting value (can be any type, including undefined)
- State is permanent (cannot change)
- `.then()` handlers execute
- `.catch()` handlers are skipped
- `.finally()` handlers execute

```javascript
Promise.resolve('Success')
  .then(value => {
    console.log('✓ Then handler:', value); // Executes
  })
  .catch(error => {
    console.log('✗ Catch handler:', error); // Skipped
  })
  .finally(() => {
    console.log('Finally handler'); // Executes
  });

// Output:
// ✓ Then handler: Success
// Finally handler
```

**3. Rejected State:**

```javascript
// Promise becomes rejected when reject() is called
const rejectedPromise = new Promise((resolve, reject) => {
  const error = new Error('Something went wrong');
  reject(error); // Transition to rejected with reason
});

rejectedPromise.catch(error => {
  console.error('Rejected with:', error.message);
  // 'Something went wrong'
});

// Immediately rejected promise
const immediateReject = Promise.reject(new Error('Instant failure'));
console.log(immediateReject); // Promise { <rejected> Error: Instant failure }
```

**Characteristics of Rejected:**
- Operation failed
- Has a reason (typically an Error object)
- State is permanent (cannot change)
- `.then()` handlers are skipped
- `.catch()` handlers execute
- `.finally()` handlers execute
- Causes "unhandled rejection" warning if not caught

```javascript
Promise.reject('Failure')
  .then(value => {
    console.log('✓ Then handler:', value); // Skipped
  })
  .catch(error => {
    console.log('✗ Catch handler:', error); // Executes
  })
  .finally(() => {
    console.log('Finally handler'); // Executes
  });

// Output:
// ✗ Catch handler: Failure
// Finally handler
```

**State Immutability:**

```javascript
// Once settled, state cannot change
const promise = new Promise((resolve, reject) => {
  resolve('First'); // Promise fulfilled
  resolve('Second'); // ❌ Ignored
  reject('Error'); // ❌ Ignored
  
  // Only the first settlement counts
});

promise.then(value => {
  console.log(value); // 'First' (only first resolve)
});

// Multiple calls have no effect
const promise2 = new Promise((resolve, reject) => {
  resolve('Success');
});

promise2.then(value => console.log(value)); // 'Success'

// Trying to change state externally - impossible
// promise2 is immutable once settled
```

**Checking Promise State (Indirectly):**

```javascript
// You cannot directly access the state property
// But you can check behavior

function getPromiseState(promise) {
  const pending = { state: 'pending' };
  
  return Promise.race([promise, pending])
    .then(
      value => value === pending ? 'pending' : 'fulfilled',
      () => 'rejected'
    );
}

// Usage
const p1 = Promise.resolve('done');
const p2 = Promise.reject('error');
const p3 = new Promise(() => {}); // never settles

getPromiseState(p1).then(console.log); // 'fulfilled'
getPromiseState(p2).then(console.log); // 'rejected'
getPromiseState(p3).then(console.log); // 'pending'
```

**State Transitions in Practice:**

**Example 1: Network Request**

```javascript
function fetchUser(id) {
  // Returns a pending promise
  const promise = fetch(`/api/users/${id}`)
    .then(response => {
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}`);
      }
      return response.json(); // Still pending
    })
    .then(user => {
      // Promise fulfilled with user data
      return user;
    })
    .catch(error => {
      // Promise rejected with error
      throw error;
    });
  
  return promise;
}

// Usage
const userPromise = fetchUser(1); // Pending
console.log(userPromise); // Promise { <pending> }

userPromise
  .then(user => {
    // Fulfilled
    console.log('User loaded:', user);
  })
  .catch(error => {
    // Rejected
    console.error('Failed to load user:', error);
  });
```

**Example 2: Multiple State Transitions**

```javascript
// Step-by-step state transitions
const promise = new Promise((resolve, reject) => {
  console.log('1. Promise created (pending)');
  
  setTimeout(() => {
    console.log('2. About to resolve...');
    resolve('Success');
    console.log('3. Promise fulfilled');
  }, 1000);
});

console.log('Initial state: pending');

promise.then(value => {
  console.log('4. Handler executed with:', value);
});

// Output:
// 1. Promise created (pending)
// Initial state: pending
// (1 second later)
// 2. About to resolve...
// 3. Promise fulfilled
// 4. Handler executed with: Success
```

**Settled vs Pending:**

```javascript
// "Settled" means either fulfilled or rejected (not pending)

function isSettled(promise) {
  return Promise.race([
    promise.then(() => true, () => true),
    new Promise(resolve => setTimeout(() => resolve(false), 0))
  ]);
}

const settled = Promise.resolve('done');
const pending = new Promise(() => {});

isSettled(settled).then(result => {
  console.log('Settled?', result); // true
});

isSettled(pending).then(result => {
  console.log('Pending?', !result); // true
});
```

**Handling All States:**

```javascript
function handlePromise(promise) {
  // Show loading (pending)
  showLoadingSpinner();
  
  promise
    .then(value => {
      // Fulfilled
      hideLoadingSpinner();
      displaySuccess(value);
    })
    .catch(error => {
      // Rejected
      hideLoadingSpinner();
      displayError(error);
    })
    .finally(() => {
      // Always runs (fulfilled or rejected)
      enableButtons();
    });
}

// Usage
const dataPromise = fetchData();
handlePromise(dataPromise);
```

**State Transitions with async/await:**

```javascript
async function processData() {
  try {
    // Promise returned by fetch starts in pending
    const response = await fetch('/api/data'); // Waits for fulfilled
    
    // If fulfilled, continue
    const data = await response.json();
    return data; // Returns fulfilled promise with data
    
  } catch (error) {
    // If rejected, catch block runs
    console.error('Error:', error);
    throw error; // Returns rejected promise
  }
}

// The async function itself returns a promise
const resultPromise = processData(); // Pending
console.log(resultPromise); // Promise { <pending> }

resultPromise
  .then(data => console.log('Success:', data)) // Fulfilled
  .catch(error => console.error('Failed:', error)); // Rejected
```

**Common Patterns:**

**1. Creating Pre-Settled Promises:**

```javascript
// Immediately fulfilled
const fulfilled = Promise.resolve('value');

// Immediately rejected
const rejected = Promise.reject(new Error('error'));

// Use for testing or wrapping sync values
function getUserFromCache(id) {
  const cached = cache.get(id);
  
  if (cached) {
    return Promise.resolve(cached); // Already fulfilled
  }
  
  return fetch(`/api/users/${id}`); // Pending promise
}
```

**2. Converting Callbacks to Promises:**

```javascript
function readFilePromise(path) {
  return new Promise((resolve, reject) => {
    // Starts in pending state
    
    fs.readFile(path, 'utf8', (error, data) => {
      if (error) {
        reject(error); // Transition to rejected
      } else {
        resolve(data); // Transition to fulfilled
      }
    });
  });
}

// Usage
const filePromise = readFilePromise('file.txt'); // Pending
filePromise
  .then(content => console.log('Content:', content)) // Fulfilled
  .catch(error => console.error('Error:', error)); // Rejected
```

**3. State-Based UI Updates:**

```javascript
class DataLoader {
  constructor() {
    this.state = 'idle'; // Not started yet
  }
  
  async load() {
    this.state = 'loading'; // Similar to pending
    this.updateUI();
    
    try {
      const data = await fetchData();
      this.state = 'success'; // Similar to fulfilled
      this.data = data;
    } catch (error) {
      this.state = 'error'; // Similar to rejected
      this.error = error;
    }
    
    this.updateUI();
  }
  
  updateUI() {
    if (this.state === 'loading') {
      showSpinner();
    } else if (this.state === 'success') {
      hideSpinner();
      displayData(this.data);
    } else if (this.state === 'error') {
      hideSpinner();
      displayError(this.error);
    }
  }
}
```

**Error States:**

```javascript
// Different ways to reject
const promise1 = new Promise((resolve, reject) => {
  reject(new Error('Error object')); // ✅ Best practice
});

const promise2 = new Promise((resolve, reject) => {
  reject('String error'); // ⚠️ Works but not recommended
});

const promise3 = new Promise((resolve, reject) => {
  throw new Error('Thrown error'); // ✅ Automatically rejected
});

const promise4 = Promise.reject(new Error('Immediate reject')); // ✅

// All transition to rejected state
```

**Unhandled Rejections:**

```javascript
// ❌ Unhandled rejection warning
const promise = Promise.reject(new Error('Oops'));
// Warning: Unhandled promise rejection

// ✅ Always handle rejections
Promise.reject(new Error('Oops'))
  .catch(error => console.error('Handled:', error));

// ✅ Or use global handler
window.addEventListener('unhandledrejection', event => {
  console.error('Unhandled rejection:', event.reason);
  event.preventDefault(); // Prevent default warning
});

process.on('unhandledRejection', (reason, promise) => {
  console.error('Unhandled rejection at:', promise, 'reason:', reason);
});
```

**Best Practices:**
- Promises start in **pending** state
- **Resolve** for success, **reject** for failure
- State is **immutable** once settled
- Always handle both fulfilled and rejected states
- Use `.catch()` to handle rejection
- Use `.finally()` for cleanup regardless of state
- Create Error objects for rejections (not strings)
- Monitor unhandled rejections in production
- Understand state to debug async flows

**Key Takeaways:**
- Three states: **pending**, **fulfilled**, **rejected**
- Pending → Fulfilled (via resolve)
- Pending → Rejected (via reject)
- Once settled (fulfilled/rejected), state cannot change
- Fulfilled promises have a value
- Rejected promises have a reason (error)
- Settled = fulfilled OR rejected (not pending)
- Understanding states is crucial for async error handling

53. What is the difference between Promise.all() and Promise.race()?

**Answer:**
`Promise.all()` waits for all promises to fulfill (or any to reject) and returns an array of results, while `Promise.race()` returns as soon as any promise settles (fulfilled or rejected), with that promise's result.

**Promise.all() - All or Nothing:**

**Syntax:** `Promise.all(iterable)`

**Behavior:**
- Waits for **all** promises to fulfill
- Returns array of results in same order
- Rejects immediately if **any** promise rejects
- Use for parallel operations that all must succeed

```javascript
const promise1 = Promise.resolve(1);
const promise2 = Promise.resolve(2);
const promise3 = Promise.resolve(3);

Promise.all([promise1, promise2, promise3])
  .then(results => {
    console.log(results); // [1, 2, 3]
    // All promises fulfilled
  })
  .catch(error => {
    console.error('At least one failed:', error);
  });

// Real-world example: Load multiple resources
Promise.all([
  fetch('/api/user'),
  fetch('/api/posts'),
  fetch('/api/comments')
])
  .then(responses => Promise.all(responses.map(r => r.json())))
  .then(([user, posts, comments]) => {
    console.log('All data loaded:', { user, posts, comments });
  })
  .catch(error => {
    console.error('Failed to load data:', error);
  });
```

**Promise.all() Rejection:**

```javascript
// If ANY promise rejects, Promise.all() rejects immediately
const promises = [
  Promise.resolve('Success 1'),
  Promise.reject('Error!'), // This causes rejection
  Promise.resolve('Success 2')
];

Promise.all(promises)
  .then(results => {
    console.log('All succeeded:', results); // Never runs
  })
  .catch(error => {
    console.error('Failed:', error); // 'Error!'
    // Other promises are ignored
  });

// Only the first rejection is caught
const promises2 = [
  Promise.reject('Error 1'),
  Promise.reject('Error 2'),
  Promise.resolve('Success')
];

Promise.all(promises2)
  .catch(error => {
    console.error(error); // 'Error 1' (first rejection)
  });
```

**Promise.race() - First to Finish:**

**Syntax:** `Promise.race(iterable)`

**Behavior:**
- Returns as soon as **any** promise settles
- Settles with the value/reason of the first settled promise
- Other promises continue but are ignored
- Use for timeouts or fastest response

```javascript
const slow = new Promise(resolve => 
  setTimeout(() => resolve('slow'), 2000)
);
const fast = new Promise(resolve => 
  setTimeout(() => resolve('fast'), 100)
);

Promise.race([slow, fast])
  .then(result => {
    console.log('Winner:', result); // 'fast'
    // Slow promise still runs but result is ignored
  });

// Real-world example: Request with timeout
const dataPromise = fetch('/api/data');
const timeoutPromise = new Promise((_, reject) =>
  setTimeout(() => reject(new Error('Timeout')), 5000)
);

Promise.race([dataPromise, timeoutPromise])
  .then(response => response.json())
  .then(data => console.log('Data:', data))
  .catch(error => console.error('Failed or timed out:', error));
```

**Key Differences:**

| Feature | Promise.all() | Promise.race() |
|---------|---------------|----------------|
| **Waits for** | All promises | First promise |
| **Returns** | Array of all results | Single result (first) |
| **Fulfills when** | All fulfill | First fulfills |
| **Rejects when** | Any rejects | First rejects |
| **Use case** | All operations needed | First response wins |
| **Result order** | Same as input order | N/A (single result) |

**Detailed Comparison:**

**1. Fulfillment:**

```javascript
// Promise.all - waits for all
const all = Promise.all([
  delay(100, 'A'),
  delay(200, 'B'),
  delay(50, 'C')
]);

all.then(results => {
  console.log(results); // ['A', 'B', 'C'] after 200ms
  // Waits for slowest (200ms)
});

// Promise.race - first to finish
const race = Promise.race([
  delay(100, 'A'),
  delay(200, 'B'),
  delay(50, 'C')
]);

race.then(result => {
  console.log(result); // 'C' after 50ms
  // Returns fastest (50ms)
});

function delay(ms, value) {
  return new Promise(resolve => setTimeout(() => resolve(value), ms));
}
```

**2. Rejection:**

```javascript
// Promise.all - fails fast on ANY rejection
const all = Promise.all([
  delay(100, 'A'),
  Promise.reject('Error'), // Immediate rejection
  delay(200, 'B')
]);

all.catch(error => {
  console.error('All failed:', error); // 'Error' immediately
  // Doesn't wait for other promises
});

// Promise.race - returns first (fulfill or reject)
const race = Promise.race([
  delay(100, 'A'),
  Promise.reject('Error'), // Immediate rejection
  delay(200, 'B')
]);

race.catch(error => {
  console.error('Race failed:', error); // 'Error' immediately
  // First to settle (reject) wins
});
```

**Practical Use Cases:**

**Promise.all() Use Cases:**

**1. Load Multiple Resources:**

```javascript
async function loadDashboard() {
  try {
    const [user, posts, comments, stats] = await Promise.all([
      fetchUser(userId),
      fetchPosts(),
      fetchComments(),
      fetchStats()
    ]);
    
    return { user, posts, comments, stats };
  } catch (error) {
    console.error('Failed to load dashboard:', error);
  }
}

// Much faster than sequential loading
// Sequential: 2s + 2s + 1s + 1s = 6s
// Parallel: max(2s, 2s, 1s, 1s) = 2s
```

**2. Batch Operations:**

```javascript
async function createMultipleUsers(userDataArray) {
  const promises = userDataArray.map(userData => 
    createUser(userData)
  );
  
  try {
    const users = await Promise.all(promises);
    console.log('All users created:', users);
    return users;
  } catch (error) {
    console.error('Failed to create some users:', error);
    // If one fails, all fail
  }
}
```

**3. Parallel File Processing:**

```javascript
const fs = require('fs').promises;

async function processFiles(filePaths) {
  const readPromises = filePaths.map(path => 
    fs.readFile(path, 'utf8')
  );
  
  try {
    const contents = await Promise.all(readPromises);
    
    // Process all contents
    const processed = contents.map(content => 
      processContent(content)
    );
    
    return processed;
  } catch (error) {
    console.error('Failed to read files:', error);
  }
}
```

**4. Validation:**

```javascript
async function validateForm(formData) {
  // Run all validations in parallel
  const validations = await Promise.all([
    validateEmail(formData.email),
    validatePassword(formData.password),
    validateUsername(formData.username),
    checkUsernameAvailable(formData.username)
  ]);
  
  // All validations must pass
  return validations.every(result => result.valid);
}
```

**Promise.race() Use Cases:**

**1. Timeout Wrapper:**

```javascript
function fetchWithTimeout(url, timeout = 5000) {
  const fetchPromise = fetch(url);
  const timeoutPromise = new Promise((_, reject) =>
    setTimeout(() => reject(new Error('Request timeout')), timeout)
  );
  
  return Promise.race([fetchPromise, timeoutPromise]);
}

// Usage
try {
  const response = await fetchWithTimeout('/api/data', 3000);
  const data = await response.json();
  console.log('Data:', data);
} catch (error) {
  if (error.message === 'Request timeout') {
    console.error('Request timed out');
  } else {
    console.error('Request failed:', error);
  }
}
```

**2. First Successful Response:**

```javascript
async function fetchFromMultipleSources() {
  const urls = [
    'https://api1.example.com/data',
    'https://api2.example.com/data',
    'https://api3.example.com/data'
  ];
  
  try {
    // Use first successful response
    const response = await Promise.race(urls.map(url => fetch(url)));
    return await response.json();
  } catch (error) {
    console.error('All sources failed:', error);
  }
}
```

**3. User Interaction Timeout:**

```javascript
async function getUserInput() {
  const userInputPromise = waitForUserClick();
  const timeoutPromise = delay(10000, 'timeout');
  
  const result = await Promise.race([
    userInputPromise,
    timeoutPromise
  ]);
  
  if (result === 'timeout') {
    console.log('User took too long');
    return null;
  }
  
  return result;
}
```

**4. Cache with Fallback:**

```javascript
async function getDataWithCache(key) {
  const cachePromise = getFromCache(key);
  const networkPromise = fetchFromNetwork(key);
  
  // Return whichever resolves first
  try {
    const data = await Promise.race([cachePromise, networkPromise]);
    console.log('Got data from:', data.source);
    return data;
  } catch (error) {
    console.error('Both failed:', error);
  }
}
```

**Advanced Patterns:**

**1. Promise.all() with Error Handling:**

```javascript
// Wrap each promise to handle errors individually
async function allWithErrorHandling(promises) {
  const wrappedPromises = promises.map(promise =>
    promise
      .then(value => ({ status: 'fulfilled', value }))
      .catch(error => ({ status: 'rejected', reason: error }))
  );
  
  return Promise.all(wrappedPromises);
}

// Usage
const results = await allWithErrorHandling([
  fetchUser(1),
  fetchUser(2),
  fetchUser(3)
]);

results.forEach((result, index) => {
  if (result.status === 'fulfilled') {
    console.log(`User ${index + 1}:`, result.value);
  } else {
    console.error(`User ${index + 1} failed:`, result.reason);
  }
});

// Modern alternative: Promise.allSettled()
const results2 = await Promise.allSettled([
  fetchUser(1),
  fetchUser(2),
  fetchUser(3)
]);
```

**2. Race with All Results:**

```javascript
// Get first result but track all
function raceWithAll(promises) {
  const results = [];
  let firstResult;
  let firstError;
  
  const wrappers = promises.map((promise, index) =>
    promise
      .then(value => {
        results[index] = { status: 'fulfilled', value };
        if (firstResult === undefined) {
          firstResult = value;
        }
        return value;
      })
      .catch(error => {
        results[index] = { status: 'rejected', reason: error };
        if (firstError === undefined) {
          firstError = error;
        }
        throw error;
      })
  );
  
  return Promise.race(wrappers).then(
    value => ({ winner: value, results }),
    error => ({ winner: error, results })
  );
}
```

**3. Batch with Concurrency Limit:**

```javascript
// Process promises in batches (not all at once)
async function promiseAllWithLimit(promises, limit) {
  const results = [];
  const executing = [];
  
  for (const [index, promise] of promises.entries()) {
    const p = Promise.resolve(promise).then(value => {
      results[index] = value;
      return value;
    });
    
    results.push(p);
    
    if (limit <= promises.length) {
      const e = p.then(() => executing.splice(executing.indexOf(e), 1));
      executing.push(e);
      
      if (executing.length >= limit) {
        await Promise.race(executing);
      }
    }
  }
  
  return Promise.all(results);
}

// Process max 5 at a time
const userIds = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
const userPromises = userIds.map(id => fetchUser(id));
const users = await promiseAllWithLimit(userPromises, 5);
```

**Related Methods:**

**Promise.allSettled() - Like all() but doesn't fail:**

```javascript
const promises = [
  Promise.resolve('Success 1'),
  Promise.reject('Error'),
  Promise.resolve('Success 2')
];

const results = await Promise.allSettled(promises);
console.log(results);
// [
//   { status: 'fulfilled', value: 'Success 1' },
//   { status: 'rejected', reason: 'Error' },
//   { status: 'fulfilled', value: 'Success 2' }
// ]

// Use when you want all results regardless of failures
```

**Promise.any() - First fulfilled (ignores rejections):**

```javascript
const promises = [
  Promise.reject('Error 1'),
  Promise.resolve('Success'),
  Promise.reject('Error 2')
];

const result = await Promise.any(promises);
console.log(result); // 'Success'

// Only fails if ALL reject
const allFail = [
  Promise.reject('Error 1'),
  Promise.reject('Error 2')
];

try {
  await Promise.any(allFail);
} catch (error) {
  console.error('All failed:', error); // AggregateError
}
```

**Comparison Table:**

| Method | Fulfills When | Rejects When | Use When |
|--------|---------------|--------------|----------|
| `Promise.all()` | All fulfill | Any rejects | Need all results |
| `Promise.race()` | First settles | First rejects | First response wins |
| `Promise.allSettled()` | All settle | Never (always fulfills) | Want all outcomes |
| `Promise.any()` | First fulfills | All reject | Any success is enough |

**Performance Considerations:**

```javascript
// Sequential (slow) - 6 seconds total
const user = await fetchUser(1);      // 2s
const posts = await fetchPosts();     // 2s
const comments = await fetchComments(); // 2s

// Promise.all (fast) - 2 seconds total (parallel)
const [user, posts, comments] = await Promise.all([
  fetchUser(1),      // 2s
  fetchPosts(),      // 2s
  fetchComments()    // 2s
]);

// Promise.race - 0.5 seconds (fastest)
const fastest = await Promise.race([
  fetchFromServer1(), // 2s
  fetchFromServer2(), // 0.5s ← wins
  fetchFromServer3()  // 1s
]);
```

**Common Mistakes:**

**1. Using race when you need all:**

```javascript
// ❌ Wrong - only gets first result
const result = await Promise.race([
  fetchUser(1),
  fetchPosts(),
  fetchComments()
]);
// Missing other data!

// ✅ Correct - gets all results
const [user, posts, comments] = await Promise.all([
  fetchUser(1),
  fetchPosts(),
  fetchComments()
]);
```

**2. Not handling all errors:**

```javascript
// ❌ One failure stops everything
await Promise.all([
  fetchUser(1),
  fetchUser(2),  // Fails
  fetchUser(3)   // Never processed
]);

// ✅ Handle errors individually
const results = await Promise.allSettled([
  fetchUser(1),
  fetchUser(2),
  fetchUser(3)
]);
```

**Best Practices:**
- Use **Promise.all()** for parallel operations (all needed)
- Use **Promise.race()** for timeouts or fastest response
- Use **Promise.allSettled()** when all outcomes matter
- Use **Promise.any()** when any success is sufficient
- Handle errors appropriately for each method
- Consider concurrency limits for many promises
- Monitor performance in production
- Document expected behavior for team

**Key Takeaways:**
- **Promise.all()**: Waits for all, rejects if any fails
- **Promise.race()**: Returns first to settle (fulfill or reject)
- all() returns array of results in order
- race() returns single result (first)
- all() is for parallel "must-have-all" operations
- race() is for timeouts and fastest-response scenarios
- allSettled() and any() are modern alternatives
- Understanding differences prevents bugs in async code

54. What is async/await?

**Answer:**
`async/await` is syntactic sugar built on top of Promises that makes asynchronous code look and behave like synchronous code. An `async` function always returns a Promise, and `await` pauses execution until a Promise resolves, making async code more readable and easier to debug.

**Basic Syntax:**

```javascript
// Function declared with async keyword
async function fetchData() {
  // await pauses execution until promise resolves
  const response = await fetch('/api/data');
  const data = await response.json();
  return data; // Automatically wrapped in Promise
}

// Equivalent using Promises
function fetchDataPromise() {
  return fetch('/api/data')
    .then(response => response.json())
    .then(data => data);
}

// Both return promises
fetchData().then(data => console.log(data));
fetchDataPromise().then(data => console.log(data));
```

**The async Keyword:**

```javascript
// async makes function return a Promise
async function getMessage() {
  return 'Hello'; // Automatically wrapped in Promise.resolve()
}

// Equivalent to:
function getMessagePromise() {
  return Promise.resolve('Hello');
}

// Both work the same
getMessage().then(msg => console.log(msg)); // 'Hello'
getMessagePromise().then(msg => console.log(msg)); // 'Hello'

// async function always returns Promise
async function getNumber() {
  return 42;
}

console.log(getNumber()); // Promise { 42 }

// Can also return Promise explicitly
async function getUser() {
  return Promise.resolve({ id: 1, name: 'John' });
}
```

**The await Keyword:**

```javascript
// await pauses execution until Promise settles
async function example() {
  console.log('1: Start');
  
  const result = await Promise.resolve('2: Resolved');
  console.log(result);
  
  console.log('3: End');
}

example();
console.log('4: After function call');

// Output:
// 1: Start
// 4: After function call
// 2: Resolved
// 3: End

// await only works inside async functions
function notAsync() {
  // const result = await fetch('/api/data'); // ❌ SyntaxError
}

// ✅ Must be in async function
async function isAsync() {
  const result = await fetch('/api/data'); // ✅ Works
}
```

**Error Handling:**

**Using try/catch:**

```javascript
async function fetchUser(id) {
  try {
    const response = await fetch(`/api/users/${id}`);
    
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }
    
    const user = await response.json();
    return user;
    
  } catch (error) {
    console.error('Error fetching user:', error);
    throw error; // Re-throw or handle
  }
}

// Usage
try {
  const user = await fetchUser(1);
  console.log('User:', user);
} catch (error) {
  console.error('Failed:', error);
}
```

**Multiple try/catch blocks:**

```javascript
async function loadData() {
  let user, posts;
  
  // Try user fetch
  try {
    user = await fetchUser(1);
    console.log('User loaded:', user);
  } catch (error) {
    console.error('User fetch failed:', error);
    user = null; // Provide default
  }
  
  // Try posts fetch (continues even if user failed)
  try {
    posts = await fetchPosts();
    console.log('Posts loaded:', posts);
  } catch (error) {
    console.error('Posts fetch failed:', error);
    posts = [];
  }
  
  return { user, posts };
}
```

**Sequential vs Parallel Execution:**

**Sequential (one after another):**

```javascript
// Waits for each operation before starting next
async function sequential() {
  console.time('sequential');
  
  const user = await fetchUser(1);        // 2 seconds
  const posts = await fetchPosts();       // 2 seconds
  const comments = await fetchComments(); // 2 seconds
  
  console.timeEnd('sequential'); // ~6 seconds
  
  return { user, posts, comments };
}
```

**Parallel (all at once):**

```javascript
// Starts all operations simultaneously
async function parallel() {
  console.time('parallel');
  
  // Start all promises
  const [user, posts, comments] = await Promise.all([
    fetchUser(1),       // 2 seconds
    fetchPosts(),       // 2 seconds
    fetchComments()     // 2 seconds
  ]);
  
  console.timeEnd('parallel'); // ~2 seconds (much faster!)
  
  return { user, posts, comments };
}

// Alternative syntax
async function parallelAlt() {
  // Start all (don't await yet)
  const userPromise = fetchUser(1);
  const postsPromise = fetchPosts();
  const commentsPromise = fetchComments();
  
  // Now await all
  const user = await userPromise;
  const posts = await postsPromise;
  const comments = await commentsPromise;
  
  return { user, posts, comments };
}
```

**Conditional Await:**

```javascript
async function loadUserData(userId, includeDetails = false) {
  // Always load user
  const user = await fetchUser(userId);
  
  if (includeDetails) {
    // Only load if needed
    user.posts = await fetchPosts(userId);
    user.comments = await fetchComments(userId);
  }
  
  return user;
}

// Dependent operations
async function processOrder(orderId) {
  const order = await fetchOrder(orderId);
  
  if (order.status === 'pending') {
    // Only process if pending
    const payment = await processPayment(order);
    const shipment = await createShipment(order);
    return { order, payment, shipment };
  }
  
  return { order };
}
```

**Practical Examples:**

**1. API Request:**

```javascript
async function getUser(id) {
  try {
    const response = await fetch(`https://api.example.com/users/${id}`);
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    const user = await response.json();
    console.log('User:', user);
    return user;
    
  } catch (error) {
    console.error('Failed to fetch user:', error);
    return null;
  }
}

// Usage
const user = await getUser(1);
if (user) {
  displayUser(user);
} else {
  showError('User not found');
}
```

**2. Multiple API Calls:**

```javascript
async function loadDashboard(userId) {
  try {
    // Load all data in parallel
    const [user, posts, notifications, stats] = await Promise.all([
      fetchUser(userId),
      fetchUserPosts(userId),
      fetchNotifications(userId),
      fetchUserStats(userId)
    ]);
    
    return {
      user,
      posts,
      notifications,
      stats,
      loadedAt: new Date()
    };
    
  } catch (error) {
    console.error('Dashboard load failed:', error);
    throw error;
  }
}

// Usage
async function initDashboard() {
  showLoadingSpinner();
  
  try {
    const dashboardData = await loadDashboard(currentUserId);
    renderDashboard(dashboardData);
  } catch (error) {
    showError('Failed to load dashboard');
  } finally {
    hideLoadingSpinner();
  }
}
```

**3. Form Submission:**

```javascript
async function handleSubmit(event) {
  event.preventDefault();
  
  const formData = new FormData(event.target);
  const data = Object.fromEntries(formData);
  
  // Disable button
  submitButton.disabled = true;
  submitButton.textContent = 'Submitting...';
  
  try {
    const response = await fetch('/api/submit', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data)
    });
    
    if (!response.ok) {
      const error = await response.json();
      throw new Error(error.message);
    }
    
    const result = await response.json();
    showSuccess('Form submitted successfully!');
    return result;
    
  } catch (error) {
    showError(`Submission failed: ${error.message}`);
  } finally {
    // Re-enable button
    submitButton.disabled = false;
    submitButton.textContent = 'Submit';
  }
}

form.addEventListener('submit', handleSubmit);
```

**4. Retry Logic:**

```javascript
async function fetchWithRetry(url, options = {}, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      console.log(`Attempt ${i + 1}/${maxRetries}`);
      
      const response = await fetch(url, options);
      
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}`);
      }
      
      return await response.json();
      
    } catch (error) {
      console.error(`Attempt ${i + 1} failed:`, error.message);
      
      // If last retry, throw error
      if (i === maxRetries - 1) {
        throw new Error(`Failed after ${maxRetries} attempts`);
      }
      
      // Exponential backoff
      const delay = Math.pow(2, i) * 1000;
      console.log(`Retrying in ${delay}ms...`);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
}

// Usage
try {
  const data = await fetchWithRetry('https://api.example.com/data');
  console.log('Data:', data);
} catch (error) {
  console.error('All retries failed:', error);
}
```

**5. Async Loops:**

```javascript
// Sequential processing
async function processUsersSequentially(userIds) {
  const results = [];
  
  for (const id of userIds) {
    const user = await fetchUser(id); // Waits for each
    const processed = await processUser(user);
    results.push(processed);
  }
  
  return results; // Slow: 10 users × 2s = 20s
}

// Parallel processing
async function processUsersParallel(userIds) {
  const promises = userIds.map(async (id) => {
    const user = await fetchUser(id);
    return await processUser(user);
  });
  
  return await Promise.all(promises); // Fast: ~2s
}

// Controlled concurrency
async function processUsersBatch(userIds, batchSize = 5) {
  const results = [];
  
  for (let i = 0; i < userIds.length; i += batchSize) {
    const batch = userIds.slice(i, i + batchSize);
    
    const batchResults = await Promise.all(
      batch.map(id => fetchUser(id))
    );
    
    results.push(...batchResults);
    console.log(`Processed batch ${i / batchSize + 1}`);
  }
  
  return results;
}
```

**6. Async Iteration:**

```javascript
// Async generator
async function* fetchPages(url) {
  let page = 1;
  let hasMore = true;
  
  while (hasMore) {
    const response = await fetch(`${url}?page=${page}`);
    const data = await response.json();
    
    yield data;
    
    hasMore = data.hasMore;
    page++;
  }
}

// Usage
async function loadAllPages() {
  for await (const pageData of fetchPages('/api/items')) {
    console.log('Page:', pageData);
    displayItems(pageData.items);
  }
}
```

**Top-Level await (ES2022):**

```javascript
// In modules, you can use await at top level
// file: data.js
export const users = await fetch('/api/users').then(r => r.json());
export const posts = await fetch('/api/posts').then(r => r.json());

// file: app.js
import { users, posts } from './data.js';
console.log('Data loaded:', users, posts);

// Before: had to wrap in async function
// (async () => {
//   const users = await fetchUsers();
//   console.log(users);
// })();
```

**Common Patterns:**

**1. Wrapper with Timeout:**

```javascript
function withTimeout(promise, ms) {
  return Promise.race([
    promise,
    new Promise((_, reject) =>
      setTimeout(() => reject(new Error('Timeout')), ms)
    )
  ]);
}

// Usage
async function fetchWithTimeout(url) {
  try {
    const response = await withTimeout(fetch(url), 5000);
    return await response.json();
  } catch (error) {
    if (error.message === 'Timeout') {
      console.error('Request timed out');
    }
    throw error;
  }
}
```

**2. Async Reduce:**

```javascript
async function asyncReduce(array, asyncFn, initialValue) {
  let accumulator = initialValue;
  
  for (const item of array) {
    accumulator = await asyncFn(accumulator, item);
  }
  
  return accumulator;
}

// Usage
const userIds = [1, 2, 3, 4, 5];
const totalPosts = await asyncReduce(
  userIds,
  async (total, userId) => {
    const posts = await fetchUserPosts(userId);
    return total + posts.length;
  },
  0
);
console.log('Total posts:', totalPosts);
```

**3. Async Event Handler:**

```javascript
button.addEventListener('click', async (event) => {
  event.target.disabled = true;
  
  try {
    const data = await fetchData();
    displayData(data);
  } catch (error) {
    showError(error.message);
  } finally {
    event.target.disabled = false;
  }
});
```

**Common Mistakes:**

**1. Forgetting await:**

```javascript
// ❌ Doesn't wait (returns Promise)
async function getUser() {
  const user = fetchUser(1); // Missing await!
  console.log(user); // Promise { <pending> }
  return user.name; // Error: undefined
}

// ✅ Correct
async function getUser() {
  const user = await fetchUser(1);
  console.log(user); // { id: 1, name: 'John' }
  return user.name; // 'John'
}
```

**2. Using await in non-async function:**

```javascript
// ❌ SyntaxError
function getData() {
  const data = await fetchData(); // Error!
  return data;
}

// ✅ Must be async
async function getData() {
  const data = await fetchData();
  return data;
}
```

**3. Sequential when could be parallel:**

```javascript
// ❌ Slow - 6 seconds
async function loadData() {
  const user = await fetchUser();      // 2s
  const posts = await fetchPosts();    // 2s
  const comments = await fetchComments(); // 2s
  return { user, posts, comments };
}

// ✅ Fast - 2 seconds
async function loadData() {
  const [user, posts, comments] = await Promise.all([
    fetchUser(),
    fetchPosts(),
    fetchComments()
  ]);
  return { user, posts, comments };
}
```

**4. Not handling errors:**

```javascript
// ❌ Unhandled errors
async function riskyOperation() {
  const data = await fetchData(); // Might throw
  return data;
}

// ✅ Handle errors
async function safeOperation() {
  try {
    const data = await fetchData();
    return data;
  } catch (error) {
    console.error('Error:', error);
    return null; // or throw
  }
}
```

**5. Async in forEach:**

```javascript
// ❌ Doesn't wait for async operations
const userIds = [1, 2, 3];
userIds.forEach(async (id) => {
  const user = await fetchUser(id);
  console.log(user);
});
console.log('Done'); // Prints before users

// ✅ Use for...of
for (const id of userIds) {
  const user = await fetchUser(id);
  console.log(user);
}
console.log('Done'); // Prints after all users

// ✅ Or Promise.all
await Promise.all(
  userIds.map(id => fetchUser(id))
);
```

**Async/Await vs Promises:**

```javascript
// Promises
function loadUser(id) {
  return fetchUser(id)
    .then(user => fetchPosts(user.id))
    .then(posts => fetchComments(posts[0].id))
    .then(comments => comments)
    .catch(error => {
      console.error('Error:', error);
      throw error;
    });
}

// Async/await (more readable)
async function loadUser(id) {
  try {
    const user = await fetchUser(id);
    const posts = await fetchPosts(user.id);
    const comments = await fetchComments(posts[0].id);
    return comments;
  } catch (error) {
    console.error('Error:', error);
    throw error;
  }
}
```

**Best Practices:**
- Use **async/await** for cleaner, more readable async code
- Always use **try/catch** for error handling
- Run **independent operations in parallel** with Promise.all()
- Don't use await in loops unless sequential is required
- Handle errors at appropriate levels
- Use finally for cleanup operations
- Avoid mixing async/await with .then/.catch
- Consider top-level await in ES modules
- Test async error paths thoroughly

**Key Takeaways:**
- **async** function always returns a Promise
- **await** pauses execution until Promise resolves
- Works only inside async functions (or top-level in modules)
- Makes async code look synchronous
- Use **try/catch** for error handling
- Run independent operations in parallel (Promise.all)
- Cleaner than Promise chains
- Foundation of modern async JavaScript

55. What is the difference between setTimeout and setInterval?

**Answer:**
`setTimeout()` executes a function once after a specified delay, while `setInterval()` executes a function repeatedly at specified time intervals until cleared. setTimeout runs once, setInterval runs continuously.

**setTimeout() - Execute Once:**

**Syntax:** `setTimeout(callback, delay, arg1, arg2, ...)`

```javascript
// Basic usage
setTimeout(() => {
  console.log('Executed after 2 seconds');
}, 2000);

console.log('This runs immediately');

// Output:
// This runs immediately
// (2 seconds later)
// Executed after 2 seconds

// With arguments
setTimeout((name, age) => {
  console.log(`${name} is ${age} years old`);
}, 1000, 'John', 30);

// Store timer ID for clearing
const timerId = setTimeout(() => {
  console.log('This will execute');
}, 5000);

// Cancel before it runs
clearTimeout(timerId);
console.log('Timer cancelled');
```

**setInterval() - Execute Repeatedly:**

**Syntax:** `setInterval(callback, delay, arg1, arg2, ...)`

```javascript
// Basic usage - runs every second
let count = 0;

const intervalId = setInterval(() => {
  count++;
  console.log(`Count: ${count}`);
  
  // Stop after 5 executions
  if (count >= 5) {
    clearInterval(intervalId);
    console.log('Interval stopped');
  }
}, 1000);

// Output (every second):
// Count: 1
// Count: 2
// Count: 3
// Count: 4
// Count: 5
// Interval stopped

// With arguments
setInterval((message) => {
  console.log(message);
}, 2000, 'Hello every 2 seconds');
```

**Key Differences:**

| Feature | setTimeout | setInterval |
|---------|-----------|-------------|
| **Executions** | Once | Repeatedly |
| **When** | After delay | Every interval |
| **Stops** | Automatically after execution | Must call clearInterval() |
| **Use case** | Delayed action | Periodic updates |
| **Timing** | Delay + execution time | May drift if execution is slow |
| **Clear function** | clearTimeout() | clearInterval() |

**Timing Behavior:**

**setTimeout:**

```javascript
console.log('Start:', Date.now());

setTimeout(() => {
  console.log('Timeout:', Date.now());
  // Heavy operation
  for (let i = 0; i < 1000000000; i++) {}
  console.log('Done:', Date.now());
}, 1000);

// Timeline:
// 0ms: Start
// 1000ms: Timeout executes
// 3000ms: Done (after heavy operation)
// Next setTimeout would start fresh
```

**setInterval:**

```javascript
console.log('Start:', Date.now());

let count = 0;
const intervalId = setInterval(() => {
  console.log(`Execution ${++count}:`, Date.now());
  
  // Heavy operation (takes 500ms)
  const start = Date.now();
  while (Date.now() - start < 500) {}
  
  if (count >= 3) clearInterval(intervalId);
}, 1000);

// Timeline:
// 0ms: Start
// 1000ms: Execution 1 (takes 500ms until 1500ms)
// 2000ms: Execution 2 (takes 500ms until 2500ms)
// 3000ms: Execution 3 (takes 500ms until 3500ms)
// Note: Interval doesn't wait for execution to complete
```

**Practical Examples:**

**setTimeout Use Cases:**

**1. Delayed Action:**

```javascript
// Show notification, hide after 3 seconds
function showNotification(message) {
  const notification = document.createElement('div');
  notification.textContent = message;
  notification.className = 'notification';
  document.body.appendChild(notification);
  
  // Remove after 3 seconds
  setTimeout(() => {
    notification.remove();
  }, 3000);
}

showNotification('File saved successfully!');
```

**2. Debouncing:**

```javascript
let timeoutId;

function debounce(func, delay) {
  return function(...args) {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => {
      func.apply(this, args);
    }, delay);
  };
}

// Usage: Search after user stops typing
const searchInput = document.getElementById('search');
const debouncedSearch = debounce((query) => {
  console.log('Searching for:', query);
  performSearch(query);
}, 300);

searchInput.addEventListener('input', (e) => {
  debouncedSearch(e.target.value);
});
```

**3. Async Operation Timeout:**

```javascript
function fetchWithTimeout(url, timeout = 5000) {
  return new Promise((resolve, reject) => {
    const timer = setTimeout(() => {
      reject(new Error('Request timeout'));
    }, timeout);
    
    fetch(url)
      .then(response => {
        clearTimeout(timer);
        resolve(response);
      })
      .catch(error => {
        clearTimeout(timer);
        reject(error);
      });
  });
}

// Usage
try {
  const response = await fetchWithTimeout('/api/data', 3000);
  const data = await response.json();
} catch (error) {
  console.error('Request failed or timed out:', error);
}
```

**4. Recursive setTimeout (Alternative to setInterval):**

```javascript
// Better than setInterval - waits for completion
function recursiveTimeout() {
  console.log('Task executed:', new Date().toLocaleTimeString());
  
  // Schedule next execution after this one completes
  setTimeout(recursiveTimeout, 1000);
}

// Start the cycle
recursiveTimeout();

// With async operations
async function poll() {
  try {
    const data = await fetchData();
    processData(data);
  } catch (error) {
    console.error('Error:', error);
  }
  
  // Wait 5 seconds before next poll
  setTimeout(poll, 5000);
}

poll();
```

**setInterval Use Cases:**

**1. Clock/Timer:**

```javascript
function updateClock() {
  const now = new Date();
  const timeString = now.toLocaleTimeString();
  document.getElementById('clock').textContent = timeString;
}

// Update every second
const clockInterval = setInterval(updateClock, 1000);
updateClock(); // Initial call

// Stop clock
// clearInterval(clockInterval);
```

**2. Countdown Timer:**

```javascript
function startCountdown(seconds) {
  let remaining = seconds;
  
  // Update immediately
  updateDisplay(remaining);
  
  const intervalId = setInterval(() => {
    remaining--;
    updateDisplay(remaining);
    
    if (remaining <= 0) {
      clearInterval(intervalId);
      console.log('Countdown finished!');
      onCountdownComplete();
    }
  }, 1000);
  
  return intervalId; // Return so it can be cancelled
}

function updateDisplay(seconds) {
  const minutes = Math.floor(seconds / 60);
  const secs = seconds % 60;
  console.log(`${minutes}:${secs.toString().padStart(2, '0')}`);
}

// Start 5-minute countdown
const timerId = startCountdown(300);

// Cancel if needed
// clearInterval(timerId);
```

**3. Polling:**

```javascript
// Check for updates every 30 seconds
const pollingInterval = setInterval(async () => {
  try {
    const updates = await checkForUpdates();
    
    if (updates.length > 0) {
      console.log('New updates:', updates);
      displayUpdates(updates);
    }
  } catch (error) {
    console.error('Polling error:', error);
  }
}, 30000);

// Stop polling when user leaves
window.addEventListener('beforeunload', () => {
  clearInterval(pollingInterval);
});
```

**4. Animation:**

```javascript
let position = 0;
const element = document.getElementById('box');

const animationInterval = setInterval(() => {
  position += 5;
  element.style.left = position + 'px';
  
  // Stop at 500px
  if (position >= 500) {
    clearInterval(animationInterval);
  }
}, 16); // ~60fps

// Note: requestAnimationFrame is better for animations
```

**Problems with setInterval:**

**1. Time Drift:**

```javascript
// setInterval doesn't account for execution time
let executionCount = 0;
const startTime = Date.now();

const driftInterval = setInterval(() => {
  executionCount++;
  
  // Simulate slow operation (100ms)
  const opStart = Date.now();
  while (Date.now() - opStart < 100) {}
  
  const elapsed = Date.now() - startTime;
  const expected = executionCount * 1000;
  const drift = elapsed - expected;
  
  console.log(`Execution ${executionCount}: drift = ${drift}ms`);
  
  if (executionCount >= 5) clearInterval(driftInterval);
}, 1000);

// Output shows increasing drift:
// Execution 1: drift = 100ms
// Execution 2: drift = 200ms
// Execution 3: drift = 300ms
// etc.
```

**2. Execution Stacking:**

```javascript
// If operation takes longer than interval
setInterval(async () => {
  console.log('Starting long operation...');
  
  // Operation takes 3 seconds
  await new Promise(resolve => setTimeout(resolve, 3000));
  
  console.log('Operation complete');
}, 1000);

// With 1-second interval but 3-second operation:
// Multiple operations may queue up
// Can cause memory issues and performance problems
```

**3. Cannot Pause/Resume Easily:**

```javascript
// setInterval can't be paused
let intervalId = setInterval(() => {
  console.log('Running...');
}, 1000);

// To "pause", must clear and restart
function pauseInterval() {
  clearInterval(intervalId);
  intervalId = null;
}

function resumeInterval() {
  if (!intervalId) {
    intervalId = setInterval(() => {
      console.log('Running...');
    }, 1000);
  }
}
```

**Better Pattern: Recursive setTimeout:**

```javascript
// ✅ Better than setInterval - waits for completion
let isRunning = true;

async function task() {
  if (!isRunning) return;
  
  console.log('Task start:', Date.now());
  
  try {
    // Async operation
    await performOperation();
    console.log('Task complete:', Date.now());
  } catch (error) {
    console.error('Task error:', error);
  }
  
  // Schedule next execution after this completes
  setTimeout(task, 1000);
}

// Start
task();

// Stop (clean)
function stop() {
  isRunning = false;
}

// Advantages:
// - No drift (each execution starts fresh)
// - No stacking (waits for completion)
// - Easy to pause/resume
// - Better error handling
```

**Clearing Timers:**

```javascript
// setTimeout
const timeoutId = setTimeout(() => {
  console.log('This will not run');
}, 5000);

clearTimeout(timeoutId); // Cancel before execution

// setInterval
const intervalId = setInterval(() => {
  console.log('Repeating...');
}, 1000);

// Stop after 5 seconds
setTimeout(() => {
  clearInterval(intervalId);
  console.log('Interval stopped');
}, 5000);

// Clear multiple timers
const timers = [];
timers.push(setTimeout(() => {}, 1000));
timers.push(setTimeout(() => {}, 2000));
timers.push(setInterval(() => {}, 3000));

// Clear all
timers.forEach(id => {
  clearTimeout(id); // Works for both setTimeout and setInterval
  clearInterval(id);
});
```

**Minimum Delay:**

```javascript
// Browsers enforce minimum delay (~4ms in modern browsers)
setTimeout(() => {
  console.log('Not instant');
}, 0);

console.log('This runs first');

// Output:
// This runs first
// Not instant

// Actual delay is usually 4-10ms, not 0ms
const start = Date.now();
setTimeout(() => {
  console.log('Delay:', Date.now() - start, 'ms'); // ~4-10ms
}, 0);
```

**Background Tab Throttling:**

```javascript
// Browsers throttle timers in background tabs
// Minimum delay becomes ~1000ms

let count = 0;
const start = Date.now();

setInterval(() => {
  count++;
  const elapsed = Date.now() - start;
  console.log(`Count: ${count}, Elapsed: ${elapsed}ms`);
}, 100);

// Active tab: ~100ms intervals
// Background tab: ~1000ms intervals (throttled)
```

**Modern Alternatives:**

**requestAnimationFrame (for animations):**

```javascript
// Better than setInterval for animations
function animate() {
  // Update animation
  element.style.left = position + 'px';
  position += 5;
  
  if (position < 500) {
    requestAnimationFrame(animate); // Request next frame
  }
}

animate();

// Advantages:
// - Synced with display refresh (~60fps)
// - Paused when tab inactive
// - Better performance
```

**Web Workers (for background tasks):**

```javascript
// Heavy computation without blocking UI
const worker = new Worker('worker.js');

worker.postMessage({ task: 'process', data: largeDataSet });

worker.onmessage = (event) => {
  console.log('Result:', event.data);
};

// worker.js
setInterval(() => {
  // Heavy processing
  const result = processData();
  postMessage(result);
}, 1000);
```

**Common Patterns:**

**1. Self-Cancelling Timeout:**

```javascript
function autoHideNotification(element, delay = 3000) {
  const timerId = setTimeout(() => {
    element.remove();
  }, delay);
  
  // Cancel on user interaction
  element.addEventListener('click', () => {
    clearTimeout(timerId);
    element.remove();
  }, { once: true });
}
```

**2. Conditional Interval:**

```javascript
let shouldContinue = true;

const intervalId = setInterval(() => {
  if (!shouldContinue) {
    clearInterval(intervalId);
    return;
  }
  
  console.log('Task running...');
}, 1000);

// Stop from anywhere
function stopTask() {
  shouldContinue = false;
}
```

**3. Interval with Max Executions:**

```javascript
function setIntervalWithLimit(callback, delay, maxExecutions) {
  let count = 0;
  
  const intervalId = setInterval(() => {
    count++;
    callback(count);
    
    if (count >= maxExecutions) {
      clearInterval(intervalId);
    }
  }, delay);
  
  return intervalId;
}

// Run 10 times then stop
setIntervalWithLimit((count) => {
  console.log('Execution:', count);
}, 1000, 10);
```

**4. Throttling with setTimeout:**

```javascript
function throttle(func, delay) {
  let timeoutId;
  let lastRun = 0;
  
  return function(...args) {
    const now = Date.now();
    const timeSinceLastRun = now - lastRun;
    
    clearTimeout(timeoutId);
    
    if (timeSinceLastRun >= delay) {
      func.apply(this, args);
      lastRun = now;
    } else {
      timeoutId = setTimeout(() => {
        func.apply(this, args);
        lastRun = Date.now();
      }, delay - timeSinceLastRun);
    }
  };
}

// Usage: Limit scroll event handling
const throttledScroll = throttle(() => {
  console.log('Scroll position:', window.scrollY);
}, 200);

window.addEventListener('scroll', throttledScroll);
```

**Best Practices:**
- Use **setTimeout** for one-time delayed actions
- Use **setInterval** for periodic tasks (but consider recursive setTimeout)
- Always **clear timers** when no longer needed
- **Recursive setTimeout** preferred over setInterval for async operations
- Use **requestAnimationFrame** for animations
- Handle **cleanup** in component unmount/page unload
- Be aware of **background tab throttling**
- Store timer IDs to enable cancellation
- Consider **memory leaks** from uncancelled timers

**Key Takeaways:**
- **setTimeout**: Executes once after delay
- **setInterval**: Executes repeatedly at intervals
- Clear with **clearTimeout()** and **clearInterval()**
- setInterval can drift or stack executions
- Recursive setTimeout is often better than setInterval
- Minimum delay is ~4ms, not 0ms
- Background tabs are throttled to ~1000ms
- Always clean up timers to prevent memory leaks

56. What is callback hell?

**Answer:**
Callback hell (also called "pyramid of doom") is a situation where callbacks are nested within callbacks multiple levels deep, making code difficult to read, maintain, and debug. It typically occurs when handling multiple asynchronous operations that depend on each other.

**Example of Callback Hell:**

```javascript
// ❌ Callback hell - deeply nested, hard to read
getUserData(userId, (error, user) => {
  if (error) {
    console.error('Error getting user:', error);
    return;
  }
  
  getUserPosts(user.id, (error, posts) => {
    if (error) {
      console.error('Error getting posts:', error);
      return;
    }
    
    getPostComments(posts[0].id, (error, comments) => {
      if (error) {
        console.error('Error getting comments:', error);
        return;
      }
      
      getCommentAuthor(comments[0].authorId, (error, author) => {
        if (error) {
          console.error('Error getting author:', error);
          return;
        }
        
        getAuthorProfile(author.id, (error, profile) => {
          if (error) {
            console.error('Error getting profile:', error);
            return;
          }
          
          // Finally can use the data!
          console.log('Profile:', profile);
        });
      });
    });
  });
});

// The dreaded pyramid shape → → → → →
```

**Real-World Example:**

```javascript
// ❌ Callback hell with file operations
fs.readFile('config.json', 'utf8', (err, config) => {
  if (err) {
    console.error('Config error:', err);
    return;
  }
  
  const parsedConfig = JSON.parse(config);
  
  fs.readFile(parsedConfig.dataFile, 'utf8', (err, data) => {
    if (err) {
      console.error('Data error:', err);
      return;
    }
    
    const processedData = processData(data);
    
    fs.writeFile('output.txt', processedData, (err) => {
      if (err) {
        console.error('Write error:', err);
        return;
      }
      
      fs.readFile('template.html', 'utf8', (err, template) => {
        if (err) {
          console.error('Template error:', err);
          return;
        }
        
        const html = populateTemplate(template, processedData);
        
        fs.writeFile('index.html', html, (err) => {
          if (err) {
            console.error('HTML write error:', err);
            return;
          }
          
          console.log('All done!');
        });
      });
    });
  });
});
```

**Problems with Callback Hell:**

**1. Readability:**
- Code grows horizontally (pyramid shape)
- Difficult to follow the flow
- Hard to understand at a glance

**2. Maintainability:**
- Hard to add new steps
- Difficult to modify existing logic
- Error-prone when making changes

**3. Error Handling:**
- Repetitive error handling code
- Easy to miss error cases
- Difficult to propagate errors

**4. Debugging:**
- Stack traces are confusing
- Hard to set breakpoints effectively
- Difficult to test individual steps

**Solutions to Callback Hell:**

**Solution 1: Named Functions (Extract callbacks):**

```javascript
// ✅ Better - named functions
function handleUser(error, user) {
  if (error) {
    console.error('Error getting user:', error);
    return;
  }
  getUserPosts(user.id, handlePosts);
}

function handlePosts(error, posts) {
  if (error) {
    console.error('Error getting posts:', error);
    return;
  }
  getPostComments(posts[0].id, handleComments);
}

function handleComments(error, comments) {
  if (error) {
    console.error('Error getting comments:', error);
    return;
  }
  console.log('Comments:', comments);
}

// Cleaner call chain
getUserData(userId, handleUser);

// Pros: Flatter, more readable
// Cons: Still callback-based, harder to pass data between functions
```

**Solution 2: Promises:**

```javascript
// ✅ Much better - Promise chain
getUserData(userId)
  .then(user => getUserPosts(user.id))
  .then(posts => getPostComments(posts[0].id))
  .then(comments => getCommentAuthor(comments[0].authorId))
  .then(author => getAuthorProfile(author.id))
  .then(profile => {
    console.log('Profile:', profile);
  })
  .catch(error => {
    console.error('Error:', error); // Single error handler!
  });

// Pros: Flat, readable, single error handler
// Cons: Still chaining, can be verbose
```

**Solution 3: Async/Await (Best Solution):**

```javascript
// ✅ Best - async/await
async function loadUserProfile(userId) {
  try {
    const user = await getUserData(userId);
    const posts = await getUserPosts(user.id);
    const comments = await getPostComments(posts[0].id);
    const author = await getCommentAuthor(comments[0].authorId);
    const profile = await getAuthorProfile(author.id);
    
    console.log('Profile:', profile);
    return profile;
  } catch (error) {
    console.error('Error:', error);
    throw error;
  }
}

loadUserProfile(userId);

// Pros: Looks synchronous, very readable, clean error handling
// Cons: None significant
```

**Before and After Comparison:**

**Callback Hell:**
```javascript
// ❌ Callback hell
authenticate(username, password, (authError, token) => {
  if (authError) return handleError(authError);
  
  getUser(token, (userError, user) => {
    if (userError) return handleError(userError);
    
    getPermissions(user.id, (permError, permissions) => {
      if (permError) return handleError(permError);
      
      loadDashboard(user, permissions, (dashError, dashboard) => {
        if (dashError) return handleError(dashError);
        
        render(dashboard);
      });
    });
  });
});
```

**With Promises:**
```javascript
// ✅ Promises
authenticate(username, password)
  .then(token => getUser(token))
  .then(user => getPermissions(user.id)
    .then(permissions => ({ user, permissions }))
  )
  .then(({ user, permissions }) => loadDashboard(user, permissions))
  .then(dashboard => render(dashboard))
  .catch(error => handleError(error));
```

**With Async/Await:**
```javascript
// ✅ Async/await (cleanest)
async function initDashboard() {
  try {
    const token = await authenticate(username, password);
    const user = await getUser(token);
    const permissions = await getPermissions(user.id);
    const dashboard = await loadDashboard(user, permissions);
    
    render(dashboard);
  } catch (error) {
    handleError(error);
  }
}

initDashboard();
```

**Converting Callbacks to Promises:**

```javascript
// Callback-based function
function readFileCallback(path, callback) {
  fs.readFile(path, 'utf8', callback);
}

// Convert to Promise
function readFilePromise(path) {
  return new Promise((resolve, reject) => {
    fs.readFile(path, 'utf8', (error, data) => {
      if (error) {
        reject(error);
      } else {
        resolve(data);
      }
    });
  });
}

// Or use Node.js util.promisify
const { promisify } = require('util');
const readFilePromise = promisify(fs.readFile);

// Usage
try {
  const data = await readFilePromise('file.txt', 'utf8');
  console.log(data);
} catch (error) {
  console.error('Error:', error);
}
```

**Modularizing Async Operations:**

```javascript
// Create reusable async modules
const userService = {
  async getUserData(userId) {
    const response = await fetch(`/api/users/${userId}`);
    if (!response.ok) throw new Error('User not found');
    return response.json();
  },
  
  async getUserPosts(userId) {
    const response = await fetch(`/api/users/${userId}/posts`);
    if (!response.ok) throw new Error('Posts not found');
    return response.json();
  },
  
  async getUserProfile(userId) {
    const user = await this.getUserData(userId);
    const posts = await this.getUserPosts(userId);
    return { ...user, posts };
  }
};

// Clean usage
async function displayUserProfile(userId) {
  try {
    const profile = await userService.getUserProfile(userId);
    renderProfile(profile);
  } catch (error) {
    showError(error.message);
  }
}
```

**Error Handling Comparison:**

**Callback Hell:**
```javascript
// ❌ Repetitive error handling
step1((err, result1) => {
  if (err) return handleError(err);
  
  step2(result1, (err, result2) => {
    if (err) return handleError(err);
    
    step3(result2, (err, result3) => {
      if (err) return handleError(err);
      
      finish(result3);
    });
  });
});
```

**Async/Await:**
```javascript
// ✅ Single try/catch
async function process() {
  try {
    const result1 = await step1();
    const result2 = await step2(result1);
    const result3 = await step3(result2);
    finish(result3);
  } catch (err) {
    handleError(err); // Catches all errors!
  }
}
```

**Parallel Operations:**

**Callback Hell (Difficult):**
```javascript
// ❌ Complex parallel execution with callbacks
let user, posts, comments;
let completed = 0;
const total = 3;

getUser((err, userData) => {
  if (err) return handleError(err);
  user = userData;
  if (++completed === total) finish();
});

getPosts((err, postsData) => {
  if (err) return handleError(err);
  posts = postsData;
  if (++completed === total) finish();
});

getComments((err, commentsData) => {
  if (err) return handleError(err);
  comments = commentsData;
  if (++completed === total) finish();
});

function finish() {
  console.log({ user, posts, comments });
}
```

**Async/Await (Simple):**
```javascript
// ✅ Clean parallel execution
async function loadAllData() {
  try {
    const [user, posts, comments] = await Promise.all([
      getUser(),
      getPosts(),
      getComments()
    ]);
    
    console.log({ user, posts, comments });
  } catch (err) {
    handleError(err);
  }
}
```

**Signs of Callback Hell:**

1. **Deep nesting** (> 3 levels)
2. **Pyramid shape** (indentation keeps growing)
3. **Repetitive error handling**
4. **Difficult to follow** logic flow
5. **Hard to modify** or add steps
6. **Variable scope issues**
7. **Closure complications**

**Prevention Strategies:**

**1. Keep callbacks shallow:**
```javascript
// Don't nest immediately - extract functions
function processStep1(data) {
  // Process
  processStep2(result);
}

function processStep2(data) {
  // Process
  processStep3(result);
}
```

**2. Use modern patterns:**
```javascript
// Prefer Promises and async/await
async function modernApproach() {
  const data = await fetchData();
  const processed = await processData(data);
  return await saveData(processed);
}
```

**3. Use libraries:**
```javascript
// Libraries like 'async' can help (legacy code)
const async = require('async');

async.waterfall([
  (callback) => getUser(userId, callback),
  (user, callback) => getPosts(user.id, callback),
  (posts, callback) => getComments(posts[0].id, callback)
], (err, comments) => {
  if (err) return console.error(err);
  console.log(comments);
});
```

**4. Promisify callback APIs:**
```javascript
// Convert old APIs to Promises
const { promisify } = require('util');

const readFileAsync = promisify(fs.readFile);
const writeFileAsync = promisify(fs.writeFile);

// Now can use with async/await
async function processFiles() {
  const data = await readFileAsync('input.txt', 'utf8');
  await writeFileAsync('output.txt', data.toUpperCase());
}
```

**Best Practices:**
- **Avoid deep nesting** - extract functions or use async/await
- **Use Promises** instead of callbacks for new code
- **Prefer async/await** for ultimate readability
- **Handle errors** at appropriate levels
- **Promisify** legacy callback-based APIs
- **Use Promise.all()** for parallel operations
- **Modularize** async operations into services
- **Keep functions focused** and single-purpose

**Key Takeaways:**
- Callback hell is **deeply nested callbacks** (pyramid shape)
- Makes code **hard to read, maintain, and debug**
- Caused by **sequential async operations** with callbacks
- **Promises** flatten the structure
- **Async/await** provides cleanest solution
- Modern JavaScript has largely solved this problem
- Converting to Promises/async-await is straightforward
- Essential to understand for legacy code maintenance

57. How do you handle errors in Promises?

**Answer:**
Errors in Promises are handled using `.catch()` for Promise chains, `try/catch` blocks with async/await, or the second argument of `.then()`. Proper error handling prevents unhandled rejections and ensures graceful failure recovery.

**Basic Error Handling Methods:**

**1. Using .catch():**

```javascript
// .catch() handles rejection
fetchUser(userId)
  .then(user => {
    console.log('User:', user);
    return user;
  })
  .catch(error => {
    console.error('Error:', error.message);
    // Handle error (show message, retry, etc.)
  });

// .catch() catches errors from any previous .then()
Promise.resolve()
  .then(() => {
    throw new Error('Something failed');
  })
  .then(() => {
    console.log('This is skipped');
  })
  .catch(error => {
    console.error('Caught:', error.message); // 'Something failed'
  });
```

**2. Using try/catch with async/await:**

```javascript
// Clean error handling with async/await
async function getUser(userId) {
  try {
    const response = await fetch(`/api/users/${userId}`);
    
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }
    
    const user = await response.json();
    return user;
    
  } catch (error) {
    console.error('Failed to get user:', error.message);
    throw error; // Re-throw or return default
  }
}

// Usage
try {
  const user = await getUser(1);
  console.log('User:', user);
} catch (error) {
  showError('Could not load user');
}
```

**3. Using .then() with two arguments:**

```javascript
// Second argument handles rejection (less common)
fetchUser(userId).then(
  user => {
    // Success handler
    console.log('Success:', user);
  },
  error => {
    // Error handler
    console.error('Error:', error);
  }
);

// Note: This doesn't catch errors thrown in success handler
fetchUser(userId).then(
  user => {
    throw new Error('Processing error'); // NOT caught by error handler!
  },
  error => {
    console.error('Error:', error); // Won't catch the above
  }
);

// ✅ Better: Use .catch() instead
fetchUser(userId)
  .then(user => {
    throw new Error('Processing error');
  })
  .catch(error => {
    console.error('Error:', error); // Catches all errors
  });
```

**Error Propagation:**

```javascript
// Errors propagate down the chain
fetchUser(1)
  .then(user => {
    console.log('User:', user);
    return fetchPosts(user.id); // If this fails...
  })
  .then(posts => {
    console.log('Posts:', posts);
    return fetchComments(posts[0].id); // ...these are skipped
  })
  .then(comments => {
    console.log('Comments:', comments);
  })
  .catch(error => {
    // ...error caught here
    console.error('Error in chain:', error);
  });

// Visualized:
// fetchUser → then (success) → fetchPosts (fails) → 
// then (skipped) → then (skipped) → catch (handles error)
```

**Error Recovery:**

```javascript
// Recover from error and continue chain
fetchUser(1)
  .catch(error => {
    console.error('User fetch failed:', error);
    return { id: 1, name: 'Guest' }; // Provide default
  })
  .then(user => {
    // This runs even if fetchUser failed
    console.log('User:', user); // Might be default
    return user;
  });

// Multiple recovery points
fetchPrimaryServer()
  .catch(error => {
    console.log('Primary failed, trying backup...');
    return fetchBackupServer();
  })
  .catch(error => {
    console.log('Backup failed, using cache...');
    return fetchFromCache();
  })
  .catch(error => {
    console.error('All sources failed:', error);
    throw new Error('Data unavailable');
  })
  .then(data => {
    console.log('Got data from some source:', data);
  });
```

**Re-throwing Errors:**

```javascript
// Catch, log, and re-throw
async function fetchData() {
  try {
    const response = await fetch('/api/data');
    return await response.json();
  } catch (error) {
    console.error('Fetch failed:', error);
    throw error; // Pass error to caller
  }
}

// Caller handles it
async function loadData() {
  try {
    const data = await fetchData();
    displayData(data);
  } catch (error) {
    showError('Failed to load data');
  }
}

// Transform errors
async function getUserData(id) {
  try {
    const response = await fetch(`/api/users/${id}`);
    
    if (response.status === 404) {
      throw new Error('User not found');
    } else if (!response.ok) {
      throw new Error(`Server error: ${response.status}`);
    }
    
    return await response.json();
  } catch (error) {
    if (error.message === 'Failed to fetch') {
      throw new Error('Network error - check your connection');
    }
    throw error; // Re-throw other errors
  }
}
```

**Handling Multiple Promises:**

**Promise.all() - Fails fast:**

```javascript
// If ANY promise rejects, Promise.all rejects
try {
  const [user, posts, comments] = await Promise.all([
    fetchUser(1),
    fetchPosts(),
    fetchComments()
  ]);
  
  console.log('All loaded:', { user, posts, comments });
} catch (error) {
  // Catches first rejection
  console.error('At least one failed:', error);
}

// Handle errors individually before Promise.all
const [user, posts, comments] = await Promise.all([
  fetchUser(1).catch(err => ({ error: err.message })),
  fetchPosts().catch(err => ({ error: err.message })),
  fetchComments().catch(err => ({ error: err.message }))
]);

// Check for errors
if (user.error) console.error('User failed:', user.error);
if (posts.error) console.error('Posts failed:', posts.error);
if (comments.error) console.error('Comments failed:', comments.error);
```

**Promise.allSettled() - Never rejects:**

```javascript
// Get results for all, regardless of success/failure
const results = await Promise.allSettled([
  fetchUser(1),
  fetchPosts(),
  fetchComments()
]);

results.forEach((result, index) => {
  if (result.status === 'fulfilled') {
    console.log(`Success ${index}:`, result.value);
  } else {
    console.error(`Failed ${index}:`, result.reason);
  }
});

// Practical example
async function loadDashboard() {
  const results = await Promise.allSettled([
    fetchUser(userId),
    fetchPosts(userId),
    fetchNotifications(userId),
    fetchStats(userId)
  ]);
  
  const [userResult, postsResult, notifResult, statsResult] = results;
  
  return {
    user: userResult.status === 'fulfilled' ? userResult.value : null,
    posts: postsResult.status === 'fulfilled' ? postsResult.value : [],
    notifications: notifResult.status === 'fulfilled' ? notifResult.value : [],
    stats: statsResult.status === 'fulfilled' ? statsResult.value : {}
  };
}
```

**Promise.any() - Rejects only if all reject:**

```javascript
try {
  // Returns first successful result
  const data = await Promise.any([
    fetchFromServer1(),
    fetchFromServer2(),
    fetchFromServer3()
  ]);
  
  console.log('Got data from first successful server:', data);
} catch (error) {
  // Only rejects if ALL promises reject
  console.error('All servers failed:', error);
  // error is AggregateError with all rejection reasons
  console.error('Reasons:', error.errors);
}
```

**Custom Error Types:**

```javascript
// Create custom error classes
class NetworkError extends Error {
  constructor(message) {
    super(message);
    this.name = 'NetworkError';
  }
}

class ValidationError extends Error {
  constructor(message, field) {
    super(message);
    this.name = 'ValidationError';
    this.field = field;
  }
}

// Use in promises
async function submitForm(data) {
  // Validation
  if (!data.email) {
    throw new ValidationError('Email is required', 'email');
  }
  
  try {
    const response = await fetch('/api/submit', {
      method: 'POST',
      body: JSON.stringify(data)
    });
    
    if (!response.ok) {
      throw new NetworkError(`HTTP ${response.status}`);
    }
    
    return await response.json();
  } catch (error) {
    if (error.name === 'TypeError' && error.message === 'Failed to fetch') {
      throw new NetworkError('Network connection failed');
    }
    throw error;
  }
}

// Handle different error types
try {
  await submitForm(formData);
} catch (error) {
  if (error instanceof ValidationError) {
    showFieldError(error.field, error.message);
  } else if (error instanceof NetworkError) {
    showNetworkError(error.message);
  } else {
    showGenericError('An unexpected error occurred');
  }
}
```

**Practical Error Handling Patterns:**

**1. Centralized Error Handler:**

```javascript
class ApiClient {
  async request(url, options = {}) {
    try {
      const response = await fetch(url, options);
      
      if (!response.ok) {
        throw await this.handleHttpError(response);
      }
      
      return await response.json();
    } catch (error) {
      throw this.handleError(error);
    }
  }
  
  async handleHttpError(response) {
    const data = await response.json().catch(() => ({}));
    
    switch (response.status) {
      case 400:
        return new ValidationError(data.message || 'Bad request');
      case 401:
        return new Error('Unauthorized - please log in');
      case 403:
        return new Error('Forbidden - insufficient permissions');
      case 404:
        return new Error('Resource not found');
      case 500:
        return new Error('Server error - please try again later');
      default:
        return new Error(`HTTP error ${response.status}`);
    }
  }
  
  handleError(error) {
    if (error.message === 'Failed to fetch') {
      return new NetworkError('Network error - check your connection');
    }
    return error;
  }
}

// Usage
const api = new ApiClient();

try {
  const user = await api.request('/api/users/1');
  console.log('User:', user);
} catch (error) {
  showError(error.message);
}
```

**2. Retry with Error Handling:**

```javascript
async function fetchWithRetry(url, maxRetries = 3) {
  let lastError;
  
  for (let i = 0; i < maxRetries; i++) {
    try {
      const response = await fetch(url);
      
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}`);
      }
      
      return await response.json();
    } catch (error) {
      lastError = error;
      console.error(`Attempt ${i + 1} failed:`, error.message);
      
      // Don't retry on client errors (4xx)
      if (error.message.includes('HTTP 4')) {
        throw error;
      }
      
      // Last attempt - don't wait
      if (i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000;
        await new Promise(resolve => setTimeout(resolve, delay));
      }
    }
  }
  
  throw new Error(`Failed after ${maxRetries} attempts: ${lastError.message}`);
}

// Usage
try {
  const data = await fetchWithRetry('/api/data');
  console.log('Data:', data);
} catch (error) {
  console.error('All retries failed:', error);
}
```

**3. Timeout with Error:**

```javascript
function withTimeout(promise, ms) {
  return Promise.race([
    promise,
    new Promise((_, reject) =>
      setTimeout(() => reject(new Error('Operation timed out')), ms)
    )
  ]);
}

// Usage
try {
  const data = await withTimeout(
    fetch('/api/slow-endpoint').then(r => r.json()),
    5000
  );
  console.log('Data:', data);
} catch (error) {
  if (error.message === 'Operation timed out') {
    console.error('Request took too long');
  } else {
    console.error('Request failed:', error);
  }
}
```

**4. Error Boundary Pattern (React-inspired):**

```javascript
async function withErrorBoundary(asyncFn, fallback) {
  try {
    return await asyncFn();
  } catch (error) {
    console.error('Error caught by boundary:', error);
    
    if (typeof fallback === 'function') {
      return fallback(error);
    }
    return fallback;
  }
}

// Usage
const user = await withErrorBoundary(
  () => fetchUser(userId),
  { id: userId, name: 'Guest' } // Fallback value
);

const posts = await withErrorBoundary(
  () => fetchPosts(),
  (error) => {
    console.log('Using cached posts');
    return getCachedPosts();
  }
);
```

**5. Conditional Error Handling:**

```javascript
async function loadData(options = {}) {
  const { throwOnError = false, defaultValue = null } = options;
  
  try {
    const response = await fetch('/api/data');
    
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }
    
    return await response.json();
  } catch (error) {
    console.error('Load failed:', error);
    
    if (throwOnError) {
      throw error;
    }
    
    return defaultValue;
  }
}

// Throw errors
try {
  const data = await loadData({ throwOnError: true });
  processData(data);
} catch (error) {
  handleError(error);
}

// Return default
const data = await loadData({ defaultValue: [] });
displayData(data); // Shows empty array if failed
```

**Unhandled Rejection Handling:**

**Browser:**

```javascript
// Global unhandled rejection handler
window.addEventListener('unhandledrejection', event => {
  console.error('Unhandled rejection:', event.reason);
  console.error('Promise:', event.promise);
  
  // Prevent default error output
  event.preventDefault();
  
  // Log to error tracking service
  logErrorToService({
    type: 'unhandled_rejection',
    error: event.reason,
    stack: event.reason?.stack
  });
  
  // Show user-friendly message
  showError('An unexpected error occurred');
});

// Example of unhandled rejection
fetch('/api/data')
  .then(r => r.json())
  .then(data => {
    throw new Error('Processing error');
  });
// No .catch() - triggers unhandledrejection event
```

**Node.js:**

```javascript
// Global unhandled rejection handler
process.on('unhandledRejection', (reason, promise) => {
  console.error('Unhandled Rejection at:', promise);
  console.error('Reason:', reason);
  
  // Log error
  logger.error('Unhandled promise rejection', {
    reason,
    stack: reason?.stack
  });
  
  // In production, might want to exit
  // process.exit(1);
});

// Handled rejection handler
process.on('rejectionHandled', (promise) => {
  console.log('Rejection was handled:', promise);
});
```

**Finally Block:**

```javascript
// .finally() runs regardless of success/failure
let isLoading = true;

fetchData()
  .then(data => {
    console.log('Data:', data);
    displayData(data);
  })
  .catch(error => {
    console.error('Error:', error);
    showError(error.message);
  })
  .finally(() => {
    isLoading = false;
    hideLoadingSpinner();
    console.log('Cleanup complete');
  });

// With async/await
async function loadData() {
  showLoadingSpinner();
  
  try {
    const data = await fetchData();
    displayData(data);
  } catch (error) {
    showError(error.message);
  } finally {
    hideLoadingSpinner(); // Always runs
  }
}
```

**Error Logging:**

```javascript
// Comprehensive error logging
async function fetchWithLogging(url) {
  const startTime = Date.now();
  
  try {
    console.log(`[${new Date().toISOString()}] Fetching: ${url}`);
    
    const response = await fetch(url);
    
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`);
    }
    
    const data = await response.json();
    const duration = Date.now() - startTime;
    
    console.log(`[${new Date().toISOString()}] Success: ${url} (${duration}ms)`);
    
    return data;
  } catch (error) {
    const duration = Date.now() - startTime;
    
    console.error(`[${new Date().toISOString()}] Error: ${url} (${duration}ms)`);
    console.error('Error details:', {
      message: error.message,
      stack: error.stack,
      url,
      timestamp: new Date().toISOString()
    });
    
    // Send to monitoring service
    sendToMonitoring({
      type: 'fetch_error',
      url,
      error: error.message,
      duration
    });
    
    throw error;
  }
}
```

**Common Mistakes:**

**1. Forgetting .catch():**

```javascript
// ❌ Unhandled rejection
fetchData(); // If this fails, unhandled rejection warning

// ✅ Always add .catch()
fetchData().catch(error => console.error(error));

// Or use async/await with try/catch
try {
  await fetchData();
} catch (error) {
  console.error(error);
}
```

**2. Not returning in .then():**

```javascript
// ❌ Error not propagated
function getData() {
  fetchData()
    .then(data => data)
    .catch(error => {
      console.error(error); // Logs but doesn't propagate
    });
  
  return Promise.resolve(); // Returns wrong promise!
}

// ✅ Return the promise chain
function getData() {
  return fetchData()
    .then(data => data)
    .catch(error => {
      console.error(error);
      throw error; // Propagate error
    });
}
```

**3. Swallowing errors:**

```javascript
// ❌ Error disappears
fetchData()
  .catch(error => {
    console.log('Error occurred');
    // Error not re-thrown or handled
  })
  .then(() => {
    // This runs even if fetchData failed!
  });

// ✅ Re-throw or provide fallback
fetchData()
  .catch(error => {
    console.log('Error occurred');
    throw error; // Propagate
    // Or: return defaultValue; // Provide fallback
  })
  .then(data => {
    // Only runs if successful (or fallback provided)
  });
```

**4. Mixing error handling styles:**

```javascript
// ❌ Confusing mix of .catch() and try/catch
async function getData() {
  return fetch('/api/data')
    .then(r => r.json())
    .catch(error => console.error(error)); // Don't mix!
}

// ✅ Consistent style (prefer async/await)
async function getData() {
  try {
    const response = await fetch('/api/data');
    return await response.json();
  } catch (error) {
    console.error(error);
    throw error;
  }
}
```

**Best Practices:**
- Always handle errors with `.catch()` or `try/catch`
- Use `.finally()` for cleanup operations
- Re-throw errors if caller should handle them
- Provide meaningful error messages
- Use custom error classes for different error types
- Log errors with context (timestamp, URL, user, etc.)
- Set up global unhandled rejection handlers
- Test error paths thoroughly
- Consider fallback values for non-critical failures
- Don't swallow errors silently

**Key Takeaways:**
- Use `.catch()` for Promise chains, `try/catch` for async/await
- Errors propagate down Promise chains to nearest `.catch()`
- `.finally()` runs regardless of success/failure
- Always handle or propagate errors (no silent failures)
- Promise.allSettled() useful when all outcomes matter
- Custom error types enable specific error handling
- Global handlers catch unhandled rejections
- Proper error handling prevents crashes and improves UX

58. What is Promise chaining?

**Answer:**
Promise chaining is the practice of linking multiple asynchronous operations together using `.then()` calls, where each `.then()` returns a new Promise. This creates a sequence of operations where each step waits for the previous one to complete, passing data through the chain.

**Basic Concept:**

```javascript
// Basic chaining
fetchUser(1)
  .then(user => {
    console.log('User:', user.name);
    return user.id; // Return value for next .then()
  })
  .then(userId => {
    console.log('User ID:', userId);
    return fetchPosts(userId); // Return promise for next .then()
  })
  .then(posts => {
    console.log('Posts:', posts.length);
    return posts[0];
  })
  .then(firstPost => {
    console.log('First post:', firstPost.title);
  })
  .catch(error => {
    console.error('Error in chain:', error);
  });

// Each .then() returns a new Promise
const promise1 = fetchUser(1);
const promise2 = promise1.then(user => user.id);
const promise3 = promise2.then(id => fetchPosts(id));
// promise1, promise2, promise3 are all different Promise objects
```

**How Chaining Works:**

```javascript
// Sequential execution
Promise.resolve(1)
  .then(value => {
    console.log('Step 1:', value); // 1
    return value + 1; // Pass 2 to next .then()
  })
  .then(value => {
    console.log('Step 2:', value); // 2
    return value * 2; // Pass 4 to next .then()
  })
  .then(value => {
    console.log('Step 3:', value); // 4
    return value ** 2; // Pass 16 to next .then()
  })
  .then(value => {
    console.log('Final:', value); // 16
  });

// Output:
// Step 1: 1
// Step 2: 2
// Step 3: 4
// Final: 16
```

**Returning Values vs Promises:**

```javascript
// Returning regular value
fetchUser(1)
  .then(user => {
    return user.id; // Returns value, wrapped in Promise
  })
  .then(userId => {
    console.log('User ID:', userId); // Gets the value
  });

// Returning Promise (flattened automatically)
fetchUser(1)
  .then(user => {
    return fetchPosts(user.id); // Returns Promise
  })
  .then(posts => {
    console.log('Posts:', posts); // Gets resolved value, not Promise
  });

// NOT returning (undefined passed to next)
fetchUser(1)
  .then(user => {
    fetchPosts(user.id); // ❌ Forgot to return!
  })
  .then(posts => {
    console.log('Posts:', posts); // undefined
  });
```

**Practical Example: Sequential API Calls:**

```javascript
// Load user, then their posts, then first post's comments
function loadUserContent(userId) {
  return fetchUser(userId)
    .then(user => {
      console.log('Loaded user:', user.name);
      return fetchPosts(user.id);
    })
    .then(posts => {
      console.log('Loaded posts:', posts.length);
      if (posts.length === 0) {
        throw new Error('User has no posts');
      }
      return fetchComments(posts[0].id);
    })
    .then(comments => {
      console.log('Loaded comments:', comments.length);
      return comments;
    })
    .catch(error => {
      console.error('Failed to load content:', error);
      throw error;
    });
}

// Usage
loadUserContent(1)
  .then(comments => displayComments(comments))
  .catch(error => showError(error.message));
```

**Passing Multiple Values:**

```javascript
// Problem: Need data from multiple steps
fetchUser(1)
  .then(user => {
    return fetchPosts(user.id); // Lost user data!
  })
  .then(posts => {
    // Can't access user here
  });

// Solution 1: Return object
fetchUser(1)
  .then(user => {
    return fetchPosts(user.id)
      .then(posts => ({ user, posts })); // Combine data
  })
  .then(({ user, posts }) => {
    console.log(`${user.name} has ${posts.length} posts`);
  });

// Solution 2: Use Promise.all
fetchUser(1)
  .then(user => {
    return Promise.all([
      Promise.resolve(user), // Keep user
      fetchPosts(user.id)     // Fetch posts
    ]);
  })
  .then(([user, posts]) => {
    console.log(`${user.name} has ${posts.length} posts`);
  });

// Solution 3: Nested scope (avoid if possible)
fetchUser(1)
  .then(user => {
    return fetchPosts(user.id)
      .then(posts => {
        // user is accessible here via closure
        console.log(`${user.name} has ${posts.length} posts`);
        return { user, posts };
      });
  });

// Solution 4: async/await (clearest)
async function loadUserData(userId) {
  const user = await fetchUser(userId);
  const posts = await fetchPosts(user.id);
  console.log(`${user.name} has ${posts.length} posts`);
  return { user, posts };
}
```

**Chaining with Transformations:**

```javascript
// Transform data through the chain
fetch('/api/users')
  .then(response => response.json())
  .then(users => {
    // Filter active users
    return users.filter(user => user.active);
  })
  .then(activeUsers => {
    // Extract names
    return activeUsers.map(user => user.name);
  })
  .then(names => {
    // Sort alphabetically
    return names.sort();
  })
  .then(sortedNames => {
    console.log('Active users:', sortedNames);
  })
  .catch(error => {
    console.error('Error processing users:', error);
  });
```

**Conditional Chaining:**

```javascript
// Branch based on condition
fetchUser(userId)
  .then(user => {
    if (user.role === 'admin') {
      return fetchAdminData(user.id);
    } else {
      return fetchUserData(user.id);
    }
  })
  .then(data => {
    console.log('Data:', data);
  });

// Early return
fetchUser(userId)
  .then(user => {
    if (!user.active) {
      throw new Error('User is not active');
    }
    return user;
  })
  .then(user => {
    // Only runs if user is active
    return loadUserDashboard(user);
  })
  .catch(error => {
    console.error('Error:', error);
  });
```

**Error Handling in Chains:**

```javascript
// Error stops chain execution
Promise.resolve()
  .then(() => {
    console.log('Step 1');
    return 'result1';
  })
  .then(() => {
    console.log('Step 2');
    throw new Error('Step 2 failed');
  })
  .then(() => {
    console.log('Step 3'); // Skipped
  })
  .then(() => {
    console.log('Step 4'); // Skipped
  })
  .catch(error => {
    console.error('Error:', error.message); // Caught here
  });

// Output:
// Step 1
// Step 2
// Error: Step 2 failed

// Continue after error
fetchData()
  .then(processData)
  .catch(error => {
    console.error('Processing failed:', error);
    return defaultData; // Recover with default
  })
  .then(data => {
    // Continues with either processed data or default
    displayData(data);
  });
```

**Multiple Catch Blocks:**

```javascript
// Catch at different levels
fetchUser(userId)
  .then(user => fetchPosts(user.id))
  .catch(error => {
    console.error('Failed to get user or posts:', error);
    return []; // Return empty array to continue
  })
  .then(posts => fetchComments(posts[0]?.id))
  .catch(error => {
    console.error('Failed to get comments:', error);
    return []; // Return empty array
  })
  .then(comments => {
    // This always runs, even if earlier steps failed
    console.log('Comments:', comments);
  });
```

**Complex Chaining Example:**

```javascript
// Real-world example: User authentication flow
function authenticateAndLoadDashboard(username, password) {
  return authenticate(username, password)
    .then(token => {
      console.log('Authenticated, token:', token);
      localStorage.setItem('auth_token', token);
      return token;
    })
    .then(token => {
      return getUser(token);
    })
    .then(user => {
      console.log('User loaded:', user.name);
      return Promise.all([
        Promise.resolve(user),
        getUserPermissions(user.id),
        getUserPreferences(user.id)
      ]);
    })
    .then(([user, permissions, preferences]) => {
      console.log('All user data loaded');
      return {
        user,
        permissions,
        preferences
      };
    })
    .then(userData => {
      return loadDashboard(userData);
    })
    .then(dashboard => {
      console.log('Dashboard ready');
      renderDashboard(dashboard);
      return dashboard;
    })
    .catch(error => {
      console.error('Authentication/load failed:', error);
      
      if (error.message.includes('Invalid credentials')) {
        showError('Invalid username or password');
      } else if (error.message.includes('Network')) {
        showError('Network error - please try again');
      } else {
        showError('Failed to load dashboard');
      }
      
      throw error;
    })
    .finally(() => {
      hideLoadingSpinner();
    });
}
```

**Chaining vs Nesting:**

```javascript
// ❌ Nesting (callback hell with promises)
fetchUser(1)
  .then(user => {
    fetchPosts(user.id)
      .then(posts => {
        fetchComments(posts[0].id)
          .then(comments => {
            console.log('Comments:', comments);
          });
      });
  });

// ✅ Chaining (flat, readable)
fetchUser(1)
  .then(user => fetchPosts(user.id))
  .then(posts => fetchComments(posts[0].id))
  .then(comments => {
    console.log('Comments:', comments);
  })
  .catch(error => {
    console.error('Error:', error);
  });
```

**Parallel Operations in Chain:**

```javascript
// Combine sequential and parallel operations
fetchUser(userId)
  .then(user => {
    // Wait for user, then fetch multiple things in parallel
    return Promise.all([
      Promise.resolve(user),
      fetchPosts(user.id),
      fetchFriends(user.id),
      fetchNotifications(user.id)
    ]);
  })
  .then(([user, posts, friends, notifications]) => {
    console.log('All loaded:', { user, posts, friends, notifications });
    return { user, posts, friends, notifications };
  })
  .catch(error => {
    console.error('Error:', error);
  });
```

**Chaining with Finally:**

```javascript
// Finally always runs
let isLoading = true;

fetchData()
  .then(data => {
    console.log('Data:', data);
    return processData(data);
  })
  .then(processed => {
    console.log('Processed:', processed);
    return saveData(processed);
  })
  .catch(error => {
    console.error('Error:', error);
    throw error;
  })
  .finally(() => {
    isLoading = false;
    console.log('Cleanup complete');
    // Runs whether chain succeeded or failed
  });
```

**Chaining vs Async/Await:**

```javascript
// Promise chaining
function loadData() {
  return fetchUser(1)
    .then(user => fetchPosts(user.id))
    .then(posts => fetchComments(posts[0].id))
    .then(comments => comments)
    .catch(error => {
      console.error('Error:', error);
      throw error;
    });
}

// Async/await (equivalent, more readable)
async function loadData() {
  try {
    const user = await fetchUser(1);
    const posts = await fetchPosts(user.id);
    const comments = await fetchComments(posts[0].id);
    return comments;
  } catch (error) {
    console.error('Error:', error);
    throw error;
  }
}

// Both return promises
loadData()
  .then(comments => console.log('Comments:', comments))
  .catch(error => console.error('Failed:', error));
```

**Common Patterns:**

**1. Retry Chain:**

```javascript
function fetchWithRetry(url, retries = 3) {
  return fetch(url)
    .catch(error => {
      if (retries > 0) {
        console.log(`Retrying... (${retries} left)`);
        return fetchWithRetry(url, retries - 1);
      }
      throw error;
    });
}
```

**2. Waterfall (Sequential with Dependencies):**

```javascript
function waterfall(tasks) {
  let result;
  
  return tasks.reduce((chain, task) => {
    return chain.then(prevResult => {
      return task(prevResult);
    });
  }, Promise.resolve());
}

// Usage
waterfall([
  () => fetchUser(1),
  (user) => fetchPosts(user.id),
  (posts) => fetchComments(posts[0].id)
])
  .then(comments => console.log('Comments:', comments))
  .catch(error => console.error('Error:', error));
```

**3. Chain with Accumulation:**

```javascript
function processItems(items) {
  return items.reduce((chain, item) => {
    return chain.then(results => {
      return processItem(item)
        .then(result => [...results, result]);
    });
  }, Promise.resolve([]));
}

// Usage
processItems([1, 2, 3, 4, 5])
  .then(results => console.log('All results:', results));
```

**Best Practices:**
- Always **return** values or promises from `.then()`
- Keep chains **flat** (avoid nesting)
- Use `.catch()` at the end for error handling
- Use `.finally()` for cleanup
- Consider **async/await** for complex chains
- Pass data through the chain efficiently
- Handle errors at appropriate levels
- Keep each `.then()` focused on one task
- Use Promise.all() for parallel operations within chains

**Key Takeaways:**
- Promise chaining links async operations with `.then()`
- Each `.then()` returns a new Promise
- Return values are passed to next `.then()`
- Returning a Promise flattens it automatically
- Errors propagate to nearest `.catch()`
- Keeps async code flat and readable
- Alternative to callback hell
- Modern async/await is often cleaner for complex chains

59. What is the microtask queue?

**Answer:**
The microtask queue is a priority queue in JavaScript's event loop that holds callbacks from Promise resolutions, `queueMicrotask()`, `MutationObserver`, and `process.nextTick()` (Node.js). Microtasks have higher priority than macrotasks (setTimeout, setInterval) and all microtasks are processed before any macrotask executes.

**What Goes in the Microtask Queue:**

1. **Promise callbacks** (`.then()`, `.catch()`, `.finally()`)
2. **queueMicrotask()** callbacks
3. **MutationObserver** callbacks
4. **process.nextTick()** (Node.js - highest priority)

```javascript
// Promise callbacks go to microtask queue
Promise.resolve().then(() => {
  console.log('Microtask: Promise');
});

// queueMicrotask
queueMicrotask(() => {
  console.log('Microtask: queueMicrotask');
});

// MutationObserver
const observer = new MutationObserver(() => {
  console.log('Microtask: MutationObserver');
});

// Synchronous code runs first
console.log('Synchronous');

// Output order:
// Synchronous
// Microtask: Promise
// Microtask: queueMicrotask
// Microtask: MutationObserver
```

**Execution Order:**

```javascript
console.log('1: Script start');

setTimeout(() => {
  console.log('2: setTimeout (macrotask)');
}, 0);

Promise.resolve().then(() => {
  console.log('3: Promise (microtask)');
});

console.log('4: Script end');

// Output:
// 1: Script start
// 4: Script end
// 3: Promise (microtask)  ← Runs before setTimeout
// 2: setTimeout (macrotask)

// Explanation:
// 1. Synchronous code runs first (1, 4)
// 2. Call stack empties
// 3. ALL microtasks run (3)
// 4. Then ONE macrotask runs (2)
```

**Microtask Queue Processing:**

```
Event Loop Cycle:
┌─────────────────────────────────┐
│ 1. Execute synchronous code    │
├─────────────────────────────────┤
│ 2. Process ALL microtasks      │ ← Queue emptied completely
│    - Run one microtask          │
│    - Check queue again          │
│    - Repeat until empty         │
├─────────────────────────────────┤
│ 3. Render (if browser)          │
├─────────────────────────────────┤
│ 4. Process ONE macrotask        │ ← Only one per cycle
├─────────────────────────────────┤
│ 5. Back to step 2               │
└─────────────────────────────────┘
```

**Key Characteristic: Process ALL Microtasks:**

```javascript
// Microtasks can add more microtasks
Promise.resolve().then(() => {
  console.log('Microtask 1');
  
  // This new microtask runs before any macrotask
  Promise.resolve().then(() => {
    console.log('Microtask 2');
  });
});

setTimeout(() => {
  console.log('Macrotask');
}, 0);

console.log('Sync');

// Output:
// Sync
// Microtask 1
// Microtask 2  ← Added during microtask processing, still runs
// Macrotask    ← Only runs after ALL microtasks done
```

**Detailed Example:**

```javascript
console.log('A');

setTimeout(() => {
  console.log('B');
}, 0);

Promise.resolve()
  .then(() => {
    console.log('C');
  })
  .then(() => {
    console.log('D');
  });

setTimeout(() => {
  console.log('E');
}, 0);

Promise.resolve().then(() => {
  console.log('F');
});

console.log('G');

// Execution flow:
// 1. console.log('A') → prints 'A'
// 2. setTimeout(B) → queued as macrotask
// 3. Promise.then(C) → queued as microtask
// 4. setTimeout(E) → queued as macrotask
// 5. Promise.then(F) → queued as microtask
// 6. console.log('G') → prints 'G'
// 7. Stack empty → process microtasks
// 8. Run C → prints 'C', queues D as microtask
// 9. Run F → prints 'F'
// 10. Run D → prints 'D'
// 11. All microtasks done → process macrotask
// 12. Run B → prints 'B'
// 13. Process macrotasks again
// 14. Run E → prints 'E'

// Output: A → G → C → F → D → B → E
```

**Microtask Queue with Nested Promises:**

```javascript
Promise.resolve()
  .then(() => {
    console.log('Promise 1');
    
    Promise.resolve()
      .then(() => {
        console.log('Promise 1.1');
      })
      .then(() => {
        console.log('Promise 1.2');
      });
  })
  .then(() => {
    console.log('Promise 2');
  });

Promise.resolve().then(() => {
  console.log('Promise 3');
});

// Output:
// Promise 1
// Promise 3
// Promise 1.1
// Promise 2
// Promise 1.2

// Explanation:
// 1. Promise 1 microtask runs, queues Promise 1.1
// 2. Promise 3 microtask runs (was already queued)
// 3. Promise 1.1 runs, queues Promise 1.2
// 4. Promise 2 runs (from first chain's second .then())
// 5. Promise 1.2 runs
```

**Microtasks Can Starve Macrotasks:**

```javascript
// ⚠️ Infinite microtask loop - macrotasks never run!
function recursiveMicrotask() {
  Promise.resolve().then(() => {
    console.log('Microtask');
    recursiveMicrotask(); // Queues another microtask
  });
}

recursiveMicrotask();

setTimeout(() => {
  console.log('This will NEVER run!');
}, 0);

// Output:
// Microtask
// Microtask
// Microtask
// ... forever
// setTimeout never executes because microtasks keep getting added
```

**Practical Example: Animation:**

```javascript
// Using microtasks for immediate updates
function updateUI() {
  element.classList.add('loading');
  
  // Microtask - updates DOM immediately after current task
  Promise.resolve().then(() => {
    element.textContent = 'Processing...';
  });
  
  // Macrotask - runs later
  setTimeout(() => {
    element.textContent = 'Done';
    element.classList.remove('loading');
  }, 1000);
}

// Better alternative: queueMicrotask
function updateUIBetter() {
  element.classList.add('loading');
  
  queueMicrotask(() => {
    element.textContent = 'Processing...';
  });
  
  setTimeout(() => {
    element.textContent = 'Done';
    element.classList.remove('loading');
  }, 1000);
}
```

**queueMicrotask() API:**

```javascript
// Explicitly queue a microtask
queueMicrotask(() => {
  console.log('This is a microtask');
});

// Equivalent to:
Promise.resolve().then(() => {
  console.log('This is a microtask');
});

// Use case: Defer work until after current execution
function processData(data) {
  // Synchronous validation
  if (!data) {
    throw new Error('No data');
  }
  
  // Defer heavy processing
  queueMicrotask(() => {
    heavyProcessing(data);
  });
  
  return 'Processing started';
}
```

**MutationObserver (Microtask):**

```javascript
// MutationObserver uses microtask queue
const observer = new MutationObserver((mutations) => {
  console.log('DOM changed (microtask):', mutations.length);
});

observer.observe(document.body, { childList: true, subtree: true });

// Make multiple changes
document.body.appendChild(document.createElement('div'));
document.body.appendChild(document.createElement('div'));
document.body.appendChild(document.createElement('div'));

console.log('Sync code done');

// Output:
// Sync code done
// DOM changed (microtask): 3
// (Observer batches all changes into one microtask)
```

**Node.js: process.nextTick (Even Higher Priority):**

```javascript
// Node.js has special queue with higher priority than microtasks
console.log('1: Start');

setTimeout(() => {
  console.log('2: setTimeout');
}, 0);

Promise.resolve().then(() => {
  console.log('3: Promise');
});

process.nextTick(() => {
  console.log('4: nextTick');
});

console.log('5: End');

// Output (Node.js):
// 1: Start
// 5: End
// 4: nextTick      ← Highest priority
// 3: Promise       ← Microtask
// 2: setTimeout    ← Macrotask

// Priority: nextTick > microtask > macrotask
```

**Comparison with Macrotasks:**

```javascript
// Mix of microtasks and macrotasks
console.log('Start');

setTimeout(() => {
  console.log('setTimeout 1');
  
  Promise.resolve().then(() => {
    console.log('Promise in setTimeout 1');
  });
}, 0);

Promise.resolve().then(() => {
  console.log('Promise 1');
  
  setTimeout(() => {
    console.log('setTimeout in Promise 1');
  }, 0);
});

setTimeout(() => {
  console.log('setTimeout 2');
}, 0);

Promise.resolve().then(() => {
  console.log('Promise 2');
});

console.log('End');

// Output:
// Start
// End
// Promise 1
// Promise 2
// setTimeout 1
// Promise in setTimeout 1
// setTimeout 2
// setTimeout in Promise 1

// Explanation:
// 1. Sync: Start, End
// 2. All microtasks: Promise 1, Promise 2
// 3. First macrotask: setTimeout 1
// 4. Microtasks from setTimeout 1: Promise in setTimeout 1
// 5. Next macrotask: setTimeout 2
// 6. Last macrotask: setTimeout in Promise 1
```

**Use Cases:**

**1. Batching DOM Updates:**

```javascript
let updates = [];

function scheduleUpdate(update) {
  updates.push(update);
  
  // Batch all updates in one microtask
  if (updates.length === 1) {
    queueMicrotask(() => {
      console.log('Applying', updates.length, 'updates');
      updates.forEach(applyUpdate);
      updates = [];
    });
  }
}

scheduleUpdate('update 1');
scheduleUpdate('update 2');
scheduleUpdate('update 3');
console.log('Scheduled 3 updates');

// Output:
// Scheduled 3 updates
// Applying 3 updates (batched in one microtask)
```

**2. Promise-based Flow Control:**

```javascript
async function processItems(items) {
  for (const item of items) {
    // Each iteration processes in microtask
    await Promise.resolve(); // Yields to microtask queue
    processItem(item);
  }
}

// Keeps UI responsive by yielding between items
```

**3. Testing Async Code:**

```javascript
// Test that runs after all microtasks
async function test() {
  let result = null;
  
  Promise.resolve().then(() => {
    result = 'success';
  });
  
  // Wait for microtasks to complete
  await Promise.resolve();
  
  console.log('Result:', result); // 'success'
}
```

**Debugging Microtask Order:**

```javascript
function logWithContext(message, type) {
  console.log(`[${type}] ${message}`);
}

console.log('[Sync] Start');

setTimeout(() => {
  logWithContext('setTimeout 1', 'Macro');
}, 0);

Promise.resolve().then(() => {
  logWithContext('Promise 1', 'Micro');
  
  Promise.resolve().then(() => {
    logWithContext('Promise 1.1', 'Micro');
  });
});

queueMicrotask(() => {
  logWithContext('queueMicrotask', 'Micro');
});

Promise.resolve().then(() => {
  logWithContext('Promise 2', 'Micro');
});

setTimeout(() => {
  logWithContext('setTimeout 2', 'Macro');
}, 0);

console.log('[Sync] End');

// Output:
// [Sync] Start
// [Sync] End
// [Micro] Promise 1
// [Micro] queueMicrotask
// [Micro] Promise 2
// [Micro] Promise 1.1
// [Macro] setTimeout 1
// [Macro] setTimeout 2
```

**Best Practices:**
- Understand **microtasks run before macrotasks**
- Use **queueMicrotask()** for explicit microtask scheduling
- **Avoid infinite microtask loops** (can freeze the browser)
- Promises automatically use microtask queue
- Microtasks are for **high-priority, immediate** work
- Use macrotasks (setTimeout) for **deferred, lower-priority** work
- In Node.js, **process.nextTick** has even higher priority
- Batch operations in microtasks for performance

**Key Takeaways:**
- Microtask queue holds Promise callbacks and queueMicrotask
- **Higher priority** than macrotask queue (setTimeout, setInterval)
- **ALL** microtasks run before **ANY** macrotask
- Microtasks can add more microtasks (processed in same cycle)
- Can starve macrotasks if infinite loop created
- Essential for understanding Promise execution order
- Part of JavaScript event loop specification
- Use queueMicrotask() for explicit scheduling

60. What is the difference between microtask and macrotask?

**Answer:**
Microtasks (Promise callbacks, queueMicrotask) have higher priority and all microtasks execute before any macrotask. Macrotasks (setTimeout, setInterval, I/O) execute one at a time per event loop cycle. Microtasks are for immediate, high-priority work; macrotasks are for deferred, scheduled work.

**Key Differences:**

| Feature | Microtask | Macrotask |
|---------|-----------|-----------|
| **Priority** | Higher | Lower |
| **Execution** | ALL per cycle | ONE per cycle |
| **Examples** | Promise.then, queueMicrotask | setTimeout, setInterval |
| **When** | After current task, before render | After microtasks, after render |
| **Can add more** | Yes, all processed same cycle | Yes, processed next cycle |
| **Use for** | Immediate, high-priority work | Deferred, scheduled work |
| **Queue name** | Microtask queue | Callback/Task queue |

**Microtasks:**

Sources:
- `Promise.then()`, `.catch()`, `.finally()`
- `queueMicrotask()`
- `MutationObserver`
- `process.nextTick()` (Node.js)

```javascript
// Microtask examples
Promise.resolve().then(() => {
  console.log('Microtask: Promise');
});

queueMicrotask(() => {
  console.log('Microtask: queueMicrotask');
});

const observer = new MutationObserver(() => {
  console.log('Microtask: MutationObserver');
});
```

**Macrotasks:**

Sources:
- `setTimeout()`
- `setInterval()`
- `setImmediate()` (Node.js)
- `requestAnimationFrame()` (browser)
- I/O operations
- UI rendering events

```javascript
// Macrotask examples
setTimeout(() => {
  console.log('Macrotask: setTimeout');
}, 0);

setInterval(() => {
  console.log('Macrotask: setInterval');
}, 1000);

// Node.js
setImmediate(() => {
  console.log('Macrotask: setImmediate');
});
```

**Execution Order Demonstration:**

```javascript
console.log('1: Sync start');

// Macrotasks
setTimeout(() => {
  console.log('2: setTimeout 1');
}, 0);

setTimeout(() => {
  console.log('3: setTimeout 2');
}, 0);

// Microtasks
Promise.resolve().then(() => {
  console.log('4: Promise 1');
});

Promise.resolve().then(() => {
  console.log('5: Promise 2');
});

console.log('6: Sync end');

// Output:
// 1: Sync start       (synchronous)
// 6: Sync end         (synchronous)
// 4: Promise 1        (microtask - ALL run)
// 5: Promise 2        (microtask - ALL run)
// 2: setTimeout 1     (macrotask - ONE runs)
// 3: setTimeout 2     (macrotask - ONE runs)

// Event loop cycles:
// Cycle 1: Sync code → All microtasks → First macrotask
// Cycle 2: Second macrotask
```

**Processing ALL Microtasks vs ONE Macrotask:**

```javascript
// Microtasks keep adding more - ALL processed
Promise.resolve().then(() => {
  console.log('Micro 1');
  
  Promise.resolve().then(() => {
    console.log('Micro 1.1');
  });
});

Promise.resolve().then(() => {
  console.log('Micro 2');
});

setTimeout(() => {
  console.log('Macro 1');
}, 0);

setTimeout(() => {
  console.log('Macro 2');
}, 0);

// Output:
// Micro 1
// Micro 2
// Micro 1.1    ← Added during microtask, still runs
// Macro 1      ← Only ONE macrotask per cycle
// Macro 2      ← Next cycle

// Microtasks: 1 → 2 → 1.1 (all in one cycle)
// Macrotasks: 1 (cycle 1) → 2 (cycle 2)
```

**Complex Example:**

```javascript
console.log('Start');

setTimeout(() => {
  console.log('Macro 1');
  
  Promise.resolve().then(() => {
    console.log('Micro in Macro 1');
  });
  
  setTimeout(() => {
    console.log('Macro 1.1');
  }, 0);
}, 0);

Promise.resolve().then(() => {
  console.log('Micro 1');
  
  setTimeout(() => {
    console.log('Macro in Micro 1');
  }, 0);
  
  Promise.resolve().then(() => {
    console.log('Micro 1.1');
  });
});

setTimeout(() => {
  console.log('Macro 2');
}, 0);

Promise.resolve().then(() => {
  console.log('Micro 2');
});

console.log('End');

// Output:
// Start                  (sync)
// End                    (sync)
// Micro 1                (all microtasks)
// Micro 2                (all microtasks)
// Micro 1.1              (all microtasks)
// Macro 1                (one macrotask)
// Micro in Macro 1       (microtasks after macrotask)
// Macro 2                (next macrotask)
// Macro in Micro 1       (next macrotask)
// Macro 1.1              (next macrotask)

// Event loop cycles:
// 1. Sync: Start, End
// 2. Microtasks: Micro 1, Micro 2, Micro 1.1
// 3. Macrotask: Macro 1
// 4. Microtasks: Micro in Macro 1
// 5. Macrotask: Macro 2
// 6. Macrotask: Macro in Micro 1
// 7. Macrotask: Macro 1.1
```

**Visual Representation:**

```
Event Loop Iteration:

┌─────────────────────────────────────┐
│  Execute synchronous code           │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│  Process ALL microtasks             │
│  ┌───────────────────────────────┐  │
│  │ Run microtask 1               │  │
│  │ If it adds microtasks, add    │  │
│  │ to queue and process          │  │
│  └───────────────────────────────┘  │
│  ┌───────────────────────────────┐  │
│  │ Run microtask 2               │  │
│  └───────────────────────────────┘  │
│  ... until microtask queue empty    │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│  Render UI (browser only)           │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│  Process ONE macrotask              │
│  ┌───────────────────────────────┐  │
│  │ Run oldest macrotask          │  │
│  └───────────────────────────────┘  │
└─────────────────────────────────────┘
              ↓
        (Loop back to microtasks)
```

**Practical Implications:**

**1. Promise vs setTimeout Timing:**

```javascript
console.time('Promise');
Promise.resolve().then(() => {
  console.timeEnd('Promise');
});

console.time('setTimeout');
setTimeout(() => {
  console.timeEnd('setTimeout');
}, 0);

// Output:
// Promise: 0.1ms     (microtask - runs immediately)
// setTimeout: 4.5ms  (macrotask - minimum delay ~4ms)
```

**2. UI Responsiveness:**

```javascript
// ❌ Blocks UI - long synchronous operation
function processHeavyData(data) {
  for (let item of data) {
    heavyComputation(item); // Blocks!
  }
}

// ✅ Yields to UI - using macrotasks
async function processHeavyDataAsync(data) {
  for (let item of data) {
    heavyComputation(item);
    
    // Yield to event loop (let UI update)
    await new Promise(resolve => setTimeout(resolve, 0));
  }
}

// ⚠️ Using microtasks - still blocks!
async function processWithMicrotask(data) {
  for (let item of data) {
    heavyComputation(item);
    
    // Microtasks don't allow UI render
    await Promise.resolve();
  }
}
```

**3. Animation Timing:**

```javascript
// Microtask - runs before next paint
Promise.resolve().then(() => {
  element.style.opacity = '0';
  // Change applied before screen updates
});

// Macrotask - runs after paint
setTimeout(() => {
  element.style.opacity = '0';
  // Might see flash of old opacity
}, 0);

// requestAnimationFrame - syncs with paint (best for animations)
requestAnimationFrame(() => {
  element.style.opacity = '0';
  // Optimized for smooth animations
});
```

**4. Event Handler Behavior:**

```javascript
button.addEventListener('click', () => {
  console.log('Click handler start');
  
  Promise.resolve().then(() => {
    console.log('Microtask in click');
  });
  
  setTimeout(() => {
    console.log('Macrotask in click');
  }, 0);
  
  console.log('Click handler end');
});

// When clicked:
// Click handler start
// Click handler end
// Microtask in click    ← Before next macrotask
// Macrotask in click    ← Next event loop cycle
```

**Node.js Differences:**

```javascript
// Node.js has additional phases
console.log('Start');

setTimeout(() => {
  console.log('setTimeout');
}, 0);

setImmediate(() => {
  console.log('setImmediate');
});

Promise.resolve().then(() => {
  console.log('Promise');
});

process.nextTick(() => {
  console.log('nextTick');
});

console.log('End');

// Output (Node.js):
// Start
// End
// nextTick         (highest priority)
// Promise          (microtask)
// setTimeout       (timers phase)
// setImmediate     (check phase)

// Node.js event loop phases:
// 1. timers (setTimeout, setInterval)
// 2. pending callbacks
// 3. idle, prepare
// 4. poll (I/O)
// 5. check (setImmediate)
// 6. close callbacks
// Between each phase: process.nextTick → microtasks
```

**Starvation Example:**

```javascript
// Microtasks can starve macrotasks
let microCount = 0;

function addMicrotask() {
  microCount++;
  
  if (microCount < 5) {
    Promise.resolve().then(() => {
      console.log('Microtask', microCount);
      addMicrotask(); // Keeps adding microtasks
    });
  }
}

addMicrotask();

setTimeout(() => {
  console.log('Macrotask (runs after all microtasks)');
}, 0);

// Output:
// Microtask 1
// Microtask 2
// Microtask 3
// Microtask 4
// Macrotask (runs after all microtasks)

// If microCount limit removed, macrotask never runs!
```

**When to Use Each:**

**Use Microtasks (Promise, queueMicrotask) when:**
- Need immediate execution after current operation
- High-priority work that must complete before rendering
- Promise-based async operations
- Batching operations in same tick
- Need to run before any timer callbacks

**Use Macrotasks (setTimeout, setInterval) when:**
- Need to defer work to next event loop cycle
- Breaking up long-running operations
- Scheduling work for specific time
- Yielding to UI for responsiveness
- Lower-priority background work

```javascript
// Microtask: Immediate priority
function updateCriticalState() {
  Promise.resolve().then(() => {
    // Runs before any setTimeout, immediately after current task
    criticalStateUpdate();
  });
}

// Macrotask: Deferred work
function scheduleDeferredWork() {
  setTimeout(() => {
    // Runs in next cycle, after microtasks and render
    backgroundProcessing();
  }, 0);
}
```

**Testing Understanding:**

```javascript
// What's the output?
console.log('A');

setTimeout(() => console.log('B'), 0);
setTimeout(() => console.log('C'), 0);

Promise.resolve().then(() => console.log('D'));
Promise.resolve().then(() => console.log('E'));

console.log('F');

// Answer: A → F → D → E → B → C
// Explanation:
// Sync: A, F
// All microtasks: D, E
// First macrotask: B
// Second macrotask: C
```

**Best Practices:**
- Use **microtasks for immediate, critical** operations
- Use **macrotasks for deferred, scheduled** work
- Understand **all microtasks run** before any macrotask
- **Avoid infinite microtask loops** (can freeze browser)
- Use setTimeout(fn, 0) to **yield to UI**
- Promises automatically use microtask queue
- In Node.js, prefer **setImmediate over setTimeout(0)** for macrotasks
- Use **queueMicrotask** for explicit microtask scheduling

**Key Takeaways:**
- **Microtasks**: Higher priority, ALL processed per cycle
- **Macrotasks**: Lower priority, ONE processed per cycle
- Microtasks: Promise callbacks, queueMicrotask, MutationObserver
- Macrotasks: setTimeout, setInterval, I/O, UI events
- Order: sync code → all microtasks → render → one macrotask
- Microtasks can add more microtasks (processed same cycle)
- Understanding crucial for debugging async timing issues
- Core concept of JavaScript event loop
