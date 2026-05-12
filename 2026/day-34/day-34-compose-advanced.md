# Day 34 - Docker Compose: Real-World Multi-Container Apps

## Goal

Today's goal was to build a production-like Docker Compose setup with a custom web app, database, cache, healthchecks, restart policies, custom networks, named volumes, labels, and scaling tests.

Yesterday we used Compose for basics. Today we built a real multi-container stack:

```text
Flask Web App -> Postgres Database
Flask Web App -> Redis Cache
```

Final services:

| Service | Image/Build | Purpose |
|---|---|---|
| `web` | Built from `./app/Dockerfile` | Python Flask app |
| `db` | `postgres:16-alpine` | Database |
| `redis` | `redis:7-alpine` | Cache |

---

# Final Architecture

```text
curl / browser
     |
     v
Flask web container: day34-web
     |                       |
     v                       v
Postgres container       Redis container
day34-db                day34-redis
```

The Flask app connects to services using Compose DNS names:

```text
DB_HOST=db
REDIS_HOST=redis
```

This works because all services are attached to the same Docker Compose network.

---

# Important Concepts

## 1. Real-world multi-container app

A real application commonly uses multiple containers:

```text
app + database + cache + queue + worker + reverse proxy
```

For this task, we used:

```text
Flask + Postgres + Redis
```

## 2. `build:` in Compose

`build:` tells Compose to build a custom image from a Dockerfile.

```yaml
web:
  build: ./app
```

This means Compose reads:

```text
./app/Dockerfile
```

and builds the Flask app image.

## 3. `depends_on`

Basic `depends_on` only controls start order. It does not guarantee that the database is ready.

```yaml
depends_on:
  - db
```

Better production-like dependency uses healthchecks:

```yaml
depends_on:
  db:
    condition: service_healthy
```

## 4. Healthcheck

A healthcheck tells Docker whether the service is actually ready.

Postgres healthcheck:

```yaml
healthcheck:
  test: ["CMD-SHELL", "pg_isready -U appuser -d appdb"]
  interval: 5s
  timeout: 5s
  retries: 5
```

Redis healthcheck:

```yaml
healthcheck:
  test: ["CMD", "redis-cli", "ping"]
  interval: 5s
  timeout: 5s
  retries: 5
```

Redis returns:

```text
PONG
```

## 5. Restart policies

| Policy | Meaning | Good For |
|---|---|---|
| `no` | Do not restart automatically | One-time containers |
| `always` | Restart whenever stopped/crashed | Databases, APIs, cache |
| `unless-stopped` | Restart unless manually stopped | Server apps |
| `on-failure` | Restart only on non-zero exit | Jobs, scripts, workers |

## 6. Named volumes

Postgres stores data at:

```text
/var/lib/postgresql/data
```

Named volume:

```yaml
volumes:
  - postgres-data:/var/lib/postgresql/data
```

This keeps DB data safe if the DB container is removed.

## 7. Custom networks

Explicit network:

```yaml
networks:
  backend-net:
    driver: bridge
```

This makes the architecture clear and controlled.

## 8. Labels

Labels add metadata to containers, networks, and volumes.

```yaml
labels:
  project: "day-34"
  service: "web"
  environment: "training"
```

Useful for filtering, monitoring, routing, and ownership.

## 9. Scaling

Compose can scale services:

```bash
docker compose up -d --scale web=3
```

But scaling breaks when a service has fixed host port mapping:

```yaml
ports:
  - "5000:5000"
```

Only one container can bind host port `5000`.

---

# Project Structure

```text
2026/day-34/
|-- app/
|   |-- Dockerfile
|   |-- app.py
|   `-- requirements.txt
|-- docker-compose.yml
|-- docker-compose-scale-test.yml
`-- day-34-compose-advanced.md
```

---

# Task 1 - Build Your Own App Stack

## Goal

Create a 3-service stack:

```text
web app: Python Flask
database: Postgres
cache: Redis
```

The app should connect to both Postgres and Redis.

## Files Created

### `app/app.py`

