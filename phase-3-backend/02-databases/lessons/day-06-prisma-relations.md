# Day 6: Prisma Relations

## Introduction

Relations are at the heart of relational databases. Prisma makes working with relations intuitive and type-safe. This lesson covers defining relations in your schema, querying related data, and performing nested writes across multiple tables.

## Learning Objectives

By the end of this lesson, you will:
- Define all types of relations in Prisma schema
- Query related data with include and select
- Perform nested creates, updates, and connects
- Filter by relation conditions
- Handle many-to-many relations
- Use relation aggregations

---

## Relation Types in Prisma

### One-to-One (1:1)

One record relates to exactly one record in another table.

```prisma
model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  profile   Profile? // Optional one-to-one
}

model Profile {
  id        Int    @id @default(autoincrement())
  bio       String
  avatar    String?
  user      User   @relation(fields: [userId], references: [id])
  userId    Int    @unique // @unique makes it one-to-one
}
```

```typescript
// Create user with profile
const user = await prisma.user.create({
  data: {
    email: 'alice@example.com',
    profile: {
      create: {
        bio: 'Software developer',
        avatar: 'avatar.jpg',
      },
    },
  },
  include: { profile: true },
});

// Query user with profile
const userWithProfile = await prisma.user.findUnique({
  where: { id: 1 },
  include: { profile: true },
});

// Query profile with user
const profile = await prisma.profile.findUnique({
  where: { userId: 1 },
  include: { user: true },
});
```

### One-to-Many (1:N)

One record relates to many records in another table.

```prisma
model User {
  id    Int    @id @default(autoincrement())
  email String @unique
  posts Post[] // One user has many posts
}

model Post {
  id       Int    @id @default(autoincrement())
  title    String
  author   User   @relation(fields: [authorId], references: [id])
  authorId Int    // Foreign key
}
```

```typescript
// Create user with posts
const user = await prisma.user.create({
  data: {
    email: 'bob@example.com',
    posts: {
      create: [
        { title: 'Post 1' },
        { title: 'Post 2' },
      ],
    },
  },
  include: { posts: true },
});

// Create post for existing user
const post = await prisma.post.create({
  data: {
    title: 'New Post',
    author: {
      connect: { id: 1 },
    },
  },
});

// Or using foreign key directly
const post2 = await prisma.post.create({
  data: {
    title: 'Another Post',
    authorId: 1,
  },
});
```

### Many-to-Many (M:N)

Many records relate to many records. Prisma supports implicit and explicit many-to-many.

#### Implicit Many-to-Many

Prisma creates the join table automatically.

```prisma
model Post {
  id    Int    @id @default(autoincrement())
  title String
  tags  Tag[]  // Many posts have many tags
}

model Tag {
  id    Int    @id @default(autoincrement())
  name  String @unique
  posts Post[] // Many tags have many posts
}
```

```typescript
// Create post with tags
const post = await prisma.post.create({
  data: {
    title: 'TypeScript Guide',
    tags: {
      create: [
        { name: 'typescript' },
        { name: 'programming' },
      ],
    },
  },
  include: { tags: true },
});

// Connect existing tags
const post2 = await prisma.post.create({
  data: {
    title: 'Another Guide',
    tags: {
      connect: [
        { name: 'typescript' },
        { id: 5 },
      ],
    },
  },
});

// Mix create and connect
const post3 = await prisma.post.create({
  data: {
    title: 'Mixed Post',
    tags: {
      create: { name: 'new-tag' },
      connect: { name: 'typescript' },
    },
  },
});
```

#### Explicit Many-to-Many

You define the join table yourself (needed when join table has extra fields).

```prisma
model Post {
  id    Int           @id @default(autoincrement())
  title String
  tags  PostTag[]
}

model Tag {
  id    Int           @id @default(autoincrement())
  name  String        @unique
  posts PostTag[]
}

// Explicit join table with extra fields
model PostTag {
  post      Post     @relation(fields: [postId], references: [id])
  postId    Int
  tag       Tag      @relation(fields: [tagId], references: [id])
  tagId     Int
  assignedAt DateTime @default(now())
  assignedBy String?

  @@id([postId, tagId]) // Composite primary key
}
```

