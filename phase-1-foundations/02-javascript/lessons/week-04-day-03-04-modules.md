# Week 4, Day 3-4: JavaScript Modules

## Introduction to Modules

Modules allow you to organize code into separate files, making it more maintainable and reusable. Each module has its own scope.

**Benefits:**
- Code organization
- Reusability
- Encapsulation
- Dependency management
- Easier testing

---

## ES Modules (ESM)

The modern, standard module system for JavaScript:

### Export Syntax

```javascript
// math.js

// Named exports
export const PI = 3.14159;

export function add(a, b) {
    return a + b;
}

export function subtract(a, b) {
    return a - b;
}

export class Calculator {
    multiply(a, b) {
        return a * b;
    }
}

// Export list
const divide = (a, b) => a / b;
const modulo = (a, b) => a % b;
export { divide, modulo };

// Rename exports
const power = (a, b) => a ** b;
export { power as pow };

// Default export (one per module)
export default function sqrt(n) {
    return Math.sqrt(n);
}
```

### Import Syntax

```javascript
// main.js

// Import default export
import sqrt from "./math.js";

// Import named exports
import { add, subtract, PI } from "./math.js";

// Import with rename
import { add as sum, subtract as minus } from "./math.js";

// Import all as namespace
import * as math from "./math.js";
console.log(math.add(1, 2));
console.log(math.PI);
console.log(math.default(16));  // sqrt

// Import default and named together
import sqrt, { add, subtract } from "./math.js";

// Side-effect import (runs module code, no bindings)
import "./init.js";
```

### Dynamic Imports

```javascript
// Import modules conditionally or on demand
async function loadModule() {
    if (condition) {
        const math = await import("./math.js");
        console.log(math.add(1, 2));
    }
}

// With destructuring
async function calculate() {
    const { add, subtract } = await import("./math.js");
    return add(5, 3);
}

// In event handlers
button.addEventListener("click", async () => {
    const { showModal } = await import("./modal.js");
    showModal();
});

// Code splitting
const routes = {
    "/home": () => import("./pages/home.js"),
    "/about": () => import("./pages/about.js"),
    "/contact": () => import("./pages/contact.js")
};

async function loadPage(path) {
    const module = await routes[path]();
    module.render();
}
```

---

## Module Patterns

### Barrel Files (Re-exports)

Consolidate exports from multiple modules:

```javascript
// components/index.js (barrel file)
export { Button } from "./Button.js";
export { Input } from "./Input.js";
export { Modal } from "./Modal.js";
export { default as Card } from "./Card.js";

// Re-export everything
export * from "./utils.js";

// Re-export with rename
export { helper as utilHelper } from "./utils.js";
```

```javascript
// Usage - import from barrel
import { Button, Input, Modal, Card } from "./components/index.js";
// or
import { Button, Input } from "./components";  // index.js is default
```

### Singleton Pattern

```javascript
// config.js
class Config {
    constructor() {
        this.settings = {
            apiUrl: "https://api.example.com",
            timeout: 5000
        };
    }

    get(key) {
        return this.settings[key];
    }

    set(key, value) {
        this.settings[key] = value;
    }
}

// Export single instance
export default new Config();

// All imports get the same instance
```

```javascript
// Usage
import config from "./config.js";
config.set("timeout", 10000);

// In another file
import config from "./config.js";
console.log(config.get("timeout"));  // 10000 (same instance)
```

### Factory Pattern

```javascript
// factory.js
export function createLogger(prefix) {
    return {
        log(message) {
            console.log(`[${prefix}] ${message}`);
        },
        error(message) {
            console.error(`[${prefix}] ERROR: ${message}`);
        }
    };
}
```

```javascript
// Usage
import { createLogger } from "./factory.js";

const appLogger = createLogger("App");
const apiLogger = createLogger("API");

appLogger.log("Started");
apiLogger.error("Failed to fetch");
```

### Service Module Pattern

```javascript
// api/userService.js
const API_URL = "https://api.example.com";

async function getUser(id) {
    const response = await fetch(`${API_URL}/users/${id}`);
    return response.json();
}

async function createUser(data) {
    const response = await fetch(`${API_URL}/users`, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(data)
    });
    return response.json();
}

async function updateUser(id, data) {
    const response = await fetch(`${API_URL}/users/${id}`, {
        method: "PUT",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(data)
    });
    return response.json();
}

async function deleteUser(id) {
    const response = await fetch(`${API_URL}/users/${id}`, {
        method: "DELETE"
    });
    return response.ok;
}

export const userService = {
    getUser,
    createUser,
    updateUser,
    deleteUser
};
```

