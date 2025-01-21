# Module 7: Docker Compose

## Learning Objectives
By the end of this module, you will be able to:
- Understand Docker Compose concepts and syntax
- Create and manage multi-container applications
- Work with environment variables and configurations
- Implement development and production setups
- Debug and troubleshoot Compose applications

## 1. Introduction to Docker Compose

### Core Concepts
- Tool for defining multi-container applications
- Uses YAML file format
- Simplifies container orchestration
- Manages application lifecycle
- Handles service dependencies

### Basic Commands
```bash
# Start services
docker compose up

# Run in detached mode
docker compose up -d

# Stop services
docker compose down

# View logs
docker compose logs

# List services
docker compose ps

# Execute command in service
docker compose exec service_name command
```

## 2. Docker Compose File Structure

### Basic docker-compose.yml
```yaml
version: '3.8'
services:
  web:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./html:/usr/share/nginx/html
  
  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: myapp
    volumes:
      - db_data:/var/lib/mysql

volumes:
  db_data:
```

### Service Configuration Options
```yaml
services:
  webapp:
    build: 
      context: ./app
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
    environment:
      NODE_ENV: development
    depends_on:
      - db
    networks:
      - frontend
      - backend
    deploy:
      replicas: 2
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
```

## 3. Building Services

### Building from Dockerfile
```yaml
services:
  app:
    build:
      context: ./app
      dockerfile: Dockerfile
      args:
        BUILD_ENV: development
    image: myapp:latest
```

### Multi-stage Builds
```yaml
services:
  app:
    build:
      context: ./app
      target: development
    volumes:
      - ./app:/usr/src/app
```

## 4. Environment Management

### Environment Variables
```yaml
# Using .env file
services:
  web:
    env_file:
      - ./web.env
    environment:
      - API_KEY
      - DEBUG=1
      - DATABASE_URL=${DB_URL}
```

### Development vs Production
```yaml
# docker-compose.override.yml
services:
  web:
    build:
      target: development
    volumes:
      - ./src:/app/src
    environment:
      NODE_ENV: development

# docker-compose.prod.yml
services:
  web:
    build:
      target: production
    environment:
      NODE_ENV: production
```

## 5. Networking in Compose

### Network Configuration
```yaml
services:
  web:
    networks:
      - frontend
      - backend
  
  db:
    networks:
      - backend

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true
```

### Service Discovery
```yaml
services:
  web:
    depends_on:
      - api
      - db
    environment:
      API_URL: http://api:3000
      DB_HOST: db
```

## 6. Volume Management

### Volume Types
```yaml
services:
  db:
    volumes:
      # Named volume
      - db_data:/var/lib/mysql
      
      # Bind mount
      - ./config:/etc/mysql/conf.d
      
      # Anonymous volume
      - /var/lib/mysql/data

volumes:
  db_data:
    driver: local
    driver_opts:
      type: nfs
      device: :/path/to/dir
      o: addr=192.168.1.1
```

## 7. Service Dependencies

### Dependency Management
```yaml
services:
  web:
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started
  
  db:
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 30s
      timeout: 10s
      retries: 3
```

## 8. Scaling and Deployment

### Service Scaling
```yaml
services:
  worker:
    image: worker:latest
    deploy:
      mode: replicated
      replicas: 6
      resources:
        limits:
          cpus: '0.5'
          memory: 256M
      restart_policy:
        condition: on-failure
```

### Load Balancing
```yaml
services:
  web:
    image: nginx
    deploy:
      mode: replicated
      replicas: 3
      labels:
        - "traefik.enable=true"
        - "traefik.http.routers.web.rule=Host(`example.com`)"
```

## 9. Development Workflow

### Local Development Setup
```yaml
services:
  frontend:
    build:
      context: ./frontend
      target: development
    volumes:
      - ./frontend:/app
      - /app/node_modules
    ports:
      - "3000:3000"
    command: npm run dev

  backend:
    build:
      context: ./backend
      target: development
    volumes:
      - ./backend:/app
    ports:
      - "5000:5000"
    command: npm run dev
```

### Testing Environment
```yaml
services:
  test:
    build:
      context: .
      target: test
    volumes:
      - .:/app
    command: npm test
    environment:
      - NODE_ENV=test
      - CI=true
```

## 10. Practical Exercises

### Exercise 1: Basic Multi-container App
1. Create a web application with database
2. Define services in docker-compose.yml
3. Implement environment variables
4. Test service communication

### Exercise 2: Development Environment
1. Set up development configuration
2. Implement hot-reloading
3. Configure debugging
4. Manage dependencies

### Exercise 3: Production Deployment
1. Create production configuration
2. Implement scaling
3. Set up monitoring
4. Configure backups

## 11. Knowledge Check
1. What is the purpose of docker-compose.yml?
2. How do you manage service dependencies?
3. What are the different types of volumes?
4. How do you scale services?
5. What is service discovery in Compose?

## Additional Resources
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Compose File Reference](https://docs.docker.com/compose/compose-file/)
- [Compose CLI Reference](https://docs.docker.com/compose/reference/)
- [Best Practices Guide](https://docs.docker.com/develop/dev-best-practices/)

## Next Steps
- Learn about container orchestration
- Explore Kubernetes basics
- Study production deployment strategies