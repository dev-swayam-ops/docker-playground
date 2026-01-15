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




