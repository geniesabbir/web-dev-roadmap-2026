# Week 3, Day 3-4: Asynchronous JavaScript

## Understanding Asynchronous Code

JavaScript is **single-threaded** but can handle asynchronous operations. This means long-running tasks don't block the main thread.

```javascript
// Synchronous (blocking)
console.log("1");
console.log("2");
console.log("3");
// Output: 1, 2, 3 (in order)

// Asynchronous (non-blocking)
console.log("1");
setTimeout(() => console.log("2"), 1000);
console.log("3");
// Output: 1, 3, 2 (2 comes after 1 second)
```

### The Event Loop

The event loop handles async operations:

1. **Call Stack** - Executes synchronous code
2. **Web APIs** - Handles async operations (setTimeout, fetch, etc.)
3. **Task Queue** - Holds callbacks ready to run
4. **Microtask Queue** - Holds Promise callbacks (priority over task queue)

```javascript
console.log("Start");

setTimeout(() => {
    console.log("Timeout");
}, 0);

Promise.resolve().then(() => {
    console.log("Promise");
});

console.log("End");

// Output:
// Start
// End
// Promise (microtask - runs first)
// Timeout (task queue - runs after microtasks)
```

---

## Callbacks

The original async pattern - passing functions to be called later:

```javascript
// Basic callback
function doSomething(callback) {
    console.log("Doing something...");
    callback();
}

doSomething(() => {
    console.log("Done!");
});

// Callback with data
function fetchData(callback) {
    setTimeout(() => {
        const data = { id: 1, name: "John" };
        callback(data);
    }, 1000);
}

fetchData((data) => {
    console.log("Received:", data);
});

// Error-first callback pattern (Node.js style)
function readFile(path, callback) {
    setTimeout(() => {
        if (path === "error") {
            callback(new Error("File not found"), null);
        } else {
            callback(null, "File contents");
        }
    }, 1000);
}

readFile("test.txt", (error, data) => {
    if (error) {
        console.error("Error:", error.message);
        return;
    }
    console.log("Data:", data);
});
```

### Callback Hell

Nested callbacks become hard to read and maintain:

```javascript
// The "Pyramid of Doom"
getUser(userId, (user) => {
    getPosts(user.id, (posts) => {
        getComments(posts[0].id, (comments) => {
            getReplies(comments[0].id, (replies) => {
                console.log("Finally got replies:", replies);
                // More nesting...
            });
        });
    });
});

// Problems:
// - Hard to read
// - Hard to handle errors
// - Hard to maintain
```

---

## Promises

Promises provide a cleaner way to handle async operations:

```javascript
// Creating a Promise
const promise = new Promise((resolve, reject) => {
    // Async operation
    setTimeout(() => {
        const success = true;

        if (success) {
            resolve("Operation successful!");
        } else {
            reject(new Error("Operation failed!"));
        }
    }, 1000);
});

// Using the Promise
promise
    .then((result) => {
        console.log("Success:", result);
    })
    .catch((error) => {
        console.error("Error:", error.message);
    });
```

### Promise States

A Promise can be in one of three states:

```javascript
// 1. Pending - initial state, not fulfilled or rejected
const pending = new Promise(() => {});
console.log(pending);  // Promise { <pending> }

// 2. Fulfilled - operation completed successfully
const fulfilled = Promise.resolve("Success");
console.log(fulfilled);  // Promise { 'Success' }

// 3. Rejected - operation failed
const rejected = Promise.reject(new Error("Failed"));
console.log(rejected);  // Promise { <rejected> Error: Failed }
```

### Promise Chaining

Chain multiple async operations:

```javascript
function step1() {
    return new Promise((resolve) => {
        setTimeout(() => resolve("Step 1 complete"), 500);
    });
}

function step2(data) {
    return new Promise((resolve) => {
        setTimeout(() => resolve(`${data} -> Step 2 complete`), 500);
    });
}

function step3(data) {
    return new Promise((resolve) => {
        setTimeout(() => resolve(`${data} -> Step 3 complete`), 500);
    });
}

// Chain them together
step1()
    .then(step2)
    .then(step3)
    .then((result) => {
        console.log("Final:", result);
        // "Step 1 complete -> Step 2 complete -> Step 3 complete"
    })
    .catch((error) => {
        console.error("Error:", error);
    });

// Compare to callbacks:
step1((result1) => {
    step2(result1, (result2) => {
        step3(result2, (result3) => {
            console.log(result3);
        });
    });
});
```

### Handling Errors

```javascript
// Single catch for the chain
fetchUser(userId)
    .then((user) => fetchPosts(user.id))
    .then((posts) => fetchComments(posts[0].id))
    .catch((error) => {
        // Catches any error in the chain
        console.error("Something went wrong:", error);
    });

// finally runs regardless of success/failure
fetchData()
    .then((data) => console.log(data))
    .catch((error) => console.error(error))
    .finally(() => {
        console.log("Cleanup - runs always");
        hideLoadingSpinner();
    });

// Handling specific errors
fetchUser(userId)
    .then((user) => {
        if (!user.active) {
            throw new Error("User is inactive");
        }
        return fetchPosts(user.id);
    })
    .catch((error) => {
        if (error.message === "User is inactive") {
            console.log("Showing inactive user message");
        } else {
            console.error("Unexpected error:", error);
        }
    });

// Recovering from errors
fetchData()
    .catch(() => {
        console.log("Using cached data");
        return getCachedData();
    })
    .then((data) => {
        console.log("Data:", data);
    });
```

### Promise Static Methods

```javascript
// Promise.resolve - create resolved promise
const resolved = Promise.resolve("value");
resolved.then(console.log);  // "value"

// Promise.reject - create rejected promise
const rejected = Promise.reject(new Error("error"));
rejected.catch(console.error);  // Error: error

// Promise.all - wait for ALL to complete
const promises = [
    fetch("/api/users"),
    fetch("/api/posts"),
    fetch("/api/comments")
];

Promise.all(promises)
    .then(([users, posts, comments]) => {
        console.log("All loaded!");
    })
    .catch((error) => {
        // If ANY fails, catch is called
        console.error("One failed:", error);
    });

// Promise.allSettled - wait for all, regardless of result
Promise.allSettled(promises)
    .then((results) => {
        results.forEach((result, i) => {
            if (result.status === "fulfilled") {
                console.log(`Promise ${i}: ${result.value}`);
            } else {
                console.log(`Promise ${i} failed: ${result.reason}`);
            }
        });
    });

// Promise.race - first to complete wins
Promise.race([
    fetch("/api/fast"),
    fetch("/api/slow")
])
    .then((first) => {
        console.log("First response:", first);
    });

// Promise.any - first successful wins
Promise.any([
    fetch("/api/server1"),
    fetch("/api/server2"),
    fetch("/api/server3")
])
    .then((first) => {
        console.log("First successful:", first);
    })
    .catch(() => {
        console.log("All failed!");
    });
```

### Creating Promises

```javascript
// Wrap callback-based function in Promise
function readFilePromise(path) {
    return new Promise((resolve, reject) => {
        fs.readFile(path, "utf8", (error, data) => {
            if (error) {
                reject(error);
            } else {
                resolve(data);
            }
        });
    });
}

// Wrap setTimeout
function delay(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
}

delay(1000).then(() => console.log("1 second passed"));

// Promisify pattern
function promisify(fn) {
    return function(...args) {
        return new Promise((resolve, reject) => {
            fn(...args, (error, result) => {
                if (error) reject(error);
                else resolve(result);
            });
        });
    };
}

const readFileAsync = promisify(fs.readFile);
```

---

## Async/Await

Modern syntax that makes async code look synchronous:

```javascript
// Using Promises
function fetchUserData(userId) {
    return fetch(`/api/users/${userId}`)
        .then(response => response.json())
        .then(user => {
            return fetch(`/api/posts?userId=${user.id}`);
        })
        .then(response => response.json());
}

// Using async/await
async function fetchUserData(userId) {
    const response = await fetch(`/api/users/${userId}`);
    const user = await response.json();

    const postsResponse = await fetch(`/api/posts?userId=${user.id}`);
    const posts = await postsResponse.json();

    return posts;
}

// Usage
fetchUserData(1)
    .then(posts => console.log(posts))
    .catch(error => console.error(error));

// Or with async IIFE
(async () => {
    const posts = await fetchUserData(1);
    console.log(posts);
})();
```

### Error Handling with try/catch

```javascript
async function getData() {
    try {
        const response = await fetch("/api/data");

        if (!response.ok) {
            throw new Error(`HTTP error! Status: ${response.status}`);
        }

        const data = await response.json();
        return data;

    } catch (error) {
        console.error("Error:", error.message);
        // Can rethrow, return default, or handle differently
        return null;
    } finally {
        console.log("Cleanup runs always");
    }
}

// Multiple try/catch for different errors
async function complexOperation() {
    let user;
    try {
        user = await fetchUser();
    } catch (error) {
        console.error("Failed to fetch user");
        throw error;  // Re-throw to stop execution
    }

    let posts;
    try {
        posts = await fetchPosts(user.id);
    } catch (error) {
        console.error("Failed to fetch posts, using empty array");
        posts = [];  // Use default
    }

    return { user, posts };
}
```

### Parallel vs Sequential

```javascript
// SEQUENTIAL - one after another (slow)
async function sequential() {
    const user = await fetchUser();     // Wait...
    const posts = await fetchPosts();   // Then wait...
    const comments = await fetchComments();  // Then wait...
    // Total time: user + posts + comments
}

// PARALLEL - all at once (fast)
async function parallel() {
    const [user, posts, comments] = await Promise.all([
        fetchUser(),
        fetchPosts(),
        fetchComments()
    ]);
    // Total time: max(user, posts, comments)
}

// Mixed - some parallel, some sequential
async function mixed() {
    // Fetch user first (needed for next calls)
    const user = await fetchUser();

    // Then fetch posts and comments in parallel
    const [posts, comments] = await Promise.all([
        fetchPosts(user.id),
        fetchComments(user.id)
    ]);

    return { user, posts, comments };
}
```

### Async in Different Contexts

```javascript
// Async function expression
const fetchData = async function() {
    return await getData();
};

// Async arrow function
const fetchData2 = async () => {
    return await getData();
};

// Async method in object
const obj = {
    async fetchData() {
        return await getData();
    }
};

// Async method in class
class DataService {
    async fetchData() {
        return await getData();
    }
}

// Async IIFE (Immediately Invoked)
(async () => {
    const data = await fetchData();
    console.log(data);
})();

// Top-level await (ES Modules only)
// In a .mjs file or type="module"
const data = await fetchData();
console.log(data);
```

### Common Patterns

```javascript
// Retry pattern
async function fetchWithRetry(url, retries = 3) {
    for (let i = 0; i < retries; i++) {
        try {
            const response = await fetch(url);
            if (!response.ok) throw new Error("Request failed");
            return await response.json();
        } catch (error) {
            if (i === retries - 1) throw error;
            console.log(`Retry ${i + 1}...`);
            await delay(1000 * (i + 1));  // Exponential backoff
        }
    }
}

// Timeout pattern
async function fetchWithTimeout(url, timeout = 5000) {
    const controller = new AbortController();
    const timeoutId = setTimeout(() => controller.abort(), timeout);

    try {
        const response = await fetch(url, { signal: controller.signal });
        return await response.json();
    } finally {
        clearTimeout(timeoutId);
    }
}

// Sequential processing of array
async function processSequentially(items) {
    const results = [];
    for (const item of items) {
        const result = await processItem(item);
        results.push(result);
    }
    return results;
}

// Parallel processing with limit
async function processWithLimit(items, limit) {
    const results = [];
    const executing = [];

    for (const item of items) {
        const promise = processItem(item).then(result => {
            executing.splice(executing.indexOf(promise), 1);
            return result;
        });

        results.push(promise);
        executing.push(promise);

        if (executing.length >= limit) {
            await Promise.race(executing);
        }
    }

    return Promise.all(results);
}
```

