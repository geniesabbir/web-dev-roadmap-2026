# Day 8-9: Practical TypeScript

## Working with Third-Party Libraries

### Installing Type Definitions

```bash
# Many packages include their own types
npm install axios  # Types included

# For packages without types, install from DefinitelyTyped
npm install lodash
npm install --save-dev @types/lodash

# Check if types exist
npm search @types/package-name

# Common type packages
npm install --save-dev @types/node     # Node.js
npm install --save-dev @types/express  # Express
npm install --save-dev @types/react    # React
npm install --save-dev @types/jest     # Jest
```

### Using Libraries with Types

```typescript
// Axios - types included
import axios, { AxiosResponse, AxiosError } from "axios";

interface User {
    id: number;
    name: string;
    email: string;
}

async function fetchUser(id: number): Promise<User> {
    const response: AxiosResponse<User> = await axios.get(`/api/users/${id}`);
    return response.data;
}

// With error handling
async function fetchUserSafe(id: number): Promise<User | null> {
    try {
        const response = await axios.get<User>(`/api/users/${id}`);
        return response.data;
    } catch (error) {
        if (axios.isAxiosError(error)) {
            console.error("API Error:", error.response?.status);
        }
        return null;
    }
}


// Lodash - needs @types/lodash
import _ from "lodash";
// or import specific functions
import { debounce, throttle, groupBy } from "lodash";

const users: User[] = [
    { id: 1, name: "John", email: "john@example.com" },
    { id: 2, name: "Jane", email: "jane@example.com" }
];

const grouped = groupBy(users, "name");  // Type: Dictionary<User[]>
const first = _.first(users);  // Type: User | undefined

// Date-fns - types included
import { format, addDays, isAfter } from "date-fns";

const today = new Date();
const formatted = format(today, "yyyy-MM-dd");  // Type: string
const nextWeek = addDays(today, 7);  // Type: Date
const isFuture = isAfter(nextWeek, today);  // Type: boolean
```

### Creating Type Definitions for Untyped Libraries

```typescript
// For a library without types: legacy-lib

// Option 1: Quick declaration (any types)
// src/types/legacy-lib.d.ts
declare module "legacy-lib" {
    export function doSomething(value: any): any;
    export const version: string;
}

// Option 2: Proper type definitions
// src/types/legacy-lib.d.ts
declare module "legacy-lib" {
    export interface Config {
        apiKey: string;
        timeout?: number;
        debug?: boolean;
    }

    export interface Result {
        success: boolean;
        data: unknown;
        error?: string;
    }

    export function initialize(config: Config): void;
    export function fetchData(endpoint: string): Promise<Result>;
    export function cleanup(): void;

    export const VERSION: string;
}

// Usage
import { initialize, fetchData, Config } from "legacy-lib";

const config: Config = {
    apiKey: "secret",
    timeout: 5000
};

initialize(config);
```

### Extending Existing Type Definitions

```typescript
// Augmenting Express Request
// src/types/express.d.ts
import { User } from "../models/User";

declare global {
    namespace Express {
        interface Request {
            user?: User;
            sessionId?: string;
            startTime?: number;
        }
    }
}

export {};  // Make this a module

// Now in your Express app
import express, { Request, Response } from "express";

app.use((req: Request, res: Response, next) => {
    req.startTime = Date.now();
    next();
});

app.get("/profile", (req: Request, res: Response) => {
    if (req.user) {
        res.json(req.user);  // TypeScript knows about user
    }
});


// Augmenting Window object
// src/types/global.d.ts
interface Window {
    analytics: {
        track(event: string, data?: object): void;
        identify(userId: string): void;
    };
    config: {
        apiUrl: string;
        environment: "development" | "staging" | "production";
    };
}

// Usage
window.analytics.track("page_view", { page: "/home" });
console.log(window.config.environment);
```

---

## Error Handling Patterns

### Typed Errors

