# Day 2: Railway - Full-Stack Application Deployment

## Introduction

Railway is a modern platform for deploying full-stack applications. Unlike Vercel which focuses on frontend, Railway excels at running backends, databases, and workers. It offers instant deployments, built-in databases, and a developer-friendly experience. Today, you'll learn to deploy complete applications with databases on Railway.

## Learning Objectives

By the end of this lesson, you will be able to:
- Deploy backend services to Railway
- Provision and connect databases
- Configure environment variables
- Set up background workers and cron jobs
- Manage multiple services in a project

---

## Getting Started with Railway

### Creating an Account

```bash
# 1. Sign up at railway.app
# 2. Connect your GitHub account

# Install Railway CLI
npm install -g @railway/cli

# Login
railway login

# Check status
railway whoami
```

### Deploying Your First App

```bash
# Initialize new project
railway init

# Or link existing project
railway link

# Deploy
railway up

# Your app is now live!
```

### Git Integration

```bash
# Connect GitHub repo in dashboard
# Settings > Connect Repo

# Automatic deployments on push
git push origin main

# Railway builds and deploys automatically
```

---

## Project Structure

### Services Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                      Railway Project                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐             │
│  │   Web API   │  │   Worker    │  │   Cron Job  │             │
│  │   (Node)    │  │   (Node)    │  │   (Node)    │             │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘             │
│         │                │                │                      │
│         └────────────────┼────────────────┘                      │
│                          │                                       │
│  ┌───────────────────────┼───────────────────────┐              │
│  │                       │                       │              │
│  │  ┌─────────────┐  ┌───┴───────┐  ┌─────────┐ │              │
│  │  │  PostgreSQL │  │   Redis   │  │  MinIO  │ │              │
│  │  │  (database) │  │  (cache)  │  │ (files) │ │              │
│  │  └─────────────┘  └───────────┘  └─────────┘ │              │
│  │                                               │              │
│  │              Shared Services                  │              │
│  └───────────────────────────────────────────────┘              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Creating a Project

```bash
# Via CLI
railway init

# Select options:
# ? Starting point: Empty Project
# ? Project name: my-fullstack-app

# Via dashboard
# 1. New Project
# 2. Deploy from GitHub repo
# 3. Or start with template
```

---

## Deploying Services

### Node.js Backend

```javascript
// server.js
const express = require('express');
const app = express();

const PORT = process.env.PORT || 3000;

app.get('/', (req, res) => {
  res.json({ status: 'healthy' });
});

app.get('/api/users', async (req, res) => {
  // Database query
  const users = await db.query('SELECT * FROM users');
  res.json(users);
});

app.listen(PORT, '0.0.0.0', () => {
  console.log(`Server running on port ${PORT}`);
});
```

```json
// package.json
{
  "name": "api",
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js"
  },
  "engines": {
    "node": "18"
  }
}
```

### Dockerfile Deployment

```dockerfile
# Dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY . .

EXPOSE 3000

CMD ["npm", "start"]
```

### railway.json Configuration

```json
{
  "$schema": "https://railway.app/railway.schema.json",
  "build": {
    "builder": "NIXPACKS",
    "buildCommand": "npm run build"
  },
  "deploy": {
    "startCommand": "npm start",
    "healthcheckPath": "/health",
    "healthcheckTimeout": 300,
    "restartPolicyType": "ON_FAILURE",
    "restartPolicyMaxRetries": 3
  }
}
```

---

## Database Services

### PostgreSQL

```bash
# Add PostgreSQL in dashboard
# + New > Database > PostgreSQL

# Or via CLI
railway add --database postgres

# Connection string is auto-generated
# DATABASE_URL=postgresql://user:pass@host:5432/railway
```

```javascript
// Using with Prisma
// schema.prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// Migrate
npx prisma migrate deploy
```

### MySQL

```bash
# Add MySQL
railway add --database mysql

# MYSQL_URL=mysql://user:pass@host:3306/railway
```

### MongoDB

```bash
# Add MongoDB
railway add --database mongo

# MONGO_URL=mongodb://user:pass@host:27017/railway
```

### Redis

```bash
# Add Redis
railway add --database redis

# REDIS_URL=redis://default:pass@host:6379
```

```javascript
// Using Redis
const Redis = require('ioredis');
const redis = new Redis(process.env.REDIS_URL);

await redis.set('key', 'value');
const value = await redis.get('key');
```

