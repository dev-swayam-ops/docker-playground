# Solutions: Volumes and Storage

Comprehensive solutions for all 10 exercises with detailed commands and explanations.

---

## Exercise 1: Create named volume and mount to container

### Solution

### Commands

```bash
# Create a named volume
docker volume create my-data

# Expected output:
# my-data

# Verify the volume was created
docker volume ls

# Expected output:
# DRIVER    VOLUME NAME
# local     my-data

# Inspect the volume details
docker volume inspect my-data

# Expected output:
# [
#   {
#     "CreatedAt": "2024-01-15T12:00:00Z",
#     "Driver": "local",
#     "Labels": {},
#     "Mountpoint": "/var/lib/docker/volumes/my-data/_data",
#     "Name": "my-data",
#     "Options": {},
#     "Scope": "local"
#   }
# ]

# Run a container and mount the named volume
docker run -d \
  --name storage-container \
  -v my-data:/data \
  alpine:latest \
  sleep 300

# Expected output:
# a1b2c3d4e5f6

# Verify the mount point
docker inspect storage-container | grep -A 20 '"Mounts"'

# Expected output:
# "Mounts": [
#   {
#     "Type": "volume",
#     "Name": "my-data",
#     "Source": "/var/lib/docker/volumes/my-data/_data",
#     "Destination": "/data",
#     "Driver": "local",
#     "Mode": "z",
#     "RW": true,
#     "Propagation": ""
#   }
# ]

# Create a file in the mounted volume
docker exec storage-container sh -c 'echo "Hello from volume" > /data/test.txt'

# Verify the file was created
docker exec storage-container cat /data/test.txt

# Expected output:
# Hello from volume

# The file persists even after removing the container
docker stop storage-container
docker rm storage-container

# Run a new container with the same volume
docker run -d \
  --name new-container \
  -v my-data:/data \
  alpine:latest \
  sleep 300

# The file still exists
docker exec new-container cat /data/test.txt

# Expected output:
# Hello from volume
```

### Explanation

**Named Volumes:**
- Created and managed by Docker
- Stored in `/var/lib/docker/volumes/` on the host
- Can be shared between multiple containers
- Persist even when container is removed
- More portable than bind mounts

**Benefits:**
- Easy backup and restore
- Can use volume drivers (NFS, etc.)
- Easier to share between containers
- Better isolation than bind mounts

---

## Exercise 2: Use bind mount from host directory

### Solution

### Commands

```bash
# Create a host directory
mkdir -p /tmp/host-data
echo "Host data" > /tmp/host-data/file.txt

# Run container with bind mount
docker run -d \
  --name bind-container \
  -v /tmp/host-data:/app/data \
  alpine:latest \
  sleep 300

# Expected output:
# b2c3d4e5f6a7

# Verify the mount
docker inspect bind-container | grep -A 20 '"Mounts"'

# Expected output:
# "Mounts": [
#   {
#     "Type": "bind",
#     "Source": "/tmp/host-data",
#     "Destination": "/app/data",
#     "Mode": "",
#     "RW": true
#   }
# ]

# Read file from container
docker exec bind-container cat /app/data/file.txt

# Expected output:
# Host data

# Create a file from the container
docker exec bind-container sh -c 'echo "Container data" > /app/data/container.txt'

# Verify it exists on the host
cat /tmp/host-data/container.txt

# Expected output:
# Container data

# Mount as read-only
docker run -d \
  --name readonly-bind \
  -v /tmp/host-data:/app/data:ro \
  alpine:latest \
  sleep 300

# Try to write (should fail)
docker exec readonly-bind sh -c 'echo "test" > /app/data/test.txt'

# Expected output:
# sh: can't create /app/data/test.txt: Read-only file system

# Cleanup
rm -rf /tmp/host-data
```

### Explanation

**Bind Mounts vs Named Volumes:**

