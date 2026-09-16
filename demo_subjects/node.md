### Q1. How does Node.js handle concurrency despite being single-threaded?

**A.** 
Node.js relies on an event-driven, non-blocking I/O model implemented via the V8 JavaScript engine and the **libuv** C library.

While JavaScript execution runs on a single main thread, asynchronous I/O operations are offloaded:
1. **Operating System Non-Blocking APIs**: For network operations (HTTP, TCP, sockets), libuv leverages OS-level multiplexing interfaces like `epoll` (Linux), `kqueue` (macOS), or `IOCP` (Windows), which notify Node when sockets are readable or writable without dedicating threads.
2. **libuv Thread Pool**: For operations that cannot be performed asynchronously by operating systems (such as file system calls `fs`, DNS lookups via `getaddrinfo`, and CPU-intensive crypto/compression methods), libuv manages a background worker thread pool (default size of 4, configurable via `UV_THREADPOOL_SIZE`).

When background operations complete, libuv pushes callback events into the Event Loop's queues, which the main JavaScript thread executes sequentially.

### Q2. What are the phases of the libuv event loop in Node.js?

**A.** 
Each tick of the libuv event loop executes through distinct phases in order:

1. **Timers**: Executes callbacks scheduled by expired `setTimeout()` and `setInterval()`.
2. **Pending Callbacks (I/O Callbacks)**: Executes I/O callbacks deferred from previous loop iterations (e.g., specific system-level errors like `ECONNREFUSED`).
3. **Idle, Prepare**: Internal libuv maintenance phase.
4. **Poll**: Retrieves new I/O events from the OS kernel and executes related callbacks. If no timers are scheduled and the queue is empty, the event loop will block and wait for incoming I/O events.
5. **Check**: Executes callbacks registered with `setImmediate()`.
6. **Close Callbacks**: Handles socket and stream closure events (e.g., `socket.on('close', ...)`).

Between each phase transition, Node immediately exhausts the **Microtask Queue** (which includes `process.nextTick` followed by Promise microtasks).

### Q3. What is the difference between process.nextTick() and setImmediate()?

**A.** 
Despite their names, their timings operate inversely:

- `process.nextTick()`: Schedules a callback to be executed immediately after the currently executing operation finishes, **before** the event loop advances to the next phase or tick. It bypasses the event loop phases entirely. Overusing `process.nextTick` recursively can completely starve the event loop of I/O operations.
- `setImmediate()`: Schedules a callback to run during the **Check phase** of the event loop, guaranteed to execute *after* the current I/O polling phase completes.

If invoked within an I/O callback (e.g., inside an `fs.readFile` callback), `setImmediate` always executes before any `setTimeout(..., 0)` scheduled in that same callback because the poll phase moves directly to the check phase.

### Q4. What are Node.js Streams and how do you handle backpressure?

**A.** 
Streams are collections of data that may not be available all at once and don't have to fit entirely in memory. Node provides four stream types: `Readable`, `Writable`, `Duplex` (both read/write), and `Transform` (duplex where output is calculated from input).

**Backpressure**:
Backpressure occurs when data is read from a source faster than the downstream writable destination can consume and write it. If unmanaged, incoming chunks accumulate in the internal memory buffer, causing excessive RAM usage and potential process crashes.

**Handling Backpressure**:
1. **`stream.pipeline()` or `.pipe()`**: Automatically manages flow control. When `writable.write(chunk)` returns `false` (meaning the internal buffer exceeded `highWaterMark`), the pipe pauses the readable stream until the writable stream emits the `'drain'` event.
2. **Manual Handling**: Check the boolean return value of `dest.write(chunk)`. If `false`, call `src.pause()` and wait for `dest.on('drain', () => src.resume())`.

### Q5. What is the difference between the Cluster module and Worker Threads in Node.js?

**A.** 
- **Cluster Module**:
  Spawns multiple separate OS processes (master and workers) that share a single server port (via round-robin load balancing). Each worker process has its own dedicated V8 instance, memory space, and event loop. Communication between processes requires Inter-Process Communication (IPC) message passing or serialized data transfer. Best suited for scaling multi-core I/O-bound web servers.
- **Worker Threads (`worker_threads`)**:
  Spawns threads within the *same* OS process. Each worker thread runs its own V8 engine and event loop, but shares memory with the parent thread using `SharedArrayBuffer` without IPC serialization overhead. Best suited for CPU-heavy tasks such as image processing, cryptographic calculation, machine learning inference, and complex data transformations.

