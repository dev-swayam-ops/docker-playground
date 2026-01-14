# Cheatsheet: Docker Basics

Quick command reference for managing containers.

## Container Running Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `docker run [IMAGE]` | Create and run container | `docker run ubuntu:22.04` |
| `docker run -d [IMAGE]` | Run in background (detached) | `docker run -d nginx` |
| `docker run -it [IMAGE]` | Run interactively with terminal | `docker run -it ubuntu bash` |
| `docker run --name [NAME] [IMAGE]` | Run with custom name | `docker run --name myapp ubuntu` |
| `docker run -p [HOST]:[CONTAINER] [IMAGE]` | Map ports | `docker run -p 8080:80 nginx` |
| `docker run -e [VAR]=[VALUE] [IMAGE]` | Set environment variable | `docker run -e DB_HOST=localhost ubuntu` |
| `docker run --memory [SIZE] [IMAGE]` | Limit memory | `docker run --memory=512m nginx` |
| `docker run --rm [IMAGE]` | Auto-remove on exit | `docker run --rm ubuntu echo "Hello"` |

---

## Container Lifecycle Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `docker ps` | List running containers | `docker ps` |
| `docker ps -a` | List all containers | `docker ps -a` |
| `docker ps -q` | List container IDs only | `docker ps -q` |
| `docker ps -a -q` | List all container IDs | `docker ps -a -q` |
| `docker start [CONTAINER]` | Start stopped container | `docker start myapp` |
| `docker stop [CONTAINER]` | Stop running container | `docker stop myapp` |
| `docker restart [CONTAINER]` | Restart container | `docker restart myapp` |
| `docker pause [CONTAINER]` | Pause container | `docker pause myapp` |
| `docker unpause [CONTAINER]` | Resume paused container | `docker unpause myapp` |
| `docker kill [CONTAINER]` | Force stop container | `docker kill myapp` |

---

## Container Management Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `docker rm [CONTAINER]` | Remove container | `docker rm myapp` |
| `docker rm -f [CONTAINER]` | Force remove running container | `docker rm -f myapp` |
| `docker rm $(docker ps -a -q)` | Remove all containers | `docker rm $(docker ps -a -q)` |
| `docker rename [OLD] [NEW]` | Rename container | `docker rename old-name new-name` |
| `docker exec [CONTAINER] [CMD]` | Run command in container | `docker exec myapp bash` |
| `docker exec -it [CONTAINER] bash` | Interactive shell in container | `docker exec -it myapp bash` |

---

## Logging and Monitoring Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `docker logs [CONTAINER]` | View container logs | `docker logs myapp` |
| `docker logs -f [CONTAINER]` | Follow logs (stream) | `docker logs -f myapp` |
| `docker logs --tail [N] [CONTAINER]` | View last N lines | `docker logs --tail 50 myapp` |
| `docker logs --since [TIME] [CONTAINER]` | Logs since time | `docker logs --since 5m myapp` |
| `docker logs --timestamps [CONTAINER]` | Add timestamps to logs | `docker logs --timestamps myapp` |
| `docker stats` | View resource usage (live) | `docker stats` |
| `docker stats --no-stream` | Resource usage snapshot | `docker stats --no-stream` |
| `docker top [CONTAINER]` | View processes in container | `docker top myapp` |

---

## Inspection and Information Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `docker inspect [CONTAINER]` | Full container details (JSON) | `docker inspect myapp` |
| `docker inspect --format='...' [CONTAINER]` | Extract specific data | `docker inspect --format='{{.State.Status}}' myapp` |
| `docker port [CONTAINER]` | Show port mappings | `docker port myapp` |
| `docker history [IMAGE]` | View image layers | `docker history ubuntu:22.04` |
| `docker diff [CONTAINER]` | Show filesystem changes | `docker diff myapp` |

---

## Useful Format Strings for docker inspect

| Format | Returns | Example |
|--------|---------|---------|
| `{{.Id}}` | Full container ID | `sha256:abc123...` |
| `{{.Name}}` | Container name (with /) | `/myapp` |
| `{{.State.Status}}` | Container status | `running` |
| `{{.State.Running}}` | Is running (true/false) | `true` |
| `{{.State.ExitCode}}` | Exit code | `0` |
| `{{.Config.Image}}` | Image name | `ubuntu:22.04` |
| `{{.Config.Cmd}}` | Default command | `[bash]` |
| `{{.NetworkSettings.IPAddress}}` | Container IP | `172.17.0.2` |
| `{{.HostConfig.Memory}}` | Memory limit (bytes) | `0` |
| `{{.Created}}` | Creation time | `2024-01-15T10:30:00...` |

---

## Common Flag Combinations

