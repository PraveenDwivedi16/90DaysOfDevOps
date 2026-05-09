# Day 31 – Dockerfile: Build Your Own Images

## Goal

Today's goal was to write Dockerfiles and build custom Docker images.

This is an important Docker skill because using existing images is only the first step. In real DevOps work, we usually create custom images for our own applications.

# Topics Covered

- What is a Dockerfile?
- How to build custom Docker images
- Common Dockerfile instructions
- `FROM`, `RUN`, `COPY`, `WORKDIR`, `EXPOSE`, `CMD`, `ENTRYPOINT`
- Difference between `CMD` and `ENTRYPOINT`
- Building a custom Nginx static website image
- Using `.dockerignore`
- Docker build context
- Docker image layer caching
- Dockerfile build optimization
- Real-world Dockerfile best practices
- Docker interview questions and answers

# 1. What is a Dockerfile?

A Dockerfile is a text file that contains instructions to build a Docker image.

Simple understanding:
Dockerfile = Recipe
Docker Image = Final package created from recipe
Docker Container = Running instance of image
Example:
FROM ubuntu

RUN apt update && apt install -y curl

CMD ["echo", "Hello from my custom image!"]

Build image:
docker build -t my-ubuntu:v1 .
Run container:
docker run my-ubuntu:v1
Expected output:
Hello from my custom image!

# 2. Dockerfile Build Flow

Dockerfile instructions run from top to bottom.
FROM selects the base image
RUN installs dependencies or changes the image during build
COPY copies files into the image
WORKDIR sets the default folder
EXPOSE documents the container port
CMD defines the default startup command

# 3. Important Dockerfile Instructions

## FROM

FROM defines the base image.

dockerfile
FROM ubuntu
Meaning: my custom image will be built on top of Ubuntu.
Other examples:
dockerfile
FROM nginx:alpine
FROM node:20-alpine
FROM python:3.12-slim
FROM php:8.3-apache

Most Dockerfiles start with `FROM`.

## RUN
`RUN` executes commands during the image build process.
dockerfile
RUN apt update && apt install -y curl

`RUN` executes during image build, not when the container starts.

## COPY

`COPY` copies files from the host machine into the Docker image.
dockerfile
COPY index.html /usr/share/nginx/html/index.html


## WORKDIR
`WORKDIR` sets the working directory inside the image/container.
dockerfile
WORKDIR /app

After this, relative commands run inside `/app`.

## EXPOSE
`EXPOSE` documents the port used by the container.
dockerfile
EXPOSE 80


Important: `EXPOSE` does not publish the port to the host machine. To access from host, use `-p`:
docker run -p 8080:80 my-image

## CMD

`CMD` defines the default command when a container starts.

dockerfile
CMD ["echo", "Hello from Docker"]

`CMD` can be overridden from `docker run`:
docker run my-image echo "custom command"

## ENTRYPOINT
`ENTRYPOINT` defines a fixed executable for the container.
dockerfile
ENTRYPOINT ["echo"]

Run:
docker run entrypoint-demo hello

Output:
hello
Here `echo` is fixed, and `hello` is passed as an argument.

# 4. Build Context

When we run:
docker build -t my-image:v1 .
The final `.` is the build context.
Meaning: Docker sends the current folder contents to Docker daemon for building the image.
If the folder contains unnecessary files like `node_modules`, `.git`, `.env`, large logs, or screenshots, the build can become slow and insecure. That is why `.dockerignore` is important.


# Task 1: Your First Dockerfile
Create a custom image that:
- Uses Ubuntu as base image
- Installs curl
- Prints a default message
- Builds with tag `my-ubuntu:v1`
- Runs successfully

## Folder
2026/day-31/my-first-image/

## Dockerfile
dockerfile
FROM ubuntu
RUN apt update && apt install -y curl
CMD ["echo", "Hello from my custom image!"]


## Commands
mkdir -p 2026/day-31/my-first-image
cd 2026/day-31/my-first-image
cat > Dockerfile <<'EOF'
FROM ubuntu
RUN apt update && apt install -y curl
CMD ["echo", "Hello from my custom image!"]
EOF

