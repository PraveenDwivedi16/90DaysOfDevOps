# Day 29 – Introduction to Docker
# Introduction
Docker is one of the most important tools in modern DevOps. It enables developers and operations teams to package applications along with their dependencies into lightweight, portable, and isolated environments called **containers**.

Before Docker, applications often worked correctly on a developer's machine but failed in testing or production due to differences in operating systems, libraries, runtimes, or configurations. Docker solves this problem by providing a consistent runtime environment across development, testing, and production.

Today, Docker is widely used in:
* CI/CD Pipelines
* Kubernetes
* Microservices Architecture
* Cloud Deployments
* DevOps Automation
* Application Packaging

This document is designed as a long-term learning guide, revision notebook, interview preparation resource, and practical reference.

# Learning Objectives

After completing Day 29, you will be able to:

* Explain what Docker is.
* Understand why containers are needed.
* Explain the difference between Containers and Virtual Machines.
* Describe Docker Architecture.
* Explain Docker Client, Docker Daemon, Images, Containers, and Docker Registry.
* Understand the complete Docker workflow.
* Build a strong foundation for future Docker topics.

# Prerequisites
Before starting Docker, you should have:
* Basic Linux command knowledge
* Basic terminal usage
* Ubuntu Linux (Local VM or AWS EC2)
* Internet connection
* Basic understanding of applications and operating systems

# Task 1 – What is Docker?
## Objective
Understand the fundamentals of Docker, why it exists, how containers work, and how Docker Architecture operates.


# What is Docker?

## Definition
Docker is an open-source containerization platform that allows developers to build, package, distribute, and run applications inside lightweight containers.
A Docker container includes everything an application needs to run:

* Application Code
* Runtime
* Libraries
* Dependencies
* Configuration Files

Because everything is packaged together, the application behaves consistently across different environments.

## Why Was Docker Created?
Before Docker, software deployment was difficult because every machine had a different environment.
For example:
Developer Machine

* Python 3.12

Testing Server

* Python 3.10

Production Server

* Python 3.11

Even though the application code was identical, the application could fail because of version differences.

This problem became known as:

**"It Works on My Machine" Problem**

Docker solves this problem by packaging the application together with everything it needs.

As long as Docker is installed, the application will run the same way on:

* Windows
* Linux
* macOS
* AWS
* Azure
* Google Cloud

# Real World Example
Imagine opening multiple restaurant branches.
Without Docker:
Every new branch requires:

* Kitchen setup
* Ingredients
* Staff training
* Equipment installation

Each branch becomes slightly different.

With Docker:
You prepare one complete kitchen kit containing:
* Equipment
* Ingredients
* Recipe
* Instructions

Every new branch receives the same kit.
Every branch works exactly the same.
Docker containers work in the same way.

# What is a Container?
A Container is a lightweight isolated environment where an application runs together with all required dependencies.
A container typically contains:

* Application
* Runtime
* Libraries
* Configuration
* Dependencies

A container **does not include an entire operating system**.
Instead, it shares the Host Operating System Kernel.


## Why Are Containers Lightweight?
Containers share the Host Linux Kernel instead of installing a complete Guest Operating System.
This makes containers:

* Fast
* Lightweight
* Portable
* Easy to Start
* Resource Efficient

# Why Do We Need Containers?
Containers solve several real-world software deployment problems.

## Environment Consistency
Applications behave the same everywhere.

Developer Machine

↓

Testing Server

↓

Production Server

↓

Cloud
Everything remains identical.

## Portability
Containers can run on:
* Laptop
* Virtual Machine
* Physical Server
* AWS
* Azure
* Google Cloud

## Fast Startup
Containers usually start within seconds because they reuse the Host Operating System Kernel.


## Lower Resource Usage
Containers require significantly less RAM and storage compared to Virtual Machines.


## Isolation
Every container runs independently.
If one container crashes, the others continue running.


# Containers vs Virtual Machines
Containers and Virtual Machines both provide isolated environments, but they work differently.

| Containers             | Virtual Machines                         |
| ---------------------- | ---------------------------------------- |
| Lightweight            | Heavy                                    |
| Share Host OS Kernel   | Have Separate Guest Operating System     |
| Start in Seconds       | Slower Startup                           |
| Lower Memory Usage     | Higher Memory Usage                      |
| Best for Microservices | Best for Full Operating System Isolation |
| Faster Deployment      | Slower Deployment                        |

# Virtual Machine Architecture
Application

↓

Libraries

↓

Guest Operating System

↓

Hypervisor

↓

Host Operating System

↓

Physical Hardware

Every Virtual Machine includes its own complete operating system.


# Container Architecture
Application

↓

Libraries

↓

Docker Engine

↓

Host Operating System

↓

Physical Hardware

Containers do not require an additional Guest Operating System.

# Why Are Containers Faster Than Virtual Machines?
Virtual Machines must:

* Boot an Operating System
* Load Services
* Initialize the Kernel
* Start Applications

Containers only start the required application because the Host Operating System is already running.

This significantly reduces startup time.

# Docker Architecture
Docker Architecture consists of five major components.


# 1. Docker Client

## What is Docker Client?
Docker Client is the command-line interface used by users to communicate with Docker.

