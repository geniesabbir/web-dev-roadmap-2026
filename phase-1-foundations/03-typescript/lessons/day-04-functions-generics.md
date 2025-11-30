# Day 4: Functions & Generics in TypeScript

## Function Types in TypeScript

TypeScript provides powerful ways to type functions, making them safer and more predictable.

### Basic Function Type Annotations

```typescript
// Function declaration with types
function add(a: number, b: number): number {
    return a + b;
}

// Arrow function with types
const multiply = (a: number, b: number): number => a * b;

// Function expression with types
const divide: (a: number, b: number) => number = function(a, b) {
    return a / b;
};

// Type alias for function
type MathOperation = (a: number, b: number) => number;

const subtract: MathOperation = (a, b) => a - b;
const modulo: MathOperation = (a, b) => a % b;
```

### Optional and Default Parameters

```typescript
// Optional parameters (must come after required)
function greet(name: string, greeting?: string): string {
    return `${greeting || "Hello"}, ${name}!`;
}

greet("John");           // "Hello, John!"
greet("John", "Hi");     // "Hi, John!"

// Default parameters
function createUser(
    name: string,
    age: number = 18,
    role: string = "user"
): object {
    return { name, age, role };
}

createUser("John");                    // { name: "John", age: 18, role: "user" }
createUser("Jane", 25);                // { name: "Jane", age: 25, role: "user" }
createUser("Admin", 30, "admin");      // { name: "Admin", age: 30, role: "admin" }

// Optional vs default - they behave differently
function example1(value?: number) {
    // value is number | undefined
    console.log(value);  // could be undefined
}

function example2(value: number = 10) {
    // value is always number
    console.log(value);  // always a number
}

example1();      // undefined
example2();      // 10
```

### Rest Parameters

```typescript
// Rest parameters must be last and are typed as arrays
function sum(...numbers: number[]): number {
    return numbers.reduce((total, n) => total + n, 0);
}

sum(1, 2, 3);           // 6
sum(1, 2, 3, 4, 5);     // 15

// Rest with other parameters
function logMessages(prefix: string, ...messages: string[]): void {
    messages.forEach(msg => console.log(`${prefix}: ${msg}`));
}

logMessages("INFO", "Server started", "Listening on port 3000");

// Typed tuple rest parameters
function formatDate(
    format: string,
    ...parts: [number, number, number]
): string {
    const [year, month, day] = parts;
    return format
        .replace("YYYY", year.toString())
        .replace("MM", month.toString().padStart(2, "0"))
        .replace("DD", day.toString().padStart(2, "0"));
}

formatDate("YYYY-MM-DD", 2024, 3, 15);  // "2024-03-15"
```

### Return Types

```typescript
// Explicit return type
function getUser(id: number): { name: string; id: number } {
    return { name: "John", id };
}

// Void - function doesn't return anything
function logMessage(message: string): void {
    console.log(message);
    // No return statement, or just: return;
}

// Never - function never returns (throws or infinite loop)
function throwError(message: string): never {
    throw new Error(message);
}

function infiniteLoop(): never {
    while (true) {
        // Does something forever
    }
}

// Never in exhaustive checks
type Color = "red" | "green" | "blue";

function getColorCode(color: Color): string {
    switch (color) {
        case "red":
            return "#FF0000";
        case "green":
            return "#00FF00";
        case "blue":
            return "#0000FF";
        default:
            // This ensures all cases are handled
            const exhaustiveCheck: never = color;
            return exhaustiveCheck;
    }
}

// Undefined vs void
function returnsUndefined(): undefined {
    return undefined;  // Must explicitly return undefined
}

function returnsVoid(): void {
    // Can return nothing, undefined, or just return;
}
```

### Function Overloads

Function overloads allow you to define multiple function signatures:

