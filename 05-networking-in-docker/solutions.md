# Solutions: Networking in Docker

Comprehensive solutions for all 10 exercises with detailed commands and explanations.

---

## Exercise 1: Create custom bridge network

### Solution

### Commands

```bash
# Create a custom bridge network
docker network create my-network

# Expected output:
# a1b2c3d4e5f6789 (network ID)

# Verify the network was created
docker network ls

# Expected output:
# NETWORK ID     NAME        DRIVER    SCOPE
# a1b2c3d4e5f6   my-network  bridge    local
# b2c3d4e5f6a7   bridge      bridge    local
# c3d4e5f6a7b8   host        host      local
# d4e5f6a7b8c9   none        null      local

# Inspect the network details
docker network inspect my-network

# Expected output:
# [
#   {
#     "Name": "my-network",
#     "Id": "a1b2c3d4e5f6789...",
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
#     "Internal": false,
#     "Attachable": false,
#     "Ingress": false,
#     "Containers": {},
#     "Options": {},
#     "Labels": {}
#   }
# ]

# Create network with custom options
docker network create \
  --driver bridge \
  --subnet 192.168.100.0/24 \
  --gateway 192.168.100.1 \
  custom-network

# Inspect to see custom settings
docker network inspect custom-network | grep -A 10 "IPAM"

# Expected output shows custom subnet and gateway

# Create network with labels
docker network create \
  --label env=production \
  --label team=backend \
  labeled-network

# Inspect to see labels
docker network inspect labeled-network | grep -A 3 '"Labels"'

# Create internal network (no external access)
docker network create \
  --internal \
  internal-network

# Inspect to verify
docker network inspect internal-network | grep "Internal"

# Expected output:
# "Internal": true
```

### Explanation

**Custom Bridge Networks:**
- User-defined bridges provide better isolation
- Automatic DNS resolution between containers
- Can specify custom subnets and gateways
- Better for multi-container applications

**Network Types:**

| Type | Isolation | DNS | Use Case |
|------|-----------|-----|----------|
| bridge | Container-to-container only | Automatic | Default, dev/test |
| host | No isolation | Host | Performance-critical |
| overlay | Multi-host | Automatic | Docker Swarm |
| none | No networking | N/A | Special cases |

---

## Exercise 2: Run containers on custom network

### Solution

### Commands

```bash
# Run first container on custom network
docker run -d \
  --name web \
  --network my-network \
  nginx:latest

# Expected output:
# a1b2c3d4e5f6

# Run second container on same network
docker run -d \
  --name database \
  --network my-network \
  alpine:latest \
  sleep 300

# Expected output:
# b2c3d4e5f6a7

# Run third container on same network
docker run -d \
  --name api \
  --network my-network \
  alpine:latest \
  sleep 300

# Expected output:
# c3d4e5f6a7b8

# Inspect network to see connected containers
docker network inspect my-network

# Expected output:
# "Containers": {
#   "a1b2c3d4e5f6": {
#     "Name": "web",
#     "EndpointID": "...",
#     "MacAddress": "...",
#     "IPv4Address": "172.18.0.2/16",
#     "IPv6Address": ""
#   },
#   "b2c3d4e5f6a7": {
#     "Name": "database",
#     "EndpointID": "...",
#     "MacAddress": "...",
#     "IPv4Address": "172.18.0.3/16",
#     "IPv6Address": ""
#   },
#   "c3d4e5f6a7b8": {
#     "Name": "api",
#     "EndpointID": "...",
#     "MacAddress": "...",
#     "IPv4Address": "172.18.0.4/16",
#     "IPv6Address": ""
#   }
# }

# Get network details from container perspective
docker inspect web | grep -A 20 '"Networks"'

# Expected output:
# "Networks": {
#   "my-network": {
#     "IPAMConfig": null,
#     "Links": null,
#     "Aliases": null,
#     "NetworkID": "a1b2c3d4e5f6789...",
#     "EndpointID": "...",
#     "Gateway": "172.18.0.1",
#     "IPAddress": "172.18.0.2",
#     "IPPrefixLen": 16,
#     "IPv6Gateway": "",
#     "IPv6Address": "",
#     "MacAddress": "02:42:ac:12:00:02"
#   }
# }

# Containers can communicate by name (DNS)
docker exec web ping -c 2 database

# Expected output:
# PING database (172.18.0.3): 56 data bytes
# 64 bytes from 172.18.0.3: seq=0 ttl=64 time=0.123 ms
# 64 bytes from 172.18.0.3: seq=1 ttl=64 time=0.087 ms

# Test communication between all containers
docker exec api ping -c 1 web

# Expected output:
# 1 packets transmitted, 1 packets received

docker exec database ping -c 1 api

# Expected output:
# 1 packets transmitted, 1 packets received
```

