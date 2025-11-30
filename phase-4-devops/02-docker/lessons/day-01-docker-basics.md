# Day 1: Docker Basics - Understanding Containerization

## Introduction

Docker revolutionized how we build, ship, and run applications. Containers provide consistent environments from development to production, eliminating the "it works on my machine" problem. Today, you'll learn what Docker is, how it works, and master the essential commands for working with containers.

## Learning Objectives

By the end of this lesson, you will be able to:
- Understand containerization concepts
- Install and configure Docker
- Run and manage containers
- Work with Docker images
- Use essential Docker commands

---

## What is Docker?

### Containers vs Virtual Machines

```
┌─────────────────────────────────────────────────────────────────┐
│                    Virtual Machines                              │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────┐  ┌─────────┐  ┌─────────┐                         │
│  │  App A  │  │  App B  │  │  App C  │                         │
│  ├─────────┤  ├─────────┤  ├─────────┤                         │
│  │ Bins/   │  │ Bins/   │  │ Bins/   │                         │
│  │ Libs    │  │ Libs    │  │ Libs    │                         │
│  ├─────────┤  ├─────────┤  ├─────────┤                         │
│  │Guest OS │  │Guest OS │  │Guest OS │  ← Full OS per VM       │
│  └─────────┘  └─────────┘  └─────────┘                         │
│  ┌─────────────────────────────────────┐                       │
│  │           Hypervisor                 │                       │
│  └─────────────────────────────────────┘                       │
│  ┌─────────────────────────────────────┐                       │
│  │           Host OS                    │                       │
│  └─────────────────────────────────────┘                       │
│  ┌─────────────────────────────────────┐                       │
│  │           Infrastructure             │                       │
│  └─────────────────────────────────────┘                       │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                       Containers                                 │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────┐  ┌─────────┐  ┌─────────┐                         │
│  │  App A  │  │  App B  │  │  App C  │                         │
│  ├─────────┤  ├─────────┤  ├─────────┤                         │
│  │ Bins/   │  │ Bins/   │  │ Bins/   │                         │
│  │ Libs    │  │ Libs    │  │ Libs    │                         │
│  └─────────┘  └─────────┘  └─────────┘                         │
│  ┌─────────────────────────────────────┐                       │
│  │         Docker Engine               │  ← Shared kernel      │
│  └─────────────────────────────────────┘                       │
│  ┌─────────────────────────────────────┐                       │
│  │           Host OS                    │                       │
│  └─────────────────────────────────────┘                       │
│  ┌─────────────────────────────────────┐                       │
│  │           Infrastructure             │                       │
│  └─────────────────────────────────────┘                       │
└─────────────────────────────────────────────────────────────────┘
```

### Key Differences

| Feature | Virtual Machines | Containers |
|---------|-----------------|------------|
| Size | GBs | MBs |
| Startup | Minutes | Seconds |
| Isolation | Full OS isolation | Process isolation |
| OS | Each has own OS | Shares host kernel |
| Resource usage | Heavy | Lightweight |
| Portability | Hardware dependent | Highly portable |

### Docker Benefits

- **Consistency** - Same environment everywhere
- **Isolation** - Dependencies don't conflict
- **Portability** - Run anywhere Docker runs
- **Efficiency** - Less overhead than VMs
- **Scalability** - Easy to scale up/down
- **Version control** - Track image changes

---

## Installing Docker

### Linux (Ubuntu/Debian)

```bash
# Update package index
sudo apt update

# Install prerequisites
sudo apt install -y \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

# Add Docker's GPG key
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
    sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Set up repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Add user to docker group (avoid sudo)
sudo usermod -aG docker $USER
newgrp docker

# Verify installation
docker --version
docker run hello-world
```

### macOS / Windows

```bash
# Install Docker Desktop from:
# https://www.docker.com/products/docker-desktop

# Verify installation
docker --version
docker run hello-world
```

---

## Docker Architecture

### Components

