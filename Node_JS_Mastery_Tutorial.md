# Node.js Mastery Tutorial - From Zero to Ninja

> A comprehensive guide transforming complete beginners into Node.js experts through structured, linear progression with extensive code examples.

---

## Table of Contents

- [Phase 1: JavaScript Essentials](#phase-1-javascript-essentials)
- [Phase 2: Node.js Fundamentals](#phase-2-nodejs-fundamentals)
- [Phase 3: Asynchronous Programming](#phase-3-asynchronous-programming)
- [Phase 4: Express.js Framework](#phase-4-expressjs-framework)
- [Phase 5: Database Integration](#phase-5-database-integration)
- [Phase 6: Authentication & Security](#phase-6-authentication--security)
- [Phase 7: Testing & Debugging](#phase-7-testing--debugging)
- [Phase 8: Advanced Topics](#phase-8-advanced-topics)
- [Phase 9: Production & Deployment](#phase-9-production--deployment)

---

## Phase 1: JavaScript Essentials

### Module 1.1: Programming Fundamentals

#### Variables and Data Types

**Variables - Your Data Containers**

Variables store data values. Think of them as labeled boxes where you store information.

```javascript
// Variable Declaration with let (can be changed)
let userName = "Alice";
let userAge = 25;
let isActive = true;

console.log(userName); // Output: Alice
console.log(userAge);  // Output: 25
console.log(isActive); // Output: true

// Changing variable values
userName = "Bob";
userAge = 30;
console.log(userName); // Output: Bob

// Variable Declaration with const (cannot be changed)
const PI = 3.14159;
const MAX_USERS = 100;
const APP_NAME = "MyApp";

console.log(PI); // Output: 3.14159

// This would cause an error:
// PI = 3.14; // TypeError: Assignment to constant variable

// Old way with var (avoid using this)
var oldStyle = "Not recommended";
```

**Data Types - Understanding Your Data**

JavaScript has several primitive data types:

```javascript
// 1. String - Text data
let firstName = "John";
let lastName = 'Doe';
let fullName = `John Doe`; // Template literal

console.log(typeof firstName); // Output: string

// 2. Number - Numeric values (integers and decimals)
let age = 25;
let price = 19.99;
let negative = -10;
let zero = 0;

console.log(typeof age); // Output: number

// 3. Boolean - True or false
let isLoggedIn = true;
let hasPermission = false;

console.log(typeof isLoggedIn); // Output: boolean

// 4. Undefined - Variable declared but not assigned
let notAssigned;
console.log(notAssigned); // Output: undefined
console.log(typeof notAssigned); // Output: undefined

// 5. Null - Intentional absence of value
let emptyValue = null;
console.log(emptyValue); // Output: null
console.log(typeof emptyValue); // Output: object (JavaScript quirk)

// 6. Symbol - Unique identifier
let id = Symbol('id');
let anotherId = Symbol('id');
console.log(id === anotherId); // Output: false (each symbol is unique)

// 7. BigInt - Large integers
let bigNumber = 1234567890123456789012345678901234567890n;
console.log(typeof bigNumber); // Output: bigint
```

**Operators - Performing Operations**

```javascript
// Arithmetic Operators
let a = 10;
let b = 3;

console.log(a + b);  // Addition: 13
console.log(a - b);  // Subtraction: 7
console.log(a * b);  // Multiplication: 30
console.log(a / b);  // Division: 3.3333...
console.log(a % b);  // Modulus (remainder): 1
console.log(a ** b); // Exponentiation: 1000

// Increment and Decrement
let count = 5;
count++;  // count is now 6
count--;  // count is now 5

// Comparison Operators
console.log(5 == "5");   // true (loose equality, converts types)
console.log(5 === "5");  // false (strict equality, checks type too)
console.log(5 != "5");   // false
console.log(5 !== "5");  // true
console.log(10 > 5);     // true
console.log(10 < 5);     // false
console.log(10 >= 10);   // true
console.log(10 <= 5);    // false

// Logical Operators
let x = true;
let y = false;

console.log(x && y);  // AND: false (both must be true)
console.log(x || y);  // OR: true (at least one must be true)
console.log(!x);      // NOT: false (inverts the value)

// Assignment Operators
let num = 10;
num += 5;  // num = num + 5; Result: 15
num -= 3;  // num = num - 3; Result: 12
num *= 2;  // num = num * 2; Result: 24
num /= 4;  // num = num / 4; Result: 6
num %= 4;  // num = num % 4; Result: 2
```

#### Control Flow

**If Statements - Making Decisions**

```javascript
// Basic if statement
let age = 18;

if (age >= 18) {
    console.log("You are an adult");
}

// If-else statement
let temperature = 25;

if (temperature > 30) {
    console.log("It's hot outside");
} else {
    console.log("It's not too hot");
}

// If-else if-else chain
let score = 85;

if (score >= 90) {
    console.log("Grade: A");
} else if (score >= 80) {
    console.log("Grade: B");
} else if (score >= 70) {
    console.log("Grade: C");
} else if (score >= 60) {
    console.log("Grade: D");
} else {
    console.log("Grade: F");
}

// Nested if statements
let isLoggedIn = true;
let hasPermission = true;

if (isLoggedIn) {
    if (hasPermission) {
        console.log("Access granted");
    } else {
        console.log("No permission");
    }
} else {
    console.log("Please log in");
}

// Ternary operator - shorthand if-else
let userAge = 20;
let canVote = userAge >= 18 ? "Yes" : "No";
console.log(canVote); // Output: Yes

// Complex condition with logical operators
let username = "admin";
let password = "secret123";

if (username === "admin" && password === "secret123") {
    console.log("Login successful");
} else {
    console.log("Invalid credentials");
}
```

**Switch Statements - Multiple Cases**

```javascript
// Basic switch statement
let day = 3;
let dayName;

switch (day) {
    case 1:
        dayName = "Monday";
        break;
    case 2:
        dayName = "Tuesday";
        break;
    case 3:
        dayName = "Wednesday";
        break;
    case 4:
        dayName = "Thursday";
        break;
    case 5:
        dayName = "Friday";
        break;
    case 6:
        dayName = "Saturday";
        break;
    case 7:
        dayName = "Sunday";
        break;
    default:
        dayName = "Invalid day";
}

console.log(dayName); // Output: Wednesday

// Switch with fall-through (multiple cases)
let fruit = "apple";

switch (fruit) {
    case "apple":
    case "pear":
    case "orange":
        console.log("This is a fruit");
        break;
    case "carrot":
    case "broccoli":
        console.log("This is a vegetable");
        break;
    default:
        console.log("Unknown food");
}

// Switch with expressions
let grade = "B";

switch (grade) {
    case "A":
        console.log("Excellent! Score: 90-100");
        break;
    case "B":
        console.log("Good! Score: 80-89");
        break;
    case "C":
        console.log("Fair! Score: 70-79");
        break;
    case "D":
        console.log("Poor! Score: 60-69");
        break;
    case "F":
        console.log("Failed! Score: Below 60");
        break;
    default:
        console.log("Invalid grade");
}
```

#### Loops - Repeating Actions

**For Loop - Count-Controlled**

```javascript
// Basic for loop
for (let i = 0; i < 5; i++) {
    console.log("Iteration: " + i);
}
// Output: Iteration: 0, 1, 2, 3, 4

// Loop through array
let fruits = ["apple", "banana", "orange", "mango"];

for (let i = 0; i < fruits.length; i++) {
    console.log(fruits[i]);
}
// Output: apple, banana, orange, mango

// Loop with different step
for (let i = 0; i <= 10; i += 2) {
    console.log(i);
}
// Output: 0, 2, 4, 6, 8, 10

// Countdown loop
for (let i = 5; i >= 1; i--) {
    console.log(i);
}
console.log("Blast off!");
// Output: 5, 4, 3, 2, 1, Blast off!

// Nested loops - Multiplication table
for (let i = 1; i <= 3; i++) {
    for (let j = 1; j <= 3; j++) {
        console.log(`${i} x ${j} = ${i * j}`);
    }
}
```

**While Loop - Condition-Controlled**

```javascript
// Basic while loop
let count = 0;

while (count < 5) {
    console.log("Count: " + count);
    count++;
}
// Output: Count: 0, 1, 2, 3, 4

// While loop with user input simulation
let password = "";
let attempts = 0;
let maxAttempts = 3;

while (password !== "secret" && attempts < maxAttempts) {
    console.log("Enter password attempt " + (attempts + 1));
    // Simulate password entry
    password = attempts === 2 ? "secret" : "wrong";
    attempts++;
}

if (password === "secret") {
    console.log("Access granted!");
} else {
    console.log("Too many attempts!");
}

// Infinite loop with break
let num = 0;

while (true) {
    console.log(num);
    num++;
    
    if (num >= 5) {
        break; // Exit the loop
    }
}
```

**Do-While Loop - Execute At Least Once**

```javascript
// Basic do-while loop
let i = 0;

do {
    console.log("Number: " + i);
    i++;
} while (i < 5);
// Output: Number: 0, 1, 2, 3, 4

// Do-while that executes once even if condition is false
let x = 10;

do {
    console.log("This runs at least once: " + x);
} while (x < 5);
// Output: This runs at least once: 10

// Menu-driven program example
let choice;
do {
    console.log("\n=== Menu ===");
    console.log("1. Option 1");
    console.log("2. Option 2");
    console.log("3. Exit");
    
    // Simulate user choice
    choice = 3; // Exit
    
    if (choice === 1) {
        console.log("You selected Option 1");
    } else if (choice === 2) {
        console.log("You selected Option 2");
    }
} while (choice !== 3);

console.log("Goodbye!");
```

**For-Of and For-In Loops**

```javascript
// For-of loop - iterate over values
let colors = ["red", "green", "blue"];

for (let color of colors) {
    console.log(color);
}
// Output: red, green, blue

// For-of with strings
let text = "Hello";

for (let char of text) {
    console.log(char);
}
// Output: H, e, l, l, o

// For-in loop - iterate over object keys
let person = {
    name: "John",
    age: 30,
    city: "New York"
};

for (let key in person) {
    console.log(key + ": " + person[key]);
}
// Output: name: John, age: 30, city: New York

// For-in with arrays (not recommended, but possible)
let numbers = [10, 20, 30];

for (let index in numbers) {
    console.log(index + ": " + numbers[index]);
}
// Output: 0: 10, 1: 20, 2: 30
```

**Break and Continue Statements**

```javascript
// Break - exit loop early
for (let i = 0; i < 10; i++) {
    if (i === 5) {
        break; // Exit loop when i is 5
    }
    console.log(i);
}
// Output: 0, 1, 2, 3, 4

// Continue - skip current iteration
for (let i = 0; i < 10; i++) {
    if (i % 2 === 0) {
        continue; // Skip even numbers
    }
    console.log(i);
}
// Output: 1, 3, 5, 7, 9

// Break in nested loops
outerLoop: for (let i = 0; i < 3; i++) {
    for (let j = 0; j < 3; j++) {
        if (i === 1 && j === 1) {
            break outerLoop; // Break out of both loops
        }
        console.log(`i: ${i}, j: ${j}`);
    }
}

// Finding a value with break
let searchNumbers = [5, 10, 15, 20, 25];
let target = 15;
let found = false;

for (let num of searchNumbers) {
    if (num === target) {
        found = true;
        console.log("Found: " + num);
        break;
    }
}

if (!found) {
    console.log("Not found");
}
```

### Module 1.2: Functions and Scope

#### Function Basics

**Function Declaration**

```javascript
// Basic function declaration
function greet() {
    console.log("Hello, World!");
}

greet(); // Output: Hello, World!

// Function with parameters
function greetUser(name) {
    console.log("Hello, " + name + "!");
}

greetUser("Alice"); // Output: Hello, Alice!
greetUser("Bob");   // Output: Hello, Bob!

// Function with multiple parameters
function add(a, b) {
    return a + b;
}

let result = add(5, 3);
console.log(result); // Output: 8

// Function with return value
function multiply(x, y) {
    return x * y;
}

let product = multiply(4, 7);
console.log(product); // Output: 28

// Function without return (returns undefined)
function logMessage(message) {
    console.log(message);
    // No return statement
}

let returnValue = logMessage("Test");
console.log(returnValue); // Output: undefined

// Function with default parameters
function greetWithDefault(name = "Guest") {
    console.log("Hello, " + name + "!");
}

greetWithDefault();        // Output: Hello, Guest!
greetWithDefault("John");  // Output: Hello, John!

// Function with rest parameters
function sum(...numbers) {
    let total = 0;
    for (let num of numbers) {
        total += num;
    }
    return total;
}

console.log(sum(1, 2, 3));       // Output: 6
console.log(sum(1, 2, 3, 4, 5)); // Output: 15
```

**Function Expressions**

```javascript
// Anonymous function expression
const square = function(num) {
    return num * num;
};

console.log(square(5)); // Output: 25

// Named function expression
const factorial = function fact(n) {
    if (n <= 1) return 1;
    return n * fact(n - 1); // Can use 'fact' inside
};

console.log(factorial(5)); // Output: 120

// Function as callback
function processArray(arr, callback) {
    for (let i = 0; i < arr.length; i++) {
        callback(arr[i]);
    }
}

processArray([1, 2, 3], function(num) {
    console.log(num * 2);
});
// Output: 2, 4, 6

// Returning functions
function multiplier(factor) {
    return function(number) {
        return number * factor;
    };
}

const double = multiplier(2);
const triple = multiplier(3);

console.log(double(5)); // Output: 10
console.log(triple(5)); // Output: 15
```

**Arrow Functions (ES6+)**

```javascript
// Basic arrow function
const greet = () => {
    console.log("Hello!");
};

greet(); // Output: Hello!

// Arrow function with one parameter (parentheses optional)
const square = num => {
    return num * num;
};

console.log(square(4)); // Output: 16

// Arrow function with implicit return (one-liner)
const cube = num => num * num * num;

console.log(cube(3)); // Output: 27

// Arrow function with multiple parameters
const add = (a, b) => a + b;

console.log(add(5, 3)); // Output: 8

// Arrow function returning object (wrap in parentheses)
const createPerson = (name, age) => ({
    name: name,
    age: age
});

console.log(createPerson("Alice", 25));
// Output: { name: 'Alice', age: 25 }

// Arrow function in array methods
const numbers = [1, 2, 3, 4, 5];

const doubled = numbers.map(num => num * 2);
console.log(doubled); // Output: [2, 4, 6, 8, 10]

const evens = numbers.filter(num => num % 2 === 0);
console.log(evens); // Output: [2, 4]

// Arrow function with this context
const person = {
    name: "John",
    hobbies: ["reading", "coding"],
    printHobbies: function() {
        this.hobbies.forEach(hobby => {
            // Arrow function inherits 'this' from parent
            console.log(this.name + " likes " + hobby);
        });
    }
};

person.printHobbies();
// Output: John likes reading
//         John likes coding
```

#### Variable Scope

**Global and Local Scope**

```javascript
// Global scope - accessible everywhere
let globalVar = "I'm global";

function testScope() {
    // Local scope - only accessible inside function
    let localVar = "I'm local";
    console.log(globalVar);  // Can access global
    console.log(localVar);   // Can access local
}

testScope();
console.log(globalVar);  // Can access global
// console.log(localVar);  // Error: localVar is not defined

// Function scope with var
function varTest() {
    var x = 1;
    if (true) {
        var x = 2;  // Same variable!
        console.log(x);  // Output: 2
    }
    console.log(x);  // Output: 2
}

varTest();

// Block scope with let and const
function letTest() {
    let x = 1;
    if (true) {
        let x = 2;  // Different variable
        console.log(x);  // Output: 2
    }
    console.log(x);  // Output: 1
}

letTest();

// Lexical scope - inner functions access outer variables
function outer() {
    let outerVar = "I'm from outer";
    
    function inner() {
        let innerVar = "I'm from inner";
        console.log(outerVar);  // Can access outer variable
        console.log(innerVar);  // Can access own variable
    }
    
    inner();
    // console.log(innerVar);  // Error: innerVar is not defined
}

outer();
```

**Closures - Functions Remember Their Environment**

```javascript
// Basic closure
function createCounter() {
    let count = 0;  // Private variable
    
    return function() {
        count++;
        return count;
    };
}

const counter1 = createCounter();
console.log(counter1());  // Output: 1
console.log(counter1());  // Output: 2
console.log(counter1());  // Output: 3

const counter2 = createCounter();
console.log(counter2());  // Output: 1 (separate counter)

// Closure with multiple functions
function createPerson(name) {
    let age = 0;
    
    return {
        getName: function() {
            return name;
        },
        getAge: function() {
            return age;
        },
        birthday: function() {
            age++;
        }
    };
}

const person = createPerson("Alice");
console.log(person.getName());  // Output: Alice
console.log(person.getAge());   // Output: 0
person.birthday();
console.log(person.getAge());   // Output: 1

// Closure in loops - common mistake
for (var i = 1; i <= 3; i++) {
    setTimeout(function() {
        console.log(i);  // Output: 4, 4, 4 (not 1, 2, 3)
    }, 1000);
}

// Fixing with let (block scope)
for (let i = 1; i <= 3; i++) {
    setTimeout(function() {
        console.log(i);  // Output: 1, 2, 3 (correct!)
    }, 1000);
}

// Fixing with closure
for (var i = 1; i <= 3; i++) {
    (function(j) {
        setTimeout(function() {
            console.log(j);  // Output: 1, 2, 3
        }, 1000);
    })(i);
}

// Practical closure example - Private data
function BankAccount(initialBalance) {
    let balance = initialBalance;
    
    return {
        deposit: function(amount) {
            balance += amount;
            return balance;
        },
        withdraw: function(amount) {
            if (amount <= balance) {
                balance -= amount;
                return balance;
            } else {
                return "Insufficient funds";
            }
        },
        getBalance: function() {
            return balance;
        }
    };
}

const myAccount = BankAccount(1000);
console.log(myAccount.getBalance());     // Output: 1000
console.log(myAccount.deposit(500));     // Output: 1500
console.log(myAccount.withdraw(200));    // Output: 1300
console.log(myAccount.withdraw(2000));   // Output: Insufficient funds
// console.log(balance);  // Error: balance is not accessible
```

### Module 1.3: Objects and Arrays

#### Working with Objects

**Object Basics**

```javascript
// Creating objects - Object literal
let person = {
    firstName: "John",
    lastName: "Doe",
    age: 30,
    isEmployed: true,
    address: {
        street: "123 Main St",
        city: "New York",
        zipCode: "10001"
    }
};

// Accessing properties - Dot notation
console.log(person.firstName);  // Output: John
console.log(person.age);        // Output: 30

// Accessing properties - Bracket notation
console.log(person["lastName"]);  // Output: Doe

// Bracket notation with variables
let propertyName = "age";
console.log(person[propertyName]);  // Output: 30

// Accessing nested properties
console.log(person.address.city);  // Output: New York

// Adding new properties
person.email = "john@example.com";
person["phone"] = "555-1234";

console.log(person.email);  // Output: john@example.com

// Modifying properties
person.age = 31;
console.log(person.age);  // Output: 31

// Deleting properties
delete person.isEmployed;
console.log(person.isEmployed);  // Output: undefined

// Checking if property exists
console.log("firstName" in person);  // Output: true
console.log("salary" in person);     // Output: false

// Object with methods
let calculator = {
    value: 0,
    add: function(num) {
        this.value += num;
        return this;
    },
    subtract: function(num) {
        this.value -= num;
        return this;
    },
    multiply: function(num) {
        this.value *= num;
        return this;
    },
    getResult: function() {
        return this.value;
    }
};

// Method chaining
calculator.add(10).multiply(2).subtract(5);
console.log(calculator.getResult());  // Output: 15
```

**Object Methods**

```javascript
// Object.keys() - Get array of keys
let user = {
    name: "Alice",
    age: 25,
    city: "Boston"
};

let keys = Object.keys(user);
console.log(keys);  // Output: ['name', 'age', 'city']

// Object.values() - Get array of values
let values = Object.values(user);
console.log(values);  // Output: ['Alice', 25, 'Boston']

// Object.entries() - Get array of [key, value] pairs
let entries = Object.entries(user);
console.log(entries);
// Output: [['name', 'Alice'], ['age', 25], ['city', 'Boston']]

// Looping through object entries
for (let [key, value] of Object.entries(user)) {
    console.log(`${key}: ${value}`);
}
// Output: name: Alice, age: 25, city: Boston

// Object.assign() - Copy properties
let target = { a: 1, b: 2 };
let source = { b: 3, c: 4 };

let result = Object.assign(target, source);
console.log(result);  // Output: { a: 1, b: 3, c: 4 }
console.log(target);  // Output: { a: 1, b: 3, c: 4 } (modified)

// Object.assign() for cloning
let original = { x: 1, y: 2 };
let clone = Object.assign({}, original);

clone.x = 10;
console.log(original.x);  // Output: 1 (unchanged)
console.log(clone.x);     // Output: 10

// Object.freeze() - Make object immutable
let frozen = Object.freeze({ name: "Bob", age: 30 });

frozen.age = 31;  // This won't work
console.log(frozen.age);  // Output: 30 (unchanged)

// Object.seal() - Prevent adding/removing properties
let sealed = Object.seal({ name: "Carol", age: 28 });

sealed.age = 29;      // This works
sealed.city = "NY";   // This doesn't work
delete sealed.name;   // This doesn't work

console.log(sealed);  // Output: { name: 'Carol', age: 29 }
```

**Object Destructuring**

```javascript
// Basic destructuring
let person = {
    name: "John",
    age: 30,
    city: "New York"
};

let { name, age, city } = person;
console.log(name);  // Output: John
console.log(age);   // Output: 30
console.log(city);  // Output: New York

// Destructuring with different variable names
let { name: personName, age: personAge } = person;
console.log(personName);  // Output: John
console.log(personAge);   // Output: 30

// Destructuring with default values
let user = { username: "alice" };
let { username, email = "no-email@example.com" } = user;

console.log(username);  // Output: alice
console.log(email);     // Output: no-email@example.com

// Nested destructuring
let employee = {
    id: 101,
    info: {
        name: "Bob",
        position: "Developer"
    }
};

let { info: { name: empName, position } } = employee;
console.log(empName);   // Output: Bob
console.log(position);  // Output: Developer

// Destructuring in function parameters
function displayUser({ name, age, city = "Unknown" }) {
    console.log(`${name} is ${age} years old and lives in ${city}`);
}

displayUser({ name: "Alice", age: 25, city: "Boston" });
// Output: Alice is 25 years old and lives in Boston

displayUser({ name: "Bob", age: 30 });
// Output: Bob is 30 years old and lives in Unknown
```

#### Working with Arrays

**Array Basics**

```javascript
// Creating arrays
let fruits = ["apple", "banana", "orange"];
let numbers = [1, 2, 3, 4, 5];
let mixed = [1, "hello", true, null, { name: "John" }];
let empty = [];

// Accessing array elements (zero-based indexing)
console.log(fruits[0]);  // Output: apple
console.log(fruits[1]);  // Output: banana
console.log(fruits[2]);  // Output: orange

// Array length
console.log(fruits.length);  // Output: 3

// Accessing last element
console.log(fruits[fruits.length - 1]);  // Output: orange

// Modifying array elements
fruits[1] = "mango";
console.log(fruits);  // Output: ['apple', 'mango', 'orange']

// Adding elements
fruits[3] = "grape";
console.log(fruits);  // Output: ['apple', 'mango', 'orange', 'grape']

// Multi-dimensional arrays
let matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
];

console.log(matrix[0][0]);  // Output: 1
console.log(matrix[1][2]);  // Output: 6
console.log(matrix[2][1]);  // Output: 8
```

**Array Methods - Adding and Removing**

```javascript
// push() - Add to end
let fruits = ["apple", "banana"];
fruits.push("orange");
console.log(fruits);  // Output: ['apple', 'banana', 'orange']

let newLength = fruits.push("mango", "grape");
console.log(fruits);      // Output: ['apple', 'banana', 'orange', 'mango', 'grape']
console.log(newLength);   // Output: 5

// pop() - Remove from end
let removed = fruits.pop();
console.log(removed);  // Output: grape
console.log(fruits);   // Output: ['apple', 'banana', 'orange', 'mango']

// unshift() - Add to beginning
fruits.unshift("strawberry");
console.log(fruits);  // Output: ['strawberry', 'apple', 'banana', 'orange', 'mango']

// shift() - Remove from beginning
let first = fruits.shift();
console.log(first);   // Output: strawberry
console.log(fruits);  // Output: ['apple', 'banana', 'orange', 'mango']

// splice() - Add, remove, or replace elements
let colors = ["red", "green", "blue", "yellow"];

// Remove 2 elements starting at index 1
let removed1 = colors.splice(1, 2);
console.log(colors);     // Output: ['red', 'yellow']
console.log(removed1);   // Output: ['green', 'blue']

// Insert elements at index 1
colors.splice(1, 0, "purple", "orange");
console.log(colors);  // Output: ['red', 'purple', 'orange', 'yellow']

// Replace elements
colors.splice(1, 2, "pink");
console.log(colors);  // Output: ['red', 'pink', 'yellow']

// slice() - Extract portion (doesn't modify original)
let numbers = [1, 2, 3, 4, 5];
let sliced = numbers.slice(1, 4);  // From index 1 to 3 (4 is exclusive)

console.log(sliced);   // Output: [2, 3, 4]
console.log(numbers);  // Output: [1, 2, 3, 4, 5] (unchanged)

// slice with negative indices
let last2 = numbers.slice(-2);
console.log(last2);  // Output: [4, 5]

// concat() - Merge arrays
let arr1 = [1, 2, 3];
let arr2 = [4, 5, 6];
let arr3 = [7, 8, 9];

let combined = arr1.concat(arr2, arr3);
console.log(combined);  // Output: [1, 2, 3, 4, 5, 6, 7, 8, 9]
console.log(arr1);      // Output: [1, 2, 3] (unchanged)
```

**Array Methods - Searching**

```javascript
// indexOf() - Find first index
let fruits = ["apple", "banana", "orange", "banana"];

console.log(fruits.indexOf("banana"));    // Output: 1
console.log(fruits.indexOf("grape"));     // Output: -1 (not found)
console.log(fruits.indexOf("banana", 2)); // Output: 3 (start from index 2)

// lastIndexOf() - Find last index
console.log(fruits.lastIndexOf("banana")); // Output: 3

// includes() - Check if element exists
console.log(fruits.includes("orange"));  // Output: true
console.log(fruits.includes("grape"));   // Output: false

// find() - Find first element matching condition
let numbers = [5, 12, 8, 130, 44];

let found = numbers.find(num => num > 10);
console.log(found);  // Output: 12 (first number > 10)

let notFound = numbers.find(num => num > 200);
console.log(notFound);  // Output: undefined

// findIndex() - Find index of first match
let index = numbers.findIndex(num => num > 10);
console.log(index);  // Output: 1

let notFoundIndex = numbers.findIndex(num => num > 200);
console.log(notFoundIndex);  // Output: -1
```

**Array Methods - Iteration**

```javascript
// forEach() - Execute function for each element
let numbers = [1, 2, 3, 4, 5];

numbers.forEach(function(num) {
    console.log(num * 2);
});
// Output: 2, 4, 6, 8, 10

// forEach with index and array
let fruits = ["apple", "banana", "orange"];

fruits.forEach((fruit, index, array) => {
    console.log(`${index}: ${fruit} of ${array.length}`);
});
// Output: 0: apple of 3, 1: banana of 3, 2: orange of 3

// map() - Transform each element
let numbers2 = [1, 2, 3, 4, 5];
let doubled = numbers2.map(num => num * 2);

console.log(doubled);   // Output: [2, 4, 6, 8, 10]
console.log(numbers2);  // Output: [1, 2, 3, 4, 5] (unchanged)

// map() with objects
let users = [
    { name: "John", age: 30 },
    { name: "Jane", age: 25 },
    { name: "Bob", age: 35 }
];

let names = users.map(user => user.name);
console.log(names);  // Output: ['John', 'Jane', 'Bob']

// filter() - Select elements matching condition
let numbers3 = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
let evens = numbers3.filter(num => num % 2 === 0);

console.log(evens);  // Output: [2, 4, 6, 8, 10]

// filter() with objects
let adults = users.filter(user => user.age >= 30);
console.log(adults);
// Output: [{ name: 'John', age: 30 }, { name: 'Bob', age: 35 }]

// reduce() - Reduce to single value
let numbers4 = [1, 2, 3, 4, 5];
let sum = numbers4.reduce((accumulator, current) => {
    return accumulator + current;
}, 0);

console.log(sum);  // Output: 15

// reduce() without initial value (uses first element)
let product = numbers4.reduce((acc, curr) => acc * curr);
console.log(product);  // Output: 120

// reduce() for complex operations
let cart = [
    { item: "book", price: 10 },
    { item: "pen", price: 2 },
    { item: "notebook", price: 5 }
];

let totalPrice = cart.reduce((total, item) => total + item.price, 0);
console.log(totalPrice);  // Output: 17

// some() - Check if at least one element matches
let numbers5 = [1, 2, 3, 4, 5];

console.log(numbers5.some(num => num > 3));   // Output: true
console.log(numbers5.some(num => num > 10));  // Output: false

// every() - Check if all elements match
console.log(numbers5.every(num => num > 0));  // Output: true
console.log(numbers5.every(num => num > 3));  // Output: false
```

**Array Methods - Sorting and Reversing**

```javascript
// sort() - Sort array (modifies original)
let fruits = ["banana", "apple", "orange", "mango"];
fruits.sort();
console.log(fruits);  // Output: ['apple', 'banana', 'mango', 'orange']

// sort() with numbers (default sorts as strings!)
let numbers = [10, 5, 40, 25, 1000, 1];
numbers.sort();
console.log(numbers);  // Output: [1, 10, 1000, 25, 40, 5] (wrong!)

// sort() with compare function for numbers
numbers.sort((a, b) => a - b);  // Ascending
console.log(numbers);  // Output: [1, 5, 10, 25, 40, 1000]

numbers.sort((a, b) => b - a);  // Descending
console.log(numbers);  // Output: [1000, 40, 25, 10, 5, 1]

// sort() with objects
let users = [
    { name: "John", age: 30 },
    { name: "Jane", age: 25 },
    { name: "Bob", age: 35 }
];

users.sort((a, b) => a.age - b.age);
console.log(users);
// Output: [{ name: 'Jane', age: 25 }, { name: 'John', age: 30 }, { name: 'Bob', age: 35 }]

// reverse() - Reverse array order
let numbers2 = [1, 2, 3, 4, 5];
numbers2.reverse();
console.log(numbers2);  // Output: [5, 4, 3, 2, 1]

// join() - Create string from array
let words = ["Hello", "World", "from", "JavaScript"];
let sentence = words.join(" ");
console.log(sentence);  // Output: Hello World from JavaScript

let csv = words.join(",");
console.log(csv);  // Output: Hello,World,from,JavaScript

// split() and join() combo
let text = "Hello World";
let reversed = text.split("").reverse().join("");
console.log(reversed);  // Output: dlroW olleH
```

**Array Destructuring**

```javascript
// Basic array destructuring
let colors = ["red", "green", "blue"];
let [first, second, third] = colors;

console.log(first);   // Output: red
console.log(second);  // Output: green
console.log(third);   // Output: blue

// Skipping elements
let numbers = [1, 2, 3, 4, 5];
let [a, , c, , e] = numbers;

console.log(a);  // Output: 1
console.log(c);  // Output: 3
console.log(e);  // Output: 5

// Default values
let [x = 10, y = 20, z = 30] = [1, 2];

console.log(x);  // Output: 1
console.log(y);  // Output: 2
console.log(z);  // Output: 30 (default)

// Rest pattern
let [first2, second2, ...rest] = [1, 2, 3, 4, 5];

console.log(first2);   // Output: 1
console.log(second2);  // Output: 2
console.log(rest);     // Output: [3, 4, 5]

// Swapping variables
let a1 = 1;
let b1 = 2;

[a1, b1] = [b1, a1];
console.log(a1);  // Output: 2
console.log(b1);  // Output: 1

// Nested destructuring
let nested = [1, [2, 3], 4];
let [n1, [n2, n3], n4] = nested;

console.log(n1);  // Output: 1
console.log(n2);  // Output: 2
console.log(n3);  // Output: 3
console.log(n4);  // Output: 4
```

### Module 1.4: Modern JavaScript (ES6+)

#### Template Literals

```javascript
// Basic template literals
let name = "John";
let age = 30;

// Old way (concatenation)
let message1 = "My name is " + name + " and I am " + age + " years old.";
console.log(message1);

// New way (template literals)
let message2 = `My name is ${name} and I am ${age} years old.`;
console.log(message2);

// Expressions in template literals
let a = 5;
let b = 10;
console.log(`${a} + ${b} = ${a + b}`);  // Output: 5 + 10 = 15

// Multi-line strings
let multiLine = `
    This is a multi-line
    string that preserves
    line breaks and indentation.
`;
console.log(multiLine);

// Function calls in template literals
function greet(name) {
    return `Hello, ${name}!`;
}

console.log(`${greet("Alice")} Welcome to JavaScript.`);
// Output: Hello, Alice! Welcome to JavaScript.

// HTML templates
let title = "My Page";
let content = "This is the content";

let html = `
    <html>
        <head>
            <title>${title}</title>
        </head>
        <body>
            <h1>${title}</h1>
            <p>${content}</p>
        </body>
    </html>
`;

console.log(html);

// Tagged templates (advanced)
function highlight(strings, ...values) {
    return strings.reduce((result, str, i) => {
        return result + str + (values[i] ? `<strong>${values[i]}</strong>` : "");
    }, "");
}

let product = "laptop";
let price = 999;
let tagged = highlight`The ${product} costs $${price}.`;
console.log(tagged);
// Output: The <strong>laptop</strong> costs $<strong>999</strong>.
```

#### Spread and Rest Operators

```javascript
// Spread operator with arrays
let arr1 = [1, 2, 3];
let arr2 = [4, 5, 6];

// Combining arrays
let combined = [...arr1, ...arr2];
console.log(combined);  // Output: [1, 2, 3, 4, 5, 6]

// Adding elements while spreading
let extended = [0, ...arr1, 3.5, ...arr2, 7];
console.log(extended);  // Output: [0, 1, 2, 3, 3.5, 4, 5, 6, 7]

// Copying arrays
let original = [1, 2, 3];
let copy = [...original];
copy.push(4);

console.log(original);  // Output: [1, 2, 3] (unchanged)
console.log(copy);      // Output: [1, 2, 3, 4]

// Spread with function arguments
function sum(a, b, c) {
    return a + b + c;
}

let numbers = [1, 2, 3];
console.log(sum(...numbers));  // Output: 6

// Spread operator with objects
let person = { name: "John", age: 30 };
let address = { city: "New York", country: "USA" };

// Combining objects
let fullInfo = { ...person, ...address };
console.log(fullInfo);
// Output: { name: 'John', age: 30, city: 'New York', country: 'USA' }

// Overriding properties
let updated = { ...person, age: 31, email: "john@example.com" };
console.log(updated);
// Output: { name: 'John', age: 31, email: 'john@example.com' }

// Rest parameters in functions
function multiply(multiplier, ...numbers) {
    return numbers.map(num => num * multiplier);
}

console.log(multiply(2, 1, 2, 3, 4, 5));
// Output: [2, 4, 6, 8, 10]

// Rest in destructuring
let [first, second, ...rest] = [1, 2, 3, 4, 5];
console.log(first);   // Output: 1
console.log(second);  // Output: 2
console.log(rest);    // Output: [3, 4, 5]

let { name, ...otherInfo } = { name: "Alice", age: 25, city: "Boston" };
console.log(name);       // Output: Alice
console.log(otherInfo);  // Output: { age: 25, city: 'Boston' }
```

#### Enhanced Object Literals

```javascript
// Property shorthand
let name = "John";
let age = 30;

// Old way
let person1 = {
    name: name,
    age: age
};

// New way (property shorthand)
let person2 = {
    name,
    age
};

console.log(person2);  // Output: { name: 'John', age: 30 }

// Method shorthand
let calculator = {
    // Old way
    add: function(a, b) {
        return a + b;
    },
    // New way (method shorthand)
    subtract(a, b) {
        return a - b;
    },
    multiply(a, b) {
        return a * b;
    }
};

console.log(calculator.add(5, 3));       // Output: 8
console.log(calculator.subtract(5, 3));  // Output: 2
console.log(calculator.multiply(5, 3));  // Output: 15

// Computed property names
let propName = "email";
let user = {
    name: "Alice",
    [propName]: "alice@example.com",
    ["age_" + 2023]: 25
};

console.log(user);
// Output: { name: 'Alice', email: 'alice@example.com', age_2023: 25 }

// Dynamic method names
let methodName = "greet";
let obj = {
    [methodName]() {
        return "Hello!";
    }
};

console.log(obj.greet());  // Output: Hello!
```

#### Classes (ES6)

```javascript
// Basic class
class Person {
    constructor(name, age) {
        this.name = name;
        this.age = age;
    }
    
    greet() {
        return `Hello, my name is ${this.name}`;
    }
    
    getAge() {
        return this.age;
    }
}

// Creating instances
let person1 = new Person("John", 30);
let person2 = new Person("Jane", 25);

console.log(person1.greet());    // Output: Hello, my name is John
console.log(person2.getAge());   // Output: 25

// Static methods
class MathHelper {
    static add(a, b) {
        return a + b;
    }
    
    static multiply(a, b) {
        return a * b;
    }
}

// Call static methods on class, not instance
console.log(MathHelper.add(5, 3));       // Output: 8
console.log(MathHelper.multiply(5, 3));  // Output: 15

// Getters and setters
class Rectangle {
    constructor(width, height) {
        this.width = width;
        this.height = height;
    }
    
    // Getter
    get area() {
        return this.width * this.height;
    }
    
    // Setter
    set dimensions(value) {
        [this.width, this.height] = value;
    }
}

let rect = new Rectangle(10, 5);
console.log(rect.area);  // Output: 50 (called like property)

rect.dimensions = [20, 10];
console.log(rect.area);  // Output: 200

// Class inheritance
class Animal {
    constructor(name) {
        this.name = name;
    }
    
    speak() {
        return `${this.name} makes a sound`;
    }
}

class Dog extends Animal {
    constructor(name, breed) {
        super(name);  // Call parent constructor
        this.breed = breed;
    }
    
    speak() {
        return `${this.name} barks`;
    }
    
    getBreed() {
        return this.breed;
    }
}

let dog = new Dog("Buddy", "Golden Retriever");
console.log(dog.speak());     // Output: Buddy barks
console.log(dog.getBreed());  // Output: Golden Retriever

// Calling parent methods
class Cat extends Animal {
    speak() {
        let parentSound = super.speak();
        return `${parentSound} and meows`;
    }
}

let cat = new Cat("Whiskers");
console.log(cat.speak());  // Output: Whiskers makes a sound and meows
```

---

## Phase 2: Node.js Fundamentals

### Module 2.1: Introduction to Node.js

#### First Node.js Program

**hello.js**
```javascript
// Your first Node.js program
console.log("Hello, Node.js!");
console.log("JavaScript running on the server!");

// Multiple values
console.log("Name:", "John", "Age:", 30);

// Variables and operations
let x = 10;
let y = 20;
console.log("Sum:", x + y);

// Process object (Node.js global)
console.log("Node version:", process.version);
console.log("Platform:", process.platform);
```

**Run in terminal:**
```bash
node hello.js
```

**Output:**
```
Hello, Node.js!
JavaScript running on the server!
Name: John Age: 30
Sum: 30
Node version: v18.17.0
Platform: win32
```

#### Global Objects in Node.js

```javascript
// __dirname - Current directory path
console.log("Current directory:", __dirname);

// __filename - Current file path
console.log("Current file:", __filename);

// process - Information about current process
console.log("Process ID:", process.pid);
console.log("Working directory:", process.cwd());
console.log("Node version:", process.version);
console.log("Platform:", process.platform);
console.log("Architecture:", process.arch);

// process.argv - Command line arguments
console.log("Arguments:", process.argv);
// Run: node file.js arg1 arg2
// Output includes node path, file path, arg1, arg2

// process.env - Environment variables
console.log("User:", process.env.USER);
console.log("Home:", process.env.HOME);

// setTimeout and setInterval (same as browser)
setTimeout(() => {
    console.log("Executed after 2 seconds");
}, 2000);

let count = 0;
let interval = setInterval(() => {
    count++;
    console.log("Count:", count);
    
    if (count === 3) {
        clearInterval(interval);
        console.log("Interval stopped");
    }
}, 1000);

// setImmediate - Execute after I/O events
setImmediate(() => {
    console.log("Executed immediately after I/O");
});

// process.nextTick - Execute before next event loop
process.nextTick(() => {
    console.log("Executed in next tick");
});

console.log("Main code");

// Execution order:
// 1. Main code
// 2. process.nextTick
// 3. setImmediate
// 4. setTimeout/setInterval
```

### Module 2.2: Node.js Modules

#### CommonJS Modules

**math.js** (Module to export)
```javascript
// Single function export
function add(a, b) {
    return a + b;
}

function subtract(a, b) {
    return a - b;
}

function multiply(a, b) {
    return a * b;
}

function divide(a, b) {
    if (b === 0) {
        throw new Error("Cannot divide by zero");
    }
    return a / b;
}

// Export multiple functions
module.exports = {
    add,
    subtract,
    multiply,
    divide
};
```

**app.js** (Importing module)
```javascript
// Import entire module
const math = require('./math');

console.log(math.add(5, 3));       // Output: 8
console.log(math.subtract(5, 3));  // Output: 2
console.log(math.multiply(5, 3));  // Output: 15
console.log(math.divide(6, 3));    // Output: 2

// Destructuring import
const { add, multiply } = require('./math');

console.log(add(10, 20));      // Output: 30
console.log(multiply(4, 5));   // Output: 20
```

**user.js** (Single export)
```javascript
// Export single class
class User {
    constructor(name, email) {
        this.name = name;
        this.email = email;
    }
    
    getInfo() {
        return `${this.name} (${this.email})`;
    }
}

module.exports = User;
```

**main.js**
```javascript
const User = require('./user');

let user1 = new User("John", "john@example.com");
let user2 = new User("Jane", "jane@example.com");

console.log(user1.getInfo());  // Output: John (john@example.com)
console.log(user2.getInfo());  // Output: Jane (jane@example.com)
```

**exports-shorthand.js**
```javascript
// Using exports (reference to module.exports)
exports.greet = function(name) {
    return `Hello, ${name}!`;
};

exports.farewell = function(name) {
    return `Goodbye, ${name}!`;
};

exports.PI = 3.14159;

// Note: Don't reassign exports
// exports = { ... };  // This won't work!
// Use module.exports = { ... }; instead
```

**use-exports.js**
```javascript
const greetings = require('./exports-shorthand');

console.log(greetings.greet("Alice"));     // Output: Hello, Alice!
console.log(greetings.farewell("Bob"));    // Output: Goodbye, Bob!
console.log(greetings.PI);                  // Output: 3.14159
```

#### ES Modules (ESM)

**package.json** (Required for ES modules)
```json
{
    "type": "module"
}
```

**mathESM.mjs**
```javascript
// Named exports
export function add(a, b) {
    return a + b;
}

export function subtract(a, b) {
    return a - b;
}

export const PI = 3.14159;

// Export at the end
function multiply(a, b) {
    return a * b;
}

function divide(a, b) {
    return a / b;
}

export { multiply, divide };
```

**userESM.mjs**
```javascript
// Default export
export default class User {
    constructor(name, email) {
        this.name = name;
        this.email = email;
    }
    
    getInfo() {
        return `${this.name} (${this.email})`;
    }
}

// Can also have named exports with default
export const adminRole = 'admin';
export const userRole = 'user';
```

**appESM.mjs**
```javascript
// Named imports
import { add, subtract, PI } from './mathESM.mjs';

console.log(add(5, 3));  // Output: 8
console.log(PI);         // Output: 3.14159

// Import all as namespace
import * as math from './mathESM.mjs';

console.log(math.multiply(4, 5));  // Output: 20

// Default import (can name it anything)
import User from './userESM.mjs';
import { adminRole } from './userESM.mjs';

const user = new User("Alice", "alice@example.com");
console.log(user.getInfo());
console.log("Role:", adminRole);

// Renaming imports
import { add as sum, subtract as diff } from './mathESM.mjs';

console.log(sum(10, 5));   // Output: 15
console.log(diff(10, 5));  // Output: 5
```

### Module 2.3: Core Modules

#### File System (fs) Module

**Basic File Operations**

```javascript
const fs = require('fs');

// Synchronous file reading
try {
    const data = fs.readFileSync('file.txt', 'utf8');
    console.log("File contents:");
    console.log(data);
} catch (error) {
    console.error("Error reading file:", error.message);
}

// Asynchronous file reading (callback)
fs.readFile('file.txt', 'utf8', (error, data) => {
    if (error) {
        console.error("Error:", error.message);
        return;
    }
    console.log("File contents:");
    console.log(data);
});

// Synchronous file writing
try {
    fs.writeFileSync('output.txt', 'Hello from Node.js!');
    console.log("File written successfully");
} catch (error) {
    console.error("Error writing file:", error.message);
}

// Asynchronous file writing
fs.writeFile('output.txt', 'Hello, World!', 'utf8', (error) => {
    if (error) {
        console.error("Error:", error.message);
        return;
    }
    console.log("File written successfully");
});

// Appending to file
fs.appendFile('log.txt', 'New log entry\n', (error) => {
    if (error) {
        console.error("Error:", error.message);
        return;
    }
    console.log("Data appended");
});
```

**File System Operations**

```javascript
const fs = require('fs');
const path = require('path');

// Check if file exists
if (fs.existsSync('file.txt')) {
    console.log("File exists");
} else {
    console.log("File does not exist");
}

// Get file information
fs.stat('file.txt', (error, stats) => {
    if (error) {
        console.error("Error:", error.message);
        return;
    }
    
    console.log("File size:", stats.size, "bytes");
    console.log("Is file:", stats.isFile());
    console.log("Is directory:", stats.isDirectory());
    console.log("Created:", stats.birthtime);
    console.log("Modified:", stats.mtime);
});

// Delete file
fs.unlink('temp.txt', (error) => {
    if (error) {
        console.error("Error deleting file:", error.message);
        return;
    }
    console.log("File deleted");
});

// Rename/Move file
fs.rename('old-name.txt', 'new-name.txt', (error) => {
    if (error) {
        console.error("Error:", error.message);
        return;
    }
    console.log("File renamed");
});

// Copy file
fs.copyFile('source.txt', 'destination.txt', (error) => {
    if (error) {
        console.error("Error:", error.message);
        return;
    }
    console.log("File copied");
});
```

**Directory Operations**

```javascript
const fs = require('fs');
const path = require('path');

// Create directory
fs.mkdir('new-folder', (error) => {
    if (error) {
        console.error("Error creating directory:", error.message);
        return;
    }
    console.log("Directory created");
});

// Create nested directories
fs.mkdir('parent/child/grandchild', { recursive: true }, (error) => {
    if (error) {
        console.error("Error:", error.message);
        return;
    }
    console.log("Nested directories created");
});

// Read directory contents
fs.readdir('./', (error, files) => {
    if (error) {
        console.error("Error:", error.message);
        return;
    }
    
    console.log("Files in current directory:");
    files.forEach(file => {
        console.log(file);
    });
});

// Read directory with file types
fs.readdir('./', { withFileTypes: true }, (error, entries) => {
    if (error) {
        console.error("Error:", error.message);
        return;
    }
    
    entries.forEach(entry => {
        if (entry.isFile()) {
            console.log("File:", entry.name);
        } else if (entry.isDirectory()) {
            console.log("Directory:", entry.name);
        }
    });
});

// Remove directory (must be empty)
fs.rmdir('old-folder', (error) => {
    if (error) {
        console.error("Error:", error.message);
        return;
    }
    console.log("Directory removed");
});

// Remove directory recursively (with contents)
fs.rm('folder-with-files', { recursive: true, force: true }, (error) => {
    if (error) {
        console.error("Error:", error.message);
        return;
    }
    console.log("Directory and contents removed");
});
```

**Promises API (fs/promises)**

```javascript
const fs = require('fs/promises');

// Read file with promises
fs.readFile('file.txt', 'utf8')
    .then(data => {
        console.log("File contents:", data);
    })
    .catch(error => {
        console.error("Error:", error.message);
    });

// Using async/await
async function fileOperations() {
    try {
        const data = await fs.readFile('input.txt', 'utf8');
        console.log("Read:", data);
        
        await fs.writeFile('output.txt', data.toUpperCase());
        console.log("Written successfully");
        
        const stats = await fs.stat('output.txt');
        console.log("File size:", stats.size, "bytes");
        
        const files = await fs.readdir('./');
        console.log("Files:", files);
        
    } catch (error) {
        console.error("Error:", error.message);
    }
}

fileOperations();
```

#### Path Module

```javascript
const path = require('path');

// Join paths
const fullPath = path.join('folder', 'subfolder', 'file.txt');
console.log(fullPath);  // folder/subfolder/file.txt

// Resolve absolute path
const absolutePath = path.resolve('folder', 'file.txt');
console.log(absolutePath);  // /full/path/to/folder/file.txt

// Get directory name
const dirname = path.dirname('/path/to/file.txt');
console.log(dirname);  // /path/to

// Get file name
const basename = path.basename('/path/to/file.txt');
console.log(basename);  // file.txt

// Get file extension
const extname = path.extname('/path/to/file.txt');
console.log(extname);  // .txt

// Parse path
const parsed = path.parse('/path/to/file.txt');
console.log(parsed);
// {
//   root: '/',
//   dir: '/path/to',
//   base: 'file.txt',
//   ext: '.txt',
//   name: 'file'
// }

// Normalize path
const normalized = path.normalize('/path//to/../file.txt');
console.log(normalized);  // /path/file.txt
```

#### OS Module

```javascript
const os = require('os');

// Platform information
console.log("Platform:", os.platform());     // 'linux', 'darwin', 'win32'
console.log("Architecture:", os.arch());     // 'x64', 'arm'
console.log("CPU cores:", os.cpus().length);

// Memory information
const totalMem = os.totalmem();
const freeMem = os.freemem();

console.log("Total memory:", (totalMem / 1024 / 1024 / 1024).toFixed(2), "GB");
console.log("Free memory:", (freeMem / 1024 / 1024 / 1024).toFixed(2), "GB");

// User information
console.log("Username:", os.userInfo().username);
console.log("Home directory:", os.homedir());
console.log("Hostname:", os.hostname());

// System uptime
console.log("System uptime:", os.uptime(), "seconds");

// Temporary directory
console.log("Temp directory:", os.tmpdir());
```

---

## Phase 3: Asynchronous Programming

### Module 3.1: Callbacks

#### Understanding Callbacks

**Description:** Callbacks are functions passed as arguments to other functions, executed after an operation completes.

```javascript
// Simple callback example
function greet(name, callback) {
    console.log("Hello, " + name);
    callback();
}

function sayGoodbye() {
    console.log("Goodbye!");
}

greet("Alice", sayGoodbye);
// Output: Hello, Alice
//         Goodbye!

// Error-first callbacks (Node.js convention)
function divide(a, b, callback) {
    if (b === 0) {
        callback(new Error("Division by zero"), null);
    } else {
        callback(null, a / b);
    }
}

divide(10, 2, (error, result) => {
    if (error) {
        console.error("Error:", error.message);
    } else {
        console.log("Result:", result);  // Output: Result: 5
    }
});
```

**Callback Hell (Pyramid of Doom):**

```javascript
const fs = require('fs');

// Callback hell - hard to read and maintain
fs.readFile('file1.txt', 'utf8', (err1, data1) => {
    if (err1) return console.error(err1);
    
    fs.readFile('file2.txt', 'utf8', (err2, data2) => {
        if (err2) return console.error(err2);
        
        fs.readFile('file3.txt', 'utf8', (err3, data3) => {
            if (err3) return console.error(err3);
            
            const combined = data1 + data2 + data3;
            
            fs.writeFile('output.txt', combined, (err4) => {
                if (err4) return console.error(err4);
                console.log("Files combined successfully");
            });
        });
    });
});
```

### Module 3.2: Promises

#### Promise Basics

**Description:** Promises represent the eventual completion (or failure) of an asynchronous operation.

```javascript
// Creating a promise
const myPromise = new Promise((resolve, reject) => {
    const success = true;
    
    setTimeout(() => {
        if (success) {
            resolve("Operation successful!");
        } else {
            reject("Operation failed!");
        }
    }, 1000);
});

// Consuming a promise
myPromise
    .then(result => {
        console.log(result);  // Output: Operation successful!
    })
    .catch(error => {
        console.error(error);
    })
    .finally(() => {
        console.log("Promise settled");
    });

// Practical example
function fetchUser(userId) {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            if (userId > 0) {
                resolve({ id: userId, name: "John Doe" });
            } else {
                reject(new Error("Invalid user ID"));
            }
        }, 1000);
    });
}

fetchUser(1)
    .then(user => {
        console.log("User:", user);
        return user.id;
    })
    .then(userId => {
        console.log("User ID:", userId);
    })
    .catch(error => {
        console.error("Error:", error.message);
    });
```

#### Promise Static Methods

```javascript
// Promise.all() - Wait for all promises
const promise1 = Promise.resolve(3);
const promise2 = new Promise(resolve => setTimeout(() => resolve(5), 1000));
const promise3 = Promise.resolve(7);

Promise.all([promise1, promise2, promise3])
    .then(results => {
        console.log(results);  // Output: [3, 5, 7]
        const sum = results.reduce((a, b) => a + b);
        console.log("Sum:", sum);  // Output: Sum: 15
    })
    .catch(error => {
        console.error("One promise failed:", error);
    });

// Promise.race() - First to settle wins
Promise.race([
    new Promise(resolve => setTimeout(() => resolve('Slow'), 2000)),
    new Promise(resolve => setTimeout(() => resolve('Fast'), 500))
])
    .then(result => {
        console.log(result);  // Output: Fast
    });
```

### Module 3.3: Async/Await

#### Async/Await Basics

**Description:** Async/await provides syntactic sugar for promises, making asynchronous code look synchronous.

```javascript
// Async function returns a promise
async function greet() {
    return "Hello, World!";
}

greet().then(message => console.log(message));

// Await waits for promise to resolve
async function fetchData() {
    const promise = new Promise(resolve => {
        setTimeout(() => resolve("Data loaded"), 1000);
    });
    
    console.log("Fetching data...");
    const result = await promise;
    console.log(result);
    return result;
}

fetchData();

// Error handling with try-catch
async function processFiles() {
    const fs = require('fs/promises');
    
    try {
        const file1 = await fs.readFile('file1.txt', 'utf8');
        const file2 = await fs.readFile('file2.txt', 'utf8');
        const file3 = await fs.readFile('file3.txt', 'utf8');
        
        const combined = file1 + file2 + file3;
        await fs.writeFile('output.txt', combined);
        
        console.log("Files combined successfully");
    } catch (error) {
        console.error("Error:", error.message);
    }
}

processFiles();

// Parallel execution
async function readFilesParallel() {
    const fs = require('fs/promises');
    
    const [file1, file2, file3] = await Promise.all([
        fs.readFile('file1.txt', 'utf8'),
        fs.readFile('file2.txt', 'utf8'),
        fs.readFile('file3.txt', 'utf8')
    ]);
    
    return [file1, file2, file3];
}
```

---

**[Tutorial continues with Express.js, Databases, Authentication, Testing, Advanced Topics, and Production Deployment in following phases...]**

---

## Conclusion

This Node.js Mastery Tutorial covers:

**Phase 1 - JavaScript Essentials:**
- ✅ Variables, data types, operators
- ✅ Control flow (if, switch, loops)
- ✅ Functions, scope, closures
- ✅ Objects and arrays
- ✅ Modern JavaScript (ES6+)
- ✅ Classes and OOP

**Phase 2 - Node.js Fundamentals:**
- ✅ Node.js basics and global objects
- ✅ CommonJS and ES modules
- ✅ File system operations
- ✅ Path and OS modules

**Phase 3 - Asynchronous Programming:**
- ✅ Callbacks and callback hell
- ✅ Promises and promise chaining
- ✅ Async/await patterns
- ✅ Error handling

**Next Steps:**

To complete your Node.js ninja journey, continue with:

**Phase 4 - Express.js:**
- Building web servers
- Routing and middleware
- REST API development
- Template engines

**Phase 5 - Database Integration:**
- MongoDB with Mongoose
- PostgreSQL with Sequelize
- CRUD operations
- Data validation

**Phase 6 - Authentication & Security:**
- JWT authentication
- Password hashing
- OAuth integration
- Security best practices

**Phase 7 - Testing:**
- Unit testing with Jest
- Integration testing
- API testing with Supertest
- Test-driven development

**Phase 8 - Advanced Topics:**
- Streams and buffers
- Worker threads
- Clustering
- WebSockets
- GraphQL
- Microservices

**Phase 9 - Production & Deployment:**
- Environment configuration
- Logging and monitoring
- Docker containerization
- CI/CD pipelines
- Cloud deployment

**Practice Projects:**
1. RESTful API with authentication
2. Real-time chat application
3. E-commerce backend
4. Blog platform
5. Task management system

**Resources:**
- Official Node.js documentation
- MDN JavaScript guide
- Express.js documentation
- MongoDB University
- Node.js design patterns

Happy coding! 🚀

---

## Phase 4: Express.js Framework

### Module 4.1: Express Basics

#### Setting Up Express

**Description:** Express.js is a minimal and flexible Node.js web application framework that provides robust features for web and mobile applications.

**Installation:**
```bash
npm init -y
npm install express
```

**Basic Server:**

```javascript
// server.js
const express = require('express');
const app = express();
const PORT = 3000;

// Basic route - responds to GET request on root path
app.get('/', (req, res) => {
    res.send('Hello, Express!');
});

// Start server and listen on port 3000
app.listen(PORT, () => {
    console.log(`Server running on http://localhost:${PORT}`);
});
```

**Run:**
```bash
node server.js
# Visit http://localhost:3000 in browser
```

#### Routes and HTTP Methods

```javascript
const express = require('express');
const app = express();

// Middleware to parse JSON bodies
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// GET request - Retrieve data
app.get('/users', (req, res) => {
    res.json([
        { id: 1, name: 'John' },
        { id: 2, name: 'Jane' }
    ]);
});

// GET with route parameters
app.get('/users/:id', (req, res) => {
    const userId = req.params.id;  // Extract parameter from URL
    res.json({ id: userId, name: 'John Doe' });
});

// Query parameters: /search?q=nodejs&page=1
app.get('/search', (req, res) => {
    const { q, page, limit } = req.query;
    res.json({
        query: q,
        page: page || 1,
        limit: limit || 10
    });
});

// POST request - Create new resource
app.post('/users', (req, res) => {
    const newUser = req.body;  // Data from request body
    // In real app: save to database
    res.status(201).json({
        message: 'User created',
        user: newUser
    });
});

// PUT request - Update entire resource
app.put('/users/:id', (req, res) => {
    const userId = req.params.id;
    const updates = req.body;
    
    res.json({
        message: 'User updated',
        id: userId,
        updates
    });
});

// PATCH request - Partial update
app.patch('/users/:id', (req, res) => {
    const userId = req.params.id;
    const updates = req.body;
    
    res.json({
        message: 'User partially updated',
        id: userId,
        updates
    });
});

// DELETE request - Remove resource
app.delete('/users/:id', (req, res) => {
    const userId = req.params.id;
    // In real app: delete from database
    res.json({
        message: 'User deleted',
        id: userId
    });
});

app.listen(3000);
```

### Module 4.2: Middleware

#### Understanding Middleware

**Description:** Middleware functions have access to request (req), response (res), and next function. They execute in sequence and can modify request/response or end the cycle.

```javascript
const express = require('express');
const app = express();

// Application-level middleware - runs for ALL requests
app.use((req, res, next) => {
    console.log(`${req.method} ${req.url} - ${new Date().toISOString()}`);
    next();  // Pass control to next middleware
});

// Route-specific middleware
const checkAuth = (req, res, next) => {
    const token = req.headers.authorization;
    
    if (token === 'Bearer secret-token') {
        next();  // Authorized, proceed
    } else {
        res.status(401).json({ error: 'Unauthorized' });
    }
};

// Apply middleware to specific route
app.get('/protected', checkAuth, (req, res) => {
    res.json({ message: 'You are authenticated!' });
});

// Multiple middleware functions
const logger = (req, res, next) => {
    console.log('Logger middleware');
    next();
};

const validator = (req, res, next) => {
    console.log('Validator middleware');
    next();
};

app.get('/multi', [logger, validator], (req, res) => {
    res.send('Response after multiple middleware');
});

// Error-handling middleware (must have 4 parameters)
app.use((err, req, res, next) => {
    console.error('Error:', err.message);
    res.status(500).json({
        error: 'Internal Server Error',
        message: err.message
    });
});

app.listen(3000);
```

#### Built-in and Third-party Middleware

```bash
npm install morgan cors helmet compression cookie-parser
```

```javascript
const express = require('express');
const morgan = require('morgan');         // Logging
const cors = require('cors');             // Cross-Origin Resource Sharing
const helmet = require('helmet');         // Security headers
const compression = require('compression'); // Gzip compression
const cookieParser = require('cookie-parser');

const app = express();

// Built-in middleware
app.use(express.json());  // Parse JSON bodies
app.use(express.urlencoded({ extended: true }));  // Parse URL-encoded bodies
app.use(express.static('public'));  // Serve static files from 'public' folder

// Third-party middleware
app.use(morgan('dev'));  // Log HTTP requests
app.use(helmet());       // Set security headers
app.use(cors());         // Enable CORS
app.use(compression());  // Compress responses
app.use(cookieParser()); // Parse cookies

// Cookie operations
app.get('/set-cookie', (req, res) => {
    res.cookie('username', 'john', { maxAge: 900000, httpOnly: true });
    res.send('Cookie set');
});

app.get('/get-cookie', (req, res) => {
    res.json({ cookies: req.cookies });
});

app.listen(3000);
```

### Module 4.3: Express Router

**routes/users.js**
```javascript
const express = require('express');
const router = express.Router();

// Router-level middleware
router.use((req, res, next) => {
    console.log('Users route accessed');
    next();
});

// GET /api/users
router.get('/', (req, res) => {
    res.json([
        { id: 1, name: 'John' },
        { id: 2, name: 'Jane' }
    ]);
});

// GET /api/users/:id
router.get('/:id', (req, res) => {
    res.json({ id: req.params.id, name: 'John Doe' });
});

// POST /api/users
router.post('/', (req, res) => {
    res.status(201).json({ message: 'User created', user: req.body });
});

// PUT /api/users/:id
router.put('/:id', (req, res) => {
    res.json({ message: 'User updated', id: req.params.id });
});

// DELETE /api/users/:id
router.delete('/:id', (req, res) => {
    res.json({ message: 'User deleted', id: req.params.id });
});

module.exports = router;
```

**server.js**
```javascript
const express = require('express');
const app = express();

app.use(express.json());

// Import routers
const usersRouter = require('./routes/users');
const productsRouter = require('./routes/products');

// Mount routers
app.use('/api/users', usersRouter);
app.use('/api/products', productsRouter);

// 404 handler
app.use((req, res) => {
    res.status(404).json({ error: 'Route not found' });
});

app.listen(3000, () => {
    console.log('Server running on port 3000');
});
```

---

## Phase 5: Database Integration

### Module 5.1: MongoDB with Mongoose

#### Setting Up Mongoose

**Installation:**
```bash
npm install mongoose
```

**Connection:**

```javascript
// config/database.js
const mongoose = require('mongoose');

const connectDB = async () => {
    try {
        await mongoose.connect('mongodb://localhost:27017/myapp', {
            useNewUrlParser: true,
            useUnifiedTopology: true
        });
        console.log('MongoDB connected successfully');
    } catch (error) {
        console.error('MongoDB connection error:', error);
        process.exit(1);
    }
};

module.exports = connectDB;
```

#### Mongoose Schema and Model

```javascript
// models/User.js
const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({
    name: {
        type: String,
        required: [true, 'Name is required'],
        trim: true,
        minlength: [3, 'Name must be at least 3 characters'],
        maxlength: [50, 'Name cannot exceed 50 characters']
    },
    email: {
        type: String,
        required: true,
        unique: true,
        lowercase: true,
        match: [/^\S+@\S+\.\S+$/, 'Please enter valid email']
    },
    password: {
        type: String,
        required: true,
        minlength: 6
    },
    age: {
        type: Number,
        min: [18, 'Must be at least 18'],
        max: [100, 'Age cannot exceed 100']
    },
    role: {
        type: String,
        enum: ['user', 'admin', 'moderator'],
        default: 'user'
    },
    isActive: {
        type: Boolean,
        default: true
    }
}, {
    timestamps: true  // Adds createdAt and updatedAt
});

// Instance method
userSchema.methods.getPublicProfile = function() {
    return {
        id: this._id,
        name: this.name,
        email: this.email
    };
};

// Static method
userSchema.statics.findByEmail = function(email) {
    return this.findOne({ email: email.toLowerCase() });
};

// Pre-save middleware
userSchema.pre('save', async function(next) {
    if (this.isModified('password')) {
        const bcrypt = require('bcrypt');
        this.password = await bcrypt.hash(this.password, 10);
    }
    next();
});

const User = mongoose.model('User', userSchema);
module.exports = User;
```

#### CRUD Operations

```javascript
const User = require('./models/User');

// CREATE
async function createUser() {
    try {
        // Method 1: Using save()
        const user = new User({
            name: 'John Doe',
            email: 'john@example.com',
            password: 'password123',
            age: 30
        });
        await user.save();
        
        // Method 2: Using create()
        const user2 = await User.create({
            name: 'Jane Smith',
            email: 'jane@example.com',
            password: 'password456',
            age: 25
        });
        
        console.log('Users created');
    } catch (error) {
        console.error('Error:', error.message);
    }
}

// READ
async function readUsers() {
    try {
        // Find all
        const allUsers = await User.find();
        
        // Find with conditions
        const activeUsers = await User.find({ isActive: true });
        
        // Find one
        const user = await User.findOne({ email: 'john@example.com' });
        
        // Find by ID
        const userById = await User.findById('60f7b3b3b3b3b3b3b3b3b3b3');
        
        // With options
        const users = await User.find({ age: { $gte: 25 } })
            .select('name email age')  // Select specific fields
            .limit(10)
            .skip(0)
            .sort({ name: 1 })  // Sort ascending
            .exec();
        
        // Count
        const count = await User.countDocuments({ isActive: true });
        
    } catch (error) {
        console.error('Error:', error.message);
    }
}

// UPDATE
async function updateUsers() {
    try {
        // Update one
        await User.updateOne(
            { email: 'john@example.com' },
            { age: 31 }
        );
        
        // Find and update (returns updated document)
        const user = await User.findOneAndUpdate(
            { email: 'john@example.com' },
            { age: 32 },
            { new: true, runValidators: true }
        );
        
    } catch (error) {
        console.error('Error:', error.message);
    }
}

// DELETE
async function deleteUsers() {
    try {
        // Delete one
        await User.deleteOne({ email: 'john@example.com' });
        
        // Find and delete
        const deletedUser = await User.findOneAndDelete(
            { email: 'john@example.com' }
        );
        
    } catch (error) {
        console.error('Error:', error.message);
    }
}
```

### Module 5.2: RESTful API with Express and MongoDB

```javascript
// controllers/userController.js
const User = require('../models/User');

// Get all users
exports.getAllUsers = async (req, res) => {
    try {
        const users = await User.find().select('-password');
        res.json({
            success: true,
            count: users.length,
            data: users
        });
    } catch (error) {
        res.status(500).json({
            success: false,
            error: error.message
        });
    }
};

// Get single user
exports.getUser = async (req, res) => {
    try {
        const user = await User.findById(req.params.id).select('-password');
        
        if (!user) {
            return res.status(404).json({
                success: false,
                error: 'User not found'
            });
        }
        
        res.json({
            success: true,
            data: user
        });
    } catch (error) {
        res.status(500).json({
            success: false,
            error: error.message
        });
    }
};

// Create user
exports.createUser = async (req, res) => {
    try {
        const user = await User.create(req.body);
        
        res.status(201).json({
            success: true,
            data: user
        });
    } catch (error) {
        res.status(400).json({
            success: false,
            error: error.message
        });
    }
};

// Update user
exports.updateUser = async (req, res) => {
    try {
        const user = await User.findByIdAndUpdate(
            req.params.id,
            req.body,
            { new: true, runValidators: true }
        );
        
        if (!user) {
            return res.status(404).json({
                success: false,
                error: 'User not found'
            });
        }
        
        res.json({
            success: true,
            data: user
        });
    } catch (error) {
        res.status(400).json({
            success: false,
            error: error.message
        });
    }
};

// Delete user
exports.deleteUser = async (req, res) => {
    try {
        const user = await User.findByIdAndDelete(req.params.id);
        
        if (!user) {
            return res.status(404).json({
                success: false,
                error: 'User not found'
            });
        }
        
        res.json({
            success: true,
            data: {}
        });
    } catch (error) {
        res.status(500).json({
            success: false,
            error: error.message
        });
    }
};
```

**Routes:**
```javascript
// routes/users.js
const express = require('express');
const router = express.Router();
const {
    getAllUsers,
    getUser,
    createUser,
    updateUser,
    deleteUser
} = require('../controllers/userController');

router.route('/')
    .get(getAllUsers)
    .post(createUser);

router.route('/:id')
    .get(getUser)
    .put(updateUser)
    .delete(deleteUser);

module.exports = router;
```

---

## Phase 6: Authentication & Security

### Module 6.1: JWT Authentication

**Installation:**
```bash
npm install jsonwebtoken bcrypt dotenv
```

**User Registration:**

```javascript
// controllers/authController.js
const User = require('../models/User');
const jwt = require('jsonwebtoken');
const bcrypt = require('bcrypt');

// Register new user
exports.register = async (req, res) => {
    try {
        const { name, email, password } = req.body;
        
        // Check if user exists
        const existingUser = await User.findOne({ email });
        if (existingUser) {
            return res.status(400).json({
                success: false,
                error: 'User already exists'
            });
        }
        
        // Create user (password will be hashed by pre-save hook)
        const user = await User.create({
            name,
            email,
            password
        });
        
        // Generate JWT token
        const token = jwt.sign(
            { id: user._id, email: user.email },
            process.env.JWT_SECRET,
            { expiresIn: '7d' }
        );
        
        res.status(201).json({
            success: true,
            token,
            user: {
                id: user._id,
                name: user.name,
                email: user.email
            }
        });
    } catch (error) {
        res.status(500).json({
            success: false,
            error: error.message
        });
    }
};

// Login user
exports.login = async (req, res) => {
    try {
        const { email, password } = req.body;
        
        // Validate input
        if (!email || !password) {
            return res.status(400).json({
                success: false,
                error: 'Please provide email and password'
            });
        }
        
        // Find user
        const user = await User.findOne({ email });
        if (!user) {
            return res.status(401).json({
                success: false,
                error: 'Invalid credentials'
            });
        }
        
        // Check password
        const isPasswordValid = await bcrypt.compare(password, user.password);
        if (!isPasswordValid) {
            return res.status(401).json({
                success: false,
                error: 'Invalid credentials'
            });
        }
        
        // Generate token
        const token = jwt.sign(
            { id: user._id, email: user.email },
            process.env.JWT_SECRET,
            { expiresIn: '7d' }
        );
        
        res.json({
            success: true,
            token,
            user: {
                id: user._id,
                name: user.name,
                email: user.email
            }
        });
    } catch (error) {
        res.status(500).json({
            success: false,
            error: error.message
        });
    }
};
```

**Authentication Middleware:**

```javascript
// middleware/auth.js
const jwt = require('jsonwebtoken');
const User = require('../models/User');

exports.protect = async (req, res, next) => {
    try {
        let token;
        
        // Get token from header
        if (req.headers.authorization &&
            req.headers.authorization.startsWith('Bearer')) {
            token = req.headers.authorization.split(' ')[1];
        }
        
        // Check if token exists
        if (!token) {
            return res.status(401).json({
                success: false,
                error: 'Not authorized to access this route'
            });
        }
        
        // Verify token
        const decoded = jwt.verify(token, process.env.JWT_SECRET);
        
        // Find user
        req.user = await User.findById(decoded.id).select('-password');
        
        if (!req.user) {
            return res.status(401).json({
                success: false,
                error: 'User not found'
            });
        }
        
        next();
    } catch (error) {
        res.status(401).json({
            success: false,
            error: 'Not authorized'
        });
    }
};

// Role-based authorization
exports.authorize = (...roles) => {
    return (req, res, next) => {
        if (!roles.includes(req.user.role)) {
            return res.status(403).json({
                success: false,
                error: `User role ${req.user.role} is not authorized`
            });
        }
        next();
    };
};
```

**Protected Routes:**

```javascript
// routes/users.js
const express = require('express');
const router = express.Router();
const { protect, authorize } = require('../middleware/auth');
const {
    getAllUsers,
    getUser,
    updateUser,
    deleteUser
} = require('../controllers/userController');

// Public routes
// ...

// Protected routes (require authentication)
router.use(protect);  // All routes below require authentication

router.get('/profile', (req, res) => {
    res.json({
        success: true,
        data: req.user
    });
});

// Admin-only routes
router.get('/admin/users', authorize('admin'), getAllUsers);
router.delete('/admin/users/:id', authorize('admin'), deleteUser);

module.exports = router;
```

### Module 6.2: Security Best Practices

```javascript
// Security packages
const helmet = require('helmet');         // Security headers
const mongoSanitize = require('express-mongo-sanitize');  // Prevent NoSQL injection
const xss = require('xss-clean');        // Prevent XSS attacks
const rateLimit = require('express-rate-limit');  // Rate limiting
const hpp = require('hpp');              // Prevent parameter pollution

const app = express();

// Set security headers
app.use(helmet());

// Prevent NoSQL injection
app.use(mongoSanitize());

// Prevent XSS attacks
app.use(xss());

// Rate limiting
const limiter = rateLimit({
    windowMs: 10 * 60 * 1000,  // 10 minutes
    max: 100  // Limit each IP to 100 requests per windowMs
});
app.use('/api', limiter);

// Prevent parameter pollution
app.use(hpp());

// CORS configuration
const cors = require('cors');
app.use(cors({
    origin: 'https://yourdomain.com',
    credentials: true,
    optionsSuccessStatus: 200
}));
```

---

## Phase 7: Testing & Debugging

### Module 7.1: Unit Testing with Jest

**Installation:**
```bash
npm install --save-dev jest supertest
```

**package.json:**
```json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch"
  },
  "jest": {
    "testEnvironment": "node",
    "coveragePathIgnorePatterns": ["/node_modules/"]
  }
}
```

**Unit Tests:**

```javascript
// __tests__/utils.test.js
const { add, subtract, multiply, divide } = require('../utils/math');

describe('Math Utilities', () => {
    // Test add function
    test('add() should add two numbers', () => {
        expect(add(2, 3)).toBe(5);
        expect(add(-1, 1)).toBe(0);
        expect(add(0, 0)).toBe(0);
    });
    
    // Test subtract function
    test('subtract() should subtract two numbers', () => {
        expect(subtract(5, 3)).toBe(2);
        expect(subtract(0, 5)).toBe(-5);
    });
    
    // Test multiply function
    test('multiply() should multiply two numbers', () => {
        expect(multiply(3, 4)).toBe(12);
        expect(multiply(-2, 3)).toBe(-6);
    });
    
    // Test divide function
    test('divide() should divide two numbers', () => {
        expect(divide(10, 2)).toBe(5);
        expect(divide(7, 2)).toBe(3.5);
    });
    
    test('divide() should throw error for division by zero', () => {
        expect(() => divide(5, 0)).toThrow('Cannot divide by zero');
    });
});

// Async tests
describe('Async Functions', () => {
    test('fetchUser() should return user data', async () => {
        const user = await fetchUser(1);
        expect(user).toHaveProperty('id', 1);
        expect(user).toHaveProperty('name');
    });
});
```

### Module 7.2: API Testing with Supertest

```javascript
// __tests__/api/users.test.js
const request = require('supertest');
const app = require('../app');
const User = require('../models/User');
const mongoose = require('mongoose');

// Setup and teardown
beforeAll(async () => {
    await mongoose.connect(process.env.TEST_DATABASE_URL);
});

afterAll(async () => {
    await mongoose.connection.close();
});

beforeEach(async () => {
    await User.deleteMany({});  // Clear database before each test
});

describe('User API', () => {
    // Test GET /api/users
    test('GET /api/users should return all users', async () => {
        // Create test users
        await User.create([
            { name: 'John', email: 'john@test.com', password: 'pass123' },
            { name: 'Jane', email: 'jane@test.com', password: 'pass456' }
        ]);
        
        const response = await request(app)
            .get('/api/users')
            .expect('Content-Type', /json/)
            .expect(200);
        
        expect(response.body.success).toBe(true);
        expect(response.body.data).toHaveLength(2);
    });
    
    // Test POST /api/users
    test('POST /api/users should create new user', async () => {
        const newUser = {
            name: 'Test User',
            email: 'test@example.com',
            password: 'password123'
        };
        
        const response = await request(app)
            .post('/api/users')
            .send(newUser)
            .expect('Content-Type', /json/)
            .expect(201);
        
        expect(response.body.success).toBe(true);
        expect(response.body.data.name).toBe(newUser.name);
        expect(response.body.data.email).toBe(newUser.email);
    });
    
    // Test GET /api/users/:id
    test('GET /api/users/:id should return single user', async () => {
        const user = await User.create({
            name: 'John',
            email: 'john@test.com',
            password: 'pass123'
        });
        
        const response = await request(app)
            .get(`/api/users/${user._id}`)
            .expect(200);
        
        expect(response.body.data.name).toBe('John');
    });
    
    // Test error handling
    test('GET /api/users/:id should return 404 for invalid ID', async () => {
        const fakeId = new mongoose.Types.ObjectId();
        
        const response = await request(app)
            .get(`/api/users/${fakeId}`)
            .expect(404);
        
        expect(response.body.success).toBe(false);
    });
});

// Test authentication
describe('Authentication API', () => {
    test('POST /api/auth/register should create user and return token', async () => {
        const newUser = {
            name: 'Test User',
            email: 'test@example.com',
            password: 'password123'
        };
        
        const response = await request(app)
            .post('/api/auth/register')
            .send(newUser)
            .expect(201);
        
        expect(response.body.success).toBe(true);
        expect(response.body.token).toBeDefined();
    });
    
    test('POST /api/auth/login should return token for valid credentials', async () => {
        // Create user first
        await User.create({
            name: 'John',
            email: 'john@test.com',
            password: 'password123'
        });
        
        const response = await request(app)
            .post('/api/auth/login')
            .send({
                email: 'john@test.com',
                password: 'password123'
            })
            .expect(200);
        
        expect(response.body.success).toBe(true);
        expect(response.body.token).toBeDefined();
    });
    
    test('Protected route should require authentication', async () => {
        await request(app)
            .get('/api/users/profile')
            .expect(401);
    });
    
    test('Protected route should work with valid token', async () => {
        // Register and get token
        const registerRes = await request(app)
            .post('/api/auth/register')
            .send({
                name: 'John',
                email: 'john@test.com',
                password: 'password123'
            });
        
        const token = registerRes.body.token;
        
        // Access protected route
        const response = await request(app)
            .get('/api/users/profile')
            .set('Authorization', `Bearer ${token}`)
            .expect(200);
        
        expect(response.body.success).toBe(true);
    });
});
```

### Module 7.3: Debugging Techniques

**Using Node.js Debugger:**

```javascript
// app.js
const express = require('express');
const app = express();

app.get('/debug', (req, res) => {
    const data = { name: 'John', age: 30 };
    
    debugger;  // Debugger will pause here
    
    const result = processData(data);
    res.json(result);
});

function processData(data) {
    debugger;  // Another breakpoint
    
    return {
        ...data,
        timestamp: new Date()
    };
}
```

**Run with debugger:**
```bash
# Node.js built-in debugger
node inspect app.js

# Chrome DevTools
node --inspect app.js
# Open chrome://inspect in Chrome

# VS Code debugging
# Use F5 or Debug panel
```

**Debug Configuration (VS Code):**
```json
// .vscode/launch.json
{
    "version": "0.2.0",
    "configurations": [
        {
            "type": "node",
            "request": "launch",
            "name": "Launch Program",
            "skipFiles": ["<node_internals>/**"],
            "program": "${workspaceFolder}/server.js",
            "env": {
                "NODE_ENV": "development"
            }
        }
    ]
}
```

**Logging Best Practices:**

```javascript
// Using Winston for logging
const winston = require('winston');

const logger = winston.createLogger({
    level: 'info',
    format: winston.format.combine(
        winston.format.timestamp(),
        winston.format.errors({ stack: true }),
        winston.format.json()
    ),
    transports: [
        // Write to file
        new winston.transports.File({ filename: 'error.log', level: 'error' }),
        new winston.transports.File({ filename: 'combined.log' })
    ]
});

// Add console logging in development
if (process.env.NODE_ENV !== 'production') {
    logger.add(new winston.transports.Console({
        format: winston.format.simple()
    }));
}

// Usage
logger.info('Server started on port 3000');
logger.warn('This is a warning');
logger.error('Error occurred', { error: err.message });
```

---

## Phase 8: Advanced Topics

### Module 8.1: Streams

**Description:** Streams allow processing data piece by piece without loading entire content into memory. Perfect for large files.

```javascript
const fs = require('fs');
const { Transform } = require('stream');

// Readable stream - Reading large file
const readStream = fs.createReadStream('large-file.txt', {
    encoding: 'utf8',
    highWaterMark: 16 * 1024  // 16KB chunks
});

readStream.on('data', (chunk) => {
    console.log('Received chunk:', chunk.length, 'bytes');
});

readStream.on('end', () => {
    console.log('Finished reading file');
});

readStream.on('error', (error) => {
    console.error('Error reading file:', error);
});

// Writable stream - Writing to file
const writeStream = fs.createWriteStream('output.txt');

writeStream.write('Hello, ');
writeStream.write('World!');
writeStream.end();  // Close stream

writeStream.on('finish', () => {
    console.log('Finished writing');
});

// Piping streams - Copy file efficiently
const source = fs.createReadStream('source.txt');
const destination = fs.createWriteStream('destination.txt');

source.pipe(destination);

source.on('end', () => {
    console.log('File copied successfully');
});

// Transform stream - Modify data while streaming
class UpperCaseTransform extends Transform {
    _transform(chunk, encoding, callback) {
        // Convert chunk to uppercase
        const upperChunk = chunk.toString().toUpperCase();
        this.push(upperChunk);
        callback();
    }
}

const upperTransform = new UpperCaseTransform();

fs.createReadStream('input.txt')
    .pipe(upperTransform)
    .pipe(fs.createWriteStream('output-upper.txt'));

// HTTP response with streams
const http = require('http');

http.createServer((req, res) => {
    const stream = fs.createReadStream('large-file.txt');
    stream.pipe(res);  // Stream file to client
}).listen(3000);
```

### Module 8.2: Worker Threads

**Description:** Worker threads allow running JavaScript in parallel, utilizing multiple CPU cores for CPU-intensive tasks.

```javascript
// worker.js - CPU-intensive task
const { parentPort, workerData } = require('worker_threads');

// Fibonacci calculation (CPU-intensive)
function fibonacci(n) {
    if (n <= 1) return n;
    return fibonacci(n - 1) + fibonacci(n - 2);
}

const result = fibonacci(workerData.num);

// Send result back to main thread
parentPort.postMessage(result);
```

```javascript
// main.js - Main thread
const { Worker } = require('worker_threads');

function runWorker(num) {
    return new Promise((resolve, reject) => {
        // Create worker
        const worker = new Worker('./worker.js', {
            workerData: { num }
        });
        
        // Listen for messages from worker
        worker.on('message', resolve);
        worker.on('error', reject);
        worker.on('exit', (code) => {
            if (code !== 0) {
                reject(new Error(`Worker stopped with exit code ${code}`));
            }
        });
    });
}

// Run multiple workers in parallel
async function runMultipleWorkers() {
    const promises = [
        runWorker(40),
        runWorker(41),
        runWorker(42)
    ];
    
    const results = await Promise.all(promises);
    console.log('Results:', results);
}

runMultipleWorkers();
```

### Module 8.3: Cluster Module

**Description:** Cluster module allows creating child processes that share the same server port, utilizing all CPU cores.

```javascript
// server-cluster.js
const cluster = require('cluster');
const http = require('http');
const os = require('os');
const numCPUs = os.cpus().length;

if (cluster.isMaster) {
    console.log(`Master process ${process.pid} is running`);
    
    // Fork workers (one per CPU core)
    for (let i = 0; i < numCPUs; i++) {
        cluster.fork();
    }
    
    // Handle worker exit
    cluster.on('exit', (worker, code, signal) => {
        console.log(`Worker ${worker.process.pid} died`);
        console.log('Starting a new worker');
        cluster.fork();  // Restart worker
    });
    
} else {
    // Workers can share any TCP connection
    http.createServer((req, res) => {
        res.writeHead(200);
        res.end(`Response from worker ${process.pid}`);
    }).listen(3000);
    
    console.log(`Worker ${process.pid} started`);
}
```

**With Express:**

```javascript
const cluster = require('cluster');
const express = require('express');
const os = require('os');

if (cluster.isMaster) {
    const numCPUs = os.cpus().length;
    console.log(`Master ${process.pid} is running`);
    console.log(`Forking ${numCPUs} workers...`);
    
    for (let i = 0; i < numCPUs; i++) {
        cluster.fork();
    }
    
    cluster.on('exit', (worker, code, signal) => {
        console.log(`Worker ${worker.process.pid} died`);
        cluster.fork();
    });
    
} else {
    const app = express();
    
    app.get('/', (req, res) => {
        res.json({
            message: 'Hello from cluster',
            worker: process.pid
        });
    });
    
    app.listen(3000, () => {
        console.log(`Worker ${process.pid} listening on port 3000`);
    });
}
```

### Module 8.4: WebSockets

**Installation:**
```bash
npm install ws
```

**WebSocket Server:**

```javascript
// websocket-server.js
const WebSocket = require('ws');
const http = require('http');

const server = http.createServer();
const wss = new WebSocket.Server({ server });

// Store connected clients
const clients = new Set();

wss.on('connection', (ws) => {
    console.log('New client connected');
    clients.add(ws);
    
    // Send welcome message
    ws.send(JSON.stringify({
        type: 'connection',
        message: 'Welcome to WebSocket server'
    }));
    
    // Handle incoming messages
    ws.on('message', (message) => {
        console.log('Received:', message.toString());
        
        const data = JSON.parse(message);
        
        // Broadcast to all clients
        broadcast({
            type: 'message',
            data: data,
            timestamp: new Date()
        });
    });
    
    // Handle client disconnect
    ws.on('close', () => {
        console.log('Client disconnected');
        clients.delete(ws);
    });
    
    // Handle errors
    ws.on('error', (error) => {
        console.error('WebSocket error:', error);
    });
});

// Broadcast message to all clients
function broadcast(message) {
    const data = JSON.stringify(message);
    
    clients.forEach((client) => {
        if (client.readyState === WebSocket.OPEN) {
            client.send(data);
        }
    });
}

server.listen(8080, () => {
    console.log('WebSocket server listening on port 8080');
});
```

**WebSocket Client:**

```javascript
// client.js
const WebSocket = require('ws');

const ws = new WebSocket('ws://localhost:8080');

ws.on('open', () => {
    console.log('Connected to server');
    
    // Send message
    ws.send(JSON.stringify({
        user: 'John',
        message: 'Hello, Server!'
    }));
});

ws.on('message', (data) => {
    const message = JSON.parse(data);
    console.log('Received:', message);
});

ws.on('close', () => {
    console.log('Disconnected from server');
});

ws.on('error', (error) => {
    console.error('Error:', error);
});
```

**Real-time Chat Application:**

```javascript
// chat-server.js
const express = require('express');
const http = require('http');
const WebSocket = require('ws');

const app = express();
const server = http.createServer(app);
const wss = new WebSocket.Server({ server });

const users = new Map();  // username -> WebSocket

wss.on('connection', (ws) => {
    let username;
    
    ws.on('message', (message) => {
        const data = JSON.parse(message);
        
        switch (data.type) {
            case 'join':
                username = data.username;
                users.set(username, ws);
                
                // Notify all users
                broadcast({
                    type: 'user-joined',
                    username: username,
                    userCount: users.size
                });
                break;
                
            case 'message':
                // Broadcast message to all users
                broadcast({
                    type: 'message',
                    username: username,
                    text: data.text,
                    timestamp: new Date()
                });
                break;
                
            case 'typing':
                // Notify others that user is typing
                broadcast({
                    type: 'typing',
                    username: username
                }, username);
                break;
        }
    });
    
    ws.on('close', () => {
        if (username) {
            users.delete(username);
            
            broadcast({
                type: 'user-left',
                username: username,
                userCount: users.size
            });
        }
    });
});

function broadcast(message, excludeUser = null) {
    const data = JSON.stringify(message);
    
    users.forEach((client, user) => {
        if (user !== excludeUser && client.readyState === WebSocket.OPEN) {
            client.send(data);
        }
    });
}

server.listen(3000, () => {
    console.log('Chat server running on port 3000');
});
```

### Module 8.5: GraphQL

**Installation:**
```bash
npm install apollo-server-express graphql
```

**GraphQL Server:**

```javascript
const express = require('express');
const { ApolloServer, gql } = require('apollo-server-express');

// Type definitions (schema)
const typeDefs = gql`
    type User {
        id: ID!
        name: String!
        email: String!
        age: Int
        posts: [Post!]!
    }
    
    type Post {
        id: ID!
        title: String!
        content: String!
        author: User!
    }
    
    type Query {
        users: [User!]!
        user(id: ID!): User
        posts: [Post!]!
        post(id: ID!): Post
    }
    
    type Mutation {
        createUser(name: String!, email: String!, age: Int): User!
        createPost(title: String!, content: String!, authorId: ID!): Post!
    }
`;

// Sample data
const users = [
    { id: '1', name: 'John Doe', email: 'john@example.com', age: 30 },
    { id: '2', name: 'Jane Smith', email: 'jane@example.com', age: 25 }
];

const posts = [
    { id: '1', title: 'First Post', content: 'Hello World', authorId: '1' },
    { id: '2', title: 'Second Post', content: 'GraphQL is awesome', authorId: '1' }
];

// Resolvers - How to fetch data
const resolvers = {
    Query: {
        users: () => users,
        user: (parent, args) => users.find(u => u.id === args.id),
        posts: () => posts,
        post: (parent, args) => posts.find(p => p.id === args.id)
    },
    
    Mutation: {
        createUser: (parent, args) => {
            const newUser = {
                id: String(users.length + 1),
                name: args.name,
                email: args.email,
                age: args.age
            };
            users.push(newUser);
            return newUser;
        },
        
        createPost: (parent, args) => {
            const newPost = {
                id: String(posts.length + 1),
                title: args.title,
                content: args.content,
                authorId: args.authorId
            };
            posts.push(newPost);
            return newPost;
        }
    },
    
    User: {
        posts: (parent) => posts.filter(p => p.authorId === parent.id)
    },
    
    Post: {
        author: (parent) => users.find(u => u.id === parent.authorId)
    }
};

// Create Apollo Server
const server = new ApolloServer({
    typeDefs,
    resolvers
});

const app = express();

async function startServer() {
    await server.start();
    server.applyMiddleware({ app });
    
    app.listen(4000, () => {
        console.log(`Server ready at http://localhost:4000${server.graphqlPath}`);
    });
}

startServer();
```

**GraphQL Queries:**

```graphql
# Get all users
query {
    users {
        id
        name
        email
        posts {
            title
        }
    }
}

# Get specific user
query {
    user(id: "1") {
        name
        email
        posts {
            title
            content
        }
    }
}

# Create user mutation
mutation {
    createUser(name: "Bob", email: "bob@example.com", age: 35) {
        id
        name
        email
    }
}

# Create post mutation
mutation {
    createPost(title: "New Post", content: "Content here", authorId: "1") {
        id
        title
        author {
            name
        }
    }
}
```

---

## Phase 9: Production & Deployment

### Module 9.1: Environment Configuration

**.env File:**
```
# .env
NODE_ENV=production
PORT=3000
DATABASE_URL=mongodb://localhost:27017/myapp
JWT_SECRET=your-secret-key-here
JWT_EXPIRE=7d
API_KEY=your-api-key
EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=587
EMAIL_USER=your-email@gmail.com
EMAIL_PASSWORD=your-password
```

**Configuration Management:**

```javascript
// config/config.js
require('dotenv').config();

module.exports = {
    env: process.env.NODE_ENV || 'development',
    port: process.env.PORT || 3000,
    database: {
        url: process.env.DATABASE_URL,
        options: {
            useNewUrlParser: true,
            useUnifiedTopology: true
        }
    },
    jwt: {
        secret: process.env.JWT_SECRET,
        expire: process.env.JWT_EXPIRE
    },
    email: {
        host: process.env.EMAIL_HOST,
        port: process.env.EMAIL_PORT,
        auth: {
            user: process.env.EMAIL_USER,
            pass: process.env.EMAIL_PASSWORD
        }
    }
};
```

**Different Environments:**

```javascript
// config/env/development.js
module.exports = {
    database: {
        url: 'mongodb://localhost:27017/myapp-dev',
        debug: true
    },
    logging: 'dev',
    cors: {
        origin: '*'
    }
};

// config/env/production.js
module.exports = {
    database: {
        url: process.env.DATABASE_URL,
        debug: false
    },
    logging: 'combined',
    cors: {
        origin: 'https://yourdomain.com'
    }
};

// Load environment-specific config
const env = process.env.NODE_ENV || 'development';
const config = require(`./env/${env}`);
```

### Module 9.2: Logging and Monitoring

**Winston Logger Setup:**

```javascript
// utils/logger.js
const winston = require('winston');
const path = require('path');

const logger = winston.createLogger({
    level: process.env.LOG_LEVEL || 'info',
    format: winston.format.combine(
        winston.format.timestamp({
            format: 'YYYY-MM-DD HH:mm:ss'
        }),
        winston.format.errors({ stack: true }),
        winston.format.splat(),
        winston.format.json()
    ),
    defaultMeta: { service: 'my-app' },
    transports: [
        // Error logs
        new winston.transports.File({
            filename: path.join('logs', 'error.log'),
            level: 'error',
            maxsize: 5242880, // 5MB
            maxFiles: 5
        }),
        // Combined logs
        new winston.transports.File({
            filename: path.join('logs', 'combined.log'),
            maxsize: 5242880,
            maxFiles: 5
        })
    ],
    exceptionHandlers: [
        new winston.transports.File({
            filename: path.join('logs', 'exceptions.log')
        })
    ]
});

// Console logging in development
if (process.env.NODE_ENV !== 'production') {
    logger.add(new winston.transports.Console({
        format: winston.format.combine(
            winston.format.colorize(),
            winston.format.simple()
        )
    }));
}

module.exports = logger;
```

**Morgan HTTP Logging:**

```javascript
const morgan = require('morgan');
const fs = require('fs');
const path = require('path');

// Create write stream for access logs
const accessLogStream = fs.createWriteStream(
    path.join(__dirname, 'logs', 'access.log'),
    { flags: 'a' }
);

// Setup morgan
app.use(morgan('combined', { stream: accessLogStream }));

// In development, also log to console
if (process.env.NODE_ENV === 'development') {
    app.use(morgan('dev'));
}
```

### Module 9.3: Docker Containerization

**Dockerfile:**

```dockerfile
# Dockerfile
FROM node:18-alpine

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy application files
COPY . .

# Expose port
EXPOSE 3000

# Create non-root user
RUN addgroup -g 1001 -S nodejs && adduser -S nodejs -u 1001
USER nodejs

# Start application
CMD ["node", "server.js"]
```

**docker-compose.yml:**

```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=mongodb://mongo:27017/myapp
    depends_on:
      - mongo
    volumes:
      - ./logs:/app/logs
    restart: unless-stopped
  
  mongo:
    image: mongo:6
    ports:
      - "27017:27017"
    volumes:
      - mongo-data:/data/db
    restart: unless-stopped
  
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - app
    restart: unless-stopped

volumes:
  mongo-data:
```

**.dockerignore:**

```
node_modules
npm-debug.log
.env
.git
.gitignore
README.md
logs
*.md
.vscode
```

**Build and Run:**

```bash
# Build image
docker build -t my-node-app .

# Run container
docker run -p 3000:3000 my-node-app

# Using docker-compose
docker-compose up -d

# View logs
docker-compose logs -f app

# Stop containers
docker-compose down
```

### Module 9.4: CI/CD with GitHub Actions

**.github/workflows/node.yml:**

```yaml
name: Node.js CI/CD

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        node-version: [16.x, 18.x]
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v3
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Run linter
      run: npm run lint
    
    - name: Run tests
      run: npm test
    
    - name: Generate coverage report
      run: npm run test:coverage
  
  build:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Login to DockerHub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}
    
    - name: Build and push Docker image
      uses: docker/build-push-action@v4
      with:
        context: .
        push: true
        tags: username/my-node-app:latest
  
  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
    - name: Deploy to production
      uses: appleboy/ssh-action@master
      with:
        host: ${{ secrets.HOST }}
        username: ${{ secrets.USERNAME }}
        key: ${{ secrets.SSH_PRIVATE_KEY }}
        script: |
          cd /app
          docker-compose pull
          docker-compose up -d
          docker system prune -f
```

### Module 9.5: Cloud Deployment

**Heroku Deployment:**

```bash
# Install Heroku CLI
# Login
heroku login

# Create app
heroku create my-node-app

# Add MongoDB addon
heroku addons:create mongodb atlas:sandbox

# Set environment variables
heroku config:set NODE_ENV=production
heroku config:set JWT_SECRET=your-secret

# Deploy
git push heroku main

# View logs
heroku logs --tail
```

**Procfile (Heroku):**
```
web: node server.js
```

**AWS EC2 Deployment:**

```bash
# Connect to EC2 instance
ssh -i key.pem ubuntu@ec2-ip-address

# Install Node.js
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# Install PM2
sudo npm install -g pm2

# Clone repository
git clone https://github.com/yourusername/your-repo.git
cd your-repo

# Install dependencies
npm install --production

# Start with PM2
pm2 start server.js --name my-app

# Save PM2 configuration
pm2 save
pm2 startup

# Monitor
pm2 monit
pm2 logs
```

**PM2 Ecosystem File:**

```javascript
// ecosystem.config.js
module.exports = {
    apps: [{
        name: 'my-app',
        script: './server.js',
        instances: 'max',  // Use all CPU cores
        exec_mode: 'cluster',
        env: {
            NODE_ENV: 'development'
        },
        env_production: {
            NODE_ENV: 'production'
        },
        error_file: './logs/err.log',
        out_file: './logs/out.log',
        log_date_format: 'YYYY-MM-DD HH:mm:ss Z',
        max_memory_restart: '1G',
        autorestart: true,
        watch: false
    }]
};
```

**Start with PM2:**
```bash
pm2 start ecosystem.config.js --env production
```

### Module 9.6: Performance Optimization

**Caching with Redis:**

```bash
npm install redis
```

```javascript
const redis = require('redis');
const client = redis.createClient({
    host: 'localhost',
    port: 6379
});

client.on('error', (err) => {
    console.error('Redis error:', err);
});

// Cache middleware
const cacheMiddleware = (duration) => {
    return (req, res, next) => {
        const key = `cache:${req.originalUrl}`;
        
        client.get(key, (err, data) => {
            if (err) throw err;
            
            if (data) {
                // Return cached data
                return res.json(JSON.parse(data));
            } else {
                // Store original send function
                res.sendResponse = res.json;
                
                // Override json function
                res.json = (body) => {
                    // Cache response
                    client.setex(key, duration, JSON.stringify(body));
                    res.sendResponse(body);
                };
                
                next();
            }
        });
    };
};

// Use caching
app.get('/api/users', cacheMiddleware(300), async (req, res) => {
    const users = await User.find();
    res.json({ success: true, data: users });
});
```

**Compression:**

```javascript
const compression = require('compression');

// Enable gzip compression
app.use(compression({
    level: 6,  // Compression level (0-9)
    threshold: 1024,  // Only compress responses larger than 1KB
    filter: (req, res) => {
        if (req.headers['x-no-compression']) {
            return false;
        }
        return compression.filter(req, res);
    }
}));
```

**Database Indexing:**

```javascript
// Create indexes in Mongoose
userSchema.index({ email: 1 });  // Single field index
userSchema.index({ name: 1, email: 1 });  // Compound index
userSchema.index({ location: '2dsphere' });  // Geospatial index

// Text search index
postSchema.index({ title: 'text', content: 'text' });

// Use indexes in queries
const users = await User.find({ email: 'john@example.com' });  // Uses index
const posts = await Post.find({ $text: { $search: 'nodejs' } });  // Text search
```

---

## Conclusion

Congratulations! You've completed the comprehensive Node.js Mastery Tutorial covering all 9 phases:

**✅ Phase 1 - JavaScript Essentials:**
- Variables, data types, operators
- Control flow and loops
- Functions, scope, closures
- Objects and arrays
- Modern ES6+ features
- Classes and OOP

**✅ Phase 2 - Node.js Fundamentals:**
- Node.js basics and globals
- Module systems (CommonJS & ES)
- File system operations
- Path and OS modules

**✅ Phase 3 - Asynchronous Programming:**
- Callbacks and callback hell
- Promises and chaining
- Async/await patterns
- Error handling

**✅ Phase 4 - Express.js Framework:**
- Server setup and routing
- Middleware (built-in & custom)
- Express Router
- Request/Response handling

**✅ Phase 5 - Database Integration:**
- MongoDB with Mongoose
- CRUD operations
- Schema design and validation
- RESTful API development

**✅ Phase 6 - Authentication & Security:**
- JWT authentication
- Password hashing with bcrypt
- Protected routes
- Security best practices (helmet, rate limiting, XSS prevention)

**✅ Phase 7 - Testing & Debugging:**
- Unit testing with Jest
- API testing with Supertest
- Debugging techniques
- Logging with Winston

**✅ Phase 8 - Advanced Topics:**
- Streams for efficient data processing
- Worker threads for CPU-intensive tasks
- Cluster module for multi-core utilization
- WebSockets for real-time communication
- GraphQL API development

**✅ Phase 9 - Production & Deployment:**
- Environment configuration
- Logging and monitoring
- Docker containerization
- CI/CD pipelines
- Cloud deployment (Heroku, AWS)
- Performance optimization

### Next Steps:

**Build Real Projects:**
1. **REST API with Authentication** - User management system
2. **Real-time Chat App** - Using WebSockets
3. **E-commerce Backend** - Products, cart, orders
4. **Blog Platform** - CRUD with comments and likes
5. **Task Management API** - Teams and projects
6. **Social Media API** - Posts, followers, feed

**Advanced Learning:**
- Microservices architecture
- Message queues (RabbitMQ, Kafka)
- Event-driven architecture
- Serverless with AWS Lambda
- TypeScript with Node.js
- NestJS framework

**Best Practices:**
- Follow RESTful conventions
- Write clean, documented code
- Implement comprehensive testing
- Use version control (Git)
- Monitor application performance
- Keep dependencies updated
- Follow security best practices

**Resources:**
- Official Node.js Documentation: https://nodejs.org/docs
- Express.js Guide: https://expressjs.com
- MDN JavaScript: https://developer.mozilla.org
- MongoDB University: https://university.mongodb.com
- Node.js Design Patterns (book)
- freeCodeCamp Node.js courses

**Community:**
- Node.js Discord
- Stack Overflow
- Reddit r/node
- GitHub discussions

You're now equipped with comprehensive Node.js knowledge from beginner to advanced level. Keep practicing, building projects, and exploring new technologies!

Happy coding! 🚀💻

---

**End of Node.js Mastery Tutorial - Complete Edition**

*Version: 2.0*  
*Last Updated: December 2025*  
*License: MIT*  
*Tutorial Size: 9 Complete Phases*  
*Skill Level: Beginner to Advanced*

