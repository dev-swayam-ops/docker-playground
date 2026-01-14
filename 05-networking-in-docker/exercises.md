# Module 05: Networking - Exercises, Solutions, Cheatsheet, Quiz

## Exercises (10)
1. Create custom bridge network
2. Run containers on custom network  
3. Test DNS between containers
4. Publish ports with -p flag
5. Expose multiple ports
6. Connect running container to network
7. Disconnect container from network
8. Use --network=host mode
9. Inspect network details
10. Create overlay network (Swarm)

## Solutions
- Create: `docker network create my-net`
- Run: `docker run --network my-net nginx`
- DNS works automatically in custom bridge
- Port: `docker run -p 8080:80 nginx`
- Connect: `docker network connect my-net [container]`
- Host mode: `docker run --network=host nginx`

## Cheatsheet

| Command | Purpose |
|---------|---------|
| `docker network create [name]` | Create network |
| `docker network ls` | List networks |
| `docker network inspect [name]` | Inspect network |
| `docker network rm [name]` | Remove network |
| `docker run --network [net]` | Run on network |
| `docker network connect [net] [container]` | Connect |
| `docker network disconnect [net] [container]` | Disconnect |
| `docker run -p [host]:[container]` | Port map |
| `docker run --network=host` | Host network mode |
| `docker run --network=none` | No network |

## Quiz (7 MCQ + 3 Short)

**MCQ1:** Default network type?
A) Bridge B) Host C) Overlay D) None
**Answer:** A

**MCQ2:** DNS in default bridge?
A) Works automatically B) Needs setup C) Not available D) Manual config
**Answer:** C (default bridge has no DNS)

**MCQ3:** Custom bridge DNS?
A) Same as default B) Automatic C) Manual D) Not available
**Answer:** B

**MCQ4:** -p 8080:80 means?
A) Host:Container B) Container:Host C) Both same D) Random
**Answer:** A

**MCQ5:** --network=host effect?
A) Isolated network B) Shares host network C) Custom network D) No network
**Answer:** B

**MCQ6:** Container to container communication?
A) IP addresses B) Container names B) DNS C) Same network
**Answer:** D

**MCQ7:** Publish port must use?
A) -p B) -P C) --expose D) All work
**Answer:** A

**Short1:** When use --network=host?
**Answer:** Performance critical apps, access host network directly, special networking.

**Short2:** How containers discover each other?
**Answer:** In custom bridge: automatic DNS (container names resolve). Default bridge: use IP addresses.

**Short3:** Port publishing vs EXPOSE?
**Answer:** EXPOSE: documentation in Dockerfile. -p: actually publishes to host.
