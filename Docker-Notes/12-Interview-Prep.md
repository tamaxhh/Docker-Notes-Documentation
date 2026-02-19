# 12. INTERVIEW PREPARATION

---

## 12.1 Frequently Asked Docker Interview Questions

### Beginner Level

**Q1: What is Docker?**
> Docker is an open platform for developing, shipping, and running applications in containers. Containers package an application with all its dependencies so it runs consistently across environments.

**Q2: What is the difference between a container and a virtual machine?**
> Containers share the host OS kernel, are lightweight (MBs), and start in seconds. VMs include a full guest OS, are heavy (GBs), and take minutes to start. Containers use process-level isolation (namespaces), VMs use hardware-level isolation (hypervisor).

**Q3: What is a Dockerfile?**
> A text file containing step-by-step instructions to build a Docker image. Each instruction creates a layer in the image.

**Q4: What is the difference between `CMD` and `ENTRYPOINT`?**
> `CMD` provides default arguments that can be overridden at runtime. `ENTRYPOINT` sets the fixed executable. When combined, `CMD` provides default arguments to `ENTRYPOINT`.

**Q5: What is the difference between `COPY` and `ADD`?**
> Both copy files from host to image. `ADD` has extras: auto-extracts tar archives and supports URLs. Best practice: use `COPY` unless you need those features.

**Q6: What are Docker volumes?**
> Volumes are Docker-managed storage that persists data beyond the container lifecycle. They're stored in Docker's storage area and survive container deletion.

**Q7: What is Docker Hub?**
> Docker Hub is the default public registry where Docker images are stored and shared. Like GitHub for container images.

---

### Intermediate Level

**Q8: Explain Docker networking. What are the network types?**
> Docker has four network drivers:
> - **Bridge** (default): Isolated network on a single host. Custom bridges support DNS.
> - **Host**: Container shares host's network stack. No isolation, best performance.
> - **Overlay**: Multi-host networking for Swarm clusters.
> - **Macvlan**: Container gets a real MAC address, appears as physical device on network.

**Q9: What are Docker layers? How does caching work?**
> Each Dockerfile instruction creates an immutable layer. Docker caches layers and reuses them if the instruction and its inputs haven't changed. Once a layer's cache is invalidated, all subsequent layers are rebuilt. Optimization: put least-changing instructions first.

**Q10: What is the difference between `docker stop` and `docker kill`?**
> `docker stop` sends SIGTERM, waits 10 seconds (grace period), then sends SIGKILL. `docker kill` sends SIGKILL immediately. Use `stop` for graceful shutdown.

**Q11: What is a multi-stage build?**
> Using multiple `FROM` statements in a Dockerfile. Build tools and dependencies stay in early stages; only the final artifacts are copied to the production stage. Results in much smaller images.

