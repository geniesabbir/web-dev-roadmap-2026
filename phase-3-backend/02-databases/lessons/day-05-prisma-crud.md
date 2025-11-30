# Day 5: Prisma CRUD Operations

## Introduction

Now that you have Prisma set up, it's time to learn how to perform CRUD (Create, Read, Update, Delete) operations. Prisma Client provides a type-safe and intuitive API for all database operations with full autocomplete support.

## Learning Objectives

By the end of this lesson, you will:
- Create records with `create()` and `createMany()`
- Read records with `findUnique()`, `findFirst()`, and `findMany()`
- Update records with `update()` and `updateMany()`
- Delete records with `delete()` and `deleteMany()`
- Use filtering, sorting, and pagination
- Perform upserts (update or create)

---

## Setup Reminder

```typescript
// src/lib/prisma.ts
import { PrismaClient } from '@prisma/client';

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined;
};

export const prisma =
  globalForPrisma.prisma ??
  new PrismaClient({
    log: ['query'], // Log queries in development
  });

if (process.env.NODE_ENV !== 'production') {
  globalForPrisma.prisma = prisma;
}

export default prisma;
```

---

## Create Operations

### create() - Single Record

```typescript
import prisma from './lib/prisma';

// Create a user
const user = await prisma.user.create({
  data: {
    email: 'alice@example.com',
    name: 'Alice Johnson',
    password: 'hashedpassword',
  },
});

console.log(user);
// { id: 1, email: 'alice@example.com', name: 'Alice Johnson', ... }
```

### Create with Relation

```typescript
// Create user with posts (nested create)
const userWithPosts = await prisma.user.create({
  data: {
    email: 'bob@example.com',
    name: 'Bob Smith',
    password: 'hashedpassword',
    posts: {
      create: [
        {
          title: 'First Post',
          slug: 'first-post',
          content: 'Hello World!',
          published: true,
        },
        {
          title: 'Second Post',
          slug: 'second-post',
          content: 'Another post',
        },
      ],
    },
  },
  include: {
    posts: true, // Include posts in response
  },
});
```

### Create with Connect (Existing Relation)

```typescript
// Create post connected to existing user and tags
const post = await prisma.post.create({
  data: {
    title: 'New Post',
    slug: 'new-post',
    content: 'Content here...',
    author: {
      connect: { id: 1 }, // Connect to existing user
    },
    category: {
      connect: { slug: 'technology' }, // Connect by unique field
    },
    tags: {
      connect: [
        { id: 1 },
        { id: 2 },
      ],
    },
  },
});

// Or connect by id directly
const post2 = await prisma.post.create({
  data: {
    title: 'Another Post',
    slug: 'another-post',
    content: 'More content...',
    authorId: 1, // Direct foreign key
    categoryId: 1,
  },
});
```

### createMany() - Multiple Records

```typescript
// Create multiple users at once (more efficient)
const result = await prisma.user.createMany({
  data: [
    { email: 'user1@example.com', name: 'User 1', password: 'hash1' },
    { email: 'user2@example.com', name: 'User 2', password: 'hash2' },
    { email: 'user3@example.com', name: 'User 3', password: 'hash3' },
  ],
  skipDuplicates: true, // Skip records with duplicate unique fields
});

console.log(result);
// { count: 3 }

// Note: createMany doesn't return created records
// Use create() in a loop if you need the records back
```

### Create with Select

```typescript
// Only return specific fields
const user = await prisma.user.create({
  data: {
    email: 'carol@example.com',
    name: 'Carol Davis',
    password: 'hashedpassword',
  },
  select: {
    id: true,
    email: true,
    name: true,
    // password excluded
  },
});
```

---

## Read Operations

### findUnique() - Single Record by Unique Field

```typescript
// Find by primary key
const user = await prisma.user.findUnique({
  where: { id: 1 },
});

// Find by unique field
const userByEmail = await prisma.user.findUnique({
  where: { email: 'alice@example.com' },
});

// Find by compound unique key
const like = await prisma.like.findUnique({
  where: {
    userId_postId: {
      userId: 1,
      postId: 1,
    },
  },
});

// Returns null if not found
if (!user) {
  console.log('User not found');
}
```

### findUniqueOrThrow()

```typescript
// Throws error if not found
try {
  const user = await prisma.user.findUniqueOrThrow({
    where: { id: 999 },
  });
} catch (error) {
  // PrismaClientKnownRequestError with code P2025
  console.log('User not found');
}
```

