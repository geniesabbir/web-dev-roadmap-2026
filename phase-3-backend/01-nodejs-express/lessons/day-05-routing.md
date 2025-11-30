# Day 5: Advanced Express Routing

## Introduction

Effective routing is the backbone of any API or web application. This lesson covers advanced routing techniques including nested routes, route parameters, query handling, and building RESTful APIs with proper patterns.

## Learning Objectives

By the end of this lesson, you will:
- Master route parameters and query strings
- Build nested and hierarchical routes
- Implement RESTful routing patterns
- Use route versioning strategies
- Handle complex URL patterns
- Organize routes for large applications

---

## Route Parameters

### Basic Parameters

```javascript
const express = require('express');
const app = express();

// Single parameter
app.get('/users/:id', (req, res) => {
  const { id } = req.params;
  res.json({ userId: id });
});

// Multiple parameters
app.get('/users/:userId/posts/:postId', (req, res) => {
  const { userId, postId } = req.params;
  res.json({ userId, postId });
});

// Optional parameters (with ?)
app.get('/products/:category/:id?', (req, res) => {
  const { category, id } = req.params;
  if (id) {
    res.json({ category, productId: id });
  } else {
    res.json({ category, message: 'All products in category' });
  }
});
```

### Parameter Constraints

```javascript
// Only match numeric IDs
app.get('/users/:id(\\d+)', (req, res) => {
  res.json({ id: parseInt(req.params.id) });
});

// Only match UUIDs
const UUID_REGEX = '[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}';
app.get(`/items/:id(${UUID_REGEX})`, (req, res) => {
  res.json({ uuid: req.params.id });
});

// Match specific values
app.get('/status/:status(active|inactive|pending)', (req, res) => {
  res.json({ status: req.params.status });
});
```

### Route Parameter Middleware

```javascript
// Pre-process parameters
app.param('userId', async (req, res, next, id) => {
  try {
    const user = await User.findById(id);
    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }
    req.user = user;
    next();
  } catch (err) {
    next(err);
  }
});

// Now all routes with :userId have req.user populated
app.get('/users/:userId', (req, res) => {
  res.json(req.user);
});

app.get('/users/:userId/posts', (req, res) => {
  res.json({ user: req.user, posts: req.user.posts });
});
```

---

## Query Strings

### Basic Query Handling

```javascript
// GET /search?q=javascript&page=1&limit=10
app.get('/search', (req, res) => {
  const {
    q,
    page = 1,      // Default value
    limit = 10,
    sort = 'date',
    order = 'desc'
  } = req.query;

  res.json({
    query: q,
    pagination: {
      page: parseInt(page),
      limit: parseInt(limit),
    },
    sorting: { sort, order }
  });
});
```

### Complex Query Patterns

```javascript
// GET /products?price[min]=10&price[max]=100&tags[]=electronics&tags[]=sale
app.get('/products', (req, res) => {
  const { price, tags, ...rest } = req.query;

  const filters = {
    priceRange: price ? { min: price.min, max: price.max } : null,
    tags: tags || [],
    other: rest
  };

  res.json({ filters });
});

// Query validation middleware
function validatePagination(req, res, next) {
  let { page, limit } = req.query;

  page = parseInt(page) || 1;
  limit = parseInt(limit) || 10;

  // Enforce limits
  if (page < 1) page = 1;
  if (limit < 1) limit = 1;
  if (limit > 100) limit = 100;

  req.pagination = { page, limit, skip: (page - 1) * limit };
  next();
}

app.get('/items', validatePagination, (req, res) => {
  const { page, limit, skip } = req.pagination;
  res.json({ page, limit, skip });
});
```

### Filter Builder Pattern

```javascript
class QueryBuilder {
  constructor(query) {
    this.query = query;
    this.filters = {};
  }

  filter() {
    const excluded = ['page', 'limit', 'sort', 'fields'];
    const queryObj = { ...this.query };

    excluded.forEach(field => delete queryObj[field]);

    // Handle operators: price[gte]=100 => price: { $gte: 100 }
    let queryStr = JSON.stringify(queryObj);
    queryStr = queryStr.replace(/\b(gte|gt|lte|lt)\b/g, match => `$${match}`);

    this.filters = JSON.parse(queryStr);
    return this;
  }

  sort() {
    if (this.query.sort) {
      // sort=price,-date => { price: 1, date: -1 }
      const sortBy = this.query.sort.split(',').reduce((acc, field) => {
        if (field.startsWith('-')) {
          acc[field.slice(1)] = -1;
        } else {
          acc[field] = 1;
        }
        return acc;
      }, {});
      this.sortBy = sortBy;
    }
    return this;
  }

  paginate() {
    const page = parseInt(this.query.page) || 1;
    const limit = parseInt(this.query.limit) || 10;
    this.pagination = {
      skip: (page - 1) * limit,
      limit,
      page,
    };
    return this;
  }

  selectFields() {
    if (this.query.fields) {
      this.fields = this.query.fields.split(',').join(' ');
    }
    return this;
  }
}

// Usage
app.get('/products', (req, res) => {
  const queryBuilder = new QueryBuilder(req.query)
    .filter()
    .sort()
    .paginate()
    .selectFields();

  res.json({
    filters: queryBuilder.filters,
    sort: queryBuilder.sortBy,
    pagination: queryBuilder.pagination,
    fields: queryBuilder.fields,
  });
});
```

