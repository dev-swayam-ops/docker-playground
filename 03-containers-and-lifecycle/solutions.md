# Solutions: Containers and Lifecycle

Comprehensive solutions for all 10 exercises with commands and explanations.

---

## Exercise 1: Create HEALTHCHECK in Dockerfile for nginx

### Solution

**Dockerfile:**
```dockerfile
FROM nginx:latest

# Add curl for health checks
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*

# Configure health check
HEALTHCHECK --interval=10s --timeout=3s --start-period=40s --retries=3 \
    CMD curl -f http://localhost/ || exit 1

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### Commands

```bash
# Build the image
docker build -t healthy-nginx:1.0 .

# Expected output:
# Sending build context to Docker daemon  2.048kB
# Step 1/4 : FROM nginx:latest
# ...
# Successfully tagged healthy-nginx:1.0

# Run the container
docker run -d --name nginx-health healthy-nginx:1.0

# Expected output:
# a1b2c3d4e5f6

# Check health status
docker inspect --format='{{.State.Health.Status}}' nginx-health

# Expected output:
# healthy

# View detailed health information
docker inspect nginx-health | grep -A 10 '"Health"'

# Expected output:
# "Health": {
#   "Status": "healthy",
#   "FailingStreak": 0,
#   "Passes": 3,
#   "Fails": 0
# }

# Watch health status change
docker inspect --format='{{.State.Health.Status}}' nginx-health
```

### Explanation

**HEALTHCHECK Parameters:**
- `--interval=10s`: Run health check every 10 seconds
- `--timeout=3s`: Wait 3 seconds for health check command to finish
- `--start-period=40s`: Give container 40 seconds to start before failing
- `--retries=3`: Mark unhealthy after 3 consecutive failures

**Health Status Values:**
- `starting`: Container is starting, not yet health-checked
- `healthy`: Health check passed
- `unhealthy`: Health check failed

---

## Exercise 2: Run container with memory and CPU limits

### Solution

### Commands

```bash
# Run container with memory limit (256MB) and CPU limit (0.5 cores)
docker run -d \
  --name limited-container \
  --memory=256m \
  --cpus=0.5 \
  nginx:latest

# Expected output:
# b1c2d3e4f5a6

# Check memory limit
docker inspect limited-container | grep -A 2 '"Memory"'

# Expected output:
# "Memory": 268435456,  (256MB in bytes)

# Check CPU quota
docker inspect limited-container | grep -A 3 '"CpuQuota"'

# Expected output:
# "CpuPeriod": 100000,
# "CpuQuota": 50000,
# "CpuShares": 1024,

# View real-time resource usage
docker stats limited-container --no-stream

# Expected output:
# CONTAINER ID   NAME                 CPU %     MEM USAGE / LIMIT
# b1c2d3e4f5a6   limited-container    0.05%     6.3 MiB / 256 MiB

# Create a container WITHOUT limits for comparison
docker run -d --name unlimited-container nginx:latest

# Compare resource usage
docker stats --no-stream

# Expected output shows:
# limited-container:     CPU %=0.05%,  MEM USAGE=6.3 MiB / 256 MiB
# unlimited-container:   CPU %=0.05%,  MEM USAGE=6.2 MiB / 7.727 GiB
```

### Explanation

**Memory Limits:**
- `--memory=256m`: Hard limit of 256MB
- If container exceeds limit, kernel kills the process (OOM)
- Can also use: `256m`, `1g`, etc.

**CPU Limits:**
- `--cpus=0.5`: Container can use max 0.5 CPU cores
- `--cpus=2`: Container can use max 2 CPU cores
- Value can be fractional (0.1, 0.5, 1.5, etc.)

**Why Limits Matter:**
- Prevent resource monopolization by single container
- Protect other containers from starvation
- Implement fair resource allocation
- Enable predictable performance

---

## Exercise 3: Set restart policy and kill container to test

### Solution

### Commands

```bash
# Run container with 'unless-stopped' restart policy
docker run -d \
  --name auto-restart-container \
  --restart=unless-stopped \
  nginx:latest

# Expected output:
# c1d2e3f4a5b6

# Verify the restart policy is set
docker inspect auto-restart-container | grep -A 5 '"RestartPolicy"'

# Expected output:
# "RestartPolicy": {
#   "Name": "unless-stopped",
#   "MaximumRetryCount": 0
# },

# Kill the container (simulate crash)
docker kill auto-restart-container

# Expected output:
# auto-restart-container