```
┌─────────────────────────────────────────────────────────────┐
│                       Docker Client                          │
│                    (docker commands)                         │
└─────────────────────────┬───────────────────────────────────┘
                          │ REST API
┌─────────────────────────▼───────────────────────────────────┐
│                      Docker Daemon                           │
│                       (dockerd)                              │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │   Images    │  │  Containers │  │  Networks   │         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
│  ┌─────────────┐  ┌─────────────┐                          │
│  │   Volumes   │  │   Plugins   │                          │
│  └─────────────┘  └─────────────┘                          │
└─────────────────────────────────────────────────────────────┘
                          │
┌─────────────────────────▼───────────────────────────────────┐
│                    Docker Registry                           │
│                (Docker Hub, private registries)              │
└─────────────────────────────────────────────────────────────┘
```

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Image** | Read-only template with application and dependencies |
| **Container** | Running instance of an image |
| **Registry** | Storage for Docker images (Docker Hub) |
| **Dockerfile** | Instructions to build an image |
| **Volume** | Persistent data storage |
| **Network** | Communication between containers |

---

## Working with Images

### Finding Images

```bash
# Search Docker Hub
docker search nginx
docker search node --limit 5

# Browse Docker Hub website
# https://hub.docker.com
```

### Pulling Images

```bash
# Pull latest version
docker pull nginx
docker pull node

# Pull specific version (tag)
docker pull node:18
docker pull node:18-alpine
docker pull postgres:14

# Pull from different registry
docker pull ghcr.io/owner/image:tag
```

### Listing Images

```bash
# List all images
docker images
docker image ls

# List with specific format
docker images --format "{{.Repository}}:{{.Tag}} - {{.Size}}"

# Show image IDs only
docker images -q

# Show all images (including intermediate)
docker images -a

# Filter images
docker images --filter "dangling=true"
docker images | grep node
```

### Removing Images

```bash
# Remove image
docker rmi nginx
docker image rm nginx

# Remove by ID
docker rmi abc123

# Remove multiple images
docker rmi nginx node postgres

# Force remove (even if used by container)
docker rmi -f nginx

# Remove all unused images
docker image prune

# Remove ALL images
docker rmi $(docker images -q)
```

### Image Information

```bash
# Inspect image details
docker image inspect nginx

# View image history (layers)
docker history nginx

# Show image size
docker images nginx --format "{{.Size}}"
```

---

## Running Containers

### Basic Run

```bash
# Run container
docker run nginx

# Run in background (detached)
docker run -d nginx

# Run with name
docker run -d --name my-nginx nginx

# Run interactively
docker run -it ubuntu bash
docker run -it node:18 node

# Run and remove when stopped
docker run --rm nginx

# Run with environment variables
docker run -d -e POSTGRES_PASSWORD=secret postgres

# Run with multiple env vars
docker run -d \
    -e POSTGRES_USER=admin \
    -e POSTGRES_PASSWORD=secret \
    -e POSTGRES_DB=myapp \
    postgres:14
```

### Port Mapping

```bash
# Map port: -p host:container
docker run -d -p 8080:80 nginx
# Access at http://localhost:8080

# Map multiple ports
docker run -d -p 8080:80 -p 443:443 nginx

# Map to random port
docker run -d -P nginx
docker port <container-id>

# Bind to specific interface
docker run -d -p 127.0.0.1:8080:80 nginx
```

### Volume Mounting

```bash
# Mount directory: -v host:container
docker run -d -v /host/path:/container/path nginx

# Mount current directory
docker run -d -v $(pwd):/app node:18

# Named volume
docker run -d -v mydata:/var/lib/data postgres

# Read-only mount
docker run -d -v $(pwd)/config:/etc/nginx/conf.d:ro nginx
```

---

## Managing Containers

### Listing Containers

```bash
# List running containers
docker ps
docker container ls

# List all containers (including stopped)
docker ps -a

# List with specific format
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"

# Show container IDs only
docker ps -q

# Show latest created container
docker ps -l
```

### Container Lifecycle

```bash
# Stop container
docker stop my-nginx
docker stop $(docker ps -q)  # Stop all

# Start stopped container
docker start my-nginx

# Restart container
docker restart my-nginx

# Pause/unpause
docker pause my-nginx
docker unpause my-nginx

# Kill container (force stop)
docker kill my-nginx
```

### Removing Containers

```bash
# Remove stopped container
docker rm my-nginx

# Force remove running container
docker rm -f my-nginx

# Remove all stopped containers
docker container prune

# Remove all containers
docker rm -f $(docker ps -aq)
```

