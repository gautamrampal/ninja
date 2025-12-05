# JavaScript Complete Mastery Tutorial - Part 2
## Continuation from Part 1

---

## 7. Objects

### 7.1 Object Basics

```javascript
// ============================================
// Creating Objects
// ============================================

// 1. Object literal (most common):
const person = {
    name: 'Alice',
    age: 30,
    city: 'NYC'
};

// 2. new Object():
const person2 = new Object();
person2.name = 'Bob';
person2.age = 25;

// 3. Object.create():
const person3 = Object.create(null); // No prototype
person3.name = 'Charlie';

// 4. Constructor function:
function Person(name, age) {
    this.name = name;
    this.age = age;
}
const person4 = new Person('David', 35);

// 5. Class (ES6):
class PersonClass {
    constructor(name, age) {
        this.name = name;
        this.age = age;
    }
}
const person5 = new PersonClass('Eve', 28);

// ============================================
// Accessing Properties
// ============================================

const user = {
    name: 'Alice',
    age: 30,
    'favorite color': 'blue' // Key with space
};

// Dot notation:
console.log(user.name); // 'Alice'
user.age = 31; // Modify

// Bracket notation:
console.log(user['name']); // 'Alice'
console.log(user['favorite color']); // Must use brackets for spaces

// Dynamic property access:
const prop = 'age';
console.log(user[prop]); // 30

// Computed property names (ES6):
const dynamicKey = 'email';
const user2 = {
    name: 'Bob',
    [dynamicKey]: 'bob@email.com', // Dynamic key
    ['is' + 'Admin']: true // Expression as key
};
console.log(user2.email); // 'bob@email.com'
console.log(user2.isAdmin); // true

// ============================================
// Property Shorthand (ES6)
// ============================================

const name = 'Alice';
const age = 30;

// Old way:
const person6 = {
    name: name,
    age: age
};

// Shorthand:
const person7 = {
    name, // Same as name: name
    age   // Same as age: age
};

// ============================================
// Method Shorthand (ES6)
// ============================================

const calculator = {
    // Old way:
    add: function(a, b) {
        return a + b;
    },
    
    // Shorthand:
    subtract(a, b) {
        return a - b;
    },
    
    // Getter:
    get pi() {
        return 3.14159;
    },
    
    // Setter:
    set value(val) {
        this._value = val;
    }
};

console.log(calculator.add(5, 3)); // 8
console.log(calculator.pi); // 3.14159 (called without ())

// ============================================
// Adding/Deleting Properties
// ============================================

const obj = { a: 1 };

// Add:
obj.b = 2;
obj['c'] = 3;

// Delete:
delete obj.a;
console.log(obj); // { b: 2, c: 3 }

// Check if property exists:
console.log('b' in obj); // true
console.log('a' in obj); // false
console.log(obj.hasOwnProperty('b')); // true

// ============================================
// Object Iteration
// ============================================

const person8 = {
    name: 'Alice',
    age: 30,
    city: 'NYC'
};

// 1. for...in loop:
for (let key in person8) {
    console.log(`${key}: ${person8[key]}`);
}

// 2. Object.keys():
Object.keys(person8).forEach(key => {
    console.log(`${key}: ${person8[key]}`);
});

// 3. Object.values():
Object.values(person8).forEach(value => {
    console.log(value);
});

// 4. Object.entries():
Object.entries(person8).forEach(([key, value]) => {
    console.log(`${key}: ${value}`);
});

// Convert back to object:
const entries = [['name', 'Bob'], ['age', 25]];
const obj2 = Object.fromEntries(entries);
```

### 7.2 Object Methods and Features

