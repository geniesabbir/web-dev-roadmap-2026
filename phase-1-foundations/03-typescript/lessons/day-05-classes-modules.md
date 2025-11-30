# Day 5: Classes & Modules in TypeScript

## Classes in TypeScript

TypeScript adds powerful type features to JavaScript classes, including access modifiers, abstract classes, and better type inference.

### Basic Class Syntax

```typescript
class Person {
    // Property declarations with types
    name: string;
    age: number;

    // Constructor
    constructor(name: string, age: number) {
        this.name = name;
        this.age = age;
    }

    // Method
    greet(): string {
        return `Hello, I'm ${this.name}`;
    }

    // Method with parameters
    celebrateBirthday(): void {
        this.age++;
        console.log(`Happy birthday! Now ${this.age} years old.`);
    }
}

const person = new Person("John", 30);
console.log(person.greet());      // "Hello, I'm John"
person.celebrateBirthday();       // "Happy birthday! Now 31 years old."
```

### Parameter Properties (Shorthand)

```typescript
// Long form
class UserLong {
    name: string;
    email: string;

    constructor(name: string, email: string) {
        this.name = name;
        this.email = email;
    }
}

// Shorthand with parameter properties
class User {
    constructor(
        public name: string,
        public email: string,
        private password: string,
        readonly id: number = Date.now()
    ) {}
    // Properties are automatically declared and assigned
}

const user = new User("John", "john@example.com", "secret123");
console.log(user.name);    // "John"
console.log(user.email);   // "john@example.com"
// console.log(user.password);  // Error: private
console.log(user.id);      // 1234567890
// user.id = 999;  // Error: readonly
```

### Access Modifiers

```typescript
class BankAccount {
    // Public - accessible everywhere (default)
    public accountHolder: string;

    // Private - only accessible within the class
    private balance: number;

    // Protected - accessible within class and subclasses
    protected accountNumber: string;

    // Readonly - can only be set in constructor
    readonly createdAt: Date;

    constructor(holder: string, initialBalance: number) {
        this.accountHolder = holder;
        this.balance = initialBalance;
        this.accountNumber = this.generateAccountNumber();
        this.createdAt = new Date();
    }

    private generateAccountNumber(): string {
        return Math.random().toString(36).substring(2, 10).toUpperCase();
    }

    public deposit(amount: number): void {
        if (amount > 0) {
            this.balance += amount;
        }
    }

    public withdraw(amount: number): boolean {
        if (amount > 0 && amount <= this.balance) {
            this.balance -= amount;
            return true;
        }
        return false;
    }

    public getBalance(): number {
        return this.balance;
    }
}

const account = new BankAccount("John Doe", 1000);
account.deposit(500);
console.log(account.getBalance());  // 1500
console.log(account.accountHolder); // "John Doe"
// console.log(account.balance);    // Error: private
// console.log(account.accountNumber); // Error: protected
```

### Static Members

```typescript
class MathUtils {
    // Static property
    static readonly PI: number = 3.14159;
    static readonly E: number = 2.71828;

    // Static method
    static square(n: number): number {
        return n * n;
    }

    static cube(n: number): number {
        return n * n * n;
    }

    static circleArea(radius: number): number {
        return MathUtils.PI * MathUtils.square(radius);
    }

    // Private static for internal use
    private static validateNumber(n: number): boolean {
        return !isNaN(n) && isFinite(n);
    }

    static safeSquare(n: number): number | null {
        if (!MathUtils.validateNumber(n)) {
            return null;
        }
        return MathUtils.square(n);
    }
}

// Access without instantiation
console.log(MathUtils.PI);           // 3.14159
console.log(MathUtils.square(5));    // 25
console.log(MathUtils.circleArea(3)); // 28.27431

// Counter example with static
class IdGenerator {
    private static nextId: number = 1;

    static generate(): number {
        return IdGenerator.nextId++;
    }

    static reset(): void {
        IdGenerator.nextId = 1;
    }
}

console.log(IdGenerator.generate());  // 1
console.log(IdGenerator.generate());  // 2
console.log(IdGenerator.generate());  // 3
IdGenerator.reset();
console.log(IdGenerator.generate());  // 1
```

### Getters and Setters

```typescript
class Temperature {
    private _celsius: number;

