# Day 6-7: Advanced Types in TypeScript

## Conditional Types

Conditional types select types based on conditions, similar to ternary expressions.

### Basic Conditional Types

```typescript
// Syntax: T extends U ? X : Y
// "If T is assignable to U, then X, else Y"

type IsString<T> = T extends string ? true : false;

type Test1 = IsString<string>;    // true
type Test2 = IsString<number>;    // false
type Test3 = IsString<"hello">;   // true (literal extends string)

// Practical example: Nullable types
type Nullable<T> = T extends null | undefined ? never : T | null;

type StringOrNull = Nullable<string>;  // string | null
type AlreadyNull = Nullable<null>;     // never

// Extract array element type
type ElementType<T> = T extends (infer E)[] ? E : never;

type StringElement = ElementType<string[]>;   // string
type NumberElement = ElementType<number[]>;   // number
type NotArray = ElementType<string>;          // never

// Flatten nested arrays
type Flatten<T> = T extends (infer E)[] ? Flatten<E> : T;

type Flat1 = Flatten<number[]>;        // number
type Flat2 = Flatten<number[][]>;      // number
type Flat3 = Flatten<number[][][]>;    // number
```

### Distributive Conditional Types

```typescript
// When T is a union, conditional type distributes over union members

type ToArray<T> = T extends unknown ? T[] : never;

// Distributes over each member
type StrOrNumArray = ToArray<string | number>;
// = ToArray<string> | ToArray<number>
// = string[] | number[]

// Prevent distribution with tuple
type ToArrayNonDist<T> = [T] extends [unknown] ? T[] : never;

type Combined = ToArrayNonDist<string | number>;
// = (string | number)[]

// Practical example: Filter union types
type FilterOut<T, U> = T extends U ? never : T;

type WithoutNull = FilterOut<string | number | null, null>;
// = string | number

type OnlyStrings = FilterOut<string | number | boolean, number | boolean>;
// = string

// Built-in Exclude is similar
type Excluded = Exclude<string | number | boolean, boolean>;
// = string | number
```

### Inferring Types with `infer`

```typescript
// Extract return type of function
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

function getUser() {
    return { id: 1, name: "John" };
}

type User = ReturnType<typeof getUser>;
// { id: number; name: string }

// Extract parameter types
type Parameters<T> = T extends (...args: infer P) => any ? P : never;

function createUser(name: string, age: number): void {}

type CreateUserParams = Parameters<typeof createUser>;
// [string, number]

// Extract first parameter
type FirstParam<T> = T extends (first: infer F, ...args: any[]) => any ? F : never;

type First = FirstParam<typeof createUser>;  // string

// Infer Promise value
type Awaited<T> = T extends Promise<infer U> ? Awaited<U> : T;

type Value1 = Awaited<Promise<string>>;           // string
type Value2 = Awaited<Promise<Promise<number>>>;  // number
type Value3 = Awaited<string>;                    // string

// Infer array/tuple types
type First<T> = T extends [infer F, ...any[]] ? F : never;
type Last<T> = T extends [...any[], infer L] ? L : never;
type Tail<T> = T extends [any, ...infer Rest] ? Rest : never;

type Arr = [1, 2, 3, 4, 5];
type F = First<Arr>;  // 1
type L = Last<Arr>;   // 5
type T = Tail<Arr>;   // [2, 3, 4, 5]

// Infer from template literal
type ExtractUserId<T> = T extends `user_${infer Id}` ? Id : never;

type UserId = ExtractUserId<"user_123">;    // "123"
type NotUser = ExtractUserId<"admin_123">;  // never

// Complex extraction
type ParseRoute<T> = T extends `/${infer Resource}/${infer Id}`
    ? { resource: Resource; id: Id }
    : T extends `/${infer Resource}`
    ? { resource: Resource }
    : never;

type Route1 = ParseRoute<"/users/123">;   // { resource: "users"; id: "123" }
type Route2 = ParseRoute<"/products">;    // { resource: "products" }
```

---

## Mapped Types

Mapped types transform existing types by iterating over their properties.

### Basic Mapped Types

