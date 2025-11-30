# Day 1: Introduction to Node.js

## Introduction

Node.js is a JavaScript runtime built on Chrome's V8 engine that allows you to run JavaScript on the server. It's the foundation for modern backend development with JavaScript, enabling full-stack development with a single language.

## Learning Objectives

By the end of this lesson, you will:
- Understand what Node.js is and how it works
- Set up a Node.js development environment
- Understand the event loop and non-blocking I/O
- Use built-in Node.js modules
- Work with the file system
- Create your first Node.js server

---

## What is Node.js?

### Definition

Node.js is:
- A **JavaScript runtime** (not a framework or language)
- Built on **Chrome's V8 JavaScript engine**
- Designed for building **scalable network applications**
- Uses an **event-driven, non-blocking I/O** model

### Why Node.js?

```
Traditional Server (Thread-based)        Node.js (Event-driven)
┌─────────────────────────────┐         ┌─────────────────────────────┐
│  Request 1 → Thread 1       │         │  Request 1 ─┐               │
│  Request 2 → Thread 2       │         │  Request 2 ─┤               │
│  Request 3 → Thread 3       │         │  Request 3 ─┼→ Event Loop   │
│  Request 4 → Waiting...     │         │  Request 4 ─┤               │
│  (Limited by thread pool)   │         │  Request 5 ─┘               │
└─────────────────────────────┘         └─────────────────────────────┘
```

**Advantages:**
1. **Same language everywhere** - JavaScript on frontend and backend
2. **Non-blocking I/O** - Handle many connections simultaneously
3. **NPM ecosystem** - Largest package registry
4. **Fast execution** - V8 engine is highly optimized
5. **Great for real-time** - WebSockets, streaming, chat apps

**Use Cases:**
- REST APIs and microservices
- Real-time applications (chat, games)
- Streaming services
- Server-side rendering
- Build tools and CLI applications

---

## Setting Up Node.js

### Installation

```bash
# Check if Node.js is installed
node --version
npm --version

# Using nvm (recommended)
# macOS/Linux
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

# Then install Node.js
nvm install node        # Latest version
nvm install 20          # Specific version (LTS)
nvm use 20              # Use specific version
nvm alias default 20    # Set default version

# Verify installation
node --version   # Should show v20.x.x
npm --version    # Should show 10.x.x
```

### Your First Node.js Program

```javascript
// hello.js
console.log('Hello, Node.js!');

// Run it
// $ node hello.js
// Output: Hello, Node.js!
```

### Node REPL (Read-Eval-Print Loop)

```bash
$ node
> 2 + 2
4
> const greeting = "Hello"
undefined
> greeting
'Hello'
> .exit
```

---

## The Event Loop

### Understanding Non-Blocking I/O

```javascript
// Blocking (Synchronous)
const fs = require('fs');

console.log('Start');
const data = fs.readFileSync('file.txt', 'utf8'); // Blocks here
console.log(data);
console.log('End');

// Output:
// Start
// [file contents]
// End
```

```javascript
// Non-Blocking (Asynchronous)
const fs = require('fs');

console.log('Start');
fs.readFile('file.txt', 'utf8', (err, data) => {
  console.log(data);
});
console.log('End');

// Output:
// Start
// End
// [file contents]
```

### Event Loop Phases

```
   ┌───────────────────────────┐
┌─>│           timers          │  <- setTimeout, setInterval
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │     pending callbacks     │  <- I/O callbacks
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │       idle, prepare       │  <- internal use
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │           poll            │  <- incoming connections, data
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │           check           │  <- setImmediate
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
└──┤      close callbacks      │  <- socket.on('close')
   └───────────────────────────┘
```

### Callback Pattern

```javascript
// Traditional callback pattern
function readFileAsync(path, callback) {
  fs.readFile(path, 'utf8', (err, data) => {
    if (err) {
      callback(err, null);
      return;
    }
    callback(null, data);
  });
}

// Error-first callback convention
readFileAsync('file.txt', (err, data) => {
  if (err) {
    console.error('Error:', err.message);
    return;
  }
  console.log('Data:', data);
});
```

### Promises and Async/Await

```javascript
// Using promises
const fs = require('fs').promises;

fs.readFile('file.txt', 'utf8')
  .then(data => console.log(data))
  .catch(err => console.error(err));

// Using async/await (recommended)
async function readFileAsync() {
  try {
    const data = await fs.readFile('file.txt', 'utf8');
    console.log(data);
  } catch (err) {
    console.error('Error:', err.message);
  }
}

readFileAsync();
```