```typescript
// Custom error classes
class AppError extends Error {
    constructor(
        message: string,
        public code: string,
        public statusCode: number = 500
    ) {
        super(message);
        this.name = "AppError";
        Error.captureStackTrace(this, this.constructor);
    }
}

class ValidationError extends AppError {
    constructor(
        message: string,
        public fields: Record<string, string[]>
    ) {
        super(message, "VALIDATION_ERROR", 400);
        this.name = "ValidationError";
    }
}

class NotFoundError extends AppError {
    constructor(resource: string, id: string | number) {
        super(`${resource} with id ${id} not found`, "NOT_FOUND", 404);
        this.name = "NotFoundError";
    }
}

class AuthenticationError extends AppError {
    constructor(message: string = "Authentication required") {
        super(message, "UNAUTHORIZED", 401);
        this.name = "AuthenticationError";
    }
}

// Error type guards
function isAppError(error: unknown): error is AppError {
    return error instanceof AppError;
}

function isValidationError(error: unknown): error is ValidationError {
    return error instanceof ValidationError;
}

// Usage
async function getUser(id: number): Promise<User> {
    const user = await db.users.findById(id);
    if (!user) {
        throw new NotFoundError("User", id);
    }
    return user;
}

// Error handling middleware
function errorHandler(error: unknown): { status: number; body: object } {
    if (isValidationError(error)) {
        return {
            status: error.statusCode,
            body: {
                error: error.message,
                fields: error.fields
            }
        };
    }

    if (isAppError(error)) {
        return {
            status: error.statusCode,
            body: {
                error: error.message,
                code: error.code
            }
        };
    }

    // Unknown error
    console.error("Unexpected error:", error);
    return {
        status: 500,
        body: { error: "Internal server error" }
    };
}
```

### Result Type Pattern

```typescript
// Result type - inspired by Rust
type Result<T, E = Error> =
    | { ok: true; value: T }
    | { ok: false; error: E };

// Helper functions
function Ok<T>(value: T): Result<T, never> {
    return { ok: true, value };
}

function Err<E>(error: E): Result<never, E> {
    return { ok: false, error };
}

// Usage
interface UserData {
    name: string;
    email: string;
}

interface ValidationErrors {
    name?: string[];
    email?: string[];
}

function validateUser(data: unknown): Result<UserData, ValidationErrors> {
    const errors: ValidationErrors = {};

    if (!data || typeof data !== "object") {
        return Err({ name: ["Invalid input"] });
    }

    const input = data as Record<string, unknown>;

    if (typeof input.name !== "string" || input.name.length < 2) {
        errors.name = ["Name must be at least 2 characters"];
    }

    if (typeof input.email !== "string" || !input.email.includes("@")) {
        errors.email = ["Invalid email format"];
    }

    if (Object.keys(errors).length > 0) {
        return Err(errors);
    }

    return Ok({
        name: input.name as string,
        email: input.email as string
    });
}

// Using the result
const result = validateUser({ name: "Jo", email: "invalid" });

if (result.ok) {
    console.log("Valid user:", result.value);
} else {
    console.log("Validation errors:", result.error);
}

// Chaining results
function map<T, U, E>(
    result: Result<T, E>,
    fn: (value: T) => U
): Result<U, E> {
    if (result.ok) {
        return Ok(fn(result.value));
    }
    return result;
}

function flatMap<T, U, E>(
    result: Result<T, E>,
    fn: (value: T) => Result<U, E>
): Result<U, E> {
    if (result.ok) {
        return fn(result.value);
    }
    return result;
}

// Example chain
const processedResult = flatMap(
    validateUser({ name: "John", email: "john@example.com" }),
    (user) => {
        // Further processing
        return Ok({ ...user, id: Date.now() });
    }
);
```

### Try-Catch Type Narrowing

```typescript
// Wrapper for async operations
async function tryCatch<T>(
    promise: Promise<T>
): Promise<[null, T] | [Error, null]> {
    try {
        const result = await promise;
        return [null, result];
    } catch (error) {
        return [error instanceof Error ? error : new Error(String(error)), null];
    }
}

// Usage
async function main() {
    const [error, user] = await tryCatch(fetchUser(1));

    if (error) {
        console.error("Failed to fetch user:", error.message);
        return;
    }

    // user is guaranteed to be User here
    console.log("User:", user.name);
}

// Type-safe error handling
interface TypedError<T extends string> {
    type: T;
    message: string;
}

type NetworkError = TypedError<"NETWORK"> & { statusCode: number };
type ValidationErr = TypedError<"VALIDATION"> & { fields: string[] };
type TimeoutError = TypedError<"TIMEOUT"> & { duration: number };

type ApiError = NetworkError | ValidationErr | TimeoutError;

function handleApiError(error: ApiError): void {
    switch (error.type) {
        case "NETWORK":
            console.log(`Network error: ${error.statusCode}`);
            break;
        case "VALIDATION":
            console.log(`Validation failed: ${error.fields.join(", ")}`);
            break;
        case "TIMEOUT":
            console.log(`Request timed out after ${error.duration}ms`);
            break;
    }
}
```

---

## API Type Patterns

### REST API Types

