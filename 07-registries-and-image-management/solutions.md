# Solutions: Registries and Image Management

Comprehensive solutions for all 10 exercises with detailed commands and explanations.

---

## Exercise 1: Login to Docker Hub

### Solution

### Commands

```bash
# Login to Docker Hub
docker login

# Expected output:
# Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
# Username: your-username
# Password: ****
# WARNING! Your password will be stored unencrypted in /home/user/.docker/config.json
# Configure a credential helper to remove this warning. See
# https://docs.docker.com/engine/reference/commandline/login/#credentials-store
#
# Login Succeeded

# Verify login was successful
docker info

# Expected output shows:
# Registries:
#  Username: your-username
#  Registry URL: https://index.docker.io/v1/

# Check stored credentials
cat ~/.docker/config.json

# Expected output (sanitized):
# {
#   "auths": {
#     "https://index.docker.io/v1/": {
#       "auth": "base64encodedstring"
#     }
#   }
# }

# Login to specific registry (not Docker Hub)
docker login -u username registry.example.com:5000

# Expected output:
# Enter password for your registry

# Verify authentication
docker inspect my-image

# Try pulling from authenticated registry
docker pull registry.example.com:5000/myimage:latest

# Expected output:
# If credentials valid, image is pulled

# View authentication status
docker info | grep -A 5 "Registries"

# Expected output:
# Shows authenticated registries

# Logout from Docker Hub
docker logout

# Expected output:
# Removing auth entry for https://index.docker.io/v1/
# Logout Succeeded

# Verify logout
docker push username/myimage:1.0

# Expected output:
# Error response from daemon: push access denied
```

### Explanation

**Docker Login:**
- Required for pushing to personal repositories
- Optional for pulling public images
- Credentials stored in ~/.docker/config.json
- Can login to multiple registries

**Security:**
- Use Docker credential helpers (pass, osxkeychain, wincred)
- Never commit credentials to git
- Use access tokens instead of passwords

---

## Exercise 2: Tag image with username

### Solution

### Commands

```bash
# Create a test image first
docker run -d --name test-container nginx:latest
docker commit test-container my-nginx:1.0

# Expected output:
# sha256:abc123def456...

# View current image
docker image ls | grep my-nginx

# Expected output:
# REPOSITORY   TAG    IMAGE ID      SIZE
# my-nginx     1.0    abc123def456  150MB

# Tag image with username for Docker Hub
docker tag my-nginx:1.0 username/my-nginx:1.0

# Expected output:
# (no output if successful)

# Verify both tags exist
docker image ls | grep -E 'my-nginx|username'

# Expected output:
# my-nginx             1.0       abc123def456  150MB
# username/my-nginx    1.0       abc123def456  150MB

# Both tags point to same image (same IMAGE ID)

# Tag with multiple versions
docker tag my-nginx:1.0 username/my-nginx:latest
docker tag my-nginx:1.0 username/my-nginx:stable

# Expected output:
# (no output)

# View all tags
docker image ls | grep username/my-nginx

# Expected output:
# username/my-nginx    1.0       abc123def456  150MB
# username/my-nginx    latest    abc123def456  150MB
# username/my-nginx    stable    abc123def456  150MB

# All have same IMAGE ID (same physical image)

# Tag with full registry path
docker tag my-nginx:1.0 registry.example.com:5000/my-nginx:1.0

# Expected output:
# (no output)

# Verify registry tag
docker image ls | grep registry.example.com

# Expected output:
# registry.example.com:5000/my-nginx    1.0    abc123def456  150MB

# Tag with date-based version
docker tag my-nginx:1.0 username/my-nginx:2024-01-15

# Expected output:
# (no output)

# Remove a specific tag (not the image)
docker rmi username/my-nginx:stable

# Expected output:
# Untagged: username/my-nginx:stable

# Verify removal
docker image ls | grep stable

# Expected output:
# (no matching images)

# Original image still exists
docker image ls | grep my-nginx:1.0

# Expected output:
# my-nginx    1.0    abc123def456  150MB

# Remove original image (removes all tags)
docker rmi my-nginx:1.0

# Expected output:
# Error response from daemon: conflict: unable to remove repository reference
# (because other tags still reference it)

# Force remove
docker rmi -f my-nginx:1.0

# Expected output:
# Untagged, Deleted layers
```

### Explanation

