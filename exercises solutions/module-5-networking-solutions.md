# Module 5: Container Networking - Exercise Solutions

## Exercise 1: Basic Network Operations

### Task 1: Exploring Docker Networks
```bash
# List all networks
docker network ls

# Expected output:
NETWORK ID     NAME      DRIVER    SCOPE
abc123def456   bridge    bridge    local
def456ghi789   host      host      local
ghi789jkl012   none      null      local

# Inspect default bridge network
docker network inspect bridge

# Create custom network
docker network create --driver bridge my_custom_net

# Verify network creation
docker network ls | grep my_custom_net
```

### Task 2: Working with Multiple Networks
```bash
# Create two different networks
docker network create frontend
docker network create backend

# Create containers in different networks
docker run -d --name nginx1 --network frontend nginx
docker run -d --name nginx2 --network backend nginx

# Connect a container to multiple networks
docker network connect frontend nginx2

# Verify connections
docker inspect nginx2 --format='{{json .NetworkSettings.Networks}}'
```

### Task 3: Network Cleanup
```bash
# Disconnect container from network
docker network disconnect frontend nginx2

# Remove networks (after stopping containers)
docker container stop nginx1 nginx2
docker container rm nginx1 nginx2
docker network rm frontend backend

# Remove all unused networks
docker network prune
```

## Exercise 2: Container Communication

### Task 1: Basic Container Networking
```bash
# Create a network for testing
docker network create test_net

# Run two nginx containers
docker run -d --name web1 --network test_net nginx
docker run -d --name web2 --network test_net nginx

# Test connectivity between containers
docker exec web1 ping -c 4 web2
docker exec web2 ping -c 4 web1

# Verify DNS resolution
docker exec web1 nslookup web2
docker exec web2 nslookup web1
```

### Task 2: Network Isolation
```bash
# Create isolated networks
docker network create --internal private_net
docker network create public_net

# Run containers in different networks
docker run -d --name private_web --network private_net nginx
docker run -d --name public_web --network public_net nginx

# Test isolation
docker exec private_web ping -c 2 8.8.8.8  # Should fail
docker exec public_web ping -c 2 8.8.8.8   # Should succeed

# Test internal communication
docker network connect private_net public_web
docker exec private_web ping -c 2 public_web
```

### Task 3: Custom Network Configuration
```bash
# Create network with custom subnet
docker network create --subnet=172.20.0.0/16 custom_net

# Create containers with static IPs
docker run -d --name static_web1 \
  --network custom_net \
  --ip 172.20.0.2 \
  nginx

docker run -d --name static_web2 \
  --network custom_net \
  --ip 172.20.0.3 \
  nginx

# Verify IP assignments
docker inspect static_web1 --format='{{.NetworkSettings.Networks.custom_net.IPAddress}}'
docker inspect static_web2 --format='{{.NetworkSettings.Networks.custom_net.IPAddress}}'

# Test connectivity
docker exec static_web1 ping -c 2 172.20.0.3
```

## Exercise 3: Port Mapping and Exposure

### Task 1: Basic Port Mapping
```bash
# Run container with port mapping
docker run -d --name web_server -p 8080:80 nginx

# Multiple port mappings
docker run -d --name multi_ports \
  -p 8081:80 \
  -p 443:443 \
  nginx

# Verify port mappings
docker port web_server
docker port multi_ports

# Test access
curl http://localhost:8080
curl http://localhost:8081
```

### Task 2: Advanced Port Configuration
```bash
# Specific interface binding
docker run -d --name local_only \
  -p 127.0.0.1:8082:80 \
  nginx

# Dynamic port mapping
docker run -d --name dynamic_ports -P nginx

# UDP port mapping
docker run -d --name udp_ports \
  -p 53:53/udp \
  nginx

# Get assigned ports
docker port dynamic_ports

# Test specific interface binding
curl http://127.0.0.1:8082
```

### Task 3: Port Range Mapping
```bash
# Map port range
docker run -d --name range_ports \
  -p 8000-8010:8000-8010 \
  nginx

# Verify range mappings
docker port range_ports | sort

# Test access to ports
for port in {8000..8010}; do
  curl -s -o /dev/null -w "Port $port: %{http_code}\n" http://localhost:$port
done
```

## Exercise 4: Network Troubleshooting

### Task 1: Debugging Network Issues
```bash
# Create test environment
docker network create debug_net
docker run -d --name debug_web1 --network debug_net nginx
docker run -d --name debug_web2 --network debug_net nginx

# Check network connectivity
docker exec debug_web1 ping -c 2 debug_web2

# Inspect network details
docker network inspect debug_net

# Check container DNS resolution
docker exec debug_web1 nslookup debug_web2

# View container network settings
docker inspect debug_web1 --format='{{json .NetworkSettings.Networks}}'
```

### Task 2: Network Monitoring
```bash
# Monitor network traffic
docker run -d --name monitored_web --network debug_net nginx
docker stats monitored_web

# Using nicolaka/netshoot for advanced debugging
docker run -it --network container:monitored_web nicolaka/netshoot
# Inside netshoot container:
iftop
tcpdump -i any
ping monitored_web
```

### Task 3: Common Issues Resolution
```bash
# DNS resolution issues
docker exec debug_web1 cat /etc/resolv.conf

# Reset container networking
docker network disconnect debug_net debug_web1
docker network connect debug_net debug_web1

# Check iptables rules
docker run --privileged --rm alpine iptables -L

# Verify network driver health
docker network ls --filter driver=bridge
```

## Best Practices

1. **Network Security**
   - Use internal networks for sensitive services
   - Implement network segmentation
   - Restrict external access where possible
   - Regular audit of network connections

2. **Performance**
   - Use appropriate network drivers
   - Monitor network usage
   - Implement load balancing
   - Regular cleanup of unused networks

3. **Troubleshooting**
   - Keep track of network mappings
   - Use meaningful container names
   - Regular monitoring of network metrics
   - Document network architecture

4. **Port Management**
   - Use specific port mappings
   - Document port assignments
   - Implement port conflict resolution
   - Regular port usage audit

## Common Issues and Solutions

1. **DNS Resolution Issues**
```bash
# Check DNS configuration
docker exec container_name cat /etc/resolv.conf

# Update DNS settings
docker run --dns 8.8.8.8 image_name
```

2. **Network Connectivity Problems**
```bash
# Verify network configuration
docker network inspect network_name

# Reset container networking
docker network disconnect network_name container_name
docker network connect network_name container_name
```

3. **Port Conflicts**
```bash
# Check port usage
netstat -tulpn | grep LISTEN

# Use different port mapping
docker run -p 8081:80 nginx  # Instead of 80:80
```

## Next Steps
- Learn about Docker Compose networking
- Explore overlay networks
- Study service discovery
- Implement network security policies

Remember to clean up resources after exercises:
```bash
# Stop all containers
docker stop $(docker ps -aq)

# Remove all containers
docker rm $(docker ps -aq)

# Remove all networks (except default ones)
docker network prune
```