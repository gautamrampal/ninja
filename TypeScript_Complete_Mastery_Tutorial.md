# TypeScript Complete Mastery Tutorial
## From Beginner to Advanced

---

## Table of Contents

1. [Introduction to TypeScript](#1-introduction-to-typescript)
2. [Getting Started](#2-getting-started)
3. [Basic Types](#3-basic-types)
4. [Variables and Type Annotations](#4-variables-and-type-annotations)
5. [Functions](#5-functions)
6. [Interfaces](#6-interfaces)
7. [Classes](#7-classes)
8. [Enums](#8-enums)
9. [Type Assertions and Type Guards](#9-type-assertions-and-type-guards)
10. [Union and Intersection Types](#10-union-and-intersection-types)
11. [Literal Types](#11-literal-types)
12. [Type Aliases](#12-type-aliases)
13. [Generics](#13-generics)
14. [Advanced Types](#14-advanced-types)
15. [Modules and Namespaces](#15-modules-and-namespaces)
16. [Decorators](#16-decorators)
17. [Advanced Generics](#17-advanced-generics)
18. [Utility Types](#18-utility-types)
19. [Conditional Types](#19-conditional-types)
20. [Mapped Types](#20-mapped-types)
21. [Template Literal Types](#21-template-literal-types)
22. [Advanced Patterns](#22-advanced-patterns)
23. [Best Practices](#23-best-practices)

---

## 1. Introduction to TypeScript

### What is TypeScript?

TypeScript is a **statically typed superset** of JavaScript that compiles to plain JavaScript. It was developed and maintained by Microsoft.

**Key Benefits:**
- **Type Safety**: Catch errors at compile-time rather than runtime
- **Better IDE Support**: Enhanced autocomplete, refactoring, and navigation
- **Self-Documenting Code**: Types serve as inline documentation
- **Modern JavaScript Features**: Use latest ECMAScript features
- **Scalability**: Better for large codebases and team collaboration

**Compilation Flow:**
```
TypeScript Code (.ts) → TypeScript Compiler (tsc) → JavaScript Code (.js) → Execution
```

---

## 2. Getting Started

### Installation

```bash
# Install TypeScript globally
npm install -g typescript

# Check version
tsc --version

# Initialize a TypeScript project
tsc --init
```

### Basic Configuration (tsconfig.json)

```json
{
  "compilerOptions": {
    "target": "ES2020",              // Target JavaScript version
    "module": "commonjs",            // Module system
    "strict": true,                  // Enable all strict type-checking options
    "esModuleInterop": true,         // Better CommonJS/ES6 module interoperability
    "skipLibCheck": true,            // Skip type checking of declaration files
    "forceConsistentCasingInFileNames": true,
    "outDir": "./dist",              // Output directory for compiled files
    "rootDir": "./src"               // Root directory of source files
  }
}
```

### Compiling TypeScript

```bash
# Compile a single file
tsc filename.ts

# Compile all files in project
tsc

# Watch mode (auto-compile on save)
tsc --watch
```

---

## 3. Basic Types

### Primitive Types

```typescript
// **Boolean**: Represents true/false values
let isDone: boolean = false;
let isActive: boolean = true;

// **Number**: All numbers in TypeScript are floating-point values
let decimal: number = 6;
let hex: number = 0xf00d;        // Hexadecimal
let binary: number = 0b1010;     // Binary
let octal: number = 0o744;       // Octal

// **String**: Textual data
let color: string = "blue";
let fullName: string = `Bob Smith`;

// Template strings (multi-line and embedded expressions)
let age: number = 37;
let sentence: string = `Hello, my name is ${fullName}.
I'll be ${age + 1} years old next month.`;

// **Null and Undefined**: Represent absence of value
let u: undefined = undefined;
let n: null = null;

// **Symbol**: Unique and immutable primitive values
let sym1: symbol = Symbol("key");
let sym2: symbol = Symbol("key");
console.log(sym1 === sym2); // false - each symbol is unique

// **BigInt**: For arbitrarily large integers (ES2020+)
let big: bigint = 100n;
```

### Array Types

```typescript
// **Array Declaration Method 1**: Using type[]
let list1: number[] = [1, 2, 3];
let names: string[] = ["Alice", "Bob", "Charlie"];

// **Array Declaration Method 2**: Using generic Array<type>
let list2: Array<number> = [1, 2, 3];
let colors: Array<string> = ["red", "green", "blue"];

// **Mixed type arrays**: Using union types
let mixed: (string | number)[] = ["hello", 42, "world"];

// **Multi-dimensional arrays**
let matrix: number[][] = [
  [1, 2, 3],
  [4, 5, 6],
  [7, 8, 9]
];
```

### Tuple Types

```typescript
// **Tuple**: Fixed-length array with known types for each position
let tuple: [string, number];
tuple = ["hello", 10];  // OK
// tuple = [10, "hello"];  // Error! Order matters

// **Accessing tuple elements**
console.log(tuple[0]);  // "hello"
console.log(tuple[1]);  // 10

// **Tuple with optional elements**
let optionalTuple: [string, number?];
optionalTuple = ["hello"];      // OK
optionalTuple = ["hello", 42];  // OK

// **Rest elements in tuples**
type StringNumberBooleans = [string, number, ...boolean[]];
let t1: StringNumberBooleans = ["hello", 1, true, false, true];

// **Named tuples (TypeScript 4.0+)**
type Range = [start: number, end: number];
let range: Range = [1, 100];
```

### Any Type

```typescript
// **any**: Opt-out of type checking (use sparingly!)
let notSure: any = 4;
notSure = "maybe a string instead";  // OK
notSure = false;                     // OK

// Useful when working with dynamic content or third-party libraries
let dynamicValue: any = getUserInput();
dynamicValue.ifItExists();  // No compile-time checking
dynamicValue.toFixed();     // No error even if method doesn't exist
```

### Unknown Type

```typescript
// **unknown**: Type-safe alternative to any
let userInput: unknown;
userInput = 5;
userInput = "hello";

// Must perform type checking before using
// let value1: string = userInput;  // Error!

// Correct approach with type guards
if (typeof userInput === "string") {
  let value2: string = userInput;  // OK - TypeScript knows it's a string
}

// **Difference between any and unknown**
let anyValue: any = "hello";
let unknownValue: unknown = "hello";

anyValue.toUpperCase();      // OK (no type checking)
// unknownValue.toUpperCase();  // Error! Must check type first

if (typeof unknownValue === "string") {
  unknownValue.toUpperCase(); // OK
}
```

### Void Type

```typescript
// **void**: Absence of any type, commonly used for functions that don't return
function warnUser(): void {
  console.log("This is a warning message");
  // No return statement or returns undefined
}

// Variables of type void can only be assigned undefined or null
let unusable: void = undefined;
```

### Never Type

```typescript
// **never**: Represents values that never occur
// Used for functions that never return (throw errors or infinite loops)

// Function that throws error
function error(message: string): never {
  throw new Error(message);
}

// Function with infinite loop
function infiniteLoop(): never {
  while (true) {
    // Never exits
  }
}

// **never in exhaustive type checking**
type Shape = Circle | Square;

function getArea(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.size ** 2;
    default:
      // If all cases are handled, this should never execute
      const _exhaustiveCheck: never = shape;
      return _exhaustiveCheck;
  }
}
```

### Object Type

```typescript
// **object**: Represents non-primitive types
let obj: object = { name: "John" };
obj = [1, 2, 3];  // Arrays are objects
obj = new Date(); // OK

// **Better approach**: Define specific object structure
let person: { name: string; age: number } = {
  name: "Alice",
  age: 30
};
```

---

## 4. Variables and Type Annotations

### Type Inference

```typescript
// **Type Inference**: TypeScript automatically infers types
let message = "Hello";  // TypeScript infers: string
// message = 123;       // Error! Type 'number' is not assignable to type 'string'

let count = 42;         // Inferred as number
let isValid = true;     // Inferred as boolean

// **Best Practice**: Let TypeScript infer when obvious
let numbers = [1, 2, 3]; // Inferred as number[]
```

### Explicit Type Annotations

```typescript
// **Explicit annotations**: Specify type manually
let username: string;
username = "JohnDoe";

let score: number = 100;
let isLoggedIn: boolean = false;

// **When to use explicit annotations**:
// 1. Variable declared without initialization
let value: string;
// Later assignment
value = "assigned later";

// 2. Function parameters (always required)
function greet(name: string): void {
  console.log(`Hello, ${name}`);
}

// 3. Complex types that aren't obvious
let data: { id: number; value: string }[];
data = [{ id: 1, value: "first" }];
```

### Const vs Let vs Var

```typescript
// **const**: Cannot be reassigned, type is literal or narrowed
const pi = 3.14159;  // Type: 3.14159 (literal type)
// pi = 3.14;        // Error!

const user = { name: "John" };
user.name = "Jane";  // OK - object properties can change
// user = { name: "Jane" };  // Error! Cannot reassign const

// **let**: Block-scoped, can be reassigned
let counter = 0;
counter = 1;  // OK

// **var**: Function-scoped (avoid using in modern TypeScript)
var oldStyle = "avoid using var";
```

---

## 5. Functions

### Function Declarations

```typescript
// **Basic function with type annotations**
function add(x: number, y: number): number {
  return x + y;
}

// **Function without return value**
function logMessage(message: string): void {
  console.log(message);
}

// **Function with type inference on return**
function multiply(a: number, b: number) {
  return a * b;  // TypeScript infers return type: number
}
```

### Function Expressions

```typescript
// **Function expression with explicit type**
const subtract = function(x: number, y: number): number {
  return x - y;
};

// **Arrow function**
const divide = (x: number, y: number): number => {
  return x / y;
};

// **Concise arrow function**
const square = (x: number): number => x * x;
```

### Function Types

```typescript
// **Defining function type**
let mathOperation: (a: number, b: number) => number;

mathOperation = (x, y) => x + y;     // OK
mathOperation = (x, y) => x * y;     // OK
// mathOperation = (x, y) => x.toString();  // Error! Return type mismatch

// **Function type alias**
type BinaryOperation = (a: number, b: number) => number;

const add: BinaryOperation = (x, y) => x + y;
const subtract: BinaryOperation = (x, y) => x - y;
```

### Optional Parameters

```typescript
// **Optional parameters**: Use ? to make parameters optional
function buildName(firstName: string, lastName?: string): string {
  if (lastName) {
    return `${firstName} ${lastName}`;
  }
  return firstName;
}

console.log(buildName("John"));          // "John"
console.log(buildName("John", "Doe"));   // "John Doe"
```

### Default Parameters

```typescript
// **Default parameters**: Provide default values
function greet(name: string, greeting: string = "Hello"): string {
  return `${greeting}, ${name}!`;
}

console.log(greet("Alice"));              // "Hello, Alice!"
console.log(greet("Bob", "Hi"));          // "Hi, Bob!"
```

### Rest Parameters

```typescript
// **Rest parameters**: Accept variable number of arguments
function sum(...numbers: number[]): number {
  return numbers.reduce((total, n) => total + n, 0);
}

console.log(sum(1, 2, 3));        // 6
console.log(sum(1, 2, 3, 4, 5));  // 15

// **Combining regular and rest parameters**
function buildArray(first: string, ...rest: string[]): string[] {
  return [first, ...rest];
}
```

### Function Overloads

```typescript
// **Function overloading**: Multiple function signatures
// Overload signatures
function process(value: string): string;
function process(value: number): number;
function process(value: boolean): boolean;

// Implementation signature (must be compatible with all overloads)
function process(value: string | number | boolean): string | number | boolean {
  if (typeof value === "string") {
    return value.toUpperCase();
  } else if (typeof value === "number") {
    return value * 2;
  } else {
    return !value;
  }
}

console.log(process("hello"));  // "HELLO"
console.log(process(5));        // 10
console.log(process(true));     // false

// **Complex overloading example**
function makeDate(timestamp: number): Date;
function makeDate(year: number, month: number, day: number): Date;
function makeDate(yearOrTimestamp: number, month?: number, day?: number): Date {
  if (month !== undefined && day !== undefined) {
    return new Date(yearOrTimestamp, month - 1, day);
  } else {
    return new Date(yearOrTimestamp);
  }
}
```

### This Parameter

```typescript
// **this parameter**: Explicitly type 'this' context
interface Card {
  suit: string;
  card: number;
}

interface Deck {
  suits: string[];
  cards: number[];
  createCardPicker(this: Deck): () => Card;
}

let deck: Deck = {
  suits: ["hearts", "spades", "clubs", "diamonds"],
  cards: Array(52),
  createCardPicker: function(this: Deck) {
    return () => {
      let pickedCard = Math.floor(Math.random() * 52);
      let pickedSuit = Math.floor(pickedCard / 13);
      
      return { suit: this.suits[pickedSuit], card: pickedCard % 13 };
    };
  }
};
```

---

## 6. Interfaces

### Basic Interface

```typescript
// **Interface**: Define the shape of an object
interface Person {
  firstName: string;
  lastName: string;
  age: number;
}

// Object must match the interface structure
const person: Person = {
  firstName: "John",
  lastName: "Doe",
  age: 30
};

// **Using interface in function**
function greetPerson(person: Person): string {
  return `Hello, ${person.firstName} ${person.lastName}`;
}
```

### Optional Properties

```typescript
// **Optional properties**: Use ? for optional fields
interface User {
  id: number;
  username: string;
  email?: string;      // Optional
  phone?: string;      // Optional
}

const user1: User = {
  id: 1,
  username: "john_doe"
  // email and phone are optional
};

const user2: User = {
  id: 2,
  username: "jane_doe",
  email: "jane@example.com"
};
```

### Readonly Properties

```typescript
// **readonly**: Properties that cannot be modified after creation
interface Point {
  readonly x: number;
  readonly y: number;
}

let p1: Point = { x: 10, y: 20 };
// p1.x = 5;  // Error! Cannot assign to 'x' because it is a read-only property

// **ReadonlyArray**: Immutable array
let numbers: ReadonlyArray<number> = [1, 2, 3, 4];
// numbers[0] = 5;     // Error!
// numbers.push(5);    // Error!
// numbers.length = 0; // Error!
```

### Function Types in Interfaces

```typescript
// **Interface for function types**
interface SearchFunc {
  (source: string, subString: string): boolean;
}

const mySearch: SearchFunc = function(src, sub) {
  return src.includes(sub);
};

console.log(mySearch("hello world", "world"));  // true
```

### Indexable Types

```typescript
// **Index signatures**: For objects with dynamic keys
interface StringArray {
  [index: number]: string;
}

let myArray: StringArray = ["Bob", "Alice"];
let myStr: string = myArray[0];

// **String index signature**
interface Dictionary {
  [key: string]: number;
}

let ages: Dictionary = {
  "Alice": 30,
  "Bob": 25,
  "Charlie": 35
};

// **Combining index signature with known properties**
interface Config {
  [key: string]: string | number;
  version: string;  // Known property
  timeout: number;  // Known property
}
```

### Class Types

```typescript
// **Interface for class structure**
interface ClockInterface {
  currentTime: Date;
  setTime(d: Date): void;
}

class Clock implements ClockInterface {
  currentTime: Date = new Date();
  
  setTime(d: Date) {
    this.currentTime = d;
  }
  
  constructor(h: number, m: number) {}
}
```

### Extending Interfaces

```typescript
// **Interface inheritance**: Extend one or more interfaces
interface Shape {
  color: string;
}

interface Square extends Shape {
  sideLength: number;
}

let square: Square = {
  color: "blue",
  sideLength: 10
};

// **Multiple inheritance**
interface Colorful {
  color: string;
}

interface Circular {
  radius: number;
}

interface ColorfulCircle extends Colorful, Circular {}

let circle: ColorfulCircle = {
  color: "red",
  radius: 42
};
```

### Hybrid Types

```typescript
// **Hybrid types**: Objects that act as both functions and objects
interface Counter {
  (start: number): string;  // Callable
  interval: number;         // Property
  reset(): void;            // Method
}

function getCounter(): Counter {
  let counter = function(start: number) {
    return `Count: ${start}`;
  } as Counter;
  
  counter.interval = 123;
  counter.reset = function() {
    console.log("Reset!");
  };
  
  return counter;
}

let c = getCounter();
c(10);           // Calling as function
c.reset();       // Calling method
c.interval = 5;  // Setting property
```

---

## 7. Classes

### Basic Class

```typescript
// **Class**: Blueprint for creating objects
class Greeter {
  // Property declaration
  greeting: string;
  
  // Constructor: Initialize object
  constructor(message: string) {
    this.greeting = message;
  }
  
  // Method
  greet(): string {
    return `Hello, ${this.greeting}`;
  }
}

// Create instance
let greeter = new Greeter("world");
console.log(greeter.greet());  // "Hello, world"
```

### Inheritance

```typescript
// **Inheritance**: Extend base class functionality
class Animal {
  name: string;
  
  constructor(name: string) {
    this.name = name;
  }
  
  move(distance: number = 0): void {
    console.log(`${this.name} moved ${distance}m.`);
  }
}

class Dog extends Animal {
  bark(): void {
    console.log("Woof! Woof!");
  }
}

const dog = new Dog("Buddy");
dog.bark();      // "Woof! Woof!"
dog.move(10);    // "Buddy moved 10m."

// **Method override with super**
class Snake extends Animal {
  constructor(name: string) {
    super(name);  // Call parent constructor
  }
  
  move(distance: number = 5): void {
    console.log("Slithering...");
    super.move(distance);  // Call parent method
  }
}

const snake = new Snake("Python");
snake.move();  // "Slithering..." then "Python moved 5m."
```

### Access Modifiers

```typescript
// **public**: Accessible everywhere (default)
// **private**: Only accessible within the class
// **protected**: Accessible within class and subclasses

class Person {
  public name: string;           // Public (can omit 'public')
  private age: number;           // Private
  protected address: string;     // Protected
  
  constructor(name: string, age: number, address: string) {
    this.name = name;
    this.age = age;
    this.address = address;
  }
  
  public getAge(): number {
    return this.age;  // Can access private within class
  }
}

class Employee extends Person {
  private department: string;
  
  constructor(name: string, age: number, address: string, dept: string) {
    super(name, age, address);
    this.department = dept;
  }
  
  public getDetails(): string {
    // Can access protected 'address' from parent
    return `${this.name} lives at ${this.address}`;
    // Cannot access private 'age' directly
  }
}

const person = new Person("John", 30, "123 Main St");
console.log(person.name);      // OK - public
// console.log(person.age);    // Error - private
// console.log(person.address);// Error - protected
```

### Readonly Modifier

```typescript
// **readonly**: Properties can only be set in constructor or declaration
class Octopus {
  readonly name: string;
  readonly numberOfLegs: number = 8;
  
  constructor(theName: string) {
    this.name = theName;
  }
}

let dad = new Octopus("Man with the 8 strong legs");
console.log(dad.name);
// dad.name = "Man with the 3-piece suit";  // Error! readonly property
```

### Parameter Properties

```typescript
// **Parameter properties**: Shorthand for declaring and initializing properties
class Point {
  // Combines declaration and initialization
  constructor(
    public x: number,
    public y: number,
    private z: number = 0
  ) {}
}

const point = new Point(10, 20);
console.log(point.x);  // 10
console.log(point.y);  // 20
// console.log(point.z);  // Error - private
```

### Getters and Setters

```typescript
// **Accessors**: Control how properties are accessed and modified
class Employee {
  private _fullName: string = "";
  
  // Getter: Read property
  get fullName(): string {
    return this._fullName;
  }
  
  // Setter: Write property with validation
  set fullName(newName: string) {
    if (newName && newName.length > 0) {
      this._fullName = newName;
    } else {
      throw new Error("Name cannot be empty");
    }
  }
}

const employee = new Employee();
employee.fullName = "Bob Smith";  // Calls setter
console.log(employee.fullName);   // Calls getter: "Bob Smith"
```

### Static Members

```typescript
// **static**: Properties/methods belong to class, not instances
class Grid {
  static origin = { x: 0, y: 0 };  // Static property
  
  // Static method
  static calculateDistance(point: { x: number; y: number }): number {
    let xDist = point.x - Grid.origin.x;
    let yDist = point.y - Grid.origin.y;
    return Math.sqrt(xDist * xDist + yDist * yDist);
  }
}

// Access via class name, not instance
console.log(Grid.origin);
console.log(Grid.calculateDistance({ x: 10, y: 10 }));
```

### Abstract Classes

```typescript
// **abstract**: Cannot be instantiated, only extended
abstract class Department {
  constructor(public name: string) {}
  
  // Concrete method
  printName(): void {
    console.log("Department name: " + this.name);
  }
  
  // Abstract method: Must be implemented by subclasses
  abstract printMeeting(): void;
}

class AccountingDepartment extends Department {
  constructor() {
    super("Accounting and Auditing");
  }
  
  // Must implement abstract method
  printMeeting(): void {
    console.log("The Accounting Department meets each Monday at 10am.");
  }
  
  generateReports(): void {
    console.log("Generating accounting reports...");
  }
}

// const dept = new Department("General");  // Error! Cannot create instance
const accounting = new AccountingDepartment();
accounting.printName();
accounting.printMeeting();
accounting.generateReports();
```

---

## 8. Enums

### Numeric Enums

```typescript
// **Numeric Enum**: Auto-incrementing values starting from 0
enum Direction {
  Up,      // 0
  Down,    // 1
  Left,    // 2
  Right    // 3
}

let dir: Direction = Direction.Up;
console.log(dir);  // 0

// **Custom starting value**
enum Status {
  Pending = 1,    // 1
  Active,         // 2
  Completed       // 3
}

// **Custom values**
enum HttpStatus {
  OK = 200,
  NotFound = 404,
  InternalError = 500
}
```

### String Enums

```typescript
// **String Enum**: Each member must be initialized with string literal
enum Color {
  Red = "RED",
  Green = "GREEN",
  Blue = "BLUE"
}

console.log(Color.Red);  // "RED"

// **Useful for debugging and meaningful runtime values**
enum LogLevel {
  Error = "ERROR",
  Warning = "WARNING",
  Info = "INFO",
  Debug = "DEBUG"
}

function log(level: LogLevel, message: string): void {
  console.log(`[${level}] ${message}`);
}

log(LogLevel.Info, "Application started");  // [INFO] Application started
```

### Heterogeneous Enums

```typescript
// **Mixed string and numeric values** (not recommended)
enum Mixed {
  No = 0,
  Yes = "YES"
}
```

### Computed and Constant Members

```typescript
// **Constant members**: Evaluated at compile time
enum FileAccess {
  None,
  Read = 1 << 1,      // Bit shift: 2
  Write = 1 << 2,     // 4
  ReadWrite = Read | Write,  // Bitwise OR: 6
  G = "123".length    // 3
}
```

### Const Enums

```typescript
// **const enum**: Completely removed during compilation, inlined at use sites
const enum Directions {
  Up,
  Down,
  Left,
  Right
}

let directions = [
  Directions.Up,
  Directions.Down,
  Directions.Left,
  Directions.Right
];

// Compiled output: let directions = [0, 1, 2, 3];
```

### Reverse Mapping

```typescript
// **Reverse mapping**: Numeric enums support reverse lookup
enum Role {
  Admin,
  User,
  Guest
}

console.log(Role.Admin);   // 0
console.log(Role[0]);      // "Admin"

// **Flow**: Number → Name and Name → Number
let roleId = Role.User;      // 1
let roleName = Role[roleId]; // "User"
```

---

## 9. Type Assertions and Type Guards

### Type Assertions

```typescript
// **Type Assertion**: Tell TypeScript "trust me, I know what I'm doing"

// **Method 1: angle-bracket syntax**
let someValue: unknown = "this is a string";
let strLength: number = (<string>someValue).length;

// **Method 2: as-syntax (preferred, required in JSX)**
let someValue2: unknown = "this is a string";
let strLength2: number = (someValue2 as string).length;

// **Practical example**
interface User {
  name: string;
  age: number;
}

let data: unknown = { name: "John", age: 30 };
let user = data as User;
console.log(user.name);
```

### Type Guards

```typescript
// **typeof type guard**: Check primitive types
function padLeft(value: string, padding: string | number): string {
  if (typeof padding === "number") {
    // TypeScript knows padding is number here
    return " ".repeat(padding) + value;
  }
  // TypeScript knows padding is string here
  return padding + value;
}

// **instanceof type guard**: Check class instances
class Bird {
  fly() {
    console.log("Flying...");
  }
}

class Fish {
  swim() {
    console.log("Swimming...");
  }
}

function move(animal: Bird | Fish) {
  if (animal instanceof Bird) {
    animal.fly();  // TypeScript knows it's Bird
  } else {
    animal.swim(); // TypeScript knows it's Fish
  }
}

// **in operator type guard**: Check if property exists
type Car = { drive(): void };
type Boat = { sail(): void };

function operate(vehicle: Car | Boat) {
  if ("drive" in vehicle) {
    vehicle.drive();  // TypeScript knows it's Car
  } else {
    vehicle.sail();   // TypeScript knows it's Boat
  }
}
```

### Custom Type Guards

```typescript
// **User-defined type guard**: Create custom type checking function
interface Cat {
  meow(): void;
}

interface Dog {
  bark(): void;
}

// Type predicate: pet is Cat
function isCat(pet: Cat | Dog): pet is Cat {
  return (pet as Cat).meow !== undefined;
}

function makeSound(pet: Cat | Dog) {
  if (isCat(pet)) {
    pet.meow();  // TypeScript knows it's Cat
  } else {
    pet.bark();  // TypeScript knows it's Dog
  }
}

// **Complex example with multiple checks**
interface Admin {
  role: "admin";
  privileges: string[];
}

interface RegularUser {
  role: "user";
  email: string;
}

type User = Admin | RegularUser;

function isAdmin(user: User): user is Admin {
  return user.role === "admin";
}

function processUser(user: User) {
  if (isAdmin(user)) {
    console.log(user.privileges);  // OK - TypeScript knows it's Admin
  } else {
    console.log(user.email);       // OK - TypeScript knows it's RegularUser
  }
}
```

### Discriminated Unions

```typescript
// **Discriminated unions**: Use common property (discriminant) to narrow types
interface Square {
  kind: "square";     // Discriminant
  size: number;
}

interface Rectangle {
  kind: "rectangle";  // Discriminant
  width: number;
  height: number;
}

interface Circle {
  kind: "circle";     // Discriminant
  radius: number;
}

type Shape = Square | Rectangle | Circle;

function getArea(shape: Shape): number {
  // TypeScript narrows type based on 'kind'
  switch (shape.kind) {
    case "square":
      return shape.size * shape.size;
    case "rectangle":
      return shape.width * shape.height;
    case "circle":
      return Math.PI * shape.radius ** 2;
  }
}
```

### Non-null Assertion Operator

```typescript
// **Non-null assertion (!)**: Tell TypeScript a value isn't null/undefined
function processValue(value: string | null) {
  // Without assertion - error
  // console.log(value.toUpperCase());
  
  // With assertion - "I guarantee this isn't null"
  console.log(value!.toUpperCase());
}

// **Use cautiously**: Only when you're certain the value exists
let element = document.getElementById("myElement")!;
element.innerHTML = "Hello";  // No null check
```

---

## 10. Union and Intersection Types

### Union Types

```typescript
// **Union type (|)**: Value can be one of several types
let value: string | number;
value = "hello";  // OK
value = 42;       // OK
// value = true;  // Error!

// **Function with union parameter**
function formatCommandline(command: string | string[]): string {
  if (typeof command === "string") {
    return command.trim();
  } else {
    return command.join(" ");
  }
}

// **Union with literal types**
type Status = "success" | "error" | "pending";
let currentStatus: Status = "success";
// currentStatus = "failed";  // Error! Not in union

// **Array of union types**
let mixed: (string | number)[] = ["hello", 42, "world", 99];
```

### Intersection Types

```typescript
// **Intersection type (&)**: Combine multiple types
interface Colorful {
  color: string;
}

interface Circle {
  radius: number;
}

// ColorfulCircle has BOTH color and radius
type ColorfulCircle = Colorful & Circle;

const cc: ColorfulCircle = {
  color: "red",
  radius: 42
};

// **Practical example: Mixing in functionality**
interface Timestamped {
  timestamp: Date;
}

interface Tagged {
  tags: string[];
}

type TimestampedAndTagged = Timestamped & Tagged;

const item: TimestampedAndTagged = {
  timestamp: new Date(),
  tags: ["important", "urgent"]
};

// **Intersection with class**
class Person {
  constructor(public name: string) {}
}

interface Loggable {
  log(): void;
}

type LoggablePerson = Person & Loggable;

// Must satisfy both Person and Loggable
const lp: LoggablePerson = Object.assign(
  new Person("John"),
  {
    log() {
      console.log(`Person: ${this.name}`);
    }
  }
);
```

### Union vs Intersection

```typescript
// **Union**: OR relationship (can be A OR B)
type StringOrNumber = string | number;
let val1: StringOrNumber = "hello";
let val2: StringOrNumber = 42;

// **Intersection**: AND relationship (must be A AND B)
interface A {
  propA: string;
}

interface B {
  propB: number;
}

type AB = A & B;
const obj: AB = {
  propA: "test",  // Must have propA
  propB: 123      // Must have propB
};
```

---

## 11. Literal Types

### String Literal Types

```typescript
// **String literal**: Exact string values
type Direction = "north" | "south" | "east" | "west";

function move(direction: Direction) {
  console.log(`Moving ${direction}`);
}

move("north");   // OK
// move("up");   // Error! "up" is not assignable to Direction

// **Combining with other types**
type Result = "success" | "error";
type HttpMethod = "GET" | "POST" | "PUT" | "DELETE";

function request(url: string, method: HttpMethod): Result {
  // Implementation
  return "success";
}
```

### Numeric Literal Types

```typescript
// **Numeric literal**: Specific number values
type DiceRoll = 1 | 2 | 3 | 4 | 5 | 6;

function rollDice(): DiceRoll {
  return (Math.floor(Math.random() * 6) + 1) as DiceRoll;
}

// **Configuration with specific values**
interface Config {
  port: 8080 | 3000 | 5000;
  environment: "development" | "staging" | "production";
}
```

### Boolean Literal Types

```typescript
// **Boolean literal**: Specific boolean value
type AlwaysTrue = true;
type AlwaysFalse = false;

// **Useful in complex type logic**
interface ValidationSuccess {
  isValid: true;
  data: string;
}

interface ValidationError {
  isValid: false;
  error: string;
}

type ValidationResult = ValidationSuccess | ValidationError;

function processValidation(result: ValidationResult) {
  if (result.isValid) {
    console.log(result.data);   // TypeScript knows isValid is true
  } else {
    console.log(result.error);  // TypeScript knows isValid is false
  }
}
```

### Template Literal Types

```typescript
// **Template literal types** (TypeScript 4.1+)
type World = "world";
type Greeting = `hello ${World}`;  // "hello world"

// **Combining with unions**
type EmailLocaleIDs = "welcome_email" | "email_heading";
type FooterLocaleIDs = "footer_title" | "footer_sendoff";
type AllLocaleIDs = `${EmailLocaleIDs | FooterLocaleIDs}_id`;
// "welcome_email_id" | "email_heading_id" | "footer_title_id" | "footer_sendoff_id"

// **Event name patterns**
type PropEventSource<T> = {
  on(eventName: `${string & keyof T}Changed`, callback: () => void): void;
};

declare function makeWatchedObject<T>(obj: T): T & PropEventSource<T>;

const person = makeWatchedObject({
  firstName: "Homer",
  age: 42
});

person.on("firstNameChanged", () => {});  // OK
// person.on("firstName", () => {});      // Error! Must end with "Changed"
```

---

## 12. Type Aliases

### Basic Type Aliases

```typescript
// **Type alias**: Create a name for any type
type StringOrNumber = string | number;
type Point = { x: number; y: number };
type ID = string | number;

// Use the alias
let id: ID = "abc123";
id = 999;

let coordinate: Point = { x: 10, y: 20 };
```

### Function Type Aliases

```typescript
// **Function type alias**
type MathOperation = (a: number, b: number) => number;

const add: MathOperation = (x, y) => x + y;
const multiply: MathOperation = (x, y) => x * y;

// **Complex function types**
type Predicate<T> = (item: T) => boolean;
type Mapper<T, U> = (item: T) => U;

const isEven: Predicate<number> = (n) => n % 2 === 0;
const toString: Mapper<number, string> = (n) => n.toString();
```

### Object Type Aliases

```typescript
// **Object structure alias**
type User = {
  id: number;
  name: string;
  email?: string;
  readonly createdAt: Date;
};

const user: User = {
  id: 1,
  name: "John",
  createdAt: new Date()
};

// **Nested object types**
type Address = {
  street: string;
  city: string;
  country: string;
};

type Company = {
  name: string;
  address: Address;
  employees: User[];
};
```

### Extending Type Aliases

```typescript
// **Extending types with intersection**
type Animal = {
  name: string;
  age: number;
};

type Bird = Animal & {
  wingspan: number;
  canFly: boolean;
};

const eagle: Bird = {
  name: "Eagle",
  age: 5,
  wingspan: 2.5,
  canFly: true
};
```

### Type Aliases vs Interfaces

```typescript
// **Interface**: Can be extended and implemented
interface PersonInterface {
  name: string;
}

interface PersonInterface {
  age: number;  // Declaration merging - adds to interface
}

// **Type**: Cannot use declaration merging
type PersonType = {
  name: string;
};

// type PersonType = {  // Error! Duplicate identifier
//   age: number;
// };

// **When to use which**:
// - Interface: For object shapes, especially in OOP and when you need declaration merging
// - Type: For unions, tuples, primitives, and complex type manipulations
```

---

## 13. Generics

### Generic Functions

```typescript
// **Generic function**: Works with multiple types while maintaining type safety
function identity<T>(arg: T): T {
  return arg;
}

// Type argument explicit
let output1 = identity<string>("hello");  // string
let output2 = identity<number>(42);       // number

// Type inference (TypeScript infers T from argument)
let output3 = identity("world");  // TypeScript infers T as string
let output4 = identity(100);      // TypeScript infers T as number

// **Array example**
function firstElement<T>(arr: T[]): T | undefined {
  return arr[0];
}

let first1 = firstElement([1, 2, 3]);      // number | undefined
let first2 = firstElement(["a", "b", "c"]);// string | undefined
```

### Generic Constraints

```typescript
// **Constraining generics**: Limit what types can be used
interface Lengthwise {
  length: number;
}

// T must have a length property
function logLength<T extends Lengthwise>(arg: T): T {
  console.log(arg.length);
  return arg;
}

logLength("hello");        // OK - string has length
logLength([1, 2, 3]);      // OK - array has length
logLength({ length: 10, value: 3 });  // OK - object has length
// logLength(42);          // Error! number doesn't have length

// **Multiple type parameters with constraints**
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

let person = { name: "John", age: 30 };
let name = getProperty(person, "name");  // "John" (string)
let age = getProperty(person, "age");    // 30 (number)
// let invalid = getProperty(person, "email");  // Error! "email" not in person
```

### Generic Interfaces

```typescript
// **Generic interface**
interface Box<T> {
  value: T;
}

let stringBox: Box<string> = { value: "hello" };
let numberBox: Box<number> = { value: 42 };

// **Generic interface for functions**
interface GenericIdentityFn<T> {
  (arg: T): T;
}

let myIdentity: GenericIdentityFn<number> = (x) => x;

// **Complex generic interface**
interface Repository<T> {
  items: T[];
  add(item: T): void;
  remove(id: number): void;
  findById(id: number): T | undefined;
}

class UserRepository implements Repository<User> {
  items: User[] = [];
  
  add(user: User): void {
    this.items.push(user);
  }
  
  remove(id: number): void {
    this.items = this.items.filter(user => user.id !== id);
  }
  
  findById(id: number): User | undefined {
    return this.items.find(user => user.id === id);
  }
}
```

### Generic Classes

```typescript
// **Generic class**
class GenericNumber<T> {
  zeroValue: T;
  add: (x: T, y: T) => T;
  
  constructor(zero: T, addFn: (x: T, y: T) => T) {
    this.zeroValue = zero;
    this.add = addFn;
  }
}

// Number version
let myGenericNumber = new GenericNumber<number>(0, (x, y) => x + y);
console.log(myGenericNumber.add(5, 10));  // 15

// String version
let stringNumeric = new GenericNumber<string>("", (x, y) => x + y);
console.log(stringNumeric.add("Hello ", "World"));  // "Hello World"

// **Stack implementation with generics**
class Stack<T> {
  private items: T[] = [];
  
  push(item: T): void {
    this.items.push(item);
  }
  
  pop(): T | undefined {
    return this.items.pop();
  }
  
  peek(): T | undefined {
    return this.items[this.items.length - 1];
  }
  
  isEmpty(): boolean {
    return this.items.length === 0;
  }
}

const numberStack = new Stack<number>();
numberStack.push(1);
numberStack.push(2);
console.log(numberStack.pop());  // 2
```

### Default Type Parameters

```typescript
// **Default generic type**
interface Container<T = string> {
  value: T;
}

let container1: Container = { value: "default is string" };
let container2: Container<number> = { value: 42 };

// **Function with default type**
function createPair<T = string, U = number>(first: T, second: U): [T, U] {
  return [first, second];
}

let pair1 = createPair("hello", 42);       // [string, number]
let pair2 = createPair<boolean>(true, 10); // [boolean, number]
```

---

## 14. Advanced Types

### Mapped Types

```typescript
// **Mapped types**: Transform properties of existing types
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};

type Partial<T> = {
  [P in keyof T]?: T[P];
};

interface Todo {
  title: string;
  description: string;
  completed: boolean;
}

type ReadonlyTodo = Readonly<Todo>;
// All properties are readonly

type PartialTodo = Partial<Todo>;
// All properties are optional

// **Adding modifiers**
type Mutable<T> = {
  -readonly [P in keyof T]: T[P];  // Remove readonly
};

type Required<T> = {
  [P in keyof T]-?: T[P];  // Remove optional
};
```

### Conditional Types

```typescript
// **Conditional type**: T extends U ? X : Y
type IsString<T> = T extends string ? true : false;

type A = IsString<string>;  // true
type B = IsString<number>;  // false

// **Practical example: Extract function return type**
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

function getUser() {
  return { name: "John", age: 30 };
}

type User = ReturnType<typeof getUser>;  // { name: string; age: number }

// **Distributive conditional types**
type ToArray<T> = T extends any ? T[] : never;
type StrOrNumArray = ToArray<string | number>;  // string[] | number[]
```

### Index Access Types

```typescript
// **Indexing into types**
type Person = {
  name: string;
  age: number;
  address: {
    street: string;
    city: string;
  };
};

type NameType = Person["name"];          // string
type AddressType = Person["address"];    // { street: string; city: string }
type CityType = Person["address"]["city"]; // string

// **Union of property types**
type PersonValues = Person[keyof Person];  // string | number | { street: string; city: string }
```

### Type Inference in Conditional Types

```typescript
// **infer keyword**: Infer types within conditional types
type GetFirstArgument<T> = T extends (first: infer F, ...args: any[]) => any
  ? F
  : never;

function exampleFunc(a: string, b: number) {
  return true;
}

type FirstArg = GetFirstArgument<typeof exampleFunc>;  // string

// **Unwrap Promise type**
type UnwrapPromise<T> = T extends Promise<infer U> ? U : T;

type NumberPromise = Promise<number>;
type Unwrapped = UnwrapPromise<NumberPromise>;  // number
```

### Recursive Types

```typescript
// **Recursive type**: Type that references itself
type JSONValue =
  | string
  | number
  | boolean
  | null
  | JSONValue[]
  | { [key: string]: JSONValue };

const data: JSONValue = {
  name: "John",
  age: 30,
  hobbies: ["reading", "gaming"],
  address: {
    street: "123 Main St",
    coordinates: [40.7, -74.0]
  }
};

// **Tree structure**
interface TreeNode<T> {
  value: T;
  children?: TreeNode<T>[];
}

const tree: TreeNode<number> = {
  value: 1,
  children: [
    { value: 2 },
    {
      value: 3,
      children: [
        { value: 4 },
        { value: 5 }
      ]
    }
  ]
};
```

---

## 15. Modules and Namespaces

### ES6 Modules - Export

```typescript
// **Named exports** - math.ts
export function add(a: number, b: number): number {
  return a + b;
}

export function subtract(a: number, b: number): number {
  return a - b;
}

export const PI = 3.14159;

// **Default export** - user.ts
export default class User {
  constructor(public name: string) {}
}

// **Exporting types and interfaces**
export interface Product {
  id: number;
  name: string;
  price: number;
}

export type ProductList = Product[];

// **Re-exporting**
export { add as addition } from './math';
export * from './utilities';
```

### ES6 Modules - Import

```typescript
// **Named imports**
import { add, subtract, PI } from './math';

// **Import with alias**
import { add as addition } from './math';

// **Import default**
import User from './user';

// **Import everything**
import * as MathUtils from './math';
MathUtils.add(1, 2);

// **Import type**
import type { Product } from './types';

// **Mixed import**
import React, { useState, useEffect } from 'react';
```

### Dynamic Imports

```typescript
// **Dynamic import**: Load modules on demand
async function loadModule() {
  const module = await import('./heavy-module');
  module.doSomething();
}

// **Conditional import**
let calculator;
if (needsAdvancedMath) {
  calculator = await import('./advanced-calculator');
} else {
  calculator = await import('./basic-calculator');
}
```

### Namespaces

```typescript
// **Namespace**: Organize code into logical groups
namespace Validation {
  export interface StringValidator {
    isValid(s: string): boolean;
  }
  
  export class ZipCodeValidator implements StringValidator {
    isValid(s: string): boolean {
      return s.length === 5 && /^\d+$/.test(s);
    }
  }
  
  export class EmailValidator implements StringValidator {
    isValid(s: string): boolean {
      return /^[\w.-]+@[\w.-]+\.\w+$/.test(s);
    }
  }
}

// Usage
let zipValidator = new Validation.ZipCodeValidator();
console.log(zipValidator.isValid("12345"));  // true

// **Nested namespaces**
namespace App {
  export namespace Models {
    export class User {
      constructor(public name: string) {}
    }
  }
  
  export namespace Services {
    export class UserService {
      getUser(): Models.User {
        return new Models.User("John");
      }
    }
  }
}
```

### Module Resolution

```typescript
// **Relative imports**: Start with ./ or ../
import { helper } from './utils/helper';
import { config } from '../config';

// **Non-relative imports**: Module name or path mapping
import * as $ from "jquery";
import { Component } from "@angular/core";

// **Path mapping in tsconfig.json**
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@models/*": ["src/models/*"],
      "@utils/*": ["src/utils/*"]
    }
  }
}

// Usage with path mapping
import { User } from "@models/user";
import { helper } from "@utils/helper";
```

---

## 16. Decorators

### Enabling Decorators

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES5",
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  }
}
```

### Class Decorators

```typescript
// **Class decorator**: Modify or replace class definition
function sealed(constructor: Function) {
  Object.seal(constructor);
  Object.seal(constructor.prototype);
}

@sealed
class Greeter {
  greeting: string;
  constructor(message: string) {
    this.greeting = message;
  }
}

// **Decorator factory**: Returns decorator function
function logger(prefix: string) {
  return function (constructor: Function) {
    console.log(`${prefix}: ${constructor.name}`);
  };
}

@logger("Created class")
class Example {}

// **Replacing constructor**
function classDecorator<T extends { new(...args: any[]): {} }>(constructor: T) {
  return class extends constructor {
    newProperty = "new property";
    hello = "override";
  };
}

@classDecorator
class MyClass {
  hello: string;
  constructor(public name: string) {
    this.hello = "original";
  }
}
```

### Method Decorators

```typescript
// **Method decorator**: Modify method behavior
function enumerable(value: boolean) {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    descriptor.enumerable = value;
  };
}

class Person {
  constructor(public name: string) {}
  
  @enumerable(false)
  greet() {
    return `Hello, ${this.name}`;
  }
}

// **Logging decorator**
function log(
  target: any,
  propertyKey: string,
  descriptor: PropertyDescriptor
) {
  const originalMethod = descriptor.value;
  
  descriptor.value = function(...args: any[]) {
    console.log(`Calling ${propertyKey} with`, args);
    const result = originalMethod.apply(this, args);
    console.log(`Result:`, result);
    return result;
  };
  
  return descriptor;
}

class Calculator {
  @log
  add(a: number, b: number): number {
    return a + b;
  }
}

const calc = new Calculator();
calc.add(2, 3);
// Logs: "Calling add with [2, 3]"
// Logs: "Result: 5"
```

### Property Decorators

```typescript
// **Property decorator**: Modify property metadata
function format(formatString: string) {
  return function (target: any, propertyKey: string) {
    let value: string;
    
    const getter = function() {
      return value;
    };
    
    const setter = function(newVal: string) {
      value = formatString.replace("%s", newVal);
    };
    
    Object.defineProperty(target, propertyKey, {
      get: getter,
      set: setter,
      enumerable: true,
      configurable: true
    });
  };
}

class Greeter {
  @format("Hello, %s")
  greeting: string;
  
  constructor(message: string) {
    this.greeting = message;
  }
}

const greeter = new Greeter("World");
console.log(greeter.greeting);  // "Hello, World"
```

### Parameter Decorators

```typescript
// **Parameter decorator**: Record metadata about parameters
function required(
  target: Object,
  propertyKey: string | symbol,
  parameterIndex: number
) {
  console.log(`Parameter at index ${parameterIndex} in ${String(propertyKey)} is required`);
}

class UserService {
  getUser(@required id: number) {
    // Method implementation
  }
}
```

### Accessor Decorators

```typescript
// **Accessor decorator**: Modify getter/setter behavior
function configurable(value: boolean) {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    descriptor.configurable = value;
  };
}

class Point {
  private _x: number;
  private _y: number;
  
  constructor(x: number, y: number) {
    this._x = x;
    this._y = y;
  }
  
  @configurable(false)
  get x() {
    return this._x;
  }
  
  @configurable(false)
  get y() {
    return this._y;
  }
}
```

### Decorator Composition

```typescript
// **Multiple decorators**: Applied from bottom to top
function first() {
  console.log("first(): factory evaluated");
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    console.log("first(): called");
  };
}

function second() {
  console.log("second(): factory evaluated");
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    console.log("second(): called");
  };
}

class ExampleClass {
  @first()
  @second()
  method() {}
}

// Output:
// first(): factory evaluated
// second(): factory evaluated
// second(): called
// first(): called
```

---

## 17. Advanced Generics

### Generic Constraints with keyof

```typescript
// **keyof operator**: Get union of object keys
interface Person {
  name: string;
  age: number;
  email: string;
}

type PersonKeys = keyof Person;  // "name" | "age" | "email"

// **Using keyof in generics**
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const person: Person = {
  name: "John",
  age: 30,
  email: "john@example.com"
};

const name = getProperty(person, "name");    // string
const age = getProperty(person, "age");      // number
// const invalid = getProperty(person, "salary");  // Error!
```

### Generic Type Inference

```typescript
// **Type inference with generics**
function map<Input, Output>(
  arr: Input[],
  func: (arg: Input) => Output
): Output[] {
  return arr.map(func);
}

// TypeScript infers Input as number, Output as string
const stringLengths = map([1, 2, 3], (n) => n.toString());

// **Multiple type parameters with inference**
function zip<A, B>(a: A[], b: B[]): [A, B][] {
  return a.map((aItem, i) => [aItem, b[i]]);
}

const zipped = zip([1, 2, 3], ["a", "b", "c"]);
// Type: [number, string][]
```

### Generic Factory Pattern

```typescript
// **Factory with generics**
interface Constructor<T> {
  new (...args: any[]): T;
}

function create<T>(Ctor: Constructor<T>, ...args: any[]): T {
  return new Ctor(...args);
}

class Product {
  constructor(public name: string, public price: number) {}
}

const product = create(Product, "Laptop", 999);  // Type: Product
```

### Advanced Generic Constraints

```typescript
// **Multiple constraints**
interface HasId {
  id: number;
}

interface HasName {
  name: string;
}

function process<T extends HasId & HasName>(item: T): string {
  return `${item.id}: ${item.name}`;
}

// **Conditional constraints**
type OnlyStringsAndNumbers<T> = T extends string | number ? T : never;

type Test1 = OnlyStringsAndNumbers<string>;   // string
type Test2 = OnlyStringsAndNumbers<boolean>;  // never

// **Recursive generic constraints**
type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends object ? DeepReadonly<T[P]> : T[P];
};

interface NestedObject {
  level1: {
    level2: {
      value: string;
    };
  };
}

type ReadonlyNested = DeepReadonly<NestedObject>;
// All nested properties are readonly
```

---

## 18. Utility Types

### Partial<T>

```typescript
// **Partial**: Make all properties optional
interface User {
  id: number;
  name: string;
  email: string;
}

type PartialUser = Partial<User>;
// { id?: number; name?: string; email?: string; }

function updateUser(user: User, updates: Partial<User>): User {
  return { ...user, ...updates };
}

const user: User = { id: 1, name: "John", email: "john@example.com" };
const updated = updateUser(user, { email: "newemail@example.com" });
```

### Required<T>

```typescript
// **Required**: Make all properties required
interface Config {
  host?: string;
  port?: number;
  debug?: boolean;
}

type RequiredConfig = Required<Config>;
// { host: string; port: number; debug: boolean; }

function initServer(config: Required<Config>) {
  // All properties are guaranteed to exist
  console.log(`Server at ${config.host}:${config.port}`);
}
```

### Readonly<T>

```typescript
// **Readonly**: Make all properties readonly
interface MutablePoint {
  x: number;
  y: number;
}

type ReadonlyPoint = Readonly<MutablePoint>;

const point: ReadonlyPoint = { x: 10, y: 20 };
// point.x = 5;  // Error! Cannot assign to 'x' because it is a read-only property
```

### Pick<T, K>

```typescript
// **Pick**: Select subset of properties
interface Article {
  title: string;
  content: string;
  author: string;
  createdAt: Date;
  updatedAt: Date;
}

type ArticlePreview = Pick<Article, "title" | "author" | "createdAt">;
// { title: string; author: string; createdAt: Date; }

const preview: ArticlePreview = {
  title: "TypeScript Guide",
  author: "John Doe",
  createdAt: new Date()
};
```

### Omit<T, K>

```typescript
// **Omit**: Exclude specified properties
interface User {
  id: number;
  name: string;
  email: string;
  password: string;
}

type UserWithoutPassword = Omit<User, "password">;
// { id: number; name: string; email: string; }

type PublicUser = Omit<User, "password" | "email">;
// { id: number; name: string; }

function displayUser(user: UserWithoutPassword) {
  console.log(`User: ${user.name}`);
  // No access to password
}
```

### Record<K, T>

```typescript
// **Record**: Create type with specific keys and value types
type PageInfo = {
  title: string;
  url: string;
};

type Pages = "home" | "about" | "contact";

const pages: Record<Pages, PageInfo> = {
  home: { title: "Home Page", url: "/" },
  about: { title: "About Us", url: "/about" },
  contact: { title: "Contact", url: "/contact" }
};

// **Dynamic object with specific value type**
type StringMap = Record<string, string>;
const translations: StringMap = {
  hello: "Hola",
  goodbye: "Adiós",
  thanks: "Gracias"
};
```

### Exclude<T, U>

```typescript
// **Exclude**: Remove types from union
type AllTypes = string | number | boolean;
type StringAndNumber = Exclude<AllTypes, boolean>;
// string | number

type T0 = Exclude<"a" | "b" | "c", "a">;  // "b" | "c"
type T1 = Exclude<string | number | (() => void), Function>;  // string | number

// **Practical example**
type Status = "pending" | "active" | "completed" | "error";
type SuccessStatus = Exclude<Status, "error">;
// "pending" | "active" | "completed"
```

### Extract<T, U>

```typescript
// **Extract**: Keep only types that are assignable to U
type T0 = Extract<"a" | "b" | "c", "a" | "f">;  // "a"
type T1 = Extract<string | number | (() => void), Function>;  // () => void

// **Filter union types**
type Primitive = string | number | boolean | null | undefined;
type StringOrNumber = Extract<Primitive, string | number>;
// string | number
```

### NonNullable<T>

```typescript
// **NonNullable**: Remove null and undefined from type
type T0 = NonNullable<string | number | undefined>;
// string | number

type T1 = NonNullable<string[] | null | undefined>;
// string[]

// **Useful in function return types**
function getValue(): string | null {
  return Math.random() > 0.5 ? "value" : null;
}

type Value = NonNullable<ReturnType<typeof getValue>>;
// string
```

### ReturnType<T>

```typescript
// **ReturnType**: Extract function return type
function createUser() {
  return {
    id: 1,
    name: "John",
    email: "john@example.com"
  };
}

type User = ReturnType<typeof createUser>;
// { id: number; name: string; email: string; }

// **With generic function**
function identity<T>(arg: T): T {
  return arg;
}

type T0 = ReturnType<typeof identity>;  // unknown
type T1 = ReturnType<() => string>;     // string
type T2 = ReturnType<(s: string) => void>;  // void
```

### Parameters<T>

```typescript
// **Parameters**: Extract function parameter types as tuple
function createPoint(x: number, y: number, z?: number) {
  return { x, y, z };
}

type PointParams = Parameters<typeof createPoint>;
// [x: number, y: number, z?: number | undefined]

// **Use in wrapper functions**
function logAndCall<T extends (...args: any[]) => any>(
  fn: T,
  ...args: Parameters<T>
): ReturnType<T> {
  console.log("Calling function with:", args);
  return fn(...args);
}
```

### ConstructorParameters<T>

```typescript
// **ConstructorParameters**: Extract constructor parameter types
class Person {
  constructor(public name: string, public age: number) {}
}

type PersonParams = ConstructorParameters<typeof Person>;
// [name: string, age: number]

// **Factory function using constructor parameters**
function createInstance<T extends new (...args: any[]) => any>(
  Ctor: T,
  ...args: ConstructorParameters<T>
): InstanceType<T> {
  return new Ctor(...args);
}

const person = createInstance(Person, "John", 30);
```

### InstanceType<T>

```typescript
// **InstanceType**: Get instance type from constructor
class Animal {
  name: string;
  constructor(name: string) {
    this.name = name;
  }
}

type AnimalInstance = InstanceType<typeof Animal>;
// Animal

// **With generic constructor**
function getInstance<T extends new (...args: any[]) => any>(
  Ctor: T
): InstanceType<T> {
  return new Ctor();
}
```

### Awaited<T>

```typescript
// **Awaited**: Unwrap Promise types (TypeScript 4.5+)
type A = Awaited<Promise<string>>;  // string
type B = Awaited<Promise<Promise<number>>>;  // number
type C = Awaited<boolean | Promise<number>>;  // boolean | number

// **Practical example**
async function fetchUser(): Promise<{ id: number; name: string }> {
  return { id: 1, name: "John" };
}

type User = Awaited<ReturnType<typeof fetchUser>>;
// { id: number; name: string; }
```

---

## 19. Conditional Types

### Basic Conditional Types

```typescript
// **Conditional type syntax**: T extends U ? X : Y
type IsString<T> = T extends string ? true : false;

type A = IsString<string>;   // true
type B = IsString<number>;   // false
type C = IsString<"hello">;  // true

// **Type filtering**
type TypeName<T> =
  T extends string ? "string" :
  T extends number ? "number" :
  T extends boolean ? "boolean" :
  T extends undefined ? "undefined" :
  T extends Function ? "function" :
  "object";

type T0 = TypeName<string>;    // "string"
type T1 = TypeName<"a">;       // "string"
type T2 = TypeName<true>;      // "boolean"
type T3 = TypeName<() => void>; // "function"
```

### Distributive Conditional Types

```typescript
// **Distributive**: Union types are distributed over conditional types
type ToArray<T> = T extends any ? T[] : never;

type StrArrOrNumArr = ToArray<string | number>;
// string[] | number[] (distributed)

// **Non-distributive**: Wrap in tuple to prevent distribution
type ToArrayNonDist<T> = [T] extends [any] ? T[] : never;

type StrOrNumArr = ToArrayNonDist<string | number>;
// (string | number)[] (not distributed)

// **Practical filtering**
type Filter<T, U> = T extends U ? T : never;

type T0 = Filter<string | number | boolean, string | number>;
// string | number
```

### Infer Keyword

```typescript
// **infer**: Infer types within conditional types

// **Extract return type**
type GetReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

type Num = GetReturnType<() => number>;  // number
type Str = GetReturnType<(x: string) => string>;  // string

// **Extract array element type**
type Flatten<T> = T extends Array<infer Item> ? Item : T;

type T0 = Flatten<string[]>;  // string
type T1 = Flatten<number>;    // number

// **Extract Promise value**
type UnwrapPromise<T> = T extends Promise<infer U> ? U : T;

type T2 = UnwrapPromise<Promise<string>>;  // string
type T3 = UnwrapPromise<number>;           // number

// **Multiple infer locations**
type GetFirstParam<T> = T extends (first: infer F, ...rest: any[]) => any ? F : never;

function example(a: string, b: number): boolean {
  return true;
}

type FirstParam = GetFirstParam<typeof example>;  // string
```

### Conditional Type Constraints

```typescript
// **Constraining conditional types**
type MessageOf<T> = T extends { message: unknown } ? T["message"] : never;

interface Email {
  message: string;
}

interface Dog {
  bark(): void;
}

type EmailMessageContents = MessageOf<Email>;  // string
type DogMessageContents = MessageOf<Dog>;      // never

// **Flatten with constraint**
type Flatten<T> = T extends Array<infer Item> ? Item : T;
type GetProperty<T, K extends keyof T> = T extends { [key in K]: infer U } ? U : never;
```

### Recursive Conditional Types

```typescript
// **Recursive types** (TypeScript 4.1+)
type DeepFlatten<T> = T extends Array<infer Item>
  ? DeepFlatten<Item>
  : T;

type T0 = DeepFlatten<number[][][]>;  // number
type T1 = DeepFlatten<string[]>;      // string

// **Deep Partial**
type DeepPartial<T> = T extends object
  ? { [P in keyof T]?: DeepPartial<T[P]> }
  : T;

interface Nested {
  a: {
    b: {
      c: string;
    };
  };
}

type PartialNested = DeepPartial<Nested>;
// All properties at all levels are optional
```

---

## 20. Mapped Types

### Basic Mapped Types

```typescript
// **Mapped type**: Transform each property in a type
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};

type Optional<T> = {
  [P in keyof T]?: T[P];
};

interface Person {
  name: string;
  age: number;
}

type ReadonlyPerson = Readonly<Person>;
// { readonly name: string; readonly age: number; }

type OptionalPerson = Optional<Person>;
// { name?: string; age?: number; }
```

### Mapping Modifiers

```typescript
// **Adding modifiers**: + prefix (default)
type ReadonlyT<T> = {
  +readonly [P in keyof T]: T[P];
};

type OptionalT<T> = {
  +? [P in keyof T]: T[P];
};

// **Removing modifiers**: - prefix
type Mutable<T> = {
  -readonly [P in keyof T]: T[P];
};

type Required<T> = {
  [P in keyof T]-?: T[P];
};

interface MaybeReadonly {
  readonly name: string;
  age?: number;
}

type Writable = Mutable<MaybeReadonly>;
// { name: string; age?: number; }

type Complete = Required<MaybeReadonly>;
// { readonly name: string; age: number; }
```

### Key Remapping

```typescript
// **Key remapping** (TypeScript 4.1+)
type Getters<T> = {
  [P in keyof T as `get${Capitalize<string & P>}`]: () => T[P];
};

interface Person {
  name: string;
  age: number;
}

type PersonGetters = Getters<Person>;
// { getName: () => string; getAge: () => number; }

// **Filter properties**
type RemoveKindField<T> = {
  [K in keyof T as Exclude<K, "kind">]: T[K];
};

interface Circle {
  kind: "circle";
  radius: number;
}

type WithoutKind = RemoveKindField<Circle>;
// { radius: number; }
```

### Conditional Mapping

```typescript
// **Conditional property types**
type Stringify<T> = {
  [P in keyof T]: T[P] extends Function ? T[P] : string;
};

interface User {
  name: string;
  age: number;
  greet(): void;
}

type StringifiedUser = Stringify<User>;
// { name: string; age: string; greet: () => void; }

// **Extract specific property types**
type ExtractStrings<T> = {
  [P in keyof T as T[P] extends string ? P : never]: T[P];
};

interface Data {
  id: number;
  name: string;
  email: string;
  count: number;
}

type StringProperties = ExtractStrings<Data>;
// { name: string; email: string; }
```

### Recursive Mapped Types

```typescript
// **Deep transformation**
type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends object
    ? DeepReadonly<T[P]>
    : T[P];
};

interface Config {
  server: {
    host: string;
    port: number;
    options: {
      timeout: number;
    };
  };
}

type ReadonlyConfig = DeepReadonly<Config>;
// All nested properties are readonly

// **Deep Nullable**
type DeepNullable<T> = {
  [P in keyof T]: T[P] extends object
    ? DeepNullable<T[P]>
    : T[P] | null;
};
```

---

## 21. Template Literal Types

### Basic Template Literals

```typescript
// **Template literal types** (TypeScript 4.1+)
type World = "world";
type Greeting = `hello ${World}`;  // "hello world"

// **With unions**
type EmailLocaleIDs = "welcome_email" | "email_heading";
type FooterLocaleIDs = "footer_title" | "footer_sendoff";

type AllLocaleIDs = `${EmailLocaleIDs | FooterLocaleIDs}_id`;
// "welcome_email_id" | "email_heading_id" | "footer_title_id" | "footer_sendoff_id"

// **Cartesian product**
type Color = "red" | "blue";
type Size = "small" | "large";

type ColorSize = `${Color}-${Size}`;
// "red-small" | "red-large" | "blue-small" | "blue-large"
```

### String Manipulation Types

```typescript
// **Intrinsic string manipulation types**

// **Uppercase**: Convert to uppercase
type UpperGreeting = Uppercase<"hello">;  // "HELLO"
type Shout<T extends string> = `${Uppercase<T>}!`;
type ShoutHello = Shout<"hello">;  // "HELLO!"

// **Lowercase**: Convert to lowercase
type LowerGreeting = Lowercase<"HELLO">;  // "hello"

// **Capitalize**: Capitalize first letter
type CapitalizedGreeting = Capitalize<"hello">;  // "Hello"

// **Uncapitalize**: Lowercase first letter
type UncapitalizedGreeting = Uncapitalize<"Hello">;  // "hello"

// **Practical example: Event names**
type PropEventSource<T> = {
  on<K extends string & keyof T>(
    eventName: `${K}Changed`,
    callback: (newValue: T[K]) => void
  ): void;
};

declare function makeWatchedObject<T>(obj: T): T & PropEventSource<T>;

const person = makeWatchedObject({
  firstName: "Homer",
  age: 42
});

person.on("firstNameChanged", (newName) => {
  console.log(`Name changed to ${newName}`);
});

// person.on("firstName", () => {});  // Error! Must be "firstNameChanged"
```

### Advanced Template Patterns

```typescript
// **HTTP method types**
type HTTPMethod = "GET" | "POST" | "PUT" | "DELETE";
type Endpoint = "/users" | "/posts" | "/comments";

type APIRoute = `${HTTPMethod} ${Endpoint}`;
// "GET /users" | "POST /users" | ... (12 combinations)

// **CSS property generator**
type CSSProperty = "margin" | "padding";
type Side = "top" | "right" | "bottom" | "left";

type CSSProperties = `${CSSProperty}-${Side}`;
// "margin-top" | "margin-right" | ... (8 combinations)

// **Database table and column**
type Table = "users" | "posts";
type Column = "id" | "name" | "createdAt";

type TableColumn = `${Table}.${Column}`;
// "users.id" | "users.name" | ... (6 combinations)
```

### Recursive Template Types

```typescript
// **Path type for nested objects**
type Path<T> = T extends object
  ? {
      [K in keyof T]: K extends string
        ? T[K] extends object
          ? `${K}` | `${K}.${Path<T[K]>}`
          : `${K}`
        : never;
    }[keyof T]
  : never;

interface User {
  name: string;
  address: {
    street: string;
    city: string;
    coordinates: {
      lat: number;
      lng: number;
    };
  };
}

type UserPath = Path<User>;
// "name" | "address" | "address.street" | "address.city" | 
// "address.coordinates" | "address.coordinates.lat" | "address.coordinates.lng"
```

---

## 22. Advanced Patterns

### Builder Pattern

```typescript
// **Fluent builder pattern with type safety**
class QueryBuilder<T> {
  private conditions: string[] = [];
  
  where(condition: string): this {
    this.conditions.push(condition);
    return this;
  }
  
  and(condition: string): this {
    this.conditions.push(`AND ${condition}`);
    return this;
  }
  
  or(condition: string): this {
    this.conditions.push(`OR ${condition}`);
    return this;
  }
  
  build(): string {
    return this.conditions.join(" ");
  }
}

const query = new QueryBuilder()
  .where("age > 18")
  .and("status = 'active'")
  .or("role = 'admin'")
  .build();
```

### Factory Pattern

```typescript
// **Abstract factory with generics**
interface Product {
  operation(): string;
}

class ConcreteProductA implements Product {
  operation(): string {
    return "Product A";
  }
}

class ConcreteProductB implements Product {
  operation(): string {
    return "Product B";
  }
}

abstract class Creator {
  abstract factoryMethod(): Product;
  
  someOperation(): string {
    const product = this.factoryMethod();
    return `Creator: ${product.operation()}`;
  }
}

class ConcreteCreatorA extends Creator {
  factoryMethod(): Product {
    return new ConcreteProductA();
  }
}

class ConcreteCreatorB extends Creator {
  factoryMethod(): Product {
    return new ConcreteProductB();
  }
}

// **Type-safe factory function**
type ProductType = "A" | "B";

function createProduct(type: ProductType): Product {
  switch (type) {
    case "A":
      return new ConcreteProductA();
    case "B":
      return new ConcreteProductB();
  }
}
```

### Repository Pattern

```typescript
// **Generic repository interface**
interface Repository<T> {
  findById(id: number): Promise<T | null>;
  findAll(): Promise<T[]>;
  create(item: Omit<T, "id">): Promise<T>;
  update(id: number, item: Partial<T>): Promise<T | null>;
  delete(id: number): Promise<boolean>;
}

interface User {
  id: number;
  name: string;
  email: string;
}

// **Implementation**
class UserRepository implements Repository<User> {
  private users: User[] = [];
  private nextId = 1;
  
  async findById(id: number): Promise<User | null> {
    return this.users.find(u => u.id === id) || null;
  }
  
  async findAll(): Promise<User[]> {
    return [...this.users];
  }
  
  async create(item: Omit<User, "id">): Promise<User> {
    const user: User = { ...item, id: this.nextId++ };
    this.users.push(user);
    return user;
  }
  
  async update(id: number, item: Partial<User>): Promise<User | null> {
    const index = this.users.findIndex(u => u.id === id);
    if (index === -1) return null;
    
    this.users[index] = { ...this.users[index], ...item };
    return this.users[index];
  }
  
  async delete(id: number): Promise<boolean> {
    const index = this.users.findIndex(u => u.id === id);
    if (index === -1) return false;
    
    this.users.splice(index, 1);
    return true;
  }
}
```

### Observer Pattern

```typescript
// **Type-safe observer pattern**
type EventMap = {
  userLoggedIn: { userId: number; timestamp: Date };
  userLoggedOut: { userId: number };
  dataChanged: { key: string; value: any };
};

type EventCallback<T> = (data: T) => void;

class EventEmitter<T extends EventMap> {
  private listeners: {
    [K in keyof T]?: EventCallback<T[K]>[];
  } = {};
  
  on<K extends keyof T>(event: K, callback: EventCallback<T[K]>): void {
    if (!this.listeners[event]) {
      this.listeners[event] = [];
    }
    this.listeners[event]!.push(callback);
  }
  
  off<K extends keyof T>(event: K, callback: EventCallback<T[K]>): void {
    const callbacks = this.listeners[event];
    if (callbacks) {
      const index = callbacks.indexOf(callback);
      if (index > -1) {
        callbacks.splice(index, 1);
      }
    }
  }
  
  emit<K extends keyof T>(event: K, data: T[K]): void {
    const callbacks = this.listeners[event];
    if (callbacks) {
      callbacks.forEach(callback => callback(data));
    }
  }
}

// Usage
const emitter = new EventEmitter<EventMap>();

emitter.on("userLoggedIn", (data) => {
  console.log(`User ${data.userId} logged in at ${data.timestamp}`);
});

emitter.emit("userLoggedIn", {
  userId: 123,
  timestamp: new Date()
});
```

### Dependency Injection

```typescript
// **Type-safe dependency injection**
interface Logger {
  log(message: string): void;
}

interface Database {
  query(sql: string): Promise<any[]>;
}

class ConsoleLogger implements Logger {
  log(message: string): void {
    console.log(message);
  }
}

class MockDatabase implements Database {
  async query(sql: string): Promise<any[]> {
    return [];
  }
}

// **Service with dependencies**
class UserService {
  constructor(
    private logger: Logger,
    private database: Database
  ) {}
  
  async getUser(id: number) {
    this.logger.log(`Fetching user ${id}`);
    const results = await this.database.query(`SELECT * FROM users WHERE id = ${id}`);
    return results[0];
  }
}

// **DI Container**
class Container {
  private services = new Map<string, any>();
  
  register<T>(name: string, instance: T): void {
    this.services.set(name, instance);
  }
  
  resolve<T>(name: string): T {
    const service = this.services.get(name);
    if (!service) {
      throw new Error(`Service ${name} not found`);
    }
    return service;
  }
}

// Setup
const container = new Container();
container.register<Logger>("logger", new ConsoleLogger());
container.register<Database>("database", new MockDatabase());

const userService = new UserService(
  container.resolve<Logger>("logger"),
  container.resolve<Database>("database")
);
```

### Mixins

```typescript
// **Mixin pattern for class composition**
type Constructor<T = {}> = new (...args: any[]) => T;

// **Mixin functions**
function Timestamped<TBase extends Constructor>(
  Base: TBase
) {
  return class extends Base {
    timestamp = new Date();
    
    getTimestamp() {
      return this.timestamp;
    }
  };
}

function Taggable<TBase extends Constructor>(
  Base: TBase
) {
  return class extends Base {
    tags: string[] = [];
    
    addTag(tag: string) {
      this.tags.push(tag);
    }
    
    getTags() {
      return [...this.tags];
    }
  };
}

// **Base class**
class Post {
  constructor(public title: string, public content: string) {}
}

// **Apply mixins**
const TimestampedPost = Timestamped(Post);
const TaggablePost = Taggable(Post);
const EnhancedPost = Taggable(Timestamped(Post));

const post = new EnhancedPost("Hello", "World");
post.addTag("typescript");
console.log(post.getTimestamp());
console.log(post.getTags());
```

---

## 23. Best Practices

### Type Inference Best Practices

```typescript
// ✅ **Good**: Let TypeScript infer when obvious
const numbers = [1, 2, 3];  // number[]
const name = "John";        // string

// ❌ **Avoid**: Redundant type annotations
const numbers: number[] = [1, 2, 3];
const name: string = "John";

// ✅ **Good**: Explicit types for function parameters
function greet(name: string): void {
  console.log(`Hello, ${name}`);
}

// ✅ **Good**: Explicit return types for public APIs
export function calculateTotal(items: Item[]): number {
  return items.reduce((sum, item) => sum + item.price, 0);
}
```

### Strict Mode Configuration

```json
// ✅ **Recommended**: Enable strict mode
{
  "compilerOptions": {
    "strict": true,                  // Enable all strict checks
    "noImplicitAny": true,           // Error on implicit any
    "strictNullChecks": true,        // Null and undefined are distinct
    "strictFunctionTypes": true,     // Stricter function type checking
    "strictBindCallApply": true,     // Check bind, call, apply
    "strictPropertyInitialization": true,
    "noImplicitThis": true,          // Error on implicit 'this'
    "alwaysStrict": true             // Parse in strict mode
  }
}
```

### Avoid Any Type

```typescript
// ❌ **Avoid**: Using any loses type safety
function process(data: any) {
  return data.value.toUpperCase();  // No type checking!
}

// ✅ **Better**: Use unknown for truly unknown types
function process(data: unknown) {
  if (typeof data === "object" && data !== null && "value" in data) {
    const value = (data as { value: unknown }).value;
    if (typeof value === "string") {
      return value.toUpperCase();
    }
  }
  throw new Error("Invalid data");
}

// ✅ **Best**: Define proper types
interface ProcessData {
  value: string;
}

function process(data: ProcessData) {
  return data.value.toUpperCase();
}
```

### Prefer Interfaces Over Type Aliases for Objects

```typescript
// ✅ **Good**: Interface for object shapes (extendable, declaration merging)
interface User {
  id: number;
  name: string;
}

interface User {
  email: string;  // Declaration merging
}

// ✅ **Good**: Type for unions, tuples, primitives
type Status = "pending" | "active" | "completed";
type Point = [number, number];
type Callback = (result: string) => void;
```

### Use Const Assertions

```typescript
// ❌ **Avoid**: Mutable inferred types
const config = {
  apiUrl: "https://api.example.com",
  timeout: 5000
};
// Type: { apiUrl: string; timeout: number }

// ✅ **Good**: Const assertion for immutable, literal types
const config = {
  apiUrl: "https://api.example.com",
  timeout: 5000
} as const;
// Type: { readonly apiUrl: "https://api.example.com"; readonly timeout: 5000 }

// **Array example**
const colors = ["red", "green", "blue"] as const;
// Type: readonly ["red", "green", "blue"]
```

### Proper Error Handling

```typescript
// ❌ **Avoid**: Untyped errors
try {
  riskyOperation();
} catch (error) {
  console.log(error.message);  // error is 'any'
}

// ✅ **Good**: Type guard for errors
try {
  riskyOperation();
} catch (error) {
  if (error instanceof Error) {
    console.log(error.message);
  } else {
    console.log("Unknown error", error);
  }
}

// ✅ **Better**: Custom error types
class ValidationError extends Error {
  constructor(public field: string, message: string) {
    super(message);
    this.name = "ValidationError";
  }
}

try {
  throw new ValidationError("email", "Invalid email format");
} catch (error) {
  if (error instanceof ValidationError) {
    console.log(`Validation failed for ${error.field}: ${error.message}`);
  } else if (error instanceof Error) {
    console.log(error.message);
  }
}
```

### Use Discriminated Unions

```typescript
// ✅ **Good**: Discriminated unions for type-safe state management
type State =
  | { status: "loading" }
  | { status: "success"; data: string }
  | { status: "error"; error: Error };

function handleState(state: State) {
  switch (state.status) {
    case "loading":
      console.log("Loading...");
      break;
    case "success":
      console.log(state.data);  // TypeScript knows data exists
      break;
    case "error":
      console.log(state.error.message);  // TypeScript knows error exists
      break;
  }
}
```

### Prefer Immutability

```typescript
// ✅ **Good**: Use readonly for immutable data
interface Config {
  readonly apiUrl: string;
  readonly timeout: number;
}

const config: Config = {
  apiUrl: "https://api.example.com",
  timeout: 5000
};

// config.apiUrl = "new url";  // Error!

// ✅ **Good**: ReadonlyArray for immutable arrays
function processItems(items: ReadonlyArray<string>) {
  // items.push("new");  // Error!
  return items.map(item => item.toUpperCase());  // OK
}
```

### Utility Types for Transformations

```typescript
// ✅ **Good**: Use built-in utility types
interface User {
  id: number;
  name: string;
  email: string;
  password: string;
}

// Public user (without password)
type PublicUser = Omit<User, "password">;

// User update (all fields optional)
type UserUpdate = Partial<User>;

// User creation (without id)
type UserCreate = Omit<User, "id">;

// Required user (all fields required)
type RequiredUser = Required<User>;
```

### Function Overloading for Flexibility

```typescript
// ✅ **Good**: Overloads for different call signatures
function createElement(tag: "a"): HTMLAnchorElement;
function createElement(tag: "div"): HTMLDivElement;
function createElement(tag: "span"): HTMLSpanElement;
function createElement(tag: string): HTMLElement {
  return document.createElement(tag);
}

const anchor = createElement("a");    // Type: HTMLAnchorElement
const div = createElement("div");    // Type: HTMLDivElement
```

### Organize Code with Namespaces and Modules

```typescript
// ✅ **Good**: Use ES6 modules for code organization
// user.model.ts
export interface User {
  id: number;
  name: string;
}

// user.service.ts
import { User } from './user.model';

export class UserService {
  getUser(id: number): User {
    // Implementation
  }
}

// ✅ **Good**: Barrel exports for clean imports
// index.ts
export * from './user.model';
export * from './user.service';

// Usage
import { User, UserService } from './users';
```

### Documentation and Comments

```typescript
// ✅ **Good**: Use JSDoc for API documentation
/**
 * Calculates the total price including tax
 * @param price - The base price
 * @param taxRate - Tax rate as decimal (e.g., 0.08 for 8%)
 * @returns The total price with tax
 * @throws {Error} If price or taxRate is negative
 */
function calculateTotal(price: number, taxRate: number): number {
  if (price < 0 || taxRate < 0) {
    throw new Error("Price and tax rate must be non-negative");
  }
  return price * (1 + taxRate);
}

// ✅ **Good**: Types are self-documenting
interface PaymentRequest {
  /** Unique transaction identifier */
  transactionId: string;
  /** Amount in cents */
  amount: number;
  /** ISO 4217 currency code */
  currency: "USD" | "EUR" | "GBP";
}
```

### Testing TypeScript Code

```typescript
// ✅ **Good**: Type-safe test assertions
import { describe, it, expect } from '@jest/globals';

interface Calculator {
  add(a: number, b: number): number;
}

class SimpleCalculator implements Calculator {
  add(a: number, b: number): number {
    return a + b;
  }
}

describe('Calculator', () => {
  let calculator: Calculator;
  
  beforeEach(() => {
    calculator = new SimpleCalculator();
  });
  
  it('should add two numbers', () => {
    const result = calculator.add(2, 3);
    expect(result).toBe(5);
  });
});
```

---

## Conclusion

This comprehensive tutorial has covered TypeScript from beginner to advanced topics:

### **Beginner Level:**
- Basic types and type annotations
- Functions and interfaces
- Classes and enums
- Type assertions and guards

### **Intermediate Level:**
- Union and intersection types
- Generics and constraints
- Advanced types (mapped, conditional)
- Modules and namespaces

### **Advanced Level:**
- Template literal types
- Recursive types
- Advanced patterns (Builder, Factory, Observer)
- Type-level programming
- Best practices and optimization

### **Next Steps:**
1. Practice with real projects
2. Explore TypeScript with frameworks (React, Angular, Vue)
3. Learn about type-level programming libraries (zod, type-fest)
4. Contribute to TypeScript open-source projects
5. Stay updated with TypeScript releases

### **Resources:**
- Official TypeScript Handbook: https://www.typescriptlang.org/docs/
- TypeScript Playground: https://www.typescriptlang.org/play
- Definitely Typed: https://github.com/DefinitelyTyped/DefinitelyTyped
- TypeScript Deep Dive: https://basarat.gitbook.io/typescript/

**Remember:** TypeScript is a journey. Start with the basics, practice regularly, and gradually advance to more complex patterns. The type system is your friend—it catches bugs before they reach production!

---

**Happy TypeScript Coding! 🚀**