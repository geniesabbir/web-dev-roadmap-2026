# Days 8-9: Building a Complete REST API

## Introduction

Over these two days, we'll build a production-ready REST API from scratch. This project brings together everything you've learned about Node.js and Express: routing, middleware, error handling, validation, and best practices.

## Learning Objectives

By the end of these lessons, you will:
- Design and implement RESTful endpoints
- Structure a scalable Express application
- Implement CRUD operations with validation
- Add pagination, filtering, and sorting
- Handle relationships between resources
- Apply security best practices

---

## Project: Blog API

We'll build a blog API with:
- **Users** - Registration, profiles
- **Posts** - CRUD with author relationship
- **Comments** - Nested under posts
- **Categories** - Many-to-many with posts

### Project Structure

```
blog-api/
├── src/
│   ├── config/
│   │   ├── index.js
│   │   └── database.js
│   ├── middleware/
│   │   ├── auth.js
│   │   ├── validate.js
│   │   ├── errorHandler.js
│   │   └── notFound.js
│   ├── modules/
│   │   ├── users/
│   │   │   ├── user.model.js
│   │   │   ├── user.routes.js
│   │   │   ├── user.controller.js
│   │   │   ├── user.service.js
│   │   │   └── user.validation.js
│   │   ├── posts/
│   │   │   ├── post.model.js
│   │   │   ├── post.routes.js
│   │   │   ├── post.controller.js
│   │   │   └── post.service.js
│   │   ├── comments/
│   │   │   └── ...
│   │   └── auth/
│   │       └── ...
│   ├── utils/
│   │   ├── AppError.js
│   │   ├── catchAsync.js
│   │   ├── response.js
│   │   └── pagination.js
│   ├── app.js
│   └── server.js
├── .env
├── .env.example
└── package.json
```

---

## Step 1: Project Setup

### Initialize Project

```bash
mkdir blog-api && cd blog-api
npm init -y
npm install express mongoose dotenv bcryptjs jsonwebtoken cors helmet morgan express-validator
npm install -D nodemon
```

### Environment Configuration

```javascript
// .env
NODE_ENV=development
PORT=3000
MONGODB_URI=mongodb://localhost:27017/blog-api
JWT_SECRET=your-super-secret-key-change-in-production
JWT_EXPIRES_IN=7d

// .env.example (commit this)
NODE_ENV=development
PORT=3000
MONGODB_URI=mongodb://localhost:27017/blog-api
JWT_SECRET=
JWT_EXPIRES_IN=7d
```

```javascript
// src/config/index.js
require('dotenv').config();

module.exports = {
  env: process.env.NODE_ENV || 'development',
  port: process.env.PORT || 3000,
  mongoUri: process.env.MONGODB_URI,
  jwt: {
    secret: process.env.JWT_SECRET,
    expiresIn: process.env.JWT_EXPIRES_IN || '7d',
  },
};
```

### Database Connection

```javascript
// src/config/database.js
const mongoose = require('mongoose');
const config = require('./index');

const connectDB = async () => {
  try {
    const conn = await mongoose.connect(config.mongoUri);
    console.log(`MongoDB Connected: ${conn.connection.host}`);
  } catch (error) {
    console.error('Database connection error:', error.message);
    process.exit(1);
  }
};

module.exports = connectDB;
```

---

## Step 2: Utilities

### AppError Class

```javascript
// src/utils/AppError.js
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

### Async Handler

```javascript
// src/utils/catchAsync.js
module.exports = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};
```

### Response Helpers

```javascript
// src/utils/response.js
exports.success = (res, data, statusCode = 200) => {
  res.status(statusCode).json({
    status: 'success',
    data,
  });
};

exports.created = (res, data) => {
  res.status(201).json({
    status: 'success',
    data,
  });
};

exports.noContent = (res) => {
  res.status(204).send();
};

