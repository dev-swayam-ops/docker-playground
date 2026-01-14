# Solutions: Docker Basics

Detailed solutions with explanations for all exercises.

---

## Solution 1: Run Your First Container

### Commands

```bash
# Run the container with a simple echo command
docker run ubuntu:22.04 echo "Hello from Docker!"

# Expected output:
# Hello from Docker!

# List all containers to find it
docker ps -a

# Expected output:
# CONTAINER ID   IMAGE          COMMAND                  CREATED        STATUS
# a1b2c3d4e5f6   ubuntu:22.04   "echo 'Hello from Do…"   2 minutes ago  Exited (0) 1 minute ago
```

### Explanation

**What Happened:**
1. Docker pulled the ubuntu:22.04 image (if not already present)
2. Created a new container from the image
3. Ran the `echo` command inside the container
4. Container exited when the command finished
5. The container still exists (not removed) but is in "Exited" state

**Key Points:**
- `docker run` always creates a **new** container
- The container exits when the command completes
- Exited containers don't disappear; they're just stopped
- Use `docker ps -a` to see exited containers
- The command output appears immediately in your terminal

---

## Solution 2: Interactive Container Session

### Commands

```bash
# Run Ubuntu in interactive mode with a bash shell
docker run -it ubuntu:22.04 bash

# Inside the container, you'll see a prompt like:
# root@a1b2c3d4e5f6:/#

# Execute these commands:
ls /home
# Output: (empty directory listing)

pwd
# Output: /

whoami
# Output: root

cat /etc/os-release
# Output:
# NAME="Ubuntu"
# VERSION="22.04.3 LTS"
# ID=ubuntu
# ID_LIKE=debian
# (additional fields...)

# Exit the container
exit

# Verify it stopped
docker ps -a
# CONTAINER ID   IMAGE          COMMAND   CREATED   STATUS
# a1b2c3d4e5f6   ubuntu:22.04   "bash"    5 min ago Exited (0) 2 minutes ago
```

### Explanation

**The `-it` Flags:**
- `-i` or `--interactive`: Keep STDIN open even if not attached (allows input)
- `-t` or `--tty`: Allocate a pseudo-terminal (shows output like a real terminal)
- Together, `-it` creates an interactive shell environment

**Container vs Host:**
- You're literally **inside** the container
- `root@a1b2c3d4e5f6` - the hostname is the container ID
- `/` is the container's filesystem, not your host's
- When you exit, the container stops

**Understanding States:**
- While interactive, the container is "Running"
- When you `exit`, the container transitions to "Exited"
- Exit code (0) means successful termination

---

## Solution 3: Name Your Containers

### Commands

```bash
# Run interactive container with a name
docker run -it --name my-web-app ubuntu:22.04 bash

# Inside the container, create a version file
echo "version 1.0" > /version.txt

# Verify the file was created
cat /version.txt
# Output: version 1.0

# Exit the container
exit

# List all containers
docker ps -a

# Expected output includes:
# CONTAINER ID   IMAGE          COMMAND   CREATED        STATUS   NAMES
# a1b2c3d4e5f6   ubuntu:22.04   "bash"    3 minutes ago  Exited   my-web-app
```

### Explanation

**Why Names Matter:**
| Aspect | Container ID | Container Name |
|--------|--------------|----------------|
| Length | 64 characters (truncated to 12) | 1-30 characters |
| Readability | `a1b2c3d4e5f6` | `my-web-app` |
| Memorability | Hard to remember | Easy to remember |
| Command Usage | Works with most commands | Works with most commands |

