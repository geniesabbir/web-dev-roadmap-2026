# Node.js & Express

**Duration:** 3-4 weeks

## Learning Objectives

By the end of this section, you will:
- Understand Node.js runtime and event loop
- Build REST APIs with Express
- Implement middleware patterns
- Handle errors properly
- Validate input data
- Structure backend projects

---

## Week 1: Node.js Fundamentals

### Day 1: Node.js Basics
**Topics:**
- What is Node.js?
- Node vs Browser JavaScript
- REPL and running files
- Global objects (`__dirname`, `__filename`, `process`)
- Module system (CommonJS vs ESM)

**Setup:**
```bash
# Check Node version (should be 18+)
node --version

# Initialize a project
mkdir my-api && cd my-api
npm init -y

# Enable ES modules in package.json
# Add: "type": "module"
```

**Exercises:**
```javascript
// exercises/01-node-basics/

// 1. Create and run a Node.js script
// 2. Use process.argv to read command line args
// 3. Work with environment variables (process.env)
// 4. Create and import modules
```

### Day 2: Core Modules
**Topics:**
- `fs` (file system)
- `path`
- `http`
- `events`
- `stream`

**Exercises:**
```javascript
// exercises/02-core-modules/

// 1. Read and write files (sync and async)
// 2. Create a simple HTTP server
// 3. Work with paths cross-platform
// 4. Use EventEmitter
```

### Day 3: Async Patterns in Node
**Topics:**
- Callbacks in Node
- Promises and util.promisify
- async/await
- Error-first callbacks
- The event loop

**Exercises:**
```javascript
// exercises/03-async-node/

// 1. Convert callback-based code to promises
// 2. Read multiple files in parallel
// 3. Process files sequentially
// 4. Handle errors in async operations
```

### Day 4: npm Ecosystem
**Topics:**
- Package.json deep dive
- npm commands
- Semantic versioning
- Dependencies vs devDependencies
- npm scripts
- Lock files

**Exercises:**
```bash
# exercises/04-npm/

# 1. Initialize a project with specific settings
# 2. Install and use popular packages
# 3. Create useful npm scripts
# 4. Understand and use .npmrc
```

### Day 5: Build a CLI Tool
**Project:** Build a simple command-line tool
- Accept command line arguments
- Read/write files
- Use colors and spinners
- Handle errors gracefully

---

## Week 2: Express.js Basics

### Day 1: Express Setup
**Topics:**
- What is Express?
- Installation and setup
- Basic server
- Request and Response objects
- Environment variables with dotenv

**Setup:**
```bash
npm install express
npm install -D typescript @types/express @types/node tsx
npm install dotenv
```

**Basic Server:**
```typescript
// src/index.ts
import express from 'express'
import dotenv from 'dotenv'

dotenv.config()

const app = express()
const PORT = process.env.PORT || 3000

app.get('/', (req, res) => {
  res.json({ message: 'Hello World!' })
})

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`)
})
```

**Exercises:**
```typescript
// exercises/05-express-setup/

// 1. Create an Express server
// 2. Add multiple routes
// 3. Use environment variables
// 4. Return JSON responses
```

### Day 2: Routing
**Topics:**
- Route methods (GET, POST, PUT, PATCH, DELETE)
- Route parameters
- Query strings
- Route handlers
- Express Router

**Exercises:**
```typescript
// exercises/06-routing/

// 1. Create CRUD routes for a resource
// 2. Use route parameters (/users/:id)
// 3. Handle query strings (/search?q=term)
// 4. Organize routes with Router
```

### Day 3: Middleware
**Topics:**
- What is middleware?
- Application-level middleware
- Router-level middleware
- Error-handling middleware
- Built-in middleware
- Third-party middleware

**Common Middleware:**
```typescript
import express from 'express'
import cors from 'cors'
import helmet from 'helmet'
import morgan from 'morgan'

const app = express()

// Built-in middleware
app.use(express.json())
app.use(express.urlencoded({ extended: true }))

