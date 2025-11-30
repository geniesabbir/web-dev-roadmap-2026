# Week 4, Day 5: Error Handling

## Introduction

Errors are inevitable in programming. Good error handling makes your code robust, debuggable, and user-friendly.

---

## Types of Errors

### Syntax Errors

Caught before code runs:

```javascript
// Missing parenthesis
function greet( {
    console.log("Hello");
}
// SyntaxError: Unexpected token '{'

// Missing quote
const message = "Hello;
// SyntaxError: Invalid or unexpected token

// Reserved word as variable
const class = "Math";
// SyntaxError: Unexpected token 'class'
```

### Runtime Errors

Occur during execution:

```javascript
// ReferenceError - variable doesn't exist
console.log(undefinedVariable);
// ReferenceError: undefinedVariable is not defined

// TypeError - wrong type operation
null.toString();
// TypeError: Cannot read property 'toString' of null

const obj = {};
obj.method();
// TypeError: obj.method is not a function

// RangeError - value out of range
const arr = new Array(-1);
// RangeError: Invalid array length

// URIError - bad URI
decodeURI("%");
// URIError: URI malformed
```

### Logic Errors

Code runs but produces wrong results:

```javascript
// No error thrown, but wrong result
function calculateArea(width, height) {
    return width + height;  // Should be width * height
}

// Off-by-one error
for (let i = 0; i <= arr.length; i++) {  // Should be < arr.length
    console.log(arr[i]);  // undefined on last iteration
}
```

---

## try/catch/finally

Handle runtime errors:

```javascript
// Basic try/catch
try {
    const data = JSON.parse("invalid json");
} catch (error) {
    console.error("Parsing failed:", error.message);
}

// With finally
try {
    // Code that might throw
    const result = riskyOperation();
    return result;
} catch (error) {
    // Handle error
    console.error("Operation failed:", error);
    return null;
} finally {
    // Always runs (cleanup)
    closeConnection();
}

// Error object properties
try {
    throw new Error("Something went wrong");
} catch (error) {
    console.log(error.name);      // "Error"
    console.log(error.message);   // "Something went wrong"
    console.log(error.stack);     // Stack trace
}

// Catch without binding (ES2019)
try {
    riskyOperation();
} catch {
    console.log("An error occurred");
}
```

### Nested try/catch

```javascript
try {
    try {
        throw new Error("Inner error");
    } catch (innerError) {
        console.log("Caught inner:", innerError.message);
        throw new Error("Outer error");
    }
} catch (outerError) {
    console.log("Caught outer:", outerError.message);
}
// Caught inner: Inner error
// Caught outer: Outer error

// Re-throwing errors
try {
    try {
        throw new Error("Original error");
    } catch (error) {
        console.log("Logging error...");
        throw error;  // Re-throw same error
    }
} catch (error) {
    console.log("Final handler:", error.message);
}
```

---

## Throwing Errors

Create and throw your own errors:

```javascript
// Throw Error object
throw new Error("Something went wrong");

// Throw custom message
function divide(a, b) {
    if (b === 0) {
        throw new Error("Cannot divide by zero");
    }
    return a / b;
}

// Throw with condition
function validateAge(age) {
    if (typeof age !== "number") {
        throw new TypeError("Age must be a number");
    }
    if (age < 0 || age > 150) {
        throw new RangeError("Age must be between 0 and 150");
    }
    return true;
}

// Throw different types
throw new TypeError("Expected a string");
throw new RangeError("Value out of range");
throw new ReferenceError("Variable not defined");

// Throw non-Error values (not recommended)
throw "Error message";  // String
throw 404;              // Number
throw { message: "Error" };  // Object
// These lose stack trace info
```

---

## Custom Error Classes

Create specialized error types:

