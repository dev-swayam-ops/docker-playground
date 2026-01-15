# Solutions: Docker Security Best Practices

Detailed solutions with commands and explanations for all exercises.

---

## Solution 1: Understanding Container User Context

### Commands

```bash
# Run a container as root (default)
docker run --name root-container ubuntu:22.04 whoami

# Expected output:
# root

# List all containers
docker ps -a

# Expected output:
# CONTAINER ID   IMAGE          COMMAND   CREATED        STATUS                  NAMES
# a1b2c3d4e5f6   ubuntu:22.04   "whoami"  2 minutes ago  Exited (0) 2 minutes ago  root-container

# Remove the container
docker rm root-container
```

### Explanation

**What Happened:**
- The container ran the `whoami` command as the root user by default
- Ubuntu image has no non-root user specified, so it defaults to root (UID 0)
- The container exited after the command completed

**Security Risk:**
If an attacker compromises your container, they gain root privileges. This can lead to:
- Container escape and host compromise
- Modification of system files
- Installation of malware or backdoors
- Full control over sensitive data

**Answers:**
1. **What user?** Root (UID 0)
2. **What's the security risk?** Root has full system access; compromise = total system access
3. **Why is non-root better?** Limited privileges reduce damage from container compromise

---

## Solution 2: Create a Non-Root User Dockerfile

### Commands

```bash
# Create Dockerfile
cat > Dockerfile << 'EOF'
FROM ubuntu:22.04

# Create a non-root user with specific UID
RUN useradd -m -u 1000 appuser

# Set working directory
WORKDIR /app

# Create a simple script
RUN echo '#!/bin/bash' > /app/app.sh && \
    echo 'echo "Hello from secure container"' >> /app/app.sh

# Make script executable
RUN chmod +x /app/app.sh

# Change ownership to appuser
RUN chown -R appuser:appuser /app

# Switch to non-root user
USER appuser

# Run the script
CMD ["/app/app.sh"]
EOF

# Build the image
docker build -t secure-ubuntu:1.0 .

# Expected output:
# Sending build context to Docker daemon  2.048kB
# Step 1/8 : FROM ubuntu:22.04
# ...
# Successfully tagged secure-ubuntu:1.0

# Run the container
docker run secure-ubuntu:1.0

# Expected output:
# Hello from secure container
```

### Explanation

**Key Points:**

| Step | Purpose | Security Benefit |
|------|---------|------------------|
| `FROM ubuntu:22.04` | Use specific version | Prevents unexpected updates |
| `useradd` | Create non-root user | Limits container privileges |
| `USER appuser` | Switch before CMD | Ensures process runs as non-root |
| `chown` | Fix file ownership | Non-root user can access files |
| `WORKDIR /app` | Set working directory | Organized file structure |

**Container User Context:**
- UID 1000 is a standard non-root user ID
- `-m` flag creates the user's home directory
- `USER` instruction changes the default user for subsequent commands

---

## Solution 3: Verify Image Layers and Size

### Commands

```bash
# Check image size
docker image ls secure-ubuntu:1.0

# Expected output:
# REPOSITORY      TAG    IMAGE ID      CREATED        SIZE
# secure-ubuntu   1.0    a1b2c3d4e5f6  5 minutes ago  77.8MB

# View image history (layers)
docker history secure-ubuntu:1.0

# Expected output:
# IMAGE          CREATED         CREATED BY                                SIZE
# a1b2c3d4e5f6   5 minutes ago   /bin/sh -c #(nop) CMD ["/app/app.sh"]     0B
# ...
# <missing>      2 weeks ago     /bin/sh -c #(nop) ADD file:123...         77.8MB

# Check Ubuntu base image size
docker image ls ubuntu:22.04

# Expected output:
# REPOSITORY  TAG     IMAGE ID      CREATED       SIZE
# ubuntu      22.04   58db3edaf2be  2 weeks ago   77.8MB

# Create Alpine-based Dockerfile
cat > Dockerfile-alpine << 'EOF'
FROM alpine:3.17

RUN apk add --no-cache ca-certificates && \
    rm -rf /var/cache/apk/*

RUN useradd -m -u 1000 appuser

WORKDIR /app

RUN echo '#!/bin/sh' > /app/app.sh && \
    echo 'echo "Hello from secure container"' >> /app/app.sh

RUN chmod +x /app/app.sh && \
    chown -R appuser:appuser /app

USER appuser

CMD ["/app/app.sh"]
EOF

# Build Alpine image
docker build -f Dockerfile-alpine -t secure-alpine:1.0 .

# Compare sizes
docker image ls | grep secure

# Expected output:
# REPOSITORY      TAG    IMAGE ID      SIZE
# secure-alpine   1.0    b2c3d4e5f6a7  5.5MB
# secure-ubuntu   1.0    a1b2c3d4e5f6  77.8MB
```

