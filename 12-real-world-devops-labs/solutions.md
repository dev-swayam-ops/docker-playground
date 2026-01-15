# Solutions: Real-World DevOps Labs

## Exercise 1: Multi-container setup

**Objective**: Build a docker-compose file with frontend (nginx), backend (Node/Python), and database (PostgreSQL).

### Commands

```bash
# Create project directory
mkdir myapp
cd myapp

# Create frontend (Nginx)
mkdir frontend
cat > frontend/Dockerfile << 'EOF'
FROM nginx:latest
COPY nginx.conf /etc/nginx/nginx.conf
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
EOF

# Create simple nginx config
cat > frontend/nginx.conf << 'EOF'
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log warn;
pid /var/run/nginx.pid;

events {
    worker_connections 1024;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';
    access_log /var/log/nginx/access.log main;
    sendfile on;
    keepalive_timeout 65;

    server {
        listen 80;
        location / {
            root /usr/share/nginx/html;
            index index.html;
        }
        location /api {
            proxy_pass http://api:5000;
        }
    }
}
EOF

# Create simple HTML
cat > frontend/index.html << 'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>My App</title>
</head>
<body>
    <h1>Welcome to My App</h1>
    <p>Frontend is running!</p>
</body>
</html>
EOF

# Create backend (Python Flask)
mkdir backend
cat > backend/Dockerfile << 'EOF'
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY app.py .
CMD ["python", "app.py"]
EOF

cat > backend/requirements.txt << 'EOF'
flask==2.3.0
psycopg2-binary==2.9.6
EOF

cat > backend/app.py << 'EOF'
from flask import Flask, jsonify
import psycopg2
import os

app = Flask(__name__)

@app.route('/')
def hello():
    return jsonify({"message": "Backend API is running!"})

@app.route('/api/data')
def get_data():
    try:
        conn = psycopg2.connect(
            host=os.getenv('DB_HOST'),
            database=os.getenv('DB_NAME'),
            user=os.getenv('DB_USER'),
            password=os.getenv('DB_PASSWORD')
        )
        cur = conn.cursor()
        cur.execute("SELECT version();")
        db_version = cur.fetchone()
        cur.close()
        conn.close()
        return jsonify({
            "message": "Connected to database",
            "db_version": db_version[0]
        })
    except Exception as e:
        return jsonify({"error": str(e)}), 500

@app.route('/health')
def health():
    return jsonify({"status": "healthy"}), 200

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
EOF

# Create main docker-compose.yml
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    container_name: myapp-frontend
    ports:
      - "80:80"
    depends_on:
      - api
    networks:
      - frontend-net
      - bridge

  api:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: myapp-api
    ports:
      - "5000:5000"
    environment:
      - DB_HOST=db
      - DB_NAME=myapp
      - DB_USER=postgres
      - DB_PASSWORD=password123
    depends_on:
      - db
    networks:
      - backend-net
      - bridge

  db:
    image: postgres:14
    container_name: myapp-db
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_DB=myapp
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password123
    volumes:
      - db_data:/var/lib/postgresql/data
    networks:
      - backend-net

volumes:
  db_data:

networks:
  frontend-net:
    driver: bridge
  backend-net:
    driver: bridge
  bridge:
    driver: bridge
EOF

# Build and start services
docker-compose build

# Expected output:
# Building api ...
# Building frontend ...
# Successfully tagged myapp_api:latest
# Successfully tagged myapp_frontend:latest

# Start services
docker-compose up -d

# Expected output:
# Creating myapp-db ... done
# Creating myapp-api ... done
# Creating myapp-frontend ... done

# Verify services are running
docker-compose ps

# Expected output:
# NAME               COMMAND                  SERVICE    STATUS
# myapp-frontend     "nginx -g daemon off"    frontend   Up 2 seconds
# myapp-api          "python app.py"          api        Up 1 second
# myapp-db           "postgres"               db         Up 2 seconds

# Test frontend
curl http://localhost

# Expected output:
# Welcome to My App

# Test backend
curl http://localhost:5000

# Expected output:
# {"message": "Backend API is running!"}

# Cleanup
docker-compose down
```

### Explanation

- `docker-compose.yml`: Defines multiple services in one file
- Services: Frontend (Nginx), API (Python), Database (PostgreSQL)
- `build`: Build images from Dockerfile instead of pulling
- `ports`: Expose services to host (external access)
- `environment`: Pass configuration to containers
- `depends_on`: Specify service startup order
- `volumes`: Persist database data
- `networks`: Isolate or connect services
- Each service gets a DNS name (frontend, api, db)
- Services communicate using internal Docker network

---

## Exercise 2: Environment management

**Objective**: Use multiple docker-compose files (.env, docker-compose.yml, docker-compose.prod.yml) for dev/prod.

### Commands

