# Day 3: Express.js Basics

## Introduction

Express.js is the most popular Node.js web framework. It provides a minimal, flexible foundation for building web applications and APIs. Understanding Express is essential for any Node.js backend developer.

## Learning Objectives

By the end of this lesson, you will:
- Set up an Express application
- Understand routing and HTTP methods
- Work with request and response objects
- Serve static files
- Use template engines
- Handle errors properly

---

## What is Express?

Express is a **minimal and flexible** Node.js web application framework that provides:

- **Routing** - Map URLs to handler functions
- **Middleware** - Process requests through a pipeline
- **Request/Response** - Enhanced objects with useful methods
- **Static files** - Serve CSS, images, JavaScript
- **Templates** - Render dynamic HTML

### Why Express?

```javascript
// Without Express (raw Node.js)
const http = require('http');

const server = http.createServer((req, res) => {
  if (req.method === 'GET' && req.url === '/users') {
    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify([{ id: 1, name: 'John' }]));
  } else if (req.method === 'POST' && req.url === '/users') {
    let body = '';
    req.on('data', chunk => body += chunk);
    req.on('end', () => {
      // Parse body, handle request...
    });
  }
  // Many more conditions...
});
```

```javascript
// With Express
const express = require('express');
const app = express();

app.use(express.json());

app.get('/users', (req, res) => {
  res.json([{ id: 1, name: 'John' }]);
});

app.post('/users', (req, res) => {
  const user = req.body; // Already parsed
  res.status(201).json(user);
});
```

---

## Setting Up Express

### Installation

```bash
mkdir my-api
cd my-api
npm init -y
npm install express
```

### Basic Server

```javascript
// app.js
const express = require('express');
const app = express();

const PORT = process.env.PORT || 3000;

// Basic route
app.get('/', (req, res) => {
  res.send('Hello, Express!');
});

// Start server
app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```

### Development Setup

```bash
npm install -D nodemon
```

```json
// package.json
{
  "scripts": {
    "start": "node app.js",
    "dev": "nodemon app.js"
  }
}
```

---

## Routing

### HTTP Methods

```javascript
const express = require('express');
const app = express();

// GET - Retrieve data
app.get('/users', (req, res) => {
  res.json([{ id: 1, name: 'John' }]);
});

// POST - Create data
app.post('/users', (req, res) => {
  res.status(201).json({ id: 2, name: 'Jane' });
});

// PUT - Update entire resource
app.put('/users/:id', (req, res) => {
  res.json({ id: req.params.id, ...req.body });
});

// PATCH - Partial update
app.patch('/users/:id', (req, res) => {
  res.json({ id: req.params.id, updated: true });
});

// DELETE - Remove data
app.delete('/users/:id', (req, res) => {
  res.status(204).send();
});

// ALL - Match all HTTP methods
app.all('/api/*', (req, res, next) => {
  console.log(`API call: ${req.method} ${req.url}`);
  next();
});
```

### Route Parameters

```javascript
// Single parameter
app.get('/users/:id', (req, res) => {
  const userId = req.params.id;
  res.json({ id: userId, name: 'User ' + userId });
});

// Multiple parameters
app.get('/users/:userId/posts/:postId', (req, res) => {
  const { userId, postId } = req.params;
  res.json({ userId, postId });
});

// Optional parameters (use ?)
app.get('/books/:isbn?', (req, res) => {
  if (req.params.isbn) {
    res.json({ isbn: req.params.isbn });
  } else {
    res.json({ message: 'List all books' });
  }
});

// Regex patterns
app.get('/users/:id(\\d+)', (req, res) => {
  // Only matches numeric IDs
  res.json({ id: parseInt(req.params.id) });
});
```

### Query Strings

```javascript
// GET /search?q=javascript&page=1&limit=10
app.get('/search', (req, res) => {
  const { q, page = 1, limit = 10 } = req.query;

  res.json({
    query: q,
    page: parseInt(page),
    limit: parseInt(limit),
  });
});

// GET /products?category=electronics&sort=price&order=asc
app.get('/products', (req, res) => {
  const { category, sort, order } = req.query;

  res.json({
    filters: { category, sort, order }
  });
});
```