### Explanation

**Image Size Comparison:**

| Aspect | Ubuntu:22.04 | Alpine:3.17 |
|--------|--------------|-------------|
| Base Size | ~77.8MB | ~7.04MB |
| Packages | Full OS | Minimal |
| Build Time | ~1 minute | ~10 seconds |
| Best For | General purpose | Lightweight, secure |

**Answers:**
1. **Which is smaller?** Alpine (5.5MB vs 77.8MB for Ubuntu)
2. **Pros/cons:**
   - **Alpine Pros:** Smaller, faster, fewer packages, fewer vulnerabilities
   - **Alpine Cons:** musl instead of glibc (compatibility issues possible)
   - **Ubuntu Pros:** Familiar, more tools available
   - **Ubuntu Cons:** Larger, more vulnerabilities to scan
3. **Security relation:** Smaller images = fewer packages = fewer vulnerabilities = smaller attack surface

---

## Solution 4: Implement Read-Only Filesystem

### Commands

```bash
# Create a test Dockerfile
cat > Dockerfile << 'EOF'
FROM alpine:3.17

RUN useradd -m -u 1000 appuser

USER appuser

CMD ["sleep", "300"]
EOF

# Build the image
docker build -t readonly-test:1.0 .

# Run with read-only filesystem
docker run -d \
  --name readonly-container \
  --read-only \
  readonly-test:1.0

# Expected output:
# a1b2c3d4e5f6

# Try to create a file (should fail)
docker exec readonly-container touch /test.txt

# Expected output:
# touch: /: Read-only file system

# Run with tmpfs to allow temporary storage
docker run -d \
  --name readonly-with-tmp \
  --read-only \
  --tmpfs /tmp \
  readonly-test:1.0

# Try creating in /tmp (should succeed)
docker exec readonly-with-tmp touch /tmp/test.txt

# Expected output: (no error)

# Verify the file was created
docker exec readonly-with-tmp ls -la /tmp/test.txt

# Expected output:
# -rw-r--r-- 1 appuser appuser 0 Jan 15 12:34 /tmp/test.txt
```

### Explanation

**Read-Only Filesystem Benefits:**
| Benefit | How It Works |
|---------|-------------|
| Prevents Modifications | Attacker can't write malware to filesystem |
| Limits Attack Surface | Reduces ways to compromise container |
| Ensures Immutability | Configuration can't be changed at runtime |
| Easy Debugging | Can see only what was in image |

**tmpfs (Temporary File System):**
- Mounted in RAM, not persisted to disk
- Perfect for temporary files, logs, cache
- Automatically cleaned up when container stops
- Data doesn't persist across restarts (by design)

**Answers:**
1. **Why did it fail?** Filesystem is mounted as read-only; no write operations allowed
2. **Why use tmpfs?** Applications need /tmp for temporary files; tmpfs provides writable space in RAM
3. **Real use cases:** 
   - Immutable production deployments
   - Preventing malware installation
   - Enforcing configuration compliance

---

## Solution 5: Drop Unnecessary Linux Capabilities

### Commands

