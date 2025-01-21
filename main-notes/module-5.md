# Module 5: Container Networking

## Learning Objectives
By the end of this module, you will be able to:
- Understand Docker networking concepts and types
- Create and manage custom networks
- Configure container-to-container communication
- Implement port mapping and exposure
- Troubleshoot common networking issues

## 1. Docker Network Types

### Default Networks
1. **bridge**: Default network for containers
2. **host**: Container uses host's network stack
3. **none**: Containers have no network access
4. **overlay**: Multi-host networking
5. **macvlan**: Assign MAC address to container

### Network Commands
```bash
# List networks
docker network ls

# Inspect network
docker network inspect NETWORK_NAME

# Create network
docker network create NETWORK_NAME

# Remove network
docker network rm NETWORK_NAME

# Remove unused networks
docker network prune
```

## 2. Working with Bridge Networks

### Default Bridge Network
```bash
# Run container on default bridge
docker run -d nginx

# Inspect bridge network
docker network inspect bridge

# Link containers (legacy)
docker run --link container1:alias container2
```

### Custom Bridge Networks
```bash
# Create custom bridge network
docker network create --driver bridge my_network

# Run container on custom network
docker run -d --network my_network --name container1 nginx

# Connect existing container to network
docker network connect my_network container2

# Disconnect from network
docker network disconnect my_network container2
```

## 3. Container-to-Container Communication

### DNS Resolution
```bash
# Create network with custom subnet
docker network create --subnet=172.20.0.0/16 my_net

# Run containers with custom IP
docker run -d --network my_net --ip 172.20.0.2 --name web nginx
docker run -d --network my_net --ip 172.20.0.3 --name db mysql

# Test communication
docker exec web ping db
```

### Network Aliases
```bash
# Create container with network alias
docker run -d --network my_net --network-alias db mysql

# Multiple aliases
docker run --network-alias db1 --network-alias db2 mysql
```

## 4. Port Mapping and Exposure

### Exposing Ports
```dockerfile
# In Dockerfile
EXPOSE 80
EXPOSE 443
```

### Port Publishing
```bash
# Publish port to random host port
docker run -P nginx

# Publish to specific port
docker run -p 8080:80 nginx

# Publish multiple ports
docker run -p 80:80 -p 443:443 nginx

# Publish to specific interface
docker run -p 127.0.0.1:80:80 nginx
```

### Port Management
```bash
# List port mappings
docker port CONTAINER

# View published ports
docker ps --format "{{.Names}}: {{.Ports}}"
```

## 5. Host Network Mode

### Using Host Network
```bash
# Run container with host network
docker run --network host nginx

# Verify network details
docker inspect --format '{{.NetworkSettings.Networks}}' CONTAINER
```

### Host Network Considerations
- Direct access to host network interfaces
- Better performance (no network bridge)
- No port mapping needed
- Less network isolation
- Security implications

## 6. Network Security

### Network Isolation
```bash
# Create isolated networks
docker network create --internal internal_net

# No outbound connectivity
docker run --network internal_net nginx
```

### Network Policies
```bash
# Enable user-defined networks only
docker daemon --config-file /etc/docker/daemon.json

# daemon.json
{
  "default-network-opts": {
    "bridge": {
      "com.docker.network.bridge.enable_icc": false
    }
  }
}
```

## 7. Advanced Networking Features

### Overlay Networks
```bash
# Initialize swarm mode
docker swarm init

# Create overlay network
docker network create -d overlay my_overlay

# Use with swarm services
docker service create --network my_overlay nginx
```

### MacVLAN Networks
```bash
# Create macvlan network
docker network create -d macvlan \
  --subnet=192.168.1.0/24 \
  --gateway=192.168.1.1 \
  -o parent=eth0 macvlan_net
```

## 8. Networking Troubleshooting

### Common Commands
```bash
# Check container networking
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' CONTAINER

# View network statistics
docker stats CONTAINER

# Network debugging tools
docker run --rm --net container:CONTAINER nicolaka/netshoot
```

### Common Issues
1. DNS resolution problems
2. Port conflicts
3. Network connectivity issues
4. Bridge network limitations

## 9. Practical Exercises

### Exercise 1: Basic Networking
1. Create custom bridge network
2. Run multiple containers
3. Test container communication
4. Implement port mapping

### Exercise 2: Network Isolation
1. Create isolated networks
2. Configure container access
3. Test network policies
4. Implement security measures

### Exercise 3: Advanced Features
1. Set up overlay network
2. Configure MacVLAN
3. Implement load balancing
4. Test high availability

## 10. Knowledge Check
1. What are the different types of Docker networks?
2. How does container DNS resolution work?
3. When should you use host networking?
4. What are network aliases used for?
5. How do you troubleshoot network connectivity issues?

## Additional Resources
- [Docker Networking Overview](https://docs.docker.com/network/)
- [Networking with Overlay Networks](https://docs.docker.com/network/overlay/)
- [Docker Network Command Reference](https://docs.docker.com/engine/reference/commandline/network/)
- [Network Security Best Practices](https://docs.docker.com/network/security/)

## Next Steps
- Learn about Docker volumes and data persistence
- Understand Docker Compose networking
- Explore container orchestration