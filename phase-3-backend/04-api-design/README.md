# API Design

**Duration:** 1-2 weeks

## Learning Objectives

By the end of this section, you will:
- Design RESTful APIs following best practices
- Implement pagination, filtering, and sorting
- Version your APIs properly
- Document APIs with OpenAPI/Swagger
- Understand GraphQL basics

---

## Day 1: REST Principles

### RESTful Design
- Resources as nouns, not verbs
- HTTP methods for actions
- Stateless communication
- HATEOAS (Hypermedia)

### URL Design
```
# Good
GET    /users              # List users
GET    /users/123          # Get user 123
POST   /users              # Create user
PUT    /users/123          # Replace user 123
PATCH  /users/123          # Update user 123
DELETE /users/123          # Delete user 123

GET    /users/123/posts    # Get posts by user 123
POST   /users/123/posts    # Create post for user 123

# Bad
GET    /getUsers
POST   /createUser
GET    /user/delete/123
POST   /users/123/createPost
```

### HTTP Status Codes
```typescript
// Success
200 OK           - Successful GET, PUT, PATCH
201 Created      - Successful POST (include Location header)
204 No Content   - Successful DELETE

// Client Errors
400 Bad Request  - Invalid input/validation error
401 Unauthorized - Missing or invalid authentication
403 Forbidden    - Authenticated but not authorized
404 Not Found    - Resource doesn't exist
409 Conflict     - Resource conflict (duplicate)
422 Unprocessable Entity - Valid JSON but semantic errors
429 Too Many Requests - Rate limited

// Server Errors
500 Internal Server Error - Unexpected server error
503 Service Unavailable   - Server temporarily down
```

### Response Format
```typescript
// Success response
{
  "status": "success",
  "data": {
    "user": {
      "id": "123",
      "email": "john@example.com",
      "name": "John Doe"
    }
  }
}

// List response with pagination
{
  "status": "success",
  "data": {
    "users": [...],
    "pagination": {
      "total": 100,
      "page": 1,
      "limit": 10,
      "pages": 10
    }
  }
}

// Error response
{
  "status": "error",
  "message": "Validation failed",
  "errors": [
    { "field": "email", "message": "Invalid email format" }
  ]
}
```

---

## Day 2: Pagination, Filtering, Sorting

### Pagination
```typescript
// Offset-based pagination
// GET /posts?page=2&limit=10

interface PaginationQuery {
  page?: number
  limit?: number
}

async function getPosts(query: PaginationQuery) {
  const page = Math.max(1, query.page || 1)
  const limit = Math.min(100, Math.max(1, query.limit || 10))
  const skip = (page - 1) * limit

  const [posts, total] = await Promise.all([
    prisma.post.findMany({ skip, take: limit }),
    prisma.post.count(),
  ])

  return {
    data: posts,
    pagination: {
      total,
      page,
      limit,
      pages: Math.ceil(total / limit),
      hasNext: page * limit < total,
      hasPrev: page > 1,
    },
  }
}

// Cursor-based pagination (better for large datasets)
// GET /posts?cursor=abc123&limit=10

async function getPostsCursor(cursor?: string, limit = 10) {
  const posts = await prisma.post.findMany({
    take: limit + 1, // Fetch one extra to check if there's more
    ...(cursor && {
      skip: 1,
      cursor: { id: cursor },
    }),
    orderBy: { createdAt: 'desc' },
  })

  const hasMore = posts.length > limit
  const data = hasMore ? posts.slice(0, -1) : posts

  return {
    data,
    nextCursor: hasMore ? data[data.length - 1].id : null,
  }
}
```

### Filtering
```typescript
// GET /posts?status=published&authorId=123&createdAfter=2024-01-01

interface PostFilters {
  status?: 'draft' | 'published'
  authorId?: string
  createdAfter?: string
  createdBefore?: string
  search?: string
}

function buildPostFilters(query: PostFilters) {
  const where: Prisma.PostWhereInput = {}

  if (query.status) {
    where.published = query.status === 'published'
  }

  if (query.authorId) {
    where.authorId = query.authorId
  }

  if (query.createdAfter || query.createdBefore) {
    where.createdAt = {}
    if (query.createdAfter) {
      where.createdAt.gte = new Date(query.createdAfter)
    }
    if (query.createdBefore) {
      where.createdAt.lte = new Date(query.createdBefore)
    }
  }

  if (query.search) {
    where.OR = [
      { title: { contains: query.search, mode: 'insensitive' } },
      { content: { contains: query.search, mode: 'insensitive' } },
    ]
  }

  return where
}
```

### Sorting
```typescript
// GET /posts?sort=createdAt&order=desc
// GET /posts?sort=-createdAt,title (prefix - for desc)

function buildSort(sort?: string) {
  if (!sort) return { createdAt: 'desc' as const }

  const orderBy: Prisma.PostOrderByWithRelationInput[] = []
  const fields = sort.split(',')

  for (const field of fields) {
    const isDesc = field.startsWith('-')
    const fieldName = isDesc ? field.slice(1) : field
    orderBy.push({ [fieldName]: isDesc ? 'desc' : 'asc' })
  }

  return orderBy
}
```

---

## Day 3: API Versioning

### Strategies

**1. URL Path Versioning (Recommended)**
```
/api/v1/users
/api/v2/users
```

**2. Header Versioning**
```
GET /api/users
Accept: application/vnd.api+json; version=1
```

**3. Query Parameter**
```
/api/users?version=1
```

