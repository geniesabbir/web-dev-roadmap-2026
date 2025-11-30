# Next.js

**Duration:** 3-4 weeks

## Learning Objectives

By the end of this section, you will:
- Build full-stack applications with Next.js
- Understand server and client components
- Implement various data fetching strategies
- Create API routes
- Deploy Next.js applications
- Use the App Router effectively

---

## Week 1: Foundations

### Day 1: Getting Started
**Topics:**
- What is Next.js and why use it?
- App Router vs Pages Router
- Project setup
- File-based routing
- Project structure best practices

**Setup:**
```bash
npx create-next-app@latest my-nextjs-app --typescript --tailwind --eslint --app
cd my-nextjs-app
npm run dev
```

**Project Structure:**
```
my-nextjs-app/
├── app/
│   ├── layout.tsx      # Root layout
│   ├── page.tsx        # Home page
│   ├── globals.css
│   └── favicon.ico
├── components/         # Shared components
├── lib/               # Utility functions
├── public/            # Static assets
└── next.config.js
```

**Exercises:**
```tsx
// exercises/01-setup/

// 1. Create a Next.js project
// 2. Explore the default files
// 3. Modify the home page
// 4. Add a new route
```

### Day 2: Routing
**Topics:**
- File-based routing
- Dynamic routes `[id]`
- Route groups `(group)`
- Parallel routes `@folder`
- Intercepting routes
- Route handlers

**Exercises:**
```tsx
// exercises/02-routing/

// 1. Create these routes:
//    /about
//    /blog
//    /blog/[slug]
//    /products/[category]/[id]
// 2. Use route groups for organization
// 3. Create a catch-all route [...slug]
```

### Day 3: Layouts and Pages
**Topics:**
- Root layout
- Nested layouts
- Templates vs layouts
- Loading UI
- Error handling
- not-found pages

**Exercises:**
```tsx
// exercises/03-layouts/

// 1. Create a root layout with header/footer
// 2. Create a dashboard layout with sidebar
// 3. Add loading.tsx for loading states
// 4. Add error.tsx for error handling
// 5. Create a custom 404 page
```

### Day 4: Navigation
**Topics:**
- Link component
- useRouter hook
- usePathname and useSearchParams
- Programmatic navigation
- Active links
- Scroll behavior

**Exercises:**
```tsx
// exercises/04-navigation/

// 1. Build a navbar with active states
// 2. Implement programmatic navigation
// 3. Handle search params
// 4. Create breadcrumbs component
```

### Day 5: Server vs Client Components
**Topics:**
- Server Components (default)
- Client Components ('use client')
- When to use each
- Composition patterns
- Serialization boundaries
- Best practices

**Exercises:**
```tsx
// exercises/05-server-client/

// 1. Identify which components need 'use client'
// 2. Refactor a page to minimize client components
// 3. Pass server data to client components
// 4. Use composition for interactivity
```

---

## Week 2: Data Fetching

### Day 1: Server-Side Data Fetching
**Topics:**
- Fetching in Server Components
- async/await in components
- Caching behavior
- Revalidation strategies
- Parallel data fetching

**Exercises:**
```tsx
// exercises/06-server-fetching/

// 1. Fetch data in a page component
// 2. Fetch in parallel with Promise.all
// 3. Implement revalidation
// 4. Handle errors gracefully
```

### Day 2: Client-Side Data Fetching
**Topics:**
- When to fetch on client
- SWR / React Query
- Loading and error states
- Optimistic updates
- Infinite scrolling

**Exercises:**
```tsx
// exercises/07-client-fetching/

// 1. Set up React Query
// 2. Implement search with debounce
// 3. Build infinite scroll list
// 4. Add optimistic updates
```

### Day 3: Static Generation (SSG)
**Topics:**
- Static rendering (default)
- generateStaticParams
- ISR (Incremental Static Regeneration)
- On-demand revalidation
- When to use SSG

**Exercises:**
```tsx
// exercises/08-static-generation/

// 1. Create static blog pages
// 2. Generate params for dynamic routes
// 3. Implement ISR with revalidation
// 4. Add on-demand revalidation
```

### Day 4: Dynamic Rendering
**Topics:**
- Dynamic rendering triggers
- cookies() and headers()
- searchParams
- Dynamic route segments
- Streaming with Suspense

**Exercises:**
```tsx
// exercises/09-dynamic-rendering/

// 1. Create a page that reads cookies
// 2. Handle searchParams
// 3. Implement streaming with Suspense
// 4. Mix static and dynamic content
```

### Day 5: API Routes
**Topics:**
- Route Handlers
- HTTP methods (GET, POST, PUT, DELETE)
- Request and Response
- Dynamic API routes
- Middleware in API routes

**Exercises:**
```tsx
// exercises/10-api-routes/

// app/api/users/route.ts
// 1. Create CRUD API routes
// 2. Handle different HTTP methods
// 3. Parse request body
// 4. Return proper responses
```

---

## Week 3: Forms, Auth & Advanced

### Day 1: Server Actions
**Topics:**
- What are Server Actions?
- 'use server' directive
- Form handling
- useFormStatus
- useFormState
- Optimistic updates

**Exercises:**
```tsx
// exercises/11-server-actions/

// 1. Create a contact form with server action
// 2. Add loading states with useFormStatus
// 3. Handle validation errors
// 4. Implement optimistic UI
```

