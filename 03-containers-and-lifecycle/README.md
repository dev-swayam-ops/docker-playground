# Module 03: Containers and Lifecycle

Master advanced container lifecycle management and health monitoring.

## What You'll Learn

- Advanced container lifecycle management
- Health checks and monitoring  
- Resource limits and constraints
- Graceful shutdown handling
- Restart policies
- Signal handling in containers

## Prerequisites

- Modules 00-02 completed
- Understanding of container basics

## Key Concepts

### Container States
Created → Running → Paused → Stopped → Removed

### Health Check  
Periodic test verifying container functionality (healthy/unhealthy/starting)

### Resource Constraints
Limits on CPU, memory, disk preventing resource monopolization

## Hands-on Lab: Health Checks & Resource Limits

### Step 1: Build Image with Health Check

```dockerfile
FROM nginx:latest
HEALTHCHECK --interval=10s --timeout=3s --retries=3 \
    CMD curl -f http://localhost/ || exit 1
```

### Step 2: Run with Health Monitoring

```bash
docker build -t healthy-nginx .
docker run -d --name nginx-healthy --health-cmd='curl -f http://localhost/ || exit 1' --health-interval=10s healthy-nginx
docker inspect --format='{{.State.Health.Status}}' nginx-healthy
# Output: healthy
```

### Step 3: Set Resource Limits

```bash
docker run -d --name limited \
  --memory=256m \
  --cpus=0.5 \
  nginx

docker stats limited --no-stream
```

### Step 4: Restart Policies

```bash
docker run -d --name auto-restart \
  --restart=unless-stopped \
  nginx
```

## Validation

- ✅ Configure health checks
- ✅ Set resource limits
- ✅ Implement restart policies
- ✅ Monitor container health

## Cleanup

```bash
docker stop $(docker ps -q)
docker rm $(docker ps -a -q)
docker rmi healthy-nginx
```

## Quick Reference

| Command | Purpose |
|---------|---------|
| `--health-cmd='...'` | Health check command |
| `--memory=256m` | Memory limit |
| `--cpus=0.5` | CPU limit |
| `--restart=always` | Auto-restart |
| `docker pause` | Pause processes |
| `docker unpause` | Resume processes |
| `docker stats` | Resource usage |

---

**Exercises:** [exercises.md](exercises.md)
