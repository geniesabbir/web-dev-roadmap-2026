# Day 4: API Documentation - OpenAPI & Swagger

## Introduction

Good API documentation is essential for developer adoption. OpenAPI (formerly Swagger) is the industry standard for describing REST APIs. Today, you'll learn how to create comprehensive API documentation that helps developers understand and integrate with your API quickly.

## Learning Objectives

By the end of this lesson, you will be able to:
- Understand OpenAPI specification structure
- Document endpoints, parameters, and responses
- Generate interactive API documentation
- Auto-generate documentation from code
- Keep documentation in sync with your API

---

## OpenAPI Specification Overview

OpenAPI is a standard format for describing REST APIs:

```yaml
# openapi.yaml
openapi: 3.0.3
info:
  title: My API
  description: A sample API for learning OpenAPI
  version: 1.0.0
  contact:
    name: API Support
    email: support@example.com

servers:
  - url: https://api.example.com/v1
    description: Production
  - url: http://localhost:3000/v1
    description: Development

paths:
  /users:
    get:
      summary: List all users
      responses:
        '200':
          description: Successful response

components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: string
```

---

## Setting Up Swagger UI

### Installation

```bash
npm install swagger-ui-express swagger-jsdoc
```

### Basic Setup

```javascript
// app.js
const express = require('express');
const swaggerUi = require('swagger-ui-express');
const swaggerJsdoc = require('swagger-jsdoc');

const app = express();

// Swagger configuration
const swaggerOptions = {
  definition: {
    openapi: '3.0.3',
    info: {
      title: 'E-Commerce API',
      version: '1.0.0',
      description: 'API for managing products, orders, and users',
      contact: {
        name: 'API Support',
        email: 'api@example.com'
      },
      license: {
        name: 'MIT',
        url: 'https://opensource.org/licenses/MIT'
      }
    },
    servers: [
      {
        url: 'http://localhost:3000/api/v1',
        description: 'Development server'
      },
      {
        url: 'https://api.example.com/v1',
        description: 'Production server'
      }
    ]
  },
  apis: ['./routes/*.js', './models/*.js']  // Files containing annotations
};

const swaggerSpec = swaggerJsdoc(swaggerOptions);

// Serve Swagger UI
app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerSpec, {
  customCss: '.swagger-ui .topbar { display: none }',
  customSiteTitle: 'My API Documentation'
}));

// Serve raw OpenAPI spec
app.get('/api-docs.json', (req, res) => {
  res.json(swaggerSpec);
});
```

---

## Documenting Endpoints with JSDoc

### Basic Endpoint Documentation

```javascript
// routes/users.js

/**
 * @swagger
 * /users:
 *   get:
 *     summary: Get all users
 *     description: Retrieve a list of all users with pagination
 *     tags:
 *       - Users
 *     parameters:
 *       - in: query
 *         name: page
 *         schema:
 *           type: integer
 *           default: 1
 *         description: Page number
 *       - in: query
 *         name: limit
 *         schema:
 *           type: integer
 *           default: 10
 *           maximum: 100
 *         description: Number of items per page
 *     responses:
 *       200:
 *         description: A list of users
 *         content:
 *           application/json:
 *             schema:
 *               type: object
 *               properties:
 *                 data:
 *                   type: array
 *                   items:
 *                     $ref: '#/components/schemas/User'
 *                 meta:
 *                   $ref: '#/components/schemas/PaginationMeta'
 *       401:
 *         description: Unauthorized
 *         content:
 *           application/json:
 *             schema:
 *               $ref: '#/components/schemas/Error'
 */
router.get('/', authenticate, userController.list);
```

### Path Parameters

```javascript
/**
 * @swagger
 * /users/{id}:
 *   get:
 *     summary: Get user by ID
 *     tags:
 *       - Users
 *     parameters:
 *       - in: path
 *         name: id
 *         required: true
 *         schema:
 *           type: string
 *           format: uuid
 *         description: User ID
 *     responses:
 *       200:
 *         description: User found
 *         content:
 *           application/json:
 *             schema:
 *               type: object
 *               properties:
 *                 data:
 *                   $ref: '#/components/schemas/User'
 *       404:
 *         description: User not found
 *         content:
 *           application/json:
 *             schema:
 *               $ref: '#/components/schemas/Error'
 */
router.get('/:id', authenticate, userController.get);
```

### Request Body

