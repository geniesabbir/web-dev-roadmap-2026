# Week 3, Day 5: Fetch API

## Introduction to Fetch

The Fetch API provides a modern way to make HTTP requests. It returns Promises and is much cleaner than the older XMLHttpRequest.

```javascript
// Basic GET request
fetch("https://api.example.com/data")
    .then(response => response.json())
    .then(data => console.log(data))
    .catch(error => console.error("Error:", error));

// With async/await
async function getData() {
    try {
        const response = await fetch("https://api.example.com/data");
        const data = await response.json();
        console.log(data);
    } catch (error) {
        console.error("Error:", error);
    }
}
```

---

## The Response Object

Fetch returns a Response object with useful properties and methods:

```javascript
const response = await fetch("https://api.example.com/data");

// Response properties
console.log(response.status);      // 200
console.log(response.statusText);  // "OK"
console.log(response.ok);          // true (status 200-299)
console.log(response.headers);     // Headers object
console.log(response.url);         // Final URL (after redirects)
console.log(response.redirected);  // true if redirected
console.log(response.type);        // "basic", "cors", "opaque"

// Response body methods (can only be read once!)
const json = await response.json();     // Parse as JSON
const text = await response.text();     // Parse as text
const blob = await response.blob();     // Parse as Blob
const buffer = await response.arrayBuffer();  // Parse as ArrayBuffer
const formData = await response.formData();   // Parse as FormData

// Clone response to read body multiple times
const clone = response.clone();
const text1 = await response.text();
const text2 = await clone.text();
```

### Checking Response Status

```javascript
async function fetchData(url) {
    const response = await fetch(url);

    // fetch doesn't reject on HTTP errors!
    if (!response.ok) {
        throw new Error(`HTTP error! Status: ${response.status}`);
    }

    return response.json();
}

// Common status codes
// 200 - OK
// 201 - Created
// 204 - No Content
// 400 - Bad Request
// 401 - Unauthorized
// 403 - Forbidden
// 404 - Not Found
// 500 - Internal Server Error
```

### Working with Headers

```javascript
const response = await fetch(url);

// Get specific header
console.log(response.headers.get("content-type"));
console.log(response.headers.get("x-custom-header"));

// Check if header exists
console.log(response.headers.has("content-type"));

// Iterate headers
for (const [key, value] of response.headers) {
    console.log(`${key}: ${value}`);
}

// Or using forEach
response.headers.forEach((value, key) => {
    console.log(`${key}: ${value}`);
});
```

---

## Making Different Request Types

### GET Request

```javascript
// Simple GET
const response = await fetch("https://api.example.com/users");
const users = await response.json();

// GET with query parameters
const params = new URLSearchParams({
    page: 1,
    limit: 10,
    sort: "name"
});

const response = await fetch(`https://api.example.com/users?${params}`);

// Alternative: build URL
const url = new URL("https://api.example.com/users");
url.searchParams.append("page", 1);
url.searchParams.append("limit", 10);

const response = await fetch(url);
```

### POST Request

```javascript
// POST with JSON body
const response = await fetch("https://api.example.com/users", {
    method: "POST",
    headers: {
        "Content-Type": "application/json"
    },
    body: JSON.stringify({
        name: "John Doe",
        email: "john@example.com"
    })
});

const newUser = await response.json();

// POST with FormData
const formData = new FormData();
formData.append("name", "John Doe");
formData.append("email", "john@example.com");
formData.append("avatar", fileInput.files[0]);

const response = await fetch("https://api.example.com/users", {
    method: "POST",
    body: formData
    // Don't set Content-Type header - browser sets it automatically with boundary
});
```

### PUT Request

```javascript
// Full update
const response = await fetch("https://api.example.com/users/123", {
    method: "PUT",
    headers: {
        "Content-Type": "application/json"
    },
    body: JSON.stringify({
        name: "John Updated",
        email: "john.updated@example.com"
    })
});
```

### PATCH Request

```javascript
// Partial update
const response = await fetch("https://api.example.com/users/123", {
    method: "PATCH",
    headers: {
        "Content-Type": "application/json"
    },
    body: JSON.stringify({
        name: "John Updated"  // Only update name
    })
});
```

### DELETE Request

```javascript
const response = await fetch("https://api.example.com/users/123", {
    method: "DELETE"
});

if (response.ok) {
    console.log("User deleted");
}

