# Solutions: Setup and Prerequisites

Detailed solutions with explanations for all exercises.

---

## Solution 1: System Requirements Check

### Commands & Output

**Windows:**
```bash
systeminfo
```

Look for these lines in the output:
- OS Name: Windows 11
- System Type: x64-based PC
- Total Physical Memory: 16384 MB

**macOS:**
```bash
# Check macOS version
system_profiler SPSoftwareDataType

# Check RAM
system_profiler SPHardwareDataType | grep Memory

# Check disk space
df -h
```

**Linux:**
```bash
# Check OS and kernel
uname -a

# Check RAM (in human-readable format)
free -h

# Check disk space
df -h
```

### Expected Output Example (Linux)
```
$ free -h
              total        used        free
Mem:          15Gi       8.2Gi       6.8Gi

$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1       100G   45G   55G  45% /
```

### Explanation
This exercise helps you:
- Understand your system's current capabilities
- Ensure you meet Docker's minimum requirements (4GB RAM, 10GB disk)
- Document baseline performance metrics for future reference

---

## Solution 2: Docker Installation

### Step-by-Step Installation

**Windows/macOS:**
1. Visit https://www.docker.com/products/docker-desktop
2. Click "Download for Windows" or "Download for Mac"
3. Run the downloaded installer
4. Follow the installation wizard (accept defaults)
5. Complete installation and restart your computer
6. Docker Desktop should auto-start

**Linux (Ubuntu/Debian):**
```bash
# Update package lists
sudo apt-get update

# Install Docker
sudo apt-get install -y docker.io

# Verify installation
docker --version

# Start Docker daemon (if not running)
sudo systemctl start docker

# Enable Docker to start on boot
sudo systemctl enable docker
```

**Linux (CentOS/RHEL):**
```bash
# Install Docker
sudo yum install -y docker

# Start Docker daemon
sudo systemctl start docker

# Enable Docker to start on boot
sudo systemctl enable docker

# Verify installation
docker --version
```

### Explanation
- **Windows/macOS:** Docker Desktop includes the Docker daemon, CLI, and additional tools in one package
- **Linux:** Docker is installed as a service that must be started and can be configured to run on boot
- The installation process varies by OS because each has different system-level requirements

---

## Solution 3: Verify Docker Installation

### Command
```bash
docker --version
```

### Expected Output
```
Docker version 24.0.0, build abcdef123
```
(Version number will be different depending on when you installed)

### What This Tells You
- Docker CLI is properly installed
- Your PATH environment variable includes the Docker binary
- Docker is accessible from your terminal/command prompt

### If It Fails
- **Error:** "docker: command not found"
  - **Solution:** Docker is not installed or not in PATH
  - Reinstall Docker or add it to your system PATH

---

## Solution 4: Check Docker Daemon Status

### Commands
```bash
# Check if Docker daemon is running
docker ps

# Start Docker daemon if needed
# Windows/macOS: Open Docker Desktop application
# Linux: sudo systemctl start docker
```

### Expected Output
```
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

An empty list means no containers are running, which is correct for a fresh installation.

### Explanation
- `docker ps` requires the Docker daemon to be running
- The daemon is the background service that manages containers
- On Windows/macOS, Docker Desktop manages the daemon for you
- On Linux, you need to manually start/enable the daemon service

---

## Solution 5: Run Your First Container

### Command
```bash
docker run hello-world
```

### Expected Output
```
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
2db29710123e: Pull complete
Digest: sha256:1a6d0c86432c7fa37b9f0a7d3e4d4d3f9d3a0c3b0c3a3f0b0c3a0c3a0c3a0c3
Status: Downloaded newer image for hello-world:latest

Hello from Docker!

