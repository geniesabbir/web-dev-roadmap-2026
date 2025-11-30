# Day 2: Node.js Modules and NPM

## Introduction

Modules are the building blocks of Node.js applications. Understanding how to create, export, and import modules, along with mastering NPM (Node Package Manager), is essential for building scalable applications.

## Learning Objectives

By the end of this lesson, you will:
- Understand CommonJS and ES Modules
- Create and export custom modules
- Use NPM to manage dependencies
- Understand package.json and package-lock.json
- Work with semantic versioning
- Publish packages to NPM

---

## Module Systems

### CommonJS (CJS) - Traditional Node.js

```javascript
// math.js - Exporting
const add = (a, b) => a + b;
const subtract = (a, b) => a - b;

// Export individual functions
module.exports.add = add;
module.exports.subtract = subtract;

// Or export as object
module.exports = {
  add,
  subtract,
  multiply: (a, b) => a * b,
};
```

```javascript
// app.js - Importing
const math = require('./math');

console.log(math.add(2, 3));      // 5
console.log(math.multiply(4, 5)); // 20

// Destructuring import
const { add, subtract } = require('./math');
console.log(add(10, 5)); // 15
```

### ES Modules (ESM) - Modern JavaScript

```javascript
// math.mjs (or .js with "type": "module" in package.json)

// Named exports
export const add = (a, b) => a + b;
export const subtract = (a, b) => a - b;

// Default export
export default function multiply(a, b) {
  return a * b;
}
```

```javascript
// app.mjs - Importing
import multiply, { add, subtract } from './math.mjs';

console.log(add(2, 3));       // 5
console.log(multiply(4, 5));  // 20

// Import all
import * as math from './math.mjs';
console.log(math.add(1, 2));
console.log(math.default(3, 4)); // default export
```

### Enabling ES Modules

```json
// package.json
{
  "name": "my-app",
  "type": "module"  // All .js files are ES modules
}
```

Or use `.mjs` extension for ES modules and `.cjs` for CommonJS.

### Key Differences

| Feature | CommonJS | ES Modules |
|---------|----------|------------|
| Syntax | `require()`, `module.exports` | `import`, `export` |
| Loading | Synchronous | Asynchronous |
| Top-level await | No | Yes |
| File extension | `.js`, `.cjs` | `.mjs` or `.js` with type: module |
| `this` at top level | `module.exports` | `undefined` |
| Dynamic import | `require()` | `import()` |

---

## Creating Modules

### Module Patterns

```javascript
// 1. Object literal pattern
// config.js
module.exports = {
  port: 3000,
  database: {
    host: 'localhost',
    name: 'mydb',
  },
};

// 2. Factory function pattern
// logger.js
module.exports = function createLogger(prefix) {
  return {
    log: (msg) => console.log(`[${prefix}] ${msg}`),
    error: (msg) => console.error(`[${prefix}] ERROR: ${msg}`),
  };
};

// Usage
const logger = require('./logger')('App');
logger.log('Starting...'); // [App] Starting...

// 3. Class pattern
// user-service.js
class UserService {
  constructor(db) {
    this.db = db;
  }

  async findById(id) {
    return this.db.query('SELECT * FROM users WHERE id = ?', [id]);
  }

  async create(userData) {
    return this.db.query('INSERT INTO users SET ?', userData);
  }
}

module.exports = UserService;

// 4. Singleton pattern
// database.js
let instance = null;

class Database {
  constructor() {
    if (instance) {
      return instance;
    }
    this.connection = null;
    instance = this;
  }

  connect() {
    if (!this.connection) {
      this.connection = createConnection();
    }
    return this.connection;
  }
}

module.exports = new Database();
```

### Module Organization

```
project/
├── src/
│   ├── config/
│   │   ├── index.js        # Re-exports all configs
│   │   ├── database.js
│   │   └── server.js
│   ├── services/
│   │   ├── index.js
│   │   ├── user.service.js
│   │   └── auth.service.js
│   ├── utils/
│   │   ├── index.js
│   │   ├── logger.js
│   │   └── helpers.js
│   └── index.js
├── package.json
└── node_modules/
```

### Barrel Exports (Index Files)

```javascript
// services/index.js
export { UserService } from './user.service.js';
export { AuthService } from './auth.service.js';
export { EmailService } from './email.service.js';

// Usage in other files
import { UserService, AuthService } from './services';
```

---

## NPM Basics

### Initializing a Project

```bash
# Create package.json interactively
npm init

# Create with defaults
npm init -y

# Example package.json
{
  "name": "my-app",
  "version": "1.0.0",
  "description": "My awesome application",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "dev": "nodemon index.js",
    "test": "jest"
  },
  "keywords": ["nodejs", "api"],
  "author": "Your Name",
  "license": "MIT"
}
```

### Installing Packages

