# Module 08: Security - Exercises, Cheatsheet, Quiz

## Exercises (10)
1. Create non-root user in Dockerfile
2. Scan image with docker scan
3. Drop unsafe capabilities
4. Mount filesystem read-only
5. Use secrets in docker swarm
6. Sign images with notary
7. Implement image scanning in CI/CD
8. Audit container access
9. Network policies and firewall
10. Security updates and patching

## Cheatsheet

| Command | Purpose |
|---------|---------|
| `docker run --user appuser` | Non-root |
| `docker scan [image]` | Scan vulnerabilities |
| `docker run --cap-drop=ALL` | Drop all capabilities |
| `docker run --cap-add=NET_BIND` | Add capability |
| `docker run --read-only` | Read-only FS |
| `docker secret create [name]` | Create secret |
| `docker run --security-opt` | Security options |

## Quiz (7 MCQ + 3 Short)

Q1: Running as root in container is?
A) Required B) Security risk C) Faster D) Default
**Answer:** B

Q2: docker scan checks for?
A) Syntax B) Vulnerabilities C) Size D) Age
**Answer:** B

Q3: --cap-drop=ALL means?
A) No capabilities B) Limited C) Same D) Unsafe
**Answer:** A

Q4: Non-root user creation?
A) RUN useradd B) USER directive C) Both D) Neither
**Answer:** C

Q5: Secrets best for?
A) Configs B) Code C) Passwords D) Logs
**Answer:** C

Q6: Image scanning when?
A) Build time B) Pull time C) Both D) Never
**Answer:** C

Q7: Minimize attack surface via?
A) Alpine B) Scan C) User D) All
**Answer:** D

Short1: Why non-root important?
**Answer:** Limits damage from compromise, prevents privilege escalation, industry standard.

Short2: Scanning vs signing images?
**Answer:** Scanning: check for vulnerabilities. Signing: verify authenticity and integrity.

Short3: Secret management strategy?
**Answer:** External vault, never in code/image, rotate regularly, audit access.