**Naming Rules:**
- Must be unique (can't have two containers with same name)
- Can contain lowercase letters, digits, hyphens, underscores
- Cannot start with hyphens
- Case-insensitive

**Benefits:**
- Easier to reference in commands
- Better for scripts and automation
- More meaningful in logs and monitoring
- Improves team collaboration (meaningful names describe purpose)

---

## Solution 4: Container Lifecycle Management

### Commands

```bash
# Verify the container exists and is stopped
docker ps -a | grep my-web-app
# CONTAINER ID   IMAGE          COMMAND   STATUS
# a1b2c3d4e5f6   ubuntu:22.04   "bash"    Exited

# Start the stopped container
docker start my-web-app

# Verify it's running
docker ps
# CONTAINER ID   IMAGE          COMMAND   CREATED        STATUS        NAMES
# a1b2c3d4e5f6   ubuntu:22.04   "bash"    10 minutes ago Up 3 seconds  my-web-app

# Stop the running container
docker stop my-web-app

# Verify it's stopped
docker ps -a
# CONTAINER ID   IMAGE          COMMAND   CREATED        STATUS                NAMES
# a1b2c3d4e5f6   ubuntu:22.04   "bash"    10 minutes ago Exited (137) 5s ago   my-web-app

# Restart the container
docker restart my-web-app

# Verify it's running again
docker ps
# Same as before - container is running
```

### Container States Explained

```
┌─────────────┐
│   Created   │  Container created but not started
└──────┬──────┘
       │ docker start
       ▼
┌─────────────┐
│   Running   │  Container is actively executing
└──────┬──────┘
   │       │
   │       └─── docker pause ──┐
   │                           ▼
   │                    ┌────────────┐
   │                    │   Paused   │
   │                    └────────────┘
   │
   └─── docker stop ──┐
                      ▼
            ┌─────────────┐
            │   Stopped   │  (Exited status)
            └──────┬──────┘
                   │ docker rm
                   ▼
            ┌─────────────┐
            │   Removed   │  (Gone from system)
            └─────────────┘
```

### Key Difference: start vs run

| Command | Action | When to Use |
|---------|--------|------------|
| `docker run` | Creates **new** container + starts it | First time running image |
| `docker start` | Starts **existing** stopped container | Resume previous container |
| `docker restart` | Stops then starts container | Reboot container |

---

## Solution 5: Execute Commands in Running Containers

### Commands

```bash
# Ensure container is running
docker start my-web-app

# Execute commands in the running container
docker exec my-web-app cat /version.txt
# Output: version 1.0

docker exec my-web-app ls /
# Output:
# bin
# boot
# dev
# etc
# home
# lib
# (many more directories)

docker exec my-web-app whoami
# Output: root
```

### Explanation

**docker run vs docker exec:**

| Aspect | `docker run` | `docker exec` |
|--------|-------------|-------------|
| Container | Creates new | Uses existing |
| Execution | Starts container | Container must be running |
| Use Case | Run image for first time | Run command in running container |
| Cleanup | Container persists | No new container |

**Real-World Example:**

Imagine a running database container:
```bash
# docker run would create a NEW database container
docker run -d mysql:8.0  # Creates NEW container

# docker exec runs command in EXISTING container
docker exec my-database mysql -u root -p dbname < data.sql
```

**Why docker exec is Useful:**
- Don't want to create multiple containers
- Container might have state/data you want to keep
- Quick commands without entering interactive shell
- Scripting and automation

---

## Solution 6: View Container Logs

### Commands

```bash
# Run a container with a sleep command (runs 100 seconds)
docker run -d --name logger alpine sleep 100
# Output: (container ID)

# View all logs
docker logs logger
# Output: (empty if sleep doesn't produce output)

# Add a message to logs
docker exec logger echo "New message"
# Output: New message

# View logs again
docker logs logger
# Output: New message

# Follow logs in real-time (like tail -f)
docker logs -f logger
# Output: (waits for new output, press Ctrl+C to stop)

# View last 5 lines
docker logs --tail 5 logger

# View logs with timestamps
docker logs --timestamps logger

# View logs from last 5 minutes
docker logs --since 5m logger
```

### Important Log Flags

| Flag | Purpose | Example |
|------|---------|---------|
| None | View all logs | `docker logs container_name` |
| `-f` | Follow (stream) logs | `docker logs -f container_name` |
| `--tail` | Last N lines | `docker logs --tail 10 container_name` |
| `--timestamps` | Add timestamps | `docker logs --timestamps container_name` |
| `--since` | Since time period | `docker logs --since 5m container_name` |

### Explanation

**What Logs Capture:**
- Anything the container writes to stdout (standard output)
- Anything the container writes to stderr (standard error)
- Even after the container exits, logs persist
- Logs are stored by Docker, not in the container

**Logs Are Persistent:**
```bash
# Container exits
docker stop logger

# But logs still exist
docker logs logger  # Still shows the message!
```

---

## Solution 7: Inspect Container Details

### Commands

```bash
# Run a container
docker run -d --name inspector ubuntu:22.04 sleep 300

# Get full JSON details
docker inspect inspector
# Output: Large JSON object with all container config

# Extract specific information
docker inspect --format='{{.NetworkSettings.IPAddress}}' inspector
# Output: 172.17.0.2

docker inspect --format='{{.State.Status}}' inspector
# Output: running

docker inspect --format='{{.Config.Image}}' inspector
# Output: ubuntu:22.04

# Get multiple values
docker inspect --format='Name: {{.Name}} | Image: {{.Config.Image}} | Status: {{.State.Status}}' inspector
# Output: Name: /inspector | Image: ubuntu:22.04 | Status: running
```

### Common Format Queries

| Query | What It Shows | Example Output |
|-------|--------------|-----------------|
| `{{.Id}}` | Full container ID | `sha256:a1b2c3d4e5f6...` |
| `{{.Name}}` | Container name (with /) | `/inspector` |
| `{{.State.Status}}` | Current status | `running` |
| `{{.State.Running}}` | Boolean running state | `true` |
| `{{.Config.Image}}` | Image used | `ubuntu:22.04` |
| `{{.NetworkSettings.IPAddress}}` | Container IP | `172.17.0.2` |
| `{{.HostConfig.Memory}}` | Memory limit in bytes | `0` (unlimited) |
| `{{.Created}}` | Creation timestamp | `2024-01-15T10:30:00...` |

### Explanation

**JSON Output:**
The full `docker inspect` output looks like:
```json
[
    {
        "Id": "sha256:a1b2c3d4e5f6...",
        "Created": "2024-01-15T10:30:00.000Z",
        "Name": "/inspector",
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "ExitCode": 0
        },
        "Config": {
            "Image": "ubuntu:22.04",
            "Cmd": ["sleep", "300"]
        },
        "NetworkSettings": {
            "IPAddress": "172.17.0.2"
        }
    }
]
```

**Why Inspect is Useful:**
- Debugging container configuration
- Verifying environment variables
- Checking IP addresses and ports
- Monitoring resource limits
- Auditing container setup

---

## Solution 8: Monitor Container Resources

### Commands

```bash
# Run containers with different workloads
docker run -d --name cpu-heavy alpine yes > /dev/null
# yes command continuously outputs 'y', using CPU

docker run -d --name sleep-app alpine sleep 300
# Just sleeps, minimal resource usage

# View live statistics
docker stats
# Output (updates every second):
# CONTAINER ID   NAME        CPU %     MEM USAGE / LIMIT     NET I/O       BLOCK I/O
# a1b2c3d4e5f6   cpu-heavy   95.2%     512 KiB / 7.727 GiB   1.2 kB / 0B   0B / 0B
# b2c3d4e5f6a1   sleep-app   0.0%      2.34 MiB / 7.727 GiB  0B / 0B       0B / 0B

# View specific containers
docker stats cpu-heavy sleep-app

# No-stream (single snapshot)
docker stats --no-stream

# Stop the containers
docker stop cpu-heavy sleep-app

# View stats again (stopped containers won't show)
docker stats
```

### Understanding the Output

| Column | Meaning | Example |
|--------|---------|---------|
| CPU % | Percentage of host CPU used | 25.5% |
| MEM USAGE | Memory currently used | 512 KiB |
| MEM LIMIT | Maximum memory allowed | 7.727 GiB |
| NET I/O | Network bytes in/out | 1.2 kB / 0B |
| BLOCK I/O | Disk bytes read/written | 0B / 0B |

### Key Observations

1. **Only Running Containers:**
   - `docker stats` only shows running containers
   - Stopped containers don't appear
   - Exited containers don't appear

2. **Real-Time Updates:**
   - Default behavior is to stream (continuously update)
   - Press Ctrl+C to stop viewing
   - Use `--no-stream` for single snapshot

3. **CPU Calculation:**
   - Percentage based on **all available CPU cores**
   - High CPU % doesn't mean the container is broken
   - `yes` command intentionally uses CPU

---

## Solution 9: Clean Up Containers

### Commands

```bash
# List all containers to see what we're cleaning
docker ps -a
# Output: All containers on system

# Count containers
docker ps -a -q | wc -l
# Output: Number (e.g., 15)

# Stop all running containers
docker stop $(docker ps -q)
# Output: Lists IDs of stopped containers

# Verify all are stopped
docker ps
# Output: (empty - no running containers)

# Remove all containers
docker rm $(docker ps -a -q)
# Output: Lists IDs of removed containers

# Verify no containers exist
docker ps -a
# Output: (empty - no containers at all)
```

### Understanding the Commands

**Breaking Down `docker stop $(docker ps -q)`:**

```
docker ps -q
  ↓
Returns list of all running container IDs:
  a1b2c3d4e5f6
  b2c3d4e5f6a1
  c3d4e5f6a1b2
  ↓
$(...) - Command substitution - passes to next command
  ↓
docker stop a1b2c3d4e5f6 b2c3d4e5f6a1 c3d4e5f6a1b2
  ↓
Stops all three containers
```

**Breaking Down `docker rm $(docker ps -a -q)`:**
Same logic but:
- `docker ps -a -q` gets ALL containers (running + stopped)
- `docker rm` removes them

### Why Clean Up Matters

| Issue | Impact | Solution |
|-------|--------|----------|
| Disk Space | Old containers consume space | `docker rm` |
| Memory | Each container reserves memory | Stop unused ones |
| Port Conflicts | Containers occupy ports | Remove unused ones |
| Clutter | Hard to find containers | Clean regularly |

### Safe Cleanup Strategy

```bash
# Step 1: List what you're about to delete
docker ps -a

# Step 2: Stop running containers
docker stop <container_id>

# Step 3: Verify what will be removed
docker ps -a

# Step 4: Remove containers one at a time (safer)
docker rm <container_id>

# OR: Remove all at once (after verifying step 3)
docker rm $(docker ps -a -q)

# Verify cleanup
docker ps -a  # Should be empty
```

---

## Solution 10: Container Port Mapping

### Commands

```bash
# Run nginx with port mapping
docker run -d --name web-server -p 8080:80 nginx

# Expected output: (container ID)

# Verify it's running
docker ps
# CONTAINER ID   IMAGE    COMMAND                  PORTS          NAMES
# a1b2c3d4e5f6   nginx    "nginx -g 'daemon of…" 0.0.0.0:8080->80/tcp   web-server

# Check port mapping specifically
docker port web-server
# Output:
# 80/tcp -> 0.0.0.0:8080

# View nginx logs to confirm startup
docker logs web-server
# Output:
# /docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform any .sh files in /docker-entrypoint.d/
# /docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
# /docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
# 10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
# 10-listen-on-ipv6-by-default.sh: info: /etc/nginx/conf.d/default.conf differs from the packaged version
# (startup messages continue...)
# 2024/01/15 10:30:00 [notice] 1#1: nginx/1.24.0
# 2024/01/15 10:30:00 [notice] 1#1: built by gcc 11.2.0
# 2024/01/15 10:30:00 [notice] 1#1: OS: Linux 5.15.0-91-generic #101-Ubuntu SMP
# 2024/01/15 10:30:00 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
# 2024/01/15 10:30:00 [notice] 1#1: start worker processes

# Stop the container
docker stop web-server

# Remove it
docker rm web-server
```

### Port Mapping Explained

**Format: `-p host_port:container_port`**

```
Host Machine (Your Computer)
┌──────────────────────────────────┐
│  Port 8080                        │
│  ↓                               │
│  Forwards to Container Port 80    │
│                                  │
│  ┌──────────────────────────────┐│
│  │  Container (web-server)       ││
│  │  ┌────────────────────────┐  ││
│  │  │ nginx listening on     │  ││
│  │  │ port 80                │  ││
│  │  └────────────────────────┘  ││
│  └──────────────────────────────┘│
└──────────────────────────────────┘
```

**Example: -p 8080:80**
- Host Port: 8080 (your computer)
- Container Port: 80 (inside container)
- When you visit `http://localhost:8080`, traffic goes to port 80 inside the container

**Multiple Port Mappings:**
```bash
# Map multiple ports
docker run -d -p 8080:80 -p 8443:443 -p 3000:3000 myapp

# Maps:
# Host:8080 -> Container:80
# Host:8443 -> Container:443
# Host:3000 -> Container:3000
```

**Why Port Mapping Matters:**
- Container ports are isolated (only accessible inside container)
- Port mapping exposes container services to the host
- Multiple containers can run on different ports
- You can't have two containers listening on same host port

---

## Summary Table: Commands Learned

| Command | Purpose | Example |
|---------|---------|---------|
| `docker run` | Create and run container | `docker run -it ubuntu bash` |
| `docker start` | Start stopped container | `docker start myapp` |
| `docker stop` | Stop running container | `docker stop myapp` |
| `docker restart` | Restart container | `docker restart myapp` |
| `docker exec` | Run command in running container | `docker exec myapp ls /` |
| `docker logs` | View container output | `docker logs myapp` |
| `docker inspect` | Get container details | `docker inspect myapp` |
| `docker stats` | View resource usage | `docker stats` |
| `docker ps` | List running containers | `docker ps` |
| `docker ps -a` | List all containers | `docker ps -a` |
| `docker rm` | Remove container | `docker rm myapp` |
| `docker port` | View port mappings | `docker port myapp` |

---

**Next Steps:**
1. Review concepts you found challenging
2. Re-do any exercises that felt difficult
3. Move on to [quiz.md](quiz.md) to test your knowledge
4. Progress to Module 02: Images and Dockerfile

