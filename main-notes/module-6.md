# Module 6: Data Management

## Learning Objectives
By the end of this module, you will be able to:
- Understand Docker storage options and their use cases
- Work with volumes for persistent data
- Implement bind mounts effectively
- Manage data sharing between containers
- Implement backup and restore strategies

## 1. Docker Storage Concepts

### Storage Types
1. **Volumes**: Managed by Docker
2. **Bind Mounts**: Direct host filesystem mapping
3. **tmpfs mounts**: Memory-only storage
4. **Named Pipes**: Inter-process communication

### Storage Commands
```bash
# List volumes
docker volume ls

# Create volume
docker volume create my_volume

# Inspect volume
docker volume inspect my_volume

# Remove volume
docker volume rm my_volume

# Remove unused volumes
docker volume prune
```

## 2. Working with Docker Volumes

### Volume Management
```bash
# Create volume with specific driver
docker volume create --driver local \
    --opt type=none \
    --opt device=/path/on/host \
    --opt o=bind \
    my_volume

# Run container with volume
docker run -v my_volume:/data nginx

# Use volume-from other container
docker run --volumes-from container1 nginx
```

### Volume Drivers
```bash
# Create volume with NFS driver
docker volume create --driver local \
    --opt type=nfs \
    --opt o=addr=192.168.1.1,rw \
    --opt device=:/path/to/dir \
    nfs_volume

# Using cloud storage volumes
docker volume create --driver rexray/ebs aws_volume
```

## 3. Bind Mounts

### Basic Bind Mount Usage
```bash
# Mount host directory
docker run -v /host/path:/container/path nginx

# Read-only bind mount
docker run -v /host/path:/container/path:ro nginx

# Using --mount flag
docker run --mount type=bind,source=/host/path,target=/container/path nginx
```

### Bind Mount Best Practices
1. Use absolute paths
2. Consider security implications
3. Handle permissions properly
4. Use read-only when possible
5. Avoid system directories

## 4. Data Sharing Patterns

### Container-to-Container
```bash
# Data volume container
docker create -v /data --name datastore alpine

# Share with other containers
docker run --volumes-from datastore app1
docker run --volumes-from datastore app2
```

### Host-to-Container
```bash
# Share specific files
docker run -v /host/config.json:/app/config.json:ro nginx

# Share multiple directories
docker run \
    -v /host/configs:/etc/configs:ro \
    -v /host/data:/data \
    nginx
```

## 5. Backup and Restore

### Volume Backup
```bash
# Backup volume to tar file
docker run --rm \
    -v my_volume:/source:ro \
    -v $(pwd):/backup \
    alpine tar czvf /backup/volume_backup.tar.gz -C /source .

# Restore from backup
docker run --rm \
    -v my_volume:/target \
    -v $(pwd):/backup \
    alpine tar xzvf /backup/volume_backup.tar.gz -C /target
```

### Container Data Backup
```bash
# Backup container data
docker container commit container_name backup_image

# Export container filesystem
docker export container_name > container_backup.tar

# Import container filesystem
docker import container_backup.tar new_image
```

## 6. Performance and Security

### Performance Optimization
```bash
# Use tmpfs for temporary data
docker run --tmpfs /tmp nginx

# Configure volume driver options
docker volume create --opt size=20GB my_volume

# Use volume cache options
docker run --volume-opt cache=shared nginx
```

### Security Considerations
```bash
# Set volume permissions
docker run -v my_volume:/data:ro,Z nginx

# Use SELinux labels
docker run --security-opt label=type:container_file_t nginx

# Encrypt sensitive data
docker volume create --opt encrypted=true secure_volume
```

## 7. Advanced Volume Features

### Volume Plugins
```bash
# Install volume plugin
docker plugin install rexray/ebs

# Create volume with plugin
docker volume create -d rexray/ebs \
    --opt size=20 \
    --opt encrypted=true \
    ebs_volume
```

### Volume Labels and Filtering
```bash
# Create labeled volume
docker volume create --label env=prod my_volume

# Filter volumes by label
docker volume ls --filter label=env=prod

# Remove volumes by label
docker volume prune --filter label=env=prod
```

## 8. Data Management Patterns

### Development Workflow
```bash
# Development setup
docker run -v $(pwd):/app \
    -v /app/node_modules \
    -p 3000:3000 \
    node:alpine npm start

# Production setup
docker volume create --name=node_modules
docker run -v node_modules:/app/node_modules node:alpine
```

### Database Management
```bash
# Create database volume
docker volume create postgres_data

# Run database with volume
docker run -d \
    -v postgres_data:/var/lib/postgresql/data \
    -e POSTGRES_PASSWORD=secret \
    postgres

# Backup database
docker exec postgres pg_dump -U postgres > backup.sql
```

## 9. Practical Exercises

### Exercise 1: Volume Management
1. Create and manage volumes
2. Implement data persistence
3. Share data between containers
4. Test volume drivers

### Exercise 2: Backup Strategies
1. Create backup procedures
2. Test restore processes
3. Automate backup tasks
4. Verify data integrity

### Exercise 3: Performance Testing
1. Compare storage options
2. Measure I/O performance
3. Optimize configurations
4. Monitor resource usage

## 10. Knowledge Check
1. What are the different types of Docker storage?
2. When should you use volumes vs bind mounts?
3. How do you back up Docker volumes?
4. What are volume drivers used for?
5. How do you share data between containers?

## Additional Resources
- [Docker Storage Overview](https://docs.docker.com/storage/)
- [Volumes Guide](https://docs.docker.com/storage/volumes/)
- [Bind Mounts](https://docs.docker.com/storage/bind-mounts/)
- [Storage Drivers](https://docs.docker.com/storage/storagedriver/)

## Next Steps
- Learn about Docker Compose
- Understand multi-container applications
- Explore container orchestration solutions