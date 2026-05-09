# Day 29 – Docker Basics
## Goal
Today's goal was to understand what Docker is and run the first containers.
## Completed Tasks
- Learned why containers exist
- Compared containers with virtual machines
- Understood Docker architecture
- Installed and verified Docker
- Ran the `hello-world` container
- Ran an Nginx container and accessed it in the browser
- Ran an Ubuntu container in interactive mode
- Listed running and stopped containers
- Checked container logs
- Executed commands inside a running container
- Stopped and removed containers
- 
# Task 1: What is Docker?

## What is Docker?

Docker is a containerization platform used to package and run applications with all required dependencies.

In simple words:

> Docker helps us run applications in the same environment everywhere: local machine, testing server, staging, and production.

It solves the common problem:

> “It works on my machine, but not on the server.”

## What is a Container?

A container is a lightweight, isolated environment where an application runs.

A container includes:

- Application code
- Runtime such as Node.js, PHP, Python, Java, etc.
- Required libraries
- System dependencies
- Configuration

A container shares the host operating system kernel, so it is lightweight compared to a virtual machine.
Example:
docker run nginx

This runs the Nginx web server inside a Docker container.

## Why Do We Need Containers?

Containers are useful because they provide:

### 1. Same Environment Everywhere

The application runs the same way on:

- Developer machine
- Testing server
- Production server

### 2. Fast Startup

Containers start in seconds.

### 3. Easy Deployment

Docker images can be built once and deployed anywhere.

### 4. Better CI/CD

Docker is widely used in CI/CD pipelines.

Example flow:
Code pushed to Git
        ↓
CI/CD pipeline builds Docker image
        ↓
Image pushed to Docker registry
        ↓
Server pulls image
        ↓
Container runs application

### 5. Microservices Support

Each service can run in its own container.

Example:
Frontend container
Backend API container
Database container
Redis container
Queue worker container

# Containers vs Virtual Machines

| Point | Container | Virtual Machine |
| OS | Shares host OS kernel | Has its own full OS |
| Startup | Starts in seconds | Takes more time |
| Size | Lightweight, usually MBs | Heavy, usually GBs |
| Performance | Fast | Comparatively slower |
| Resource Usage | Less CPU/RAM | More CPU/RAM |
| Isolation | Process-level isolation | Full OS-level isolation |
| Best For | Apps, microservices, CI/CD | Full OS isolation, legacy workloads |

## Simple Difference

A virtual machine is like a full rented house.

A container is like an isolated room with only the required things inside.

# Docker Architecture

Docker architecture has the following main components:

## 1. Docker Client

Docker Client is the command-line tool used to run Docker commands.

Example:
docker run nginx
docker ps
docker stop my-nginx

The Docker Client sends requests to the Docker Daemon.

## 2. Docker Daemon

Docker Daemon is the background service that does the actual work.

It manages:

- Images
- Containers
- Networks
- Volumes
- Builds
- Pulls from registry

## 3. Docker Image

A Docker image is a read-only template used to create containers.

Examples:
nginx
ubuntu
node
mysql
redis
mongo
Simple understanding:
Image = template
Container = running instance of image

## 4. Docker Container

A container is a running or stopped instance of an image.

Example:

docker run --name my-nginx nginx

## 5. Docker Registry

A Docker registry stores Docker images.

Most common registry:

Docker Hub

Example:

docker pull nginx
docker pull ubuntu

## Docker Architecture Diagram
User / Terminal
     |
     | docker run nginx
     v
Docker Client
     |
     | API request
     v
Docker Daemon
     |
     | checks local image
     v
Docker Registry / Docker Hub
     |
     | pulls image if not available
     v
Docker Image
     |
     | creates container
     v
Docker Container

## Docker Architecture in My Own Words

When I run a command like:

docker run nginx

Docker Client sends this request to Docker Daemon.  
Docker Daemon checks whether the Nginx image exists locally.  
If the image is not available, Docker pulls it from Docker Hub.  
After that, Docker creates and starts a container from that image.

# Task 2: Install Docker

## Verify Docker Installation

