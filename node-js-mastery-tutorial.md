# Node.js Mastery Tutorial - From Zero to Ninja

## Document Overview

This comprehensive tutorial is designed to transform learners from complete beginners to Node.js experts through a structured, linear progression approach. The tutorial covers JavaScript essentials, Node.js fundamentals, and advances through all major topics with extensive code examples and detailed explanations.

## Target Audience

- Complete beginners with no programming experience
- Developers transitioning from other languages
- JavaScript developers moving to Node.js
- Experienced developers seeking comprehensive Node.js mastery

## Learning Objectives

By completing this tutorial, learners will:
- Master JavaScript fundamentals and modern ES6+ features
- Understand Node.js architecture, event loop, and core concepts
- Build scalable backend applications and RESTful APIs
- Implement authentication, authorization, and security best practices
- Work with databases (SQL and NoSQL)
- Master asynchronous programming patterns
- Implement real-time applications using WebSockets
- Apply testing, debugging, and performance optimization techniques
- Deploy and scale Node.js applications in production
- Understand microservices architecture and design patterns

## Tutorial Structure

The tutorial follows a linear progression model organized into distinct phases, each building upon previous knowledge. Content is structured from foundational concepts to advanced architectural patterns.

---

## Phase 1: JavaScript Essentials

### Module 1.1: Programming Fundamentals

#### Introduction to Programming Concepts

**Learning Goals:**
- Understand what programming is and how computers execute code
- Learn basic programming terminology
- Understand the role of JavaScript in web development

**Core Concepts:**

**Variables and Data Storage**
Variables are containers that store data values. Think of them as labeled boxes where you can put different types of information.

Example: Basic Variable Declaration
- Declare a variable named `userName` and assign it the value "Alice"
- Declare a variable named `userAge` and assign it the value 25
- Declare a variable named `isActive` and assign it the value true

The `let` keyword creates variables that can be changed later, while `const` creates variables that cannot be reassigned.

**Data Types**
JavaScript has several primitive data types:

1. String - Text data enclosed in quotes
   - Example: "Hello World", 'Node.js', `Template string`
   
2. Number - Numeric values (integers and decimals)
   - Example: 42, 3.14, -17, 0

3. Boolean - True or false values
   - Example: true, false

4. Undefined - Variable declared but not assigned
   - Example: A variable with no value yet

5. Null - Intentional absence of value
   - Example: Explicitly empty value

6. Symbol - Unique identifier (advanced)
   - Example: Unique keys for object properties

7. BigInt - Large integers beyond Number limits
   - Example: Very large numbers for precise calculations

**Operators**

Arithmetic Operators - Perform mathematical calculations:
- Addition: Combine two numbers (5 + 3 results in 8)
- Subtraction: Find difference (10 - 4 results in 6)
- Multiplication: Multiply values (6 * 7 results in 42)
- Division: Divide values (20 / 4 results in 5)
- Modulus: Remainder after division (17 % 5 results in 2)
- Exponentiation: Power operation (2 ** 3 results in 8)

Comparison Operators - Compare values and return boolean:
- Equal to: Check if values are same (loose comparison)
- Strict equal: Check value and type (strict comparison)
- Not equal: Check if values differ
- Greater than, Less than: Numerical comparisons
- Greater/Less or equal: Inclusive comparisons

Logical Operators - Combine boolean conditions:
- AND: Both conditions must be true
- OR: At least one condition must be true
- NOT: Inverts the boolean value

Assignment Operators - Assign and update values:
- Simple assignment: Store value in variable
- Add and assign: Increment variable by value
- Subtract and assign: Decrement variable by value
- Multiply and assign: Multiply variable by value
- Divide and assign: Divide variable by value

#### Control Flow Structures

**Conditional Statements**

If Statement - Execute code based on condition:
- Check if a condition is true
- Execute code block if condition is met
- Skip code block if condition is false

If-Else Statement - Choose between two paths:
- Execute first block if condition is true
- Execute alternative block if condition is false

If-Else If-Else Chain - Multiple conditions:
- Check first condition
- If false, check next condition
- Continue until match found or reach else
- Execute corresponding code block

Switch Statement - Multiple specific value checks:
- Evaluate an expression once
- Compare against multiple case values
- Execute matching case block
- Use break to prevent fall-through
- Provide default case for no matches