### Container Information

```bash
# View logs
docker logs my-nginx
docker logs -f my-nginx        # Follow
docker logs --tail 100 my-nginx  # Last 100 lines
docker logs --since 1h my-nginx  # Last hour

# Inspect container
docker inspect my-nginx

# View resource usage
docker stats
docker stats my-nginx

# View running processes
docker top my-nginx

# View port mappings
docker port my-nginx
```

### Executing Commands

```bash
# Run command in container
docker exec my-nginx ls /etc/nginx

# Interactive shell
docker exec -it my-nginx bash
docker exec -it my-nginx sh

# Run as specific user
docker exec -u root my-nginx whoami

# Set working directory
docker exec -w /app my-container npm test
```

---

## Practical Examples

### Running a Web Server

```bash
# Run Nginx
docker run -d \
    --name webserver \
    -p 8080:80 \
    -v $(pwd)/html:/usr/share/nginx/html:ro \
    nginx

# Create HTML file
mkdir html
echo "<h1>Hello Docker!</h1>" > html/index.html

# Access at http://localhost:8080
```

### Running a Database

```bash
# Run PostgreSQL
docker run -d \
    --name postgres \
    -e POSTGRES_USER=admin \
    -e POSTGRES_PASSWORD=secret \
    -e POSTGRES_DB=myapp \
    -p 5432:5432 \
    -v pgdata:/var/lib/postgresql/data \
    postgres:14

# Connect to database
docker exec -it postgres psql -U admin -d myapp
```

### Running a Node.js App

```bash
# Run Node.js interactively
docker run -it --rm node:18 node

# Run with mounted source
docker run -it --rm \
    -v $(pwd):/app \
    -w /app \
    node:18 \
    npm install

# Run dev server
docker run -it --rm \
    -v $(pwd):/app \
    -w /app \
    -p 3000:3000 \
    node:18 \
    npm run dev
```

---

## Docker Cleanup

### Removing Unused Resources

```bash
# Remove stopped containers
docker container prune

# Remove unused images
docker image prune

# Remove unused volumes
docker volume prune

# Remove unused networks
docker network prune

# Remove everything unused
docker system prune

# Remove everything including volumes
docker system prune -a --volumes

# Check disk usage
docker system df
```

---

## Practice Exercises

### Exercise 1: Run Web Server

```bash
# 1. Pull nginx:alpine image
# 2. Run container on port 8080
# 3. Create custom HTML file
# 4. Mount it to the container
# 5. View in browser
```

### Exercise 2: Database Container

```bash
# 1. Run PostgreSQL container
# 2. Set environment variables
# 3. Create a named volume for data
# 4. Connect and create a table
# 5. Stop and start - verify data persists
```

### Exercise 3: Container Management

```bash
# 1. Run 3 nginx containers with different names
# 2. List all running containers
# 3. View logs from one container
# 4. Execute a command inside container
# 5. Stop and remove all containers
```

---

## Quick Reference

### Essential Commands

| Command | Description |
|---------|-------------|
| `docker run` | Create and start container |
| `docker ps` | List containers |
| `docker images` | List images |
| `docker pull` | Download image |
| `docker stop` | Stop container |
| `docker start` | Start container |
| `docker rm` | Remove container |
| `docker rmi` | Remove image |
| `docker logs` | View container logs |
| `docker exec` | Run command in container |

### Run Options

| Option | Description |
|--------|-------------|
| `-d` | Detached mode |
| `-p host:container` | Port mapping |
| `-v host:container` | Volume mount |
| `-e VAR=value` | Environment variable |
| `--name` | Container name |
| `--rm` | Remove on exit |
| `-it` | Interactive terminal |

---

## Key Takeaways

1. **Containers are lightweight** - Share host kernel
2. **Images are templates** - Containers are instances
3. **Use -d for background** - Detached mode
4. **Use -p for ports** - Map container to host
5. **Use -v for data** - Persist beyond container life
6. **Clean up regularly** - docker system prune

---

## What's Next?

Tomorrow, we'll learn how to **build custom Docker images** using Dockerfiles - defining exactly what goes into your application containers.