```bash
# Install and save to dependencies
npm install express
npm i express  # Short form

# Install multiple packages
npm install express mongoose dotenv

# Install specific version
npm install express@4.18.2

# Install latest version of major version
npm install express@4

# Install as dev dependency
npm install --save-dev jest
npm install -D jest

# Install globally
npm install -g nodemon

# Install from package.json
npm install
npm i

# Install production dependencies only
npm install --production
```

### Managing Packages

```bash
# Update packages
npm update              # Update all
npm update express      # Update specific

# Check outdated packages
npm outdated

# Uninstall package
npm uninstall express
npm remove express
npm rm express

# List installed packages
npm list
npm list --depth=0      # Top level only
npm list -g --depth=0   # Global packages

# View package info
npm view express
npm view express versions  # All versions

# Search packages
npm search express

# Check for security vulnerabilities
npm audit
npm audit fix           # Auto-fix vulnerabilities
```

---

## Package.json Deep Dive

### Complete Structure

```json
{
  "name": "my-application",
  "version": "1.0.0",
  "description": "A Node.js application",
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "type": "module",
  "engines": {
    "node": ">=18.0.0"
  },
  "scripts": {
    "start": "node dist/index.js",
    "dev": "tsx watch src/index.ts",
    "build": "tsc",
    "test": "vitest",
    "test:coverage": "vitest --coverage",
    "lint": "eslint src/",
    "format": "prettier --write src/"
  },
  "dependencies": {
    "express": "^4.18.2",
    "mongoose": "^7.0.0"
  },
  "devDependencies": {
    "typescript": "^5.0.0",
    "vitest": "^1.0.0",
    "@types/express": "^4.17.17"
  },
  "peerDependencies": {
    "react": "^18.0.0"
  },
  "optionalDependencies": {
    "fsevents": "^2.3.0"
  },
  "keywords": ["api", "nodejs", "express"],
  "author": {
    "name": "Your Name",
    "email": "email@example.com",
    "url": "https://yourwebsite.com"
  },
  "license": "MIT",
  "repository": {
    "type": "git",
    "url": "https://github.com/username/repo.git"
  },
  "bugs": {
    "url": "https://github.com/username/repo/issues"
  },
  "homepage": "https://github.com/username/repo#readme",
  "files": [
    "dist",
    "README.md"
  ],
  "exports": {
    ".": {
      "import": "./dist/index.js",
      "require": "./dist/index.cjs"
    },
    "./utils": "./dist/utils/index.js"
  }
}
```

### Dependency Types

```json
{
  // Production dependencies - required to run
  "dependencies": {
    "express": "^4.18.2"
  },

  // Development only - not in production
  "devDependencies": {
    "jest": "^29.0.0"
  },

  // Required by packages using yours
  "peerDependencies": {
    "react": "^18.0.0"
  },

  // Nice to have, won't fail if missing
  "optionalDependencies": {
    "fsevents": "^2.3.0"
  }
}
```

---

## Semantic Versioning

### Version Format

```
MAJOR.MINOR.PATCH
  │     │     │
  │     │     └─── Bug fixes (backward compatible)
  │     └───────── New features (backward compatible)
  └─────────────── Breaking changes
```

### Version Ranges

```json
{
  "dependencies": {
    // Exact version
    "exact": "1.0.0",

    // Caret (^) - Allow minor and patch updates
    "caret": "^1.2.3",     // >=1.2.3 <2.0.0

    // Tilde (~) - Allow only patch updates
    "tilde": "~1.2.3",     // >=1.2.3 <1.3.0

    // Greater than
    "gt": ">1.2.3",

    // Greater than or equal
    "gte": ">=1.2.3",

    // Range
    "range": ">=1.2.3 <2.0.0",

    // OR
    "or": "1.2.3 || 2.0.0",

    // Latest
    "latest": "*",

    // Any version
    "any": "x"
  }
}
```

### Best Practices

```json
{
  "dependencies": {
    // Use caret for most dependencies
    "express": "^4.18.2",

    // Pin exact version for critical deps
    "stripe": "14.0.0",

    // Use tilde for databases (less risky updates)
    "mongodb": "~6.2.0"
  }
}
```

---

## Package-lock.json

### Purpose

- **Locks exact versions** of all dependencies
- **Ensures reproducible builds** across environments
- **Speeds up installation** with cached resolution

### Structure

```json
{
  "name": "my-app",
  "version": "1.0.0",
  "lockfileVersion": 3,
  "packages": {
    "": {
      "name": "my-app",
      "version": "1.0.0",
      "dependencies": {
        "express": "^4.18.2"
      }
    },
    "node_modules/express": {
      "version": "4.18.2",
      "resolved": "https://registry.npmjs.org/express/-/express-4.18.2.tgz",
      "integrity": "sha512-5/PsL6i...",
      "dependencies": {
        "accepts": "~1.3.8",
        "body-parser": "1.20.1"
      }
    }
  }
}
```

