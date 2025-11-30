# Day 2: Basic Types in TypeScript

## Primitive Types

TypeScript includes all JavaScript primitive types plus a few additional ones.

### string

```typescript
// String type
let firstName: string = "John";
let lastName: string = 'Doe';
let fullName: string = `${firstName} ${lastName}`;

// Template literals
let greeting: string = `Hello, ${firstName}!`;

// String methods work as expected
let upper: string = firstName.toUpperCase();
let length: number = firstName.length;
```

### number

```typescript
// All numbers (no int/float distinction)
let integer: number = 42;
let decimal: number = 3.14;
let negative: number = -10;
let hex: number = 0xff;
let binary: number = 0b1010;
let octal: number = 0o17;

// Special number values
let infinity: number = Infinity;
let negInfinity: number = -Infinity;
let notANumber: number = NaN;

// BigInt for large integers
let bigNumber: bigint = 9007199254740991n;
let anotherBig: bigint = BigInt("9007199254740991");
```

### boolean

```typescript
let isActive: boolean = true;
let isComplete: boolean = false;

// Result of comparisons
let isValid: boolean = 5 > 3;  // true
let isEmpty: boolean = "".length === 0;  // true
```

### null and undefined

```typescript
// Explicit null
let empty: null = null;

// Explicit undefined
let notDefined: undefined = undefined;

// With strictNullChecks (recommended)
let name: string = null;      // Error!
let age: number = undefined;  // Error!

// Allow null/undefined with union types
let maybeName: string | null = null;
let maybeAge: number | undefined = undefined;

// Check for null/undefined
function greet(name: string | null) {
    if (name === null) {
        return "Hello, stranger!";
    }
    return `Hello, ${name}!`;
}
```

### symbol

```typescript
// Unique identifier
const id: symbol = Symbol("id");
const anotherId: symbol = Symbol("id");

console.log(id === anotherId);  // false (always unique)

// Use as object key
const ID = Symbol("id");
const user = {
    name: "John",
    [ID]: 12345
};

console.log(user[ID]);  // 12345
```

---

## Special Types

### any

Disables type checking - use sparingly!

```typescript
// any accepts anything
let value: any = "hello";
value = 42;
value = true;
value = { name: "John" };

// No type errors, but no safety
let anything: any = "hello";
anything.nonExistent();  // No error at compile time, but crashes at runtime!

// When to use any:
// 1. Migrating JavaScript to TypeScript
// 2. Working with third-party libraries without types
// 3. When you truly don't know the type

// Prefer unknown over any when possible
```

### unknown

Safer alternative to `any` - requires type checking before use:

```typescript
let value: unknown = "hello";

// Cannot use directly
// value.toUpperCase();  // Error: Object is of type 'unknown'

// Must narrow the type first
if (typeof value === "string") {
    console.log(value.toUpperCase());  // OK
}

// Type assertion (use carefully)
let str = value as string;
console.log(str.toUpperCase());

// unknown vs any
let anyValue: any = "hello";
anyValue.anything();  // No error (dangerous!)

let unknownValue: unknown = "hello";
// unknownValue.anything();  // Error! (safe)
```

### void

Represents no return value:

```typescript
// Function that doesn't return anything
function log(message: string): void {
    console.log(message);
}

// Arrow function with void
const warn = (message: string): void => {
    console.warn(message);
};

// void variables can only be undefined or null
let nothing: void = undefined;
// let something: void = "hello";  // Error!
```

### never

Represents values that never occur:

```typescript
// Function that never returns (throws or infinite loop)
function throwError(message: string): never {
    throw new Error(message);
}

function infiniteLoop(): never {
    while (true) {
        // Never ends
    }
}

// Exhaustive type checking
type Shape = "circle" | "square";

function getArea(shape: Shape): number {
    switch (shape) {
        case "circle":
            return Math.PI * 5 * 5;
        case "square":
            return 5 * 5;
        default:
            // This ensures all cases are handled
            const exhaustiveCheck: never = shape;
            throw new Error(`Unknown shape: ${exhaustiveCheck}`);
    }
}
```

---

## Arrays

### Basic Arrays

```typescript
// Array of strings
let names: string[] = ["John", "Jane", "Bob"];

// Array of numbers
let scores: number[] = [90, 85, 78, 92];

// Alternative generic syntax
let ids: Array<number> = [1, 2, 3];

// Array methods are typed
names.push("Alice");     // OK
// names.push(123);      // Error: Argument of type 'number' is not assignable

let first: string = names[0];
let doubled: number[] = scores.map(s => s * 2);
```

