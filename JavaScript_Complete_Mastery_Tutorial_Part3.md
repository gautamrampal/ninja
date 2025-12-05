# JavaScript Complete Mastery Tutorial - Part 3
## Advanced Concepts

---

## 9. Advanced Functions

### 9.1 IIFE (Immediately Invoked Function Expression)

```javascript
// ============================================
// IIFE Pattern
// ============================================

// Basic IIFE:
(function() {
    console.log('I run immediately!');
})();

// With parameters:
(function(name) {
    console.log(`Hello, ${name}!`);
})('Alice');

// Arrow function IIFE:
(() => {
    console.log('Arrow IIFE');
})();

// ============================================
// Why use IIFE?
// ============================================

// 1. Create private scope (pre-ES6):
var privateModule = (function() {
    // Private variables
    var privateVar = 'I am private';
    
    // Public API
    return {
        getPrivate: function() {
            return privateVar;
        },
        setPrivate: function(value) {
            privateVar = value;
        }
    };
})();

console.log(privateModule.getPrivate()); // 'I am private'
// console.log(privateVar); // ReferenceError

// 2. Avoid polluting global scope:
(function() {
    var tempVar = 'Not global';
    // Do work...
})();
// tempVar doesn't exist in global scope

// 3. Async IIFE (modern use case):
(async function() {
    const data = await fetch('/api/data');
    console.log(data);
})();
```

### 9.2 Call, Apply, and Bind

```javascript
// ============================================
// call() - Invoke with specified 'this' and arguments
// ============================================

function greet(greeting, punctuation) {
    return `${greeting}, ${this.name}${punctuation}`;
}

const person1 = { name: 'Alice' };
const person2 = { name: 'Bob' };

console.log(greet.call(person1, 'Hello', '!')); // \"Hello, Alice!\"
console.log(greet.call(person2, 'Hi', '.')); // \"Hi, Bob.\"

// Borrowing methods:
const obj1 = {
    name: 'Object 1',
    greet: function() {
        return `Hello from ${this.name}`;
    }
};

const obj2 = { name: 'Object 2' };
console.log(obj1.greet.call(obj2)); // \"Hello from Object 2\"

// ============================================
// apply() - Like call, but takes array of arguments
// ============================================

console.log(greet.apply(person1, ['Greetings', '?'])); // \"Greetings, Alice?\"

// Use case: Math.max with array
const numbers = [1, 5, 3, 9, 2];
const max = Math.max.apply(null, numbers);
console.log(max); // 9

// Modern way (spread):
console.log(Math.max(...numbers)); // 9

// ============================================
// bind() - Create new function with bound 'this'
// ============================================

const person1Greet = greet.bind(person1);
console.log(person1Greet('Hey', '~')); // \"Hey, Alice~\"

// Partial application with bind:
const person1Hello = greet.bind(person1, 'Hello');
console.log(person1Hello('!')); // \"Hello, Alice!\"
console.log(person1Hello('?')); // \"Hello, Alice?\"

// Use case: Event handlers
const button = {
    content: 'Click me',
    click: function() {
        console.log(`Button says: ${this.content}`);
    }
};

// Without bind (loses 'this'):
// document.getElementById('btn').addEventListener('click', button.click);

// With bind (preserves 'this'):
document.getElementById('btn').addEventListener('click', button.click.bind(button));

// ============================================
// Comparison
// ============================================

/*
 * Method | Invokes immediately | Arguments format | Returns
 * ----------------------------------------------------------------
 * call   | Yes                 | Comma-separated  | Function result
 * apply  | Yes                 | Array            | Function result
 * bind   | No                  | Comma-separated  | New function
 */

// ============================================
// Implementing bind()
// ============================================

Function.prototype.myBind = function(context, ...fixedArgs) {
    const fn = this;
    return function(...args) {
        return fn.apply(context, [...fixedArgs, ...args]);
    };
};

const myBoundFunc = greet.myBind(person1, 'Custom');
console.log(myBoundFunc('!!!')); // \"Custom, Alice!!!\"
```

---

## 10. Prototypes and Inheritance

### 10.1 Prototypes

```javascript
// ============================================
// Prototype Chain
// ============================================

/*
 * Every object in JavaScript has an internal [[Prototype]] property
 * that references another object (or null)
 * 
 * When accessing a property:
 * 1. Check the object itself
 * 2. If not found, check its prototype
 * 3. If not found, check prototype's prototype
 * 4. Continue until null (end of chain)
 */

// Constructor function:
function Person(name, age) {
    this.name = name;
    this.age = age;
}

// Add method to prototype:
Person.prototype.greet = function() {
    return `Hi, I'm ${this.name}`;
};

