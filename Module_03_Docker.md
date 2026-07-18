# MODULE 3: DOCKER

## 📚 Overview
Docker revolutionized application deployment through containerization. This module covers Docker fundamentals, images, containers, networking, volumes, and real-world production practices.

---

## 1. CONTAINERIZATION BASICS

### What is a Container?
- **Lightweight virtualization** - Process-level isolation
- **Consistent environment** - Runs same everywhere
- **Package everything** - App + dependencies + runtime
- **Fast startup** - Seconds vs minutes

### Containers vs VMs
```
Virtual Machines          Containers
- Hypervisor              - Docker Engine
- Full OS per VM          - Shared OS kernel
- GB in size              - MB in size
- Minutes to start        - Seconds to start
- Higher resource cost    - Lower resource cost
- More isolation          - Process-level isolation
```

---

## 2. DOCKER ARCHITECTURE

### Docker Engine Components
```
Docker Client (CLI)
  ↓
Docker Daemon (API)
  ↓
├── Container Runtime (containerd)
├── Image Management
├── Network Management
└── Storage Management
  ↓
Operating System Kernel
  ↓
├── Namespaces    (pid, net, ipc, uts, mnt, user)
├── Cgroups       (resource limits)
├── UnionFS       (layered file system)
└── SELinux/AppArmor (security)
```

### Key Concepts
```
Image   - Blueprint (read-only layers)
Container - Running instance of image
Registry - Repository of images
Volume - Data storage (outside container)
Network - Communication between containers
```

---

## 3. DOCKER IMAGES

### Image Management
```bash
# List images
docker images                    # Local images
docker images -a                # Include intermediate images

# Pull image
docker pull ubuntu:22.04         # From Docker Hub
docker pull nginx:latest

# Build image
docker build -t myapp:1.0 .      # Build from Dockerfile
docker build -t myapp:1.0 -f Dockerfile.prod .  # Specific Dockerfile
docker build --no-cache -t myapp:1.0 .  # Rebuild all layers

# Tag image
docker tag myapp:1.0 myapp:latest
docker tag myapp:1.0 myrepo/myapp:1.0

# Search images
docker search ubuntu
docker search --filter stars=100 ubuntu

# Remove image
docker rmi myapp:1.0             # Remove image
docker rmi image1 image2 image3  # Multiple images

# Image history
docker history myapp:1.0         # Show layers
docker inspect myapp:1.0         # Full image details
```

### Dockerfile
```dockerfile
# Multi-stage Dockerfile example
FROM golang:1.20 AS builder

WORKDIR /app
COPY . .
RUN go build -o myapp main.go

FROM ubuntu:22.04

WORKDIR /app
COPY --from=builder /app/myapp .

EXPOSE 8080
CMD ["./myapp"]

# Dockerfile Instructions:
FROM         - Base image
WORKDIR      - Working directory
COPY         - Copy files from host
ADD          - Copy files from host (with URL support)
RUN          - Execute command during build
ENV          - Environment variable
EXPOSE       - Expose port (documentation)
CMD          - Default command
ENTRYPOINT   - Configure container as executable
VOLUME       - Mount point
USER         - Run as user
ARG          - Build argument
LABEL        - Metadata
```

### Build Best Practices
```dockerfile
# Good Dockerfile
FROM ubuntu:22.04

RUN apt-get update && apt-get install -y \
    python3 \
    python3-pip && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

EXPOSE 8000
CMD ["python3", "app.py"]

# Key practices:
# 1. Use specific base image versions
# 2. Minimize layers (combine RUN commands)
# 3. Clean up after installation
# 4. Use .dockerignore
# 5. Copy requirements first (layer caching)
# 6. Multi-stage builds for smaller images
```

---

## 4. CONTAINERS

### Container Lifecycle
```
docker run
  ↓
Container created (docker create)
  ↓
Container started (docker start)
  ↓
Running
  ↓
Paused (docker pause) ↔ Unpaused (docker unpause)
  ↓
Stopped (docker stop)
  ↓
Removed (docker rm)
```

