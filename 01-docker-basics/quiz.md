# Quiz: Docker Basics

Test your knowledge of Docker commands and container management!

## Instructions
- **MCQ Questions (1-7):** Choose the best answer
- **Short Answer (8-10):** Write brief responses
- **Time Limit:** 20-25 minutes
- **Passing Score:** 70% (7+ correct)
- **Resources:** [cheatsheet.md](cheatsheet.md), [README.md](README.md), [solutions.md](solutions.md)

---

## Multiple Choice Questions (7)

### Question 1: Container vs Image
**What is the main difference between a Docker image and a Docker container?**

A) An image is a running container; a container is a stored image  
B) An image is a template/blueprint; a container is a running instance  
C) Images and containers are the same thing  
D) Containers are smaller than images  

**Answer:** B) An image is a template/blueprint; a container is a running instance  
**Explanation:** An image is immutable and serves as a blueprint. A container is a running (or stopped) instance created from that image.

---

### Question 2: Running a Container Interactively
**What do the `-it` flags do when running a container?**

A) `-i` sets input buffer, `-t` sets timeout  
B) `-i` keeps STDIN open, `-t` allocates a pseudo-TTY (terminal)  
C) `-i` and `-t` do the same thing  
D) They are optional flags with no effect  

**Answer:** B) `-i` keeps STDIN open, `-t` allocates a pseudo-TTY (terminal)  
**Explanation:** Using `-it` together allows you to interact with the container's shell in your terminal, like typing commands.

---

### Question 3: Container Naming
**Which command creates a new container named "mywebapp" from the nginx image?**

A) `docker run --name mywebapp nginx`  
B) `docker create mywebapp nginx`  
C) `docker run mywebapp --name nginx`  
D) `docker name mywebapp nginx`  

**Answer:** A) `docker run --name mywebapp nginx`  
**Explanation:** The `--name` flag must come before the image name. Proper format: `docker run --name [name] [image]`

---

### Question 4: Starting vs Running
**What is the difference between `docker run` and `docker start`?**

A) They are the same command  
B) `docker run` creates a new container; `docker start` starts an existing stopped container  
C) `docker run` is slower than `docker start`  
D) `docker start` can only be used with named containers  

**Answer:** B) `docker run` creates a new container; `docker start` starts an existing stopped container  
**Explanation:** `docker run` always creates a NEW container from an image. `docker start` resumes a previously created but stopped container.

---

### Question 5: Port Mapping
**In the command `docker run -p 8080:80 nginx`, what does the mapping mean?**

A) Host port 80 connects to container port 8080  
B) Host port 8080 connects to container port 80  
C) Both ports must be the same  
D) The ports are unrelated  

**Answer:** B) Host port 8080 connects to container port 80  
**Explanation:** The format is `-p [host_port]:[container_port]`. Requests to localhost:8080 on your computer go to port 80 inside the container.

---

### Question 6: Container Logs
**You want to view the output of a running container and continuously watch for new messages. Which command should you use?**

A) `docker logs [container]`  
B) `docker logs -f [container]`  
C) `docker view [container]`  
D) `docker output [container]`  

**Answer:** B) `docker logs -f [container]`  
**Explanation:** The `-f` flag stands for "follow" (like `tail -f` in Linux). It continuously streams new output.

---

### Question 7: Executing Commands in Running Container
**You have a running container named "database". How do you execute an SQL query without entering the interactive shell?**

A) `docker run database query`  
B) `docker exec database mysql -u root < query.sql`  
C) `docker start database mysql -u root < query.sql`  
D) `docker command database mysql -u root < query.sql`  

**Answer:** B) `docker exec database mysql -u root < query.sql`  
**Explanation:** `docker exec` runs commands in an existing running container. `docker run` would create a new container.

---

## Short Answer Questions (3)

### Question 8: Container Lifecycle States
**Describe the main states a Docker container goes through in its lifecycle, from creation to removal. What commands transition between these states?**

