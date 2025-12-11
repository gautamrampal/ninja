# Rust Complete Mastery Tutorial - Advanced

## Table of Contents
1. [Advanced Type System](#advanced-type-system)
2. [Macros](#macros)
3. [Unsafe Rust](#unsafe-rust)
4. [Advanced Traits](#advanced-traits)
5. [Advanced Patterns](#advanced-patterns)
6. [Advanced Functions and Closures](#advanced-functions-and-closures)
7. [Advanced Lifetimes](#advanced-lifetimes)
8. [Advanced Concurrency](#advanced-concurrency)
9. [Async Programming](#async-programming)
10. [Foreign Function Interface (FFI)](#foreign-function-interface-ffi)
11. [Performance Optimization](#performance-optimization)
12. [Advanced Error Handling](#advanced-error-handling)
13. [Type-Level Programming](#type-level-programming)
14. [Advanced Memory Management](#advanced-memory-management)
15. [Production-Ready Patterns](#production-ready-patterns)

---

## Advanced Type System

### Newtype Pattern
```rust
// Create a wrapper type for type safety
struct Kilometers(u32);
struct Miles(u32);

impl Kilometers {
    fn to_miles(&self) -> Miles {
        Miles((self.0 as f64 * 0.621371) as u32)
    }
}

impl Miles {
    fn to_kilometers(&self) -> Kilometers {
        Kilometers((self.0 as f64 * 1.60934) as u32)
    }
}

fn main() {
    let distance_km = Kilometers(100);
    let distance_miles = distance_km.to_miles();
    
    println!("100 km = {} miles", distance_miles.0);
    
    // Can't accidentally mix types
    // let sum = distance_km.0 + distance_miles.0;  // This compiles but is semantically wrong
    // Better to prevent this at type level
}
```

### Type Aliases
```rust
type Kilometers = i32;
type Thunk = Box<dyn Fn() + Send + 'static>;

fn main() {
    let x: Kilometers = 5;
    let y: i32 = 10;
    
    println!("x + y = {}", x + y);  // Both are i32
    
    let f: Thunk = Box::new(|| println!("Hello!"));
    f();
}
```

### Never Type (!)
```rust
fn main() {
    // ! means "never returns"
    let x: i32 = match "123".parse() {
        Ok(num) => num,
        Err(_) => panic!("Failed to parse"),  // panic! has type !
    };
    
    println!("x = {}", x);
}

// Function that never returns
fn infinite_loop() -> ! {
    loop {
        println!("Forever...");
    }
}

// Another example
fn quit(code: i32) -> ! {
    std::process::exit(code);
}
```

### Dynamically Sized Types (DST)
```rust
fn main() {
    // str is a DST (size unknown at compile time)
    // Always use &str (reference to str)
    let s: &str = "hello";
    
    // [T] is a DST
    // Always use &[T] or Box<[T]>
    let numbers: &[i32] = &[1, 2, 3];
    
    // Trait objects are DSTs
    trait MyTrait {
        fn method(&self);
    }
    
    struct MyType;
    impl MyTrait for MyType {
        fn method(&self) {
            println!("Method called");
        }
    }
    
    let obj: &dyn MyTrait = &MyType;
    obj.method();
}
```

### Sized Trait
```rust
// T: Sized is implicit for generic types
fn generic<T>(t: T) {
    // T must have known size at compile time
}

// Relax the Sized bound with ?Sized
fn relaxed<T: ?Sized>(t: &T) {
    // T might not have known size
    // Must use reference or pointer
}

fn main() {
    let s = "hello";
    relaxed(s);  // str doesn't have Sized
}
```

---

## Macros

### Declarative Macros with macro_rules!
```rust
// Simple macro
macro_rules! say_hello {
    () => {
        println!("Hello!");
    };
}

// Macro with parameters
macro_rules! create_function {
    ($func_name:ident) => {
        fn $func_name() {
            println!("Function {:?} was called", stringify!($func_name));
        }
    };
}

// Macro with multiple patterns
macro_rules! vec_of_strings {
    ($($x:expr),*) => {
        {
            let mut temp_vec = Vec::new();
            $(
                temp_vec.push($x.to_string());
            )*
            temp_vec
        }
    };
}

fn main() {
    say_hello!();
    
    create_function!(foo);
    foo();
    
    let v = vec_of_strings!["hello", "world", "rust"];
    println!("{:?}", v);
}
```

### Advanced Macro Patterns
```rust
// Implementing a custom vec! macro
macro_rules! my_vec {
    () => {
        Vec::new()
    };
    ($($x:expr),+ $(,)?) => {
        {
            let mut temp_vec = Vec::new();
            $(
                temp_vec.push($x);
            )+
            temp_vec
        }
    };
}

// Count elements at compile time
macro_rules! count {
    () => (0usize);
    ($x:tt $($xs:tt)*) => (1usize + count!($($xs)*));
}

// Match different types
macro_rules! type_of {
    ($x:expr) => {{
        fn type_name_of<T>(_: &T) -> &'static str {
            std::any::type_name::<T>()
        }
        type_name_of(&$x)
    }};
}

fn main() {
    let v1 = my_vec![];
    let v2 = my_vec![1, 2, 3];
    let v3 = my_vec![1, 2, 3,];  // Trailing comma allowed
    
    println!("Count: {}", count!(a b c d e));
    
    let x = 5;
    let y = "hello";
    println!("Type of x: {}", type_of!(x));
    println!("Type of y: {}", type_of!(y));
}
```

### Procedural Macros (Overview)
```rust
// Procedural macros are defined in separate crates
// Three types:
// 1. Custom derive macros (#[derive(MyMacro)])
// 2. Attribute-like macros (#[my_macro])
// 3. Function-like macros (my_macro!(...))

// Example: Custom derive macro (requires separate crate)
/*
// In a separate crate with proc-macro = true
use proc_macro::TokenStream;
use quote::quote;
use syn;

#[proc_macro_derive(MyTrait)]
pub fn my_trait_derive(input: TokenStream) -> TokenStream {
    let ast = syn::parse(input).unwrap();
    impl_my_trait(&ast)
}

fn impl_my_trait(ast: &syn::DeriveInput) -> TokenStream {
    let name = &ast.ident;
    let gen = quote! {
        impl MyTrait for #name {
            fn my_method(&self) {
                println!("MyTrait method called");
            }
        }
    };
    gen.into()
}
*/

fn main() {
    println!("Procedural macros require separate crate setup");
}
```

### Macro Debugging
```rust
macro_rules! debug_macro {
    ($($x:tt)*) => {
        {
            println!("Macro input: {}", stringify!($($x)*));
            $($x)*
        }
    };
}

fn main() {
    debug_macro!(let x = 5 + 3);
    println!("x = {}", x);
    
    // Use cargo expand to see macro expansion
    // cargo install cargo-expand
    // cargo expand
}
```

---

## Unsafe Rust

### Unsafe Superpowers
```rust
fn main() {
    // Five unsafe superpowers:
    // 1. Dereference raw pointers
    // 2. Call unsafe functions
    // 3. Access/modify mutable static variables
    // 4. Implement unsafe traits
    // 5. Access fields of unions
}
```

### Raw Pointers
```rust
fn main() {
    let mut num = 5;
    
    // Create raw pointers (safe)
    let r1 = &num as *const i32;
    let r2 = &mut num as *mut i32;
    
    // Dereference raw pointers (unsafe)
    unsafe {
        println!("r1: {}", *r1);
        println!("r2: {}", *r2);
        
        *r2 = 10;
        println!("num: {}", num);
    }
    
    // Can create raw pointers to arbitrary memory
    let address = 0x012345usize;
    let r = address as *const i32;
    
    // Dereferencing this would be undefined behavior
    // unsafe {
    //     println!("{}", *r);
    // }
}
```

### Unsafe Functions
```rust
unsafe fn dangerous() {
    println!("This is dangerous!");
}

fn split_at_mut(slice: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
    let len = slice.len();
    let ptr = slice.as_mut_ptr();
    
    assert!(mid <= len);
    
    unsafe {
        (
            std::slice::from_raw_parts_mut(ptr, mid),
            std::slice::from_raw_parts_mut(ptr.add(mid), len - mid),
        )
    }
}

fn main() {
    unsafe {
        dangerous();
    }
    
    let mut v = vec![1, 2, 3, 4, 5, 6];
    let (left, right) = split_at_mut(&mut v, 3);
    
    println!("Left: {:?}", left);
    println!("Right: {:?}", right);
}
```

### Calling External Functions (FFI Preview)
```rust
extern "C" {
    fn abs(input: i32) -> i32;
}

fn main() {
    unsafe {
        println!("Absolute value of -3: {}", abs(-3));
    }
}
```

### Mutable Static Variables
```rust
static mut COUNTER: u32 = 0;

fn add_to_count(inc: u32) {
    unsafe {
        COUNTER += inc;
    }
}

fn main() {
    add_to_count(3);
    
    unsafe {
        println!("COUNTER: {}", COUNTER);
    }
}
```

### Unsafe Traits
```rust
unsafe trait Foo {
    // Trait methods
}

unsafe impl Foo for i32 {
    // Implementation
}

fn main() {
    // Send and Sync are unsafe traits
    // Implementing them for types with raw pointers requires unsafe
}
```

### Union Types
```rust
union MyUnion {
    f1: u32,
    f2: f32,
}

fn main() {
    let u = MyUnion { f1: 1 };
    
    unsafe {
        let f = u.f1;
        println!("f1: {}", f);
        
        // Accessing f2 would be unsafe and likely garbage
        // let f2 = u.f2;
    }
}
```

---

## Advanced Traits

### Associated Types
```rust
trait Iterator {
    type Item;  // Associated type
    
    fn next(&mut self) -> Option<Self::Item>;
}

struct Counter {
    count: u32,
}

impl Iterator for Counter {
    type Item = u32;  // Specify associated type
    
    fn next(&mut self) -> Option<Self::Item> {
        self.count += 1;
        if self.count < 6 {
            Some(self.count)
        } else {
            None
        }
    }
}

fn main() {
    let mut counter = Counter { count: 0 };
    
    while let Some(n) = counter.next() {
        println!("{}", n);
    }
}
```

### Associated Types vs Generic Types
```rust
// With generics - can implement multiple times
trait IteratorGeneric<T> {
    fn next(&mut self) -> Option<T>;
}

// With associated types - can only implement once
trait IteratorAssoc {
    type Item;
    fn next(&mut self) -> Option<Self::Item>;
}

fn main() {
    println!("Associated types enforce single implementation");
}
```

### Default Generic Type Parameters
```rust
use std::ops::Add;

#[derive(Debug, PartialEq)]
struct Point {
    x: i32,
    y: i32,
}

impl Add for Point {
    type Output = Point;
    
    fn add(self, other: Point) -> Point {
        Point {
            x: self.x + other.x,
            y: self.y + other.y,
        }
    }
}

// Add trait definition (simplified):
// trait Add<Rhs = Self> {  // Rhs defaults to Self
//     type Output;
//     fn add(self, rhs: Rhs) -> Self::Output;
// }

// Custom implementation with different Rhs
struct Millimeters(u32);
struct Meters(u32);

impl Add<Meters> for Millimeters {
    type Output = Millimeters;
    
    fn add(self, other: Meters) -> Millimeters {
        Millimeters(self.0 + (other.0 * 1000))
    }
}

fn main() {
    let p1 = Point { x: 1, y: 0 };
    let p2 = Point { x: 2, y: 3 };
    
    assert_eq!(p1 + p2, Point { x: 3, y: 3 });
    
    let mm = Millimeters(1000);
    let m = Meters(1);
    let total = mm + m;
    println!("Total: {} mm", total.0);
}
```

### Fully Qualified Syntax
```rust
trait Pilot {
    fn fly(&self);
}

trait Wizard {
    fn fly(&self);
}

struct Human;

impl Pilot for Human {
    fn fly(&self) {
        println!("This is your captain speaking.");
    }
}

impl Wizard for Human {
    fn fly(&self) {
        println!("Up!");
    }
}

impl Human {
    fn fly(&self) {
        println!("*waving arms furiously*");
    }
}

fn main() {
    let person = Human;
    
    // Ambiguous - calls Human::fly
    person.fly();
    
    // Fully qualified syntax
    Pilot::fly(&person);
    Wizard::fly(&person);
    
    // Alternative syntax
    <Human as Pilot>::fly(&person);
    <Human as Wizard>::fly(&person);
}
```

### Supertraits
```rust
use std::fmt;

trait OutlinePrint: fmt::Display {
    fn outline_print(&self) {
        let output = self.to_string();
        let len = output.len();
        println!("{}", "*".repeat(len + 4));
        println!("*{}*", " ".repeat(len + 2));
        println!("* {} *", output);
        println!("*{}*", " ".repeat(len + 2));
        println!("{}", "*".repeat(len + 4));
    }
}

struct Point {
    x: i32,
    y: i32,
}

impl fmt::Display for Point {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "({}, {})", self.x, self.y)
    }
}

impl OutlinePrint for Point {}

fn main() {
    let p = Point { x: 3, y: 4 };
    p.outline_print();
}
```

### Newtype Pattern for External Traits
```rust
use std::fmt;

// Can't implement external trait on external type
// impl fmt::Display for Vec<String> { ... }  // Error

// But can use newtype pattern
struct Wrapper(Vec<String>);

impl fmt::Display for Wrapper {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "[{}]", self.0.join(", "))
    }
}

fn main() {
    let w = Wrapper(vec![String::from("hello"), String::from("world")]);
    println!("w = {}", w);
}
```

### Trait Object Safety
```rust
// Trait objects require object-safe traits
// A trait is object-safe if:
// 1. Return type isn't Self
// 2. No generic type parameters

trait Clone {
    fn clone(&self) -> Self;  // Not object-safe (returns Self)
}

trait Draw {
    fn draw(&self);  // Object-safe
}

struct Button;

impl Draw for Button {
    fn draw(&self) {
        println!("Drawing button");
    }
}

fn main() {
    let button = Button;
    let drawable: &dyn Draw = &button;  // OK
    drawable.draw();
    
    // let clonable: &dyn Clone = &button;  // Error: Clone not object-safe
}
```

---

## Advanced Patterns

### Pattern Matching Everywhere
```rust
fn main() {
    // In function parameters
    fn print_coordinates(&(x, y): &(i32, i32)) {
        println!("x: {}, y: {}", x, y);
    }
    
    let point = (3, 5);
    print_coordinates(&point);
    
    // In closures
    let numbers = vec![(1, 2), (3, 4), (5, 6)];
    let sum: i32 = numbers.iter()
        .map(|(x, y)| x + y)
        .sum();
    println!("Sum: {}", sum);
}
```

### Refutability
```rust
fn main() {
    // Irrefutable patterns (always match)
    let x = 5;  // Pattern always matches
    let (x, y, z) = (1, 2, 3);  // Pattern always matches
    
    // Refutable patterns (might not match)
    let v = vec![1, 2, 3];
    // let [x, y] = v;  // Error: pattern might not match
    
    // if let for refutable patterns
    if let [x, y] = &v[..2] {
        println!("x: {}, y: {}", x, y);
    }
    
    // Match arms must be refutable
    match v.get(0) {
        Some(x) => println!("First: {}", x),
        None => println!("No first element"),
    }
}
```

### Pattern Syntax

#### Matching Literals
```rust
fn main() {
    let x = 1;
    
    match x {
        1 => println!("one"),
        2 => println!("two"),
        3 => println!("three"),
        _ => println!("anything"),
    }
}
```

#### Matching Named Variables
```rust
fn main() {
    let x = Some(5);
    let y = 10;
    
    match x {
        Some(50) => println!("Got 50"),
        Some(y) => println!("Matched, y = {:?}", y),  // New y shadows outer y
        _ => println!("Default case, x = {:?}", x),
    }
    
    println!("at the end: x = {:?}, y = {:?}", x, y);
}
```

#### Multiple Patterns
```rust
fn main() {
    let x = 1;
    
    match x {
        1 | 2 => println!("one or two"),
        3 => println!("three"),
        _ => println!("anything"),
    }
}
```

#### Matching Ranges
```rust
fn main() {
    let x = 5;
    
    match x {
        1..=5 => println!("one through five"),
        _ => println!("something else"),
    }
    
    let x = 'c';
    
    match x {
        'a'..='j' => println!("early ASCII letter"),
        'k'..='z' => println!("late ASCII letter"),
        _ => println!("something else"),
    }
}
```

#### Destructuring Structs
```rust
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p = Point { x: 0, y: 7 };
    
    let Point { x: a, y: b } = p;
    println!("a: {}, b: {}", a, b);
    
    // Shorthand
    let Point { x, y } = p;
    println!("x: {}, y: {}", x, y);
    
    // Match specific values
    match p {
        Point { x, y: 0 } => println!("On the x axis at {}", x),
        Point { x: 0, y } => println!("On the y axis at {}", y),
        Point { x, y } => println!("On neither axis: ({}, {})", x, y),
    }
}
```

#### Destructuring Enums
```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

fn main() {
    let msg = Message::ChangeColor(0, 160, 255);
    
    match msg {
        Message::Quit => println!("Quit"),
        Message::Move { x, y } => println!("Move to ({}, {})", x, y),
        Message::Write(text) => println!("Text: {}", text),
        Message::ChangeColor(r, g, b) => println!("RGB({}, {}, {})", r, g, b),
    }
}
```

#### Nested Destructuring
```rust
enum Color {
    Rgb(i32, i32, i32),
    Hsv(i32, i32, i32),
}

enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(Color),
}

fn main() {
    let msg = Message::ChangeColor(Color::Hsv(0, 160, 255));
    
    match msg {
        Message::ChangeColor(Color::Rgb(r, g, b)) => {
            println!("RGB({}, {}, {})", r, g, b);
        }
        Message::ChangeColor(Color::Hsv(h, s, v)) => {
            println!("HSV({}, {}, {})", h, s, v);
        }
        _ => (),
    }
}
```

#### Ignoring Values
```rust
fn main() {
    // Ignore entire value
    let _ = 5;
    
    // Ignore parts of a value
    let (x, _, z) = (1, 2, 3);
    println!("x: {}, z: {}", x, z);
    
    // Ignore remaining parts
    let numbers = (2, 4, 8, 16, 32);
    match numbers {
        (first, .., last) => {
            println!("First: {}, Last: {}", first, last);
        }
    }
    
    // Ignore unused variable (suppress warning)
    let _unused = 5;
}
```

#### Match Guards
```rust
fn main() {
    let num = Some(4);
    
    match num {
        Some(x) if x < 5 => println!("less than five: {}", x),
        Some(x) => println!("{}", x),
        None => (),
    }
    
    let x = 4;
    let y = false;
    
    match x {
        4 | 5 | 6 if y => println!("yes"),
        _ => println!("no"),
    }
}
```

#### @ Bindings
```rust
enum Message {
    Hello { id: i32 },
}

fn main() {
    let msg = Message::Hello { id: 5 };
    
    match msg {
        Message::Hello {
            id: id_variable @ 3..=7,
        } => println!("Found an id in range: {}", id_variable),
        Message::Hello { id: 10..=12 } => {
            println!("Found an id in another range")
        }
        Message::Hello { id } => println!("Found some other id: {}", id),
    }
}
```

---

## Advanced Functions and Closures

### Function Pointers
```rust
fn add_one(x: i32) -> i32 {
    x + 1
}

fn do_twice(f: fn(i32) -> i32, arg: i32) -> i32 {
    f(arg) + f(arg)
}

fn main() {
    let answer = do_twice(add_one, 5);
    println!("Answer: {}", answer);
    
    // Function pointers implement all three closure traits
    let numbers = vec![1, 2, 3];
    
    // Using closure
    let strings: Vec<String> = numbers.iter()
        .map(|i| i.to_string())
        .collect();
    
    // Using function pointer
    let strings: Vec<String> = numbers.iter()
        .map(ToString::to_string)
        .collect();
    
    println!("{:?}", strings);
}
```

### Returning Closures
```rust
fn returns_closure() -> Box<dyn Fn(i32) -> i32> {
    Box::new(|x| x + 1)
}

fn main() {
    let f = returns_closure();
    println!("Result: {}", f(5));
}
```

### Higher-Order Functions
```rust
fn compose<F, G, A, B, C>(f: F, g: G) -> impl Fn(A) -> C
where
    F: Fn(A) -> B,
    G: Fn(B) -> C,
{
    move |x| g(f(x))
}

fn main() {
    let add_one = |x: i32| x + 1;
    let double = |x: i32| x * 2;
    
    let add_one_then_double = compose(add_one, double);
    
    println!("Result: {}", add_one_then_double(5));  // (5 + 1) * 2 = 12
}
```

---

## Advanced Lifetimes

### Lifetime Subtyping
```rust
// 'a: 'b means 'a outlives 'b
fn parser<'a, 'b>(s: &'a str) -> &'b str
where
    'a: 'b,
{
    &s[..5]
}

fn main() {
    let string = String::from("hello world");
    let result = parser(&string);
    println!("Result: {}", result);
}
```

### Lifetime Bounds
```rust
use std::fmt::Display;

fn longest_with_announcement<'a, T>(
    x: &'a str,
    y: &'a str,
    ann: T,
) -> &'a str
where
    T: Display,
{
    println!("Announcement! {}", ann);
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

fn main() {
    let s1 = String::from("long string");
    let s2 = "short";
    
    let result = longest_with_announcement(&s1, s2, "Comparing strings");
    println!("Longest: {}", result);
}
```

### Higher-Rank Trait Bounds (HRTB)
```rust
// for<'a> is a higher-rank trait bound
fn call_with_ref<F>(f: F)
where
    F: for<'a> Fn(&'a i32),
{
    let value = 42;
    f(&value);
}

fn main() {
    call_with_ref(|x| println!("Value: {}", x));
}
```

---

## Advanced Concurrency

### Atomic Types
```rust
use std::sync::atomic::{AtomicUsize, Ordering};
use std::sync::Arc;
use std::thread;

fn main() {
    let counter = Arc::new(AtomicUsize::new(0));
    let mut handles = vec![];
    
    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            for _ in 0..1000 {
                counter.fetch_add(1, Ordering::SeqCst);
            }
        });
        handles.push(handle);
    }
    
    for handle in handles {
        handle.join().unwrap();
    }
    
    println!("Result: {}", counter.load(Ordering::SeqCst));
}
```

### Lock-Free Data Structures
```rust
use std::sync::atomic::{AtomicPtr, Ordering};
use std::ptr;

struct Node<T> {
    data: T,
    next: AtomicPtr<Node<T>>,
}

struct Stack<T> {
    head: AtomicPtr<Node<T>>,
}

impl<T> Stack<T> {
    fn new() -> Stack<T> {
        Stack {
            head: AtomicPtr::new(ptr::null_mut()),
        }
    }
    
    fn push(&self, data: T) {
        let new_node = Box::into_raw(Box::new(Node {
            data,
            next: AtomicPtr::new(ptr::null_mut()),
        }));
        
        loop {
            let head = self.head.load(Ordering::Acquire);
            unsafe {
                (*new_node).next.store(head, Ordering::Release);
            }
            
            if self.head
                .compare_exchange(head, new_node, Ordering::Release, Ordering::Acquire)
                .is_ok()
            {
                break;
            }
        }
    }
}

fn main() {
    let stack = Stack::new();
    stack.push(1);
    stack.push(2);
    stack.push(3);
    
    println!("Lock-free stack created");
}
```

### Scoped Threads
```rust
use std::thread;

fn main() {
    let mut data = vec![1, 2, 3];
    
    thread::scope(|s| {
        s.spawn(|| {
            println!("Length: {}", data.len());
        });
        
        s.spawn(|| {
            for x in &data {
                println!("Value: {}", x);
            }
        });
    });
    
    data.push(4);
    println!("Data: {:?}", data);
}
```

---

## Async Programming

### Basic async/await
```rust
use std::future::Future;
use std::pin::Pin;
use std::task::{Context, Poll};

// Async function
async fn hello_world() {
    println!("Hello, async world!");
}

// Async block
fn async_block_example() -> impl Future<Output = ()> {
    async {
        println!("In async block");
    }
}

// Manual Future implementation
struct MyFuture;

impl Future for MyFuture {
    type Output = i32;
    
    fn poll(self: Pin<&mut Self>, _cx: &mut Context<'_>) -> Poll<Self::Output> {
        Poll::Ready(42)
    }
}

fn main() {
    // Note: Need an async runtime like tokio or async-std to actually run
    println!("Async examples (need runtime to execute)");
}
```

### Using Tokio Runtime
```rust
// Add to Cargo.toml:
// [dependencies]
// tokio = { version = "1", features = ["full"] }

/*
use tokio::time::{sleep, Duration};

#[tokio::main]
async fn main() {
    let handle = tokio::spawn(async {
        sleep(Duration::from_secs(1)).await;
        println!("Task completed!");
    });
    
    println!("Main thread continues");
    
    handle.await.unwrap();
}
*/

fn main() {
    println!("Add tokio dependency to run async code");
}
```

### Async Traits and Combinators
```rust
/*
use tokio::time::{sleep, Duration};

async fn fetch_data() -> String {
    sleep(Duration::from_secs(1)).await;
    String::from("Data fetched")
}

async fn process_data(data: String) -> String {
    sleep(Duration::from_secs(1)).await;
    format!("Processed: {}", data)
}

#[tokio::main]
async fn main() {
    // Sequential
    let data = fetch_data().await;
    let result = process_data(data).await;
    println!("{}", result);
    
    // Parallel
    let (data1, data2) = tokio::join!(
        fetch_data(),
        fetch_data(),
    );
    
    println!("data1: {}, data2: {}", data1, data2);
}
*/

fn main() {
    println!("Async combinators example");
}
```

---

## Foreign Function Interface (FFI)

### Calling C from Rust
```rust
// Link to C library
#[link(name = "m")]  // libm (math library)
extern "C" {
    fn sqrt(x: f64) -> f64;
    fn pow(x: f64, y: f64) -> f64;
}

fn main() {
    unsafe {
        let x = 4.0;
        println!("sqrt({}) = {}", x, sqrt(x));
        println!("pow(2, 3) = {}", pow(2.0, 3.0));
    }
}
```

### Calling Rust from C
```rust
#[no_mangle]
pub extern "C" fn rust_function(x: i32) -> i32 {
    x * 2
}

// In C:
// extern int rust_function(int x);
// int result = rust_function(21);  // Returns 42

fn main() {
    println!("Rust function callable from C");
}
```

### Working with C Strings
```rust
use std::ffi::{CStr, CString};
use std::os::raw::c_char;

extern "C" {
    fn strlen(s: *const c_char) -> usize;
}

fn main() {
    // Rust string to C string
    let rust_str = "Hello, C!";
    let c_str = CString::new(rust_str).unwrap();
    
    unsafe {
        let len = strlen(c_str.as_ptr());
        println!("Length: {}", len);
    }
    
    // C string to Rust string
    let c_str_raw = c_str.as_ptr();
    let rust_str = unsafe {
        CStr::from_ptr(c_str_raw).to_str().unwrap()
    };
    
    println!("String: {}", rust_str);
}
```

---

## Performance Optimization

### Zero-Cost Abstractions
```rust
fn main() {
    // Iterator version (zero-cost abstraction)
    let numbers: Vec<_> = (0..1000).collect();
    let sum: i32 = numbers.iter().sum();
    
    // Manual loop (compiles to same assembly)
    let mut total = 0;
    for i in 0..1000 {
        total += i;
    }
    
    assert_eq!(sum, total);
}
```

### Inline Hints
```rust
#[inline]
fn add(a: i32, b: i32) -> i32 {
    a + b
}

#[inline(always)]
fn multiply(a: i32, b: i32) -> i32 {
    a * b
}

#[inline(never)]
fn divide(a: i32, b: i32) -> i32 {
    a / b
}

fn main() {
    let result = add(multiply(5, 3), divide(10, 2));
    println!("Result: {}", result);
}
```

### SIMD (Single Instruction Multiple Data)
```rust
// Requires nightly Rust
/*
#![feature(portable_simd)]
use std::simd::f32x4;

fn main() {
    let a = f32x4::from_array([1.0, 2.0, 3.0, 4.0]);
    let b = f32x4::from_array([5.0, 6.0, 7.0, 8.0]);
    
    let sum = a + b;
    println!("SIMD sum: {:?}", sum.to_array());
}
*/

fn main() {
    println!("SIMD requires nightly Rust");
}
```

### Avoiding Allocations
```rust
use std::borrow::Cow;

fn process_string(s: &str) -> Cow<str> {
    if s.contains(' ') {
        Cow::Owned(s.replace(' ', "_"))
    } else {
        Cow::Borrowed(s)
    }
}

fn main() {
    let s1 = "hello";
    let s2 = "hello world";
    
    let r1 = process_string(s1);  // No allocation
    let r2 = process_string(s2);  // Allocation
    
    println!("r1: {}, r2: {}", r1, r2);
}
```

### Benchmarking
```rust
// Use criterion crate for benchmarking
/*
// In Cargo.toml:
[dev-dependencies]
criterion = "0.5"

[[bench]]
name = "my_benchmark"
harness = false

// In benches/my_benchmark.rs:
use criterion::{black_box, criterion_group, criterion_main, Criterion};

fn fibonacci(n: u64) -> u64 {
    match n {
        0 => 1,
        1 => 1,
        n => fibonacci(n-1) + fibonacci(n-2),
    }
}

fn criterion_benchmark(c: &mut Criterion) {
    c.bench_function("fib 20", |b| b.iter(|| fibonacci(black_box(20))));
}

criterion_group!(benches, criterion_benchmark);
criterion_main!(benches);
*/

fn main() {
    println!("Use criterion crate for benchmarking");
}
```

---

## Advanced Error Handling

### Custom Error Types
```rust
use std::fmt;
use std::error::Error;

#[derive(Debug)]
enum MyError {
    IoError(std::io::Error),
    ParseError(std::num::ParseIntError),
    Custom(String),
}

impl fmt::Display for MyError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            MyError::IoError(e) => write!(f, "IO error: {}", e),
            MyError::ParseError(e) => write!(f, "Parse error: {}", e),
            MyError::Custom(msg) => write!(f, "Custom error: {}", msg),
        }
    }
}

impl Error for MyError {
    fn source(&self) -> Option<&(dyn Error + 'static)> {
        match self {
            MyError::IoError(e) => Some(e),
            MyError::ParseError(e) => Some(e),
            MyError::Custom(_) => None,
        }
    }
}

impl From<std::io::Error> for MyError {
    fn from(error: std::io::Error) -> Self {
        MyError::IoError(error)
    }
}

impl From<std::num::ParseIntError> for MyError {
    fn from(error: std::num::ParseIntError) -> Self {
        MyError::ParseError(error)
    }
}

fn do_something() -> Result<i32, MyError> {
    let num: i32 = "42".parse()?;
    Ok(num)
}

fn main() {
    match do_something() {
        Ok(n) => println!("Success: {}", n),
        Err(e) => println!("Error: {}", e),
    }
}
```

### Using thiserror and anyhow
```rust
// Add to Cargo.toml:
// thiserror = "1.0"
// anyhow = "1.0"

/*
use thiserror::Error;

#[derive(Error, Debug)]
enum DataStoreError {
    #[error("data store disconnected")]
    Disconnect(#[from] io::Error),
    #[error("the data for key `{0}` is not available")]
    Redaction(String),
    #[error("invalid header (expected {expected:?}, found {found:?})")]
    InvalidHeader {
        expected: String,
        found: String,
    },
    #[error("unknown data store error")]
    Unknown,
}

use anyhow::{Context, Result};

fn read_config() -> Result<String> {
    std::fs::read_to_string("config.txt")
        .context("Failed to read config file")
}
*/

fn main() {
    println!("Use thiserror and anyhow for better error handling");
}
```

---

## Type-Level Programming

### Phantom Types
```rust
use std::marker::PhantomData;

struct Kilometers;
struct Miles;

struct Distance<Unit> {
    value: f64,
    _unit: PhantomData<Unit>,
}

impl Distance<Kilometers> {
    fn to_miles(self) -> Distance<Miles> {
        Distance {
            value: self.value * 0.621371,
            _unit: PhantomData,
        }
    }
}

impl Distance<Miles> {
    fn to_kilometers(self) -> Distance<Kilometers> {
        Distance {
            value: self.value * 1.60934,
            _unit: PhantomData,
        }
    }
}

fn main() {
    let distance_km = Distance::<Kilometers> {
        value: 100.0,
        _unit: PhantomData,
    };
    
    let distance_miles = distance_km.to_miles();
    println!("100 km = {} miles", distance_miles.value);
}
```

### Type-State Pattern
```rust
struct Locked;
struct Unlocked;

struct Door<State> {
    _state: PhantomData<State>,
}

impl Door<Locked> {
    fn new() -> Self {
        Door { _state: PhantomData }
    }
    
    fn unlock(self) -> Door<Unlocked> {
        println!("Door unlocked");
        Door { _state: PhantomData }
    }
}

impl Door<Unlocked> {
    fn lock(self) -> Door<Locked> {
        println!("Door locked");
        Door { _state: PhantomData }
    }
    
    fn open(&self) {
        println!("Door opened");
    }
}

use std::marker::PhantomData;

fn main() {
    let door = Door::<Locked>::new();
    // door.open();  // Error: locked door can't be opened
    
    let door = door.unlock();
    door.open();  // OK
    
    let door = door.lock();
    // door.open();  // Error again
}
```

---

## Advanced Memory Management

### Memory Layout
```rust
use std::mem;

#[repr(C)]
struct CRepr {
    a: u8,
    b: u32,
}

#[repr(packed)]
struct PackedRepr {
    a: u8,
    b: u32,
}

fn main() {
    println!("Size of u8: {}", mem::size_of::<u8>());
    println!("Size of u32: {}", mem::size_of::<u32>());
    println!("Size of CRepr: {}", mem::size_of::<CRepr>());
    println!("Size of PackedRepr: {}", mem::size_of::<PackedRepr>());
    
    println!("Align of CRepr: {}", mem::align_of::<CRepr>());
    println!("Align of PackedRepr: {}", mem::align_of::<PackedRepr>());
}
```

### Custom Allocators
```rust
use std::alloc::{GlobalAlloc, Layout, System};
use std::sync::atomic::{AtomicUsize, Ordering};

struct CountingAllocator;

static ALLOCATED: AtomicUsize = AtomicUsize::new(0);

unsafe impl GlobalAlloc for CountingAllocator {
    unsafe fn alloc(&self, layout: Layout) -> *mut u8 {
        let ret = System.alloc(layout);
        if !ret.is_null() {
            ALLOCATED.fetch_add(layout.size(), Ordering::SeqCst);
        }
        ret
    }
    
    unsafe fn dealloc(&self, ptr: *mut u8, layout: Layout) {
        System.dealloc(ptr, layout);
        ALLOCATED.fetch_sub(layout.size(), Ordering::SeqCst);
    }
}

#[global_allocator]
static ALLOCATOR: CountingAllocator = CountingAllocator;

fn main() {
    let _v = vec![1, 2, 3, 4, 5];
    println!("Allocated bytes: {}", ALLOCATED.load(Ordering::SeqCst));
}
```

---

## Production-Ready Patterns

### Builder Pattern
```rust
#[derive(Debug)]
struct Request {
    url: String,
    method: String,
    headers: Vec<(String, String)>,
    body: Option<String>,
}

struct RequestBuilder {
    url: String,
    method: String,
    headers: Vec<(String, String)>,
    body: Option<String>,
}

impl RequestBuilder {
    fn new(url: String) -> Self {
        RequestBuilder {
            url,
            method: String::from("GET"),
            headers: Vec::new(),
            body: None,
        }
    }
    
    fn method(mut self, method: &str) -> Self {
        self.method = method.to_string();
        self
    }
    
    fn header(mut self, key: &str, value: &str) -> Self {
        self.headers.push((key.to_string(), value.to_string()));
        self
    }
    
    fn body(mut self, body: String) -> Self {
        self.body = Some(body);
        self
    }
    
    fn build(self) -> Request {
        Request {
            url: self.url,
            method: self.method,
            headers: self.headers,
            body: self.body,
        }
    }
}

fn main() {
    let request = RequestBuilder::new(String::from("https://api.example.com"))
        .method("POST")
        .header("Content-Type", "application/json")
        .body(String::from(r#"{"key": "value"}"#))
        .build();
    
    println!("{:#?}", request);
}
```

### State Machine Pattern
```rust
trait State {}

struct Draft;
struct PendingReview;
struct Published;

impl State for Draft {}
impl State for PendingReview {}
impl State for Published {}

struct Post<S: State> {
    content: String,
    _state: PhantomData<S>,
}

impl Post<Draft> {
    fn new() -> Self {
        Post {
            content: String::new(),
            _state: PhantomData,
        }
    }
    
    fn add_text(&mut self, text: &str) {
        self.content.push_str(text);
    }
    
    fn request_review(self) -> Post<PendingReview> {
        Post {
            content: self.content,
            _state: PhantomData,
        }
    }
}

impl Post<PendingReview> {
    fn approve(self) -> Post<Published> {
        Post {
            content: self.content,
            _state: PhantomData,
        }
    }
}

impl Post<Published> {
    fn content(&self) -> &str {
        &self.content
    }
}

use std::marker::PhantomData;

fn main() {
    let mut post = Post::<Draft>::new();
    post.add_text("I ate a salad for lunch today");
    
    let post = post.request_review();
    let post = post.approve();
    
    println!("Published: {}", post.content());
}
```

### Dependency Injection
```rust
trait Database {
    fn get(&self, key: &str) -> Option<String>;
    fn set(&mut self, key: &str, value: String);
}

struct InMemoryDatabase {
    data: std::collections::HashMap<String, String>,
}

impl Database for InMemoryDatabase {
    fn get(&self, key: &str) -> Option<String> {
        self.data.get(key).cloned()
    }
    
    fn set(&mut self, key: &str, value: String) {
        self.data.insert(key.to_string(), value);
    }
}

struct UserService<D: Database> {
    db: D,
}

impl<D: Database> UserService<D> {
    fn new(db: D) -> Self {
        UserService { db }
    }
    
    fn get_user(&self, id: &str) -> Option<String> {
        self.db.get(id)
    }
}

fn main() {
    let db = InMemoryDatabase {
        data: std::collections::HashMap::new(),
    };
    
    let service = UserService::new(db);
    let user = service.get_user("123");
    
    println!("User: {:?}", user);
}
```

---

## Summary

In this advanced tutorial, you learned:
- Advanced type system features (newtypes, type aliases, DSTs)
- Writing and using macros
- Unsafe Rust and FFI
- Advanced traits and patterns
- Async programming basics
- Performance optimization techniques
- Advanced error handling
- Type-level programming
- Production-ready design patterns

### Congratulations!
You've completed the comprehensive Rust mastery tutorial series. You now have:
- **Basic**: Foundation in Rust syntax and concepts
- **Intermediate**: Proficiency in common Rust patterns
- **Advanced**: Expertise in advanced Rust features

### Next Steps
- Build real-world projects
- Contribute to open-source Rust projects
- Explore specific domains (web, systems, embedded, etc.)
- Read "The Rustonomicon" for even deeper unsafe Rust knowledge
- Study production Rust codebases

---

**You're now a Rust Master! 🦀✨**
