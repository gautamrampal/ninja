# JavaScript Complete Mastery Tutorial - Part 4
## Error Handling, Modules, Design Patterns & Modern Features

---

## 13. Error Handling

### 13.1 Error Types

```javascript
// ============================================
// Built-in Error Types
// ============================================

// 1. Error (base class):
const error = new Error('Something went wrong');
console.log(error.message); // \"Something went wrong\"
console.log(error.name); // \"Error\"
console.log(error.stack); // Stack trace

// 2. SyntaxError - Invalid JavaScript syntax:
try {
    eval('const x ='); // Invalid syntax
} catch (e) {
    console.log(e instanceof SyntaxError); // true
}

// 3. ReferenceError - Variable doesn't exist:
try {
    console.log(nonExistentVariable);
} catch (e) {
    console.log(e instanceof ReferenceError); // true
}

// 4. TypeError - Wrong type operation:
try {
    null.toString();
} catch (e) {
    console.log(e instanceof TypeError); // true
}

// 5. RangeError - Number out of range:
try {
    const arr = new Array(-1); // Negative length
} catch (e) {
    console.log(e instanceof RangeError); // true
}

// 6. URIError - Malformed URI:
try {
    decodeURIComponent('%');
} catch (e) {
    console.log(e instanceof URIError); // true
}

// ============================================
// Custom Errors
// ============================================

class ValidationError extends Error {
    constructor(message) {
        super(message);
        this.name = 'ValidationError';
    }
}

class DatabaseError extends Error {
    constructor(message, query) {
        super(message);
        this.name = 'DatabaseError';
        this.query = query;
    }
}

function validateUser(user) {
    if (!user.name) {
        throw new ValidationError('Name is required');
    }
    if (!user.email) {
        throw new ValidationError('Email is required');
    }
}

try {
    validateUser({ name: 'Alice' });
} catch (error) {
    if (error instanceof ValidationError) {
        console.log('Validation failed:', error.message);
    } else {
        console.log('Unknown error:', error);
    }
}
```

### 13.2 Try-Catch-Finally

```javascript
// ============================================
// Basic Try-Catch
// ============================================

try {
    // Code that might throw
    const result = riskyOperation();
    console.log(result);
} catch (error) {
    // Handle error
    console.error('Error occurred:', error.message);
}

// ============================================
// Finally Block
// ============================================

function readFile(filename) {
    let file;
    
    try {
        file = openFile(filename);
        const content = file.read();
        return content;
    } catch (error) {
        console.error('Error reading file:', error);
        return null;
    } finally {
        // Always executes (cleanup)
        if (file) {
            file.close();
        }
        console.log('Cleanup complete');
    }
}

// ============================================
// Error Propagation
// ============================================

function level3() {
    throw new Error('Error at level 3');
}

function level2() {
    level3(); // Error propagates up
}

function level1() {
    try {
        level2();
    } catch (error) {
        console.log('Caught at level 1:', error.message);
    }
}

level1(); // \"Caught at level 1: Error at level 3\"

// ============================================
// Rethrowing Errors
// ============================================

function process(data) {
    try {
        if (!data) {
            throw new Error('No data provided');
        }
        // Process data...
    } catch (error) {
        console.error('Processing failed:', error.message);
        throw error; // Re-throw for caller to handle
    }
}

try {
    process(null);
} catch (error) {
    console.log('Caller handles error');
}

// ============================================
// Async Error Handling
// ============================================

// With Promises:
asyncOperation()
    .then(result => {
        return processResult(result);
    })
    .catch(error => {
        console.error('Promise error:', error);
    });

// With async/await:
async function handleAsync() {
    try {
        const result = await asyncOperation();
        return await processResult(result);
    } catch (error) {
        console.error('Async error:', error);
        throw error;
    }
}

// Unhandled promise rejections:
window.addEventListener('unhandledrejection', event => {
    console.error('Unhandled promise rejection:', event.reason);
});

// ============================================
// Error Handling Best Practices
// ============================================

// 1. Fail fast - validate early:
function processOrder(order) {
    // Validate first
    if (!order) throw new Error('Order is required');
    if (!order.items || order.items.length === 0) {
        throw new Error('Order must have items');
    }
    
    // Then process
    return calculateTotal(order);
}

// 2. Provide context:
class OrderError extends Error {
    constructor(message, orderId, userId) {
        super(message);
        this.name = 'OrderError';
        this.orderId = orderId;
        this.userId = userId;
        this.timestamp = new Date();
    }
}

// 3. Don't swallow errors:
// Bad:
try {
    riskyOperation();
} catch (error) {
    // Silent failure - hard to debug!
}

// Good:
try {
    riskyOperation();
} catch (error) {
    console.error('Operation failed:', error);
    // Or log to error tracking service
    logError(error);
}
```