docker --version
docker info

## Run Hello World Container

Command:
docker run hello-world

## Output Summary

Docker successfully printed:
Hello from Docker!
This message shows that your installation appears to be working correctly.

## What Happened?

Docker performed these steps:

1. Docker Client contacted Docker Daemon.
2. Docker Daemon checked for the `hello-world` image locally.
3. Image was not found locally.
4. Docker pulled the image from Docker Hub.
5. Docker created a container from the image.
6. Container printed the output.
7. Container exited successfully.

# Task 3: Run Real Containers

## Run Nginx Container

Command:

docker run -d --name my-nginx -p 8080:80 nginx

## Command Explanation

| Part | Meaning |
|---|---|
| `docker run` | Create and start a new container |
| `-d` | Detached mode, run in background |
| `--name my-nginx` | Give custom container name |
| `-p 8080:80` | Map host port 8080 to container port 80 |
| `nginx` | Image name |

## Browser Test

URL:
http://localhost:8080

Result:

Nginx welcome page opened successfully.

## List Running Containers

Command:
docker ps
Example output showed:
nginx   Up   0.0.0.0:8080->80/tcp   my-nginx

Meaning:

- Nginx container was running
- Host port 8080 was mapped to container port 80
- Container name was `my-nginx`

## Check Container Logs

Command:

docker logs my-nginx

Observed log:
GET / HTTP/1.1" 200
GET /favicon.ico HTTP/1.1" 404

## Log Explanation

| Status | Meaning |
|---|---|
| `200` | Request successful |
| `404` | File not found |

The `favicon.ico` 404 is normal because the browser automatically tries to load a favicon.


## Run Command Inside Running Container

Command:
docker exec -it my-nginx sh

Inside container, commands used:
pwd
ls
whoami
cat /etc/os-release
ls /usr/share/nginx/html
cat /usr/share/nginx/html/index.html
exit

## Learning

The Nginx container had its own Linux filesystem.

Nginx default website files were located at:

/usr/share/nginx/html

The default page file was:

/usr/share/nginx/html/index.html

## Run Ubuntu Container in Interactive Mode

Command:
docker run -it ubuntu bash

Inside Ubuntu container:
pwd
ls
whoami
cat /etc/os-release
apt update
apt install -y curl
curl --version
exit

## Learning

Ubuntu container behaved like a mini Linux machine.

After running `exit`, the Ubuntu container stopped because its main process was `bash`.

Important point:

> A Docker container runs only while its main process is running.

## List All Containers

Command:
docker ps -a

This showed:

- Running Nginx container
- Stopped Ubuntu container
- Stopped hello-world container

## List Docker Images

Command:
docker images
Images available locally:

hello-world:latest
nginx:latest
ubuntu:latest

# Task 4: Explore Docker

## Run Container in Detached Mode

Command:
docker run -d --name custom-nginx -p 9090:80 nginx

## What is Detached Mode?

Detached mode means the container runs in the background and terminal becomes free.

Flag:
-d

## Give Container a Custom Name

Command:
docker run -d --name custom-nginx nginx

Flag:
--name
## Map Host Port to Container Port

Command:
docker run -d --name custom-nginx -p 9090:80 nginx
Meaning:
Host port 9090 → Container port 80
Browser URL:
http://localhost:9090

## Check Logs

Command:
docker logs custom-nginx

## Run Command Inside Running Container

Command:
docker exec -it custom-nginx sh
Inside container:
hostname
pwd
ls /usr/share/nginx/html
exit
## Stop Container
Command:

docker stop custom-nginx

## Remove Container

Command:
docker rm custom-nginx

## Verify Cleanup

Commands:

docker ps
docker ps -a
Result:
- No running containers
- Stopped Ubuntu and hello-world containers remained
- 
# Important Docker Commands

## Version and Info

docker --version
docker info
## Run Container

docker run IMAGE_NAME
Example:

docker run hello-world

## Run Container in Background
docker run -d IMAGE_NAME

## Run Container with Name
docker run --name my-container IMAGE_NAME