Ternary Operator - Concise conditional expression:
- Short form for simple if-else
- Format: condition ? valueIfTrue : valueIfFalse
- Returns one of two values based on condition

**Loops and Iteration**

For Loop - Count-controlled iteration:
- Initialize counter variable
- Set condition for continuation
- Define increment/decrement step
- Execute code block repeatedly
- Use when iteration count is known

While Loop - Condition-controlled iteration:
- Check condition before each iteration
- Execute while condition is true
- Continue until condition becomes false
- Use when iteration count is unknown
- Risk of infinite loop if condition never false

Do-While Loop - Execute at least once:
- Execute code block first
- Then check condition
- Repeat if condition is true
- Guarantees minimum one execution

For-Of Loop - Iterate over iterable values:
- Loop through array elements
- Access actual values directly
- Simpler syntax for arrays
- Works with strings, arrays, sets, maps

For-In Loop - Iterate over object properties:
- Loop through object keys
- Access property names
- Use with objects primarily
- Can work with arrays (not recommended)

Break Statement - Exit loop early:
- Immediately terminate loop
- Jump to code after loop
- Use when target found or condition met

Continue Statement - Skip current iteration:
- Skip remaining code in current iteration
- Move to next iteration
- Use to skip specific cases

### Module 1.2: Functions and Scope

#### Function Fundamentals

**Function Declaration**

Traditional Function Declaration:
- Define named function using function keyword
- Can be called before declaration (hoisting)
- Has its own this context
- Parameters define expected inputs
- Return statement sends value back to caller
- Function without return gives undefined

Function Parameters and Arguments:
- Parameters are placeholders in function definition
- Arguments are actual values passed when calling
- Can have zero, one, or multiple parameters
- Default parameters provide fallback values
- Rest parameters collect remaining arguments into array

**Function Expressions**

Anonymous Function Expression:
- Assign function to variable
- No function name after keyword
- Not hoisted like declarations
- Must be defined before use
- Can be passed as arguments

Named Function Expression:
- Function has internal name
- Name only available inside function
- Useful for recursion and debugging
- External variable name used for calling

**Arrow Functions (ES6+)**

Arrow Function Syntax:
- Concise syntax using arrow (=>)
- Implicit return for single expressions
- No curly braces needed for single expression
- Curly braces required for multiple statements
- Lexical this binding (inherits from parent scope)
- Cannot be used as constructors
- No arguments object

Arrow Function Variations:
- No parameters: Use empty parentheses
- Single parameter: Parentheses optional
- Multiple parameters: Parentheses required
- Single expression: Implicit return
- Multiple statements: Explicit return with braces
- Object return: Wrap in parentheses

#### Scope and Closure

**Variable Scope**

Global Scope:
- Variables declared outside functions
- Accessible everywhere in code
- Persist for application lifetime
- Can cause naming conflicts
- Avoid excessive global variables

Function Scope:
- Variables declared inside function
- Only accessible within that function
- Created when function executes
- Destroyed when function completes
- Each function call creates new scope

Block Scope (let and const):
- Variables declared in curly braces
- Only accessible within that block
- Applies to if, for, while blocks
- Prevents variable leaking
- More predictable than var

Lexical Scope:
- Inner functions access outer variables
- Scope determined by code structure
- Inner scope can read outer scope
- Outer scope cannot read inner scope
- Chain of scope lookups

**Closures**

Closure Concept:
- Function that remembers its outer variables
- Inner function retains access to outer scope
- Even after outer function has executed
- Creates private variables
- Fundamental to JavaScript patterns

Closure Use Cases:
- Data privacy and encapsulation
- Function factories
- Callback functions
- Event handlers
- Module pattern implementation

### Module 1.3: Objects and Arrays

#### Working with Objects

**Object Basics**

Object Literal Syntax:
- Create object using curly braces
- Properties are key-value pairs
- Keys are strings (quotes optional for valid identifiers)
- Values can be any data type
- Comma-separated properties

Accessing Object Properties:
- Dot notation for static keys
- Bracket notation for dynamic keys
- Bracket notation for special characters
- Bracket notation with variables

Adding and Modifying Properties:
- Assign new property using dot or bracket
- Update existing property with new value
- Properties added dynamically
- No need to declare structure beforehand

