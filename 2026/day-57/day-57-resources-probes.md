# Day-57 Resources, Limits and Probes

# Introduction

In previous days, we created Pods and Deployments.

But Kubernetes still does not know:

* How much CPU the application needs
* How much memory the application needs
* Whether the application is healthy
* Whether the application is ready to receive traffic

This is why Kubernetes provides:

1. Resource Requests
2. Resource Limits
3. Liveness Probes
4. Readiness Probes
5. Startup Probes

These features are heavily used in production Kubernetes clusters.

---

# Learning Objectives

After completing this lab you should understand:

* Requests vs Limits
* CPU and Memory behavior
* OOMKilled
* QoS Classes
* Pod Scheduling
* Liveness Probe
* Readiness Probe
* Startup Probe
* Troubleshooting unhealthy Pods

---

# What Are Resources?

Resources tell Kubernetes how much CPU and Memory a Pod needs.

Without resources Kubernetes cannot make good scheduling decisions.

Example:

A node has:

```text
CPU = 4 Core
Memory = 8GB
```

A Pod requires:

```text
CPU = 100m
Memory = 128Mi
```

Scheduler checks node capacity and places the Pod.

---

# Real World Example

Think about a hotel.

Node = Hotel

Pod = Customer

Request = Room booking

Limit = Maximum room usage

Customer says:

```text
I need at least one room.
```

This is Request.

Hotel says:

```text
You cannot use more than 2 rooms.
```

This is Limit.

---

# Requests vs Limits

This is one of the most important Kubernetes interview topics.

---

# Requests

Requests are the minimum resources guaranteed to the Pod.

Example:

```yaml
requests:
  cpu: 100m
  memory: 128Mi
```

Meaning:

```text
Minimum CPU = 0.1 Core
Minimum Memory = 128Mi
```

Scheduler uses Requests.

---

# Why Requests Are Important

Scheduler must know:

```text
Can this Pod fit on this node?
```

Scheduler checks:

```text
Requests
```

NOT:

```text
Limits
```

Senior Interview Question:

Does Scheduler use Requests or Limits?

Answer:

```text
Scheduler uses Requests.
```

---

# Limits

Limits are maximum resources allowed.

Example:

```yaml
limits:
  cpu: 250m
  memory: 256Mi
```

Meaning:

```text
CPU cannot exceed 250m
Memory cannot exceed 256Mi
```

Kubelet enforces Limits.

---

# Easy Example

Application:

```text
Request = 100m CPU
Limit = 250m CPU
```

Meaning:

```text
Need at least 100m
Can use up to 250m
```

---

# CPU Units

CPU is measured in cores.

Examples:

```text
1000m = 1 Core
500m = 0.5 Core
250m = 0.25 Core
100m = 0.1 Core
```

Interview Question:

What is 100m CPU?

Answer:

```text
0.1 CPU Core
```

---

# Memory Units

Memory is measured using:

```text
Mi = Mebibyte
Gi = Gibibyte
```

Examples:

```text
128Mi
256Mi
1Gi
2Gi
```

Approximation:

```text
1024Mi = 1Gi
```

---

# CPU vs Memory

This is a very important interview topic.

CPU and Memory behave differently.

---

# CPU Over Limit

Example:

```yaml
limits:
  cpu: 250m
```

Application tries to use:

```text
500m CPU
```

Result:

```text
Container survives
CPU is throttled
```

Container is NOT killed.

---

# Memory Over Limit

Example:

```yaml
limits:
  memory: 100Mi
```

Application tries:

```text
200Mi Memory
```

Result:

```text
Container is killed
```

Reason:

```text
OOMKilled
```

---

# Why CPU Is Not Killed

CPU is compressible.

Kubernetes can reduce CPU usage.

Example:

```text
Requested = 100m
Using = 500m
Allowed = 250m
```

Kubernetes slows the container.

---

# Why Memory Is Killed

Memory is not compressible.

Example:

```text
Limit = 100Mi
Used = 200Mi
```

Node cannot magically create memory.

Linux Kernel kills the process.

---

# OOMKilled

OOM means:

```text
Out Of Memory
```

This happens when a container exceeds its memory limit.

---

# Real World Example

Java Application has memory leak.

Memory usage:

```text
100Mi
150Mi
200Mi
250Mi
300Mi
```

Limit:

```text
256Mi
```

Result:

```text
OOMKilled
```

---

# Exit Code 137

OOMKilled usually shows:

```text
Exit Code: 137
```

Why?

Formula:

```text
128 + SIGKILL(9)
```

Result:

```text
137
```

Interview Question:

What exit code usually indicates OOMKilled?

Answer:

```text
137
```

---

# QoS Classes

QoS = Quality of Service

Kubernetes assigns every Pod a QoS class.

There are three classes.

---

# 1. Guaranteed

Requests and Limits are equal.