```typescript
// Syntax: { [K in Keys]: Type }

// Make all properties optional
type Partial<T> = {
    [K in keyof T]?: T[K];
};

// Make all properties required
type Required<T> = {
    [K in keyof T]-?: T[K];  // -? removes optional
};

// Make all properties readonly
type Readonly<T> = {
    readonly [K in keyof T]: T[K];
};

// Make all properties mutable
type Mutable<T> = {
    -readonly [K in keyof T]: T[K];  // -readonly removes readonly
};

// Example
interface User {
    readonly id: number;
    name: string;
    email?: string;
}

type PartialUser = Partial<User>;
// { id?: number; name?: string; email?: string }

type RequiredUser = Required<User>;
// { id: number; name: string; email: string }

type MutableUser = Mutable<User>;
// { id: number; name: string; email?: string }
```

### Key Remapping

```typescript
// Rename keys with 'as' clause
type Getters<T> = {
    [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

interface Person {
    name: string;
    age: number;
}

type PersonGetters = Getters<Person>;
// { getName: () => string; getAge: () => number }

// Setters
type Setters<T> = {
    [K in keyof T as `set${Capitalize<string & K>}`]: (value: T[K]) => void;
};

type PersonSetters = Setters<Person>;
// { setName: (value: string) => void; setAge: (value: number) => void }

// Filter keys by type
type FilterByType<T, ValueType> = {
    [K in keyof T as T[K] extends ValueType ? K : never]: T[K];
};

interface Mixed {
    id: number;
    name: string;
    age: number;
    active: boolean;
}

type NumberProps = FilterByType<Mixed, number>;
// { id: number; age: number }

type StringProps = FilterByType<Mixed, string>;
// { name: string }

// Exclude specific keys
type OmitByKey<T, K extends keyof T> = {
    [P in keyof T as P extends K ? never : P]: T[P];
};

type WithoutId = OmitByKey<Mixed, "id">;
// { name: string; age: number; active: boolean }
```

### Recursive Mapped Types

```typescript
// Deep partial - makes nested objects optional too
type DeepPartial<T> = {
    [K in keyof T]?: T[K] extends object ? DeepPartial<T[K]> : T[K];
};

interface Config {
    server: {
        host: string;
        port: number;
        ssl: {
            enabled: boolean;
            cert: string;
        };
    };
    database: {
        host: string;
        name: string;
    };
}

type PartialConfig = DeepPartial<Config>;
// All nested properties are also optional

const config: PartialConfig = {
    server: {
        port: 3000
        // host is optional, ssl is optional
    }
    // database is optional
};

// Deep readonly
type DeepReadonly<T> = {
    readonly [K in keyof T]: T[K] extends object ? DeepReadonly<T[K]> : T[K];
};

type ReadonlyConfig = DeepReadonly<Config>;
// All nested properties are also readonly

// Deep required
type DeepRequired<T> = {
    [K in keyof T]-?: T[K] extends object ? DeepRequired<T[K]> : T[K];
};
```

---

## Template Literal Types

Template literal types combine literal types with string templates.

### Basic Template Literals

```typescript
// Combining literals
type Greeting = `Hello, ${string}`;

const g1: Greeting = "Hello, World";    // OK
const g2: Greeting = "Hello, TypeScript"; // OK
// const g3: Greeting = "Hi, World";    // Error

// Union expansion
type Color = "red" | "green" | "blue";
type Size = "small" | "medium" | "large";

type ColorSize = `${Color}-${Size}`;
// "red-small" | "red-medium" | "red-large" |
// "green-small" | "green-medium" | "green-large" |
// "blue-small" | "blue-medium" | "blue-large"

// HTTP methods
type HttpMethod = "GET" | "POST" | "PUT" | "DELETE";
type Endpoint = "/users" | "/products" | "/orders";

type ApiRoute = `${HttpMethod} ${Endpoint}`;
// "GET /users" | "GET /products" | ... (12 combinations)

// Event names
type EventType = "click" | "focus" | "blur";
type EventHandler = `on${Capitalize<EventType>}`;
// "onClick" | "onFocus" | "onBlur"
```

### String Manipulation Types