### Explanation

**Running on Custom Networks:**
- Containers get IP addresses from network subnet
- Docker DNS automatically resolves container names
- Isolation from default bridge network
- Service discovery works by container name

**Network Connectivity:**
- Containers on same network can communicate
- Containers on different networks cannot (without explicit routes)
- DNS names resolve within network scope
- Gateway provides default route

---

## Exercise 3: Test DNS between containers

### Solution

### Commands

```bash
# Use the network from previous exercise or create new one
docker network create dns-test

# Run containers
docker run -d \
  --name resolver \
  --network dns-test \
  alpine:latest \
  sleep 300

docker run -d \
  --name server \
  --network dns-test \
  alpine:latest \
  sleep 300

# Test DNS resolution
docker exec resolver ping -c 1 server

# Expected output:
# PING server (172.19.0.3): 56 data bytes
# 64 bytes from 172.19.0.3: seq=0 ttl=64 time=0.087 ms
# --- server statistics ---
# 1 packets transmitted, 1 packets received, 0% packet loss

# Get the resolved IP
docker exec resolver nslookup server

# Expected output:
# Server:    127.0.0.11
# Address:   127.0.0.11:53
# 
# Name:      server
# Address:   172.19.0.3

# DNS lookup by IP address (reverse DNS)
docker exec resolver nslookup 172.19.0.3

# Expected output:
# Server:    127.0.0.11
# Address:   127.0.0.11:53
#
# Name:      server
# Address:   172.19.0.3

# Test with multiple names
docker run -d \
  --name db1 \
  --network dns-test \
  alpine:latest \
  sleep 300

docker run -d \
  --name db2 \
  --network dns-test \
  alpine:latest \
  sleep 300

docker run -d \
  --name cache \
  --network dns-test \
  alpine:latest \
  sleep 300

# Resolve all names
docker exec resolver sh -c 'for name in server db1 db2 cache; do echo "=== $name ==="; nslookup $name; done'

# Expected output:
# === server ===
# Name:      server
# Address:   172.19.0.3
# === db1 ===
# Name:      db1
# Address:   172.19.0.4
# === db2 ===
# Name:      db2
# Address:   172.19.0.5
# === cache ===
# Name:      cache
# Address:   172.19.0.6

# Test communication by name
docker exec server sh -c 'nc -zv db1 80'
docker exec db1 sh -c 'nc -zv cache 6379'

# Test with network aliases
docker run -d \
  --name primary-db \
  --network dns-test \
  --network-alias db \
  --network-alias database \
  alpine:latest \
  sleep 300

# All aliases resolve to the same container
docker exec resolver nslookup db

# Expected output shows IP of primary-db

docker exec resolver nslookup database

# Expected output shows same IP

docker exec resolver nslookup primary-db

# Expected output shows same IP

# Test DNS in default bridge network (should fail)
docker run -d \
  --name no-dns \
  nginx:latest

# This should fail (no DNS in default bridge)
docker exec no-dns ping -c 1 web

# Expected output:
# ping: bad address 'web'

# But IP address works
docker inspect no-dns | grep "IPAddress"
# Get the IP address

docker exec web ping -c 1 <IP_of_no-dns>

# Expected output:
# 1 packets transmitted, 1 packets received (if on default bridge)
```

### Explanation

**Docker DNS:**
- Built-in DNS server at 127.0.0.11:53
- Works only in custom bridge networks
- Automatic resolution of container names
- Default bridge network doesn't have DNS
- Network aliases provide alternative names

**DNS Resolution:**
1. Container asks 127.0.0.11 for name
2. Docker daemon resolves to container IP
3. Response returned to querying container
4. Traffic flows directly between containers

**Default Bridge Limitations:**
- No automatic DNS resolution
- Must use `--link` (deprecated) or IP addresses
- Custom networks are preferred

---

## Exercise 4: Publish ports with -p flag