    constructor(celsius: number) {
        this._celsius = celsius;
    }

    // Getter
    get celsius(): number {
        return this._celsius;
    }

    // Setter with validation
    set celsius(value: number) {
        if (value < -273.15) {
            throw new Error("Temperature below absolute zero!");
        }
        this._celsius = value;
    }

    // Computed property with getter only (readonly)
    get fahrenheit(): number {
        return (this._celsius * 9/5) + 32;
    }

    get kelvin(): number {
        return this._celsius + 273.15;
    }
}

const temp = new Temperature(25);
console.log(temp.celsius);     // 25
console.log(temp.fahrenheit);  // 77
console.log(temp.kelvin);      // 298.15

temp.celsius = 30;
console.log(temp.fahrenheit);  // 86

// temp.celsius = -300;  // Error: Temperature below absolute zero!

// Another example with validation
class User {
    private _email: string = "";

    get email(): string {
        return this._email;
    }

    set email(value: string) {
        if (!value.includes("@")) {
            throw new Error("Invalid email address");
        }
        this._email = value.toLowerCase();
    }
}
```

### Inheritance

```typescript
// Base class
class Animal {
    constructor(
        public name: string,
        protected age: number
    ) {}

    speak(): string {
        return `${this.name} makes a sound`;
    }

    getInfo(): string {
        return `${this.name} is ${this.age} years old`;
    }
}

// Derived class
class Dog extends Animal {
    constructor(
        name: string,
        age: number,
        public breed: string
    ) {
        super(name, age);  // Must call super() first
    }

    // Override method
    speak(): string {
        return `${this.name} barks: Woof!`;
    }

    // New method
    fetch(): string {
        return `${this.name} fetches the ball`;
    }

    // Access protected member
    celebrateBirthday(): void {
        this.age++;  // OK - protected is accessible
        console.log(`${this.name} is now ${this.age}`);
    }
}

class Cat extends Animal {
    constructor(
        name: string,
        age: number,
        public indoor: boolean
    ) {
        super(name, age);
    }

    speak(): string {
        return `${this.name} meows: Meow!`;
    }

    purr(): string {
        return `${this.name} purrs contentedly`;
    }
}

const dog = new Dog("Buddy", 3, "Golden Retriever");
const cat = new Cat("Whiskers", 5, true);

console.log(dog.speak());   // "Buddy barks: Woof!"
console.log(cat.speak());   // "Whiskers meows: Meow!"
console.log(dog.fetch());   // "Buddy fetches the ball"
console.log(cat.purr());    // "Whiskers purrs contentedly"

// Polymorphism
const animals: Animal[] = [dog, cat];
animals.forEach(animal => console.log(animal.speak()));
```

### Abstract Classes

```typescript
// Abstract class - cannot be instantiated
abstract class Shape {
    constructor(public color: string) {}

    // Abstract method - must be implemented by subclasses
    abstract getArea(): number;
    abstract getPerimeter(): number;

    // Regular method - shared by all subclasses
    describe(): string {
        return `A ${this.color} shape with area ${this.getArea().toFixed(2)}`;
    }
}

class Circle extends Shape {
    constructor(
        color: string,
        public radius: number
    ) {
        super(color);
    }

    getArea(): number {
        return Math.PI * this.radius ** 2;
    }

    getPerimeter(): number {
        return 2 * Math.PI * this.radius;
    }
}

class Rectangle extends Shape {
    constructor(
        color: string,
        public width: number,
        public height: number
    ) {
        super(color);
    }

    getArea(): number {
        return this.width * this.height;
    }

    getPerimeter(): number {
        return 2 * (this.width + this.height);
    }
}

class Triangle extends Shape {
    constructor(
        color: string,
        public base: number,
        public height: number,
        public side1: number,
        public side2: number
    ) {
        super(color);
    }

    getArea(): number {
        return (this.base * this.height) / 2;
    }

    getPerimeter(): number {
        return this.base + this.side1 + this.side2;
    }
}

// const shape = new Shape("red");  // Error: Cannot instantiate abstract class