```typescript
// Base types
interface ApiResponse<T> {
    data: T;
    meta?: {
        total?: number;
        page?: number;
        limit?: number;
    };
}

interface ApiError {
    error: {
        code: string;
        message: string;
        details?: unknown;
    };
}

type ApiResult<T> = ApiResponse<T> | ApiError;

// Helper
function isApiError(response: ApiResult<unknown>): response is ApiError {
    return "error" in response;
}

// Entity types
interface User {
    id: number;
    name: string;
    email: string;
    createdAt: string;
    updatedAt: string;
}

interface Post {
    id: number;
    title: string;
    content: string;
    authorId: number;
    author?: User;
    createdAt: string;
}

// Request/Response types
interface CreateUserRequest {
    name: string;
    email: string;
    password: string;
}

interface UpdateUserRequest {
    name?: string;
    email?: string;
}

interface PaginationParams {
    page?: number;
    limit?: number;
    sort?: string;
    order?: "asc" | "desc";
}

// API client
class ApiClient {
    private baseUrl: string;

    constructor(baseUrl: string) {
        this.baseUrl = baseUrl;
    }

    private async request<T>(
        method: string,
        path: string,
        body?: unknown
    ): Promise<T> {
        const response = await fetch(`${this.baseUrl}${path}`, {
            method,
            headers: {
                "Content-Type": "application/json"
            },
            body: body ? JSON.stringify(body) : undefined
        });

        if (!response.ok) {
            const error = await response.json();
            throw new Error(error.message || "API request failed");
        }

        return response.json();
    }

    // Users
    async getUsers(params?: PaginationParams): Promise<ApiResponse<User[]>> {
        const query = new URLSearchParams(params as Record<string, string>);
        return this.request("GET", `/users?${query}`);
    }

    async getUser(id: number): Promise<ApiResponse<User>> {
        return this.request("GET", `/users/${id}`);
    }

    async createUser(data: CreateUserRequest): Promise<ApiResponse<User>> {
        return this.request("POST", "/users", data);
    }

    async updateUser(
        id: number,
        data: UpdateUserRequest
    ): Promise<ApiResponse<User>> {
        return this.request("PATCH", `/users/${id}`, data);
    }

    async deleteUser(id: number): Promise<void> {
        await this.request("DELETE", `/users/${id}`);
    }

    // Posts
    async getPosts(
        params?: PaginationParams & { authorId?: number }
    ): Promise<ApiResponse<Post[]>> {
        const query = new URLSearchParams(params as Record<string, string>);
        return this.request("GET", `/posts?${query}`);
    }
}

// Usage
const api = new ApiClient("https://api.example.com");

async function main() {
    const { data: users } = await api.getUsers({ page: 1, limit: 10 });
    console.log(users);  // Type: User[]

    const { data: user } = await api.getUser(1);
    console.log(user.name);  // Type-safe access
}
```

### Type-Safe Fetch Wrapper

```typescript
// HTTP method types
type HttpMethod = "GET" | "POST" | "PUT" | "PATCH" | "DELETE";

// Request options
interface RequestOptions {
    headers?: Record<string, string>;
    params?: Record<string, string | number | boolean>;
    timeout?: number;
}

// Typed fetch wrapper
async function typedFetch<T>(
    url: string,
    method: HttpMethod = "GET",
    body?: unknown,
    options: RequestOptions = {}
): Promise<T> {
    const controller = new AbortController();
    const timeoutId = options.timeout
        ? setTimeout(() => controller.abort(), options.timeout)
        : undefined;

    try {
        // Build URL with query params
        const urlWithParams = new URL(url);
        if (options.params) {
            Object.entries(options.params).forEach(([key, value]) => {
                urlWithParams.searchParams.set(key, String(value));
            });
        }

        const response = await fetch(urlWithParams.toString(), {
            method,
            headers: {
                "Content-Type": "application/json",
                ...options.headers
            },
            body: body ? JSON.stringify(body) : undefined,
            signal: controller.signal
        });

        if (!response.ok) {
            throw new Error(`HTTP ${response.status}: ${response.statusText}`);
        }

        return await response.json() as T;
    } finally {
        if (timeoutId) clearTimeout(timeoutId);
    }
}

// Typed API endpoints
interface Endpoints {
    "GET /users": {
        params: { page?: number; limit?: number };
        response: User[];
    };
    "GET /users/:id": {
        params: { id: number };
        response: User;
    };
    "POST /users": {
        body: CreateUserRequest;
        response: User;
    };
    "PUT /users/:id": {
        params: { id: number };
        body: UpdateUserRequest;
        response: User;
    };
}

// Advanced: Type-safe API client using endpoints
type ExtractParams<T extends string> = T extends `${string}:${infer Param}/${infer Rest}`
    ? Param | ExtractParams<`/${Rest}`>
    : T extends `${string}:${infer Param}`
        ? Param
        : never;

type ExtractMethod<T extends string> = T extends `${infer M} ${string}` ? M : never;
type ExtractPath<T extends string> = T extends `${string} ${infer P}` ? P : never;
```