```typescript
// Overload signatures (no implementation)
function formatValue(value: string): string;
function formatValue(value: number): string;
function formatValue(value: boolean): string;

// Implementation signature (must be compatible with all overloads)
function formatValue(value: string | number | boolean): string {
    if (typeof value === "string") {
        return value.toUpperCase();
    } else if (typeof value === "number") {
        return value.toFixed(2);
    } else {
        return value ? "YES" : "NO";
    }
}

formatValue("hello");   // "HELLO"
formatValue(3.14159);   // "3.14"
formatValue(true);      // "YES"

// More complex overload example
function createElement(tag: "a"): HTMLAnchorElement;
function createElement(tag: "div"): HTMLDivElement;
function createElement(tag: "span"): HTMLSpanElement;
function createElement(tag: string): HTMLElement;
function createElement(tag: string): HTMLElement {
    return document.createElement(tag);
}

const anchor = createElement("a");     // Type: HTMLAnchorElement
const div = createElement("div");      // Type: HTMLDivElement
const custom = createElement("custom"); // Type: HTMLElement

// Overloads with different parameter counts
function padding(all: number): string;
function padding(vertical: number, horizontal: number): string;
function padding(top: number, right: number, bottom: number, left: number): string;
function padding(a: number, b?: number, c?: number, d?: number): string {
    if (b === undefined) {
        return `${a}px`;
    } else if (c === undefined) {
        return `${a}px ${b}px`;
    } else {
        return `${a}px ${b}px ${c}px ${d}px`;
    }
}

padding(10);              // "10px"
padding(10, 20);          // "10px 20px"
padding(10, 20, 30, 40);  // "10px 20px 30px 40px"
```

### This Parameter

```typescript
// Typing 'this' in function
interface User {
    name: string;
    greet(this: User): string;
}

const user: User = {
    name: "John",
    greet() {
        return `Hello, I'm ${this.name}`;
    }
};

user.greet();  // "Hello, I'm John"

// This prevents incorrect context
const greetFn = user.greet;
// greetFn();  // Error: 'this' context of type 'void' is not assignable

// Using this in callbacks
class Counter {
    count = 0;

    // Arrow function preserves 'this'
    increment = () => {
        this.count++;
    };

    // Regular method needs bind or arrow
    decrement() {
        this.count--;
    }
}

const counter = new Counter();
const incrementFn = counter.increment;
incrementFn();  // Works - arrow function preserves 'this'

const decrementFn = counter.decrement;
// decrementFn();  // Error at runtime - 'this' is undefined
```

### Callback Types

```typescript
// Simple callback
function fetchData(callback: (data: string) => void): void {
    setTimeout(() => callback("Data loaded"), 1000);
}

fetchData((data) => console.log(data));

// Callback with error handling
type Callback<T> = (error: Error | null, result: T | null) => void;

function readFile(path: string, callback: Callback<string>): void {
    try {
        const content = "file content";
        callback(null, content);
    } catch (error) {
        callback(error as Error, null);
    }
}

readFile("file.txt", (error, content) => {
    if (error) {
        console.error(error.message);
    } else {
        console.log(content);
    }
});

// Event handler types
type ClickHandler = (event: MouseEvent) => void;
type ChangeHandler = (event: Event) => void;

const handleClick: ClickHandler = (event) => {
    console.log(`Clicked at ${event.clientX}, ${event.clientY}`);
};

// Higher-order function types
type Transformer<T> = (item: T) => T;
type Predicate<T> = (item: T) => boolean;
type Reducer<T, R> = (accumulator: R, item: T) => R;

const double: Transformer<number> = (n) => n * 2;
const isEven: Predicate<number> = (n) => n % 2 === 0;
const sumReducer: Reducer<number, number> = (acc, n) => acc + n;
```

---

## Introduction to Generics

Generics allow you to write reusable code that works with multiple types while maintaining type safety.

### Why Generics?

```typescript
// Without generics - loses type information
function identityAny(value: any): any {
    return value;
}

const result1 = identityAny("hello");  // Type: any - we lost the string type!

// With generics - preserves type information
function identity<T>(value: T): T {
    return value;
}

const result2 = identity("hello");     // Type: string
const result3 = identity(42);          // Type: number (42 literal)
const result4 = identity<number>(42);  // Type: number (explicit)

