# Day 30 - Docker Images & Container Lifecycle

> #90DaysOfDevOps - Day 30

# Learning Objectives

After completing this lab, you will be able to:
- Understand Docker Images
- Understand the relationship between Images and Containers
- Pull images from Docker Hub
- Compare Ubuntu and Alpine images
- Inspect Docker images
- Remove Docker images
- Understand Docker Image Layers
- Understand the complete Container Lifecycle
- Work with running containers
- View logs and inspect containers
- Perform Docker cleanup
- Prepare for Docker interview questions

# Prerequisites Revision (Day 29)

Before starting Day 30, you should remember these concepts.

## What is Docker?

**Definition**

Docker is a platform used to build, ship, and run applications inside containers.

**Simple Explanation**

Docker helps developers package an application with everything it needs (code, libraries, dependencies, configuration) so it runs the same everywhere.

## What is Docker Engine?

Docker Engine is the software responsible for creating, running, and managing Docker containers.

## What is Docker Hub?

Docker Hub is an online repository used to download and share Docker images.

Official Website:

https://hub.docker.com

## What is Docker Image?

**Definition**

A Docker Image is a **Read-Only Template** used to create Containers.

Remember this definition throughout the Docker course.

## What is Docker Container?

**Definition**

A Docker Container is a **Running Instance of a Docker Image**.

## Docker Image vs Docker Container

| Docker Image | Docker Container |
|--------------|------------------|
| Read-Only Template | Running Instance |
| Stored on Disk | Runs in Memory |
| Used to Create Containers | Created from Image |
| Cannot Run Directly | Running Application |

# Task 1 - Docker Images

## Objective
Learn how Docker Images work.
You will learn:

- Pull Images
- List Images
- Compare Image Sizes
- Inspect Images
- Remove Images

# What is a Docker Image?

## Definition

A Docker Image is a **Read-Only Template** used to create Docker Containers.

This is the only definition you need to remember.
## Real Life Example

Think of a house blueprint.

Blueprint

↓

Build House

Docker works exactly the same.

Docker Image

↓

Docker Container

One Image can create multiple Containers.
Docker Image
      │
      ├── Container 1
      ├── Container 2
      └── Container 3

# Where Do Images Come From?
Images are downloaded from Docker Hub.
Docker Hub
      │
      ▼
docker pull
      │
      ▼
Local Machine
      │
      ▼
docker run
      │
      ▼
Container

# Task 1.1 - Pull Docker Images

## Command
docker pull nginx
### Explanation

| Command | Description |
|----------|-------------|
| docker | Docker CLI |
| pull | Download Image |
| nginx | Image Name |

## Pull Ubuntu
docker pull ubuntu

## Pull Alpine
docker pull alpine

# Expected Output
Using default tag: latest
latest: Pulling from library/nginx
Status: Downloaded newer image


# Output Explanation
## Using default tag: latest
Docker automatically downloads the **latest version** if no tag is specified.
Example
docker pull nginx
is equal to
docker pull nginx:latest


## Pulling from library/nginx
Docker is downloading the official Nginx image from Docker Hub.

## Downloaded newer image
The image has been successfully downloaded to your local machine.

# List All Images
Command
docker images
or
docker image ls

Both commands are identical.
# Example Output
REPOSITORY     TAG      IMAGE ID      CREATED      SIZE

nginx          latest   xxxxx         2 weeks      192MB

ubuntu         latest   xxxxx         4 weeks      80MB

alpine         latest   xxxxx         8MB

# Understanding Output

| Column | Description |
|----------|-------------|
| Repository | Image Name |
| Tag | Version |
| Image ID | Unique Image Identifier |
| Created | Image Creation Time |
| Size | Image Size |

# Ubuntu vs Alpine

One common interview question is:

**Why is Alpine much smaller than Ubuntu?**

## Ubuntu
- Full Linux Distribution
- More Packages
- More Libraries
- Beginner Friendly

Approximate Size

70–90 MB

## Alpine
- Minimal Linux Distribution
- Lightweight
- Small Download Size
- Fast Startup
- Production Friendly
Approximate Size
5–10 MB

# Interview Answer
Why is Alpine smaller?
Because Alpine contains only the minimum required packages and libraries, while Ubuntu includes many additional tools and packages.
Therefore Alpine images are much smaller.

# Inspect Docker Image
Command
docker inspect nginx
or
docker image inspect nginx

Both commands display detailed image information.

# Important Fields in docker inspect
You do NOT need to remember the entire JSON output.
Only remember these important fields.

