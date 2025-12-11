# Rust Complete Mastery Tutorial - Basic

## Table of Contents
1. [Introduction to Rust](#introduction-to-rust)
2. [Setting Up Rust Environment](#setting-up-rust-environment)
3. [Hello World - Your First Rust Program](#hello-world---your-first-rust-program)
4. [Variables and Mutability](#variables-and-mutability)
5. [Data Types](#data-types)
6. [Functions](#functions)
7. [Comments](#comments)
8. [Control Flow](#control-flow)
9. [Ownership](#ownership)
10. [References and Borrowing](#references-and-borrowing)
11. [Slices](#slices)
12. [Structs](#structs)
13. [Enums and Pattern Matching](#enums-and-pattern-matching)
14. [Error Handling](#error-handling)
15. [Modules and Packages](#modules-and-packages)

---

## Introduction to Rust

### What is Rust?
Rust is a systems programming language that focuses on three key goals:
- **Safety**: Memory safety without garbage collection
- **Speed**: Performance comparable to C/C++
- **Concurrency**: Fearless concurrency without data races

### Why Rust?
- Memory safety guaranteed at compile time
- No garbage collector
- Zero-cost abstractions
- Thread safety
- Excellent tooling (Cargo, Rustfmt, Clippy)
- Growing ecosystem

### Key Features
```rust
// Rust prevents common bugs at compile time
// - No null pointer exceptions
// - No data races
// - No use-after-free
// - No buffer overflows
```

---

## Setting Up Rust Environment

### Installing Rust (rustup)
```bash
# On Linux/macOS
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# On Windows
# Download and run rustup-init.exe from https://rustup.rs/
```

### Verify Installation
```bash
# Check Rust compiler version
rustc --version

# Check Cargo (package manager) version
cargo --version

# Check rustup version
rustup --version
```

### Update Rust
```bash
# Update to the latest stable version
rustup update
```

### Useful Cargo Commands
```bash
# Create new project
cargo new my_project

# Create new library
cargo new --lib my_library

# Build project
cargo build

# Build with optimizations (release mode)
cargo build --release

# Run project
cargo run

# Check code without building
cargo check

# Run tests
cargo test

# Format code
cargo fmt

# Lint code
cargo clippy
```

---

## Hello World - Your First Rust Program

### Basic Hello World
```rust
// main.rs - Entry point of a Rust program
fn main() {
    // println! is a macro (note the !)
    println!("Hello, World!");
}
```

**Compilation and Execution:**
```bash
# Compile using rustc
rustc main.rs

# Run the executable
./main  # On Linux/macOS
main.exe  # On Windows
```

### Using Cargo
```bash
# Create new project
cargo new hello_world
cd hello_world

# Project structure:
# hello_world/
# ├── Cargo.toml  (Package configuration)
# └── src/
#     └── main.rs  (Source code)

# Run the project
cargo run
```

### Advanced Hello World
```rust
fn main() {
    // String formatting
    let name = "Rustacean";
    let age = 25;
    
    // Positional arguments
    println!("Hello, {}! You are {} years old.", name, age);
    
    // Named arguments
    println!("Hello, {name}! You are {age} years old.");
    
    // Formatting options
    println!("Binary: {:b}, Hex: {:x}, Octal: {:o}", 42, 42, 42);
    
    // Debug formatting
    let numbers = vec![1, 2, 3];
    println!("Debug: {:?}", numbers);
    println!("Pretty debug: {:#?}", numbers);
}
```

---

## Variables and Mutability

### Immutable Variables (Default)
```rust
fn main() {
    // Variables are immutable by default
    let x = 5;
    println!("The value of x is: {}", x);
    
    // This would cause a compile error:
    // x = 6;  // Error: cannot assign twice to immutable variable
}
```

### Mutable Variables
```rust
fn main() {
    // Use 'mut' keyword for mutable variables
    let mut x = 5;
    println!("The value of x is: {}", x);
    
    x = 6;  // This is allowed
    println!("The value of x is now: {}", x);
}
```

### Constants
```rust
// Constants are always immutable
// Must have type annotation
// Can be declared in any scope, including global
const MAX_POINTS: u32 = 100_000;
const PI: f64 = 3.141592653589793;

fn main() {
    println!("Max points: {}", MAX_POINTS);
    println!("Pi: {}", PI);
}
```

### Shadowing
```rust
fn main() {
    // Shadowing allows reusing variable names
    let x = 5;
    
    // Shadow the previous x
    let x = x + 1;
    
    {
        // Shadow in inner scope
        let x = x * 2;
        println!("Inner x: {}", x);  // 12
    }
    
    println!("Outer x: {}", x);  // 6
    
    // Shadowing can change type
    let spaces = "   ";
    let spaces = spaces.len();  // Now it's a number
    println!("Number of spaces: {}", spaces);
}
```

---

## Data Types

### Scalar Types

#### Integer Types
```rust
fn main() {
    // Signed integers: i8, i16, i32, i64, i128, isize
    let signed_8bit: i8 = -128;
    let signed_32bit: i32 = -2_147_483_648;
    
    // Unsigned integers: u8, u16, u32, u64, u128, usize
    let unsigned_8bit: u8 = 255;
    let unsigned_32bit: u32 = 4_294_967_295;
    
    // Default is i32
    let default_int = 42;
    
    // Number literals
    let decimal = 98_222;        // Decimal
    let hex = 0xff;              // Hexadecimal
    let octal = 0o77;            // Octal
    let binary = 0b1111_0000;    // Binary
    let byte = b'A';             // Byte (u8 only)
    
    println!("Decimal: {}, Hex: {}, Binary: {}", decimal, hex, binary);
}
```

#### Floating-Point Types
```rust
fn main() {
    // f32 (32-bit) and f64 (64-bit, default)
    let x: f32 = 3.0;
    let y: f64 = 2.0;  // Default type
    
    let z = 2.5;  // f64 by default
    
    println!("x: {}, y: {}, z: {}", x, y, z);
}
```

#### Boolean Type
```rust
fn main() {
    let t = true;
    let f: bool = false;
    
    // Boolean operations
    let and = t && f;
    let or = t || f;
    let not = !t;
    
    println!("AND: {}, OR: {}, NOT: {}", and, or, not);
}
```

#### Character Type
```rust
fn main() {
    // char is 4 bytes (Unicode Scalar Value)
    let c = 'z';
    let z: char = 'ℤ';
    let heart_eyed_cat = '😻';
    let chinese = '中';
    
    println!("Characters: {} {} {} {}", c, z, heart_eyed_cat, chinese);
}
```

### Compound Types

#### Tuples
```rust
fn main() {
    // Fixed-size, mixed-type collection
    let tup: (i32, f64, u8) = (500, 6.4, 1);
    
    // Destructuring
    let (x, y, z) = tup;
    println!("x: {}, y: {}, z: {}", x, y, z);
    
    // Access by index
    let five_hundred = tup.0;
    let six_point_four = tup.1;
    let one = tup.2;
    
    println!("Values: {}, {}, {}", five_hundred, six_point_four, one);
    
    // Unit type (empty tuple)
    let unit = ();
}
```

#### Arrays
```rust
fn main() {
    // Fixed-size, same-type collection
    let array = [1, 2, 3, 4, 5];
    
    // Type annotation: [type; size]
    let array: [i32; 5] = [1, 2, 3, 4, 5];
    
    // Initialize with same value
    let array = [3; 5];  // [3, 3, 3, 3, 3]
    
    // Access elements
    let first = array[0];
    let second = array[1];
    
    println!("First: {}, Second: {}", first, second);
    
    // Array length
    let length = array.len();
    println!("Array length: {}", length);
    
    // Iterate over array
    for element in &array {
        println!("Element: {}", element);
    }
}
```

### Type Conversion
```rust
fn main() {
    // Explicit type conversion (casting)
    let integer = 42;
    let float = integer as f64;
    
    let x = 2.5;
    let y = x as i32;  // Truncates to 2
    
    println!("Integer: {}, Float: {}", integer, float);
    println!("x: {}, y (truncated): {}", x, y);
    
    // Parse strings to numbers
    let num_str = "42";
    let num: i32 = num_str.parse().expect("Not a number!");
    println!("Parsed number: {}", num);
}
```

---

## Functions

### Basic Functions
```rust
fn main() {
    greet();
    say_hello("Alice");
    let result = add(5, 3);
    println!("5 + 3 = {}", result);
}

// Function with no parameters or return value
fn greet() {
    println!("Hello!");
}

// Function with parameters
fn say_hello(name: &str) {
    println!("Hello, {}!", name);
}

// Function with return value
fn add(a: i32, b: i32) -> i32 {
    a + b  // Expression without semicolon returns value
}
```

### Statements vs Expressions
```rust
fn main() {
    // Statement: doesn't return a value
    let y = 6;
    
    // Expression: returns a value
    let x = {
        let y = 3;
        y + 1  // No semicolon - this is an expression
    };
    
    println!("x: {}, y: {}", x, y);
}

// Function returning last expression
fn five() -> i32 {
    5  // No semicolon
}

// Explicit return
fn explicit_return() -> i32 {
    return 5;  // With semicolon
}
```

### Multiple Return Values
```rust
fn main() {
    let (sum, product) = calculate(4, 5);
    println!("Sum: {}, Product: {}", sum, product);
}

fn calculate(a: i32, b: i32) -> (i32, i32) {
    (a + b, a * b)
}
```

### Early Return
```rust
fn divide(a: f64, b: f64) -> f64 {
    if b == 0.0 {
        println!("Cannot divide by zero!");
        return 0.0;
    }
    
    a / b
}

fn main() {
    println!("10 / 2 = {}", divide(10.0, 2.0));
    println!("10 / 0 = {}", divide(10.0, 0.0));
}
```

---

## Comments

### Line Comments
```rust
fn main() {
    // This is a line comment
    // It extends to the end of the line
    
    let x = 5;  // You can also put comments at the end of lines
}
```

### Block Comments
```rust
fn main() {
    /*
     * This is a block comment
     * It can span multiple lines
     */
    
    let x = /* inline block comment */ 5;
}
```

### Documentation Comments
```rust
/// This is a documentation comment for the following item
/// It uses three slashes and supports Markdown
/// 
/// # Examples
/// 
/// ```
/// let result = add(2, 3);
/// assert_eq!(result, 5);
/// ```
fn add(a: i32, b: i32) -> i32 {
    a + b
}

//! This is a module-level documentation comment
//! It documents the enclosing item (like a module or crate)

fn main() {
    println!("Sum: {}", add(2, 3));
}
```

---

## Control Flow

### If Expressions
```rust
fn main() {
    let number = 7;
    
    // Basic if
    if number < 5 {
        println!("Number is less than 5");
    }
    
    // if-else
    if number % 2 == 0 {
        println!("Number is even");
    } else {
        println!("Number is odd");
    }
    
    // if-else if-else
    if number < 0 {
        println!("Negative");
    } else if number == 0 {
        println!("Zero");
    } else {
        println!("Positive");
    }
    
    // if is an expression (returns a value)
    let condition = true;
    let number = if condition { 5 } else { 6 };
    println!("Number: {}", number);
}
```

### Loop - Infinite Loop
```rust
fn main() {
    let mut counter = 0;
    
    loop {
        counter += 1;
        
        if counter == 5 {
            break;  // Exit loop
        }
        
        println!("Counter: {}", counter);
    }
    
    // Loop can return a value
    let result = loop {
        counter += 1;
        
        if counter == 10 {
            break counter * 2;
        }
    };
    
    println!("Result: {}", result);
}
```

### While Loop
```rust
fn main() {
    let mut number = 3;
    
    while number != 0 {
        println!("{}!", number);
        number -= 1;
    }
    
    println!("LIFTOFF!");
}
```

### For Loop
```rust
fn main() {
    // Iterate over array
    let array = [10, 20, 30, 40, 50];
    
    for element in array {
        println!("Value: {}", element);
    }
    
    // Iterate over range
    for number in 1..4 {
        println!("Number: {}", number);  // 1, 2, 3 (exclusive end)
    }
    
    // Inclusive range
    for number in 1..=4 {
        println!("Number: {}", number);  // 1, 2, 3, 4 (inclusive end)
    }
    
    // Reverse iteration
    for number in (1..4).rev() {
        println!("Countdown: {}", number);  // 3, 2, 1
    }
    
    // Enumerate (get index and value)
    for (index, value) in array.iter().enumerate() {
        println!("Index {}: Value {}", index, value);
    }
}
```

### Loop Labels
```rust
fn main() {
    let mut count = 0;
    
    'outer: loop {
        println!("Outer loop");
        let mut remaining = 10;
        
        loop {
            println!("  Inner loop: remaining = {}", remaining);
            
            if remaining == 9 {
                break;  // Breaks inner loop
            }
            
            if count == 2 {
                break 'outer;  // Breaks outer loop
            }
            
            remaining -= 1;
        }
        
        count += 1;
    }
    
    println!("Exited outer loop");
}
```

---

## Ownership

### Ownership Rules
```rust
// 1. Each value in Rust has an owner
// 2. There can only be one owner at a time
// 3. When the owner goes out of scope, the value is dropped
```

### Variable Scope
```rust
fn main() {
    {                      // s is not valid here, not yet declared
        let s = "hello";   // s is valid from this point forward
        println!("{}", s);
    }                      // scope is over, s is no longer valid
    
    // println!("{}", s); // Error: s not found in scope
}
```

### Memory and Allocation
```rust
fn main() {
    // String literal (immutable, fixed size, stack)
    let s1 = "hello";
    
    // String type (mutable, growable, heap)
    let mut s2 = String::from("hello");
    s2.push_str(", world!");
    
    println!("{}", s2);
}
```

### Move Semantics
```rust
fn main() {
    // Simple types (Copy trait) - stack copy
    let x = 5;
    let y = x;  // x is copied to y
    println!("x: {}, y: {}", x, y);  // Both valid
    
    // Complex types (heap data) - ownership moves
    let s1 = String::from("hello");
    let s2 = s1;  // s1 is moved to s2
    
    // println!("{}", s1);  // Error: s1 is no longer valid
    println!("{}", s2);     // Only s2 is valid
}
```

### Clone - Deep Copy
```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1.clone();  // Explicitly deep copy
    
    println!("s1: {}, s2: {}", s1, s2);  // Both valid
}
```

### Ownership and Functions
```rust
fn main() {
    let s = String::from("hello");
    takes_ownership(s);  // s is moved into the function
    
    // println!("{}", s);  // Error: s is no longer valid
    
    let x = 5;
    makes_copy(x);  // x is copied (i32 has Copy trait)
    
    println!("x: {}", x);  // x is still valid
}

fn takes_ownership(some_string: String) {
    println!("{}", some_string);
}  // some_string is dropped here

fn makes_copy(some_integer: i32) {
    println!("{}", some_integer);
}  // some_integer goes out of scope, nothing special happens
```

### Return Values and Scope
```rust
fn main() {
    let s1 = gives_ownership();  // Function returns ownership
    
    let s2 = String::from("hello");
    let s3 = takes_and_gives_back(s2);  // s2 is moved, s3 gets ownership
    
    println!("s1: {}, s3: {}", s1, s3);
}

fn gives_ownership() -> String {
    let some_string = String::from("yours");
    some_string  // Ownership is moved to caller
}

fn takes_and_gives_back(a_string: String) -> String {
    a_string  // Ownership is moved to caller
}
```

---

## References and Borrowing

### Immutable References
```rust
fn main() {
    let s1 = String::from("hello");
    
    // Pass reference (borrowing) instead of ownership
    let len = calculate_length(&s1);
    
    println!("Length of '{}' is {}", s1, len);  // s1 still valid
}

fn calculate_length(s: &String) -> usize {
    s.len()
}  // s goes out of scope, but doesn't own the data, so nothing is dropped
```

### Mutable References
```rust
fn main() {
    let mut s = String::from("hello");
    
    change(&mut s);
    
    println!("{}", s);
}

fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```

### Borrowing Rules
```rust
fn main() {
    let mut s = String::from("hello");
    
    // Rule 1: You can have multiple immutable references
    let r1 = &s;
    let r2 = &s;
    println!("{} and {}", r1, r2);
    
    // Rule 2: You can have only ONE mutable reference at a time
    let r3 = &mut s;
    // let r4 = &mut s;  // Error: cannot borrow s as mutable more than once
    
    println!("{}", r3);
    
    // Rule 3: Cannot have mutable and immutable references simultaneously
    let r5 = &s;
    // let r6 = &mut s;  // Error: cannot borrow s as mutable
    println!("{}", r5);
}
```

### Reference Scope
```rust
fn main() {
    let mut s = String::from("hello");
    
    let r1 = &s;
    let r2 = &s;
    println!("{} and {}", r1, r2);
    // r1 and r2 are no longer used after this point
    
    let r3 = &mut s;  // This is OK because r1 and r2 are no longer used
    println!("{}", r3);
}
```

### Dangling References Prevention
```rust
fn main() {
    // let reference_to_nothing = dangle();  // Error: prevented by compiler
    let string = no_dangle();
    println!("{}", string);
}

// This would create a dangling reference (compile error)
// fn dangle() -> &String {
//     let s = String::from("hello");
//     &s  // Error: s will be dropped, leaving dangling reference
// }

// Correct: return ownership instead
fn no_dangle() -> String {
    let s = String::from("hello");
    s  // Ownership is moved out
}
```

---

## Slices

### String Slices
```rust
fn main() {
    let s = String::from("hello world");
    
    // String slice syntax: &s[start..end]
    let hello = &s[0..5];   // "hello"
    let world = &s[6..11];  // "world"
    
    // Shorthand syntax
    let hello = &s[..5];    // Same as &s[0..5]
    let world = &s[6..];    // Same as &s[6..11]
    let whole = &s[..];     // Same as &s[0..11]
    
    println!("hello: {}, world: {}", hello, world);
    
    // String literals are slices
    let literal: &str = "hello world";
    
    // Finding first word
    let word = first_word(&s);
    println!("First word: {}", word);
}

fn first_word(s: &str) -> &str {
    let bytes = s.as_bytes();
    
    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[..i];
        }
    }
    
    &s[..]
}
```

### Array Slices
```rust
fn main() {
    let array = [1, 2, 3, 4, 5];
    
    // Array slice
    let slice = &array[1..3];  // [2, 3]
    
    println!("Slice: {:?}", slice);
    
    // Iterate over slice
    for element in slice {
        println!("Element: {}", element);
    }
}
```

### Slice as Function Parameter
```rust
fn main() {
    let s = String::from("hello world");
    let a = [1, 2, 3, 4, 5];
    
    // Can pass String or &str
    println!("First word: {}", first_word(&s));
    println!("First word: {}", first_word("hello world"));
    
    // Array slice
    println!("Sum: {}", sum(&a));
    println!("Sum: {}", sum(&a[1..3]));
}

fn first_word(s: &str) -> &str {
    let bytes = s.as_bytes();
    
    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[..i];
        }
    }
    
    &s[..]
}

fn sum(slice: &[i32]) -> i32 {
    let mut total = 0;
    for &value in slice {
        total += value;
    }
    total
}
```

---

## Structs

### Defining and Instantiating Structs
```rust
// Define a struct
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool,
}

fn main() {
    // Create an instance
    let user1 = User {
        email: String::from("user@example.com"),
        username: String::from("username123"),
        active: true,
        sign_in_count: 1,
    };
    
    println!("Username: {}", user1.username);
    
    // Mutable instance
    let mut user2 = User {
        email: String::from("another@example.com"),
        username: String::from("anotheruser"),
        active: true,
        sign_in_count: 1,
    };
    
    user2.email = String::from("newemail@example.com");
}
```

### Struct Update Syntax
```rust
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool,
}

fn main() {
    let user1 = User {
        email: String::from("user@example.com"),
        username: String::from("username123"),
        active: true,
        sign_in_count: 1,
    };
    
    // Create new instance using values from user1
    let user2 = User {
        email: String::from("another@example.com"),
        ..user1  // Use remaining fields from user1
    };
    
    // Note: user1.username is moved to user2
    // println!("{}", user1.username);  // Error
    println!("{}", user2.username);
}
```

### Tuple Structs
```rust
// Tuple structs have names but fields don't
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

fn main() {
    let black = Color(0, 0, 0);
    let origin = Point(0, 0, 0);
    
    // Access by index
    println!("Black: ({}, {}, {})", black.0, black.1, black.2);
    
    // Color and Point are different types
    // let color: Color = origin;  // Error: type mismatch
}
```

### Unit-Like Structs
```rust
// Structs without any fields
struct AlwaysEqual;

fn main() {
    let subject = AlwaysEqual;
    
    // Useful for implementing traits without data
}
```

### Methods
```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    // Method: takes &self as first parameter
    fn area(&self) -> u32 {
        self.width * self.height
    }
    
    // Method with mutable reference
    fn double(&mut self) {
        self.width *= 2;
        self.height *= 2;
    }
    
    // Method with multiple parameters
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}

fn main() {
    let rect = Rectangle {
        width: 30,
        height: 50,
    };
    
    println!("Area: {}", rect.area());
    
    let mut rect2 = Rectangle {
        width: 10,
        height: 20,
    };
    
    rect2.double();
    println!("Doubled: {:?}", rect2);
    
    println!("Can rect hold rect2? {}", rect.can_hold(&rect2));
}
```

### Associated Functions
```rust
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    // Associated function (like static method)
    // Doesn't take self as parameter
    fn square(size: u32) -> Rectangle {
        Rectangle {
            width: size,
            height: size,
        }
    }
    
    fn new(width: u32, height: u32) -> Rectangle {
        Rectangle { width, height }
    }
}

fn main() {
    // Called using :: syntax
    let sq = Rectangle::square(25);
    let rect = Rectangle::new(30, 50);
    
    println!("Square: {}x{}", sq.width, sq.height);
    println!("Rectangle: {}x{}", rect.width, rect.height);
}
```

### Multiple impl Blocks
```rust
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

impl Rectangle {
    fn perimeter(&self) -> u32 {
        2 * (self.width + self.height)
    }
}

fn main() {
    let rect = Rectangle {
        width: 30,
        height: 50,
    };
    
    println!("Area: {}", rect.area());
    println!("Perimeter: {}", rect.perimeter());
}
```

---

## Enums and Pattern Matching

### Defining Enums
```rust
// Basic enum
enum IpAddrKind {
    V4,
    V6,
}

// Enum with data
enum IpAddr {
    V4(u8, u8, u8, u8),
    V6(String),
}

// Enum with different types
enum Message {
    Quit,                       // No data
    Move { x: i32, y: i32 },    // Named fields
    Write(String),              // Single String
    ChangeColor(i32, i32, i32), // Three i32s
}

fn main() {
    let four = IpAddrKind::V4;
    let six = IpAddrKind::V6;
    
    let home = IpAddr::V4(127, 0, 0, 1);
    let loopback = IpAddr::V6(String::from("::1"));
    
    let msg = Message::Write(String::from("hello"));
}
```

### Enum Methods
```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

impl Message {
    fn call(&self) {
        match self {
            Message::Quit => println!("Quit"),
            Message::Move { x, y } => println!("Move to ({}, {})", x, y),
            Message::Write(text) => println!("Text: {}", text),
            Message::ChangeColor(r, g, b) => println!("RGB({}, {}, {})", r, g, b),
        }
    }
}

fn main() {
    let msg = Message::Write(String::from("hello"));
    msg.call();
}
```

### The Option Enum
```rust
fn main() {
    // Option<T> is defined in standard library
    // enum Option<T> {
    //     Some(T),
    //     None,
    // }
    
    let some_number = Some(5);
    let some_string = Some("a string");
    let absent_number: Option<i32> = None;
    
    // Must handle None case
    let x = 5;
    let y = Some(10);
    
    // let sum = x + y;  // Error: can't add Option<i32> to i32
    
    // Must extract value from Option
    match y {
        Some(value) => println!("Sum: {}", x + value),
        None => println!("No value"),
    }
}
```

### Match Expression
```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}

fn main() {
    let coin = Coin::Quarter;
    println!("Value: {} cents", value_in_cents(coin));
}
```

### Patterns That Bind to Values
```rust
#[derive(Debug)]
enum UsState {
    Alabama,
    Alaska,
    // ... etc
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            println!("State quarter from {:?}!", state);
            25
        }
    }
}

fn main() {
    let coin = Coin::Quarter(UsState::Alaska);
    println!("Value: {} cents", value_in_cents(coin));
}
```

### Matching with Option<T>
```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        None => None,
        Some(i) => Some(i + 1),
    }
}