---

## Global Objects

### Available Globally

```javascript
// __dirname - Directory of current file
console.log(__dirname); // /Users/user/project

// __filename - Full path to current file
console.log(__filename); // /Users/user/project/app.js

// process - Information about current process
console.log(process.cwd());        // Current working directory
console.log(process.env.NODE_ENV); // Environment variables
console.log(process.argv);         // Command line arguments
console.log(process.pid);          // Process ID
console.log(process.platform);     // Operating system

// global - Global namespace (like window in browser)
global.myVar = 'Hello';
console.log(myVar); // Hello

// console - Logging
console.log('Info');
console.error('Error');
console.warn('Warning');
console.time('timer');
console.timeEnd('timer');

// setTimeout, setInterval, setImmediate
setTimeout(() => console.log('After 1 second'), 1000);
setInterval(() => console.log('Every 2 seconds'), 2000);
setImmediate(() => console.log('Immediate'));

// Buffer - Binary data handling
const buf = Buffer.from('Hello');
console.log(buf); // <Buffer 48 65 6c 6c 6f>
console.log(buf.toString()); // Hello
```

### Process Object Deep Dive

```javascript
// Command line arguments
// node app.js arg1 arg2
console.log(process.argv);
// ['/usr/bin/node', '/path/to/app.js', 'arg1', 'arg2']

// Environment variables
process.env.MY_VAR = 'value';
console.log(process.env.MY_VAR); // value

// Exit the process
process.exit(0);  // Success
process.exit(1);  // Error

// Handle uncaught exceptions
process.on('uncaughtException', (err) => {
  console.error('Uncaught Exception:', err);
  process.exit(1);
});

// Handle unhandled promise rejections
process.on('unhandledRejection', (reason, promise) => {
  console.error('Unhandled Rejection:', reason);
});

// Memory usage
console.log(process.memoryUsage());
// {
//   rss: 35white651584,
//   heapTotal: 7012352,
//   heapUsed: 5234872,
//   external: 8272
// }
```

---

## Built-in Modules

### Path Module

```javascript
const path = require('path');

// Join paths
const fullPath = path.join(__dirname, 'folder', 'file.txt');
console.log(fullPath); // /Users/user/project/folder/file.txt

// Resolve to absolute path
const absolute = path.resolve('folder', 'file.txt');
console.log(absolute); // /Users/user/project/folder/file.txt

// Get file extension
console.log(path.extname('file.txt'));  // .txt
console.log(path.extname('file.tar.gz')); // .gz

// Get filename
console.log(path.basename('/path/to/file.txt'));      // file.txt
console.log(path.basename('/path/to/file.txt', '.txt')); // file

// Get directory name
console.log(path.dirname('/path/to/file.txt')); // /path/to

// Parse path
const parsed = path.parse('/path/to/file.txt');
console.log(parsed);
// {
//   root: '/',
//   dir: '/path/to',
//   base: 'file.txt',
//   ext: '.txt',
//   name: 'file'
// }

// Format path
const formatted = path.format({
  dir: '/path/to',
  name: 'file',
  ext: '.txt'
});
console.log(formatted); // /path/to/file.txt
```

### File System Module

```javascript
const fs = require('fs');
const fsPromises = require('fs').promises;

// --- Synchronous Operations ---
// Read file
const data = fs.readFileSync('file.txt', 'utf8');

// Write file
fs.writeFileSync('file.txt', 'Hello, World!');

// Append to file
fs.appendFileSync('file.txt', '\nMore content');

// Check if exists
const exists = fs.existsSync('file.txt');

// --- Asynchronous with Callbacks ---
fs.readFile('file.txt', 'utf8', (err, data) => {
  if (err) throw err;
  console.log(data);
});

// --- Asynchronous with Promises (Recommended) ---
async function fileOperations() {
  // Read
  const content = await fsPromises.readFile('file.txt', 'utf8');
  console.log(content);

  // Write
  await fsPromises.writeFile('output.txt', 'Hello!');

  // Append
  await fsPromises.appendFile('output.txt', '\nWorld!');

  // Copy
  await fsPromises.copyFile('source.txt', 'dest.txt');

  // Rename/Move
  await fsPromises.rename('old.txt', 'new.txt');

  // Delete
  await fsPromises.unlink('file.txt');

  // Create directory
  await fsPromises.mkdir('new-folder', { recursive: true });

  // Read directory
  const files = await fsPromises.readdir('.');
  console.log(files);

  // Get file stats
  const stats = await fsPromises.stat('file.txt');
  console.log(stats.isFile());      // true
  console.log(stats.isDirectory()); // false
  console.log(stats.size);          // file size in bytes

  // Remove directory
  await fsPromises.rmdir('folder', { recursive: true });
}
```