### Solution

### Commands

```bash
# Run container with port mapping
docker run -d \
  --name web-server \
  -p 8080:80 \
  nginx:latest

# Expected output:
# a1b2c3d4e5f6

# Verify the port is published
docker port web-server

# Expected output:
# 80/tcp -> 0.0.0.0:8080

# Test access from host
curl http://localhost:8080

# Expected output:
# (nginx default page HTML)

# Check port in docker ps
docker ps | grep web-server

# Expected output:
# ... 0.0.0.0:8080->80/tcp ...

# Map multiple ports
docker run -d \
  --name app-server \
  -p 3000:3000 \
  -p 5432:5432 \
  -p 6379:6379 \
  alpine:latest \
  sleep 300

# Verify all ports are published
docker port app-server

# Expected output:
# 3000/tcp -> 0.0.0.0:3000
# 5432/tcp -> 0.0.0.0:5432
# 6379/tcp -> 0.0.0.0:6379

# Map to specific host interface
docker run -d \
  --name localhost-only \
  -p 127.0.0.1:9000:80 \
  nginx:latest

# Can only access from localhost
curl http://localhost:9000

# Expected output:
# (nginx page)

# Cannot access from other IPs (unless on same machine)

# Map UDP port
docker run -d \
  --name dns-server \
  -p 53:53/udp \
  alpine:latest \
  sleep 300

# Verify UDP port
docker port dns-server

# Expected output:
# 53/udp -> 0.0.0.0:53

# Map same container port to different host ports
docker run -d \
  --name web-replica1 \
  -p 8081:80 \
  nginx:latest

docker run -d \
  --name web-replica2 \
  -p 8082:80 \
  nginx:latest

docker run -d \
  --name web-replica3 \
  -p 8083:80 \
  nginx:latest

# Test access to all replicas
curl http://localhost:8081
curl http://localhost:8082
curl http://localhost:8083

# All are accessible on different host ports
# but same container port (80)

# Check all port mappings
docker ps | grep web-replica
```

### Explanation

**Port Publishing:**
- `-p` flag maps host port to container port
- Format: `-p [host_port]:[container_port]`
- Can specify protocol: `-p 53:53/udp`
- Can specify IP: `-p 127.0.0.1:port:port`

**Common Patterns:**
- `8080:80` - Access web service on 8080
- `3306:3306` - Database on standard port
- `127.0.0.1:5432:5432` - Only local access
- `port/udp` - UDP protocol

---

## Exercise 5: Expose multiple ports

### Solution

### Commands

```bash
# Create Dockerfile with EXPOSE instructions
cat > Dockerfile << 'EOF'
FROM nginx:latest

# Expose web server port
EXPOSE 80

# Expose custom monitoring port
EXPOSE 9000

# Expose health check port
EXPOSE 8888

CMD ["nginx", "-g", "daemon off;"]
EOF

# Build the image
docker build -t multi-port-app:1.0 .

# Expected output:
# Successfully tagged multi-port-app:1.0

# Run without -p (EXPOSE only documents, doesn't publish)
docker run -d \
  --name expose-demo \
  multi-port-app:1.0

# Check what's exposed
docker inspect expose-demo | grep -A 5 '"ExposedPorts"'

# Expected output:
# "ExposedPorts": {
#   "80/tcp": {},
#   "8888/tcp": {},
#   "9000/tcp": {}
# }

# But ports aren't accessible from host yet
# curl http://localhost:80  # Fails

# Publish all exposed ports
docker run -d \
  --name expose-publish \
  -P \
  multi-port-app:1.0

# The -P flag publishes all EXPOSE ports to random ports
docker port expose-publish

# Expected output:
# 80/tcp -> 0.0.0.0:32768
# 8888/tcp -> 0.0.0.0:32769
# 9000/tcp -> 0.0.0.0:32770

# Test access
curl http://localhost:32768

# Expected output:
# (nginx page)

# Publish specific exposed ports
docker run -d \
  --name expose-selective \
  -p 8080:80 \
  -p 9000:9000 \
  multi-port-app:1.0

# Only these two are published (8888 is not)
docker port expose-selective

# Expected output:
# 80/tcp -> 0.0.0.0:8080
# 9000/tcp -> 0.0.0.0:9000

# Map exposed port to different host port
docker run -d \
  --name expose-remap \
  -p 4000:80 \
  -p 5000:9000 \
  -p 6000:8888 \
  multi-port-app:1.0

# All ports remapped
docker port expose-remap

# Expected output:
# 80/tcp -> 0.0.0.0:4000
# 8888/tcp -> 0.0.0.0:6000
# 9000/tcp -> 0.0.0.0:5000

# Test access
curl http://localhost:4000  # Port 80
curl http://localhost:5000  # Port 9000
curl http://localhost:6000  # Port 8888
```