```typescript
// Built-in string manipulation types
type Upper = Uppercase<"hello">;       // "HELLO"
type Lower = Lowercase<"HELLO">;       // "hello"
type Cap = Capitalize<"hello">;        // "Hello"
type Uncap = Uncapitalize<"Hello">;    // "hello"

// CamelCase to snake_case
type CamelToSnake<S extends string> = S extends `${infer C}${infer Rest}`
    ? C extends Uppercase<C>
        ? `_${Lowercase<C>}${CamelToSnake<Rest>}`
        : `${C}${CamelToSnake<Rest>}`
    : S;

type Snake = CamelToSnake<"getUserById">;
// "get_user_by_id"

// Extract parts
type ExtractDomain<T extends string> =
    T extends `${string}@${infer Domain}` ? Domain : never;

type Domain = ExtractDomain<"user@example.com">;  // "example.com"

// Parse paths
type PathParams<Path extends string> =
    Path extends `${string}:${infer Param}/${infer Rest}`
        ? Param | PathParams<`/${Rest}`>
        : Path extends `${string}:${infer Param}`
            ? Param
            : never;

type Params = PathParams<"/users/:userId/posts/:postId">;
// "userId" | "postId"
```

### Practical Template Literal Patterns

```typescript
// CSS unit types
type CSSUnit = "px" | "em" | "rem" | "%" | "vh" | "vw";
type CSSValue = `${number}${CSSUnit}`;

const width: CSSValue = "100px";
const height: CSSValue = "50vh";
// const invalid: CSSValue = "100";  // Error

// UUID pattern
type HexChar = "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9"
             | "a" | "b" | "c" | "d" | "e" | "f";

// Simplified UUID check (full would be complex)
type UUIDLike = `${string}-${string}-${string}-${string}-${string}`;

// Environment variables
type EnvKey = `${Uppercase<string>}_${Uppercase<string>}`;
// Matches: "DATABASE_URL", "API_KEY", etc.

// BEM naming convention
type Block = string;
type Element = string;
type Modifier = string;

type BEMClass = `${Block}__${Element}--${Modifier}`
              | `${Block}__${Element}`
              | `${Block}--${Modifier}`
              | Block;

const className1: BEMClass = "button__icon--large";
const className2: BEMClass = "card__header";
```

---

## Utility Types Deep Dive

### Built-in Utility Types

```typescript
// Partial<T> - Make all properties optional
interface User {
    id: number;
    name: string;
    email: string;
}

type PartialUser = Partial<User>;
// { id?: number; name?: string; email?: string }

function updateUser(id: number, updates: Partial<User>): User {
    // Can pass any subset of User properties
}

// Required<T> - Make all properties required
interface Config {
    host?: string;
    port?: number;
}

type RequiredConfig = Required<Config>;
// { host: string; port: number }

// Readonly<T> - Make all properties readonly
type ReadonlyUser = Readonly<User>;

const user: ReadonlyUser = { id: 1, name: "John", email: "john@example.com" };
// user.name = "Jane";  // Error

// Pick<T, K> - Select specific properties
type UserName = Pick<User, "id" | "name">;
// { id: number; name: string }

// Omit<T, K> - Exclude specific properties
type UserWithoutEmail = Omit<User, "email">;
// { id: number; name: string }

// Exclude<T, U> - Exclude types from union
type StringOrNumber = string | number | boolean;
type OnlyStringOrNumber = Exclude<StringOrNumber, boolean>;
// string | number

// Extract<T, U> - Extract types from union
type OnlyBoolean = Extract<StringOrNumber, boolean>;
// boolean

// NonNullable<T> - Remove null and undefined
type MaybeString = string | null | undefined;
type DefinitelyString = NonNullable<MaybeString>;
// string
```

### Record and Key Types

```typescript
// Record<K, V> - Create object type with keys K and values V
type UserRoles = Record<string, boolean>;
// { [key: string]: boolean }

const roles: UserRoles = {
    admin: true,
    editor: false,
    viewer: true
};

// With union keys
type Role = "admin" | "editor" | "viewer";
type Permissions = Record<Role, string[]>;

const perms: Permissions = {
    admin: ["read", "write", "delete"],
    editor: ["read", "write"],
    viewer: ["read"]
};

// Status map
type Status = "pending" | "active" | "completed";
type StatusLabels = Record<Status, string>;

const labels: StatusLabels = {
    pending: "Waiting",
    active: "In Progress",
    completed: "Done"
};

// Keyof and indexed access
type UserKeys = keyof User;           // "id" | "name" | "email"
type NameType = User["name"];         // string
type IdOrName = User["id" | "name"];  // number | string
```