---

## CommonJS (Node.js)

The module system originally used in Node.js:

### Exporting

```javascript
// math.js

// Individual exports
exports.add = (a, b) => a + b;
exports.subtract = (a, b) => a - b;
exports.PI = 3.14159;

// Or using module.exports
module.exports = {
    add: (a, b) => a + b,
    subtract: (a, b) => a - b,
    PI: 3.14159
};

// Default export style
module.exports = function sqrt(n) {
    return Math.sqrt(n);
};

// Class export
class Calculator {
    add(a, b) { return a + b; }
}
module.exports = Calculator;
```

### Importing

```javascript
// main.js

// Import entire module
const math = require("./math.js");
console.log(math.add(1, 2));
console.log(math.PI);

// Destructuring
const { add, subtract } = require("./math.js");
console.log(add(1, 2));

// Import default
const sqrt = require("./math.js");
console.log(sqrt(16));
```

### ESM vs CommonJS

```javascript
// ESM (modern, use in browser and Node.js)
import { add } from "./math.js";
export const result = add(1, 2);

// CommonJS (Node.js traditional)
const { add } = require("./math");
module.exports.result = add(1, 2);

// Key differences:
// 1. ESM is statically analyzed (imports at top)
// 2. CommonJS is dynamic (can require conditionally)
// 3. ESM has named exports and default export
// 4. ESM uses file extension in browser
// 5. ESM supports top-level await
```

---

## Using Modules in Browser

### Script Module

```html
<!-- Type module for ESM -->
<script type="module" src="main.js"></script>

<!-- Inline module -->
<script type="module">
    import { greet } from "./utils.js";
    greet("World");
</script>

<!-- Fallback for old browsers -->
<script nomodule src="fallback.js"></script>
```

### CORS Considerations

```javascript
// Modules are fetched with CORS
// Local file:// won't work - use a server

// Development server options:
// - npx serve
// - python -m http.server
// - VS Code Live Server extension
```

### Import Maps

```html
<!-- Define module paths -->
<script type="importmap">
{
    "imports": {
        "lodash": "https://cdn.jsdelivr.net/npm/lodash-es@4.17.21/lodash.js",
        "@app/": "./src/",
        "utils": "./src/utils/index.js"
    }
}
</script>

<script type="module">
    import _ from "lodash";
    import { helper } from "@app/helpers.js";
    import { format } from "utils";
</script>
```

---

## Module Organization

### Project Structure

```
project/
├── src/
│   ├── index.js           # Entry point
│   ├── config.js          # Configuration
│   ├── constants.js       # Constants
│   │
│   ├── components/        # UI Components
│   │   ├── index.js       # Barrel file
│   │   ├── Button.js
│   │   ├── Input.js
│   │   └── Modal.js
│   │
│   ├── services/          # API/Business logic
│   │   ├── index.js
│   │   ├── api.js
│   │   ├── userService.js
│   │   └── authService.js
│   │
│   ├── utils/             # Utility functions
│   │   ├── index.js
│   │   ├── format.js
│   │   ├── validate.js
│   │   └── helpers.js
│   │
│   ├── hooks/             # Custom hooks (React)
│   │   └── index.js
│   │
│   └── types/             # TypeScript types
│       └── index.ts
│
├── tests/
│   └── ...
│
├── package.json
└── index.html
```

### Naming Conventions

```javascript
// Files
// PascalCase for components/classes
Button.js
UserService.js

// camelCase for utilities
formatDate.js
validateEmail.js

// kebab-case for config/constants
api-config.js
error-codes.js

// index.js for barrel exports
components/index.js
```

---

## Circular Dependencies

When modules depend on each other:

```javascript
// a.js
import { b } from "./b.js";
export const a = "A: " + b;

// b.js
import { a } from "./a.js";
export const b = "B: " + a;  // Problem!

// Solutions:

// 1. Restructure to remove circular dependency
// shared.js
export const shared = "shared value";

// a.js
import { shared } from "./shared.js";
export const a = "A: " + shared;

// b.js
import { shared } from "./shared.js";
export const b = "B: " + shared;

// 2. Use function to delay evaluation
// a.js
export function getA() {
    return "A: " + getB();
}

// b.js
import { getA } from "./a.js";
export function getB() {
    return "B";
}

// 3. Combine into single module
// combined.js
export const a = "A";
export const b = "B: " + a;
```

