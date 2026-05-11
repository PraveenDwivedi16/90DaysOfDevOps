# Day 33 – Docker Compose: Multi-Container Basics
## Goal
Today's goal was to run multi-container applications using a single Docker Compose YAML file.

Docker Compose simplifies container management by defining containers, networks, volumes, ports, and environment variables in one file.

# Topics Covered

- What is Docker Compose?
- Why Docker Compose is used
- Docker Compose YAML structure
- Running Nginx with Compose
- Running WordPress and MySQL with Compose
- Compose default network
- Service names as DNS names
- Compose named volumes
- Docker Compose lifecycle commands
- Environment variables directly in Compose
- `.env` file with Compose
- `docker compose config`
- Basic, scenario-based, and advanced interview questions

# 1. What is Docker Compose?

Docker Compose is a tool used to define and run multi-container Docker applications.

Instead of manually running many commands like:
docker network create
docker volume create
docker run
docker logs

we define everything in:

docker-compose.yml

Then start the full stack using:

docker compose up -d

# 2. Why Docker Compose?

Without Compose, running WordPress and MySQL requires many commands:

docker network create app-net
docker volume create mysql-data

docker run -d --name mysql-db --network app-net \
  -e MYSQL_ROOT_PASSWORD=rootpass \
  -v mysql-data:/var/lib/mysql \
  mysql:8.0

docker run -d --name wordpress --network app-net \
  -p 8080:80 \
  -e WORDPRESS_DB_HOST=mysql-db \
  wordpress

With Compose, the same setup can be managed from a YAML file:

```yaml
services:
  db:
    image: mysql:8.0

  wordpress:
    image: wordpress:latest

Then:

docker compose up -d

Compose automatically creates:

- Containers
- Default network
- Named volumes
- DNS resolution between services
- Port mappings
- Environment variable configuration

# 3. Docker Compose Basic Concepts

## Service

A service is a container definition.

```yaml
services:
  web:
    image: nginx:alpine
```

Here `web` is a service.

---

## Image

The image used by a service.

```yaml
image: nginx:alpine
```

---

## Ports

Port mapping between host and container.

```yaml
ports:
  - "8080:80"
```

Meaning:

```text
Host port 8080 → Container port 80
```

---

## Environment Variables

Direct values:

```yaml
environment:
  MYSQL_ROOT_PASSWORD: rootpass
```

Using `.env`:

```yaml
environment:
  MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
```

---

## Volumes

Named volume:

```yaml
volumes:
  - db_data:/var/lib/mysql

volumes:
  db_data:
```

---

## Networks

Compose automatically creates a default network for the project.

Example:

```text
wordpress-compose_default
```

Services can communicate using service names.

Example:

```yaml
WORDPRESS_DB_HOST: db:3306
```

Here `db` is the MySQL service name.

---

# Task 1 – Install & Verify

## Command

```bash
docker compose version
```

## Output

```text
Docker Compose version v5.1.3
```

## Result

Docker Compose is installed and available using the modern Compose command:

```bash
docker compose
```

Old Compose command:

```bash
docker-compose
```

Modern Compose command:

```bash
docker compose
```

---

# Task 2 – Your First Compose File

## Goal

Create a simple Compose file that runs a single Nginx container with port mapping.

## Folder

```text
2026/day-33/compose-basics/
```

## docker-compose.yml

```yaml
services:
  web:
    image: nginx:alpine
    container_name: compose-nginx
    ports:
      - "8085:80"
```

## Commands

```bash
mkdir -p ~/2026/day-33/compose-basics
cd ~/2026/day-33/compose-basics
```

```bash
cat > docker-compose.yml <<'EOF'
services:
  web:
    image: nginx:alpine
    container_name: compose-nginx
    ports:
      - "8085:80"
EOF
```

Start:

```bash
docker compose up -d
```

Check:

```bash
docker compose ps
```

Access:

```bash
curl http://localhost:8085
```

Observed output included:

```html
<title>Welcome to nginx!</title>
<h1>Welcome to nginx!</h1>
```

Stop and remove:

```bash
docker compose down
```

Observed:

```text
✔ Container compose-nginx        Removed
✔ Network compose-basics_default Removed
```

## Result

Nginx successfully ran through Docker Compose and was accessible on port `8085`.

---

# Task 3 – WordPress + MySQL Compose Setup

## Goal

Run two containers:

- WordPress
- MySQL

Requirements:

- Same network
- MySQL named volume for persistence
- WordPress connects to MySQL using service name
- Access WordPress in browser/server
- Verify volume persists after `docker compose down`

## Folder

```text
2026/day-33/wordpress-compose/
```

## Initial docker-compose.yml

```yaml
services:
  db:
    image: mysql:8.0
    container_name: compose-mysql
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: rootpass
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wpuser
      MYSQL_PASSWORD: wppass
    volumes:
      - db_data:/var/lib/mysql

  wordpress:
    image: wordpress:latest
    container_name: compose-wordpress
    restart: unless-stopped
    depends_on:
      - db
    ports:
      - "8086:80"
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wpuser
      WORDPRESS_DB_PASSWORD: wppass
      WORDPRESS_DB_NAME: wordpress