fn main() {
    let five = Some(5);
    let six = plus_one(five);
    let none = plus_one(None);
    
    println!("six: {:?}, none: {:?}", six, none);
}
```

### Catch-All Patterns
```rust
fn main() {
    let dice_roll = 9;
    
    match dice_roll {
        3 => add_fancy_hat(),
        7 => remove_fancy_hat(),
        other => move_player(other),  // Catch-all with value
    }
    
    match dice_roll {
        3 => add_fancy_hat(),
        7 => remove_fancy_hat(),
        _ => reroll(),  // Catch-all without value
    }
}

fn add_fancy_hat() {}
fn remove_fancy_hat() {}
fn move_player(num_spaces: u8) {}
fn reroll() {}
```

### if let - Concise Control Flow
```rust
fn main() {
    let some_value = Some(3);
    
    // Verbose match
    match some_value {
        Some(3) => println!("three"),
        _ => (),
    }
    
    // Concise if let
    if let Some(3) = some_value {
        println!("three");
    }
    
    // if let with else
    let coin = Coin::Penny;
    
    let mut count = 0;
    if let Coin::Quarter(state) = coin {
        println!("State quarter from {:?}!", state);
    } else {
        count += 1;
    }
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}

#[derive(Debug)]
enum UsState {
    Alabama,
    Alaska,
}
```

---

## Error Handling

### Unrecoverable Errors with panic!
```rust
fn main() {
    // Explicit panic
    // panic!("crash and burn");
    
    // Accessing invalid index causes panic
    let v = vec![1, 2, 3];
    // v[99];  // panic: index out of bounds
}
```

### Recoverable Errors with Result
```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    // Result enum:
    // enum Result<T, E> {
    //     Ok(T),
    //     Err(E),
    // }
    
    // Basic Result handling with match
    let file_result = File::open("hello.txt");
    
    let file = match file_result {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Problem creating file: {:?}", e),
            },
            other_error => {
                panic!("Problem opening file: {:?}", other_error);
            }
        },
    };
}
```

### Shortcuts: unwrap and expect
```rust
use std::fs::File;