### Route Methods

```javascript
// Chained route handlers
app.route('/articles')
  .get((req, res) => {
    res.json([{ title: 'Article 1' }]);
  })
  .post((req, res) => {
    res.status(201).json(req.body);
  });

app.route('/articles/:id')
  .get((req, res) => {
    res.json({ id: req.params.id });
  })
  .put((req, res) => {
    res.json({ id: req.params.id, ...req.body });
  })
  .delete((req, res) => {
    res.status(204).send();
  });
```

---

## Express Router

### Creating Routers

```javascript
// routes/users.js
const express = require('express');
const router = express.Router();

// All routes here are prefixed with /users (set in main app)
router.get('/', (req, res) => {
  res.json([{ id: 1, name: 'John' }]);
});

router.get('/:id', (req, res) => {
  res.json({ id: req.params.id });
});

router.post('/', (req, res) => {
  res.status(201).json(req.body);
});

router.put('/:id', (req, res) => {
  res.json({ id: req.params.id, ...req.body });
});

router.delete('/:id', (req, res) => {
  res.status(204).send();
});

module.exports = router;
```

```javascript
// routes/posts.js
const express = require('express');
const router = express.Router();

router.get('/', (req, res) => {
  res.json([{ id: 1, title: 'Post 1' }]);
});

router.get('/:id', (req, res) => {
  res.json({ id: req.params.id });
});

module.exports = router;
```

### Using Routers

```javascript
// app.js
const express = require('express');
const app = express();

const usersRouter = require('./routes/users');
const postsRouter = require('./routes/posts');

app.use(express.json());

// Mount routers
app.use('/users', usersRouter);
app.use('/posts', postsRouter);
app.use('/api/v1/users', usersRouter); // API versioning

app.listen(3000);
```

### Router-level Middleware

```javascript
const express = require('express');
const router = express.Router();

// Middleware for all routes in this router
router.use((req, res, next) => {
  console.log('Router middleware:', req.method, req.url);
  next();
});

// Middleware for specific route
router.get('/:id', validateId, (req, res) => {
  res.json({ id: req.params.id });
});

function validateId(req, res, next) {
  if (isNaN(req.params.id)) {
    return res.status(400).json({ error: 'Invalid ID' });
  }
  next();
}

module.exports = router;
```

---

## Request Object

### Common Properties

```javascript
app.get('/example', (req, res) => {
  // HTTP method
  console.log(req.method);  // GET

  // URL and path
  console.log(req.url);           // /example?foo=bar
  console.log(req.originalUrl);   // /example?foo=bar
  console.log(req.path);          // /example
  console.log(req.baseUrl);       // '' (or router prefix)

  // Parameters
  console.log(req.params);  // Route params { id: '123' }
  console.log(req.query);   // Query string { foo: 'bar' }
  console.log(req.body);    // POST/PUT body (needs middleware)

  // Headers
  console.log(req.headers);           // All headers
  console.log(req.get('Content-Type')); // Get specific header
  console.log(req.header('Accept'));    // Alias for get()

  // Other useful properties
  console.log(req.ip);              // Client IP
  console.log(req.hostname);        // Host name
  console.log(req.protocol);        // http or https
  console.log(req.secure);          // true if https
  console.log(req.xhr);             // true if AJAX request
  console.log(req.cookies);         // Cookies (needs cookie-parser)

  res.send('OK');
});
```

### Working with Body

```javascript
const express = require('express');
const app = express();

// Parse JSON bodies
app.use(express.json());

// Parse URL-encoded bodies (form data)
app.use(express.urlencoded({ extended: true }));

app.post('/users', (req, res) => {
  const { name, email } = req.body;
  res.json({ name, email });
});
```

---

## Response Object

### Sending Responses

```javascript
app.get('/response-examples', (req, res) => {
  // Send string
  res.send('Hello World');

  // Send JSON
  res.json({ message: 'Hello' });

  // Send status + JSON
  res.status(201).json({ created: true });

  // Send only status
  res.sendStatus(204);  // No Content

  // End response
  res.status(200).end();
});
```

