# TypeScript & Express.js Complete Mastery Tutorial 🚀

> **A comprehensive guide from beginner to ninja level** - Master TypeScript fundamentals, advanced patterns, OOP concepts, design patterns, and Express.js integration with real-world examples.

---

## Table of Contents

1. [Introduction to TypeScript](#introduction-to-typescript)
2. [Understanding TypeScript and Type Interfacing](#understanding-typescript-and-type-interfacing)
3. [Generics and Interfaces in TypeScript](#generics-and-interfaces-in-typescript)
4. [Key Patterns in TypeScript - Utility Types and Mapped Types](#key-patterns-in-typescript)
5. [Object-Oriented Programming with TypeScript](#object-oriented-programming-with-typescript)
6. [Low-Level Design Patterns in TypeScript](#low-level-design-patterns-in-typescript)
7. [Express.js with TypeScript](#expressjs-with-typescript)
8. [Advanced Topics and Best Practices](#advanced-topics-and-best-practices)
9. [Real-World Projects](#real-world-projects)
10. [Conclusion](#conclusion)

---

## Introduction to TypeScript

### What is TypeScript?

TypeScript is a **statically-typed superset of JavaScript** that compiles to plain JavaScript. It adds optional static typing, classes, and interfaces to JavaScript, enabling better tooling, code quality, and maintainability.

**Key Benefits:**
- **Type Safety**: Catch errors at compile-time instead of runtime
- **Better IDE Support**: Autocomplete, refactoring, and navigation
- **Self-Documenting Code**: Types serve as inline documentation
- **Scalability**: Easier to maintain large codebases
- **Modern Features**: Access to latest ECMAScript features

### Setting Up TypeScript

```bash
# Install TypeScript globally
npm install -g typescript

# Check version
tsc --version

# Initialize a TypeScript project
npm init -y
tsc --init
```

### Basic TypeScript Configuration (tsconfig.json)

```json
{
  "compilerOptions": {
    "target": "ES2020",                  // Target JavaScript version
    "module": "commonjs",                // Module system to use
    "lib": ["ES2020"],                   // Standard library to include
    "outDir": "./dist",                  // Output directory for compiled files
    "rootDir": "./src",                  // Root directory of source files
    "strict": true,                      // Enable all strict type-checking options
    "esModuleInterop": true,             // Enables emit interoperability between CommonJS and ES Modules
    "skipLibCheck": true,                // Skip type checking of declaration files
    "forceConsistentCasingInFileNames": true, // Ensure consistent casing in imports
    "resolveJsonModule": true,           // Allow importing JSON modules
    "declaration": true,                 // Generate .d.ts declaration files
    "sourceMap": true,                   // Generate source maps for debugging
    "removeComments": true,              // Remove comments from output
    "noImplicitAny": true,               // Raise error on expressions with implied 'any' type
    "strictNullChecks": true,            // Enable strict null checks
    "strictFunctionTypes": true,         // Enable strict checking of function types
    "noUnusedLocals": true,              // Report errors on unused locals
    "noUnusedParameters": true,          // Report errors on unused parameters
    "noImplicitReturns": true,           // Report error when not all code paths return a value
    "noFallthroughCasesInSwitch": true   // Report errors for fallthrough cases in switch
  },
  "include": ["src/**/*"],               // Files to include in compilation
  "exclude": ["node_modules", "dist"]    // Files to exclude from compilation
}
```

---

## Understanding TypeScript and Type Interfacing

### Basic Types

TypeScript provides several basic types to define the shape of data:

```typescript
// ============================================
// PRIMITIVE TYPES
// ============================================

// Boolean - represents true/false values
let isDone: boolean = false;
let isActive: boolean = true;

// Number - all numbers in TypeScript are floating point values
let decimal: number = 6;           // Decimal number
let hex: number = 0xf00d;          // Hexadecimal (61453 in decimal)
let binary: number = 0b1010;       // Binary (10 in decimal)
let octal: number = 0o744;         // Octal (484 in decimal)
let big: bigint = 100n;            // BigInt for very large integers

// String - textual data enclosed in single, double quotes, or backticks
let color: string = "blue";
let fullName: string = `John Doe`;
let age: number = 30;
// Template strings allow embedded expressions using ${}
let sentence: string = `Hello, my name is ${fullName}. I'll be ${age + 1} years old next year.`;

// ============================================
// SPECIAL TYPES
// ============================================

// Void - typically used as return type for functions that don't return a value
function warnUser(): void {
  console.log("This is a warning message");
  // No return statement needed
}

// Null and Undefined - represent absence of value
let u: undefined = undefined;  // Variable explicitly has no value assigned
let n: null = null;            // Variable explicitly set to null

// Never - represents values that never occur (e.g., functions that always throw)
function error(message: string): never {
  throw new Error(message);    // Function never returns normally
}

// Function that never reaches its end point
function infiniteLoop(): never {
  while (true) {
    // Infinite loop - never returns
  }
}

// Unknown - type-safe counterpart of 'any' (requires type checking before use)
let notSure: unknown = 4;
notSure = "maybe a string instead";
notSure = false;

// Must narrow down the type before using unknown
if (typeof notSure === "string") {
  console.log(notSure.toUpperCase()); // OK - we've verified it's a string
}

// Any - opt-out of type checking (use sparingly!)
let looselyTyped: any = 4;
looselyTyped = "can be changed to string";
looselyTyped = false;            // No type checking occurs
looselyTyped.toFixed();          // No error even if method doesn't exist

// ============================================
// ARRAY TYPES
// ============================================

// Array - ordered collection of values of the same type
let list: number[] = [1, 2, 3];           // Array of numbers (preferred syntax)
let names: Array<string> = ["Alice", "Bob"]; // Generic array syntax

// Multi-dimensional arrays
let matrix: number[][] = [
  [1, 2, 3],
  [4, 5, 6],
  [7, 8, 9]
];

// ============================================
// TUPLE TYPES
// ============================================

// Tuple - array with fixed number of elements whose types are known
let tuple: [string, number];
tuple = ["hello", 10];           // OK - matches the tuple definition
// tuple = [10, "hello"];        // Error - wrong order
// tuple = ["hello", 10, true];  // Error - too many elements

// Accessing tuple elements - types are enforced
console.log(tuple[0].substring(1)); // OK - tuple[0] is string
// console.log(tuple[1].substring(1)); // Error - tuple[1] is number

// Tuple with optional elements
let optionalTuple: [string, number?];
optionalTuple = ["hello"];       // OK - second element is optional
optionalTuple = ["hello", 10];   // Also OK

// Rest elements in tuples
let restTuple: [string, ...number[]];
restTuple = ["hello", 1, 2, 3, 4]; // OK - first is string, rest are numbers

// ============================================
// ENUM TYPES
// ============================================

// Numeric Enum - auto-incrementing values starting from 0
enum Direction {
  Up,        // 0
  Down,      // 1
  Left,      // 2
  Right      // 3
}

let dir: Direction = Direction.Up;  // dir = 0
console.log(Direction.Down);        // 1

// Numeric Enum with custom initial value
enum Status {
  Pending = 1,    // 1
  Active,         // 2 (auto-incremented)
  Completed       // 3 (auto-incremented)
}

// String Enum - each member must be initialized with string literal
enum FileAccess {
  Read = "READ",
  Write = "WRITE",
  Execute = "EXECUTE"
}

// Heterogeneous Enum - mixed string and numeric (not recommended)
enum Mixed {
  No = 0,
  Yes = "YES"
}

// Const Enum - completely removed during compilation (better performance)
const enum Directions {
  Up,
  Down,
  Left,
  Right
}

let directions = [
  Directions.Up,    // Replaced with literal 0 at compile time
  Directions.Down   // Replaced with literal 1 at compile time
];

// ============================================
// OBJECT TYPES
// ============================================

// Object - represents non-primitive types
let obj: object = { name: "John" };  // Accepts any object

// Specific object shape
let person: { name: string; age: number } = {
  name: "Alice",
  age: 30
};

// Optional properties using ?
let optionalPerson: { name: string; age?: number } = {
  name: "Bob"  // age is optional
};

// Readonly properties - cannot be changed after initialization
let readonlyPerson: { readonly name: string; age: number } = {
  name: "Charlie",
  age: 25
};
// readonlyPerson.name = "David"; // Error - cannot reassign readonly property

// Index signatures - for objects with unknown property names
let dictionary: { [key: string]: number } = {
  "apple": 1,
  "banana": 2,
  "orange": 3
};

// ============================================
// TYPE ASSERTIONS
// ============================================

// Type assertions - tell compiler to trust you about the type
let someValue: unknown = "this is a string";

// Angle-bracket syntax (not usable in JSX/TSX)
let strLength1: number = (<string>someValue).length;

// As-syntax (preferred, works in JSX/TSX)
let strLength2: number = (someValue as string).length;

// ============================================
// LITERAL TYPES
// ============================================

// String literal types - exact string value
let literalString: "hello" = "hello";
// literalString = "world";  // Error - must be exactly "hello"

// Numeric literal types
let literalNumber: 42 = 42;

// Boolean literal types
let literalBoolean: true = true;

// Union of literal types (useful for specific allowed values)
type CardinalDirection = "North" | "East" | "South" | "West";
let direction: CardinalDirection = "North";  // OK
// direction = "Northeast";  // Error - not in allowed values

type DiceRoll = 1 | 2 | 3 | 4 | 5 | 6;
let roll: DiceRoll = 4;  // OK
// roll = 7;  // Error - must be 1-6

// ============================================
// UNION TYPES
// ============================================

// Union - value can be one of several types
let multiType: number | string;
multiType = 42;           // OK - number
multiType = "hello";      // OK - string
// multiType = true;      // Error - boolean not in union

// Union with literal types
type Status = "success" | "error" | "pending";
let currentStatus: Status = "success";

// Function parameter with union type
function printId(id: number | string) {
  // Type narrowing with typeof
  if (typeof id === "string") {
    console.log(`ID (uppercase): ${id.toUpperCase()}`);
  } else {
    console.log(`ID (number): ${id}`);
  }
}

printId(101);        // ID (number): 101
printId("abc123");   // ID (uppercase): ABC123

// ============================================
// INTERSECTION TYPES
// ============================================

// Intersection - combines multiple types into one
type Person = {
  name: string;
  age: number;
};

type Employee = {
  employeeId: number;
  department: string;
};

// Combine Person and Employee
type EmployeePerson = Person & Employee;

let emp: EmployeePerson = {
  name: "John",
  age: 30,
  employeeId: 12345,
  department: "Engineering"
  // Must have all properties from both types
};

// ============================================
// TYPE ALIASES
// ============================================

// Type alias - create a name for any type
type ID = number | string;
type Point = { x: number; y: number };
type Callback = (data: string) => void;

let userId: ID = 123;
let userName: ID = "user123";

let point: Point = { x: 10, y: 20 };

let processData: Callback = (data) => {
  console.log(`Processing: ${data}`);
};
```

### Type Interfacing

Interfaces define the structure of objects and can be implemented by classes:

```typescript
// ============================================
// BASIC INTERFACES
// ============================================

// Interface - defines contract for object shape
interface User {
  id: number;              // Required property
  name: string;            // Required property
  email: string;           // Required property
  age?: number;            // Optional property (may or may not exist)
  readonly createdAt: Date; // Readonly property (cannot be modified after creation)
}

// Creating an object that implements the User interface
const user: User = {
  id: 1,
  name: "Alice",
  email: "alice@example.com",
  createdAt: new Date()
  // age is optional, so we can omit it
};

// user.createdAt = new Date(); // Error - cannot modify readonly property
user.age = 25;  // OK - can add optional property later

// ============================================
// FUNCTION TYPE INTERFACES
// ============================================

// Interface for function signature
interface MathOperation {
  (x: number, y: number): number;  // Function that takes two numbers and returns a number
}

// Implement the function interface
const add: MathOperation = (x, y) => {
  return x + y;  // Parameters and return type inferred from interface
};

const subtract: MathOperation = (x, y) => x - y;

console.log(add(5, 3));       // 8
console.log(subtract(10, 4)); // 6

// ============================================
// INDEXABLE TYPES
// ============================================

// Interface with index signature - for array-like or dictionary objects
interface StringArray {
  [index: number]: string;  // Indexed with number, returns string
}

let myArray: StringArray = ["Bob", "Alice"];
let firstItem: string = myArray[0];  // "Bob"

// Dictionary with string keys
interface NumberDictionary {
  [key: string]: number;  // Any string key maps to a number value
  length: number;         // Specific named property
  // name: string;        // Error - all properties must be number (index signature type)
}

let scores: NumberDictionary = {
  "math": 95,
  "english": 88,
  "science": 92,
  length: 3
};

// ============================================
// EXTENDING INTERFACES
// ============================================

// Base interface
interface Animal {
  name: string;
  age: number;
}

// Extended interface - inherits all properties from Animal
interface Dog extends Animal {
  breed: string;           // Additional property specific to Dog
  bark(): void;            // Additional method
}

const myDog: Dog = {
  name: "Buddy",
  age: 3,
  breed: "Golden Retriever",
  bark() {
    console.log("Woof! Woof!");
  }
};

// Multiple interface extension
interface Swimmer {
  swim(): void;
}

interface Flyer {
  fly(): void;
}

// Interface extending multiple interfaces
interface Duck extends Animal, Swimmer, Flyer {
  quack(): void;
}

const duck: Duck = {
  name: "Donald",
  age: 2,
  swim() { console.log("Swimming..."); },
  fly() { console.log("Flying..."); },
  quack() { console.log("Quack!"); }
};

// ============================================
// INTERFACE VS TYPE ALIAS
// ============================================

// Interface - can be reopened to add new properties (declaration merging)
interface Window {
  title: string;
}

interface Window {
  size: number;  // Adds to existing Window interface
}

const win: Window = {
  title: "Main Window",
  size: 1024
};

// Type alias - cannot be reopened
type Point = {
  x: number;
  y: number;
};

// type Point = {    // Error - duplicate identifier
//   z: number;
// };

// ============================================
// HYBRID TYPES
// ============================================

// Interface that acts as both function and object
interface Counter {
  (start: number): string;  // Callable signature
  interval: number;          // Property
  reset(): void;             // Method
}

// Creating a Counter - requires careful implementation
function getCounter(): Counter {
  // Create function that will be the Counter
  let counter = (function (start: number) {
    return `Starting from ${start}`;
  }) as Counter;

  // Add properties to the function
  counter.interval = 123;
  counter.reset = function () {
    console.log("Counter reset");
  };

  return counter;
}

let c = getCounter();
console.log(c(10));        // "Starting from 10" (calling as function)
c.reset();                 // "Counter reset" (calling method)
c.interval = 5.0;          // Setting property

// ============================================
// INTERFACE FOR CLASS IMPLEMENTATION
// ============================================

// Interface defining what a class must implement
interface ClockInterface {
  currentTime: Date;          // Property that must exist
  setTime(d: Date): void;     // Method that must be implemented
  getTime(): Date;            // Method that must be implemented
}

// Class implementing the interface
class Clock implements ClockInterface {
  currentTime: Date = new Date();  // Required property
  
  // Required method implementation
  setTime(d: Date) {
    this.currentTime = d;
  }
  
  // Required method implementation
  getTime(): Date {
    return this.currentTime;
  }
  
  // Additional methods/properties not in interface are allowed
  constructor(h: number, m: number) {
    console.log(`Clock created: ${h}:${m}`);
  }
}

const clock = new Clock(12, 30);
clock.setTime(new Date());
console.log(clock.getTime());

// ============================================
// INTERFACE WITH CONSTRUCTOR SIGNATURE
// ============================================

// Interface for the constructor itself
interface ClockConstructor {
  new (hour: number, minute: number): ClockInterface;  // Constructor signature
}

// Interface for instance methods
interface ClockInterface2 {
  tick(): void;
}

// Factory function that uses constructor interface
function createClock(
  ctor: ClockConstructor,  // Constructor must match this signature
  hour: number,
  minute: number
): ClockInterface2 {
  return new ctor(hour, minute);  // Create instance using provided constructor
}

// Digital clock implementation
class DigitalClock implements ClockInterface2 {
  constructor(h: number, m: number) {
    console.log(`Digital: ${h}:${m}`);
  }
  
  tick() {
    console.log("beep beep");
  }
}

// Analog clock implementation
class AnalogClock implements ClockInterface2 {
  constructor(h: number, m: number) {
    console.log(`Analog: ${h}:${m}`);
  }
  
  tick() {
    console.log("tick tock");
  }
}

// Create instances using factory
let digital = createClock(DigitalClock, 12, 17);
let analog = createClock(AnalogClock, 7, 32);

digital.tick();  // "beep beep"
analog.tick();   // "tick tock"

// ============================================
// OPTIONAL AND READONLY IN INTERFACES
// ============================================

interface Config {
  readonly apiUrl: string;      // Must be set at initialization, cannot change
  timeout?: number;             // Optional - may or may not be provided
  retryAttempts?: number;       // Optional
  headers: {                    // Nested object type
    [key: string]: string;      // Index signature for flexible headers
  };
}

const config: Config = {
  apiUrl: "https://api.example.com",
  timeout: 5000,
  headers: {
    "Content-Type": "application/json",
    "Authorization": "Bearer token123"
  }
};

// config.apiUrl = "https://new-api.com";  // Error - readonly property
config.timeout = 10000;  // OK - not readonly

// ============================================
// REAL-WORLD EXAMPLE: API RESPONSE INTERFACE
// ============================================

// Define structure for API responses
interface ApiResponse<T> {
  success: boolean;
  data: T;                      // Generic type for flexible data
  message?: string;             // Optional message
  errors?: string[];            // Optional array of errors
  metadata: {
    timestamp: number;
    requestId: string;
    version: string;
  };
}

// User data structure
interface UserData {
  id: number;
  username: string;
  email: string;
  roles: string[];
}

// Type-safe API response
const userResponse: ApiResponse<UserData> = {
  success: true,
  data: {
    id: 1,
    username: "johndoe",
    email: "john@example.com",
    roles: ["user", "admin"]
  },
  metadata: {
    timestamp: Date.now(),
    requestId: "req-123-456",
    version: "1.0.0"
  }
};

// Function using the interface
function handleApiResponse<T>(response: ApiResponse<T>): T | null {
  if (response.success) {
    console.log(`Success: ${response.message || 'Operation completed'}`);
    return response.data;  // Return typed data
  } else {
    console.error(`Errors: ${response.errors?.join(', ')}`);
    return null;
  }
}

const userData = handleApiResponse(userResponse);
if (userData) {
  console.log(`User: ${userData.username}`);  // Type-safe access
}
```

### Advanced Type Features

```typescript
// ============================================
// TYPE GUARDS
// ============================================

// Type guards - runtime checks that narrow down types

// typeof type guard - works with primitive types
function processValue(value: string | number) {
  if (typeof value === "string") {
    // TypeScript knows value is string here
    return value.toUpperCase();
  } else {
    // TypeScript knows value is number here
    return value.toFixed(2);
  }
}

// instanceof type guard - works with classes
class Bird {
  fly() {
    console.log("Flying...");
  }
  layEggs() {
    console.log("Laying eggs...");
  }
}

class Fish {
  swim() {
    console.log("Swimming...");
  }
  layEggs() {
    console.log("Laying eggs...");
  }
}

function move(pet: Bird | Fish) {
  if (pet instanceof Bird) {
    pet.fly();  // TypeScript knows pet is Bird
  } else {
    pet.swim(); // TypeScript knows pet is Fish
  }
  pet.layEggs(); // Available on both types
}

// User-defined type guard - custom type checking logic
interface Car {
  drive(): void;
}

interface Boat {
  sail(): void;
}

// Type predicate function - return type is "parameterName is Type"
function isCar(vehicle: Car | Boat): vehicle is Car {
  // Check if vehicle has drive method
  return (vehicle as Car).drive !== undefined;
}

function operate(vehicle: Car | Boat) {
  if (isCar(vehicle)) {
    vehicle.drive();  // TypeScript knows it's Car
  } else {
    vehicle.sail();   // TypeScript knows it's Boat
  }
}

// in operator type guard - checks if property exists
type Admin = {
  name: string;
  privileges: string[];
};

type NormalUser = {
  name: string;
};

function greetUser(user: Admin | NormalUser) {
  console.log(`Hello ${user.name}`);
  
  // Check if privileges property exists
  if ("privileges" in user) {
    console.log(`Privileges: ${user.privileges.join(", ")}`);
  }
}

// ============================================
// DISCRIMINATED UNIONS
// ============================================

// Pattern using literal types to discriminate union types

// Each interface has a common "kind" property with literal type
interface Circle {
  kind: "circle";     // Discriminant property
  radius: number;
}

interface Square {
  kind: "square";     // Discriminant property
  sideLength: number;
}

interface Rectangle {
  kind: "rectangle";  // Discriminant property
  width: number;
  height: number;
}

// Union of all shape types
type Shape = Circle | Square | Rectangle;

// Calculate area using discriminated union
function getArea(shape: Shape): number {
  // TypeScript uses "kind" to narrow the type
  switch (shape.kind) {
    case "circle":
      // shape is Circle here
      return Math.PI * shape.radius ** 2;
    
    case "square":
      // shape is Square here
      return shape.sideLength ** 2;
    
    case "rectangle":
      // shape is Rectangle here
      return shape.width * shape.height;
    
    default:
      // Exhaustiveness checking - ensures all cases handled
      const _exhaustiveCheck: never = shape;
      return _exhaustiveCheck;
  }
}

const circle: Circle = { kind: "circle", radius: 5 };
const square: Square = { kind: "square", sideLength: 10 };

console.log(getArea(circle));   // 78.54
console.log(getArea(square));   // 100

// ============================================
// REAL-WORLD EXAMPLE: API ACTION TYPES
// ============================================

// Common pattern in Redux/state management

interface FetchStartAction {
  type: "FETCH_START";
  payload: {
    endpoint: string;
  };
}

interface FetchSuccessAction {
  type: "FETCH_SUCCESS";
  payload: {
    data: any;
    timestamp: number;
  };
}

interface FetchErrorAction {
  type: "FETCH_ERROR";
  payload: {
    error: string;
    code: number;
  };
}

// Union of all action types
type FetchAction = FetchStartAction | FetchSuccessAction | FetchErrorAction;

// Reducer function with type narrowing
function fetchReducer(state: any, action: FetchAction) {
  switch (action.type) {
    case "FETCH_START":
      // action.payload.endpoint is available
      console.log(`Starting fetch: ${action.payload.endpoint}`);
      return { ...state, loading: true };
    
    case "FETCH_SUCCESS":
      // action.payload.data and timestamp are available
      console.log(`Fetch successful at ${action.payload.timestamp}`);
      return { ...state, loading: false, data: action.payload.data };
    
    case "FETCH_ERROR":
      // action.payload.error and code are available
      console.log(`Error ${action.payload.code}: ${action.payload.error}`);
      return { ...state, loading: false, error: action.payload.error };
    
    default:
      return state;
  }
}

// ============================================
// NULLABLE TYPES
// ============================================

// With strictNullChecks enabled, null and undefined are separate types

let stringValue: string = "hello";
// stringValue = null;      // Error - null is not assignable to string

let nullableString: string | null = "hello";
nullableString = null;      // OK - explicitly allows null

let undefinableString: string | undefined = "hello";
undefinableString = undefined;  // OK - explicitly allows undefined

let optionalValue: string | null | undefined;
optionalValue = "hello";
optionalValue = null;
optionalValue = undefined;  // All are valid

// Optional chaining - safely access nested properties
interface UserProfile {
  name: string;
  address?: {              // Optional property
    street?: string;
    city?: string;
    country: string;
  };
}

const user1: UserProfile = {
  name: "John",
  address: {
    city: "New York",
    country: "USA"
  }
};

const user2: UserProfile = {
  name: "Jane"
  // No address
};

// Safe property access with optional chaining
console.log(user1.address?.city);        // "New York"
console.log(user2.address?.city);        // undefined (no error)
console.log(user1.address?.street);      // undefined (property doesn't exist)

// Non-null assertion operator - tell TypeScript value is not null
function processUser(user: UserProfile) {
  // We know address exists in this context
  const country = user.address!.country;  // ! asserts address is not null/undefined
  console.log(country);
}

// Nullish coalescing operator - provide default for null/undefined
const displayCity = user2.address?.city ?? "Unknown City";  // "Unknown City"
const displayName = user2.name ?? "Anonymous";              // "John" (name exists)

// ============================================
// TYPE NARROWING WITH TRUTHINESS
// ============================================

function printLength(str: string | null | undefined) {
  // Truthiness check narrows type
  if (str) {
    // str is string here (null and undefined are falsy)
    console.log(str.length);
  } else {
    console.log("No string provided");
  }
}

// Narrowing with equality checks
function compare(x: string | number, y: string | boolean) {
  if (x === y) {
    // TypeScript knows both must be string (only common type)
    x.toUpperCase();
    y.toUpperCase();
  }
}

// ============================================
// ASSERTION FUNCTIONS
// ============================================

// Functions that assert a condition and narrow types

function assert(condition: any, msg?: string): asserts condition {
  if (!condition) {
    throw new Error(msg || "Assertion failed");
  }
}

function processInput(input: string | null) {
  assert(input !== null, "Input cannot be null");
  // After assert, TypeScript knows input is string
  console.log(input.toUpperCase());
}

// Type assertion function
function assertIsString(value: any): asserts value is string {
  if (typeof value !== "string") {
    throw new Error("Value must be a string");
  }
}

function processValue2(value: unknown) {
  assertIsString(value);
  // After assertion, value is known to be string
  console.log(value.toUpperCase());
}
```

---

## Generics and Interfaces in TypeScript

Generics allow you to create reusable components that work with multiple types while maintaining type safety.

```typescript
// ============================================
// BASIC GENERICS
// ============================================

// Generic function - works with any type
function identity<T>(arg: T): T {
  return arg;  // Return value has same type as input
}

// Usage - TypeScript infers the type
let output1 = identity<string>("hello");  // Explicit type
let output2 = identity(42);                // Inferred as number
let output3 = identity({ name: "John" }); // Inferred as object

// Generic function with array
function getFirstElement<T>(arr: T[]): T | undefined {
  return arr[0];  // Returns first element or undefined
}

const firstNumber = getFirstElement([1, 2, 3]);       // number | undefined
const firstName = getFirstElement(["a", "b", "c"]);   // string | undefined

// Multiple type parameters
function pair<T, U>(first: T, second: U): [T, U] {
  return [first, second];  // Return tuple of both types
}

const numberStringPair = pair(1, "hello");     // [number, string]
const booleanArrayPair = pair(true, [1, 2]);   // [boolean, number[]]

// ============================================
// GENERIC INTERFACES
// ============================================

// Generic interface definition
interface Box<T> {
  value: T;                    // Property of generic type
  getValue(): T;               // Method returning generic type
  setValue(value: T): void;    // Method accepting generic type
}

// Implement generic interface with specific type
const numberBox: Box<number> = {
  value: 42,
  getValue() {
    return this.value;
  },
  setValue(value: number) {
    this.value = value;
  }
};

const stringBox: Box<string> = {
  value: "hello",
  getValue() {
    return this.value;
  },
  setValue(value: string) {
    this.value = value;
  }
};

// Generic interface with multiple type parameters
interface Pair<K, V> {
  key: K;
  value: V;
}

const userIdName: Pair<number, string> = {
  key: 1,
  value: "Alice"
};

const configPair: Pair<string, boolean> = {
  key: "isDarkMode",
  value: true
};

// ============================================
// GENERIC CLASSES
// ============================================

// Generic class - type parameter used throughout the class
class GenericNumber<T> {
  zeroValue: T;                       // Property of type T
  add: (x: T, y: T) => T;            // Function property
  
  constructor(zero: T, addFn: (x: T, y: T) => T) {
    this.zeroValue = zero;
    this.add = addFn;
  }
}

// Create instance with number type
const numberAdder = new GenericNumber<number>(
  0,                           // zeroValue
  (x, y) => x + y             // add function
);

console.log(numberAdder.add(5, 10));  // 15

// Create instance with string type
const stringAdder = new GenericNumber<string>(
  "",                          // zeroValue
  (x, y) => x + y             // concatenation for strings
);

console.log(stringAdder.add("Hello ", "World"));  // "Hello World"

// Generic Stack implementation
class Stack<T> {
  private items: T[] = [];     // Internal storage

  // Add element to top of stack
  push(item: T): void {
    this.items.push(item);
  }

  // Remove and return top element
  pop(): T | undefined {
    return this.items.pop();
  }

  // View top element without removing
  peek(): T | undefined {
    return this.items[this.items.length - 1];
  }

  // Check if stack is empty
  isEmpty(): boolean {
    return this.items.length === 0;
  }

  // Get stack size
  size(): number {
    return this.items.length;
  }

  // Clear all elements
  clear(): void {
    this.items = [];
  }
}

// Usage
const numberStack = new Stack<number>();
numberStack.push(1);
numberStack.push(2);
numberStack.push(3);
console.log(numberStack.pop());   // 3
console.log(numberStack.peek());  // 2

const stringStack = new Stack<string>();
stringStack.push("first");
stringStack.push("second");
console.log(stringStack.size());  // 2

// ============================================
// GENERIC CONSTRAINTS
// ============================================

// Constraint - generic type must have certain properties
interface Lengthwise {
  length: number;
}

// T must have length property
function logLength<T extends Lengthwise>(arg: T): T {
  console.log(arg.length);  // OK - we know length exists
  return arg;
}

logLength("hello");              // OK - string has length
logLength([1, 2, 3]);           // OK - array has length
logLength({ length: 10 });      // OK - object has length
// logLength(42);               // Error - number doesn't have length

// Using type parameter in constraints
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];  // K is guaranteed to be a key of T
}

const person = {
  name: "Alice",
  age: 30,
  city: "New York"
};

const personName = getProperty(person, "name");  // "Alice" - type is string
const personAge = getProperty(person, "age");    // 30 - type is number
// const invalid = getProperty(person, "invalid"); // Error - "invalid" is not a key

// Multiple constraints with intersection
interface Named {
  name: string;
}

interface Aged {
  age: number;
}

// T must satisfy both Named and Aged
function printPersonInfo<T extends Named & Aged>(person: T): void {
  console.log(`${person.name} is ${person.age} years old`);
}

printPersonInfo({ name: "Bob", age: 25 });  // OK

// ============================================
// GENERIC FACTORY PATTERN
// ============================================

// Generic factory function using constructor type
function create<T>(Constructor: new () => T): T {
  return new Constructor();  // Create instance of type T
}

class Person {
  name: string = "Unknown";
  greet() {
    console.log(`Hello, I'm ${this.name}`);
  }
}

class Product {
  title: string = "Untitled";
  price: number = 0;
}

const person = create(Person);    // Type is Person
const product = create(Product);  // Type is Product

person.greet();  // "Hello, I'm Unknown"

// Factory with parameters
function createWithParams<T>(
  Constructor: new (...args: any[]) => T,
  ...args: any[]
): T {
  return new Constructor(...args);
}

class Car {
  constructor(public brand: string, public year: number) {}
}

const myCar = createWithParams(Car, "Toyota", 2023);
console.log(myCar.brand);  // "Toyota"

// ============================================
// GENERIC UTILITY FUNCTIONS
// ============================================

// Merge two objects of different types
function merge<T, U>(obj1: T, obj2: U): T & U {
  return { ...obj1, ...obj2 };  // Combine properties
}

const merged = merge(
  { name: "John" },
  { age: 30, city: "NYC" }
);
// merged has type: { name: string } & { age: number; city: string }
console.log(merged.name, merged.age, merged.city);

// Filter array by predicate
function filter<T>(array: T[], predicate: (item: T) => boolean): T[] {
  const result: T[] = [];
  for (const item of array) {
    if (predicate(item)) {
      result.push(item);
    }
  }
  return result;
}

const numbers = [1, 2, 3, 4, 5, 6];
const evenNumbers = filter(numbers, (n) => n % 2 === 0);  // [2, 4, 6]

const words = ["hello", "world", "typescript"];
const longWords = filter(words, (w) => w.length > 5);     // ["typescript"]

// Map function
function map<T, U>(array: T[], transform: (item: T) => U): U[] {
  const result: U[] = [];
  for (const item of array) {
    result.push(transform(item));
  }
  return result;
}

const doubled = map([1, 2, 3], (n) => n * 2);           // [2, 4, 6]
const lengths = map(["a", "bb", "ccc"], (s) => s.length); // [1, 2, 3]

// ============================================
// GENERIC PROMISE HANDLING
// ============================================

// Async function with generic return type
async function fetchData<T>(url: string): Promise<T> {
  const response = await fetch(url);
  const data: T = await response.json();  // Parse JSON as type T
  return data;
}

// Usage with specific type
interface Todo {
  id: number;
  title: string;
  completed: boolean;
}

async function getTodo(id: number): Promise<Todo> {
  const todo = await fetchData<Todo>(
    `https://jsonplaceholder.typicode.com/todos/${id}`
  );
  return todo;
}

// Generic retry function
async function retry<T>(
  fn: () => Promise<T>,
  maxAttempts: number = 3
): Promise<T> {
  let lastError: Error;
  
  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    try {
      return await fn();  // Try to execute function
    } catch (error) {
      lastError = error as Error;
      console.log(`Attempt ${attempt} failed, retrying...`);
    }
  }
  
  throw lastError!;  // All attempts failed
}

// Usage
async function unreliableOperation(): Promise<string> {
  if (Math.random() > 0.5) {
    throw new Error("Random failure");
  }
  return "Success!";
}

retry(unreliableOperation, 5)
  .then(result => console.log(result))
  .catch(error => console.error("All attempts failed:", error));

// ============================================
// REAL-WORLD EXAMPLE: REPOSITORY PATTERN
// ============================================

// Generic repository interface
interface Repository<T> {
  findById(id: number): Promise<T | null>;
  findAll(): Promise<T[]>;
  create(item: Omit<T, 'id'>): Promise<T>;
  update(id: number, item: Partial<T>): Promise<T | null>;
  delete(id: number): Promise<boolean>;
}

// User entity
interface User {
  id: number;
  username: string;
  email: string;
  createdAt: Date;
}

// Implement repository for User
class UserRepository implements Repository<User> {
  private users: User[] = [];
  private currentId = 1;

  async findById(id: number): Promise<User | null> {
    const user = this.users.find(u => u.id === id);
    return user || null;
  }

  async findAll(): Promise<User[]> {
    return [...this.users];  // Return copy
  }

  async create(userData: Omit<User, 'id'>): Promise<User> {
    const newUser: User = {
      ...userData,
      id: this.currentId++,
      createdAt: new Date()
    };
    this.users.push(newUser);
    return newUser;
  }

  async update(id: number, userData: Partial<User>): Promise<User | null> {
    const index = this.users.findIndex(u => u.id === id);
    if (index === -1) return null;
    
    this.users[index] = { ...this.users[index], ...userData };
    return this.users[index];
  }

  async delete(id: number): Promise<boolean> {
    const index = this.users.findIndex(u => u.id === id);
    if (index === -1) return false;
    
    this.users.splice(index, 1);
    return true;
  }
}

// Usage
const userRepo = new UserRepository();

async function demonstrateRepo() {
  // Create users
  const user1 = await userRepo.create({
    username: "alice",
    email: "alice@example.com",
    createdAt: new Date()
  });
  
  const user2 = await userRepo.create({
    username: "bob",
    email: "bob@example.com",
    createdAt: new Date()
  });
  
  // Find all
  const allUsers = await userRepo.findAll();
  console.log("All users:", allUsers);
  
  // Update
  await userRepo.update(user1.id, { email: "newalice@example.com" });
  
  // Find by ID
  const foundUser = await userRepo.findById(user1.id);
  console.log("Found user:", foundUser);
  
  // Delete
  await userRepo.delete(user2.id);
}
```

### Generic Constraints and Advanced Patterns

```typescript
// ============================================
// DEFAULT GENERIC TYPES
// ============================================

// Generic with default type
interface ApiResponse<T = any> {
  data: T;
  status: number;
  message: string;
}

// Use without specifying type - defaults to 'any'
const response1: ApiResponse = {
  data: { anything: "goes here" },
  status: 200,
  message: "OK"
};

// Use with specific type
const response2: ApiResponse<User[]> = {
  data: [{ id: 1, username: "alice", email: "alice@example.com", createdAt: new Date() }],
  status: 200,
  message: "OK"
};

// ============================================
// CONDITIONAL TYPES
// ============================================

// Conditional type - type depends on condition
type IsString<T> = T extends string ? true : false;

type Test1 = IsString<string>;  // true
type Test2 = IsString<number>;  // false

// Extract return type from function
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

function getUser(): User {
  return { id: 1, username: "test", email: "test@example.com", createdAt: new Date() };
}

type UserReturnType = ReturnType<typeof getUser>;  // User

// Exclude types from union
type Exclude<T, U> = T extends U ? never : T;

type NumberOrString = number | string;
type OnlyNumber = Exclude<NumberOrString, string>;  // number

// Extract types from union
type Extract<T, U> = T extends U ? T : never;

type OnlyString = Extract<NumberOrString, string>;  // string

// ============================================
// MAPPED TYPES WITH GENERICS
// ============================================

// Make all properties optional
type Partial<T> = {
  [P in keyof T]?: T[P];
};

interface Todo {
  title: string;
  description: string;
  completed: boolean;
}

type PartialTodo = Partial<Todo>;
// {
//   title?: string;
//   description?: string;
//   completed?: boolean;
// }

// Make all properties required
type Required<T> = {
  [P in keyof T]-?: T[P];  // -? removes optional modifier
};

type RequiredTodo = Required<PartialTodo>;  // Back to Todo

// Make all properties readonly
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};

type ReadonlyTodo = Readonly<Todo>;
// {
//   readonly title: string;
//   readonly description: string;
//   readonly completed: boolean;
// }

// Pick specific properties
type Pick<T, K extends keyof T> = {
  [P in K]: T[P];
};

type TodoPreview = Pick<Todo, "title" | "completed">;
// {
//   title: string;
//   completed: boolean;
// }

// Omit specific properties
type Omit<T, K extends keyof T> = Pick<T, Exclude<keyof T, K>>;

type TodoWithoutDescription = Omit<Todo, "description">;
// {
//   title: string;
//   completed: boolean;
// }

// ============================================
// RECURSIVE GENERIC TYPES
// ============================================

// Recursive type for nested objects
type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends object ? DeepPartial<T[P]> : T[P];
};

interface NestedConfig {
  database: {
    host: string;
    port: number;
    credentials: {
      username: string;
      password: string;
    };
  };
  cache: {
    enabled: boolean;
    ttl: number;
  };
}

type PartialConfig = DeepPartial<NestedConfig>;
// All properties at all levels are optional

const config: PartialConfig = {
  database: {
    credentials: {
      username: "admin"
      // password is optional
    }
    // host and port are optional
  }
  // cache is optional
};

// ============================================
// GENERIC BUILDER PATTERN
// ============================================

class QueryBuilder<T> {
  private filters: Array<(item: T) => boolean> = [];
  private sortFn?: (a: T, b: T) => number;
  private limitValue?: number;

  // Add filter condition
  where(predicate: (item: T) => boolean): this {
    this.filters.push(predicate);
    return this;  // Return this for chaining
  }

  // Set sort function
  orderBy(compareFn: (a: T, b: T) => number): this {
    this.sortFn = compareFn;
    return this;
  }

  // Set limit
  limit(count: number): this {
    this.limitValue = count;
    return this;
  }

  // Execute query
  execute(data: T[]): T[] {
    let result = [...data];

    // Apply filters
    for (const filter of this.filters) {
      result = result.filter(filter);
    }

    // Apply sorting
    if (this.sortFn) {
      result.sort(this.sortFn);
    }

    // Apply limit
    if (this.limitValue !== undefined) {
      result = result.slice(0, this.limitValue);
    }

    return result;
  }
}

// Usage
interface Product {
  id: number;
  name: string;
  price: number;
  category: string;
}

const products: Product[] = [
  { id: 1, name: "Laptop", price: 1200, category: "Electronics" },
  { id: 2, name: "Phone", price: 800, category: "Electronics" },
  { id: 3, name: "Desk", price: 300, category: "Furniture" },
  { id: 4, name: "Chair", price: 150, category: "Furniture" },
];

const query = new QueryBuilder<Product>()
  .where(p => p.category === "Electronics")  // Filter by category
  .where(p => p.price < 1000)                // Filter by price
  .orderBy((a, b) => b.price - a.price)      // Sort by price descending
  .limit(1);                                  // Take top 1

const results = query.execute(products);
console.log(results);  // [{ id: 2, name: "Phone", price: 800, category: "Electronics" }]
```

---

## Key Patterns in TypeScript

### Utility Types and Mapped Types

TypeScript provides powerful built-in utility types for type transformations:

```typescript
// ============================================
// BUILT-IN UTILITY TYPES
// ============================================

interface User {
  id: number;
  name: string;
  email: string;
  age: number;
  address: string;
}

// Partial<T> - makes all properties optional
type PartialUser = Partial<User>;
const updateUser: PartialUser = {
  name: "John"  // Only updating name, rest are optional
};

// Required<T> - makes all properties required
interface OptionalUser {
  id?: number;
  name?: string;
  email?: string;
}

type RequiredUser = Required<OptionalUser>;
const user: RequiredUser = {
  id: 1,
  name: "Alice",
  email: "alice@example.com"  // All must be provided
};

// Readonly<T> - makes all properties readonly
type ReadonlyUser = Readonly<User>;
const immutableUser: ReadonlyUser = {
  id: 1,
  name: "Bob",
  email: "bob@example.com",
  age: 30,
  address: "123 Main St"
};
// immutableUser.name = "Bobby";  // Error - readonly property

// Pick<T, K> - creates type with subset of properties
type UserPreview = Pick<User, "id" | "name">;
const preview: UserPreview = {
  id: 1,
  name: "Alice"
  // email, age, address not needed
};

// Omit<T, K> - creates type excluding specific properties
type UserWithoutId = Omit<User, "id">;
const newUser: UserWithoutId = {
  name: "Charlie",
  email: "charlie@example.com",
  age: 25,
  address: "456 Oak Ave"
  // id is omitted
};

// Record<K, T> - creates type with specific keys and value type
type UserRoles = "admin" | "user" | "guest";
type RolePermissions = Record<UserRoles, string[]>;

const permissions: RolePermissions = {
  admin: ["read", "write", "delete"],
  user: ["read", "write"],
  guest: ["read"]
};

// ============================================
// EXTRACT AND EXCLUDE
// ============================================

type AllTypes = string | number | boolean | null | undefined;

// Extract<T, U> - extracts types assignable to U
type StringOrNumber = Extract<AllTypes, string | number>;  // string | number

// Exclude<T, U> - excludes types assignable to U
type NoNullish = Exclude<AllTypes, null | undefined>;  // string | number | boolean

// ============================================
// NON-NULLABLE
// ============================================

type MaybeString = string | null | undefined;

// NonNullable<T> - removes null and undefined
type DefinitelyString = NonNullable<MaybeString>;  // string

// ============================================
// RETURN TYPE AND PARAMETERS
// ============================================

function getUser(id: number): User {
  return {
    id,
    name: "User" + id,
    email: `user${id}@example.com`,
    age: 30,
    address: "123 Main St"
  };
}

// ReturnType<T> - extracts return type of function
type GetUserReturn = ReturnType<typeof getUser>;  // User

// Parameters<T> - extracts parameter types as tuple
type GetUserParams = Parameters<typeof getUser>;  // [number]

function updateProfile(userId: number, data: Partial<User>): boolean {
  return true;
}

type UpdateProfileParams = Parameters<typeof updateProfile>;  // [number, Partial<User>]

// ============================================
// INSTANCE TYPE AND CONSTRUCTOR PARAMETERS
// ============================================

class Database {
  constructor(public connectionString: string, public timeout: number = 5000) {}
  
  connect(): void {
    console.log("Connecting...");
  }
}

// InstanceType<T> - gets instance type of constructor
type DatabaseInstance = InstanceType<typeof Database>;  // Database

// ConstructorParameters<T> - gets constructor parameter types
type DatabaseParams = ConstructorParameters<typeof Database>;  // [string, number?]

// ============================================
// CUSTOM MAPPED TYPES
// ============================================

// Make specific properties optional
type OptionalKeys<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>;

type UserWithOptionalEmail = OptionalKeys<User, "email">;
const user1: UserWithOptionalEmail = {
  id: 1,
  name: "Alice",
  age: 30,
  address: "123 Main St"
  // email is now optional
};

// Make all properties nullable
type Nullable<T> = {
  [P in keyof T]: T[P] | null;
};

type NullableUser = Nullable<User>;
const nullUser: NullableUser = {
  id: 1,
  name: null,
  email: null,
  age: null,
  address: null
};

// Deep readonly - makes nested objects readonly too
type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends object ? DeepReadonly<T[P]> : T[P];
};

interface Company {
  name: string;
  address: {
    street: string;
    city: string;
    country: string;
  };
  employees: {
    name: string;
    role: string;
  }[];
}

type ReadonlyCompany = DeepReadonly<Company>;
const company: ReadonlyCompany = {
  name: "Tech Corp",
  address: {
    street: "123 Tech St",
    city: "San Francisco",
    country: "USA"
  },
  employees: [
    { name: "Alice", role: "Engineer" }
  ]
};
// company.name = "New Name";  // Error
// company.address.city = "NYC";  // Error
// company.employees[0].name = "Bob";  // Error

// ============================================
// CONDITIONAL MAPPED TYPES
// ============================================

// Extract all function property names
type FunctionPropertyNames<T> = {
  [K in keyof T]: T[K] extends Function ? K : never;
}[keyof T];

// Extract all non-function property names
type NonFunctionPropertyNames<T> = {
  [K in keyof T]: T[K] extends Function ? never : K;
}[keyof T];

interface Example {
  name: string;
  age: number;
  greet(): void;
  calculate(x: number): number;
}

type ExampleFunctions = FunctionPropertyNames<Example>;  // "greet" | "calculate"
type ExampleData = NonFunctionPropertyNames<Example>;    // "name" | "age"

// Create type with only function properties
type FunctionsOnly<T> = Pick<T, FunctionPropertyNames<T>>;
type ExampleMethods = FunctionsOnly<Example>;
// {
//   greet(): void;
//   calculate(x: number): number;
// }

// Create type with only data properties
type DataOnly<T> = Pick<T, NonFunctionPropertyNames<T>>;
type ExampleDataProps = DataOnly<Example>;
// {
//   name: string;
//   age: number;
// }

// ============================================
// PROMISE UNWRAPPING
// ============================================

// Extract type from Promise
type Awaited<T> = T extends Promise<infer U> ? U : T;

type PromisedNumber = Promise<number>;
type UnwrappedNumber = Awaited<PromisedNumber>;  // number

type NestedPromise = Promise<Promise<string>>;
type UnwrappedString = Awaited<Awaited<NestedPromise>>;  // string

// Recursive unwrapping
type DeepAwaited<T> = T extends Promise<infer U> ? DeepAwaited<U> : T;

type DeeplyNested = Promise<Promise<Promise<boolean>>>;
type FullyUnwrapped = DeepAwaited<DeeplyNested>;  // boolean

// ============================================
// REAL-WORLD EXAMPLE: FORM VALIDATION TYPES
// ============================================

// Base form data
interface FormData {
  username: string;
  email: string;
  age: number;
  acceptTerms: boolean;
}

// Validation error type - each field can have error message
type ValidationErrors<T> = {
  [K in keyof T]?: string;
};

// Form state includes data, errors, and status flags
type FormState<T> = {
  data: T;
  errors: ValidationErrors<T>;
  touched: Partial<Record<keyof T, boolean>>;
  isSubmitting: boolean;
  isValid: boolean;
};

// Example form state
const formState: FormState<FormData> = {
  data: {
    username: "",
    email: "",
    age: 0,
    acceptTerms: false
  },
  errors: {
    username: "Username is required",
    email: "Invalid email format"
  },
  touched: {
    username: true,
    email: true
  },
  isSubmitting: false,
  isValid: false
};

// Field validator function type
type Validator<T> = (value: T) => string | undefined;

// Validators for each field
type FormValidators<T> = {
  [K in keyof T]?: Validator<T[K]>;
};

const validators: FormValidators<FormData> = {
  username: (value) => {
    if (!value) return "Username is required";
    if (value.length < 3) return "Username must be at least 3 characters";
    return undefined;
  },
  email: (value) => {
    if (!value) return "Email is required";
    if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value)) return "Invalid email format";
    return undefined;
  },
  age: (value) => {
    if (value < 18) return "Must be at least 18 years old";
    if (value > 120) return "Invalid age";
    return undefined;
  }
};

// ============================================
// GETTERS TYPE - CONVERT PROPERTIES TO GETTERS
// ============================================

type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

interface Person {
  name: string;
  age: number;
}

type PersonGetters = Getters<Person>;
// {
//   getName(): string;
//   getAge(): number;
// }

// Implementation
const personWithGetters: PersonGetters = {
  getName() {
    return "John";
  },
  getAge() {
    return 30;
  }
};

// ============================================
// MUTABLE - REMOVE READONLY
// ============================================

type Mutable<T> = {
  -readonly [P in keyof T]: T[P];
};

interface ReadonlyPerson {
  readonly name: string;
  readonly age: number;
}

type MutablePerson = Mutable<ReadonlyPerson>;
// {
//   name: string;
//   age: number;
// }

const mutablePerson: MutablePerson = {
  name: "Alice",
  age: 30
};
mutablePerson.name = "Bob";  // OK - no longer readonly
```

### Template Literal Types

```typescript
// ============================================
// TEMPLATE LITERAL TYPES
// ============================================

// Create string literal types using template syntax
type EventName = "click" | "scroll" | "mousemove";
type HandlerName = `on${Capitalize<EventName>}`;
// "onClick" | "onScroll" | "onMousemove"

// Combining multiple unions
type HTTPMethod = "GET" | "POST" | "PUT" | "DELETE";
type APIVersion = "v1" | "v2";
type Endpoint = `/${APIVersion}/users`;
// "/v1/users" | "/v2/users"

// Complex combinations
type Route = `/${APIVersion}/${HTTPMethod}/users`;
// "/v1/GET/users" | "/v1/POST/users" | ... (8 combinations)

// ============================================
// STRING MANIPULATION TYPES
// ============================================

// Uppercase - convert to uppercase
type UppercaseGreeting = Uppercase<"hello">;  // "HELLO"

// Lowercase - convert to lowercase
type LowercaseGreeting = Lowercase<"WORLD">;  // "world"

// Capitalize - capitalize first letter
type CapitalizedName = Capitalize<"alice">;  // "Alice"

// Uncapitalize - lowercase first letter
type UncapitalizedName = Uncapitalize<"Bob">;  // "bob"

// ============================================
// PRACTICAL EXAMPLES
// ============================================

// CSS property names
type CSSProperty = "color" | "background" | "border";
type CSSVariable = `--${CSSProperty}`;
// "--color" | "--background" | "--border"

const primaryColor: CSSVariable = "--color";

// Event handler types
type DOMEvent = "click" | "dblclick" | "mouseenter" | "mouseleave";
type EventHandler<E extends DOMEvent> = `on${Capitalize<E>}`;

type ClickHandler = EventHandler<"click">;  // "onClick"

// Getter/Setter pattern
type PropName = "name" | "age" | "email";
type Getter<P extends string> = `get${Capitalize<P>}`;
type Setter<P extends string> = `set${Capitalize<P>}`;

type NameGetter = Getter<"name">;  // "getName"
type NameSetter = Setter<"name">;  // "setName"

// All getters and setters for properties
type AllGetters = Getter<PropName>;
// "getName" | "getAge" | "getEmail"

type AllSetters = Setter<PropName>;
// "setName" | "setAge" | "setEmail"

// ============================================
// EXTRACTING FROM TEMPLATE LITERALS
// ============================================

// Extract parts from template literal types
type ExtractRouteParams<T extends string> = 
  T extends `${string}:${infer Param}/${infer Rest}`
    ? Param | ExtractRouteParams<`/${Rest}`>
    : T extends `${string}:${infer Param}`
    ? Param
    : never;

type Route1 = "/users/:userId/posts/:postId";
type RouteParams = ExtractRouteParams<Route1>;  // "userId" | "postId"

// ============================================
// REAL-WORLD: API ENDPOINT BUILDER
// ============================================

type Resource = "users" | "posts" | "comments";
type Action = "list" | "get" | "create" | "update" | "delete";
type APIRoute = `/${Resource}/${Action}`;

// Creates: "/users/list", "/users/get", "/posts/create", etc.

interface APIEndpoints {
  [K: APIRoute]: {
    method: HTTPMethod;
    handler: (req: any, res: any) => void;
  };
}
```

---

## Object-Oriented Programming with TypeScript

### Classes and Inheritance

```typescript
// ============================================
// BASIC CLASS DEFINITION
// ============================================

class Person {
  // Properties
  name: string;
  age: number;
  
  // Constructor - initializes new instances
  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }
  
  // Method
  greet(): void {
    console.log(`Hello, my name is ${this.name} and I'm ${this.age} years old.`);
  }
  
  // Method with return value
  getInfo(): string {
    return `${this.name} (${this.age})`;
  }
}

// Create instance
const person1 = new Person("Alice", 30);
person1.greet();  // "Hello, my name is Alice and I'm 30 years old."

// ============================================
// ACCESS MODIFIERS
// ============================================

class BankAccount {
  // Public - accessible from anywhere (default)
  public accountNumber: string;
  
  // Private - only accessible within the class
  private balance: number;
  
  // Protected - accessible within class and subclasses
  protected accountType: string;
  
  constructor(accountNumber: string, initialBalance: number) {
    this.accountNumber = accountNumber;
    this.balance = initialBalance;
    this.accountType = "Standard";
  }
  
  // Public method to access private balance
  getBalance(): number {
    return this.balance;  // OK - within class
  }
  
  // Public method to modify private balance
  deposit(amount: number): void {
    if (amount > 0) {
      this.balance += amount;
      console.log(`Deposited $${amount}. New balance: $${this.balance}`);
    }
  }
  
  withdraw(amount: number): boolean {
    if (amount > 0 && amount <= this.balance) {
      this.balance -= amount;
      console.log(`Withdrew $${amount}. New balance: $${this.balance}`);
      return true;
    }
    console.log("Insufficient funds");
    return false;
  }
}

const account = new BankAccount("ACC123", 1000);
console.log(account.accountNumber);  // OK - public
console.log(account.getBalance());   // OK - public method
// console.log(account.balance);     // Error - private property
account.deposit(500);
account.withdraw(200);

// ============================================
// PARAMETER PROPERTIES (SHORTHAND)
// ============================================

// Shorthand: declare and initialize properties in constructor
class Product {
  constructor(
    public id: number,           // Automatically creates public property
    public name: string,
    private price: number,       // Automatically creates private property
    protected category: string   // Automatically creates protected property
  ) {
    // Properties are automatically assigned
  }
  
  getPrice(): number {
    return this.price;
  }
  
  setPrice(newPrice: number): void {
    if (newPrice > 0) {
      this.price = newPrice;
    }
  }
}

const product = new Product(1, "Laptop", 1200, "Electronics");
console.log(product.name);   // "Laptop"
console.log(product.getPrice());  // 1200

// ============================================
// READONLY PROPERTIES
// ============================================

class Employee {
  // Readonly - can only be initialized, not modified later
  readonly employeeId: number;
  readonly hireDate: Date;
  name: string;
  
  constructor(id: number, name: string) {
    this.employeeId = id;  // OK - initialization in constructor
    this.hireDate = new Date();
    this.name = name;
  }
  
  updateName(newName: string): void {
    this.name = newName;  // OK - not readonly
    // this.employeeId = 999;  // Error - cannot modify readonly
  }
}

const emp = new Employee(101, "Bob");
// emp.employeeId = 102;  // Error - readonly property

// ============================================
// GETTERS AND SETTERS
// ============================================

class Temperature {
  private _celsius: number = 0;
  
  // Getter - accessed like a property
  get celsius(): number {
    return this._celsius;
  }
  
  // Setter - set like a property
  set celsius(value: number) {
    if (value < -273.15) {
      throw new Error("Temperature below absolute zero!");
    }
    this._celsius = value;
  }
  
  // Computed property with only getter
  get fahrenheit(): number {
    return (this._celsius * 9/5) + 32;
  }
  
  // Setter for fahrenheit
  set fahrenheit(value: number) {
    this._celsius = (value - 32) * 5/9;
  }
}

const temp = new Temperature();
temp.celsius = 25;                   // Uses setter
console.log(temp.celsius);           // Uses getter: 25
console.log(temp.fahrenheit);        // Uses getter: 77

temp.fahrenheit = 86;                // Uses setter
console.log(temp.celsius);           // 30

// ============================================
// STATIC MEMBERS
// ============================================

class MathUtils {
  // Static property - belongs to class, not instances
  static PI: number = 3.14159;
  static version: string = "1.0.0";
  
  // Static method - called on class, not instance
  static calculateCircleArea(radius: number): number {
    return this.PI * radius * radius;  // this refers to class
  }
  
  static max(...numbers: number[]): number {
    return Math.max(...numbers);
  }
  
  // Instance method (non-static)
  calculateSquareArea(side: number): number {
    return side * side;
  }
}

// Access static members via class name
console.log(MathUtils.PI);  // 3.14159
console.log(MathUtils.calculateCircleArea(5));  // 78.54

// Instance methods require creating an instance
const utils = new MathUtils();
console.log(utils.calculateSquareArea(4));  // 16

// Static factory pattern
class User {
  constructor(
    public username: string,
    public email: string,
    private passwordHash: string
  ) {}
  
  // Static factory method
  static createFromEmail(email: string): User {
    const username = email.split('@')[0];
    const passwordHash = User.hashPassword("temporary");
    return new User(username, email, passwordHash);
  }
  
  static hashPassword(password: string): string {
    // Simplified hash function
    return `hashed_${password}`;
  }
}

const newUser = User.createFromEmail("alice@example.com");
console.log(newUser.username);  // "alice"

// ============================================
// INHERITANCE
// ============================================

// Base class (parent)
class Animal {
  constructor(protected name: string, protected age: number) {}
  
  makeSound(): void {
    console.log("Some generic sound");
  }
  
  getInfo(): string {
    return `${this.name} is ${this.age} years old`;
  }
}

// Derived class (child) - extends base class
class Dog extends Animal {
  constructor(name: string, age: number, private breed: string) {
    super(name, age);  // Call parent constructor
  }
  
  // Override parent method
  makeSound(): void {
    console.log("Woof! Woof!");
  }
  
  // New method specific to Dog
  fetch(): void {
    console.log(`${this.name} is fetching the ball!`);
  }
  
  // Override and extend
  getInfo(): string {
    return `${super.getInfo()} and is a ${this.breed}`;
  }
}

class Cat extends Animal {
  constructor(name: string, age: number, private indoor: boolean) {
    super(name, age);
  }
  
  makeSound(): void {
    console.log("Meow!");
  }
  
  scratch(): void {
    console.log(`${this.name} is scratching!`);
  }
}

const dog = new Dog("Buddy", 3, "Golden Retriever");
dog.makeSound();  // "Woof! Woof!"
dog.fetch();      // "Buddy is fetching the ball!"
console.log(dog.getInfo());  // "Buddy is 3 years old and is a Golden Retriever"

const cat = new Cat("Whiskers", 2, true);
cat.makeSound();  // "Meow!"
cat.scratch();    // "Whiskers is scratching!"

// Polymorphism - treating derived classes as base class
const animals: Animal[] = [dog, cat];
animals.forEach(animal => {
  animal.makeSound();  // Calls appropriate overridden method
});

// ============================================
// ABSTRACT CLASSES
// ============================================

// Abstract class - cannot be instantiated directly
abstract class Shape {
  constructor(protected color: string) {}
  
  // Abstract method - must be implemented by derived classes
  abstract calculateArea(): number;
  abstract calculatePerimeter(): number;
  
  // Concrete method - inherited by derived classes
  describe(): void {
    console.log(`This is a ${this.color} shape with area ${this.calculateArea()}`);
  }
}

class Circle extends Shape {
  constructor(color: string, private radius: number) {
    super(color);
  }
  
  // Must implement abstract methods
  calculateArea(): number {
    return Math.PI * this.radius ** 2;
  }
  
  calculatePerimeter(): number {
    return 2 * Math.PI * this.radius;
  }
}

class Rectangle extends Shape {
  constructor(
    color: string,
    private width: number,
    private height: number
  ) {
    super(color);
  }
  
  calculateArea(): number {
    return this.width * this.height;
  }
  
  calculatePerimeter(): number {
    return 2 * (this.width + this.height);
  }
}

// const shape = new Shape("red");  // Error - cannot instantiate abstract class

const circle = new Circle("blue", 5);
circle.describe();  // "This is a blue shape with area 78.54"

const rectangle = new Rectangle("green", 10, 5);
console.log(rectangle.calculateArea());  // 50

// ============================================
// INTERFACES WITH CLASSES
// ============================================

// Interface defines contract
interface Printable {
  print(): void;
}

interface Serializable {
  serialize(): string;
  deserialize(data: string): void;
}

// Class implementing multiple interfaces
class Document implements Printable, Serializable {
  constructor(private content: string, private title: string) {}
  
  // Implement Printable
  print(): void {
    console.log(`=== ${this.title} ===`);
    console.log(this.content);
  }
  
  // Implement Serializable
  serialize(): string {
    return JSON.stringify({
      title: this.title,
      content: this.content
    });
  }
  
  deserialize(data: string): void {
    const parsed = JSON.parse(data);
    this.title = parsed.title;
    this.content = parsed.content;
  }
}

const doc = new Document("Hello World", "My Document");
doc.print();

const serialized = doc.serialize();
console.log(serialized);  // {"title":"My Document","content":"Hello World"}

// ============================================
// METHOD OVERLOADING
// ============================================

class Calculator {
  // Multiple signatures
  add(a: number, b: number): number;
  add(a: string, b: string): string;
  add(a: number[], b: number[]): number[];
  
  // Implementation (must handle all signatures)
  add(a: any, b: any): any {
    if (typeof a === "number" && typeof b === "number") {
      return a + b;
    }
    if (typeof a === "string" && typeof b === "string") {
      return a + b;
    }
    if (Array.isArray(a) && Array.isArray(b)) {
      return [...a, ...b];
    }
  }
}

const calc = new Calculator();
console.log(calc.add(5, 3));           // 8
console.log(calc.add("Hello ", "World"));  // "Hello World"
console.log(calc.add([1, 2], [3, 4])); // [1, 2, 3, 4]

// ============================================
// REAL-WORLD EXAMPLE: USER MANAGEMENT SYSTEM
// ============================================

// Base User class
abstract class BaseUser {
  protected createdAt: Date;
  protected lastLogin?: Date;
  
  constructor(
    public readonly id: number,
    public username: string,
    protected email: string
  ) {
    this.createdAt = new Date();
  }
  
  // Abstract methods
  abstract getPermissions(): string[];
  abstract canAccessResource(resource: string): boolean;
  
  // Concrete methods
  login(): void {
    this.lastLogin = new Date();
    console.log(`${this.username} logged in at ${this.lastLogin}`);
  }
  
  getAccountAge(): number {
    return Date.now() - this.createdAt.getTime();
  }
}

// Admin user
class AdminUser extends BaseUser {
  private static adminCount: number = 0;
  
  constructor(id: number, username: string, email: string) {
    super(id, username, email);
    AdminUser.adminCount++;
  }
  
  getPermissions(): string[] {
    return ["read", "write", "delete", "admin"];
  }
  
  canAccessResource(resource: string): boolean {
    return true;  // Admin can access everything
  }
  
  static getAdminCount(): number {
    return AdminUser.adminCount;
  }
}

// Regular user
class RegularUser extends BaseUser {
  private resources: string[] = [];
  
  constructor(id: number, username: string, email: string) {
    super(id, username, email);
  }
  
  getPermissions(): string[] {
    return ["read", "write"];
  }
  
  canAccessResource(resource: string): boolean {
    return this.resources.includes(resource);
  }
  
  grantAccess(resource: string): void {
    if (!this.resources.includes(resource)) {
      this.resources.push(resource);
    }
  }
}

// Usage
const admin = new AdminUser(1, "admin", "admin@example.com");
const user1 = new RegularUser(2, "alice", "alice@example.com");
const user2 = new RegularUser(3, "bob", "bob@example.com");

admin.login();
user1.grantAccess("documents");
console.log(user1.canAccessResource("documents"));  // true
console.log(user1.canAccessResource("admin-panel"));  // false
console.log(admin.canAccessResource("admin-panel"));  // true
```

### Abstraction, Polymorphism and Inheritance

```typescript
// ============================================
// ABSTRACTION
// ============================================

// Hide complex implementation details behind simple interface
abstract class PaymentProcessor {
  constructor(protected amount: number) {}
  
  // Template method pattern - defines algorithm structure
  processPayment(): boolean {
    if (!this.validateAmount()) {
      console.log("Invalid amount");
      return false;
    }
    
    if (!this.authenticate()) {
      console.log("Authentication failed");
      return false;
    }
    
    if (!this.charge()) {
      console.log("Charge failed");
      return false;
    }
    
    this.sendConfirmation();
    return true;
  }
  
  // Common validation
  private validateAmount(): boolean {
    return this.amount > 0;
  }
  
  // Abstract methods - subclasses implement details
  protected abstract authenticate(): boolean;
  protected abstract charge(): boolean;
  protected abstract sendConfirmation(): void;
}

// Credit card payment
class CreditCardPayment extends PaymentProcessor {
  constructor(
    amount: number,
    private cardNumber: string,
    private cvv: string
  ) {
    super(amount);
  }
  
  protected authenticate(): boolean {
    console.log("Authenticating credit card...");
    // Validate card number and CVV
    return this.cardNumber.length === 16 && this.cvv.length === 3;
  }
  
  protected charge(): boolean {
    console.log(`Charging $${this.amount} to credit card`);
    // Process credit card charge
    return true;
  }
  
  protected sendConfirmation(): void {
    console.log("Credit card payment confirmation sent");
  }
}

// PayPal payment
class PayPalPayment extends PaymentProcessor {
  constructor(
    amount: number,
    private email: string,
    private password: string
  ) {
    super(amount);
  }
  
  protected authenticate(): boolean {
    console.log("Authenticating PayPal account...");
    // Validate PayPal credentials
    return this.email.includes('@') && this.password.length > 0;
  }
  
  protected charge(): boolean {
    console.log(`Charging $${this.amount} via PayPal`);
    // Process PayPal charge
    return true;
  }
  
  protected sendConfirmation(): void {
    console.log(`PayPal confirmation sent to ${this.email}`);
  }
}

// Client code doesn't need to know implementation details
function checkout(processor: PaymentProcessor): void {
  const success = processor.processPayment();
  console.log(success ? "Payment successful" : "Payment failed");
}

const creditCard = new CreditCardPayment(100, "1234567890123456", "123");
const paypal = new PayPalPayment(50, "user@example.com", "password");

checkout(creditCard);
checkout(paypal);

// ============================================
// POLYMORPHISM
// ============================================

// Different classes responding to same method call in their own way
interface Vehicle {
  start(): void;
  stop(): void;
  getSpeed(): number;
}

class Car implements Vehicle {
  private speed: number = 0;
  
  start(): void {
    console.log("Car engine started");
    this.speed = 10;
  }
  
  stop(): void {
    console.log("Car engine stopped");
    this.speed = 0;
  }
  
  getSpeed(): number {
    return this.speed;
  }
  
  honk(): void {
    console.log("Beep beep!");
  }
}

class Bicycle implements Vehicle {
  private speed: number = 0;
  
  start(): void {
    console.log("Bicycle pedaling started");
    this.speed = 5;
  }
  
  stop(): void {
    console.log("Bicycle stopped");
    this.speed = 0;
  }
  
  getSpeed(): number {
    return this.speed;
  }
  
  ringBell(): void {
    console.log("Ring ring!");
  }
}

class Motorcycle implements Vehicle {
  private speed: number = 0;
  
  start(): void {
    console.log("Motorcycle engine roaring");
    this.speed = 15;
  }
  
  stop(): void {
    console.log("Motorcycle engine off");
    this.speed = 0;
  }
  
  getSpeed(): number {
    return this.speed;
  }
}

// Polymorphism in action - same interface, different implementations
function startJourney(vehicle: Vehicle): void {
  vehicle.start();  // Each vehicle starts differently
  console.log(`Current speed: ${vehicle.getSpeed()} km/h`);
}

const car = new Car();
const bike = new Bicycle();
const moto = new Motorcycle();

startJourney(car);   // "Car engine started" - Speed: 10
startJourney(bike);  // "Bicycle pedaling started" - Speed: 5
startJourney(moto);  // "Motorcycle engine roaring" - Speed: 15

// ============================================
// COMPOSITION OVER INHERITANCE
// ============================================

// Instead of deep inheritance hierarchies, use composition

// Behaviors as separate classes
class FlyBehavior {
  fly(): void {
    console.log("Flying through the air");
  }
}

class NoFlyBehavior {
  fly(): void {
    console.log("Cannot fly");
  }
}

class SwimBehavior {
  swim(): void {
    console.log("Swimming in water");
  }
}

class NoSwimBehavior {
  swim(): void {
    console.log("Cannot swim");
  }
}

// Animal composed of behaviors
class ComposedAnimal {
  constructor(
    private name: string,
    private flyBehavior: FlyBehavior | NoFlyBehavior,
    private swimBehavior: SwimBehavior | NoSwimBehavior
  ) {}
  
  performFly(): void {
    console.log(`${this.name}:`);
    this.flyBehavior.fly();
  }
  
  performSwim(): void {
    console.log(`${this.name}:`);
    this.swimBehavior.swim();
  }
  
  // Can dynamically change behavior
  setFlyBehavior(behavior: FlyBehavior | NoFlyBehavior): void {
    this.flyBehavior = behavior;
  }
}

const duck = new ComposedAnimal(
  "Duck",
  new FlyBehavior(),
  new SwimBehavior()
);

const penguin = new ComposedAnimal(
  "Penguin",
  new NoFlyBehavior(),
  new SwimBehavior()
);

duck.performFly();     // "Duck: Flying through the air"
duck.performSwim();    // "Duck: Swimming in water"

penguin.performFly();  // "Penguin: Cannot fly"
penguin.performSwim(); // "Penguin: Swimming in water"

// ============================================
// MIXINS
// ============================================

// Mixin pattern - add functionality to classes
type Constructor<T = {}> = new (...args: any[]) => T;

// Timestamped mixin
function Timestamped<TBase extends Constructor>(Base: TBase) {
  return class extends Base {
    createdAt = new Date();
    
    getAge(): number {
      return Date.now() - this.createdAt.getTime();
    }
  };
}

// Activatable mixin
function Activatable<TBase extends Constructor>(Base: TBase) {
  return class extends Base {
    isActive = false;
    
    activate(): void {
      this.isActive = true;
      console.log("Activated");
    }
    
    deactivate(): void {
      this.isActive = false;
      console.log("Deactivated");
    }
  };
}

// Base class
class BaseEntity {
  constructor(public id: number) {}
}

// Apply mixins
const TimestampedEntity = Timestamped(BaseEntity);
const ActivatableEntity = Activatable(BaseEntity);
const FullEntity = Timestamped(Activatable(BaseEntity));

// Usage
const entity = new FullEntity(1);
console.log(entity.id);
entity.activate();
console.log(entity.isActive);  // true
console.log(entity.getAge());  // milliseconds since creation
```

---

## Low-Level Design Patterns in TypeScript

### Creational Patterns

```typescript
// ============================================
// SINGLETON PATTERN
// ============================================

// Ensures only one instance of a class exists
class Database {
  private static instance: Database;
  private connection: string;
  
  // Private constructor prevents direct instantiation
  private constructor() {
    this.connection = "Connected to database";
    console.log("Database instance created");
  }
  
  // Global access point to instance
  static getInstance(): Database {
    if (!Database.instance) {
      Database.instance = new Database();
    }
    return Database.instance;
  }
  
  query(sql: string): void {
    console.log(`Executing: ${sql}`);
  }
  
  getConnection(): string {
    return this.connection;
  }
}

// Usage
const db1 = Database.getInstance();
const db2 = Database.getInstance();

console.log(db1 === db2);  // true - same instance
db1.query("SELECT * FROM users");

// Modern singleton with module pattern
class Logger {
  private logs: string[] = [];
  
  log(message: string): void {
    const timestamp = new Date().toISOString();
    this.logs.push(`[${timestamp}] ${message}`);
    console.log(`[${timestamp}] ${message}`);
  }
  
  getLogs(): string[] {
    return [...this.logs];
  }
  
  clearLogs(): void {
    this.logs = [];
  }
}

// Export single instance
export const logger = new Logger();

// ============================================
// FACTORY PATTERN
// ============================================

// Creates objects without specifying exact class
interface Notification {
  send(message: string): void;
}

class EmailNotification implements Notification {
  constructor(private recipient: string) {}
  
  send(message: string): void {
    console.log(`Sending email to ${this.recipient}: ${message}`);
  }
}

class SMSNotification implements Notification {
  constructor(private phoneNumber: string) {}
  
  send(message: string): void {
    console.log(`Sending SMS to ${this.phoneNumber}: ${message}`);
  }
}

class PushNotification implements Notification {
  constructor(private deviceId: string) {}
  
  send(message: string): void {
    console.log(`Sending push to device ${this.deviceId}: ${message}`);
  }
}

// Factory class
class NotificationFactory {
  createNotification(type: string, target: string): Notification {
    switch (type) {
      case "email":
        return new EmailNotification(target);
      case "sms":
        return new SMSNotification(target);
      case "push":
        return new PushNotification(target);
      default:
        throw new Error(`Unknown notification type: ${type}`);
    }
  }
}

// Usage
const factory = new NotificationFactory();
const emailNotif = factory.createNotification("email", "user@example.com");
const smsNotif = factory.createNotification("sms", "+1234567890");

emailNotif.send("Welcome!");
smsNotif.send("Verification code: 123456");

// ============================================
// BUILDER PATTERN
// ============================================

// Construct complex objects step by step
class HttpRequest {
  private url: string = "";
  private method: string = "GET";
  private headers: Map<string, string> = new Map();
  private body?: any;
  private timeout: number = 5000;
  
  setUrl(url: string): this {
    this.url = url;
    return this;  // Return this for chaining
  }
  
  setMethod(method: string): this {
    this.method = method;
    return this;
  }
  
  addHeader(key: string, value: string): this {
    this.headers.set(key, value);
    return this;
  }
  
  setBody(body: any): this {
    this.body = body;
    return this;
  }
  
  setTimeout(timeout: number): this {
    this.timeout = timeout;
    return this;
  }
  
  async execute(): Promise<any> {
    console.log(`${this.method} ${this.url}`);
    console.log("Headers:", Array.from(this.headers.entries()));
    if (this.body) {
      console.log("Body:", this.body);
    }
    console.log(`Timeout: ${this.timeout}ms`);
    // Actual HTTP request would go here
    return { status: 200, data: {} };
  }
}

// Usage - fluent API
const request = new HttpRequest()
  .setUrl("https://api.example.com/users")
  .setMethod("POST")
  .addHeader("Content-Type", "application/json")
  .addHeader("Authorization", "Bearer token123")
  .setBody({ name: "John", email: "john@example.com" })
  .setTimeout(10000);

request.execute();

// Advanced builder with director
class User {
  constructor(
    public firstName: string,
    public lastName: string,
    public email: string,
    public age?: number,
    public address?: string,
    public phone?: string
  ) {}
}

class UserBuilder {
  private firstName: string = "";
  private lastName: string = "";
  private email: string = "";
  private age?: number;
  private address?: string;
  private phone?: string;
  
  setFirstName(firstName: string): this {
    this.firstName = firstName;
    return this;
  }
  
  setLastName(lastName: string): this {
    this.lastName = lastName;
    return this;
  }
  
  setEmail(email: string): this {
    this.email = email;
    return this;
  }
  
  setAge(age: number): this {
    this.age = age;
    return this;
  }
  
  setAddress(address: string): this {
    this.address = address;
    return this;
  }
  
  setPhone(phone: string): this {
    this.phone = phone;
    return this;
  }
  
  build(): User {
    // Validation before building
    if (!this.firstName || !this.lastName || !this.email) {
      throw new Error("Required fields missing");
    }
    
    return new User(
      this.firstName,
      this.lastName,
      this.email,
      this.age,
      this.address,
      this.phone
    );
  }
}

// Usage
const user = new UserBuilder()
  .setFirstName("John")
  .setLastName("Doe")
  .setEmail("john.doe@example.com")
  .setAge(30)
  .setAddress("123 Main St")
  .build();

console.log(user);

// Director for common build patterns
class UserDirector {
  static buildAdminUser(builder: UserBuilder): User {
    return builder
      .setFirstName("Admin")
      .setLastName("User")
      .setEmail("admin@example.com")
      .build();
  }
  
  static buildGuestUser(builder: UserBuilder): User {
    return builder
      .setFirstName("Guest")
      .setLastName("User")
      .setEmail("guest@example.com")
      .build();
  }
}

const admin = UserDirector.buildAdminUser(new UserBuilder());
const guest = UserDirector.buildGuestUser(new UserBuilder());
```

### Structural Patterns

```typescript
// ============================================
// ADAPTER PATTERN
// ============================================

// Makes incompatible interfaces work together

// Old system
class OldPaymentService {
  processPayment(amount: number, currency: string): void {
    console.log(`Processing ${amount} ${currency} via old system`);
  }
}

// New interface our application expects
interface PaymentService {
  pay(amount: number, details: { currency: string; method: string }): void;
}

// Adapter wraps old system to work with new interface
class PaymentAdapter implements PaymentService {
  constructor(private oldService: OldPaymentService) {}
  
  pay(amount: number, details: { currency: string; method: string }): void {
    // Adapt new interface to old implementation
    console.log(`Using payment method: ${details.method}`);
    this.oldService.processPayment(amount, details.currency);
  }
}

// Usage
const oldService = new OldPaymentService();
const adapter = new PaymentAdapter(oldService);

adapter.pay(100, { currency: "USD", method: "credit-card" });

// ============================================
// FACADE PATTERN
// ============================================

// Provides simplified interface to complex subsystem

// Complex subsystems
class CPU {
  freeze(): void { console.log("CPU: Freezing..."); }
  jump(position: number): void { console.log(`CPU: Jumping to ${position}`); }
  execute(): void { console.log("CPU: Executing..."); }
}

class Memory {
  load(position: number, data: string): void {
    console.log(`Memory: Loading "${data}" at position ${position}`);
  }
}

class HardDrive {
  read(sector: number, size: number): string {
    console.log(`HardDrive: Reading ${size} bytes from sector ${sector}`);
    return "boot data";
  }
}

// Facade provides simple interface
class ComputerFacade {
  private cpu: CPU;
  private memory: Memory;
  private hardDrive: HardDrive;
  
  constructor() {
    this.cpu = new CPU();
    this.memory = new Memory();
    this.hardDrive = new HardDrive();
  }
  
  // Simple method hides complex interaction
  start(): void {
    console.log("Starting computer...");
    this.cpu.freeze();
    const bootData = this.hardDrive.read(0, 1024);
    this.memory.load(0, bootData);
    this.cpu.jump(0);
    this.cpu.execute();
    console.log("Computer started!");
  }
}

// Usage - much simpler!
const computer = new ComputerFacade();
computer.start();

// ============================================
// DECORATOR PATTERN
// ============================================

// Add behavior to objects dynamically

interface Coffee {
  cost(): number;
  description(): string;
}

// Base component
class SimpleCoffee implements Coffee {
  cost(): number {
    return 5;
  }
  
  description(): string {
    return "Simple coffee";
  }
}

// Base decorator
abstract class CoffeeDecorator implements Coffee {
  constructor(protected coffee: Coffee) {}
  
  abstract cost(): number;
  abstract description(): string;
}

// Concrete decorators
class MilkDecorator extends CoffeeDecorator {
  cost(): number {
    return this.coffee.cost() + 1.5;
  }
  
  description(): string {
    return this.coffee.description() + ", milk";
  }
}

class SugarDecorator extends CoffeeDecorator {
  cost(): number {
    return this.coffee.cost() + 0.5;
  }
  
  description(): string {
    return this.coffee.description() + ", sugar";
  }
}

class WhippedCreamDecorator extends CoffeeDecorator {
  cost(): number {
    return this.coffee.cost() + 2;
  }
  
  description(): string {
    return this.coffee.description() + ", whipped cream";
  }
}

// Usage - stack decorators
let coffee: Coffee = new SimpleCoffee();
console.log(`${coffee.description()} = $${coffee.cost()}`);

coffee = new MilkDecorator(coffee);
console.log(`${coffee.description()} = $${coffee.cost()}`);

coffee = new SugarDecorator(coffee);
console.log(`${coffee.description()} = $${coffee.cost()}`);

coffee = new WhippedCreamDecorator(coffee);
console.log(`${coffee.description()} = $${coffee.cost()}`);
// Output: "Simple coffee, milk, sugar, whipped cream = $9"

// ============================================
// PROXY PATTERN
// ============================================

// Provides placeholder/surrogate for another object

interface Image {
  display(): void;
}

// Real object - expensive to create
class RealImage implements Image {
  constructor(private filename: string) {
    this.loadFromDisk();
  }
  
  private loadFromDisk(): void {
    console.log(`Loading image: ${this.filename}`);
    // Simulate expensive operation
  }
  
  display(): void {
    console.log(`Displaying image: ${this.filename}`);
  }
}

// Proxy - controls access to real object
class ProxyImage implements Image {
  private realImage?: RealImage;
  
  constructor(private filename: string) {}
  
  display(): void {
    // Lazy loading - only create when needed
    if (!this.realImage) {
      this.realImage = new RealImage(this.filename);
    }
    this.realImage.display();
  }
}

// Usage
const image1 = new ProxyImage("photo1.jpg");
const image2 = new ProxyImage("photo2.jpg");

// Image not loaded yet
console.log("Images created");

// Image loaded on first display
image1.display();  // Loading... then displaying
image1.display();  // Only displaying (already loaded)

image2.display();  // Loading... then displaying
```

### Behavioral Patterns

```typescript
// ============================================
// STRATEGY PATTERN
// ============================================

// Define family of algorithms, encapsulate each, make them interchangeable

interface SortStrategy {
  sort(data: number[]): number[];
}

class BubbleSort implements SortStrategy {
  sort(data: number[]): number[] {
    console.log("Using Bubble Sort");
    const arr = [...data];
    for (let i = 0; i < arr.length; i++) {
      for (let j = 0; j < arr.length - 1 - i; j++) {
        if (arr[j] > arr[j + 1]) {
          [arr[j], arr[j + 1]] = [arr[j + 1], arr[j]];
        }
      }
    }
    return arr;
  }
}

class QuickSort implements SortStrategy {
  sort(data: number[]): number[] {
    console.log("Using Quick Sort");
    if (data.length <= 1) return data;
    
    const pivot = data[Math.floor(data.length / 2)];
    const left = data.filter(x => x < pivot);
    const middle = data.filter(x => x === pivot);
    const right = data.filter(x => x > pivot);
    
    return [...this.sort(left), ...middle, ...this.sort(right)];
  }
}

class MergeSort implements SortStrategy {
  sort(data: number[]): number[] {
    console.log("Using Merge Sort");
    if (data.length <= 1) return data;
    
    const mid = Math.floor(data.length / 2);
    const left = this.sort(data.slice(0, mid));
    const right = this.sort(data.slice(mid));
    
    return this.merge(left, right);
  }
  
  private merge(left: number[], right: number[]): number[] {
    const result: number[] = [];
    let i = 0, j = 0;
    
    while (i < left.length && j < right.length) {
      if (left[i] < right[j]) {
        result.push(left[i++]);
      } else {
        result.push(right[j++]);
      }
    }
    
    return result.concat(left.slice(i)).concat(right.slice(j));
  }
}

// Context uses strategy
class DataSorter {
  private strategy: SortStrategy;
  
  constructor(strategy: SortStrategy) {
    this.strategy = strategy;
  }
  
  setStrategy(strategy: SortStrategy): void {
    this.strategy = strategy;
  }
  
  sort(data: number[]): number[] {
    return this.strategy.sort(data);
  }
}

// Usage
const data = [64, 34, 25, 12, 22, 11, 90];
const sorter = new DataSorter(new BubbleSort());

console.log(sorter.sort(data));

// Change strategy at runtime
sorter.setStrategy(new QuickSort());
console.log(sorter.sort(data));

sorter.setStrategy(new MergeSort());
console.log(sorter.sort(data));

// ============================================
// OBSERVER PATTERN
// ============================================

// One-to-many dependency where objects are notified of state changes

interface Observer {
  update(data: any): void;
}

interface Subject {
  attach(observer: Observer): void;
  detach(observer: Observer): void;
  notify(): void;
}

// Concrete subject
class NewsAgency implements Subject {
  private observers: Observer[] = [];
  private news: string = "";
  
  attach(observer: Observer): void {
    const exists = this.observers.includes(observer);
    if (exists) {
      console.log("Observer already attached");
      return;
    }
    this.observers.push(observer);
    console.log("Observer attached");
  }
  
  detach(observer: Observer): void {
    const index = this.observers.indexOf(observer);
    if (index === -1) {
      console.log("Observer not found");
      return;
    }
    this.observers.splice(index, 1);
    console.log("Observer detached");
  }
  
  notify(): void {
    console.log("Notifying all observers...");
    for (const observer of this.observers) {
      observer.update(this.news);
    }
  }
  
  setNews(news: string): void {
    this.news = news;
    this.notify();  // Automatically notify on state change
  }
  
  getNews(): string {
    return this.news;
  }
}

// Concrete observers
class NewsChannel implements Observer {
  constructor(private channelName: string) {}
  
  update(news: string): void {
    console.log(`${this.channelName} received news: ${news}`);
  }
}

class NewsWebsite implements Observer {
  constructor(private siteName: string) {}
  
  update(news: string): void {
    console.log(`${this.siteName} published: ${news}`);
  }
}

// Usage
const agency = new NewsAgency();

const cnn = new NewsChannel("CNN");
const bbc = new NewsChannel("BBC");
const website = new NewsWebsite("News.com");

agency.attach(cnn);
agency.attach(bbc);
agency.attach(website);

agency.setNews("Breaking: Major event occurred!");
// All observers notified

agency.detach(bbc);
agency.setNews("Update: More details available");
// Only CNN and website notified

// ============================================
// COMMAND PATTERN
// ============================================

// Encapsulate requests as objects

interface Command {
  execute(): void;
  undo(): void;
}

// Receiver - knows how to perform operations
class Light {
  private isOn: boolean = false;
  
  turnOn(): void {
    this.isOn = true;
    console.log("Light is ON");
  }
  
  turnOff(): void {
    this.isOn = false;
    console.log("Light is OFF");
  }
  
  getStatus(): boolean {
    return this.isOn;
  }
}

// Concrete commands
class TurnOnCommand implements Command {
  constructor(private light: Light) {}
  
  execute(): void {
    this.light.turnOn();
  }
  
  undo(): void {
    this.light.turnOff();
  }
}

class TurnOffCommand implements Command {
  constructor(private light: Light) {}
  
  execute(): void {
    this.light.turnOff();
  }
  
  undo(): void {
    this.light.turnOn();
  }
}

// Invoker - triggers commands
class RemoteControl {
  private history: Command[] = [];
  
  executeCommand(command: Command): void {
    command.execute();
    this.history.push(command);  // Save for undo
  }
  
  undo(): void {
    const command = this.history.pop();
    if (command) {
      command.undo();
    } else {
      console.log("Nothing to undo");
    }
  }
}

// Usage
const light = new Light();
const turnOn = new TurnOnCommand(light);
const turnOff = new TurnOffCommand(light);
const remote = new RemoteControl();

remote.executeCommand(turnOn);   // Light is ON
remote.executeCommand(turnOff);  // Light is OFF
remote.undo();                   // Light is ON (undo turn off)
remote.undo();                   // Light is OFF (undo turn on)

// ============================================
// STATE PATTERN
// ============================================

// Object behavior changes based on its internal state

interface State {
  insertCoin(): void;
  ejectCoin(): void;
  dispense(): void;
}

// Context
class VendingMachine {
  private noCoinState: State;
  private hasCoinState: State;
  private soldState: State;
  private currentState: State;
  private itemCount: number;
  
  constructor(itemCount: number) {
    this.noCoinState = new NoCoinState(this);
    this.hasCoinState = new HasCoinState(this);
    this.soldState = new SoldState(this);
    this.itemCount = itemCount;
    this.currentState = this.noCoinState;
  }
  
  setState(state: State): void {
    this.currentState = state;
  }
  
  insertCoin(): void {
    this.currentState.insertCoin();
  }
  
  ejectCoin(): void {
    this.currentState.ejectCoin();
  }
  
  dispense(): void {
    this.currentState.dispense();
  }
  
  releaseItem(): void {
    if (this.itemCount > 0) {
      console.log("Item dispensed");
      this.itemCount--;
    }
  }
  
  getNoCoinState(): State { return this.noCoinState; }
  getHasCoinState(): State { return this.hasCoinState; }
  getSoldState(): State { return this.soldState; }
  getItemCount(): number { return this.itemCount; }
}

// Concrete states
class NoCoinState implements State {
  constructor(private machine: VendingMachine) {}
  
  insertCoin(): void {
    console.log("Coin inserted");
    this.machine.setState(this.machine.getHasCoinState());
  }
  
  ejectCoin(): void {
    console.log("No coin to eject");
  }
  
  dispense(): void {
    console.log("Insert coin first");
  }
}

class HasCoinState implements State {
  constructor(private machine: VendingMachine) {}
  
  insertCoin(): void {
    console.log("Coin already inserted");
  }
  
  ejectCoin(): void {
    console.log("Coin ejected");
    this.machine.setState(this.machine.getNoCoinState());
  }
  
  dispense(): void {
    console.log("Processing...");
    this.machine.setState(this.machine.getSoldState());
    this.machine.dispense();
  }
}

class SoldState implements State {
  constructor(private machine: VendingMachine) {}
  
  insertCoin(): void {
    console.log("Please wait, dispensing item");
  }
  
  ejectCoin(): void {
    console.log("Cannot eject, item being dispensed");
  }
  
  dispense(): void {
    this.machine.releaseItem();
    if (this.machine.getItemCount() > 0) {
      this.machine.setState(this.machine.getNoCoinState());
    } else {
      console.log("Out of items");
    }
  }
}

// Usage
const vendingMachine = new VendingMachine(2);

vendingMachine.insertCoin();  // Coin inserted
vendingMachine.dispense();    // Processing... Item dispensed

vendingMachine.insertCoin();  // Coin inserted
vendingMachine.ejectCoin();   // Coin ejected

vendingMachine.dispense();    // Insert coin first

// ============================================
// CHAIN OF RESPONSIBILITY PATTERN
// ============================================

// Pass request along chain of handlers

abstract class Handler {
  private nextHandler?: Handler;
  
  setNext(handler: Handler): Handler {
    this.nextHandler = handler;
    return handler;  // Allow chaining
  }
  
  handle(request: string): string | null {
    // If can handle, process it
    const result = this.process(request);
    if (result !== null) {
      return result;
    }
    
    // Otherwise, pass to next handler
    if (this.nextHandler) {
      return this.nextHandler.handle(request);
    }
    
    return null;
  }
  
  protected abstract process(request: string): string | null;
}

// Concrete handlers
class BasicSupportHandler extends Handler {
  protected process(request: string): string | null {
    if (request === "basic") {
      return "Basic Support: Issue resolved";
    }
    return null;
  }
}

class TechnicalSupportHandler extends Handler {
  protected process(request: string): string | null {
    if (request === "technical") {
      return "Technical Support: Issue resolved";
    }
    return null;
  }
}

class ManagerHandler extends Handler {
  protected process(request: string): string | null {
    if (request === "manager") {
      return "Manager: Issue escalated and resolved";
    }
    return null;
  }
}

// Build chain
const basic = new BasicSupportHandler();
const technical = new TechnicalSupportHandler();
const manager = new ManagerHandler();

basic.setNext(technical).setNext(manager);

// Send requests through chain
console.log(basic.handle("basic"));      // Basic Support
console.log(basic.handle("technical"));  // Technical Support
console.log(basic.handle("manager"));    // Manager
console.log(basic.handle("unknown"));    // null

// ============================================
// TEMPLATE METHOD PATTERN
// ============================================

// Define skeleton of algorithm, let subclasses override specific steps

abstract class DataParser {
  // Template method - defines the algorithm structure
  parse(filePath: string): void {
    const data = this.readFile(filePath);
    const parsedData = this.parseData(data);
    const validated = this.validateData(parsedData);
    this.processData(validated);
    this.cleanup();
  }
  
  // Concrete method
  private readFile(path: string): string {
    console.log(`Reading file: ${path}`);
    return "raw data";  // Simulated
  }
  
  // Abstract methods - subclasses must implement
  protected abstract parseData(data: string): any;
  protected abstract validateData(data: any): any;
  
  // Hook methods - subclasses can override (optional)
  protected processData(data: any): void {
    console.log("Processing data:", data);
  }
  
  protected cleanup(): void {
    console.log("Cleanup completed");
  }
}

class JSONParser extends DataParser {
  protected parseData(data: string): any {
    console.log("Parsing JSON data");
    return JSON.parse(data || "{}");
  }
  
  protected validateData(data: any): any {
    console.log("Validating JSON data");
    // Add JSON-specific validation
    return data;
  }
}

class CSVParser extends DataParser {
  protected parseData(data: string): any {
    console.log("Parsing CSV data");
    // CSV parsing logic
    return data.split(",");
  }
  
  protected validateData(data: any): any {
    console.log("Validating CSV data");
    // Add CSV-specific validation
    return data;
  }
  
  // Override hook method
  protected cleanup(): void {
    console.log("CSV-specific cleanup");
    super.cleanup();
  }
}

// Usage
const jsonParser = new JSONParser();
jsonParser.parse("data.json");

const csvParser = new CSVParser();
csvParser.parse("data.csv");
```

---

## Express.js with TypeScript

### Setting Up Express with TypeScript

```bash
# Create project directory
mkdir express-ts-app
cd express-ts-app

# Initialize npm project
npm init -y

# Install dependencies
npm install express
npm install --save-dev typescript @types/node @types/express
npm install --save-dev ts-node nodemon

# Initialize TypeScript
npx tsc --init
```

**Enhanced tsconfig.json for Express:**

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "lib": ["ES2020"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "moduleResolution": "node",
    "allowSyntheticDefaultImports": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

**package.json scripts:**

```json
{
  "scripts": {
    "dev": "nodemon --exec ts-node src/index.ts",
    "build": "tsc",
    "start": "node dist/index.js",
    "watch": "tsc --watch"
  }
}
```

### Basic Express Server with TypeScript

```typescript
// ============================================
// src/index.ts - BASIC EXPRESS SERVER
// ============================================

import express, { Express, Request, Response, NextFunction } from 'express';

// Create Express application instance
const app: Express = express();
const PORT: number = process.env.PORT ? parseInt(process.env.PORT) : 3000;

// ============================================
// MIDDLEWARE
// ============================================

// Built-in middleware - parse JSON request bodies
app.use(express.json());

// Built-in middleware - parse URL-encoded bodies
app.use(express.urlencoded({ extended: true }));

// Custom logging middleware
app.use((req: Request, res: Response, next: NextFunction) => {
  const timestamp = new Date().toISOString();
  console.log(`[${timestamp}] ${req.method} ${req.path}`);
  next();  // Pass control to next middleware
});

// ============================================
// ROUTES
// ============================================

// Simple GET route
app.get('/', (req: Request, res: Response) => {
  res.json({
    message: 'Welcome to Express with TypeScript!',
    timestamp: new Date().toISOString()
  });
});

// Route with path parameter
app.get('/users/:id', (req: Request, res: Response) => {
  const userId = req.params.id;  // Extract path parameter
  res.json({
    message: `Fetching user ${userId}`,
    userId: userId
  });
});

// Route with query parameters
app.get('/search', (req: Request, res: Response) => {
  const query = req.query.q;         // Get query parameter ?q=...
  const limit = req.query.limit;      // Get query parameter ?limit=...
  
  res.json({
    query: query,
    limit: limit,
    results: []
  });
});

// POST route - handle request body
app.post('/users', (req: Request, res: Response) => {
  const userData = req.body;  // Parse JSON body
  
  console.log('Creating user:', userData);
  
  res.status(201).json({  // 201 Created
    message: 'User created successfully',
    user: userData
  });
});

// PUT route - update resource
app.put('/users/:id', (req: Request, res: Response) => {
  const userId = req.params.id;
  const updates = req.body;
  
  res.json({
    message: `User ${userId} updated`,
    updates: updates
  });
});

// DELETE route
app.delete('/users/:id', (req: Request, res: Response) => {
  const userId = req.params.id;
  
  res.json({
    message: `User ${userId} deleted`
  });
});

// ============================================
// ERROR HANDLING
// ============================================

// 404 handler - must be after all routes
app.use((req: Request, res: Response) => {
  res.status(404).json({
    error: 'Route not found',
    path: req.path
  });
});

// Global error handler - must be last
app.use((err: Error, req: Request, res: Response, next: NextFunction) => {
  console.error('Error:', err.message);
  res.status(500).json({
    error: 'Internal server error',
    message: err.message
  });
});

// ============================================
// START SERVER
// ============================================

app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});

