# Quiz: Setup and Prerequisites

Test your knowledge of Docker setup and installation!

## Instructions
- **MCQ Questions (1-7):** Choose the best answer
- **Short Answer (8-10):** Write brief responses
- **Time Limit:** 15-20 minutes
- **Passing Score:** 70% (7+ correct)
- **Use:** [cheatsheet.md](cheatsheet.md) and [README.md](README.md) as references

---

## Multiple Choice Questions (7)

### Question 1: System Requirements
**What is the minimum RAM requirement for Docker?**

A) 2 GB  
B) 4 GB  
C) 8 GB  
D) 16 GB  

**Answer:** B) 4 GB  
**Explanation:** Docker requires a minimum of 4 GB RAM. 8 GB is recommended for comfortable development work.

---

### Question 2: Installation Method
**Which of the following is the recommended way to install Docker on Windows 10?**

A) Install Docker Engine separately  
B) Download and run Docker Desktop  
C) Install from source code  
D) Use Windows Subsystem for Linux only  

**Answer:** B) Download and run Docker Desktop  
**Explanation:** Docker Desktop is the official, recommended installer for Windows and macOS. It includes Docker Engine, CLI, and additional tools.

---

### Question 3: Docker Daemon
**What is the Docker daemon?**

A) A command-line interface for Docker  
B) An image repository  
C) A background service that manages containers  
D) A programming language  

**Answer:** C) A background service that manages containers  
**Explanation:** The Docker daemon (dockerd) is the background process that manages container lifecycle, images, networks, and storage.

---

### Question 4: Linux Permissions
**On Linux, how do you allow a user to run Docker commands without sudo?**

A) `sudo usermod -aG docker $USER`  
B) `chmod 777 docker`  
C) Disable the firewall  
D) Run Docker as a regular user  

**Answer:** A) `sudo usermod -aG docker $USER`  
**Explanation:** Adding the user to the docker group grants access to the Docker socket. The user must log out and back in for changes to take effect.

---

### Question 5: Verification Command
**Which command shows the version of both Docker CLI and daemon?**

A) `docker --version`  
B) `docker version`  
C) `docker info`  
D) `docker ps`  

**Answer:** B) `docker version`  
**Explanation:** `docker version` shows both Client (CLI) and Server (daemon) versions. `docker --version` only shows CLI version.

---

### Question 6: Container vs Image
**What is the relationship between a Docker image and a container?**

A) They are the same thing  
B) An image is a blueprint; a container is a running instance  
C) A container is a blueprint; an image is a running instance  
D) Images and containers are independent concepts  

**Answer:** B) An image is a blueprint; a container is a running instance  
**Explanation:** Images are immutable templates. Containers are instances created and run from images.

---

### Question 7: First Container
**What happens when you run `docker run hello-world` for the first time?**

A) Docker creates an image locally first  
B) Docker pulls the image from Docker Hub, creates a container, and runs it  
C) Docker connects to the internet and streams the image  
D) Docker looks for the image; if not found, it fails  

**Answer:** B) Docker pulls the image from Docker Hub, creates a container, and runs it  
**Explanation:** On first run, Docker automatically pulls (downloads) the image from Docker Hub's registry if it's not available locally.

---

## Short Answer Questions (3)

### Question 8: Docker vs Virtual Machines
**In 2-3 sentences, explain how Docker containers differ from virtual machines.**

**Model Answer:**
Docker containers are lightweight applications that share the host OS kernel, making them faster and smaller than virtual machines. Virtual machines include a complete operating system for each instance, making them heavier and slower. Containers are more efficient for deployment and resource usage.

**Grading:** Full points for mentioning:
- Containers share OS kernel, VMs have separate OS
- Containers are lighter/faster
- Containers use fewer resources

---

### Question 9: Installation Verification Steps
**List three commands you would run to verify Docker is properly installed.**

**Model Answer:**
1. `docker --version` - Verify CLI is installed
2. `docker run hello-world` - Test Docker daemon and container functionality
3. `docker ps` or `docker images` - Confirm Docker system is working

**Grading:** Full points for three different verification commands that check different aspects.

---