const alice = new Person('Alice', 30);
const bob = new Person('Bob', 25);

console.log(alice.greet()); // \"Hi, I'm Alice\"
console.log(bob.greet()); // \"Hi, I'm Bob\"

// Both share the same method:
console.log(alice.greet === bob.greet); // true

// Prototype chain:
/*
 * alice
 * \u2502
 * \u251c\u2500 name: 'Alice'
 * \u2514\u2500 age: 30
 *    \u2502
 *    \u2514\u2500\u2500> [[Prototype]]: Person.prototype
 *          \u2502
 *          \u251c\u2500 greet: function
 *          \u2502
 *          \u2514\u2500\u2500> [[Prototype]]: Object.prototype
 *                \u2502
 *                \u251c\u2500 toString: function
 *                \u251c\u2500 valueOf: function
 *                \u251c\u2500 ...
 *                \u2502
 *                \u2514\u2500\u2500> [[Prototype]]: null
 */

// Accessing prototype:
console.log(Object.getPrototypeOf(alice) === Person.prototype); // true
console.log(alice.__proto__ === Person.prototype); // true (deprecated)

// Check prototype chain:
console.log(alice instanceof Person); // true
console.log(alice instanceof Object); // true

// ============================================
// Own Properties vs Prototype Properties
// ============================================

console.log(alice.hasOwnProperty('name')); // true (own property)
console.log(alice.hasOwnProperty('greet')); // false (on prototype)

// for...in loops through own and inherited properties:
for (let key in alice) {
    console.log(key); // name, age, greet
}

// Only own properties:
for (let key in alice) {
    if (alice.hasOwnProperty(key)) {
        console.log(key); // name, age
    }
}

// Modern way - Object.keys() (only own properties):
console.log(Object.keys(alice)); // ['name', 'age']

// ============================================
// Modifying Prototypes
// ============================================

// Add property to prototype:
Person.prototype.species = 'Homo sapiens';
console.log(alice.species); // 'Homo sapiens'
console.log(bob.species); // 'Homo sapiens'

// Shadowing (override prototype property):
alice.species = 'Custom';
console.log(alice.species); // 'Custom' (own property)
console.log(bob.species); // 'Homo sapiens' (prototype property)

// ============================================
// Object.create() - Create with specific prototype
// ============================================

const personProto = {
    greet() {
        return `Hello, I'm ${this.name}`;
    }
};

const charlie = Object.create(personProto);
charlie.name = 'Charlie';
console.log(charlie.greet()); // \"Hello, I'm Charlie\"

// Create with null prototype (no inheritance):
const bareObject = Object.create(null);
bareObject.name = 'Bare';
console.log(bareObject.toString); // undefined (no Object.prototype)

// ============================================
// Prototype Inheritance Pattern
// ============================================

function Animal(name) {
    this.name = name;
}

Animal.prototype.eat = function() {
    return `${this.name} is eating`;
};

function Dog(name, breed) {
    Animal.call(this, name); // Call parent constructor
    this.breed = breed;
}

// Set up prototype chain:
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog; // Fix constructor reference

Dog.prototype.bark = function() {
    return `${this.name} says woof!`;
};

const dog = new Dog('Rex', 'Labrador');
console.log(dog.eat()); // \"Rex is eating\" (from Animal)
console.log(dog.bark()); // \"Rex says woof!\" (from Dog)
console.log(dog instanceof Dog); // true
console.log(dog instanceof Animal); // true
```

---

## 11. Classes (ES6+)

### 11.1 Class Basics

```javascript
// ============================================
// Class Declaration
// ============================================

class Person {
    // Constructor:
    constructor(name, age) {
        this.name = name;
        this.age = age;
    }
    
    // Method:
    greet() {
        return `Hello, I'm ${this.name}`;
    }
    
    // Another method:
    getAge() {
        return this.age;
    }
}

const alice = new Person('Alice', 30);
console.log(alice.greet()); // \"Hello, I'm Alice\"

// Classes are just syntactic sugar over prototypes:
console.log(typeof Person); // 'function'
console.log(Person.prototype.greet); // function

// ============================================
// Class Expression
// ============================================

const PersonClass = class {
    constructor(name) {
        this.name = name;
    }
};

// Named class expression:
const PersonClass2 = class Person {
    constructor(name) {
        this.name = name;
    }
};

// ============================================
// Getters and Setters
// ============================================

class Rectangle {
    constructor(width, height) {
        this._width = width;
        this._height = height;
    }
    
    // Getter:
    get area() {
        return this._width * this._height;
    }
    
