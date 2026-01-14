# Cheatsheet: Setup and Prerequisites

Quick reference for Docker installation and setup commands.

## Installation Commands

| Command | Purpose | Example |
|---------|---------|---------|
| Visit docker.com | Download Docker Desktop | https://www.docker.com/products/docker-desktop |
| `apt-get update` | Update package lists (Linux) | `sudo apt-get update` |
| `apt-get install docker.io` | Install Docker (Ubuntu/Debian) | `sudo apt-get install -y docker.io` |
| `yum install docker` | Install Docker (CentOS/RHEL) | `sudo yum install -y docker` |
| `systemctl start docker` | Start Docker daemon (Linux) | `sudo systemctl start docker` |
| `systemctl enable docker` | Enable Docker on boot (Linux) | `sudo systemctl enable docker` |

---

## Verification Commands

| Command | Purpose | Output |
|---------|---------|--------|
| `docker --version` | Check Docker CLI version | Docker version 24.0.0, build abcdef123 |
| `docker version` | Detailed version info (client + server) | Client and Server version details |
| `docker info` | Complete Docker system information | Containers, Images, Storage info |
| `docker ps` | List running containers | Table of containers |
| `docker ps -a` | List all containers (running + stopped) | All containers on system |
| `docker images` | List all images | All downloaded images |
| `docker stats` | Real-time container resource usage | CPU, Memory, Network I/O |

---

## Docker Daemon Management

| Command | Purpose | OS |
|---------|---------|-----|
| Open Docker Desktop | Start daemon (GUI) | Windows/macOS |
| `sudo systemctl start docker` | Start daemon service | Linux |
| `sudo systemctl stop docker` | Stop daemon service | Linux |
| `sudo systemctl restart docker` | Restart daemon service | Linux |
| `sudo systemctl status docker` | Check daemon status | Linux |
| `sudo systemctl enable docker` | Auto-start on boot | Linux |

---

## Linux-Specific Setup

| Command | Purpose | Details |
|---------|---------|---------|
| `sudo groupadd docker` | Create docker group | Only if it doesn't exist |
| `sudo usermod -aG docker $USER` | Add user to docker group | Grants Docker socket access |
| `newgrp docker` | Activate group in current session | Alternative: log out and in |
| `groups $USER` | Check user's groups | Verify docker is in the list |

---

## Container Management

| Command | Purpose | Example |
|---------|---------|---------|
| `docker run [IMAGE]` | Create and run a container | `docker run hello-world` |
| `docker run --name [NAME] [IMAGE]` | Run with custom name | `docker run --name myapp ubuntu` |
| `docker run -d [IMAGE]` | Run in background (detached) | `docker run -d nginx` |
| `docker run -it [IMAGE] bash` | Run interactively with terminal | `docker run -it ubuntu bash` |
| `docker run -p [HOST]:[PORT] [IMAGE]` | Map ports | `docker run -p 8080:80 nginx` |
| `docker ps` | List running containers | Shows running containers only |
| `docker ps -a` | List all containers | Shows all containers |
| `docker rm [CONTAINER_ID]` | Remove container | `docker rm abc123def456` |
| `docker rm $(docker ps -a -q)` | Remove all containers | Dangerous - use carefully! |

---

## Image Management

| Command | Purpose | Example |
|---------|---------|---------|
| `docker images` | List all images | Shows local image repository |
| `docker pull [IMAGE]` | Download image from registry | `docker pull ubuntu:22.04` |
| `docker rmi [IMAGE]` | Remove image | `docker rmi hello-world` |
| `docker rmi [IMAGE_ID]` | Remove by image ID | `docker rmi abc123def456` |
| `docker image prune` | Remove unused images | Cleans up dangling images |
| `docker tag [IMAGE] [NEW_NAME]` | Create image alias | `docker tag ubuntu myubuntu` |

---

## Cleanup Commands

| Command | Purpose | Warning |
|---------|---------|---------|
| `docker rm [CONTAINER_ID]` | Remove specific container | Container must be stopped first |
| `docker container prune` | Remove all stopped containers | ⚠️ Removes all stopped containers |
| `docker rmi [IMAGE]` | Remove specific image | ⚠️ Image must not be in use |
| `docker image prune` | Remove dangling images | Only removes unused layers |
| `docker system prune` | Remove unused containers/images | ⚠️ Removes all unused items |
| `docker system prune -a` | Remove all unused items | ⚠️ Very aggressive cleanup |

