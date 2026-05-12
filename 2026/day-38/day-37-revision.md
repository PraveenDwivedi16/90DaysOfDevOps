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
An image is a read-only template, and a container is a running instance of that image. Dockerfiles define how images are built.
Volumes persist data beyond the container lifecycle. Custom networks allow containers to communicate using names. 
Docker Compose manages multi-container applications. Multi-stage builds reduce image size and improve security. Docker Hub is used to distribute images.
