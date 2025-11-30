# Performance Optimization

**Duration:** 2 weeks

## Learning Objectives

By the end of this section, you will:
- Optimize Core Web Vitals
- Improve backend response times
- Implement effective caching
- Profile and debug performance issues

---

## Week 1: Frontend Performance

### Core Web Vitals

**LCP (Largest Contentful Paint)** - < 2.5s
- Optimize images
- Preload critical resources
- Use CDN

**FID (First Input Delay)** - < 100ms
- Minimize JavaScript
- Break up long tasks
- Use web workers

**CLS (Cumulative Layout Shift)** - < 0.1
- Set image dimensions
- Reserve space for ads/embeds
- Avoid inserting content above existing content

### Image Optimization

```tsx
// Next.js Image component
import Image from 'next/image'

// Automatically optimizes
<Image
  src="/hero.jpg"
  alt="Hero"
  width={1200}
  height={600}
  priority // For above-the-fold images
  placeholder="blur"
  blurDataURL={blurPlaceholder}
/>

// Responsive images
<Image
  src="/product.jpg"
  alt="Product"
  sizes="(max-width: 768px) 100vw, 50vw"
  fill
  style={{ objectFit: 'cover' }}
/>
```

### Code Splitting

```tsx
// Dynamic imports
import dynamic from 'next/dynamic'

const HeavyComponent = dynamic(() => import('./HeavyComponent'), {
  loading: () => <Skeleton />,
  ssr: false, // Only load on client
})

// Route-based splitting (automatic in Next.js)
// Each page is its own bundle
```

### Bundle Analysis

```bash
# Next.js
npm install @next/bundle-analyzer

# next.config.js
const withBundleAnalyzer = require('@next/bundle-analyzer')({
  enabled: process.env.ANALYZE === 'true',
})

module.exports = withBundleAnalyzer({})

# Run
ANALYZE=true npm run build
```

### Critical CSS

```tsx
// Inline critical CSS
<head>
  <style dangerouslySetInnerHTML={{
    __html: criticalCSS
  }} />
  <link rel="preload" href="/styles.css" as="style" />
</head>
```

### Resource Hints

```html
<!-- Preconnect to required origins -->
<link rel="preconnect" href="https://api.example.com" />

<!-- Prefetch likely next page -->
<link rel="prefetch" href="/dashboard" />

<!-- Preload critical resources -->
<link rel="preload" href="/fonts/inter.woff2" as="font" crossorigin />
```

---

## Week 2: Backend Performance

### Database Optimization

```typescript
// N+1 Query Problem
// BAD - N+1 queries
const posts = await prisma.post.findMany()
for (const post of posts) {
  const author = await prisma.user.findUnique({ where: { id: post.authorId } })
}

// GOOD - Single query with include
const posts = await prisma.post.findMany({
  include: { author: true }
})

// Selective fields
const users = await prisma.user.findMany({
  select: { id: true, name: true } // Only fetch needed fields
})

// Pagination
const posts = await prisma.post.findMany({
  take: 20,
  skip: 0,
  orderBy: { createdAt: 'desc' }
})
```

### Query Performance

```sql
-- Add indexes for frequently queried columns
CREATE INDEX idx_posts_author_id ON posts(author_id);
CREATE INDEX idx_posts_created_at ON posts(created_at DESC);

-- Composite index for common query patterns
CREATE INDEX idx_posts_author_published ON posts(author_id, published);

-- Analyze query performance
EXPLAIN ANALYZE SELECT * FROM posts WHERE author_id = 1;
```

### Caching Strategies

```typescript
// Response caching
import { Redis } from 'ioredis'

const redis = new Redis(process.env.REDIS_URL)

async function getCachedPosts(page: number) {
  const cacheKey = `posts:page:${page}`

  // Try cache first
  const cached = await redis.get(cacheKey)
  if (cached) return JSON.parse(cached)

  // Fetch from DB
  const posts = await prisma.post.findMany({
    skip: (page - 1) * 20,
    take: 20,
  })

  // Cache for 5 minutes
  await redis.set(cacheKey, JSON.stringify(posts), 'EX', 300)

  return posts
}

// Invalidate cache on write
async function createPost(data: PostData) {
  const post = await prisma.post.create({ data })

  // Invalidate all page caches
  const keys = await redis.keys('posts:page:*')
  if (keys.length) await redis.del(...keys)

  return post
}
```

### API Response Time

```typescript
// Compression
import compression from 'compression'
app.use(compression())

// Response time logging
app.use((req, res, next) => {
  const start = Date.now()
  res.on('finish', () => {
    const duration = Date.now() - start
    console.log(`${req.method} ${req.path} - ${duration}ms`)
  })
  next()
})

// Connection pooling (Prisma handles this)
// Configure in DATABASE_URL:
// postgresql://...?connection_limit=10
```

### Async Operations

```typescript
// Parallel execution
const [users, posts, comments] = await Promise.all([
  getUsers(),
  getPosts(),
  getComments()
])

// Background jobs for heavy operations
import { Queue } from 'bullmq'

const emailQueue = new Queue('emails')

// Don't wait for email to send
await emailQueue.add('welcome', { userId: user.id })
res.json({ success: true })

// Process in background
const worker = new Worker('emails', async (job) => {
  await sendEmail(job.data)
})
```

---

## Performance Monitoring

### Frontend Monitoring

```typescript
// Web Vitals
import { onCLS, onFID, onLCP } from 'web-vitals'

function sendToAnalytics(metric) {
  fetch('/api/analytics', {
    method: 'POST',
    body: JSON.stringify(metric),
  })
}

onCLS(sendToAnalytics)
onFID(sendToAnalytics)
onLCP(sendToAnalytics)
```

### Backend Monitoring

```typescript
// Request timing
const histogram = new promClient.Histogram({
  name: 'http_request_duration_seconds',
  help: 'HTTP request duration',
  labelNames: ['method', 'path', 'status'],
})

app.use((req, res, next) => {
  const end = histogram.startTimer()
  res.on('finish', () => {
    end({ method: req.method, path: req.route?.path, status: res.statusCode })
  })
  next()
})
```

---

## Optimization Checklist

### Frontend
- [ ] Images optimized (WebP, proper sizing)
- [ ] Code split and lazy loaded
- [ ] Critical CSS inlined
- [ ] Fonts preloaded
- [ ] Third-party scripts deferred
- [ ] CDN configured

### Backend
- [ ] Database queries optimized
- [ ] Indexes added for common queries
- [ ] Caching implemented
- [ ] Response compression enabled
- [ ] Connection pooling configured
- [ ] Background jobs for heavy work

---

## Resources

- [web.dev Performance](https://web.dev/performance/)
- [PageSpeed Insights](https://pagespeed.web.dev/)
- [Lighthouse](https://developers.google.com/web/tools/lighthouse)

---

**Next:** [AI Integration](../03-ai-integration/README.md)
