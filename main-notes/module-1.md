# Module 1: Understanding Container Basics

## Learning Objectives
By the end of this module, you will be able to:
- Explain the concept of containerization and its benefits
- Differentiate between containers and virtual machines
- Understand Docker's core architecture
- Install Docker on various operating systems

## 1. Introduction to Containerization

### What is Containerization?
Containerization is a lightweight alternative to full machine virtualization that involves encapsulating an application in a container with its own operating environment. This approach:
- Ensures consistent operation across environments
- Enables applications to run reliably when moved from one computing environment to another
- Provides better resource utilization compared to traditional virtualization

### Why Do We Need Containerization?
- **Consistency**: Eliminates "it works on my machine" problems
- **Isolation**: Applications run in isolated environments
- **Portability**: Containers can run anywhere Docker is installed
- **Efficiency**: Uses fewer resources than virtual machines
- **Scalability**: Easy to scale applications up or down
- **Version Control**: Better management of application versions

## 2. Containers vs. Virtual Machines

### Virtual Machines
- Run a full copy of an operating system
- Take up more space
- Slower to start
- Complete isolation
- Hardware-level virtualization

### Containers
- Share the host OS kernel
- Lightweight
- Start quickly
- Process-level isolation
- OS-level virtualization

## 3. Docker Architecture

### Core Components
1. **Docker Daemon (dockerd)**
   - Manages Docker objects
   - Handles container lifecycle
   - Listens for Docker API requests

2. **Docker Client**
   - Command-line interface
   - Communicates with Docker daemon
   - Primary way users interact with Docker

3. **Docker Registry**
   - Stores Docker images
   - Docker Hub (public registry)
   - Private registries

4. **Docker Objects**
   - Images: Read-only templates
   - Containers: Runnable instances of images
   - Networks: Communication between containers
   - Volumes: Persistent data storage

## 4. Installing Docker

### System Requirements
- Windows 10/11 Professional or Enterprise
- macOS 10.15 or newer
- Linux: Ubuntu, Debian, CentOS, Fedora

### Installation Steps

#### For Ubuntu
```bash
# Update package index
sudo apt update

# Install prerequisites
sudo apt install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

# Add Docker's official GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Set up stable repository
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io

# Verify installation
sudo docker run hello-world
```

#### For macOS
1. Download Docker Desktop from the official website
2. Double-click the downloaded .dmg file
3. Drag Docker to Applications
4. Start Docker from Applications

#### For Windows
1. Enable WSL 2 feature
2. Download Docker Desktop for Windows
3. Run the installer
4. Start Docker Desktop
5. Verify installation through PowerShell

## 5. Post-Installation Steps

### Configure Docker to Start on Boot
```bash
sudo systemctl enable docker
```

### Add User to Docker Group (Linux)
```bash
sudo usermod -aG docker $USER
newgrp docker
```

### Verify Installation
```bash
docker --version
docker info
docker run hello-world
```

## 6. Practical Exercises

### Exercise 1: Basic Docker Commands
1. Check Docker version
2. View system-wide information
3. Run hello-world container
4. List all containers

### Exercise 2: Docker Help
1. Explore Docker help system
2. Review common commands
3. Understand command syntax

## 7. Knowledge Check
1. What is the main difference between containers and VMs?
2. Name the core components of Docker architecture
3. Why is containerization beneficial for development teams?
4. What is Docker Hub?
5. Explain the relationship between Docker client and Docker daemon

## Additional Resources
- [Official Docker Documentation](https://docs.docker.com/)
- [Docker Get Started Guide](https://docs.docker.com/get-started/)
- [Docker Hub](https://hub.docker.com/)

## Next Steps
- Explore basic Docker commands
- Learn about container lifecycle
- Understand image management