fn main() {
    // unwrap: returns value or panics
    // let file = File::open("hello.txt").unwrap();
    
    // expect: like unwrap but with custom message
    let file = File::open("hello.txt")
        .expect("Failed to open hello.txt");
}
```

### Propagating Errors
```rust
use std::fs::File;
use std::io::{self, Read};

// Return Result to caller
fn read_username_from_file() -> Result<String, io::Error> {
    let file_result = File::open("username.txt");
    
    let mut file = match file_result {
        Ok(file) => file,
        Err(e) => return Err(e),
    };
    
    let mut username = String::new();
    
    match file.read_to_string(&mut username) {
        Ok(_) => Ok(username),
        Err(e) => Err(e),
    }
}

fn main() {
    match read_username_from_file() {
        Ok(username) => println!("Username: {}", username),
        Err(e) => println!("Error: {}", e),
    }
}
```

### The ? Operator
```rust
use std::fs::File;
use std::io::{self, Read};

// Shortcut for propagating errors
fn read_username_from_file() -> Result<String, io::Error> {
    let mut file = File::open("username.txt")?;  // Returns error if Err
    let mut username = String::new();
    file.read_to_string(&mut username)?;
    Ok(username)
}

// Even shorter
fn read_username_short() -> Result<String, io::Error> {
    let mut username = String::new();
    File::open("username.txt")?.read_to_string(&mut username)?;
    Ok(username)
}

