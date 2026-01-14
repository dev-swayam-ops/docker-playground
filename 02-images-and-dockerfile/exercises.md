# Exercises: Images and Dockerfile

Practice building Docker images with these hands-on exercises.

## Exercise 1: Basic Dockerfile
**Difficulty:** Easy

Write a simple Dockerfile for a Node.js application.

**Task:**
1. Create directory: `mkdir node-app && cd node-app`
2. Create app.js with a simple "Hello World" HTTP server
3. Write Dockerfile using node:18-slim base image
4. Build image: `docker build -t node-app:1.0 .`
5. Verify image exists: `docker images | grep node-app`

---

## Exercise 2: Multi-line RUN Commands
**Difficulty:** Easy

Optimize Dockerfile using multi-line RUN commands.

**Task:**
1. Create Dockerfile that installs multiple packages
2. Use `&&` and backslash for multi-line commands
3. Build without --no-cache to see layer caching
4. Modify one package and rebuild to see layer reuse

---

## Exercise 3: .dockerignore File
**Difficulty:** Easy

Exclude unnecessary files from build context.

**Task:**
1. Create .dockerignore file
2. Add: .git, node_modules, .env, *.log
3. Create a Dockerfile that copies everything
4. Compare build context size with and without .dockerignore

---

## Exercise 4: Working Directory
**Difficulty:** Easy

Use WORKDIR instruction properly.

**Task:**
1. Create Dockerfile with multiple WORKDIR instructions
2. Test: Run container and verify pwd
3. Understand how WORKDIR affects paths
4. Use WORKDIR for all file operations

---

## Exercise 5: Environment Variables
**Difficulty:** Medium

Configure environment variables in Dockerfile.

**Task:**
1. Create app that uses environment variables
2. Define ENV in Dockerfile (APP_PORT=8080, DEBUG=true)
3. Build and run container
4. Override with docker run -e flag
5. Verify which value is used

---

## Exercise 6: Exposing Ports
**Difficulty:** Medium

Document and publish container ports.

**Task:**
1. Create Python Flask app serving on port 5000
2. Add EXPOSE 5000 in Dockerfile
3. Build image
4. Run with -p 8000:5000 mapping
5. Access from host via localhost:8000

---

## Exercise 7: CMD vs ENTRYPOINT
**Difficulty:** Medium

Understand difference between CMD and ENTRYPOINT.

**Task:**
1. Create two Dockerfiles with CMD and ENTRYPOINT
2. Build both images
3. Run each with and without arguments
4. Observe how each behaves with parameter overrides

---

## Exercise 8: Multi-stage Build
**Difficulty:** Medium

Reduce image size using multi-stage builds.

**Task:**
1. Create Go or Java app Dockerfile
2. Use builder stage for compilation
3. Use runtime stage for execution only
4. Compare final image sizes
5. Document the size reduction percentage

---

## Exercise 9: Build Arguments
**Difficulty:** Medium

Use ARG for flexible image builds.

**Task:**
1. Create Dockerfile with ARG BASE_VERSION=22.04
2. Use in FROM instruction
3. Build with: `docker build --build-arg BASE_VERSION=20.04 .`
4. Create multiple images with different base versions
5. Verify each uses correct base

---

## Exercise 10: Layer Caching Optimization
**Difficulty:** Medium

Optimize Dockerfile layer caching strategy.

**Task:**
1. Create Dockerfile for Python app with pip dependencies
2. Copy requirements.txt before app.py
3. Modify only app.py and rebuild
4. Observe: Only relevant layers rebuild
5. Compare with reverse order (time difference)

---

## Solutions Available

See [solutions.md](solutions.md) for detailed solutions with commands and explanations.