---

## Configuration and Environment Types

### Environment Variables

```typescript
// env.d.ts - Declare environment variable types
declare namespace NodeJS {
    interface ProcessEnv {
        NODE_ENV: "development" | "production" | "test";
        PORT: string;
        DATABASE_URL: string;
        JWT_SECRET: string;
        API_KEY?: string;
        DEBUG?: string;
    }
}

// env.ts - Validated environment config
interface EnvConfig {
    nodeEnv: "development" | "production" | "test";
    port: number;
    databaseUrl: string;
    jwtSecret: string;
    apiKey: string | undefined;
    isDebug: boolean;
}

function loadEnv(): EnvConfig {
    const required = ["NODE_ENV", "PORT", "DATABASE_URL", "JWT_SECRET"];
    const missing = required.filter(key => !process.env[key]);

    if (missing.length > 0) {
        throw new Error(`Missing environment variables: ${missing.join(", ")}`);
    }

    return {
        nodeEnv: process.env.NODE_ENV as EnvConfig["nodeEnv"],
        port: parseInt(process.env.PORT, 10),
        databaseUrl: process.env.DATABASE_URL,
        jwtSecret: process.env.JWT_SECRET,
        apiKey: process.env.API_KEY,
        isDebug: process.env.DEBUG === "true"
    };
}

export const env = loadEnv();

// Usage
console.log(`Server starting on port ${env.port}`);
if (env.isDebug) {
    console.log("Debug mode enabled");
}
```

### Configuration Objects

```typescript
// config.ts
interface DatabaseConfig {
    host: string;
    port: number;
    name: string;
    user: string;
    password: string;
    ssl: boolean;
    poolSize: number;
}

interface ServerConfig {
    port: number;
    host: string;
    cors: {
        origin: string | string[];
        credentials: boolean;
    };
    rateLimit: {
        windowMs: number;
        max: number;
    };
}

interface LoggingConfig {
    level: "debug" | "info" | "warn" | "error";
    format: "json" | "pretty";
    file?: string;
}

interface AppConfig {
    database: DatabaseConfig;
    server: ServerConfig;
    logging: LoggingConfig;
}

// Default config with type safety
const defaultConfig: AppConfig = {
    database: {
        host: "localhost",
        port: 5432,
        name: "myapp",
        user: "postgres",
        password: "",
        ssl: false,
        poolSize: 10
    },
    server: {
        port: 3000,
        host: "0.0.0.0",
        cors: {
            origin: "*",
            credentials: true
        },
        rateLimit: {
            windowMs: 15 * 60 * 1000,  // 15 minutes
            max: 100
        }
    },
    logging: {
        level: "info",
        format: "json"
    }
};

// Deep merge for config overrides
type DeepPartial<T> = {
    [P in keyof T]?: T[P] extends object ? DeepPartial<T[P]> : T[P];
};

function deepMerge<T extends object>(target: T, source: DeepPartial<T>): T {
    const result = { ...target };

    for (const key in source) {
        const sourceValue = source[key];
        const targetValue = target[key];

        if (
            sourceValue !== undefined &&
            typeof sourceValue === "object" &&
            typeof targetValue === "object" &&
            !Array.isArray(sourceValue)
        ) {
            result[key] = deepMerge(targetValue, sourceValue as DeepPartial<typeof targetValue>);
        } else if (sourceValue !== undefined) {
            result[key] = sourceValue as T[typeof key];
        }
    }

    return result;
}

function loadConfig(overrides: DeepPartial<AppConfig> = {}): AppConfig {
    return deepMerge(defaultConfig, overrides);
}

// Usage
const config = loadConfig({
    server: {
        port: 8080
    },
    logging: {
        level: "debug",
        format: "pretty"
    }
});
```

---

## Testing Patterns

### Type-Safe Mocks

