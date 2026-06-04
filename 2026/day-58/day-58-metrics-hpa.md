# Day-58 - Metrics Server and Horizontal Pod Autoscaler (HPA)

## Objective

Today we learned:

* What Metrics Server is
* Why HPA needs Metrics Server
* How kubectl top works
* How HPA calculates replicas
* Difference between autoscaling/v1 and autoscaling/v2
* How to generate load
* How HPA automatically scales Pods
* Production best practices

---

# What is Metrics Server?

Metrics Server is a lightweight Kubernetes component that collects CPU and Memory usage from nodes and pods.

Without Metrics Server:

```bash
kubectl top nodes
kubectl top pods
```

will not work.

HPA also cannot work without Metrics Server.

---

# Metrics Flow

```text
Pod
 ↓
Kubelet
 ↓
Metrics Server
 ↓
Metrics API
 ↓
kubectl top
 ↓
HPA
```

---

# Why HPA Needs Metrics Server?

HPA makes scaling decisions based on CPU or Memory usage.

Example:

```text
CPU Usage = 80%
Target = 50%
```

HPA will increase replicas.

Metrics Server provides this usage information.

---

# Difference Between Requests, Limits and Actual Usage

Example:

```yaml
resources:
  requests:
    cpu: 200m

  limits:
    cpu: 500m
```

Meaning:

```text
Request = Minimum reserved CPU

Limit = Maximum allowed CPU
```

Current usage:

```bash
kubectl top pod
```

Output:

```text
CPU = 100m
```

Meaning:

```text
Reserved = 200m
Allowed = 500m
Currently Using = 100m
```

---

# HPA CPU Utilization Calculation

Formula:

```text
CPU Utilization =
(Current CPU Usage / CPU Request)
× 100
```

Example:

```text
Current CPU = 100m

Request CPU = 200m
```

Calculation:

```text
100 / 200 × 100

= 50%
```

HPA sees:

```text
50% Utilization
```

---

# HPA Replica Calculation Formula

Formula:

```text
desiredReplicas =
ceil(
currentReplicas *
(currentUsage / targetUsage)
)
```

Example:

```text
Current Replicas = 2

Current CPU = 80%

Target CPU = 50%
```

Calculation:

```text
2 × (80 / 50)

= 3.2

ceil(3.2)

= 4
```

Result:

```text
HPA creates 4 replicas
```

---

# Task-1 Install Metrics Server

## Check Metrics Server

```bash
kubectl get pods -n kube-system | grep metrics-server
```

No output means Metrics Server is not installed.

---

## Install Metrics Server

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

---

## Kind Cluster TLS Issue

Error:

```text
tls: failed to verify certificate
```

Fix:

Edit Metrics Server deployment:

```bash
kubectl edit deployment metrics-server -n kube-system
```

Add:

```yaml
- --kubelet-insecure-tls
```

Lab environments only.

Never use in production.

---

## Verify Metrics Server

```bash
kubectl top nodes
```

Output:

```text
NAME                            CPU    MEMORY
control-plane                   305m   955Mi
worker                          91m    216Mi
worker2                         62m    ...
```

Metrics Server working successfully.

---

# Task-2 Explore kubectl top

## Commands

```bash
kubectl top nodes
```

```bash
kubectl top pods -A
```

```bash
kubectl top pods -A --sort-by=cpu
```

---

## Highest CPU Pod

Output:

```text
kube-apiserver
26m CPU
```

Highest CPU consuming Pod.

---

## kubectl top Shows

```text
Actual CPU Usage
Actual Memory Usage
```

---

## kubectl describe Shows

```text
Requests
Limits
Configuration
```

Not actual usage.

---

# Task-3 Create Deployment with CPU Requests

## Deployment YAML

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: php-apache

spec:
  replicas: 1

  selector:
    matchLabels:
      app: php-apache

  template:
    metadata:
      labels:
        app: php-apache

    spec:
      containers:
      - name: php-apache
        image: registry.k8s.io/hpa-example

        ports:
        - containerPort: 80

        resources:
          requests:
            cpu: 200m
```

---

## YAML Explanation

### replicas

```yaml
replicas: 1
```

Create one Pod.

---

### selector

```yaml
selector:
```

Connects Deployment with Pods.

---

### labels

```yaml
labels:
```

Identity of Pod.

---

### image

```yaml
image: registry.k8s.io/hpa-example
```

CPU intensive demo application.

---

### requests

```yaml
cpu: 200m
```

Required for HPA.

Without requests:

```text
TARGETS = <unknown>
```

---

## Apply Deployment

```bash
kubectl apply -f php-apache.yaml
```

---

## Verify

```bash
kubectl get pods
```

---

# Service Creation

## Create Service

```bash
kubectl expose deployment php-apache --port=80
```

---

## Service YAML

```yaml
apiVersion: v1
kind: Service