## Image ID
Id
Unique identifier of the image.

## RepoTags
nginx:latest
Image Name and Version.
## Created
Shows when the image was built.

## ExposedPorts
Example
80/tcp
This tells Docker that the application uses Port 80.
It DOES NOT publish the port.
Port publishing happens using
docker run -p


## Environment Variables
Example
NGINX_VERSION

PATH

Environment Variables provide configuration information to the application.

## Entrypoint
Example
/docker-entrypoint.sh
This script runs first when the container starts.

## CMD
Example
nginx -g "daemon off;"

This is the default command executed when the container starts.

## Operating System
Example
Linux
Even if Docker Desktop is running on Windows, Linux containers still use the Linux kernel through WSL2.

## Architecture
Example
amd64
The image is built for 64-bit x86 processors.

## RootFS
Shows all image layers.
This concept will be covered in Task 2.

# Remove Docker Image
Command
docker image rm alpine
or
docker rmi alpine
Both commands remove a Docker image.

# Important Note
Docker will not remove an image if a container is using it.
Example Error
conflict:
unable to delete image
image is being used by stopped container
Solution
Remove the container first.
Then remove the image.

# Production Notes

- Use official images whenever possible.
- Keep images small for faster deployment.
- Prefer Alpine images in production when suitable.
- Never delete Kubernetes (KIND) images unless you no longer need the cluster.
- Always inspect images before using them in production.

# Commands Learned
docker pull nginx

docker pull ubuntu

docker pull alpine

docker images

docker image ls

docker inspect nginx

docker image inspect nginx

docker image rm alpine

docker rmi alpine

# Part 1 Summary
In this section you learned:
- Docker Image
- Docker Hub
- Pull Images
- List Images
- Compare Ubuntu and Alpine
- Inspect Images
- Remove Images
The next section covers **Docker Image Layers**, one of the most important Docker concepts.

# Task 2 - Docker Image Layers

## Objective

In this task, you will learn:

- What is a Docker Image Layer?
- Why Docker uses Layers
- How to view Image Layers
- Understand `docker image history`
- Why some layers show `0B`
- What is Docker Build Cache
- Layer Reuse
- Production Best Practices

---

# What is a Docker Image Layer?

## Definition

A **Docker Image Layer** is a **Read-Only Layer** of a Docker Image.

Multiple layers together create a complete Docker Image.

> **Remember**
>
> **Docker Image = Multiple Read-Only Layers**

---

# Simple Example

Imagine building a burger.

```
Bread
   ↓
Cheese
   ↓
Patty
   ↓
Sauce
   ↓
Vegetables
   ↓
Burger
```

Docker Image works in the same way.

```
Base Linux
      ↓
Install Packages
      ↓
Install Nginx
      ↓
Copy Files
      ↓
Configuration
      ↓
Docker Image
```

Each step creates **one Layer**.

---

# Why Does Docker Use Layers?

Without layers, every image would store everything from the beginning.

Example

```
Ubuntu
   ↓
Install Nginx
```

and

```
Ubuntu
   ↓
Install Apache
```

If Docker didn't use layers, Ubuntu would be stored twice.

Instead Docker shares the Ubuntu layer.

```
Ubuntu Layer
      │
      ├── Nginx Image
      │
      └── Apache Image
```

## Benefits

- Saves Disk Space
- Faster Downloads
- Faster Builds
- Reuses Existing Layers

---

# Image Layer Example

A simplified Nginx Image looks like this.

```
Layer 5 → Configuration

Layer 4 → Nginx Package

Layer 3 → Required Libraries

Layer 2 → Linux Packages

Layer 1 → Debian Base Image
```

The actual number of layers depends on how the image was built.

---

# View Image Layers

Command

```bash
docker image history nginx
```

---

# Command Explanation

| Command | Description |
|----------|-------------|
| docker | Docker CLI |
| image | Image Operation |
| history | Show Image Layers |
| nginx | Image Name |

---

# Example Output

```
IMAGE        CREATED      CREATED BY             SIZE

xxxxx        2 days ago   CMD nginx             0B

xxxxx        2 days ago   EXPOSE 80             0B

xxxxx        2 days ago   COPY files            16KB

xxxxx        2 days ago   RUN apt install       87MB

xxxxx        2 days ago   FROM debian           87MB
```

Your output may look different depending on the image version.

---

# Understanding the Output