---

## 14. Modules

### 14.1 ES6 Modules

```javascript
// ============================================
// Exporting
// ============================================

// mathUtils.js

// Named exports:
export const PI = 3.14159;

export function add(a, b) {
    return a + b;
}

export function multiply(a, b) {
    return a * b;
}

// Or export all at once:
const PI2 = 3.14159;
function subtract(a, b) {
    return a - b;
}
function divide(a, b) {
    return a / b;
}

export { PI2, subtract, divide };

// Export with rename:
export { divide as div };

// Default export (one per module):
export default function calculate(operation, a, b) {
    switch(operation) {
        case 'add': return a + b;
        case 'subtract': return a - b;
        case 'multiply': return a * b;
        case 'divide': return a / b;
        default: throw new Error('Unknown operation');
    }
}

// ============================================
// Importing
// ============================================

// main.js

// Import named exports:
import { add, multiply, PI } from './mathUtils.js';

console.log(add(5, 3)); // 8
console.log(PI); // 3.14159

// Import with rename:
import { multiply as mult } from './mathUtils.js';

// Import all as namespace:
import * as MathUtils from './mathUtils.js';
console.log(MathUtils.add(5, 3)); // 8
console.log(MathUtils.PI); // 3.14159

// Import default:
import calculate from './mathUtils.js';

// Import default + named:
import calculate, { add, multiply } from './mathUtils.js';

// Import for side effects only:
import './setupGlobals.js';

// ============================================
// Dynamic Imports
// ============================================

// Load module conditionally:
async function loadModule(moduleName) {
    if (moduleName === 'math') {
        const module = await import('./mathUtils.js');
        return module;
    }
}

// Code splitting:
button.addEventListener('click', async () => {
    const { default: calculate } = await import('./mathUtils.js');
    const result = calculate('add', 5, 3);
    console.log(result);
});

// ============================================
// Module Patterns
// ============================================

// 1. Config module:
// config.js
export const API_URL = 'https://api.example.com';
export const TIMEOUT = 5000;
export const MAX_RETRIES = 3;

// 2. Service module:
// userService.js
import { API_URL } from './config.js';

export class UserService {
    async getUser(id) {
        const response = await fetch(`${API_URL}/users/${id}`);
        return response.json();
    }
    
    async createUser(userData) {
        const response = await fetch(`${API_URL}/users`, {
            method: 'POST',
            body: JSON.stringify(userData)
        });
        return response.json();
    }
}

// 3. Singleton pattern:
// database.js
class Database {
    constructor() {
        if (Database.instance) {
            return Database.instance;
        }
        Database.instance = this;
        this.connection = null;
    }
    
    connect() {
        // Connection logic
    }
}

export default new Database();

// ============================================
// Re-exporting
// ============================================

// utils/index.js
export { add, multiply } from './math.js';
export { formatDate, parseDate } from './date.js';
export { validateEmail } from './validation.js';

// Now can import from single file:
import { add, formatDate, validateEmail } from './utils/index.js';
```

### 14.2 CommonJS (Node.js)

