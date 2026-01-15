## Cheatsheet

| Command | Purpose |
|---------|---------|
| `docker-compose up -d` | Start all services in background |
| `docker-compose ps` | Show all containers and health status |
| `docker-compose logs -f [service]` | Follow logs for specific service |
| `docker-compose down -v` | Stop and remove all services and volumes |
| `docker-compose build --no-cache [service]` | Rebuild service image |
| `docker-compose exec [service] bash` | Execute command in running service |
| `docker-compose up -d --scale api=3` | Scale service to 3 replicas |
| `docker network create [network]` | Create custom network |
| `docker secret create [name] [file]` | Create secret for sensitive data |