Whenever a user executes:
docker run nginx
the Docker Client receives the command and sends it to the Docker Daemon.


## Responsibilities

* Accept Docker commands
* Send requests to Docker Daemon
* Display results

## Real World Example

Customer

↓

Places Order

↓

Restaurant
The customer does not cook the food.
Similarly, Docker Client does not create containers.

# 2. Docker Daemon
## What is Docker Daemon?
Docker Daemon (dockerd) is the core background service responsible for managing Docker.
Without Docker Daemon, Docker cannot function.

## Responsibilities

* Pull Images
* Create Containers
* Start Containers
* Stop Containers
* Remove Containers
* Manage Networks
* Manage Volumes

## Real World Example
Customer

↓

Waiter

↓

Chef

Docker Client = Waiter

Docker Daemon = Chef

The chef performs the actual work.

# 3. Docker Image

## What is a Docker Image?
A Docker Image is a **read-only template** used to create Docker Containers.
Examples:

* nginx
* ubuntu
* mysql
* redis

An image cannot execute applications by itself.
It must first become a container.

## Image Characteristics
* Read-only
* Reusable
* Portable
* Version Controlled

## Real World Example
Recipe

↓
Cake
Recipe = Image
Cake = Container
# 4. Docker Container
## What is a Docker Container?
A Docker Container is the running instance of a Docker Image.

Relationship:
Docker Image

↓

docker run

↓

Docker Container

One image can create multiple containers.

# 5. Docker Registry

## What is Docker Registry?
A Docker Registry is a storage location for Docker Images.
The default public registry is:

**Docker Hub**
Whenever Docker cannot find an image locally, it automatically downloads it from Docker Hub.


# Docker Workflow
When the following command is executed:
docker run nginx


Docker performs the following workflow:

1. Docker Client receives the command.
2. Docker Client sends the request to Docker Daemon.
3. Docker Daemon checks whether the image exists locally.
4. If the image is missing, Docker Daemon downloads it from Docker Hub.
5. Docker Daemon creates a new container.
6. Docker Daemon starts the container.
7. The running application becomes available.


# Docker Architecture Diagram

                User
                  │
                  ▼
           Docker Client
                  │
                  ▼
           Docker Daemon
            │          │
            │          ▼
            │    Docker Registry
            │     (Docker Hub)
            │
            ▼
       Docker Image
            │
            ▼
     Docker Container

# Important Points
* Docker is a containerization platform.
* Containers package applications with dependencies.
* Containers share the Host Operating System Kernel.
* Docker Images are templates.
* Docker Containers are running instances of Images.
* Docker Hub stores Docker Images.
* Docker Daemon performs all Docker operations.
* Docker Client only sends commands.

# Quick Revision

* Docker = Container Platform
* Container = Running Application + Dependencies
* Image = Blueprint
* Docker Client = Accepts Commands
* Docker Daemon = Performs Operations
* Docker Hub = Stores Images
* Image → Container using `docker run`


# Interview Questions and Answers

## 1. What is Docker?
Docker is an open-source platform that packages applications and their dependencies into lightweight containers for consistent deployment across different environments.

## 2. What is a Docker Container?
A Docker Container is a lightweight isolated runtime environment created from a Docker Image.

## 3. What is the difference between a Docker Image and a Docker Container?
A Docker Image is a read-only template, while a Docker Container is a running instance of that image.

## 4. Why are Containers lightweight?
Containers share the Host Operating System Kernel instead of installing a separate Guest Operating System.


## 5. Explain Docker Architecture.
Docker Architecture consists of Docker Client, Docker Daemon, Docker Images, Docker Containers, and Docker Registry. The client sends commands to the daemon, which manages images and containers.


## 6. What is Docker Hub?
Docker Hub is the default public registry that stores and distributes Docker Images.

## 7. What happens internally when you run `docker run nginx`?
Docker checks for the image locally, downloads it from Docker Hub if necessary, creates a container from the image, and starts the container.


# Task 2 – Install Docker

## Objective
The objective of this task is to understand the components that make Docker work before installing and using Docker on a system.
Many beginners think Docker is a single software application. In reality, Docker is a collection of multiple components working together to build, run, and manage containers.
Understanding these components is essential because they are frequently discussed in DevOps interviews and are used in real-world production environments.


# Docker Components
Docker is composed of several components. Each component has a specific responsibility.
The major Docker components are:

* Docker Desktop
* Docker Engine
* Docker Daemon
* Docker CLI
* Docker Compose
* Docker Hub
* containerd
* runc


# Docker Desktop
## What is Docker Desktop?
Docker Desktop is an application that makes Docker easy to install and use on Windows and macOS.
It provides a complete Docker development environment by packaging multiple Docker components together.
Linux users normally install Docker Engine directly instead of Docker Desktop.

## Why Do We Need Docker Desktop?
Windows and macOS cannot run Docker Engine in the same way Linux can.
Docker Desktop provides:
* Docker Engine
* Docker CLI
* Docker Compose
* Docker Dashboard (GUI)
* Kubernetes (Optional)
* WSL 2 Integration (Windows)
Everything is installed and configured automatically.


