# Day 3: Interfaces & Type Aliases

## Type Aliases

Create custom names for types:

```typescript
// Basic type alias
type ID = string | number;
type Username = string;

let userId: ID = "abc123";
let anotherId: ID = 12345;

// Object type alias
type User = {
    id: ID;
    name: string;
    email: string;
};

const user: User = {
    id: 1,
    name: "John",
    email: "john@example.com"
};

// Function type alias
type Greeting = (name: string) => string;

const greet: Greeting = (name) => `Hello, ${name}!`;

// Array type alias
type StringArray = string[];
type NumberArray = Array<number>;

// Union type alias
type Status = "pending" | "active" | "completed";
type Result<T> = T | null;

// Complex type alias
type ApiResponse<T> = {
    data: T;
    status: number;
    message: string;
    timestamp: Date;
};
```

---

## Interfaces

Define object shapes:

```typescript
// Basic interface
interface User {
    id: number;
    name: string;
    email: string;
}

const user: User = {
    id: 1,
    name: "John",
    email: "john@example.com"
};

// Optional properties
interface Config {
    host: string;
    port: number;
    secure?: boolean;  // Optional
    timeout?: number;  // Optional
}

const config: Config = {
    host: "localhost",
    port: 3000
    // secure and timeout are optional
};

// Readonly properties
interface Point {
    readonly x: number;
    readonly y: number;
}

const point: Point = { x: 10, y: 20 };
// point.x = 5;  // Error: Cannot assign to 'x' because it is a read-only property

// Index signatures
interface StringMap {
    [key: string]: string;
}

const map: StringMap = {
    name: "John",
    city: "New York",
    country: "USA"
};

// Mixed index signature
interface MixedMap {
    [key: string]: string | number;
    length: number;  // Specific property
}
```

### Interface Methods

```typescript
interface Calculator {
    // Method signature
    add(a: number, b: number): number;
    subtract(a: number, b: number): number;

    // Optional method
    multiply?(a: number, b: number): number;

    // Property function
    divide: (a: number, b: number) => number;
}

const calc: Calculator = {
    add(a, b) { return a + b; },
    subtract(a, b) { return a - b; },
    divide: (a, b) => a / b
};

console.log(calc.add(5, 3));       // 8
console.log(calc.subtract(10, 4)); // 6
console.log(calc.divide(20, 4));   // 5
```

### Interface Extension

```typescript
// Extend single interface
interface Person {
    name: string;
    age: number;
}

interface Employee extends Person {
    employeeId: number;
    department: string;
}

const employee: Employee = {
    name: "John",
    age: 30,
    employeeId: 12345,
    department: "Engineering"
};

// Extend multiple interfaces
interface Printable {
    print(): void;
}

interface Loggable {
    log(): void;
}

interface Document extends Printable, Loggable {
    title: string;
    content: string;
}

const doc: Document = {
    title: "Report",
    content: "...",
    print() { console.log("Printing..."); },
    log() { console.log("Logging..."); }
};

// Override in extension
interface Animal {
    name: string;
    makeSound(): void;
}

interface Dog extends Animal {
    breed: string;
    makeSound(): void;  // Can redeclare (must be compatible)
}
```

### Interface Merging (Declaration Merging)

```typescript
// Same name = merged
interface User {
    name: string;
}

interface User {
    age: number;
}

interface User {
    email: string;
}

// All properties combined
const user: User = {
    name: "John",
    age: 30,
    email: "john@example.com"
};

// Useful for extending library types
interface Window {
    myCustomProperty: string;
}

// Now Window has myCustomProperty
window.myCustomProperty = "custom";
```

---

## Type Aliases vs Interfaces

### When to Use Each

```typescript
// INTERFACES - Best for:

// 1. Object shapes (especially for public APIs)
interface User {
    id: number;
    name: string;
}

// 2. Class implementations
interface Printable {
    print(): void;
}

class Document implements Printable {
    print() { console.log("Printing..."); }
}

// 3. Extension/inheritance
interface Employee extends User {
    department: string;
}

// 4. Declaration merging
interface Config { timeout: number; }
interface Config { retries: number; }


// TYPE ALIASES - Best for:

// 1. Union types
type Status = "pending" | "active" | "completed";

// 2. Intersection types
type PersonWithAddress = Person & Address;

// 3. Function types
type Handler = (event: Event) => void;

// 4. Tuples
type Coordinate = [number, number];

// 5. Primitive aliases
type ID = string | number;

// 6. Mapped types
type Readonly<T> = { readonly [K in keyof T]: T[K] };

// 7. Conditional types
type NonNullable<T> = T extends null | undefined ? never : T;
```