```javascript
// Basic custom error
class ValidationError extends Error {
    constructor(message) {
        super(message);
        this.name = "ValidationError";
    }
}

throw new ValidationError("Invalid input");

// With additional properties
class ApiError extends Error {
    constructor(message, status, data = null) {
        super(message);
        this.name = "ApiError";
        this.status = status;
        this.data = data;

        // Maintains proper stack trace
        if (Error.captureStackTrace) {
            Error.captureStackTrace(this, ApiError);
        }
    }

    toJSON() {
        return {
            name: this.name,
            message: this.message,
            status: this.status,
            data: this.data
        };
    }
}

// Usage
throw new ApiError("Not found", 404, { resource: "user" });

// Handling custom errors
try {
    await fetchUser(999);
} catch (error) {
    if (error instanceof ApiError) {
        if (error.status === 404) {
            showNotFoundPage();
        } else if (error.status === 401) {
            redirectToLogin();
        }
    } else {
        throw error;  // Re-throw unknown errors
    }
}
```

### Error Hierarchy

```javascript
// Base application error
class AppError extends Error {
    constructor(message, code) {
        super(message);
        this.name = this.constructor.name;
        this.code = code;
        Error.captureStackTrace?.(this, this.constructor);
    }
}

// Specific error types
class NetworkError extends AppError {
    constructor(message = "Network error") {
        super(message, "NETWORK_ERROR");
    }
}

class ValidationError extends AppError {
    constructor(message, field) {
        super(message, "VALIDATION_ERROR");
        this.field = field;
    }
}

class AuthenticationError extends AppError {
    constructor(message = "Authentication required") {
        super(message, "AUTH_ERROR");
    }
}

class NotFoundError extends AppError {
    constructor(resource) {
        super(`${resource} not found`, "NOT_FOUND");
        this.resource = resource;
    }
}

// Usage
try {
    throw new NotFoundError("User");
} catch (error) {
    if (error instanceof AppError) {
        console.log(`${error.code}: ${error.message}`);
    }
}
```

---

## Async Error Handling

### Promises

```javascript
// .catch() for Promise errors
fetchData()
    .then(data => processData(data))
    .then(result => displayResult(result))
    .catch(error => {
        console.error("Error:", error);
    });

// Chain catches for different handling
fetchData()
    .then(data => {
        if (!data.valid) {
            throw new ValidationError("Invalid data");
        }
        return processData(data);
    })
    .catch(error => {
        if (error instanceof ValidationError) {
            return getDefaultData();  // Recovery
        }
        throw error;  // Re-throw other errors
    })
    .then(data => displayData(data))
    .catch(error => {
        console.error("Final error handler:", error);
    });

// Promise.all error handling
Promise.all([
    fetchUser(1),
    fetchUser(2),
    fetchUser(3)
])
    .then(users => console.log(users))
    .catch(error => {
        // If ANY promise rejects
        console.error("One failed:", error);
    });

// Promise.allSettled for partial success
const results = await Promise.allSettled([
    fetchUser(1),
    fetchUser(999),  // Might fail
    fetchUser(3)
]);

results.forEach((result, index) => {
    if (result.status === "fulfilled") {
        console.log(`User ${index}:`, result.value);
    } else {
        console.error(`User ${index} failed:`, result.reason);
    }
});
```

### async/await

```javascript
// Basic try/catch with async
async function fetchUserData(userId) {
    try {
        const response = await fetch(`/api/users/${userId}`);

        if (!response.ok) {
            throw new ApiError("Fetch failed", response.status);
        }

        const data = await response.json();
        return data;

    } catch (error) {
        if (error instanceof ApiError) {
            console.error(`API Error ${error.status}:`, error.message);
        } else {
            console.error("Unexpected error:", error);
        }
        throw error;  // Re-throw if needed
    }
}

// Multiple async operations
async function processOrder(orderId) {
    let order, user, inventory;

    try {
        order = await fetchOrder(orderId);
    } catch (error) {
        throw new Error(`Failed to fetch order: ${error.message}`);
    }

    try {
        user = await fetchUser(order.userId);
    } catch (error) {
        throw new Error(`Failed to fetch user: ${error.message}`);
    }

    try {
        inventory = await checkInventory(order.items);
    } catch (error) {
        throw new Error(`Failed to check inventory: ${error.message}`);
    }

    return { order, user, inventory };
}

// Wrapper for async error handling
function tryCatch(promise) {
    return promise
        .then(data => [null, data])
        .catch(error => [error, null]);
}

// Usage
async function main() {
    const [error, data] = await tryCatch(fetchData());

    if (error) {
        console.error("Error:", error);
        return;
    }

    console.log("Data:", data);
}
```

