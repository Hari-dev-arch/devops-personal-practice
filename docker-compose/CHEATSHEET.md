# 🐳 Docker Compose Command Cheat Sheet

<p align="center">

![Docker Compose](https://img.shields.io/badge/Docker_Compose-Command_Cheat_Sheet-2496ED?logo=docker&logoColor=white)
![DevOps](https://img.shields.io/badge/DevOps-Quick_Reference-brightgreen)
![Platform](https://img.shields.io/badge/Platform-Linux-orange)

### ⚡ Quick • Practical • Topic-wise

</p>

---

# 🧩 1. Basic Compose Commands

### Start Application

```bash
docker compose up
```

### Start in Background

```bash
docker compose up -d
```

### Build and Start

```bash
docker compose up -d --build
```

### Stop Services

```bash
docker compose stop
```

### Start Existing Stopped Services

```bash
docker compose start
```

### Restart Services

```bash
docker compose restart
```

### Restart Specific Service

```bash
docker compose restart backend
```

### Stop and Remove Compose Application

```bash
docker compose down
```

### Remove Application Including Volumes

> ⚠️ Deletes Compose-managed named volumes.

```bash
docker compose down -v
```

---

# 📋 2. Container / Service Status

### List Running Services

```bash
docker compose ps
```

### List All Services

```bash
docker compose ps -a
```

### List Compose Images

```bash
docker compose images
```

### List Processes

```bash
docker compose top
```

---

# 📝 3. Logs

### View All Logs

```bash
docker compose logs
```

### Service Logs

```bash
docker compose logs backend
```

### Follow Logs Live

```bash
docker compose logs -f backend
```

### Last 100 Lines

```bash
docker compose logs --tail=100 backend
```

### Logs with Timestamps

```bash
docker compose logs -t backend
```

---

# 🖥️ 4. Execute Commands Inside Container

### Open Shell

```bash
docker compose exec backend sh
```

### Bash Shell

```bash
docker compose exec backend bash
```

### Check Environment Variables

```bash
docker compose exec backend env
```

### Check Specific Variable

```bash
docker compose exec backend printenv APP_ENV
```

### Check Files

```bash
docker compose exec backend ls -lah
```

### Check Working Directory

```bash
docker compose exec backend pwd
```

### Check Runtime User

```bash
docker compose exec backend whoami
```

---

# 🔨 5. Build

### Build All Services

```bash
docker compose build
```

### Build Specific Service

```bash
docker compose build backend
```

### Build Without Cache

```bash
docker compose build --no-cache backend
```

### Build and Apply Changes

```bash
docker compose up -d --build
```

> `docker compose build` builds the image.  
> `docker compose up -d` creates/recreates services as required.

---

# 🏷️ 6. Image and Build

### Use Existing Image

```yaml
services:
  frontend:
    image: nginx:latest
```

### Build from Dockerfile

```yaml
services:
  backend:
    build: .
```

### Build and Give Image a Tag

```yaml
services:
  backend:
    build: .
    image: my-backend:v1
```

Mental model:

```text
build: .              → HOW to build
image: my-backend:v1  → image NAME + TAG
```

---

# 🌐 7. Ports

### Compose YAML

```yaml
services:
  backend:
    ports:
      - "9090:8080"
```

Meaning:

```text
HOST : CONTAINER
9090 : 8080
```

Test:

```bash
curl http://localhost:9090
```

---

# 🌱 8. Environment Variables

### Direct Environment Variable

```yaml
services:
  backend:
    environment:
      APP_ENV: staging
```

### Variable from `.env`

`.env`

```env
APP_ENV=staging
```

`compose.yaml`

```yaml
services:
  backend:
    environment:
      APP_ENV: ${APP_ENV}
```

Check resolved configuration:

```bash
docker compose config
```

Check inside container:

```bash
docker compose exec backend printenv APP_ENV
```

---

# 📄 9. env_file

`backend.env`

```env
APP_ENV=staging
DB_HOST=database
```

Compose:

```yaml
services:
  backend:
    env_file:
      - backend.env
```

---

# 💾 10. Named Volumes

### Compose YAML

```yaml
services:
  storage:
    image: busybox
    volumes:
      - app-data:/data

volumes:
  app-data:
```

Mental model:

```text
app-data : /data
    │        │
    │        └── Container path
    │
    └── Docker named volume
```

### List Volumes

```bash
docker volume ls
```

### Inspect Volume

```bash
docker volume inspect <volume_name>
```

Example:

```bash
docker volume inspect docker-compose-lab_app-data
```

### Remove Volume

```bash
docker volume rm <volume_name>
```

> ⚠️ Removing a volume deletes its stored data.

---

# 📁 11. Bind Mounts

### Relative Host Path

```yaml
volumes:
  - ./nginx.conf:/etc/nginx/conf.d/default.conf:ro
```

Meaning:

```text
./nginx.conf
      ↓
Host file
      ↓
Container file
      ↓
Read-only (:ro)
```

### Absolute Host Path

```yaml
volumes:
  - /home/user/app/config:/app/config
```

---

# 🌐 12. Networks

### Define Custom Network

```yaml
networks:
  app-network:
```

### Attach Service

```yaml
services:
  backend:
    networks:
      - app-network
```

### Multiple Services

```yaml
services:

  frontend:
    networks:
      - app-network

  backend:
    networks:
      - app-network

networks:
  app-network:
```

Communication:

```text
frontend
    │
    │ app-network
    ↓
backend:8080
```

Docker Compose provides service-name DNS.

Example Nginx:

```nginx
proxy_pass http://backend:8080;
```

### List Networks

```bash
docker network ls
```

### Inspect Network

```bash
docker network inspect <network_name>
```

---

# ❤️ 13. Healthcheck

```yaml
services:
  backend:

    healthcheck:
      test:
        [
          "CMD",
          "python",
          "-c",
          "import urllib.request; urllib.request.urlopen('http://localhost:8080/')"
        ]

      interval: 10s
      timeout: 3s
      retries: 3
```

Check:

```bash
docker compose ps
```

Possible status:

```text
Up (healthy)
Up (unhealthy)
```

Mental model:

```text
RUNNING ≠ HEALTHY

Running → process exists
Healthy → application healthcheck passes
```

---

# 🔗 14. depends_on

### Wait for Healthy Backend

```yaml
services:

  frontend:
    depends_on:
      backend:
        condition: service_healthy
```

Flow:

```text
backend starts
      ↓
healthcheck
      ↓
backend healthy
      ↓
frontend starts
```

---

# 🔄 15. Restart Policy

```yaml
services:
  backend:
    restart: unless-stopped
```

Common policies:

```text
no
always
on-failure
unless-stopped
```

Example:

```yaml
restart: unless-stopped
```

Meaning:

```text
Unexpected crash
      ↓
Docker restarts container

Manual stop
      ↓
Docker respects the stop
```

---

# 📈 16. Scaling

### Scale Backend to 3 Containers

```bash
docker compose up -d --scale backend=3
```

Check:

```bash
docker compose ps
```

Example:

```text
backend-1
backend-2
backend-3
```

### ⚠️ Fixed Host Port Problem

This can prevent normal scaling:

```yaml
backend:
  ports:
    - "9090:8080"
```

Multiple replicas cannot all bind the same fixed host port on the same host.

Common architecture:

```text
                    ┌── backend-1:8080
Client → Proxy ─────┼── backend-2:8080
                    └── backend-3:8080
```

---

# 🧱 17. Multi-Stage Dockerfile

```dockerfile
# Stage 1 - Build

FROM alpine:latest AS builder

WORKDIR /build

RUN echo "Application artifact" > app.txt


# Stage 2 - Runtime

FROM alpine:latest

WORKDIR /app

COPY --from=builder /build/app.txt .

CMD ["cat", "/app/app.txt"]
```

Build:

```bash
docker build -t multistage-app:v1 .
```

Run:

```bash
docker run --rm multistage-app:v1
```

Mental model:

```text
Builder
Source + Build Tools
        ↓
     Artifact
        ↓
Runtime Image
Artifact + Runtime Requirements
```

---

# 🧪 18. Validate Compose YAML

### Validate Configuration

```bash
docker compose config
```

This should normally be run before applying important changes:

```text
EDIT
 ↓
docker compose config
 ↓
docker compose up -d
 ↓
VERIFY
```

---

# 🔍 19. Troubleshooting — First Checks

### 1. Check Services

```bash
docker compose ps -a
```

### 2. Check Logs

```bash
docker compose logs backend
```

### 3. Follow Logs

```bash
docker compose logs -f backend
```

### 4. Check Resolved Compose Config

```bash
docker compose config
```

### 5. Enter Container

```bash
docker compose exec backend sh
```

### 6. Check Environment

```bash
docker compose exec backend env
```

### 7. Check Network

```bash
docker network ls
```

```bash
docker network inspect <network_name>
```

### 8. Check Volume

```bash
docker volume ls
```

```bash
docker volume inspect <volume_name>
```

---

# 🚨 20. Troubleshooting Scenarios

## Container Not Running

Check:

```bash
docker compose ps -a
```

Then:

```bash
docker compose logs <service>
```

---

## Container Keeps Restarting

```bash
docker compose ps -a
```

```bash
docker compose logs --tail=100 <service>
```

Check:

```text
Application crash
Bad command
Missing environment variable
Permission issue
Dependency unavailable
```

---

## Container Running but Application Not Working

Check:

```bash
docker compose ps
```

Then:

```bash
docker compose logs <service>
```

Enter container:

```bash
docker compose exec <service> sh
```

Check listening processes/tools available in the image as appropriate.

---

## Service is Unhealthy

```bash
docker compose ps
```

Check logs:

```bash
docker compose logs <service>
```

Inspect health information:

```bash
docker inspect <container_name>
```

Useful formatted check:

```bash
docker inspect --format='{{json .State.Health}}' <container_name>
```

---

## Environment Variable Wrong

Check resolved Compose config:

```bash
docker compose config
```

Check inside container:

```bash
docker compose exec backend printenv APP_ENV
```

Check `.env`:

```bash
cat .env
```

---

## Port Already in Use

Typical error:

```text
address already in use
```

Check running containers:

```bash
docker ps
```

Check host listening ports:

```bash
ss -lntp
```

Then verify Compose port mapping:

```yaml
ports:
  - "8080:80"
```

---

## Service Cannot Reach Another Service

Check both services are running:

```bash
docker compose ps
```

Check networks:

```bash
docker network ls
```

Inspect project network:

```bash
docker network inspect <network_name>
```

From a container, test the service name if the image has a suitable client:

```bash
docker compose exec frontend sh
```

Then for example:

```bash
wget -qO- http://backend:8080
```

Use the **Compose service name**, not `localhost`, for another container:

```text
backend:8080     ✅
localhost:8080   ❌ for reaching another container
```

---

## Volume Data Missing

Check volume:

```bash
docker volume ls
```

Inspect:

```bash
docker volume inspect <volume_name>
```

Check mount inside container:

```bash
docker compose exec storage ls -lah /data
```

Remember:

```text
Container removed → named volume can survive
Volume removed    → volume data is deleted
```

---

## YAML / Indentation Error

Validate:

```bash
docker compose config
```

Correct top-level structure:

```yaml
services:

volumes:

networks:
```

---

## Code Changed but Application Still Shows Old Code

Restart alone may not rebuild the image:

```bash
docker compose restart backend
```

Instead, when source is copied into the image:

```bash
docker compose up -d --build
```

Mental model:

```text
Code changed
    ↓
Build new image
    ↓
Recreate/apply container
```

---

# 🛠️ 21. Useful Inspection Commands

### Show Resolved Configuration

```bash
docker compose config
```

### Show Service Names

```bash
docker compose config --services
```

### Show Images

```bash
docker compose images
```

### Show Processes

```bash
docker compose top
```

### Container Details

```bash
docker inspect <container_name>
```

### Docker Networks

```bash
docker network ls
```

### Docker Volumes

```bash
docker volume ls
```

---

# 🧹 22. Cleanup

### Stop Services

```bash
docker compose stop
```

### Remove Containers + Compose Networks

```bash
docker compose down
```

### Remove Containers + Networks + Volumes

```bash
docker compose down -v
```

> ⚠️ Be careful with `-v`.

### Remove Images Manually

```bash
docker image rm <image>
```

---

# 🏗️ 23. Full Docker Compose Example

```yaml
services:

  backend:
    build: .
    image: way-compose-backend:v1

    environment:
      APP_ENV: ${APP_ENV}

    networks:
      - app-network

    healthcheck:
      test:
        [
          "CMD",
          "python",
          "-c",
          "import urllib.request; urllib.request.urlopen('http://localhost:8080/')"
        ]
      interval: 10s
      timeout: 3s
      retries: 3

    restart: unless-stopped


  frontend:
    image: nginx:latest

    ports:
      - "8080:80"

    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf:ro

    networks:
      - app-network

    depends_on:
      backend:
        condition: service_healthy

    restart: unless-stopped


  storage:
    image: busybox

    command: ["sh", "-c", "sleep infinity"]

    volumes:
      - app-data:/data

    restart: unless-stopped


volumes:
  app-data:


networks:
  app-network:
```

---

# 🗺️ 24. Architecture of the Example

```text
                    HOST
                     │
                  :8080
                     │
                     ▼
              ┌─────────────┐
              │  frontend   │
              │    Nginx    │
              └──────┬──────┘
                     │
                 app-network
                     │
                     ▼
              ┌─────────────┐
              │   backend   │
              │    :8080    │
              └─────────────┘


              ┌─────────────┐
              │   storage   │
              │    /data    │
              └──────┬──────┘
                     │
                     ▼
                  app-data
               Named Volume
```

---

# ⚡ 25. Most Used Commands

```bash
docker compose config

docker compose up -d

docker compose up -d --build

docker compose ps

docker compose ps -a

docker compose logs backend

docker compose logs -f backend

docker compose exec backend sh

docker compose restart backend

docker compose stop

docker compose start

docker compose down

docker compose build backend

docker compose up -d --scale backend=3
```

---

# 🧠 26. Quick Mental Map

```text
compose.yaml
     │
     ├── services
     │     ├── image / build
     │     ├── ports
     │     ├── environment
     │     ├── volumes
     │     ├── networks
     │     ├── healthcheck
     │     ├── depends_on
     │     └── restart
     │
     ├── volumes
     │
     └── networks
```

---

# 🔥 27. Troubleshooting Flow

```text
Problem
   ↓
docker compose ps -a
   ↓
docker compose logs <service>
   ↓
docker compose config
   ↓
docker compose exec <service> sh
   ↓
Check ENV / Network / Port / Volume / Health
   ↓
Find root cause
   ↓
Fix
   ↓
docker compose up -d
   ↓
Verify
```

---

# 🎯 Docker vs Docker Compose

| Docker | Docker Compose |
|---|---|
| `docker run` | `docker compose up` |
| `docker ps` | `docker compose ps` |
| `docker logs` | `docker compose logs` |
| `docker exec` | `docker compose exec` |
| `docker build` | `docker compose build` |
| Individual containers | Multi-service application |
| CLI configuration | YAML configuration |

---

<p align="center">

### 🐳 Docker Compose

**Define → Validate → Deploy → Verify → Troubleshoot**

```text
compose.yaml
     ↓
docker compose config
     ↓
docker compose up -d
     ↓
docker compose ps
     ↓
docker compose logs
```

</p>