exports.paginated = (res, data, pagination) => {
  res.status(200).json({
    status: 'success',
    results: data.length,
    pagination,
    data,
  });
};
```

### Pagination Helper

```javascript
// src/utils/pagination.js
class QueryBuilder {
  constructor(query, queryString) {
    this.query = query;
    this.queryString = queryString;
  }

  filter() {
    const queryObj = { ...this.queryString };
    const excludedFields = ['page', 'sort', 'limit', 'fields', 'search'];
    excludedFields.forEach(field => delete queryObj[field]);

    // Handle operators: price[gte]=100 => price: { $gte: 100 }
    let queryStr = JSON.stringify(queryObj);
    queryStr = queryStr.replace(/\b(gte|gt|lte|lt|in)\b/g, match => `$${match}`);

    this.query = this.query.find(JSON.parse(queryStr));
    return this;
  }

  search(fields) {
    if (this.queryString.search) {
      const searchRegex = new RegExp(this.queryString.search, 'i');
      const searchQuery = fields.map(field => ({ [field]: searchRegex }));
      this.query = this.query.find({ $or: searchQuery });
    }
    return this;
  }

  sort() {
    if (this.queryString.sort) {
      const sortBy = this.queryString.sort.split(',').join(' ');
      this.query = this.query.sort(sortBy);
    } else {
      this.query = this.query.sort('-createdAt');
    }
    return this;
  }

  select() {
    if (this.queryString.fields) {
      const fields = this.queryString.fields.split(',').join(' ');
      this.query = this.query.select(fields);
    } else {
      this.query = this.query.select('-__v');
    }
    return this;
  }

  paginate() {
    const page = parseInt(this.queryString.page) || 1;
    const limit = parseInt(this.queryString.limit) || 10;
    const skip = (page - 1) * limit;

    this.query = this.query.skip(skip).limit(limit);
    this.pagination = { page, limit, skip };
    return this;
  }
}

module.exports = QueryBuilder;
```

---

## Step 3: User Module

### User Model

```javascript
// src/modules/users/user.model.js
const mongoose = require('mongoose');
const bcrypt = require('bcryptjs');

const userSchema = new mongoose.Schema({
  name: {
    type: String,
    required: [true, 'Name is required'],
    trim: true,
    minlength: [2, 'Name must be at least 2 characters'],
    maxlength: [50, 'Name cannot exceed 50 characters'],
  },
  email: {
    type: String,
    required: [true, 'Email is required'],
    unique: true,
    lowercase: true,
    trim: true,
  },
  password: {
    type: String,
    required: [true, 'Password is required'],
    minlength: [8, 'Password must be at least 8 characters'],
    select: false,
  },
  role: {
    type: String,
    enum: ['user', 'admin'],
    default: 'user',
  },
  avatar: String,
  bio: {
    type: String,
    maxlength: [500, 'Bio cannot exceed 500 characters'],
  },
  active: {
    type: Boolean,
    default: true,
    select: false,
  },
}, {
  timestamps: true,
  toJSON: { virtuals: true },
  toObject: { virtuals: true },
});

// Virtual for user's posts
userSchema.virtual('posts', {
  ref: 'Post',
  foreignField: 'author',
  localField: '_id',
});

// Hash password before saving
userSchema.pre('save', async function(next) {
  if (!this.isModified('password')) return next();
  this.password = await bcrypt.hash(this.password, 12);
  next();
});

// Filter out inactive users
userSchema.pre(/^find/, function(next) {
  this.find({ active: { $ne: false } });
  next();
});

// Compare passwords
userSchema.methods.comparePassword = async function(candidatePassword) {
  return bcrypt.compare(candidatePassword, this.password);
};

module.exports = mongoose.model('User', userSchema);
```

### User Validation

```javascript
// src/modules/users/user.validation.js
const { body } = require('express-validator');

exports.createUser = [
  body('name')
    .trim()
    .notEmpty().withMessage('Name is required')
    .isLength({ min: 2, max: 50 }).withMessage('Name must be 2-50 characters'),
  body('email')
    .trim()
    .notEmpty().withMessage('Email is required')
    .isEmail().withMessage('Invalid email format')
    .normalizeEmail(),
  body('password')
    .notEmpty().withMessage('Password is required')
    .isLength({ min: 8 }).withMessage('Password must be at least 8 characters'),
];

