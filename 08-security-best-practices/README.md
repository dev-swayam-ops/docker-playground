# Module 08: Docker Security Best Practices

Secure your Docker containers and images with industry-standard practices. Learn how to protect your applications from common vulnerabilities and threats.

## What You'll Learn

- How to use non-root users inside containers
- How to implement read-only root filesystems
- How to scan images for vulnerabilities
- How to manage secrets securely (not as environment variables)
- How to use security options and capabilities
- How to implement resource limits
- How to keep images minimal and up-to-date
- How to use Docker Content Trust for image verification
- How to apply the principle of least privilege
- How to audit and monitor container behavior

## Prerequisites

- Docker installed and running (Module 00)
- Basic Docker commands knowledge (Module 01)
- Understanding of container basics (Module 03)
- Basic understanding of Linux users and permissions
- Familiarity with Dockerfiles (Module 02)

## Key Concepts

### Non-Root Users
Running containers as the root user is a security risk. If an attacker compromises your container, they gain root privileges on the host. **Solution:** Create and use a non-root user inside your container.

### Read-Only Filesystem
Prevents unauthorized modifications to the container's filesystem. Attackers can't write malware or modify critical files if the filesystem is read-only.

### Image Scanning
Tools like `docker scan` or Trivy identify known vulnerabilities in container images before deployment. Catch security issues early.

### Secrets Management
Never pass sensitive data (passwords, API keys, database credentials) as environment variables. They're visible in container inspect output and logs. Use Docker Secrets or external secret management tools.

### Security Capabilities
Linux capabilities allow fine-grained permission management. Instead of running as root, grant only specific capabilities needed for your application.

### Resource Limits
Prevent containers from consuming excessive CPU/memory (DoS attacks). Set `--memory` and `--cpus` limits.

### Image Minimalism
Smaller images with fewer packages reduce the attack surface. Use slim/alpine base images and remove unnecessary dependencies.

### Docker Content Trust
Sign images with keys to verify their authenticity and prevent tampering.

### Principle of Least Privilege
Give containers only the minimum permissions, capabilities, and resources needed to function.

### Container Audit
Monitor and log container activity to detect suspicious behavior or security incidents.

---

## Hands-on Lab: Building a Secure Container

### Lab Objective
Create and run a secure container with non-root user, read-only filesystem, and resource limits.

### Step 1: Create a Dockerfile with Security Best Practices

```dockerfile
# Use specific version (not 'latest')
FROM ubuntu:22.04

# Update packages and remove cache to reduce image size
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    curl \
    ca-certificates && \
    rm -rf /var/lib/apt/lists/*

# Create a non-root user
RUN useradd -m -u 1000 appuser

# Set working directory
WORKDIR /app

# Copy application files
COPY --chown=appuser:appuser app.sh /app/

# Make script executable
RUN chmod +x /app/app.sh

# Switch to non-root user
USER appuser

# Run the application
CMD ["/app/app.sh"]
```

### Step 2: Create the Application Script

```bash
cat > app.sh << 'EOF'
#!/bin/bash
echo "Application running as: $(whoami)"
echo "Current user ID: $(id)"
echo "Current directory: $(pwd)"
echo "Application is secure!"
sleep 300
EOF
```

### Step 3: Build the Image

```bash
# Build the image
docker build -t secure-app:1.0 .

# Expected output:
# [+] Building 15.2s (9/9) FINISHED
# ...
```

### Step 4: Run with Security Best Practices

```bash
# Run container with security options
docker run -d \
  --name secure-container \
  --user appuser \
  --read-only \
  --cap-drop=ALL \
  --cap-add=NET_BIND_SERVICE \
  --memory=256m \
  --cpus=0.5 \
  --security-opt=no-new-privileges:true \
  --tmpfs /tmp \
  --tmpfs /run \
  secure-app:1.0

# Expected output:
# a1b2c3d4e5f6 (container ID)
```

### Step 5: Verify Container is Secure

