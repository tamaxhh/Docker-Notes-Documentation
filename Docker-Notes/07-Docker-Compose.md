# 7. DOCKER COMPOSE

> **Source**: [docs.docker.com/compose/](https://docs.docker.com/compose/)

---

## 7.1 What Problem Does Compose Solve?

Without Compose, running a multi-container app requires multiple long `docker run` commands:

```bash
# WITHOUT Compose — painful manual process
docker network create app-net
docker volume create pgdata
docker run -d --name db --network app-net -v pgdata:/var/lib/postgresql/data \
  -e POSTGRES_PASSWORD=secret postgres:16
docker run -d --name redis --network app-net redis:7
docker run -d --name web --network app-net -p 8080:80 \
  -e DB_HOST=db -e REDIS_HOST=redis myapp
```

**Docker Compose** lets you define all of this in **one YAML file** and manage it with one command.

```bash
# WITH Compose — one command
docker compose up -d
docker compose down
```

---

## 7.2 YAML Structure Breakdown

```yaml
# compose.yaml (or docker-compose.yml)

services:       # Containers to run
  web:
    image: nginx
    ports:
      - "8080:80"
    volumes:
      - ./html:/usr/share/nginx/html
    depends_on:
      - api
    networks:
      - frontend

  api:
    build: ./api              # Build from Dockerfile
    environment:
      - DB_HOST=db
      - DB_PORT=5432
    depends_on:
      - db
    networks:
      - frontend
      - backend

  db:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: myapp
    volumes:
      - pgdata:/var/lib/postgresql/data
    networks:
      - backend

volumes:        # Named volumes
  pgdata:

networks:       # Custom networks
  frontend:
  backend:
```

### Top-Level Keys

| Key | Purpose |
|---|---|
| `services` | Define containers (name, image, config) |
| `volumes` | Declare named volumes |
| `networks` | Define custom networks |
| `configs` | Mount config files (Swarm) |
| `secrets` | Manage sensitive data (Swarm) |

---

## 7.3 Service Configuration Options

```yaml
services:
  myservice:
    # --- Image / Build ---
    image: nginx:1.25                    # Use existing image
    build:                               # OR build from Dockerfile
      context: ./app
      dockerfile: Dockerfile.prod
      args:
        BUILD_ENV: production

    # --- Networking ---
    ports:
      - "8080:80"                        # host:container
      - "443:443"
    expose:
      - "3000"                           # Internal only (to other services)
    networks:
      - mynet

    # --- Storage ---
    volumes:
      - pgdata:/var/lib/postgresql/data  # Named volume
      - ./src:/app/src                   # Bind mount
      - ./config:/app/config:ro          # Read-only

    # --- Environment ---
    environment:
      DB_HOST: postgres
      DB_PORT: "5432"
    env_file:
      - .env

    # --- Dependencies ---
    depends_on:
      db:
        condition: service_healthy       # Wait for healthcheck
      redis:
        condition: service_started

    # --- Resource Limits ---
    deploy:
      resources:
        limits:
          cpus: "1.0"
          memory: 512M
        reservations:
          cpus: "0.5"
          memory: 256M

    # --- Restart Policy ---
    restart: unless-stopped              # always | on-failure | no

    # --- Health Check ---
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:80"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 10s

    # --- Misc ---
    container_name: my-web               # Fixed name (not recommended)
    working_dir: /app
    command: ["python", "app.py"]
    entrypoint: ["/entrypoint.sh"]
    user: "1000:1000"
    stdin_open: true                     # -i
    tty: true                            # -t
```

---

## 7.4 Compose CLI Commands

```bash
# Start all services (detached)
docker compose up -d

# Start and rebuild images
docker compose up -d --build

# Stop and remove containers, networks
docker compose down

# Stop and remove EVERYTHING including volumes
docker compose down -v

# View running services
docker compose ps

# View logs
docker compose logs
docker compose logs -f web       # Follow specific service

# Scale a service
docker compose up -d --scale web=3

# Execute command in a service
docker compose exec web bash

# Run a one-off command
docker compose run --rm web python manage.py migrate

# Restart a service
docker compose restart web

# Pull latest images
docker compose pull

# View the resolved config
docker compose config
```

---

## 7.5 Example: Full-Stack Web Application

```yaml
# compose.yaml — Blog Application
services:
  # Frontend
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    environment:
      - API_URL=http://api:8000
    depends_on:
      - api
    networks:
      - frontend-net

  # Backend API
  api:
    build: ./backend
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/blog
      - REDIS_URL=redis://cache:6379
    depends_on:
      db:
        condition: service_healthy
      cache:
        condition: service_started
    volumes:
      - uploads:/app/uploads
    networks:
      - frontend-net
      - backend-net

  # Database
  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: blog
    volumes:
      - pgdata:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user -d blog"]
      interval: 5s
      timeout: 5s
      retries: 5
    networks:
      - backend-net

  # Cache
  cache:
    image: redis:7-alpine
    volumes:
      - redis-data:/data
    networks:
      - backend-net

  # Admin tool
  adminer:
    image: adminer
    ports:
      - "8888:8080"
    depends_on:
      - db
    networks:
      - backend-net

volumes:
  pgdata:
  redis-data:
  uploads:

networks:
  frontend-net:
  backend-net:
```

```bash
# Run the entire stack
docker compose up -d

# Check status
docker compose ps

# View API logs
docker compose logs -f api

# Shut down everything
docker compose down -v
```

---

## 7.6 Compose V2 Concepts

> Docker Compose V2 is the current version (plugin-based, `docker compose` without hyphen).

| Feature | Compose V1 (Legacy) | Compose V2 (Current) |
|---|---|---|
| Command | `docker-compose` | `docker compose` |
| Written in | Python | Go |
| Installed as | Standalone binary | Docker CLI plugin |
| File names | `docker-compose.yml` | `compose.yaml` (preferred) |
| Performance | Slower | Faster |

**V2 Additions:**
- `depends_on` with `condition: service_healthy`
- `profiles` — group services that only start on demand
- Better `watch` support for live development

```yaml
# Profiles example
services:
  web:
    image: nginx
  
  debug:
    image: busybox
    profiles:
      - debug         # Only starts with --profile debug
```

```bash
docker compose up -d                    # Starts web only
docker compose --profile debug up -d    # Starts web + debug
```

---

> **Next**: [08-Security.md](./08-Security.md) →