```python
import os
import time

import psycopg2
import redis
from flask import Flask, jsonify

app = Flask(__name__)

DB_HOST = os.getenv("DB_HOST", "db")
DB_NAME = os.getenv("DB_NAME", "appdb")
DB_USER = os.getenv("DB_USER", "appuser")
DB_PASSWORD = os.getenv("DB_PASSWORD", "apppass")
REDIS_HOST = os.getenv("REDIS_HOST", "redis")


def check_database():
    try:
        connection = psycopg2.connect(
            host=DB_HOST,
            database=DB_NAME,
            user=DB_USER,
            password=DB_PASSWORD,
            connect_timeout=3,
        )
        cursor = connection.cursor()
        cursor.execute("SELECT version();")
        version = cursor.fetchone()[0]
        cursor.close()
        connection.close()
        return True, version
    except Exception as error:
        return False, str(error)


def check_redis():
    try:
        client = redis.Redis(host=REDIS_HOST, port=6379, socket_connect_timeout=3)
        client.set("day34", "redis-connected")
        value = client.get("day34").decode("utf-8")
        return True, value
    except Exception as error:
        return False, str(error)


@app.route("/")
def home():
    db_ok, db_message = check_database()
    redis_ok, redis_message = check_redis()

    return jsonify(
        {
            "message": "Hello from Flask App - Updated in Task 4",
            "database": "OK" if db_ok else "FAILED",
            "database_info": db_message,
            "redis": "OK" if redis_ok else "FAILED",
            "redis_info": redis_message,
            "container": os.getenv("HOSTNAME", "unknown"),
            "timestamp": int(time.time()),
        }
    )


@app.route("/health")
def health():
    return jsonify({"status": "healthy"})


if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)

```

### `app/requirements.txt`

```text
flask==3.0.3
psycopg2-binary==2.9.9
redis==5.0.8

```

### `app/Dockerfile`

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

EXPOSE 5000

CMD ["python", "app.py"]

```

## Initial Compose File

```yaml
services:
  web:
    build: ./app
    container_name: day34-web
    ports:
      - "5000:5000"
    environment:
      DB_HOST: db
      DB_NAME: appdb
      DB_USER: appuser
      DB_PASSWORD: apppass
      REDIS_HOST: redis
    depends_on:
      - db
      - redis

  db:
    image: postgres:16-alpine
    container_name: day34-db
    environment:
      POSTGRES_DB: appdb
      POSTGRES_USER: appuser
      POSTGRES_PASSWORD: apppass

  redis:
    image: redis:7-alpine
    container_name: day34-redis
```

## Commands

```bash
mkdir -p ~/2026/day-34/app
cd ~/2026/day-34

docker compose up -d --build
docker compose ps
curl http://localhost:5000
```

## Output

```text
day34-db      postgres:16-alpine   Up   5432/tcp
day34-redis   redis:7-alpine       Up   6379/tcp
day34-web     day-34-web           Up   0.0.0.0:5000->5000/tcp
```

App response:

```json
{
  "database": "OK",
  "redis": "OK",
  "message": "Hello from Flask Docker Compose App"
}
```

## Task 1 Result

The Flask app successfully connected to Postgres and Redis using service names:

```text
db
redis
```

---

# Task 2 - depends_on and Healthchecks

## Goal

Make the app wait until Postgres and Redis are truly ready.

## Updated Compose Logic

```yaml
depends_on:
  db:
    condition: service_healthy
  redis:
    condition: service_healthy
```

## Postgres Healthcheck

```yaml
healthcheck:
  test: ["CMD-SHELL", "pg_isready -U appuser -d appdb"]
  interval: 5s
  timeout: 5s
  retries: 5
```

## Redis Healthcheck

```yaml
healthcheck:
  test: ["CMD", "redis-cli", "ping"]
  interval: 5s
  timeout: 5s
  retries: 5
```

## Commands

```bash
docker compose down
docker compose up -d --build
docker compose ps

docker inspect --format='{{json .State.Health}}' day34-db
docker inspect --format='{{json .State.Health}}' day34-redis

curl http://localhost:5000
```

## Output Observed

Postgres:

```text
ExitCode: 0
/var/run/postgresql:5432 - accepting connections
```

Redis:

```text
Status: healthy
Output: PONG
```

App:

```json
{
  "database": "OK",
  "redis": "OK"
}
```

## Task 2 Result

The app waited for healthy DB and Redis services before starting.

---

# Task 3 - Restart Policies

## Goal

Test restart policies and understand when to use them.

## `restart: always`

Used for database service:

```yaml
restart: always
```

Good for long-running services like:

```text
database
cache
API
worker
```

## `restart: on-failure`

Changed DB policy to:

```yaml
restart: on-failure
```

Test command:

```bash
docker kill day34-db
sleep 5
docker ps -a | grep day34-db
docker inspect -f '{{.RestartCount}}' day34-db
grep -n "restart:" docker-compose.yml
```

Observed:

```text
day34-db Exited (137)
RestartCount: 0
restart: on-failure
```

## Task 3 Result

In this test, the database did not restart with `on-failure`.

Conclusion:

```text
For databases, use restart: always or restart: unless-stopped.
For jobs/scripts/workers, use restart: on-failure.
```

---

# Task 4 - Custom Dockerfiles in Compose

## Goal

Use a custom Dockerfile and rebuild after code changes.

## Compose Build Setting

```yaml
web:
  build: ./app
