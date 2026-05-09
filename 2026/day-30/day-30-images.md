# Day 30 – Docker Images & Container Lifecycle

## Goal
Today's goal was to understand how Docker images and containers actually work.
## Topics Covered

- Relationship between Docker images and containers
- Docker image layers and caching
- Difference between Ubuntu and Alpine images
- Docker image inspection
- Docker container lifecycle
- Working with running containers
- Docker logs, exec, inspect, port mapping
- Docker cleanup and disk usage
  
# 1. Docker Images

## What is a Docker Image?

A Docker image is a read-only template used to create containers.

It contains:

- Application code
- Runtime
- Libraries
- System dependencies
- Environment variables
- Default command
- Filesystem

Simple understanding:
Docker Image = Template / Blueprint
Docker Container = Running or stopped instance of an image

Example:
docker run nginx
Here:
nginx = Docker image
running web server = Docker container

## Image vs Container

| Point | Docker Image | Docker Container |
|---|---|---|
| Meaning | Read-only template | Running/stopped instance |
| State | Static | Dynamic |
| Writable? | No | Yes, container has writable layer |
| Used for | Creating containers | Running applications |
| Example | `nginx:latest` | `my-nginx` |
| List command | `docker images` | `docker ps`, `docker ps -a` |
| Remove command | `docker rmi image-name` | `docker rm container-name` |

Important:

> One Docker image can be used to create many containers.

Example:

docker run --name nginx1 nginx
docker run --name nginx2 nginx

Both containers are created from the same `nginx` image.

# Task 1: Docker Images

## Pull Images from Docker Hub

Commands:

docker pull nginx
docker pull ubuntu
docker pull alpine

These commands download images from Docker Hub to the local machine.

## List Images

Command:

docker images
Observed images:
IMAGE           DISK USAGE   CONTENT SIZE
alpine:latest   13.1MB       3.95MB
nginx:latest    240MB        65.8MB
ubuntu:latest   160MB        45.3MB

## Ubuntu vs Alpine

### Ubuntu

Ubuntu is a full Linux distribution.

It includes more packages, libraries, and compatibility tools.

Advantages:

- Easier for learning
- Easier for debugging
- Better compatibility with many packages
- Uses `glibc`

Disadvantages:

- Larger image size
- More packages than needed for small production apps

### Alpine

Alpine is a very small Linux distribution designed for containers.

Advantages:

- Very small image size
- Faster pull
- Less disk usage
- Smaller attack surface

Disadvantages:

- Minimal tools available by default
- Some packages may have compatibility issues
- Uses `musl libc` instead of `glibc`

## Why is Alpine Smaller than Ubuntu?

Alpine is smaller because it includes only minimal Linux utilities and libraries.

Ubuntu is bigger because it provides a more complete Linux environment.

Interview answer:

> Alpine is smaller because it is a minimal Linux distribution designed for containers. Ubuntu is larger because it contains more system packages, libraries, and debugging tools. Alpine is good for small production images, while Ubuntu is easier for compatibility and troubleshooting.

## Inspect an Image

Command:

docker image inspect nginx

Filtered commands used:
docker image inspect nginx --format='{{.Id}}'
docker image inspect nginx --format='{{.Architecture}}'
docker image inspect nginx --format='{{.Os}}'
docker image inspect nginx --format='{{.Config.ExposedPorts}}'
docker image inspect nginx --format='{{.Config.Entrypoint}}'
docker image inspect nginx --format='{{.Config.Cmd}}'

Observed output:
Image ID: sha256:a7a9f7e9f549699206cd61c9d14c5434c7ed85e510e837211e0c2d24c9bdb9ac
Architecture: amd64
OS: linux
Exposed Port: 80/tcp
Entrypoint: /docker-entrypoint.sh
CMD: nginx -g daemon off;