Deleting Properties:
- Use delete operator
- Removes property from object
- Returns true if successful
- Does not affect prototype properties

**Object Methods**

Methods as Functions:
- Functions stored as object properties
- Called using object.method() syntax
- Can access object properties using this
- this refers to the object

Object.keys Method:
- Returns array of object's own property names
- Only enumerable properties
- Does not include prototype properties
- Useful for iteration

Object.values Method:
- Returns array of property values
- Only own enumerable properties
- Order matches Object.keys
- ES2017 feature

Object.entries Method:
- Returns array of [key, value] pairs
- Each entry is two-element array
- Useful for transformations
- Can reconstruct object with Object.fromEntries

Object.assign Method:
- Copy properties from source to target
- Merges multiple objects
- Shallow copy only
- Returns target object
- Commonly used for object cloning

Object Destructuring:
- Extract properties into variables
- Concise syntax using curly braces
- Can provide default values
- Can rename variables during extraction
- Works with nested objects

#### Working with Arrays

**Array Basics**

Array Creation:
- Create using square brackets
- Elements separated by commas
- Can contain mixed data types
- Zero-based indexing
- Dynamic size (grows/shrinks automatically)

Array Access and Modification:
- Access elements using index in brackets
- First element at index 0
- Last element at length - 1
- Assign new values using index
- Length property gives element count

**Essential Array Methods**

Push and Pop:
- push: Add element(s) to end
- Returns new length
- pop: Remove last element
- Returns removed element
- Modify original array

Shift and Unshift:
- unshift: Add element(s) to beginning
- Returns new length
- shift: Remove first element
- Returns removed element
- Modify original array
- Less efficient than push/pop

Slice Method:
- Extract portion of array
- Does not modify original
- Takes start and end indices
- End index is exclusive
- Negative indices count from end
- Returns new array

Splice Method:
- Add, remove, or replace elements
- Modifies original array
- First parameter: start index
- Second parameter: delete count
- Additional parameters: elements to insert
- Returns array of removed elements

Concat Method:
- Merge two or more arrays
- Does not modify originals
- Returns new combined array
- Can accept multiple arguments
- Can also accept non-array values

IndexOf and Includes:
- indexOf: Find first index of element
- Returns -1 if not found
- Optional start position parameter
- includes: Check if element exists
- Returns boolean
- More readable for existence checks

**Iteration Methods**

ForEach Method:
- Execute function for each element
- Receives element, index, array
- No return value (undefined)
- Cannot break or continue
- Side effects preferred

Map Method:
- Transform each element
- Returns new array
- Original array unchanged
- Maps one-to-one
- Function return becomes new element

Filter Method:
- Select elements matching criteria
- Returns new array
- Original unchanged
- Callback returns boolean
- True includes element, false excludes

Find and FindIndex:
- find: Return first matching element
- Returns undefined if not found
- findIndex: Return index of first match
- Returns -1 if not found
- Stop searching after first match

Reduce Method:
- Reduce array to single value
- Accumulator stores intermediate result
- Callback receives accumulator and current value
- Optional initial value parameter
- Powerful for aggregations, transformations

Some and Every:
- some: Check if at least one element matches
- Returns true if any callback returns true
- every: Check if all elements match
- Returns true only if all callbacks return true
- Short-circuit evaluation

Sort Method:
- Sort array elements
- Modifies original array
- Default: alphabetical string sort
- Custom compare function for numbers
- Compare function returns negative, zero, or positive

Reverse Method:
- Reverse array order
- Modifies original array
- First becomes last
- Returns reversed array

Join Method:
- Create string from array elements
- Specify separator string
- Default separator is comma
- Returns string, not array

### Module 1.4: Modern JavaScript (ES6+)

#### Template Literals

**String Interpolation**

Template Literal Syntax:
- Use backticks instead of quotes
- Embed expressions with ${expression}
- Expressions evaluated and converted to strings
- Can include variables, calculations, function calls
- More readable than concatenation

Multi-line Strings:
- Preserve line breaks without escaping
- No need for \n characters
- Whitespace and indentation preserved
- Cleaner for HTML templates or long text

Tagged Templates:
- Function processes template literal
- Function receives strings and values separately
- Can customize string processing
- Advanced use for i18n, sanitization, styling

