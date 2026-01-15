# Exercises: Docker Security Best Practices

Practice implementing security controls in Docker containers. Complete these 10 exercises to master Docker security fundamentals.

---

## Exercise 1: Understanding Container User Context
**Difficulty:** Easy

Learn how container users work and identify the difference between root and non-root users.

**Task:**
1. Run a container as root (default):
   ```bash
   docker run --name root-container ubuntu:22.04 whoami
   ```
2. Examine the output - what user is the container running as?
3. List all containers and check the container's status:
   ```bash
   docker ps -a
   ```
4. Remove the container:
   ```bash
   docker rm root-container
   ```

**Questions to Answer:**
- What user was displayed in the output?
- What's the security risk of running as root?
- In your own words, why is non-root better?

---

## Exercise 2: Create a Non-Root User Dockerfile
**Difficulty:** Easy

Create a Dockerfile that builds an image with a non-root user.

**Task:**
1. Create a Dockerfile with the following requirements:
   - Base image: `ubuntu:22.04`
   - Create a user named `appuser` with ID 1000
   - Set a working directory to `/app`
   - Create a simple script that echoes "Hello from secure container"
   - Make the script executable
   - Switch to the `appuser` before running the script

2. Build the image:
   ```bash
   docker build -t secure-ubuntu:1.0 .
   ```

3. Run the container and verify it executes as `appuser`:
   ```bash
   docker run secure-ubuntu:1.0
   ```

**Hints:**
- Use `useradd -m -u 1000 appuser` to create the user
- Use `USER appuser` to switch users
- Create the script using `RUN echo > file.sh`

---

## Exercise 3: Verify Image Layers and Size
**Difficulty:** Easy

Understand how to check image size and identify opportunities for optimization.

**Task:**
1. Check the size of your image:
   ```bash
   docker image ls secure-ubuntu:1.0
   ```

2. View the image history to see each layer:
   ```bash
   docker history secure-ubuntu:1.0
   ```

3. For comparison, check the size of the base ubuntu image:
   ```bash
   docker image ls ubuntu:22.04
   ```

4. Create a new Dockerfile using Alpine base image:
   - Base image: `alpine:3.17`
   - Create the same non-root user and script
   - Build as `secure-alpine:1.0`

5. Compare sizes:
   ```bash
   docker image ls | grep secure
   ```

**Questions to Answer:**
- Which image is smaller and why?
- What are the pros and cons of using Alpine?
- How does image size relate to security?

---

## Exercise 4: Implement Read-Only Filesystem
**Difficulty:** Medium

Create and test a container with a read-only root filesystem.

**Task:**
1. Create a simple Dockerfile that:
   - Runs a sleep command
   - Has no special requirements
   - Uses a non-root user

2. Build the image as `readonly-test:1.0`

3. Run the container with read-only filesystem:
   ```bash
   docker run -d \
     --name readonly-container \
     --read-only \
     readonly-test:1.0
   ```

4. Try to create a file in the container (should fail):
   ```bash
   docker exec readonly-container touch /test.txt
   ```

5. Verify the error and document what happened

6. Now run the same container with tmpfs for temporary storage:
   ```bash
   docker run -d \
     --name readonly-with-tmp \
     --read-only \
     --tmpfs /tmp \
     readonly-test:1.0
   ```

7. Try creating a file in /tmp (should succeed):
   ```bash
   docker exec readonly-with-tmp touch /tmp/test.txt
   ```

**Questions to Answer:**
- Why did the first attempt fail?
- Why would you use `--tmpfs` with `--read-only`?
- What are real-world use cases for read-only filesystems?

---

## Exercise 5: Drop Unnecessary Linux Capabilities
**Difficulty:** Medium

Learn to restrict Linux capabilities to follow the principle of least privilege.

**Task:**
1. Check what capabilities a normal container has:
   ```bash
   docker run --rm ubuntu:22.04 capsh --print
   ```