export default app;
```

### Dynamic Data with URL Parameters and Query Strings

```typescript
// ============================================
// ADVANCED ROUTING PATTERNS
// ============================================

import { Router, Request, Response } from 'express';

const router = Router();

// Multiple path parameters
router.get('/posts/:userId/comments/:commentId', (req: Request, res: Response) => {
  const { userId, commentId } = req.params;
  
  res.json({
    userId,
    commentId,
    message: `Fetching comment ${commentId} from user ${userId}`
  });
});

// Optional path parameters using regex
router.get('/files/:filename([\\w-]+\\.(jpg|png|gif))', (req: Request, res: Response) => {
  const { filename } = req.params;
  
  res.json({
    message: `Downloading file: ${filename}`,
    type: 'image'
  });
});

// Query string parsing with validation
interface ProductQuery {
  category?: string;
  minPrice?: string;
  maxPrice?: string;
  sort?: 'price' | 'name' | 'rating';
  order?: 'asc' | 'desc';
  page?: string;
  limit?: string;
}

router.get('/products', (req: Request<{}, {}, {}, ProductQuery>, res: Response) => {
  // Parse and validate query parameters
  const {
    category,
    minPrice = '0',
    maxPrice = '10000',
    sort = 'name',
    order = 'asc',
    page = '1',
    limit = '20'
  } = req.query;
  
  // Convert string params to numbers
  const filters = {
    category,
    minPrice: parseFloat(minPrice),
    maxPrice: parseFloat(maxPrice),
    sort,
    order,
    page: parseInt(page),
    limit: parseInt(limit)
  };
  
  // Validate numeric values
  if (isNaN(filters.minPrice) || isNaN(filters.maxPrice)) {
    res.status(400).json({ error: 'Invalid price range' });
    return;
  }
  
  if (isNaN(filters.page) || filters.page < 1) {
    res.status(400).json({ error: 'Invalid page number' });
    return;
  }
  
  res.json({
    message: 'Products filtered',
    filters,
    results: []  // Actual product query would go here
  });
});