### Response Methods

```javascript
// Set status code
res.status(404);
res.status(201);

// Set headers
res.set('X-Custom-Header', 'value');
res.set({
  'Content-Type': 'application/json',
  'X-Request-Id': '12345',
});

// Get header
res.get('Content-Type');

// Redirect
res.redirect('/new-url');
res.redirect(301, '/permanent-new-url');
res.redirect('back');  // Go back

// Send file
res.sendFile('/path/to/file.pdf');
res.sendFile('file.pdf', { root: __dirname });

// Download file
res.download('/path/to/file.pdf');
res.download('/path/to/file.pdf', 'custom-name.pdf');

// Render template (needs template engine)
res.render('index', { title: 'Home', user: req.user });

// Set cookie
res.cookie('name', 'value', { maxAge: 900000, httpOnly: true });
res.clearCookie('name');

// Format response based on Accept header
res.format({
  'text/plain': () => res.send('Hello'),
  'text/html': () => res.send('<h1>Hello</h1>'),
  'application/json': () => res.json({ message: 'Hello' }),
  default: () => res.status(406).send('Not Acceptable'),
});
```

### Chaining

```javascript
res
  .status(201)
  .set('X-Custom', 'value')
  .json({ success: true });
```

---

## Serving Static Files

### Basic Setup

```javascript
const express = require('express');
const path = require('path');
const app = express();

// Serve static files from 'public' directory
app.use(express.static('public'));

// With virtual path prefix
app.use('/static', express.static('public'));

// Absolute path (recommended)
app.use(express.static(path.join(__dirname, 'public')));

// Multiple directories
app.use(express.static('public'));
app.use(express.static('files'));

// With options
app.use(express.static('public', {
  maxAge: '1d',      // Cache for 1 day
  etag: true,        // Enable ETags
  index: 'index.html', // Default file
  extensions: ['html', 'htm'], // Try extensions
}));
```

### Project Structure

```
project/
├── app.js
└── public/
    ├── index.html
    ├── css/
    │   └── style.css
    ├── js/
    │   └── app.js
    └── images/
        └── logo.png
```

Access files:
- `http://localhost:3000/index.html`
- `http://localhost:3000/css/style.css`
- `http://localhost:3000/images/logo.png`

---

## Template Engines

### EJS Setup

```bash
npm install ejs
```

```javascript
const express = require('express');
const path = require('path');
const app = express();

// Set view engine
app.set('view engine', 'ejs');
app.set('views', path.join(__dirname, 'views'));

app.get('/', (req, res) => {
  res.render('index', {
    title: 'Home Page',
    user: { name: 'John' },
    items: ['Apple', 'Banana', 'Cherry'],
  });
});
```

```html
<!-- views/index.ejs -->
<!DOCTYPE html>
<html>
<head>
  <title><%= title %></title>
</head>
<body>
  <h1>Welcome, <%= user.name %>!</h1>

  <% if (items.length > 0) { %>
    <ul>
      <% items.forEach(item => { %>
        <li><%= item %></li>
      <% }); %>
    </ul>
  <% } %>

  <%- include('partials/footer') %>
</body>
</html>
```

```html
<!-- views/partials/footer.ejs -->
<footer>
  <p>&copy; 2024 My App</p>
</footer>
```

### Pug (Jade) Setup

```bash
npm install pug
```

```javascript
app.set('view engine', 'pug');
app.set('views', './views');

app.get('/', (req, res) => {
  res.render('index', { title: 'Home', user: 'John' });
});
```

```pug
//- views/index.pug
doctype html
html
  head
    title= title
  body
    h1 Welcome, #{user}!
    ul
      each item in items
        li= item
```

---

## Error Handling

### Basic Error Handling

