# Module 4: Building Custom Images

## Learning Objectives
By the end of this module, you will be able to:
- Write efficient Dockerfiles for different applications
- Implement multi-stage builds
- Create optimized images for various programming languages
- Apply security best practices
- Understand and implement image optimization techniques

## 1. Advanced Dockerfile Concepts

### Multi-stage Builds
```dockerfile
# Build stage
FROM node:16 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Production stage
FROM node:16-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY package*.json ./
RUN npm install --production
CMD ["npm", "start"]
```

### Build Arguments and Environment Variables
```dockerfile
# Build arguments
ARG NODE_VERSION=16
FROM node:${NODE_VERSION}

# Build-time configuration
ARG BUILD_ENV=production
RUN if [ "$BUILD_ENV" = "development" ] ; then npm install ; else npm ci --only=production ; fi

# Runtime environment variables
ENV APP_ENV=production
ENV PORT=3000
```

## 2. Language-Specific Dockerfiles

### Node.js Application
```dockerfile
FROM node:16-alpine

# Install dependencies only when needed
COPY package*.json ./
RUN npm ci --only=production

# Copy app source
COPY . .

# Use non-root user
USER node

EXPOSE 3000
CMD ["node", "server.js"]
```

### Python Application
```dockerfile
FROM python:3.9-slim

WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application
COPY . .

CMD ["python", "app.py"]
```

### Java Application
```dockerfile
FROM eclipse-temurin:17-jdk-alpine as builder
WORKDIR /app
COPY . .
RUN ./gradlew build

FROM eclipse-temurin:17-jre-alpine
COPY --from=builder /app/build/libs/*.jar app.jar
ENTRYPOINT ["java", "-jar", "app.jar"]
```

## 3. Image Optimization Techniques

### Minimizing Layer Size
1. Combine RUN commands
```dockerfile
# Bad
RUN apt-get update
RUN apt-get install -y package1
RUN apt-get install -y package2

# Good
RUN apt-get update && \
    apt-get install -y \
    package1 \
    package2 \
    && rm -rf /var/lib/apt/lists/*
```

2. Use .dockerignore
```plaintext
# .dockerignore
node_modules
npm-debug.log
Dockerfile
.dockerignore
.git
.gitignore
README.md
```

3. Clean up in same layer
```dockerfile
RUN wget https://example.com/big-file.tar.gz \
    && tar -xzf big-file.tar.gz \
    && rm big-file.tar.gz
```

### Layer Caching Strategies
1. Order instructions from least to most frequently changing
2. Separate build-time dependencies
3. Use cache mounts for package managers

```dockerfile
# Use BuildKit cache mount
RUN --mount=type=cache,target=/var/cache/apt \
    apt-get update && apt-get install -y package1
```

## 4. Security Best Practices

### Run as Non-Root User
```dockerfile
# Create user and group
RUN groupadd -r appuser && useradd -r -g appuser appuser

# Set ownership
COPY --chown=appuser:appuser . .

# Switch to non-root user
USER appuser
```

### Minimize Attack Surface
```dockerfile
# Use specific versions
FROM node:16.14-alpine3.14

# Remove unnecessary tools
RUN apk add --no-cache python3 && \
    rm -rf /var/cache/apk/*

# Use read-only root filesystem
CMD ["node", "--read-only-root-fs", "app.js"]
```

### Scan for Vulnerabilities
```bash
# Scan image using Docker Scout
docker scout cves image:tag

# Use security-focused base images
FROM gcr.io/distroless/nodejs
```

## 5. Multi-Architecture Images

### Building for Multiple Platforms
```bash
# Create buildx builder
docker buildx create --use

# Build and push multi-arch image
docker buildx build --platform linux/amd64,linux/arm64 -t username/image:tag --push .
```

### Platform-Specific Instructions
```dockerfile
FROM --platform=$BUILDPLATFORM node:16-alpine

ARG TARGETPLATFORM
ARG BUILDPLATFORM

RUN echo "Building on $BUILDPLATFORM for $TARGETPLATFORM"
```

## 6. Testing Docker Builds

### Build Testing
```bash
# Test build with different arguments
docker build --build-arg ENV=test .

# Verify build outputs
docker build -t test-image . && docker run test-image npm test
```

### Integration Testing
```yaml
# docker-compose.test.yml
version: '3.8'
services:
  sut:
    build: .
    command: npm test
    environment:
      - NODE_ENV=test
```

## 7. Practical Exercises

### Exercise 1: Multi-stage Build
1. Create a React application
2. Write a multi-stage Dockerfile
3. Compare image sizes with and without multi-stage build
4. Implement build arguments

### Exercise 2: Optimization
1. Analyze current image size
2. Implement layer optimization
3. Add .dockerignore
4. Measure improvement

### Exercise 3: Security Implementation
1. Add non-root user
2. Implement security best practices
3. Scan image for vulnerabilities
4. Fix identified issues

## 8. Knowledge Check
1. Why are multi-stage builds important?
2. How does layer caching work?
3. What are the key security considerations?
4. How do you optimize Dockerfile for different languages?
5. What is the purpose of BuildKit cache mounts?

## Additional Resources
- [BuildKit Documentation](https://docs.docker.com/build/buildkit/)
- [Multi-stage Builds Guide](https://docs.docker.com/build/building/multi-stage/)
- [Security Best Practices](https://docs.docker.com/develop/security-best-practices/)
- [Docker Scout](https://docs.docker.com/scout/)

## Next Steps
- Learn about Docker networking
- Understand container communication
- Explore Docker Compose for multi-container applications