```javascript
/**
 * @swagger
 * /users:
 *   post:
 *     summary: Create a new user
 *     tags:
 *       - Users
 *     requestBody:
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             $ref: '#/components/schemas/CreateUserInput'
 *           example:
 *             name: John Doe
 *             email: john@example.com
 *             password: SecurePass123!
 *     responses:
 *       201:
 *         description: User created successfully
 *         content:
 *           application/json:
 *             schema:
 *               type: object
 *               properties:
 *                 data:
 *                   $ref: '#/components/schemas/User'
 *         headers:
 *           Location:
 *             schema:
 *               type: string
 *             description: URL of the created user
 *       400:
 *         description: Validation error
 *         content:
 *           application/json:
 *             schema:
 *               $ref: '#/components/schemas/ValidationError'
 *       409:
 *         description: Email already exists
 */
router.post('/', validateUser, userController.create);
```

---

## Defining Schemas

### Models/Components

```javascript
// models/schemas.js

/**
 * @swagger
 * components:
 *   schemas:
 *     User:
 *       type: object
 *       required:
 *         - id
 *         - email
 *         - name
 *       properties:
 *         id:
 *           type: string
 *           format: uuid
 *           description: Unique user identifier
 *           example: 550e8400-e29b-41d4-a716-446655440000
 *         email:
 *           type: string
 *           format: email
 *           description: User's email address
 *           example: john@example.com
 *         name:
 *           type: string
 *           minLength: 2
 *           maxLength: 100
 *           description: User's full name
 *           example: John Doe
 *         avatar:
 *           type: string
 *           format: uri
 *           nullable: true
 *           description: URL to user's avatar image
 *         role:
 *           type: string
 *           enum: [user, admin, moderator]
 *           default: user
 *           description: User's role in the system
 *         createdAt:
 *           type: string
 *           format: date-time
 *           description: Account creation timestamp
 *         updatedAt:
 *           type: string
 *           format: date-time
 *           description: Last update timestamp
 *
 *     CreateUserInput:
 *       type: object
 *       required:
 *         - email
 *         - name
 *         - password
 *       properties:
 *         email:
 *           type: string
 *           format: email
 *         name:
 *           type: string
 *           minLength: 2
 *         password:
 *           type: string
 *           minLength: 8
 *           description: Must contain uppercase, lowercase, number, and special char
 *
 *     UpdateUserInput:
 *       type: object
 *       properties:
 *         name:
 *           type: string
 *         avatar:
 *           type: string
 *           format: uri
 *
 *     PaginationMeta:
 *       type: object
 *       properties:
 *         page:
 *           type: integer
 *           example: 1
 *         limit:
 *           type: integer
 *           example: 10
 *         total:
 *           type: integer
 *           example: 100
 *         totalPages:
 *           type: integer
 *           example: 10
 *
 *     Error:
 *       type: object
 *       properties:
 *         error:
 *           type: object
 *           properties:
 *             code:
 *               type: string
 *               example: NOT_FOUND
 *             message:
 *               type: string
 *               example: Resource not found
 *
 *     ValidationError:
 *       type: object
 *       properties:
 *         error:
 *           type: object
 *           properties:
 *             code:
 *               type: string
 *               example: VALIDATION_ERROR
 *             message:
 *               type: string
 *             details:
 *               type: array
 *               items:
 *                 type: object
 *                 properties:
 *                   field:
 *                     type: string
 *                   message:
 *                     type: string
 */
```

---

## Authentication Documentation

### Security Schemes

```javascript
/**
 * @swagger
 * components:
 *   securitySchemes:
 *     bearerAuth:
 *       type: http
 *       scheme: bearer
 *       bearerFormat: JWT
 *       description: Enter your JWT token
 *
 *     apiKeyAuth:
 *       type: apiKey
 *       in: header
 *       name: X-API-Key
 *       description: API key for service-to-service calls
 *
 *     cookieAuth:
 *       type: apiKey
 *       in: cookie
 *       name: session
 */
```

### Applying Security

```javascript
/**
 * @swagger
 * /users:
 *   get:
 *     summary: List users
 *     security:
 *       - bearerAuth: []
 *     responses:
 *       200:
 *         description: Success
 *       401:
 *         description: Unauthorized - Invalid or missing token
 */

// Or apply globally in swagger options
const swaggerOptions = {
  definition: {
    // ...
    security: [
      { bearerAuth: [] }
    ]
  }
};
```

### Public Endpoints

```javascript
/**
 * @swagger
 * /auth/login:
 *   post:
 *     summary: User login
 *     security: []  # Empty array = no auth required
 *     requestBody:
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             type: object
 *             required:
 *               - email
 *               - password
 *             properties:
 *               email:
 *                 type: string
 *               password:
 *                 type: string
 *     responses:
 *       200:
 *         description: Login successful
 */
```

