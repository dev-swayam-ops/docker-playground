# Solutions: Docker Compose

Comprehensive solutions for all 10 exercises with detailed commands and explanations.

---

## Exercise 1: Create simple docker-compose.yml (web + db)

### Solution

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  web:
    image: nginx:latest
    container_name: web-service
    ports:
      - "8080:80"
    depends_on:
      - database
    networks:
      - app-network

  database:
    image: alpine:latest
    container_name: db-service
    command: sleep 300
    networks:
      - app-network

networks:
  app-network:
    driver: bridge
```

### Commands

```bash
# Start services in background
docker-compose up -d

# Expected output:
# Creating network "app_app-network" with driver "bridge"
# Creating web-service ... done
# Creating db-service ... done

# Verify services are running
docker-compose ps

# Expected output:
# NAME        COMMAND                  SERVICE     STATUS      PORTS
# web-service "nginx -g 'daemon ..."  web         Up 5 seconds 0.0.0.0:8080->80/tcp
# db-service  "sleep 300"              database    Up 5 seconds

# Test web service is accessible
curl http://localhost:8080

# Expected output:
# (nginx default page)

# View logs
docker-compose logs

# Expected output:
# web-service  | /docker-entrypoint.sh: /docker-entrypoint.d/ is not empty...
# db-service   | (no logs)

# List containers created by compose
docker ps

# Expected output shows:
# web-service and db-service containers

# List networks created by compose
docker network ls | grep app_

# Expected output:
# (network name created by compose)

# Stop services
docker-compose down

# Expected output:
# Stopping web-service ... done
# Stopping db-service ... done
# Removing web-service ... done
# Removing db-service ... done
# Removing network app_app-network

# Verify containers are removed
docker ps -a | grep -E 'web-service|db-service'

# Expected output:
# (no containers)
```

### Explanation

**Compose File Structure:**
- `version`: Compose file format version
- `services`: Define containers
- `networks`: Define custom networks
- `volumes`: Define named volumes

**Service Properties:**
- `image`: Docker image to use
- `container_name`: Custom container name
- `ports`: Port mapping
- `depends_on`: Service dependencies
- `networks`: Connect to networks

---

## Exercise 2: Define services with images and ports

### Solution

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  web:
    image: nginx:1.21-alpine
    container_name: web-app
    ports:
      - "8080:80"
      - "8443:443"

  api:
    image: python:3.11-slim
    container_name: api-app
    ports:
      - "5000:5000"
    command: python -m http.server 5000

  db:
    image: postgres:15-alpine
    container_name: database
    ports:
      - "5432:5432"
    environment:
      POSTGRES_PASSWORD: password123

  cache:
    image: redis:7-alpine
    container_name: redis-cache
    ports:
      - "6379:6379"
```

### Commands

```bash
# Start all services
docker-compose up -d

# Expected output:
# Creating web-app ... done
# Creating api-app ... done
# Creating database ... done
# Creating redis-cache ... done

# Verify all are running
docker-compose ps

# Expected output:
# NAME             COMMAND                  SERVICE   STATUS      PORTS
# web-app          "/docker-entrypoint.sh"  web       Up 5s       0.0.0.0:8080->80/tcp, 0.0.0.0:8443->443/tcp
# api-app          "python -m http.serve…"  api       Up 5s       0.0.0.0:5000->5000/tcp
# database         "postgres"               db        Up 5s       0.0.0.0:5432->5432/tcp
# redis-cache      "redis-server"           cache     Up 5s       0.0.0.0:6379->6379/tcp

# Test each service
curl http://localhost:8080  # Web server

# Expected output:
# (nginx page)

curl http://localhost:5000  # API server

# Expected output:
# (directory listing)

# Test database connection
docker exec database psql -U postgres -c "SELECT 1"

# Expected output:
#  ?column?
# ----------
#         1

# Test Redis connection
docker exec redis-cache redis-cli ping

# Expected output:
# PONG

# View specific service logs
docker-compose logs web

# Expected output:
# web-app | /docker-entrypoint.sh: /docker-entrypoint.d/ is not empty...

# Stop and remove
docker-compose down

# Expected output:
# Stopping services and removing containers
```

