# Node.js Complete Mastery Tutorial - Zero to Ninja

> **Master Node.js from fundamentals to advanced internals** - A comprehensive guide covering V8 engine, LibUV, Event Loop, Streams, Module Systems, HTTP servers, and production-ready applications.

---

## 📋 Table of Contents

### Part 1: Foundation & Internals
- [Chapter 1: Node.js Architecture & Internals](#chapter-1-nodejs-architecture--internals)
- [Chapter 2: V8 Engine Deep Dive](#chapter-2-v8-engine-deep-dive)
- [Chapter 3: LibUV and Event Loop](#chapter-3-libuv-and-event-loop)
- [Chapter 4: Module System in Node.js](#chapter-4-module-system-in-nodejs)

### Part 2: Core Concepts
- [Chapter 5: Event-Driven Architecture](#chapter-5-event-driven-architecture)
- [Chapter 6: Streams in Node.js](#chapter-6-streams-in-nodejs)
- [Chapter 7: Buffers and Binary Data](#chapter-7-buffers-and-binary-data)
- [Chapter 8: File System Operations](#chapter-8-file-system-operations)

### Part 3: Network Programming
- [Chapter 9: HTTP and HTTPS](#chapter-9-http-and-https)
- [Chapter 10: Building Raw HTTP Servers](#chapter-10-building-raw-http-servers)
- [Chapter 11: HTTP Client Implementation](#chapter-11-http-client-implementation)
- [Chapter 12: Network Operations](#chapter-12-network-operations)

### Part 4: Advanced Topics
- [Chapter 13: Process and Child Processes](#chapter-13-process-and-child-processes)
- [Chapter 14: Cluster and Worker Threads](#chapter-14-cluster-and-worker-threads)
- [Chapter 15: Performance Optimization](#chapter-15-performance-optimization)
- [Chapter 16: Production Best Practices](#chapter-16-production-best-practices)

---

## Chapter 1: Node.js Architecture & Internals

### Understanding Node.js Architecture

Node.js is not just a JavaScript runtime; it's a complex architecture combining multiple components working together to provide a powerful platform for building scalable network applications.

#### The Complete Architecture Stack

```
┌─────────────────────────────────────────┐
│         Node.js Application             │
│    (Your JavaScript/TypeScript Code)    │
├─────────────────────────────────────────┤
│          Node.js Core Modules           │
│   (fs, http, crypto, stream, etc.)      │
├─────────────────────────────────────────┤
│         Node.js Bindings (C++)          │
│   (Bridge between JS and C++ layers)    │
├──────────────┬──────────────────────────┤
│   V8 Engine  │       LibUV              │
│  (JS Runtime)│  (Async I/O, Event Loop) │
├──────────────┴──────────────────────────┤
│       Operating System (Linux/Win/Mac)  │
└─────────────────────────────────────────┘
```

**Architecture Components Explained:**

1. **Application Layer** - Your JavaScript code using Node.js APIs
2. **Core Modules** - Built-in modules written in JavaScript and C++
3. **Bindings Layer** - C++ code connecting JavaScript to lower-level operations
4. **V8 Engine** - Compiles and executes JavaScript code
5. **LibUV** - Provides event loop and asynchronous I/O operations
6. **Operating System** - Underlying system APIs for file, network, and process management

### Why Node.js is Different

#### Traditional Multi-Threaded Server Model

```javascript
// Conceptual representation of traditional multi-threaded server
// Each request gets its own thread

/*
Client Request 1  →  Thread 1  →  Processing  →  Response
Client Request 2  →  Thread 2  →  Processing  →  Response
Client Request 3  →  Thread 3  →  Processing  →  Response
Client Request 4  →  Thread 4  →  Processing  →  Response

Problems:
- Each thread consumes memory (typically 2MB+ per thread)
- Context switching overhead between threads
- Thread pool limits concurrent requests
- Blocking I/O blocks entire thread
*/
```

#### Node.js Single-Threaded Event Loop Model

```javascript
// Node.js uses a single thread with non-blocking I/O
// All requests handled by one thread using event loop

/*
Client Request 1  ──┐
Client Request 2  ──┤
Client Request 3  ──┼──→  Single Thread  →  Event Loop  →  Non-blocking I/O
Client Request 4  ──┤                                    ↓
Client Request 5  ──┘                                Callbacks

Advantages:
- Minimal memory footprint
- No context switching overhead
- Can handle thousands of concurrent connections
- Efficient for I/O-bound operations
*/
```

### The Node.js Event-Driven Model

#### Synchronous vs Asynchronous Execution

```javascript
// SYNCHRONOUS (BLOCKING) CODE
// Each operation must complete before the next starts

console.log('1. Starting synchronous operations');

// Reading file synchronously - blocks execution
const fs = require('fs');
const data = fs.readFileSync('large-file.txt', 'utf8');
// Execution pauses here until file is completely read (could be seconds)
console.log('2. File read complete');

const data2 = fs.readFileSync('another-file.txt', 'utf8');
// Again, execution pauses
console.log('3. Second file read complete');

console.log('4. All operations complete');

/* Output Order (predictable but slow):
1. Starting synchronous operations
2. File read complete
3. Second file read complete
4. All operations complete

Total time: Sum of all operation times (sequential)
*/
```

```javascript
// ASYNCHRONOUS (NON-BLOCKING) CODE
// Operations start immediately, callbacks handle results

console.log('1. Starting asynchronous operations');

// Reading file asynchronously - doesn't block
fs.readFile('large-file.txt', 'utf8', (err, data) => {
  // This callback executes when file reading completes
  console.log('2. File read complete');
});

fs.readFile('another-file.txt', 'utf8', (err, data) => {
  // This callback executes when file reading completes
  console.log('3. Second file read complete');
});

console.log('4. Initiated all operations');

/* Output Order (unpredictable order for callbacks):
1. Starting asynchronous operations
4. Initiated all operations
(either 2 or 3, depending on which file loads first)
(either 2 or 3, the other one)

Total time: Maximum of individual operation times (parallel)
*/
```

### Node.js Runtime Components

#### Core Component: Process Object

```javascript
// The global process object provides information about and control over 
// the current Node.js process

// 1. Environment Information
console.log('Node.js Version:', process.version);        // v18.17.0
console.log('Platform:', process.platform);              // linux, darwin, win32
console.log('Architecture:', process.arch);              // x64, arm, etc.
console.log('Process ID:', process.pid);                 // 12345
console.log('Working Directory:', process.cwd());        // /home/user/project

// 2. Memory Usage
const memoryUsage = process.memoryUsage();
console.log('Memory Usage:');
console.log('  RSS (Resident Set Size):', memoryUsage.rss / 1024 / 1024, 'MB');
console.log('  Heap Total:', memoryUsage.heapTotal / 1024 / 1024, 'MB');
console.log('  Heap Used:', memoryUsage.heapUsed / 1024 / 1024, 'MB');
console.log('  External:', memoryUsage.external / 1024 / 1024, 'MB');

// RSS: Total memory allocated for the process
// Heap Total: Total heap allocated by V8
// Heap Used: Actual memory used by objects
// External: Memory used by C++ objects bound to JavaScript

// 3. CPU Usage
const cpuUsage = process.cpuUsage();
console.log('CPU Usage:');
console.log('  User CPU time:', cpuUsage.user, 'microseconds');
console.log('  System CPU time:', cpuUsage.system, 'microseconds');

// 4. Process Uptime
console.log('Process Uptime:', process.uptime(), 'seconds');

// 5. Environment Variables
// Access environment variables (sensitive data, configuration)
console.log('NODE_ENV:', process.env.NODE_ENV);
console.log('PORT:', process.env.PORT);
console.log('DATABASE_URL:', process.env.DATABASE_URL);

// 6. Command Line Arguments
// process.argv contains command line arguments
// node app.js --port 3000 --env production
console.log('Command Line Arguments:', process.argv);
// ['node', 'app.js', '--port', '3000', '--env', 'production']

// 7. Exit Codes
// Exit the process with status code
// 0 = success, non-zero = error
process.exit(0);  // Successful exit
process.exit(1);  // Exit with error

// 8. Process Events
// Listening to process events
process.on('exit', (code) => {
  console.log(`Process exiting with code: ${code}`);
  // Cleanup operations (must be synchronous)
});

process.on('uncaughtException', (err) => {
  console.error('Uncaught Exception:', err);
  // Log error, cleanup, then exit
  process.exit(1);
});

process.on('unhandledRejection', (reason, promise) => {
  console.error('Unhandled Promise Rejection:', reason);
  // Handle promise rejections that weren't caught
});

process.on('SIGTERM', () => {
  console.log('SIGTERM signal received: closing HTTP server');
  // Graceful shutdown
});

process.on('SIGINT', () => {
  console.log('SIGINT signal received: closing HTTP server');
  // Ctrl+C pressed
});
```

#### Global Objects in Node.js

```javascript
// Global objects available everywhere without requiring modules

// 1. console - Logging and debugging
console.log('Standard output');
console.error('Error output');
console.warn('Warning output');
console.info('Info output');
console.debug('Debug output');
console.table([{name: 'Alice', age: 25}, {name: 'Bob', age: 30}]);
console.time('operation');
// ... some operation ...
console.timeEnd('operation');  // Logs: operation: 123.456ms

// 2. Buffer - Binary data handling
const buf = Buffer.from('Hello World');
console.log(buf);  // <Buffer 48 65 6c 6c 6f 20 57 6f 72 6c 64>
console.log(buf.toString());  // Hello World

// 3. setTimeout, setInterval, setImmediate
// setTimeout: Execute after specified milliseconds
const timeoutId = setTimeout(() => {
  console.log('Executed after 1 second');
}, 1000);

// Clear timeout before it executes
clearTimeout(timeoutId);

// setInterval: Execute repeatedly at intervals
const intervalId = setInterval(() => {
  console.log('Executed every 2 seconds');
}, 2000);

// Clear interval to stop execution
clearInterval(intervalId);

// setImmediate: Execute after current event loop phase
setImmediate(() => {
  console.log('Executed immediately after I/O events');
});

// 4. __dirname and __filename (not available in ES modules)
console.log('Current directory:', __dirname);   // /home/user/project
console.log('Current file:', __filename);       // /home/user/project/app.js

// In ES modules, use import.meta.url
// import { fileURLToPath } from 'url';
// import { dirname } from 'path';
// const __filename = fileURLToPath(import.meta.url);
// const __dirname = dirname(__filename);

// 5. module and require (CommonJS)
console.log('Current module:', module);
console.log('Module exports:', module.exports);
console.log('Module filename:', module.filename);
console.log('Module loaded:', module.loaded);

// 6. global object (equivalent to window in browsers)
// Not recommended to use, but available
global.myGlobalVar = 'Available everywhere';
console.log(myGlobalVar);  // Available everywhere

// 7. process.nextTick - Execute before next event loop iteration
process.nextTick(() => {
  console.log('Executed before any I/O operations');
});

// Execution order demonstration
console.log('1. Synchronous');

setImmediate(() => {
  console.log('4. setImmediate');
});

process.nextTick(() => {
  console.log('2. nextTick');
});

setTimeout(() => {
  console.log('3. setTimeout');
}, 0);

// Output order:
// 1. Synchronous
// 2. nextTick
// 3. setTimeout
// 4. setImmediate
```

---

## Chapter 2: V8 Engine Deep Dive

### What is V8?

V8 is Google's open-source JavaScript and WebAssembly engine, written in C++. It compiles JavaScript source code to native machine code instead of using an interpreter.

#### V8 Architecture Overview

```
┌─────────────────────────────────────────────────────┐
│                JavaScript Source Code                │
└───────────────────┬─────────────────────────────────┘
                    ↓
        ┌───────────────────────┐
        │   Parser & Scanner    │
        │  (Syntax Analysis)    │
        └───────────┬───────────┘
                    ↓
        ┌───────────────────────┐
        │   Abstract Syntax     │
        │   Tree (AST)          │
        └───────────┬───────────┘
                    ↓
        ┌───────────────────────┐
        │   Ignition            │
        │   (Interpreter)       │
        │   Generates Bytecode  │
        └───────────┬───────────┘
                    ↓
        ┌───────────────────────┐
        │   TurboFan            │
        │   (Optimizing JIT)    │
        │   Machine Code        │
        └───────────┬───────────┘
                    ↓
        ┌───────────────────────┐
        │   Native Execution    │
        └───────────────────────┘
```

### V8 Compilation Pipeline

#### 1. Parsing Phase

```javascript
// JavaScript source code
function add(a, b) {
  return a + b;
}

// Step 1: Scanner (Lexical Analysis)
// Breaks code into tokens:
// [FUNCTION, IDENTIFIER('add'), LEFT_PAREN, IDENTIFIER('a'), COMMA, 
//  IDENTIFIER('b'), RIGHT_PAREN, LEFT_BRACE, RETURN, IDENTIFIER('a'), 
//  PLUS, IDENTIFIER('b'), SEMICOLON, RIGHT_BRACE]

// Step 2: Parser (Syntax Analysis)
// Creates Abstract Syntax Tree (AST):
/*
FunctionDeclaration {
  name: "add",
  params: [
    Identifier { name: "a" },
    Identifier { name: "b" }
  ],
  body: BlockStatement {
    body: [
      ReturnStatement {
        argument: BinaryExpression {
          operator: "+",
          left: Identifier { name: "a" },
          right: Identifier { name: "b" }
        }
      }
    ]
  }
}
*/
```

#### 2. Ignition: The Interpreter

```javascript
// Ignition converts AST to bytecode for fast startup

// JavaScript function
function multiply(x, y) {
  const result = x * y;
  return result;
}

// Ignition Bytecode (simplified representation)
/*
LdaNamedProperty a0, [0]    // Load x
Star r0                      // Store in register 0
LdaNamedProperty a1, [0]    // Load y
Mul r0, [1]                 // Multiply with r0
Star r1                      // Store result
Return r1                    // Return result

Benefits:
- Fast startup (no compilation delay)
- Lower memory footprint
- Good for code that runs infrequently
*/

// The interpreter executes bytecode directly
multiply(5, 3);  // Executed by Ignition initially
```

#### 3. TurboFan: The Optimizing Compiler

```javascript
// TurboFan optimizes "hot" code (frequently executed)

function calculateSum(n) {
  let sum = 0;
  // This loop will be detected as "hot" code
  for (let i = 1; i <= n; i++) {
    sum += i;
  }
  return sum;
}

// When called many times, TurboFan kicks in
for (let i = 0; i < 100000; i++) {
  calculateSum(1000);  // TurboFan optimizes this after initial runs
}

/*
TurboFan Optimization Process:
1. Profiling: Collects type information during execution
2. Speculation: Assumes types will remain consistent
3. Optimization: Generates optimized machine code
4. Deoptimization: Falls back if assumptions are violated

Optimizations applied:
- Inline caching (fast property access)
- Hidden classes (efficient property storage)
- Inline function calls (reduce function call overhead)
- Elimination of redundant code
- Loop unrolling
*/
```

### Memory Management in V8

#### Heap Structure

```javascript
// V8 Heap is divided into several generations

/*
┌─────────────────────────────────────┐
│         V8 Heap Memory              │
├─────────────────────────────────────┤
│  New Space (Young Generation)       │
│  ├── From Space (Survivor)          │
│  └── To Space (Survivor)            │
│  Short-lived objects                │
│  Size: ~1-8 MB                      │
├─────────────────────────────────────┤
│  Old Space (Old Generation)         │
│  Long-lived objects                 │
│  Size: Up to ~1.4 GB (64-bit)       │
├─────────────────────────────────────┤
│  Large Object Space                 │
│  Objects > 1MB                      │
├─────────────────────────────────────┤
│  Code Space                         │
│  JIT compiled code                  │
├─────────────────────────────────────┤
│  Map Space                          │
│  Hidden classes & object shapes     │
└─────────────────────────────────────┘
*/

// Example: Object allocation and garbage collection

function createObjects() {
  // These objects are allocated in New Space (young generation)
  const obj1 = { name: 'Alice', age: 25 };
  const obj2 = { name: 'Bob', age: 30 };
  const obj3 = { name: 'Carol', age: 35 };
  
  // Short-lived objects that will be garbage collected soon
  return obj1;  // obj2 and obj3 become eligible for GC
}

// This object survives and moves to Old Space
const persistentObject = createObjects();

// Force garbage collection (only in --expose-gc mode)
// node --expose-gc app.js
if (global.gc) {
  global.gc();  // Triggers garbage collection
}

// Monitor memory before and after GC
console.log('Before GC:', process.memoryUsage());
if (global.gc) global.gc();
console.log('After GC:', process.memoryUsage());
```

#### Garbage Collection Strategies

```javascript
// V8 uses two main garbage collection algorithms

// 1. SCAVENGER (Minor GC) - For New Space
/*
Young generation objects are collected frequently
Uses Cheney's algorithm (semi-space copying)

Process:
1. Allocate objects in "From Space"
2. When full, copy live objects to "To Space"
3. Swap "From" and "To" spaces
4. Dead objects are implicitly collected (not copied)
5. Fast but only for small heap

Triggering: Every few MB of allocations
Duration: Few milliseconds
*/

// Example: Creating temporary objects
function processData() {
  for (let i = 0; i < 10000; i++) {
    // These temporary objects are quickly collected by Scavenger
    const temp = { index: i, data: 'temporary' };
    // temp goes out of scope after each iteration
  }
}

processData();

// 2. MARK-SWEEP-COMPACT (Major GC) - For Old Space
/*
Old generation collection is less frequent but more comprehensive

Mark Phase:
- Start from roots (global objects, stack)
- Mark all reachable objects
- Use DFS (Depth-First Search)

Sweep Phase:
- Scan heap and collect unmarked objects
- Add to free list

Compact Phase:
- Move live objects together
- Reduce fragmentation
- Update pointers

Triggering: When old space is nearly full
Duration: 10-100+ milliseconds (can cause pauses)
*/

// Example: Long-lived objects in Old Space
const cache = {};

function cacheData(key, value) {
  // These objects persist and eventually move to Old Space
  cache[key] = value;
}

for (let i = 0; i < 100000; i++) {
  cacheData(`key${i}`, { data: `value${i}` });
}

// 3. INCREMENTAL MARKING
// Reduces GC pause time by marking in small increments

/*
Instead of marking all objects at once:
1. Mark a few objects
2. Let application run
3. Mark a few more objects
4. Repeat until all marked
5. Then do sweep and compact

Benefits:
- Reduces pause times
- Better for responsive applications
- Enabled by default in V8
*/

// 4. CONCURRENT MARKING
// Marking happens on separate threads while app runs

/*
Main thread continues executing JavaScript
Background threads perform marking
Reduces impact on application performance
*/

// Memory Leak Example (What to Avoid)
const leakyArray = [];

function createLeak() {
  // Objects added to array never released
  setInterval(() => {
    leakyArray.push(new Array(1000000).fill('leak'));
    console.log('Array length:', leakyArray.length);
    console.log('Memory:', process.memoryUsage().heapUsed / 1024 / 1024, 'MB');
  }, 100);
}

// Don't run this! It will consume all memory
// createLeak();

// Proper memory management
function properCleanup() {
  const cache = new Map();
  const maxSize = 1000;
  
  function addToCache(key, value) {
    // Limit cache size to prevent unbounded growth
    if (cache.size >= maxSize) {
      // Remove oldest entry (FIFO)
      const firstKey = cache.keys().next().value;
      cache.delete(firstKey);
    }
    cache.set(key, value);
  }
  
  return { addToCache, cache };
}

const { addToCache } = properCleanup();
```

### Hidden Classes and Inline Caching

```javascript
// V8 uses hidden classes for efficient property access

// BAD: Properties added in different order = different hidden classes
function Point(x, y) {
  this.x = x;  // Hidden Class C0 -> C1 (added x)
  this.y = y;  // Hidden Class C1 -> C2 (added y)
}

function Point2(x, y) {
  this.y = y;  // Hidden Class C0 -> C3 (added y) - DIFFERENT!
  this.x = x;  // Hidden Class C3 -> C4 (added x) - DIFFERENT!
}

const p1 = new Point(1, 2);    // Uses Hidden Class C2
const p2 = new Point2(1, 2);   // Uses Hidden Class C4 - Not optimized together!

// GOOD: Same property order = same hidden class
function OptimizedPoint(x, y) {
  this.x = x;
  this.y = y;
}

const p3 = new OptimizedPoint(1, 2);
const p4 = new OptimizedPoint(3, 4);
// Both p3 and p4 share the same hidden class - OPTIMIZED!

// BAD: Adding properties after construction changes hidden class
function Rectangle(width, height) {
  this.width = width;
  this.height = height;
}

const rect = new Rectangle(10, 20);
// rect has hidden class HC1

rect.color = 'red';  // Adding property changes hidden class to HC2
// Now rect has different hidden class, breaking optimization

// GOOD: Define all properties in constructor
function OptimizedRectangle(width, height, color) {
  this.width = width;
  this.height = height;
  this.color = color || null;  // Always define, even if null
}

const rect1 = new OptimizedRectangle(10, 20, 'red');
const rect2 = new OptimizedRectangle(30, 40, 'blue');
// Both share same hidden class throughout their lifecycle

// BAD: Deleting properties
const obj = { a: 1, b: 2, c: 3 };
delete obj.b;  // Changes hidden class, deoptimizes

// GOOD: Set to null or undefined instead
const obj2 = { a: 1, b: 2, c: 3 };
obj2.b = null;  // Keeps hidden class intact

// Inline Caching Example
function getX(point) {
  return point.x;  // V8 caches the location of 'x' property
}

// First call: V8 learns the hidden class and property location
getX(p3);

// Subsequent calls with same hidden class: Direct memory access (fast!)
getX(p4);

// Different hidden class: Cache miss, slower lookup
getX({ y: 2, x: 1 });  // Different property order = different hidden class

// Performance comparison
console.time('Optimized');
for (let i = 0; i < 1000000; i++) {
  getX(p3);  // Same hidden class every time - FAST
}
console.timeEnd('Optimized');

console.time('Not Optimized');
for (let i = 0; i < 1000000; i++) {
  getX({ y: i, x: i });  // New object each time - SLOW
}
console.timeEnd('Not Optimized');

/*
Results (approximate):
Optimized: 2ms
Not Optimized: 50ms

The optimized version is 25x faster!
*/
```

### V8 Optimization Tips

```javascript
// 1. Use monomorphic functions (single type)
// GOOD: Function always receives same type
function addNumbers(a, b) {
  return a + b;
}

addNumbers(1, 2);      // Always numbers
addNumbers(5, 10);     // V8 can optimize for numbers only

// BAD: Function receives different types (polymorphic)
function addAnything(a, b) {
  return a + b;
}

addAnything(1, 2);         // Numbers
addAnything('hello', 'world');  // Strings
addAnything({}, {});       // Objects
// V8 cannot optimize as effectively (megamorphic)

// 2. Initialize all object properties in constructor
class User {
  constructor(name, age, email) {
    this.name = name;
    this.age = age;
    this.email = email;
    this.verified = false;  // Initialize even optional properties
    this.lastLogin = null;  // Initialize with null if not set
  }
}

// 3. Avoid changing object shape
const config = {
  apiUrl: 'https://api.example.com',
  timeout: 5000,
  retries: 3
};

// BAD: Don't add properties later
// config.newProperty = 'value';

// GOOD: Define all properties upfront
const betterConfig = {
  apiUrl: 'https://api.example.com',
  timeout: 5000,
  retries: 3,
  newProperty: null  // Even if not used initially
};

// 4. Use arrays for collections of same type
// GOOD: All elements are numbers
const numbers = [1, 2, 3, 4, 5];

// BAD: Mixed types (elements kind transitions)
const mixed = [1, 'two', 3, 'four'];  // V8 stores differently, less efficient

// 5. Avoid sparse arrays
// GOOD: Dense array
const dense = [1, 2, 3, 4, 5];

// BAD: Sparse array (holes in array)
const sparse = [];
sparse[0] = 1;
sparse[1000] = 2;  // Creates sparse array, stored differently

// 6. Avoid arguments object in hot functions
// BAD: Using arguments
function slowSum() {
  let total = 0;
  for (let i = 0; i < arguments.length; i++) {
    total += arguments[i];
  }
  return total;
}

// GOOD: Use rest parameters
function fastSum(...numbers) {
  let total = 0;
  for (let i = 0; i < numbers.length; i++) {
    total += numbers[i];
  }
  return total;
}

// 7. Avoid try-catch in hot code paths
// BAD: Try-catch prevents optimization
function processItem(item) {
  try {
    return item.process();
  } catch (e) {
    return null;
  }
}

// GOOD: Move try-catch to wrapper
function safeProcess(item) {
  return item.process();
}

function processWithErrorHandling(item) {
  try {
    return safeProcess(item);  // Inner function can be optimized
  } catch (e) {
    return null;
  }
}

// 8. Use TypedArrays for numeric data
// Regular array
const regularArray = new Array(1000);
for (let i = 0; i < 1000; i++) {
  regularArray[i] = i;
}

// TypedArray (much faster for numeric operations)
const typedArray = new Float64Array(1000);
for (let i = 0; i < 1000; i++) {
  typedArray[i] = i;
}

// Performance comparison
console.time('Regular Array');
let sum1 = 0;
for (let i = 0; i < regularArray.length; i++) {
  sum1 += regularArray[i];
}
console.timeEnd('Regular Array');

console.time('Typed Array');
let sum2 = 0;
for (let i = 0; i < typedArray.length; i++) {
  sum2 += typedArray[i];
}
console.timeEnd('Typed Array');

// TypedArray is significantly faster!
```

---

## Chapter 3: LibUV and Event Loop

### What is LibUV?

LibUV is a multi-platform C library that provides asynchronous I/O operations, implemented using the best available mechanisms on each platform (epoll on Linux, kqueue on macOS, IOCP on Windows).

#### LibUV Architecture

```
┌───────────────────────────────┐
│     Node.js JavaScript        │
└───────────┬───────────────────┘
            ↓
┌───────────────────────────────┐
│    Node.js C++ Bindings       │
└───────────┬───────────────────┘
            ↓
┌───────────────────────────────┐
│          LibUV                │
│  ┌─────────────────────────┐  │
│  │    Event Loop           │  │
│  ├─────────────────────────┤  │
│  │    Thread Pool          │  │
│  ├─────────────────────────┤  │
│  │    Platform Abstraction │  │
│  │    - epoll (Linux)      │  │
│  │    - kqueue (macOS)     │  │
│  │    - IOCP (Windows)     │  │
│  └─────────────────────────┘  │
└───────────┬───────────────────┘
            ↓
┌───────────────────────────────┐
│    Operating System           │
└───────────────────────────────┘
```

### The Event Loop Explained

The Event Loop is the heart of Node.js's asynchronous execution model. It's what allows Node.js to perform non-blocking I/O operations despite JavaScript being single-threaded.

#### Event Loop Phases

```
   ┌───────────────────────────┐
┌─>│           timers          │  Execute setTimeout/setInterval callbacks
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │     pending callbacks     │  Execute I/O callbacks deferred to next iteration
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │       idle, prepare       │  Internal use only
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐      ┌───────────────┐
│  │           poll            │<─────┤  incoming:    │
│  │                           │      │  connections, │
│  │                           │      │  data, etc.   │
│  └─────────────┬─────────────┘      └───────────────┘
│  ┌─────────────┴─────────────┐
│  │           check           │  Execute setImmediate() callbacks
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
└──┤      close callbacks      │  Execute close event callbacks
   └───────────────────────────┘
```

#### Detailed Phase Explanation with Code

```javascript
// PHASE 1: TIMERS
// Executes callbacks scheduled by setTimeout() and setInterval()

console.log('Start');

setTimeout(() => {
  console.log('setTimeout 1 - 0ms');
}, 0);

setTimeout(() => {
  console.log('setTimeout 2 - 10ms');
}, 10);

// Timers phase checks if timer threshold has been reached
// If yes, executes the callback

// PHASE 2: PENDING CALLBACKS
// Executes I/O callbacks that were deferred (e.g., TCP errors)

const net = require('net');
const client = net.connect({ port: 9999 });  // Non-existent port

client.on('error', (err) => {
  // This callback may be deferred to pending callbacks phase
  console.log('Connection error');
});

// PHASE 3: IDLE, PREPARE
// Internal use by Node.js

// PHASE 4: POLL
// Retrieve new I/O events and execute I/O-related callbacks
// This is where most application code runs

const fs = require('fs');

// File read callback executes in poll phase
fs.readFile('file.txt', 'utf8', (err, data) => {
  console.log('File read complete');
  
  // setTimeout from within I/O callback
  setTimeout(() => {
    console.log('setTimeout inside readFile');
  }, 0);
  
  // setImmediate from within I/O callback
  setImmediate(() => {
    console.log('setImmediate inside readFile');
  });
});

/*
POLL PHASE BEHAVIOR:
1. If poll queue is not empty: Execute callbacks synchronously until queue empty
2. If poll queue is empty:
   - If setImmediate() callbacks exist: Go to check phase
   - If timers scheduled: Go back to timers phase
   - Otherwise: Wait for callbacks to be added to queue
*/

// PHASE 5: CHECK
// setImmediate() callbacks execute here

setImmediate(() => {
  console.log('setImmediate 1');
});

setImmediate(() => {
  console.log('setImmediate 2');
});

// PHASE 6: CLOSE CALLBACKS
// Execute close event callbacks

const server = require('http').createServer();

server.on('close', () => {
  console.log('Server closed');  // Executes in close callbacks phase
});

server.listen(3000);
setTimeout(() => {
  server.close();
}, 100);

console.log('End');

/*
TYPICAL OUTPUT ORDER:
Start
End
setTimeout 1 - 0ms
setImmediate 1
setImmediate 2
setTimeout 2 - 10ms
File read complete
setImmediate inside readFile  (executes before setTimeout!)
setTimeout inside readFile
Server closed
Connection error
*/
```

#### process.nextTick() and Promise Microtasks

```javascript
// process.nextTick() and Promises have HIGHER priority than event loop phases
// They are processed BETWEEN event loop phases

// MICROTASK QUEUES (Higher Priority):
// 1. process.nextTick() queue - Highest priority
// 2. Promise microtask queue - Second highest

console.log('1. Script start');

// Macrotask: Added to timers phase
setTimeout(() => {
  console.log('6. setTimeout');
}, 0);

// Microtask: Higher priority than setTimeout
Promise.resolve().then(() => {
  console.log('4. Promise 1');
}).then(() => {
  console.log('5. Promise 2');
});

// Highest priority: Executes before promises
process.nextTick(() => {
  console.log('3. nextTick 1');
});

process.nextTick(() => {
  console.log('2. nextTick 2');  // But wait! nextTick uses LIFO
});

// Macrotask: Added to check phase
setImmediate(() => {
  console.log('7. setImmediate');
});

console.log('8. Script end');

/*
ACTUAL OUTPUT ORDER:
1. Script start
8. Script end
2. nextTick 2
3. nextTick 1
4. Promise 1
5. Promise 2
6. setTimeout
7. setImmediate

EXPLANATION:
1. Synchronous code executes first (Script start, Script end)
2. Microtask queue processes (nextTick queue fully, then Promise queue fully)
3. Event loop phases execute (timers -> check)
*/

// NESTED MICROTASKS EXAMPLE
console.log('Start');

process.nextTick(() => {
  console.log('nextTick 1');
  
  process.nextTick(() => {
    console.log('nextTick nested');
  });
});

Promise.resolve().then(() => {
  console.log('Promise 1');
  
  return Promise.resolve();
}).then(() => {
  console.log('Promise 2');
});

process.nextTick(() => {
  console.log('nextTick 2');
});

console.log('End');

/*
OUTPUT:
Start
End
nextTick 1
nextTick 2
nextTick nested  (Added during nextTick processing)
Promise 1
Promise 2
*/

// PRACTICAL EXAMPLE: Why use process.nextTick()

class EventEmitter {
  constructor() {
    this.events = {};
  }
  
  on(event, listener) {
    if (!this.events[event]) {
      this.events[event] = [];
    }
    this.events[event].push(listener);
  }
  
  emit(event, ...args) {
    if (this.events[event]) {
      this.events[event].forEach(listener => {
        // Use nextTick to emit asynchronously
        // Allows event listeners to be added before emission
        process.nextTick(() => listener(...args));
      });
    }
  }
}

const emitter = new EventEmitter();

// Without nextTick, this listener might miss events emitted during construction
emitter.on('data', (data) => {
  console.log('Received:', data);
});

// Emit in same tick - listener will still catch it due to nextTick
emitter.emit('data', 'Hello');

// WARNING: process.nextTick() can starve the event loop
function recursiveNextTick() {
  process.nextTick(recursiveNextTick);  // DON'T DO THIS!
  // This prevents the event loop from ever progressing
}

// recursiveNextTick();  // Will block event loop forever!

// BETTER: Use setImmediate() for recursive async operations
function recursiveImmediate() {
  setImmediate(recursiveImmediate);  // Allows event loop to progress
}

// recursiveImmediate();  // Event loop continues processing between calls
```

### LibUV Thread Pool

```javascript
// LibUV uses a thread pool for certain operations that can't be done asynchronously
// by the operating system

/*
Thread Pool Operations:
1. File system operations (most of them)
2. DNS lookup (dns.lookup)
3. CPU-intensive crypto operations
4. Zlib (compression/decompression)

Default thread pool size: 4 threads
Can be changed with UV_THREADPOOL_SIZE environment variable (max 1024)
*/

const fs = require('fs');
const crypto = require('crypto');

// Example: File operations use thread pool
console.time('4 File Reads');

// Reading 4 files concurrently
// With default pool size of 4, all execute in parallel
fs.readFile('file1.txt', () => console.log('File 1 done'));
fs.readFile('file2.txt', () => console.log('File 2 done'));
fs.readFile('file3.txt', () => console.log('File 3 done'));
fs.readFile('file4.txt', () => console.log('File 4 done'));

console.timeEnd('4 File Reads');

console.time('5 File Reads');

// Reading 5 files concurrently
// 5th file waits for a thread to become available
fs.readFile('file1.txt', () => console.log('File 1 done'));
fs.readFile('file2.txt', () => console.log('File 2 done'));
fs.readFile('file3.txt', () => console.log('File 3 done'));
fs.readFile('file4.txt', () => console.log('File 4 done'));
fs.readFile('file5.txt', () => console.log('File 5 done'));  // Waits for available thread

console.timeEnd('5 File Reads');

// Crypto operations use thread pool
console.time('Crypto Operations');

crypto.pbkdf2('password', 'salt', 100000, 512, 'sha512', () => {
  console.log('Crypto 1 done');
});

crypto.pbkdf2('password', 'salt', 100000, 512, 'sha512', () => {
  console.log('Crypto 2 done');
});

crypto.pbkdf2('password', 'salt', 100000, 512, 'sha512', () => {
  console.log('Crypto 3 done');
});

crypto.pbkdf2('password', 'salt', 100000, 512, 'sha512', () => {
  console.log('Crypto 4 done');
});

console.timeEnd('Crypto Operations');

// Changing thread pool size (set before requiring Node.js modules)
// process.env.UV_THREADPOOL_SIZE = 8;  // Increases pool to 8 threads

// DNS lookup uses thread pool
const dns = require('dns');

dns.lookup('google.com', (err, address) => {
  console.log('DNS lookup:', address);
  // Uses thread pool
});

// But dns.resolve does NOT use thread pool
dns.resolve('google.com', (err, addresses) => {
  console.log('DNS resolve:', addresses);
  // Uses network I/O (asynchronous without thread pool)
});

// Network I/O does NOT use thread pool
const http = require('http');

// HTTP requests use kernel async I/O (epoll/kqueue/IOCP)
// Does not consume thread pool threads
http.get('http://example.com', (res) => {
  console.log('HTTP response received');
  // Fully asynchronous, no thread pool involved
});

/*
THREAD POOL VS KERNEL ASYNC I/O:

Thread Pool:
- File system operations
- CPU-intensive tasks
- Limited by pool size
- Consumes threads from pool

Kernel Async I/O:
- Network operations
- Socket communication
- Not limited by thread pool
- Uses OS-level async mechanisms
*/

// Performance optimization: Increase thread pool for FS-heavy apps
// process.env.UV_THREADPOOL_SIZE = Math.max(4, require('os').cpus().length);
```

### Event Loop Performance Patterns

```javascript
// PATTERN 1: Deferring Heavy Computation

// BAD: Blocking computation
function fibonacci(n) {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
}

// This blocks the event loop for seconds!
console.log(fibonacci(40));  // DON'T DO THIS in production

// GOOD: Break computation into chunks
function fibonacciAsync(n, callback) {
  setImmediate(() => {
    if (n <= 1) {
      callback(n);
    } else {
      fibonacciAsync(n - 1, (val1) => {
        fibonacciAsync(n - 2, (val2) => {
          callback(val1 + val2);
        });
      });
    }
  });
}

// Now event loop can process other events between calculations
fibonacciAsync(10, (result) => {
  console.log('Fibonacci result:', result);
});

// BETTER: Use Worker Threads for CPU-intensive tasks
const { Worker } = require('worker_threads');

function fibonacciWorker(n) {
  return new Promise((resolve, reject) => {
    const worker = new Worker(`
      const { parentPort, workerData } = require('worker_threads');
      
      function fib(n) {
        if (n <= 1) return n;
        return fib(n - 1) + fib(n - 2);
      }
      
      parentPort.postMessage(fib(workerData));
    `, { eval: true, workerData: n });
    
    worker.on('message', resolve);
    worker.on('error', reject);
  });
}

// Main thread remains responsive while worker calculates
fibonacciWorker(40).then(result => {
  console.log('Fibonacci from worker:', result);
});

// PATTERN 2: Batching Operations

// BAD: Process array items one by one
async function processArray(items) {
  for (const item of items) {
    await processItem(item);  // Waits for each item
  }
}

// GOOD: Batch process items
async function processBatched(items, batchSize = 100) {
  for (let i = 0; i < items.length; i += batchSize) {
    const batch = items.slice(i, i + batchSize);
    await Promise.all(batch.map(item => processItem(item)));
    
    // Allow event loop to process other events between batches
    await new Promise(resolve => setImmediate(resolve));
  }
}

// PATTERN 3: Avoiding Blocking Operations

// BAD: Synchronous file operations block event loop
const fs = require('fs');
const data = fs.readFileSync('large-file.txt', 'utf8');  // BLOCKS!
console.log(data);

// GOOD: Asynchronous file operations
fs.readFile('large-file.txt', 'utf8', (err, data) => {
  console.log(data);  // Event loop continues while reading
});

// BETTER: Use promises for cleaner async code
const fsPromises = require('fs').promises;

async function readFilesAsync() {
  const data = await fsPromises.readFile('large-file.txt', 'utf8');
  console.log(data);
}

readFilesAsync();

// PATTERN 4: Monitoring Event Loop Lag

const start = Date.now();
let lastCheck = start;

function checkEventLoopLag() {
  const now = Date.now();
  const lag = now - lastCheck - 100;  // Expected to be ~100ms
  
  if (lag > 50) {  // Lag > 50ms indicates blocked event loop
    console.warn(`Event loop lag detected: ${lag}ms`);
  }
  
  lastCheck = now;
  setTimeout(checkEventLoopLag, 100);
}

checkEventLoopLag();

// PATTERN 5: Using setImmediate vs setTimeout(0)

// In I/O callbacks, setImmediate is always faster
fs.readFile('file.txt', () => {
  // This executes first (check phase)
  setImmediate(() => console.log('setImmediate'));
  
  // This executes second (next timers phase)
  setTimeout(() => console.log('setTimeout'), 0);
});

// Outside I/O callbacks, order is unpredictable
setImmediate(() => console.log('immediate'));
setTimeout(() => console.log('timeout'), 0);

// BEST PRACTICE: Use setImmediate for deferring work in I/O callbacks
```

---

## Chapter 4: Module System in Node.js

### Understanding Modules

Every file in Node.js is treated as a separate module. Modules help organize code, promote reusability, and prevent global namespace pollution.

#### How Module System Works

```javascript
// When you require a module, Node.js wraps the module code in a function:

// YOUR CODE (math.js):
const add = (a, b) => a + b;
const subtract = (a, b) => a - b;
module.exports = { add, subtract };

// WHAT NODE.JS ACTUALLY DOES (invisible to you):
(function(exports, require, module, __filename, __dirname) {
  // Your code goes here
  const add = (a, b) => a + b;
  const subtract = (a, b) => a - b;
  module.exports = { add, subtract };
  
  return module.exports;
});

/*
FUNCTION PARAMETERS:
- exports: Reference to module.exports
- require: Function to load other modules
- module: Reference to current module
- __filename: Absolute path to current file
- __dirname: Absolute path to current directory
*/

// This is why these variables are available in every module
// without explicit declaration!
```

### CommonJS Module System

```javascript
// FILE: utils/math.js
// EXPORTING WITH module.exports

// Method 1: Export object with multiple functions
const add = (a, b) => {
  return a + b;
};

const subtract = (a, b) => {
  return a - b;
};

const multiply = (a, b) => {
  return a * b;
};

const divide = (a, b) => {
  if (b === 0) throw new Error('Division by zero');
  return a / b;
};

// Export multiple functions as object properties
module.exports = {
  add,
  subtract,
  multiply,
  divide
};

// Method 2: Export single function or class
module.exports = function Calculator() {
  this.add = (a, b) => a + b;
  this.subtract = (a, b) => a - b;
};

// Method 3: Adding properties to exports
exports.add = (a, b) => a + b;
exports.subtract = (a, b) => a - b;

// WARNING: Don't reassign exports directly
// exports = { add, subtract };  // WRONG! Breaks the reference
// Always use: module.exports = { add, subtract };
```

```javascript
// FILE: app.js
// IMPORTING MODULES

// Import local module (relative path required)
const math = require('./utils/math');

// Use imported functions
console.log(math.add(5, 3));       // 8
console.log(math.subtract(10, 4)); // 6
console.log(math.multiply(6, 7));  // 42
console.log(math.divide(20, 4));   // 5

// Destructure specific exports
const { add, multiply } = require('./utils/math');

console.log(add(2, 3));      // 5
console.log(multiply(4, 5)); // 20

// Import core/built-in modules (no path needed)
const fs = require('fs');      // File system
const http = require('http');  // HTTP server
const path = require('path');  // Path manipulation
const os = require('os');      // Operating system info

// Import third-party modules (from node_modules)
const express = require('express');
const mongoose = require('mongoose');
const axios = require('axios');

// These must be installed first:
// npm install express mongoose axios
```

### Module Resolution Algorithm

```javascript
// When you call require('./module'), Node.js follows this algorithm:

/*
RESOLUTION STEPS:

1. If path starts with '/', './', or '../':
   - Load as file or directory (local module)
   
2. If path doesn't start with path prefix:
   - Try to load as core module (built-in)
   - If not core module, load from node_modules
   
3. For file loading:
   a. Try exact filename: require('./math')
   b. Try with .js extension: ./math.js
   c. Try with .json extension: ./math.json  
   d. Try with .node extension: ./math.node (native addon)
   
4. For directory loading:
   a. Look for package.json with "main" field
   b. Try index.js
   c. Try index.json
   d. Try index.node

5. For node_modules loading:
   a. Start from current directory's node_modules
   b. Check parent directory's node_modules
   c. Continue up to root directory
   d. Check global node_modules
*/

// EXAMPLES:

// 1. Relative path - looks in current directory
require('./math');         // ./math.js or ./math/index.js

// 2. Absolute path
require('/home/user/app/math');  // Absolute file system path

// 3. Core module
require('fs');            // Built-in Node.js module

// 4. Third-party from node_modules
require('express');       // node_modules/express/

// 5. Nested node_modules
require('lodash');
/*
Searches in order:
  ./node_modules/lodash
  ../node_modules/lodash
  ../../node_modules/lodash
  ...up to root
  /usr/lib/node_modules/lodash (global)
*/

// 6. Loading directory as module
require('./lib');
/*
Tries in order:
  ./lib/package.json (read "main" field)
  ./lib/index.js
  ./lib/index.json
  ./lib/index.node
*/

// 7. Loading JSON file
const config = require('./config.json');
// Automatically parses JSON and returns object
console.log(config.apiUrl);

// 8. Loading native addon
const addon = require('./build/Release/addon.node');
// Loads compiled C++ addon
```

### Module Caching

```javascript
// NODE.JS CACHES MODULES AFTER FIRST REQUIRE

// FILE: counter.js
let count = 0;

module.exports = {
  increment: () => ++count,
  get: () => count
};

// FILE: app.js
const counter1 = require('./counter');
const counter2 = require('./counter');  // Returns cached version!

console.log(counter1.increment());  // 1
console.log(counter1.increment());  // 2

console.log(counter2.get());        // 2 (same module instance!)
console.log(counter1 === counter2); // true (identical references)

// Cache is based on resolved filename
console.log(require.cache);
/*
{
  '/path/to/app.js': Module { ... },
  '/path/to/counter.js': Module { ... },
  ...
}
*/

// CLEARING CACHE (rarely needed, useful for testing)
delete require.cache[require.resolve('./counter')];

const counter3 = require('./counter');  // Fresh instance
console.log(counter3.get());  // 0 (new module instance)

// PRACTICAL USE: Creating singleton pattern
// FILE: database.js
class Database {
  constructor() {
    if (!Database.instance) {
      this.connection = null;
      Database.instance = this;
    }
    return Database.instance;
  }
  
  connect(url) {
    console.log('Connecting to database:', url);
    this.connection = { url, connected: true };
  }
  
  getConnection() {
    return this.connection;
  }
}

module.exports = new Database();  // Export instance, not class

// FILE: app.js
const db1 = require('./database');
const db2 = require('./database');

db1.connect('mongodb://localhost/mydb');
console.log(db2.getConnection());  // Same connection!
```

### ES Modules (ESM) in Node.js

```javascript
// Modern JavaScript module system (ES6+)
// Requires package.json with "type": "module" OR .mjs extension

// FILE: package.json
{
  "type": "module"
}

// FILE: math.mjs (or math.js with "type": "module")
// NAMED EXPORTS
export const add = (a, b) => {
  return a + b;
};

export const subtract = (a, b) => {
  return a - b;
};

export const multiply = (a, b) => {
  return a * b;
};

// Export as declared
export function divide(a, b) {
  if (b === 0) throw new Error('Division by zero');
  return a / b;
}

// Export list at end
const PI = 3.14159;
const E = 2.71828;

export { PI, E };

// DEFAULT EXPORT (one per module)
export default class Calculator {
  add(a, b) {
    return a + b;
  }
  
  subtract(a, b) {
    return a - b;
  }
}

// Can have both named and default exports
export { Calculator as Calc };

// FILE: app.mjs
// IMPORTING ES MODULES

// Import default export
import Calculator from './math.mjs';

const calc = new Calculator();
console.log(calc.add(5, 3));

// Import named exports
import { add, subtract, multiply } from './math.mjs';

console.log(add(10, 5));
console.log(subtract(10, 5));

// Import all named exports as namespace
import * as math from './math.mjs';

console.log(math.add(2, 3));
console.log(math.PI);
console.log(math.E);

// Import with alias
import { add as sum, multiply as product } from './math.mjs';

console.log(sum(1, 2));
console.log(product(3, 4));

// Import default and named together
import Calculator, { add, PI } from './math.mjs';

// Dynamic imports (works anywhere, returns Promise)
async function loadMath() {
  const math = await import('./math.mjs');
  console.log(math.add(5, 5));
}

loadMath();

// Or with .then()
import('./math.mjs').then(math => {
  console.log(math.multiply(2, 3));
});

// Conditional dynamic import
const moduleName = process.env.USE_ALT ? './alt-math.mjs' : './math.mjs';
const math = await import(moduleName);
```

### CommonJS vs ES Modules

```javascript
// KEY DIFFERENCES

/*
┌─────────────────┬───────────────────┬────────────────────┐
│   Feature       │   CommonJS        │   ES Modules       │
├─────────────────┼───────────────────┼────────────────────┤
│ Syntax          │ require/exports   │ import/export      │
│ Loading         │ Synchronous       │ Asynchronous       │
│ When loaded     │ Runtime           │ Parse time         │
│ Tree shaking    │ No                │ Yes                │
│ Top-level await │ No                │ Yes                │
│ File extension  │ .js, .cjs         │ .mjs, .js*         │
│ this value      │ module.exports    │ undefined          │
│ Dynamic imports │ Native            │ import()           │
└─────────────────┴───────────────────┴────────────────────┘

* with "type": "module" in package.json
*/

// INTEROPERABILITY

// FILE: commonjs-module.cjs
module.exports = {
  greet: (name) => `Hello, ${name}!`
};

// FILE: esm-module.mjs
// ESM can import CommonJS
import cjsModule from './commonjs-module.cjs';
console.log(cjsModule.greet('Alice'));

// But can't use named imports from CommonJS
// import { greet } from './commonjs-module.cjs';  // ERROR!

// FILE: require-esm.cjs
// CommonJS CANNOT directly require ESM
// const esmModule = require('./esm-module.mjs');  // ERROR!

// Must use dynamic import
(async () => {
  const esmModule = await import('./esm-module.mjs');
  console.log(esmModule.default);
})();

// PRACTICAL HYBRID APPROACH

// FILE: package.json
{
  "type": "module",
  "exports": {
    ".": {
      "import": "./esm/index.mjs",
      "require": "./cjs/index.cjs"
    }
  }
}

// This allows package to support both module systems
```

### Creating Your Own Package

```javascript
// FILE: package.json
{
  "name": "my-awesome-package",
  "version": "1.0.0",
  "description": "A demo package showing module patterns",
  "main": "index.js",              // Entry point for CommonJS
  "module": "index.mjs",           // Entry point for ESM (some bundlers)
  "type": "commonjs",              // Default module type
  "exports": {
    ".": {
      "import": "./index.mjs",     // ESM import
      "require": "./index.js"      // CommonJS require
    },
    "./utils": {
      "import": "./utils.mjs",
      "require": "./utils.js"
    }
  },
  "keywords": ["demo", "modules"],
  "author": "Your Name",
  "license": "MIT"
}

// FILE: index.js (CommonJS entry)
const { createLogger } = require('./utils');

class MyPackage {
  constructor(options = {}) {
    this.options = options;
    this.logger = createLogger(options.logLevel);
  }
  
  doSomething() {
    this.logger.info('Doing something...');
    return 'Done!';
  }
}

module.exports = MyPackage;
module.exports.default = MyPackage;  // For ESM default import
module.exports.createLogger = require('./utils').createLogger;

// FILE: index.mjs (ESM entry)
import { createLogger } from './utils.mjs';

export default class MyPackage {
  constructor(options = {}) {
    this.options = options;
    this.logger = createLogger(options.logLevel);
  }
  
  doSomething() {
    this.logger.info('Doing something...');
    return 'Done!';
  }
}

export { createLogger };

// FILE: utils.js (CommonJS utils)
const createLogger = (level = 'info') => {
  return {
    info: (msg) => console.log(`[INFO] ${msg}`),
    warn: (msg) => console.warn(`[WARN] ${msg}`),
    error: (msg) => console.error(`[ERROR] ${msg}`)
  };
}

exports.createLogger = createLogger;

// USAGE EXAMPLES

// CommonJS consumer
const MyPackage = require('my-awesome-package');
const pkg = new MyPackage({ logLevel: 'info' });
pkg.doSomething();

// ESM consumer
import MyPackage from 'my-awesome-package';
const pkg = new MyPackage({ logLevel: 'info' });
pkg.doSomething();
```

### Module Best Practices

```javascript
// 1. ONE MODULE, ONE PURPOSE

// BAD: Too many unrelated functions in one module
module.exports = {
  formatDate: () => {},
  sendEmail: () => {},
  validatePassword: () => {},
  generateReport: () => {}
};

// GOOD: Separate modules by concern
// date-utils.js
exports.formatDate = () => {};

// email.js  
exports.sendEmail = () => {};

// validators.js
exports.validatePassword = () => {};

// reports.js
exports.generateReport = () => {};

// 2. CLEAR NAMING CONVENTIONS

// GOOD: Descriptive file and function names
// user-controller.js
exports.createUser = async (userData) => {};
exports.updateUser = async (userId, updates) => {};
exports.deleteUser = async (userId) => {};

// 3. AVOID CIRCULAR DEPENDENCIES

// BAD: Circular dependency
// a.js
const b = require('./b');
exports.doSomething = () => b.doSomethingElse();

// b.js
const a = require('./a');  // Circular!
exports.doSomethingElse = () => a.doSomething();

// GOOD: Extract shared logic to third module
// shared.js
exports.sharedLogic = () => {};

// a.js
const shared = require('./shared');
exports.doSomething = () => shared.sharedLogic();

// b.js
const shared = require('./shared');
exports.doSomethingElse = () => shared.sharedLogic();

// 4. USE INDEX.JS FOR CLEAN IMPORTS

// Instead of:
const userModel = require('./models/user');
const postModel = require('./models/post');
const commentModel = require('./models/comment');

// Create models/index.js:
module.exports = {
  User: require('./user'),
  Post: require('./post'),
  Comment: require('./comment')
};

// Now import as:
const { User, Post, Comment } = require('./models');

// 5. DOCUMENT YOUR EXPORTS

/**
 * User authentication utilities
 * @module auth
 */

/**
 * Hashes a password using bcrypt
 * @param {string} password - Plain text password
 * @param {number} rounds - Number of salt rounds (default: 10)
 * @returns {Promise<string>} Hashed password
 */
exports.hashPassword = async (password, rounds = 10) => {
  const bcrypt = require('bcrypt');
  return bcrypt.hash(password, rounds);
};

/**
 * Compares password with hash
 * @param {string} password - Plain text password  
 * @param {string} hash - Hashed password
 * @returns {Promise<boolean>} True if passwords match
 */
exports.comparePassword = async (password, hash) => {
  const bcrypt = require('bcrypt');
  return bcrypt.compare(password, hash);
};
```

---

## Chapter 5: Event-Driven Architecture