## What Information Can We See from Image Inspect?
docker image inspect` shows:

- Image ID
- OS
- Architecture
- Created date
- Environment variables
- Entrypoint
- CMD
- Exposed ports
- Labels
- Layers
- Repo tags


## Entrypoint and CMD

For Nginx:
Entrypoint: /docker-entrypoint.sh
CMD: nginx -g daemon off;
Meaning:

Docker starts the entrypoint script first.  
Then it runs the default command.

Together, Nginx starts like this:

/docker-entrypoint.sh nginx -g "daemon off;"

`daemon off;` keeps Nginx running in the foreground.

This is important because:

> A Docker container runs only while its main foreground process is running.

## Remove an Image

First, check containers:

docker ps -a

If a stopped container is using the image, remove the container first:

docker rm unruffled_sammet

Then remove the image:
docker rmi hello-world

Observed result:

Untagged: hello-world:latest
Deleted: sha256:f9078146...

# 2. Docker Image Layers

## What are Docker Image Layers?

Docker images are made of multiple read-only layers.

Each layer represents a filesystem change or metadata instruction.

Example Dockerfile:

dockerfile
FROM ubuntu
RUN apt update
RUN apt install -y nginx
COPY index.html /usr/share/nginx/html/index.html
CMD ["nginx", "-g", "daemon off;"]

Possible layers:
Layer 1: Ubuntu base image
Layer 2: apt update
Layer 3: install nginx
Layer 4: copy index.html
Layer 5: CMD metadata

## Why Docker Uses Layers

Docker uses layers for:

- Faster builds
- Build caching
- Less disk usage
- Reusing common layers
- Faster image pull
- Faster image push

Example:

If multiple images use the same base layer, Docker downloads that base layer only once.

## Image Layer and Container Writable Layer

Docker image layers are read-only.

When a container starts, Docker adds a thin writable layer on top.

Container Writable Layer
------------------------
Image Layer 5
Image Layer 4
Image Layer 3
Image Layer 2
Image Layer 1

If files are changed inside a running container, those changes go into the writable container layer.

If the container is removed, the writable layer is also removed.


# Task 2: Image Layers

## View Nginx Image History

Command:

docker image history nginx

Observed output included:
CMD ["nginx" "-g" "daemon off;"]                0B
STOPSIGNAL SIGQUIT                              0B
EXPOSE map[80/tcp:{}]                           0B
ENTRYPOINT ["/docker-entrypoint.sh"]            0B
COPY docker-entrypoint.sh /                     8.19kB
RUN /bin/sh -c set -x && groupadd ...           86.7MB
ENV NGINX_VERSION=1.29.8                        0B
LABEL maintainer=NGINX Docker Maintainers       0B
Debian base layer                               87.4MB

## Explanation of Layers

### Base Layer
Debian base layer: 87.4MB
This is the base OS layer used by the Nginx image.

### RUN Layer

RUN /bin/sh -c set -x && groupadd ...
SIZE: 86.7MB

This layer adds real filesystem changes such as packages, users, groups, and Nginx setup.

`RUN` commands usually create filesystem layers.

### COPY Layers
COPY docker-entrypoint.sh /
COPY 10-listen-on-ipv6-by-default.sh ...
COPY 20-envsubst-on-templates.sh ...
COPY 30-tune-worker-processes.sh ...

These layers copy files into the image.

They have small sizes because small shell scripts are added.

### ENV Layers
ENV NGINX_VERSION=1.29.8
ENV NJS_VERSION=0.9.6

These layers show `0B` because they store metadata and do not add files.

### EXPOSE Layer

EXPOSE map[80/tcp:{}]
SIZE: 0B
This documents that the container listens on port `80`.

Important:

> `EXPOSE` does not publish the port to the host machine. For host access, we still need `-p`.

Example:
docker run -p 8080:80 nginx

### ENTRYPOINT and CMD Layers
ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["nginx" "-g" "daemon off;"]

These are metadata instructions, so they show `0B`.

## Why Some Layers Show 0B

Some layers show `0B` because they do not add files to the filesystem.

Examples:

- `CMD`
- `ENTRYPOINT`
- `ENV`
- `EXPOSE`
- `LABEL`
- `STOPSIGNAL`
- `WORKDIR`

These mostly change image metadata.

## Why Some Layers Have Size

Layers have size when they add or modify files.

Examples:

- `RUN apt install`
- `COPY file`
- `ADD file`
- Base OS layer
  
## What is `<missing>` in Docker Image History?

`<missing>` in image history is normal.

It means Docker is showing intermediate image layers that do not have separate user-friendly image IDs in the local image list.

It does not mean the image is broken.

# 3. Docker Container Lifecycle

## What is Container Lifecycle?

Container lifecycle means the different states a container goes through.

Main states:
Created
Running
Paused
Exited
Removed

Lifecycle flow:
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

# Task 3: Container Lifecycle Practice

## 1. Create a Container Without Starting It

Command:

docker create --name lifecycle-nginx -p 8082:80 nginx

Check status:
docker ps -a
Expected status:
Created
Explanation:

docker create` creates a container but does not start it.