```typescript
// Creating type-safe mocks
interface UserService {
    getUser(id: number): Promise<User>;
    createUser(data: CreateUserRequest): Promise<User>;
    updateUser(id: number, data: UpdateUserRequest): Promise<User>;
    deleteUser(id: number): Promise<void>;
}

// Mock factory
type MockFunction<T extends (...args: any[]) => any> = jest.Mock<
    ReturnType<T>,
    Parameters<T>
>;

type MockedService<T> = {
    [K in keyof T]: T[K] extends (...args: any[]) => any
        ? MockFunction<T[K]>
        : T[K];
};

function createMockService<T extends object>(): MockedService<T> {
    return new Proxy({} as MockedService<T>, {
        get: (target, prop) => {
            if (!(prop in target)) {
                (target as any)[prop] = jest.fn();
            }
            return (target as any)[prop];
        }
    });
}

// Usage in tests
describe("UserController", () => {
    let userService: MockedService<UserService>;

    beforeEach(() => {
        userService = createMockService<UserService>();
    });

    it("should get user by id", async () => {
        const mockUser: User = {
            id: 1,
            name: "John",
            email: "john@example.com",
            createdAt: new Date().toISOString(),
            updatedAt: new Date().toISOString()
        };

        userService.getUser.mockResolvedValue(mockUser);

        const result = await userService.getUser(1);

        expect(result).toEqual(mockUser);
        expect(userService.getUser).toHaveBeenCalledWith(1);
    });
});
```

### Test Data Factories

```typescript
// Factory for creating test data
interface Factory<T> {
    build(overrides?: Partial<T>): T;
    buildMany(count: number, overrides?: Partial<T>): T[];
}

function createFactory<T>(defaults: () => T): Factory<T> {
    return {
        build(overrides?: Partial<T>): T {
            return { ...defaults(), ...overrides };
        },
        buildMany(count: number, overrides?: Partial<T>): T[] {
            return Array.from({ length: count }, () => this.build(overrides));
        }
    };
}

// User factory
let userIdCounter = 1;

const userFactory = createFactory<User>(() => ({
    id: userIdCounter++,
    name: `User ${userIdCounter}`,
    email: `user${userIdCounter}@example.com`,
    createdAt: new Date().toISOString(),
    updatedAt: new Date().toISOString()
}));

// Post factory with relations
const postFactory = createFactory<Post>(() => {
    const author = userFactory.build();
    return {
        id: Date.now(),
        title: "Test Post",
        content: "Test content",
        authorId: author.id,
        author,
        createdAt: new Date().toISOString()
    };
});

// Usage in tests
describe("PostService", () => {
    it("should create a post", () => {
        const user = userFactory.build({ name: "Author" });
        const post = postFactory.build({
            authorId: user.id,
            title: "My First Post"
        });

        expect(post.authorId).toBe(user.id);
        expect(post.title).toBe("My First Post");
    });

    it("should handle multiple posts", () => {
        const posts = postFactory.buildMany(5);
        expect(posts).toHaveLength(5);
    });
});
```

### Type Assertions in Tests

```typescript
// Custom type assertion helpers
function assertType<T>(value: unknown): asserts value is T {
    // Runtime assertion if needed
}

function assertDefined<T>(
    value: T | null | undefined,
    message?: string
): asserts value is T {
    if (value === null || value === undefined) {
        throw new Error(message ?? "Value should be defined");
    }
}

// Type guard matchers for Jest
declare global {
    namespace jest {
        interface Matchers<R> {
            toBeUser(): R;
            toHaveValidEmail(): R;
        }
    }
}

expect.extend({
    toBeUser(received: unknown) {
        const isUser = (
            typeof received === "object" &&
            received !== null &&
            "id" in received &&
            "name" in received &&
            "email" in received
        );

        return {
            pass: isUser,
            message: () => `expected ${received} to be a User`
        };
    },

    toHaveValidEmail(received: { email?: string }) {
        const hasValidEmail = (
            typeof received.email === "string" &&
            received.email.includes("@")
        );

        return {
            pass: hasValidEmail,
            message: () => `expected ${received.email} to be a valid email`
        };
    }
});

// Usage
test("user has valid structure", () => {
    const user = userFactory.build();
    expect(user).toBeUser();
    expect(user).toHaveValidEmail();
});
```

---

## Common Patterns and Best Practices

### Builder Pattern