### OS Module

```javascript
const os = require('os');

// System info
console.log(os.platform());  // darwin, linux, win32
console.log(os.arch());      // x64, arm64
console.log(os.cpus());      // CPU information
console.log(os.totalmem());  // Total memory in bytes
console.log(os.freemem());   // Free memory in bytes
console.log(os.homedir());   // User's home directory
console.log(os.tmpdir());    // Temp directory
console.log(os.hostname());  // Computer name
console.log(os.uptime());    // System uptime in seconds

// Network interfaces
console.log(os.networkInterfaces());

// User info
console.log(os.userInfo());
```

### URL Module

```javascript
const { URL, URLSearchParams } = require('url');

// Parse URL
const myURL = new URL('https://example.com:8080/path?query=value#hash');

console.log(myURL.protocol);  // https:
console.log(myURL.hostname);  // example.com
console.log(myURL.port);      // 8080
console.log(myURL.pathname);  // /path
console.log(myURL.search);    // ?query=value
console.log(myURL.hash);      // #hash
console.log(myURL.origin);    // https://example.com:8080

// Modify URL
myURL.pathname = '/new-path';
console.log(myURL.href); // https://example.com:8080/new-path?query=value#hash

// Work with query params
const params = new URLSearchParams('?name=John&age=30');
console.log(params.get('name'));     // John
console.log(params.has('age'));      // true
params.append('city', 'NYC');
params.delete('age');
console.log(params.toString());      // name=John&city=NYC

// Iterate over params
for (const [key, value] of params) {
  console.log(`${key}: ${value}`);
}
```

---

## Creating an HTTP Server

### Basic HTTP Server

```javascript
const http = require('http');

const server = http.createServer((req, res) => {
  // req = request object (what the client sent)
  // res = response object (what we send back)

  console.log(`${req.method} ${req.url}`);

  // Set response headers
  res.writeHead(200, { 'Content-Type': 'text/plain' });

  // Send response body
  res.end('Hello, World!');
});

const PORT = 3000;
server.listen(PORT, () => {
  console.log(`Server running at http://localhost:${PORT}/`);
});
```

### Handling Different Routes

```javascript
const http = require('http');

const server = http.createServer((req, res) => {
  const { method, url } = req;

  // Set default content type
  res.setHeader('Content-Type', 'application/json');

  // Route handling
  if (method === 'GET' && url === '/') {
    res.writeHead(200);
    res.end(JSON.stringify({ message: 'Welcome to the API' }));
  } else if (method === 'GET' && url === '/users') {
    res.writeHead(200);
    res.end(JSON.stringify([
      { id: 1, name: 'John' },
      { id: 2, name: 'Jane' }
    ]));
  } else if (method === 'GET' && url.startsWith('/users/')) {
    const id = url.split('/')[2];
    res.writeHead(200);
    res.end(JSON.stringify({ id, name: `User ${id}` }));
  } else {
    res.writeHead(404);
    res.end(JSON.stringify({ error: 'Not found' }));
  }
});

server.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

### Handling POST Requests

```javascript
const http = require('http');

const server = http.createServer((req, res) => {
  if (req.method === 'POST' && req.url === '/users') {
    let body = '';

    // Collect data chunks
    req.on('data', chunk => {
      body += chunk.toString();
    });

    // All data received
    req.on('end', () => {
      try {
        const user = JSON.parse(body);
        console.log('Received user:', user);

        res.writeHead(201, { 'Content-Type': 'application/json' });
        res.end(JSON.stringify({ id: Date.now(), ...user }));
      } catch (error) {
        res.writeHead(400, { 'Content-Type': 'application/json' });
        res.end(JSON.stringify({ error: 'Invalid JSON' }));
      }
    });
  } else {
    res.writeHead(404);
    res.end('Not found');
  }
});

server.listen(3000);
```

### Serving Static Files