volumes:
  db_data:
```

## Start Services

```bash
docker compose up -d
```

## Verify Services

```bash
docker compose ps
```

Observed:

```text
compose-mysql       mysql:8.0          db          Up
compose-wordpress   wordpress:latest   wordpress   Up   0.0.0.0:8086->80/tcp
```

## Verify WordPress

```bash
curl -I http://localhost:8086
```

Observed:

```text
HTTP/1.1 302 Found
Location: http://localhost:8086/wp-admin/install.php
```

This means WordPress is running and redirecting to the setup page.

## Verify Volume

```bash
docker volume ls
```

Observed:

```text
wordpress-compose_db_data
```

Compose created this volume from:

```yaml
volumes:
  db_data:
```

Compose prefixes the project name:

```text
db_data → wordpress-compose_db_data
```

## Persistence Test

```bash
docker compose down
docker compose up -d
docker compose ps
docker volume ls
```

Observed:

```text
✔ Container compose-wordpress       Removed
✔ Container compose-mysql           Removed
✔ Network wordpress-compose_default Removed

✔ Network wordpress-compose_default Created
✔ Container compose-mysql           Started
✔ Container compose-wordpress       Started
```

Volume still existed:

```text
local     wordpress-compose_db_data
```

## Important

`docker compose down` removes containers and network but keeps named volumes.

This command deletes volumes:

```bash
docker compose down -v
```

Do not use `down -v` if you want database data to stay.

---

# Task 4 – Compose Commands

## Start Services in Detached Mode

```bash
docker compose up -d
```

## View Running Services

```bash
docker compose ps
```

Observed:

```text
compose-mysql       Up
compose-wordpress   Up   0.0.0.0:8086->80/tcp
```

## View Logs of All Services

```bash
docker compose logs
docker compose logs --tail=20
docker compose logs -f
```

## View Logs of Specific Service

```bash
docker compose logs db
docker compose logs wordpress
docker compose logs wordpress --tail=20
docker compose logs -f wordpress
```

## Stop Services Without Removing

```bash
docker compose stop
```

Observed:

```text
✔ Container compose-wordpress Stopped
✔ Container compose-mysql     Stopped
```

This stops containers but keeps them.

## Start Stopped Services

```bash
docker compose start
```

Observed:

```text
✔ Container compose-mysql     Started
✔ Container compose-wordpress Started
```

## Remove Containers and Network

```bash
docker compose down
```

Observed:

```text
✔ Container compose-wordpress       Removed
✔ Container compose-mysql           Removed
✔ Network wordpress-compose_default Removed
```

Volume still existed:

```bash
docker volume ls | grep wordpress-compose_db_data
```

Observed:

```text
local     wordpress-compose_db_data
```

## Rebuild Images

```bash
docker compose up -d --build
```

Observed:

```text
✔ Network wordpress-compose_default Created
✔ Container compose-mysql           Started
✔ Container compose-wordpress       Started
```

In this WordPress setup, public images are used:

```yaml
image: mysql:8.0
image: wordpress:latest
```

So there is no custom Dockerfile to rebuild, but this command is important for services using:

```yaml
build: .
```

---

# Task 5 – Environment Variables

## Goal

Use environment variables:

1. Directly in `docker-compose.yml`
2. Using a `.env` file

---

## Direct Environment Variables

Initially, variables were written directly:

```yaml
environment:
  MYSQL_ROOT_PASSWORD: rootpass
  MYSQL_DATABASE: wordpress
  MYSQL_USER: wpuser
  MYSQL_PASSWORD: wppass
```

This works, but values are hardcoded.

Problems:

- Passwords are visible in Compose file
- Hard to manage different environments
- Risk of committing secrets

---

## .env File

Compose automatically reads `.env` from the same folder as `docker-compose.yml`.

## wordpress-compose/.env

```env
MYSQL_ROOT_PASSWORD=rootpass
MYSQL_DATABASE=wordpress
MYSQL_USER=wpuser
MYSQL_PASSWORD=wppass

