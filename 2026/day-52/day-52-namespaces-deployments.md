# Day 52 – Kubernetes Namespaces and Deployments
## Objective
In Day 52, I learned about:

- Kubernetes Namespaces
- Deployments
- Self-healing
- Scaling
- Rolling Updates
- Rollback

# Task 1: Explore Default Namespaces

## What is a Namespace?
A Namespace is a logical separation inside a Kubernetes cluster.

It helps organize resources and separate environments.

Example:
- Development Environment → `dev`
- Testing Environment → `staging`
- Production Environment → `production`

## Check Default Namespaces
kubectl get namespaces

# Output showed:
default
kube-system
kube-public
kube-node-lease
local-path-storage

# Meaning of Default Namespaces
1. default
This is the default namespace.
If we do not mention a namespace, resources are created here.

# Example:
kubectl run nginx --image=nginx
This Pod will be created in the default namespace.

2. kube-system
This namespace contains internal Kubernetes components.

# Examples:
kube-apiserver
kube-scheduler
kube-controller-manager
coredns
etcd

# Command:
kubectl get pods -n kube-system
Important:
We should never delete Pods from kube-system because they are important for cluster working.

3. kube-public
This namespace contains publicly readable resources.

4. kube-node-lease
This namespace is used to track node heartbeat and health.

# Verification
Command:
kubectl get pods -n kube-system
I found 8 running Pods.

# Task 2: Create and Use Custom Namespaces

Create Namespaces
# Command:
kubectl create namespace dev
kubectl create namespace staging

# Verify:
kubectl get namespaces

# Result:
Namespaces were created successfully.

# Create Namespace Using YAML

File: namespace.yaml

apiVersion: v1
kind: Namespace
metadata:
  name: production

# Apply:
kubectl apply -f namespace.yaml

# Run Pods in Specific Namespaces

Commands:
kubectl run nginx-dev --image=nginx:latest -n dev
kubectl run nginx-staging --image=nginx:latest -n staging

# Check Pods
Default namespace Pods

kubectl get pods
This command only shows Pods from the default namespace.

# Specific Namespace Pods
kubectl get pods -n dev
kubectl get pods -n staging

# All Namespace pods
kubectl get pods -A 
This command shows Pods from all namespaces.

# I Learned
kubectl get pods → only default namespace
kubectl get pods -n <namespace> → specific namespace
kubectl get pods -A → all namespaces

# Task 3: Create First Deployment

# What is Deployment?
Deployment is a Kubernetes object that manages Pods.
It ensures the required number of Pods are always running.

# Example:
If replicas = 3, Kubernetes will always keep 3 Pods running.
If one Pod crashes, Kubernetes automatically creates a new Pod.

# Deployment YAML File

File: nginx-deployment.yml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: dev
  labels:
    app: nginx

spec:
  replicas: 3

  selector:
    matchLabels:
      app: nginx

  template:
    metadata:
      labels:
        app: nginx

    spec:
      containers:
      - name: nginx
        image: nginx:1.24
        ports:
        - containerPort: 80

# Deployment YAML Explanation
apiVersion
apps/v1
API version for Deployment.

# kind
Deployment
Defines resource type.

# metadata
Contains Deployment name and namespace.

# replicas
replicas: 3
Kubernetes will maintain 3 Pods.

# selector.matchLabels
Used to connect Deployment with Pods.
Labels must match Pod labels.

# template
This is Pod blueprint.
Deployment creates Pods using this template.

# containers
Defines container settings.
Example:
image: nginx:1.24
This image is used for Pod creation.

# Apply Deployment
Command:
kubectl apply -f nginx-deployment.yml

# Verify Deployment
Commands:
kubectl get deployments -n dev
kubectl get pods -n dev

# Deployment Output Meaning

Example:
READY      3/3
UP-TO-DATE 3
AVAILABLE  3
READY

3 Pods are running successfully.

UP-TO-DATE

All Pods are using latest configuration.

AVAILABLE

Pods are healthy and ready.

# Behind the Scene
Deployment creates ReplicaSet.
ReplicaSet creates Pods.

# Flow:

Deployment
    ↓
ReplicaSet
    ↓
Pods

# Task 4: Self-Healing
What is Self-Healing?
If a Pod is deleted, Deployment creates a new Pod automatically.

# Delete Pod
Command:
kubectl get pods -n dev

# Delete one Pod:
kubectl delete pod <pod-name> -n dev

# Check again:
kubectl get pods -n dev