# Check that it was killed
docker ps -a | grep auto-restart-container

# Expected output:
# c1d2e3f4a5b6   ...   Exited (137)   2 seconds ago   auto-restart-container

# Wait a moment and check again - container should restart
sleep 3
docker ps | grep auto-restart-container

# Expected output:
# c1d2e3f4a5b6   ...   Up 1 second   auto-restart-container

# View restart count
docker inspect auto-restart-container | grep -i restart

# Expected output shows RestartPolicy with restart

# Test different restart policies

# Policy: 'always' - restart even if exited cleanly
docker run -d \
  --name always-restart \
  --restart=always \
  nginx

# Policy: 'on-failure' - restart only on failure (non-zero exit code)
docker run -d \
  --name on-failure-restart \
  --restart=on-failure:5 \
  nginx

# Policy: 'no' - do not restart
docker run -d \
  --name no-restart \
  --restart=no \
  nginx

# Compare restart policies
docker inspect always-restart | grep -A 5 '"RestartPolicy"'
docker inspect on-failure-restart | grep -A 5 '"RestartPolicy"'
docker inspect no-restart | grep -A 5 '"RestartPolicy"'

# Expected outputs:
# always-restart: Name="always"
# on-failure-restart: Name="on-failure", MaximumRetryCount=5
# no-restart: Name="no"
```

### Explanation

**Restart Policies:**

| Policy | Behavior | Use Case |
|--------|----------|----------|
| `no` | Do not restart | One-time jobs |
| `always` | Always restart | Critical services |
| `unless-stopped` | Restart unless explicitly stopped | Production apps |
| `on-failure:5` | Restart only on failure, max 5 times | Apps with expected failures |

**Exit Codes:**
- `0`: Successful exit (clean shutdown)
- `1-255`: Error/failure exit
- `137`: SIGKILL received (killed)
- `143`: SIGTERM received (terminated)

---

## Exercise 4: Monitor container health status with docker inspect

### Solution

### Commands

```bash
# Run a container with health check
docker run -d \
  --name health-monitor \
  --health-cmd='curl -f http://localhost/ || exit 1' \
  --health-interval=5s \
  --health-timeout=2s \
  --health-retries=3 \
  nginx:latest

# Expected output:
# d1e2f3a4b5c6

# Get basic health status
docker inspect --format='{{.State.Health.Status}}' health-monitor

# Expected output:
# starting  (or healthy after a few seconds)

# Get full health information
docker inspect health-monitor | grep -A 20 '"Health"'

# Expected output:
# "Health": {
#   "Status": "healthy",
#   "FailingStreak": 0,
#   "Passes": 2,
#   "Fails": 0,
#   "Log": [
#     {
#       "Start": "2024-01-15T12:34:56.789Z",
#       "End": "2024-01-15T12:34:56.895Z",
#       "ExitCode": 0,
#       "Output": "..."
#     }
#   ]
# }

# Extract specific health fields
docker inspect --format='Status: {{.State.Health.Status}}, Passes: {{.State.Health.Passes}}, Fails: {{.State.Health.Fails}}' health-monitor

# Expected output:
# Status: healthy, Passes: 2, Fails: 0

# Monitor health check logs
docker inspect health-monitor | jq '.[] | .State.Health.Log'

# Expected output:
# [
#   {
#     "Start": "2024-01-15T12:34:56.789Z",
#     "End": "2024-01-15T12:34:56.895Z",
#     "ExitCode": 0,
#     "Output": "..."
#   }
# ]

# Simulate unhealthy state by stopping nginx
docker exec health-monitor bash -c "service nginx stop || exit 1"

# Wait for health checks to fail
sleep 15

# Check status - should be unhealthy
docker inspect --format='{{.State.Health.Status}}' health-monitor

# Expected output:
# unhealthy

# View the failed health check logs
docker inspect health-monitor | jq '.[] | .State.Health.Log[-3:]'

# Expected output shows last 3 health checks with non-zero exit codes
```

### Explanation

**Health Check Information:**
- `Status`: Current health state (starting, healthy, unhealthy)
- `FailingStreak`: Count of consecutive failed checks
- `Passes`: Total successful checks
- `Fails`: Total failed checks
- `Log`: Array of last 10 health check results

**Log Entry Fields:**
- `Start`: When health check started
- `End`: When health check completed
- `ExitCode`: 0 = success, non-zero = failure
- `Output`: Command output (usually hidden for success)

---

## Exercise 5: Pause and unpause a running container

### Solution

### Commands

```bash
# Run a container
docker run -d \
  --name pause-demo \
  nginx:latest

