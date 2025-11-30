# Day 5: Introduction to GraphQL

## Introduction

GraphQL is a query language for APIs that gives clients the power to ask for exactly what they need. Unlike REST where the server decides what data to return, GraphQL lets clients specify the exact data they want. Today, you'll learn GraphQL fundamentals and how to build a GraphQL API with Node.js.

## Learning Objectives

By the end of this lesson, you will be able to:
- Understand GraphQL concepts and benefits
- Write GraphQL schemas and types
- Create queries and mutations
- Implement resolvers
- Compare GraphQL with REST

---

## GraphQL vs REST

### The REST Problem

```javascript
// REST: Multiple requests for related data
GET /users/123           // Get user
GET /users/123/posts     // Get user's posts
GET /posts/1/comments    // Get post comments
GET /users/456           // Get comment author

// Over-fetching: Getting more data than needed
GET /users/123
// Returns: { id, name, email, phone, address, preferences, ... }
// But you only needed: { name }

// Under-fetching: Not getting enough data
GET /posts
// Returns: { id, title, authorId }
// Now need another request to get author names
```

### The GraphQL Solution

```graphql
# One request, exact data needed
query {
  user(id: "123") {
    name
    posts {
      title
      comments {
        text
        author {
          name
        }
      }
    }
  }
}
```

---

## Core GraphQL Concepts

### Schema Definition Language (SDL)

```graphql
# Types define the shape of your data
type User {
  id: ID!
  name: String!
  email: String!
  posts: [Post!]!
}

type Post {
  id: ID!
  title: String!
  content: String!
  author: User!
  comments: [Comment!]!
  createdAt: String!
}

type Comment {
  id: ID!
  text: String!
  author: User!
  post: Post!
}

# Query type defines read operations
type Query {
  users: [User!]!
  user(id: ID!): User
  posts: [Post!]!
  post(id: ID!): Post
}

# Mutation type defines write operations
type Mutation {
  createUser(input: CreateUserInput!): User!
  createPost(input: CreatePostInput!): Post!
  deletePost(id: ID!): Boolean!
}

# Input types for mutations
input CreateUserInput {
  name: String!
  email: String!
  password: String!
}

input CreatePostInput {
  title: String!
  content: String!
}
```

### Type System

```graphql
# Scalar Types
String    # Text
Int       # 32-bit integer
Float     # Double-precision floating point
Boolean   # true or false
ID        # Unique identifier

# Type Modifiers
String    # Nullable string
String!   # Non-null string
[String]  # Nullable list of nullable strings
[String!] # Nullable list of non-null strings
[String!]!# Non-null list of non-null strings

# Custom Scalars
scalar DateTime
scalar Email
scalar URL
```

---

## Setting Up GraphQL Server

### Installation

```bash
npm install @apollo/server graphql
```

### Basic Server Setup

```javascript
// server.js
const { ApolloServer } = require('@apollo/server');
const { startStandaloneServer } = require('@apollo/server/standalone');

// Type definitions (schema)
const typeDefs = `#graphql
  type User {
    id: ID!
    name: String!
    email: String!
  }

  type Query {
    users: [User!]!
    user(id: ID!): User
  }

  type Mutation {
    createUser(name: String!, email: String!): User!
  }
`;

// Sample data
const users = [
  { id: '1', name: 'John Doe', email: 'john@example.com' },
  { id: '2', name: 'Jane Smith', email: 'jane@example.com' }
];

// Resolvers
const resolvers = {
  Query: {
    users: () => users,
    user: (_, { id }) => users.find(u => u.id === id)
  },
  Mutation: {
    createUser: (_, { name, email }) => {
      const user = {
        id: String(users.length + 1),
        name,
        email
      };
      users.push(user);
      return user;
    }
  }
};

// Create and start server
const server = new ApolloServer({ typeDefs, resolvers });

startStandaloneServer(server, { listen: { port: 4000 } })
  .then(({ url }) => {
    console.log(`🚀 Server ready at ${url}`);
  });
```

---

## Writing Queries

### Basic Queries