**Model Answer:**
A container goes through these states:
1. **Created** - Container exists but hasn't started → `docker start`
2. **Running** - Container is actively executing → `docker stop` or `docker pause`
3. **Stopped/Exited** - Container has stopped → `docker start`, `docker restart`, or `docker rm`
4. **Removed** - Container deleted from system

Transitions:
- Created → Running: `docker start`
- Running → Stopped: `docker stop`
- Stopped → Running: `docker start`
- Any state → Removed: `docker rm` (stop first if running)

**Grading:** Full points for mentioning:
- At least 3 main states
- One transition command per state
- Understanding that removal is permanent

---

### Question 9: Debugging a Stopped Container
**A container named "app" suddenly stopped. List three ways to find out why it exited.**

**Model Answer:**
1. **Check logs:** `docker logs app` - View the output/errors before exit
2. **Inspect exit code:** `docker inspect --format='{{.State.ExitCode}}' app` - Returns 0 for success, non-zero for errors
3. **Check full state information:** `docker inspect app` - Look at State section for details
4. **Verify container exists:** `docker ps -a` - Confirm container is in Exited state

Alternative good answers:
- `docker logs app` (primary debugging method)
- Look at last few lines: `docker logs --tail 50 app`
- Check timestamps: `docker logs --timestamps app`

**Grading:** Full points for any 3 valid debugging methods that actually show information about why the container stopped.

---

### Question 10: Cleanup Operations
**You want to remove all stopped containers from your system. Write the command and explain what each part does.**

**Model Answer:**
```bash
docker rm $(docker ps -a -q)
```

**Explanation of each part:**
- `docker ps -a` - Lists all containers (running and stopped)
- `-q` - Shows only container IDs (quiet mode)
- `$(...)` - Command substitution; passes the output to the next command
- `docker rm` - Removes the containers
- Combined: Passes all container IDs to `docker rm`, removing them all

**Alternative approach:**
```bash
docker container prune
```
Removes all stopped containers interactively (asks for confirmation)

**Grading:** Full points for:
- Correct command syntax
- Understanding what `-q` does
- Understanding command substitution `$(...)`
- Mentioning that running containers must be stopped first (or use `docker rm -f`)

---

## Answer Key Summary

| Question | Answer | Difficulty | Topic |
|----------|--------|------------|-------|
| 1 | B | Easy | Image vs Container |
| 2 | B | Easy | Container Interaction |
| 3 | A | Easy | Container Naming |
| 4 | B | Medium | Lifecycle Commands |
| 5 | B | Medium | Port Mapping |
| 6 | B | Medium | Logging |
| 7 | B | Medium | Executing Commands |
| 8 | Essay | Medium | Lifecycle States |
| 9 | Essay | Medium | Debugging |
| 10 | Essay | Easy | Cleanup |

---

## Scoring Guide

**MCQ (Questions 1-7):**
- 1 point per correct answer
- Maximum: 7 points

**Short Answer (Questions 8-10):**
- 1 point per question for complete answer
- 0.5 points for partial/incomplete answers
- Maximum: 3 points

**Total Score: 10 points**

**Scoring Levels:**
- 9-10 points: Excellent! Ready for Module 02
- 7-8 points: Good understanding! Review weak areas
- 5-6 points: Decent progress. Redo exercises before proceeding
- Below 5: Review [README.md](README.md) and [exercises.md](exercises.md) again

---

## Self-Assessment Checklist

After the quiz, can you:

- [ ] Run a container interactively with `-it` flags?
- [ ] List all containers using `docker ps -a`?
- [ ] Start and stop containers?
- [ ] Execute commands in running containers with `docker exec`?
- [ ] View container logs and follow them in real-time?
- [ ] Understand port mapping (host:container)?
- [ ] Inspect container configuration details?
- [ ] Monitor container resource usage?
- [ ] Clean up containers properly?
- [ ] Debug why a container stopped?

