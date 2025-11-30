# Day 8: MongoDB Basics

## Introduction

MongoDB is a popular NoSQL document database that stores data in flexible, JSON-like documents. It's known for its scalability, flexible schema, and great developer experience. This lesson covers MongoDB fundamentals, CRUD operations, and integration with Node.js.

## Learning Objectives

By the end of this lesson, you will:
- Understand MongoDB concepts and data model
- Set up MongoDB locally and in the cloud
- Perform CRUD operations
- Use queries, filters, and operators
- Design document schemas
- Integrate MongoDB with Node.js using Mongoose

---

## MongoDB Concepts

### Document Database vs Relational

```
Relational (SQL)              Document (MongoDB)
─────────────────────────────────────────────────
Database                      Database
Table                         Collection
Row                           Document
Column                        Field
Primary Key                   _id
Foreign Key                   Reference / Embedding
JOIN                          $lookup / Embedding
```

### Document Structure

```javascript
// MongoDB Document (BSON - Binary JSON)
{
  "_id": ObjectId("507f1f77bcf86cd799439011"),
  "name": "John Doe",
  "email": "john@example.com",
  "age": 30,
  "address": {
    "street": "123 Main St",
    "city": "New York",
    "country": "USA"
  },
  "tags": ["developer", "nodejs", "mongodb"],
  "posts": [
    {
      "title": "Hello World",
      "content": "My first post",
      "createdAt": ISODate("2024-01-15T10:00:00Z")
    }
  ],
  "createdAt": ISODate("2024-01-01T00:00:00Z"),
  "updatedAt": ISODate("2024-01-15T10:00:00Z")
}
```

### When to Use MongoDB

**Good for:**
- Flexible, evolving schemas
- Hierarchical/nested data
- High write throughput
- Horizontal scaling
- Rapid prototyping
- Content management
- Real-time analytics
- IoT applications

**Consider SQL instead for:**
- Complex transactions
- Many-to-many relationships
- Strong data integrity requirements
- Complex reporting/analytics

---

## Setup

### Local Installation

```bash
# macOS with Homebrew
brew tap mongodb/brew
brew install mongodb-community
brew services start mongodb-community

# Docker (recommended)
docker run -d --name mongodb \
  -p 27017:27017 \
  -e MONGO_INITDB_ROOT_USERNAME=admin \
  -e MONGO_INITDB_ROOT_PASSWORD=password \
  -v mongodb_data:/data/db \
  mongo:7

# Ubuntu
sudo apt-get install -y mongodb
sudo systemctl start mongodb
```

### MongoDB Atlas (Cloud)