Example:

```yaml
resources:
  requests:
    cpu: 100m
    memory: 128Mi

  limits:
    cpu: 100m
    memory: 128Mi
```

Result:

```text
QoS = Guaranteed
```

Highest priority.

---

# Easy Example

Employee reserved exactly:

```text
2GB RAM
```

and

```text
Maximum = 2GB RAM
```

Guaranteed.

---

# 2. Burstable

Requests are lower than Limits.

Example:

```yaml
resources:
  requests:
    cpu: 100m
    memory: 128Mi

  limits:
    cpu: 250m
    memory: 256Mi
```

Result:

```text
QoS = Burstable
```

Most common in production.

---

# Easy Example

Need:

```text
1 Room
```

Can use:

```text
2 Rooms
```

Burstable.

---

# 3. BestEffort

No Requests.

No Limits.

Example:

```yaml
resources: {}
```

Result:

```text
QoS = BestEffort
```

Lowest priority.

---

# QoS Interview Question

Which QoS class is highest priority?

Answer:

```text
Guaranteed
```

---

# QoS Interview Question

Which QoS class is lowest priority?

Answer:

```text
BestEffort
```

---

# Task-1 Resource Requests and Limits

Goal:

Create a Pod with:

```yaml
requests:
  cpu: 100m
  memory: 128Mi

limits:
  cpu: 250m
  memory: 256Mi
```

---

# YAML Used

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: resource-demo

spec:
  containers:
  - name: nginx
    image: nginx

    resources:
      requests:
        cpu: "100m"
        memory: "128Mi"

      limits:
        cpu: "250m"
        memory: "256Mi"
```

---

# YAML Explanation

apiVersion

```yaml
apiVersion: v1
```

Defines API version.

---

kind

```yaml
kind: Pod
```

Create a Pod object.

---

metadata

```yaml
metadata:
  name: resource-demo
```

Pod name.

---

image

```yaml
image: nginx
```

Nginx container image.

---

resources

```yaml
resources:
```

Resource configuration section.

---

requests.cpu

```yaml
cpu: 100m
```

Minimum CPU guarantee.

---

requests.memory

```yaml
memory: 128Mi
```

Minimum memory guarantee.

---

limits.cpu

```yaml
cpu: 250m
```

Maximum CPU allowed.

---

limits.memory

```yaml
memory: 256Mi
```

Maximum memory allowed.

---

# Commands

Create Pod

```bash
kubectl apply -f resource-demo.yaml
```

Check Pod

```bash
kubectl get pods
```

Describe Pod

```bash
kubectl describe pod resource-demo
```

---

# Verification

Output:

```text
QoS Class: Burstable
```

Why?

Because:

```text
Requests != Limits
```

---

# Task-1 Interview Questions

Question:

What is a Request?

Answer:

Minimum guaranteed resource.

---

Question:

What is a Limit?

Answer:

Maximum allowed resource.

---

Question:

Who uses Requests?

Answer:

Scheduler.

---

Question:

Who enforces Limits?

Answer:

Kubelet.

---

Question:

What QoS class did resource-demo have?

Answer:

Burstable.

---

# Task-2 OOMKilled

Goal:

Create a Pod that exceeds memory limit.

Observe:

```text
OOMKilled
```

---

# YAML Used

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: oom-demo

spec:
  containers:
  - name: stress

    image: polinux/stress

    command:
    - stress

    args:
    - --vm
    - "1"
    - --vm-bytes
    - 200M
    - --vm-hang
    - "1"

    resources:
      limits:
        memory: "100Mi"
```

---

# Why This Pod Fails

Container tries:

```text
200M Memory
```

Allowed:

```text
100Mi Memory
```

Result:

```text
OOMKilled
```

---

# YAML Explanation

Image

```yaml
image: polinux/stress
```

Special image used for stress testing.

---

--vm 1

```text
Create one memory worker.
```

---

--vm-bytes 200M

```text
Allocate 200MB memory.
```

---

--vm-hang 1

```text
Keep allocated memory.
```

---

Memory Limit

```yaml
limits:
  memory: 100Mi
```

Maximum allowed memory.

---

# Commands

Apply

```bash
kubectl apply -f oom-demo.yaml
```

Check

```bash
kubectl get pod oom-demo
```

Describe

```bash
kubectl describe pod oom-demo
```

---

# Expected Output

```text
STATUS: OOMKilled
```

and

```text
Exit Code: 137
```

---

# What Happened Internally

Container:

```text
Needs 200M
```

Limit:

```text
100Mi
```

Linux Kernel:

```text
Kill Process
```

Kubernetes:

```text
Restart Container
```

---

# Real World Example

Application memory leak.

Memory continuously increases.

Limit reached.

Result:

```text
OOMKilled
```

---

# Task-2 Interview Questions

Question:

What does OOM mean?

Answer:

Out Of Memory.

---

Question:

What happens when memory limit is exceeded?

Answer:

Container is killed.

---

Question:

What happens when CPU limit is exceeded?

Answer:

Container is throttled.

Question:
What exit code is usually seen during OOMKilled?

Answer:
137.

Question:
Why is memory killed but CPU throttled?

Answer:
CPU is compressible.
Memory is not compressible.

# Day-57 Resources, Limits and Probes

# Part-2

# Task-3 Pending Pod (Requesting Too Much Resources)

In Task-3 we intentionally created a Pod that requested more resources than available in the cluster.

Goal:

```text id="w0n7xa"
Understand how Kubernetes Scheduler works.
```

---

# Why This Task Is Important

Many beginners think:

```text id="a0e73v"
Pod Created = Pod Running
```

This is wrong.

A Pod can be:

```text id="0sn4xv"
Created
Pending
Not Scheduled
```

for a long time.

---

# Real World Example

Suppose you have:

```text id="j4x3r4"
Node-1 = 4 CPU
Node-2 = 4 CPU
Node-3 = 8 CPU
```

Total:

```text id="z9z4qa"
16 CPU
```

Pod requests:

```yaml id="sq9uzv"
cpu: 100
```

Meaning:

```text id="n96p1f"
100 CPU Core
```

Scheduler says:

```text id="7u2duy"
No node can satisfy this request.
```

Result:

```text id="shzv74"
Pending
```

---

# YAML Used

```yaml id="yjlwm9"
apiVersion: v1
kind: Pod

metadata:
  name: huge-request

spec:
  containers:
  - name: nginx
    image: nginx

    resources:
      requests:
        cpu: "100"
        memory: "128Gi"
```

---

# YAML Explanation

cpu: "100"

Means:

```text id="7jk6eq"
100 CPU Core
```

NOT:

```text id="xy9u0k"
100m
```

Important difference:

```text id="ys2hpo"
100m = 0.1 CPU

100 = 100 CPU
```

---

memory: "128Gi"

Means:

```text id="vxqg3l"
128 Gigabytes RAM
```

Most clusters do not even have this much memory.

---

# Commands Used

Apply:

```bash id="o0mlj4"
kubectl apply -f huge-request.yaml
```

Check:

```bash id="slm4r7"
kubectl get pod huge-request
```

Describe:

```bash id="tvyc92"
kubectl describe pod huge-request
```

---

# Output Observed

```text id="2xw15j"
STATUS = Pending
```

and

```text id="43k8gr"
Node = <none>
```

Very important.

If Node is:

```text id="s9z56p"
<none>
```

then Scheduler failed to find a suitable node.

---

# Scheduler Internals

What happens after:

```bash id="76c6t3"
kubectl apply
```

Flow:

```text id="yzrlh2"
API Server
      ↓
Pod Created
      ↓
Scheduler
      ↓
Check Nodes
      ↓
Check Requests
      ↓
No Suitable Node
      ↓
Pending
```

---

# Understanding FailedScheduling Event

Your event:

```text id="j8tlnr"
0/3 nodes are available
```

Meaning:

```text id="n0u4wg"
Cluster has 3 nodes.
```

But:

```text id="s1c2z4"
0 nodes can run this Pod.
```

---

# Insufficient CPU

Event:

```text id="nm12vd"
2 Insufficient cpu
```

Meaning:

Pod requested:

```text id="r6qv6y"
100 CPU
```

Available:

```text id="kpv4wx"
Much lower
```

Scheduler rejected the Pod.

---

# Insufficient Memory

Event:

```text id="29f8e4"
2 Insufficient memory
```

Meaning:

Requested:

```text id="9ghwcb"
128Gi
```

Available:

```text id="4z0l3x"
Much lower
```

Scheduler rejected the Pod.

---

# Untolerated Taints

Event:

```text id="xyhjlwm"
1 node(s) had untolerated taint
```

Meaning:

One node has a taint.

Pod does not have matching toleration.

Scheduler cannot place Pod there.

---

# Easy Example

Node says:

```text id="2gll4y"
Only Dev Pods Allowed
```

Pod says:

```text id="owkqcv"
I do not have permission.
```

Scheduler rejects the node.

---

# Senior Interview Question

What does Scheduler use?

Answer:

```text id="88mjlwm"
Requests
```

NOT:

```text id="0xjs7r"
Limits
```

Very important.

---

# Senior Interview Question

Can a Pod be created but not scheduled?

Answer:

```text id="vxjq8x"
Yes
```

Example:

```text id="zqg5qz"
Pending Pod
```

---

# Pending vs ContainerCreating

Many interviews ask this.

---

# Pending

Meaning:

```text id="sjhl1k"
Node not assigned.
```

Example:

```text id="z15r83"
Insufficient CPU
Insufficient Memory
```

---

