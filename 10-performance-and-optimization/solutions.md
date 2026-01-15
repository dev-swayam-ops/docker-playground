# Solutions: Performance & Optimization

## Exercise 1: Set memory limits

**Objective**: Run a container with 256MB memory limit using `--memory=256m`.

### Commands

```bash
# Run container with 256MB memory limit
docker run -d --name memory-limited --memory=256m nginx

# Expected output:
# <container-id>

# Verify the memory limit was applied
docker inspect memory-limited | grep -A 5 "Memory"

# Expected output:
# "Memory": 268435456,  (this is 256MB in bytes)

# Check memory usage with stats
docker stats --no-stream memory-limited

# Expected output:
# CONTAINER ID   NAME              CPU %   MEM USAGE / LIMIT   MEM %
# <id>           memory-limited    0.1%    2.3MiB / 256MiB     0.90%

# Test the limit by running container that needs more memory
docker run -d --name mem-test --memory=256m alpine sh -c "dd if=/dev/urandom bs=1M count=300 of=/dev/null" 2>&1

# Expected output:
# Container will be killed/stopped when exceeding 256MB

# Verify container stopped due to memory limit
docker ps -a | grep mem-test

# Expected output:
# Shows container in Exited state

# Cleanup
docker stop memory-limited mem-test 2>/dev/null
docker rm memory-limited mem-test 2>/dev/null
```

### Explanation

- `--memory=256m`: Sets hard memory limit to 256MB
- If container tries to use more, kernel kills the process
- Memory units: b (bytes), k (kilobytes), m (megabytes), g (gigabytes)
- Good practice for production to prevent OOM (Out Of Memory) issues

---

## Exercise 2: Set CPU limits

**Objective**: Run a container restricted to 0.5 CPUs using `--cpus=0.5`.

### Commands

```bash
# Run container with 0.5 CPU limit
docker run -d --name cpu-limited --cpus=0.5 alpine sh -c "while true; do echo 'calculating'; done"

# Expected output:
# <container-id>

# Verify CPU limit was applied
docker inspect cpu-limited | grep -A 2 "CpuQuota"

# Expected output:
# "CpuQuota": 50000,
# "CpuPeriod": 100000,

# Monitor CPU usage with stats
docker stats --no-stream cpu-limited

# Expected output:
# CONTAINER ID   NAME          CPU %    MEM USAGE / LIMIT   MEM %
# <id>           cpu-limited   49.5%    0.9MiB / 1.95GiB    0.04%

# (Shows ~49.5% because limited to 0.5 CPU)

# Create container without limit for comparison
docker run -d --name cpu-unlimited alpine sh -c "while true; do echo 'calculating'; done"

# Expected output:
# <container-id>

# Compare CPU usage
docker stats --no-stream cpu-limited cpu-unlimited

# Expected output:
# cpu-limited shows ~50% CPU
# cpu-unlimited shows ~100% CPU (uses full core)

# Cleanup
docker stop cpu-limited cpu-unlimited 2>/dev/null
docker rm cpu-limited cpu-unlimited 2>/dev/null
```

### Explanation

- `--cpus=0.5`: Limits container to 0.5 CPU cores (half a core)
- Internally uses CpuQuota and CpuPeriod (CpuQuota/CpuPeriod = CPU limit)
- `--cpus=1` = 1 full CPU core
- `--cpus=2` = 2 CPU cores
- Useful for multi-container deployments to prevent single container from consuming all CPU

---

## Exercise 3: Monitor container stats

**Objective**: Use `docker stats` to track resource usage over time.

### Commands