```javascript
// ============================================
// Exporting (CommonJS)
// ============================================

// mathUtils.js

// Export individual functions:
exports.add = function(a, b) {
    return a + b;
};

exports.multiply = function(a, b) {
    return a * b;
};

// Or export all at once:
module.exports = {
    add: (a, b) => a + b,
    multiply: (a, b) => a * b,
    PI: 3.14159
};

// Export single value:
module.exports = class Calculator {
    add(a, b) {
        return a + b;
    }
};

// ============================================
// Importing (CommonJS)
// ============================================

// main.js

// Import entire module:
const mathUtils = require('./mathUtils');
console.log(mathUtils.add(5, 3));

// Destructure:
const { add, multiply } = require('./mathUtils');

// Import single export:
const Calculator = require('./calculator');
const calc = new Calculator();
```

---

## 15. Iterators and Generators

### 15.1 Iterators

```javascript
// ============================================
// Iterator Protocol
// ============================================

// An iterator must have a next() method that returns:
// { value: any, done: boolean }

const myIterator = {
    current: 0,
    last: 5,
    
    next() {
        if (this.current <= this.last) {
            return { value: this.current++, done: false };
        } else {
            return { done: true };
        }
    }
};

console.log(myIterator.next()); // { value: 0, done: false }
console.log(myIterator.next()); // { value: 1, done: false }
// ... continues until
// { done: true }

// ============================================
// Iterable Protocol
// ============================================

// An iterable must have a Symbol.iterator method
// that returns an iterator

const myIterable = {
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

// Can use with for...of:
for (const value of myIterable) {
    console.log(value); // 1, 2, 3, 4, 5
}

// Can use spread:
const arr = [...myIterable]; // [1, 2, 3, 4, 5]

// ============================================
// Custom Iterator Example: Range
// ============================================

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
                } else {
                    return { done: true };
                }
            }
        };
    }
}

const range = new Range(1, 10, 2);
for (const num of range) {
    console.log(num); // 1, 3, 5, 7, 9
}

// ============================================
// Built-in Iterables
// ============================================

// Arrays:
const arr2 = [1, 2, 3];
for (const value of arr2) {
    console.log(value);
}

// Strings:
const str = 'hello';
for (const char of str) {
    console.log(char); // h, e, l, l, o
}

// Maps:
const map = new Map([['a', 1], ['b', 2]]);
for (const [key, value] of map) {
    console.log(key, value);
}

// Sets:
const set = new Set([1, 2, 3]);
for (const value of set) {
    console.log(value);
}
```

### 15.2 Generators