// DELETE with body (some APIs require it)
const response = await fetch("https://api.example.com/items", {
    method: "DELETE",
    headers: {
        "Content-Type": "application/json"
    },
    body: JSON.stringify({
        ids: [1, 2, 3]
    })
});
```

---

## Request Options

```javascript
const response = await fetch(url, {
    // HTTP method
    method: "POST",  // GET, POST, PUT, PATCH, DELETE, etc.

    // Headers
    headers: {
        "Content-Type": "application/json",
        "Authorization": "Bearer token123",
        "X-Custom-Header": "value"
    },

    // Request body
    body: JSON.stringify(data),

    // CORS mode
    mode: "cors",  // "cors", "no-cors", "same-origin"

    // Credentials (cookies)
    credentials: "include",  // "omit", "same-origin", "include"

    // Cache mode
    cache: "default",  // "default", "no-store", "reload", "no-cache", "force-cache"

    // Redirect handling
    redirect: "follow",  // "follow", "manual", "error"

    // Referrer policy
    referrerPolicy: "no-referrer",

    // AbortController signal (for cancellation)
    signal: controller.signal
});
```

### Setting Headers

```javascript
// Using Headers object
const headers = new Headers();
headers.append("Content-Type", "application/json");
headers.append("Authorization", "Bearer token123");

const response = await fetch(url, {
    method: "POST",
    headers: headers,
    body: JSON.stringify(data)
});

// Using plain object (more common)
const response = await fetch(url, {
    headers: {
        "Content-Type": "application/json",
        "Authorization": "Bearer token123"
    }
});

// Adding headers to existing
const response = await fetch(url, {
    headers: new Headers({
        ...existingHeaders,
        "X-New-Header": "value"
    })
});
```

---

## Authentication

### Bearer Token

```javascript
const token = "your-jwt-token";

const response = await fetch("https://api.example.com/protected", {
    headers: {
        "Authorization": `Bearer ${token}`
    }
});
```

### Basic Authentication

```javascript
const username = "user";
const password = "pass";
const credentials = btoa(`${username}:${password}`);

const response = await fetch("https://api.example.com/protected", {
    headers: {
        "Authorization": `Basic ${credentials}`
    }
});
```

### Cookies

```javascript
// Include cookies in request
const response = await fetch("https://api.example.com/data", {
    credentials: "include"  // Send cookies even for cross-origin
});

// Same-origin only (default)
const response = await fetch("https://api.example.com/data", {
    credentials: "same-origin"
});

// Never send cookies
const response = await fetch("https://api.example.com/data", {
    credentials: "omit"
});
```

---

## Error Handling

```javascript
async function fetchWithErrorHandling(url, options = {}) {
    try {
        const response = await fetch(url, options);

        // Check for HTTP errors
        if (!response.ok) {
            // Try to get error message from response
            let errorMessage;
            try {
                const errorData = await response.json();
                errorMessage = errorData.message || response.statusText;
            } catch {
                errorMessage = response.statusText;
            }

            throw new Error(`HTTP ${response.status}: ${errorMessage}`);
        }

        return await response.json();

    } catch (error) {
        // Network errors (no internet, DNS failure, etc.)
        if (error.name === "TypeError") {
            throw new Error("Network error - please check your connection");
        }

        // Abort errors
        if (error.name === "AbortError") {
            throw new Error("Request was cancelled");
        }

        // Re-throw other errors
        throw error;
    }
}

// Usage
try {
    const data = await fetchWithErrorHandling("https://api.example.com/data");
    console.log(data);
} catch (error) {
    console.error("Failed:", error.message);
    showErrorToUser(error.message);
}
```

### Custom Error Classes

```javascript
class ApiError extends Error {
    constructor(message, status, data = null) {
        super(message);
        this.name = "ApiError";
        this.status = status;
        this.data = data;
    }
}

class NetworkError extends Error {
    constructor(message = "Network error") {
        super(message);
        this.name = "NetworkError";
    }
}

async function fetchApi(url, options = {}) {
    let response;

    try {
        response = await fetch(url, options);
    } catch (error) {
        throw new NetworkError("Unable to connect to server");
    }

    if (!response.ok) {
        let data;
        try {
            data = await response.json();
        } catch {
            data = null;
        }

        throw new ApiError(
            data?.message || `HTTP ${response.status}`,
            response.status,
            data
        );
    }

    return response.json();
}