---

## Environment Variables

### Setting Variables

```bash
# Via CLI
railway variables set DATABASE_URL="postgres://..."
railway variables set API_KEY="secret123"

# List variables
railway variables

# Delete variable
railway variables delete API_KEY
```

### Shared Variables

```bash
# In dashboard:
# 1. Go to project settings
# 2. Shared Variables tab
# 3. Add variables available to all services
```

### Reference Variables

```bash
# Reference other service variables
DATABASE_URL=${{Postgres.DATABASE_URL}}
REDIS_URL=${{Redis.REDIS_URL}}

# Reference within same service
API_URL=https://${{RAILWAY_STATIC_URL}}
```

### Variable Types

```
┌─────────────────────────────────────────────────────────────────┐
│                    Railway Variables                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Service Variables    → Specific to one service                 │
│  Shared Variables     → Available to all services               │
│  Reference Variables  → ${{ServiceName.VAR_NAME}}               │
│                                                                  │
│  Built-in Variables:                                            │
│  • RAILWAY_ENVIRONMENT   → production/staging                   │
│  • RAILWAY_STATIC_URL    → Public URL                           │
│  • RAILWAY_PRIVATE_DOMAIN → Internal URL                        │
│  • PORT                  → Assigned port                        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Networking

### Public Domains

```bash
# Generate domain
# Settings > Networking > Generate Domain

# Custom domain
# Settings > Networking > Custom Domain
# Add: api.example.com

# Configure DNS:
# CNAME: api → your-app.up.railway.app
```

### Private Networking

```bash
# Internal service communication
# Use RAILWAY_PRIVATE_DOMAIN

# Service A calling Service B
const response = await fetch(
  `http://${process.env.SERVICE_B_PRIVATE_DOMAIN}/api/data`
);

# No public exposure, lower latency
```

### TCP Proxy

```bash
# For non-HTTP services (databases, custom protocols)
# Settings > Networking > TCP Proxy

# Get external host:port for direct connections
# Useful for database clients
```

---

## Background Workers

### Worker Service

```javascript
// worker.js
const Queue = require('bull');

const emailQueue = new Queue('email', process.env.REDIS_URL);

emailQueue.process(async (job) => {
  const { to, subject, body } = job.data;
  await sendEmail(to, subject, body);
  console.log(`Email sent to ${to}`);
});

console.log('Worker started');
```

```json
// railway.json (for worker service)
{
  "deploy": {
    "startCommand": "node worker.js"
  }
}
```

### Cron Jobs

```javascript
// cron.js
const cron = require('node-cron');

// Run every hour
cron.schedule('0 * * * *', async () => {
  console.log('Running hourly task');
  await cleanupOldData();
});

// Run daily at midnight
cron.schedule('0 0 * * *', async () => {
  console.log('Running daily report');
  await generateDailyReport();
});

console.log('Cron jobs scheduled');
```

### Railway Cron Service

```json
// railway.json
{
  "deploy": {
    "startCommand": "node cron-task.js",
    "cronSchedule": "0 * * * *"
  }
}
```

---

## Multi-Service Setup

### Monorepo Deployment

```
my-app/
├── apps/
│   ├── api/
│   │   ├── package.json
│   │   └── railway.json
│   ├── worker/
│   │   ├── package.json
│   │   └── railway.json
│   └── web/
│       └── package.json
├── packages/
│   └── shared/
└── package.json
```

```json
// apps/api/railway.json
{
  "build": {
    "builder": "NIXPACKS",
    "watchPatterns": ["apps/api/**", "packages/shared/**"]
  },
  "deploy": {
    "startCommand": "npm start --workspace=apps/api"
  }
}
```

### Service Communication

```javascript
// API service
const express = require('express');
const app = express();

app.post('/api/send-email', async (req, res) => {
  // Add job to queue (worker picks it up)
  await emailQueue.add({
    to: req.body.email,
    subject: 'Welcome!',
    body: 'Thanks for signing up'
  });

  res.json({ status: 'queued' });
});
```

---

## Deployment Configuration

### Build Settings

```json
// railway.json
{
  "build": {
    "builder": "NIXPACKS",
    "buildCommand": "npm run build",
    "watchPatterns": ["src/**", "package.json"]
  }
}
```

### Health Checks

```json
{
  "deploy": {
    "healthcheckPath": "/health",
    "healthcheckTimeout": 300
  }
}
```

```javascript
// Health endpoint
app.get('/health', async (req, res) => {
  try {
    // Check database
    await db.query('SELECT 1');

    // Check Redis
    await redis.ping();

    res.json({ status: 'healthy' });
  } catch (error) {
    res.status(503).json({ status: 'unhealthy', error: error.message });
  }
});
```

### Replicas

```bash
# Scale service (Pro plan)
# Settings > Deploy > Replicas