```bash
# Start a few containers with different workloads
docker run -d --name idle-container nginx

docker run -d --name cpu-hog alpine sh -c "while true; do sha256sum < /dev/urandom; done"

# Expected output:
# <container-ids>

# View live stats (press Ctrl+C to exit)
docker stats

# Expected output:
# CONTAINER ID   NAME              CPU %   MEM USAGE / LIMIT   MEM %   NET I/O        BLOCK I/O
# <id>           idle-container    0.1%    5.3MiB / 1.95GiB    0.26%   0B / 0B        0B / 0B
# <id>           cpu-hog           98.7%   1.2MiB / 1.95GiB    0.06%   0B / 0B        0B / 0B

# View stats snapshot (no streaming)
docker stats --no-stream

# Expected output:
# Single snapshot of all containers

# Monitor specific container
docker stats --no-stream idle-container

# Expected output:
# Stats for only idle-container

# Monitor multiple specific containers
docker stats --no-stream idle-container cpu-hog

# Expected output:
# Stats for both containers

# Custom format - show only container name and CPU
docker stats --no-stream --format "table {{.Container}}\t{{.CPUPerc}}"

# Expected output:
# CONTAINER       CPU %
# idle-conta      0.1%
# cpu-hog         98.7%

# Monitor in loop with timestamps
for i in {1..3}; do
  echo "=== Sample $i at $(date +%H:%M:%S) ==="
  docker stats --no-stream --format "{{.Container}}: CPU={{.CPUPerc}} MEM={{.MemUsage}}"
  sleep 2
done

# Expected output:
# Multiple samples showing changes over time

# Cleanup
docker stop idle-container cpu-hog 2>/dev/null
docker rm idle-container cpu-hog 2>/dev/null
```

### Explanation

- `docker stats`: Shows real-time resource usage (CPU, Memory, Network I/O, Block I/O)
- Updates every 2 seconds by default
- `--no-stream`: Shows single snapshot instead of live updates
- `--format`: Customize output (e.g., show only specific fields)
- Useful for monitoring which containers consume most resources

---

## Exercise 4: Compare image sizes

**Objective**: Build two versions of an image (standard and slim base) and compare sizes.

### Commands

```bash
# Create a simple Dockerfile with standard Python base
cat > Dockerfile.standard << 'EOF'
FROM python:3.11
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
CMD ["python", "app.py"]
EOF

# Create same Dockerfile with slim base
cat > Dockerfile.slim << 'EOF'
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
CMD ["python", "app.py"]
EOF

# Create requirements.txt
echo "flask" > requirements.txt

# Create dummy app.py
echo "print('Hello')" > app.py

# Build standard version
docker build -f Dockerfile.standard -t myapp:standard .

# Expected output:
# Successfully tagged myapp:standard

# Build slim version
docker build -f Dockerfile.slim -t myapp:slim .

# Expected output:
# Successfully tagged myapp:slim

# List both images to compare sizes
docker image ls myapp

# Expected output:
# REPOSITORY   TAG        IMAGE ID       SIZE
# myapp        standard   abc123def456   895MB
# myapp        slim       def456abc789   345MB

# Calculate size difference
STANDARD_SIZE=$(docker image inspect myapp:standard --format='{{.Size}}')
SLIM_SIZE=$(docker image inspect myapp:slim --format='{{.Size}}')
DIFFERENCE=$((STANDARD_SIZE - SLIM_SIZE))
PERCENT=$((DIFFERENCE * 100 / STANDARD_SIZE))

echo "Standard size: $((STANDARD_SIZE / 1024 / 1024))MB"
echo "Slim size: $((SLIM_SIZE / 1024 / 1024))MB"
echo "Difference: $((DIFFERENCE / 1024 / 1024))MB ($PERCENT% smaller)"

# Expected output:
# Standard size: 895MB
# Slim size: 345MB
# Difference: 550MB (61% smaller)

# View image layers comparison
echo "=== Standard image layers ==="
docker history myapp:standard

echo "=== Slim image layers ==="
docker history myapp:slim

# Expected output:
# Slim has smaller layers due to minimal base

# Cleanup
docker rmi myapp:standard myapp:slim 2>/dev/null
rm -f Dockerfile.standard Dockerfile.slim requirements.txt app.py
```

### Explanation

- Standard Python base image (~895MB) includes build tools, documentation, locale data
- Slim variant (~345MB) removes non-essential packages (61% smaller)
- Alpine variant would be even smaller (~165MB, 82% reduction)
- Trade-off: slim/alpine may have compatibility issues but save significant space
- Important for production deployments and container registries

---

## Exercise 5: Optimize Dockerfile

**Objective**: Reduce a Dockerfile by combining RUN commands with && and ; operators.

### Commands