    // Getter:
    get width() {
        return this._width;
    }
    
    // Setter with validation:
    set width(value) {
        if (value <= 0) {
            throw new Error('Width must be positive');
        }
        this._width = value;
    }
}

const rect = new Rectangle(10, 5);
console.log(rect.area); // 50 (called without ())
rect.width = 20; // Setter
console.log(rect.area); // 100

// ============================================
// Static Methods and Properties
// ============================================

class MathUtils {
    static PI = 3.14159; // Static property
    
    static add(a, b) {
        return a + b;
    }
    
    static multiply(a, b) {
        return a * b;
    }
}

// Call on class, not instance:
console.log(MathUtils.add(5, 3)); // 8
console.log(MathUtils.PI); // 3.14159

// const utils = new MathUtils();
// utils.add(5, 3); // TypeError! Not available on instance

// Use case: Factory methods
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

// ============================================
// Private Fields and Methods (ES2022)
// ============================================

class BankAccount {
    // Private field (prefixed with #):
    #balance = 0;
    #transactions = [];
    
    constructor(initialBalance) {
        this.#balance = initialBalance;
    }
    
    // Private method:
    #recordTransaction(amount) {
        this.#transactions.push({
            amount,
            date: new Date()
        });
    }
    
    // Public method:
    deposit(amount) {
        this.#balance += amount;
        this.#recordTransaction(amount);
        return this.#balance;
    }
    
    withdraw(amount) {
        if (amount > this.#balance) {
            throw new Error('Insufficient funds');
        }
        this.#balance -= amount;
        this.#recordTransaction(-amount);
        return this.#balance;
    }
    
    getBalance() {
        return this.#balance;
    }
}

const account = new BankAccount(1000);
account.deposit(500);
console.log(account.getBalance()); // 1500
// console.log(account.#balance); // SyntaxError! Private field
```

### 11.2 Inheritance

```javascript
// ============================================
// Class Inheritance
// ============================================

class Animal {
    constructor(name) {
        this.name = name;
    }
    
    eat() {
        return `${this.name} is eating`;
    }
    
    sleep() {
        return `${this.name} is sleeping`;
    }
}

class Dog extends Animal {
    constructor(name, breed) {
        super(name); // Call parent constructor
        this.breed = breed;
    }
    
    bark() {
        return `${this.name} says woof!`;
    }
    
    // Override parent method:
    eat() {
        return `${this.name} the ${this.breed} is eating`;
    }
}

const dog = new Dog('Rex', 'Labrador');
console.log(dog.bark()); // \"Rex says woof!\"
console.log(dog.eat()); // \"Rex the Labrador is eating\" (overridden)
console.log(dog.sleep()); // \"Rex is sleeping\" (inherited)

// ============================================
// Super keyword
// ============================================

class Cat extends Animal {
    constructor(name, color) {
        super(name); // Must call super first!
        this.color = color;
    }
    
    eat() {
        // Call parent method:
        const parentEat = super.eat();
        return `${parentEat} quietly`;
    }
}

const cat = new Cat('Whiskers', 'orange');
console.log(cat.eat()); // \"Whiskers is eating quietly\"

// ============================================
// Inheritance Chain
// ============================================

class Puppy extends Dog {
    constructor(name, breed, age) {
        super(name, breed);
        this.age = age;
    }
    
    play() {
        return `${this.name} is playing`;
    }
}

const puppy = new Puppy('Max', 'Beagle', 1);
console.log(puppy.play()); // Own method
console.log(puppy.bark()); // From Dog
console.log(puppy.sleep()); // From Animal

console.log(puppy instanceof Puppy); // true
console.log(puppy instanceof Dog); // true
console.log(puppy instanceof Animal); // true
console.log(puppy instanceof Object); // true

// ============================================
// Static Methods Inheritance
// ============================================

class Parent {
    static staticMethod() {
        return 'Parent static';
    }
}

class Child extends Parent {
    static staticMethod() {
        // Call parent static method:
        return super.staticMethod() + ' + Child static';
    }
}

console.log(Child.staticMethod()); // \"Parent static + Child static\"

// ============================================
// Extending Built-in Classes
// ============================================

class MyArray extends Array {
    // Add custom method:
    first() {
        return this[0];
    }
    
    last() {
        return this[this.length - 1];
    }
}

const arr = new MyArray(1, 2, 3, 4, 5);
console.log(arr.first()); // 1
console.log(arr.last()); // 5
console.log(arr.map(x => x * 2)); // Still returns MyArray
```

---

## 12. Asynchronous JavaScript

### 12.1 Callbacks

```javascript
// ============================================
// Callback Pattern
// ============================================