docker build -t my-ubuntu:v1 .
docker images
docker run my-ubuntu:v1
docker run my-ubuntu:v1 curl --version


Expected message:
Hello from my custom image!
Observed curl verification:
curl 8.18.0 ...

## Learning
This task proves that a custom image was created, Ubuntu was used as the base image, curl was installed during build, and `CMD` ran when the container started.
When we run:
docker run my-ubuntu:v1 curl --version
the custom command replaces the default CMD.


# Task 2: Dockerfile Instructions
## Goal
Create a Dockerfile using:
- `FROM`
- `RUN`
- `COPY`
- `WORKDIR`
- `EXPOSE`
- `CMD`

## Folder Structure
2026/day-31/dockerfile-instructions/
├── Dockerfile
└── message.txt

## message.txt
Hello from Dockerfile COPY instruction!
This file was copied from host machine into Docker image.

## Dockerfile
FROM ubuntu
RUN apt update && apt install -y curl
WORKDIR /app
COPY message.txt /app/message.txt
EXPOSE 8080
CMD ["cat", "/app/message.txt"]


## Commands
cd ~/2026/day-31
mkdir -p dockerfile-instructions
cd dockerfile-instructions

cat > message.txt <<'EOF'
Hello from Dockerfile COPY instruction!
This file was copied from host machine into Docker image.
EOF

cat > Dockerfile <<'EOF'
FROM ubuntu
RUN apt update && apt install -y curl
WORKDIR /app
COPY message.txt /app/message.txt
EXPOSE 8080
CMD ["cat", "/app/message.txt"]
EOF

docker build -t dockerfile-instructions:v1 .
docker run dockerfile-instructions:v1
docker run dockerfile-instructions:v1 pwd
docker run dockerfile-instructions:v1 ls
docker run dockerfile-instructions:v1 cat message.txt


Observed output:
Hello from Dockerfile COPY instruction!
This file was copied from host machine into Docker image.
/app
message.txt

## Explanation
- `FROM ubuntu` uses Ubuntu as the base image.
- `RUN apt update && apt install -y curl` installs curl during build.
- `WORKDIR /app` sets `/app` as the working directory.
- `COPY message.txt /app/message.txt` copies a host file into the image.
- `EXPOSE 8080` documents that the container may use port 8080.
- `CMD ["cat", "/app/message.txt"]` prints the copied file by default.

Important: `EXPOSE` does not publish a port. Use `docker run -p` for actual access from the host.

# Task 3: CMD vs ENTRYPOINT
## CMD Demo
2026/day-31/cmd-vs-entrypoint/cmd-demo/

Dockerfile:
FROM alpine
CMD ["echo", "hello"]
Commands:
cd ~/2026/day-31
mkdir -p cmd-vs-entrypoint/cmd-demo
cd cmd-vs-entrypoint/cmd-demo
cat > Dockerfile <<'EOF'
FROM alpine
CMD ["echo", "hello"]
EOF

docker build -t cmd-demo:v1 .
docker run cmd-demo:v1
docker run cmd-demo:v1 echo "custom message"


Expected:
hello
custom message

CMD provides a default command. When we pass a custom command in `docker run`, it replaces CMD.

## ENTRYPOINT Demo
2026/day-31/cmd-vs-entrypoint/entrypoint-demo/

Dockerfile:
FROM alpine
ENTRYPOINT ["echo"]

Commands:
cd ~/2026/day-31
mkdir -p cmd-vs-entrypoint/entrypoint-demo
cd cmd-vs-entrypoint/entrypoint-demo

cat > Dockerfile <<'EOF'
FROM alpine

ENTRYPOINT ["echo"]
EOF

docker build -t entrypoint-demo:v1 .
docker run entrypoint-demo:v1 hello
docker run entrypoint-demo:v1 hardik docker

Observed:
hello
hardik docker

ENTRYPOINT fixed the executable as `echo`. Arguments after image name were passed to `echo`.

## Override ENTRYPOINT

Command:
docker run --entrypoint ls entrypoint-demo:v1 /


On Git Bash, this may fail because Git Bash converts `/` into a Windows path.

Correct Git Bash command:
MSYS_NO_PATHCONV=1 docker run --entrypoint ls entrypoint-demo:v1 /
or:
docker run --entrypoint ls entrypoint-demo:v1 //