const circle = new Circle("blue", 5);
const rectangle = new Rectangle("green", 4, 6);

console.log(circle.describe());     // "A blue shape with area 78.54"
console.log(rectangle.describe());  // "A green shape with area 24.00"

// Array of shapes
const shapes: Shape[] = [circle, rectangle, new Triangle("red", 3, 4, 5, 5)];
const totalArea = shapes.reduce((sum, shape) => sum + shape.getArea(), 0);
```

### Implementing Interfaces

```typescript
// Interface definition
interface Printable {
    print(): string;
}

interface Saveable {
    save(): Promise<boolean>;
    load(id: string): Promise<void>;
}

interface Identifiable {
    id: string;
}

// Class implementing multiple interfaces
class Document implements Printable, Saveable, Identifiable {
    id: string;
    content: string;

    constructor(id: string, content: string) {
        this.id = id;
        this.content = content;
    }

    print(): string {
        return `Document ${this.id}: ${this.content}`;
    }

    async save(): Promise<boolean> {
        console.log(`Saving document ${this.id}...`);
        // Simulate async save
        return true;
    }

    async load(id: string): Promise<void> {
        console.log(`Loading document ${id}...`);
        this.id = id;
        // Simulate async load
        this.content = "Loaded content";
    }
}

// Interface extending other interfaces
interface VersionedDocument extends Printable, Saveable, Identifiable {
    version: number;
    getHistory(): string[];
}

// Abstract class with interface
abstract class BaseEntity implements Identifiable {
    abstract id: string;
    abstract validate(): boolean;
}

class Product extends BaseEntity {
    id: string;
    name: string;
    price: number;

    constructor(id: string, name: string, price: number) {
        super();
        this.id = id;
        this.name = name;
        this.price = price;
    }

    validate(): boolean {
        return this.price > 0 && this.name.length > 0;
    }
}
```

### Generic Classes

```typescript
// Generic class
class Repository<T extends { id: string | number }> {
    private items: Map<string | number, T> = new Map();

    add(item: T): void {
        this.items.set(item.id, item);
    }

    get(id: string | number): T | undefined {
        return this.items.get(id);
    }

    getAll(): T[] {
        return Array.from(this.items.values());
    }

    update(id: string | number, updates: Partial<T>): T | undefined {
        const item = this.items.get(id);
        if (item) {
            const updated = { ...item, ...updates };
            this.items.set(id, updated);
            return updated;
        }
        return undefined;
    }

    delete(id: string | number): boolean {
        return this.items.delete(id);
    }

    find(predicate: (item: T) => boolean): T | undefined {
        for (const item of this.items.values()) {
            if (predicate(item)) {
                return item;
            }
        }
        return undefined;
    }

    filter(predicate: (item: T) => boolean): T[] {
        return this.getAll().filter(predicate);
    }
}

// Usage
interface User {
    id: number;
    name: string;
    email: string;
}

interface Product {
    id: string;
    name: string;
    price: number;
}

const userRepo = new Repository<User>();
userRepo.add({ id: 1, name: "John", email: "john@example.com" });
userRepo.add({ id: 2, name: "Jane", email: "jane@example.com" });

const productRepo = new Repository<Product>();
productRepo.add({ id: "p1", name: "Laptop", price: 999 });

console.log(userRepo.get(1));  // { id: 1, name: "John", ... }
console.log(productRepo.filter(p => p.price > 500));  // [{ id: "p1", ... }]
```

---

## TypeScript Modules

Modules help organize code into separate files with explicit imports and exports.

### Export Syntax

```typescript
// math.ts

// Named exports
export const PI = 3.14159;
export const E = 2.71828;

export function add(a: number, b: number): number {
    return a + b;
}

export function subtract(a: number, b: number): number {
    return a - b;
}

// Export interface
export interface MathResult {
    value: number;
    operation: string;
}

// Export type
export type MathOperation = (a: number, b: number) => number;

// Export class
export class Calculator {
    private history: MathResult[] = [];

    calculate(a: number, b: number, op: MathOperation, opName: string): number {
        const result = op(a, b);
        this.history.push({ value: result, operation: opName });
        return result;
    }