| Aspect | Bind Mount | Named Volume |
|--------|-----------|--------------|
| Location | Any host directory | Docker managed location |
| Portability | Less portable | More portable |
| Performance | Slightly lower | Better |
| Permissions | Can have issues | Better handled |
| Use case | Development | Production |

**Permissions:**
- Bind mounts inherit host file permissions
- May need `chown` or proper permissions on host
- Named volumes handle permissions automatically

---

## Exercise 3: Share volume between multiple containers

### Solution

### Commands

```bash
# Create a shared volume
docker volume create shared-volume

# Run first container
docker run -d \
  --name container-a \
  -v shared-volume:/shared \
  alpine:latest \
  sh -c 'while true; do date >> /shared/log.txt; sleep 5; done'

# Expected output:
# c3d4e5f6a7b8

# Run second container
docker run -d \
  --name container-b \
  -v shared-volume:/shared \
  alpine:latest \
  sleep 300

# Expected output:
# d4e5f6a7b8c9

# From container-b, read the log from container-a
docker exec container-b tail -f /shared/log.txt

# Expected output:
# Mon Jan 15 12:00:00 UTC 2024
# Mon Jan 15 12:00:05 UTC 2024
# Mon Jan 15 12:00:10 UTC 2024
# (and so on...)

# Write something from container-b
docker exec container-b sh -c 'echo "Message from B" >> /shared/messages.txt'

# Read from container-a
docker exec container-a cat /shared/messages.txt

# Expected output:
# Message from B

# Verify both containers see the same volume
docker exec container-a ls -la /shared/

# Expected output:
# total XX
# drwxr-xr-x    3 root     root          4096 Jan 15 12:00 .
# drwxr-xr-x    1 root     root          4096 Jan 15 12:00 ..
# -rw-r--r--    1 root     root           128 Jan 15 12:00 log.txt
# -rw-r--r--    1 root     root            18 Jan 15 12:00 messages.txt

docker exec container-b ls -la /shared/

# Expected output: (same files)

# Add a third container
docker run -d \
  --name container-c \
  -v shared-volume:/shared \
  alpine:latest \
  sleep 300

# All three can access the same volume
docker exec container-c cat /shared/messages.txt

# Expected output:
# Message from B
```

### Explanation

**Multi-Container Volume Sharing:**
- Multiple containers can mount the same volume
- Read and write operations are visible to all
- Useful for shared state, logs, data exchange
- Named volumes make this easy

**Use Cases:**
1. Multiple replicas sharing configuration
2. Log aggregation
3. Database data sharing
4. Temporary data exchange between services

---

## Exercise 4: Backup volume data

### Solution

### Commands

```bash
# Create volume with some data
docker volume create backup-demo
docker run -d \
  --name backup-source \
  -v backup-demo:/data \
  alpine:latest \
  sh -c 'echo "Important data" > /data/file.txt && sleep 300'

# Create a backup directory on the host
mkdir -p /tmp/backups

# Method 1: Using tar inside container
docker run --rm \
  -v backup-demo:/data \
  -v /tmp/backups:/backup \
  alpine:latest \
  tar czf /backup/volume-backup.tar.gz -C /data .

# Expected output:
# (no output if successful)

# Verify the backup was created
ls -lah /tmp/backups/volume-backup.tar.gz

# Expected output:
# -rw-r--r-- 1 root root 156 Jan 15 12:00 /tmp/backups/volume-backup.tar.gz

# Check contents of backup
tar tzf /tmp/backups/volume-backup.tar.gz

# Expected output:
# ./
# ./file.txt

# Method 2: Using docker cp (from container)
docker cp backup-source:/data /tmp/backups/volume-copy

# Expected output:
# (no output if successful)

# Verify the copy
ls -la /tmp/backups/volume-copy/

# Expected output:
# total XX
# -rw-r--r-- 1 root root 15 Jan 15 12:00 file.txt

# Method 3: Using a backup container (more robust)
docker run --name backup-container \
  --volumes-from backup-source \
  -v /tmp/backups:/backup \
  alpine:latest \
  tar czf /backup/volume-backup-v2.tar.gz -C /data .

# Wait for it to complete
docker wait backup-container

# Verify
ls -lah /tmp/backups/volume-backup-v2.tar.gz

# Create an additional file and backup again
docker exec backup-source sh -c 'echo "More data" >> /data/file.txt'

# Take incremental backup with timestamp
docker run --rm \
  -v backup-demo:/data \
  -v /tmp/backups:/backup \
  alpine:latest \
  tar czf /backup/volume-backup-$(date +%Y%m%d-%H%M%S).tar.gz -C /data .

# List all backups
ls -lah /tmp/backups/

# Expected output shows multiple backups with timestamps
```