```bash
# Create non-optimized Dockerfile with separate RUN commands
cat > Dockerfile.bad << 'EOF'
FROM ubuntu:22.04
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y wget
RUN apt-get install -y git
RUN rm -rf /var/lib/apt/lists/*
WORKDIR /app
COPY . .
CMD ["bash"]
EOF

# Create optimized Dockerfile with combined RUN commands
cat > Dockerfile.good << 'EOF'
FROM ubuntu:22.04
RUN apt-get update && apt-get install -y curl wget git && rm -rf /var/lib/apt/lists/*
WORKDIR /app
COPY . .
CMD ["bash"]
EOF

# Build non-optimized version
docker build -f Dockerfile.bad -t myapp:bad .

# Expected output:
# Step 1/6: FROM ubuntu:22.04
# Step 2/6: RUN apt-get update
# ...
# Successfully tagged myapp:bad

# Build optimized version
docker build -f Dockerfile.good -t myapp:good .

# Expected output:
# Successfully tagged myapp:good

# Compare layer counts
BAD_LAYERS=$(docker history myapp:bad | wc -l)
GOOD_LAYERS=$(docker history myapp:good | wc -l)

echo "Non-optimized layers: $BAD_LAYERS"
echo "Optimized layers: $GOOD_LAYERS"

# Expected output:
# Non-optimized layers: 9
# Optimized layers: 6

# View image sizes
docker image ls myapp

# Expected output:
# REPOSITORY   TAG    IMAGE ID       SIZE
# myapp        bad    abc123def456   100MB
# myapp        good   def456abc789   95MB

# View layers detail
echo "=== Non-optimized Dockerfile ==="
cat Dockerfile.bad

echo "=== Optimized Dockerfile ==="
cat Dockerfile.good

# Expected output:
# Shows the difference in structure

# Cleanup
docker rmi myapp:bad myapp:good 2>/dev/null
rm -f Dockerfile.bad Dockerfile.good
```

### Explanation

- Each `RUN` instruction creates a new layer
- Non-optimized: 5 separate RUN commands = 5 layers
- Optimized: 1 RUN with && chains = 1 layer
- Combining RUN commands reduces image size and layer count
- Always clean up package manager cache in same RUN: `rm -rf /var/lib/apt/lists/*`
- Reduces image bloat from intermediate layers

---

## Exercise 6: Create multi-stage build

**Objective**: Build an image using two stages to reduce final size.

### Commands

```bash
# Create multi-stage Dockerfile
cat > Dockerfile.multistage << 'EOF'
# Stage 1: Builder
FROM golang:1.20 AS builder
WORKDIR /build
COPY main.go .
RUN go build -o app main.go

# Stage 2: Runtime
FROM alpine:3.18
WORKDIR /app
COPY --from=builder /build/app .
CMD ["./app"]
EOF

# Create sample Go application
cat > main.go << 'EOF'
package main
import (
    "fmt"
    "os"
)
func main() {
    fmt.Println("Hello from Go!")
}
EOF

# Build multi-stage image
docker build -f Dockerfile.multistage -t myapp:multistage .

# Expected output:
# Step 1/8: FROM golang:1.20 AS builder
# Step 2/8: WORKDIR /build
# ...
# Successfully tagged myapp:multistage

# Create single-stage Dockerfile for comparison
cat > Dockerfile.single << 'EOF'
FROM golang:1.20
WORKDIR /app
COPY main.go .
RUN go build -o app main.go
CMD ["./app"]
EOF

# Build single-stage image
docker build -f Dockerfile.single -t myapp:single .

# Expected output:
# Successfully tagged myapp:single

# Compare image sizes
docker image ls myapp

# Expected output:
# REPOSITORY   TAG           IMAGE ID       SIZE
# myapp        multistage    abc123def456   15MB
# myapp        single        def456abc789   800MB

# Calculate savings
MULTI=$(docker image inspect myapp:multistage --format='{{.Size}}')
SINGLE=$(docker image inspect myapp:single --format='{{.Size}}')
SAVINGS=$((SINGLE - MULTI))
PERCENT=$((SAVINGS * 100 / SINGLE))

echo "Multi-stage size: $((MULTI / 1024 / 1024))MB"
echo "Single-stage size: $((SINGLE / 1024 / 1024))MB"
echo "Saved: $((SAVINGS / 1024 / 1024))MB ($PERCENT% reduction)"

# Expected output:
# Multi-stage size: 15MB
# Single-stage size: 800MB
# Saved: 785MB (98% reduction)

# View which layers are included in final image
echo "=== Multi-stage final image (includes only runtime stage) ==="
docker history myapp:multistage

echo "=== Single-stage image (includes builder tools) ==="
docker history myapp:single

# Expected output:
# Multi-stage shows only alpine base + binary
# Single-stage shows golang toolchain + binary

# Cleanup
docker rmi myapp:multistage myapp:single 2>/dev/null
rm -f Dockerfile.multistage Dockerfile.single main.go
```