```javascript
// ============================================
// Object.assign() - Copy/Merge objects
// ============================================

const target = { a: 1, b: 2 };
const source1 = { b: 3, c: 4 };
const source2 = { c: 5, d: 6 };

// Merge into target (mutates target):
Object.assign(target, source1, source2);
console.log(target); // { a: 1, b: 3, c: 5, d: 6 }

// Clone without mutation:
const original = { a: 1, b: 2 };
const clone = Object.assign({}, original);

// Modern way - Spread operator (ES2018):
const clone2 = { ...original };
const merged = { ...source1, ...source2 };

// ============================================
// Object.freeze() - Prevent modifications
// ============================================

const frozen = Object.freeze({ a: 1, b: 2 });
frozen.a = 999; // Ignored in strict mode, error in strict mode
frozen.c = 3; // Can't add properties
delete frozen.b; // Can't delete
console.log(frozen); // { a: 1, b: 2 } unchanged

// Check if frozen:
console.log(Object.isFrozen(frozen)); // true

// Shallow freeze (nested objects not frozen):
const nested = Object.freeze({
    a: 1,
    obj: { b: 2 }
});
nested.obj.b = 999; // Works! Nested object not frozen
console.log(nested.obj.b); // 999

// Deep freeze function:
function deepFreeze(obj) {
    Object.freeze(obj);
    Object.values(obj).forEach(value => {
        if (typeof value === 'object' && value !== null) {
            deepFreeze(value);
        }
    });
    return obj;
}

// ============================================
// Object.seal() - Prevent add/delete, allow modify
// ============================================

const sealed = Object.seal({ a: 1, b: 2 });
sealed.a = 999; // ✓ Can modify
sealed.c = 3; // ✗ Can't add
delete sealed.b; // ✗ Can't delete
console.log(sealed); // { a: 999, b: 2 }

// ============================================
// Object.preventExtensions() - Prevent add only
// ============================================

const obj3 = Object.preventExtensions({ a: 1 });
obj3.a = 999; // ✓ Can modify
obj3.b = 2; // ✗ Can't add
delete obj3.a; // ✓ Can delete
console.log(Object.isExtensible(obj3)); // false

// ============================================
// Summary of Object Methods
// ============================================

/*
 * Method              | Add | Delete | Modify | Check
 * --------------------------------------------------------
 * freeze()            | ✗   | ✗      | ✗      | isFrozen()
 * seal()              | ✗   | ✗      | ✓      | isSealed()
 * preventExtensions() | ✗   | ✓      | ✓      | isExtensible()
 */

// ============================================
// Object Descriptors
// ============================================

// Properties have hidden attributes (descriptors):
const obj4 = { name: 'Alice' };

// Get descriptor:
const descriptor = Object.getOwnPropertyDescriptor(obj4, 'name');
console.log(descriptor);
/*
{
    value: 'Alice',
    writable: true,      // Can be reassigned
    enumerable: true,    // Shows up in for...in
    configurable: true   // Can be deleted/reconfigured
}
*/

// Define property with custom descriptor:
Object.defineProperty(obj4, 'age', {
    value: 30,
    writable: false,     // Can't modify
    enumerable: false,   // Won't show in for...in
    configurable: false  // Can't delete or reconfigure
});

obj4.age = 999; // Ignored (writable: false)
console.log(obj4.age); // 30
console.log(Object.keys(obj4)); // ['name'] - age not enumerable
delete obj4.age; // Fails (configurable: false)

// Define multiple properties:
Object.defineProperties(obj4, {
    city: {
        value: 'NYC',
        writable: true
    },
    country: {
        value: 'USA',
        writable: false
    }
});

// ============================================
// Getters and Setters
// ============================================

const person9 = {
    firstName: 'Alice',
    lastName: 'Smith',
    
    // Getter:
    get fullName() {
        return `${this.firstName} ${this.lastName}`;
    },
    
    // Setter:
    set fullName(name) {
        const parts = name.split(' ');
        this.firstName = parts[0];
        this.lastName = parts[1];
    }
};

console.log(person9.fullName); // \"Alice Smith\" (getter)
person9.fullName = 'Bob Jones'; // (setter)
console.log(person9.firstName); // \"Bob\"
console.log(person9.lastName); // \"Jones\"

// Getters/setters with defineProperty:
Object.defineProperty(person9, 'age', {
    get() {
        return this._age;
    },
    set(value) {
        if (value < 0) {
            throw new Error('Age cannot be negative');
        }
        this._age = value;
    }
});

person9.age = 30; // Sets _age
console.log(person9.age); // Gets _age

// ============================================
// Object References and Comparison
// ============================================

// Objects are compared by reference, not value:
const obj5 = { a: 1 };
const obj6 = { a: 1 };
const obj7 = obj5;

console.log(obj5 === obj6); // false (different objects)
console.log(obj5 === obj7); // true (same reference)

// Shallow comparison:
function shallowEqual(obj1, obj2) {
    const keys1 = Object.keys(obj1);
    const keys2 = Object.keys(obj2);
    
    if (keys1.length !== keys2.length) return false;
    
    for (let key of keys1) {
        if (obj1[key] !== obj2[key]) return false;
    }
    
    return true;
}

console.log(shallowEqual({ a: 1, b: 2 }, { a: 1, b: 2 })); // true

// Deep comparison (recursive):
function deepEqual(obj1, obj2) {
    if (obj1 === obj2) return true;
    
    if (typeof obj1 !== 'object' || typeof obj2 !== 'object' ||
        obj1 === null || obj2 === null) {
        return false;
    }
    
    const keys1 = Object.keys(obj1);
    const keys2 = Object.keys(obj2);
    
    if (keys1.length !== keys2.length) return false;
    
    for (let key of keys1) {
        if (!deepEqual(obj1[key], obj2[key])) return false;
    }
    
    return true;
}
```