### Explanation

**Service Definitions:**
- Each service maps to a container
- `image`: Specifies base image and version
- `ports`: Exposes ports from container to host
- `environment`: Sets environment variables
- `command`: Overrides default container command

**Port Mapping Format:**
- `host_port:container_port`
- Can map multiple ports per service
- Enable external access to services

---

## Exercise 3: Set environment variables

### Solution

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  web:
    image: nginx:latest
    container_name: web-service
    environment:
      - NGINX_HOST=localhost
      - NGINX_PORT=80
      - LOG_LEVEL=info

  api:
    image: python:3.11-slim
    container_name: api-service
    environment:
      DATABASE_URL: postgresql://user:password@db:5432/mydb
      REDIS_URL: redis://cache:6379
      DEBUG: "true"
      API_PORT: 5000
      API_HOST: 0.0.0.0

  database:
    image: postgres:15-alpine
    container_name: db-service
    environment:
      POSTGRES_USER: appuser
      POSTGRES_PASSWORD: secretpass123
      POSTGRES_DB: production_db
      POSTGRES_INITDB_ARGS: "--encoding=UTF8"

  cache:
    image: redis:7-alpine
    container_name: cache-service
    environment:
      - REDIS_PASSWORD=cachepass123
```

### Commands

```bash
# Create .env file for sensitive data
cat > .env << 'EOF'
DB_USER=appuser
DB_PASSWORD=supersecret123
REDIS_PASSWORD=redispass
API_KEY=sk-1234567890abcdef
EOF

# Reference environment file in docker-compose
# Add to docker-compose.yml:
# env_file:
#   - .env

# Start services
docker-compose up -d

# Expected output:
# Creating services with environment variables set

# Verify environment variables are set in container
docker exec api-service env | grep -E 'DATABASE|REDIS|DEBUG'

# Expected output:
# DATABASE_URL=postgresql://user:password@db:5432/mydb
# REDIS_URL=redis://cache:6379
# DEBUG=true

# Check database container
docker exec db-service psql -U appuser -d production_db -c "SELECT 1"

# Expected output:
#  ?column?
# ----------
#         1

# View specific service logs
docker-compose logs database

# Expected output shows database startup with env vars

# Override environment variable at runtime
docker-compose run -e DEBUG=false api-service env | grep DEBUG

# Expected output:
# DEBUG=false

# Use separate .env files for different environments
cat > .env.prod << 'EOF'
DB_USER=produser
DB_PASSWORD=productionpass123
DEBUG=false
EOF

# Use specific env file
docker-compose --env-file .env.prod up -d

# Stop and clean up
docker-compose down

# Expected output shows services removed
```

### Explanation

**Environment Variables:**
- Can be set directly in compose file
- Can reference .env files
- Can be overridden at runtime
- Useful for configuration without modifying images

**Methods:**
1. Direct: `VARIABLE: value`
2. List: `- VARIABLE=value`
3. File: `env_file: .env`
4. Runtime: `-e VARIABLE=value`

---

## Exercise 4: Add volumes to services

### Solution

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  web:
    image: nginx:latest
    container_name: web-service
    ports:
      - "8080:80"
    volumes:
      - ./html:/usr/share/nginx/html:ro  # Bind mount (read-only)
      - web_logs:/var/log/nginx          # Named volume

  api:
    image: python:3.11-slim
    container_name: api-service
    volumes:
      - ./app:/app:rw                    # Bind mount (read-write)
      - app_cache:/app/cache             # Named volume
    command: sleep 300

  database:
    image: postgres:15-alpine
    container_name: db-service
    volumes:
      - db_data:/var/lib/postgresql/data # Named volume (persistence)
    environment:
      POSTGRES_PASSWORD: secret

  cache:
    image: redis:7-alpine
    container_name: cache-service
    volumes:
      - cache_data:/data                 # Named volume

volumes:
  web_logs:
  app_cache:
  db_data:
  cache_data:
```

### Commands

