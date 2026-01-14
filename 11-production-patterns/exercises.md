# Module 11: Exercises, Solutions, Cheatsheet & Quiz

## Exercises

1. **Implement health checks**: Add HEALTHCHECK instruction to a Dockerfile for a web application.
2. **Test health check**: Verify health check status using `docker inspect` with health filters.
3. **Configure auto-restart**: Use `--restart=unless-stopped` to ensure containers restart on failure.
4. **Implement rolling updates**: Update a service image without downtime using docker service update.
5. **Setup centralized logging**: Configure a container to send logs to syslog or ELK stack.
6. **Manage secrets**: Use Docker secrets to manage database credentials securely.
7. **Monitor with alerts**: Set up monitoring to alert when CPU/memory exceeds thresholds.
8. **Create backup strategy**: Implement volume backups for persistent data.
9. **Load balance traffic**: Use docker-compose with multiple replicas and external load balancer.
10. **Plan incident response**: Document procedures for common production failures.

## Solutions

1. **Health checks**: Add to Dockerfile: `HEALTHCHECK --interval=10s CMD curl http://localhost`. Validates application is responding.
2. **Test health**: `docker inspect [container] | jq '.State.Health.Status'` - Returns "healthy", "unhealthy", or "starting".
3. **Auto-restart**: `docker run --restart=unless-stopped [image]` - Restarts unless stopped manually. Prevents downtime from crashes.
4. **Rolling updates**: `docker service update --image [new-image] [service]` - Updates containers one-by-one. Requires Docker Swarm.
5. **Centralized logging**: Use log driver: `docker run --log-driver syslog [image]`. Sends all logs to central location.
6. **Secrets management**: Create secret: `docker secret create db_pass mypassfile.txt`, then use in docker-compose.yml with `secrets:` section.
7. **Monitoring alerts**: Use tools like Prometheus + Alertmanager. Set thresholds based on `docker stats` baseline data.
8. **Backup strategy**: Regular snapshots: `docker run --rm -v [vol]:/backup -v myvolume:/data busybox tar czf /backup/backup.tar.gz /data`.
9. **Load balance**: Use docker-compose with `deploy: replicas: 3` and external reverse proxy (nginx, HAProxy).
10. **Incident response**: Document runbooks for: container crashes (check logs), out-of-memory (increase limit), network issues (inspect network).

## Cheatsheet

| Command | Purpose |
|---------|---------|
| `docker run --restart=always [image]` | Auto-restart on failure |
| `docker run --health-cmd="curl http://localhost" [image]` | Enable health check |
| `docker inspect {{.State.Health.Status}} [container]` | Check health status |
| `docker service update --image [new] [service]` | Rolling update |
| `docker secret create [name] [file]` | Create secret |
| `docker logs [container]` | View container logs |
| `docker stats --no-stream [container]` | Single stats snapshot |
| `docker update --memory=1g [container]` | Update resource limits |
| `docker cp [container]:[path] [local-path]` | Backup container files |

## Quiz

**Answers:**

1. What restart policy ensures container restarts unless stopped manually?
   - Answer: `--restart=unless-stopped`

2. How do you check if a container's health check is passing?
   - Answer: `docker inspect [container] | jq '.State.Health.Status'` or `docker ps` to see health status

3. What Docker feature manages sensitive data like database passwords?
   - Answer: Docker secrets

4. How do you perform zero-downtime updates in Docker Swarm?
   - Answer: `docker service update --image [new-image] [service]`

5. What log driver sends container logs to a central server?
   - Answer: `syslog` or other centralized logging drivers

6. What HEALTHCHECK instruction verifies a web application is running?
   - Answer: `HEALTHCHECK --interval=10s CMD curl http://localhost`

7. What command backs up a Docker volume?
   - Answer: `docker run --rm -v [volume]:/backup -v [source]:/data busybox tar czf /backup/backup.tar.gz /data`

8. **(Short Answer)** Describe a rolling update process for zero-downtime deployments.
   - Answer: Stop one container at a time, replace it with new version, verify health, then move to next. External load balancer reroutes traffic.

9. **(Short Answer)** What are three monitoring metrics you should track in production?
   - Answer: CPU usage, memory usage, disk I/O, network throughput, error rates, and response times.

10. **(Short Answer)** How would you respond to a container repeatedly crashing in production?
    - Answer: Check logs for errors, inspect resource limits, verify configuration, increase limits if needed, deploy fix, and implement health checks.

Passing Score: 7/10
