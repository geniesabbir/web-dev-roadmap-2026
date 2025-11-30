# Day 2: Next.js Routing - Mastering the App Router

## Introduction

Next.js App Router provides powerful file-based routing with support for dynamic routes, nested layouts, route groups, parallel routes, and intercepting routes. Today we'll master all these routing patterns.

## Learning Objectives

By the end of this lesson, you will:
- Create dynamic and catch-all routes
- Organize routes with route groups
- Build parallel routes for complex UIs
- Implement intercepting routes for modals
- Handle route parameters and search params

---

## Route Basics Recap

### File-Based Routing

```
app/
├── page.tsx           → /
├── about/page.tsx     → /about
├── blog/page.tsx      → /blog
└── contact/page.tsx   → /contact
```

### Route Segments

Each folder represents a route segment:

```
app/
└── dashboard/
    └── settings/
        └── profile/
            └── page.tsx  → /dashboard/settings/profile
```

---

## Dynamic Routes

### Single Dynamic Segment

```
app/
└── blog/
    └── [slug]/
        └── page.tsx  → /blog/:slug
```

```tsx
// app/blog/[slug]/page.tsx
interface PageProps {
  params: { slug: string };
}

export default function BlogPost({ params }: PageProps) {
  // params.slug = "my-first-post" for /blog/my-first-post
  return <h1>Post: {params.slug}</h1>;
}
```

### Multiple Dynamic Segments

```
app/
└── shop/
    └── [category]/
        └── [product]/
            └── page.tsx  → /shop/:category/:product
```

```tsx
// app/shop/[category]/[product]/page.tsx
interface PageProps {
  params: {
    category: string;
    product: string;
  };
}

export default function ProductPage({ params }: PageProps) {
  // /shop/electronics/phone
  // params = { category: "electronics", product: "phone" }
  return (
    <div>
      <p>Category: {params.category}</p>
      <p>Product: {params.product}</p>
    </div>
  );
}
```

### Catch-All Routes

```
app/
└── docs/
    └── [...slug]/
        └── page.tsx  → /docs/* (catches all)
```

```tsx
// app/docs/[...slug]/page.tsx
interface PageProps {
  params: { slug: string[] };
}

export default function DocsPage({ params }: PageProps) {
  // /docs/getting-started/installation
  // params.slug = ["getting-started", "installation"]
  return (
    <div>
      <p>Path segments: {params.slug.join(' / ')}</p>
    </div>
  );
}
```

### Optional Catch-All Routes

```
app/
└── docs/
    └── [[...slug]]/
        └── page.tsx  → /docs and /docs/* both match
```

```tsx
// app/docs/[[...slug]]/page.tsx
interface PageProps {
  params: { slug?: string[] };
}

export default function DocsPage({ params }: PageProps) {
  // /docs → params.slug = undefined
  // /docs/intro → params.slug = ["intro"]
  // /docs/intro/setup → params.slug = ["intro", "setup"]

  if (!params.slug) {
    return <h1>Documentation Home</h1>;
  }

  return <h1>Docs: {params.slug.join(' / ')}</h1>;
}
```

---

## Route Groups

Route groups organize routes without affecting URL structure:

```
app/
├── (marketing)/
│   ├── layout.tsx      # Marketing layout
│   ├── page.tsx        # / (home)
│   ├── about/page.tsx  # /about
│   └── pricing/page.tsx # /pricing
├── (shop)/
│   ├── layout.tsx      # Shop layout
│   ├── products/page.tsx # /products
│   └── cart/page.tsx   # /cart
└── (dashboard)/
    ├── layout.tsx      # Dashboard layout
    └── dashboard/page.tsx # /dashboard
```

### Use Cases

1. **Different layouts** for different sections
2. **Organization** without URL impact
3. **Feature-based** code organization

```tsx
// app/(marketing)/layout.tsx
export default function MarketingLayout({ children }) {
  return (
    <div className="marketing">
      <MarketingHeader />
      {children}
      <MarketingFooter />
    </div>
  );
}

// app/(dashboard)/layout.tsx
export default function DashboardLayout({ children }) {
  return (
    <div className="dashboard">
      <Sidebar />
      <main>{children}</main>
    </div>
  );
}
```

---

## Parallel Routes

Parallel routes allow rendering multiple pages in the same layout simultaneously:

```
app/
└── dashboard/
    ├── layout.tsx
    ├── page.tsx
    ├── @analytics/
    │   └── page.tsx
    ├── @team/
    │   └── page.tsx
    └── @notifications/
        └── page.tsx
```

### Using Slots