```bash
# Create environment file (.env)
cat > .env << 'EOF'
COMPOSE_PROJECT_NAME=myapp
DB_USER=postgres
DB_PASSWORD=password123
DB_NAME=myapp
API_PORT=5000
EOF

# Create development docker-compose file
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  db:
    image: postgres:14
    environment:
      POSTGRES_DB: ${DB_NAME}
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    ports:
      - "5432:5432"
    volumes:
      - db_data:/var/lib/postgresql/data

  api:
    build: ./backend
    ports:
      - "${API_PORT}:5000"
    environment:
      DEBUG: "true"
      DB_HOST: db
      DB_USER: ${DB_USER}
      DB_PASSWORD: ${DB_PASSWORD}
    depends_on:
      - db

  frontend:
    build: ./frontend
    ports:
      - "80:80"
    depends_on:
      - api

volumes:
  db_data:
EOF

# Create production override file
cat > docker-compose.prod.yml << 'EOF'
version: '3.8'

services:
  db:
    restart: unless-stopped
    environment:
      POSTGRES_INITDB_ARGS: "-c shared_buffers=256MB"

  api:
    restart: unless-stopped
    environment:
      DEBUG: "false"
    deploy:
      resources:
        limits:
          cpus: '1'
          memory: 512M

  frontend:
    restart: unless-stopped
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 256M
EOF

# Run development stack
echo "=== Development Stack ==="
docker-compose up -d

# Expected output:
# Creating myapp_db_1 ... done
# Creating myapp_api_1 ... done
# Creating myapp_frontend_1 ... done

# Verify dev config
docker-compose config | grep -A 5 "DEBUG"

# Expected output:
# Shows DEBUG=true

# Stop development
docker-compose down

# Run production stack
echo "=== Production Stack ==="
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d

# Expected output:
# Creating myapp_db_1 ... done
# Creating myapp_api_1 ... done
# Creating myapp_frontend_1 ... done

# Verify prod config
docker-compose -f docker-compose.yml -f docker-compose.prod.yml config | grep -A 5 "DEBUG"

# Expected output:
# Shows DEBUG=false

# Check resource limits in production
docker-compose -f docker-compose.yml -f docker-compose.prod.yml ps

# Expected output:
# Shows services with production limits applied

# View final merged configuration
docker-compose -f docker-compose.yml -f docker-compose.prod.yml config

# Expected output:
# Merged configuration with all overrides applied

# Cleanup
docker-compose -f docker-compose.yml -f docker-compose.prod.yml down
```

### Explanation

- `.env` file: Store environment variables used across compose files
- Variables referenced as `${VARIABLE_NAME}`
- `docker-compose.yml`: Base configuration for development
- `docker-compose.prod.yml`: Override/extend for production
- Multiple compose files merged when specified with `-f` flag
- Production overrides: restart policies, resource limits, logging
- Development config: debug mode, relaxed limits for faster iteration
- Separation of concerns: same service definition, different configs
- Best practice: Use .env for secrets in development only
- Production: Use Docker secrets or environment management tools

---

## Exercise 3: Build and push pipeline

**Objective**: Create a Dockerfile, build locally, tag, and push to Docker Hub (simulating CI/CD).

### Commands

```bash
# Create Dockerfile
cat > Dockerfile << 'EOF'
FROM python:3.11-slim

WORKDIR /app

# Copy requirements and install
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY app.py .

# Expose port
EXPOSE 5000

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=40s --retries=3 \
    CMD python -c "import requests; requests.get('http://localhost:5000/health')"

# Run application
CMD ["python", "app.py"]
EOF

# Create application
cat > app.py << 'EOF'
from flask import Flask, jsonify
app = Flask(__name__)

@app.route('/health')
def health():
    return jsonify({"status": "healthy"}), 200

@app.route('/')
def hello():
    return jsonify({"message": "Hello from app!"})

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
EOF

cat > requirements.txt << 'EOF'
flask==2.3.0
EOF

# Step 1: Build image locally
echo "=== Step 1: Build image ==="
docker build -t myapp:1.0 .

# Expected output:
# Sending build context to Docker daemon ...
# Step 1/7 : FROM python:3.11-slim
# ...
# Successfully tagged myapp:1.0

# Verify image was created
docker images | grep myapp

# Expected output:
# myapp    1.0    abc123    5 minutes ago    200MB

# Step 2: Test locally
echo "=== Step 2: Test locally ==="
docker run -d -p 5000:5000 --name test-app myapp:1.0

# Expected output:
# <container-id>

# Wait for startup
sleep 5

# Test health check
curl http://localhost:5000/health

# Expected output:
# {"status": "healthy"}

# Test main endpoint
curl http://localhost:5000

# Expected output:
# {"message": "Hello from app!"}

# Stop test container
docker stop test-app
docker rm test-app

# Step 3: Tag image for Docker Hub
echo "=== Step 3: Tag image ==="
# Replace 'username' with your actual Docker Hub username
docker tag myapp:1.0 username/myapp:1.0
docker tag myapp:1.0 username/myapp:latest

# Verify tags
docker images | grep myapp

# Expected output:
# myapp              1.0      abc123    10 minutes ago   200MB
# username/myapp     1.0      abc123    10 minutes ago   200MB
# username/myapp     latest   abc123    10 minutes ago   200MB

# Step 4: Login to Docker Hub (if not already logged in)
echo "=== Step 4: Login to Docker Hub ==="
docker login

# Expected output:
# Login with your Docker Hub credentials
# Login Succeeded

# Step 5: Push to Docker Hub
echo "=== Step 5: Push to registry ==="
docker push username/myapp:1.0

# Expected output:
# The push refers to repository [docker.io/username/myapp]
# <layers pushed>
# 1.0: digest: sha256:... size: 1234

docker push username/myapp:latest

# Expected output:
# Latest pushed

# Verify image on Docker Hub
echo "Image is now available at: https://hub.docker.com/r/username/myapp"

# Step 6: Pull and run from Docker Hub
echo "=== Step 6: Pull from registry ==="
docker pull username/myapp:1.0

# Expected output:
# 1.0: Pulling from username/myapp
# ...
# Digest: sha256:...
# Status: Downloaded newer image for username/myapp:1.0

# Run pulled image
docker run -d -p 5001:5000 --name deployed-app username/myapp:1.0

# Expected output:
# <container-id>

# Test deployed app
sleep 3
curl http://localhost:5001/health

# Expected output:
# {"status": "healthy"}

# Cleanup
docker stop deployed-app
docker rm deployed-app
docker logout
```