### Explanation

- Multi-stage builds use multiple `FROM` statements
- Builder stage (Stage 1): Compiles/builds the application (large)
- Runtime stage (Stage 2): Only includes compiled binary (small)
- `COPY --from=builder`: Copy artifacts from previous stage
- Final image only includes runtime stage, builder tools are discarded
- Can achieve 90-98% size reduction for compiled languages

---

## Exercise 7: Verify layer caching

**Objective**: Make multiple changes to a Dockerfile and observe which layers are cached.

### Commands

```bash
# Create initial Dockerfile
cat > Dockerfile << 'EOF'
FROM ubuntu:22.04
RUN apt-get update && apt-get install -y curl
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY app.py .
CMD ["python", "app.py"]
EOF

# Create files
echo "flask" > requirements.txt
echo "print('hello')" > app.py

# Build 1: Initial build (no cache)
echo "=== Build 1: Initial build ==="
docker build -t myapp:v1 .

# Expected output:
# Step 1: FROM ubuntu:22.04
# Step 2: RUN apt-get update && apt-get install -y curl
# ...
# Successfully tagged myapp:v1

# Build 2: Rebuild without changes (full cache)
echo "=== Build 2: Rebuild (full cache) ==="
docker build -t myapp:v2 .

# Expected output:
# Step 1/6: FROM ubuntu:22.04 (Using cache)
# Step 2/6: RUN apt-get update && apt-get install -y curl (Using cache)
# ...
# (Much faster, uses cache for all steps)

# Build 3: Change app.py
echo "print('hello world')" > app.py

echo "=== Build 3: After changing app.py ==="
docker build -t myapp:v3 .

# Expected output:
# Step 1-4: Using cache (FROM, RUN, WORKDIR, COPY requirements.txt, RUN pip)
# Step 5: COPY app.py . (not cached - file changed)
# Step 6: CMD (not cached - depends on previous step)

# Build 4: Change requirements.txt
echo -e "flask\nrequests" > requirements.txt

echo "=== Build 4: After changing requirements.txt ==="
docker build -t myapp:v4 .

# Expected output:
# Step 1-3: Using cache (FROM, RUN apt, WORKDIR)
# Step 4: COPY requirements.txt . (not cached - file changed)
# Step 5: RUN pip install (not cached - depends on previous)
# Step 6-7: Not cached

# Demonstrate cache invalidation
echo "=== Layer caching breakdown ==="
echo "Layers are cached if:"
echo "1. Base image unchanged"
echo "2. RUN command unchanged"
echo "3. Files (COPY/ADD) unchanged"
echo ""
echo "Layers are NOT cached if:"
echo "1. Previous layer was not cached"
echo "2. Source files changed"
echo "3. RUN command changed"

# Cleanup
docker rmi myapp:v1 myapp:v2 myapp:v3 myapp:v4 2>/dev/null
rm -f Dockerfile requirements.txt app.py
```

### Explanation

- Docker caches each layer after successful build
- If nothing changes, Docker reuses the cached layer (very fast)
- If a layer changes, all following layers are invalidated and rebuilt
- Order matters: Put stable instructions (FROM, RUN system stuff) before changing ones (COPY source)
- Build cache key includes: base image, command, file contents
- Leveraging cache dramatically speeds up iterative development

---

## Exercise 8: Use .dockerignore

**Objective**: Create a .dockerignore file to exclude unnecessary files from the build.

### Commands