```javascript
const http = require('http');
const fs = require('fs');
const path = require('path');

const MIME_TYPES = {
  '.html': 'text/html',
  '.css': 'text/css',
  '.js': 'text/javascript',
  '.json': 'application/json',
  '.png': 'image/png',
  '.jpg': 'image/jpeg',
  '.gif': 'image/gif',
  '.svg': 'image/svg+xml',
};

const server = http.createServer((req, res) => {
  // Get the file path
  let filePath = path.join(__dirname, 'public', req.url);

  // Default to index.html
  if (req.url === '/') {
    filePath = path.join(__dirname, 'public', 'index.html');
  }

  // Get file extension
  const ext = path.extname(filePath);
  const contentType = MIME_TYPES[ext] || 'application/octet-stream';

  // Read and serve the file
  fs.readFile(filePath, (err, content) => {
    if (err) {
      if (err.code === 'ENOENT') {
        res.writeHead(404);
        res.end('File not found');
      } else {
        res.writeHead(500);
        res.end('Server error');
      }
      return;
    }

    res.writeHead(200, { 'Content-Type': contentType });
    res.end(content);
  });
});

server.listen(3000, () => {
  console.log('Static file server on port 3000');
});
```

---

## Events Module

### EventEmitter

```javascript
const EventEmitter = require('events');

// Create an emitter
const emitter = new EventEmitter();

// Register a listener
emitter.on('userCreated', (user) => {
  console.log('User created:', user);
});

// Register a one-time listener
emitter.once('serverStarted', () => {
  console.log('Server started (this runs once)');
});

// Emit an event
emitter.emit('userCreated', { id: 1, name: 'John' });
emitter.emit('serverStarted');
emitter.emit('serverStarted'); // Won't trigger (once)

// Remove a listener
const callback = () => console.log('Callback');
emitter.on('test', callback);
emitter.removeListener('test', callback);

// Get listener count
console.log(emitter.listenerCount('userCreated')); // 1
```

### Custom Event-Based Class

```javascript
const EventEmitter = require('events');

class UserService extends EventEmitter {
  constructor() {
    super();
    this.users = [];
  }

  createUser(userData) {
    const user = {
      id: Date.now(),
      ...userData,
      createdAt: new Date()
    };

    this.users.push(user);
    this.emit('user:created', user);
    return user;
  }

  deleteUser(id) {
    const index = this.users.findIndex(u => u.id === id);
    if (index !== -1) {
      const [user] = this.users.splice(index, 1);
      this.emit('user:deleted', user);
      return user;
    }
    this.emit('user:error', new Error('User not found'));
    return null;
  }
}

// Usage
const userService = new UserService();

userService.on('user:created', (user) => {
  console.log('New user:', user);
  // Send welcome email, log to analytics, etc.
});

userService.on('user:deleted', (user) => {
  console.log('User deleted:', user);
});

userService.on('user:error', (error) => {
  console.error('Error:', error.message);
});

userService.createUser({ name: 'John', email: 'john@example.com' });
```

---

## Streams

### Types of Streams

```javascript
const fs = require('fs');

// Readable Stream - Reading data
const readStream = fs.createReadStream('large-file.txt', {
  encoding: 'utf8',
  highWaterMark: 64 * 1024 // 64KB chunks
});

readStream.on('data', (chunk) => {
  console.log(`Received ${chunk.length} bytes`);
});

readStream.on('end', () => {
  console.log('Finished reading');
});

readStream.on('error', (err) => {
  console.error('Error:', err);
});

// Writable Stream - Writing data
const writeStream = fs.createWriteStream('output.txt');

writeStream.write('Hello ');
writeStream.write('World!');
writeStream.end(); // Signal no more data

writeStream.on('finish', () => {
  console.log('Finished writing');
});
```

### Piping Streams

```javascript
const fs = require('fs');
const zlib = require('zlib');

// Simple pipe - copy file
fs.createReadStream('input.txt')
  .pipe(fs.createWriteStream('output.txt'));

// Pipe through transform - compress file
fs.createReadStream('input.txt')
  .pipe(zlib.createGzip())
  .pipe(fs.createWriteStream('input.txt.gz'));

// Decompress
fs.createReadStream('input.txt.gz')
  .pipe(zlib.createGunzip())
  .pipe(fs.createWriteStream('decompressed.txt'));

// HTTP response streaming
const http = require('http');

http.createServer((req, res) => {
  const readStream = fs.createReadStream('large-video.mp4');
  readStream.pipe(res);
}).listen(3000);
```

---

## Exercise: Build a Simple File Server

Create a Node.js server that:
1. Serves static files from a `public` directory
2. Has an API endpoint to list files in a directory
3. Has an API endpoint to read a specific file
4. Handles errors gracefully

---

## Solution

