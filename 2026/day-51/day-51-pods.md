Day 51 – Kubernetes Pods & Manifests

1. What is a Kubernetes Manifest?
A Kubernetes manifest is a YAML file that defines the desired state of a resource.

Kubernetes reads this file and ensures the system matches the defined configuration.

2. Anatomy of a Kubernetes Manifest
Every manifest contains four main fields:

1. apiVersion
Defines the API version used.

Example:
apiVersion: v1

2. kind
Defines the resource type.

Example:
kind: Pod

3. metadata
Contains identifying data like name and labels.

Example:
metadata:
  name: nginx-pod
  labels:
    app: nginx

4. spec
Defines the desired state (what should run).

Example:
spec:
  containers:
  - name: nginx
    image: nginx

3. Pod Manifests

# Nginx Pod
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80

# BusyBox Pod
apiVersion: v1
kind: Pod
metadata:
  name: busybox-pod
  labels:
    app: busybox
    environment: dev
spec:
  containers:
  - name: busybox
    image: busybox:latest
    command: ["sh", "-c", "echo Hello from BusyBox && sleep 3600"]

# Multi Label Pod
apiVersion: v1
kind: Pod
metadata:
  name: multi-label-pod
  labels:
    app: myapp
    environment: staging
    team: devops
spec:
  containers:
  - name: nginx
    image: nginx

4. Imperative vs Declarative

# Imperative
Direct command-based approach.

Example:
kubectl run redis-pod --image=redis

Quick
Not reusable
Not recommended for production

# Declarative
YAML-based approach.

# Example:
kubectl apply -f pod.yaml
Reusable
Version controlled
Best for production

# Key Commands Used
kubectl get pods
kubectl describe pod <name>
kubectl logs <name>
kubectl exec -it <name> -- /bin/sh
kubectl apply -f file.yaml
kubectl delete pod <name>

# 6. Labels and Filtering
Labels are key-value pairs used to organize resources.

Examples:
kubectl get pods --show-labels
kubectl get pods -l app=nginx
kubectl get pods -l environment=dev

# 7. What happens when you delete a Pod?
When a standalone Pod is deleted:

It is permanently removed
It is NOT recreated automatically
There is no controller managing it

This is why Deployments are used in production.

# YAML FILES
1:- nginx-pod.yml

apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80

2:- busybox-pod.yml
apiVersion: v1
kind: Pod
metadata:
  name: busybox-pod
  labels:
    app: busybox
    environment: dev
spec:
  containers:
  - name: busybox
    image: busybox:latest
    command: ["sh", "-c", "echo Hello from BusyBox && sleep 3600"]

3:- multi-label-pod.yml
apiVersion: v1
kind: Pod
metadata:
  name: multi-label-pod
  labels:
    app: myapp
    environment: staging
    team: devops
spec:
  containers:
  - name: nginx
    image: nginx


# Day 51 – Kubernetes Pods & Manifests (Interview Q/A + Important Points)

1. What is Kubernetes?
Answer:
Kubernetes is a container orchestration platform used to deploy, manage, scale, and automate containerized applications.

2. What is a Pod in Kubernetes?
Answer:
A Pod is the smallest deployable unit in Kubernetes that contains one or more containers.

3. Why do we use Pods?
Answer:
Pods are used to run containers in Kubernetes.

Benefits:
Shared network
Shared storage
Same lifecycle

4. What is a Kubernetes Manifest?
Answer:
A Kubernetes manifest is a YAML file that defines the desired state of a Kubernetes resource.

Example:
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:

5. What are the four required fields in a Kubernetes manifest?
Answer:
apiVersion → Define API version 
kind → Define esource type 
metadata → Define Name & labels 
spec → Define Desired state 

6. What is apiVersion?
Answer:
It defines which Kubernetes API version should be used.

Example:
apiVersion: v1
Important:
Pod → v1
Deployment → apps/v1