```javascript
// ============================================
// Generator Functions
// ============================================

// Generator function (function* syntax):
function* simpleGenerator() {
    yield 1;
    yield 2;
    yield 3;
}

const gen = simpleGenerator();
console.log(gen.next()); // { value: 1, done: false }
console.log(gen.next()); // { value: 2, done: false }
console.log(gen.next()); // { value: 3, done: false }
console.log(gen.next()); // { value: undefined, done: true }

// Can iterate with for...of:
for (const value of simpleGenerator()) {
    console.log(value); // 1, 2, 3
}

// ============================================
// Generator Execution Flow
// ============================================

function* flowDemo() {
    console.log('Start');
    yield 1;
    console.log('After first yield');
    yield 2;
    console.log('After second yield');
    yield 3;
    console.log('End');
}

const gen2 = flowDemo();
gen2.next(); // Logs: \"Start\", returns { value: 1, done: false }
gen2.next(); // Logs: \"After first yield\", returns { value: 2, done: false }
gen2.next(); // Logs: \"After second yield\", returns { value: 3, done: false }
gen2.next(); // Logs: \"End\", returns { value: undefined, done: true }

// ============================================
// Infinite Sequences
// ============================================

function* infiniteSequence() {
    let i = 0;
    while (true) {
        yield i++;
    }
}

const infinite = infiniteSequence();
console.log(infinite.next().value); // 0
console.log(infinite.next().value); // 1
console.log(infinite.next().value); // 2

// Fibonacci generator:
function* fibonacci() {
    let [prev, curr] = [0, 1];
    while (true) {
        yield curr;
        [prev, curr] = [curr, prev + curr];
    }
}

const fib = fibonacci();
console.log(fib.next().value); // 1
console.log(fib.next().value); // 1
console.log(fib.next().value); // 2
console.log(fib.next().value); // 3
console.log(fib.next().value); // 5

// ============================================
// Passing Values to Generators
// ============================================

function* generatorWithInput() {
    const x = yield 'Enter first number';
    console.log('Received:', x);
    
    const y = yield 'Enter second number';
    console.log('Received:', y);
    
    return x + y;
}

const gen3 = generatorWithInput();
console.log(gen3.next());      // { value: 'Enter first number', done: false }
console.log(gen3.next(5));     // Logs \"Received: 5\", { value: 'Enter second number', done: false }
console.log(gen3.next(10));    // Logs \"Received: 10\", { value: 15, done: true }

// ============================================
// Generator Delegation (yield*)
// ============================================

function* gen1() {
    yield 1;
    yield 2;
}

function* gen2() {
    yield 3;
    yield 4;
}

function* combined() {
    yield* gen1(); // Delegate to gen1
    yield* gen2(); // Delegate to gen2
    yield 5;
}

console.log([...combined()]); // [1, 2, 3, 4, 5]

// Flatten nested arrays:
function* flatten(arr) {
    for (const item of arr) {
        if (Array.isArray(item)) {
            yield* flatten(item); // Recursive delegation
        } else {
            yield item;
        }
    }
}

const nested = [1, [2, [3, 4], 5], 6];
console.log([...flatten(nested)]); // [1, 2, 3, 4, 5, 6]

// ============================================
// Practical Use Cases
// ============================================

// 1. ID Generator:
function* idGenerator() {
    let id = 1;
    while (true) {
        yield id++;
    }
}

const getId = idGenerator();
console.log(getId.next().value); // 1
console.log(getId.next().value); // 2

// 2. Pagination:
function* paginate(items, pageSize) {
    for (let i = 0; i < items.length; i += pageSize) {
        yield items.slice(i, i + pageSize);
    }
}

const data = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
for (const page of paginate(data, 3)) {
    console.log(page); // [1,2,3], [4,5,6], [7,8,9], [10]
}

// 3. Tree Traversal:
class TreeNode {
    constructor(value, children = []) {
        this.value = value;
        this.children = children;
    }
    
    *[Symbol.iterator]() {
        yield this.value;
        for (const child of this.children) {
            yield* child;
        }
    }
}

const tree = new TreeNode('root', [
    new TreeNode('child1'),
    new TreeNode('child2', [
        new TreeNode('grandchild')
    ])
]);

console.log([...tree]); // ['root', 'child1', 'child2', 'grandchild']
```

---

## 16. Advanced Async Patterns

### 16.1 Promise Combinators

```javascript
// ============================================
// Promise.all() - All must succeed
// ============================================

const fetchUsers = () => Promise.resolve(['Alice', 'Bob']);
const fetchPosts = () => Promise.resolve(['Post1', 'Post2']);
const fetchComments = () => Promise.resolve(['Comment1', 'Comment2']);

Promise.all([fetchUsers(), fetchPosts(), fetchComments()])
    .then(([users, posts, comments]) => {
        console.log({ users, posts, comments });
    })
    .catch(error => {
        console.log('At least one failed:', error);
    });

// ============================================
// Promise.allSettled() - Wait for all regardless
// ============================================

Promise.allSettled([
    Promise.resolve(1),
    Promise.reject('Error'),
    Promise.resolve(3)
])
    .then(results => {
        results.forEach(result => {
            if (result.status === 'fulfilled') {
                console.log('Success:', result.value);
            } else {
                console.log('Failed:', result.reason);
            }
        });
    });

// ============================================
// Promise.race() - First to settle wins
// ============================================

// Timeout pattern:
function fetchWithTimeout(url, timeout) {
    return Promise.race([
        fetch(url),
        new Promise((_, reject) => 
            setTimeout(() => reject(new Error('Timeout')), timeout)
        )
    ]);
}

// ============================================
// Promise.any() - First success wins
// ============================================

// Try multiple servers:
Promise.any([
    fetch('https://server1.com/api'),
    fetch('https://server2.com/api'),
    fetch('https://server3.com/api')
])
    .then(response => {
        console.log('First successful response:', response);
    })
    .catch(errors => {
        console.log('All failed:', errors);
    });
```