```javascript
// server.js
const http = require('http');
const fs = require('fs').promises;
const path = require('path');

const PORT = 3000;
const PUBLIC_DIR = path.join(__dirname, 'public');

const MIME_TYPES = {
  '.html': 'text/html',
  '.css': 'text/css',
  '.js': 'text/javascript',
  '.json': 'application/json',
  '.png': 'image/png',
  '.jpg': 'image/jpeg',
  '.txt': 'text/plain',
};

async function handleStaticFile(req, res) {
  let filePath = path.join(PUBLIC_DIR, req.url);

  if (req.url === '/') {
    filePath = path.join(PUBLIC_DIR, 'index.html');
  }

  try {
    const content = await fs.readFile(filePath);
    const ext = path.extname(filePath);
    const contentType = MIME_TYPES[ext] || 'application/octet-stream';

    res.writeHead(200, { 'Content-Type': contentType });
    res.end(content);
  } catch (err) {
    if (err.code === 'ENOENT') {
      return false; // File not found, try other routes
    }
    throw err;
  }
  return true;
}

async function handleApiListFiles(req, res) {
  try {
    const files = await fs.readdir(PUBLIC_DIR);
    const fileDetails = await Promise.all(
      files.map(async (file) => {
        const stats = await fs.stat(path.join(PUBLIC_DIR, file));
        return {
          name: file,
          size: stats.size,
          isDirectory: stats.isDirectory(),
          modified: stats.mtime,
        };
      })
    );

    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify(fileDetails, null, 2));
  } catch (err) {
    res.writeHead(500, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({ error: 'Failed to list files' }));
  }
}

async function handleApiReadFile(req, res, filename) {
  const filePath = path.join(PUBLIC_DIR, filename);

  // Security: Prevent directory traversal
  if (!filePath.startsWith(PUBLIC_DIR)) {
    res.writeHead(403, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({ error: 'Access denied' }));
    return;
  }

  try {
    const content = await fs.readFile(filePath, 'utf8');
    const stats = await fs.stat(filePath);

    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({
      name: filename,
      size: stats.size,
      content: content,
    }));
  } catch (err) {
    if (err.code === 'ENOENT') {
      res.writeHead(404, { 'Content-Type': 'application/json' });
      res.end(JSON.stringify({ error: 'File not found' }));
    } else {
      res.writeHead(500, { 'Content-Type': 'application/json' });
      res.end(JSON.stringify({ error: 'Failed to read file' }));
    }
  }
}

const server = http.createServer(async (req, res) => {
  const { method, url } = req;

  console.log(`${method} ${url}`);

  try {
    // API Routes
    if (url === '/api/files' && method === 'GET') {
      await handleApiListFiles(req, res);
      return;
    }

    if (url.startsWith('/api/files/') && method === 'GET') {
      const filename = url.replace('/api/files/', '');
      await handleApiReadFile(req, res, filename);
      return;
    }

    // Static files
    const handled = await handleStaticFile(req, res);
    if (handled) return;

    // 404 for everything else
    res.writeHead(404, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({ error: 'Not found' }));
  } catch (err) {
    console.error('Server error:', err);
    res.writeHead(500, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({ error: 'Internal server error' }));
  }
});

server.listen(PORT, () => {
  console.log(`Server running at http://localhost:${PORT}/`);
  console.log(`API endpoints:`);
  console.log(`  GET /api/files - List all files`);
  console.log(`  GET /api/files/:filename - Read a specific file`);
});
```

Create a test file structure:

```bash
mkdir public
echo "<h1>Hello!</h1>" > public/index.html
echo "Test content" > public/test.txt
echo "body { color: blue; }" > public/styles.css
```

Test the server:

```bash
node server.js

# In another terminal:
curl http://localhost:3000/
curl http://localhost:3000/api/files
curl http://localhost:3000/api/files/test.txt
```

---

## Key Takeaways

1. **Node.js is a runtime** - It runs JavaScript outside the browser
2. **Event-driven, non-blocking** - Perfect for I/O-heavy operations
3. **Single-threaded but concurrent** - Event loop handles multiple operations
4. **Built-in modules** - fs, path, http, events, os, url
5. **Prefer async/await** - Over callbacks for cleaner code
6. **Streams for large files** - Don't load everything into memory
7. **Error handling is critical** - Always handle errors in callbacks and promises

---

## What's Next?

Tomorrow we'll learn about **Node.js Modules and NPM** - understanding CommonJS, ES modules, and how to use the world's largest package ecosystem!