```typescript
class QueryBuilder<T> {
    private query: {
        select: (keyof T)[];
        where: Partial<T>;
        orderBy?: keyof T;
        order: "asc" | "desc";
        limit?: number;
        offset?: number;
    };

    constructor() {
        this.query = {
            select: [],
            where: {},
            order: "asc"
        };
    }

    select<K extends keyof T>(...fields: K[]): this {
        this.query.select = fields;
        return this;
    }

    where(conditions: Partial<T>): this {
        this.query.where = { ...this.query.where, ...conditions };
        return this;
    }

    orderBy(field: keyof T, order: "asc" | "desc" = "asc"): this {
        this.query.orderBy = field;
        this.query.order = order;
        return this;
    }

    limit(count: number): this {
        this.query.limit = count;
        return this;
    }

    offset(count: number): this {
        this.query.offset = count;
        return this;
    }

    build(): typeof this.query {
        return { ...this.query };
    }

    // For SQL generation
    toSQL(): string {
        const { select, where, orderBy, order, limit, offset } = this.query;

        let sql = `SELECT ${select.length ? select.join(", ") : "*"} FROM table`;

        if (Object.keys(where).length > 0) {
            const conditions = Object.entries(where)
                .map(([key, value]) => `${key} = '${value}'`)
                .join(" AND ");
            sql += ` WHERE ${conditions}`;
        }

        if (orderBy) {
            sql += ` ORDER BY ${String(orderBy)} ${order.toUpperCase()}`;
        }

        if (limit !== undefined) {
            sql += ` LIMIT ${limit}`;
        }

        if (offset !== undefined) {
            sql += ` OFFSET ${offset}`;
        }

        return sql;
    }
}

// Usage
const query = new QueryBuilder<User>()
    .select("id", "name", "email")
    .where({ name: "John" })
    .orderBy("createdAt", "desc")
    .limit(10)
    .build();
```

### Repository Pattern

```typescript
interface Repository<T extends { id: string | number }> {
    findById(id: T["id"]): Promise<T | null>;
    findAll(options?: FindOptions<T>): Promise<T[]>;
    create(data: Omit<T, "id" | "createdAt" | "updatedAt">): Promise<T>;
    update(id: T["id"], data: Partial<T>): Promise<T | null>;
    delete(id: T["id"]): Promise<boolean>;
}

interface FindOptions<T> {
    where?: Partial<T>;
    orderBy?: keyof T;
    order?: "asc" | "desc";
    limit?: number;
    offset?: number;
}

// Implementation
class UserRepository implements Repository<User> {
    private db: Database;

    constructor(db: Database) {
        this.db = db;
    }

    async findById(id: number): Promise<User | null> {
        return this.db.query<User>("SELECT * FROM users WHERE id = ?", [id]);
    }

    async findAll(options: FindOptions<User> = {}): Promise<User[]> {
        let sql = "SELECT * FROM users";
        const params: unknown[] = [];

        if (options.where) {
            const conditions = Object.entries(options.where)
                .map(([key]) => {
                    params.push(options.where![key as keyof User]);
                    return `${key} = ?`;
                })
                .join(" AND ");
            sql += ` WHERE ${conditions}`;
        }

        if (options.orderBy) {
            sql += ` ORDER BY ${String(options.orderBy)} ${options.order || "ASC"}`;
        }

        if (options.limit) {
            sql += ` LIMIT ${options.limit}`;
        }

        if (options.offset) {
            sql += ` OFFSET ${options.offset}`;
        }

        return this.db.queryAll<User>(sql, params);
    }

    async create(data: Omit<User, "id" | "createdAt" | "updatedAt">): Promise<User> {
        const now = new Date().toISOString();
        const user = {
            ...data,
            createdAt: now,
            updatedAt: now
        };

        const id = await this.db.insert("users", user);
        return { ...user, id } as User;
    }

    async update(id: number, data: Partial<User>): Promise<User | null> {
        const updates = { ...data, updatedAt: new Date().toISOString() };
        await this.db.update("users", id, updates);
        return this.findById(id);
    }

    async delete(id: number): Promise<boolean> {
        const result = await this.db.delete("users", id);
        return result.affectedRows > 0;
    }
}
```

### Singleton Pattern

```typescript
class Database {
    private static instance: Database | null = null;
    private connection: DatabaseConnection | null = null;

    private constructor() {}

    static getInstance(): Database {
        if (!Database.instance) {
            Database.instance = new Database();
        }
        return Database.instance;
    }

    async connect(config: DatabaseConfig): Promise<void> {
        if (this.connection) {
            return;
        }
        this.connection = await createConnection(config);
    }

    async query<T>(sql: string, params?: unknown[]): Promise<T | null> {
        if (!this.connection) {
            throw new Error("Database not connected");
        }
        return this.connection.query<T>(sql, params);
    }

    async disconnect(): Promise<void> {
        if (this.connection) {
            await this.connection.close();
            this.connection = null;
        }
    }
}

// Usage
const db = Database.getInstance();
await db.connect(config);
const user = await db.query<User>("SELECT * FROM users WHERE id = ?", [1]);
```

### Dependency Injection