# Expected output:
# e1f2a3b4c5d6

# Verify container is running
docker ps | grep pause-demo

# Expected output shows container is running

# Pause the container (suspend all processes)
docker pause pause-demo

# Expected output:
# pause-demo

# Check container status
docker ps | grep pause-demo

# Expected output:
# e1f2a3b4c5d6   ...   Up ...   (Paused)

# Try to access the paused container
curl http://localhost  # May timeout if port mapped

# Wait a bit
sleep 5

# Unpause the container (resume all processes)
docker unpause pause-demo

# Expected output:
# pause-demo

# Verify container is running again
docker ps | grep pause-demo

# Expected output:
# e1f2a3b4c5d6   ...   Up ...   (no Paused indicator)

# Container should respond again
curl http://localhost
```

### Explanation

**Pause vs Stop:**

| Operation | State | Processes | Purpose |
|-----------|-------|-----------|---------|
| `docker pause` | Running (Paused) | Frozen | Temporary suspension |
| `docker stop` | Stopped | Exited | Graceful shutdown |
| `docker kill` | Stopped | Killed | Forced termination |

**Use Cases for Pause:**
- Temporarily freeze container without stopping
- Save current state for later inspection
- Maintenance without restarting
- Resource management (prevent CPU usage)

---

## Exercise 6: Create graceful shutdown handler in Python app

### Solution

**app.py:**
```python
#!/usr/bin/env python3
import signal
import time
import sys

class Application:
    def __init__(self):
        self.running = True
        # Register signal handlers
        signal.signal(signal.SIGTERM, self.handle_shutdown)
        signal.signal(signal.SIGINT, self.handle_shutdown)
    
    def handle_shutdown(self, signum, frame):
        print(f"\n[INFO] Received signal {signum}, initiating graceful shutdown...")
        self.running = False
        print("[INFO] Cleaning up resources...")
        time.sleep(1)  # Simulate cleanup
        print("[INFO] Shutdown complete")
        sys.exit(0)
    
    def run(self):
        print("[INFO] Application started, PID:", os.getpid())
        counter = 0
        while self.running:
            counter += 1
            print(f"[INFO] Running... {counter}")
            try:
                time.sleep(2)
            except KeyboardInterrupt:
                break
        print("[INFO] Exiting")

if __name__ == "__main__":
    import os
    app = Application()
    app.run()
```

**Dockerfile:**
```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY app.py /app/

# Make executable
RUN chmod +x /app/app.py

# Handle SIGTERM for graceful shutdown
CMD ["python3", "/app/app.py"]
```

### Commands

```bash
# Build the image
docker build -t graceful-app:1.0 .

# Expected output:
# Successfully tagged graceful-app:1.0

# Run the container
docker run -d --name graceful-demo graceful-app:1.0

# Expected output:
# f1a2b3c4d5e6

# Check logs
docker logs graceful-demo

# Expected output:
# [INFO] Application started, PID: 1
# [INFO] Running... 1
# [INFO] Running... 2
# ...

# Stop the container gracefully (sends SIGTERM)
docker stop graceful-demo

# Expected output:
# graceful-demo

# Check logs for graceful shutdown
docker logs graceful-demo

# Expected output includes:
# [INFO] Received signal 15, initiating graceful shutdown...
# [INFO] Cleaning up resources...
# [INFO] Shutdown complete
```

### Explanation

**Signal Handling:**
- `SIGTERM (15)`: Graceful termination request (default for `docker stop`)
- `SIGKILL (9)`: Force termination (used by `docker kill`)
- `SIGINT (2)`: Interrupt (Ctrl+C)

**Best Practices:**
1. Always handle SIGTERM for graceful shutdown
2. Close connections, flush buffers, cleanup resources
3. Set reasonable timeout for cleanup (default 10 seconds)
4. Use `docker stop` instead of `docker kill` when possible

---

## Exercise 7: Monitor multiple containers with docker stats

### Solution

### Commands

```bash
# Run multiple containers
docker run -d --name app1 nginx:latest
docker run -d --name app2 nginx:latest
docker run -d --name app3 --memory=128m nginx:latest

# Expected output:
# Container IDs for each

# Get real-time stats for all containers
docker stats