## CMD vs ENTRYPOINT Summary
| Point | CMD | ENTRYPOINT |
|---|---|---|
| Purpose | Default command | Fixed executable |
| Override behavior | Easily replaced | Not easily replaced |
| Docker run arguments | Replace CMD | Passed as arguments |
| Use case | Default app behavior | CLI tools / fixed startup |
| Example | `CMD ["npm", "start"]` | `ENTRYPOINT ["dotnet", "App.dll"]` |

Use CMD when you want default behavior that users can override. Use ENTRYPOINT when the container should always run a fixed executable.


# Task 4: Build a Simple Web App Image

## Goal

Create a custom Nginx website image.

The image should:

- Use `nginx:alpine`
- Copy custom `index.html`
- Expose port 80
- Run Nginx
- Open in browser

## Folder Structure
2026/day-31/my-website/
├── Dockerfile
└── index.html

## index.html
<!DOCTYPE html>
<html>
<head>
    <title>My Docker Website</title>
</head>
<body>
    <h1>Hello from my custom Docker website!</h1>
    <p>This static website is running inside an Nginx Docker container.</p>
    <p>Day 31 - Dockerfile practice</p>
</body>
</html>

## Dockerfile
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/index.html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]

## Commands
cd ~/2026/day-31
mkdir -p my-website
cd my-website

cat > index.html <<'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>My Docker Website</title>
</head>
<body>
    <h1>Hello from my custom Docker website!</h1>
    <p>This static website is running inside an Nginx Docker container.</p>
    <p>Day 31 - Dockerfile practice</p>
</body>
</html>
EOF

cat > Dockerfile <<'EOF'
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/index.html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
EOF

docker build -t my-website:v1 .
docker run -d --name my-website-container -p 8083:80 my-website:v1
docker ps

Open browser:
http://localhost:8083

## Logs
docker logs my-website-container

Observed:
GET / HTTP/1.1" 200
GET /favicon.ico HTTP/1.1" 404

Explanation:

- `200` means homepage loaded successfully.
- `404` for favicon is normal because no `favicon.ico` was created.

## Verify File Inside Container
docker exec -it my-website-container sh
cat /usr/share/nginx/html/index.html
exit

Observed output included:
<h1>Hello from my custom Docker website!</h1>
Nginx serves static files from:
/usr/share/nginx/html/
So copying `index.html` there made Nginx serve our custom website.

# Task 5: .dockerignore

## What is `.dockerignore`?

`.dockerignore` tells Docker which files/folders to exclude from build context.
It works like `.gitignore`, but for Docker builds.

Example:
node_modules
.git
*.md
.env

## Why `.dockerignore` is Important

1. Faster builds: Docker does not send unnecessary files to build context.
2. Smaller images: Unnecessary files are not copied into the image.
3. Security: Secret files like `.env` are not included in Docker images.

Important: Never copy secrets into Docker images.

## Folder Structure
2026/day-31/dockerignore-demo/
├── Dockerfile
├── .dockerignore
├── app.txt
├── README.md
├── .env
├── node_modules/
│   └── dummy.txt
└── .git/
    └── dummy.txt

## .dockerignore
node_modules
.git
*.md
.env

## Dockerfile
FROM alpine
WORKDIR /app
COPY . .
CMD ["ls", "-la", "/app"]


## Commands
cd ~/2026/day-31
mkdir -p dockerignore-demo
cd dockerignore-demo

mkdir -p node_modules .git

cat > app.txt <<'EOF'
This file should be copied into Docker image.
EOF

cat > README.md <<'EOF'
This markdown file should be ignored by Docker.
EOF

cat > .env <<'EOF'
SECRET_KEY=super-secret-value
DB_PASSWORD=do-not-copy-this
EOF

cat > node_modules/dummy.txt <<'EOF'
This node_modules file should be ignored.
EOF

cat > .git/dummy.txt <<'EOF'
This git file should be ignored.
EOF

cat > .dockerignore <<'EOF'
node_modules
.git
*.md
.env
EOF

cat > Dockerfile <<'EOF'
FROM alpine
WORKDIR /app
COPY . .
CMD ["ls", "-la", "/app"]
EOF

