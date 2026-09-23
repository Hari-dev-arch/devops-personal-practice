# 🐳 Docker + Docker Compose + Docker Swarm
## Master DevOps Operations Cheat Sheet

> **Purpose:** One practical reference for daily operations, deployments and troubleshooting.

---

# 🧠 1. First Understand the Flow

## Docker

```text
Dockerfile
    ↓
docker build
    ↓
Image
    ↓
docker run
    ↓
Container
    ↓
Application
```

## Docker Compose

```text
compose.yaml
    ↓
docker compose up
    ↓
Services
    ↓
Containers
    ↓
Networks / Volumes
```

## Docker Swarm

```text
Dockerfile
    ↓
Build Image
    ↓
Registry
    ↓
stack.yaml
    ↓
Swarm Service
    ↓
Tasks
    ↓
Containers
```

---

# ⚡ 2. Which One Am I Looking At?

Start here:

```bash
docker ps
```

Check Compose:

```bash
docker compose ps
```

Check Swarm:

```bash
docker info | grep -i swarm
docker service ls
docker stack ls
```

Think:

```text
docker run       → individual container

docker compose   → multiple containers on a host

docker service   → Swarm-managed service

docker stack     → group of Swarm services
```

---

# 🔍 3. MASTER TROUBLESHOOTING ORDER

When someone says:

> "Application is down."

**Do not restart immediately.**

Follow:

```text
1. Is container/service running?
        ↓
2. What is its state?
        ↓
3. What do logs say?
        ↓
4. Is application listening?
        ↓
5. Is health endpoint working?
        ↓
6. Is port/network correct?
        ↓
7. Are env/config/secrets correct?
        ↓
8. Is storage okay?
        ↓
9. Are CPU/RAM okay?
        ↓
10. Fix root cause
        ↓
11. Verify again
```

Golden rule:

```text
OBSERVE → IDENTIFY → VERIFY → FIX → VERIFY
```

---

# 🐳 4. DOCKER — First Checks

## See running containers

```bash
docker ps
```

Include stopped containers:

```bash
docker ps -a
```

---

## Check container logs

```bash
docker logs <container>
```

Last 100 lines:

```bash
docker logs --tail 100 <container>
```

Follow live:

```bash
docker logs -f <container>
```

With timestamps:

```bash
docker logs -t --tail 100 <container>
```

---

## Inspect container

```bash
docker inspect <container>
```

Useful for:

```text
IP
Ports
Volumes
Networks
Environment
Restart policy
Health
Image
```

---

## Check container state

```bash
docker inspect <container> \
  --format '{{.State.Status}}'
```

Health:

```bash
docker inspect <container> \
  --format '{{json .State.Health}}'
```

---

# 🖥️ 5. Enter Container

```bash
docker exec -it <container> sh
```

If Bash exists:

```bash
docker exec -it <container> bash
```

Remember your context:

```text
HOST
ubuntu@server:~$

CONTAINER
/app #
```

Don't confuse host paths with container paths.

---

# 🌐 6. Ports

Check published ports:

```bash
docker port <container>
```

Example:

```text
0.0.0.0:8080 → container:80
```

Test from host:

```bash
curl -I http://localhost:8080
```

Check listening ports on Linux host:

```bash
ss -lntp
```

---

# 🌉 7. Docker Networks

List:

```bash
docker network ls
```

Inspect:

```bash
docker network inspect <network>
```

Container networks:

```bash
docker inspect <container> \
  --format '{{json .NetworkSettings.Networks}}'
```

### Remember

```text
bridge
→ normal Docker / single-host networking

user-defined bridge
→ containers can communicate by name

overlay
→ Swarm cross-node networking
```

---

# 💾 8. Docker Storage

List volumes:

```bash
docker volume ls
```

Inspect:

```bash
docker volume inspect <volume>
```

Container mounts:

```bash
docker inspect <container> \
  --format '{{json .Mounts}}'
```

### Named Volume

```text
Docker manages storage location.
```

### Bind Mount

```text
Host path → Container path
```

Example:

```text
/opt/app/data → /app/data
```

⚠️ Before deleting a container/volume, confirm whether important persistent data exists.

---

# 📊 9. CPU / Memory

Live usage:

```bash
docker stats
```

Single container:

```bash
docker stats <container>
```

Host:

```bash
free -h
df -h
top
```

Check Docker disk usage:

```bash
docker system df
```

Do **not** blindly run:

```bash
docker system prune -a
```

on production.

Investigate first.

---

# ⚙️ 10. Environment Variables

Inside container:

```bash
docker exec <container> printenv
```

Specific variable:

```bash
docker exec <container> printenv APP_ENV
```

From inspect:

```bash
docker inspect <container> \
  --format '{{range .Config.Env}}{{println .}}{{end}}'
```

Remember:

```text
Dockerfile ENV
      ↓
image default

docker run -e
      ↓
runtime value

Compose environment / env_file
      ↓
runtime value

CI/CD credentials
      ↓
may inject values during build/deployment
```

Never paste production passwords/tokens into tickets, GitHub or chat.

---

# ♻️ 11. Restart Policy

Inspect:

```bash
docker inspect <container> \
  --format '{{json .HostConfig.RestartPolicy}}'
```

Common policies:

```text
no
always
unless-stopped
on-failure
```

---

# 🏗️ 12. Docker Image Commands

List:

```bash
docker images
```

Pull:

```bash
docker pull nginx:1.28
```

Build:

```bash
docker build -t myapp:1 .
```

Run:

```bash
docker run -d --name myapp myapp:1
```

Tag:

```bash
docker tag myapp:1 username/myapp:1
```

Push:

```bash
docker push username/myapp:1
```

---

# 📄 13. Dockerfile Quick Reference

```dockerfile
FROM node:22-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

ENV NODE_ENV=production

EXPOSE 3000

CMD ["npm", "start"]
```

Remember:

```text
FROM     → base image
WORKDIR  → working directory
COPY     → copy files
RUN      → build-time command
ENV      → environment variable
EXPOSE   → documents application port
CMD      → default startup command
```

---

# 🏗️ 14. Multi-Stage Build

```dockerfile
FROM node:22-alpine AS builder

WORKDIR /app
COPY . .
RUN npm install
RUN npm run build


FROM node:22-alpine

WORKDIR /app

COPY --from=builder /app/dist ./dist

CMD ["node", "dist/server.js"]
```

Purpose:

```text
Build dependencies
      ↓
Build application
      ↓
Copy only required output
      ↓
Smaller runtime image
```

---

# 🧩 15. DOCKER COMPOSE — Daily Commands

Go to project directory:

```bash
cd /opt/myapp
```

Validate configuration:

```bash
docker compose config
```

Start:

```bash
docker compose up -d
```

Status:

```bash
docker compose ps
```

Logs:

```bash
docker compose logs --tail 100
```

Specific service:

```bash
docker compose logs --tail 100 <service>
```

Follow:

```bash
docker compose logs -f <service>
```

Enter service:

```bash
docker compose exec <service> sh
```

Restart service:

```bash
docker compose restart <service>
```

Stop/remove:

```bash
docker compose down
```

Rebuild:

```bash
docker compose build
```

Build + start:

```bash
docker compose up -d --build
```

---

# 📄 16. Simple Compose File

```yaml
services:

  web:
    image: nginx:1.28

    ports:
      - "8080:80"

    environment:
      APP_ENV: staging

    restart: unless-stopped

    networks:
      - app-network

networks:

  app-network:
    driver: bridge
```

Deploy:

```bash
docker compose up -d
```

Verify:

```bash
docker compose ps
```

Test:

```bash
curl -I http://localhost:8080
```

---

# 🔐 17. Compose `.env`

Example:

```env
APP_ENV=staging
APP_PORT=8080
```

Compose:

```yaml
services:

  app:

    image: myapp:1

    environment:
      APP_ENV: ${APP_ENV}

    ports:
      - "${APP_PORT}:3000"
```

Check resolved configuration:

```bash
docker compose config
```

This is extremely useful when debugging variable substitution.

---

# 🩺 18. Healthcheck

Example:

```yaml
healthcheck:

  test:
    - CMD
    - curl
    - -f
    - http://localhost:3000/health

  interval: 30s
  timeout: 5s
  retries: 3
```

Check:

```bash
docker inspect <container> \
  --format '{{json .State.Health}}'
```

Application test:

```bash
curl -i http://localhost:3000/health
```

---

# 🚨 19. COMPOSE TROUBLESHOOTING ORDER

Application down:

```bash
docker compose ps
```

Then:

```bash
docker compose logs --tail 100 <service>
```

Then:

```bash
docker compose config
```

Then inspect actual container:

```bash
docker inspect <container>
```

Then check:

```bash
docker stats
df -h
free -h
```

Think:

```text
ps
 ↓
logs
 ↓
config
 ↓
container inspect
 ↓
network / ports
 ↓
env
 ↓
storage
 ↓
resources
```

---

# 🐝 20. DOCKER SWARM — Mental Model

```text
Swarm Cluster
     ↓
Manager
     ↓
Service
     ↓
Tasks
     ↓
Containers
```

Remember:

```text
Service
→ desired application state

Task
→ one scheduled service instance

Container
→ actual running process

Replica
→ number of desired tasks
```

---

# 🐝 21. Swarm First Commands

Check Swarm:

```bash
docker info | grep -i swarm
```

Nodes:

```bash
docker node ls
```

Services:

```bash
docker service ls
```

Stacks:

```bash
docker stack ls
```

---

# 🔥 22. SWARM MASTER TROUBLESHOOTING ORDER

Someone says:

> "Swarm application is down."

Start:

```bash
docker service ls
```

Example:

```text
myapp_web    1/3
```

Then:

```bash
docker service ps myapp_web
```

Look for:

```text
Failed
Rejected
Pending
Shutdown
Running
```

Then:

```bash
docker service logs --tail 100 myapp_web
```

Then:

```bash
docker node ls
```

Then:

```bash
docker service inspect myapp_web
```

### Memorize this:

```text
service ls
    ↓
service ps
    ↓
service logs
    ↓
node ls
    ↓
service inspect
```

---

# 📦 23. Swarm Stack Commands

Deploy:

```bash
docker stack deploy -c stack.yaml myapp
```

List stacks:

```bash
docker stack ls
```

Services:

```bash
docker stack services myapp
```

Tasks:

```bash
docker stack ps myapp
```

Remove:

```bash
docker stack rm myapp
```

---

# 🔢 24. Swarm Scale

```bash
docker service scale myapp_web=5
```

Verify:

```bash
docker service ls
```

Expected:

```text
myapp_web   5/5
```

---

# ♻️ 25. Swarm Self-Healing

Desired:

```text
3 replicas
```

One crashes:

```text
3 → 2
```

Swarm creates replacement:

```text
2 → 3
```

You normally troubleshoot the **service/task**, not manually recreate individual task containers.

---

# 🌉 26. Swarm Overlay Network

Create:

```bash
docker network create \
  --driver overlay \
  app-network
```

List:

```bash
docker network ls
```

Inspect:

```bash
docker network inspect app-network
```

Remember:

```text
Bridge  → primarily single host

Overlay → Swarm nodes
```

---

# 🔐 27. Swarm Secrets

List:

```bash
docker secret ls
```

Create:

```bash
printf 'value' | docker secret create db_password -
```

Service receives it under:

```text
/run/secrets/
```

Inside container:

```bash
ls -l /run/secrets/
```

Remember:

```text
Secret → sensitive
Config → non-sensitive
```

---

# 📄 28. Swarm Config

List:

```bash
docker config ls
```

Create:

```bash
docker config create app_config app.conf
```

Inspect:

```bash
docker config inspect app_config
```

---

# 🔄 29. Rolling Update

Stack:

```yaml
update_config:

  parallelism: 1
  delay: 5s
  order: start-first
```

Means:

```text
Update one replica
      ↓
Wait 5 sec
      ↓
Next replica
```

`start-first`:

```text
Start new
   ↓
Stop old
```

`stop-first`:

```text
Stop old
   ↓
Start new
```

Watch deployment:

```bash
watch docker stack ps myapp
```

---

# ⏪ 30. Rollback

Bad deployment:

```bash
docker service rollback myapp_web
```

Check:

```bash
docker service ps myapp_web
```

Remember:

> Rollback restores the previous service specification.

---

# 🛠️ 31. Node Maintenance

Check:

```bash
docker node ls
```

Drain:

```bash
docker node update \
  --availability drain \
  swarm02
```

After maintenance:

```bash
docker node update \
  --availability active \
  swarm02
```

Remember:

```text
Ready + Active
→ healthy and schedulable

Ready + Drain
→ healthy but don't schedule workload tasks

Down
→ node unavailable
```

