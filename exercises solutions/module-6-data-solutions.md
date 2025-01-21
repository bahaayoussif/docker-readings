# Module 6: Data Management - Exercise Solutions

## Exercise 1: Working with Docker Volumes

### Task 1: Basic Volume Operations
```bash
# Create named volume
docker volume create data_volume

# List volumes
docker volume ls

# Inspect volume
docker volume inspect data_volume

# Run container with volume
docker run -d \
  --name nginx_with_volume \
  -v data_volume:/usr/share/nginx/html \
  nginx

# Verify volume mounting
docker inspect nginx_with_volume --format='{{json .Mounts}}'
```

### Task 2: Volume Data Persistence
```bash
# Create container with volume and add data
docker run -d --name web1 -v data_volume:/app nginx
docker exec web1 sh -c "echo 'Hello from volume' > /app/test.txt"

# Stop and remove first container
docker stop web1
docker rm web1

# Create new container using same volume
docker run -d --name web2 -v data_volume:/app nginx
docker exec web2 cat /app/test.txt

# Expected output: Hello from volume
```

### Task 3: Volume Drivers and Options
```bash
# Create volume with specific driver options
docker volume create --driver local \
  --opt type=none \
  --opt device=/home/user/data \
  --opt o=bind \
  external_volume

# Use volume with specific options
docker run -d \
  --name custom_volume_container \
  -v external_volume:/data \
  nginx

# Verify driver options
docker volume inspect external_volume
```

## Exercise 2: Bind Mounts

### Task 1: Basic Bind Mount
```bash
# Create host directory and file
mkdir ~/docker-data
echo "Host data" > ~/docker-data/host-file.txt

# Run container with bind mount
docker run -d \
  --name bind_mount_test \
  -v ~/docker-data:/container-data \
  nginx

# Verify bind mount
docker exec bind_mount_test ls /container-data
docker exec bind_mount_test cat /container-data/host-file.txt
```

### Task 2: Read-Only Bind Mounts
```bash
# Run container with read-only bind mount
docker run -d \
  --name readonly_test \
  -v ~/docker-data:/container-data:ro \
  nginx

# Try to modify data (should fail)
docker exec readonly_test sh -c "echo 'test' > /container-data/new-file.txt"
# Expected error: Read-only file system

# Verify read access
docker exec readonly_test cat /container-data/host-file.txt
```

### Task 3: Development Setup with Bind Mounts
```bash
# Create development directory
mkdir ~/dev-project
cat > ~/dev-project/index.html << EOF
<!DOCTYPE html>
<html>
<body>
    <h1>Development Site</h1>
</body>
</html>
EOF

# Run container with development bind mount
docker run -d \
  --name dev_env \
  -v ~/dev-project:/usr/share/nginx/html \
  -p 8080:80 \
  nginx

# Test live updates
echo "<p>Updated content</p>" >> ~/dev-project/index.html
curl http://localhost:8080
```

## Exercise 3: Data Backup and Restore

### Task 1: Volume Backup
```bash
# Create volume with data
docker volume create backup_test
docker run --rm -v backup_test:/data alpine sh -c "echo 'test data' > /data/file.txt"

# Backup volume to tar file
docker run --rm \
  -v backup_test:/source:ro \
  -v $(pwd):/backup \
  alpine tar czf /backup/volume_backup.tar.gz -C /source .

# Verify backup
ls -lh volume_backup.tar.gz
```

### Task 2: Volume Restore
```bash
# Create new volume for restore
docker volume create restore_test

# Restore from backup
docker run --rm \
  -v restore_test:/target \
  -v $(pwd):/backup \
  alpine sh -c "cd /target && tar xzf /backup/volume_backup.tar.gz"

# Verify restore
docker run --rm \
  -v restore_test:/data \
  alpine cat /data/file.txt
```

### Task 3: Container Data Backup
```bash
# Create container with data
docker run -d --name data_container nginx
docker exec data_container sh -c "echo 'container data' > /test.txt"

# Backup container to image
docker commit data_container backup_image

# Create tar of the container
docker export data_container > container_backup.tar

# Restore from tar
docker import container_backup.tar restored_image
```

## Exercise 4: Database Data Management