### Function Utility Types

```typescript
// ReturnType<T> - Get return type of function
function getUser() {
    return { id: 1, name: "John", email: "john@example.com" };
}

type UserFromFn = ReturnType<typeof getUser>;
// { id: number; name: string; email: string }

// Parameters<T> - Get parameter types as tuple
function createUser(name: string, age: number, email: string): void {}

type CreateUserParams = Parameters<typeof createUser>;
// [string, number, string]

// ConstructorParameters<T> - Get constructor parameters
class Person {
    constructor(public name: string, public age: number) {}
}

type PersonParams = ConstructorParameters<typeof Person>;
// [string, number]

// InstanceType<T> - Get instance type of class
type PersonInstance = InstanceType<typeof Person>;
// Person

// ThisParameterType<T> - Get 'this' type
function greet(this: User): string {
    return `Hello, ${this.name}`;
}

type GreetThis = ThisParameterType<typeof greet>;
// User

// OmitThisParameter<T> - Remove 'this' from function type
type GreetWithoutThis = OmitThisParameter<typeof greet>;
// () => string
```

### Custom Utility Types

```typescript
// Make specific properties optional
type PartialBy<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>;

interface Post {
    id: number;
    title: string;
    content: string;
    publishedAt: Date;
}

type DraftPost = PartialBy<Post, "id" | "publishedAt">;
// { title: string; content: string; id?: number; publishedAt?: Date }

// Make specific properties required
type RequiredBy<T, K extends keyof T> = Omit<T, K> & Required<Pick<T, K>>;

// Nullable version of type
type Nullable<T> = { [K in keyof T]: T[K] | null };

type NullableUser = Nullable<User>;
// { id: number | null; name: string | null; email: string | null }

// Get only function properties
type FunctionPropertyNames<T> = {
    [K in keyof T]: T[K] extends Function ? K : never;
}[keyof T];

type FunctionProperties<T> = Pick<T, FunctionPropertyNames<T>>;

class Service {
    url: string = "";
    fetch(): void {}
    post(data: object): void {}
}

type ServiceMethods = FunctionProperties<Service>;
// { fetch: () => void; post: (data: object) => void }

// Get non-function properties
type NonFunctionPropertyNames<T> = {
    [K in keyof T]: T[K] extends Function ? never : K;
}[keyof T];

type NonFunctionProperties<T> = Pick<T, NonFunctionPropertyNames<T>>;

type ServiceData = NonFunctionProperties<Service>;
// { url: string }
```

---

## Type Guards and Narrowing

### Type Predicates

```typescript
// Type predicate function
interface Cat {
    meow(): void;
    purr(): void;
}

interface Dog {
    bark(): void;
    fetch(): void;
}

type Pet = Cat | Dog;

// Custom type guard
function isCat(pet: Pet): pet is Cat {
    return "meow" in pet;
}

function isDog(pet: Pet): pet is Dog {
    return "bark" in pet;
}

function handlePet(pet: Pet) {
    if (isCat(pet)) {
        pet.meow();   // TypeScript knows it's Cat
        pet.purr();
    } else {
        pet.bark();   // TypeScript knows it's Dog
        pet.fetch();
    }
}

// Type guard for arrays
function isStringArray(arr: unknown): arr is string[] {
    return Array.isArray(arr) && arr.every(item => typeof item === "string");
}

function processData(data: unknown) {
    if (isStringArray(data)) {
        data.forEach(s => console.log(s.toUpperCase()));
    }
}

// Generic type guard
function isDefined<T>(value: T | null | undefined): value is T {
    return value !== null && value !== undefined;
}

const values = [1, null, 2, undefined, 3];
const definedValues = values.filter(isDefined);  // number[]
```

### Assertion Functions