// Usage
try {
    const data = await fetchApi("/api/users");
} catch (error) {
    if (error instanceof NetworkError) {
        showOfflineMessage();
    } else if (error instanceof ApiError) {
        if (error.status === 401) {
            redirectToLogin();
        } else if (error.status === 404) {
            showNotFoundPage();
        } else {
            showErrorMessage(error.message);
        }
    }
}
```

---

## Cancelling Requests

Using AbortController:

```javascript
// Create controller
const controller = new AbortController();
const signal = controller.signal;

// Start fetch with signal
const fetchPromise = fetch("https://api.example.com/data", { signal })
    .then(response => response.json())
    .then(data => console.log(data))
    .catch(error => {
        if (error.name === "AbortError") {
            console.log("Request was cancelled");
        } else {
            console.error("Error:", error);
        }
    });

// Cancel the request
controller.abort();

// With async/await
async function fetchWithCancel() {
    const controller = new AbortController();

    // Cancel after 5 seconds
    const timeoutId = setTimeout(() => controller.abort(), 5000);

    try {
        const response = await fetch(url, { signal: controller.signal });
        clearTimeout(timeoutId);
        return await response.json();
    } catch (error) {
        if (error.name === "AbortError") {
            throw new Error("Request timed out");
        }
        throw error;
    }
}

// Cancel on component unmount (React pattern)
useEffect(() => {
    const controller = new AbortController();

    fetch(url, { signal: controller.signal })
        .then(response => response.json())
        .then(setData)
        .catch(error => {
            if (error.name !== "AbortError") {
                setError(error);
            }
        });

    return () => controller.abort();
}, [url]);
```

### Timeout Helper

```javascript
function fetchWithTimeout(url, options = {}, timeout = 5000) {
    const controller = new AbortController();
    const signal = controller.signal;

    const timeoutId = setTimeout(() => {
        controller.abort();
    }, timeout);

    return fetch(url, { ...options, signal })
        .finally(() => clearTimeout(timeoutId));
}

// Usage
try {
    const response = await fetchWithTimeout(url, {}, 3000);
    const data = await response.json();
} catch (error) {
    if (error.name === "AbortError") {
        console.log("Request timed out");
    }
}
```

---

## File Upload

### Single File

```javascript
async function uploadFile(file) {
    const formData = new FormData();
    formData.append("file", file);

    const response = await fetch("/api/upload", {
        method: "POST",
        body: formData
    });

    return response.json();
}

// With file input
const fileInput = document.getElementById("fileInput");
fileInput.addEventListener("change", async () => {
    const file = fileInput.files[0];
    if (file) {
        const result = await uploadFile(file);
        console.log("Uploaded:", result);
    }
});
```

### Multiple Files

```javascript
async function uploadFiles(files) {
    const formData = new FormData();

    for (const file of files) {
        formData.append("files", file);
    }

    const response = await fetch("/api/upload-multiple", {
        method: "POST",
        body: formData
    });

    return response.json();
}
```

### With Progress (XMLHttpRequest)

```javascript
function uploadWithProgress(file, onProgress) {
    return new Promise((resolve, reject) => {
        const xhr = new XMLHttpRequest();
        const formData = new FormData();
        formData.append("file", file);

        xhr.upload.addEventListener("progress", (e) => {
            if (e.lengthComputable) {
                const percent = (e.loaded / e.total) * 100;
                onProgress(percent);
            }
        });

        xhr.addEventListener("load", () => {
            if (xhr.status >= 200 && xhr.status < 300) {
                resolve(JSON.parse(xhr.responseText));
            } else {
                reject(new Error(`Upload failed: ${xhr.status}`));
            }
        });

        xhr.addEventListener("error", () => reject(new Error("Network error")));

        xhr.open("POST", "/api/upload");
        xhr.send(formData);
    });
}

// Usage
uploadWithProgress(file, (percent) => {
    console.log(`Upload progress: ${percent.toFixed(1)}%`);
    progressBar.style.width = `${percent}%`;
}).then(result => {
    console.log("Upload complete:", result);
});
```

---

## File Download

```javascript
// Download as blob
async function downloadFile(url, filename) {
    const response = await fetch(url);
    const blob = await response.blob();

    // Create download link
    const downloadUrl = URL.createObjectURL(blob);
    const link = document.createElement("a");
    link.href = downloadUrl;
    link.download = filename;

    // Trigger download
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);

    // Clean up
    URL.revokeObjectURL(downloadUrl);
}

