# 9. DOCKER IN PRODUCTION

> **Source**: [docs.docker.com/config/containers/](https://docs.docker.com/config/containers/)

---

## 9.1 Logging Drivers

> **Source**: [docs.docker.com/config/containers/logging/](https://docs.docker.com/config/containers/logging/)

Docker captures `stdout` and `stderr` from containers and routes them via logging drivers.

| Driver | Description |
|---|---|
| `json-file` | Default. Writes JSON logs to disk |
| `syslog` | Sends to syslog daemon |
| `journald` | Writes to systemd journal |
| `fluentd` | Sends to Fluentd collector |
| `awslogs` | Amazon CloudWatch |
| `gcplogs` | Google Cloud Logging |
| `splunk` | Splunk HTTP Event Collector |
| `none` | No logging |

```bash
# Use a specific logging driver
docker run -d --log-driver=json-file --log-opt max-size=10m --log-opt max-file=3 nginx

# Set default in /etc/docker/daemon.json
# {
#   "log-driver": "json-file",
#   "log-opts": {
#     "max-size": "10m",
#     "max-file": "5"
#   }
# }

# Check a container's log driver
docker inspect --format '{{.HostConfig.LogConfig.Type}}' web
```

**Critical Production Tip:** Always set `max-size` and `max-file` — without them, logs grow forever and fill the disk.

---

## 9.2 Resource Limits

```bash
# Memory
docker run --memory="256m" myapp                     # Hard limit
docker run --memory="256m" --memory-swap="512m" myapp # Limit + swap
docker run --memory="256m" --oom-kill-disable myapp   # Don't kill on OOM (dangerous)

# CPU
docker run --cpus="1.5" myapp           # Use max 1.5 cores
docker run --cpu-shares=512 myapp       # Relative weight (default: 1024)
docker run --cpuset-cpus="0,1" myapp    # Pin to specific cores

# PIDs
docker run --pids-limit=100 myapp       # Max 100 processes
```

**In Docker Compose:**
```yaml
services:
  web:
    image: myapp
    deploy:
      resources:
        limits:
          cpus: "1.0"
          memory: 512M
        reservations:
          cpus: "0.25"
          memory: 128M
```

---

## 9.3 Restart Policies

> **Source**: [docs.docker.com/config/containers/start-containers-automatically/](https://docs.docker.com/config/containers/start-containers-automatically/)

| Policy | Behavior |
|---|---|
| `no` | Never restart (default) |
| `on-failure` | Restart only if exit code ≠ 0 |
| `on-failure:5` | Restart on failure, max 5 attempts |
| `always` | Always restart (even on manual stop after daemon restart) |
| `unless-stopped` | Like `always`, but not after manual `docker stop` |

```bash
docker run -d --restart=unless-stopped --name web nginx

# Update restart policy on existing container
docker update --restart=always web

# Check restart count
docker inspect -f '{{.RestartCount}}' web
```

**Best Practice:** Use `unless-stopped` for production services. Use `on-failure` for batch jobs.

---

## 9.4 Healthchecks

> **Source**: [docs.docker.com/reference/dockerfile/#healthcheck](https://docs.docker.com/reference/dockerfile/#healthcheck)

```dockerfile
# In Dockerfile
HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
  CMD curl -f http://localhost:8080/health || exit 1
```

```yaml
# In Compose
services:
  web:
    image: myapp
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
      interval: 30s
      timeout: 5s
      retries: 3
      start_period: 10s
```

```bash
# Check health status
docker inspect --format '{{.State.Health.Status}}' web
# Values: starting | healthy | unhealthy

# View health check log
docker inspect --format '{{json .State.Health}}' web | python -m json.tool
```

**Health States:**
```
Container starts
     │
     ▼
 [starting]  ─── start_period (grace period) ───►
     │
     ▼
 [healthy]   ◄── health check passes
     │
     ▼ (after 'retries' failures)
 [unhealthy] ─── restart policy kicks in if configured
```

---

## 9.5 Scaling Concepts

### Horizontal Scaling with Compose

```bash
# Scale a service to multiple instances
docker compose up -d --scale web=3

# Use a load balancer
```

```yaml
# compose.yaml with load balancer
services:
  lb:
    image: nginx
    ports:
      - "80:80"
    volumes:
      - ./nginx-lb.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - web

  web:
    image: myapp
    # No ports published — load balancer handles external traffic
    deploy:
      replicas: 3
```

---

## 9.6 Docker Swarm Basics

> **Source**: [docs.docker.com/engine/swarm/](https://docs.docker.com/engine/swarm/)

Docker Swarm is Docker's **built-in orchestration** tool. It turns a pool of Docker hosts into a single virtual Docker host.

```
┌──────────────────────────────────────────────┐
│                 DOCKER SWARM                 │
│                                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │ Manager  │  │ Worker   │  │ Worker   │   │
│  │  Node 1  │  │  Node 2  │  │  Node 3  │   │
│  │          │  │          │  │          │   │
│  │ ┌──────┐ │  │ ┌──────┐ │  │ ┌──────┐ │   │
│  │ │web.1 │ │  │ │web.2 │ │  │ │web.3 │ │   │
│  │ └──────┘ │  │ └──────┘ │  │ └──────┘ │   │
│  │ ┌──────┐ │  │ ┌──────┐ │  │          │   │
│  │ │db.1  │ │  │ │db.2  │ │  │          │   │
│  │ └──────┘ │  │ └──────┘ │  │          │   │
│  └──────────┘  └──────────┘  └──────────┘   │
└──────────────────────────────────────────────┘
```

```bash
# Initialize Swarm (on manager node)
docker swarm init --advertise-addr <MANAGER-IP>

# Join as worker (on worker nodes)
docker swarm join --token <TOKEN> <MANAGER-IP>:2377

# Create a service
docker service create --name web --replicas 3 -p 80:80 nginx

# List services
docker service ls

# Scale a service
docker service scale web=5

# Update a service (rolling update)
docker service update --image nginx:1.26 web

# View tasks (containers)
docker service ps web

# Remove a service
docker service rm web

# Leave Swarm
docker swarm leave           # Worker
docker swarm leave --force   # Manager
```

**Swarm Key Concepts:**
- **Node**: A Docker host in the cluster
- **Manager node**: Orchestrates workload, maintains cluster state
- **Worker node**: Runs containers
- **Service**: Definition of tasks to run (replicated or global)
- **Task**: A running container managed by Swarm
- **Ingress network**: Built-in load balancing across nodes

---

> **Next**: [10-Advanced-Topics.md](./10-Advanced-Topics.md) →
