# 11. TROUBLESHOOTING

---

## 11.1 Common Docker Errors

### Error: "port is already allocated"

```
Error: driver failed programming external connectivity:
Bind for 0.0.0.0:8080 failed: port is already allocated
```

**Fix:**
```bash
# Find what's using the port
# Linux/Mac:
lsof -i :8080
# Windows:
netstat -ano | findstr :8080

# Either stop that process, or use a different port
docker run -p 8081:80 nginx
```

---

### Error: "no space left on device"

```bash
# Check Docker disk usage
docker system df

# Clean up everything unused
docker system prune -a --volumes

# Check specific culprits
docker images --format '{{.Size}}\t{{.Repository}}:{{.Tag}}' | sort -rh | head -20
```

---

### Error: "image not found" / "manifest unknown"

```
Error: pull access denied, repository does not exist or may require authentication
```

**Fix:**
```bash
# Check image name / tag spelling
docker search <image-name>

# Login if private registry
docker login

# Check if tag exists on Docker Hub
# Visit: hub.docker.com/r/<user>/<image>/tags
```

---

### Error: Container exits immediately

```bash
# Check exit code and logs
docker ps -a              # See exit code in STATUS column
docker logs <container>   # See what happened

# Common causes:
# Exit code 0   → process completed (normal for one-shot commands)
# Exit code 1   → application error (check logs)
# Exit code 137 → OOM killed (increase memory limit)
# Exit code 139 → segfault
# Exit code 143 → SIGTERM (normal graceful shutdown)

# Keep container running for debugging
docker run -it --entrypoint sh myimage
```

---

### Error: "Cannot connect to the Docker daemon"

```
Cannot connect to the Docker daemon at unix:///var/run/docker.sock.
Is the docker daemon running?
```

**Fix:**
```bash
# Linux: Start Docker daemon
sudo systemctl start docker
sudo systemctl enable docker

# Windows/Mac: Start Docker Desktop application

# Check Docker daemon status
sudo systemctl status docker

# Permission issue? Add user to docker group
sudo usermod -aG docker $USER
# Then logout and login
```

---

### Error: Build context too large

```
Sending build context to Docker daemon  2.5GB
```

**Fix:** Create/update `.dockerignore`:
```
.git
node_modules
*.md
.env
__pycache__
.vscode
dist
build
*.tar.gz
```

---

### Error: DNS resolution failure inside container

```bash
# Test DNS from inside container
docker run --rm alpine nslookup google.com

# Fix: Specify DNS server
docker run --dns 8.8.8.8 myapp

# Or configure daemon-wide in /etc/docker/daemon.json
# { "dns": ["8.8.8.8", "8.8.4.4"] }
```

---

### Error: "exec format error"

```
exec /usr/bin/myapp: exec format error
```

**Cause:** Architecture mismatch (e.g., built for ARM, running on AMD64).

```bash
# Check image architecture
docker image inspect myapp | grep Architecture

# Build for correct platform
docker buildx build --platform linux/amd64 -t myapp .
```

---

## 11.2 Debugging Workflow

```
Container not working?
        │
        ▼
┌───────────────────┐
│ 1. Check status   │  docker ps -a
│    & exit code     │
└───────┬───────────┘
        │
        ▼
┌───────────────────┐
│ 2. Check logs      │  docker logs <container>
│                    │  docker logs --tail 50 <container>
└───────┬───────────┘
        │
        ▼
┌───────────────────┐
│ 3. Inspect config  │  docker inspect <container>
│    (env, mounts,   │  Check env vars, volumes,
│     network)       │  network settings
└───────┬───────────┘
        │
        ▼
┌───────────────────┐
│ 4. Get a shell     │  docker exec -it <container> sh
│    inside          │  Check files, processes, network
└───────┬───────────┘
        │
        ▼
┌───────────────────┐
│ 5. Check network   │  docker network inspect <net>
│                    │  ping / curl from inside container
└───────┬───────────┘
        │
        ▼
┌───────────────────┐
│ 6. Check resources │  docker stats
│                    │  docker system df
└───────┬───────────┘
        │
        ▼
┌───────────────────┐
│ 7. Check events    │  docker system events
│                    │  (real-time Docker events)
└───────────────────┘
```

### Useful Debugging Commands

```bash
# See all processes in a container
docker top <container>

# See real-time stats
docker stats

# See filesystem changes since image
docker diff <container>

# Export container filesystem for inspection
docker export <container> -o container-fs.tar

# Run a one-off debug container on same network
docker run --rm -it --network <same-net> alpine sh
# Then: ping, nslookup, wget, curl from inside
```

---

## 11.3 Performance Optimization Tips

### Image Size Optimization

```dockerfile
# 1. Use small base images
FROM node:20-alpine          # ~50MB vs ~350MB for node:20

# 2. Multi-stage builds
FROM node:20 AS builder
RUN npm run build
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html

# 3. Minimize layers
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl && \
    rm -rf /var/lib/apt/lists/*

# 4. Use .dockerignore
```

### Build Speed

```bash
# 1. Order Dockerfile for cache efficiency
# (dependencies before code)

# 2. Use BuildKit
DOCKER_BUILDKIT=1 docker build .

# 3. Use cache mounts
RUN --mount=type=cache,target=/root/.cache/pip pip install -r requirements.txt

# 4. Use multi-stage to parallelize
FROM node:20 AS frontend
FROM python:3.11 AS backend
```

### Runtime Performance

```bash
# 1. Set appropriate resource limits
docker run --cpus="2" --memory="1g" myapp

# 2. Use host network when latency matters
docker run --network host myapp

# 3. Use tmpfs for temporary data
docker run --tmpfs /tmp:rw,noexec,nosuid myapp

# 4. Monitor with docker stats
docker stats --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"
```

---

> **Next**: [12-Interview-Prep.md](./12-Interview-Prep.md) →