**Image Tagging:**
- Format: `[registry]/[repository]:[tag]`
- Same image can have multiple tags
- Tags point to same IMAGE ID
- Removing tag doesn't remove image
- Useful for versioning and organization

**Best Practices:**
1. Use semantic versioning (1.0.0, 1.0.1, 1.1.0)
2. Use `latest` tag carefully
3. Tag for all environments (dev, stage, prod)
4. Document tag meanings

---

## Exercise 3: Push image to Docker Hub

### Solution

### Commands

```bash
# Ensure logged in
docker login

# Expected output:
# Login Succeeded

# Have tagged image ready
docker tag my-nginx:1.0 username/my-nginx:1.0

# Expected output:
# (no output)

# Push image to Docker Hub
docker push username/my-nginx:1.0

# Expected output:
# The push refers to repository [docker.io/username/my-nginx]
# a1b2c3d4e5f6: Pushing [===============>                         ]  20 MiB/50 MiB
# b2c3d4e5f6a7: Pushing [=======================================] 100 MiB
# c3d4e5f6a7b8: Pushed
# 1.0: digest: sha256:abc123def456... size: 2048

# Verify push was successful
docker image ls | grep username/my-nginx

# Expected output:
# username/my-nginx    1.0    abc123def456  150MB

# View push history
docker history username/my-nginx:1.0

# Expected output shows all layers

# Push with latest tag
docker push username/my-nginx:latest

# Expected output:
# The push refers to repository [docker.io/username/my-nginx]
# latest: digest: sha256:abc123def456... size: 2048

# Push to private registry
docker push registry.example.com:5000/my-nginx:1.0

# Expected output:
# (similar to Docker Hub)

# Push multiple tags
docker push username/my-nginx:1.0 username/my-nginx:latest

# Expected output:
# Pushes both tags

# Monitor push progress
docker push username/my-nginx:1.0 --quiet

# Expected output:
# (minimal output)

# Push with detailed logging
docker push username/my-nginx:1.0

# Expected output:
# Shows layer-by-layer push progress

# Verify image on Docker Hub
# Visit https://hub.docker.com/r/username/my-nginx
# Shows pushed tags and image details

# Pull image back (from another machine or account)
docker pull username/my-nginx:1.0

# Expected output:
# 1.0: Pulling from username/my-nginx
# a1b2c3d4e5f6: Pull complete
# b2c3d4e5f6a7: Pull complete
# Digest: sha256:abc123def456...
# Status: Downloaded newer image for username/my-nginx:1.0

# Verify pulled image is identical
docker image inspect username/my-nginx:1.0 | grep '"Id"'

# Expected output:
# Same SHA256 as original
```

### Explanation

**Pushing Images:**
- Requires login to registry
- Transfers all image layers
- First push is slower (uploads all data)
- Subsequent pushes only upload changed layers
- Can push to Docker Hub or private registry

**Layer Caching:**
- Docker caches layers locally
- Unchanged layers not re-uploaded
- Makes repeated pushes faster
- Layered structure enables efficiency

---

## Exercise 4: Pull public image

### Solution

### Commands

```bash
# Search for a public image first
docker search nginx

# Expected output:
# NAME                              DESCRIPTION                STARS   OFFICIAL
# nginx                             Official build of Nginx    17000   [OK]
# jwilder/nginx-proxy               Automated Nginx reverse …  2100
# bitnami/nginx                     Bitnami Nginx Docker …     500

# Pull official nginx image
docker pull nginx:latest

# Expected output:
# latest: Pulling from library/nginx
# a1b2c3d4e5f6: Pull complete
# b2c3d4e5f6a7: Pull complete
# Digest: sha256:abc123def456...
# Status: Downloaded newer image for nginx:latest

# Verify pull was successful
docker image ls | grep nginx

# Expected output:
# nginx    latest    abc123def456  150MB

# Pull specific version
docker pull nginx:1.21-alpine

# Expected output:
# Shows pull progress

# Verify both versions exist
docker image ls nginx

# Expected output:
# REPOSITORY   TAG            IMAGE ID      SIZE
# nginx        latest         abc123def456  150MB
# nginx        1.21-alpine    def456abc789  100MB

# Pull from specific registry
docker pull gcr.io/google-samples/hello-app:1.0

# Expected output:
# Shows pull from Google Container Registry

# Verify gcr image
docker image ls | grep google

# Expected output:
# gcr.io/google-samples/hello-app    1.0    ...

# Pull multiple images
docker pull ubuntu:22.04 alpine:latest postgres:15

# Expected output:
# Shows three images being pulled

# Verify all downloaded
docker image ls | grep -E 'ubuntu|alpine|postgres'

# Expected output:
# All three images listed

# Pull with digest (specific version)
docker pull nginx@sha256:abc123def456...

# Expected output:
# Shows pull by digest (guarantees exact image)

# Check image details after pull
docker inspect nginx:latest | grep -A 10 '"RepoDigests"'

# Expected output:
# Shows digest and repo info

# Run pulled image
docker run -d --name web -p 8080:80 nginx:latest

# Expected output:
# Container ID

# Verify it works
curl http://localhost:8080

# Expected output:
# (nginx default page)

# Check pull history
docker history nginx:latest

# Expected output:
# Shows all layers downloaded
```

