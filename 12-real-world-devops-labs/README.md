# Module 12: Real-World DevOps Labs

Practical end-to-end scenarios combining Docker with DevOps workflows.

## What You'll Learn

- Building multi-container applications
- CI/CD pipeline integration
- Infrastructure as code practices
- Monitoring and logging stacks
- Production deployment scenarios
- Troubleshooting complex issues
- Best practices for DevOps teams

## Lab Scenario: Complete Application Stack

```bash
# docker-compose.yml for production-like setup
version: '3.8'
services:
  frontend:
    image: nginx:latest
    ports:
      - "80:80"
    depends_on:
      - api
  
  api:
    build: .
    environment:
      DATABASE_URL: postgresql://user:pass@db:5432/app
    depends_on:
      - db
  
  db:
    image: postgres:14
    environment:
      POSTGRES_PASSWORD: secure_password
    volumes:
      - db_data:/var/lib/postgresql/data

volumes:
  db_data:
```

## Key Scenarios

- Multi-stage development (dev → staging → production)
- Blue-green deployments for zero downtime
- Monitoring with Prometheus and Grafana
- ELK stack for centralized logging
- Secrets management across environments
- Network troubleshooting in complex setups
- Performance optimization in production

---

**Exercises:** [exercises.md](exercises.md)