| Column | Meaning |
|----------|---------|
| IMAGE | Layer ID |
| CREATED | Layer Creation Time |
| CREATED BY | Dockerfile Instruction |
| SIZE | Layer Size |

---

# Reading Image History

Always read the history **from Bottom to Top**.

```
Debian Base Image
        ↓
RUN Install Packages
        ↓
COPY Files
        ↓
ENV Variables
        ↓
ENTRYPOINT
        ↓
EXPOSE Port
        ↓
CMD
        ↓
Final Image
```

The bottom layer is created first.

The top layer is created last.

---

# Understanding Important Layers

## Base Image

Example

```
FROM debian
```

Every Docker Image starts from a Base Image.

Common Base Images:

- alpine
- ubuntu
- debian
- python
- node
- openjdk

---

## RUN Layer

Example

```dockerfile
RUN apt-get update
RUN apt-get install nginx
```

Purpose

- Install software
- Create users
- Download packages
- Configure the system

RUN usually creates the largest layers.

---

## COPY Layer

Example

```dockerfile
COPY index.html /usr/share/nginx/html
```

Purpose

Copy files from your local machine into the image.

COPY increases image size.

---

## ENV Layer

Example

```dockerfile
ENV APP_ENV=production
```

Purpose

Set Environment Variables.

Usually shows **0B** because it stores metadata, not files.

---

## LABEL Layer

Example

```dockerfile
LABEL maintainer="admin@example.com"
```

Purpose

Stores image metadata.

Usually **0B**.

---

## ENTRYPOINT Layer

Example

```dockerfile
ENTRYPOINT ["/docker-entrypoint.sh"]
```

Purpose

Defines the startup program for the container.

Usually **0B**.

---

## EXPOSE Layer

Example

```dockerfile
EXPOSE 80
```

Purpose

Documents which port the application uses.

It **does not publish** the port.

Usually **0B**.

---

## CMD Layer

Example

```dockerfile
CMD ["nginx","-g","daemon off;"]
```

Purpose

Defines the default command executed when the container starts.

Usually **0B**.

---

# Why Do Some Layers Show 0B?

Many beginners think this is an error.

It is completely normal.

Commands like

- CMD
- ENV
- LABEL
- EXPOSE
- ENTRYPOINT

only store configuration or metadata.

They do not copy files or install software.

Therefore their size is usually shown as **0B**.

---

# Commands That Usually Increase Image Size

These instructions normally create larger layers.

```dockerfile
RUN

COPY

ADD
```

---

# Commands That Usually Show 0B

```dockerfile
CMD

ENV

LABEL

ENTRYPOINT

EXPOSE
```

> **Note**
>
> There can be exceptions depending on how the image is built, but for beginners this rule is sufficient.

---

# Why Do We See `<missing>`?

Example

```
<missing>
```

This is **not an error**.

Docker removes intermediate image IDs after the final image is built.

The layer still exists.

Only the temporary image ID is no longer available.

---

# Docker Build Cache

One of Docker's biggest advantages is **Build Cache**.

Example

Suppose you have this Dockerfile.

```dockerfile
FROM ubuntu

RUN apt update

RUN apt install nginx

COPY index.html /var/www/html
```

Docker builds four layers.

Later you only change **index.html**.

Docker will reuse:

- FROM
- RUN apt update
- RUN apt install nginx

Only the COPY layer is rebuilt.

This makes Docker builds much faster.

---

# Layer Reuse

Imagine five applications using Ubuntu.

```
Ubuntu Layer
      │
      ├── App 1
      ├── App 2
      ├── App 3
      ├── App 4
      └── App 5
```

Docker stores the Ubuntu layer only once.

Every application reuses the same layer.

Benefits:

- Less Storage
- Faster Downloads
- Faster Builds

---

# Production Notes

- Choose a small base image whenever possible.
- Keep the number of layers low.
- Combine multiple RUN commands when appropriate.
- Remove temporary files during image build.
- Reuse existing layers to improve build performance.
- Use Docker Build Cache to reduce build time.

---

# Best Practices

✅ Use official base images.

✅ Prefer Alpine when the application supports it.

✅ Copy only required files.

✅ Avoid unnecessary packages.

✅ Keep Dockerfiles clean and readable.

---

# Common Mistakes

❌ Thinking an Image is one single file.

❌ Reading image history from top to bottom.

❌ Assuming `0B` means an error.

❌ Using very large base images without a reason.

❌ Ignoring Docker Build Cache.

---

# Commands Learned

```bash
docker image history nginx
```

---

# Quick Revision