// Generic maintains relationship between input and output
function firstElement<T>(arr: T[]): T | undefined {
    return arr[0];
}

const first1 = firstElement(["a", "b", "c"]);  // Type: string | undefined
const first2 = firstElement([1, 2, 3]);         // Type: number | undefined
```

### Generic Syntax

```typescript
// Single type parameter
function wrap<T>(value: T): { value: T } {
    return { value };
}

// Multiple type parameters
function pair<T, U>(first: T, second: U): [T, U] {
    return [first, second];
}

const p1 = pair("hello", 42);      // Type: [string, number]
const p2 = pair(true, { x: 1 });   // Type: [boolean, { x: number }]

// Naming conventions
// T - Type (most common)
// K - Key
// V - Value
// E - Element
// R - Return type
// S, U - Additional types

function mapObject<K extends string, V, R>(
    obj: Record<K, V>,
    fn: (value: V, key: K) => R
): Record<K, R> {
    const result = {} as Record<K, R>;
    for (const key in obj) {
        result[key] = fn(obj[key], key);
    }
    return result;
}
```

### Generic Constraints

```typescript
// Constraint with extends
function getLength<T extends { length: number }>(item: T): number {
    return item.length;
}

getLength("hello");        // 5
getLength([1, 2, 3]);      // 3
getLength({ length: 10 }); // 10
// getLength(123);         // Error: number doesn't have length

// Constraint with interface
interface Printable {
    print(): string;
}

function printAll<T extends Printable>(items: T[]): void {
    items.forEach(item => console.log(item.print()));
}

// Using keyof constraint
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
    return obj[key];
}

const user = { name: "John", age: 30, email: "john@example.com" };

getProperty(user, "name");   // Type: string
getProperty(user, "age");    // Type: number
// getProperty(user, "foo"); // Error: "foo" is not a key of user

// Multiple constraints
interface HasId {
    id: number;
}

interface HasName {
    name: string;
}

function logEntity<T extends HasId & HasName>(entity: T): void {
    console.log(`${entity.id}: ${entity.name}`);
}

logEntity({ id: 1, name: "Test", extra: "data" });  // OK
```

### Generic Functions

```typescript
// Array utilities with generics
function map<T, U>(array: T[], fn: (item: T, index: number) => U): U[] {
    return array.map(fn);
}

const numbers = [1, 2, 3];
const doubled = map(numbers, n => n * 2);        // Type: number[]
const strings = map(numbers, n => n.toString()); // Type: string[]

// Filter with type guard
function filter<T>(array: T[], predicate: (item: T) => boolean): T[] {
    return array.filter(predicate);
}

const evens = filter([1, 2, 3, 4], n => n % 2 === 0);  // Type: number[]

// Reduce
function reduce<T, R>(
    array: T[],
    reducer: (acc: R, item: T) => R,
    initial: R
): R {
    return array.reduce(reducer, initial);
}

const sum = reduce([1, 2, 3], (acc, n) => acc + n, 0);  // Type: number

// Find with undefined
function find<T>(array: T[], predicate: (item: T) => boolean): T | undefined {
    return array.find(predicate);
}

const found = find([1, 2, 3], n => n > 2);  // Type: number | undefined
```

### Generic Interfaces

```typescript
// Generic interface
interface Container<T> {
    value: T;
    getValue(): T;
    setValue(value: T): void;
}

class Box<T> implements Container<T> {
    constructor(public value: T) {}

    getValue(): T {
        return this.value;
    }

    setValue(value: T): void {
        this.value = value;
    }
}

const numberBox = new Box(42);        // Box<number>
const stringBox = new Box("hello");   // Box<string>

// Generic interface with multiple types
interface KeyValuePair<K, V> {
    key: K;
    value: V;
}

const pair1: KeyValuePair<string, number> = { key: "age", value: 30 };
const pair2: KeyValuePair<number, string> = { key: 1, value: "first" };

// Generic interface for functions
interface Transformer<Input, Output> {
    (input: Input): Output;
}