### Explanation

**EXPOSE vs -p:**

| Aspect | EXPOSE | -p |
|--------|--------|-----|
| Purpose | Documentation | Publishing |
| Actual Effect | None on networking | Opens port |
| Required | No | Yes to access |
| Usage | Dockerfile | docker run |

**Best Practices:**
1. Use EXPOSE in Dockerfile for documentation
2. Use -p to actually publish ports
3. Use -P for quick testing with random ports
4. Map to non-standard host ports in development

---

## Exercise 6: Connect running container to network

### Solution

### Commands

```bash
# Create two networks
docker network create network-a
docker network create network-b

# Run container on network-a
docker run -d \
  --name app \
  --network network-a \
  alpine:latest \
  sleep 300

# Expected output:
# a1b2c3d4e5f6

# Verify container is on network-a
docker network inspect network-a | grep -A 10 '"Containers"'

# Expected output shows 'app' container

# Now connect the same container to network-b
docker network connect network-b app

# Expected output:
# (no output if successful)

# Verify container is now on both networks
docker network inspect network-a | grep '"Name": "app"'

# Expected output shows app in network-a

docker network inspect network-b | grep '"Name": "app"'

# Expected output shows app in network-b

# Check from container perspective
docker inspect app | grep -A 30 '"Networks"'

# Expected output:
# "Networks": {
#   "network-a": {
#     "IPAMConfig": null,
#     "Links": null,
#     "Aliases": null,
#     "NetworkID": "...",
#     "EndpointID": "...",
#     "Gateway": "172.18.0.1",
#     "IPAddress": "172.18.0.2",
#     ...
#   },
#   "network-b": {
#     "IPAMConfig": null,
#     "Links": null,
#     "Aliases": null,
#     "NetworkID": "...",
#     "EndpointID": "...",
#     "Gateway": "172.19.0.1",
#     "IPAddress": "172.19.0.2",
#     ...
#   }
# }

# Create containers on each network
docker run -d \
  --name service-a \
  --network network-a \
  alpine:latest \
  sleep 300

docker run -d \
  --name service-b \
  --network network-b \
  alpine:latest \
  sleep 300

# Service-a and service-b can't communicate yet
docker exec service-a ping -c 1 service-b

# Expected output:
# ping: can't resolve service-b

# But app container can reach both
docker exec app ping -c 1 service-a

# Expected output:
# 1 packets transmitted, 1 packets received

docker exec app ping -c 1 service-b

# Expected output:
# 1 packets transmitted, 1 packets received

# Connect service-b to network-a
docker network connect network-a service-b

# Now service-a and service-b can communicate
docker exec service-a ping -c 1 service-b

# Expected output:
# 1 packets transmitted, 1 packets received

# Connect service-a to network-b
docker network connect network-b service-a

# All containers can now reach each other
docker exec service-a ping -c 1 service-b
docker exec service-b ping -c 1 service-a
docker exec service-b ping -c 1 app
```

### Explanation

**Multi-Network Containers:**
- Container can be connected to multiple networks
- Gets different IP on each network
- Can reach containers on all connected networks
- Useful for multi-tier applications

**Use Cases:**
1. Gradual network migration
2. Communication across network boundaries
3. Bridge multiple services
4. Staging/production transitions

---

## Exercise 7: Disconnect container from network

### Solution

### Commands