### Explanation

- Build: Create Docker image from Dockerfile using `docker build`
- Tag: Label image with version/registry for organization
- Push: Upload image to registry (Docker Hub, ECR, GCR, etc.)
- CI/CD: Automate this process in GitHub Actions, GitLab CI, Jenkins
- Version tags: Track multiple versions of same image
- Latest tag: Points to most recent production release
- Registry: Central repository for storing and distributing images
- Security scan: Most registries scan for vulnerabilities
- Image naming: `registry/username/imagename:tag`

---

## Exercise 4: Health check integration

**Objective**: Add health checks to all services and verify them reporting correct status.

### Commands

```bash
# Create services with health checks
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  frontend:
    image: nginx
    ports:
      - "80:80"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/"]
      interval: 10s
      timeout: 5s
      retries: 3
      start_period: 20s

  api:
    image: python:3.11-slim
    command: python -m http.server 5000
    ports:
      - "5000:5000"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:5000/"]
      interval: 10s
      timeout: 5s
      retries: 3
      start_period: 20s

  db:
    image: postgres:14
    environment:
      POSTGRES_PASSWORD: password
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s
EOF

# Start services
docker-compose up -d

# Expected output:
# Creating <project>_frontend_1 ... done
# Creating <project>_api_1 ... done
# Creating <project>_db_1 ... done

# Wait for services to start
sleep 30

# Check health status
echo "=== Frontend health ==="
docker-compose ps | grep frontend

# Expected output:
# Shows health status: healthy

echo "=== API health ==="
docker-compose ps | grep api

# Expected output:
# Shows health status: healthy

echo "=== Database health ==="
docker-compose ps | grep db

# Expected output:
# Shows health status: healthy

# Detailed health information
echo "=== Detailed health info ==="
docker-compose exec db pg_isready -U postgres

# Expected output:
# accepting connections

# Query health status via docker inspect
docker inspect $(docker-compose ps -q frontend) --format='{{json .State.Health}}' | jq .

# Expected output:
# {
#   "Status": "healthy",
#   "FailingStreak": 0,
#   "Runs": [...]
# }

# View all health check runs
docker inspect $(docker-compose ps -q db) --format='{{json .State.Health.Runs}}' | jq .[-1]

# Expected output:
# Latest health check result

# Monitor health over time
for i in {1..5}; do
  echo "Sample $i at $(date +%H:%M:%S)"
  docker-compose ps | grep -E "frontend|api|db"
  sleep 5
done

# Expected output:
# Health status tracked over time

# Simulate unhealthy service by stopping it
docker-compose stop api

# Expected output:
# Stopping <project>_api_1 ... done

# Wait for health check to detect failure
sleep 15

# Check status
docker-compose ps | grep api

# Expected output:
# Shows api as unhealthy/exited

# Restart and verify recovery
docker-compose up -d api

# Expected output:
# Starting <project>_api_1 ... done

# Wait and check recovery
sleep 25
docker-compose ps | grep api

# Expected output:
# Shows api as healthy again

# Cleanup
docker-compose down
```

### Explanation

- Health check: Tells Docker if container is working properly
- Test: Command to run (curl, pg_isready, custom script)
- Interval: How often to check (default 30s)
- Timeout: Max time to wait for response (default 5s)
- Retries: Failures before marking unhealthy (default 3)
- Start period: Grace period before checks count (default 0s)
- Status: healthy, unhealthy, starting
- Docker Compose: Shows health in `docker-compose ps` output
- Auto-restart: Can trigger container restart on unhealthy status
- Essential for production reliability and monitoring

---

## Exercise 5: Networking verification

**Objective**: Verify services can communicate using DNS names (frontend to api to db).

### Commands