const stringify: Transformer<number, string> = (n) => n.toString();
const parse: Transformer<string, number> = (s) => parseInt(s, 10);

// Generic interface with default type
interface Response<T = any> {
    data: T;
    status: number;
    message: string;
}

const response1: Response<string[]> = {
    data: ["item1", "item2"],
    status: 200,
    message: "Success"
};

const response2: Response = {  // Uses default 'any'
    data: { anything: "goes" },
    status: 200,
    message: "OK"
};
```

### Generic Type Aliases

```typescript
// Generic type alias
type Nullable<T> = T | null;
type Optional<T> = T | undefined;
type Maybe<T> = T | null | undefined;

let name: Nullable<string> = "John";
name = null;  // OK

// Generic array alias
type List<T> = T[];
type ReadonlyList<T> = readonly T[];

const numbers: List<number> = [1, 2, 3];
const frozen: ReadonlyList<string> = ["a", "b", "c"];
// frozen.push("d");  // Error: readonly

// Generic object type
type Dictionary<T> = {
    [key: string]: T;
};

const scores: Dictionary<number> = {
    math: 95,
    english: 88,
    science: 92
};

// Generic function type
type AsyncFunction<T> = () => Promise<T>;

const fetchUser: AsyncFunction<{ name: string }> = async () => {
    return { name: "John" };
};

// Conditional generic type
type ArrayOrSingle<T> = T extends any[] ? T : T[];

type Test1 = ArrayOrSingle<string>;     // string[]
type Test2 = ArrayOrSingle<number[]>;   // number[]
```

### Generic Classes

```typescript
// Generic class
class Stack<T> {
    private items: T[] = [];

    push(item: T): void {
        this.items.push(item);
    }

    pop(): T | undefined {
        return this.items.pop();
    }

    peek(): T | undefined {
        return this.items[this.items.length - 1];
    }

    isEmpty(): boolean {
        return this.items.length === 0;
    }

    size(): number {
        return this.items.length;
    }
}

const numberStack = new Stack<number>();
numberStack.push(1);
numberStack.push(2);
numberStack.pop();  // Type: number | undefined

const stringStack = new Stack<string>();
stringStack.push("hello");
// stringStack.push(123);  // Error: number is not assignable to string

// Generic class with constraints
class SortedList<T extends { compareTo(other: T): number }> {
    private items: T[] = [];

    add(item: T): void {
        this.items.push(item);
        this.items.sort((a, b) => a.compareTo(b));
    }

    getAll(): T[] {
        return [...this.items];
    }
}

// Generic class with multiple types
class Pair<T, U> {
    constructor(
        public first: T,
        public second: U
    ) {}

    swap(): Pair<U, T> {
        return new Pair(this.second, this.first);
    }

    map<V, W>(
        firstFn: (value: T) => V,
        secondFn: (value: U) => W
    ): Pair<V, W> {
        return new Pair(firstFn(this.first), secondFn(this.second));
    }
}

const pair = new Pair("hello", 42);
const swapped = pair.swap();                    // Pair<number, string>
const mapped = pair.map(s => s.length, n => n * 2);  // Pair<number, number>
```

---

## Advanced Generic Patterns

### Generic Default Types

```typescript
// Default type parameter
interface ApiResponse<T = unknown> {
    data: T;
    error: string | null;
    loading: boolean;
}

// Without specifying type - uses default
const response1: ApiResponse = {
    data: "anything",
    error: null,
    loading: false
};

// With specific type
const response2: ApiResponse<{ id: number; name: string }> = {
    data: { id: 1, name: "John" },
    error: null,
    loading: false
};

// Multiple defaults
interface Config<
    T = string,
    U = number,
    V = boolean
> {
    value: T;
    count: U;
    enabled: V;
}

const config1: Config = { value: "test", count: 5, enabled: true };
const config2: Config<number> = { value: 42, count: 5, enabled: true };
const config3: Config<number, string> = { value: 42, count: "five", enabled: true };
```

### Mapped Types with Generics

```typescript
// Create readonly version of any type
type Readonly<T> = {
    readonly [K in keyof T]: T[K];
};

