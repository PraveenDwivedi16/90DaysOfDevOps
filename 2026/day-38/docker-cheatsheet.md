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