```bash
# Create docker-compose with multiple services
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  frontend:
    image: curlimages/curl
    command: sleep 3600
    networks:
      - app-net
    depends_on:
      - api

  api:
    image: curlimages/curl
    command: sleep 3600
    networks:
      - app-net
    depends_on:
      - db

  db:
    image: postgres:14
    environment:
      POSTGRES_PASSWORD: password
    networks:
      - app-net

networks:
  app-net:
    driver: bridge
EOF

# Start services
docker-compose up -d

# Expected output:
# Creating <project>_frontend_1 ... done
# Creating <project>_api_1 ... done
# Creating <project>_db_1 ... done

# Test DNS resolution from frontend
echo "=== Frontend can resolve API ==="
docker-compose exec frontend nslookup api

# Expected output:
# Name: api
# Address: 172.xx.xx.xx

# Test DNS resolution for database
echo "=== Frontend can resolve database ==="
docker-compose exec frontend nslookup db

# Expected output:
# Name: db
# Address: 172.xx.xx.xx

# Test connectivity: Frontend to API
echo "=== Frontend to API connectivity ==="
docker-compose exec frontend ping -c 3 api

# Expected output:
# PING api (172.xx.xx.xx): 56 data bytes
# 3 packets transmitted, 3 packets received

# Test connectivity: API to Database
echo "=== API to DB connectivity ==="
docker-compose exec api ping -c 3 db

# Expected output:
# PING db (172.xx.xx.xx): 56 data bytes
# 3 packets transmitted, 3 packets received

# Test port connectivity: API to Database on port 5432
echo "=== Test port connectivity ==="
docker-compose exec api nc -zv db 5432

# Expected output:
# Connection to db port 5432 [tcp/postgresql] succeeded!

# View network details
docker network ls | grep app-net

# Expected output:
# Shows app-net network

# Inspect network
docker network inspect $(docker-compose ps -q api | xargs -I {} docker inspect {} --format='{{json .NetworkSettings.Networks}}' | jq -r 'keys[0]' | xargs docker network ls -q | head -1)

# Expected output:
# Shows all containers connected to network

# Create alternative: Test with custom network
cat > docker-compose-custom.yml << 'EOF'
version: '3.8'

services:
  service1:
    image: nginx
    networks:
      - frontend-net

  service2:
    image: nginx
    networks:
      - backend-net

  service3:
    image: nginx
    networks:
      - frontend-net
      - backend-net

networks:
  frontend-net:
  backend-net:
EOF

# Test multi-network connectivity
docker-compose -f docker-compose-custom.yml up -d

# Service1 can reach Service3 (both on frontend-net)
docker-compose -f docker-compose-custom.yml exec service1 ping -c 1 service3

# Expected output:
# Success

# Service2 can reach Service3 (both on backend-net)
docker-compose -f docker-compose-custom.yml exec service2 ping -c 1 service3

# Expected output:
# Success

# Service1 cannot reach Service2 (different networks)
docker-compose -f docker-compose-custom.yml exec service1 ping -c 1 service2

# Expected output:
# Name or service not known (DNS fails)

# Cleanup
docker-compose down
docker-compose -f docker-compose-custom.yml down
rm -f docker-compose.yml docker-compose-custom.yml
```

### Explanation

- Docker Compose creates internal bridge network
- Services get DNS names matching service name
- Service discovery: Automatic DNS resolution
- Ping: Test basic connectivity
- nslookup: Test DNS resolution
- nc (netcat): Test specific port connectivity
- Network isolation: Services on different networks can't communicate
- Multi-network: Service can join multiple networks
- Internal DNS: Only works within Docker network
- External: Use published ports to expose outside Docker

---

## Exercise 6: Volume management

**Objective**: Create persistent volumes for database data and verify data survives container restart.

### Commands

```bash
# Create docker-compose with persistent volume
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  db:
    image: postgres:14
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
    volumes:
      - db_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

volumes:
  db_data:
EOF

# Start database
docker-compose up -d

# Expected output:
# Creating <project>_db_1 ... done

# Add test data to database
sleep 10
docker-compose exec db psql -U postgres -d myapp -c "CREATE TABLE users (id SERIAL, name TEXT);"

# Expected output:
# CREATE TABLE

docker-compose exec db psql -U postgres -d myapp -c "INSERT INTO users (name) VALUES ('Alice'), ('Bob'), ('Charlie');"

# Expected output:
# INSERT 0 3

# Verify data exists
docker-compose exec db psql -U postgres -d myapp -c "SELECT * FROM users;"

# Expected output:
# id |  name
# ---|-------
#  1 | Alice
#  2 | Bob
#  3 | Charlie

# Get volume info
docker volume ls | grep db_data

# Expected output:
# Shows db_data volume

# Stop and remove container
docker-compose down

# Expected output:
# Removing <project>_db_1 ... done

# Verify volume still exists
docker volume ls | grep db_data

# Expected output:
# Volume still exists

# Restart services
docker-compose up -d

# Expected output:
# Creating <project>_db_1 ... done

# Verify data survived restart
sleep 10
docker-compose exec db psql -U postgres -d myapp -c "SELECT * FROM users;"

# Expected output:
# Same data as before:
# id |  name
# ---|-------
#  1 | Alice
#  2 | Bob
#  3 | Charlie

# Create volume backup
echo "=== Create backup ==="
mkdir -p backups
docker run --rm -v db_data:/data -v $(pwd)/backups:/backup \
  busybox tar czf /backup/db_backup_$(date +%Y%m%d).tar.gz /data

# Expected output:
# Backup created

# Verify backup
ls -lh backups/

# Expected output:
# Shows backup file

# Create restore test: Create new volume from backup
docker volume create db_data_restored

docker run --rm -v db_data_restored:/data -v $(pwd)/backups:/backup \
  busybox sh -c "cd /data && tar xzf /backup/db_backup_*.tar.gz --strip-components=1"

# Expected output:
# Restore completed

# Inspect volume usage
docker system df

# Expected output:
# Shows volume sizes and usage

# Manual volume location (Linux)
docker volume inspect db_data

# Expected output:
# Shows mount path: /var/lib/docker/volumes/db_data/_data

# Cleanup
docker-compose down
docker volume rm db_data db_data_restored
rm -rf backups docker-compose.yml
```