```bash
# Create directories for bind mounts
mkdir -p html app

# Create sample HTML file
echo "<h1>Hello from Compose</h1>" > html/index.html

# Start services
docker-compose up -d

# Expected output:
# Creating volumes and services

# Verify volumes are created
docker volume ls | grep -E 'web_logs|app_cache|db_data|cache_data'

# Expected output shows all named volumes

# Check volume details
docker volume inspect compose_web_logs

# Expected output:
# [
#   {
#     "Name": "compose_web_logs",
#     "Driver": "local",
#     "Mountpoint": "/var/lib/docker/volumes/compose_web_logs/_data",
#     ...
#   }
# ]

# Test bind mount (read-only web volume)
docker exec web-service ls -la /usr/share/nginx/html/

# Expected output:
# index.html file exists

# Try to write to read-only mount (should fail)
docker exec web-service touch /usr/share/nginx/html/test.txt

# Expected output:
# Read-only file system

# Test read-write bind mount
docker exec api-service ls -la /app/

# Expected output:
# app directory content

# Create file from container
docker exec api-service touch /app/testfile.txt

# Verify file exists on host
ls -la app/testfile.txt

# Expected output:
# -rw-r--r-- 1 root root 0 Jan 15 12:00 app/testfile.txt

# Check database volume
docker exec db-service ls -la /var/lib/postgresql/data/

# Expected output shows PostgreSQL data files

# Write data to database
docker exec db-service psql -U postgres -c "CREATE TABLE test (id SERIAL PRIMARY KEY);"

# Stop and remove containers
docker-compose down

# Expected output:
# Stopping services

# Verify database data persists
docker-compose up -d

# Check the table still exists
docker exec db-service psql -U postgres -c "\\dt"

# Expected output shows the test table

# Cleanup volumes
docker-compose down -v

# Expected output:
# Removes containers and volumes
```

### Explanation

**Volume Types in Compose:**

| Type | Syntax | Persistence | Use Case |
|------|--------|-------------|----------|
| Named | `volume_name:/path` | Yes | Data persistence |
| Bind | `./local:/path` | Yes | Development |
| Read-only | `:/path:ro` | Yes | Config files |
| Read-write | `:/path:rw` | Yes | Default |

**Volume Management:**
- Named volumes defined in `volumes` section
- Bind mounts use host relative paths
- `:ro` suffix makes mount read-only
- `:rw` is default for bind mounts

---

## Exercise 5: Define networks

### Solution

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  web:
    image: nginx:latest
    container_name: web-service
    ports:
      - "8080:80"
    networks:
      - frontend  # Connect to frontend network

  api:
    image: python:3.11-slim
    container_name: api-service
    networks:
      - frontend  # Connect to frontend
      - backend   # Connect to backend

  database:
    image: postgres:15-alpine
    container_name: db-service
    networks:
      - backend   # Connect to backend only

  cache:
    image: redis:7-alpine
    container_name: cache-service
    networks:
      - backend   # Connect to backend only

networks:
  frontend:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
  
  backend:
    driver: bridge
    ipam:
      config:
        - subnet: 172.21.0.0/16
```

### Commands

```bash
# Start services
docker-compose up -d

# Expected output:
# Creating networks and services

# List networks created by compose
docker network ls | grep -E 'frontend|backend'

# Expected output:
# compose_frontend  bridge
# compose_backend   bridge

# Inspect frontend network
docker network inspect compose_frontend

# Expected output:
# "Containers": {
#   "web-service": { ... },
#   "api-service": { ... }
# }

# Inspect backend network
docker network inspect compose_backend

# Expected output:
# "Containers": {
#   "api-service": { ... },
#   "database": { ... },
#   "cache-service": { ... }
# }

# Test network isolation - web can reach api
docker exec web-service ping -c 1 api

# Expected output:
# 1 packets transmitted, 1 packets received

# Test network isolation - web cannot reach database (different network)
docker exec web-service ping -c 1 database

# Expected output:
# ping: can't resolve database

# API can reach both frontend and backend
docker exec api-service ping -c 1 web

# Expected output:
# 1 packets transmitted, 1 packets received

docker exec api-service ping -c 1 database

