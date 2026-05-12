# Docker Cheat Sheet

A short job-ready Docker reference for daily use.

---

# Container Commands

| Command | Use |
|---|---|
| `docker run hello-world` | Run a test container from Docker Hub |
| `docker run -it ubuntu bash` | Run Ubuntu container in interactive mode |
| `docker run -d nginx` | Run container in detached/background mode |
| `docker run --name my-nginx nginx` | Run container with custom name |
| `docker run -d -p 8080:80 nginx` | Map host port 8080 to container port 80 |
| `docker ps` | List running containers |
| `docker ps -a` | List all containers, including stopped ones |
| `docker stop container_name` | Stop a running container gracefully |
| `docker kill container_name` | Force kill a running container |
| `docker start container_name` | Start a stopped container |
| `docker restart container_name` | Restart a container |
| `docker rm container_name` | Remove a stopped container |
| `docker rm -f container_name` | Force remove running/stopped container |
| `docker exec -it container_name sh` | Enter a running container shell |
| `docker exec container_name command` | Run one command inside a running container |
| `docker logs container_name` | View container logs |
| `docker logs -f container_name` | Follow live logs |
| `docker inspect container_name` | View full container details |
| `docker inspect -f '{{.State.Status}}' container_name` | Show container status only |
| `docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' container_name` | Show container IP address |

---

# Image Commands

| Command | Use |
|---|---|
| `docker images` | List local images |
| `docker image ls` | List local images |
| `docker pull nginx` | Pull image from Docker Hub |
| `docker build -t my-image:v1 .` | Build image from Dockerfile |
| `docker build -f Dockerfile.prod -t my-image:v1 .` | Build using specific Dockerfile |
| `docker run my-image:v1` | Run container from image |
| `docker image inspect image_name` | Inspect image details |
| `docker image history image_name` | Show image layers |
| `docker tag local-image:v1 username/repo:v1` | Tag image for Docker Hub |
| `docker push username/repo:v1` | Push image to Docker Hub |
| `docker pull username/repo:v1` | Pull image from Docker Hub |
| `docker rmi image_name` | Remove local image |
| `docker rmi -f image_name` | Force remove local image |

---

# Volume Commands

| Command | Use |
|---|---|
| `docker volume create my-volume` | Create named volume |
| `docker volume ls` | List volumes |
| `docker volume inspect my-volume` | Inspect volume details |
| `docker volume rm my-volume` | Remove volume |
| `docker volume prune` | Remove unused volumes |
| `docker run -v my-volume:/data alpine` | Mount named volume |
| `docker run -v /host/path:/container/path nginx` | Bind mount host folder |
| `docker run -v $(pwd):/app alpine` | Bind mount current folder |

---

# Network Commands

| Command | Use |
|---|---|
| `docker network ls` | List Docker networks |
| `docker network create my-net` | Create custom bridge network |
| `docker network inspect my-net` | Inspect network details |
| `docker network rm my-net` | Remove network |
| `docker network connect my-net container_name` | Connect existing container to network |
| `docker network disconnect my-net container_name` | Disconnect container from network |
| `docker run --network my-net --name app alpine` | Run container on custom network |
| `docker exec app1 ping app2` | Test name-based communication |
| `docker exec app1 ping 172.18.0.3` | Test IP-based communication |

---

# Docker Compose Commands

| Command | Use |
|---|---|
| `docker compose version` | Check Compose version |
| `docker compose up` | Start services in foreground |
| `docker compose up -d` | Start services in detached mode |
| `docker compose up -d --build` | Rebuild and start services |
| `docker compose ps` | List Compose services |
| `docker compose logs` | View logs of all services |
| `docker compose logs -f` | Follow logs of all services |
| `docker compose logs service_name` | View logs for one service |
| `docker compose stop` | Stop services without removing |
| `docker compose start` | Start stopped services |
| `docker compose restart` | Restart services |
| `docker compose down` | Remove containers and default network |
| `docker compose down -v` | Remove containers, network, and volumes |
| `docker compose build` | Build Compose images |
| `docker compose config` | Show final resolved Compose config |
| `docker compose exec service_name sh` | Enter a Compose service container |
| `docker compose -f file.yml up -d` | Run Compose with specific file |
| `docker compose up -d --scale web=3` | Scale a service to 3 replicas |

