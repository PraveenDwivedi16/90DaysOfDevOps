# Day 36 – Docker Project: Dockerize a Full Application

## Goal

Today's goal is to Dockerize a complete application end-to-end.

This is a real DevOps-style task:

```text
Choose an app
Write Dockerfile
Add .dockerignore
Add Docker Compose
Add database
Add volume
Add custom network
Add environment variables
Add healthcheck
Build and test locally
Push image to Docker Hub
Document everything
Test fresh pull flow
```

---

# App Chosen

## Chosen App

I chose:

```text
Python Flask API + PostgreSQL Database
```

## Why I chose this app

I chose Flask + PostgreSQL because it is simple enough to understand clearly, but still close to a real-world backend application.

This app demonstrates important Docker and DevOps concepts:

- Custom application Dockerfile
- Multi-stage Docker build
- Non-root user
- PostgreSQL database container
- Docker Compose
- Environment variables
- Database persistence using named volume
- Custom Docker network
- Healthcheck for database
- App-to-database communication using service name
- Docker Hub image push and pull workflow

---

# Final Project Structure

```text
2026/day-36/
│
├── app/
│   ├── app.py
│   ├── requirements.txt
│   └── Dockerfile
│
├── .dockerignore
├── .env.example
├── docker-compose.yml
├── docker-compose.hub.yml
├── README.md
└── day-36-docker-project.md
```

---

# What the App Does

This is a small Flask API.

It has endpoints:

```text
GET /          → Shows app status and database connection status
GET /health    → Healthcheck endpoint
GET /users     → Lists users from PostgreSQL
POST /users    → Adds a user to PostgreSQL
```

The app automatically creates a `users` table if it does not exist.

---

# Task 1 – Pick Your App

## Requirement

Choose one app:

- Python Flask/Django app with database
- Node.js Express with MongoDB
- Static website + backend API
- Existing GitHub project
- Open-source app

## Selected

```text
Python Flask API with PostgreSQL
```

## Reason

This is a good beginner-to-real-world Docker project because it has:

```text
App container
Database container
Environment variables
Persistent volume
Custom network
Healthcheck
Docker Hub deployment
```

---

# Task 2 – Write the Dockerfile

## App Code

File:

```text
app/app.py
```

```python
import os
import time

import psycopg2
from flask import Flask, jsonify, request

app = Flask(__name__)

DB_HOST = os.getenv("DB_HOST", "db")
DB_PORT = os.getenv("DB_PORT", "5432")
DB_NAME = os.getenv("DB_NAME", "appdb")
DB_USER = os.getenv("DB_USER", "appuser")
DB_PASSWORD = os.getenv("DB_PASSWORD", "apppass")


def get_connection():
    return psycopg2.connect(
        host=DB_HOST,
        port=DB_PORT,
        database=DB_NAME,
        user=DB_USER,
        password=DB_PASSWORD,
        connect_timeout=5,
    )


def init_db():
    connection = get_connection()
    cursor = connection.cursor()
    cursor.execute(
        '''
        CREATE TABLE IF NOT EXISTS users (
            id SERIAL PRIMARY KEY,
            name VARCHAR(100) NOT NULL,
            created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
        );
        '''
    )
    connection.commit()
    cursor.close()
    connection.close()


def check_database():
    try:
        connection = get_connection()
        cursor = connection.cursor()
        cursor.execute("SELECT 1;")
        cursor.fetchone()
        cursor.close()
        connection.close()
        return True
    except Exception:
        return False


@app.route("/")
def home():
    db_ok = check_database()
    return jsonify(
        {
            "message": "Day 36 Dockerized Flask App",
            "database": "OK" if db_ok else "FAILED",
            "timestamp": int(time.time()),
        }
    )


@app.route("/health")
def health():
    return jsonify({"status": "healthy"})


@app.route("/users", methods=["GET"])
def get_users():
    init_db()
    connection = get_connection()
    cursor = connection.cursor()
    cursor.execute("SELECT id, name, created_at FROM users ORDER BY id;")
    rows = cursor.fetchall()
    cursor.close()
    connection.close()

    users = [
        {"id": row[0], "name": row[1], "created_at": str(row[2])}
        for row in rows
    ]

    return jsonify({"users": users})


@app.route("/users", methods=["POST"])
def create_user():
    init_db()
    data = request.get_json(silent=True) or {}
    name = data.get("name", "").strip()

    if not name:
        return jsonify({"error": "name is required"}), 400

    connection = get_connection()
    cursor = connection.cursor()
    cursor.execute("INSERT INTO users (name) VALUES (%s) RETURNING id;", (name,))
    user_id = cursor.fetchone()[0]
    connection.commit()
    cursor.close()
    connection.close()

    return jsonify({"id": user_id, "name": name}), 201


if __name__ == "__main__":
    init_db()
    app.run(host="0.0.0.0", port=5000)
```

