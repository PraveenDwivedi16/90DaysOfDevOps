# Day 50 – Kubernetes Architecture and Cluster Setup
##  Kubernetes Story

Kubernetes was created to solve the challenges of managing containerized applications at scale. While Docker made it easy to run containers, managing hundreds of containers across multiple servers became complex.

Manual container management introduced several issues such as difficulty in scaling, lack of self-healing, complex networking, and no built-in load balancing.

Kubernetes automates deployment, scaling, networking, and management of containers, making it easier to handle large-scale applications.

Kubernetes was developed by Google and inspired by its internal system called Borg. It is written in Go language.

The word "Kubernetes" comes from Greek, meaning "Helmsman" or "Ship Captain", which represents controlling and managing containerized applications.


##  Kubernetes Architecture

Kubernetes architecture consists of two main components:

###  Control Plane

* **API Server**: Acts as the entry point for all commands and communication.
* **etcd**: A key-value database that stores the cluster state.
* **Scheduler**: Assigns pods to appropriate worker nodes.
* **Controller Manager**: Ensures the desired state matches the actual state.

###  Worker Node

* **kubelet**: Node agent that receives instructions and manages pods.
* **kube-proxy**: Handles networking and communication between pods.
* **Container Runtime**: Runs the actual containers (e.g., containerd).


## kubectl apply Flow
When we run:
kubectl apply -f pod.yaml

The following happens:

1. The request is sent to the API Server
2. The API Server validates the request
3. The desired state is stored in etcd
4. The Scheduler assigns a node
5. The kubelet ensures the pod runs on that node
6. The container runtime starts the container

##  kubectl Installation

kubectl is a command-line tool used to interact with the Kubernetes cluster.

### Installation (Windows)

kubectl was already installed on the system.

### Verification

kubectl version --client

##  Cluster Setup (kind)

For local Kubernetes cluster setup, **kind (Kubernetes in Docker)** was used.

### Why kind?

* Lightweight and fast
* Uses Docker containers as nodes
* Ideal for DevOps practice

### Create Cluster
kind create cluster --name devops-cluster
### Verify Cluster
kubectl cluster-info
kubectl get nodes

##  Cluster Exploration

### List Namespaces
kubectl get namespaces

### List All Pods
kubectl get pods -A

### kube-system Pods
kubectl get pods -n kube-system

### kube-system Components

* **kube-apiserver**: Entry point of the cluster
* **etcd**: Stores cluster data
* **kube-scheduler**: Assigns pods to nodes
* **kube-controller-manager**: Maintains desired state
* **coredns**: Handles DNS and service discovery
* **kube-proxy**: Manages networking rules


## 🔄 Cluster Lifecycle

### Delete Cluster
kind delete cluster --name devops-cluster

### Recreate Cluster
kind create cluster --name devops-cluster

### Verify
kubectl get nodes

## kubeconfig
kubeconfig is a configuration file used by kubectl to connect to the Kubernetes cluster.

It stores:

* Cluster details
* User credentials
* Context information

### Default Location
~/.kube/config
On Windows:

C:\Users\<your-user>\.kube\config

# Day 50 – Kubernetes Interview Questions & Answers

1. What is Kubernetes?
Answer:

Kubernetes is an open-source container orchestration platform used to automate deployment, scaling, networking, and management of containerized applications.

Simple:
👉 Docker run container 
👉 Kubernetes manage containers

2. Why do we need Kubernetes if Docker already exists?

Answer:
Docker only runs containers, but Kubernetes manages containers at scale.

Docker limitations:
No auto-scaling
No self-healing
No built-in load balancing
Difficult multi-server management

Kubernetes solves these problems.

3. Who created Kubernetes?
Answer:
Kubernetes was originally developed by Google and inspired by Google's internal system called Borg.

It is written in Go language.

4. What is the meaning of Kubernetes?
Answer:
Kubernetes is a Greek word meaning:

👉 Helmsman or Ship Captain
It represents controlling and managing containerized applications.

5. What are the main components of Kubernetes architecture?
Answer:
Kubernetes architecture has two parts:

# Control Plane
API Server
etcd
Scheduler
Controller Manager
# Worker Node
kubelet
kube-proxy
Container Runtime

6. What is API Server?
Answer:
API Server is the entry point of Kubernetes.

All requests and kubectl commands go through API Server.
Example:
kubectl get pods
This request first goes to API Server.

7. What is etcd?
Answer:
etcd is a key-value database used to store Kubernetes cluster state.

It stores:
Pods
Nodes
Configurations
Secrets

8. What is Scheduler?
Answer:
Scheduler decides:
"Which node should run the pod?"

It checks:
CPU
Memory
Resources
and assigns the best node.

9. What is Controller Manager?
Answer:
Controller Manager ensures:
Desired state = Actual state

Example:
If 3 pods are required and 1 crashes → it creates a new pod automatically.

10. What is kubelet?
Answer:
kubelet is an agent running on every worker node.
Responsibilities:
Talks to API Server
Ensures pods are running

11. What is kube-proxy?
Answer:
kube-proxy manages networking rules.

It allows:
Pod communication
Service routing

12. What is Container Runtime?
Answer:
Container Runtime runs actual containers.
Examples:
containerd
CRI-O
Earlier Docker was used.

13. What happens when you run:
kubectl apply -f pod.yaml

Answer:
Flow:
Request goes to API Server
API Server validates request
Data stored in etcd
Scheduler selects node
kubelet runs pod
Container runtime starts container

14. What happens if API Server goes down?
Answer:
New deployments stop 
kubectl commands stop 
Running applications continue 

15. What happens if Worker Node goes down?
Answer:
Pods running on that node fail.
Kubernetes automatically creates new pods on another healthy node.
(Self-healing)

16. What is kubectl?
Answer:
kubectl is a command-line tool used to interact with Kubernetes clusters.
Common commands:
kubectl get pods
kubectl get nodes
kubectl describe pod

17. What is kind?
Answer:
kind stands for:
Kubernetes IN Docker
It creates local Kubernetes clusters using Docker containers.

18. What is kubeconfig?
Answer:
kubeconfig is a configuration file used by kubectl to connect to Kubernetes clusters.
It stores:
Cluster details
Credentials

Context
Default location:
~/.kube/config

Windows:
C:\Users\<user>\.kube\config

🔥 SUPER IMPORTANT INTERVIEW POINTS
1. Kubernetes = Declarative

❌ Wrong thinking:

Run this container

✅ Correct thinking:

I want 3 pods

Kubernetes maintains desired state.

2. Pods are smallest unit

Not container.

Interview trap question ❗

❌ Container
✅ Pod

3. API Server = Heart of Kubernetes

Every command goes through API Server.

4. etcd is VERY critical

If etcd data lost:
👉 cluster state lost

5. Kubernetes components run as Pods

This is a favorite interview question.

Example:

kubectl get pods -n kube-system
6. Kubernetes ≠ Docker replacement

Interview trap ❗

❌ Kubernetes replaces Docker

✅ Correct:
Kubernetes manages containers.

7. kubectl talks to API Server

Not directly to worker node.

8. Scheduler only schedules

It does NOT run pods.

Interview trap ❗

kubelet runs pods.

9. Controller Manager maintains desired state

Example:
3 pods required → 1 deleted → auto recreate

10. Worker node = actual workload

Control plane manages.
Worker node executes.

🎯 Most Asked Freshers Questions
What is Kubernetes?
Why Kubernetes?
Difference between Docker and Kubernetes?
What is Pod?
What is kubelet?
What is API Server?
What happens when pod crashes?
What is etcd?
What is kind/minikube?
What happens when you run kubectl apply?