### Q6. How do CommonJS (require) and ES Modules (import) differ in Node.js?

**A.** 
1. **Loading Mechanism**: CommonJS (`require`) is synchronous and evaluates modules dynamically at runtime. ES Modules (`import`) are asynchronous and static; dependencies are parsed and linked before code execution begins.
2. **Top-Level Await**: Supported natively in ES Modules; disallowed in CommonJS modules.
3. **Module Scope Variables**: CommonJS provides `__dirname`, `__filename`, `module`, and `exports`. In ES Modules, these variables are absent; equivalent file paths are derived using `import.meta.url` and `fileURLToPath()`.
4. **Tree Shaking**: ES Modules allow static analysis at build time, enabling bundlers to eliminate dead code. CommonJS dynamic imports prevent complete dead code elimination.
5. **Interop**: ESM can import CommonJS modules using default imports, but CommonJS cannot synchronously `require()` an ESM module; it must use dynamic `import()`.

### Q7. What is the difference between Buffer and TypedArray, and how does Node allocate buffer memory?

**A.** 
- **Buffer**: A Node.js-specific class that inherits from JavaScript's `Uint8Array`. It represents a fixed-size chunk of binary raw memory allocated **outside** the V8 JavaScript garbage-collected heap in raw C++ memory.
- **TypedArray**: Standardized ECMAScript array-like views (`Uint8Array`, `Float32Array`) over an `ArrayBuffer` within the JS specification.

**Allocation**:
- `Buffer.alloc(size)`: Allocates zero-filled, safe memory initialized to 0.
- `Buffer.allocUnsafe(size)`: Fast allocation without zero-filling; may contain sensitive residual data from previously freed memory until overwritten.
- For allocations smaller than 4KB (via `Buffer.from` or small slices), Node optimizes using an internal 8KB pre-allocated memory pool (`Buffer.poolSize`), avoiding frequent OS system calls.

### Q8. How do you handle unhandled promise rejections and uncaught exceptions in production?

**A.** 
In production Node.js services:

1. **`process.on('uncaughtException', (err) => { ... })`**: Indicates an unexpected error reached the top of the event loop. The process is now in an undefined state with potential memory corruption or leaked sockets. The recommended practice is to log the stack trace to an external logging pipeline, close open servers and database connections gracefully within a short timeout, and call `process.exit(1)`. A process manager (such as PM2, Docker, or Kubernetes) should automatically restart the instance.
2. **`process.on('unhandledRejection', (reason, promise) => { ... })`**: In modern Node.js versions, unhandled promise rejections emit an unhandledRejection event and terminate the process by default with a non-zero exit code. Always catch async promise chains using `.catch()` or `try/catch` within `async/await` blocks.

### Q9. How does middleware chaining work in Express/Connect and what is the role of next()?

**A.** 
Express middleware functions have access to the request object (`req`), response object (`res`), and the next middleware function in the application's request-response cycle (`next`).

They form an ordered pipeline (the "Chain of Responsibility" pattern). When a middleware executes:
- It can inspect or mutate `req` and `res` (e.g., parse JSON, authenticate session tokens).
- It must either terminate the cycle by sending a response (`res.send()`, `res.json()`) or pass control to the next handler by calling `next()`.
- Calling `next(err)` with an argument instructs Express to skip all remaining standard middleware in the stack and jump directly to error-handling middleware (functions with four arguments: `(err, req, res, next)`).
- Forgetting to call either `next()` or send a response leaves client connections hanging indefinitely until timeout.

### Q10. How do you identify and diagnose a memory leak in a running Node.js process?

**A.** 
Step-by-step diagnostic workflow:

1. **Metrics Monitoring**: Track memory usage over time (`process.memoryUsage()`) checking `heapUsed`, `heapTotal`, and `external`. A persistent upward trend in `heapUsed` that never drops after garbage collection indicates a leak.
2. **Heap Snapshots**: Start the process with the V8 inspector enabled (`node --inspect app.js`) or programmatically use `v8.getHeapSnapshot()`. Take two or more snapshots under load separated by an interval.
3. **Comparison Analysis**: Load snapshots in Chrome DevTools Memory panel. Use the **Comparison** view between Snapshot 1 and Snapshot 2. Sort by "Delta" and "# Allocations" to identify objects that are accumulating without being collected.
4. **Retainer Tree**: Inspect the retainer graph for leaking objects to identify what root reference (e.g., an unbounded cache Map, global event listener, or closure) is preventing garbage collection.