Docker Image

↓

Multiple Read-Only Layers

↓

Each Dockerfile instruction creates a new layer

↓

Layers are reused

↓

Reuse saves storage

↓

Reuse makes builds faster

↓

Docker Build Cache uses existing layers

---

# Interview Questions

### 1. What is a Docker Layer?

A Docker Layer is a read-only part of a Docker Image.

---

### 2. Why does Docker use Layers?

To save storage, speed up builds, and reuse existing data.

---

### 3. Which command shows Image Layers?

```bash
docker image history <image-name>
```

---

### 4. Why do some layers show 0B?
Because instructions like `CMD`, `ENV`, `LABEL`, `ENTRYPOINT`, and `EXPOSE` only store metadata or configuration and do not add files.

### 5. What is Docker Build Cache?
Docker Build Cache allows Docker to reuse unchanged layers, making image builds much faster.


### 6. Why is Alpine commonly used in production?
Because it is lightweight, secure, and has a much smaller image size than full Linux distributions.
# Task 3 - Docker Container Lifecycle

## Objective

In this task, you will learn:

- What is a Docker Container Lifecycle?
- Difference between `docker create`, `docker run`, and `docker start`
- Container States
- Pause and Unpause Containers
- Stop, Restart, Kill, and Remove Containers
- Best Practices
- Common Mistakes

---

# What is a Container Lifecycle?

## Definition

A **Container Lifecycle** is the complete journey of a container from creation to deletion.

Every container passes through different states during its life.

# Container Lifecycle Flow

Docker Image
│
▼
Create
│
▼
Created
│
▼
Start
│
▼
Running
│
├──────────────┐
▼              │
Pause          │
│              │
▼              │
Unpause ───────┘
│
▼
Running
│
▼
Stop
│
▼
Restart
│
▼
Kill
│
▼
Remove

## Understanding Each State
| State   | Meaning                                    |
| ------- | ------------------------------------------ |
| Created | Container exists but is not running        |
| Running | Container is actively running              |
| Paused  | Container processes are temporarily frozen |
| Exited  | Container has stopped                      |
| Removed | Container has been deleted                 |

Create a Container
Command
docker create --name myubuntu ubuntu sleep infinity
Command Breakdown
Part	Meaning
docker	Docker CLI
create	Create a container only
--name	Assign container name
myubuntu	Container Name
ubuntu	Image Name
sleep infinity	Keep the container alive
Why Use sleep infinity?

Ubuntu containers normally exit immediately because they do not have a long-running process.

Example

docker run ubuntu

The container starts.

The main process finishes.

The container exits immediately.

To keep it alive we use

sleep infinity

which keeps the main process running forever.

Check Container State

Command

docker ps -a

Example

STATUS

Created

Meaning

The container exists but has not started.

Start a Container

Command

docker start myubuntu
Purpose

Starts an existing container.

After starting, check

docker ps

Example

STATUS

Up 15 seconds

Meaning

The container is running successfully.

docker ps vs docker ps -a
Command	Shows
docker ps	Running Containers Only
docker ps -a	All Containers

This is a common interview question.

Pause a Container

Command

docker pause myubuntu
Definition

Pause temporarily freezes all processes inside a running container.

The container is not stopped.

It is only paused.

Example

Movie Running

↓

Pause

↓

Movie Stops Temporarily

↓

Resume

↓

Movie Continues

Docker behaves similarly.

Verify Pause

Command

docker ps

Example

STATUS

Up 2 minutes (Paused)
Unpause a Container

Command

docker unpause myubuntu

Purpose

Resume a paused container.

Stop a Container

Command

docker stop myubuntu
Definition

Gracefully stops a running container.

Docker first sends a stop signal.

The application gets time to save work and shut down safely.

Example Status

Exited (0)

Exit Code

0 = Success
Restart a Container

Command

docker restart myubuntu

Equivalent to

docker stop

↓

docker start

Used when you want to restart an application.

Kill a Container

Command

docker kill myubuntu
Definition

Immediately stops a running container.

Docker does not wait for graceful shutdown.

Stop vs Kill
docker stop	docker kill
Graceful Shutdown	Immediate Shutdown
Sends Stop Signal	Sends Kill Signal
Production Recommended	Emergency Use
Remove a Container

Command

docker rm myubuntu

Purpose

Deletes the container.

Important Rule

Removing a container does NOT remove the image.

Ubuntu Image
      │
      ▼
Ubuntu Container

Delete Container

↓

Image Still Exists

