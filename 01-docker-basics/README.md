# Module 01: Docker Basics

Master the fundamental Docker commands and workflows. Learn how to run, manage, and interact with containers.

## What You'll Learn

- How to run containers using `docker run`
- How to manage container lifecycle (start, stop, restart)
- How to view container logs and debug issues
- How to inspect container details and configuration
- How to execute commands inside running containers
- How to understand container naming and identification
- How to view resource usage and container processes

## Prerequisites

- Docker installed and running (Module 00)
- Basic terminal/command-line knowledge
- Understanding of what containers and images are
- Access to Docker Hub for pulling images

## Key Concepts

### Container Lifecycle
The states a container goes through: Created → Running → Stopped → Removed. Each state has different characteristics and management options.

### Container ID vs Name
Every container has a unique ID (SHA-256 hash) and can have an optional human-readable name. You can reference containers by either.

### Port Mapping
The process of linking ports from the host machine to ports inside the container. Format: `-p host_port:container_port`.

### Volume Mounting
Connecting directories from your host machine to directories inside the container. Format: `-v host_path:container_path`.

### Image vs Container Instance
A container is a running instance of an image. Multiple containers can run from the same image simultaneously.

### Foreground vs Detached Mode
- **Foreground (-it):** Container runs interactively in your terminal
- **Detached (-d):** Container runs in the background

### Docker Logging
Containers capture stdout and stderr. Use `docker logs` to view the output even after the container exits.

## Hands-on Lab: Managing Your First Container

### Lab Objective
Run an Ubuntu container, interact with it, inspect its details, and manage its lifecycle.

### Step 1: Run an Interactive Container

```bash
# Run Ubuntu container in interactive mode
docker run -it --name my-ubuntu ubuntu:22.04 bash

# You should now be inside the container shell
# Try these commands:
ls /              # List root directory
whoami            # Check current user
cat /etc/os-release  # See OS information
exit              # Exit the container
```

**Expected Output:**
```
root@abc123def456:/#
root
NAME="Ubuntu"
VERSION="22.04.3 LTS"
...
```

**What's Happening:**
- `-it` makes the container interactive with a terminal
- `--name my-ubuntu` gives the container a friendly name
- `ubuntu:22.04` is the image name and version tag
- `bash` is the command to run inside the container

### Step 2: View Container Status

```bash
# List all containers (including stopped)
docker ps -a

# Expected output:
# CONTAINER ID   IMAGE          COMMAND   CREATED        STATUS
# abc123def456   ubuntu:22.04   "bash"    2 minutes ago  Exited (0) 1 minute ago  my-ubuntu
```

### Step 3: Start the Stopped Container

```bash
# Start the container you just exited
docker start my-ubuntu

# Verify it's running
docker ps
```

### Step 4: Execute Command in Running Container

```bash
# Run a command inside the running container without entering shell
docker exec my-ubuntu ls /

# Expected output:
# bin
# boot
# dev
# etc
# ...
```

### Step 5: View Container Logs

```bash
# View all output from the container
docker logs my-ubuntu

# Follow logs in real-time (like tail -f)
docker logs -f my-ubuntu

# Exit with Ctrl+C
```

### Step 6: Inspect Container Details

```bash
# Get detailed JSON configuration of the container
docker inspect my-ubuntu

# Get specific information (e.g., IP address)
docker inspect --format='{{.NetworkSettings.IPAddress}}' my-ubuntu
```

**Expected Output Format:**
```json
{
    "Id": "abc123def456...",
    "Created": "2024-01-15T10:30:00.000Z",
    "State": {
        "Status": "running",
        "Running": true,
        "Paused": false
    },
    "Config": {
        "Hostname": "abc123def456",
        "Image": "ubuntu:22.04"
    }
}
```

### Step 7: Monitor Container Resources

```bash
# View real-time CPU and memory usage
docker stats my-ubuntu

# Expected output:
# CONTAINER ID   NAME        CPU %     MEM USAGE / LIMIT
# abc123def456   my-ubuntu   0.00%     5.23 MiB / 7.727 GiB
```

### Step 8: Stop and Remove Container

```bash
# Stop the running container
docker stop my-ubuntu

# Verify it's stopped
docker ps -a

# Remove the container completely
docker rm my-ubuntu

# Verify it's gone
docker ps -a
```

## Validation

Your understanding is validated when you can:
- ✅ Run a container interactively with `docker run -it`
- ✅ List running and stopped containers with `docker ps` and `docker ps -a`
- ✅ Start a stopped container with `docker start`
- ✅ Execute commands in running containers with `docker exec`
- ✅ View container logs with `docker logs`
- ✅ Inspect container configuration with `docker inspect`
- ✅ Monitor container resources with `docker stats`
- ✅ Stop and remove containers cleanly

## Cleanup

Remove all test containers and images:

```bash
# Stop all running containers
docker stop $(docker ps -q)

# Remove all containers
docker rm $(docker ps -a -q)

# Remove Ubuntu image if you want
docker rmi ubuntu:22.04
```

## Common Mistakes

### ❌ Mistake 1: Forgetting Container Name in Commands
**Symptom:** "Error: No such container"
**Solution:** Use container ID or name exactly as shown in `docker ps -a`

### ❌ Mistake 2: Running Container in Foreground Without -it
**Symptom:** Can't interact with the container
**Solution:** Always use `-it` flags when you need an interactive shell

### ❌ Mistake 3: Trying to Start an Already Running Container
**Symptom:** "Container already in use"
**Solution:** Use `docker restart` to restart, or `docker stop` first

### ❌ Mistake 4: Not Naming Containers Explicitly
**Symptom:** Hard to remember which container ID does what
**Solution:** Always use `--name` flag for clarity

### ❌ Mistake 5: Removing Containers with Running Processes
**Symptom:** "Cannot remove: container is running"
**Solution:** Stop container first with `docker stop` or use `docker rm -f`

## Troubleshooting

### Issue: Container exits immediately
**Solution:** Check logs with `docker logs [container_name]` to see what went wrong

### Issue: Can't connect to container port
**Solution:** Verify port mapping with `docker port [container_name]` and check if container is running

### Issue: "Cannot execute binary file" error
**Solution:** Ensure the image contains the command you're trying to run

### Issue: Too many containers cluttering output
**Solution:** Use `docker rm` to delete old containers, or use `--rm` flag to auto-remove on exit

### Issue: Container uses too much memory
**Solution:** Limit with `docker run --memory=512m image_name`

## Next Steps

You're ready for:

1. **Module 02: Images and Dockerfile** - Learn how to create custom images
2. **Module 03: Containers and Lifecycle** - Master advanced container management
3. **Module 04: Volumes and Storage** - Persist data across containers

## Quick Reference

| Command | Purpose |
|---------|---------|
| `docker run` | Create and run a container |
| `docker ps` | List running containers |
| `docker ps -a` | List all containers |
| `docker start` | Start a stopped container |
| `docker stop` | Stop a running container |
| `docker restart` | Restart a container |
| `docker exec` | Execute command in running container |
| `docker logs` | View container output |
| `docker inspect` | View container details |
| `docker stats` | View resource usage |
| `docker rm` | Remove container |
| `docker rename` | Rename container |

---

**Need Help?**
- Official Docs: https://docs.docker.com/reference/cli/docker/container/
- Previous Module: [00-setup-and-prerequisites](../00-setup-and-prerequisites/)
- Exercises: [exercises.md](exercises.md)
- Solutions: [solutions.md](solutions.md)
