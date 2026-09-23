# 🐳 Docker Swarm — Practical End-to-End Guide

> A practical Docker Swarm reference covering architecture, services, replicas, stacks, networking, scheduling, updates, rollback, secrets, configs, maintenance, and troubleshooting.

---

## 📌 Contents

1. Swarm Basics
2. Initialize Swarm
3. Manager & Worker Nodes
4. Service, Task & Container
5. Replicas & Scaling
6. Self-Healing
7. Docker Run vs Swarm Service
8. Published Ports & Routing Mesh
9. Overlay Networks
10. Docker Stack
11. YAML Anchors
12. Placement Constraints
13. Resource Limits
14. Resource Reservations
15. Restart Policy
16. Rolling Updates
17. Image Updates
18. Rollback
19. Swarm Secrets
20. Swarm Configs
21. Node Drain & Maintenance
22. Replica Distribution
23. Traefik / Load Balancer Concept
24. Troubleshooting
25. Complete Sample `stack.yaml`
26. Command Cheat Sheet

---

# 1. 🧠 What is Docker Swarm?

Docker Swarm allows multiple Docker hosts to work together as a **cluster**.

```text
                 Docker Swarm
                      │
             ┌────────┴────────┐
             │                 │
          Manager           Workers
             │                 │
        schedules tasks    run workloads
```

Main idea:

```text
You define desired state
        ↓
Swarm maintains that state
```

Example:

```yaml
replicas: 3
```

means:

> I always want 3 instances of this service running.

---

# 2. 🚀 Initialize Swarm

Check current status:

```bash
docker info
```

Initialize Swarm:

```bash
docker swarm init
```

Check nodes:

```bash
docker node ls
```

Example:

```text
HOSTNAME         STATUS   AVAILABILITY   MANAGER STATUS
swarm01          Ready    Active         Leader
```

---

# 3. 🖥️ Manager & Worker Nodes

## Manager

Manager nodes control the Swarm.

They handle:

- cluster state
- service definitions
- task scheduling
- desired state
- node management

Check nodes:

```bash
docker node ls
```

Get worker join command:

```bash
docker swarm join-token worker
```

Get manager join command:

```bash
docker swarm join-token manager
```

A new worker joins using the generated command:

```bash
docker swarm join --token <WORKER_TOKEN> <MANAGER_IP>:2377
```

> `<WORKER_TOKEN>` and `<MANAGER_IP>` are placeholders. Do not type the `< >`.

---

# 4. 📦 Service → Task → Container

One of the most important Swarm concepts:

```text
SERVICE
   ↓
desired application state
   ↓
TASK
   ↓
one scheduled instance
   ↓
CONTAINER
   ↓
actual running process
```

Example:

```bash
docker service create --name web nginx:latest
```

Check services:

```bash
docker service ls
```

Check tasks:

```bash
docker service ps web
```

Check actual containers on the current node:

```bash
docker ps
```

---

# 5. 🔢 Replicas & Scaling

Create 3 replicas:

```bash
docker service create \
  --name web \
  --replicas 3 \
  nginx:latest
```

Or scale an existing service:

```bash
docker service scale web=3
```

Verify:

```bash
docker service ls
```

Expected:

```text
NAME   MODE         REPLICAS
web    replicated   3/3
```

Meaning:

```text
Desired = 3
Running = 3
```

---

# 6. ♻️ Self-Healing / Desired State

Suppose:

```text
Desired replicas = 3
Running replicas = 3
```

One container crashes.

Now:

```text
Running = 2
Desired = 3
```

Swarm detects the difference and creates a replacement task.

```text
Container failure
      ↓
Swarm detects 2/3
      ↓
Creates replacement task
      ↓
Back to 3/3
```

This is **desired-state reconciliation / self-healing**.

---

# 7. 🆚 `docker run` vs Swarm Service

Normal Docker:

```bash
docker run -d nginx
```

You manage the container.

Swarm:

```bash
docker service create --name web nginx
```

Swarm manages the service.

Remember:

```text
docker run
    ↓
normal container

docker service
    ↓
Swarm-managed workload
```

Normal Docker commands and Docker Compose can still work while Swarm mode is enabled.