// Array query parameters
router.get('/search/multi', (req: Request, res: Response) => {
  // URL: /search/multi?tags=javascript&tags=typescript&tags=node
  let tags = req.query.tags;
  
  // Ensure tags is always an array
  if (typeof tags === 'string') {
    tags = [tags];
  }
  
  res.json({
    searchTags: tags,
    resultCount: 0
  });
});

// Nested object query parameters
router.get('/advanced-search', (req: Request, res: Response) => {
  // URL: /advanced-search?filter[status]=active&filter[role]=admin
  const filter = req.query.filter as any;
  
  res.json({
    appliedFilters: filter,
    results: []
  });
});

export default router;
```

### Middleware - Most Important Aspect of Express

```typescript
// ============================================
// MIDDLEWARE DEEP DIVE
// ============================================

import { Request, Response, NextFunction } from 'express';
import { AuthRequest } from '../types/express';

// ============================================
// LOGGING MIDDLEWARE
// ============================================

// Morgan-style request logger
export const requestLogger = (
  req: Request,
  res: Response,
  next: NextFunction
): void => {
  const startTime = Date.now();
  
  // Capture original res.json to log response
  const originalJson = res.json.bind(res);
  res.json = (body: any) => {
    const duration = Date.now() - startTime;
    console.log({
      method: req.method,
      path: req.path,
      statusCode: res.statusCode,
      duration: `${duration}ms`,
      userAgent: req.get('user-agent')
    });
    return originalJson(body);
  };
  
  next();
};

