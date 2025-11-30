# Day 1: TypeScript Introduction & Setup

## What is TypeScript?

TypeScript is a **superset of JavaScript** that adds static typing. It compiles to plain JavaScript that runs anywhere JavaScript runs.

```typescript
// JavaScript
function add(a, b) {
    return a + b;
}
add("hello", "world");  // "helloworld" - probably not intended!

// TypeScript
function add(a: number, b: number): number {
    return a + b;
}
add("hello", "world");  // Error! Argument of type 'string' is not assignable
```

### Why TypeScript?

1. **Catch errors early** - Find bugs at compile time, not runtime
2. **Better tooling** - Autocomplete, refactoring, navigation
3. **Self-documenting** - Types describe what code does
4. **Safer refactoring** - Confidence when changing code
5. **Scalability** - Essential for large codebases
6. **Industry standard** - Used by most companies and frameworks

### TypeScript vs JavaScript

| Feature | JavaScript | TypeScript |
|---------|------------|------------|
| Type System | Dynamic | Static |
| Compilation | Interpreted | Compiled to JS |
| Error Detection | Runtime | Compile time |
| IDE Support | Basic | Excellent |
| Learning Curve | Lower | Slightly higher |
| Browser Support | Direct | Requires compilation |

---

## Installation and Setup

### Installing TypeScript

```bash
# Global installation
npm install -g typescript

# Check version
tsc --version

# Project-local installation (recommended)
npm init -y
npm install --save-dev typescript

# Use local TypeScript
npx tsc --version
```

### Initializing a TypeScript Project

```bash
# Create tsconfig.json
tsc --init

# Or with npx
npx tsc --init
```

### tsconfig.json Explained

```json
{
    "compilerOptions": {
        // Target JavaScript version
        "target": "ES2020",

        // Module system
        "module": "ESNext",
        "moduleResolution": "node",

        // Output directory
        "outDir": "./dist",
        "rootDir": "./src",

        // Strict type checking (recommended)
        "strict": true,
        "noImplicitAny": true,
        "strictNullChecks": true,

        // Additional checks
        "noUnusedLocals": true,
        "noUnusedParameters": true,
        "noImplicitReturns": true,

        // Source maps for debugging
        "sourceMap": true,

        // Allow JavaScript files
        "allowJs": true,

        // Skip type checking node_modules
        "skipLibCheck": true,

        // ES Module interop
        "esModuleInterop": true,
        "allowSyntheticDefaultImports": true,

        // Resolve JSON modules
        "resolveJsonModule": true,

        // Declaration files
        "declaration": true
    },
    "include": ["src/**/*"],
    "exclude": ["node_modules", "dist"]
}
```

### Recommended tsconfig.json for Beginners

```json
{
    "compilerOptions": {
        "target": "ES2020",
        "module": "ESNext",
        "moduleResolution": "node",
        "outDir": "./dist",
        "rootDir": "./src",
        "strict": true,
        "esModuleInterop": true,
        "skipLibCheck": true,
        "forceConsistentCasingInFileNames": true,
        "sourceMap": true
    },
    "include": ["src/**/*"],
    "exclude": ["node_modules"]
}
```

---

## Project Structure

```
my-typescript-project/
├── src/
│   ├── index.ts
│   ├── utils/
│   │   └── helpers.ts
│   └── types/
│       └── index.ts
├── dist/           # Compiled JavaScript
├── node_modules/
├── package.json
├── tsconfig.json
└── README.md
```

### package.json Scripts

```json
{
    "name": "my-typescript-project",
    "version": "1.0.0",
    "scripts": {
        "build": "tsc",
        "watch": "tsc --watch",
        "start": "node dist/index.js",
        "dev": "tsc --watch & nodemon dist/index.js",
        "lint": "eslint src/**/*.ts",
        "clean": "rm -rf dist"
    },
    "devDependencies": {
        "typescript": "^5.0.0",
        "@types/node": "^20.0.0"
    }
}
```

---

## Compiling TypeScript

### Basic Compilation

```bash
# Compile single file
tsc hello.ts

# Compile with config
tsc

# Watch mode
tsc --watch
# or
tsc -w

# Compile specific file ignoring tsconfig
tsc hello.ts --outDir dist
```

### Your First TypeScript File

```typescript
// src/index.ts

// Variable with type annotation
const message: string = "Hello, TypeScript!";

// Function with typed parameters and return
function greet(name: string): string {
    return `Hello, ${name}!`;
}

// Using the function
const greeting = greet("World");
console.log(greeting);

// Type error example (uncomment to see error)
// greet(123);  // Error: Argument of type 'number' is not assignable

// Object with type
const user: { name: string; age: number } = {
    name: "John",
    age: 30
};

console.log(`${user.name} is ${user.age} years old`);
```