```

## Code Change

Changed message from:

```text
Hello from Flask Docker Compose App
```

to:

```text
Hello from Flask App - Updated in Task 4
```

Command:

```bash
sed -i 's/Hello from Flask Docker Compose App/Hello from Flask App - Updated in Task 4/' app/app.py
```

Rebuild:

```bash
docker compose up -d --build
```

Verify:

```bash
curl http://localhost:5000
```

Output:

```json
{
  "message": "Hello from Flask App - Updated in Task 4",
  "database": "OK",
  "redis": "OK"
}
```

## Task 4 Result

Compose rebuilt the custom app image and the updated code appeared in the running container.

---

# Task 5 - Named Networks, Named Volumes, and Labels

## Goal

Make the Compose file more production-like.

Added:

```text
Explicit network: backend-net
Named volume: postgres-data
Labels: project, service, environment
```

## Final Compose File

```yaml
services:
  web:
    build: ./app
    container_name: day34-web
    ports:
      - "5000:5000"
    environment:
      DB_HOST: db
      DB_NAME: appdb
      DB_USER: appuser
      DB_PASSWORD: apppass
      REDIS_HOST: redis
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy
    networks:
      - backend-net
    labels:
      project: "day-34"
      service: "web"
      environment: "training"

  db:
    image: postgres:16-alpine
    container_name: day34-db
    restart: always
    environment:
      POSTGRES_DB: appdb
      POSTGRES_USER: appuser
      POSTGRES_PASSWORD: apppass
    volumes:
      - postgres-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U appuser -d appdb"]
      interval: 5s
      timeout: 5s
      retries: 5
    networks:
      - backend-net
    labels:
      project: "day-34"
      service: "database"
      environment: "training"

  redis:
    image: redis:7-alpine
    container_name: day34-redis
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 5s
      retries: 5
    networks:
      - backend-net
    labels:
      project: "day-34"
      service: "cache"
      environment: "training"

networks:
  backend-net:
    driver: bridge
    labels:
      project: "day-34"
      environment: "training"

volumes:
  postgres-data:
    labels:
      project: "day-34"
      service: "database-storage"

```

## Commands

```bash
docker compose down
docker compose up -d --build
docker compose ps
curl http://localhost:5000

docker network ls | grep backend
docker volume ls | grep postgres

docker inspect day34-web --format '{{json .Config.Labels}}'
docker inspect day34-db --format '{{json .Config.Labels}}'
docker inspect day34-redis --format '{{json .Config.Labels}}'
```

## Output Observed

Services:

```text
day34-db      Up (healthy)
day34-redis   Up (healthy)
day34-web     Up
```

Network:

```text
day-34_backend-net
```

Volume:

```text
day-34_postgres-data
```

Labels:

```json
"project":"day-34"
"environment":"training"
"service":"web"
"service":"database"
"service":"cache"
```

## Task 5 Result

Explicit network, named volume, and labels were successfully added.

---

# Task 6 - Scaling Bonus

## Goal

Try scaling the web app to 3 replicas.

Command:

```bash
docker compose up -d --scale web=3
```

## Why Scaling Can Break

Current web service had:

```yaml
container_name: day34-web
ports:
  - "5000:5000"
```

Problem 1:

```text
Fixed container_name prevents multiple replicas because names must be unique.
```

Problem 2:

```text
Fixed host port 5000 cannot be used by multiple containers.
```

## Scale Test

Created scale test file:

```bash
cp docker-compose.yml docker-compose-scale-test.yml
sed -i '/container_name: day34-web/d' docker-compose-scale-test.yml
```

Then ran:

```bash
docker compose -f docker-compose-scale-test.yml up -d --scale web=3
```

Observed error:

```text
Bind for 0.0.0.0:5000 failed: port is already allocated
```

## Task 6 Result

Scaling failed because all web replicas tried to use the same host port.

## Real-world Solution

Use a load balancer or reverse proxy:

```text
Nginx / Traefik / Load Balancer
        |
        v