    getHistory(): MathResult[] {
        return [...this.history];
    }
}

// Export after declaration
const multiply = (a: number, b: number): number => a * b;
const divide = (a: number, b: number): number => a / b;

export { multiply, divide };

// Rename on export
const internalModulo = (a: number, b: number): number => a % b;
export { internalModulo as modulo };
```

### Default Exports

```typescript
// logger.ts

// Default export - one per module
export default class Logger {
    private prefix: string;

    constructor(prefix: string = "") {
        this.prefix = prefix;
    }

    log(message: string): void {
        console.log(`${this.prefix}[LOG] ${message}`);
    }

    error(message: string): void {
        console.error(`${this.prefix}[ERROR] ${message}`);
    }

    warn(message: string): void {
        console.warn(`${this.prefix}[WARN] ${message}`);
    }
}

// Can also have named exports alongside default
export type LogLevel = "log" | "error" | "warn";
export const DEFAULT_PREFIX = "[App] ";


// Alternative: export default at the end
// user.ts
class User {
    constructor(public name: string) {}
}

export default User;


// Function as default export
// utils.ts
export default function formatDate(date: Date): string {
    return date.toISOString();
}
```

### Import Syntax

```typescript
// main.ts

// Named imports
import { add, subtract, PI, Calculator } from "./math";
import { MathResult, MathOperation } from "./math";

console.log(add(2, 3));  // 5
console.log(PI);         // 3.14159

const calc = new Calculator();
calc.calculate(10, 5, add, "add");

// Rename on import
import { multiply as mul, divide as div } from "./math";
console.log(mul(4, 5));  // 20

// Import all as namespace
import * as MathUtils from "./math";
console.log(MathUtils.PI);           // 3.14159
console.log(MathUtils.add(2, 3));    // 5
console.log(MathUtils.subtract(5, 2)); // 3

// Default import (can use any name)
import Logger from "./logger";
import MyLogger from "./logger";  // Same thing, different name

const logger = new Logger("[Main] ");
logger.log("Application started");

// Combining default and named imports
import Logger, { LogLevel, DEFAULT_PREFIX } from "./logger";

// Import for side effects only
import "./polyfills";  // Just runs the module

// Type-only imports (removed at runtime)
import type { MathResult } from "./math";
import { type MathOperation } from "./math";

// This ensures the import is only used for types
const result: MathResult = { value: 10, operation: "add" };
```

### Re-exporting

```typescript
// index.ts (barrel file)

// Re-export everything from a module
export * from "./math";
export * from "./logger";

// Re-export specific items
export { add, subtract } from "./math";
export { Logger } from "./logger";

// Re-export with rename
export { Calculator as MathCalculator } from "./math";

// Re-export default as named
export { default as Logger } from "./logger";

// Re-export and rename
export { multiply as mul } from "./math";

// Aggregate multiple modules
export * as MathUtils from "./math";
export * as StringUtils from "./strings";


// Usage from another file
import { add, Logger, MathCalculator, MathUtils } from "./index";
// or
import { add, Logger } from "./";  // index.ts is default
```

### Module Organization Patterns

```typescript
// Project structure
/*
src/
├── index.ts           # Main entry, re-exports public API
├── types/
│   ├── index.ts       # Re-exports all types
│   ├── user.ts
│   └── product.ts
├── services/
│   ├── index.ts       # Re-exports all services
│   ├── user.service.ts
│   └── product.service.ts
├── utils/
│   ├── index.ts
│   ├── string.utils.ts
│   └── date.utils.ts
└── models/
    ├── index.ts
    ├── user.model.ts
    └── product.model.ts
*/

// types/user.ts
export interface User {
    id: number;
    name: string;
    email: string;
}

export interface CreateUserDTO {
    name: string;
    email: string;
    password: string;
}

export interface UpdateUserDTO {
    name?: string;
    email?: string;
}

// types/index.ts
export * from "./user";
export * from "./product";

// services/user.service.ts
import { User, CreateUserDTO, UpdateUserDTO } from "../types";

export class UserService {
    private users: User[] = [];

    create(dto: CreateUserDTO): User {
        const user: User = {
            id: Date.now(),
            name: dto.name,
            email: dto.email
        };
        this.users.push(user);
        return user;
    }

