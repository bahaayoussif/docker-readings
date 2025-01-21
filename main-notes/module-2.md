# Module 2: Working with Docker Commands

## Learning Objectives
By the end of this module, you will be able to:
- Use essential Docker CLI commands
- Manage container lifecycle
- Work with container logs and shell access
- Understand container states and transitions

## 1. Essential Docker CLI Commands

### Container Management Commands
```bash
# Running Containers
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
docker start CONTAINER
docker stop CONTAINER
docker restart CONTAINER
docker pause CONTAINER
docker unpause CONTAINER
docker rm CONTAINER

# Container Information
docker ps # List running containers
docker ps -a # List all containers
docker inspect CONTAINER
docker logs CONTAINER
docker stats CONTAINER
docker top CONTAINER

# Container Interaction
docker exec -it CONTAINER COMMAND
docker attach CONTAINER
```

### Common docker run Options
```bash
# Port mapping
-p, --publish HOST_PORT:CONTAINER_PORT

# Environment variables
-e, --env KEY=VALUE

# Detached mode
-d, --detach

# Interactive mode with terminal
-it

# Container name
--name CONTAINER_NAME

# Volume mounting
-v, --volume HOST_PATH:CONTAINER_PATH

# Network
--network NETWORK_NAME
```

## 2. Container Lifecycle Management

### Starting Containers
```bash
# Basic container run
docker run nginx

# Run in detached mode
docker run -d nginx

# Run with a specific name
docker run --name my-nginx nginx

# Run with port mapping
docker run -p 8080:80 nginx
```

### Stopping Containers
```bash
# Stop a running container
docker stop CONTAINER_ID

# Stop multiple containers
docker stop CONTAINER_ID1 CONTAINER_ID2

# Stop all running containers
docker stop $(docker ps -q)
```

### Removing Containers
```bash
# Remove a stopped container
docker rm CONTAINER_ID

# Force remove a running container
docker rm -f CONTAINER_ID

# Remove all stopped containers
docker container prune
```

## 3. Container Logs and Monitoring

### Viewing Logs
```bash
# View container logs
docker logs CONTAINER_ID

# Follow log output
docker logs -f CONTAINER_ID

# Show timestamps
docker logs -t CONTAINER_ID

# Show last n lines
docker logs --tail n CONTAINER_ID

# Show logs since timestamp
docker logs --since TIMESTAMP CONTAINER_ID
```

### Monitoring Container Performance
```bash
# View container resource usage statistics
docker stats

# View running processes in a container
docker top CONTAINER_ID

# Inspect container details
docker inspect CONTAINER_ID
```

## 4. Working with Container Shell

### Accessing Container Shell
```bash
# Start an interactive shell
docker exec -it CONTAINER_ID /bin/bash

# Run a specific command
docker exec CONTAINER_ID COMMAND

# Attach to a running container
docker attach CONTAINER_ID
```

### Common Shell Operations
```bash
# Check container environment variables
docker exec CONTAINER_ID env

# View container filesystem
docker exec CONTAINER_ID ls -la

# Create/modify files
docker exec CONTAINER_ID touch /tmp/newfile
```

## 5. Container States and Transitions

### Understanding Container States
- **Created**: Container is created but not started
- **Running**: Container is running with all processes active
- **Paused**: Container processes are paused
- **Stopped**: Container processes are stopped
- **Exited**: Container has completed execution or was stopped
- **Dead**: Container is in an unrecoverable state

### State Transition Commands
```bash
# Create a new container (created state)
docker create IMAGE

# Start container (running state)
docker start CONTAINER_ID

# Pause container (paused state)
docker pause CONTAINER_ID

# Unpause container (running state)
docker unpause CONTAINER_ID

# Stop container (exited state)
docker stop CONTAINER_ID
```

## 6. Practical Exercises

### Exercise 1: Basic Container Management
1. Run an nginx container in detached mode
2. List running containers
3. Stop the container
4. Remove the container

### Exercise 2: Container Logs and Shell Access
1. Run a container with a specific name
2. View container logs
3. Access container shell
4. Execute commands inside container

### Exercise 3: Container Monitoring
1. Run multiple containers
2. Monitor resource usage
3. Inspect container details
4. View running processes

## 7. Knowledge Check
1. What is the difference between `docker run` and `docker start`?
2. How do you access the shell of a running container?
3. What are the different container states?
4. How do you view container logs?
5. What is the purpose of the `-d` flag in `docker run`?

## Additional Resources
- [Docker CLI Reference](https://docs.docker.com/engine/reference/commandline/cli/)
- [Docker Run Reference](https://docs.docker.com/engine/reference/run/)
- [Docker Logs Guide](https://docs.docker.com/engine/reference/commandline/logs/)

## Next Steps
- Learn about Docker images
- Understand image layers
- Create custom images using Dockerfile