exports.updateUser = [
  body('name')
    .optional()
    .trim()
    .isLength({ min: 2, max: 50 }).withMessage('Name must be 2-50 characters'),
  body('bio')
    .optional()
    .trim()
    .isLength({ max: 500 }).withMessage('Bio cannot exceed 500 characters'),
];
```

### User Service

```javascript
// src/modules/users/user.service.js
const User = require('./user.model');
const AppError = require('../../utils/AppError');
const QueryBuilder = require('../../utils/pagination');

exports.findAll = async (query) => {
  const features = new QueryBuilder(User.find(), query)
    .filter()
    .search(['name', 'email'])
    .sort()
    .select()
    .paginate();

  const users = await features.query;
  const total = await User.countDocuments();

  return {
    users,
    pagination: {
      ...features.pagination,
      total,
      pages: Math.ceil(total / features.pagination.limit),
    },
  };
};

exports.findById = async (id) => {
  const user = await User.findById(id);
  if (!user) {
    throw new AppError('User not found', 404);
  }
  return user;
};

exports.findByEmail = async (email) => {
  return User.findOne({ email }).select('+password');
};

exports.create = async (data) => {
  const existingUser = await User.findOne({ email: data.email });
  if (existingUser) {
    throw new AppError('Email already registered', 409);
  }
  return User.create(data);
};

exports.update = async (id, data) => {
  const user = await User.findByIdAndUpdate(id, data, {
    new: true,
    runValidators: true,
  });
  if (!user) {
    throw new AppError('User not found', 404);
  }
  return user;
};

exports.delete = async (id) => {
  const user = await User.findByIdAndUpdate(id, { active: false });
  if (!user) {
    throw new AppError('User not found', 404);
  }
};
```

### User Controller

```javascript
// src/modules/users/user.controller.js
const userService = require('./user.service');
const catchAsync = require('../../utils/catchAsync');
const { success, created, noContent, paginated } = require('../../utils/response');

exports.getAll = catchAsync(async (req, res) => {
  const { users, pagination } = await userService.findAll(req.query);
  paginated(res, users, pagination);
});

exports.getOne = catchAsync(async (req, res) => {
  const user = await userService.findById(req.params.id);
  success(res, { user });
});

exports.getMe = catchAsync(async (req, res) => {
  const user = await userService.findById(req.user.id);
  success(res, { user });
});

exports.updateMe = catchAsync(async (req, res) => {
  const { name, bio, avatar } = req.body;
  const user = await userService.update(req.user.id, { name, bio, avatar });
  success(res, { user });
});

exports.deleteMe = catchAsync(async (req, res) => {
  await userService.delete(req.user.id);
  noContent(res);
});
```

### User Routes

```javascript
// src/modules/users/user.routes.js
const express = require('express');
const router = express.Router();
const controller = require('./user.controller');
const validation = require('./user.validation');
const { protect, restrictTo } = require('../../middleware/auth');
const validate = require('../../middleware/validate');

// Current user routes
router.use(protect);

router.get('/me', controller.getMe);
router.patch('/me', validate(validation.updateUser), controller.updateMe);
router.delete('/me', controller.deleteMe);

// Admin routes
router.use(restrictTo('admin'));

router.get('/', controller.getAll);
router.get('/:id', controller.getOne);

module.exports = router;
```

---

## Step 4: Auth Module

### Auth Controller

```javascript
// src/modules/auth/auth.controller.js
const jwt = require('jsonwebtoken');
const userService = require('../users/user.service');
const catchAsync = require('../../utils/catchAsync');
const AppError = require('../../utils/AppError');
const config = require('../../config');
const { success, created } = require('../../utils/response');

const signToken = (id) => {
  return jwt.sign({ id }, config.jwt.secret, {
    expiresIn: config.jwt.expiresIn,
  });
};

