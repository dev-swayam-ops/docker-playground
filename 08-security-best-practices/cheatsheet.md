# Cheatsheet: Docker Security Best Practices

Quick reference for implementing security in Docker containers.

---

## Running Secure Containers

| Command | Purpose | Example |
|---------|---------|---------|
| `docker run --user [UID]` | Run as non-root user | `docker run --user 1000 ubuntu` |
| `docker run --read-only` | Make filesystem read-only | `docker run --read-only ubuntu` |
| `docker run --tmpfs [PATH]` | Create temporary filesystem | `docker run --tmpfs /tmp ubuntu` |
| `docker run --cap-drop=ALL` | Drop all capabilities | `docker run --cap-drop=ALL ubuntu` |
| `docker run --cap-add=[CAP]` | Add specific capability | `docker run --cap-add=NET_BIND_SERVICE ubuntu` |
| `docker run --memory [SIZE]` | Limit memory usage | `docker run --memory=256m ubuntu` |
| `docker run --cpus [COUNT]` | Limit CPU usage | `docker run --cpus=0.5 ubuntu` |
| `docker run --security-opt=no-new-privileges:true` | Prevent privilege escalation | `docker run --security-opt=no-new-privileges:true ubuntu` |
| `docker run -v [PATH]:[MPATH]:ro` | Mount volume read-only | `docker run -v /secret:/app/secret:ro ubuntu` |
| `docker run --pids-limit [NUM]` | Limit process count | `docker run --pids-limit=100 ubuntu` |

---

## Security Options Combinations

| Use Case | Command |
|----------|---------|
| **Minimal Privilege** | `docker run --cap-drop=ALL --user 1000 --read-only --tmpfs /tmp` |
| **Web Server** | `docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE --user 1000 --memory=512m --cpus=1` |
| **Database** | `docker run --cap-drop=ALL --user 999 --memory=2g --cpus=2 --pids-limit=1000` |
| **Development** | `docker run --user 1000 -v $(pwd):/app` |
| **Production** | `docker run --cap-drop=ALL --read-only --tmpfs /tmp --memory=256m --security-opt=no-new-privileges:true --user 1000` |

---

## Dockerfile Security

| Dockerfile Instruction | Purpose | Secure Example |
|------------------------|---------|-----------------|
| `FROM` | Base image | `FROM ubuntu:22.04` (specific version, not latest) |
| `RUN` | Execute command | `RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*` |
| `RUN useradd` | Create non-root user | `RUN useradd -m -u 1000 appuser` |
| `USER` | Set default user | `USER appuser` (before CMD) |
| `COPY --chown` | Copy with ownership | `COPY --chown=appuser:appuser . /app/` |
| `RUN chmod` | Fix permissions | `RUN chmod +x /app/script.sh` |
| `HEALTHCHECK` | Container health | `HEALTHCHECK CMD curl http://localhost/health` |
| `LABEL` | Metadata | `LABEL maintainer="you@example.com"` |

---

## Image Scanning & Verification

| Command | Purpose | Example |
|---------|---------|---------|
| `docker scan [IMAGE]` | Scan for vulnerabilities | `docker scan myapp:1.0` |
| `docker image inspect [IMAGE]` | Get image details | `docker image inspect ubuntu:22.04` |
| `docker history [IMAGE]` | View image layers | `docker history myapp:1.0` |
| `docker image ls` | List images with size | `docker image ls myapp` |

---

## Container Inspection & Auditing

| Command | Purpose | Example |
|---------|---------|---------|
| `docker inspect [CONTAINER]` | Full container details | `docker inspect myapp` |
| `docker inspect [CONTAINER] \| grep User` | Check running user | `docker inspect myapp \| grep User` |
| `docker inspect [CONTAINER] \| grep ReadonlyRootfs` | Check filesystem mode | `docker inspect myapp \| grep ReadonlyRootfs` |
| `docker inspect [CONTAINER] \| grep Cap` | Check capabilities | `docker inspect myapp \| grep Cap` |
| `docker inspect [CONTAINER] \| grep Memory` | Check memory limit | `docker inspect myapp \| grep Memory` |
| `docker top [CONTAINER]` | View running processes | `docker top myapp` |
| `docker stats [CONTAINER]` | Resource usage | `docker stats myapp --no-stream` |
| `docker logs [CONTAINER]` | View container logs | `docker logs myapp` |

---

## Secrets Management

| Method | Command | Security Level |
|--------|---------|-----------------|
| **Volume Mount** | `docker run -v /secret:/run/secret:ro` | Good |
| **File Input** | `docker run < secret.txt` | Good |
| **Docker Secrets** | `docker secret create db_pass -` | Better |
| **Environment (.env)** | `docker run --env-file .env` | Good (for non-sensitive) |
| **Vault Integration** | External tool required | Best |

**Important:** Never use plain `-e DB_PASSWORD=secret` in production!

---

## Docker-Compose Security

