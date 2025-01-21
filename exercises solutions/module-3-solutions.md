# Module 3: Docker Images - Exercise Solutions

## Exercise 1: Working with Docker Hub

### Task 1: Docker Hub Account Setup
```bash
# Login to Docker Hub
docker login

# Expected output
Login with your Docker ID to push and pull images from Docker Hub.
Username: your_username
Password: 
Login Succeeded
```

### Task 2: Searching and Pulling Images
```bash
# Search for official Node.js images
docker search node

# Pull different Node.js versions
docker pull node:18
docker pull node:18-alpine
docker pull node:16

# List downloaded images
docker images | grep node

# Compare image sizes
docker images --format "table {{.Repository}}:{{.Tag}}\t{{.Size}}" | grep node

# Expected output
node:18           1.1GB
node:18-alpine    175MB
node:16           985MB
```

### Task 3: Image Tags and Versioning
```bash
# Tag an existing image
docker tag node:18 myregistry/node:latest
docker tag node:18 myregistry/node:v18

# List images with new tags
docker images | grep node

# Push images to Docker Hub (replace 'myregistry' with your username)
docker push myregistry/node:latest
docker push myregistry/node:v18
```

## Exercise 2: Image Management

### Task 1: Basic Image Operations
```bash
# List all images
docker images

# Remove specific image
docker rmi node:16

# Remove image by ID
docker rmi abc123def456

# Remove all unused images
docker image prune

# Remove all unused images including unused containers
docker image prune -a

# Force remove all images
docker rmi -f $(docker images -q)
```

### Task 2: Image History and Inspection
```bash
# View image history
docker history node:18

# Inspect image details
docker inspect node:18

# Filter specific information
docker inspect --format='{{.Config.Env}}' node:18
docker inspect --format='{{.Config.ExposedPorts}}' node:18

# Save image to tar file
docker save -o node18.tar node:18

# Load image from tar file
docker load -i node18.tar
```

### Task 3: Multi-platform Images
```bash
# Check image platforms
docker inspect --format='{{.Architecture}}' node:18

# Pull specific platform image
docker pull --platform linux/arm64 node:18
docker pull --platform linux/amd64 node:18

# List images with platforms
docker images --format "table {{.Repository}}:{{.Tag}}\t{{.OS}}/{{.Architecture}}"
```

## Exercise 3: Creating Basic Images

### Task 1: Simple Web Application
```bash
# Create project directory
mkdir simple-web-app
cd simple-web-app

# Create index.html
cat > index.html << EOF
<!DOCTYPE html>
<html>
<head>
    <title>Simple Web App</title>
</head>
<body>
    <h1>Hello Docker!</h1>
</body>
</html>
EOF

# Create Dockerfile
cat > Dockerfile << EOF
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/
EXPOSE 80
EOF

# Build image
docker build -t simple-web:v1 .

# Run container
docker run -d -p 8080:80 simple-web:v1

# Test
curl http://localhost:8080
```

### Task 2: Node.js Application
```bash
# Create project directory
mkdir node-app
cd node-app

# Create package.json
cat > package.json << EOF
{
  "name": "docker-node-app",
  "version": "1.0.0",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  }
}
EOF

# Create server.js
cat > server.js << EOF
const http = require('http');
const server = http.createServer((req, res) => {
  res.writeHead(200, {'Content-Type': 'text/plain'});
  res.end('Hello from Node.js!');
});
server.listen(3000);
EOF

# Create Dockerfile
cat > Dockerfile << EOF
FROM node:18-alpine
WORKDIR /app
COPY package.json server.js ./
EXPOSE 3000
CMD ["node", "server.js"]
EOF

# Build image
docker build -t node-app:v1 .

# Run container
docker run -d -p 3000:3000 node-app:v1

# Test
curl http://localhost:3000
```

### Task 3: Python Application
```bash
# Create project directory
mkdir python-app
cd python-app

# Create requirements.txt
cat > requirements.txt << EOF
flask==2.0.1
EOF

# Create app.py
cat > app.py << EOF
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello():
    return 'Hello from Python Flask!'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
EOF

# Create Dockerfile
cat > Dockerfile << EOF
FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY app.py .
EXPOSE 5000
CMD ["python", "app.py"]
EOF

# Build image
docker build -t python-app:v1 .

# Run container
docker run -d -p 5000:5000 python-app:v1

# Test
curl http://localhost:5000
```

## Exercise 4: Image Optimization

### Task 1: Multi-stage Builds
```bash
# Create React application
cat > Dockerfile << EOF
# Build stage
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine
COPY --from=builder /app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
EOF

# Build optimized image
docker build -t react-app:optimized .

# Compare sizes
docker images
```

### Task 2: Layer Optimization
```bash
# Original Dockerfile
cat > Dockerfile.original << EOF
FROM ubuntu:20.04
RUN apt-get update
RUN apt-get install -y python3
RUN apt-get install -y python3-pip
RUN pip install flask
COPY app.py .
CMD ["python3", "app.py"]
EOF

# Optimized Dockerfile
cat > Dockerfile.optimized << EOF
FROM ubuntu:20.04
RUN apt-get update && \
    apt-get install -y python3 python3-pip && \
    pip install flask && \
    rm -rf /var/lib/apt/lists/*
COPY app.py .
CMD ["python3", "app.py"]
EOF

# Build both images
docker build -f Dockerfile.original -t app:original .
docker build -f Dockerfile.optimized -t app:optimized .

# Compare sizes
docker images
```

### Task 3: Using .dockerignore
```bash
# Create .dockerignore
cat > .dockerignore << EOF
node_modules
npm-debug.log
Dockerfile
.dockerignore
.git
.gitignore
README.md
coverage
build
.env
EOF

# Build image with and without .dockerignore
# Compare build times and image sizes
```

## Common Issues and Solutions

### Issue 1: Image Build Failures
```bash
# Check build context
docker build --no-cache .

# Debug layer creation
docker build --progress=plain .

# Check available disk space
docker system df
```

### Issue 2: Pull/Push Issues
```bash
# Re-authenticate with registry
docker logout
docker login

# Check registry configuration
docker info | grep Registry
```

### Issue 3: Image Size Problems
```bash
# Analyze image layers
docker history image_name

# Use multi-stage builds
# Implement layer optimization
# Clean up in same layer
```

## Best Practices

1. **Image Building**
   - Use specific base image versions
   - Minimize layer count
   - Clean up in same layer
   - Use .dockerignore

2. **Image Tagging**
   - Use semantic versioning
   - Never use only 'latest' tag
   - Include build information
   - Use descriptive tags

3. **Security**
   - Scan images for vulnerabilities
   - Use minimal base images
   - Don't store secrets in images
   - Keep base images updated

4. **Optimization**
   - Implement multi-stage builds
   - Optimize layer caching
   - Remove unnecessary files
   - Use appropriate base images

## Next Steps
After completing these exercises, you should:
1. Understand Docker image concepts
2. Be able to work with Docker Hub
3. Know how to build optimized images
4. Understand image management

Ready to move on to Module 4 to learn about:
- Advanced Dockerfile concepts
- Multi-stage builds
- Security best practices
- Production considerations