```typescript
// Assertion function - throws if condition is false
function assertIsString(value: unknown): asserts value is string {
    if (typeof value !== "string") {
        throw new Error(`Expected string, got ${typeof value}`);
    }
}

function processValue(value: unknown) {
    assertIsString(value);
    // After assertion, value is string
    console.log(value.toUpperCase());
}

// Assert non-null
function assertDefined<T>(
    value: T | null | undefined,
    message?: string
): asserts value is T {
    if (value === null || value === undefined) {
        throw new Error(message ?? "Value is not defined");
    }
}

function getUser(id: number): User | null {
    // ...
    return null;
}

const user = getUser(1);
assertDefined(user, "User not found");
// user is now User (not User | null)
console.log(user.name);

// Generic assertion
function assert(condition: unknown, message?: string): asserts condition {
    if (!condition) {
        throw new Error(message ?? "Assertion failed");
    }
}

function divide(a: number, b: number): number {
    assert(b !== 0, "Cannot divide by zero");
    return a / b;
}
```

### Discriminated Unions

```typescript
// Using a common discriminant property
interface Square {
    kind: "square";
    size: number;
}

interface Rectangle {
    kind: "rectangle";
    width: number;
    height: number;
}

interface Circle {
    kind: "circle";
    radius: number;
}

type Shape = Square | Rectangle | Circle;

function getArea(shape: Shape): number {
    switch (shape.kind) {
        case "square":
            return shape.size ** 2;
        case "rectangle":
            return shape.width * shape.height;
        case "circle":
            return Math.PI * shape.radius ** 2;
    }
}

// Exhaustive check helper
function assertNever(value: never): never {
    throw new Error(`Unexpected value: ${value}`);
}

function getAreaSafe(shape: Shape): number {
    switch (shape.kind) {
        case "square":
            return shape.size ** 2;
        case "rectangle":
            return shape.width * shape.height;
        case "circle":
            return Math.PI * shape.radius ** 2;
        default:
            return assertNever(shape);  // Compile error if case is missing
    }
}

// API responses
interface SuccessResponse<T> {
    status: "success";
    data: T;
}

interface ErrorResponse {
    status: "error";
    error: {
        code: string;
        message: string;
    };
}

interface LoadingResponse {
    status: "loading";
}

type ApiResponse<T> = SuccessResponse<T> | ErrorResponse | LoadingResponse;

function handleResponse<T>(response: ApiResponse<T>) {
    switch (response.status) {
        case "success":
            console.log("Data:", response.data);
            break;
        case "error":
            console.error(`Error ${response.error.code}: ${response.error.message}`);
            break;
        case "loading":
            console.log("Loading...");
            break;
    }
}
```

---

## Advanced Generic Patterns

### Constrained Generic Factories

```typescript
// Factory with constructor constraint
interface Constructor<T = {}> {
    new (...args: any[]): T;
}

function createInstance<T>(ctor: Constructor<T>, ...args: any[]): T {
    return new ctor(...args);
}

class Person {
    constructor(public name: string, public age: number) {}
}

const person = createInstance(Person, "John", 30);

// Mixin pattern
type GConstructor<T = {}> = new (...args: any[]) => T;

function Timestamped<TBase extends GConstructor>(Base: TBase) {
    return class extends Base {
        timestamp = Date.now();
    };
}

function Tagged<TBase extends GConstructor>(Base: TBase) {
    return class extends Base {
        tag: string = "";
        setTag(tag: string) {
            this.tag = tag;
        }
    };
}

class User {
    constructor(public name: string) {}
}

const TimestampedUser = Timestamped(User);
const TaggedTimestampedUser = Tagged(Timestamped(User));

const user = new TaggedTimestampedUser("John");
user.setTag("premium");
console.log(user.timestamp);
```

### Variadic Tuple Types

