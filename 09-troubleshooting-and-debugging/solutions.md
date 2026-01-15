# Solutions: Troubleshooting & Debugging

Comprehensive solutions for all 10 exercises with detailed commands and explanations.

---

## Exercise 1: View container logs with tail

### Solution

### Commands

```bash
# Create a test container that outputs text every second
docker run -d --name log-test alpine sh -c 'for i in {1..100}; do echo "Log message $i"; sleep 1; done'

# Expected output:
# Container ID (running in background)

# Verify container is running
docker ps | grep log-test

# Expected output:
# CONTAINER ID    IMAGE    COMMAND              STATUS              PORTS    NAMES
# abc123def456    alpine   sh -c 'for i in...'  Up 5 seconds               log-test

# View last 5 lines of logs
docker logs --tail 5 log-test

# Expected output:
# Log message 96
# Log message 97
# Log message 98
# Log message 99
# Log message 100

# View last 10 lines
docker logs --tail 10 log-test

# Expected output:
# Shows last 10 messages

# View last line only
docker logs --tail 1 log-test

# Expected output:
# Log message 100

# Tail with specific number
docker logs --tail 20 log-test

# Expected output:
# Shows last 20 messages

# Combine with grep to find specific errors
docker logs --tail 100 log-test | grep "error"

# Expected output:
# (no output if no errors)

# View all logs (could be many)
docker logs log-test

# Expected output:
# All 100 messages displayed

# Wait for container to finish
docker wait log-test

# Expected output:
# Exit code (0 for success)

# Check final exit status
docker inspect --format='{{.State.ExitCode}}' log-test

# Expected output:
# 0 (success)

# View logs after container stops
docker logs --tail 5 log-test

# Expected output:
# Last 5 messages still visible

# Using tail to monitor completion
while [ $(docker inspect --format='{{.State.Running}}' log-test) = "true" ]; do
  echo "Container still running..."
  sleep 10
done

# Expected output:
# Waits for container to finish

# Check total lines of output
docker logs log-test | wc -l

# Expected output:
# 100 (number of lines)
```

### Explanation

**Log Viewing Fundamentals:**
- `docker logs [container]`: Shows all logs from start to current
- `--tail [n]`: Show only last n lines (useful for large logs)
- Logs are stored even after container stops
- Can access logs from exited containers
- Useful for quick error diagnostics

**Common Use Cases:**
1. Quick error checks without following live logs
2. Searching for specific errors in history
3. Verifying container completed successfully
4. Performance analysis from historical logs

---

## Exercise 2: Follow logs in real-time

### Solution

### Commands

```bash
# Start a container that logs continuously
docker run -d --name live-logs alpine sh -c 'while true; do echo "$(date): Service running"; sleep 2; done'

# Expected output:
# Container ID

# Verify container is running
docker ps | grep live-logs

# Expected output:
# Shows running container

# Follow logs in real-time
docker logs -f live-logs

# Expected output:
# 2024-01-15T12:00:00Z: Service running
# 2024-01-15T12:00:02Z: Service running
# 2024-01-15T12:00:04Z: Service running
# (continues until Ctrl+C pressed)

# To stop following logs (in terminal)
# Press: Ctrl+C

# Follow with timestamps
docker logs -f --timestamps live-logs

# Expected output:
# Shows logs with RFC3339 timestamps
# 2024-01-15T12:00:00.123456Z (actual ISO8601 format)

# Follow last 10 lines then continue
docker logs -f --tail 10 live-logs

# Expected output:
# Shows last 10 lines, then streams new logs

# Follow with grep filter (in separate terminal)
docker logs -f live-logs | grep "error"

# Expected output:
# Shows only lines containing "error"

# Follow logs from multiple containers
docker logs -f live-logs container2 container3

# Expected output:
# Interleaves logs from all containers

# Start another terminal and run a container
# (In another terminal window)
docker run -d --name app-server nginx

# In first terminal, follow its logs
docker logs -f app-server

# Expected output:
# Shows nginx startup logs
# Shows access logs as requests arrive

# Stop following from script
timeout 30 docker logs -f live-logs

# Expected output:
# Follows logs for 30 seconds then stops

# Redirect logs to file while following
docker logs -f live-logs > container-logs.txt 2>&1 &

# Expected output:
# Runs in background, writes to file

# View file being written to
tail -f container-logs.txt

# Expected output:
# Shows logs being appended

# Follow logs with specific until time
docker logs --since 2024-01-15T12:00:00Z live-logs

# Expected output:
# Shows logs from that time onwards

# Follow with both since and until
docker logs --since 2024-01-15T11:00:00Z --until 2024-01-15T12:00:00Z live-logs

# Expected output:
# Shows logs in time range

# Cleanup
docker kill live-logs
docker rm live-logs

# Expected output:
# Container removed
```