This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://www.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/engine/userguide/
```

### Explanation
What happens in this process:
1. Docker searches for the `hello-world` image locally
2. Since it doesn't exist, Docker pulls it from Docker Hub (the default registry)
3. Docker creates a new container from this image
4. The container runs and outputs the greeting message
5. The container automatically exits when done

---

## Solution 6: Inspect Downloaded Images

### Command
```bash
docker images
```

### Expected Output
```
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
hello-world   latest    d2c94e258d87   10 months ago   13.3kB
```

### Detailed Output with Explanation
| Column | Meaning | Example |
|--------|---------|---------|
| REPOSITORY | Image name | hello-world |
| TAG | Version/variant | latest |
| IMAGE ID | Unique identifier (SHA-256 hash) | d2c94e258d87 |
| CREATED | When image was created | 10 months ago |
| SIZE | Image file size | 13.3kB |

### To Get More Details
```bash
# Get full image ID
docker images --no-trunc

# Get images with specific filtering
docker images hello-world

# Format output as JSON
docker images --format "table {{.Repository}}\t{{.Size}}"
```

### Explanation
- The `hello-world` image is very small (13.3kB) because it only prints text
- Real application images (Ubuntu, Node.js, etc.) are much larger
- Images are stored in Docker's local image cache
- Multiple images can have the same name but different tags (versions)

---

## Solution 7: Get Detailed Docker Information

### Command
```bash
docker info
```

### Expected Output Structure
```
Client:
 Version:           24.0.0
 API version:       1.43
 Go version:        go1.20.0
 Git commit:        abcdef123
 Built:             Wed Dec 13 11:42:28 2023

Server:
 Containers:        1
  Running:          0
  Paused:           0
  Stopped:          1
 Images:            1
 Server Version:    24.0.0
 Storage Driver:    overlay2
  Backing Filesystem: extfs
 Total Memory:      15.50GB
 Name:              docker-desktop
 ID:                ABCD:EFGH:IJKL:MNOP
```

### Key Information Breakdown

| Field | Meaning |
|-------|---------|
| Client Version | Docker CLI version on your computer |
| Server Version | Docker daemon version |
| Containers | Total containers on your system |
| Running | Actively running containers |
| Stopped | Stopped but not removed containers |
| Images | Total images in local cache |
| Storage Driver | How Docker stores images/containers (usually overlay2) |
| Total Memory | Memory available to Docker |

### To Extract Specific Information
```bash
# Just show container count
docker info | grep "Containers:"

# Show storage driver
docker info | grep "Storage Driver:"

# Show all running containers
docker ps
```

### Explanation
- `docker info` is the comprehensive diagnostic command
- It shows both client (your CLI) and server (daemon) information
- Storage Driver tells you how Docker manages filesystem layers
- This information is useful for troubleshooting and capacity planning

---

## Solution 8: Configure Docker for Linux

### Full Process

```bash
# Step 1: Create docker group (if it doesn't exist)
sudo groupadd docker

# Step 2: Add your user to docker group
sudo usermod -aG docker $USER

# Step 3: Apply group changes (option 1 - activate group for current session)
newgrp docker

# OR Step 3 (option 2 - log out and log back in)
# Just close and reopen your terminal

# Step 4: Verify it works without sudo
docker ps
docker run hello-world
```

### Expected Output
After step 4, you should see:
```
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```
(No errors and no sudo required!)

### Verification Commands
```bash
# Check if your user is in docker group
groups $USER

# Expected output includes: docker

# Test without sudo
docker run hello-world

# Should NOT require password or sudo
```

### Why This Works
- By default, Docker daemon runs as root (for security)
- Only root can access the Docker socket
- Adding your user to the docker group grants socket access
- This avoids typing `sudo` for every Docker command

### ⚠️ Security Note
- Users in the docker group have privileges equivalent to root
- Only add trusted users to the docker group
- Be cautious about who has docker access on shared systems

---

## Solution 9: Explore Docker Help

### Command Structure
```bash
docker --help                # General Docker help
docker <command> --help      # Help for specific command
docker <command> <subcommand> --help  # Help for subcommand
```

### Key Flags Reference

```bash
# View all available flags
docker run --help
```

| Flag | Short | Purpose | Example |
|------|-------|---------|---------|
| `--interactive` | `-i` | Keep STDIN open even if not attached | `docker run -i ubuntu bash` |
| `--tty` | `-t` | Allocate a pseudo-TTY (terminal) | `docker run -t ubuntu` |
| `--detach` | `-d` | Run in background | `docker run -d nginx` |
| `--name` | - | Assign a name to container | `docker run --name myapp ubuntu` |
| `--publish` | `-p` | Map ports (host:container) | `docker run -p 8080:80 nginx` |
| `--environment` | `-e` | Set environment variables | `docker run -e KEY=VALUE ubuntu` |
| `--volume` | `-v` | Mount a volume/directory | `docker run -v /path:/container/path ubuntu` |

### Common Command Help Requests
```bash
# Docker basics
docker --help                  # All commands
docker run --help             # Running containers
docker ps --help              # Listing containers
docker images --help          # Managing images
docker logs --help            # Viewing container logs
docker exec --help            # Running commands in containers
```

### Understanding Help Output Format
```
Usage:    docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