# Expected output:
# CONTAINER ID   NAME    CPU %   MEM USAGE / LIMIT     NET I/O        BLOCK I/O   PIDS
# a1b2c3d4e5f6   app1    0.05%   6.3 MiB / 7.727 GiB   656 B / 0 B    0 B / 0 B   1
# b2c3d4e5f6a7   app2    0.04%   6.2 MiB / 7.727 GiB   656 B / 0 B    0 B / 0 B   1
# c3d4e5f6a7b8   app3    0.03%   5.9 MiB / 128 MiB    656 B / 0 B    0 B / 0 B   1

# Get one-shot stats (no continuous update)
docker stats --no-stream

# Expected output:
# Same as above, but doesn't update continuously

# Get stats for specific containers only
docker stats app1 app3 --no-stream

# Expected output:
# Only shows app1 and app3

# Save stats to file for analysis
docker stats --no-stream > container_stats.txt

# View the saved stats
cat container_stats.txt
```

### Explanation

**docker stats Columns:**
| Column | Meaning |
|--------|---------|
| CONTAINER ID | Container identifier |
| NAME | Container name |
| CPU % | CPU usage percentage |
| MEM USAGE / LIMIT | Memory used vs max limit |
| NET I/O | Network bytes in/out |
| BLOCK I/O | Disk bytes read/written |
| PIDS | Number of processes |

**Monitoring Patterns:**
1. Watch real-time usage: `docker stats`
2. One-time snapshot: `docker stats --no-stream`
3. Specific containers: `docker stats app1 app2`
4. Combined with other tools: `docker stats > stats.log &`

---

## Exercise 8: Configure on-failure restart policy with max retry count

### Solution

### Commands

```bash
# Create a script that fails
cat > fail.sh << 'EOF'
#!/bin/bash
echo "Attempt starting..."
exit 1
EOF

chmod +x fail.sh

# Create Dockerfile
cat > Dockerfile << 'EOF'
FROM alpine:latest
COPY fail.sh /app/fail.sh
CMD ["/app/fail.sh"]
EOF

# Build the image
docker build -t failing-app:1.0 .

# Expected output:
# Successfully tagged failing-app:1.0

# Run with on-failure policy, max 3 retries
docker run -d \
  --name retry-demo \
  --restart=on-failure:3 \
  failing-app:1.0

# Expected output:
# g1h2i3j4k5l6

# Watch it restart multiple times
docker ps -a | grep retry-demo

# Check restart count in inspect output
docker inspect retry-demo | grep -A 10 '"RestartPolicy"'

# Expected output:
# "RestartPolicy": {
#   "Name": "on-failure",
#   "MaximumRetryCount": 3
# }

# View logs showing multiple restarts
docker logs retry-demo

# Expected output:
# Attempt starting...
# Attempt starting...
# Attempt starting...
# Attempt starting...

# After 3 failures (4 attempts total), check if it stopped
docker ps | grep retry-demo

# Expected output: (empty, container should be stopped)

# View full history
docker ps -a | grep retry-demo

# Expected output:
# g1h2i3j4k5l6   ...   Exited (1)   5 seconds ago   retry-demo
```

### Explanation

**on-failure Policy:**
- Restarts container only if it exits with non-zero exit code
- `on-failure`: Restart indefinitely on failure
- `on-failure:N`: Restart max N times on failure
- Total attempts = MaximumRetryCount + 1

**Exit Code 0 vs Non-zero:**
- Exit code 0 = success → `on-failure` won't restart
- Exit code 1+ = failure → `on-failure` will restart

---

## Exercise 9: Track container events with docker events

### Solution

### Commands

```bash
# Terminal 1: Watch all events
docker events

# Expected output (streams events):
# 2024-01-15T12:00:00.000000000Z container create a1b2c3d4e5f6
# 2024-01-15T12:00:00.100000000Z container attach a1b2c3d4e5f6
# 2024-01-15T12:00:00.200000000Z container start a1b2c3d4e5f6

# Terminal 2: Create and manage containers to see events
docker run -d --name event-test nginx

# Terminal 1: Should show:
# container create event-test
# container start event-test
# ... (health check events if configured)

# Back in Terminal 2:
docker pause event-test

# Terminal 1: Should show:
# container pause event-test

# Stop the container
docker stop event-test

# Terminal 1: Should show:
# container unpause event-test
# container stop event-test

# Remove the container
docker rm event-test

# Terminal 1: Should show:
# container destroy event-test

# Filter events by type
docker events --filter type=container

# Expected output:
# (only container events, no image/network events)

