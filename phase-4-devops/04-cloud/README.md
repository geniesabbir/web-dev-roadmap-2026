# Cloud Deployment

**Duration:** 1-2 weeks

## Learning Objectives

By the end of this section, you will:
- Deploy frontend to Vercel
- Deploy backend to Railway/Render
- Set up databases in the cloud
- Configure custom domains and SSL
- Understand AWS basics

---

## Platform Overview

| Platform | Best For | Pricing |
|----------|----------|---------|
| **Vercel** | Next.js, static sites | Free tier available |
| **Railway** | Backend APIs, databases | Free tier, usage-based |
| **Render** | Full-stack apps | Free tier available |
| **AWS** | Enterprise, custom needs | Pay-as-you-go |
| **DigitalOcean** | VPS, Kubernetes | $5+/month |

---

## Day 1: Vercel (Frontend)

### Deploy Next.js
```bash
# Install Vercel CLI
npm i -g vercel

# Login
vercel login

# Deploy (from project root)
vercel

# Deploy to production
vercel --prod
```

### Project Settings
1. Connect GitHub repository
2. Configure build settings:
   - Framework: Next.js
   - Build Command: `npm run build`
   - Output Directory: `.next`
3. Add environment variables
4. Configure custom domain

### Environment Variables
```bash
# Add via CLI
vercel env add DATABASE_URL production

# Or in dashboard:
# Settings > Environment Variables
```

### vercel.json Configuration
```json
{
  "buildCommand": "npm run build",
  "outputDirectory": ".next",
  "framework": "nextjs",
  "regions": ["iad1"],
  "headers": [
    {
      "source": "/api/(.*)",
      "headers": [
        { "key": "Cache-Control", "value": "no-store" }
      ]
    }
  ],
  "rewrites": [
    { "source": "/api/:path*", "destination": "https://api.example.com/:path*" }
  ]
}
```

---

## Day 2: Railway (Backend)

### Deploy via Dashboard
1. Create account at railway.app
2. New Project > Deploy from GitHub
3. Select repository
4. Configure environment variables
5. Deploy

### Deploy via CLI
```bash
# Install Railway CLI
npm i -g @railway/cli

# Login
railway login

# Initialize project
railway init

# Link to existing project
railway link

# Deploy
railway up

# Open dashboard
railway open
```

### Add PostgreSQL
1. New > Database > PostgreSQL
2. Copy connection string
3. Add to service environment variables

### railway.json
```json
{
  "$schema": "https://railway.app/railway.schema.json",
  "build": {
    "builder": "NIXPACKS"
  },
  "deploy": {
    "startCommand": "npm start",
    "healthcheckPath": "/health",
    "healthcheckTimeout": 100,
    "restartPolicyType": "ON_FAILURE",
    "restartPolicyMaxRetries": 10
  }
}
```

---

## Day 3: Render (Alternative)

### Deploy Backend
1. Create account at render.com
2. New > Web Service
3. Connect repository
4. Configure:
   - Environment: Node
   - Build Command: `npm install && npm run build`
   - Start Command: `npm start`
5. Add environment variables
6. Deploy

### Add PostgreSQL
1. New > PostgreSQL
2. Copy Internal Database URL
3. Add to web service environment

### render.yaml (Infrastructure as Code)
```yaml
services:
  - type: web
    name: api
    env: node
    buildCommand: npm install && npm run build
    startCommand: npm start
    healthCheckPath: /health
    envVars:
      - key: NODE_ENV
        value: production
      - key: DATABASE_URL
        fromDatabase:
          name: mydb
          property: connectionString

databases:
  - name: mydb
    plan: free
```

---

## Day 4: Domain & SSL Setup

### Custom Domain on Vercel
1. Go to Project Settings > Domains
2. Add domain (e.g., myapp.com)
3. Configure DNS:
   ```
   Type: A
   Name: @
   Value: 76.76.21.21

   Type: CNAME
   Name: www
   Value: cname.vercel-dns.com
   ```

### Custom Domain on Railway
1. Settings > Domains
2. Add custom domain
3. Configure DNS:
   ```
   Type: CNAME
   Name: api
   Value: your-app.up.railway.app
   ```

### SSL Certificates
- Vercel: Automatic via Let's Encrypt
- Railway: Automatic
- Render: Automatic
- Manual: Use Certbot or Cloudflare

---

## Day 5: AWS Basics

### Key Services
- **EC2**: Virtual servers
- **S3**: Object storage (files, images)
- **RDS**: Managed databases
- **Lambda**: Serverless functions
- **CloudFront**: CDN

### S3 for File Uploads
```typescript
// Install SDK
// npm install @aws-sdk/client-s3

import { S3Client, PutObjectCommand } from '@aws-sdk/client-s3'

const s3 = new S3Client({
  region: process.env.AWS_REGION,
  credentials: {
    accessKeyId: process.env.AWS_ACCESS_KEY_ID!,
    secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY!,
  },
})

async function uploadFile(buffer: Buffer, key: string, contentType: string) {
  const command = new PutObjectCommand({
    Bucket: process.env.S3_BUCKET_NAME,
    Key: key,
    Body: buffer,
    ContentType: contentType,
  })

  await s3.send(command)

  return `https://${process.env.S3_BUCKET_NAME}.s3.amazonaws.com/${key}`
}
```

### Environment Variable Management
```bash
# Development
cp .env.example .env.local

# Production (set in platform dashboard)
DATABASE_URL=postgresql://...
REDIS_URL=redis://...
AWS_ACCESS_KEY_ID=...
AWS_SECRET_ACCESS_KEY=...
S3_BUCKET_NAME=...
```

---

## Production Checklist

### Before Deploying
- [ ] Environment variables set
- [ ] Database migrations ready
- [ ] Build succeeds locally
- [ ] Tests pass
- [ ] Security headers configured
- [ ] CORS configured correctly
- [ ] Rate limiting enabled

### After Deploying
- [ ] Health check endpoint works
- [ ] All features functional
- [ ] SSL certificate active
- [ ] Monitoring set up
- [ ] Error tracking enabled (Sentry)
- [ ] Backup strategy in place

### Monitoring
```typescript
// Health check endpoint
app.get('/health', (req, res) => {
  res.json({
    status: 'healthy',
    timestamp: new Date().toISOString(),
    uptime: process.uptime(),
  })
})
```

### Error Tracking with Sentry
```bash
npm install @sentry/node
```

```typescript
import * as Sentry from '@sentry/node'

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  environment: process.env.NODE_ENV,
  tracesSampleRate: 1.0,
})

// Error handler middleware
app.use(Sentry.Handlers.errorHandler())
```

---

## Resources

- [Vercel Documentation](https://vercel.com/docs)
- [Railway Documentation](https://docs.railway.app/)
- [Render Documentation](https://render.com/docs)
- [AWS Getting Started](https://aws.amazon.com/getting-started/)

---

## Checklist Before Moving On

- [ ] Deployed frontend to Vercel
- [ ] Deployed backend to Railway/Render
- [ ] Database set up in cloud
- [ ] Custom domain configured
- [ ] SSL working
- [ ] CI/CD pipeline deploying automatically
- [ ] Basic monitoring in place

---

**Phase 4 Complete!**

You now have:
- Applications deployed to production
- CI/CD pipelines automating deployments
- Understanding of cloud platforms

**Next:** [Phase 5: Advanced Topics](../../phase-5-advanced/README.md)