```tsx
// app/dashboard/layout.tsx
export default function DashboardLayout({
  children,
  analytics,
  team,
  notifications,
}: {
  children: React.ReactNode;
  analytics: React.ReactNode;
  team: React.ReactNode;
  notifications: React.ReactNode;
}) {
  return (
    <div className="dashboard-grid">
      <main>{children}</main>
      <aside className="sidebar">
        {analytics}
        {team}
      </aside>
      <div className="notifications">{notifications}</div>
    </div>
  );
}

// app/dashboard/@analytics/page.tsx
export default function AnalyticsSlot() {
  return <AnalyticsWidget />;
}

// app/dashboard/@team/page.tsx
export default function TeamSlot() {
  return <TeamWidget />;
}
```

### Conditional Rendering with Parallel Routes

```tsx
// app/dashboard/layout.tsx
import { auth } from '@/lib/auth';

export default async function DashboardLayout({
  children,
  admin,
  user,
}: {
  children: React.ReactNode;
  admin: React.ReactNode;
  user: React.ReactNode;
}) {
  const session = await auth();
  const isAdmin = session?.user?.role === 'admin';

  return (
    <div>
      {children}
      {isAdmin ? admin : user}
    </div>
  );
}
```

### Default Slots

```tsx
// app/dashboard/@analytics/default.tsx
// Shown when the parallel route has no match
export default function AnalyticsDefault() {
  return <p>Select an analytics view</p>;
}
```

---

## Intercepting Routes

Intercepting routes allow you to load a route within the current layout while showing the intercepted route in a different context (like a modal).

### Convention

```
(.)  - Same level
(..) - One level up
(..)(..) - Two levels up
(...) - Root app directory
```

### Example: Photo Modal

```
app/
├── @modal/
│   └── (.)photo/[id]/
│       └── page.tsx    # Intercepted route (modal)
├── photo/
│   └── [id]/
│       └── page.tsx    # Full page route
├── layout.tsx
└── page.tsx            # Feed with photos
```

```tsx
// app/layout.tsx
export default function RootLayout({
  children,
  modal,
}: {
  children: React.ReactNode;
  modal: React.ReactNode;
}) {
  return (
    <html>
      <body>
        {children}
        {modal}
      </body>
    </html>
  );
}

// app/page.tsx - Photo feed
import Link from 'next/link';

const photos = [
  { id: 1, url: '/photo1.jpg' },
  { id: 2, url: '/photo2.jpg' },
];

export default function Feed() {
  return (
    <div className="grid grid-cols-3 gap-4">
      {photos.map(photo => (
        <Link key={photo.id} href={`/photo/${photo.id}`}>
          <img src={photo.url} alt="" />
        </Link>
      ))}
    </div>
  );
}

// app/@modal/(.)photo/[id]/page.tsx - Modal version
import Modal from '@/components/Modal';

export default function PhotoModal({ params }: { params: { id: string } }) {
  return (
    <Modal>
      <h2>Photo {params.id}</h2>
      <img src={`/photo${params.id}.jpg`} alt="" />
    </Modal>
  );
}

// app/photo/[id]/page.tsx - Full page version
export default function PhotoPage({ params }: { params: { id: string } }) {
  return (
    <div className="photo-page">
      <h1>Photo {params.id}</h1>
      <img src={`/photo${params.id}.jpg`} alt="" className="full-photo" />
      <p>Photo details and comments...</p>
    </div>
  );
}
```

### Modal Component

```tsx
// components/Modal.tsx
'use client';

import { useRouter } from 'next/navigation';

export default function Modal({ children }: { children: React.ReactNode }) {
  const router = useRouter();

  return (
    <div className="modal-overlay" onClick={() => router.back()}>
      <div className="modal-content" onClick={e => e.stopPropagation()}>
        <button className="close-btn" onClick={() => router.back()}>×</button>
        {children}
      </div>
    </div>
  );
}
```

---

## Search Params

### Accessing Search Params in Server Components

```tsx
// app/search/page.tsx
interface PageProps {
  searchParams: { q?: string; page?: string };
}

export default function SearchPage({ searchParams }: PageProps) {
  const query = searchParams.q || '';
  const page = parseInt(searchParams.page || '1');

  return (
    <div>
      <h1>Search Results for: {query}</h1>
      <p>Page: {page}</p>
    </div>
  );
}
```

### Accessing Search Params in Client Components

