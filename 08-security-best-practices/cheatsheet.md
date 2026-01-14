# Cheatsheet - Security

| Command | Purpose |
|---------|---------|
| `docker run --user appuser` | Non-root |
| `docker scan [image]` | Scan image |
| `docker run --cap-drop=ALL` | Drop caps |
| `docker run --cap-add=NET_BIND` | Add cap |
| `docker run --read-only` | Read-only |
| `docker secret create` | Create secret |
| `docker run --security-opt` | Security opts |