### Explanation

**Pulling Images:**
- Downloads image from registry to local machine
- Stores in Docker's image cache
- Can specify version tag
- Can use digest for exact version
- No login required for public images

**Image Registries:**
- Docker Hub (default): hub.docker.com
- Google Container Registry (GCR): gcr.io
- Azure Container Registry (ACR): *.azurecr.io
- AWS ECR: *.dkr.ecr.*.amazonaws.com

---

## Exercise 5: Search Docker Hub

### Solution

### Commands

```bash
# Basic search
docker search nginx

# Expected output:
# NAME                              DESCRIPTION                STARS   OFFICIAL
# nginx                             Official build of Nginx    17000   [OK]
# jwilder/nginx-proxy               Automated Nginx reverse…   2100
# bitnami/nginx                     Bitnami Nginx Docker…      500
# ...

# Search with filters
docker search --filter is-official=true nginx

# Expected output:
# Only official nginx images

# Search with star limit
docker search --filter stars=1000 nginx

# Expected output:
# Only images with 1000+ stars

# Search for no-automated builds only
docker search --filter is-automated=false nginx

# Expected output:
# Images not auto-built

# Limit results
docker search nginx --limit 5

# Expected output:
# Only 5 results

# Search for alternative
docker search postgres

# Expected output:
# postgres          Official PostgreSQL...  11000  [OK]
# bitnami/postgres  Bitnami PostgreSQL…     500

# Search for database images
docker search --filter "description=database" database

# Expected output:
# Various database images

# Search for minimal images
docker search alpine

# Expected output:
# alpine            Lightweight Linux distro  8000  [OK]

# Get detailed info without search (using pull + inspect)
docker pull alpine:latest

# Expected output:
# Image pulled

docker inspect alpine:latest

# Expected output:
# Detailed image metadata

# Check image history
docker history alpine:latest

# Expected output:
# Shows base image and layers

# Compare image sizes
docker search --limit 5 nginx | awk '{print $1, "stars:", $3}'

# Expected output:
# Lists images and star counts

# Note: `docker search` is limited
# For detailed search: visit hub.docker.com in browser
```

### Explanation

**Docker Search:**
- Limited to image name and description
- Can filter by official, automated status
- Can filter by minimum stars
- Better to browse hub.docker.com for details

**Search Results:**
- NAME: Repository and image name
- DESCRIPTION: Short description
- STARS: Community ratings
- OFFICIAL: Official maintained
- AUTOMATED: Auto-built from GitHub

---

## Exercise 6: Create private registry

### Solution

### Commands