---

# 📊 32. Swarm Resources

```yaml
deploy:

  resources:

    reservations:
      memory: 256M

    limits:
      cpus: "0.50"
      memory: 512M
```

Remember:

```text
Reservation
→ scheduler requirement

Limit
→ maximum allowed usage
```

---

# 📍 33. Placement

Manager only:

```yaml
placement:

  constraints:
    - node.role == manager
```

Maximum one replica per node:

```yaml
placement:

  max_replicas_per_node: 1
```

Spread preference:

```yaml
placement:

  preferences:
    - spread: engine.labels.zone
```

---

# 📄 34. Simple Production-Style Swarm Stack

```yaml
x-deploy: &default-deploy

  replicas: 2

  update_config:
    parallelism: 1
    delay: 5s
    order: start-first
    failure_action: rollback

  restart_policy:
    condition: on-failure

  resources:

    reservations:
      memory: 128M

    limits:
      cpus: "0.50"
      memory: 512M


services:

  web:

    image: registry.example.com/myapp:1.0

    networks:
      - app-network

    secrets:
      - db_password

    deploy:
      <<: *default-deploy


networks:

  app-network:
    driver: overlay


secrets:

  db_password:
    external: true
```

---

# 🚦 35. HTTP / Application Troubleshooting

Container can be running while application is broken.

Check:

```bash
docker ps
```

Then test application:

```bash
curl -i http://localhost:<PORT>
```

Health endpoint:

```bash
curl -i http://localhost:<PORT>/health
```

Check listening ports:

```bash
ss -lntp
```

Think layer-by-layer:

```text
Process running?
      ↓
Port listening?
      ↓
Application responding?
      ↓
Reverse proxy working?
      ↓
DNS working?
      ↓
External request working?
```

---

# 🌐 36. Network Troubleshooting Flow

Application A cannot reach Application B:

```text
1. Is B running?
2. Is B listening?
3. Correct port?
4. Same/connected network?
5. DNS/service name resolving?
6. Can A connect to B?
7. Firewall/security rule?
8. Application error?
```

Useful commands:

```bash
docker network ls
docker network inspect <network>
docker inspect <container>
ss -lntp
curl -v http://host:port
```

If available:

```bash
nc -vz host port
```

---

# 💽 37. Disk Full Troubleshooting

Check host:

```bash
df -h
```

Docker usage:

```bash
docker system df
```

Container sizes:

```bash
docker ps -s
```

Check Docker directory carefully:

```bash
du -sh /var/lib/docker/* 2>/dev/null
```

Large logs:

```bash
du -sh /var/lib/docker/containers/* 2>/dev/null
```

Rule:

```text
Find what consumes disk
        ↓
Understand whether needed
        ↓
Choose safe cleanup
        ↓
Verify disk
```

Never make this your first command:

```bash
docker system prune -a
```

---

# 🧠 38. Container Keeps Restarting

Start:

```bash
docker ps -a
```

Then:

```bash
docker logs --tail 100 <container>
```

Then:

```bash
docker inspect <container>
```

Check:

```text
Exit code
Application exception
Missing ENV
Missing secret
DB connection
Port conflict
Permission
OOM
Healthcheck
Volume/path
```

---

# 💥 39. Container Exited

```bash
docker ps -a
```

Get exit code:

```bash
docker inspect <container> \
  --format '{{.State.ExitCode}}'
```

Check reason:

```bash
docker inspect <container> \
  --format '{{.State.Error}}'
```

Logs:

```bash
docker logs --tail 100 <container>
```

Check OOM:

```bash
docker inspect <container> \
  --format '{{.State.OOMKilled}}'
```

---

# 🔎 40. Image Version Troubleshooting

Container image:

```bash
docker inspect <container> \
  --format '{{.Config.Image}}'
```

Swarm service image:

```bash
docker service inspect <service> \
  --format '{{.Spec.TaskTemplate.ContainerSpec.Image}}'
```

Compose resolved image:

```bash
docker compose config
```

Useful when:

```text
Expected version ≠ running version
```

---

# 🔥 41. "It Works Inside Container But Not Browser"

Think:

```text
Application
   ↓
Container port
   ↓
Published port
   ↓
Host
   ↓
Reverse proxy / LB
   ↓
DNS
   ↓
Browser
```