---

## RESTful Route Patterns

### Resource-Based Routes

```javascript
// users.routes.js
const express = require('express');
const router = express.Router();

// Collection routes
router.route('/')
  .get(getUsers)        // GET    /users
  .post(createUser);    // POST   /users

// Individual resource routes
router.route('/:id')
  .get(getUser)         // GET    /users/:id
  .put(updateUser)      // PUT    /users/:id
  .patch(patchUser)     // PATCH  /users/:id
  .delete(deleteUser);  // DELETE /users/:id

module.exports = router;
```

### Nested Resources

```javascript
// users/:userId/posts routes
const router = express.Router({ mergeParams: true });

// GET /users/:userId/posts
router.get('/', async (req, res) => {
  const { userId } = req.params;
  const posts = await Post.find({ author: userId });
  res.json(posts);
});

// POST /users/:userId/posts
router.post('/', async (req, res) => {
  const { userId } = req.params;
  const post = await Post.create({ ...req.body, author: userId });
  res.status(201).json(post);
});

// GET /users/:userId/posts/:postId
router.get('/:postId', async (req, res) => {
  const { userId, postId } = req.params;
  const post = await Post.findOne({ _id: postId, author: userId });
  res.json(post);
});

module.exports = router;

// Main app
const userPostsRouter = require('./routes/userPosts');
app.use('/users/:userId/posts', userPostsRouter);
```

### Action Routes

```javascript
// For non-CRUD actions, use descriptive sub-paths
const router = express.Router();

// Standard CRUD
router.route('/')
  .get(getUsers)
  .post(createUser);

router.route('/:id')
  .get(getUser)
  .put(updateUser)
  .delete(deleteUser);

// Action routes
router.post('/:id/activate', activateUser);
router.post('/:id/deactivate', deactivateUser);
router.post('/:id/reset-password', resetPassword);
router.post('/:id/send-verification', sendVerification);
router.get('/:id/activity', getUserActivity);
router.get('/:id/permissions', getUserPermissions);
router.put('/:id/permissions', updateUserPermissions);
```

---

## Route Organization

### By Feature (Recommended)

```
src/
├── features/
│   ├── users/
│   │   ├── users.routes.js
│   │   ├── users.controller.js
│   │   ├── users.service.js
│   │   └── users.validation.js
│   ├── posts/
│   │   ├── posts.routes.js
│   │   ├── posts.controller.js
│   │   └── posts.service.js
│   └── auth/
│       ├── auth.routes.js
│       └── auth.controller.js
├── middleware/
├── utils/
└── app.js
```

```javascript
// features/users/users.routes.js
const express = require('express');
const router = express.Router();
const controller = require('./users.controller');
const { validateUser } = require('./users.validation');
const { authenticate, authorize } = require('../../middleware/auth');

router.route('/')
  .get(authenticate, controller.getAll)
  .post(authenticate, authorize('admin'), validateUser, controller.create);

router.route('/:id')
  .get(authenticate, controller.getOne)
  .put(authenticate, validateUser, controller.update)
  .delete(authenticate, authorize('admin'), controller.delete);

module.exports = router;
```

### Route Index File

```javascript
// routes/index.js
const express = require('express');
const router = express.Router();

const usersRouter = require('../features/users/users.routes');
const postsRouter = require('../features/posts/posts.routes');
const authRouter = require('../features/auth/auth.routes');

router.use('/auth', authRouter);
router.use('/users', usersRouter);
router.use('/posts', postsRouter);

module.exports = router;

// app.js
const routes = require('./routes');
app.use('/api/v1', routes);
```

---

## API Versioning

### URL Path Versioning

```javascript
// Most common approach
const v1Router = require('./routes/v1');
const v2Router = require('./routes/v2');

app.use('/api/v1', v1Router);
app.use('/api/v2', v2Router);

// routes/v1/index.js
const router = express.Router();
router.use('/users', require('./users'));
module.exports = router;

// routes/v2/index.js
const router = express.Router();
router.use('/users', require('./users')); // Updated version
module.exports = router;
```