```bash
# Check that it's running as non-root
docker exec secure-container whoami
# Expected output: appuser

# Check the security options
docker inspect secure-container | grep -A 20 '"SecurityOpt"'
# Expected output shows security options are applied
```

### Step 6: Scan Image for Vulnerabilities

```bash
# Scan the image for known vulnerabilities
docker scan secure-app:1.0

# Expected output:
# Scanning image secure-app:1.0
# ✓ Image passes security scan
# OR lists any found vulnerabilities
```

### Step 7: View Resource Limits

```bash
# Check memory and CPU limits
docker inspect secure-container | grep -E '"Memory"|"CpuQuota"'

# Expected output:
# "Memory": 268435456,  (256MB in bytes)
# "CpuQuota": 50000,
```

---

## Validation Checklist

After completing the lab, verify:

- [ ] Container runs as non-root user (not root)
- [ ] Filesystem is read-only (can't create files in /app)
- [ ] Resource limits are applied (memory and CPU)
- [ ] Security capabilities are dropped
- [ ] No unnecessary packages in image
- [ ] Image was scanned for vulnerabilities
- [ ] `docker inspect` shows security settings
- [ ] Container can still access /tmp and /run (tmpfs)
- [ ] Image uses specific version tag (not 'latest')

### Validation Commands

```bash
# Verify non-root user
docker exec secure-container whoami
# Should output: appuser

# Verify read-only filesystem (should fail)
docker exec secure-container touch /app/test.txt
# Should get: Read-only file system

# Verify resource limits
docker inspect secure-container | grep -E 'Memory|CpuQuota'

# Verify capabilities
docker inspect secure-container | grep SecurityOpt
```

---

## Cleanup

After finishing the lab:

```bash
# Stop the container
docker stop secure-container

# Remove the container
docker rm secure-container

# Remove the image
docker rmi secure-app:1.0

# Clean up temporary files
rm -f app.sh Dockerfile
```

---

## Common Mistakes

### ❌ Mistake 1: Running as Root
```dockerfile
# BAD
FROM ubuntu:22.04
RUN apt-get update && apt-get install -y myapp
CMD ["myapp"]
# Container runs as root by default
```

**Fix:** Create and use a non-root user
```dockerfile
# GOOD
RUN useradd -m appuser
USER appuser
```

### ❌ Mistake 2: Using 'latest' Tag
```dockerfile
# BAD
FROM ubuntu:latest
```

**Why:** You don't know what version you're getting. Updates may break your app.

**Fix:** Specify exact version
```dockerfile
# GOOD
FROM ubuntu:22.04
```

### ❌ Mistake 3: Storing Secrets in Environment Variables
```bash
# BAD
docker run -e DB_PASSWORD=secret123 myapp
docker inspect myapp  # Password visible in output!
```

**Fix:** Use Docker Secrets or external tools
```bash
# GOOD
echo "secret123" | docker secret create db_password -
docker service create --secret db_password myapp
```

### ❌ Mistake 4: Including Unnecessary Packages
```dockerfile
# BAD
FROM ubuntu:22.04
RUN apt-get update && apt-get install -y \
    curl \
    git \
    vim \
    telnet \
    && ...
# All packages increase attack surface
```

**Fix:** Only install what you need
```dockerfile
# GOOD
FROM alpine:3.17
RUN apk add --no-cache curl ca-certificates
# Smaller image, fewer vulnerabilities
```

### ❌ Mistake 5: Not Setting Resource Limits
```bash
# BAD
docker run myapp
# Container can consume all host resources
```

**Fix:** Always set memory and CPU limits
```bash
# GOOD
docker run \
  --memory=512m \
  --cpus=1.0 \
  myapp
```

### ❌ Mistake 6: Running with All Capabilities
```bash
# BAD
docker run --cap-add=ALL myapp
```

**Fix:** Drop unnecessary capabilities, add only what's needed
```bash
# GOOD
docker run \
  --cap-drop=ALL \
  --cap-add=NET_BIND_SERVICE \
  myapp
```

---

## Troubleshooting

### Issue: "Permission Denied" When Writing to Volume

**Symptom:**
```
docker run -v /data:/app/data myapp
# Error: Permission denied
```

**Cause:** Volume has wrong ownership or permissions.

**Solution:**
```bash
# Check volume ownership
docker run -it alpine ls -la /app/data

# Fix with chown in Dockerfile
COPY --chown=appuser:appuser . /app
```

### Issue: "Read-only file system" Error

**Symptom:**
```
docker run --read-only myapp
# Error: Read-only file system
```

**Cause:** Application needs to write temporary files.

**Solution:** Mount tmpfs for temporary directories
```bash
docker run \
  --read-only \
  --tmpfs /tmp \
  --tmpfs /var/log \
  myapp
```

### Issue: Container Keeps Exiting

**Symptom:**
```
docker run myapp
# Container exits immediately
```

**Cause:** Process inside container is crashing.

**Solution:** Check logs
```bash
# View error messages
docker logs <container_id>

# Run interactively to debug
docker run -it myapp /bin/bash
```

### Issue: "Cannot connect to Docker daemon"

**Symptom:**
```
docker run myapp
# Error: Cannot connect to Docker daemon
```

**Cause:** Docker daemon is not running.

**Solution:**
```bash
# Start Docker daemon
sudo systemctl start docker

# Verify it's running
docker ps
```

### Issue: Image Scan Shows Vulnerabilities

**Symptom:**
```
docker scan myapp
# HIGH severity: CVE-2024-XXXXX
```

**Solution:**
1. Update base image to latest patched version
2. Remove unused packages
3. Use minimal base images (Alpine, Distroless)

```dockerfile
# BAD: Outdated vulnerable image
FROM ubuntu:18.04

# GOOD: Current patched image
FROM ubuntu:22.04
```

### Issue: Out of Memory (OOM)

**Symptom:**
```
docker run myapp
# Killed (container stops abruptly)
```

**Cause:** Application using too much memory, no limit set.

**Solution:** Set and increase memory limits
```bash
docker run \
  --memory=512m \
  --memory-swap=512m \
  myapp
```

---

## Next Steps

### Continue Learning

1. **Module 09: Troubleshooting and Debugging**
   - Debug security issues
   - Monitor container behavior
   - Analyze logs for security events

2. **Docker Security Advanced Topics**
   - AppArmor and SELinux profiles
   - Network policies
   - Pod security policies (Kubernetes)

3. **Container Scanning & Registry Security**
   - Automated image scanning in CI/CD
   - Private registry setup
   - Image signing and verification

4. **Secrets Management**
   - HashiCorp Vault integration
   - AWS Secrets Manager
   - Kubernetes Secrets

### Practical Projects

1. **Secure Multi-Service Stack**
   - Build a docker-compose with multiple secure services
   - Implement service-to-service authentication
   - Use separate non-root users for each service

2. **Security Scanning Pipeline**
   - Integrate `docker scan` into your build process
   - Fail builds on high-severity vulnerabilities
   - Track and audit security compliance

3. **Hardened Base Image**
   - Create your own minimal base image
   - Pre-audit all included packages
   - Document security decisions

### Resources

- [Docker Security Best Practices Documentation](https://docs.docker.com/develop/security-best-practices/)
- [CIS Docker Benchmark](https://www.cisecurity.org/benchmark/docker)
- [NIST Application Container Security Guide](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-190.pdf)
- [Docker Content Trust](https://docs.docker.com/engine/security/trust/)

### Quiz Yourself

- What's the main risk of running containers as root?
- Name 3 ways to reduce container image size
- When would you use `--read-only` vs `--tmpfs`?
- What's the difference between dropping and adding capabilities?
- Why not use environment variables for secrets?

---

**Status:** Ready for exercises  
**Estimated Time:** 30-45 minutes  
**Difficulty:** Intermediate

Next → [Exercises](exercises.md)
