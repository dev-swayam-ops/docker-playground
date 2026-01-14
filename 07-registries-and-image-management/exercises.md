# Module 07: Registries - Exercises, Cheatsheet, Quiz

## Exercises (10)
1. Login to Docker Hub
2. Tag image with username
3. Push image to Docker Hub
4. Pull public image
5. Search Docker Hub
6. Create private registry
7. Image versioning strategy
8. Remove old image tags
9. Inspect image metadata
10. Cleanup unused images

## Cheatsheet

| Command | Purpose |
|---------|---------|
| `docker login` | Login Hub |
| `docker tag [src] [dst]` | Tag |
| `docker push [image]` | Push |
| `docker pull [image]` | Pull |
| `docker search [term]` | Search |
| `docker logout` | Logout |
| `docker image prune` | Cleanup |
| `docker image ls` | List images |

## Quiz (7 MCQ + 3 Short)

Q1: Docker Hub auth file?
A) .docker/config.json B) .dockerrc C) docker.conf D) config.yaml
**Answer:** A

Q2: Image tag format?
A) image/tag B) registry/name:tag C) tag:image D) image-tag
**Answer:** B

Q3: Push requires?
A) Tag with username B) Registry login C) Both D) Neither
**Answer:** C

Q4: Private registry benefit?
A) Faster B) Control C) Security D) All
**Answer:** D

Q5: Image versioning best practice?
A) latest only B) Semantic versioning C) Random D) None
**Answer:** B

Q6: Prune command removes?
A) All images B) Unused only C) Tagged only D) Running
**Answer:** B

Q7: Pull command default registry?
A) Private B) Docker Hub C) Custom D) Local
**Answer:** B

Short1: When use private registry?
**Answer:** Proprietary code, internal apps, faster access (LAN), security requirements.

Short2: Image tag strategy?
**Answer:** Major.Minor.Patch (1.2.3), latest for current, stable for production releases.

Short3: Security with registries?
**Answer:** Scan images, sign images, restrict push access, audit logs.