// ============================================
// CORS MIDDLEWARE
// ============================================

// Custom CORS configuration
export const corsMiddleware = (
  allowedOrigins: string[] = ['http://localhost:3000']
) => {
  return (req: Request, res: Response, next: NextFunction): void => {
    const origin = req.headers.origin;
    
    // Check if origin is allowed
    if (origin && allowedOrigins.includes(origin)) {
      res.setHeader('Access-Control-Allow-Origin', origin);
    }
    
    // Set other CORS headers
    res.setHeader('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, OPTIONS');
    res.setHeader('Access-Control-Allow-Headers', 'Content-Type, Authorization');
    res.setHeader('Access-Control-Allow-Credentials', 'true');
    res.setHeader('Access-Control-Max-Age', '86400'); // 24 hours
    
    // Handle preflight requests
    if (req.method === 'OPTIONS') {
      res.sendStatus(204);  // No Content
      return;
    }
    
    next();
  };
};

// ============================================
// CACHING MIDDLEWARE
// ============================================

interface CacheStore {
  [key: string]: {
    data: any;
    expiresAt: number;
  };
}

const cache: CacheStore = {};

// Cache middleware
export const cacheMiddleware = (durationSeconds: number = 60) => {
  return (req: Request, res: Response, next: NextFunction): void => {
    // Only cache GET requests
    if (req.method !== 'GET') {
      next();
      return;
    }
    
    const key = req.originalUrl || req.url;
    const cachedData = cache[key];
    
    // Return cached data if valid
    if (cachedData && Date.now() < cachedData.expiresAt) {
      console.log(`Cache HIT: ${key}`);
      res.json(cachedData.data);
      return;
    }
    
    console.log(`Cache MISS: ${key}`);
    
    // Override res.json to cache response
    const originalJson = res.json.bind(res);
    res.json = (body: any) => {
      // Store in cache
      cache[key] = {
        data: body,
        expiresAt: Date.now() + (durationSeconds * 1000)
      };
      return originalJson(body);
    };
    
    next();
  };
};

