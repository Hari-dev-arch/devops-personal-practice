# 🐳 Docker — Practical DevOps Notes

![Docker](https://img.shields.io/badge/Docker-Practical_Learning-2496ED?logo=docker&logoColor=white)
![Level](https://img.shields.io/badge/Level-Fundamentals-success)
![Focus](https://img.shields.io/badge/Focus-Hands--On-orange)

> [!NOTE]
> My personal Docker reference for hands-on practice, troubleshooting, interviews, and day-to-day DevOps work.

---

## 🧭 Quick Navigation

| Topic | What I Need to Remember |
|---|---|
| 🐳 Containers | Running instance of an image |
| 📦 Images | Packaged application |
| 🔌 Ports | `HOST:CONTAINER` |
| 🌐 Network | Container communication + DNS |
| 💾 Storage | Volume / Bind Mount |
| ⚙️ Dockerfile | Instructions to build image |
| 🌱 ENV / ARG | Runtime / Build-time variables |
| ❤️ Healthcheck | Is application actually healthy? |
| 📊 Resources | CPU / Memory limits |
| 🏗️ Multi-stage | Build big → Run small |
| ☁️ Registry | Tag → Push → Pull |
| 🔧 Troubleshooting | Check → Understand → Fix → Validate |

---

# 🧠 1. Docker Mental Model

```text
Source Code
     │
     ▼
 Dockerfile
     │
     │ docker build
     ▼
   IMAGE
     │
     │ docker run
     ▼
 CONTAINER
     │
     ▼
Application
```

### Simple Definitions

| Component | Meaning |
|---|---|
| **Dockerfile** | Instructions to create an image |
| **Image** | Packaged application/template |
| **Container** | Running instance of an image |
| **Registry** | Remote storage for images |

### Basic Commands

```bash
docker build -t myapp:v1 .
docker run -d --name app01 myapp:v1
docker ps
```

> [!TIP]
> **Dockerfile → Image → Container**  
> This is the first Docker flow to remember.

---

# 📦 2. Images

### List images

```bash
docker images
```

### Pull

```bash
docker pull nginx
docker pull nginx:1.27
```

### Inspect

```bash
docker inspect nginx
```

### Remove

```bash
docker rmi nginx:1.27
```

> [!IMPORTANT]
> **Image = template. Container = running instance.**

---

# 🐳 3. Container Lifecycle

```text
IMAGE
  │
  │ docker run
  ▼
RUNNING
  │
  ├── docker stop ──► STOPPED
  │
  └── docker restart
```

### Commands

```bash
docker run -d --name web01 nginx

docker ps

docker ps -a

docker stop web01

docker start web01

docker restart web01

docker rm web01
```

### Inspect

```bash
docker inspect web01
```

> [!WARNING]
> On company/production systems, don't remove or restart containers before verifying what they are used for.

---

# ⚙️ 4. PID 1 — Why Does a Container Stay Running?

A container normally lives while its **main process (PID 1)** lives.

```text
PID 1 running
      │
      ▼
Container running


PID 1 exits
      │
      ▼
Container stops
```

Check:

```bash
docker top web01
```

Example:

```dockerfile
CMD ["sleep", "3600"]
```

`sleep` becomes the main process, so the container stays alive until it finishes.

---

# 🖥️ 5. Enter a Container

```bash
docker exec -it web01 sh
```

If Bash exists:

```bash
docker exec -it web01 bash
```

Run only one command:

```bash
docker exec web01 whoami
docker exec web01 pwd
```

> [!IMPORTANT]
> HOST and CONTAINER are different environments.

```text
HOST
ubuntu@server

        ↓ docker exec

CONTAINER
root / appuser / nginx
```

---

# 📜 6. Logs

```bash
docker logs web01
```

Follow:

```bash
docker logs -f web01
```

Last 100:

```bash
docker logs --tail 100 web01
```

### Troubleshooting flow

```text
Problem
   ↓
docker ps -a
   ↓
docker logs
   ↓
docker inspect
```

---

# 🔌 7. Ports

Run Nginx:

```bash
docker run -d \
  --name web01 \
  -p 8080:80 \
  nginx
```

### Remember

```text
-p HOST:CONTAINER

8080 : 80
  │     │
  │     └── Application inside container
  │
  └──────── Host/server port
```

Test:

```bash
curl http://localhost:8080
```

> [!IMPORTANT]
> `-p` does not make an application listen on a port.
>
> The application itself must already be listening inside the container.

---

# 🚪 8. EXPOSE vs `-p`

Dockerfile:

```dockerfile
EXPOSE 8080
```

Runtime:

```bash
docker run -p 9090:8080 myapp:v1
```

| Command | Purpose |
|---|---|
| `EXPOSE 8080` | Documents container port |
| `-p 9090:8080` | Actually publishes the port |

```text
Browser
   │
   ▼
HOST :9090
   │
   ▼
CONTAINER :8080
   │
   ▼
Application
```

---

# 🌐 9. Docker Networking

List:

```bash
docker network ls
```

Create:

```bash
docker network create app-network
```

Run containers:

```bash
docker run -d \
  --name frontend \
  --network app-network \
  nginx
```

```bash
docker run -d \
  --name backend \
  --network app-network \
  nginx
```

Inspect:

```bash
docker network inspect app-network
```

### Docker DNS

On a user-defined network:

```text
frontend
    │
    │ Docker DNS
    ▼
backend
```

Application can use:

```text
http://backend:8080
```

instead of depending on changing container IPs.

> [!TIP]
> **User-defined network = container-name DNS.**

---

## Other Network Modes

### Host

```bash
docker run --network host nginx
```

Shares host networking.

### None

```bash
docker run --network none nginx
```

No normal network connectivity.

### Internal network

```bash
docker network create --internal backend-network
```

Useful for isolated backend communication.

---

# 💾 10. Docker Storage

Three important types:

```text
Named Volume
Anonymous Volume
Bind Mount
```

---

## 🟢 Named Volume

```bash
docker volume create company-data
```

```bash
docker run -d \
  --name app01 \
  -v company-data:/data \
  ubuntu sleep 3600
```

Format:

```text
VOLUME_NAME : CONTAINER_PATH

company-data:/data
```

Docker manages the host-side storage.

---

## 🟡 Anonymous Volume

```bash
docker run -d \
  --name app01 \
  -v /data \
  ubuntu sleep 3600
```

Docker generates the volume name.

---

## 🔵 Bind Mount

```bash
docker run -d \
  --name app01 \
  -v /home/ubuntu/app-data:/data \
  ubuntu sleep 3600
```

Format:

```text
HOST_PATH : CONTAINER_PATH
```

Example:

```text
/home/ubuntu/app-data:/data
```

### Read-only

```bash
-v /home/ubuntu/config:/config:ro
```

---

## Volume vs Bind Mount

| Storage | Example | Host location |
|---|---|---|
| Named volume | `app-data:/data` | Docker manages |
| Bind mount | `/opt/data:/data` | We choose |
| Anonymous | `/data` | Docker generates |

> [!TIP]
> **Volume = Docker-managed storage.**  
> **Bind mount = exact host path chosen by us.**

---

# 📊 11. CPU & Memory

Memory:

```bash
docker run -d \
  --name app01 \
  --memory=256m \
  nginx
```

CPU:

```bash
docker run -d \
  --name app01 \
  --cpus="0.5" \
  nginx
```

Both:

```bash
docker run -d \
  --name app01 \
  --memory=256m \
  --cpus="0.5" \
  nginx
```

Monitor:

```bash
docker stats
```

One check:

```bash
docker stats --no-stream
```

---

## 💥 OOMKilled

Check:

```bash
docker inspect -f '{{.State.OOMKilled}}' app01
```

If:

```text
true
```

the container was killed because of an out-of-memory condition.

```text
Container stopped
       ↓
docker inspect
       ↓
OOMKilled = true?
       ↓
Check memory usage + limit
```

---

# 🔄 12. Restart Policy

```bash
docker run -d \
  --name web01 \
  --restart unless-stopped \
  nginx
```

Common policies:

```text
no
always
unless-stopped
on-failure
```

Inspect:

```bash
docker inspect \
  -f '{{.HostConfig.RestartPolicy.Name}}' \
  web01
```

---

# 💽 13. Docker Disk Usage

```bash
docker system df
```

Detailed:

```bash
docker system df -v
```

> [!CAUTION]
> Never see `RECLAIMABLE` and immediately delete data on a production server.
>
> First identify what it is and whether it is being used.

---

# 🐳 14. Dockerfile

Example:

```dockerfile
FROM ubuntu:24.04

WORKDIR /opt/myapp

RUN useradd -m -s /bin/bash appuser

COPY application.jar .

ARG APP_VERSION=1.0

ENV APP_ENV=production

RUN chown -R appuser:appuser /opt/myapp

USER appuser

EXPOSE 8080

CMD ["java", "-jar", "application.jar"]
```

Build:

```bash
docker build -t myapp:v1 .
```

---

# 🧱 15. Dockerfile Instructions — Easy Reference

| Instruction | Remember |
|---|---|
| `FROM` | Base image |
| `WORKDIR` | Working directory |
| `COPY` | Copy files into image |
| `RUN` | Execute during build |
| `ARG` | Build-time variable |
| `ENV` | Runtime/default variable |
| `USER` | Container runtime user |
| `EXPOSE` | Container port metadata |
| `HEALTHCHECK` | Test application health |
| `CMD` | Default runtime command |
| `ENTRYPOINT` | Main executable |

---

# 📂 16. WORKDIR

```dockerfile
WORKDIR /opt/myapp
```

Think:

```text
WORKDIR ≈ cd
```

Then:

```dockerfile
COPY application.jar .
```

means:

```text
/opt/myapp/application.jar
```

---

# 📋 17. COPY

```dockerfile
COPY application.jar /opt/myapp/application.jar
```

`COPY` happens during:

```text
docker build
```

It is **not** a live host/container connection.

For live host sharing use:

```text
Bind Mount
```

---

# 🚫 18. `.dockerignore`

Example:

```text
*.log
.git
secret.txt
node_modules
```

Purpose:

```text
Keep unnecessary files out of build context.
```

---

# 🔨 19. RUN vs CMD

This table is worth memorizing:

| BUILD TIME | RUN TIME |
|---|---|
| `RUN` | `CMD` |
| `ARG` | `ENV` |

Example:

```dockerfile
RUN apt-get update
```

Runs during:

```bash
docker build
```

Example:

```dockerfile
CMD ["java", "-jar", "app.jar"]
```

Runs when the container starts.

> [!TIP]
> **RUN builds the image. CMD runs the application.**

---

# 🌱 20. ARG vs ENV

### ARG

```dockerfile
ARG APP_VERSION=1.0
```

Override:

```bash
docker build \
  --build-arg APP_VERSION=2.0 \
  -t myapp:v2 .
```

Used during **BUILD**.

### ENV

```dockerfile
ENV APP_ENV=development
```

Override:

```bash
docker run \
  -e APP_ENV=production \
  myapp:v1
```

Used/defaulted during **RUNTIME**.

### Memory Trick

```text
BUILD TIME          RUN TIME
----------          --------

ARG                 ENV

RUN                 CMD
```

---

# 👤 21. USER & Permissions

Create user:

```dockerfile
RUN useradd -m -s /bin/bash appuser
```

Prepare permissions:

```dockerfile
RUN chown -R appuser:appuser /opt/myapp
```

Switch:

```dockerfile
USER appuser
```

Check:

```bash
docker exec app01 whoami
```

Expected:

```text
appuser
```

Correct order:

```text
Create
  ↓
Copy
  ↓
chown
  ↓
USER appuser
  ↓
Run application
```

> [!WARNING]
> Don't use `chmod 777` as the automatic solution to permission problems.

---

# ❤️ 22. HEALTHCHECK

Example:

```dockerfile
HEALTHCHECK \
  --interval=10s \
  --timeout=3s \
  --retries=3 \
  CMD curl -f http://localhost:80/ || exit 1
```

Meaning:

```text
Check every 10 sec
       ↓
Allow max 3 sec
       ↓
3 consecutive failures
       ↓
UNHEALTHY
```

Check:

```bash
docker ps
```

Detailed:

```bash
docker inspect \
  -f '{{json .State.Health}}' \
  web01
```

States:

```text
health: starting
       ↓
     healthy

OR

health: starting
       ↓
    unhealthy
```

### Very important

```text
Up ≠ Healthy
```

Example:

```text
Up 10 minutes (unhealthy)
```

means:

```text
Main process = RUNNING
Healthcheck  = FAILING
```

HEALTHCHECK alone does **not automatically restart a standalone Docker container**.

---

## Wrong Healthcheck Example

Application works:

```text
/
```

but Docker checks:

```text
/does-not-exist
```

Result:

```text
HTTP 404
   ↓
curl -f fails
   ↓
FailingStreak = 3
   ↓
unhealthy
```

So:

> [!IMPORTANT]
> A bad healthcheck can mark a perfectly running application as unhealthy.

---

# 🧊 23. Image Immutability

Suppose:

```text
Dockerfile
   ↓
build
   ↓
way-web:v10
   ↓
container-A
```

Now change Dockerfile and build:

```text
Modified Dockerfile
   ↓
build
   ↓
way-web:v11
   ↓
container-B
```

`v10` does NOT change.

`container-A` does NOT change.

Remember:

```text
Changing Dockerfile
        ≠
changing existing image/container
```

You must:

```text
Edit
 ↓
Build new image
 ↓
Create/update container
```

---

# ⚡ 24. Docker Build Cache

Docker reuses unchanged build work.

You may see:

```text
CACHED
```

Simple rule:

```text
Things changing less often
        ↓
put earlier

Things changing frequently
        ↓
put later
```

This helps builds finish faster.

---

# 🧬 25. Image Layers

Inspect:

```bash
docker history myapp:v1
```

Example:

```bash
docker history way-web:v10
```

Simple mental model:

```text
Base Image
    ↓
Install dependency
    ↓
Copy application
    ↓
Configuration
    ↓
Final Image
```

Don't memorize the full `docker history` output.

Remember:

> **Layers/history help show how an image was constructed.**

---

# 🪶 26. Image Optimization

Goal:

```text
Smaller image
Cleaner image
Faster pull
Faster deployment
Less unnecessary content
```

Instead of unnecessary package steps:

```dockerfile
RUN apt-get update
RUN apt-get install -y curl
```

prefer related package work together:

```dockerfile
RUN apt-get update && \
    apt-get install -y curl && \
    rm -rf /var/lib/apt/lists/*
```

Main rules:

```text
✓ Appropriate base image
✓ Only required packages
✓ .dockerignore
✓ Remove package cache
✓ Good cache ordering
✓ Multi-stage builds
```

---

# 🏗️ 27. Multi-Stage Build ⭐

This is one of the most useful production concepts.

### Problem

Suppose Angular/React needs:

```text
Node
npm
source code
build tools
```

to BUILD.

But production only needs:

```text
Nginx
+
built static files
```

### Solution

```text
┌─────────────────────────────┐
│ STAGE 1 — BUILD             │
│                             │
│ Node + npm + source         │
│            ↓                │
│       npm run build         │
│            ↓                │
│          dist/              │
└─────────────┬───────────────┘
              │
              │ copy only output
              ▼
┌─────────────────────────────┐
│ STAGE 2 — RUNTIME           │
│                             │
│ Nginx                       │
│   +                         │
│ dist/                       │
│                             │
│ Production Image            │
└─────────────────────────────┘
```

### Dockerfile

```dockerfile
# =========================
# Stage 1 - BUILD
# =========================

FROM node:22 AS builder

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

RUN npm run build


# =========================
# Stage 2 - RUNTIME
# =========================

FROM nginx:latest

COPY --from=builder \
     /app/dist \
     /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

### Most important line

```dockerfile
COPY --from=builder /app/dist /usr/share/nginx/html
```

Meaning:

```text
builder stage
/app/dist
     │
     │ COPY
     ▼
final Nginx image
/usr/share/nginx/html
```

### Why?

Without multi-stage:

```text
Node + npm + source + build tools + application
```

With multi-stage:

```text
Nginx + final application files
```

> [!TIP]
> **Multi-stage = BUILD big → RUN small.**

---

# ☕ 28. Java Multi-Stage Example

```dockerfile
# BUILD
FROM maven:3.9-eclipse-temurin-21 AS builder

WORKDIR /app

COPY . .

RUN mvn clean package -DskipTests


# RUNTIME
FROM eclipse-temurin:21-jre

WORKDIR /app

COPY --from=builder \
     /app/target/app.jar \
     app.jar

EXPOSE 8080

CMD ["java", "-jar", "app.jar"]
```

Mental model:

```text
Maven + Source
      ↓
   app.jar
      ↓
Java Runtime + app.jar
```

Maven does not need to be in the final runtime image.

---

# ➕ 29. ADD vs COPY

Normally use:

```dockerfile
COPY
```

Example:

```dockerfile
COPY app.jar /opt/app/app.jar
```

Simple rule:

```text
Normal file copying
       ↓
      COPY

Need a specific ADD feature
       ↓
      ADD
```

---

# 🏷️ 30. Image Tags

```text
way-web:v1
way-web:v2
way-web:v10
```

Format:

```text
IMAGE:TAG
```

Build:

```bash
docker build -t way-web:v10 .
```

Explicit versions make deployments and rollback easier to understand.

---

# ☁️ 31. Docker Hub / Registry

Full flow:

```text
Dockerfile
    ↓
docker build
    ↓
Local Image
    ↓
docker tag
    ↓
docker push
    ↓
DOCKER HUB
    ↓
docker pull
    ↓
Another Server
    ↓
docker run
```

Login:

```bash
docker login
```

Check:

```bash
docker info | grep -i username
```

> [!CAUTION]
> Never commit Docker passwords/tokens to GitHub.

---

## Tag

General:

```bash
docker tag SOURCE TARGET
```

Example:

```bash
docker tag \
  way-web:v10 \
  username/way-web:v10
```

Verify:

```bash
docker images username/way-web
```

---

## Push

```bash
docker push username/way-web:v10
```

---

## Pull

On another server:

```bash
docker pull username/way-web:v10
```

Run:

```bash
docker run -d \
  --name web01 \
  -p 8080:80 \
  username/way-web:v10
```

---

# 🚀 32. Real CI/CD Image Flow

```text
Developer
    ↓
Git Repository
    ↓
Jenkins
    ↓
docker build
    ↓
docker tag
    ↓
docker push
    ↓
Registry
    ↓
Deployment
    ↓
Container / Swarm / Kubernetes
```

---

# 🔧 33. Troubleshooting ⭐

Use this method:

```text
SCENARIO
    ↓
HYPOTHESIS
    ↓
READ-ONLY CHECK
    ↓
CONFIRM ROOT CAUSE
    ↓
FIX
    ↓
VALIDATE
```

When someone says:

> Application is not working.

Start with:

```bash
docker ps
docker ps -a
docker logs --tail 100 container-name
docker inspect container-name
docker stats --no-stream
docker top container-name
```

Then ask:

| Problem | Check |
|---|---|
| Container exited | Logs / CMD |
| Restarting | Logs / application crash |
| OOM | `.State.OOMKilled` |
| Port issue | `docker ps`, listener |
| Permission | USER / ownership |
| Network | Docker network / DNS |
| Data missing | Volume / bind mount |
| Unhealthy | Healthcheck output |
| Wrong version | Image/tag |
| Wrong config | ENV |

---

# 🚨 34. Common Errors

## Container Name Already Exists

Check first:

```bash
docker ps -a --filter name=web01
```

Don't immediately delete it.

---

## Port Already Allocated

Check:

```bash
docker ps
```

Look for:

```text
0.0.0.0:8080->80/tcp
```

Either use another port or, after verification, stop the old workload if appropriate.

---

## Permission Denied

Check:

```bash
docker exec app01 whoami
docker exec app01 ls -ld /opt/myapp
```

Possible fix in Dockerfile:

```dockerfile
RUN chown -R appuser:appuser /opt/myapp
USER appuser
```

---

## Container Is Running but Website Doesn't Open

Check:

```bash
docker ps
docker logs app01
docker inspect app01
```

Remember:

```text
-p 9090:8080
```

doesn't mean something is actually listening on container port `8080`.

---

# 🔍 35. Useful Inspect Commands

### Health

```bash
docker inspect \
  -f '{{json .State.Health}}' \
  web01
```

### OOM

```bash
docker inspect \
  -f '{{.State.OOMKilled}}' \
  web01
```

### Restart Policy

```bash
docker inspect \
  -f '{{.HostConfig.RestartPolicy.Name}}' \
  web01
```

### CMD

```bash
docker inspect \
  -f '{{json .Config.Cmd}}' \
  web01
```

### ENTRYPOINT

```bash
docker inspect \
  -f '{{json .Config.Entrypoint}}' \
  web01
```

### IP

```bash
docker inspect \
  -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' \
  web01
```

---

# 🧰 36. Daily Command Cheat Sheet

```bash
# Containers
docker ps
docker ps -a

# Images
docker images

# Lifecycle
docker start container
docker stop container
docker restart container

# Logs
docker logs --tail 100 container
docker logs -f container

# Shell
docker exec -it container sh

# Processes
docker top container

# Inspect
docker inspect container

# Resources
docker stats

# Networks
docker network ls
docker network inspect network

# Volumes
docker volume ls
docker volume inspect volume

# Disk
docker system df
docker system df -v

# History
docker history image:tag

# Build
docker build -t image:v1 .

# Run
docker run -d --name app01 image:v1

# Port
docker run -d \
  --name app01 \
  -p 8080:80 \
  image:v1

# ENV
docker run -d \
  -e APP_ENV=production \
  image:v1

# Volume
docker run -d \
  -v app-data:/data \
  image:v1

# Bind Mount
docker run -d \
  -v /host/data:/data \
  image:v1

# Network
docker run -d \
  --network app-network \
  image:v1

# Resources
docker run -d \
  --memory=512m \
  --cpus="1.0" \
  image:v1

# Registry
docker tag image:v1 username/image:v1
docker push username/image:v1
docker pull username/image:v1
```

---

# 🎯 37. Final Memory Sheet

```text
┌──────────────────────────────────────────────┐
│              DOCKER CORE FLOW                │
├──────────────────────────────────────────────┤
│                                              │
│ Dockerfile                                   │
│     ↓                                        │
│ docker build                                 │
│     ↓                                        │
│ Image                                        │
│     ↓                                        │
│ docker run                                   │
│     ↓                                        │
│ Container                                    │
│                                              │
└──────────────────────────────────────────────┘
```

### Build vs Runtime

```text
BUILD TIME              RUN TIME
----------              --------

ARG                     ENV

RUN                     CMD
```

### Ports

```text
EXPOSE 8080
     =
container port metadata


-p 9090:8080
     =
HOST 9090 → CONTAINER 8080
```

### Storage

```text
Named Volume
app-data:/data

Bind Mount
/host/data:/data
```

### Network

```text
User-defined network
        ↓
Container-name DNS
```

### Health

```text
Up ≠ Healthy

healthy   → healthcheck passing
unhealthy → healthcheck failing
```

### Multi-stage

```text
BUILD BIG
   ↓
artifact
   ↓
RUN SMALL
```

### Registry

```text
BUILD
  ↓
TAG
  ↓
PUSH
  ↓
REGISTRY
  ↓
PULL
  ↓
RUN
```

---

# 🛡️ 38. Production Checklist

Before deployment:

- [ ] Correct image/tag?
- [ ] Correct container port?
- [ ] Correct host/service port?
- [ ] Environment variables correct?
- [ ] Secrets handled safely?
- [ ] Correct network?
- [ ] Persistent data required?
- [ ] Volume/bind mount correct?
- [ ] CPU/memory limits?
- [ ] Non-root user where possible?
- [ ] Healthcheck valid?
- [ ] Logs available?
- [ ] Registry image available?
- [ ] Rollback version available?

---

# 🏁 Learning Path

```text
Docker Fundamentals
        ↓
Docker Practical Refresh
        ↓
Docker Troubleshooting
        ↓
Docker Compose
        ↓
Docker Swarm
        ↓
Kubernetes
        ↓
AWS
        ↓
Terraform
```

---

## ⭐ Golden Rule

> [!IMPORTANT]
> Don't memorize every Docker command.
>
> Learn to answer:
>
> **What is running?**  
> **Which image?**  
> **Which process?**  
> **Which port?**  
> **Which network?**  
> **Where is the data?**  
> **Which environment?**  
> **Is it healthy?**  
> **What do the logs say?**  
> **What changed?**

### DevOps troubleshooting mindset

```text
CHECK → UNDERSTAND → FIX → VALIDATE
```

---

**Next:** Docker full practical refresh → Docker Compose → Docker Swarm 🚀