// Create optional version of any type
type Partial<T> = {
    [K in keyof T]?: T[K];
};

// Create required version of any type
type Required<T> = {
    [K in keyof T]-?: T[K];
};

// Pick specific properties
type Pick<T, K extends keyof T> = {
    [P in K]: T[P];
};

// Omit specific properties
type Omit<T, K extends keyof T> = {
    [P in Exclude<keyof T, K>]: T[P];
};

// Practical example
interface User {
    id: number;
    name: string;
    email: string;
    password: string;
}

type PublicUser = Omit<User, "password">;
// { id: number; name: string; email: string }

type UserCredentials = Pick<User, "email" | "password">;
// { email: string; password: string }

type PartialUser = Partial<User>;
// All properties optional

type ReadonlyUser = Readonly<User>;
// All properties readonly
```

### Conditional Types

```typescript
// Basic conditional type
type IsString<T> = T extends string ? true : false;

type Test1 = IsString<string>;   // true
type Test2 = IsString<number>;   // false

// Extract types from other types
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

function getUser() {
    return { id: 1, name: "John" };
}

type UserType = ReturnType<typeof getUser>;
// { id: number; name: string }

// Parameter types
type Parameters<T> = T extends (...args: infer P) => any ? P : never;

function createUser(name: string, age: number): void {}

type CreateUserParams = Parameters<typeof createUser>;
// [string, number]

// Array element type
type ElementType<T> = T extends (infer E)[] ? E : never;

type StringArrayElement = ElementType<string[]>;  // string
type NumberArrayElement = ElementType<number[]>;  // number

// Promise unwrapping
type Awaited<T> = T extends Promise<infer U> ? Awaited<U> : T;

type Result1 = Awaited<Promise<string>>;           // string
type Result2 = Awaited<Promise<Promise<number>>>;  // number
```

### Generic Utility Functions

```typescript
// Type-safe object entries
function entries<T extends object>(obj: T): [keyof T, T[keyof T]][] {
    return Object.entries(obj) as [keyof T, T[keyof T]][];
}

const user = { name: "John", age: 30 };
const userEntries = entries(user);
// Type: ["name" | "age", string | number][]

// Type-safe object keys
function keys<T extends object>(obj: T): (keyof T)[] {
    return Object.keys(obj) as (keyof T)[];
}

const userKeys = keys(user);  // Type: ("name" | "age")[]

// Type-safe merge
function merge<T extends object, U extends object>(
    target: T,
    source: U
): T & U {
    return { ...target, ...source };
}

const merged = merge({ a: 1 }, { b: "hello" });
// Type: { a: number } & { b: string }

// Type-safe pick
function pick<T extends object, K extends keyof T>(
    obj: T,
    keys: K[]
): Pick<T, K> {
    const result = {} as Pick<T, K>;
    keys.forEach(key => {
        result[key] = obj[key];
    });
    return result;
}

const picked = pick(user, ["name"]);
// Type: { name: string }

// Type-safe omit
function omit<T extends object, K extends keyof T>(
    obj: T,
    keys: K[]
): Omit<T, K> {
    const result = { ...obj };
    keys.forEach(key => delete result[key]);
    return result as Omit<T, K>;
}

const omitted = omit(user, ["age"]);
// Type: { name: string }
```

---

## Practical Examples

### API Response Handler

```typescript
// Generic API response types
interface ApiSuccess<T> {
    success: true;
    data: T;
}

interface ApiError {
    success: false;
    error: {
        code: string;
        message: string;
    };
}

type ApiResponse<T> = ApiSuccess<T> | ApiError;

// Generic fetch wrapper
async function fetchApi<T>(url: string): Promise<ApiResponse<T>> {
    try {
        const response = await fetch(url);
        if (!response.ok) {
            return {
                success: false,
                error: {
                    code: response.status.toString(),
                    message: response.statusText
                }
            };
        }
        const data: T = await response.json();
        return { success: true, data };
    } catch (error) {
        return {
            success: false,
            error: {
                code: "NETWORK_ERROR",
                message: error instanceof Error ? error.message : "Unknown error"
            }
        };
    }
}

