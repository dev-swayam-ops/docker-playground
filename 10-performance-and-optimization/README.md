# Module 10: Performance & Optimization

Optimize Docker images and containers for speed and efficiency.

## What You'll Learn

- Image optimization techniques
- Layer caching strategies
- Resource limits and allocation
- Startup time optimization
- Memory and CPU management
- Container build efficiency
- Performance monitoring

## Lab

```bash
# Set resource limits
docker run --memory=512m --cpus=1 [image]

# Monitor performance
docker stats [container]

# Optimize Dockerfile
FROM python:3.11-slim
RUN apt-get update && apt-get install -y \
    curl \
    && rm -rf /var/apt/cache

# Multi-stage build
FROM python:3.11 AS builder
RUN pip install -r requirements.txt
FROM python:3.11-slim
COPY --from=builder /usr/local /usr/local
```

## Key Concepts

- Layer caching: Docker caches layers; keep changeable code last
- Image size: Use slim/alpine base images
- Resource limits: --memory and --cpus prevent resource exhaustion
- Build speed: Leverage .dockerignore and layer caching
- Multi-stage builds: Reduce final image size by excluding build dependencies

---

**Exercises:** [exercises.md](exercises.md)
This folder contains information about Docker performance and optimization.