### Explanation

**Backup Methods:**

| Method | Pros | Cons |
|--------|------|------|
| tar | Simple, portable | Requires container |
| docker cp | Direct copy | Slower for large data |
| Volume snapshot | Fast (if supported) | Driver dependent |
| docker run with tar | Clean, isolated | Extra process |

**Best Practices:**
1. Regular automated backups
2. Include timestamp in backup names
3. Test restore process regularly
4. Store backups off-host
5. Consider compression for large volumes

---

## Exercise 5: Restore from backup

### Solution

### Commands

```bash
# Create a fresh volume for restore
docker volume create restore-demo

# Extract backup to the new volume
docker run --rm \
  -v restore-demo:/data \
  -v /tmp/backups:/backup \
  alpine:latest \
  tar xzf /backup/volume-backup.tar.gz -C /data

# Expected output:
# (no output if successful)

# Verify the restore
docker run -it --rm \
  -v restore-demo:/data \
  alpine:latest \
  cat /data/file.txt

# Expected output:
# Important data

# Verify all files were restored
docker run -it --rm \
  -v restore-demo:/data \
  alpine:latest \
  ls -la /data/

# Expected output:
# total XX
# drwxr-xr-x    2 root     root          4096 Jan 15 12:00 .
# drwxr-xr-x    1 root     root          4096 Jan 15 12:00 ..
# -rw-r--r--    1 root     root            15 Jan 15 12:00 file.txt

# Compare restore results with original
docker run -it --rm \
  -v backup-demo:/data \
  alpine:latest \
  md5sum /data/file.txt

# Expected output:
# abc123def456 /data/file.txt

docker run -it --rm \
  -v restore-demo:/data \
  alpine:latest \
  md5sum /data/file.txt

# Expected output:
# abc123def456 /data/file.txt (should match)

# Restore with verification
docker run --rm \
  -v restore-demo:/data \
  -v /tmp/backups:/backup \
  alpine:latest \
  sh -c 'tar xzf /backup/volume-backup.tar.gz -C /data && echo "Restore completed" && ls -la /data/'

# Expected output:
# Restore completed
# (file listing)

# Restore to a running container's volume
docker run -d \
  --name restore-container \
  -v restore-demo:/data \
  alpine:latest \
  sleep 300

docker run --rm \
  -v restore-demo:/data \
  -v /tmp/backups:/backup \
  alpine:latest \
  tar xzf /backup/volume-backup-*.tar.gz -C /data 2>/dev/null || true

# Verify from running container
docker exec restore-container ls -la /data/

# Expected output shows restored files
```

### Explanation

**Restore Process:**
1. Create new volume or use existing one
2. Extract backup archive into volume
3. Verify integrity (checksums)
4. Test application with restored data
5. Keep original backup until verified

**Verification Steps:**
- Check file count matches backup
- Verify checksums (md5sum, sha256sum)
- Test application with restored data
- Check file permissions are correct

---

## Exercise 6: Use read-only mounts

### Solution

### Commands