// Synchronous callback:
function greet(name, callback) {
    const message = `Hello, ${name}`;
    callback(message);
}

greet('Alice', (msg) => {
    console.log(msg); // \"Hello, Alice\"
});

// Asynchronous callback:
console.log('Start');

setTimeout(() => {
    console.log('Delayed message');
}, 1000);

console.log('End');
// Output: Start, End, Delayed message

// ============================================
// Callback Hell
// ============================================

// Simulated async operations:
function step1(callback) {
    setTimeout(() => {
        console.log('Step 1 complete');
        callback();
    }, 1000);
}

function step2(callback) {
    setTimeout(() => {
        console.log('Step 2 complete');
        callback();
    }, 1000);
}

function step3(callback) {
    setTimeout(() => {
        console.log('Step 3 complete');
        callback();
    }, 1000);
}

// Nested callbacks (callback hell):
step1(() => {
    step2(() => {
        step3(() => {
            console.log('All steps complete');
        });
    });
});

// Problems:
// 1. Hard to read (pyramid of doom)
// 2. Hard to handle errors
// 3. Hard to modify flow
```

### 12.2 Promises

```javascript
// ============================================
// Creating Promises
// ============================================

// Promise constructor:
const promise = new Promise((resolve, reject) => {
    // Async operation
    const success = true;
    
    setTimeout(() => {
        if (success) {
            resolve('Operation successful!');
        } else {
            reject('Operation failed!');
        }
    }, 1000);
});

// Promise states:
/*
 * 1. Pending: Initial state
 * 2. Fulfilled: Operation completed successfully (resolve called)
 * 3. Rejected: Operation failed (reject called)
 */

// ============================================
// Consuming Promises
// ============================================

promise
    .then(result => {
        console.log(result); // \"Operation successful!\"
        return 'Next value';
    })
    .then(value => {
        console.log(value); // \"Next value\"
    })
    .catch(error => {
        console.error(error); // Called if rejected
    })
    .finally(() => {
        console.log('Cleanup'); // Always called
    });

// ============================================
// Promise Chaining
// ============================================

function asyncOperation(value, delay) {
    return new Promise((resolve) => {
        setTimeout(() => {
            console.log(`Processed: ${value}`);
            resolve(value * 2);
        }, delay);
    });
}

asyncOperation(1, 1000)
    .then(result => asyncOperation(result, 1000))
    .then(result => asyncOperation(result, 1000))
    .then(result => {
        console.log(`Final result: ${result}`); // 8
    });

// Flow:
/*
 * 1. asyncOperation(1) \u2192 returns Promise \u2192 resolves with 2
 * 2. asyncOperation(2) \u2192 returns Promise \u2192 resolves with 4
 * 3. asyncOperation(4) \u2192 returns Promise \u2192 resolves with 8
 * 4. Final then receives 8
 */

// ============================================
// Error Handling
// ============================================

function mightFail(shouldFail) {
    return new Promise((resolve, reject) => {
        if (shouldFail) {
            reject(new Error('Something went wrong'));
        } else {
            resolve('Success');
        }
    });
}