docker build -t dockerignore-demo:v1 .
docker run dockerignore-demo:v1

Observed output:
.dockerignore
Dockerfile
app.txt

Ignored files were not present:
.env
node_modules
.git
README.md

## Verification Commands

docker run dockerignore-demo:v1 sh -c "test -f /app/.env && echo env-found || echo env-not-found"
docker run dockerignore-demo:v1 sh -c "test -d /app/node_modules && echo node_modules-found || echo node_modules-not-found"
docker run dockerignore-demo:v1 sh -c "test -d /app/.git && echo git-found || echo git-not-found"
docker run dockerignore-demo:v1 sh -c "test -f /app/README.md && echo readme-found || echo readme-not-found"

Observed:
env-not-found
node_modules-not-found
git-not-found
readme-not-found

# Task 6: Build Optimization

## Goal

Understand Docker build cache and why Dockerfile layer order matters.

## Docker Build Cache

Docker uses layer caching. Each Dockerfile instruction can create a layer. If the instruction and its input files do not change, Docker reuses the cached layer.

You may see:
CACHED
during build.

## Why Layer Order Matters

Docker builds top to bottom. If one layer changes, Docker rebuilds that layer and all layers after it.

Bad example:
dockerfile
FROM node:20
WORKDIR /app
COPY . .
RUN npm install
CMD ["npm", "start"]


Problem:
Small source code change
↓
COPY . . changes
↓
RUN npm install runs again
↓
Build becomes slow

Better example:

dockerfile
FROM node:20
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
CMD ["npm", "start"]


Benefit:
Only source code changed
↓
package.json unchanged
↓
npm install layer reused from cache
↓
Build is faster

Rule: put rarely changing instructions first and frequently changing instructions later.

## Folder Structure

2026/day-31/build-optimization/
├── bad.Dockerfile
├── good.Dockerfile
├── package.txt
└── app.txt

## Bad Dockerfile
dockerfile
FROM alpine

WORKDIR /app

COPY . .

RUN echo "Installing dependencies from package.txt" && cat package.txt

CMD ["cat", "app.txt"]

## Good Dockerfile
dockerfile
FROM alpine
WORKDIR /app
COPY package.txt .
RUN echo "Installing dependencies from package.txt" && cat package.txt
COPY app.txt .
CMD ["cat", "app.txt"]


## Commands
bash
cd ~/2026/day-31
mkdir -p build-optimization
cd build-optimization

cat > package.txt <<'EOF'
curl
EOF

cat > app.txt <<'EOF'
Version 1: Hello from optimized Docker build!
EOF

cat > bad.Dockerfile <<'EOF'
FROM alpine
WORKDIR /app
COPY . .
RUN echo "Installing dependencies from package.txt" && cat package.txt
CMD ["cat", "app.txt"]
EOF

cat > good.Dockerfile <<'EOF'
FROM alpine
WORKDIR /app
COPY package.txt .
RUN echo "Installing dependencies from package.txt" && cat package.txt
COPY app.txt .
CMD ["cat", "app.txt"]
EOF

docker build -f bad.Dockerfile -t bad-cache-demo:v1 .
docker build -f bad.Dockerfile -t bad-cache-demo:v1 .

cat > app.txt <<'EOF'
Version 2: I changed only the app file.
EOF

docker build -f bad.Dockerfile -t bad-cache-demo:v2 .

cat > app.txt <<'EOF'
Version 1: Hello from optimized Docker build!
EOF

docker build -f good.Dockerfile -t good-cache-demo:v1 .
docker build -f good.Dockerfile -t good-cache-demo:v1 .

cat > app.txt <<'EOF'
Version 2: I changed only the app file.
EOF

docker build -f good.Dockerfile -t good-cache-demo:v2 .

docker run bad-cache-demo:v2
docker run good-cache-demo:v2

Observed:
Version 2: I changed only the app file.
Version 2: I changed only the app file.

## Build Optimization Learning

Bad Dockerfile behavior:
app.txt changed
↓
COPY . . rebuilt
↓
RUN dependency step rebuilt
↓
slower build

