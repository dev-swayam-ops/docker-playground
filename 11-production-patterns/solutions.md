# Solutions: Production Patterns

## Exercise 1: Implement health checks

**Objective**: Add HEALTHCHECK instruction to a Dockerfile for a web application.

### Commands

```bash
# Create a simple web app
cat > app.py << 'EOF'
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello():
    return 'Hello World', 200

@app.route('/health')
def health():
    return 'OK', 200

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
EOF

# Create Dockerfile with HEALTHCHECK
cat > Dockerfile << 'EOF'
FROM python:3.11-slim
WORKDIR /app
COPY app.py .
RUN pip install flask
HEALTHCHECK --interval=10s --timeout=5s --start-period=40s --retries=3 \
  CMD curl -f http://localhost:5000/health || exit 1
CMD ["python", "app.py"]
EOF

# Build the image
docker build -t myapp:health .

# Expected output:
# Successfully tagged myapp:health

# Run container with health check
docker run -d --name myapp-container myapp:health

# Expected output:
# <container-id>

# Check health status immediately
docker inspect myapp-container | grep -A 5 "Health"

# Expected output:
# "Health": {
#   "Status": "starting",
#   "FailingStreak": 0,
#   "Runs": []
# }

# Wait 40 seconds for app to start, then check again
sleep 45
docker inspect myapp-container | grep -A 5 "Health"

# Expected output:
# "Status": "healthy"

# Cleanup
docker stop myapp-container
docker rm myapp-container
docker rmi myapp:health
rm -f app.py Dockerfile
```

### Explanation

- `HEALTHCHECK`: Tells Docker how to check if container is healthy
- `--interval=10s`: Check health every 10 seconds
- `--timeout=5s`: Wait max 5 seconds for check to complete
- `--start-period=40s`: Give container 40 seconds to start before health checks count
- `--retries=3`: Mark unhealthy after 3 consecutive failures
- Health check uses `curl` to test `/health` endpoint
- Responds with exit code 0 (success) or 1 (failure)
- Docker automatically monitors and reports status

---

## Exercise 2: Test health check

**Objective**: Verify health check status using `docker inspect` with health filters.

### Commands

```bash
# Start a container with health check (using image from Exercise 1)
docker run -d --name web-app myapp:health

# Expected output:
# <container-id>

# View complete health status
docker inspect web-app --format='{{json .State.Health}}' | jq .

# Expected output:
# {
#   "Status": "starting",
#   "FailingStreak": 0,
#   "Runs": []
# }

# Wait for application to start
sleep 45

# Check health status again
docker inspect web-app --format='{{json .State.Health}}' | jq .

# Expected output:
# {
#   "Status": "healthy",
#   "FailingStreak": 0,
#   "Runs": [
#     {
#       "Start": "2026-01-15T...",
#       "End": "2026-01-15T...",
#       "ExitCode": 0,
#       "Output": ""
#     }
#   ]
# }

# Query just the status
docker inspect web-app --format='{{.State.Health.Status}}'

# Expected output:
# healthy

# View all health check runs
docker inspect web-app --format='{{json .State.Health.Runs}}' | jq .

# Expected output:
# Shows all health check executions with timestamps and exit codes

# Check if specific container is healthy
if [ "$(docker inspect web-app --format='{{.State.Health.Status}}')" = "healthy" ]; then
  echo "Container is healthy"
else
  echo "Container is NOT healthy"
fi

# Expected output:
# Container is healthy

# Get failing streak count
docker inspect web-app --format='{{.State.Health.FailingStreak}}'

# Expected output:
# 0 (no failures)

# Cleanup
docker stop web-app
docker rm web-app
```

### Explanation

- `docker inspect`: Shows detailed container information including health status
- `--format`: Extract specific fields using Go template syntax
- Health status can be: "starting", "healthy", or "unhealthy"
- FailingStreak: Number of consecutive failed health checks
- Runs: Array of all health check executions with results
- Useful for monitoring and automation (restart unhealthy containers)
- Can integrate with orchestration tools (Swarm, Kubernetes)

---

## Exercise 3: Configure auto-restart

**Objective**: Use `--restart=unless-stopped` to ensure containers restart on failure.

### Commands