    findById(id: number): User | undefined {
        return this.users.find(u => u.id === id);
    }

    update(id: number, dto: UpdateUserDTO): User | undefined {
        const user = this.findById(id);
        if (user) {
            Object.assign(user, dto);
            return user;
        }
        return undefined;
    }
}

// services/index.ts
export { UserService } from "./user.service";
export { ProductService } from "./product.service";

// src/index.ts (main entry)
export * from "./types";
export * from "./services";
export * from "./models";
export * from "./utils";
```

### Dynamic Imports

```typescript
// Static import (bundled together)
import { heavyFunction } from "./heavy-module";

// Dynamic import (code splitting, lazy loading)
async function loadHeavyModule() {
    const module = await import("./heavy-module");
    module.heavyFunction();
}

// Conditional loading
async function loadFeature(featureName: string) {
    switch (featureName) {
        case "analytics":
            const analytics = await import("./features/analytics");
            return analytics.init();

        case "charts":
            const charts = await import("./features/charts");
            return charts.render();

        default:
            throw new Error(`Unknown feature: ${featureName}`);
    }
}

// Type-safe dynamic import
interface ChartModule {
    render: (data: number[]) => void;
    destroy: () => void;
}

async function loadCharts(): Promise<ChartModule> {
    return import("./features/charts") as Promise<ChartModule>;
}

// Usage
const charts = await loadCharts();
charts.render([1, 2, 3, 4, 5]);

// Lazy loading in React (example pattern)
const LazyComponent = async () => {
    const { default: Component } = await import("./HeavyComponent");
    return Component;
};
```

### Module Resolution

```typescript
// tsconfig.json module settings
{
    "compilerOptions": {
        // Module system
        "module": "ESNext",        // or "CommonJS", "ES2020"

        // How to resolve imports
        "moduleResolution": "node",  // or "bundler" for modern bundlers

        // Base URL for non-relative imports
        "baseUrl": "./src",

        // Path aliases
        "paths": {
            "@/*": ["*"],
            "@components/*": ["components/*"],
            "@utils/*": ["utils/*"],
            "@types/*": ["types/*"]
        },

        // Allow importing JSON files
        "resolveJsonModule": true,

        // Allow default imports from CommonJS modules
        "esModuleInterop": true,
        "allowSyntheticDefaultImports": true
    }
}

// With path aliases configured:
import { Button } from "@components/Button";
import { formatDate } from "@utils/date";
import { User } from "@types/user";
import config from "@/config.json";

// Instead of:
import { Button } from "../../../components/Button";
import { formatDate } from "../../utils/date";
```

---

## Namespaces (Legacy)

Namespaces are TypeScript's original module system. Modern code should use ES modules instead.

```typescript
// Namespace declaration
namespace Validation {
    // Export to make available outside namespace
    export interface StringValidator {
        isValid(s: string): boolean;
    }

    // Private to namespace (no export)
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;

    export class EmailValidator implements StringValidator {
        isValid(s: string): boolean {
            return emailRegex.test(s);
        }
    }

    export class ZipCodeValidator implements StringValidator {
        isValid(s: string): boolean {
            return /^\d{5}(-\d{4})?$/.test(s);
        }
    }
}

// Usage
const emailValidator = new Validation.EmailValidator();
console.log(emailValidator.isValid("test@example.com"));  // true

// Nested namespaces
namespace App {
    export namespace Models {
        export interface User {
            id: number;
            name: string;
        }
    }

    export namespace Services {
        export class UserService {
            getUser(id: number): Models.User {
                return { id, name: "John" };
            }
        }
    }
}

const userService = new App.Services.UserService();
const user: App.Models.User = userService.getUser(1);
```

### When to Use Namespaces vs Modules

```typescript
// USE MODULES (ES imports/exports) for:
// - Modern applications
// - Node.js projects
// - Bundled applications (webpack, vite, etc.)
// - New code

// USE NAMESPACES for:
// - Legacy browser code without bundler
// - Declaration files (.d.ts) for global libraries
// - Extending existing declarations (declaration merging)

