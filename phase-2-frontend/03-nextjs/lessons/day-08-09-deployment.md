# Days 8-9: Next.js Deployment - Going to Production

## Introduction

Deploying a Next.js application requires understanding different hosting options, environment configuration, optimization techniques, and monitoring. This lesson covers deploying to Vercel (the creators of Next.js), as well as other platforms like Railway, Docker, and self-hosting options.

## Learning Objectives

By the end of this lesson, you will:
- Deploy to Vercel with optimal configuration
- Set up environment variables for production
- Configure custom domains and SSL
- Implement CI/CD pipelines
- Optimize for production performance
- Monitor and debug production issues

---

## Production Checklist

Before deploying, ensure your app is production-ready:

```
□ Environment variables configured
□ Database connected and migrated
□ Authentication secrets set
□ Error boundaries implemented
□ Loading states for all async operations
□ SEO metadata on all pages
□ Images optimized
□ No console.log in production code
□ Security headers configured
□ Rate limiting on API routes
```

---

## Deploying to Vercel

### Quick Deploy

1. Push code to GitHub
2. Go to [vercel.com](https://vercel.com)
3. Import your repository
4. Configure environment variables
5. Deploy!

### Vercel CLI

```bash
# Install Vercel CLI
npm i -g vercel

# Login
vercel login

# Deploy (follow prompts)
vercel

# Deploy to production
vercel --prod

# Link existing project
vercel link
```

### Project Configuration

```json
// vercel.json
{
  "framework": "nextjs",
  "buildCommand": "npm run build",
  "devCommand": "npm run dev",
  "installCommand": "npm install",
  "regions": ["iad1"],
  "env": {
    "NEXT_PUBLIC_API_URL": "@next-public-api-url"
  },
  "headers": [
    {
      "source": "/api/(.*)",
      "headers": [
        { "key": "Access-Control-Allow-Origin", "value": "*" }
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

---

## Environment Variables

### Local Development

```
# .env.local (not committed)
DATABASE_URL=postgresql://localhost:5432/myapp
NEXTAUTH_SECRET=dev-secret-key
NEXTAUTH_URL=http://localhost:3000
```

### Production Variables

Set in Vercel Dashboard → Project → Settings → Environment Variables:

```
DATABASE_URL        → Production database URL
NEXTAUTH_SECRET     → Strong secret (npx auth secret)
NEXTAUTH_URL        → https://your-domain.com
GITHUB_ID           → OAuth client ID
GITHUB_SECRET       → OAuth client secret
```

### Environment Variable Types

```
Production    → Only available in production
Preview       → Available in preview deployments
Development   → Available locally (vercel dev)
```

### Runtime vs Build-time

```tsx
// NEXT_PUBLIC_ prefix → Available in browser (build-time)
const apiUrl = process.env.NEXT_PUBLIC_API_URL;

// No prefix → Server-only (runtime)
const dbUrl = process.env.DATABASE_URL; // Only in Server Components/API

// next.config.js
module.exports = {
  env: {
    // Build-time substitution
    CUSTOM_KEY: process.env.CUSTOM_KEY,
  },
};
```

---

## Database Deployment

### Vercel Postgres

```bash
# Install Vercel Postgres
npm i @vercel/postgres
```

```tsx
// lib/db.ts
import { sql } from '@vercel/postgres';

export async function getUsers() {
  const { rows } = await sql`SELECT * FROM users`;
  return rows;
}
```

### Prisma with External Database

```bash
# Generate Prisma Client
npx prisma generate

# Run migrations in production
npx prisma migrate deploy
```

```tsx
// lib/prisma.ts
import { PrismaClient } from '@prisma/client';

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined;
};

export const prisma = globalForPrisma.prisma ?? new PrismaClient();

if (process.env.NODE_ENV !== 'production') {
  globalForPrisma.prisma = prisma;
}
```

### Database Providers

- **Vercel Postgres** - Native integration
- **PlanetScale** - MySQL-compatible, serverless
- **Supabase** - PostgreSQL with extras
- **Railway** - PostgreSQL, easy setup
- **Neon** - Serverless PostgreSQL

---

## Custom Domain Setup

### Vercel Dashboard

1. Go to Project → Settings → Domains
2. Add your domain
3. Configure DNS:
   - A Record: `76.76.21.21`
   - CNAME: `cname.vercel-dns.com`

### DNS Configuration

```
# For apex domain (example.com)
Type: A
Name: @
Value: 76.76.21.21

