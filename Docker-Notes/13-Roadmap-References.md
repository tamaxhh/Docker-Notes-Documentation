# 13. 30-DAY DOCKER MASTERY ROADMAP & REFERENCES

---

## 30-Day Docker Mastery Roadmap

### Week 1: Foundation (Days 1–7)

| Day | Topic | Hands-On Practice |
|---|---|---|
| **1** | What is Docker, Install Docker Desktop | Run `docker run hello-world`, explore Docker Desktop |
| **2** | Images & Containers basics | Pull 5 images, run/stop/remove containers |
| **3** | Docker CLI mastery | Practice `ps`, `logs`, `exec`, `inspect`, `cp` |
| **4** | Write your first Dockerfile | Dockerize a simple "Hello World" app (any language) |
| **5** | Dockerfile instructions deep dive | Practice `FROM`, `RUN`, `CMD`, `COPY`, `ENTRYPOINT`, `WORKDIR` |
| **6** | Volumes & Bind Mounts | Run Postgres with a named volume, mount source code for live reload |
| **7** | Review & Practice | Re-do Days 1–6 from memory. Fix mistakes. |

### Week 2: Networking & Compose (Days 8–14)

| Day | Topic | Hands-On Practice |
|---|---|---|
| **8** | Docker Networking (bridge, host) | Create custom network, run 2 containers that communicate by name |
| **9** | Ports & Environment Variables | Map ports, pass env vars, use `--env-file` |
| **10** | Docker Compose intro | Write `compose.yaml` for a 2-service app (web + db) |
| **11** | Compose deep dive | Add healthchecks, `depends_on`, volumes, multiple networks |
| **12** | Full-stack Compose project | Build a 3-service app (frontend + API + database) |
| **13** | Layers, Cache & `.dockerignore` | Optimize a Dockerfile for cache. Measure image sizes. |
| **14** | Review & Practice | Build a complete project from scratch with Compose |

### Week 3: Production & Security (Days 15–21)

| Day | Topic | Hands-On Practice |
|---|---|---|
| **15** | Multi-stage builds | Refactor a project to use multi-stage. Compare image sizes. |
| **16** | Security: non-root, capabilities | Add `USER`, `--cap-drop`, `--read-only` to your project |
| **17** | Image scanning | Scan your images with `docker scout cves`, fix vulnerabilities |
| **18** | Logging & Resource limits | Configure log rotation, set memory/CPU limits |
| **19** | Restart policies & Healthchecks | Add `restart: unless-stopped` and healthchecks to Compose |
| **20** | Private Registry | Run a local registry, push/pull images to it |
| **21** | Review & Practice | Secure and optimize your full project |

### Week 4: Advanced & Interview (Days 22–30)

| Day | Topic | Hands-On Practice |
|---|---|---|
| **22** | BuildKit features | Use cache mounts, build secrets, multi-platform builds |
| **23** | Docker Swarm basics | Init Swarm, create services, scale, rolling updates |
| **24** | Overlay networks & Stacks | Deploy a stack across nodes |
| **25** | Docker Context | Set up contexts for local and remote Docker hosts |
| **26** | Troubleshooting | Practice debugging: follow the debugging workflow on broken containers |
| **27** | Performance optimization | Reduce image sizes, speed up builds, tune runtime |
| **28** | Interview prep: Beginner Q&A | Answer all beginner/intermediate questions from memory |
| **29** | Interview prep: Advanced Q&A | Work through scenario-based problems and case studies |
| **30** | Final Project | Build, secure, optimize, and deploy a production-ready multi-container app |

---

## Recommended Official Documentation Links