Good Dockerfile behavior:
app.txt changed
↓
COPY package.txt . cached
↓
RUN dependency step cached
↓
COPY app.txt . rebuilt
↓
faster build

# All Dockerfiles Created

## 1. my-first-image/Dockerfile
dockerfile
FROM ubuntu

RUN apt update && apt install -y curl

CMD ["echo", "Hello from my custom image!"]

## 2. dockerfile-instructions/Dockerfile
dockerfile
FROM ubuntu

RUN apt update && apt install -y curl

WORKDIR /app

COPY message.txt /app/message.txt

EXPOSE 8080

CMD ["cat", "/app/message.txt"]

## 3. cmd-demo/Dockerfile
dockerfile
FROM alpine

CMD ["echo", "hello"]

## 4. entrypoint-demo/Dockerfile
dockerfile
FROM alpine

ENTRYPOINT ["echo"]

## 5. my-website/Dockerfile
dockerfile
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/index.html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]

## 6. dockerignore-demo/Dockerfile
dockerfile
FROM alpine
WORKDIR /app
COPY . .
CMD ["ls", "-la", "/app"]


## 7. dockerignore-demo/.dockerignore
node_modules
.git
*.md
.env

## 8. build-optimization/bad.Dockerfile
dockerfile
FROM alpine
WORKDIR /app
COPY . .
RUN echo "Installing dependencies from package.txt" && cat package.txt
CMD ["cat", "app.txt"]


## 9. build-optimization/good.Dockerfile
dockerfile
FROM alpine
WORKDIR /app
COPY package.txt .
RUN echo "Installing dependencies from package.txt" && cat package.txt
COPY app.txt .
CMD ["cat", "app.txt"]


# Common Commands Used
docker build -t image-name:tag .
docker build -f Dockerfile-name -t image-name:tag .
docker run image-name:tag
docker run -d --name container-name -p host-port:container-port image-name:tag
docker images
docker ps
docker ps -a
docker logs container-name
docker exec -it container-name sh

## 1. Dockerfile Change Not Reflected

If Dockerfile is changed but container output is still old, rebuild image:
docker build -t image-name:tag .

Docker image does not change automatically after editing Dockerfile.

## 2. Port Not Opening in Browser

Check:
docker ps
docker logs container-name
Make sure port mapping exists:
0.0.0.0:8083->80/tcp

# Practice Questions

## Basic Questions

1. What is a Dockerfile?
2. What is the purpose of `FROM`?
3. What is the difference between `RUN` and `CMD`?
4. What does `COPY` do?
5. What is `WORKDIR` used for?
6. Does `EXPOSE` publish a port?
7. What command builds a Docker image?
8. What does the `.` mean in `docker build -t image:v1 .`?

## Scenario-Based Questions

1. Your Dockerfile changed, but container output is still old. What should you do?
2. Your image build is very slow because `node_modules` is being sent to Docker. What should you add?
3. Your `.env` file is copied into the Docker image. Why is this dangerous?
4. You changed only source code, but dependency installation runs again. What is wrong with Dockerfile ordering?
5. Your container runs Nginx but browser cannot access it. What should you check?
6. You want to always run a fixed executable but pass arguments dynamically. Should you use CMD or ENTRYPOINT?
7. You want users to override the default command easily. Should you use CMD or ENTRYPOINT?


# Interview Questions and Answers

## Q1. What is a Dockerfile?

A Dockerfile is a text file that contains instructions to build a Docker image. It defines the base image, dependencies, files to copy, working directory, exposed ports, and default command.

## Q2. What is the difference between Dockerfile, image, and container?

A Dockerfile is the recipe. A Docker image is the package built from the Dockerfile. A Docker container is a running or stopped instance of that image.

Dockerfile → Docker Image → Docker Container

## Q3. What is the purpose of `FROM`?

`FROM` defines the base image for the Docker image.
FROM ubuntu

This means the custom image will be built on top of Ubuntu.

## Q4. What is the difference between `RUN` and `CMD`?

`RUN` executes commands during image build time and creates image layers. `CMD` defines the default command that runs when a container starts.

## Q5. What is the difference between `CMD` and `ENTRYPOINT`?