```bash
# Create volume with data
docker volume create readonly-demo
docker run --rm \
  -v readonly-demo:/data \
  alpine:latest \
  sh -c 'echo "Read-only data" > /data/config.txt'

# Mount as read-only
docker run -d \
  --name readonly-container \
  -v readonly-demo:/data:ro \
  alpine:latest \
  sleep 300

# Expected output:
# e5f6a7b8c9d0

# Try to write (should fail)
docker exec readonly-container sh -c 'echo "new" > /data/newfile.txt'

# Expected output:
# sh: can't create /data/newfile.txt: Read-only file system

# Read operations work fine
docker exec readonly-container cat /data/config.txt

# Expected output:
# Read-only data

# Verify the mount is read-only
docker inspect readonly-container | grep -A 15 '"Mounts"'

# Expected output includes:
# "RW": false

# Try to modify existing file (should fail)
docker exec readonly-container sh -c 'echo "modified" >> /data/config.txt'

# Expected output:
# sh: can't open /data/config.txt: Read-only file system

# Mix read-only and read-write mounts
docker run -d \
  --name mixed-container \
  -v readonly-demo:/config:ro \
  -v /tmp/writable:/data \
  alpine:latest \
  sleep 300

# This should fail (read-only)
docker exec mixed-container sh -c 'touch /config/file.txt'

# Expected output:
# Read-only file system

# This should succeed (read-write)
docker exec mixed-container sh -c 'touch /data/file.txt'

# Expected output:
# (no error)

# Verify files exist
docker exec mixed-container ls -la /data/

# Expected output:
# file.txt exists

# Useful with bind mounts too
mkdir -p /tmp/readonly-host
echo "Host config" > /tmp/readonly-host/config.txt

docker run -d \
  --name readonly-bind \
  -v /tmp/readonly-host:/config:ro \
  alpine:latest \
  sleep 300

# Try to modify (should fail)
docker exec readonly-bind sh -c 'echo "change" >> /config/config.txt'

# Expected output:
# Read-only file system
```

### Explanation

**Read-Only Mount Use Cases:**
1. Configuration files that shouldn't be modified
2. Preventing accidental data modification
3. Security - limit what containers can change
4. Compliance requirements
5. Protecting shared resources

**Mount Modes:**
- Default (rw): Read and write allowed
- `:ro`: Read-only, no writes allowed
- `:rshared`, `:rslave`, `:rprivate`: Propagation modes

---

## Exercise 7: Anonymous volumes

### Solution

### Commands

```bash
# Run container with anonymous volume (no name specified)
docker run -d \
  --name anon-container \
  -v /data \
  alpine:latest \
  sh -c 'echo "Anonymous volume data" > /data/file.txt && sleep 300'

# Expected output:
# f6a7b8c9d0e1

# List volumes - the anonymous volume will appear with a hash name
docker volume ls

# Expected output:
# DRIVER    VOLUME NAME
# local     a1b2c3d4e5f6789 (hash-based name)
# local     my-data

# Identify the anonymous volume
ANON_VOLUME=$(docker inspect anon-container | grep -oP '(?<="Name": ")[^"]*' | head -1)

# Verify the data
docker inspect anon-container | grep -A 10 '"Mounts"'

# Expected output:
# "Mounts": [
#   {
#     "Type": "volume",
#     "Name": "a1b2c3d4e5f6789",
#     "Source": "/var/lib/docker/volumes/a1b2c3d4e5f6789/_data",
#     "Destination": "/data",
#     "Driver": "local",
#     "Mode": "",
#     "RW": true,
#     "Propagation": ""
#   }
# ]

# Data persists as long as the container exists
docker stop anon-container
docker ps -a | grep anon-container

# Expected output shows container is stopped but exists

# When container is removed, anonymous volume is NOT automatically deleted
docker rm anon-container

# The volume still exists
docker volume ls

# But it's now orphaned (not used by any container)

# To automatically remove anonymous volumes
docker run -d \
  --name auto-cleanup \
  -v /data \
  --rm \
  alpine:latest \
  sleep 10

# Wait for it to complete
docker wait auto-cleanup 2>/dev/null || true

# The volume is automatically cleaned up
docker volume ls

# Check if the anonymous volume is still there
# (it might not be if --rm was used)

# Create multiple anonymous volumes in one container
docker run -d \
  --name multi-anon \
  -v /data1 \
  -v /data2 \
  -v /data3 \
  alpine:latest \
  sh -c 'echo "data1" > /data1/file.txt && echo "data2" > /data2/file.txt && echo "data3" > /data3/file.txt && sleep 300'

# List volumes - three anonymous volumes created
docker volume ls | tail -3

# Inspect shows all three
docker inspect multi-anon | grep '"Name":'

# Expected output shows three volume names
```