### Key Differences

```typescript
// 1. Declaration merging
// Interface: YES
interface A { x: number; }
interface A { y: number; }
// A has both x and y

// Type: NO
type B = { x: number; };
// type B = { y: number; };  // Error: Duplicate identifier

// 2. Extends vs Intersection
// Interface extends
interface Animal { name: string; }
interface Dog extends Animal { breed: string; }

// Type intersection
type Animal2 = { name: string; };
type Dog2 = Animal2 & { breed: string; };

// 3. Implements
interface Printable { print(): void; }
class Doc implements Printable {  // Works
    print() {}
}

type Printable2 = { print(): void; };
class Doc2 implements Printable2 {  // Also works
    print() {}
}

// 4. Computed properties
type Keys = "a" | "b" | "c";
type Obj = { [K in Keys]: string };  // Type can do this

// interface Obj2 { [K in Keys]: string };  // Interface cannot
```

---

## Function Types

### Type Alias for Functions

```typescript
// Basic function type
type Greeting = (name: string) => string;

const greet: Greeting = (name) => `Hello, ${name}!`;

// Function with multiple parameters
type Calculator = (a: number, b: number) => number;

const add: Calculator = (a, b) => a + b;
const multiply: Calculator = (a, b) => a * b;

// Function returning void
type Logger = (message: string) => void;

// Function with optional parameters
type Formatter = (value: string, uppercase?: boolean) => string;

const format: Formatter = (value, uppercase = false) => {
    return uppercase ? value.toUpperCase() : value;
};

// Function with rest parameters
type Joiner = (...strings: string[]) => string;

const join: Joiner = (...strings) => strings.join(", ");
```

### Interface for Functions

```typescript
// Call signature
interface Greeting {
    (name: string): string;
}

const greet: Greeting = (name) => `Hello, ${name}!`;

// With properties (callable object)
interface Counter {
    (start: number): number;
    count: number;
    reset(): void;
}

function createCounter(): Counter {
    const counter = ((start: number) => {
        counter.count = start;
        return counter.count;
    }) as Counter;

    counter.count = 0;
    counter.reset = () => { counter.count = 0; };

    return counter;
}

const myCounter = createCounter();
myCounter(10);     // Start at 10
myCounter.count;   // 10
myCounter.reset(); // Reset to 0
```

### Overloaded Functions

```typescript
// Function overloads
interface StringProcessor {
    (value: string): string;
    (value: string[]): string[];
}

const process: StringProcessor = (value: string | string[]): any => {
    if (Array.isArray(value)) {
        return value.map(s => s.toUpperCase());
    }
    return value.toUpperCase();
};

process("hello");           // "HELLO"
process(["a", "b", "c"]);   // ["A", "B", "C"]

// Type alias version
type Overloaded = {
    (x: number): number;
    (x: string): string;
};
```

---

## Generic Interfaces and Types

### Generic Type Aliases

```typescript
// Generic type alias
type Container<T> = {
    value: T;
};

const numberContainer: Container<number> = { value: 42 };
const stringContainer: Container<string> = { value: "hello" };

// Multiple type parameters
type Pair<K, V> = {
    key: K;
    value: V;
};

const pair: Pair<string, number> = { key: "age", value: 30 };

// Constrained generic
type Lengthwise<T extends { length: number }> = {
    value: T;
    length: number;
};

const arr: Lengthwise<string[]> = {
    value: ["a", "b"],
    length: 2
};

// Default type parameter
type Response<T = any> = {
    data: T;
    status: number;
};

const response: Response = { data: "anything", status: 200 };
const typedResponse: Response<User> = { data: user, status: 200 };
```

### Generic Interfaces

```typescript
// Generic interface
interface Box<T> {
    value: T;
    getValue(): T;
    setValue(value: T): void;
}

class NumberBox implements Box<number> {
    constructor(public value: number) {}

    getValue(): number {
        return this.value;
    }

    setValue(value: number): void {
        this.value = value;
    }
}

// Generic interface with methods
interface Repository<T> {
    getById(id: number): T | undefined;
    getAll(): T[];
    add(item: T): void;
    update(id: number, item: T): void;
    delete(id: number): void;
}

// Implementation
class UserRepository implements Repository<User> {
    private users: User[] = [];

    getById(id: number): User | undefined {
        return this.users.find(u => u.id === id);
    }

    getAll(): User[] {
        return [...this.users];
    }

    add(user: User): void {
        this.users.push(user);
    }

    update(id: number, user: User): void {
        const index = this.users.findIndex(u => u.id === id);
        if (index !== -1) {
            this.users[index] = user;
        }
    }

    delete(id: number): void {
        this.users = this.users.filter(u => u.id !== id);
    }
}
```