# Filter by specific container
docker events --filter container=event-test &

# Run the container in another terminal
docker run -d --name event-test nginx

# Expected output:
# (events only for event-test container)

# Filter by event type
docker events --filter event=start

# Expected output:
# (only start events)

# Save events to file for analysis
docker events --filter type=container > events.log &
EVENT_PID=$!

# Perform some container operations
docker run -d --name event1 nginx
docker stop event1
docker rm event1

# Stop the event monitoring
kill $EVENT_PID

# View the captured events
cat events.log
```

### Explanation

**docker events Output Format:**
- Timestamp: When the event occurred
- Type: container, image, network, volume, etc.
- Action: create, start, stop, destroy, etc.
- Container ID
- Additional attributes

**Event Types & Actions:**

| Type | Common Actions |
|------|----------------|
| container | create, start, stop, pause, unpause, restart, destroy |
| image | build, load, save, pull, push |
| network | create, connect, disconnect, destroy |
| volume | create, mount, unmount, destroy |

**Monitoring Use Cases:**
1. Real-time container activity tracking
2. Audit logging for compliance
3. Automated response to container events
4. Performance analysis and correlation

---

## Exercise 10: Simulate container failure and observe auto-restart

### Solution

### Commands

```bash
# Create a Dockerfile with an app that randomly fails
cat > app.py << 'EOF'
#!/usr/bin/env python3
import os
import sys
import time
import random

if __name__ == "__main__":
    pid = os.getpid()
    print(f"[INFO] Process started, PID: {pid}")
    
    # Simulate work
    time.sleep(2)
    
    # Randomly fail or succeed
    if random.random() < 0.7:  # 70% chance of failure
        print("[ERROR] Simulating failure...")
        sys.exit(1)
    else:
        print("[INFO] Process completed successfully")
        sys.exit(0)
EOF

cat > Dockerfile << 'EOF'
FROM python:3.11-slim
COPY app.py /app/app.py
CMD ["python3", "/app/app.py"]
EOF

# Build the image
docker build -t flaky-app:1.0 .

# Expected output:
# Successfully tagged flaky-app:1.0

# Run with always restart policy
docker run -d \
  --name auto-restart-demo \
  --restart=always \
  flaky-app:1.0

# Expected output:
# h1i2j3k4l5m6

# Watch it restart multiple times
watch -n 1 'docker inspect h1i2j3k4l5m6 | grep -A 2 "\"Status\":'

# Or check manually multiple times
for i in {1..10}; do
  echo "Check $i:"
  docker ps | grep auto-restart-demo | awk '{print $10, $11, $12, $13}'
  docker logs --tail 1 auto-restart-demo
  sleep 3
done

# Expected output shows:
# Alternating between:
# - "Up X seconds" (running)
# - Restart message in logs

# View full logs showing multiple restarts
docker logs auto-restart-demo

# Expected output:
# [INFO] Process started, PID: 1
# [ERROR] Simulating failure...
# [INFO] Process started, PID: 1
# [INFO] Process completed successfully
# [INFO] Process started, PID: 1
# [ERROR] Simulating failure...
# ... (repeats as container keeps restarting)

# Check how many times it has restarted
docker inspect auto-restart-demo | grep -i restartcount

# Expected output:
# "RestartCount": 5,  (varies based on runtime)

# View restart policy
docker inspect auto-restart-demo | grep -A 3 '"RestartPolicy"'

# Expected output:
# "RestartPolicy": {
#   "Name": "always",
#   "MaximumRetryCount": 0
# }

# Track restart events
docker events --filter container=auto-restart-demo --filter event=start &
EVENT_PID=$!

# Wait and observe
sleep 15

# Stop the event monitoring
kill $EVENT_PID 2>/dev/null

# Clean up
docker stop auto-restart-demo
docker rm auto-restart-demo
rm app.py Dockerfile
```

### Explanation

**Restart Behavior:**
- With `--restart=always`, container restarts immediately on failure
- Each restart increments `RestartCount`
- Useful for critical services that must be available
- Can add backoff delay with other orchestration tools

**Monitoring Restarts:**
1. Check `RestartCount` via `docker inspect`
2. Monitor logs for error patterns
3. Watch `docker events` for restart events
4. Use `docker stats` to see resource patterns

**Best Practices:**
- Don't rely solely on restart policies
- Implement proper health checks
- Log failures for analysis
- Set up alerting for repeated failures
- Fix root cause instead of relying on auto-restart