// Declaration merging example
// Extending Express Request in @types
declare namespace Express {
    interface Request {
        user?: {
            id: number;
            name: string;
        };
    }
}
```

---

## Declaration Files

Declaration files (.d.ts) describe the types of JavaScript code.

### Creating Declaration Files

```typescript
// math.d.ts - Declaration file for a JavaScript library

// Function declarations
declare function add(a: number, b: number): number;
declare function subtract(a: number, b: number): number;

// Variable declaration
declare const PI: number;

// Class declaration
declare class Calculator {
    constructor();
    add(a: number, b: number): number;
    subtract(a: number, b: number): number;
    multiply(a: number, b: number): number;
    divide(a: number, b: number): number;
}

// Interface (already declaration-only)
interface MathResult {
    value: number;
    operation: string;
}

// Type alias (already declaration-only)
type MathOperation = (a: number, b: number) => number;

// Module declaration
declare module "legacy-math-lib" {
    export function calculate(expr: string): number;
    export const version: string;
}
```

### Ambient Declarations

```typescript
// globals.d.ts - Declare global types

// Global variable
declare const API_URL: string;
declare const DEBUG: boolean;

// Global function
declare function log(message: string): void;

// Extend Window interface
interface Window {
    myApp: {
        version: string;
        init(): void;
    };
}

// Extend global namespace
declare global {
    interface Array<T> {
        customMethod(): T[];
    }

    function myGlobalFunction(): void;
}

// Must export something to make this a module
export {};

// Usage anywhere in the project
console.log(API_URL);
window.myApp.init();
[1, 2, 3].customMethod();
```

### Module Augmentation

```typescript
// Augment existing module
import { Request } from "express";

declare module "express" {
    interface Request {
        user?: {
            id: string;
            email: string;
            roles: string[];
        };
        sessionId?: string;
    }
}

// Now in your code:
import { Request, Response, NextFunction } from "express";

function authMiddleware(req: Request, res: Response, next: NextFunction) {
    req.user = { id: "123", email: "user@example.com", roles: ["user"] };
    req.sessionId = "session-abc";
    next();
}

function protectedRoute(req: Request, res: Response) {
    // TypeScript knows about req.user and req.sessionId
    if (req.user) {
        res.json({ userId: req.user.id });
    }
}
```

---

## Practice Exercises

### Exercise 1: Create a Task Manager Class

```typescript
// Create a TaskManager class with:
// - tasks array (private)
// - addTask(title: string, priority: Priority): Task
// - completeTask(id: number): boolean
// - getTasks(filter?: TaskFilter): Task[]
// - getStats(): TaskStats

// Your code here
```

**Solution:**
```typescript
type Priority = "low" | "medium" | "high";
type Status = "pending" | "in-progress" | "completed";

interface Task {
    id: number;
    title: string;
    priority: Priority;
    status: Status;
    createdAt: Date;
    completedAt?: Date;
}

interface TaskFilter {
    priority?: Priority;
    status?: Status;
}

interface TaskStats {
    total: number;
    pending: number;
    inProgress: number;
    completed: number;
    byPriority: Record<Priority, number>;
}

class TaskManager {
    private tasks: Task[] = [];
    private nextId: number = 1;

    addTask(title: string, priority: Priority = "medium"): Task {
        const task: Task = {
            id: this.nextId++,
            title,
            priority,
            status: "pending",
            createdAt: new Date()
        };
        this.tasks.push(task);
        return task;
    }

    updateStatus(id: number, status: Status): boolean {
        const task = this.tasks.find(t => t.id === id);
        if (!task) return false;

        task.status = status;
        if (status === "completed") {
            task.completedAt = new Date();
        }
        return true;
    }

    completeTask(id: number): boolean {
        return this.updateStatus(id, "completed");
    }

    getTasks(filter?: TaskFilter): Task[] {
        let result = [...this.tasks];

        if (filter?.priority) {
            result = result.filter(t => t.priority === filter.priority);
        }
        if (filter?.status) {
            result = result.filter(t => t.status === filter.status);
        }

        return result;
    }

