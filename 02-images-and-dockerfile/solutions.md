# Solutions: Images and Dockerfile

Detailed solutions for all Dockerfile exercises.

---

## Solution 1: Basic Dockerfile

### Steps

```bash
mkdir node-app && cd node-app

# Create app.js
cat > app.js << 'EOF'
const http = require('http');

const server = http.createServer((req, res) => {
  res.writeHead(200, {'Content-Type': 'text/plain'});
  res.end('Hello from Docker!\n');
});

server.listen(3000, '0.0.0.0', () => {
  console.log('Server running on port 3000');
});
EOF

# Create Dockerfile
cat > Dockerfile << 'EOF'
FROM node:18-slim
WORKDIR /app
COPY app.js .
EXPOSE 3000
CMD ["node", "app.js"]
EOF

# Build image
docker build -t node-app:1.0 .

# Verify
docker images | grep node-app
```

### Explanation
- Base image includes Node.js runtime
- WORKDIR sets /app as working directory
- COPY brings app.js into container
- EXPOSE documents port usage
- CMD specifies default start command

---

## Solution 2: Multi-line RUN Commands

### Dockerfile

```dockerfile
FROM ubuntu:22.04

# Bad: Creates multiple layers (inefficient)
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y wget
RUN apt-get install -y git

# Good: Single layer with multiple commands
RUN apt-get update && \
    apt-get install -y curl wget git && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

### Explanation
- Multiple RUN commands = multiple layers = larger image
- Combined with && = single layer = smaller image
- `apt-get clean` removes package cache
- Each layer adds disk space

### Building and Testing

```bash
# First build (caches all layers)
docker build -t multi-run:1.0 .

# Modify Dockerfile (change git to vim)
# Second build (reuses cached layers up to change point)
docker build -t multi-run:2.0 .
```

---

## Solution 3: .dockerignore File

### .dockerignore

```
.git
.gitignore
node_modules
npm-debug.log
.env
.DS_Store
*.log
.vscode
.idea
test
```

### Dockerfile

```dockerfile
FROM node:18-slim
WORKDIR /app
COPY . .
RUN npm install
CMD ["npm", "start"]
```

### Compare Sizes

```bash
# Without .dockerignore
docker build -t myapp:no-ignore .
# Build context: ~150MB (includes node_modules, .git, etc.)

# With .dockerignore
docker build -t myapp:with-ignore .
# Build context: ~2MB (excludes unnecessary files)
```

### Result
- Build is 10-20x faster with .dockerignore
- Excludes dev dependencies, version control, caches
- Only sends relevant files to Docker daemon

---

## Solution 4: Working Directory

### Dockerfile

```dockerfile
FROM ubuntu:22.04
WORKDIR /app
RUN mkdir -p subdir
WORKDIR subdir
RUN touch file.txt
WORKDIR /app
```

### Test

```bash
docker build -t workdir-test .
docker run -it workdir-test pwd
# Output: /app (current WORKDIR at end)

docker run -it workdir-test ls -la subdir/
# Output: file.txt exists (created in WORKDIR subdir)
```

### Key Points
- WORKDIR creates directory if doesn't exist
- All subsequent commands use that working directory
- Default is / if not specified
- Multiple WORKDIR commands are cumulative

---

## Solution 5: Environment Variables

### app.py

```python
import os
from flask import Flask

app = Flask(__name__)
port = int(os.getenv('APP_PORT', 5000))
debug = os.getenv('DEBUG', 'False') == 'True'

@app.route('/')
def hello():
    return {'message': 'Hello!', 'debug': debug}

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=port, debug=debug)
```

### Dockerfile

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install flask
COPY app.py .
ENV APP_PORT=8080
ENV DEBUG=false
EXPOSE 8080
CMD ["python", "app.py"]
```

### Testing

```bash
# Build
docker build -t env-test .

# Run with default ENV
docker run -p 8080:8080 env-test
# Uses APP_PORT=8080, DEBUG=false

# Override at runtime
docker run -p 9000:9000 -e APP_PORT=9000 -e DEBUG=true env-test
# Uses APP_PORT=9000, DEBUG=true
```

---

## Solution 6: Exposing Ports

### Flask App

```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello():
    return 'Hello from Docker!'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

### Dockerfile

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install flask
COPY app.py .
EXPOSE 5000
CMD ["python", "app.py"]
```

### Run and Test

```bash
docker build -t flask-app .
docker run -d --name flask -p 8000:5000 flask-app

# Access from host
curl http://localhost:8000
# Output: Hello from Docker!
```