---

## Troubleshooting Commands

| Command | Purpose | Use When |
|---------|---------|----------|
| `docker logs [CONTAINER_ID]` | View container output | Container isn't behaving right |
| `docker inspect [CONTAINER]` | Detailed container info | Need deep diagnostics |
| `docker exec [CONTAINER] [CMD]` | Run command in container | Container is running |
| `docker stats` | Live resource monitoring | Checking CPU/memory usage |
| `docker top [CONTAINER]` | Processes inside container | See what's running |

---

## File Paths Reference

| Location | OS | Purpose |
|----------|-----|---------|
| `%AppData%\Docker\log.txt` | Windows | Docker Desktop logs |
| `~/.docker/config.json` | macOS/Linux | Docker client config |
| `/var/lib/docker/` | Linux | Docker data directory |
| `/var/run/docker.sock` | Linux | Docker daemon socket |

---

## System Requirements Checklist

| Requirement | Minimum | Recommended |
|-------------|---------|-------------|
| RAM | 4 GB | 8 GB or more |
| Disk Space | 10 GB | 20 GB or more |
| Virtualization | Required (VT-x/AMD-V) | Enabled in BIOS |
| WSL 2 (Windows) | Required for Windows 11 | Latest version |

---

## Help Commands

| Command | Purpose |
|---------|---------|
| `docker --help` | All Docker commands |
| `docker [COMMAND] --help` | Specific command help |
| `docker run --help` | Docker run options |
| `docker ps --help` | Docker ps options |

---

## Common Flags Quick Reference

| Flag | Short | Type | Purpose |
|------|-------|------|---------|
| `--detach` | `-d` | bool | Run in background |
| `--interactive` | `-i` | bool | Keep STDIN open |
| `--tty` | `-t` | bool | Allocate pseudo-TTY |
| `--name` | - | string | Container name |
| `--publish` | `-p` | string | Port mapping (host:container) |
| `--environment` | `-e` | string | Environment variable |
| `--volume` | `-v` | string | Mount volume/directory |
| `--rm` | - | bool | Auto-remove on exit |

---

## Status/State Reference

| Status | Meaning | Action |
|--------|---------|--------|
| Running | Container actively executing | Use `docker stop` to halt |
| Exited | Container stopped (not removed) | Use `docker restart` or `docker rm` |
| Paused | Container suspended | Use `docker unpause` to resume |
| Created | Container made but not started | Use `docker start` to begin |

---

## Useful Tips

💡 **Pro Tips:**
- Use `docker ps -a` to see all containers before cleanup
- Use `--name` flag for easy container identification
- Use `-d` flag to run services (nginx, databases, etc.)
- Use `-it` flags together for interactive shells
- Use `docker logs` to debug container issues

⚠️ **Safety Tips:**
- Always use `docker ps -a` before removing containers
- Test `docker rm` on one container before using prune
- Back up important data before cleanup operations
- Never use `docker system prune -a` lightly

🔍 **Debugging Tips:**
- Use `docker logs [CONTAINER_ID]` for error messages
- Use `docker exec -it [CONTAINER] bash` to enter running container
- Use `docker stats` to monitor resource usage
- Use `docker inspect [CONTAINER]` for detailed configuration

---

## Quick Start Command Sequence

```bash
# Install (Windows/macOS - use GUI installer)
# Linux:
sudo apt-get update
sudo apt-get install -y docker.io

# Start daemon (Linux)
sudo systemctl start docker

# Verify installation
docker --version
docker ps

# Run first container
docker run hello-world

# List images and containers
docker images
docker ps -a

# Cleanup
docker rm <container_id>
docker rmi hello-world
```

---

## Resources

- **Official Documentation:** https://docs.docker.com/
- **Docker Hub:** https://hub.docker.com/
- **Community Support:** https://community.docker.com/
- **Issue Reports:** https://github.com/moby/moby/issues

---

**Last Updated:** January 2026  
**Docker Version:** 24.0+  
**Validity:** Current for Docker Desktop and Docker CE