### Explanation

**Anonymous Volumes:**
- Created automatically with `-v /path` syntax
- Assigned random hash-based names
- Persist after container removal (unless `--rm` used)
- Harder to reference later
- Useful for temporary data

**Anonymous vs Named:**

| Aspect | Anonymous | Named |
|--------|-----------|-------|
| Naming | Hash-based | User-defined |
| Lifetime | Persistent unless --rm | Persistent |
| Reusability | Hard to reuse | Easy to reuse |
| Cleanup | Manual | Manual or prune |
| Use case | Temporary data | Persistent data |

**Cleanup:**
```bash
# Remove unused anonymous volumes
docker volume prune

# Remove all dangling volumes
docker volume prune --force
```

---

## Exercise 8: Volume driver inspection

### Solution

### Commands

```bash
# List all volumes
docker volume ls

# Expected output:
# DRIVER    VOLUME NAME
# local     my-data
# local     backup-demo
# local     readonly-demo

# Inspect a specific volume
docker volume inspect my-data

# Expected output:
# [
#   {
#     "CreatedAt": "2024-01-15T12:00:00Z",
#     "Driver": "local",
#     "Labels": {},
#     "Mountpoint": "/var/lib/docker/volumes/my-data/_data",
#     "Name": "my-data",
#     "Options": {},
#     "Scope": "local",
#     "Propagation": ""
#   }
# ]

# Extract specific fields
docker volume inspect --format='{{.Driver}}' my-data

# Expected output:
# local

docker volume inspect --format='{{.Mountpoint}}' my-data

# Expected output:
# /var/lib/docker/volumes/my-data/_data

# Check volume size (if supported by driver)
docker volume inspect my-data | grep -i size

# List all volume drivers available
docker info | grep -A 20 'Storage Drivers'

# Expected output:
# Storage Drivers: overlay2
# (Shows the storage driver being used)

# Create volume with labels
docker volume create \
  --label env=production \
  --label app=web \
  labeled-volume

# Inspect to see labels
docker volume inspect labeled-volume

# Expected output includes:
# "Labels": {
#   "env": "production",
#   "app": "web"
# }

# Filter volumes by label
docker volume ls --filter label=env=production

# Expected output:
# DRIVER    VOLUME NAME
# local     labeled-volume

# Get volume driver information
docker plugin ls

# Expected output shows available volume plugins

# Create volume with custom driver options
docker volume create \
  --driver local \
  --opt type=tmpfs \
  --opt device=tmpfs \
  --opt o=size=128m \
  tmpfs-volume

# Inspect to see options
docker volume inspect tmpfs-volume

# Expected output includes:
# "Options": {
#   "device": "tmpfs",
#   "o": "size=128m",
#   "type": "tmpfs"
# }

# Use the tmpfs volume
docker run -d \
  --name tmpfs-test \
  -v tmpfs-volume:/ramdisk \
  alpine:latest \
  sleep 300

# Check available space in tmpfs
docker exec tmpfs-test df -h /ramdisk

# Expected output:
# Filesystem      Size  Used Avail Use% Mounted on
# tmpfs           128M  0    128M   0% /ramdisk
```

### Explanation

**Volume Drivers:**
- `local`: Default, stores on host
- `nfs`: Network File System
- Custom drivers: Installed via plugins

**Volume Information:**
- `Name`: Volume identifier
- `Driver`: Storage backend
- `Mountpoint`: Host location
- `Labels`: Metadata tags
- `Options`: Driver-specific settings

**Practical Uses:**
1. Identify volume locations for backup
2. Monitor volume size and usage
3. Apply labels for organization
4. Configure driver-specific options