## 2. Start Container

Command:

docker start lifecycle-nginx
docker ps -a

Expected status:
Up

Explanation:

The container is now running.

## 3. Pause Container

Command:
docker pause lifecycle-nginx
docker ps -a

Expected status:
Up ... (Paused)

Explanation:

Pause freezes the running processes inside the container.

The container is not stopped, but its processes are suspended.

## 4. Unpause Container

Command:
docker unpause lifecycle-nginx
docker ps -a

Expected status:
Up
Explanation:

The container resumes from paused state.

## 5. Stop Container

Command:
docker stop lifecycle-nginx
docker ps -a
Expected status:
Exited
Explanation:
docker stop gracefully stops the container.

It sends a stop signal first and gives the process time to shut down.

## 6. Restart Container

Command:
docker restart lifecycle-nginx
docker ps -a
Expected status:
Up
Explanation:
docker restart stops and starts the container again.

## 7. Kill Container

Command:
docker kill lifecycle-nginx
docker ps -a
Observed status:
Exited (137)
Explanation:
docker kill forcefully stops the container.
Exit code `137` commonly means the container was killed using a kill signal.

## 8. Remove Container
Command:
docker rm lifecycle-nginx
docker ps -a
Expected result:
lifecycle-nginx should not appear
Explanation:
`docker rm` removes a stopped container.

## Stop vs Kill

| Command | Meaning |
| `docker stop` | Graceful shutdown |
| `docker kill` | Forceful shutdown |

Interview answer:

> `docker stop` sends a graceful termination signal first, allowing the application to shut down properly. `docker kill` immediately sends a kill signal and forcefully stops the container.

# 4. Working with Running Containers

# Task 4: Running Container Practice

## Run Nginx in Detached Mode

Command:
docker run -d --name day30-nginx -p 8081:80 nginx

Explanation:

| Part | Meaning |
|---|---|
| `docker run` | Create and start container |
| `-d` | Detached mode, run in background |
| `--name day30-nginx` | Custom container name |
| `-p 8081:80` | Host port 8081 to container port 80 |
| `nginx` | Image name |

Browser URL:
http://localhost:8081

## Check Running Containers

Command:
docker ps
Expected:
day30-nginx   Up   0.0.0.0:8081->80/tcp

## View Container Logs

Command:
docker logs day30-nginx

If browser was opened, logs show:
GET / HTTP/1.1" 200
GET /favicon.ico HTTP/1.1" 404

Explanation:
- `200` means success
- `404` means file not found
- `favicon.ico` 404 is normal because the browser asks for a website icon

## View Real-Time Logs

Command:
docker logs -f day30-nginx

Then refresh:
http://localhost:8081

New log lines appear live.

To exit follow mode:
Ctrl + C
Important:

> `Ctrl + C` exits log follow mode. It does not stop the container.

## Exec Into Running Container

Command:
docker exec -it day30-nginx sh

Inside container:
pwd
ls
cat /etc/os-release
ls /usr/share/nginx/html
exit
Explanation:

`docker exec` runs a command inside an already running container.

`-it` gives an interactive terminal.

## Run Single Command Without Entering Container

Commands:
docker exec day30-nginx hostname
docker exec day30-nginx cat /etc/os-release
docker exec day30-nginx ls /usr/share/nginx/html

This is useful for automation and debugging.

Example use case:
CI/CD pipeline can run a command inside a container without opening interactive shell.

## Inspect Container

Command:
docker inspect day30-nginx

This shows detailed JSON information about the container.

It includes:

- Container ID
- Image ID
- Created time
- State
- IP address
- Port mappings
- Mounts
- Networks
- Environment variables
- Entrypoint
- Command
## Find Container IP Address

Command:
docker inspect -f "{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}" day30-nginx

Explanation:

This command filters the inspect output and shows only the container IP address.

## Find Port Mapping

Command:
docker port day30-nginx
Example output:
80/tcp -> 0.0.0.0:8081
80/tcp -> [::]:8081

Explanation:

Container port `80` is mapped to host port `8081`.

## Find Mounts

Command:
docker inspect -f "{{json .Mounts}}" day30-nginx

If no volumes are mounted, output may be:
Explanation:

Mounts show volumes or bind mounts attached to the container.

## Find Container State
Command:
docker inspect -f "{{.State.Status}}" day30-nginx
Example:
running

# 5. Docker Cleanup

## Why Cleanup is Important