### 7.3 Object Destructuring

```javascript
// ============================================
// Basic Destructuring
// ============================================

const person = {
    name: 'Alice',
    age: 30,
    city: 'NYC',
    country: 'USA'
};

// Extract properties:
const { name, age } = person;
console.log(name); // 'Alice'
console.log(age); // 30

// Rename variables:
const { name: fullName, age: years } = person;
console.log(fullName); // 'Alice'
console.log(years); // 30

// Default values:
const { name: n, job = 'Unknown' } = person;
console.log(job); // 'Unknown' (not in object)

// Rest pattern:
const { name: n2, ...rest } = person;
console.log(rest); // { age: 30, city: 'NYC', country: 'USA' }

// ============================================
// Nested Destructuring
// ============================================

const user = {
    id: 1,
    name: 'Alice',
    address: {
        street: '123 Main St',
        city: 'NYC',
        coordinates: {
            lat: 40.7128,
            lng: -74.0060
        }
    }
};

// Destructure nested objects:
const {
    name: userName,
    address: {
        city: userCity,
        coordinates: { lat, lng }
    }
} = user;

console.log(userName); // 'Alice'
console.log(userCity); // 'NYC'
console.log(lat, lng); // 40.7128 -74.0060

// ============================================
// Function Parameter Destructuring
// ============================================

// Instead of:
function displayUser(user) {
    console.log(user.name, user.age);
}

// Use destructuring:
function displayUser2({ name, age, city = 'Unknown' }) {
    console.log(name, age, city);
}

displayUser2({ name: 'Alice', age: 30 }); // Alice 30 Unknown

// With default parameter:
function configure({ timeout = 5000, retries = 3 } = {}) {
    console.log(timeout, retries);
}

configure({ timeout: 10000 }); // 10000 3
configure(); // 5000 3 (uses default object)

// ============================================
// Swapping Variables
// ============================================

let a = 1, b = 2;
[a, b] = [b, a]; // Swap
console.log(a, b); // 2 1

// With objects:
let obj8 = { x: 1, y: 2 };
({ x: obj8.y, y: obj8.x } = obj8);
console.log(obj8); // { x: 2, y: 1 }
```

---

## 8. Arrays

### 8.1 Array Basics