// Using fs::read_to_string
fn read_username_shortest() -> Result<String, io::Error> {
    std::fs::read_to_string("username.txt")
}

fn main() {
    match read_username_from_file() {
        Ok(username) => println!("Username: {}", username),
        Err(e) => println!("Error: {}", e),
    }
}
```

### The ? Operator with Option
```rust
fn last_char_of_first_line(text: &str) -> Option<char> {
    text.lines().next()?.chars().last()
}

fn main() {
    let text = "Hello\nWorld";
    match last_char_of_first_line(text) {
        Some(c) => println!("Last char: {}", c),
        None => println!("No character found"),
    }
}
```

---

## Modules and Packages

### Packages and Crates
```rust
// A package contains one or more crates
// Cargo.toml defines a package
// 
// Package structure:
// my_package/
// ├── Cargo.toml
// ├── src/
// │   ├── main.rs      (binary crate root)
// │   ├── lib.rs       (library crate root)
// │   └── bin/
// │       └── other.rs (additional binary crate)
```

### Defining Modules
```rust
// src/lib.rs
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
        
        fn seat_at_table() {}
    }
    
    mod serving {
        fn take_order() {}
        
        fn serve_order() {}
        
        fn take_payment() {}
    }
}

// Use module from same crate
pub fn eat_at_restaurant() {
    // Absolute path
    crate::front_of_house::hosting::add_to_waitlist();
    
    // Relative path
    front_of_house::hosting::add_to_waitlist();
}
```

### Paths for Referring to Items
```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // Absolute path
    crate::front_of_house::hosting::add_to_waitlist();
    
    // Relative path
    front_of_house::hosting::add_to_waitlist();
}
```

### Using super
```rust
fn serve_order() {}