```graphql
# Get all users
query {
  users {
    id
    name
  }
}

# Get specific user
query {
  user(id: "1") {
    id
    name
    email
  }
}

# Named query with variables
query GetUser($userId: ID!) {
  user(id: $userId) {
    id
    name
    email
  }
}

# Variables (sent separately)
{
  "userId": "1"
}
```

### Nested Queries

```graphql
query {
  user(id: "1") {
    name
    posts {
      title
      comments {
        text
        author {
          name
        }
      }
    }
  }
}
```

### Query Aliases

```graphql
query {
  firstUser: user(id: "1") {
    name
  }
  secondUser: user(id: "2") {
    name
  }
}
```

### Fragments

```graphql
# Reusable field selections
fragment UserFields on User {
  id
  name
  email
}

query {
  user(id: "1") {
    ...UserFields
    posts {
      title
    }
  }
  users {
    ...UserFields
  }
}
```

---

## Writing Mutations

### Basic Mutations

```graphql
# Create user
mutation {
  createUser(name: "Bob", email: "bob@example.com") {
    id
    name
  }
}

# With input type
mutation CreateUser($input: CreateUserInput!) {
  createUser(input: $input) {
    id
    name
    email
  }
}

# Variables
{
  "input": {
    "name": "Bob",
    "email": "bob@example.com",
    "password": "secret123"
  }
}

# Update mutation
mutation {
  updateUser(id: "1", input: { name: "John Updated" }) {
    id
    name
  }
}

# Delete mutation
mutation {
  deleteUser(id: "1")
}
```

---

## Resolvers In-Depth

### Resolver Function Signature

```javascript
// fieldName: (parent, args, context, info) => result

const resolvers = {
  Query: {
    // Root resolvers receive null as parent
    users: (parent, args, context, info) => {
      // parent: null (root query)
      // args: query arguments
      // context: shared data (auth, db connections)
      // info: query AST information
      return context.db.users.findMany();
    },

    user: (_, { id }, { db }) => {
      return db.users.findUnique({ where: { id } });
    }
  },

  // Type resolvers for relationships
  User: {
    // Parent is the user object
    posts: (user, _, { db }) => {
      return db.posts.findMany({
        where: { authorId: user.id }
      });
    },

    // Computed field
    fullName: (user) => {
      return `${user.firstName} ${user.lastName}`;
    }
  },

  Post: {
    author: (post, _, { db }) => {
      return db.users.findUnique({
        where: { id: post.authorId }
      });
    },

    comments: (post, _, { db }) => {
      return db.comments.findMany({
        where: { postId: post.id }
      });
    }
  }
};
```

### Context Setup

```javascript
const server = new ApolloServer({ typeDefs, resolvers });

startStandaloneServer(server, {
  context: async ({ req }) => {
    // Get auth token from headers
    const token = req.headers.authorization?.replace('Bearer ', '');

    // Get user from token
    const user = token ? await verifyToken(token) : null;

    return {
      db: prisma,
      user,
      loaders: createDataLoaders()  // For N+1 prevention
    };
  }
});
```

---

## Complete Example: Blog API

### Schema

```javascript
const typeDefs = `#graphql
  type User {
    id: ID!
    name: String!
    email: String!
    avatar: String
    posts: [Post!]!
    comments: [Comment!]!
    createdAt: String!
  }

  type Post {
    id: ID!
    title: String!
    content: String!
    published: Boolean!
    author: User!
    comments: [Comment!]!
    createdAt: String!
    updatedAt: String!
  }

  type Comment {
    id: ID!
    text: String!
    author: User!
    post: Post!
    createdAt: String!
  }

  type Query {
    # User queries
    me: User
    user(id: ID!): User
    users(limit: Int, offset: Int): [User!]!

    # Post queries
    posts(
      published: Boolean
      authorId: ID
      limit: Int
      offset: Int
    ): [Post!]!
    post(id: ID!): Post

    # Search
    search(term: String!): [SearchResult!]!
  }

  union SearchResult = User | Post

  type Mutation {
    # Auth
    register(input: RegisterInput!): AuthPayload!
    login(email: String!, password: String!): AuthPayload!

    # Posts
    createPost(input: CreatePostInput!): Post!
    updatePost(id: ID!, input: UpdatePostInput!): Post!
    deletePost(id: ID!): Boolean!
    publishPost(id: ID!): Post!

    # Comments
    createComment(postId: ID!, text: String!): Comment!
    deleteComment(id: ID!): Boolean!
  }

  type AuthPayload {
    token: String!
    user: User!
  }

  input RegisterInput {
    name: String!
    email: String!
    password: String!
  }

  input CreatePostInput {
    title: String!
    content: String!
  }

  input UpdatePostInput {
    title: String
    content: String
  }