const createSendToken = (user, statusCode, res) => {
  const token = signToken(user._id);

  // Remove password from output
  user.password = undefined;

  const response = { token, user };

  if (statusCode === 201) {
    created(res, response);
  } else {
    success(res, response);
  }
};

exports.register = catchAsync(async (req, res) => {
  const { name, email, password } = req.body;
  const user = await userService.create({ name, email, password });
  createSendToken(user, 201, res);
});

exports.login = catchAsync(async (req, res) => {
  const { email, password } = req.body;

  if (!email || !password) {
    throw new AppError('Please provide email and password', 400);
  }

  const user = await userService.findByEmail(email);

  if (!user || !(await user.comparePassword(password))) {
    throw new AppError('Invalid email or password', 401);
  }

  createSendToken(user, 200, res);
});
```

### Auth Routes

```javascript
// src/modules/auth/auth.routes.js
const express = require('express');
const router = express.Router();
const controller = require('./auth.controller');
const { createUser } = require('../users/user.validation');
const validate = require('../../middleware/validate');

router.post('/register', validate(createUser), controller.register);
router.post('/login', controller.login);

module.exports = router;
```

### Auth Middleware

```javascript
// src/middleware/auth.js
const jwt = require('jsonwebtoken');
const User = require('../modules/users/user.model');
const AppError = require('../utils/AppError');
const catchAsync = require('../utils/catchAsync');
const config = require('../config');

exports.protect = catchAsync(async (req, res, next) => {
  let token;

  if (req.headers.authorization?.startsWith('Bearer')) {
    token = req.headers.authorization.split(' ')[1];
  }

  if (!token) {
    throw new AppError('Not authenticated', 401);
  }

  const decoded = jwt.verify(token, config.jwt.secret);
  const user = await User.findById(decoded.id);

  if (!user) {
    throw new AppError('User no longer exists', 401);
  }

  req.user = user;
  next();
});

exports.restrictTo = (...roles) => {
  return (req, res, next) => {
    if (!roles.includes(req.user.role)) {
      throw new AppError('Not authorized', 403);
    }
    next();
  };
};
```

---

## Step 5: Post Module

### Post Model

```javascript
// src/modules/posts/post.model.js
const mongoose = require('mongoose');
const slugify = require('slugify');

const postSchema = new mongoose.Schema({
  title: {
    type: String,
    required: [true, 'Title is required'],
    trim: true,
    maxlength: [200, 'Title cannot exceed 200 characters'],
  },
  slug: String,
  content: {
    type: String,
    required: [true, 'Content is required'],
  },
  excerpt: {
    type: String,
    maxlength: [500, 'Excerpt cannot exceed 500 characters'],
  },
  coverImage: String,
  author: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true,
  },
  categories: [{
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Category',
  }],
  tags: [String],
  status: {
    type: String,
    enum: ['draft', 'published'],
    default: 'draft',
  },
  publishedAt: Date,
  views: {
    type: Number,
    default: 0,
  },
}, {
  timestamps: true,
  toJSON: { virtuals: true },
  toObject: { virtuals: true },
});

// Indexes
postSchema.index({ slug: 1 });
postSchema.index({ author: 1, status: 1 });
postSchema.index({ title: 'text', content: 'text' });

// Virtual for comments
postSchema.virtual('comments', {
  ref: 'Comment',
  foreignField: 'post',
  localField: '_id',
});

// Generate slug before saving
postSchema.pre('save', function(next) {
  if (this.isModified('title')) {
    this.slug = slugify(this.title, { lower: true, strict: true });
  }
  if (this.isModified('status') && this.status === 'published' && !this.publishedAt) {
    this.publishedAt = new Date();
  }
  if (!this.excerpt && this.content) {
    this.excerpt = this.content.substring(0, 200) + '...';
  }
  next();
});

// Populate author on find
postSchema.pre(/^find/, function(next) {
  this.populate({
    path: 'author',
    select: 'name avatar',
  });
  next();
});