```javascript
// ============================================
// Creating Arrays
// ============================================

// Array literal:
const arr1 = [1, 2, 3, 4, 5];

// Array constructor:
const arr2 = new Array(5); // [empty × 5] - length 5, empty slots
const arr3 = new Array(1, 2, 3); // [1, 2, 3]

// Array.of() - Create array from arguments:
const arr4 = Array.of(5); // [5] - not length 5!
const arr5 = Array.of(1, 2, 3); // [1, 2, 3]

// Array.from() - Create from iterable:
const arr6 = Array.from('hello'); // ['h', 'e', 'l', 'l', 'o']
const arr7 = Array.from({ length: 5 }, (_, i) => i); // [0, 1, 2, 3, 4]

// Fill array:
const arr8 = new Array(5).fill(0); // [0, 0, 0, 0, 0]

// ============================================
// Accessing Elements
// ============================================

const fruits = ['apple', 'banana', 'cherry'];

console.log(fruits[0]); // 'apple'
console.log(fruits[1]); // 'banana'
console.log(fruits[fruits.length - 1]); // 'cherry' (last element)
console.log(fruits.at(-1)); // 'cherry' (ES2022 - negative indexing)
console.log(fruits.at(-2)); // 'banana'

// Modify:
fruits[1] = 'blueberry';
console.log(fruits); // ['apple', 'blueberry', 'cherry']

// ============================================
// Array Length and Sparse Arrays
// ============================================

const arr9 = [1, 2, 3];
console.log(arr9.length); // 3

// Changing length:
arr9.length = 5;
console.log(arr9); // [1, 2, 3, empty × 2]

arr9.length = 2;
console.log(arr9); // [1, 2] - truncated!

// Sparse array (with holes):
const sparse = [1, , , 4]; // Holes at index 1 and 2
console.log(sparse.length); // 4
console.log(sparse[1]); // undefined
console.log(1 in sparse); // false (hole, not undefined value)

// Adding elements beyond length:
const arr10 = [1, 2];
arr10[5] = 6;
console.log(arr10); // [1, 2, empty × 3, 6]
```

### 8.2 Array Methods - Mutating