// Download with progress
async function downloadWithProgress(url, onProgress) {
    const response = await fetch(url);

    const contentLength = response.headers.get("content-length");
    const total = parseInt(contentLength, 10);

    const reader = response.body.getReader();
    const chunks = [];
    let received = 0;

    while (true) {
        const { done, value } = await reader.read();

        if (done) break;

        chunks.push(value);
        received += value.length;

        onProgress((received / total) * 100);
    }

    return new Blob(chunks);
}
```

---

## Practical Examples

### Example 1: API Client Class

```javascript
class ApiClient {
    constructor(baseUrl, defaultHeaders = {}) {
        this.baseUrl = baseUrl;
        this.defaultHeaders = {
            "Content-Type": "application/json",
            ...defaultHeaders
        };
    }

    setAuthToken(token) {
        this.defaultHeaders["Authorization"] = `Bearer ${token}`;
    }

    clearAuthToken() {
        delete this.defaultHeaders["Authorization"];
    }

    async request(endpoint, options = {}) {
        const url = `${this.baseUrl}${endpoint}`;

        const config = {
            ...options,
            headers: {
                ...this.defaultHeaders,
                ...options.headers
            }
        };

        if (config.body && typeof config.body === "object") {
            config.body = JSON.stringify(config.body);
        }

        const response = await fetch(url, config);

        if (!response.ok) {
            const error = await response.json().catch(() => ({}));
            throw new Error(error.message || `HTTP ${response.status}`);
        }

        // Handle empty responses
        const text = await response.text();
        return text ? JSON.parse(text) : null;
    }

    get(endpoint, options) {
        return this.request(endpoint, { ...options, method: "GET" });
    }

    post(endpoint, body, options) {
        return this.request(endpoint, { ...options, method: "POST", body });
    }

    put(endpoint, body, options) {
        return this.request(endpoint, { ...options, method: "PUT", body });
    }

    patch(endpoint, body, options) {
        return this.request(endpoint, { ...options, method: "PATCH", body });
    }

    delete(endpoint, options) {
        return this.request(endpoint, { ...options, method: "DELETE" });
    }
}

// Usage
const api = new ApiClient("https://api.example.com");
api.setAuthToken("your-token");

const users = await api.get("/users");
const newUser = await api.post("/users", { name: "John" });
await api.put("/users/123", { name: "Updated" });
await api.delete("/users/123");
```

### Example 2: Data Fetching Hook (React Pattern)

```javascript
function useFetch(url, options = {}) {
    const [data, setData] = useState(null);
    const [loading, setLoading] = useState(true);
    const [error, setError] = useState(null);

    useEffect(() => {
        const controller = new AbortController();

        async function fetchData() {
            setLoading(true);
            setError(null);

            try {
                const response = await fetch(url, {
                    ...options,
                    signal: controller.signal
                });

                if (!response.ok) {
                    throw new Error(`HTTP ${response.status}`);
                }

                const json = await response.json();
                setData(json);
            } catch (err) {
                if (err.name !== "AbortError") {
                    setError(err);
                }
            } finally {
                setLoading(false);
            }
        }

        fetchData();

        return () => controller.abort();
    }, [url]);

    return { data, loading, error };
}

// Usage in component
function UserList() {
    const { data: users, loading, error } = useFetch("/api/users");

    if (loading) return <div>Loading...</div>;
    if (error) return <div>Error: {error.message}</div>;

    return (
        <ul>
            {users.map(user => (
                <li key={user.id}>{user.name}</li>
            ))}
        </ul>
    );
}
```

### Example 3: Retry Logic

```javascript
async function fetchWithRetry(url, options = {}, retries = 3, delay = 1000) {
    let lastError;

    for (let attempt = 1; attempt <= retries; attempt++) {
        try {
            const response = await fetch(url, options);

            if (!response.ok) {
                throw new Error(`HTTP ${response.status}`);
            }

            return await response.json();

        } catch (error) {
            lastError = error;
            console.log(`Attempt ${attempt} failed:`, error.message);

            if (attempt < retries) {
                // Wait before retrying (exponential backoff)
                await new Promise(resolve =>
                    setTimeout(resolve, delay * Math.pow(2, attempt - 1))
                );
            }
        }
    }

    throw new Error(`Failed after ${retries} attempts: ${lastError.message}`);
}

// Usage
try {
    const data = await fetchWithRetry("/api/data", {}, 3, 1000);
    console.log("Success:", data);
} catch (error) {
    console.error("All retries failed:", error);
}
```

### Example 4: Request Queue

```javascript
class RequestQueue {
    constructor(maxConcurrent = 5) {
        this.maxConcurrent = maxConcurrent;
        this.pending = [];
        this.running = 0;
    }