# Expected output:
# 1 packets transmitted, 1 packets received

# Database cannot reach web
docker exec database ping -c 1 web

# Expected output:
# ping: can't resolve web

# Check IP addresses on different networks
docker inspect web-service | jq '.[] | .NetworkSettings.Networks'

# Expected output:
# {
#   "compose_frontend": {
#     "IPAddress": "172.20.0.2",
#     ...
#   }
# }

docker inspect api-service | jq '.[] | .NetworkSettings.Networks'

# Expected output shows two networks:
# {
#   "compose_frontend": { "IPAddress": "172.20.0.3", ... },
#   "compose_backend": { "IPAddress": "172.21.0.2", ... }
# }

# Stop and remove
docker-compose down

# Expected output:
# Removing services and networks
```

### Explanation

**Network Isolation Benefits:**
1. Security: Limit service-to-service communication
2. Performance: Reduce traffic between networks
3. Organization: Logical grouping of services
4. Multi-tenancy: Separate application tiers

**Network Configuration:**
- `driver`: Type of network (bridge, host, overlay)
- `ipam`: IP address management settings
- `subnet`: Custom subnet range
- Services connect via network name

---

## Exercise 6: Use depends_on for startup order

### Solution

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  web:
    image: nginx:latest
    container_name: web-service
    ports:
      - "8080:80"
    depends_on:
      api:
        condition: service_started  # Wait for API to start
    networks:
      - app-network

  api:
    image: python:3.11-slim
    container_name: api-service
    ports:
      - "5000:5000"
    depends_on:
      - database
      - cache
    command: sleep 300
    networks:
      - app-network

  database:
    image: postgres:15-alpine
    container_name: db-service
    depends_on:
      - cache      # Wait for cache first
    environment:
      POSTGRES_PASSWORD: secret
    networks:
      - app-network
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

  cache:
    image: redis:7-alpine
    container_name: cache-service
    networks:
      - app-network
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

networks:
  app-network:
```

### Commands

```bash
# Start services with dependency order
docker-compose up -d

# Expected output:
# Creating cache-service ... done
# Creating db-service ... done
# Creating api-service ... done
# Creating web-service ... done

# Verify startup order in logs
docker-compose logs | head -50

# Expected output shows cache starting first, then database, api, web

# Check health of each service
docker-compose ps

# Expected output:
# NAME              COMMAND              SERVICE    STATUS      HEALTH
# cache-service     "redis-server"       cache      Up 30s      healthy
# db-service        "postgres"           database   Up 20s      healthy
# api-service       "python -m http..."  api        Up 10s
# web-service       "nginx..."           web        Up 5s

# Test service communication
docker exec web-service curl -s http://api:5000/

# Expected output:
# (API response)

# Test database connectivity from API
docker exec api-service nc -zv database 5432

# Expected output:
# database 5432 open

# Test cache connectivity
docker exec api-service redis-cli -h cache ping

# Expected output:
# PONG

# Watch startup process
docker-compose up --build

# Press Ctrl+C to stop logs (services continue running)

# View service startup times
docker inspect web-service | jq '.State.StartedAt'
docker inspect api-service | jq '.State.StartedAt'
docker inspect database | jq '.State.StartedAt'
docker inspect cache-service | jq '.State.StartedAt'

# Stop and remove
docker-compose down

# Expected output:
# Stopping services in reverse order
```

### Explanation

**depends_on Conditions:**
- `service_started`: Wait for service to start (default)
- `service_healthy`: Wait for health check to pass
- Multiple dependencies: Wait for all to be ready

**Startup Order:**
1. Services with no dependencies start first
2. Services wait for their dependencies
3. Order specified by depends_on relationships
4. Health checks ensure readiness

---

## Exercise 7: Build images from Dockerfile

### Solution

**Dockerfile:**
```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY app.py .

CMD ["python", "app.py"]
```

**requirements.txt:**
```
Flask==2.3.0
redis==4.5.0
```

