# Module 09: Troubleshooting & Debugging

Debug and fix Docker container issues effectively.

## What You'll Learn

- Reading and analyzing logs
- Inspecting container state
- Debugging network issues
- Analyzing container performance
- Common error patterns
- Debugging tools and techniques
- Monitoring container health

## Lab

```bash
# View logs
docker logs [container] -f

# Inspect state
docker inspect [container] | jq .State

# Debug network
docker exec [container] ping other-container
docker network inspect [network]

# Monitor performance
docker stats [container]

# Check processes
docker top [container]
```

## Debugging Steps

1. Check logs: `docker logs [container]`
2. Inspect state: `docker inspect [container]`
3. Check network: `docker network inspect [net]`
4. Monitor resources: `docker stats`
5. Check processes: `docker top`
6. Verify configuration: `docker inspect --format`

---

**Exercises:** [exercises.md](exercises.md)
This folder contains information about troubleshooting and debugging Docker applications.