```bash
# Create a private registry using Docker
docker run -d \
  --name private-registry \
  -p 5000:5000 \
  -v registry-data:/var/lib/registry \
  registry:2

# Expected output:
# Container ID

# Verify registry is running
docker ps | grep private-registry

# Expected output:
# registry-data   registry:2    Up 5 seconds    0.0.0.0:5000->5000/tcp

# Test registry health
curl http://localhost:5000/v2/

# Expected output:
# {} (empty, indicating success)

# Create image to push to private registry
docker run -d --name test nginx:latest
docker commit test my-app:1.0

# Expected output:
# Image created

# Tag image for private registry
docker tag my-app:1.0 localhost:5000/my-app:1.0

# Expected output:
# (no output)

# Push to private registry
docker push localhost:5000/my-app:1.0

# Expected output:
# The push refers to repository [localhost:5000/my-app]
# (push progress)

# List images in private registry
curl http://localhost:5000/v2/_catalog

# Expected output:
# {"repositories":["my-app"]}

# List tags for image in registry
curl http://localhost:5000/v2/my-app/tags/list

# Expected output:
# {"name":"my-app","tags":["1.0"]}

# Pull from private registry
docker pull localhost:5000/my-app:1.0

# Expected output:
# (pull progress)

# Create secure private registry with TLS
docker run -d \
  --name secure-registry \
  -p 5000:5000 \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \
  -v registry-data:/var/lib/registry \
  -v ./certs:/certs \
  registry:2

# (Requires valid SSL certificates)

# Create registry with authentication
docker run -d \
  --name auth-registry \
  -p 5000:5000 \
  -e REGISTRY_AUTH=htpasswd \
  -e REGISTRY_AUTH_HTPASSWD_REALM="Registry Realm" \
  -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
  -v registry-data:/var/lib/registry \
  -v ./auth:/auth \
  registry:2

# Create htpasswd file for authentication
htpasswd -Bc ./auth/htpasswd username

# Expected output:
# Prompts for password

# Login to authenticated registry
docker login -u username localhost:5000

# Expected output:
# Prompts for password

# Push to authenticated registry
docker push localhost:5000/my-app:1.0

# Expected output:
# (push progress)

# View registry configuration
curl http://localhost:5000/v2/

# Expected output:
# {} if authenticated

# Check registry logs
docker logs private-registry

# Expected output:
# Shows push/pull activity

# Cleanup registry
docker stop private-registry
docker rm private-registry
docker volume rm registry-data
```

### Explanation

**Private Registry:**
- Self-hosted Docker repository
- Uses official registry:2 image
- Requires volume for persistence
- Can add TLS and authentication
- Useful for enterprise/secure deployments

**Registry API:**
- `/v2/_catalog`: List all repositories
- `/v2/[repo]/tags/list`: List tags for repo
- `/v2/[repo]/manifests/[tag]`: Get manifest

---

## Exercise 7: Image versioning strategy

### Solution

### Commands

```bash
# Semantic versioning approach
docker build -t my-app:1.0.0 .
docker build -t my-app:1.0 .
docker build -t my-app:1 .
docker build -t my-app:latest .

# Expected output:
# Image tagged with multiple versions

# Verify all tags
docker image ls my-app

# Expected output:
# my-app    1.0.0    abc123...  150MB
# my-app    1.0      abc123...  150MB
# my-app    1        abc123...  150MB
# my-app    latest   abc123...  150MB

# Date-based versioning
docker tag my-app:1.0 my-app:2024-01-15
docker tag my-app:1.0 my-app:2024-01

# Expected output:
# (no output)

# Environment-based versioning
docker tag my-app:1.0 my-app:stable
docker tag my-app:1.0 my-app:beta
docker tag my-app:1.0 my-app:dev

# Expected output:
# (no output)

# View versioning strategy
docker image ls my-app

# Expected output shows all version tags

# Git-based versioning (using commit hash)
COMMIT=$(git rev-parse --short HEAD)
docker tag my-app:1.0 my-app:$COMMIT

# Expected output:
# Image tagged with commit hash

# Build with version from file
VERSION=$(cat version.txt)
docker build -t my-app:$VERSION .

# Expected output:
# Image built with version from file

# Automated version tagging script
cat > tag-versions.sh << 'EOF'
#!/bin/bash
IMAGE=$1
VERSION=$2

# Tag with full version
docker tag $IMAGE:latest $IMAGE:$VERSION

# Extract major.minor
MAJOR_MINOR=$(echo $VERSION | cut -d. -f1-2)
docker tag $IMAGE:latest $IMAGE:$MAJOR_MINOR

# Extract major
MAJOR=$(echo $VERSION | cut -d. -f1)
docker tag $IMAGE:latest $IMAGE:$MAJOR

# Keep latest
docker tag $IMAGE:latest $IMAGE:stable
EOF

chmod +x tag-versions.sh

# Use the script
./tag-versions.sh my-app 1.2.3

# Expected output:
# Image tagged as: 1.2.3, 1.2, 1, stable, latest

# View all versions
docker image ls my-app --format "table {{.Tag}}\t{{.Size}}"

# Expected output:
# TAG       SIZE
# 1.2.3     150MB
# 1.2       150MB
# 1         150MB
# stable    150MB
# latest    150MB

# Push multiple versions
for tag in latest stable 1.2.3 1.2 1; do
  docker push username/my-app:$tag
done

# Expected output:
# Pushes all tags

# Document versioning strategy
cat > VERSIONING.md << 'EOF'
# Image Versioning Strategy

## Tagging Convention

- `latest`: Most recent build
- `stable`: Latest stable release
- `1.2.3`: Semantic version (major.minor.patch)
- `1.2`: Minor version (for latest patch)
- `1`: Major version (for latest minor/patch)
- `dev`, `beta`: Pre-release versions
- `YYYY-MM-DD`: Date-based snapshots

## Usage

- Use specific versions (1.2.3) for production
- Use major/minor versions for stability with updates
- Use latest only for development
EOF

# Clean up old versions
docker rmi my-app:2024-01-01

# Expected output:
# Untagged and removed
```

