# Module 09: Exercises, Solutions, Cheatsheet & Quiz

## Exercises

1. **View container logs with tail**: Run a container that outputs text every second. Capture the last 5 lines of logs using `docker logs --tail 5`.
2. **Follow logs in real-time**: Follow logs as they appear using `docker logs -f` on a running container.
3. **Timestamp logs**: View logs with timestamps using `docker logs --timestamps`.
4. **Inspect container state**: Use `docker inspect [container]` to retrieve the container's State field.
5. **Check network connectivity**: Use `docker exec` to ping another container on the same network.
6. **Monitor resource usage**: Use `docker stats` to view CPU, memory, and network stats.
7. **View running processes**: Use `docker top [container]` to list processes inside a container.
8. **Debug using interactive shell**: Run an interactive bash shell inside a running container using `docker exec -it`.
9. **Inspect network configuration**: Use `docker network inspect [network]` to verify container connectivity.
10. **Analyze container events**: Use `docker events` to monitor container creation, startup, and shutdown events.

## Solutions

1. **Tail logs**: `docker logs --tail 5 [container]` - Shows last 5 log lines. Useful for quick error checks.
2. **Follow logs**: `docker logs -f [container]` - Streams logs live. Press Ctrl+C to exit.
3. **Timestamp logs**: `docker logs --timestamps [container]` - Adds RFC3339 timestamps to each line.
4. **Inspect state**: `docker inspect [container] | jq '.State'` - Shows Running, Paused, Restarting status.
5. **Network debugging**: `docker exec [container] ping [other-container]` - Tests connectivity. Requires DNS resolution.
6. **Monitor stats**: `docker stats [container]` - Updates every 2 seconds. Shows CPU %, memory, I/O.
7. **List processes**: `docker top [container]` - Shows UID, PID, PPID, and command for each process.
8. **Interactive shell**: `docker exec -it [container] bash` - Allows manual exploration inside container.
9. **Network inspect**: `docker network inspect [network]` - Shows connected containers and their IPs.
10. **Monitor events**: `docker events --filter type=container` - Shows real-time container events.

## Cheatsheet

| Command | Purpose |
|---------|---------|
| `docker logs [container]` | View container logs |
| `docker logs -f [container]` | Follow logs in real-time |
| `docker logs --tail 10 [container]` | View last 10 lines |
| `docker logs --timestamps [container]` | Add timestamps to logs |
| `docker inspect [container]` | View container details |
| `docker stats [container]` | Monitor CPU, memory usage |
| `docker top [container]` | List running processes |
| `docker exec -it [container] bash` | Interactive shell access |
| `docker network inspect [network]` | Inspect network details |
| `docker events --filter type=container` | Monitor container events |

## Quiz

**Answers:**

1. What command shows the last 20 lines of logs?
   - Answer: `docker logs --tail 20 [container]`

2. How do you monitor real-time resource usage?
   - Answer: `docker stats [container]`

3. What command lists processes inside a container?
   - Answer: `docker top [container]`

4. How do you get an interactive shell in a running container?
   - Answer: `docker exec -it [container] bash`

5. What flag shows timestamps in logs?
   - Answer: `--timestamps`

6. How do you monitor container events in real-time?
   - Answer: `docker events --filter type=container`

7. What command inspects network configuration?
   - Answer: `docker network inspect [network]`

8. **(Short Answer)** Describe the debugging workflow when a container exits unexpectedly.
   - Answer: Check logs (`docker logs`), inspect state (`docker inspect`), review configuration, and check resource limits.

9. **(Short Answer)** How would you debug a networking issue between two containers?
   - Answer: Use `docker network inspect` to verify connection, `docker exec` to ping between containers, and check DNS resolution.

10. **(Short Answer)** What tools would you use to profile container performance issues?
    - Answer: `docker stats` for resource usage, `docker top` for processes, `docker logs` for errors, and application-level profiling.

Passing Score: 7/10