mod back_of_house {
    fn fix_incorrect_order() {
        cook_order();
        super::serve_order();  // Go up to parent module
    }
    
    fn cook_order() {}
}
```

### Pub Struct and Enum
```rust
mod back_of_house {
    pub struct Breakfast {
        pub toast: String,        // Public field
        seasonal_fruit: String,   // Private field
    }
    
    impl Breakfast {
        pub fn summer(toast: &str) -> Breakfast {
            Breakfast {
                toast: String::from(toast),
                seasonal_fruit: String::from("peaches"),
            }
        }
    }
    
    // All variants are public if enum is public
    pub enum Appetizer {
        Soup,
        Salad,
    }
}

pub fn eat_at_restaurant() {
    let mut meal = back_of_house::Breakfast::summer("Rye");
    meal.toast = String::from("Wheat");
    
    // meal.seasonal_fruit = String::from("blueberries");  // Error: private
    
    let order1 = back_of_house::Appetizer::Soup;
    let order2 = back_of_house::Appetizer::Salad;
}
```

### The use Keyword
```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

// Bring into scope
use crate::front_of_house::hosting;

// Can also use full path
// use crate::front_of_house::hosting::add_to_waitlist;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
    // Or: add_to_waitlist(); if using full path
}
```

### Idiomatic use Paths
```rust
// Functions: bring parent module
use std::collections::HashMap;

