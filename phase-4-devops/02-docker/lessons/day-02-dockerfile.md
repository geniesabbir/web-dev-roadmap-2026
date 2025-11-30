# Day 2: Dockerfile - Building Custom Images

## Introduction

A Dockerfile is a text file containing instructions to build a Docker image. It defines everything your application needs: the base OS, dependencies, code, configuration, and how to run the app. Today, you'll learn to write efficient Dockerfiles to containerize any application.

## Learning Objectives

By the end of this lesson, you will be able to:
- Write Dockerfiles from scratch
- Understand Dockerfile instructions
- Build and tag Docker images
- Optimize images for size and speed
- Create images for Node.js applications

---

## Dockerfile Basics

### Structure

```dockerfile
# Comment
INSTRUCTION arguments

# Example
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

### Building an Image

```bash
# Build from current directory
docker build -t myapp .

# Build with specific Dockerfile
docker build -f Dockerfile.prod -t myapp:prod .

# Build with build arguments
docker build --build-arg NODE_ENV=production -t myapp .

# Build without cache
docker build --no-cache -t myapp .

# View build output
docker build -t myapp . --progress=plain
```

---

## Dockerfile Instructions

### FROM - Base Image

```dockerfile
# Start from official image
FROM node:18

# Use specific version
FROM node:18.17.0

# Use Alpine (smaller)
FROM node:18-alpine

# Use slim variant
FROM node:18-slim

# Multi-stage: name the stage
FROM node:18 AS builder

# Start from scratch (empty)
FROM scratch
```

### WORKDIR - Working Directory

```dockerfile
# Set working directory
WORKDIR /app

# Creates directory if it doesn't exist
WORKDIR /usr/src/app

# Can use multiple times
WORKDIR /app
WORKDIR src
# Now in /app/src
```

### COPY - Copy Files

```dockerfile
# Copy single file
COPY package.json .

# Copy multiple files
COPY package.json package-lock.json ./

# Copy directory
COPY src/ ./src/

# Copy all files
COPY . .

# Copy with new name
COPY config.json /app/settings.json

# Copy from build stage
COPY --from=builder /app/dist ./dist

# Set ownership
COPY --chown=node:node . .
```

### ADD - Add Files (with extras)

```dockerfile
# Similar to COPY but can:
# 1. Extract tar files
ADD archive.tar.gz /app/

# 2. Download from URL (avoid - use curl instead)
ADD https://example.com/file.txt /app/
```

### RUN - Execute Commands

```dockerfile
# Shell form
RUN npm install

# Exec form (preferred)
RUN ["npm", "install"]

