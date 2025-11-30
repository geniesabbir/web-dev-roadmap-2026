# Day 4: Server Components - The New React Paradigm

## Introduction

React Server Components (RSC) represent a fundamental shift in how we build React applications. They allow components to render on the server, reducing client-side JavaScript and enabling direct access to backend resources. Next.js App Router is built on this paradigm.

## Learning Objectives

By the end of this lesson, you will:
- Understand the Server Component architecture
- Know when to use Server vs Client Components
- Master composition patterns for both types
- Optimize component boundaries for performance
- Handle the "use client" directive effectively

---

## The Server Component Model

### Traditional React (Client-Side)

```
Browser                        Server
┌─────────────────────┐       ┌──────────────┐
│  Download JS Bundle │◄──────│   Send HTML  │
│         │           │       │   + JS       │
│         ▼           │       └──────────────┘
│  Hydrate Components │
│         │           │              │
│         ▼           │              │
│   Render + Fetch    │─────────────►│
│         │           │              │
│         ▼           │◄─────────────│
│   Display Data      │         JSON Data
└─────────────────────┘
```

### Server Components

```
Server                         Browser
┌─────────────────────┐       ┌──────────────────┐
│  Render Components  │       │  Stream HTML     │
│         │           │──────►│       │          │
│  Access DB/APIs     │       │       ▼          │
│         │           │       │  Hydrate Client  │
│         ▼           │       │  Components Only │
│  Generate HTML/RSC  │       │                  │
└─────────────────────┘       └──────────────────┘
```

---

## Server Components (Default)

In Next.js App Router, all components are Server Components by default:

```tsx
// app/page.tsx - This is a Server Component
import { db } from '@/lib/db';

export default async function HomePage() {
  // ✅ Direct database access
  const posts = await db.post.findMany();

  // ✅ Access environment variables (including secrets)
  const apiKey = process.env.SECRET_API_KEY;

  // ✅ No client-side JavaScript shipped for this component
  return (
    <main>
      <h1>Latest Posts</h1>
      {posts.map(post => (
        <article key={post.id}>
          <h2>{post.title}</h2>
          <p>{post.excerpt}</p>
        </article>
      ))}
    </main>
  );
}
```

### Server Component Capabilities

| Can Do | Cannot Do |
|--------|-----------|
| Fetch data directly | Use React hooks (useState, useEffect) |
| Access backend resources | Add event listeners |
| Keep sensitive data on server | Access browser APIs |
| Import server-only code | Store component state |
| Stream large dependencies | Use Context providers |

---

## Client Components

Add `'use client'` directive for interactivity:

```tsx
// components/Counter.tsx
'use client';

import { useState } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(c => c + 1)}>
        Increment
      </button>
    </div>
  );
}
```

### Client Component Capabilities

| Can Do | Cannot Do |
|--------|-----------|
| Use React hooks | Import server-only code |
| Add event listeners | Access backend directly |
| Access browser APIs | Use async/await at component level |
| Manage component state | Import components without 'use client' |
| Use Context | Keep secrets secure |

---

## When to Use Which?

### Use Server Components For:

```tsx
// ✅ Data fetching
async function PostList() {
  const posts = await db.post.findMany();
  return <div>{posts.map(p => <Post key={p.id} post={p} />)}</div>;
}

// ✅ Accessing backend services
async function UserProfile({ userId }) {
  const user = await userService.getById(userId);
  return <Profile user={user} />;
}

// ✅ Rendering static content
function Footer() {
  return (
    <footer>
      <p>© 2025 My Company</p>
      <nav>{/* Static links */}</nav>
    </footer>
  );
}

// ✅ Large dependencies (render on server, don't ship to client)
import { marked } from 'marked';
import hljs from 'highlight.js';

function MarkdownRenderer({ content }) {
  const html = marked(content, { highlight: code => hljs.highlightAuto(code).value });
  return <div dangerouslySetInnerHTML={{ __html: html }} />;
}
```

### Use Client Components For:

```tsx
// ✅ Interactive elements
'use client';

function ToggleButton() {
  const [isOpen, setIsOpen] = useState(false);
  return <button onClick={() => setIsOpen(!isOpen)}>{isOpen ? 'Close' : 'Open'}</button>;
}

// ✅ Form inputs
'use client';

function SearchInput() {
  const [query, setQuery] = useState('');
  return <input value={query} onChange={e => setQuery(e.target.value)} />;
}

// ✅ Browser APIs
'use client';

function LocationDisplay() {
  const [coords, setCoords] = useState(null);

  useEffect(() => {
    navigator.geolocation.getCurrentPosition(pos => {
      setCoords({ lat: pos.coords.latitude, lng: pos.coords.longitude });
    });
  }, []);

  return coords ? <p>Lat: {coords.lat}, Lng: {coords.lng}</p> : <p>Loading...</p>;
}

// ✅ Event handlers
'use client';

function LikeButton({ postId }) {
  const handleLike = async () => {
    await fetch(`/api/posts/${postId}/like`, { method: 'POST' });
  };

  return <button onClick={handleLike}>Like</button>;
}
```