// Clear cache
export const clearCache = (): void => {
  Object.keys(cache).forEach(key => delete cache[key]);
  console.log('Cache cleared');
};

// ============================================
// REQUEST VALIDATION MIDDLEWARE
// ============================================

// Schema-based validation
interface ValidationSchema {
  body?: { [key: string]: (value: any) => boolean | string };
  params?: { [key: string]: (value: any) => boolean | string };
  query?: { [key: string]: (value: any) => boolean | string };
}

export const validate = (schema: ValidationSchema) => {
  return (req: Request, res: Response, next: NextFunction): void => {
    const errors: string[] = [];
    
    // Validate body
    if (schema.body) {
      for (const [key, validator] of Object.entries(schema.body)) {
        const result = validator(req.body[key]);
        if (result !== true) {
          errors.push(typeof result === 'string' ? result : `Invalid ${key}`);
        }
      }
    }
    
    // Validate params
    if (schema.params) {
      for (const [key, validator] of Object.entries(schema.params)) {
        const result = validator(req.params[key]);
        if (result !== true) {
          errors.push(typeof result === 'string' ? result : `Invalid param ${key}`);
        }
      }
    }
    
    // Validate query
    if (schema.query) {
      for (const [key, validator] of Object.entries(schema.query)) {
        const result = validator(req.query[key]);
        if (result !== true) {
          errors.push(typeof result === 'string' ? result : `Invalid query ${key}`);
        }
      }
    }
    
    if (errors.length > 0) {
      res.status(400).json({
        error: 'Validation failed',
        details: errors
      });
      return;
    }
    
    next();
  };
};