```typescript
// Service interfaces
interface ILogger {
    log(message: string): void;
    error(message: string, error?: Error): void;
}

interface IUserRepository {
    findById(id: number): Promise<User | null>;
    save(user: User): Promise<User>;
}

interface IEmailService {
    send(to: string, subject: string, body: string): Promise<void>;
}

// Implementations
class ConsoleLogger implements ILogger {
    log(message: string): void {
        console.log(`[LOG] ${message}`);
    }

    error(message: string, error?: Error): void {
        console.error(`[ERROR] ${message}`, error);
    }
}

// Service with dependencies
class UserService {
    constructor(
        private readonly logger: ILogger,
        private readonly userRepo: IUserRepository,
        private readonly emailService: IEmailService
    ) {}

    async registerUser(data: CreateUserRequest): Promise<User> {
        this.logger.log(`Registering user: ${data.email}`);

        const user = await this.userRepo.save({
            id: Date.now(),
            name: data.name,
            email: data.email,
            createdAt: new Date().toISOString(),
            updatedAt: new Date().toISOString()
        });

        await this.emailService.send(
            user.email,
            "Welcome!",
            `Hello ${user.name}, welcome to our platform!`
        );

        this.logger.log(`User registered: ${user.id}`);
        return user;
    }
}

// Simple DI container
class Container {
    private services = new Map<string, unknown>();

    register<T>(token: string, instance: T): void {
        this.services.set(token, instance);
    }

    resolve<T>(token: string): T {
        const service = this.services.get(token);
        if (!service) {
            throw new Error(`Service not found: ${token}`);
        }
        return service as T;
    }
}

// Setup
const container = new Container();
container.register<ILogger>("logger", new ConsoleLogger());
container.register<IUserRepository>("userRepo", new UserRepository(db));
container.register<IEmailService>("emailService", new EmailService());

const userService = new UserService(
    container.resolve<ILogger>("logger"),
    container.resolve<IUserRepository>("userRepo"),
    container.resolve<IEmailService>("emailService")
);
```

---

## Practice Exercises

### Exercise 1: Type-Safe Event System

```typescript
// Create a type-safe event system for a shopping cart
// Events: itemAdded, itemRemoved, cartCleared, checkoutStarted

// Your code here
```

**Solution:**
```typescript
interface CartItem {
    id: string;
    name: string;
    price: number;
    quantity: number;
}

interface CartEvents {
    itemAdded: { item: CartItem };
    itemRemoved: { itemId: string };
    itemUpdated: { itemId: string; quantity: number };
    cartCleared: {};
    checkoutStarted: { items: CartItem[]; total: number };
    checkoutCompleted: { orderId: string };
}

type EventHandler<T> = (payload: T) => void;

class ShoppingCart {
    private items: Map<string, CartItem> = new Map();
    private handlers: Map<keyof CartEvents, Set<EventHandler<any>>> = new Map();

    on<K extends keyof CartEvents>(
        event: K,
        handler: EventHandler<CartEvents[K]>
    ): () => void {
        if (!this.handlers.has(event)) {
            this.handlers.set(event, new Set());
        }
        this.handlers.get(event)!.add(handler);

        // Return unsubscribe function
        return () => {
            this.handlers.get(event)?.delete(handler);
        };
    }

    private emit<K extends keyof CartEvents>(
        event: K,
        payload: CartEvents[K]
    ): void {
        this.handlers.get(event)?.forEach(handler => handler(payload));
    }

    addItem(item: CartItem): void {
        this.items.set(item.id, item);
        this.emit("itemAdded", { item });
    }

    removeItem(itemId: string): void {
        this.items.delete(itemId);
        this.emit("itemRemoved", { itemId });
    }

    updateQuantity(itemId: string, quantity: number): void {
        const item = this.items.get(itemId);
        if (item) {
            item.quantity = quantity;
            this.emit("itemUpdated", { itemId, quantity });
        }
    }

    clear(): void {
        this.items.clear();
        this.emit("cartCleared", {});
    }

    checkout(): string {
        const items = Array.from(this.items.values());
        const total = items.reduce((sum, item) => sum + item.price * item.quantity, 0);

        this.emit("checkoutStarted", { items, total });

        const orderId = `order_${Date.now()}`;
        this.emit("checkoutCompleted", { orderId });

        this.clear();
        return orderId;
    }

    getItems(): CartItem[] {
        return Array.from(this.items.values());
    }
}

// Usage
const cart = new ShoppingCart();

const unsubscribe = cart.on("itemAdded", ({ item }) => {
    console.log(`Added: ${item.name}`);
});

cart.on("checkoutStarted", ({ total }) => {
    console.log(`Checkout started. Total: $${total}`);
});

cart.addItem({ id: "1", name: "Laptop", price: 999, quantity: 1 });
cart.addItem({ id: "2", name: "Mouse", price: 29, quantity: 2 });

const orderId = cart.checkout();
console.log(`Order completed: ${orderId}`);
```