# What Happened?
Kubernetes noticed:
Desired Pods = 3
Current Pods = 2

So it created a new Pod automatically.

# Important:
The new Pod name is different from the deleted Pod.

# Difference Between Pod and Deployment
# Standalone Pod
Deleted Pod is gone forever.

# Deployment Pod
Deleted Pod comes back automatically.

# Task 5: Scale Deployment
What is Scaling?
Scaling means increasing or decreasing the number of Pods.

# Scale Up
Increase Pods:

kubectl scale deployment nginx-deployment --replicas=5 -n dev

# Check:
kubectl get pods -n dev

# Result:
New Pods were created.
Some Pods first showed:
Pending

# Reason:
Kubernetes was preparing Pods.

# Steps:
Scheduler assigns node
Image pulls
Container starts
Pod becomes Running

# Scale Down
Reduce Pods:
kubectl scale deployment nginx-deployment --replicas=2 -n dev

# Check:
kubectl get pods -n dev

# Result:
Extra Pods moved to:
Terminating
Then Kubernetes removed them. 

# Two Ways to Scale
1. Imperative Method

Direct command:
kubectl scale deployment nginx-deployment --replicas=5

2. Declarative Method
Change replicas in YAML:

replicas: 4

Then apply:
kubectl apply -f nginx-deployment.yml

# Task 6: Rolling Update and Rollback
What is Rolling Update?
Rolling update means updating application version without downtime.
Pods are updated one by one.

# Update Deployment Image

Command:
kubectl set image deployment/nginx-deployment nginx=nginx:1.25 -n dev

# Check Rollout Status

Command:
kubectl rollout status deployment/nginx-deployment -n dev

Kubernetes replaced old Pods one by one.

# Why No Downtime?
Kubernetes rule:
Create new healthy Pod first
Then delete old Pod
This keeps application available.

# Check Rollout History

Command:
kubectl rollout history deployment/nginx-deployment -n dev

Result:
Revision history was stored.

# Rollback
Go back to old version:

kubectl rollout undo deployment/nginx-deployment -n dev

# Verify:
kubectl describe deployment nginx-deployment -n dev | grep Image

# Result:
Image changed back to:
nginx:1.24

# Task 7: Cleanup

# Delete resources:

kubectl delete deployment nginx-deployment -n dev

kubectl delete pod nginx-dev -n dev

kubectl delete pod nginx-staging -n staging

kubectl delete namespace dev staging production

# Verify Cleanup

Commands:
kubectl get namespaces
kubectl get pods -A

# Result:
Resources were removed successfully.
Important Interview Points

# What is Namespace?
Logical separation inside cluster.

# What is Deployment?
Deployment manages Pods and keeps required replicas running.

# What is Self-Healing?
Automatic Pod recreation.

# What is Scaling?
Increase or decrease Pod count.

# What is Rolling Update?
Update application without downtime.

# What is Rollback?
Restore previous stable version.


# Day 52 – Kubernetes Namespaces and Deployments
This document covers Kubernetes Namespaces and Deployments with hands-on commands, explanations, and real-world concepts.

1. What are Namespaces?
Namespaces are logical partitions inside a Kubernetes cluster used to organize and isolate resources.

# Why use Namespaces?
Environment separation (dev, staging, production)
Resource isolation
Access control (RBAC)

# Default Namespaces
Namespace	Description

# default	Default working namespace
# kube-system	Kubernetes internal components
# kube-public	Public resources
# kube-node-lease	Node heartbeat tracking

# Commands Used
kubectl get namespaces
kubectl get pods -n kube-system

# Custom Namespaces
kubectl create namespace dev
kubectl create namespace staging

Verify:
kubectl get namespaces

# Run Pods in Namespaces
kubectl run nginx-dev --image=nginx -n dev
kubectl run nginx-staging --image=nginx -n staging

Check:
kubectl get pods -A

2. What is a Deployment?
A Deployment ensures that a specified number of Pods are always running.

# Deployment YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: dev
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.24
        ports:
        - containerPort: 80

# YAML Explanation
replicas: Number of pods
selector: Connects Deployment to Pods
template: Pod blueprint
containers: Container details

# Apply Deployment
kubectl apply -f nginx-deployment.yaml

Check:
kubectl get deployments -n dev
kubectl get pods -n dev

# Self-Healing
Delete a pod:
kubectl delete pod <pod-name> -n dev

# Kubernetes automatically recreates it.
Scaling
Scale Up
kubectl scale deployment nginx-deployment --replicas=5 -n dev
Scale Down
kubectl scale deployment nginx-deployment --replicas=2 -n dev