### Day 2: Forms with Server Actions
**Topics:**
- Progressive enhancement
- Form validation with Zod
- Error handling
- Redirects after actions
- Revalidation after mutations

**Exercises:**
```tsx
// exercises/12-forms/

// 1. Build a todo app with server actions
// 2. Add Zod validation
// 3. Display validation errors
// 4. Revalidate data after mutations
```

### Day 3: Authentication
**Topics:**
- Auth strategies overview
- NextAuth.js (Auth.js)
- Protected routes
- Middleware for auth
- Session management

**Exercises:**
```tsx
// exercises/13-auth/

// 1. Set up NextAuth with credentials
// 2. Add OAuth provider (GitHub)
// 3. Protect routes with middleware
// 4. Access session in components
```

### Day 4: Middleware
**Topics:**
- What is middleware?
- Middleware use cases
- Request/response modification
- Redirects and rewrites
- Conditional middleware

**Exercises:**
```tsx
// exercises/14-middleware/

// middleware.ts
// 1. Redirect unauthenticated users
// 2. Add security headers
// 3. Implement rate limiting
// 4. A/B testing with cookies
```

### Day 5: Optimization
**Topics:**
- Image optimization
- Font optimization
- Script optimization
- Metadata and SEO
- Analytics integration

**Exercises:**
```tsx
// exercises/15-optimization/

// 1. Optimize images with next/image
// 2. Add custom fonts with next/font
// 3. Configure metadata for SEO
// 4. Add Open Graph images
```

---

## Week 4: Project & Deployment

### Day 1-3: Build a Full Application
**Project: Blog Platform**
- Home page with posts list
- Individual post pages (SSG)
- Admin section (protected)
- Create/edit posts (Server Actions)
- Comments system
- Search functionality
- Dark mode
- SEO optimization

### Day 4: Testing
**Topics:**
- Testing Server Components
- Testing Client Components
- Testing API routes
- E2E with Playwright

### Day 5: Deployment
**Topics:**
- Vercel deployment
- Environment variables
- Build optimization
- Monitoring and analytics
- Edge vs Node runtime

**Exercises:**
```bash
# Deploy to Vercel
npm i -g vercel
vercel

# Or connect GitHub repo to Vercel
```

---

## Project Structure Template

```
app/
├── (auth)/
│   ├── login/
│   │   └── page.tsx
│   └── register/
│       └── page.tsx
├── (marketing)/
│   ├── layout.tsx
│   ├── page.tsx
│   ├── about/
│   │   └── page.tsx
│   └── pricing/
│       └── page.tsx
├── dashboard/
│   ├── layout.tsx
│   ├── page.tsx
│   ├── loading.tsx
│   ├── error.tsx
│   └── settings/
│       └── page.tsx
├── api/
│   ├── auth/
│   │   └── [...nextauth]/
│   │       └── route.ts
│   └── posts/
│       ├── route.ts
│       └── [id]/
│           └── route.ts
├── layout.tsx
├── not-found.tsx
└── globals.css

components/
├── ui/
│   ├── Button.tsx
│   ├── Card.tsx
│   └── Input.tsx
├── forms/
│   └── PostForm.tsx
└── layout/
    ├── Header.tsx
    ├── Footer.tsx
    └── Sidebar.tsx

lib/
├── actions/
│   └── posts.ts
├── db/
│   └── index.ts
└── utils/
    └── helpers.ts
```

---

## Common Patterns

### Server Action with Validation
```tsx
// lib/actions/posts.ts
'use server'

import { z } from 'zod'
import { revalidatePath } from 'next/cache'

const PostSchema = z.object({
  title: z.string().min(1, 'Title is required'),
  content: z.string().min(10, 'Content must be at least 10 characters'),
})

export async function createPost(formData: FormData) {
  const validated = PostSchema.safeParse({
    title: formData.get('title'),
    content: formData.get('content'),
  })

  if (!validated.success) {
    return { error: validated.error.flatten().fieldErrors }
  }

  // Save to database
  await db.post.create({ data: validated.data })

  revalidatePath('/posts')
  redirect('/posts')
}
```

### Protected API Route
```tsx
// app/api/posts/route.ts
import { getServerSession } from 'next-auth'
import { NextResponse } from 'next/server'

export async function POST(request: Request) {
  const session = await getServerSession()

  if (!session) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
  }

  const body = await request.json()
  // Process request...

  return NextResponse.json({ success: true })
}
```

---

## Resources

### Documentation
- [Next.js Documentation](https://nextjs.org/docs)
- [Next.js Learn Course](https://nextjs.org/learn)

### Examples
- [Next.js Examples](https://github.com/vercel/next.js/tree/canary/examples)
- [Vercel Templates](https://vercel.com/templates)

---

## Checklist Before Moving On

- [ ] Understand App Router structure
- [ ] Can create routes and layouts
- [ ] Know when to use Server vs Client components
- [ ] Can implement data fetching strategies
- [ ] Can create API routes
- [ ] Understand Server Actions
- [ ] Can implement authentication
- [ ] Know how to use middleware
- [ ] Can optimize for performance and SEO
- [ ] Deployed a Next.js app to Vercel
- [ ] Completed Blog Platform project

---

**Next:** [State Management](../04-state-management/README.md)
