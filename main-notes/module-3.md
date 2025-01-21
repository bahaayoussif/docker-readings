# Module 3: Docker Images

## Learning Objectives
By the end of this module, you will be able to:
- Understand Docker images and their layered architecture
- Work with Docker Hub and official images
- Manage local images effectively
- Create basic images using Dockerfile
- Implement image tagging and versioning strategies

## 1. Understanding Docker Images

### What is a Docker Image?
- Read-only template for creating containers
- Contains application code, runtime, libraries, and dependencies
- Built using a Dockerfile
- Composed of multiple layers
- Shared across containers

### Image Layers
- Each instruction in Dockerfile creates a layer
- Layers are cached and reusable
- Only modified layers need to be rebuilt
- Layers are read-only
- Containers add a writable layer on top

### Image Naming Convention
```
registry/repository:tag
# Example: docker.io/nginx:1.19
```

## 2. Working with Docker Hub

### Introduction to Docker Hub
- Official Docker registry
- Public and private repositories
- Official and verified images
- Image vulnerability scanning
- Automated builds

### Basic Docker Hub Commands
```bash
# Login to Docker Hub
docker login

# Search for images
docker search IMAGE_NAME

# Pull an image
docker pull IMAGE_NAME[:TAG]

# Push an image
docker push IMAGE_NAME[:TAG]

# Logout from Docker Hub
docker logout
```

## 3. Managing Docker Images

### Basic Image Commands
```bash
# List local images
docker images
docker image ls

# Remove an image
docker rmi IMAGE_NAME
docker image rm IMAGE_NAME

# Remove all unused images
docker image prune

# Remove all images
docker image prune -a

# Show image history
docker history IMAGE_NAME

# Inspect image details
docker inspect IMAGE_NAME
```

### Image Tags
```bash
# Tag an image
docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]

# Remove a tag
docker rmi IMAGE_NAME:TAG

# List image tags
docker images IMAGE_NAME
```

## 4. Creating Images with Dockerfile

### Basic Dockerfile Instructions
```dockerfile
# Base image
FROM ubuntu:20.04

# Set working directory
WORKDIR /app

# Copy files
COPY . .
ADD source dest

# Run commands
RUN apt-get update && apt-get install -y nodejs
RUN npm install

# Set environment variables
ENV NODE_ENV=production

# Expose ports
EXPOSE 3000

# Define entry point
ENTRYPOINT ["node"]

# Define default commands
CMD ["app.js"]
```

### Building Images
```bash
# Build an image
docker build -t IMAGE_NAME[:TAG] .

# Build with different Dockerfile
docker build -f Dockerfile.dev -t IMAGE_NAME .

# Build with build arguments
docker build --build-arg ARG_NAME=value -t IMAGE_NAME .
```

## 5. Dockerfile Best Practices

### Optimization Techniques
1. Use appropriate base images
2. Minimize number of layers
3. Leverage build cache
4. Clean up in same layer
5. Use .dockerignore file
6. Multi-stage builds for smaller images

### Example Optimized Dockerfile
```dockerfile
# Use specific version
FROM node:16-alpine

# Combine commands to reduce layers
RUN apk add --no-cache python3 make g++ && \
    npm install -g npm@latest

WORKDIR /app

# Copy package files first to leverage cache
COPY package*.json ./
RUN npm ci --only=production

# Copy application code
COPY . .

# Use specific user
USER node

EXPOSE 3000
CMD ["npm", "start"]
```

## 6. Image Tagging Strategies

### Common Tagging Conventions
- Semantic Versioning (1.0.0, 1.0.1)
- Latest tag for most recent version
- Environment tags (dev, staging, prod)
- Git commit hashes
- Date-based tags

### Tagging Examples
```bash
# Version tags
docker tag myapp:latest myapp:1.0.0
docker tag myapp:latest myapp:1.0

# Environment tags
docker tag myapp:latest myapp:prod
docker tag myapp:latest myapp:dev

# Multiple tags during build
docker build -t myapp:latest -t myapp:1.0.0 .
```

## 7. Practical Exercises

### Exercise 1: Working with Docker Hub
1. Search for official Node.js image
2. Pull different versions of Node.js
3. Compare image sizes
4. Remove unused images

### Exercise 2: Creating Custom Images
1. Create a simple web application
2. Write a Dockerfile
3. Build the image
4. Run a container from the image
5. Push the image to Docker Hub

### Exercise 3: Image Optimization
1. Analyze image layers
2. Optimize Dockerfile
3. Compare image sizes
4. Implement multi-stage builds

## 8. Knowledge Check
1. What is a Docker image layer?
2. How does Docker Hub authentication work?
3. What is the purpose of .dockerignore?
4. Explain the difference between CMD and ENTRYPOINT
5. Why are multi-stage builds useful?

## Additional Resources
- [Dockerfile Reference](https://docs.docker.com/engine/reference/builder/)
- [Docker Hub Guide](https://docs.docker.com/docker-hub/)
- [Official Images Guide](https://docs.docker.com/docker-hub/official_images/)
- [Best Practices Guide](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)

## Next Steps
- Learn about multi-stage builds