---

## Utility Types with Interfaces

### Built-in Utility Types

```typescript
interface User {
    id: number;
    name: string;
    email: string;
    age: number;
}

// Partial - all properties optional
type PartialUser = Partial<User>;
const partial: PartialUser = { name: "John" };

// Required - all properties required
type RequiredUser = Required<User>;

// Readonly - all properties readonly
type ReadonlyUser = Readonly<User>;
const frozen: ReadonlyUser = { id: 1, name: "John", email: "john@email.com", age: 30 };
// frozen.name = "Jane";  // Error!

// Pick - select specific properties
type UserName = Pick<User, "id" | "name">;
const nameOnly: UserName = { id: 1, name: "John" };

// Omit - exclude specific properties
type UserWithoutEmail = Omit<User, "email">;
const noEmail: UserWithoutEmail = { id: 1, name: "John", age: 30 };

// Record - create object type with specific keys
type UserRecord = Record<string, User>;
const users: UserRecord = {
    john: { id: 1, name: "John", email: "john@email.com", age: 30 }
};

// Extract - extract from union
type Status = "pending" | "active" | "completed" | "cancelled";
type ActiveStatus = Extract<Status, "active" | "completed">;
// Type: "active" | "completed"

// Exclude - exclude from union
type InactiveStatus = Exclude<Status, "active">;
// Type: "pending" | "completed" | "cancelled"

// NonNullable - remove null and undefined
type MaybeString = string | null | undefined;
type DefiniteString = NonNullable<MaybeString>;
// Type: string
```

### Creating Custom Utility Types

```typescript
// Make specific properties optional
type PartialBy<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>;

interface User {
    id: number;
    name: string;
    email: string;
}

type UserWithOptionalEmail = PartialBy<User, "email">;
// { id: number; name: string; email?: string }

// Make specific properties required
type RequiredBy<T, K extends keyof T> = Omit<T, K> & Required<Pick<T, K>>;

// Deep partial
type DeepPartial<T> = {
    [P in keyof T]?: T[P] extends object ? DeepPartial<T[P]> : T[P];
};

interface NestedConfig {
    server: {
        host: string;
        port: number;
    };
    database: {
        name: string;
        user: string;
    };
}

const partial: DeepPartial<NestedConfig> = {
    server: { host: "localhost" }  // port is optional
};

// Nullable
type Nullable<T> = T | null;
type NullableUser = Nullable<User>;
```

---

## Practical Patterns

### API Response Types

```typescript
// Generic API response
interface ApiResponse<T> {
    data: T;
    status: number;
    message: string;
    timestamp: string;
}

interface PaginatedResponse<T> extends ApiResponse<T[]> {
    pagination: {
        page: number;
        limit: number;
        total: number;
        totalPages: number;
    };
}

interface ErrorResponse {
    error: {
        code: string;
        message: string;
        details?: Record<string, string>;
    };
    status: number;
}

// Type for fetch result
type FetchResult<T> = ApiResponse<T> | ErrorResponse;

// Type guard
function isErrorResponse(response: FetchResult<any>): response is ErrorResponse {
    return "error" in response;
}

// Usage
async function fetchUsers(): Promise<FetchResult<User[]>> {
    const response = await fetch("/api/users");
    return response.json();
}

const result = await fetchUsers();
if (isErrorResponse(result)) {
    console.error(result.error.message);
} else {
    console.log(result.data);
}
```

### Form Types

```typescript
// Form field interface
interface FormField<T> {
    value: T;
    error?: string;
    touched: boolean;
    dirty: boolean;
}

// Generic form state
type FormState<T> = {
    [K in keyof T]: FormField<T[K]>;
};

// Example form
interface LoginForm {
    email: string;
    password: string;
    rememberMe: boolean;
}

type LoginFormState = FormState<LoginForm>;
// {
//     email: FormField<string>;
//     password: FormField<string>;
//     rememberMe: FormField<boolean>;
// }

const formState: LoginFormState = {
    email: { value: "", touched: false, dirty: false },
    password: { value: "", touched: false, dirty: false },
    rememberMe: { value: false, touched: false, dirty: false }
};
```