### 16.2 Async Patterns

```javascript
// ============================================
// Sequential Execution
// ============================================

async function processSequentially(items) {
    const results = [];
    
    for (const item of items) {
        const result = await processItem(item);
        results.push(result);
    }
    
    return results;
}

// ============================================
// Parallel Execution
// ============================================

async function processParallel(items) {
    return await Promise.all(
        items.map(item => processItem(item))
    );
}

// ============================================
// Concurrency Limit
// ============================================

async function processWithLimit(items, limit) {
    const results = [];
    
    for (let i = 0; i < items.length; i += limit) {
        const batch = items.slice(i, i + limit);
        const batchResults = await Promise.all(
            batch.map(item => processItem(item))
        );
        results.push(...batchResults);
    }
    
    return results;
}

// ============================================
// Retry Logic
// ============================================

async function retry(fn, maxRetries = 3, delay = 1000) {
    for (let i = 0; i < maxRetries; i++) {
        try {
            return await fn();
        } catch (error) {
            if (i === maxRetries - 1) throw error;
            
            console.log(`Retry ${i + 1}/${maxRetries}`);
            await new Promise(resolve => setTimeout(resolve, delay));
        }
    }
}

// Usage:
const result = await retry(() => fetch('/api/data'), 3, 2000);

// ============================================
// Exponential Backoff
// ============================================

async function retryWithBackoff(fn, maxRetries = 3) {
    for (let i = 0; i < maxRetries; i++) {
        try {
            return await fn();
        } catch (error) {
            if (i === maxRetries - 1) throw error;
            
            const delay = Math.pow(2, i) * 1000; // 1s, 2s, 4s, 8s...
            console.log(`Waiting ${delay}ms before retry`);
            await new Promise(resolve => setTimeout(resolve, delay));
        }
    }
}

// ============================================
// Queue Pattern
// ============================================

class AsyncQueue {
    constructor(concurrency = 1) {
        this.concurrency = concurrency;
        this.running = 0;
        this.queue = [];
    }
    
    async add(fn) {
        while (this.running >= this.concurrency) {
            await new Promise(resolve => this.queue.push(resolve));
        }
        
        this.running++;
        
        try {
            return await fn();
        } finally {
            this.running--;
            const resolve = this.queue.shift();
            if (resolve) resolve();
        }
    }
}

// Usage:
const queue = new AsyncQueue(2); // Max 2 concurrent

const tasks = [1, 2, 3, 4, 5].map(i => 
    queue.add(() => processItem(i))
);

await Promise.all(tasks);
```

---

## 17. Memory Management

### 17.1 Garbage Collection