---

## Timers

### setTimeout

Execute code after a delay:

```javascript
// Basic usage
const timeoutId = setTimeout(() => {
    console.log("Executed after 2 seconds");
}, 2000);

// With arguments
setTimeout((name, greeting) => {
    console.log(`${greeting}, ${name}!`);
}, 1000, "John", "Hello");

// Cancel timeout
clearTimeout(timeoutId);

// Zero delay (still async)
setTimeout(() => console.log("Last"), 0);
console.log("First");
// Output: "First", "Last"
```

### setInterval

Execute code repeatedly:

```javascript
// Basic usage
let count = 0;
const intervalId = setInterval(() => {
    count++;
    console.log(`Count: ${count}`);

    if (count >= 5) {
        clearInterval(intervalId);
    }
}, 1000);

// Self-adjusting interval (for accuracy)
function accurateInterval(callback, interval) {
    let expected = Date.now() + interval;

    function step() {
        const drift = Date.now() - expected;
        callback();
        expected += interval;
        setTimeout(step, Math.max(0, interval - drift));
    }

    setTimeout(step, interval);
}
```

### requestAnimationFrame

For smooth animations:

```javascript
// Basic animation
let position = 0;

function animate() {
    position += 2;
    element.style.left = position + "px";

    if (position < 500) {
        requestAnimationFrame(animate);
    }
}

requestAnimationFrame(animate);

// With timestamp for smooth animation
let start;
function smoothAnimate(timestamp) {
    if (!start) start = timestamp;
    const progress = timestamp - start;

    // Move 100px per second
    element.style.left = Math.min(progress / 10, 100) + "px";

    if (progress < 1000) {
        requestAnimationFrame(smoothAnimate);
    }
}

requestAnimationFrame(smoothAnimate);

// Cancel animation
const animationId = requestAnimationFrame(animate);
cancelAnimationFrame(animationId);
```

---

## Practical Examples

### Example 1: Data Fetching Service

```javascript
class DataService {
    constructor(baseUrl) {
        this.baseUrl = baseUrl;
        this.cache = new Map();
    }

    async fetch(endpoint, options = {}) {
        const url = `${this.baseUrl}${endpoint}`;

        // Check cache
        if (options.cache && this.cache.has(url)) {
            return this.cache.get(url);
        }

        try {
            const response = await fetch(url, {
                headers: {
                    "Content-Type": "application/json",
                    ...options.headers
                },
                ...options
            });

            if (!response.ok) {
                throw new Error(`HTTP ${response.status}: ${response.statusText}`);
            }

            const data = await response.json();

            // Store in cache
            if (options.cache) {
                this.cache.set(url, data);
            }

            return data;

        } catch (error) {
            console.error(`Failed to fetch ${url}:`, error);
            throw error;
        }
    }

    async get(endpoint, options) {
        return this.fetch(endpoint, { ...options, method: "GET" });
    }

    async post(endpoint, data, options) {
        return this.fetch(endpoint, {
            ...options,
            method: "POST",
            body: JSON.stringify(data)
        });
    }

    async put(endpoint, data, options) {
        return this.fetch(endpoint, {
            ...options,
            method: "PUT",
            body: JSON.stringify(data)
        });
    }

    async delete(endpoint, options) {
        return this.fetch(endpoint, { ...options, method: "DELETE" });
    }

    clearCache() {
        this.cache.clear();
    }
}

// Usage
const api = new DataService("https://api.example.com");

async function loadUserData(userId) {
    try {
        const user = await api.get(`/users/${userId}`, { cache: true });
        const posts = await api.get(`/users/${userId}/posts`);
        return { user, posts };
    } catch (error) {
        console.error("Failed to load user data:", error);
        return null;
    }
}
```