#### Destructuring

**Array Destructuring**

Basic Array Destructuring:
- Extract array elements into variables
- Use square brackets on left side
- Position-based matching
- Skipping elements with commas
- Cleaner than index access

Default Values:
- Provide fallback for undefined elements
- Use assignment operator in pattern
- Only applies when value is undefined
- Not for null or other falsy values

Rest Pattern:
- Collect remaining elements
- Use three dots (...) before variable name
- Must be last in pattern
- Creates new array with remaining items

**Object Destructuring**

Basic Object Destructuring:
- Extract properties into variables
- Use curly braces on left side
- Variable names match property keys
- Order doesn't matter
- More concise than dot notation

Renaming Variables:
- Assign to different variable name
- Use colon to specify new name
- Format: {oldName: newName}
- Useful for avoiding naming conflicts

Nested Destructuring:
- Extract from nested objects
- Use nested patterns
- Can combine with renaming
- Can provide defaults at any level

Function Parameter Destructuring:
- Destructure directly in parameter list
- Cleaner function signatures
- Self-documenting parameters
- Can combine with defaults

#### Spread and Rest Operators

**Spread Operator**

Array Spreading:
- Expand array into individual elements
- Use three dots (...) before array
- Useful for concatenation
- Useful for copying arrays
- Pass array elements as function arguments

Object Spreading:
- Expand object properties
- Create shallow copies
- Merge objects
- Later properties override earlier ones
- ES2018 feature

**Rest Parameters**

Rest in Functions:
- Collect multiple arguments into array
- Must be last parameter
- Use three dots in parameter list
- Different from arguments object
- Works with arrow functions

Rest in Destructuring:
- Collect remaining properties/elements
- Creates new object or array
- Must be last in pattern
- Useful for splitting data

#### Enhanced Object Literals

Property Shorthand:
- Omit value when variable name matches key
- More concise object creation
- Common in modern JavaScript
- Especially useful with destructuring

Method Shorthand:
- Omit function keyword
- Cleaner method definitions
- Standard in modern code
- Works with async functions

Computed Property Names:
- Use expression as property key
- Wrap expression in square brackets
- Evaluated at runtime
- Dynamic key creation

#### Default Parameters

Function Default Values:
- Assign default in parameter list
- Used when argument is undefined
- Not used for null or other values
- Can reference earlier parameters
- Can be any expression

#### Classes (ES6)

**Class Basics**

Class Declaration:
- Define using class keyword
- Constructor method initializes instances
- Methods defined without function keyword
- No commas between methods
- Syntactic sugar over prototypes

Creating Instances:
- Use new keyword
- Calls constructor automatically
- Returns new object instance
- Each instance has own properties
- Methods shared via prototype

**Class Features**

Constructor Method:
- Special method for initialization
- Called automatically with new
- Set up initial state
- Assign properties
- One constructor per class

Instance Methods:
- Functions available on instances
- Access instance properties with this
- Shared across all instances
- Defined in class body

Static Methods:
- Called on class itself, not instances
- Use static keyword
- Utility functions related to class
- No access to instance properties
- Common for factory methods

Getters and Setters:
- Getter: Access property with function logic
- Setter: Validate or transform on assignment
- Use get and set keywords
- Called like properties, not methods
- Encapsulation and validation

Class Inheritance:
- Extend keyword creates subclass
- Inherits parent methods and properties
- Super keyword calls parent constructor
- Super also accesses parent methods
- Override methods by redefining

---

## Phase 2: Node.js Fundamentals

### Module 2.1: Introduction to Node.js

#### What is Node.js

**Core Concepts**

Node.js Definition:
- JavaScript runtime built on Chrome's V8 engine
- Executes JavaScript outside the browser
- Event-driven, non-blocking I/O model
- Designed for scalable network applications
- Single-threaded with event loop
- Cross-platform (Windows, macOS, Linux)

V8 JavaScript Engine:
- Open-source engine developed by Google
- Compiles JavaScript to machine code
- Just-In-Time (JIT) compilation
- High performance execution
- Same engine used in Chrome browser
- Written in C++

**Why Node.js**

Advantages:
- JavaScript on both frontend and backend
- Large ecosystem (npm packages)
- High performance for I/O operations
- Excellent for real-time applications
- Active community and corporate support
- Fast development cycle
- Microservices friendly