### Explanation

**Versioning Strategies:**

| Strategy | Format | Example | Use Case |
|----------|--------|---------|----------|
| Semantic | major.minor.patch | 1.2.3 | Production |
| Major.minor | major.minor | 1.2 | Allow patch updates |
| Major | major | 1 | Allow minor updates |
| Stability | stable, latest | stable | Default behaviors |
| Date | YYYY-MM-DD | 2024-01-15 | Snapshots |
| Environment | dev, beta, prod | beta | Environment tracking |

**Best Practices:**
1. Never use `latest` in production
2. Use semantic versioning for releases
3. Have a `stable` tag for current production
4. Keep at least 2 previous versions
5. Document versioning policy

---

## Exercise 8: Remove old image tags

### Solution

### Commands

```bash
# View all images and tags
docker image ls my-app

# Expected output:
# my-app    1.0      abc123...  150MB
# my-app    1.1      abc123...  150MB
# my-app    1.2      def456...  150MB
# my-app    2.0      ghi789...  200MB
# my-app    latest   ghi789...  200MB

# Remove specific tag (not the image)
docker rmi my-app:1.0

# Expected output:
# Untagged: my-app:1.0

# Verify tag removed
docker image ls my-app | grep "1.0"

# Expected output:
# (no matching results)

# Remove multiple old tags
docker rmi my-app:1.1 my-app:2.0-rc1 my-app:beta

# Expected output:
# Untagged: my-app:1.1
# Untagged: my-app:2.0-rc1
# Untagged: my-app:beta

# Remove dangling tags (no container using them)
docker image prune

# Expected output:
# WARNING! This will remove all dangling images.
# Are you sure you want to continue? [y/N] y
# Deleted Images:
# untagged my-app@sha256:...
# Total reclaimed space: 50MB

# Remove images by pattern
docker image rm $(docker image ls 'my-app:1.*' -q)

# Expected output:
# Removes all 1.x versions

# Remove images older than date
# (Manual approach - find and remove)
docker image ls --format "{{.Repository}}:{{.Tag}}\t{{.CreatedAt}}" | grep my-app

# Expected output:
# Shows creation dates

# Remove by force if in use
docker rmi -f my-app:1.0

# Expected output:
# Forces removal even if in use (not recommended)

# Cleanup script for old tags
cat > cleanup-old-tags.sh << 'EOF'
#!/bin/bash
IMAGE=$1
KEEP=$2  # Number of versions to keep

# Get tags sorted by creation date
TAGS=$(docker image ls --format "{{.Tag}}" $IMAGE | head -n -$KEEP)

# Remove old tags
for TAG in $TAGS; do
  echo "Removing $IMAGE:$TAG"
  docker rmi $IMAGE:$TAG
done
EOF

chmod +x cleanup-old-tags.sh

# Use cleanup script (keep 3 latest versions)
./cleanup-old-tags.sh my-app 3

# Expected output:
# Removes all but 3 latest versions

# Remove all versions except latest and stable
docker image ls my-app --format "{{.Tag}}" | grep -v -E 'latest|stable' | while read TAG; do
  docker rmi my-app:$TAG
done

# Expected output:
# Removes all tags except latest and stable

# Cleanup unused images (not used by any container)
docker image prune -a

# Expected output:
# WARNING! This will remove all images without at least one container associated to them.
# Are you sure you want to continue? [y/N] y
# Deleted Images:
# (list of removed images)
# Total reclaimed space: XXX MB

# View remaining images
docker image ls my-app

# Expected output:
# Only kept versions remain
```

