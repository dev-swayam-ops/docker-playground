# Module 07: Registries & Image Management

Manage Docker images and registries for collaboration and deployment.

## What You'll Learn

- Docker Hub and image registries
- Push and pull images
- Image tagging and versioning
- Private registries
- Image scanning and security
- Image layers and metadata
- Image cleanup and maintenance

## Lab

```bash
# Login to Docker Hub
docker login

# Tag image for registry
docker tag myapp:1.0 username/myapp:1.0

# Push to registry
docker push username/myapp:1.0

# Pull from registry
docker pull username/myapp:1.0

# Logout
docker logout
```

## Quick Commands

| Command | Purpose |
|---------|---------|
| `docker login [registry]` | Login |
| `docker logout` | Logout |
| `docker tag [src] [dst]` | Tag image |
| `docker push [image]` | Push to registry |
| `docker pull [image]` | Pull from registry |
| `docker search [term]` | Search Docker Hub |
| `docker image prune` | Cleanup images |

---

**Exercises:** [exercises.md](exercises.md)
This folder contains information about Docker registries and image management.