**app.py:**
```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello():
    return 'Hello from Custom Image!'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  api:
    build:
      context: .
      dockerfile: Dockerfile
    image: my-api:1.0
    container_name: api-service
    ports:
      - "5000:5000"
    depends_on:
      - cache
    networks:
      - app-network

  database:
    build:
      context: ./db
      dockerfile: Dockerfile
    image: custom-postgres:1.0
    container_name: db-service
    environment:
      POSTGRES_PASSWORD: secret
    networks:
      - app-network

  cache:
    image: redis:7-alpine
    container_name: cache-service
    networks:
      - app-network

networks:
  app-network:
```

### Commands

```bash
# Build images from Dockerfile
docker-compose build

# Expected output:
# Building api
# Sending build context to Docker daemon  2.048kB
# Step 1/5 : FROM python:3.11-slim
# ...
# Successfully tagged my-api:1.0

# Verify images are built
docker image ls | grep -E 'my-api|custom-postgres'

# Expected output:
# my-api           1.0       abc123    10 minutes ago  150MB
# custom-postgres  1.0       def456    5 minutes ago   200MB

# Start services with built images
docker-compose up -d

# Expected output:
# Creating api-service ... done
# Creating database-service ... done
# Creating cache-service ... done

# Test the custom API image
curl http://localhost:5000/

# Expected output:
# Hello from Custom Image!

# View image details
docker inspect my-api:1.0 | grep -A 10 '"Config"'

# Expected output shows Dockerfile configuration

# Build with specific Dockerfile
docker-compose build --build-arg BUILD_DATE=$(date -u +'%Y-%m-%dT%H:%M:%SZ') api

# Expected output:
# Building api with build args

# Build without cache (fresh build)
docker-compose build --no-cache

# Expected output:
# Full rebuild, no cache reuse

# Build specific service
docker-compose build database

# Expected output:
# Building only database image

# Check build logs
docker-compose build --progress=plain api

# Expected output:
# Detailed build progress

# Stop and remove
docker-compose down

# Remove built images
docker rmi my-api:1.0 custom-postgres:1.0
```

### Explanation

**Build Configuration:**
- `context`: Directory containing Dockerfile
- `dockerfile`: Dockerfile path (default: Dockerfile)
- `args`: Build-time arguments
- `target`: Multi-stage target

**Benefits:**
- Custom images for each service
- Version control of images
- Reproducible builds
- Development and production images

---

## Exercise 8: Override compose settings with environment

### Solution

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  web:
    image: nginx:latest
    container_name: ${WEB_CONTAINER_NAME:-web-service}
    ports:
      - "${WEB_PORT:-8080}:80"
    environment:
      - ENVIRONMENT=${APP_ENV:-development}
      - LOG_LEVEL=${LOG_LEVEL:-info}

  api:
    image: python:3.11-slim
    container_name: ${API_CONTAINER_NAME:-api-service}
    ports:
      - "${API_PORT:-5000}:5000"
    environment:
      DATABASE_HOST: ${DB_HOST:-database}
      DATABASE_PORT: ${DB_PORT:-5432}
      DATABASE_NAME: ${DB_NAME:-mydb}
      DEBUG: ${DEBUG:-false}

  database:
    image: postgres:15-alpine
    container_name: ${DB_CONTAINER_NAME:-db-service}
    ports:
      - "${DB_PORT:-5432}:5432"
    environment:
      POSTGRES_USER: ${DB_USER:-postgres}
      POSTGRES_PASSWORD: ${DB_PASSWORD:-secret}
      POSTGRES_DB: ${DB_NAME:-mydb}
```

### Commands

```bash
# Create .env file with custom values
cat > .env << 'EOF'
WEB_CONTAINER_NAME=production-web
WEB_PORT=80
API_CONTAINER_NAME=production-api
API_PORT=3000
APP_ENV=production
LOG_LEVEL=warn
DB_HOST=prod-db.example.com
DB_PORT=5432
DB_USER=produser
DB_PASSWORD=prodpass123
DB_NAME=production_db
DEBUG=false
EOF

# Start with environment variables from .env
docker-compose up -d

# Expected output:
# Creating production-web ... done
# Creating production-api ... done
# Creating production-db-service ... done

# Verify containers use environment values
docker ps | grep production