```javascript
// ============================================
// Mutating Methods (modify original array)
// ============================================

// push() - Add to end, returns new length:
const arr = [1, 2, 3];
const len = arr.push(4, 5);
console.log(arr); // [1, 2, 3, 4, 5]
console.log(len); // 5

// pop() - Remove from end, returns removed element:
const last = arr.pop();
console.log(arr); // [1, 2, 3, 4]
console.log(last); // 5

// unshift() - Add to beginning, returns new length:
arr.unshift(0);
console.log(arr); // [0, 1, 2, 3, 4]

// shift() - Remove from beginning, returns removed element:
const first = arr.shift();
console.log(arr); // [1, 2, 3, 4]
console.log(first); // 0

// ============================================
// splice() - Add/remove elements anywhere
// ============================================

const arr11 = ['a', 'b', 'c', 'd', 'e'];

// Remove elements:
// splice(startIndex, deleteCount, ...itemsToAdd)
const removed = arr11.splice(2, 2); // Remove 2 elements starting at index 2
console.log(arr11); // ['a', 'b', 'e']
console.log(removed); // ['c', 'd']

// Add elements:
arr11.splice(2, 0, 'x', 'y'); // At index 2, remove 0, add 'x', 'y'
console.log(arr11); // ['a', 'b', 'x', 'y', 'e']

// Replace elements:
arr11.splice(1, 2, 'new1', 'new2'); // At index 1, remove 2, add 2
console.log(arr11); // ['a', 'new1', 'new2', 'y', 'e']

// ============================================
// sort() - Sort array in place
// ============================================

const numbers = [3, 1, 4, 1, 5, 9, 2, 6];

// Default sort (converts to strings!):
numbers.sort();
console.log(numbers); // [1, 1, 2, 3, 4, 5, 6, 9] (works for this case)

const numbers2 = [10, 5, 40, 25, 1000, 1];
numbers2.sort();
console.log(numbers2); // [1, 10, 1000, 25, 40, 5] - Wrong! String sort

// Correct numeric sort:
numbers2.sort((a, b) => a - b); // Ascending
console.log(numbers2); // [1, 5, 10, 25, 40, 1000]

numbers2.sort((a, b) => b - a); // Descending
console.log(numbers2); // [1000, 40, 25, 10, 5, 1]

// Sort objects:
const people = [
    { name: 'Alice', age: 30 },
    { name: 'Bob', age: 25 },
    { name: 'Charlie', age: 35 }
];

people.sort((a, b) => a.age - b.age); // Sort by age
console.log(people);
// [{ name: 'Bob', age: 25 }, { name: 'Alice', age: 30 }, { name: 'Charlie', age: 35 }]

// String sort:
const names = ['Charlie', 'Alice', 'Bob'];
names.sort(); // Alphabetical
console.log(names); // ['Alice', 'Bob', 'Charlie']

// Case-insensitive sort:
const names2 = ['charlie', 'Alice', 'bob'];
names2.sort((a, b) => a.toLowerCase().localeCompare(b.toLowerCase()));

// ============================================
// reverse() - Reverse array in place
// ============================================

const arr12 = [1, 2, 3, 4, 5];
arr12.reverse();
console.log(arr12); // [5, 4, 3, 2, 1]

// ============================================
// fill() - Fill array with static value
// ============================================

const arr13 = [1, 2, 3, 4, 5];
arr13.fill(0); // Fill all with 0
console.log(arr13); // [0, 0, 0, 0, 0]

const arr14 = [1, 2, 3, 4, 5];
arr14.fill(9, 2, 4); // Fill from index 2 to 4 (exclusive)
console.log(arr14); // [1, 2, 9, 9, 5]

// ============================================
// copyWithin() - Copy part of array to another location
// ============================================

const arr15 = [1, 2, 3, 4, 5];
arr15.copyWithin(0, 3); // Copy from index 3 to index 0
console.log(arr15); // [4, 5, 3, 4, 5]
```

### 8.3 Array Methods - Non-Mutating

```javascript
// ============================================
// Non-Mutating Methods (return new array/value)
// ============================================

// concat() - Merge arrays:
const arr1 = [1, 2];
const arr2 = [3, 4];
const merged = arr1.concat(arr2, [5, 6]);
console.log(merged); // [1, 2, 3, 4, 5, 6]
console.log(arr1); // [1, 2] - unchanged

// Modern way - Spread:
const merged2 = [...arr1, ...arr2, 5, 6];

// ============================================
// slice() - Extract portion of array
// ============================================

const arr = ['a', 'b', 'c', 'd', 'e'];

const sliced = arr.slice(1, 4); // From index 1 to 4 (exclusive)
console.log(sliced); // ['b', 'c', 'd']
console.log(arr); // ['a', 'b', 'c', 'd', 'e'] - unchanged

// Negative indices:
console.log(arr.slice(-2)); // ['d', 'e'] - last 2 elements
console.log(arr.slice(1, -1)); // ['b', 'c', 'd'] - exclude first and last

// Copy array:
const copy = arr.slice();

// ============================================
// join() - Join elements into string
// ============================================

const words = ['Hello', 'World'];
const sentence = words.join(' ');
console.log(sentence); // \"Hello World\"

const path = ['home', 'user', 'documents'];
console.log(path.join('/')); // \"home/user/documents\"

// ============================================
// indexOf() / lastIndexOf() - Find index
// ============================================

const numbers = [1, 2, 3, 2, 1];

console.log(numbers.indexOf(2)); // 1 (first occurrence)
console.log(numbers.lastIndexOf(2)); // 3 (last occurrence)
console.log(numbers.indexOf(5)); // -1 (not found)

// Start searching from index:
console.log(numbers.indexOf(2, 2)); // 3 (start from index 2)

// ============================================
// includes() - Check if element exists
// ============================================

const fruits = ['apple', 'banana', 'cherry'];

console.log(fruits.includes('banana')); // true
console.log(fruits.includes('grape')); // false

// includes() can find NaN (unlike indexOf):
const arr3 = [1, 2, NaN, 4];
console.log(arr3.includes(NaN)); // true
console.log(arr3.indexOf(NaN)); // -1 (can't find NaN)

// ============================================
// toString() / toLocaleString()
// ============================================

const arr4 = [1, 2, 3];
console.log(arr4.toString()); // \"1,2,3\"

const dates = [new Date(), new Date()];
console.log(dates.toLocaleString()); // Locale-specific formatting
```