---

# Cleanup Commands

| Command | Use |
|---|---|
| `docker system df` | Show Docker disk usage |
| `docker container prune` | Remove stopped containers |
| `docker image prune` | Remove dangling images |
| `docker image prune -a` | Remove unused images |
| `docker volume prune` | Remove unused volumes |
| `docker network prune` | Remove unused networks |
| `docker system prune` | Remove unused containers, networks, images |
| `docker system prune -a` | Remove all unused images also |
| `docker system prune -a --volumes` | Remove unused images, containers, networks, and volumes |
| `docker rm -f $(docker ps -aq)` | Remove all containers |
| `docker rmi -f $(docker images -q)` | Remove all images |
| `docker stop $(docker ps -q)` | Stop all running containers |

Use cleanup commands carefully, especially commands that remove volumes.

---

# Dockerfile Instructions

| Instruction | Use |
|---|---|
| `FROM image:tag` | Set base image |
| `WORKDIR /app` | Set working directory |
| `COPY source destination` | Copy files from host to image |
| `ADD source destination` | Copy files, also supports URL/tar auto-extract |
| `RUN command` | Run command during image build |
| `EXPOSE 80` | Document container port |
| `ENV KEY=value` | Set environment variable |
| `ARG NAME=value` | Set build-time variable |
| `CMD ["command"]` | Default container command |
| `ENTRYPOINT ["command"]` | Fixed executable for container |
| `USER appuser` | Run as non-root user |
| `LABEL key=value` | Add metadata |
| `HEALTHCHECK CMD command` | Add container healthcheck |

---

# Dockerfile Examples

## Basic Dockerfile

```dockerfile
FROM alpine:3.20
WORKDIR /app
COPY app.txt .
CMD ["cat", "app.txt"]
```

## Python Flask Dockerfile

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY app.py .
EXPOSE 5000
CMD ["python", "app.py"]
```

## Multi-Stage Go Dockerfile

```dockerfile
FROM golang:1.22-alpine AS builder
WORKDIR /app
COPY main.go .
RUN go build -o app main.go

FROM alpine:3.20
WORKDIR /app
COPY --from=builder /app/app .
EXPOSE 8080
CMD ["./app"]
```

## Non-Root User Example

```dockerfile
FROM alpine:3.20
WORKDIR /app
RUN adduser -D appuser
COPY app .
RUN chown appuser:appuser /app/app
USER appuser
CMD ["./app"]
```

---

# Docker Compose Examples

## Nginx Compose

```yaml
services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
```

## App + Postgres Compose

```yaml
services:
  app:
    build: ./app
    ports:
      - "5000:5000"
    environment:
      DB_HOST: db
    depends_on:
      db:
        condition: service_healthy
    networks:
      - app-net

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: appdb
      POSTGRES_USER: appuser
      POSTGRES_PASSWORD: apppass
    volumes:
      - db-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U appuser -d appdb"]
      interval: 5s
      timeout: 5s
      retries: 5
    networks:
      - app-net

networks:
  app-net:

volumes:
  db-data:
```

---

# Port Mapping

| Syntax | Meaning |
|---|---|
| `-p 8080:80` | Host port 8080 maps to container port 80 |
| `-p 5000:5000` | Host port 5000 maps to container port 5000 |
| `ports: ["8080:80"]` | Compose port mapping |
| `expose: ["5000"]` | Expose port only to internal Docker network |

---

# CMD vs ENTRYPOINT

| Feature | CMD | ENTRYPOINT |
|---|---|---|
| Purpose | Default command | Fixed executable |
| Override | Easily overridden | Not easily overridden |
| Docker run args | Replace CMD | Passed as args |
| Use case | Default app command | CLI-style container |

Example CMD:

```dockerfile
CMD ["echo", "hello"]
```

Example ENTRYPOINT:

```dockerfile
ENTRYPOINT ["echo"]
```

---

# Named Volume vs Bind Mount

| Feature | Named Volume | Bind Mount |
|---|---|---|
| Managed by | Docker | Host/user |
| Path | Docker chooses | User provides |
| Best for | Database persistence | Local development |
| Example | `mysql-data:/var/lib/mysql` | `/root/site:/usr/share/nginx/html` |

---

# Default Bridge vs Custom Network

| Feature | Default Bridge | Custom Bridge |
|---|---|---|
| Created by | Docker | User/Compose |
| Name-based DNS | Limited | Supported |
| IP communication | Yes | Yes |
| Recommended for app stacks | No | Yes |

---

# Docker Hub Commands

| Command | Use |
|---|---|
| `docker login` | Login to Docker Hub |
| `docker tag app:v1 username/app:v1` | Tag image |
| `docker push username/app:v1` | Push image |
| `docker pull username/app:v1` | Pull image |
| `docker logout` | Logout |

---

# Quick Debug Commands

| Problem | Command |
|---|---|
| Container not running | `docker ps -a` |
| App error | `docker logs container_name` |
| Compose app error | `docker compose logs service_name` |
| Port issue | `docker ps` |
| Network issue | `docker network inspect network_name` |
| Volume issue | `docker volume inspect volume_name` |
| Env issue in Compose | `docker compose config` |
| Disk full | `docker system df` |
| Healthcheck issue | `docker inspect container --format '{{json .State.Health}}'` |

---

# Job-Ready Golden Rules

1. Use specific image tags, not only `latest`.
2. Use `.dockerignore`.
3. Use non-root users where possible.
4. Use volumes for persistent data.
5. Use custom networks for app stacks.
6. Use Compose for multi-container local setup.
7. Use healthchecks for databases.
8. Use multi-stage builds for smaller images.
9. Do not put secrets inside images.
10. Tag and push images properly to Docker Hub.


day-37-revision.md
# Day 37 – Docker Revision & Self-Check

## Goal

Day 37 is a revision day.

The goal is to consolidate everything learned from Days 29–36 so Docker becomes practical and job-ready.

Covered topics:

```text
Day 29: Docker basics
Day 30: Images and container lifecycle
Day 31: Dockerfile
Day 32: Volumes and networking
Day 33: Docker Compose basics
Day 34: Advanced Compose
Day 35: Multi-stage builds and Docker Hub
Day 36: Full Docker project
```

---

# Self-Assessment Checklist

Mark each item honestly:

```text
can do
shaky
haven't done
```

| Skill | Status | Notes |
|---|---|---|
| Run a container from Docker Hub | can do | `docker run hello-world`, `docker run nginx` |
| Run container in interactive mode | can do | `docker run -it ubuntu bash` |
| Run container in detached mode | can do | `docker run -d nginx` |
| List running containers | can do | `docker ps` |
| List all containers | can do | `docker ps -a` |
| Stop containers | can do | `docker stop name` |
| Remove containers | can do | `docker rm name` |
| Remove images | can do | `docker rmi image` |
| Explain image layers | can do | Dockerfile instructions create reusable layers |
| Explain Docker build cache | can do | Unchanged layers are reused |
| Write Dockerfile from scratch | can do | FROM, RUN, COPY, WORKDIR, CMD |
| Explain CMD vs ENTRYPOINT | can do | CMD default command, ENTRYPOINT fixed executable |
| Build custom image | can do | `docker build -t image:v1 .` |
| Tag custom image | can do | `docker tag local user/repo:v1` |
| Create named volumes | can do | `docker volume create` |
| Use named volumes | can do | `-v volume:/path` |
| Use bind mounts | can do | `-v /host/path:/container/path` |
| Create custom networks | can do | `docker network create my-net` |
| Connect containers by name | can do | Works on custom bridge networks |
| Write multi-container Compose file | can do | App + DB + cache |
| Use `.env` in Compose | can do | `${VARIABLE}` syntax |
| Write multi-stage Dockerfile | can do | builder stage + runtime stage |
| Push image to Docker Hub | shaky | Depends on login/token access |
| Use healthchecks | can do | `pg_isready`, `redis-cli ping` |
| Use `depends_on` with health condition | can do | `condition: service_healthy` |

---

# Weak Spots to Revisit

Pick 2 topics you feel shaky about.

Suggested weak topics:

## Weak Spot 1: Docker Hub Push Flow

Redo:

```bash
docker login
docker tag local-image:v1 username/repo:v1
docker push username/repo:v1
docker pull username/repo:v1
```

Checklist:

```text
Can login
Can tag correctly
Can push
Can see image on Docker Hub
Can remove local image
Can pull and run again
```

---

## Weak Spot 2: Multi-Stage Builds

Redo:

```bash
docker build -f Dockerfile.single -t app-single:v1 .
docker build -f Dockerfile.multistage -t app-multi:v1 .
docker images
```

Checklist:

```text
Can explain builder stage
Can explain runtime stage
Can compare size
Can explain why multi-stage is smaller
```

---

# Quick-Fire Questions and Answers

## Q1. What is the difference between an image and a container?

An image is a read-only template/package that contains app code, runtime, dependencies, and configuration.

A container is a running or stopped instance of an image.

Simple:

```text
Image = blueprint
Container = running instance
```

Example:

```bash
docker pull nginx
docker run nginx
```

`nginx` image is used to create an Nginx container.

---

## Q2. What happens to data inside a container when you remove it?

Data stored inside the container writable layer is deleted when the container is removed.

If data must persist, use:

```text
Named volume
Bind mount
External database/storage
```

Example:

```bash
docker run -v mysql-data:/var/lib/mysql mysql:8.0
```

---

## Q3. How do two containers on the same custom network communicate?

They communicate using container name or service name as DNS name.

Example:

```bash
docker network create my-net
docker run -d --name app --network my-net alpine sleep 1000
docker run -d --name db --network my-net alpine sleep 1000
docker exec app ping db
```

In Compose:

```yaml
WORDPRESS_DB_HOST: db:3306
```

Here `db` is the service name.

---

## Q4. What does `docker compose down -v` do differently from `docker compose down`?

`docker compose down` removes:

```text
containers
default network
```

It keeps named volumes.

`docker compose down -v` removes:

```text
containers
default network
named volumes
```

So `down -v` can delete database data.

---

## Q5. Why are multi-stage builds useful?

Multi-stage builds make images smaller and more secure.

Builder stage contains:

```text
compiler
build tools
source code
dependencies
```

Runtime stage contains only:

```text
built artifact
minimal runtime
```

This reduces image size and attack surface.

---

## Q6. What is the difference between COPY and ADD?

`COPY` copies files from host to image.

`ADD` can also:

```text
Download files from URLs
Automatically extract local tar archives
```

Best practice:

> Use `COPY` unless you specifically need `ADD` features.

Example:

```dockerfile
COPY app.py /app/app.py
```

---

## Q7. What does `-p 8080:80` mean?

It maps host port 8080 to container port 80.

```text
host:container
8080:80
```

So browser accesses:

```text
http://localhost:8080
```

and traffic goes to container port:

```text
80
```

---

## Q8. How do you check how much disk space Docker is using?

Use:

```bash
docker system df
```

Detailed view:

```bash
docker system df -v
```

---

# Days 29–36 Full Revision

## Day 29 – Introduction to Docker

Key concepts:

```text
Container
Image
Docker Hub
Docker client
Docker daemon
Registry
```

Important commands:

```bash
docker run hello-world
docker ps
docker ps -a
docker images
docker run -d --name my-nginx -p 8080:80 nginx
docker logs my-nginx
docker exec -it my-nginx sh
```

Important learning:

```text
Docker client talks to Docker daemon.
Daemon pulls image from Docker Hub.
Daemon creates and runs container from image.
Container can run in foreground, detached, or interactive mode.
```

---

## Day 30 – Images and Container Lifecycle

Key concepts:

```text
Image layers
Build cache
Container lifecycle
Created
Running
Paused
Stopped
Killed
Removed
```

Commands:

```bash
docker pull nginx
docker pull ubuntu
docker pull alpine
docker images
docker image inspect nginx
docker image history nginx
docker create --name lifecycle-nginx nginx
docker start lifecycle-nginx
docker pause lifecycle-nginx
docker unpause lifecycle-nginx
docker stop lifecycle-nginx
docker restart lifecycle-nginx
docker kill lifecycle-nginx
docker rm lifecycle-nginx
```

Important learning:

```text
Images are made of layers.
Layers are reused by Docker.
Containers are instances of images.
Containers can be created without starting.
```

---

## Day 31 – Dockerfile

Key concepts:

```text
Dockerfile
FROM
RUN
COPY
WORKDIR
EXPOSE
CMD
ENTRYPOINT
.dockerignore
Build cache
Layer order
```

Commands:

```bash
docker build -t my-ubuntu:v1 .
docker run my-ubuntu:v1
docker build -f Dockerfile -t app:v1 .
```

CMD vs ENTRYPOINT:

```text
CMD = default command, easily overridden
ENTRYPOINT = fixed executable, arguments appended
```

Build optimization:

```text
Copy dependency files first.
Install dependencies.
Copy source code later.
```

Example:

```dockerfile
FROM node:20
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
CMD ["npm", "start"]
```

---

## Day 32 – Volumes and Networking

Key concepts:

```text
Container data is ephemeral
Named volumes persist data
Bind mounts map host folder
Default bridge network
Custom bridge network
Container name DNS
```

Commands:

```bash
docker volume create mysql-data
docker volume ls
docker volume inspect mysql-data
docker run -v mysql-data:/var/lib/mysql mysql:8.0
docker run -v /root/site:/usr/share/nginx/html nginx
docker network ls
docker network create my-app-net
docker run --network my-app-net --name app1 alpine sleep 1000
docker exec app1 ping app2
```

Important learning:

```text
Default bridge supports IP communication.
Custom bridge supports name-based communication.
Use volumes for database persistence.
Use bind mounts for local development.
```

---

## Day 33 – Docker Compose Basics

Key concepts:

```text
docker-compose.yml
services
ports
environment
volumes
default network
service name DNS
.env file
```

Commands:

```bash
docker compose version
docker compose up -d
docker compose ps
docker compose logs
docker compose logs service
docker compose stop
docker compose start
docker compose down
docker compose down -v
docker compose config
docker compose up -d --build
```

Important learning:

```text
Compose runs multi-container apps.
Compose creates a default network.
Service names become DNS names.
.env variables can be referenced with ${VAR_NAME}.
```

---

## Day 34 – Advanced Compose

Key concepts:

```text
App + DB + cache
Custom Dockerfile in Compose
Healthchecks
depends_on with service_healthy
Restart policies
Named networks
Named volumes
Labels
Scaling
```

Commands:

```bash
docker compose up -d --build
docker inspect --format='{{json .State.Health}}' container
docker kill container
docker inspect -f '{{.RestartCount}}' container
docker compose up -d --scale web=3
```

Important learning:

```text
depends_on alone does not mean service is ready.
Healthchecks verify readiness.
Scaling fails with fixed host port mappings.
Use load balancer/reverse proxy for scaling.
```

---

## Day 35 – Multi-Stage Builds and Docker Hub

Key concepts:

```text
Single-stage image
Multi-stage image
Builder stage
Runtime stage
Docker Hub
Image tags
Push and pull
Non-root user
Minimal image
```

Commands:

```bash
docker build -f Dockerfile.single -t app-single:v1 .
docker build -f Dockerfile.multistage -t app-multi:v1 .
docker images
docker login
docker tag app-multi:v1 username/app:v1
docker push username/app:v1
docker pull username/app:v1
```

Important learning:

```text
Multi-stage builds reduce image size.
Final image should not contain compiler/build tools.
Use specific tags.
Avoid running as root.
```

---

## Day 36 – Full Docker Project

Key concepts:

```text
End-to-end Dockerization
App Dockerfile
Compose file
Database
Volume
Network
Healthcheck
.env
Docker Hub image
README
Fresh pull test
```

Commands:

```bash
docker build -t day36-flask-app:v1 ./app
docker compose up -d --build
docker compose ps
curl http://localhost:5010
docker tag day36-flask-app:v1 username/day36-flask-app:v1
docker push username/day36-flask-app:v1
docker compose -f docker-compose.hub.yml up -d
```

Important learning:

```text
A real Docker project includes app code, Dockerfile, Compose, DB, volume, network, env, registry, and README.
```

---

# Redo Hands-On Task 1 – Volumes

## Goal

Confirm database data persists using named volume.

Commands:

```bash
docker rm -f mysql-test
docker volume create mysql-test-data

