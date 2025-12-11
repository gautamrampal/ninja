# Zod Complete Mastery Tutorial: From Beginner to Advanced

## Table of Contents
1. [Introduction](#introduction)
2. [What is Zod?](#what-is-zod)
3. [Installation and Setup](#installation-and-setup)
4. [Beginner Level](#beginner-level)
5. [Intermediate Level](#intermediate-level)
6. [Advanced Level](#advanced-level)
7. [Expert/Ninja Level](#expert-ninja-level)
8. [Real-World Use Cases](#real-world-use-cases)
9. [Best Practices](#best-practices)
10. [Performance Optimization](#performance-optimization)
11. [Conclusion](#conclusion)

---

## Introduction

**Zod** is a TypeScript-first schema declaration and validation library. It provides a powerful, type-safe way to validate data at runtime while automatically inferring static TypeScript types from your schemas. This tutorial will take you from absolute beginner to ninja-level mastery of Zod.

### Why Zod?

- **TypeScript-first**: Designed with TypeScript in mind, offering excellent type inference
- **Zero dependencies**: Lightweight and fast
- **Composable**: Build complex schemas from simple primitives
- **Immutable**: Schema transformations return new schemas
- **Concise**: Minimal API surface with maximum power
- **Works everywhere**: Browser, Node.js, Deno, Bun

### Prerequisites

- Basic knowledge of JavaScript/TypeScript
- Understanding of type systems (helpful but not required)
- Node.js or Bun installed

---

## What is Zod?

Zod is a validation library that allows you to:

1. **Define schemas** for your data structures
2. **Validate data** at runtime
3. **Infer TypeScript types** automatically from schemas
4. **Transform data** during validation
5. **Handle errors** gracefully

### Core Concepts

```typescript
// Schema: A blueprint for your data
// Validation: Checking if data matches the schema
// Type Inference: Automatically getting TypeScript types from schemas
// Parsing: Validation + returning typed data
```

---

## Installation and Setup

### Using npm

```bash
npm install zod
```

### Using Yarn

```bash
yarn add zod
```

### Using pnpm

```bash
pnpm add zod
```

### Using Bun

```bash
bun add zod
```

### Basic Import

```typescript
import { z } from "zod";
```

### TypeScript Configuration

Ensure your `tsconfig.json` has strict mode enabled for best results:

```json
{
  "compilerOptions": {
    "strict": true,
    "strictNullChecks": true
  }
}
```

---

## Beginner Level

### 1. Primitive Types

Zod provides validators for all JavaScript primitive types.

```typescript
import { z } from "zod";

// String schema
const stringSchema = z.string();

// Number schema
const numberSchema = z.number();

// Boolean schema
const booleanSchema = z.boolean();

// Date schema
const dateSchema = z.date();

// BigInt schema
const bigintSchema = z.bigint();

// Symbol schema
const symbolSchema = z.symbol();

// Undefined schema
const undefinedSchema = z.undefined();

// Null schema
const nullSchema = z.null();

// Void schema (accepts undefined)
const voidSchema = z.void();

// Any schema (accepts anything - use sparingly!)
const anySchema = z.any();

// Unknown schema (safer than any)
const unknownSchema = z.unknown();

// Never schema (accepts nothing)
const neverSchema = z.never();
```

### 2. Basic Validation

```typescript
// Parsing - throws error if validation fails
const result = stringSchema.parse("hello"); // ✅ Returns "hello"
stringSchema.parse(123); // ❌ Throws ZodError

// Safe parsing - returns success/error object
const safeResult = stringSchema.safeParse("hello");
if (safeResult.success) {
  console.log(safeResult.data); // "hello"
} else {
  console.log(safeResult.error); // ZodError object
}

// Example with numbers
const ageSchema = z.number();
const age = ageSchema.parse(25); // ✅ Returns 25

const safeParse = ageSchema.safeParse("25");
console.log(safeParse.success); // false
console.log(safeParse.error?.issues); // Validation error details
```

### 3. String Validations

```typescript
// Basic string validations
const emailSchema = z.string().email(); // Validates email format
const urlSchema = z.string().url(); // Validates URL format
const uuidSchema = z.string().uuid(); // Validates UUID format
const cuidSchema = z.string().cuid(); // Validates CUID format

// Length validations
const usernameSchema = z.string().min(3).max(20); // 3-20 characters
const exactLengthSchema = z.string().length(10); // Exactly 10 characters

// Pattern validations (regex)
const phoneSchema = z.string().regex(/^\+?[1-9]\d{1,14}$/); // Phone number
const alphanumericSchema = z.string().regex(/^[a-zA-Z0-9]+$/);

// String transformations
const trimmedSchema = z.string().trim(); // Removes whitespace
const lowercaseSchema = z.string().toLowerCase(); // Converts to lowercase
const uppercaseSchema = z.string().toUpperCase(); // Converts to uppercase

// Custom error messages
const passwordSchema = z.string()
  .min(8, { message: "Password must be at least 8 characters" })
  .regex(/[A-Z]/, { message: "Password must contain uppercase letter" })
  .regex(/[0-9]/, { message: "Password must contain a number" });

// Example usage
const email = emailSchema.parse("user@example.com"); // ✅
const username = usernameSchema.parse("john_doe"); // ✅
```

### 4. Number Validations

```typescript
// Basic number validations
const positiveSchema = z.number().positive(); // > 0
const negativeSchema = z.number().negative(); // < 0
const nonNegativeSchema = z.number().nonnegative(); // >= 0
const nonPositiveSchema = z.number().nonpositive(); // <= 0

// Range validations
const ageSchema = z.number().min(0).max(120);
const percentageSchema = z.number().min(0).max(100);
const scoreSchema = z.number().gte(0).lte(100); // Greater/Less than or equal

// Integer validation
const integerSchema = z.number().int(); // Must be whole number
const safeIntSchema = z.number().int().safe(); // Within JavaScript safe integer range

// Multiple validation
const priceSchema = z.number().positive().multipleOf(0.01); // Price with 2 decimals

// Finite number (not Infinity or NaN)
const finiteSchema = z.number().finite();

// Custom validations
const evenNumberSchema = z.number().refine((val) => val % 2 === 0, {
  message: "Number must be even",
});

// Example usage
const age = ageSchema.parse(25); // ✅
const price = priceSchema.parse(19.99); // ✅
const evenNum = evenNumberSchema.parse(4); // ✅
```

### 5. Object Schemas

```typescript
// Basic object schema
const userSchema = z.object({
  name: z.string(),
  age: z.number(),
  email: z.string().email(),
});

// Type inference - automatically get TypeScript type
type User = z.infer<typeof userSchema>;
// Equivalent to:
// type User = {
//   name: string;
//   age: number;
//   email: string;
// }

// Parse object
const user = userSchema.parse({
  name: "John Doe",
  age: 30,
  email: "john@example.com",
});

// Nested objects
const addressSchema = z.object({
  street: z.string(),
  city: z.string(),
  country: z.string(),
  zipCode: z.string(),
});

const userWithAddressSchema = z.object({
  name: z.string(),
  age: z.number(),
  address: addressSchema, // Nested schema
});

type UserWithAddress = z.infer<typeof userWithAddressSchema>;

// Example usage
const userWithAddress = userWithAddressSchema.parse({
  name: "Jane Doe",
  age: 28,
  address: {
    street: "123 Main St",
    city: "New York",
    country: "USA",
    zipCode: "10001",
  },
});
```

### 6. Array Schemas

```typescript
// Array of strings
const stringArraySchema = z.array(z.string());

// Array of numbers
const numberArraySchema = z.array(z.number());

// Array of objects
const usersArraySchema = z.array(userSchema);

// Array validations
const nonEmptyArray = z.array(z.string()).nonempty(); // At least 1 element
const minLengthArray = z.array(z.string()).min(2); // At least 2 elements
const maxLengthArray = z.array(z.string()).max(10); // At most 10 elements
const exactLengthArray = z.array(z.string()).length(5); // Exactly 5 elements

// Example usage
const tags = stringArraySchema.parse(["typescript", "zod", "validation"]);

const users = usersArraySchema.parse([
  { name: "Alice", age: 25, email: "alice@example.com" },
  { name: "Bob", age: 30, email: "bob@example.com" },
]);
```

### 7. Optional and Nullable

```typescript
// Optional - can be undefined
const optionalStringSchema = z.string().optional();
type OptionalString = z.infer<typeof optionalStringSchema>; // string | undefined

// Nullable - can be null
const nullableStringSchema = z.string().nullable();
type NullableString = z.infer<typeof nullableStringSchema>; // string | null

// Both optional and nullable
const optionalNullableSchema = z.string().optional().nullable();
type OptionalNullable = z.infer<typeof optionalNullableSchema>; // string | null | undefined

// Optional with default value
const nameSchema = z.string().optional().default("Anonymous");

// Object with optional fields
const profileSchema = z.object({
  username: z.string(),
  bio: z.string().optional(),
  website: z.string().url().optional(),
  age: z.number().nullable(),
});

type Profile = z.infer<typeof profileSchema>;
// {
//   username: string;
//   bio?: string | undefined;
//   website?: string | undefined;
//   age: number | null;
// }

// Example usage
const profile = profileSchema.parse({
  username: "johndoe",
  age: null,
  // bio and website are optional, so can be omitted
});
```

### 8. Literal and Enum Types

```typescript
// Literal values
const literalSchema = z.literal("hello"); // Only accepts "hello"
const numberLiteralSchema = z.literal(42); // Only accepts 42
const booleanLiteralSchema = z.literal(true); // Only accepts true

// Enums - using native TypeScript enum
enum Role {
  Admin = "admin",
  User = "user",
  Guest = "guest",
}

const roleEnumSchema = z.nativeEnum(Role);

// Enums - using Zod enum
const statusSchema = z.enum(["pending", "approved", "rejected"]);
type Status = z.infer<typeof statusSchema>; // "pending" | "approved" | "rejected"

// Union of literals (similar to enum)
const directionSchema = z.union([
  z.literal("north"),
  z.literal("south"),
  z.literal("east"),
  z.literal("west"),
]);

// Object with enum/literal fields
const taskSchema = z.object({
  id: z.string().uuid(),
  title: z.string(),
  status: statusSchema,
  priority: z.enum(["low", "medium", "high"]),
  role: roleEnumSchema,
});

// Example usage
const task = taskSchema.parse({
  id: "550e8400-e29b-41d4-a716-446655440000",
  title: "Complete tutorial",
  status: "pending",
  priority: "high",
  role: Role.Admin,
});
```

### 9. Default Values

```typescript
// Simple defaults
const stringWithDefault = z.string().default("default value");
const numberWithDefault = z.number().default(0);
const booleanWithDefault = z.boolean().default(false);

// Function defaults (computed at parse time)
const timestampSchema = z.date().default(() => new Date());
const uuidSchema = z.string().default(() => crypto.randomUUID());

// Object with defaults
const configSchema = z.object({
  port: z.number().default(3000),
  host: z.string().default("localhost"),
  debug: z.boolean().default(false),
  timeout: z.number().default(30000),
});

// Parse with missing fields - defaults will be applied
const config = configSchema.parse({});
console.log(config);
// {
//   port: 3000,
//   host: "localhost",
//   debug: false,
//   timeout: 30000
// }

// Partial parse with some fields
const customConfig = configSchema.parse({
  port: 8080,
  debug: true,
});
console.log(customConfig);
// {
//   port: 8080,
//   host: "localhost",
//   debug: true,
//   timeout: 30000
// }
```

### 10. Error Handling

```typescript
import { z, ZodError } from "zod";

// Basic error handling
try {
  const userSchema = z.object({
    email: z.string().email(),
    age: z.number().min(18),
  });

  userSchema.parse({
    email: "invalid-email",
    age: 15,
  });
} catch (error) {
  if (error instanceof ZodError) {
    console.log(error.issues);
    // [
    //   {
    //     code: "invalid_string",
    //     validation: "email",
    //     message: "Invalid email",
    //     path: ["email"]
    //   },
    //   {
    //     code: "too_small",
    //     minimum: 18,
    //     type: "number",
    //     inclusive: true,
    //     message: "Number must be greater than or equal to 18",
    //     path: ["age"]
    //   }
    // ]
  }
}

// Safe parse (no exception thrown)
const result = userSchema.safeParse({
  email: "invalid-email",
  age: 15,
});

if (!result.success) {
  console.log(result.error.format());
  // {
  //   email: { _errors: ["Invalid email"] },
  //   age: { _errors: ["Number must be greater than or equal to 18"] },
  //   _errors: []
  // }
}

// Custom error messages
const passwordSchema = z
  .string()
  .min(8, "Password must be at least 8 characters")
  .regex(/[A-Z]/, "Password must contain an uppercase letter")
  .regex(/[0-9]/, "Password must contain a number");

// Formatting errors for display
function formatZodError(error: ZodError): Record<string, string> {
  const formatted: Record<string, string> = {};
  
  error.issues.forEach((issue) => {
    const path = issue.path.join(".");
    formatted[path] = issue.message;
  });
  
  return formatted;
}

const parseResult = passwordSchema.safeParse("weak");
if (!parseResult.success) {
  console.log(formatZodError(parseResult.error));
  // {
  //   "": "Password must be at least 8 characters"
  // }
}
```

---

## Intermediate Level

### 1. Union Types

```typescript
// Simple union
const stringOrNumberSchema = z.union([z.string(), z.number()]);
type StringOrNumber = z.infer<typeof stringOrNumberSchema>; // string | number

// Discriminated union (tagged union)
const successResponseSchema = z.object({
  status: z.literal("success"),
  data: z.object({
    id: z.string(),
    name: z.string(),
  }),
});

const errorResponseSchema = z.object({
  status: z.literal("error"),
  error: z.object({
    code: z.string(),
    message: z.string(),
  }),
});

const apiResponseSchema = z.discriminatedUnion("status", [
  successResponseSchema,
  errorResponseSchema,
]);

type ApiResponse = z.infer<typeof apiResponseSchema>;

// Example usage
function handleResponse(response: ApiResponse) {
  if (response.status === "success") {
    console.log(response.data.name); // TypeScript knows this is available
  } else {
    console.log(response.error.message); // TypeScript knows this is available
  }
}

// Union of object types with different shapes
const paymentMethodSchema = z.union([
  z.object({
    type: z.literal("credit_card"),
    cardNumber: z.string(),
    expiryDate: z.string(),
    cvv: z.string(),
  }),
  z.object({
    type: z.literal("paypal"),
    email: z.string().email(),
  }),
  z.object({
    type: z.literal("bank_transfer"),
    accountNumber: z.string(),
    routingNumber: z.string(),
  }),
]);
```

### 2. Intersection Types

```typescript
// Basic intersection
const nameSchema = z.object({
  firstName: z.string(),
  lastName: z.string(),
});

const contactSchema = z.object({
  email: z.string().email(),
  phone: z.string(),
});

// Combine two schemas using intersection
const personSchema = z.intersection(nameSchema, contactSchema);
// Or using .and()
const personSchema2 = nameSchema.and(contactSchema);

type Person = z.infer<typeof personSchema>;
// {
//   firstName: string;
//   lastName: string;
//   email: string;
//   phone: string;
// }

// Multiple intersections
const addressSchema = z.object({
  street: z.string(),
  city: z.string(),
});

const fullPersonSchema = nameSchema.and(contactSchema).and(addressSchema);

// Practical example: Base + Extensions
const baseUserSchema = z.object({
  id: z.string().uuid(),
  createdAt: z.date(),
  updatedAt: z.date(),
});

const adminUserSchema = baseUserSchema.and(
  z.object({
    role: z.literal("admin"),
    permissions: z.array(z.string()),
  })
);

const regularUserSchema = baseUserSchema.and(
  z.object({
    role: z.literal("user"),
    subscription: z.enum(["free", "premium"]),
  })
);
```

### 3. Record, Map, and Set Schemas

```typescript
// Record - object with string keys and typed values
const stringRecordSchema = z.record(z.string()); // { [key: string]: string }
const numberRecordSchema = z.record(z.number()); // { [key: string]: number }

// Record with specific key types
const userRolesSchema = z.record(
  z.enum(["admin", "user", "guest"]), // Keys must be one of these
  z.object({
    permissions: z.array(z.string()),
    level: z.number(),
  })
);

// Example usage
const roles = userRolesSchema.parse({
  admin: { permissions: ["read", "write", "delete"], level: 3 },
  user: { permissions: ["read", "write"], level: 2 },
  guest: { permissions: ["read"], level: 1 },
});

// Map schema
const userMapSchema = z.map(
  z.string(), // Key type
  z.object({  // Value type
    name: z.string(),
    age: z.number(),
  })
);

const users = new Map();
users.set("user1", { name: "Alice", age: 25 });
users.set("user2", { name: "Bob", age: 30 });

const validatedUsers = userMapSchema.parse(users);

// Set schema
const stringSetSchema = z.set(z.string());
const numberSetSchema = z.set(z.number());

// Set with validations
const uniqueEmailsSchema = z.set(z.string().email()).min(1).max(100);

const emails = new Set(["alice@example.com", "bob@example.com"]);
const validatedEmails = uniqueEmailsSchema.parse(emails);

// Practical example: Configuration with dynamic keys
const envConfigSchema = z.record(
  z.string().regex(/^[A-Z_]+$/), // Environment variable names
  z.string()
);

const config = envConfigSchema.parse({
  DATABASE_URL: "postgresql://localhost:5432/mydb",
  API_KEY: "secret-key",
  PORT: "3000",
});
```

### 4. Tuple Schemas

```typescript
// Basic tuple
const coordinatesSchema = z.tuple([z.number(), z.number()]);
type Coordinates = z.infer<typeof coordinatesSchema>; // [number, number]

// Tuple with different types
const userTupleSchema = z.tuple([
  z.string(), // name
  z.number(), // age
  z.boolean(), // isActive
]);

// Example usage
const coordinates = coordinatesSchema.parse([40.7128, -74.0060]); // ✅
const user = userTupleSchema.parse(["John", 30, true]); // ✅

// Tuple with rest elements
const mixedTupleSchema = z.tuple([
  z.string(),
  z.number(),
]).rest(z.boolean()); // First two are string and number, rest are booleans

type MixedTuple = z.infer<typeof mixedTupleSchema>;
// [string, number, ...boolean[]]

const mixed = mixedTupleSchema.parse(["hello", 42, true, false, true]);

// Practical example: CSV row parsing
const csvRowSchema = z.tuple([
  z.string(), // ID
  z.string(), // Name
  z.string().transform(val => parseInt(val)), // Age (string to number)
  z.string().email(), // Email
  z.enum(["active", "inactive"]), // Status
]);

const row = csvRowSchema.parse([
  "001",
  "John Doe",
  "30",
  "john@example.com",
  "active",
]);

console.log(row);
// ["001", "John Doe", 30, "john@example.com", "active"]
```

### 5. Recursive Schemas

```typescript
// Define interface for type inference
interface Category {
  name: string;
  subcategories: Category[];
}

// Recursive schema using z.lazy()
const categorySchema: z.ZodType<Category> = z.lazy(() =>
  z.object({
    name: z.string(),
    subcategories: z.array(categorySchema), // References itself
  })
);

// Example usage
const categories = categorySchema.parse({
  name: "Electronics",
  subcategories: [
    {
      name: "Computers",
      subcategories: [
        {
          name: "Laptops",
          subcategories: [],
        },
        {
          name: "Desktops",
          subcategories: [],
        },
      ],
    },
    {
      name: "Mobile Phones",
      subcategories: [],
    },
  ],
});

// Tree structure example
interface TreeNode {
  value: number;
  left?: TreeNode;
  right?: TreeNode;
}

const treeNodeSchema: z.ZodType<TreeNode> = z.lazy(() =>
  z.object({
    value: z.number(),
    left: treeNodeSchema.optional(),
    right: treeNodeSchema.optional(),
  })
);

const tree = treeNodeSchema.parse({
  value: 10,
  left: {
    value: 5,
    left: { value: 3 },
    right: { value: 7 },
  },
  right: {
    value: 15,
    right: { value: 20 },
  },
});

// JSON value (recursive)
type JsonValue = 
  | string 
  | number 
  | boolean 
  | null 
  | JsonValue[] 
  | { [key: string]: JsonValue };

const jsonValueSchema: z.ZodType<JsonValue> = z.lazy(() =>
  z.union([
    z.string(),
    z.number(),
    z.boolean(),
    z.null(),
    z.array(jsonValueSchema),
    z.record(jsonValueSchema),
  ])
);
```

### 6. Transform and Refine

```typescript
// Transform - modify data after validation
const stringToNumberSchema = z.string().transform((val) => parseInt(val));
const num = stringToNumberSchema.parse("123"); // Returns 123 (number)

// Transform with validation
const trimAndLowerSchema = z
  .string()
  .transform((val) => val.trim().toLowerCase())
  .refine((val) => val.length > 0, {
    message: "String cannot be empty after trimming",
  });

// Multiple transformations
const dateStringSchema = z
  .string()
  .regex(/^\d{4}-\d{2}-\d{2}$/)
  .transform((str) => new Date(str))
  .refine((date) => !isNaN(date.getTime()), {
    message: "Invalid date",
  });

// Refine - add custom validation logic
const passwordSchema = z
  .string()
  .min(8)
  .refine((val) => /[A-Z]/.test(val), {
    message: "Password must contain uppercase letter",
  })
  .refine((val) => /[a-z]/.test(val), {
    message: "Password must contain lowercase letter",
  })
  .refine((val) => /[0-9]/.test(val), {
    message: "Password must contain number",
  })
  .refine((val) => /[^A-Za-z0-9]/.test(val), {
    message: "Password must contain special character",
  });

// SuperRefine - more powerful, can add multiple issues
const confirmPasswordSchema = z
  .object({
    password: z.string().min(8),
    confirmPassword: z.string(),
  })
  .superRefine((data, ctx) => {
    if (data.password !== data.confirmPassword) {
      ctx.addIssue({
        code: z.ZodIssueCode.custom,
        message: "Passwords do not match",
        path: ["confirmPassword"],
      });
    }
  });

// Practical example: Form data transformation
const signupFormSchema = z.object({
  username: z
    .string()
    .min(3)
    .transform((val) => val.toLowerCase()),
  email: z
    .string()
    .email()
    .transform((val) => val.toLowerCase().trim()),
  age: z
    .string()
    .transform((val) => parseInt(val))
    .refine((val) => val >= 18, {
      message: "Must be 18 or older",
    }),
  agreeToTerms: z
    .string()
    .optional()
    .transform((val) => val === "on"),
});

const formData = signupFormSchema.parse({
  username: "JohnDoe",
  email: "  JOHN@EXAMPLE.COM  ",
  age: "25",
  agreeToTerms: "on",
});

console.log(formData);
// {
//   username: "johndoe",
//   email: "john@example.com",
//   age: 25,
//   agreeToTerms: true
// }
```

### 7. Preprocessing and Postprocessing

```typescript
// Preprocess - transform input before validation
const preprocessedSchema = z.preprocess(
  (input) => {
    // Convert input to expected type before validation
    if (typeof input === "string") {
      return parseInt(input);
    }
    return input;
  },
  z.number().positive()
);

const result = preprocessedSchema.parse("123"); // Returns 123 (number)

// Practical: Handle JSON strings
const jsonSchema = z.preprocess(
  (input) => {
    if (typeof input === "string") {
      try {
        return JSON.parse(input);
      } catch {
        return input;
      }
    }
    return input;
  },
  z.object({
    name: z.string(),
    age: z.number(),
  })
);

const parsed = jsonSchema.parse('{"name":"John","age":30}');
// Returns { name: "John", age: 30 }

// Coercion helpers
const coercedNumberSchema = z.coerce.number(); // Converts to number
const coercedBooleanSchema = z.coerce.boolean(); // Converts to boolean
const coercedDateSchema = z.coerce.date(); // Converts to Date

// Examples
coercedNumberSchema.parse("123"); // 123
coercedNumberSchema.parse("12.5"); // 12.5
coercedBooleanSchema.parse("true"); // true
coercedBooleanSchema.parse(1); // true
coercedDateSchema.parse("2023-01-01"); // Date object

// Practical: Query parameter parsing
const queryParamsSchema = z.object({
  page: z.coerce.number().positive().default(1),
  limit: z.coerce.number().positive().max(100).default(10),
  sortBy: z.enum(["name", "date", "popular"]).default("date"),
  order: z.enum(["asc", "desc"]).default("asc"),
  search: z.string().optional(),
});

// Parse URL query parameters (all strings)
const params = queryParamsSchema.parse({
  page: "2",
  limit: "20",
  sortBy: "popular",
  search: "zod tutorial",
});

console.log(params);
// {
//   page: 2,
//   limit: 20,
//   sortBy: "popular",
//   order: "asc",
//   search: "zod tutorial"
// }
```

### 8. Partial, Pick, and Omit

```typescript
const userSchema = z.object({
  id: z.string().uuid(),
  username: z.string(),
  email: z.string().email(),
  password: z.string().min(8),
  age: z.number(),
  createdAt: z.date(),
});

// Partial - all fields optional
const partialUserSchema = userSchema.partial();
type PartialUser = z.infer<typeof partialUserSchema>;
// {
//   id?: string;
//   username?: string;
//   email?: string;
//   password?: string;
//   age?: number;
//   createdAt?: Date;
// }

// Partial with specific fields
const partialEmailUserSchema = userSchema.partial({
  email: true,
  age: true,
});
// email and age are optional, rest are required

// Required - opposite of partial
const requiredUserSchema = partialUserSchema.required();

// Pick - select specific fields
const userCredentialsSchema = userSchema.pick({
  username: true,
  password: true,
});
type UserCredentials = z.infer<typeof userCredentialsSchema>;
// {
//   username: string;
//   password: string;
// }

// Omit - exclude specific fields
const publicUserSchema = userSchema.omit({
  password: true,
  createdAt: true,
});
type PublicUser = z.infer<typeof publicUserSchema>;
// {
//   id: string;
//   username: string;
//   email: string;
//   age: number;
// }

// Extend - add new fields
const adminUserSchema = userSchema.extend({
  role: z.literal("admin"),
  permissions: z.array(z.string()),
});

// Merge - combine two schemas
const timestampSchema = z.object({
  createdAt: z.date(),
  updatedAt: z.date(),
});

const productSchema = z.object({
  id: z.string(),
  name: z.string(),
  price: z.number(),
});

const productWithTimestampSchema = productSchema.merge(timestampSchema);

// Practical example: Update DTO
const createUserSchema = userSchema.omit({
  id: true,
  createdAt: true,
});

const updateUserSchema = userSchema
  .omit({
    id: true,
    createdAt: true,
  })
  .partial();

type CreateUserDto = z.infer<typeof createUserSchema>;
type UpdateUserDto = z.infer<typeof updateUserSchema>;
```

### 9. Async Validation

```typescript
// Simple async refine
const uniqueEmailSchema = z
  .string()
  .email()
  .refine(
    async (email) => {
      // Simulated database check
      const existingUser = await checkEmailExists(email);
      return !existingUser;
    },
    {
      message: "Email already exists",
    }
  );

// Async function for validation
async function checkEmailExists(email: string): Promise<boolean> {
  // Simulate API call
  await new Promise((resolve) => setTimeout(resolve, 100));
  const existingEmails = ["existing@example.com", "taken@example.com"];
  return existingEmails.includes(email);
}

// Parse async
const result = await uniqueEmailSchema.parseAsync("new@example.com"); // ✅
// await uniqueEmailSchema.parseAsync("existing@example.com"); // ❌ Throws

// Safe parse async
const safeResult = await uniqueEmailSchema.safeParseAsync("test@example.com");
if (safeResult.success) {
  console.log("Email is unique:", safeResult.data);
} else {
  console.log("Validation failed:", safeResult.error);
}

// Multiple async validations
const signupSchema = z.object({
  username: z
    .string()
    .min(3)
    .refine(
      async (username) => {
        const exists = await checkUsernameExists(username);
        return !exists;
      },
      { message: "Username already taken" }
    ),
  email: z
    .string()
    .email()
    .refine(
      async (email) => {
        const exists = await checkEmailExists(email);
        return !exists;
      },
      { message: "Email already registered" }
    ),
  password: z.string().min(8),
});

async function checkUsernameExists(username: string): Promise<boolean> {
  await new Promise((resolve) => setTimeout(resolve, 100));
  return ["admin", "root", "user"].includes(username);
}

// Practical example: Form validation with API checks
async function validateSignupForm(data: unknown) {
  try {
    const validated = await signupSchema.parseAsync(data);
    return { success: true, data: validated };
  } catch (error) {
    if (error instanceof z.ZodError) {
      return { success: false, errors: error.format() };
    }
    throw error;
  }
}

// Usage
const formResult = await validateSignupForm({
  username: "newuser",
  email: "new@example.com",
  password: "SecurePass123",
});

console.log(formResult);
```

### 10. Custom Error Maps

```typescript
import { z, ZodError, ZodIssueCode } from "zod";

// Custom error map for localization
const customErrorMap: z.ZodErrorMap = (issue, ctx) => {
  switch (issue.code) {
    case ZodIssueCode.too_small:
      if (issue.type === "string") {
        return { message: `String must be at least ${issue.minimum} characters` };
      }
      if (issue.type === "number") {
        return { message: `Number must be at least ${issue.minimum}` };
      }
      break;
    case ZodIssueCode.too_big:
      if (issue.type === "string") {
        return { message: `String must be at most ${issue.maximum} characters` };
      }
      break;
    case ZodIssueCode.invalid_string:
      if (issue.validation === "email") {
        return { message: "Please enter a valid email address" };
      }
      if (issue.validation === "url") {
        return { message: "Please enter a valid URL" };
      }
      break;
    case ZodIssueCode.invalid_type:
      return {
        message: `Expected ${issue.expected}, received ${issue.received}`,
      };
  }
  
  return { message: ctx.defaultError };
};

// Set global error map
z.setErrorMap(customErrorMap);

// Or use per-schema
const userSchema = z.object({
  email: z.string().email(),
  age: z.number().min(18),
});

const result = userSchema.safeParse({
  email: "invalid",
  age: 15,
});

console.log(result.error?.issues);
// [
//   {
//     code: "invalid_string",
//     validation: "email",
//     message: "Please enter a valid email address",
//     path: ["email"]
//   },
//   {
//     code: "too_small",
//     minimum: 18,
//     type: "number",
//     message: "Number must be at least 18",
//     path: ["age"]
//   }
// ]

// Localization example
const errorMessages = {
  en: {
    required: "This field is required",
    invalidEmail: "Please enter a valid email address",
    minLength: (min: number) => `Must be at least ${min} characters`,
    maxLength: (max: number) => `Must be at most ${max} characters`,
  },
  es: {
    required: "Este campo es obligatorio",
    invalidEmail: "Por favor ingrese un correo electrónico válido",
    minLength: (min: number) => `Debe tener al menos ${min} caracteres`,
    maxLength: (max: number) => `Debe tener como máximo ${max} caracteres`,
  },
};

function createLocalizedErrorMap(locale: "en" | "es"): z.ZodErrorMap {
  const messages = errorMessages[locale];
  
  return (issue, ctx) => {
    switch (issue.code) {
      case ZodIssueCode.invalid_type:
        if (issue.received === "undefined") {
          return { message: messages.required };
        }
        break;
      case ZodIssueCode.invalid_string:
        if (issue.validation === "email") {
          return { message: messages.invalidEmail };
        }
        break;
      case ZodIssueCode.too_small:
        if (issue.type === "string") {
          return { message: messages.minLength(issue.minimum as number) };
        }
        break;
      case ZodIssueCode.too_big:
        if (issue.type === "string") {
          return { message: messages.maxLength(issue.maximum as number) };
        }
        break;
    }
    
    return { message: ctx.defaultError };
  };
}

// Use Spanish error messages
z.setErrorMap(createLocalizedErrorMap("es"));
```

---

## Advanced Level

### 1. Schema Composition Patterns

```typescript
// Base schemas for reusability
const timestampSchema = z.object({
  createdAt: z.date(),
  updatedAt: z.date(),
});

const softDeleteSchema = z.object({
  deletedAt: z.date().nullable(),
  isDeleted: z.boolean().default(false),
});

const auditSchema = z.object({
  createdBy: z.string().uuid(),
  updatedBy: z.string().uuid(),
});

// Compose complex schemas
const baseEntitySchema = timestampSchema.merge(softDeleteSchema);

const auditedEntitySchema = baseEntitySchema.merge(auditSchema);

// Factory pattern for common schemas
function createEntitySchema<T extends z.ZodRawShape>(shape: T) {
  return z.object(shape).merge(auditedEntitySchema);
}

// Usage
const productSchema = createEntitySchema({
  id: z.string().uuid(),
  name: z.string(),
  price: z.number().positive(),
  category: z.string(),
});

const orderSchema = createEntitySchema({
  id: z.string().uuid(),
  userId: z.string().uuid(),
  items: z.array(
    z.object({
      productId: z.string().uuid(),
      quantity: z.number().positive().int(),
      price: z.number().positive(),
    })
  ),
  total: z.number().positive(),
  status: z.enum(["pending", "processing", "shipped", "delivered", "cancelled"]),
});

// Mixin pattern
function withPagination<T extends z.ZodTypeAny>(schema: T) {
  return z.object({
    data: z.array(schema),
    pagination: z.object({
      page: z.number().positive().int(),
      limit: z.number().positive().int(),
      total: z.number().nonnegative().int(),
      totalPages: z.number().positive().int(),
    }),
  });
}

const paginatedUsersSchema = withPagination(userSchema);

// Conditional schemas
function createConditionalSchema(userType: "admin" | "user") {
  const baseSchema = z.object({
    id: z.string().uuid(),
    username: z.string(),
    email: z.string().email(),
  });

  if (userType === "admin") {
    return baseSchema.extend({
      role: z.literal("admin"),
      permissions: z.array(z.string()),
      canAccessAdmin: z.literal(true),
    });
  }

  return baseSchema.extend({
    role: z.literal("user"),
    subscription: z.enum(["free", "premium"]),
  });
}

const adminSchema = createConditionalSchema("admin");
const userSchema = createConditionalSchema("user");
```

### 2. Advanced Validation Patterns

```typescript
// Cross-field validation
const eventSchema = z
  .object({
    title: z.string(),
    startDate: z.date(),
    endDate: z.date(),
    isAllDay: z.boolean(),
    startTime: z.string().optional(),
    endTime: z.string().optional(),
  })
  .superRefine((data, ctx) => {
    // Validate end date is after start date
    if (data.endDate <= data.startDate) {
      ctx.addIssue({
        code: z.ZodIssueCode.custom,
        message: "End date must be after start date",
        path: ["endDate"],
      });
    }

    // If not all-day event, times are required
    if (!data.isAllDay) {
      if (!data.startTime) {
        ctx.addIssue({
          code: z.ZodIssueCode.custom,
          message: "Start time is required for non-all-day events",
          path: ["startTime"],
        });
      }
      if (!data.endTime) {
        ctx.addIssue({
          code: z.ZodIssueCode.custom,
          message: "End time is required for non-all-day events",
          path: ["endTime"],
        });
      }
    }
  });

// Conditional validation with discriminated unions
const notificationSchema = z.discriminatedUnion("type", [
  z.object({
    type: z.literal("email"),
    to: z.string().email(),
    subject: z.string(),
    body: z.string(),
    attachments: z.array(z.string().url()).optional(),
  }),
  z.object({
    type: z.literal("sms"),
    phoneNumber: z.string().regex(/^\+?[1-9]\d{1,14}$/),
    message: z.string().max(160),
  }),
  z.object({
    type: z.literal("push"),
    deviceId: z.string(),
    title: z.string(),
    body: z.string(),
    data: z.record(z.unknown()).optional(),
  }),
]);

// Complex business rules
const orderItemSchema = z.object({
  productId: z.string(),
  quantity: z.number().positive().int(),
  unitPrice: z.number().positive(),
  discount: z.number().min(0).max(1), // 0-100% as decimal
});

const orderSchema = z
  .object({
    items: z.array(orderItemSchema).min(1),
    subtotal: z.number().positive(),
    tax: z.number().nonnegative(),
    shipping: z.number().nonnegative(),
    total: z.number().positive(),
    discountCode: z.string().optional(),
  })
  .superRefine((data, ctx) => {
    // Calculate expected subtotal
    const calculatedSubtotal = data.items.reduce((sum, item) => {
      const itemTotal = item.quantity * item.unitPrice * (1 - item.discount);
      return sum + itemTotal;
    }, 0);

    // Validate subtotal matches
    if (Math.abs(data.subtotal - calculatedSubtotal) > 0.01) {
      ctx.addIssue({
        code: z.ZodIssueCode.custom,
        message: "Subtotal does not match items total",
        path: ["subtotal"],
      });
    }

    // Validate total
    const calculatedTotal = data.subtotal + data.tax + data.shipping;
    if (Math.abs(data.total - calculatedTotal) > 0.01) {
      ctx.addIssue({
        code: z.ZodIssueCode.custom,
        message: "Total does not match subtotal + tax + shipping",
        path: ["total"],
      });
    }

    // Validate shipping threshold
    if (data.subtotal >= 50 && data.shipping > 0) {
      ctx.addIssue({
        code: z.ZodIssueCode.custom,
        message: "Free shipping applies for orders over $50",
        path: ["shipping"],
      });
    }
  });

// State machine validation
type OrderState = "pending" | "paid" | "processing" | "shipped" | "delivered" | "cancelled";

const orderStateTransitionSchema = z
  .object({
    currentState: z.enum(["pending", "paid", "processing", "shipped", "delivered", "cancelled"]),
    newState: z.enum(["pending", "paid", "processing", "shipped", "delivered", "cancelled"]),
    reason: z.string().optional(),
  })
  .superRefine((data, ctx) => {
    const validTransitions: Record<OrderState, OrderState[]> = {
      pending: ["paid", "cancelled"],
      paid: ["processing", "cancelled"],
      processing: ["shipped", "cancelled"],
      shipped: ["delivered"],
      delivered: [],
      cancelled: [],
    };

    const allowed = validTransitions[data.currentState];

    if (!allowed.includes(data.newState)) {
      ctx.addIssue({
        code: z.ZodIssueCode.custom,
        message: `Cannot transition from ${data.currentState} to ${data.newState}`,
        path: ["newState"],
      });
    }

    // Require reason for cancellation
    if (data.newState === "cancelled" && !data.reason) {
      ctx.addIssue({
        code: z.ZodIssueCode.custom,
        message: "Reason is required when cancelling",
        path: ["reason"],
      });
    }
  });
```

### 3. Schema Registry and References

```typescript
// Schema registry for managing multiple schemas
class SchemaRegistry {
  private schemas = new Map<string, z.ZodTypeAny>();

  register<T extends z.ZodTypeAny>(name: string, schema: T): void {
    this.schemas.set(name, schema);
  }

  get<T extends z.ZodTypeAny>(name: string): T {
    const schema = this.schemas.get(name);
    if (!schema) {
      throw new Error(`Schema ${name} not found`);
    }
    return schema as T;
  }

  validate<T extends z.ZodTypeAny>(
    schemaName: string,
    data: unknown
  ): z.infer<T> {
    const schema = this.get<T>(schemaName);
    return schema.parse(data);
  }

  safeValidate<T extends z.ZodTypeAny>(
    schemaName: string,
    data: unknown
  ): z.SafeParseReturnType<unknown, z.infer<T>> {
    const schema = this.get<T>(schemaName);
    return schema.safeParse(data);
  }
}

// Usage
const registry = new SchemaRegistry();

// Register schemas
registry.register(
  "User",
  z.object({
    id: z.string().uuid(),
    username: z.string(),
    email: z.string().email(),
  })
);

registry.register(
  "Product",
  z.object({
    id: z.string().uuid(),
    name: z.string(),
    price: z.number().positive(),
  })
);

// Use schemas
const user = registry.validate("User", {
  id: "550e8400-e29b-41d4-a716-446655440000",
  username: "johndoe",
  email: "john@example.com",
});

// Schema references
const addressSchema = z.object({
  street: z.string(),
  city: z.string(),
  country: z.string(),
  zipCode: z.string(),
});

const userWithAddressSchema = z.object({
  id: z.string().uuid(),
  username: z.string(),
  email: z.string().email(),
  address: addressSchema, // Reference to another schema
});

// Circular references using lazy
interface Comment {
  id: string;
  text: string;
  author: string;
  replies: Comment[];
}

const commentSchema: z.ZodType<Comment> = z.lazy(() =>
  z.object({
    id: z.string().uuid(),
    text: z.string(),
    author: z.string(),
    replies: z.array(commentSchema),
  })
);

// Schema versioning
class VersionedSchemaRegistry {
  private schemas = new Map<string, Map<number, z.ZodTypeAny>>();

  register<T extends z.ZodTypeAny>(
    name: string,
    version: number,
    schema: T
  ): void {
    if (!this.schemas.has(name)) {
      this.schemas.set(name, new Map());
    }
    this.schemas.get(name)!.set(version, schema);
  }

  get<T extends z.ZodTypeAny>(name: string, version: number): T {
    const versions = this.schemas.get(name);
    if (!versions) {
      throw new Error(`Schema ${name} not found`);
    }
    const schema = versions.get(version);
    if (!schema) {
      throw new Error(`Schema ${name} version ${version} not found`);
    }
    return schema as T;
  }

  getLatest<T extends z.ZodTypeAny>(name: string): T {
    const versions = this.schemas.get(name);
    if (!versions || versions.size === 0) {
      throw new Error(`Schema ${name} not found`);
    }
    const latestVersion = Math.max(...versions.keys());
    return this.get<T>(name, latestVersion);
  }
}

// Usage
const versionedRegistry = new VersionedSchemaRegistry();

// Version 1
versionedRegistry.register(
  "User",
  1,
  z.object({
    id: z.string(),
    name: z.string(),
  })
);

// Version 2 with additional fields
versionedRegistry.register(
  "User",
  2,
  z.object({
    id: z.string().uuid(),
    name: z.string(),
    email: z.string().email(),
    createdAt: z.date(),
  })
);

const userV1 = versionedRegistry.get("User", 1);
const userV2 = versionedRegistry.get("User", 2);
const latestUser = versionedRegistry.getLatest("User");
```

### 4. Form Validation with Zod

```typescript
// React Hook Form integration example
import { z } from "zod";

// Define form schema
const loginFormSchema = z.object({
  email: z.string().email("Invalid email address"),
  password: z.string().min(8, "Password must be at least 8 characters"),
  rememberMe: z.boolean().default(false),
});

type LoginFormData = z.infer<typeof loginFormSchema>;

// Multi-step form validation
const step1Schema = z.object({
  firstName: z.string().min(2),
  lastName: z.string().min(2),
  email: z.string().email(),
});

const step2Schema = z.object({
  address: z.string().min(5),
  city: z.string().min(2),
  country: z.string().min(2),
  zipCode: z.string().regex(/^\d{5}(-\d{4})?$/),
});

const step3Schema = z.object({
  cardNumber: z.string().regex(/^\d{16}$/),
  expiryDate: z.string().regex(/^(0[1-9]|1[0-2])\/\d{2}$/),
  cvv: z.string().regex(/^\d{3,4}$/),
});

const completeFormSchema = step1Schema.merge(step2Schema).merge(step3Schema);

type CompleteFormData = z.infer<typeof completeFormSchema>;

// Form with file upload
const MAX_FILE_SIZE = 5 * 1024 * 1024; // 5MB
const ACCEPTED_IMAGE_TYPES = ["image/jpeg", "image/png", "image/webp"];

const profileFormSchema = z.object({
  username: z.string().min(3).max(20),
  bio: z.string().max(500).optional(),
  avatar: z
    .instanceof(File)
    .refine((file) => file.size <= MAX_FILE_SIZE, {
      message: "File size must be less than 5MB",
    })
    .refine((file) => ACCEPTED_IMAGE_TYPES.includes(file.type), {
      message: "Only .jpg, .png and .webp formats are supported",
    })
    .optional(),
  socialLinks: z.object({
    twitter: z.string().url().optional(),
    github: z.string().url().optional(),
    website: z.string().url().optional(),
  }),
});

// Dynamic form validation
function createDynamicFormSchema(fields: string[]) {
  const shape: Record<string, z.ZodTypeAny> = {};

  fields.forEach((field) => {
    switch (field) {
      case "email":
        shape[field] = z.string().email();
        break;
      case "phone":
        shape[field] = z.string().regex(/^\+?[1-9]\d{1,14}$/);
        break;
      case "age":
        shape[field] = z.number().min(0).max(120);
        break;
      case "name":
        shape[field] = z.string().min(2);
        break;
      default:
        shape[field] = z.string();
    }
  });

  return z.object(shape);
}

const dynamicSchema = createDynamicFormSchema(["email", "phone", "age"]);

// Form validation with custom messages
const registrationFormSchema = z
  .object({
    username: z
      .string()
      .min(3, "Username must be at least 3 characters")
      .max(20, "Username must not exceed 20 characters")
      .regex(
        /^[a-zA-Z0-9_]+$/,
        "Username can only contain letters, numbers and underscores"
      ),
    email: z.string().email("Please enter a valid email address"),
    password: z
      .string()
      .min(8, "Password must be at least 8 characters")
      .regex(/[A-Z]/, "Password must contain at least one uppercase letter")
      .regex(/[a-z]/, "Password must contain at least one lowercase letter")
      .regex(/[0-9]/, "Password must contain at least one number")
      .regex(
        /[^A-Za-z0-9]/,
        "Password must contain at least one special character"
      ),
    confirmPassword: z.string(),
    terms: z.literal(true, {
      errorMap: () => ({ message: "You must accept the terms and conditions" }),
    }),
  })
  .refine((data) => data.password === data.confirmPassword, {
    message: "Passwords do not match",
    path: ["confirmPassword"],
  });
```

### 5. API Request/Response Validation

```typescript
// REST API schemas
const createUserRequestSchema = z.object({
  body: z.object({
    username: z.string().min(3).max(20),
    email: z.string().email(),
    password: z.string().min(8),
  }),
  params: z.object({}),
  query: z.object({}),
});

const getUserRequestSchema = z.object({
  params: z.object({
    id: z.string().uuid(),
  }),
  query: z.object({
    include: z.enum(["posts", "comments", "all"]).optional(),
  }),
  body: z.object({}),
});

const listUsersRequestSchema = z.object({
  query: z.object({
    page: z.coerce.number().positive().default(1),
    limit: z.coerce.number().positive().max(100).default(10),
    sortBy: z.enum(["createdAt", "username", "email"]).default("createdAt"),
    order: z.enum(["asc", "desc"]).default("desc"),
    search: z.string().optional(),
    role: z.enum(["admin", "user", "guest"]).optional(),
  }),
  params: z.object({}),
  body: z.object({}),
});

// Response schemas
const userResponseSchema = z.object({
  id: z.string().uuid(),
  username: z.string(),
  email: z.string().email(),
  createdAt: z.date(),
  updatedAt: z.date(),
});

const paginatedResponseSchema = <T extends z.ZodTypeAny>(dataSchema: T) =>
  z.object({
    data: z.array(dataSchema),
    meta: z.object({
      page: z.number(),
      limit: z.number(),
      total: z.number(),
      totalPages: z.number(),
    }),
  });

const errorResponseSchema = z.object({
  error: z.object({
    code: z.string(),
    message: z.string(),
    details: z.array(
      z.object({
        field: z.string(),
        message: z.string(),
      })
    ).optional(),
  }),
});

// Express.js middleware example
import { Request, Response, NextFunction } from "express";

function validateRequest<T extends z.ZodTypeAny>(schema: T) {
  return async (req: Request, res: Response, next: NextFunction) => {
    try {
      const validated = await schema.parseAsync({
        body: req.body,
        query: req.query,
        params: req.params,
      });
      
      // Replace request with validated data
      req.body = validated.body;
      req.query = validated.query;
      req.params = validated.params;
      
      next();
    } catch (error) {
      if (error instanceof z.ZodError) {
        return res.status(400).json({
          error: {
            code: "VALIDATION_ERROR",
            message: "Invalid request data",
            details: error.issues.map((issue) => ({
              field: issue.path.join("."),
              message: issue.message,
            })),
          },
        });
      }
      next(error);
    }
  };
}

// Usage in Express routes
/*
app.post(
  "/users",
  validateRequest(createUserRequestSchema),
  async (req, res) => {
    // req.body is now typed and validated
    const userData = req.body;
    // ... create user logic
  }
);

app.get(
  "/users/:id",
  validateRequest(getUserRequestSchema),
  async (req, res) => {
    // req.params and req.query are now typed and validated
    const { id } = req.params;
    const { include } = req.query;
    // ... get user logic
  }
);

app.get(
  "/users",
  validateRequest(listUsersRequestSchema),
  async (req, res) => {
    // req.query is now typed and validated with defaults applied
    const { page, limit, sortBy, order, search, role } = req.query;
    // ... list users logic
  }
);
*/

// GraphQL integration
const createPostInputSchema = z.object({
  title: z.string().min(1).max(200),
  content: z.string().min(1),
  tags: z.array(z.string()).max(10),
  published: z.boolean().default(false),
});

const updatePostInputSchema = createPostInputSchema.partial();

// tRPC integration example
const trpcProcedureInput = z.object({
  userId: z.string().uuid(),
  postId: z.string().uuid(),
  content: z.string().min(1).max(500),
});

// WebSocket message validation
const wsMessageSchema = z.discriminatedUnion("type", [
  z.object({
    type: z.literal("subscribe"),
    channel: z.string(),
  }),
  z.object({
    type: z.literal("unsubscribe"),
    channel: z.string(),
  }),
  z.object({
    type: z.literal("message"),
    channel: z.string(),
    data: z.unknown(),
  }),
  z.object({
    type: z.literal("ping"),
  }),
]);

type WSMessage = z.infer<typeof wsMessageSchema>;
```

### 6. Database Schema Validation

```typescript
// Prisma schema validation
const prismaUserSchema = z.object({
  id: z.string().uuid(),
  email: z.string().email(),
  username: z.string().min(3).max(20),
  password: z.string(),
  role: z.enum(["USER", "ADMIN", "MODERATOR"]),
  profile: z
    .object({
      bio: z.string().nullable(),
      avatar: z.string().url().nullable(),
      dateOfBirth: z.date().nullable(),
    })
    .nullable(),
  posts: z.array(
    z.object({
      id: z.string().uuid(),
      title: z.string(),
      content: z.string(),
      published: z.boolean(),
      createdAt: z.date(),
    })
  ),
  createdAt: z.date(),
  updatedAt: z.date(),
});

// MongoDB document validation
const mongoProductSchema = z.object({
  _id: z.string(),
  name: z.string(),
  description: z.string(),
  price: z.number().positive(),
  category: z.string(),
  tags: z.array(z.string()),
  stock: z.number().int().nonnegative(),
  images: z.array(z.string().url()),
  specs: z.record(z.unknown()),
  reviews: z.array(
    z.object({
      userId: z.string(),
      rating: z.number().min(1).max(5),
      comment: z.string(),
      createdAt: z.date(),
    })
  ),
  createdAt: z.date(),
  updatedAt: z.date(),
});

// SQL table validation (for ORM results)
const sqlOrderSchema = z.object({
  id: z.number().int().positive(),
  userId: z.number().int().positive(),
  orderNumber: z.string(),
  status: z.enum(["pending", "processing", "shipped", "delivered", "cancelled"]),
  subtotal: z.number().positive(),
  tax: z.number().nonnegative(),
  shipping: z.number().nonnegative(),
  total: z.number().positive(),
  shippingAddress: z.object({
    street: z.string(),
    city: z.string(),
    state: z.string(),
    zipCode: z.string(),
    country: z.string(),
  }),
  createdAt: z.date(),
  updatedAt: z.date(),
});

// Type-safe database queries
type DatabaseUser = z.infer<typeof prismaUserSchema>;
type DatabaseProduct = z.infer<typeof mongoProductSchema>;
type DatabaseOrder = z.infer<typeof sqlOrderSchema>;

// Validate database results
function validateDatabaseResult<T extends z.ZodTypeAny>(
  schema: T,
  data: unknown
): z.infer<T> {
  return schema.parse(data);
}

// Example usage
const dbUser = validateDatabaseResult(prismaUserSchema, {
  /* ... raw DB result ... */
});

// Migration data validation
const migrationSchema = z.object({
  version: z.number().int().positive(),
  name: z.string(),
  up: z.function(),
  down: z.function(),
  timestamp: z.date(),
});
```

### 7. Environment Variable Validation

```typescript
// Environment variables schema
const envSchema = z.object({
  // Node environment
  NODE_ENV: z.enum(["development", "production", "test"]).default("development"),
  
  // Server configuration
  PORT: z.coerce.number().positive().default(3000),
  HOST: z.string().default("localhost"),
  
  // Database
  DATABASE_URL: z.string().url(),
  DB_HOST: z.string(),
  DB_PORT: z.coerce.number().positive().default(5432),
  DB_NAME: z.string(),
  DB_USER: z.string(),
  DB_PASSWORD: z.string(),
  
  // Redis
  REDIS_URL: z.string().url().optional(),
  REDIS_HOST: z.string().optional(),
  REDIS_PORT: z.coerce.number().positive().optional(),
  
  // API Keys
  API_KEY: z.string().min(32),
  JWT_SECRET: z.string().min(32),
  ENCRYPTION_KEY: z.string().min(32),
  
  // External services
  AWS_ACCESS_KEY_ID: z.string().optional(),
  AWS_SECRET_ACCESS_KEY: z.string().optional(),
  AWS_REGION: z.string().default("us-east-1"),
  S3_BUCKET: z.string().optional(),
  
  // Email
  SMTP_HOST: z.string().optional(),
  SMTP_PORT: z.coerce.number().positive().optional(),
  SMTP_USER: z.string().optional(),
  SMTP_PASSWORD: z.string().optional(),
  EMAIL_FROM: z.string().email().optional(),
  
  // Feature flags
  ENABLE_ANALYTICS: z.coerce.boolean().default(false),
  ENABLE_CACHING: z.coerce.boolean().default(true),
  ENABLE_RATE_LIMITING: z.coerce.boolean().default(true),
  
  // Logging
  LOG_LEVEL: z.enum(["error", "warn", "info", "debug"]).default("info"),
  LOG_FORMAT: z.enum(["json", "text"]).default("json"),
});

// Validate and export environment variables
export const env = envSchema.parse(process.env);

// Type-safe environment variables
type Env = z.infer<typeof envSchema>;

// Usage
console.log(`Server running on ${env.HOST}:${env.PORT}`);
console.log(`Database: ${env.DATABASE_URL}`);

// Conditional validation based on environment
const productionEnvSchema = envSchema.extend({
  // Additional required fields in production
  SSL_CERT: z.string(),
  SSL_KEY: z.string(),
  SENTRY_DSN: z.string().url(),
});

const developmentEnvSchema = envSchema.extend({
  // Development-specific options
  DEBUG: z.coerce.boolean().default(true),
  HOT_RELOAD: z.coerce.boolean().default(true),
});

function validateEnv() {
  const nodeEnv = process.env.NODE_ENV;
  
  if (nodeEnv === "production") {
    return productionEnvSchema.parse(process.env);
  } else if (nodeEnv === "development") {
    return developmentEnvSchema.parse(process.env);
  }
  
  return envSchema.parse(process.env);
}

export const config = validateEnv();
```

### 8. Testing with Zod

```typescript
// Mock data generation
function generateMockUser(): z.infer<typeof userSchema> {
  return userSchema.parse({
    id: crypto.randomUUID(),
    username: "testuser",
    email: "test@example.com",
    age: 25,
    createdAt: new Date(),
  });
}

// Factory pattern for test data
class UserFactory {
  private defaults: Partial<z.infer<typeof userSchema>> = {
    id: crypto.randomUUID(),
    username: "testuser",
    email: "test@example.com",
    age: 25,
    createdAt: new Date(),
  };

  create(overrides?: Partial<z.infer<typeof userSchema>>): z.infer<typeof userSchema> {
    const data = { ...this.defaults, ...overrides };
    return userSchema.parse(data);
  }

  createMany(count: number): z.infer<typeof userSchema>[] {
    return Array.from({ length: count }, (_, i) =>
      this.create({
        id: crypto.randomUUID(),
        username: `user${i}`,
        email: `user${i}@example.com`,
      })
    );
  }
}

const userFactory = new UserFactory();

// Unit tests
import { describe, it, expect } from "vitest";

describe("User Schema Validation", () => {
  it("should validate correct user data", () => {
    const validUser = {
      id: crypto.randomUUID(),
      username: "johndoe",
      email: "john@example.com",
      age: 30,
      createdAt: new Date(),
    };

    const result = userSchema.safeParse(validUser);
    expect(result.success).toBe(true);
  });

  it("should reject invalid email", () => {
    const invalidUser = {
      id: crypto.randomUUID(),
      username: "johndoe",
      email: "invalid-email",
      age: 30,
      createdAt: new Date(),
    };

    const result = userSchema.safeParse(invalidUser);
    expect(result.success).toBe(false);
    if (!result.success) {
      expect(result.error.issues[0].path).toEqual(["email"]);
    }
  });

  it("should reject negative age", () => {
    const invalidUser = {
      id: crypto.randomUUID(),
      username: "johndoe",
      email: "john@example.com",
      age: -5,
      createdAt: new Date(),
    };

    const result = userSchema.safeParse(invalidUser);
    expect(result.success).toBe(false);
  });
});

// Integration test helpers
function createTestSchema<T extends z.ZodTypeAny>(schema: T) {
  return {
    expectValid(data: unknown) {
      const result = schema.safeParse(data);
      if (!result.success) {
        throw new Error(
          `Expected valid data but got errors: ${JSON.stringify(result.error.issues)}`
        );
      }
      return result.data;
    },

    expectInvalid(data: unknown, expectedErrorPath?: string[]) {
      const result = schema.safeParse(data);
      if (result.success) {
        throw new Error("Expected validation to fail but it succeeded");
      }
      if (expectedErrorPath) {
        const hasExpectedError = result.error.issues.some(
          (issue) => JSON.stringify(issue.path) === JSON.stringify(expectedErrorPath)
        );
        if (!hasExpectedError) {
          throw new Error(
            `Expected error at path ${expectedErrorPath} but got errors at: ${result.error.issues
              .map((i) => JSON.stringify(i.path))
              .join(", ")}`
          );
        }
      }
      return result.error;
    },
  };
}

// Usage in tests
const userTest = createTestSchema(userSchema);

it("validates user creation", () => {
  const user = userTest.expectValid({
    id: crypto.randomUUID(),
    username: "newuser",
    email: "new@example.com",
    age: 25,
    createdAt: new Date(),
  });
  
  expect(user.username).toBe("newuser");
});

it("rejects invalid email", () => {
  const error = userTest.expectInvalid(
    {
      id: crypto.randomUUID(),
      username: "newuser",
      email: "invalid",
      age: 25,
      createdAt: new Date(),
    },
    ["email"]
  );
  
  expect(error.issues).toHaveLength(1);
});
```

### 9. Performance Optimization

```typescript
// Schema caching
const schemaCache = new Map<string, z.ZodTypeAny>();

function getCachedSchema(key: string, factory: () => z.ZodTypeAny): z.ZodTypeAny {
  if (!schemaCache.has(key)) {
    schemaCache.set(key, factory());
  }
  return schemaCache.get(key)!;
}

// Usage
const userSchema = getCachedSchema("user", () =>
  z.object({
    id: z.string().uuid(),
    username: z.string(),
    email: z.string().email(),
  })
);

// Lazy schema evaluation
const expensiveSchema = z.lazy(() => {
  // Only computed when first used
  return z.object({
    // ... complex schema definition
  });
});

// Batch validation
function validateBatch<T extends z.ZodTypeAny>(
  schema: T,
  items: unknown[]
): { valid: z.infer<T>[]; invalid: { item: unknown; error: z.ZodError }[] } {
  const valid: z.infer<T>[] = [];
  const invalid: { item: unknown; error: z.ZodError }[] = [];

  for (const item of items) {
    const result = schema.safeParse(item);
    if (result.success) {
      valid.push(result.data);
    } else {
      invalid.push({ item, error: result.error });
    }
  }

  return { valid, invalid };
}

// Streaming validation for large datasets
async function* validateStream<T extends z.ZodTypeAny>(
  schema: T,
  items: AsyncIterable<unknown>
) {
  for await (const item of items) {
    const result = schema.safeParse(item);
    yield result;
  }
}

// Usage
async function processLargeDataset() {
  const dataStream = fetchDataStream(); // Returns AsyncIterable
  
  for await (const result of validateStream(userSchema, dataStream)) {
    if (result.success) {
      await processUser(result.data);
    } else {
      logError(result.error);
    }
  }
}

// Partial validation for performance
const fullUserSchema = z.object({
  id: z.string().uuid(),
  username: z.string(),
  email: z.string().email(),
  profile: z.object({
    bio: z.string(),
    avatar: z.string().url(),
    // ... many more fields
  }),
  // ... many more fields
});

// Only validate what you need
const quickCheckSchema = fullUserSchema.pick({
  id: true,
  username: true,
});

// Benchmark validation performance
function benchmarkValidation<T extends z.ZodTypeAny>(
  schema: T,
  data: unknown,
  iterations: number = 1000
) {
  const start = performance.now();
  
  for (let i = 0; i < iterations; i++) {
    schema.safeParse(data);
  }
  
  const end = performance.now();
  const totalTime = end - start;
  const avgTime = totalTime / iterations;
  
  console.log(`Total time: ${totalTime.toFixed(2)}ms`);
  console.log(`Average time per validation: ${avgTime.toFixed(4)}ms`);
  console.log(`Validations per second: ${(1000 / avgTime).toFixed(0)}`);
}
```

### 10. Error Handling Patterns

```typescript
// Custom error class
class ValidationError extends Error {
  constructor(
    message: string,
    public zodError: z.ZodError
  ) {
    super(message);
    this.name = "ValidationError";
  }

  getErrors() {
    return this.zodError.issues.map((issue) => ({
      path: issue.path.join("."),
      message: issue.message,
    }));
  }

  toJSON() {
    return {
      name: this.name,
      message: this.message,
      errors: this.getErrors(),
    };
  }
}

// Error handler
function handleValidationError(error: z.ZodError): ValidationError {
  return new ValidationError("Validation failed", error);
}

// Result type pattern
type Result<T, E = Error> =
  | { success: true; data: T }
  | { success: false; error: E };

function validateSafely<T extends z.ZodTypeAny>(
  schema: T,
  data: unknown
): Result<z.infer<T>, ValidationError> {
  const result = schema.safeParse(data);
  
  if (result.success) {
    return { success: true, data: result.data };
  }
  
  return {
    success: false,
    error: new ValidationError("Validation failed", result.error),
  };
}

// Usage
const result = validateSafely(userSchema, userData);

if (result.success) {
  console.log("User:", result.data);
} else {
  console.error("Validation errors:", result.error.getErrors());
}

// Error aggregation
class ValidationErrorAggregator {
  private errors: ValidationError[] = [];

  add(error: ValidationError) {
    this.errors.push(error);
  }

  hasErrors(): boolean {
    return this.errors.length > 0;
  }

  getAllErrors() {
    return this.errors.flatMap((e) => e.getErrors());
  }

  throw() {
    if (this.hasErrors()) {
      throw new ValidationError(
        `Multiple validation errors occurred`,
        new z.ZodError(
          this.errors.flatMap((e) => e.zodError.issues)
        )
      );
    }
  }
}

// Usage
const aggregator = new ValidationErrorAggregator();

for (const item of items) {
  const result = userSchema.safeParse(item);
  if (!result.success) {
    aggregator.add(new ValidationError("Invalid user", result.error));
  }
}

aggregator.throw(); // Throws if any errors accumulated

// Localized error messages
function createLocalizedErrors(zodError: z.ZodError, locale: string) {
  const translations: Record<string, Record<string, string>> = {
    en: {
      required: "This field is required",
      invalidEmail: "Invalid email address",
      tooShort: "Too short",
      tooLong: "Too long",
    },
    es: {
      required: "Este campo es obligatorio",
      invalidEmail: "Dirección de correo electrónico no válida",
      tooShort: "Demasiado corto",
      tooLong: "Demasiado largo",
    },
  };

  return zodError.issues.map((issue) => {
    let messageKey = "unknown";
    
    if (issue.code === "invalid_type" && issue.received === "undefined") {
      messageKey = "required";
    } else if (issue.code === "invalid_string" && "validation" in issue && issue.validation === "email") {
      messageKey = "invalidEmail";
    } else if (issue.code === "too_small") {
      messageKey = "tooShort";
    } else if (issue.code === "too_big") {
      messageKey = "tooLong";
    }

    return {
      path: issue.path.join("."),
      message: translations[locale]?.[messageKey] || issue.message,
    };
  });
}
```

---

## Expert/Ninja Level

### 1. Building Type-Safe APIs

```typescript
// Contract-first API design
const apiContract = {
  // User endpoints
  createUser: {
    method: "POST" as const,
    path: "/users",
    request: z.object({
      username: z.string().min(3),
      email: z.string().email(),
      password: z.string().min(8),
    }),
    response: z.object({
      id: z.string().uuid(),
      username: z.string(),
      email: z.string().email(),
      createdAt: z.date(),
    }),
  },
  
  getUser: {
    method: "GET" as const,
    path: "/users/:id",
    params: z.object({
      id: z.string().uuid(),
    }),
    response: z.object({
      id: z.string().uuid(),
      username: z.string(),
      email: z.string().email(),
      createdAt: z.date(),
    }),
  },
  
  updateUser: {
    method: "PUT" as const,
    path: "/users/:id",
    params: z.object({
      id: z.string().uuid(),
    }),
    request: z.object({
      username: z.string().min(3).optional(),
      email: z.string().email().optional(),
    }),
    response: z.object({
      id: z.string().uuid(),
      username: z.string(),
      email: z.string().email(),
      updatedAt: z.date(),
    }),
  },
};

// Type-safe API client
class TypeSafeApiClient {
  constructor(private baseUrl: string) {}

  async createUser(
    data: z.infer<typeof apiContract.createUser.request>
  ): Promise<z.infer<typeof apiContract.createUser.response>> {
    const response = await fetch(`${this.baseUrl}${apiContract.createUser.path}`, {
      method: apiContract.createUser.method,
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(data),
    });

    const json = await response.json();
    return apiContract.createUser.response.parse(json);
  }

  async getUser(
    params: z.infer<typeof apiContract.getUser.params>
  ): Promise<z.infer<typeof apiContract.getUser.response>> {
    const path = apiContract.getUser.path.replace(":id", params.id);
    const response = await fetch(`${this.baseUrl}${path}`);
    const json = await response.json();
    return apiContract.getUser.response.parse(json);
  }
}

// Generic API client factory
function createApiClient<T extends Record<string, any>>(contract: T, baseUrl: string) {
  const client: any = {};

  for (const [name, endpoint] of Object.entries(contract)) {
    client[name] = async (data: any) => {
      let path = endpoint.path;
      
      // Replace path parameters
      if (endpoint.params && data.params) {
        for (const [key, value] of Object.entries(data.params)) {
          path = path.replace(`:${key}`, value as string);
        }
      }

      const options: RequestInit = {
        method: endpoint.method,
      };

      if (endpoint.request && data.body) {
        options.headers = { "Content-Type": "application/json" };
        options.body = JSON.stringify(endpoint.request.parse(data.body));
      }

      const response = await fetch(`${baseUrl}${path}`, options);
      const json = await response.json();
      
      return endpoint.response.parse(json);
    };
  }

  return client;
}
```

### 2. Advanced Type Inference

```typescript
// Extract schema types
type InferInput<T extends z.ZodTypeAny> = z.input<T>;
type InferOutput<T extends z.ZodTypeAny> = z.output<T>;

// Schema with transformations
const transformSchema = z.object({
  name: z.string().transform((s) => s.toUpperCase()),
  age: z.string().transform((s) => parseInt(s)),
});

type Input = InferInput<typeof transformSchema>;
// { name: string; age: string }

type Output = InferOutput<typeof transformSchema>;
// { name: string; age: number }

// Conditional types based on schema
type IsOptional<T extends z.ZodTypeAny> = T extends z.ZodOptional<any> ? true : false;
type IsNullable<T extends z.ZodTypeAny> = T extends z.ZodNullable<any> ? true : false;

// Deep partial type
type DeepPartial<T> = T extends object
  ? { [P in keyof T]?: DeepPartial<T[P]> }
  : T;

function createDeepPartialSchema<T extends z.ZodTypeAny>(schema: T): z.ZodType<DeepPartial<z.infer<T>>> {
  if (schema instanceof z.ZodObject) {
    const shape: any = {};
    for (const [key, value] of Object.entries(schema.shape)) {
      shape[key] = createDeepPartialSchema(value as z.ZodTypeAny).optional();
    }
    return z.object(shape) as any;
  }
  
  return schema.optional() as any;
}

// Usage
const userSchema = z.object({
  id: z.string(),
  profile: z.object({
    name: z.string(),
    age: z.number(),
  }),
});

const deepPartialUserSchema = createDeepPartialSchema(userSchema);
type DeepPartialUser = z.infer<typeof deepPartialUserSchema>;
// {
//   id?: string;
//   profile?: {
//     name?: string;
//     age?: number;
//   };
// }

// Type-level schema manipulation
type ExtractShape<T extends z.ZodObject<any>> = T extends z.ZodObject<infer S> ? S : never;

type PickSchema<T extends z.ZodObject<any>, K extends keyof ExtractShape<T>> = z.ZodObject<
  Pick<ExtractShape<T>, K>
>;

type OmitSchema<T extends z.ZodObject<any>, K extends keyof ExtractShape<T>> = z.ZodObject<
  Omit<ExtractShape<T>, K>
>;
```

### 3. Plugin System

```typescript
// Zod plugin interface
interface ZodPlugin<T extends z.ZodTypeAny = z.ZodTypeAny> {
  name: string;
  transform?: (schema: T) => z.ZodTypeAny;
  validate?: (data: unknown, schema: T) => void;
  preprocess?: (data: unknown) => unknown;
  postprocess?: (data: z.infer<T>) => z.infer<T>;
}

// Plugin manager
class ZodPluginManager {
  private plugins: Map<string, ZodPlugin> = new Map();

  register(plugin: ZodPlugin) {
    this.plugins.set(plugin.name, plugin);
  }

  apply<T extends z.ZodTypeAny>(schema: T, pluginNames: string[]): z.ZodTypeAny {
    let result: z.ZodTypeAny = schema;

    for (const name of pluginNames) {
      const plugin = this.plugins.get(name);
      if (!plugin) continue;

      // Apply transformations
      if (plugin.transform) {
        result = plugin.transform(result);
      }

      // Add preprocessing
      if (plugin.preprocess) {
        const preprocess = plugin.preprocess;
        result = z.preprocess(preprocess, result);
      }
    }

    return result;
  }
}

// Example plugins
const trimStringsPlugin: ZodPlugin = {
  name: "trimStrings",
  preprocess: (data) => {
    if (typeof data === "string") {
      return data.trim();
    }
    if (typeof data === "object" && data !== null) {
      const result: any = Array.isArray(data) ? [] : {};
      for (const [key, value] of Object.entries(data)) {
        result[key] = typeof value === "string" ? value.trim() : value;
      }
      return result;
    }
    return data;
  },
};

const removeEmptyStringsPlugin: ZodPlugin = {
  name: "removeEmptyStrings",
  preprocess: (data) => {
    if (typeof data === "object" && data !== null) {
      const result: any = {};
      for (const [key, value] of Object.entries(data)) {
        if (value !== "") {
          result[key] = value;
        }
      }
      return result;
    }
    return data;
  },
};

const auditPlugin: ZodPlugin<z.ZodObject<any>> = {
  name: "audit",
  transform: (schema) => {
    return schema.extend({
      createdAt: z.date().default(() => new Date()),
      updatedAt: z.date().default(() => new Date()),
    }) as any;
  },
};

// Usage
const pluginManager = new ZodPluginManager();
pluginManager.register(trimStringsPlugin);
pluginManager.register(removeEmptyStringsPlugin);
pluginManager.register(auditPlugin);

const baseSchema = z.object({
  name: z.string(),
  email: z.string().email(),
});

const enhancedSchema = pluginManager.apply(baseSchema, [
  "trimStrings",
  "removeEmptyStrings",
  "audit",
]);
```

### 4. Custom Schema Types

```typescript
// Custom branded types
const brandedString = <T extends string>(brand: T) => {
  return z.string().transform((val) => val as string & { __brand: T });
};

type UserId = string & { __brand: "UserId" };
type Email = string & { __brand: "Email" };

const userIdSchema = brandedString("UserId");
const emailSchema = z.string().email().transform((val) => val as Email);

// Custom validators
function createPhoneNumberSchema(countryCode: string) {
  const patterns: Record<string, RegExp> = {
    US: /^\+1\d{10}$/,
    UK: /^\+44\d{10}$/,
    IN: /^\+91\d{10}$/,
  };

  const pattern = patterns[countryCode];
  if (!pattern) {
    throw new Error(`Unknown country code: ${countryCode}`);
  }

  return z.string().regex(pattern, {
    message: `Invalid ${countryCode} phone number`,
  });
}

const usPhoneSchema = createPhoneNumberSchema("US");
const ukPhoneSchema = createPhoneNumberSchema("UK");

// Custom refinement helpers
function createRangeSchema(min: number, max: number) {
  return z.number().refine((val) => val >= min && val <= max, {
    message: `Value must be between ${min} and ${max}`,
  });
}

const percentageSchema = createRangeSchema(0, 100);
const ratingSchema = createRangeSchema(1, 5);

// Temporal validations
const futureDateSchema = z.date().refine(
  (date) => date > new Date(),
  { message: "Date must be in the future" }
);

const pastDateSchema = z.date().refine(
  (date) => date < new Date(),
  { message: "Date must be in the past" }
);

const businessHoursSchema = z.date().refine(
  (date) => {
    const hour = date.getHours();
    const day = date.getDay();
    return day >= 1 && day <= 5 && hour >= 9 && hour < 17;
  },
  { message: "Must be during business hours (Mon-Fri, 9AM-5PM)" }
);

// Financial validators
const currencySchema = z.object({
  amount: z.number().positive(),
  currency: z.enum(["USD", "EUR", "GBP", "JPY"]),
}).transform((val) => ({
  ...val,
  // Normalize to 2 decimal places
  amount: Math.round(val.amount * 100) / 100,
}));

const creditCardSchema = z
  .string()
  .regex(/^\d{13,19}$/)
  .refine((val) => {
    // Luhn algorithm validation
    let sum = 0;
    let isEven = false;
    
    for (let i = val.length - 1; i >= 0; i--) {
      let digit = parseInt(val[i]);
      
      if (isEven) {
        digit *= 2;
        if (digit > 9) {
          digit -= 9;
        }
      }
      
      sum += digit;
      isEven = !isEven;
    }
    
    return sum % 10 === 0;
  }, { message: "Invalid credit card number" });

// Geographic validators
const coordinatesSchema = z.object({
  latitude: z.number().min(-90).max(90),
  longitude: z.number().min(-180).max(180),
});

const ipAddressSchema = z.union([
  // IPv4
  z.string().regex(
    /^((25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.){3}(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)$/
  ),
  // IPv6
  z.string().regex(
    /^(([0-9a-fA-F]{1,4}:){7}[0-9a-fA-F]{1,4}|::)$/
  ),
]);
```

### 5. Schema Migrations

```typescript
// Version 1 schema
const userSchemaV1 = z.object({
  id: z.string(),
  name: z.string(),
  email: z.string(),
});

// Version 2 schema (split name into firstName/lastName)
const userSchemaV2 = z.object({
  id: z.string().uuid(),
  firstName: z.string(),
  lastName: z.string(),
  email: z.string().email(),
});

// Version 3 schema (add timestamps)
const userSchemaV3 = z.object({
  id: z.string().uuid(),
  firstName: z.string(),
  lastName: z.string(),
  email: z.string().email(),
  createdAt: z.date(),
  updatedAt: z.date(),
});

// Migration functions
function migrateV1ToV2(dataV1: z.infer<typeof userSchemaV1>): z.infer<typeof userSchemaV2> {
  const [firstName, lastName] = dataV1.name.split(" ");
  return {
    id: dataV1.id,
    firstName: firstName || dataV1.name,
    lastName: lastName || "",
    email: dataV1.email,
  };
}

function migrateV2ToV3(dataV2: z.infer<typeof userSchemaV2>): z.infer<typeof userSchemaV3> {
  return {
    ...dataV2,
    createdAt: new Date(),
    updatedAt: new Date(),
  };
}

// Migration manager
class SchemaMigrationManager<T> {
  private migrations: Map<string, (data: any) => any> = new Map();
  private schemas: Map<string, z.ZodTypeAny> = new Map();

  registerSchema(version: string, schema: z.ZodTypeAny) {
    this.schemas.set(version, schema);
  }

  registerMigration(fromVersion: string, toVersion: string, migrate: (data: any) => any) {
    this.migrations.set(`${fromVersion}->${toVersion}`, migrate);
  }

  migrate(data: any, fromVersion: string, toVersion: string): T {
    const fromSchema = this.schemas.get(fromVersion);
    if (!fromSchema) {
      throw new Error(`Schema version ${fromVersion} not found`);
    }

    // Validate input with source schema
    const validatedData = fromSchema.parse(data);

    // Apply migration
    const migrationKey = `${fromVersion}->${toVersion}`;
    const migration = this.migrations.get(migrationKey);
    
    if (!migration) {
      throw new Error(`Migration ${migrationKey} not found`);
    }

    const migratedData = migration(validatedData);

    // Validate output with target schema
    const toSchema = this.schemas.get(toVersion);
    if (!toSchema) {
      throw new Error(`Schema version ${toVersion} not found`);
    }

    return toSchema.parse(migratedData);
  }

  migrateToLatest(data: any, fromVersion: string): T {
    // Get all version numbers
    const versions = Array.from(this.schemas.keys()).sort();
    const fromIndex = versions.indexOf(fromVersion);
    
    if (fromIndex === -1) {
      throw new Error(`Unknown version: ${fromVersion}`);
    }

    let currentData = data;
    let currentVersion = fromVersion;

    // Apply migrations sequentially
    for (let i = fromIndex; i < versions.length - 1; i++) {
      const nextVersion = versions[i + 1];
      currentData = this.migrate(currentData, currentVersion, nextVersion);
      currentVersion = nextVersion;
    }

    return currentData;
  }
}

// Usage
const migrationManager = new SchemaMigrationManager();

migrationManager.registerSchema("v1", userSchemaV1);
migrationManager.registerSchema("v2", userSchemaV2);
migrationManager.registerSchema("v3", userSchemaV3);

migrationManager.registerMigration("v1", "v2", migrateV1ToV2);
migrationManager.registerMigration("v2", "v3", migrateV2ToV3);

// Migrate old data
const oldData = {
  id: "123",
  name: "John Doe",
  email: "john@example.com",
};

const latestData = migrationManager.migrateToLatest(oldData, "v1");
console.log(latestData);
```

### 6. Code Generation

```typescript
// Generate TypeScript types from schema
function generateTypeScriptType(schema: z.ZodTypeAny, name: string): string {
  if (schema instanceof z.ZodObject) {
    const shape = schema.shape;
    const fields = Object.entries(shape)
      .map(([key, value]) => {
        const fieldType = getTypeScriptType(value as z.ZodTypeAny);
        const optional = value instanceof z.ZodOptional ? "?" : "";
        return `  ${key}${optional}: ${fieldType};`;
      })
      .join("\n");
    
    return `export interface ${name} {\n${fields}\n}`;
  }
  
  return `export type ${name} = ${getTypeScriptType(schema)};`;
}

function getTypeScriptType(schema: z.ZodTypeAny): string {
  if (schema instanceof z.ZodString) return "string";
  if (schema instanceof z.ZodNumber) return "number";
  if (schema instanceof z.ZodBoolean) return "boolean";
  if (schema instanceof z.ZodDate) return "Date";
  if (schema instanceof z.ZodArray) {
    return `Array<${getTypeScriptType(schema.element)}>`;
  }
  if (schema instanceof z.ZodOptional) {
    return `${getTypeScriptType(schema.unwrap())} | undefined`;
  }
  if (schema instanceof z.ZodNullable) {
    return `${getTypeScriptType(schema.unwrap())} | null`;
  }
  return "unknown";
}

// Generate validation functions
function generateValidationFunction(schema: z.ZodTypeAny, name: string): string {
  return `
export function validate${name}(data: unknown): ${name} {
  return ${name}Schema.parse(data);
}

export function validate${name}Safe(data: unknown): z.SafeParseReturnType<unknown, ${name}> {
  return ${name}Schema.safeParse(data);
}
`;
}

// Generate JSON Schema
function zodToJsonSchema(schema: z.ZodTypeAny): any {
  if (schema instanceof z.ZodString) {
    return { type: "string" };
  }
  if (schema instanceof z.ZodNumber) {
    return { type: "number" };
  }
  if (schema instanceof z.ZodBoolean) {
    return { type: "boolean" };
  }
  if (schema instanceof z.ZodObject) {
    const shape = schema.shape;
    const properties: any = {};
    const required: string[] = [];
    
    for (const [key, value] of Object.entries(shape)) {
      properties[key] = zodToJsonSchema(value as z.ZodTypeAny);
      if (!(value instanceof z.ZodOptional)) {
        required.push(key);
      }
    }
    
    return {
      type: "object",
      properties,
      required,
    };
  }
  if (schema instanceof z.ZodArray) {
    return {
      type: "array",
      items: zodToJsonSchema(schema.element),
    };
  }
  return {};
}

// Usage
const userSchema = z.object({
  id: z.string().uuid(),
  username: z.string(),
  email: z.string().email(),
  age: z.number().optional(),
});

const typeDefinition = generateTypeScriptType(userSchema, "User");
console.log(typeDefinition);
// export interface User {
//   id: string;
//   username: string;
//   email: string;
//   age?: number;
// }

const validationFunctions = generateValidationFunction(userSchema, "User");
console.log(validationFunctions);

const jsonSchema = zodToJsonSchema(userSchema);
console.log(JSON.stringify(jsonSchema, null, 2));
```

---

## Real-World Use Cases

### 1. E-Commerce Product Catalog

```typescript
// Product schema with variants
const productVariantSchema = z.object({
  id: z.string().uuid(),
  sku: z.string(),
  name: z.string(),
  price: z.number().positive(),
  compareAtPrice: z.number().positive().optional(),
  inventory: z.number().int().nonnegative(),
  attributes: z.record(z.string()),
  images: z.array(z.string().url()).min(1),
});

const productSchema = z.object({
  id: z.string().uuid(),
  name: z.string().min(1).max(200),
  description: z.string().max(5000),
  category: z.string(),
  brand: z.string(),
  tags: z.array(z.string()).max(20),
  variants: z.array(productVariantSchema).min(1),
  specifications: z.record(z.string()),
  seo: z.object({
    title: z.string().max(60),
    description: z.string().max(160),
    keywords: z.array(z.string()),
  }),
  status: z.enum(["draft", "active", "archived"]),
  createdAt: z.date(),
  updatedAt: z.date(),
});

type Product = z.infer<typeof productSchema>;
```

### 2. User Authentication System

```typescript
// Registration
const registrationSchema = z
  .object({
    email: z.string().email(),
    password: z
      .string()
      .min(8)
      .regex(/[A-Z]/, "Must contain uppercase")
      .regex(/[a-z]/, "Must contain lowercase")
      .regex(/[0-9]/, "Must contain number")
      .regex(/[^A-Za-z0-9]/, "Must contain special character"),
    confirmPassword: z.string(),
    firstName: z.string().min(2),
    lastName: z.string().min(2),
    acceptTerms: z.literal(true),
  })
  .refine((data) => data.password === data.confirmPassword, {
    message: "Passwords don't match",
    path: ["confirmPassword"],
  });

// Login
const loginSchema = z.object({
  email: z.string().email(),
  password: z.string(),
  rememberMe: z.boolean().default(false),
});

// Password reset
const passwordResetRequestSchema = z.object({
  email: z.string().email(),
});

const passwordResetSchema = z
  .object({
    token: z.string().uuid(),
    password: z.string().min(8),
    confirmPassword: z.string(),
  })
  .refine((data) => data.password === data.confirmPassword, {
    message: "Passwords don't match",
    path: ["confirmPassword"],
  });

// Two-factor authentication
const twoFactorSetupSchema = z.object({
  method: z.enum(["sms", "email", "authenticator"]),
  phoneNumber: z.string().optional(),
});

const twoFactorVerifySchema = z.object({
  code: z.string().length(6).regex(/^\d{6}$/),
});
```

### 3. Blog/CMS System

```typescript
// Blog post schema
const blogPostSchema = z.object({
  id: z.string().uuid(),
  title: z.string().min(1).max(200),
  slug: z
    .string()
    .regex(/^[a-z0-9-]+$/, "Slug must be lowercase with hyphens")
    .min(1)
    .max(200),
  excerpt: z.string().max(300).optional(),
  content: z.string().min(1),
  coverImage: z.string().url().optional(),
  author: z.object({
    id: z.string().uuid(),
    name: z.string(),
    avatar: z.string().url().optional(),
  }),
  categories: z.array(z.string()).min(1).max(5),
  tags: z.array(z.string()).max(20),
  status: z.enum(["draft", "published", "scheduled", "archived"]),
  publishedAt: z.date().nullable(),
  scheduledFor: z.date().nullable(),
  seo: z.object({
    title: z.string().max(60),
    description: z.string().max(160),
    ogImage: z.string().url().optional(),
    keywords: z.array(z.string()),
  }),
  readingTime: z.number().positive(),
  views: z.number().nonnegative().default(0),
  likes: z.number().nonnegative().default(0),
  createdAt: z.date(),
  updatedAt: z.date(),
});

// Comment schema
const commentSchema = z.object({
  id: z.string().uuid(),
  postId: z.string().uuid(),
  author: z.object({
    id: z.string().uuid(),
    name: z.string(),
    avatar: z.string().url().optional(),
  }),
  content: z.string().min(1).max(1000),
  parentId: z.string().uuid().nullable(),
  status: z.enum(["pending", "approved", "spam", "deleted"]),
  createdAt: z.date(),
  updatedAt: z.date(),
});
```

### 4. Payment Processing

```typescript
// Payment intent schema
const paymentIntentSchema = z.object({
  id: z.string().uuid(),
  amount: z.number().positive(),
  currency: z.enum(["USD", "EUR", "GBP", "JPY"]),
  customerId: z.string().uuid(),
  paymentMethod: z.discriminatedUnion("type", [
    z.object({
      type: z.literal("credit_card"),
      cardNumber: z.string().regex(/^\d{16}$/),
      expiryMonth: z.number().int().min(1).max(12),
      expiryYear: z.number().int().min(new Date().getFullYear()),
      cvv: z.string().regex(/^\d{3,4}$/),
      billingAddress: z.object({
        street: z.string(),
        city: z.string(),
        state: z.string(),
        zipCode: z.string(),
        country: z.string().length(2),
      }),
    }),
    z.object({
      type: z.literal("bank_account"),
      accountNumber: z.string(),
      routingNumber: z.string(),
      accountType: z.enum(["checking", "savings"]),
    }),
    z.object({
      type: z.literal("digital_wallet"),
      provider: z.enum(["paypal", "apple_pay", "google_pay"]),
      token: z.string(),
    }),
  ]),
  metadata: z.record(z.string()).optional(),
  description: z.string().optional(),
  status: z.enum(["pending", "processing", "succeeded", "failed", "canceled"]),
  createdAt: z.date(),
});

// Refund schema
const refundSchema = z.object({
  id: z.string().uuid(),
  paymentIntentId: z.string().uuid(),
  amount: z.number().positive(),
  reason: z.enum(["duplicate", "fraudulent", "requested_by_customer"]),
  notes: z.string().optional(),
  status: z.enum(["pending", "succeeded", "failed"]),
  createdAt: z.date(),
});
```

### 5. Real-Time Chat Application

```typescript
// Chat message schema
const chatMessageSchema = z.object({
  id: z.string().uuid(),
  conversationId: z.string().uuid(),
  senderId: z.string().uuid(),
  type: z.enum(["text", "image", "file", "audio", "video"]),
  content: z.string(),
  attachments: z
    .array(
      z.object({
        id: z.string().uuid(),
        type: z.string(),
        url: z.string().url(),
        size: z.number().positive(),
        name: z.string(),
      })
    )
    .optional(),
  metadata: z
    .object({
      mentions: z.array(z.string().uuid()).optional(),
      replyTo: z.string().uuid().optional(),
      edited: z.boolean().default(false),
      editedAt: z.date().optional(),
    })
    .optional(),
  status: z.enum(["sending", "sent", "delivered", "read", "failed"]),
  createdAt: z.date(),
});

// Conversation schema
const conversationSchema = z.object({
  id: z.string().uuid(),
  type: z.enum(["direct", "group", "channel"]),
  name: z.string().optional(),
  avatar: z.string().url().optional(),
  participants: z.array(
    z.object({
      userId: z.string().uuid(),
      role: z.enum(["owner", "admin", "member"]),
      joinedAt: z.date(),
      lastReadAt: z.date().optional(),
    })
  ),
  lastMessage: chatMessageSchema.optional(),
  unreadCount: z.number().nonnegative().default(0),
  createdAt: z.date(),
  updatedAt: z.date(),
});
```

---

## Best Practices

### 1. Schema Organization

```typescript
// ❌ Bad: All schemas in one file
// schemas.ts - 5000+ lines

// ✅ Good: Organize by domain
// schemas/
//   user/
//     user.schema.ts
//     profile.schema.ts
//     auth.schema.ts
//   product/
//     product.schema.ts
//     category.schema.ts
//   order/
//     order.schema.ts
//     payment.schema.ts

// ✅ Good: Shared/base schemas
// schemas/base/
//   timestamp.schema.ts
//   pagination.schema.ts
//   audit.schema.ts

export const timestampSchema = z.object({
  createdAt: z.date(),
  updatedAt: z.date(),
});

export const paginationSchema = z.object({
  page: z.number().positive().int().default(1),
  limit: z.number().positive().int().max(100).default(10),
});
```

### 2. Reusability

```typescript
// ✅ Good: Create reusable schema factories
function createTimestampedSchema<T extends z.ZodRawShape>(shape: T) {
  return z.object(shape).merge(timestampSchema);
}

function createPageable<T extends z.ZodTypeAny>(dataSchema: T) {
  return z.object({
    data: z.array(dataSchema),
    pagination: z.object({
      page: z.number().int().positive(),
      limit: z.number().int().positive(),
      total: z.number().int().nonnegative(),
      totalPages: z.number().int().positive(),
    }),
  });
}

// Usage
const userSchema = createTimestampedSchema({
  id: z.string().uuid(),
  username: z.string(),
  email: z.string().email(),
});

const paginatedUsers = createPageable(userSchema);
```

### 3. Error Messages

```typescript
// ❌ Bad: Generic error messages
const badSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
});

// ✅ Good: Specific, helpful error messages
const goodSchema = z.object({
  email: z.string().email("Please enter a valid email address"),
  password: z
    .string()
    .min(8, "Password must be at least 8 characters long")
    .regex(/[A-Z]/, "Password must contain at least one uppercase letter")
    .regex(/[a-z]/, "Password must contain at least one lowercase letter")
    .regex(/[0-9]/, "Password must contain at least one number"),
});
```

### 4. Performance Considerations

```typescript
// ✅ Good: Cache schemas
const schemas = new Map<string, z.ZodTypeAny>();

function getSchema(name: string): z.ZodTypeAny {
  if (!schemas.has(name)) {
    // Create schema only once
    const schema = createSchema(name);
    schemas.set(name, schema);
  }
  return schemas.get(name)!;
}

// ✅ Good: Use lazy evaluation for expensive schemas
const expensiveSchema = z.lazy(() => {
  // Only created when first accessed
  return createComplexSchema();
});

// ✅ Good: Avoid unnecessary validations
const quickValidation = fullSchema.pick({ id: true, status: true });
```

### 5. Type Safety

```typescript
// ✅ Good: Always infer types from schemas
const userSchema = z.object({
  id: z.string().uuid(),
  username: z.string(),
});

type User = z.infer<typeof userSchema>;

// ❌ Bad: Defining types separately
interface User {
  id: string;
  username: string;
}

const userSchema = z.object({
  id: z.string().uuid(),
  username: z.string(),
});

// ✅ Good: Use const assertions for enum-like values
const USER_ROLES = ["admin", "user", "guest"] as const;
const userRoleSchema = z.enum(USER_ROLES);
type UserRole = z.infer<typeof userRoleSchema>;
```

### 6. Testing

```typescript
// ✅ Good: Test schemas thoroughly
describe("UserSchema", () => {
  it("accepts valid data", () => {
    const valid = {
      id: crypto.randomUUID(),
      username: "johndoe",
      email: "john@example.com",
    };
    expect(userSchema.safeParse(valid).success).toBe(true);
  });

  it("rejects invalid email", () => {
    const invalid = {
      id: crypto.randomUUID(),
      username: "johndoe",
      email: "not-an-email",
    };
    const result = userSchema.safeParse(invalid);
    expect(result.success).toBe(false);
    if (!result.success) {
      expect(result.error.issues[0].path).toEqual(["email"]);
    }
  });

  // Test edge cases
  it("handles empty strings", () => {
    const invalid = {
      id: crypto.randomUUID(),
      username: "",
      email: "john@example.com",
    };
    expect(userSchema.safeParse(invalid).success).toBe(false);
  });

  // Test transformations
  it("trims whitespace", () => {
    const data = {
      id: crypto.randomUUID(),
      username: "  johndoe  ",
      email: "john@example.com",
    };
    const result = userSchemaWithTrim.parse(data);
    expect(result.username).toBe("johndoe");
  });
});
```

---

## Performance Optimization

### Benchmarking

```typescript
import { z } from "zod";

// Simple benchmark utility
function benchmark(name: string, fn: () => void, iterations: number = 10000) {
  const start = performance.now();
  for (let i = 0; i < iterations; i++) {
    fn();
  }
  const end = performance.now();
  const totalTime = end - start;
  console.log(`${name}: ${totalTime.toFixed(2)}ms (${(totalTime / iterations).toFixed(4)}ms per iteration)`);
}

// Benchmark validation
const testData = {
  id: crypto.randomUUID(),
  username: "testuser",
  email: "test@example.com",
};

benchmark("Simple validation", () => {
  userSchema.parse(testData);
});

benchmark("Safe parse", () => {
  userSchema.safeParse(testData);
});

benchmark("With transformation", () => {
  userSchemaWithTransform.parse(testData);
});
```

### Optimization Strategies

```typescript
// 1. Avoid unnecessary refinements
// ❌ Slow
const slowSchema = z.string().refine((val) => val.length > 3).refine((val) => val.length < 20);

// ✅ Fast
const fastSchema = z.string().min(4).max(19);

// 2. Use lazy schemas for expensive operations
const lazySchema = z.lazy(() => {
  return expensiveSchemaCreation();
});

// 3. Cache parsed results when possible
const resultCache = new Map<string, any>();

function cachedParse<T extends z.ZodTypeAny>(schema: T, data: unknown, key: string) {
  if (resultCache.has(key)) {
    return resultCache.get(key);
  }
  const result = schema.parse(data);
  resultCache.set(key, result);
  return result;
}

// 4. Use discriminated unions for better performance
// ✅ Fast - Zod can quickly determine which schema to use
const fastUnion = z.discriminatedUnion("type", [
  z.object({ type: z.literal("a"), valueA: z.string() }),
  z.object({ type: z.literal("b"), valueB: z.number() }),
]);

// ❌ Slower - Zod must try each schema
const slowUnion = z.union([
  z.object({ type: z.literal("a"), valueA: z.string() }),
  z.object({ type: z.literal("b"), valueB: z.number() }),
]);

// 5. Minimize async validations
// Only use async when necessary (e.g., database checks)
const syncSchema = z.string().email(); // Fast
const asyncSchema = z.string().refine(async (val) => {
  return await checkDatabase(val); // Slow
});
```

---

## Conclusion

### Key Takeaways

1. **Type Safety First**: Zod provides runtime validation with compile-time type inference, giving you the best of both worlds.

2. **Composability**: Build complex schemas from simple primitives using composition patterns.

3. **Flexibility**: Transform, refine, and customize schemas to fit your exact needs.

4. **Error Handling**: Comprehensive error reporting helps users understand validation failures.

5. **Performance**: Use optimization techniques for production applications with high-throughput requirements.

6. **Real-World Ready**: Zod handles everything from simple form validation to complex API contracts.

### When to Use Zod

✅ **Use Zod when:**
- Building TypeScript applications
- Validating API requests/responses
- Parsing external data (user input, API responses, file uploads)
- Ensuring data integrity at runtime
- You need automatic type inference
- Building type-safe APIs

❌ **Consider alternatives when:**
- Working with pure JavaScript (consider alternatives like Joi or Yup)
- Need visual schema builders (consider JSON Schema)
- Performance is absolutely critical (consider lighter alternatives)
- Working with very old TypeScript versions

### Further Learning

**Official Resources:**
- [Zod Documentation](https://zod.dev)
- [Zod GitHub Repository](https://github.com/colinhacks/zod)
- [Zod Discord Community](https://discord.gg/zod)

**Related Libraries:**
- **zod-to-json-schema**: Convert Zod schemas to JSON Schema
- **zod-to-ts**: Generate TypeScript types from Zod schemas
- **zod-mock**: Generate mock data from Zod schemas
- **zod-prisma**: Generate Zod schemas from Prisma schemas
- **trpc**: Type-safe APIs with Zod validation
- **react-hook-form**: Form validation with Zod

**Integration Examples:**
- Next.js API routes with Zod
- Express.js middleware with Zod
- GraphQL resolvers with Zod
- tRPC procedures with Zod
- Remix actions/loaders with Zod

### Advanced Topics Not Covered

- Custom error map localization at scale
- Zod with GraphQL code generation
- Zod in microservices architectures
- Zod with event-driven systems
- Advanced schema versioning strategies
- Zod performance profiling and optimization
- Building Zod plugins and extensions
- Zod in monorepo environments

### Final Thoughts

Zod has revolutionized how we handle validation in TypeScript applications. Its elegant API, powerful type inference, and excellent developer experience make it an essential tool for modern TypeScript development.

By mastering Zod from beginner to ninja level, you now have the skills to:
- Build robust, type-safe applications
- Validate data at runtime with confidence
- Create maintainable schemas that evolve with your application
- Handle complex validation scenarios with ease
- Optimize validation performance for production use

Remember: **The schema is the source of truth**. Let Zod schemas drive your types, not the other way around.

---

## Appendix

### Quick Reference

#### Primitive Types
```typescript
z.string()     // string
z.number()     // number  
z.boolean()    // boolean
z.date()       // Date
z.bigint()     // bigint
z.undefined()  // undefined
z.null()       // null
z.any()        // any
z.unknown()    // unknown
z.never()      // never
z.void()       // void (undefined)
```

#### String Validations
```typescript
z.string().min(3)
z.string().max(20)
z.string().length(10)
z.string().email()
z.string().url()
z.string().uuid()
z.string().regex(/pattern/)
z.string().trim()
z.string().toLowerCase()
z.string().toUpperCase()
```

#### Number Validations
```typescript
z.number().min(0)
z.number().max(100)
z.number().int()
z.number().positive()
z.number().negative()
z.number().nonnegative()
z.number().nonpositive()
z.number().multipleOf(5)
z.number().finite()
z.number().safe()
```

#### Object Operations
```typescript
schema.pick({ key: true })
schema.omit({ key: true })
schema.partial()
schema.required()
schema.extend({ newKey: z.string() })
schema.merge(otherSchema)
```

#### Modifiers
```typescript
z.optional()   // T | undefined
z.nullable()   // T | null
z.default(val) // T with default
z.catch(val)   // T with fallback on error
```

#### Utility Types
```typescript
z.array(schema)
z.tuple([schema1, schema2])
z.record(schema)
z.map(keySchema, valueSchema)
z.set(schema)
z.union([schema1, schema2])
z.discriminatedUnion("key", [...])
z.intersection(schema1, schema2)
z.enum(["a", "b", "c"])
z.literal("value")
z.lazy(() => schema)
```

#### Methods
```typescript
schema.parse(data)           // Throws on error
schema.safeParse(data)       // Returns result object
schema.parseAsync(data)      // Async parse
schema.safeParseAsync(data)  // Async safe parse
schema.refine(fn, options)   // Custom validation
schema.transform(fn)         // Data transformation
schema.superRefine(fn)       // Advanced refinement
```

### Common Patterns

```typescript
// Pagination
const paginationSchema = z.object({
  page: z.coerce.number().positive().int().default(1),
  limit: z.coerce.number().positive().int().max(100).default(10),
});

// API Response
const apiResponseSchema = <T extends z.ZodTypeAny>(dataSchema: T) =>
  z.object({
    success: z.boolean(),
    data: dataSchema,
    message: z.string().optional(),
  });

// Timestamps
const timestampSchema = z.object({
  createdAt: z.date(),
  updatedAt: z.date(),
});

// Soft Delete
const softDeleteSchema = z.object({
  deletedAt: z.date().nullable(),
  isDeleted: z.boolean().default(false),
});

// Password Validation
const passwordSchema = z
  .string()
  .min(8)
  .regex(/[A-Z]/, "Must contain uppercase")
  .regex(/[a-z]/, "Must contain lowercase")
  .regex(/[0-9]/, "Must contain number")
  .regex(/[^A-Za-z0-9]/, "Must contain special character");

// Email Validation
const emailSchema = z.string().email().toLowerCase();

// UUID Validation
const uuidSchema = z.string().uuid();

// URL Validation
const urlSchema = z.string().url();

// Date Range
const dateRangeSchema = z
  .object({
    startDate: z.date(),
    endDate: z.date(),
  })
  .refine((data) => data.endDate > data.startDate, {
    message: "End date must be after start date",
    path: ["endDate"],
  });

// Conditional Fields
const conditionalSchema = z
  .object({
    type: z.enum(["email", "sms"]),
    email: z.string().email().optional(),
    phone: z.string().optional(),
  })
  .refine(
    (data) => {
      if (data.type === "email") return !!data.email;
      if (data.type === "sms") return !!data.phone;
      return true;
    },
    {
      message: "Required field missing",
    }
  );
```

### Troubleshooting

**Problem: "Type 'unknown' is not assignable to type..."**
```typescript
// ❌ Wrong
const data: User = userSchema.safeParse(input);

// ✅ Correct
const result = userSchema.safeParse(input);
if (result.success) {
  const data: User = result.data;
}
```

**Problem: Circular dependencies**
```typescript
// ✅ Use z.lazy()
interface Node {
  value: string;
  children: Node[];
}

const nodeSchema: z.ZodType<Node> = z.lazy(() =>
  z.object({
    value: z.string(),
    children: z.array(nodeSchema),
  })
);
```

**Problem: Type inference not working**
```typescript
// ❌ Wrong - using 'as const' after schema
const schema = z.object({ name: z.string() }) as const;

// ✅ Correct - use z.infer
const schema = z.object({ name: z.string() });
type Schema = z.infer<typeof schema>;
```

**Problem: Optional vs Nullable confusion**
```typescript
// Can be undefined, but not null
z.string().optional() // string | undefined

// Can be null, but not undefined  
z.string().nullable() // string | null

// Can be both
z.string().optional().nullable() // string | null | undefined
```

**Problem: Custom error messages not showing**
```typescript
// ❌ Wrong - message in wrong place
z.string().min(3, { message: "Too short" }).email();

// ✅ Correct - message for specific validation
z.string().min(3, { message: "Too short" }).email({ message: "Invalid email" });
```

### Performance Tips

1. **Cache schemas**: Don't recreate schemas on every request
2. **Use discriminated unions**: Faster than regular unions
3. **Avoid async when possible**: Sync validation is much faster
4. **Lazy evaluation**: Use `z.lazy()` for expensive schemas
5. **Pick/Omit instead of full validation**: When you only need some fields
6. **Batch validations**: Validate arrays of data together
7. **Use safeParse**: Avoid try/catch overhead when handling errors

---

## Practice Exercises

### Beginner Level

1. Create a user registration schema with email, password, and age validation
2. Build a product schema with name, price, and category
3. Implement form validation for a contact form
4. Create a schema for blog post metadata
5. Validate environment variables for a Node.js app

### Intermediate Level

1. Build a multi-step form with different schemas per step
2. Create an API request/response validation system
3. Implement a schema with complex cross-field validation
4. Build a recursive category tree schema
5. Create a payment processing schema with multiple payment methods

### Advanced Level

1. Build a schema registry with versioning
2. Implement a type-safe API client using Zod schemas
3. Create a schema migration system
4. Build custom Zod plugins
5. Implement a code generator that creates schemas from TypeScript interfaces

### Expert Level

1. Build a complete authentication system with Zod validation
2. Create a type-safe GraphQL schema using Zod
3. Implement a complex e-commerce checkout flow
4. Build a real-time chat system with message validation
5. Create a schema-driven form builder

---

**Congratulations!** You've completed the Zod Complete Mastery Tutorial. You now have comprehensive knowledge of Zod from basic validation to advanced patterns. Keep practicing and building real-world applications to solidify your skills.

Remember: The best way to learn is by doing. Start implementing Zod in your projects today!

---

*Tutorial Version: 1.0*  
*Last Updated: December 2025*  
*Zod Version: 3.x*  

**Happy Coding! 🚀**