### Event Handler Types

```typescript
// Custom event types
interface CustomEvent<T = any> {
    type: string;
    payload: T;
    timestamp: Date;
}

type EventHandler<T = any> = (event: CustomEvent<T>) => void;

interface EventEmitter {
    on<T>(event: string, handler: EventHandler<T>): void;
    off<T>(event: string, handler: EventHandler<T>): void;
    emit<T>(event: string, payload: T): void;
}

// Typed event map
interface AppEvents {
    login: { userId: number; timestamp: Date };
    logout: { userId: number };
    error: { message: string; code: number };
}

type TypedEventHandler<K extends keyof AppEvents> = (event: CustomEvent<AppEvents[K]>) => void;

interface TypedEventEmitter {
    on<K extends keyof AppEvents>(event: K, handler: TypedEventHandler<K>): void;
    emit<K extends keyof AppEvents>(event: K, payload: AppEvents[K]): void;
}
```

---

## Practice Exercises

### Exercise 1: Define an E-commerce Interface

```typescript
// Create interfaces for:
// 1. Product (id, name, price, description, category)
// 2. CartItem (product, quantity)
// 3. Cart (items, total, addItem method, removeItem method)

// Your code here
```

**Solution:**
```typescript
interface Product {
    id: number;
    name: string;
    price: number;
    description: string;
    category: string;
}

interface CartItem {
    product: Product;
    quantity: number;
}

interface Cart {
    items: CartItem[];
    readonly total: number;
    addItem(product: Product, quantity?: number): void;
    removeItem(productId: number): void;
    clear(): void;
}

// Implementation
class ShoppingCart implements Cart {
    items: CartItem[] = [];

    get total(): number {
        return this.items.reduce(
            (sum, item) => sum + item.product.price * item.quantity,
            0
        );
    }

    addItem(product: Product, quantity = 1): void {
        const existing = this.items.find(item => item.product.id === product.id);
        if (existing) {
            existing.quantity += quantity;
        } else {
            this.items.push({ product, quantity });
        }
    }

    removeItem(productId: number): void {
        this.items = this.items.filter(item => item.product.id !== productId);
    }

    clear(): void {
        this.items = [];
    }
}
```

### Exercise 2: Generic Data Service

```typescript
// Create a generic DataService interface that:
// - Has CRUD operations
// - Works with any entity type
// - Entities must have an 'id' property

// Your code here
```

**Solution:**
```typescript
interface Entity {
    id: number;
}

interface DataService<T extends Entity> {
    getById(id: number): Promise<T | null>;
    getAll(): Promise<T[]>;
    create(data: Omit<T, "id">): Promise<T>;
    update(id: number, data: Partial<Omit<T, "id">>): Promise<T | null>;
    delete(id: number): Promise<boolean>;
}

// Example entity
interface User extends Entity {
    name: string;
    email: string;
}

// Implementation
class UserService implements DataService<User> {
    private users: User[] = [];
    private nextId = 1;

    async getById(id: number): Promise<User | null> {
        return this.users.find(u => u.id === id) ?? null;
    }

    async getAll(): Promise<User[]> {
        return [...this.users];
    }

    async create(data: Omit<User, "id">): Promise<User> {
        const user: User = { id: this.nextId++, ...data };
        this.users.push(user);
        return user;
    }

    async update(id: number, data: Partial<Omit<User, "id">>): Promise<User | null> {
        const user = await this.getById(id);
        if (user) {
            Object.assign(user, data);
            return user;
        }
        return null;
    }

    async delete(id: number): Promise<boolean> {
        const index = this.users.findIndex(u => u.id === id);
        if (index !== -1) {
            this.users.splice(index, 1);
            return true;
        }
        return false;
    }
}
```

---

## Key Takeaways

1. **Type aliases** can represent any type (primitives, unions, tuples, functions)
2. **Interfaces** are best for object shapes and can be extended/merged
3. **Interfaces merge**, type aliases don't
4. **Use interfaces** for public APIs and class contracts
5. **Use type aliases** for unions, intersections, and mapped types
6. **Generic interfaces** create reusable, type-safe abstractions
7. **Utility types** like `Partial`, `Pick`, `Omit` transform types
8. **Index signatures** allow dynamic property names

---

**Next Lesson:** [Day 4 - Functions & Generics](./day-04-functions-generics.md)