Image and Container are independent.

docker create vs docker run vs docker start

This is one of the most frequently asked Docker interview questions.

Command	Description
docker create	Create only
docker start	Start existing container
docker run	Create + Start

Remember

docker run

=

docker create

+

docker start
Complete Lifecycle Commands
docker create --name myubuntu ubuntu sleep infinity

docker ps -a

docker start myubuntu

docker ps

docker pause myubuntu

docker unpause myubuntu

docker stop myubuntu

docker restart myubuntu

docker kill myubuntu

docker rm myubuntu
Best Practices

✅ Give meaningful container names.

✅ Use docker stop before using docker kill.

✅ Remove unused containers regularly.

✅ Verify container state using docker ps.

Common Mistakes

❌ Thinking docker create starts a container.

❌ Using docker kill instead of docker stop.

❌ Forgetting docker ps -a.

❌ Confusing Images and Containers.

Quick Revision
docker create

↓

Created

↓

docker start

↓

Running

↓

docker pause

↓

Paused

↓

docker unpause

↓

Running

↓

docker stop

↓

Exited

↓

docker restart

↓

Running

↓

docker kill

↓

Exited

↓

docker rm

↓

Removed
Interview Questions
1. What is Container Lifecycle?

Container Lifecycle is the complete journey of a container from creation to deletion.

2. Difference between docker create and docker run?

docker create

Creates a container but does not start it.

docker run

Creates and starts a new container.

3. Difference between docker stop and docker kill?

docker stop

Gracefully stops a container.

docker kill

Immediately terminates a container.

4. Which command shows all containers?
docker ps -a
5. Can a container exist without running?

Yes.

A container can remain in the Created state until it is started.

6. Does removing a container remove the image?

No.

Images and Containers are separate objects.

Scenario-Based Questions
Scenario 1

A developer created a container but it is not running.

Which command should you use?

Answer

docker start <container-name>
Scenario 2

Your application is not responding, but you want a graceful shutdown.

Which command should you use?

Answer

docker stop <container-name>
Scenario 3

A container is completely stuck and refuses to stop.

Which command should you use?

Answer

docker kill <container-name>
Scenario 4

You accidentally deleted a container.

Can you still create another container?

Answer

Yes.

If the image still exists, Docker can create a new container from the same image.

# Task 4 - Working with Running Containers

## Objective

In this task, you will learn:

- Run a container in Detached Mode
- Port Mapping
- View Container Logs
- Follow Logs in Real Time
- Execute Commands inside a Running Container
- Inspect a Running Container
- Docker Cleanup Commands
- Docker Disk Usage

---

# Running a Container in Detached Mode

## Definition

Detached Mode allows a container to run **in the background** without occupying your terminal.

This is the most common way containers are run in production.

---

# Command

```bash
docker run -d --name mynginx -p 8080:80 nginx
```

---

# Command Breakdown

| Option | Description |
|----------|-------------|
| docker | Docker CLI |
| run | Create and Start Container |
| -d | Detached Mode (Background) |
| --name | Assign Container Name |
| mynginx | Container Name |
| -p | Publish Port |
| 8080 | Host Port |
| 80 | Container Port |
| nginx | Docker Image |

---

# Understanding Detached Mode

Without Detached Mode

```
Terminal
    │
Application Output
```

The terminal remains busy.

---

With Detached Mode

```
Terminal
      │
      ├──────── Free
      │
Docker Container
      │
Application Running
```

The application keeps running in the background.

---

# Port Mapping

One of the most important Docker concepts.

Command

```bash
docker run -p 8080:80 nginx
```

Meaning

```
Browser
      │
localhost:8080
      │
Host Port 8080
      │
Docker Network
      │
Container Port 80
      │
Nginx
```

Always remember

```
HostPort : ContainerPort
```

Example

```
8080 : 80
```

Host uses Port **8080**

Container uses Port **80**

---

# Verify Running Container

Command

```bash
docker ps
```

Example

```
CONTAINER ID

STATUS

PORTS

0185e46bef09

Up 30 seconds

0.0.0.0:8080->80/tcp
```

---

# Browser Test

Open

```
http://localhost:8080
```

Expected Output

```
Welcome to nginx!
```

If this page opens successfully,

your container is working correctly.

---

# docker logs

## Definition

Logs contain information generated by an application while it is running.

Logs help us troubleshoot problems.

---

# View Logs

Command

```bash
docker logs mynginx
```

Purpose

Displays all logs generated by the container.

---

# Example

