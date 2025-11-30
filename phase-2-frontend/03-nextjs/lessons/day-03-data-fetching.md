# Day 3: Data Fetching in Next.js

## Introduction

Next.js provides multiple patterns for fetching data, each optimized for different scenarios. Server Components fundamentally change how we think about data fetching, allowing direct database access and API calls on the server with automatic caching and optimization.

## Learning Objectives

By the end of this lesson, you will:
- Understand data fetching patterns in Server Components
- Master caching and revalidation strategies
- Fetch data in parallel vs sequential patterns
- Handle loading and error states
- Use Server Actions for mutations

---

## Data Fetching Fundamentals

### Server Components (Default)

```tsx
// This entire component runs on the server
// No API endpoint needed - fetch directly!
export default async function ProductsPage() {
  const products = await fetch('https://api.example.com/products');
  const data = await products.json();

  return (
    <div>
      {data.map(product => (
        <div key={product.id}>{product.name}</div>
      ))}
    </div>
  );
}
```

### Direct Database Access

```tsx
// No API layer needed!
import { db } from '@/lib/db';

export default async function UsersPage() {
  // Query database directly in Server Component
  const users = await db.user.findMany({
    where: { active: true },
    orderBy: { createdAt: 'desc' },
  });

  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

---

## Fetch API Extensions

Next.js extends the native `fetch` API with caching and revalidation:

### Caching Behavior

```tsx
// Cached forever (default for GET requests)
const res = await fetch('https://api.example.com/data');

// Revalidate every 60 seconds
const res = await fetch('https://api.example.com/data', {
  next: { revalidate: 60 }
});

// Don't cache (always fresh)
const res = await fetch('https://api.example.com/data', {
  cache: 'no-store'
});

// Force cache (never revalidate)
const res = await fetch('https://api.example.com/data', {
  cache: 'force-cache'
});
```

### Cache Tags for Fine-Grained Control

```tsx
// Tag requests for targeted revalidation
const res = await fetch('https://api.example.com/products', {
  next: { tags: ['products'] }
});

const res = await fetch(`https://api.example.com/products/${id}`, {
  next: { tags: ['products', `product-${id}`] }
});
```

### Revalidation

```tsx
// In a Server Action or Route Handler
import { revalidateTag, revalidatePath } from 'next/cache';

// Revalidate all requests with 'products' tag
revalidateTag('products');

// Revalidate specific product
revalidateTag(`product-${id}`);

// Revalidate a specific path
revalidatePath('/products');

// Revalidate a dynamic path
revalidatePath(`/products/${id}`);
```

---

## Sequential vs Parallel Data Fetching

### Sequential Fetching (Waterfall)

```tsx
// ❌ Slow - each request waits for the previous one
export default async function Page() {
  const user = await getUser();           // 1000ms
  const posts = await getPosts(user.id);  // 500ms
  const comments = await getComments();   // 500ms
  // Total: ~2000ms

  return <div>...</div>;
}
```

### Parallel Fetching

```tsx
// ✅ Fast - all requests start simultaneously
export default async function Page() {
  // Start all requests at once
  const [user, posts, comments] = await Promise.all([
    getUser(),      // 1000ms
    getPosts(),     // 500ms
    getComments()   // 500ms
  ]);
  // Total: ~1000ms (longest request)

  return <div>...</div>;
}
```

### Parallel with Dependencies

```tsx
export default async function Page({ params }) {
  // Start independent requests
  const userPromise = getUser(params.id);
  const settingsPromise = getSettings();

  // Wait for user to get dependent data
  const user = await userPromise;
  const postsPromise = getPosts(user.id);

  // Await remaining
  const [settings, posts] = await Promise.all([
    settingsPromise,
    postsPromise
  ]);

  return <div>...</div>;
}
```

---

## Loading UI with Suspense

### Loading File

```tsx
// app/dashboard/loading.tsx
export default function Loading() {
  return <DashboardSkeleton />;
}
```

### Suspense Boundaries

```tsx
// app/dashboard/page.tsx
import { Suspense } from 'react';

export default function Dashboard() {
  return (
    <div>
      <h1>Dashboard</h1>

      {/* Each section loads independently */}
      <Suspense fallback={<StatsSkeleton />}>
        <Stats />
      </Suspense>

      <Suspense fallback={<ChartSkeleton />}>
        <RevenueChart />
      </Suspense>

      <Suspense fallback={<TableSkeleton />}>
        <RecentOrders />
      </Suspense>
    </div>
  );
}