### findFirst() - First Matching Record

```typescript
// Find first user with a condition
const admin = await prisma.user.findFirst({
  where: { role: 'ADMIN' },
});

// With ordering
const latestPost = await prisma.post.findFirst({
  where: { published: true },
  orderBy: { publishedAt: 'desc' },
});
```

### findMany() - Multiple Records

```typescript
// Find all users
const allUsers = await prisma.user.findMany();

// With conditions
const activeUsers = await prisma.user.findMany({
  where: { isActive: true },
});

// With pagination
const paginatedUsers = await prisma.user.findMany({
  skip: 10,  // Offset
  take: 10,  // Limit
});

// With ordering
const sortedUsers = await prisma.user.findMany({
  orderBy: { createdAt: 'desc' },
});

// Multiple order fields
const users = await prisma.user.findMany({
  orderBy: [
    { role: 'asc' },
    { name: 'asc' },
  ],
});
```

### Select and Include

```typescript
// Select specific fields only
const users = await prisma.user.findMany({
  select: {
    id: true,
    email: true,
    name: true,
    // Other fields excluded
  },
});

// Include relations
const usersWithPosts = await prisma.user.findMany({
  include: {
    posts: true,
  },
});

// Select fields from relations
const usersWithPostTitles = await prisma.user.findMany({
  select: {
    id: true,
    name: true,
    posts: {
      select: {
        id: true,
        title: true,
      },
    },
  },
});

// Filter included relations
const usersWithPublishedPosts = await prisma.user.findMany({
  include: {
    posts: {
      where: { published: true },
      orderBy: { createdAt: 'desc' },
      take: 5,
    },
  },
});
```

### count()

```typescript
// Count all records
const totalUsers = await prisma.user.count();

// Count with conditions
const activeUsers = await prisma.user.count({
  where: { isActive: true },
});

// Count with select (multiple counts)
const stats = await prisma.user.count({
  select: {
    _all: true,
    email: true, // Count non-null emails
  },
});
```

---

## Filtering (Where Clause)

### Basic Filters

```typescript
// Equals
const user = await prisma.user.findMany({
  where: { name: 'Alice' },
});

// Not equals
const users = await prisma.user.findMany({
  where: { role: { not: 'ADMIN' } },
});

// In array
const users = await prisma.user.findMany({
  where: { role: { in: ['ADMIN', 'AUTHOR'] } },
});

// Not in array
const users = await prisma.user.findMany({
  where: { role: { notIn: ['USER'] } },
});
```

### Comparison Filters

```typescript
// Greater than
const posts = await prisma.post.findMany({
  where: { views: { gt: 100 } },
});

// Greater than or equal
const posts = await prisma.post.findMany({
  where: { views: { gte: 100 } },
});

// Less than
const posts = await prisma.post.findMany({
  where: { views: { lt: 50 } },
});

// Less than or equal
const posts = await prisma.post.findMany({
  where: { views: { lte: 50 } },
});

// Combining comparisons
const posts = await prisma.post.findMany({
  where: {
    views: { gte: 10, lte: 100 },
  },
});
```

### String Filters

```typescript
// Contains (case-sensitive)
const users = await prisma.user.findMany({
  where: { name: { contains: 'john' } },
});

// Contains (case-insensitive)
const users = await prisma.user.findMany({
  where: { name: { contains: 'john', mode: 'insensitive' } },
});

// Starts with
const users = await prisma.user.findMany({
  where: { email: { startsWith: 'admin' } },
});

// Ends with
const users = await prisma.user.findMany({
  where: { email: { endsWith: '@example.com' } },
});
```

### Null Filters

```typescript
// Is null
const usersWithoutBio = await prisma.user.findMany({
  where: { bio: null },
});

// Is not null
const usersWithBio = await prisma.user.findMany({
  where: { bio: { not: null } },
});

// Or using isSet (for optional fields)
const users = await prisma.user.findMany({
  where: { bio: { not: null } },
});
```

### Date Filters

```typescript
// Exact date
const posts = await prisma.post.findMany({
  where: {
    publishedAt: new Date('2024-01-01'),
  },
});

// Date range
const posts = await prisma.post.findMany({
  where: {
    publishedAt: {
      gte: new Date('2024-01-01'),
      lt: new Date('2024-02-01'),
    },
  },
});

// Posts from last 7 days
const recentPosts = await prisma.post.findMany({
  where: {
    publishedAt: {
      gte: new Date(Date.now() - 7 * 24 * 60 * 60 * 1000),
    },
  },
});
```