## Features of Docker Desktop
* Easy installation
* Graphical Dashboard
* Docker Engine
* Docker CLI
* Docker Compose
* Extensions
* Kubernetes Support
* Image Management
* Container Management
* Volume Management
* Network Management

## Real World Example
Imagine buying a new laptop.
Instead of installing every software manually:

* Browser
* Office
* Media Player
* Drivers

Someone provides a laptop with everything already installed.
Docker Desktop works similarly.
It provides all Docker tools together.


## Important Note
Docker Desktop is mainly used on:

* Windows
* macOS
Linux servers usually use Docker Engine directly.


# Docker Engine
## What is Docker Engine?
Docker Engine is the core runtime of Docker.
It is responsible for building, running, stopping, and managing containers.
Without Docker Engine, Docker cannot create or execute containers.

## Responsibilities of Docker Engine
Docker Engine performs tasks such as:
* Pull Images
* Build Images
* Create Containers
* Start Containers
* Stop Containers
* Remove Containers
* Manage Volumes
* Manage Networks

## How Does Docker Engine Work?
When a user executes:
docker run nginx

Docker Engine performs the following actions:
1. Receives the request.
2. Checks whether the image exists locally.
3. Downloads the image if required.
4. Creates a container.
5. Starts the container.

## Real World Example
Restaurant Example

Customer

↓

Waiter

↓

Chef

The chef prepares the food.
Similarly,
Docker Engine performs all container operations.


## Interview Tip
Many people confuse Docker Engine with Docker CLI.
Remember:
Docker CLI sends commands.
Docker Engine performs the work.

# Docker CLI

## What is Docker CLI?
CLI stands for **Command Line Interface**.
Docker CLI is the interface through which users communicate with Docker.
Every Docker command starts from the CLI.

Examples:
docker run nginx
docker ps
docker images
docker stop my-nginx

## Responsibilities
Docker CLI is responsible for:
* Accepting user commands
* Sending requests to Docker Daemon
* Displaying results

## Real World Example
Think about using Google Search.
You type a query.
Google processes it and returns results.
Similarly,
You type Docker commands.
Docker CLI forwards those commands to Docker Engine.


## Important Note
Docker CLI **does not create containers.**
It only communicates with Docker Engine.


# Docker Daemon

## What is Docker Daemon?
Docker Daemon (dockerd) is the background service that manages Docker resources.
It continuously runs in the background waiting for Docker commands.
Whenever Docker CLI sends a request, Docker Daemon processes it.

## Responsibilities
Docker Daemon can:

* Pull Images
* Push Images
* Build Images
* Create Containers
* Start Containers
* Stop Containers
* Delete Containers
* Manage Networks
* Manage Volumes


## Workflow
Docker CLI

↓

Docker Daemon

↓

Docker Engine Operations

## Real World Example
Customer

↓

Reception

↓

Kitchen
Docker CLI = Reception
Docker Daemon = Kitchen
The kitchen performs all the work.

## Interview Tip
Without Docker Daemon, Docker commands cannot execute.


# Docker Compose

## What is Docker Compose?
Docker Compose is a tool used to define and manage multiple containers using a single YAML configuration file.
Instead of running multiple commands manually, Docker Compose allows us to start all required containers with one command.


## Example
Suppose an application requires:

* Frontend
* Backend
* Database
* Redis

Without Docker Compose:
docker run frontend
docker run backend
docker run mysql
docker run redis

With Docker Compose:
docker compose up

Everything starts automatically.

## Compose File
Docker Compose uses:
docker-compose.yml

or
compose.yaml


## Benefits

* Easy multi-container deployment
* Simple configuration
* Better readability
* Easier maintenance
* Faster development

## Real World Example
Imagine organizing a cricket tournament.
Without a manager:
You call every player individually.
With a manager:
One phone call reaches the entire team.
Docker Compose works like that manager.


## Note
Docker Compose will be covered in detail on future Docker learning days.


# Docker Hub

## What is Docker Hub?
Docker Hub is the default public Docker Registry.
It stores Docker Images.
Whenever Docker cannot find an image locally, it downloads it from Docker Hub.


## Popular Images Available
* Ubuntu
* Nginx
* MySQL
* PostgreSQL
* Redis
* MongoDB
* Jenkins
* Python
* Node.js


## Docker Hub Workflow

