# JavaScript Complete Mastery Tutorial
## From Basic to Advanced - Comprehensive Guide

---

## Table of Contents

### Part 1: Fundamentals & Core Concepts
1. [Introduction to JavaScript](#1-introduction-to-javascript)
2. [Variables and Data Types](#2-variables-and-data-types)
3. [Operators](#3-operators)
4. [Control Flow](#4-control-flow)
5. [Functions](#5-functions)
6. [Scope and Closures](#6-scope-and-closures)
7. [Objects](#7-objects)
8. [Arrays](#8-arrays)

### Part 2: Intermediate Concepts
9. [Advanced Functions](#9-advanced-functions)
10. [Prototypes and Inheritance](#10-prototypes-and-inheritance)
11. [Classes (ES6+)](#11-classes-es6)
12. [Asynchronous JavaScript](#12-asynchronous-javascript)
13. [Error Handling](#13-error-handling)
14. [Modules](#14-modules)
15. [Iterators and Generators](#15-iterators-and-generators)

### Part 3: Advanced Concepts
16. [Advanced Async Patterns](#16-advanced-async-patterns)
17. [Memory Management](#17-memory-management)
18. [Design Patterns](#18-design-patterns)
19. [Functional Programming](#19-functional-programming)
20. [Performance Optimization](#20-performance-optimization)
21. [Web APIs](#21-web-apis)
22. [Advanced ES6+ Features](#22-advanced-es6-features)

---

## Part 1: Fundamentals & Core Concepts

---

## 1. Introduction to JavaScript

### 1.1 What is JavaScript?

JavaScript is a high-level, interpreted programming language that conforms to the ECMAScript specification. It's one of the core technologies of the World Wide Web, alongside HTML and CSS.

**Key Characteristics:**
- **Dynamic typing**: Variables can hold values of any type
- **Prototype-based**: Uses prototypes rather than classes (though ES6 added class syntax)
- **First-class functions**: Functions are treated as values
- **Event-driven**: Designed to respond to user interactions
- **Single-threaded**: Uses an event loop for concurrency

### 1.2 JavaScript Execution Environment

```javascript
// JavaScript runs in different environments:

// 1. Browser Environment
// - Has access to DOM (Document Object Model)
// - Window object is the global object
console.log(window); // Browser global object

// 2. Node.js Environment
// - No DOM, but has file system access
// - Global object is 'global'
console.log(global); // Node.js global object

// 3. How JavaScript Code is Executed:
/*
 * Flow of Execution:
 * 1. Code is parsed (syntax checking)
 * 2. Execution context is created
 * 3. Memory allocation phase (hoisting)
 * 4. Code execution phase
 */
```

### 1.3 The Event Loop

Understanding how JavaScript handles asynchronous operations:

```javascript
/*
 * JavaScript Event Loop Architecture:
 * 
 * Call Stack → Web APIs → Callback Queue → Event Loop → Call Stack
 * 
 * Flow:
 * 1. Synchronous code runs on the call stack
 * 2. Async operations (setTimeout, fetch) are handled by Web APIs
 * 3. Callbacks are placed in the callback queue
 * 4. Event loop checks if call stack is empty
 * 5. If empty, moves callbacks from queue to stack
 */

console.log('1'); // Executes first (synchronous)

setTimeout(() => {
    console.log('2'); // Executes last (asynchronous - moved to Web API)
}, 0);

console.log('3'); // Executes second (synchronous)

// Output: 1, 3, 2
// Logic: Even with 0ms delay, setTimeout is async and goes through the event loop
```

---

## 2. Variables and Data Types

### 2.1 Variable Declarations

JavaScript has three ways to declare variables, each with different scoping rules:

```javascript
// ============================================
// VAR - Function Scoped (Old way - avoid in modern code)
// ============================================

function varExample() {
    var x = 1; // Function scoped
    
    if (true) {
        var x = 2; // Same variable! Overwrites the outer x
        console.log(x); // 2
    }
    
    console.log(x); // 2 (not 1!) - var ignores block scope
}

// Hoisting with var:
console.log(hoistedVar); // undefined (not an error!)
var hoistedVar = 'I am hoisted';
// Why? JavaScript moves declarations to top:
// var hoistedVar; // Declaration is hoisted
// console.log(hoistedVar); // undefined
// hoistedVar = 'I am hoisted'; // Assignment stays in place

// ============================================
// LET - Block Scoped (Modern, preferred for reassignable variables)
// ============================================

function letExample() {
    let x = 1; // Block scoped
    
    if (true) {
        let x = 2; // Different variable! Separate scope
        console.log(x); // 2
    }
    
    console.log(x); // 1 (maintains its value)
}

// Temporal Dead Zone (TDZ):
// console.log(hoistedLet); // ReferenceError! Cannot access before initialization
let hoistedLet = 'I am not hoisted the same way';
// Why? Let IS hoisted but not initialized until the line executes

// ============================================
// CONST - Block Scoped, Immutable Binding (Modern, preferred for constants)
// ============================================

const PI = 3.14159;
// PI = 3.14; // TypeError! Cannot reassign const

// IMPORTANT: const prevents reassignment, NOT mutation!
const person = { name: 'Alice' };
person.name = 'Bob'; // ✓ Allowed! Modifying object properties
person.age = 30;     // ✓ Allowed! Adding properties
// person = {}; // ✗ TypeError! Cannot reassign the binding

const arr = [1, 2, 3];
arr.push(4); // ✓ Allowed! Mutating array
// arr = []; // ✗ TypeError! Cannot reassign

// Best Practice Flow:
// 1. Always use const by default
// 2. Use let only when you need to reassign
// 3. Never use var in modern JavaScript
```

### 2.2 Primitive Data Types

JavaScript has 7 primitive data types:

```javascript
// ============================================
// 1. NUMBER - All numbers are 64-bit floating point
// ============================================

let integer = 42;
let float = 3.14;
let negative = -10;
let scientific = 5e3; // 5000
let binary = 0b1010; // 10 in decimal
let octal = 0o12; // 10 in decimal
let hex = 0xFF; // 255 in decimal

// Special numeric values:
let infinity = Infinity; // Result of division by zero
let negInfinity = -Infinity;
let notANumber = NaN; // "Not a Number" - result of invalid math

// Important NaN behavior:
console.log(NaN === NaN); // false! NaN is not equal to itself
console.log(isNaN(NaN)); // true - use this to check for NaN
console.log(Number.isNaN(NaN)); // true - more reliable

// Number precision issues (IEEE 754):
console.log(0.1 + 0.2); // 0.30000000000000004 (not 0.3!)
console.log(0.1 + 0.2 === 0.3); // false!
// Solution: Use toFixed() or compare with epsilon
const epsilon = Number.EPSILON;
console.log(Math.abs((0.1 + 0.2) - 0.3) < epsilon); // true

// ============================================
// 2. STRING - Sequence of characters
// ============================================

let single = 'Single quotes';
let double = "Double quotes";
let backtick = `Backticks for templates`;

// String concatenation:
let firstName = 'John';
let lastName = 'Doe';
let fullName = firstName + ' ' + lastName; // Old way

// Template literals (ES6+) - much better!
let greeting = `Hello, ${firstName} ${lastName}!`;
let multiline = `
    This is
    a multiline
    string
`;

// Expression interpolation:
let a = 5, b = 10;
console.log(`Sum: ${a + b}`); // "Sum: 15"
console.log(`Comparison: ${a > b ? 'a is greater' : 'b is greater'}`);

// String methods (strings are immutable!):
let text = 'JavaScript';
console.log(text.length); // 10
console.log(text.toUpperCase()); // "JAVASCRIPT"
console.log(text.toLowerCase()); // "javascript"
console.log(text.slice(0, 4)); // "Java"
console.log(text.indexOf('Script')); // 4
console.log(text.includes('Script')); // true
console.log(text.split('')); // ['J','a','v','a','S','c','r','i','p','t']

// ============================================
// 3. BOOLEAN - true or false
// ============================================

let isActive = true;
let isCompleted = false;

// Truthy and Falsy values (crucial concept!):
/*
 * Falsy values (convert to false):
 * - false
 * - 0, -0
 * - '' (empty string)
 * - null
 * - undefined
 * - NaN
 * 
 * Everything else is truthy!
 */

// Examples of truthiness:
if ('0') console.log('String "0" is truthy'); // Executes!
if ([]) console.log('Empty array is truthy'); // Executes!
if ({}) console.log('Empty object is truthy'); // Executes!

// Explicit conversion:
console.log(Boolean(0)); // false
console.log(Boolean('')); // false
console.log(Boolean('hello')); // true
console.log(Boolean([])); // true

// ============================================
// 4. UNDEFINED - Variable declared but not assigned
// ============================================

let notAssigned;
console.log(notAssigned); // undefined

function noReturn() {
    // No return statement
}
console.log(noReturn()); // undefined

let obj = {};
console.log(obj.nonExistent); // undefined

// ============================================
// 5. NULL - Intentional absence of value
// ============================================

let emptyValue = null; // Explicitly "nothing"

// null vs undefined:
console.log(typeof undefined); // "undefined"
console.log(typeof null); // "object" (JavaScript bug, but won't be fixed)

console.log(null == undefined); // true (loose equality)
console.log(null === undefined); // false (strict equality)

// ============================================
// 6. SYMBOL - Unique identifier (ES6+)
// ============================================

let sym1 = Symbol('description');
let sym2 = Symbol('description');
console.log(sym1 === sym2); // false! Each symbol is unique

// Use case: Private object properties
const privateKey = Symbol('private');
const obj2 = {
    [privateKey]: 'This is private',
    public: 'This is public'
};
console.log(obj2[privateKey]); // Accessible with the symbol
console.log(Object.keys(obj2)); // ['public'] - symbols are hidden!

// ============================================
// 7. BIGINT - Arbitrary precision integers (ES2020)
// ============================================

// Regular numbers have limits:
console.log(Number.MAX_SAFE_INTEGER); // 9007199254740991

// BigInt for larger numbers:
let bigNum = 1234567890123456789012345678901234567890n; // Note the 'n'
let bigNum2 = BigInt('1234567890123456789012345678901234567890');

// BigInt operations:
console.log(bigNum + 10n); // Must use BigInt on both sides
// console.log(bigNum + 10); // TypeError! Cannot mix BigInt and Number
console.log(bigNum * 2n);
console.log(bigNum / 3n); // Integer division only

// ============================================
// Type Checking and Conversion
// ============================================

// typeof operator:
console.log(typeof 42); // "number"
console.log(typeof 'hello'); // "string"
console.log(typeof true); // "boolean"
console.log(typeof undefined); // "undefined"
console.log(typeof null); // "object" (quirk!)
console.log(typeof Symbol()); // "symbol"
console.log(typeof 123n); // "bigint"
console.log(typeof {}); // "object"
console.log(typeof []); // "object" (arrays are objects!)
console.log(typeof function(){}); // "function"

// Type coercion (implicit conversion):
console.log('5' + 3); // "53" (number to string)
console.log('5' - 3); // 2 (string to number)
console.log('5' * '2'); // 10 (both to numbers)
console.log(true + 1); // 2 (true becomes 1)
console.log(false + 1); // 1 (false becomes 0)

// Explicit conversion:
String(123); // "123"
Number('123'); // 123
Number('hello'); // NaN
parseInt('123px'); // 123 (stops at first non-digit)
parseFloat('3.14.15'); // 3.14
Boolean(0); // false
```

### 2.3 Reference Types (Objects)

```javascript
// ============================================
// Objects, Arrays, and Functions are reference types
// ============================================

// Primitive vs Reference comparison:

// Primitives: Copied by value
let x = 10;
let y = x; // y gets a COPY of x's value
x = 20;
console.log(y); // Still 10 (independent copy)

// References: Copied by reference
let obj1 = { value: 10 };
let obj2 = obj1; // obj2 gets a REFERENCE to the same object
obj1.value = 20;
console.log(obj2.value); // 20! (both point to same object)

// Visual representation:
/*
 * Primitives:
 * x: [10] ← Stack
 * y: [10] ← Stack (separate copy)
 * 
 * References:
 * obj1: [ref:0x001] ← Stack
 * obj2: [ref:0x001] ← Stack (same reference)
 *       ↓
 * {value: 20} ← Heap (single object)
 */

// Equality comparisons:
console.log({} === {}); // false! Different objects in memory
console.log([] === []); // false! Different arrays in memory

let arr1 = [1, 2, 3];
let arr2 = arr1;
console.log(arr1 === arr2); // true! Same reference

// Cloning objects (shallow copy):
let original = { a: 1, b: 2 };
let clone1 = { ...original }; // Spread operator
let clone2 = Object.assign({}, original); // Object.assign

original.a = 999;
console.log(clone1.a); // 1 (independent copy)

// Deep copy problem:
let nested = { a: 1, b: { c: 2 } };
let shallowCopy = { ...nested };
nested.b.c = 999;
console.log(shallowCopy.b.c); // 999! Nested objects still share reference

// Deep copy solutions:
let deepCopy1 = JSON.parse(JSON.stringify(nested)); // Simple but limited
let deepCopy2 = structuredClone(nested); // Modern, better approach
```

---

## 3. Operators

### 3.1 Arithmetic Operators

```javascript
// ============================================
// Basic Arithmetic
// ============================================

let add = 10 + 5; // 15
let subtract = 10 - 5; // 5
let multiply = 10 * 5; // 50
let divide = 10 / 5; // 2
let remainder = 10 % 3; // 1 (modulo)
let power = 2 ** 3; // 8 (exponentiation - ES2016)

// Increment/Decrement:
let count = 0;
count++; // Post-increment: returns old value, then increments
console.log(count); // 1

let result1 = count++; // result1 = 1, count = 2
let result2 = ++count; // count = 3, result2 = 3

// Flow explanation:
/*
 * count++ (post-increment):
 * 1. Save current value (1)
 * 2. Increment count (2)
 * 3. Return saved value (1)
 * 
 * ++count (pre-increment):
 * 1. Increment count (3)
 * 2. Return new value (3)
 */

// Unary plus/minus:
let str = '42';
let num = +str; // Convert to number: 42
let neg = -num; // Negate: -42

// Special cases:
console.log(5 / 0); // Infinity
console.log(-5 / 0); // -Infinity
console.log(0 / 0); // NaN
console.log(Math.sqrt(-1)); // NaN
```

### 3.2 Comparison Operators

```javascript
// ============================================
// Equality Operators
// ============================================

// == (Loose equality - performs type coercion)
console.log(5 == '5'); // true (string converted to number)
console.log(true == 1); // true
console.log(false == 0); // true
console.log(null == undefined); // true
console.log('' == 0); // true

// === (Strict equality - no type coercion)
console.log(5 === '5'); // false (different types)
console.log(true === 1); // false
console.log(null === undefined); // false

// BEST PRACTICE: Always use === and !== (strict)
// Why? Prevents unexpected type coercion bugs

// Comparison flow with ==:
/*
 * 5 == '5'
 * ↓
 * 1. Check if types are same? No
 * 2. Try to convert '5' to number → 5
 * 3. Compare 5 == 5 → true
 */

// ============================================
// Relational Operators
// ============================================

let a = 10, b = 5;

console.log(a > b); // true
console.log(a < b); // false
console.log(a >= 10); // true
console.log(a <= 10); // true

// String comparison (lexicographical):
console.log('apple' < 'banana'); // true (dictionary order)
console.log('2' < '12'); // false! String comparison, not numeric
console.log(2 < 12); // true (numeric)

// Mixed type comparison:
console.log('10' > 5); // true (string '10' converted to 10)

// ============================================
// Special Cases
// ============================================

// NaN comparisons:
console.log(NaN == NaN); // false
console.log(NaN === NaN); // false
console.log(Object.is(NaN, NaN)); // true! (special equality)

// -0 and +0:
console.log(-0 === +0); // true
console.log(Object.is(-0, +0)); // false! (can distinguish)
```

### 3.3 Logical Operators

```javascript
// ============================================
// AND (&&), OR (||), NOT (!)
// ============================================

// AND - Returns first falsy value or last value
console.log(true && true); // true
console.log(true && false); // false
console.log('Hello' && 'World'); // 'World' (both truthy, returns last)
console.log(0 && 'World'); // 0 (first falsy value)

// AND evaluation flow (short-circuit):
/*
 * a && b && c
 * ↓
 * 1. Evaluate a: if falsy, return a (don't check b or c)
 * 2. Evaluate b: if falsy, return b (don't check c)
 * 3. Evaluate c: return c
 */

// OR - Returns first truthy value or last value
console.log(false || true); // true
console.log(false || false); // false
console.log('Hello' || 'World'); // 'Hello' (first truthy)
console.log(0 || 'Default'); // 'Default' (0 is falsy)

// OR evaluation flow (short-circuit):
/*
 * a || b || c
 * ↓
 * 1. Evaluate a: if truthy, return a (don't check b or c)
 * 2. Evaluate b: if truthy, return b (don't check c)
 * 3. Evaluate c: return c
 */

// NOT - Inverts boolean value
console.log(!true); // false
console.log(!false); // true
console.log(!0); // true (0 is falsy)
console.log(!!'hello'); // true (double negation converts to boolean)

// Practical uses:

// 1. Default values with ||
function greet(name) {
    name = name || 'Guest'; // If name is falsy, use 'Guest'
    return `Hello, ${name}`;
}
console.log(greet()); // "Hello, Guest"
console.log(greet('Alice')); // "Hello, Alice"

// Problem with ||: falsy values aren't always "missing"
console.log(greet(0)); // "Hello, Guest" (0 treated as missing!)

// 2. Conditional execution with &&
let user = { name: 'Alice' };
user && console.log(user.name); // Safe access, logs "Alice"

// 3. Guard clauses:
function process(data) {
    if (!data) return; // Exit early if no data
    // Process data...
}

// ============================================
// Nullish Coalescing (??) - ES2020
// ============================================

// Returns right side only if left is null or undefined
let value1 = null ?? 'default'; // 'default'
let value2 = undefined ?? 'default'; // 'default'
let value3 = 0 ?? 'default'; // 0 (not null/undefined!)
let value4 = '' ?? 'default'; // '' (not null/undefined!)

// Comparison with ||:
console.log(0 || 'default'); // 'default' (0 is falsy)
console.log(0 ?? 'default'); // 0 (0 is not null/undefined)

// Use case: Distinguishing 0/'' from null/undefined
function configure(timeout) {
    timeout = timeout ?? 5000; // 0 is valid, only replace null/undefined
}

// ============================================
// Optional Chaining (?.) - ES2020
// ============================================

let user1 = {
    name: 'Alice',
    address: {
        city: 'NYC'
    }
};

let user2 = {
    name: 'Bob'
    // No address
};

// Without optional chaining (unsafe):
// console.log(user2.address.city); // TypeError!

// With optional chaining:
console.log(user1.address?.city); // "NYC"
console.log(user2.address?.city); // undefined (no error!)

// Flow of ?. operator:
/*
 * user2.address?.city
 * ↓
 * 1. Evaluate user2.address
 * 2. Is it null/undefined? Yes → return undefined (stop)
 * 3. If no → continue to .city
 */

// Works with methods:
user.greet?.(); // Calls greet() only if it exists

// Works with arrays:
let arr = null;
console.log(arr?.[0]); // undefined (no error)
```

### 3.4 Assignment Operators

```javascript
// ============================================
// Compound Assignment
// ============================================

let x = 10;

// Arithmetic assignments:
x += 5; // x = x + 5 → 15
x -= 3; // x = x - 3 → 12
x *= 2; // x = x * 2 → 24
x /= 4; // x = x / 4 → 6
x %= 4; // x = x % 4 → 2
x **= 3; // x = x ** 3 → 8

// Logical assignments (ES2021):
let a = null;
a ||= 'default'; // a = a || 'default' → 'default'

let b = 5;
b &&= 10; // b = b && 10 → 10 (b was truthy)

let c = null;
c ??= 'default'; // c = c ?? 'default' → 'default'

// Use case: Lazy initialization
let cache = {};
function getValue(key) {
    cache[key] ??= expensiveComputation(); // Only compute if not cached
    return cache[key];
}

// ============================================
// Destructuring Assignment
// ============================================

// Array destructuring:
let [first, second, third] = [1, 2, 3];
console.log(first); // 1
console.log(second); // 2

// Skip elements:
let [, , third2] = [1, 2, 3];
console.log(third2); // 3

// Rest pattern:
let [head, ...tail] = [1, 2, 3, 4, 5];
console.log(head); // 1
console.log(tail); // [2, 3, 4, 5]

// Default values:
let [x1 = 0, y1 = 0] = [5];
console.log(x1, y1); // 5, 0

// Object destructuring:
let person = { name: 'Alice', age: 30, city: 'NYC' };
let { name, age } = person;
console.log(name); // "Alice"

// Rename variables:
let { name: fullName, age: years } = person;
console.log(fullName); // "Alice"

// Default values:
let { name: n, country = 'USA' } = person;
console.log(country); // "USA"

// Nested destructuring:
let user3 = {
    id: 1,
    profile: {
        name: 'Alice',
        email: 'alice@example.com'
    }
};
let { profile: { name: userName, email } } = user3;
console.log(userName, email); // "Alice", "alice@example.com"

// Practical use: Function parameters
function displayUser({ name, age = 'Unknown' }) {
    console.log(`${name} is ${age} years old`);
}
displayUser({ name: 'Bob', age: 25 }); // "Bob is 25 years old"
displayUser({ name: 'Charlie' }); // "Charlie is Unknown years old"
```

### 3.5 Other Operators

```javascript
// ============================================
// Ternary (Conditional) Operator
// ============================================

let age = 20;
let status = age >= 18 ? 'Adult' : 'Minor';

// Flow:
/*
 * condition ? valueIfTrue : valueIfFalse
 * ↓
 * 1. Evaluate condition
 * 2. If truthy → evaluate and return valueIfTrue
 * 3. If falsy → evaluate and return valueIfFalse
 */

// Nested ternary (use sparingly!):
let score = 85;
let grade = score >= 90 ? 'A' :
            score >= 80 ? 'B' :
            score >= 70 ? 'C' : 'F';

// ============================================
// Comma Operator
// ============================================

// Evaluates all expressions, returns last
let result = (1 + 2, 3 + 4); // result = 7

// Common use: Multiple expressions in for loop
for (let i = 0, j = 10; i < 5; i++, j--) {
    console.log(i, j);
}

// ============================================
// typeof Operator
// ============================================

console.log(typeof 42); // "number"
console.log(typeof 'hello'); // "string"
console.log(typeof true); // "boolean"
console.log(typeof undefined); // "undefined"
console.log(typeof Symbol()); // "symbol"
console.log(typeof {}); // "object"
console.log(typeof []); // "object" (arrays are objects)
console.log(typeof null); // "object" (historical bug)
console.log(typeof function(){}); // "function"

// ============================================
// instanceof Operator
// ============================================

console.log([] instanceof Array); // true
console.log({} instanceof Object); // true
console.log(new Date() instanceof Date); // true

class Person {}
let person1 = new Person();
console.log(person1 instanceof Person); // true

// Flow of instanceof:
/*
 * obj instanceof Constructor
 * ↓
 * 1. Check if Constructor.prototype is in obj's prototype chain
 * 2. Walk up the chain: obj → obj.__proto__ → obj.__proto__.__proto__ → ...
 * 3. If found → true, else → false
 */

// ============================================
// delete Operator
// ============================================

let obj = { a: 1, b: 2, c: 3 };
delete obj.b; // Removes property 'b'
console.log(obj); // { a: 1, c: 3 }

let arr2 = [1, 2, 3, 4, 5];
delete arr2[2]; // Creates a "hole" in array
console.log(arr2); // [1, 2, empty, 4, 5]
console.log(arr2.length); // Still 5!

// Better for arrays:
arr2.splice(2, 1); // Remove 1 element at index 2

// ============================================
// in Operator
// ============================================

let car = { brand: 'Tesla', model: 'Model 3' };
console.log('brand' in car); // true
console.log('color' in car); // false

// Checks inherited properties too:
console.log('toString' in car); // true (inherited from Object)

// Array indices:
let numbers = [10, 20, 30];
console.log(0 in numbers); // true
console.log(5 in numbers); // false

// ============================================
// Spread Operator (...) - ES6
// ============================================

// Array spreading:
let arr1 = [1, 2, 3];
let arr22 = [4, 5, 6];
let combined = [...arr1, ...arr22]; // [1, 2, 3, 4, 5, 6]

// Clone array:
let original = [1, 2, 3];
let copy = [...original];

// Function arguments:
function sum(a, b, c) {
    return a + b + c;
}
let nums = [1, 2, 3];
console.log(sum(...nums)); // 6

// Object spreading (ES2018):
let obj1 = { a: 1, b: 2 };
let obj2 = { c: 3, d: 4 };
let merged = { ...obj1, ...obj2 }; // { a: 1, b: 2, c: 3, d: 4 }

// Override properties:
let defaults = { theme: 'light', fontSize: 14 };
let user4 = { ...defaults, theme: 'dark' }; // theme overridden to 'dark'

// ============================================
// Rest Parameter (...) - ES6
// ============================================

// In function parameters (gather remaining arguments):
function multiply(multiplier, ...numbers) {
    return numbers.map(n => n * multiplier);
}
console.log(multiply(2, 1, 2, 3, 4)); // [2, 4, 6, 8]

// In destructuring:
let [first1, ...rest] = [1, 2, 3, 4, 5];
console.log(first1); // 1
console.log(rest); // [2, 3, 4, 5]
```

---

## 4. Control Flow

### 4.1 Conditional Statements

```javascript
// ============================================
// IF-ELSE Statement
// ============================================

let temperature = 25;

// Simple if:
if (temperature > 30) {
    console.log('It\'s hot!');
}

// If-else:
if (temperature > 30) {
    console.log('It\'s hot!');
} else {
    console.log('It\'s not hot.');
}

// If-else if-else chain:
if (temperature > 30) {
    console.log('It\'s hot!');
} else if (temperature > 20) {
    console.log('It\'s warm.');
} else if (temperature > 10) {
    console.log('It\'s cool.');
} else {
    console.log('It\'s cold!');
}

// Flow diagram:
/*
 * Start
 * ↓
 * Is temp > 30? ──Yes──> "It's hot!" ──> End
 * │
 * No
 * ↓
 * Is temp > 20? ──Yes──> "It's warm." ─> End
 * │
 * No
 * ↓
 * Is temp > 10? ──Yes──> "It's cool." ─> End
 * │
 * No
 * ↓
 * "It's cold!" ───────────> End
 */

// ============================================
// SWITCH Statement
// ============================================

let day = 3;
let dayName;

switch (day) {
    case 1:
        dayName = 'Monday';
        break; // Important! Without break, falls through
    case 2:
        dayName = 'Tuesday';
        break;
    case 3:
        dayName = 'Wednesday';
        break;
    case 4:
        dayName = 'Thursday';
        break;
    case 5:
        dayName = 'Friday';
        break;
    case 6:
    case 7: // Fall-through intentionally (grouped cases)
        dayName = 'Weekend';
        break;
    default:
        dayName = 'Invalid day';
}

// Switch with expressions:
let result;
switch (true) { // Trick: use true to evaluate expressions
    case temperature > 30:
        result = 'hot';
        break;
    case temperature > 20:
        result = 'warm';
        break;
    default:
        result = 'cold';
}

// Modern alternative: Object lookup
const dayMap = {
    1: 'Monday',
    2: 'Tuesday',
    3: 'Wednesday',
    4: 'Thursday',
    5: 'Friday',
    6: 'Weekend',
    7: 'Weekend'
};
let dayName2 = dayMap[day] || 'Invalid day';

// Switch flow:
/*
 * 1. Evaluate switch expression once
 * 2. Compare with each case using === (strict equality)
 * 3. Execute matching case block
 * 4. If no break, continue to next case (fall-through)
 * 5. If no match, execute default (if present)
 */
```

### 4.2 Loops

```javascript
// ============================================
// FOR Loop
// ============================================

// Traditional for loop:
for (let i = 0; i < 5; i++) {
    console.log(i); // 0, 1, 2, 3, 4
}

// Loop flow:
/*
 * for (initialization; condition; increment)
 * ↓
 * 1. Execute initialization (let i = 0) ──once──┐
 * 2. Check condition (i < 5) ─────────────←────┘
 *    If false → exit loop                 │
 * 3. Execute loop body                     │
 * 4. Execute increment (i++) ────────────┘
 * 5. Go to step 2
 */

// Multiple variables:
for (let i = 0, j = 10; i < j; i++, j--) {
    console.log(`i=${i}, j=${j}`);
}

// Nested loops:
for (let i = 1; i <= 3; i++) {
    for (let j = 1; j <= 3; j++) {
        console.log(`${i} x ${j} = ${i * j}`);
    }
}

// ============================================
// WHILE Loop
// ============================================

let count = 0;
while (count < 5) {
    console.log(count);
    count++;
}

// While flow:
/*
 * while (condition) { body }
 * ↓
 * 1. Check condition ─────────────←────┐
 *    If false → exit loop              │
 * 2. Execute body ─────────────────┘
 * 3. Go to step 1
 */

// Common pattern: Read until end
let items = ['a', 'b', 'c'];
let i = 0;
while (i < items.length) {
    console.log(items[i]);
    i++;
}

// ============================================
// DO-WHILE Loop
// ============================================

let num = 0;
do {
    console.log(num); // Executes at least once!
    num++;
} while (num < 5);

// Difference from while: body executes before condition check
let x = 10;
do {
    console.log(x); // This runs once even though x < 5 is false
} while (x < 5);

// ============================================
// FOR...OF Loop (ES6) - Iterate over values
// ============================================

let fruits = ['apple', 'banana', 'cherry'];

// Clean syntax for array iteration:
for (let fruit of fruits) {
    console.log(fruit); // 'apple', 'banana', 'cherry'
}

// Works with any iterable:
let str2 = 'hello';
for (let char of str2) {
    console.log(char); // 'h', 'e', 'l', 'l', 'o'
}

// With index (using entries()):
for (let [index, fruit] of fruits.entries()) {
    console.log(`${index}: ${fruit}`);
}

// ============================================
// FOR...IN Loop - Iterate over keys
// ============================================

let person = { name: 'Alice', age: 30, city: 'NYC' };

// Iterate over object keys:
for (let key in person) {
    console.log(`${key}: ${person[key]}`);
}
// Output:
// name: Alice
// age: 30
// city: NYC

// Warning: for...in iterates over inherited properties too!
Object.prototype.inherited = 'value';
for (let key in person) {
    console.log(key); // Includes 'inherited'!
}

// Solution: Use hasOwnProperty()
for (let key in person) {
    if (person.hasOwnProperty(key)) {
        console.log(`${key}: ${person[key]}`);
    }
}

// Better modern approach for objects:
Object.keys(person).forEach(key => {
    console.log(`${key}: ${person[key]}`);
});

Object.entries(person).forEach(([key, value]) => {
    console.log(`${key}: ${value}`);
});

// ============================================
// for...in vs for...of
// ============================================

let arr = ['a', 'b', 'c'];

// for...in gives indices (keys):
for (let key in arr) {
    console.log(key); // '0', '1', '2' (strings!)
}

// for...of gives values:
for (let value of arr) {
    console.log(value); // 'a', 'b', 'c'
}

// Rule of thumb:
// for...in → Objects (keys)
// for...of → Arrays, Strings, Iterables (values)
```

### 4.3 Loop Control

```javascript
// ============================================
// BREAK - Exit loop immediately
// ============================================

for (let i = 0; i < 10; i++) {
    if (i === 5) {
        break; // Exit loop when i is 5
    }
    console.log(i); // 0, 1, 2, 3, 4
}

// Find first match:
let numbers = [1, 3, 5, 8, 10, 12];
let firstEven;
for (let num of numbers) {
    if (num % 2 === 0) {
        firstEven = num;
        break; // Stop searching after first match
    }
}
console.log(firstEven); // 8

// ============================================
// CONTINUE - Skip current iteration
// ============================================

for (let i = 0; i < 5; i++) {
    if (i === 2) {
        continue; // Skip when i is 2
    }
    console.log(i); // 0, 1, 3, 4 (skips 2)
}

// Print only even numbers:
for (let i = 0; i < 10; i++) {
    if (i % 2 !== 0) {
        continue; // Skip odd numbers
    }
    console.log(i); // 0, 2, 4, 6, 8
}

// ============================================
// LABELS - Named loops for break/continue
// ============================================

// Break out of nested loop:
outerLoop: for (let i = 0; i < 3; i++) {
    for (let j = 0; j < 3; j++) {
        if (i === 1 && j === 1) {
            break outerLoop; // Breaks out of both loops
        }
        console.log(`i=${i}, j=${j}`);
    }
}

// Continue outer loop from inner:
outerLoop2: for (let i = 0; i < 3; i++) {
    for (let j = 0; j < 3; j++) {
        if (j === 1) {
            continue outerLoop2; // Skip to next i
        }
        console.log(`i=${i}, j=${j}`);
    }
}

// Flow with labels:
/*
 * outerLoop: for (...) {
 *     innerLoop: for (...) {
 *         break outerLoop; ────> Exits both loops
 *         break innerLoop; ────> Exits only inner loop
 *         continue outerLoop; ──> Skips to next outer iteration
 *     }
 * }
 */
```

---

## 5. Functions

### 5.1 Function Declarations

```javascript
// ============================================
// Function Declaration
// ============================================

function greet(name) {
    return `Hello, ${name}!`;
}

console.log(greet('Alice')); // "Hello, Alice!"

// Key features of function declarations:
// 1. Hoisted to the top of scope
console.log(sayHi()); // Works! (called before declaration)
function sayHi() {
    return 'Hi!';
}

// 2. Have a name (useful for debugging)
// 3. Can be called before they are defined

// Hoisting flow:
/*
 * Code written:
 * console.log(sayHi());
 * function sayHi() { return 'Hi!'; }
 * 
 * How JavaScript sees it:
 * function sayHi() { return 'Hi!'; } // Hoisted to top
 * console.log(sayHi()); // Now this works
 */

// ============================================
// Function Expression
// ============================================

const greet2 = function(name) {
    return `Hello, ${name}!`;
};

console.log(greet2('Bob')); // "Hello, Bob!"

// Key differences from declarations:
// 1. NOT hoisted (can't use before definition)
// console.log(sayBye()); // ReferenceError!
const sayBye = function() {
    return 'Bye!';
};

// 2. Can be anonymous or named
const namedExpr = function myFunc() {
    // 'myFunc' only available inside function
    return 'Named expression';
};

// ============================================
// Arrow Functions (ES6)
// ============================================

// Concise syntax:
const add = (a, b) => a + b; // Implicit return

// Equivalent to:
const add2 = function(a, b) {
    return a + b;
};

// Various syntaxes:
const noParams = () => 'No parameters';
const oneParam = x => x * 2; // Parentheses optional for single param
const twoParams = (x, y) => x + y;
const multiLine = (x, y) => {
    let result = x + y;
    return result; // Explicit return needed with {}
};

// Return object literal (wrap in parentheses!):
const makeObject = (name, age) => ({ name, age });
// Without (): => { name, age } // Interpreted as function body!

// ============================================
// Key Difference: 'this' binding
// ============================================

// Regular function: 'this' depends on how it's called
const obj1 = {
    name: 'Regular',
    regularFunc: function() {
        console.log(this.name); // 'this' refers to obj1
    }
};
obj1.regularFunc(); // "Regular"

// Arrow function: 'this' is lexically bound (inherits from surrounding)
const obj2 = {
    name: 'Arrow',
    arrowFunc: () => {
        console.log(this.name); // 'this' NOT obj2! Inherits from outer scope
    }
};
obj2.arrowFunc(); // undefined (or global object in non-strict mode)

// Practical use: Callbacks
const timer = {
    seconds: 0,
    start: function() {
        // Regular function: 'this' would be wrong inside setInterval
        setInterval(() => {
            this.seconds++; // Arrow function preserves 'this'
            console.log(this.seconds);
        }, 1000);
    }
};

// ============================================
// When to use each:
// ============================================

/*
 * Function Declaration:
 * ✓ Need hoisting
 * ✓ Top-level functions
 * ✓ Need function name for recursion/debugging
 * 
 * Function Expression:
 * ✓ Assign to variables/properties
 * ✓ Prevent hoisting
 * ✓ Conditional function creation
 * 
 * Arrow Function:
 * ✓ Callbacks and higher-order functions
 * ✓ Need lexical 'this'
 * ✓ Short, simple functions
 * ✗ Methods that need 'this'
 * ✗ Constructors (can't use 'new')
 * ✗ Need 'arguments' object
 */
```

### 5.2 Function Parameters

```javascript
// ============================================
// Default Parameters (ES6)
// ============================================

function greet(name = 'Guest', greeting = 'Hello') {
    return `${greeting}, ${name}!`;
}

console.log(greet()); // "Hello, Guest!"
console.log(greet('Alice')); // "Hello, Alice!"
console.log(greet('Bob', 'Hi')); // "Hi, Bob!"

// Default parameter expressions:
function createUser(name, id = Date.now()) {
    return { name, id }; // Different ID each call
}

// Defaults can reference earlier parameters:
function makeFullName(first, last, full = `${first} ${last}`) {
    return full;
}

// Old way (pre-ES6):
function oldGreet(name) {
    name = name || 'Guest'; // Problem: '' and 0 also become 'Guest'
}

// ============================================
// Rest Parameters (...args)
// ============================================

// Gather remaining arguments into array:
function sum(...numbers) {
    return numbers.reduce((total, n) => total + n, 0);
}

console.log(sum(1, 2, 3)); // 6
console.log(sum(1, 2, 3, 4, 5)); // 15

// Combine with regular parameters:
function multiply(multiplier, ...numbers) {
    return numbers.map(n => n * multiplier);
}

console.log(multiply(2, 1, 2, 3)); // [2, 4, 6]

// Rules:
// 1. Must be last parameter
// function bad(a, ...rest, b) {} // SyntaxError!
// 2. Only one rest parameter allowed

// ============================================
// Arguments Object (Old way - avoid in modern code)
// ============================================

function oldSum() {
    // 'arguments' is array-like but NOT an array
    let total = 0;
    for (let i = 0; i < arguments.length; i++) {
        total += arguments[i];
    }
    return total;
}

console.log(oldSum(1, 2, 3)); // 6

// Problems with arguments:
// 1. Not available in arrow functions
// 2. Not a real array (no array methods)
// 3. Less clear than rest parameters

// Converting to array:
function argsToArray() {
    let argsArray = Array.from(arguments); // ES6
    let argsArray2 = [...arguments]; // ES6 spread
    let argsArray3 = Array.prototype.slice.call(arguments); // Old way
}

// ============================================
// Destructuring Parameters
// ============================================

// Object destructuring:
function displayUser({ name, age, city = 'Unknown' }) {
    console.log(`${name}, ${age}, from ${city}`);
}

displayUser({ name: 'Alice', age: 30, city: 'NYC' });
displayUser({ name: 'Bob', age: 25 }); // city = 'Unknown'

// Array destructuring:
function getCoordinates([x, y]) {
    console.log(`X: ${x}, Y: ${y}`);
}

getCoordinates([10, 20]);

// Nested destructuring:
function processUser({ name, address: { city, country } }) {
    console.log(`${name} from ${city}, ${country}`);
}

processUser({
    name: 'Alice',
    address: { city: 'NYC', country: 'USA' }
});
```

### 5.3 Return Values

```javascript
// ============================================
// Return Statement
// ============================================

function add(a, b) {
    return a + b; // Exit function and return value
    console.log('Never executed'); // Unreachable code
}

// No return = undefined:
function noReturn() {
    console.log('Do something');
    // Implicitly returns undefined
}

console.log(noReturn()); // undefined

// Early return (guard clauses):
function divide(a, b) {
    if (b === 0) {
        return 'Error: Division by zero'; // Exit early
    }
    return a / b;
}

// Multiple return points:
function checkAge(age) {
    if (age < 0) return 'Invalid';
    if (age < 18) return 'Minor';
    if (age < 65) return 'Adult';
    return 'Senior';
}

// ============================================
// Returning Objects and Arrays
// ============================================

// Return object:
function createPerson(name, age) {
    return {
        name: name,
        age: age,
        greet: function() {
            return `Hi, I'm ${this.name}`;
        }
    };
}

// With shorthand (ES6):
function createPerson2(name, age) {
    return {
        name, // Same as name: name
        age,
        greet() { // Method shorthand
            return `Hi, I'm ${this.name}`;
        }
    };
}

// Return array:
function getMinMax(numbers) {
    return [Math.min(...numbers), Math.max(...numbers)];
}

let [min, max] = getMinMax([3, 1, 4, 1, 5, 9]);

// ============================================
// Return Functions (Higher-order functions)
// ============================================

function multiplier(factor) {
    return function(number) {
        return number * factor;
    };
}

let double = multiplier(2);
let triple = multiplier(3);

console.log(double(5)); // 10
console.log(triple(5)); // 15

// Arrow function version:
const multiplier2 = factor => number => number * factor;
```

### 5.4 Function as First-Class Citizens

```javascript
// ============================================
// Functions are values - can be:
// 1. Assigned to variables
// 2. Passed as arguments
// 3. Returned from functions
// 4. Stored in data structures
// ============================================

// 1. Assigned to variables:
const myFunc = function() { return 'Hi'; };
const myFunc2 = myFunc; // Copy reference

// 2. Passed as arguments (Callbacks):
function executeCallback(callback) {
    console.log('Before callback');
    callback();
    console.log('After callback');
}

executeCallback(function() {
    console.log('Inside callback');
});

// Common use: Array methods
let numbers = [1, 2, 3, 4, 5];
let doubled = numbers.map(function(n) { return n * 2; });
let doubled2 = numbers.map(n => n * 2); // Arrow function

// 3. Returned from functions:
function createGreeter(greeting) {
    return function(name) {
        return `${greeting}, ${name}!`;
    };
}

let sayHello = createGreeter('Hello');
let sayHi = createGreeter('Hi');

console.log(sayHello('Alice')); // "Hello, Alice!"
console.log(sayHi('Bob')); // "Hi, Bob!"

// 4. Stored in data structures:
const operations = {
    add: (a, b) => a + b,
    subtract: (a, b) => a - b,
    multiply: (a, b) => a * b,
    divide: (a, b) => a / b
};

console.log(operations.add(5, 3)); // 8
console.log(operations.multiply(5, 3)); // 15

const handlers = [
    () => console.log('Handler 1'),
    () => console.log('Handler 2'),
    () => console.log('Handler 3')
];

handlers.forEach(handler => handler());

// ============================================
// Higher-Order Functions
// ============================================

// Function that takes or returns functions

// Example 1: Function that takes a function
function repeat(n, action) {
    for (let i = 0; i < n; i++) {
        action(i);
    }
}

repeat(3, console.log); // Logs: 0, 1, 2

// Example 2: Function that returns a function
function greaterThan(n) {
    return m => m > n;
}

let greaterThan10 = greaterThan(10);
console.log(greaterThan10(11)); // true
console.log(greaterThan10(9)); // false

// Example 3: Function transformation
function unless(test, then) {
    if (!test) then();
}

repeat(5, n => {
    unless(n % 2 === 1, () => {
        console.log(n, 'is even');
    });
});

// ============================================
// Practical Higher-Order Functions
// ============================================

// Debounce: Delay function execution
function debounce(func, delay) {
    let timeoutId;
    return function(...args) {
        clearTimeout(timeoutId);
        timeoutId = setTimeout(() => func(...args), delay);
    };
}

// Usage:
const expensiveOperation = debounce(() => {
    console.log('Executing expensive operation');
}, 1000);

// Throttle: Limit function execution rate
function throttle(func, limit) {
    let inThrottle;
    return function(...args) {
        if (!inThrottle) {
            func(...args);
            inThrottle = true;
            setTimeout(() => inThrottle = false, limit);
        }
    };
}

// Memoization: Cache function results
function memoize(fn) {
    const cache = {};
    return function(...args) {
        const key = JSON.stringify(args);
        if (key in cache) {
            return cache[key];
        }
        const result = fn(...args);
        cache[key] = result;
        return result;
    };
}

const slowFibonacci = memoize(function fib(n) {
    if (n <= 1) return n;
    return fib(n - 1) + fib(n - 2);
});
```

### 5.5 Recursion

```javascript
// ============================================
// Recursive Functions
// ============================================

// Function that calls itself

// Example 1: Factorial
function factorial(n) {
    // Base case: stop recursion
    if (n === 0 || n === 1) {
        return 1;
    }
    // Recursive case
    return n * factorial(n - 1);
}

console.log(factorial(5)); // 120

// Flow of factorial(5):
/*
 * factorial(5)
 * ↓
 * 5 * factorial(4)
 *     ↓
 *     4 * factorial(3)
 *         ↓
 *         3 * factorial(2)
 *             ↓
 *             2 * factorial(1)
 *                 ↓
 *                 return 1 (base case)
 *             return 2 * 1 = 2
 *         return 3 * 2 = 6
 *     return 4 * 6 = 24
 * return 5 * 24 = 120
 */

// Example 2: Fibonacci
function fibonacci(n) {
    if (n <= 1) return n;
    return fibonacci(n - 1) + fibonacci(n - 2);
}

console.log(fibonacci(7)); // 13

// Call tree for fibonacci(5):
/*
 *                    fib(5)
 *                   /      \
 *               fib(4)      fib(3)
 *              /     \      /     \
 *          fib(3)  fib(2) fib(2) fib(1)
 *          /   \    /  \   /  \
 *      fib(2) fib(1) ...
 *      /  \
 *  fib(1) fib(0)
 */

// Example 3: Sum array recursively
function sumArray(arr) {
    if (arr.length === 0) return 0;
    return arr[0] + sumArray(arr.slice(1));
}

console.log(sumArray([1, 2, 3, 4, 5])); // 15

// ============================================
// Recursion vs Iteration
// ============================================

// Recursive approach:
function powerRecursive(base, exponent) {
    if (exponent === 0) return 1;
    return base * powerRecursive(base, exponent - 1);
}

// Iterative approach:
function powerIterative(base, exponent) {
    let result = 1;
    for (let i = 0; i < exponent; i++) {
        result *= base;
    }
    return result;
}

/*
 * Recursion vs Iteration:
 * 
 * Recursion:
 * ✓ More elegant for tree/graph structures
 * ✓ Natural for divide-and-conquer algorithms
 * ✗ Stack overflow risk with deep recursion
 * ✗ More memory usage (call stack)
 * 
 * Iteration:
 * ✓ Better performance (no function call overhead)
 * ✓ No stack overflow risk
 * ✗ Can be more complex for certain problems
 */

// ============================================
// Tail Recursion Optimization
// ============================================

// Non-tail recursive (operations after recursive call):
function factorialNonTail(n) {
    if (n === 1) return 1;
    return n * factorialNonTail(n - 1); // Multiplication after call
}

// Tail recursive (no operations after recursive call):
function factorialTail(n, accumulator = 1) {
    if (n === 1) return accumulator;
    return factorialTail(n - 1, n * accumulator); // Last action is call
}

// Tail recursion can be optimized by compilers to iteration
// (not all JS engines support this yet)

// ============================================
// Common Recursive Patterns
// ============================================

// 1. Tree traversal:
const tree = {
    value: 1,
    children: [
        { value: 2, children: [] },
        { value: 3, children: [
            { value: 4, children: [] }
        ]}
    ]
};

function sumTree(node) {
    let sum = node.value;
    for (let child of node.children) {
        sum += sumTree(child); // Recursive call for each child
    }
    return sum;
}

// 2. Deep clone:
function deepClone(obj) {
    if (obj === null || typeof obj !== 'object') {
        return obj;
    }
    
    if (Array.isArray(obj)) {
        return obj.map(item => deepClone(item));
    }
    
    const cloned = {};
    for (let key in obj) {
        if (obj.hasOwnProperty(key)) {
            cloned[key] = deepClone(obj[key]);
        }
    }
    return cloned;
}

// 3. Flatten nested array:
function flatten(arr) {
    let result = [];
    for (let item of arr) {
        if (Array.isArray(item)) {
            result = result.concat(flatten(item)); // Recursive
        } else {
            result.push(item);
        }
    }
    return result;
}

console.log(flatten([1, [2, [3, 4], 5], 6])); // [1, 2, 3, 4, 5, 6]
```

---

## 6. Scope and Closures

### 6.1 Scope

```javascript
// ============================================
// Global Scope
// ============================================

var globalVar = 'I am global';
let globalLet = 'Also global';
const globalConst = 'Global too';

function test() {
    console.log(globalVar); // Accessible
}

// In browser:
// console.log(window.globalVar); // Accessible
// console.log(window.globalLet); // undefined (let/const not on window)

// ============================================
// Function Scope
// ============================================

function functionScope() {
    var functionVar = 'Function scoped';
    let functionLet = 'Also function scoped';
    
    if (true) {
        var insideIf = 'Still function scoped'; // var ignores blocks
        let blockScoped = 'Block scoped'; // let respects blocks
    }
    
    console.log(insideIf); // ✓ Accessible
    // console.log(blockScoped); // ✗ ReferenceError
}

// console.log(functionVar); // ReferenceError: not accessible outside

// ============================================
// Block Scope (ES6)
// ============================================

if (true) {
    let blockLet = 'Block scoped';
    const blockConst = 'Also block scoped';
    var notBlock = 'NOT block scoped';
}

// console.log(blockLet); // ReferenceError
// console.log(blockConst); // ReferenceError
console.log(notBlock); // ✓ Accessible (var leaks out)

// Loops create block scopes:
for (let i = 0; i < 3; i++) {
    // 'i' is scoped to this loop
    setTimeout(() => console.log(i), 100);
} // Logs: 0, 1, 2

for (var j = 0; j < 3; j++) {
    // 'j' is NOT block scoped
    setTimeout(() => console.log(j), 100);
} // Logs: 3, 3, 3 (j is 3 when timeouts execute)

// ============================================
// Lexical Scope (Static Scope)
// ============================================

// Scope is determined by code structure, not execution context

const outerVar = 'outer';

function outer() {
    const middleVar = 'middle';
    
    function inner() {
        const innerVar = 'inner';
        console.log(innerVar);   // ✓ Own scope
        console.log(middleVar);  // ✓ Parent scope
        console.log(outerVar);   // ✓ Grandparent scope
    }
    
    inner();
    // console.log(innerVar); // ✗ Can't access child scope
}

// Scope chain visualization:
/*
 * Global Scope
 * └── outerVar
 *     └── outer()
 *         └── middleVar
 *             └── inner()
 *                 └── innerVar
 * 
 * Inner function looks up scope chain:
 * innerVar → found in inner()
 * middleVar → not in inner(), found in outer()
 * outerVar → not in inner() or outer(), found in global
 */

// ============================================
// Shadowing
// ============================================

let name = 'Global';

function shadowExample() {
    let name = 'Function'; // Shadows global
    
    if (true) {
        let name = 'Block'; // Shadows function scope
        console.log(name); // 'Block'
    }
    
    console.log(name); // 'Function'
}

console.log(name); // 'Global'

// Shadowing flow:
/*
 * Variable lookup:
 * 1. Check current scope
 * 2. If found → use it (shadowing prevents looking further)
 * 3. If not found → check parent scope
 * 4. Repeat until global scope or variable found
 */
```

### 6.2 Closures

```javascript
// ============================================
// What is a Closure?
// ============================================

/*
 * A closure is a function that has access to variables in its outer scope,
 * even after the outer function has returned.
 * 
 * Every function creates a closure - it "closes over" its surrounding scope.
 */

// Basic closure:
function outer() {
    let count = 0; // Private variable
    
    function inner() {
        count++; // Access outer variable
        console.log(count);
    }
    
    return inner;
}

const counter = outer(); // outer() executes and returns inner()
counter(); // 1 - inner() still has access to count!
counter(); // 2 - count persists between calls
counter(); // 3

// Closure flow:
/*
 * 1. outer() is called:
 *    - count = 0 is created in outer's scope
 *    - inner function is created (captures count in closure)
 *    - inner is returned
 * 2. outer() finishes execution
 *    - Normally, count would be garbage collected
 *    - But inner still references it, so it's preserved!
 * 3. counter() (which is inner) is called:
 *    - Can still access and modify count
 */

// ============================================
// Practical Closure Examples
// ============================================

// 1. Data Privacy (Encapsulation):
function createBankAccount(initialBalance) {
    let balance = initialBalance; // Private variable
    
    return {
        deposit(amount) {
            balance += amount;
            return balance;
        },
        withdraw(amount) {
            if (amount > balance) {
                return 'Insufficient funds';
            }
            balance -= amount;
            return balance;
        },
        getBalance() {
            return balance;
        }
    };
}

const account = createBankAccount(100);
console.log(account.deposit(50)); // 150
console.log(account.withdraw(30)); // 120
console.log(account.getBalance()); // 120
// console.log(account.balance); // undefined - balance is private!

// 2. Function Factories:
function multiplierFactory(multiplier) {
    return function(number) {
        return number * multiplier;
    };
}

const double = multiplierFactory(2);
const triple = multiplierFactory(3);
const quadruple = multiplierFactory(4);

console.log(double(5)); // 10
console.log(triple(5)); // 15
console.log(quadruple(5)); // 20

// Each function "remembers" its own multiplier value!

// 3. Event Handlers with Private State:
function setupButton(buttonId) {
    let clickCount = 0; // Private to this button
    
    const button = document.getElementById(buttonId);
    button.addEventListener('click', function() {
        clickCount++;
        console.log(`Button ${buttonId} clicked ${clickCount} times`);
    });
}

// 4. Module Pattern:
const calculator = (function() {
    // Private variables and functions
    let result = 0;
    
    function log(operation, value) {
        console.log(`${operation}: ${value}`);
    }
    
    // Public API
    return {
        add(num) {
            result += num;
            log('Add', num);
            return this; // For chaining
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

calculator.add(5).add(3).subtract(2);
console.log(calculator.getResult()); // 6

// 5. Iterators:
function makeIterator(array) {
    let index = 0;
    
    return {
        next() {
            return index < array.length ?
                { value: array[index++], done: false } :
                { done: true };
        },
        hasNext() {
            return index < array.length;
        }
    };
}

const iter = makeIterator(['a', 'b', 'c']);
console.log(iter.next()); // { value: 'a', done: false }
console.log(iter.next()); // { value: 'b', done: false }
console.log(iter.next()); // { value: 'c', done: false }
console.log(iter.next()); // { done: true }

// ============================================
// Common Closure Pitfalls
// ============================================

// Pitfall 1: Loop closure problem
function createFunctions() {
    const functions = [];
    
    // Wrong: using var
    for (var i = 0; i < 3; i++) {
        functions.push(function() {
            console.log(i); // All closures share same 'i'
        });
    }
    
    return functions;
}

const funcs = createFunctions();
funcs[0](); // 3 (not 0!)
funcs[1](); // 3 (not 1!)
funcs[2](); // 3 (not 2!)

// Why? All functions close over the same 'i', which ends up as 3

// Solution 1: Use let (block scoped)
function createFunctions2() {
    const functions = [];
    
    for (let i = 0; i < 3; i++) { // let creates new 'i' each iteration
        functions.push(function() {
            console.log(i);
        });
    }
    
    return functions;
}

const funcs2 = createFunctions2();
funcs2[0](); // 0 ✓
funcs2[1](); // 1 ✓
funcs2[2](); // 2 ✓

// Solution 2: IIFE (Immediately Invoked Function Expression)
function createFunctions3() {
    const functions = [];
    
    for (var i = 0; i < 3; i++) {
        (function(j) { // Create new scope with parameter
            functions.push(function() {
                console.log(j);
            });
        })(i); // Pass current i value
    }
    
    return functions;
}

// Pitfall 2: Memory leaks
function createHeavyObject() {
    const heavyData = new Array(1000000).fill('data'); // Large array
    
    return {
        // This closure keeps heavyData in memory
        getData() {
            return heavyData[0];
        }
    };
}

// Solution: Only close over what you need
function createHeavyObject2() {
    const heavyData = new Array(1000000).fill('data');
    const firstItem = heavyData[0]; // Copy only what you need
    
    return {
        getData() {
            return firstItem; // heavyData can be garbage collected
        }
    };
}

// ============================================
// Advanced Closure Patterns
// ============================================

// Partial Application:
function partial(fn, ...fixedArgs) {
    return function(...remainingArgs) {
        return fn(...fixedArgs, ...remainingArgs);
    };
}

function add3(a, b, c) {
    return a + b + c;
}

const add5and = partial(add3, 5); // Fix first argument
console.log(add5and(3, 2)); // 5 + 3 + 2 = 10

// Currying:
function curry(fn) {
    return function curried(...args) {
        if (args.length >= fn.length) {
            return fn(...args);
        }
        return function(...moreArgs) {
            return curried(...args, ...moreArgs);
        };
    };
}

const curriedAdd = curry(add3);
console.log(curriedAdd(1)(2)(3)); // 6
console.log(curriedAdd(1, 2)(3)); // 6
console.log(curriedAdd(1)(2, 3)); // 6

// Memoization with closure:
function memoize(fn) {
    const cache = new Map(); // Private cache
    
    return function(...args) {
        const key = JSON.stringify(args);
        
        if (cache.has(key)) {
            console.log('Cache hit!');
            return cache.get(key);
        }
        
        console.log('Computing...');
        const result = fn(...args);
        cache.set(key, result);
        return result;
    };
}

const expensiveFunction = memoize((n) => {
    let sum = 0;
    for (let i = 0; i < n; i++) {
        sum += i;
    }
    return sum;
});

console.log(expensiveFunction(1000000)); // Computing... (slow)
console.log(expensiveFunction(1000000)); // Cache hit! (instant)
```

### 6.3 The 'this' Keyword

```javascript
// ============================================
// What is 'this'?
// ============================================

/*
 * 'this' refers to the object that is executing the current function.
 * Its value depends on HOW the function is called, not where it's defined.
 */

// ============================================
// Rule 1: Method Invocation
// ============================================

const person = {
    name: 'Alice',
    greet() {
        console.log(`Hello, I'm ${this.name}`);
        // 'this' refers to the object (person)
    }
};

person.greet(); // "Hello, I'm Alice"

// But if we extract the method:
const greetFunction = person.greet;
greetFunction(); // "Hello, I'm undefined" (in strict mode)
// 'this' is now undefined (or global object in non-strict mode)

// ============================================
// Rule 2: Function Invocation
// ============================================

function showThis() {
    console.log(this);
}

showThis(); // undefined (strict mode) or global object (non-strict)

// In browser (non-strict): window
// In Node.js (non-strict): global

// ============================================
// Rule 3: Constructor Invocation
// ============================================

function Person(name) {
    this.name = name; // 'this' refers to new object
    this.greet = function() {
        console.log(`Hi, I'm ${this.name}`);
    };
}

const alice = new Person('Alice');
alice.greet(); // "Hi, I'm Alice"

// Flow with 'new':
/*
 * 1. Create empty object: {}
 * 2. Set 'this' to point to that object
 * 3. Execute constructor function
 * 4. Return 'this' (the new object)
 */

// ============================================
// Rule 4: Explicit Binding (call, apply, bind)
// ============================================

function introduce(greeting, punctuation) {
    console.log(`${greeting}, I'm ${this.name}${punctuation}`);
}

const person1 = { name: 'Alice' };
const person2 = { name: 'Bob' };

// call: invoke immediately with arguments
introduce.call(person1, 'Hello', '!'); // "Hello, I'm Alice!"
introduce.call(person2, 'Hi', '.'); // "Hi, I'm Bob."

// apply: invoke immediately with array of arguments
introduce.apply(person1, ['Greetings', '?']); // "Greetings, I'm Alice?"

// bind: create new function with bound 'this'
const person1Introduce = introduce.bind(person1);
person1Introduce('Hey', '~'); // "Hey, I'm Alice~"

// Difference:
/*
 * call(thisArg, arg1, arg2, ...) - immediate invocation, separate args
 * apply(thisArg, [args]) - immediate invocation, array of args
 * bind(thisArg) - returns new function, doesn't invoke
 */

// ============================================
// Rule 5: Arrow Functions
// ============================================

// Arrow functions don't have their own 'this'
// They inherit 'this' from surrounding scope (lexical this)

const obj = {
    name: 'Regular',
    regular: function() {
        console.log('Regular:', this.name);
    },
    arrow: () => {
        console.log('Arrow:', this.name);
    },
    nested: function() {
        const innerArrow = () => {
            console.log('Inner arrow:', this.name);
        };
        innerArrow();
    }
};

obj.regular(); // "Regular: Regular" (this = obj)
obj.arrow(); // "Arrow: undefined" (this = global/undefined)
obj.nested(); // "Inner arrow: Regular" (inherits from nested())

// Practical use case:
const timer = {
    seconds: 0,
    start: function() {
        setInterval(() => {
            this.seconds++; // Arrow function preserves 'this'
            console.log(this.seconds);
        }, 1000);
    }
};

// If we used regular function:
const brokenTimer = {
    seconds: 0,
    start: function() {
        setInterval(function() {
            this.seconds++; // 'this' is NOT brokenTimer!
            console.log(this.seconds); // NaN
        }, 1000);
    }
};

// Old solution (pre-ES6):
const oldTimer = {
    seconds: 0,
    start: function() {
        const self = this; // Capture 'this' in variable
        setInterval(function() {
            self.seconds++; // Use captured 'this'
            console.log(self.seconds);
        }, 1000);
    }
};

// ============================================
// 'this' Decision Tree
// ============================================

/*
 * How to determine 'this':
 * 
 * 1. Arrow function? → lexical 'this' (from surrounding scope)
 * 2. Called with 'new'? → newly created object
 * 3. Called with call/apply/bind? → explicitly bound object
 * 4. Called as method (obj.method())? → obj
 * 5. Otherwise → undefined (strict) or global object (non-strict)
 */

// ============================================
// Common 'this' Pitfalls
// ============================================

// Pitfall 1: Losing 'this' in callbacks
class Button {
    constructor(label) {
        this.label = label;
    }
    
    // Wrong:
    clickWrong() {
        document.getElementById('btn').addEventListener('click', function() {
            console.log(this.label); // undefined! 'this' is the button element
        });
    }
    
    // Right - Arrow function:
    clickRight() {
        document.getElementById('btn').addEventListener('click', () => {
            console.log(this.label); // Correct! Lexical 'this'
        });
    }
    
    // Right - Bind:
    clickBind() {
        document.getElementById('btn').addEventListener('click', function() {
            console.log(this.label);
        }.bind(this));
    }
}

// Pitfall 2: Nested method calls
const calculator = {
    value: 0,
    add(n) {
        this.value += n;
        return this; // Return 'this' for chaining
    },
    multiply(n) {
        this.value *= n;
        return this;
    }
};

calculator.add(5).multiply(2); // Works! 'this' is maintained