# Or via CLI
railway service update --replicas 3
```

---

## Complete Example

### Full-Stack Application

```yaml
# Project structure
project/
├── api/
│   ├── src/
│   │   └── index.ts
│   ├── package.json
│   └── railway.json
├── worker/
│   ├── src/
│   │   └── index.ts
│   ├── package.json
│   └── railway.json
└── README.md
```

```typescript
// api/src/index.ts
import express from 'express';
import { PrismaClient } from '@prisma/client';
import Redis from 'ioredis';
import Queue from 'bull';

const app = express();
const prisma = new PrismaClient();
const redis = new Redis(process.env.REDIS_URL!);
const emailQueue = new Queue('email', process.env.REDIS_URL!);

app.use(express.json());

// Health check
app.get('/health', async (req, res) => {
  await prisma.$queryRaw`SELECT 1`;
  await redis.ping();
  res.json({ status: 'healthy' });
});

// API routes
app.get('/api/users', async (req, res) => {
  const users = await prisma.user.findMany();
  res.json(users);
});

app.post('/api/users', async (req, res) => {
  const user = await prisma.user.create({ data: req.body });

  // Queue welcome email
  await emailQueue.add({ userId: user.id, type: 'welcome' });

  res.status(201).json(user);
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, '0.0.0.0', () => {
  console.log(`API running on port ${PORT}`);
});
```

```typescript
// worker/src/index.ts
import Queue from 'bull';
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();
const emailQueue = new Queue('email', process.env.REDIS_URL!);

emailQueue.process(async (job) => {
  const { userId, type } = job.data;

  const user = await prisma.user.findUnique({ where: { id: userId } });
  if (!user) return;

  if (type === 'welcome') {
    await sendWelcomeEmail(user.email);
  }

  console.log(`Processed ${type} email for user ${userId}`);
});

console.log('Worker started');
```

---

## Railway CLI Commands

### Common Commands

```bash
# Project management
railway init           # Initialize new project
railway link           # Link to existing project
railway up             # Deploy

# Services
railway add            # Add service
railway service        # List services
railway logs           # View logs
railway logs -f        # Follow logs

# Variables
railway variables      # List variables
railway variables set  # Set variable

# Database
railway connect        # Connect to database CLI

# Development
railway run            # Run command with env vars
railway shell          # Shell with env vars
```

---

## Practice Exercises

### Exercise 1: Deploy API with Database

```bash
# 1. Create Express API
# 2. Add PostgreSQL service
# 3. Connect with Prisma
# 4. Deploy and test
```

### Exercise 2: Background Worker

```bash
# 1. Add Redis service
# 2. Create worker service
# 3. Queue jobs from API
# 4. Process in worker
```

### Exercise 3: Multi-Service App

```bash
# 1. API service
# 2. Worker service
# 3. PostgreSQL database
# 4. Redis cache
# 5. Configure networking
```

---

## Quick Reference

### CLI Commands

| Command | Description |
|---------|-------------|
| `railway init` | Initialize project |
| `railway up` | Deploy |
| `railway logs` | View logs |
| `railway variables` | Manage env vars |
| `railway connect` | Database CLI |
| `railway run` | Run with env vars |

### Built-in Variables

| Variable | Description |
|----------|-------------|
| `PORT` | Assigned port |
| `RAILWAY_ENVIRONMENT` | Environment name |
| `RAILWAY_STATIC_URL` | Public URL |
| `RAILWAY_PRIVATE_DOMAIN` | Internal URL |

---

## Key Takeaways

1. **Full-stack ready** - Backend + databases + workers
2. **One-click databases** - PostgreSQL, MySQL, Redis, MongoDB
3. **Private networking** - Internal service communication
4. **Easy scaling** - Replicas and auto-scaling
5. **Git integration** - Push to deploy
6. **CLI power** - Manage everything from terminal

---

## What's Next?

Tomorrow, we'll learn about **Domains and SSL** - configuring custom domains, SSL certificates, and DNS management.