`;
```

### Resolvers

```javascript
const bcrypt = require('bcrypt');
const jwt = require('jsonwebtoken');

const resolvers = {
  Query: {
    me: (_, __, { user, db }) => {
      if (!user) return null;
      return db.user.findUnique({ where: { id: user.id } });
    },

    user: (_, { id }, { db }) => {
      return db.user.findUnique({ where: { id } });
    },

    users: (_, { limit = 10, offset = 0 }, { db }) => {
      return db.user.findMany({ take: limit, skip: offset });
    },

    posts: (_, { published, authorId, limit = 10, offset = 0 }, { db }) => {
      const where = {};
      if (published !== undefined) where.published = published;
      if (authorId) where.authorId = authorId;

      return db.post.findMany({
        where,
        take: limit,
        skip: offset,
        orderBy: { createdAt: 'desc' }
      });
    },

    post: (_, { id }, { db }) => {
      return db.post.findUnique({ where: { id } });
    },

    search: async (_, { term }, { db }) => {
      const users = await db.user.findMany({
        where: { name: { contains: term, mode: 'insensitive' } }
      });

      const posts = await db.post.findMany({
        where: {
          OR: [
            { title: { contains: term, mode: 'insensitive' } },
            { content: { contains: term, mode: 'insensitive' } }
          ]
        }
      });

      return [...users, ...posts];
    }
  },

  Mutation: {
    register: async (_, { input }, { db }) => {
      const { name, email, password } = input;

      const existing = await db.user.findUnique({ where: { email } });
      if (existing) {
        throw new Error('Email already registered');
      }

      const hashedPassword = await bcrypt.hash(password, 12);

      const user = await db.user.create({
        data: { name, email, password: hashedPassword }
      });

      const token = jwt.sign({ userId: user.id }, process.env.JWT_SECRET);

      return { token, user };
    },

    login: async (_, { email, password }, { db }) => {
      const user = await db.user.findUnique({ where: { email } });
      if (!user) {
        throw new Error('Invalid credentials');
      }

      const valid = await bcrypt.compare(password, user.password);
      if (!valid) {
        throw new Error('Invalid credentials');
      }

      const token = jwt.sign({ userId: user.id }, process.env.JWT_SECRET);

      return { token, user };
    },

    createPost: async (_, { input }, { user, db }) => {
      if (!user) throw new Error('Authentication required');

      return db.post.create({
        data: {
          ...input,
          authorId: user.id,
          published: false
        }
      });
    },

    updatePost: async (_, { id, input }, { user, db }) => {
      if (!user) throw new Error('Authentication required');

      const post = await db.post.findUnique({ where: { id } });
      if (!post || post.authorId !== user.id) {
        throw new Error('Post not found or unauthorized');
      }

      return db.post.update({
        where: { id },
        data: input
      });
    },

    publishPost: async (_, { id }, { user, db }) => {
      if (!user) throw new Error('Authentication required');

      const post = await db.post.findUnique({ where: { id } });
      if (!post || post.authorId !== user.id) {
        throw new Error('Post not found or unauthorized');
      }

      return db.post.update({
        where: { id },
        data: { published: true }
      });
    },

    deletePost: async (_, { id }, { user, db }) => {
      if (!user) throw new Error('Authentication required');

      const post = await db.post.findUnique({ where: { id } });
      if (!post || post.authorId !== user.id) {
        throw new Error('Post not found or unauthorized');
      }

      await db.post.delete({ where: { id } });
      return true;
    },

    createComment: async (_, { postId, text }, { user, db }) => {
      if (!user) throw new Error('Authentication required');

      return db.comment.create({
        data: {
          text,
          postId,
          authorId: user.id
        }
      });
    }
  },

  // Type resolvers
  User: {
    posts: (user, _, { db }) => {
      return db.post.findMany({ where: { authorId: user.id } });
    },
    comments: (user, _, { db }) => {
      return db.comment.findMany({ where: { authorId: user.id } });
    }
  },

  Post: {
    author: (post, _, { db }) => {
      return db.user.findUnique({ where: { id: post.authorId } });
    },
    comments: (post, _, { db }) => {
      return db.comment.findMany({ where: { postId: post.id } });
    }
  },

  Comment: {
    author: (comment, _, { db }) => {
      return db.user.findUnique({ where: { id: comment.authorId } });
    },
    post: (comment, _, { db }) => {
      return db.post.findUnique({ where: { id: comment.postId } });
    }
  },

  // Union type resolver
  SearchResult: {
    __resolveType(obj) {
      if (obj.email) return 'User';
      if (obj.title) return 'Post';
      return null;
    }
  }
};
```

