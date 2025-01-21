# Module 4: Building Custom Images - Exercise Solutions

## Exercise 1: Multi-stage Builds

### Task 1: React Application with Multi-stage Build
```bash
# Create React project directory
mkdir react-app
cd react-app

# Create package.json
cat > package.json << EOF
{
  "name": "react-app",
  "version": "0.1.0",
  "private": true,
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "react-scripts": "5.0.1"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build"
  }
}
EOF

# Create multi-stage Dockerfile
cat > Dockerfile << EOF
# Build stage
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine
COPY --from=builder /app/build /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
EOF

# Create nginx configuration
cat > nginx.conf << EOF
server {
    listen 80;
    location / {
        root /usr/share/nginx/html;
        index index.html;
        try_files $uri $uri/ /index.html;
    }
}
EOF

# Build image
docker build -t react-app:prod .

# Run container
docker run -d -p 80:80 react-app:prod
```

### Task 2: Go Application with Multi-stage Build
```bash
# Create Go project
mkdir go-app
cd go-app

# Create main.go
cat > main.go << EOF
package main

import (
    "fmt"
    "net/http"
)

func main() {
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "Hello from Go!")
    })
    http.ListenAndServe(":8080", nil)
}
EOF

# Create multi-stage Dockerfile
cat > Dockerfile << EOF
# Build stage
FROM golang:1.19-alpine AS builder
WORKDIR /app
COPY go.* ./
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o main .

# Production stage
FROM alpine:3.14
RUN apk --no-cache add ca-certificates
COPY --from=builder /app/main /app/main
EXPOSE 8080
CMD ["/app/main"]
EOF

# Build and run
docker build -t go-app:prod .
docker run -d -p 8080:8080 go-app:prod
```

## Exercise 2: Language-Specific Optimizations

### Task 1: Python Application with Poetry
```bash
# Create Python project
mkdir python-app
cd python-app

# Create pyproject.toml
cat > pyproject.toml << EOF
[tool.poetry]
name = "python-app"
version = "0.1.0"
description = "Python application with Poetry"

[tool.poetry.dependencies]
python = "^3.9"
fastapi = "^0.68.0"
uvicorn = "^0.15.0"

[build-system]
requires = ["poetry-core>=1.0.0"]
build-backend = "poetry.core.masonry.api"
EOF

# Create optimized Dockerfile
cat > Dockerfile << EOF
FROM python:3.9-slim as builder

RUN pip install poetry
WORKDIR /app
COPY pyproject.toml poetry.lock* ./
RUN poetry export -f requirements.txt --output requirements.txt --without-hashes

FROM python:3.9-slim
WORKDIR /app
COPY --from=builder /app/requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
EOF

# Build and run
docker build -t python-app:prod .
docker run -d -p 8000:8000 python-app:prod
```

### Task 2: Node.js Application with Yarn
```bash
# Create Node.js project
mkdir node-app
cd node-app

# Create package.json
cat > package.json << EOF
{
  "name": "node-app",
  "version": "1.0.0",
  "private": true,
  "dependencies": {
    "express": "^4.17.1"
  }
}
EOF

# Create optimized Dockerfile
cat > Dockerfile << EOF
FROM node:18-alpine AS builder
WORKDIR /app
COPY package.json yarn.lock* ./
RUN yarn install --frozen-lockfile
COPY . .

FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .
USER node
CMD ["node", "server.js"]
EOF

# Build and run
docker build -t node-app:prod .
docker run -d -p 3000:3000 node-app:prod
```

## Exercise 3: Security Implementation

### Task 1: Secure Node.js Application
```bash
# Create secure Dockerfile
cat > Dockerfile << EOF
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .

FROM node:18-alpine
RUN apk add --no-cache dumb-init
ENV NODE_ENV production
WORKDIR /app
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nodejs -u 1001
COPY --from=builder --chown=nodejs:nodejs /app .
USER nodejs
EXPOSE 3000
ENTRYPOINT ["dumb-init", "--"]
CMD ["node", "server.js"]
EOF

# Build with security scanning
docker build -t secure-node-app:prod .
docker scout cves secure-node-app:prod
```

### Task 2: Security Hardening
```bash
# Create security-focused Dockerfile
cat > Dockerfile << EOF
FROM alpine:3.14 AS builder
RUN apk add --no-cache ca-certificates

FROM scratch
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
COPY app /app
USER 1000:1000
ENTRYPOINT ["/app"]
EOF

# Add security labels
docker build -t hardened-app:prod \
  --label "org.opencontainers.image.authors=security@example.com" \
  --label "org.opencontainers.image.vendor=ACME Secure" \
  --label "org.opencontainers.image.title=Hardened App" \
  .
```

## Exercise 4: Image Optimization Techniques

### Task 1: Layer Optimization
```bash
# Create optimized Dockerfile
cat > Dockerfile << EOF
# Bad practice (multiple layers)
FROM ubuntu:20.04
RUN apt-get update
RUN apt-get install -y python3
RUN apt-get install -y python3-pip
RUN pip install flask
RUN apt-get clean
RUN rm -rf /var/lib/apt/lists/*

# Good practice (single layer)
FROM ubuntu:20.04
RUN apt-get update && \
    apt-get install -y python3 python3-pip && \
    pip install flask && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
EOF

# Compare builds
docker build -t app:unoptimized -f Dockerfile.unoptimized .
docker build -t app:optimized -f Dockerfile.optimized .
docker images
```

### Task 2: Build Cache Optimization
```bash
# Create cache-optimized Dockerfile
cat > Dockerfile << EOF
FROM node:18-alpine

# Dependencies layer
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Application layer
COPY . .
CMD ["npm", "start"]
EOF

# Use BuildKit cache mount
docker build --build-arg BUILDKIT_INLINE_CACHE=1 -t app:cached .
```

## Common Issues and Solutions

### Issue 1: Build Context Size
```bash
# Create .dockerignore
cat > .dockerignore << EOF
node_modules
npm-debug.log
Dockerfile
.dockerignore
.git
.env
*.md
tests
coverage
EOF

# Check context size
docker build --no-cache .
```

### Issue 2: Layer Cache Issues
```bash
# Force rebuild without cache
docker build --no-cache .

# View build cache usage
docker build --progress=plain .
```

## Best Practices

1. **Multi-stage Builds**
   - Separate build and runtime stages
   - Use appropriate base images
   - Copy only necessary artifacts

2. **Security**
   - Run as non-root user
   - Use minimal base images
   - Implement security scanning
   - Update dependencies regularly

3. **Optimization**
   - Minimize layer count
   - Optimize cache usage
   - Remove unnecessary files
   - Use appropriate base images

4. **Image Size**
   - Use alpine/slim base images
   - Clean up in same layer
   - Remove unnecessary dependencies
   - Implement multi-stage builds

## Next Steps
After completing these exercises, proceed to:
- Container networking
- Volume management
- Docker Compose
- Container orchestration