### Explanation

- Volumes: Persist data outside container filesystem
- Named volumes: Managed by Docker, survives container lifecycle
- Bind mounts: Mount host directory into container
- `volumes:` in compose: Define and reference volumes
- Container removal: With volume, data persists
- Backup: Export volume data using tar
- Restore: Create new volume from backup
- Database volumes: Essential for production data persistence
- Best practice: Always use volumes for databases
- Data loss: Without volumes, data lost when container removed

---

## Exercise 7: Log aggregation

**Objective**: Collect logs from all services and view them with `docker-compose logs`.

### Commands

```bash
# Create services that generate logs
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  web:
    image: nginx
    ports:
      - "80:80"
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

  api:
    image: python:3.11
    command: python -c "
import time
import sys
for i in range(20):
    print(f'API Log {i}: Request processed', flush=True)
    sys.stdout.flush()
    time.sleep(1)
"
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

  worker:
    image: python:3.11
    command: python -c "
import time
import sys
for i in range(20):
    print(f'Worker Log {i}: Task completed', flush=True)
    sys.stdout.flush()
    time.sleep(2)
"
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
EOF

# Start services
docker-compose up -d

# Expected output:
# Creating <project>_web_1 ... done
# Creating <project>_api_1 ... done
# Creating <project>_worker_1 ... done

# View all logs
echo "=== View all logs ==="
docker-compose logs

# Expected output:
# Shows logs from all services

# View logs with follow (stream)
echo "=== Stream logs in real-time ==="
docker-compose logs -f

# Expected output:
# Real-time log streaming
# Press Ctrl+C to stop

# View logs for specific service
echo "=== View API logs only ==="
docker-compose logs api

# Expected output:
# Shows only API service logs

# View logs for multiple services
echo "=== View API and worker logs ==="
docker-compose logs api worker

# Expected output:
# Shows logs from api and worker only

# View last 20 lines
echo "=== View last 20 lines ==="
docker-compose logs --tail 20

# Expected output:
# Last 20 log lines from all services

# View logs since specific time
echo "=== View logs from last 30 seconds ==="
docker-compose logs --since 30s

# Expected output:
# Logs from last 30 seconds

# Stream with timestamps
echo "=== Stream logs with timestamps ==="
docker-compose logs -f --timestamps

# Expected output:
# Logs with timestamp prefixes

# Save logs to file
echo "=== Save logs to file ==="
docker-compose logs > all_logs.txt 2>&1

# Expected output:
# Logs saved to all_logs.txt

# View with grep
docker-compose logs | grep "API Log 5"

# Expected output:
# Shows lines matching pattern

# Monitor specific service with follow
echo "=== Monitor API service ==="
docker-compose logs -f --tail 10 api

# Expected output:
# Last 10 lines of API logs, then streaming updates

# View container logs directly (alternative)
docker logs $(docker-compose ps -q api)

# Expected output:
# Direct container logs

# Check log driver settings
docker inspect $(docker-compose ps -q web) | grep -A 5 "LogConfig"

# Expected output:
# Shows json-file driver configuration

# Cleanup
docker-compose down
rm -f all_logs.txt docker-compose.yml
```

### Explanation

- `docker-compose logs`: Aggregates logs from all containers
- `-f`: Follow/stream mode (real-time)
- `--tail N`: Show last N lines
- `--since`: Filter logs from specific time
- `--timestamps`: Add timestamp to each log line
- Log drivers: json-file (default), syslog, splunk, awslogs, etc.
- Log rotation: max-size and max-file options prevent disk fill
- Service-specific logs: View logs for individual services
- Log aggregation: Essential for multi-container troubleshooting
- Production: Centralize logs (ELK, Splunk, DataDog, Prometheus)

---

## Exercise 8: Network isolation

**Objective**: Create separate networks for frontend and api tiers; verify frontend cannot directly access database.

### Commands