```bash
# Create project structure
mkdir -p myproject/{src,tests,.git}
echo "source code" > myproject/src/main.py
echo "test code" > myproject/tests/test.py
echo "git data" > myproject/.git/config
echo "SECRET_KEY=xyz" > myproject/.env

# Create basic Dockerfile
cat > myproject/Dockerfile << 'EOF'
FROM python:3.11-slim
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt 2>/dev/null || true
CMD ["python", "src/main.py"]
EOF

cat > myproject/requirements.txt << 'EOF'
flask
EOF

# Build WITHOUT .dockerignore
echo "=== Build 1: Without .dockerignore ==="
docker build -t myapp:without -f myproject/Dockerfile myproject/

# Expected output:
# COPY . . includes: src/, tests/, .git/, .env, etc.
# Build context includes all files

# Check build context size (conceptually larger)
# Expected: All files included

# Create .dockerignore
cat > myproject/.dockerignore << 'EOF'
.git
.gitignore
.env
tests
__pycache__
*.pyc
.pytest_cache
.vscode
.idea
EOF

# Build WITH .dockerignore
echo "=== Build 2: With .dockerignore ==="
docker build -t myapp:with -f myproject/Dockerfile myproject/

# Expected output:
# COPY . . now excludes files listed in .dockerignore
# Build is faster (smaller context)
# Image is smaller (fewer files copied)

# Verify final images
docker image ls myapp

# Expected output:
# REPOSITORY   TAG      IMAGE ID       SIZE
# myapp        without  abc123def456   200MB
# myapp        with     def456abc789   185MB

# View what's in each container
echo "=== Files in 'without' ==="
docker run --rm myapp:without find /app -type f | head -10

echo "=== Files in 'with' ==="
docker run --rm myapp:with find /app -type f | head -10

# Expected output:
# 'without' has .git, tests, .env files
# 'with' does not have those files

# Demonstrate .dockerignore syntax
cat > .dockerignore-examples << 'EOF'
# .dockerignore syntax

# Exclude specific files
.env
.git
.gitignore
.DS_Store

# Exclude directories
tests/
node_modules/
.vscode/

# Exclude by pattern
*.log
*.pyc
__pycache__/

# Negation: exclude pattern but include specific file
*.log          # Exclude all .log files
!important.log # But include important.log

# Comments and empty lines
# This is a comment (ignored)
               # Empty line (ignored)
EOF

cat .dockerignore-examples

# Cleanup
docker rmi myapp:without myapp:with 2>/dev/null
rm -rf myproject .dockerignore-examples
```

### Explanation

- `.dockerignore`: Same syntax as `.gitignore`
- Files listed are excluded from build context
- Reduces build context size (faster build)
- Excludes sensitive files (.env, .git, .ssh)
- Common exclusions: .git, tests/, node_modules/, .env, *.log, __pycache__/
- Improves security by preventing secrets from leaking into images

---

## Exercise 9: Measure startup time

**Objective**: Time container startup with and without resource limits.

### Commands

```bash
# Create test application with startup delay
cat > app.py << 'EOF'
import time
import sys

print("Starting application...", flush=True)
time.sleep(2)  # Simulate startup work
print("Application ready!", flush=True)
sys.stdout.flush()

# Keep running
time.sleep(60)
EOF

# Create Dockerfile
cat > Dockerfile << 'EOF'
FROM python:3.11-slim
WORKDIR /app
COPY app.py .
CMD ["python", "app.py"]
EOF

# Build image
docker build -t myapp:timing .

# Expected output:
# Successfully tagged myapp:timing

# Measure startup WITHOUT limits
echo "=== Startup time WITHOUT resource limits ==="
time docker run --rm myapp:timing

# Expected output:
# real    0m2.XXXs (approximately 2 seconds for script to run)

# Measure startup WITH memory limit
echo "=== Startup time WITH memory limit (512MB) ==="
time docker run --rm --memory=512m myapp:timing

# Expected output:
# real    0m2.XXXs (may be slightly slower due to memory pressure)

# Measure startup WITH CPU limit
echo "=== Startup time WITH CPU limit (0.5) ==="
time docker run --rm --cpus=0.5 myapp:timing

# Expected output:
# real    0m3.XXXs or more (slower due to CPU throttling)

# Benchmark multiple runs
echo "=== Benchmark: 3 runs without limits ==="
for i in {1..3}; do
  echo "Run $i:"
  time docker run --rm myapp:timing > /dev/null 2>&1
done

# Expected output:
# Multiple timing measurements showing consistency

echo "=== Benchmark: 3 runs with --cpus=0.5 ==="
for i in {1..3}; do
  echo "Run $i:"
  time docker run --rm --cpus=0.5 myapp:timing > /dev/null 2>&1
done

# Expected output:
# Slower times compared to runs without limits

# Cleanup
docker rmi myapp:timing 2>/dev/null
rm -f app.py Dockerfile
```