### Explanation

**Real-time Log Monitoring:**
- `docker logs -f`: Follow (stream) logs continuously
- Updates with new output automatically
- Press Ctrl+C to stop following
- Container remains running
- Useful for real-time troubleshooting

**Streaming Use Cases:**
1. Monitor application startup
2. Watch for errors as they occur
3. Verify application health
4. Debug performance issues in real-time
5. Monitor scheduled jobs or cron executions

---

## Exercise 3: Timestamp logs

### Solution

### Commands

```bash
# Create test container with dated output
docker run -d --name timestamp-test alpine sh -c 'echo "Start"; sleep 5; echo "Middle"; sleep 5; echo "End"'

# Expected output:
# Container ID

# View logs without timestamps
docker logs timestamp-test

# Expected output:
# Start
# Middle
# End

# View logs with timestamps
docker logs --timestamps timestamp-test

# Expected output:
# 2024-01-15T12:00:00.123456Z Start
# 2024-01-15T12:00:05.234567Z Middle
# 2024-01-15T12:00:10.345678Z End

# Timestamps are RFC3339/ISO8601 format

# Create container with application logging
docker run -d --name app-test \
  -e TZ=UTC \
  alpine sh -c 'for i in {1..5}; do echo "[$(date +'%Y-%m-%d %H:%M:%S')] Event $i"; sleep 2; done'

# Expected output:
# Container ID

# View with timestamps (two formats shown)
docker logs --timestamps app-test

# Expected output:
# 2024-01-15T12:00:00.123456Z [2024-01-15 12:00:00] Event 1
# 2024-01-15T12:00:02.234567Z [2024-01-15 12:00:02] Event 2
# etc.

# Combine timestamps with tail
docker logs --timestamps --tail 2 app-test

# Expected output:
# Last 2 lines with timestamps

# Combine timestamps with follow
docker logs --timestamps -f app-test

# Expected output:
# Streams logs with timestamps as they appear

# Parse timestamps for analysis
docker logs --timestamps timestamp-test | awk -F'Z' '{print $1}'

# Expected output:
# Extracts just the timestamp part
# 2024-01-15T12:00:00.123456
# 2024-01-15T12:00:05.234567
# etc.

# Calculate time between events
docker logs --timestamps app-test | awk -F'Z' '{
  if (prev) {
    print "Time since last log: " NR - prev " seconds"
  }
  prev = NR
}'

# Expected output:
# Shows intervals between logs

# Logs with since parameter (using timestamps)
docker logs --timestamps --since 2024-01-15T12:00:02Z timestamp-test

# Expected output:
# Shows logs from that timestamp onwards

# Logs with until parameter
docker logs --timestamps --until 2024-01-15T12:00:08Z timestamp-test

# Expected output:
# Shows logs before that timestamp

# Save timestamped logs to file
docker logs --timestamps app-test > timestamped-logs.txt

# Expected output:
# File contains all logs with timestamps

# View saved timestamped logs
cat timestamped-logs.txt

# Expected output:
# 2024-01-15T12:00:00.123456Z [2024-01-15 12:00:00] Event 1
# 2024-01-15T12:00:02.234567Z [2024-01-15 12:00:02] Event 2
# etc.

# Parse and filter by time range
docker logs --timestamps app-test | awk '
  $1 > "2024-01-15T12:00:01Z" && $1 < "2024-01-15T12:00:05Z"'

# Expected output:
# Shows logs within time window

# Cleanup
docker rm -f timestamp-test app-test

# Expected output:
# Containers removed
```

### Explanation

**Log Timestamps:**
- RFC3339/ISO8601 format with microseconds
- Format: `YYYY-MM-DDTHH:MM:SS.ffffffZ`
- Essential for correlating events
- Helps track latency between operations
- Useful for performance analysis

**Timestamp Uses:**
1. Correlate events across services
2. Identify performance bottlenecks
3. Calculate processing time
4. Audit compliance requirements
5. Time-based filtering

---

## Exercise 4: Inspect container state

### Solution

### Commands