### Header-Based Versioning

```javascript
function versionRouter(versions) {
  return (req, res, next) => {
    const requestedVersion = req.headers['api-version'] || 'v1';

    if (versions[requestedVersion]) {
      return versions[requestedVersion](req, res, next);
    }

    res.status(400).json({ error: `API version ${requestedVersion} not supported` });
  };
}

// Usage
const v1Handler = require('./handlers/v1/users');
const v2Handler = require('./handlers/v2/users');

app.get('/api/users', versionRouter({
  v1: v1Handler.getUsers,
  v2: v2Handler.getUsers,
}));
```

### Query Parameter Versioning

```javascript
function versionMiddleware(req, res, next) {
  const version = req.query.version || req.headers['api-version'] || '1';
  req.apiVersion = parseInt(version);
  next();
}

app.use(versionMiddleware);

app.get('/api/users', (req, res) => {
  if (req.apiVersion >= 2) {
    // V2 response format
    return res.json({ data: users, meta: { total: users.length } });
  }
  // V1 response format
  res.json(users);
});
```

---

## Advanced Patterns

### Route Factory

```javascript
function createCrudRoutes(resource, controller) {
  const router = express.Router();

  router.route('/')
    .get(controller.getAll)
    .post(controller.create);

  router.route('/:id')
    .get(controller.getOne)
    .put(controller.update)
    .patch(controller.patch)
    .delete(controller.delete);

  return router;
}

// Usage
const userController = require('./controllers/user');
const postController = require('./controllers/post');

app.use('/api/users', createCrudRoutes('user', userController));
app.use('/api/posts', createCrudRoutes('post', postController));
```

### Generic Controller Factory

```javascript
function createController(Model) {
  return {
    async getAll(req, res, next) {
      try {
        const items = await Model.find();
        res.json(items);
      } catch (err) {
        next(err);
      }
    },

    async getOne(req, res, next) {
      try {
        const item = await Model.findById(req.params.id);
        if (!item) {
          return res.status(404).json({ error: 'Not found' });
        }
        res.json(item);
      } catch (err) {
        next(err);
      }
    },

    async create(req, res, next) {
      try {
        const item = await Model.create(req.body);
        res.status(201).json(item);
      } catch (err) {
        next(err);
      }
    },

    async update(req, res, next) {
      try {
        const item = await Model.findByIdAndUpdate(
          req.params.id,
          req.body,
          { new: true, runValidators: true }
        );
        if (!item) {
          return res.status(404).json({ error: 'Not found' });
        }
        res.json(item);
      } catch (err) {
        next(err);
      }
    },

    async delete(req, res, next) {
      try {
        const item = await Model.findByIdAndDelete(req.params.id);
        if (!item) {
          return res.status(404).json({ error: 'Not found' });
        }
        res.status(204).send();
      } catch (err) {
        next(err);
      }
    },
  };
}

// Usage
const User = require('./models/User');
const userController = createController(User);
```

### Wildcard Routes

```javascript
// Catch-all for SPA
app.get('*', (req, res) => {
  res.sendFile(path.join(__dirname, 'public', 'index.html'));
});

// API catch-all
app.all('/api/*', (req, res) => {
  res.status(404).json({ error: 'API endpoint not found' });
});
```

---

## Route Security

### Route-Level Authentication

```javascript
const router = express.Router();

// Public routes
router.get('/products', getProducts);
router.get('/products/:id', getProduct);

// Protected routes
router.use(authenticate); // All routes below require auth

router.post('/products', authorize('admin'), createProduct);
router.put('/products/:id', authorize('admin'), updateProduct);
router.delete('/products/:id', authorize('admin'), deleteProduct);

// User-specific routes
router.get('/orders', getUserOrders);
router.post('/orders', createOrder);
```

### Resource Ownership

```javascript
async function checkOwnership(req, res, next) {
  const resource = await Resource.findById(req.params.id);

  if (!resource) {
    return res.status(404).json({ error: 'Not found' });
  }

  if (resource.owner.toString() !== req.user.id) {
    return res.status(403).json({ error: 'Not authorized' });
  }

  req.resource = resource;
  next();
}

router.put('/:id', authenticate, checkOwnership, updateResource);
router.delete('/:id', authenticate, checkOwnership, deleteResource);
```

---

## Exercise: Build a Blog API Routes

Design and implement routes for a blog application with:
- Posts (CRUD + publish/unpublish actions)
- Comments (nested under posts)
- Tags (many-to-many with posts)
- User profiles and posts

---

## Solution

