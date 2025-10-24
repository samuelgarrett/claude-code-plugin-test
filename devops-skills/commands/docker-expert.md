# Docker Expert

**Description**: Build efficient, secure Docker containers following best practices

## Core Concepts

- **Image**: Read-only template for containers
- **Container**: Running instance of an image
- **Dockerfile**: Instructions to build an image
- **Registry**: Storage for images (Docker Hub, ECR, etc.)
- **Volume**: Persistent data storage
- **Network**: Container communication

## Dockerfile Best Practices

### Multi-Stage Build
```dockerfile
# ✅ Best practice: Multi-stage build for smaller images
# Stage 1: Build
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

# Stage 2: Production
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
EXPOSE 3000
USER node
CMD ["node", "dist/server.js"]

# ❌ Bad: Single stage with dev dependencies
FROM node:18
WORKDIR /app
COPY . .
RUN npm install  # Installs dev dependencies too
CMD ["node", "server.js"]
```

### Layer Optimization
```dockerfile
# ✅ Good: Optimize layer caching
FROM python:3.11-slim
WORKDIR /app

# Install dependencies first (cached if unchanged)
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code (changes frequently)
COPY . .

# ❌ Bad: Copy everything first
FROM python:3.11-slim
COPY . /app  # Every code change invalidates all layers
RUN pip install -r requirements.txt
```

### Security
```dockerfile
# ✅ Security best practices
FROM node:18-alpine  # Use specific versions and minimal images

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

WORKDIR /app

# Install dependencies
COPY package*.json ./
RUN npm ci --only=production && \
    npm cache clean --force

COPY --chown=nodejs:nodejs . .

# Run as non-root
USER nodejs

EXPOSE 3000
CMD ["node", "server.js"]

# ❌ Bad: Security issues
FROM node:latest  # Don't use 'latest'
COPY . .
RUN npm install
CMD ["node", "server.js"]  # Running as root!
```

## Common Docker Commands

```bash
# Build image
docker build -t myapp:1.0 .
docker build -t myapp:1.0 -f Dockerfile.prod .

# Run container
docker run -d -p 3000:3000 --name myapp myapp:1.0
docker run -it --rm myapp:1.0 sh  # Interactive shell

# Environment variables
docker run -e NODE_ENV=production -e PORT=3000 myapp:1.0

# Volumes
docker run -v $(pwd)/data:/app/data myapp:1.0

# View logs
docker logs myapp
docker logs -f myapp  # Follow logs

# Execute commands in running container
docker exec -it myapp sh
docker exec myapp ls -la

# Stop/start/remove
docker stop myapp
docker start myapp
docker rm myapp
docker rm -f myapp  # Force remove

# Image management
docker images
docker rmi myapp:1.0
docker image prune  # Remove unused images

# Container management
docker ps           # Running containers
docker ps -a        # All containers
docker container prune  # Remove stopped containers
```

## Docker Compose

```yaml
# docker-compose.yml
version: '3.8'

services:
  # Web application
  web:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://db:5432/myapp
    depends_on:
      - db
      - redis
    volumes:
      - ./logs:/app/logs
    networks:
      - app-network
    restart: unless-stopped

  # Database
  db:
    image: postgres:15-alpine
    environment:
      - POSTGRES_DB=myapp
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - app-network
    restart: unless-stopped

  # Cache
  redis:
    image: redis:7-alpine
    networks:
      - app-network
    restart: unless-stopped

volumes:
  db-data:

networks:
  app-network:
    driver: bridge
```

```bash
# Docker Compose commands
docker-compose up -d          # Start services
docker-compose down           # Stop and remove
docker-compose logs -f web    # View logs
docker-compose exec web sh    # Execute command
docker-compose ps             # List services
docker-compose restart web    # Restart service
```

## .dockerignore

```
# .dockerignore - Exclude files from build context
node_modules
npm-debug.log
.git
.gitignore
.env
.env.local
*.md
dist
coverage
.DS_Store
```

## Healthchecks

```dockerfile
# Add healthcheck to Dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
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
  console.log(`STATUS: ${res.statusCode}`);
  process.exit(res.statusCode === 200 ? 0 : 1);
});

request.on('error', (err) => {
  console.log('ERROR', err);
  process.exit(1);
});

request.end();
```

## Best Practices Checklist

- [ ] Use specific image versions (not `latest`)
- [ ] Use minimal base images (alpine)
- [ ] Multi-stage builds for smaller images
- [ ] Optimize layer caching
- [ ] Run as non-root user
- [ ] Use .dockerignore
- [ ] Add healthchecks
- [ ] Set resource limits
- [ ] Use secrets management (not ENV vars for secrets)
- [ ] Scan images for vulnerabilities

## When to Use This Skill

- Containerizing applications
- Creating Dockerfiles
- Setting up multi-container applications
- Optimizing Docker images
- Debugging container issues

---

**Remember**: Small, secure, and efficient containers are the goal!