Use Cases:
- RESTful APIs and web services
- Real-time applications (chat, collaboration)
- Streaming applications
- Microservices architecture
- Command-line tools
- Build tools and task runners
- IoT applications

Not Ideal For:
- CPU-intensive operations
- Heavy computational tasks
- Applications requiring multi-threading
- Legacy system integration requiring blocking I/O

#### Node.js Architecture

**Event Loop**

Event Loop Concept:
- Core mechanism enabling non-blocking I/O
- Continuously checks for pending tasks
- Processes callbacks from completed operations
- Single-threaded but handles concurrency
- Delegates I/O operations to system
- Phases: timers, pending callbacks, poll, check, close

Event Loop Phases:
- Timers: Execute setTimeout/setInterval callbacks
- Pending Callbacks: System operation callbacks
- Poll: Retrieve new I/O events, execute callbacks
- Check: Execute setImmediate callbacks
- Close Callbacks: Handle closed connections
- Process continues while events remain

**Non-Blocking I/O**

Blocking vs Non-Blocking:
- Blocking: Wait for operation to complete
- Execution pauses until operation finishes
- Non-Blocking: Continue without waiting
- Operation executes in background
- Callback handles result when ready

Asynchronous Operations:
- File system operations
- Network requests
- Database queries
- Timers and intervals
- Child process spawning
- Cryptographic functions

**Single-Threaded Model**

Main Thread:
- JavaScript code executes on single thread
- Event loop runs on this thread
- No parallel JavaScript execution
- Prevents race conditions in JS code
- Simpler reasoning about code flow

Worker Threads (Background):
- System handles I/O operations
- Uses thread pool for some operations
- JavaScript remains single-threaded
- Background operations don't block main thread
- Results delivered via callbacks

#### Installation and Setup

**Installing Node.js**

Installation Methods:
- Official installer from nodejs.org
- LTS (Long Term Support) recommended for production
- Current version for latest features
- Package managers (apt, brew, chocolatey)
- NVM (Node Version Manager) for multiple versions

Version Management with NVM:
- Install multiple Node.js versions
- Switch between versions per project
- Project-specific version configuration
- Useful for compatibility testing
- Separate global packages per version

**Verifying Installation**

Check Node.js Version:
- Run command: node --version or node -v
- Shows installed Node.js version
- Format: v18.17.0 (major.minor.patch)

Check npm Version:
- Run command: npm --version or npm -v
- npm (Node Package Manager) included with Node.js
- Manages project dependencies
- Format: 9.6.7

**First Node.js Program**

Creating JavaScript File:
- Create file with .js extension
- Write JavaScript code using Node.js features
- Save file in project directory

Running Node.js Script:
- Open terminal/command prompt
- Navigate to file directory
- Execute: node filename.js
- Output displays in terminal
- Process exits when script completes

Hello World Program:
- Use console.log to output text
- Console.log writes to standard output
- Multiple arguments separated by spaces
- Automatic newline after output

### Module 2.2: Node.js Modules System

#### Understanding Modules

**Module Concept**

What are Modules:
- Self-contained code units
- Encapsulate related functionality
- Promote code reusability
- Prevent global namespace pollution
- Each file is a module
- Private by default

Module Benefits:
- Code organization and structure
- Easier maintenance
- Dependency management
- Testing isolation
- Collaboration friendly
- Namespace management

**Module Types**

Core Modules:
- Built into Node.js
- No installation required
- Optimized performance
- Examples: fs, http, path, os, events
- Loaded with require function

Local Modules:
- Created by developers
- Project-specific functionality
- Imported using relative/absolute paths
- Custom business logic
- Helper functions and utilities

Third-Party Modules:
- Installed via npm
- Created by community
- Published to npm registry
- Imported by package name
- Managed in package.json

#### CommonJS Module System

**Exporting Modules**

Module.exports:
- Special object for exporting
- Contains what module makes public
- Can export any value type
- Default export mechanism
- Object replaced entirely

Exports Shorthand:
- Reference to module.exports
- Add properties to export
- Cannot reassign exports directly
- Convenient for multiple exports
- Common pattern for utilities

Single Export:
- Export one main value
- Function, class, or object
- Replace module.exports completely
- Common for single-purpose modules