# ContainerCreating

Meaning:

```text id="3n0vhu"
Node assigned.
```

But Kubernetes is still:

```text id="ujz2qs"
Pulling Image
Mounting Volumes
Creating Network
```

---

# Easy Example

Pending:

```text id="nl1r7t"
No hotel room available.
```

ContainerCreating:

```text id="o2trlu"
Room assigned.
Preparing room.
```

---

# Troubleshooting Pending Pods

First command:

```bash id="5pbl7o"
kubectl describe pod <pod-name>
```

Always check:

```text id="y7kmq1"
Events Section
```

---

# Additional Commands

Check Node Capacity:

```bash id="8m91eh"
kubectl describe node <node-name>
```

---

Check Resource Usage:

```bash id="zxur6s"
kubectl top node
```

---

Check Allocatable Resources:

```bash id="ll5ewp"
kubectl describe node
```

Look for:

```text id="56tmnp"
Allocatable
```

section.

---

# Resource Requests Best Practice

Bad:

```yaml id="8lw4rh"
cpu: 100
memory: 128Gi
```

Good:

```yaml id="u6byo0"
cpu: 100m
memory: 128Mi
```

Always use realistic values.

---

# Advanced Concept - Requests Only

Example:

```yaml id="xv1yk5"
resources:
  requests:
    cpu: 100m
    memory: 128Mi
```

No limits.

---

# Benefits

```text id="z7h2sa"
Less chance of OOMKilled
```

---

# Risks

Container may consume:

```text id="k0l3dn"
Unlimited Memory
```

Potential node problems.

---

# Advanced Concept - Limits Only

Example:

```yaml id="3bwdml"
resources:
  limits:
    memory: 256Mi
```

Possible.

But production best practice:

```text id="d0z4n6"
Always define Requests and Limits.
```

---

# Advanced Concept - ResourceQuota

ResourceQuota limits total namespace consumption.

Example:

```text id="pgm0jk"
Namespace can use only:

4 CPU
8Gi Memory
```

Useful in multi-team clusters.

---

# Advanced Concept - LimitRange

LimitRange can automatically apply:

```text id="0y7rq2"
Default Requests
Default Limits
```

to Pods.

---

# Advanced Concept - Eviction

Very important interview topic.

Eviction is NOT OOMKilled.

---

# OOMKilled

Container exceeds:

```text id="u9gc4u"
Its own memory limit
```

---

# Evicted

Node runs out of resources.

Kubernetes removes Pods.

Example:

```text id="4n84eh"
Node memory pressure
```

Result:

```text id="xq97g1"
Pod Evicted
```

---

# OOMKilled vs Evicted

OOMKilled:

```text id="ikjlwm"
Container Problem
```

Evicted:

```text id="yd93gk"
Node Problem
```

---

# Senior Interview Question

Difference between OOMKilled and Evicted?

Answer:

```text id="pbl54e"
OOMKilled = Container exceeded memory limit

Evicted = Node resource pressure
```

---

# Advanced Concept - QoS and Eviction

When node is under pressure:

Kubernetes evicts Pods in order.

---

# First Evicted

```text id="8k41mf"
BestEffort
```

---

# Then

```text id="6eqh7l"
Burstable
```

---

# Last Evicted

```text id="vn3m8z"
Guaranteed
```

Highest protection.

---

# Real World Example

Node Memory Pressure.

Pods:

```text id="31vwfa"
Pod-A = BestEffort

Pod-B = Burstable

Pod-C = Guaranteed
```

Kubernetes removes:

```text id="bjlwm4"
Pod-A first
```

---

# Senior Level Interview Questions

Question:

Does Scheduler use Requests or Limits?

Answer:

Requests.

---

Question:

What causes Pending status?

Answer:

No suitable node found.

---

Question:

What command is used first to troubleshoot Pending Pods?

Answer:

```bash id="0qj0mg"
kubectl describe pod
```

---

Question:

Difference between Pending and ContainerCreating?

Answer:

Pending = No Node

ContainerCreating = Node assigned but setup not finished.

---

Question:
What is ResourceQuota?

Answer:
Namespace level resource control.

Question:
What is LimitRange?

Answer:
Defines default resource requests and limits.

Question:
Difference between OOMKilled and Evicted?

Answer:
OOMKilled = Container limit exceeded
Evicted = Node pressure

Question:
Which QoS class is evicted first?

Answer:
BestEffort.


Question:
Which QoS class gets maximum protection?

Answer:
Guaranteed.

# Day-57 Resources, Limits and Probes

# Part-3

# Kubernetes Probes

Resources help Kubernetes decide:

```text id="6jjlwm"
Where to run a Pod
```

Probes help Kubernetes decide:

```text id="y2w83f"
Is the application healthy?
```

Without probes:

```text id="mj3c0z"
Container may be running
Application may be broken
```