Test from inside:

```bash
curl http://localhost:<APP_PORT>
```

Then host:

```bash
curl http://localhost:<PUBLISHED_PORT>
```

Then external URL.

Find the **first layer that fails**.

---

# 🔐 42. "Application Cannot Connect to DB"

Check application logs first.

Then verify:

```text
DB hostname
DB port
Username
Password/secret
Network connectivity
DNS
Firewall
Database itself
Connection limits
```

Connectivity test when appropriate:

```bash
nc -vz <DB_HOST> <DB_PORT>
```

Do not immediately blame Docker.

---

# 📋 43. Before Deployment

Quick checklist:

```text
☐ Correct branch/version?
☐ Correct image tag?
☐ Correct environment?
☐ Correct env variables?
☐ Secrets available?
☐ Config available?
☐ Network exists?
☐ Storage correct?
☐ Resource limits reasonable?
☐ Healthcheck correct?
☐ Rollback path known?
```

---

# ✅ 44. After Deployment

Verify:

## Docker

```bash
docker ps
docker logs --tail 50 <container>
```

## Compose

```bash
docker compose ps
docker compose logs --tail 50
```

## Swarm

```bash
docker service ls
docker stack services <stack>
docker stack ps <stack>
```

Then:

```bash
curl <health-endpoint>
```

Finally check logs again.

---

# 🚨 45. Production Safety

Before running a command ask:

```text
READ ONLY?
or
CHANGES STATE?
```

Usually safe investigation:

```bash
docker ps
docker images
docker logs
docker inspect
docker stats
docker network ls
docker volume ls
docker service ls
docker service ps
docker service logs
docker node ls
docker stack ls
docker stack services
docker stack ps
```

Commands requiring more care:

```bash
docker restart
docker rm
docker rmi
docker compose down
docker service rm
docker stack rm
docker service update
docker service rollback
docker node update
docker system prune
docker swarm leave
```

Always understand impact first.

---

# 🏆 46. THE COMMANDS TO REMEMBER

## Docker problem

```bash
docker ps -a
docker logs --tail 100 <container>
docker inspect <container>
docker stats
docker network inspect <network>
```

## Compose problem

```bash
docker compose ps
docker compose logs --tail 100 <service>
docker compose config
docker inspect <container>
```

## Swarm problem

```bash
docker service ls
docker service ps <service>
docker service logs --tail 100 <service>
docker node ls
docker service inspect <service>
```

## Host problem

```bash
df -h
free -h
top
ss -lntp
```

## Application problem

```bash
curl -iv http://localhost:<port>
```

---

# 🎯 47. ONE MASTER TROUBLESHOOTING MAP

```text
                 USER REPORTS ISSUE
                        │
                        ▼
              WHAT IS RUNNING IT?
                        │
          ┌─────────────┼─────────────┐
          ▼             ▼             ▼
       Docker        Compose        Swarm
          │             │             │
          ▼             ▼             ▼
      docker ps     compose ps    service ls
          │             │             │
          ▼             ▼             ▼
        logs           logs        service ps
          │             │             │
          ▼             ▼             ▼
       inspect        config          logs
          │             │             │
          └─────────────┼─────────────┘
                        ▼
                 CHECK APPLICATION
                        │
                 curl health/API
                        │
                        ▼
                 CHECK NETWORK
                        │
            ports / DNS / connectivity
                        │
                        ▼
                  CHECK CONFIG
                        │
               ENV / secret / config
                        │
                        ▼
                  CHECK STORAGE
                        │
               volume / disk / mount
                        │
                        ▼
                  CHECK RESOURCE
                        │
                 CPU / RAM / OOM
                        │
                        ▼
                  ROOT CAUSE
                        │
                        ▼
                       FIX
                        │
                        ▼
                     VERIFY
```

---

# 💡 Final Rule

When production breaks:

```text
Don't guess.
Don't restart first.
Don't delete first.

Check status.
Check tasks/process.
Check logs.
Check configuration.
Check connectivity.
Check resources.
Find root cause.
Fix.
Verify.
```

## DevOps mindset

```text
OBSERVE
   ↓
NARROW DOWN
   ↓
PROVE
   ↓
FIX
   ↓
VERIFY
```

That workflow is more important than memorizing hundreds of Docker commands.