```bash
# Create a running container
docker run -d --name state-test nginx

# Expected output:
# Container ID

# View container state
docker inspect state-test

# Expected output:
# Large JSON with all container details
# Includes State field with Running, Paused, Restarting, etc.

# Get just the State field
docker inspect state-test | jq '.State'

# Expected output:
# {
#   "Status": "running",
#   "Running": true,
#   "Paused": false,
#   "Restarting": false,
#   "OOMKilled": false,
#   "Dead": false,
#   "Pid": 1234,
#   "ExitCode": 0,
#   "Error": "",
#   "StartedAt": "2024-01-15T12:00:00.123456Z",
#   "FinishedAt": "0001-01-01T00:00:00Z"
# }

# Get running status
docker inspect --format='{{.State.Running}}' state-test

# Expected output:
# true

# Get process ID
docker inspect --format='{{.State.Pid}}' state-test

# Expected output:
# 1234 (actual PID inside container)

# Get exit code
docker inspect --format='{{.State.ExitCode}}' state-test

# Expected output:
# 0 (exit code if stopped)

# Get start time
docker inspect --format='{{.State.StartedAt}}' state-test

# Expected output:
# 2024-01-15T12:00:00.123456Z

# Check if paused
docker inspect --format='{{.State.Paused}}' state-test

# Expected output:
# false

# Pause container
docker pause state-test

# Expected output:
# state-test

# Check paused status
docker inspect --format='{{.State.Paused}}' state-test

# Expected output:
# true

# Check Running status while paused
docker inspect --format='{{.State.Running}}' state-test

# Expected output:
# false

# Unpause container
docker unpause state-test

# Expected output:
# state-test

# Verify resumed
docker inspect --format='{{.State.Running}}' state-test

# Expected output:
# true

# Stop container
docker stop state-test

# Expected output:
# state-test

# Check state after stop
docker inspect state-test | jq '.State'

# Expected output:
# {
#   "Status": "exited",
#   "Running": false,
#   "Paused": false,
#   "Restarting": false,
#   "OOMKilled": false,
#   "Dead": false,
#   "Pid": 0,
#   "ExitCode": 0,
#   "Error": "",
#   "StartedAt": "2024-01-15T12:00:00.123456Z",
#   "FinishedAt": "2024-01-15T12:00:30.456789Z"
# }

# Check exit code
docker inspect --format='{{.State.ExitCode}}' state-test

# Expected output:
# 0

# Create container that exits with error
docker run -d --name error-test alpine sh -c 'exit 1'

# Wait for it to exit
sleep 2

# Check error exit code
docker inspect --format='{{.State.ExitCode}}' error-test

# Expected output:
# 1

# Container that runs out of memory
docker run -d --name oom-test --memory 10m ubuntu sh -c 'malloc(1000000000)'

# Check OOMKilled status
sleep 2
docker inspect --format='{{.State.OOMKilled}}' oom-test

# Expected output:
# true (if memory exceeded)

# View complete state JSON
docker inspect state-test > container-state.json

# Expected output:
# Full JSON saved to file

# Pretty print state information
docker inspect --format='
Status: {{.State.Status}}
Running: {{.State.Running}}
Paused: {{.State.Paused}}
PID: {{.State.Pid}}
ExitCode: {{.State.ExitCode}}
Started: {{.State.StartedAt}}
Finished: {{.State.FinishedAt}}' state-test

# Expected output:
# Status: running
# Running: true
# Paused: false
# PID: 1234
# ExitCode: 0
# Started: 2024-01-15T12:00:00.123456Z
# Finished: 0001-01-01T00:00:00Z

# Cleanup
docker rm -f state-test error-test oom-test

# Expected output:
# Containers removed
```

### Explanation

**Container State Fields:**

| Field | Description |
|-------|-------------|
| Status | Current status (created, running, paused, restarting, exited, dead) |
| Running | Boolean - is container running |
| Paused | Boolean - is container paused |
| Restarting | Boolean - is container restarting |
| OOMKilled | Boolean - killed by out-of-memory |
| Dead | Boolean - is container dead |
| Pid | Process ID of main process |
| ExitCode | Exit code from container |
| Error | Error message if any |
| StartedAt | When container started |
| FinishedAt | When container stopped |

**State Inspection Uses:**
1. Diagnose why container stopped
2. Check if restart policy is active
3. Identify out-of-memory issues
4. Verify container is running
5. Get execution timeline

---

## Exercise 5: Check network connectivity

### Solution

### Commands

