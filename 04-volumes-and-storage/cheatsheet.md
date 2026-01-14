# Cheatsheet - Volumes & Storage

| Command | Purpose |
|---------|---------|
| `docker volume create [name]` | Create volume |
| `docker volume ls` | List volumes |
| `docker volume rm [name]` | Remove volume |
| `docker run -v [vol]:/path` | Mount volume |
| `docker run -v /host:/container` | Bind mount |
| `docker run -v /path:ro` | Read-only |
| `docker volume inspect [name]` | Inspect volume |
| `docker volume prune` | Cleanup unused |
