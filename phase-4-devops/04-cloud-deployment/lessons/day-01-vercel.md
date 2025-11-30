# Day 1: Vercel - Deploying Frontend Applications

## Introduction

Vercel is the platform built by the creators of Next.js, optimized for frontend frameworks and serverless functions. It offers zero-configuration deployments, automatic HTTPS, global CDN, and seamless Git integration. Today, you'll learn to deploy and manage applications on Vercel.

## Learning Objectives

By the end of this lesson, you will be able to:
- Deploy applications to Vercel
- Configure environment variables and domains
- Set up preview deployments
- Use Vercel serverless functions
- Optimize performance with Edge Functions

---

## Getting Started with Vercel

### Creating an Account

```bash
# 1. Sign up at vercel.com
# 2. Connect your GitHub/GitLab/Bitbucket account
# 3. Install Vercel CLI (optional)

npm install -g vercel

# Login
vercel login
```

### Deploying Your First App

```bash
# From project directory
vercel

# Follow prompts:
# ? Set up and deploy? Yes
# ? Which scope? your-username
# ? Link to existing project? No
# ? Project name? my-app
# ? Directory? ./
# ? Override settings? No

# Your app is now live!
# https://my-app-abc123.vercel.app
```

### Git Integration

```bash
# Simply push to GitHub - Vercel deploys automatically

git add .
git commit -m "Add new feature"
git push origin main

# Vercel detects the push and deploys
# Production: main branch
# Preview: all other branches/PRs
```

---

## Project Configuration

### vercel.json

```json
{
  "version": 2,
  "name": "my-app",
  "builds": [
    {
      "src": "package.json",
      "use": "@vercel/next"
    }
  ],
  "routes": [
    {
      "src": "/api/(.*)",
      "dest": "/api/$1"
    },
    {
      "src": "/(.*)",
      "dest": "/$1"
    }
  ],
  "env": {
    "NODE_ENV": "production"
  },
  "build": {
    "env": {
      "NEXT_PUBLIC_API_URL": "https://api.example.com"
    }
  },
  "headers": [
    {
      "source": "/api/(.*)",
      "headers": [
        {
          "key": "Cache-Control",
          "value": "s-maxage=60, stale-while-revalidate"
        }
      ]
    }
  ],
  "redirects": [
    {
      "source": "/old-page",
      "destination": "/new-page",
      "permanent": true
    }
  ],
  "rewrites": [
    {
      "source": "/blog/:slug",
      "destination": "/posts/:slug"
    }
  ]
}
```

### Framework Presets

```json
// Next.js (automatic)
{
  "framework": "nextjs"
}

// React (Create React App)
{
  "framework": "create-react-app",
  "buildCommand": "npm run build",
  "outputDirectory": "build"
}

// Vite
{
  "framework": "vite",
  "buildCommand": "npm run build",
  "outputDirectory": "dist"
}

// Static HTML
{
  "buildCommand": null,
  "outputDirectory": "public"
}
```

---

## Environment Variables

### Setting Variables

```bash
# Via CLI
vercel env add DATABASE_URL production
vercel env add API_KEY preview development

# List variables
vercel env ls

# Remove variable
vercel env rm DATABASE_URL production

# Pull variables locally
vercel env pull .env.local
```

### Environment Types

```
┌─────────────────────────────────────────────────────────────────┐
│                    Environment Variables                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Production    → Main branch deployments                        │
│  Preview       → PR/branch deployments                          │
│  Development   → Local development (vercel dev)                 │
│                                                                  │
│  Variable Types:                                                 │
│  • Plain text  → Visible in logs                                │
│  • Secret      → Encrypted, not shown in logs                   │
│  • Sensitive   → Encrypted, can't be read after creation        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Using in Code

```javascript
// Next.js - Server-side
const apiKey = process.env.API_KEY;

// Next.js - Client-side (must prefix with NEXT_PUBLIC_)
const publicUrl = process.env.NEXT_PUBLIC_API_URL;

// Environment-specific config
const config = {
  apiUrl: process.env.NEXT_PUBLIC_API_URL,
  debug: process.env.NODE_ENV === 'development'
};
```

---

## Custom Domains

### Adding a Domain

```bash
# Via CLI
vercel domains add example.com
vercel domains add www.example.com