---

## Global Error Handling

### Browser

```javascript
// Unhandled errors
window.addEventListener("error", (event) => {
    console.error("Global error:", event.error);

    // Send to error tracking service
    errorTracker.log({
        message: event.message,
        filename: event.filename,
        lineno: event.lineno,
        colno: event.colno,
        error: event.error
    });

    // Prevent default browser handling
    event.preventDefault();
});

// Unhandled promise rejections
window.addEventListener("unhandledrejection", (event) => {
    console.error("Unhandled rejection:", event.reason);

    errorTracker.log({
        type: "unhandledrejection",
        reason: event.reason
    });

    event.preventDefault();
});

// Resource loading errors (images, scripts, etc.)
window.addEventListener("error", (event) => {
    if (event.target !== window) {
        console.error("Resource failed to load:", event.target.src);
    }
}, true);  // Use capture phase
```

### Node.js

```javascript
// Uncaught exceptions
process.on("uncaughtException", (error) => {
    console.error("Uncaught exception:", error);

    // Log error
    logger.error(error);

    // Graceful shutdown
    process.exit(1);
});

// Unhandled promise rejections
process.on("unhandledRejection", (reason, promise) => {
    console.error("Unhandled rejection:", reason);

    // In Node 15+, this causes exit by default
});

// Graceful shutdown
const gracefulShutdown = () => {
    console.log("Shutting down gracefully...");

    server.close(() => {
        database.disconnect(() => {
            process.exit(0);
        });
    });

    // Force exit after timeout
    setTimeout(() => {
        process.exit(1);
    }, 10000);
};

process.on("SIGTERM", gracefulShutdown);
process.on("SIGINT", gracefulShutdown);
```

---

## Error Logging and Tracking

### Creating a Logger

```javascript
class Logger {
    constructor(options = {}) {
        this.level = options.level || "info";
        this.levels = {
            error: 0,
            warn: 1,
            info: 2,
            debug: 3
        };
    }

    shouldLog(level) {
        return this.levels[level] <= this.levels[this.level];
    }

    formatMessage(level, message, data) {
        const timestamp = new Date().toISOString();
        const dataStr = data ? ` ${JSON.stringify(data)}` : "";
        return `[${timestamp}] [${level.toUpperCase()}] ${message}${dataStr}`;
    }

    log(level, message, data) {
        if (!this.shouldLog(level)) return;

        const formatted = this.formatMessage(level, message, data);

        switch (level) {
            case "error":
                console.error(formatted);
                break;
            case "warn":
                console.warn(formatted);
                break;
            default:
                console.log(formatted);
        }
    }

    error(message, data) { this.log("error", message, data); }
    warn(message, data) { this.log("warn", message, data); }
    info(message, data) { this.log("info", message, data); }
    debug(message, data) { this.log("debug", message, data); }
}

const logger = new Logger({ level: "debug" });
logger.error("Something went wrong", { userId: 123 });
```

### Error Tracking Service

```javascript
class ErrorTracker {
    constructor(apiEndpoint) {
        this.endpoint = apiEndpoint;
        this.queue = [];
        this.isSending = false;

        this.setupGlobalHandlers();
    }

    setupGlobalHandlers() {
        window.addEventListener("error", (e) => {
            this.capture(e.error);
        });

        window.addEventListener("unhandledrejection", (e) => {
            this.capture(e.reason);
        });
    }

    capture(error) {
        const errorData = {
            message: error?.message || String(error),
            stack: error?.stack,
            timestamp: Date.now(),
            url: window.location.href,
            userAgent: navigator.userAgent
        };

        this.queue.push(errorData);
        this.flush();
    }

    async flush() {
        if (this.isSending || this.queue.length === 0) return;

        this.isSending = true;
        const errors = [...this.queue];
        this.queue = [];

        try {
            await fetch(this.endpoint, {
                method: "POST",
                headers: { "Content-Type": "application/json" },
                body: JSON.stringify({ errors })
            });
        } catch (e) {
            // Put errors back in queue
            this.queue.unshift(...errors);
        } finally {
            this.isSending = false;
        }
    }
}

const errorTracker = new ErrorTracker("/api/errors");
```