`CMD` provides a default command that can be easily overridden. `ENTRYPOINT` defines a fixed executable. Arguments passed in `docker run` are added to ENTRYPOINT.

Use CMD for default behavior. Use ENTRYPOINT when the container should always run a specific executable.

## Q6. Give an example of CMD being overridden.

Dockerfile:
FROM alpine
CMD ["echo", "hello"]

Run normally:

docker run cmd-demo:v1

Output:
hello

Override CMD:
docker run cmd-demo:v1 echo "custom message"

Output:
custom message

## Q7. Give an example of ENTRYPOINT.

Dockerfile:
FROM alpine
ENTRYPOINT ["echo"]

Run:
docker run entrypoint-demo:v1 hello


Output:
hello

Here `echo` is fixed and `hello` is passed as an argument.

## Q8. What is `.dockerignore`?

`.dockerignore` is a file that tells Docker which files and folders to exclude from the build context. It improves build speed, reduces image size, and prevents secrets from entering Docker images.

## Q9. Why should `.env` be added to `.dockerignore`?

`.env` files often contain secrets like database passwords, API keys, JWT secrets, and cloud credentials. If copied into the image, those secrets can be exposed to anyone who can access the image.

## Q10. What is Docker build context?

Build context is the set of files sent to Docker daemon during image build. In `docker build -t my-image:v1 .`, the `.` means the current folder is the build context.

## Q11. What is Docker layer caching?

Docker layer caching means Docker reuses unchanged layers from previous builds. If an instruction and its input files have not changed, Docker uses cache instead of rebuilding that layer. This makes builds faster.

## Q12. Why does Dockerfile instruction order matter?

Docker builds instructions from top to bottom. If one layer changes, Docker rebuilds that layer and all layers after it. Rarely changing instructions should be placed earlier, and frequently changing instructions should be placed later.

Good order:
dockerfile
COPY package*.json ./
RUN npm install
COPY . .

## Q13. What is the difference between `EXPOSE` and `-p`?

`EXPOSE` only documents the port that the container listens on. `-p` actually maps the container port to the host port.
dockerfile
EXPOSE 80

Only documentation.
docker run -p 8083:80 my-website:v1
Actual port mapping.

## Q14. Why does Nginx Dockerfile use `daemon off;`?

Docker containers need a foreground process to keep running. By default, Nginx can run as a background daemon. `daemon off;` keeps Nginx running in the foreground, so the container stays alive.

## Q15. How do you build an image with a custom Dockerfile name?

Use `-f`.
docker build -f good.Dockerfile -t good-cache-demo:v1 .

# Real-World DevOps Best Practices

## 1. Use Small Base Images

Prefer small images where appropriate:
dockerfile
FROM nginx:alpine
FROM node:20-alpine
FROM python:3.12-slim

But do not use Alpine blindly if your app has compatibility issues.

## 2. Use `.dockerignore`

Always ignore unnecessary or sensitive files:
node_modules
.git
.env
logs
coverage
*.md

## 3. Optimize Layer Order

Bad:
dockerfile
COPY . .
RUN npm install

Good:
dockerfile
COPY package*.json ./
RUN npm install
COPY . .


## 4. Do Not Store Secrets in Images

Never copy `.env`, AWS credentials, private keys, database passwords, or API keys into images. Use environment variables, Docker secrets, Kubernetes secrets, or cloud secret managers.

## 5. Keep Containers Single-Purpose

One container should usually run one main process.

Example:

frontend container
backend API container
database container
redis container
worker container

# Final Revision Notes

- Dockerfile is used to build Docker images.
- `FROM` defines base image.
- `RUN` runs commands during image build.
- `COPY` copies files into image.
- `WORKDIR` sets working directory.
- `EXPOSE` documents container port.
- `CMD` sets default command.
- `ENTRYPOINT` sets fixed executable.
- `CMD` can be overridden easily.
- `ENTRYPOINT` receives arguments from `docker run`.
- `.dockerignore` excludes files from build context.
- Docker build context is the folder sent to Docker daemon.
- Docker uses layer caching for faster builds.
- Layer order matters for build speed.
- Put rarely changing layers first.
- Put frequently changing files later.
- Never copy `.env` or secrets into images.

Also include the Dockerfiles created in each task folder.