```bash
# Create a simple app that crashes after 5 seconds
cat > crash_app.py << 'EOF'
import time
import sys

print("Application started")
time.sleep(5)
print("Crashing now...")
sys.exit(1)
EOF

# Create Dockerfile
cat > Dockerfile << 'EOF'
FROM python:3.11-slim
WORKDIR /app
COPY crash_app.py .
CMD ["python", "crash_app.py"]
EOF

# Build image
docker build -t crashapp .

# Expected output:
# Successfully tagged crashapp

# Start container WITHOUT restart policy
echo "=== Testing without restart policy ==="
docker run -d --name no-restart crashapp

# Expected output:
# <container-id>

# Check status immediately
docker ps | grep no-restart

# Expected output:
# Shows running container

# Wait for crash
sleep 10

# Check status after crash
docker ps | grep no-restart

# Expected output:
# Container is gone (not running)

# Check if it stopped
docker ps -a | grep no-restart

# Expected output:
# Shows container with "Exited" status

# Clean up
docker rm no-restart

# Start container WITH --restart=unless-stopped
echo "=== Testing with --restart=unless-stopped ==="
docker run -d --restart=unless-stopped --name with-restart crashapp

# Expected output:
# <container-id>

# Check status after crash (wait 10+ seconds)
sleep 15
docker ps | grep with-restart

# Expected output:
# Container is running (restarted automatically)

# View restart count
docker inspect with-restart --format='{{.RestartCount}}'

# Expected output:
# 1 (or higher if time has passed)

# Manually stop the container
docker stop with-restart

# Expected output:
# with-restart

# Verify it doesn't restart after manual stop
sleep 5
docker ps | grep with-restart

# Expected output:
# Container is NOT running (respects manual stop)

# Cleanup
docker rm with-restart
docker rmi crashapp
rm -f crash_app.py Dockerfile
```

### Explanation

- `--restart` policies control what happens when container exits:
  - `no`: Do not automatically restart (default)
  - `always`: Always restart unless explicitly stopped
  - `unless-stopped`: Restart unless manually stopped (recommended for production)
  - `on-failure`: Only restart on non-zero exit code
  - `on-failure:3`: Restart max 3 times on failure
