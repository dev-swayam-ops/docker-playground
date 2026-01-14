# Cheatsheet - Networking

| Command | Purpose |
|---------|---------|
| `docker network create [name]` | Create network |
| `docker network ls` | List networks |
| `docker network inspect [name]` | Inspect |
| `docker run --network [net]` | Run on net |
| `docker network connect [net] [container]` | Connect |
| `docker network disconnect [net] [container]` | Disconnect |
| `docker run -p [host]:[container]` | Port map |
| `docker run --expose [port]` | Expose port |
| `docker run --network=host` | Host mode |
| `docker run --network=none` | No network |