Compile and run:

```bash
tsc
node dist/index.js
```

---

## Using ts-node for Development

Run TypeScript directly without manual compilation:

```bash
# Install ts-node
npm install --save-dev ts-node

# Run TypeScript file
npx ts-node src/index.ts

# With nodemon for auto-reload
npm install --save-dev nodemon
npx nodemon --exec ts-node src/index.ts
```

### nodemon.json Configuration

```json
{
    "watch": ["src"],
    "ext": "ts",
    "exec": "ts-node src/index.ts"
}
```

---

## Type Annotations Basics

### Variable Type Annotations

```typescript
// Explicit type annotations
let name: string = "John";
let age: number = 30;
let isActive: boolean = true;
let data: null = null;
let value: undefined = undefined;

// Type inference (TypeScript figures out the type)
let inferredString = "Hello";  // TypeScript knows this is a string
let inferredNumber = 42;       // TypeScript knows this is a number

// When to use annotations:
// 1. When declaring without initializing
let laterAssigned: string;
laterAssigned = "value";

// 2. When type can't be inferred
let mixed: string | number;
mixed = "hello";
mixed = 42;

// 3. When you want to be explicit
const config: { port: number; host: string } = {
    port: 3000,
    host: "localhost"
};
```

### Function Type Annotations

```typescript
// Parameter and return types
function add(a: number, b: number): number {
    return a + b;
}

// Arrow function
const multiply = (a: number, b: number): number => a * b;

// Void return type (no return value)
function log(message: string): void {
    console.log(message);
}

// Optional parameters
function greet(name: string, greeting?: string): string {
    return `${greeting || "Hello"}, ${name}!`;
}

greet("John");           // "Hello, John!"
greet("John", "Hi");     // "Hi, John!"

// Default parameters
function createUser(name: string, age: number = 18): object {
    return { name, age };
}

// Rest parameters
function sum(...numbers: number[]): number {
    return numbers.reduce((total, n) => total + n, 0);
}

sum(1, 2, 3, 4, 5);  // 15
```

### Array Type Annotations

```typescript
// Array of strings
let names: string[] = ["John", "Jane", "Bob"];

// Array of numbers
let scores: number[] = [90, 85, 78];

// Alternative syntax
let ids: Array<number> = [1, 2, 3];

// Mixed array (tuple)
let tuple: [string, number] = ["John", 30];

// Array of objects
let users: { name: string; age: number }[] = [
    { name: "John", age: 30 },
    { name: "Jane", age: 25 }
];
```

### Object Type Annotations

```typescript
// Inline object type
let user: { name: string; age: number; email?: string } = {
    name: "John",
    age: 30
};

// With optional property
user.email = "john@example.com";

// Readonly properties
let config: { readonly apiKey: string; timeout: number } = {
    apiKey: "secret123",
    timeout: 5000
};

// config.apiKey = "new";  // Error: Cannot assign to 'apiKey' because it is a read-only property
config.timeout = 10000;    // OK
```

---

## Type Inference

TypeScript automatically infers types when possible:

```typescript
// TypeScript infers: string
let message = "Hello";

// TypeScript infers: number
let count = 42;

// TypeScript infers: boolean
let isValid = true;

// TypeScript infers: string[]
let fruits = ["apple", "banana", "cherry"];

// TypeScript infers: { name: string; age: number }
let person = {
    name: "John",
    age: 30
};

// TypeScript infers return type: number
function double(n: number) {
    return n * 2;  // Returns number, TypeScript knows
}

// TypeScript infers: number
const result = double(5);

// When inference fails, annotation is needed
let value;         // Type: any (bad!)
let value2: string;  // Type: string (good!)
```

### Best Practices for Type Annotations

```typescript
// Let TypeScript infer when it can
const name = "John";  // ✓ Good
const name: string = "John";  // Unnecessary but not wrong

// Annotate function parameters
function greet(name: string) {  // ✓ Always annotate params
    return `Hello, ${name}`;
}

// Annotate when declaring without initializing
let laterValue: string;  // ✓ Needed

// Annotate complex return types
function getData(): Promise<User[]> {  // ✓ Clearer
    return fetch("/api/users").then(r => r.json());
}

// Annotate when type can't be inferred
const handlers: { [key: string]: () => void } = {};  // ✓ Complex type
```

---

## IDE Integration

### VS Code Setup

1. **Install extensions:**
   - TypeScript and JavaScript Language Features (built-in)
   - ESLint
   - Prettier

2. **VS Code settings (settings.json):**