WORDPRESS_DB_HOST=db:3306
WORDPRESS_DB_USER=wpuser
WORDPRESS_DB_PASSWORD=wppass
WORDPRESS_DB_NAME=wordpress
WORDPRESS_PORT=8086
```

---

## Updated docker-compose.yml With Variables

```yaml
services:
  db:
    image: mysql:8.0
    container_name: compose-mysql
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
    volumes:
      - db_data:/var/lib/mysql

  wordpress:
    image: wordpress:latest
    container_name: compose-wordpress
    restart: unless-stopped
    depends_on:
      - db
    ports:
      - "${WORDPRESS_PORT}:80"
    environment:
      WORDPRESS_DB_HOST: ${WORDPRESS_DB_HOST}
      WORDPRESS_DB_USER: ${WORDPRESS_DB_USER}
      WORDPRESS_DB_PASSWORD: ${WORDPRESS_DB_PASSWORD}
      WORDPRESS_DB_NAME: ${WORDPRESS_DB_NAME}

volumes:
  db_data:
```

---

## Verify Resolved Config

```bash
docker compose config
```

Observed output included:

```yaml
networks:
  default:
    name: wordpress-compose_default
volumes:
  db_data:
    name: wordpress-compose_db_data
```

This proves Compose resolved the final configuration.

---

## Restart Services

```bash
docker compose down
docker compose up -d
docker compose ps
```

Observed:

```text
compose-mysql       Up
compose-wordpress   Up   0.0.0.0:8086->80/tcp
```

## Verify WordPress

```bash
curl -I http://localhost:8086
```

Observed:

```text
HTTP/1.1 302 Found
Location: http://localhost:8086/wp-admin/install.php
```

## Verify Environment Inside WordPress

```bash
docker compose exec wordpress env | grep WORDPRESS_DB
```

Observed:

```text
WORDPRESS_DB_NAME=wordpress
WORDPRESS_DB_HOST=db:3306
WORDPRESS_DB_USER=wpuser
WORDPRESS_DB_PASSWORD=wppass
```

## Verify Environment Inside MySQL

```bash
docker compose exec db env | grep MYSQL
```

Observed:

```text
MYSQL_DATABASE=wordpress
MYSQL_USER=wpuser
MYSQL_PASSWORD=wppass
MYSQL_ROOT_PASSWORD=rootpass
MYSQL_MAJOR=8.0
MYSQL_VERSION=8.0.46-1.el9
MYSQL_SHELL_VERSION=8.0.46-1.el9
```

## Result

Environment variables from `.env` were successfully picked up by Docker Compose.

---

# All Compose Files Created

## 1. compose-basics/docker-compose.yml

```yaml
services:
  web:
    image: nginx:alpine
    container_name: compose-nginx
    ports:
      - "8085:80"
```

## 2. wordpress-compose/docker-compose.yml

```yaml
services:
  db:
    image: mysql:8.0
    container_name: compose-mysql
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
    volumes:
      - db_data:/var/lib/mysql

  wordpress:
    image: wordpress:latest
    container_name: compose-wordpress
    restart: unless-stopped
    depends_on:
      - db
    ports:
      - "${WORDPRESS_PORT}:80"
    environment:
      WORDPRESS_DB_HOST: ${WORDPRESS_DB_HOST}
      WORDPRESS_DB_USER: ${WORDPRESS_DB_USER}
      WORDPRESS_DB_PASSWORD: ${WORDPRESS_DB_PASSWORD}
      WORDPRESS_DB_NAME: ${WORDPRESS_DB_NAME}

volumes:
  db_data:
```

## 3. wordpress-compose/.env

```env
MYSQL_ROOT_PASSWORD=rootpass
MYSQL_DATABASE=wordpress
MYSQL_USER=wpuser
MYSQL_PASSWORD=wppass

WORDPRESS_DB_HOST=db:3306
WORDPRESS_DB_USER=wpuser
WORDPRESS_DB_PASSWORD=wppass
WORDPRESS_DB_NAME=wordpress
WORDPRESS_PORT=8086
```

---

# Important Commands Summary

## Version

```bash
docker compose version
```

## Start Services

```bash
docker compose up
docker compose up -d
```

## View Services

```bash
docker compose ps
```

## Logs

```bash
docker compose logs
docker compose logs -f
docker compose logs db
docker compose logs wordpress
docker compose logs wordpress --tail=20
```

## Stop Without Removing

```bash
docker compose stop
```

## Start Stopped Services

```bash
docker compose start
```

## Remove Containers and Network

```bash
docker compose down
```

## Remove Containers, Network, and Volumes

```bash
docker compose down -v
```

## Rebuild

```bash
docker compose up -d --build
docker compose build
```

## Show Final Resolved Config

```bash
docker compose config
```

## Execute Command Inside Service

```bash
docker compose exec wordpress env
docker compose exec db env
```

---

# Screenshots to Attach

Attach screenshots for:

1. `docker compose version`
2. `compose-basics/docker-compose.yml`
3. `docker compose up -d` for Nginx
4. `curl http://localhost:8085`
5. `docker compose down` for Nginx
6. WordPress `docker-compose.yml`
7. `docker compose ps` showing WordPress and MySQL
8. `curl -I http://localhost:8086`
9. `docker volume ls` showing `wordpress-compose_db_data`
10. `docker compose stop`
11. `docker compose start`
12. `docker compose down`
13. `docker compose up -d --build`
14. `.env` file
15. `docker compose config`
16. `docker compose exec wordpress env | grep WORDPRESS_DB`
17. `docker compose exec db env | grep MYSQL`