```javascript
// ============================================
// How Garbage Collection Works
// ============================================

/*
 * JavaScript uses automatic garbage collection (GC)
 * 
 * Mark-and-Sweep Algorithm:
 * 1. GC marks all reachable objects (from roots)
 * 2. Sweeps (deletes) unmarked objects
 * 3. Compacts memory
 * 
 * Roots:
 * - Global objects
 * - Currently executing function's local variables
 * - Other functions on call stack
 */

// ============================================
// Memory Leaks
// ============================================

// 1. Global variables (never garbage collected):
// Bad:
function leak() {
    accidentalGlobal = 'I leak!'; // Implicit global
}

// Good:
function noLeak() {
    const localVar = 'I don\\'t leak!';
}

// 2. Forgotten timers:
// Bad:
const data = loadHugeData();
setInterval(() => {
    console.log(data); // data can\\'t be garbage collected
}, 1000);

// Good:
const data2 = loadHugeData();
const timerId = setInterval(() => {
    console.log(data2);
}, 1000);
// Clear when done:
clearInterval(timerId);

// 3. Closures holding references:
// Bad:
function createClosure() {
    const hugeData = new Array(10000).fill('data');
    
    return function() {
        // Even if we don\\'t use hugeData,
        // it\\'s kept in closure scope
        console.log('Hello');
    };
}

// Good:
function createClosure2() {
    const hugeData = new Array(10000).fill('data');
    const needed = hugeData[0]; // Extract only what you need
    
    return function() {
        console.log(needed); // Only needed is retained
    };
}

// 4. Detached DOM nodes:
let button = document.getElementById('myButton');
document.body.removeChild(button);
// button still references DOM node - leak!
button = null; // Fix: clear reference

// 5. Event listeners:
class Component {
    constructor() {
        this.data = new Array(10000).fill('data');
        this.handleClick = this.handleClick.bind(this);
        
        document.addEventListener('click', this.handleClick);
    }
    
    destroy() {
        // Must remove listener!
        document.removeEventListener('click', this.handleClick);
    }
    
    handleClick() {
        console.log(this.data);
    }
}
```

### 17.2 Performance Optimization

```javascript
// ============================================
// Object Pooling
// ============================================

class ObjectPool {
    constructor(factory, reset, size = 10) {
        this.factory = factory;
        this.reset = reset;
        this.pool = [];
        this.active = new Set();
        
        // Pre-fill pool
        for (let i = 0; i < size; i++) {
            this.pool.push(factory());
        }
    }
    
    acquire() {
        let obj = this.pool.pop();
        if (!obj) {
            obj = this.factory();
        }
        this.active.add(obj);
        return obj;
    }
    
    release(obj) {
        if (this.active.has(obj)) {
            this.active.delete(obj);
            this.reset(obj);
            this.pool.push(obj);
        }
    }
}

// Usage:
const vectorPool = new ObjectPool(
    () => ({ x: 0, y: 0 }), // Factory
    (v) => { v.x = 0; v.y = 0; }, // Reset
    100 // Initial size
);

const v1 = vectorPool.acquire();
v1.x = 10;
v1.y = 20;
// Use v1...
vectorPool.release(v1); // Return to pool

// ============================================
// Debounce and Throttle
// ============================================

// Debounce: Execute after quiet period
function debounce(fn, delay) {
    let timeoutId;
    
    return function(...args) {
        clearTimeout(timeoutId);
        timeoutId = setTimeout(() => fn.apply(this, args), delay);
    };
}

// Usage: Search as user types
const searchAPI = debounce((query) => {
    fetch(`/api/search?q=${query}`);
}, 300);

// Throttle: Execute at most once per interval
function throttle(fn, interval) {
    let lastCall = 0;
    
    return function(...args) {
        const now = Date.now();
        if (now - lastCall >= interval) {
            lastCall = now;
            fn.apply(this, args);
        }
    };
}

// Usage: Scroll event
const handleScroll = throttle(() => {
    console.log('Scroll position:', window.scrollY);
}, 100);

window.addEventListener('scroll', handleScroll);

// ============================================
// Lazy Evaluation
// ============================================

class LazyValue {
    constructor(fn) {
        this.fn = fn;
        this.evaluated = false;
        this.value = undefined;
    }
    
    get() {
        if (!this.evaluated) {
            this.value = this.fn();
            this.evaluated = true;
        }
        return this.value;
    }
}

// Usage:
const expensiveComputation = new LazyValue(() => {
    console.log('Computing...');
    return Array(1000000).fill(0).map((_, i) => i * 2);
});

// Not computed yet
console.log('Created lazy value');

// Computed on first access
const result = expensiveComputation.get();

// Cached on subsequent access
const result2 = expensiveComputation.get();
```

