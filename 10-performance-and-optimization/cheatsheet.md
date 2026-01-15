## Cheatsheet

| Command | Purpose |
|---------|---------|
| `docker run --memory=512m [image]` | Limit memory |
| `docker run --cpus=1 [image]` | Limit CPU cores |
| `docker stats [container]` | Monitor performance |
| `docker image ls --format "table {{.Repository}}\t{{.Size}}"` | Show image sizes |
| `docker history [image]` | Analyze layer sizes |
| `docker build --no-cache [image]` | Build without cache |
| `docker image prune -a` | Remove unused images |
| `docker system df` | Show Docker resource usage |
| `docker ps --format "table {{.Names}}\t{{.CPUPerc}}\t{{.MemUsage}}"` | Format container stats |