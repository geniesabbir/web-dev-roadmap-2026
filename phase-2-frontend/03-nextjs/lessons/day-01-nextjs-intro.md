# Day 1: Introduction to Next.js - The React Framework for Production

## Introduction

Next.js is the most popular React framework, created by Vercel. It provides a production-ready foundation with features like server-side rendering, static site generation, file-based routing, API routes, and excellent developer experience out of the box.

## Learning Objectives

By the end of this lesson, you will:
- Understand what Next.js offers beyond React
- Set up a Next.js project with App Router
- Understand the project structure
- Create your first pages and layouts
- Know the difference between Server and Client Components

---

## Why Next.js?

### React Alone vs Next.js

| Feature | React (Vite) | Next.js |
|---------|--------------|---------|
| Routing | Manual (React Router) | Built-in file-based |
| SSR | Manual setup | Built-in |
| SSG | Not available | Built-in |
| API Routes | Separate backend | Built-in |
| Image Optimization | Manual | Built-in |
| SEO | Manual | Built-in |
| Code Splitting | Manual | Automatic |
| TypeScript | Manual config | Zero config |

### Key Features

1. **Server Components** - Render on server, reduce client JS
2. **File-based Routing** - Create routes by adding files
3. **Data Fetching** - Multiple strategies (SSR, SSG, ISR)
4. **API Routes** - Backend endpoints in the same project
5. **Image Optimization** - Automatic image optimization
6. **Font Optimization** - Google Fonts with zero layout shift
7. **Metadata API** - SEO-friendly page metadata

---

## Setting Up Next.js

### Create New Project

```bash
# Create new Next.js app with create-next-app
npx create-next-app@latest my-app

# Answer the prompts:
# ✔ Would you like to use TypeScript? Yes
# ✔ Would you like to use ESLint? Yes
# ✔ Would you like to use Tailwind CSS? Yes
# ✔ Would you like to use `src/` directory? Yes
# ✔ Would you like to use App Router? (recommended) Yes
# ✔ Would you like to customize the default import alias? No

# Navigate and start
cd my-app
npm run dev
```

### Project Structure

```
my-app/
├── src/
│   └── app/                 # App Router directory
│       ├── layout.tsx       # Root layout (required)
│       ├── page.tsx         # Home page (/)
│       ├── globals.css      # Global styles
│       ├── favicon.ico      # Favicon
│       └── about/           # /about route
│           └── page.tsx
├── public/                  # Static files
│   └── images/
├── next.config.js           # Next.js configuration
├── tailwind.config.ts       # Tailwind configuration
├── tsconfig.json            # TypeScript configuration
└── package.json
```

---

## App Router Basics

### The `app` Directory

The `app` directory uses the new App Router (Next.js 13+):

```
app/
├── layout.tsx      # Root layout (wraps all pages)
├── page.tsx        # Home page (/)
├── loading.tsx     # Loading UI
├── error.tsx       # Error UI
├── not-found.tsx   # 404 page
├── about/
│   └── page.tsx    # /about
├── blog/
│   ├── page.tsx    # /blog
│   └── [slug]/
│       └── page.tsx # /blog/:slug (dynamic)
└── api/
    └── hello/
        └── route.ts # API endpoint
```

### Special Files

| File | Purpose |
|------|---------|
| `page.tsx` | UI for a route (makes route accessible) |
| `layout.tsx` | Shared UI for segment and children |
| `loading.tsx` | Loading UI (Suspense boundary) |
| `error.tsx` | Error UI (Error boundary) |
| `not-found.tsx` | 404 UI |
| `route.ts` | API endpoint |

---

## Creating Pages

### Basic Page

```tsx
// src/app/page.tsx - Home page at /
export default function HomePage() {
  return (
    <main>
      <h1>Welcome to My App</h1>
      <p>This is the home page.</p>
    </main>
  );
}
```

### About Page

```tsx
// src/app/about/page.tsx - About page at /about
export default function AboutPage() {
  return (
    <main>
      <h1>About Us</h1>
      <p>Learn more about our company.</p>
    </main>
  );
}
```

### Dynamic Page

```tsx
// src/app/blog/[slug]/page.tsx - Dynamic route at /blog/:slug
interface PageProps {
  params: { slug: string };
}

export default function BlogPost({ params }: PageProps) {
  return (
    <article>
      <h1>Blog Post: {params.slug}</h1>
      <p>This is the content for {params.slug}.</p>
    </article>
  );
}
```

---

## Layouts

Layouts wrap pages and persist across navigation:

### Root Layout (Required)