---

## 18. Design Patterns

### 18.1 Creational Patterns

```javascript
// ============================================
// Singleton
// ============================================

class Database {
    constructor() {
        if (Database.instance) {
            return Database.instance;
        }
        
        Database.instance = this;
        this.connection = null;
    }
    
    connect() {
        if (!this.connection) {
            this.connection = 'Connected';
            console.log('Database connected');
        }
        return this.connection;
    }
}

const db1 = new Database();
const db2 = new Database();
console.log(db1 === db2); // true

// Modern way with modules:
// database.js
class DatabaseConnection {
    constructor() {
        this.connection = null;
    }
    
    connect() {
        // Connection logic
    }
}

export default new DatabaseConnection();

// ============================================
// Factory
// ============================================

class User {
    constructor(name, role) {
        this.name = name;
        this.role = role;
    }
}

class UserFactory {
    static createUser(name, role) {
        switch(role) {
            case 'admin':
                return new User(name, 'admin');
            case 'moderator':
                return new User(name, 'moderator');
            case 'user':
            default:
                return new User(name, 'user');
        }
    }
}

const admin = UserFactory.createUser('Alice', 'admin');
const user = UserFactory.createUser('Bob', 'user');

// ============================================
// Builder
// ============================================

class RequestBuilder {
    constructor() {
        this.url = '';
        this.method = 'GET';
        this.headers = {};
        this.body = null;
    }
    
    setUrl(url) {
        this.url = url;
        return this; // For chaining
    }
    
    setMethod(method) {
        this.method = method;
        return this;
    }
    
    setHeader(key, value) {
        this.headers[key] = value;
        return this;
    }
    
    setBody(body) {
        this.body = body;
        return this;
    }
    
    build() {
        return {
            url: this.url,
            method: this.method,
            headers: this.headers,
            body: this.body
        };
    }
}

// Usage:
const request = new RequestBuilder()
    .setUrl('/api/users')
    .setMethod('POST')
    .setHeader('Content-Type', 'application/json')
    .setBody({ name: 'Alice' })
    .build();
```

### 18.2 Structural Patterns

```javascript
// ============================================
// Module Pattern
// ============================================

const Calculator = (function() {
    // Private variables
    let result = 0;
    
    // Private function
    function log(operation, value) {
        console.log(`${operation}: ${value}`);
    }
    
    // Public API
    return {
        add(num) {
            result += num;
            log('Add', num);
            return this;
        },
        
        subtract(num) {
            result -= num;
            log('Subtract', num);
            return this;
        },
        
        getResult() {
            return result;
        },
        
        reset() {
            result = 0;
            return this;
        }
    };
})();

Calculator.add(10).subtract(3).add(5);
console.log(Calculator.getResult()); // 12

// ============================================
// Proxy Pattern
// ============================================

const targetObj = {
    name: 'Original',
    secret: 'Hidden'
};

const proxy = new Proxy(targetObj, {
    get(target, prop) {
        if (prop === 'secret') {
            console.log('Access denied!');
            return undefined;
        }
        console.log(`Getting ${prop}`);
        return target[prop];
    },
    
    set(target, prop, value) {
        console.log(`Setting ${prop} to ${value}`);
        if (prop === 'age' && typeof value !== 'number') {
            throw new Error('Age must be a number');
        }
        target[prop] = value;
        return true;
    }
});

console.log(proxy.name); // Logs \"Getting name\", returns \"Original\"
console.log(proxy.secret); // Logs \"Access denied!\", returns undefined
proxy.age = 30; // Logs \"Setting age to 30\"

// ============================================
// Decorator Pattern
// ============================================

class Coffee {
    cost() {
        return 5;
    }
    
    description() {
        return 'Coffee';
    }
}

class MilkDecorator {
    constructor(coffee) {
        this.coffee = coffee;
    }
    
    cost() {
        return this.coffee.cost() + 1;
    }
    
    description() {
        return this.coffee.description() + ' + Milk';
    }
}

class SugarDecorator {
    constructor(coffee) {
        this.coffee = coffee;
    }
    
    cost() {
        return this.coffee.cost() + 0.5;
    }
    
    description() {
        return this.coffee.description() + ' + Sugar';
    }
}

let coffee = new Coffee();
coffee = new MilkDecorator(coffee);
coffee = new SugarDecorator(coffee);

console.log(coffee.description()); // \"Coffee + Milk + Sugar\"
console.log(coffee.cost()); // 6.5
```

