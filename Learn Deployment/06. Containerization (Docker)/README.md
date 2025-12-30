# Containerization with Docker

## Table of Contents
1. [What is Docker?](#what-is-docker)
2. [Docker Architecture](#docker-architecture)
3. [Docker Images](#docker-images)
4. [Docker Containers](#docker-containers)
5. [Dockerfile](#dockerfile)
6. [Docker Compose](#docker-compose)
7. [Multi-Stage Builds](#multi-stage-builds)
8. [Docker Networking](#docker-networking)
9. [Docker Volumes](#docker-volumes)
10. [Best Practices](#best-practices)
11. [Orchestration (Kubernetes Intro)](#orchestration)

---

## What is Docker?

### Definition
Docker is a platform for developing, shipping, and running applications in containers. Containers package application code with all dependencies, ensuring consistency across environments.

### Why Docker?
```
Problem:
"It works on my machine!" 😤

Without Docker:
- Different OS versions
- Different dependency versions
- Different configurations
- Hard to replicate production locally

With Docker:
- Same container everywhere ✅
- Consistent environment
- Easy to replicate
- Isolated applications
```

### Virtual Machines vs Containers
```
┌─────────────────────────────────────────────────────┐
│                Virtual Machines                      │
├─────────────────────────────────────────────────────┤
│  App A    │   App B    │   App C                    │
│  Bins     │   Bins     │   Bins                     │
│  Libs     │   Libs     │   Libs                     │
├───────────┼────────────┼────────────────────────────┤
│  Guest OS │  Guest OS  │  Guest OS                  │
├───────────┴────────────┴────────────────────────────┤
│              Hypervisor                              │
├──────────────────────────────────────────────────────┤
│              Host OS                                 │
├──────────────────────────────────────────────────────┤
│              Infrastructure                          │
└──────────────────────────────────────────────────────┘

VM Characteristics:
- Heavy (GBs)
- Slow to start (minutes)
- Full OS per application
- More isolated


┌─────────────────────────────────────────────────────┐
│                  Containers                          │
├─────────────────────────────────────────────────────┤
│  App A    │   App B    │   App C                    │
│  Bins     │   Bins     │   Bins                     │
│  Libs     │   Libs     │   Libs                     │
├───────────┴────────────┴────────────────────────────┤
│              Docker Engine                           │
├──────────────────────────────────────────────────────┤
│              Host OS                                 │
├──────────────────────────────────────────────────────┤
│              Infrastructure                          │
└──────────────────────────────────────────────────────┘

Container Characteristics:
- Lightweight (MBs)
- Fast to start (seconds)
- Share host OS kernel
- Less isolated but efficient
```

### Key Benefits
```
1. Consistency
   Development = Staging = Production

2. Isolation
   Each container runs independently

3. Portability
   Run anywhere: laptop, cloud, on-premises

4. Efficiency
   Share OS kernel, use less resources

5. Scalability
   Start/stop containers quickly

6. Version Control
   Image versioning like Git
```

---

## Docker Architecture

### Components
```
┌────────────────────────────────────────────────┐
│              Docker Client                     │
│  (docker build, docker run, docker push)       │
└────────────┬───────────────────────────────────┘
             │ REST API
┌────────────▼───────────────────────────────────┐
│              Docker Daemon                     │
│  - Manages images                              │
│  - Manages containers                          │
│  - Manages networks                            │
│  - Manages volumes                             │
└────────────┬───────────────────────────────────┘
             │
┌────────────▼───────────────────────────────────┐
│              Docker Registry                   │
│  (Docker Hub, Private Registry)                │
│  - Stores images                               │
└────────────────────────────────────────────────┘
```

### Key Concepts
```
Image:
- Read-only template
- Contains application code + dependencies
- Built from Dockerfile
- Example: node:18-alpine

Container:
- Running instance of an image
- Isolated process
- Has own filesystem, network
- Example: Running Node.js app

Registry:
- Repository for images
- Docker Hub (public)
- Private registries
- Example: hub.docker.com

Dockerfile:
- Recipe to build image
- Text file with instructions
- Example: FROM, RUN, COPY, CMD
```

---

## Docker Images

### What is a Docker Image?
A lightweight, standalone package containing everything needed to run an application.

### Image Layers
```
┌─────────────────────────────────────┐
│  Layer 4: CMD ["node", "app.js"]    │  (4 bytes)
├─────────────────────────────────────┤
│  Layer 3: COPY . .                  │  (1 MB)
├─────────────────────────────────────┤
│  Layer 2: RUN npm install           │  (50 MB)
├─────────────────────────────────────┤
│  Layer 1: FROM node:18-alpine       │  (40 MB)
└─────────────────────────────────────┘

Each instruction in Dockerfile = New layer
Layers are cached and reused
Changes only rebuild affected layers
```

### Common Image Commands
```bash
# Pull image from registry
docker pull node:18-alpine

# List local images
docker images

# Build image from Dockerfile
docker build -t myapp:1.0 .

# Tag image
docker tag myapp:1.0 username/myapp:1.0

# Push image to registry
docker push username/myapp:1.0

# Remove image
docker rmi myapp:1.0

# Inspect image layers
docker history myapp:1.0

# Remove unused images
docker image prune
```

### Image Naming Convention
```
[registry]/[username]/[repository]:[tag]

Examples:
node:18-alpine
  └─ Official Node.js image, version 18, alpine variant

docker.io/library/postgres:14
  └─ Docker Hub official Postgres 14

myregistry.com/myteam/myapp:v1.2.3
  └─ Private registry, team namespace, versioned app

myapp:latest
  └─ Local image, latest tag (default)
```

---

## Docker Containers

### What is a Container?
A running instance of a Docker image with its own isolated filesystem, network, and processes.

### Container Lifecycle
```
Created → Running → Paused → Stopped → Deleted

docker create  → Container created (not running)
docker start   → Container running
docker pause   → Container paused (frozen)
docker unpause → Container running again
docker stop    → Container stopped gracefully
docker kill    → Container stopped forcefully
docker rm      → Container removed
```

### Common Container Commands
```bash
# Run container (create + start)
docker run -d -p 3000:3000 --name myapp myapp:1.0

# Run with environment variables
docker run -e DATABASE_URL=postgres://... myapp:1.0

# Run with volume mount
docker run -v $(pwd):/app myapp:1.0

# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# View container logs
docker logs myapp
docker logs -f myapp  # Follow logs (tail -f)

# Execute command in running container
docker exec -it myapp bash
docker exec myapp ls /app

# Stop container
docker stop myapp

# Start stopped container
docker start myapp

# Restart container
docker restart myapp

# Remove container
docker rm myapp
docker rm -f myapp  # Force remove (even if running)

# Remove all stopped containers
docker container prune
```

### Container Options
```bash
# Common flags
-d              # Detached mode (background)
-p 3000:3000    # Port mapping (host:container)
--name myapp    # Container name
-e KEY=value    # Environment variable
-v /host:/container  # Volume mount
--rm            # Auto-remove after exit
-it             # Interactive terminal
--network mynet # Connect to network
--link db:db    # Link to another container (legacy)
--restart unless-stopped  # Restart policy

# Example with multiple options
docker run -d \
  --name backend \
  -p 4000:4000 \
  -e NODE_ENV=production \
  -e DATABASE_URL=postgres://db:5432/mydb \
  -v $(pwd)/logs:/app/logs \
  --network myapp-network \
  --restart unless-stopped \
  myapp:1.0
```

---

## Dockerfile

### What is a Dockerfile?
A text file containing instructions to build a Docker image.

### Basic Dockerfile Structure
```dockerfile
# Dockerfile

# Base image
FROM node:18-alpine

# Metadata
LABEL maintainer="you@example.com"
LABEL version="1.0"

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy application code
COPY . .

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD node healthcheck.js || exit 1

# Default command
CMD ["node", "server.js"]
```

### Common Dockerfile Instructions

#### FROM - Base Image
```dockerfile
# Official Node.js image
FROM node:18-alpine

# Specific version
FROM node:18.16.0-alpine

# Multi-stage build
FROM node:18-alpine AS build
FROM nginx:alpine AS production
```

#### WORKDIR - Set Working Directory
```dockerfile
# Sets /app as working directory
WORKDIR /app

# All subsequent commands run from /app
# Equivalent to: cd /app
```

#### COPY vs ADD
```dockerfile
# COPY - Simple copy (preferred)
COPY package.json .
COPY src/ ./src/
COPY . .

# ADD - Copy with extras (auto-extract tar, download URLs)
ADD archive.tar.gz /app/  # Auto-extracts
ADD https://example.com/file.txt /app/  # Downloads
```

#### RUN - Execute Commands
```dockerfile
# Install packages
RUN apt-get update && apt-get install -y curl

# Install npm dependencies
RUN npm ci --only=production

# Multiple commands (chain with &&)
RUN npm install && \
    npm run build && \
    npm prune --production
```

#### ENV - Environment Variables
```dockerfile
# Set environment variables
ENV NODE_ENV=production
ENV PORT=3000
ENV DATABASE_URL=postgres://localhost/db

# Use in subsequent commands
RUN echo "Port: $PORT"
```

#### ARG - Build Arguments
```dockerfile
# Define build-time variables
ARG NODE_VERSION=18
ARG BUILD_DATE

# Use in FROM
FROM node:${NODE_VERSION}-alpine

# Use in RUN
RUN echo "Built on ${BUILD_DATE}"

# Build with: docker build --build-arg BUILD_DATE=$(date) .
```

#### EXPOSE - Document Port
```dockerfile
# Document which port app listens on
EXPOSE 3000

# Note: This doesn't actually publish the port
# Still need: docker run -p 3000:3000
```

#### CMD vs ENTRYPOINT
```dockerfile
# CMD - Default command (can be overridden)
CMD ["node", "server.js"]
# Override: docker run myapp node other.js

# ENTRYPOINT - Always runs (arguments appended)
ENTRYPOINT ["node"]
CMD ["server.js"]
# Override: docker run myapp other.js
# Runs: node other.js

# Both together
ENTRYPOINT ["docker-entrypoint.sh"]
CMD ["node", "server.js"]
```

#### VOLUME - Define Mount Points
```dockerfile
# Create mount point
VOLUME /app/data

# Container data persists even if container is removed
```

#### USER - Set User
```dockerfile
# Create non-root user (security best practice)
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Switch to non-root user
USER appuser

# All subsequent commands run as appuser
```

### Complete Dockerfile Examples

#### Frontend (React)
```dockerfile
# Build stage
FROM node:18-alpine AS build

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine

# Copy build files to nginx
COPY --from=build /app/build /usr/share/nginx/html

# Copy custom nginx config
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

#### Backend (Node.js)
```dockerfile
FROM node:18-alpine

# Install dumb-init (proper signal handling)
RUN apk add --no-cache dumb-init

# Create app directory
WORKDIR /app

# Install dependencies
COPY package*.json ./
RUN npm ci --only=production

# Copy app source
COPY . .

# Use non-root user
RUN chown -R node:node /app
USER node

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s \
  CMD node -e "require('http').get('http://localhost:3000/health', (r) => { process.exit(r.statusCode === 200 ? 0 : 1) })"

# Use dumb-init to handle signals properly
ENTRYPOINT ["dumb-init", "--"]
CMD ["node", "server.js"]
```

#### Full Stack (NestJS)
```dockerfile
# Development stage
FROM node:18-alpine AS development

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

CMD ["npm", "run", "start:dev"]

# Build stage
FROM node:18-alpine AS build

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

# Production stage
FROM node:18-alpine AS production

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY --from=build /app/dist ./dist

USER node

EXPOSE 3000

CMD ["node", "dist/main.js"]
```

---

## Docker Compose

### What is Docker Compose?
Tool for defining and running multi-container Docker applications using a YAML file.

### Why Docker Compose?
```
Without Compose:
docker network create mynetwork
docker run -d --name db --network mynetwork postgres
docker run -d --name backend --network mynetwork -p 4000:4000 mybackend
docker run -d --name frontend --network mynetwork -p 3000:3000 myfrontend

With Compose:
docker-compose up
```

### Basic docker-compose.yml
```yaml
version: '3.8'

services:
  # Frontend service
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    ports:
      - "3000:80"
    environment:
      - REACT_APP_API_URL=http://localhost:4000
    depends_on:
      - backend
    networks:
      - app-network

  # Backend service
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    ports:
      - "4000:4000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgres://postgres:password@db:5432/mydb
    depends_on:
      - db
    networks:
      - app-network
    volumes:
      - ./backend/logs:/app/logs

  # Database service
  db:
    image: postgres:14-alpine
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=mydb
    ports:
      - "5432:5432"
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - app-network

networks:
  app-network:
    driver: bridge

volumes:
  postgres-data:
```

### Docker Compose Commands
```bash
# Start services
docker-compose up

# Start in background
docker-compose up -d

# Build images before starting
docker-compose up --build

# Stop services
docker-compose stop

# Stop and remove containers
docker-compose down

# Stop and remove containers, volumes
docker-compose down -v

# View logs
docker-compose logs
docker-compose logs -f backend

# List running services
docker-compose ps

# Execute command in service
docker-compose exec backend sh

# Scale service
docker-compose up -d --scale backend=3

# View config
docker-compose config
```

### Full Stack Example
```yaml
# docker-compose.yml
version: '3.8'

services:
  # React Frontend
  frontend:
    build:
      context: ./frontend
      target: production
    ports:
      - "80:80"
    environment:
      - REACT_APP_API_URL=http://localhost:4000/api
    depends_on:
      - backend
    restart: unless-stopped

  # NestJS Backend
  backend:
    build:
      context: ./backend
      target: production
    ports:
      - "4000:4000"
    environment:
      - NODE_ENV=production
      - PORT=4000
      - DATABASE_URL=postgresql://postgres:password@postgres:5432/myapp
      - REDIS_URL=redis://redis:6379
      - JWT_SECRET=${JWT_SECRET}
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_started
    restart: unless-stopped
    volumes:
      - ./backend/uploads:/app/uploads

  # PostgreSQL Database
  postgres:
    image: postgres:14-alpine
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=myapp
    ports:
      - "5432:5432"
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./database/init.sql:/docker-entrypoint-initdb.d/init.sql
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped

  # Redis Cache
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    command: redis-server --appendonly yes
    restart: unless-stopped

  # Nginx Reverse Proxy
  nginx:
    image: nginx:alpine
    ports:
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./nginx/ssl:/etc/nginx/ssl
    depends_on:
      - frontend
      - backend
    restart: unless-stopped

volumes:
  postgres-data:
  redis-data:
```

### Development vs Production Compose

#### docker-compose.dev.yml
```yaml
version: '3.8'

services:
  frontend:
    build:
      context: ./frontend
      target: development
    ports:
      - "3000:3000"
    volumes:
      - ./frontend:/app
      - /app/node_modules
    environment:
      - CHOKIDAR_USEPOLLING=true  # Hot reload
    command: npm start

  backend:
    build:
      context: ./backend
      target: development
    ports:
      - "4000:4000"
      - "9229:9229"  # Debug port
    volumes:
      - ./backend:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
    command: npm run start:dev

  postgres:
    image: postgres:14-alpine
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=dev_password
      - POSTGRES_DB=myapp_dev
    ports:
      - "5432:5432"
```

#### Usage
```bash
# Development
docker-compose -f docker-compose.dev.yml up

# Production
docker-compose up

# Override with multiple files
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up
```

---

## Multi-Stage Builds

### Why Multi-Stage Builds?
- Smaller final images
- Separate build and runtime dependencies
- Better security (no build tools in production)

### Basic Multi-Stage Example
```dockerfile
# Stage 1: Build
FROM node:18-alpine AS build

WORKDIR /app

COPY package*.json ./
RUN npm install  # All dependencies (including devDependencies)

COPY . .
RUN npm run build  # Build TypeScript, bundle, etc.

# Stage 2: Production
FROM node:18-alpine AS production

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production  # Only production dependencies

# Copy built files from build stage
COPY --from=build /app/dist ./dist

USER node

CMD ["node", "dist/main.js"]
```

### Advanced Multi-Stage Example
```dockerfile
# Base stage - shared dependencies
FROM node:18-alpine AS base
WORKDIR /app
COPY package*.json ./

# Dependencies stage
FROM base AS dependencies
RUN npm ci
COPY . .

# Test stage
FROM dependencies AS test
RUN npm run lint
RUN npm test

# Build stage
FROM dependencies AS build
RUN npm run build

# Production stage
FROM node:18-alpine AS production
WORKDIR /app

# Copy only production dependencies
COPY package*.json ./
RUN npm ci --only=production

# Copy built application
COPY --from=build /app/dist ./dist

# Security: non-root user
USER node

EXPOSE 3000
CMD ["node", "dist/main.js"]
```

### Image Size Comparison
```
Without multi-stage:
- Base: node:18            (~180 MB)
- Dependencies: node_modules (~200 MB)
- Dev dependencies         (~100 MB)
- Source code              (~10 MB)
Total: ~490 MB

With multi-stage:
- Base: node:18-alpine     (~40 MB)
- Production node_modules  (~50 MB)
- Built code               (~5 MB)
Total: ~95 MB

Reduction: 80% smaller! 🎉
```

---

## Docker Networking

### Network Types
```
┌──────────────────────────────────────────────┐
│ Network Drivers                               │
├──────────────────────────────────────────────┤
│ bridge   - Default, isolated network         │
│ host     - Use host's network                │
│ none     - No networking                     │
│ overlay  - Multi-host networking (Swarm)     │
│ macvlan  - Assign MAC address to container   │
└──────────────────────────────────────────────┘
```

### Bridge Network (Default)
```bash
# Create custom bridge network
docker network create mynetwork

# Run containers on same network
docker run -d --name db --network mynetwork postgres
docker run -d --name backend --network mynetwork mybackend

# Containers can communicate using container names
# backend can connect to: postgres://db:5432
```

### Network Commands
```bash
# List networks
docker network ls

# Create network
docker network create mynetwork

# Inspect network
docker network inspect mynetwork

# Connect container to network
docker network connect mynetwork mycontainer

# Disconnect container from network
docker network disconnect mynetwork mycontainer

# Remove network
docker network rm mynetwork
```

### Container Communication
```yaml
# docker-compose.yml - Automatic networking
services:
  backend:
    build: ./backend
    networks:
      - app-network
    environment:
      # Use service name as hostname
      - DATABASE_URL=postgres://postgres:password@db:5432/mydb
      - REDIS_URL=redis://cache:6379

  db:
    image: postgres:14
    networks:
      - app-network

  cache:
    image: redis:7
    networks:
      - app-network

networks:
  app-network:
    driver: bridge
```

```typescript
// Backend connecting to database
// Use service name from docker-compose.yml
const connection = await createConnection({
  type: 'postgres',
  host: 'db',  // Service name, not 'localhost'
  port: 5432,
  username: 'postgres',
  password: 'password',
  database: 'mydb'
});
```

---

## Docker Volumes

### Why Volumes?
- Persist data beyond container lifecycle
- Share data between containers
- Store data outside container filesystem

### Volume Types
```
1. Named Volumes (Managed by Docker)
   docker volume create mydata
   docker run -v mydata:/app/data myapp

2. Bind Mounts (Host directory)
   docker run -v /host/path:/container/path myapp

3. tmpfs Mounts (In-memory)
   docker run --tmpfs /app/temp myapp
```

### Volume Commands
```bash
# Create volume
docker volume create mydata

# List volumes
docker volume ls

# Inspect volume
docker volume inspect mydata

# Remove volume
docker volume rm mydata

# Remove unused volumes
docker volume prune
```

### Using Volumes
```bash
# Named volume
docker run -d \
  -v postgres-data:/var/lib/postgresql/data \
  postgres:14

# Bind mount (development)
docker run -d \
  -v $(pwd)/src:/app/src \
  myapp

# Read-only mount
docker run -d \
  -v $(pwd)/config:/app/config:ro \
  myapp
```

### Volumes in Docker Compose
```yaml
services:
  backend:
    image: mybackend
    volumes:
      # Named volume
      - logs-data:/app/logs
      
      # Bind mount (development)
      - ./src:/app/src
      
      # Bind mount (read-only)
      - ./config:/app/config:ro

  postgres:
    image: postgres:14
    volumes:
      # Named volume for data persistence
      - postgres-data:/var/lib/postgresql/data
      
      # Bind mount for init scripts
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql:ro

volumes:
  logs-data:
  postgres-data:
```

### Data Persistence Example
```bash
# Create container with volume
docker run -d --name db -v mydata:/var/lib/postgresql/data postgres:14

# Add data to database
docker exec -it db psql -U postgres -c "CREATE TABLE users (id SERIAL PRIMARY KEY, name TEXT);"

# Remove container
docker rm -f db

# Create new container with same volume
docker run -d --name db2 -v mydata:/var/lib/postgresql/data postgres:14

# Data still exists!
docker exec -it db2 psql -U postgres -c "SELECT * FROM users;"
```

---

## Best Practices

### 1. Use Official Base Images
```dockerfile
# Good - Official, maintained
FROM node:18-alpine
FROM postgres:14-alpine
FROM nginx:alpine

# Bad - Unknown source
FROM random/nodejs:latest
```

### 2. Use Specific Tags
```dockerfile
# Good - Specific version
FROM node:18.16.0-alpine

# Bad - Can break unexpectedly
FROM node:latest
```

### 3. Minimize Layers
```dockerfile
# Bad - Multiple layers
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y git
RUN apt-get clean

# Good - Single layer
RUN apt-get update && \
    apt-get install -y curl git && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

### 4. Order Instructions by Change Frequency
```dockerfile
# Good order
FROM node:18-alpine

WORKDIR /app

# Rarely changes - cached
COPY package*.json ./
RUN npm ci

# Changes frequently - rebuild from here
COPY . .

RUN npm run build

CMD ["node", "server.js"]
```

### 5. Use .dockerignore
```
# .dockerignore
node_modules
npm-debug.log
.git
.env
.env.local
dist
build
coverage
.vscode
.idea
*.md
.gitignore
Dockerfile
docker-compose.yml
```

### 6. Run as Non-Root User
```dockerfile
# Create user
RUN addgroup -S appgroup && \
    adduser -S appuser -G appgroup

# Change ownership
RUN chown -R appuser:appgroup /app

# Switch user
USER appuser

# Security benefit: Limited permissions
```

### 7. Use Multi-Stage Builds
```dockerfile
# Build with all tools
FROM node:18 AS build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Production with minimal size
FROM node:18-alpine
WORKDIR /app
COPY --from=build /app/dist ./dist
COPY package*.json ./
RUN npm ci --only=production
USER node
CMD ["node", "dist/main.js"]
```

### 8. Add Health Checks
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD node healthcheck.js || exit 1
```

```javascript
// healthcheck.js
const http = require('http');

const options = {
  host: 'localhost',
  port: 3000,
  path: '/health',
  timeout: 2000
};

const request = http.request(options, (res) => {
  process.exit(res.statusCode === 200 ? 0 : 1);
});

request.on('error', () => {
  process.exit(1);
});

request.end();
```

### 9. Use BuildKit for Better Caching
```bash
# Enable BuildKit
export DOCKER_BUILDKIT=1

# Or prefix command
DOCKER_BUILDKIT=1 docker build .
```

```dockerfile
# syntax=docker/dockerfile:1.4

# Use cache mounts
RUN --mount=type=cache,target=/root/.npm \
    npm install
```

### 10. Security Scanning
```bash
# Scan image for vulnerabilities
docker scan myapp:latest

# Use Snyk
snyk container test myapp:latest

# Use Trivy
trivy image myapp:latest
```

---

## Orchestration (Kubernetes Intro)

### What is Orchestration?
Managing multiple containers across multiple hosts.

### Docker Swarm vs Kubernetes
```
Docker Swarm:
- Simpler to set up
- Good for small clusters
- Built into Docker
- Less powerful

Kubernetes:
- Industry standard
- Complex but powerful
- Huge ecosystem
- Better for large scale
```

### Kubernetes Basics
```yaml
# Deployment - Manages Pods
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: 3  # Run 3 instances
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: backend
        image: myapp:1.0.0
        ports:
        - containerPort: 4000
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: url

---
# Service - Load balancer
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  type: LoadBalancer
  selector:
    app: backend
  ports:
  - port: 80
    targetPort: 4000
```

### Key Kubernetes Concepts
```
Pod:
- Smallest deployable unit
- Can have one or more containers
- Shares network and storage

Deployment:
- Manages Pods
- Handles scaling and updates
- Self-healing

Service:
- Stable network endpoint
- Load balances across Pods
- Service discovery

ConfigMap:
- Store configuration
- Non-sensitive data

Secret:
- Store sensitive data
- Encrypted at rest

Ingress:
- HTTP/HTTPS routing
- SSL termination
- Virtual hosting
```

---

## Interview Questions

### Basic Questions

**Q1: What is Docker and why use it?**
```
Docker is a containerization platform that packages applications with their dependencies.

Benefits:
1. Consistency
   - Same environment everywhere
   - "Works on my machine" solved

2. Isolation
   - Applications don't interfere
   - Clean dependencies

3. Portability
   - Run anywhere (cloud, local, etc.)

4. Efficiency
   - Lightweight (vs VMs)
   - Fast startup

5. Scalability
   - Easy to scale up/down
```

**Q2: Difference between Docker Image and Container?**
```
Image:
- Template/Blueprint
- Read-only
- Built from Dockerfile
- Stored in registry
- Example: node:18-alpine

Container:
- Running instance of image
- Has its own filesystem
- Can be started/stopped
- Temporary (unless volume used)
- Example: Running Node.js app

Analogy:
Image = Class
Container = Object instance
```

**Q3: What is Dockerfile?**
```
Text file with instructions to build Docker image.

Key instructions:
FROM   - Base image
COPY   - Copy files
RUN    - Execute commands
ENV    - Set environment variables
EXPOSE - Document port
CMD    - Default command

Example:
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
CMD ["node", "app.js"]
```

### Advanced Questions

**Q4: Explain Docker networking**
```
Types:
1. Bridge (default)
   - Isolated network
   - Containers communicate by name
   - docker network create mynetwork

2. Host
   - Use host's network
   - No isolation
   - Better performance

3. None
   - No networking
   - Complete isolation

Container Communication:
services:
  backend:
    networks: [app-net]
    environment:
      DB_HOST: db  # Use service name

  db:
    networks: [app-net]

networks:
  app-net:
```

**Q5: How do you optimize Docker images?**
```
1. Use Alpine base images
   FROM node:18-alpine  # 40MB vs 180MB

2. Multi-stage builds
   Build stage: All tools
   Final stage: Only runtime

3. Order instructions by change frequency
   COPY package.json (changes rarely)
   COPY . (changes often)

4. Minimize layers
   RUN apt-get update && apt-get install && rm -rf /var/lib/apt/lists/*

5. Use .dockerignore
   node_modules
   .git
   *.md

6. Use BuildKit caching
   RUN --mount=type=cache,target=/root/.npm npm install

Result:
Before: 500MB
After: 95MB
```

**Q6: Explain multi-stage builds**
```dockerfile
# Build stage - has all dev tools
FROM node:18 AS build
WORKDIR /app
COPY package*.json ./
RUN npm install  # All dependencies
COPY . .
RUN npm run build  # TypeScript compilation

# Production stage - minimal
FROM node:18-alpine AS production
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production  # Only prod deps
COPY --from=build /app/dist ./dist
USER node
CMD ["node", "dist/main.js"]

Benefits:
- Final image only has runtime needs
- Smaller size (80% reduction)
- Better security (no build tools)
```

**Q7: How do you handle secrets in Docker?**
```
1. Environment Variables (Basic)
docker run -e DB_PASSWORD=secret myapp

2. Docker Secrets (Swarm)
docker secret create db_password ./password.txt
docker service create --secret db_password myapp

3. External Secret Management
- AWS Secrets Manager
- HashiCorp Vault
- Azure Key Vault

4. At runtime (Best)
# Fetch from secret store
const secret = await secretManager.getSecret('db-password');

Never:
❌ Hardcode in Dockerfile
❌ Commit .env to git
❌ Build secrets into image
```

**Q8: Docker vs VM?**
```
Virtual Machine:
- Full OS per application
- Heavy (GBs)
- Slow startup (minutes)
- Hypervisor
- Strong isolation
- More resource intensive

Docker Container:
- Share host OS kernel
- Lightweight (MBs)
- Fast startup (seconds)
- Docker Engine
- Process-level isolation
- Efficient resources

When to use:
VM: Need different OS, strong isolation
Docker: Microservices, cloud-native apps
```

---

## Summary

### Docker Workflow
```
1. Write Dockerfile
   ↓
2. Build Image
   docker build -t myapp:1.0 .
   ↓
3. Run Container
   docker run -d -p 3000:3000 myapp:1.0
   ↓
4. Push to Registry
   docker push username/myapp:1.0
   ↓
5. Deploy Anywhere
   docker pull username/myapp:1.0
   docker run myapp:1.0
```

### Key Takeaways
1. **Containers are lightweight**: Share OS kernel, fast startup
2. **Images are immutable**: Version controlled, reproducible
3. **Use multi-stage builds**: Smaller, more secure images
4. **Docker Compose**: Manage multi-container apps easily
5. **Networking**: Containers communicate using service names
6. **Volumes**: Persist data beyond container lifecycle
7. **Security**: Run as non-root, scan for vulnerabilities
8. **Orchestration**: Kubernetes for production at scale

Docker enables consistent, portable, and efficient application deployment! 🐋