```bash
# Create docker-compose with isolated networks
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  frontend:
    image: nginx
    ports:
      - "80:80"
    networks:
      - frontend-net
    depends_on:
      - api

  api:
    image: python:3.11
    command: python -m http.server 5000
    networks:
      - frontend-net
      - api-net
    ports:
      - "5000:5000"
    depends_on:
      - db

  db:
    image: postgres:14
    environment:
      POSTGRES_PASSWORD: password
    networks:
      - api-net
    ports:
      - "5432:5432"

networks:
  frontend-net:
    driver: bridge
  api-net:
    driver: bridge
EOF

# Start services
docker-compose up -d

# Expected output:
# Creating <project>_frontend_1 ... done
# Creating <project>_api_1 ... done
# Creating <project>_db_1 ... done

# Verify networks created
docker network ls | grep -E "frontend-net|api-net"

# Expected output:
# Shows both networks

# Test: Frontend can reach API (both on frontend-net)
echo "=== Frontend can reach API ==="
docker-compose exec frontend ping -c 1 api

# Expected output:
# Success (PING succeeds)

# Test: API can reach Database (both on api-net)
echo "=== API can reach database ==="
docker-compose exec api ping -c 1 db

# Expected output:
# Success (PING succeeds)

# Test: Frontend CANNOT reach Database (different networks)
echo "=== Frontend cannot reach database ==="
docker-compose exec frontend ping -c 1 db

# Expected output:
# Error: Name or service not known (DNS fails)
# or Network unreachable

# Test: Frontend cannot reach API on database port
echo "=== Frontend cannot access database port ==="
docker-compose exec frontend nc -zv db 5432

# Expected output:
# Connection refused or Name not known

# View network configuration
echo "=== Frontend network connections ==="
docker inspect $(docker-compose ps -q frontend) | grep -A 20 "Networks"

# Expected output:
# Shows frontend connected only to frontend-net

echo "=== API network connections ==="
docker inspect $(docker-compose ps -q api) | grep -A 20 "Networks"

# Expected output:
# Shows api connected to both frontend-net and api-net

echo "=== Database network connections ==="
docker inspect $(docker-compose ps -q db) | grep -A 20 "Networks"

# Expected output:
# Shows db connected only to api-net

# List containers on each network
echo "=== Containers on frontend-net ==="
docker network inspect $(docker network ls --filter name=frontend-net -q) --format '{{.Containers}}'

# Expected output:
# Shows frontend and api

echo "=== Containers on api-net ==="
docker network inspect $(docker network ls --filter name=api-net -q) --format '{{.Containers}}'

# Expected output:
# Shows api and db

# Demonstrate network isolation benefit
echo "=== Network isolation prevents direct DB access from frontend ==="
echo "This is secure: Frontend only knows about API service"
echo "Database details are hidden from frontend"
echo "Database can only be accessed through API layer"

# Cleanup
docker-compose down
docker network prune -f
rm -f docker-compose.yml
```

### Explanation

- Network isolation: Separate networks limit service communication
- Frontend network: Only frontend and API
- API network: Only API and database
- API acts as bridge: Only service touching both networks
- Security: Prevents frontend from accessing database directly
- Layered architecture: Each tier can only see what it needs
- Firewall-like behavior: Network boundaries enforce access control
- Best practice: Minimize service interconnectedness
- Production: Implement network policies and firewalls
- Zero-trust: Assume breach, control all communication

---

## Exercise 9: Rolling restart

**Objective**: Update a service image and perform rolling restart with zero downtime.

### Commands

```bash
# Create initial docker-compose
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  web:
    image: nginx:1.20
    ports:
      - "80:80"
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
EOF

# Note: deploy config requires Swarm mode or Docker Compose v2
# Alternative for Compose v1: Use manual restart

# Start initial services
docker-compose up -d

# Expected output:
# Creating <project>_web_1 ... done
# Creating <project>_web_2 ... done
# Creating <project>_web_3 ... done

# Verify running version
docker-compose ps

# Expected output:
# All containers running nginx:1.20

# Check nginx version inside container
docker-compose exec web nginx -v

# Expected output:
# nginx version: nginx/1.20.x

# Simulate traffic to show zero downtime
cat > monitor.sh << 'EOF'
#!/bin/bash
for i in {1..30}; do
  echo "Request $i: $(curl -s -o /dev/null -w '%{http_code}' http://localhost/)"
  sleep 1
done
EOF

chmod +x monitor.sh

# Start monitoring in background
./monitor.sh > /tmp/requests.log 2>&1 &

# Update image to new version
echo "=== Updating image ==="
docker-compose exec web sed -i 's/1.20/1.21/g' docker-compose.yml 2>/dev/null || true

# Method 1: Manual rolling update
echo "=== Manual rolling restart ==="
for container in $(docker-compose ps -q web); do
  echo "Restarting $container..."
  docker stop $container
  sleep 5
  docker-compose up -d
  sleep 5
done

# Expected output:
# Containers restart one by one

# Method 2: Update specific service
docker-compose pull web

docker-compose up -d web --no-deps --build

# Expected output:
# Service updated

# Verify no downtime in requests
wait
cat /tmp/requests.log | grep -c "200"

# Expected output:
# Should show all requests succeeded (200 status)

# Verify new image
docker-compose exec web nginx -v

# Expected output:
# nginx version: nginx/1.21.x (or newer)

# Alternative: Using docker service (Swarm mode)
echo "=== Using Docker Swarm for rolling update ==="

# Initialize Swarm
docker swarm init 2>/dev/null || true

# Create service with replicas
docker service create \
  --name web-service \
  --replicas 3 \
  -p 80:80 \
  nginx:1.20

# Expected output:
# Service created

# View service status
docker service ps web-service

# Expected output:
# Shows 3 replicas running

# Update service with new image (rolling by default)
docker service update \
  --image nginx:1.21 \
  --update-delay 10s \
  --update-parallelism 1 \
  web-service

# Expected output:
# Service updated

# Watch rolling update
docker service ps -f "desired-state=running" web-service

# Expected output:
# Shows containers updating one by one

# Verify all replicas updated
sleep 40
docker service ps web-service | grep -c "nginx:1.21"

# Expected output:
# 3 (all replicas running new image)

# Cleanup
docker-compose down 2>/dev/null
docker service rm web-service 2>/dev/null
docker swarm leave --force 2>/dev/null
rm -f monitor.sh docker-compose.yml /tmp/requests.log
```