---

## Composition Patterns

### Pattern 1: Server Component with Client Children

```tsx
// app/page.tsx (Server Component)
import InteractiveWidget from '@/components/InteractiveWidget';

async function getData() {
  return await fetch('https://api.example.com/data');
}

export default async function Page() {
  const data = await getData();

  return (
    <div>
      {/* Server-rendered content */}
      <h1>{data.title}</h1>
      <p>{data.description}</p>

      {/* Client component for interactivity */}
      <InteractiveWidget initialData={data.widget} />
    </div>
  );
}
```

### Pattern 2: Passing Server Components as Children

```tsx
// components/ClientWrapper.tsx
'use client';

import { useState } from 'react';

export default function ClientWrapper({ children }) {
  const [isOpen, setIsOpen] = useState(true);

  return (
    <div>
      <button onClick={() => setIsOpen(!isOpen)}>
        {isOpen ? 'Hide' : 'Show'}
      </button>
      {isOpen && children}
    </div>
  );
}

// app/page.tsx (Server Component)
import ClientWrapper from '@/components/ClientWrapper';
import ServerContent from '@/components/ServerContent'; // Server Component

export default function Page() {
  return (
    <ClientWrapper>
      {/* ServerContent renders on server, passed as children */}
      <ServerContent />
    </ClientWrapper>
  );
}
```

### Pattern 3: Slots Pattern

```tsx
// components/Modal.tsx
'use client';

import { useState } from 'react';

export default function Modal({
  trigger,
  content,
}: {
  trigger: React.ReactNode;
  content: React.ReactNode;
}) {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <>
      <div onClick={() => setIsOpen(true)}>{trigger}</div>
      {isOpen && (
        <div className="modal-overlay" onClick={() => setIsOpen(false)}>
          <div className="modal-content" onClick={e => e.stopPropagation()}>
            {content}
          </div>
        </div>
      )}
    </>
  );
}

// app/page.tsx (Server Component)
import Modal from '@/components/Modal';
import ProductDetails from '@/components/ProductDetails'; // Server Component

export default async function Page() {
  const product = await getProduct();

  return (
    <Modal
      trigger={<button>View Details</button>}
      content={<ProductDetails product={product} />}
    />
  );
}
```

---

## Boundary Considerations

### The "use client" Boundary

```
Server Component Tree
         │
         ▼
┌─────────────────────┐
│  'use client'       │ ← Client boundary
│  ClientComponent    │
│    │                │
│    ▼                │
│  All children are   │
│  client components  │
└─────────────────────┘
```

### Pushing Client Boundary Down

```tsx
// ❌ Bad - Everything becomes client
'use client';

export default function ProductPage({ params }) {
  const [quantity, setQuantity] = useState(1);
  const product = useProduct(params.id); // Client-side fetch

  return (
    <div>
      <h1>{product.name}</h1>
      <p>{product.description}</p>
      <input value={quantity} onChange={e => setQuantity(e.target.value)} />
    </div>
  );
}

// ✅ Good - Only interactive part is client
// app/product/[id]/page.tsx (Server Component)
export default async function ProductPage({ params }) {
  const product = await getProduct(params.id); // Server fetch

  return (
    <div>
      <h1>{product.name}</h1>
      <p>{product.description}</p>
      <QuantityInput /> {/* Only this is client */}
    </div>
  );
}

// components/QuantityInput.tsx (Client Component)
'use client';

import { useState } from 'react';

export default function QuantityInput() {
  const [quantity, setQuantity] = useState(1);
  return <input value={quantity} onChange={e => setQuantity(e.target.value)} />;
}
```

---

## Data Patterns

### Fetching in Server Components

```tsx
// app/users/page.tsx
import { Suspense } from 'react';

// Async Server Component
async function UserList() {
  const users = await db.user.findMany();

  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}

export default function UsersPage() {
  return (
    <div>
      <h1>Users</h1>
      <Suspense fallback={<p>Loading users...</p>}>
        <UserList />
      </Suspense>
    </div>
  );
}
```

### Passing Data to Client Components