- `unless-stopped` is ideal for production: survives Docker daemon restart
- Manual `docker stop` is respected (container won't restart)
- `RestartCount` tracks how many times container has been restarted
- Helps maintain uptime without manual intervention

---

## Exercise 4: Implement rolling updates

**Objective**: Update a service image without downtime using docker service update.

### Commands

```bash
# Initialize Docker Swarm (required for services)
docker swarm init

# Expected output:
# Swarm initialized: current node (xxxx) is now a manager.

# Create a service with initial image
docker service create \
  --name web-service \
  --replicas 3 \
  -p 80:5000 \
  myapp:v1

# Expected output:
# Service created successfully: web-service

# Check service status
docker service ls

# Expected output:
# ID        NAME          MODE        REPLICAS   IMAGE       PORTS
# xxx       web-service   replicated  3/3        myapp:v1    *:80->5000/tcp

# View running tasks
docker service ps web-service

# Expected output:
# ID       NAME              IMAGE       NODE       DESIRED STATE  CURRENT STATE
# xxx      web-service.1     myapp:v1    docker     Running        Running
# yyy      web-service.2     myapp:v1    docker     Running        Running
# zzz      web-service.3     myapp:v1    docker     Running        Running

# Perform rolling update to new image
docker service update --image myapp:v2 web-service

# Expected output:
# web-service

# Watch the update in progress
docker service ps web-service

# Expected output:
# Old tasks (v1) gradually stop, new tasks (v2) start
# At least one task always running (zero downtime)

# Verify all replicas are running new version
docker service ps web-service | grep -c "myapp:v2"

# Expected output:
# 3 (all three replicas updated)

# Check service status after update
docker service ls | grep web-service

# Expected output:
# Shows 3/3 replicas running on myapp:v2

# Update with custom update strategy
docker service update \
  --update-delay 10s \
  --update-parallelism 1 \
  --image myapp:v3 \
  web-service

# Expected output:
# web-service
# Updates 1 replica every 10 seconds

# Rollback to previous version if needed
docker service update --rollback web-service

# Expected output:
# web-service
# Service rolls back to previous image

# View service update status
docker service inspect web-service --format='{{json .UpdateStatus}}' | jq .

# Expected output:
# Shows state, completed tasks, runtime info

# Cleanup
docker service rm web-service
docker swarm leave --force
```

### Explanation

- Docker Swarm provides orchestration features (requires `docker swarm init`)
- Services maintain desired number of replicas across nodes
- Rolling updates gradually replace old tasks with new ones
- `--update-delay`: Wait between updating each task
- `--update-parallelism`: Number of tasks to update simultaneously
- Zero downtime: other replicas handle requests during update
- Load balancer distributes traffic across all replicas
- Rollback available if update fails or causes issues
- More advanced than single containers, requires Swarm mode

---

## Exercise 5: Setup centralized logging

**Objective**: Configure a container to send logs to syslog or ELK stack.

### Commands

```bash
# Setup syslog logging
# Create a simple application
cat > app.py << 'EOF'
import logging
import logging.handlers
import time

# Configure logging to send to syslog
handler = logging.handlers.SysLogHandler(address=('localhost', 514))
logger = logging.getLogger('myapp')
logger.addHandler(handler)
logger.setLevel(logging.INFO)

while True:
    logger.info("Application is running")
    time.sleep(5)
EOF

# Create Dockerfile with syslog log driver
cat > Dockerfile << 'EOF'
FROM python:3.11-slim
WORKDIR /app
COPY app.py .
CMD ["python", "app.py"]
EOF

# Build image
docker build -t myapp-logging .

# Expected output:
# Successfully tagged myapp-logging

# Run container with syslog logging
docker run -d \
  --name app-syslog \
  --log-driver syslog \
  --log-opt syslog-address=udp://localhost:514 \
  myapp-logging

# Expected output:
# <container-id>

# View logs sent to syslog
tail -f /var/log/syslog | grep myapp

# Expected output:
# Shows application logs sent to syslog

# Alternative: Use json-file driver (default) but centralize
docker run -d \
  --name app-json \
  myapp-logging

# View logs with docker logs
docker logs app-json

# Expected output:
# Application is running

# Setup ELK Stack example (Elasticsearch, Logstash, Kibana)
# Run Elasticsearch container
docker run -d \
  --name elasticsearch \
  -e "discovery.type=single-node" \
  -p 9200:9200 \
  docker.elastic.co/elasticsearch/elasticsearch:7.14.0

# Expected output:
# <container-id>

# Run Logstash container (simplified)
docker run -d \
  --name logstash \
  -p 5000:5000/udp \
  docker.elastic.co/logstash/logstash:7.14.0

# Expected output:
# <container-id>

# Run container with ELK logging via syslog
docker run -d \
  --name app-elk \
  --link elasticsearch:elasticsearch \
  --link logstash:logstash \
  myapp-logging

# Expected output:
# <container-id>

# Check Elasticsearch for logs
curl -s http://localhost:9200/_search?q=myapp | jq .

# Expected output:
# JSON response with indexed logs

# View available log drivers
docker run --help | grep -A 20 "log-driver"

# Expected output:
# Lists all supported log drivers:
# syslog, splunk, awslogs, gcplogs, sumologic, etc.

# Cleanup
docker stop app-syslog app-json app-elk elasticsearch logstash 2>/dev/null
docker rm app-syslog app-json app-elk elasticsearch logstash 2>/dev/null
docker rmi myapp-logging 2>/dev/null
rm -f app.py Dockerfile
```

### Explanation

- Default logging: Docker stores logs locally in container
- Centralized logging: Aggregate logs from multiple containers
- Syslog: Traditional logging system, sends to port 514 (UDP/TCP)
- Log drivers available: syslog, splunk, awslogs, gcplogs, sumologic, etc.
- ELK Stack: Popular for centralized logging in enterprises
  - Elasticsearch: Stores and indexes logs
  - Logstash: Processes and routes logs
  - Kibana: Web UI for searching and visualizing logs
- `--log-driver`: Specify which driver to use
- `--log-opt`: Additional options for the log driver
- Important for production: track issues across containers
- Enables audit trails, troubleshooting, and compliance

---

## Exercise 6: Manage secrets

**Objective**: Use Docker secrets to manage database credentials securely.

### Commands

```bash
# Secrets only work in Swarm mode, so initialize Swarm first
docker swarm init

# Expected output:
# Swarm initialized...

# Create a secret from a file
echo "super_secret_password_123" > db_password.txt
docker secret create db_password db_password.txt

# Expected output:
# <secret-id>

# Create another secret
echo "admin" > db_username.txt
docker secret create db_username db_username.txt

# Expected output:
# <secret-id>

# List all secrets
docker secret ls

# Expected output:
# ID        NAME           DRIVER    CREATED
# xxx       db_password    -         10 seconds ago
# yyy       db_username    -         5 seconds ago

# Inspect a secret (doesn't show value)
docker secret inspect db_password

# Expected output:
# [
#   {
#     "ID": "xxx...",
#     "Name": "db_password",
#     "CreatedAt": "2026-01-15...",
#     "UpdatedAt": "2026-01-15...",
#     "Spec": {...}
#   }
# ]

# Use secrets in docker-compose
cat > docker-compose.yml << 'EOF'
version: '3.1'

services:
  web:
    image: nginx
    ports:
      - "80:80"
    secrets:
      - db_password
      - db_username
    environment:
      DB_USER_FILE: /run/secrets/db_username
      DB_PASS_FILE: /run/secrets/db_password

secrets:
  db_password:
    external: true
  db_username:
    external: true
EOF

# Deploy service using secrets
docker stack deploy -c docker-compose.yml mystack

# Expected output:
# Creating service mystack_web

# Access secret inside container
docker exec <container-id> cat /run/secrets/db_password

# Expected output:
# super_secret_password_123

# Note: Secrets are mounted at /run/secrets/<secret-name>
# Available only to containers that reference them
# Encrypted in transit and at rest

# Remove a secret (only if no service uses it)
docker service rm mystack_web

docker secret rm db_password

# Expected output:
# db_password

# Verify secret is gone
docker secret ls

# Expected output:
# ID        NAME            DRIVER    CREATED
# yyy       db_username     -         5 minutes ago

# Cleanup
docker stack rm mystack
docker secret rm db_username 2>/dev/null
docker swarm leave --force
rm -f db_password.txt db_username.txt docker-compose.yml
```

### Explanation

- Secrets: Secure way to manage sensitive data (passwords, API keys, tokens)
- Only available in Swarm mode (or Kubernetes)
- Stored encrypted, transmitted securely
- Not included in image, injected at runtime
- Mounted at `/run/secrets/<secret-name>` inside container
- Cannot be viewed via `docker inspect` or logs
- Automatically removed when service stops referencing them
- Alternative: Use environment variables or credential managers
- Production best practice: never hardcode credentials in Dockerfile
- Can also use: HashiCorp Vault, AWS Secrets Manager, etc.

---

## Exercise 7: Monitor with alerts

**Objective**: Set up monitoring to alert when CPU/memory exceeds thresholds.

### Commands

```bash
# Create a monitoring script
cat > monitor.sh << 'EOF'
#!/bin/bash

# Simple monitoring script for container resources
CONTAINER=$1
CPU_THRESHOLD=50  # percent
MEM_THRESHOLD=80  # percent

echo "Monitoring $CONTAINER..."
echo "CPU Threshold: ${CPU_THRESHOLD}%"
echo "Memory Threshold: ${MEM_THRESHOLD}%"

while true; do
  # Get current stats
  STATS=$(docker stats --no-stream --format "{{.CPUPerc}},{{.MemPerc}}" $CONTAINER)
  CPU=$(echo $STATS | cut -d',' -f1 | sed 's/%//')
  MEM=$(echo $STATS | cut -d',' -f2 | sed 's/%//')

  echo "CPU: ${CPU}% | Memory: ${MEM}%"

  # Check thresholds
  if (( $(echo "$CPU > $CPU_THRESHOLD" | bc -l) )); then
    echo "ALERT: CPU usage (${CPU}%) exceeds threshold (${CPU_THRESHOLD}%)"
  fi

  if (( $(echo "$MEM > $MEM_THRESHOLD" | bc -l) )); then
    echo "ALERT: Memory usage (${MEM}%) exceeds threshold (${MEM_THRESHOLD}%)"
  fi

  sleep 5
done
EOF

chmod +x monitor.sh

# Run container with resource limits
docker run -d \
  --name monitored-app \
  --memory=512m \
  --cpus=1 \
  myapp

# Expected output:
# <container-id>

# Start monitoring
./monitor.sh monitored-app

# Expected output:
# Monitoring monitored-app...
# CPU Threshold: 50%
# Memory Threshold: 80%
# CPU: 5% | Memory: 20%
# CPU: 5% | Memory: 20%
# ...

# Use docker stats directly for real-time monitoring
docker stats --format "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}" monitored-app

# Expected output:
# CONTAINER       CPU %    MEM USAGE
# monitored-app   5%       105MiB

# Use Prometheus for production monitoring (example setup)
cat > prometheus.yml << 'EOF'
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'docker'
    static_configs:
      - targets: ['localhost:9323']
EOF

# Run Prometheus container
docker run -d \
  --name prometheus \
  -v $(pwd)/prometheus.yml:/etc/prometheus/prometheus.yml \
  -p 9090:9090 \
  prom/prometheus

# Expected output:
# <container-id>

# Access Prometheus UI
echo "Prometheus available at http://localhost:9090"

# Query CPU usage in Prometheus
curl -s 'http://localhost:9090/api/v1/query?query=container_cpu_usage_seconds_total' | jq .

# Expected output:
# JSON response with metrics

# Setup Alertmanager for notifications
cat > alertmanager.yml << 'EOF'
global:
  resolve_timeout: 5m

route:
  receiver: 'default'

receivers:
- name: 'default'
  webhook_configs:
  - url: 'http://localhost:5001/'
EOF

# Configure alert rules
cat > rules.yml << 'EOF'
groups:
- name: container_alerts
  rules:
  - alert: HighCPUUsage
    expr: container_cpu_usage_seconds_total > 0.5
    for: 1m
    annotations:
      summary: "High CPU usage"
EOF

# Cleanup
docker stop monitored-app prometheus 2>/dev/null
docker rm monitored-app prometheus 2>/dev/null
rm -f monitor.sh prometheus.yml alertmanager.yml rules.yml
```

### Explanation

- Monitoring: Track container resource usage over time
- `docker stats`: Built-in tool for real-time resource monitoring
- CPU/Memory thresholds: Customize based on application needs
- Prometheus: Popular open-source monitoring system
- Alertmanager: Sends alerts when thresholds exceeded
- Alert channels: Email, Slack, PagerDuty, webhooks, etc.
- Baselines: Establish normal usage patterns first
- Production setup: Use managed services (CloudWatch, Datadog, New Relic)
- Historical data: Essential for capacity planning
- Proactive alerts: Catch issues before affecting users

---

## Exercise 8: Create backup strategy

**Objective**: Implement volume backups for persistent data.

### Commands

```bash
# Create a volume
docker volume create mydata

# Expected output:
# mydata

# Run container with volume
docker run -d \
  --name db-container \
  -v mydata:/data \
  postgres:14

# Expected output:
# <container-id>

# Add some data to the volume
docker exec db-container psql -U postgres -c "CREATE TABLE test (id INT, name TEXT);"

# Expected output:
# CREATE TABLE

# Backup volume using tar
echo "=== Method 1: Backup using tar ==="
docker run --rm \
  -v mydata:/data \
  -v $(pwd)/backups:/backup \
  busybox \
  tar czf /backup/mydata-backup-$(date +%Y%m%d).tar.gz /data

# Expected output:
# backup-20260115.tar.gz created

# Verify backup file
ls -lh backups/

# Expected output:
# -rw-r--r-- 1 user user 1.2M Jan 15 12:00 mydata-backup-20260115.tar.gz

# Backup method 2: Using docker cp
echo "=== Method 2: Backup using docker cp ==="
docker run -d \
  --name backup-helper \
  -v mydata:/data \
  busybox sleep 3600

# Copy data out
docker cp backup-helper:/data backups/data-copy-$(date +%Y%m%d)

# Expected output:
# Successfully copied...

# Cleanup helper
docker stop backup-helper
docker rm backup-helper

# Restore from backup
echo "=== Restore from backup ==="

# Create new volume
docker volume create restored-data

# Restore using tar
docker run --rm \
  -v restored-data:/data \
  -v $(pwd)/backups:/backup \
  busybox \
  sh -c "cd /data && tar xzf /backup/mydata-backup-20260115.tar.gz --strip-components=1"

# Expected output:
# Files restored

# Verify restored volume
docker run --rm \
  -v restored-data:/data \
  busybox \
  ls -la /data

# Expected output:
# Shows restored files

# Automated backup script
cat > backup.sh << 'EOF'
#!/bin/bash

VOLUME_NAME="mydata"
BACKUP_DIR="./backups"
RETENTION_DAYS=7

# Create backup
mkdir -p $BACKUP_DIR
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

docker run --rm \
  -v $VOLUME_NAME:/data \
  -v $BACKUP_DIR:/backup \
  busybox \
  tar czf /backup/backup-$TIMESTAMP.tar.gz /data

echo "Backup created: backup-$TIMESTAMP.tar.gz"

# Remove old backups
find $BACKUP_DIR -name "backup-*.tar.gz" -mtime +$RETENTION_DAYS -delete

echo "Old backups removed"
EOF

chmod +x backup.sh

# Schedule backup (using cron)
echo "0 2 * * * /path/to/backup.sh" | crontab -

# Expected output:
# Cron job scheduled (runs daily at 2 AM)

# Verify backup worked
./backup.sh

# Expected output:
# Backup created: backup-20260115_120000.tar.gz

# Cleanup
docker stop db-container 2>/dev/null
docker rm db-container 2>/dev/null
docker volume rm mydata restored-data 2>/dev/null
rm -f backup.sh backups/*.tar.gz
```

### Explanation

- Persistent data: Stored in volumes, separate from container lifecycle
- Backup: Essential for production databases and important data
- Methods: tar, rsync, native database tools (mysqldump, pg_dump)
- Retention policy: Keep backups for X days, delete old ones
- Automation: Schedule regular backups with cron
- Testing: Periodically restore backups to verify integrity
- Off-site: Store backups on separate server or cloud storage
- Encryption: Encrypt sensitive data before storing backups
- RTO/RPO: Define acceptable recovery time and data loss
- Disaster recovery: Document restoration procedures

---

## Exercise 9: Load balance traffic

**Objective**: Use docker-compose with multiple replicas and external load balancer.

### Commands

```bash
# Create multiple instances of an application
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  web1:
    image: nginx
    container_name: web1
    ports:
      - "8001:80"
    volumes:
      - ./html1:/usr/share/nginx/html
    networks:
      - webnet

  web2:
    image: nginx
    container_name: web2
    ports:
      - "8002:80"
    volumes:
      - ./html2:/usr/share/nginx/html
    networks:
      - webnet

  web3:
    image: nginx
    container_name: web3
    ports:
      - "8003:80"
    volumes:
      - ./html3:/usr/share/nginx/html
    networks:
      - webnet

  loadbalancer:
    image: haproxy:2.0
    container_name: loadbalancer
    ports:
      - "80:80"
    volumes:
      - ./haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg
    depends_on:
      - web1
      - web2
      - web3
    networks:
      - webnet

networks:
  webnet:
    driver: bridge
EOF

# Create web content for each instance
mkdir -p html1 html2 html3

echo "<h1>Web Server 1</h1>" > html1/index.html
echo "<h1>Web Server 2</h1>" > html2/index.html
echo "<h1>Web Server 3</h1>" > html3/index.html

# Create HAProxy configuration
cat > haproxy.cfg << 'EOF'
global
  log stdout local0
  maxconn 1024

defaults
  log global
  mode http
  timeout connect 5000ms
  timeout client 50000ms
  timeout server 50000ms

frontend http_front
  bind *:80
  default_backend http_back

backend http_back
  balance roundrobin
  server web1 web1:80 check
  server web2 web2:80 check
  server web3 web3:80 check
EOF

# Start the stack
docker-compose up -d

# Expected output:
# Creating web1 ... done
# Creating web2 ... done
# Creating web3 ... done
# Creating loadbalancer ... done

# Verify all services are running
docker-compose ps

# Expected output:
# NAME             COMMAND             SERVICE        STATUS
# web1             nginx -g daemon     web1           Up 10 seconds
# web2             nginx -g daemon     web2           Up 10 seconds
# web3             nginx -g daemon     web3           Up 10 seconds
# loadbalancer     /docker-entrypoint  loadbalancer   Up 10 seconds

# Test load balancing
echo "=== Testing load balancer ==="
for i in {1..6}; do
  curl -s http://localhost/ | grep -o "Web Server [0-9]"
done

# Expected output:
# Web Server 1
# Web Server 2
# Web Server 3
# Web Server 1
# Web Server 2
# Web Server 3
# (Requests distributed round-robin across backends)

# Stop one web server
docker-compose stop web2

# Expected output:
# Stopping loadbalancer ... done
# Stopping web2 ... done

# Test load balancer with reduced capacity
echo "=== Testing with web2 down ==="
for i in {1..4}; do
  curl -s http://localhost/ | grep -o "Web Server [0-9]"
done

# Expected output:
# Web Server 1
# Web Server 3
# Web Server 1
# Web Server 3
# (Requests only to web1 and web3)

# Restart web2
docker-compose up -d web2

# Expected output:
# Starting web2 ... done

# Alternative: Use Nginx as load balancer
cat > nginx.conf << 'EOF'
upstream backends {
  server web1:80;
  server web2:80;
  server web3:80;
}

server {
  listen 80;
  location / {
    proxy_pass http://backends;
  }
}
EOF

# View HAProxy stats page
echo "HAProxy stats available at http://localhost:8404/stats"

# Cleanup
docker-compose down

# Expected output:
# Removing loadbalancer ... done
# Removing web3 ... done
# Removing web2 ... done
# Removing web1 ... done
# Removing network ... done

rm -rf html1 html2 html3 haproxy.cfg nginx.conf docker-compose.yml
```

### Explanation

- Load balancer: Distributes traffic across multiple backend servers
- Round-robin: Each request goes to next server in rotation
- Health checks: HAProxy/Nginx detect and skip unhealthy backends
- Horizontal scaling: Add more servers without changing client code
- Docker Compose: Simplifies multi-container setup
- Networks: Services communicate within internal bridge network
- Port mapping: Expose services to host/external networks
- HAProxy: Powerful open-source load balancer
- Nginx: Web server with load balancing capabilities
- Production: Use cloud load balancers (AWS ELB, Azure LB, GCP LB)

---

## Exercise 10: Plan incident response

**Objective**: Document procedures for common production failures.

### Commands

```bash
# Create incident response runbook
cat > INCIDENT_RESPONSE.md << 'EOF'
# Incident Response Runbook

## 1. Container Crashes (Exit Code > 0)

**Detection**: `docker ps` shows container not running, or health check fails

**Investigation**:
```bash
# Check container logs
docker logs <container-id> --tail=100

# Check exit code
docker inspect <container-id> --format='{{.State.ExitCode}}'

# Common exit codes:
# 0 = success
# 1 = general error
# 2 = misuse of shell command
# 126 = cannot execute command
# 137 = killed by SIGKILL (out of memory)
# 139 = killed by SIGSEGV (segmentation fault)
```

**Resolution**:
- Check application logs for errors
- Verify environment variables are set correctly
- Check resource limits (memory, disk)
- Verify all dependent services are running
- Restart container: `docker restart <container-id>`
- If persistent, rollback to previous image version

---

## 2. Out of Memory (OOM) Issues

**Detection**: Container killed, exit code 137, "Cannot allocate memory" in logs

**Investigation**:
```bash
# Check memory usage
docker stats <container-id>

# Check memory limits
docker inspect <container-id> --format='{{.HostConfig.Memory}}'

# Monitor over time
docker stats --format "table {{.Container}}\t{{.MemUsage}}"
```

**Resolution**:
- Increase memory limit: `docker update --memory=1g <container-id>`
- Optimize application: reduce caching, improve algorithm efficiency
- Restart with more memory: `docker stop && docker run --memory=1g`
- Implement health check to restart on memory stress
- Scale horizontally: distribute load across multiple containers

---

## 3. Disk Space Issues

**Detection**: "No space left on device", containers fail to start

**Investigation**:
```bash
# Check disk usage
df -h

# Check Docker disk usage
docker system df

# Find large containers/images
docker images --format "table {{.Repository}}:{{.Tag}}\t{{.Size}}" | sort -k3 -h

# Find large volumes
docker volume ls --format "{{.Name}}" | xargs -I {} du -sh /var/lib/docker/volumes/{}/
```

**Resolution**:
- Remove unused images: `docker image prune -a`
- Remove unused volumes: `docker volume prune`
- Remove stopped containers: `docker container prune`
- Clean build cache: `docker builder prune`
- Expand disk partition (infrastructure task)
- Implement log rotation for containers

---

## 4. Network Connectivity Issues

**Detection**: Services cannot communicate, "Connection refused" errors

**Investigation**:
```bash
# Check network status
docker network ls
docker network inspect <network-name>

# Check container network settings
docker inspect <container-id> | grep -A 10 "Networks"

# Test connectivity
docker exec <container1> ping <container2>
docker exec <container1> curl http://<container2>:port

# Check DNS resolution
docker exec <container-id> cat /etc/resolv.conf
docker exec <container-id> nslookup <service-name>
```

**Resolution**:
- Verify containers are on same network
- Check firewall rules and port mappings
- Verify service names in docker-compose
- Restart networking: `docker network disconnect && docker network connect`
- Check DNS (use default bridge vs custom network)
- For host networking: verify no port conflicts

---

## 5. High CPU Usage

**Detection**: `docker stats` shows CPU % > 80%, server load high

**Investigation**:
```bash
# Monitor CPU usage
docker stats <container-id>

# Check processes inside container
docker exec <container-id> top
docker exec <container-id> ps aux

# Check for infinite loops or stuck processes
docker logs <container-id> --tail=50
```

**Resolution**:
- Limit CPU: `docker update --cpus=0.5 <container-id>`
- Kill specific process: `docker exec <container-id> kill -9 <pid>`
- Optimize application code (check for inefficient loops)
- Scale horizontally: run multiple replicas
- Reduce workload or increase resources
- Profile application using APM tools

---

## 6. Database Connection Issues

**Detection**: "Connection refused", "Too many connections" errors

**Investigation**:
```bash
# Check if database is running
docker ps | grep database

# Check database logs
docker logs <db-container> --tail=50

# Test connectivity from app container
docker exec <app-container> nc -zv <db-host> 5432

# Check connection count (for PostgreSQL)
docker exec <db-container> psql -U postgres -c "SELECT count(*) FROM pg_stat_activity;"
```

**Resolution**:
- Verify database container is running
- Check network connectivity between app and database
- Increase max connections: adjust database config
- Clear stale connections: restart database
- Add connection pooling (PgBouncer, ProxySQL)
- Check for connection leaks in application

---

## 7. Service Restart Loop

**Detection**: Container repeatedly restarts, health check fails

**Investigation**:
```bash
# Check restart count
docker inspect <container-id> --format='{{.RestartCount}}'

# View restart events
docker events --filter "container=<name>"

# Check health status
docker inspect <container-id> | grep -A 10 "Health"

# Check logs for startup errors
docker logs <container-id> | head -20
```

**Resolution**:
- Extend start-period: give more time for startup
- Fix startup issue: check dependencies, configuration
- Temporarily disable restart: `docker stop <container-id>`
- Fix underlying issue, then manually restart
- Increase timeouts in health checks
- Verify all dependencies are running

---

## General Troubleshooting Commands

```bash
# Get full container information
docker inspect <container-id>

# View real-time logs
docker logs -f <container-id>

# Execute command in running container
docker exec -it <container-id> /bin/bash

# Check resource limits
docker stats <container-id>

# Get event stream
docker events --filter "container=<name>"

# Clean up Docker system
docker system prune -a

# Check Docker daemon logs (Linux)
journalctl -u docker
systemctl status docker
```

---

## Prevention Best Practices

1. **Resource Limits**: Always set memory and CPU limits
2. **Health Checks**: Implement for all services
3. **Logging**: Centralize logs for easy analysis
4. **Monitoring**: Set up alerts for critical metrics
5. **Testing**: Load test before production deployment
6. **Documentation**: Keep runbooks up to date
7. **Automation**: Auto-remediate common issues
8. **Backups**: Regular backups of critical data
9. **Updates**: Keep images and Docker daemon updated
10. **Security**: Scan images for vulnerabilities
EOF

cat INCIDENT_RESPONSE.md

# Simulate container crash
echo "=== Simulating container crash ==="
docker run -d \
  --name crash-demo \
  alpine sh -c "exit 1"

# Check status
docker ps -a | grep crash-demo

# Expected output:
# Shows container in Exited state

# View exit code
docker inspect crash-demo --format='{{.State.ExitCode}}'

# Expected output:
# 1

# View logs
docker logs crash-demo

# Expected output:
# (empty or error message)

# Simulate OOM
echo "=== Simulating OOM scenario ==="
docker run -d \
  --name oom-demo \
  --memory=64m \
  alpine sh -c "dd if=/dev/zero bs=1M count=100 of=/dev/null"

# Monitor memory
sleep 5
docker stats --no-stream oom-demo

# Expected output:
# Shows memory usage near limit

# Check if killed
docker ps -a | grep oom-demo

# Expected output:
# Container will be killed when exceeding limit

# Cleanup
docker rm crash-demo oom-demo 2>/dev/null
rm -f INCIDENT_RESPONSE.md
```

### Explanation

- Incident response: Documented procedures for common failures
- Runbook: Step-by-step guide for troubleshooting and resolution
- Log analysis: Always check logs first for error details
- Root cause analysis: Understand why failure occurred
- Preventive measures: Implement safeguards to prevent recurrence
- Alerting: Set up early warning systems
- Recovery procedures: Document steps to restore service
- Postmortem: Review incident after resolution
- Continuous improvement: Update procedures based on learnings
- Automation: Automate responses to common issues where possible