```bash
# Check capabilities in normal container
docker run --rm ubuntu:22.04 capsh --print

# Expected output:
# Current: cap_chown,cap_dac_override,cap_setfuid,cap_setgid,cap_net_raw,cap_sys_chroot,cap_kill,cap_net_bind_service=ep
# Bounding set: (similar)
# ...

# Run container with all capabilities dropped
docker run -d \
  --name no-caps \
  --cap-drop=ALL \
  ubuntu:22.04 sleep 300

# Try to modify IP forwarding (requires NET_ADMIN capability)
docker exec no-caps bash -c "echo 1 > /proc/sys/net/ipv4/ip_forward"

# Expected output:
# bash: /proc/sys/net/ipv4/ip_forward: Permission denied

# Run with minimal capabilities (only NET_BIND_SERVICE)
docker run -d \
  --name minimal-caps \
  --cap-drop=ALL \
  --cap-add=NET_BIND_SERVICE \
  ubuntu:22.04 sleep 300

# Inspect capabilities
docker inspect no-caps | grep -A 10 '"CapAdd"'

# Expected output:
# "CapAdd": null,
# "CapDrop": [
#   "ALL"
# ],

docker inspect minimal-caps | grep -A 10 '"CapAdd"'

# Expected output:
# "CapAdd": [
#   "NET_BIND_SERVICE"
# ],
# "CapDrop": [
#   "ALL"
# ],
```

### Explanation

**Common Linux Capabilities:**

| Capability | Purpose | Risk Level |
|------------|---------|-----------|
| NET_BIND_SERVICE | Bind to ports < 1024 | Medium |
| NET_ADMIN | Network administration | High |
| SYS_ADMIN | System administration | Critical |
| CHOWN | Change file owner | Medium |
| DAC_OVERRIDE | Bypass file permissions | High |
| SETUID | Change effective UID | High |

**Principle of Least Privilege:**
Start with all capabilities dropped, add only what your application truly needs.

**Best Practices:**
```bash
# BAD: Allow all capabilities (default)
docker run myapp

# GOOD: Drop all, add specific ones
docker run \
  --cap-drop=ALL \
  --cap-add=NET_BIND_SERVICE \
  myapp
```

---

## Solution 6: Set Resource Limits

### Commands

```bash
# Run without limits (baseline)
docker run -d --name unlimited alpine yes > /dev/null

# Check resources
docker stats unlimited --no-stream

# Expected output:
# CONTAINER ID   NAME        CPU %   MEM USAGE / LIMIT
# a1b2c3d4e5f6   unlimited   95.6%   5.23 MiB / 7.727 GiB

# Stop and remove
docker stop unlimited && docker rm unlimited

# Run with memory limit
docker run -d \
  --name memory-limited \
  --memory=64m \
  alpine yes > /dev/null

# Check resources (memory will be capped)
docker stats memory-limited --no-stream

# Expected output:
# CONTAINER ID   NAME              CPU %   MEM USAGE / LIMIT
# b2c3d4e5f6a7   memory-limited    85.2%   63.98 MiB / 64 MiB

# Run with memory and CPU limits
docker run -d \
  --name full-limits \
  --memory=128m \
  --cpus=0.5 \
  alpine yes > /dev/null

# View all
docker stats --no-stream

# Expected output:
# CONTAINER ID   NAME            CPU %     MEM USAGE / LIMIT
# b2c3d4e5f6a7   memory-limited  85.2%     63.98 MiB / 64 MiB
# c3d4e5f6a7b8   full-limits     49.8%     127.93 MiB / 128 MiB
```

### Explanation

**Memory Limits:**
```bash
--memory=256m        # Hard limit: container can't use more
--memory-swap=512m   # Total memory + swap limit
--memoryswap=-1      # Unlimited swap (not recommended)
```

**CPU Limits:**
```bash
--cpus=0.5           # 50% of one CPU core
--cpus=2             # 2 full CPU cores
--cpu-period=100000  # Advanced: CPU period (microseconds)
--cpu-quota=50000    # Advanced: CPU quota per period
```

**Out of Memory (OOM) Behavior:**
- Memory limit exceeded → kernel kills the process
- Container exits with error code 137 (SIGKILL)
- `docker inspect` shows `OOMKilled: true`

