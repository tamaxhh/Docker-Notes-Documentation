# 8. DOCKER SECURITY

> **Source**: [docs.docker.com/engine/security/](https://docs.docker.com/engine/security/)

---

## 8.1 Linux Namespaces

Docker uses Linux **namespaces** to provide isolation for containers. Each container gets its own isolated view of the system.

```
┌─────────────────────────────────────────────────────┐
│                     HOST SYSTEM                     │
│                                                     │
│  ┌─────────────────┐    ┌─────────────────┐         │
│  │   Container A   │    │   Container B   │         │
│  │                 │    │                 │         │
│  │ PID namespace   │    │ PID namespace   │         │
│  │  PID 1: nginx   │    │  PID 1: python  │         │
│  │                 │    │                 │         │
│  │ NET namespace   │    │ NET namespace   │         │
│  │  eth0: 172.17.  │    │  eth0: 172.17.  │         │
│  │  0.2            │    │  0.3            │         │
│  │                 │    │                 │         │
│  │ MNT namespace   │    │ MNT namespace   │         │
│  │  /app (own FS)  │    │  /app (own FS)  │         │
│  └─────────────────┘    └─────────────────┘         │
└─────────────────────────────────────────────────────┘
```

| Namespace | What it Isolates |
|---|---|
| **PID** | Process IDs — container sees only its own processes |
| **NET** | Network interfaces, IPs, routing tables |
| **MNT** | Filesystem mount points |
| **UTS** | Hostname and domain name |
| **IPC** | Inter-process communication |
| **USER** | User and group IDs |

---

## 8.2 Control Groups (cgroups)

**cgroups** limit and track the resources a container can use.

```bash
# Limit CPU
docker run --cpus="1.5" myapp        # Max 1.5 CPU cores
docker run --cpu-shares=512 myapp     # Relative weight

# Limit Memory
docker run --memory="512m" myapp      # Hard limit
docker run --memory="512m" --memory-swap="1g" myapp   # Memory + swap

# Limit I/O
docker run --device-read-bps /dev/sda:1mb myapp

# View resource usage
docker stats
```

**What cgroups control:**
- CPU time and cores
- Memory (RAM + swap)
- Disk I/O bandwidth
- Network bandwidth (via traffic control)
- Number of processes (PIDs)

---

## 8.3 Image Scanning

> **Source**: [docs.docker.com/scout/](https://docs.docker.com/scout/)

```bash
# Docker Scout (built-in scanner)
docker scout cves myapp:1.0
docker scout quickview myapp:1.0
docker scout recommendations myapp:1.0

# Scan during build
docker build -t myapp:1.0 .
docker scout cves myapp:1.0
```

**Best Practices:**
- Scan images in CI/CD before deploying
- Use minimal base images (alpine, distroless) to reduce attack surface
- Update base images regularly
- Pin dependency versions

---

## 8.4 Least Privilege Principle

**Run containers with minimum required permissions.**

```dockerfile
# 1. Create non-root user in Dockerfile
FROM node:20-alpine
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
WORKDIR /app
COPY --chown=appuser:appgroup . .
USER appuser
CMD ["node", "server.js"]
```

```bash
# 2. Drop capabilities
docker run --cap-drop ALL --cap-add NET_BIND_SERVICE myapp

# 3. Read-only filesystem
docker run --read-only --tmpfs /tmp myapp

# 4. No new privileges
docker run --security-opt no-new-privileges myapp

# 5. Don't run --privileged (EVER in production)
# ❌ docker run --privileged myapp   # Full host access!
```

---

## 8.5 Rootless Mode

> **Source**: [docs.docker.com/engine/security/rootless/](https://docs.docker.com/engine/security/rootless/)

Run the Docker daemon and containers **without root privileges**.

```bash
# Install rootless Docker
dockerd-rootless-setuptool.sh install

# Set environment
export PATH=$HOME/bin:$PATH
export DOCKER_HOST=unix://$XDG_RUNTIME_DIR/docker.sock

# Verify
docker info | grep "Root Dir"
# Should show a path under $HOME
```

**Benefits:**
- Even if a container is compromised, attacker has no root on host
- Protects against Docker daemon exploits

**Limitations:**
- Cannot bind to ports < 1024
- Some storage drivers not supported
- Performance overhead on some operations

---

## 8.6 Docker Bench for Security

> **Source**: [github.com/docker/docker-bench-security](https://github.com/docker/docker-bench-security)

An open-source script that checks for common security best practices around deploying Docker containers in production.

```bash
# Run Docker Bench
docker run --rm --net host --pid host --userns host \
  --cap-add audit_control \
  -v /etc:/etc:ro \
  -v /var/lib:/var/lib:ro \
  -v /var/run/docker.sock:/var/run/docker.sock:ro \
  docker/docker-bench-security

# It checks:
# [PASS] 1.1 - Ensure a separate partition for containers
# [WARN] 2.1 - Ensure network traffic is restricted
# [PASS] 4.1 - Ensure a user for the container has been created
# ...
```

---

## 8.7 Docker Content Trust (DCT)

> **Source**: [docs.docker.com/engine/security/trust/](https://docs.docker.com/engine/security/trust/)

Ensures you're pulling images that are **signed and verified** by the publisher.

```bash
# Enable content trust
export DOCKER_CONTENT_TRUST=1

# Now pulls/pushes require signed images
docker pull nginx         # Only works if nginx image is signed
docker push myapp:1.0     # Signs the image before pushing

# Disable
export DOCKER_CONTENT_TRUST=0
```

---

## 8.8 Security Checklist Summary

| Practice | Command / Action |
|---|---|
| Run as non-root | `USER appuser` in Dockerfile |
| Drop capabilities | `--cap-drop ALL --cap-add <needed>` |
| Read-only filesystem | `--read-only --tmpfs /tmp` |
| No privileged mode | Never use `--privileged` |
| Scan images | `docker scout cves <image>` |
| Use minimal bases | `alpine`, `slim`, `distroless` |
| Pin versions | `FROM python:3.11.7-slim` |
| Enable content trust | `DOCKER_CONTENT_TRUST=1` |
| Rootless mode | `dockerd-rootless-setuptool.sh` |
| Audit with Bench | `docker/docker-bench-security` |
| Limit resources | `--memory`, `--cpus` |
| No secrets in images | Use `--env-file` or secrets managers |

---

> **Next**: [09-Production.md](./09-Production.md) →
