# Day 5: Docker Best Practices - Security, Optimization, and Production

## Introduction

Building Docker images that work is just the first step. Production-ready containers need to be secure, efficient, and maintainable. Today, you'll learn industry best practices for creating Docker containers that are ready for real-world deployment.

## Learning Objectives

By the end of this lesson, you will be able to:
- Implement security best practices
- Optimize images for size and performance
- Structure projects for maintainability
- Handle secrets and sensitive data
- Create production-ready configurations

---

## Security Best Practices

### Run as Non-Root User

```dockerfile
# BAD: Running as root (default)
FROM node:18
WORKDIR /app
COPY . .
RUN npm install
CMD ["node", "server.js"]
# Container runs as root - security risk!

# GOOD: Create and use non-root user
FROM node:18-alpine

# Create non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

WORKDIR /app

# Change ownership of work directory
COPY --chown=appuser:appgroup . .

RUN npm ci --only=production

# Switch to non-root user
USER appuser

CMD ["node", "server.js"]
```

### Use Official Base Images

```dockerfile
# GOOD: Official images are regularly updated
FROM node:18-alpine
FROM python:3.11-slim
FROM nginx:alpine

# BAD: Random images from unknown publishers
FROM random-user/custom-node  # Avoid!

# Verify image source
docker trust inspect nginx
```

### Minimize Attack Surface

```dockerfile
# Use minimal base images
FROM node:18-alpine  # ~120MB vs ~900MB for node:18

# Remove unnecessary packages
RUN apk add --no-cache curl \
    && rm -rf /var/cache/apk/*

# Don't include build tools in production
FROM node:18-alpine AS builder
RUN npm ci && npm run build

FROM node:18-alpine AS production
# Only copy built artifacts
COPY --from=builder /app/dist ./dist
```

### Scan for Vulnerabilities

```bash
# Scan image for vulnerabilities
docker scout cves myapp:latest

# Using Trivy
trivy image myapp:latest

# Using Snyk
snyk container test myapp:latest

# Integrate in CI/CD
docker build -t myapp . && trivy image --exit-code 1 myapp
```

### Read-Only Filesystem

```yaml
# docker-compose.yml
services:
  app:
    image: myapp
    read_only: true
    tmpfs:
      - /tmp
      - /var/run
    volumes:
      - logs:/app/logs  # Only writable location
```

### Security Options

```yaml
services:
  app:
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE  # Only if needed
```

---

## Image Optimization

### Minimize Image Size

```dockerfile
# 1. Use Alpine base images
FROM node:18-alpine  # vs node:18

# 2. Multi-stage builds
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:18-alpine AS production
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
CMD ["node", "dist/server.js"]

# 3. Clean up in same layer
RUN apt-get update && apt-get install -y \
    package1 \
    package2 \
    && rm -rf /var/lib/apt/lists/* \
    && apt-get clean

# 4. Use .dockerignore
# node_modules
# .git
# *.md
# test/
```

### Layer Optimization

```dockerfile
# Order by change frequency (least → most)
FROM node:18-alpine

WORKDIR /app

# 1. System dependencies (rarely change)
RUN apk add --no-cache tini

# 2. Package files (change sometimes)
COPY package*.json ./
RUN npm ci --only=production

# 3. Source code (changes often)
COPY . .

# Entry point
ENTRYPOINT ["/sbin/tini", "--"]
CMD ["node", "server.js"]
```

### Reduce Build Context

```bash
# .dockerignore
node_modules
npm-debug.log
.git
.gitignore
.env
.env.*
*.md
README.md
docs/
test/
coverage/
.nyc_output/
.docker/
Dockerfile*
docker-compose*
.dockerignore
.vscode/
.idea/
*.log
*.tmp
```

### Image Size Comparison

```bash
# Check image sizes
docker images

# Analyze layers
docker history myapp --no-trunc

# Use dive for detailed analysis
dive myapp:latest

# Typical size reduction:
# node:18       ~900MB
# node:18-slim  ~200MB
# node:18-alpine ~120MB
# distroless    ~100MB
```

---

## Handling Secrets

### Environment Variables (Basic)