---

# Troubleshooting Notes

## 1. curl: Recv failure: Connection reset by peer

This can happen if WordPress or MySQL has just started and is not fully ready.

Fix:

```bash
docker compose logs wordpress --tail=30
docker compose logs db --tail=30
sleep 20
curl -I http://localhost:8086
```

---

## 2. Browser Cannot Access EC2 Port

If this works:

```bash
curl http://localhost:8086
```

but browser cannot open:

```text
http://EC2_PUBLIC_IP:8086
```

check AWS Security Group inbound rules.

Allow TCP port:

```text
8086
```

---

## 3. WordPress Cannot Connect to Database

Check logs:

```bash
docker compose logs wordpress
docker compose logs db
```

Verify:

```yaml
WORDPRESS_DB_HOST: db:3306
WORDPRESS_DB_USER: wpuser
WORDPRESS_DB_PASSWORD: wppass
WORDPRESS_DB_NAME: wordpress
```

Also verify the MySQL service name is:

```yaml
db:
```

---

## 4. Environment Variables Not Working

Run:

```bash
docker compose config
```

This shows the final resolved Compose file.

---

## 5. Data Lost After Down

Check whether this command was used:

```bash
docker compose down -v
```

`-v` removes volumes.

Normal command:

```bash
docker compose down
```

keeps named volumes.

---

# Practice Questions

## Basic Questions

1. What is Docker Compose?
2. What is a `docker-compose.yml` file?
3. What is a service in Docker Compose?
4. Which command starts Compose services?
5. What is the difference between `docker compose up` and `docker compose up -d`?
6. What does `docker compose ps` show?
7. What does `docker compose down` do?
8. Does `docker compose down` remove named volumes by default?
9. What command shows Compose logs?
10. What is the purpose of `.env` in Compose?

## Scenario-Based Questions

1. You have WordPress and MySQL containers. How can WordPress reach MySQL?
2. Your WordPress data disappeared after cleanup. What command may have caused it?
3. You changed `.env`, but Compose still uses old values. What should you run?
4. Your browser cannot access WordPress on EC2, but `curl localhost` works. What should you check?
5. You want to stop services without removing containers. Which command should you use?
6. You want to rebuild custom images before starting services. Which command should you use?
7. You want to see logs only for MySQL service. Which command should you use?
8. You want to verify variables from `.env` are picked up. Which command helps?

---

# Interview Questions and Answers

## Q1. What is Docker Compose?

Docker Compose is a tool used to define and run multi-container Docker applications using a YAML file. It allows us to manage services, networks, volumes, ports, and environment variables with one file and one command.

Example:

```bash
docker compose up -d
```

---

## Q2. What is the difference between Docker and Docker Compose?

Docker is used to build and run individual containers.

Docker Compose is used to define and run multi-container applications.

Example:

```bash
docker run nginx
```

runs one container.

```bash
docker compose up -d
```

can run multiple services like WordPress and MySQL together.

---

## Q3. What is a service in Docker Compose?

A service is a container definition inside a Compose file.

Example:

```yaml
services:
  web:
    image: nginx
```

Here `web` is a service.

---

## Q4. What network does Compose create?

Docker Compose automatically creates a default network for the project.

Example:

```text
wordpress-compose_default
```

All services in the same Compose file join this network by default.

---

## Q5. How do services communicate in Docker Compose?

Services communicate using service names as DNS names.

Example:

```yaml
services:
  db:
    image: mysql:8.0

  wordpress:
    image: wordpress
    environment:
      WORDPRESS_DB_HOST: db:3306
```

WordPress connects to MySQL using hostname:

```text
db
```

---

## Q6. Why does WordPress use `db:3306` instead of an IP address?

Because `db` is the Compose service name and Docker Compose provides DNS resolution on its default network.