```typescript
// Create with explicit join table
const post = await prisma.post.create({
  data: {
    title: 'New Post',
    tags: {
      create: {
        assignedBy: 'admin',
        tag: {
          connect: { name: 'typescript' },
        },
      },
    },
  },
  include: {
    tags: {
      include: { tag: true },
    },
  },
});

// Query posts with tag info
const posts = await prisma.post.findMany({
  include: {
    tags: {
      include: { tag: true },
      orderBy: { assignedAt: 'desc' },
    },
  },
});
```

### Self-Relations

A model that references itself.

```prisma
model Employee {
  id         Int        @id @default(autoincrement())
  name       String
  managerId  Int?
  manager    Employee?  @relation("EmployeeManager", fields: [managerId], references: [id])
  reports    Employee[] @relation("EmployeeManager")
}

model Comment {
  id       Int       @id @default(autoincrement())
  content  String
  parentId Int?
  parent   Comment?  @relation("CommentReplies", fields: [parentId], references: [id])
  replies  Comment[] @relation("CommentReplies")
}
```

```typescript
// Create employee hierarchy
const ceo = await prisma.employee.create({
  data: { name: 'CEO' },
});

const cto = await prisma.employee.create({
  data: {
    name: 'CTO',
    manager: { connect: { id: ceo.id } },
  },
});

const developer = await prisma.employee.create({
  data: {
    name: 'Developer',
    managerId: cto.id,
  },
});

// Query with hierarchy
const ctoWithReports = await prisma.employee.findUnique({
  where: { id: cto.id },
  include: {
    manager: true,
    reports: true,
  },
});

// Nested comment replies
const comment = await prisma.comment.create({
  data: {
    content: 'Great post!',
    replies: {
      create: [
        { content: 'Thanks!' },
        { content: 'I agree' },
      ],
    },
  },
  include: { replies: true },
});
```

---

## Multiple Relations Between Same Models

```prisma
model User {
  id            Int     @id @default(autoincrement())
  name          String
  writtenPosts  Post[]  @relation("WrittenPosts")
  editedPosts   Post[]  @relation("EditedPosts")
}

model Post {
  id        Int   @id @default(autoincrement())
  title     String
  author    User  @relation("WrittenPosts", fields: [authorId], references: [id])
  authorId  Int
  editor    User? @relation("EditedPosts", fields: [editorId], references: [id])
  editorId  Int?
}
```

```typescript
// Create post with author and editor
const post = await prisma.post.create({
  data: {
    title: 'Reviewed Article',
    author: { connect: { id: 1 } },
    editor: { connect: { id: 2 } },
  },
  include: {
    author: true,
    editor: true,
  },
});

// Get user's written and edited posts
const user = await prisma.user.findUnique({
  where: { id: 1 },
  include: {
    writtenPosts: true,
    editedPosts: true,
  },
});
```

---

## Querying Relations

### Include - Fetch Related Data

```typescript
// Include one relation
const user = await prisma.user.findUnique({
  where: { id: 1 },
  include: { posts: true },
});

// Include multiple relations
const user = await prisma.user.findUnique({
  where: { id: 1 },
  include: {
    posts: true,
    profile: true,
    comments: true,
  },
});

// Nested includes
const user = await prisma.user.findUnique({
  where: { id: 1 },
  include: {
    posts: {
      include: {
        comments: {
          include: {
            author: true,
          },
        },
        tags: true,
      },
    },
  },
});

// Include with filtering
const user = await prisma.user.findUnique({
  where: { id: 1 },
  include: {
    posts: {
      where: { published: true },
      orderBy: { createdAt: 'desc' },
      take: 5,
    },
  },
});
```

### Select with Relations

```typescript
// Select specific fields including relations
const user = await prisma.user.findUnique({
  where: { id: 1 },
  select: {
    id: true,
    name: true,
    email: true,
    posts: {
      select: {
        id: true,
        title: true,
        _count: {
          select: { comments: true },
        },
      },
    },
  },
});

// Deeply nested select
const posts = await prisma.post.findMany({
  select: {
    id: true,
    title: true,
    author: {
      select: {
        name: true,
        profile: {
          select: { avatar: true },
        },
      },
    },
    tags: {
      select: { name: true },
    },
  },
});
```

### Relation Count