**If you answered "No" to any question:**
- Review [README.md](README.md) for concepts
- Re-do relevant [exercises.md](exercises.md) exercises
- Check [solutions.md](solutions.md) for detailed explanations
- Use [cheatsheet.md](cheatsheet.md) for quick reference

---

## Common Quiz Mistakes

### ❌ Mistake 1: Confusing docker run and docker start
- **Wrong:** "docker run starts a stopped container"
- **Right:** "docker run creates a NEW container; docker start resumes a stopped one"

### ❌ Mistake 2: Port mapping format
- **Wrong:** "-p 80:8080 maps host:8080 to container:80"
- **Right:** "-p 8080:80 maps host:8080 to container:80"

### ❌ Mistake 3: What -it does
- **Wrong:** "-i and -t are interchangeable"
- **Right:** "-i keeps STDIN open; -t allocates terminal; both needed for interaction"

### ❌ Mistake 4: Logs vs real-time monitoring
- **Wrong:** "docker logs shows live updates like docker stats"
- **Right:** "docker logs -f shows output from past + new; docker stats shows CPU/memory now"

### ❌ Mistake 5: Removing containers
- **Wrong:** "You can remove a container while it's running"
- **Right:** "Must stop first, or use docker rm -f to force"

---

## Helpful Comparison Tables

### docker run vs docker start

| Aspect | docker run | docker start |
|--------|-----------|-------------|
| Creates Container | Yes | No |
| Starts Existing | No | Yes |
| Use When | First time with image | Resume stopped container |
| Example | `docker run ubuntu` | `docker start myapp` |

### Logging Commands

| Command | Shows | Updates |
|---------|--------|---------|
| `docker logs` | All past logs | Once (snapshot) |
| `docker logs -f` | Past + new | Continuously (follow) |
| `docker stats` | CPU/Memory | Live (real-time) |
| `docker top` | Running processes | Once (snapshot) |

### Container Reference

| To Reference | Use | Examples |
|-------------|-----|----------|
| Specific container | Name or ID | `docker logs myapp` or `docker logs a1b2c3...` |
| All containers | `$(docker ps -a -q)` | `docker rm $(docker ps -a -q)` |
| Running only | `$(docker ps -q)` | `docker stop $(docker ps -q)` |

---

## Practice Questions (Not Graded)

Try these additional questions to strengthen your understanding:

**Q: What happens when you exit an interactive container?**
A: The container stops/exits but still exists on disk. You can restart it with `docker start`.

**Q: Can you have two containers with the same name?**
A: No, names must be unique. You'll get an error trying to create a second container with the same name.

**Q: What's the difference between docker stop and docker kill?**
A: `docker stop` sends SIGTERM (graceful shutdown); `docker kill` sends SIGKILL (forced termination).

**Q: Can multiple containers run from the same image?**
A: Yes, you can create many containers from one image. Each is independent.

**Q: Where are Docker logs stored?**
A: Docker logs are stored on the host filesystem in Docker's data directory, not inside the container.

---

## Next Steps After Quiz

**If you scored 7+:**
✅ Great! Proceed to [Module 02: Images and Dockerfile](../02-images-and-dockerfile/)

**If you scored below 7:**
📚 Review these resources:
1. Re-read [README.md](README.md) focusing on weak concepts
2. Complete any exercises you skipped
3. Study [solutions.md](solutions.md) in detail
4. Use [cheatsheet.md](cheatsheet.md) as reference
5. Retake this quiz after review

---

## Additional Resources

**For More Practice:**
- Docker Labs: https://www.docker.com/101-tutorial/
- Play with Docker: https://www.docker.com/play-with-docker
- Docker Documentation: https://docs.docker.com/reference/cli/docker/

**For Questions:**
- Docker Community: https://community.docker.com/
- Stack Overflow: Tag with `docker`
- GitHub Issues: https://github.com/moby/moby/issues

---

**Good luck! 🚀**

Remember: Docker basics are essential. Master these commands and you'll be comfortable with all advanced Docker topics!