# Expected output:
# production-web
# production-api

# Check environment variables set in container
docker exec production-api env | grep DATABASE

# Expected output:
# DATABASE_HOST=prod-db.example.com
# DATABASE_PORT=5432
# DATABASE_NAME=production_db

# Override with command-line variables
WEB_PORT=9000 docker-compose up -d

# Expected output:
# Web service now on port 9000

# Verify port change
docker ps | grep "9000"

# Expected output shows port 9000

# Use different .env file for staging
cat > .env.staging << 'EOF'
WEB_PORT=8000
APP_ENV=staging
LOG_LEVEL=debug
DEBUG=true
EOF

# Use specific env file
docker-compose --env-file .env.staging up -d

# Stop current services
docker-compose down

# Expected output:
# Removing services

# Test with no .env (use defaults)
docker-compose up -d

# Expected output:
# Using default values from compose file

# Verify defaults are used
docker ps | grep "web-service"

# Expected output shows default container name

# Set environment at runtime
export APP_ENV=testing
export LOG_LEVEL=debug
docker-compose up -d

# Expected output:
# Services start with testing environment
```

### Explanation

**Environment Override Syntax:**
- `${VARIABLE:-default}` - Use VARIABLE, fallback to default
- `${VARIABLE}` - Required variable, error if not set
- `.env` file automatically loaded
- Command-line `-e` takes precedence

**Priority Order:**
1. Command-line `-e` flag (highest)
2. Environment variables
3. `.env` file
4. Default values in compose (lowest)

---

## Exercise 9: Scale services with scale command

### Solution

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  web:
    image: nginx:latest
    ports:
      - "8000-8003:80"  # Port range for scaled instances
    networks:
      - app-network

  api:
    image: python:3.11-slim
    ports:
      - "5000-5003:5000"  # Port range for scaled instances
    command: sleep 300
    networks:
      - app-network

  database:
    image: postgres:15-alpine
    environment:
      POSTGRES_PASSWORD: secret
    networks:
      - app-network

networks:
  app-network:
```

### Commands

```bash
# Start services with default replicas (1)
docker-compose up -d

# Expected output:
# Creating services (1 instance each)

# Verify running services
docker-compose ps

# Expected output:
# NAME     COMMAND       SERVICE    REPLICAS
# compose_web_1    nginx      web        1/1
# compose_api_1    sleep 300  api        1/1
# compose_database_1 postgres database  1/1

# Scale web service to 3 instances
docker-compose up -d --scale web=3

# Expected output:
# Creating compose_web_1 ... done
# Creating compose_web_2 ... done
# Creating compose_web_3 ... done

# Verify scaling
docker-compose ps | grep web

# Expected output:
# compose_web_1  nginx  web  ...
# compose_web_2  nginx  web  ...
# compose_web_3  nginx  web  ...

# Scale API to 4 instances
docker-compose up -d --scale api=4

# Expected output:
# Creating api_1, api_2, api_3, api_4

# View all scaled services
docker-compose ps

# Expected output:
# web: 3 instances
# api: 4 instances
# database: 1 instance

# Test load balancing (DNS rounds through instances)
docker exec compose_web_1 nslookup api

# Expected output:
# Name:      api
# Address:   (first instance IP)

# Scale down web service
docker-compose up -d --scale web=2

# Expected output:
# Removing compose_web_3

# Verify web now has 2 instances
docker-compose ps | grep web

# Expected output:
# compose_web_1
# compose_web_2

# Scale all to 1 (reset)
docker-compose up -d --scale web=1 --scale api=1

# Expected output:
# Stopping excess instances

# Verify
docker-compose ps

# Expected output:
# All services back to 1 instance

# Scale to 0 (stop without removing)
docker-compose up -d --scale web=0

# Expected output:
# Stopping web instances

# Docker service create alternative (for Swarm)
# Note: docker-compose scale is deprecated in newer versions
# Use: docker-compose up -d --scale service=N

# Check port assignments for scaled instances
docker-compose ps | grep "8000"

# Expected output:
# 0.0.0.0:8000->80
# 0.0.0.0:8001->80
# 0.0.0.0:8002->80
# 0.0.0.0:8003->80

# Stop and clean up
docker-compose down

# Expected output:
# Removing all instances
```

