# 6. DOCKER STORAGE

> **Source**: [docs.docker.com/storage/](https://docs.docker.com/storage/)

---

## 6.1 Storage Overview

```
┌──────────────────────────────────────────────────────────────┐
│                   DOCKER STORAGE OPTIONS                     │
│                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │
│  │   Volumes    │  │ Bind Mounts  │  │   tmpfs      │       │
│  │              │  │              │  │   Mounts     │       │
│  │ Managed by   │  │ Host file    │  │ In-memory    │       │
│  │ Docker       │  │ system path  │  │ only (Linux) │       │
│  │              │  │              │  │              │       │
│  │ Best for:    │  │ Best for:    │  │ Best for:    │       │
│  │ Production   │  │ Development  │  │ Sensitive    │       │
│  │ Persistence  │  │ Source code  │  │ temp data    │       │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘       │
│         │                 │                 │               │
│         ▼                 ▼                 ▼               │
│  ┌──────────────────────────────────────────────────────┐   │
│  │                   Container Layer                    │   │
│  │              (read-write, ephemeral)                 │   │
│  └──────────────────────────────────────────────────────┘   │
│  ┌──────────────────────────────────────────────────────┐   │
│  │                   Image Layers                       │   │
│  │                  (read-only)                         │   │
│  └──────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────┘
```

---

## 6.2 Volumes vs Bind Mounts (Detailed)

| Feature | Volumes | Bind Mounts |
|---|---|---|
| **Managed by** | Docker | User/OS |
| **Location on host** | `/var/lib/docker/volumes/` | Any path you specify |
| **Create** | `docker volume create` | Specify path in `-v` |
| **Portability** | ✅ Works on any Docker host | ❌ Tied to host structure |
| **Pre-populated** | ✅ Can populate from container | ❌ Overwrites container dir |
| **Backup** | Via Docker commands | Regular filesystem backup |
| **Performance** | Optimized by Docker | Depends on host filesystem |
| **Security** | Docker manages permissions | Host permissions apply |
| **Best for** | Databases, persistent data | Dev (live code reload) |

---

## 6.3 Named Volumes

```bash
# Create
docker volume create pgdata

# Use in container
docker run -d \
  --name postgres \
  -v pgdata:/var/lib/postgresql/data \
  -e POSTGRES_PASSWORD=secret \
  postgres:16

# Data persists even after removing the container
docker rm -f postgres
# pgdata still exists!
docker volume ls

# Re-create container with same volume
docker run -d \
  --name postgres-new \
  -v pgdata:/var/lib/postgresql/data \
  -e POSTGRES_PASSWORD=secret \
  postgres:16
# All data is intact!

# Inspect volume
docker volume inspect pgdata

# Backup a volume
docker run --rm \
  -v pgdata:/data \
  -v $(pwd):/backup \
  alpine tar czf /backup/pgdata-backup.tar.gz -C /data .

# Restore a volume
docker run --rm \
  -v pgdata:/data \
  -v $(pwd):/backup \
  alpine tar xzf /backup/pgdata-backup.tar.gz -C /data
```

---

## 6.4 Bind Mounts in Practice

```bash
# Development: mount source code for live reload
docker run -d \
  --name dev-server \
  -v $(pwd)/src:/app/src \
  -p 3000:3000 \
  node-dev-server

# Mount config file (read-only)
docker run -d \
  --name web \
  -v $(pwd)/nginx.conf:/etc/nginx/nginx.conf:ro \
  nginx

# Mount options with --mount (more explicit)
docker run -d \
  --name web \
  --mount type=bind,source=$(pwd)/html,target=/usr/share/nginx/html,readonly \
  nginx
```

**`-v` vs `--mount` syntax:**
```bash
# -v (short form) — creates host dir if it doesn't exist
-v /host/path:/container/path:ro

# --mount (long form) — errors if host dir doesn't exist (safer)
--mount type=bind,source=/host/path,target=/container/path,readonly
```

**Best Practice:** Use `--mount` in scripts and production for explicitness. Use `-v` for quick CLI use.

---

## 6.5 Volume Drivers

Volume drivers let you store data on remote hosts or cloud providers instead of the local filesystem.

```bash
# Use a volume driver (example: local with NFS)
docker volume create \
  --driver local \
  --opt type=nfs \
  --opt o=addr=192.168.1.100,rw \
  --opt device=:/path/to/dir \
  nfs-volume

# Use with container
docker run -d -v nfs-volume:/data myapp
```

**Common drivers:**
| Driver | Storage |
|---|---|
| `local` | Local filesystem (default) |
| `nfs` | Network File System |
| Various cloud plugins | AWS EBS, Azure Files, GCP PD |

---

## 6.6 Data Persistence Best Practices

1. **Always use named volumes for databases** — never anonymous volumes
   ```bash
   # ✅ Named volume
   docker run -v pgdata:/var/lib/postgresql/data postgres
   
   # ❌ Anonymous volume (hard to identify later)
   docker run -v /var/lib/postgresql/data postgres
   ```

2. **Backup volumes regularly**
   ```bash
   docker run --rm -v mydata:/source -v $(pwd):/backup \
     alpine tar czf /backup/mydata.tar.gz -C /source .
   ```

3. **Use read-only mounts where possible**
   ```bash
   docker run -v config:/etc/app/config:ro myapp
   ```

4. **Don't store data in the container's writable layer** — it's lost on `docker rm`

5. **Use `.dockerignore`** to avoid copying data directories into images

6. **For development:** Use bind mounts for source code + named volumes for dependencies
   ```bash
   docker run \
     -v $(pwd)/src:/app/src \       # Bind mount: source code (editable)
     -v node_modules:/app/node_modules \  # Named volume: dependencies (cached)
     myapp
   ```

---

> **Next**: [07-Docker-Compose.md](./07-Docker-Compose.md) →