```javascript
// routes/posts.js
const express = require('express');
const router = express.Router();
const { authenticate, authorize } = require('../middleware/auth');

// Post CRUD
router.route('/')
  .get(getPosts)           // GET /posts - List all published posts
  .post(authenticate, createPost);  // POST /posts - Create post

router.route('/:id')
  .get(getPost)            // GET /posts/:id
  .put(authenticate, checkPostOwner, updatePost)     // PUT /posts/:id
  .delete(authenticate, checkPostOwner, deletePost); // DELETE /posts/:id

// Post actions
router.post('/:id/publish', authenticate, checkPostOwner, publishPost);
router.post('/:id/unpublish', authenticate, checkPostOwner, unpublishPost);
router.post('/:id/like', authenticate, likePost);
router.delete('/:id/like', authenticate, unlikePost);

// Post tags
router.get('/:id/tags', getPostTags);
router.post('/:id/tags', authenticate, checkPostOwner, addPostTag);
router.delete('/:id/tags/:tagId', authenticate, checkPostOwner, removePostTag);

module.exports = router;

// routes/comments.js (nested under posts)
const router = express.Router({ mergeParams: true }); // Access parent params

router.route('/')
  .get(getComments)        // GET /posts/:postId/comments
  .post(authenticate, createComment);  // POST /posts/:postId/comments

router.route('/:commentId')
  .get(getComment)         // GET /posts/:postId/comments/:commentId
  .put(authenticate, checkCommentOwner, updateComment)
  .delete(authenticate, checkCommentOwner, deleteComment);

module.exports = router;

// routes/tags.js
const router = express.Router();

router.get('/', getTags);           // GET /tags
router.get('/:slug', getTagBySlug); // GET /tags/:slug
router.get('/:slug/posts', getTagPosts); // GET /tags/:slug/posts

// Admin only
router.post('/', authenticate, authorize('admin'), createTag);
router.put('/:id', authenticate, authorize('admin'), updateTag);
router.delete('/:id', authenticate, authorize('admin'), deleteTag);

module.exports = router;

// routes/users.js
const router = express.Router();

// Public
router.get('/:username', getUserProfile);
router.get('/:username/posts', getUserPosts);

// Authenticated user's own routes
router.get('/me', authenticate, getMyProfile);
router.put('/me', authenticate, updateMyProfile);
router.get('/me/posts', authenticate, getMyPosts);
router.get('/me/drafts', authenticate, getMyDrafts);

module.exports = router;

// routes/index.js
const express = require('express');
const router = express.Router();

const postsRouter = require('./posts');
const commentsRouter = require('./comments');
const tagsRouter = require('./tags');
const usersRouter = require('./users');

router.use('/posts', postsRouter);
router.use('/posts/:postId/comments', commentsRouter);
router.use('/tags', tagsRouter);
router.use('/users', usersRouter);

module.exports = router;

// app.js
const routes = require('./routes');
app.use('/api/v1', routes);
```

API endpoints summary:
```
Posts:
GET    /api/v1/posts
POST   /api/v1/posts
GET    /api/v1/posts/:id
PUT    /api/v1/posts/:id
DELETE /api/v1/posts/:id
POST   /api/v1/posts/:id/publish
POST   /api/v1/posts/:id/unpublish
POST   /api/v1/posts/:id/like
DELETE /api/v1/posts/:id/like

Comments:
GET    /api/v1/posts/:postId/comments
POST   /api/v1/posts/:postId/comments
GET    /api/v1/posts/:postId/comments/:commentId
PUT    /api/v1/posts/:postId/comments/:commentId
DELETE /api/v1/posts/:postId/comments/:commentId

Tags:
GET    /api/v1/tags
GET    /api/v1/tags/:slug
GET    /api/v1/tags/:slug/posts
POST   /api/v1/tags (admin)
PUT    /api/v1/tags/:id (admin)
DELETE /api/v1/tags/:id (admin)

Users:
GET    /api/v1/users/:username
GET    /api/v1/users/:username/posts
GET    /api/v1/users/me
PUT    /api/v1/users/me
GET    /api/v1/users/me/posts
GET    /api/v1/users/me/drafts
```

---

## Key Takeaways

1. **Use route parameters wisely** - Validate and preprocess with `app.param()`
2. **Query strings for filtering** - Build reusable query builders
3. **Follow REST conventions** - Resources, HTTP methods, nested routes
4. **Organize by feature** - Keep related code together
5. **Version your API** - URL path versioning is most common
6. **Factory patterns** - Reduce boilerplate with generic controllers
7. **Secure routes appropriately** - Authentication and authorization per route

---

## What's Next?

Tomorrow we'll cover **Error Handling in Express** - building robust error handling systems, custom error classes, and global error middleware!