metadata:
  name: php-apache

spec:
  selector:
    app: php-apache

  ports:
  - port: 80
    targetPort: 80
```

---

## Service Explanation

### selector

Select Pods with:

```yaml
app: php-apache
```

---

### port

Service Port.

---

### targetPort

Container Port.

---

# Task-4 Create HPA Imperatively

## Command

```bash
kubectl autoscale deployment php-apache \
--cpu-percent=50 \
--min=1 \
--max=10
```

---

## Explanation

### cpu-percent=50

Target CPU utilization.

---

### min=1

Never below 1 Pod.

---

### max=10

Never above 10 Pods.

---

## Verify

```bash
kubectl get hpa
```

Output:

```text
cpu: 0%/50%
```

Meaning:

```text
Current CPU = 0%
Target CPU = 50%
```

---

# Task-5 Generate Load

## Create Load Generator

```bash
kubectl run load-generator \
--image=busybox:1.36 \
--restart=Never \
-- /bin/sh -c "while true; do wget -q -O- http://php-apache; done"
```

---

## Watch HPA

```bash
kubectl get hpa php-apache --watch
```

Output:

```text
cpu: 213%/50%
```

Meaning:

```text
Current CPU = 213%
Target CPU = 50%
```

---

## Scaling Result

Before:

```text
1 Pod
```

After:

```text
8 Pods
```

HPA automatically scaled Pods.

---

## Verify

```text
1 Pod → 8 Pods
```

Successful autoscaling.

---

# Task-6 Create HPA Using YAML

## Delete Old HPA

```bash
kubectl delete hpa php-apache
```

---

## HPA YAML (autoscaling/v2)

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler

metadata:
  name: php-apache

spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: php-apache

  minReplicas: 1
  maxReplicas: 10

  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50

  behavior:
    scaleUp:
      stabilizationWindowSeconds: 0

    scaleDown:
      stabilizationWindowSeconds: 300
```

---

# YAML Explanation

## scaleTargetRef

Tells HPA which Deployment to scale.

```yaml
name: php-apache
```

---

## averageUtilization

```yaml
averageUtilization: 50
```

Target CPU = 50%.

---

## scaleUp

```yaml
scaleUp:
```

Controls scaling increase.

---

## stabilizationWindowSeconds: 0

Scale immediately.

---

## scaleDown

```yaml
scaleDown:
```

Controls scale down.

---

## stabilizationWindowSeconds: 300

Wait 5 minutes before reducing replicas.

Prevents thrashing.

---

# What is Thrashing?

Example:

```text
8 Pods
↓
1 Pod
↓
8 Pods
↓
1 Pod
```

Repeated scaling.

This wastes resources.

Stabilization window prevents this.

---

# autoscaling/v1 vs autoscaling/v2

| Feature          | v1      | v2  |
| ---------------- | ------- | --- |
| CPU Metrics      | Yes     | Yes |
| Memory Metrics   | No      | Yes |
| Custom Metrics   | No      | Yes |
| External Metrics | No      | Yes |
| Behavior Rules   | No      | Yes |
| Policies         | No      | Yes |
| Production Ready | Limited | Yes |

---

# Task-7 Cleanup

Delete HPA:

```bash
kubectl delete hpa php-apache
```

Delete Service:

```bash
kubectl delete service php-apache
```

Delete Deployment:

```bash
kubectl delete deployment php-apache
```

Delete Load Generator:

```bash
kubectl delete pod load-generator
```

Leave Metrics Server running.

---

# Production Notes

## Use autoscaling/v2

Production clusters should use:

```text
autoscaling/v2
```

because it supports advanced scaling.

---

## Always Configure Requests

Example:

```yaml
requests:
  cpu: 200m
```

Without requests:

```text
HPA will not work.
```

---

## Avoid Using Insecure TLS

Do not use:

```yaml
--kubelet-insecure-tls
```

in production.

Use proper certificates.

---

## Monitor Scale Events

Always check:

```bash
kubectl describe hpa
```

to understand scaling decisions.

---

## Set Reasonable Min and Max Replicas

Example:

```yaml
minReplicas: 2
maxReplicas: 20
```