# For www subdomain
Type: CNAME
Name: www
Value: cname.vercel-dns.com

# For custom subdomain
Type: CNAME
Name: app
Value: cname.vercel-dns.com
```

### SSL/HTTPS

Vercel provides automatic SSL certificates via Let's Encrypt - no configuration needed.

---

## CI/CD Pipeline

### GitHub Actions

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run linter
        run: npm run lint

      - name: Run type check
        run: npm run type-check

      - name: Run tests
        run: npm test

      - name: Build
        run: npm run build

  deploy-preview:
    needs: test
    runs-on: ubuntu-latest
    if: github.event_name == 'pull_request'

    steps:
      - uses: actions/checkout@v4

      - name: Deploy to Vercel (Preview)
        uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}

  deploy-production:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'

    steps:
      - uses: actions/checkout@v4

      - name: Deploy to Vercel (Production)
        uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          vercel-args: '--prod'
```

### package.json Scripts

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "type-check": "tsc --noEmit",
    "test": "jest",
    "test:e2e": "playwright test",
    "db:push": "prisma db push",
    "db:migrate": "prisma migrate deploy",
    "postinstall": "prisma generate"
  }
}
```

---

## Production Optimization

### Next.js Config

```tsx
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  // Strict mode for better development experience
  reactStrictMode: true,

  // Image optimization
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 'images.example.com',
      },
    ],
    formats: ['image/avif', 'image/webp'],
  },

  // Compression
  compress: true,

  // Headers for security
  async headers() {
    return [
      {
        source: '/:path*',
        headers: [
          {
            key: 'X-DNS-Prefetch-Control',
            value: 'on',
          },
          {
            key: 'Strict-Transport-Security',
            value: 'max-age=63072000; includeSubDomains; preload',
          },
          {
            key: 'X-Content-Type-Options',
            value: 'nosniff',
          },
          {
            key: 'X-Frame-Options',
            value: 'DENY',
          },
          {
            key: 'X-XSS-Protection',
            value: '1; mode=block',
          },
          {
            key: 'Referrer-Policy',
            value: 'origin-when-cross-origin',
          },
        ],
      },
    ];
  },

  // Redirects
  async redirects() {
    return [
      {
        source: '/old-blog/:slug',
        destination: '/blog/:slug',
        permanent: true,
      },
    ];
  },

  // Experimental features
  experimental: {
    // Enable PPR (Partial Prerendering)
    ppr: true,
  },
};

module.exports = nextConfig;
```

### Bundle Analysis

```bash
# Install analyzer
npm i @next/bundle-analyzer

# next.config.js
const withBundleAnalyzer = require('@next/bundle-analyzer')({
  enabled: process.env.ANALYZE === 'true',
});

module.exports = withBundleAnalyzer(nextConfig);

# Run analysis
ANALYZE=true npm run build
```

---

## Docker Deployment

### Dockerfile

```dockerfile
# Dockerfile
FROM node:20-alpine AS base

# Install dependencies only when needed
FROM base AS deps
RUN apk add --no-cache libc6-compat
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci

# Rebuild the source code only when needed
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .

# Generate Prisma Client
RUN npx prisma generate

# Build
ENV NEXT_TELEMETRY_DISABLED 1
RUN npm run build

# Production image
FROM base AS runner
WORKDIR /app

ENV NODE_ENV production
ENV NEXT_TELEMETRY_DISABLED 1

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000
ENV PORT 3000

CMD ["node", "server.js"]
```

### Docker Compose

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build: .
    ports:
      - '3000:3000'
    environment:
      - DATABASE_URL=postgresql://postgres:password@db:5432/myapp
      - NEXTAUTH_URL=http://localhost:3000
      - NEXTAUTH_SECRET=your-secret
    depends_on:
      - db

  db:
    image: postgres:15-alpine
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=myapp
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

### Next.js Standalone Output

```tsx
// next.config.js
module.exports = {
  output: 'standalone',
};
```

---

## Railway Deployment

```bash
# Install Railway CLI
npm i -g @railway/cli

# Login
railway login

# Initialize project
railway init

# Deploy
railway up