fn main() {
    let mut map = HashMap::new();
    map.insert(1, 2);
}

// Structs, enums, etc: bring full path
use std::fmt::Result;
use std::io::Result as IoResult;  // Use 'as' for name conflicts

fn function1() -> Result {
    Ok(())
}

fn function2() -> IoResult<()> {
    Ok(())
}
```

### Re-exporting with pub use
```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

// Re-export for external code
pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```

### Nested Paths
```rust
// Instead of:
// use std::io;
// use std::cmp::Ordering;

// Use nested paths:
use std::{cmp::Ordering, io};

// Instead of:
// use std::io;
// use std::io::Write;

// Use:
use std::io::{self, Write};
```

### Glob Operator
```rust
// Bring all public items into scope
use std::collections::*;

fn main() {
    let mut map = HashMap::new();
    let mut set = HashSet::new();
}
```

### Separating Modules into Files
```rust
// src/lib.rs
mod front_of_house;  // Looks for src/front_of_house.rs or src/front_of_house/mod.rs

pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}

// src/front_of_house.rs
pub mod hosting;  // Looks for src/front_of_house/hosting.rs

// src/front_of_house/hosting.rs
pub fn add_to_waitlist() {}
```

---

## Practice Exercises

### Exercise 1: Temperature Converter
```rust
// Convert between Fahrenheit and Celsius