### Explanation

**Removing Image Tags:**
- `docker rmi image:tag`: Remove tag (not image if other tags exist)
- `docker rmi -f image:tag`: Force removal
- `docker image prune`: Remove dangling images
- `docker image prune -a`: Remove unused images

**Best Practices:**
1. Keep stable and latest always
2. Keep at least 2-3 old versions
3. Regularly clean up old tags
4. Archive important versions separately
5. Use automated cleanup scripts

---

## Exercise 9: Inspect image metadata

### Solution

### Commands

```bash
# Basic image inspection
docker inspect my-app:1.0

# Expected output:
# [
#   {
#     "Id": "sha256:abc123def456...",
#     "RepoTags": ["my-app:1.0"],
#     "RepoDigests": ["my-app@sha256:..."],
#     "Parent": "sha256:parent...",
#     "Comment": "",
#     "Created": "2024-01-15T12:00:00Z",
#     "Container": "abc123...",
#     ...
#   }
# ]

# Get specific fields
docker inspect --format='{{.Id}}' my-app:1.0

# Expected output:
# sha256:abc123def456...

# Get image creation date
docker inspect --format='{{.Created}}' my-app:1.0

# Expected output:
# 2024-01-15T12:00:00Z

# Get image size
docker inspect --format='{{.Size}}' my-app:1.0

# Expected output:
# 157286400 (bytes)

# Get image virtual size
docker inspect --format='{{.VirtualSize}}' my-app:1.0

# Expected output:
# 157286400

# Get image config
docker inspect --format='{{json .Config}}' my-app:1.0

# Expected output:
# {"Hostname":"","Domainname":"","User":"","AttachStdin":false,...}

# Pretty print config
docker inspect --format='{{json .Config}}' my-app:1.0 | jq

# Expected output:
# Pretty-printed JSON config

# Get working directory
docker inspect --format='{{.Config.WorkingDir}}' my-app:1.0

# Expected output:
# /app (or whatever WORKDIR is set to)

# Get exposed ports
docker inspect --format='{{.Config.ExposedPorts}}' my-app:1.0

# Expected output:
# map[5000/tcp:{} 8080/tcp:{}]

# Get environment variables
docker inspect --format='{{.Config.Env}}' my-app:1.0

# Expected output:
# [PATH=... LC_ALL=C.UTF-8 PYTHON_VERSION=...]

# Pretty print environment
docker inspect --format='{{json .Config.Env}}' my-app:1.0 | jq

# Get volumes
docker inspect --format='{{.Config.Volumes}}' my-app:1.0

# Expected output:
# map[/data:{}]

# Get labels
docker inspect --format='{{json .Config.Labels}}' my-app:1.0 | jq

# Expected output:
# {
#   "version": "1.0",
#   "maintainer": "user@example.com"
# }

# Get entrypoint
docker inspect --format='{{.Config.Entrypoint}}' my-app:1.0

# Expected output:
# [/bin/sh -c python app.py]

# Get CMD
docker inspect --format='{{.Config.Cmd}}' my-app:1.0

# Expected output:
# [python, app.py]

# View image history
docker history my-app:1.0

# Expected output:
# IMAGE         CREATED          CREATED BY                                SIZE      COMMENT
# abc123def456  15 minutes ago   /bin/sh -c pip install -r requirements.txt  25MB
# def456abc789  30 minutes ago   /bin/sh -c apt-get update && apt-get…      50MB
# ghi789def456  1 hour ago       /bin/sh -c mkdir -p /app                    0B

# View image layers
docker image history my-app:1.0 --no-trunc --quiet

# Expected output:
# sha256:abc123def456...
# sha256:def456abc789...
# sha256:ghi789def456...

# Compare two images
docker inspect --format='{{.RepoDigests}}' my-app:1.0
docker inspect --format='{{.RepoDigests}}' my-app:1.1

# Expected output:
# Different digests if different images

# Get all metadata in one command
docker inspect my-app:1.0 | jq '.[0] | keys'

# Expected output:
# Lists all available metadata fields
```

### Explanation

**Image Metadata:**
- `Id`: Unique image ID (SHA256 hash)
- `RepoTags`: Tags applied to image
- `Created`: When image was built
- `Size`: Uncompressed image size
- `Config`: Container configuration
- `Env`: Environment variables
- `Volumes`: Mounted volumes
- `Labels`: Custom metadata

