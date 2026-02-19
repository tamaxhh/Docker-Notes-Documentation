# 3. WORKFLOW & COMMANDS

> **Source**: [docs.docker.com/reference/cli/docker/](https://docs.docker.com/reference/cli/docker/)

---

## 3.1 Image Lifecycle

```
┌───────────┐    ┌───────────┐    ┌───────────┐    ┌───────────┐
│  Write    │    │           │    │ Push to   │    │  Pull on  │
│ Dockerfile├───►│ docker    ├───►│ Registry  ├───►│  another  │
│           │    │ build     │    │           │    │  machine  │
└───────────┘    └───────────┘    └───────────┘    └───────────┘
                      │
                      ▼
               ┌───────────┐
               │ docker    │
               │ run       │  ← Creates container
               └───────────┘
```

```bash
# Build → Tag → Push → Pull
docker build -t myapp:1.0 .       # Build
docker tag myapp:1.0 user/myapp   # Tag for registry
docker push user/myapp:1.0        # Push to registry
docker pull user/myapp:1.0        # Pull on another machine
docker rmi myapp:1.0              # Remove locally
```

---

## 3.2 Container Lifecycle

```
           docker create
                │
                ▼
┌──────────────────────────┐
│         CREATED          │
└────────────┬─────────────┘
             │ docker start
             ▼
┌──────────────────────────┐
│         RUNNING          │◄──── docker restart
└──┬────────────┬──────────┘
   │            │
   │ docker     │ docker stop
   │ pause      │ (SIGTERM → SIGKILL)
   ▼            ▼
┌────────┐  ┌──────────────────┐
│ PAUSED │  │     STOPPED      │
└────────┘  │    (EXITED)      │
            └────────┬─────────┘
                     │ docker rm
                     ▼
            ┌──────────────────┐
            │     REMOVED      │
            └──────────────────┘
```

```bash
# Full lifecycle
docker create --name web nginx    # Created (not running)
docker start web                  # Running
docker pause web                  # Paused
docker unpause web                # Running again
docker stop web                   # Stopped (graceful: SIGTERM, then SIGKILL after 10s)
docker kill web                   # Stopped (immediate: SIGKILL)
docker rm web                     # Removed

# Shortcut: docker run = create + start
docker run -d --name web nginx
```

---

## 3.3 Essential CLI Commands Reference

### System Commands
```bash
docker version                    # Client & Server version
docker info                       # System-wide info (containers, images, storage)
docker system df                  # Disk usage by Docker
docker system events              # Real-time Docker events
```

### Image Commands
```bash
docker images                     # List images (alias: docker image ls)
docker pull <image>:<tag>         # Download from registry
docker build -t <name>:<tag> .   # Build from Dockerfile
docker rmi <image>                # Remove image
docker image prune                # Remove dangling images
docker image prune -a             # Remove ALL unused images
docker tag <src> <dest>           # Create alias for image
docker save -o file.tar <image>   # Export image to tar
docker load -i file.tar           # Import image from tar
```

### Container Commands
```bash
docker run [options] <image>      # Create and start container
docker ps                         # List running containers
docker ps -a                      # List ALL containers
docker stop <container>           # Graceful stop
docker kill <container>           # Force stop
docker start <container>          # Start stopped container
docker restart <container>        # Stop + Start
docker rm <container>             # Remove container
docker rm -f <container>          # Force remove (running too)
docker rename <old> <new>         # Rename container
docker wait <container>           # Block until container stops → exit code
```

### `docker run` Frequently Used Options

| Flag | Purpose | Example |
|---|---|---|
| `-d` | Detached mode (background) | `docker run -d nginx` |
| `-it` | Interactive + TTY (terminal) | `docker run -it ubuntu bash` |
| `--name` | Assign a name | `docker run --name web nginx` |
| `-p` | Publish ports | `docker run -p 8080:80 nginx` |
| `-v` | Mount volume/bind mount | `docker run -v data:/app nginx` |
| `-e` | Set env variable | `docker run -e PORT=3000 myapp` |
| `--env-file` | Load env file | `docker run --env-file .env myapp` |
| `--network` | Connect to network | `docker run --network mynet nginx` |
| `--rm` | Auto-remove on stop | `docker run --rm nginx` |
| `--restart` | Restart policy | `docker run --restart=always nginx` |
| `-w` | Working directory | `docker run -w /app myimage` |
| `--cpus` | CPU limit | `docker run --cpus="1.5" myapp` |
| `--memory` | Memory limit | `docker run --memory="512m" myapp` |

---

## 3.4 Docker Logs

```bash
# View logs of a container
docker logs web

# Follow logs (live stream)
docker logs -f web

# Show last N lines
docker logs --tail 100 web

# Show logs since a timestamp
docker logs --since 2024-01-01T00:00:00 web

# Show timestamps with each line
docker logs -t web

# Combine: last 50 lines + follow + timestamps
docker logs -f --tail 50 -t web
```

---

## 3.5 Docker Inspect

`docker inspect` returns detailed JSON information about any Docker object.

```bash
# Inspect a container
docker inspect web

# Get specific field using Go template
docker inspect --format '{{.State.Status}}' web
docker inspect --format '{{.NetworkSettings.IPAddress}}' web
docker inspect --format '{{.Mounts}}' web

# Inspect an image
docker inspect nginx

# Inspect a volume
docker volume inspect mydata

# Inspect a network
docker network inspect bridge
```

**Useful inspect queries:**
```bash
# Container IP address
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' web

# Container's exposed ports
docker inspect -f '{{.Config.ExposedPorts}}' web

# Environment variables
docker inspect -f '{{.Config.Env}}' web

# Restart count
docker inspect -f '{{.RestartCount}}' web
```

---

## 3.6 Exec vs Attach

| Feature | `docker exec` | `docker attach` |
|---|---|---|
| What it does | Runs a **new** process inside a running container | Connects to the **main** process (PID 1) |
| Use case | Debugging, running one-off commands | Viewing output of the main process |
| Terminal | Gets its own stdin/stdout | Shares main process stdin/stdout |
| Exit behavior | Only the exec process exits | Can stop the container if you Ctrl+C |

```bash
# docker exec — open a new shell
docker exec -it web bash
docker exec -it web sh          # For Alpine images
docker exec web cat /etc/nginx/nginx.conf

# docker attach — attach to main process
docker attach web
# Press Ctrl+P, Ctrl+Q to detach without stopping

# To safely detach from attach:
docker attach --detach-keys="ctrl-d" web
```

**Rule of Thumb:** Almost always use `docker exec`. Use `docker attach` only when you specifically need to interact with the main process.

---

## 3.7 Prune Commands (Cleaning Up)

Docker objects accumulate over time. Prune commands clean up unused resources.

```bash
# Remove stopped containers
docker container prune

# Remove unused images (dangling only)
docker image prune

# Remove ALL unused images (not just dangling)
docker image prune -a

# Remove unused volumes
docker volume prune

# Remove unused networks
docker network prune

# NUCLEAR OPTION: Remove everything unused
docker system prune

# Remove everything INCLUDING volumes
docker system prune -a --volumes

# See what space Docker is using
docker system df
docker system df -v    # Verbose
```

**Example output of `docker system df`:**
```
TYPE            TOTAL    ACTIVE   SIZE      RECLAIMABLE
Images          15       3        4.2GB     3.1GB (73%)
Containers      5        2        120MB     80MB (66%)
Local Volumes   8        3        2.1GB     1.5GB (71%)
Build Cache     0        0        0B        0B
```

**Warning:** `docker volume prune` deletes data permanently. Always verify what will be pruned before confirming.

---

> **Next**: [04-Dockerfile-Deep-Dive.md](./04-Dockerfile-Deep-Dive.md) →
