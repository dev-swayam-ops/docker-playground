## Cheatsheet

| Command | Purpose |
|---------|---------|
| `docker run --restart=always [image]` | Auto-restart on failure |
| `docker run --health-cmd="curl http://localhost" [image]` | Enable health check |
| `docker inspect {{.State.Health.Status}} [container]` | Check health status |
| `docker service update --image [new] [service]` | Rolling update |
| `docker secret create [name] [file]` | Create secret |
| `docker logs [container]` | View container logs |
| `docker stats --no-stream [container]` | Single stats snapshot |
| `docker update --memory=1g [container]` | Update resource limits |
| `docker cp [container]:[path] [local-path]` | Backup container files |