```tsx
// ✅ Good - Pass serializable data
// app/product/[id]/page.tsx
export default async function ProductPage({ params }) {
  const product = await getProduct(params.id);

  return (
    <div>
      <h1>{product.name}</h1>
      {/* Pass plain object, not database model */}
      <AddToCartButton
        productId={product.id}
        price={product.price}
      />
    </div>
  );
}

// ❌ Bad - Passing non-serializable data
export default async function Page() {
  const connection = await getDbConnection();
  return <ClientComponent connection={connection} />; // Error!
}
```

### Server Actions for Mutations

```tsx
// app/actions.ts
'use server';

import { revalidatePath } from 'next/cache';

export async function addToCart(productId: string, quantity: number) {
  await db.cartItem.create({
    data: { productId, quantity, userId: getCurrentUserId() },
  });
  revalidatePath('/cart');
}

// components/AddToCartButton.tsx
'use client';

import { addToCart } from '@/app/actions';

export default function AddToCartButton({ productId, price }) {
  const handleClick = async () => {
    await addToCart(productId, 1);
  };

  return (
    <button onClick={handleClick}>
      Add to Cart - ${price}
    </button>
  );
}
```

---

## Performance Optimization

### Streaming with Suspense

```tsx
// app/dashboard/page.tsx
import { Suspense } from 'react';

export default function Dashboard() {
  return (
    <div>
      <h1>Dashboard</h1>

      {/* These stream independently */}
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

async function Stats() {
  await delay(1000); // Simulated delay
  const stats = await getStats();
  return <StatsDisplay stats={stats} />;
}

async function RevenueChart() {
  await delay(2000); // Longer delay
  const data = await getRevenueData();
  return <Chart data={data} />;
}

async function RecentOrders() {
  await delay(500); // Fast
  const orders = await getOrders();
  return <OrdersTable orders={orders} />;
}
```

### Preloading Data

```tsx
// lib/data.ts
import { cache } from 'react';

export const getUser = cache(async (id: string) => {
  const user = await db.user.findUnique({ where: { id } });
  return user;
});

export const preloadUser = (id: string) => {
  void getUser(id);
};

// app/user/[id]/page.tsx
import { getUser, preloadUser } from '@/lib/data';

export default async function UserPage({ params }) {
  // Start loading immediately
  preloadUser(params.id);

  // Do other work...

  // Use the preloaded data
  const user = await getUser(params.id);

  return <UserProfile user={user} />;
}
```

### Parallel Data Fetching

```tsx
export default async function Page() {
  // ❌ Sequential - slow
  const user = await getUser();
  const posts = await getPosts();
  const comments = await getComments();

  // ✅ Parallel - fast
  const [user, posts, comments] = await Promise.all([
    getUser(),
    getPosts(),
    getComments(),
  ]);

  return <div>...</div>;
}
```

---

## Common Mistakes

### 1. Using Hooks in Server Components

```tsx
// ❌ Wrong - Can't use hooks in Server Components
export default async function Page() {
  const [count, setCount] = useState(0); // Error!
  return <div>{count}</div>;
}

// ✅ Correct - Extract to Client Component
// components/Counter.tsx
'use client';
export default function Counter() {
  const [count, setCount] = useState(0);
  return <div>{count}</div>;
}

// app/page.tsx
import Counter from '@/components/Counter';
export default function Page() {
  return <Counter />;
}
```

### 2. Importing Server Code in Client Components

```tsx
// ❌ Wrong - Can't import server-only code in client
'use client';

import { db } from '@/lib/db'; // Error!

export default function Component() {
  // ...
}

// ✅ Correct - Use Server Action
'use client';

import { getData } from '@/app/actions';

export default function Component() {
  const handleClick = async () => {
    const data = await getData();
  };
}
```

### 3. Not Serializing Data

```tsx
// ❌ Wrong - Functions aren't serializable
export default async function Page() {
  const user = await getUser();
  return <ClientComponent onClick={() => console.log(user)} />; // Error!
}

// ✅ Correct - Pass serializable data
export default async function Page() {
  const user = await getUser();
  return <ClientComponent userId={user.id} />;
}
```

---

## Exercise: Build a Dashboard

Create a dashboard with:
1. Server-rendered stats
2. Interactive chart (client)
3. Real-time notifications (client)
4. User profile with edit (mixed)

---

## Solution