```bash
# Create a custom bridge network
docker network create test-network

# Expected output:
# Network ID

# Run first container on network
docker run -d --name web --network test-network nginx

# Expected output:
# Container ID

# Run second container on same network
docker run -d --name db --network test-network alpine sleep 3600

# Expected output:
# Container ID

# Verify both containers on network
docker network inspect test-network

# Expected output:
# Shows both containers connected

# Check connectivity from web to db using ping
docker exec web ping -c 4 db

# Expected output:
# PING db (172.18.0.3) 56(84) bytes of data
# 64 bytes from 172.18.0.3: icmp_seq=1 ttl=64 time=0.123 ms
# 64 bytes from 172.18.0.3: icmp_seq=2 ttl=64 time=0.098 ms
# 64 bytes from 172.18.0.3: icmp_seq=3 ttl=64 time=0.105 ms
# 64 bytes from 172.18.0.3: icmp_seq=4 ttl=64 time=0.112 ms
# 
# --- db statistics ---
# 4 packets transmitted, 4 received, 0% packet loss, time 3ms

# Check reverse connectivity
docker exec db ping -c 4 web

# Expected output:
# Similar success message

# DNS name resolution test
docker exec web nslookup db

# Expected output:
# Server: 127.0.0.11
# Address: 127.0.0.11#53
# 
# Non-authoritative answer:
# Name: db
# Address: 172.18.0.3

# Resolve by IP directly
docker exec web ping -c 1 172.18.0.3

# Expected output:
# Successful ping to IP address

# Test port connectivity (curl)
docker run -d --name web2 --network test-network -p 8080:80 nginx

# Expected output:
# Container ID

# Test connectivity on port 80
docker exec db curl -I web2:80

# Expected output:
# HTTP/1.1 200 OK
# Server: nginx/1.25.0
# Date: Mon, 15 Jan 2024 12:00:00 GMT
# ...

# Test with telnet simulation
docker exec db sh -c 'echo "test" | nc -w1 web2 80'

# Expected output:
# Shows connection established (or refused if port blocked)

# Test DNS from alpine (has nslookup)
docker exec db nslookup web2

# Expected output:
# Name: web2
# Address: 172.18.0.4

# Run container NOT on network
docker run -d --name isolated alpine sleep 3600

# Expected output:
# Container ID (on default bridge)

# Try to ping from network to isolated
docker exec web ping -c 1 isolated

# Expected output:
# ping: bad address 'isolated'
# (cannot reach containers not on same network)

# Connect isolated container to network
docker network connect test-network isolated

# Now try ping again
docker exec web ping -c 1 isolated

# Expected output:
# Successfully pings

# Create container on different network
docker network create other-network
docker run -d --name other --network other-network alpine sleep 3600

# Expected output:
# Container ID

# Try to reach from test-network
docker exec web ping -c 1 other

# Expected output:
# ping: bad address 'other'
# (different networks are isolated)

# Check network status
docker network ls

# Expected output:
# Shows all networks

# View all connections for web container
docker inspect --format='{{json .NetworkSettings.Networks}}' web | jq

# Expected output:
# Shows all networks container is connected to

# Cleanup
docker rm -f web db web2 isolated other
docker network rm test-network other-network

# Expected output:
# All removed
```

### Explanation

**Docker Network Connectivity:**
- Containers on same network can communicate by container name
- DNS resolution happens automatically via embedded DNS (127.0.0.11:53)
- Each container has unique IP on its network
- Default bridge doesn't support DNS names (use custom networks)
- Multiple networks can be connected to one container

**Testing Tools:**
- `ping`: ICMP echo test
- `curl`: HTTP request test
- `nslookup`/`dig`: DNS resolution
- `nc`: Generic TCP connection test
- `telnet`: Port connectivity test

---

## Exercise 6: Monitor resource usage

### Solution

### Commands

```bash
# Run test containers
docker run -d --name cpu-test ubuntu sh -c 'while true; do false; done'
docker run -d --name memory-test --memory 512m ubuntu sh -c 'while true; do x=$(dd if=/dev/zero bs=1M count=500); done' 2>/dev/null &

# Expected output:
# Container IDs

# View resource stats
docker stats

# Expected output:
# CONTAINER ID   NAME            CPU %   MEM USAGE / LIMIT   MEM %   NET I/O    BLOCK I/O
# abc123def456   cpu-test        95.45%  8.2MiB / 7.8GiB     0.10%   0B / 0B    0B / 0B
# def456abc789   memory-test     5.23%   512MiB / 512MiB    100.00% 0B / 0B    0B / 0B

# (Press Ctrl+C to stop)

# View stats for specific container
docker stats cpu-test

# Expected output:
# Only cpu-test stats

# View stats without streaming (one snapshot)
docker stats --no-stream

# Expected output:
# Single line of stats for all containers

# View stats for multiple containers
docker stats cpu-test memory-test

# Expected output:
# Stats for both containers

# Save stats to file (one snapshot)
docker stats --no-stream > stats.txt

# Expected output:
# File contains stats

# View file
cat stats.txt

# Expected output:
# CONTAINER ID   NAME            CPU %   MEM USAGE...

# Monitor stats with specific format
docker stats --format "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}"

# Expected output:
# CONTAINER      CPU %       MEM USAGE / LIMIT
# cpu-test       95.45%      8.2MiB / 7.8GiB
# memory-test    5.23%       512MiB / 512MiB

# Monitor just container names and memory
docker stats --format "table {{.Container}}\t{{.MemPerc}}"

# Expected output:
# CONTAINER      MEM %
# cpu-test       0.10%
# memory-test    100.00%

# Watch stats in loop (every 5 seconds)
watch -n 5 'docker stats --no-stream'

# Expected output:
# Updates every 5 seconds

# Capture stats history
for i in {1..10}; do
  docker stats --no-stream >> stats-history.txt
  sleep 2
done

# Expected output:
# File contains multiple snapshots

# Analyze captured stats
cat stats-history.txt | grep cpu-test

# Expected output:
# Shows cpu-test stats over time

# Monitor with curl (via Docker API)
curl -s --unix-socket /var/run/docker.sock \
  'http://localhost/containers/cpu-test/stats?stream=false' | jq '.memory_stats'

# Expected output:
# Detailed memory statistics in JSON

# Check CPU throttling
docker stats --no-stream | awk '{
  if (NR>1) print $1, $3, $4
}'

# Expected output:
# Shows which containers are using most CPU

# Check memory pressure
docker stats --no-stream | awk '{
  if (NR>1 && substr($5, length($5)-1) == "100") print "WARNING:", $1, "at", $5
}'

# Expected output:
# Alerts if any container at 100% memory

# Run lightweight container
docker run -d --name light alpine sleep 3600

# Compare stats
docker stats --no-stream light cpu-test memory-test

# Expected output:
# Shows very different resource usage

# Cleanup (force stop memory test if stuck)
docker kill cpu-test memory-test light

# Expected output:
# Containers killed

# View stats after stop
docker stats --no-stream

# Expected output:
# Shows stats while containers are stopping/stopped
```