# List domains
vercel domains ls

# Verify domain
vercel domains verify example.com
```

### DNS Configuration

```
# For apex domain (example.com)
Type: A
Name: @
Value: 76.76.21.21

# For subdomain (www.example.com)
Type: CNAME
Name: www
Value: cname.vercel-dns.com

# For wildcard (*.example.com)
Type: CNAME
Name: *
Value: cname.vercel-dns.com
```

### Domain in vercel.json

```json
{
  "alias": ["example.com", "www.example.com"]
}
```

---

## Serverless Functions

### API Routes (Next.js)

```typescript
// app/api/users/route.ts (App Router)
import { NextResponse } from 'next/server';

export async function GET() {
  const users = await db.user.findMany();
  return NextResponse.json(users);
}

export async function POST(request: Request) {
  const body = await request.json();
  const user = await db.user.create({ data: body });
  return NextResponse.json(user, { status: 201 });
}

// pages/api/users.ts (Pages Router)
import type { NextApiRequest, NextApiResponse } from 'next';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method === 'GET') {
    const users = await db.user.findMany();
    return res.json(users);
  }

  if (req.method === 'POST') {
    const user = await db.user.create({ data: req.body });
    return res.status(201).json(user);
  }

  res.status(405).end();
}
```

### Standalone Serverless Functions

```typescript
// api/hello.ts
import type { VercelRequest, VercelResponse } from '@vercel/node';

export default function handler(
  request: VercelRequest,
  response: VercelResponse
) {
  const { name = 'World' } = request.query;
  response.status(200).json({ message: `Hello ${name}!` });
}
```

### Function Configuration

```json
// vercel.json
{
  "functions": {
    "api/heavy-task.ts": {
      "memory": 1024,
      "maxDuration": 60
    },
    "api/quick.ts": {
      "memory": 128,
      "maxDuration": 10
    }
  }
}
```

---

## Edge Functions

### Edge Runtime

```typescript
// app/api/geo/route.ts
export const runtime = 'edge';

export async function GET(request: Request) {
  const { geo } = request;

  return new Response(
    JSON.stringify({
      country: geo?.country,
      city: geo?.city,
      region: geo?.region
    }),
    {
      headers: { 'Content-Type': 'application/json' }
    }
  );
}
```

### Edge Middleware

```typescript
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  // Geo-based routing
  const country = request.geo?.country || 'US';

  if (country === 'DE') {
    return NextResponse.redirect(new URL('/de', request.url));
  }

  // A/B testing
  const bucket = Math.random() < 0.5 ? 'a' : 'b';
  const response = NextResponse.next();
  response.cookies.set('bucket', bucket);

  return response;
}

export const config = {
  matcher: ['/((?!api|_next/static|favicon.ico).*)']
};
```

### Edge vs Serverless

```
┌─────────────────────────────────────────────────────────────────┐
│              Edge Functions vs Serverless Functions              │
├─────────────────┬───────────────────┬───────────────────────────┤
│                 │ Edge Functions    │ Serverless Functions      │
├─────────────────┼───────────────────┼───────────────────────────┤
│ Location        │ Global (CDN edge) │ Regional                  │
│ Cold start      │ ~0ms              │ ~250ms                    │
│ Max duration    │ 30s               │ 60s (Pro), 300s (Ent)     │
│ Memory          │ 128MB             │ Up to 3008MB              │
│ Node.js APIs    │ Limited (Web API) │ Full                      │
│ Use case        │ Auth, routing     │ Database, heavy compute   │
└─────────────────┴───────────────────┴───────────────────────────┘
```

---

## Preview Deployments

### Automatic Previews

```bash
# Every PR gets a unique preview URL
# https://my-app-git-feature-branch-username.vercel.app

# Comment on PR with preview link
# Automatically updated on each push
```

### Preview Environment Variables

```bash
# Set preview-specific variables
vercel env add DATABASE_URL preview