// Separate async components
async function Stats() {
  const stats = await getStats();
  return <StatsDisplay stats={stats} />;
}

async function RevenueChart() {
  const data = await getRevenueData();
  return <Chart data={data} />;
}

async function RecentOrders() {
  const orders = await getRecentOrders();
  return <OrdersTable orders={orders} />;
}
```

### Streaming Benefits

With Suspense, each component streams to the client as it resolves:

```
1. Initial HTML with skeletons sent
2. Stats resolves → streams to client
3. Chart resolves → streams to client
4. Orders resolves → streams to client
```

---

## Error Handling

### Error File

```tsx
// app/dashboard/error.tsx
'use client';

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  return (
    <div>
      <h2>Something went wrong!</h2>
      <button onClick={() => reset()}>Try again</button>
    </div>
  );
}
```

### Error Boundaries with Suspense

```tsx
import { Suspense } from 'react';
import { ErrorBoundary } from 'react-error-boundary';

export default function Page() {
  return (
    <ErrorBoundary fallback={<p>Error loading stats</p>}>
      <Suspense fallback={<Loading />}>
        <Stats />
      </Suspense>
    </ErrorBoundary>
  );
}
```

---

## Data Fetching Patterns

### Pattern 1: Fetch in Page

```tsx
// app/products/page.tsx
async function getProducts() {
  const res = await fetch('https://api.example.com/products', {
    next: { revalidate: 3600 } // Revalidate hourly
  });
  return res.json();
}