**Q12: How do containers communicate with each other?**
> Containers on the same custom bridge network can communicate by container name (Docker's built-in DNS at 127.0.0.11). Containers on the default bridge can only communicate by IP address.

**Q13: What is Docker Compose?**
> A tool for defining and running multi-container applications using a YAML file. One `docker compose up` command creates networks, volumes, and starts all services.

**Q14: What is the difference between `docker exec` and `docker attach`?**
> `exec` starts a new process inside a running container. `attach` connects to the container's main process (PID 1). `exec` is safer — exiting won't stop the container.

---

### Advanced Level

**Q15: How does Docker achieve isolation?**
> Docker uses Linux kernel features:
> - **Namespaces**: Isolate PID, network, filesystem, IPC, hostname, and user IDs
> - **cgroups**: Limit CPU, memory, I/O, and PIDs
> - **Union filesystem**: Layered filesystem with copy-on-write

**Q16: What is Docker Swarm? How does it differ from Kubernetes?**
> Swarm is Docker's built-in orchestration. It's simpler to set up but less feature-rich. Kubernetes is more complex but offers more flexibility, auto-scaling, and a larger ecosystem. Swarm: `docker service`, Kubernetes: `kubectl`.

**Q17: Explain Docker's storage drivers.**
> Storage drivers manage the layered filesystem:
> - **overlay2** (default, recommended): Efficient, good performance
> - **devicemapper**: Block-level, used in older systems
> - **btrfs/zfs**: For those specific filesystems
> Layers are read-only; containers add a thin writable layer on top.

**Q18: How would you troubleshoot a container that keeps restarting?**
> 1. `docker ps -a` — check exit code
> 2. `docker logs <container>` — check application errors
> 3. `docker inspect <container>` — check config (env vars, mounts, restart policy)
> 4. `docker stats` — check if OOM killed (exit code 137)
> 5. `docker events` — check Docker-level events
> 6. If needed: `docker run -it --entrypoint sh <image>` — debug interactively

**Q19: How do you secure Docker in production?**
> - Run as non-root user (`USER` instruction)
> - Drop capabilities (`--cap-drop ALL`)
> - Use read-only filesystem (`--read-only`)
> - Scan images (`docker scout cves`)
> - Use minimal base images (alpine/distroless)
> - Never use `--privileged`
> - Enable content trust (`DOCKER_CONTENT_TRUST=1`)
> - Set resource limits (`--memory`, `--cpus`)
> - Use rootless mode

**Q20: What is BuildKit and why is it important?**
> BuildKit is Docker's next-gen build engine (default since Docker 23.0). Benefits:
> - Parallel build stages
> - Better caching
> - Build-time secrets (never stored in layers)
> - SSH forwarding
> - Cache mounts for package managers
> - Multi-platform builds via `docker buildx`

---

## 12.2 Scenario-Based Problems

### Scenario 1: Application works in dev, fails in production

**Situation:** Your Node.js app runs perfectly locally but crashes in the Docker container with "module not found" errors.

**Diagnosis:**
```bash
docker logs myapp              # Check error details
docker exec -it myapp sh       # Check if node_modules exists
docker exec -it myapp ls /app/node_modules
```

**Common Causes:**
1. `.dockerignore` excludes `node_modules` (correct), but `npm install` fails silently
2. Native modules compiled for wrong architecture
3. Missing build tools in production image (multi-stage build removes them)

**Fix:**
```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .

FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app .
CMD ["node", "server.js"]
```

---

### Scenario 2: Container running out of memory

**Situation:** Docker container is killed with exit code 137.

**Diagnosis:**
```bash
docker inspect <container> | grep -i "OOM"
docker stats               # Watch memory usage
```

**Fix:**
```bash
# Increase memory limit
docker run --memory="1g" --memory-swap="2g" myapp

# Or fix the memory leak in your application
# Add healthcheck to auto-restart
docker run --memory="512m" --restart=on-failure:3 \
  --health-cmd="curl -f localhost:8080/health || exit 1" myapp
```

---

### Scenario 3: Containers can't communicate

**Situation:** `web` container can't connect to `db` container.

**Diagnosis:**
```bash
# Check if on same network
docker network inspect bridge
docker inspect web --format '{{.NetworkSettings.Networks}}'
docker inspect db --format '{{.NetworkSettings.Networks}}'

# Test connectivity from inside web
docker exec -it web ping db
docker exec -it web nslookup db
```

**Fix:**
```bash
# Create custom network (enables DNS)
docker network create app-net
docker run -d --name db --network app-net postgres
docker run -d --name web --network app-net -e DB_HOST=db myapp
```

---

### Scenario 4: Docker build is very slow

**Diagnosis:**
```bash
# Check build context size
# Look at "Sending build context to Docker daemon" message

# Check layer caching
docker image history myapp
```

**Fixes:**
1. Add `.dockerignore` (exclude `.git`, `node_modules`, etc.)
2. Reorder Dockerfile — put `COPY package.json` before `COPY . .`
3. Use BuildKit: `DOCKER_BUILDKIT=1 docker build .`
4. Use cache mounts: `RUN --mount=type=cache,target=/root/.cache/pip ...`

---

### Scenario 5: Data lost after container restart

**Diagnosis:** Data was stored in the container's writable layer, not a volume.

**Fix:**
```bash
# Use a named volume
docker run -v mydata:/app/data myapp

# For databases, always use volumes
docker run -v pgdata:/var/lib/postgresql/data postgres
```

---

## 12.3 Practical Debugging Case Studies

### Case Study 1: High CPU Usage

```bash
# Step 1: Identify the culprit
docker stats --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"

# Step 2: See processes inside container
docker top <container>

# Step 3: Get a shell and use top/htop
docker exec -it <container> sh
top

# Step 4: Limit CPU
docker update --cpus="1.0" <container>
```

### Case Study 2: Image Vulnerability Report

```bash
# Step 1: Scan the image
docker scout cves myapp:production

# Step 2: Check which base image has issues
docker scout recommendations myapp:production

# Step 3: Update base image
# Change FROM python:3.11-slim → FROM python:3.11.7-slim

# Step 4: Rebuild and re-scan
docker build -t myapp:production .
docker scout cves myapp:production
```

### Case Study 3: Network Connectivity Debug

```bash
# Step 1: Run a debug container on the same network
docker run --rm -it --network mynet alpine sh

# Step 2: Install tools
apk add --no-cache curl bind-tools

# Step 3: Test DNS
nslookup api
nslookup db

# Step 4: Test connectivity
curl -v http://api:8080/health
ping db

# Step 5: Check firewall (from host)
iptables -L -n | grep DOCKER    # Linux only
```

---

> **Next**: [13-Roadmap-References.md](./13-Roadmap-References.md) →