Kubernetes would not know.

---

# Why Probes Are Important

Real production example:

```text id="pk4h7d"
Java Application
```

Container:

```text id="zjlwm1"
Running
```

Application:

```text id="9pv4kl"
Hung
Deadlock
Not responding
```

Without probes:

```text id="76m5qh"
Users face issues
```

Kubernetes does nothing.

---

# Three Types of Probes

Kubernetes provides:

```text id="9gm8qh"
Liveness Probe
Readiness Probe
Startup Probe
```

Each solves a different problem.

---

# Easy Way To Remember

Liveness:

```text id="2vjlwm"
Are you alive?
```

Readiness:

```text id="c4pm1r"
Are you ready?
```

Startup:

```text id="0w83kz"
Have you started?
```

---

# Liveness Probe

Purpose:

```text id="g1y7nv"
Detect dead or stuck applications.
```

If Liveness Probe fails:

```text id="7m6jlwm"
Container Restart
```

---

# Real World Example

Application enters deadlock.

Container:

```text id="i4pn2d"
Running
```

Application:

```text id="xt8jlwm"
Not responding
```

Liveness Probe fails.

Kubernetes:

```text id="ap4w8e"
Kills container
Starts container again
```

---

# Task-4 Liveness Probe

Goal:

Create a file.

Delete file after 30 seconds.

Probe checks file.

When file disappears:

```text id="u1m7zh"
Probe fails
```

---

# YAML Used

```yaml id="3h5jlwm"
apiVersion: v1
kind: Pod

metadata:
  name: liveness-demo

spec:
  containers:
  - name: busybox
    image: busybox

    command:
    - /bin/sh
    - -c
    - |
      touch /tmp/healthy;
      sleep 30;
      rm -f /tmp/healthy;
      sleep 600

    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy

      periodSeconds: 5
      failureThreshold: 3
```

---

# YAML Explanation

Create File

```bash id="91jlwm"
touch /tmp/healthy
```

Probe succeeds.

---

Wait

```bash id="gk4w7z"
sleep 30
```

Application appears healthy.

---

Delete File

```bash id="x5pm1r"
rm -f /tmp/healthy
```

Probe starts failing.

---

Probe Command

```bash id="t2jlwm"
cat /tmp/healthy
```

If file exists:

```text id="8m4pnw"
Exit Code 0
Success
```

---

If file missing:

```text id="k1y83f"
Non-zero Exit Code
Failure
```

---

# Probe Timing

periodSeconds:

```yaml id="3jlwm9"
periodSeconds: 5
```

Meaning:

Probe runs every:

```text id="u4pm7k"
5 seconds
```

---

failureThreshold:

```yaml id="w6y83m"
failureThreshold: 3
```

Meaning:

```text id="g9jlwm"
Fail #1
Fail #2
Fail #3
Restart
```

---

# Timeline

```text id="n2pm4z"
0 sec    File exists

5 sec    Success

10 sec   Success

15 sec   Success

20 sec   Success

25 sec   Success

30 sec   File removed

35 sec   Fail #1

40 sec   Fail #2

45 sec   Fail #3

Restart
```

---

# Output Observed

```text id="z7y83k"
Restart Count: 1
```

And:

```text id="4jlwm8"
Liveness probe failed
```

And:

```text id="b1pm5r"
Container busybox failed liveness probe
```

---

# Important Interview Question

What happens when Liveness Probe fails?

Answer:

```text id="m8y83v"
Container Restart
```

---

# Important Interview Question

Does Liveness remove Pod from Service?

Answer:

```text id="k3jlwm"
No
```

Its job is restart.

---

# Readiness Probe

Purpose:

```text id="v4pm7w"
Control traffic
```

Readiness asks:

```text id="2y83fn"
Can this application receive requests?
```

---

# If Readiness Fails

Kubernetes:

```text id="f7jlwm"
Removes Pod from endpoints
```

But:

```text id="u1pm8r"
Does NOT restart container
```

Very important.

---

# Real World Example

Application:

```text id="9y83mk"
Running
```

Database:

```text id="p4jlwm"
Down
```

Application cannot serve traffic.

Readiness:

```text id="w7pm2d"
Fail
```

Kubernetes:

```text id="r3y83v"
Stop sending traffic
```

---

# Task-5 Readiness Probe

Goal:

Use nginx.

Probe homepage.

Delete homepage.

Observe readiness failure.

---

# YAML Used

```yaml id="x6jlwm"
apiVersion: v1
kind: Pod

metadata:
  name: readiness-demo
  labels:
    app: nginx

spec:
  containers:
  - name: nginx
    image: nginx

    readinessProbe:
      httpGet:
        path: /
        port: 80

      periodSeconds: 5
      failureThreshold: 3
```

---

# Why Labels Were Needed

Service uses selectors.

Example:

Pod:

```yaml id="k8pm4w"
labels:
  app: nginx
```

Service:

```yaml id="y2jlwm"
selector:
  app: nginx
```

Labels connect Service and Pod.

---

# Service Created

```bash id="m5y83f"
kubectl expose pod readiness-demo --port=80 --name=readiness-svc
```

---

# Endpoint Verification

Output:

```text id="u8jlwm"
10.x.x.x:80
```

Meaning:

```text id="d4pm7z"
Pod is Ready
Traffic Allowed
```

---

# Breaking Readiness

Command:

```bash id="j1y83m"
kubectl exec readiness-demo -- rm /usr/share/nginx/html/index.html
```

Homepage removed.

---

# Probe Result

Observed:

```text id="p7jlwm"
HTTP probe failed with statuscode: 403
```

or sometimes:

```text id="t3pm8w"
404
```

Both mean:

```text id="v6y83k"
Probe Failed
```

---

# Endpoint Removed

Observed:

```text id="q2jlwm"
ENDPOINTS = Empty
```

Meaning:

```text id="n5pm4r"
Service stopped sending traffic
```

---

# Most Important Observation

Container:

```text id="x8y83v"
Still Running
```

Restart Count:

```text id="m4jlwm"
0
```

Meaning:

```text id="d7pm2z"
Readiness does NOT restart containers
```

---

# Liveness vs Readiness

Liveness Failure:

```text id="u3y83f"
Restart
```

Readiness Failure:

```text id="p9jlwm"
Remove From Endpoints
```

---

# Startup Probe

Purpose:

```text id="k5pm8w"
Give slow applications time to start.
```

---

# Problem Without Startup Probe

Spring Boot Example:

Startup Time:

```text id="w1y83m"
40 seconds
```

Liveness starts:

```text id="f6jlwm"
5 seconds
```

Application not ready yet.

Probe fails.

Container restarts.

Application never starts.

---

# Startup Probe Solution

Startup Probe says:

```text id="z2pm4r"
Wait for startup
```

Until Startup succeeds:

```text id="t8y83v"
Liveness Disabled
Readiness Disabled
```

---

# Task-6 Startup Probe

Goal:

Application needs:

```text id="j4jlwm"
20 seconds
```

to start.

Startup Probe gives:

```text id="r7pm2z"
60 seconds budget
```

---

# YAML Used

```yaml id="m1y83f"
apiVersion: v1
kind: Pod

metadata:
  name: startup-demo

spec:
  containers:
  - name: busybox
    image: busybox

    command:
    - /bin/sh
    - -c
    - |
      sleep 20
      touch /tmp/started
      sleep 600

    startupProbe:
      exec:
        command:
        - cat
        - /tmp/started

      periodSeconds: 5
      failureThreshold: 12

    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/started

      periodSeconds: 5
```

---

# Startup Probe Calculation

periodSeconds:

```yaml id="v9jlwm"
5
```

failureThreshold:

```yaml id="g3pm8w"
12
```

Calculation:

```text id="y6y83m"
12 × 5
=
60 seconds
```

Application needs:

```text id="c2jlwm"
20 seconds
```

Result:

```text id="n8pm4z"
Startup Success
```

---

# Output Observed

Initially:

```text id="r5y83v"
0/1 Running
```

Later:

```text id="h1jlwm"
1/1 Running
```

Reason:

```text id="s4pm2r"
Application completed startup
```

---

# Startup Probe Events

Observed:

```text id="w7y83f"
Startup probe failed
```

Initially expected.

File did not exist yet.

After:

```bash id="k9jlwm"
touch /tmp/started
```

Probe passed.

---

# Interview Question

What happens if failureThreshold becomes 2?

Calculation:

```text id="p3pm8z"
2 × 5
=
10 seconds
```

Application needs:

```text id="u6y83m"
20 seconds
```

Result:

```text id="e2jlwm"
Container Restart Loop
```

Application never starts.

---

# Probe Comparison Table

| Probe     | Purpose          | Failure Result  |
| --------- | ---------------- | --------------- |
| Liveness  | Is app alive?    | Restart         |
| Readiness | Is app ready?    | Remove endpoint |
| Startup   | Has app started? | Restart         |

---

# Easy Memory Trick

Liveness:

```text id="b8pm4r"
Alive?
```

Readiness:

```text id="j5y83v"
Ready?
```

Startup:

```text id="q1jlwm"
Started?
```

---

# Probe Types

Kubernetes supports:

---

# Exec Probe

Example:

```yaml id="m7pm2z"
exec:
  command:
  - cat
  - /tmp/healthy
```

Runs command inside container.

---

# HTTP Probe

Example:

```yaml id="t4y83f"
httpGet:
  path: /
  port: 80
```

Sends HTTP request.

# TCP Probe
Example:
yaml 
id="z9jlwm"
tcpSocket:
  port: 3306


