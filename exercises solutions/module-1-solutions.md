# Module 1: Understanding Container Basics - Exercise Solutions

## Exercise 1: Basic Docker Commands

### Task 1: Check Docker Version and Info
```bash
# Check Docker version
docker --version

# Expected output
Docker version 24.0.7, build afdd53b

# View detailed Docker information
docker info

# Expected output
Client:
 Context:    default
 Debug Mode: false
 Plugins:
  buildx: Docker Buildx (Docker Inc.)
  compose: Docker Compose (Docker Inc.)
  dev: Docker Dev Environments (Docker Inc.)
  extension: Manages Docker extensions (Docker Inc.)
  init: Creates Docker-related starter files (Docker Inc.)
  sbom: View the packaged-based Software Bill Of Materials (SBOM) for an image
  scan: Docker Scan (Docker Inc.)
  scout: Command line tool for Docker Scout (Docker Inc.)

Server:
 Containers: 12
  Running: 2
  Paused: 0
  Stopped: 10
 Images: 35
 Server Version: 24.0.7
 Storage Driver: overlay2
 ...
```

### Task 2: Run Hello World Container
```bash
# Run hello-world container
docker run hello-world

# Expected output
Hello from Docker!
This message shows that your installation appears to be working correctly.
...

# List all containers including stopped ones
docker ps -a

# Expected output
CONTAINER ID   IMAGE         COMMAND    CREATED         STATUS                     PORTS     NAMES
abc123def456   hello-world   "/hello"   2 minutes ago   Exited (0) 2 minutes ago            wonderful_wilson
```

### Task 3: Clean Up
```bash
# Remove the hello-world container
docker rm $(docker ps -a -q --filter "ancestor=hello-world")

# Remove the hello-world image
docker rmi hello-world

# Verify removal
docker ps -a
docker images
```

## Exercise 2: Working with a Simple Web Server

### Task 1: Run Nginx Container
```bash
# Run nginx container in detached mode with port mapping
docker run -d -p 8080:80 --name my-nginx nginx

# Verify container is running
docker ps

# Expected output
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                  NAMES
xyz789abc123   nginx     "/docker-entrypoint.…"   1 minute ago    Up 1 minute    0.0.0.0:8080->80/tcp   my-nginx
```

### Task 2: Test the Web Server
```bash
# Using curl to test (or visit http://localhost:8080 in browser)
curl http://localhost:8080

# Expected output
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
</html>
```

### Task 3: Container Management
```bash
# Stop the nginx container
docker stop my-nginx

# Start it again
docker start my-nginx

# View container logs
docker logs my-nginx

# Remove container
docker stop my-nginx
docker rm my-nginx
```

## Exercise 3: Container Resource Usage

### Task 1: Run Resource-Limited Container
```bash
# Run nginx with memory and CPU limits
docker run -d --name limited-nginx \
  --memory="512m" \
  --cpus="1.0" \
  -p 8081:80 \
  nginx

# Verify container is running with limits
docker inspect limited-nginx | grep -A 10 "HostConfig"

# Expected output shows resource constraints
```

### Task 2: Monitor Resource Usage
```bash
# View container statistics
docker stats limited-nginx

# Expected output
CONTAINER ID   NAME            CPU %     MEM USAGE / LIMIT   MEM %     NET I/O       BLOCK I/O     PIDS
abc123def456   limited-nginx   0.00%     2.789MB / 512MB     0.54%     656B / 656B   0B / 0B       2
```

### Task 3: Clean Up Resources
```bash
# Stop and remove container
docker stop limited-nginx
docker rm limited-nginx

# Remove nginx image
docker rmi nginx
```

## Exercise 4: Container Shell Access

### Task 1: Interactive Container
```bash
# Run Ubuntu container interactively
docker run -it --name my-ubuntu ubuntu

# Inside container, run some commands
whoami
pwd
ls -la
apt update
apt install -y curl
curl --version

# Exit container
exit
```

### Task 2: Container Restart and Attach
```bash
# Start stopped container
docker start my-ubuntu

# Attach to container
docker attach my-ubuntu

# Or execute specific command
docker exec -it my-ubuntu bash
```

### Task 3: Clean Up
```bash
# Stop and remove container
docker stop my-ubuntu
docker rm my-ubuntu

# Remove Ubuntu image
docker rmi ubuntu
```

## Exercise 5: System Maintenance

### Task 1: Review System Usage
```bash
# View disk usage
docker system df

# Expected output
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          5         2         1.84GB    1.22GB (66%)
Containers      3         1         165.7MB   165.4MB (99%)
Local Volumes   2         1         428.3MB   214.1MB (50%)
Build Cache     0         0         0B        0B
```

### Task 2: Clean Up System
```bash
# Remove all stopped containers
docker container prune

# Remove unused images
docker image prune

# Remove unused volumes
docker volume prune

# Remove all unused objects (use with caution)
docker system prune -a --volumes
```

### Task 3: Verify Cleanup
```bash
# Check system status after cleanup
docker system df

# List remaining containers
docker ps -a

# List remaining images
docker images

# List remaining volumes
docker volume ls
```

## Common Issues and Solutions

### Issue 1: Permission Denied
```bash
# If you get permission errors, add user to docker group
sudo usermod -aG docker $USER
newgrp docker

# Verify group membership
groups
```

### Issue 2: Port Already in Use
```bash
# Check what's using the port
sudo lsof -i :8080

# Use a different port
docker run -d -p 8081:80 nginx
```

### Issue 3: Container Won't Start
```bash
# Check container logs
docker logs container_name

# Inspect container
docker inspect container_name

# Common fixes
docker container prune  # Remove stopped containers
docker system prune     # Clean up system
```

## Best Practices

1. **Resource Management**
   - Always set memory limits
   - Monitor resource usage
   - Clean up unused resources regularly

2. **Container Naming**
   - Use meaningful names
   - Avoid default names
   - Follow naming conventions

3. **Security**
   - Don't run as root
   - Use official images
   - Keep images updated

4. **Monitoring**
   - Check logs regularly
   - Monitor resource usage
   - Set up alerts for issues

## Next Steps
After completing these exercises, you should:
1. Understand basic Docker commands
2. Be able to manage containers
3. Know how to monitor resources
4. Be comfortable with container interaction

Move on to Module 2 to learn about:
- Advanced Docker commands
- Container networking
- Volume management
- Docker Compose basics