---

## Organizing Documentation with Tags

```javascript
/**
 * @swagger
 * tags:
 *   - name: Authentication
 *     description: User authentication endpoints
 *   - name: Users
 *     description: User management
 *   - name: Products
 *     description: Product catalog
 *   - name: Orders
 *     description: Order management
 *   - name: Admin
 *     description: Administrative operations
 */

// Then use in endpoints
/**
 * @swagger
 * /users:
 *   get:
 *     tags:
 *       - Users
 */
```

---

## Response Examples

```javascript
/**
 * @swagger
 * /products:
 *   get:
 *     summary: List products
 *     tags:
 *       - Products
 *     responses:
 *       200:
 *         description: Product list
 *         content:
 *           application/json:
 *             schema:
 *               type: object
 *               properties:
 *                 data:
 *                   type: array
 *                   items:
 *                     $ref: '#/components/schemas/Product'
 *             examples:
 *               electronics:
 *                 summary: Electronics category
 *                 value:
 *                   data:
 *                     - id: "1"
 *                       name: "Laptop"
 *                       price: 999.99
 *                       category: "electronics"
 *                     - id: "2"
 *                       name: "Headphones"
 *                       price: 149.99
 *                       category: "electronics"
 *               emptyList:
 *                 summary: No products found
 *                 value:
 *                   data: []
 *                   meta:
 *                     total: 0
 */
```

---

## File Upload Documentation

```javascript
/**
 * @swagger
 * /users/{id}/avatar:
 *   post:
 *     summary: Upload user avatar
 *     tags:
 *       - Users
 *     parameters:
 *       - in: path
 *         name: id
 *         required: true
 *         schema:
 *           type: string
 *     requestBody:
 *       required: true
 *       content:
 *         multipart/form-data:
 *           schema:
 *             type: object
 *             required:
 *               - avatar
 *             properties:
 *               avatar:
 *                 type: string
 *                 format: binary
 *                 description: Image file (max 5MB, jpg/png only)
 *     responses:
 *       200:
 *         description: Avatar uploaded
 *         content:
 *           application/json:
 *             schema:
 *               type: object
 *               properties:
 *                 avatarUrl:
 *                   type: string
 *                   format: uri
 *       413:
 *         description: File too large
 */
```

---

## Separate OpenAPI YAML File

For larger APIs, maintain a separate YAML file:

```yaml
# openapi.yaml
openapi: 3.0.3
info:
  title: E-Commerce API
  version: 1.0.0
  description: |
    # Introduction
    Welcome to the E-Commerce API documentation.

    ## Authentication
    Most endpoints require authentication via JWT token.
    Include the token in the Authorization header:
    ```
    Authorization: Bearer <your-token>
    ```

    ## Rate Limiting
    - 100 requests per minute for authenticated users
    - 20 requests per minute for anonymous users

    ## Errors
    All errors follow this format:
    ```json
    {
      "error": {
        "code": "ERROR_CODE",
        "message": "Human readable message"
      }
    }
    ```

servers:
  - url: https://api.example.com/v1
    description: Production
  - url: https://staging-api.example.com/v1
    description: Staging
  - url: http://localhost:3000/v1
    description: Local Development

tags:
  - name: Authentication
    description: Login, register, and token management
  - name: Users
    description: User profile and management
  - name: Products
    description: Product catalog operations
  - name: Orders
    description: Order processing
  - name: Cart
    description: Shopping cart management

paths:
  /auth/register:
    $ref: './paths/auth.yaml#/register'
  /auth/login:
    $ref: './paths/auth.yaml#/login'
  /users:
    $ref: './paths/users.yaml#/collection'
  /users/{id}:
    $ref: './paths/users.yaml#/single'
  /products:
    $ref: './paths/products.yaml#/collection'

components:
  schemas:
    $ref: './components/schemas.yaml'
  securitySchemes:
    $ref: './components/security.yaml'
  responses:
    $ref: './components/responses.yaml'

security:
  - bearerAuth: []
```

### Loading YAML File

```javascript
const YAML = require('yamljs');
const swaggerDocument = YAML.load('./openapi.yaml');

app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerDocument));
```

---

## Auto-Generate from TypeScript

Using `tsoa` for TypeScript-first documentation:

```bash
npm install tsoa
```