```tsx
// src/app/layout.tsx
import type { Metadata } from 'next';
import { Inter } from 'next/font/google';
import './globals.css';

const inter = Inter({ subsets: ['latin'] });

export const metadata: Metadata = {
  title: 'My App',
  description: 'Built with Next.js',
};

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body className={inter.className}>
        <header>
          <nav>
            <a href="/">Home</a>
            <a href="/about">About</a>
            <a href="/blog">Blog</a>
          </nav>
        </header>
        <main>{children}</main>
        <footer>
          <p>© 2025 My App</p>
        </footer>
      </body>
    </html>
  );
}
```

### Nested Layouts

```tsx
// src/app/dashboard/layout.tsx
export default function DashboardLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <div className="dashboard">
      <aside className="sidebar">
        <nav>
          <a href="/dashboard">Overview</a>
          <a href="/dashboard/analytics">Analytics</a>
          <a href="/dashboard/settings">Settings</a>
        </nav>
      </aside>
      <div className="content">{children}</div>
    </div>
  );
}

// src/app/dashboard/page.tsx
export default function DashboardPage() {
  return <h1>Dashboard Overview</h1>;
}

// src/app/dashboard/analytics/page.tsx
export default function AnalyticsPage() {
  return <h1>Analytics</h1>;
}
```

---

## Server vs Client Components

### Server Components (Default)

By default, all components in the `app` directory are Server Components:

```tsx
// src/app/users/page.tsx - Server Component
async function getUsers() {
  const res = await fetch('https://api.example.com/users');
  return res.json();
}

export default async function UsersPage() {
  // This runs on the server
  const users = await getUsers();

  return (
    <div>
      <h1>Users</h1>
      <ul>
        {users.map((user: any) => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
    </div>
  );
}
```

**Server Component Benefits:**
- Direct database access
- Keep sensitive data on server
- Reduce client JavaScript
- Automatic code splitting

### Client Components

Add `'use client'` directive for interactivity:

```tsx
// src/components/Counter.tsx
'use client';

import { useState } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(c => c + 1)}>+</button>
    </div>
  );
}
```

**Use Client Components When:**
- Using React hooks (useState, useEffect)
- Browser-only APIs (localStorage, window)
- Event listeners (onClick, onChange)
- Custom hooks with state

### Mixing Server and Client Components

```tsx
// src/app/page.tsx - Server Component
import Counter from '@/components/Counter'; // Client Component

async function getData() {
  const res = await fetch('https://api.example.com/data');
  return res.json();
}

export default async function HomePage() {
  const data = await getData(); // Server-side fetch

  return (
    <div>
      <h1>{data.title}</h1>
      <p>{data.description}</p>
      <Counter /> {/* Client Component for interactivity */}
    </div>
  );
}
```

---

## Navigation

### Link Component

```tsx
import Link from 'next/link';

export default function Navigation() {
  return (
    <nav>
      <Link href="/">Home</Link>
      <Link href="/about">About</Link>
      <Link href="/blog">Blog</Link>
      <Link href="/blog/my-post">Blog Post</Link>

      {/* Prefetching disabled */}
      <Link href="/contact" prefetch={false}>Contact</Link>

      {/* Replace instead of push to history */}
      <Link href="/login" replace>Login</Link>
    </nav>
  );
}
```

### Programmatic Navigation

```tsx
'use client';

import { useRouter } from 'next/navigation';

export default function LoginForm() {
  const router = useRouter();

  const handleLogin = async () => {
    // ... login logic
    router.push('/dashboard');      // Navigate
    // router.replace('/dashboard'); // Navigate without history
    // router.back();                // Go back
    // router.forward();             // Go forward
    // router.refresh();             // Refresh current route
  };

  return <button onClick={handleLogin}>Login</button>;
}
```

### Active Links

```tsx
'use client';

import Link from 'next/link';
import { usePathname } from 'next/navigation';

export default function NavLink({
  href,
  children,
}: {
  href: string;
  children: React.ReactNode;
}) {
  const pathname = usePathname();
  const isActive = pathname === href;

  return (
    <Link
      href={href}
      className={isActive ? 'nav-link active' : 'nav-link'}
    >
      {children}
    </Link>
  );
}
```

---

## Metadata and SEO

### Static Metadata

```tsx
// src/app/about/page.tsx
import type { Metadata } from 'next';

export const metadata: Metadata = {
  title: 'About Us',
  description: 'Learn about our company and mission',
  keywords: ['about', 'company', 'team'],
  openGraph: {
    title: 'About Us',
    description: 'Learn about our company',
    images: ['/images/about-og.jpg'],
  },
};

export default function AboutPage() {
  return <h1>About Us</h1>;
}
```

### Dynamic Metadata