Docker images, containers, volumes, and build cache can consume disk space.

In real DevOps work, cleanup is required on:

- Developer machines
- CI/CD runners
- Build servers
- Test environments
- Shared servers

# Task 5: Cleanup

## Check Running Containers

Command:
docker ps

## Stop All Running Containers
Git Bash / Linux command:
docker stop $(docker ps -q)
Explanation:
docker ps -q

returns only running container IDs.

Then:
docker stop $(docker ps -q)
stops all running containers.
If no containers are running, Docker may show an error. That is okay.

## Remove All Stopped Containers
Command:
docker container prune

It asks:
Are you sure you want to continue? [y/N]

Type:
y
Force version:
docker container prune -f
## Remove Unused Images
Command:
docker image prune
This removes dangling images.
Force version:
docker image prune -f
To remove all unused images not used by any container:
docker image prune -a
Be careful with `-a`.

## Check Docker Disk Usage

Command:
docker system df
It shows disk usage for:
- Images
- Containers
- Local volumes
- Build cache

Example:
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          3         1         400MB     100MB
Containers      0         0         0B        0B
Local Volumes   0         0         0B        0B
Build Cache     0         0         0B        0B

## Full Cleanup Commands
docker ps
docker stop $(docker ps -q)
docker ps
docker ps -a
docker container prune
docker image prune
docker system df

# Important Commands Summary

## Image Commands
docker pull nginx
docker pull ubuntu
docker pull alpine
docker images
docker image inspect nginx
docker image history nginx
docker rmi image-name

## Container Lifecycle Commands
docker create --name lifecycle-nginx -p 8082:80 nginx
docker start lifecycle-nginx
docker pause lifecycle-nginx
docker unpause lifecycle-nginx
docker stop lifecycle-nginx
docker restart lifecycle-nginx
docker kill lifecycle-nginx
docker rm lifecycle-nginx
docker ps
docker ps -a

## Running Container Commands
docker run -d --name day30-nginx -p 8081:80 nginx
docker logs day30-nginx
docker logs -f day30-nginx
docker exec -it day30-nginx sh
docker exec day30-nginx hostname
docker inspect day30-nginx
docker port day30-nginx

## Cleanup Commands
docker stop $(docker ps -q)
docker container prune
docker image prune
docker image prune -a
docker system df
docker system prune
# Practice Questions

## Basic Questions

1. What is a Docker image?
2. What is a Docker container?
3. What is the difference between an image and a container?
4. What command lists Docker images?
5. What command lists running containers?
6. What command lists all containers including stopped ones?
7. What command removes a stopped container?
8. What command removes an image?

## Scenario-Based Questions

1. You pulled `ubuntu` and `alpine`. Alpine is much smaller. Why?
2. You created a container using `docker create`, but it is not running. Why?
3. Your container shows `Exited (137)`. What could be the reason?
4. You want to see live logs of a container. Which command will you use?
5. You want to run `hostname` inside a running container without entering the shell. Which command will you use?
6. You want to find the IP address of a container. Which command can you use?
7. You want to remove all stopped containers. Which command will you use?
8. Docker is consuming too much disk space. Which command helps check usage?


# Interview Questions and Answers

## Q1. What is the relationship between Docker image and container?

### Answer

A Docker image is a read-only template that contains application code, dependencies, runtime, and configuration. A container is a running or stopped instance created from that image.

One image can be used to create multiple containers.

Example:

docker run --name nginx1 nginx
docker run --name nginx2 nginx

Both containers are created from the same `nginx` image.

## Q2. What are Docker image layers?

### Answer

Docker image layers are read-only filesystem changes created during the image build process. Each instruction in a Dockerfile can create a layer.

Docker uses layers for caching, reuse, faster builds, faster image pulls, and reduced disk usage.

Example:

dockerfile
FROM ubuntu
RUN apt update
COPY index.html /app/index.html
CMD ["nginx", "-g", "daemon off;"]

Each step can create a layer.

## Q3. Why do some Docker image history layers show 0B?

### Answer

Some Dockerfile instructions only change metadata and do not add files to the filesystem.

Examples:

- `CMD`
- `ENTRYPOINT`
- `ENV`
- `EXPOSE`
- `LABEL`
- `STOPSIGNAL`

Because they do not add filesystem content, their layer size can show as `0B`.

## Q4. Why is Alpine image smaller than Ubuntu?

### Answer

Alpine is smaller because it is a minimal Linux distribution designed for containers. It includes fewer packages and libraries.