---

## requirements.txt

File:

```text
app/requirements.txt
```

```text
flask==3.0.3
psycopg2-binary==2.9.9
gunicorn==22.0.0
```

## Package Explanation

| Package | Purpose |
|---|---|
| `flask` | Web framework |
| `psycopg2-binary` | PostgreSQL driver |
| `gunicorn` | Production-style WSGI server |

---

# Dockerfile

File:

```text
app/Dockerfile
```

```dockerfile
# Stage 1: Builder stage
FROM python:3.12-slim AS builder

# Set working directory
WORKDIR /app

# Prevent Python from writing pyc files and enable unbuffered logs
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

# Copy dependency file first for better Docker cache
COPY requirements.txt .

# Install dependencies into a temporary install location
RUN pip install --no-cache-dir --prefix=/install -r requirements.txt


# Stage 2: Runtime stage
FROM python:3.12-slim

# Set working directory
WORKDIR /app

# Create non-root user for security
RUN useradd --create-home --shell /bin/bash appuser

# Copy installed dependencies from builder stage
COPY --from=builder /install /usr/local

# Copy application code
COPY app.py .

# Change ownership to non-root user
RUN chown -R appuser:appuser /app

# Switch to non-root user
USER appuser

# Flask/Gunicorn runs on port 5000
EXPOSE 5000

# Start the app using Gunicorn
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "app:app"]
```

---

## Dockerfile Explanation Line by Line

```dockerfile
FROM python:3.12-slim AS builder
```

Uses Python slim image as a builder stage.

```dockerfile
WORKDIR /app
```

Sets `/app` as working directory.

```dockerfile
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1
```

Prevents `.pyc` files and makes logs visible immediately.

```dockerfile
COPY requirements.txt .
```

Copies only dependency file first for better cache.

```dockerfile
RUN pip install --no-cache-dir --prefix=/install -r requirements.txt
```

Installs dependencies into `/install`.

```dockerfile
FROM python:3.12-slim
```

Starts clean runtime stage.

```dockerfile
RUN useradd --create-home --shell /bin/bash appuser
```

Creates non-root user.

```dockerfile
COPY --from=builder /install /usr/local
```

Copies installed dependencies from builder stage.

```dockerfile
COPY app.py .
```

Copies app code.

```dockerfile
RUN chown -R appuser:appuser /app
```

Gives app folder ownership to non-root user.

```dockerfile
USER appuser
```

Runs app as non-root.

```dockerfile
EXPOSE 5000
```

Documents application port.

```dockerfile
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "app:app"]
```

Starts the app with Gunicorn.

---

# .dockerignore

File:

```text
.dockerignore
```

```text
.git
__pycache__
*.pyc
.env
*.md
.vscode
.idea
node_modules
logs
```

## Why .dockerignore is important

It prevents unnecessary or sensitive files from going into the Docker build context.

Benefits:

- Faster builds
- Smaller images
- Better security
- Avoids copying `.env`
- Avoids copying Git history

---

# Build and Test Locally

## Build Image

```bash
cd ~/2026/day-36
docker build -t day36-flask-app:v1 ./app
```

## Run App Only

For full test, use Compose because the app needs PostgreSQL.

---

# Task 3 – Add Docker Compose

## Requirement

Create a `docker-compose.yml` with:

- App service built from Dockerfile
- Database service
- Volume for database persistence
- Custom network
- Environment variables using `.env`
- Database healthcheck

---

# .env.example