```tsx
// src/app/blog/[slug]/page.tsx
import type { Metadata } from 'next';

interface Props {
  params: { slug: string };
}

// Generate metadata dynamically
export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const post = await getPost(params.slug);

  return {
    title: post.title,
    description: post.excerpt,
    openGraph: {
      title: post.title,
      description: post.excerpt,
      images: [post.image],
    },
  };
}

export default async function BlogPost({ params }: Props) {
  const post = await getPost(params.slug);
  return <article>{/* ... */}</article>;
}
```

---

## Loading and Error States

### Loading UI

```tsx
// src/app/dashboard/loading.tsx
export default function Loading() {
  return (
    <div className="loading">
      <div className="spinner" />
      <p>Loading...</p>
    </div>
  );
}
```

### Error UI

```tsx
// src/app/dashboard/error.tsx
'use client';

interface ErrorProps {
  error: Error & { digest?: string };
  reset: () => void;
}

export default function Error({ error, reset }: ErrorProps) {
  return (
    <div className="error">
      <h2>Something went wrong!</h2>
      <p>{error.message}</p>
      <button onClick={() => reset()}>Try again</button>
    </div>
  );
}
```

### Not Found

```tsx
// src/app/not-found.tsx
import Link from 'next/link';

export default function NotFound() {
  return (
    <div className="not-found">
      <h2>Page Not Found</h2>
      <p>Could not find the requested resource.</p>
      <Link href="/">Return Home</Link>
    </div>
  );
}
```

### Triggering Not Found

```tsx
// src/app/blog/[slug]/page.tsx
import { notFound } from 'next/navigation';

export default async function BlogPost({ params }: Props) {
  const post = await getPost(params.slug);

  if (!post) {
    notFound(); // Renders not-found.tsx
  }

  return <article>{/* ... */}</article>;
}
```

---

## Styling in Next.js

### Global CSS

```tsx
// src/app/layout.tsx
import './globals.css';
```

### CSS Modules

```tsx
// src/components/Button.module.css
.button {
  padding: 10px 20px;
  border-radius: 8px;
}

.primary {
  background: blue;
  color: white;
}

// src/components/Button.tsx
import styles from './Button.module.css';

export default function Button({ variant = 'primary', children }) {
  return (
    <button className={`${styles.button} ${styles[variant]}`}>
      {children}
    </button>
  );
}
```

### Tailwind CSS (Recommended)

```tsx
// Tailwind is pre-configured with create-next-app
export default function Card({ title, children }) {
  return (
    <div className="rounded-lg shadow-md p-6 bg-white dark:bg-gray-800">
      <h2 className="text-xl font-bold mb-4">{title}</h2>
      {children}
    </div>
  );
}
```

---

## Exercise: Build a Simple Blog

Create a blog with the following structure:

```
app/
├── page.tsx           # Home with featured posts
├── layout.tsx         # Root layout with navigation
├── blog/
│   ├── page.tsx       # Blog listing page
│   └── [slug]/
│       └── page.tsx   # Individual blog post
└── about/
    └── page.tsx       # About page
```

### Requirements

1. Navigation in layout
2. Home page with welcome message
3. Blog listing page
4. Dynamic blog post pages
5. About page
6. Custom 404 page
7. Loading states

---

## Solution

### Root Layout

```tsx
// src/app/layout.tsx
import type { Metadata } from 'next';
import Link from 'next/link';
import './globals.css';

export const metadata: Metadata = {
  title: 'My Blog',
  description: 'A simple blog built with Next.js',
};

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>
        <header className="bg-gray-900 text-white p-4">
          <nav className="max-w-4xl mx-auto flex gap-6">
            <Link href="/" className="font-bold text-xl">My Blog</Link>
            <Link href="/blog" className="hover:underline">Blog</Link>
            <Link href="/about" className="hover:underline">About</Link>
          </nav>
        </header>
        <main className="max-w-4xl mx-auto p-4">{children}</main>
        <footer className="text-center p-4 text-gray-500">
          © 2025 My Blog
        </footer>
      </body>
    </html>
  );
}
```

### Home Page

```tsx
// src/app/page.tsx
import Link from 'next/link';

const featuredPosts = [
  { slug: 'getting-started', title: 'Getting Started with Next.js' },
  { slug: 'server-components', title: 'Understanding Server Components' },
];

export default function HomePage() {
  return (
    <div>
      <h1 className="text-4xl font-bold mb-6">Welcome to My Blog</h1>
      <p className="text-lg mb-8">
        Exploring web development with Next.js and React.
      </p>

      <section>
        <h2 className="text-2xl font-semibold mb-4">Featured Posts</h2>
        <div className="grid gap-4">
          {featuredPosts.map(post => (
            <Link
              key={post.slug}
              href={`/blog/${post.slug}`}
              className="block p-4 border rounded-lg hover:shadow-md transition"
            >
              <h3 className="font-medium">{post.title}</h3>
            </Link>
          ))}
        </div>
      </section>
    </div>
  );
}
```