### Explanation
- EXPOSE documents container port (doesn't publish)
- -p actually publishes port to host
- Format: -p [hostPort]:[containerPort]

---

## Solution 7: CMD vs ENTRYPOINT

### Version 1: Using CMD

```dockerfile
FROM ubuntu:22.04
CMD ["echo", "Hello from CMD"]
```

### Version 2: Using ENTRYPOINT

```dockerfile
FROM ubuntu:22.04
ENTRYPOINT ["echo"]
CMD ["Hello from ENTRYPOINT"]
```

### Testing

```bash
# Build both
docker build -t cmd-test -f Dockerfile.cmd .
docker build -t entrypoint-test -f Dockerfile.entrypoint .

# Test CMD version
docker run cmd-test
# Output: Hello from CMD
docker run cmd-test echo "Override"
# Output: Override (CMD replaced)

# Test ENTRYPOINT version
docker run entrypoint-test
# Output: Hello from ENTRYPOINT
docker run entrypoint-test "Custom message"
# Output: Custom message (appends to ENTRYPOINT)
```

### Key Differences
- **CMD:** Can be completely replaced by docker run arguments
- **ENTRYPOINT:** Arguments append to entrypoint command
- Use ENTRYPOINT for fixed executable, CMD for parameters

---

## Solution 8: Multi-stage Build

### Without Multi-stage (Large)

```dockerfile
FROM golang:1.20
WORKDIR /app
COPY . .
RUN go build -o app main.go
EXPOSE 8080
CMD ["./app"]
# Final image includes Go compiler (~800MB)
```

### With Multi-stage (Small)

```dockerfile
# Build stage
FROM golang:1.20 AS builder
WORKDIR /app
COPY . .
RUN go build -o app main.go

# Runtime stage
FROM alpine:latest
WORKDIR /app
COPY --from=builder /app/app .
EXPOSE 8080
CMD ["./app"]
# Final image is only alpine (~10MB)
```

### Compare

```bash
# Build single-stage
docker build -t app:single -f Dockerfile.single .
# Size: ~800MB

# Build multi-stage
docker build -t app:multi -f Dockerfile.multi .
# Size: ~20MB

docker images | grep app
# Shows size difference of 40x!
```

---

## Solution 9: Build Arguments

### Dockerfile

```dockerfile
ARG BASE_VERSION=22.04
FROM ubuntu:${BASE_VERSION}
ARG APP_NAME=myapp
LABEL app=${APP_NAME}
RUN echo "Building ${APP_NAME} on Ubuntu ${BASE_VERSION}"
```

### Build Multiple Versions

```bash
# Build with default (22.04)
docker build -t myapp:ubuntu-22 .

# Build with 20.04
docker build -t myapp:ubuntu-20 --build-arg BASE_VERSION=20.04 .

# Verify
docker inspect myapp:ubuntu-22 | grep -A5 '"Config"'
docker inspect myapp:ubuntu-20 | grep -A5 '"Config"'
```

---

## Solution 10: Layer Caching Optimization

### Dockerfile (Optimized)

```dockerfile
FROM python:3.11-slim
WORKDIR /app

# Copy requirements first (changes less frequently)
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code (changes frequently)
COPY app.py .

EXPOSE 5000
CMD ["python", "app.py"]
```

### Dockerfile (Unoptimized)

```dockerfile
FROM python:3.11-slim
WORKDIR /app

# Copy everything first
COPY . .
RUN pip install --no-cache-dir -r requirements.txt

EXPOSE 5000
CMD ["python", "app.py"]
```

### Testing Cache

```bash
# First build
time docker build -t app:opt .
# Takes ~30 seconds (installs all layers)

# Modify only app.py
echo "# comment" >> app.py

# Rebuild optimized (reuses cached layers)
time docker build -t app:opt .
# Takes ~2 seconds (reuses pip layer)

# Rebuild unoptimized (reinstalls everything)
time docker build -t app:unopt .
# Takes ~30 seconds (pip layer invalidated)
```

### Key Learning
- Layers are cached by content hash
- If layer input changes, cache is invalidated
- Place frequently-changed files last
- Results in 10-15x faster rebuilds

---

## Summary: Dockerfile Best Practices

1. **Use .dockerignore** - Smaller build context
2. **Multi-line RUN** - Fewer layers
3. **Order matters** - Dependencies before code
4. **Use official images** - Security and size
5. **Remove unnecessary files** - Smaller images
6. **Multi-stage builds** - Reduced final size
7. **Set WORKDIR** - Organized structure
8. **Use ENTRYPOINT** - Better entry point control
9. **Document with EXPOSE** - Port clarity
10. **Leverage caching** - Faster rebuilds

---

**Next:** Review [quiz.md](quiz.md) to test your knowledge!