# Open dashboard
railway open
```

### railway.json

```json
{
  "$schema": "https://railway.app/railway.schema.json",
  "build": {
    "builder": "NIXPACKS"
  },
  "deploy": {
    "startCommand": "npm start",
    "healthcheckPath": "/api/health"
  }
}
```

---

## Monitoring and Analytics

### Vercel Analytics

```bash
npm i @vercel/analytics
```

```tsx
// app/layout.tsx
import { Analytics } from '@vercel/analytics/react';

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        {children}
        <Analytics />
      </body>
    </html>
  );
}
```

### Vercel Speed Insights

```bash
npm i @vercel/speed-insights
```

```tsx
// app/layout.tsx
import { SpeedInsights } from '@vercel/speed-insights/next';

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        {children}
        <SpeedInsights />
      </body>
    </html>
  );
}
```

### Error Tracking with Sentry

```bash
npx @sentry/wizard@latest -i nextjs
```

```tsx
// sentry.client.config.ts
import * as Sentry from '@sentry/nextjs';

Sentry.init({
  dsn: process.env.NEXT_PUBLIC_SENTRY_DSN,
  tracesSampleRate: 0.1,
  environment: process.env.NODE_ENV,
});
```

---

## Health Checks

### API Health Endpoint

```tsx
// app/api/health/route.ts
import { NextResponse } from 'next/server';
import { prisma } from '@/lib/prisma';

export async function GET() {
  try {
    // Check database connection
    await prisma.$queryRaw`SELECT 1`;

    return NextResponse.json({
      status: 'healthy',
      timestamp: new Date().toISOString(),
      services: {
        database: 'connected',
      },
    });
  } catch (error) {
    return NextResponse.json(
      {
        status: 'unhealthy',
        timestamp: new Date().toISOString(),
        error: 'Database connection failed',
      },
      { status: 503 }
    );
  }
}
```

---

## Caching Strategies

### Static Generation

```tsx
// Statically generated at build time
export default async function Page() {
  const data = await fetch('https://api.example.com/posts', {
    cache: 'force-cache',
  });
  // ...
}
```

### Incremental Static Regeneration

```tsx
// Regenerate every 60 seconds
export const revalidate = 60;

export default async function Page() {
  const data = await fetch('https://api.example.com/posts');
  // ...
}
```

### On-Demand Revalidation

```tsx
// app/api/revalidate/route.ts
import { revalidatePath, revalidateTag } from 'next/cache';
import { NextResponse } from 'next/server';

export async function POST(request: Request) {
  const { secret, path, tag } = await request.json();

  if (secret !== process.env.REVALIDATION_SECRET) {
    return NextResponse.json({ error: 'Invalid secret' }, { status: 401 });
  }

  if (path) {
    revalidatePath(path);
  }

  if (tag) {
    revalidateTag(tag);
  }

  return NextResponse.json({ revalidated: true });
}
```

---

## Deployment Checklist

```
Pre-Deployment:
□ All tests passing
□ No TypeScript errors
□ ESLint passing
□ Build successful locally
□ Environment variables documented

Environment Setup:
□ Production database ready
□ All secrets configured
□ OAuth callbacks updated for production URLs
□ Email service configured (if applicable)

Performance:
□ Images optimized
□ Code splitting implemented
□ Caching strategies defined
□ CDN configured for static assets

Security:
□ Security headers configured
□ Rate limiting on APIs
□ Input validation on all endpoints
□ CORS properly configured
□ Secrets not exposed in client code

Monitoring:
□ Error tracking setup (Sentry)
□ Analytics enabled
□ Health check endpoint
□ Logging configured
□ Alerts for errors/downtime

Post-Deployment:
□ Smoke test all critical paths
□ Verify OAuth login works
□ Check email delivery
□ Monitor error rates
□ Check Core Web Vitals
```

---

## Key Takeaways

1. **Vercel** provides the easiest deployment for Next.js
2. **Environment variables** must be properly configured for each environment
3. **Database migrations** should run during deployment
4. **CI/CD pipelines** ensure code quality before deployment
5. **Security headers** protect your application
6. **Monitoring** helps catch issues early
7. **Caching strategies** improve performance significantly

---

## Congratulations!

You've completed the Next.js module! You now know how to:
- Build full-stack applications with Next.js
- Implement authentication and authorization
- Deploy to production with confidence
- Monitor and maintain your application

**Next up: State Management** - Zustand, React Query, and advanced state patterns!