**Answers:**
1. **How does limit affect process?** Process can only use allocated memory; exceeding triggers OOM killer
2. **What happens if exceeded?** Container is killed, exits with code 137
3. **Why set CPU limits?** Prevent container from consuming all host CPUs; ensures other containers get resources
4. **Determine limits:** Monitor application, set 1.5x typical peak usage

---

## Solution 7: Scan Image for Vulnerabilities

### Commands

```bash
# Scan secure image (minimal vulnerabilities)
docker scan secure-ubuntu:1.0

# Expected output:
# Scanning image secure-ubuntu:1.0
# ...
# ✓ Image passes security scan
# OR lists low-severity vulnerabilities

# Create intentionally vulnerable Dockerfile
cat > Dockerfile-old << 'EOF'
FROM ubuntu:16.04

RUN apt-get update && apt-get install -y curl
EOF

# Build old image
docker build -f Dockerfile-old -t old-image:1.0 .

# Scan old image
docker scan old-image:1.0

# Expected output:
# Scanning image old-image:1.0
# ...
# Found 23 vulnerabilities (15 high, 8 medium)

# Update to newer base image
cat > Dockerfile-new << 'EOF'
FROM ubuntu:22.04

RUN apt-get update && apt-get install -y curl && \
    rm -rf /var/lib/apt/lists/*
EOF

# Build new image
docker build -f Dockerfile-new -t new-image:1.0 .

# Scan new image
docker scan new-image:1.0

# Expected output:
# Scanning image new-image:1.0
# ...
# Found 2 vulnerabilities (1 low, 1 medium)
```

### Explanation

**Why Old Image Has More Vulnerabilities:**
- Ubuntu 16.04 was released in 2016, no longer supported
- Older packages with known CVEs
- Security patches not applied
- Deprecated libraries still included

**Vulnerability Severity Levels:**

| Level | Action | Example |
|-------|--------|---------|
| Low | Monitor, plan update | Non-critical DoS |
| Medium | Update soon | Authentication bypass |
| High | Update ASAP | Remote Code Execution |
| Critical | Immediate action | Kernel exploit |

**Best Practices:**
1. Use latest patch version of base image
2. Set up automated scanning in CI/CD
3. Rebuild images regularly (daily/weekly)
4. Remove unnecessary packages
5. Use minimal base images (Alpine, Distroless)

---

## Solution 8: Implement Dockerfile Security Best Practices

### Commands

```bash
# Create production-ready Dockerfile
cat > Dockerfile << 'EOF'
# Specific version (not latest)
FROM alpine:3.17

# Install only necessary packages, remove cache
RUN apk add --no-cache \
    curl \
    ca-certificates && \
    rm -rf /var/cache/apk/*

# Create non-root user
RUN useradd -m -u 1000 appuser

# Set working directory
WORKDIR /app

# Copy application with correct ownership
COPY --chown=appuser:appuser . /app/

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8080/health || exit 1

# Switch to non-root user
USER appuser

# Default command
CMD ["sleep", "300"]
EOF

# Build the image
docker build -t production-app:1.0 .

# Expected output:
# Successfully tagged production-app:1.0

# Verify non-root user
docker run --rm production-app:1.0 id

# Expected output:
# uid=1000(appuser) gid=1000(appuser) groups=1000(appuser)

# Check image size
docker image ls production-app:1.0

# Expected output:
# REPOSITORY         TAG    SIZE
# production-app     1.0    12.3MB

# Run with full security options
docker run -d \
  --name prod-secure \
  --read-only \
  --cap-drop=ALL \
  --memory=256m \
  --cpus=0.5 \
  --security-opt=no-new-privileges:true \
  --tmpfs /tmp \
  --tmpfs /run \
  production-app:1.0

# Verify it runs as non-root
docker exec prod-secure id

# Expected output:
# uid=1000(appuser) gid=1000(appuser) groups=1000(appuser)

# Check filesystem is read-only
docker exec prod-secure ls -la /

# Expected output shows no write permission for rootfs
```

### Explanation

**Security Best Practices Checklist:**

