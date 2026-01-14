# Exercises: Setup and Prerequisites

Complete these exercises to practice Docker installation and basic verification.

## Exercise 1: System Requirements Check
**Difficulty:** Easy

Verify your system meets Docker requirements.

**Task:**
1. Check your operating system version
2. Check available RAM on your computer
3. Check available disk space
4. Document your findings in a text file

**Hints:**
- Windows: Use `systeminfo` command
- macOS: Use `system_profiler SPSoftwareDataType` and `df -h`
- Linux: Use `uname -a`, `free -h`, and `df -h`

---

## Exercise 2: Docker Installation
**Difficulty:** Easy

Install Docker on your system.

**Task:**
1. Download Docker Desktop (or Docker CE for Linux)
2. Follow the official installation guide for your OS
3. Complete the installation process
4. Restart your computer if required

**Hints:**
- Official Download: https://www.docker.com/products/docker-desktop
- Linux users can use package managers (apt, yum, etc.)
- Don't skip the restart step!

---

## Exercise 3: Verify Docker Installation
**Difficulty:** Easy

Confirm Docker is properly installed on your system.

**Task:**
1. Open a terminal/command prompt
2. Run `docker --version`
3. Take a screenshot of the output
4. Verify the version number is displayed

**Expected Output Example:**
```
Docker version 24.0.0, build abcdef123
```

---

## Exercise 4: Check Docker Daemon Status
**Difficulty:** Easy

Ensure the Docker daemon is running.

**Task:**
1. Run `docker ps` command
2. If it fails, start Docker Desktop or Docker daemon
3. Run `docker ps` again
4. Verify it shows an empty container list (or containers if you have any)

**Expected Output Example:**
```
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

---

## Exercise 5: Run Your First Container
**Difficulty:** Easy

Execute the hello-world container to test Docker functionality.

**Task:**
1. Run `docker run hello-world`
2. Read the output carefully
3. Note the key message: "Hello from Docker!"
4. Save the full output to a file

**Expected Output Includes:**
```
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

---

## Exercise 6: Inspect Downloaded Images
**Difficulty:** Medium

View images downloaded on your system.

**Task:**
1. Run `docker images` command
2. Count how many images are on your system
3. Identify the image ID of the `hello-world` image
4. Note the size of each image
5. Create a summary with: Image name, Tag, Image ID, Size

**Expected Output Format:**
```
REPOSITORY    TAG       IMAGE ID      CREATED        SIZE
hello-world   latest    d2c94e258d...  months ago     13.3kB
```

---

## Exercise 7: Get Detailed Docker Information
**Difficulty:** Medium

Learn about your Docker installation details.

**Task:**
1. Run `docker info` command
2. Identify and document:
   - Docker Client Version
   - Docker Server Version
   - Storage Driver
   - Number of containers
   - Number of images
3. Capture the output in a text file

**Key Information to Look For:**
```
Client:
  Version: 24.0.0
Server:
  Containers: X
  Running: Y
  Images: Z
  Storage Driver: overlay2
```

---

## Exercise 8: Configure Docker for Linux (Linux Only)
**Difficulty:** Medium

Set up Docker to run without sudo on Linux systems.

**Task:**
1. Run `docker ps` with sudo: `sudo docker ps`
2. Create a docker group: `sudo groupadd docker`
3. Add your user: `sudo usermod -aG docker $USER`
4. Log out and log back in
5. Run `docker ps` without sudo
6. Verify it works without requiring a password

**Commands in Sequence:**
```bash
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
docker ps
```

---

## Exercise 9: Explore Docker Help
**Difficulty:** Medium

Get familiar with Docker command documentation.

**Task:**
1. Run `docker --help`
2. Run `docker run --help`
3. Find and document:
   - What does the `-it` flag do?
   - What does the `-d` flag do?
   - What does the `--name` flag do?
   - What does the `-p` flag do?
4. Create a quick reference guide

**Commands:**
```bash
docker --help
docker run --help
docker ps --help
docker images --help
```

---

## Exercise 10: Cleanup After Testing
**Difficulty:** Medium

Learn to manage and remove containers and images.

**Task:**
1. List all containers: `docker ps -a`
2. List all images: `docker images`
3. Remove the hello-world container (use container ID from step 1)
4. Remove the hello-world image
5. Verify they're removed with `docker ps -a` and `docker images`

**Commands:**
```bash
docker ps -a
docker images
docker rm <container_id>
docker rmi hello-world
docker ps -a
docker images
```

---

## Tips for Success

- ✅ Take notes as you complete each exercise
- ✅ Save command outputs for reference
- ✅ If a command fails, read the error message carefully
- ✅ Use `docker <command> --help` for additional options
- ✅ Don't worry if some commands fail initially - troubleshooting is part of learning!

## Next Steps

After completing these exercises:
1. Review the solutions to compare your approach
2. Try variations of the commands
3. Refer to the cheatsheet for quick lookup
4. Move to Module 01: Docker Basics

---

**Stuck?** Check [solutions.md](solutions.md) for detailed solutions and explanations.
