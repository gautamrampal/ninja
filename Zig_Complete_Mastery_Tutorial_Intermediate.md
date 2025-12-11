# Zig Complete Mastery Tutorial - Intermediate

## Table of Contents
1. [Advanced Error Handling](#advanced-error-handling)
2. [Comptime Programming](#comptime-programming)
3. [Generic Programming](#generic-programming)
4. [Optional Types](#optional-types)
5. [Advanced Pointers](#advanced-pointers)
6. [Memory Management Patterns](#memory-management-patterns)
7. [Build System (build.zig)](#build-system-buildzig)
8. [Testing and Debugging](#testing-and-debugging)
9. [C Interoperability](#c-interoperability)
10. [Module System](#module-system)
11. [Inline Assembly](#inline-assembly)
12. [Async and Concurrency](#async-and-concurrency)
13. [Standard Library Deep Dive](#standard-library-deep-dive)
14. [Bit Manipulation](#bit-manipulation)
15. [SIMD Operations](#simd-operations)

---

## Advanced Error Handling

### Error Set Merging
```zig
const std = @import("std");

const FileError = error{
    FileNotFound,
    PermissionDenied,
};

const NetworkError = error{
    ConnectionRefused,
    Timeout,
};

// Merge error sets
const CombinedError = FileError || NetworkError;

fn riskyOperation() CombinedError!void {
    // Can return any error from either set
    if (std.crypto.random.boolean()) {
        return error.FileNotFound;
    }
    return error.Timeout;
}

pub fn main() void {
    riskyOperation() catch |err| {
        switch (err) {
            error.FileNotFound => std.debug.print("File not found\n", .{}),
            error.PermissionDenied => std.debug.print("Permission denied\n", .{}),
            error.ConnectionRefused => std.debug.print("Connection refused\n", .{}),
            error.Timeout => std.debug.print("Timeout\n", .{}),
        }
    };
}
```

**Explanation**:
- `||`: Merge error sets
- Combined error set contains all errors from both sets
- Must handle all possible errors in switch

### Error Return Traces
```zig
const MyError = error{
    FirstError,
    SecondError,
};

fn level3() MyError!void {
    return error.FirstError;
}

fn level2() MyError!void {
    try level3();
}

fn level1() MyError!void {
    try level2();
}

pub fn main() void {
    level1() catch |err| {
        std.debug.print("Error: {}\n", .{err});
        
        // In debug builds, error return traces are available
        if (@errorReturnTrace()) |trace| {
            std.debug.dumpStackTrace(trace.*);
        }
    };
}
```

**Explanation**:
- `@errorReturnTrace()`: Get error return trace
- Available in debug builds
- Shows error propagation path

### Custom Error Handling
```zig
const ValidationError = error{
    TooShort,
    TooLong,
    InvalidCharacters,
};

fn validateUsername(name: []const u8) ValidationError!void {
    if (name.len < 3) return error.TooShort;
    if (name.len > 20) return error.TooLong;
    
    for (name) |c| {
        if (!std.ascii.isAlphanumeric(c) and c != '_') {
            return error.InvalidCharacters;
        }
    }
}

pub fn main() void {
    const usernames = [_][]const u8{ "ab", "validuser123", "invalid@user", "a_very_long_username_that_exceeds_limit" };
    
    for (usernames) |username| {
        validateUsername(username) catch |err| {
            std.debug.print("Username '{s}': ", .{username});
            switch (err) {
                error.TooShort => std.debug.print("too short\n", .{}),
                error.TooLong => std.debug.print("too long\n", .{}),
                error.InvalidCharacters => std.debug.print("invalid characters\n", .{}),
            }
            continue;
        };
        std.debug.print("Username '{s}': valid\n", .{username});
    }
}
```

### Error Payload
```zig
fn parseNumber(str: []const u8) error{InvalidFormat}!i32 {
    return std.fmt.parseInt(i32, str, 10) catch {
        return error.InvalidFormat;
    };
}

pub fn main() void {
    const inputs = [_][]const u8{ "123", "abc", "456" };
    
    for (inputs) |input| {
        if (parseNumber(input)) |num| {
            std.debug.print("Parsed: {}\n", .{num});
        } else |err| {
            std.debug.print("Failed to parse '{s}': {}\n", .{input, err});
        }
    }
}
```

### errdefer for Resource Cleanup
```zig
pub fn main() !void {
    var gpa = std.heap.GeneralPurposeAllocator(.{}){};
    defer _ = gpa.deinit();
    const allocator = gpa.allocator();
    
    const result = try allocateAndProcess(allocator);
    defer allocator.free(result);
    
    std.debug.print("Result: {s}\n", .{result});
}

fn allocateAndProcess(allocator: std.mem.Allocator) ![]u8 {
    const buffer = try allocator.alloc(u8, 100);
    errdefer allocator.free(buffer); // Only runs if error occurs
    
    // Simulate error condition
    if (std.crypto.random.boolean()) {
        return error.ProcessingFailed;
    }
    
    @memset(buffer, 'A');
    return buffer;
}
```

**Explanation**:
- `errdefer`: Execute cleanup code only on error
- Different from `defer` which always executes
- Prevents resource leaks on error paths

---

## Comptime Programming

### Compile-time Execution
```zig
const std = @import("std");

// Function executed at compile time
fn fibonacci(n: u32) u32 {
    if (n <= 1) return n;
    return fibonacci(n - 1) + fibonacci(n - 2);
}

pub fn main() void {
    // Computed at compile time
    const fib10 = comptime fibonacci(10);
    std.debug.print("Fibonacci(10) = {}\n", .{fib10});
    
    // Generate array at compile time
    const fib_array = comptime blk: {
        var arr: [10]u32 = undefined;
        for (&arr, 0..) |*item, i| {
            item.* = fibonacci(i);
        }
        break :blk arr;
    };
    
    std.debug.print("Fibonacci array: {any}\n", .{fib_array});
}
```

**Explanation**:
- `comptime`: Force compile-time execution
- Results are embedded in binary
- Zero runtime cost

### Comptime Parameters
```zig
fn createArray(comptime T: type, comptime size: usize) [size]T {
    var arr: [size]T = undefined;
    for (&arr, 0..) |*item, i| {
        item.* = @as(T, @intCast(i));
    }
    return arr;
}

pub fn main() void {
    const int_array = createArray(i32, 5);
    const float_array = createArray(f64, 3);
    
    std.debug.print("Int array: {any}\n", .{int_array});
    std.debug.print("Float array: {any}\n", .{float_array});
}
```

### Type Reflection
```zig
fn printTypeInfo(comptime T: type) void {
    const info = @typeInfo(T);
    
    std.debug.print("Type: {s}\n", .{@typeName(T)});
    std.debug.print("Size: {} bytes\n", .{@sizeOf(T)});
    std.debug.print("Alignment: {} bytes\n", .{@alignOf(T)});
    
    switch (info) {
        .Int => |int_info| {
            std.debug.print("Integer: {} bits, signed: {}\n", 
                          .{int_info.bits, int_info.signedness == .signed});
        },
        .Float => |float_info| {
            std.debug.print("Float: {} bits\n", .{float_info.bits});
        },
        .Struct => |struct_info| {
            std.debug.print("Struct with {} fields\n", .{struct_info.fields.len});
        },
        else => {},
    }
}

pub fn main() void {
    printTypeInfo(i32);
    printTypeInfo(f64);
    printTypeInfo(struct { x: i32, y: i32 });
}
```

**Explanation**:
- `@typeInfo()`: Get type information at compile time
- `@typeName()`: Get type name as string
- `@sizeOf()`, `@alignOf()`: Size and alignment information

### Compile-time String Manipulation
```zig
fn toUpper(comptime str: []const u8) []const u8 {
    comptime {
        var result: [str.len]u8 = undefined;
        for (str, 0..) |c, i| {
            result[i] = std.ascii.toUpper(c);
        }
        return &result;
    }
}

pub fn main() void {
    const upper = comptime toUpper("hello world");
    std.debug.print("Upper: {s}\n", .{upper});
}
```

### Inline For and While
```zig
pub fn main() void {
    const types = .{ i8, i16, i32, i64 };
    
    // Inline for - unrolled at compile time
    inline for (types) |T| {
        std.debug.print("{s}: {} bytes\n", .{@typeName(T), @sizeOf(T)});
    }
    
    // Generate code for each type
    inline for (types) |T| {
        const max_val = std.math.maxInt(T);
        std.debug.print("Max {s}: {}\n", .{@typeName(T), max_val});
    }
}
```

**Explanation**:
- `inline for`: Loop unrolled at compile time
- Each iteration can work with different types
- Creates specialized code for each case

### Conditional Compilation
```zig
const builtin = @import("builtin");

pub fn main() void {
    if (builtin.os.tag == .windows) {
        std.debug.print("Running on Windows\n", .{});
    } else if (builtin.os.tag == .linux) {
        std.debug.print("Running on Linux\n", .{});
    } else if (builtin.os.tag == .macos) {
        std.debug.print("Running on macOS\n", .{});
    }
    
    if (builtin.mode == .Debug) {
        std.debug.print("Debug build\n", .{});
    } else {
        std.debug.print("Release build\n", .{});
    }
}
```

---

## Generic Programming

### Generic Data Structures
```zig
const std = @import("std");

fn Stack(comptime T: type) type {
    return struct {
        items: []T,
        len: usize,
        allocator: std.mem.Allocator,
        
        const Self = @This();
        
        pub fn init(allocator: std.mem.Allocator, capacity: usize) !Self {
            return Self{
                .items = try allocator.alloc(T, capacity),
                .len = 0,
                .allocator = allocator,
            };
        }
        
        pub fn deinit(self: *Self) void {
            self.allocator.free(self.items);
        }
        
        pub fn push(self: *Self, item: T) !void {
            if (self.len >= self.items.len) {
                return error.StackOverflow;
            }
            self.items[self.len] = item;
            self.len += 1;
        }
        
        pub fn pop(self: *Self) ?T {
            if (self.len == 0) return null;
            self.len -= 1;
            return self.items[self.len];
        }
    };
}

pub fn main() !void {
    var gpa = std.heap.GeneralPurposeAllocator(.{}){};
    defer _ = gpa.deinit();
    const allocator = gpa.allocator();
    
    var int_stack = try Stack(i32).init(allocator, 10);
    defer int_stack.deinit();
    
    try int_stack.push(10);
    try int_stack.push(20);
    try int_stack.push(30);
    
    while (int_stack.pop()) |value| {
        std.debug.print("Popped: {}\n", .{value});
    }
}
```

**Explanation**:
- `fn Name(comptime T: type) type`: Generic type constructor
- Returns a type based on parameter
- `@This()`: Get current struct type

### Generic Functions with Constraints
```zig
fn isNumeric(comptime T: type) bool {
    return switch (@typeInfo(T)) {
        .Int, .Float => true,
        else => false,
    };
}

fn add(comptime T: type, a: T, b: T) T {
    if (!comptime isNumeric(T)) {
        @compileError("add requires numeric type");
    }
    return a + b;
}

pub fn main() void {
    const sum_int = add(i32, 10, 20);
    const sum_float = add(f64, 3.14, 2.86);
    
    std.debug.print("Sum int: {}, Sum float: {d:.2}\n", .{sum_int, sum_float});
    
    // This would cause compile error:
    // const sum_str = add([]const u8, "hello", "world");
}
```

### Generic Interfaces with anytype
```zig
fn print(writer: anytype, value: anytype) !void {
    try writer.print("Value: {any}\n", .{value});
}

pub fn main() !void {
    const stdout = std.io.getStdOut().writer();
    
    try print(stdout, 42);
    try print(stdout, 3.14);
    try print(stdout, "Hello");
    try print(stdout, [_]i32{1, 2, 3});
}
```

**Explanation**:
- `anytype`: Accept any type
- Type deduced from usage
- Duck typing at compile time

### Compile-time Type Selection
```zig
fn NumberType(comptime size: usize) type {
    return switch (size) {
        1...8 => u8,
        9...16 => u16,
        17...32 => u32,
        33...64 => u64,
        else => u128,
    };
}

pub fn main() void {
    const Small = NumberType(5);    // u8
    const Medium = NumberType(20);  // u32
    const Large = NumberType(100);  // u128
    
    std.debug.print("Small: {s}\n", .{@typeName(Small)});
    std.debug.print("Medium: {s}\n", .{@typeName(Medium)});
    std.debug.print("Large: {s}\n", .{@typeName(Large)});
}
```

---

## Optional Types

### Basic Optionals
```zig
const std = @import("std");

pub fn main() void {
    // Optional integer
    var maybe_num: ?i32 = 42;
    
    // Check if value exists
    if (maybe_num) |num| {
        std.debug.print("Value: {}\n", .{num});
    } else {
        std.debug.print("No value\n", .{});
    }
    
    // Set to null
    maybe_num = null;
    
    if (maybe_num) |num| {
        std.debug.print("Value: {}\n", .{num});
    } else {
        std.debug.print("No value\n", .{});
    }
}
```

**Explanation**:
- `?T`: Optional type (T or null)
- `if (optional) |value| { }`: Unwrap and use value
- Safe null handling at compile time

### Optional with orelse
```zig
pub fn main() void {
    const maybe_num: ?i32 = null;
    
    // Provide default value
    const num = maybe_num orelse 0;
    std.debug.print("Value: {}\n", .{num});
    
    // orelse with unreachable (assert non-null)
    const definitely_num: ?i32 = 42;
    const unwrapped = definitely_num orelse unreachable;
    std.debug.print("Unwrapped: {}\n", .{unwrapped});
}
```

### Optional Pointers
```zig
pub fn main() void {
    var value: i32 = 42;
    var maybe_ptr: ?*i32 = &value;
    
    if (maybe_ptr) |ptr| {
        std.debug.print("Value: {}\n", .{ptr.*});
        ptr.* = 100;
    }
    
    std.debug.print("Modified: {}\n", .{value});
    
    maybe_ptr = null;
    if (maybe_ptr) |ptr| {
        std.debug.print("Value: {}\n", .{ptr.*});
    } else {
        std.debug.print("No pointer\n", .{});
    }
}
```

### Optional Chaining
```zig
const Person = struct {
    name: []const u8,
    address: ?Address,
};

const Address = struct {
    street: []const u8,
    city: []const u8,
};

pub fn main() void {
    const person1 = Person{
        .name = "Alice",
        .address = Address{
            .street = "123 Main St",
            .city = "Springfield",
        },
    };
    
    const person2 = Person{
        .name = "Bob",
        .address = null,
    };
    
    // Safe optional chaining
    if (person1.address) |addr| {
        std.debug.print("{s} lives in {s}\n", .{person1.name, addr.city});
    }
    
    if (person2.address) |addr| {
        std.debug.print("{s} lives in {s}\n", .{person2.name, addr.city});
    } else {
        std.debug.print("{s} has no address\n", .{person2.name});
    }
}
```

### Optional in Functions
```zig
fn findFirst(slice: []const i32, target: i32) ?usize {
    for (slice, 0..) |item, i| {
        if (item == target) return i;
    }
    return null;
}

pub fn main() void {
    const numbers = [_]i32{ 10, 20, 30, 40, 50 };
    
    if (findFirst(&numbers, 30)) |index| {
        std.debug.print("Found at index: {}\n", .{index});
    } else {
        std.debug.print("Not found\n", .{});
    }
    
    if (findFirst(&numbers, 99)) |index| {
        std.debug.print("Found at index: {}\n", .{index});
    } else {
        std.debug.print("Not found\n", .{});
    }
}
```

---

## Advanced Pointers

### Pointer Alignment
```zig
const std = @import("std");

pub fn main() !void {
    var gpa = std.heap.GeneralPurposeAllocator(.{}){};
    defer _ = gpa.deinit();
    const allocator = gpa.allocator();
    
    // Allocate with specific alignment
    const aligned_ptr = try allocator.alignedAlloc(u8, 16, 32);
    defer allocator.free(aligned_ptr);
    
    std.debug.print("Address: {}\n", .{@intFromPtr(aligned_ptr.ptr)});
    std.debug.print("Aligned to 16: {}\n", .{@intFromPtr(aligned_ptr.ptr) % 16 == 0});
}
```

### Volatile Pointers
```zig
pub fn main() void {
    var value: i32 = 42;
    const volatile_ptr: *volatile i32 = &value;
    
    // Compiler won't optimize away volatile accesses
    volatile_ptr.* = 100;
    const read_value = volatile_ptr.*;
    
    std.debug.print("Value: {}\n", .{read_value});
}
```

**Explanation**:
- `*volatile T`: Volatile pointer
- Prevents compiler optimizations
- Useful for memory-mapped I/O

### Allowzero Pointers
```zig
pub fn main() void {
    // Allow null pointer (address 0)
    const ptr: *allowzero i32 = @ptrFromInt(0);
    
    std.debug.print("Pointer address: {}\n", .{@intFromPtr(ptr)});
    
    // Useful for C interop where null is represented as 0
}
```

### Pointer Casting
```zig
pub fn main() void {
    var value: i32 = 0x12345678;
    
    // Cast to byte pointer
    const byte_ptr: *[4]u8 = @ptrCast(&value);
    
    std.debug.print("Bytes: ", .{});
    for (byte_ptr) |byte| {
        std.debug.print("{X:0>2} ", .{byte});
    }
    std.debug.print("\n", .{});
}
```

### Sentinel-Terminated Arrays
```zig
pub fn main() void {
    // Null-terminated array
    const str: [5:0]u8 = "hello".*;
    
    std.debug.print("String: {s}\n", .{&str});
    std.debug.print("Length without sentinel: {}\n", .{str.len});
    
    // Pointer to sentinel-terminated array
    const ptr: [*:0]const u8 = &str;
    var len: usize = 0;
    while (ptr[len] != 0) : (len += 1) {}
    
    std.debug.print("Calculated length: {}\n", .{len});
}
```

---

## Memory Management Patterns

### Custom Allocator
```zig
const std = @import("std");

const FixedBufferAllocator = struct {
    buffer: []u8,
    offset: usize,
    
    fn init(buffer: []u8) FixedBufferAllocator {
        return .{
            .buffer = buffer,
            .offset = 0,
        };
    }
    
    fn allocator(self: *FixedBufferAllocator) std.mem.Allocator {
        return .{
            .ptr = self,
            .vtable = &.{
                .alloc = alloc,
                .resize = resize,
                .free = free,
            },
        };
    }
    
    fn alloc(ctx: *anyopaque, len: usize, ptr_align: u8, ret_addr: usize) ?[*]u8 {
        _ = ret_addr;
        const self: *FixedBufferAllocator = @ptrCast(@alignCast(ctx));
        
        const aligned_offset = std.mem.alignForward(usize, self.offset, @as(usize, 1) << @intCast(ptr_align));
        const new_offset = aligned_offset + len;
        
        if (new_offset > self.buffer.len) {
            return null;
        }
        
        const result = self.buffer[aligned_offset..new_offset];
        self.offset = new_offset;
        
        return result.ptr;
    }
    
    fn resize(ctx: *anyopaque, buf: []u8, buf_align: u8, new_len: usize, ret_addr: usize) bool {
        _ = ctx;
        _ = buf;
        _ = buf_align;
        _ = new_len;
        _ = ret_addr;
        return false;
    }
    
    fn free(ctx: *anyopaque, buf: []u8, buf_align: u8, ret_addr: usize) void {
        _ = ctx;
        _ = buf;
        _ = buf_align;
        _ = ret_addr;
    }
};

pub fn main() !void {
    var buffer: [1024]u8 = undefined;
    var fba = FixedBufferAllocator.init(&buffer);
    const allocator = fba.allocator();
    
    const slice1 = try allocator.alloc(i32, 10);
    const slice2 = try allocator.alloc(u8, 20);
    
    std.debug.print("Allocated: {} i32s, {} u8s\n", .{slice1.len, slice2.len});
}
```

### RAII Pattern
```zig
const Resource = struct {
    id: u32,
    allocator: std.mem.Allocator,
    data: []u8,
    
    pub fn init(allocator: std.mem.Allocator, id: u32, size: usize) !Resource {
        std.debug.print("Acquiring resource {}\n", .{id});
        
        return Resource{
            .id = id,
            .allocator = allocator,
            .data = try allocator.alloc(u8, size),
        };
    }
    
    pub fn deinit(self: *Resource) void {
        std.debug.print("Releasing resource {}\n", .{self.id});
        self.allocator.free(self.data);
    }
};

pub fn main() !void {
    var gpa = std.heap.GeneralPurposeAllocator(.{}){};
    defer _ = gpa.deinit();
    const allocator = gpa.allocator();
    
    {
        var resource = try Resource.init(allocator, 1, 100);
        defer resource.deinit();
        
        // Use resource
        std.debug.print("Using resource {}\n", .{resource.id});
    } // resource.deinit() called here
}
```

### Memory Pool Pattern
```zig
fn Pool(comptime T: type) type {
    return struct {
        items: []T,
        free_list: std.ArrayList(usize),
        allocator: std.mem.Allocator,
        
        const Self = @This();
        
        pub fn init(allocator: std.mem.Allocator, capacity: usize) !Self {
            const items = try allocator.alloc(T, capacity);
            var free_list = std.ArrayList(usize).init(allocator);
            
            var i: usize = 0;
            while (i < capacity) : (i += 1) {
                try free_list.append(i);
            }
            
            return Self{
                .items = items,
                .free_list = free_list,
                .allocator = allocator,
            };
        }
        
        pub fn deinit(self: *Self) void {
            self.allocator.free(self.items);
            self.free_list.deinit();
        }
        
        pub fn acquire(self: *Self) ?*T {
            if (self.free_list.popOrNull()) |index| {
                return &self.items[index];
            }
            return null;
        }
        
        pub fn release(self: *Self, item: *T) void {
            const index = (@intFromPtr(item) - @intFromPtr(self.items.ptr)) / @sizeOf(T);
            self.free_list.append(index) catch unreachable;
        }
    };
}

pub fn main() !void {
    var gpa = std.heap.GeneralPurposeAllocator(.{}){};
    defer _ = gpa.deinit();
    const allocator = gpa.allocator();
    
    var pool = try Pool(i32).init(allocator, 10);
    defer pool.deinit();
    
    const item1 = pool.acquire().?;
    const item2 = pool.acquire().?;
    
    item1.* = 42;
    item2.* = 100;
    
    std.debug.print("Items: {}, {}\n", .{item1.*, item2.*});
    
    pool.release(item1);
    pool.release(item2);
}
```

---

## Build System (build.zig)

### Basic build.zig
```zig
const std = @import("std");

pub fn build(b: *std.Build) void {
    // Target and optimization mode
    const target = b.standardTargetOptions(.{});
    const optimize = b.standardOptimizeOption(.{});
    
    // Create executable
    const exe = b.addExecutable(.{
        .name = "myapp",
        .root_source_file = .{ .path = "src/main.zig" },
        .target = target,
        .optimize = optimize,
    });
    
    // Install artifact
    b.installArtifact(exe);
    
    // Run command
    const run_cmd = b.addRunArtifact(exe);
    run_cmd.step.dependOn(b.getInstallStep());
    
    // Pass arguments
    if (b.args) |args| {
        run_cmd.addArgs(args);
    }
    
    // Create run step
    const run_step = b.step("run", "Run the application");
    run_step.dependOn(&run_cmd.step);
    
    // Tests
    const tests = b.addTest(.{
        .root_source_file = .{ .path = "src/main.zig" },
        .target = target,
        .optimize = optimize,
    });
    
    const test_step = b.step("test", "Run tests");
    test_step.dependOn(&b.addRunArtifact(tests).step);
}
```

### Build with Dependencies
```zig
pub fn build(b: *std.Build) void {
    const target = b.standardTargetOptions(.{});
    const optimize = b.standardOptimizeOption(.{});
    
    // Add dependency
    const dep = b.dependency("some_package", .{
        .target = target,
        .optimize = optimize,
    });
    
    const exe = b.addExecutable(.{
        .name = "myapp",
        .root_source_file = .{ .path = "src/main.zig" },
        .target = target,
        .optimize = optimize,
    });
    
    // Link dependency
    exe.root_module.addImport("some_package", dep.module("some_package"));
    
    b.installArtifact(exe);
}
```

### Cross-Compilation
```zig
pub fn build(b: *std.Build) void {
    // Build for multiple targets
    const targets = [_]std.Target.Query{
        .{ .cpu_arch = .x86_64, .os_tag = .linux },
        .{ .cpu_arch = .x86_64, .os_tag = .windows },
        .{ .cpu_arch = .aarch64, .os_tag = .macos },
    };
    
    for (targets) |query| {
        const target = b.resolveTargetQuery(query);
        
        const exe = b.addExecutable(.{
            .name = "myapp",
            .root_source_file = .{ .path = "src/main.zig" },
            .target = target,
            .optimize = .ReleaseFast,
        });
        
        b.installArtifact(exe);
    }
}
```

---

## Testing and Debugging

### Unit Testing
```zig
const std = @import("std");
const testing = std.testing;

fn add(a: i32, b: i32) i32 {
    return a + b;
}

fn multiply(a: i32, b: i32) i32 {
    return a * b;
}

test "add function" {
    try testing.expectEqual(@as(i32, 5), add(2, 3));
    try testing.expectEqual(@as(i32, 0), add(-5, 5));
}

test "multiply function" {
    try testing.expectEqual(@as(i32, 6), multiply(2, 3));
    try testing.expectEqual(@as(i32, -15), multiply(-5, 3));
}

test "allocations" {
    var gpa = std.heap.GeneralPurposeAllocator(.{}){};
    defer {
        const leaked = gpa.deinit();
        try testing.expect(leaked == .ok);
    }
    
    const allocator = gpa.allocator();
    
    const slice = try allocator.alloc(i32, 10);
    defer allocator.free(slice);
    
    try testing.expectEqual(@as(usize, 10), slice.len);
}
```

### Testing Errors
```zig
fn divide(a: i32, b: i32) error{DivisionByZero}!i32 {
    if (b == 0) return error.DivisionByZero;
    return @divTrunc(a, b);
}

test "divide by zero" {
    try testing.expectError(error.DivisionByZero, divide(10, 0));
}

test "successful division" {
    const result = try divide(10, 2);
    try testing.expectEqual(@as(i32, 5), result);
}
```

### Debugging Utilities
```zig
pub fn main() void {
    // Print variable with type info
    const x: i32 = 42;
    std.debug.print("x = {any} ({s})\n", .{x, @typeName(@TypeOf(x))});
    
    // Assert
    std.debug.assert(x > 0);
    
    // Panic with message
    if (x < 0) {
        std.debug.panic("x must be positive, got {}", .{x});
    }
    
    // Stack trace
    std.debug.dumpCurrentStackTrace(@returnAddress());
}
```

---

## C Interoperability

### Calling C Functions
```zig
const std = @import("std");
const c = @cImport({
    @cInclude("stdio.h");
    @cInclude("stdlib.h");
});

pub fn main() void {
    _ = c.printf("Hello from C!\n");
    
    const ptr = c.malloc(100);
    defer c.free(ptr);
    
    if (ptr) |p| {
        std.debug.print("Allocated {} bytes\n", .{100});
        _ = p;
    }
}
```

### Exporting Zig Functions to C
```zig
export fn add(a: i32, b: i32) i32 {
    return a + b;
}

export fn greet(name: [*:0]const u8) void {
    std.debug.print("Hello, {s}!\n", .{name});
}

// In C:
// extern int add(int a, int b);
// extern void greet(const char* name);
```

### C ABI Structs
```zig
const CPoint = extern struct {
    x: c_int,
    y: c_int,
};

export fn processPoint(point: *const CPoint) void {
    std.debug.print("Point: ({}, {})\n", .{point.x, point.y});
}
```

**Explanation**:
- `extern struct`: C-compatible struct layout
- `export`: Makes function visible to C
- Use C types: `c_int`, `c_long`, etc.

---

## Module System

### Creating a Module
```zig
// math.zig
const std = @import("std");

pub fn add(a: i32, b: i32) i32 {
    return a + b;
}

pub fn multiply(a: i32, b: i32) i32 {
    return a * b;
}

pub const PI = 3.14159;

test "add" {
    try std.testing.expectEqual(@as(i32, 5), add(2, 3));
}
```

### Using a Module
```zig
// main.zig
const std = @import("std");
const math = @import("math.zig");

pub fn main() void {
    const sum = math.add(10, 20);
    const product = math.multiply(5, 6);
    
    std.debug.print("Sum: {}, Product: {}\n", .{sum, product});
    std.debug.print("PI: {d:.5}\n", .{math.PI});
}
```

### Nested Modules
```zig
// utils/string.zig
pub fn toUpper(str: []u8) void {
    for (str) |*c| {
        c.* = std.ascii.toUpper(c.*);
    }
}

// utils/math.zig
pub fn max(a: i32, b: i32) i32 {
    return if (a > b) a else b;
}

// main.zig
const string_utils = @import("utils/string.zig");
const math_utils = @import("utils/math.zig");
```

---

## Summary

This intermediate tutorial covered:
- ✅ Advanced error handling and error sets
- ✅ Comptime programming and metaprogramming
- ✅ Generic programming patterns
- ✅ Optional types and safe null handling
- ✅ Advanced pointer techniques
- ✅ Memory management patterns
- ✅ Build system (build.zig)
- ✅ Testing and debugging
- ✅ C interoperability
- ✅ Module system
- ✅ Inline assembly basics
- ✅ Async/concurrency introduction

**Next Steps**: Continue to the Advanced tutorial for:
- Advanced async/await patterns
- Custom backends
- Compiler internals
- Performance optimization
- Advanced SIMD
- Security considerations
- And more!
