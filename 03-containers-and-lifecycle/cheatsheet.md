# Cheatsheet: Containers and Lifecycle

## Health Checks

```bash
# In Dockerfile
HEALTHCHECK --interval=10s --timeout=3s --retries=3 CMD curl -f http://localhost/

# In docker run
docker run --health-cmd='curl -f http://localhost/' --health-interval=10s nginx
```

## Resource Limits

```bash
# Memory limit
docker run --memory=512m --memory-swap=1g nginx

# CPU limit
docker run --cpus=1.5 --cpuset-cpus=0,1 nginx

# Combined
docker run --memory=512m --cpus=0.5 nginx
```

## Restart Policies

```bash
# Always restart
docker run --restart=always nginx

# Unless manually stopped
docker run --restart=unless-stopped nginx

# On failure with max retries
docker run --restart=on-failure:5 nginx

# No restart (default)
docker run --restart=no nginx
```

## Container Lifecycle Commands

| Command | Purpose |
|---------|---------|
| `docker pause [container]` | Pause processes |
| `docker unpause [container]` | Resume processes |
| `docker kill [container]` | Force kill (SIGKILL) |
| `docker stop [container]` | Graceful stop (SIGTERM) |
| `docker start [container]` | Start stopped container |
| `docker restart [container]` | Restart container |

## Monitoring

```bash
# Resource usage live
docker stats [container]

# Processes inside container
docker top [container]

# Container events
docker events --filter container=[container]

# Health status
docker inspect --format='{{.State.Health.Status}}' [container]
```

## Signal Handling

| Signal | Command | Behavior |
|--------|---------|----------|
| SIGTERM | docker stop | Graceful shutdown (15s grace) |
| SIGKILL | docker kill | Force termination (immediate) |
| SIGPAUSE | docker pause | Freeze processes |
| SIGUNPAUSE | docker unpause | Resume processes |