| Practice | Implementation | Why |
|----------|-----------------|-----|
| Specific version | `FROM alpine:3.17` | Reproducible builds |
| Non-root user | `RUN useradd` + `USER` | Limits damage from breach |
| Minimal image | Alpine/Distroless | Fewer packages = fewer vulns |
| No cache | `rm -rf /var/cache` | Reduces layer size |
| Labels | `LABEL maintainer=` | Metadata for tracking |
| Health check | `HEALTHCHECK` | Auto-restarts unhealthy containers |
| Specific files | `COPY . /app/` | Only include needed files |
| Correct ownership | `--chown=` | Non-root user can access files |

---

## Solution 9: Create a Secrets File

### Commands

```bash
# Create a secret file
echo "my-database-password" > db-secret.txt

# Run container with secret mounted as volume (NOT env var)
docker run -d \
  --name secret-app \
  -v $(pwd)/db-secret.txt:/run/secrets/db_password:ro \
  ubuntu:22.04 sleep 300

# Expected output:
# a1b2c3d4e5f6

# Read secret from inside container
docker exec secret-app cat /run/secrets/db_password

# Expected output:
# my-database-password

# Inspect the container
docker inspect secret-app | grep -A 20 '"Env"'

# Expected output:
# "Env": [
#   "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
# ],
# Notice: NO password in environment variables!

# Check mount points (verify volume is mounted)
docker inspect secret-app | grep -A 10 '"Mounts"'

# Expected output:
# "Mounts": [
#   {
#     "Type": "bind",
#     "Source": "...",
#     "Destination": "/run/secrets/db_password",
#     "Mode": "ro",
#     "RW": false
#   }
# ],

# Cleanup
docker stop secret-app && docker rm secret-app
rm db-secret.txt
```

### Explanation

**Why NOT Environment Variables:**

```bash
# BAD: Secret visible in multiple places
docker run -e DB_PASSWORD=secret123 myapp

# Visible in:
# 1. docker inspect <container> | grep Env
# 2. ps aux (process list)
# 3. Container logs (if application logs it)
# 4. History files in bash
# 5. Docker daemon logs
```

**Why Volume Mount is Better:**

```bash
# GOOD: Secret not in environment
docker run -v /path/to/secret:/run/secrets/password:ro myapp

# Advantages:
# 1. Not visible in docker inspect
# 2. Can be changed without rebuilding image
# 3. Read-only mount prevents accidental writes
# 4. Cleaner separation of concerns
```

**Production Secret Management:**

| Tool | Use Case | Complexity |
|------|----------|-----------|
| Volume mounts | Development | Low |
| Docker Secrets | Swarm mode | Medium |
| HashiCorp Vault | Enterprise | High |
| AWS Secrets Manager | AWS environments | Medium |
| Kubernetes Secrets | Kubernetes | Medium |

**Answers:**
1. **Why volume better?** Password not exposed in docker inspect, process list, or logs
2. **If used env var:** Would be visible in `docker inspect`, environment variable listing, and command arguments
3. **Real-world tools:** Docker Secrets, Vault, AWS Secrets Manager, Kubernetes Secrets

---

## Solution 10: Audit and Monitor Container Security

### Commands