### Blog Page

```tsx
// src/app/blog/page.tsx
import Link from 'next/link';
import type { Metadata } from 'next';

export const metadata: Metadata = {
  title: 'Blog | My Blog',
  description: 'Read our latest articles',
};

const posts = [
  { slug: 'getting-started', title: 'Getting Started with Next.js', date: '2025-01-01' },
  { slug: 'server-components', title: 'Understanding Server Components', date: '2025-01-05' },
  { slug: 'data-fetching', title: 'Data Fetching in Next.js', date: '2025-01-10' },
];

export default function BlogPage() {
  return (
    <div>
      <h1 className="text-3xl font-bold mb-6">Blog</h1>
      <div className="space-y-4">
        {posts.map(post => (
          <article key={post.slug} className="border-b pb-4">
            <Link href={`/blog/${post.slug}`}>
              <h2 className="text-xl font-semibold hover:text-blue-600">
                {post.title}
              </h2>
            </Link>
            <time className="text-gray-500">{post.date}</time>
          </article>
        ))}
      </div>
    </div>
  );
}
```

### Dynamic Blog Post

```tsx
// src/app/blog/[slug]/page.tsx
import { notFound } from 'next/navigation';
import type { Metadata } from 'next';

interface Props {
  params: { slug: string };
}

const posts: Record<string, { title: string; content: string; date: string }> = {
  'getting-started': {
    title: 'Getting Started with Next.js',
    content: 'Next.js is a powerful React framework...',
    date: '2025-01-01',
  },
  'server-components': {
    title: 'Understanding Server Components',
    content: 'Server Components are a new paradigm...',
    date: '2025-01-05',
  },
  'data-fetching': {
    title: 'Data Fetching in Next.js',
    content: 'Next.js provides multiple ways to fetch data...',
    date: '2025-01-10',
  },
};

export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const post = posts[params.slug];

  if (!post) {
    return { title: 'Post Not Found' };
  }

  return {
    title: `${post.title} | My Blog`,
    description: post.content.slice(0, 160),
  };
}

export default function BlogPost({ params }: Props) {
  const post = posts[params.slug];

  if (!post) {
    notFound();
  }

  return (
    <article>
      <h1 className="text-3xl font-bold mb-2">{post.title}</h1>
      <time className="text-gray-500 block mb-6">{post.date}</time>
      <div className="prose">{post.content}</div>
    </article>
  );
}
```

### About Page

```tsx
// src/app/about/page.tsx
import type { Metadata } from 'next';

export const metadata: Metadata = {
  title: 'About | My Blog',
  description: 'Learn more about me and this blog',
};

export default function AboutPage() {
  return (
    <div>
      <h1 className="text-3xl font-bold mb-6">About</h1>
      <p className="mb-4">
        This blog is dedicated to exploring modern web development
        with Next.js, React, and related technologies.
      </p>
      <p>
        I'm a developer passionate about building great user experiences
        and sharing knowledge with the community.
      </p>
    </div>
  );
}
```

### 404 Page

```tsx
// src/app/not-found.tsx
import Link from 'next/link';

export default function NotFound() {
  return (
    <div className="text-center py-20">
      <h1 className="text-6xl font-bold text-gray-300">404</h1>
      <h2 className="text-2xl font-semibold mt-4 mb-2">Page Not Found</h2>
      <p className="text-gray-500 mb-6">
        The page you're looking for doesn't exist.
      </p>
      <Link
        href="/"
        className="inline-block bg-blue-600 text-white px-6 py-2 rounded-lg hover:bg-blue-700"
      >
        Go Home
      </Link>
    </div>
  );
}
```

### Loading State

```tsx
// src/app/blog/loading.tsx
export default function Loading() {
  return (
    <div className="animate-pulse">
      <div className="h-8 bg-gray-200 rounded w-1/3 mb-6" />
      <div className="space-y-4">
        {[1, 2, 3].map(i => (
          <div key={i} className="border-b pb-4">
            <div className="h-6 bg-gray-200 rounded w-2/3 mb-2" />
            <div className="h-4 bg-gray-100 rounded w-1/4" />
          </div>
        ))}
      </div>
    </div>
  );
}
```

---

## Key Takeaways

1. **App Router** is the modern way to build Next.js apps
2. **Server Components** are the default - reduce client JS
3. **Client Components** need `'use client'` directive
4. **File-based routing** - folders and page.tsx create routes
5. **Layouts** persist across navigation and wrap pages
6. **Metadata API** provides built-in SEO support
7. **Special files** (loading, error, not-found) handle UI states

---

## What's Next?

Tomorrow we'll dive deep into **Routing** - dynamic routes, route groups, parallel routes, and intercepting routes!