// Third-party middleware
app.use(cors())
app.use(helmet())
app.use(morgan('dev'))

// Custom middleware
app.use((req, res, next) => {
  console.log(`${req.method} ${req.path}`)
  next()
})
```

**Exercises:**
```typescript
// exercises/07-middleware/

// 1. Create a logging middleware
// 2. Create an authentication middleware
// 3. Create a request timing middleware
// 4. Set up error handling middleware
```

### Day 4: Request Handling
**Topics:**
- Request body parsing
- File uploads (multer)
- Request validation
- Response methods
- Status codes

**Exercises:**
```typescript
// exercises/08-request-handling/

// 1. Parse JSON and form data
// 2. Handle file uploads
// 3. Return different response types
// 4. Use appropriate status codes
```

### Day 5: Error Handling
**Topics:**
- Error handling patterns
- Custom error classes
- Async error handling
- 404 handling
- Global error handler

**Error Handling Pattern:**
```typescript
// src/utils/AppError.ts
export class AppError extends Error {
  statusCode: number
  isOperational: boolean

  constructor(message: string, statusCode: number) {
    super(message)
    this.statusCode = statusCode
    this.isOperational = true

    Error.captureStackTrace(this, this.constructor)
  }
}

// src/middleware/errorHandler.ts
import { Request, Response, NextFunction } from 'express'
import { AppError } from '../utils/AppError'

export function errorHandler(
  err: Error,
  req: Request,
  res: Response,
  next: NextFunction
) {
  if (err instanceof AppError) {
    return res.status(err.statusCode).json({
      status: 'error',
      message: err.message,
    })
  }

  console.error('ERROR:', err)
  return res.status(500).json({
    status: 'error',
    message: 'Something went wrong',
  })
}

// Async wrapper
export const asyncHandler = (fn: Function) => {
  return (req: Request, res: Response, next: NextFunction) => {
    Promise.resolve(fn(req, res, next)).catch(next)
  }
}
```

---

## Week 3: Building APIs

### Day 1: REST API Structure
**Topics:**
- Project structure
- Controllers, services, routes
- Separation of concerns
- Dependency injection basics

**Project Structure:**
```
src/
├── index.ts
├── app.ts
├── config/
│   └── index.ts
├── routes/
│   ├── index.ts
│   └── userRoutes.ts
├── controllers/
│   └── userController.ts
├── services/
│   └── userService.ts
├── middleware/
│   ├── auth.ts
│   ├── validate.ts
│   └── errorHandler.ts
├── utils/
│   └── AppError.ts
└── types/
    └── index.ts
```

### Day 2: Validation
**Topics:**
- Why validate?
- Zod for validation
- Request validation middleware
- Schema reuse
- Custom validators

**Setup:**
```bash
npm install zod
```

**Validation Pattern:**
```typescript
// src/schemas/userSchema.ts
import { z } from 'zod'

export const createUserSchema = z.object({
  body: z.object({
    email: z.string().email('Invalid email'),
    password: z.string().min(8, 'Password must be at least 8 characters'),
    name: z.string().min(2, 'Name must be at least 2 characters'),
  }),
})

export const updateUserSchema = z.object({
  body: z.object({
    email: z.string().email().optional(),
    name: z.string().min(2).optional(),
  }),
  params: z.object({
    id: z.string().uuid(),
  }),
})

// src/middleware/validate.ts
import { Request, Response, NextFunction } from 'express'
import { AnyZodObject, ZodError } from 'zod'

export const validate = (schema: AnyZodObject) => {
  return async (req: Request, res: Response, next: NextFunction) => {
    try {
      await schema.parseAsync({
        body: req.body,
        query: req.query,
        params: req.params,
      })
      next()
    } catch (error) {
      if (error instanceof ZodError) {
        return res.status(400).json({
          status: 'error',
          errors: error.errors,
        })
      }
      next(error)
    }
  }
}

