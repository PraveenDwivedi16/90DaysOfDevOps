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