// Type-safe usage
interface User {
    id: number;
    name: string;
    email: string;
}

async function getUser(id: number): Promise<User | null> {
    const response = await fetchApi<User>(`/api/users/${id}`);

    if (response.success) {
        return response.data;  // Type: User
    } else {
        console.error(response.error.message);
        return null;
    }
}
```

### Event Emitter

```typescript
// Generic event emitter
type EventHandler<T> = (data: T) => void;

class EventEmitter<Events extends Record<string, any>> {
    private handlers: {
        [K in keyof Events]?: EventHandler<Events[K]>[];
    } = {};

    on<K extends keyof Events>(
        event: K,
        handler: EventHandler<Events[K]>
    ): void {
        if (!this.handlers[event]) {
            this.handlers[event] = [];
        }
        this.handlers[event]!.push(handler);
    }

    off<K extends keyof Events>(
        event: K,
        handler: EventHandler<Events[K]>
    ): void {
        const handlers = this.handlers[event];
        if (handlers) {
            const index = handlers.indexOf(handler);
            if (index !== -1) {
                handlers.splice(index, 1);
            }
        }
    }

    emit<K extends keyof Events>(event: K, data: Events[K]): void {
        const handlers = this.handlers[event];
        if (handlers) {
            handlers.forEach(handler => handler(data));
        }
    }
}

// Usage with type-safe events
interface AppEvents {
    login: { userId: number; timestamp: Date };
    logout: { userId: number };
    error: { message: string; code: number };
}

const emitter = new EventEmitter<AppEvents>();

// Type-safe event handlers
emitter.on("login", (data) => {
    console.log(`User ${data.userId} logged in at ${data.timestamp}`);
});

emitter.on("error", (data) => {
    console.error(`Error ${data.code}: ${data.message}`);
});

// Type-safe emit
emitter.emit("login", { userId: 123, timestamp: new Date() });
// emitter.emit("login", { userId: "wrong" });  // Error: string not assignable to number
```

### Form State Manager

```typescript
// Generic form state
interface FormState<T> {
    values: T;
    errors: Partial<Record<keyof T, string>>;
    touched: Partial<Record<keyof T, boolean>>;
    isValid: boolean;
    isSubmitting: boolean;
}

type Validator<T> = (value: T[keyof T], values: T) => string | null;
type Validators<T> = Partial<Record<keyof T, Validator<T>>>;

class FormManager<T extends Record<string, any>> {
    private state: FormState<T>;
    private validators: Validators<T>;

    constructor(initialValues: T, validators: Validators<T> = {}) {
        this.validators = validators;
        this.state = {
            values: initialValues,
            errors: {},
            touched: {},
            isValid: true,
            isSubmitting: false
        };
    }

    getValue<K extends keyof T>(field: K): T[K] {
        return this.state.values[field];
    }

    setValue<K extends keyof T>(field: K, value: T[K]): void {
        this.state.values[field] = value;
        this.validateField(field);
    }

    private validateField<K extends keyof T>(field: K): void {
        const validator = this.validators[field];
        if (validator) {
            const error = validator(this.state.values[field], this.state.values);
            if (error) {
                this.state.errors[field] = error;
            } else {
                delete this.state.errors[field];
            }
        }
        this.updateValidity();
    }

    private updateValidity(): void {
        this.state.isValid = Object.keys(this.state.errors).length === 0;
    }

    setTouched<K extends keyof T>(field: K): void {
        this.state.touched[field] = true;
    }

    getState(): FormState<T> {
        return { ...this.state };
    }

    async submit(
        onSubmit: (values: T) => Promise<void>
    ): Promise<boolean> {
        // Validate all fields
        for (const field of Object.keys(this.state.values) as (keyof T)[]) {
            this.validateField(field);
            this.setTouched(field);
        }

        if (!this.state.isValid) {
            return false;
        }

        this.state.isSubmitting = true;
        try {
            await onSubmit(this.state.values);
            return true;
        } catch (error) {
            return false;
        } finally {
            this.state.isSubmitting = false;
        }
    }
}