Avoid extremely high values.

---

## Use Resource Limits

Protect cluster resources.

Example:

```yaml
limits:
  cpu: 500m
```

---

# Common Mistakes

## Mistake 1

No CPU requests.

Result:

```text
TARGETS = <unknown>
```

---

## Mistake 2

Metrics Server not installed.

Result:

```text
Metrics API not available
```

---

## Mistake 3

Using CPU limits instead of requests.

HPA uses:

```text
Requests
```

not limits.

---

## Mistake 4

Using autoscaling/v1 in advanced production environments.

---

## Mistake 5

Very high maxReplicas.

Can waste resources.

---

## Mistake 6

No stabilization window.

May cause thrashing.

---

## Mistake 7

Forgetting to expose Service.

Load generator cannot reach Pods.

---

## Mistake 8

Not checking HPA events.

Use:

```bash
kubectl describe hpa
```

---

## Mistake 9

Assuming kubectl top shows requests.

It shows actual usage.

---

## Mistake 10

Deleting HPA and expecting Pods to disappear.

Pods belong to Deployment.

---

# Architecture Diagram

```text
Metrics Server
       ↓
Metrics API
       ↓
HPA
       ↓
Deployment
       ↓
ReplicaSet
       ↓
Pods
```

---

# 25+ Interview Questions and Answers

## 1. What is Metrics Server?

Collects CPU and Memory metrics from nodes and pods.

---

## 2. Why is Metrics Server required?

HPA needs usage metrics.

---

## 3. What command shows node usage?

```bash
kubectl top nodes
```

---

## 4. What command shows pod usage?

```bash
kubectl top pods
```

---

## 5. Does kubectl top show requests?

No.

It shows actual usage.

---

## 6. What is HPA?

Horizontal Pod Autoscaler.

---

## 7. What does HPA scale?

Pods.

---

## 8. What metrics can HPA use?

CPU, Memory, Custom Metrics.

---

## 9. What is the most common HPA mistake?

Missing CPU requests.

---

## 10. What happens without requests?

TARGETS shows unknown.

---

## 11. What is averageUtilization?

Target utilization percentage.

---

## 12. What is minReplicas?

Minimum number of Pods.

---

## 13. What is maxReplicas?

Maximum number of Pods.

---

## 14. What is scaleTargetRef?

Target resource to scale.

---

## 15. What is autoscaling/v2?

Advanced HPA API.

---

## 16. Difference between v1 and v2?

v2 supports behavior and custom metrics.

---

## 17. What is stabilizationWindowSeconds?

Waiting period before scaling action.

---

## 18. Why use scale down stabilization?

Avoid thrashing.

---

## 19. What is thrashing?

Continuous scale up and scale down.

---

## 20. What does cpu: 50% mean?

Target CPU utilization.

---

## 21. What does cpu: 213%/50% mean?

Current CPU is 213%, target is 50%.

---

## 22. Can HPA work without Metrics Server?

No.

---

## 23. What owns Pods?

ReplicaSet.

---

## 24. What owns ReplicaSet?

Deployment.

---

## 25. Does deleting HPA delete Pods?

No.

---

## 26. How often are metrics collected?

Approximately every 15 seconds.

---

## 27. What is 1000m CPU?

1 CPU core.

---

## 28. What is 500m CPU?

0.5 CPU core.

---

## 29. What is the HPA replica formula?

```text
desiredReplicas =
ceil(
currentReplicas *
(currentUsage / targetUsage)
)
```

---

## 30. Which command explains HPA decisions?
kubectl describe hpa

Scenario 1
Q: HPA is created but TARGETS shows <unknown>. What will you check?
Answer:

I will check:

Metrics Server is running or not
kubectl get pods -n kube-system
Metrics API is working or not
kubectl top nodes
kubectl top pods
CPU requests are configured or not
kubectl describe pod <pod-name>

Most common reason:

CPU requests are missing.
Scenario 2
Q: Metrics Server is running but kubectl top pods gives "Metrics API not available".
Answer:

I will check Metrics Server logs.

kubectl logs -n kube-system deployment/metrics-server

Common reasons:

TLS certificate issue
Metrics Server not ready
Network issue

In Kind cluster, usually:

--kubelet-insecure-tls

is required.

Scenario 3
Q: CPU utilization is 90% but HPA is not scaling Pods.
Answer:

I will check:

kubectl describe hpa

Possible reasons:

HPA reached maxReplicas
Metrics unavailable
Wrong target metric
Deployment not linked properly
Scenario 4
Q: HPA shows 200% CPU utilization. What does it mean?
Answer:

CPU usage is double the requested CPU.

Example:

requests:
  cpu: 200m

Current usage:

400m

Calculation:

400 / 200 × 100

= 200%
Scenario 5
Q: HPA is scaling Pods but users still report slowness.
Answer:

I will check:

CPU bottleneck?
Memory bottleneck?
Database bottleneck?
Network bottleneck?

HPA only adds Pods.

It cannot fix:

Slow Database
Slow Storage
Slow External APIs
Scenario 6
Q: Pods are restarting continuously and HPA is also scaling.
Answer:

I will check:

kubectl get pods
kubectl describe pod
kubectl logs

Reason may be:

CrashLoopBackOff
OOMKilled
Application Error

HPA cannot fix application crashes.

Scenario 7
Q: HPA was working yesterday but today TARGETS is unknown.
Answer:

Check:

kubectl top pods

If it fails:

Metrics Server issue

Then check:

kubectl get pods -n kube-system

and

kubectl logs -n kube-system deployment/metrics-server
Scenario 8
Q: After deleting HPA, Pods are still running. Why?
Answer:

HPA only manages replica count.

Pods are owned by:

Deployment
 ↓
ReplicaSet
 ↓
Pods

Deleting HPA does not delete Deployment or Pods.

Scenario 9
Q: HPA scaled Deployment from 2 Pods to 10 Pods in a few minutes. Is it normal?
Answer:

Yes.

If CPU remains above target:

Example:

Current CPU = 250%
Target CPU = 50%

HPA will keep increasing replicas until utilization reaches target.

Scenario 10
Q: HPA is not scaling below 10 Pods even though CPU is only 20%.
Answer:

Check:

kubectl describe hpa

Look for:

stabilizationWindowSeconds

Example:

scaleDown:
  stabilizationWindowSeconds: 300

HPA waits 5 minutes before scaling down.

Scenario 11
Q: HPA is created successfully but replicas remain 1.
Answer:

Current CPU may be below target.

Example:

Current = 20%
Target = 50%

No scale-up required.

Scenario 12
Q: HPA says cpu: 50%/50% but replicas are not increasing.
Answer:

Current utilization already equals target.

Cluster is healthy.

No scaling action required.

Scenario 13
Q: HPA works in dev cluster but not in production cluster.
Answer:

I will check:

Metrics Server
RBAC permissions
Network policies
Resource requests
HPA configuration
Scenario 14
Q: Pods are using 800m CPU but HPA shows only 40%.
Answer:

Check CPU request.

Example:

requests:
  cpu: 2000m

Calculation:

800 / 2000 × 100

= 40%

HPA calculates percentage using requests.

Scenario 15
Q: What would happen if maxReplicas is set to 3 and traffic becomes huge?
Answer:

HPA cannot scale beyond:

maxReplicas: 3

Result:

CPU remains high
Application may become slow
Scenario 16
Q: During Black Friday sale, traffic suddenly increases. How does HPA help?
Answer:

HPA detects high CPU.

Example:

CPU > 50%

Automatically:

2 Pods
↓
4 Pods
↓
8 Pods
↓
16 Pods

depending on maxReplicas.

Scenario 17
Q: Why should CPU requests always be defined when using HPA?
Answer:

HPA calculates:

Usage / Request × 100

Without requests:

TARGETS = <unknown>

and scaling will not work.

Scenario 18
Q: Your manager says application cost increased suddenly. What HPA-related things will you check?
Answer:

Check:

kubectl describe hpa

Review:

minReplicas
maxReplicas
CPU target
Scale events

Maybe:

CPU target too low
maxReplicas too high

causing unnecessary scaling.

Scenario 19
Q: HPA scaled to maxReplicas but CPU is still 90%. What does it indicate?
Answer:

Application needs more resources.

Possible solutions:

Increase maxReplicas
Increase node capacity
Optimize application
Use Cluster Autoscaler

Scenario 20 (Senior Level)
Q: Difference between HPA and Cluster Autoscaler?
Answer:

HPA:

Scales Pods

Example:

2 Pods → 10 Pods

Cluster Autoscaler:

Scales Nodes

Example:

3 Nodes → 6 Nodes

Best practice:

HPA + Cluster Autoscaler together

for production environments.

Golden Interview Tip

Remember this line:

Metrics Server provides metrics.
HPA scales Pods.
Cluster Autoscaler scales Nodes.