```
Configuration complete

nginx started

worker process started
```

---

# docker logs -f

## Definition

Shows existing logs and continuously displays new logs in real time.

---

# Command

```bash
docker logs -f mynginx
```

The option

```
-f
```

means

```
Follow
```

This command continues running until you press

```
Ctrl + C
```

Pressing Ctrl+C stops the logs command only.

The container continues running.

---

# Example

```
GET /

200

GET /favicon.ico

404
```

---

# Understanding Log Messages

## HTTP 200

Request completed successfully.

---

## HTTP 404

Requested resource was not found.

Example

```
favicon.ico
```

This is completely normal.

Most browsers automatically request

```
favicon.ico
```

---

## HTTP 304

The browser already has the file in cache.

No need to download it again.

---

# docker exec

## Definition

docker exec is used to execute commands inside a running container.

---

# Login to Container

Command

```bash
docker exec -it mynginx bash
```

If bash is unavailable

```bash
docker exec -it mynginx sh
```

---

# Command Breakdown

| Option | Description |
|----------|-------------|
| docker | Docker CLI |
| exec | Execute Command |
| -i | Interactive Mode |
| -t | Allocate Terminal |
| mynginx | Container Name |
| bash | Shell |

---

# Useful Commands Inside Container

Current Directory

```bash
pwd
```

List Files

```bash
ls
```

Move to Nginx Directory

```bash
cd /usr/share/nginx/html
```

View Default Page

```bash
cat index.html
```

Output

```
Welcome to nginx!
```

This is the same page displayed in the browser.

---

# Run a Single Command

Sometimes you only need to execute one command.

No need to login.

Example

```bash
docker exec mynginx pwd
```

Example

```bash
docker exec mynginx whoami
```

Example

```bash
docker exec mynginx ls /usr/share/nginx/html
```

Docker executes the command and immediately returns the output.

---

# docker inspect (Container)

Command

```bash
docker inspect mynginx
```

Unlike Image Inspect,

this command provides detailed information about a running container.

---

# Important Fields

## IP Address

Example

```
IPAddress

172.17.0.2
```

This is the internal Docker network IP.

---

## Ports

Example

```
80/tcp

↓

HostPort

8080
```

Confirms the published port mapping.

---

## Mounts

Example

```
"Mounts": []
```

No Docker Volumes are attached.

Volumes will be covered in Day 32.

---

# Task 5 - Cleanup Commands

Containers and images consume disk space.

Regular cleanup is important.

---

# Stop Container

```bash
docker stop mynginx
```

Stops a running container.

---

# Remove Container

```bash
docker rm mynginx
```

Deletes the stopped container.

---

# Remove Dangling Images

```bash
docker image prune
```

Purpose

Removes unused dangling images.

---

# Force Remove Dangling Images

```bash
docker image prune -f
```

Skips confirmation.

---

# Check Docker Disk Usage

Command

```bash
docker system df
```

Example

```
TYPE

Images

Containers

Volumes

Build Cache
```

---

# Understanding docker system df

## Images

Shows image storage usage.

---

## Containers

Shows container storage usage.

---

## Local Volumes

Shows persistent volume storage.

---

## Build Cache

Shows Docker Build Cache usage.

Docker reuses this cache to speed up future image builds.

---

# docker system prune

Command

```bash
docker system prune
```

Removes

- Stopped Containers
- Unused Networks
- Dangling Images
- Build Cache

---

# Remove Everything

```bash
docker system prune -a --volumes
```

⚠ Warning

This removes:

- Containers
- Images
- Networks
- Volumes
- Build Cache

Use this command carefully.

---

# Important Note (KIND Users)

If you are using KIND Kubernetes,

DO NOT execute

```bash
docker stop $(docker ps -q)
```

because it will stop the Kubernetes nodes.

Example

```
kind-control-plane

kind-worker

kind-worker2
```

Stopping these containers will stop the Kubernetes cluster.

---

# Production Best Practices

✅ Use meaningful container names.

✅ Use Detached Mode in production.

✅ Always check logs when troubleshooting.

✅ Use docker exec only when necessary.

✅ Remove unused containers regularly.

✅ Monitor Docker disk usage.

✅ Avoid using docker system prune -a on production servers.

---

# Common Mistakes

❌ Forgetting Port Mapping

❌ Confusing Host Port with Container Port

❌ Closing docker logs -f using Docker Desktop instead of Ctrl+C

❌ Logging into containers unnecessarily for a single command

