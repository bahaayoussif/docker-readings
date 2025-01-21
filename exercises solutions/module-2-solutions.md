# Module 2: Working with Docker Commands - Exercise Solutions

## Exercise 1: Container Management

### Task 1: Running Multiple Containers
```bash
# Run first nginx container
docker run -d -p 8081:80 --name web1 nginx

# Run second nginx container
docker run -d -p 8082:80 --name web2 nginx

# Run MongoDB container
docker run -d -p 27017:27017 --name db1 mongo

# List all running containers
docker ps

# Expected output
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS                     NAMES
abc123def456   nginx     "/docker-entrypoint.…"   30 seconds ago   Up 29 seconds   0.0.0.0:8081->80/tcp      web1
def456ghi789   nginx     "/docker-entrypoint.…"   25 seconds ago   Up 24 seconds   0.0.0.0:8082->80/tcp      web2
ghi789jkl012   mongo     "docker-entrypoint.s…"   20 seconds ago   Up 19 seconds   0.0.0.0:27017->27017/tcp  db1
```

### Task 2: Container Operations
```bash
# Stop first nginx container
docker stop web1

# Restart MongoDB container
docker restart db1

# Pause second nginx container
docker pause web2

# Unpause second nginx container
docker unpause web2

# View only running containers
docker ps --filter status=running

# View all containers including stopped ones
docker ps -a
```

### Task 3: Container Cleanup
```bash
# Remove stopped container
docker rm web1

# Force remove running container
docker rm -f web2

# Remove all containers
docker rm -f $(docker ps -aq)

# Verify all containers are removed
docker ps -a
```

## Exercise 2: Container Logs and Inspection

### Task 1: Generate and View Logs
```bash
# Run nginx container with name
docker run -d --name webserver nginx

# Generate some log entries
curl http://localhost:80
curl http://localhost:80/nonexistent

# View all logs
docker logs webserver

# Follow log output
docker logs -f webserver

# Show last 5 log entries
docker logs --tail 5 webserver

# Show logs with timestamps
docker logs -t webserver
```

### Task 2: Container Inspection
```bash
# Inspect container details
docker inspect webserver

# Filter specific information
docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' webserver

# View container stats
docker stats webserver

# View running processes
docker top webserver
```

### Task 3: Container Events
```bash
# Watch container events in one terminal
docker events

# In another terminal, perform operations
docker stop webserver
docker start webserver
docker restart webserver
```

## Exercise 3: Working with Container Shell

### Task 1: Interactive Shell Access
```bash
# Run Ubuntu container interactively
docker run -it --name ubuntu1 ubuntu

# Inside container:
apt update
apt install -y curl wget
curl --version
exit

# Restart and reattach
docker start ubuntu1
docker attach ubuntu1
```

### Task 2: Execute Commands in Running Container
```bash
# Run nginx container
docker run -d --name nginx1 nginx

# Execute commands in container
docker exec nginx1 ls -la /etc/nginx
docker exec nginx1 nginx -v
docker exec nginx1 ps aux

# Start interactive shell in running container
docker exec -it nginx1 bash

# Inside container:
ls -la
ps aux
nginx -t
exit
```

### Task 3: Container Environment
```bash
# Run container with environment variables
docker run -d --name app1 \
  -e "DB_HOST=db.example.com" \
  -e "DB_PORT=5432" \
  nginx

# View environment variables
docker exec app1 env

# Add new environment variable to running container
docker exec -it app1 bash -c "export NEW_VAR=value && env"
```

## Exercise 4: Resource Management

### Task 1: Resource Constraints
```bash
# Run container with memory limits
docker run -d --name limited_nginx \
  --memory="256m" \
  --memory-swap="512m" \
  --cpus="0.5" \
  nginx

# Verify resource limits
docker inspect --format='{{.HostConfig.Memory}}' limited_nginx
docker inspect --format='{{.HostConfig.NanoCpus}}' limited_nginx
```

### Task 2: Resource Monitoring
```bash
# Monitor single container
docker stats limited_nginx

# Monitor all containers
docker stats

# Get specific container metrics
docker stats --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}" limited_nginx
```

### Task 3: Update Resource Limits
```bash
# Update container resources
docker update --memory="512m" --cpus="1.0" limited_nginx

# Verify changes
docker inspect --format='{{.HostConfig.Memory}}' limited_nginx
```

## Exercise 5: Container Networking Basics

### Task 1: Network Inspection
```bash
# List networks
docker network ls

# Inspect bridge network
docker network inspect bridge

# Create new network
docker network create mynetwork

# Run container in custom network
docker run -d --name nettest1 --network mynetwork nginx
```

### Task 2: Container Communication
```bash
# Run two containers in same network
docker run -d --name web1 --network mynetwork nginx
docker run -d --name web2 --network mynetwork nginx

# Test communication between containers
docker exec web1 ping web2

# Connect container to multiple networks
docker network connect bridge web1
```

### Task 3: Network Cleanup
```bash
# Disconnect container from network
docker network disconnect mynetwork web1

# Remove network
docker network rm mynetwork

# Remove all unused networks
docker network prune
```

## Common Issues and Solutions

### Issue 1: Container Won't Stop
```bash
# Force stop container
docker kill container_name

# If still running
docker rm -f container_name
```

### Issue 2: Container Keeps Restarting
```bash
# Check container logs
docker logs container_name

# Check restart policy
docker inspect --format='{{.HostConfig.RestartPolicy}}' container_name

# Update restart policy
docker update --restart=no container_name
```

### Issue 3: Can't Execute Commands
```bash
# Check if container is running
docker ps

# Check container status
docker inspect --format='{{.State.Status}}' container_name

# Restart container if needed
docker restart container_name
```

## Best Practices

1. **Container Management**
   - Use meaningful container names
   - Always clean up unused containers
   - Monitor resource usage
   - Use restart policies appropriately

2. **Logging**
   - Use log rotation
   - Implement structured logging
   - Monitor log sizes
   - Use appropriate log drivers

3. **Resource Usage**
   - Set appropriate limits
   - Monitor resource consumption
   - Plan for scaling
   - Use resource reservation when needed

4. **Networking**
   - Use custom networks for isolation
   - Implement proper security measures
   - Plan network topology
   - Monitor network usage

## Next Steps
After completing these exercises, you should:
1. Be comfortable with container lifecycle management
2. Understand container logging and monitoring
3. Know how to work with container shells
4. Have basic understanding of container networking

Ready to move on to Module 3 to learn about:
- Docker images
- Building custom images
- Image management
- Working with Docker Hub