# 2. CORE CONCEPTS

> **Source of Truth**: [docs.docker.com/get-started/overview](https://docs.docker.com/get-started/overview/)

---

## 2.1 Images

**Simple Explanation:**
An image is a blueprint/template for creating containers. It contains your app code, libraries, dependencies, and configuration — everything needed to run. Images are **read-only**.

**Technical Definition:**
> A Docker image is a read-only template with instructions for creating a Docker container. Often, an image is based on another image with some additional customization. — *Docker Docs*

**Real-World Analogy:**
An image is like a **recipe**. You can use the same recipe to cook the dish many times (create many containers), but the recipe itself never changes.

### CLI Commands

```bash
# Pull an image from Docker Hub
docker pull nginx
docker pull python:3.11-slim

# List local images
docker images
docker image ls

# Remove an image
docker rmi nginx
docker image rm nginx

# Build an image from Dockerfile
docker build -t myapp:1.0 .

# Inspect an image
docker image inspect nginx

# View image history (layers)
docker image history nginx

# Tag an image
docker tag myapp:1.0 myregistry.com/myapp:1.0

# Push to registry
docker push myregistry.com/myapp:1.0

# Save/Load image as tar
docker save -o myapp.tar myapp:1.0
docker load -i myapp.tar
```

**Common Mistakes:**
- Not specifying a tag → defaults to `:latest`, which can be unpredictable
- Using large base images (e.g., `python:3.11` at 900MB) instead of slim/alpine variants
- Not cleaning up unused images → disk fills up. Use `docker image prune`

---

## 2.2 Containers

**Simple Explanation:**
A container is a **running instance** of an image. If an image is a recipe, a container is the actual dish you cooked from that recipe. You can run multiple containers from the same image.

**Technical Definition:**
> A container is a runnable instance of an image. You can create, start, stop, move, or delete a container using the Docker CLI or API. — *Docker Docs*

**Real-World Analogy:**
Image = **Class** in programming. Container = **Object** (instance of that class).

### CLI Commands

```bash
# Run a container
docker run nginx                  # Foreground
docker run -d nginx               # Detached (background)
docker run -d --name web nginx    # With a name
docker run -it ubuntu bash        # Interactive terminal

# List containers
docker ps                         # Running only
docker ps -a                      # All (including stopped)

# Stop / Start / Restart
docker stop web
docker start web
docker restart web

# Remove a container
docker rm web
docker rm -f web                  # Force (even if running)

# Execute command in running container
docker exec -it web bash
docker exec web ls /etc/nginx

# View logs
docker logs web
docker logs -f web                # Follow (live tail)

# Copy files to/from container
docker cp myfile.txt web:/app/
docker cp web:/app/log.txt ./

# Container stats
docker stats
docker top web
```

**Common Mistakes:**
- Forgetting `-d` flag → container blocks your terminal
- Using `docker run` repeatedly instead of `docker start` on stopped containers → creates duplicates
- Not naming containers → Docker assigns random names, hard to manage
- Storing data inside containers → data is lost when container is removed

---

## 2.3 Dockerfile

**Simple Explanation:**
A Dockerfile is a text file with step-by-step instructions to build an image. It's like a **recipe card** that Docker follows to create your image.

**Technical Definition:**
> A Dockerfile is a text document that contains all the commands a user could call on the command line to assemble an image. — *Docker Docs*

**Real-World Analogy:**
A Dockerfile is like an **IKEA instruction manual** — follow the steps in order and you get the finished product.

### Basic Example

```dockerfile
# Start from a base image
FROM python:3.11-slim

# Set working directory
WORKDIR /app

# Copy dependency file first (for caching)
COPY requirements.txt .

# Install dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Expose port
EXPOSE 5000

# Define startup command
CMD ["python", "app.py"]
```

```bash
# Build image from Dockerfile
docker build -t myapp:1.0 .

# Build with a specific Dockerfile
docker build -f Dockerfile.dev -t myapp:dev .
```

**Common Mistakes:**
- Putting `COPY . .` before `RUN pip install` → breaks build cache on every code change
- Using `RUN apt-get update` and `RUN apt-get install` as separate layers → stale cache
- Not using `.dockerignore` → bloated images with `node_modules`, `.git`, etc.

---

## 2.4 Layers

**Simple Explanation:**
Every instruction in a Dockerfile creates a **layer**. Layers are stacked on top of each other to form the final image. Docker caches layers so rebuilds are fast.

**Technical Definition:**
> Each layer is the result of a Dockerfile instruction. Layers are stacked and each one is a delta of the changes from the previous layer. — *Docker Docs*

**Real-World Analogy:**
Like **layers of a cake** — each instruction adds a new layer. If you change only the frosting (top layer), you don't need to re-bake the whole cake.

```
┌─────────────────────────────┐
│  Layer 5: CMD ["python"...] │  ← Metadata only
├─────────────────────────────┤
│  Layer 4: COPY . .          │  ← Your code
├─────────────────────────────┤
│  Layer 3: RUN pip install   │  ← Dependencies
├─────────────────────────────┤
│  Layer 2: COPY requirements │  ← requirements.txt
├─────────────────────────────┤
│  Layer 1: FROM python:3.11  │  ← Base image
└─────────────────────────────┘
```

**Key Points:**
- Layers are **read-only** and **cached**
- Only changed layers and layers above them are rebuilt
- Order matters: put **least-changing** instructions first
- Combine related `RUN` commands with `&&` to reduce layers

```bash
# View layers of an image
docker image history myapp:1.0
```

---

## 2.5 Volumes

**Simple Explanation:**
Volumes are Docker's way of storing data that persists even after a container is deleted. Think of them as **external USB drives** for containers.

**Technical Definition:**
> Volumes are the preferred mechanism for persisting data generated by and used by Docker containers. Volumes are completely managed by Docker. — *Docker Docs*

**Real-World Analogy:**
A volume is like a **locker at the gym**. You (the container) may come and go, but your stuff in the locker stays.

### CLI Commands

```bash
# Create a volume
docker volume create mydata

# List volumes
docker volume ls

# Inspect a volume
docker volume inspect mydata

# Use a volume in a container
docker run -d -v mydata:/app/data nginx

# Remove a volume
docker volume rm mydata

# Remove all unused volumes
docker volume prune
```

**Common Mistakes:**
- Confusing volumes with bind mounts (see 2.7)
- Not using named volumes → anonymous volumes with random hashes, hard to manage
- Forgetting to back up volumes before removing them

---

## 2.6 Networks

**Simple Explanation:**
Docker networks allow containers to communicate with each other. By default, containers are isolated — networks let you connect specific containers together.

**Technical Definition:**
> Docker networking allows containers to connect to each other and to non-Docker workloads. Docker's networking subsystem is pluggable using drivers. — *Docker Docs*

**Real-World Analogy:**
Networks are like **private LAN cables** — only devices (containers) plugged into the same network can talk to each other.

### CLI Commands

```bash
# List networks
docker network ls

# Create a network
docker network create mynet

# Run container on a network
docker run -d --name web --network mynet nginx

# Connect a running container to a network
docker network connect mynet mycontainer

# Disconnect
docker network disconnect mynet mycontainer

# Inspect a network
docker network inspect mynet

# Remove a network
docker network rm mynet
```

**Common Mistakes:**
- Using the default `bridge` network → no DNS resolution between containers
- Not creating custom networks → containers can't find each other by name
- Forgetting to put dependent containers on the same network

---

## 2.7 Bind Mounts

**Simple Explanation:**
Bind mounts map a **specific folder on your host machine** to a folder inside the container. Changes on either side are reflected immediately. Great for development.

**Technical Definition:**
> Bind mounts have been around since the early days of Docker. Bind mounts have limited functionality compared to volumes. A file or directory on the host machine is mounted into a container. — *Docker Docs*

**Real-World Analogy:**
A bind mount is like **sharing a Google Doc** — changes by anyone appear instantly for everyone.

```bash
# Bind mount (host path : container path)
docker run -d -v /path/on/host:/app nginx

# Windows example
docker run -d -v C:\Users\me\project:/app nginx

# Read-only bind mount
docker run -d -v $(pwd)/config:/app/config:ro nginx
```

### Volumes vs Bind Mounts

| Feature | Volume | Bind Mount |
|---|---|---|
| Managed by Docker | ✅ Yes | ❌ No |
| Location | Docker's storage area | Anywhere on host |
| Portability | High | Tied to host path |
| Best For | Production, data persistence | Development, config files |
| Backup | `docker volume` commands | Regular file backup |

---

## 2.8 Environment Variables

**Simple Explanation:**
Environment variables let you pass configuration to containers at runtime without changing the image. Like **settings knobs** you can adjust per deployment.

**Real-World Analogy:**
Like the **thermostat** on your oven — the recipe (image) is the same, but you adjust the temperature (env vars) based on what you're cooking.

```bash
# Pass single env var
docker run -e MY_VAR=hello nginx

# Pass multiple
docker run -e DB_HOST=db -e DB_PORT=5432 myapp

# From a file
docker run --env-file .env myapp
```

```dockerfile
# Set default in Dockerfile
ENV APP_ENV=production
ENV PORT=8080
```

**Common Mistakes:**
- Hardcoding secrets in Dockerfiles — use `--env-file` or secrets management instead
- Using `ARG` when you need runtime config — `ARG` is build-time only, `ENV` persists at runtime

---

## 2.9 Ports & Exposure

**Simple Explanation:**
Containers are isolated by default. To access a service inside a container from outside, you need to **publish (map) ports**.

**Real-World Analogy:**
A container is a building with closed doors. **Port mapping** is like opening a specific door to the outside world.

```
Host Machine                    Container
┌──────────────┐               ┌──────────────┐
│              │    -p 8080:80 │              │
│  Port 8080   ├──────────────►│  Port 80     │
│              │               │  (nginx)     │
└──────────────┘               └──────────────┘
```

```bash
# Map host port 8080 to container port 80
docker run -d -p 8080:80 nginx

# Map multiple ports
docker run -d -p 8080:80 -p 443:443 nginx

# Map to specific host interface
docker run -d -p 127.0.0.1:8080:80 nginx

# Random host port
docker run -d -P nginx

# View port mappings
docker port mycontainer
```

```dockerfile
# EXPOSE in Dockerfile (documentation only, does NOT publish)
EXPOSE 80
EXPOSE 443
```

**Common Mistakes:**
- Thinking `EXPOSE` in Dockerfile publishes the port — it doesn't! You still need `-p`
- Port conflicts — another process already using the host port
- Mapping to `0.0.0.0` in production — use specific interfaces for security

---

## 2.10 Registry (Docker Hub & Private Registries)

**Simple Explanation:**
A registry is a **storage/distribution system** for Docker images. Docker Hub is the default public registry (like GitHub for images), but you can run your own private registry.

**Technical Definition:**
> A Docker registry stores Docker images. Docker Hub is a public registry anyone can use, and Docker looks for images on Docker Hub by default. — *Docker Docs*

**Real-World Analogy:**
Docker Hub is like the **App Store** for Docker images. You pull images from it, or push your own apps for others to use.

```bash
# Login to Docker Hub
docker login

# Pull from Docker Hub
docker pull nginx
docker pull myuser/myapp:v1

# Push to Docker Hub
docker tag myapp:1.0 myuser/myapp:1.0
docker push myuser/myapp:1.0

# Run a private registry
docker run -d -p 5000:5000 --name registry registry:2

# Push to private registry
docker tag myapp:1.0 localhost:5000/myapp:1.0
docker push localhost:5000/myapp:1.0

# Search Docker Hub
docker search nginx
```

**Common Mistakes:**
- Pulling `:latest` in production — always use specific version tags
- Not scanning images for vulnerabilities before using them
- Pushing sensitive data (secrets, passwords) baked into images

---

> **Next**: [03-Workflow-Commands.md](./03-Workflow-Commands.md) →