OPTIONS section shows:
  --flag, -f TYPE              Description of what flag does
  --flag TYPE                  Type is usually string, bool, int, etc.
```

### Explanation
- Docker help is comprehensive and always accurate
- Using `--help` is faster than searching online
- Learning to read help pages makes you independent
- Each command can have different options/flags

---

## Solution 10: Cleanup After Testing

### Full Cleanup Process

```bash
# Step 1: List all containers (including stopped ones)
docker ps -a

# Output example:
# CONTAINER ID   IMAGE         COMMAND    CREATED        STATUS
# a1b2c3d4e5f6   hello-world   "/hello"   2 minutes ago  Exited (0) 1 minute ago

# Step 2: List all images
docker images

# Output example:
# REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
# hello-world   latest    d2c94e258d87   10 months ago   13.3kB

# Step 3: Remove the container (use the CONTAINER ID)
docker rm a1b2c3d4e5f6

# Step 4: Remove the image
docker rmi hello-world

# Alternative: Remove by image name
docker rmi hello-world:latest

# Step 5: Verify cleanup
docker ps -a    # Should be empty
docker images   # Should be empty
```

### Advanced Cleanup Commands

```bash
# Remove all stopped containers at once
docker container prune

# Remove all dangling images (unused images)
docker image prune

# Remove all unused containers and images
docker system prune

# Remove everything including volumes (careful!)
docker system prune -a --volumes
```

### Understanding the Output

When you remove a container:
```
a1b2c3d4e5f6
```
This hash is the container ID - it confirms the container was removed.

When you remove an image:
```
Untagged: hello-world:latest
Untagged: hello-world@sha256:...
Deleted: sha256:d2c94e258d87...
Deleted: sha256:1a1d0c86432c7...
```
Shows the image and its layers being deleted.

### Why Cleanup Matters
- **Disk space:** Images and containers consume disk space
- **Organization:** Keeping your system clean prevents confusion
- **Security:** Removing old images reduces attack surface
- **Performance:** Fewer images means faster `docker images` output

### Explanation
- Containers are stopped but not removed by default
- You must explicitly remove containers and images
- `docker ps` shows only running containers
- `docker ps -a` shows all containers (running + stopped)
- Always use `docker ps -a` to see the full picture before cleanup

---

## Summary of Key Concepts

| Concept | What It Is | Where You Use It |
|---------|-----------|------------------|
| Image | Blueprint/template for containers | `docker images`, `docker pull` |
| Container | Running instance of an image | `docker ps`, `docker run` |
| Daemon | Background Docker service | Must be running for any docker commands |
| Registry | Central image repository (Docker Hub) | `docker pull`, `docker push` |
| Docker Group | Linux user group for Docker access | Linux systems only |

---

## Tips for Future Learning

✅ **Do:**
- Save command outputs for reference
- Read error messages - they usually tell you what's wrong
- Use `docker <command> --help` whenever unsure
- Practice these commands multiple times
- Explore variations of the commands

❌ **Don't:**
- Skip the restart step during installation
- Assume Docker is running without checking
- Give docker group access to untrusted users
- Delete images you might need later (at least initially)

---

**Next Steps:**
1. Review each solution and compare with your work
2. Try variations: `docker run ubuntu bash`, `docker run -it alpine`
3. Move to Module 01: Docker Basics for the next chapter
4. Use [cheatsheet.md](cheatsheet.md) for quick command reference

