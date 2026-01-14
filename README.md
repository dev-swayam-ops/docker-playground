# Docker Playground - Complete Learning Curriculum

A comprehensive, hands-on Docker learning platform designed for beginners to intermediate learners. This repository contains 12 progressive modules covering everything from Docker basics to production-ready DevOps patterns.

## 📚 Curriculum Overview

| Module | Topic | Focus |
|--------|-------|-------|
| **00** | [Setup & Prerequisites](./00-setup-and-prerequisites) | Installation, verification, system requirements |
| **01** | [Docker Basics](./01-docker-basics) | Containers, images, CLI commands, lifecycle |
| **02** | [Images & Dockerfile](./02-images-and-dockerfile) | Building images, Dockerfile instructions, optimization |
| **03** | [Containers & Lifecycle](./03-containers-and-lifecycle) | Health checks, resource limits, restart policies |
| **04** | [Volumes & Storage](./04-volumes-and-storage) | Persistence, bind mounts, backup/restore |
| **05** | [Networking](./05-networking-in-docker) | Bridge/host/overlay modes, DNS, service discovery |
| **06** | [Docker Compose](./06-docker-compose) | Multi-container orchestration, YAML configuration |
| **07** | [Registries & Image Management](./07-registries-and-image-management) | Docker Hub, tagging, pushing, pulling, private registries |
| **08** | [Security Best Practices](./08-security-best-practices) | Non-root users, scanning, capabilities, secrets |
| **09** | [Troubleshooting & Debugging](./09-troubleshooting-and-debugging) | Logs, inspection, diagnostics, monitoring |
| **10** | [Performance & Optimization](./10-performance-and-optimization) | Resource limits, caching, image optimization |
| **11** | [Production Patterns](./11-production-patterns) | Rolling updates, monitoring, incidents, backups |
| **12** | [Real-World DevOps Labs](./12-real-world-devops-labs) | End-to-end scenarios, CI/CD, infrastructure patterns |

## 🎯 Learning Path

**Recommended progression:**
1. Start with Module 00 for environment setup
2. Progress through Modules 01-05 for core Docker concepts
3. Move to Modules 06-08 for orchestration and security
4. Cover Modules 09-10 for operational excellence
5. Complete Modules 11-12 for production readiness

*Can be consumed in any order after Module 00.*

## 📖 What's in Each Module?

Each module includes:
- **README.md** - Comprehensive guide with hands-on lab examples
- **exercises.md** - 10 progressive exercises (easy → medium) with solutions
- **solutions.md** - Detailed explanations and command outputs
- **cheatsheet.md** - Quick command reference with examples
- **quiz.md** - 10-question self-assessment (7 MCQ + 3 short answer)

**Total Content:** 60 files, 50+ hours of learning material

## 🚀 Quick Start

### Prerequisites
- Docker 20.10+ installed and running
- Basic command-line familiarity
- 4GB+ RAM recommended
- Internet connection for image downloads

### Getting Started

```bash
# Clone the repository
git clone <repo-url>
cd docker-playground

# Navigate to any module
cd 01-docker-basics

# Read the guide
cat README.md

# Work through exercises
cat exercises.md

# Verify your environment (Module 00 specific)
docker --version
docker run hello-world
```

## 💡 How to Use This Repository

### For Self-Paced Learning
1. Read the module README
2. Execute the lab examples
3. Work through all 10 exercises
4. Compare your solutions with provided answers
5. Use the cheatsheet as reference
6. Take the quiz and aim for 70% passing score

### For Study Groups
- Discuss lab scenarios in Module 12
- Collaborate on exercises
- Review production patterns (Module 11)
- Practice troubleshooting (Module 09) together

### For Hands-On Practice
- Modify labs to experiment with variations
- Combine concepts across modules
- Build a personal project using Module 12 patterns
- Document your learning journey

## 📊 Learning Outcomes

By completing this curriculum, you will:

✅ Understand Docker architecture and containerization concepts  
✅ Build, run, and manage Docker containers  
✅ Create optimized Dockerfiles and images  
✅ Implement persistent storage with volumes  
✅ Configure container networking and service discovery  
✅ Orchestrate multi-container applications with Docker Compose  
✅ Manage images in registries  
✅ Implement security best practices  
✅ Debug and troubleshoot container issues  
✅ Optimize performance and resource usage  
✅ Deploy containers in production environments  
✅ Implement DevOps patterns and CI/CD workflows  

## 🛠️ Technologies Covered

- Docker CLI and daemon
- Dockerfile instruction set
- Docker Compose and YAML
- Docker networking modes (bridge, host, overlay)
- Volume management (named, bind mount, tmpfs)
- Docker registries and image tagging
- Docker secrets and configuration
- Health checks and monitoring
- Security practices (scanning, capabilities, non-root)
- Production deployment patterns

## 📝 Module-Specific Tips

| Module | Time | Key Tip |
|--------|------|---------|
| 00 | 30 min | Ensure Docker daemon is running before proceeding |
| 01 | 2 hrs | Practice all CLI commands multiple times |
| 02 | 3 hrs | Build images locally; understand layer caching |
| 03-05 | 6 hrs | Focus on network connectivity between containers |
| 06 | 2 hrs | Start with simple 2-service compose files |
| 07 | 2 hrs | Create free Docker Hub account for image practice |
| 08 | 3 hrs | Understand security trade-offs (flexibility vs security) |
| 09 | 2 hrs | Reproduce failures to understand debugging |
| 10 | 2 hrs | Monitor resource usage with `docker stats` |
| 11 | 3 hrs | Plan before implementing production scenarios |
| 12 | 4 hrs | Build a real application using all learned concepts |

## ✨ Best Practices

- **Read before doing** - Understand concepts first, then apply
- **Hands-on labs** - Execute every command in the README labs
- **Experiment** - Modify examples to test your understanding
- **Review solutions** - Check your work against provided answers
- **Quiz yourself** - Take the module quiz to assess progress
- **Build projects** - Create applications combining multiple modules
- **Document** - Keep notes on new concepts and commands

## 🤝 Contributing

Found an issue or have suggestions? Please:
1. Review the existing content first
2. Test any changes locally
3. Ensure consistency with existing modules
4. Document your additions clearly

## 📞 Support & Resources

- **Docker Official Docs:** https://docs.docker.com
- **Docker Best Practices:** https://docs.docker.com/develop/dev-best-practices/
- **Docker Security:** https://docs.docker.com/engine/security/
- **Community:** Docker community forums and Slack

## 📈 Progress Tracking

Track your progress through the curriculum:
- [ ] Module 00: Setup & Prerequisites
- [ ] Module 01: Docker Basics
- [ ] Module 02: Images & Dockerfile
- [ ] Module 03: Containers & Lifecycle
- [ ] Module 04: Volumes & Storage
- [ ] Module 05: Networking
- [ ] Module 06: Docker Compose
- [ ] Module 07: Registries & Image Management
- [ ] Module 08: Security Best Practices
- [ ] Module 09: Troubleshooting & Debugging
- [ ] Module 10: Performance & Optimization
- [ ] Module 11: Production Patterns
- [ ] Module 12: Real-World DevOps Labs

## 📄 License

This educational material is provided as-is for learning purposes.

---

**Happy Learning! 🐳**

Start with [Module 00: Setup & Prerequisites](./00-setup-and-prerequisites/README.md)