```typescript
// Spread in tuple types
type Concat<T extends unknown[], U extends unknown[]> = [...T, ...U];

type Combined = Concat<[1, 2], [3, 4]>;
// [1, 2, 3, 4]

// Function with variable arguments
type Fn<Args extends unknown[], Return> = (...args: Args) => Return;

const add: Fn<[number, number], number> = (a, b) => a + b;
const concat: Fn<[string, string, string], string> = (a, b, c) => a + b + c;

// Prepend argument
type PrependArg<F extends (...args: any) => any, A> =
    F extends (...args: infer Args) => infer R
        ? (arg: A, ...args: Args) => R
        : never;

type OriginalFn = (a: number, b: string) => boolean;
type WithId = PrependArg<OriginalFn, { id: number }>;
// (arg: { id: number }, a: number, b: string) => boolean

// Curry type
type Curry<F extends (...args: any) => any> =
    F extends (...args: infer Args) => infer R
        ? Args extends [infer First, ...infer Rest]
            ? (arg: First) => Curry<(...args: Rest) => R>
            : R
        : never;

declare function curry<F extends (...args: any) => any>(fn: F): Curry<F>;

function add3(a: number, b: number, c: number): number {
    return a + b + c;
}

const curriedAdd = curry(add3);
// (arg: number) => (arg: number) => (arg: number) => number
const result = curriedAdd(1)(2)(3);  // 6
```

### Recursive Types

```typescript
// JSON type
type JSONValue =
    | string
    | number
    | boolean
    | null
    | JSONValue[]
    | { [key: string]: JSONValue };

const json: JSONValue = {
    name: "John",
    age: 30,
    active: true,
    tags: ["admin", "user"],
    metadata: {
        created: "2024-01-01",
        nested: {
            deep: true
        }
    }
};

// File system tree
interface FileNode {
    name: string;
    content: string;
}

interface DirectoryNode {
    name: string;
    children: FileSystemNode[];
}

type FileSystemNode = FileNode | DirectoryNode;

const fileSystem: DirectoryNode = {
    name: "root",
    children: [
        { name: "file1.txt", content: "Hello" },
        {
            name: "folder",
            children: [
                { name: "file2.txt", content: "World" }
            ]
        }
    ]
};

// Linked list
interface ListNode<T> {
    value: T;
    next: ListNode<T> | null;
}

const list: ListNode<number> = {
    value: 1,
    next: {
        value: 2,
        next: {
            value: 3,
            next: null
        }
    }
};

// Binary tree
interface TreeNode<T> {
    value: T;
    left: TreeNode<T> | null;
    right: TreeNode<T> | null;
}
```

---

## Practice Exercises

### Exercise 1: Deep Pick

```typescript
// Create DeepPick that can pick nested properties
// DeepPick<{ a: { b: { c: string } } }, "a.b.c"> should be { a: { b: { c: string } } }

// Your code here
```

**Solution:**
```typescript
type Split<S extends string, D extends string> =
    S extends `${infer T}${D}${infer U}` ? [T, ...Split<U, D>] : [S];

type DeepPick<T, Path extends string> = Path extends `${infer Key}.${infer Rest}`
    ? Key extends keyof T
        ? { [K in Key]: DeepPick<T[K], Rest> }
        : never
    : Path extends keyof T
        ? { [K in Path]: T[K] }
        : never;

// Test
interface DeepObject {
    user: {
        name: string;
        address: {
            city: string;
            country: string;
        };
    };
    meta: {
        version: number;
    };
}

type PickedCity = DeepPick<DeepObject, "user.address.city">;
// { user: { address: { city: string } } }

type PickedName = DeepPick<DeepObject, "user.name">;
// { user: { name: string } }
```

### Exercise 2: Type-Safe Event Emitter

```typescript
// Create a type-safe event emitter type
// Events should be strictly typed

// Your code here
```

