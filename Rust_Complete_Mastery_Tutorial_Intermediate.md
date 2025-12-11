# Rust Complete Mastery Tutorial - Intermediate

## Table of Contents
1. [Collections](#collections)
2. [Generic Types](#generic-types)
3. [Traits](#traits)
4. [Lifetimes](#lifetimes)
5. [Iterators](#iterators)
6. [Closures](#closures)
7. [Smart Pointers](#smart-pointers)
8. [Concurrency](#concurrency)
9. [Fearless Concurrency Patterns](#fearless-concurrency-patterns)
10. [Functional Programming Features](#functional-programming-features)
11. [Testing](#testing)
12. [Input/Output](#inputoutput)
13. [Building Command Line Programs](#building-command-line-programs)
14. [Working with Environment Variables](#working-with-environment-variables)
15. [Writing to Standard Error](#writing-to-standard-error)

---

## Collections

### Vectors - Vec<T>

#### Creating Vectors
```rust
fn main() {
    // Create empty vector with type annotation
    let v: Vec<i32> = Vec::new();
    
    // Create vector with initial values using vec! macro
    let v = vec![1, 2, 3, 4, 5];
    
    // Create with capacity
    let mut v: Vec<i32> = Vec::with_capacity(10);
    
    println!("Vector: {:?}", v);
}
```

#### Updating Vectors
```rust
fn main() {
    let mut v = Vec::new();
    
    // Add elements
    v.push(5);
    v.push(6);
    v.push(7);
    v.push(8);
    
    println!("Vector: {:?}", v);
    
    // Remove last element
    if let Some(last) = v.pop() {
        println!("Popped: {}", last);
    }
    
    // Insert at index
    v.insert(1, 10);
    
    // Remove at index
    v.remove(2);
    
    println!("Final vector: {:?}", v);
}
```

#### Reading Elements
```rust
fn main() {
    let v = vec![1, 2, 3, 4, 5];
    
    // Using indexing - panics if out of bounds
    let third = v[2];
    println!("Third element: {}", third);
    
    // Using get method - returns Option<&T>
    match v.get(2) {
        Some(third) => println!("Third element: {}", third),
        None => println!("No third element"),
    }
    
    // Safe access for potentially invalid index
    let index = 10;
    match v.get(index) {
        Some(value) => println!("Value at {}: {}", index, value),
        None => println!("Index {} out of bounds", index),
    }
}
```

#### Iterating Over Vectors
```rust
fn main() {
    let v = vec![100, 32, 57];
    
    // Immutable iteration
    for i in &v {
        println!("{}", i);
    }
    
    // Mutable iteration
    let mut v = vec![100, 32, 57];
    for i in &mut v {
        *i += 50;  // Dereference to modify
    }
    
    println!("Modified vector: {:?}", v);
    
    // Consuming iteration
    let v = vec![1, 2, 3];
    for i in v {
        println!("{}", i);
    }
    // v is no longer valid here
}
```

#### Storing Multiple Types with Enums
```rust
enum SpreadsheetCell {
    Int(i32),
    Float(f64),
    Text(String),
}

fn main() {
    let row = vec![
        SpreadsheetCell::Int(3),
        SpreadsheetCell::Text(String::from("blue")),
        SpreadsheetCell::Float(10.12),
    ];
    
    for cell in &row {
        match cell {
            SpreadsheetCell::Int(i) => println!("Int: {}", i),
            SpreadsheetCell::Float(f) => println!("Float: {}", f),
            SpreadsheetCell::Text(s) => println!("Text: {}", s),
        }
    }
}
```

### Strings

#### Creating Strings
```rust
fn main() {
    // Create new empty String
    let mut s = String::new();
    
    // Create from string literal
    let s = "initial contents".to_string();
    let s = String::from("initial contents");
    
    // Strings are UTF-8 encoded
    let hello = String::from("السلام عليكم");
    let hello = String::from("Dobrý den");
    let hello = String::from("Hello");
    let hello = String::from("שָׁלוֹם");
    let hello = String::from("नमस्ते");
    let hello = String::from("こんにちは");
    let hello = String::from("안녕하세요");
    let hello = String::from("你好");
    let hello = String::from("Olá");
    let hello = String::from("Здравствуйте");
    let hello = String::from("Hola");
    
    println!("Greetings: {}", hello);
}
```

#### Updating Strings
```rust
fn main() {
    // push_str adds a string slice
    let mut s = String::from("foo");
    s.push_str("bar");
    println!("{}", s);  // foobar
    
    // push adds a single character
    let mut s = String::from("lo");
    s.push('l');
    println!("{}", s);  // lol
    
    // Concatenation with +
    let s1 = String::from("Hello, ");
    let s2 = String::from("world!");
    let s3 = s1 + &s2;  // s1 is moved here
    println!("{}", s3);
    
    // format! macro doesn't take ownership
    let s1 = String::from("tic");
    let s2 = String::from("tac");
    let s3 = String::from("toe");
    let s = format!("{}-{}-{}", s1, s2, s3);
    println!("{}", s);
}
```

#### String Slicing
```rust
fn main() {
    let hello = "Здравствуйте";
    
    // Slice by byte index
    let s = &hello[0..4];  // Зд (each Cyrillic letter is 2 bytes)
    println!("{}", s);
    
    // This would panic at runtime:
    // let s = &hello[0..1];  // Invalid UTF-8 boundary
}
```

#### Iterating Over Strings
```rust
fn main() {
    let s = String::from("नमस्ते");
    
    // Iterate over characters
    println!("Characters:");
    for c in s.chars() {
        println!("{}", c);
    }
    
    // Iterate over bytes
    println!("Bytes:");
    for b in s.bytes() {
        println!("{}", b);
    }
}
```

#### String Methods
```rust
fn main() {
    let s = String::from("  Hello, World!  ");
    
    // Length
    println!("Length: {}", s.len());
    
    // Check if empty
    println!("Is empty: {}", s.is_empty());
    
    // Contains
    println!("Contains 'World': {}", s.contains("World"));
    
    // Starts with / Ends with
    println!("Starts with '  Hello': {}", s.starts_with("  Hello"));
    println!("Ends with '!  ': {}", s.ends_with("!  "));
    
    // Trim whitespace
    let trimmed = s.trim();
    println!("Trimmed: '{}'", trimmed);
    
    // Replace
    let replaced = s.replace("World", "Rust");
    println!("Replaced: '{}'", replaced);
    
    // Split
    let s = String::from("apple,banana,cherry");
    for part in s.split(',') {
        println!("Part: {}", part);
    }
    
    // To uppercase / lowercase
    let s = String::from("Hello");
    println!("Uppercase: {}", s.to_uppercase());
    println!("Lowercase: {}", s.to_lowercase());
}
```

### Hash Maps - HashMap<K, V>

#### Creating Hash Maps
```rust
use std::collections::HashMap;

fn main() {
    // Create new HashMap
    let mut scores = HashMap::new();
    
    // Insert key-value pairs
    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);
    
    println!("Scores: {:?}", scores);
    
    // Create from vectors
    let teams = vec![String::from("Blue"), String::from("Yellow")];
    let initial_scores = vec![10, 50];
    
    let scores: HashMap<_, _> = teams.into_iter()
        .zip(initial_scores.into_iter())
        .collect();
    
    println!("Scores from vectors: {:?}", scores);
}
```

#### Ownership and Hash Maps
```rust
use std::collections::HashMap;

fn main() {
    let field_name = String::from("Favorite color");
    let field_value = String::from("Blue");
    
    let mut map = HashMap::new();
    map.insert(field_name, field_value);
    // field_name and field_value are no longer valid
    
    // For types that implement Copy trait
    let x = 5;
    let y = 10;
    let mut numbers = HashMap::new();
    numbers.insert(x, y);
    // x and y are still valid
    println!("x: {}, y: {}", x, y);
}
```

#### Accessing Values
```rust
use std::collections::HashMap;

fn main() {
    let mut scores = HashMap::new();
    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);
    
    // Get value with get method
    let team_name = String::from("Blue");
    let score = scores.get(&team_name);
    
    match score {
        Some(s) => println!("Score: {}", s),
        None => println!("Team not found"),
    }
    
    // Iterate over key-value pairs
    for (key, value) in &scores {
        println!("{}: {}", key, value);
    }
}
```

#### Updating Hash Maps
```rust
use std::collections::HashMap;

fn main() {
    let mut scores = HashMap::new();
    
    // Overwriting a value
    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Blue"), 25);
    println!("Overwritten: {:?}", scores);
    
    // Only insert if key doesn't exist
    scores.entry(String::from("Yellow")).or_insert(50);
    scores.entry(String::from("Blue")).or_insert(50);  // Won't overwrite
    println!("With entry: {:?}", scores);
    
    // Update based on old value
    let text = "hello world wonderful world";
    let mut map = HashMap::new();
    
    for word in text.split_whitespace() {
        let count = map.entry(word).or_insert(0);
        *count += 1;
    }
    
    println!("Word count: {:?}", map);
}
```

---

## Generic Types

### Generic Functions
```rust
// Find largest value in a slice
fn largest<T: PartialOrd + Copy>(list: &[T]) -> T {
    let mut largest = list[0];
    
    for &item in list {
        if item > largest {
            largest = item;
        }
    }
    
    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];
    let result = largest(&number_list);
    println!("Largest number: {}", result);
    
    let char_list = vec!['y', 'm', 'a', 'q'];
    let result = largest(&char_list);
    println!("Largest char: {}", result);
}
```

### Generic Structs
```rust
// Single type parameter
struct Point<T> {
    x: T,
    y: T,
}

// Multiple type parameters
struct Point2<T, U> {
    x: T,
    y: U,
}

fn main() {
    let integer_point = Point { x: 5, y: 10 };
    let float_point = Point { x: 1.0, y: 4.0 };
    
    // Mixed types
    let mixed_point = Point2 { x: 5, y: 4.0 };
    
    println!("Integer point: ({}, {})", integer_point.x, integer_point.y);
    println!("Float point: ({}, {})", float_point.x, float_point.y);
    println!("Mixed point: ({}, {})", mixed_point.x, mixed_point.y);
}
```

### Generic Enums
```rust
// Option enum (from standard library)
enum Option<T> {
    Some(T),
    None,
}

// Result enum (from standard library)
enum Result<T, E> {
    Ok(T),
    Err(E),
}

fn main() {
    let some_number = Some(5);
    let some_string = Some("a string");
    let absent_number: Option<i32> = None;
}
```

### Generic Methods
```rust
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}

// Method for specific type
impl Point<f32> {
    fn distance_from_origin(&self) -> f32 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}

// Multiple generic types
struct Point2<T, U> {
    x: T,
    y: U,
}

impl<T, U> Point2<T, U> {
    fn mixup<V, W>(self, other: Point2<V, W>) -> Point2<T, W> {
        Point2 {
            x: self.x,
            y: other.y,
        }
    }
}

fn main() {
    let p = Point { x: 5, y: 10 };
    println!("p.x = {}", p.x());
    
    let p = Point { x: 3.0, y: 4.0 };
    println!("Distance: {}", p.distance_from_origin());
    
    let p1 = Point2 { x: 5, y: 10.4 };
    let p2 = Point2 { x: "Hello", y: 'c' };
    let p3 = p1.mixup(p2);
    println!("p3.x = {}, p3.y = {}", p3.x, p3.y);
}
```

---

## Traits

### Defining Traits
```rust
// Define a trait
pub trait Summary {
    fn summarize(&self) -> String;
}

// Implement trait for a type
pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}

pub struct Tweet {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub retweet: bool,
}

impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}

fn main() {
    let tweet = Tweet {
        username: String::from("horse_ebooks"),
        content: String::from("of course, as you probably already know, people"),
        reply: false,
        retweet: false,
    };
    
    println!("1 new tweet: {}", tweet.summarize());
}
```

### Default Implementations
```rust
pub trait Summary {
    fn summarize_author(&self) -> String;
    
    // Default implementation
    fn summarize(&self) -> String {
        format!("(Read more from {}...)", self.summarize_author())
    }
}

pub struct Tweet {
    pub username: String,
    pub content: String,
}

impl Summary for Tweet {
    fn summarize_author(&self) -> String {
        format!("@{}", self.username)
    }
    
    // Can override default implementation
    // fn summarize(&self) -> String {
    //     format!("{}: {}", self.username, self.content)
    // }
}

fn main() {
    let tweet = Tweet {
        username: String::from("horse_ebooks"),
        content: String::from("of course, as you probably already know, people"),
    };
    
    println!("New tweet: {}", tweet.summarize());
}
```

### Traits as Parameters
```rust
pub trait Summary {
    fn summarize(&self) -> String;
}

// Trait bound syntax
pub fn notify(item: &impl Summary) {
    println!("Breaking news! {}", item.summarize());
}

// Full trait bound syntax
pub fn notify_full<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}

// Multiple trait bounds
use std::fmt::Display;

pub fn notify_multiple(item: &(impl Summary + Display)) {
    println!("{}", item);
}

// With generics
pub fn notify_generic<T: Summary + Display>(item: &T) {
    println!("{}", item);
}

// Where clause for readability
fn some_function<T, U>(t: &T, u: &U) -> i32
    where T: Display + Clone,
          U: Clone + Summary
{
    0
}
```

### Returning Traits
```rust
pub trait Summary {
    fn summarize(&self) -> String;
}

pub struct Tweet {
    pub username: String,
    pub content: String,
}

impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}

// Return type that implements a trait
fn returns_summarizable() -> impl Summary {
    Tweet {
        username: String::from("horse_ebooks"),
        content: String::from("of course, as you probably already know, people"),
    }
}

fn main() {
    let tweet = returns_summarizable();
    println!("{}", tweet.summarize());
}
```

### Conditional Method Implementation
```rust
use std::fmt::Display;

struct Pair<T> {
    x: T,
    y: T,
}

impl<T> Pair<T> {
    fn new(x: T, y: T) -> Self {
        Self { x, y }
    }
}

// Only implement for types that implement Display and PartialOrd
impl<T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("The largest member is x = {}", self.x);
        } else {
            println!("The largest member is y = {}", self.y);
        }
    }
}

fn main() {
    let pair = Pair::new(5, 10);
    pair.cmp_display();
}
```

### Common Traits

#### Display and Debug
```rust
use std::fmt;

struct Point {
    x: i32,
    y: i32,
}

// Implement Display trait
impl fmt::Display for Point {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "({}, {})", self.x, self.y)
    }
}

// Derive Debug trait
#[derive(Debug)]
struct Point2 {
    x: i32,
    y: i32,
}

fn main() {
    let p = Point { x: 3, y: 4 };
    println!("Point: {}", p);  // Uses Display
    
    let p2 = Point2 { x: 3, y: 4 };
    println!("Point2: {:?}", p2);  // Uses Debug
    println!("Point2: {:#?}", p2);  // Pretty Debug
}
```

#### Clone and Copy
```rust
// Clone: explicit deep copy
#[derive(Clone)]
struct Person {
    name: String,
    age: u32,
}

// Copy: implicit copy (only for stack-only data)
#[derive(Copy, Clone)]
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let person1 = Person {
        name: String::from("Alice"),
        age: 30,
    };
    
    let person2 = person1.clone();  // Explicit clone
    println!("Person: {}, {}", person2.name, person2.age);
    
    let point1 = Point { x: 1, y: 2 };
    let point2 = point1;  // Implicit copy
    println!("Point1: ({}, {})", point1.x, point1.y);  // Still valid
    println!("Point2: ({}, {})", point2.x, point2.y);
}
```

---

## Lifetimes

### Generic Lifetimes in Functions
```rust
// Without lifetimes - won't compile
// fn longest(x: &str, y: &str) -> &str {
//     if x.len() > y.len() {
//         x
//     } else {
//         y
//     }
// }

// With lifetime annotations
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";
    
    let result = longest(string1.as_str(), string2);
    println!("Longest string: {}", result);
    
    // Lifetime scope
    let string1 = String::from("long string is long");
    {
        let string2 = String::from("xyz");
        let result = longest(string1.as_str(), string2.as_str());
        println!("Longest string: {}", result);
    }
}
```

### Lifetime Annotations in Structs
```rust
// Struct holding a reference needs lifetime annotation
struct ImportantExcerpt<'a> {
    part: &'a str,
}

impl<'a> ImportantExcerpt<'a> {
    fn level(&self) -> i32 {
        3
    }
    
    // Lifetime elision rules apply
    fn announce_and_return_part(&self, announcement: &str) -> &str {
        println!("Attention please: {}", announcement);
        self.part
    }
}

fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.').next().expect("Could not find a '.'");
    
    let excerpt = ImportantExcerpt {
        part: first_sentence,
    };
    
    println!("Excerpt: {}", excerpt.part);
    println!("Level: {}", excerpt.level());
}
```

### Lifetime Elision Rules
```rust
// These functions don't need explicit lifetime annotations
// due to lifetime elision rules

// Rule 1: Each parameter gets its own lifetime
fn first_word(s: &str) -> &str {
    let bytes = s.as_bytes();
    
    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }
    
    &s[..]
}

// Rule 2: If there's exactly one input lifetime, 
// it's assigned to all output lifetimes
fn return_string(s: &str) -> &str {
    s
}

// Multiple parameters need explicit annotations
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

fn main() {
    let s = String::from("hello world");
    let word = first_word(&s);
    println!("First word: {}", word);
}
```

### The Static Lifetime
```rust
fn main() {
    // 'static lifetime lives for entire program duration
    let s: &'static str = "I have a static lifetime.";
    
    // String literals have static lifetime
    let s = "hello world";
    
    println!("{}", s);
}
```

### Generic Type Parameters, Trait Bounds, and Lifetimes Together
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
    let string1 = String::from("abcd");
    let string2 = "xyz";
    
    let result = longest_with_announcement(
        string1.as_str(),
        string2,
        "Today is someone's birthday!",
    );
    
    println!("Longest string: {}", result);
}
```

---

## Iterators

### Creating Iterators
```rust
fn main() {
    let v1 = vec![1, 2, 3];
    
    // Create iterator
    let v1_iter = v1.iter();
    
    // Use iterator in for loop
    for val in v1_iter {
        println!("Got: {}", val);
    }
    
    // Different iterator methods
    let v = vec![1, 2, 3];
    
    let iter = v.iter();        // Immutable references
    let iter_mut = v.iter_mut(); // Mutable references (needs mut v)
    let into_iter = v.into_iter(); // Takes ownership
}
```

### The Iterator Trait
```rust
// Iterator trait definition (simplified)
pub trait Iterator {
    type Item;
    
    fn next(&mut self) -> Option<Self::Item>;
    
    // Many more methods with default implementations
}

fn main() {
    let v1 = vec![1, 2, 3];
    let mut v1_iter = v1.iter();
    
    // next() returns Option<&T>
    assert_eq!(v1_iter.next(), Some(&1));
    assert_eq!(v1_iter.next(), Some(&2));
    assert_eq!(v1_iter.next(), Some(&3));
    assert_eq!(v1_iter.next(), None);
}
```

### Methods That Consume the Iterator
```rust
fn main() {
    let v1 = vec![1, 2, 3];
    let v1_iter = v1.iter();
    
    // sum() consumes the iterator
    let total: i32 = v1_iter.sum();
    println!("Sum: {}", total);
    
    // v1_iter can't be used anymore
    
    // Other consuming adaptors
    let v2 = vec![1, 2, 3, 4, 5];
    
    let max = v2.iter().max();
    println!("Max: {:?}", max);
    
    let min = v2.iter().min();
    println!("Min: {:?}", min);
    
    let count = v2.iter().count();
    println!("Count: {}", count);
}
```

### Methods That Produce Other Iterators
```rust
fn main() {
    let v1: Vec<i32> = vec![1, 2, 3];
    
    // map() produces new iterator
    let v2: Vec<_> = v1.iter().map(|x| x + 1).collect();
    println!("v2: {:?}", v2);
    
    // filter() produces new iterator
    let v3: Vec<_> = v1.iter().filter(|x| *x % 2 == 0).collect();
    println!("Even numbers: {:?}", v3);
    
    // Chain multiple iterator adaptors
    let v4: Vec<_> = v1.iter()
        .map(|x| x * 2)
        .filter(|x| *x > 3)
        .collect();
    println!("v4: {:?}", v4);
}
```

### Using Closures That Capture Environment
```rust
#[derive(PartialEq, Debug)]
struct Shoe {
    size: u32,
    style: String,
}

fn shoes_in_size(shoes: Vec<Shoe>, shoe_size: u32) -> Vec<Shoe> {
    shoes.into_iter().filter(|s| s.size == shoe_size).collect()
}

fn main() {
    let shoes = vec![
        Shoe {
            size: 10,
            style: String::from("sneaker"),
        },
        Shoe {
            size: 13,
            style: String::from("sandal"),
        },
        Shoe {
            size: 10,
            style: String::from("boot"),
        },
    ];
    
    let in_my_size = shoes_in_size(shoes, 10);
    
    println!("Shoes in size 10: {:#?}", in_my_size);
}
```

### Common Iterator Methods
```rust
fn main() {
    let numbers = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
    
    // map - transform each element
    let doubled: Vec<_> = numbers.iter().map(|x| x * 2).collect();
    println!("Doubled: {:?}", doubled);
    
    // filter - keep elements matching predicate
    let evens: Vec<_> = numbers.iter().filter(|x| *x % 2 == 0).collect();
    println!("Evens: {:?}", evens);
    
    // fold - accumulate values
    let sum = numbers.iter().fold(0, |acc, x| acc + x);
    println!("Sum: {}", sum);
    
    // any - check if any element matches
    let has_even = numbers.iter().any(|x| x % 2 == 0);
    println!("Has even: {}", has_even);
    
    // all - check if all elements match
    let all_positive = numbers.iter().all(|x| *x > 0);
    println!("All positive: {}", all_positive);
    
    // find - get first matching element
    let first_even = numbers.iter().find(|x| *x % 2 == 0);
    println!("First even: {:?}", first_even);
    
    // take - take first n elements
    let first_three: Vec<_> = numbers.iter().take(3).collect();
    println!("First three: {:?}", first_three);
    
    // skip - skip first n elements
    let skip_three: Vec<_> = numbers.iter().skip(3).collect();
    println!("Skip three: {:?}", skip_three);
    
    // enumerate - get index and value
    for (i, val) in numbers.iter().enumerate() {
        println!("Index {}: {}", i, val);
    }
    
    // zip - combine two iterators
    let letters = vec!['a', 'b', 'c'];
    let zipped: Vec<_> = numbers.iter().zip(letters.iter()).collect();
    println!("Zipped: {:?}", zipped);
}
```

### Creating Custom Iterators
```rust
struct Counter {
    count: u32,
}

impl Counter {
    fn new() -> Counter {
        Counter { count: 0 }
    }
}

impl Iterator for Counter {
    type Item = u32;
    
    fn next(&mut self) -> Option<Self::Item> {
        if self.count < 5 {
            self.count += 1;
            Some(self.count)
        } else {
            None
        }
    }
}

fn main() {
    let mut counter = Counter::new();
    
    println!("Manual iteration:");
    while let Some(n) = counter.next() {
        println!("{}", n);
    }
    
    // Use other iterator methods
    let sum: u32 = Counter::new()
        .zip(Counter::new().skip(1))
        .map(|(a, b)| a * b)
        .filter(|x| x % 3 == 0)
        .sum();
    
    println!("Sum: {}", sum);
}
```

---

## Closures

### Basic Closure Syntax
```rust
fn main() {
    // Basic closure
    let add_one = |x| x + 1;
    println!("5 + 1 = {}", add_one(5));
    
    // Closure with type annotations
    let add_one = |x: i32| -> i32 { x + 1 };
    println!("5 + 1 = {}", add_one(5));
    
    // Multiline closure
    let multiply = |x, y| {
        let result = x * y;
        result
    };
    println!("3 * 4 = {}", multiply(3, 4));
    
    // Closure vs function
    fn  add_one_fn(x: i32) -> i32 { x + 1 }
    let add_one_cl = |x: i32| -> i32 { x + 1 };
    
    println!("Function: {}", add_one_fn(5));
    println!("Closure: {}", add_one_cl(5));
}
```

### Capturing the Environment
```rust
fn main() {
    let x = 4;
    
    // Closure captures x from environment
    let equal_to_x = |z| z == x;
    
    let y = 4;
    assert!(equal_to_x(y));
    
    // Functions can't capture environment
    // fn equal_to_x(z: i32) -> bool { z == x }  // Error
}
```

### Closure Type Inference
```rust
fn main() {
    let example_closure = |x| x;
    
    let s = example_closure(String::from("hello"));
    // Type is locked to String now
    
    // let n = example_closure(5);  // Error: expected String, found i32
}
```

### Fn Traits
```rust
fn main() {
    // FnOnce - consumes captured values (takes ownership)
    let x = vec![1, 2, 3];
    let consume = || {
        println!("Consuming: {:?}", x);
        drop(x);  // Takes ownership
    };
    consume();
    // consume();  // Error: can't call twice
    
    // FnMut - mutably borrows values
    let mut count = 0;
    let mut increment = || {
        count += 1;
        println!("Count: {}", count);
    };
    increment();
    increment();
    println!("Final count: {}", count);
    
    // Fn - immutably borrows values
    let text = String::from("hello");
    let print = || println!("Text: {}", text);
    print();
    print();
    println!("Original text: {}", text);
}
```

### move Keyword
```rust
use std::thread;

fn main() {
    let x = vec![1, 2, 3];
    
    // Force closure to take ownership
    let closure = move || println!("x: {:?}", x);
    
    // x is no longer valid here
    // println!("{:?}", x);  // Error
    
    closure();
    
    // Useful with threads
    let list = vec![1, 2, 3];
    thread::spawn(move || println!("List: {:?}", list))
        .join()
        .unwrap();
    
    // list is no longer valid
}
```

### Storing Closures
```rust
struct Cacher<T>
where
    T: Fn(u32) -> u32,
{
    calculation: T,
    value: Option<u32>,
}

impl<T> Cacher<T>
where
    T: Fn(u32) -> u32,
{
    fn new(calculation: T) -> Cacher<T> {
        Cacher {
            calculation,
            value: None,
        }
    }
    
    fn value(&mut self, arg: u32) -> u32 {
        match self.value {
            Some(v) => v,
            None => {
                let v = (self.calculation)(arg);
                self.value = Some(v);
                v
            }
        }
    }
}

fn main() {
    let mut expensive_closure = Cacher::new(|num| {
        println!("Calculating slowly...");
        std::thread::sleep(std::time::Duration::from_secs(2));
        num
    });
    
    println!("Result: {}", expensive_closure.value(10));
    println!("Cached result: {}", expensive_closure.value(10));  // Instant
}
```

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
    
    // Function pointers implement all three Fn traits
    let list_of_numbers = vec![1, 2, 3];
    let list_of_strings: Vec<String> =
        list_of_numbers.iter().map(|i| i.to_string()).collect();
    
    // Can also use function name directly
    let list_of_strings: Vec<String> =
        list_of_numbers.iter().map(ToString::to_string).collect();
    
    println!("Strings: {:?}", list_of_strings);
}
```

---

## Smart Pointers

### Box<T> - Heap Allocation
```rust
fn main() {
    // Store value on heap
    let b = Box::new(5);
    println!("b = {}", b);
    
    // Recursive types need Box
    #[derive(Debug)]
    enum List {
        Cons(i32, Box<List>),
        Nil,
    }
    
    use List::{Cons, Nil};
    
    let list = Cons(1, Box::new(Cons(2, Box::new(Cons(3, Box::new(Nil))))));
    println!("List: {:?}", list);
}
```

### Deref Trait
```rust
use std::ops::Deref;

struct MyBox<T>(T);

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox(x)
    }
}

impl<T> Deref for MyBox<T> {
    type Target = T;
    
    fn deref(&self) -> &Self::Target {
        &self.0
    }
}

fn hello(name: &str) {
    println!("Hello, {}!", name);
}

fn main() {
    let x = 5;
    let y = MyBox::new(x);
    
    assert_eq!(5, x);
    assert_eq!(5, *y);  // Deref coercion
    
    let m = MyBox::new(String::from("Rust"));
    hello(&m);  // &MyBox<String> -> &String -> &str (deref coercion)
}
```

### Drop Trait
```rust
struct CustomSmartPointer {
    data: String,
}

impl Drop for CustomSmartPointer {
    fn drop(&mut self) {
        println!("Dropping CustomSmartPointer with data `{}`!", self.data);
    }
}

fn main() {
    let c = CustomSmartPointer {
        data: String::from("my stuff"),
    };
    let d = CustomSmartPointer {
        data: String::from("other stuff"),
    };
    println!("CustomSmartPointers created.");
    
    // Early drop
    drop(c);
    println!("CustomSmartPointer dropped early.");
}  // d is dropped here
```

### Rc<T> - Reference Counting
```rust
use std::rc::Rc;

#[derive(Debug)]
enum List {
    Cons(i32, Rc<List>),
    Nil,
}

use List::{Cons, Nil};

fn main() {
    let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil)))));
    println!("Count after creating a: {}", Rc::strong_count(&a));
    
    let b = Cons(3, Rc::clone(&a));
    println!("Count after creating b: {}", Rc::strong_count(&a));
    
    {
        let c = Cons(4, Rc::clone(&a));
        println!("Count after creating c: {}", Rc::strong_count(&a));
    }
    
    println!("Count after c goes out of scope: {}", Rc::strong_count(&a));
}
```

### RefCell<T> - Interior Mutability
```rust
use std::cell::RefCell;

fn main() {
    let x = RefCell::new(5);
    
    // Borrow immutably
    let a = x.borrow();
    println!("a: {}", a);
    drop(a);  // Must drop before borrowing mutably
    
    // Borrow mutably
    let mut b = x.borrow_mut();
    *b += 1;
    drop(b);
    
    println!("x: {:?}", x);
    
    // Multiple immutable borrows OK
    let c = x.borrow();
    let d = x.borrow();
    println!("c: {}, d: {}", c, d);
}
```

### Combining Rc<T> and RefCell<T>
```rust
use std::rc::Rc;
use std::cell::RefCell;

#[derive(Debug)]
enum List {
    Cons(Rc<RefCell<i32>>, Rc<List>),
    Nil,
}

use List::{Cons, Nil};

fn main() {
    let value = Rc::new(RefCell::new(5));
    
    let a = Rc::new(Cons(Rc::clone(&value), Rc::new(Nil)));
    let b = Cons(Rc::new(RefCell::new(3)), Rc::clone(&a));
    let c = Cons(Rc::new(RefCell::new(4)), Rc::clone(&a));
    
    *value.borrow_mut() += 10;
    
    println!("a: {:?}", a);
    println!("b: {:?}", b);
    println!("c: {:?}", c);
}
```

---

## Concurrency

### Creating Threads
```rust
use std::thread;
use std::time::Duration;

fn main() {
    // Spawn a new thread
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("Number {} from spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });
    
    for i in 1..5 {
        println!("Number {} from main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }
    
    // Wait for thread to finish
    handle.join().unwrap();
}
```

### Moving Values into Threads
```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];
    
    let handle = thread::spawn(move || {
        println!("Vector: {:?}", v);
    });
    
    // v is no longer valid here
    // println!("{:?}", v);  // Error
    
    handle.join().unwrap();
}
```

### Message Passing with Channels
```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    // Create channel
    let (tx, rx) = mpsc::channel();
    
    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
        // val is no longer valid
    });
    
    // Receive value
    let received = rx.recv().unwrap();
    println!("Received: {}", received);
}
```

### Sending Multiple Values
```rust
use std::sync::mpsc;
use std::thread;
use std::time::Duration;

fn main() {
    let (tx, rx) = mpsc::channel();
    
    thread::spawn(move || {
        let vals = vec![
            String::from("hi"),
            String::from("from"),
            String::from("the"),
            String::from("thread"),
        ];
        
        for val in vals {
            tx.send(val).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });
    
    // Treat rx as iterator
    for received in rx {
        println!("Received: {}", received);
    }
}
```

### Multiple Producers
```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();
    
    // Clone transmitter
    let tx1 = tx.clone();
    thread::spawn(move || {
        let vals = vec![
            String::from("hi"),
            String::from("from"),
            String::from("the"),
            String::from("thread"),
        ];
        
        for val in vals {
            tx1.send(val).unwrap();
        }
    });
    
    thread::spawn(move || {
        let vals = vec![
            String::from("more"),
            String::from("messages"),
            String::from("for"),
            String::from("you"),
        ];
        
        for val in vals {
            tx.send(val).unwrap();
        }
    });
    
    for received in rx {
        println!("Received: {}", received);
    }
}
```

### Shared State with Mutex
```rust
use std::sync::Mutex;

fn main() {
    let m = Mutex::new(5);
    
    {
        let mut num = m.lock().unwrap();
        *num = 6;
    }  // Lock is released here
    
    println!("m = {:?}", m);
}
```

### Sharing Mutex Between Threads
```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    // Arc: Atomic Reference Counting
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];
    
    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();
            *num += 1;
        });
        handles.push(handle);
    }
    
    for handle in handles {
        handle.join().unwrap();
    }
    
    println!("Result: {}", *counter.lock().unwrap());
}
```

---

## Fearless Concurrency Patterns

### Thread Pool Pattern
```rust
use std::sync::{mpsc, Arc, Mutex};
use std::thread;

type Job = Box<dyn FnOnce() + Send + 'static>;

struct ThreadPool {
    workers: Vec<Worker>,
    sender: mpsc::Sender<Job>,
}

impl ThreadPool {
    fn new(size: usize) -> ThreadPool {
        let (sender, receiver) = mpsc::channel();
        let receiver = Arc::new(Mutex::new(receiver));
        
        let mut workers = Vec::with_capacity(size);
        
        for id in 0..size {
            workers.push(Worker::new(id, Arc::clone(&receiver)));
        }
        
        ThreadPool { workers, sender }
    }
    
    fn execute<F>(&self, f: F)
    where
        F: FnOnce() + Send + 'static,
    {
        let job = Box::new(f);
        self.sender.send(job).unwrap();
    }
}

struct Worker {
    id: usize,
    thread: thread::JoinHandle<()>,
}

impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        let thread = thread::spawn(move || loop {
            let job = receiver.lock().unwrap().recv().unwrap();
            println!("Worker {} executing job", id);
            job();
        });
        
        Worker { id, thread }
    }
}

fn main() {
    let pool = ThreadPool::new(4);
    
    for i in 0..8 {
        pool.execute(move || {
            println!("Executing task {}", i);
            thread::sleep(std::time::Duration::from_secs(1));
        });
    }
    
    thread::sleep(std::time::Duration::from_secs(10));
}
```

---

## Functional Programming Features

### Combining Iterators and Closures
```rust
#[derive(Debug)]
struct Person {
    name: String,
    age: u32,
}

fn main() {
    let people = vec![
        Person { name: String::from("Alice"), age: 30 },
        Person { name: String::from("Bob"), age: 25 },
        Person { name: String::from("Carol"), age: 35 },
    ];
    
    // Filter and map
    let adults: Vec<&str> = people
        .iter()
        .filter(|p| p.age >= 30)
        .map(|p| p.name.as_str())
        .collect();
    
    println!("Adults: {:?}", adults);
    
    // Calculate average age
    let average_age: f64 = people.iter()
        .map(|p| p.age as f64)
        .sum::<f64>() / people.len() as f64;
    
    println!("Average age: {:.1}", average_age);
}
```

### Improving Performance
```rust
fn main() {
    let numbers: Vec<_> = (1..=1000).collect();
    
    // Iterator version (faster)
    let sum: i32 = numbers.iter().sum();
    
    // Loop version
    let mut total = 0;
    for num in &numbers {
        total += num;
    }
    
    assert_eq!(sum, total);
}
```

---

## Testing

### Writing Tests
```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
    
    #[test]
    fn another_test() {
        assert!(true);
    }
}

fn main() {
    println!("Run with: cargo test");
}
```

### Assert Macros
```rust
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}

#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_add() {
        assert_eq!(add(2, 2), 4);
        assert_ne!(add(2, 2), 5);
    }
    
    #[test]
    fn test_with_message() {
        let result = add(2, 2);
        assert!(result == 4, "Expected 4, got {}", result);
    }
}
```

### Testing for Panics
```rust
pub fn divide(a: i32, b: i32) -> i32 {
    if b == 0 {
        panic!("Division by zero!");
    }
    a / b
}

#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    #[should_panic(expected = "Division by zero")]
    fn test_divide_by_zero() {
        divide(10, 0);
    }
}
```

### Testing with Result
```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() -> Result<(), String> {
        if 2 + 2 == 4 {
            Ok(())
        } else {
            Err(String::from("two plus two does not equal four"))
        }
    }
}
```

### Controlling Test Execution
```bash
# Run tests
cargo test

# Run tests with output
cargo test -- --show-output

# Run specific test
cargo test test_name

# Run tests matching pattern
cargo test add

# Run ignored tests
cargo test -- --ignored

# Run all tests
cargo test -- --include-ignored
```

---

## Input/Output

### Reading from Files
```rust
use std::fs;
use std::io::{self, Read};

fn main() -> io::Result<()> {
    // Read entire file to string
    let contents = fs::read_to_string("file.txt")?;
    println!("Contents:\n{}", contents);
    
    // Read to bytes
    let bytes = fs::read("file.txt")?;
    println!("Bytes read: {}", bytes.len());
    
    Ok(())
}
```

### Writing to Files
```rust
use std::fs;
use std::io::Write;

fn main() -> std::io::Result<()> {
    // Write string to file (overwrites)
    fs::write("output.txt", "Hello, file!")?;
    
    // Append to file
    let mut file = fs::OpenOptions::new()
        .append(true)
        .open("output.txt")?;
    
    file.write_all(b"\nAppended line")?;
    
    Ok(())
}
```

### Reading User Input
```rust
use std::io;

fn main() {
    println!("Enter your name:");
    
    let mut input = String::new();
    io::stdin()
        .read_line(&mut input)
        .expect("Failed to read line");
    
    let input = input.trim();
    println!("Hello, {}!", input);
}
```

---

## Building Command Line Programs

### Parsing Command Line Arguments
```rust
use std::env;

fn main() {
    let args: Vec<String> = env::args().collect();
    
    if args.len() < 3 {
        eprintln!("Usage: {} <query> <filename>", args[0]);
        std::process::exit(1);
    }
    
    let query = &args[1];
    let filename = &args[2];
    
    println!("Searching for '{}' in file '{}'", query, filename);
}
```

### Better Argument Parsing with Struct
```rust
use std::env;
use std::process;

struct Config {
    query: String,
    filename: String,
}

impl Config {
    fn new(args: &[String]) -> Result<Config, &'static str> {
        if args.len() < 3 {
            return Err("Not enough arguments");
        }
        
        let query = args[1].clone();
        let filename = args[2].clone();
        
        Ok(Config { query, filename })
    }
}

fn main() {
    let args: Vec<String> = env::args().collect();
    
    let config = Config::new(&args).unwrap_or_else(|err| {
        eprintln!("Problem parsing arguments: {}", err);
        process::exit(1);
    });
    
    println!("Searching for '{}' in '{}'", config.query, config.filename);
}
```

---

## Working with Environment Variables

### Reading Environment Variables
```rust
use std::env;

fn main() {
    // Get specific variable
    match env::var("USER") {
        Ok(val) => println!("USER: {}", val),
        Err(e) => println!("Couldn't read USER: {}", e),
    }
    
    // Get with default
    let home = env::var("HOME").unwrap_or_else(|_| String::from("/default"));
    println!("HOME: {}", home);
    
    // List all variables
    for (key, value) in env::vars() {
        println!("{}: {}", key, value);
    }
}
```

### Setting Environment Variables
```rust
use std::env;

fn main() {
    // Set variable
    env::set_var("MY_VAR", "my_value");
    
    // Read it back
    let val = env::var("MY_VAR").unwrap();
    println!("MY_VAR: {}", val);
    
    // Remove variable
    env::remove_var("MY_VAR");
}
```

---

## Writing to Standard Error

### Using eprintln!
```rust
use std::io::{self, Write};

fn main() {
    // Write to stdout
    println!("This goes to stdout");
    
    // Write to stderr
    eprintln!("This goes to stderr");
    
    // Using write! macros
    writeln!(io::stdout(), "To stdout").unwrap();
    writeln!(io::stderr(), "To stderr").unwrap();
}
```

---

## Summary

In this intermediate tutorial, you learned:
- Working with collections (Vec, String, HashMap)
- Generic types, traits, and lifetimes
- Functional programming with iterators and closures
- Smart pointers (Box, Rc, RefCell)
- Fearless concurrency with threads and channels
- Testing your code
- File I/O and command-line programs

### Next Steps
Continue to the **Advanced Tutorial** to learn:
- Advanced type system features
- Macros
- Unsafe Rust
- Advanced traits
- Advanced patterns
- And much more!

---

**Keep Rustling! 🦀**