    getStats(): TaskStats {
        const stats: TaskStats = {
            total: this.tasks.length,
            pending: 0,
            inProgress: 0,
            completed: 0,
            byPriority: { low: 0, medium: 0, high: 0 }
        };

        for (const task of this.tasks) {
            // Count by status
            switch (task.status) {
                case "pending":
                    stats.pending++;
                    break;
                case "in-progress":
                    stats.inProgress++;
                    break;
                case "completed":
                    stats.completed++;
                    break;
            }

            // Count by priority
            stats.byPriority[task.priority]++;
        }

        return stats;
    }

    deleteTask(id: number): boolean {
        const index = this.tasks.findIndex(t => t.id === id);
        if (index === -1) return false;

        this.tasks.splice(index, 1);
        return true;
    }
}

// Usage
const manager = new TaskManager();

manager.addTask("Learn TypeScript", "high");
manager.addTask("Write documentation", "medium");
manager.addTask("Review code", "low");

manager.completeTask(1);

console.log(manager.getTasks({ status: "pending" }));
console.log(manager.getStats());
```

### Exercise 2: Module Structure

```typescript
// Create a modular user management system:
// - types/user.ts - User interfaces
// - services/user.service.ts - UserService class
// - utils/validation.ts - Validation functions
// - index.ts - Re-export public API

// Your code here
```

**Solution:**
```typescript
// types/user.ts
export interface User {
    id: string;
    email: string;
    name: string;
    createdAt: Date;
    updatedAt: Date;
}

export interface CreateUserInput {
    email: string;
    name: string;
    password: string;
}

export interface UpdateUserInput {
    email?: string;
    name?: string;
}

export type UserWithoutDates = Omit<User, "createdAt" | "updatedAt">;


// utils/validation.ts
export function isValidEmail(email: string): boolean {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return emailRegex.test(email);
}

export function isValidName(name: string): boolean {
    return name.length >= 2 && name.length <= 100;
}

export function isValidPassword(password: string): boolean {
    // At least 8 chars, one uppercase, one lowercase, one number
    const passwordRegex = /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d).{8,}$/;
    return passwordRegex.test(password);
}

export interface ValidationResult {
    valid: boolean;
    errors: string[];
}

export function validateCreateUser(input: {
    email: string;
    name: string;
    password: string;
}): ValidationResult {
    const errors: string[] = [];

    if (!isValidEmail(input.email)) {
        errors.push("Invalid email format");
    }
    if (!isValidName(input.name)) {
        errors.push("Name must be 2-100 characters");
    }
    if (!isValidPassword(input.password)) {
        errors.push("Password must be 8+ chars with uppercase, lowercase, and number");
    }

    return { valid: errors.length === 0, errors };
}


// services/user.service.ts
import { User, CreateUserInput, UpdateUserInput } from "../types/user";
import { validateCreateUser, isValidEmail } from "../utils/validation";

export class UserService {
    private users: Map<string, User> = new Map();

    create(input: CreateUserInput): User {
        const validation = validateCreateUser(input);
        if (!validation.valid) {
            throw new Error(validation.errors.join(", "));
        }

        // Check for duplicate email
        if (this.findByEmail(input.email)) {
            throw new Error("Email already in use");
        }

        const user: User = {
            id: crypto.randomUUID(),
            email: input.email,
            name: input.name,
            createdAt: new Date(),
            updatedAt: new Date()
        };

        this.users.set(user.id, user);
        return user;
    }

    findById(id: string): User | undefined {
        return this.users.get(id);
    }

    findByEmail(email: string): User | undefined {
        for (const user of this.users.values()) {
            if (user.email === email) {
                return user;
            }
        }
        return undefined;
    }

    update(id: string, input: UpdateUserInput): User {
        const user = this.users.get(id);
        if (!user) {
            throw new Error("User not found");
        }

        if (input.email) {
            if (!isValidEmail(input.email)) {
                throw new Error("Invalid email format");
            }
            const existing = this.findByEmail(input.email);
            if (existing && existing.id !== id) {
                throw new Error("Email already in use");
            }
            user.email = input.email;
        }

        if (input.name !== undefined) {
            user.name = input.name;
        }

        user.updatedAt = new Date();
        return user;
    }

    delete(id: string): boolean {
        return this.users.delete(id);
    }