# Multiple commands
RUN apt-get update && apt-get install -y \
    curl \
    git \
    && rm -rf /var/lib/apt/lists/*

# Each RUN creates a layer - combine when possible
RUN npm install \
    && npm run build \
    && npm prune --production
```

### ENV - Environment Variables

```dockerfile
# Set single variable
ENV NODE_ENV=production

# Set multiple variables
ENV NODE_ENV=production \
    PORT=3000 \
    API_URL=https://api.example.com

# Use in subsequent instructions
ENV APP_HOME=/app
WORKDIR $APP_HOME
```

### ARG - Build Arguments

```dockerfile
# Define build argument
ARG NODE_VERSION=18

# Use in FROM
ARG NODE_VERSION=18
FROM node:${NODE_VERSION}-alpine

# Use in instructions
ARG BUILD_DATE
LABEL build-date=$BUILD_DATE

# Build with:
# docker build --build-arg NODE_VERSION=20 -t myapp .
```

### EXPOSE - Document Ports

```dockerfile
# Document port (doesn't publish)
EXPOSE 3000

# Multiple ports
EXPOSE 80 443

# With protocol
EXPOSE 3000/tcp
EXPOSE 5432/udp
```

### CMD - Default Command

```dockerfile
# Exec form (preferred)
CMD ["node", "server.js"]

# Shell form
CMD node server.js

# With arguments
CMD ["npm", "start"]

# Can be overridden at runtime:
# docker run myapp node other-script.js
```

### ENTRYPOINT - Fixed Command

```dockerfile
# Set entrypoint
ENTRYPOINT ["node"]

# CMD becomes arguments to ENTRYPOINT
ENTRYPOINT ["node"]
CMD ["server.js"]
# Runs: node server.js

# Override with --entrypoint:
# docker run --entrypoint npm myapp install
```

### USER - Set User

```dockerfile
# Run as specific user
USER node

# Create and use user
RUN useradd -m appuser
USER appuser

# Use UID
USER 1000
```

### LABEL - Metadata

```dockerfile
LABEL maintainer="developer@example.com"
LABEL version="1.0"
LABEL description="My application"

# Multiple labels
LABEL maintainer="dev@example.com" \
      version="1.0" \
      description="My app"
```

### HEALTHCHECK - Container Health

```dockerfile
# HTTP health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:3000/health || exit 1

# Simple check
HEALTHCHECK CMD wget -q --spider http://localhost:3000 || exit 1

# Disable health check
HEALTHCHECK NONE
```

---

## Node.js Dockerfile Examples

### Basic Node.js App

```dockerfile
FROM node:18-alpine

WORKDIR /app

# Copy package files first (better caching)
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy source code
COPY . .

# Set environment
ENV NODE_ENV=production
ENV PORT=3000

# Expose port
EXPOSE 3000

# Run as non-root user
USER node

# Start application
CMD ["node", "server.js"]
```

### Development Dockerfile

```dockerfile
FROM node:18-alpine

WORKDIR /app

# Install dependencies including devDependencies
COPY package*.json ./
RUN npm install

# Copy source
COPY . .

# Expose port
EXPOSE 3000

# Development command with hot reload
CMD ["npm", "run", "dev"]
```

### Production Multi-Stage Build

```dockerfile
# Build stage
FROM node:18-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

# Production stage
FROM node:18-alpine AS production

WORKDIR /app

# Copy package files
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force

# Copy built files from builder
COPY --from=builder /app/dist ./dist

# Security: non-root user
USER node

ENV NODE_ENV=production
EXPOSE 3000

CMD ["node", "dist/server.js"]
```

### Next.js Dockerfile

```dockerfile
# Dependencies stage
FROM node:18-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci

# Builder stage
FROM node:18-alpine AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build

# Runner stage
FROM node:18-alpine AS runner
WORKDIR /app

ENV NODE_ENV=production

# Copy necessary files
COPY --from=builder /app/public ./public
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static

USER node
EXPOSE 3000
ENV PORT=3000

CMD ["node", "server.js"]
```

---

## .dockerignore File

### Purpose

```bash
# .dockerignore - exclude files from build context

# Dependencies
node_modules
npm-debug.log

# Build output
dist
build
.next

# Development files
.git
.gitignore
.env
.env.*
*.md
README.md

# IDE
.vscode
.idea
*.swp

# Test files
coverage
*.test.js
*.spec.js
__tests__

# Docker files
Dockerfile*
docker-compose*
.dockerignore

# OS files
.DS_Store
Thumbs.db
```

### Benefits

- Faster builds (smaller context)
- Smaller images (no unnecessary files)
- Security (no secrets in image)
- Better cache utilization

---

## Build Optimization

### Layer Caching

```dockerfile
# BAD - Cache invalidated on any code change
FROM node:18-alpine
WORKDIR /app
COPY . .
RUN npm install

# GOOD - Dependencies cached separately
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
```

### Minimize Layers

```dockerfile
# BAD - Many layers
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y git
RUN rm -rf /var/lib/apt/lists/*

# GOOD - Single layer
RUN apt-get update && apt-get install -y \
    curl \
    git \
    && rm -rf /var/lib/apt/lists/*
```

### Use Alpine Images

```dockerfile
# Standard Node image: ~900MB
FROM node:18

# Alpine Node image: ~120MB
FROM node:18-alpine

# Install Alpine packages if needed
RUN apk add --no-cache curl git
```

### Multi-Stage Builds

```dockerfile
# Build stage (with dev dependencies)
FROM node:18-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage (minimal)
FROM node:18-alpine
WORKDIR /app
COPY --from=build /app/dist ./dist
COPY --from=build /app/node_modules ./node_modules
CMD ["node", "dist/index.js"]
```

---

## Building and Tagging

### Build Commands

```bash
# Basic build
docker build -t myapp .

# With tag
docker build -t myapp:v1.0 .

# Multiple tags
docker build -t myapp:latest -t myapp:v1.0 .

# With registry prefix
docker build -t registry.example.com/myapp:v1.0 .

# Build specific target (multi-stage)
docker build --target builder -t myapp:builder .
```

### Tagging Images

```bash
# Tag existing image
docker tag myapp myapp:v1.0
docker tag myapp registry.example.com/myapp:v1.0

# Retag image
docker tag myapp:old myapp:new
```

### Pushing to Registry

```bash
# Login to Docker Hub
docker login

# Push image
docker push myapp:v1.0

# Push to other registry
docker login registry.example.com
docker push registry.example.com/myapp:v1.0
```

---

## Image Analysis

### Inspecting Images

```bash
# View image details
docker inspect myapp

# View image history (layers)
docker history myapp

# See layer sizes
docker history myapp --no-trunc

# Check image size
docker images myapp
```

### Using dive Tool

```bash
# Install dive
# https://github.com/wagoodman/dive

# Analyze image layers
dive myapp

# Features:
# - View layer contents
# - See wasted space
# - Identify optimization opportunities
```

---

## Practice Exercises

### Exercise 1: Basic Dockerfile

```dockerfile
# Create a Dockerfile for a simple Express app:
# 1. Use node:18-alpine
# 2. Set working directory
# 3. Copy and install dependencies
# 4. Copy source code
# 5. Expose port and set CMD
```

### Exercise 2: Multi-Stage Build

```dockerfile
# Create a multi-stage Dockerfile:
# 1. Build stage: install all deps, run build
# 2. Production stage: copy only necessary files
# 3. Compare image sizes
```

### Exercise 3: Optimization

```bash
# 1. Build an image without .dockerignore
# 2. Add .dockerignore and rebuild
# 3. Compare sizes and build times
# 4. Use dive to analyze layers
```

---

## Quick Reference

### Common Instructions

| Instruction | Purpose |
|-------------|---------|
| `FROM` | Base image |
| `WORKDIR` | Set working directory |
| `COPY` | Copy files |
| `RUN` | Execute command |
| `ENV` | Set environment variable |
| `ARG` | Build-time variable |
| `EXPOSE` | Document port |
| `CMD` | Default command |
| `ENTRYPOINT` | Fixed command |
| `USER` | Set user |
| `HEALTHCHECK` | Health check |

### Build Commands

| Command | Description |
|---------|-------------|
| `docker build -t name .` | Build image |
| `docker build -f file` | Use specific Dockerfile |
| `docker build --no-cache` | Build without cache |
| `docker tag` | Tag image |
| `docker push` | Push to registry |

---

## Key Takeaways

1. **Order matters** - Put frequently changing files last
2. **Minimize layers** - Combine RUN commands
3. **Use .dockerignore** - Exclude unnecessary files
4. **Multi-stage builds** - Smaller production images
5. **Use Alpine** - Smaller base images
6. **Non-root user** - Better security

---

## What's Next?

Tomorrow, we'll learn about **Docker Compose** - defining and running multi-container applications with a single configuration file.
