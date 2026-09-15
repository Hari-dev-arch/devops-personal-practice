<div align="center">

# 🐳 Docker Command Cheat Sheet

![Docker](https://img.shields.io/badge/Docker-Command%20Cheat%20Sheet-2496ED?logo=docker&logoColor=white)
![DevOps](https://img.shields.io/badge/DevOps-Quick%20Reference-success)
![Linux](https://img.shields.io/badge/Platform-Linux-orange)

### ⚡ Quick • Practical • Topic-wise

</div>

---

# 🐳 1. Containers

### List Running Containers

```bash
docker ps
```

### List All Containers

```bash
docker ps -a
```

### Run Container

```bash
docker run image:tag
```

### Run in Background

```bash
docker run -d image:tag
```

### Run With Name

```bash
docker run -d --name app01 image:tag
```

### Run With Port

```bash
docker run -d \
  --name app01 \
  -p 9090:8080 \
  image:tag
```

### Run Interactive

```bash
docker run -it ubuntu bash
```

### Automatically Remove After Exit

```bash
docker run --rm image:tag
```

### Start Container

```bash
docker start app01
```

### Stop Container

```bash
docker stop app01
```

### Restart Container

```bash
docker restart app01
```

### Pause / Unpause

```bash
docker pause app01
docker unpause app01
```

### Kill Container

```bash
docker kill app01
```

### Rename Container

```bash
docker rename app01 app02
```

### Remove Container

```bash
docker rm app01
```

### Force Remove

```bash
docker rm -f app01
```

### Remove All Stopped Containers

```bash
docker container prune
```

---

# 📜 2. Logs

### View Logs

```bash
docker logs app01
```

### Last 100 Lines

```bash
docker logs --tail 100 app01
```

### Follow Logs

```bash
docker logs -f app01
```

### Logs With Timestamp

```bash
docker logs -t app01
```

### Last 10 Minutes

```bash
docker logs --since 10m app01
```

### Last 1 Hour

```bash
docker logs --since 1h app01
```

### Logs Since Specific Time

```bash
docker logs --since "2026-09-15T10:00:00" app01
```

---

# 🖥️ 3. Exec / Container Access

### Open Shell

```bash
docker exec -it app01 sh
```

### Open Bash

```bash
docker exec -it app01 bash
```

### Check User

```bash
docker exec app01 whoami
```

### Check Working Directory

```bash
docker exec app01 pwd
```

### List Files

```bash
docker exec app01 ls -lah
```

### Check Environment

```bash
docker exec app01 env
```

### Run Command as Root

```bash
docker exec -u root -it app01 sh
```

---

# 🔍 4. Container Inspect

### Full Inspect

```bash
docker inspect app01
```

### Container Status

```bash
docker inspect \
  -f '{{.State.Status}}' \
  app01
```

### Running?

```bash
docker inspect \
  -f '{{.State.Running}}' \
  app01
```

### Exit Code

```bash
docker inspect \
  -f '{{.State.ExitCode}}' \
  app01
```

### OOMKilled

```bash
docker inspect \
  -f '{{.State.OOMKilled}}' \
  app01
```

### PID

```bash
docker inspect \
  -f '{{.State.Pid}}' \
  app01
```

### Started Time

```bash
docker inspect \
  -f '{{.State.StartedAt}}' \
  app01
```

### Finished Time

```bash
docker inspect \
  -f '{{.State.FinishedAt}}' \
  app01
```

### Health

```bash
docker inspect \
  -f '{{json .State.Health}}' \
  app01
```

### Image

```bash
docker inspect \
  -f '{{.Config.Image}}' \
  app01
```

### CMD

```bash
docker inspect \
  -f '{{json .Config.Cmd}}' \
  app01
```

### ENTRYPOINT

```bash
docker inspect \
  -f '{{json .Config.Entrypoint}}' \
  app01
```

### Runtime User

```bash
docker inspect \
  -f '{{.Config.User}}' \
  app01
```

### Environment Variables

```bash
docker inspect \
  -f '{{range .Config.Env}}{{println .}}{{end}}' \
  app01
```

### Networks

```bash
docker inspect \
  -f '{{json .NetworkSettings.Networks}}' \
  app01
```

### IP Address

```bash
docker inspect \
  -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' \
  app01
```

### Published Ports

```bash
docker inspect \
  -f '{{json .NetworkSettings.Ports}}' \
  app01
```

### Mounts

```bash
docker inspect \
  -f '{{json .Mounts}}' \
  app01
```

### Readable Mounts

```bash
docker inspect \
  -f '{{range .Mounts}}{{println .Type .Source "->" .Destination}}{{end}}' \
  app01
```

### Restart Policy

```bash
docker inspect \
  -f '{{.HostConfig.RestartPolicy.Name}}' \
  app01
```

### Memory Limit

```bash
docker inspect \
  -f '{{.HostConfig.Memory}}' \
  app01
```

### CPU Limit

```bash
docker inspect \
  -f '{{.HostConfig.NanoCpus}}' \
  app01
```

---

# ⚙️ 5. Processes & Resources

### Container Processes

```bash
docker top app01
```

### All Container Resource Usage

```bash
docker stats
```

### One Container

```bash
docker stats app01
```

### One-Time Output

```bash
docker stats app01 --no-stream
```

---

# 🔌 6. Ports

### Run With Port Mapping

```bash
docker run -d \
  --name app01 \
  -p 9090:8080 \
  image:tag
```

### Check Published Ports

```bash
docker port app01
```

### Check From `docker ps`

```bash
docker ps
```

### Inspect Ports

```bash
docker inspect \
  -f '{{json .NetworkSettings.Ports}}' \
  app01
```

### Test Application

```bash
curl -i http://localhost:9090
```

### Test Health Endpoint

```bash
curl -i http://localhost:9090/health
```

---

# 🌐 7. Networks

### List Networks

```bash
docker network ls
```

### Create Network

```bash
docker network create app-network
```

### Create Internal Network

```bash
docker network create \
  --internal \
  backend-network
```

### Inspect Network

```bash
docker network inspect app-network
```

### Run Container on Network

```bash
docker run -d \
  --name app01 \
  --network app-network \
  image:tag
```

### Connect Existing Container

```bash
docker network connect \
  app-network \
  app01
```

### Disconnect Container

```bash
docker network disconnect \
  app-network \
  app01
```

### Remove Network

```bash
docker network rm app-network
```

### Remove Unused Networks

```bash
docker network prune
```

### Host Network

```bash
docker run \
  --network host \
  image:tag
```

### No Network

```bash
docker run \
  --network none \
  image:tag
```

### Check Container Networks

```bash
docker inspect \
  -f '{{json .NetworkSettings.Networks}}' \
  app01
```

---

# 💾 8. Volumes

### List Volumes

```bash
docker volume ls
```

### Create Volume

```bash
docker volume create app-data
```

### Inspect Volume

```bash
docker volume inspect app-data
```

### Use Named Volume

```bash
docker run -d \
  --name app01 \
  -v app-data:/data \
  image:tag
```

### Read-Only Named Volume

```bash
docker run -d \
  --name app01 \
  -v app-data:/data:ro \
  image:tag
```

### Remove Volume

```bash
docker volume rm app-data
```

### Remove Unused Volumes

```bash
docker volume prune
```

### Check Container Mounts

```bash
docker inspect \
  -f '{{json .Mounts}}' \
  app01
```

### Readable Mount Output

```bash
docker inspect \
  -f '{{range .Mounts}}{{println .Type .Source "->" .Destination}}{{end}}' \
  app01
```

---

# 📂 9. Bind Mounts

### Bind Mount

```bash
docker run -d \
  --name app01 \
  -v /host/data:/data \
  image:tag
```

### Read-Only Bind Mount

```bash
docker run -d \
  --name app01 \
  -v /host/config:/config:ro \
  image:tag
```

### Using `--mount`

```bash
docker run -d \
  --name app01 \
  --mount type=bind,source=/host/data,target=/data \
  image:tag
```

### Inspect Mount

```bash
docker inspect \
  -f '{{json .Mounts}}' \
  app01
```

---

# 📊 10. CPU & Memory Limits

### Memory Limit

```bash
docker run -d \
  --name app01 \
  --memory=256m \
  image:tag
```

### CPU Limit

```bash
docker run -d \
  --name app01 \
  --cpus="0.5" \
  image:tag
```

### CPU + Memory

```bash
docker run -d \
  --name app01 \
  --memory=256m \
  --cpus="0.5" \
  image:tag
```

### Check Usage

```bash
docker stats app01 --no-stream
```

### Check OOMKilled

```bash
docker inspect \
  -f '{{.State.OOMKilled}}' \
  app01
```

### Check Memory Limit

```bash
docker inspect \
  -f '{{.HostConfig.Memory}}' \
  app01
```

### Check CPU Limit

```bash
docker inspect \
  -f '{{.HostConfig.NanoCpus}}' \
  app01
```

---

# 🔄 11. Restart Policy

### Always

```bash
docker run -d \
  --restart always \
  --name app01 \
  image:tag
```

### Unless Stopped

```bash
docker run -d \
  --restart unless-stopped \
  --name app01 \
  image:tag
```

### On Failure

```bash
docker run -d \
  --restart on-failure \
  --name app01 \
  image:tag
```

### Check Policy

```bash
docker inspect \
  -f '{{.HostConfig.RestartPolicy.Name}}' \
  app01
```

### Update Existing Container

```bash
docker update \
  --restart unless-stopped \
  app01
```

---

# ❤️ 12. Healthcheck

### Check Health From `docker ps`

```bash
docker ps
```

### Inspect Health

```bash
docker inspect \
  -f '{{json .State.Health}}' \
  app01
```

### Health Status Only

```bash
docker inspect \
  -f '{{.State.Health.Status}}' \
  app01
```

### Dockerfile Healthcheck

```dockerfile
HEALTHCHECK --interval=10s --timeout=3s --retries=3 \
  CMD curl -f http://localhost:8080/health || exit 1
```

---

# 📦 13. Images

### List Images

```bash
docker images
```

### Alternative

```bash
docker image ls
```

### Pull Image

```bash
docker pull nginx:latest
```

### Pull Specific Version

```bash
docker pull nginx:1.27
```

### Inspect Image

```bash
docker inspect nginx:1.27
```

### Image History

```bash
docker history nginx:1.27
```

### Tag Image

```bash
docker tag \
  myapp:v1 \
  username/myapp:v1
```

### Remove Image

```bash
docker rmi myapp:v1
```

### Force Remove Image

```bash
docker rmi -f myapp:v1
```

### Remove Dangling Images

```bash
docker image prune
```

### Remove All Unused Images

```bash
docker image prune -a
```

---

# 🏗️ 14. Docker Build

### Build

```bash
docker build -t myapp:v1 .
```

### Build Without Cache

```bash
docker build \
  --no-cache \
  -t myapp:v1 .
```

### Build With ARG

```bash
docker build \
  --build-arg APP_VERSION=2.0 \
  -t myapp:v2 .
```

### Build From Different Dockerfile

```bash
docker build \
  -f Dockerfile.prod \
  -t myapp:prod .
```

### Show Image History

```bash
docker history myapp:v1
```

---

# 🐳 15. Dockerfile Commands

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY app.py .

RUN useradd -m appuser

ARG APP_VERSION=1.0

ENV APP_ENV=production

RUN chown -R appuser:appuser /app

USER appuser

EXPOSE 8080

HEALTHCHECK --interval=10s --timeout=3s --retries=3 \
  CMD curl -f http://localhost:8080/health || exit 1

CMD ["python", "app.py"]
```

### Dockerfile Instructions

| Command | Purpose |
|---|---|
| 🟦 `FROM` | Base image |
| 🟩 `WORKDIR` | Working directory |
| 🟨 `COPY` | Copy files |
| 🟧 `ADD` | Copy with additional features |
| 🟥 `RUN` | Build-time command |
| 🟪 `ARG` | Build-time variable |
| 🟫 `ENV` | Runtime/default environment |
| 👤 `USER` | Runtime user |
| 🔌 `EXPOSE` | Container port metadata |
| ❤️ `HEALTHCHECK` | Health test |
| ▶️ `CMD` | Default command |
| ⚙️ `ENTRYPOINT` | Main executable |

---

# 🏗️ 16. Multi-Stage Build

```dockerfile
# BUILD
FROM node:22 AS builder

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

RUN npm run build


# RUNTIME
FROM nginx:latest

COPY --from=builder \
  /app/dist \
  /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

### Build

```bash
docker build -t web:v1 .
```

### Inspect Layers

```bash
docker history web:v1
```

---

# 🌱 17. Environment Variables

### Pass One Variable

```bash
docker run -d \
  -e APP_ENV=production \
  image:tag
```

### Multiple Variables

```bash
docker run -d \
  -e APP_ENV=production \
  -e APP_PORT=8080 \
  image:tag
```

### Environment File

```bash
docker run -d \
  --env-file .env \
  image:tag
```

### Check Environment

```bash
docker exec app01 env
```

### Inspect Environment

```bash
docker inspect \
  -f '{{range .Config.Env}}{{println .}}{{end}}' \
  app01
```

---

# 📋 18. Copy Files

### Host → Container

```bash
docker cp \
  ./test.txt \
  app01:/tmp/test.txt
```

### Container → Host

```bash
docker cp \
  app01:/tmp/test.txt \
  ./test.txt
```

---

# ☁️ 19. Docker Registry

### Login

```bash
docker login
```

### Logout

```bash
docker logout
```

### Tag

```bash
docker tag \
  myapp:v1 \
  username/myapp:v1
```

### Push

```bash
docker push username/myapp:v1
```

### Pull

```bash
docker pull username/myapp:v1
```

### Run

```bash
docker run -d \
  --name app01 \
  username/myapp:v1
```

---

# 💽 20. Docker Disk Usage

### Summary

```bash
docker system df
```

### Detailed

```bash
docker system df -v
```

### Docker Directory Size

```bash
sudo du -sh /var/lib/docker
```

---

# 🧹 21. Cleanup Commands

> [!WARNING]
> **Check before running cleanup commands on production systems.**

### Check First

```bash
docker ps -a
docker images
docker volume ls
docker network ls
docker system df -v
```

### Containers

```bash
docker container prune
```

### Images

```bash
docker image prune
```

### All Unused Images

```bash
docker image prune -a
```

### Networks

```bash
docker network prune
```

### Volumes

```bash
docker volume prune
```

### Build Cache

```bash
docker builder prune
```

### All Build Cache

```bash
docker builder prune -a
```

### System

```bash
docker system prune
```

### More Aggressive

```bash
docker system prune -a
```

---

# ℹ️ 22. Docker Information

### Docker Version

```bash
docker --version
```

### Client + Server Version

```bash
docker version
```

### Docker System Information

```bash
docker info
```

### Docker Service Status

```bash
systemctl status docker
```

### Docker Daemon Logs

```bash
journalctl -u docker
```

### Follow Docker Daemon Logs

```bash
journalctl -u docker -f
```

---

# 🚨 23. Troubleshooting Commands

### Container Not Working

```bash
docker ps -a
```

```bash
docker logs --tail 100 app01
```

```bash
docker inspect app01
```

```bash
docker stats app01 --no-stream
```

```bash
docker top app01
```

```bash
docker exec -it app01 sh
```

---

### Container Exited

```bash
docker inspect \
  -f '{{.State.ExitCode}}' \
  app01
```

```bash
docker inspect \
  -f '{{.State.OOMKilled}}' \
  app01
```

```bash
docker logs --tail 100 app01
```

---

### Website Not Opening

```bash
docker ps
```

```bash
docker port app01
```

```bash
docker logs --tail 100 app01
```

```bash
curl -i http://localhost:9090
```

---

### Network Problem

```bash
docker network ls
```

```bash
docker network inspect app-network
```

```bash
docker inspect \
  -f '{{json .NetworkSettings.Networks}}' \
  app01
```

---

### Storage Problem

```bash
docker volume ls
```

```bash
docker volume inspect app-data
```

```bash
docker inspect \
  -f '{{json .Mounts}}' \
  app01
```

---

### Memory Problem

```bash
docker stats app01 --no-stream
```

```bash
docker inspect \
  -f '{{.State.OOMKilled}}' \
  app01
```

```bash
docker inspect \
  -f '{{.HostConfig.Memory}}' \
  app01
```

---

### Health Problem

```bash
docker ps
```

```bash
docker inspect \
  -f '{{json .State.Health}}' \
  app01
```

---

# ⚡ 24. Most Used Commands

```bash
# Containers
docker ps
docker ps -a

# Logs
docker logs --tail 100 app01
docker logs -f app01

# Inspect
docker inspect app01

# Resources
docker stats app01 --no-stream

# Processes
docker top app01

# Shell
docker exec -it app01 sh

# Ports
docker port app01

# Images
docker images

# Networks
docker network ls
docker network inspect app-network

# Volumes
docker volume ls
docker volume inspect app-data

# Disk
docker system df -v

# Build
docker build -t image:v1 .

# History
docker history image:v1
```

---

# 🎯 25. Quick Command Map

| Need | Command |
|---|---|
| 🐳 What is running? | `docker ps` |
| 🔴 What stopped? | `docker ps -a` |
| 📜 What happened? | `docker logs` |
| 🔍 Full configuration? | `docker inspect` |
| 📊 CPU / RAM? | `docker stats` |
| ⚙️ Processes? | `docker top` |
| 🖥️ Enter container? | `docker exec` |
| 🔌 Port? | `docker port` |
| 🌐 Network? | `docker network inspect` |
| 💾 Storage? | `docker volume inspect` |
| 📦 Images? | `docker images` |
| 💽 Disk usage? | `docker system df -v` |
| 🏗️ Build? | `docker build` |
| ☁️ Upload image? | `docker push` |
| ☁️ Download image? | `docker pull` |

---

<div align="center">

## 🐳 Docker Cheat Sheet Complete

### `PS → LOGS → INSPECT → STATS → TOP → EXEC`

**Next Cheat Sheets**

🧩 **Docker Compose** → 🐝 **Docker Swarm** → ☸️ **Kubernetes**

</div>
