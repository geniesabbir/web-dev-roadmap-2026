# Day 4: Multi-Container Applications - Networking and Patterns

## Introduction

Modern applications are composed of multiple services that need to communicate. Docker provides powerful networking capabilities that allow containers to discover and connect to each other. Today, you'll learn how to architect and connect multi-container applications effectively.

## Learning Objectives

By the end of this lesson, you will be able to:
- Understand Docker networking concepts
- Connect containers across networks
- Implement service discovery
- Apply common multi-container patterns
- Debug network connectivity issues

---

## Docker Networking Basics

### Network Types

```
┌─────────────────────────────────────────────────────────────────┐
│                    Docker Network Drivers                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  bridge (default)     Host-isolated network for containers       │
│  ├── Container A ──────┬────── Container B                      │
│  │                     │                                         │
│  └─────────────────────┴─── docker0 bridge                      │
│                                                                  │
│  host                 Container uses host's network directly     │
│  ├── Container ─────────── Host Network Stack                   │
│                                                                  │
│  none                 No network connectivity                    │
│  └── Container (isolated)                                        │
│                                                                  │
│  overlay              Multi-host networking (Swarm)              │
│  └── Containers across multiple Docker hosts                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Default Bridge Network

```bash
# List networks
docker network ls
# NETWORK ID     NAME      DRIVER    SCOPE
# abc123...      bridge    bridge    local
# def456...      host      host      local
# ghi789...      none      null      local

# Inspect default bridge
docker network inspect bridge

# Containers on default bridge communicate by IP only
docker run -d --name web nginx
docker run -d --name api node:18
docker exec web ping 172.17.0.3  # Works by IP
docker exec web ping api         # Doesn't work by name
```

### User-Defined Networks

```bash
# Create custom network
docker network create myapp-network

# Run containers on custom network
docker run -d --name web --network myapp-network nginx
docker run -d --name api --network myapp-network node:18

# Now DNS works!
docker exec web ping api  # Works by name!

# Connect existing container to network
docker network connect myapp-network existing-container

# Disconnect from network
docker network disconnect myapp-network container-name
```

---

## Service Discovery

### DNS-Based Discovery

```yaml
# docker-compose.yml
version: "3.8"

services:
  frontend:
    build: ./frontend
    environment:
      # Reference other services by name
      - API_URL=http://api:4000
    depends_on:
      - api

  api:
    build: ./api
    environment:
      # Database connection by service name
      - DATABASE_URL=postgres://user:pass@db:5432/myapp
      - REDIS_URL=redis://cache:6379
    depends_on:
      - db
      - cache

  db:
    image: postgres:14

  cache:
    image: redis:7

# All services can reach each other by name
# frontend → api → db
#               → cache
```

### How DNS Works

```
┌─────────────────────────────────────────────────────────────────┐
│                Container DNS Resolution                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. App requests "db" hostname                                   │
│                     │                                            │
│  2. Docker's DNS ───┴──► Resolves to container IP               │
│     (127.0.0.11)                                                 │
│                                                                  │
│  3. Returns IP ────────► 172.18.0.4                             │
│                                                                  │
│  4. App connects to 172.18.0.4:5432                             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Service Aliases

```yaml
services:
  db:
    image: postgres:14
    networks:
      default:
        aliases:
          - database
          - postgres
          - pg

# Now db is reachable as:
# - db
# - database
# - postgres
# - pg
```

---

## Network Configuration

### Creating Networks

```yaml
version: "3.8"

services:
  frontend:
    networks:
      - frontend-net

  api:
    networks:
      - frontend-net
      - backend-net

  db:
    networks:
      - backend-net

networks:
  frontend-net:
    driver: bridge

  backend-net:
    driver: bridge
    internal: true  # No external access
```

### Network Isolation

```yaml
# Secure network architecture
version: "3.8"

services:
  # Public-facing
  nginx:
    image: nginx
    ports:
      - "80:80"
    networks:
      - public
      - app

  # Application tier
  api:
    build: ./api
    networks:
      - app
      - data
    # No ports exposed to host

  # Data tier
  db:
    image: postgres:14
    networks:
      - data
    # Completely isolated from public

networks:
  public:
    # External access allowed

  app:
    # API layer

  data:
    internal: true  # No external access
```

### Custom Network Settings

```yaml
networks:
  custom:
    driver: bridge
    driver_opts:
      com.docker.network.bridge.name: custom-br0

    ipam:
      driver: default
      config:
        - subnet: 172.28.0.0/16
          gateway: 172.28.0.1

services:
  app:
    networks:
      custom:
        ipv4_address: 172.28.0.10
```

