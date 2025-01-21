# Module 7: Docker Compose - Exercise Solutions

## Exercise 1: Basic Multi-container Application

### Task 1: WordPress with MySQL
```yaml
# docker-compose.yml
version: '3.8'

services:
  db:
    image: mysql:8.0
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: somewordpress
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress

  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    ports:
      - "8080:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - wordpress_data:/var/www/html

volumes:
  db_data:
  wordpress_data:
```

```bash
# Start the application
docker compose up -d

# Verify services are running
docker compose ps

# Check logs
docker compose logs

# Stop the application
docker compose down
```

### Task 2: Simple Web Application with Redis
```yaml
# docker-compose.yml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "5000:5000"
    environment:
      REDIS_HOST: redis
    depends_on:
      - redis

  redis:
    image: redis:alpine
    volumes:
      - redis_data:/data

volumes:
  redis_data:
```

```python
# app.py
from flask import Flask
import redis
import os

app = Flask(__name__)
redis_client = redis.Redis(host=os.getenv('REDIS_HOST', 'redis'))

@app.route('/')
def hello():
    redis_client.incr('hits')
    return f'Hello! This page has been viewed {redis_client.get("hits").decode("utf-8")} times.'

if __name__ == "__main__":
    app.run(host="0.0.0.0")
```

```dockerfile
# Dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["python", "app.py"]
```

## Exercise 2: Development Environment

### Task 1: Node.js Application with MongoDB
```yaml
# docker-compose.dev.yml
version: '3.8'

services:
  app:
    build: 
      context: .
      target: development
    volumes:
      - .:/app
      - /app/node_modules
    ports:
      - "3000:3000"
    environment:
      NODE_ENV: development
      MONGO_URL: mongodb://db:27017/devdb
    command: npm run dev

  db:
    image: mongo:latest
    volumes:
      - mongo_data:/data/db
    ports:
      - "27017:27017"

volumes:
  mongo_data:
```

```dockerfile
# Dockerfile
FROM node:18-alpine as development
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
CMD ["npm", "run", "dev"]

FROM node:18-alpine as production
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
CMD ["npm", "start"]
```

### Task 2: Development Tools Integration
```yaml
# docker-compose.tools.yml
version: '3.8'

services:
  app:
    extends:
      file: docker-compose.dev.yml
      service: app
    
  db:
    extends:
      file: docker-compose.dev.yml
      service: db

  adminer:
    image: adminer
    ports:
      - "8080:8080"
    depends_on:
      - db

  test:
    build: 
      context: .
      target: development
    volumes:
      - .:/app
      - /app/node_modules
    command: npm test
    environment:
      NODE_ENV: test
      MONGO_URL: mongodb://db:27017/testdb
```

## Exercise 3: Production Configuration

### Task 1: Production Setup
```yaml
# docker-compose.prod.yml
version: '3.8'

services:
  app:
    build:
      context: .
      target: production
    restart: always
    environment:
      NODE_ENV: production
      MONGO_URL: mongodb://db:27017/proddb
    depends_on:
      - db

  db:
    image: mongo:latest
    volumes:
      - mongo_data:/data/db
    restart: always

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - app

volumes:
  mongo_data:
```

```nginx
# nginx.conf
server {
    listen 80;
    server_name localhost;

    location / {
        proxy_pass http://app:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

### Task 2: Environment Configuration
```bash
# .env.development
NODE_ENV=development
MONGO_URL=mongodb://db:27017/devdb
PORT=3000

# .env.production
NODE_ENV=production
MONGO_URL=mongodb://db:27017/proddb
PORT=3000
```

## Exercise 4: Service Scaling and High Availability

### Task 1: Load Balanced Application
```yaml
# docker-compose.scale.yml
version: '3.8'

services:
  app:
    build: .
    deploy:
      replicas: 3
      restart_policy:
        condition: on-failure
        max_attempts: 3
    environment:
      REDIS_URL: redis://redis:6379

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - app

  redis:
    image: redis:alpine
    volumes:
      - redis_data:/data
    deploy:
      replicas: 1

volumes:
  redis_data:
```

```nginx
# nginx.conf for load balancing
upstream app_servers {
    server app:3000;
}

server {
    listen 80;
    
    location / {
        proxy_pass http://app_servers;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### Task 2: Health Checks
```yaml
# docker-compose.health.yml
version: '3.8'

services:
  app:
    build: .
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

  db:
    image: mongo:latest
    healthcheck:
      test: ["CMD", "mongo", "--eval", "db.adminCommand('ping')"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
```

## Exercise 5: Monitoring and Logging

### Task 1: Logging Setup
```yaml
# docker-compose.logging.yml
version: '3.8'

services:
  app:
    logging:
      driver: "json-file"
      options:
        max-size: "200k"
        max-file: "10"
    
  elasticsearch:
    image: elasticsearch:7.9.3
    environment:
      - discovery.type=single-node
    volumes:
      - elasticsearch_data:/usr/share/elasticsearch/data

  kibana:
    image: kibana:7.9.3
    ports:
      - "5601:5601"
    depends_on:
      - elasticsearch

  filebeat:
    image: elastic/filebeat:7.9.3
    volumes:
      - ./filebeat.yml:/usr/share/filebeat/filebeat.yml:ro
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro

volumes:
  elasticsearch_data:
```

### Task 2: Monitoring Configuration
```yaml
# docker-compose.monitoring.yml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus

  grafana:
    image: grafana/grafana
    ports:
      - "3000:3000"
    volumes:
      - grafana_data:/var/lib/grafana
    depends_on:
      - prometheus

volumes:
  prometheus_data:
  grafana_data:
```

## Best Practices

1. **Environment Management**
```bash
# Use .env files for different environments
touch .env.development .env.production .env.test

# Run with specific environment
docker compose --env-file .env.development up
```

2. **Volume Management**
```yaml
# Named volumes for persistence
volumes:
  data:
    name: "${PROJECT_NAME}_data"
    external: true
```

3. **Network Security**
```yaml
# Create separate networks
networks:
  frontend:
    driver: bridge
  backend:
    internal: true
```

4. **Resource Limits**
```yaml
# Set resource constraints
services:
  app:
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: 512M
```

## Common Issues and Solutions

1. **Container Dependency Issues**
```yaml
services:
  app:
    depends_on:
      db:
        condition: service_healthy
```

2. **Volume Permission Problems**
```yaml
services:
  app:
    user: "1000:1000"
    volumes:
      - ./data:/app/data
```

3. **Network Connectivity**
```yaml
# Use custom networks
networks:
  app_net:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
```

## Next Steps
- Explore Docker Swarm
- Learn Kubernetes basics
- Implement CI/CD pipelines
- Study container orchestration

Cleanup:
```bash
# Stop all containers
docker compose down

# Remove volumes
docker compose down -v

# Remove all unused resources
docker system prune -a
```