| Use Case | Command | Notes |
|----------|---------|-------|
| Interactive shell | `docker run -it ubuntu bash` | `-it` required for terminal |
| Background service | `docker run -d nginx` | `-d` for detached mode |
| With port mapping | `docker run -d -p 8080:80 nginx` | Expose container port |
| With volume mount | `docker run -v /host:/container ubuntu` | Share directories |
| With environment vars | `docker run -e KEY=value ubuntu` | Pass configuration |
| With memory limit | `docker run --memory=512m ubuntu` | Restrict resource usage |
| Auto-cleanup | `docker run --rm ubuntu echo "hi"` | Remove after exit |
| Named container | `docker run --name myapp ubuntu` | Easy reference |

---

## Container State Reference

| State | Meaning | Next Action |
|-------|---------|-------------|
| Created | Container exists but hasn't started | `docker start` |
| Running | Container is actively executing | `docker stop`, `docker pause` |
| Paused | Container is suspended | `docker unpause` |
| Exited | Container stopped (exit code shown) | `docker start`, `docker restart`, `docker rm` |
| Removed | Container deleted from system | Can't perform actions |
| Dead | Error state (rare) | `docker rm` |

---

## Debugging Quick Tips

| Problem | Solution | Command |
|---------|----------|---------|
| Container won't start | Check logs | `docker logs [container]` |
| Can't connect to port | Verify mapping | `docker port [container]` |
| Container using too much CPU | Check processes | `docker top [container]` |
| Container using too much memory | Check stats | `docker stats [container]` |
| Need to enter running container | Use exec | `docker exec -it [container] bash` |
| Container exits too fast | Check exit code | `docker inspect --format='{{.State.ExitCode}}'` |
| Can't find container | Check all containers | `docker ps -a` |

---

## Command Chaining Examples

### Stop and Remove All Containers

```bash
# Stop all running containers
docker stop $(docker ps -q)

# Remove all containers
docker rm $(docker ps -a -q)

# Combine: stop and remove in one command
docker rm -f $(docker ps -a -q)
```

### Get Specific Information

```bash
# Get all container names
docker ps -a --format '{{.Names}}'

# Get container IDs and names
docker ps -a --format 'table {{.ID}}\t{{.Names}}'

# Get running containers and their ports
docker ps --format 'table {{.Names}}\t{{.Ports}}'
```

### Monitor Specific Container

```bash
# Watch container logs continuously
docker logs -f myapp

# Watch resource usage
docker stats myapp

# Watch processes in container
watch docker top myapp  # (Linux/macOS only)
```

---

## Common Flags Reference

| Flag | Short | Type | Description |
|------|-------|------|-------------|
| `--all` | `-a` | bool | Show all containers (including stopped) |
| `--quiet` | `-q` | bool | Only display container IDs |
| `--interactive` | `-i` | bool | Keep STDIN open |
| `--tty` | `-t` | bool | Allocate pseudo-TTY |
| `--detach` | `-d` | bool | Run in background |
| `--name` | - | string | Assign name to container |
| `--publish` | `-p` | string | Publish ports (host:container) |
| `--expose` | - | int | Expose port inside container |
| `--env` | `-e` | string | Set environment variable |
| `--volume` | `-v` | string | Mount volume/directory |
| `--memory` | - | string | Memory limit (e.g., 512m, 1g) |
| `--cpus` | - | string | CPU limit (e.g., 0.5, 1.0) |
| `--restart` | - | string | Restart policy (no, always, unless-stopped, on-failure) |
| `--rm` | - | bool | Auto-remove when container exits |
| `--format` | - | string | Pretty-print output |
| `--follow` | `-f` | bool | Follow output (logs) |
| `--tail` | - | int | Number of lines to show |
| `--timestamps` | - | bool | Show timestamps in output |

---

## Quick Troubleshooting

**"No such container"**
```bash
# List all containers to find exact name
docker ps -a
```

**"Cannot connect to Docker daemon"**
```bash
# Start Docker
# Windows/macOS: Open Docker Desktop
# Linux: sudo systemctl start docker
```

**"Permission denied"**
```bash
# Linux: Add user to docker group
sudo usermod -aG docker $USER
```

**"Port already in use"**
```bash
# Use different port
docker run -p 8081:80 nginx  # Use 8081 instead of 8080
```

**"Container exited immediately"**
```bash
# Check why it exited
docker logs [container_name]
```

---

## Useful Aliases (Optional)

Add to your `.bashrc` or `.zshrc`:

```bash
# Shorter commands
alias dps='docker ps'
alias dpsa='docker ps -a'
alias dls='docker ps -a'  # List containers
alias drm='docker rm'
alias drun='docker run'
alias dlog='docker logs'
alias dinsp='docker inspect'

# Common combinations
alias drmall='docker rm $(docker ps -a -q)'  # Remove all containers
alias dstopall='docker stop $(docker ps -q)' # Stop all running
alias dkillall='docker kill $(docker ps -q)' # Kill all running
```

---

**Need more info?**
- Official Command Reference: https://docs.docker.com/reference/cli/docker/
- Previous Module: [00-setup-and-prerequisites](../00-setup-and-prerequisites/)
- Exercises: [exercises.md](exercises.md)
- Solutions: [solutions.md](solutions.md)

---

**Last Updated:** January 2026
**Docker Version:** 24.0+