❌ Running cleanup commands without checking active containers

❌ Accidentally stopping KIND Kubernetes containers

---

# Commands Learned

```bash
docker run -d --name mynginx -p 8080:80 nginx

docker ps

docker logs mynginx

docker logs -f mynginx

docker exec -it mynginx bash

docker exec -it mynginx sh

docker exec mynginx pwd

docker exec mynginx whoami

docker exec mynginx ls /usr/share/nginx/html

docker inspect mynginx

docker stop mynginx

docker rm mynginx

docker image prune

docker image prune -f

docker system df

docker system prune

docker system prune -a --volumes
```

---

# Quick Revision

```
docker run -d
        ↓
Background Container

docker logs
        ↓
View Logs

docker logs -f
        ↓
Live Logs

docker exec
        ↓
Run Commands

docker inspect
        ↓
Container Details

docker stop
        ↓
Graceful Stop

docker rm
        ↓
Remove Container

docker system df
        ↓
Disk Usage

docker image prune
        ↓
Remove Dangling Images
```

---

# Interview Questions

## 1. What is Detached Mode?

Detached Mode runs a container in the background while keeping the terminal free.

---

## 2. What does -p 8080:80 mean?

It maps Host Port 8080 to Container Port 80.

---

## 3. Difference between docker logs and docker logs -f?

docker logs shows existing logs.

docker logs -f continuously follows new logs.

---

## 4. What is docker exec used for?

It executes commands inside a running container.

---

## 5. Which command shows Docker disk usage?

```bash
docker system df
```

---

## 6. What information does docker inspect provide?

- Container IP Address
- Port Mapping
- Environment Variables
- Mounts
- Network Configuration
- Container State

---

## 7. What does docker image prune remove?

Unused dangling images.

---

## 8. Why should docker system prune -a be used carefully?

Because it removes unused images, containers, networks, build cache, and can accidentally remove resources you still need.

# Day 30 - Final Revision Notes

---

# One Page Quick Revision

## Docker

Docker is a platform used to build, ship, and run applications inside containers.

---

## Docker Image

A Docker Image is a **Read-Only Template** used to create Containers.

---

## Docker Container

A Docker Container is a **Running Instance of a Docker Image.**

---

## Docker Hub

Docker Hub is an online repository used to download and share Docker Images.

---

## Docker Image Layer

A Docker Image consists of multiple **Read-Only Layers**.

Each Dockerfile instruction creates a new layer.

---

## Docker Container Lifecycle

```
Image
   │
Create
   │
Created
   │
Start
   │
Running
   │
Pause
   │
Running
   │
Stop
   │
Exited
   │
Restart
   │
Running
   │
Kill
   │
Exited
   │
Remove
```

---

# Commands Cheat Sheet

## Images

```bash
docker pull nginx

docker pull ubuntu

docker pull alpine

docker images

docker image ls

docker inspect nginx

docker image history nginx

docker image rm alpine
```

---

## Container Lifecycle

```bash
docker create --name myubuntu ubuntu sleep infinity

docker start myubuntu

docker pause myubuntu

docker unpause myubuntu

docker stop myubuntu

docker restart myubuntu

docker kill myubuntu

docker rm myubuntu
```

---

## Running Containers

```bash
docker run -d --name mynginx -p 8080:80 nginx

docker ps

docker ps -a

docker logs mynginx

docker logs -f mynginx

docker exec -it mynginx bash

docker exec mynginx pwd

docker inspect mynginx
```

---

## Cleanup

```bash
docker image prune

docker system df

docker system prune

docker system prune -a --volumes
```

---

# Production Best Practices

✅ Use official Docker images.

✅ Use meaningful container names.

✅ Prefer Alpine images when suitable.

✅ Keep images small.

✅ Always inspect logs before debugging.

✅ Remove unused containers.

✅ Monitor Docker disk usage.

✅ Avoid using root user in production containers.

✅ Use docker stop instead of docker kill.

✅ Use docker system prune carefully.

---

# Common Mistakes

❌ Image and Container are considered the same.

❌ Forgetting HostPort and ContainerPort order.

❌ Assuming EXPOSE publishes ports.

❌ Deleting Kubernetes KIND containers.

❌ Ignoring docker logs during troubleshooting.

❌ Using docker kill unnecessarily.

❌ Running docker system prune -a on production servers.

❌ Forgetting to remove unused containers.

---

# Docker Command Flow

```
docker pull

↓

docker images

↓

docker create

↓

docker start

↓

docker ps

↓

docker logs

↓

docker exec

↓

docker inspect

↓

docker stop

↓

docker rm
```