// Common validators
export const validators = {
  required: (value: any) => {
    return value !== undefined && value !== null && value !== '' ? true : 'Field is required';
  },
  
  email: (value: string) => {
    const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return regex.test(value) ? true : 'Invalid email format';
  },
  
  minLength: (min: number) => (value: string) => {
    return value && value.length >= min ? true : `Must be at least ${min} characters`;
  },
  
  maxLength: (max: number) => (value: string) => {
    return value && value.length <= max ? true : `Must be at most ${max} characters`;
  },
  
  numeric: (value: any) => {
    return !isNaN(Number(value)) ? true : 'Must be a number';
  },
  
  inRange: (min: number, max: number) => (value: any) => {
    const num = Number(value);
    return num >= min && num <= max ? true : `Must be between ${min} and ${max}`;
  },
  
  oneOf: (options: any[]) => (value: any) => {
    return options.includes(value) ? true : `Must be one of: ${options.join(', ')}`;
  }
};

// Usage example
/*
router.post('/register', validate({
  body: {
    username: validators.required,
    email: (value) => validators.required(value) && validators.email(value),
    password: validators.minLength(8)
  }
}), registerUser);
*/

// ============================================
// COMPRESSION MIDDLEWARE
// ============================================

export const compressionMiddleware = (
  req: Request,
  res: Response,
  next: NextFunction
): void => {
  // Check if client accepts gzip
  const acceptEncoding = req.headers['accept-encoding'] || '';
  
  if (!acceptEncoding.includes('gzip')) {
    next();
    return;
  }
  
  // Only compress JSON responses
  const originalJson = res.json.bind(res);
  res.json = (body: any) => {
    const jsonString = JSON.stringify(body);
    
    // Only compress if body is large enough
    if (jsonString.length > 1024) {  // 1KB threshold
      console.log('Compressing response');
      // In production, use actual gzip compression
      res.setHeader('Content-Encoding', 'gzip');
    }
    
    return originalJson(body);
  };
  
  next();
};

// ============================================
// SECURITY MIDDLEWARE
// ============================================

// Security headers
export const securityHeaders = (
  req: Request,
  res: Response,
  next: NextFunction
): void => {
  // Prevent clickjacking
  res.setHeader('X-Frame-Options', 'DENY');
  
  // Prevent MIME type sniffing
  res.setHeader('X-Content-Type-Options', 'nosniff');
  
  // Enable XSS filter
  res.setHeader('X-XSS-Protection', '1; mode=block');
  
  // Strict transport security (HTTPS only)
  res.setHeader('Strict-Transport-Security', 'max-age=31536000; includeSubDomains');
  
  // Content security policy
  res.setHeader(
    'Content-Security-Policy',
    "default-src 'self'; script-src 'self' 'unsafe-inline'"
  );
  
  next();
};

// Sanitize user input
export const sanitizeInput = (
  req: Request,
  res: Response,
  next: NextFunction
): void => {
  // Sanitize body
  if (req.body) {
    req.body = sanitizeObject(req.body);
  }
  
  // Sanitize query
  if (req.query) {
    req.query = sanitizeObject(req.query);
  }
  
  next();
};

function sanitizeObject(obj: any): any {
  if (typeof obj !== 'object' || obj === null) {
    return sanitizeString(obj);
  }
  
  const sanitized: any = Array.isArray(obj) ? [] : {};
  
  for (const key in obj) {
    if (obj.hasOwnProperty(key)) {
      sanitized[key] = sanitizeObject(obj[key]);
    }
  }
  
  return sanitized;
}

function sanitizeString(value: any): any {
  if (typeof value !== 'string') return value;
  
  // Remove potentially dangerous characters
  return value
    .replace(/<script[^>]*>.*?<\/script>/gi, '')  // Remove script tags
    .replace(/<[^>]+>/g, '')                      // Remove HTML tags
    .replace(/javascript:/gi, '')                  // Remove javascript: protocol
    .trim();
}
```

### Writing Express Code with TypeScript

```typescript
// ============================================
// COMPLETE EXPRESS APP WITH BEST PRACTICES
// ============================================

// src/config/env.ts - Environment configuration
export interface AppConfig {
  port: number;
  nodeEnv: 'development' | 'production' | 'test';
  dbUrl: string;
  jwtSecret: string;
  corsOrigins: string[];
}

export const config: AppConfig = {
  port: parseInt(process.env.PORT || '3000'),
  nodeEnv: (process.env.NODE_ENV as any) || 'development',
  dbUrl: process.env.DATABASE_URL || 'mongodb://localhost:27017/myapp',
  jwtSecret: process.env.JWT_SECRET || 'your-secret-key',
  corsOrigins: process.env.CORS_ORIGINS?.split(',') || ['http://localhost:3000']
};

// ============================================
// src/utils/logger.ts - Logging utility
// ============================================

export enum LogLevel {
  ERROR = 'ERROR',
  WARN = 'WARN',
  INFO = 'INFO',
  DEBUG = 'DEBUG'
}

class Logger {
  private logLevel: LogLevel;
  
  constructor(level: LogLevel = LogLevel.INFO) {
    this.logLevel = level;
  }
  
  private shouldLog(level: LogLevel): boolean {
    const levels = [LogLevel.ERROR, LogLevel.WARN, LogLevel.INFO, LogLevel.DEBUG];
    return levels.indexOf(level) <= levels.indexOf(this.logLevel);
  }
  
  private formatMessage(level: LogLevel, message: string, meta?: any): string {
    const timestamp = new Date().toISOString();
    const metaStr = meta ? ` | ${JSON.stringify(meta)}` : '';
    return `[${timestamp}] [${level}] ${message}${metaStr}`;
  }
  
  error(message: string, error?: Error | any): void {
    if (this.shouldLog(LogLevel.ERROR)) {
      console.error(this.formatMessage(LogLevel.ERROR, message, {
        error: error?.message,
        stack: error?.stack
      }));
    }
  }
  
  warn(message: string, meta?: any): void {
    if (this.shouldLog(LogLevel.WARN)) {
      console.warn(this.formatMessage(LogLevel.WARN, message, meta));
    }
  }
  
  info(message: string, meta?: any): void {
    if (this.shouldLog(LogLevel.INFO)) {
      console.log(this.formatMessage(LogLevel.INFO, message, meta));
    }
  }
  