docker run -d --name mysql-test \
  -e MYSQL_ROOT_PASSWORD=rootpass \
  -e MYSQL_DATABASE=demo \
  -v mysql-test-data:/var/lib/mysql \
  mysql:8.0
```

Enter DB:

```bash
docker exec -it mysql-test mysql -uroot -prootpass demo
```

SQL:

```sql
CREATE TABLE students (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(100));
INSERT INTO students (name) VALUES ('Docker Revision');
SELECT * FROM students;
exit;
```

Remove container:

```bash
docker rm -f mysql-test
```

Run new container with same volume:

```bash
docker run -d --name mysql-test \
  -e MYSQL_ROOT_PASSWORD=rootpass \
  -e MYSQL_DATABASE=demo \
  -v mysql-test-data:/var/lib/mysql \
  mysql:8.0
```

Check data:

```bash
docker exec -it mysql-test mysql -uroot -prootpass demo -e "SELECT * FROM students;"
```

Expected:

```text
Docker Revision
```

---

# Redo Hands-On Task 2 – Multi-Stage Build

## Goal

Compare image sizes.

Commands:

```bash
mkdir -p ~/docker-revision-go
cd ~/docker-revision-go
```

Create app:

```bash
cat > main.go <<'EOF'
package main

import "fmt"