Checks whether TCP port is open.

# Senior Interview Questions

Question:
What happens when Readiness fails?

Answer:
Pod removed from endpoints.


Question:
What happens when Liveness fails?

Answer:
Container restart.


Question:
What happens when Startup fails?

Answer:
Container restart.


Question:
Does Readiness restart Pods?

Answer:
No.


Question:
Can a Pod be Running but Not Ready?

Answer:
Yes.
Example:
text id="p6pm4w"
READY 0/1
STATUS Running


Question:
Why was Startup Probe introduced?

Answer:
To support slow starting applications.


Question:
When are Liveness and Readiness disabled?

Answer:
While Startup Probe is running.

# Day-57 Resources, Limits and Probes

# Part-4 (Final Part)

# Complete Command Reference

## Resource Requests and Limits

Create Pod

```bash
kubectl apply -f resource-demo.yaml
```

Check Pod

```bash
kubectl get pods
```

Describe Pod

```bash
kubectl describe pod resource-demo
```

Check QoS Class

```bash
kubectl describe pod resource-demo
```

Look for:

```text
QoS Class: Burstable
```

---

# OOMKilled Commands

Create Pod

```bash
kubectl apply -f oom-demo.yaml
```

Watch Pod

```bash
kubectl get pod oom-demo -w
```

Describe Pod

```bash
kubectl describe pod oom-demo
```

Look for:

```text
Reason: OOMKilled
Exit Code: 137
```

---

# Pending Pod Commands

Create Pod

```bash
kubectl apply -f huge-request.yaml
```

Check Status

```bash
kubectl get pod huge-request
```

Describe Pod

```bash
kubectl describe pod huge-request
```

Look for:

```text
FailedScheduling
Insufficient CPU
Insufficient Memory
```

---

# Liveness Probe Commands

Apply

```bash
kubectl apply -f liveness-demo.yaml
```

Watch

```bash
kubectl get pod liveness-demo -w
```

Describe

```bash
kubectl describe pod liveness-demo
```

Check:

```text
Restart Count
Liveness Probe Failed
```

---

# Readiness Probe Commands

Apply

```bash
kubectl apply -f readiness-demo.yaml
```

Create Service

```bash
kubectl expose pod readiness-demo --port=80 --name=readiness-svc
```

Check Endpoints

```bash
kubectl get endpoints readiness-svc
```

Break Probe

```bash
kubectl exec readiness-demo -- rm /usr/share/nginx/html/index.html
```

Check Again

```bash
kubectl get pod readiness-demo
kubectl get endpoints readiness-svc
```

---

# Startup Probe Commands

Apply

```bash
kubectl apply -f startup-demo.yaml
```

Watch

```bash
kubectl get pod startup-demo -w
```

Describe

```bash
kubectl describe pod startup-demo
```

---

# Cleanup Commands

```bash
kubectl delete pod resource-demo
kubectl delete pod oom-demo
kubectl delete pod huge-request
kubectl delete pod liveness-demo
kubectl delete pod readiness-demo
kubectl delete pod startup-demo

kubectl delete svc readiness-svc
```

---

# Troubleshooting Guide

# Problem 1: Pod Pending

Symptoms:

```text
STATUS = Pending
```

Check:

```bash
kubectl describe pod <pod-name>
```

Common Reasons:

```text
Insufficient CPU
Insufficient Memory
Taints
Node Not Ready
```

---

# Problem 2: OOMKilled

Symptoms:

```text
OOMKilled
Exit Code 137
```

Check:

```bash
kubectl describe pod
kubectl logs
kubectl top pod
```

Solutions:

```text
Increase Memory Limit
Fix Memory Leak
Optimize Application
```

---

# Problem 3: CrashLoopBackOff

Symptoms:

```text
CrashLoopBackOff
```

Check:

```bash
kubectl logs <pod>
kubectl describe pod
```

Common Reasons:

```text
Wrong Command
Application Crash
Probe Failure
```

---

# Problem 4: Readiness Failure

Symptoms:

```text
READY 0/1
```

Check:

```bash
kubectl describe pod
```

Look for:

```text
Readiness Probe Failed
```

---

# Problem 5: Liveness Failure

Symptoms:

```text
Restart Count Increasing
```

Check:

```bash
kubectl describe pod
```

Look for:

```text
Liveness Probe Failed
```

---

# Production Best Practices

# Resource Best Practices

Always define:

```yaml
requests:
  cpu: 100m
  memory: 128Mi

limits:
  cpu: 200m
  memory: 256Mi
```

Do not leave resources empty.

---

# Probe Best Practices

Always use:

```text
Readiness Probe
```

in production.

Reason:

```text
Protect Users
Protect Load Balancer
```

---

# Startup Probe Best Practice

Use Startup Probe for:

```text
Spring Boot
Java Applications
Large Applications
Slow Startup Services
```