```typescript
// controllers/userController.ts
import { Controller, Get, Post, Body, Path, Route, Tags, Security, Response } from 'tsoa';
import { User, CreateUserInput } from '../models/user';

@Route('users')
@Tags('Users')
export class UserController extends Controller {
  /**
   * Get all users with pagination
   * @param page Page number
   * @param limit Items per page
   */
  @Get()
  @Security('bearerAuth')
  public async getUsers(
    @Query() page: number = 1,
    @Query() limit: number = 10
  ): Promise<{ data: User[]; meta: PaginationMeta }> {
    // Implementation
  }

  /**
   * Get a specific user by ID
   * @param id The user's unique identifier
   */
  @Get('{id}')
  @Security('bearerAuth')
  @Response<NotFoundError>(404, 'User not found')
  public async getUser(@Path() id: string): Promise<User> {
    // Implementation
  }

  /**
   * Create a new user
   */
  @Post()
  @Response<ValidationError>(400, 'Validation failed')
  @Response<ConflictError>(409, 'Email already exists')
  public async createUser(@Body() body: CreateUserInput): Promise<User> {
    // Implementation
  }
}
```

---

## Interactive Features

### Try It Out Configuration

```javascript
app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerSpec, {
  swaggerOptions: {
    persistAuthorization: true,  // Remember auth token
    tryItOutEnabled: true,       // Enable "Try it out" by default
    displayRequestDuration: true // Show request duration
  }
}));
```

### Custom CSS

```javascript
const customCss = `
  .swagger-ui .topbar { display: none; }
  .swagger-ui .info { margin-bottom: 20px; }
  .swagger-ui .scheme-container { background: #f5f5f5; padding: 15px; }
`;

app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerSpec, {
  customCss
}));
```

---

## Validation Against Spec

```javascript
// Validate requests match OpenAPI spec
const OpenApiValidator = require('express-openapi-validator');

app.use(
  OpenApiValidator.middleware({
    apiSpec: './openapi.yaml',
    validateRequests: true,
    validateResponses: true  // Also validate responses
  })
);

// Error handler for validation failures
app.use((err, req, res, next) => {
  if (err.status === 400) {
    return res.status(400).json({
      error: {
        code: 'VALIDATION_ERROR',
        message: err.message,
        details: err.errors
      }
    });
  }
  next(err);
});
```

---

## Generating Client SDKs

```bash
# Generate TypeScript client
npx openapi-generator-cli generate \
  -i http://localhost:3000/api-docs.json \
  -g typescript-axios \
  -o ./sdk/typescript

# Generate Python client
npx openapi-generator-cli generate \
  -i http://localhost:3000/api-docs.json \
  -g python \
  -o ./sdk/python
```

---

## Documentation Best Practices

### 1. Write Descriptive Summaries

```javascript
/**
 * @swagger
 * /orders/{id}/cancel:
 *   post:
 *     summary: Cancel an order
 *     description: |
 *       Cancels an existing order if it hasn't been shipped yet.
 *
 *       **Restrictions:**
 *       - Cannot cancel shipped orders
 *       - Cannot cancel orders older than 24 hours
 *       - Refund will be processed within 5-7 business days
 */
```

### 2. Include All Possible Responses

```javascript
/**
 * @swagger
 * responses:
 *   200:
 *     description: Order cancelled successfully
 *   400:
 *     description: Order cannot be cancelled (already shipped)
 *   401:
 *     description: Authentication required
 *   403:
 *     description: Not authorized to cancel this order
 *   404:
 *     description: Order not found
 *   409:
 *     description: Order already cancelled
 */
```

### 3. Use Examples

```javascript
/**
 * @swagger
 * components:
 *   examples:
 *     UserExample:
 *       value:
 *         id: "550e8400-e29b-41d4-a716-446655440000"
 *         name: "John Doe"
 *         email: "john@example.com"
 *         role: "user"
 *         createdAt: "2024-01-15T10:30:00Z"
 */
```

---

## Practice Exercise

Document a complete API:

1. **Set up Swagger UI** in your Express app
2. **Document these endpoints**:
   - `POST /auth/register`
   - `POST /auth/login`
   - `GET /products` (with pagination, filtering)
   - `GET /products/:id`
   - `POST /products` (admin only)
   - `POST /orders`
3. **Define schemas** for all request/response bodies
4. **Add authentication** documentation
5. **Include examples** and error responses

---

## Key Takeaways

1. **OpenAPI is the standard** - Use it for REST API documentation
2. **Document while coding** - JSDoc comments keep docs in sync
3. **Include examples** - Real examples help developers understand
4. **Document errors** - Show all possible error responses
5. **Keep it updated** - Outdated docs are worse than none
6. **Enable interactivity** - Let developers test endpoints

---

## What's Next?

Tomorrow, we'll explore **GraphQL** - an alternative to REST that gives clients more control over the data they receive.