### Explanation

**Resource Statistics:**

| Metric | Description |
|--------|-------------|
| CPU % | Percentage of host CPU used |
| MEM USAGE | Current memory used |
| MEM LIMIT | Memory limit for container |
| MEM % | Percentage of limit used |
| NET I/O | Network input/output |
| BLOCK I/O | Disk read/write |

**Performance Monitoring:**
1. Track resource trends over time
2. Identify resource leaks (ever-growing memory)
3. Verify resource limits are enforced
4. Capacity planning
5. Performance optimization

---

## Exercise 7: View running processes

### Solution

### Commands

```bash
# Run a container with multiple processes
docker run -d --name process-test nginx

# Expected output:
# Container ID

# List processes in container
docker top process-test

# Expected output:
# UID     PID     PPID    C   STIME   TTY STAT    TIME     COMMAND
# root    1       0       0   12:00   ?   Ss      0:00     nginx: master process
# www-data 12     1       0   12:00   ?   S       0:00     nginx: worker process
# www-data 13     1       0   12:00   ?   S       0:00     nginx: worker process

# Get specific format
docker top process-test -o pid,user,cmd

# Expected output:
# PID   USER      CMD
# 1     root      nginx: master process
# 12    www-data  nginx: worker process
# 13    www-data  nginx: worker process

# Show all fields
docker top process-test -o all

# Expected output:
# F   UID   PID   PPID  C  SZ    RSS  WCHAN       TTY STAT    TIME     COMMAND
# (all fields shown)

# Show with environment variables
docker top process-test -e

# Expected output:
# Shows environment variables for each process

# Count processes
docker top process-test | wc -l

# Expected output:
# 4 (header + 3 processes)

# Get specific process
docker top process-test | grep worker

# Expected output:
# 12    www-data 13:00 ?    S   0:00   nginx: worker process
# 13    www-data 13:00 ?    S   0:00   nginx: worker process

# Run application with subprocess
docker run -d --name app-test python:3.11 python -c '
import subprocess
subprocess.Popen(["sleep", "3600"])
subprocess.Popen(["sleep", "3600"])
while True:
    import time
    time.sleep(1)
'

# Expected output:
# Container ID

# View parent-child relationships
docker top app-test -o pid,ppid,user,cmd

# Expected output:
# PID   PPID   USER     CMD
# 1     0      root     python -c ...
# 12    1      root     sleep 3600
# 13    1      root     sleep 3600

# Monitor processes over time
for i in {1..3}; do
  echo "=== Snapshot $i ==="
  docker top process-test -o pid,user,cmd
  sleep 2
done

# Expected output:
# Shows process list at different times

# Compare processes between containers
echo "=== process-test ==="
docker top process-test -o pid,cmd
echo "=== app-test ==="
docker top app-test -o pid,cmd

# Expected output:
# Shows differences in process trees

# Get resource usage per process (using ps aux format)
docker top process-test -o user,pid,%cpu,%mem,cmd

# Expected output:
# USER     PID %CPU %MEM CMD
# root     1   0.0  0.1  nginx: master process
# www-data 12  0.0  0.1  nginx: worker process
# www-data 13  0.0  0.1  nginx: worker process

# Identify zombie processes
docker top process-test | grep 'Z'

# Expected output:
# (only if zombies exist)

# Check for resource-heavy processes
docker top process-test -o pid,%cpu,%mem,cmd | sort -k2 -nr

# Expected output:
# Sorted by CPU usage, highest first

# Monitor process creation
(while true; do docker top app-test -o pid,cmd | wc -l; sleep 1; done) &
WATCH_PID=$!

# After watching
kill $WATCH_PID

# Expected output:
# Shows process count changes

# Use ps inside container for more details
docker exec process-test ps aux

# Expected output:
# USER       PID %CPU %MEM VSZ   RSS TTY STAT START   TIME COMMAND
# root       1   0.0  0.1   10408 5212 ?   Ss  12:00   0:00 nginx: master process
# www-data   12  0.0  0.1   10596 5348 ?   S   12:00   0:00 nginx: worker process
# www-data   13  0.0  0.1   10596 5348 ?   S   12:00   0:00 nginx: worker process

# Cleanup
docker rm -f process-test app-test

# Expected output:
# Containers removed
```

