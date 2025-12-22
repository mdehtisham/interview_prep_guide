# Top Node.js Interview Questions

## Core Concepts & Internal Architecture
1. What is Node.js and, significantly, does it run on a single thread?
**Answer:**
Node.js is a runtime environment for executing JavaScript on the server-side, built on Google Chrome's V8 JavaScript engine.
-   **Thread Model:** Node.js runs the **V8 Javascript Engine on a single thread**. This means your written code is executed one line at a time.
-   **HOWEVER:** The Node.js *Runtime* (specifically the `libuv` library) is not single-threaded. It maintains a pool of threads (the Worker Pool) written in C++ to handle expensive tasks like File I/O, DNS lookups, and specialized crypto operations.
-   *Interview Key:* "Node.js is single-threaded in its Event Loop, but multi-threaded in its background processing."

2. Explain the concept of the Event Loop in Node.js. What are its phases?
**Answer:**
The Event Loop is the mechanism that allows Node.js to perform non-blocking I/O operations despite being single-threaded. It offloads operations to the system kernel whenever possible.
When Node.js starts, it initializes the event loop, processes the provided input script, and then enters a loop with the following phases:
1.  **Timers:** Executes callbacks scheduled by `setTimeout()` and `setInterval()`.
2.  **Pending Callbacks:** Executes I/O callbacks deferred to the next loop iteration (system errors mostly).
3.  **Idle, Prepare:** Internal use only.
4.  **Poll (Critical Phase):** Retrieves new I/O events; executes I/O related callbacks (almost all code happens here). It can block here if the queue is empty.
5.  **Check:** Executes `setImmediate()` callbacks.
6.  **Close Callbacks:** Executes close events (e.g., `socket.on('close', ...)`).

3. What is libuv and what is its role in Node.js?
**Answer:**
`libuv` is a multi-platform support library written in C primarily for asynchronous I/O.
-   **Role:** It enforces the asynchronous, event-driven style.
-   It provides the **Event Loop**.
-   It maintains the **Thread Pool** (default size 4, configurable via `UV_THREADPOOL_SIZE`).
-   It abstracts OS differences (Windows IOCP vs Linux epoll), allowing you to write valid Node.js code on any OS.

4. Explain the difference between `process.nextTick()` and `setImmediate()`.
**Answer:**
-   **`process.nextTick()` (Not actually part of the Event Loop):** It fires **immediately** after the current operation completes, *before* the Event Loop continues to the next phase.
    -   *Risk:* Recursively calling `nextTick` will starve I/O and freeze the server.
-   **`setImmediate()`:** It fires in the **Check** phase of the Event Loop (after the Poll phase).
    -   *Use Case:* It is the "polite" way to queue code to run "as soon as possible but after IO".

5. What is the difference between blocking and non-blocking I/O?
**Answer:**
-   **Blocking (Synchronous):** The execution of additional JavaScript in the Node.js process must wait until the non-JavaScript operation completes.
    -   Example: `fs.readFileSync()`. The server halts. No other user can be served during this time.
-   **Non-Blocking (Asynchronous):** The system call returns immediately (usually with no data), and the Node.js process can continue executing other code. When the data is ready, a callback is added to the Event Loop queue.
    -   Example: `fs.readFile()`. The server initiates the read and moves on. 1000 other users can be served while the disk is spinning.
6. How does Node.js handle concurrency if it is single-threaded?
**Answer:**
Node.js uses an **Event-Driven, Non-Blocking I/O model**.
-   **Delegation:** When a request involves I/O (database, file, network), the main thread *delegates* this task to the native system kernel (via libuv) and immediately moves to the next request. It does **not** wait.
-   **Callbacks/Promises:** It registers a callback function to be executed when the task finishes.
-   **Conclusion:** It handles high concurrency for *I/O bound* tasks because the single thread only acts as a dispatcher (like a waiter taking orders), while the actual "cooking" happens in the kernel or thread pool.