fn fahrenheit_to_celsius(f: f64) -> f64 {
    (f - 32.0) * 5.0 / 9.0
}

fn celsius_to_fahrenheit(c: f64) -> f64 {
    c * 9.0 / 5.0 + 32.0
}

fn main() {
    let f = 98.6;
    let c = fahrenheit_to_celsius(f);
    println!("{}°F = {:.2}°C", f, c);
    
    let c = 37.0;
    let f = celsius_to_fahrenheit(c);
    println!("{}°C = {:.2}°F", c, f);
}
```

### Exercise 2: Fibonacci Sequence
```rust
fn fibonacci(n: u32) -> u64 {
    match n {
        0 => 0,
        1 => 1,
        _ => fibonacci(n - 1) + fibonacci(n - 2),
    }
}

// Iterative version (more efficient)
fn fibonacci_iterative(n: u32) -> u64 {
    if n == 0 {
        return 0;
    }
    
    let mut a = 0;
    let mut b = 1;
    
    for _ in 1..n {
        let temp = a + b;
        a = b;
        b = temp;
    }
    
    b
}

fn main() {
    for i in 0..10 {
        println!("fibonacci({}) = {}", i, fibonacci_iterative(i));
    }
}
```

### Exercise 3: Rectangle Area Calculator
```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn new(width: u32, height: u32) -> Rectangle {
        Rectangle { width, height }
    }
    
    fn square(size: u32) -> Rectangle {
        Rectangle {
            width: size,
            height: size,
        }
    }
    
    fn area(&self) -> u32 {
        self.width * self.height
    }
    
    fn perimeter(&self) -> u32 {
        2 * (self.width + self.height)
    }
    
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}

fn main() {
    let rect1 = Rectangle::new(30, 50);
    let rect2 = Rectangle::new(10, 40);
    let rect3 = Rectangle::new(60, 45);
    let square = Rectangle::square(25);
    
    println!("rect1: {:?}", rect1);
    println!("Area: {}", rect1.area());
    println!("Perimeter: {}", rect1.perimeter());
    
    println!("Can rect1 hold rect2? {}", rect1.can_hold(&rect2));
    println!("Can rect1 hold rect3? {}", rect1.can_hold(&rect3));
    
    println!("Square: {:?}", square);
}
```

---

## Summary

In this basic tutorial, you learned:
- Setting up Rust environment
- Variables, data types, and mutability
- Functions and control flow
- Ownership, borrowing, and references
- Structs, enums, and pattern matching
- Error handling with Result and Option
- Organizing code with modules

### Next Steps
Continue to the **Intermediate Tutorial** to learn:
- Collections (Vec, HashMap, HashSet)
- Generics and Traits
- Lifetimes
- Iterators and Closures
- Smart Pointers
- Concurrency
- And much more!

---

**Happy Rust Programming! 🦀**
