# Module 06: Docker Compose - Exercises & Support

## Exercises (10)
1. Create simple docker-compose.yml (web + db)
2. Define services with images and ports
3. Set environment variables
4. Add volumes to services
5. Define networks
6. Use depends_on for startup order
7. Build images from Dockerfile
8. Override compose settings with environment
9. Scale services with scale command
10. Health checks in compose

## Solutions
- compose.yml structure: version, services (image, ports, env, volumes)
- Networks: automatic (service names resolve via DNS)
- Volumes: named or bind mounts
- Dependencies: depends_on service_name
- Build: build: . or build: context, dockerfile

## Cheatsheet

| Command | Purpose |
|---------|---------|
| `docker-compose up -d` | Start services |
| `docker-compose down` | Stop and remove |
| `docker-compose ps` | List services |
| `docker-compose logs -f` | Follow logs |
| `docker-compose exec [service] [cmd]` | Run command |
| `docker-compose build` | Build images |
| `docker-compose scale [service]=N` | Scale |
| `docker-compose restart [service]` | Restart |
| `docker-compose pause` | Pause services |
| `docker-compose unpause` | Resume services |

## Quiz (7 MCQ + 3 Short)
Q1: Compose file format?
A) JSON B) YAML C) XML D) TOML
**Answer:** B

Q2: Service name resolution?
A) IP required B) Automatic DNS C) Manual config D) Not available
**Answer:** B

Q3: depends_on does what?
A) Waits for service B) Starts after C) Documents dependency D) All
**Answer:** C (documents, doesn't wait)

Q4: Override docker-compose with?
A) Config file B) Command line -e C) Environment vars D) None
**Answer:** C

Q5: Scale command syntax?
A) scale service 3 B) scale service=3 C) scale=service:3 D) 3-scale service
**Answer:** B

Q6: Compose networks automatic?
A) No B) Manual only C) Yes, automatic D) requires config
**Answer:** C

Q7: docker-compose.override.yml?
A) Overrides default B) Used in dev C) Local config D) All above
**Answer:** D

Short1: When use docker-compose override?
**Answer:** Development environments, local config overrides, sensitive data.

Short2: Services communicate how?
**Answer:** Via automatic DNS - service names resolve to container IPs on same network.

Short3: Production best practices?
**Answer:** Use override files, separate compose files, secrets management, health checks.
