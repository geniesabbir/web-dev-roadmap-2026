# Phase 3: Backend Development

**Duration:** 10-12 weeks
**Goal:** Build secure, scalable backend systems and APIs

## Overview

This phase teaches you to build the server-side of applications. You'll learn to create APIs, work with databases, implement authentication, and handle data securely.

## Prerequisites

Before starting this phase, ensure you:
- Completed Phase 1 and 2
- Understand JavaScript/TypeScript deeply
- Know how frontend apps consume APIs
- Comfortable with async programming

## Sections

### 1. [Node.js & Express](./01-nodejs-express/README.md) (3-4 weeks)
- Node.js fundamentals
- Express.js framework
- Middleware
- REST API design
- Error handling
- File uploads
- Validation

### 2. [Databases](./02-databases/README.md) (3 weeks)
- SQL fundamentals (PostgreSQL)
- NoSQL basics (MongoDB)
- ORMs (Prisma, Drizzle)
- Database design
- Migrations
- Relationships
- Query optimization

### 3. [Authentication & Security](./03-auth-security/README.md) (2 weeks)
- Password hashing
- JWT tokens
- Session management
- OAuth 2.0
- Rate limiting
- Input sanitization
- OWASP Top 10

### 4. [API Design](./04-api-design/README.md) (1-2 weeks)
- RESTful best practices
- API versioning
- Pagination
- Filtering and sorting
- Documentation (OpenAPI)
- GraphQL introduction

### 5. [Testing](./05-testing/README.md) (1 week)
- Unit testing
- Integration testing
- API testing
- Test databases
- Mocking

## Phase 3 Projects

### Project 1: RESTful Blog API
- User registration/login
- CRUD for posts and comments
- Role-based access (admin, author, reader)
- Image uploads
- Pagination and filtering
- Full test coverage

### Project 2: E-commerce Backend
- Product management
- Shopping cart
- Order processing
- Payment integration (Stripe)
- Inventory tracking
- Webhooks

## Tech Stack for This Phase

```
Node.js 20+
Express.js / Fastify
TypeScript
PostgreSQL
Prisma ORM
Redis (caching)
JWT (auth)
Zod (validation)
Jest / Vitest (testing)
```

## Checklist

- [ ] Complete Node.js & Express
- [ ] Complete Databases
- [ ] Complete Authentication & Security
- [ ] Complete API Design
- [ ] Complete Testing
- [ ] Build Blog API
- [ ] Build E-commerce Backend

## Tips for Success

1. **Think about data first** - Design your database before coding
2. **Security is not optional** - Always validate, sanitize, authenticate
3. **Test your APIs** - Use Postman/Insomnia during development
4. **Handle errors properly** - Users should get meaningful errors
5. **Document as you go** - Future you will thank you

---

**Next:** [Phase 4: DevOps & Deployment](../phase-4-devops/README.md)