module.exports = mongoose.model('Post', postSchema);
```

### Post Service

```javascript
// src/modules/posts/post.service.js
const Post = require('./post.model');
const AppError = require('../../utils/AppError');
const QueryBuilder = require('../../utils/pagination');

exports.findAll = async (query, options = {}) => {
  let filter = {};

  if (options.author) {
    filter.author = options.author;
  }

  if (options.onlyPublished) {
    filter.status = 'published';
  }

  const features = new QueryBuilder(Post.find(filter), query)
    .filter()
    .search(['title', 'content', 'excerpt'])
    .sort()
    .select()
    .paginate();

  const posts = await features.query;
  const total = await Post.countDocuments(filter);

  return {
    posts,
    pagination: {
      ...features.pagination,
      total,
      pages: Math.ceil(total / features.pagination.limit),
    },
  };
};

exports.findBySlug = async (slug) => {
  const post = await Post.findOne({ slug, status: 'published' })
    .populate('categories')
    .populate({
      path: 'comments',
      match: { parent: null },
      populate: { path: 'author', select: 'name avatar' },
    });

  if (!post) {
    throw new AppError('Post not found', 404);
  }

  // Increment views
  post.views += 1;
  await post.save({ validateBeforeSave: false });

  return post;
};

exports.findById = async (id, userId = null) => {
  const post = await Post.findById(id);

  if (!post) {
    throw new AppError('Post not found', 404);
  }

  // Check ownership for drafts
  if (post.status === 'draft' && post.author._id.toString() !== userId) {
    throw new AppError('Post not found', 404);
  }

  return post;
};

exports.create = async (data, authorId) => {
  return Post.create({ ...data, author: authorId });
};

exports.update = async (id, data, userId) => {
  const post = await Post.findById(id);

  if (!post) {
    throw new AppError('Post not found', 404);
  }

  if (post.author._id.toString() !== userId) {
    throw new AppError('Not authorized to update this post', 403);
  }

  Object.assign(post, data);
  await post.save();

  return post;
};

exports.delete = async (id, userId, isAdmin = false) => {
  const post = await Post.findById(id);

  if (!post) {
    throw new AppError('Post not found', 404);
  }

  if (!isAdmin && post.author._id.toString() !== userId) {
    throw new AppError('Not authorized to delete this post', 403);
  }

  await post.deleteOne();
};

exports.publish = async (id, userId) => {
  return this.update(id, { status: 'published' }, userId);
};

exports.unpublish = async (id, userId) => {
  return this.update(id, { status: 'draft', publishedAt: null }, userId);
};
```

### Post Controller

```javascript
// src/modules/posts/post.controller.js
const postService = require('./post.service');
const catchAsync = require('../../utils/catchAsync');
const { success, created, noContent, paginated } = require('../../utils/response');

exports.getAll = catchAsync(async (req, res) => {
  const { posts, pagination } = await postService.findAll(req.query, {
    onlyPublished: true,
  });
  paginated(res, posts, pagination);
});

exports.getBySlug = catchAsync(async (req, res) => {
  const post = await postService.findBySlug(req.params.slug);
  success(res, { post });
});

exports.getOne = catchAsync(async (req, res) => {
  const post = await postService.findById(req.params.id, req.user?.id);
  success(res, { post });
});

exports.create = catchAsync(async (req, res) => {
  const post = await postService.create(req.body, req.user.id);
  created(res, { post });
});

exports.update = catchAsync(async (req, res) => {
  const post = await postService.update(req.params.id, req.body, req.user.id);
  success(res, { post });
});

exports.delete = catchAsync(async (req, res) => {
  await postService.delete(req.params.id, req.user.id, req.user.role === 'admin');
  noContent(res);
});

exports.publish = catchAsync(async (req, res) => {
  const post = await postService.publish(req.params.id, req.user.id);
  success(res, { post });
});

exports.unpublish = catchAsync(async (req, res) => {
  const post = await postService.unpublish(req.params.id, req.user.id);
  success(res, { post });
});