# Use preview database for testing
DATABASE_URL=postgres://preview-db.example.com/preview
```

### Protected Previews

```json
// vercel.json
{
  "github": {
    "enabled": true,
    "silent": false
  }
}
```

```bash
# Enable password protection in Vercel dashboard
# Settings > General > Password Protection
```

---

## Caching and Performance

### Cache Headers

```json
// vercel.json
{
  "headers": [
    {
      "source": "/static/(.*)",
      "headers": [
        {
          "key": "Cache-Control",
          "value": "public, max-age=31536000, immutable"
        }
      ]
    },
    {
      "source": "/api/(.*)",
      "headers": [
        {
          "key": "Cache-Control",
          "value": "s-maxage=60, stale-while-revalidate=300"
        }
      ]
    }
  ]
}
```

### ISR (Incremental Static Regeneration)

```typescript
// Next.js App Router
export const revalidate = 60; // Revalidate every 60 seconds

async function Page() {
  const data = await fetchData();
  return <div>{data}</div>;
}

// Next.js Pages Router
export async function getStaticProps() {
  const data = await fetchData();

  return {
    props: { data },
    revalidate: 60
  };
}
```

### Edge Caching

```typescript
// API route with edge caching
export async function GET() {
  const data = await fetchExpensiveData();

  return new Response(JSON.stringify(data), {
    headers: {
      'Content-Type': 'application/json',
      'Cache-Control': 's-maxage=3600, stale-while-revalidate'
    }
  });
}
```

---

## Vercel CLI Commands

### Common Commands

```bash
# Deploy
vercel                    # Deploy to preview
vercel --prod             # Deploy to production

# Development
vercel dev                # Local development server
vercel dev --listen 3001  # Custom port

# Environment
vercel env ls             # List variables
vercel env pull           # Pull to .env.local
vercel env add NAME       # Add variable

# Domains
vercel domains ls         # List domains
vercel domains add        # Add domain

# Projects
vercel projects ls        # List projects
vercel link               # Link directory to project
vercel unlink             # Unlink project

# Logs
vercel logs               # View deployment logs
vercel logs --follow      # Stream logs

# Inspect
vercel inspect [url]      # Deployment details
```

---

## Deployment Configuration

### Build Settings

```json
// vercel.json
{
  "buildCommand": "npm run build",
  "devCommand": "npm run dev",
  "installCommand": "npm install",
  "outputDirectory": "dist",
  "framework": "nextjs"
}
```

### Monorepo Setup

```json
// vercel.json (in root)
{
  "projects": [
    {
      "name": "frontend",
      "root": "apps/web"
    },
    {
      "name": "api",
      "root": "apps/api"
    }
  ]
}
```

### Ignored Build Step

```bash
# vercel.json or in dashboard
{
  "git": {
    "deploymentEnabled": {
      "main": true,
      "develop": true,
      "feature/*": false
    }
  }
}

# Or use ignore script
# Settings > Git > Ignored Build Step
# Command: git diff HEAD^ HEAD --quiet ./apps/web
```

---

## Practice Exercises

### Exercise 1: Deploy Next.js App

```bash
# 1. Create a Next.js app
# 2. Push to GitHub
# 3. Connect to Vercel
# 4. Add environment variables
# 5. Add custom domain
```

### Exercise 2: API Routes

```typescript
# Create API routes that:
# 1. Return data from a database
# 2. Handle authentication
# 3. Use proper caching headers
```

### Exercise 3: Preview Deployments

```bash
# 1. Create a feature branch
# 2. Open a PR
# 3. Test the preview deployment
# 4. Merge and verify production
```

---

## Quick Reference

### CLI Commands

| Command | Description |
|---------|-------------|
| `vercel` | Deploy to preview |
| `vercel --prod` | Deploy to production |
| `vercel dev` | Local development |
| `vercel env ls` | List env variables |
| `vercel domains ls` | List domains |
| `vercel logs` | View logs |

### Configuration

| Setting | File/Location |
|---------|---------------|
| Build config | `vercel.json` |
| Env variables | Dashboard / CLI |
| Domains | Dashboard / CLI |
| Team settings | Dashboard |

---

## Key Takeaways

1. **Zero-config deployment** - Push to Git, auto-deploy
2. **Preview deployments** - Every PR gets a URL
3. **Edge Functions** - Global, fast execution
4. **Built-in CDN** - Automatic caching
5. **Serverless functions** - API routes included
6. **Environment separation** - Production/Preview/Development

---

## What's Next?

Tomorrow, we'll learn about **Railway** - deploying full-stack applications with databases, background workers, and more complex infrastructure.