### Best Practices

```bash
# Always commit package-lock.json
git add package-lock.json

# Use npm ci for CI/CD (clean install from lock file)
npm ci

# Don't edit package-lock.json manually

# Update lock file when updating packages
npm update
```

---

## NPM Scripts

### Common Scripts

```json
{
  "scripts": {
    // Lifecycle scripts
    "preinstall": "echo 'Before npm install'",
    "postinstall": "echo 'After npm install'",
    "prepublish": "npm run build",

    // Common scripts
    "start": "node dist/index.js",
    "dev": "nodemon src/index.js",
    "build": "tsc",
    "test": "jest",
    "lint": "eslint src/",
    "format": "prettier --write .",

    // Custom scripts
    "db:migrate": "prisma migrate dev",
    "db:seed": "node scripts/seed.js",
    "docker:up": "docker-compose up -d",
    "docker:down": "docker-compose down"
  }
}
```

### Running Scripts

```bash
# Run a script
npm run dev
npm run build

# Special scripts (don't need 'run')
npm start
npm test

# Run with arguments
npm run dev -- --port 4000

# Run multiple scripts
npm run lint && npm run test

# Run scripts in parallel (with npm-run-all)
npm-run-all --parallel lint test
```

### Script Hooks

```json
{
  "scripts": {
    "pretest": "npm run lint",      // Runs before test
    "test": "jest",
    "posttest": "echo 'Tests done'", // Runs after test

    "prebuild": "rm -rf dist",
    "build": "tsc",
    "postbuild": "cp package.json dist/"
  }
}
```

---

## Working with Environment Variables

### Using dotenv

```bash
npm install dotenv
```

```javascript
// .env
PORT=3000
DATABASE_URL=mongodb://localhost/mydb
API_KEY=your-secret-key

// app.js
require('dotenv').config();

console.log(process.env.PORT);        // 3000
console.log(process.env.DATABASE_URL);// mongodb://localhost/mydb
```

### Different Environments

```
.env                 # Default, loaded in all environments
.env.local           # Local overrides, not committed
.env.development     # Development environment
.env.production      # Production environment
.env.test            # Test environment
```

```javascript
// Load based on NODE_ENV
const envFile = `.env.${process.env.NODE_ENV || 'development'}`;
require('dotenv').config({ path: envFile });
```

### Best Practices

```bash
# .gitignore
.env
.env.local
.env*.local

# Commit example file
.env.example
```

```javascript
// Validate required env vars
const required = ['DATABASE_URL', 'API_KEY'];
for (const key of required) {
  if (!process.env[key]) {
    throw new Error(`Missing required environment variable: ${key}`);
  }
}
```

---

## Creating Your Own Package

### Package Structure

```
my-package/
├── src/
│   ├── index.js
│   └── utils.js
├── dist/           # Compiled output
├── tests/
│   └── index.test.js
├── package.json
├── README.md
├── LICENSE
├── .gitignore
└── .npmignore
```

### Package.json for Publishing

```json
{
  "name": "@username/my-package",
  "version": "1.0.0",
  "description": "A useful package",
  "main": "dist/index.js",
  "module": "dist/index.mjs",
  "types": "dist/index.d.ts",
  "files": [
    "dist"
  ],
  "exports": {
    ".": {
      "require": "./dist/index.js",
      "import": "./dist/index.mjs",
      "types": "./dist/index.d.ts"
    }
  },
  "scripts": {
    "build": "tsup src/index.ts --format cjs,esm --dts",
    "prepublishOnly": "npm run build"
  },
  "repository": {
    "type": "git",
    "url": "https://github.com/username/my-package.git"
  },
  "keywords": ["utility", "helper"],
  "author": "Your Name",
  "license": "MIT"
}
```

### Publishing

```bash
# Login to NPM
npm login

# Publish public package
npm publish

# Publish scoped package (public)
npm publish --access public

# Update version and publish
npm version patch  # 1.0.0 -> 1.0.1
npm version minor  # 1.0.0 -> 1.1.0
npm version major  # 1.0.0 -> 2.0.0
npm publish

# Unpublish (within 72 hours)
npm unpublish @username/my-package@1.0.0
```

---

## npx - Execute Packages

```bash
# Run without installing globally
npx create-react-app my-app
npx ts-node script.ts
npx eslint .

# Run specific version
npx express@4.18.0 --help

# Run from GitHub
npx github:username/repo

# Initialize common projects
npx create-next-app@latest
npx create-vite@latest
npx prisma init
```

---

## Workspaces (Monorepos)

### Setup

```json
// package.json (root)
{
  "name": "my-monorepo",
  "private": true,
  "workspaces": [
    "packages/*",
    "apps/*"
  ]
}
```