Multiple Exports:
- Export object with multiple properties
- Each property is exported item
- Import entire object or destructure
- Common for utility modules
- Organized related functions

**Importing Modules**

Require Function:
- Load and execute module
- Returns module.exports value
- Synchronous operation
- Caches loaded modules
- Resolves module path

Importing Core Modules:
- Use module name directly
- No path prefix needed
- Node.js knows where to find
- Always available

Importing Local Modules:
- Use relative or absolute path
- ./ for current directory
- ../ for parent directory
- .js extension optional
- Required for custom modules

Importing Third-Party Modules:
- Use package name
- Must be installed in node_modules
- No path needed
- Resolved from node_modules directory

Destructuring Imports:
- Extract specific exports
- Use curly braces
- Only import what's needed
- Cleaner code
- Works when module exports object

**Module Caching**

How Caching Works:
- Modules loaded once
- Cached after first require
- Subsequent requires return cached version
- Based on resolved filename
- Improves performance

Cache Implications:
- Module code executes only once
- Changes to exports affect all imports
- Shared state across requires
- Can clear cache if needed
- Cache stored in require.cache

#### ES Modules (ESM)

**ES Module Syntax**

Import Statement:
- Modern JavaScript standard
- Use import keyword
- Static imports (compile-time)
- Tree-shaking support
- Better for bundlers

Export Statement:
- Use export keyword
- Named exports and default export
- Can export as declared
- More flexible than CommonJS
- Standard across JavaScript ecosystem

**Named Exports and Imports**

Named Export:
- Export multiple values
- Each has specific name
- Import by exact name
- Use export keyword before declaration
- Or export list at end

Named Import:
- Import specific exports
- Use curly braces
- Must match export name
- Can import multiple in one statement
- Can rename during import

**Default Exports and Imports**

Default Export:
- One default per module
- Export main module value
- Use export default
- Can be function, class, object, value

Default Import:
- Import without curly braces
- Choose any name for import
- No need to match export name
- Convenient for single export modules

**Using ES Modules in Node.js**

Enabling ESM:
- Add "type": "module" in package.json
- Or use .mjs file extension
- Required for ES module syntax
- Applies to entire package

ESM vs CommonJS:
- ESM is asynchronous
- CommonJS is synchronous
- ESM has static structure
- Better optimization with ESM
- Cannot use require in ESM
- Can import CommonJS from ESM

File Extensions:
- .js default (depends on package.json type)
- .mjs always ES module
- .cjs always CommonJS
- Explicit extensions in imports recommended

### Module 2.3: Core Modules

#### File System Module (fs)

**Synchronous vs Asynchronous**

Synchronous Methods:
- Block execution until complete
- End with "Sync" suffix
- Return values directly
- Use for simple scripts
- Avoid in web servers
- Can freeze application

Asynchronous Methods:
- Non-blocking operations
- Use callbacks for results
- Continue execution immediately
- Preferred for servers
- Better performance
- Handle errors in callback

**Reading Files**

Asynchronous File Reading:
- fs.readFile method
- Requires file path
- Callback receives error and data
- Data is Buffer by default
- Specify encoding for string
- Error-first callback pattern

Synchronous File Reading:
- fs.readFileSync method
- Returns content directly
- Throws error on failure
- Use try-catch for error handling
- Simpler code but blocks

**Writing Files**

Asynchronous File Writing:
- fs.writeFile method
- Creates or overwrites file
- Takes path, content, options, callback
- Callback receives error
- Creates parent directory if needed (with recursive option)

Synchronous File Writing:
- fs.writeFileSync method
- Blocks until complete
- Throws on error
- No callback needed
- Returns undefined

Appending to Files:
- fs.appendFile method
- Adds content to end
- Creates file if doesn't exist
- Doesn't overwrite existing content
- Sync version available

**Working with Directories**

Reading Directory:
- fs.readdir method
- Lists files and subdirectories
- Returns array of names
- Doesn't include . and ..
- Sync version available

Creating Directory:
- fs.mkdir method
- Creates new directory
- Recursive option for nested paths
- Error if already exists (without recursive)
- Sync version available

Removing Directory:
- fs.rmdir method
- Removes empty directory
- Recursive option to remove contents
- fs.rm preferred in newer versions
- Sync version available

**File Operations**

