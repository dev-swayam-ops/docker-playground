# Module 04: Volumes & Storage - Exercises, Solutions, Cheatsheet, Quiz

## Exercises (10)
1. Create named volume and mount to container
2. Use bind mount from host directory  
3. Share volume between multiple containers
4. Backup volume data
5. Restore from backup
6. Use read-only mounts
7. Anonymous volumes
8. Volume driver inspection
9. Mount tmpfs (temporary) storage
10. Clean up unused volumes

## Solutions
- Named: `docker volume create my-data` → `docker run -v my-data:/data`
- Bind: `docker run -v /host/path:/container/path`
- Backup: `docker run --rm -v my-data:/data -v $(pwd):/backup alpine tar czf /backup/data.tar.gz /data`
- Share: Multiple containers `--mount source=same-volume,target=/path`

## Cheatsheet

| Command | Purpose |
|---------|---------|
| `docker volume create [name]` | Create named volume |
| `docker volume ls` | List all volumes |
| `docker volume inspect [name]` | Volume details |
| `docker volume rm [name]` | Remove volume |
| `docker volume prune` | Remove unused |
| `docker run -v [vol]:/path` | Mount volume |
| `docker run -v /host:/container` | Bind mount |
| `docker run -v /path:ro` | Read-only mount |
| `docker run --tmpfs /path` | Temporary storage |

## Quiz (7 MCQ + 3 Short)

**MCQ1:** Named volume benefit?
A) Faster B) Managed by Docker C) Smaller D) None
**Answer:** B

**MCQ2:** Bind mount requirement?
A) Volume must exist B) Host directory must exist C) Docker creates it D) Auto-created
**Answer:** B

**MCQ3:** Backup volume command creates?
A) Docker backup B) tar.gz file C) Snapshot D) Copy
**Answer:** B

**MCQ4:** Read-only mount flag?
A) :ro B) :read C) :readonly D) :nowrite
**Answer:** A

**MCQ5:** tmpfs mount is?
A) Persistent B) Temporary (RAM) C) Network D) Encrypted
**Answer:** B

**MCQ6:** Share volume between containers?
A) Create copy B) Same volume name C) Network D) Link containers
**Answer:** B

**MCQ7:** prune command removes?
A) All volumes B) Unused volumes C) Named only D) Host volumes
**Answer:** B

**Short1:** When to use named vs bind mount?
**Answer:** Named: permanent data, managed. Bind: dev, config files, host integration.

**Short2:** How to backup volume?
**Answer:** Run temporary container mounting volume and host directory, tar/copy data.

**Short3:** Volume security considerations?
**Answer:** Host permissions apply, ownership, encryption, backup strategy.
