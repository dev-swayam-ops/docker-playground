# Module 05: Networking in Docker

Connect containers and manage Docker networks securely.

## What You'll Learn

- Docker networking modes (bridge, host, overlay, none)
- Creating and managing custom networks
- Container DNS and service discovery
- Port publishing and communication
- Network isolation and security
- Inter-container networking

## Key Concepts

**Bridge Network:** Default isolated network where containers get IPs
**Host Network:** Container shares host network (no isolation)
**Overlay Network:** Multi-host networking for Docker Swarm

## Lab

```bash
# Create network
docker network create my-network

# Run containers on network
docker run -d --name web --network my-network nginx
docker run -d --name db --network my-network alpine sleep 3600

# Test communication
docker exec web ping db
# Works due to DNS in custom bridge network
```

## Quick Commands

| Command | Purpose |
|---------|---------|
| `docker network create [name]` | Create network |
| `docker network ls` | List networks |
| `docker run --network [name]` | Run on network |
| `docker network inspect [name]` | View network |
| `docker network connect [net] [container]` | Add container |
| `docker network disconnect [net] [container]` | Remove container |
| `docker run -p [host]:[container]` | Publish port |

---

**Exercises:** [exercises.md](exercises.md)