```text
docker run nginx

↓

Check Local Image

↓

Not Found

↓

Docker Hub

↓

Download Image

↓

Create Container


## Real World Example
Think about Amazon.
Products are stored in Amazon warehouses.
Similarly,
Docker Images are stored inside Docker Hub.

## Interview Tip
Docker Hub stores Images.
It does not store running Containers.


# containerd

## What is containerd?
containerd is a container runtime responsible for managing the lifecycle of containers.
It handles:

* Image transfer
* Container execution
* Storage
* Networking

Docker Engine internally uses containerd.

## Why Should You Know This?
Although beginners rarely interact directly with containerd, Kubernetes uses container runtimes such as containerd extensively.
Understanding this helps in future Kubernetes learning.


# runc
## What is runc?
runc is a lightweight container runtime responsible for actually creating and starting Linux containers.
Docker Engine uses containerd.
containerd uses runc.


## Simplified Workflow
Docker CLI

↓

Docker Daemon

↓

Docker Engine

↓

containerd

↓

runc

↓

Linux Container

# Complete Docker Component Architecture

                    User
                      │
                      ▼
                Docker CLI
                      │
                      ▼
               Docker Daemon
                      │
                      ▼
               Docker Engine
                      │
              ┌───────┴────────┐
              │                │
              ▼                ▼
        Docker Hub        Local Images
              │
              ▼
          containerd
              │
              ▼
             runc
              │
              ▼
      Running Docker Container


# Docker Components Summary
| Component      | Responsibility                            |
| -------------- | ----------------------------------------- |
| Docker Desktop | Complete Docker package for Windows/macOS |
| Docker Engine  | Core container runtime                    |
| Docker CLI     | Accepts Docker commands                   |
| Docker Daemon  | Executes Docker operations                |
| Docker Compose | Multi-container management                |
| Docker Hub     | Stores Docker Images                      |
| containerd     | Container lifecycle management            |
| runc           | Creates Linux containers                  |


# Common Mistakes
* Thinking Docker Desktop and Docker Engine are the same.
* Confusing Docker CLI with Docker Daemon.
* Assuming Docker Hub stores running containers.
* Believing Docker Compose is mandatory for every container.
* Ignoring the difference between Docker Engine and Docker Desktop.


# Best Practices
* Learn Docker CLI before relying on the Docker Desktop GUI.
* Use official Docker Hub images whenever possible.
* Understand Docker Engine architecture before learning Kubernetes.
* Practice Docker commands regularly.
* Use Docker Compose for multi-container applications.


# Quick Revision

* Docker Desktop = Complete package
* Docker Engine = Core runtime
* Docker CLI = User Interface
* Docker Daemon = Background service
* Docker Compose = Multi-container tool
* Docker Hub = Image repository
* containerd = Container manager
* runc = Container runtime

# Interview Questions

### 1. What is Docker Engine?
Docker Engine is the core runtime responsible for creating, running, and managing Docker containers.


### 2. What is the difference between Docker Engine and Docker Desktop?
Docker Engine is the runtime that manages containers.
Docker Desktop is a complete application that includes Docker Engine, Docker CLI, Docker Compose, and other tools for Windows and macOS.


### 3. What is Docker CLI?
Docker CLI is the command-line interface used to communicate with Docker.


### 4. What is Docker Daemon?
Docker Daemon is the background service that executes Docker commands and manages Docker resources.

### 5. What is Docker Compose?
Docker Compose is a tool used to manage multiple containers using a YAML configuration file.


### 6. What is Docker Hub?
Docker Hub is the default public registry used to store and distribute Docker images.

### 7. What is containerd?
containerd is a container runtime responsible for managing the lifecycle of containers.

### 8. What is runc?
runc is the low-level runtime that creates and starts Linux containers.


# Task 2 – Docker Installation, Verification, and First Container

## Objective
The objective of this task is to:
* Install Docker
* Verify the installation
* Understand the Docker installation process
* Run the first Docker container
* Understand what happens internally when a container starts

# Docker Installation

## Installing Docker on Ubuntu
Docker can be installed from the official Docker repository or the Ubuntu repository.
After installation, the Docker service (Docker Daemon) starts automatically.

> **Note**
>
> In production environments, always install Docker from the official Docker repository to receive the latest stable releases.


# Verify Docker Installation
Installing Docker is not enough.

Always verify that:
* Docker CLI is installed.
* Docker Engine is running.
* Docker Daemon is responding.

# Command 1 – Check Docker Version

## Syntax
docker --version

## Purpose
Displays the installed Docker CLI version.

## Example
docker --version


## Sample Output
Docker version 29.1.3, build 29.1.3

## Command Breakdown

| Part      | Description                      |
| --------- | -------------------------------- |
| docker    | Docker CLI                       |
| --version | Display installed Docker version |

## What Does It Verify?
* Docker CLI is installed.
* The `docker` command is recognized by the operating system.

## What It Does NOT Verify
This command **does not confirm** that Docker Engine is running.
Many beginners misunderstand this.

Example:
Docker CLI may be installed, but Docker Daemon may be stopped.
In that case:
docker --version

works successfully,
while
docker info

fails.

# Command 2 – Check Docker Information
## Syntax
docker info


## Purpose
Displays complete information about Docker Engine.
This command is one of the most important troubleshooting commands.


## Example
docker info

## Important Sections

### Client
Shows information about Docker CLI.

Example:
Client:
Version: 29.x.x

### Server
Shows information about Docker Engine.
If this section exists,
Docker Daemon is running correctly.


### Containers
Displays:

* Running Containers
* Paused Containers
* Stopped Containers


### Images
Displays the number of downloaded images.


### Storage Driver
Shows the storage driver used by Docker.

Example:
overlay2

or
overlayfs

### Docker Root Directory
Example:
/var/lib/docker


This is where Docker stores:

* Images
* Containers
* Volumes
* Metadata


### Operating System
Shows the operating system where Docker is running.

Example:
Ubuntu 24.04


### CPUs
Displays the number of CPUs available to Docker.


### Total Memory
Displays the memory available to Docker.


# Why Is docker info Important?
A DevOps Engineer usually runs this command first during troubleshooting because it immediately shows whether Docker Engine is healthy.


# Common Error
Cannot connect to the Docker daemon

Reason:
Docker Daemon is not running.
Possible Solutions:
sudo systemctl start docker
or

sudo systemctl restart docker

Verify again:
docker info

# Running the First Container

## Command
docker run hello-world


# Command Breakdown

| Part        | Description                  |
| ----------- | ---------------------------- |
| docker      | Docker CLI                   |
| run         | Create and start a container |
| hello-world | Docker Image                 |


# What Does docker run Actually Do?
Many beginners think `docker run` simply starts a container.
In reality, it performs multiple operations automatically.
Internally, Docker behaves like this:

docker pull (if image does not exist)

↓

docker create

↓

docker start
One command performs all three operations.

# Internal Workflow
Suppose you execute:

docker run hello-world


Docker performs the following steps.


## Step 1
Docker CLI receives the command.

User

↓

Docker CLI


## Step 2
Docker CLI sends the request to Docker Daemon.

Docker CLI

↓

Docker Daemon


## Step 3
Docker Daemon checks whether the image exists locally.

Question:
Is hello-world image available?


Two possibilities exist.

### Case 1
Image already exists.
Docker immediately creates the container.


### Case 2
Image does not exist.

Docker downloads the image from Docker Hub.


## Step 4
Docker creates a container using the image.

Relationship:

Image

↓

Container

## Step 5
Docker starts the container.

## Step 6
The application inside the container executes.
The Hello World program prints a success message.

## Step 7
The program finishes execution.
Since the main process exits,
the container automatically stops.


# Complete Workflow
User

↓

docker run hello-world

↓

Docker CLI

↓

Docker Daemon

↓

Check Local Image

↓

Pull Image (If Required)

↓

Create Container

↓

Start Container

↓

Application Executes

↓

Main Process Ends

↓

Container Stops

# Understanding the Output
Example:
Unable to find image 'hello-world:latest' locally

Meaning:
Docker searched the local machine.
The image was not found.


Example:
Pulling from library/hello-world

Meaning:
Docker started downloading the image from Docker Hub.

Example:
Downloaded newer image

Meaning:
The image has been successfully downloaded.
It is now stored locally.
Future executions will reuse the same image.

Example:
Hello from Docker!

Meaning:
Everything worked successfully.
This confirms:
* Docker CLI
* Docker Daemon
* Docker Hub
* Docker Image
* Docker Container

are all functioning correctly.

# Why Did the Hello World Container Stop?
The Hello World application performs only one task.

Print Message

↓

Exit
Once the main application exits,
the container also exits.


# Golden Rule
A Docker Container continues running only while its **main process** is running.
When the main process exits,
the container stops automatically.

# Real Project Example
Suppose an Nginx container is running.
The Nginx web server remains active.
Therefore,
the container also remains active.
If the Nginx process crashes,
the container stops.


# Difference Between Hello World and Nginx

| Hello World       | Nginx             |
| ----------------- | ----------------- |
| Prints Message    | Runs Web Server   |
| Exits Immediately | Continues Running |
| Stopped Container | Running Container |

# Important Notes

* Docker automatically downloads missing images.
* Images are downloaded only once.
* Containers are created from images.
* Images remain available after containers are removed.


# Best Practices
* Always verify Docker using both `docker --version` and `docker info`.
* Read the output of `hello-world` carefully instead of ignoring it.
* Understand the complete workflow of `docker run`.
* Remember that `docker run` performs multiple operations internally.


# Common Mistakes
* Assuming `docker --version` confirms Docker Engine is running.
* Confusing Images with Containers.
* Thinking Docker downloads the image every time.
* Assuming containers never stop automatically.

# Quick Revision
* `docker --version` → Check Docker CLI.
* `docker info` → Verify Docker Engine.
* `docker run` → Pull → Create → Start.
* `hello-world` exits because its main process finishes.
* Images remain even after containers are removed.


# Interview Questions and Answers

## 1. What is the purpose of `docker --version`?
It displays the installed Docker CLI version.

## 2. What is the purpose of `docker info`?
It verifies Docker Engine and displays detailed Docker configuration information.


## 3. What happens internally when `docker run hello-world` is executed?
Docker checks for the image locally, downloads it if necessary, creates a container, starts it, executes the application, and stops the container after the application exits.

## 4. Why did Docker download the image?
Because it was not available on the local machine.


## 5. Why did the Hello World container stop automatically?
Because its main application completed execution.


## 6. Does Docker download the image every time?
No.
Docker downloads the image only if it does not already exist locally.


## 7. What does "Unable to find image locally" mean?
Docker searched the local image cache but could not find the requested image.

## 8. What is the most useful Docker troubleshooting command?
docker info
because it verifies whether Docker Engine is running and displays detailed environment information.

# Task 3 – Run Real Containers

## Objective
The objective of this task is to learn how to work with real Docker containers.
After completing this task, you will be able to:

* Run a web server inside a container
* Access a container from a browser
* Understand detached mode
* Understand interactive mode
* List running containers
* List stopped containers
* Stop containers
* Remove containers

# Running an Nginx Container

## Why Nginx?
Nginx is one of the most popular web servers in the world.
It is commonly used for:
* Hosting websites
* Reverse Proxy
* Load Balancing
* API Gateway
* Static File Hosting

Learning Docker with Nginx is an industry standard.

# Command
docker run -d -p 8080:80 --name my-nginx nginx

# Command Breakdown

## docker
Docker CLI command.


## run
Creates and starts a new container.
If the image does not exist locally, Docker downloads it automatically.

## -d

### Detached Mode
Runs the container in the background.
Without `-d`:

The terminal remains occupied.
With `-d`:
The terminal is immediately available for the next command.

## -p
### Port Mapping
Syntax
HostPort:ContainerPort

Example
8080:80

Meaning
Host Machine
Port:
8080

↓
Docker forwards traffic to
Container Port
80
## Why Is Port Mapping Needed?
Containers have their own network.
Without port mapping,
the application inside the container cannot be accessed from outside.


## Real World Example
Imagine an apartment building.
The apartment number is:
80
The building gate number is:
8080

Visitors first reach the building gate,
then they are directed to the apartment.
Host Port works like the gate.
Container Port works like the apartment.

## --name
Assigns a custom name to the container.

Example
my-nginx

Without this option Docker generates random names such as:
happy_panda

boring_tesla

angry_newton

Using meaningful names makes container management much easier.

## nginx
This is the Docker Image.
Docker creates a container using this image.


# Internal Workflow
docker run

↓

Check Image

↓

Download Image (If Missing)

↓

Create Container

↓

Start Container

↓

Nginx Starts

↓

Container Running

# Expected Output
Docker returns a long hexadecimal value.
Example
2d6e6e4c53b4d5e7c6a8d4d...

This is the Container ID.

# Browser Access
If Docker is running on an AWS EC2 instance,
open:
http://<EC2-Public-IP>:8080

You should see:
Welcome to nginx!


# Common Problem
Browser cannot open the page.
Possible Reasons:
* Security Group not allowing the host port
* Wrong port mapping
* Container is stopped

# Example
Container
80
Host
8080

Correct Command
docker run -d -p 8080:80 nginx
If Security Group only allows port 8080,
using
-p 80:80
will not work.

# Interactive Containers
Some containers are used interactively.
Ubuntu is the most common example.

# Command
docker run -it --name my-ubuntu ubuntu bash

# Command Breakdown

## -i
Interactive Mode
Keeps standard input open.
Allows users to type commands.


## -t
Allocates a terminal.
Provides a Linux shell experience.


## bash
Starts Bash immediately after the container starts.


# Why Use Interactive Mode?
Interactive mode allows us to:

* Learn Linux
* Debug applications
* Test commands
* Explore the container

# Commands Practiced
pwd
Shows current working directory.
ls
Lists files and directories.
cat /etc/os-release

Displays operating system information.
hostname
Displays the container hostname.
Normally,
the hostname is the Container ID.
exit
Leaves the container.

# Why Did the Ubuntu Container Stop?
The main process inside the container was:

After executing
exit

the Bash process ended.
When the main process exits,
the container also exits.


# Golden Rule
A Docker container runs only while its main process is running.


# Listing Containers
# docker ps

## Command
docker ps
## Purpose
Displays only running containers.

## Example Output
my-nginx
Since Nginx keeps running,
it appears here.


# docker ps -a

## Command
docker ps -a

## Purpose
Displays every container.

Including:

* Running
* Exited
* Stopped

# Example
my-nginx

my-ubuntu

hello-world

# Understanding STATUS
Example
Up 5 minutes

Meaning
Container is currently running.
Example
Exited (0)

Meaning
Container finished successfully.
Exit Code
0

means success.

# Difference Between docker ps and docker ps -a

| docker ps                   | docker ps -a                |
| --------------------------- | --------------------------- |
| Running Containers          | All Containers              |
| Excludes Stopped Containers | Includes Stopped Containers |

# Stopping Containers

## Command
docker stop my-nginx

## Purpose
Stops a running container.

# Removing Containers

## Command
docker rm my-nginx

## Purpose
Removes a stopped container.

# Why Must We Stop Before Removing?
Docker protects running containers.
Trying to remove a running container results in an error.
Correct sequence:
docker stop

↓

docker rm

# Force Remove
docker rm -f my-nginx

This command:

* Stops the container
* Removes the container

in one step.

# Real Project Example
Production Server

EC2

↓

Docker

↓

Nginx Container

↓

Users

When a new application version is available:

Stop old container

↓

Remove old container

↓

Run new container
This is a common deployment workflow.


# Best Practices
* Use meaningful container names.
* Use detached mode for services.
* Use interactive mode only when required.
* Remove unused containers regularly.
* Verify containers using `docker ps`.


# Common Mistakes
* Forgetting port mapping.
* Using the wrong host port.
* Forgetting to stop containers before removing them.
* Confusing host ports with container ports.
* Assuming exited containers are deleted automatically.


# Quick Revision
* `docker run` → Create and Start
* `-d` → Detached Mode
* `-it` → Interactive Terminal
* `docker ps` → Running Containers
* `docker ps -a` → All Containers
* `docker stop` → Stop Container
* `docker rm` → Remove Container

# Interview Questions

### 1. Why do we use detached mode?
Detached mode allows containers to run in the background without occupying the terminal.

### 2. What is the difference between `-d` and `-it`?
`-d` runs the container in the background.
`-it` opens an interactive terminal inside the container.

### 3. What is port mapping?
Port mapping connects a host port to a container port using the `-p` option.

### 4. Why did the Ubuntu container stop after typing `exit`?
Because the Bash process ended, and the main process of the container exited.

### 5. Why did the Nginx container continue running?
Because the Nginx service continued running as the container's main process.


### 6. What is the difference between `docker ps` and `docker ps -a`?
`docker ps` shows only running containers, while `docker ps -a` shows all containers.

### 7. Why can't Docker remove a running container?
Docker prevents accidental deletion of active containers. The container must be stopped before it can be removed.

# Task 4 – Explore Docker

## Objective
The objective of this task is to become comfortable interacting with running containers and understanding how Docker manages them.
In this task, you will learn:

* Detached Mode
* Interactive Mode
* Custom Container Names
* Port Mapping
* Viewing Logs
* Executing Commands Inside Running Containers

# Detached Mode

## What is Detached Mode?
Detached Mode allows a container to run in the background.
Instead of attaching your terminal to the container, Docker immediately returns the terminal so you can continue executing other commands.

## Command
docker run -d nginx

## Why Use Detached Mode?
Without Detached Mode:

* Terminal remains attached to the container.
* You cannot easily execute other commands.

With Detached Mode:
* Container continues running.
* Terminal becomes available immediately.
  
## Real Project Usage
Almost every production container runs in Detached Mode.
Examples:
* Nginx
* Apache
* Jenkins
* SonarQube
* GitLab
* Prometheus
* Grafana

# Interactive Mode

## What is Interactive Mode?
Interactive Mode allows users to interact directly with a container through a terminal.

## Command
docker run -it ubuntu bash

## Why Use Interactive Mode?
Interactive Mode is useful for:

* Learning Linux
* Debugging
* Testing commands
* Inspecting files
* Troubleshooting applications
  
# Detached Mode vs Interactive Mode

| Detached Mode           | Interactive Mode                |
| ----------------------- | ------------------------------- |
| Runs in background      | Opens terminal                  |
| Used for servers        | Used for debugging              |
| Terminal is free        | Terminal remains attached       |
| Production environments | Development and troubleshooting |

# Container Naming
## Why Use Container Names?
Docker automatically assigns random names if a name is not provided.

Example:
happy_einstein

nostalgic_turing

boring_tesla
Random names are difficult to remember.
Meaningful names improve readability.

## Command
docker run --name my-nginx nginx

## Best Practice
Use names that describe the application.

Examples:
frontend

backend

mysql-db

redis-cache

jenkins-server

prometheus

# Port Mapping

## Why Is Port Mapping Required?
Containers have their own isolated network.
Applications running inside containers are not directly accessible from the host.
Port mapping connects the host machine to the container.

## Syntax
HostPort:ContainerPort

## Example
docker run -p 8080:80 nginx

Meaning:
Browser

↓

Host Port 8080

↓

Docker

↓
Container Port 80

## AWS Example
Suppose:
Security Group allows:
8080

Then the container should be started using:
docker run -d -p 8080:80 nginx

Opening:
http://EC2-Public-IP:8080
will access the web server.


# Viewing Container Logs
## What are Logs?
Logs are records generated by applications running inside containers.
They help administrators monitor applications and troubleshoot problems.


## Command
docker logs my-nginx

## What Does It Show?
* Startup messages
* Errors
* Warnings
* Access logs
* Application output

## Why Are Logs Important?
Suppose an application crashes.
The first command a DevOps Engineer usually executes is:

docker logs <container-name>
Logs often reveal the root cause.


## Real Project Example
An application is not accessible.
Instead of restarting immediately,

first inspect:
docker logs backend
The logs may show:

* Database connection failure
* Missing environment variable
* Port conflict
* Configuration error

# Executing Commands Inside a Running Container

## What is docker exec?
The `docker exec` command allows users to execute commands inside an already running container.


## Command
docker exec -it my-nginx bash

If Bash is unavailable:
docker exec -it my-nginx sh

## Command Breakdown

| Option   | Description                              |
| -------- | ---------------------------------------- |
| exec     | Execute command inside running container |
| -i       | Interactive input                        |
| -t       | Allocate terminal                        |
| my-nginx | Container name                           |
| bash     | Start Bash shell                         |


# What Can You Do Inside the Container?
Examples:
pwd
ls
cat /etc/os-release
nginx -v
ps -ef
exit

# docker run vs docker exec
| docker run              | docker exec                        |
| ----------------------- | ---------------------------------- |
| Creates a new container | Uses an existing running container |
| Starts application      | Executes commands inside container |
| Requires an image       | Requires a running container       |

# Real Project Example
Suppose a production application is failing.
Incorrect approach:
Delete the container immediately.
Correct approach:
docker logs backend
docker exec -it backend bash
Investigate first.
Then decide the next action.


# Important Docker Commands Cheat Sheet

| Command          | Purpose                                |
| ---------------- | -------------------------------------- |
| docker --version | Show Docker version                    |
| docker info      | Show Docker information                |
| docker images    | List downloaded images                 |
| docker ps        | Show running containers                |
| docker ps -a     | Show all containers                    |
| docker run       | Create and start container             |
| docker stop      | Stop container                         |
| docker start     | Start stopped container                |
| docker restart   | Restart container                      |
| docker rm        | Remove container                       |
| docker rmi       | Remove image                           |
| docker logs      | View logs                              |
| docker exec -it  | Enter running container                |
| docker inspect   | Display detailed container information |
| docker pull      | Download image                         |
| docker stats     | Show live resource usage               |


# Frequently Used Docker Flags
| Flag   | Description                 |
| ------ | --------------------------- |
| -d     | Detached Mode               |
| -it    | Interactive Terminal        |
| -p     | Port Mapping                |
| --name | Container Name              |
| -f     | Force                       |
| --rm   | Remove container after exit |


# Common Troubleshooting
## Problem
Browser cannot access container.

### Possible Causes
* Wrong port mapping
* Container stopped
* Security Group blocked
* Firewall issue

## Problem
Container exited immediately.

### Possible Causes
* Main process finished
* Incorrect startup command
* Application crashed


## Problem
Cannot connect to Docker daemon.

### Solution
sudo systemctl status docker
sudo systemctl restart docker

## Problem
Container name already exists.

### Solution
docker rm <container-name>
or
docker run --name different-name

# Best Practices
* Use meaningful container names.
* Keep images updated.
* Remove unused containers.
* Monitor container logs regularly.
* Use official Docker Hub images.
* Verify Docker installation before troubleshooting.
* Use Docker Compose for multi-container applications.
* Never expose unnecessary ports.
* Document all container configurations.
  
# Common Beginner Mistakes
* Confusing Images with Containers.
* Forgetting port mapping.
* Forgetting Security Group rules.
* Using `docker run` instead of `docker exec`.
* Removing running containers.
* Ignoring container logs.
* Using random container names.


# Complete Day 29 Quick Revision
## Docker Concepts
* Docker packages applications into containers.
* Containers share the Host OS Kernel.
* Images create Containers.
* Docker Hub stores Images.

## Docker Workflow
Docker CLI

↓

Docker Daemon

↓

Image

↓

Container

↓

Application

## Commands You Practiced
docker --version
docker info
docker run hello-world
docker run -d -p 8080:80 --name my-nginx nginx
docker run -it ubuntu bash
docker ps
docker ps -a
docker stop
docker rm
docker logs
docker exec -it

# Day 29 Interview Questions

## Basic Level
1. What is Docker?
2. Why was Docker created?
3. What is a Container?
4. What is a Docker Image?
5. What is Docker Hub?
6. Explain Docker Architecture.
7. What is Docker Engine?
8. What is Docker CLI?
9. What is Docker Daemon?
10. What is Docker Compose?


## Intermediate Level
1. Difference between Containers and Virtual Machines.
2. Difference between Docker Image and Docker Container.
3. Difference between `docker run` and `docker exec`.
4. Difference between `docker ps` and `docker ps -a`.
5. Difference between Detached Mode and Interactive Mode.
6. Explain Port Mapping.
7. Why do containers stop automatically?
8. Why is `hello-world` used?
9. Explain Docker workflow.
10. Explain the purpose of Docker logs.


# Scenario-Based Interview Questions
### Scenario 1
An application is not opening in the browser.
What will you check?

**Answer**
* Container Status
* Port Mapping
* Security Group
* Firewall
* Application Logs


### Scenario 2
A running container suddenly stopped.
How will you troubleshoot?

**Answer**
* Check logs
* Check exit code
* Inspect container
* Verify application process


### Scenario 3
You need to investigate an application without creating a new container.
Which command will you use?
docker exec -it <container-name> bash

### Scenario 4
A developer says, "The application works on my laptop but not on the server."
How does Docker solve this problem?

**Answer**
Docker packages the application together with all required dependencies, ensuring the same environment across development, testing, and production.


### Scenario 5
You accidentally stopped a production container.
How do you start it again?
docker start <container-name>


# Practice Lab
Complete these tasks without referring to notes.
1. Verify Docker installation.
2. Run Hello World.
3. Run an Nginx container.
4. Access it through the browser.
5. Run an Ubuntu interactive container.
6. View running containers.
7. View all containers.
8. Stop a container.
9. Remove a container.
10. View container logs.
11. Enter a running container.
12. Explain the Docker workflow.

# Summary
Day 29 introduced the core concepts of Docker and containerization. You learned why Docker was created, how containers differ from virtual machines, and how Docker Architecture works. You explored Docker components, verified the installation, ran your first containers, accessed applications through port mapping, interacted with containers, viewed logs, managed the container lifecycle, and practiced the most commonly used Docker commands.

These concepts form the foundation for all upcoming Docker topics, including Docker Images, Dockerfile, Volumes, Networking, Docker Compose, Kubernetes, and modern DevOps workflows.