### Logical Operators

```typescript
// AND (implicit)
const posts = await prisma.post.findMany({
  where: {
    published: true,
    views: { gt: 100 },
  },
});

// AND (explicit)
const posts = await prisma.post.findMany({
  where: {
    AND: [
      { published: true },
      { views: { gt: 100 } },
    ],
  },
});

// OR
const posts = await prisma.post.findMany({
  where: {
    OR: [
      { title: { contains: 'prisma' } },
      { content: { contains: 'prisma' } },
    ],
  },
});

// NOT
const users = await prisma.user.findMany({
  where: {
    NOT: { role: 'ADMIN' },
  },
});

// Complex combination
const posts = await prisma.post.findMany({
  where: {
    AND: [
      { published: true },
      {
        OR: [
          { title: { contains: 'typescript' } },
          { title: { contains: 'javascript' } },
        ],
      },
    ],
  },
});
```

### Relation Filters

```typescript
// Posts by author name
const posts = await prisma.post.findMany({
  where: {
    author: {
      name: { contains: 'john' },
    },
  },
});

// Users with published posts
const users = await prisma.user.findMany({
  where: {
    posts: {
      some: { published: true },
    },
  },
});

// Users with NO published posts
const users = await prisma.user.findMany({
  where: {
    posts: {
      none: { published: true },
    },
  },
});

// Users where ALL posts are published
const users = await prisma.user.findMany({
  where: {
    posts: {
      every: { published: true },
    },
  },
});

// Posts with specific tags
const posts = await prisma.post.findMany({
  where: {
    tags: {
      some: {
        name: { in: ['javascript', 'typescript'] },
      },
    },
  },
});
```

---

## Update Operations

### update() - Single Record

```typescript
// Update by unique field
const user = await prisma.user.update({
  where: { id: 1 },
  data: { name: 'Alice Updated' },
});

// Update multiple fields
const user = await prisma.user.update({
  where: { email: 'alice@example.com' },
  data: {
    name: 'Alice Johnson Updated',
    bio: 'New bio here',
    isActive: false,
  },
});
```

### Numeric Updates

```typescript
// Increment
const post = await prisma.post.update({
  where: { id: 1 },
  data: {
    views: { increment: 1 },
  },
});

// Decrement
const post = await prisma.post.update({
  where: { id: 1 },
  data: {
    likes: { decrement: 1 },
  },
});

// Multiply
const product = await prisma.product.update({
  where: { id: 1 },
  data: {
    price: { multiply: 1.1 }, // 10% increase
  },
});

// Divide
const product = await prisma.product.update({
  where: { id: 1 },
  data: {
    price: { divide: 2 },
  },
});
```

### Update Relations

```typescript
// Connect to existing relation
const post = await prisma.post.update({
  where: { id: 1 },
  data: {
    category: {
      connect: { id: 2 },
    },
  },
});

// Disconnect relation (set to null)
const post = await prisma.post.update({
  where: { id: 1 },
  data: {
    category: {
      disconnect: true,
    },
  },
});

// Update many-to-many
const post = await prisma.post.update({
  where: { id: 1 },
  data: {
    tags: {
      // Add tags
      connect: [{ id: 3 }, { id: 4 }],
      // Remove tags
      disconnect: [{ id: 1 }],
    },
  },
});

// Set (replace all)
const post = await prisma.post.update({
  where: { id: 1 },
  data: {
    tags: {
      set: [{ id: 1 }, { id: 2 }], // Replace all tags
    },
  },
});
```

### updateMany() - Multiple Records

```typescript
// Update all matching records
const result = await prisma.post.updateMany({
  where: { published: false },
  data: { views: 0 },
});

console.log(result);
// { count: 10 }

// Update with conditions
const result = await prisma.user.updateMany({
  where: {
    lastLogin: {
      lt: new Date(Date.now() - 90 * 24 * 60 * 60 * 1000),
    },
  },
  data: { isActive: false },
});
```

---

## Delete Operations

### delete() - Single Record