// Usage
interface LoginForm {
    email: string;
    password: string;
    rememberMe: boolean;
}

const loginForm = new FormManager<LoginForm>(
    {
        email: "",
        password: "",
        rememberMe: false
    },
    {
        email: (value) => {
            if (!value) return "Email is required";
            if (!value.includes("@")) return "Invalid email";
            return null;
        },
        password: (value) => {
            if (!value) return "Password is required";
            if (value.length < 8) return "Password must be at least 8 characters";
            return null;
        }
    }
);

loginForm.setValue("email", "john@example.com");  // Type-safe
loginForm.setValue("password", "secret123");
// loginForm.setValue("email", 123);  // Error: number not assignable to string
```

---

## Practice Exercises

### Exercise 1: Generic Stack with Min/Max

```typescript
// Create a generic Stack class that tracks min/max for comparable items
// Requirements:
// - push(item: T): void
// - pop(): T | undefined
// - getMin(): T | undefined (only for numeric types)
// - getMax(): T | undefined (only for numeric types)

// Your code here
```

**Solution:**
```typescript
class MinMaxStack<T extends number> {
    private items: T[] = [];
    private minStack: T[] = [];
    private maxStack: T[] = [];

    push(item: T): void {
        this.items.push(item);

        // Update min stack
        if (this.minStack.length === 0 || item <= this.minStack[this.minStack.length - 1]) {
            this.minStack.push(item);
        }

        // Update max stack
        if (this.maxStack.length === 0 || item >= this.maxStack[this.maxStack.length - 1]) {
            this.maxStack.push(item);
        }
    }

    pop(): T | undefined {
        const item = this.items.pop();

        if (item !== undefined) {
            if (item === this.minStack[this.minStack.length - 1]) {
                this.minStack.pop();
            }
            if (item === this.maxStack[this.maxStack.length - 1]) {
                this.maxStack.pop();
            }
        }

        return item;
    }

    getMin(): T | undefined {
        return this.minStack[this.minStack.length - 1];
    }

    getMax(): T | undefined {
        return this.maxStack[this.maxStack.length - 1];
    }

    peek(): T | undefined {
        return this.items[this.items.length - 1];
    }

    size(): number {
        return this.items.length;
    }
}

// Test
const stack = new MinMaxStack<number>();
stack.push(5);
stack.push(2);
stack.push(8);
stack.push(1);

console.log(stack.getMin());  // 1
console.log(stack.getMax());  // 8
stack.pop();
console.log(stack.getMin());  // 2
```

### Exercise 2: Generic Cache

```typescript
// Create a generic cache with expiration
// Requirements:
// - set(key: string, value: T, ttlMs?: number): void
// - get(key: string): T | undefined
// - has(key: string): boolean
// - delete(key: string): boolean
// - clear(): void

// Your code here
```

**Solution:**
```typescript
interface CacheEntry<T> {
    value: T;
    expiresAt: number | null;
}

class Cache<T> {
    private store: Map<string, CacheEntry<T>> = new Map();
    private defaultTtlMs: number | null;

    constructor(defaultTtlMs: number | null = null) {
        this.defaultTtlMs = defaultTtlMs;
    }

    set(key: string, value: T, ttlMs?: number): void {
        const ttl = ttlMs ?? this.defaultTtlMs;
        const expiresAt = ttl ? Date.now() + ttl : null;

        this.store.set(key, { value, expiresAt });
    }

    get(key: string): T | undefined {
        const entry = this.store.get(key);

        if (!entry) {
            return undefined;
        }

        if (entry.expiresAt && Date.now() > entry.expiresAt) {
            this.store.delete(key);
            return undefined;
        }

        return entry.value;
    }

    has(key: string): boolean {
        return this.get(key) !== undefined;
    }

    delete(key: string): boolean {
        return this.store.delete(key);
    }

