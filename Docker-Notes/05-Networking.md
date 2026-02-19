# 5. DOCKER NETWORKING

> **Source**: [docs.docker.com/network/](https://docs.docker.com/network/)

---

## 5.1 Network Drivers Overview

```
┌────────────────────────────────────────────────────────────────┐
│                    DOCKER NETWORK DRIVERS                      │
│                                                                │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐      │
│  │  bridge  │  │   host   │  │ overlay  │  │ macvlan  │      │
│  │ (default)│  │          │  │          │  │          │      │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘      │
│       │              │             │              │            │
│  Single host    No network    Multi-host     Physical         │
│  container      isolation     (Swarm)        network          │
│  isolation                                   access           │
└────────────────────────────────────────────────────────────────┘
```

---

## 5.2 Bridge Network (Default)

The default network driver. Containers on the same bridge network can communicate.

```
┌───────────────── Host Machine ─────────────────┐
│                                                 │
│  ┌────────────── docker0 bridge ──────────────┐│
│  │                                            ││
│  │  ┌──────────┐          ┌──────────┐        ││
│  │  │ Container│  ◄────►  │ Container│        ││
│  │  │  web     │  bridge  │  api     │        ││
│  │  │ 172.17.  │  net     │ 172.17.  │        ││
│  │  │ 0.2      │          │ 0.3      │        ││
│  │  └──────────┘          └──────────┘        ││
│  └────────────────────────────────────────────┘│
│                                                 │
│  Internet ◄──── NAT ◄──── docker0              │
└─────────────────────────────────────────────────┘
```

**Default bridge vs Custom bridge:**

| Feature | Default `bridge` | Custom bridge |
|---|---|---|
| DNS Resolution | ❌ By IP only | ✅ By container name |
| Isolation | All containers share it | Only connected containers |
| Auto-connect | All containers by default | Explicit connection |
| On-the-fly connect/disconnect | ❌ | ✅ |

```bash
# Create custom bridge (RECOMMENDED)
docker network create mynet

# Run containers on custom network
docker run -d --name web --network mynet nginx
docker run -d --name api --network mynet myapi

# Now 'web' can reach 'api' by name:
# curl http://api:8080  (from inside 'web')

# Inspect
docker network inspect mynet
```

---

## 5.3 Host Network

Container shares the host's network namespace. **No network isolation**. Container ports are directly on the host.

```bash
docker run -d --network host nginx
# nginx is now on host's port 80 directly — no -p needed
```

| Pros | Cons |
|---|---|
| Best performance (no NAT overhead) | No port isolation |
| Simplest for single-container setups | Port conflicts with host |
| | Linux only (not on Docker Desktop) |

---

## 5.4 Overlay Network

Connects containers across **multiple Docker hosts** (Swarm mode). Used for multi-node deployments.

```bash
# Initialize Swarm (on manager node)
docker swarm init

# Create overlay network
docker network create --driver overlay my-overlay

# Deploy service on overlay
docker service create --name web --network my-overlay nginx
```

```
┌──── Host 1 ───┐      ┌──── Host 2 ───┐
│  ┌──────────┐  │      │  ┌──────────┐ │
│  │ Container│  │      │  │ Container│ │
│  │   web    │  │      │  │   api    │ │
│  └────┬─────┘  │      │  └────┬─────┘ │
│       │        │      │       │       │
│  ┌────┴────────┴──────┴───────┴────┐  │
│  │        Overlay Network          │  │
│  │     (VXLAN tunnel)              │  │
│  └─────────────────────────────────┘  │
└────────────────┘      └───────────────┘
```

---

## 5.5 Macvlan Network

Assigns a **real MAC address** to each container, making it appear as a physical device on the network. Containers get IPs from the physical network.

```bash
docker network create -d macvlan \
  --subnet=192.168.1.0/24 \
  --gateway=192.168.1.1 \
  -o parent=eth0 \
  my-macvlan

docker run -d --network my-macvlan --ip 192.168.1.100 nginx
```

**Use Case:** Legacy applications that need to be directly on the physical network, IoT devices, applications that need to appear as physical hosts.

---

## 5.6 Container-to-Container Communication

### Same Custom Bridge Network
```bash
# Containers can communicate by NAME
docker network create app-net
docker run -d --name db --network app-net postgres
docker run -d --name web --network app-net -e DB_HOST=db myapp
# 'web' connects to 'db' using hostname "db"
```

### Different Networks
```bash
# Containers on different networks CANNOT communicate by default
docker network create frontend
docker network create backend

docker run -d --name web --network frontend nginx
docker run -d --name api --network backend myapi
# web cannot reach api!

# Connect api to both networks
docker network connect frontend api
# Now web CAN reach api
```

### Between Host and Container
```bash
# Use published ports (-p)
docker run -d -p 8080:80 nginx
# Access from host: http://localhost:8080
```

---

## 5.7 How DNS Works Inside Docker

> **Source**: [docs.docker.com/network/#dns-services](https://docs.docker.com/network/)

Docker has a **built-in DNS server** for custom networks (NOT the default bridge).

```
┌──────────────────────────────────────────────┐
│            Custom Bridge Network             │
│                                              │
│  ┌──────────┐         ┌──────────┐           │
│  │  web     │  "db"   │  db      │           │
│  │          ├────────►│          │           │
│  │          │         │          │           │
│  └──────────┘         └──────────┘           │
│         │                                    │
│         ▼                                    │
│  ┌─────────────────────┐                     │
│  │ Docker DNS Server   │                     │
│  │ (127.0.0.11)        │                     │
│  │                     │                     │
│  │ "db" → 172.18.0.3   │                     │
│  │ "web" → 172.18.0.2  │                     │
│  └─────────────────────┘                     │
└──────────────────────────────────────────────┘
```

**Key Points:**
- Docker DNS is at `127.0.0.11` inside containers
- Works **only on custom networks**, not on the default `bridge`
- Resolves container names AND network aliases
- Network aliases allow multiple names for the same container:
  ```bash
  docker run -d --name db --network mynet --network-alias database postgres
  # Both "db" and "database" resolve to this container
  ```

---

> **Next**: [06-Storage.md](./06-Storage.md) →
