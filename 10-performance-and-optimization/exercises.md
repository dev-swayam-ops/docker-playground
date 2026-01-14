# Module 10: Exercises, Solutions, Cheatsheet & Quiz

## Exercises

1. **Set memory limits**: Run a container with 256MB memory limit using `--memory=256m`.
2. **Set CPU limits**: Run a container restricted to 0.5 CPUs using `--cpus=0.5`.
3. **Monitor container stats**: Use `docker stats` to track resource usage over time.
4. **Compare image sizes**: Build two versions of an image (standard and slim base) and compare sizes.
5. **Optimize Dockerfile**: Reduce a Dockerfile by combining RUN commands with && and ; operators.
6. **Create multi-stage build**: Build an image using two stages to reduce final size.
7. **Verify layer caching**: Make multiple changes to a Dockerfile and observe which layers are cached.
8. **Use .dockerignore**: Create a .dockerignore file to exclude unnecessary files from the build.
9. **Measure startup time**: Time container startup with and without resource limits.
10. **Profile image layers**: Use `docker history [image]` to analyze layer sizes and content.

## Solutions

1. **Memory limits**: `docker run --memory=256m [image]` - Prevents container from exceeding 256MB RAM.
2. **CPU limits**: `docker run --cpus=0.5 [image]` - Restricts container to 0.5 CPU cores max.
3. **Monitor stats**: `docker stats [container]` - Updates every 2 seconds, shows % of host resources.
4. **Compare images**: Use `docker image ls` to see sizes. Slim images are typically 60-80% smaller.
5. **Combine RUN**: `RUN apt-get update && apt-get install -y curl && rm -rf /var/apt/cache` - Reduces layers.
6. **Multi-stage**: First stage builds dependencies, second stage copies only needed artifacts.
7. **Layer caching**: Docker reuses layers if unchanged. Put Dockerfile instructions that change frequently at end.
8. **.dockerignore**: List files/directories to exclude (*.log, node_modules, .git). Same syntax as .gitignore.
9. **Measure time**: Use `time docker run --rm [image] echo test` to measure startup duration.
10. **Layer history**: `docker history [image]` shows each layer, its size, and creation command.

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

## Quiz

**Answers:**

1. What flag sets memory limit to 1GB?
   - Answer: `--memory=1g`

2. How do you prevent a container from using more than 2 CPU cores?
   - Answer: `--cpus=2`

3. Which Docker command shows CPU and memory usage?
   - Answer: `docker stats`

4. What Dockerfile instruction reduces layer count?
   - Answer: Combining commands with `&&` in RUN statements

5. What file excludes unnecessary files from docker build?
   - Answer: `.dockerignore`

6. What command shows the size of each layer?
   - Answer: `docker history [image]`

7. How do multi-stage builds reduce image size?
   - Answer: By excluding build dependencies from the final image

8. **(Short Answer)** Describe three optimization techniques for Dockerfile.
   - Answer: Use slim/alpine base images, combine RUN commands, exclude build dependencies with multi-stage builds.

9. **(Short Answer)** Why should you put frequently-changing instructions at the end of a Dockerfile?
   - Answer: Docker caches layers; moving changeable instructions to end preserves cached layers.

10. **(Short Answer)** How would you identify if a container is using excessive memory?
    - Answer: Use `docker stats` to check memory usage; if it's near the limit, increase `--memory` or optimize the application.

Passing Score: 7/10