**Common Uses:**
1. Verify image source and authenticity
2. Check image configuration
3. Audit image contents
4. Compare image versions
5. Extract metadata for automation

---

## Exercise 10: Cleanup unused images

### Solution

### Commands

```bash
# View all images
docker image ls

# Expected output:
# REPOSITORY            TAG       IMAGE ID      SIZE
# my-app                1.0       abc123def456  150MB
# my-app                1.1       def456abc789  150MB
# nginx                 latest    ghi789def456  150MB
# postgres              15        jkl012mno345  300MB
# unused-image          old       pqr345stu678  200MB

# Identify dangling images (unreferenced layers)
docker image ls -f dangling=true

# Expected output:
# Shows untagged images

# Remove dangling images
docker image prune

# Expected output:
# WARNING! This will remove all dangling images.
# Are you sure? [y/N] y
# Deleted Images:
# untagged my-app@sha256:...
# Total reclaimed space: 50MB

# Remove dangling images without confirmation
docker image prune -f

# Expected output:
# Removes immediately without prompt

# Remove unused images (not used by any container)
docker image prune -a

# Expected output:
# WARNING! This will remove all images without at least one container associated.
# Are you sure? [y/N] y
# Deleted Images:
# untagged my-app:old
# untagged unused-image:old
# ...
# Total reclaimed space: 450MB

# Remove images and containers
docker system prune

# Expected output:
# WARNING! This will remove...
# Total reclaimed space: XXX MB

# Remove everything (images, containers, volumes, networks)
docker system prune -a

# Expected output:
# WARNING! This will remove all unused images, containers, networks and optionally volumes.
# Reclaimed space: XXX MB

# Remove specific image
docker rmi my-app:1.0

# Expected output:
# Untagged: my-app:1.0

# Remove multiple images
docker rmi my-app:1.0 postgres:15 unused-image:old

# Expected output:
# Removes all three images

# Remove images by pattern
docker image rm $(docker image ls 'nginx*' -q)

# Expected output:
# Removes all nginx images

# Remove images before specific date (manual)
# First, list with creation dates
docker image ls --format "table {{.Repository}}:{{.Tag}}\t{{.CreatedAt}}"

# Expected output:
# Shows all images with creation dates

# Then manually remove old ones
docker rmi image:tag

# Cleanup script for automatic maintenance
cat > docker-cleanup.sh << 'EOF'
#!/bin/bash

echo "Removing dangling images..."
docker image prune -f

echo "Removing dangling containers..."
docker container prune -f

echo "Removing dangling volumes..."
docker volume prune -f

echo "Removing dangling networks..."
docker network prune -f

# Remove images unused for 72+ hours
docker image prune -a --filter "until=72h"

echo "Cleanup complete!"
EOF

chmod +x docker-cleanup.sh

# Run cleanup script
./docker-cleanup.sh

# Expected output:
# Shows deleted items and reclaimed space

# Schedule cleanup with cron (Linux)
# Add to crontab: 0 2 * * * /path/to/docker-cleanup.sh

# View disk usage
docker system df

# Expected output:
# TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
# Images          10        3         2.5GB     2.0GB (80%)
# Containers      15        2         500MB     450MB (90%)
# Volumes         5         2         1.0GB     500MB (50%)
# Local Volumes   0         0         0B        0B

# Remove specific size of images
# (No direct size-based removal, but can script it)

# Most aggressive cleanup (careful!)
docker rmi $(docker image ls -q)

# Expected output:
# Removes ALL images (including running ones if forced)

# Backup before cleanup
tar -czf docker-images-backup.tar.gz /var/lib/docker/

# Expected output:
# Backups all Docker images

# Verify cleanup was successful
docker image ls

# Expected output:
# Only remaining images shown
```

### Explanation

**Cleanup Commands:**

| Command | What It Removes |
|---------|-----------------|
| `docker image prune` | Dangling images only |
| `docker image prune -a` | Unused images |
| `docker container prune` | Stopped containers |
| `docker system prune` | Containers, images, networks |
| `docker system prune -a` | Everything except running |
| `docker system prune -a --volumes` | Including volumes |

**Best Practices:**
1. Regular maintenance (daily/weekly)
2. Keep important images backed up
3. Use automated cleanup scripts
4. Monitor disk usage
5. Have cleanup policy documented
