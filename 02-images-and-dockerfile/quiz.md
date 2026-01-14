# Quiz: Images and Dockerfile

Test your Dockerfile and image building knowledge!

## Instructions
- **MCQ (1-7):** Choose best answer
- **Short Answer (8-10):** Brief responses
- **Passing Score:** 70% (7+/10)

---

## Multiple Choice Questions

### Question 1: Dockerfile Base Image
**What does the FROM instruction do?**

A) Runs a command in the container  
B) Specifies the base image for building  
C) Copies files into the container  
D) Exposes a port  

**Answer:** B) Specifies the base image for building  
**Explanation:** FROM is the first instruction, always required, sets the starting point.

---

### Question 2: Layer Caching
**When does Docker reuse a cached layer?**

A) Always reuses layers  
B) When instruction input hasn't changed  
C) When the image tag is the same  
D) Never reuses layers  

**Answer:** B) When instruction input hasn't changed  
**Explanation:** Docker compares instruction and files. If identical, uses cached layer.

---

### Question 3: COPY vs ADD
**When should you use ADD instead of COPY?**

A) Always use ADD  
B) When adding tar archives that need extraction  
C) When copying application code  
D) They're exactly the same  

**Answer:** B) When adding tar archives that need extraction  
**Explanation:** ADD can extract tar files; COPY is simpler and preferred for files.

---

### Question 4: .dockerignore
**What does .dockerignore do?**

A) Ignores files during container execution  
B) Excludes files from build context  
C) Deletes files from the image  
D) Prevents file copying  

**Answer:** B) Excludes files from build context  
**Explanation:** Reduces build context size, faster builds, similar to .gitignore.

---

### Question 5: Multi-stage Build
**What is the main benefit of multi-stage builds?**

A) Faster compilation  
B) Better documentation  
C) Reduced final image size  
D) Multiple containers  

**Answer:** C) Reduced final image size  
**Explanation:** Build in one stage, copy only artifacts to runtime stage.

---

### Question 6: CMD vs ENTRYPOINT
**What happens when you run a container with CMD and override arguments?**

A) CMD and arguments both execute  
B) CMD is replaced entirely  
C) Arguments are ignored  
D) Error occurs  

**Answer:** B) CMD is replaced entirely  
**Explanation:** CMD can be completely overridden; ENTRYPOINT is fixed.

---

### Question 7: Image Layers
**How many layers does each instruction create?**

A) Every instruction creates a layer  
B) Only RUN creates layers  
C) Each instruction that modifies filesystem creates a layer  
D) Layers are created randomly  

**Answer:** C) Each instruction that modifies filesystem creates a layer  
**Explanation:** FROM, RUN, COPY create layers; ENV, LABEL do not (minor metadata).

---

## Short Answer Questions

### Question 8: Optimize Dockerfile
**Write a Dockerfile snippet that properly orders instructions for optimal caching. Explain why.**

**Model Answer:**
```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
CMD ["python", "app.py"]
```

**Explanation:**
- Requirements (dependencies) copied first - changes less frequently
- Application code copied next - changes frequently
- If only code changes, pip layer is cached and reused
- Results in faster rebuilds (seconds vs minutes)

---

### Question 9: Image Size Reduction
**List three techniques to reduce Docker image size and explain one in detail.**

**Model Answer:**

Three techniques:
1. **Use alpine/slim base images** - Alpine: 5MB vs Ubuntu: 70MB
2. **Multi-stage builds** - Compiler in builder stage, not in runtime
3. **Remove package caches** - `RUN apt clean && rm -rf /var/lib/apt/lists/*`
4. **.dockerignore** - Exclude node_modules, .git, test files

**Detailed explanation of multi-stage:**
```
Builder stage: golang:1.20 (800MB) - compiles application
Runtime stage: alpine:latest (5MB) - runs only binary
COPY --from=builder /app/binary .
Final image: 20MB instead of 800MB (40x smaller)
```

---

### Question 10: Dockerfile Security
**What security improvements should be made to this Dockerfile?**

```dockerfile
FROM ubuntu:latest
RUN apt-get update && apt-get install -y python3
COPY . /app
WORKDIR /app
CMD ["python3", "app.py"]
```

**Model Answer:**

Improvements:
1. **Specify base image version:** `FROM ubuntu:22.04` (security updates, reproducibility)
2. **Create non-root user:** 
   ```dockerfile
   RUN useradd -m appuser
   USER appuser
   ```
3. **Use minimal image:** `FROM python:3.11-slim` instead of ubuntu
4. **Add .dockerignore:** Exclude .git, secrets, unnecessary files
5. **Run as non-root:** Prevents privilege escalation
6. **Remove apt cache:** `&& rm -rf /var/lib/apt/lists/*`

**Enhanced Dockerfile:**
```dockerfile
FROM python:3.11-slim
RUN apt-get update && apt-get install -y curl && apt-get clean && rm -rf /var/lib/apt/lists/*
RUN useradd -m appuser
WORKDIR /app
COPY --chown=appuser:appuser . .
USER appuser
CMD ["python3", "app.py"]
```

---

## Answer Key Summary

| Q | Answer | Difficulty |
|---|--------|------------|
| 1 | B | Easy |
| 2 | B | Easy |
| 3 | B | Medium |
| 4 | B | Easy |
| 5 | C | Medium |
| 6 | B | Medium |
| 7 | C | Medium |
| 8 | Essay | Medium |
| 9 | Essay | Hard |
| 10 | Essay | Hard |

---

## Scoring

- **MCQ:** 1 point each = 7 points
- **Short Answer:** 1 point each = 3 points
- **Total:** 10 points
- **Passing:** 7+/10 (70%)

---

## Self-Assessment

Can you:
- [ ] Write a basic Dockerfile?
- [ ] Explain what layers are?
- [ ] Optimize Dockerfile for caching?
- [ ] Use multi-stage builds?
- [ ] Reduce image size?
- [ ] Apply security best practices?
- [ ] Understand COPY vs ADD?
- [ ] Explain CMD vs ENTRYPOINT?

**If not:** Review [README.md](README.md), [exercises.md](exercises.md), [solutions.md](solutions.md)

---

**Next:** Proceed to Module 03: Containers and Lifecycle!