## Run Container with Port Mapping
docker run -d --name my-nginx -p 8080:80 nginx

## Run Interactive Container
docker run -it ubuntu bash

## List Running Containers
docker ps

## List All Containers
docker ps -a

## List Images
docker images

## Stop Container
docker stop CONTAINER_NAME

## Remove Container
docker rm CONTAINER_NAME

## Force Remove Container
docker rm -f CONTAINER_NAME

## View Logs
docker logs CONTAINER_NAME

## Follow Logs
docker logs -f CONTAINER_NAME

## Execute Command Inside Container
docker exec -it CONTAINER_NAME sh

or:
docker exec -it CONTAINER_NAME bash

# Why This Matters for DevOps

Docker is the foundation of modern DevOps.
It is used in:

- CI/CD pipelines
- Kubernetes
- Microservices
- Cloud deployments
- Local development environments
- Testing environments
- Production deployments

Real-world flow:
Developer writes code
        ↓
Docker image is built
        ↓
Image pushed to registry
        ↓
Server/Kubernetes pulls image
        ↓
Container runs application

# Practice Questions

## Basic Questions

1. What is Docker?
2. What is a container?
3. What is a Docker image?
4. What is the difference between image and container?
5. What does `docker ps` show?
6. What does `docker ps -a` show?
7. What is Docker Hub?

## Scenario-Based Questions

8. You ran an Ubuntu container using `docker run -it ubuntu bash`. After typing `exit`, the container stopped. Why?
9. Your Nginx container is running on port 80 inside the container. You want to access it on localhost port 8080. Which command will you use?
10. You want to check logs of a running container named `api-server`. Which command will you use?

## Interview-Level Questions

11. What is the difference between containers and virtual machines?
12. Why are containers useful in CI/CD?
13. What happens internally when you run `docker run nginx`?
14. Why should production containers avoid running as root?
15. What is the role of Docker Daemon?

# Real DevOps Interview Questions

## Question 1

What happens when you run this command?

docker run -d --name web -p 8080:80 nginx

## Sample Answer

This command creates and starts an Nginx container in detached mode.  
The container name is `web`.  
It maps host port `8080` to container port `80`, so the Nginx web page can be accessed using `http://localhost:8080`.

## Question 2

What is the difference between Docker image and Docker container?

## Sample Answer

A Docker image is a read-only template that contains application code and dependencies.  
A Docker container is a running or stopped instance created from an image.

## Question 3

Why does an Ubuntu container stop after running `exit`?

## Sample Answer

The main process of the Ubuntu container is `bash`.  
When we type `exit`, the bash process ends.  
A Docker container runs only while its main process is running, so the container stops.

## Question 4

What is port mapping in Docker?

## Sample Answer

Port mapping connects a port on the host machine to a port inside the container.

Example:

docker run -d -p 8080:80 nginx

This maps host port `8080` to container port `80`.

## Question 5

How do you debug a running container?

## Sample Answer

I can debug a running container using logs and exec commands.

Examples:
docker logs container-name
docker exec -it container-name sh

`docker logs` helps check application logs.  
`docker exec` helps enter the container and inspect files, processes, or configuration.

# Revision Notes

## Key Points

- Docker is a containerization platform.
- Container is a lightweight isolated environment.
- Image is a template.
- Container is an instance of an image.
- Docker Client sends commands.
- Docker Daemon performs actual Docker operations.
- Docker Hub stores public Docker images.
- Containers are lightweight compared to virtual machines.
- Containers are widely used in DevOps, CI/CD, Kubernetes, and microservices.
- A container runs only while its main process is running.

## Important Commands
docker --version
docker info
docker run hello-world
docker pull nginx
docker run -d --name my-nginx -p 8080:80 nginx
docker ps
docker ps -a
docker images
docker logs my-nginx
docker exec -it my-nginx sh
docker run -it ubuntu bash
docker stop my-nginx
docker rm my-nginx
docker rm -f my-nginx

## Most Important Command Pattern
docker run -d --name container-name -p host-port:container-port image-name
Example:
docker run -d --name my-nginx -p 8080:80 nginx