    clear(): void {
        this.store.clear();
    }

    // Bonus: cleanup expired entries
    cleanup(): number {
        let count = 0;
        const now = Date.now();

        for (const [key, entry] of this.store.entries()) {
            if (entry.expiresAt && now > entry.expiresAt) {
                this.store.delete(key);
                count++;
            }
        }

        return count;
    }
}

// Test
interface User {
    id: number;
    name: string;
}

const userCache = new Cache<User>(5000);  // 5 second default TTL

userCache.set("user:1", { id: 1, name: "John" });
userCache.set("user:2", { id: 2, name: "Jane" }, 10000);  // 10 second TTL

console.log(userCache.get("user:1"));  // { id: 1, name: "John" }
console.log(userCache.has("user:2"));  // true
```

### Exercise 3: Generic Result Type

```typescript
// Create a Result type similar to Rust's Result<T, E>
// Requirements:
// - Result<T, E> can be either Ok<T> or Err<E>
// - Methods: isOk(), isErr(), unwrap(), unwrapOr(default), map(), mapErr()

// Your code here
```

**Solution:**
```typescript
class Ok<T> {
    readonly ok = true;
    readonly err = false;

    constructor(public readonly value: T) {}

    isOk(): this is Ok<T> {
        return true;
    }

    isErr(): this is Err<never> {
        return false;
    }

    unwrap(): T {
        return this.value;
    }

    unwrapOr(_default: T): T {
        return this.value;
    }

    map<U>(fn: (value: T) => U): Result<U, never> {
        return new Ok(fn(this.value));
    }

    mapErr<F>(_fn: (error: never) => F): Result<T, F> {
        return this as unknown as Result<T, F>;
    }
}

class Err<E> {
    readonly ok = false;
    readonly err = true;

    constructor(public readonly error: E) {}

    isOk(): this is Ok<never> {
        return false;
    }

    isErr(): this is Err<E> {
        return true;
    }

    unwrap(): never {
        throw new Error(`Called unwrap on Err: ${this.error}`);
    }

    unwrapOr<T>(defaultValue: T): T {
        return defaultValue;
    }

    map<U>(_fn: (value: never) => U): Result<U, E> {
        return this as unknown as Result<U, E>;
    }

    mapErr<F>(fn: (error: E) => F): Result<never, F> {
        return new Err(fn(this.error));
    }
}

type Result<T, E> = Ok<T> | Err<E>;

// Helper functions
function ok<T>(value: T): Ok<T> {
    return new Ok(value);
}

function err<E>(error: E): Err<E> {
    return new Err(error);
}

// Usage
function divide(a: number, b: number): Result<number, string> {
    if (b === 0) {
        return err("Division by zero");
    }
    return ok(a / b);
}

const result1 = divide(10, 2);
if (result1.isOk()) {
    console.log(`Result: ${result1.value}`);  // Result: 5
}

const result2 = divide(10, 0);
console.log(result2.unwrapOr(0));  // 0

const doubled = divide(10, 2).map(n => n * 2);
console.log(doubled.unwrap());  // 10
```

---

## Key Takeaways

1. **Function types** - Type parameters, return types, optional/default parameters
2. **Function overloads** - Multiple signatures for different input types
3. **Generics preserve types** - Unlike `any`, generics maintain type information
4. **Generic constraints** - Use `extends` to limit what types can be used
5. **Generic interfaces/classes** - Reusable type-safe containers
6. **Conditional types** - Types that depend on other types
7. **Mapped types** - Transform existing types into new ones
8. **Utility types** - Built-in generics like `Partial`, `Pick`, `Omit`

---

## Self-Check Questions

1. When should you use generics instead of `any`?
2. What does the `extends` keyword do in generic constraints?
3. How do function overloads work in TypeScript?
4. What's the difference between `T | undefined` and `T?` in parameters?
5. When would you use `infer` in conditional types?
6. How do you create a generic type with a default?

---

**Next Lesson:** [Day 5 - Classes & Modules](./day-05-classes-modules.md)
