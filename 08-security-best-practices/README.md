# Module 08: Security Best Practices

Secure Docker containers and images for production.

## What You'll Learn

- Run containers as non-root users
- Image scanning and vulnerabilities
- Secret management
- Security scanning tools
- Container isolation and capabilities
- Network security
- Secure image building practices

## Lab

```bash
# Run as non-root user
docker run -d --user appuser nginx

# Scan image for vulnerabilities
docker scan myimage:1.0

# Limit capabilities
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE nginx

# Use secrets
docker secret create db_password password.txt
docker service create --secret db_password myapp
```

## Best Practices

- ✅ Run as non-root
- ✅ Use minimal images (alpine)
- ✅ Scan images regularly
- ✅ Limit capabilities
- ✅ Use secrets for sensitive data
- ✅ Keep images updated
- ✅ Monitor containers

---

**Exercises:** [exercises.md](exercises.md)
This folder contains information about Docker security best practices.