### Container Management
```bash
# Run container
docker run -d --name myapp -p 8080:8000 myapp:1.0
  -d              Detached mode
  --name          Container name
  -p 8080:8000    Port mapping (host:container)
  -e VAR=value    Environment variable
  -v /host:/app   Volume mount
  -it             Interactive terminal
  --rm            Remove after exit
  --cpus=1        CPU limit
  --memory=512m   Memory limit

# List containers
docker ps                        # Running containers
docker ps -a                     # All containers
docker ps -q                     # Only container IDs

# Container info
docker inspect myapp             # Full container details
docker logs myapp                # Container logs
docker logs -f myapp             # Follow logs
docker logs --tail 50 myapp      # Last 50 lines
docker stats myapp               # Live stats
docker top myapp                 # Running processes

# Execute in container
docker exec -it myapp bash       # Interactive shell
docker exec myapp ps aux         # Run command

# Container control
docker start myapp               # Start container
docker stop myapp                # Graceful stop
docker kill myapp                # Force stop
docker restart myapp             # Restart
docker pause myapp               # Pause
docker unpause myapp             # Resume

# Remove container
docker rm myapp                  # Remove
docker rm -f myapp               # Force remove
docker container prune           # Remove stopped
```

### Networking Modes
```bash
# Bridge network (default)
docker run --network bridge myapp

# Host network
docker run --network host myapp

# Container network
docker run --network container:other_app myapp

# Custom network
docker network create mynet
docker run --network mynet myapp
docker network connect mynet myapp
docker network disconnect mynet myapp

# Network commands
docker network ls
docker network inspect mynet
docker network rm mynet
```

---

## 5. DOCKER VOLUMES

### Volume Types
```
Bind Mounts     - Mount host directory into container
  docker run -v /host/path:/container/path myapp

Named Volumes   - Named volume, managed by Docker
  docker volume create mydata
  docker run -v mydata:/app/data myapp

tmpfs Mounts    - Temporary in-memory storage
  docker run --tmpfs /tmp myapp
```

### Volume Management
```bash
# Create volume
docker volume create mydata

# List volumes
docker volume ls

# Volume details
docker volume inspect mydata

# Remove volume
docker volume rm mydata
docker volume prune               # Remove unused

# Bind mount
docker run -v /host/data:/app/data myapp

# Read-only mount
docker run -v /data:/data:ro myapp
```

---

## 6. DOCKER COMPOSE

### Compose File
```yaml
version: '3.8'

services:
  web:
    image: myapp:1.0
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8080:8000"
    environment:
      - DB_HOST=db
      - DB_USER=postgres
    volumes:
      - ./app:/app
      - data:/app/data
    depends_on:
      - db
    networks:
      - mynet
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000"]
      interval: 30s
      timeout: 10s
      retries: 3

  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: mydb
    volumes:
      - db_data:/var/lib/postgresql/data
    networks:
      - mynet

volumes:
  data:
  db_data:

networks:
  mynet:
```

### Docker Compose Commands
```bash
# Start services
docker-compose up                 # Foreground
docker-compose up -d              # Background
docker-compose up --build         # Rebuild images

# Stop services
docker-compose down               # Stop and remove
docker-compose down -v            # Also remove volumes
docker-compose stop               # Stop (keep containers)

# Service management
docker-compose ps                 # Running services
docker-compose logs               # View logs
docker-compose logs -f web        # Follow service logs
docker-compose exec web bash      # Execute in service
docker-compose restart            # Restart all
docker-compose restart web        # Restart specific

# Remove containers/images
docker-compose rm                 # Remove containers
docker-compose rmi                # Remove images
```

---

## 7. REGISTRY & HUB

### Docker Hub
```bash
# Login
docker login
docker logout

# Push image
docker tag myapp:1.0 username/myapp:1.0
docker push username/myapp:1.0

# Pull image
docker pull username/myapp:1.0
docker pull username/myapp          # Latest

# Search
docker search myapp
```

### Private Registry
```bash
# Run private registry
docker run -d -p 5000:5000 --restart=always registry:2

# Push to private registry
docker tag myapp:1.0 localhost:5000/myapp:1.0
docker push localhost:5000/myapp:1.0

# Pull from private registry
docker pull localhost:5000/myapp:1.0
```