```yaml
# Secure docker-compose.yml example
version: '3.8'

services:
  web:
    image: myapp:1.0          # Specific version
    user: "1000"              # Non-root user
    read_only: true           # Read-only filesystem
    tmpfs:
      - /tmp                  # Temporary storage
      - /var/log
    cap_drop:
      - ALL                   # Drop all capabilities
    cap_add:
      - NET_BIND_SERVICE      # Add only what needed
    memswap_limit: 256m       # Memory limit
    mem_limit: 256m
    cpus: 0.5                 # CPU limit
    security_opt:
      - no-new-privileges:true
    volumes:
      - secret_file:/run/secrets/db_password:ro

volumes:
  secret_file:
    driver: local
```

---

## Common Security Patterns

### Pattern 1: Web Server
```dockerfile
FROM nginx:1.23-alpine
RUN useradd -m -u 101 nginx
USER nginx
EXPOSE 8080  # Non-privileged port
```

```bash
docker run -d \
  --user 101 \
  --read-only \
  --tmpfs /var/run \
  --cap-drop=ALL \
  --cap-add=NET_BIND_SERVICE \
  -p 8080:8080 \
  web:1.0
```

### Pattern 2: Database
```dockerfile
FROM postgres:15-alpine
USER postgres
```

```bash
docker run -d \
  --user 999 \
  --memory=2g \
  --cpus=2 \
  --cap-drop=ALL \
  -v postgres_data:/var/lib/postgresql/data \
  db:15
```

### Pattern 3: Worker/Batch Job
```dockerfile
FROM python:3.11-slim
RUN useradd -m worker
USER worker
WORKDIR /app
```

```bash
docker run -d \
  --user worker \
  --memory=512m \
  --cpus=1 \
  --cap-drop=ALL \
  worker:1.0
```

---

## Linux Capabilities Reference

| Capability | Allows | Typical Usage |
|------------|--------|---------------|
| `NET_BIND_SERVICE` | Bind to ports < 1024 | Web servers |
| `NET_ADMIN` | Network configuration | VPN, routing |
| `SYS_ADMIN` | Mount, namespace operations | Container runtimes |
| `CHOWN` | Change file owner | Limited |
| `DAC_OVERRIDE` | Bypass file permissions | Not recommended |
| `SETUID` | Change effective UID | Avoid if possible |
| `KILL` | Send signals to processes | Process managers |
| `SYS_TIME` | Change system time | Time sync services |

**Best Practice:** Start with `--cap-drop=ALL`, then add back ONLY what's needed.

---

## Security Limits Reference

| Resource | Limit | Purpose | Example |
|----------|-------|---------|---------|
| Memory | RAM limit | Prevent OOM/DoS | `--memory=512m` |
| Memory Swap | Virtual memory | Control total memory | `--memswap=512m` |
| CPU | Core limit | Prevent CPU monopoly | `--cpus=0.5` |
| PIDs | Process count | Prevent process bomb | `--pids-limit=100` |
| File Descriptors | Open files | Limit resource leak | Set in container |
| Network Bandwidth | Bytes/sec | QoS control | Docker Swarm only |

---

## Quick Security Checklist

```bash
# ✓ Run as non-root
docker run --user 1000 [image]

# ✓ Drop unnecessary capabilities
docker run --cap-drop=ALL [image]

# ✓ Set memory limit
docker run --memory=256m [image]

# ✓ Set CPU limit
docker run --cpus=0.5 [image]

# ✓ Make filesystem read-only
docker run --read-only [image]

# ✓ Prevent privilege escalation
docker run --security-opt=no-new-privileges:true [image]

# ✓ Use specific image version
docker run alpine:3.17 [image]

# ✓ Scan for vulnerabilities
docker scan [image]:tag

# ✓ Inspect security settings
docker inspect [container] | grep -E 'User|CapDrop|Memory|ReadonlyRootfs'

# ✓ Check running processes
docker top [container]
```

---

## Troubleshooting Security Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| Permission Denied | Non-root user can't access file | Use `COPY --chown=user:user` or adjust perms |
| Read-only Filesystem | App needs to write | Use `--tmpfs /tmp` or `--tmpfs /var/log` |
| Port Bind Failed | Port < 1024 needs NET_BIND_SERVICE | Add `--cap-add=NET_BIND_SERVICE` |
| OOM Killed | Memory limit too low | Increase with `--memory=` |
| Can't Run Shell | Container exits immediately | Override CMD: `docker run -it [image] /bin/bash` |
| Can't Create Dir | Read-only filesystem | Create needed dirs in tmpfs or image |

---

## Related Modules

- **[Module 01: Docker Basics](../01-docker-basics/)** - Container fundamentals
- **[Module 02: Images and Dockerfile](../02-images-and-dockerfile/)** - Image creation
- **[Module 03: Containers and Lifecycle](../03-containers-and-lifecycle/)** - Container management
- **[Module 09: Troubleshooting and Debugging](../09-troubleshooting-and-debugging/)** - Debugging secure containers

---

**Quick Links:**
- [README: Full explanation](README.md)
- [Exercises: Hands-on practice](exercises.md)
- [Solutions: Step-by-step answers](solutions.md)

---

**Last Updated:** January 2026  
**Status:** Ready to use  
**Difficulty:** Intermediate
