# Day 5: API Routes - Building Backend in Next.js

## Introduction

Next.js allows you to build API endpoints directly in your application using Route Handlers. Combined with Server Actions, you have powerful tools for handling data mutations, integrations, and backend logic without needing a separate server.

## Learning Objectives

By the end of this lesson, you will:
- Create API Route Handlers
- Handle different HTTP methods
- Work with request/response objects
- Implement authentication patterns
- Use Server Actions for form submissions
- Build RESTful APIs

---

## Route Handlers Basics

### Creating a Route Handler

```tsx
// app/api/hello/route.ts
import { NextResponse } from 'next/server';

export async function GET() {
  return NextResponse.json({ message: 'Hello, World!' });
}
```

### File Convention

```
app/
└── api/
    ├── hello/
    │   └── route.ts      → GET /api/hello
    ├── users/
    │   ├── route.ts      → GET/POST /api/users
    │   └── [id]/
    │       └── route.ts  → GET/PUT/DELETE /api/users/:id
    └── posts/
        └── [slug]/
            └── route.ts  → /api/posts/:slug
```

---

## HTTP Methods

### All Methods in One File

```tsx
// app/api/users/route.ts
import { NextResponse } from 'next/server';
import { db } from '@/lib/db';

// GET /api/users
export async function GET() {
  const users = await db.user.findMany();
  return NextResponse.json(users);
}

// POST /api/users
export async function POST(request: Request) {
  const body = await request.json();

  const user = await db.user.create({
    data: body,
  });

  return NextResponse.json(user, { status: 201 });
}
```

### Dynamic Routes

```tsx
// app/api/users/[id]/route.ts
import { NextResponse } from 'next/server';
import { db } from '@/lib/db';

interface Props {
  params: { id: string };
}

// GET /api/users/:id
export async function GET(request: Request, { params }: Props) {
  const user = await db.user.findUnique({
    where: { id: params.id },
  });

  if (!user) {
    return NextResponse.json(
      { error: 'User not found' },
      { status: 404 }
    );
  }

  return NextResponse.json(user);
}

// PUT /api/users/:id
export async function PUT(request: Request, { params }: Props) {
  const body = await request.json();

  try {
    const user = await db.user.update({
      where: { id: params.id },
      data: body,
    });
    return NextResponse.json(user);
  } catch (error) {
    return NextResponse.json(
      { error: 'User not found' },
      { status: 404 }
    );
  }
}

// DELETE /api/users/:id
export async function DELETE(request: Request, { params }: Props) {
  try {
    await db.user.delete({
      where: { id: params.id },
    });
    return new NextResponse(null, { status: 204 });
  } catch (error) {
    return NextResponse.json(
      { error: 'User not found' },
      { status: 404 }
    );
  }
}

// PATCH /api/users/:id
export async function PATCH(request: Request, { params }: Props) {
  const body = await request.json();

  const user = await db.user.update({
    where: { id: params.id },
    data: body,
  });

  return NextResponse.json(user);
}
```

---

## Request Object

### Query Parameters

```tsx
// GET /api/search?q=hello&page=1
export async function GET(request: Request) {
  const { searchParams } = new URL(request.url);
  const query = searchParams.get('q');
  const page = parseInt(searchParams.get('page') || '1');

  const results = await search(query, page);

  return NextResponse.json(results);
}
```

### Request Body

```tsx
export async function POST(request: Request) {
  // JSON body
  const json = await request.json();

  // Form data
  const formData = await request.formData();
  const name = formData.get('name');

  // Text
  const text = await request.text();

  // Array buffer (binary)
  const buffer = await request.arrayBuffer();
}
```

### Headers

```tsx
export async function GET(request: Request) {
  // Read headers
  const authHeader = request.headers.get('authorization');
  const contentType = request.headers.get('content-type');

  // Create response with headers
  return NextResponse.json(data, {
    headers: {
      'X-Custom-Header': 'value',
      'Cache-Control': 'max-age=3600',
    },
  });
}
```

### Cookies

```tsx
import { cookies } from 'next/headers';

export async function GET() {
  const cookieStore = cookies();

  // Read cookie
  const token = cookieStore.get('token');

  // Set cookie
  cookieStore.set('session', 'abc123', {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'lax',
    maxAge: 60 * 60 * 24 * 7, // 1 week
  });

  // Delete cookie
  cookieStore.delete('token');

  return NextResponse.json({ success: true });
}
```

---

## Response Helpers

### Different Response Types