  debug(message: string, meta?: any): void {
    if (this.shouldLog(LogLevel.DEBUG)) {
      console.log(this.formatMessage(LogLevel.DEBUG, message, meta));
    }
  }
}

export const logger = new Logger(
  process.env.NODE_ENV === 'production' ? LogLevel.INFO : LogLevel.DEBUG
);

// ============================================
// src/services/userService.ts - Business logic
// ============================================

import { logger } from '../utils/logger';

export interface User {
  id: number;
  username: string;
  email: string;
  passwordHash: string;
  role: 'user' | 'admin';
  createdAt: Date;
  updatedAt: Date;
}

export interface CreateUserDTO {
  username: string;
  email: string;
  password: string;
}

export interface UpdateUserDTO {
  username?: string;
  email?: string;
  password?: string;
}

// Service layer - contains business logic
export class UserService {
  private users: User[] = [];
  private nextId = 1;
  
  async findAll(): Promise<User[]> {
    logger.info('Fetching all users');
    // Exclude password hashes from response
    return this.users.map(user => this.sanitizeUser(user));
  }
  
  async findById(id: number): Promise<User | null> {
    logger.info('Fetching user by ID', { id });
    const user = this.users.find(u => u.id === id);
    return user ? this.sanitizeUser(user) : null;
  }
  
  async findByEmail(email: string): Promise<User | null> {
    logger.debug('Fetching user by email', { email });
    const user = this.users.find(u => u.email === email);
    return user || null;
  }
  
  async create(data: CreateUserDTO): Promise<User> {
    logger.info('Creating new user', { username: data.username });
    
    // Check if user exists
    const existing = await this.findByEmail(data.email);
    if (existing) {
      logger.warn('User creation failed - email exists', { email: data.email });
      throw new Error('User with this email already exists');
    }
    
    // Hash password (simplified - use bcrypt in production)
    const passwordHash = this.hashPassword(data.password);
    
    const newUser: User = {
      id: this.nextId++,
      username: data.username,
      email: data.email,
      passwordHash,
      role: 'user',
      createdAt: new Date(),
      updatedAt: new Date()
    };
    
    this.users.push(newUser);
    logger.info('User created successfully', { id: newUser.id });
    
    return this.sanitizeUser(newUser);
  }
  
  async update(id: number, data: UpdateUserDTO): Promise<User | null> {
    logger.info('Updating user', { id });
    
    const index = this.users.findIndex(u => u.id === id);
    if (index === -1) {
      logger.warn('User update failed - not found', { id });
      return null;
    }
    
    const updates: Partial<User> = {
      ...data,
      updatedAt: new Date()
    };
    
    if (data.password) {
      updates.passwordHash = this.hashPassword(data.password);
      delete (updates as any).password;
    }
    
    this.users[index] = { ...this.users[index], ...updates };
    logger.info('User updated successfully', { id });
    
    return this.sanitizeUser(this.users[index]);
  }
  
  async delete(id: number): Promise<boolean> {
    logger.info('Deleting user', { id });
    
    const index = this.users.findIndex(u => u.id === id);
    if (index === -1) {
      logger.warn('User deletion failed - not found', { id });
      return false;
    }
    
    this.users.splice(index, 1);
    logger.info('User deleted successfully', { id });
    
    return true;
  }
  
  async verifyCredentials(email: string, password: string): Promise<User | null> {
    logger.debug('Verifying user credentials', { email });
    
    const user = await this.findByEmail(email);
    if (!user) {
      logger.warn('Credential verification failed - user not found', { email });
      return null;
    }
    
    const isValid = this.verifyPassword(password, (user as any).passwordHash);
    if (!isValid) {
      logger.warn('Credential verification failed - invalid password', { email });
      return null;
    }
    
    logger.info('Credentials verified successfully', { email });
    return this.sanitizeUser(user as any);
  }
  
  // Helper methods
  private sanitizeUser(user: User): User {
    const { passwordHash, ...sanitized } = user as any;
    return sanitized;
  }
  
  private hashPassword(password: string): string {
    // Use bcrypt in production
    return `hashed_${password}`;
  }
  
  private verifyPassword(password: string, hash: string): boolean {
    // Use bcrypt.compare in production
    return hash === `hashed_${password}`;
  }
}

// Export singleton instance
export const userService = new UserService();

// ============================================
// src/controllers/authController.ts
// ============================================

import { Request, Response } from 'express';
import { userService } from '../services/userService';
import { logger } from '../utils/logger';
import { AppError } from '../middleware/errorHandler';

interface LoginRequest extends Request {
  body: {
    email: string;
    password: string;
  };
}

interface RegisterRequest extends Request {
  body: {
    username: string;
    email: string;
    password: string;
  };
}

// Login controller
export const login = async (req: LoginRequest, res: Response): Promise<void> => {
  try {
    const { email, password } = req.body;
    
    // Verify credentials
    const user = await userService.verifyCredentials(email, password);
    
    if (!user) {
      throw new AppError(401, 'Invalid email or password');
    }
    
    // Generate token (simplified - use JWT in production)
    const token = `token_${user.id}_${Date.now()}`;
    
    res.json({
      success: true,
      data: {
        user,
        token
      },
      message: 'Login successful'
    });
  } catch (error) {
    logger.error('Login failed', error);
    throw error;
  }
};

// Register controller
export const register = async (req: RegisterRequest, res: Response): Promise<void> => {
  try {
    const { username, email, password } = req.body;
    
    // Create user
    const user = await userService.create({ username, email, password });
    
    // Generate token
    const token = `token_${user.id}_${Date.now()}`;
    
    res.status(201).json({
      success: true,
      data: {
        user,
        token
      },
      message: 'Registration successful'
    });
  } catch (error) {
    logger.error('Registration failed', error);
    throw error;
  }
};

// Logout controller
export const logout = async (req: AuthRequest, res: Response): Promise<void> => {
  // In production, invalidate token (add to blacklist, etc.)
  logger.info('User logged out', { userId: req.user?.id });
  
  res.json({
    success: true,
    message: 'Logout successful'
  });
};

// ============================================
// COMPLETE APPLICATION SETUP
// ============================================

// src/index.ts - Entry point
import app from './app';
import { config } from './config/env';
import { logger } from './utils/logger';

const startServer = (): void => {
  try {
    app.listen(config.port, () => {
      logger.info(`Server started successfully`, {
        port: config.port,
        environment: config.nodeEnv
      });
    });
  } catch (error) {
    logger.error('Failed to start server', error);
    process.exit(1);
  }
};

// Handle uncaught exceptions
process.on('uncaughtException', (error: Error) => {
  logger.error('Uncaught exception', error);
  process.exit(1);
});

// Handle unhandled promise rejections
process.on('unhandledRejection', (reason: any) => {
  logger.error('Unhandled rejection', reason);
  process.exit(1);
});

// Graceful shutdown
process.on('SIGTERM', () => {
  logger.info('SIGTERM received, shutting down gracefully');
  process.exit(0);
});

startServer();
```

---

## Advanced Topics and Best Practices

### Decorators

```typescript
// ============================================
// TYPESCRIPT DECORATORS
// ============================================

// Enable decorators in tsconfig.json:
// "experimentalDecorators": true,
// "emitDecoratorMetadata": true

// ============================================
// CLASS DECORATORS
// ============================================

// Class decorator - receives constructor as parameter
function Component(config: { selector: string }) {
  return function <T extends { new(...args: any[]): {} }>(constructor: T) {
    // Add metadata to class
    return class extends constructor {
      selector = config.selector;
      
      log() {
        console.log(`Component: ${this.selector}`);
      }
    };
  };
}

@Component({ selector: 'app-user' })
class UserComponent {
  name = 'User Component';
}

const comp = new UserComponent();
(comp as any).log();  // "Component: app-user"

// ============================================
// METHOD DECORATORS
// ============================================

// Method decorator - logs execution time
function LogExecutionTime(
  target: any,
  propertyKey: string,
  descriptor: PropertyDescriptor
) {
  const originalMethod = descriptor.value;
  
  descriptor.value = async function (...args: any[]) {
    const start = Date.now();
    const result = await originalMethod.apply(this, args);
    const duration = Date.now() - start;
    console.log(`${propertyKey} executed in ${duration}ms`);
    return result;
  };
  
  return descriptor;
}

class DataService {
  @LogExecutionTime
  async fetchData(id: number): Promise<any> {
    // Simulate async operation
    await new Promise(resolve => setTimeout(resolve, 100));
    return { id, data: 'Some data' };
  }
}

const service = new DataService();
service.fetchData(1);  // Logs: "fetchData executed in ~100ms"

// Cache method results
function Memoize(
  target: any,
  propertyKey: string,
  descriptor: PropertyDescriptor
) {
  const originalMethod = descriptor.value;
  const cache = new Map<string, any>();
  
  descriptor.value = function (...args: any[]) {
    const key = JSON.stringify(args);
    
    if (cache.has(key)) {
      console.log('Cache hit');
      return cache.get(key);
    }
    
    console.log('Cache miss');
    const result = originalMethod.apply(this, args);
    cache.set(key, result);
    return result;
  };
  
  return descriptor;
}

class Calculator {
  @Memoize
  expensiveCalculation(n: number): number {
    console.log('Performing calculation...');
    return n * n;
  }
}

const calc = new Calculator();
calc.expensiveCalculation(5);  // Cache miss, performs calculation
calc.expensiveCalculation(5);  // Cache hit, returns cached result

// ============================================
// PROPERTY DECORATORS
// ============================================

// Property decorator - validate value
function Min(minValue: number) {
  return function (target: any, propertyKey: string) {
    let value: number;
    
    const getter = () => value;
    const setter = (newValue: number) => {
      if (newValue < minValue) {
        throw new Error(`${propertyKey} must be at least ${minValue}`);
      }
      value = newValue;
    };
    
    Object.defineProperty(target, propertyKey, {
      get: getter,
      set: setter,
      enumerable: true,
      configurable: true
    });
  };
}

class Product {
  @Min(0)
  price: number = 0;
  
  @Min(1)
  quantity: number = 1;
}

const product = new Product();
product.price = 50;    // OK
// product.price = -10;  // Error: price must be at least 0

// ============================================
// PARAMETER DECORATORS
// ============================================

// Parameter decorator - mark required parameters
function Required(target: any, propertyKey: string, parameterIndex: number) {
  const existingRequiredParameters: number[] = 
    Reflect.getOwnMetadata('required', target, propertyKey) || [];
  
  existingRequiredParameters.push(parameterIndex);
  
  Reflect.defineMetadata(
    'required',
    existingRequiredParameters,
    target,
    propertyKey
  );
}

function ValidateRequired(
  target: any,
  propertyKey: string,
  descriptor: PropertyDescriptor
) {
  const originalMethod = descriptor.value;
  
  descriptor.value = function (...args: any[]) {
    const requiredParameters: number[] = 
      Reflect.getOwnMetadata('required', target, propertyKey) || [];
    
    for (const index of requiredParameters) {
      if (args[index] === undefined || args[index] === null) {
        throw new Error(`Parameter at index ${index} is required`);
      }
    }
    
    return originalMethod.apply(this, args);
  };
  
  return descriptor;
}

class UserService {
  @ValidateRequired
  createUser(@Required username: string, @Required email: string, age?: number) {
    return { username, email, age };
  }
}

const userService = new UserService();
userService.createUser('john', 'john@example.com');  // OK
// userService.createUser('john', null);  // Error: Parameter required

// ============================================
// REAL-WORLD: API ROUTE DECORATORS
// ============================================

import 'reflect-metadata';

// HTTP method decorators
function Get(path: string = '') {
  return function (target: any, propertyKey: string) {
    Reflect.defineMetadata('path', path, target, propertyKey);
    Reflect.defineMetadata('method', 'GET', target, propertyKey);
  };
}

function Post(path: string = '') {
  return function (target: any, propertyKey: string) {
    Reflect.defineMetadata('path', path, target, propertyKey);
    Reflect.defineMetadata('method', 'POST', target, propertyKey);
  };
}

// Controller decorator
function Controller(basePath: string = '') {
  return function <T extends { new(...args: any[]): {} }>(constructor: T) {
    Reflect.defineMetadata('basePath', basePath, constructor);
    return constructor;
  };
}

// Usage
@Controller('/api/users')
class UserController {
  @Get('/')
  getAll() {
    return { users: [] };
  }
  
  @Get('/:id')
  getById() {
    return { user: {} };
  }
  
  @Post('/')
  create() {
    return { created: true };
  }
}

// Register routes automatically
function registerRoutes(controller: any, app: any) {
  const basePath = Reflect.getMetadata('basePath', controller);
  const prototype = controller.prototype;
  
  for (const methodName of Object.getOwnPropertyNames(prototype)) {
    const method = Reflect.getMetadata('method', prototype, methodName);
    const path = Reflect.getMetadata('path', prototype, methodName);
    
    if (method && path !== undefined) {
      const fullPath = basePath + path;
      console.log(`Registered ${method} ${fullPath}`);
      // app[method.toLowerCase()](fullPath, prototype[methodName]);
    }
  }
}
```

### Async/Await Patterns

```typescript
// ============================================
// ASYNC/AWAIT PATTERNS
// ============================================

// Sequential execution
async function sequentialFetch() {
  console.log('Fetching users...');
  const users = await fetchUsers();  // Wait for completion
  
  console.log('Fetching posts...');
  const posts = await fetchPosts();  // Wait for completion
  
  return { users, posts };
}

// Parallel execution
async function parallelFetch() {
  console.log('Fetching data in parallel...');
  
  // Start all promises at once
  const [users, posts, comments] = await Promise.all([
    fetchUsers(),
    fetchPosts(),
    fetchComments()
  ]);
  
  return { users, posts, comments };
}

// Race condition - first to complete wins
async function fetchWithTimeout<T>(promise: Promise<T>, timeout: number): Promise<T> {
  const timeoutPromise = new Promise<T>((_, reject) => {
    setTimeout(() => reject(new Error('Timeout')), timeout);
  });
  
  return Promise.race([promise, timeoutPromise]);
}

// Usage
try {
  const data = await fetchWithTimeout(fetchUsers(), 5000);  // 5 second timeout
  console.log(data);
} catch (error) {
  console.error('Request timed out or failed');
}

// Handling multiple promises with different results
async function fetchAllSettled() {
  const results = await Promise.allSettled([
    fetchUsers(),
    fetchPosts(),
    Promise.reject(new Error('Failed'))
  ]);
  
  results.forEach((result, index) => {
    if (result.status === 'fulfilled') {
      console.log(`Promise ${index} succeeded:`, result.value);
    } else {
      console.log(`Promise ${index} failed:`, result.reason);
    }
  });
}

// Retry logic
async function retryOperation<T>(
  operation: () => Promise<T>,
  maxRetries: number = 3,
  delay: number = 1000
): Promise<T> {
  let lastError: Error;
  
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await operation();
    } catch (error) {
      lastError = error as Error;
      console.log(`Attempt ${attempt} failed, retrying...`);
      
      if (attempt < maxRetries) {
        await new Promise(resolve => setTimeout(resolve, delay));
      }
    }
  }
  
  throw lastError!;
}

// Usage
const data = await retryOperation(() => fetchUsers(), 3, 2000);

// Async iteration
async function* asyncGenerator() {
  yield await fetchUsers();
  yield await fetchPosts();
  yield await fetchComments();
}

async function consumeAsyncGenerator() {
  for await (const data of asyncGenerator()) {
    console.log('Received:', data);
  }
}

// Helper functions (mock implementations)
function fetchUsers(): Promise<any[]> {
  return new Promise(resolve => setTimeout(() => resolve([]), 1000));
}

function fetchPosts(): Promise<any[]> {
  return new Promise(resolve => setTimeout(() => resolve([]), 1000));
}

function fetchComments(): Promise<any[]> {
  return new Promise(resolve => setTimeout(() => resolve([]), 1000));
}
```

### Best Practices

```typescript
// ============================================
// TYPESCRIPT BEST PRACTICES
// ============================================

// 1. Use strict null checks
function processUser(user: User | null): void {
  // Bad
  // console.log(user.name);  // Error with strictNullChecks
  
  // Good
  if (user) {
    console.log(user.name);  // Type narrowed to User
  }
  
  // Or use optional chaining
  console.log(user?.name);
}

// 2. Prefer interfaces for object shapes, types for unions
interface UserData {  // Good for objects
  id: number;
  name: string;
}

type Status = 'active' | 'inactive' | 'pending';  // Good for unions

// 3. Use readonly for immutability
interface Config {
  readonly apiUrl: string;
  readonly timeout: number;
}

const config: Readonly<Config> = {
  apiUrl: 'https://api.example.com',
  timeout: 5000
};

// config.apiUrl = 'new-url';  // Error

// 4. Leverage union types instead of enums when possible
type Direction = 'north' | 'south' | 'east' | 'west';

function move(direction: Direction) {
  // TypeScript ensures only valid values
}

// 5. Use const assertions for literal types
const colors = ['red', 'green', 'blue'] as const;
type Color = typeof colors[number];  // 'red' | 'green' | 'blue'

// 6. Type guards for type safety
function isUser(obj: any): obj is User {
  return obj && typeof obj.id === 'number' && typeof obj.name === 'string';
}

function processData(data: unknown) {
  if (isUser(data)) {
    console.log(data.name);  // TypeScript knows it's User
  }
}

// 7. Use generics for reusable code
function wrapInArray<T>(value: T): T[] {
  return [value];
}

const numbers = wrapInArray(42);        // number[]
const strings = wrapInArray('hello');   // string[]

// 8. Avoid any, use unknown for truly unknown types
function processUnknown(value: unknown) {
  // Must narrow type before using
  if (typeof value === 'string') {
    console.log(value.toUpperCase());
  }
}

// 9. Use discriminated unions
type Success = { status: 'success'; data: any };
type Error = { status: 'error'; message: string };
type Result = Success | Error;

function handleResult(result: Result) {
  if (result.status === 'success') {
    console.log(result.data);  // TypeScript knows it's Success
  } else {
    console.log(result.message);  // TypeScript knows it's Error
  }
}

// 10. Use utility types
type PartialUser = Partial<User>;       // All properties optional
type RequiredUser = Required<User>;     // All properties required
type UserPreview = Pick<User, 'id' | 'name'>;  // Select properties
type UserWithoutId = Omit<User, 'id'>;  // Exclude properties
```

---

## Real-World Projects

### Project 1: RESTful API with CRUD Operations

```typescript
// ============================================
// COMPLETE REST API PROJECT
// ============================================