---

## Exercise 9: Mount tmpfs (temporary) storage

### Solution

### Commands

```bash
# Run container with tmpfs mount
docker run -d \
  --name tmpfs-demo \
  --tmpfs /tmp \
  alpine:latest \
  sleep 300

# Expected output:
# g7h8i9j0k1l2

# Verify tmpfs is mounted
docker inspect tmpfs-demo | grep -A 10 '"Mounts"'

# Expected output:
# "Mounts": [
#   {
#     "Type": "tmpfs",
#     "Destination": "/tmp",
#     "Mode": "",
#     "RW": true
#   }
# ]

# Check available tmpfs space
docker exec tmpfs-demo df -h /tmp

# Expected output:
# Filesystem      Size  Used Avail Use% Mounted on
# tmpfs           ...   ...  ...   ...% /tmp

# Create data in tmpfs
docker exec tmpfs-demo sh -c 'echo "Temporary data" > /tmp/file.txt && echo "More data" >> /tmp/file.txt'

# Verify data exists
docker exec tmpfs-demo cat /tmp/file.txt

# Expected output:
# Temporary data
# More data

# Data is in memory (fast)
docker exec tmpfs-demo ls -la /tmp/

# Expected output:
# -rw-r--r-- 1 root root 29 Jan 15 12:00 file.txt

# When container stops, tmpfs data is lost
docker stop tmpfs-demo
docker start tmpfs-demo

# Data is gone
docker exec tmpfs-demo ls -la /tmp/

# Expected output:
# (file.txt is gone)

# Mount tmpfs with size limit
docker run -d \
  --name tmpfs-limited \
  --tmpfs /tmp:size=64m \
  alpine:latest \
  sleep 300

# Check size limit
docker exec tmpfs-limited df -h /tmp

# Expected output:
# Filesystem      Size  Used Avail Use% Mounted on
# tmpfs           64M   0    64M   0% /tmp

# Try to exceed the limit
docker exec tmpfs-limited sh -c 'dd if=/dev/zero of=/tmp/largefile bs=1M count=80'

# Expected output:
# dd: error writing '/tmp/largefile': No space left on device

# Mount multiple tmpfs locations
docker run -d \
  --name multi-tmpfs \
  --tmpfs /tmp:size=128m \
  --tmpfs /var/log:size=64m \
  --tmpfs /run:size=32m \
  alpine:latest \
  sleep 300

# Verify all are mounted
docker inspect multi-tmpfs | grep -B 2 'tmpfs'

# Expected output shows three tmpfs mounts

# Check all sizes
docker exec multi-tmpfs df -h | grep tmpfs

# Expected output shows three tmpfs with different sizes

# tmpfs use cases: temporary files, caches, logs
docker run -d \
  --name app-tmpfs \
  --tmpfs /app/cache \
  --tmpfs /app/logs \
  alpine:latest \
  sh -c 'mkdir -p /app/cache /app/logs && sleep 300'

# Write to cache
docker exec app-tmpfs sh -c 'echo "cache data" > /app/cache/cache.dat'

# Write to logs
docker exec app-tmpfs sh -c 'echo "$(date) - Event" > /app/logs/app.log'

# Verify
docker exec app-tmpfs cat /app/cache/cache.dat
docker exec app-tmpfs cat /app/logs/app.log
```

### Explanation

**tmpfs (Temporary File System):**
- Stored in RAM, not on disk
- Much faster than disk storage
- Data lost when container stops
- Perfect for temporary files, caches, logs

**Use Cases:**
1. Application caches
2. Session data
3. Temporary processing files
4. Log files
5. Runtime state

**Benefits:**
- Speed: RAM is much faster than disk
- Isolation: Data doesn't persist
- Memory efficient: Can set size limits
- Security: Sensitive data in RAM only

**Limitations:**
- Limited by available RAM
- Data lost on container stop
- Not suitable for persistent data

---

## Exercise 10: Clean up unused volumes