File:

```text
.env.example
```

```env
APP_PORT=5010

DB_HOST=db
DB_PORT=5432
DB_NAME=appdb
DB_USER=appuser
DB_PASSWORD=apppass

POSTGRES_DB=appdb
POSTGRES_USER=appuser
POSTGRES_PASSWORD=apppass
```

For local running, copy:

```bash
cp .env.example .env
```

---

# docker-compose.yml

File:

```text
docker-compose.yml
```

```yaml
services:
  app:
    build: ./app
    image: day36-flask-app:v1
    container_name: day36-app
    restart: unless-stopped
    ports:
      - "${APP_PORT}:5000"
    environment:
      DB_HOST: ${DB_HOST}
      DB_PORT: ${DB_PORT}
      DB_NAME: ${DB_NAME}
      DB_USER: ${DB_USER}
      DB_PASSWORD: ${DB_PASSWORD}
    depends_on:
      db:
        condition: service_healthy
    networks:
      - day36-net
    labels:
      project: "day-36"
      service: "flask-app"
      environment: "training"

  db:
    image: postgres:16-alpine
    container_name: day36-db
    restart: unless-stopped
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - day36-postgres-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
      interval: 5s
      timeout: 5s
      retries: 5
    networks:
      - day36-net
    labels:
      project: "day-36"
      service: "postgres-db"
      environment: "training"

networks:
  day36-net:
    driver: bridge
    labels:
      project: "day-36"

volumes:
  day36-postgres-data:
    labels:
      project: "day-36"
      service: "postgres-data"
```

---

## Compose Explanation

## App service

```yaml
app:
  build: ./app
```

Builds the application image from `app/Dockerfile`.

```yaml
image: day36-flask-app:v1
```

Names the built image.

```yaml
ports:
  - "${APP_PORT}:5000"
```

Maps host port from `.env`.

```yaml
depends_on:
  db:
    condition: service_healthy
```

App waits until DB is healthy.

---

## DB service

```yaml
image: postgres:16-alpine
```

Uses lightweight Postgres image.

```yaml
volumes:
  - day36-postgres-data:/var/lib/postgresql/data
```

Persists database data.

```yaml
healthcheck:
  test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
```

Checks DB readiness.

---

# Run with Docker Compose

## Start

```bash
cd ~/2026/day-36
cp .env.example .env
docker compose up -d --build
```

## Check

```bash
docker compose ps
```

Expected:

```text
day36-db    Up (healthy)
day36-app   Up
```

## Test API

```bash
curl http://localhost:5010
```

Expected:

```json
{
  "message": "Day 36 Dockerized Flask App",
  "database": "OK"
}
```

## Add User

```bash
curl -X POST http://localhost:5010/users \
  -H "Content-Type: application/json" \
  -d '{"name":"Hardik"}'
```

Expected:

```json
{
  "id": 1,
  "name": "Hardik"
}
```

## List Users

```bash
curl http://localhost:5010/users
```

Expected:

```json
{
  "users": [
    {
      "id": 1,
      "name": "Hardik",
      "created_at": "..."
    }
  ]
}
```

---

# Task 4 – Ship It to Docker Hub

## Goal

Tag the app image and push it to Docker Hub.

---

## Step 1: Login

```bash
docker login
```

Use Docker Hub username and access token.

---

## Step 2: Tag Image

Replace `YOUR_DOCKERHUB_USERNAME` with your username.

```bash
docker tag day36-flask-app:v1 YOUR_DOCKERHUB_USERNAME/day36-flask-app:v1
docker tag day36-flask-app:v1 YOUR_DOCKERHUB_USERNAME/day36-flask-app:latest
```

---

## Step 3: Push Image

```bash
docker push YOUR_DOCKERHUB_USERNAME/day36-flask-app:v1
docker push YOUR_DOCKERHUB_USERNAME/day36-flask-app:latest
```

---

## Docker Hub Link

Write your final Docker Hub link here:

```text
https://hub.docker.com/r/YOUR_DOCKERHUB_USERNAME/day36-flask-app
```

---

# docker-compose.hub.yml

This file is used for testing the image pulled from Docker Hub instead of building locally.