---

## Common Patterns

### Pattern 1: Reverse Proxy

```yaml
version: "3.8"

services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - frontend
      - api

  frontend:
    build: ./frontend
    # No ports exposed

  api:
    build: ./api
    # No ports exposed
```

```nginx
# nginx.conf
upstream frontend {
    server frontend:3000;
}

upstream api {
    server api:4000;
}

server {
    listen 80;

    location / {
        proxy_pass http://frontend;
    }

    location /api {
        proxy_pass http://api;
    }
}
```

### Pattern 2: API Gateway

```yaml
version: "3.8"

services:
  gateway:
    build: ./gateway
    ports:
      - "8080:8080"
    environment:
      - USER_SERVICE=http://user-service:3001
      - ORDER_SERVICE=http://order-service:3002
      - PRODUCT_SERVICE=http://product-service:3003

  user-service:
    build: ./services/user

  order-service:
    build: ./services/order

  product-service:
    build: ./services/product
```

### Pattern 3: Sidecar Pattern

```yaml
version: "3.8"

services:
  app:
    build: ./app
    volumes:
      - logs:/var/log/app

  # Sidecar: Log collector
  log-collector:
    image: fluent/fluentd
    volumes:
      - logs:/var/log/app:ro
      - ./fluentd.conf:/fluentd/etc/fluent.conf

  # Sidecar: Metrics exporter
  metrics:
    image: prom/node-exporter
    network_mode: "service:app"

volumes:
  logs:
```

### Pattern 4: Database Per Service

```yaml
version: "3.8"

services:
  user-service:
    build: ./user-service
    environment:
      - DATABASE_URL=postgres://user:pass@user-db:5432/users

  order-service:
    build: ./order-service
    environment:
      - DATABASE_URL=postgres://user:pass@order-db:5432/orders

  user-db:
    image: postgres:14
    volumes:
      - user-data:/var/lib/postgresql/data

  order-db:
    image: postgres:14
    volumes:
      - order-data:/var/lib/postgresql/data

volumes:
  user-data:
  order-data:
```

---

## Load Balancing

### Docker Compose Scaling

```bash
# Scale service to multiple instances
docker compose up -d --scale api=3

# Nginx config for load balancing
```

```yaml
version: "3.8"

services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - api

  api:
    build: ./api
    # No ports - accessed through nginx
    deploy:
      replicas: 3
```

```nginx
# nginx.conf
upstream api {
    # Docker DNS resolves to all instances
    server api:4000;
}

server {
    listen 80;

    location /api {
        proxy_pass http://api;
    }
}
```

### Health-Based Routing

```yaml
services:
  api:
    build: ./api
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:4000/health"]
      interval: 10s
      timeout: 5s
      retries: 3
      start_period: 30s

  nginx:
    depends_on:
      api:
        condition: service_healthy
```

---

## Inter-Container Communication

### REST API Calls

```javascript
// user-service/index.js
const express = require('express');
const axios = require('axios');
const app = express();

// Call another service
app.get('/user/:id/orders', async (req, res) => {
  try {
    // Use service name as hostname
    const orders = await axios.get(
      `http://order-service:3002/orders?userId=${req.params.id}`
    );
    res.json(orders.data);
  } catch (error) {
    res.status(500).json({ error: 'Order service unavailable' });
  }
});

app.listen(3001);
```

### Message Queue Pattern

```yaml
version: "3.8"

services:
  producer:
    build: ./producer
    environment:
      - RABBITMQ_URL=amqp://rabbitmq:5672
    depends_on:
      rabbitmq:
        condition: service_healthy

  consumer:
    build: ./consumer
    environment:
      - RABBITMQ_URL=amqp://rabbitmq:5672
    depends_on:
      rabbitmq:
        condition: service_healthy

  rabbitmq:
    image: rabbitmq:3-management
    ports:
      - "15672:15672"  # Management UI
    healthcheck:
      test: rabbitmq-diagnostics check_running
      interval: 10s
      timeout: 5s
      retries: 5
```

### Shared Cache

```yaml
version: "3.8"

services:
  api-1:
    build: ./api
    environment:
      - REDIS_URL=redis://redis:6379

  api-2:
    build: ./api
    environment:
      - REDIS_URL=redis://redis:6379

  redis:
    image: redis:7-alpine
    volumes:
      - redis-data:/data

volumes:
  redis-data:
```

---

## Debugging Network Issues

### Inspect Networks

```bash
# List all networks
docker network ls

# Detailed network info
docker network inspect myapp-network

# See container IPs
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' container-name
```

### Test Connectivity

```bash
# Shell into container
docker exec -it container-name sh