```yaml
# docker-compose.yml
services:
  app:
    environment:
      - DATABASE_URL  # From host environment
    env_file:
      - .env.production
```

### Docker Secrets (Swarm)

```bash
# Create secret
echo "my-secret-password" | docker secret create db_password -

# Use in compose
services:
  db:
    secrets:
      - db_password
    environment:
      - POSTGRES_PASSWORD_FILE=/run/secrets/db_password

secrets:
  db_password:
    external: true
```

### Build-Time Secrets

```dockerfile
# Dockerfile
# syntax=docker/dockerfile:1.4

FROM node:18-alpine

# Mount secret during build (not stored in layer)
RUN --mount=type=secret,id=npm_token \
    NPM_TOKEN=$(cat /run/secrets/npm_token) \
    npm install

# Build with:
# docker build --secret id=npm_token,src=.npmrc .
```

### Never Do This

```dockerfile
# BAD: Secrets in environment or copy
ENV API_KEY=secret123
COPY .env /app/.env
ARG SECRET_KEY
RUN echo $SECRET_KEY > /app/secret

# These are visible in image history!
docker history myapp
```

---

## Production Configuration

### Health Checks

```dockerfile
FROM node:18-alpine

WORKDIR /app
COPY . .
RUN npm ci --only=production

EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD wget -q --spider http://localhost:3000/health || exit 1

CMD ["node", "server.js"]
```

```javascript
// server.js - Health endpoint
app.get('/health', (req, res) => {
  // Check dependencies
  const dbHealthy = checkDatabase();
  const cacheHealthy = checkRedis();

  if (dbHealthy && cacheHealthy) {
    res.status(200).json({ status: 'healthy' });
  } else {
    res.status(503).json({ status: 'unhealthy' });
  }
});
```

### Graceful Shutdown

```javascript
// server.js
const server = app.listen(PORT);

// Handle shutdown signals
process.on('SIGTERM', gracefulShutdown);
process.on('SIGINT', gracefulShutdown);

async function gracefulShutdown(signal) {
  console.log(`Received ${signal}, shutting down gracefully`);

  // Stop accepting new requests
  server.close(() => {
    console.log('HTTP server closed');
  });

  // Close database connections
  await db.close();
  await redis.quit();

  // Exit
  process.exit(0);
}
```

```dockerfile
# Use tini for proper signal handling
FROM node:18-alpine

RUN apk add --no-cache tini

ENTRYPOINT ["/sbin/tini", "--"]
CMD ["node", "server.js"]
```

### Resource Limits

```yaml
# docker-compose.yml
services:
  app:
    deploy:
      resources:
        limits:
          cpus: '1'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 128M

    # Restart policy
    restart: unless-stopped

    # Logging
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"
```

### Production Compose File

```yaml
# docker-compose.prod.yml
version: "3.8"

services:
  app:
    image: myapp:${VERSION:-latest}
    restart: always
    read_only: true
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
    tmpfs:
      - /tmp
    healthcheck:
      test: ["CMD", "wget", "-q", "--spider", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    deploy:
      resources:
        limits:
          cpus: '1'
          memory: 512M
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "5"
    environment:
      - NODE_ENV=production
    env_file:
      - .env.production

  nginx:
    image: nginx:alpine
    restart: always
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./certs:/etc/nginx/certs:ro
    depends_on:
      app:
        condition: service_healthy
```

---

## Logging Best Practices

### Log to STDOUT/STDERR

```javascript
// Log to stdout (Docker captures automatically)
console.log('Application started');
console.error('Error occurred');

// Or use structured logging
const pino = require('pino');
const logger = pino();

logger.info({ port: 3000 }, 'Server started');
logger.error({ err: error }, 'Request failed');
```

```dockerfile
# Don't write to files inside container
# BAD:
RUN mkdir /app/logs
# App writes to /app/logs/app.log

# GOOD:
# App writes to stdout/stderr
# Docker handles log collection
```

### Logging Configuration

```yaml
services:
  app:
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "5"
        labels: "app,environment"
        env: "NODE_ENV"

# View logs
# docker compose logs -f app
```