```bash
# Create and run secure container
docker run -d \
  --name audit-test \
  --user 1000 \
  --read-only \
  --cap-drop=ALL \
  --memory=256m \
  --security-opt=no-new-privileges:true \
  ubuntu:22.04 sleep 300

# Expected output:
# a1b2c3d4e5f6

# Inspect security settings (detailed)
docker inspect audit-test | jq '.[] | {
  User,
  ReadonlyRootfs,
  SecurityOpt,
  HostConfig: {
    Memory,
    CpuQuota,
    CapAdd,
    CapDrop,
    Privileged
  }
}'

# Expected output:
# {
#   "User": "1000",
#   "ReadonlyRootfs": true,
#   "SecurityOpt": [
#     "no-new-privileges=true"
#   ],
#   "HostConfig": {
#     "Memory": 268435456,
#     "CapAdd": null,
#     "CapDrop": ["ALL"]
#   }
# }

# Check running processes
docker top audit-test

# Expected output:
# UID      PID      PPID     C   STIME   TTY   TIME     CMD
# 1000     1        0        0   12:34   ?     00:00:00 sleep 300

# View container logs
docker logs audit-test

# Expected output: (likely empty, unless container prints logs)

# Get real-time statistics
docker stats audit-test --no-stream

# Expected output:
# CONTAINER ID   NAME        CPU %   MEM USAGE / LIMIT   NET I/O   BLOCK I/O
# a1b2c3d4e5f6   audit-test  0.1%    0.5 MiB / 256 MiB   0B / 0B   0B / 0B

# Create comprehensive audit report
cat > audit-report.txt << 'EOF'
=== SECURITY AUDIT REPORT ===

Container: audit-test
Date: $(date)

IDENTITY:
- User: appuser (UID 1000)
- Root Access: NO
- Privilege Escalation: NOT ALLOWED (no-new-privileges)

FILESYSTEM:
- Root Filesystem: READ-ONLY
- Write Access: /tmp, /run only (via tmpfs)
- Immutable Configuration: YES

CAPABILITIES:
- Dropped: ALL
- Added: NONE
- Privilege Level: Minimal

RESOURCE LIMITS:
- Memory: 256MB
- CPU: Not limited
- Running Processes: sleep (1 process)

SECURITY POSTURE:
- Non-root: ✓
- Minimal capabilities: ✓
- Resource limited: ✓
- Read-only filesystem: ✓
- No privilege escalation: ✓

COMPLIANCE: ✓ PASS
EOF

cat audit-report.txt
```

### Explanation

**Security Audit Checklist:**

| Item | What to Check | Expected Result |
|------|---------------|-----------------|
| User | `User` in inspect | Non-root (UID > 0) |
| Readonly | `ReadonlyRootfs` | true |
| Capabilities | `CapAdd`/`CapDrop` | CapDrop=ALL, CapAdd=specific |
| Privileges | `no-new-privileges` | true |
| Memory | `Memory` value | Set to appropriate limit |
| CPU | `CpuQuota` | Set if CPU limited |
| Processes | `docker top` | Expected process running |
| Ports | `PortBindings` | Only necessary ports |

**Auditing Best Practices:**
1. **Automated Scanning:** Integrate into CI/CD pipeline
2. **Regular Reviews:** Audit all containers weekly
3. **Policy Enforcement:** Require security settings
4. **Compliance Tracking:** Document and maintain audit trails
5. **Incident Response:** Know what to check if breach suspected

**Answers:**
1. **How to audit for compliance?**
   - Use tools like `docker-bench` to check against CIS benchmarks
   - Inspect containers for security settings
   - Scan images for vulnerabilities
   - Monitor logs for suspicious activity

2. **What should be in audit?**
   - User context (uid/gid)
   - Filesystem access level
   - Applied capabilities
   - Resource limits
   - Listening ports
   - Environment variables
   - Mounted volumes

3. **Automate auditing:**
   - Shell script to inspect all containers
   - Save output to audit log
   - Run as scheduled job (cron)
   - Compare against baseline

---

## Summary of Security Best Practices

| Practice | Why | How |
|----------|-----|-----|
| Non-root user | Limit breach damage | `USER appuser` in Dockerfile |
| Specific versions | Reproducible, patched | `FROM ubuntu:22.04` (not latest) |
| Drop capabilities | Least privilege | `--cap-drop=ALL --cap-add=...` |
| Resource limits | Prevent DoS | `--memory=256m --cpus=0.5` |
| Read-only FS | Prevent modification | `--read-only --tmpfs /tmp` |
| Scan images | Find vulnerabilities | `docker scan image:tag` |
| Minimal image | Fewer vulns | Use Alpine, Distroless |
| Secrets in volumes | Security | `-v secret:/run/secrets:ro` |
| Security audits | Compliance | Inspect, logs, stats |

**Next Challenge:** Create a full Docker Compose stack with all these security practices applied to multiple services!

---

**Time to Complete:** 90-120 minutes  
**Exercises Covered:** 10/10  
**Difficulty:** Easy → Medium  
**Status:** Ready for advanced modules