---

# 8. 🌐 Published Ports & Routing Mesh

Publish a service port:

```bash
docker service update \
  --publish-add 8081:80 \
  web
```

Meaning:

```text
Host/Swarm port 8081
        ↓
Routing Mesh
        ↓
Service
        ↓
Container port 80
```

Inspect published ports:

```bash
docker service inspect web \
  --format '{{json .Endpoint.Ports}}'
```

Typical result:

```json
[
  {
    "Protocol": "tcp",
    "TargetPort": 80,
    "PublishedPort": 8081,
    "PublishMode": "ingress"
  }
]
```

### Routing Mesh

Routing Mesh allows traffic arriving on a Swarm published port to be routed to an available task of the service.

```text
Client
  ↓
Swarm Node :8081
  ↓
Ingress / Routing Mesh
  ↓
Available service task :80
```

---

# 9. 🌉 Overlay Network

Create an overlay network:

```bash
docker network create \
  --driver overlay \
  app-network
```

Check:

```bash
docker network ls
```

Attach an existing service:

```bash
docker service update \
  --network-add app-network \
  web
```

### Bridge vs Overlay

```text
bridge
   ↓
primarily communication on one Docker host

overlay
   ↓
Swarm services across multiple nodes
```

Example:

```text
Node 1                      Node 2

frontend                    backend
   │                           │
   └────── overlay network ────┘
```

---

# 10. 📚 Docker Stack

A **stack** groups multiple Swarm services and related resources under one deployment.

Example:

```yaml
services:
  web:
    image: nginx:latest

    networks:
      - app-network

    deploy:
      replicas: 3

networks:
  app-network:
    driver: overlay
```

Deploy:

```bash
docker stack deploy -c stack.yaml myapp
```

Check stacks:

```bash
docker stack ls
```

Check services:

```bash
docker stack services myapp
```

Check tasks:

```bash
docker stack ps myapp
```

Swarm prefixes resources with the stack name:

```text
Stack:   myapp
Service: web

Result:
myapp_web
```

Network:

```text
app-network
     ↓
myapp_app-network
```

Remove stack:

```bash
docker stack rm myapp
```

---

# 11. ⚓ YAML Anchors

YAML anchors prevent repeating the same configuration.

Without anchors:

```yaml
service1:
  deploy:
    replicas: 2
    restart_policy:
      condition: on-failure

service2:
  deploy:
    replicas: 2
    restart_policy:
      condition: on-failure
```

Instead:

```yaml
x-deploy: &site-deploy
  replicas: 2

  restart_policy:
    condition: on-failure
```

Reuse:

```yaml
services:

  service1:
    deploy:
      <<: *site-deploy

  service2:
    deploy:
      <<: *site-deploy
```

Remember:

```text
&name  → define/save

*name  → reference/reuse

<<     → merge here
```

---

# 12. 📍 Placement Constraints

Placement decides **where a service is allowed to run**.

Example:

```yaml
deploy:
  placement:
    constraints:
      - node.role == manager
```

Meaning:

```text
Run this service only on Manager nodes
```

Other deployments may use node labels:

```yaml
constraints:
  - node.labels.type == application
```

Placement is useful when a workload must run only on specific nodes.

---

# 13. 🧮 Resource Limits

Example:

```yaml
deploy:
  resources:
    limits:
      cpus: "0.25"
      memory: 128M
```

Meaning:

```text
CPU limit    = 0.25 CPU
Memory limit = 128 MB
```

`0.25` CPU means approximately **25% of one CPU core**.

Do not randomly choose production limits.

Good process:

```text
Observe usage
     ↓
Check normal + peak usage
     ↓
Add reasonable headroom
     ↓
Set limits
     ↓
Monitor again
```

Useful command during investigation:

```bash
docker stats
```

---

# 14. 📊 Resource Reservations

Example:

```yaml
resources:

  reservations:
    memory: 256M

  limits:
    memory: 512M
```

Difference:

```text
Reservation
    ↓
Amount Swarm considers required
when scheduling the task

Limit
    ↓
Maximum resource the container
is allowed to consume
```

Example:

```text
reservation = 256 MB
limit       = 512 MB
```

Swarm considers the requested reservation when choosing a node.

---

# 15. 🔁 Restart Policy

Example:

```yaml
restart_policy:
  condition: on-failure
```

Meaning:

```text
Task fails
    ↓
Swarm attempts recovery
    ↓
Desired state restored
```

More detailed example:

```yaml
restart_policy:
  condition: on-failure
  delay: 3s
  window: 60s
```

---

# 16. 🔄 Rolling Updates

Example:

```yaml
update_config:
  parallelism: 1
  delay: 5s
  order: start-first
```

Meaning:

```text
parallelism: 1
→ update one task at a time

delay: 5s
→ wait 5 seconds between batches

order: start-first
→ start new task before stopping old task
```

Flow:

```text
Old replica
    ↓
Start new replica
    ↓
New replica running
    ↓
Stop old replica
    ↓
Wait 5 seconds
    ↓
Next replica
```

---

# 17. 🆕 Image Updates

Suppose current image:

```yaml
image: nginx:latest
```

Change to:

```yaml
image: nginx:1.28
```

Deploy again:

```bash
docker stack deploy -c stack.yaml myapp
```

Watch the rollout:

```bash
watch docker stack ps myapp
```

With:

```yaml
parallelism: 1
delay: 5s
```

Swarm replaces replicas gradually.

---

# 18. ⏪ Rollback

If the new deployment causes problems:

```bash
docker service rollback myapp_web
```

Flow:

```text
Old version
     ↓
Deploy new version
     ↓
Problem detected
     ↓
docker service rollback
     ↓
Previous service configuration restored
```

Check:

```bash
docker service ps myapp_web
```

Old tasks remain visible in task history as `Shutdown`.

---

# 19. 🔐 Swarm Secrets

Use secrets for sensitive data such as:

```text
Database passwords
API tokens
Private keys
Certificates
Credentials
```

Create a test secret:

```bash
printf 'test-password-123' | docker secret create db_password -
```

Explanation:

```text
printf
   ↓
produces the value

|
   ↓
passes it through stdin

docker secret create
   ↓
stores it as a Swarm secret
```

List secrets:

```bash
docker secret ls
```

Attach secret in stack:

```yaml
services:

  web:
    image: nginx:latest

    secrets:
      - db_password

secrets:

  db_password:
    external: true
```

`external: true` means:

> The secret already exists in Swarm.

Inside the container, the secret is normally available under:

```text
/run/secrets/
```

Example:

```bash
ls -l /run/secrets/
```

Read test secret:

```bash
cat /run/secrets/db_password
```

### `.env` vs Secret

```text
.env / environment variables
        ↓
normal runtime configuration

Swarm Secret
        ↓
sensitive information
```

Avoid putting real secrets directly in shell commands because they may be recorded in shell history.

---

# 20. 📄 Swarm Configs

Configs are similar to secrets but intended for **non-sensitive configuration**.

Create file:

```bash
echo 'environment=staging' > app.conf
```

Create config:

```bash
docker config create app_config app.conf
```

Check:

```bash
docker config ls
```

Remember:

```text
Secret
→ sensitive

Config
→ non-sensitive configuration
```

Docker gives every config an internal ID:

```text
NAME         ID
app_config   zkojft...
```

Normally we reference the **name**, not the long ID.

---

# 21. 🛠️ Node Drain & Maintenance

Suppose a Swarm node needs maintenance.

Drain it:

```bash
docker node update \
  --availability drain \
  <node_name>
```

Example:

```bash
docker node update \
  --availability drain \
  swarm02
```

Check:

```bash
docker node ls
```

Example:

```text
STATUS   AVAILABILITY
Ready    Drain
```

Important:

```text
Ready
→ node itself is healthy

Drain
→ don't schedule workload tasks here
```

In a multi-node cluster:

```text
Node drained
     ↓
Swarm tries to move tasks
     ↓
Other eligible Active nodes
```

After maintenance:

```bash
docker node update \
  --availability active \
  <node_name>
```

Drain state persists across a normal server restart until changed back to `Active`.

---

# 22. 🗺️ Replica Distribution

Example:

```yaml
placement:

  max_replicas_per_node: 1

  preferences:
    - spread: engine.labels.zone
```

## `max_replicas_per_node`

```yaml
max_replicas_per_node: 1
```

Means:

> Maximum one replica of this service on each node.

Example:

```text
Node 1 → replica 1
Node 2 → replica 2
Node 3 → replica 3
```

Useful for availability.

## Spread Preference

```yaml
preferences:
  - spread: engine.labels.zone
```

If nodes have labels:

```text
node1 → zone=A
node2 → zone=B
node3 → zone=C
```

Swarm prefers distributing tasks across the different zone label values.

Remember:

```text
max_replicas_per_node
→ spread across servers

spread: engine.labels.zone
→ prefer distribution across zones
```

---

# 23. 🚦 Traefik / Load Balancer Concept

Traefik is **not required by Docker Swarm**.

It is one possible reverse proxy/load-balancer design.

Example architecture:

```text
Browser
   ↓
app.example.com
   ↓
Traefik
   ↓
Swarm service
   ↓
Application :3000
```

Example labels:

```yaml
labels:
  - traefik.enable=true
  - traefik.http.routers.web.rule=Host(`app.example.com`)
  - traefik.http.services.web.loadbalancer.server.port=3000
```

Meaning:

```text
Host = app.example.com
          ↓
Traefik router
          ↓
web service
          ↓
port 3000
```

Traefik may access Docker/Swarm information through:

```yaml
volumes:
  - /var/run/docker.sock:/var/run/docker.sock:ro
```

Some stacks may instead use:

```text
Nginx
HAProxy
Custom LB image
Direct published ports
Other ingress solutions
```

So:

> **Traefik is a design choice, not a Swarm requirement.**

---

# 24. 🔧 Swarm Troubleshooting

When someone reports:

> "The Swarm application is down."

Do not immediately restart or redeploy.

Use this flow.

---

## Step 1 — Check services

```bash
docker service ls
```

Look at replicas:

```text
3/3 → healthy
1/3 → investigate
0/3 → investigate
```

---

## Step 2 — Check tasks

```bash
docker service ps <service_name>
```

Example:

```bash
docker service ps myapp_web
```

Look for:

```text
Running
Pending
Rejected
Failed
Shutdown
```

Also inspect the `ERROR` column.

---

## Step 3 — Check logs

```bash
docker service logs --tail 100 <service_name>
```

Example:

```bash
docker service logs --tail 100 myapp_web
```

---

## Step 4 — Check nodes

```bash
docker node ls
```

Look for:

```text
Ready / Active → good

Ready / Drain
→ healthy but unavailable for workload scheduling

Down
→ node problem
```

---

## Step 5 — Check service configuration

```bash
docker service inspect <service_name>
```

Example:

```bash
docker service inspect myapp_web
```

Useful for checking:

```text
Image
Networks
Secrets
Resource limits
Placement
Update configuration
Ports
```

---

## Quick Troubleshooting Flow

```text
Application reported down
          ↓
docker service ls
          ↓
Replica problem?
          ↓
docker service ps <service>
          ↓
Task failure / pending / rejected?
          ↓
docker service logs <service>
          ↓
docker node ls
          ↓
Inspect service/network/config if required
```

---

# 25. 🧩 Complete Sample `stack.yaml`

This example combines the major concepts covered in this guide.

```yaml
x-deploy: &default-deploy

  replicas: 3

  update_config:
    parallelism: 1
    delay: 5s
    order: start-first
    failure_action: rollback

  restart_policy:
    condition: on-failure
    delay: 3s

  resources:

    reservations:
      memory: 64M

    limits:
      cpus: "0.25"
      memory: 128M

  placement:
    constraints:
      - node.role == manager


services:

  web:

    image: nginx:1.28

    networks:
      - app-network

    secrets:
      - db_password

    configs:
      - source: app_config
        target: /etc/app/app.conf

    deploy:
      <<: *default-deploy


networks:

  app-network:
    driver: overlay


secrets:

  db_password:
    external: true


configs:

  app_config:
    external: true
```