// Project structure:
// src/
//   ├── config/
//   │   └── database.ts
//   ├── models/
//   │   └── User.ts
//   ├── controllers/
//   │   └── userController.ts
//   ├── routes/
//   │   └── userRoutes.ts
//   ├── middleware/
//   │   ├── auth.ts
//   │   └── validation.ts
//   ├── services/
//   │   └── userService.ts
//   ├── utils/
//   │   └── logger.ts
//   ├── app.ts
//   └── index.ts

// Example: Complete user management system
// (Code already shown in previous sections)
```

### Project 2: Authentication System with JWT

```typescript
// ============================================
// JWT AUTHENTICATION SYSTEM
// ============================================

// Install: npm install jsonwebtoken bcrypt
// npm install --save-dev @types/jsonwebtoken @types/bcrypt

import jwt from 'jsonwebtoken';
import bcrypt from 'bcrypt';
import { Request, Response, NextFunction } from 'express';

// JWT configuration
const JWT_SECRET = process.env.JWT_SECRET || 'your-secret-key';
const JWT_EXPIRES_IN = '7d';

// Generate JWT token
export function generateToken(payload: object): string {
  return jwt.sign(payload, JWT_SECRET, {
    expiresIn: JWT_EXPIRES_IN
  });
}

// Verify JWT token
export function verifyToken(token: string): any {
  try {
    return jwt.verify(token, JWT_SECRET);
  } catch (error) {
    throw new Error('Invalid token');
  }
}

// Hash password
export async function hashPassword(password: string): Promise<string> {
  const saltRounds = 10;
  return await bcrypt.hash(password, saltRounds);
}

// Compare password
export async function comparePassword(
  password: string,
  hash: string
): Promise<boolean> {
  return await bcrypt.compare(password, hash);
}

// Authentication middleware
export const authenticateJWT = (
  req: AuthRequest,
  res: Response,
  next: NextFunction
): void => {
  const authHeader = req.headers.authorization;
  
  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    res.status(401).json({ error: 'No token provided' });
    return;
  }
  
  const token = authHeader.substring(7);  // Remove 'Bearer '
  
  try {
    const decoded = verifyToken(token);
    req.user = decoded;
    next();
  } catch (error) {
    res.status(403).json({ error: 'Invalid or expired token' });
  }
};

// Login endpoint
export const loginWithJWT = async (req: Request, res: Response): Promise<void> => {
  const { email, password } = req.body;
  
  // Find user (replace with actual database query)
  const user = await userService.findByEmail(email);
  
  if (!user) {
    res.status(401).json({ error: 'Invalid credentials' });
    return;
  }
  
  // Verify password
  const isValid = await comparePassword(password, (user as any).passwordHash);
  
  if (!isValid) {
    res.status(401).json({ error: 'Invalid credentials' });
    return;
  }
  
  // Generate token
  const token = generateToken({
    id: user.id,
    email: user.email,
    role: (user as any).role
  });
  
  res.json({
    success: true,
    data: {
      user,
      token
    }
  });
};

// Register endpoint
export const registerWithJWT = async (req: Request, res: Response): Promise<void> => {
  const { username, email, password } = req.body;
  
  // Hash password
  const passwordHash = await hashPassword(password);
  
  // Create user (replace with actual database operation)
  const user = await userService.create({
    username,
    email,
    password: passwordHash
  });
  
  // Generate token
  const token = generateToken({
    id: user.id,
    email: user.email,
    role: 'user'
  });
  
  res.status(201).json({
    success: true,
    data: {
      user,
      token
    }
  });
};
```

---

## Conclusion

Congratulations! 🎉 You've completed this comprehensive TypeScript and Express.js mastery tutorial. You've learned:

✅ **TypeScript Fundamentals**
- Basic types, interfaces, and type annotations
- Advanced types: unions, intersections, utility types
- Generics and type constraints
- Template literal types and mapped types

✅ **Object-Oriented Programming**
- Classes, inheritance, and polymorphism
- Abstract classes and interfaces
- Access modifiers and encapsulation
- Composition and mixins

✅ **Design Patterns**
- Creational: Singleton, Factory, Builder
- Structural: Adapter, Facade, Decorator, Proxy
- Behavioral: Strategy, Observer, Command, State, Chain of Responsibility, Template Method

✅ **Express.js with TypeScript**
- Project setup and configuration
- Type-safe routing and controllers
- Middleware patterns and implementation
- Error handling and validation
- Authentication and authorization

✅ **Advanced Topics**
- Decorators and metadata
- Async/await patterns
- Best practices and code organization
- Real-world project structure

### Next Steps

1. **Build Real Projects**: Apply these concepts in actual applications
2. **Explore Advanced Libraries**: Prisma, TypeORM, NestJS
3. **Learn Testing**: Jest, Supertest for API testing
4. **Master Databases**: PostgreSQL, MongoDB integration
5. **Deploy Applications**: Docker, CI/CD, cloud platforms
6. **Study Source Code**: Read open-source TypeScript projects
7. **Stay Updated**: Follow TypeScript and Node.js releases

### Resources for Further Learning

- **Official Documentation**:
  - [TypeScript Handbook](https://www.typescriptlang.org/docs/)
  - [Express.js Guide](https://expressjs.com/)
  
- **Advanced Topics**:
  - TypeScript Deep Dive
  - Design Patterns in TypeScript
  - Clean Code principles
  
- **Community**:
  - TypeScript Discord
  - Stack Overflow
  - GitHub discussions

### Final Thoughts

Mastery comes with practice. Build projects, make mistakes, learn from them, and keep coding. TypeScript and Express.js are powerful tools that, when combined, enable you to build robust, scalable, and maintainable applications.

Happy coding! 🚀

---

**Author**: Ninja Developer Mastery Collection  
**Last Updated**: 2025  
**License**: MIT  

*This tutorial is part of the comprehensive developer education series aimed at achieving technical mastery through hands-on learning and real-world examples.*
```

### Type-Safe Request and Response

```typescript
// ============================================
// src/types/express.ts - CUSTOM TYPES
// ============================================

import { Request, Response } from 'express';

// Extend Express Request with custom properties
export interface AuthRequest extends Request {
  user?: {
    id: number;
    username: string;
    email: string;
    role: string;
  };
}

// Type-safe request body
export interface CreateUserRequest extends Request {
  body: {
    username: string;
    email: string;
    password: string;
    age?: number;
  };
}

// Type-safe request params
export interface UserIdRequest extends Request {
  params: {
    id: string;
  };
}

// Type-safe query parameters
export interface SearchRequest extends Request {
  query: {
    q?: string;
    limit?: string;
    page?: string;
    sort?: 'asc' | 'desc';
  };
}

// Response data types
export interface ApiResponse<T> {
  success: boolean;
  data?: T;
  error?: string;
  message?: string;
}

export interface PaginatedResponse<T> {
  data: T[];
  pagination: {
    page: number;
    limit: number;
    total: number;
    totalPages: number;
  };
}

// ============================================
// src/controllers/userController.ts
// ============================================

import { Response } from 'express';
import {
  CreateUserRequest,
  UserIdRequest,
  SearchRequest,
  ApiResponse,
  PaginatedResponse
} from '../types/express';

// User model/interface
interface User {
  id: number;
  username: string;
  email: string;
  age?: number;
  createdAt: Date;
}

// In-memory user storage (replace with database)
let users: User[] = [
  {
    id: 1,
    username: 'alice',
    email: 'alice@example.com',
    age: 30,
    createdAt: new Date()
  },
  {
    id: 2,
    username: 'bob',
    email: 'bob@example.com',
    age: 25,
    createdAt: new Date()
  }
];

let nextId = 3;

// ============================================
// CONTROLLER FUNCTIONS
// ============================================

// Get all users with pagination
export const getAllUsers = (
  req: SearchRequest,
  res: Response<PaginatedResponse<User>>
): void => {
  // Parse query parameters with defaults
  const page = parseInt(req.query.page || '1');
  const limit = parseInt(req.query.limit || '10');
  const sort = req.query.sort || 'asc';
  
  // Sort users
  const sortedUsers = [...users].sort((a, b) => {
    return sort === 'asc' ? a.id - b.id : b.id - a.id;
  });
  
  // Paginate
  const startIndex = (page - 1) * limit;
  const endIndex = startIndex + limit;
  const paginatedUsers = sortedUsers.slice(startIndex, endIndex);
  
  // Calculate pagination metadata
  const totalPages = Math.ceil(users.length / limit);
  
  res.json({
    data: paginatedUsers,
    pagination: {
      page,
      limit,
      total: users.length,
      totalPages
    }
  });
};

// Get user by ID
export const getUserById = (
  req: UserIdRequest,
  res: Response<ApiResponse<User>>
): void => {
  const userId = parseInt(req.params.id);
  
  // Validate ID
  if (isNaN(userId)) {
    res.status(400).json({
      success: false,
      error: 'Invalid user ID'
    });
    return;
  }
  
  // Find user
  const user = users.find(u => u.id === userId);
  
  if (!user) {
    res.status(404).json({
      success: false,
      error: 'User not found'
    });
    return;
  }
  
  res.json({
    success: true,
    data: user
  });
};

// Create new user
export const createUser = (
  req: CreateUserRequest,
  res: Response<ApiResponse<User>>
): void => {
  const { username, email, password, age } = req.body;
  
  // Validation
  if (!username || !email || !password) {
    res.status(400).json({
      success: false,
      error: 'Missing required fields: username, email, password'
    });
    return;
  }
  
  // Check if user already exists
  const existingUser = users.find(u => u.email === email);
  if (existingUser) {
    res.status(409).json({  // 409 Conflict
      success: false,
      error: 'User with this email already exists'
    });
    return;
  }
  
  // Create new user
  const newUser: User = {
    id: nextId++,
    username,
    email,
    age,
    createdAt: new Date()
  };
  
  users.push(newUser);
  
  res.status(201).json({
    success: true,
    data: newUser,
    message: 'User created successfully'
  });
};

// Update user
export const updateUser = (
  req: UserIdRequest & CreateUserRequest,
  res: Response<ApiResponse<User>>
): void => {
  const userId = parseInt(req.params.id);
  const updates = req.body;
  
  // Validate ID
  if (isNaN(userId)) {
    res.status(400).json({
      success: false,
      error: 'Invalid user ID'
    });
    return;
  }
  
  // Find user
  const userIndex = users.findIndex(u => u.id === userId);
  
  if (userIndex === -1) {
    res.status(404).json({
      success: false,
      error: 'User not found'
    });
    return;
  }
  
  // Update user (merge updates)
  users[userIndex] = {
    ...users[userIndex],
    ...updates,
    id: userId  // Prevent ID change
  };
  
  res.json({
    success: true,
    data: users[userIndex],
    message: 'User updated successfully'
  });
};

// Delete user
export const deleteUser = (
  req: UserIdRequest,
  res: Response<ApiResponse<null>>
): void => {
  const userId = parseInt(req.params.id);
  
  // Validate ID
  if (isNaN(userId)) {
    res.status(400).json({
      success: false,
      error: 'Invalid user ID'
    });
    return;
  }
  
  // Find user
  const userIndex = users.findIndex(u => u.id === userId);
  
  if (userIndex === -1) {
    res.status(404).json({
      success: false,
      error: 'User not found'
    });
    return;
  }
  
  // Delete user
  users.splice(userIndex, 1);
  
  res.json({
    success: true,
    message: 'User deleted successfully'
  });
};

// Search users
export const searchUsers = (
  req: SearchRequest,
  res: Response<ApiResponse<User[]>>
): void => {
  const searchQuery = req.query.q?.toLowerCase() || '';
  
  if (!searchQuery) {
    res.status(400).json({
      success: false,
      error: 'Search query required'
    });
    return;
  }
  
  // Search by username or email
  const results = users.filter(u =>
    u.username.toLowerCase().includes(searchQuery) ||
    u.email.toLowerCase().includes(searchQuery)
  );
  
  res.json({
    success: true,
    data: results
  });
};
```

### Routing and Middleware

```typescript
// ============================================
// src/routes/userRoutes.ts - ROUTER
// ============================================

import { Router } from 'express';
import {
  getAllUsers,
  getUserById,
  createUser,
  updateUser,
  deleteUser,
  searchUsers
} from '../controllers/userController';
import { validateUser } from '../middleware/validation';
import { authenticate } from '../middleware/auth';
import { rateLimiter } from '../middleware/rateLimiter';

const router = Router();

// Apply middleware to all routes in this router
router.use(rateLimiter);  // Rate limiting

// Public routes
router.get('/search', searchUsers);        // Search users
router.get('/', getAllUsers);              // Get all users
router.get('/:id', getUserById);           // Get user by ID

// Protected routes (require authentication)
router.post('/', authenticate, validateUser, createUser);     // Create user
router.put('/:id', authenticate, validateUser, updateUser);   // Update user
router.delete('/:id', authenticate, deleteUser);              // Delete user

export default router;

// ============================================
// src/middleware/validation.ts - VALIDATION
// ============================================

import { Response, NextFunction } from 'express';
import { CreateUserRequest } from '../types/express';

// Validation middleware for user creation
export const validateUser = (
  req: CreateUserRequest,
  res: Response,
  next: NextFunction
): void => {
  const { username, email, password, age } = req.body;
  
  // Validate username
  if (!username || username.length < 3) {
    res.status(400).json({
      error: 'Username must be at least 3 characters'
    });
    return;
  }
  
  // Validate email format
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  if (!email || !emailRegex.test(email)) {
    res.status(400).json({
      error: 'Invalid email format'
    });
    return;
  }
  
  // Validate password
  if (!password || password.length < 6) {
    res.status(400).json({
      error: 'Password must be at least 6 characters'
    });
    return;
  }
  
  // Validate age if provided
  if (age !== undefined && (age < 0 || age > 150)) {
    res.status(400).json({
      error: 'Age must be between 0 and 150'
    });
    return;
  }
  
  // Validation passed, continue to next middleware
  next();
};

// ============================================
// src/middleware/auth.ts - AUTHENTICATION
// ============================================

import { Response, NextFunction } from 'express';
import { AuthRequest } from '../types/express';

// Simple authentication middleware (replace with JWT in production)
export const authenticate = (
  req: AuthRequest,
  res: Response,
  next: NextFunction
): void => {
  // Get auth token from header
  const token = req.headers.authorization;
  
  if (!token) {
    res.status(401).json({  // 401 Unauthorized
      error: 'No authentication token provided'
    });
    return;
  }
  
  // Verify token (simplified - use JWT in production)
  if (token !== 'Bearer valid-token') {
    res.status(403).json({  // 403 Forbidden
      error: 'Invalid authentication token'
    });
    return;
  }
  
  // Attach user info to request (from token payload)
  req.user = {
    id: 1,
    username: 'admin',
    email: 'admin@example.com',
    role: 'admin'
  };
  
  // Continue to next middleware
  next();
};

// Role-based authorization middleware
export const authorize = (...roles: string[]) => {
  return (req: AuthRequest, res: Response, next: NextFunction): void => {
    if (!req.user) {
      res.status(401).json({
        error: 'Not authenticated'
      });
      return;
    }
    
    if (!roles.includes(req.user.role)) {
      res.status(403).json({
        error: 'Insufficient permissions'
      });
      return;
    }
    
    next();
  };
};

// ============================================
// src/middleware/rateLimiter.ts - RATE LIMITING
// ============================================

import { Request, Response, NextFunction } from 'express';

// Simple in-memory rate limiter
interface RateLimitStore {
  [key: string]: {
    count: number;
    resetTime: number;
  };
}

const store: RateLimitStore = {};
const WINDOW_MS = 60 * 1000;  // 1 minute window
const MAX_REQUESTS = 100;      // Max requests per window

export const rateLimiter = (
  req: Request,
  res: Response,
  next: NextFunction
): void => {
  // Use IP address as identifier
  const identifier = req.ip || req.socket.remoteAddress || 'unknown';
  const now = Date.now();
  
  // Initialize or get existing record
  if (!store[identifier] || now > store[identifier].resetTime) {
    store[identifier] = {
      count: 1,
      resetTime: now + WINDOW_MS
    };
    next();
    return;
  }
  
  // Increment count
  store[identifier].count++;
  
  // Check if limit exceeded
  if (store[identifier].count > MAX_REQUESTS) {
    res.status(429).json({  // 429 Too Many Requests
      error: 'Too many requests, please try again later',
      retryAfter: Math.ceil((store[identifier].resetTime - now) / 1000)
    });
    return;
  }
  
  // Set rate limit headers
  res.setHeader('X-RateLimit-Limit', MAX_REQUESTS);
  res.setHeader('X-RateLimit-Remaining', MAX_REQUESTS - store[identifier].count);
  res.setHeader('X-RateLimit-Reset', store[identifier].resetTime);
  
  next();
};

// ============================================
// src/middleware/errorHandler.ts - ERROR HANDLING
// ============================================

import { Request, Response, NextFunction } from 'express';

// Custom error class
export class AppError extends Error {
  constructor(
    public statusCode: number,
    public message: string,
    public isOperational: boolean = true
  ) {
    super(message);
    Object.setPrototypeOf(this, AppError.prototype);
  }
}

// Async error wrapper
export const asyncHandler = (
  fn: (req: Request, res: Response, next: NextFunction) => Promise<any>
) => {
  return (req: Request, res: Response, next: NextFunction) => {
    Promise.resolve(fn(req, res, next)).catch(next);
  };
};

// Global error handler
export const errorHandler = (
  err: Error | AppError,
  req: Request,
  res: Response,
  next: NextFunction
): void => {
  // Log error
  console.error('Error:', {
    message: err.message,
    stack: err.stack,
    path: req.path,
    method: req.method
  });
  
  // Handle custom AppError
  if (err instanceof AppError) {
    res.status(err.statusCode).json({
      error: err.message,
      ...(process.env.NODE_ENV === 'development' && { stack: err.stack })
    });
    return;
  }
  
  // Handle other errors
  res.status(500).json({
    error: 'Internal server error',
    ...(process.env.NODE_ENV === 'development' && {
      message: err.message,
      stack: err.stack
    })
  });
};

// ============================================
// src/app.ts - MAIN APPLICATION
// ============================================

import express, { Express } from 'express';
import userRoutes from './routes/userRoutes';
import { errorHandler } from './middleware/errorHandler';

const app: Express = express();

// Global middleware
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// Request logging
app.use((req, res, next) => {
  console.log(`${req.method} ${req.path}`);
  next();
});

// Mount routes
app.use('/api/users', userRoutes);  // All user routes under /api/users

// Health check
app.get('/health', (req, res) => {
  res.json({ status: 'ok', timestamp: new Date().toISOString() });
});

// 404 handler
app.use((req, res) => {
  res.status(404).json({ error: 'Route not found' });
});

// Error handler (must be last)
app.use(errorHandler);

export default app;