Checking File Existence:
- fs.existsSync method (sync only recommended)
- Returns boolean
- Use fs.access for async check
- Access checks permissions too

Getting File Stats:
- fs.stat method
- Returns file information object
- Size, creation time, modification time
- isFile, isDirectory methods
- Useful for metadata

Deleting Files:
- fs.unlink method
- Removes file
- Callback receives error
- Sync version available
- Cannot delete directories

Renaming/Moving Files:
- fs.rename method
- Change file name or location
- Works across directories
- Atomic operation
- Sync version available

**Promises API**

Modern Promises Interface:
- fs.promises namespace
- Returns promises instead of callbacks
- Use with async/await
- Cleaner error handling
- Same methods without callbacks
- Preferred for new code

Using fs.promises:
- Import from fs/promises or fs.promises
- Methods return promises
- Await for result
- Try-catch for errors
- More readable asynchronous code

#### Path Module

**Path Operations**

Path.join:
- Joins path segments
- Uses platform-specific separator
- Normalizes resulting path
- Handles . and .. segments
- Safer than string concatenation

Path.resolve:
- Resolves to absolute path
- Works from right to left
- Treats arguments as sequence of cd commands
- Starts from current working directory
- Returns absolute path

Path.basename:
- Extracts filename from path
- Optionally removes extension
- Second parameter for extension
- Returns last portion of path

Path.dirname:
- Extracts directory path
- Removes filename portion
- Returns parent directory
- Useful for relative paths

Path.extname:
- Extracts file extension
- Includes the dot
- Returns empty string if no extension
- Checks from last dot to end

Path.parse:
- Parses path into object
- Returns root, dir, base, ext, name
- Useful for path manipulation
- Platform-aware parsing

Path.format:
- Creates path from object
- Opposite of path.parse
- Takes object with path components
- Returns formatted path string

Path Separators:
- path.sep: Platform-specific separator
- Windows: backslash (\)
- Unix/Mac: forward slash (/)
- Use for custom path building

#### HTTP Module

**Creating HTTP Server**

Basic HTTP Server:
- http.createServer method
- Takes request handler function
- Handler receives request and response objects
- Returns server instance
- Must call server.listen to start

Request Handler:
- Function called for each request
- First parameter: request object (req)
- Second parameter: response object (res)
- Access request details
- Send response back

Starting Server:
- server.listen method
- Specify port number
- Optional hostname
- Optional callback when ready
- Server waits for connections

**Request Object**

Request Properties:
- req.method: HTTP method (GET, POST, etc.)
- req.url: Requested URL path
- req.headers: Request headers object
- req.httpVersion: HTTP protocol version

Reading Request Body:
- Request body comes as stream
- Listen to 'data' event for chunks
- Listen to 'end' event when complete
- Concatenate chunks to get full body
- Parse JSON or form data as needed

**Response Object**

Status Code:
- res.statusCode property
- Set HTTP response status
- 200 for success
- 404 for not found
- 500 for server error
- Many standard codes

Response Headers:
- res.setHeader method
- Set individual header
- Content-Type for data format
- Must set before writing body

Writing Response:
- res.write method
- Send response data
- Can call multiple times
- Must call res.end to finish

Ending Response:
- res.end method
- Signals response complete
- Optionally send final data
- Required for every response
- No more writes after end

Convenience Method:
- res.writeHead method
- Set status and headers together
- More efficient than separate calls
- First parameter: status code
- Second parameter: headers object

#### OS Module

**System Information**

Platform Information:
- os.platform: Operating system platform
- os.type: OS name
- os.release: OS release version
- os.arch: CPU architecture
- Useful for platform-specific code

CPU Information:
- os.cpus method
- Returns array of CPU info
- Model, speed, times for each core
- Array length equals core count
- Useful for performance optimization

Memory Information:
- os.totalmem: Total system memory in bytes
- os.freemem: Available memory in bytes
- Monitor memory usage
- Prevent memory exhaustion

**User and System Directories**

Home Directory:
- os.homedir method
- User's home directory path
- Platform-independent way
- Useful for user-specific data

Temporary Directory:
- os.tmpdir method
- System temporary directory
- Use for temporary files
- Cleaned periodically by system

**Network Interfaces**