> Note: `node.role == manager` is included here only to demonstrate placement constraints. Real applications should use placement rules based on their actual infrastructure requirements.

Before deploying this example, the external secret/config must already exist.

Example:

```bash
printf 'test-password-123' | docker secret create db_password -
```

```bash
echo 'environment=staging' > app.conf
docker config create app_config app.conf
```

Deploy:

```bash
docker stack deploy -c stack.yaml myapp
```

Verify:

```bash
docker stack services myapp
```

```bash
docker stack ps myapp
```

---

# 26. ⚡ Command Cheat Sheet

## Swarm

```bash
docker swarm init
docker info
```

---

## Nodes

```bash
docker node ls
```

```bash
docker swarm join-token worker
```

```bash
docker swarm join-token manager
```

Drain:

```bash
docker node update --availability drain <node>
```

Activate:

```bash
docker node update --availability active <node>
```

---

## Services

```bash
docker service ls
```

```bash
docker service create --name web nginx
```

```bash
docker service ps web
```

```bash
docker service inspect web
```

```bash
docker service logs --tail 100 web
```

```bash
docker service scale web=3
```

```bash
docker service rollback web
```

Remove:

```bash
docker service rm web
```

---

## Stack

Deploy:

```bash
docker stack deploy -c stack.yaml myapp
```

List:

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

## Networks

```bash
docker network ls
```

Create overlay:

```bash
docker network create --driver overlay app-network
```

Inspect:

```bash
docker network inspect app-network
```

---

## Secrets

Create:

```bash
printf 'dummy-value' | docker secret create my_secret -
```

List:

```bash
docker secret ls
```

Inspect metadata:

```bash
docker secret inspect my_secret
```

Remove when no service uses it:

```bash
docker secret rm my_secret
```

---

## Configs

Create:

```bash
docker config create app_config app.conf
```

List:

```bash
docker config ls
```

Inspect:

```bash
docker config inspect app_config
```

Remove when unused:

```bash
docker config rm app_config
```

---

## Containers Running Swarm Tasks

```bash
docker ps
```

Enter a task container on the current node:

```bash
docker exec -it <container_id> sh
```

Check mounted secrets:

```bash
ls -l /run/secrets/
```

---

# 🧠 Important Mental Model

```text
Dockerfile
    ↓
Build Image
    ↓
Registry
    ↓
Stack YAML
    ↓
Swarm Service
    ↓
Tasks
    ↓
Containers
```

And:

```text
Swarm Manager
     ↓
Desired State
     ↓
Scheduler
     ↓
Eligible Nodes
     ↓
Tasks / Containers
```

---

# 🚨 Production Troubleshooting Rule

Before changing anything:

```text
1. Observe
2. Check service
3. Check tasks
4. Check logs
5. Check nodes
6. Check configuration
7. Identify root cause
8. Then fix
```

Avoid jumping directly to:

```bash
docker service rm ...
docker stack rm ...
docker system prune ...
docker swarm leave --force
```

Those are **not first troubleshooting steps**.

---

# 🎯 Key Points to Remember

```text
Service     → desired application state

Task        → one scheduled instance of a service

Replica     → desired number of running tasks

Manager     → controls and schedules Swarm

Worker      → executes workloads

Overlay     → cross-node service networking

Routing Mesh
            → routes published-port traffic to service tasks

Stack       → group of Swarm services/resources

Placement   → controls where tasks may run

Reservation → scheduling requirement

Limit       → maximum resource usage

Secret      → sensitive information

Config      → non-sensitive configuration

Drain       → stop scheduling workloads on a node

Rolling Update
            → gradually replace old tasks with new tasks

Rollback    → return service to previous configuration
```

---

## 🏁 Final Swarm Troubleshooting Shortcut

When you see:

```text
REPLICAS 0/3
```

start here:

```bash
docker service ls
```

then:

```bash
docker service ps <service_name>
```

then:

```bash
docker service logs --tail 100 <service_name>
```

then:

```bash
docker node ls
```

**Understand the failure first. Fix second.**

---

> **Learning goal:** Don't memorize every Docker Swarm command. Understand `desired state → scheduling → tasks → containers → networking → recovery`, then use the right command to verify each layer.