2. Document some of the capabilities listed (pick 5)

3. Run a container with all capabilities dropped:
   ```bash
   docker run -d \
     --name no-caps \
     --cap-drop=ALL \
     ubuntu:22.04 sleep 300
   ```

4. Try to use a capability that was dropped (try to write to /proc/sys):
   ```bash
   docker exec no-caps bash -c "echo 1 > /proc/sys/net/ipv4/ip_forward"
   ```

5. Verify the error occurred as expected

6. Now add back only the NET_BIND_SERVICE capability:
   ```bash
   docker run -d \
     --name minimal-caps \
     --cap-drop=ALL \
     --cap-add=NET_BIND_SERVICE \
     ubuntu:22.04 sleep 300
   ```

7. Inspect both containers to see their capabilities:
   ```bash
   docker inspect no-caps | grep -i cap
   docker inspect minimal-caps | grep -i cap
   ```

**Hints:**
- `capsh` shows Linux capabilities
- Dropping ALL is restrictive but secure
- Only add back what your app truly needs

---

## Exercise 6: Set Resource Limits
**Difficulty:** Medium

Implement memory and CPU limits to prevent resource exhaustion attacks.

**Task:**
1. Run a container without limits (baseline):
   ```bash
   docker run -d --name unlimited alpine yes > /dev/null
   ```

2. Check its resource usage:
   ```bash
   docker stats unlimited --no-stream
   ```

3. Stop and remove it:
   ```bash
   docker stop unlimited && docker rm unlimited
   ```

4. Run the same container WITH memory limit:
   ```bash
   docker run -d \
     --name memory-limited \
     --memory=64m \
     alpine yes > /dev/null
   ```

5. Check its usage (it should be constrained):
   ```bash
   docker stats memory-limited --no-stream
   ```

6. Run a container with both memory and CPU limits:
   ```bash
   docker run -d \
     --name full-limits \
     --memory=128m \
     --cpus=0.5 \
     alpine yes > /dev/null
   ```

7. Compare all three in a table:
   ```bash
   docker stats --no-stream
   ```

**Questions to Answer:**
- How does memory limit affect the process?
- What happens if a container tries to exceed its memory limit?
- Why set CPU limits?
- How would you determine appropriate limits for your app?

---

## Exercise 7: Scan Image for Vulnerabilities
**Difficulty:** Medium

Use Docker security scanning tools to identify vulnerabilities in images.

**Task:**
1. Build or use an existing image (from previous exercises)

2. Scan the image for vulnerabilities:
   ```bash
   docker scan secure-ubuntu:1.0
   ```

3. Document any vulnerabilities found (if any)

4. Create a Dockerfile using a very old base image to intentionally find vulnerabilities:
   ```dockerfile
   FROM ubuntu:16.04
   RUN apt-get update && apt-get install -y curl
   ```

5. Build it as `old-image:1.0` and scan:
   ```bash
   docker build -t old-image:1.0 .
   docker scan old-image:1.0
   ```

6. Compare the vulnerability count with your secure image

7. Update the Dockerfile to use a newer base image and scan again:
   ```dockerfile
   FROM ubuntu:22.04
   RUN apt-get update && apt-get install -y curl
   ```

8. Build as `new-image:1.0` and scan:
   ```bash
   docker build -t new-image:1.0 .
   docker scan new-image:1.0
   ```

**Questions to Answer:**
- What was the vulnerability count for each image?
- How does the base image version affect security?
- What should you do when vulnerabilities are found?

---

## Exercise 8: Implement Dockerfile Security Best Practices
**Difficulty:** Medium

Create a production-ready Dockerfile incorporating all security best practices.

**Task:**
1. Create a Dockerfile that includes:
   - Specific version tag (not 'latest')
   - Non-root user
   - Minimal base image
   - Only necessary packages installed
   - No default CMD that runs as root
   - Labels for metadata
   - Health check (if applicable)