7. What are the "Worker Threads" in Node.js and when should you use them?
**Answer:**
Typically, Node.js is poor at **CPU-intensive** tasks (e.g., image resizing, video compression, complex math) because these operations block the single main thread, freezing the server for all clients.
-   **Worker Threads (`worker_threads`):** This module allows running JavaScript in parallel threads.
-   **How it works:** Each worker has its own V8 instance and Event Loop but shares memory via `SharedArrayBuffer`.
-   **Use Case:** ONLY for heavy CPU tasks. Do not use them for I/O (Node's default non-blocking I/O is already more efficient for that).

8. Explain the Reactor Pattern in Node.js.
**Answer:**
The Reactor Pattern is the theoretical design pattern behind Node.js's asynchronous nature.
1.  **Resources** (I/O operations) are requested by the application.
2.  The **Event Demultiplexer** (libuv) accepts these requests and delegates them to synchronous Event Handlers.
3.  The Demultiplexer waits for events (completion of I/O) to occur.
4.  When an event occurs, it is queued in the **Event Queue**.
5.  The **Event Loop** iterates and executes the callbacks associated with those events.

9. What is the difference between `module.exports` and `exports`?
**Answer:**
-   **`module.exports`**: This is the *actual* object that is returned as the result of a `require()` call.
-   **`exports`**: This is just a convenience variable (a reference/pointer) that initially points to `module.exports`.
-   **The Trap:**
    -   `exports.func = ...` works (modifies the object).
    -   `exports = { ... }` **FAILS**. You just broke the reference. `module.exports` remains empty, and your module exports nothing.
    -   *Best Practice:* Always stick to explicit `module.exports` assignment if you are exporting a single class or function.

10. What is the difference between CommonJS (`require`) and ES Modules (`import`)?
**Answer:**
-   **CommonJS (CJS):**
    -   Syntax: `const fs = require('fs');`
    -   **Synchronous:** Modules are loaded instantly (blocking).
    -   Dynamic: You can `require()` inside `if` statements.
    -   Default in legacy Node.js.
-   **ES Modules (ESM):**
    -   Syntax: `import fs from 'fs';`
    -   **Asynchronous:** Designed for static analysis and tree-shaking (removing unused code).
    -   Static: Imports must be at the top level.
    -   Requires `"type": "module"` in `package.json` or `.mjs` extension.
    -   Modern Standard.

## Asynchronous Programming
11. What is "Callback Hell" and how do you avoid it?
**Answer:**
Callback Hell (or "Pyramid of Doom") happens when you nest multiple asynchronous callbacks inside each other, causing the code to grow horizontally and becoming unreadable/unmaintainable.
**Solutions:**
1.  **Named Functions:** Extract named functions to flatten layout.
2.  **Promises:** Use `.then().then()` chaining.
3.  **Async/Await (Best):** Makes asynchronous code look like synchronous code.
4.  **Libraries:** `async.js` (waterfall series) - mostly obsolete now.

12. Explain Promises and their states.
**Answer:**
A Promise is an object representing the eventual completion (or failure) of an asynchronous operation.
It has 3 states:
1.  **Pending:** Initial state, neither fulfilled nor rejected.
2.  **Fulfilled (Resolved):** Operation completed successfully.
3.  **Rejected:** Operation failed.
*Note:* A promise is "Settled" if it is not pending (either resolved or rejected). It is immutable once settled.

13. What is the difference between `Promise.all()`, `Promise.allSettled()`, `Promise.race()`, and `Promise.any()`?
**Answer:**
-   **`Promise.all([p1, p2])`**: Waits for **ALL** to resolve. If **ONE** fails, the whole thing fails immediately (rejects).
-   **`Promise.allSettled([p1, p2])`**: Waits for **ALL** to finish, regardless of success or failure. Returns an array of objects `{ status: 'fulfilled' | 'rejected', value, reason }`. (Added in ES2020).
-   **`Promise.race([p1, p2])`**: Returns the result of the **FIRST** promise to settle (either resolve or reject).
-   **`Promise.any([p1, p2])`**: Returns the result of the **FIRST** promise to **fulfill**. Only rejects if *all* promises reject. (Added in ES2021).

14. What happens if you don't handle a rejected promise in Node.js?
**Answer:**
Historically, it would just log a warning `UnhandledPromiseRejectionWarning`.
However, in modern Node.js versions (v15+), an unhandled rejection **crashes the process** with exit code 1.
**Fix:** Always use `.catch()` or `try-catch` blocks. You can also listen globally:
`process.on('unhandledRejection', (reason, promise) => { ... })` (last resort).

15. Explain `async/await`. How is it different from Promises under the hood?
**Answer:**
`async/await` is syntactic sugar built **on top of Promises**. It uses **Generators** (`yield`) internally to pause execution.
-   **Difference:** There is no difference in functionality. It's purely about readability.
    -   Promises use `.then()` chains.
    -   Await pauses the execution flow of the function until the promise resolves.
    -   Await throws errors instead of rejecting, so you use standard `try/catch` blocks.
-   **Key:** An `async` function *always* returns a Promise, even if you return a simple number like `return 1`.
16. Can you pause the execution of a Node.js script?
**Answer:**
**Yes and No.**
-   **No:** You cannot "pause" (sleep) the thread in a blocking way (like `sleep(1000)` in C/Python) without freezing the entire event loop (bad!).
-   **Yes:** You can delay execution asynchronously causing a "pause" effect using `await`:
    ```javascript
    await new Promise(resolve => setTimeout(resolve, 1000));
    ```

17. What is an EventEmitter? How does it work?
**Answer:**
`EventEmitter` is the core class in the `events` module that drives Node's Event-Driven Architecture. Many core APIs (Streams, HTTP Request/Response) inherit from it.
-   **Mechanism:** It maintains a list of listeners (functions).
    -   `.on('event', listener)`: Adds a function to the array.
    -   `.emit('event', data)`: Synchronously calls all functions registered for that event.
-   **Memory Leak Warning:** If you bind events but never remove them (`.off`), the listener array grows indefinitely.

## Streams & Buffers
18. What are Buffers in Node.js? Why are they needed?
**Answer:**
A Buffer is a chunk of raw memory allocated outside the V8 heap.
-   **Why:** JavaScript historically didn't handle binary data well (it only had Strings).
-   **Usage:** Buffers are used to represent fixed-length sequences of bytes (like raw file contents, image data, TCP packets).
-   **Properties:** Buffers are generally fixed size. You can convert them to string: `buf.toString('utf8')`.

19. What are Streams in Node.js? Name the four types of streams.
**Answer:**
Streams are collections of data provided over time (chunks) rather than all at once. They are instances of typical `EventEmitter`.
**Four Types:**
1.  **Readable:** Data you can read (e.g., `fs.createReadStream`).
2.  **Writable:** Data you can write to (e.g., `fs.createWriteStream`, `res` in HTTP).
3.  **Duplex:** Both Readable and Writable (e.g., TCP Socket).
4.  **Transform:** Duplex stream where output is computed from input (e.g., zlib compression, crypto Streams).

20. Explain the concept of "Backpressure" in streams.
**Answer:**
Backpressure occurs when the **Readable Stream** (producer) emits data faster than the **Writable Stream** (consumer) can process/write it.
-   **Problem:** If you keep pushing, memory fills up with buffered chunks, causing a crash.
-   **Solution:** Node.js streams accept a return value. If `write(chunk)` returns `false`, the producer must **pause** reading.
-   **Fix:** Use `.pipe()`! It handles backpressure automatically for you.
21. Usage of the pipe() function.
**Answer:**
`pipe()` is essentially a "connector" between a readable stream and a writable stream.
-   **Usage:** `sourceStream.pipe(destinationStream)`.
-   **Benefit:** It automatically manages **Backpressure**. If the destination is slow, pipe signals the source to pause reading.
-   **Example:** `fs.createReadStream('input.txt').pipe(zlib.createGzip()).pipe(fs.createWriteStream('input.txt.gz'));`

22. How do you convert a Buffer to JSON and vice versa?
**Answer:**
-   **Buffer to JSON:** `buf.toJSON()` returns a structure `{ type: 'Buffer', data: [ ...byte array... ] }`.
-   **JSON to Buffer:** `Buffer.from(json.data)`.
-   *Note:* Usually, you want to convert Buffer to String (`buf.toString()`) to interpret it as text/JSON string, then parse it.

23. What is the difference between `readFile` and `createReadStream`? When to use which?
**Answer:**
-   **`fs.readFile`:** Reads the **entire** file into memory (Buffer) and invokes the callback.
    -   *Use Case:* Small configuration files.
    -   *Danger:* Reading a 4GB file will crash the app (Out of Memory).
-   **`fs.createReadStream`:** Reads the file in **chunks** (default 64KB).
    -   *Use Case:* Large files, http responses, logs.
    -   *Benefit:* Low memory footprint. You can start sending data to the user before the file is fully read.

24. How do you handle errors in streams (since `try-catch` doesn't work well)?
**Answer:**
Streams emit events. `try-catch` only catches errors in synchronous code, not in asynchronous Event Emitters.
-   **Solution:** You MUST listen to the `error` event.
    ```javascript
    stream.on('error', (err) => console.error(err));
    ```
-   **Pipeline:** In newer Node versions, use `stream.pipeline()` helper which accepts a single callback for errors across the entire pipe chain.

## Networking (HTTP & TCP)
25. How do you create a simple HTTP server in Node.js without frameworks?
**Answer:**
Using the built-in `http` module.
```javascript
const http = require('http');

const server = http.createServer((req, res) => {
  if (req.url === '/') {
    res.writeHead(200, { 'Content-Type': 'text/plain' });
    res.end('Hello World');
  }
});

server.listen(3000);
```
26. What is the difference between HTTP and HTTPS modules in Node.js?
**Answer:**
-   **`http`:** Transmits data in plain text (port 80).
-   **`https`:** Transmits data encrypted via SSL/TLS (port 443).
-   **Code Difference:** `https` requires an `options` object with private key and certificate buffers passed to `createServer`.

27. How does Node.js handle SSL/TLS?
**Answer:**
It uses OpenSSL (bundled with Node.js) via the `tls` module.
The `https` module is just a wrapper that uses `tls` to establish an encrypted socket and then speaks HTTP over it.
```javascript
const opts = {
  key: fs.readFileSync('key.pem'),
  cert: fs.readFileSync('cert.pem')
};
https.createServer(opts, app).listen(443);
```

28. What is the `net` module used for?
**Answer:**
The `net` module is a **Low-Level TCP/IPC** library.
-   It allows you to create raw TCP servers and clients.
-   **Relation:** `http` module is built *on top* of `net`.
-   **Use Case:** Chat protocols, connecting to Databases, Inter-Process Communication (IPC) via Unix sockets.

29. How does Node.js handle Keep-Alive connections?
**Answer:**
Keep-Alive allows reusing the same TCP connection for multiple HTTP requests (reducing handshake overhead).
-   **Default:** Since Node 19+, Keep-Alive is ON by default (5 seconds). Before that, it was OFF.
-   **Agent:** You manage it via `http.Agent`.
    ```javascript
    const agent = new http.Agent({ keepAlive: true, keepAliveMsecs: 3000 });
    http.request({ agent, ... });
    ```

30. Explain CORS (Cross-Origin Resource Sharing) in the context of a Node.js server.
**Answer:**
CORS is a browser security feature, but the **Server** must grant permission.
-   When a browser makes a request from `domain-a.com` to `api-domain-b.com`, it sends an `Origin: domain-a.com` header.
-   The Server must respond with `Access-Control-Allow-Origin: domain-a.com` (or `*`).
-   If the server doesn't send this header, the **Browser** blocks the response JavaScript from reading the data.
-   **Preflight:** For complex requests (PUT/DELETE/Custom Headers), the browser first sends an `OPTIONS` request to check permissions.
31. What is the maximum size of HTTP headers in Node.js?
**Answer:**
-   Default is **16KB** (since Node v14+). Including 8KB for header name/value pairs.
-   Can strictly configure via `--max-http-header-size`.
-   **Impact:** A bloated cookie or JWT token in headers exceeding this limit will cause the server to drop the connection or return 431 Request Header Fields Too Large.

## File System & OS
32. How do you read a file synchronously vs asynchronously?
**Answer:**
-   **Async (Preferred):**
    `fs.readFile('path', (err, data) => {})` or `await fs.promises.readFile('path')`.
    -   Does not block event loop.
-   **Sync (Startup only):**
    `const data = fs.readFileSync('path')`.
    -   Blocks event loop. **Never** use this in request handlers (express routes). Only acceptable at startup (loading config) or in CLI scripts.

33. How do you watch for file changes in Node.js?
**Answer:**
-   **`fs.watch(dirname, callback)`**: Efficient, fast, but platform-dependent. Inconsistent across OS implementations (sometimes fires twice).
-   **`fs.watchFile(filename, callback)`**: Polls the file stats periodically (heavy CPU if interval is short).
-   **Production:** Use a wrapper library like **`chokidar`** (used by VS Code, Webpack, NestJS) which smoothes out OS inconsistencies and handles edge cases.

34. What are file descriptors?
**Answer:**
A file descriptor is an **integer** (id) that the OS kernel assigns to an open file.
-   When you `fs.open()`, you get an integer (e.g., `3`).
-   You pass this integer to `fs.read()`, `fs.write()`, and `fs.close()` instead of the filename.
-   **Limit:** OS has a limit on open file descriptors (`ulimit -n`). Failing to close files causes "EMFILE: too many open files" errors.

35. How do you get memory usage information of the current process?
**Answer:**
`process.memoryUsage()`. Returns an object:
-   **rss:** Resident Set Size (Total memory allocated for the process execution).
-   **heapTotal:** Total size of the allocated heap.
-   **heapUsed:** Actual memory used during execution. (If this keeps growing without dropping, you have a leak).
-   **external:** Memory used by C++ objects bound to V8 (Buffers).
36. How do you get the number of CPUs available on the OS?
**Answer:**
`import os from 'os';`
`os.cpus().length`.
*Use Case:* Essential when using the `cluster` module. You typically fork exactly as many worker processes as there are CPU cores to maximize hardware usage.

37. What is `__dirname` and `__filename`? Are they available in ES Modules?
**Answer:**
-   **CommonJS:**
    -   `__dirname`: Absolute path to the directory of the current file.
    -   `__filename`: Absolute path to the current file itself.
-   **ES Modules (ESM):**
    -   **NO**, they are NOT available.
    -   **Workaround:** You must reconstruct them:
        ```javascript
        import { fileURLToPath } from 'url';
        import { dirname } from 'path';
        
        const __filename = fileURLToPath(import.meta.url);
        const __dirname = dirname(__filename);
        ```

## Error Handling & Debugging
38. What is the difference between an Operational Error and a Programmer Error?
**Answer:**
-   **Operational Errors:** Runtime problems experienced by correctly-written programs (e.g., Request Timeout, 500 DB Error, Socket Hang-up, Invalid User Input).
    -   *Action:* Handle gracefully, log, maybe retry or return 4xx/5xx to user.
-   **Programmer Errors:** Bugs in the code (e.g., Reading property of undefined, syntax error, passing string to function expecting number).
    -   *Action:* Fix the code. The process should ideally crash (fail fast) so it can be restarted in a clean state (by PM2/Docker), rather than running in an undefined/corrupted state.

39. How do you use the built-in debugger in Node.js?
**Answer:**
1.  **Run:** `node inspect app.js` (CLI debugger) OR `node --inspect app.js` (Chrome DevTools).
2.  **Attach:** Open Chrome `chrome://inspect`. Click "Open dedicated DevTools for Node".
3.  **Breakpoints:** You can put `debugger;` statement in your code, and execution will pause there, allowing you to inspect variables.

40. What is the purpose of `Error.captureStackTrace`?
**Answer:**
It creates a `.stack` property on the target object.
-   **Usage:** When creating Custom Error classes.
-   **Benefit:** It allows you to **hide the implementation details** of the error generation itself from the stack trace, making the trace start from where the error was *instantiated*, not where the helper function created it.
    ```javascript
    class MyError extends Error {
      constructor(msg) {
        super(msg);
        Error.captureStackTrace(this, MyError);
      }
    }
    ```
41. How to handle uncaught exceptions? `process.on('uncaughtException')`.
**Answer:**
This event is emitted when an exception bubbles all the way back to the event loop without being caught.
-   **Usage:**
    ```javascript
    process.on('uncaughtException', (err) => {
      console.error('CRITICAL:', err);
      process.exit(1); // MANDATORY
    });
    ```
-   **Golden Rule:** You **MUST** exit the process. The application state is now corrupted/undefined. Let your process manager (PM2/Kubernetes) restart it cleanly. Never try to "resume" execution after an uncaught exception.

42. Why is it bad practice to just ignore errors in a callback?
**Answer:**
If you define `fs.readFile('file', (err, data) => { })` and ignore `err`:
1.  **Unknown State:** Code execution proceeds assuming success when it failed. calling `.toString()` on `data` (which is likely undefined) will throw a synchronous error later, crashing the stack.
2.  **Debugging Nightmare:** You swallow the evidence of the failure (Disk Full, Permission Denied), making it impossible to diagnose production issues.

43. How do memory leaks happen in Node.js and how can you detect them?
**Answer:**
-   **Causes:**
    1.  **Global Variables:** storing data in an array that never gets cleared.
    2.  **Event Listeners:** `stream.on('data')` without `stream.off()`.
    3.  **Closures:** Holding references to large objects in closures.
-   **Detection:**
    1.  **Tools:** `node --inspect` + start Chrome DevTools -> Memory Tab -> Take **Heap Snapshot**. Take two snapshots and compare/diff them.
    2.  **Symptoms:** `process.memoryUsage().heapUsed` strictly increasing over time.

## Security
44. What are some common security vulnerabilities in Node.js applications?
**Answer:**
1.  **ReDoS (Regex Denial of Service):** Badly written regex that freezes the thread on specific inputs.
2.  **Parameter Pollution:** Sending multiple params with same name `?id=1&id=2` to confuse code expecting a string but getting an array.
3.  **Command Injection:** Passing unsanitized user input to `exec()` or `spawn()`.
4.  **Prototype Pollution:** Modifying `Object.prototype` to affect all objects in the app.

45. How do you prevent ReDoS (Regular Expression Denial of Service)?
**Answer:**
-   **Avoid:** Nested quantifiers like `(a+)+`.
-   **Tools:** Use linting tools like `eslint-plugin-clean-regex`. Use `safe-regex` library to test patterns.
-   **Strategy:** Don't write complex regex yourself. Use a validation library (`class-validator`, `joi`, `validator.js`) which uses battle-tested patterns.
-   **Fallback:** If processing user-provided regex, run it in a Worker Thread or separate process sandbox with a timeout.
46. Why should you not run Node.js as root?
**Answer:**
**Principle of Least Privilege.**
If a hacker compromises your application (via Remote Code Execution vulnerability):
-   **Running as Root:** They gain full control of the entire server (can wipe disk, install rootkits).
-   **Running as User (node):** They are confined to that user's permissions. They cannot access system files or install system-wide malware.
-   *Tip:* In Docker, use `USER node` at the end of your Dockerfile.

47. What is the purpose of `helmet` middleware (conceptually)?
**Answer:**
Helmet is a collection of 14+ middleware functions that set security-related HTTP headers.
-   **Hides Info:** Removes `X-Powered-By: Express` (prevents targeting specific framework exploits).
-   **Mitigates Attacks:**
    -   `Content-Security-Policy`: Prevents XSS.
    -   `X-Frame-Options`: Prevents Clickjacking.
    -   `Strict-Transport-Security` (HSTS): Enforces HTTPS.

48. How do you securely handle passwords in Node.js? (hashing vs encryption)
**Answer:**
-   **NEVER store plain text.**
-   **Encryption is bidirectional** (can be decrypted). Do not use this for passwords.
-   **Hashing is unidirectional** (cannot be reversed). Use this.
-   **Tool:** Use **bcrypt** (or Argon2).
    -   It handles salting automatically (preventing Rainbow Table attacks).
    -   It is slow by design (preventing Brute Force attacks).
    ```javascript
    const hash = await bcrypt.hash(password, 10);
    const match = await bcrypt.compare(input, hash);
    ```

49. How do you prevent Parameter Pollution?
**Answer:**
Parameter Pollution (`HPP`) attacks occur when a malicious user sends the same parameter multiple times: `GET /search?id=123&id=456`.
-   **Node Behavior:** `req.query.id` becomes an Array `['123', '456']`.
-   **Crash:** calling `id.toUpperCase()` crashes the app because Arrays don't have that method.
-   **Fix:** Use the `hpp` middleware. It cleans `req.query` to ensure you only get the last value (or first), avoiding type confusion.

## Scaling & Performance
50. What is the Cluster module? How does it help in scaling?
**Answer:**
-   **Problem:** Node.js is single-threaded. On a 16-core CPU, 15 cores are sitting idle.
-   **Solution:** The `cluster` module allows you to Fork the main process into multiple **Worker Processes** (usually 1 per core).
-   **Mechanism:**
    -   Main process (Master) listens to the port.
    -   It distributes incoming connections to Workers (Round Robin default).
    -   Result: You process 16x more requests in parallel.
-   *Note:* In modern deployments (Kubernetes), we rarely use `cluster`. We prefer scaling **Replica Sets** (containers) instead.
51. What is the difference between Clustering and Load Balancing?
**Answer:**
-   **Clustering (Vertical Scaling):**
    -   Happens on a **Single Machine**.
    -   Utilizes multiple CPU cores by creating worker processes.
    -   Shared memory/resources strictly limited.
-   **Load Balancing (Horizontal Scaling):**
    -   Happens across **Multiple Machines** (or Containers).
    -   A Load Balancer (Nginx, AWS ALB) sits in front and distributes traffic to Server A, Server B, Server C.
    -   If Server A crashes, B and C keep running. Unaffected by single machine hardware limits.

52. How does the PM2 process manager help in production?
**Answer:**
PM2 is a production process manager.
1.  **Keep Alive:** Automatically restarts the app if it crashes.
2.  **Startup:** Starts the app automatically when the OS boots.
3.  **Cluster Mode:** `pm2 start app.js -i max` automatically enables Clustering (uses all CPUs) without changing code.
4.  **Zero-Downtime Reload:** `pm2 reload all` restarts workers one by one, ensuring requests are never dropped during deployment.

53. What is a "Reverse Proxy" (like Nginx) and why put it in front of Node.js?
**Answer:**
Node.js is great at logic, but bad at handling raw internet traffic (SSL termination, DOS attacks, Serving Static Files).
**Role of Nginx:**
1.  **Security:** Headers sanitization, slow-loris protection.
2.  **SSL Termination:** Decrypts HTTPS requests and passes plain HTTP to Node (saving Node's CPU).
3.  **Caching:** Serves images/CSS directly from disk (100x faster than Node).
4.  **Load Balancing:** Distributes traffic to multiple Node instances.

54. How do you optimize the performance of a Node.js application?
**Answer:**
1.  **Code:** Use `async/await` properly (avoid `Promise.all` blocking). Use Streams for large data.
2.  **Database:** Indexing SQL/NoSQL. Use Caching (Redis) for read-heavy data.
3.  **Architecture:** Offload CPU tasks to Worker Threads or Microservices.
4.  **Event Loop:** ensure no function blocks the loop for > 10ms.
5.  **Infrastructure:** Use a CDN for static assets. Use Nginx and PM2.

55. Explain the concept of "Event Loop Lag".
**Answer:**
Event Loop Lag measures **how long the loop is blocked**.
-   If the loop is executing a heavy JSON.parse that takes 100ms, the lag is 100ms.
-   **Consequence:** During this time, the server cannot accept new connections or process timers.
-   **Monitoring:** Libraries like `perf_hooks` or APM tools (NewRelic/Datadog) track this metric. High lag = CPU bottleneck.

## NPM & Modules
56. What is the difference between `package.json` and `package-lock.json`?
**Answer:**
-   **`package.json`:** Defines the **Project Manifest**. It lists the functional dependencies and their *allowed version ranges* (e.g., `"axios": "^1.0.0"`). It does NOT guarantee the exact same installation on every machine.
-   **`package-lock.json`:** Defines the **Exact Dependency Tree**. It records the specific version installed, its resolution URL, and a SHA integrity hash. It ensures that `npm install` on your machine and the CI/CD server yields the exact same `node_modules` structure, preventing "It works on my machine" bugs.

57. What is the difference between `dependencies` and `devDependencies`?
**Answer:**
-   **`dependencies`:** Libraries required for the application to **run in production** (e.g., `express`, `lodash`, `mongoose`).
    -   Installed via `npm install <pkg>`.
-   **`devDependencies`:** Libraries required only for **development, testing, or building** (e.g., `jest`, `eslint`, `typescript`). They are NOT installed when you run `npm install --production`.
    -   Installed via `npm install -D <pkg>`.

58. What is Semantic Versioning (SemVer)? What do `^` and `~` mean?
**Answer:**
SemVer is a versioning standard: `Major.Minor.Patch` (e.g., `1.4.2`).
-   **Major:** Breaking changes.
-   **Minor:** New features (backward compatible).
-   **Patch:** Bug fixes (backward compatible).
-   **Caret (`^`) (Default):** Updates to the latest **Minor** version. `^1.2.3` matches `1.9.9` but not `2.0.0`.
-   **Tilde (`~`):** Updates to the latest **Patch** version. `~1.2.3` matches `1.2.9` but not `1.3.0`.

59. How do you resolve circular dependencies in Node.js?
**Answer:**
-   **CommonJS (`require`):** Node.js returns a **partial copy** of the exports of the unfinished module to the infinite loop breaker. This often results in an empty object `{}` inside one of the modules.
    -   *Fix:* Move exports to the top or refactor logic into a third "shared" file.
-   **ES Modules (`import`):** ESM handles it better using "Live Bindings". The variable exists but might throw `ReferenceError` if accessed before initialization ends.

60. What is `npx` and how is it different from `npm`?
**Answer:**
-   **`npm` (Node Package Manager):** A tool to manage/install packages. It manages your `node_modules`.
-   **`npx` (Node Package Execute):** A tool to **execute** binaries from packages.
    -   *Feature:* It can run a command from a package **without installing it globally**.
    -   *Example:* `npx create-react-app my-app`. It downloads the tool to a temp cache, runs it, and deletes it.
    -   *Local Binaries:* It can also run binaries installed locally in your project (e.g., `npx jest` instead of `./node_modules/.bin/jest`).
61. How do you create a CLI tool using Node.js?
**Answer:**
1.  **Shebang:** Add `#!/usr/bin/env node` at the very top of your JS entry file (`bin.js`). This tells the system to use the node interpreter.
2.  **Package.json:** Add a `"bin"` object mapping the command name to the file.
    ```json
    "bin": {
      "my-tool": "./bin/bin.js"
    }
    ```
3.  **Link:** Run `npm link` locally to test it. Now typing `my-tool` in the terminal runs your script.

62. What is the purpose of `.npmrc` file?
**Answer:**
It is a configuration file for the NPM CLI.
-   **Scopes:** Configure where to fetch packages for specific scopes (e.g., `@mycorp:registry=https://private.registry.com`).
-   **Auth:** Store authentication tokens for private registries.
-   **Settings:** Set default flags (e.g., `save-exact=true` to always pin versions).

## Testing
63. What are the popular testing frameworks for Node.js?
**Answer:**
1.  **Jest:** (Most Popular) "Reviewer's Choice". All-in-one (Runner + Assertion + Mocking + Coverage). Developed by Meta.
2.  **Mocha:** Flexible test runner. Requires external assertion libraries (Chai) and mocking tools (Sinon).
3.  **Vitest:** New, extremely fast, specifically designed for Vite/ESM projects but works greatly with Node.js too.
4.  **Tap:** Outputs "Test Anything Protocol".

64. How do you mock external dependencies (like an API or DB) in Node.js tests?
**Answer:**
-   **Jest:** `jest.mock('axios')`. This replaces the entire module with a mock object.
    ```javascript
    import axios from 'axios';
    jest.mock('axios');
    axios.get.mockResolvedValue({ data: 'fake data' });
    ```
-   **Nock:** Specialized library for mocking **HTTP requests**. It intercepts outgoing network calls at the protocol level.
-   **Sinon:** Standalone library for Spies, Stubs, and Mocks (often used with Mocha).

65. What is the difference between Unit Testing and Integration Testing in Node.js context?
**Answer:**
-   **Unit Testing:** Tests a single function/class **in isolation**. All dependencies (DB, API, File System) are **Mocked**.
    -   *Goal:* Logic correctness.
    -   *Speed:* Milliseconds.
-   **Integration Testing:** Tests how modules **work together**. Typically involves a real (or Dockerized) database and real HTTP requests managed by tools like `supertest`.
    -   *Goal:* Wiring/Flow correctness.
    -   *Speed:* Seconds/Minutes.

## Advanced / Tricky
66. Does `setTimeout(fn, 0)` execute immediately?
**Answer:**
**No.**
-   It schedules the callback for the **Timers Phase** of the *next* Event Loop iteration (or the current one if we are just starting).
-   Even with `0ms`, there is a minimum delay (typically 1ms) defined by the system.
-   Most importantly, it executes **after** the current synchronous code stack finishes and **after** any `process.nextTick` callbacks.

67. What is the output order of: `console.log`, `process.nextTick`, `setTimeout`, `setImmediate`?
**Answer:**
```javascript
console.log('A');
setTimeout(() => console.log('B'), 0);
setImmediate(() => console.log('C'));
process.nextTick(() => console.log('D'));
console.log('E');
```
**Output:** `A`, `E`, `D`, `B`, `C` (usually).
1.  **A, E:** Synchronous code runs first.
2.  **D:** `nextTick` runs immediately after synchronous code finishes (before Event Loop continues).
3.  **B:** `setTimeout` (Timers Phase).
4.  **C:** `setImmediate` (Check Phase).
*Note:* The order of B and C can swap depending on I/O context and process startup speed, but typically B comes before C in a simple script.

68. How does the V8 garbage collector work (briefly)?
**Answer:**
V8 uses a Generational Garbage Collector (Orinoco).
1.  **New Space (Young Generation):** Where new objects are born. It's small (1-8MB). Cleanup is fast (Scavenge/Minor GC).
2.  **Old Space (Old Generation):** Objects that survive 2 Scavenge cycles are promoted here. It's large. Cleanup is heavy (Mark-Sweep-Compact/Major GC).
3.  **Mechanism:** It traverses the object graph from the "Root". Reachable objects are marked "Alive". Unreachable objects are swept (memory freed).

69. What is the `vm` module and is it safe to run untrusted code in it?
**Answer:**
The `vm` module allows compiling and running code within V8 contexts immediately.
-   **Usage:** creating a sandbox context (`vm.createContext`).
-   **Safety:** **NO, IT IS NOT SAFE.**
    -   It is NOT a security mechanism.
    -   Code running inside `vm` can easily escape the sandbox (e.g., via `this.constructor.constructor('return process')().exit()`) to access the main process.
    -   *Production Sandbox:* Use `vm2` (library) or isolated processes/Docker.

70. How does Node.js handle Copy-on-Write (CoW) when forking processes?
**Answer:**
When you use `child_process.fork()` or `cluster.fork()`:
-   On Unix systems, `fork()` utilizes OS-level Copy-on-Write.
-   The child process shares the same physical memory pages as the parent initially.
-   **Optimization:** New memory is ONLY allocated when the child (or parent) **modifies** ("Writes") a variable.
-   **Benefit:** This makes forking extremely fast and memory-efficient compared to a full memory copy at startup.

71. Explain the "thundering herd" problem and how it applies to Node.js cluster?
**Answer:**
The Thundering Herd problem occurs when a large number of processes/threads waiting for an event are all "woken up" when that event occurs, but only one can handle it.
-   **In Node.js Cluster:** Historically, if multiple worker processes listened on the same port, the OS kernel would wake them all up for an incoming connection (accept mutex). This caused CPU spikes.
-   **Solution:** Modern Node.js cluster on Windows uses Round-Robin distribution by default (master accepts connection and hands it to a worker), avoiding this. On Linux, `SO_REUSEPORT` can also mitigate this.

72. What is the difference between a "Microtask" and a "Macrotask" in the context of the Event Loop?
**Answer:**
-   **Macrotasks:** Standard Event Loop phases (Timers, I/O, Check). `setTimeout`, `setImmediate`, `I/O callbacks`. One Macrotask is processed per loop tick.
-   **Microtasks:** Higher priority tasks. `process.nextTick` and `Promise` callbacks.
-   **Significance:** The Microtask queue is processed **completely** (drained) *between* every Macrotask. If you recursively schedule microtasks (infinite loop of promises), the Event Loop will never move to the next phase (I/O starvation).

73. How would you handle a scenario where the Event Loop is blocked by a third-party library's synchronous code?
**Answer:**
If you cannot replace the library:
1.  **Worker Threads:** Move the blocking call to a `Worker`. This puts the execution on a separate thread/event loop, keeping the main server responsive.
2.  **Child Process:** Spawn a separate process to handle that specific task.
3.  **Partitioning (if applicable):** Break the large task into smaller chunks and use `setImmediate` or `setTimeout` between chunks to allow the Event Loop to breathe.

74. What is `util.promisify` and how does it work internally?
**Answer:**
It converts a traditional Callback-style function (Error-First Callback) into a function that returns a Promise.
-   **Internal Logic:** It wraps the original function. When the wrapped function is called, it returns a `new Promise`. It passes a custom callback to the original function. If that callback receives `err`, it rejects the promise; otherwise, it resolves it.
-   **Custom Symbol:** Libraries can expose a custom promisified implementation via `util.promisify.custom` symbol for optimal performance.

75. Can you explain how Node.js handles DNS lookups? Is it always asynchronous?
**Answer:**
-   **dns.lookup (Default):** It uses the OS's system-level resolver (`getaddrinfo`). This is **Blocking** at the thread level. **However**, `libuv` runs this in the **Thread Pool** to make it effectively asynchronous for Node.js.
    -   *caveat:* If you do thousands of lookups, you can exhaust the Thread Pool (default size 4), blocking FS operations!
-   **dns.resolve:** Uses a library `c-ares`. It performs network queries directly. It does **not** rely on the libuv thread pool and is truly non-blocking. Use this if you need high-performance DNS resolution.
76. What are the limits of JSON.parse/stringify in Node.js, and how do you handle JSON larger than the heap?
**Answer:**
-   **Limit:** `JSON.parse` and `JSON.stringify` build the entire object/string in the V8 Heap.
    -   If the JSON string is larger than available heap memory (e.g., 500MB string on a 512MB limit), the process **Crashes (OOM)**.
    -   Also, parsing large JSON is **Blocking** (CPU intensive).
-   **Solution:** Streaming. Use libraries like `JSONStream` or `bfj`. They parse/stringify the JSON token-by-token as a Stream, keeping memory usage constant regardless of file size.

77. How does the `http.Agent` `maxSockets` parameter affect application performance?
**Answer:**
`maxSockets` controls how many concurrent TCP sockets the Agent keeps open per origin.
-   **Node < v0.12:** Default was 5. (Performance bottleneck!).
-   **Node Modern:** Default is `Infinity`.
-   **Impact:** If restricted (e.g., set to 10), and you make 100 requests to an external API, 90 requests will be **queued** locally, increasing latency.
-   *Tuning:* Restrict it only if the *target* server has strict rate limits or connection limits.

78. What is "Keep-Alive" timeout and why does it matter for Load Balancers?
**Answer:**
It defines how long an idle TCP connection remains open.
-   **The Issue:** If your Node.js Keep-Alive timeout is **longer** than your Load Balancer's (AWS ALB/Nginx) timeout.
    1.  ALB silently closes the connection after 60s.
    2.  Node still thinks it's open (e.g., 120s).
    3.  Node tries to write to the socket.
    4.  **Error:** `ECONNRESET`.
-   **Fix:** Ensure `server.keepAliveTimeout` is slightly **larger** than the Load Balancer's idle timeout.

79. Explain the implementation of a "Graceful Shutdown" for a Node.js API.
**Answer:**
When receiving `SIGTERM` (from Kubernetes/Docker):
1.  **Stop accepting new connections:** `server.close()`.
2.  **Finish ongoing requests:** Valid connections are allowed to complete.
3.  **Close Resources:** Database connections, Redis clients, file handles.
4.  **Exit:** `process.exit(0)`.
-   *Why?* Without this, you kill active requests mid-flight, resulting in 502 Bad Gateway errors for users.

80. What is N-API (Node-API) and why is it preferred over raw V8 bindings?
**Answer:**
N-API is an ABI-stable (Application Binary Interface) layer for building Native Addons (C++).
-   **Problem with V8:** V8 C++ APIs change constantly. An addon built for Node 12 breaks on Node 14.
-   **Solution:** N-API provides a stable C interface independent of the underlying JavaScript engine.
-   **Result:** You compile your addon **once**, and it works on all future versions of Node.js without recompilation.
81. How does Node.js implement "Uncaught Exception" handling differently for Promises vs Callbacks?
**Answer:**
-   **Callbacks:** If a sync error is thrown inside an async callback and not caught, it bubbles up to the main loop and triggers `uncaughtException`, causing a crash/exit.
-   **Promises:** A rejection is effectively an **Event**, not an Error stack unwind. It triggers `unhandledRejection`. 
    -   *Crucial difference:* Historically, promises didn't crash the process (just warned). Since Node 15+, they DO crash the process, unifying behavior, but the mechanism (event vs exception bubbling) differs.

82. What happens to the Event Loop when you run a CPU-intensive loop like `while(true) {}`?
**Answer:**
It **blocks the Main Thread** completely.
-   The Event Loop is just a C++ loop running on that same thread.
-   If your JS code stays in `while(true)`, control is never returned to `libuv`.
-   **Consequence:** No I/O is processed, no network requests are accepted, no timers fire. The process appears "frozen" or "dead" to the outside world.

83. Does Node.js cache module `require` calls? How do you clear that cache?
**Answer:**
**Yes.** Use `require.cache`.
-   When `require('./a.js')` is called, Node first checks `require.cache`. If found, it returns the *exported object* immediately without reading the file or executing code.
-   **To Clear:** `delete require.cache[require.resolve('./a.js')]`.
-   *Risk:* This can lead to memory leaks (old objects staying in memory) and weird singleton behavior (different parts of app holding different versions of the module).

84. What is the impact of changing `UV_THREADPOOL_SIZE`?
**Answer:**
This environment variable controls the number of C++ threads used for blocking tasks (FS, Crypto, DNS lookup).
-   **Default:** 4.
-   **Scenario:** If you are doing mass encryption (`pbkdf2`) or reading 100 files simultaneously, 4 threads become the bottleneck.
-   **Impact:** Increasing it (e.g., to 64 or 128) can significantly improve throughput for these specific operations. Note: It does NOT help with network I/O (which is handled by OS kernel, not thread pool).

85. How do you implement a custom Readable Stream?
**Answer:**
1.  Import `Readable` from `stream`.
2.  Extend the class and implement the `_read(size)` method.
3.  Use `this.push(data)` to add data to the internal queue.
4.  Use `this.push(null)` to signal End-Of-Stream.
```javascript
class MyStream extends Readable {
  _read(size) {
    this.push('some data');
    this.push(null); // EOF
  }
}
```