Ubuntu is larger because it provides a more complete Linux environment with more tools and libraries.

Alpine is good for small production images, but Ubuntu is often easier for compatibility and debugging.

## Q5. What is the difference between `docker create` and `docker run`?

### Answer

`docker create` creates a container but does not start it.

`docker run` creates and starts a container in one step.

These two commands are equivalent:
docker run nginx
and:
docker create nginx
docker start container-id

## Q6. What is the difference between `docker stop` and `docker kill`?

### Answer

`docker stop` gracefully stops a container by sending a termination signal and allowing the application time to shut down.

`docker kill` forcefully stops the container immediately by sending a kill signal.

In production, `docker stop` is safer because it allows graceful shutdown.


## Q7. What does `Exited (137)` mean?

### Answer

`Exited (137)` usually means the container was forcefully killed.

It can happen when:

- `docker kill` was used
- The process was killed by the system
- The container ran out of memory in some cases

In our task, it happened after running:
docker kill lifecycle-nginx

## Q8. What is the purpose of `docker inspect`?

### Answer

`docker inspect` shows detailed JSON information about Docker objects like containers and images.

For containers, it shows:

- State
- IP address
- Network details
- Port mappings
- Mounts
- Environment variables
- Entrypoint
- Command
- Created time

Example:
docker inspect day30-nginx

Filtered IP command:
docker inspect -f "{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}" day30-nginx

## Q9. What is port mapping in Docker?

### Answer

Port mapping connects a port on the host machine to a port inside the container.

Example:
docker run -d --name web -p 8081:80 nginx

This maps:
Host port 8081 → Container port 80

So the Nginx container can be accessed using:
http://localhost:8081

## Q10. What is the difference between `EXPOSE` and `-p`?

### Answer

`EXPOSE` documents which port the container application listens on, but it does not publish the port to the host.

`-p` actually maps the container port to the host port.

Example:

dockerfile
EXPOSE 80

This only documents port 80.

docker run -p 8080:80 nginx

This actually maps host port 8080 to container port 80.

## Q11. How do you check real-time logs of a container?

### Answer

Use:
docker logs -f container-name

Example:
docker logs -f day30-nginx

`-f` means follow mode. It shows logs live as they are generated.

## Q12. How do you execute a command inside a running container?

### Answer

Use `docker exec`.

Interactive shell:

docker exec -it day30-nginx sh

Single command:

docker exec day30-nginx hostname

## Q13. How do you stop all running containers?

### Answer

In Git Bash or Linux:
docker stop $(docker ps -q)

`docker ps -q` returns only running container IDs.  
Then `docker stop` stops those containers.

## Q14. How do you remove all stopped containers?

### Answer
Use:
docker container prune
Force mode:
docker container prune -f

## Q15. How do you check Docker disk usage?

### Answer
Use:
docker system df
It shows disk usage for:
- Images
- Containers
- Local volumes
- Build cache

# Real-World DevOps Use Cases

## 1. CI/CD Pipeline
Docker images are built during CI/CD pipelines.
Example flow:
Developer pushes code
        ↓
Pipeline starts
        ↓
Docker image is built
        ↓
Tests run inside container
        ↓
Image pushed to registry
        ↓
Deployment pulls image
        ↓
Container runs in server/Kubernetes

## 2. Debugging Production Issues
Useful commands:
docker logs container-name
docker logs -f container-name
docker exec -it container-name sh
docker inspect container-name
docker port container-name

These commands help troubleshoot:

- Application errors
- Port issues
- Network issues
- Missing files
- Wrong environment variables
- Container crashes

## 3. Build Server Cleanup
On CI/CD runners, Docker can consume disk space.
Common cleanup commands:
docker container prune -f
docker image prune -f
docker system df
For deeper cleanup:
docker system prune -f
Use carefully.

# Final Revision Notes

## Key Points

- Docker image is a read-only template.
- Docker container is a running or stopped instance of an image.
- One image can create multiple containers.
- Docker images are made of layers.
- Docker uses layers for caching and reuse.
- Alpine is much smaller than Ubuntu because it is minimal.
- `docker create` creates a container but does not start it.
- `docker run` creates and starts a container.
- `docker stop` gracefully stops a container.
- `docker kill` forcefully stops a container.
- `docker logs -f` shows real-time logs.
- `docker exec` runs commands inside a running container.
- `docker inspect` shows detailed JSON information.
- `docker system df` shows Docker disk usage.