```yaml
services:
  app:
    image: YOUR_DOCKERHUB_USERNAME/day36-flask-app:v1
    container_name: day36-app
    restart: unless-stopped
    ports:
      - "${APP_PORT}:5000"
    environment:
      DB_HOST: ${DB_HOST}
      DB_PORT: ${DB_PORT}
      DB_NAME: ${DB_NAME}
      DB_USER: ${DB_USER}
      DB_PASSWORD: ${DB_PASSWORD}
    depends_on:
      db:
        condition: service_healthy
    networks:
      - day36-net

  db:
    image: postgres:16-alpine
    container_name: day36-db
    restart: unless-stopped
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - day36-postgres-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
      interval: 5s
      timeout: 5s
      retries: 5
    networks:
      - day36-net

networks:
  day36-net:
    driver: bridge

volumes:
  day36-postgres-data:
```

Important:

Before using, replace:

```text
YOUR_DOCKERHUB_USERNAME
```

with your real Docker Hub username.

---

# README.md

Your project README should include:

```md
# Day 36 Flask Docker Project

## What this app does

This is a Flask API connected to a PostgreSQL database.

Endpoints:

- GET /
- GET /health
- GET /users
- POST /users

## How to run

cp .env.example .env
docker compose up -d --build

## Test

curl http://localhost:5010

curl -X POST http://localhost:5010/users \
  -H "Content-Type: application/json" \
  -d '{"name":"Hardik"}'

curl http://localhost:5010/users

## Environment variables

APP_PORT=5010
DB_HOST=db
DB_PORT=5432
DB_NAME=appdb
DB_USER=appuser
DB_PASSWORD=apppass

POSTGRES_DB=appdb
POSTGRES_USER=appuser
POSTGRES_PASSWORD=apppass

## Docker Hub

Image:

YOUR_DOCKERHUB_USERNAME/day36-flask-app:v1
```

---

# Task 5 – Test the Whole Flow

## Goal

Remove local containers/images and verify app works fresh using Docker Hub image.

---

## Step 1: Stop and Remove Compose Stack

```bash
docker compose down
```

Optional if you want to test fresh DB data:

```bash
docker compose down -v
```

Warning:

```text
-v removes database volume and deletes data.
```

---

## Step 2: Remove Local App Images

```bash
docker rmi day36-flask-app:v1
docker rmi YOUR_DOCKERHUB_USERNAME/day36-flask-app:v1
docker rmi YOUR_DOCKERHUB_USERNAME/day36-flask-app:latest
```

If image is in use, remove containers first.

---

## Step 3: Pull from Docker Hub

```bash
docker pull YOUR_DOCKERHUB_USERNAME/day36-flask-app:v1
```

---

## Step 4: Run Compose Using Hub Image

```bash
docker compose -f docker-compose.hub.yml up -d
```

---

## Step 5: Verify

```bash
docker compose -f docker-compose.hub.yml ps
curl http://localhost:5010
curl http://localhost:5010/health
```

Expected:

```json
{
  "message": "Day 36 Dockerized Flask App",
  "database": "OK"
}
```

---

# Challenges Faced and Solutions

## Challenge 1: App starts before database is ready

Problem:

```text
Flask app may start before PostgreSQL is ready.
```

Solution:

Added DB healthcheck and:

```yaml
depends_on:
  db:
    condition: service_healthy
```

---

## Challenge 2: Data should persist after container removal

Problem:

```text
Database data can be lost if stored only in container writable layer.
```

Solution:

Used named volume:

```yaml
day36-postgres-data:/var/lib/postgresql/data
```

---

## Challenge 3: Avoid running container as root

Problem:

```text
Root containers are less secure.
```

Solution:

Created non-root user in Dockerfile:

```dockerfile
RUN useradd --create-home --shell /bin/bash appuser
USER appuser
```

---

## Challenge 4: Avoid copying unnecessary files

Problem:

```text
Docker build context may include .env, .git, cache files, etc.
```

Solution:

Added `.dockerignore`.

---

## Challenge 5: Verify Docker Hub image works fresh

Problem:

```text
Local image may hide problems that happen after pull.
```

Solution:

Created `docker-compose.hub.yml` and tested using pulled image.

---

# Final Image Size

After building, check image size:

```bash
docker images | grep day36-flask-app
```

Write your final size:

```text
Final image size: ______ MB
```

---

# Docker Hub Link

Add your link:

```text
Docker Hub: https://hub.docker.com/r/YOUR_DOCKERHUB_USERNAME/day36-flask-app
```

---

# Important Commands Summary

## Build

```bash
docker build -t day36-flask-app:v1 ./app
```

## Compose up

```bash
docker compose up -d --build
```

## Check containers

```bash
docker compose ps
```

## Logs

```bash
docker compose logs app
docker compose logs db
```

## Test API

```bash
curl http://localhost:5010
curl http://localhost:5010/health
curl http://localhost:5010/users
```

## Add user

```bash
curl -X POST http://localhost:5010/users \
  -H "Content-Type: application/json" \
  -d '{"name":"Hardik"}'
```

## Docker Hub

```bash
docker login
docker tag day36-flask-app:v1 YOUR_DOCKERHUB_USERNAME/day36-flask-app:v1
docker push YOUR_DOCKERHUB_USERNAME/day36-flask-app:v1
docker pull YOUR_DOCKERHUB_USERNAME/day36-flask-app:v1
```

## Run from Docker Hub image

```bash
docker compose -f docker-compose.hub.yml up -d
```

---

# Screenshots to Attach

Attach screenshots for:

1. Project folder structure
2. Flask app code
3. Dockerfile
4. `.dockerignore`
5. `.env.example`
6. `docker-compose.yml`
7. `docker compose up -d --build`
8. `docker compose ps`
9. `curl http://localhost:5010`
10. `POST /users`
11. `GET /users`
12. Docker image size
13. Docker Hub login
14. Docker tag command
15. Docker push command
16. Docker Hub repository page
17. Docker pull command
18. Fresh run using `docker-compose.hub.yml`

---

# Practice Questions

## Basic Questions

1. What app did you Dockerize?
2. Why did you choose Flask + PostgreSQL?
3. What is the purpose of Dockerfile?
4. What is the purpose of `.dockerignore`?
5. Why use Docker Compose?
6. Why use environment variables?
7. Why use a named volume for PostgreSQL?
8. What is a Docker network?
9. Why add a healthcheck?
10. Why push image to Docker Hub?

---

## Scenario-Based Questions

1. Your app cannot connect to PostgreSQL. What will you check?
2. Your app starts before DB is ready. How will you fix it?
3. Your database data disappears after restart. What is missing?
4. Your image contains `.env`. What did you forget?
5. Docker Hub push fails. What will you check?
6. App works locally but not after pull. How will you debug?
7. You changed code but app still shows old behavior. What command should you run?
8. Your container runs as root. How will you fix it?
9. Your Compose file has hardcoded passwords. What is a better approach?
10. Your app should run fresh on another server. What files are required?

---

# Interview Questions and Answers

## Q1. How do you Dockerize a full application?

Steps:

1. Understand app runtime and dependencies.
2. Write Dockerfile.
3. Add `.dockerignore`.
4. Build image.
5. Test container locally.
6. Add Docker Compose for database and dependencies.
7. Add environment variables.
8. Add volumes and networks.
9. Add healthchecks.
10. Push image to registry.
11. Test fresh pull and run.

---

## Q2. Why did you use Docker Compose?

Docker Compose is useful for running multi-container applications. This project needs both Flask app and PostgreSQL database, so Compose manages both services, network, volume, and environment variables together.

---

## Q3. Why did you use a named volume for PostgreSQL?

PostgreSQL data is stored in:

```text
/var/lib/postgresql/data
```

If data remains only inside the container, it can be lost when the container is removed. A named volume persists the data outside the container lifecycle.

---

## Q4. Why did you add a database healthcheck?

The app should not start before the DB is ready. The healthcheck uses `pg_isready` to confirm PostgreSQL is accepting connections.

---

## Q5. What does `depends_on` with `condition: service_healthy` do?

It tells Compose to wait until the database healthcheck passes before starting the app.

---

## Q6. Why did you use a non-root user?

