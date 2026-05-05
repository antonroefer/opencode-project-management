---
name: docker-basics
description: Containerization with Docker and Docker Compose. Includes Dockerfile best practices, multi-stage builds, volume management, networking, and common patterns for development and production. Use when containerizing applications, setting up local dev environments with Docker, creating Dockerfiles, or when the user mentions Docker, container, Docker Compose, or containerization.
---

# Docker Basics Skill

Containerize applications effectively using Docker and Docker Compose with best practices.

## When to Use
- Containerizing an application for deployment
- Setting up local development environment
- Creating reproducible builds
- Managing multi-service applications
- When the user mentions Docker, containers, or Docker Compose

## Dockerfile Best Practices

### Basic Structure
```dockerfile
# Use specific version, not 'latest'
FROM python:3.12-slim

# Set working directory
WORKDIR /app

# Copy dependency files first (leverage cache)
COPY requirements.txt .

# Install dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Non-root user for security
RUN useradd -m -u 1000 appuser && chown -R appuser:appuser /app
USER appuser

# Expose port
EXPOSE 8000

# Run the application
CMD ["python", "main.py"]
```

### Multi-Stage Builds (Reduce Image Size)
```dockerfile
# Build stage
FROM node:20 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM node:20-slim
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/package*.json ./
RUN npm ci --production
EXPOSE 3000
CMD ["node", "dist/server.js"]
```

### Best Practices Checklist
- [ ] Use `.dockerignore` to exclude unnecessary files
- [ ] Don't run as root user
- [ ] Use specific tags, not `latest`
- [ ] Combine RUN commands to reduce layers
- [ ] Use multi-stage builds for compiled languages
- [ ] Don't store secrets in images
- [ ] Clean up package manager caches

### `.dockerignore`
```
node_modules/
__pycache__/
*.pyc
.git/
.env
.DS_Store
*.log
dist/
build/
```

## Docker Compose

### Basic `docker-compose.yml`
```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgres://user:pass@db:5432/mydb
    volumes:
      - .:/app
    depends_on:
      - db
    restart: unless-stopped

  db:
    image: postgres:16
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: mydb
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

volumes:
  postgres_data:
```

### Common Commands
```bash
# Start services
docker compose up -d

# View logs
docker compose logs -f app

# Stop services
docker compose down

# Rebuild after changes
docker compose up -d --build

# Execute command in container
docker compose exec app python manage.py migrate

# Clean up
docker compose down -v  # Also remove volumes
```

## Development Patterns

### Hot-Reload Development
```yaml
# docker-compose.dev.yml
services:
  app:
    volumes:
      - .:/app  # Live code mounting
    command: npm run dev  # Dev server with hot reload
    environment:
      - NODE_ENV=development
```

### Production vs Development
```bash
# Development
docker compose -f docker-compose.yml -f docker-compose.dev.yml up

# Production
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

## Volume Management

### Types of Volumes
```bash
# Named volume (managed by Docker)
docker volume create mydata
docker run -v mydata:/data myimage

# Bind mount (host directory)
docker run -v /host/path:/container/path myimage

# Anonymous volume (ephemeral)
docker run -v /data myimage
```

### When to Use Volumes
- **Named volumes**: Persistent data (databases, user uploads)
- **Bind mounts**: Development code (live reload)
- **Anonymous volumes**: Temporary data, caches

## Networking

### Default Network
```yaml
services:
  app:
    # Automatically joins default network
    # Accessible by service name: http://db:5432
```

### Custom Network
```yaml
networks:
  frontend:
  backend:

services:
  app:
    networks:
      - frontend
      - backend
  db:
    networks:
      - backend  # Isolated from frontend
```

## Common Patterns

### Database Migrations
```yaml
services:
  app:
    depends_on:
      db:
        condition: service_healthy
    command: >
      sh -c "python manage.py migrate &&
             python manage.py collectstatic --noinput &&
             gunicorn app.wsgi:application"
```

### Health Checks
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8000/health || exit 1
```

## Verification

After setting up Docker:
1. Test build: `docker build -t myapp .`
2. Run container: `docker run -p 8000:8000 myapp`
3. Check logs: `docker logs <container-id>`
4. Test health endpoint: `curl http://localhost:8000/health`
5. Verify volume persistence: restart container, check data
6. Run `docker compose config` to validate compose file