### Exercise 2: Type-Safe Form Validation

```typescript
// Create a type-safe form validation system
// Support: required, email, minLength, maxLength, pattern

// Your code here
```

**Solution:**
```typescript
type ValidationRule<T> = {
    validate: (value: T) => boolean;
    message: string;
};

type FieldRules<T> = {
    [K in keyof T]?: ValidationRule<T[K]>[];
};

type ValidationErrors<T> = {
    [K in keyof T]?: string[];
};

type ValidationResult<T> =
    | { valid: true; data: T }
    | { valid: false; errors: ValidationErrors<T> };

// Built-in validators
const validators = {
    required: <T>(message = "This field is required"): ValidationRule<T> => ({
        validate: (value) => value !== null && value !== undefined && value !== "",
        message
    }),

    email: (message = "Invalid email format"): ValidationRule<string> => ({
        validate: (value) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value),
        message
    }),

    minLength: (min: number, message?: string): ValidationRule<string> => ({
        validate: (value) => value.length >= min,
        message: message ?? `Must be at least ${min} characters`
    }),

    maxLength: (max: number, message?: string): ValidationRule<string> => ({
        validate: (value) => value.length <= max,
        message: message ?? `Must be no more than ${max} characters`
    }),

    pattern: (regex: RegExp, message: string): ValidationRule<string> => ({
        validate: (value) => regex.test(value),
        message
    }),

    min: (min: number, message?: string): ValidationRule<number> => ({
        validate: (value) => value >= min,
        message: message ?? `Must be at least ${min}`
    }),

    max: (max: number, message?: string): ValidationRule<number> => ({
        validate: (value) => value <= max,
        message: message ?? `Must be no more than ${max}`
    })
};

class FormValidator<T extends Record<string, unknown>> {
    constructor(private rules: FieldRules<T>) {}

    validate(data: T): ValidationResult<T> {
        const errors: ValidationErrors<T> = {};
        let hasErrors = false;

        for (const field in this.rules) {
            const fieldRules = this.rules[field];
            if (!fieldRules) continue;

            const value = data[field];
            const fieldErrors: string[] = [];

            for (const rule of fieldRules) {
                if (!rule.validate(value as any)) {
                    fieldErrors.push(rule.message);
                }
            }

            if (fieldErrors.length > 0) {
                errors[field] = fieldErrors;
                hasErrors = true;
            }
        }

        if (hasErrors) {
            return { valid: false, errors };
        }

        return { valid: true, data };
    }
}

// Usage
interface RegistrationForm {
    email: string;
    password: string;
    confirmPassword: string;
    age: number;
}

const registrationValidator = new FormValidator<RegistrationForm>({
    email: [
        validators.required("Email is required"),
        validators.email()
    ],
    password: [
        validators.required("Password is required"),
        validators.minLength(8, "Password must be at least 8 characters"),
        validators.pattern(
            /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/,
            "Password must contain uppercase, lowercase, and number"
        )
    ],
    confirmPassword: [
        validators.required("Please confirm your password")
    ],
    age: [
        validators.required("Age is required"),
        validators.min(18, "Must be 18 or older")
    ]
});

const result = registrationValidator.validate({
    email: "test@example.com",
    password: "SecurePass123",
    confirmPassword: "SecurePass123",
    age: 25
});

if (result.valid) {
    console.log("Form is valid:", result.data);
} else {
    console.log("Validation errors:", result.errors);
}
```

---

## Key Takeaways

1. **Type definitions** - Install @types packages or create custom .d.ts files
2. **Error handling** - Use custom error classes and Result types
3. **API types** - Create strict types for requests and responses
4. **Configuration** - Type environment variables and config objects
5. **Testing** - Use type-safe mocks and factories
6. **Patterns** - Apply Builder, Repository, Singleton, DI patterns with types
7. **Module augmentation** - Extend third-party types when needed

---

## Self-Check Questions

1. How do you add types for a library without @types?
2. What's the difference between Result type and throwing errors?
3. How do you type environment variables?
4. What makes a mock "type-safe"?
5. When should you use the Builder pattern?
6. How do you extend Express Request types?

---

**Next Lesson:** [Day 10 - TypeScript with React](./day-10-ts-with-react.md)