### Multi-dimensional Arrays

```typescript
// 2D array
let matrix: number[][] = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
];

// 3D array
let cube: number[][][] = [
    [[1, 2], [3, 4]],
    [[5, 6], [7, 8]]
];

// Access elements
let value: number = matrix[0][1];  // 2
```

### Readonly Arrays

```typescript
// Cannot modify
const numbers: readonly number[] = [1, 2, 3];
// numbers.push(4);  // Error!
// numbers[0] = 10;  // Error!

// Alternative syntax
const moreNumbers: ReadonlyArray<number> = [4, 5, 6];

// Can still read
console.log(numbers[0]);  // 1
console.log(numbers.length);  // 3

// Create new array from readonly
const doubled = numbers.map(n => n * 2);  // OK - creates new array
```

---

## Tuples

Fixed-length arrays with specific types at each position:

```typescript
// Basic tuple
let person: [string, number] = ["John", 30];

// Access by index
let name: string = person[0];  // "John"
let age: number = person[1];   // 30

// Destructuring
let [userName, userAge] = person;

// Error: wrong types or length
// let wrong: [string, number] = [30, "John"];  // Error: types swapped
// let short: [string, number] = ["John"];      // Error: missing element

// Optional elements
let optionalTuple: [string, number?] = ["John"];
let fullTuple: [string, number?] = ["John", 30];

// Rest elements
let mixedTuple: [string, ...number[]] = ["scores", 90, 85, 78];

// Named tuples (TypeScript 4.0+)
type Point = [x: number, y: number];
type RGB = [red: number, green: number, blue: number];

const point: Point = [10, 20];
const color: RGB = [255, 128, 0];

// Readonly tuples
const readonlyPoint: readonly [number, number] = [10, 20];
// readonlyPoint[0] = 5;  // Error!
```

### Tuple Use Cases

```typescript
// Function returning multiple values
function getNameAndAge(): [string, number] {
    return ["John", 30];
}

const [name, age] = getNameAndAge();

// React useState pattern
function useState<T>(initial: T): [T, (value: T) => void] {
    let state = initial;
    const setState = (value: T) => { state = value; };
    return [state, setState];
}

const [count, setCount] = useState(0);

// Coordinate pairs
type Coordinate = [number, number];
const locations: Coordinate[] = [
    [40.7128, -74.0060],  // New York
    [51.5074, -0.1278]    // London
];

// Key-value pairs
type Entry = [string, number];
const entries: Entry[] = [
    ["apple", 5],
    ["banana", 3],
    ["cherry", 8]
];
```

---

## Enums

Named constants:

### Numeric Enums

```typescript
// Basic enum (values start at 0)
enum Direction {
    Up,      // 0
    Down,    // 1
    Left,    // 2
    Right    // 3
}

let dir: Direction = Direction.Up;
console.log(dir);  // 0
console.log(Direction[0]);  // "Up" (reverse mapping)

// Custom starting value
enum Status {
    Pending = 1,
    Active,     // 2
    Completed,  // 3
    Cancelled   // 4
}

// Custom values
enum HttpStatus {
    OK = 200,
    Created = 201,
    BadRequest = 400,
    NotFound = 404,
    ServerError = 500
}

// Using enums
function handleStatus(status: HttpStatus) {
    if (status === HttpStatus.OK) {
        console.log("Success!");
    }
}
```

### String Enums

```typescript
enum Color {
    Red = "RED",
    Green = "GREEN",
    Blue = "BLUE"
}

let color: Color = Color.Red;
console.log(color);  // "RED"

// No reverse mapping for string enums
// console.log(Color["RED"]);  // Error!

// More readable in logs/debugging
enum LogLevel {
    Error = "ERROR",
    Warn = "WARN",
    Info = "INFO",
    Debug = "DEBUG"
}

function log(level: LogLevel, message: string) {
    console.log(`[${level}] ${message}`);
}

log(LogLevel.Error, "Something went wrong");
// Output: [ERROR] Something went wrong
```

### Const Enums

More efficient - inlined at compile time:

```typescript
const enum Direction {
    Up,
    Down,
    Left,
    Right
}

let dir = Direction.Up;
// Compiles to: let dir = 0;

// No runtime object, just values
// console.log(Direction);  // Error!
```

### Enum Alternatives

Consider using objects with `as const`:

```typescript
// Object with as const
const Direction = {
    Up: "UP",
    Down: "DOWN",
    Left: "LEFT",
    Right: "RIGHT"
} as const;

type Direction = typeof Direction[keyof typeof Direction];
// Type: "UP" | "DOWN" | "LEFT" | "RIGHT"

let dir: Direction = Direction.Up;

// Benefits over enum:
// - No runtime overhead
// - Works with string literal unions
// - Better tree-shaking
```

---

## Union Types

Value can be one of several types:

```typescript
// Basic union
let id: string | number;
id = "abc123";
id = 123;

// Union in function parameters
function format(value: string | number): string {
    if (typeof value === "string") {
        return value.toUpperCase();
    }
    return value.toFixed(2);
}

format("hello");  // "HELLO"
format(3.14159);  // "3.14"

// Union of literal types
type Status = "pending" | "active" | "completed";
let status: Status = "pending";
// status = "unknown";  // Error!

// Union with null
function find(id: number): string | null {
    if (id === 1) return "Found";
    return null;
}

// Discriminated unions
type Shape =
    | { kind: "circle"; radius: number }
    | { kind: "square"; side: number }
    | { kind: "rectangle"; width: number; height: number };

function area(shape: Shape): number {
    switch (shape.kind) {
        case "circle":
            return Math.PI * shape.radius ** 2;
        case "square":
            return shape.side ** 2;
        case "rectangle":
            return shape.width * shape.height;
    }
}
```

---

## Intersection Types

Combine multiple types:

```typescript
// Combine two types
type Person = {
    name: string;
    age: number;
};

type Employee = {
    employeeId: number;
    department: string;
};

type EmployeePerson = Person & Employee;

const employee: EmployeePerson = {
    name: "John",
    age: 30,
    employeeId: 12345,
    department: "Engineering"
};

// Extend interfaces
interface Printable {
    print(): void;
}

interface Loggable {
    log(): void;
}

type PrintableLoggable = Printable & Loggable;

class Document implements PrintableLoggable {
    print() { console.log("Printing..."); }
    log() { console.log("Logging..."); }
}

// Intersection with primitives
type StringAndNumber = string & number;  // Type: never (impossible)

// Practical use: mixin pattern
type Constructor<T = {}> = new (...args: any[]) => T;

function Timestamped<TBase extends Constructor>(Base: TBase) {
    return class extends Base {
        timestamp = Date.now();
    };
}
```

---

## Literal Types

Exact values as types:

```typescript
// String literals
let direction: "up" | "down" | "left" | "right";
direction = "up";     // OK
// direction = "north";  // Error!

// Number literals
let diceRoll: 1 | 2 | 3 | 4 | 5 | 6;
diceRoll = 3;
// diceRoll = 7;  // Error!

// Boolean literal (rarely useful alone)
let alwaysTrue: true = true;

// Combining with other types
type HttpMethod = "GET" | "POST" | "PUT" | "DELETE";

function request(url: string, method: HttpMethod) {
    // ...
}

request("/api/users", "GET");
// request("/api/users", "PATCH");  // Error!

// Object with literal properties
type Config = {
    readonly mode: "development" | "production";
    port: number;
};

const config: Config = {
    mode: "development",
    port: 3000
};

// as const for literal inference
const directions = ["up", "down", "left", "right"] as const;
// Type: readonly ["up", "down", "left", "right"]

type Direction = typeof directions[number];
// Type: "up" | "down" | "left" | "right"
```

---

## Type Assertions

Tell TypeScript you know better:

```typescript
// Angle bracket syntax (not in JSX)
let value: unknown = "hello";
let length: number = (<string>value).length;

// as syntax (preferred)
let length2: number = (value as string).length;

// When to use assertions
// 1. When you know more than TypeScript
const canvas = document.getElementById("canvas") as HTMLCanvasElement;
const ctx = canvas.getContext("2d");

// 2. With unknown type
function processInput(input: unknown) {
    if (typeof input === "string") {
        return input.toUpperCase();  // TypeScript narrows automatically
    }
    // But sometimes you need to assert
    return (input as { toString(): string }).toString();
}

// 3. Non-null assertion (use sparingly!)
function getValue(map: Map<string, number>, key: string) {
    // We know the key exists
    return map.get(key)!;  // ! asserts non-null
}

// const assertions
const colors = ["red", "green", "blue"] as const;
// Type: readonly ["red", "green", "blue"]

const config = {
    endpoint: "/api",
    timeout: 5000
} as const;
// Type: { readonly endpoint: "/api"; readonly timeout: 5000 }

// Double assertion (last resort!)
// let value = (something as unknown as TargetType);
```

