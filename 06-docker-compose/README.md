# Module 06: Docker Compose

Define and run multi-container Docker applications.

## What You'll Learn

- Docker Compose syntax and structure
- Services, networks, volumes in compose
- Environment variables and configs
- Dependency management
- Scaling services
- Production vs development configurations

## Key Concepts

**Service:** A container running application code
**Compose File:** YAML file defining all services
**Network:** Automatic networking between services
**Volume:** Data persistence in compose

## Lab: Basic Compose File

```yaml
version: '3.8'
services:
  web:
    image: nginx:latest
    ports:
      - "8080:80"
    depends_on:
      - db
  
  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: secret
    volumes:
      - db_data:/var/lib/postgresql/data

volumes:
  db_data:
```

```bash
# Run
docker-compose up -d

# Stop
docker-compose down

# Logs
docker-compose logs -f web
```

## Quick Commands

| Command | Purpose |
|---------|---------|
| `docker-compose up -d` | Start services |
| `docker-compose down` | Stop and remove |
| `docker-compose ps` | List services |
| `docker-compose logs [service]` | View logs |
| `docker-compose scale web=3` | Scale service |
| `docker-compose build` | Build images |
| `docker-compose restart [service]` | Restart service |

---

**Exercises:** [exercises.md](exercises.md)

