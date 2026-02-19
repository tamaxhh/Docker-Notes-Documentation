# 1. FOUNDATION LEVEL — Docker from Zero

> **Source of Truth**: [docs.docker.com/get-started/overview](https://docs.docker.com/get-started/overview/)

---

## 1.1 What is Docker?

**Simple Explanation:**
Docker is a tool that lets you package your application along with everything it needs (code, libraries, settings) into a single portable unit called a **container**. That container runs the same way on any machine — your laptop, your colleague's laptop, or a production server.

**Technical Definition (Official Docs):**
> Docker is an open platform for developing, shipping, and running applications. Docker enables you to separate your applications from your infrastructure so you can deliver software quickly.

**Real-World Analogy:**
Think of Docker like a **shipping container** in global trade. Before shipping containers, goods were loaded loosely onto ships — fragile, inconsistent, slow. Shipping containers standardized everything: any crane can lift them, any truck can carry them, any ship can hold them. Docker does the same for software.

---

## 1.2 Why Docker Exists (The Problem It Solves)

### The "Works on My Machine" Problem

```
Developer A: "The app works perfectly on my machine!"
Developer B: "It crashes on mine..."
Ops Team:    "It doesn't even start in production..."
```

**Root Causes:**
- Different OS versions
- Different library versions
- Missing dependencies
- Different environment variables
- Different file paths

**Docker's Solution:**
Package the application AND its entire environment into a container. The container is identical everywhere.

### Before Docker vs After Docker

```
┌─────────────────────────────────────────────────────────┐
│                    BEFORE DOCKER                        │
│                                                         │
│  Developer's     Staging         Production             │
│  Machine         Server          Server                 │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐             │
│  │ App v1  │    │ App v1  │    │ App v1  │             │
│  │ Python  │    │ Python  │    │ Python  │             │
│  │ 3.9     │    │ 3.8     │    │ 3.7     │  ← MISMATCH│
│  │ Ubuntu  │    │ CentOS  │    │ Amazon  │  ← MISMATCH│
│  │ 20.04   │    │ 7       │    │ Linux 2 │             │
│  └─────────┘    └─────────┘    └─────────┘             │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│                    AFTER DOCKER                         │
│                                                         │
│  Developer's     Staging         Production             │
│  Machine         Server          Server                 │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐             │
│  │ Container│    │ Container│    │ Container│            │
│  │ App v1  │    │ App v1  │    │ App v1  │  ← SAME    │
│  │ Python  │    │ Python  │    │ Python  │  ← SAME    │
│  │ 3.9     │    │ 3.9     │    │ 3.9     │             │
│  │ Ubuntu  │    │ Ubuntu  │    │ Ubuntu  │  ← SAME    │
│  └─────────┘    └─────────┘    └─────────┘             │
└─────────────────────────────────────────────────────────┘
```

---

## 1.3 Containers vs Virtual Machines

```
┌────────────────────────────────────────────────────────────────────┐
│           VIRTUAL MACHINES              CONTAINERS                │
│                                                                    │
│  ┌───────┐ ┌───────┐ ┌───────┐   ┌───────┐ ┌───────┐ ┌───────┐  │
│  │ App A │ │ App B │ │ App C │   │ App A │ │ App B │ │ App C │  │
│  ├───────┤ ├───────┤ ├───────┤   ├───────┤ ├───────┤ ├───────┤  │
│  │ Bins/ │ │ Bins/ │ │ Bins/ │   │ Bins/ │ │ Bins/ │ │ Bins/ │  │
│  │ Libs  │ │ Libs  │ │ Libs  │   │ Libs  │ │ Libs  │ │ Libs  │  │
│  ├───────┤ ├───────┤ ├───────┤   └───┬───┘ └───┬───┘ └───┬───┘  │
│  │Guest  │ │Guest  │ │Guest  │       │         │         │       │
│  │  OS   │ │  OS   │ │  OS   │   ┌───┴─────────┴─────────┴───┐  │
│  └───┬───┘ └───┬───┘ └───┬───┘   │      Docker Engine        │  │
│  ┌───┴─────────┴─────────┴───┐   ├───────────────────────────┤  │
│  │       Hypervisor          │   │        Host OS             │  │
│  ├───────────────────────────┤   ├───────────────────────────┤  │
│  │       Host OS             │   │      Infrastructure       │  │
│  ├───────────────────────────┤   └───────────────────────────┘  │
│  │     Infrastructure        │                                   │
│  └───────────────────────────┘                                   │
└────────────────────────────────────────────────────────────────────┘
```

| Feature | Virtual Machine | Container |
|---|---|---|
| **OS** | Each VM has its own full OS | Shares host OS kernel |
| **Size** | Gigabytes | Megabytes |
| **Startup** | Minutes | Seconds |
| **Performance** | Near-native with overhead | Near-native (minimal overhead) |
| **Isolation** | Strong (hardware-level) | Process-level (via namespaces) |
| **Resource Usage** | Heavy | Lightweight |
| **Portability** | Less portable | Highly portable |
| **Use Case** | Full OS isolation needed | Microservices, CI/CD, scaling |

**Analogy:**
- **VM** = Each app gets its own **house** (full OS, full resources).
- **Container** = Each app gets its own **apartment** in a shared building (shared OS kernel, isolated space).

---

## 1.4 Core Docker Architecture

> **Source**: [docs.docker.com/get-started/overview/#docker-architecture](https://docs.docker.com/get-started/overview/#docker-architecture)

```
┌──────────────────────────────────────────────────────────┐
│                    DOCKER ARCHITECTURE                    │
│                                                          │
│  ┌────────────┐         ┌──────────────────────────────┐│
│  │   Docker   │         │        Docker Host           ││
│  │   Client   │  REST   │  ┌────────────────────────┐  ││
│  │            │  API    │  │    Docker Daemon        │  ││
│  │ docker run ├────────►│  │      (dockerd)         │  ││
│  │ docker pull│         │  │                        │  ││
│  │ docker build         │  │  ┌──────┐  ┌──────┐   │  ││
│  │            │         │  │  │ Img1 │  │ Img2 │   │  ││
│  └────────────┘         │  │  └──────┘  └──────┘   │  ││
│                         │  │  ┌──────┐  ┌──────┐   │  ││
│                         │  │  │Cont1 │  │Cont2 │   │  ││
│                         │  │  └──────┘  └──────┘   │  ││
│                         │  └────────────────────────┘  ││
│                         └──────────────────────────────┘│
│                                      │                   │
│                                      ▼                   │
│                         ┌──────────────────────────────┐│
│                         │       Docker Registry        ││
│                         │      (Docker Hub, etc.)      ││
│                         └──────────────────────────────┘│
└──────────────────────────────────────────────────────────┘
```

### Components:

| Component | Role |
|---|---|
| **Docker Client** | CLI tool (`docker` command). Sends commands to the daemon. |
| **Docker Daemon** (`dockerd`) | Background service that manages images, containers, networks, volumes. |
| **Docker Engine** | The client + daemon + REST API together. |
| **Docker Registry** | Stores Docker images. Docker Hub is the default public registry. |
| **Docker Desktop** | GUI application for Mac/Windows that bundles Engine + CLI + Compose + Kubernetes. |

---

## 1.5 Docker Installation Basics

> **Source**: [docs.docker.com/get-docker/](https://docs.docker.com/get-docker/)

### Windows

```powershell
# 1. Download Docker Desktop from https://docs.docker.com/desktop/install/windows-install/
# 2. Enable WSL 2 (Windows Subsystem for Linux)
wsl --install
# 3. Run Docker Desktop installer
# 4. Verify installation:
docker --version
docker run hello-world
```

**Requirements:** Windows 10/11 64-bit, WSL 2 enabled, BIOS virtualization enabled.

### macOS

```bash
# 1. Download Docker Desktop from https://docs.docker.com/desktop/install/mac-install/
# 2. Drag Docker.app to Applications
# 3. Open Docker Desktop
# 4. Verify:
docker --version
docker run hello-world
```

### Linux (Ubuntu/Debian)

```bash
# Remove old versions
sudo apt-get remove docker docker-engine docker.io containerd runc

# Set up repository
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg

# Add Docker's official GPG key
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
  sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Set up the repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io \
  docker-buildx-plugin docker-compose-plugin

# Verify
sudo docker run hello-world

# (Optional) Run Docker without sudo
sudo usermod -aG docker $USER
# Log out and back in for this to take effect
```

### Post-Installation Verification

```bash
docker --version          # Check Docker version
docker compose version    # Check Compose version
docker info               # Detailed system info
docker run hello-world    # Run test container
```

---

> **Next**: [02-Core-Concepts.md](./02-Core-Concepts.md) →