```javascript
const express = require('express');
const app = express();

// Route that throws error
app.get('/error', (req, res) => {
  throw new Error('Something went wrong!');
});

// Async error (must use next or try/catch)
app.get('/async-error', async (req, res, next) => {
  try {
    await someAsyncOperation();
    res.json({ success: true });
  } catch (error) {
    next(error); // Pass error to error handler
  }
});

// 404 handler (must be after all routes)
app.use((req, res, next) => {
  res.status(404).json({ error: 'Not Found' });
});

// Error handler (must be last, with 4 parameters)
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ error: 'Internal Server Error' });
});
```

### Custom Error Class

```javascript
// errors/AppError.js
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
    this.status = `${statusCode}`.startsWith('4') ? 'fail' : 'error';
    this.isOperational = true;

    Error.captureStackTrace(this, this.constructor);
  }
}

module.exports = AppError;
```

```javascript
// Usage in routes
const AppError = require('./errors/AppError');

app.get('/users/:id', async (req, res, next) => {
  const user = await User.findById(req.params.id);

  if (!user) {
    return next(new AppError('User not found', 404));
  }

  res.json(user);
});

// Error handler
app.use((err, req, res, next) => {
  err.statusCode = err.statusCode || 500;
  err.status = err.status || 'error';

  res.status(err.statusCode).json({
    status: err.status,
    message: err.message,
    ...(process.env.NODE_ENV === 'development' && { stack: err.stack }),
  });
});
```

### Async Error Wrapper

```javascript
// Wrap async handlers to catch errors
const catchAsync = (fn) => {
  return (req, res, next) => {
    fn(req, res, next).catch(next);
  };
};

// Usage
app.get('/users', catchAsync(async (req, res) => {
  const users = await User.findAll();
  res.json(users);
}));

app.get('/users/:id', catchAsync(async (req, res) => {
  const user = await User.findById(req.params.id);
  if (!user) throw new AppError('User not found', 404);
  res.json(user);
}));
```

---

## Complete Application Structure

```
my-api/
├── app.js              # Express app setup
├── server.js           # Server entry point
├── package.json
├── .env
├── routes/
│   ├── index.js
│   ├── users.js
│   └── posts.js
├── controllers/
│   ├── userController.js
│   └── postController.js
├── middleware/
│   ├── auth.js
│   └── errorHandler.js
├── utils/
│   ├── AppError.js
│   └── catchAsync.js
├── public/
│   ├── css/
│   └── js/
└── views/
    └── index.ejs
```

```javascript
// app.js
const express = require('express');
const path = require('path');

const usersRouter = require('./routes/users');
const postsRouter = require('./routes/posts');
const AppError = require('./utils/AppError');
const errorHandler = require('./middleware/errorHandler');

const app = express();

// Body parsing
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// Static files
app.use(express.static(path.join(__dirname, 'public')));

// View engine
app.set('view engine', 'ejs');
app.set('views', path.join(__dirname, 'views'));

// Routes
app.use('/api/users', usersRouter);
app.use('/api/posts', postsRouter);

// 404 handler
app.use((req, res, next) => {
  next(new AppError(`Can't find ${req.originalUrl}`, 404));
});

// Error handler
app.use(errorHandler);

module.exports = app;
```

```javascript
// server.js
require('dotenv').config();
const app = require('./app');

const PORT = process.env.PORT || 3000;

app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```

---

## Exercise: Build a Tasks API

Create a simple REST API for managing tasks:
- GET /api/tasks - List all tasks
- GET /api/tasks/:id - Get a single task
- POST /api/tasks - Create a task
- PUT /api/tasks/:id - Update a task
- DELETE /api/tasks/:id - Delete a task

---

## Solution

```javascript
// app.js
const express = require('express');
const app = express();

app.use(express.json());

// In-memory database
let tasks = [
  { id: 1, title: 'Learn Express', completed: false },
  { id: 2, title: 'Build API', completed: false },
];

let nextId = 3;

// Validation helper
const validateTask = (task) => {
  const errors = [];
  if (!task.title) errors.push('Title is required');
  if (task.title && task.title.length < 3) {
    errors.push('Title must be at least 3 characters');
  }
  return errors;
};