---

# Frequently Asked Interview Questions

## 1. What is Docker?

Docker is a platform used to build, ship, and run applications inside containers.

---

## 2. What is a Docker Image?

A Docker Image is a Read-Only Template used to create Containers.

---

## 3. What is a Docker Container?

A Docker Container is a Running Instance of a Docker Image.

---

## 4. Difference between Image and Container?

Image is a template.

Container is a running instance created from that template.

---

## 5. What is Docker Hub?

Docker Hub is an online repository used to store and download Docker Images.

---

## 6. Which command downloads an image?

```bash
docker pull
```

---

## 7. Which command lists images?

```bash
docker images
```

---

## 8. Which command shows image history?

```bash
docker image history
```

---

## 9. Why does Docker use Layers?

To reuse data, reduce storage, and speed up builds.

---

## 10. Why do some layers show 0B?

Because commands like CMD, ENV, LABEL, and EXPOSE only store metadata.

---

## 11. Difference between docker create and docker run?

docker create creates a container only.

docker run creates and starts the container.

---

## 12. Difference between docker start and docker run?

docker start starts an existing container.

docker run creates and starts a new container.

---

## 13. Difference between docker stop and docker kill?

docker stop performs a graceful shutdown.

docker kill immediately terminates the container.

---

## 14. What is Detached Mode?

Detached Mode runs the container in the background.

---

## 15. What does -p 8080:80 mean?

Host Port 8080 is mapped to Container Port 80.

---

## 16. Difference between docker logs and docker logs -f?

docker logs shows existing logs.

docker logs -f continuously follows new logs.

---

## 17. What is docker exec used for?

To execute commands inside a running container.

---

## 18. What information does docker inspect provide?

- IP Address
- Ports
- Mounts
- Environment Variables
- Network Settings
- Container State

---

## 19. Which command checks Docker disk usage?

```bash
docker system df
```

---

## 20. What is docker image prune?

Removes unused dangling images.

---

## 21. What is Build Cache?

Docker Build Cache reuses unchanged layers to speed up image builds.

---

## 22. Can one image create multiple containers?

Yes.

One image can create any number of containers.

---

## 23. Can a container exist without running?

Yes.

A container can remain in the Created state.

---

## 24. Does removing a container remove its image?

No.

Image and Container are separate Docker objects.

---

## 25. Why is Alpine preferred in production?

Because it is lightweight and has a much smaller image size.

---

# Scenario-Based Questions

## Scenario 1

Your application is running but not responding.

What should you check first?

**Answer**

```bash
docker logs <container-name>
```

---

## Scenario 2

A developer created a container but it never started.

Which command should you use?

```bash
docker start <container-name>
```

---

## Scenario 3

You need to execute a single command inside a running container.

Which command should you use?

```bash
docker exec <container-name> <command>
```

---

## Scenario 4

You need interactive shell access.

Which command should you use?

```bash
docker exec -it <container-name> bash
```

or

```bash
docker exec -it <container-name> sh
```

---

## Scenario 5

A running application needs a graceful shutdown.

Which command should you use?

```bash
docker stop
```

---

## Scenario 6

The container is completely frozen.

Which command should you use?

```bash
docker kill
```

---

## Scenario 7

You accidentally removed a container.

Can you recreate it?

Yes.

If the image still exists.

---

## Scenario 8

A browser cannot access your container.

Which things should you check?

- Is the container running?
- Is port mapping correct?
- Is the application listening on the correct port?
- Check docker logs.
- Check firewall.

---

## Scenario 9

Your Docker host is running out of storage.

Which command helps you analyze disk usage?

```bash
docker system df
```

---

## Scenario 10

You want to clean unused images only.

Which command should you use?

```bash
docker image prune
```

---

# Mind Map

```
Docker

│

├── Images

│      ├── Pull

│      ├── Inspect

│      ├── History

│      └── Remove

│

├── Layers

│      ├── RUN

│      ├── COPY

│      ├── ENV

│      ├── CMD

│      └── ENTRYPOINT

│

├── Containers

│      ├── Create

│      ├── Start

│      ├── Pause

│      ├── Stop

│      ├── Restart

│      ├── Kill

│      └── Remove

│

├── Running Containers

│      ├── Logs

│      ├── Exec

│      ├── Inspect

│      └── Port Mapping

│

└── Cleanup

       ├── image prune

       ├── system df

       └── system prune