```tsx
'use client';

import { useSearchParams, useRouter, usePathname } from 'next/navigation';

export default function SearchFilters() {
  const searchParams = useSearchParams();
  const router = useRouter();
  const pathname = usePathname();

  const currentQuery = searchParams.get('q') || '';
  const currentCategory = searchParams.get('category') || 'all';

  const updateSearchParams = (key: string, value: string) => {
    const params = new URLSearchParams(searchParams);
    if (value) {
      params.set(key, value);
    } else {
      params.delete(key);
    }
    router.push(`${pathname}?${params.toString()}`);
  };

  return (
    <div className="filters">
      <input
        type="search"
        value={currentQuery}
        onChange={e => updateSearchParams('q', e.target.value)}
        placeholder="Search..."
      />
      <select
        value={currentCategory}
        onChange={e => updateSearchParams('category', e.target.value)}
      >
        <option value="all">All Categories</option>
        <option value="electronics">Electronics</option>
        <option value="clothing">Clothing</option>
      </select>
    </div>
  );
}
```

---

## Static and Dynamic Route Generation

### generateStaticParams

Pre-generate static routes at build time:

```tsx
// app/blog/[slug]/page.tsx
export async function generateStaticParams() {
  const posts = await getPosts();

  return posts.map((post) => ({
    slug: post.slug,
  }));
}

export default function BlogPost({ params }: { params: { slug: string } }) {
  return <h1>Post: {params.slug}</h1>;
}
```

### Dynamic Segments with Catch-All

```tsx
// app/docs/[...slug]/page.tsx
export async function generateStaticParams() {
  return [
    { slug: ['getting-started'] },
    { slug: ['getting-started', 'installation'] },
    { slug: ['api', 'reference'] },
    { slug: ['api', 'reference', 'components'] },
  ];
}
```

### Route Segment Config

```tsx
// Force dynamic rendering
export const dynamic = 'force-dynamic';

// Force static rendering
export const dynamic = 'force-static';

// Revalidate every 60 seconds
export const revalidate = 60;

// Don't generate at build time, generate on first request
export const dynamicParams = true;
```

---

## Route Handlers

API routes in the app directory:

### Basic Route Handler

```tsx
// app/api/users/route.ts
import { NextResponse } from 'next/server';

export async function GET() {
  const users = await getUsers();
  return NextResponse.json(users);
}

export async function POST(request: Request) {
  const body = await request.json();
  const user = await createUser(body);
  return NextResponse.json(user, { status: 201 });
}
```

### Dynamic Route Handlers

```tsx
// app/api/users/[id]/route.ts
import { NextResponse } from 'next/server';

interface Props {
  params: { id: string };
}

export async function GET(request: Request, { params }: Props) {
  const user = await getUser(params.id);

  if (!user) {
    return NextResponse.json(
      { error: 'User not found' },
      { status: 404 }
    );
  }

  return NextResponse.json(user);
}

export async function PUT(request: Request, { params }: Props) {
  const body = await request.json();
  const user = await updateUser(params.id, body);
  return NextResponse.json(user);
}

export async function DELETE(request: Request, { params }: Props) {
  await deleteUser(params.id);
  return new NextResponse(null, { status: 204 });
}
```

### Request Helpers

```tsx
// app/api/search/route.ts
import { NextResponse } from 'next/server';

export async function GET(request: Request) {
  const { searchParams } = new URL(request.url);
  const query = searchParams.get('q');
  const page = searchParams.get('page') || '1';

  // Get headers
  const authHeader = request.headers.get('authorization');

  // Get cookies
  const cookies = request.cookies;
  const token = cookies.get('token');

  return NextResponse.json({ query, page });
}
```

---

## Middleware

Route middleware for authentication, redirects, etc.:

```tsx
// middleware.ts (at project root)
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  const token = request.cookies.get('token');
  const { pathname } = request.nextUrl;

  // Protect dashboard routes
  if (pathname.startsWith('/dashboard')) {
    if (!token) {
      return NextResponse.redirect(new URL('/login', request.url));
    }
  }

  // Redirect logged in users away from auth pages
  if (pathname.startsWith('/login') || pathname.startsWith('/signup')) {
    if (token) {
      return NextResponse.redirect(new URL('/dashboard', request.url));
    }
  }

  // Add custom header
  const response = NextResponse.next();
  response.headers.set('x-custom-header', 'my-value');

  return response;
}

// Configure which paths middleware runs on
export const config = {
  matcher: [
    '/dashboard/:path*',
    '/login',
    '/signup',
  ],
};
```

### Middleware Patterns

```tsx
// Geolocation-based redirect
export function middleware(request: NextRequest) {
  const country = request.geo?.country || 'US';

  if (country === 'GB' && request.nextUrl.pathname === '/') {
    return NextResponse.redirect(new URL('/uk', request.url));
  }

  return NextResponse.next();
}

// A/B Testing
export function middleware(request: NextRequest) {
  const bucket = request.cookies.get('bucket')?.value ||
    (Math.random() < 0.5 ? 'a' : 'b');

  const response = NextResponse.next();

  if (!request.cookies.get('bucket')) {
    response.cookies.set('bucket', bucket);
  }

  // Rewrite to variant
  if (bucket === 'b') {
    return NextResponse.rewrite(new URL('/home-b', request.url));
  }

  return response;
}
```