```typescript
// Delete by unique field
const user = await prisma.user.delete({
  where: { id: 1 },
});

// Delete by unique field
const user = await prisma.user.delete({
  where: { email: 'alice@example.com' },
});

// Returns deleted record
console.log(user);
```

### deleteMany() - Multiple Records

```typescript
// Delete all matching records
const result = await prisma.post.deleteMany({
  where: { published: false },
});

console.log(result);
// { count: 5 }

// Delete all records (be careful!)
const result = await prisma.session.deleteMany();

// Delete old records
const result = await prisma.log.deleteMany({
  where: {
    createdAt: {
      lt: new Date(Date.now() - 30 * 24 * 60 * 60 * 1000),
    },
  },
});
```

### Soft Delete Pattern

```typescript
// Add to schema:
// deletedAt DateTime?

// "Delete" by setting deletedAt
const user = await prisma.user.update({
  where: { id: 1 },
  data: { deletedAt: new Date() },
});

// Query only non-deleted records
const activeUsers = await prisma.user.findMany({
  where: { deletedAt: null },
});

// Restore
const user = await prisma.user.update({
  where: { id: 1 },
  data: { deletedAt: null },
});
```

---

## Upsert Operations

### upsert() - Update or Create

```typescript
// Create if not exists, update if exists
const user = await prisma.user.upsert({
  where: { email: 'alice@example.com' },
  update: {
    name: 'Alice Updated',
    lastLogin: new Date(),
  },
  create: {
    email: 'alice@example.com',
    name: 'Alice New',
    password: 'hashedpassword',
  },
});

// Useful for sync operations
const profile = await prisma.profile.upsert({
  where: { userId: 1 },
  update: {
    bio: 'Updated bio',
    avatar: 'new-avatar.jpg',
  },
  create: {
    userId: 1,
    bio: 'Initial bio',
  },
});
```

---

## Pagination

### Offset Pagination

```typescript
// Page-based pagination
const page = 2;
const pageSize = 10;

const users = await prisma.user.findMany({
  skip: (page - 1) * pageSize,
  take: pageSize,
  orderBy: { createdAt: 'desc' },
});

// Get total for pagination info
const total = await prisma.user.count();
const totalPages = Math.ceil(total / pageSize);
```

### Cursor Pagination

```typescript
// First page
const firstPage = await prisma.post.findMany({
  take: 10,
  orderBy: { id: 'asc' },
});

// Get cursor from last item
const lastPost = firstPage[firstPage.length - 1];
const cursor = lastPost.id;

// Next page
const nextPage = await prisma.post.findMany({
  take: 10,
  skip: 1, // Skip the cursor itself
  cursor: { id: cursor },
  orderBy: { id: 'asc' },
});

// With where clause
const posts = await prisma.post.findMany({
  take: 10,
  skip: 1,
  cursor: { id: cursor },
  where: { published: true },
  orderBy: { id: 'asc' },
});
```

---

## Practical Examples

### User Service

```typescript
// src/services/user.service.ts
import prisma from '../lib/prisma';
import { Prisma } from '@prisma/client';

export class UserService {
  async findAll(params: {
    page?: number;
    pageSize?: number;
    search?: string;
    role?: string;
  }) {
    const { page = 1, pageSize = 10, search, role } = params;

    const where: Prisma.UserWhereInput = {
      isActive: true,
      ...(search && {
        OR: [
          { name: { contains: search, mode: 'insensitive' } },
          { email: { contains: search, mode: 'insensitive' } },
        ],
      }),
      ...(role && { role: role as any }),
    };

    const [users, total] = await Promise.all([
      prisma.user.findMany({
        where,
        skip: (page - 1) * pageSize,
        take: pageSize,
        orderBy: { createdAt: 'desc' },
        select: {
          id: true,
          email: true,
          name: true,
          role: true,
          createdAt: true,
          _count: {
            select: { posts: true },
          },
        },
      }),
      prisma.user.count({ where }),
    ]);

    return {
      data: users,
      meta: {
        total,
        page,
        pageSize,
        totalPages: Math.ceil(total / pageSize),
      },
    };
  }

  async findById(id: number) {
    return prisma.user.findUnique({
      where: { id },
      include: {
        posts: {
          where: { published: true },
          orderBy: { createdAt: 'desc' },
          take: 5,
        },
        _count: {
          select: { posts: true, comments: true },
        },
      },
    });
  }

  async create(data: Prisma.UserCreateInput) {
    return prisma.user.create({
      data,
      select: {
        id: true,
        email: true,
        name: true,
        role: true,
      },
    });
  }

  async update(id: number, data: Prisma.UserUpdateInput) {
    return prisma.user.update({
      where: { id },
      data,
    });
  }

  async delete(id: number) {
    // Soft delete
    return prisma.user.update({
      where: { id },
      data: { isActive: false },
    });
  }
}
```