### Task 1: PostgreSQL with Volume
```bash
# Create volume for PostgreSQL data
docker volume create pgdata

# Run PostgreSQL with volume
docker run -d \
  --name postgres_db \
  -e POSTGRES_PASSWORD=mysecret \
  -v pgdata:/var/lib/postgresql/data \
  postgres:13

# Create test data
docker exec -it postgres_db psql -U postgres -c "CREATE DATABASE testdb;"
docker exec -it postgres_db psql -U postgres -c "CREATE TABLE test (id serial PRIMARY KEY, name VARCHAR);"
docker exec -it postgres_db psql -U postgres -c "INSERT INTO test (name) VALUES ('test data');"

# Verify persistence
docker stop postgres_db
docker rm postgres_db

# Run new container with same volume
docker run -d \
  --name postgres_db2 \
  -e POSTGRES_PASSWORD=mysecret \
  -v pgdata:/var/lib/postgresql/data \
  postgres:13

# Check data persistence
docker exec -it postgres_db2 psql -U postgres -c "SELECT * FROM test;"
```

### Task 2: Database Backup Strategies
```bash
# Backup PostgreSQL database
docker exec postgres_db2 pg_dump -U postgres testdb > backup.sql

# Stop container and remove
docker stop postgres_db2
docker rm postgres_db2

# Start new container and restore
docker run -d \
  --name postgres_db3 \
  -e POSTGRES_PASSWORD=mysecret \
  -v pgdata:/var/lib/postgresql/data \
  postgres:13

# Restore database
cat backup.sql | docker exec -i postgres_db3 psql -U postgres testdb
```

## Exercise 5: Volume Management and Maintenance

### Task 1: Volume Cleanup
```bash
# List all volumes
docker volume ls

# Remove specific volume
docker volume rm volume_name

# Remove all unused volumes
docker volume prune

# Force remove volumes
docker volume rm -f volume_name
```

### Task 2: Volume Labels
```bash
# Create volumes with labels
docker volume create \
  --label environment=production \
  --label app=webserver \
  prod_data

# Filter volumes by label
docker volume ls --filter label=environment=production

# Inspect volume labels
docker volume inspect prod_data
```

### Task 3: Volume Monitoring
```bash
# Check volume usage
docker system df -v

# Monitor container volume usage
docker run -d --name volume_test -v data_volume:/data nginx
docker exec volume_test df -h /data
```

## Best Practices

1. **Data Organization**
   ```bash
   # Use meaningful volume names
   docker volume create app_data
   docker volume create db_data
   
   # Use labels for organization
   docker volume create \
     --label project=myapp \
     --label environment=prod \
     myapp_prod_data
   ```

2. **Backup Strategy**
   ```bash
   # Regular backup script
   #!/bin/bash
   BACKUP_DIR="/backup"
   DATE=$(date +%Y%m%d)
   
   # Backup each volume
   for volume in $(docker volume ls -q); do
     docker run --rm \
       -v $volume:/source:ro \
       -v $BACKUP_DIR:/backup \
       alpine tar czf /backup/${volume}_${DATE}.tar.gz -C /source .
   done
   ```

3. **Security Considerations**
   ```bash
   # Use read-only mounts when possible
   docker run -v data:/data:ro nginx
   
   # Set proper permissions
   docker run \
     --user 1000:1000 \
     -v data:/data \
     nginx
   ```

## Common Issues and Solutions

1. **Permission Issues**
```bash
# Fix permissions on host
sudo chown -R user:user ~/docker-data

# Use specific user in container
docker run --user $(id -u):$(id -g) -v ~/data:/data nginx
```

2. **Volume Mount Problems**
```bash
# Check mount points
docker inspect container_name --format='{{json .Mounts}}'

# Verify host directory exists
mkdir -p ~/docker-data

# Check SELinux context
sudo chcon -Rt svirt_sandbox_file_t ~/docker-data
```

3. **Space Issues**
```bash
# Check volume usage
docker system df -v

# Clean up unused volumes
docker volume prune

# Move Docker root directory
sudo systemctl stop docker
sudo mv /var/lib/docker /new/path
sudo ln -s /new/path /var/lib/docker
sudo systemctl start docker
```

## Next Steps
- Learn about Docker Compose volume management
- Explore backup automation
- Study monitoring solutions
- Implement disaster recovery strategies

Remember to clean up after exercises:
```bash
# Remove all containers
docker rm -f $(docker ps -aq)

# Remove all volumes
docker volume prune -f

# Remove backup files
rm -f *.tar.gz
rm -f backup.sql
```