```bash
# Using containers from previous exercise
# app is connected to both network-a and network-b

# Verify current connections
docker inspect app | grep -A 20 '"Networks"'

# Expected output shows both networks

# Disconnect from network-b
docker network disconnect network-b app

# Expected output:
# (no output if successful)

# Verify disconnection
docker network inspect network-b | grep '"Containers"'

# Expected output:
# "Containers": {}  (app is no longer listed)

# Container still has network-a
docker inspect app | grep -A 10 '"Networks"'

# Expected output:
# "Networks": {
#   "network-a": { ... }
# }

# Verify container can no longer reach network-b services
docker exec app ping -c 1 service-b

# Expected output (if service-b not on network-a):
# ping: can't resolve service-b

# Disconnect from network-a
docker network disconnect network-a app

# Now container has no networks
docker inspect app | grep -A 5 '"Networks"'

# Expected output:
# "Networks": {}

# Container now has no network connectivity
docker exec app ping -c 1 8.8.8.8

# Expected output:
# ping: sendto: Network unreachable

# Reconnect to network
docker network connect network-a app

# Verify reconnection
docker inspect app | grep -A 10 '"Networks"'

# Expected output shows network-a

# Test connectivity restored
docker exec app ping -c 1 service-a

# Expected output:
# 1 packets transmitted, 1 packets received

# Force disconnect all containers from network
docker network disconnect --force network-a app

# Expected output:
# (no output)

# Verify all disconnected
docker network inspect network-a | grep '"Containers"'

# Expected output:
# "Containers": {}

# Clean up
docker network rm network-a network-b
```

### Explanation

**Connecting and Disconnecting:**
- `docker network connect` adds container to network
- `docker network disconnect` removes from network
- Container keeps running during disconnect
- Can be done on running containers
- No service interruption if other networks available

**Practical Uses:**
1. Load balancing across networks
2. Gradual infrastructure updates
3. Multi-environment deployments
4. Service reorganization

---

## Exercise 8: Use --network=host mode

### Solution

### Commands

```bash
# Run container with host network mode
docker run -d \
  --name host-mode \
  --network host \
  nginx:latest

# Expected output:
# a1b2c3d4e5f6

# Check network mode
docker inspect host-mode | grep -A 5 '"NetworkMode"'

# Expected output:
# "NetworkMode": "host"

# Container uses host's network directly (no isolation)
docker inspect host-mode | grep -A 20 '"Networks"'

# Expected output:
# "Networks": {}  (no Docker networks)

# Container can access host ports without -p flag
docker exec host-mode netstat -tuln | grep -E ':80|:443'

# Expected output shows nginx listening on host ports

# Host ports are directly accessible
curl http://localhost:80

# Expected output:
# (nginx page)

# Port conflicts may occur if app already listening
# Try to run another app on same port
docker run -d \
  --name host-app \
  --network host \
  alpine:latest \
  sh -c 'echo "test" | nc -l 80'

# This may fail or succeed depending on port availability

# Check what app is listening
docker exec host-mode netstat -tuln | grep LISTEN

# Expected output shows listening ports from container

# Compare with bridge network (default)
docker run -d \
  --name bridge-mode \
  nginx:latest

# Bridge mode container doesn't have direct port access
docker port bridge-mode

# Expected output:
# (empty, no ports published)

# Bridge container has Docker network
docker inspect bridge-mode | grep -A 20 '"Networks"'

# Expected output shows bridge network

# Host mode performance is better (no network translation)
# But less isolation

# Check IP addresses in host mode
docker exec host-mode hostname -I

# Expected output shows host IP addresses

# Compare with bridge mode
docker exec bridge-mode hostname -I

# Expected output shows container IP (172.x.x.x)

# Network interfaces visible in host mode
docker exec host-mode ip addr show

# Expected output shows all host interfaces

# Compare with bridge mode
docker exec bridge-mode ip addr show

# Expected output shows fewer interfaces (only eth0)

# Host mode DNS uses host's configuration
docker exec host-mode cat /etc/resolv.conf

# Expected output shows host's DNS servers

# Compare with bridge mode
docker exec bridge-mode cat /etc/resolv.conf

# Expected output shows Docker's DNS resolver (127.0.0.11)
```

### Explanation

**Host Network Mode:**
- Container uses host's network stack
- No network isolation
- Direct port binding without -p
- Better performance (no translation overhead)
- Useful for network tools, monitoring

**Comparison:**

| Aspect | Bridge (Default) | Host |
|--------|-----------------|------|
| Isolation | Yes | No |
| Port mapping | Needed (-p) | Direct |
| IP address | Container IP | Host IP |
| Performance | Slightly lower | Higher |
| Security | Better | Lower |
| Use case | Most apps | Monitoring, tools |