```tsx
import { NextResponse } from 'next/server';

// JSON response
return NextResponse.json({ data: 'value' });

// JSON with status
return NextResponse.json({ error: 'Not found' }, { status: 404 });

// Redirect
return NextResponse.redirect(new URL('/login', request.url));

// Rewrite (proxy)
return NextResponse.rewrite(new URL('/api/v2/users', request.url));

// Empty response
return new NextResponse(null, { status: 204 });

// Text response
return new NextResponse('Hello', {
  headers: { 'Content-Type': 'text/plain' },
});

// Stream response
const stream = new ReadableStream({
  start(controller) {
    controller.enqueue('Hello');
    controller.enqueue(' World');
    controller.close();
  },
});
return new NextResponse(stream);
```

---

## Error Handling

### Try-Catch Pattern

```tsx
export async function POST(request: Request) {
  try {
    const body = await request.json();

    // Validate
    if (!body.email || !body.name) {
      return NextResponse.json(
        { error: 'Email and name are required' },
        { status: 400 }
      );
    }

    const user = await db.user.create({ data: body });

    return NextResponse.json(user, { status: 201 });
  } catch (error) {
    console.error('Error creating user:', error);

    if (error.code === 'P2002') {
      return NextResponse.json(
        { error: 'Email already exists' },
        { status: 409 }
      );
    }

    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}
```

### Validation with Zod

```tsx
import { z } from 'zod';
import { NextResponse } from 'next/server';

const UserSchema = z.object({
  name: z.string().min(1).max(100),
  email: z.string().email(),
  age: z.number().min(0).max(150).optional(),
});

export async function POST(request: Request) {
  try {
    const body = await request.json();

    const validatedData = UserSchema.parse(body);

    const user = await db.user.create({ data: validatedData });

    return NextResponse.json(user, { status: 201 });
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json(
        { error: 'Validation failed', details: error.errors },
        { status: 400 }
      );
    }

    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}
```

---

## Authentication Patterns

### Basic Auth Check

```tsx
import { NextResponse } from 'next/server';
import { verifyToken } from '@/lib/auth';

export async function GET(request: Request) {
  const authHeader = request.headers.get('authorization');

  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return NextResponse.json(
      { error: 'Unauthorized' },
      { status: 401 }
    );
  }

  const token = authHeader.split(' ')[1];

  try {
    const user = await verifyToken(token);
    // Continue with authenticated request
    return NextResponse.json({ user });
  } catch (error) {
    return NextResponse.json(
      { error: 'Invalid token' },
      { status: 401 }
    );
  }
}
```

### Reusable Auth Middleware

```tsx
// lib/api-auth.ts
import { NextResponse } from 'next/server';
import { verifyToken } from './auth';

export async function withAuth(
  request: Request,
  handler: (request: Request, user: User) => Promise<NextResponse>
) {
  const authHeader = request.headers.get('authorization');

  if (!authHeader?.startsWith('Bearer ')) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  try {
    const token = authHeader.split(' ')[1];
    const user = await verifyToken(token);
    return handler(request, user);
  } catch {
    return NextResponse.json({ error: 'Invalid token' }, { status: 401 });
  }
}

// app/api/protected/route.ts
import { withAuth } from '@/lib/api-auth';

export async function GET(request: Request) {
  return withAuth(request, async (req, user) => {
    // user is authenticated
    const data = await getData(user.id);
    return NextResponse.json(data);
  });
}
```

---

## Server Actions

Server Actions are an alternative to API routes for mutations:

### Basic Server Action

```tsx
// app/actions.ts
'use server';

import { revalidatePath } from 'next/cache';
import { redirect } from 'next/navigation';
import { db } from '@/lib/db';

export async function createPost(formData: FormData) {
  const title = formData.get('title') as string;
  const content = formData.get('content') as string;

  await db.post.create({
    data: { title, content },
  });

  revalidatePath('/posts');
  redirect('/posts');
}
```

### Using in Forms

```tsx
// app/posts/new/page.tsx
import { createPost } from '@/app/actions';

export default function NewPost() {
  return (
    <form action={createPost}>
      <input name="title" required />
      <textarea name="content" required />
      <button type="submit">Create</button>
    </form>
  );
}
```

### With Validation and Error Handling

