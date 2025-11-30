# Day 3: Docker Compose - Multi-Container Applications

## Introduction

Real applications rarely run in isolation. A typical web app needs a database, cache, perhaps a message queue. Docker Compose lets you define all these services in a single YAML file and manage them together. Today, you'll learn to orchestrate multi-container applications with Docker Compose.

## Learning Objectives

By the end of this lesson, you will be able to:
- Write Docker Compose configuration files
- Define multi-service applications
- Manage container networking and volumes
- Use environment variables and secrets
- Run development and production setups

---

## Docker Compose Basics

### What is Docker Compose?

```
┌─────────────────────────────────────────────────────────────────┐
│                     docker-compose.yml                          │
├─────────────────────────────────────────────────────────────────┤
│  version: "3.8"                                                 │
│                                                                 │
│  services:                                                      │
│    ┌─────────┐    ┌─────────┐    ┌─────────┐                   │
│    │   web   │────│   api   │────│   db    │                   │
│    │  nginx  │    │  node   │    │ postgres│                   │
│    └─────────┘    └─────────┘    └─────────┘                   │
│         │              │              │                         │
│    volumes:       networks:      environment:                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

One command to start everything:
$ docker compose up
```

### Installation

```bash
# Docker Compose V2 (recommended) - included with Docker Desktop
docker compose version

# Or install separately
sudo apt install docker-compose-plugin

# Legacy V1 command (deprecated)
docker-compose up  # Old syntax
docker compose up  # New syntax (no hyphen)
```

---

## Compose File Structure

### Basic Structure

```yaml
# docker-compose.yml

version: "3.8"

services:
  # Service definitions
  web:
    image: nginx
    ports:
      - "80:80"

  api:
    build: ./api
    ports:
      - "3000:3000"

  db:
    image: postgres:14
    environment:
      POSTGRES_PASSWORD: secret

volumes:
  # Named volumes

networks:
  # Custom networks
```

### Simple Example

```yaml
version: "3.8"

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgres://user:pass@db:5432/myapp
    depends_on:
      - db

  db:
    image: postgres:14-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: myapp
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

---

## Service Configuration

### Image vs Build

```yaml
services:
  # Use existing image
  nginx:
    image: nginx:alpine

  # Build from Dockerfile
  app:
    build: .

  # Build with options
  api:
    build:
      context: ./api
      dockerfile: Dockerfile.prod
      args:
        NODE_ENV: production

  # Build and image name
  web:
    build: .
    image: myapp:latest
```

### Ports

```yaml
services:
  web:
    ports:
      # Host:Container
      - "8080:80"

      # Multiple ports
      - "80:80"
      - "443:443"

      # Only container port (random host port)
      - "3000"

      # Bind to specific interface
      - "127.0.0.1:3000:3000"

      # UDP port
      - "5000:5000/udp"
```

### Environment Variables

```yaml
services:
  app:
    environment:
      # Key=Value format
      NODE_ENV: production
      DEBUG: "true"

      # List format
      - NODE_ENV=production
      - DEBUG=true

    # From .env file
    env_file:
      - .env
      - .env.local

    # From host environment
    environment:
      - API_KEY  # Uses $API_KEY from host
```

### Volumes

```yaml
services:
  app:
    volumes:
      # Named volume
      - data:/app/data

      # Bind mount (host path)
      - ./src:/app/src

      # Read-only mount
      - ./config:/app/config:ro

      # Anonymous volume
      - /app/node_modules

volumes:
  data:
    # Use default driver

  postgres_data:
    driver: local
    driver_opts:
      type: none
      device: /data/postgres
      o: bind
```

### Networks

```yaml
services:
  frontend:
    networks:
      - frontend-net

  backend:
    networks:
      - frontend-net
      - backend-net

  database:
    networks:
      - backend-net

networks:
  frontend-net:
  backend-net:
    driver: bridge

# Default: all services on same network
```

### Dependencies

```yaml
services:
  app:
    depends_on:
      - db
      - redis

  # With health check condition
  app:
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started

  db:
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5
```

### Resource Limits

```yaml
services:
  app:
    deploy:
      resources:
        limits:
          cpus: "0.5"
          memory: 512M
        reservations:
          cpus: "0.25"
          memory: 256M
```

### Restart Policy

```yaml
services:
  app:
    restart: always

  # Options:
  # - "no" (default)
  # - always
  # - on-failure
  # - unless-stopped
```

### Command and Entrypoint

```yaml
services:
  app:
    command: npm run dev

  worker:
    command: ["node", "worker.js"]

  custom:
    entrypoint: /app/entrypoint.sh
    command: ["--config", "/app/config.yml"]
```

---

## Docker Compose Commands

### Basic Commands

```bash
# Start services
docker compose up

# Start in background
docker compose up -d

# Start specific service
docker compose up app

# Stop services
docker compose down

# Stop and remove volumes
docker compose down -v

# Stop and remove images
docker compose down --rmi all
```

### Build Commands

```bash
# Build images
docker compose build

# Build without cache
docker compose build --no-cache

# Build specific service
docker compose build app

# Build and start
docker compose up --build
```

### Management Commands

```bash
# List running containers
docker compose ps

# View logs
docker compose logs
docker compose logs -f          # Follow
docker compose logs app         # Specific service
docker compose logs --tail 100  # Last 100 lines

# Execute command in service
docker compose exec app bash
docker compose exec db psql -U postgres

# Run one-off command
docker compose run app npm test
docker compose run --rm app npm install package

# Scale service
docker compose up -d --scale worker=3
```

### Lifecycle Commands

```bash
# Start stopped containers
docker compose start

# Stop running containers
docker compose stop

# Restart containers
docker compose restart

# Pause/unpause
docker compose pause
docker compose unpause