### Explanation

- Startup time includes: container initialization + application startup
- Resource limits impact startup time (CPU limit slows startup)
- Memory limits less likely to affect startup unless app needs significant memory
- `time` command provides real/user/sys metrics
- Real = wall-clock time (what users experience)
- Useful for performance optimization and SLA planning
- Can identify bottlenecks in startup sequence

---

## Exercise 10: Profile image layers

**Objective**: Use `docker history [image]` to analyze layer sizes and content.

### Commands

```bash
# Create multi-layer Dockerfile
cat > Dockerfile << 'EOF'
FROM python:3.11
RUN apt-get update && apt-get install -y curl wget git
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["python", "app.py"]
EOF

# Create files
echo "flask" > requirements.txt
echo "print('hello')" > app.py

# Build image
docker build -t myapp:profiled .

# Expected output:
# Successfully tagged myapp:profiled

# View complete layer history
echo "=== Complete layer history ==="
docker history myapp:profiled

# Expected output:
# IMAGE        CREATED          CREATED BY                          SIZE     COMMENT
# <id>         1 minute ago     /bin/sh -c #(nop) CMD ...           0B
# <id>         1 minute ago     /bin/sh -c #(nop) COPY file:...     150B
# <id>         1 minute ago     /bin/sh -c pip install -r req...    45MB
# <id>         2 minutes ago    /bin/sh -c #(nop) COPY file:...     12B
# <id>         2 minutes ago    /bin/sh -c #(nop) WORKDIR ...       0B
# <id>         5 minutes ago    /bin/sh -c apt-get update && ...    75MB
# <id>         2 weeks ago      /bin/sh -c #(nop) WORKDIR ...       0B
# <id>         2 weeks ago      /bin/sh -c #(nop)  ENV ...          0B

# View without truncation
echo "=== Full layer details (no truncation) ==="
docker history myapp:profiled --no-trunc

# Expected output:
# Shows complete commands for each layer

# Analyze largest layers
echo "=== Largest layers ==="
docker history myapp:profiled --format "table {{.Size}}\t{{.CreatedBy}}" | sort -hr | head -5

# Expected output:
# 75MB    /bin/sh -c apt-get update && apt-get install -y curl wget git
# 45MB    /bin/sh -c pip install -r requirements.txt
# 150B    /bin/sh -c #(nop) COPY file:...
# 12B     /bin/sh -c #(nop) COPY file:...

# Get total image size
echo "=== Total image size ==="
docker image inspect myapp:profiled --format='{{.Size}}' | awk '{printf "%.2f MB\n", $1/1024/1024}'

# Expected output:
# 120.34 MB

# Analyze specific layer command
echo "=== Layer created by apt-get ==="
docker history myapp:profiled --format "table {{.CreatedBy}}\t{{.Size}}" | grep apt-get

# Expected output:
# Shows apt layer and its size (75MB)

# Identify optimization opportunities
echo "=== Optimization analysis ==="
echo "Largest layer: apt-get (75MB)"
echo "  - Could save space by removing package manager cache"
echo ""
echo "Second largest: pip install (45MB)"
echo "  - Could use --no-cache-dir flag to reduce size"

# Cleanup
docker rmi myapp:profiled 2>/dev/null
rm -f Dockerfile requirements.txt app.py
```

### Explanation

- `docker history [image]`: Shows all layers and their sizes
- Each line represents one layer (one RUN, COPY, FROM, etc.)
- `--no-trunc`: Show full commands (not truncated)
- Largest layers are optimization targets
- Can identify redundant or unnecessary operations
- Common optimizations:
  - Remove apt cache: `rm -rf /var/lib/apt/lists/*`
  - Use pip `--no-cache-dir` flag
  - Combine RUN commands to reduce layers
  - Use .dockerignore to reduce COPY size
- Total image size = sum of all layers (with deduplication for shared base layers)