exports.getMyPosts = catchAsync(async (req, res) => {
  const { posts, pagination } = await postService.findAll(req.query, {
    author: req.user.id,
  });
  paginated(res, posts, pagination);
});
```

### Post Routes

```javascript
// src/modules/posts/post.routes.js
const express = require('express');
const router = express.Router();
const controller = require('./post.controller');
const { protect, restrictTo } = require('../../middleware/auth');

// Public routes
router.get('/', controller.getAll);
router.get('/slug/:slug', controller.getBySlug);

// Protected routes
router.use(protect);

router.get('/me', controller.getMyPosts);
router.post('/', controller.create);

router.route('/:id')
  .get(controller.getOne)
  .patch(controller.update)
  .delete(controller.delete);

router.post('/:id/publish', controller.publish);
router.post('/:id/unpublish', controller.unpublish);

module.exports = router;
```

---

## Step 6: Comments Module

```javascript
// src/modules/comments/comment.model.js
const mongoose = require('mongoose');

const commentSchema = new mongoose.Schema({
  content: {
    type: String,
    required: [true, 'Comment content is required'],
    maxlength: [1000, 'Comment cannot exceed 1000 characters'],
  },
  post: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Post',
    required: true,
  },
  author: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true,
  },
  parent: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Comment',
    default: null,
  },
}, {
  timestamps: true,
  toJSON: { virtuals: true },
  toObject: { virtuals: true },
});

// Virtual for replies
commentSchema.virtual('replies', {
  ref: 'Comment',
  foreignField: 'parent',
  localField: '_id',
});

commentSchema.pre(/^find/, function(next) {
  this.populate({
    path: 'author',
    select: 'name avatar',
  });
  next();
});

module.exports = mongoose.model('Comment', commentSchema);
```

---

## Step 7: App Setup

### Validation Middleware

```javascript
// src/middleware/validate.js
const { validationResult } = require('express-validator');
const AppError = require('../utils/AppError');

module.exports = (validations) => {
  return async (req, res, next) => {
    await Promise.all(validations.map(validation => validation.run(req)));

    const errors = validationResult(req);
    if (errors.isEmpty()) {
      return next();
    }

    const extractedErrors = errors.array().map(err => ({
      field: err.path,
      message: err.msg,
    }));

    const error = new AppError('Validation failed', 400);
    error.errors = extractedErrors;
    next(error);
  };
};
```

### Error Handler

```javascript
// src/middleware/errorHandler.js
const AppError = require('../utils/AppError');
const config = require('../config');

const handleCastErrorDB = (err) => {
  return new AppError(`Invalid ${err.path}: ${err.value}`, 400);
};

const handleDuplicateFieldsDB = (err) => {
  const field = Object.keys(err.keyValue)[0];
  return new AppError(`${field} already exists`, 409);
};

const handleValidationErrorDB = (err) => {
  const errors = Object.values(err.errors).map(e => ({
    field: e.path,
    message: e.message,
  }));
  const error = new AppError('Validation failed', 400);
  error.errors = errors;
  return error;
};

const handleJWTError = () => new AppError('Invalid token', 401);
const handleJWTExpiredError = () => new AppError('Token expired', 401);

module.exports = (err, req, res, next) => {
  err.statusCode = err.statusCode || 500;
  err.status = err.status || 'error';

  if (config.env === 'development') {
    return res.status(err.statusCode).json({
      status: err.status,
      error: err,
      message: err.message,
      errors: err.errors,
      stack: err.stack,
    });
  }

  let error = { ...err, message: err.message };

  if (err.name === 'CastError') error = handleCastErrorDB(err);
  if (err.code === 11000) error = handleDuplicateFieldsDB(err);
  if (err.name === 'ValidationError') error = handleValidationErrorDB(err);
  if (err.name === 'JsonWebTokenError') error = handleJWTError();
  if (err.name === 'TokenExpiredError') error = handleJWTExpiredError();

  if (error.isOperational) {
    return res.status(error.statusCode).json({
      status: error.status,
      message: error.message,
      ...(error.errors && { errors: error.errors }),
    });
  }

  console.error('ERROR:', err);
  res.status(500).json({
    status: 'error',
    message: 'Something went wrong',
  });
};
```

### App Configuration

```javascript
// src/app.js
const express = require('express');
const cors = require('cors');
const helmet = require('helmet');
const morgan = require('morgan');

