# Quiz: Containers and Lifecycle

## Multiple Choice (7 Questions)

**Q1: Health Check Purpose**
A) Auto-stops bad containers  
B) Marks container as healthy/unhealthy  
C) Replaces containers  
D) Deletes data  
**Answer:** B

**Q2: Memory Limit Flag**
A) --memory-max  
B) --memory  
C) --ram  
D) --mem  
**Answer:** B

**Q3: Restart Policy unless-stopped**
A) Never restart  
B) Always restart  
C) Restart except when manually stopped  
D) Restart on failure  
**Answer:** C

**Q4: docker pause Effect**
A) Stops container  
B) Suspends processes  
C) Removes container  
D) Backs up  
**Answer:** B

**Q5: Live Resource Usage**
A) docker logs  
B) docker stats  
C) docker history  
D) docker inspect  
**Answer:** B

**Q6: on-failure:3 Policy**
A) Restart 3 times  
B) Restart max 3x on failure  
C) Restart after 3s  
D) 3 containers  
**Answer:** B

**Q7: docker stop Signal**
A) SIGKILL  
B) SIGTERM  
C) SIGHUP  
D) SIGINT  
**Answer:** B

## Short Answer (3 Questions)

**Q8: Health Check vs Restart Policy**
Answer: Health check monitors status (healthy/unhealthy). Restart policy automatically restarts on exit/failure.

**Q9: Graceful Shutdown Implementation**
Answer: App catches SIGTERM, cleans up resources, exits. docker stop sends SIGTERM with 15s grace period.

**Q10: Recommended Resource Limits**
Answer: Set --memory and --cpus to prevent resource monopolization. Example: --memory=512m --cpus=0.5

---

**Passing Score:** 70% (7+ points)

