# Module 12: Exercises, Solutions, Cheatsheet & Quiz

## Exercises

1. **Multi-container setup**: Build a docker-compose file with frontend (nginx), backend (Node/Python), and database (PostgreSQL).
2. **Environment management**: Use multiple docker-compose files (.env, docker-compose.yml, docker-compose.prod.yml) for dev/prod.
3. **Build and push pipeline**: Create a Dockerfile, build locally, tag, and push to Docker Hub (simulating CI/CD).
4. **Health check integration**: Add health checks to all services and verify them reporting correct status.
5. **Networking verification**: Verify services can communicate using DNS names (frontend to api to db).
6. **Volume management**: Create persistent volumes for database data and verify data survives container restart.
7. **Log aggregation**: Collect logs from all services and view them with `docker-compose logs`.
8. **Network isolation**: Create separate networks for frontend and api tiers; verify frontend cannot directly access database.
9. **Rolling restart**: Use docker-compose to update an image and perform rolling restart with zero downtime.
10. **Production checklist**: Document and implement security, monitoring, backup, and incident response procedures.

## Solutions

1. **Multi-container compose**: Define 3 services (frontend, api, db) with image/build, ports, environment, and depends_on fields. Frontend on port 80, api on 5000, db on 5432.
2. **Environment files**: Create `.env` with shared variables, `docker-compose.yml` for dev, `.prod.yml` override for production. Run with `docker-compose -f docker-compose.yml -f .prod.yml up`.
3. **CI/CD pipeline**: Execute `docker build -t myapp:latest .`, `docker tag myapp:latest user/myapp:latest`, `docker push user/myapp:latest`. Automate with GitHub Actions/GitLab CI.
4. **Health checks**: Add to each service: `healthcheck: test: ["CMD", "curl", "-f", "http://localhost/health"]`. Verify with `docker-compose ps` or `docker inspect`.
5. **Network testing**: Use `docker exec api ping db` to test. Docker Compose creates network automatically; services resolve via DNS names.
6. **Persistent volumes**: Add `volumes: db_data:` under db service, then reference with `volumes: - db_data:/var/lib/postgresql/data`. Data persists across restarts.
7. **Log aggregation**: Run `docker-compose logs -f` to stream all logs. Add `--tail 100` to show last 100 lines.
8. **Network isolation**: Create two networks: `docker-compose.yml` defines frontend on `external-net`, api and db on `internal-net`. Frontend cannot reach db directly.
9. **Rolling restart**: Update image in compose file, run `docker-compose up -d --no-deps --build [service]`. Each container updates independently.
10. **Production checklist**: Include: non-root users (RUN useradd -m appuser), resource limits (--memory), logging (log-driver), secrets management, health checks, monitoring alerts.

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

## Quiz

**Answers:**

1. What docker-compose field defines service dependencies?
   - Answer: `depends_on`

2. How do you provide different configuration for production vs development?
   - Answer: Use multiple compose files (.env, docker-compose.yml, docker-compose.prod.yml) and merge with -f flag

3. What command scales a service to multiple replicas?
   - Answer: `docker-compose up -d --scale [service]=[count]`

4. How do services on same docker-compose network communicate?
   - Answer: By service name as DNS hostname (e.g., api service accessible as "api")

5. What docker-compose command shows all running containers and their health status?
   - Answer: `docker-compose ps`

6. How do you execute a command inside a running service?
   - Answer: `docker-compose exec [service] [command]`

7. What flag removes volumes when stopping docker-compose?
   - Answer: `-v` or `--volumes`

8. **(Short Answer)** Describe a blue-green deployment strategy using Docker.
   - Answer: Run two complete environments (blue and green). Deploy new code to inactive environment, test, then switch traffic to it.

9. **(Short Answer)** What security considerations are critical for production Docker deployments?
   - Answer: Non-root users, image scanning, secrets management, resource limits, network isolation, and regular updates.

10. **(Short Answer)** How would you implement automated backups of database volumes in production?
    - Answer: Use docker volumes backup command on schedule, store backups remotely, test restore process regularly, document recovery procedures.

Passing Score: 7/10