---

## Module Best Practices

### Single Responsibility

```javascript
// Bad - too many responsibilities
// utils.js
export function formatDate() {}
export function validateEmail() {}
export function fetchData() {}
export function sortArray() {}

// Good - separate by purpose
// format.js
export function formatDate() {}
export function formatCurrency() {}

// validate.js
export function validateEmail() {}
export function validatePhone() {}

// api.js
export function fetchData() {}
export function postData() {}
```

### Explicit Dependencies

```javascript
// Bad - global dependencies
window.$ = require("jquery");

// Good - explicit imports
import $ from "jquery";
```

### Avoid Side Effects

```javascript
// Bad - side effects on import
// init.js
console.log("Module loaded");
document.addEventListener("DOMContentLoaded", setup);

// Good - explicit initialization
// init.js
export function initialize() {
    console.log("Initializing...");
    document.addEventListener("DOMContentLoaded", setup);
}

// main.js
import { initialize } from "./init.js";
initialize();  // Explicit call
```

### Consistent Export Style

```javascript
// Pick one style and stick with it

// Style 1: Named exports only
export function add() {}
export function subtract() {}

// Style 2: Default export for main, named for utilities
export default class Calculator {}
export { add, subtract };

// Style 3: Object export
const calculator = { add, subtract };
export default calculator;
```

---

## Practical Examples

### Example 1: API Module

```javascript
// api/config.js
export const API_BASE = "https://api.example.com";
export const DEFAULT_HEADERS = {
    "Content-Type": "application/json"
};

// api/client.js
import { API_BASE, DEFAULT_HEADERS } from "./config.js";

class ApiClient {
    constructor(baseUrl = API_BASE) {
        this.baseUrl = baseUrl;
        this.token = null;
    }

    setToken(token) {
        this.token = token;
    }

    async request(endpoint, options = {}) {
        const url = `${this.baseUrl}${endpoint}`;
        const headers = {
            ...DEFAULT_HEADERS,
            ...(this.token && { Authorization: `Bearer ${this.token}` }),
            ...options.headers
        };

        const response = await fetch(url, {
            ...options,
            headers
        });

        if (!response.ok) {
            throw new Error(`HTTP ${response.status}`);
        }

        return response.json();
    }

    get(endpoint) {
        return this.request(endpoint);
    }

    post(endpoint, data) {
        return this.request(endpoint, {
            method: "POST",
            body: JSON.stringify(data)
        });
    }

    put(endpoint, data) {
        return this.request(endpoint, {
            method: "PUT",
            body: JSON.stringify(data)
        });
    }

    delete(endpoint) {
        return this.request(endpoint, { method: "DELETE" });
    }
}

export default new ApiClient();

// api/users.js
import api from "./client.js";

export async function getUsers() {
    return api.get("/users");
}

export async function getUser(id) {
    return api.get(`/users/${id}`);
}

export async function createUser(data) {
    return api.post("/users", data);
}

// api/index.js
export { default as api } from "./client.js";
export * from "./users.js";
export * from "./posts.js";
```

### Example 2: Event Bus Module

```javascript
// events/EventBus.js
class EventBus {
    #events = new Map();

    on(event, callback) {
        if (!this.#events.has(event)) {
            this.#events.set(event, new Set());
        }
        this.#events.get(event).add(callback);

        return () => this.off(event, callback);
    }

    off(event, callback) {
        if (this.#events.has(event)) {
            this.#events.get(event).delete(callback);
        }
    }

    emit(event, data) {
        if (this.#events.has(event)) {
            this.#events.get(event).forEach(cb => cb(data));
        }
    }

    once(event, callback) {
        const wrapper = (data) => {
            callback(data);
            this.off(event, wrapper);
        };
        return this.on(event, wrapper);
    }
}

// Singleton export
export default new EventBus();

// Usage
// component-a.js
import eventBus from "./events/EventBus.js";
eventBus.emit("user:login", { id: 1, name: "John" });

// component-b.js
import eventBus from "./events/EventBus.js";
eventBus.on("user:login", (user) => {
    console.log("User logged in:", user);
});
```