### Solution

### Commands

```bash
# List all volumes (including unused)
docker volume ls

# Expected output:
# DRIVER    VOLUME NAME
# local     my-data
# local     backup-demo
# local     readonly-demo
# local     a1b2c3d4e5f6789  (unused)
# local     x9y8z7w6v5u4t3   (unused)

# Identify unused volumes (not attached to any container)
docker volume ls --filter dangling=true

# Expected output shows only dangling volumes

# Remove a specific volume
docker volume rm my-data

# Expected output:
# my-data

# Verify it's gone
docker volume ls | grep my-data

# Expected output:
# (no matching volume)

# Remove multiple specific volumes
docker volume rm backup-demo readonly-demo

# Expected output:
# backup-demo
# readonly-demo

# Remove all dangling volumes at once
docker volume prune

# Expected output:
# WARNING! This will remove all local volumes not used by at least one container.
# Are you sure you want to continue? [y/N] y
# Deleted Volumes:
# a1b2c3d4e5f6789
# x9y8z7w6v5u4t3
#
# Total reclaimed space: 256 MB

# Force removal without prompt
docker volume prune --force

# Expected output shows deleted volumes

# Combine with container cleanup
docker container prune --force
docker volume prune --force
docker image prune --force

# Expected output shows all three cleanups

# Check disk space saved
df -h /var/lib/docker/volumes/

# List remaining volumes
docker volume ls

# Expected output:
# DRIVER    VOLUME NAME
# (only in-use volumes remain)

# Create a script for regular cleanup
cat > cleanup-volumes.sh << 'EOF'
#!/bin/bash
echo "Cleaning up Docker volumes..."
docker volume prune --force
echo "Cleaning up Docker containers..."
docker container prune --force
echo "Cleaning up Docker images..."
docker image prune --force
echo "Cleanup complete!"
EOF

chmod +x cleanup-volumes.sh

# Run the cleanup script
./cleanup-volumes.sh

# Expected output:
# Cleaning up Docker volumes...
# Deleted Volumes: (list)
# Total reclaimed space: XXX MB
# ...

# Cleanup individual volumes when removing container
docker run -d \
  --name temp-container \
  -v temp-volume:/data \
  alpine:latest \
  sleep 10

# Wait for container to finish
docker wait temp-container 2>/dev/null || true

# Remove container with its volume
docker rm temp-container

# The named volume still exists (not automatic)
docker volume ls | grep temp-volume

# To auto-remove volumes with container, use --rm flag
docker run --rm \
  -v temp-volume2:/data \
  alpine:latest \
  sh -c 'echo "data" > /data/file.txt'

# After completion, the anonymous volume is removed
# but temp-volume2 (named) persists

# Verify
docker volume ls | grep temp-volume2

# Expected output:
# temp-volume2 still exists (named volume)
```

### Explanation

**Volume Cleanup Strategies:**

| Scenario | Command | Result |
|----------|---------|--------|
| Remove specific volume | `docker volume rm name` | Only that volume removed |
| Remove dangling volumes | `docker volume prune` | Only unused volumes removed |
| Remove all volumes | `docker volume prune -a` | All unused volumes removed |
| Auto-remove on container rm | Use `--rm` flag | Anonymous volumes removed |

**Best Practices:**
1. Regularly clean up dangling volumes
2. Use named volumes intentionally
3. Use `--rm` for temporary containers
4. Automate cleanup with scripts
5. Keep important data volumes explicitly

**Cleanup Script for Cron:**
```bash
#!/bin/bash
TIMESTAMP=$(date +%Y-%m-%d_%H-%M-%S)
echo "[$TIMESTAMP] Starting Docker cleanup..." >> /var/log/docker-cleanup.log
docker volume prune --force >> /var/log/docker-cleanup.log 2>&1
docker container prune --force >> /var/log/docker-cleanup.log 2>&1
echo "[$TIMESTAMP] Docker cleanup completed" >> /var/log/docker-cleanup.log
```