```json
{
    "editor.formatOnSave": true,
    "editor.defaultFormatter": "esbenp.prettier-vscode",
    "typescript.updateImportsOnFileMove.enabled": "always",
    "typescript.suggest.autoImports": true,
    "editor.codeActionsOnSave": {
        "source.organizeImports": true
    }
}
```

### IDE Features

```typescript
// Hover for type info
const user = { name: "John", age: 30 };
// Hover over 'user' to see: { name: string; age: number }

// Auto-completion
user.  // Shows: name, age

// Error highlighting
const x: string = 123;  // Red underline: Type 'number' is not assignable to type 'string'

// Quick fixes
const unused = "test";  // Yellow: 'unused' is declared but never used
// Click lightbulb for fix options

// Go to definition
// Ctrl+Click on function name

// Find all references
// Shift+F12 or right-click
```

---

## Common Errors and Solutions

### Error: Cannot find module

```typescript
// Error: Cannot find module './utils'

// Solution 1: Check file extension
import { helper } from "./utils";      // Looks for utils.ts or utils/index.ts
import { helper } from "./utils.js";   // After compilation

// Solution 2: Add type declarations
// npm install --save-dev @types/module-name

// Solution 3: Create declaration file
// src/types/module-name.d.ts
declare module "module-name" {
    export function something(): void;
}
```

### Error: Type 'X' is not assignable to type 'Y'

```typescript
// Error
let name: string = 123;

// Solution: Use correct type
let name: string = "John";

// Or change the type
let name: number = 123;

// Or use union type if both are valid
let name: string | number = 123;
```

### Error: Property 'X' does not exist on type 'Y'

```typescript
// Error
const user = { name: "John" };
console.log(user.age);  // Property 'age' does not exist

// Solution 1: Add the property to the type
const user: { name: string; age?: number } = { name: "John" };

// Solution 2: Use type assertion (if you're sure)
const user = { name: "John" } as { name: string; age: number };

// Solution 3: Use optional chaining
console.log(user.age ?? "unknown");
```

---

## Practice Exercises

### Exercise 1: Type a User Object

```typescript
// Create a typed user object with:
// - name (string)
// - age (number)
// - email (optional string)
// - isAdmin (boolean)

// Your code here
```

**Solution:**
```typescript
const user: {
    name: string;
    age: number;
    email?: string;
    isAdmin: boolean;
} = {
    name: "John Doe",
    age: 30,
    isAdmin: false
};

// With email
const admin: typeof user = {
    name: "Admin",
    age: 35,
    email: "admin@example.com",
    isAdmin: true
};
```

### Exercise 2: Type a Function

```typescript
// Create a function that:
// - Takes an array of numbers
// - Returns the sum of all numbers
// - Properly typed

// Your code here
```

**Solution:**
```typescript
function sumArray(numbers: number[]): number {
    return numbers.reduce((sum, num) => sum + num, 0);
}

// Arrow function version
const sumArrayArrow = (numbers: number[]): number => {
    return numbers.reduce((sum, num) => sum + num, 0);
};

// Test
console.log(sumArray([1, 2, 3, 4, 5]));  // 15
```

### Exercise 3: Fix Type Errors

```typescript
// Fix all type errors in this code

let count = "5";
count = count + 1;

function multiply(a, b) {
    return a * b;
}

const result = multiply("3", 4);

const users = [
    { name: "John" },
    { name: "Jane", age: 25 }
];

users.forEach(user => {
    console.log(user.age);
});
```

**Solution:**
```typescript
let count: number = 5;
count = count + 1;

function multiply(a: number, b: number): number {
    return a * b;
}

const result = multiply(3, 4);

const users: { name: string; age?: number }[] = [
    { name: "John" },
    { name: "Jane", age: 25 }
];

users.forEach(user => {
    console.log(user.age ?? "Age not provided");
});
```

---

## Key Takeaways

1. **TypeScript adds types to JavaScript** - Catches errors before runtime
2. **Use `tsc` to compile** - TypeScript → JavaScript
3. **tsconfig.json** configures the compiler
4. **Type annotations** describe what types values should be
5. **Type inference** lets TypeScript figure out types automatically
6. **IDE integration** provides autocomplete, errors, and navigation
7. **Start with `strict: true`** for maximum safety

---

## Self-Check Questions

1. What is TypeScript and why use it?
2. What does `tsc` do?
3. What's the difference between type annotation and type inference?
4. When should you explicitly annotate types?
5. What does `strict: true` do in tsconfig.json?
6. How do you make a property optional?

---

**Next Lesson:** [Day 2 - Basic Types](./day-02-basic-types.md)