1. Go to [mongodb.com/atlas](https://www.mongodb.com/atlas)
2. Create free cluster
3. Get connection string:
   ```
   mongodb+srv://username:password@cluster0.xxxxx.mongodb.net/dbname
   ```

### MongoDB Shell (mongosh)

```bash
# Connect to local MongoDB
mongosh

# Connect to Atlas
mongosh "mongodb+srv://cluster0.xxxxx.mongodb.net/mydb" --username admin

# Basic commands
show dbs          # List databases
use mydb          # Switch/create database
show collections  # List collections
db.stats()        # Database statistics
exit              # Exit shell
```

---

## CRUD Operations

### Create (Insert)

```javascript
// Insert one document
db.users.insertOne({
  name: "Alice",
  email: "alice@example.com",
  age: 25,
  role: "user"
});

// Insert multiple documents
db.users.insertMany([
  { name: "Bob", email: "bob@example.com", age: 30 },
  { name: "Carol", email: "carol@example.com", age: 28 },
  { name: "Dave", email: "dave@example.com", age: 35 }
]);

// Returns inserted ID(s)
// { acknowledged: true, insertedId: ObjectId("...") }
```

### Read (Find)

```javascript
// Find all documents
db.users.find();

// Find with filter
db.users.find({ role: "user" });

// Find one
db.users.findOne({ email: "alice@example.com" });

// Find by ID
db.users.findOne({ _id: ObjectId("507f1f77bcf86cd799439011") });

// Projection (select fields)
db.users.find({}, { name: 1, email: 1, _id: 0 });

// Pretty print
db.users.find().pretty();
```

### Update

```javascript
// Update one document
db.users.updateOne(
  { email: "alice@example.com" },  // Filter
  { $set: { age: 26, role: "admin" } }  // Update
);

// Update multiple documents
db.users.updateMany(
  { role: "user" },
  { $set: { active: true } }
);

// Replace entire document
db.users.replaceOne(
  { email: "alice@example.com" },
  { name: "Alice", email: "alice@example.com", age: 26, role: "admin" }
);

// Upsert (update or insert)
db.users.updateOne(
  { email: "new@example.com" },
  { $set: { name: "New User", email: "new@example.com" } },
  { upsert: true }
);
```

### Delete

```javascript
// Delete one document
db.users.deleteOne({ email: "alice@example.com" });

// Delete multiple documents
db.users.deleteMany({ role: "user" });

// Delete all documents in collection
db.users.deleteMany({});

// Drop entire collection
db.users.drop();
```

---

## Query Operators

### Comparison Operators

```javascript
// Equal
db.users.find({ age: 30 });
db.users.find({ age: { $eq: 30 } });

// Not equal
db.users.find({ age: { $ne: 30 } });

// Greater than
db.users.find({ age: { $gt: 25 } });

// Greater than or equal
db.users.find({ age: { $gte: 25 } });

// Less than
db.users.find({ age: { $lt: 35 } });

// Less than or equal
db.users.find({ age: { $lte: 35 } });

// In array
db.users.find({ role: { $in: ["admin", "moderator"] } });

// Not in array
db.users.find({ role: { $nin: ["banned", "suspended"] } });
```

### Logical Operators

```javascript
// AND (implicit)
db.users.find({ age: { $gte: 25 }, role: "user" });

// AND (explicit)
db.users.find({
  $and: [
    { age: { $gte: 25 } },
    { role: "user" }
  ]
});

// OR
db.users.find({
  $or: [
    { role: "admin" },
    { age: { $gt: 30 } }
  ]
});

// NOT
db.users.find({
  age: { $not: { $gt: 30 } }
});

// NOR (none of the conditions)
db.users.find({
  $nor: [
    { role: "banned" },
    { active: false }
  ]
});
```

### Element Operators

```javascript
// Field exists
db.users.find({ phone: { $exists: true } });
db.users.find({ phone: { $exists: false } });

// Field type
db.users.find({ age: { $type: "number" } });
db.users.find({ tags: { $type: "array" } });
```

### Array Operators

```javascript
// Array contains value
db.posts.find({ tags: "mongodb" });

// Array contains all values
db.posts.find({ tags: { $all: ["mongodb", "nodejs"] } });

// Array size
db.posts.find({ tags: { $size: 3 } });

// Array element match
db.users.find({
  scores: { $elemMatch: { $gte: 80, $lt: 90 } }
});
```

### String Operators

```javascript
// Regex search
db.users.find({ name: { $regex: /john/i } });  // Case insensitive
db.users.find({ email: { $regex: /@example\.com$/ } });

// Text search (requires text index)
db.articles.createIndex({ title: "text", content: "text" });
db.articles.find({ $text: { $search: "mongodb tutorial" } });
```

---

## Update Operators

### Field Update Operators

```javascript
// Set field value
db.users.updateOne(
  { _id: userId },
  { $set: { name: "New Name", "address.city": "Boston" } }
);

// Unset (remove) field
db.users.updateOne(
  { _id: userId },
  { $unset: { temporaryField: "" } }
);

// Increment numeric field
db.posts.updateOne(
  { _id: postId },
  { $inc: { views: 1, likes: 5 } }
);

// Multiply
db.products.updateOne(
  { _id: productId },
  { $mul: { price: 1.1 } }  // 10% increase
);

// Rename field
db.users.updateMany(
  {},
  { $rename: { "name": "fullName" } }
);

// Min/Max (only update if value is less/greater)
db.products.updateOne(
  { _id: productId },
  { $min: { lowPrice: 10 }, $max: { highPrice: 100 } }
);

// Current date
db.users.updateOne(
  { _id: userId },
  { $currentDate: { lastModified: true, lastLogin: { $type: "timestamp" } } }
);
```

### Array Update Operators

```javascript
// Push to array
db.posts.updateOne(
  { _id: postId },
  { $push: { tags: "newTag" } }
);

// Push multiple
db.posts.updateOne(
  { _id: postId },
  { $push: { tags: { $each: ["tag1", "tag2"] } } }
);

// Push with position and slice
db.posts.updateOne(
  { _id: postId },
  {
    $push: {
      comments: {
        $each: [{ text: "New comment" }],
        $position: 0,  // Add at beginning
        $slice: 10     // Keep only 10 items
      }
    }
  }
);

// Add to set (only if not exists)
db.posts.updateOne(
  { _id: postId },
  { $addToSet: { tags: "uniqueTag" } }
);

// Pull (remove from array)
db.posts.updateOne(
  { _id: postId },
  { $pull: { tags: "oldTag" } }
);

// Pull with condition
db.posts.updateOne(
  { _id: postId },
  { $pull: { comments: { author: "spammer" } } }
);

// Pop (remove first or last)
db.posts.updateOne(
  { _id: postId },
  { $pop: { tags: 1 } }  // -1 for first, 1 for last
);

// Update specific array element
db.posts.updateOne(
  { _id: postId, "comments._id": commentId },
  { $set: { "comments.$.text": "Updated comment" } }
);
```

---

## Aggregation Pipeline

### Basic Aggregation

```javascript
// Match and group
db.orders.aggregate([
  { $match: { status: "completed" } },
  { $group: {
      _id: "$customerId",
      totalSpent: { $sum: "$total" },
      orderCount: { $sum: 1 }
    }
  }
]);

// Full pipeline example
db.orders.aggregate([
  // Stage 1: Filter
  { $match: {
      createdAt: { $gte: ISODate("2024-01-01") }
    }
  },
  // Stage 2: Lookup (join)
  { $lookup: {
      from: "users",
      localField: "customerId",
      foreignField: "_id",
      as: "customer"
    }
  },
  // Stage 3: Unwind array
  { $unwind: "$customer" },
  // Stage 4: Group
  { $group: {
      _id: "$customer.country",
      revenue: { $sum: "$total" },
      orders: { $sum: 1 }
    }
  },
  // Stage 5: Sort
  { $sort: { revenue: -1 } },
  // Stage 6: Limit
  { $limit: 10 },
  // Stage 7: Project (reshape)
  { $project: {
      country: "$_id",
      revenue: 1,
      orders: 1,
      avgOrder: { $divide: ["$revenue", "$orders"] }
    }
  }
]);
```

### Common Aggregation Stages

```javascript
// $project - Select/reshape fields
{ $project: { name: 1, email: 1, fullName: { $concat: ["$firstName", " ", "$lastName"] } } }

// $match - Filter documents
{ $match: { status: "active", age: { $gte: 18 } } }

// $group - Group and aggregate
{ $group: { _id: "$category", count: { $sum: 1 }, avgPrice: { $avg: "$price" } } }

// $sort - Order documents
{ $sort: { createdAt: -1, name: 1 } }

// $limit - Limit results
{ $limit: 10 }

// $skip - Skip documents
{ $skip: 20 }

// $lookup - Join collections
{ $lookup: { from: "orders", localField: "_id", foreignField: "userId", as: "orders" } }

// $unwind - Deconstruct array
{ $unwind: "$tags" }

// $addFields - Add new fields
{ $addFields: { totalPrice: { $multiply: ["$price", "$quantity"] } } }

// $count - Count documents
{ $count: "total" }
```

---

## Indexing

### Create Indexes

```javascript
// Single field index
db.users.createIndex({ email: 1 });  // 1 = ascending, -1 = descending

// Compound index
db.posts.createIndex({ authorId: 1, createdAt: -1 });

// Unique index
db.users.createIndex({ email: 1 }, { unique: true });

// Text index
db.articles.createIndex({ title: "text", content: "text" });

// TTL index (auto-delete after time)
db.sessions.createIndex({ createdAt: 1 }, { expireAfterSeconds: 3600 });

// Partial index
db.orders.createIndex(
  { customerId: 1 },
  { partialFilterExpression: { status: "active" } }
);
```

### Manage Indexes

```javascript
// List indexes
db.users.getIndexes();

// Drop index
db.users.dropIndex("email_1");

// Drop all indexes
db.users.dropIndexes();

// Explain query (check index usage)
db.users.find({ email: "test@example.com" }).explain("executionStats");
```

---

## Node.js with Mongoose

### Setup

```bash
npm install mongoose
```

### Connection

```javascript
// src/lib/mongoose.js
const mongoose = require('mongoose');

const connectDB = async () => {
  try {
    await mongoose.connect(process.env.MONGODB_URI, {
      // Options are now mostly automatic in Mongoose 6+
    });
    console.log('MongoDB connected');
  } catch (error) {
    console.error('MongoDB connection error:', error);
    process.exit(1);
  }
};

// Handle connection events
mongoose.connection.on('disconnected', () => {
  console.log('MongoDB disconnected');
});

mongoose.connection.on('error', (err) => {
  console.error('MongoDB error:', err);
});

module.exports = connectDB;
```

### Defining Schemas

```javascript
// src/models/User.js
const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({
  name: {
    type: String,
    required: [true, 'Name is required'],
    trim: true,
    minlength: [2, 'Name must be at least 2 characters'],
    maxlength: [100, 'Name cannot exceed 100 characters'],
  },
  email: {
    type: String,
    required: [true, 'Email is required'],
    unique: true,
    lowercase: true,
    match: [/^\S+@\S+\.\S+$/, 'Please enter a valid email'],
  },
  password: {
    type: String,
    required: true,
    minlength: 8,
    select: false,  // Don't include in queries by default
  },
  role: {
    type: String,
    enum: ['user', 'admin', 'moderator'],
    default: 'user',
  },
  profile: {
    bio: String,
    avatar: String,
    website: String,
  },
  tags: [String],
  isActive: {
    type: Boolean,
    default: true,
  },
}, {
  timestamps: true,  // Adds createdAt and updatedAt
  toJSON: { virtuals: true },
  toObject: { virtuals: true },
});

// Index
userSchema.index({ email: 1 });
userSchema.index({ 'profile.website': 1 }, { sparse: true });

// Virtual field
userSchema.virtual('posts', {
  ref: 'Post',
  localField: '_id',
  foreignField: 'author',
});

// Instance method
userSchema.methods.comparePassword = async function(candidatePassword) {
  return bcrypt.compare(candidatePassword, this.password);
};

// Static method
userSchema.statics.findByEmail = function(email) {
  return this.findOne({ email: email.toLowerCase() });
};

// Pre-save hook
userSchema.pre('save', async function(next) {
  if (!this.isModified('password')) return next();
  this.password = await bcrypt.hash(this.password, 12);
  next();
});

// Pre-find hook
userSchema.pre(/^find/, function(next) {
  this.find({ isActive: { $ne: false } });
  next();
});

module.exports = mongoose.model('User', userSchema);
```

### CRUD with Mongoose

```javascript
const User = require('./models/User');

// Create
const user = await User.create({
  name: 'Alice',
  email: 'alice@example.com',
  password: 'securepassword',
});

// Find
const users = await User.find({ role: 'user' });
const user = await User.findById(userId);
const user = await User.findOne({ email: 'alice@example.com' });

// Update
const user = await User.findByIdAndUpdate(
  userId,
  { name: 'Alice Updated' },
  { new: true, runValidators: true }
);

// Delete
await User.findByIdAndDelete(userId);
await User.deleteMany({ isActive: false });

// Query builder
const users = await User.find()
  .where('age').gte(18)
  .where('role').equals('user')
  .select('name email')
  .sort('-createdAt')
  .limit(10)
  .populate('posts');
```

### Population (Joins)

```javascript
// Post schema with reference
const postSchema = new mongoose.Schema({
  title: String,
  content: String,
  author: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true,
  },
  comments: [{
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Comment',
  }],
});

// Populate author
const post = await Post.findById(postId)
  .populate('author', 'name email');

// Deep populate
const post = await Post.findById(postId)
  .populate({
    path: 'comments',
    populate: {
      path: 'author',
      select: 'name avatar',
    },
  });
```

---

## Schema Design Patterns

### Embedding vs Referencing

```javascript
// Embedded (denormalized) - Good for 1:few, data accessed together
const userSchema = new mongoose.Schema({
  name: String,
  addresses: [{  // Embedded
    street: String,
    city: String,
    country: String,
  }],
});

// Referenced (normalized) - Good for 1:many, independent access
const userSchema = new mongoose.Schema({
  name: String,
  posts: [{
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Post',
  }],
});

// Hybrid - Embed summary, reference full
const postSchema = new mongoose.Schema({
  title: String,
  author: {
    _id: mongoose.Schema.Types.ObjectId,
    name: String,  // Denormalized for quick access
  },
});
```

### When to Embed

- Data is queried together frequently
- One-to-few relationship
- Data doesn't change often
- No need to access embedded data independently

### When to Reference

- Data is large or unbounded
- Data is accessed independently
- Many-to-many relationships
- Data changes frequently

---

## Key Takeaways

1. **Documents are flexible** - No fixed schema, but use Mongoose for validation
2. **Embed for performance** - Data accessed together should live together
3. **Reference for scale** - Large or growing data should be separate
4. **Index your queries** - Check explain() to verify index usage
5. **Use aggregation** - Powerful for analytics and complex queries
6. **Mongoose adds structure** - Schemas, validation, middleware
7. **Design for queries** - Structure data based on access patterns

---

## What's Next?

Tomorrow we'll learn about **Redis Caching** - implementing high-performance caching for your applications!
