# Module 00: Setup and Prerequisites

Welcome to the Docker learning journey! This module covers everything you need to install and configure Docker on your system.

## What You'll Learn

- How to check system requirements for Docker
- How to install Docker on Windows, macOS, and Linux
- How to verify Docker is installed correctly
- How to configure Docker to run without sudo (Linux)
- How to use Docker commands for the first time
- How to troubleshoot common installation issues

## Prerequisites

- A computer with administrator access
- 4GB RAM minimum (8GB recommended)
- 10GB free disk space
- Internet connection
- Basic command-line knowledge

## Key Concepts

### Docker
A containerization platform that packages applications and dependencies together for consistent deployment across different environments.

### Container
A lightweight, standalone executable package that contains everything needed to run an application (code, runtime, system tools, libraries).

### Docker Engine
The core runtime that manages containers. Includes the daemon, REST API, and CLI.

### Image
A blueprint for creating containers. It's immutable and contains the application code and dependencies.

### Registry
A centralized location where Docker images are stored and distributed (e.g., Docker Hub).

## Hands-on Lab: Installing and Verifying Docker

### Lab Objective
Install Docker on your system and verify it's working correctly.

### Step 1: Download Docker Desktop

**For Windows/macOS:**
1. Visit https://www.docker.com/products/docker-desktop
2. Download Docker Desktop for your OS
3. Run the installer and follow the prompts
4. Restart your computer when installation completes

**For Linux (Ubuntu):**
```bash
# Update package manager
sudo apt-get update

# Install Docker
sudo apt-get install -y docker.io

# Add current user to docker group
sudo usermod -aG docker $USER

# Log out and log back in for group changes to take effect
```

### Step 2: Verify Installation

```bash
# Check Docker version
docker --version

# Expected output:
# Docker version 24.0.0, build abcdef123
```

### Step 3: Test Docker with Hello World

```bash
# Run your first container
docker run hello-world

# Expected output:
# Unable to find image 'hello-world:latest' locally
# latest: Pulling from library/hello-world
# 2db29710123e: Pull complete
# Digest: sha256:abc123...
# Status: Downloaded newer image for hello-world:latest
#
# Hello from Docker!
# This message shows that your installation appears to be working correctly.
# ...
```

### Step 4: Check Docker Info

```bash
# Get detailed Docker information
docker info

# Expected output:
# Client:
#  Version:           24.0.0
#  API version:       1.43
#
# Server:
#  Containers:        1
#  Running:           0
#  ...
```

## Validation

Your Docker installation is successful when:
- ✅ `docker --version` shows a version number
- ✅ `docker run hello-world` executes without errors
- ✅ `docker ps` returns (possibly empty) container list
- ✅ `docker images` shows downloaded images
- ✅ `docker info` displays system information

## Cleanup

Remove the hello-world test container and image:

```bash
# List all containers (including stopped ones)
docker ps -a

# Remove the hello-world container
docker rm <container_id>

# Remove the hello-world image
docker rmi hello-world
```

## Common Mistakes

### ❌ Mistake 1: Docker daemon not running
**Symptom:** "Cannot connect to Docker daemon"
**Solution:** Start Docker Desktop or the Docker daemon service

### ❌ Mistake 2: Permission denied error (Linux)
**Symptom:** "Got permission denied while trying to connect to Docker daemon"
**Solution:** Run `sudo usermod -aG docker $USER` and restart your terminal

### ❌ Mistake 3: Wrong Docker installation
**Symptom:** `docker` command not found
**Solution:** Verify installation path, reinstall Docker, or add Docker to PATH

### ❌ Mistake 4: Insufficient disk space
**Symptom:** "No space left on device" when pulling images
**Solution:** Free up disk space or increase Docker's disk allocation

### ❌ Mistake 5: WSL2 not installed (Windows)
**Symptom:** Docker Desktop won't start on Windows
**Solution:** Install Windows Subsystem for Linux 2 (WSL2) first

## Troubleshooting

### Issue: Docker Desktop won't start
- Check if virtualization is enabled in BIOS
- Restart Docker Desktop
- Check logs: `%AppData%\Docker\log.txt` (Windows)
- Reinstall Docker Desktop if needed

### Issue: `docker run` is very slow
- Check available disk space
- Ensure Docker has sufficient CPU/memory allocation
- Verify internet connection for image downloads

### Issue: Connection timeout when pulling images
- Check your internet connection
- Try pulling from a different registry
- Check Docker Hub status: https://www.docker.com/products/docker-hub

### Issue: Port conflicts
- Check what's using the port: `docker port <container_id>`
- Use a different port: `docker run -p 8081:8080 image_name`

### Issue: Out of memory errors
- Increase Docker's memory allocation in settings
- Remove unused images and containers
- Monitor with: `docker stats`

## Next Steps

You're all set! Proceed to:

1. **Module 01: Docker Basics** - Learn Docker core commands and workflow
2. **Module 02: Images and Dockerfile** - Understand how to create custom images
3. **Module 03: Containers and Lifecycle** - Master container management

## Quick Reference

| Command | Purpose |
|---------|---------|
| `docker --version` | Check Docker version |
| `docker run image_name` | Run a container |
| `docker ps` | List running containers |
| `docker ps -a` | List all containers |
| `docker images` | List downloaded images |
| `docker info` | Display system information |

---

**Need Help?**
- Official Docs: https://docs.docker.com/
- Docker Community: https://community.docker.com/
- Stack Overflow: Tag your questions with `docker`
