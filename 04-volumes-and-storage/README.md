# Module 04: Volumes and Storage

Persist data across containers and manage Docker storage effectively.

## What You'll Learn

- Docker volumes (named, anonymous, host)
- Bind mounts vs named volumes
- Volume drivers and plugins
- Data persistence strategies
- Sharing data between containers
- Backup and restore
- Storage optimization

## Key Concepts

### Volume
Persistent storage outside container filesystem

### Bind Mount
Host directory mapped to container directory

### Named Volume
Managed volume created and tracked by Docker

## Hands-on Lab

```bash
# Create volume
docker volume create my-data

# Run with volume
docker run -d --name app1 --mount source=my-data,target=/data alpine sleep 3600
docker exec app1 sh -c 'echo "data" > /data/file.txt'

# Data persists after container removal
docker stop app1 && docker rm app1
docker run -d --name app2 --mount source=my-data,target=/data alpine sleep 3600
docker exec app2 cat /data/file.txt  # Output: data
```

## Cleanup

```bash
docker volume prune
docker volume rm my-data
```

## Quick Reference

| Command | Purpose |
|---------|---------|
| `docker volume create [name]` | Create volume |
| `docker volume ls` | List volumes |
| `docker volume rm [name]` | Remove volume |
| `docker run -v [name]:/path` | Mount volume |
| `docker run --mount source=name,target=/path` | Mount (verbose) |
| `docker volume inspect [name]` | View details |

---

**Exercises:** [exercises.md](exercises.md)