# Rolling Update
Update image:
kubectl set image deployment/nginx-deployment nginx=nginx:1.25 -n dev

Check rollout:
kubectl rollout status deployment/nginx-deployment -n dev

# Rollback
kubectl rollout undo deployment/nginx-deployment -n dev
Verify:
kubectl describe deployment nginx-deployment -n dev | grep Image

# Key Differences
Feature	Pod	Deployment
Self-healing	
Scaling	
Rolling Update

# Cleanup
kubectl delete deployment nginx-deployment -n dev
kubectl delete namespace dev staging


# 20 Interview Questions & Answers (Day 52)

1. What is a Namespace?
Answer:
Namespace is a logical isolation inside a Kubernetes cluster used to separate resources.

2. Why use Namespaces?
Answer:
Isolation
Team separation
Environment separation
RBAC

3. Default namespaces in Kubernetes?
Answer:
default
kube-system
kube-public
kube-node-lease

4. What is kube-system?
Answer:
Contains Kubernetes internal components like API server, scheduler, DNS.

5. Difference between Pod and Deployment?
Pod	Deployment
Manual	Managed
No healing	Self-healing
No scaling	Scaling

6. What is Deployment?
Answer:
Deployment manages Pods and ensures desired replicas are running.

7. What is ReplicaSet?
Answer:
ReplicaSet maintains the desired number of Pods.

8. Relationship between Deployment and ReplicaSet?
Answer:
Deployment creates and manages ReplicaSets.

9. What happens when a Pod is deleted from Deployment?
Answer:
Deployment recreates it automatically.

10. What is scaling?
Answer:
Increasing or decreasing pod count.

11. Difference between Scale Up & Scale Down?
Answer:
Scale up increases pods, scale down reduces pods.

12. Imperative vs Declarative scaling?
# Imperative:
kubectl scale

# Declarative:
Edit YAML + apply.

13. What is Rolling Update?
Answer:
Updating application gradually without downtime.

14. Why Rolling Update is important?
Answer:
Ensures zero downtime.

15. What is Rollback?
Answer:
Restoring previous stable version.

16. How to check rollout history?
kubectl rollout history deployment/nginx

17. What are labels in Deployment?
Answer:
Used to identify Pods.

18. What happens if selector and labels mismatch?
Answer:
Deployment will not manage Pods.

19. What command shows ReplicaSets?
kubectl get rs -n dev

20. How does Kubernetes achieve self-healing?
Answer:
Controller checks desired state and recreates failed Pods.

# Senior-Level Interview Questions
1. Why not run Pods directly in production?
Answer:
Pods are ephemeral. If deleted, they don’t recover automatically.

2. Why Deployment instead of ReplicaSet directly?
Answer:
Deployment provides:
rolling update
rollback
easier management

3. What happens internally during rolling update?
Answer:
New ReplicaSet created
New Pods started
Old Pods gradually removed

4. How Deployment ensures zero downtime?
Answer:
New pods become healthy before old pods terminate.

5. What is desired state in Kubernetes?
Answer:
The state defined in YAML that Kubernetes continuously tries to maintain.

6. What happens if node crashes?
Answer:
Pods recreated on another node (multi-node cluster).

7. Explain READY, AVAILABLE, UP-TO-DATE
READY: Running pods
UP-TO-DATE: Latest spec pods
AVAILABLE: Healthy pods

8. Why labels are important?
Answer:
For pod selection and service communication.

9. How Deployment rollback works?
Answer:
Kubernetes stores revision history and restores previous ReplicaSet.

10. Difference between recreate and rolling strategy?
Rolling:
No downtime

Recreate:
Old app stopped first, downtime possible

# Senior Interview Questions (Task 5/6)

Q1. Why did pod show Pending?
Answer:
Scheduler assignment, image pulling, or container initialization.

Q2. Why does Kubernetes terminate pods gracefully?
Answer:
To avoid request failure and data loss.

Q3. What is the difference between scaling and rollout?
Scaling: Pod count changes
Rollout: App version changes

Q4. How does Kubernetes ensure zero downtime?
Answer:
New pod becomes healthy before old pod removal.

Q5. What happens internally during rollout?
Answer:
New ReplicaSet created, old gradually removed.

Q6. Why rollback is useful?
Answer:
Fast recovery from failed deployment.

Q7. Difference between recreate and rolling update?
Rolling: No downtime
Recreate: Downtime possible