web-1:5000
web-2:5000
web-3:5000
```

For scaled app services, use:

```yaml
expose:
  - "5000"
```

instead of:

```yaml
ports:
  - "5000:5000"
```

---

# Final Files Created

## `app/Dockerfile`

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

EXPOSE 5000

CMD ["python", "app.py"]

```

## `app/requirements.txt`

```text
flask==3.0.3
psycopg2-binary==2.9.9
redis==5.0.8

```

## `docker-compose.yml`

```yaml
services:
  web:
    build: ./app
    container_name: day34-web
    ports:
      - "5000:5000"
    environment:
      DB_HOST: db
      DB_NAME: appdb
      DB_USER: appuser
      DB_PASSWORD: apppass
      REDIS_HOST: redis
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy
    networks:
      - backend-net
    labels:
      project: "day-34"
      service: "web"
      environment: "training"

  db:
    image: postgres:16-alpine
    container_name: day34-db
    restart: always
    environment:
      POSTGRES_DB: appdb
      POSTGRES_USER: appuser
      POSTGRES_PASSWORD: apppass
    volumes:
      - postgres-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U appuser -d appdb"]
      interval: 5s
      timeout: 5s
      retries: 5
    networks:
      - backend-net
    labels:
      project: "day-34"
      service: "database"
      environment: "training"

  redis:
    image: redis:7-alpine
    container_name: day34-redis
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 5s
      retries: 5
    networks:
      - backend-net
    labels:
      project: "day-34"
      service: "cache"
      environment: "training"

networks:
  backend-net:
    driver: bridge
    labels:
      project: "day-34"
      environment: "training"

volumes:
  postgres-data:
    labels:
      project: "day-34"
      service: "database-storage"

```

## `docker-compose-scale-test.yml`

```yaml
services:
  web:
    build: ./app
    ports:
      - "5000:5000"
    environment:
      DB_HOST: db
      DB_NAME: appdb
      DB_USER: appuser
      DB_PASSWORD: apppass
      REDIS_HOST: redis
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy
    networks:
      - backend-net
    labels:
      project: "day-34"
      service: "web"
      environment: "training"

  db:
    image: postgres:16-alpine
    container_name: day34-db
    restart: always
    environment:
      POSTGRES_DB: appdb
      POSTGRES_USER: appuser
      POSTGRES_PASSWORD: apppass
    volumes:
      - postgres-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U appuser -d appdb"]
      interval: 5s
      timeout: 5s
      retries: 5
    networks:
      - backend-net
    labels:
      project: "day-34"
      service: "database"
      environment: "training"

  redis:
    image: redis:7-alpine
    container_name: day34-redis
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 5s
      retries: 5
    networks:
      - backend-net
    labels:
      project: "day-34"
      service: "cache"
      environment: "training"

networks:
  backend-net:
    driver: bridge
    labels:
      project: "day-34"
      environment: "training"

volumes:
  postgres-data:
    labels:
      project: "day-34"
      service: "database-storage"

```

---

# Important Commands Summary

## Start stack

```bash
docker compose up -d --build
```

## Check services

```bash
docker compose ps
```

## Test app

```bash
curl http://localhost:5000
```

## Check health

```bash
docker inspect --format='{{json .State.Health}}' day34-db
docker inspect --format='{{json .State.Health}}' day34-redis
```

## Rebuild after code change

```bash
docker compose up -d --build
```

## Check network

```bash
docker network ls | grep backend
```

## Check volume

```bash
docker volume ls | grep postgres
```

## Check labels

```bash
docker inspect day34-web --format '{{json .Config.Labels}}'
docker inspect day34-db --format '{{json .Config.Labels}}'
docker inspect day34-redis --format '{{json .Config.Labels}}'
```

## Scaling test

```bash
docker compose -f docker-compose-scale-test.yml up -d --scale web=3
```

---

# Troubleshooting Notes

## App starts before DB is ready

Use healthchecks and:

```yaml
depends_on:
  db:
    condition: service_healthy
```

## Code change not visible

Rebuild:

```bash
docker compose up -d --build
```

## DB healthcheck failing

Check logs:

```bash
docker compose logs db
```

Verify healthcheck matches DB user and DB name:

```bash
pg_isready -U appuser -d appdb
```

## Scaling fails with port error

Error:

```text
Bind for 0.0.0.0:5000 failed: port is already allocated
```

Reason:

```text
Multiple replicas cannot bind same host port.
```

Solution:

```text
Use reverse proxy/load balancer and expose internal port only.
```

---

# Practice Questions

## Basic Questions

1. What is a multi-container application?
2. What does `build: ./app` do?
3. What is the difference between `image:` and `build:`?
4. What is `depends_on` used for?
5. Why is basic `depends_on` not enough for databases?
6. What is a healthcheck?
7. What does `condition: service_healthy` do?
8. What is a restart policy?
9. What is a named volume?
10. What is a custom Docker network?

## Scenario-Based Questions

1. Your app starts before Postgres is ready. What should you add?
2. Your Flask code changed but output is old. What command should you run?
3. Your database container crashes and should come back automatically. Which restart policy should you use?
4. Your web app cannot connect to Redis. What should you check?
5. Your DB healthcheck fails. What command helps debug?
6. Your scaling command fails with port already allocated. Why?
7. You want to scale web replicas. What should replace fixed `ports`?
8. You need metadata for monitoring. What Compose feature can help?

---

# Interview Questions and Answers

## Q1. What is the difference between `image:` and `build:` in Compose?

`image:` uses an existing image from Docker Hub or local Docker.

`build:` builds a custom image from a Dockerfile.

Example:

```yaml
redis:
  image: redis:7-alpine

web:
  build: ./app
```

## Q2. Why is basic `depends_on` not enough?

Basic `depends_on` only controls start order. It does not wait until the dependency is ready. A database can be started but not ready to accept connections.

Use healthchecks with:

```yaml
condition: service_healthy
```

## Q3. What is a Docker healthcheck?

A healthcheck is a command Docker runs periodically to check if a container is healthy.

Example for Postgres:

```yaml
test: ["CMD-SHELL", "pg_isready -U appuser -d appdb"]
```

## Q4. What is `restart: always`?

It tells Docker to restart the container whenever it stops or crashes. It is useful for long-running services like databases, APIs, and caches.

## Q5. What is `restart: on-failure`?

It restarts the container only if the main process exits with a non-zero exit code. It is useful for jobs, scripts, and workers.

## Q6. Why use named volumes for databases?

Named volumes store database data outside the container writable layer. This keeps data safe when the container is recreated.

## Q7. Why use a custom network?

A custom network makes service communication explicit and allows containers to talk using service names.

## Q8. What are Docker labels?

Labels are metadata attached to Docker objects. They help with filtering, monitoring, routing, automation, and ownership tracking.

## Q9. Why did scaling fail with `--scale web=3`?

Scaling failed because every web replica tried to bind the same host port `5000`. Only one container can bind one host port.

## Q10. How do you scale properly?

Remove fixed host port mapping from replicas and place a load balancer or reverse proxy in front.

Example:

```yaml
expose:
  - "5000"
```

Then use Nginx, Traefik, Docker Swarm, Kubernetes, ECS, or another orchestrator/load balancer.

---

# Advanced Interview Questions

## Q11. How would you improve this setup for production?

Use secrets, non-root containers, resource limits, centralized logging, monitoring, reverse proxy, TLS, backup strategy, managed database, and an orchestrator like Kubernetes or ECS.

## Q12. Is Docker Compose enough for production?

Compose is excellent for local development, demos, testing, and small deployments. For large production workloads, Kubernetes, ECS, Docker Swarm, or Azure Container Apps are more suitable.

## Q13. What is the difference between `ports` and `expose`?

`ports` publishes a container port to the host.

`expose` makes a port available only to other containers on the same network.

## Q14. Why avoid fixed `container_name` for scalable services?

Because container names must be unique. Fixed `container_name` prevents Compose from creating multiple replicas.

## Q15. What is Redis used for?

Redis is commonly used for caching, sessions, queues, pub/sub, rate limiting, and temporary fast storage.

---

# Final Revision Notes

- Compose can run real multi-container apps.
- `build:` builds a custom image from a Dockerfile.
- `image:` uses an existing image.
- Healthchecks verify readiness.
- `condition: service_healthy` waits for healthy services.
- `restart: always` is useful for long-running services.
- `restart: on-failure` is useful for jobs and workers.
- Named volumes persist DB data.
- Custom networks make architecture explicit.
- Labels help organize and automate Docker resources.
- Scaling fails with fixed host ports.
- Use load balancers for real scaling.