// Routes
app.get('/api/tasks', (req, res) => {
  const { completed, search } = req.query;

  let result = [...tasks];

  // Filter by completion status
  if (completed !== undefined) {
    result = result.filter(t => t.completed === (completed === 'true'));
  }

  // Search by title
  if (search) {
    result = result.filter(t =>
      t.title.toLowerCase().includes(search.toLowerCase())
    );
  }

  res.json({
    count: result.length,
    tasks: result,
  });
});

app.get('/api/tasks/:id', (req, res) => {
  const id = parseInt(req.params.id);
  const task = tasks.find(t => t.id === id);

  if (!task) {
    return res.status(404).json({ error: 'Task not found' });
  }

  res.json(task);
});

app.post('/api/tasks', (req, res) => {
  const { title, completed = false } = req.body;

  const errors = validateTask({ title });
  if (errors.length > 0) {
    return res.status(400).json({ errors });
  }

  const task = {
    id: nextId++,
    title,
    completed,
    createdAt: new Date().toISOString(),
  };

  tasks.push(task);
  res.status(201).json(task);
});

app.put('/api/tasks/:id', (req, res) => {
  const id = parseInt(req.params.id);
  const taskIndex = tasks.findIndex(t => t.id === id);

  if (taskIndex === -1) {
    return res.status(404).json({ error: 'Task not found' });
  }

  const { title, completed } = req.body;

  const errors = validateTask({ title });
  if (errors.length > 0) {
    return res.status(400).json({ errors });
  }

  tasks[taskIndex] = {
    ...tasks[taskIndex],
    title,
    completed: completed ?? tasks[taskIndex].completed,
    updatedAt: new Date().toISOString(),
  };

  res.json(tasks[taskIndex]);
});

app.patch('/api/tasks/:id', (req, res) => {
  const id = parseInt(req.params.id);
  const task = tasks.find(t => t.id === id);

  if (!task) {
    return res.status(404).json({ error: 'Task not found' });
  }

  // Toggle completed status
  if (req.body.completed !== undefined) {
    task.completed = req.body.completed;
  }

  if (req.body.title) {
    const errors = validateTask({ title: req.body.title });
    if (errors.length > 0) {
      return res.status(400).json({ errors });
    }
    task.title = req.body.title;
  }

  task.updatedAt = new Date().toISOString();
  res.json(task);
});

app.delete('/api/tasks/:id', (req, res) => {
  const id = parseInt(req.params.id);
  const taskIndex = tasks.findIndex(t => t.id === id);

  if (taskIndex === -1) {
    return res.status(404).json({ error: 'Task not found' });
  }

  tasks.splice(taskIndex, 1);
  res.status(204).send();
});

// 404 handler
app.use((req, res) => {
  res.status(404).json({ error: 'Not Found' });
});

// Error handler
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ error: 'Internal Server Error' });
});

const PORT = 3000;
app.listen(PORT, () => {
  console.log(`Tasks API running on http://localhost:${PORT}`);
});
```

Test with curl:

```bash
# List all tasks
curl http://localhost:3000/api/tasks

# Get single task
curl http://localhost:3000/api/tasks/1

# Create task
curl -X POST http://localhost:3000/api/tasks \
  -H "Content-Type: application/json" \
  -d '{"title": "New Task"}'

# Update task
curl -X PUT http://localhost:3000/api/tasks/1 \
  -H "Content-Type: application/json" \
  -d '{"title": "Updated Task", "completed": true}'

# Delete task
curl -X DELETE http://localhost:3000/api/tasks/1

# Filter completed
curl http://localhost:3000/api/tasks?completed=true

# Search
curl http://localhost:3000/api/tasks?search=learn
```

---

## Key Takeaways

1. **Express simplifies Node.js** - Clean routing and middleware system
2. **Use routers for organization** - Split routes into modules
3. **Understand req and res** - Master the request/response objects
4. **Handle errors properly** - Use error middleware and custom errors
5. **Static files are easy** - Just use `express.static()`
6. **Template engines for HTML** - EJS, Pug, Handlebars
7. **Structure matters** - Organize code in routes, controllers, middleware

---

## What's Next?

Tomorrow we'll dive deep into **Express Middleware** - the powerful concept that makes Express so flexible and extensible!