# View resource usage
docker compose top
```

---

## Complete Examples

### Full-Stack Web Application

```yaml
# docker-compose.yml

version: "3.8"

services:
  # Frontend - React/Next.js
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    environment:
      - NEXT_PUBLIC_API_URL=http://localhost:4000
    depends_on:
      - api
    volumes:
      - ./frontend:/app
      - /app/node_modules
    command: npm run dev

  # Backend API - Node.js/Express
  api:
    build: ./api
    ports:
      - "4000:4000"
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgres://user:password@db:5432/myapp
      - REDIS_URL=redis://redis:6379
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started
    volumes:
      - ./api:/app
      - /app/node_modules
    command: npm run dev

  # Database - PostgreSQL
  db:
    image: postgres:14-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: myapp
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./db/init.sql:/docker-entrypoint-initdb.d/init.sql
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user -d myapp"]
      interval: 5s
      timeout: 5s
      retries: 5

  # Cache - Redis
  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data
    command: redis-server --appendonly yes

  # Reverse Proxy - Nginx
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - frontend
      - api

volumes:
  postgres_data:
  redis_data:

networks:
  default:
    name: myapp-network
```

### Development Environment

```yaml
# docker-compose.yml

version: "3.8"

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
    volumes:
      # Mount source code
      - .:/app
      # Preserve node_modules
      - /app/node_modules
    environment:
      - NODE_ENV=development
      - DEBUG=app:*
    command: npm run dev

  db:
    image: postgres:14-alpine
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: dev
      POSTGRES_PASSWORD: dev
      POSTGRES_DB: dev
    volumes:
      - pgdata:/var/lib/postgresql/data

  # Database admin UI
  adminer:
    image: adminer
    ports:
      - "8080:8080"
    depends_on:
      - db

  # Email testing
  mailhog:
    image: mailhog/mailhog
    ports:
      - "1025:1025"
      - "8025:8025"

volumes:
  pgdata:
```

### Production Setup

```yaml
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
          cpus: "1"
          memory: 1G
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  nginx:
    image: nginx:alpine
    restart: always
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/prod.conf:/etc/nginx/nginx.conf:ro
      - ./certs:/etc/nginx/certs:ro
    depends_on:
      - app

  db:
    image: postgres:14-alpine
    restart: always
    environment:
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: ${DB_NAME}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    deploy:
      resources:
        limits:
          cpus: "0.5"
          memory: 512M

volumes:
  postgres_data:
```

---

## Environment Files

### Using .env Files

```bash
# .env
POSTGRES_USER=myuser
POSTGRES_PASSWORD=secret123
POSTGRES_DB=myapp
NODE_ENV=development
```

```yaml
# docker-compose.yml
services:
  db:
    image: postgres:14
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
```

### Multiple Environments

```bash
# .env.development
NODE_ENV=development
DEBUG=true
DATABASE_URL=postgres://dev:dev@db:5432/dev

# .env.production
NODE_ENV=production
DEBUG=false
DATABASE_URL=postgres://prod:secret@db:5432/prod
```

```yaml
services:
  app:
    env_file:
      - .env
      - .env.${ENV:-development}
```

```bash
# Use specific environment
ENV=production docker compose up
```

---

## Multiple Compose Files

### Override Files

```bash
# Base configuration
docker-compose.yml

# Development overrides
docker-compose.override.yml  # Auto-loaded

# Production overrides
docker-compose.prod.yml
```

```yaml
# docker-compose.yml (base)
services:
  app:
    image: myapp
    environment:
      - NODE_ENV=production

# docker-compose.override.yml (dev - auto-loaded)
services:
  app:
    build: .
    volumes:
      - .:/app
    environment:
      - NODE_ENV=development
    command: npm run dev
```

```bash
# Development (uses override automatically)
docker compose up

# Production
docker compose -f docker-compose.yml -f docker-compose.prod.yml up
```

---

## Practice Exercises

### Exercise 1: Basic Setup

```yaml
# Create a compose file with:
# 1. Node.js app on port 3000
# 2. PostgreSQL database
# 3. Named volume for data persistence
# 4. Environment variables
```

### Exercise 2: Full Stack

```yaml
# Create a compose file with:
# 1. React frontend
# 2. Express API
# 3. PostgreSQL database
# 4. Redis cache
# 5. Proper networking
```

### Exercise 3: Dev/Prod Split

```yaml
# Create separate compose files:
# 1. Base configuration
# 2. Development overrides (volumes, ports)
# 3. Production configuration (resources, health checks)
```

---

## Quick Reference

### Common Options

| Key | Description |
|-----|-------------|
| `image` | Image to use |
| `build` | Build configuration |
| `ports` | Port mappings |
| `volumes` | Volume mounts |
| `environment` | Environment variables |
| `env_file` | Env file path |
| `depends_on` | Service dependencies |
| `networks` | Network connections |
| `restart` | Restart policy |
| `command` | Override CMD |
| `healthcheck` | Health check config |

### Commands

| Command | Description |
|---------|-------------|
| `docker compose up` | Start services |
| `docker compose up -d` | Start in background |
| `docker compose down` | Stop and remove |
| `docker compose build` | Build images |
| `docker compose logs` | View logs |
| `docker compose exec` | Execute command |
| `docker compose ps` | List containers |

---

## Key Takeaways

1. **One file, many services** - Define entire stack
2. **Use depends_on** - Control startup order
3. **Named volumes** - Persist data across restarts
4. **env_file** - Keep secrets separate
5. **Override files** - Dev vs production configs
6. **Health checks** - Ensure services are ready

---

## What's Next?

Tomorrow, we'll dive deeper into **Multi-Container Applications** - networking, service discovery, and common patterns for microservices.