```tsx
// app/actions.ts
'use server';

import { z } from 'zod';
import { revalidatePath } from 'next/cache';

const PostSchema = z.object({
  title: z.string().min(1, 'Title is required').max(100),
  content: z.string().min(10, 'Content must be at least 10 characters'),
});

type State = {
  errors?: {
    title?: string[];
    content?: string[];
  };
  message?: string;
} | null;

export async function createPost(prevState: State, formData: FormData): Promise<State> {
  const validatedFields = PostSchema.safeParse({
    title: formData.get('title'),
    content: formData.get('content'),
  });

  if (!validatedFields.success) {
    return {
      errors: validatedFields.error.flatten().fieldErrors,
    };
  }

  try {
    await db.post.create({
      data: validatedFields.data,
    });
  } catch (error) {
    return {
      message: 'Database error. Failed to create post.',
    };
  }

  revalidatePath('/posts');
  redirect('/posts');
}
```

### Client Component with Server Action

```tsx
// app/posts/new/page.tsx
'use client';

import { useFormState, useFormStatus } from 'react-dom';
import { createPost } from '@/app/actions';

function SubmitButton() {
  const { pending } = useFormStatus();

  return (
    <button type="submit" disabled={pending}>
      {pending ? 'Creating...' : 'Create Post'}
    </button>
  );
}

export default function NewPost() {
  const [state, formAction] = useFormState(createPost, null);

  return (
    <form action={formAction}>
      <div>
        <input name="title" placeholder="Title" />
        {state?.errors?.title && (
          <p className="error">{state.errors.title[0]}</p>
        )}
      </div>

      <div>
        <textarea name="content" placeholder="Content" />
        {state?.errors?.content && (
          <p className="error">{state.errors.content[0]}</p>
        )}
      </div>

      {state?.message && (
        <p className="error">{state.message}</p>
      )}

      <SubmitButton />
    </form>
  );
}
```

### Server Action with Return Value

```tsx
// app/actions.ts
'use server';

export async function toggleLike(postId: string) {
  const session = await getSession();
  if (!session) throw new Error('Unauthorized');

  const existingLike = await db.like.findFirst({
    where: { postId, userId: session.userId },
  });

  if (existingLike) {
    await db.like.delete({ where: { id: existingLike.id } });
    return { liked: false };
  }

  await db.like.create({
    data: { postId, userId: session.userId },
  });

  return { liked: true };
}

// components/LikeButton.tsx
'use client';

import { useState, useTransition } from 'react';
import { toggleLike } from '@/app/actions';

export default function LikeButton({ postId, initialLiked }) {
  const [liked, setLiked] = useState(initialLiked);
  const [isPending, startTransition] = useTransition();

  const handleClick = () => {
    startTransition(async () => {
      const result = await toggleLike(postId);
      setLiked(result.liked);
    });
  };

  return (
    <button onClick={handleClick} disabled={isPending}>
      {liked ? '❤️' : '🤍'} {isPending && '...'}
    </button>
  );
}
```

---

## File Uploads

### Route Handler Approach

```tsx
// app/api/upload/route.ts
import { NextResponse } from 'next/server';
import { writeFile } from 'fs/promises';
import path from 'path';

export async function POST(request: Request) {
  const formData = await request.formData();
  const file = formData.get('file') as File;

  if (!file) {
    return NextResponse.json({ error: 'No file uploaded' }, { status: 400 });
  }

  const bytes = await file.arrayBuffer();
  const buffer = Buffer.from(bytes);

  // Create unique filename
  const filename = `${Date.now()}-${file.name}`;
  const filepath = path.join(process.cwd(), 'public/uploads', filename);

  await writeFile(filepath, buffer);

  return NextResponse.json({
    url: `/uploads/${filename}`,
    filename,
    size: file.size,
  });
}
```

### Server Action Approach

```tsx
// app/actions.ts
'use server';

import { writeFile } from 'fs/promises';
import path from 'path';

export async function uploadFile(formData: FormData) {
  const file = formData.get('file') as File;

  if (!file || file.size === 0) {
    return { error: 'No file provided' };
  }

  const bytes = await file.arrayBuffer();
  const buffer = Buffer.from(bytes);

  const filename = `${Date.now()}-${file.name}`;
  const filepath = path.join(process.cwd(), 'public/uploads', filename);

  await writeFile(filepath, buffer);

  return { url: `/uploads/${filename}` };
}
```

---

## CORS Configuration

```tsx
// app/api/public/route.ts
import { NextResponse } from 'next/server';

export async function GET() {
  const data = await getData();

  return NextResponse.json(data, {
    headers: {
      'Access-Control-Allow-Origin': '*',
      'Access-Control-Allow-Methods': 'GET, POST, OPTIONS',
      'Access-Control-Allow-Headers': 'Content-Type, Authorization',
    },
  });
}

export async function OPTIONS() {
  return new NextResponse(null, {
    headers: {
      'Access-Control-Allow-Origin': '*',
      'Access-Control-Allow-Methods': 'GET, POST, OPTIONS',
      'Access-Control-Allow-Headers': 'Content-Type, Authorization',
    },
  });
}
```

