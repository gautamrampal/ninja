# Zig Complete Mastery Tutorial - Basic

## Table of Contents
1. [Introduction to Zig](#introduction-to-zig)
2. [Installation and Setup](#installation-and-setup)
3. [Hello World](#hello-world)
4. [Basic Syntax and Structure](#basic-syntax-and-structure)
5. [Variables and Constants](#variables-and-constants)
6. [Data Types](#data-types)
7. [Operators](#operators)
8. [Control Flow](#control-flow)
9. [Functions](#functions)
10. [Error Handling Basics](#error-handling-basics)
11. [Arrays and Slices](#arrays-and-slices)
12. [Strings](#strings)
13. [Structs](#structs)
14. [Enums](#enums)
15. [Unions](#unions)
16. [Pointers](#pointers)
17. [Memory Allocation Basics](#memory-allocation-basics)
18. [Standard Library Basics](#standard-library-basics)

---

## Introduction to Zig

### What is Zig?
Zig is a general-purpose programming language and toolchain designed for maintaining robust, optimal, and reusable software. It was created by Andrew Kelley in 2015.

### Key Features
- **No hidden control flow**: What you see is what you get
- **No hidden memory allocations**: Explicit memory management
- **No preprocessor, no macros**: Clean, simple metaprogramming
- **Optional type system**: Compile-time optionality checking
- **Manual memory management**: Full control over allocations
- **Cross-compilation**: Built-in cross-compilation support
- **Integration with C**: Seamless C interoperability
- **Build system**: Integrated build system written in Zig

### Philosophy
Zig prioritizes:
- **Simplicity**: Easy to read, easy to understand
- **Performance**: Compete with C and C++
- **Safety**: Catch bugs at compile time when possible
- **Maintainability**: Code should be easy to maintain

---

## Installation and Setup

### Installing Zig

#### Windows
```bash
# Download from https://ziglang.org/download/
# Extract the archive
# Add to PATH environment variable
```

#### Linux
```bash
# Download tarball
wget https://ziglang.org/download/0.11.0/zig-linux-x86_64-0.11.0.tar.xz
tar -xf zig-linux-x86_64-0.11.0.tar.xz
sudo mv zig-linux-x86_64-0.11.0 /opt/zig
export PATH=$PATH:/opt/zig
```

#### macOS
```bash
# Using Homebrew
brew install zig
```

### Verify Installation
```bash
zig version
```

### Editor Setup
- **VSCode**: Install "Zig Language" extension
- **Vim/Neovim**: Use zig.vim plugin
- **Emacs**: Use zig-mode
- **Sublime Text**: Install Zig package

---

## Hello World

### Basic Hello World
```zig
// hello.zig
const std = @import("std");

pub fn main() void {
    std.debug.print("Hello, World!\n", .{});
}
```

**Explanation**:
- `const std = @import("std")`: Imports the standard library
- `pub fn main() void`: Defines a public function named main that returns nothing
- `std.debug.print()`: Prints to stderr (for debugging)
- `.{}`: Empty tuple for format arguments

### Compile and Run
```bash
zig build-exe hello.zig
./hello
```

### Using stdout
```zig
const std = @import("std");

pub fn main() !void {
    const stdout = std.io.getStdOut().writer();
    try stdout.print("Hello, World!\n", .{});
}
```

**Explanation**:
- `!void`: Function can return an error or void
- `try`: Propagates errors to the caller
- `stdout.print()`: Prints to standard output

---

## Basic Syntax and Structure

### File Structure
```zig
// Import statements at the top
const std = @import("std");
const print = std.debug.print;

// Constants and global variables
const MAX_SIZE = 100;
var global_counter: u32 = 0;

// Function definitions
pub fn main() void {
    // Function body
    print("Program starts here\n", .{});
}

// Helper functions
fn helperFunction() void {
    // Implementation
}
```

### Comments
```zig
// Single-line comment

/// Documentation comment for the following item
/// Can span multiple lines
pub fn documentedFunction() void {}

//! Top-level documentation comment
//! Describes the file or module
```

### Identifiers
```zig
// Valid identifiers
const my_variable = 42;
const myVariable = 42;
const MY_CONSTANT = 42;
const _private = 42;

// Reserved keywords cannot be used as identifiers
// const if = 42;  // Error!
```

### Code Blocks
```zig
// Braces define scope
{
    const x = 10;
    // x is visible here
}
// x is not visible here
```

---

## Variables and Constants

### Constants with `const`
```zig
const std = @import("std");

pub fn main() void {
    // Constants must be initialized and cannot be changed
    const x: i32 = 42;
    const y = 100; // Type inference
    
    // x = 50; // Error: cannot assign to constant
    
    std.debug.print("x = {}, y = {}\n", .{x, y});
}
```

**Explanation**:
- `const`: Declares an immutable binding
- Type can be inferred or explicitly specified
- Must be initialized at declaration

### Variables with `var`
```zig
const std = @import("std");

pub fn main() void {
    // Variables can be changed
    var x: i32 = 42;
    std.debug.print("Initial x = {}\n", .{x});
    
    x = 100; // OK
    std.debug.print("Modified x = {}\n", .{x});
    
    // Type inference
    var y = 50;
    y += 10;
    std.debug.print("y = {}\n", .{y});
}
```

**Explanation**:
- `var`: Declares a mutable binding
- Can be reassigned after initialization

### Undefined Values
```zig
pub fn main() void {
    // Declare without initializing (unsafe)
    var x: i32 = undefined;
    
    // Must assign before use
    x = 42;
    std.debug.print("x = {}\n", .{x});
}
```

**Explanation**:
- `undefined`: Memory is allocated but not initialized
- Using undefined values before assignment is undefined behavior
- Useful for performance when you'll assign before use

### Shadowing
```zig
pub fn main() void {
    const x = 10;
    {
        const x = 20; // Shadows outer x
        std.debug.print("Inner x = {}\n", .{x}); // 20
    }
    std.debug.print("Outer x = {}\n", .{x}); // 10
}
```

---

## Data Types

### Integer Types

#### Signed Integers
```zig
const std = @import("std");

pub fn main() void {
    const i8_val: i8 = -128;      // 8-bit signed
    const i16_val: i16 = -32768;  // 16-bit signed
    const i32_val: i32 = -2147483648; // 32-bit signed
    const i64_val: i64 = -9223372036854775808; // 64-bit signed
    const i128_val: i128 = -170141183460469231731687303715884105728; // 128-bit signed
    
    std.debug.print("i8: {}, i16: {}, i32: {}\n", .{i8_val, i16_val, i32_val});
}
```

#### Unsigned Integers
```zig
pub fn main() void {
    const u8_val: u8 = 255;       // 8-bit unsigned
    const u16_val: u16 = 65535;   // 16-bit unsigned
    const u32_val: u32 = 4294967295; // 32-bit unsigned
    const u64_val: u64 = 18446744073709551615; // 64-bit unsigned
    const u128_val: u128 = 340282366920938463463374607431768211455; // 128-bit unsigned
    
    std.debug.print("u8: {}, u16: {}, u32: {}\n", .{u8_val, u16_val, u32_val});
}
```

#### Arbitrary Bit-Width Integers
```zig
pub fn main() void {
    const i3_val: i3 = -4;   // 3-bit signed (-4 to 3)
    const u5_val: u5 = 31;   // 5-bit unsigned (0 to 31)
    const i17_val: i17 = 65536; // 17-bit signed
    
    std.debug.print("i3: {}, u5: {}, i17: {}\n", .{i3_val, u5_val, i17_val});
}
```

**Explanation**:
- Zig supports integers of any bit width from 1 to 65535
- Format: `i{N}` for signed, `u{N}` for unsigned

### Floating Point Types
```zig
pub fn main() void {
    const f16_val: f16 = 3.14;    // 16-bit float
    const f32_val: f32 = 3.14159; // 32-bit float
    const f64_val: f64 = 3.14159265359; // 64-bit float
    const f128_val: f128 = 3.14159265358979323846; // 128-bit float
    
    std.debug.print("f16: {d:.2}, f32: {d:.5}, f64: {d:.10}\n", 
                    .{f16_val, f32_val, f64_val});
}
```

### Boolean Type
```zig
pub fn main() void {
    const is_true: bool = true;
    const is_false: bool = false;
    
    std.debug.print("is_true: {}, is_false: {}\n", .{is_true, is_false});
    
    // Boolean operations
    const result = is_true and !is_false;
    std.debug.print("result: {}\n", .{result});
}
```

### Character Type
```zig
pub fn main() void {
    // No dedicated char type, use u8 or u21 (Unicode codepoint)
    const letter: u8 = 'A';
    const unicode_char: u21 = '😀';
    
    std.debug.print("letter: {c}, value: {}\n", .{letter, letter});
    std.debug.print("unicode: {u}\n", .{unicode_char});
}
```

### Void Type
```zig
pub fn main() void {
    // void represents no value
    const nothing: void = {};
    doNothing();
}

fn doNothing() void {
    // Function that returns nothing
}
```

### Type Type
```zig
pub fn main() void {
    // type is the type of types
    const MyInt = i32;
    const x: MyInt = 42;
    
    std.debug.print("x = {}\n", .{x});
    
    // Types are first-class values
    const T = if (true) i32 else u32;
    const y: T = 100;
    std.debug.print("y = {}\n", .{y});
}
```

### Comptime Int and Float
```zig
pub fn main() void {
    // Compile-time integers have arbitrary precision
    const comptime_int = 999999999999999999999999999;
    const comptime_float = 3.14159265358979323846264338327950288;
    
    // They coerce to appropriate runtime types
    const runtime_int: u64 = comptime_int % 1000000;
    const runtime_float: f64 = comptime_float;
    
    std.debug.print("runtime_int: {}, runtime_float: {d:.10}\n", 
                    .{runtime_int, runtime_float});
}
```

---

## Operators

### Arithmetic Operators
```zig
const std = @import("std");

pub fn main() void {
    const a: i32 = 10;
    const b: i32 = 3;
    
    // Basic arithmetic
    const sum = a + b;          // Addition: 13
    const diff = a - b;         // Subtraction: 7
    const product = a * b;      // Multiplication: 30
    const quotient = @divTrunc(a, b); // Integer division: 3
    const remainder = @rem(a, b);     // Remainder: 1
    
    std.debug.print("sum: {}, diff: {}, product: {}\n", .{sum, diff, product});
    std.debug.print("quotient: {}, remainder: {}\n", .{quotient, remainder});
    
    // Negation
    const neg = -a; // -10
    std.debug.print("negation: {}\n", .{neg});
}
```

**Explanation**:
- Standard operators: `+`, `-`, `*`
- `@divTrunc()`: Truncating division
- `@rem()`: Remainder operation
- No `/` or `%` operators for integers (must be explicit)

### Wrapping Arithmetic
```zig
pub fn main() void {
    // Wrapping operations (overflow wraps around)
    var x: u8 = 255;
    x +%= 1; // Wraps to 0
    std.debug.print("Wrapped: {}\n", .{x});
    
    var y: u8 = 0;
    y -%= 1; // Wraps to 255
    std.debug.print("Wrapped subtraction: {}\n", .{y});
    
    // Wrapping multiplication
    var z: u8 = 200;
    z *%= 2; // Wraps around
    std.debug.print("Wrapped multiplication: {}\n", .{z});
}
```

**Explanation**:
- `+%`, `-%`, `*%`: Wrapping arithmetic operators
- Overflow wraps around instead of causing undefined behavior

### Saturating Arithmetic
```zig
pub fn main() void {
    // Saturating operations (clamp at min/max)
    var x: u8 = 255;
    x +|= 1; // Saturates at 255
    std.debug.print("Saturated: {}\n", .{x});
    
    var y: u8 = 0;
    y -|= 1; // Saturates at 0
    std.debug.print("Saturated subtraction: {}\n", .{y});
}
```

**Explanation**:
- `+|`, `-|`, `*|`: Saturating arithmetic operators
- Values clamp at type minimum/maximum

### Bitwise Operators
```zig
pub fn main() void {
    const a: u8 = 0b11110000;
    const b: u8 = 0b10101010;
    
    const and_result = a & b;   // Bitwise AND: 10100000
    const or_result = a | b;    // Bitwise OR:  11111010
    const xor_result = a ^ b;   // Bitwise XOR: 01011010
    const not_result = ~a;      // Bitwise NOT: 00001111
    
    std.debug.print("AND: {b:0>8}\n", .{and_result});
    std.debug.print("OR:  {b:0>8}\n", .{or_result});
    std.debug.print("XOR: {b:0>8}\n", .{xor_result});
    std.debug.print("NOT: {b:0>8}\n", .{not_result});
}
```

### Bit Shift Operators
```zig
pub fn main() void {
    const value: u8 = 0b00001111;
    
    const left_shift = value << 2;  // Left shift: 00111100
    const right_shift = value >> 2; // Right shift: 00000011
    
    std.debug.print("Original:    {b:0>8}\n", .{value});
    std.debug.print("Left shift:  {b:0>8}\n", .{left_shift});
    std.debug.print("Right shift: {b:0>8}\n", .{right_shift});
}
```

### Comparison Operators
```zig
pub fn main() void {
    const a = 10;
    const b = 20;
    
    const equal = a == b;        // Equal to: false
    const not_equal = a != b;    // Not equal to: true
    const less = a < b;          // Less than: true
    const greater = a > b;       // Greater than: false
    const less_eq = a <= b;      // Less or equal: true
    const greater_eq = a >= b;   // Greater or equal: false
    
    std.debug.print("==: {}, !=: {}, <: {}, >: {}\n", 
                    .{equal, not_equal, less, greater});
}
```

### Logical Operators
```zig
pub fn main() void {
    const t = true;
    const f = false;
    
    const and_result = t and f;  // Logical AND: false
    const or_result = t or f;    // Logical OR: true
    const not_result = !t;       // Logical NOT: false
    
    std.debug.print("AND: {}, OR: {}, NOT: {}\n", 
                    .{and_result, or_result, not_result});
}
```

### Assignment Operators
```zig
pub fn main() void {
    var x: i32 = 10;
    
    x += 5;  // x = x + 5 (15)
    x -= 3;  // x = x - 3 (12)
    x *= 2;  // x = x * 2 (24)
    
    var y: u8 = 0b11110000;
    y &= 0b10101010;  // Bitwise AND assignment
    y |= 0b00001111;  // Bitwise OR assignment
    y ^= 0b11111111;  // Bitwise XOR assignment
    
    std.debug.print("x: {}, y: {b:0>8}\n", .{x, y});
}
```

---

## Control Flow

### If Statements
```zig
const std = @import("std");

pub fn main() void {
    const number = 42;
    
    // Basic if
    if (number > 0) {
        std.debug.print("Positive\n", .{});
    }
    
    // If-else
    if (number % 2 == 0) {
        std.debug.print("Even\n", .{});
    } else {
        std.debug.print("Odd\n", .{});
    }
    
    // If-else if-else
    if (number < 0) {
        std.debug.print("Negative\n", .{});
    } else if (number == 0) {
        std.debug.print("Zero\n", .{});
    } else {
        std.debug.print("Positive\n", .{});
    }
}
```

### If as Expression
```zig
pub fn main() void {
    const number = 42;
    
    // If expression returns a value
    const result = if (number > 0) "positive" else "non-positive";
    std.debug.print("Number is {s}\n", .{result});
    
    // Must have else when used as expression
    const abs_value = if (number >= 0) number else -number;
    std.debug.print("Absolute value: {}\n", .{abs_value});
}
```

### While Loops
```zig
pub fn main() void {
    // Basic while loop
    var i: u32 = 0;
    while (i < 5) {
        std.debug.print("i = {}\n", .{i});
        i += 1;
    }
    
    // While with continue expression
    var j: u32 = 0;
    while (j < 5) : (j += 1) {
        std.debug.print("j = {}\n", .{j});
    }
}
```

**Explanation**:
- Basic `while (condition) { body }`
- Continue expression: `while (condition) : (continue_expr) { body }`
- Continue expression runs after each iteration

### While with Break and Continue
```zig
pub fn main() void {
    var i: u32 = 0;
    while (i < 10) : (i += 1) {
        if (i == 3) continue; // Skip 3
        if (i == 7) break;    // Stop at 7
        std.debug.print("i = {}\n", .{i});
    }
}
```

### While as Expression
```zig
pub fn main() void {
    var i: u32 = 0;
    
    // While can return a value on break
    const result = while (i < 10) : (i += 1) {
        if (i == 5) break i * 2;
    } else 0;
    
    std.debug.print("Result: {}\n", .{result}); // 10
}
```

### For Loops
```zig
pub fn main() void {
    const items = [_]i32{ 1, 2, 3, 4, 5 };
    
    // Iterate over array
    for (items) |item| {
        std.debug.print("item = {}\n", .{item});
    }
    
    // Iterate with index
    for (items, 0..) |item, index| {
        std.debug.print("items[{}] = {}\n", .{index, item});
    }
}
```

**Explanation**:
- `for (array) |item| { }`: Iterate over elements
- `for (array, 0..) |item, index| { }`: Iterate with index
- `0..`: Range starting from 0

### For with Range
```zig
pub fn main() void {
    // Iterate over range
    for (0..5) |i| {
        std.debug.print("i = {}\n", .{i});
    }
    
    // Iterate with step
    var sum: u32 = 0;
    for (0..10) |i| {
        if (i % 2 == 0) {
            sum += i;
        }
    }
    std.debug.print("Sum of evens: {}\n", .{sum});
}
```

### Switch Statements
```zig
pub fn main() void {
    const value: u8 = 2;
    
    // Basic switch
    switch (value) {
        0 => std.debug.print("Zero\n", .{}),
        1 => std.debug.print("One\n", .{}),
        2 => std.debug.print("Two\n", .{}),
        else => std.debug.print("Other\n", .{}),
    }
    
    // Switch with ranges
    const score: u8 = 85;
    switch (score) {
        0...59 => std.debug.print("F\n", .{}),
        60...69 => std.debug.print("D\n", .{}),
        70...79 => std.debug.print("C\n", .{}),
        80...89 => std.debug.print("B\n", .{}),
        90...100 => std.debug.print("A\n", .{}),
        else => std.debug.print("Invalid\n", .{}),
    }
}
```

**Explanation**:
- Switch must be exhaustive (cover all cases or have `else`)
- `...`: Inclusive range in switch cases

### Switch as Expression
```zig
pub fn main() void {
    const value: u8 = 2;
    
    // Switch returns a value
    const result = switch (value) {
        0 => "zero",
        1 => "one",
        2 => "two",
        else => "many",
    };
    
    std.debug.print("Result: {s}\n", .{result});
}
```

### Switch with Multiple Cases
```zig
pub fn main() void {
    const char: u8 = 'a';
    
    const is_vowel = switch (char) {
        'a', 'e', 'i', 'o', 'u' => true,
        'A', 'E', 'I', 'O', 'U' => true,
        else => false,
    };
    
    std.debug.print("Is vowel: {}\n", .{is_vowel});
}
```

---

## Functions

### Basic Function Definition
```zig
const std = @import("std");

// Function with no parameters and no return value
fn greet() void {
    std.debug.print("Hello!\n", .{});
}

// Function with parameters
fn add(a: i32, b: i32) i32 {
    return a + b;
}

pub fn main() void {
    greet();
    const sum = add(5, 3);
    std.debug.print("Sum: {}\n", .{sum});
}
```

**Explanation**:
- `fn` keyword declares a function
- Parameters: `name: Type`
- Return type after parameters
- `void` means no return value

### Function Parameters
```zig
// Pass by value (copy)
fn incrementValue(x: i32) i32 {
    return x + 1;
}

// Pass by reference (pointer)
fn incrementPointer(x: *i32) void {
    x.* += 1;
}

pub fn main() void {
    var value: i32 = 10;
    
    // Pass by value
    const new_value = incrementValue(value);
    std.debug.print("Original: {}, New: {}\n", .{value, new_value});
    
    // Pass by reference
    incrementPointer(&value);
    std.debug.print("Modified: {}\n", .{value});
}
```

### Multiple Return Values
```zig
fn divmod(a: i32, b: i32) struct { quotient: i32, remainder: i32 } {
    return .{
        .quotient = @divTrunc(a, b),
        .remainder = @rem(a, b),
    };
}

pub fn main() void {
    const result = divmod(17, 5);
    std.debug.print("Quotient: {}, Remainder: {}\n", 
                    .{result.quotient, result.remainder});
    
    // Destructure
    const div_result = divmod(20, 3);
    const q = div_result.quotient;
    const r = div_result.remainder;
    std.debug.print("20 / 3 = {} remainder {}\n", .{q, r});
}
```

### Function Pointers
```zig
fn add(a: i32, b: i32) i32 {
    return a + b;
}

fn multiply(a: i32, b: i32) i32 {
    return a * b;
}

fn apply(operation: *const fn (i32, i32) i32, a: i32, b: i32) i32 {
    return operation(a, b);
}

pub fn main() void {
    const result1 = apply(&add, 5, 3);
    const result2 = apply(&multiply, 5, 3);
    
    std.debug.print("Add: {}, Multiply: {}\n", .{result1, result2});
}
```

### Generic Functions (Compile-time Parameters)
```zig
fn maximum(comptime T: type, a: T, b: T) T {
    return if (a > b) a else b;
}

pub fn main() void {
    const max_int = maximum(i32, 10, 20);
    const max_float = maximum(f64, 3.14, 2.71);
    
    std.debug.print("Max int: {}, Max float: {d:.2}\n", 
                    .{max_int, max_float});
}
```

**Explanation**:
- `comptime T: type`: Compile-time type parameter
- Function is monomorphized at compile time

### Inline Functions
```zig
inline fn square(x: i32) i32 {
    return x * x;
}

pub fn main() void {
    const result = square(5);
    std.debug.print("Square: {}\n", .{result});
}
```

**Explanation**:
- `inline`: Suggests inlining to compiler
- Reduces function call overhead

### Variadic Functions (Compile-time)
```zig
fn sum(args: anytype) i32 {
    var total: i32 = 0;
    inline for (args) |arg| {
        total += arg;
    }
    return total;
}

pub fn main() void {
    const result = sum(.{ 1, 2, 3, 4, 5 });
    std.debug.print("Sum: {}\n", .{result});
}
```

**Explanation**:
- `anytype`: Accepts any type
- Tuple `(.{})` passed as arguments
- `inline for`: Unrolled at compile time

---

## Error Handling Basics

### Error Sets
```zig
const std = @import("std");

// Define error set
const FileError = error{
    FileNotFound,
    PermissionDenied,
    InvalidFormat,
};

// Function that can return errors
fn openFile(name: []const u8) FileError!void {
    if (std.mem.eql(u8, name, "bad")) {
        return FileError.FileNotFound;
    }
    std.debug.print("File '{s}' opened\n", .{name});
}

pub fn main() void {
    openFile("good.txt") catch |err| {
        std.debug.print("Error: {}\n", .{err});
    };
    
    openFile("bad") catch |err| {
        std.debug.print("Error: {}\n", .{err});
    };
}
```

**Explanation**:
- `error { ... }`: Defines an error set
- `ErrorSet!Type`: Function can return error or Type
- `catch |err| { }`: Handle errors

### Try and Catch
```zig
fn riskyOperation(flag: bool) error{Failed}!i32 {
    if (flag) {
        return error.Failed;
    }
    return 42;
}

pub fn main() void {
    // Using catch with default value
    const result1 = riskyOperation(false) catch 0;
    std.debug.print("Result1: {}\n", .{result1});
    
    const result2 = riskyOperation(true) catch 0;
    std.debug.print("Result2: {}\n", .{result2});
    
    // Using catch with error handling
    const result3 = riskyOperation(true) catch |err| {
        std.debug.print("Caught error: {}\n", .{err});
        return;
    };
    std.debug.print("Result3: {}\n", .{result3});
}
```

### Error Propagation with Try
```zig
fn inner() error{Oops}!i32 {
    return error.Oops;
}

fn outer() error{Oops}!i32 {
    const value = try inner(); // Propagates error to caller
    return value;
}

pub fn main() void {
    const result = outer() catch |err| {
        std.debug.print("Error in outer: {}\n", .{err});
        return;
    };
    std.debug.print("Result: {}\n", .{result});
}
```

**Explanation**:
- `try`: Unwraps value or returns error to caller
- Simplifies error propagation

### Error Union
```zig
fn divide(a: i32, b: i32) error{DivisionByZero}!i32 {
    if (b == 0) {
        return error.DivisionByZero;
    }
    return @divTrunc(a, b);
}

pub fn main() void {
    const result1 = divide(10, 2) catch unreachable;
    std.debug.print("10 / 2 = {}\n", .{result1});
    
    const result2 = divide(10, 0) catch |err| blk: {
        std.debug.print("Error: {}\n", .{err});
        break :blk 0;
    };
    std.debug.print("10 / 0 = {}\n", .{result2});
}
```

### Inferred Error Sets
```zig
// Zig can infer error set
fn operation(fail: bool) !i32 {
    if (fail) {
        return error.OperationFailed;
    }
    return 100;
}

pub fn main() void {
    const result = operation(false) catch 0;
    std.debug.print("Result: {}\n", .{result});
}
```

**Explanation**:
- `!Type`: Inferred error set
- Compiler determines possible errors

---

## Arrays and Slices

### Arrays
```zig
const std = @import("std");

pub fn main() void {
    // Array with explicit size
    const numbers: [5]i32 = [5]i32{ 1, 2, 3, 4, 5 };
    
    // Array with inferred size
    const items = [_]i32{ 10, 20, 30 };
    
    // Access elements
    std.debug.print("First: {}, Last: {}\n", .{numbers[0], numbers[4]});
    
    // Array length
    std.debug.print("Length: {}\n", .{numbers.len});
    
    // Iterate
    for (items) |item| {
        std.debug.print("Item: {}\n", .{item});
    }
}
```

**Explanation**:
- `[N]T`: Array of N elements of type T
- `[_]T{...}`: Size inferred from initializer
- `.len`: Array length (compile-time constant)

### Array Initialization
```zig
pub fn main() void {
    // All zeros
    const zeros = [_]i32{0} ** 5; // [0, 0, 0, 0, 0]
    
    // Repeated value
    const fives = [_]i32{5} ** 3; // [5, 5, 5]
    
    // Mixed initialization
    const mixed = [_]i32{ 1, 2 } ++ [_]i32{ 3, 4 };
    
    std.debug.print("Zeros: {any}\n", .{zeros});
    std.debug.print("Fives: {any}\n", .{fives});
    std.debug.print("Mixed: {any}\n", .{mixed});
}
```

**Explanation**:
- `**`: Array repetition operator
- `++`: Array concatenation operator (compile-time)

### Slices
```zig
pub fn main() void {
    const array = [_]i32{ 1, 2, 3, 4, 5 };
    
    // Slice of entire array
    const slice: []const i32 = &array;
    
    // Sub-slice
    const sub = array[1..4]; // Elements 1, 2, 3
    
    std.debug.print("Full slice: {any}\n", .{slice});
    std.debug.print("Sub-slice: {any}\n", .{sub});
    std.debug.print("Slice length: {}\n", .{slice.len});
}
```

**Explanation**:
- `[]T`: Slice of type T (pointer + length)
- `array[start..end]`: Create sub-slice (end is exclusive)
- Slices are fat pointers (pointer + length)

### Mutable Slices
```zig
pub fn main() void {
    var array = [_]i32{ 1, 2, 3, 4, 5 };
    const slice: []i32 = &array;
    
    // Modify through slice
    slice[0] = 10;
    slice[2] = 30;
    
    std.debug.print("Modified: {any}\n", .{array});
}
```

### Multidimensional Arrays
```zig
pub fn main() void {
    // 2D array
    const matrix = [_][3]i32{
        [_]i32{ 1, 2, 3 },
        [_]i32{ 4, 5, 6 },
        [_]i32{ 7, 8, 9 },
    };
    
    // Access elements
    std.debug.print("matrix[1][2] = {}\n", .{matrix[1][2]});
    
    // Iterate
    for (matrix, 0..) |row, i| {
        std.debug.print("Row {}: {any}\n", .{i, row});
    }
}
```

---

## Strings

### String Literals
```zig
const std = @import("std");

pub fn main() void {
    // String literals are []const u8
    const hello: []const u8 = "Hello, World!";
    const multiline =
        \\This is a multiline
        \\string literal using
        \\backslash notation
    ;
    
    std.debug.print("{s}\n", .{hello});
    std.debug.print("{s}\n", .{multiline});
}
```

**Explanation**:
- String literals are `[]const u8` (slice of bytes)
- `\\`: Multiline string literal
- `{s}`: Format specifier for strings

### String Length and Indexing
```zig
pub fn main() void {
    const text = "Hello";
    
    std.debug.print("Length: {}\n", .{text.len});
    std.debug.print("First char: {c}\n", .{text[0]});
    std.debug.print("Last char: {c}\n", .{text[text.len - 1]});
}
```

### String Comparison
```zig
pub fn main() void {
    const str1 = "hello";
    const str2 = "hello";
    const str3 = "world";
    
    // Compare with std.mem.eql
    const equal1 = std.mem.eql(u8, str1, str2);
    const equal2 = std.mem.eql(u8, str1, str3);
    
    std.debug.print("str1 == str2: {}\n", .{equal1});
    std.debug.print("str1 == str3: {}\n", .{equal2});
}
```

**Explanation**:
- Cannot use `==` for string comparison
- Use `std.mem.eql()` for equality
- Compares byte-by-byte

### String Formatting
```zig
pub fn main() !void {
    const stdout = std.io.getStdOut().writer();
    
    const name = "Alice";
    const age: u32 = 30;
    const height: f64 = 5.6;
    
    // Format string
    try stdout.print("Name: {s}, Age: {}, Height: {d:.1}ft\n", 
                     .{name, age, height});
}
```

**Explanation**:
- `{s}`: String format
- `{}`: Default format
- `{d:.1}`: Float with 1 decimal place

### String Concatenation (Compile-time)
```zig
pub fn main() void {
    const hello = "Hello";
    const world = "World";
    
    // Compile-time concatenation
    const greeting = hello ++ ", " ++ world ++ "!";
    
    std.debug.print("{s}\n", .{greeting});
}
```

### UTF-8 Strings
```zig
pub fn main() void {
    const emoji = "Hello 👋 World 🌍";
    
    std.debug.print("String: {s}\n", .{emoji});
    std.debug.print("Byte length: {}\n", .{emoji.len});
    
    // Iterate Unicode codepoints
    var iter = std.unicode.Utf8View.init(emoji) catch unreachable;
    var it = iter.iterator();
    while (it.nextCodepoint()) |codepoint| {
        std.debug.print("Codepoint: U+{X}\n", .{codepoint});
    }
}
```

---

## Structs

### Basic Struct Definition
```zig
const std = @import("std");

const Person = struct {
    name: []const u8,
    age: u32,
    height: f64,
};

pub fn main() void {
    // Create struct instance
    const alice = Person{
        .name = "Alice",
        .age = 30,
        .height = 5.6,
    };
    
    std.debug.print("Name: {s}, Age: {}, Height: {d:.1}\n", 
                    .{alice.name, alice.age, alice.height});
}
```

**Explanation**:
- `struct { }`: Define a struct type
- `.field = value`: Field initialization
- Access fields with `.` operator

### Struct Methods
```zig
const Rectangle = struct {
    width: f64,
    height: f64,
    
    // Method with self parameter
    pub fn area(self: Rectangle) f64 {
        return self.width * self.height;
    }
    
    // Mutating method
    pub fn scale(self: *Rectangle, factor: f64) void {
        self.width *= factor;
        self.height *= factor;
    }
};

pub fn main() void {
    var rect = Rectangle{
        .width = 10.0,
        .height = 5.0,
    };
    
    std.debug.print("Area: {d:.2}\n", .{rect.area()});
    
    rect.scale(2.0);
    std.debug.print("Scaled area: {d:.2}\n", .{rect.area()});
}
```

**Explanation**:
- Methods are functions with `self` parameter
- `self: T`: Immutable method
- `self: *T`: Mutable method (can modify struct)

### Default Field Values
```zig
const Config = struct {
    host: []const u8 = "localhost",
    port: u16 = 8080,
    debug: bool = false,
};

pub fn main() void {
    const config1 = Config{};
    const config2 = Config{ .port = 3000 };
    
    std.debug.print("Config1: {s}:{}\n", .{config1.host, config1.port});
    std.debug.print("Config2: {s}:{}\n", .{config2.host, config2.port});
}
```

### Struct Initialization Functions
```zig
const Point = struct {
    x: f64,
    y: f64,
    
    pub fn init(x: f64, y: f64) Point {
        return Point{ .x = x, .y = y };
    }
    
    pub fn origin() Point {
        return Point{ .x = 0.0, .y = 0.0 };
    }
};

pub fn main() void {
    const p1 = Point.init(3.0, 4.0);
    const p2 = Point.origin();
    
    std.debug.print("P1: ({d:.1}, {d:.1})\n", .{p1.x, p1.y});
    std.debug.print("P2: ({d:.1}, {d:.1})\n", .{p2.x, p2.y});
}
```

### Anonymous Structs
```zig
pub fn main() void {
    // Anonymous struct
    const point = .{ .x = 10, .y = 20 };
    
    std.debug.print("Point: ({}, {})\n", .{point.x, point.y});
    
    // Function returning anonymous struct
    const result = .{ .success = true, .value = 42 };
    std.debug.print("Success: {}, Value: {}\n", .{result.success, result.value});
}
```

### Packed Structs
```zig
const Flags = packed struct {
    read: bool,
    write: bool,
    execute: bool,
    _padding: u5 = 0,
};

pub fn main() void {
    const flags = Flags{
        .read = true,
        .write = false,
        .execute = true,
    };
    
    // Packed struct has guaranteed memory layout
    std.debug.print("Size: {} bytes\n", .{@sizeOf(Flags)});
    std.debug.print("Read: {}, Write: {}, Execute: {}\n", 
                    .{flags.read, flags.write, flags.execute});
}
```

**Explanation**:
- `packed struct`: Guaranteed memory layout, no padding
- Useful for bit manipulation and low-level programming

---

## Enums

### Basic Enums
```zig
const std = @import("std");

const Color = enum {
    Red,
    Green,
    Blue,
};

pub fn main() void {
    const color = Color.Red;
    
    std.debug.print("Color: {}\n", .{color});
    
    // Switch on enum
    switch (color) {
        Color.Red => std.debug.print("Stop!\n", .{}),
        Color.Green => std.debug.print("Go!\n", .{}),
        Color.Blue => std.debug.print("Caution!\n", .{}),
    }
}
```

### Enum with Integer Values
```zig
const Status = enum(u8) {
    OK = 0,
    Error = 1,
    Pending = 2,
};

pub fn main() void {
    const status = Status.OK;
    
    // Get integer value
    const value = @intFromEnum(status);
    std.debug.print("Status value: {}\n", .{value});
    
    // Create from integer
    const from_int: Status = @enumFromInt(1);
    std.debug.print("From int: {}\n", .{from_int});
}
```

**Explanation**:
- `enum(T)`: Enum with specific integer type
- `@intFromEnum()`: Get integer value
- `@enumFromInt()`: Create from integer

### Enum Methods
```zig
const Direction = enum {
    North,
    South,
    East,
    West,
    
    pub fn opposite(self: Direction) Direction {
        return switch (self) {
            .North => .South,
            .South => .North,
            .East => .West,
            .West => .East,
        };
    }
};

pub fn main() void {
    const dir = Direction.North;
    const opp = dir.opposite();
    
    std.debug.print("Direction: {}, Opposite: {}\n", .{dir, opp});
}
```

### Enum with Data (Tagged Union)
```zig
// See Unions section for tagged unions
```

---

## Unions

### Basic Unions
```zig
const std = @import("std");

const Value = union {
    int: i32,
    float: f64,
    boolean: bool,
};

pub fn main() void {
    var value = Value{ .int = 42 };
    
    // Access active field
    std.debug.print("Int value: {}\n", .{value.int});
    
    // Change active field
    value = Value{ .float = 3.14 };
    std.debug.print("Float value: {d:.2}\n", .{value.float});
}
```

**Explanation**:
- Union stores one of several possible types
- Only one field is active at a time
- Accessing wrong field is undefined behavior

### Tagged Unions
```zig
const Value = union(enum) {
    int: i32,
    float: f64,
    boolean: bool,
    
    pub fn print(self: Value) void {
        switch (self) {
            .int => |v| std.debug.print("Int: {}\n", .{v}),
            .float => |v| std.debug.print("Float: {d:.2}\n", .{v}),
            .boolean => |v| std.debug.print("Bool: {}\n", .{v}),
        }
    }
};

pub fn main() void {
    const values = [_]Value{
        Value{ .int = 42 },
        Value{ .float = 3.14 },
        Value{ .boolean = true },
    };
    
    for (values) |value| {
        value.print();
    }
}
```

**Explanation**:
- `union(enum)`: Tagged union with automatic enum tag
- Safe to check which field is active
- Use switch to handle all cases

### Explicit Tag Type
```zig
const Tag = enum {
    IntType,
    FloatType,
    StringType,
};

const Data = union(Tag) {
    IntType: i32,
    FloatType: f64,
    StringType: []const u8,
};

pub fn main() void {
    const data = Data{ .StringType = "Hello" };
    
    // Get tag
    const tag = std.meta.activeTag(data);
    std.debug.print("Tag: {}\n", .{tag});
    
    // Switch on tag
    switch (data) {
        .IntType => |v| std.debug.print("Int: {}\n", .{v}),
        .FloatType => |v| std.debug.print("Float: {}\n", .{v}),
        .StringType => |v| std.debug.print("String: {s}\n", .{v}),
    }
}
```

---

## Pointers

### Single-Item Pointers
```zig
const std = @import("std");

pub fn main() void {
    var x: i32 = 42;
    
    // Get pointer to x
    const ptr: *i32 = &x;
    
    // Dereference pointer
    std.debug.print("Value: {}\n", .{ptr.*});
    
    // Modify through pointer
    ptr.* = 100;
    std.debug.print("Modified: {}\n", .{x});
}
```

**Explanation**:
- `*T`: Pointer to type T
- `&value`: Get pointer to value
- `ptr.*`: Dereference pointer

### Const Pointers
```zig
pub fn main() void {
    const x: i32 = 42;
    
    // Pointer to const
    const ptr: *const i32 = &x;
    
    std.debug.print("Value: {}\n", .{ptr.*});
    
    // ptr.* = 100; // Error: cannot modify through const pointer
}
```

### Many-Item Pointers
```zig
pub fn main() void {
    var array = [_]i32{ 1, 2, 3, 4, 5 };
    
    // Pointer to many items
    const ptr: [*]i32 = &array;
    
    // Index like an array
    std.debug.print("First: {}\n", .{ptr[0]});
    std.debug.print("Second: {}\n", .{ptr[1]});
    
    // Modify
    ptr[0] = 10;
    std.debug.print("Modified: {any}\n", .{array});
}
```

**Explanation**:
- `[*]T`: Pointer to unknown number of items
- No length information (unlike slices)
- Use when working with C APIs

### Pointer Arithmetic
```zig
pub fn main() void {
    var array = [_]i32{ 10, 20, 30, 40 };
    var ptr: [*]i32 = &array;
    
    std.debug.print("ptr[0]: {}\n", .{ptr[0]});
    
    // Advance pointer
    ptr += 1;
    std.debug.print("ptr[0] after advancing: {}\n", .{ptr[0]});
    
    // Pointer difference
    const start: [*]i32 = &array;
    const end = start + array.len;
    const diff = @intFromPtr(end) - @intFromPtr(start);
    std.debug.print("Byte difference: {}\n", .{diff});
}
```

### Sentinel-Terminated Pointers
```zig
pub fn main() void {
    // Null-terminated string
    const str: [*:0]const u8 = "Hello";
    
    var i: usize = 0;
    while (str[i] != 0) : (i += 1) {
        std.debug.print("{c}", .{str[i]});
    }
    std.debug.print("\n", .{});
}
```

**Explanation**:
- `[*:sentinel]T`: Pointer terminated by sentinel value
- Common for C strings (`[*:0]u8`)

---

## Memory Allocation Basics

### Stack Allocation
```zig
const std = @import("std");

pub fn main() void {
    // Stack-allocated array
    var array: [100]i32 = undefined;
    
    // Initialize
    for (&array, 0..) |*item, i| {
        item.* = @intCast(i);
    }
    
    std.debug.print("First: {}, Last: {}\n", .{array[0], array[99]});
}
```

**Explanation**:
- Stack allocation is automatic
- Fast but limited size
- Freed when scope exits

### Heap Allocation with Allocator
```zig
pub fn main() !void {
    // Get general purpose allocator
    var gpa = std.heap.GeneralPurposeAllocator(.{}){};
    defer _ = gpa.deinit();
    
    const allocator = gpa.allocator();
    
    // Allocate single item
    const ptr = try allocator.create(i32);
    defer allocator.destroy(ptr);
    
    ptr.* = 42;
    std.debug.print("Value: {}\n", .{ptr.*});
}
```

**Explanation**:
- `allocator.create()`: Allocate single item
- `allocator.destroy()`: Free single item
- `defer`: Ensure cleanup happens

### Allocating Arrays
```zig
pub fn main() !void {
    var gpa = std.heap.GeneralPurposeAllocator(.{}){};
    defer _ = gpa.deinit();
    
    const allocator = gpa.allocator();
    
    // Allocate array
    const array = try allocator.alloc(i32, 100);
    defer allocator.free(array);
    
    // Initialize
    for (array, 0..) |*item, i| {
        item.* = @intCast(i);
    }
    
    std.debug.print("Length: {}, First: {}, Last: {}\n", 
                    .{array.len, array[0], array[99]});
}
```

**Explanation**:
- `allocator.alloc()`: Allocate slice
- `allocator.free()`: Free slice
- Returns slice, not pointer

### Arena Allocator
```zig
pub fn main() !void {
    var arena = std.heap.ArenaAllocator.init(std.heap.page_allocator);
    defer arena.deinit(); // Frees all at once
    
    const allocator = arena.allocator();
    
    // Allocate multiple items
    const arr1 = try allocator.alloc(i32, 50);
    const arr2 = try allocator.alloc(i32, 100);
    
    // No need to free individually
    // arena.deinit() frees everything
    
    std.debug.print("Allocated arrays: {} and {} items\n", 
                    .{arr1.len, arr2.len});
}
```

**Explanation**:
- Arena allocator for bulk allocations
- Single `deinit()` frees everything
- Useful for request/response patterns

---

## Standard Library Basics

### Printing and Formatting
```zig
const std = @import("std");

pub fn main() !void {
    const stdout = std.io.getStdOut().writer();
    
    // Print to stdout
    try stdout.print("Hello, {s}!\n", .{"World"});
    
    // Format numbers
    try stdout.print("Integer: {}, Float: {d:.2}\n", .{42, 3.14159});
    
    // Binary, octal, hex
    const num: u8 = 42;
    try stdout.print("Binary: {b}, Octal: {o}, Hex: {x}\n", .{num, num, num});
    
    // Debug print (to stderr)
    std.debug.print("Debug: {any}\n", .{[_]i32{1, 2, 3}});
}
```

### Reading Input
```zig
pub fn main() !void {
    const stdin = std.io.getStdIn().reader();
    const stdout = std.io.getStdOut().writer();
    
    try stdout.print("Enter your name: ", .{});
    
    var buffer: [100]u8 = undefined;
    const input = try stdin.readUntilDelimiter(&buffer, '\n');
    
    try stdout.print("Hello, {s}!\n", .{input});
}
```

### ArrayList
```zig
pub fn main() !void {
    var gpa = std.heap.GeneralPurposeAllocator(.{}){};
    defer _ = gpa.deinit();
    
    const allocator = gpa.allocator();
    
    // Create ArrayList
    var list = std.ArrayList(i32).init(allocator);
    defer list.deinit();
    
    // Append items
    try list.append(10);
    try list.append(20);
    try list.append(30);
    
    std.debug.print("List: {any}\n", .{list.items});
    std.debug.print("Length: {}\n", .{list.items.len});
    
    // Pop item
    const last = list.pop();
    std.debug.print("Popped: {}\n", .{last});
}
```

### HashMap
```zig
pub fn main() !void {
    var gpa = std.heap.GeneralPurposeAllocator(.{}){};
    defer _ = gpa.deinit();
    
    const allocator = gpa.allocator();
    
    // Create HashMap
    var map = std.StringHashMap(i32).init(allocator);
    defer map.deinit();
    
    // Insert entries
    try map.put("apple", 5);
    try map.put("banana", 3);
    try map.put("orange", 7);
    
    // Get value
    if (map.get("apple")) |value| {
        std.debug.print("Apples: {}\n", .{value});
    }
    
    // Iterate
    var iter = map.iterator();
    while (iter.next()) |entry| {
        std.debug.print("{s}: {}\n", .{entry.key_ptr.*, entry.value_ptr.*});
    }
}
```

### File I/O
```zig
pub fn main() !void {
    // Write to file
    {
        const file = try std.fs.cwd().createFile("test.txt", .{});
        defer file.close();
        
        try file.writeAll("Hello, File!\n");
    }
    
    // Read from file
    {
        const file = try std.fs.cwd().openFile("test.txt", .{});
        defer file.close();
        
        var buffer: [1024]u8 = undefined;
        const bytes_read = try file.readAll(&buffer);
        
        std.debug.print("Read: {s}", .{buffer[0..bytes_read]});
    }
}
```

### Testing
```zig
const std = @import("std");
const testing = std.testing;

fn add(a: i32, b: i32) i32 {
    return a + b;
}

test "add function" {
    const result = add(2, 3);
    try testing.expectEqual(@as(i32, 5), result);
}

test "add negative numbers" {
    const result = add(-5, 3);
    try testing.expectEqual(@as(i32, -2), result);
}
```

**Run tests**:
```bash
zig test file.zig
```

---

## Summary

This basic tutorial covered:
- ✅ Installation and setup
- ✅ Basic syntax and structure
- ✅ Variables and constants
- ✅ Data types (integers, floats, booleans, etc.)
- ✅ Operators (arithmetic, bitwise, logical, comparison)
- ✅ Control flow (if, while, for, switch)
- ✅ Functions and parameters
- ✅ Error handling basics
- ✅ Arrays and slices
- ✅ Strings and UTF-8
- ✅ Structs and methods
- ✅ Enums and unions
- ✅ Pointers and references
- ✅ Memory allocation basics
- ✅ Standard library essentials

**Next Steps**: Continue to the Intermediate tutorial for:
- Advanced error handling
- Comptime programming
- Generic data structures
- Async/await
- C interoperability
- Build system
- And more!

---

**Practice Exercises**:

1. Create a calculator that performs basic arithmetic operations
2. Implement a struct for a `BankAccount` with deposit/withdraw methods
3. Write a function that finds the maximum value in an array
4. Create a tagged union for different geometric shapes (Circle, Rectangle, Triangle)
5. Implement a simple text file reader with error handling