### 18.3 Behavioral Patterns

```javascript
// ============================================
// Observer Pattern
// ============================================

class Subject {
    constructor() {
        this.observers = [];
    }
    
    subscribe(observer) {
        this.observers.push(observer);
    }
    
    unsubscribe(observer) {
        this.observers = this.observers.filter(obs => obs !== observer);
    }
    
    notify(data) {
        this.observers.forEach(observer => observer.update(data));
    }
}

class Observer {
    constructor(name) {
        this.name = name;
    }
    
    update(data) {
        console.log(`${this.name} received:`, data);
    }
}

const subject = new Subject();
const obs1 = new Observer('Observer 1');
const obs2 = new Observer('Observer 2');

subject.subscribe(obs1);
subject.subscribe(obs2);

subject.notify('Hello!'); // Both observers receive

// ============================================
// Strategy Pattern
// ============================================

class PaymentStrategy {
    pay(amount) {
        throw new Error('Must implement pay()');
    }
}

class CreditCardStrategy extends PaymentStrategy {
    pay(amount) {
        console.log(`Paid $${amount} with credit card`);
    }
}

class PayPalStrategy extends PaymentStrategy {
    pay(amount) {
        console.log(`Paid $${amount} with PayPal`);
    }
}

class ShoppingCart {
    constructor(paymentStrategy) {
        this.paymentStrategy = paymentStrategy;
    }
    
    setPaymentStrategy(strategy) {
        this.paymentStrategy = strategy;
    }
    
    checkout(amount) {
        this.paymentStrategy.pay(amount);
    }
}

const cart = new ShoppingCart(new CreditCardStrategy());
cart.checkout(100); // \"Paid $100 with credit card\"

cart.setPaymentStrategy(new PayPalStrategy());
cart.checkout(50); // \"Paid $50 with PayPal\"

// ============================================
// Chain of Responsibility
// ============================================

class Handler {
    setNext(handler) {
        this.next = handler;
        return handler;
    }
    
    handle(request) {
        if (this.next) {
            return this.next.handle(request);
        }
        return null;
    }
}

class AuthHandler extends Handler {
    handle(request) {
        if (!request.user) {
            console.log('Auth failed');
            return false;
        }
        console.log('Auth passed');
        return super.handle(request);
    }
}

class ValidationHandler extends Handler {
    handle(request) {
        if (!request.data) {
            console.log('Validation failed');
            return false;
        }
        console.log('Validation passed');
        return super.handle(request);
    }
}

class SaveHandler extends Handler {
    handle(request) {
        console.log('Saved:', request.data);
        return true;
    }
}

const authHandler = new AuthHandler();
const validationHandler = new ValidationHandler();
const saveHandler = new SaveHandler();

authHandler
    .setNext(validationHandler)
    .setNext(saveHandler);

authHandler.handle({
    user: { id: 1 },
    data: { name: 'Alice' }
});
```

---

*This comprehensive JavaScript tutorial covers fundamental to advanced concepts with detailed descriptions, examples, and explanations of flow and logic. The tutorial is split across 4 parts for better organization.*