### Example 2: Promise Queue

```javascript
class PromiseQueue {
    constructor(concurrency = 1) {
        this.concurrency = concurrency;
        this.queue = [];
        this.running = 0;
    }

    add(promiseFactory) {
        return new Promise((resolve, reject) => {
            this.queue.push({
                promiseFactory,
                resolve,
                reject
            });
            this.runNext();
        });
    }

    runNext() {
        if (this.running >= this.concurrency) return;
        if (this.queue.length === 0) return;

        const { promiseFactory, resolve, reject } = this.queue.shift();
        this.running++;

        promiseFactory()
            .then(resolve)
            .catch(reject)
            .finally(() => {
                this.running--;
                this.runNext();
            });
    }
}

// Usage - process 2 at a time
const queue = new PromiseQueue(2);

const urls = ["/api/1", "/api/2", "/api/3", "/api/4", "/api/5"];

urls.forEach(url => {
    queue.add(() => fetch(url).then(r => r.json()))
        .then(data => console.log(`${url}:`, data))
        .catch(error => console.error(`${url} failed:`, error));
});
```

### Example 3: Debounced Auto-Save

```javascript
class AutoSave {
    constructor(saveFunction, delay = 1000) {
        this.save = saveFunction;
        this.delay = delay;
        this.timeoutId = null;
        this.pendingData = null;
        this.isSaving = false;
    }

    async trigger(data) {
        this.pendingData = data;

        // Clear existing timeout
        if (this.timeoutId) {
            clearTimeout(this.timeoutId);
        }

        // Don't queue if already saving
        if (this.isSaving) {
            return;
        }

        // Set new timeout
        this.timeoutId = setTimeout(() => this.executeSave(), this.delay);
    }

    async executeSave() {
        if (this.pendingData === null) return;

        this.isSaving = true;
        const dataToSave = this.pendingData;
        this.pendingData = null;

        try {
            await this.save(dataToSave);
            console.log("Saved successfully");
        } catch (error) {
            console.error("Save failed:", error);
            // Re-queue failed save
            this.pendingData = dataToSave;
        } finally {
            this.isSaving = false;

            // Check if new data arrived during save
            if (this.pendingData !== null) {
                this.timeoutId = setTimeout(() => this.executeSave(), this.delay);
            }
        }
    }

    flush() {
        if (this.timeoutId) {
            clearTimeout(this.timeoutId);
            this.timeoutId = null;
        }
        return this.executeSave();
    }
}

// Usage
const autoSave = new AutoSave(async (data) => {
    await fetch("/api/save", {
        method: "POST",
        body: JSON.stringify(data)
    });
});

// In text editor
textArea.addEventListener("input", (e) => {
    autoSave.trigger({ content: e.target.value });
});

// Save before leaving
window.addEventListener("beforeunload", () => {
    autoSave.flush();
});
```

### Example 4: Polling Service

