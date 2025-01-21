# Module 8: Container Orchestration Preview - Exercise Solutions

## Exercise 1: Docker Swarm Basics

### Task 1: Swarm Initialization and Node Management
```bash
# Initialize Swarm on manager node
docker swarm init --advertise-addr <MANAGER-IP>

# Expected output
Swarm initialized: current node (dxn1zf6l61qsb1josjja83ngz) is now a manager.

# Join a worker node
docker swarm join --token <WORKER-TOKEN> <MANAGER-IP>:2377

# List nodes
docker node ls

# Promote worker to manager
docker node promote <NODE-NAME>

# View join tokens
docker swarm join-token manager
docker swarm join-token worker
```

### Task 2: Deploy Basic Service
```bash
# Create simple service
docker service create \
  --name web-service \
  --replicas 3 \
  --publish 80:80 \
  nginx

# List services
docker service ls

# Inspect service
docker service ps web-service

# Scale service
docker service scale web-service=5

# Update service
docker service update \
  --image nginx:alpine \
  --update-parallelism 2 \
  --update-delay 20s \
  web-service
```

### Task 3: Service Management
```bash
# Create service with constraints
docker service create \
  --name db-service \
  --constraint 'node.role==worker' \
  --mount type=volume,source=db_data,target=/var/lib/mysql \
  --env MYSQL_ROOT_PASSWORD=secret \
  mysql:8.0

# Create overlay network
docker network create --driver overlay backend-network

# Attach service to network
docker service update \
  --network-add backend-network \
  db-service
```

## Exercise 2: Kubernetes Basics with Minikube

### Task 1: Setup and Basic Commands
```bash
# Start Minikube
minikube start

# Check cluster status
kubectl cluster-info
kubectl get nodes

# Create deployment
kubectl create deployment nginx-deploy --image=nginx

# Expose deployment
kubectl expose deployment nginx-deploy \
  --type=LoadBalancer \
  --port=80

# View resources
kubectl get deployments
kubectl get pods
kubectl get services
```

### Task 2: Basic Pod Management
```yaml
# nginx-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

```bash
# Apply pod configuration
kubectl apply -f nginx-pod.yaml

# Get pod information
kubectl describe pod nginx-pod

# Access pod logs
kubectl logs nginx-pod

# Execute command in pod
kubectl exec -it nginx-pod -- /bin/bash

# Delete pod
kubectl delete pod nginx-pod
```

### Task 3: Deployment Configuration
```yaml
# nginx-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: "0.5"
            memory: "512Mi"
          requests:
            cpu: "0.2"
            memory: "256Mi"
```

## Exercise 3: Advanced Orchestration Features

### Task 1: Docker Swarm Stack Deployment
```yaml
# docker-compose.stack.yml
version: '3.8'

services:
  web:
    image: nginx:alpine
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
      restart_policy:
        condition: on-failure
    ports:
      - "80:80"
    networks:
      - webnet

  visualizer:
    image: dockersamples/visualizer
    ports:
      - "8080:8080"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    deploy:
      placement:
        constraints: [node.role == manager]

networks:
  webnet:
```

```bash
# Deploy stack
docker stack deploy -c docker-compose.stack.yml myapp

# List stacks
docker stack ls

# List stack services
docker stack services myapp

# Remove stack
docker stack rm myapp
```

### Task 2: Kubernetes Configuration Management
```yaml
# config-map.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database_url: "mongodb://db:27017"
  api_key: "development-key"

---
# secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  db_password: base64encodedpassword
  api_token: base64encodedtoken

---
# deployment-with-config.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:1.0
        env:
        - name: DB_URL
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: database_url
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: db_password
```

## Exercise 4: High Availability and Scaling

### Task 1: Load Balancing in Swarm
```bash
# Create service with load balancing
docker service create \
  --name web \
  --publish published=8080,target=80 \
  --replicas 5 \
  --update-delay 10s \
  --update-parallelism 2 \
  nginx

# Enable rolling updates
docker service update \
  --update-delay 30s \
  --update-parallelism 1 \
  --image nginx:alpine \
  web
```

### Task 2: Kubernetes Scaling and Updates
```yaml
# deployment-scaling.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: scaling-demo
spec:
  replicas: 3
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  selector:
    matchLabels:
      app: scaling-demo
  template:
    metadata:
      labels:
        app: scaling-demo
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        resources:
          requests:
            cpu: "100m"
          limits:
            cpu: "200m"
```

```bash
# Apply autoscaling
kubectl autoscale deployment scaling-demo \
  --cpu-percent=50 \
  --min=3 \
  --max=10

# View autoscaling status
kubectl get hpa
```

## Best Practices

1. **Resource Management**
```yaml
# Kubernetes resource limits
resources:
  limits:
    cpu: "1"
    memory: "1Gi"
  requests:
    cpu: "500m"
    memory: "512Mi"

# Swarm resource constraints
deploy:
  resources:
    limits:
      cpus: '0.50'
      memory: 512M
```

2. **High Availability Configuration**
```yaml
# Swarm configuration
deploy:
  mode: replicated
  replicas: 3
  placement:
    constraints:
      - node.role == worker
  update_config:
    parallelism: 1
    delay: 10s
    order: start-first

# Kubernetes configuration
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
```

3. **Security Best Practices**
```yaml
# Swarm secrets
secrets:
  db_password:
    external: true

# Kubernetes security context
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  capabilities:
    drop:
      - ALL
```

## Common Issues and Solutions

1. **Node Communication Issues**
```bash
# Check Swarm node status
docker node ls
docker node inspect node-id

# Check Kubernetes node status
kubectl get nodes
kubectl describe node node-name
```

2. **Service Discovery Problems**
```bash
# Swarm service inspection
docker service inspect service-name
docker service logs service-name

# Kubernetes service debugging
kubectl describe service service-name
kubectl get endpoints service-name
```

3. **Resource Constraints**
```bash
# Check resource usage
docker stats
kubectl top nodes
kubectl top pods
```

## Next Steps
- Learn advanced Kubernetes concepts
- Explore service mesh solutions
- Study CI/CD integration
- Implement monitoring solutions

Remember to clean up:
```bash
# Swarm cleanup
docker stack rm stack-name
docker swarm leave --force

# Kubernetes cleanup
kubectl delete deployment --all
kubectl delete service --all
minikube stop
```