```typescript
// Count related records
const users = await prisma.user.findMany({
  include: {
    _count: {
      select: {
        posts: true,
        comments: true,
      },
    },
  },
});
// Result: { ..., _count: { posts: 5, comments: 12 } }

// Count with filter
const users = await prisma.user.findMany({
  include: {
    _count: {
      select: {
        posts: { where: { published: true } },
      },
    },
  },
});

// Order by relation count
const users = await prisma.user.findMany({
  orderBy: {
    posts: { _count: 'desc' },
  },
});
```

---

## Filtering by Relations

### some, every, none

```typescript
// Users with AT LEAST ONE published post
const usersWithPosts = await prisma.user.findMany({
  where: {
    posts: {
      some: { published: true },
    },
  },
});

// Users where ALL posts are published
const usersAllPublished = await prisma.user.findMany({
  where: {
    posts: {
      every: { published: true },
    },
  },
});

// Users with NO posts (or no published posts)
const usersNoPosts = await prisma.user.findMany({
  where: {
    posts: {
      none: { published: true },
    },
  },
});

// Complex relation filters
const users = await prisma.user.findMany({
  where: {
    posts: {
      some: {
        AND: [
          { published: true },
          { views: { gt: 100 } },
          {
            tags: {
              some: { name: 'featured' },
            },
          },
        ],
      },
    },
  },
});
```

### is and isNot

```typescript
// Posts by specific author
const posts = await prisma.post.findMany({
  where: {
    author: {
      is: { email: 'alice@example.com' },
    },
  },
});

// Posts NOT by specific author
const posts = await prisma.post.findMany({
  where: {
    author: {
      isNot: { role: 'ADMIN' },
    },
  },
});

// Posts with no category
const posts = await prisma.post.findMany({
  where: {
    category: null,
  },
});

// Posts with a category
const posts = await prisma.post.findMany({
  where: {
    category: { isNot: null },
  },
});
```

---

## Nested Writes

### Nested Create

```typescript
// Create user with related data
const user = await prisma.user.create({
  data: {
    email: 'new@example.com',
    name: 'New User',
    profile: {
      create: { bio: 'Hello!' },
    },
    posts: {
      create: [
        {
          title: 'First Post',
          tags: {
            create: [{ name: 'intro' }],
            connect: [{ name: 'welcome' }],
          },
        },
      ],
    },
  },
  include: {
    profile: true,
    posts: { include: { tags: true } },
  },
});
```

### Nested Update

```typescript
// Update user and their relations
const user = await prisma.user.update({
  where: { id: 1 },
  data: {
    name: 'Updated Name',
    profile: {
      update: { bio: 'Updated bio' },
    },
    posts: {
      updateMany: {
        where: { published: false },
        data: { published: true },
      },
    },
  },
});

// Update or create profile (upsert)
const user = await prisma.user.update({
  where: { id: 1 },
  data: {
    profile: {
      upsert: {
        create: { bio: 'New profile' },
        update: { bio: 'Updated profile' },
      },
    },
  },
});
```

### Connect, Disconnect, Set

```typescript
// Connect existing records
const post = await prisma.post.update({
  where: { id: 1 },
  data: {
    tags: {
      connect: [{ id: 1 }, { id: 2 }],
    },
  },
});

// Disconnect records
const post = await prisma.post.update({
  where: { id: 1 },
  data: {
    tags: {
      disconnect: [{ id: 1 }],
    },
    category: {
      disconnect: true, // For one-to-one/many
    },
  },
});

// Set (replace all)
const post = await prisma.post.update({
  where: { id: 1 },
  data: {
    tags: {
      set: [{ id: 3 }, { id: 4 }], // Remove all existing, add these
    },
  },
});

// Set empty (remove all)
const post = await prisma.post.update({
  where: { id: 1 },
  data: {
    tags: { set: [] },
  },
});
```

### connectOrCreate

```typescript
// Connect if exists, create if not
const post = await prisma.post.create({
  data: {
    title: 'New Post',
    authorId: 1,
    tags: {
      connectOrCreate: [
        {
          where: { name: 'typescript' },
          create: { name: 'typescript' },
        },
        {
          where: { name: 'prisma' },
          create: { name: 'prisma' },
        },
      ],
    },
  },
});
```

### Nested Delete

```typescript
// Delete related records
const user = await prisma.user.update({
  where: { id: 1 },
  data: {
    posts: {
      delete: [{ id: 1 }, { id: 2 }],
    },
  },
});

// Delete many by condition
const user = await prisma.user.update({
  where: { id: 1 },
  data: {
    posts: {
      deleteMany: { published: false },
    },
  },
});

// Delete with cascade (if defined in schema)
const user = await prisma.user.delete({
  where: { id: 1 },
  // All related posts deleted if onDelete: Cascade
});
```