Running as root is risky. If the app is compromised, root privileges increase potential damage. A non-root user reduces security risk.

---

## Q7. Why use `.dockerignore`?

`.dockerignore` prevents unnecessary or sensitive files from entering the build context or image. It improves speed, security, and image cleanliness.

---

## Q8. Why use environment variables?

Environment variables allow configuration without changing code. They make the app portable across local, staging, and production environments.

---

## Q9. Why use Docker Hub?

Docker Hub allows storing and sharing Docker images. Once pushed, the image can be pulled and run on another machine or deployment server.

---

## Q10. What is the difference between docker-compose.yml and docker-compose.hub.yml?

`docker-compose.yml` builds the app locally using:

```yaml
build: ./app
```

`docker-compose.hub.yml` uses the already pushed Docker Hub image:

```yaml
image: YOUR_DOCKERHUB_USERNAME/day36-flask-app:v1
```

This verifies that the app can run fresh from the registry.

---

# Senior-Level Interview Questions and Answers

## Q11. What would you improve before production?

Improvements:

- Use secrets manager instead of `.env`
- Add CI/CD pipeline
- Add vulnerability scanning
- Add resource limits
- Add structured logging
- Add reverse proxy
- Use managed PostgreSQL
- Add backups
- Add monitoring and alerts
- Use Kubernetes/ECS for orchestration
- Use immutable image tags

---

## Q12. Why should images use immutable tags?

Mutable tags like `latest` can change over time, causing unpredictable deployments.

Immutable tags like:

```text
v1.0.0
git-sha-abc123
```

ensure the exact same image is deployed.

---

## Q13. How do you debug app-to-database connection issues?

Check:

```bash
docker compose ps
docker compose logs app
docker compose logs db
docker network inspect
docker compose exec app env
docker compose exec db pg_isready -U appuser -d appdb
```

Also verify:

- DB_HOST
- DB_PORT
- DB_NAME
- DB_USER
- DB_PASSWORD
- Network membership
- Healthcheck status

---

## Q14. Why is using a managed database often better in production?

Managed databases provide:

- Backups
- Replication
- Monitoring
- Patching
- High availability
- Automated recovery
- Better operational reliability

Containerized DBs are useful for local development and testing, but production usually benefits from managed DB services.

---

## Q15. What is the difference between build-time and run-time configuration?

Build-time configuration is baked into the image during `docker build`.

Run-time configuration is passed when the container starts, usually through environment variables or secrets.

Best practice:

> Keep environment-specific values as run-time configuration.

---

## Q16. Why should secrets not be inside Docker images?

Images can be pushed to registries and shared. If secrets are baked into images, anyone with image access may extract them.

Use secrets managers or runtime environment injection.

---

## Q17. How would you make this app production-ready?

I would:

- Use Gunicorn with multiple workers
- Put Nginx or load balancer in front
- Use managed PostgreSQL
- Add migrations
- Add logging and monitoring
- Add health endpoints
- Add resource limits
- Use Docker secrets
- Use CI/CD
- Use vulnerability scanning
- Deploy with Kubernetes/ECS

---

## Q18. Why is healthcheck important in orchestration?

Orchestrators use healthchecks to know whether a container is healthy. If unhealthy, they can restart it or stop sending traffic to it.

---

## Q19. How would you reduce image size?

Use:

- Slim/alpine base image
- Multi-stage build
- `.dockerignore`
- Remove cache
- Avoid unnecessary packages
- Copy only required files

---

## Q20. What does “works on my machine” problem mean and how does Docker solve it?

It means the app works on one developer machine but fails elsewhere due to different dependencies, OS, or configuration.

Docker solves this by packaging app runtime, dependencies, and configuration into a reproducible container image.

---

# Final Revision Notes

## Key Points

- Dockerizing a full app means app code, Dockerfile, Compose, DB, volumes, network, env variables, and registry flow.
- `.dockerignore` protects build context.
- Non-root user improves security.
- PostgreSQL needs volume persistence.
- Compose service names are DNS names.
- Healthchecks prevent app from starting too early.
- Docker Hub is used to distribute images.
- Fresh pull test proves the image works outside local build cache.
- Documentation is part of DevOps delivery.