### Example 3: Store Module

```javascript
// store/store.js
function createStore(initialState = {}) {
    let state = initialState;
    const listeners = new Set();

    return {
        getState() {
            return { ...state };
        },

        setState(updater) {
            const newState = typeof updater === "function"
                ? updater(state)
                : updater;

            state = { ...state, ...newState };
            listeners.forEach(listener => listener(state));
        },

        subscribe(listener) {
            listeners.add(listener);
            return () => listeners.delete(listener);
        },

        select(selector) {
            return selector(state);
        }
    };
}

export default createStore;

// store/appStore.js
import createStore from "./store.js";

const initialState = {
    user: null,
    isLoading: false,
    items: []
};

export const store = createStore(initialState);

// Actions
export const actions = {
    setUser(user) {
        store.setState({ user });
    },

    setLoading(isLoading) {
        store.setState({ isLoading });
    },

    addItem(item) {
        store.setState(state => ({
            items: [...state.items, item]
        }));
    },

    removeItem(id) {
        store.setState(state => ({
            items: state.items.filter(item => item.id !== id)
        }));
    }
};

// Selectors
export const selectors = {
    getUser: state => state.user,
    isLoggedIn: state => state.user !== null,
    getItems: state => state.items
};
```

---

## Testing Modules

```javascript
// math.js
export function add(a, b) {
    return a + b;
}

export function divide(a, b) {
    if (b === 0) throw new Error("Cannot divide by zero");
    return a / b;
}

// math.test.js
import { add, divide } from "./math.js";

describe("Math module", () => {
    describe("add", () => {
        it("should add two numbers", () => {
            expect(add(2, 3)).toBe(5);
        });

        it("should handle negative numbers", () => {
            expect(add(-1, 1)).toBe(0);
        });
    });

    describe("divide", () => {
        it("should divide two numbers", () => {
            expect(divide(10, 2)).toBe(5);
        });

        it("should throw on division by zero", () => {
            expect(() => divide(10, 0)).toThrow("Cannot divide by zero");
        });
    });
});

// Mocking modules
// In Jest:
jest.mock("./api.js", () => ({
    fetchUser: jest.fn().mockResolvedValue({ id: 1, name: "John" })
}));
```

---

## Practice Exercises

### Exercise 1: Create a Module System

```javascript
// Create a counter module with:
// - Private count variable
// - increment(), decrement(), getCount(), reset()
```

**Solution:**
```javascript
// counter.js
let count = 0;

export function increment() {
    count++;
}

export function decrement() {
    count--;
}

export function getCount() {
    return count;
}

export function reset() {
    count = 0;
}

// Or as class
class Counter {
    #count = 0;

    increment() { this.#count++; }
    decrement() { this.#count--; }
    getCount() { return this.#count; }
    reset() { this.#count = 0; }
}

export default new Counter();
```

### Exercise 2: Create a Plugin System

```javascript
// Create a plugin system where modules can register plugins
```

**Solution:**
```javascript
// plugins.js
const plugins = new Map();

export function register(name, plugin) {
    if (plugins.has(name)) {
        throw new Error(`Plugin "${name}" already registered`);
    }
    plugins.set(name, plugin);
}

export function get(name) {
    return plugins.get(name);
}

export function getAll() {
    return [...plugins.entries()];
}

export function run(name, ...args) {
    const plugin = plugins.get(name);
    if (plugin && typeof plugin.execute === "function") {
        return plugin.execute(...args);
    }
    throw new Error(`Plugin "${name}" not found or has no execute method`);
}

// Usage
// loggerPlugin.js
import { register } from "./plugins.js";

register("logger", {
    execute(message) {
        console.log(`[LOG] ${message}`);
    }
});
```

---

## Key Takeaways

1. **ES Modules** are the standard - use `import`/`export`
2. **Named exports** for multiple items, **default** for main export
3. **Dynamic imports** for code splitting and lazy loading
4. **Barrel files** consolidate exports for cleaner imports
5. **Avoid circular dependencies** - restructure if needed
6. **One responsibility per module** keeps code maintainable
7. **Explicit dependencies** make code easier to understand
8. **CommonJS** (`require`) is still used in Node.js but ESM is the future

---

**Next Lesson:** [Day 5 - Error Handling](./week-04-day-05-error-handling.md)