**Limitations:**
- Not available on Docker Desktop (as expected)
- Less isolation
- Port conflicts more likely
- Security implications

---

## Exercise 9: Inspect network details

### Solution

### Commands

```bash
# Create a network with containers
docker network create inspect-demo
docker run -d --name service1 --network inspect-demo alpine:latest sleep 300
docker run -d --name service2 --network inspect-demo alpine:latest sleep 300

# Get full network details
docker network inspect inspect-demo

# Expected output:
# [
#   {
#     "Name": "inspect-demo",
#     "Id": "abc123...",
#     "Created": "2024-01-15T12:00:00Z",
#     "Scope": "local",
#     "Driver": "bridge",
#     "EnableIPv6": false,
#     "IPAM": {
#       "Driver": "default",
#       "Config": [
#         {
#           "Subnet": "172.20.0.0/16",
#           "Gateway": "172.20.0.1"
#         }
#       ]
#     },
#     "Internal": false,
#     "Attachable": false,
#     "Ingress": false,
#     "Containers": {
#       "id1": {
#         "Name": "service1",
#         "EndpointID": "...",
#         "MacAddress": "02:42:ac:14:00:02",
#         "IPv4Address": "172.20.0.2/16",
#         "IPv6Address": ""
#       },
#       "id2": {
#         "Name": "service2",
#         "EndpointID": "...",
#         "MacAddress": "02:42:ac:14:00:03",
#         "IPv4Address": "172.20.0.3/16",
#         "IPv6Address": ""
#       }
#     },
#     "Options": {},
#     "Labels": {}
#   }
# ]

# Extract specific information
docker network inspect --format='{{.Driver}}' inspect-demo

# Expected output:
# bridge

docker network inspect --format='{{.IPAM.Config}}' inspect-demo

# Expected output:
# [{172.20.0.0/16 172.20.0.1}]

# List containers on network
docker network inspect --format='{{range .Containers}}{{.Name}}{{end}}' inspect-demo

# Expected output:
# service1service2

# Get container count
docker network inspect --format='{{len .Containers}}' inspect-demo

# Expected output:
# 2

# Get detailed container info
docker network inspect inspect-demo | jq '.[] | .Containers'

# Expected output shows all containers with their info

# Check container's view of the network
docker inspect service1 | grep -A 25 '"NetworkSettings"'

# Expected output:
# "NetworkSettings": {
#   "Bridge": "",
#   "Gateway": "",
#   "IPAddress": "",
#   "IPPrefixLen": 0,
#   "IPv6Gateway": "",
#   "GlobalIPv6Address": "",
#   "GlobalIPv6PrefixLen": 0,
#   "MacAddress": "",
#   "Networks": {
#     "inspect-demo": {
#       "IPAMConfig": null,
#       "Links": null,
#       "Aliases": null,
#       "NetworkID": "abc123...",
#       "EndpointID": "...",
#       "Gateway": "172.20.0.1",
#       "IPAddress": "172.20.0.2",
#       "IPPrefixLen": 16,
#       "IPv6Gateway": "",
#       "IPv6Address": "",
#       "MacAddress": "02:42:ac:14:00:02"
#     }
#   }
# }

# List all networks with detailed info
docker network ls --format "table {{.Name}}\t{{.Driver}}\t{{.Scope}}"

# Expected output:
# NAME            DRIVER    SCOPE
# bridge          bridge    local
# host            host      local
# inspect-demo    bridge    local
# none            null      local

# Compare networks side by side
docker network ls
docker network inspect bridge inspect-demo | jq '.[] | {Name, Driver, Scope, Containers: (.Containers | length)}'

# Get gateway information
docker network inspect inspect-demo | jq '.[] | .IPAM.Config[0].Gateway'

# Expected output:
# "172.20.0.1"

# Get network labels
docker network inspect --format='{{.Labels}}' inspect-demo

# Expected output:
# map[]  (or specific labels if set)

# Detailed port inspection from containers
docker port service1

# Expected output:
# (empty, no port mapping)

# Get all network interfaces in container
docker exec service1 ip addr show

# Expected output shows container's network interfaces
```

### Explanation

**Network Inspection Keys:**

| Key | Meaning |
|-----|---------|
| Name | Network name |
| Id | Unique network ID |
| Driver | Network driver (bridge, host, etc.) |
| Scope | local or global |
| Containers | Connected containers |
| IPAM | IP allocation settings |
| Gateway | Default gateway |
| Subnet | Network subnet |