---

## Development vs Production

### Dockerfile Comparison

```dockerfile
# Dockerfile.dev
FROM node:18

WORKDIR /app

# Install all dependencies
COPY package*.json ./
RUN npm install

# Source mounted via volume
# CMD in docker-compose

# Dockerfile.prod
FROM node:18-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build && npm prune --production

FROM node:18-alpine

RUN apk add --no-cache tini
RUN addgroup -S app && adduser -S app -G app

WORKDIR /app

COPY --from=builder --chown=app:app /app/dist ./dist
COPY --from=builder --chown=app:app /app/node_modules ./node_modules
COPY --from=builder --chown=app:app /app/package.json ./

USER app

HEALTHCHECK --interval=30s CMD wget -q --spider http://localhost:3000/health || exit 1

ENTRYPOINT ["/sbin/tini", "--"]
CMD ["node", "dist/server.js"]
```

### Compose Comparison

```yaml
# docker-compose.yml (development)
version: "3.8"

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
    volumes:
      - .:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
      - DEBUG=app:*
    command: npm run dev

# docker-compose.prod.yml
version: "3.8"

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile.prod
    restart: always
    environment:
      - NODE_ENV=production
    env_file:
      - .env.production
    deploy:
      resources:
        limits:
          cpus: '1'
          memory: 512M
    healthcheck:
      test: wget -q --spider http://localhost:3000/health
      interval: 30s
```

---

## CI/CD Integration

### GitHub Actions Example

```yaml
# .github/workflows/docker.yml
name: Docker Build

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and test
        run: |
          docker build -t myapp:test .
          docker run --rm myapp:test npm test

      - name: Scan for vulnerabilities
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: myapp:test
          exit-code: 1
          severity: HIGH,CRITICAL

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: |
            ghcr.io/${{ github.repository }}:latest
            ghcr.io/${{ github.repository }}:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

---

## Checklist: Production-Ready Container

```markdown
## Security
- [ ] Non-root user
- [ ] Official base image
- [ ] Minimal base (Alpine/distroless)
- [ ] No secrets in image
- [ ] Read-only filesystem
- [ ] Dropped capabilities
- [ ] Vulnerability scan passed

## Optimization
- [ ] Multi-stage build
- [ ] Proper .dockerignore
- [ ] Layer caching optimized
- [ ] Image size minimized

## Reliability
- [ ] Health check defined
- [ ] Graceful shutdown handling
- [ ] Resource limits set
- [ ] Restart policy configured

## Observability
- [ ] Logs to stdout/stderr
- [ ] Structured logging
- [ ] Log rotation configured

## Maintainability
- [ ] Tagged with version
- [ ] Labels added
- [ ] Documentation updated
```

---

## Quick Reference

### Security Checklist

| Practice | Implementation |
|----------|----------------|
| Non-root user | `USER appuser` |
| Read-only | `read_only: true` |
| Drop capabilities | `cap_drop: ALL` |
| No privileges | `no-new-privileges:true` |
| Scan images | `trivy image myapp` |

### Optimization Checklist

| Practice | Implementation |
|----------|----------------|
| Minimal base | `FROM node:18-alpine` |
| Multi-stage | `FROM ... AS builder` |
| Layer caching | Order by change frequency |
| .dockerignore | Exclude node_modules, .git |
| Clean up | `rm -rf /var/cache/*` |

---

## Key Takeaways

1. **Never run as root** - Create non-root user
2. **Use minimal images** - Alpine or distroless
3. **Multi-stage builds** - Separate build from production
4. **Handle secrets properly** - Never in image layers
5. **Add health checks** - Enable orchestration
6. **Graceful shutdown** - Handle SIGTERM
7. **Set resource limits** - Prevent resource exhaustion
8. **Scan for vulnerabilities** - Integrate in CI/CD

---

## Congratulations!

You've completed the Docker section! You now know how to:
- Build and run containers
- Write optimized Dockerfiles
- Compose multi-container applications
- Network containers together
- Apply security and production best practices

Next up is **CI/CD** - automating builds, tests, and deployments with GitHub Actions!