2. Example structure:
   ```dockerfile
   FROM alpine:3.17
   
   RUN apk add --no-cache curl ca-certificates && \
       rm -rf /var/cache/apk/*
   
   RUN useradd -m -u 1000 appuser
   
   COPY --chown=appuser:appuser . /app/
   
   WORKDIR /app
   
   USER appuser
   
   CMD ["tail", "-f", "/dev/null"]
   ```

3. Build the image as `production-app:1.0`

4. Verify:
   - Image builds successfully
   - Container runs as non-root user
   - Image size is reasonable

5. Run with full security options:
   ```bash
   docker run -d \
     --name prod-secure \
     --read-only \
     --cap-drop=ALL \
     --memory=256m \
     --cpus=0.5 \
     --security-opt=no-new-privileges:true \
     --tmpfs /tmp \
     production-app:1.0
   ```

6. Verify it runs correctly:
   ```bash
   docker exec prod-secure id
   docker exec prod-secure ls -la /
   ```

---

## Exercise 9: Create a Secrets File (Without Using Environment Variables)
**Difficulty:** Medium

Understand the difference between passing secrets as environment variables (BAD) vs other methods (GOOD).

**Task:**
1. Create a secret file:
   ```bash
   echo "my-database-password" > db-secret.txt
   ```

2. Run a container with the secret as a volume mount (NOT env var):
   ```bash
   docker run -d \
     --name secret-app \
     -v $(pwd)/db-secret.txt:/run/secrets/db_password:ro \
     ubuntu:22.04 sleep 300
   ```

3. Read the secret inside the container:
   ```bash
   docker exec secret-app cat /run/secrets/db_password
   ```

4. Now inspect the container - verify the secret is NOT in environment variables:
   ```bash
   docker inspect secret-app | grep -i env
   docker inspect secret-app | grep -i "db.password"
   ```

5. Cleanup the secret file after testing:
   ```bash
   rm db-secret.txt
   docker stop secret-app && docker rm secret-app
   ```

**Questions to Answer:**
- Why is using volume mounts better than environment variables?
- Where would the password be visible if you used `-e DB_PASSWORD=...`?
- What real-world tools could you use instead of files for secrets?

---

## Exercise 10: Audit and Monitor Container Security
**Difficulty:** Medium

Learn to inspect containers for security configurations and monitor their behavior.

**Task:**
1. Create and run a secure container from a previous exercise:
   ```bash
   docker run -d \
     --name audit-test \
     --user 1000 \
     --read-only \
     --cap-drop=ALL \
     --memory=256m \
     --security-opt=no-new-privileges:true \
     ubuntu:22.04 sleep 300
   ```

2. Inspect the container's security settings:
   ```bash
   docker inspect audit-test | jq '.[] | {User, ReadonlyRootfs, SecurityOpt, HostConfig}'
   ```

3. Document each security setting and its value

4. Check running processes:
   ```bash
   docker top audit-test
   ```

5. View container logs:
   ```bash
   docker logs audit-test
   ```

6. Get real-time statistics:
   ```bash
   docker stats audit-test --no-stream
   ```

7. Create a comprehensive audit report documenting:
   - Container user and UID
   - Filesystem access level (read-only or not)
   - Applied capabilities
   - Resource limits
   - Running processes
   - Security options

**Questions to Answer:**
- How would you audit containers for compliance?
- What should be included in a security audit?
- How would you automate this auditing process?

---

## Summary

You've now completed all 10 exercises covering:
- ✓ Container user contexts
- ✓ Non-root user implementation
- ✓ Image optimization
- ✓ Read-only filesystems
- ✓ Linux capabilities
- ✓ Resource limits
- ✓ Vulnerability scanning
- ✓ Security best practices
- ✓ Secrets management
- ✓ Security auditing

**Next Step:** Check [solutions.md](solutions.md) for detailed solutions and explanations.

**Time Estimate:** 90-120 minutes total  
**Difficulty Progression:** Easy → Medium
