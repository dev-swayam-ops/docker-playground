# Module 02: Images and Dockerfile

Learn how to create custom Docker images using Dockerfiles. Master the fundamentals of containerizing applications.

## What You'll Learn

- How to understand Docker images and layers
- How to write efficient Dockerfiles
- How to use FROM, RUN, COPY, ADD, CMD, and ENTRYPOINT instructions
- How to build images from Dockerfiles
- How to optimize image size and build time
- How to handle multi-stage builds
- How to use .dockerignore for efficient builds

## Prerequisites

- Docker installed and running (Module 00)
- Understanding of containers and container lifecycle (Modules 01 & 03)
- Basic understanding of Linux commands
- Text editor for writing Dockerfiles

## Key Concepts

### Dockerfile
A text file containing instructions to build a Docker image. Each instruction creates a new layer in the image.

### Layer
Each instruction in a Dockerfile creates a layer. Docker caches layers for faster rebuilds when nothing changes.

### Base Image
The starting point for your Dockerfile, specified in the FROM instruction. Examples: ubuntu, alpine, node, python.

### Build Context
All files and directories available during the image build process (the directory where docker build is run).

### .dockerignore
A file that specifies which files/directories to exclude from the build context (similar to .gitignore).

## Hands-on Lab: Build Your First Image

### Lab Objective
Create a simple Python web application, write a Dockerfile, build an image, and run it as a container.

### Step 1: Create Project Structure

```bash
mkdir my-python-app && cd my-python-app
cat > app.py << 'EOF'
from http.server import HTTPServer, SimpleHTTPRequestHandler
import json

class MyHandler(SimpleHTTPRequestHandler):
    def do_GET(self):
        self.send_response(200)
        self.send_header('Content-type', 'application/json')
        self.end_headers()
        response = {"message": "Hello from Docker!", "version": "1.0"}
        self.wfile.write(json.dumps(response).encode())

if __name__ == '__main__':
    server = HTTPServer(('0.0.0.0', 8000), MyHandler)
    print("Server running on port 8000...")
    server.serve_forever()
EOF
```

### Step 2: Create Dockerfile

```dockerfile
# Use official Python runtime as base image
FROM python:3.11-slim

# Set working directory
WORKDIR /app

# Copy application files
COPY app.py .

# Expose port
EXPOSE 8000

# Run the application
CMD ["python", "app.py"]
```

### Step 3: Build the Image

```bash
docker build -t my-python-app:1.0 .

# Expected output:
# Step 1/5 : FROM python:3.11-slim
#  ---> abc123def456
# Step 2/5 : WORKDIR /app
#  ---> Running in temp...
# Successfully built abc789def012
# Successfully tagged my-python-app:1.0
```

### Step 4: Run Container from Image

```bash
docker run -d --name my-app -p 8080:8000 my-python-app:1.0

# Test the application
curl http://localhost:8080
# Expected: {"message": "Hello from Docker!", "version": "1.0"}
```

### Step 5: View Image Layers

```bash
docker history my-python-app:1.0
# Shows each layer created by Dockerfile instructions
```

## Validation

Your Dockerfile skills are validated when you can:
- ✅ Write a Dockerfile with appropriate base image
- ✅ Use FROM, RUN, COPY, and CMD instructions correctly
- ✅ Build an image with docker build -t name:tag .
- ✅ Run a container from your custom image
- ✅ Expose ports in Dockerfile
- ✅ Understand image layers and caching

## Cleanup

```bash
docker stop my-app
docker rm my-app
docker rmi my-python-app:1.0
```

## Common Mistakes

### ❌ Mistake 1: Not Using .dockerignore
**Symptom:** Large build context, slow builds
**Solution:** Create .dockerignore to exclude node_modules, .git, etc.

### ❌ Mistake 2: Running as Root
**Symptom:** Security vulnerability
**Solution:** Create non-root user in Dockerfile

### ❌ Mistake 3: Large Images
**Symptom:** Slow deployment
**Solution:** Use slim/alpine base images, multi-stage builds

### ❌ Mistake 4: Not Caching Properly
**Symptom:** Long rebuild times
**Solution:** Copy requirements before source code

### ❌ Mistake 5: Using ADD Instead of COPY
**Symptom:** Unnecessary complexity
**Solution:** Use COPY for files; ADD for tar archives only

## Troubleshooting

### Issue: Build fails with "base image not found"
**Solution:** Verify base image exists: `docker pull ubuntu:22.04`

### Issue: Image is too large
**Solution:** Use alpine images, clean package managers, remove caches

### Issue: "No such file or directory" during COPY
**Solution:** Verify file paths relative to build context

## Next Steps

1. **Module 03: Containers and Lifecycle** - Advanced container management
2. **Module 04: Volumes and Storage** - Persistent data
3. **Module 05: Networking in Docker** - Connect containers

## Quick Reference

| Instruction | Purpose | Example |
|-------------|---------|---------|
| FROM | Base image | FROM ubuntu:22.04 |
| RUN | Execute command | RUN apt-get install -y curl |
| COPY | Copy files | COPY . /app |
| ADD | Copy + extract | ADD archive.tar.gz /app |
| WORKDIR | Set directory | WORKDIR /app |
| EXPOSE | Document port | EXPOSE 8080 |
| CMD | Default command | CMD ["python", "app.py"] |
| ENTRYPOINT | Entry point | ENTRYPOINT ["python"] |
| ENV | Environment var | ENV DEBUG=true |
| USER | Set user | USER appuser |

---

**Need Help?**
- Official Docs: https://docs.docker.com/engine/reference/builder/
- Previous Module: [01-docker-basics](../01-docker-basics/)
- Exercises: [exercises.md](exercises.md)
