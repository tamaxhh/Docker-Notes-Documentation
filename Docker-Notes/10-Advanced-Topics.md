# 10. ADVANCED TOPICS

> **Source**: Various sections of [docs.docker.com](https://docs.docker.com)

---

## 10.1 Docker Swarm (Extended)

> **Source**: [docs.docker.com/engine/swarm/](https://docs.docker.com/engine/swarm/)

### Service Types

| Type | Description | Example |
|---|---|---|
| **Replicated** | Runs N copies across nodes | Web servers, APIs |
| **Global** | Runs exactly one copy per node | Monitoring agents, log collectors |

```bash
# Replicated service
docker service create --name web --replicas 3 nginx

# Global service
docker service create --name monitor --mode global datadog/agent

# Rolling update
docker service update \
  --update-parallelism 2 \
  --update-delay 10s \
  --image nginx:1.26 \
  web

# Rollback
docker service rollback web
```

### Stack Deployments

```bash
# Deploy a stack from compose file
docker stack deploy -c compose.yaml mystack

# List stacks
docker stack ls

# List services in a stack
docker stack services mystack

# Remove a stack
docker stack rm mystack
```

### Swarm Secrets & Configs

```bash
# Create a secret
echo "my-password" | docker secret create db_password -

# Use in service
docker service create --name web --secret db_password nginx
# Secret available at /run/secrets/db_password inside container

# Create config
docker config create nginx_conf ./nginx.conf

# Use in service
docker service create --name web --config nginx_conf nginx
```

---

## 10.2 Docker Registry (Private)

> **Source**: [docs.docker.com/registry/](https://docs.docker.com/registry/)

```bash
# Run a private registry
docker run -d -p 5000:5000 --name registry \
  -v registry-data:/var/lib/registry \
  registry:2

# Push an image to it
docker tag myapp:1.0 localhost:5000/myapp:1.0
docker push localhost:5000/myapp:1.0

# Pull from it
docker pull localhost:5000/myapp:1.0

# List images in registry (API)
curl http://localhost:5000/v2/_catalog

# Secure with TLS
docker run -d -p 5000:5000 \
  -v /certs:/certs \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \
  registry:2
```

---

## 10.3 BuildKit

> **Source**: [docs.docker.com/build/buildkit/](https://docs.docker.com/build/buildkit/)

BuildKit is Docker's **next-generation build engine**. It's the default since Docker 23.0.

**Key Features:**
- Parallel build stages
- Better caching (including remote cache)
- Build secrets (never stored in image)
- SSH forwarding during builds
- Heredoc support in Dockerfiles

```bash
# Enable BuildKit (if not default)
DOCKER_BUILDKIT=1 docker build -t myapp .

# Or set permanently in /etc/docker/daemon.json
# { "features": { "buildkit": true } }
```

### Build Secrets

```dockerfile
# Pass a secret during build (never stored in image layers)
RUN --mount=type=secret,id=npmrc,target=/root/.npmrc npm install
```

```bash
docker build --secret id=npmrc,src=$HOME/.npmrc -t myapp .
```

### SSH Forwarding

```dockerfile
# Clone private repos during build
RUN --mount=type=ssh git clone git@github.com:user/private-repo.git
```

```bash
docker build --ssh default -t myapp .
```

### Cache Mounts

```dockerfile
# Cache package manager downloads between builds
RUN --mount=type=cache,target=/root/.cache/pip \
    pip install -r requirements.txt
```

### Multi-Platform Builds

```bash
# Create builder for multi-platform
docker buildx create --name mybuilder --use
docker buildx inspect --bootstrap

# Build for multiple platforms
docker buildx build --platform linux/amd64,linux/arm64 -t myapp:1.0 --push .
```

---

## 10.4 Docker Context

> **Source**: [docs.docker.com/engine/context/working-with-contexts/](https://docs.docker.com/engine/context/working-with-contexts/)

Docker contexts let you switch between multiple Docker endpoints (local, remote, cloud).

```bash
# List contexts
docker context ls

# Create a context for a remote Docker host
docker context create remote-server \
  --docker "host=ssh://user@remote-server"

# Switch context
docker context use remote-server

# Now all docker commands target the remote server
docker ps   # Shows remote containers!

# Switch back
docker context use default
```

---

## 10.5 Docker Extensions

> **Source**: [docs.docker.com/desktop/extensions/](https://docs.docker.com/desktop/extensions/)

Docker Extensions add extra functionality to Docker Desktop through a marketplace:

- **Disk Usage** — visualize what's consuming disk
- **Logs Explorer** — advanced log viewer
- **Snyk** — vulnerability scanning
- **Portainer** — container management UI
- **Tailscale** — private networking

```bash
# Install an extension (via Docker Desktop or CLI)
docker extension install portainer/portainer-docker-extension

# List installed extensions
docker extension ls

# Remove
docker extension rm portainer/portainer-docker-extension
```

---

## 10.6 Docker Desktop Architecture

```
┌──────────────────────────────────────────────────────┐
│                  DOCKER DESKTOP                      │
│                                                      │
│  ┌──────────┐  ┌──────────┐  ┌──────────────────┐   │
│  │ Docker   │  │ Docker   │  │   Kubernetes     │   │
│  │ CLI      │  │ Compose  │  │   (optional)     │   │
│  └────┬─────┘  └────┬─────┘  └────────┬─────────┘   │
│       │              │                 │             │
│       ▼              ▼                 ▼             │
│  ┌───────────────────────────────────────────────┐   │
│  │              Docker Engine                    │   │
│  │  (runs inside a lightweight Linux VM)         │   │
│  │                                               │   │
│  │  Windows: WSL 2 or Hyper-V backend            │   │
│  │  macOS: Apple Virtualization Framework        │   │
│  │  Linux: Native                                │   │
│  └───────────────────────────────────────────────┘   │
│                                                      │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐           │
│  │ Volume   │  │ BuildKit │  │Extensions│           │
│  │ Sharing  │  │          │  │          │           │
│  └──────────┘  └──────────┘  └──────────┘           │
└──────────────────────────────────────────────────────┘
```

**Key Points:**
- On **Windows** and **macOS**, Docker runs inside a lightweight Linux VM
- **WSL 2** is the recommended backend on Windows (better performance)
- Docker Desktop includes: Engine, CLI, Compose, BuildKit, Kubernetes, Extensions
- **Resource settings** (CPU, Memory, Disk) configured in Docker Desktop Settings

---

> **Next**: [11-Troubleshooting.md](./11-Troubleshooting.md) →