---

# Liveness Best Practice

Do not make liveness too aggressive.

Bad:

```yaml
failureThreshold: 1
```

Good:

```yaml
failureThreshold: 3
```

---

# Readiness Best Practice

Readiness should verify:

```text
Application
Database
Dependencies
```

---

# Senior Level Concepts

# cgroups

Linux uses:

```text
cgroups
```

to enforce:

```text
CPU Limits
Memory Limits
```

Kubernetes itself does not directly limit resources.

Linux cgroups do.

---

# Scheduler Uses Requests

Very important.

Scheduler checks:

```text
Requests
```

Scheduler ignores:

```text
Limits
```

during placement.

---

# Why CPU Is Throttled

CPU can be slowed down.

Example:

```text
Limit = 250m
Usage = 500m
```

Result:

```text
Throttle
```

---

# Why Memory Is Killed

Memory cannot be compressed.

Example:

```text
Limit = 100Mi
Usage = 200Mi
```

Result:

```text
OOMKilled
```

---

# EndpointSlice

You observed:

```text
Endpoints is deprecated
```

Modern Kubernetes uses:

```text
EndpointSlice
```

instead of:

```text
Endpoints
```

Reason:

```text
Better Scalability
Better Performance
```

---

# Quick Revision Notes

Requests

```text
Minimum Guaranteed
```

Limits

```text
Maximum Allowed
```

Scheduler Uses

```text
Requests
```

Kubelet Enforces

```text
Limits
```

CPU Over Limit

```text
Throttle
```

Memory Over Limit

```text
OOMKilled
```

Exit Code

```text
137
```

Guaranteed

```text
Requests = Limits
```

Burstable

```text
Requests < Limits
```

BestEffort

```text
No Requests
No Limits
```

Liveness Fail

```text
Restart
```

Readiness Fail

```text
Remove Endpoint
```

Startup Fail

```text
Restart
```

---

# Interview Questions and Answers

## Beginner Level

### Q1. What is a Resource Request?

Minimum guaranteed resource.

---

### Q2. What is a Resource Limit?

Maximum allowed resource.

---

### Q3. What is 100m CPU?

0.1 CPU Core.

---

### Q4. What is OOMKilled?

Container exceeded memory limit.

---

### Q5. What exit code is usually seen in OOMKilled?

137.

---

### Q6. What happens if CPU limit is exceeded?

CPU is throttled.

---

### Q7. What happens if memory limit is exceeded?

Container is killed.

---

### Q8. What are QoS Classes?

Guaranteed, Burstable, BestEffort.

---

### Q9. Which QoS class is highest?

Guaranteed.

---

### Q10. Which QoS class is lowest?

BestEffort.

---

# Intermediate Level

### Q11. Who uses Requests?

Scheduler.

---

### Q12. Who enforces Limits?

Kubelet.

---

### Q13. Difference between Requests and Limits?

Request = Scheduling

Limit = Enforcement

---

### Q14. What causes Pending Pods?

No suitable node available.

---

### Q15. Difference between Pending and ContainerCreating?

Pending = No Node

ContainerCreating = Node Assigned

---

### Q16. What happens when Liveness fails?

Container Restart.

---

### Q17. What happens when Readiness fails?

Endpoint Removed.

---

### Q18. Does Readiness restart Pods?

No.

---

### Q19. Can a Pod be Running but Not Ready?

Yes.

---

### Q20. What is Startup Probe?

Probe for slow starting applications.

---

# Senior Level

### Q21. Does Scheduler use Requests or Limits?

Requests.

---

### Q22. Why does Scheduler ignore Limits?

Scheduling is based on guaranteed resources.

---

### Q23. Why is CPU throttled but memory killed?

CPU is compressible.

Memory is not compressible.

---

### Q24. What is ResourceQuota?

Namespace level resource control.

---

### Q25. What is LimitRange?

Default resource settings.

---

### Q26. Difference between OOMKilled and Evicted?

OOMKilled = Container Issue

Evicted = Node Issue

---

### Q27. Which QoS class is evicted first?

BestEffort.

---

### Q28. Which QoS class is evicted last?

Guaranteed.

---

### Q29. What happens if Startup Probe budget is too small?

Restart Loop.

---

### Q30. Why was Startup Probe introduced?

To support slow starting applications.

---

### Q31. Difference between Liveness and Readiness?

Liveness = Restart

Readiness = Traffic Control

---

### Q32. Difference between Readiness and Startup?

Readiness = Traffic

Startup = Startup Protection

---

### Q33. What is EndpointSlice?
Modern replacement for Endpoints.


### Q34. How do you troubleshoot OOMKilled?

kubectl describe pod
kubectl logs
kubectl top pod


### Q35. How do you troubleshoot Pending Pods?
kubectl describe pod
kubectl describe node
kubectl top node