# Test DNS resolution
nslookup other-service
dig other-service

# Test connectivity
ping other-service
curl http://other-service:port/endpoint

# Check listening ports
netstat -tuln
ss -tuln
```

### Debug Container

```yaml
services:
  debug:
    image: nicolaka/netshoot
    command: sleep infinity
    networks:
      - myapp-network

# Run:
# docker compose run debug
# Then use: ping, curl, dig, nslookup, etc.
```

### Common Issues

```bash
# Issue: Can't resolve hostname
# Solution: Ensure same network
docker network connect mynetwork container

# Issue: Connection refused
# Solution: Check service is listening on 0.0.0.0
# Not 127.0.0.1 (localhost only)

# Issue: Timeout
# Solution: Check firewall, security groups
# Check service is healthy
docker compose ps
docker compose logs service-name
```

---

## Real-World Example

### E-commerce Microservices

```yaml
version: "3.8"

services:
  # API Gateway
  gateway:
    build: ./gateway
    ports:
      - "8080:8080"
    environment:
      - JWT_SECRET=${JWT_SECRET}
    networks:
      - public
      - internal

  # User Service
  user-service:
    build: ./services/user
    environment:
      - DATABASE_URL=postgres://user:pass@user-db:5432/users
      - REDIS_URL=redis://cache:6379
    networks:
      - internal
      - user-net
    depends_on:
      - user-db
      - cache

  user-db:
    image: postgres:14-alpine
    volumes:
      - user-data:/var/lib/postgresql/data
    networks:
      - user-net

  # Product Service
  product-service:
    build: ./services/product
    environment:
      - DATABASE_URL=postgres://user:pass@product-db:5432/products
      - ELASTICSEARCH_URL=http://search:9200
    networks:
      - internal
      - product-net
    depends_on:
      - product-db
      - search

  product-db:
    image: postgres:14-alpine
    volumes:
      - product-data:/var/lib/postgresql/data
    networks:
      - product-net

  search:
    image: elasticsearch:8.9.0
    environment:
      - discovery.type=single-node
    volumes:
      - search-data:/usr/share/elasticsearch/data
    networks:
      - product-net

  # Order Service
  order-service:
    build: ./services/order
    environment:
      - DATABASE_URL=postgres://user:pass@order-db:5432/orders
      - RABBITMQ_URL=amqp://rabbitmq:5672
    networks:
      - internal
      - order-net
    depends_on:
      - order-db
      - rabbitmq

  order-db:
    image: postgres:14-alpine
    volumes:
      - order-data:/var/lib/postgresql/data
    networks:
      - order-net

  # Shared Services
  cache:
    image: redis:7-alpine
    networks:
      - internal

  rabbitmq:
    image: rabbitmq:3-management
    networks:
      - internal
      - order-net

networks:
  public:
  internal:
  user-net:
    internal: true
  product-net:
    internal: true
  order-net:
    internal: true

volumes:
  user-data:
  product-data:
  order-data:
  search-data:
```

---

## Practice Exercises

### Exercise 1: Service Communication

```yaml
# Create two services that communicate:
# 1. API service that exposes /data endpoint
# 2. Frontend that fetches from API
# 3. Verify communication works
```

### Exercise 2: Network Isolation

```yaml
# Create a secure architecture:
# 1. Public nginx on public network
# 2. API on app network (connected to nginx)
# 3. Database on internal-only network
# 4. Verify database is not accessible from outside
```

### Exercise 3: Debugging

```bash
# 1. Create a multi-service app
# 2. Intentionally break networking
# 3. Use debug techniques to identify issue
# 4. Fix and verify
```

---

## Quick Reference

### Network Commands

| Command | Description |
|---------|-------------|
| `docker network ls` | List networks |
| `docker network create` | Create network |
| `docker network inspect` | Network details |
| `docker network connect` | Add container to network |
| `docker network disconnect` | Remove from network |
| `docker network rm` | Delete network |

### Compose Network Options

| Option | Description |
|--------|-------------|
| `driver` | Network driver (bridge, overlay) |
| `internal` | No external access |
| `ipam` | IP address management |
| `aliases` | Additional DNS names |

---

## Key Takeaways

1. **User-defined networks** enable DNS resolution
2. **Services discover each other by name**
3. **Isolate tiers** with multiple networks
4. **Internal networks** block external access
5. **Health checks** ensure service readiness
6. **Debug tools** help troubleshoot connectivity

---

## What's Next?

Tomorrow, we'll cover **Docker Best Practices** - security, optimization, and production-ready configurations.