export default async function ProductsPage() {
  const products = await getProducts();

  return (
    <div className="grid">
      {products.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  );
}
```

### Pattern 2: Fetch in Component

```tsx
// components/ProductList.tsx
async function getProducts() {
  const res = await fetch('https://api.example.com/products');
  return res.json();
}

export default async function ProductList() {
  const products = await getProducts();

  return (
    <ul>
      {products.map(product => (
        <li key={product.id}>{product.name}</li>
      ))}
    </ul>
  );
}

// app/page.tsx
import { Suspense } from 'react';
import ProductList from '@/components/ProductList';

export default function Home() {
  return (
    <main>
      <h1>Products</h1>
      <Suspense fallback={<p>Loading products...</p>}>
        <ProductList />
      </Suspense>
    </main>
  );
}
```

### Pattern 3: Preloading Data

```tsx
// lib/data.ts
import { cache } from 'react';

// Deduplicate and cache the request
export const getUser = cache(async (id: string) => {
  const res = await fetch(`https://api.example.com/users/${id}`);
  return res.json();
});

// Preload function
export const preloadUser = (id: string) => {
  void getUser(id);
};

// app/users/[id]/page.tsx
import { getUser, preloadUser } from '@/lib/data';

export default async function UserPage({ params }) {
  // Start fetching early
  preloadUser(params.id);

  // ... other code ...

  // Will use the cached result
  const user = await getUser(params.id);

  return <div>{user.name}</div>;
}
```

### Pattern 4: Database Queries

```tsx
// lib/db.ts
import { PrismaClient } from '@prisma/client';
import { cache } from 'react';

const prisma = new PrismaClient();

export const getUser = cache(async (id: string) => {
  return prisma.user.findUnique({
    where: { id },
    include: {
      posts: true,
      profile: true,
    },
  });
});

export const getPosts = cache(async () => {
  return prisma.post.findMany({
    orderBy: { createdAt: 'desc' },
    include: { author: true },
  });
});
```

---

## Static Generation (SSG)

### generateStaticParams

```tsx
// app/blog/[slug]/page.tsx
export async function generateStaticParams() {
  const posts = await getPosts();

  return posts.map((post) => ({
    slug: post.slug,
  }));
}

export default async function Post({ params }) {
  const post = await getPost(params.slug);
  return <article>{post.content}</article>;
}
```

### Dynamic Rendering Control

```tsx
// Force static (default)
export const dynamic = 'force-static';

// Force dynamic
export const dynamic = 'force-dynamic';

// Auto (Next.js decides)
export const dynamic = 'auto';

// Error on dynamic (build fails if dynamic)
export const dynamic = 'error';
```

---

## Server Actions

### Basic Server Action

```tsx
// app/actions.ts
'use server';

import { revalidatePath } from 'next/cache';
import { db } from '@/lib/db';

export async function createPost(formData: FormData) {
  const title = formData.get('title') as string;
  const content = formData.get('content') as string;

  await db.post.create({
    data: { title, content },
  });

  revalidatePath('/posts');
}
```

### Using in Forms

```tsx
// app/posts/new/page.tsx
import { createPost } from '@/app/actions';

export default function NewPost() {
  return (
    <form action={createPost}>
      <input name="title" placeholder="Title" required />
      <textarea name="content" placeholder="Content" required />
      <button type="submit">Create Post</button>
    </form>
  );
}
```

### With Client-Side State

```tsx
'use client';

import { useFormStatus, useFormState } from 'react-dom';
import { createPost } from '@/app/actions';

function SubmitButton() {
  const { pending } = useFormStatus();

  return (
    <button type="submit" disabled={pending}>
      {pending ? 'Creating...' : 'Create Post'}
    </button>
  );
}

export default function NewPostForm() {
  const [state, formAction] = useFormState(createPost, null);

  return (
    <form action={formAction}>
      <input name="title" placeholder="Title" />
      <textarea name="content" placeholder="Content" />
      {state?.error && <p className="error">{state.error}</p>}
      <SubmitButton />
    </form>
  );
}
```

### Server Action with Validation

```tsx
// app/actions.ts
'use server';

import { z } from 'zod';
import { revalidatePath } from 'next/cache';
import { redirect } from 'next/navigation';

const PostSchema = z.object({
  title: z.string().min(1).max(100),
  content: z.string().min(10),
});

export async function createPost(prevState: any, formData: FormData) {
  const rawData = {
    title: formData.get('title'),
    content: formData.get('content'),
  };

  const validatedData = PostSchema.safeParse(rawData);

  if (!validatedData.success) {
    return {
      errors: validatedData.error.flatten().fieldErrors,
    };
  }

  try {
    await db.post.create({
      data: validatedData.data,
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

---

## Client-Side Data Fetching

For data that changes frequently or is user-specific:

### Using SWR

```tsx
'use client';

import useSWR from 'swr';

const fetcher = (url: string) => fetch(url).then(res => res.json());

export default function Profile() {
  const { data, error, isLoading } = useSWR('/api/user', fetcher);

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error loading profile</div>;

  return <div>Hello, {data.name}!</div>;
}
```

### Using React Query (TanStack Query)

```tsx
'use client';

import { useQuery } from '@tanstack/react-query';

export default function UserList() {
  const { data, isLoading, error } = useQuery({
    queryKey: ['users'],
    queryFn: () => fetch('/api/users').then(res => res.json()),
  });

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return (
    <ul>
      {data.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

---

## Exercise: Build a Blog with Full Data Flow

Create a blog with:
1. List of posts (SSG with revalidation)
2. Individual post pages (dynamic)
3. Create post form (Server Action)
4. Comments section (Client-side fetching)

---

## Solution

```tsx
// lib/data.ts
import { cache } from 'react';

export const getPosts = cache(async () => {
  const res = await fetch('https://api.example.com/posts', {
    next: { tags: ['posts'], revalidate: 3600 }
  });
  return res.json();
});

export const getPost = cache(async (slug: string) => {
  const res = await fetch(`https://api.example.com/posts/${slug}`, {
    next: { tags: ['posts', `post-${slug}`] }
  });
  return res.json();
});

// app/actions.ts
'use server';

import { revalidateTag } from 'next/cache';
import { redirect } from 'next/navigation';
import { z } from 'zod';

const PostSchema = z.object({
  title: z.string().min(1, 'Title is required'),
  content: z.string().min(10, 'Content must be at least 10 characters'),
  slug: z.string().min(1, 'Slug is required'),
});

export async function createPost(prevState: any, formData: FormData) {
  const result = PostSchema.safeParse({
    title: formData.get('title'),
    content: formData.get('content'),
    slug: formData.get('slug'),
  });

  if (!result.success) {
    return { errors: result.error.flatten().fieldErrors };
  }

  try {
    await fetch('https://api.example.com/posts', {
      method: 'POST',
      body: JSON.stringify(result.data),
    });
  } catch (error) {
    return { message: 'Failed to create post' };
  }

  revalidateTag('posts');
  redirect('/blog');
}

// app/blog/page.tsx
import Link from 'next/link';
import { Suspense } from 'react';
import { getPosts } from '@/lib/data';

async function PostList() {
  const posts = await getPosts();

  return (
    <div className="space-y-4">
      {posts.map((post: any) => (
        <article key={post.slug} className="border-b pb-4">
          <Link href={`/blog/${post.slug}`}>
            <h2 className="text-xl font-semibold hover:text-blue-600">
              {post.title}
            </h2>
          </Link>
          <p className="text-gray-600">{post.excerpt}</p>
        </article>
      ))}
    </div>
  );
}

export default function BlogPage() {
  return (
    <div>
      <div className="flex justify-between items-center mb-6">
        <h1 className="text-3xl font-bold">Blog</h1>
        <Link href="/blog/new" className="btn">New Post</Link>
      </div>

      <Suspense fallback={<PostListSkeleton />}>
        <PostList />
      </Suspense>
    </div>
  );
}

// app/blog/[slug]/page.tsx
import { Suspense } from 'react';
import { getPost, getPosts } from '@/lib/data';
import { notFound } from 'next/navigation';
import Comments from '@/components/Comments';

export async function generateStaticParams() {
  const posts = await getPosts();
  return posts.map((post: any) => ({ slug: post.slug }));
}

export async function generateMetadata({ params }: { params: { slug: string } }) {
  const post = await getPost(params.slug);
  return {
    title: post?.title || 'Post Not Found',
    description: post?.excerpt,
  };
}

export default async function PostPage({ params }: { params: { slug: string } }) {
  const post = await getPost(params.slug);

  if (!post) {
    notFound();
  }

  return (
    <article>
      <h1 className="text-3xl font-bold mb-4">{post.title}</h1>
      <time className="text-gray-500">{post.date}</time>
      <div className="prose mt-6">{post.content}</div>

      <section className="mt-10">
        <h2 className="text-2xl font-bold mb-4">Comments</h2>
        <Suspense fallback={<p>Loading comments...</p>}>
          <Comments postSlug={params.slug} />
        </Suspense>
      </section>
    </article>
  );
}

// components/Comments.tsx
'use client';

import useSWR from 'swr';

const fetcher = (url: string) => fetch(url).then(res => res.json());

export default function Comments({ postSlug }: { postSlug: string }) {
  const { data: comments, error, isLoading } = useSWR(
    `/api/posts/${postSlug}/comments`,
    fetcher,
    { refreshInterval: 30000 } // Refresh every 30 seconds
  );

  if (isLoading) return <p>Loading comments...</p>;
  if (error) return <p>Error loading comments</p>;

  return (
    <div className="space-y-4">
      {comments.map((comment: any) => (
        <div key={comment.id} className="bg-gray-50 p-4 rounded">
          <p className="font-medium">{comment.author}</p>
          <p>{comment.content}</p>
        </div>
      ))}
    </div>
  );
}

// app/blog/new/page.tsx
'use client';

import { useFormState, useFormStatus } from 'react-dom';
import { createPost } from '@/app/actions';

function SubmitButton() {
  const { pending } = useFormStatus();
  return (
    <button
      type="submit"
      disabled={pending}
      className="bg-blue-600 text-white px-4 py-2 rounded"
    >
      {pending ? 'Publishing...' : 'Publish Post'}
    </button>
  );
}

export default function NewPostPage() {
  const [state, formAction] = useFormState(createPost, null);

  return (
    <div>
      <h1 className="text-3xl font-bold mb-6">New Post</h1>

      <form action={formAction} className="space-y-4">
        <div>
          <label className="block mb-1">Title</label>
          <input
            name="title"
            className="w-full border rounded p-2"
            required
          />
          {state?.errors?.title && (
            <p className="text-red-500 text-sm">{state.errors.title}</p>
          )}
        </div>

        <div>
          <label className="block mb-1">Slug</label>
          <input
            name="slug"
            className="w-full border rounded p-2"
            required
          />
          {state?.errors?.slug && (
            <p className="text-red-500 text-sm">{state.errors.slug}</p>
          )}
        </div>

        <div>
          <label className="block mb-1">Content</label>
          <textarea
            name="content"
            rows={10}
            className="w-full border rounded p-2"
            required
          />
          {state?.errors?.content && (
            <p className="text-red-500 text-sm">{state.errors.content}</p>
          )}
        </div>

        {state?.message && (
          <p className="text-red-500">{state.message}</p>
        )}

        <SubmitButton />
      </form>
    </div>
  );
}
```

---

## Key Takeaways

1. **Server Components** fetch data directly without API endpoints
2. **Parallel fetching** with `Promise.all` for better performance
3. **Suspense** enables streaming and granular loading states
4. **Cache and revalidation** control data freshness
5. **Server Actions** handle mutations with form submissions
6. **Tags and paths** for fine-grained cache invalidation
7. **Client-side fetching** for frequently changing or user-specific data

---

## What's Next?

Tomorrow we'll explore **Server Components** in depth - understanding the React Server Component architecture, when to use client vs server, and optimization patterns!
