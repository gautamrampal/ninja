# Testing - Unit and Performance Testing Complete Mastery Tutorial

## Table of Contents
1. [Introduction to Testing](#introduction-to-testing)
2. [Unit Testing Fundamentals](#unit-testing-fundamentals)
3. [Writing Unit Tests using Jest Framework](#writing-unit-tests-using-jest-framework)
4. [Test Driven Development (TDD) in Node.js](#test-driven-development-tdd-in-nodejs)
5. [Mocking Dependencies and API Calls](#mocking-dependencies-and-api-calls)
6. [Integration Testing of REST APIs](#integration-testing-of-rest-apis)
7. [Testing Asynchronous Code](#testing-asynchronous-code)
8. [Continuous Testing in CI/CD](#continuous-testing-in-cicd)
9. [Performance Testing with K6](#performance-testing-with-k6)
10. [Performance Testing and Monitoring with Postman](#performance-testing-with-postman)
11. [Best Practices and Patterns](#best-practices-and-patterns)
12. [Real-World Testing Scenarios](#real-world-testing-scenarios)

---

## Introduction to Testing

### What is Software Testing?

Software testing is the process of evaluating and verifying that a software application works as expected. It helps identify bugs, ensure quality, and validate that the software meets requirements.

### Types of Testing

```javascript
/*
 * Testing Pyramid - A visual representation of testing strategy
 * 
 *           /\
 *          /E2E\         End-to-End Tests (Few, Slow, Expensive)
 *         /------\
 *        /  Int.  \      Integration Tests (Some, Medium Speed)
 *       /----------\
 *      /   Unit     \    Unit Tests (Many, Fast, Cheap)
 *     /--------------\
 * 
 * Strategy: More unit tests at the base, fewer E2E tests at the top
 */

// Unit Tests: Test individual functions/methods in isolation
function add(a, b) {
  return a + b;
}

// Integration Tests: Test how components work together
async function getUserProfile(userId) {
  const user = await database.findUser(userId);
  const posts = await database.getUserPosts(userId);
  return { user, posts };
}

// End-to-End Tests: Test complete user workflows
// Example: User login -> Dashboard -> Create Post -> Logout
```

### Why Testing Matters

1. **Bug Detection**: Find issues before production
2. **Confidence**: Refactor code safely
3. **Documentation**: Tests serve as living documentation
4. **Design**: Forces better code architecture
5. **Regression Prevention**: Ensure old features still work

### Testing Terminology

```javascript
/*
 * Key Testing Concepts:
 * 
 * - Test Suite: Collection of related test cases
 * - Test Case: Individual test scenario
 * - Assertion: Expected vs Actual comparison
 * - Mock: Fake object that simulates real behavior
 * - Stub: Predefined responses for function calls
 * - Spy: Records information about function calls
 * - Fixture: Fixed state used as baseline for tests
 * - Coverage: Percentage of code tested
 */

// Example Test Structure
describe('Calculator Test Suite', () => {  // Test Suite
  test('should add two numbers', () => {    // Test Case
    expect(add(2, 3)).toBe(5);              // Assertion
  });
});
```

---

## Unit Testing Fundamentals

### What is Unit Testing?

Unit testing involves testing the smallest parts of an application (units) in isolation. A unit is typically a single function or method.

### Characteristics of Good Unit Tests

```javascript
/*
 * F.I.R.S.T Principles:
 * 
 * F - Fast: Tests should run quickly
 * I - Independent: Tests should not depend on each other
 * R - Repeatable: Same results every time
 * S - Self-Validating: Clear pass/fail result
 * T - Timely: Written alongside or before code
 */

// GOOD: Fast and Independent
test('should calculate circle area', () => {
  const radius = 5;
  const area = Math.PI * radius * radius;
  expect(calculateArea(radius)).toBeCloseTo(area);
});

// BAD: Slow and Dependent
let globalCounter = 0;
test('should increment counter', () => {
  globalCounter++;  // Depends on previous test state
  expect(globalCounter).toBe(1);  // Will fail if run out of order
});
```

### AAA Pattern (Arrange-Act-Assert)

```javascript
/*
 * AAA Pattern: Standard structure for test cases
 * 
 * 1. Arrange: Set up test data and conditions
 * 2. Act: Execute the function being tested
 * 3. Assert: Verify the result
 */

// Example: Testing a User Service
class UserService {
  constructor(database) {
    this.database = database;
  }

  async createUser(userData) {
    // Validate user data
    if (!userData.email || !userData.name) {
      throw new Error('Email and name are required');
    }
    
    // Check if user already exists
    const existing = await this.database.findByEmail(userData.email);
    if (existing) {
      throw new Error('User already exists');
    }
    
    // Create and return user
    return await this.database.create(userData);
  }
}

// Test using AAA Pattern
describe('UserService.createUser', () => {
  test('should create a new user with valid data', async () => {
    // ARRANGE: Set up test data and dependencies
    const mockDatabase = {
      findByEmail: jest.fn().mockResolvedValue(null),
      create: jest.fn().mockResolvedValue({ id: 1, email: 'test@example.com' })
    };
    const userService = new UserService(mockDatabase);
    const userData = { email: 'test@example.com', name: 'Test User' };
    
    // ACT: Execute the function
    const result = await userService.createUser(userData);
    
    // ASSERT: Verify the results
    expect(result).toHaveProperty('id');
    expect(result.email).toBe('test@example.com');
    expect(mockDatabase.create).toHaveBeenCalledWith(userData);
  });
});
```

### Test Coverage Concepts

```javascript
/*
 * Code Coverage Types:
 * 
 * 1. Statement Coverage: % of statements executed
 * 2. Branch Coverage: % of conditional branches tested
 * 3. Function Coverage: % of functions called
 * 4. Line Coverage: % of lines executed
 */

// Example: Function with branches
function getUserDiscount(user) {
  // Branch 1: Check if user exists
  if (!user) {
    return 0;
  }
  
  // Branch 2: Check premium status
  if (user.isPremium) {
    // Branch 3: Check loyalty level
    if (user.loyaltyYears > 5) {
      return 0.25;  // 25% discount for long-term premium
    }
    return 0.15;    // 15% discount for premium
  }
  
  // Branch 4: Regular user
  return 0.05;      // 5% discount for regular users
}

// Tests for 100% Branch Coverage
describe('getUserDiscount', () => {
  test('should return 0 for null user', () => {
    expect(getUserDiscount(null)).toBe(0);  // Tests Branch 1
  });
  
  test('should return 0.25 for long-term premium', () => {
    const user = { isPremium: true, loyaltyYears: 6 };
    expect(getUserDiscount(user)).toBe(0.25);  // Tests Branches 2 & 3
  });
  
  test('should return 0.15 for premium user', () => {
    const user = { isPremium: true, loyaltyYears: 2 };
    expect(getUserDiscount(user)).toBe(0.15);  // Tests Branch 2
  });
  
  test('should return 0.05 for regular user', () => {
    const user = { isPremium: false, loyaltyYears: 1 };
    expect(getUserDiscount(user)).toBe(0.05);  // Tests Branch 4
  });
});
```

---

## Writing Unit Tests using Jest Framework

### Jest Setup and Configuration

```bash
# Install Jest
npm install --save-dev jest

# Install TypeScript support (optional)
npm install --save-dev @types/jest ts-jest

# Install Babel support (for ES6+ modules)
npm install --save-dev babel-jest @babel/core @babel/preset-env
```

```javascript
// jest.config.js - Jest Configuration File
module.exports = {
  // Test environment (node or jsdom for browser simulation)
  testEnvironment: 'node',
  
  // Coverage settings
  collectCoverage: true,
  coverageDirectory: 'coverage',
  coverageReporters: ['text', 'lcov', 'html'],
  
  // Coverage thresholds (fail if below these percentages)
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    }
  },
  
  // Test file patterns
  testMatch: [
    '**/__tests__/**/*.js',
    '**/?(*.)+(spec|test).js'
  ],
  
  // Setup files to run before tests
  setupFilesAfterEnv: ['<rootDir>/jest.setup.js'],
  
  // Module path aliases
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/src/$1'
  },
  
  // Verbose output
  verbose: true
};
```

```json
// package.json - Test Scripts
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "test:verbose": "jest --verbose"
  }
}
```

### Jest Matchers and Assertions

```javascript
/*
 * Jest provides various matchers for different assertion types
 * Matchers are used with expect() to verify test results
 */

describe('Jest Matchers Examples', () => {
  
  // Equality Matchers
  test('equality matchers', () => {
    expect(2 + 2).toBe(4);                    // Strict equality (===)
    expect({ name: 'John' }).toEqual({ name: 'John' });  // Deep equality
    expect([1, 2, 3]).toEqual([1, 2, 3]);     // Array equality
  });
  
  // Truthiness Matchers
  test('truthiness matchers', () => {
    expect(null).toBeNull();                  // Checks for null
    expect(undefined).toBeUndefined();        // Checks for undefined
    expect(true).toBeDefined();               // Checks if defined
    expect(1).toBeTruthy();                   // Checks truthy value
    expect(0).toBeFalsy();                    // Checks falsy value
  });
  
  // Number Matchers
  test('number matchers', () => {
    expect(10).toBeGreaterThan(5);            // 10 > 5
    expect(5).toBeLessThan(10);               // 5 < 10
    expect(10).toBeGreaterThanOrEqual(10);    // 10 >= 10
    expect(5).toBeLessThanOrEqual(5);         // 5 <= 5
    expect(0.1 + 0.2).toBeCloseTo(0.3);       // Floating point comparison
  });
  
  // String Matchers
  test('string matchers', () => {
    expect('Hello World').toMatch(/World/);   // Regex match
    expect('test@example.com').toMatch(/^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$/);
    expect('JavaScript').toContain('Script');  // Substring check
  });
  
  // Array and Iterable Matchers
  test('array matchers', () => {
    const numbers = [1, 2, 3, 4, 5];
    expect(numbers).toContain(3);             // Array contains element
    expect(numbers).toHaveLength(5);          // Array length
    expect(numbers).toEqual(expect.arrayContaining([2, 4]));  // Contains subset
  });
  
  // Object Matchers
  test('object matchers', () => {
    const user = { id: 1, name: 'John', email: 'john@example.com' };
    expect(user).toHaveProperty('name');      // Has property
    expect(user).toHaveProperty('name', 'John');  // Property with value
    expect(user).toMatchObject({ name: 'John' });  // Partial match
  });
  
  // Exception Matchers
  test('exception matchers', () => {
    const throwError = () => { throw new Error('Oops!'); };
    expect(throwError).toThrow();             // Throws any error
    expect(throwError).toThrow('Oops!');      // Throws specific message
    expect(throwError).toThrow(Error);        // Throws specific error type
  });
});
```

### Testing Functions and Classes

```javascript
/*
 * Example: Testing a Shopping Cart Class
 * Demonstrates comprehensive unit testing of class methods
 */

// shopping-cart.js - Implementation
class ShoppingCart {
  constructor() {
    this.items = [];
    this.discountCode = null;
  }
  
  addItem(product, quantity = 1) {
    // Validate product
    if (!product || !product.id || !product.price) {
      throw new Error('Invalid product');
    }
    
    // Validate quantity
    if (quantity < 1) {
      throw new Error('Quantity must be at least 1');
    }
    
    // Check if item already exists
    const existingItem = this.items.find(item => item.product.id === product.id);
    
    if (existingItem) {
      existingItem.quantity += quantity;
    } else {
      this.items.push({ product, quantity });
    }
    
    return this.items.length;
  }
  
  removeItem(productId) {
    const index = this.items.findIndex(item => item.product.id === productId);
    
    if (index === -1) {
      throw new Error('Product not found in cart');
    }
    
    this.items.splice(index, 1);
    return this.items.length;
  }
  
  updateQuantity(productId, quantity) {
    if (quantity < 1) {
      throw new Error('Quantity must be at least 1');
    }
    
    const item = this.items.find(item => item.product.id === productId);
    
    if (!item) {
      throw new Error('Product not found in cart');
    }
    
    item.quantity = quantity;
  }
  
  applyDiscount(code) {
    const validCodes = {
      'SAVE10': 0.10,
      'SAVE20': 0.20,
      'SAVE30': 0.30
    };
    
    if (!validCodes[code]) {
      throw new Error('Invalid discount code');
    }
    
    this.discountCode = code;
    return validCodes[code];
  }
  
  getTotal() {
    const subtotal = this.items.reduce((total, item) => {
      return total + (item.product.price * item.quantity);
    }, 0);
    
    if (this.discountCode) {
      const validCodes = {
        'SAVE10': 0.10,
        'SAVE20': 0.20,
        'SAVE30': 0.30
      };
      const discount = validCodes[this.discountCode] || 0;
      return subtotal * (1 - discount);
    }
    
    return subtotal;
  }
  
  clear() {
    this.items = [];
    this.discountCode = null;
  }
  
  getItemCount() {
    return this.items.reduce((count, item) => count + item.quantity, 0);
  }
}

// shopping-cart.test.js - Tests
describe('ShoppingCart', () => {
  let cart;
  let sampleProduct;
  
  // Setup: Run before each test
  beforeEach(() => {
    cart = new ShoppingCart();
    sampleProduct = { id: 1, name: 'Widget', price: 10.00 };
  });
  
  // Teardown: Run after each test (optional)
  afterEach(() => {
    cart.clear();
  });
  
  describe('addItem', () => {
    test('should add item to empty cart', () => {
      const count = cart.addItem(sampleProduct);
      
      expect(count).toBe(1);
      expect(cart.items).toHaveLength(1);
      expect(cart.items[0]).toEqual({
        product: sampleProduct,
        quantity: 1
      });
    });
    
    test('should add multiple quantities', () => {
      cart.addItem(sampleProduct, 3);
      
      expect(cart.items[0].quantity).toBe(3);
    });
    
    test('should increment quantity for existing item', () => {
      cart.addItem(sampleProduct, 2);
      cart.addItem(sampleProduct, 3);
      
      expect(cart.items).toHaveLength(1);
      expect(cart.items[0].quantity).toBe(5);
    });
    
    test('should throw error for invalid product', () => {
      expect(() => cart.addItem(null)).toThrow('Invalid product');
      expect(() => cart.addItem({})).toThrow('Invalid product');
      expect(() => cart.addItem({ id: 1 })).toThrow('Invalid product');
    });
    
    test('should throw error for invalid quantity', () => {
      expect(() => cart.addItem(sampleProduct, 0)).toThrow('Quantity must be at least 1');
      expect(() => cart.addItem(sampleProduct, -1)).toThrow('Quantity must be at least 1');
    });
  });
  
  describe('removeItem', () => {
    beforeEach(() => {
      cart.addItem(sampleProduct);
    });
    
    test('should remove item from cart', () => {
      const count = cart.removeItem(sampleProduct.id);
      
      expect(count).toBe(0);
      expect(cart.items).toHaveLength(0);
    });
    
    test('should throw error for non-existent item', () => {
      expect(() => cart.removeItem(999)).toThrow('Product not found in cart');
    });
  });
  
  describe('updateQuantity', () => {
    beforeEach(() => {
      cart.addItem(sampleProduct, 2);
    });
    
    test('should update item quantity', () => {
      cart.updateQuantity(sampleProduct.id, 5);
      
      expect(cart.items[0].quantity).toBe(5);
    });
    
    test('should throw error for invalid quantity', () => {
      expect(() => cart.updateQuantity(sampleProduct.id, 0))
        .toThrow('Quantity must be at least 1');
    });
    
    test('should throw error for non-existent item', () => {
      expect(() => cart.updateQuantity(999, 5))
        .toThrow('Product not found in cart');
    });
  });
  
  describe('applyDiscount', () => {
    test('should apply valid discount code', () => {
      const discount = cart.applyDiscount('SAVE10');
      
      expect(discount).toBe(0.10);
      expect(cart.discountCode).toBe('SAVE10');
    });
    
    test('should throw error for invalid code', () => {
      expect(() => cart.applyDiscount('INVALID'))
        .toThrow('Invalid discount code');
    });
  });
  
  describe('getTotal', () => {
    test('should calculate total for single item', () => {
      cart.addItem(sampleProduct, 2);
      
      expect(cart.getTotal()).toBe(20.00);
    });
    
    test('should calculate total for multiple items', () => {
      cart.addItem(sampleProduct, 2);
      cart.addItem({ id: 2, name: 'Gadget', price: 15.00 }, 1);
      
      expect(cart.getTotal()).toBe(35.00);
    });
    
    test('should apply discount to total', () => {
      cart.addItem(sampleProduct, 2);  // $20
      cart.applyDiscount('SAVE20');    // 20% off
      
      expect(cart.getTotal()).toBe(16.00);  // $20 * 0.80
    });
    
    test('should return 0 for empty cart', () => {
      expect(cart.getTotal()).toBe(0);
    });
  });
  
  describe('getItemCount', () => {
    test('should return total item count', () => {
      cart.addItem(sampleProduct, 2);
      cart.addItem({ id: 2, name: 'Gadget', price: 15.00 }, 3);
      
      expect(cart.getItemCount()).toBe(5);
    });
    
    test('should return 0 for empty cart', () => {
      expect(cart.getItemCount()).toBe(0);
    });
  });
});
```

### Test Organization and Structure

```javascript
/*
 * Best practices for organizing tests:
 * 
 * 1. Use describe blocks to group related tests
 * 2. Use nested describe blocks for methods/features
 * 3. Use clear, descriptive test names
 * 4. Follow naming convention: "should [expected behavior] when [condition]"
 */

describe('UserAuthentication', () => {
  describe('login', () => {
    describe('with valid credentials', () => {
      test('should return user object with token', async () => {
        // Test implementation
      });
      
      test('should set authentication cookie', async () => {
        // Test implementation
      });
    });
    
    describe('with invalid credentials', () => {
      test('should throw authentication error', async () => {
        // Test implementation
      });
      
      test('should not set authentication cookie', async () => {
        // Test implementation
      });
    });
    
    describe('with missing credentials', () => {
      test('should throw validation error', async () => {
        // Test implementation
      });
    });
  });
  
  describe('logout', () => {
    test('should clear authentication token', async () => {
      // Test implementation
    });
    
    test('should remove authentication cookie', async () => {
      // Test implementation
    });
  });
});
```

---

## Test Driven Development (TDD) in Node.js

### TDD Cycle: Red-Green-Refactor

```javascript
/*
 * TDD Process:
 * 
 * 1. RED: Write a failing test first
 * 2. GREEN: Write minimal code to make test pass
 * 3. REFACTOR: Improve code while keeping tests green
 * 
 * Benefits:
 * - Better design through testing first
 * - Higher test coverage
 * - Confidence in refactoring
 * - Living documentation
 */

// Example: Building a Password Validator using TDD

// STEP 1: RED - Write failing test
describe('PasswordValidator', () => {
  test('should reject password shorter than 8 characters', () => {
    const validator = new PasswordValidator();
    expect(validator.validate('abc123')).toBe(false);
  });
});

// Run test - it fails (RED) because PasswordValidator doesn't exist

// STEP 2: GREEN - Write minimal code to pass
class PasswordValidator {
  validate(password) {
    return password.length >= 8;
  }
}

// Run test - it passes (GREEN)

// STEP 3: Add more tests (RED)
test('should reject password without uppercase letter', () => {
  const validator = new PasswordValidator();
  expect(validator.validate('password123')).toBe(false);
});

// STEP 4: Update code (GREEN)
class PasswordValidator {
  validate(password) {
    if (password.length < 8) return false;
    if (!/[A-Z]/.test(password)) return false;
    return true;
  }
}

// STEP 5: Continue cycle for all requirements
test('should reject password without lowercase letter', () => {
  const validator = new PasswordValidator();
  expect(validator.validate('PASSWORD123')).toBe(false);
});

test('should reject password without number', () => {
  const validator = new PasswordValidator();
  expect(validator.validate('Password')).toBe(false);
});

test('should accept valid password', () => {
  const validator = new PasswordValidator();
  expect(validator.validate('Password123')).toBe(true);
});

// Final implementation after all tests pass
class PasswordValidator {
  validate(password) {
    if (password.length < 8) return false;
    if (!/[A-Z]/.test(password)) return false;
    if (!/[a-z]/.test(password)) return false;
    if (!/[0-9]/.test(password)) return false;
    return true;
  }
}

// REFACTOR: Improve code readability
class PasswordValidator {
  constructor() {
    this.rules = [
      { test: (pwd) => pwd.length >= 8, message: 'Min 8 characters' },
      { test: (pwd) => /[A-Z]/.test(pwd), message: 'Needs uppercase' },
      { test: (pwd) => /[a-z]/.test(pwd), message: 'Needs lowercase' },
      { test: (pwd) => /[0-9]/.test(pwd), message: 'Needs number' }
    ];
  }
  
  validate(password) {
    return this.rules.every(rule => rule.test(password));
  }
  
  getErrors(password) {
    return this.rules
      .filter(rule => !rule.test(password))
      .map(rule => rule.message);
  }
}
```

### TDD Example: Building a Task Manager API

```javascript
/*
 * Real-world TDD Example: Task Manager API
 * Following TDD principles to build a complete feature
 */

// task-manager.test.js - Start with tests

describe('TaskManager', () => {
  let taskManager;
  
  beforeEach(() => {
    taskManager = new TaskManager();
  });
  
  // TEST 1: Create task
  describe('createTask', () => {
    test('should create task with title and description', () => {
      const task = taskManager.createTask({
        title: 'Buy groceries',
        description: 'Milk, eggs, bread'
      });
      
      expect(task).toHaveProperty('id');
      expect(task.title).toBe('Buy groceries');
      expect(task.description).toBe('Milk, eggs, bread');
      expect(task.completed).toBe(false);
      expect(task.createdAt).toBeInstanceOf(Date);
    });
    
    test('should throw error if title is missing', () => {
      expect(() => taskManager.createTask({ description: 'Test' }))
        .toThrow('Title is required');
    });
    
    test('should generate unique IDs for each task', () => {
      const task1 = taskManager.createTask({ title: 'Task 1' });
      const task2 = taskManager.createTask({ title: 'Task 2' });
      
      expect(task1.id).not.toBe(task2.id);
    });
  });
  
  // TEST 2: Get all tasks
  describe('getAllTasks', () => {
    test('should return empty array initially', () => {
      expect(taskManager.getAllTasks()).toEqual([]);
    });
    
    test('should return all created tasks', () => {
      taskManager.createTask({ title: 'Task 1' });
      taskManager.createTask({ title: 'Task 2' });
      
      const tasks = taskManager.getAllTasks();
      expect(tasks).toHaveLength(2);
    });
  });
  
  // TEST 3: Get task by ID
  describe('getTaskById', () => {
    test('should return task with matching ID', () => {
      const created = taskManager.createTask({ title: 'Test Task' });
      const found = taskManager.getTaskById(created.id);
      
      expect(found).toEqual(created);
    });
    
    test('should return null for non-existent ID', () => {
      expect(taskManager.getTaskById('invalid-id')).toBeNull();
    });
  });
  
  // TEST 4: Update task
  describe('updateTask', () => {
    test('should update task properties', () => {
      const task = taskManager.createTask({ title: 'Old Title' });
      
      const updated = taskManager.updateTask(task.id, {
        title: 'New Title',
        description: 'New Description'
      });
      
      expect(updated.title).toBe('New Title');
      expect(updated.description).toBe('New Description');
    });
    
    test('should throw error for non-existent task', () => {
      expect(() => taskManager.updateTask('invalid-id', { title: 'Test' }))
        .toThrow('Task not found');
    });
    
    test('should not update id or createdAt', () => {
      const task = taskManager.createTask({ title: 'Test' });
      const originalId = task.id;
      const originalDate = task.createdAt;
      
      taskManager.updateTask(task.id, {
        id: 'new-id',
        createdAt: new Date(),
        title: 'Updated'
      });
      
      const updated = taskManager.getTaskById(originalId);
      expect(updated.id).toBe(originalId);
      expect(updated.createdAt).toBe(originalDate);
    });
  });
  
  // TEST 5: Delete task
  describe('deleteTask', () => {
    test('should remove task from list', () => {
      const task = taskManager.createTask({ title: 'Test' });
      
      taskManager.deleteTask(task.id);
      
      expect(taskManager.getTaskById(task.id)).toBeNull();
      expect(taskManager.getAllTasks()).toHaveLength(0);
    });
    
    test('should throw error for non-existent task', () => {
      expect(() => taskManager.deleteTask('invalid-id'))
        .toThrow('Task not found');
    });
  });
  
  // TEST 6: Mark as complete
  describe('markAsComplete', () => {
    test('should set completed to true', () => {
      const task = taskManager.createTask({ title: 'Test' });
      
      const completed = taskManager.markAsComplete(task.id);
      
      expect(completed.completed).toBe(true);
      expect(completed.completedAt).toBeInstanceOf(Date);
    });
    
    test('should throw error for non-existent task', () => {
      expect(() => taskManager.markAsComplete('invalid-id'))
        .toThrow('Task not found');
    });
  });
  
  // TEST 7: Filter tasks
  describe('filterTasks', () => {
    beforeEach(() => {
      taskManager.createTask({ title: 'Task 1', tags: ['work'] });
      taskManager.createTask({ title: 'Task 2', tags: ['personal'] });
      const task3 = taskManager.createTask({ title: 'Task 3', tags: ['work'] });
      taskManager.markAsComplete(task3.id);
    });
    
    test('should filter by completed status', () => {
      const completed = taskManager.filterTasks({ completed: true });
      expect(completed).toHaveLength(1);
      expect(completed[0].title).toBe('Task 3');
    });
    
    test('should filter by tags', () => {
      const workTasks = taskManager.filterTasks({ tags: 'work' });
      expect(workTasks).toHaveLength(2);
    });
  });
});

// task-manager.js - Implementation (written after tests)

const { v4: uuidv4 } = require('uuid');

class TaskManager {
  constructor() {
    this.tasks = [];
  }
  
  createTask({ title, description = '', tags = [] }) {
    if (!title) {
      throw new Error('Title is required');
    }
    
    const task = {
      id: uuidv4(),
      title,
      description,
      tags,
      completed: false,
      createdAt: new Date(),
      completedAt: null
    };
    
    this.tasks.push(task);
    return { ...task };  // Return copy to prevent mutation
  }
  
  getAllTasks() {
    return this.tasks.map(task => ({ ...task }));
  }
  
  getTaskById(id) {
    const task = this.tasks.find(t => t.id === id);
    return task ? { ...task } : null;
  }
  
  updateTask(id, updates) {
    const taskIndex = this.tasks.findIndex(t => t.id === id);
    
    if (taskIndex === -1) {
      throw new Error('Task not found');
    }
    
    // Prevent updating protected fields
    const { id: _, createdAt: __, ...allowedUpdates } = updates;
    
    this.tasks[taskIndex] = {
      ...this.tasks[taskIndex],
      ...allowedUpdates
    };
    
    return { ...this.tasks[taskIndex] };
  }
  
  deleteTask(id) {
    const taskIndex = this.tasks.findIndex(t => t.id === id);
    
    if (taskIndex === -1) {
      throw new Error('Task not found');
    }
    
    this.tasks.splice(taskIndex, 1);
  }
  
  markAsComplete(id) {
    const task = this.tasks.find(t => t.id === id);
    
    if (!task) {
      throw new Error('Task not found');
    }
    
    task.completed = true;
    task.completedAt = new Date();
    
    return { ...task };
  }
  
  filterTasks({ completed, tags } = {}) {
    return this.tasks.filter(task => {
      if (completed !== undefined && task.completed !== completed) {
        return false;
      }
      
      if (tags && !task.tags.includes(tags)) {
        return false;
      }
      
      return true;
    }).map(task => ({ ...task }));
  }
}

module.exports = TaskManager;
```

### TDD Best Practices

```javascript
/*
 * TDD Best Practices and Common Patterns
 */

// 1. Test one thing at a time
// GOOD
test('should validate email format', () => {
  expect(isValidEmail('test@example.com')).toBe(true);
});

test('should reject invalid email format', () => {
  expect(isValidEmail('invalid-email')).toBe(false);
});

// BAD - Testing multiple things
test('should validate email', () => {
  expect(isValidEmail('test@example.com')).toBe(true);
  expect(isValidEmail('invalid')).toBe(false);
  expect(isValidEmail('')).toBe(false);
  expect(isValidEmail(null)).toBe(false);
});

// 2. Use descriptive test names
// GOOD
test('should throw error when password is less than 8 characters', () => {});

// BAD
test('password test', () => {});

// 3. Test behavior, not implementation
// GOOD - Tests the outcome
test('should return sorted array', () => {
  const result = sortArray([3, 1, 2]);
  expect(result).toEqual([1, 2, 3]);
});

// BAD - Tests implementation details
test('should call quicksort algorithm', () => {
  const spy = jest.spyOn(sorter, 'quicksort');
  sortArray([3, 1, 2]);
  expect(spy).toHaveBeenCalled();
});

// 4. Keep tests independent
// GOOD - Each test sets up its own data
describe('User Service', () => {
  test('should create user', () => {
    const user = createUser({ name: 'John' });
    expect(user.name).toBe('John');
  });
  
  test('should update user', () => {
    const user = createUser({ name: 'John' });
    const updated = updateUser(user.id, { name: 'Jane' });
    expect(updated.name).toBe('Jane');
  });
});

// BAD - Tests depend on each other
let sharedUser;

test('should create user', () => {
  sharedUser = createUser({ name: 'John' });
  expect(sharedUser.name).toBe('John');
});

test('should update user', () => {
  // Fails if previous test didn't run
  const updated = updateUser(sharedUser.id, { name: 'Jane' });
  expect(updated.name).toBe('Jane');
});
```

---

## Mocking Dependencies and API Calls using Libraries like Sinon

### Introduction to Mocking

```javascript
/*
 * Mocking Concepts:
 * 
 * - Mock: Fake object that simulates real behavior
 * - Stub: Replaces function with predefined response
 * - Spy: Records information about function calls
 * 
 * Why Mock?
 * 1. Isolate unit under test
 * 2. Avoid external dependencies (DB, APIs)
 * 3. Control test scenarios
 * 4. Speed up tests
 */

// Install dependencies
// npm install --save-dev sinon
// npm install --save-dev @types/sinon  // For TypeScript
```

### Jest Built-in Mocking

```javascript
/*
 * Jest provides built-in mocking capabilities
 * Most common approach in modern Node.js testing
 */

// Example: Mocking a Database Service
class Database {
  async findUserById(id) {
    // Real implementation would query database
    throw new Error('Not implemented');
  }
  
  async saveUser(user) {
    // Real implementation would save to database
    throw new Error('Not implemented');
  }
}

class UserService {
  constructor(database) {
    this.database = database;
  }
  
  async getUserProfile(userId) {
    const user = await this.database.findUserById(userId);
    
    if (!user) {
      throw new Error('User not found');
    }
    
    return {
      id: user.id,
      name: user.name,
      email: user.email,
      // Don't expose password
    };
  }
  
  async updateUserEmail(userId, newEmail) {
    const user = await this.database.findUserById(userId);
    
    if (!user) {
      throw new Error('User not found');
    }
    
    user.email = newEmail;
    await this.database.saveUser(user);
    
    return user;
  }
}

// user-service.test.js - Using Jest Mocks
describe('UserService', () => {
  let userService;
  let mockDatabase;
  
  beforeEach(() => {
    // Create mock database with jest.fn()
    mockDatabase = {
      findUserById: jest.fn(),
      saveUser: jest.fn()
    };
    
    userService = new UserService(mockDatabase);
  });
  
  describe('getUserProfile', () => {
    test('should return user profile', async () => {
      // Arrange: Set up mock return value
      const mockUser = {
        id: 1,
        name: 'John Doe',
        email: 'john@example.com',
        password: 'hashed_password'
      };
      mockDatabase.findUserById.mockResolvedValue(mockUser);
      
      // Act: Call the method
      const profile = await userService.getUserProfile(1);
      
      // Assert: Verify results
      expect(profile).toEqual({
        id: 1,
        name: 'John Doe',
        email: 'john@example.com'
      });
      expect(profile).not.toHaveProperty('password');
      expect(mockDatabase.findUserById).toHaveBeenCalledWith(1);
    });
    
    test('should throw error if user not found', async () => {
      // Arrange: Mock returns null
      mockDatabase.findUserById.mockResolvedValue(null);
      
      // Act & Assert
      await expect(userService.getUserProfile(999))
        .rejects.toThrow('User not found');
    });
  });
  
  describe('updateUserEmail', () => {
    test('should update user email', async () => {
      // Arrange
      const mockUser = {
        id: 1,
        name: 'John Doe',
        email: 'old@example.com'
      };
      mockDatabase.findUserById.mockResolvedValue(mockUser);
      mockDatabase.saveUser.mockResolvedValue(undefined);
      
      // Act
      const updated = await userService.updateUserEmail(1, 'new@example.com');
      
      // Assert
      expect(updated.email).toBe('new@example.com');
      expect(mockDatabase.saveUser).toHaveBeenCalledWith(
        expect.objectContaining({ email: 'new@example.com' })
      );
    });
  });
});
```

### Mocking HTTP Requests with Jest

```javascript
/*
 * Mocking External API Calls
 * Using jest.mock() to mock entire modules
 */

// api-client.js - HTTP Client
const axios = require('axios');

class WeatherService {
  constructor(apiKey) {
    this.apiKey = apiKey;
    this.baseUrl = 'https://api.weather.com';
  }
  
  async getCurrentWeather(city) {
    try {
      const response = await axios.get(`${this.baseUrl}/current`, {
        params: {
          city,
          apiKey: this.apiKey
        }
      });
      
      return {
        temperature: response.data.temp,
        condition: response.data.condition,
        humidity: response.data.humidity
      };
    } catch (error) {
      if (error.response && error.response.status === 404) {
        throw new Error('City not found');
      }
      throw new Error('Weather service unavailable');
    }
  }
  
  async getForecast(city, days = 7) {
    const response = await axios.get(`${this.baseUrl}/forecast`, {
      params: {
        city,
        days,
        apiKey: this.apiKey
      }
    });
    
    return response.data.forecast.map(day => ({
      date: day.date,
      high: day.high_temp,
      low: day.low_temp,
      condition: day.condition
    }));
  }
}

// weather-service.test.js - Mocking axios
jest.mock('axios');  // Mock the entire axios module
const axios = require('axios');

describe('WeatherService', () => {
  let weatherService;
  
  beforeEach(() => {
    weatherService = new WeatherService('test-api-key');
    jest.clearAllMocks();  // Clear mock call history
  });
  
  describe('getCurrentWeather', () => {
    test('should return current weather data', async () => {
      // Arrange: Mock axios.get response
      const mockResponse = {
        data: {
          temp: 72,
          condition: 'Sunny',
          humidity: 45
        }
      };
      axios.get.mockResolvedValue(mockResponse);
      
      // Act
      const weather = await weatherService.getCurrentWeather('San Francisco');
      
      // Assert
      expect(weather).toEqual({
        temperature: 72,
        condition: 'Sunny',
        humidity: 45
      });
      
      expect(axios.get).toHaveBeenCalledWith(
        'https://api.weather.com/current',
        {
          params: {
            city: 'San Francisco',
            apiKey: 'test-api-key'
          }
        }
      );
    });
    
    test('should throw error for invalid city', async () => {
      // Arrange: Mock 404 error
      const mockError = {
        response: { status: 404 }
      };
      axios.get.mockRejectedValue(mockError);
      
      // Act & Assert
      await expect(weatherService.getCurrentWeather('InvalidCity'))
        .rejects.toThrow('City not found');
    });
    
    test('should throw error for service unavailable', async () => {
      // Arrange: Mock network error
      axios.get.mockRejectedValue(new Error('Network error'));
      
      // Act & Assert
      await expect(weatherService.getCurrentWeather('San Francisco'))
        .rejects.toThrow('Weather service unavailable');
    });
  });
  
  describe('getForecast', () => {
    test('should return forecast data', async () => {
      // Arrange
      const mockResponse = {
        data: {
          forecast: [
            { date: '2024-01-01', high_temp: 75, low_temp: 60, condition: 'Clear' },
            { date: '2024-01-02', high_temp: 73, low_temp: 58, condition: 'Cloudy' }
          ]
        }
      };
      axios.get.mockResolvedValue(mockResponse);
      
      // Act
      const forecast = await weatherService.getForecast('San Francisco', 2);
      
      // Assert
      expect(forecast).toHaveLength(2);
      expect(forecast[0]).toEqual({
        date: '2024-01-01',
        high: 75,
        low: 60,
        condition: 'Clear'
      });
    });
  });
});
```

### Using Sinon for Advanced Mocking

```javascript
/*
 * Sinon provides additional mocking capabilities:
 * - Stubs: Replace functions with custom behavior
 * - Spies: Monitor function calls without changing behavior
 * - Fake timers: Control time in tests
 */

const sinon = require('sinon');

// Example: File System Operations
const fs = require('fs').promises;

class FileLogger {
  constructor(filePath) {
    this.filePath = filePath;
  }
  
  async log(message) {
    const timestamp = new Date().toISOString();
    const logEntry = `[${timestamp}] ${message}\n`;
    
    try {
      await fs.appendFile(this.filePath, logEntry);
      return true;
    } catch (error) {
      console.error('Failed to write log:', error);
      return false;
    }
  }
  
  async getLogs() {
    try {
      const content = await fs.readFile(this.filePath, 'utf-8');
      return content.split('\n').filter(line => line.trim());
    } catch (error) {
      if (error.code === 'ENOENT') {
        return [];
      }
      throw error;
    }
  }
}

// file-logger.test.js - Using Sinon
describe('FileLogger with Sinon', () => {
  let fileLogger;
  let fsStub;
  
  beforeEach(() => {
    fileLogger = new FileLogger('/var/log/app.log');
  });
  
  afterEach(() => {
    // Restore all stubs
    sinon.restore();
  });
  
  describe('log', () => {
    test('should write log entry to file', async () => {
      // Create stub for fs.appendFile
      fsStub = sinon.stub(fs, 'appendFile').resolves();
      
      const result = await fileLogger.log('Test message');
      
      expect(result).toBe(true);
      expect(fsStub.calledOnce).toBe(true);
      expect(fsStub.firstCall.args[0]).toBe('/var/log/app.log');
      expect(fsStub.firstCall.args[1]).toMatch(/\[.*\] Test message\n/);
    });
    
    test('should return false on write error', async () => {
      // Stub throws error
      fsStub = sinon.stub(fs, 'appendFile').rejects(new Error('Disk full'));
      
      const result = await fileLogger.log('Test message');
      
      expect(result).toBe(false);
    });
  });
  
  describe('getLogs', () => {
    test('should return log entries', async () => {
      const mockContent = '[2024-01-01] Entry 1\n[2024-01-02] Entry 2\n';
      fsStub = sinon.stub(fs, 'readFile').resolves(mockContent);
      
      const logs = await fileLogger.getLogs();
      
      expect(logs).toHaveLength(2);
      expect(logs[0]).toBe('[2024-01-01] Entry 1');
    });
    
    test('should return empty array for non-existent file', async () => {
      const error = new Error('File not found');
      error.code = 'ENOENT';
      fsStub = sinon.stub(fs, 'readFile').rejects(error);
      
      const logs = await fileLogger.getLogs();
      
      expect(logs).toEqual([]);
    });
  });
});

// Example: Spying on console.log
describe('Spy Examples', () => {
  test('should log error message', async () => {
    // Create spy for console.error
    const errorSpy = sinon.spy(console, 'error');
    
    const fsStub = sinon.stub(fs, 'appendFile')
      .rejects(new Error('Write failed'));
    
    const fileLogger = new FileLogger('/var/log/app.log');
    await fileLogger.log('Test');
    
    // Verify console.error was called
    expect(errorSpy.calledOnce).toBe(true);
    expect(errorSpy.firstCall.args[0]).toBe('Failed to write log:');
    
    errorSpy.restore();
    fsStub.restore();
  });
});

// Example: Fake Timers
describe('Fake Timers', () => {
  let clock;
  
  beforeEach(() => {
    // Create fake timer
    clock = sinon.useFakeTimers(new Date('2024-01-01T00:00:00Z'));
  });
  
  afterEach(() => {
    clock.restore();
  });
  
  test('should use consistent timestamp', async () => {
    const fsStub = sinon.stub(fs, 'appendFile').resolves();
    const fileLogger = new FileLogger('/var/log/app.log');
    
    await fileLogger.log('Message 1');
    clock.tick(1000);  // Advance time by 1 second
    await fileLogger.log('Message 2');
    
    const call1 = fsStub.firstCall.args[1];
    const call2 = fsStub.secondCall.args[1];
    
    expect(call1).toContain('[2024-01-01T00:00:00');
    expect(call2).toContain('[2024-01-01T00:00:01');
    
    fsStub.restore();
  });
});
```

### Mock Patterns and Best Practices

```javascript
/*
 * Common Mocking Patterns
 */

// Pattern 1: Partial Mocking
// Mock only specific methods of an object
const userRepository = {
  findById: jest.fn(),
  findAll: jest.fn(),
  save: jest.fn()
};

// Use real implementation for some methods
userRepository.findAll.mockImplementation(() => {
  return database.users;  // Real database call
});

// Pattern 2: Mock Implementation with Different Responses
test('should handle multiple mock responses', async () => {
  const mockFetch = jest.fn();
  
  // First call returns success
  mockFetch.mockResolvedValueOnce({ status: 200, data: 'Success' });
  
  // Second call returns error
  mockFetch.mockResolvedValueOnce({ status: 500, data: 'Error' });
  
  const result1 = await mockFetch();
  const result2 = await mockFetch();
  
  expect(result1.status).toBe(200);
  expect(result2.status).toBe(500);
});

// Pattern 3: Mock Factory Function
function createMockDatabase() {
  return {
    findUserById: jest.fn(),
    saveUser: jest.fn(),
    deleteUser: jest.fn(),
    // Reset all mocks
    resetMocks() {
      this.findUserById.mockReset();
      this.saveUser.mockReset();
      this.deleteUser.mockReset();
    }
  };
}

describe('User Service Tests', () => {
  let mockDb;
  
  beforeEach(() => {
    mockDb = createMockDatabase();
  });
  
  test('test 1', () => {
    mockDb.findUserById.mockResolvedValue({ id: 1 });
    // ... test code
  });
});

// Pattern 4: Verify Mock Call Arguments
test('should call API with correct parameters', async () => {
  const mockApi = jest.fn();
  
  await fetchUserData(mockApi, 123);
  
  // Verify called with exact arguments
  expect(mockApi).toHaveBeenCalledWith(123);
  
  // Verify called with object containing properties
  expect(mockApi).toHaveBeenCalledWith(
    expect.objectContaining({
      userId: 123,
      includeProfile: true
    })
  );
  
  // Verify called with matching pattern
  expect(mockApi).toHaveBeenCalledWith(
    expect.stringMatching(/^user-/)
  );
});

// Pattern 5: Mock Class Constructor
jest.mock('./database');
const Database = require('./database');

test('should create database instance', () => {
  const mockInstance = {
    connect: jest.fn(),
    query: jest.fn()
  };
  
  Database.mockImplementation(() => mockInstance);
  
  const service = new UserService();
  
  expect(Database).toHaveBeenCalled();
  expect(mockInstance.connect).toHaveBeenCalled();
});
```

---

## Integration Testing of REST APIs using Supertest

### Introduction to Supertest

```javascript
/*
 * Supertest: HTTP assertion library for testing Node.js HTTP servers
 * 
 * Features:
 * - Test Express/Koa/etc. applications
 * - Make HTTP requests in tests
 * - Assert responses
 * - No need to start actual server
 */

// Install dependencies
// npm install --save-dev supertest
// npm install express  // Web framework
```

### Setting Up Express API for Testing

```javascript
// app.js - Express Application
const express = require('express');
const app = express();

app.use(express.json());

// In-memory data store for testing
let users = [
  { id: 1, name: 'John Doe', email: 'john@example.com' },
  { id: 2, name: 'Jane Smith', email: 'jane@example.com' }
];
let nextId = 3;

// GET /api/users - Get all users
app.get('/api/users', (req, res) => {
  res.json(users);
});

// GET /api/users/:id - Get user by ID
app.get('/api/users/:id', (req, res) => {
  const user = users.find(u => u.id === parseInt(req.params.id));
  
  if (!user) {
    return res.status(404).json({ error: 'User not found' });
  }
  
  res.json(user);
});

// POST /api/users - Create new user
app.post('/api/users', (req, res) => {
  const { name, email } = req.body;
  
  // Validation
  if (!name || !email) {
    return res.status(400).json({ error: 'Name and email are required' });
  }
  
  // Check duplicate email
  if (users.some(u => u.email === email)) {
    return res.status(409).json({ error: 'Email already exists' });
  }
  
  const user = {
    id: nextId++,
    name,
    email
  };
  
  users.push(user);
  res.status(201).json(user);
});

// PUT /api/users/:id - Update user
app.put('/api/users/:id', (req, res) => {
  const userId = parseInt(req.params.id);
  const userIndex = users.findIndex(u => u.id === userId);
  
  if (userIndex === -1) {
    return res.status(404).json({ error: 'User not found' });
  }
  
  const { name, email } = req.body;
  
  if (!name || !email) {
    return res.status(400).json({ error: 'Name and email are required' });
  }
  
  users[userIndex] = {
    ...users[userIndex],
    name,
    email
  };
  
  res.json(users[userIndex]);
});

// DELETE /api/users/:id - Delete user
app.delete('/api/users/:id', (req, res) => {
  const userId = parseInt(req.params.id);
  const userIndex = users.findIndex(u => u.id === userId);
  
  if (userIndex === -1) {
    return res.status(404).json({ error: 'User not found' });
  }
  
  users.splice(userIndex, 1);
  res.status(204).send();
});

// Error handling middleware
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ error: 'Internal server error' });
});

module.exports = app;
```

### Writing Integration Tests with Supertest

```javascript
// app.test.js - Integration Tests
const request = require('supertest');
const app = require('./app');

describe('User API Integration Tests', () => {
  
  // Reset data before each test
  beforeEach(() => {
    // This would reset the database in a real application
    // For this example, we're using in-memory data
  });
  
  describe('GET /api/users', () => {
    test('should return all users', async () => {
      const response = await request(app)
        .get('/api/users')
        .expect(200)  // Assert status code
        .expect('Content-Type', /json/);  // Assert content type
      
      expect(response.body).toBeInstanceOf(Array);
      expect(response.body.length).toBeGreaterThan(0);
      expect(response.body[0]).toHaveProperty('id');
      expect(response.body[0]).toHaveProperty('name');
      expect(response.body[0]).toHaveProperty('email');
    });
  });
  
  describe('GET /api/users/:id', () => {
    test('should return user by ID', async () => {
      const response = await request(app)
        .get('/api/users/1')
        .expect(200);
      
      expect(response.body).toMatchObject({
        id: 1,
        name: expect.any(String),
        email: expect.any(String)
      });
    });
    
    test('should return 404 for non-existent user', async () => {
      const response = await request(app)
        .get('/api/users/9999')
        .expect(404);
      
      expect(response.body).toHaveProperty('error', 'User not found');
    });
    
    test('should return 404 for invalid ID format', async () => {
      await request(app)
        .get('/api/users/invalid')
        .expect(404);
    });
  });
  
  describe('POST /api/users', () => {
    test('should create new user', async () => {
      const newUser = {
        name: 'Alice Johnson',
        email: 'alice@example.com'
      };
      
      const response = await request(app)
        .post('/api/users')
        .send(newUser)
        .expect(201)
        .expect('Content-Type', /json/);
      
      expect(response.body).toMatchObject({
        id: expect.any(Number),
        name: 'Alice Johnson',
        email: 'alice@example.com'
      });
      
      // Verify user was actually created
      const getResponse = await request(app)
        .get(`/api/users/${response.body.id}`)
        .expect(200);
      
      expect(getResponse.body).toMatchObject(newUser);
    });
    
    test('should return 400 for missing name', async () => {
      const response = await request(app)
        .post('/api/users')
        .send({ email: 'test@example.com' })
        .expect(400);
      
      expect(response.body.error).toContain('required');
    });
    
    test('should return 400 for missing email', async () => {
      const response = await request(app)
        .post('/api/users')
        .send({ name: 'Test User' })
        .expect(400);
      
      expect(response.body.error).toContain('required');
    });
    
    test('should return 409 for duplicate email', async () => {
      const response = await request(app)
        .post('/api/users')
        .send({
          name: 'Duplicate User',
          email: 'john@example.com'  // Existing email
        })
        .expect(409);
      
      expect(response.body.error).toContain('already exists');
    });
  });
  
  describe('PUT /api/users/:id', () => {
    test('should update existing user', async () => {
      const updates = {
        name: 'John Updated',
        email: 'john.updated@example.com'
      };
      
      const response = await request(app)
        .put('/api/users/1')
        .send(updates)
        .expect(200);
      
      expect(response.body).toMatchObject({
        id: 1,
        ...updates
      });
    });
    
    test('should return 404 for non-existent user', async () => {
      await request(app)
        .put('/api/users/9999')
        .send({ name: 'Test', email: 'test@example.com' })
        .expect(404);
    });
    
    test('should return 400 for invalid data', async () => {
      await request(app)
        .put('/api/users/1')
        .send({ name: 'Test' })  // Missing email
        .expect(400);
    });
  });
  
  describe('DELETE /api/users/:id', () => {
    test('should delete existing user', async () => {
      await request(app)
        .delete('/api/users/1')
        .expect(204);
      
      // Verify user was deleted
      await request(app)
        .get('/api/users/1')
        .expect(404);
    });
    
    test('should return 404 for non-existent user', async () => {
      await request(app)
        .delete('/api/users/9999')
        .expect(404);
    });
  });
  
  describe('API Error Handling', () => {
    test('should handle JSON parse errors', async () => {
      await request(app)
        .post('/api/users')
        .set('Content-Type', 'application/json')
        .send('invalid json')
        .expect(400);
    });
  });
});
```

### Testing Authentication and Authorization

```javascript
/*
 * Testing Protected Routes with Authentication
 */

// auth-app.js - Express app with JWT authentication
const express = require('express');
const jwt = require('jsonwebtoken');
const app = express();

const SECRET_KEY = 'test-secret-key';

app.use(express.json());

// Authentication middleware
const authenticateToken = (req, res, next) => {
  const authHeader = req.headers['authorization'];
  const token = authHeader && authHeader.split(' ')[1];
  
  if (!token) {
    return res.status(401).json({ error: 'Access token required' });
  }
  
  jwt.verify(token, SECRET_KEY, (err, user) => {
    if (err) {
      return res.status(403).json({ error: 'Invalid or expired token' });
    }
    req.user = user;
    next();
  });
};

// POST /api/login - Login endpoint
app.post('/api/login', (req, res) => {
  const { username, password } = req.body;
  
  // Simple validation (in real app, check against database)
  if (username === 'admin' && password === 'password123') {
    const token = jwt.sign(
      { username, role: 'admin' },
      SECRET_KEY,
      { expiresIn: '1h' }
    );
    
    return res.json({ token });
  }
  
  res.status(401).json({ error: 'Invalid credentials' });
});

// GET /api/protected - Protected route
app.get('/api/protected', authenticateToken, (req, res) => {
  res.json({
    message: 'Access granted',
    user: req.user
  });
});

// GET /api/admin - Admin only route
app.get('/api/admin', authenticateToken, (req, res) => {
  if (req.user.role !== 'admin') {
    return res.status(403).json({ error: 'Admin access required' });
  }
  
  res.json({ message: 'Admin panel' });
});

module.exports = app;

// auth-app.test.js - Testing authentication
describe('Authentication API Tests', () => {
  let authToken;
  
  describe('POST /api/login', () => {
    test('should return token for valid credentials', async () => {
      const response = await request(app)
        .post('/api/login')
        .send({
          username: 'admin',
          password: 'password123'
        })
        .expect(200);
      
      expect(response.body).toHaveProperty('token');
      expect(typeof response.body.token).toBe('string');
      
      // Save token for subsequent tests
      authToken = response.body.token;
    });
    
    test('should return 401 for invalid credentials', async () => {
      const response = await request(app)
        .post('/api/login')
        .send({
          username: 'admin',
          password: 'wrongpassword'
        })
        .expect(401);
      
      expect(response.body.error).toBe('Invalid credentials');
    });
  });
  
  describe('GET /api/protected', () => {
    beforeEach(async () => {
      // Get fresh token before each test
      const response = await request(app)
        .post('/api/login')
        .send({ username: 'admin', password: 'password123' });
      authToken = response.body.token;
    });
    
    test('should allow access with valid token', async () => {
      const response = await request(app)
        .get('/api/protected')
        .set('Authorization', `Bearer ${authToken}`)
        .expect(200);
      
      expect(response.body.message).toBe('Access granted');
      expect(response.body.user).toHaveProperty('username', 'admin');
    });
    
    test('should deny access without token', async () => {
      const response = await request(app)
        .get('/api/protected')
        .expect(401);
      
      expect(response.body.error).toContain('token required');
    });
    
    test('should deny access with invalid token', async () => {
      await request(app)
        .get('/api/protected')
        .set('Authorization', 'Bearer invalid-token')
        .expect(403);
    });
  });
  
  describe('GET /api/admin', () => {
    test('should allow access for admin role', async () => {
      const response = await request(app)
        .post('/api/login')
        .send({ username: 'admin', password: 'password123' });
      
      await request(app)
        .get('/api/admin')
        .set('Authorization', `Bearer ${response.body.token}`)
        .expect(200);
    });
  });
});
```

### Testing Database Integration

```javascript
/*
 * Integration Tests with Real Database
 * Using Test Database for Integration Testing
 */

// database.js - Database connection
const { MongoClient } = require('mongodb');

class Database {
  constructor(url, dbName) {
    this.url = url;
    this.dbName = dbName;
    this.client = null;
    this.db = null;
  }
  
  async connect() {
    this.client = await MongoClient.connect(this.url);
    this.db = this.client.db(this.dbName);
  }
  
  async disconnect() {
    if (this.client) {
      await this.client.close();
    }
  }
  
  async insertUser(user) {
    const result = await this.db.collection('users').insertOne(user);
    return { id: result.insertedId, ...user };
  }
  
  async findUserById(id) {
    return await this.db.collection('users').findOne({ _id: id });
  }
  
  async findUserByEmail(email) {
    return await this.db.collection('users').findOne({ email });
  }
  
  async deleteAllUsers() {
    await this.db.collection('users').deleteMany({});
  }
}

module.exports = Database;

// user-service-db.test.js - Database Integration Tests
const { MongoMemoryServer } = require('mongodb-memory-server');
const Database = require('./database');

describe('User Service Database Integration', () => {
  let mongoServer;
  let database;
  
  // Setup: Start in-memory MongoDB before all tests
  beforeAll(async () => {
    mongoServer = await MongoMemoryServer.create();
    const mongoUri = mongoServer.getUri();
    
    database = new Database(mongoUri, 'test-db');
    await database.connect();
  });
  
  // Teardown: Stop MongoDB after all tests
  afterAll(async () => {
    await database.disconnect();
    await mongoServer.stop();
  });
  
  // Clean database before each test
  beforeEach(async () => {
    await database.deleteAllUsers();
  });
  
  describe('insertUser', () => {
    test('should insert user into database', async () => {
      const user = {
        name: 'John Doe',
        email: 'john@example.com',
        age: 30
      };
      
      const result = await database.insertUser(user);
      
      expect(result).toHaveProperty('id');
      expect(result.name).toBe('John Doe');
      expect(result.email).toBe('john@example.com');
    });
  });
  
  describe('findUserById', () => {
    test('should retrieve user by ID', async () => {
      const inserted = await database.insertUser({
        name: 'Jane Smith',
        email: 'jane@example.com'
      });
      
      const found = await database.findUserById(inserted.id);
      
      expect(found).not.toBeNull();
      expect(found.name).toBe('Jane Smith');
    });
    
    test('should return null for non-existent ID', async () => {
      const { ObjectId } = require('mongodb');
      const fakeId = new ObjectId();
      
      const found = await database.findUserById(fakeId);
      
      expect(found).toBeNull();
    });
  });
  
  describe('findUserByEmail', () => {
    test('should find user by email', async () => {
      await database.insertUser({
        name: 'Test User',
        email: 'test@example.com'
      });
      
      const found = await database.findUserByEmail('test@example.com');
      
      expect(found).not.toBeNull();
      expect(found.name).toBe('Test User');
    });
  });
});
```

---

## Testing Asynchronous Code and Handling Promises in Tests

### Testing Promises

```javascript
/*
 * Testing asynchronous code is common in Node.js
 * Jest provides multiple ways to test async operations
 */

// async-functions.js - Asynchronous functions to test
const fetch = require('node-fetch');

class AsyncService {
  // Returns a promise
  async fetchUser(userId) {
    const response = await fetch(`https://api.example.com/users/${userId}`);
    
    if (!response.ok) {
      throw new Error('User not found');
    }
    
    return await response.json();
  }
  
  // Promise with timeout
  async delayedOperation(ms) {
    return new Promise((resolve) => {
      setTimeout(() => {
        resolve('Operation completed');
      }, ms);
    });
  }
  
  // Multiple async operations
  async processData(data) {
    const step1 = await this.validateData(data);
    const step2 = await this.transformData(step1);
    const step3 = await this.saveData(step2);
    return step3;
  }
  
  async validateData(data) {
    return new Promise((resolve, reject) => {
      if (!data || data.length === 0) {
        reject(new Error('Invalid data'));
      } else {
        resolve(data);
      }
    });
  }
  
  async transformData(data) {
    return data.map(item => item.toUpperCase());
  }
  
  async saveData(data) {
    return { success: true, count: data.length };
  }
}

module.exports = AsyncService;
```

### Method 1: Async/Await (Recommended)

```javascript
/*
 * Using async/await is the most readable way to test async code
 */

describe('AsyncService - async/await', () => {
  let service;
  
  beforeEach(() => {
    service = new AsyncService();
  });
  
  // Test successful async operation
  test('should fetch user data', async () => {
    // Mock fetch
    global.fetch = jest.fn().mockResolvedValue({
      ok: true,
      json: async () => ({ id: 1, name: 'John' })
    });
    
    const user = await service.fetchUser(1);
    
    expect(user).toEqual({ id: 1, name: 'John' });
    expect(fetch).toHaveBeenCalledWith('https://api.example.com/users/1');
  });
  
  // Test async error handling
  test('should throw error for failed fetch', async () => {
    global.fetch = jest.fn().mockResolvedValue({
      ok: false
    });
    
    // Use expect().rejects for async errors
    await expect(service.fetchUser(999))
      .rejects
      .toThrow('User not found');
  });
  
  // Test delayed operations
  test('should complete delayed operation', async () => {
    const result = await service.delayedOperation(100);
    
    expect(result).toBe('Operation completed');
  }, 10000); // Optional timeout in milliseconds
  
  // Test multiple async steps
  test('should process data through multiple steps', async () => {
    const input = ['hello', 'world'];
    
    const result = await service.processData(input);
    
    expect(result).toEqual({
      success: true,
      count: 2
    });
  });
  
  // Test async rejection
  test('should reject invalid data', async () => {
    await expect(service.validateData([]))
      .rejects
      .toThrow('Invalid data');
  });
});
```

### Common Testing Anti-Patterns

```javascript
// ANTI-PATTERN 1: Not returning or awaiting async operations
// BAD
test('will silently fail', () => {
  asyncOperation().then(result => {
    expect(result).toBe('expected');  // Assertion never runs!
  });
});

// GOOD
test('properly waits for async', async () => {
  const result = await asyncOperation();
  expect(result).toBe('expected');
});

// ANTI-PATTERN 2: Testing implementation details
// BAD
test('uses specific sorting algorithm', () => {
  const spy = jest.spyOn(module, '_quickSort');
  sort([3,1,2]);
  expect(spy).toHaveBeenCalled();
});

// GOOD
test('returns sorted array', () => {
  expect(sort([3,1,2])).toEqual([1,2,3]);
});

// ANTI-PATTERN 3: Too many assertions in one test
// BAD
test('user operations', () => {
  const user = createUser();
  expect(user.name).toBe('John');
  
  updateUser(user.id, { name: 'Jane' });
  expect(user.name).toBe('Jane');
  
  deleteUser(user.id);
  expect(getUser(user.id)).toBeNull();
});

// GOOD: Split into separate tests
test('creates user with name', () => {
  const user = createUser({ name: 'John' });
  expect(user.name).toBe('John');
});

test('updates user name', () => {
  const user = createUser({ name: 'John' });
  updateUser(user.id, { name: 'Jane' });
  expect(user.name).toBe('Jane');
});
```

---

## Setting Up Continuous Testing in CI/CD Pipelines

### GitHub Actions Configuration

```yaml
# .github/workflows/test.yml

name: Run Tests

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    name: Test on Node ${{ matrix.node }}
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        node: [16, 18, 20]
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests with coverage
        run: npm run test:coverage
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
```

---

## Stress and Performance Testing using K6

### Basic K6 Test

```javascript
// load-test.js
import http from 'k6/http';
import { sleep, check } from 'k6';

export const options = {
  stages: [
    { duration: '30s', target: 20 },
    { duration: '1m', target: 50 },
    { duration: '30s', target: 0 },
  ],
  thresholds: {
    http_req_failed: ['rate<0.01'],
    http_req_duration: ['p(95)<500'],
  },
};

export default function () {
  const response = http.get('http://localhost:3000/api/users');
  
  check(response, {
    'status is 200': (r) => r.status === 200,
    'response time < 500ms': (r) => r.timings.duration < 500,
  });
  
  sleep(1);
}
```

---

## Performance Testing and Monitoring using Postman

### Newman CLI Testing

```bash
# Install Newman
npm install -g newman newman-reporter-htmlextra

# Run collection
newman run collection.json \
  --environment env.json \
  --reporters cli,htmlextra \
  --reporter-htmlextra-export report.html

# Run with iterations (load testing)
newman run collection.json --iteration-count 100
```

---

## Best Practices and Patterns

### Testing Best Practices Summary

```javascript
// 1. Write descriptive test names
test('should return 404 when user does not exist', () => {});

// 2. Follow AAA pattern
test('example', () => {
  // Arrange
  const input = 'test';
  
  // Act
  const result = process(input);
  
  // Assert
  expect(result).toBe('expected');
});

// 3. Keep tests independent
beforeEach(() => {
  // Reset state before each test
});

// 4. Test edge cases
test('handles empty input', () => {});
test('handles null input', () => {});
test('handles very large input', () => {});

// 5. Use appropriate matchers
expect(value).toBe(expected);        // Strict equality
expect(object).toEqual(expected);    // Deep equality
expect(array).toContain(item);       // Array contains
expect(fn).toThrow(error);           // Throws error
```

---

## Real-World Testing Scenarios

### E-commerce Checkout Testing

```javascript
class CheckoutService {
  async processCheckout(cart, payment) {
    // 1. Validate cart
    if (cart.items.length === 0) {
      throw new Error('Cart is empty');
    }
    
    // 2. Check inventory
    await this.inventoryService.checkStock(cart.items);
    
    // 3. Process payment
    const paymentResult = await this.paymentService.charge(payment);
    
    // 4. Create order
    const order = await this.orderService.create({
      items: cart.items,
      total: cart.total,
      paymentId: paymentResult.id
    });
    
    // 5. Send confirmation
    await this.emailService.sendOrderConfirmation(order);
    
    return order;
  }
}

describe('CheckoutService', () => {
  test('should successfully process checkout', async () => {
    // Arrange: Mock all dependencies
    const mockInventory = { checkStock: jest.fn().mockResolvedValue(true) };
    const mockPayment = { charge: jest.fn().mockResolvedValue({ id: 'pay-123' }) };
    const mockOrder = { create: jest.fn().mockResolvedValue({ id: 'ord-123' }) };
    const mockEmail = { sendOrderConfirmation: jest.fn() };
    
    const service = new CheckoutService(
      mockInventory,
      mockPayment,
      mockOrder,
      mockEmail
    );
    
    const cart = {
      items: [{ id: 1, price: 50, quantity: 2 }],
      total: 100
    };
    
    // Act
    const order = await service.processCheckout(cart, { card: '****1234' });
    
    // Assert
    expect(order.id).toBe('ord-123');
    expect(mockInventory.checkStock).toHaveBeenCalledWith(cart.items);
    expect(mockPayment.charge).toHaveBeenCalled();
    expect(mockEmail.sendOrderConfirmation).toHaveBeenCalled();
  });
  
  test('should handle payment failure', async () => {
    const mockPayment = {
      charge: jest.fn().mockRejectedValue(new Error('Payment failed'))
    };
    
    const service = new CheckoutService(null, mockPayment, null, null);
    
    await expect(service.processCheckout({ items: [{}], total: 100 }, {}))
      .rejects.toThrow('Payment failed');
  });
});
```

---

## Conclusion

This comprehensive tutorial has covered:

### Key Takeaways

1. **Unit Testing**: Write focused tests for individual functions using Jest
2. **TDD**: Write tests first, then implement (Red-Green-Refactor)
3. **Mocking**: Isolate units using mocks, stubs, and spies
4. **Integration Testing**: Test component interactions with Supertest
5. **Async Testing**: Handle promises and async operations correctly
6. **CI/CD**: Automate testing in pipelines
7. **Performance Testing**: Use K6 for load and stress testing
8. **Best Practices**: Write clear, maintainable, independent tests

### Testing Pyramid

```
       /\
      /E2E\       Few tests: Complete workflows
     /------\
    / Integ. \    Some tests: Component integration  
   /----------\
  / Unit Tests \  Many tests: Individual functions
 /--------------\
```

### Coverage Goals

- **Unit Tests**: 70-80% of tests
- **Integration Tests**: 15-20% of tests
- **E2E Tests**: 5-10% of tests
- **Code Coverage**: 80%+ target

### Essential Tools

| Tool | Purpose |
|------|--------|
| Jest | Testing framework |
| Supertest | HTTP testing |
| Sinon | Advanced mocking |
| K6 | Load testing |
| Newman | Postman CLI |

### Next Steps

1. Start writing tests for critical code
2. Set up CI/CD pipelines
3. Aim for 80%+ coverage
4. Make testing part of development workflow
5. Keep learning and improving

**Remember**: "Code without tests is broken by design."

### Resources

- Jest Documentation: https://jestjs.io/
- K6 Documentation: https://k6.io/docs/
- Testing JavaScript: https://testingjavascript.com/
- Test Driven Development by Kent Beck

---

**Happy Testing! 🚀**

May your tests always pass and your code always work!