---

## N+1 Problem & DataLoader

### The Problem

```graphql
query {
  posts {    # 1 query for posts
    author { # N queries for authors (one per post)
      name
    }
  }
}
```

### Solution: DataLoader

```javascript
const DataLoader = require('dataloader');

// Create loaders
const createLoaders = (db) => ({
  userLoader: new DataLoader(async (userIds) => {
    const users = await db.user.findMany({
      where: { id: { in: userIds } }
    });

    // Return in same order as requested IDs
    const userMap = new Map(users.map(u => [u.id, u]));
    return userIds.map(id => userMap.get(id));
  })
});

// In context
startStandaloneServer(server, {
  context: async ({ req }) => ({
    db: prisma,
    loaders: createLoaders(prisma)
  })
});

// In resolver
const resolvers = {
  Post: {
    author: (post, _, { loaders }) => {
      return loaders.userLoader.load(post.authorId);
    }
  }
};
```

---

## Error Handling

```javascript
const { GraphQLError } = require('graphql');

const resolvers = {
  Mutation: {
    createPost: async (_, { input }, { user }) => {
      if (!user) {
        throw new GraphQLError('Authentication required', {
          extensions: { code: 'UNAUTHENTICATED' }
        });
      }

      if (!input.title.trim()) {
        throw new GraphQLError('Title cannot be empty', {
          extensions: {
            code: 'BAD_USER_INPUT',
            field: 'title'
          }
        });
      }

      // ...
    }
  }
};
```

---

## GraphQL vs REST Summary

| Feature | REST | GraphQL |
|---------|------|---------|
| Data fetching | Multiple endpoints | Single endpoint |
| Over-fetching | Common | Solved |
| Under-fetching | Common | Solved |
| Versioning | URL versioning | Schema evolution |
| Caching | HTTP caching | Requires client libs |
| File upload | Native | Requires spec extension |
| Learning curve | Lower | Higher |

### When to Use GraphQL

- Complex data requirements
- Multiple client types (web, mobile)
- Rapidly evolving APIs
- Real-time subscriptions needed

### When to Use REST

- Simple CRUD operations
- File downloads/uploads
- Caching is critical
- Team unfamiliar with GraphQL

---

## Practice Exercise

Build a GraphQL API for a task management app:

1. **Define types**: User, Task, Project
2. **Create queries**: Get tasks, filter by status
3. **Create mutations**: Create/update/delete tasks
4. **Add authentication**: Protect mutations
5. **Implement DataLoader**: Prevent N+1 queries

---

## Key Takeaways

1. **Clients define data needs** - No over/under-fetching
2. **Strong typing** - Schema defines contract
3. **Single endpoint** - All operations through one URL
4. **Introspection** - Self-documenting API
5. **Use DataLoader** - Prevent N+1 query problems
6. **Consider trade-offs** - GraphQL isn't always the answer

---

## Congratulations!

You've completed the API Design section! You now understand:
- REST API principles and best practices
- Pagination, filtering, and sorting
- API versioning strategies
- API documentation with OpenAPI/Swagger
- GraphQL fundamentals

In the next section, we'll explore **Backend Testing** - how to ensure your API works correctly through unit and integration tests.