### Explanation

- Rolling restart: Update containers one-by-one with zero downtime
- Manual approach: Stop/start containers sequentially
- Docker Compose: `docker-compose up` replaces containers gracefully
- Docker Swarm: `docker service update` handles rolling updates automatically
- `--update-delay`: Wait between updating each container
- `--update-parallelism`: How many containers to update simultaneously
- Zero downtime: Other replicas handle requests during update
- Load balancer: Distributes traffic, masks container restarts
- Health checks: Verify container ready before handling traffic
- Rollback: Easy to revert if issues found during update

---

## Exercise 10: Production checklist

**Objective**: Document and implement security, monitoring, backup, and incident response procedures.

### Commands

```bash
# Create production checklist document
cat > PRODUCTION_CHECKLIST.md << 'EOF'
# Production Deployment Checklist

## Security

- [ ] Non-root user in Dockerfile: `RUN useradd -m appuser && chown -R appuser:appuser /app`
- [ ] Image scanning: Check for vulnerabilities before pushing
- [ ] Secrets management: Use Docker secrets or environment management tool
- [ ] Network isolation: Separate networks for different tiers
- [ ] Resource limits: Set memory and CPU limits
- [ ] Read-only filesystem: `--read-only` for immutable containers
- [ ] No hardcoded credentials: Use secrets or env variables
- [ ] Regular updates: Keep base images and dependencies updated
- [ ] Access control: Limit who can deploy/manage containers

## Monitoring & Observability

- [ ] Health checks: HEALTHCHECK in Dockerfile or healthcheck in compose
- [ ] Resource monitoring: docker stats, Prometheus, Datadog
- [ ] Centralized logging: ELK, Splunk, or cloud logging service
- [ ] Metrics: CPU, memory, network, disk, application-specific metrics
- [ ] Alerting: Set thresholds and notification channels
- [ ] Distributed tracing: Track requests across services
- [ ] APM tools: Application performance monitoring
- [ ] Log retention: Define log retention policies

## Reliability & Availability

- [ ] Restart policy: `--restart=unless-stopped`
- [ ] Multiple replicas: At least 2-3 replicas per service
- [ ] Load balancing: Distribute traffic across replicas
- [ ] Graceful shutdown: Handle SIGTERM signals properly
- [ ] Database backups: Automated backup strategy
- [ ] Disaster recovery: Tested backup restoration
- [ ] SLOs/SLIs: Define uptime and performance targets
- [ ] Incident response plan: Documented procedures

## Data Management

- [ ] Persistent volumes: Database data in volumes, not containers
- [ ] Backup strategy: Daily/weekly backups with retention policy
- [ ] Backup testing: Regularly test backup restoration
- [ ] Data encryption: Encrypt data at rest and in transit
- [ ] Data retention: Know retention requirements for compliance
- [ ] Disaster recovery: Test full system recovery procedures

## Deployment

- [ ] Version tagging: Use semantic versioning (1.0.0, 1.0.1, etc.)
- [ ] Docker Hub/Registry: Push images to registry
- [ ] Registry security: Private registry with access controls
- [ ] Deploy automation: CI/CD pipeline for deployments
- [ ] Staging environment: Test in prod-like environment first
- [ ] Rollback plan: Quick rollback if issues found
- [ ] Deployment documentation: Clear deployment procedures
- [ ] Feature flags: Gradual rollout of new features

## Configuration Management

- [ ] Environment files: .env for dev, separate for prod
- [ ] Configuration validation: Verify all required vars set
- [ ] Secrets management: Never commit secrets to git
- [ ] Configuration versioning: Track config changes
- [ ] Documentation: Document all configuration options
- [ ] Configuration auditing: Log who changed what and when

## Maintenance

- [ ] Image cleanup: Remove old/unused images regularly
- [ ] Container cleanup: Remove stopped containers
- [ ] Volume cleanup: Remove orphaned volumes
- [ ] Log rotation: Prevent logs from filling disk
- [ ] Database maintenance: Indexes, vacuuming, optimization
- [ ] Dependency updates: Regular security and feature updates
- [ ] Documentation updates: Keep runbooks current
- [ ] Disaster recovery drills: Regular practice restores

## Compliance & Auditing

- [ ] Vulnerability scanning: Regular scans of images
- [ ] Compliance requirements: Meet regulatory requirements
- [ ] Audit logging: Log all administrative actions
- [ ] Data access logs: Track who accessed what data
- [ ] Change management: Documented process for changes
- [ ] Security policies: Document security standards
- [ ] Incident documentation: Record and learn from incidents
EOF

cat PRODUCTION_CHECKLIST.md

# Create production-ready docker-compose
cat > docker-compose.prod.yml << 'EOF'
version: '3.8'

services:
  web:
    image: myapp:1.0.0
    restart: unless-stopped
    ports:
      - "80:80"
    environment:
      LOG_LEVEL: info
      DEBUG: "false"
    volumes:
      - app_logs:/var/log/app
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    security_opt:
      - no-new-privileges:true
    user: appuser
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
        labels: "service=web"

  db:
    image: postgres:14-alpine
    restart: unless-stopped
    environment:
      POSTGRES_DB: ${DB_NAME}
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - db_data:/var/lib/postgresql/data
    deploy:
      resources:
        limits:
          cpus: '1'
          memory: 1024M
        reservations:
          cpus: '0.5'
          memory: 512M
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER}"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    security_opt:
      - no-new-privileges:true
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
        labels: "service=db"

volumes:
  app_logs:
    driver: local
  db_data:
    driver: local
EOF

# Create incident response runbook
cat > RUNBOOK.md << 'EOF'
# Incident Response Runbook

## Common Issues

### 1. Service Restart Loop
**Symptoms**: Container keeps restarting

**Diagnosis**:
```bash
docker-compose logs web | tail -50
docker inspect <container> --format='{{.State.ExitCode}}'
```

**Resolution**:
- Check logs for errors
- Verify environment variables
- Check resource limits
- Restart with: `docker-compose restart web`

### 2. Out of Memory
**Symptoms**: Container killed, exit 137

**Diagnosis**:
```bash
docker stats web
docker inspect web --format='{{.HostConfig.Memory}}'
```

**Resolution**:
- Increase limit in docker-compose
- Scale horizontally with multiple replicas
- Optimize application

### 3. Database Connection Issues
**Symptoms**: "Connection refused" errors

**Diagnosis**:
```bash
docker-compose exec db pg_isready
docker-compose logs db | tail -20
```

**Resolution**:
- Verify db service running: `docker-compose ps db`
- Check database logs for errors
- Restart: `docker-compose restart db`

## Backup & Recovery

### Create Backup
```bash
docker run --rm -v db_data:/data -v /backup:/backup \
  busybox tar czf /backup/backup-$(date +%Y%m%d).tar.gz /data