---

## Exercise: Build a Complete API

Create a REST API for a todo application:

1. GET /api/todos - List all todos
2. POST /api/todos - Create a todo
3. GET /api/todos/[id] - Get single todo
4. PUT /api/todos/[id] - Update todo
5. DELETE /api/todos/[id] - Delete todo
6. Server Action for toggling completion

---

## Solution

```tsx
// lib/db.ts (simulated database)
let todos = [
  { id: '1', title: 'Learn Next.js', completed: false, createdAt: new Date() },
  { id: '2', title: 'Build API', completed: false, createdAt: new Date() },
];

export const db = {
  todo: {
    findMany: async () => todos,
    findUnique: async ({ where }: { where: { id: string } }) =>
      todos.find(t => t.id === where.id),
    create: async ({ data }: { data: any }) => {
      const todo = { ...data, id: Date.now().toString(), createdAt: new Date() };
      todos.push(todo);
      return todo;
    },
    update: async ({ where, data }: { where: { id: string }; data: any }) => {
      const index = todos.findIndex(t => t.id === where.id);
      if (index === -1) throw new Error('Not found');
      todos[index] = { ...todos[index], ...data };
      return todos[index];
    },
    delete: async ({ where }: { where: { id: string } }) => {
      const index = todos.findIndex(t => t.id === where.id);
      if (index === -1) throw new Error('Not found');
      todos.splice(index, 1);
    },
  },
};

// app/api/todos/route.ts
import { NextResponse } from 'next/server';
import { z } from 'zod';
import { db } from '@/lib/db';

const TodoSchema = z.object({
  title: z.string().min(1).max(200),
  completed: z.boolean().optional(),
});

export async function GET() {
  const todos = await db.todo.findMany();
  return NextResponse.json(todos);
}

export async function POST(request: Request) {
  try {
    const body = await request.json();
    const data = TodoSchema.parse(body);

    const todo = await db.todo.create({ data });

    return NextResponse.json(todo, { status: 201 });
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json(
        { error: 'Validation failed', details: error.errors },
        { status: 400 }
      );
    }
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}

// app/api/todos/[id]/route.ts
import { NextResponse } from 'next/server';
import { db } from '@/lib/db';

interface Props {
  params: { id: string };
}

export async function GET(request: Request, { params }: Props) {
  const todo = await db.todo.findUnique({ where: { id: params.id } });

  if (!todo) {
    return NextResponse.json({ error: 'Todo not found' }, { status: 404 });
  }

  return NextResponse.json(todo);
}

export async function PUT(request: Request, { params }: Props) {
  try {
    const body = await request.json();
    const todo = await db.todo.update({
      where: { id: params.id },
      data: body,
    });
    return NextResponse.json(todo);
  } catch (error) {
    return NextResponse.json({ error: 'Todo not found' }, { status: 404 });
  }
}

export async function DELETE(request: Request, { params }: Props) {
  try {
    await db.todo.delete({ where: { id: params.id } });
    return new NextResponse(null, { status: 204 });
  } catch (error) {
    return NextResponse.json({ error: 'Todo not found' }, { status: 404 });
  }
}

// app/actions.ts
'use server';

import { revalidatePath } from 'next/cache';
import { db } from '@/lib/db';

export async function toggleTodo(id: string) {
  const todo = await db.todo.findUnique({ where: { id } });

  if (!todo) {
    throw new Error('Todo not found');
  }

  await db.todo.update({
    where: { id },
    data: { completed: !todo.completed },
  });

  revalidatePath('/todos');
}

export async function createTodo(formData: FormData) {
  const title = formData.get('title') as string;

  if (!title || title.trim().length === 0) {
    return { error: 'Title is required' };
  }

  await db.todo.create({
    data: { title: title.trim(), completed: false },
  });

  revalidatePath('/todos');
}
```

---

## Key Takeaways

1. **Route Handlers** replace API Routes in App Router
2. **Each HTTP method** is a separate export (GET, POST, PUT, DELETE)
3. **Dynamic routes** use `[param]` folder naming
4. **NextResponse** provides helpers for JSON, redirects, and more
5. **Server Actions** are simpler for form submissions
6. **Validation** with Zod ensures type-safe data
7. **Authentication** can be handled via headers or middleware

---

## What's Next?

Tomorrow we'll cover **Authentication** in Next.js using NextAuth.js and building secure authentication flows!