### Explanation

**Process Information Fields:**

| Field | Description |
|-------|-------------|
| UID | User ID running process |
| PID | Process ID |
| PPID | Parent Process ID |
| %CPU | CPU usage percentage |
| %MEM | Memory usage percentage |
| VSZ | Virtual memory size |
| RSS | Resident set size (physical memory) |
| TTY | Terminal (? = none) |
| STAT | Process state (S=sleeping, R=running, Z=zombie) |
| CMD | Command line |

**Process Monitoring Uses:**
1. Verify main process is running
2. Identify resource-heavy processes
3. Find zombie processes
4. Understand process hierarchy
5. Diagnose runaway processes

---

## Exercise 8: Debug using interactive shell

### Solution

### Commands

```bash
# Run a web application container
docker run -d --name web-app nginx

# Expected output:
# Container ID

# Open interactive shell
docker exec -it web-app bash

# Expected output:
# bash-5.1#
# (Interactive prompt)

# Inside the container, explore
# List root directory
ls -la

# Expected output:
# Shows nginx installation files

# Check processes
ps aux

# Expected output:
# Shows nginx master and worker processes

# View nginx config
cat /etc/nginx/nginx.conf | head -20

# Expected output:
# First 20 lines of config

# Check installed packages
dpkg -l | grep nginx

# Expected output:
# Shows nginx package version

# Verify port binding
netstat -tlnp 2>/dev/null | grep 80

# Expected output:
# tcp  0  0 0.0.0.0:80  0.0.0.0:*  LISTEN  1/nginx: master

# Check environment variables
env

# Expected output:
# Shows all environment variables in container

# View logs from inside
cat /var/log/nginx/access.log

# Expected output:
# Empty or shows access logs

# Test connectivity
curl http://localhost:80

# Expected output:
# <html>
# <head>
#   <title>Welcome to nginx!</title>
# </head>
# ...

# Exit shell
exit

# Expected output:
# Returns to host shell

# Open shell as specific user
docker exec -it web-app su - www-data -s /bin/bash

# Expected output:
# www-data@xxxxx:/$
# (different user prompt)

# Check current user
whoami

# Expected output:
# www-data

# Exit
exit

# Run interactive session without keeping container
docker run -it --rm alpine sh

# Expected output:
# Interactive alpine shell
# / #

# Try some commands
apk update

# Expected output:
# Shows package manager output

# Exit (removes container on exit with --rm)
exit

# Run interactive with volume for debugging
docker run -it --rm -v /tmp:/workspace alpine sh

# In container
cd /workspace
ls -la

# Expected output:
# Shows host's /tmp directory

# Create test file
echo "test data" > testfile.txt

# Back on host (in another terminal)
cat /tmp/testfile.txt

# Expected output:
# test data

# Exit
exit

# Interactive debugging of failed container
docker run -it --rm alpine false

# Expected output:
# Returns immediately with exit code 1

# But can debug first
docker run -it --rm alpine sh

# Do debugging
ls /
echo "Test"

# Exit
exit

# Run with interactive debugger
docker run -it --name debugger --link web-app:web nginx bash

# Inside debugger
ping web

# Expected output:
# PING web (nginx container IP) ...

# Debug network
curl http://web

# Expected output:
# nginx welcome page

# Exit
exit

# Cleanup
docker rm -f web-app debugger

# Expected output:
# Containers removed
```

### Explanation

**Interactive Shell Debugging:**
- `docker exec -it [container] bash`: Open interactive bash
- `-i`: Interactive (keep stdin open)
- `-t`: Allocate pseudo-TTY
- Useful for manual inspection and testing
- Can run as different users with `su`

**Debugging Techniques:**
1. Check running processes with `ps`
2. Verify network connectivity with `ping`/`curl`
3. Review configuration files
4. Check logs in real-time
5. Test application endpoints
6. Inspect environment variables

