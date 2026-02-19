# 4. DOCKERFILE DEEP DIVE

> **Source**: [docs.docker.com/reference/dockerfile/](https://docs.docker.com/reference/dockerfile/)

---

## 4.1 All Major Instructions

### FROM — Base Image

Every Dockerfile **must** start with `FROM`. It sets the base image.

```dockerfile
FROM ubuntu:22.04
FROM python:3.11-slim
FROM node:20-alpine
FROM scratch                # Empty base (for static binaries)
```

**Best Practice:** Use specific tags (`python:3.11-slim`), never rely on `:latest` for reproducible builds.

---

### RUN — Execute Commands During Build

Runs a command and creates a new layer.

```dockerfile
# Shell form
RUN apt-get update && apt-get install -y curl

# Exec form
RUN ["apt-get", "install", "-y", "curl"]

# Best practice: chain with && and clean up in same layer
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
      curl \
      wget \
    && rm -rf /var/lib/apt/lists/*
```

---

### CMD — Default Command at Runtime

Sets the **default** command when running a container. Can be overridden by the user.

```dockerfile
# Exec form (preferred)
CMD ["python", "app.py"]

# Shell form
CMD python app.py

# As parameters to ENTRYPOINT
CMD ["--port", "8080"]
```

**Only the LAST `CMD` in a Dockerfile takes effect.**

---

### ENTRYPOINT — Fixed Command

Like `CMD`, but **cannot be easily overridden**. Used when the container should always run a specific executable.

```dockerfile
ENTRYPOINT ["python", "app.py"]
```

### CMD + ENTRYPOINT Together

```dockerfile
ENTRYPOINT ["python", "app.py"]
CMD ["--port", "8080"]
# Result: python app.py --port 8080

# User can override CMD part:
# docker run myapp --port 3000
# Result: python app.py --port 3000
```

| | CMD | ENTRYPOINT |
|---|---|---|
| Override | `docker run myimg <command>` overrides CMD | Need `--entrypoint` flag to override |
| Use Case | Default args / full default command | Fixed executable |

---

### COPY — Copy Files from Host

```dockerfile
COPY requirements.txt /app/
COPY . /app/
COPY --chown=appuser:appgroup files/ /app/
```

---

### ADD — Copy with Extras

Like `COPY` but also:
- Auto-extracts `.tar` archives
- Supports URLs

```dockerfile
ADD app.tar.gz /app/         # Extracts automatically
ADD https://example.com/f /app/   # Downloads from URL
```

**Best Practice:** Prefer `COPY` over `ADD` unless you need auto-extraction or URL fetching.

---

### WORKDIR — Set Working Directory

```dockerfile
WORKDIR /app
# All subsequent commands run in /app
RUN pwd          # /app
COPY . .         # Copies to /app
CMD ["python", "app.py"]   # Runs in /app
```

Creates the directory if it doesn't exist. Use absolute paths.

---

### ENV — Set Environment Variables

```dockerfile
ENV APP_ENV=production
ENV PORT=8080
ENV DB_HOST=localhost DB_PORT=5432
```

Available during build AND runtime. Override at runtime with `docker run -e`.

---

### ARG — Build-time Variables

```dockerfile
ARG PYTHON_VERSION=3.11
FROM python:${PYTHON_VERSION}-slim

ARG BUILD_DATE
LABEL build-date=$BUILD_DATE
```

```bash
docker build --build-arg PYTHON_VERSION=3.12 --build-arg BUILD_DATE=$(date) .
```

**Key Difference:** `ARG` = build-time only. `ENV` = build-time + runtime.

---

### EXPOSE — Document Ports

```dockerfile
EXPOSE 80
EXPOSE 443
EXPOSE 8080/udp
```

**Does NOT publish ports** — it's documentation. You still need `-p` at runtime.

---

### VOLUME — Declare Mount Points

```dockerfile
VOLUME /data
VOLUME ["/data", "/logs"]
```

Creates an anonymous volume at the specified path. Prefer named volumes via `docker run -v`.

---

### Other Instructions

```dockerfile
# LABEL — Metadata
LABEL maintainer="you@example.com"
LABEL version="1.0"

# USER — Run as non-root
RUN adduser --disabled-password appuser
USER appuser

# HEALTHCHECK — Container health monitoring
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD curl -f http://localhost:8080/health || exit 1

# SHELL — Change default shell
SHELL ["/bin/bash", "-c"]

# STOPSIGNAL — Signal for graceful shutdown
STOPSIGNAL SIGTERM
```

---

## 4.2 Multi-Stage Builds

> **Source**: [docs.docker.com/build/building/multi-stage/](https://docs.docker.com/build/building/multi-stage/)

Multi-stage builds let you use multiple `FROM` statements to create intermediate images, then copy only what you need into the final image. Result: **smaller, more secure images**.

```dockerfile
# Stage 1: Build
FROM node:20 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: Production
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

```
Without multi-stage:   ~1.2 GB (Node + npm + source + build tools)
With multi-stage:      ~25 MB  (Nginx + built static files only)
```

### Go Example

```dockerfile
# Build stage
FROM golang:1.21 AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 go build -o myapp

# Production stage
FROM scratch
COPY --from=builder /app/myapp /myapp
ENTRYPOINT ["/myapp"]
```

Final image: **just the binary** — a few MB.

---

## 4.3 Build Cache

Docker caches each layer. If nothing changes, the cached layer is reused.

**Cache Rules:**
1. If the Dockerfile instruction hasn't changed → cache hit
2. For `COPY`/`ADD` → file checksums are compared
3. Once a cache miss occurs, **all subsequent layers are rebuilt**

**Optimize for cache:**
```dockerfile
# ✅ GOOD — dependencies cached separately from code
COPY package.json package-lock.json ./
RUN npm ci
COPY . .  # Only this layer rebuilds when code changes

# ❌ BAD — everything rebuilds on any change
COPY . .
RUN npm ci
```

```bash
# Build without cache
docker build --no-cache -t myapp .

# Use BuildKit for better caching
DOCKER_BUILDKIT=1 docker build -t myapp .
```

---

## 4.4 Best Practices (Official Docs)

> **Source**: [docs.docker.com/build/building/best-practices/](https://docs.docker.com/build/building/best-practices/)

1. **Use `.dockerignore`** — exclude files not needed in the image
   ```
   # .dockerignore
   .git
   node_modules
   *.md
   .env
   __pycache__
   .vscode
   ```

2. **Minimize layers** — combine related `RUN` commands
   ```dockerfile
   RUN apt-get update && \
       apt-get install -y --no-install-recommends curl && \
       rm -rf /var/lib/apt/lists/*
   ```

3. **Use multi-stage builds** — keep production images lean

4. **Use specific base image tags** — not `:latest`

5. **Order instructions for caching** — least-changing first

6. **Don't install unnecessary packages** — use `--no-install-recommends`

7. **Run as non-root user**
   ```dockerfile
   RUN addgroup -S appgroup && adduser -S appuser -G appgroup
   USER appuser
   ```

8. **Use `COPY` instead of `ADD`** — unless you need auto-extraction

9. **One container = one process** — don't run multiple services in one container

10. **Label your images** — for organization and automation

---

## 4.5 Security Recommendations

> **Source**: [docs.docker.com/build/building/best-practices/#security](https://docs.docker.com/build/building/best-practices/)

```dockerfile
# 1. Never hardcode secrets
# ❌ BAD
ENV API_KEY=sk-12345

# ✅ GOOD — pass at runtime
# docker run -e API_KEY=$API_KEY myapp

# 2. Use BuildKit secrets for build-time secrets
# docker build --secret id=mysecret,src=./secret.txt .
RUN --mount=type=secret,id=mysecret cat /run/secrets/mysecret

# 3. Scan images for vulnerabilities
# docker scout cves myapp:1.0

# 4. Use minimal base images
FROM python:3.11-slim    # Not python:3.11
FROM node:20-alpine      # Not node:20

# 5. Pin package versions
RUN pip install flask==3.0.0

# 6. Use non-root user (see above)
```

---

> **Next**: [05-Networking.md](./05-Networking.md) →