func main() {
    fmt.Println("Docker revision multi-stage build")
}
EOF
```

Single-stage:

```bash
cat > Dockerfile.single <<'EOF'
FROM golang:1.22
WORKDIR /app
COPY main.go .
RUN go build -o app main.go
CMD ["./app"]
EOF
```

Multi-stage:

```bash
cat > Dockerfile.multi <<'EOF'
FROM golang:1.22-alpine AS builder
WORKDIR /app
COPY main.go .
RUN go build -o app main.go

FROM alpine:3.20
WORKDIR /app
COPY --from=builder /app/app .
CMD ["./app"]
EOF
```

Build:

```bash
docker build -f Dockerfile.single -t revision-single:v1 .
docker build -f Dockerfile.multi -t revision-multi:v1 .
docker images | grep revision
```

Expected:

```text
revision-multi is much smaller than revision-single
```

---

# Common Mistakes and Fixes

## Mistake 1: Running Docker command inside database shell

Wrong:

```sql
mysql> docker ps
```

Correct:

```bash
root@server:~# docker ps
```

---

## Mistake 2: Forgetting database name in MySQL

Error:

```text
ERROR 1046 (3D000): No database selected
```

Fix:

```sql
USE demo;
```

or:

```bash
mysql -uroot -prootpass demo
```

---

## Mistake 3: App starts before DB

Fix:

```yaml
depends_on:
  db:
    condition: service_healthy