---

## Error Handling Patterns

### Result Type Pattern

```javascript
// Return result object instead of throwing
class Result {
    constructor(value, error = null) {
        this.value = value;
        this.error = error;
    }

    static ok(value) {
        return new Result(value, null);
    }

    static fail(error) {
        return new Result(null, error);
    }

    get isOk() {
        return this.error === null;
    }

    get isError() {
        return this.error !== null;
    }

    unwrap() {
        if (this.isError) {
            throw this.error;
        }
        return this.value;
    }

    map(fn) {
        if (this.isError) return this;
        return Result.ok(fn(this.value));
    }

    flatMap(fn) {
        if (this.isError) return this;
        return fn(this.value);
    }
}

// Usage
function divide(a, b) {
    if (b === 0) {
        return Result.fail(new Error("Cannot divide by zero"));
    }
    return Result.ok(a / b);
}

const result = divide(10, 2);
if (result.isOk) {
    console.log("Result:", result.value);
} else {
    console.error("Error:", result.error);
}

// Chaining
const final = divide(10, 2)
    .map(x => x * 2)
    .map(x => x + 1);

console.log(final.value);  // 11
```

### Error Boundary Pattern (React)

```javascript
class ErrorBoundary extends React.Component {
    constructor(props) {
        super(props);
        this.state = { hasError: false, error: null };
    }

    static getDerivedStateFromError(error) {
        return { hasError: true, error };
    }

    componentDidCatch(error, errorInfo) {
        // Log to error tracking service
        errorTracker.capture(error, errorInfo);
    }

    render() {
        if (this.state.hasError) {
            return (
                <div className="error-fallback">
                    <h2>Something went wrong</h2>
                    <button onClick={() => this.setState({ hasError: false })}>
                        Try again
                    </button>
                </div>
            );
        }

        return this.props.children;
    }
}

// Usage
<ErrorBoundary>
    <MyComponent />
</ErrorBoundary>
```

### Retry Pattern

```javascript
async function withRetry(fn, options = {}) {
    const {
        retries = 3,
        delay = 1000,
        backoff = 2,
        shouldRetry = () => true
    } = options;

    let lastError;

    for (let attempt = 1; attempt <= retries; attempt++) {
        try {
            return await fn();
        } catch (error) {
            lastError = error;

            if (attempt === retries || !shouldRetry(error)) {
                throw error;
            }

            console.log(`Attempt ${attempt} failed, retrying...`);
            await new Promise(r => setTimeout(r, delay * Math.pow(backoff, attempt - 1)));
        }
    }

    throw lastError;
}

// Usage
const data = await withRetry(
    () => fetch("/api/data").then(r => r.json()),
    {
        retries: 3,
        delay: 1000,
        shouldRetry: (error) => error.status !== 401
    }
);
```

### Circuit Breaker Pattern

```javascript
class CircuitBreaker {
    constructor(fn, options = {}) {
        this.fn = fn;
        this.failureThreshold = options.failureThreshold || 5;
        this.resetTimeout = options.resetTimeout || 30000;

        this.state = "CLOSED";
        this.failures = 0;
        this.lastFailureTime = null;
    }

    async call(...args) {
        if (this.state === "OPEN") {
            if (Date.now() - this.lastFailureTime > this.resetTimeout) {
                this.state = "HALF_OPEN";
            } else {
                throw new Error("Circuit breaker is OPEN");
            }
        }

        try {
            const result = await this.fn(...args);
            this.onSuccess();
            return result;
        } catch (error) {
            this.onFailure();
            throw error;
        }
    }

    onSuccess() {
        this.failures = 0;
        this.state = "CLOSED";
    }

    onFailure() {
        this.failures++;
        this.lastFailureTime = Date.now();

        if (this.failures >= this.failureThreshold) {
            this.state = "OPEN";
        }
    }
}

// Usage
const protectedFetch = new CircuitBreaker(
    (url) => fetch(url).then(r => r.json()),
    { failureThreshold: 3, resetTimeout: 10000 }
);

try {
    const data = await protectedFetch.call("/api/data");
} catch (error) {
    if (error.message.includes("Circuit breaker")) {
        // Use cached data or show error
    }
}
```

