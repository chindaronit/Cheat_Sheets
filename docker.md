# 🐳 Docker Cheat Sheet

### 🔹 Docker Basics

```bash
# Check Docker version
docker --version

# Show system-wide information
docker info

# List all running containers
docker ps

# List all containers (including stopped)
docker ps -a

# List images
docker images

# List volumes
docker volume ls

# List networks
docker network ls
```

### 🔹 Container Management

```bash
# Run a container from an image
docker run <image_id>

# Run container with a custom name
docker run --name <container_name> <image_id>

# Run in detached mode
docker run -d <image_id>

# Run interactively with shell access
docker run -it <image_id> /bin/bash

# Start/Stop/Restart a container
docker start <container_id>
docker stop <container_id>
docker restart <container_id>

# Remove a container
docker rm <container_id>

# Remove all stopped containers
docker container prune
```

## 🔹 Image Management

```bash
# Pull an image from Docker Hub
docker pull <image>

# Build an image from Dockerfile
docker build -t <image_name>:<tag> .

# Tag an image
docker tag <image_id> <repo>/<image_name>:<tag>

# Push image to registry
docker push <repo>/<image_name>:<tag>

# Remove an image
docker rmi <image_id>

# Remove unused images
docker image prune
```

## 🔹 Logs & Monitoring

```bash
# View container logs
docker logs <container_id>

# Follow logs in real time
docker logs -f <container_id>

# Check resource usage
docker stats

# Inspect container details
docker inspect <container_id>

# Tail logs with timestamps
docker logs -ft <container_id>

# View container events in real-time
docker events

# Benchmark container performance
docker run --rm --cpus=1 --memory=512m <image> stress --cpu 2

```

### 🔹 Exec & Copy

```bash
# Run a command inside a running container
docker exec -it <container_id> <command>

# Start a shell session inside container
docker exec -it <container_id> /bin/bash
or
docker exec -it <container_id> /bin/sh

# Copy files from host to container
docker cp host_file <container_id>:/path/in/container

# Copy files from container to host
docker cp <container_id>:/path/in/container host_file
```

### 🔹 Volume

```bash
# Create a volume
docker volume create <volume_name>

# Run container with a mounted volume
docker run -v <volume_name>:/path/in/container <image_id>

# Inspect a volume
docker volume inspect <volume_name>

# Remove a volume
docker volume rm <volume_name>

# Remove unused volumes
docker volume prune
```

### 🔹 Networking

```bash
# List networks
docker network ls

# Create a network
docker network create <network_name>

# Connect container to a network
docker network connect <network_name> <container_id>

# Disconnect container from a network
docker network disconnect <network_name> <container_id>

# Inspect a network
docker network inspect <network_name>
```

### 🔹 Docker Compose

```bash
# Start services defined in docker-compose.yml
docker compose up

# Start in detached mode
docker compose up -d

# Stop services
docker compose down

# Stop services and remove volumes
docker compose down -v

# Rebuild and restart
docker compose up --build

# View logs
docker compose logs -f
```

### 🔹 Cleanup & Maintenance

```bash
# Remove all stopped containers
docker container prune

# Remove all unused images
docker image prune -a

# Remove all unused volumes
docker volume prune

# Remove everything not in use
docker system prune -a

# Remove all exited containers
docker rm $(docker ps -aq -f status=exited)

# Remove all dangling images
docker rmi $(docker images -f "dangling=true" -q)

# Remove everything (containers, networks, volumes, images)
docker system prune -a --volumes
```

### 🔹 Resource Limits

```bash
# Limit memory
docker run -m 512m <image>

# Limit CPU (50% of one core)
docker run --cpus=".5" <image>

# Limit restart policy
docker run --restart=always <image>
```

### 🔹 Debugging & Troubleshooting

```bash
# Get process list inside container
docker top <container_id>

# Open a shell inside paused/failing container
docker exec -it <container_id> sh

# Export container filesystem as tar
docker export <container_id> > container.tar

# Import tar as image
docker import container.tar <new_image>

# Show networking details
docker network inspect bridge
```

### 🔹Flags

```bash
#----------- Container Lifecycle -----------#

--rm                # Automatically remove container when it exits
--restart=always    # Restart policy (no, on-failure, always, unless-stopped)
--name mycontainer  # Assign a custom name to the container
--detach , -d       # Run in detached (background) mode
--tty , -t          # Allocate a pseudo-TTY (Teletypewriter. better looking shell)
--interactive , -i  # Keep STDIN open (useful with shells)

#----------- Resource Limits -----------#

--cpus="1.5"        # Limit CPU usage (e.g., 1.5 cores)
--cpu-shares=512    # Relative CPU weighting (default 1024)
--memory=512m       # Limit memory usage
--memory-swap=1g    # Memory + swap limit
--memory-reservation=200m  # Soft memory limit
--pids-limit=100    # Max number of processes inside container

#----------- Network -----------#

--network=bridge    # Attach container to a network (bridge, host, none, or custom)
--hostname=myhost   # Set hostname inside container
--dns=8.8.8.8       # Custom DNS server
--publish 8080:80   # Map host port to container port (host:container)
--publish-all , -P  # Publish all exposed ports to random host ports
--mac-address=XX:XX:XX:XX:XX:XX  # Set custom MAC address

#----------- Volume & Storage -----------#

--volume , -v host_path:container_path  # Mount volume or bind mount
--mount type=bind,source=/src,target=/app  # More advanced mount syntax
--tmpfs /tmp        # Create tmpfs mount inside container
--read-only         # Mount root filesystem as read-only
--device=/dev/sda:/dev/xvda  # Give container access to host device

#----------- Logging & Monitoring -----------#

--log-driver=json-file    # Logging driver (json-file, syslog, journald, fluentd, etc.)
--log-opt max-size=10m    # Limit log file size
--log-opt max-file=3      # Rotate log files
--health-cmd="curl -f http://localhost/ || exit 1"  # Health check
--health-interval=30s     # Time between health checks
--health-retries=3        # Retries before marking unhealthy
--health-timeout=10s      # Max time before health check fails

#----------- Environment & Config -----------#
--env , -e VAR=value   # Set environment variable
--env-file=env.list    # Load env vars from file
--workdir , -w /app    # Set working directory inside container
--entrypoint=/bin/bash # Override image entrypoint


```

`--cpu-shares` is a relative term \
imagine you've run two containers

```bash
docker run -d --cpu-shares=512 myapp1
docker run -d --cpu-shares=1024 myapp2
```

Here, Container A (512) has **half** the CPU share of Container B (1024).

`--entrypoint=/bin/bash` this can be useful to enter the bash of stopped container which get crashing and restarting. (stuck in restart loop)