### Question 10: Troubleshooting Permission Error
**You see this error: "Got permission denied while trying to connect to Docker daemon" on Linux. What is the most likely cause and how would you fix it?**

**Model Answer:**
**Cause:** Your user is not in the docker group or doesn't have permission to access the Docker socket (`/var/run/docker.sock`).

**Solution:**
```bash
sudo usermod -aG docker $USER
newgrp docker
```
Or log out and back in for the group change to take effect.

**Grading:** Full points for:
- Identifying the permission/socket issue
- Providing the correct `usermod` command
- Mentioning the need to activate group changes

---

## Answer Key Summary

| Question | Answer | Difficulty |
|----------|--------|------------|
| 1 | B | Easy |
| 2 | B | Easy |
| 3 | C | Medium |
| 4 | A | Medium |
| 5 | B | Medium |
| 6 | B | Easy |
| 7 | B | Medium |
| 8 | Essay | Medium |
| 9 | Essay | Easy |
| 10 | Essay | Medium |

---

## Scoring Guide

**MCQ (Questions 1-7):**
- 1 point per correct answer
- Maximum: 7 points

**Short Answer (Questions 8-10):**
- 1 point per question if answer matches model answer
- 0.5 points for partial/incomplete answers
- Maximum: 3 points

**Total Score: 10 points**

**Scoring Levels:**
- 9-10 points: Excellent! Ready for Module 01
- 7-8 points: Good! Review weak areas, then proceed
- 5-6 points: Needs review. Go back to README and exercises
- Below 5: Please review all materials before proceeding

---

## Self-Assessment Questions

After completing the quiz, ask yourself:

✅ **Can you...**
- [ ] Install Docker on your operating system?
- [ ] Verify Docker is working correctly?
- [ ] Explain what a container is?
- [ ] Run a container using `docker run`?
- [ ] List containers and images?
- [ ] Troubleshoot basic Docker issues?
- [ ] Understand the difference between images and containers?
- [ ] Configure Docker permissions on Linux?

**If you answered "No" to any question**, review:
- [README.md](README.md) for concepts
- [exercises.md](exercises.md) for practice
- [solutions.md](solutions.md) for detailed explanations
- [cheatsheet.md](cheatsheet.md) for quick reference

---

## Common Quiz Mistakes

### ❌ Mistake 1: Confusing Images and Containers
- **Wrong:** "An image is a running container"
- **Right:** "An image is a template; a container is a running instance of that template"

### ❌ Mistake 2: Installation Method
- **Wrong:** "Docker needs to be installed the same way on all operating systems"
- **Right:** "Windows/macOS use Docker Desktop; Linux uses package managers"

### ❌ Mistake 3: Docker Daemon Understanding
- **Wrong:** "The daemon is only used by Docker CLI"
- **Right:** "The daemon is the core service that manages all containers and images"

### ❌ Mistake 4: Permission Issues
- **Wrong:** "Use chmod to fix Docker permission errors"
- **Right:** "Add user to docker group using `usermod -aG docker`"

### ❌ Mistake 5: Version Command
- **Wrong:** "Use `docker --version` to check both CLI and daemon versions"
- **Right:** "Use `docker version` for both, or `docker --version` for just CLI"

---

## Next Steps After Quiz

**If you scored 7+:**
✅ Great work! You're ready for Module 01: Docker Basics

**If you scored below 7:**
📚 Review these resources:
1. Re-read [README.md](README.md) focusing on weak areas
2. Re-do exercises related to your mistakes
3. Review [solutions.md](solutions.md) for deeper understanding
4. Use [cheatsheet.md](cheatsheet.md) for quick reference
5. Retake this quiz after review

---

## Additional Resources

**For More Practice:**
- Docker Documentation: https://docs.docker.com/get-started/
- Interactive Docker Tutorial: https://www.docker.com/101-tutorial/
- Play with Docker (Free): https://www.docker.com/play-with-docker

**For Questions:**
- Docker Community Forums: https://community.docker.com/
- Stack Overflow: Tag questions with `docker`
- GitHub Issues: https://github.com/moby/moby/issues

---

**Good luck! 🚀 You've got this!**