---

## Exercise 9: Inspect network configuration

### Solution

### Commands

```bash
# Create custom network
docker network create app-network

# Expected output:
# Network ID

# Run containers on network
docker run -d --name web --network app-network -p 8080:80 nginx
docker run -d --name cache --network app-network redis

# Expected output:
# Two container IDs

# Inspect network
docker network inspect app-network

# Expected output:
# [
#   {
#     "Name": "app-network",
#     "Id": "abc123...",
#     "Created": "2024-01-15T12:00:00Z",
#     "Scope": "local",
#     "Driver": "bridge",
#     "EnableIPv6": false,
#     "IPAM": {
#       "Driver": "default",
#       "Config": [
#         {
#           "Subnet": "172.18.0.0/16",
#           "Gateway": "172.18.0.1"
#         }
#       ]
#     },
#     "Containers": {
#       "abc123...": {
#         "Name": "web",
#         "EndpointID": "def456...",
#         "MacAddress": "02:42:ac:12:00:02",
#         "IPv4Address": "172.18.0.2/16",
#         "IPv6Address": ""
#       },
#       "def456...": {
#         "Name": "cache",
#         "EndpointID": "ghi789...",
#         "MacAddress": "02:42:ac:12:00:03",
#         "IPv4Address": "172.18.0.3/16",
#         "IPv6Address": ""
#       }
#     }
#   }
# ]

# Get specific network information
docker network inspect --format='{{json .Containers}}' app-network | jq

# Expected output:
# Pretty-printed container connections

# Get gateway IP
docker network inspect --format='{{index .IPAM.Config 0 .Gateway}}' app-network

# Expected output:
# 172.18.0.1

# Get subnet
docker network inspect --format='{{index .IPAM.Config 0 .Subnet}}' app-network

# Expected output:
# 172.18.0.0/16

# Get container IP address
docker inspect --format='{{.NetworkSettings.Networks.app-network.IPAddress}}' web

# Expected output:
# 172.18.0.2

# Get all networks for container
docker inspect --format='{{json .NetworkSettings.Networks}}' web | jq

# Expected output:
# Shows all networks container is on

# Get container network interface
docker exec web ip addr

# Expected output:
# Shows network interfaces and IPs

# Get routing table
docker exec web ip route

# Expected output:
# default via 172.18.0.1 dev eth0
# 172.18.0.0/16 dev eth0 proto kernel scope link src 172.18.0.2

# Get DNS configuration
docker exec web cat /etc/resolv.conf

# Expected output:
# nameserver 127.0.0.11
# options ndots:0

# Test DNS resolution
docker exec web nslookup cache

# Expected output:
# Server: 127.0.0.11
# Address: 127.0.0.11#53
# Name: cache
# Address: 172.18.0.3

# Get MAC address
docker inspect --format='{{.NetworkSettings.Networks.app-network.MacAddress}}' web

# Expected output:
# 02:42:ac:12:00:02

# Run container on multiple networks
docker network create other-network
docker network connect other-network web

# Expected output:
# Connection made

# Verify container on both networks
docker inspect --format='{{json .NetworkSettings.Networks}}' web | jq 'keys'

# Expected output:
# [
#   "app-network",
#   "other-network"
# ]

# View both network connections
docker inspect --format='
App Network IP: {{.NetworkSettings.Networks.app-network.IPAddress}}
Other Network IP: {{.NetworkSettings.Networks.other-network.IPAddress}}' web

# Expected output:
# App Network IP: 172.18.0.2
# Other Network IP: 172.19.0.2

# Disconnect from network
docker network disconnect other-network web

# Verify disconnection
docker inspect --format='{{json .NetworkSettings.Networks}}' web | jq 'keys'

# Expected output:
# [
#   "app-network"
# ]

# List all networks
docker network ls

# Expected output:
# NETWORK ID   NAME            DRIVER   SCOPE
# abc123...    app-network     bridge   local
# def456...    other-network   bridge   local
# ghi789...    bridge          bridge   local
# jkl012...    host            host     local
# mno345...    none            null     local

# Inspect bridge network (default)
docker network inspect bridge

# Expected output:
# Shows default bridge network config

# Cleanup
docker rm -f web cache
docker network rm app-network other-network

# Expected output:
# All removed
```

### Explanation

**Network Configuration Fields:**

| Field | Description |
|-------|-------------|
| Name | Network name |
| Driver | bridge, overlay, host, none, macvlan |
| Subnet | IP range for network |
| Gateway | Default gateway |
| Containers | Containers connected |
| IPAddress | Container IP on network |
| MacAddress | Container MAC address |

**Network Debugging:**
1. Verify container IP addresses
2. Check subnet and gateway
3. Confirm DNS resolution working
4. View network driver type
5. Track which containers connected