### Explanation

**Scaling Services:**
- `--scale service=N`: Run N instances
- Port ranges needed for multiple instances
- DNS load balances across instances
- Useful for horizontal scaling

**Limitations:**
- All instances use same configuration
- Port conflicts if not careful
- Database doesn't scale horizontally
- Suited for stateless services

---

## Exercise 10: Health checks in compose

### Solution

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  web:
    image: nginx:latest
    container_name: web-service
    ports:
      - "8080:80"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    depends_on:
      database:
        condition: service_healthy

  api:
    image: python:3.11-slim
    container_name: api-service
    ports:
      - "5000:5000"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:5000/health"]
      interval: 15s
      timeout: 5s
      retries: 2
      start_period: 10s
    command: sleep 300
    depends_on:
      - database

  database:
    image: postgres:15-alpine
    container_name: db-service
    environment:
      POSTGRES_PASSWORD: secret
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 5s

  cache:
    image: redis:7-alpine
    container_name: cache-service
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 3
```

### Commands

```bash
# Start services with health checks
docker-compose up -d

# Expected output:
# Creating services

# Watch health status
docker-compose ps

# Expected output initially:
# NAME              STATUS              HEALTH
# web-service       Up 5 seconds         (health: starting)
# api-service       Up 5 seconds         (no healthcheck output)
# db-service        Up 5 seconds         (health: starting)
# cache-service     Up 5 seconds         (health: starting)

# Wait and check again
sleep 30
docker-compose ps

# Expected output:
# web-service       Up 35 seconds        (health: healthy)
# db-service        Up 35 seconds        (health: healthy)
# cache-service     Up 35 seconds        (health: healthy)

# Inspect health status in detail
docker inspect web-service | jq '.[] | .State.Health'

# Expected output:
# {
#   "Status": "healthy",
#   "FailingStreak": 0,
#   "Passes": 2,
#   "Fails": 0,
#   "Log": [ ... ]
# }

# View health check logs
docker exec web-service curl -f http://localhost/

# Expected output:
# (nginx default page)

# Simulate failure (stop nginx)
docker exec web-service nginx -s stop

# Wait for health check to fail
sleep 15

# Check status
docker-compose ps | grep web-service

# Expected output:
# web-service       Up 45 seconds        (health: unhealthy)

# Restart service
docker-compose restart web-service

# Check recovery
sleep 10
docker-compose ps | grep web-service

# Expected output:
# web-service       Up 5 seconds         (health: healthy)

# Monitor health over time
watch -n 5 'docker-compose ps'

# Press Ctrl+C to exit

# Health check logs
docker inspect web-service | jq '.[] | .State.Health.Log[-3:]'

# Expected output shows last 3 health checks

# Test database health
docker exec db-service pg_isready -U postgres

# Expected output:
# accepting connections

# Test cache health
docker exec cache-service redis-cli ping

# Expected output:
# PONG

# Get overall compose health status
docker-compose ps --format "table {{.Service}}\t{{.Status}}\t{{.Health}}"

# Expected output:
# SERVICE    STATUS         HEALTH
# web        Up 1 minute    healthy
# api        Up 1 minute    (no healthcheck)
# database   Up 1 minute    healthy
# cache      Up 1 minute    healthy

# Stop and remove
docker-compose down

# Expected output:
# Removing services
```

### Explanation

**Health Check Parameters:**
- `test`: Command to run (CMD, CMD-SHELL, NONE)
- `interval`: Check frequency (default 30s)
- `timeout`: Max time for check (default 30s)
- `retries`: Failures before unhealthy (default 3)
- `start_period`: Grace period before checking (default 0)

**Health Check Types:**
- `["CMD", ...]`: Exec form, most reliable
- `["CMD-SHELL", "..."]`: Shell form, runs in /bin/sh
- `NONE`: Disable health check

**Benefits:**
1. Automatic restart of unhealthy services
2. Better dependency management with condition
3. Monitoring and alerting integration
4. Service readiness verification