### Implementation
```typescript
// src/routes/v1/index.ts
import { Router } from 'express'
import userRoutes from './userRoutes'
import postRoutes from './postRoutes'

const v1Router = Router()

v1Router.use('/users', userRoutes)
v1Router.use('/posts', postRoutes)

export default v1Router

// src/routes/v2/index.ts
import { Router } from 'express'
import userRoutes from './userRoutes' // Updated version

const v2Router = Router()

v2Router.use('/users', userRoutes)

export default v2Router

// src/app.ts
import v1Router from './routes/v1'
import v2Router from './routes/v2'

app.use('/api/v1', v1Router)
app.use('/api/v2', v2Router)
```

---

## Day 4: API Documentation

### OpenAPI/Swagger
```bash
npm install swagger-ui-express swagger-jsdoc
```

```typescript
// src/config/swagger.ts
import swaggerJsdoc from 'swagger-jsdoc'

const options: swaggerJsdoc.Options = {
  definition: {
    openapi: '3.0.0',
    info: {
      title: 'My API',
      version: '1.0.0',
      description: 'API documentation',
    },
    servers: [
      { url: 'http://localhost:3000/api/v1', description: 'Development' },
      { url: 'https://api.myapp.com/v1', description: 'Production' },
    ],
    components: {
      securitySchemes: {
        bearerAuth: {
          type: 'http',
          scheme: 'bearer',
          bearerFormat: 'JWT',
        },
      },
    },
  },
  apis: ['./src/routes/**/*.ts'],
}

export const swaggerSpec = swaggerJsdoc(options)

// src/app.ts
import swaggerUi from 'swagger-ui-express'
import { swaggerSpec } from './config/swagger'

app.use('/docs', swaggerUi.serve, swaggerUi.setup(swaggerSpec))
```

### Route Documentation
```typescript
/**
 * @openapi
 * /users:
 *   get:
 *     summary: Get all users
 *     tags: [Users]
 *     security:
 *       - bearerAuth: []
 *     parameters:
 *       - in: query
 *         name: page
 *         schema:
 *           type: integer
 *           default: 1
 *       - in: query
 *         name: limit
 *         schema:
 *           type: integer
 *           default: 10
 *     responses:
 *       200:
 *         description: List of users
 *         content:
 *           application/json:
 *             schema:
 *               type: object
 *               properties:
 *                 status:
 *                   type: string
 *                   example: success
 *                 data:
 *                   type: array
 *                   items:
 *                     $ref: '#/components/schemas/User'
 *       401:
 *         description: Unauthorized
 */
router.get('/', authenticate, userController.getAll)

/**
 * @openapi
 * components:
 *   schemas:
 *     User:
 *       type: object
 *       properties:
 *         id:
 *           type: string
 *           format: uuid
 *         email:
 *           type: string
 *           format: email
 *         name:
 *           type: string
 *         createdAt:
 *           type: string
 *           format: date-time
 */
```

---

## Day 5: GraphQL Introduction

### Why GraphQL?
- Client specifies exact data needed
- Single endpoint
- Strongly typed
- Real-time with subscriptions

### Basic Setup
```bash
npm install @apollo/server graphql
```

```typescript
// src/graphql/schema.ts
export const typeDefs = `#graphql
  type User {
    id: ID!
    email: String!
    name: String!
    posts: [Post!]!
    createdAt: String!
  }

  type Post {
    id: ID!
    title: String!
    content: String
    published: Boolean!
    author: User!
    createdAt: String!
  }

  type Query {
    users: [User!]!
    user(id: ID!): User
    posts(published: Boolean): [Post!]!
    post(id: ID!): Post
  }

  type Mutation {
    createUser(email: String!, name: String!, password: String!): User!
    createPost(title: String!, content: String, authorId: ID!): Post!
    publishPost(id: ID!): Post!
  }
`

// src/graphql/resolvers.ts
export const resolvers = {
  Query: {
    users: () => prisma.user.findMany(),
    user: (_: unknown, { id }: { id: string }) =>
      prisma.user.findUnique({ where: { id } }),
    posts: (_: unknown, { published }: { published?: boolean }) =>
      prisma.post.findMany({ where: { published } }),
  },
  User: {
    posts: (parent: { id: string }) =>
      prisma.post.findMany({ where: { authorId: parent.id } }),
  },
  Post: {
    author: (parent: { authorId: string }) =>
      prisma.user.findUnique({ where: { id: parent.authorId } }),
  },
  Mutation: {
    createPost: (_: unknown, args: { title: string; content?: string; authorId: string }) =>
      prisma.post.create({ data: args }),
  },
}
```

---

## API Design Checklist

### URLs
- [ ] Use nouns, not verbs
- [ ] Plural resource names
- [ ] Nested resources for relationships
- [ ] Consistent naming convention

### Responses
- [ ] Consistent response format
- [ ] Appropriate status codes
- [ ] Meaningful error messages
- [ ] Include pagination metadata

### Features
- [ ] Pagination on list endpoints
- [ ] Filtering support
- [ ] Sorting support
- [ ] Field selection (sparse fieldsets)

### Documentation
- [ ] OpenAPI/Swagger docs
- [ ] Authentication documented
- [ ] Request/response examples
- [ ] Error codes documented

---

## Resources

### Documentation
- [REST API Design](https://restfulapi.net/)
- [OpenAPI Specification](https://swagger.io/specification/)
- [GraphQL Docs](https://graphql.org/learn/)

---

## Checklist Before Moving On

- [ ] Can design RESTful APIs
- [ ] Implement pagination properly
- [ ] Implement filtering and sorting
- [ ] Version APIs correctly
- [ ] Document APIs with Swagger
- [ ] Understand GraphQL basics

---

**Next:** [Testing](../05-testing/README.md)