---

## Exercise 10: Analyze container events

### Solution

### Commands

```bash
# Monitor all events
docker events

# Expected output:
# 2024-01-15T12:00:00.123456Z container create abc123...
# 2024-01-15T12:00:01.234567Z container start abc123...
# 2024-01-15T12:00:30.345678Z container stop abc123...
# (Shows all container events)

# (Press Ctrl+C to stop)

# Monitor events for specific container
docker events --filter type=container

# Expected output:
# Shows only container events (creates, starts, stops, etc.)

# Monitor image events
docker events --filter type=image

# Expected output:
# build, pull, push, tag, untag, delete events

# In one terminal, start monitoring
docker events --filter type=container --format='{{.Time}} {{.Type}} {{.Action}} {{.Actor.ID}}'

# Expected output:
# 2024-01-15T12:00:00Z container create abc123def456...
# (waits for events)

# In another terminal, run container
docker run -d --name test-event alpine sleep 100

# Back in monitoring terminal:
# Expected output:
# 2024-01-15T12:00:05Z container create abc123def456...
# 2024-01-15T12:00:05Z container start abc123def456...

# Filter by status
docker events --filter status=start

# Expected output:
# Only shows start events

# Filter by status=stop
docker events --filter status=stop

# Expected output:
# Only shows stop events

# Monitor with custom format
docker events --format='{{.Actor.Attributes.name}}: {{.Action}}'

# Expected output:
# test-event: create
# test-event: start

# Monitor multiple filters
docker events --filter type=container --filter status=start

# Expected output:
# Only container start events

# Monitor specific container by name
docker run -d --name target-app nginx &

docker events --filter container=target-app

# Expected output:
# Events only for target-app container

# Monitor image pull events
docker events --filter type=image --filter status=pull &
MONITOR_PID=$!

# Pull an image
docker pull alpine:latest

# Expected output in monitor:
# pull, download, update events

# Monitor with time range (past events)
docker events --since 2024-01-15T11:00:00Z --until 2024-01-15T12:00:00Z

# Expected output:
# Shows historical events in time range

# Create event log
docker events > docker-events.log &
EVENT_LOG_PID=$!

# Run some operations
docker run -d --name app1 nginx
sleep 2
docker stop app1
sleep 2
docker rm app1

# Stop logging
kill $EVENT_LOG_PID

# View event log
cat docker-events.log

# Expected output:
# create, start, stop, destroy events for app1

# Parse event log for specific actions
grep "start" docker-events.log

# Expected output:
# Shows only start events

# Count events by type
grep -o '"Type":"[^"]*"' docker-events.log | sort | uniq -c

# Expected output:
# Shows count of each event type

# Filter by image name
docker events --filter image=nginx

# Expected output:
# Shows events related to nginx image

# Monitor with actor labels
docker events --format='{{json .}}'

# Expected output:
# Full JSON format with all details

# Parse JSON events
docker events --format='{{json .}}' | jq -r '{time: .Time, type: .Type, action: .Action, actor: .Actor.ID}'

# Expected output:
# {
#   "time": "2024-01-15T12:00:00Z",
#   "type": "container",
#   "action": "start",
#   "actor": "abc123..."
# }

# Create monitoring script
cat > monitor-events.sh << 'EOF'
#!/bin/bash

# Monitor and log events with alerts
docker events --format='{{.Time}} {{.Type}} {{.Action}} {{.Actor.Attributes.name}}' | while read event; do
  echo "$(date): $event"
  
  # Alert on crashes
  if echo "$event" | grep -q "die"; then
    echo "ALERT: Container died - $event" | mail -s "Docker Alert" admin@example.com
  fi
done
EOF

chmod +x monitor-events.sh

# Run monitoring script
./monitor-events.sh &

# Cleanup
docker rm -f test-event target-app app1 2>/dev/null
kill $MONITOR_PID 2>/dev/null
kill $EVENT_LOG_PID 2>/dev/null
kill %1 2>/dev/null

# Expected output:
# All cleaned up
```

### Explanation

**Container Events:**

| Event | When It Occurs |
|-------|----------------|
| create | Container created |
| start | Container started |
| run | Container started |
| stop | Container stopped |
| kill | Container killed |
| pause | Container paused |
| unpause | Container unpaused |
| restart | Container restarted |
| die | Container exited/died |
| destroy | Container removed |

**Event Filtering:**
- `type`: container, image, network, volume, etc.
- `status`: action type
- `container`: specific container
- `image`: specific image
- `since`/`until`: time range

**Event Monitoring Uses:**
1. Real-time operational dashboards
2. Container lifecycle tracking
3. Audit logging
4. Alerting on failures
5. Performance analysis triggers