Using service names is better because container IPs can change, but service names remain stable.

---

## Q7. What does `depends_on` do?

`depends_on` controls service startup order.

Example:

```yaml
depends_on:
  - db
```

This starts `db` before WordPress.

Important:

> `depends_on` does not always mean the database is fully ready. It only controls startup order.

---

## Q8. What is a Compose named volume?

A Compose named volume stores persistent data outside containers.

Example:

```yaml
services:
  db:
    volumes:
      - db_data:/var/lib/mysql

volumes:
  db_data:
```

Compose creates a volume like:

```text
wordpress-compose_db_data
```

---

## Q9. Does `docker compose down` remove volumes?

No. By default, `docker compose down` removes containers and networks but keeps named volumes.

To remove volumes too:

```bash
docker compose down -v
```

---

## Q10. What is the difference between `docker compose stop` and `docker compose down`?

`docker compose stop` stops containers but keeps them.

`docker compose down` stops and removes containers and the Compose network.

Named volumes are kept unless `-v` is used.

---

## Q11. What is the purpose of `.env` in Docker Compose?

`.env` stores environment variables that Compose can use inside `docker-compose.yml`.

Example `.env`:

```env
MYSQL_PASSWORD=wppass
```

Compose file:

```yaml
MYSQL_PASSWORD: ${MYSQL_PASSWORD}
```

---

## Q12. How do you verify `.env` variables are being used?

Use:

```bash
docker compose config
```

This shows the final resolved Compose configuration.

You can also check inside the container:

```bash
docker compose exec wordpress env | grep WORDPRESS_DB
```

---

## Q13. How do you view logs of all services?

Use:

```bash
docker compose logs
```

For live logs:

```bash
docker compose logs -f
```

---

## Q14. How do you view logs of a specific service?

Use:

```bash
docker compose logs service-name
```

Example:

```bash
docker compose logs wordpress
docker compose logs db
```

---

## Q15. How do you rebuild images with Compose?

Use:

```bash
docker compose up -d --build
```

or:

```bash
docker compose build
docker compose up -d
```

This is useful when services use:

```yaml
build: .
```

---

# Advanced Interview Questions and Answers

## Q16. Why should we avoid hardcoding passwords in docker-compose.yml?

Hardcoded passwords are visible to anyone who can read the Compose file. They may be committed to Git by mistake.

Better options:

- `.env` file for local development
- CI/CD secret variables
- Docker secrets
- Kubernetes secrets
- Cloud secret managers

---

## Q17. What is the risk of using `container_name` in Compose?

`container_name` gives a fixed container name, which can be useful for learning. But in larger projects or scaling scenarios, it can cause conflicts because Compose cannot create multiple containers with the same fixed name.

For production-like Compose, service names are usually enough.

---

## Q18. What happens if you run `docker compose up -d` after `docker compose down`?

Compose recreates the containers and network. Named volumes remain, so database data persists if volumes were not removed.

---

## Q19. What happens if you run `docker compose down -v`?

Compose removes containers, networks, and named volumes.

This deletes persistent database data stored in Compose volumes.

---

## Q20. How does Compose know which `.env` file to use?

By default, Compose reads the `.env` file from the same directory where the Compose command is executed and where the Compose file is located.

---

# Real-World DevOps Use Cases

## 1. Local Development Stack

A project may have:

```yaml
services:
  frontend:
    build: ./frontend

  backend:
    build: ./backend

  db:
    image: mysql:8.0

  redis:
    image: redis:alpine
```

Start all:

```bash
docker compose up -d
```

---

## 2. WordPress Local Setup

```yaml
services:
  db:
    image: mysql:8.0
    volumes:
      - db_data:/var/lib/mysql

  wordpress:
    image: wordpress:latest
    ports:
      - "8086:80"
    environment:
      WORDPRESS_DB_HOST: db:3306

volumes:
  db_data:
```

---

## 3. CI/CD Testing

Compose can start dependencies for tests:

```bash
docker compose up -d db redis
npm test
docker compose down

# Final Revision Notes

## Key Points

- Docker Compose manages multi-container applications.
- Compose uses `docker-compose.yml`.
- A service is a container definition.
- Compose automatically creates a default network.
- Services communicate using service names.
- Named volumes persist data.
- `docker compose up -d` starts services in background.
- `docker compose ps` shows services.
- `docker compose logs` shows logs.
- `docker compose stop` stops without removing.
- `docker compose down` removes containers and network.
- `docker compose down -v` removes volumes too.
- `.env` stores variables for Compose.
- `docker compose config` shows final resolved configuration.
- WordPress connects to MySQL using service name `db`.