### Getting Started
| Topic | URL |
|---|---|
| Docker Overview | [docs.docker.com/get-started/overview/](https://docs.docker.com/get-started/overview/) |
| Get Docker | [docs.docker.com/get-docker/](https://docs.docker.com/get-docker/) |
| Getting Started Guide | [docs.docker.com/get-started/](https://docs.docker.com/get-started/) |

### Core References
| Topic | URL |
|---|---|
| Dockerfile Reference | [docs.docker.com/reference/dockerfile/](https://docs.docker.com/reference/dockerfile/) |
| Docker CLI Reference | [docs.docker.com/reference/cli/docker/](https://docs.docker.com/reference/cli/docker/) |
| Docker Compose Reference | [docs.docker.com/compose/compose-file/](https://docs.docker.com/compose/compose-file/) |
| Docker Hub | [hub.docker.com](https://hub.docker.com) |

### Building
| Topic | URL |
|---|---|
| Build Best Practices | [docs.docker.com/build/building/best-practices/](https://docs.docker.com/build/building/best-practices/) |
| Multi-Stage Builds | [docs.docker.com/build/building/multi-stage/](https://docs.docker.com/build/building/multi-stage/) |
| BuildKit | [docs.docker.com/build/buildkit/](https://docs.docker.com/build/buildkit/) |
| Build Cache | [docs.docker.com/build/cache/](https://docs.docker.com/build/cache/) |

### Networking
| Topic | URL |
|---|---|
| Networking Overview | [docs.docker.com/network/](https://docs.docker.com/network/) |
| Bridge Network | [docs.docker.com/network/drivers/bridge/](https://docs.docker.com/network/drivers/bridge/) |
| Overlay Network | [docs.docker.com/network/drivers/overlay/](https://docs.docker.com/network/drivers/overlay/) |

### Storage
| Topic | URL |
|---|---|
| Storage Overview | [docs.docker.com/storage/](https://docs.docker.com/storage/) |
| Volumes | [docs.docker.com/storage/volumes/](https://docs.docker.com/storage/volumes/) |
| Bind Mounts | [docs.docker.com/storage/bind-mounts/](https://docs.docker.com/storage/bind-mounts/) |

### Security
| Topic | URL |
|---|---|
| Security Overview | [docs.docker.com/engine/security/](https://docs.docker.com/engine/security/) |
| Rootless Mode | [docs.docker.com/engine/security/rootless/](https://docs.docker.com/engine/security/rootless/) |
| Content Trust | [docs.docker.com/engine/security/trust/](https://docs.docker.com/engine/security/trust/) |
| Docker Scout | [docs.docker.com/scout/](https://docs.docker.com/scout/) |

### Production
| Topic | URL |
|---|---|
| Logging | [docs.docker.com/config/containers/logging/](https://docs.docker.com/config/containers/logging/) |
| Restart Policies | [docs.docker.com/config/containers/start-containers-automatically/](https://docs.docker.com/config/containers/start-containers-automatically/) |
| Resource Constraints | [docs.docker.com/config/containers/resource_constraints/](https://docs.docker.com/config/containers/resource_constraints/) |

### Swarm & Orchestration
| Topic | URL |
|---|---|
| Swarm Overview | [docs.docker.com/engine/swarm/](https://docs.docker.com/engine/swarm/) |
| Swarm Tutorial | [docs.docker.com/engine/swarm/swarm-tutorial/](https://docs.docker.com/engine/swarm/swarm-tutorial/) |
| Stack Deploy | [docs.docker.com/engine/swarm/stack-deploy/](https://docs.docker.com/engine/swarm/stack-deploy/) |

### Docker Desktop & Extensions
| Topic | URL |
|---|---|
| Docker Desktop | [docs.docker.com/desktop/](https://docs.docker.com/desktop/) |
| Extensions | [docs.docker.com/desktop/extensions/](https://docs.docker.com/desktop/extensions/) |
| Docker Context | [docs.docker.com/engine/context/working-with-contexts/](https://docs.docker.com/engine/context/working-with-contexts/) |

---

## Quick Reference Card

```
┌──────────────────────────────────────────────────────────────┐
│               DOCKER QUICK REFERENCE CARD                    │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  BUILD          docker build -t name:tag .                   │
│  RUN            docker run -d -p 8080:80 --name web nginx    │
│  LIST           docker ps -a                                 │
│  LOGS           docker logs -f web                           │
│  SHELL          docker exec -it web bash                     │
│  STOP           docker stop web                              │
│  REMOVE         docker rm web                                │
│  IMAGES         docker images                                │
│  PULL           docker pull nginx:latest                     │
│  PUSH           docker push user/image:tag                   │
│  NETWORK        docker network create mynet                  │
│  VOLUME         docker volume create mydata                  │
│  COMPOSE UP     docker compose up -d                         │
│  COMPOSE DOWN   docker compose down -v                       │
│  CLEANUP        docker system prune -a --volumes             │
│  INSPECT        docker inspect web                           │
│  STATS          docker stats                                 │
│  SCAN           docker scout cves image:tag                  │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

**Congratulations!** You now have a complete reference from Docker zero to advanced. Follow the 30-day roadmap, practice daily, and use the official docs as your source of truth.

> *These notes are based on the official Docker documentation at [docs.docker.com](https://docs.docker.com). Always refer to the latest official docs for the most up-to-date information.*