```javascript
class PollingService {
    constructor(fetchFn, interval = 5000) {
        this.fetchFn = fetchFn;
        this.interval = interval;
        this.isRunning = false;
        this.callbacks = [];
    }

    onUpdate(callback) {
        this.callbacks.push(callback);
        return () => {
            this.callbacks = this.callbacks.filter(cb => cb !== callback);
        };
    }

    notify(data) {
        this.callbacks.forEach(cb => cb(data));
    }

    async poll() {
        if (!this.isRunning) return;

        try {
            const data = await this.fetchFn();
            this.notify({ status: "success", data });
        } catch (error) {
            this.notify({ status: "error", error });
        }

        if (this.isRunning) {
            this.timeoutId = setTimeout(() => this.poll(), this.interval);
        }
    }

    start() {
        if (this.isRunning) return;
        this.isRunning = true;
        this.poll();
    }

    stop() {
        this.isRunning = false;
        if (this.timeoutId) {
            clearTimeout(this.timeoutId);
        }
    }

    setInterval(interval) {
        this.interval = interval;
    }
}

// Usage
const statusPoller = new PollingService(
    () => fetch("/api/status").then(r => r.json()),
    3000
);

const unsubscribe = statusPoller.onUpdate(({ status, data, error }) => {
    if (status === "success") {
        updateStatusDisplay(data);
    } else {
        showError(error);
    }
});

statusPoller.start();

// Later
statusPoller.stop();
unsubscribe();
```

---

## Practice Exercises

### Exercise 1: Implement Promise.all

```javascript
// Implement your own Promise.all

function myPromiseAll(promises) {
    // Your code here
}

// Test
myPromiseAll([
    Promise.resolve(1),
    Promise.resolve(2),
    Promise.resolve(3)
]).then(console.log);  // [1, 2, 3]
```

**Solution:**
```javascript
function myPromiseAll(promises) {
    return new Promise((resolve, reject) => {
        const results = [];
        let completed = 0;

        if (promises.length === 0) {
            resolve([]);
            return;
        }

        promises.forEach((promise, index) => {
            Promise.resolve(promise)
                .then(value => {
                    results[index] = value;
                    completed++;

                    if (completed === promises.length) {
                        resolve(results);
                    }
                })
                .catch(reject);
        });
    });
}
```

### Exercise 2: Implement delay with cancel

```javascript
// Create a delay function that can be cancelled

function cancellableDelay(ms) {
    // Your code here
    // Return { promise, cancel }
}

// Test
const { promise, cancel } = cancellableDelay(5000);
promise.then(() => console.log("Done")).catch(() => console.log("Cancelled"));
cancel();  // Should log "Cancelled"
```

**Solution:**
```javascript
function cancellableDelay(ms) {
    let timeoutId;
    let rejectFn;

    const promise = new Promise((resolve, reject) => {
        rejectFn = reject;
        timeoutId = setTimeout(resolve, ms);
    });

    const cancel = () => {
        clearTimeout(timeoutId);
        rejectFn(new Error("Cancelled"));
    };

    return { promise, cancel };
}
```

### Exercise 3: Sequential async map

```javascript
// Process array items one at a time asynchronously

async function asyncMapSequential(array, asyncFn) {
    // Your code here
}

// Test
asyncMapSequential([1, 2, 3], async (n) => {
    await delay(100);
    return n * 2;
}).then(console.log);  // [2, 4, 6]
```

**Solution:**
```javascript
async function asyncMapSequential(array, asyncFn) {
    const results = [];
    for (const item of array) {
        results.push(await asyncFn(item));
    }
    return results;
}
```

---

## Key Takeaways

1. **JavaScript is single-threaded** but handles async via event loop
2. **Callbacks** were the original pattern but lead to "callback hell"
3. **Promises** provide cleaner async handling with `.then()` and `.catch()`
4. **async/await** makes async code look synchronous
5. **Promise.all** for parallel, loop for sequential
6. **try/catch** for error handling in async functions
7. **Microtasks** (Promises) run before macrotasks (setTimeout)
8. **Always handle errors** in async code

---

## Self-Check Questions

1. What are the three states of a Promise?
2. What's the difference between `Promise.all` and `Promise.allSettled`?
3. How do you handle errors with async/await?
4. Why does `setTimeout(fn, 0)` not run immediately?
5. How do you run promises in parallel?
6. What's the difference between microtasks and macrotasks?
7. How do you cancel a fetch request?

---

**Next Lesson:** [Day 5 - Fetch API](./week-03-day-05-fetch-api.md)