### 8.4 Array Iteration Methods

```javascript
// ============================================
// forEach() - Execute function for each element
// ============================================

const numbers = [1, 2, 3, 4, 5];

numbers.forEach((num, index, array) => {
    console.log(`Index ${index}: ${num}`);
});

// forEach doesn't return anything:
const result = numbers.forEach(n => n * 2);
console.log(result); // undefined

// Can't break out of forEach:
numbers.forEach(num => {
    if (num === 3) return; // Only skips this iteration, doesn't break
    console.log(num);
}); // Logs: 1, 2, 4, 5

// ============================================
// map() - Transform each element, return new array
// ============================================

const doubled = numbers.map(n => n * 2);
console.log(doubled); // [2, 4, 6, 8, 10]
console.log(numbers); // [1, 2, 3, 4, 5] - unchanged

// With index:
const indexed = numbers.map((n, i) => `${i}: ${n}`);
console.log(indexed); // ['0: 1', '1: 2', '2: 3', '3: 4', '4: 5']

// Map object properties:
const users = [
    { name: 'Alice', age: 30 },
    { name: 'Bob', age: 25 }
];
const names = users.map(user => user.name);
console.log(names); // ['Alice', 'Bob']

// ============================================
// filter() - Return array of elements that pass test
// ============================================

const evens = numbers.filter(n => n % 2 === 0);
console.log(evens); // [2, 4]

// Filter objects:
const adults = users.filter(user => user.age >= 18);

// Remove falsy values:
const arr5 = [0, 1, false, 2, '', 3, null, undefined, NaN];
const truthy = arr5.filter(Boolean);
console.log(truthy); // [1, 2, 3]

// ============================================
// reduce() - Reduce array to single value
// ============================================

// Sum:
const sum = numbers.reduce((accumulator, current) => {
    return accumulator + current;
}, 0); // 0 is initial value
console.log(sum); // 15

// Reduce flow:
/*
 * Initial: accumulator = 0
 * Iteration 1: accumulator = 0 + 1 = 1
 * Iteration 2: accumulator = 1 + 2 = 3
 * Iteration 3: accumulator = 3 + 3 = 6
 * Iteration 4: accumulator = 6 + 4 = 10
 * Iteration 5: accumulator = 10 + 5 = 15
 * Return: 15
 */

// Product:
const product = numbers.reduce((acc, curr) => acc * curr, 1);
console.log(product); // 120

// Max value:
const max = numbers.reduce((max, curr) => curr > max ? curr : max);
console.log(max); // 5

// Count occurrences:
const words = ['apple', 'banana', 'apple', 'cherry', 'banana', 'apple'];
const count = words.reduce((acc, word) => {
    acc[word] = (acc[word] || 0) + 1;
    return acc;
}, {});
console.log(count); // { apple: 3, banana: 2, cherry: 1 }

// Group by property:
const people = [
    { name: 'Alice', age: 30 },
    { name: 'Bob', age: 25 },
    { name: 'Charlie', age: 30 }
];
const groupedByAge = people.reduce((acc, person) => {
    const age = person.age;
    if (!acc[age]) acc[age] = [];
    acc[age].push(person);
    return acc;
}, {});
console.log(groupedByAge);
// { 25: [{ name: 'Bob', age: 25 }], 30: [{ name: 'Alice', age: 30 }, { name: 'Charlie', age: 30 }] }

// Flatten array:
const nested = [[1, 2], [3, 4], [5, 6]];
const flattened = nested.reduce((acc, arr) => acc.concat(arr), []);
console.log(flattened); // [1, 2, 3, 4, 5, 6]

// Modern way:
console.log(nested.flat()); // [1, 2, 3, 4, 5, 6]

// reduceRight() - Same as reduce, but right to left:
const arr6 = [1, 2, 3, 4];
const result2 = arr6.reduceRight((acc, curr) => acc + curr);
console.log(result2); // 10 (same result, different order)

// ============================================
// find() / findIndex() - Find first matching element
// ============================================

const users2 = [
    { id: 1, name: 'Alice' },
    { id: 2, name: 'Bob' },
    { id: 3, name: 'Charlie' }
];

// find() - Returns element or undefined:
const user = users2.find(u => u.id === 2);
console.log(user); // { id: 2, name: 'Bob' }

const notFound = users2.find(u => u.id === 99);
console.log(notFound); // undefined

// findIndex() - Returns index or -1:
const index = users2.findIndex(u => u.name === 'Bob');
console.log(index); // 1

// findLast() / findLastIndex() - ES2023 (right to left):
const lastEven = numbers.findLast(n => n % 2 === 0);
console.log(lastEven); // 4

// ============================================
// some() / every() - Test if any/all pass test
// ============================================

const numbers2 = [1, 2, 3, 4, 5];

// some() - Returns true if ANY element passes:
const hasEven = numbers2.some(n => n % 2 === 0);
console.log(hasEven); // true

const hasNegative = numbers2.some(n => n < 0);
console.log(hasNegative); // false

// every() - Returns true if ALL elements pass:
const allPositive = numbers2.every(n => n > 0);
console.log(allPositive); // true

const allEven = numbers2.every(n => n % 2 === 0);
console.log(allEven); // false

// ============================================
// flat() / flatMap() - Flatten nested arrays
// ============================================

const nested2 = [1, [2, 3], [4, [5, 6]]];

// flat() - Flatten one level by default:
console.log(nested2.flat()); // [1, 2, 3, 4, [5, 6]]

// Specify depth:
console.log(nested2.flat(2)); // [1, 2, 3, 4, 5, 6]
console.log(nested2.flat(Infinity)); // Flatten all levels

// flatMap() - Map then flatten:
const arr7 = [1, 2, 3];
const result3 = arr7.flatMap(n => [n, n * 2]);
console.log(result3); // [1, 2, 2, 4, 3, 6]

// Equivalent to:
const result4 = arr7.map(n => [n, n * 2]).flat();
```

