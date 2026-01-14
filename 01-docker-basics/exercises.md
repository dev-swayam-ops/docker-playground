# Exercises: Docker Basics

Practice Docker commands and container management with these hands-on exercises.

## Exercise 1: Run Your First Container
**Difficulty:** Easy

Run a simple container and observe its behavior.

**Task:**
1. Run `docker run ubuntu:22.04 echo "Hello from Docker!"`
2. Observe the output
3. List all containers with `docker ps -a`
4. Find the container you just ran and note its ID
5. Identify the container's status (should be Exited)

**Hints:**
- This container runs once and exits immediately
- Use `docker ps -a` to see exited containers
- Container output appears on your terminal

---

## Exercise 2: Interactive Container Session
**Difficulty:** Easy

Run a container and interact with it directly.

**Task:**
1. Run `docker run -it ubuntu:22.04 bash`
2. Inside the container, execute these commands:
   - `ls /home`
   - `pwd` (print working directory)
   - `whoami` (show current user)
   - `cat /etc/os-release` (show OS info)
3. Type `exit` to leave the container
4. Verify the container is stopped with `docker ps -a`

**Expected Shell Prompt:**
```
root@<container_id>:/#
```

---

## Exercise 3: Name Your Containers
**Difficulty:** Easy

Run a container with a human-readable name.

**Task:**
1. Run `docker run -it --name my-web-app ubuntu:22.04 bash`
2. Inside the container, create a file: `echo "version 1.0" > /version.txt`
3. Exit the container
4. Run `docker ps -a` and find your container by name
5. Note: Why is the name useful compared to the container ID?

**Hints:**
- Container names must be unique (can't create a new one with same name)
- Names are easier to remember and type
- Use `docker rename` if you want to change the name

---

## Exercise 4: Container Lifecycle Management
**Difficulty:** Easy

Manage the container states: start, stop, restart.

**Task:**
1. Verify you have a stopped container named `my-web-app` from Exercise 3
2. Start the stopped container: `docker start my-web-app`
3. Verify it's running: `docker ps`
4. Stop it: `docker stop my-web-app`
5. Verify it's stopped: `docker ps -a`
6. Restart it: `docker restart my-web-app`
7. Verify it's running again: `docker ps`

**Expected Output from docker ps:**
```
CONTAINER ID   IMAGE          COMMAND   CREATED   STATUS        PORTS   NAMES
abc123...      ubuntu:22.04   "bash"    5 min ago Up 2 seconds          my-web-app
```

---

## Exercise 5: Execute Commands in Running Containers
**Difficulty:** Medium

Run commands inside containers without entering interactive mode.

**Task:**
1. Ensure `my-web-app` container is running (start it if needed)
2. Execute commands using `docker exec`:
   - `docker exec my-web-app cat /version.txt`
   - `docker exec my-web-app ls /`
   - `docker exec my-web-app whoami`
3. Document the output from each command
4. Notice: Container keeps running while you run commands

**Key Difference:**
- `docker run` creates a new container
- `docker exec` runs commands in an existing container

---

## Exercise 6: View Container Logs
**Difficulty:** Medium

Learn to retrieve and monitor container output.

**Task:**
1. Run a simple container: `docker run -d --name logger alpine sleep 100`
2. View its logs: `docker logs logger`
3. Add more output: `docker exec logger echo "New message"`
4. View logs again to see the new message
5. Follow logs in real-time: `docker logs -f logger` (press Ctrl+C to exit)
6. View last 5 lines: `docker logs --tail 5 logger`

**Commands:**
```bash
docker run -d --name logger alpine sleep 100
docker logs logger
docker logs -f logger
docker logs --tail 5 logger
```

---

## Exercise 7: Inspect Container Details
**Difficulty:** Medium

Deep dive into container configuration.

**Task:**
1. Run a container: `docker run -d --name inspector ubuntu:22.04 sleep 300`
2. Get full details: `docker inspect inspector`
3. Extract specific information:
   - IP Address: `docker inspect --format='{{.NetworkSettings.IPAddress}}' inspector`
   - Status: `docker inspect --format='{{.State.Status}}' inspector`
   - Image: `docker inspect --format='{{.Config.Image}}' inspector`
4. Document the information you find

**Useful Format Strings:**
- `{{.Id}}` - Full container ID
- `{{.State.Status}}` - Container status
- `{{.NetworkSettings.IPAddress}}` - Container IP
- `{{.Config.Image}}` - Image used

---

## Exercise 8: Monitor Container Resources
**Difficulty:** Medium

Check CPU and memory usage of containers.

**Task:**
1. Run multiple containers:
   - `docker run -d --name cpu-heavy alpine yes > /dev/null`
   - `docker run -d --name sleep-app alpine sleep 300`
2. View live statistics: `docker stats`
3. View specific container: `docker stats cpu-heavy sleep-app`
4. Document CPU and memory usage
5. Stop the containers: `docker stop cpu-heavy sleep-app`
6. Run `docker stats` again to see stopped containers aren't listed

**Expected Output:**
```
CONTAINER ID   NAME        CPU %   MEM USAGE / LIMIT
abc123...      cpu-heavy   25.5%   512 KiB / 7.727 GiB
def456...      sleep-app   0.0%    2.34 MiB / 7.727 GiB
```

---

## Exercise 9: Clean Up Containers
**Difficulty:** Medium

Learn proper cleanup and removal techniques.

**Task:**
1. List all containers: `docker ps -a`
2. Count how many containers exist
3. Stop all running containers: `docker stop $(docker ps -q)`
4. Remove all containers: `docker rm $(docker ps -a -q)`
5. Verify all containers are gone: `docker ps -a`
6. Document: Why is cleanup important?

**Commands:**
```bash
docker ps -a
docker stop $(docker ps -q)        # Stop all running
docker rm $(docker ps -a -q)       # Remove all containers
docker ps -a                       # Verify empty
```

---

## Exercise 10: Container Port Mapping (Introduction)
**Difficulty:** Medium

Map container ports to host ports.

**Task:**
1. Run nginx container with port mapping: `docker run -d --name web-server -p 8080:80 nginx`
2. Verify it's running: `docker ps`
3. Check port mapping: `docker port web-server`
4. View logs to confirm nginx started: `docker logs web-server`
5. Stop the container: `docker stop web-server`
6. Remove it: `docker rm web-server`

**Commands:**
```bash
docker run -d --name web-server -p 8080:80 nginx
docker ps
docker port web-server
docker logs web-server
docker stop web-server
docker rm web-server
```

**Port Explanation:**
- Format: `-p host_port:container_port`
- `8080:80` maps host port 8080 to container port 80
- Host: your computer, Container: inside the Docker container

---

## Tips for Success

- ✅ Save all outputs as you progress
- ✅ Use `docker ps -a` frequently to see all containers
- ✅ When stuck, use `docker logs` to debug
- ✅ Read error messages carefully
- ✅ Use container names instead of IDs for clarity
- ✅ Always stop containers before removing them

## Common Patterns

**Start interactive session:**
```bash
docker run -it --name myapp image:tag bash
```

**Run in background:**
```bash
docker run -d --name myapp image:tag
```

**Execute command in running container:**
```bash
docker exec myapp command
```

**View output:**
```bash
docker logs myapp
docker logs -f myapp  # Follow output
```

---

**Stuck?** Check [solutions.md](solutions.md) for detailed walkthroughs.

**Next:** After completing exercises, take the [quiz.md](quiz.md) to test your knowledge!