**Common Queries:**
1. List all containers on network
2. Get gateway and subnet info
3. Check IP addresses assigned
4. Verify network connectivity
5. Monitor network usage

---

## Exercise 10: Create overlay network (Swarm)

### Solution

### Commands

```bash
# Initialize Docker Swarm (required for overlay networks)
docker swarm init

# Expected output:
# Swarm initialized: current node (abc123...) is now a manager.
# To add a worker to this swarm, run the following command:
#     docker swarm join --token SWMTKN-... <IP>:2377

# Create an overlay network
docker network create \
  --driver overlay \
  --subnet 10.0.0.0/24 \
  overlay-network

# Expected output:
# abc123def456  (network ID)

# Verify the network was created
docker network ls | grep overlay

# Expected output:
# abc123def456  overlay-network  overlay  swarm

# Inspect overlay network
docker network inspect overlay-network

# Expected output:
# [
#   {
#     "Name": "overlay-network",
#     "Id": "abc123...",
#     "Created": "2024-01-15T12:00:00Z",
#     "Scope": "swarm",
#     "Driver": "overlay",
#     "EnableIPv6": false,
#     "IPAM": {
#       "Driver": "default",
#       "Config": [
#         {
#           "Subnet": "10.0.0.0/24",
#           "Gateway": "10.0.0.1"
#         }
#       ]
#     },
#     "Internal": false,
#     "Attachable": false,
#     "Ingress": false,
#     "Containers": {},
#     "Options": {},
#     "Labels": {}
#   }
# ]

# Create a service on overlay network
docker service create \
  --name web \
  --network overlay-network \
  --replicas 2 \
  nginx:latest

# Expected output:
# abc123def456 (service ID)

# Create another service
docker service create \
  --name db \
  --network overlay-network \
  --replicas 1 \
  alpine:latest \
  sleep 300

# List services
docker service ls

# Expected output:
# ID        NAME   MODE        REPLICAS  IMAGE
# abc123    web    replicated  2/2       nginx:latest
# def456    db     replicated  1/1       alpine:latest

# Inspect the service
docker service inspect web

# Expected output includes network information

# Check service endpoint
docker service inspect --format='{{json .Endpoint.VirtualIPs}}' web

# Expected output shows virtual IP on overlay network

# Access services by name (DNS service discovery)
docker exec <container_id> ping web

# Expected output:
# 1 packets transmitted, 1 packets received

# Load balancing across replicas
# The DNS name resolves to all replicas

# Get service logs
docker service logs web

# Expected output shows logs from all replicas

# Scale the service
docker service scale web=3

# Expected output:
# web scaled to 3

# Verify scaling
docker service ls

# Expected output shows web replicas=3/3

# Create overlay network with custom options
docker network create \
  --driver overlay \
  --subnet 10.1.0.0/24 \
  --opt com.docker.network.driver.overlay.vxlanid=256 \
  custom-overlay

# Inspect to see options
docker network inspect custom-overlay | grep -A 3 '"Options"'

# Test DNS service discovery
docker service create \
  --name app1 \
  --network overlay-network \
  alpine:latest \
  sleep 300

docker service create \
  --name app2 \
  --network overlay-network \
  alpine:latest \
  sh -c 'while true; do ping app1; sleep 1; done'

# View logs to see DNS resolution working
docker service logs app2 | head -5

# Remove services
docker service rm web db app1 app2

# Leave Swarm mode (if desired)
docker swarm leave --force

# Expected output:
# Node left the swarm.
```

### Explanation

**Overlay Networks:**
- Multi-host networking for Docker Swarm
- Uses VXLAN tunnel for encryption
- Service discovery via DNS
- Load balancing built-in
- Requires Swarm mode

**Key Concepts:**

| Concept | Description |
|---------|-------------|
| Service | Group of replicas |
| VIP | Virtual IP for service |
| DNS | Service name resolution |
| Load balance | Round-robin across replicas |
| VXLAN | Encrypted tunnel protocol |

**Service Discovery:**
- Services get DNS name
- Resolves to Virtual IP
- VIP load balances across replicas
- Automatic failover

**Limitations:**
- Requires Swarm mode
- More complex than bridge networks
- Not needed for single-host deployments
- For production multi-host setups