7. What is metadata in Kubernetes?
Answer:
Metadata contains identifying information about a resource.

Example:
metadata:
  name: nginx-pod
  labels:
    app: nginx

8. What is spec in Kubernetes?
Answer:
Spec defines the desired state of the resource.

Example:
spec:
  containers:
  - name: nginx
    image: nginx

9. What is the difference between Imperative and Declarative approach?
# Imperative:
Direct command-based approach.

Example:
kubectl run nginx --image=nginx

# Declarative:
YAML-based approach.

Example:
kubectl apply -f nginx-pod.yaml
Important:
Production me mostly Declarative approach use hota hai.

10. Why is Declarative approach preferred?
Answer:
Because:
Reusable
Version controlled
Easy to manage
Best for GitOps

11. What is kubectl?
Answer:
kubectl is the command-line tool used to interact with Kubernetes clusters.

12. Difference between kubectl apply and kubectl create?
kubectl create
Creates a resource only once.
kubectl apply
Creates OR updates the resource.
Important:
Real projects:
kubectl apply -f file.yaml
mostly use this 

13. What does kubectl get pods do?
Answer:
Shows all running Pods.
Command:
kubectl get pods

14. What is kubectl describe pod?
Answer:
Provides detailed information about a Pod.
Shows:
Events
Errors
IP
Container info
Command:
kubectl describe pod nginx-pod

15. What is kubectl logs?
Answer:
Shows container logs/output.
Example:
kubectl logs busybox-pod
Output:
Hello from BusyBox

16. What is kubectl exec?
Answer:
Used to access the container shell.
Example:
kubectl exec -it nginx-pod -- /bin/bash

17. Why did BusyBox crash without command?
Answer:
BusyBox does not run a long-running process by default.
So:
Container starts → exits → Kubernetes restarts it → CrashLoopBackOff.

Golden Line:
Kubernetes expects a long-running process.

18. What is CrashLoopBackOff?
Answer:
A state where a container repeatedly crashes and Kubernetes continuously tries to restart it.

19. What are Labels in Kubernetes?
Answer:
Labels are key-value pairs used to organize and filter resources.
Example:
labels:
  app: nginx
  environment: dev

20. How do you filter Pods using labels?
Command:
kubectl get pods -l app=nginx

21. What is dry-run in Kubernetes?
Answer:
Dry-run validates configuration without creating resources.
Client Side:
kubectl apply -f pod.yaml --dry-run=client

Checks:
✔ Syntax only

Server Side:
kubectl apply -f pod.yaml --dry-run=server

Checks:
✔ Full validation

22. Difference between client and server dry-run?
Client	Server
Basic check	Full validation
Local	API server

23. What happens if image field is missing?
Answer:
Kubernetes throws an error:
spec.containers[0].image: Required value

24. What happens when a standalone Pod is deleted?
Answer:
The Pod is permanently deleted.
It will NOT be recreated automatically because no controller manages it.

IMPORTANT POINTS (INTERVIEW GOLD)
1.
Kubernetes manages Pods, not containers.

2.
YAML is the heart of Kubernetes.
3.
Always validate YAML before applying:
--dry-run=server

4.
Labels are heavily used in:
Services
Deployments
Monitoring
Filtering

❓Tricky Interview Questions
Q: Why is BusyBox crashing?
Ans:
No long-running process.

Q: Which command is better — create or apply?
Ans:
apply (because it updates existing resources)

Q: Why is Pod not recommended in production?
Ans:
Because Pod is not self-healing.

Q: Why use labels?
Ans:
For grouping and filtering resources.

What You Missed in Task (Important Additions)
1. kubectl get pods -o wide
Shows:
Pod IP
Node name
Command:
kubectl get pods -o wide

2. kubectl explain pod
Used to understand manifest fields.
Command:
kubectl explain pod
Advanced:
kubectl explain pod.spec

3. Generate YAML quickly
Command:
kubectl run test-pod --image=nginx --dry-run=client -o yaml