### Post Service

```typescript
// src/services/post.service.ts
import prisma from '../lib/prisma';
import { Prisma } from '@prisma/client';

export class PostService {
  async findAll(params: {
    page?: number;
    pageSize?: number;
    search?: string;
    category?: string;
    tag?: string;
    authorId?: number;
    published?: boolean;
  }) {
    const { page = 1, pageSize = 10, search, category, tag, authorId, published = true } = params;

    const where: Prisma.PostWhereInput = {
      published,
      ...(search && {
        OR: [
          { title: { contains: search, mode: 'insensitive' } },
          { content: { contains: search, mode: 'insensitive' } },
        ],
      }),
      ...(category && { category: { slug: category } }),
      ...(tag && { tags: { some: { slug: tag } } }),
      ...(authorId && { authorId }),
    };

    const [posts, total] = await Promise.all([
      prisma.post.findMany({
        where,
        skip: (page - 1) * pageSize,
        take: pageSize,
        orderBy: { publishedAt: 'desc' },
        include: {
          author: {
            select: { id: true, name: true, avatar: true },
          },
          category: {
            select: { id: true, name: true, slug: true },
          },
          tags: {
            select: { id: true, name: true, slug: true },
          },
          _count: {
            select: { comments: true, likes: true },
          },
        },
      }),
      prisma.post.count({ where }),
    ]);

    return {
      data: posts,
      meta: {
        total,
        page,
        pageSize,
        totalPages: Math.ceil(total / pageSize),
      },
    };
  }

  async findBySlug(slug: string) {
    const post = await prisma.post.findUnique({
      where: { slug },
      include: {
        author: {
          select: { id: true, name: true, avatar: true, bio: true },
        },
        category: true,
        tags: true,
        comments: {
          where: { parentId: null },
          orderBy: { createdAt: 'desc' },
          include: {
            author: {
              select: { id: true, name: true, avatar: true },
            },
            replies: {
              include: {
                author: {
                  select: { id: true, name: true, avatar: true },
                },
              },
            },
          },
        },
        _count: {
          select: { likes: true },
        },
      },
    });

    // Increment views
    if (post) {
      await prisma.post.update({
        where: { id: post.id },
        data: { views: { increment: 1 } },
      });
    }

    return post;
  }

  async create(authorId: number, data: {
    title: string;
    slug: string;
    content: string;
    excerpt?: string;
    categoryId?: number;
    tagIds?: number[];
    published?: boolean;
  }) {
    const { tagIds, ...postData } = data;

    return prisma.post.create({
      data: {
        ...postData,
        authorId,
        publishedAt: data.published ? new Date() : null,
        ...(tagIds && {
          tags: {
            connect: tagIds.map(id => ({ id })),
          },
        }),
      },
      include: {
        author: { select: { id: true, name: true } },
        category: true,
        tags: true,
      },
    });
  }

  async toggleLike(postId: number, userId: number) {
    const existingLike = await prisma.like.findUnique({
      where: {
        userId_postId: { userId, postId },
      },
    });

    if (existingLike) {
      await prisma.like.delete({
        where: { id: existingLike.id },
      });
      return { liked: false };
    } else {
      await prisma.like.create({
        data: { userId, postId },
      });
      return { liked: true };
    }
  }
}
```

---

## Key Takeaways

1. **Type-safe queries** - Prisma Client provides full autocomplete and type checking
2. **Use select/include wisely** - Only fetch data you need
3. **Batch operations** - Use `createMany`, `updateMany`, `deleteMany` for bulk operations
4. **Relation queries** - Use `some`, `none`, `every` for filtering by relations
5. **Pagination** - Use cursor pagination for large datasets
6. **Upsert** - Perfect for "create or update" scenarios
7. **Soft delete** - Consider using a `deletedAt` field instead of hard deletes

---

## What's Next?

Tomorrow we'll explore **Prisma Relations** in depth - learning advanced relation queries, nested writes, and handling complex data relationships!