mightFail(false)
    .then(result => {
        console.log(result);
        return mightFail(true); // This will fail
    })
    .then(result => {
        console.log('This won\\'t run');
    })
    .catch(error => {
        console.error('Caught error:', error.message);
        return 'Recovered'; // Can recover from error
    })
    .then(result => {
        console.log(result); // \"Recovered\"
    });

// ============================================
// Promise.all() - Wait for all promises
// ============================================

const promise1 = Promise.resolve(1);
const promise2 = Promise.resolve(2);
const promise3 = Promise.resolve(3);

Promise.all([promise1, promise2, promise3])
    .then(results => {
        console.log(results); // [1, 2, 3]
    });

// If any promise rejects, Promise.all rejects:
Promise.all([
    Promise.resolve(1),
    Promise.reject('Error'),
    Promise.resolve(3)
])
    .then(results => {
        console.log('Won\\'t run');
    })
    .catch(error => {
        console.log(error); // \"Error\"
    });

// Use case: Multiple parallel requests:
Promise.all([
    fetch('/api/users'),
    fetch('/api/posts'),
    fetch('/api/comments')
])
    .then(([users, posts, comments]) => {
        console.log('All data loaded');
    });

// ============================================
// Promise.race() - First to settle wins
// ============================================

const slow = new Promise(resolve => setTimeout(() => resolve('slow'), 2000));
const fast = new Promise(resolve => setTimeout(() => resolve('fast'), 1000));

Promise.race([slow, fast])
    .then(result => {
        console.log(result); // \"fast\"
    });

// Use case: Timeout pattern:
function withTimeout(promise, ms) {
    const timeout = new Promise((_, reject) => {
        setTimeout(() => reject(new Error('Timeout')), ms);
    });
    return Promise.race([promise, timeout]);
}

// ============================================
// Promise.allSettled() - Wait for all, don't fail
// ============================================

Promise.allSettled([
    Promise.resolve(1),
    Promise.reject('Error'),
    Promise.resolve(3)
])
    .then(results => {
        console.log(results);
        /*
        [
            { status: 'fulfilled', value: 1 },
            { status: 'rejected', reason: 'Error' },
            { status: 'fulfilled', value: 3 }
        ]
        */
    });

// ============================================
// Promise.any() - First to fulfill wins
// ============================================

Promise.any([
    Promise.reject('Error 1'),
    Promise.resolve('Success'),
    Promise.reject('Error 2')
])
    .then(result => {
        console.log(result); // \"Success\"
    });

// ============================================
// Creating Resolved/Rejected Promises
// ============================================

const resolvedPromise = Promise.resolve('Already resolved');
const rejectedPromise = Promise.reject('Already rejected');

// Useful for consistent async interface:
function getData(cached) {
    if (cached) {
        return Promise.resolve(cachedData);
    }
    return fetch('/api/data');
}
```

### 12.3 Async/Await

```javascript
// ============================================
// Async Functions
// ============================================

// async keyword makes function return a Promise:
async function myAsyncFunction() {
    return 'Hello';
}

myAsyncFunction().then(result => {
    console.log(result); // \"Hello\"
});

// Equivalent to:
function myPromiseFunction() {
    return Promise.resolve('Hello');
}

// ============================================
// Await keyword
// ============================================

// await pauses execution until Promise settles:
async function fetchData() {
    console.log('Fetching...');
    
    const result = await new Promise(resolve => {
        setTimeout(() => resolve('Data'), 1000);
    });
    
    console.log(result); // \"Data\" (after 1 second)
    return result;
}

fetchData();

// ============================================
// Sequential vs Parallel
// ============================================

// Sequential (waits for each):
async function sequential() {
    const result1 = await asyncOperation(1, 1000); // Wait 1s
    const result2 = await asyncOperation(2, 1000); // Wait 1s
    const result3 = await asyncOperation(3, 1000); // Wait 1s
    return [result1, result2, result3];
    // Total time: ~3 seconds
}

// Parallel (all at once):
async function parallel() {
    const [result1, result2, result3] = await Promise.all([
        asyncOperation(1, 1000),
        asyncOperation(2, 1000),
        asyncOperation(3, 1000)
    ]);
    return [result1, result2, result3];
    // Total time: ~1 second
}

// ============================================
// Error Handling
// ============================================

async function handleErrors() {
    try {
        const result = await mightFail(true);
        console.log(result);
    } catch (error) {
        console.error('Error:', error.message);
    } finally {
        console.log('Cleanup');
    }
}

// Multiple operations:
async function multipleOperations() {
    try {
        const user = await fetchUser();
        const posts = await fetchPosts(user.id);
        const comments = await fetchComments(posts[0].id);
        return comments;
    } catch (error) {
        console.error('Failed:', error);
        throw error; // Re-throw if needed
    }
}

// ============================================
// Async Patterns
// ============================================

// 1. Retry pattern:
async function retry(fn, maxRetries = 3) {
    for (let i = 0; i < maxRetries; i++) {
        try {
            return await fn();
        } catch (error) {
            if (i === maxRetries - 1) throw error;
            console.log(`Retry ${i + 1}/${maxRetries}`);
        }
    }
}

// 2. Timeout:
async function timeout(promise, ms) {
    return Promise.race([
        promise,
        new Promise((_, reject) => 
            setTimeout(() => reject(new Error('Timeout')), ms)
        )
    ]);
}

// 3. Sequential processing:
async function processSequentially(items) {
    const results = [];
    for (const item of items) {
        const result = await processItem(item);
        results.push(result);
    }
    return results;
}

// 4. Parallel processing with concurrency limit:
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
// Async Iteration
// ============================================

// for await...of loop:
async function processStream(stream) {
    for await (const chunk of stream) {
        console.log(chunk);
    }
}

// Async generator:
async function* asyncGenerator() {
    yield await Promise.resolve(1);
    yield await Promise.resolve(2);
    yield await Promise.resolve(3);
}

(async () => {
    for await (const value of asyncGenerator()) {
        console.log(value); // 1, 2, 3
    }
})();
```

---

*Continued in Part 4 with more advanced topics including error handling, modules, design patterns, and modern JavaScript features...*