    list(): User[] {
        return Array.from(this.users.values());
    }
}


// index.ts (barrel file)
export * from "./types/user";
export { UserService } from "./services/user.service";
export {
    isValidEmail,
    isValidName,
    isValidPassword,
    validateCreateUser,
    type ValidationResult
} from "./utils/validation";
```

### Exercise 3: Abstract Repository Pattern

```typescript
// Create an abstract Repository<T> class that:
// - Has abstract methods: create, findById, update, delete
// - Implements common methods: findAll, count, exists
// - Create InMemoryRepository<T> extending it

// Your code here
```

**Solution:**
```typescript
interface Entity {
    id: string;
}

abstract class Repository<T extends Entity> {
    // Abstract methods - must be implemented
    abstract create(data: Omit<T, "id">): Promise<T>;
    abstract findById(id: string): Promise<T | null>;
    abstract update(id: string, data: Partial<T>): Promise<T | null>;
    abstract delete(id: string): Promise<boolean>;
    abstract findAll(): Promise<T[]>;

    // Implemented methods
    async exists(id: string): Promise<boolean> {
        const entity = await this.findById(id);
        return entity !== null;
    }

    async count(): Promise<number> {
        const all = await this.findAll();
        return all.length;
    }

    async findByIds(ids: string[]): Promise<T[]> {
        const results: T[] = [];
        for (const id of ids) {
            const entity = await this.findById(id);
            if (entity) {
                results.push(entity);
            }
        }
        return results;
    }
}

class InMemoryRepository<T extends Entity> extends Repository<T> {
    protected items: Map<string, T> = new Map();

    async create(data: Omit<T, "id">): Promise<T> {
        const id = crypto.randomUUID();
        const entity = { ...data, id } as T;
        this.items.set(id, entity);
        return entity;
    }

    async findById(id: string): Promise<T | null> {
        return this.items.get(id) ?? null;
    }

    async findAll(): Promise<T[]> {
        return Array.from(this.items.values());
    }

    async update(id: string, data: Partial<T>): Promise<T | null> {
        const existing = this.items.get(id);
        if (!existing) return null;

        const updated = { ...existing, ...data, id } as T;
        this.items.set(id, updated);
        return updated;
    }

    async delete(id: string): Promise<boolean> {
        return this.items.delete(id);
    }

    // Additional helper for in-memory
    async find(predicate: (item: T) => boolean): Promise<T[]> {
        return Array.from(this.items.values()).filter(predicate);
    }
}

// Usage example
interface User extends Entity {
    name: string;
    email: string;
}

class UserRepository extends InMemoryRepository<User> {
    async findByEmail(email: string): Promise<User | null> {
        const users = await this.find(u => u.email === email);
        return users[0] ?? null;
    }
}

// Test
async function test() {
    const repo = new UserRepository();

    const user = await repo.create({ name: "John", email: "john@example.com" });
    console.log("Created:", user);

    const found = await repo.findById(user.id);
    console.log("Found:", found);

    const updated = await repo.update(user.id, { name: "John Doe" });
    console.log("Updated:", updated);

    const count = await repo.count();
    console.log("Count:", count);

    const byEmail = await repo.findByEmail("john@example.com");
    console.log("By email:", byEmail);
}

test();
```

---

## Key Takeaways

1. **Access modifiers** - `public`, `private`, `protected`, `readonly`
2. **Parameter properties** - Shorthand for declaring and assigning in constructor
3. **Static members** - Belong to class itself, not instances
4. **Abstract classes** - Cannot be instantiated, provide base for inheritance
5. **ES Modules** - Use `import`/`export` for modern code organization
6. **Barrel files** - Re-export for cleaner imports
7. **Declaration files** - Describe types for JavaScript code
8. **Module augmentation** - Extend existing type definitions

---

## Self-Check Questions

1. What's the difference between `private` and `protected`?
2. When would you use static members?
3. What's the difference between abstract classes and interfaces?
4. When should you use default exports vs named exports?
5. What is a barrel file and why use it?
6. How do you augment an existing module's types?

---

**Next Lesson:** [Day 6-7 - Advanced Types](./day-06-07-advanced-types.md)