```

### Restore from Backup
```bash
docker-compose down
docker volume rm db_data
docker volume create db_data
docker run --rm -v db_data:/data -v /backup:/backup \
  busybox tar xzf /backup/backup-*.tar.gz
docker-compose up -d
```
EOF

cat RUNBOOK.md

# Create deployment script
cat > deploy.sh << 'EOF'
#!/bin/bash
set -e

IMAGE=$1
TAG=${2:-latest}

if [ -z "$IMAGE" ]; then
  echo "Usage: ./deploy.sh <image> [tag]"
  exit 1
fi

echo "Deploying $IMAGE:$TAG"

# Pull latest image
docker pull $IMAGE:$TAG

# Update docker-compose file
sed -i "s|image: .*|image: $IMAGE:$TAG|" docker-compose.prod.yml

# Deploy with rolling restart
docker-compose -f docker-compose.prod.yml up -d web --no-deps

echo "Deployment complete!"
echo "Verify: docker-compose -f docker-compose.prod.yml ps"
EOF

chmod +x deploy.sh

# Create monitoring script
cat > monitor.sh << 'EOF'
#!/bin/bash

echo "=== System Health Check ==="
docker-compose ps

echo ""
echo "=== Resource Usage ==="
docker stats --no-stream

echo ""
echo "=== Log Summary ==="
docker-compose logs --tail 10

echo ""
echo "=== Service Health ==="
for service in web db; do
  status=$(docker-compose ps $service | grep -o "healthy\|unhealthy\|starting" || echo "unknown")
  echo "$service: $status"
done
EOF

chmod +x monitor.sh

# Test monitoring
./monitor.sh

# View what was created
echo ""
echo "=== Created files ==="
ls -la *.md *.sh docker-compose.prod.yml .env 2>/dev/null || true

# Cleanup
rm -f PRODUCTION_CHECKLIST.md RUNBOOK.md deploy.sh monitor.sh docker-compose.prod.yml
```

### Explanation

- Production checklist: Comprehensive guide for production deployment
- Security: Non-root users, no hardcoded secrets, image scanning
- Monitoring: Health checks, metrics, centralized logging, alerting
- Reliability: Restart policies, multiple replicas, load balancing
- Data: Persistent volumes, automated backups, tested recovery
- Deployment: Versioning, registry, CI/CD automation
- Configuration: Environment management, no secrets in code
- Incident response: Documented procedures for common issues
- Maintenance: Regular updates, cleanup, optimization
- Compliance: Auditing, vulnerability scanning, documentation

---

