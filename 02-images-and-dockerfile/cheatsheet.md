# Cheatsheet: Images and Dockerfile

Quick reference for Docker image building and Dockerfile commands.

## Dockerfile Instructions

| Instruction | Purpose | Example |
|-------------|---------|---------|
| FROM | Base image | FROM ubuntu:22.04 |
| RUN | Execute command | RUN apt-get install -y curl |
| COPY | Copy files | COPY . /app |
| ADD | Copy + extract archives | ADD app.tar.gz /app |
| WORKDIR | Set working directory | WORKDIR /app |
| EXPOSE | Document port | EXPOSE 8080 |
| ENV | Environment variable | ENV DEBUG=true |
| ARG | Build argument | ARG NODE_VERSION=18 |
| USER | Set user | USER appuser |
| CMD | Default command | CMD ["python", "app.py"] |
| ENTRYPOINT | Entry point | ENTRYPOINT ["./app"] |
| VOLUME | Volume mount point | VOLUME /data |
| LABEL | Metadata | LABEL version="1.0" |
| HEALTHCHECK | Health check | HEALTHCHECK CMD curl -f http://localhost/ |

---

## Docker Build Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `docker build -t [name:tag] .` | Build image | docker build -t myapp:1.0 . |
| `docker build -f [Dockerfile]` | Custom Dockerfile | docker build -f custom.Dockerfile . |
| `docker build --no-cache` | Disable caching | docker build --no-cache -t app . |
| `docker build --build-arg KEY=VALUE` | Pass build arg | docker build --build-arg NODE_ENV=prod . |
| `docker build --target [stage]` | Multi-stage specific | docker build --target runtime . |
| `docker build -t [name:tag] [context]` | Custom context | docker build -t app . /path/to/context |

---

## Image Management Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `docker images` | List images | docker images |
| `docker images [name]` | Filter images | docker images ubuntu |
| `docker image inspect [image]` | Image details | docker image inspect ubuntu:22.04 |
| `docker image history [image]` | Image layers | docker image history myapp:1.0 |
| `docker image tag [src] [dst]` | Tag image | docker image tag app:1.0 app:latest |
| `docker rmi [image]` | Remove image | docker rmi myapp:1.0 |
| `docker rmi -f [image]` | Force remove | docker rmi -f myapp:1.0 |

---

## Dockerfile Best Practices

| Practice | Example | Benefit |
|----------|---------|---------|
| Use specific base version | FROM ubuntu:22.04 | Reproducibility |
| .dockerignore file | .git, node_modules | Faster builds |
| Multi-line RUN | RUN apt && apt && rm -rf | Fewer layers |
| Order by change frequency | Dependencies, then code | Better caching |
| Remove caches | RUN apt clean && rm -rf | Smaller images |
| Non-root user | USER appuser | Security |
| Multi-stage build | FROM ... AS builder | Reduced size |
| EXPOSE port | EXPOSE 8080 | Documentation |

---

## Common Dockerfile Patterns

### Python App

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 5000
CMD ["python", "app.py"]
```

### Node.js App

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json .
RUN npm ci --only=production
COPY . .
EXPOSE 3000
CMD ["node", "index.js"]
```

### Go App (Multi-stage)

```dockerfile
FROM golang:1.20 AS builder
WORKDIR /app
COPY . .
RUN CGO_ENABLED=0 go build -o app .

FROM alpine:latest
COPY --from=builder /app/app .
EXPOSE 8080
CMD ["./app"]
```

---

## Build Arguments vs Environment Variables

| Aspect | ARG | ENV |
|--------|-----|-----|
| Scope | Build time only | Runtime + build |
| In Dockerfile | Yes | Yes |
| Override at build | --build-arg | (set in container) |
| In running container | No | Yes |
| Use case | Build options | App configuration |

---

## Layer Caching Strategy

| Order | Impact |
|-------|--------|
| FROM | (Most stable) |
| System dependencies | (Install OS packages) |
| Application dependencies | (Install app requirements) |
| Application code | (Most likely to change) |
| CMD/ENTRYPOINT | (Almost never changes) |

**Rule:** Place frequently-changed layers at the end

---

## Multi-stage Build Pattern

```dockerfile
# Stage 1: Builder
FROM golang:1.20 AS builder
WORKDIR /build
COPY . .
RUN go build -o myapp .

# Stage 2: Runtime
FROM alpine:latest
WORKDIR /app
COPY --from=builder /build/myapp .
CMD ["./myapp"]
```

**Benefits:**
- Compiler not in final image
- Smaller final size (golang: 800MB → 20MB)
- Faster deployments

---

## Size Optimization Tips

| Tip | Command | Savings |
|-----|---------|---------|
| Use alpine | FROM alpine:latest | 90% |
| Use slim | FROM python:3.11-slim | 80% |
| Multi-stage | (see pattern above) | 95% |
| Remove cache | RUN apt clean && rm -rf | 50-80% |
| .dockerignore | (exclude unnecessary) | 20-40% |
| Combine RUN | RUN a && b && c | 10-20% |

---

## Dockerfile Commands Execution Order

```
1. Validate Dockerfile syntax
2. Create base image container
3. Execute each instruction (in order)
4. Each instruction = new layer (if changes)
5. Commit layer to image
6. Tag final image
```

---

## Common Errors and Fixes

| Error | Cause | Fix |
|-------|-------|-----|
| "base image not found" | Image doesn't exist | docker pull [image] |
| "COPY failed" | Wrong path | Verify file in context |
| "command not found" | Binary not in image | Install in RUN |
| "permission denied" | User permissions | Use USER instruction |
| "port already in use" | Container running | docker stop, then run |

---

## Debugging Tips

```bash
# Run intermediate container
docker run -it [image_id] bash

# View build logs
docker build -t app . 2>&1 | less

# Check layer content
docker history --no-trunc [image]

# Inspect image
docker inspect [image] | jq '.Config'

# View layer file system
docker run -it [image] ls -la /
```

---

## Build Optimization Checklist

- [ ] Using .dockerignore
- [ ] Multi-line RUN commands
- [ ] Base image version specified
- [ ] Dependencies before code
- [ ] Removing caches after apt/yum/pip
- [ ] Non-root user set
- [ ] HEALTHCHECK configured
- [ ] Minimal base image (alpine/slim)
- [ ] Multi-stage for compiled languages
- [ ] Layer caching optimized

---

**Need more?**
- Official Docs: https://docs.docker.com/engine/reference/builder/
- Module Exercises: [exercises.md](exercises.md)
- Solutions: [solutions.md](solutions.md)