```

Add DB healthcheck.

---

## Mistake 4: Data lost after Compose down

Check if you used:

```bash
docker compose down -v
```

This removes volumes.

---

## Mistake 5: Scaling fails

Error:

```text
port is already allocated
```

Reason:

```yaml
ports:
  - "5000:5000"
```

Only one container can bind host port 5000.

Fix:

```text
Use reverse proxy/load balancer
Use expose instead of ports for replicas
```

---

# Final Self-Check Answers

## Can I explain Docker in simple words?

Yes.

Docker packages an application with its dependencies into a container so it can run consistently across machines.

---

## Can I explain the flow from Dockerfile to container?

Yes.

```text
Dockerfile → docker build → image → docker run → container
```

---

## Can I explain persistent data?

Yes.

Container data is temporary unless stored in a volume or bind mount.

---

## Can I explain container communication?

Yes.

Containers on a custom Docker network can communicate using container names.

---

## Can I write Compose for app + DB?

Yes.

Example:

```yaml
services:
  app:
    build: ./app
    depends_on:
      db:
        condition: service_healthy

  db:
    image: postgres:16-alpine
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:
```

---

## Can I explain multi-stage builds?

Yes.

Build tools stay in builder stage. Final stage gets only the built artifact.

---

# Interview-Ready Summary

Docker is used to package and run applications in containers. 
An image is a read-only template, and a container is a running instance of that image.
Dockerfiles define how images are built. Volumes persist data beyond the container lifecycle.
Custom networks allow containers to communicate using names. Docker Compose manages multi-container applications. 
Multi-stage builds reduce image size and improve security. Docker Hub is used to distribute images.