---

## Type Narrowing

TypeScript narrows types based on conditions:

```typescript
// typeof narrowing
function process(value: string | number) {
    if (typeof value === "string") {
        // TypeScript knows value is string here
        return value.toUpperCase();
    }
    // TypeScript knows value is number here
    return value.toFixed(2);
}

// Truthiness narrowing
function printAll(strs: string | string[] | null) {
    if (strs) {
        if (typeof strs === "object") {
            // string[]
            strs.forEach(s => console.log(s));
        } else {
            // string
            console.log(strs);
        }
    }
}

// Equality narrowing
function example(x: string | number, y: string | boolean) {
    if (x === y) {
        // Both must be string
        console.log(x.toUpperCase());
    }
}

// instanceof narrowing
function processDate(value: Date | string) {
    if (value instanceof Date) {
        return value.toISOString();
    }
    return new Date(value).toISOString();
}

// in operator narrowing
type Fish = { swim: () => void };
type Bird = { fly: () => void };

function move(animal: Fish | Bird) {
    if ("swim" in animal) {
        animal.swim();
    } else {
        animal.fly();
    }
}

// Type predicates (custom type guards)
function isFish(animal: Fish | Bird): animal is Fish {
    return "swim" in animal;
}

function move2(animal: Fish | Bird) {
    if (isFish(animal)) {
        animal.swim();  // TypeScript knows it's Fish
    } else {
        animal.fly();   // TypeScript knows it's Bird
    }
}
```

---

## Practice Exercises

### Exercise 1: Define Types

```typescript
// Define types for:
// 1. A product with id (number), name (string), price (number), inStock (boolean)
// 2. An array of products
// 3. A function that takes a product and returns a formatted string

// Your code here
```

**Solution:**
```typescript
type Product = {
    id: number;
    name: string;
    price: number;
    inStock: boolean;
};

type ProductList = Product[];

function formatProduct(product: Product): string {
    const stock = product.inStock ? "In Stock" : "Out of Stock";
    return `${product.name} - $${product.price.toFixed(2)} (${stock})`;
}

// Test
const products: ProductList = [
    { id: 1, name: "Laptop", price: 999.99, inStock: true },
    { id: 2, name: "Mouse", price: 29.99, inStock: false }
];

console.log(formatProduct(products[0]));
// "Laptop - $999.99 (In Stock)"
```

### Exercise 2: Union Types

```typescript
// Create a function that handles different input types:
// - string: return uppercase
// - number: return doubled
// - boolean: return "yes" or "no"

// Your code here
```

**Solution:**
```typescript
function process(value: string | number | boolean): string | number {
    if (typeof value === "string") {
        return value.toUpperCase();
    }
    if (typeof value === "number") {
        return value * 2;
    }
    return value ? "yes" : "no";
}

console.log(process("hello"));  // "HELLO"
console.log(process(21));       // 42
console.log(process(true));     // "yes"
```

### Exercise 3: Tuple Types

```typescript
// Create a function that returns user info as a tuple:
// [id: number, name: string, isAdmin: boolean]

// Your code here
```

**Solution:**
```typescript
type UserInfo = [id: number, name: string, isAdmin: boolean];

function getUserInfo(userId: number): UserInfo {
    // Simulated data
    const users: Record<number, UserInfo> = {
        1: [1, "John", false],
        2: [2, "Admin", true]
    };

    return users[userId] ?? [0, "Unknown", false];
}

const [id, name, isAdmin] = getUserInfo(1);
console.log(`User ${id}: ${name}, Admin: ${isAdmin}`);
```

---

## Key Takeaways

1. **Primitives**: string, number, boolean, null, undefined, symbol, bigint
2. **any** disables type checking - use **unknown** instead when possible
3. **void** for functions without return, **never** for functions that don't return
4. **Arrays** use `type[]` or `Array<type>` syntax
5. **Tuples** are fixed-length arrays with specific types per position
6. **Enums** create named constants - consider `as const` objects as alternative
7. **Union types** (`|`) allow multiple types
8. **Intersection types** (`&`) combine types
9. **Literal types** are exact values as types
10. **Type narrowing** refines types based on conditions

---

**Next Lesson:** [Day 3 - Interfaces & Type Aliases](./day-03-interfaces-type-aliases.md)