    async add(requestFn) {
        return new Promise((resolve, reject) => {
            this.pending.push({ requestFn, resolve, reject });
            this.processQueue();
        });
    }

    async processQueue() {
        if (this.running >= this.maxConcurrent) return;
        if (this.pending.length === 0) return;

        const { requestFn, resolve, reject } = this.pending.shift();
        this.running++;

        try {
            const result = await requestFn();
            resolve(result);
        } catch (error) {
            reject(error);
        } finally {
            this.running--;
            this.processQueue();
        }
    }
}

// Usage
const queue = new RequestQueue(3);  // Max 3 concurrent requests

const urls = ["/api/1", "/api/2", "/api/3", "/api/4", "/api/5"];

const results = await Promise.all(
    urls.map(url =>
        queue.add(() => fetch(url).then(r => r.json()))
    )
);
```

---

## Practice Exercises

### Exercise 1: Create a User CRUD Service

```javascript
// Implement a UserService with full CRUD operations
class UserService {
    constructor(baseUrl) {
        this.baseUrl = baseUrl;
    }

    async getAll() { /* TODO */ }
    async getById(id) { /* TODO */ }
    async create(userData) { /* TODO */ }
    async update(id, userData) { /* TODO */ }
    async delete(id) { /* TODO */ }
}
```

**Solution:**
```javascript
class UserService {
    constructor(baseUrl) {
        this.baseUrl = baseUrl;
    }

    async getAll() {
        const response = await fetch(`${this.baseUrl}/users`);
        if (!response.ok) throw new Error(`HTTP ${response.status}`);
        return response.json();
    }

    async getById(id) {
        const response = await fetch(`${this.baseUrl}/users/${id}`);
        if (!response.ok) throw new Error(`HTTP ${response.status}`);
        return response.json();
    }

    async create(userData) {
        const response = await fetch(`${this.baseUrl}/users`, {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify(userData)
        });
        if (!response.ok) throw new Error(`HTTP ${response.status}`);
        return response.json();
    }

    async update(id, userData) {
        const response = await fetch(`${this.baseUrl}/users/${id}`, {
            method: "PUT",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify(userData)
        });
        if (!response.ok) throw new Error(`HTTP ${response.status}`);
        return response.json();
    }

    async delete(id) {
        const response = await fetch(`${this.baseUrl}/users/${id}`, {
            method: "DELETE"
        });
        if (!response.ok) throw new Error(`HTTP ${response.status}`);
        return true;
    }
}
```

### Exercise 2: Implement fetch with caching

```javascript
// Create a fetch wrapper that caches responses
function createCachedFetch(ttl = 60000) {
    // Your code here
}
```

**Solution:**
```javascript
function createCachedFetch(ttl = 60000) {
    const cache = new Map();

    return async function cachedFetch(url, options = {}) {
        const cacheKey = `${options.method || "GET"}:${url}`;

        // Check cache
        if (cache.has(cacheKey)) {
            const { data, timestamp } = cache.get(cacheKey);
            if (Date.now() - timestamp < ttl) {
                return data;
            }
            cache.delete(cacheKey);
        }

        // Fetch fresh data
        const response = await fetch(url, options);
        if (!response.ok) throw new Error(`HTTP ${response.status}`);

        const data = await response.json();

        // Cache the result
        cache.set(cacheKey, { data, timestamp: Date.now() });

        return data;
    };
}

const cachedFetch = createCachedFetch(30000);  // 30 second cache
const data = await cachedFetch("/api/data");
```

---

## Key Takeaways

1. **fetch returns a Promise** - use `.then()` or `async/await`
2. **fetch doesn't reject on HTTP errors** - check `response.ok`
3. **Response body can only be read once** - use `clone()` if needed
4. **Set Content-Type for JSON** - `"application/json"`
5. **Use AbortController** to cancel requests
6. **FormData for file uploads** - don't set Content-Type manually
7. **Always handle errors** - network and HTTP errors separately
8. **Use credentials: "include"** for cookies in cross-origin requests

---

## Self-Check Questions

1. How do you send JSON data in a POST request?
2. Why doesn't fetch reject on 404 or 500 errors?
3. How do you cancel a fetch request?
4. What's the difference between `response.json()` and `response.text()`?
5. How do you send cookies with cross-origin requests?
6. How do you handle file uploads with fetch?
7. What's the purpose of the `mode` option in fetch?

---

**Next Lesson:** [Week 4, Day 1-2 - ES6+ Features](./week-04-day-01-02-es6-features.md)