```tsx
// app/dashboard/page.tsx
import { Suspense } from 'react';
import Stats from '@/components/Stats';
import InteractiveChart from '@/components/InteractiveChart';
import NotificationBell from '@/components/NotificationBell';
import UserProfile from '@/components/UserProfile';
import { getUser, getStats, getChartData } from '@/lib/data';

export default async function DashboardPage() {
  // Parallel fetch
  const [user, stats, chartData] = await Promise.all([
    getUser(),
    getStats(),
    getChartData(),
  ]);

  return (
    <div className="dashboard">
      <header className="flex justify-between">
        <h1>Dashboard</h1>
        <NotificationBell userId={user.id} />
      </header>

      <Suspense fallback={<StatsSkeleton />}>
        <Stats data={stats} />
      </Suspense>

      <div className="grid grid-cols-2 gap-4">
        <InteractiveChart initialData={chartData} />

        <Suspense fallback={<ProfileSkeleton />}>
          <UserProfile user={user} />
        </Suspense>
      </div>
    </div>
  );
}

// components/Stats.tsx (Server Component)
export default function Stats({ data }) {
  return (
    <div className="grid grid-cols-4 gap-4">
      <StatCard title="Revenue" value={`$${data.revenue}`} />
      <StatCard title="Orders" value={data.orders} />
      <StatCard title="Customers" value={data.customers} />
      <StatCard title="Growth" value={`${data.growth}%`} />
    </div>
  );
}

// components/InteractiveChart.tsx (Client Component)
'use client';

import { useState } from 'react';
import { LineChart, Line, XAxis, YAxis } from 'recharts';

export default function InteractiveChart({ initialData }) {
  const [timeRange, setTimeRange] = useState('week');
  const [data, setData] = useState(initialData);

  const handleTimeRangeChange = async (range: string) => {
    setTimeRange(range);
    const newData = await fetch(`/api/chart?range=${range}`).then(r => r.json());
    setData(newData);
  };

  return (
    <div className="chart-container">
      <div className="flex gap-2 mb-4">
        {['day', 'week', 'month', 'year'].map(range => (
          <button
            key={range}
            onClick={() => handleTimeRangeChange(range)}
            className={timeRange === range ? 'active' : ''}
          >
            {range}
          </button>
        ))}
      </div>
      <LineChart data={data} width={400} height={300}>
        <XAxis dataKey="date" />
        <YAxis />
        <Line type="monotone" dataKey="value" stroke="#8884d8" />
      </LineChart>
    </div>
  );
}

// components/NotificationBell.tsx (Client Component)
'use client';

import { useState, useEffect } from 'react';
import useSWR from 'swr';

export default function NotificationBell({ userId }) {
  const [isOpen, setIsOpen] = useState(false);
  const { data: notifications } = useSWR(
    `/api/notifications/${userId}`,
    fetcher,
    { refreshInterval: 30000 }
  );

  const unreadCount = notifications?.filter(n => !n.read).length || 0;

  return (
    <div className="relative">
      <button onClick={() => setIsOpen(!isOpen)}>
        🔔 {unreadCount > 0 && <span className="badge">{unreadCount}</span>}
      </button>
      {isOpen && (
        <div className="dropdown">
          {notifications?.map(n => (
            <div key={n.id} className={n.read ? 'read' : 'unread'}>
              {n.message}
            </div>
          ))}
        </div>
      )}
    </div>
  );
}

// components/UserProfile.tsx (Mixed - Server with Client parts)
import EditProfileButton from './EditProfileButton';

export default function UserProfile({ user }) {
  return (
    <div className="profile-card">
      <img src={user.avatar} alt={user.name} />
      <h2>{user.name}</h2>
      <p>{user.email}</p>
      <p>{user.bio}</p>
      <EditProfileButton userId={user.id} />
    </div>
  );
}

// components/EditProfileButton.tsx (Client Component)
'use client';

import { useState } from 'react';
import { updateProfile } from '@/app/actions';

export default function EditProfileButton({ userId }) {
  const [isEditing, setIsEditing] = useState(false);

  if (!isEditing) {
    return <button onClick={() => setIsEditing(true)}>Edit Profile</button>;
  }

  return (
    <form action={updateProfile}>
      <input type="hidden" name="userId" value={userId} />
      <input name="name" placeholder="Name" />
      <textarea name="bio" placeholder="Bio" />
      <button type="submit">Save</button>
      <button type="button" onClick={() => setIsEditing(false)}>Cancel</button>
    </form>
  );
}
```

---

## Key Takeaways

1. **Server Components are default** in Next.js App Router
2. **Use Client Components** only for interactivity (hooks, events, browser APIs)
3. **Push client boundaries down** to minimize client-side JavaScript
4. **Pass Server Components as children** to Client Components when needed
5. **Use Suspense** for streaming and independent loading states
6. **Server Actions** enable mutations from Client Components
7. **Serialize data** when passing to Client Components

---

## What's Next?

Tomorrow we'll learn about **API Routes** and **Server Actions** - building backend functionality directly in your Next.js application!