**Solution:**
```typescript
type EventMap = Record<string, any>;

type EventKey<T extends EventMap> = string & keyof T;
type EventPayload<T extends EventMap, K extends EventKey<T>> = T[K];
type EventCallback<T extends EventMap, K extends EventKey<T>> =
    (payload: EventPayload<T, K>) => void;

interface TypedEventEmitter<T extends EventMap> {
    on<K extends EventKey<T>>(event: K, callback: EventCallback<T, K>): void;
    off<K extends EventKey<T>>(event: K, callback: EventCallback<T, K>): void;
    emit<K extends EventKey<T>>(event: K, payload: EventPayload<T, K>): void;
    once<K extends EventKey<T>>(event: K, callback: EventCallback<T, K>): void;
}

// Implementation
class EventEmitter<T extends EventMap> implements TypedEventEmitter<T> {
    private handlers: { [K in keyof T]?: EventCallback<T, K>[] } = {};

    on<K extends EventKey<T>>(event: K, callback: EventCallback<T, K>): void {
        if (!this.handlers[event]) {
            this.handlers[event] = [];
        }
        this.handlers[event]!.push(callback);
    }

    off<K extends EventKey<T>>(event: K, callback: EventCallback<T, K>): void {
        const handlers = this.handlers[event];
        if (handlers) {
            const index = handlers.indexOf(callback);
            if (index !== -1) {
                handlers.splice(index, 1);
            }
        }
    }

    emit<K extends EventKey<T>>(event: K, payload: EventPayload<T, K>): void {
        const handlers = this.handlers[event];
        if (handlers) {
            handlers.forEach(handler => handler(payload));
        }
    }

    once<K extends EventKey<T>>(event: K, callback: EventCallback<T, K>): void {
        const onceCallback = (payload: EventPayload<T, K>) => {
            callback(payload);
            this.off(event, onceCallback as EventCallback<T, K>);
        };
        this.on(event, onceCallback as EventCallback<T, K>);
    }
}

// Usage
interface AppEvents {
    login: { userId: string; timestamp: Date };
    logout: { userId: string };
    error: { code: number; message: string };
}

const emitter = new EventEmitter<AppEvents>();

emitter.on("login", (payload) => {
    console.log(`User ${payload.userId} logged in at ${payload.timestamp}`);
});

emitter.emit("login", { userId: "123", timestamp: new Date() });
// emitter.emit("login", { wrong: "payload" });  // Error!
```

### Exercise 3: Builder Pattern Types

```typescript
// Create type-safe builder pattern
// Builder should track which properties have been set

// Your code here
```

**Solution:**
```typescript
type BuilderState<T, Set extends keyof T> = {
    [K in keyof T]: K extends Set ? T[K] : T[K] | undefined;
};

interface Builder<T, Required extends keyof T, Set extends keyof T = never> {
    set<K extends keyof T>(
        key: K,
        value: T[K]
    ): Builder<T, Required, Set | K>;

    build: Set extends Required ? () => T : never;
}

// Factory function
function createBuilder<T, Required extends keyof T = keyof T>(): Builder<T, Required> {
    const state: Partial<T> = {};

    return {
        set<K extends keyof T>(key: K, value: T[K]) {
            state[key] = value;
            return this as Builder<T, Required, any>;
        },
        build() {
            return state as T;
        }
    } as Builder<T, Required>;
}

// Usage
interface UserConfig {
    name: string;
    email: string;
    age?: number;
    role?: string;
}

type RequiredUserFields = "name" | "email";

const builder = createBuilder<UserConfig, RequiredUserFields>();

// Must set required fields before build is available
// const user = builder.build();  // Error: build is never

const user = builder
    .set("name", "John")
    .set("email", "john@example.com")
    .set("age", 30)
    .build();  // OK: all required fields set

console.log(user);
// { name: "John", email: "john@example.com", age: 30 }
```

---

## Key Takeaways

1. **Conditional types** - `T extends U ? X : Y` for type-level conditionals
2. **Infer keyword** - Extract types from other types
3. **Mapped types** - Transform types by iterating properties
4. **Template literal types** - Type-safe string manipulation
5. **Utility types** - Built-in types for common transformations
6. **Type predicates** - Custom type guards with `is`
7. **Assertion functions** - Runtime assertions with type narrowing
8. **Discriminated unions** - Safe pattern matching with exhaustive checks

---

## Self-Check Questions

1. What's the difference between `Extract` and `Exclude`?
2. How does `infer` work in conditional types?
3. When would you use template literal types?
4. What's the difference between `type predicate` and `assertion function`?
5. How do you ensure exhaustive handling of discriminated unions?
6. What are recursive types and when are they useful?

---

**Next Lesson:** [Day 8-9 - Practical TypeScript](./day-08-09-practical-typescript.md)