### Structure

```
my-monorepo/
├── package.json
├── packages/
│   ├── shared/
│   │   └── package.json
│   └── utils/
│       └── package.json
└── apps/
    ├── web/
    │   └── package.json
    └── api/
        └── package.json
```

### Commands

```bash
# Install all workspace dependencies
npm install

# Run script in specific workspace
npm run build -w packages/shared
npm run test -w apps/web

# Add dependency to workspace
npm install express -w apps/api

# Add workspace as dependency
# In apps/api/package.json:
{
  "dependencies": {
    "@monorepo/shared": "*"
  }
}
```

---

## Exercise: Create a Logger Package

Create a reusable logger package with:
1. Different log levels (debug, info, warn, error)
2. Colorized output
3. Timestamps
4. Configurable options

---

## Solution

```javascript
// src/index.js
const colors = {
  reset: '\x1b[0m',
  red: '\x1b[31m',
  green: '\x1b[32m',
  yellow: '\x1b[33m',
  blue: '\x1b[34m',
  gray: '\x1b[90m',
};

const levels = {
  debug: { color: colors.gray, priority: 0 },
  info: { color: colors.blue, priority: 1 },
  warn: { color: colors.yellow, priority: 2 },
  error: { color: colors.red, priority: 3 },
};

class Logger {
  constructor(options = {}) {
    this.prefix = options.prefix || '';
    this.minLevel = options.minLevel || 'debug';
    this.showTimestamp = options.showTimestamp ?? true;
    this.colorize = options.colorize ?? true;
  }

  #formatMessage(level, message) {
    const parts = [];

    if (this.showTimestamp) {
      const timestamp = new Date().toISOString();
      parts.push(`[${timestamp}]`);
    }

    parts.push(`[${level.toUpperCase()}]`);

    if (this.prefix) {
      parts.push(`[${this.prefix}]`);
    }

    parts.push(message);

    const formatted = parts.join(' ');

    if (this.colorize) {
      const { color } = levels[level];
      return `${color}${formatted}${colors.reset}`;
    }

    return formatted;
  }

  #shouldLog(level) {
    return levels[level].priority >= levels[this.minLevel].priority;
  }

  #log(level, ...args) {
    if (!this.#shouldLog(level)) return;

    const message = args
      .map(arg => (typeof arg === 'object' ? JSON.stringify(arg, null, 2) : arg))
      .join(' ');

    const formatted = this.#formatMessage(level, message);

    if (level === 'error') {
      console.error(formatted);
    } else {
      console.log(formatted);
    }
  }

  debug(...args) {
    this.#log('debug', ...args);
  }

  info(...args) {
    this.#log('info', ...args);
  }

  warn(...args) {
    this.#log('warn', ...args);
  }

  error(...args) {
    this.#log('error', ...args);
  }

  child(options) {
    return new Logger({
      prefix: options.prefix
        ? `${this.prefix}:${options.prefix}`
        : this.prefix,
      minLevel: options.minLevel || this.minLevel,
      showTimestamp: options.showTimestamp ?? this.showTimestamp,
      colorize: options.colorize ?? this.colorize,
    });
  }
}

function createLogger(options) {
  return new Logger(options);
}

module.exports = { Logger, createLogger };
module.exports.default = createLogger;
```

```javascript
// Usage example
const { createLogger } = require('./src/index');

const logger = createLogger({
  prefix: 'App',
  minLevel: 'debug',
  showTimestamp: true,
  colorize: true,
});

logger.debug('Debugging application...');
logger.info('Server started on port 3000');
logger.warn('Memory usage is high');
logger.error('Database connection failed');

// Child logger
const dbLogger = logger.child({ prefix: 'Database' });
dbLogger.info('Connected to MongoDB');
dbLogger.error('Query failed', { table: 'users', error: 'timeout' });
```

```json
// package.json
{
  "name": "@yourname/logger",
  "version": "1.0.0",
  "description": "A simple colorized logger for Node.js",
  "main": "src/index.js",
  "exports": {
    ".": "./src/index.js"
  },
  "files": ["src"],
  "keywords": ["logger", "logging", "console"],
  "author": "Your Name",
  "license": "MIT"
}
```

---

## Key Takeaways

1. **Two module systems** - CommonJS (require) and ES Modules (import)
2. **ES Modules are the future** - Use them for new projects
3. **package.json is crucial** - Defines your project and dependencies
4. **Semantic versioning matters** - Understand ^, ~, and exact versions
5. **Commit package-lock.json** - Ensures reproducible builds
6. **Use npm scripts** - Automate common tasks
7. **dotenv for configuration** - Keep secrets out of code
8. **npx for one-off commands** - No global installs needed

---

## What's Next?

Tomorrow we'll learn **Express.js Basics** - the most popular Node.js framework for building web applications and APIs!
