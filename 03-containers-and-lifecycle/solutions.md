# Solutions, Cheatsheet, Quiz for Module 03

## Solutions
- **Health Check:** Use HEALTHCHECK instruction in Dockerfile or --health-cmd in docker run
- **Resource Limits:** docker run --memory=256m --cpus=0.5 [image]
- **Restart Policy:** docker run --restart=unless-stopped [image]
- **Monitor:** docker stats, docker top, docker logs

## Cheatsheet

| Command | Purpose |
|---------|---------|
| `--health-cmd` | Set health check command |
| `--memory=256m` | Limit memory to 256MB |
| `--cpus=0.5` | Limit to 50% of 1 CPU |
| `--restart=always` | Always restart on exit |
| `--restart=unless-stopped` | Restart except when manually stopped |
| `--restart=on-failure:5` | Restart max 5 times on failure |
| `docker pause [container]` | Pause processes |
| `docker unpause [container]` | Resume processes |
| `docker stats` | View CPU/memory usage |
| `docker kill [container]` | Force stop |

## Quiz (7 MCQ + 3 Short Answer)

**MCQ 1:** What does HEALTHCHECK do?  
A) Stops unhealthy containers automatically  
B) Monitors container and marks status  
C) Replaces the container  
D) Only logs information  
**Answer:** B

**MCQ 2:** How to limit container memory?  
A) --memory-max  
B) --memory  
C) --ram  
D) --mem-limit  
**Answer:** B

**MCQ 3:** What does --restart=unless-stopped mean?  
A) Never restart  
B) Always restart  
C) Restart except when manually stopped  
D) Restart only on failure  
**Answer:** C

**MCQ 4:** docker pause does what?  
A) Stops container  
B) Suspends processes (container still running)  
C) Removes container  
D) Backs up container  
**Answer:** B

**MCQ 5:** How to see resource usage live?  
A) docker logs  
B) docker stats  
C) docker history  
D) docker inspect  
**Answer:** B

**MCQ 6:** Restart policy on-failure:3 means?  
A) Restart 3 times total  
B) Restart max 3 times on failure  
C) Restart after 3 seconds  
D) Restart 3 containers  
**Answer:** B

**MCQ 7:** What signal does docker stop send?  
A) SIGKILL  
B) SIGTERM  
C) SIGHUP  
D) SIGINT  
**Answer:** B

**Short Answer 1:** Explain health check vs restart policy  
**Answer:** Health check monitors status (marks healthy/unhealthy). Restart policy restarts container on exit/failure.

**Short Answer 2:** How to configure graceful shutdown?  
**Answer:** Use SIGTERM handler in app, docker stop sends SIGTERM (15sec grace period), app cleans up

**Short Answer 3:** What resource limits should you set?  
**Answer:** --memory for max RAM, --cpus for CPU cores, set limits to prevent resource monopolization