// Usage in routes
router.post('/users', validate(createUserSchema), userController.create)
```

### Day 3: Controllers and Services
**Topics:**
- Controller responsibilities
- Service layer pattern
- Thin controllers
- Business logic separation

**Example:**
```typescript
// src/services/userService.ts
import { prisma } from '../config/database'
import { AppError } from '../utils/AppError'
import bcrypt from 'bcrypt'

export const userService = {
  async findAll() {
    return prisma.user.findMany({
      select: { id: true, email: true, name: true, createdAt: true },
    })
  },

  async findById(id: string) {
    const user = await prisma.user.findUnique({
      where: { id },
      select: { id: true, email: true, name: true, createdAt: true },
    })

    if (!user) {
      throw new AppError('User not found', 404)
    }

    return user
  },

  async create(data: { email: string; password: string; name: string }) {
    const existingUser = await prisma.user.findUnique({
      where: { email: data.email },
    })

    if (existingUser) {
      throw new AppError('Email already exists', 400)
    }

    const hashedPassword = await bcrypt.hash(data.password, 10)

    return prisma.user.create({
      data: { ...data, password: hashedPassword },
      select: { id: true, email: true, name: true, createdAt: true },
    })
  },
}

// src/controllers/userController.ts
import { Request, Response } from 'express'
import { userService } from '../services/userService'
import { asyncHandler } from '../middleware/errorHandler'

export const userController = {
  getAll: asyncHandler(async (req: Request, res: Response) => {
    const users = await userService.findAll()
    res.json({ status: 'success', data: users })
  }),

  getById: asyncHandler(async (req: Request, res: Response) => {
    const user = await userService.findById(req.params.id)
    res.json({ status: 'success', data: user })
  }),

  create: asyncHandler(async (req: Request, res: Response) => {
    const user = await userService.create(req.body)
    res.status(201).json({ status: 'success', data: user })
  }),
}
```

### Day 4-5: Mini Project
**Build: Task Management API**
- User registration
- Task CRUD operations
- Assign tasks to users
- Filter by status/priority
- Validation on all endpoints
- Proper error handling
- API documentation (comments or basic docs)

---

## Week 4: Advanced Topics

### Day 1: Rate Limiting & Security
**Topics:**
- Rate limiting
- Helmet security headers
- CORS configuration
- Input sanitization

```typescript
import rateLimit from 'express-rate-limit'

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // limit each IP to 100 requests per windowMs
  message: 'Too many requests, please try again later',
})

app.use('/api', limiter)
```

### Day 2: Logging
**Topics:**
- Winston logger
- Log levels
- Log formatting
- File and console transports

```typescript
import winston from 'winston'

export const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json()
  ),
  transports: [
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' }),
  ],
})

if (process.env.NODE_ENV !== 'production') {
  logger.add(new winston.transports.Console({
    format: winston.format.simple(),
  }))
}
```

### Day 3: Caching
**Topics:**
- Why cache?
- In-memory caching
- Redis basics
- Cache invalidation

### Day 4-5: Complete Blog API Project
Extend your task API or build a new Blog API:
- Posts, comments, categories, tags
- Image uploads to cloud storage
- Search functionality
- Pagination
- Full validation
- Rate limiting
- Logging

---

## Resources

### Documentation
- [Node.js Docs](https://nodejs.org/docs/)
- [Express.js Guide](https://expressjs.com/en/guide/routing.html)
- [Zod Documentation](https://zod.dev/)

### Practice
- [Node.js Best Practices](https://github.com/goldbergyoni/nodebestpractices)

---

## Checklist Before Moving On

- [ ] Understand Node.js fundamentals
- [ ] Can create Express servers
- [ ] Understand middleware patterns
- [ ] Can handle errors properly
- [ ] Can validate requests with Zod
- [ ] Understand controller/service separation
- [ ] Completed Task Management API
- [ ] Completed Blog API project

---

**Next:** [Databases](../02-databases/README.md)