---

## Practical Examples

### Blog Post with Full Relations

```typescript
// Create complete blog post
const post = await prisma.post.create({
  data: {
    title: 'Complete Guide to Prisma',
    slug: 'complete-guide-to-prisma',
    content: 'Full content here...',
    excerpt: 'Learn Prisma from scratch',
    published: true,
    publishedAt: new Date(),
    author: {
      connect: { email: 'author@example.com' },
    },
    category: {
      connectOrCreate: {
        where: { slug: 'tutorials' },
        create: { name: 'Tutorials', slug: 'tutorials' },
      },
    },
    tags: {
      connectOrCreate: [
        { where: { slug: 'prisma' }, create: { name: 'Prisma', slug: 'prisma' } },
        { where: { slug: 'database' }, create: { name: 'Database', slug: 'database' } },
      ],
    },
  },
  include: {
    author: { select: { id: true, name: true } },
    category: true,
    tags: true,
  },
});
```

### User Dashboard Data

```typescript
// Get user with all their activity
const userDashboard = await prisma.user.findUnique({
  where: { id: userId },
  select: {
    id: true,
    name: true,
    email: true,
    profile: {
      select: { bio: true, avatar: true },
    },
    posts: {
      where: { published: true },
      orderBy: { createdAt: 'desc' },
      take: 5,
      select: {
        id: true,
        title: true,
        slug: true,
        views: true,
        _count: { select: { comments: true, likes: true } },
      },
    },
    comments: {
      orderBy: { createdAt: 'desc' },
      take: 10,
      select: {
        id: true,
        content: true,
        createdAt: true,
        post: { select: { title: true, slug: true } },
      },
    },
    _count: {
      select: {
        posts: true,
        comments: true,
        likes: true,
      },
    },
  },
});
```

### Feed with Complex Relations

```typescript
// Get personalized feed
const feed = await prisma.post.findMany({
  where: {
    published: true,
    OR: [
      // Posts from followed users
      {
        author: {
          followers: {
            some: { followerId: currentUserId },
          },
        },
      },
      // Posts with followed tags
      {
        tags: {
          some: {
            followers: {
              some: { userId: currentUserId },
            },
          },
        },
      },
    ],
  },
  orderBy: { publishedAt: 'desc' },
  take: 20,
  include: {
    author: {
      select: { id: true, name: true, avatar: true },
    },
    tags: { select: { name: true, slug: true } },
    _count: { select: { comments: true, likes: true } },
    likes: {
      where: { userId: currentUserId },
      select: { id: true },
    },
  },
});

// Add isLiked flag
const feedWithLikes = feed.map(post => ({
  ...post,
  isLiked: post.likes.length > 0,
  likes: undefined, // Remove likes array
}));
```

---

## Referential Actions

### onDelete and onUpdate

```prisma
model Post {
  id       Int    @id @default(autoincrement())
  author   User   @relation(fields: [authorId], references: [id], onDelete: Cascade)
  authorId Int

  category Category? @relation(fields: [categoryId], references: [id], onDelete: SetNull)
  categoryId Int?
}

model Comment {
  id     Int  @id @default(autoincrement())
  post   Post @relation(fields: [postId], references: [id], onDelete: Cascade)
  postId Int

  author User @relation(fields: [authorId], references: [id], onDelete: Restrict)
  authorId Int
}
```

**Options:**
- `Cascade` - Delete related records
- `SetNull` - Set foreign key to null
- `SetDefault` - Set to default value
- `Restrict` - Prevent deletion (default)
- `NoAction` - Similar to Restrict

---

## Key Takeaways

1. **Relations are type-safe** - Prisma validates relation queries at compile time
2. **Use include for loading relations** - Don't query related data separately
3. **Filter with some/every/none** - Powerful relation filtering
4. **connectOrCreate is powerful** - Avoid manual existence checks
5. **Set cascades carefully** - onDelete: Cascade can be dangerous
6. **Count without loading** - Use _count for efficient relation counting
7. **Nested writes are atomic** - All operations in one transaction

---

## What's Next?

Tomorrow we'll explore **Advanced Prisma** - transactions, raw queries, middleware, and performance optimization!