const authRoutes = require('./modules/auth/auth.routes');
const userRoutes = require('./modules/users/user.routes');
const postRoutes = require('./modules/posts/post.routes');
const AppError = require('./utils/AppError');
const errorHandler = require('./middleware/errorHandler');
const config = require('./config');

const app = express();

// Security middleware
app.use(helmet());
app.use(cors());

// Logging
if (config.env === 'development') {
  app.use(morgan('dev'));
}

// Body parsing
app.use(express.json({ limit: '10kb' }));

// Routes
app.use('/api/auth', authRoutes);
app.use('/api/users', userRoutes);
app.use('/api/posts', postRoutes);

// Health check
app.get('/health', (req, res) => {
  res.json({ status: 'ok', timestamp: new Date().toISOString() });
});

// 404 handler
app.use((req, res, next) => {
  next(new AppError(`Cannot ${req.method} ${req.originalUrl}`, 404));
});

// Error handler
app.use(errorHandler);

module.exports = app;
```

### Server Entry

```javascript
// src/server.js
const app = require('./app');
const config = require('./config');
const connectDB = require('./config/database');

process.on('uncaughtException', (err) => {
  console.error('UNCAUGHT EXCEPTION:', err);
  process.exit(1);
});

connectDB().then(() => {
  const server = app.listen(config.port, () => {
    console.log(`Server running on port ${config.port} in ${config.env} mode`);
  });

  process.on('unhandledRejection', (err) => {
    console.error('UNHANDLED REJECTION:', err);
    server.close(() => process.exit(1));
  });

  process.on('SIGTERM', () => {
    console.log('SIGTERM received. Shutting down...');
    server.close(() => console.log('Process terminated'));
  });
});
```

---

## API Endpoints Summary

```
Auth:
POST   /api/auth/register     - Register new user
POST   /api/auth/login        - Login

Users:
GET    /api/users/me          - Get current user
PATCH  /api/users/me          - Update current user
DELETE /api/users/me          - Delete current user
GET    /api/users             - Get all users (admin)
GET    /api/users/:id         - Get user by ID (admin)

Posts:
GET    /api/posts             - Get all published posts
GET    /api/posts/slug/:slug  - Get post by slug
GET    /api/posts/me          - Get current user's posts
POST   /api/posts             - Create post
GET    /api/posts/:id         - Get post by ID
PATCH  /api/posts/:id         - Update post
DELETE /api/posts/:id         - Delete post
POST   /api/posts/:id/publish - Publish post
POST   /api/posts/:id/unpublish - Unpublish post
```

---

## Testing the API

```bash
# Register
curl -X POST http://localhost:3000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"name":"John","email":"john@example.com","password":"password123"}'

# Login
curl -X POST http://localhost:3000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"john@example.com","password":"password123"}'

# Create post (use token from login)
curl -X POST http://localhost:3000/api/posts \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -d '{"title":"My First Post","content":"This is the content"}'

# Get posts with pagination
curl "http://localhost:3000/api/posts?page=1&limit=10&sort=-createdAt"

# Search posts
curl "http://localhost:3000/api/posts?search=javascript"
```

---

## Key Takeaways

1. **Modular structure** - Organize by feature, not by type
2. **Separation of concerns** - Routes, controllers, services
3. **Consistent responses** - Use response helpers
4. **Proper error handling** - Custom errors, global handler
5. **Validation** - Use express-validator
6. **Security** - Helmet, CORS, JWT auth
7. **Query features** - Pagination, filtering, sorting

---

## What's Next?

Tomorrow we'll complete the project with additional features and deployment preparation!