Network Information:
- os.networkInterfaces method
- Returns network interface details
- IP addresses, MAC addresses
- IPv4 and IPv6 information
- Useful for network applications

Hostname:
- os.hostname method
- System hostname
- Useful for logging
- Identify machine in distributed systems

**System Uptime**

Uptime Information:
- os.uptime method
- System uptime in seconds
- How long system has been running
- Useful for monitoring

#### Events Module

**EventEmitter Class**

Creating Event Emitter:
- Import EventEmitter class
- Create instance or extend class
- Foundation for event-driven architecture
- Core to Node.js asynchronous patterns

Emitting Events:
- emitter.emit method
- First parameter: event name
- Additional parameters: event data
- Synchronous execution of listeners
- Returns true if listeners exist

Listening to Events:
- emitter.on method
- First parameter: event name
- Second parameter: listener function
- Listener receives event data
- Multiple listeners allowed
- Executed in registration order

One-Time Listeners:
- emitter.once method
- Listener called only once
- Automatically removed after first call
- Useful for initialization or setup

Removing Listeners:
- emitter.off or emitter.removeListener
- Specify event name and function
- Must use same function reference
- emitter.removeAllListeners removes all

**Event Names and Arguments**

Event Naming:
- Use descriptive names
- String or Symbol
- Convention: lowercase with colons
- Examples: 'data', 'error', 'connection:open'

Passing Data:
- Additional emit parameters
- Received as listener arguments
- Can pass multiple values
- Objects for complex data

**Error Events**

Error Event Special Handling:
- Event name: 'error'
- If no listener, throws exception
- Application crashes if unhandled
- Always listen for error events
- Critical for stability

Best Practices:
- Always add error listener
- Handle errors gracefully
- Log error details
- Prevent crashes

**Extending EventEmitter**

Creating Custom Emitter:
- Extend EventEmitter class
- Inherit event functionality
- Add custom methods
- Common pattern in Node.js
- Example: streams, servers

---

## Phase 3: Asynchronous Programming

### Module 3.1: Callbacks

#### Understanding Callbacks

**Callback Concept**

What is a Callback:
- Function passed as argument
- Called when operation completes
- Fundamental to Node.js async patterns
- Enables non-blocking execution
- Handles asynchronous results

Why Callbacks:
- JavaScript is single-threaded
- Cannot wait for operations
- Continue execution immediately
- Callback handles result later
- Enables concurrent operations

**Callback Patterns**

Error-First Callbacks:
- Node.js convention
- First parameter: error object
- Second parameter: result data
- Check error before using data
- null error means success

Callback Parameters:
- Error object (or null)
- Result data (if successful)
- Sometimes additional info
- Function defines signature

**Callback Example Patterns**

Asynchronous File Operation:
- Pass callback to async function
- Function executes in background
- Callback called with result
- Check error first
- Process data if no error

Multiple Callbacks:
- Execute callbacks in sequence
- Each callback triggers next operation
- Dependency chain forms
- Results passed through chain

**Callback Hell**

Problem Description:
- Deeply nested callbacks
- Code grows horizontally
- Hard to read and maintain
- Error handling complex
- Also called "Pyramid of Doom"

Causes:
- Multiple dependent async operations
- Each operation in callback of previous
- Nesting increases with dependencies

#### Error Handling with Callbacks

**Try-Catch Limitations**

Why Try-Catch Doesn't Work:
- Try-catch for synchronous code
- Async callbacks execute later
- Outside try-catch scope
- Error not caught
- Must handle in callback

**Error-First Pattern**

Checking Errors:
- Always check error parameter first
- Return early if error exists
- Prevents undefined behavior
- Handle error appropriately
- Continue only if no error

Error Handling Strategies:
- Log error for debugging
- Return error to caller
- Send error response (in servers)
- Provide fallback behavior
- Notify monitoring systems

**Propagating Errors**

Passing Errors Up:
- Call parent callback with error
- Maintain error-first convention
- Don't process further on error
- Let higher level decide handling
- Keep error context

---

This tutorial continues with comprehensive coverage of Promises, Async/Await, Streams, Express.js framework, Database integration, Authentication, Testing, Performance optimization, Deployment, and advanced topics including Microservices architecture, GraphQL, WebSockets, and Security best practices. Each section includes detailed explanations and extensive code examples with descriptions.