### 8.5 Array Destructuring

```javascript
// ============================================
// Basic Destructuring
// ============================================

const arr = [1, 2, 3, 4, 5];

// Extract elements:
const [first, second] = arr;
console.log(first); // 1
console.log(second); // 2

// Skip elements:
const [, , third] = arr;
console.log(third); // 3

// Rest pattern:
const [head, ...tail] = arr;
console.log(head); // 1
console.log(tail); // [2, 3, 4, 5]

// Default values:
const [a = 0, b = 0, c = 0, d = 0] = [1, 2];
console.log(a, b, c, d); // 1 2 0 0

// ============================================
// Swapping Variables
// ============================================

let x = 1, y = 2;
[x, y] = [y, x];
console.log(x, y); // 2 1

// ============================================
// Nested Destructuring
// ============================================

const nested = [1, [2, 3], 4];
const [a1, [b1, c1], d1] = nested;
console.log(a1, b1, c1, d1); // 1 2 3 4

// ============================================
// Function Returns
// ============================================

function getMinMax(arr) {
    return [Math.min(...arr), Math.max(...arr)];
}

const [min, max] = getMinMax([3, 1, 4, 1, 5, 9]);
console.log(min, max); // 1 9
```

---

*This continues the comprehensive JavaScript tutorial with detailed explanations of objects and arrays. The tutorial will continue with more advanced topics in subsequent parts.*