---

## Best Practices

### 1. Be Specific with Errors

```javascript
// Bad
throw new Error("Error");

// Good
throw new ValidationError("Email format is invalid", "email");
throw new NotFoundError("User with ID 123");
throw new AuthenticationError("Token expired");
```

### 2. Don't Swallow Errors

```javascript
// Bad - error is hidden
try {
    riskyOperation();
} catch (error) {
    // Nothing here
}

// Good - at least log it
try {
    riskyOperation();
} catch (error) {
    console.error("Operation failed:", error);
    // Handle appropriately
}
```

### 3. Clean Up Resources

```javascript
// Use finally for cleanup
const file = openFile("data.txt");
try {
    processFile(file);
} catch (error) {
    console.error("Processing failed:", error);
} finally {
    closeFile(file);  // Always runs
}
```

### 4. Provide Context

```javascript
// Bad
throw new Error("Failed");

// Good
throw new Error(`Failed to process order ${orderId}: ${originalError.message}`);

// Or wrap errors
class ProcessingError extends Error {
    constructor(message, orderId, originalError) {
        super(message);
        this.orderId = orderId;
        this.originalError = originalError;
    }
}
```

### 5. Fail Fast

```javascript
// Validate early
function processOrder(order) {
    if (!order) {
        throw new ValidationError("Order is required");
    }
    if (!order.items?.length) {
        throw new ValidationError("Order must have items");
    }
    if (!order.userId) {
        throw new ValidationError("Order must have user ID");
    }

    // Now proceed with processing...
}
```

---

## Practice Exercises

### Exercise 1: Create a Safe JSON Parser

```javascript
// Create a function that safely parses JSON
// Returns default value on failure
function safeJsonParse(json, defaultValue = null) {
    // Your code here
}
```

**Solution:**
```javascript
function safeJsonParse(json, defaultValue = null) {
    try {
        return JSON.parse(json);
    } catch (error) {
        console.warn("JSON parse failed:", error.message);
        return defaultValue;
    }
}

// Usage
const data = safeJsonParse('{"valid": true}');  // { valid: true }
const fallback = safeJsonParse('invalid', { default: true });  // { default: true }
```

### Exercise 2: Create Error with Context

```javascript
// Create a function that wraps errors with context
function withContext(fn, context) {
    // Your code here
}
```

**Solution:**
```javascript
function withContext(fn, context) {
    return async (...args) => {
        try {
            return await fn(...args);
        } catch (error) {
            const wrappedError = new Error(`${context}: ${error.message}`);
            wrappedError.originalError = error;
            wrappedError.stack = error.stack;
            throw wrappedError;
        }
    };
}

// Usage
const fetchUser = withContext(
    async (id) => {
        const response = await fetch(`/api/users/${id}`);
        if (!response.ok) throw new Error("Not found");
        return response.json();
    },
    "Failed to fetch user"
);

// Error: "Failed to fetch user: Not found"
```

---

## Key Takeaways

1. **Use try/catch/finally** for synchronous error handling
2. **.catch() or try/catch** for async error handling
3. **Create custom error classes** for specific error types
4. **Never swallow errors** - at minimum, log them
5. **Provide context** in error messages
6. **Handle errors at the right level** - not too early, not too late
7. **Set up global handlers** as a safety net
8. **Use error tracking** in production
9. **Fail fast** - validate early
10. **Clean up resources** in finally blocks

---

## Congratulations!

You've completed the JavaScript fundamentals section! You now have a solid understanding of:
- Variables, data types, and operators
- Functions and scope
- Objects and arrays
- DOM manipulation
- Events
- Asynchronous JavaScript
- ES6+ features
- Modules
- Error handling

**Next:** Move on to TypeScript to add type safety to your JavaScript!