---

## 8. DOCKER SECURITY

### Security Best Practices
```dockerfile
# Don't run as root
FROM ubuntu:22.04
RUN useradd -m appuser
USER appuser

# Read-only filesystem
docker run --read-only myapp

# Drop capabilities
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE myapp

# No new privileges
docker run --security-opt=no-new-privileges myapp

# Scan images
docker scan myapp:1.0
```

### Registry Security
```bash
# Private registry with authentication
docker run -d -p 5000:5000 \
  -e REGISTRY_AUTH=htpasswd \
  -e REGISTRY_AUTH_HTPASSWD_REALM="Registry Realm" \
  -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
  -v /auth:/auth \
  registry:2
```

---

## 9. TROUBLESHOOTING

### Common Issues
```bash
# Container won't start
docker logs myapp              # Check logs
docker inspect myapp           # Check config
docker run -it myapp bash      # Run interactively

# Image build fails
docker build --no-cache -t myapp .   # Rebuild
docker build -t myapp . 2>&1 | tee build.log  # Save logs

# Port already in use
docker ps                      # Find running containers
docker port myapp              # Check container ports
docker stop container_id       # Stop conflicting container

# Out of space
docker system df               # Disk usage
docker system prune            # Remove unused

# High memory usage
docker stats                   # Real-time stats
docker run --memory=512m myapp # Limit memory
```

---

## HANDS-ON LABS

### Lab 1: Build and Run Container
```bash
# Create application
mkdir myapp && cd myapp
cat > app.py << 'EOF'
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello():
    return 'Hello from Docker!'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8000)
EOF

cat > requirements.txt << 'EOF'
Flask==2.0.1
EOF

# Create Dockerfile
cat > Dockerfile << 'EOF'
FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
EXPOSE 8000
CMD ["python", "app.py"]
EOF

# Build image
docker build -t myflaskapp:1.0 .

# Run container
docker run -d -p 8000:8000 --name myapp myflaskapp:1.0

# Test
curl http://localhost:8000
```

### Lab 2: Docker Compose Multi-container App
```yaml
# docker-compose.yml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DB_HOST=db
    depends_on:
      - db
    volumes:
      - ./app:/app

  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: mydb
    volumes:
      - db_data:/var/lib/postgresql/data

volumes:
  db_data:
```

---

## INTERVIEW QUESTIONS

### Beginner
1. **What is Docker?**
   - Containerization platform for packaging applications

2. **Difference between image and container?**
   - Image: Blueprintread-only)
   - Container: Running instance

3. **What is a Dockerfile?**
   - Text file defining image build steps

4. **Key Docker commands?**
   - docker build, docker run, docker ps, docker logs

5. **What is volume?**
   - Persistent data storage outside container

### Intermediate
6. **Explain Docker networking?**
   - Bridge (default), Host, Container networks

7. **Multi-stage Dockerfile?**
   - Build in one stage, copy artifacts to another
   - Reduces final image size

8. **Docker security best practices?**
   - Don't run as root
   - Drop capabilities
   - Scan images
   - Use private registries

9. **How to scale containers?**
   - Kubernetes, Docker Swarm, container orchestration

10. **Docker compose vs Docker run?**
    - Compose: Multi-container, configuration file
    - Run: Single container, CLI

### Advanced
11. **Optimize Docker images?**
    - Use specific base image versions
    - Minimize layers
    - Multi-stage builds
    - Use .dockerignore

12. **Container orchestration tools?**
    - Kubernetes (most popular)
    - Docker Swarm
    - Nomad

13. **How to handle secrets in Docker?**
    - Use environment variables
    - Docker secrets
    - External secret management

14. **Container registry security?**
    - Authentication
    - Image scanning
    - Access control

15. **How to monitor containers?**
    - docker stats
    - Container monitoring tools
    - Prometheus + Grafana

---

**Total Interview Questions: 30+**
**Total Labs: 2 comprehensive labs**