---

## Exercises

### Exercise 1: E-commerce Routes

Create routes for an e-commerce site:

```
/                         # Home
/products                 # All products
/products/[category]      # Products by category
/products/[category]/[id] # Single product
/cart                     # Shopping cart
/checkout                 # Checkout
/orders                   # Order history
/orders/[id]              # Single order
```

### Exercise 2: Dashboard with Parallel Routes

Create a dashboard with parallel routes:

- Main content area
- Stats widget (@stats)
- Recent activity (@activity)
- Quick actions (@actions)

---

## Solution 1: E-commerce Routes

```
app/
├── page.tsx                              # Home
├── products/
│   ├── page.tsx                          # All products
│   └── [category]/
│       ├── page.tsx                      # Category products
│       └── [id]/
│           └── page.tsx                  # Single product
├── cart/
│   └── page.tsx                          # Cart
├── checkout/
│   └── page.tsx                          # Checkout
└── orders/
    ├── page.tsx                          # Order history
    └── [id]/
        └── page.tsx                      # Single order
```

```tsx
// app/products/[category]/[id]/page.tsx
interface PageProps {
  params: {
    category: string;
    id: string;
  };
}

export default async function ProductPage({ params }: PageProps) {
  const product = await getProduct(params.id);

  return (
    <div>
      <nav className="breadcrumb">
        <a href="/products">Products</a>
        <span>/</span>
        <a href={`/products/${params.category}`}>{params.category}</a>
        <span>/</span>
        <span>{product.name}</span>
      </nav>

      <h1>{product.name}</h1>
      <p>{product.description}</p>
      <p>${product.price}</p>
    </div>
  );
}

// Generate static params for popular products
export async function generateStaticParams() {
  const products = await getPopularProducts();

  return products.map(p => ({
    category: p.category,
    id: p.id,
  }));
}
```

## Solution 2: Dashboard Parallel Routes

```
app/
└── dashboard/
    ├── layout.tsx
    ├── page.tsx
    ├── @stats/
    │   ├── page.tsx
    │   └── default.tsx
    ├── @activity/
    │   ├── page.tsx
    │   └── default.tsx
    └── @actions/
        ├── page.tsx
        └── default.tsx
```

```tsx
// app/dashboard/layout.tsx
export default function DashboardLayout({
  children,
  stats,
  activity,
  actions,
}: {
  children: React.ReactNode;
  stats: React.ReactNode;
  activity: React.ReactNode;
  actions: React.ReactNode;
}) {
  return (
    <div className="dashboard-layout">
      <div className="dashboard-header">
        <h1>Dashboard</h1>
        {actions}
      </div>

      <div className="dashboard-grid">
        <div className="main-content">{children}</div>
        <div className="sidebar">
          <div className="widget">{stats}</div>
          <div className="widget">{activity}</div>
        </div>
      </div>
    </div>
  );
}

// app/dashboard/@stats/page.tsx
export default async function StatsSlot() {
  const stats = await getStats();

  return (
    <div className="stats-widget">
      <h3>Statistics</h3>
      <div className="stat">
        <span>Revenue</span>
        <strong>${stats.revenue}</strong>
      </div>
      <div className="stat">
        <span>Orders</span>
        <strong>{stats.orders}</strong>
      </div>
    </div>
  );
}

// app/dashboard/@activity/page.tsx
export default async function ActivitySlot() {
  const activities = await getRecentActivity();

  return (
    <div className="activity-widget">
      <h3>Recent Activity</h3>
      <ul>
        {activities.map(activity => (
          <li key={activity.id}>
            {activity.action} - {activity.time}
          </li>
        ))}
      </ul>
    </div>
  );
}

// app/dashboard/@actions/page.tsx
import Link from 'next/link';

export default function ActionsSlot() {
  return (
    <div className="quick-actions">
      <Link href="/dashboard/new-order" className="btn">
        New Order
      </Link>
      <Link href="/dashboard/reports" className="btn">
        Reports
      </Link>
    </div>
  );
}
```

---

## Key Takeaways

1. **Dynamic routes** use `[param]` folder naming
2. **Catch-all routes** use `[...param]` for flexible paths
3. **Route groups** `(folder)` organize without URL impact
4. **Parallel routes** `@folder` render multiple pages simultaneously
5. **Intercepting routes** `(.)` show routes in modals
6. **generateStaticParams** pre-renders dynamic routes
7. **Middleware** handles auth, redirects, and request modifications

---

## What's Next?

Tomorrow we'll explore **Data Fetching** - Server Components, caching, revalidation, and the best patterns for loading data in Next.js!
