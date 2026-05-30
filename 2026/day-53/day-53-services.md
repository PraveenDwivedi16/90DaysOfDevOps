# Day 53 – Kubernetes Services

## Goal of this task

In this task, we learned how to expose a Kubernetes Deployment using different Service types.

We created one Deployment with 3 Nginx Pods and exposed it using:

1. `ClusterIP`
2. `NodePort`
3. `LoadBalancer`

We also verified service communication from inside the cluster, tested DNS-based service discovery, tested temporary port forwarding, and cleaned up all resources.

---

## Why do we need Kubernetes Services?

In Kubernetes, every Pod gets its own IP address.

But there are two main problems:

1. **Pod IPs are not stable**
   - If a Pod restarts, gets deleted, or is recreated, it may get a new IP address.

2. **A Deployment usually runs multiple Pods**
   - If we have 3 Pods, which Pod IP should the client call?

Example:

```text
Deployment: web-app

Pods:
web-app-pod-1 -> 10.244.1.8
web-app-pod-2 -> 10.244.1.9
web-app-pod-3 -> 10.244.2.9
```

If we directly call Pod IPs, our application becomes unstable because Pod IPs can change.

A **Service** solves this problem.

A Kubernetes Service gives us:

- A stable IP address
- A stable DNS name
- Load balancing across matching Pods
- A single entry point to access multiple Pods

Traffic flow:

```text
Client
  |
  v
Service stable IP / DNS
  |
  v
Pod 1 / Pod 2 / Pod 3
```

---

# Task 1: Deploy the Application

## What we did

First, we created a Deployment named `web-app` with 3 Nginx Pods.

The Deployment will be exposed later using different Service types.

---

## File: `app-deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  labels:
    app: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
```

---

## YAML explanation

### `apiVersion: apps/v1`

This tells Kubernetes that we are using the `apps/v1` API version.

Deployments belong to the `apps/v1` API group.

---

### `kind: Deployment`

This tells Kubernetes to create a Deployment.

A Deployment is responsible for:

- Creating Pods
- Maintaining the desired number of Pods
- Recreating Pods if they fail
- Managing rolling updates

---

### `metadata.name: web-app`

This is the name of the Deployment.

We can check it using:

```bash
kubectl get deploy
```

---

### `replicas: 3`

This tells Kubernetes to run 3 replicas of the Pod.

So Kubernetes will create 3 Nginx Pods.

---

### `selector.matchLabels`

```yaml
selector:
  matchLabels:
    app: web-app
```

The Deployment uses this selector to manage Pods with the label:

```yaml
app: web-app
```

Important:

```text
Deployment selector must match Pod template labels.
```

---

### `template.metadata.labels`

```yaml
template:
  metadata:
    labels:
      app: web-app
```

These labels are added to the Pods created by this Deployment.

Later, Services will use this label to find the Pods.

---

### Container section

```yaml
containers:
- name: nginx
  image: nginx:1.25
  ports:
  - containerPort: 80
```

This means:

- Container name is `nginx`
- Image is `nginx:1.25`
- The container listens on port `80`

---

## Commands used

```bash
kubectl apply -f app-deployment.yaml
```

Check Deployment:

```bash
kubectl get deploy
```

Check Pods:

```bash
kubectl get pods
```

Check Pods with IP addresses:

```bash
kubectl get pods -o wide
```

---

## Actual verification

Deployment output:

```text
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
web-app   3/3     3            3           15m
```

Pods output:

```text
NAME                       READY   STATUS    RESTARTS   AGE   IP           NODE
web-app-6cffb4b956-f4chg   1/1     Running   0          14m   10.244.1.9   praveen-cluster-worker
web-app-6cffb4b956-fv684   1/1     Running   0          14m   10.244.1.8   praveen-cluster-worker
web-app-6cffb4b956-ltnnm   1/1     Running   0          14m   10.244.2.9   praveen-cluster-worker2
```

## Important points

- All 3 Pods were running.
- Each Pod had a different IP.
- These Pod IPs are not stable and can change if Pods restart.
- This is why we need Services.

---

# Task 2: ClusterIP Service

## What is ClusterIP?

`ClusterIP` is the default Kubernetes Service type.

It exposes the application only inside the cluster.

This means other Pods inside the Kubernetes cluster can access the Service, but it cannot be directly accessed from the browser or host machine.

Traffic flow:

```text
Pod inside cluster
  |
  v
ClusterIP Service
  |
  v
web-app Pods
```

---

## File: `clusterip-service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-app-clusterip
spec:
  type: ClusterIP
  selector:
    app: web-app
  ports:
  - port: 80
    targetPort: 80
```

---

## YAML explanation

### `kind: Service`

This tells Kubernetes to create a Service.

---

### `metadata.name: web-app-clusterip`

This is the Service name.

This name is also used for Kubernetes DNS.

Inside the cluster, we can access this Service using:

```text
http://web-app-clusterip
```

---

### `type: ClusterIP`

This creates an internal-only Service.

It gets a stable internal IP address.

---

### `selector.app: web-app`

```yaml
selector:
  app: web-app
```

This is very important.

The Service sends traffic to Pods that have this label:

```yaml
app: web-app
```

If the selector does not match the Pod labels, the Service will have no endpoints and traffic will go nowhere.

---

### `port: 80`

This is the port exposed by the Service.

---

### `targetPort: 80`

This is the port on the target Pod/container.

Traffic flow:

```text
Service port 80 -> Pod targetPort 80
```

---

## Commands used

Create the Service:

```bash
kubectl apply -f clusterip-service.yaml
```

Check Services:

```bash
kubectl get svc
```

Check endpoints:

```bash
kubectl get endpoints web-app-clusterip
```

Test from inside the cluster:

```bash
kubectl run test-client --image=busybox:latest --rm -it --restart=Never -- sh
```

Inside the test Pod:

```sh
wget -qO- http://web-app-clusterip
exit
```

---

## Actual verification

Service output:

```text
NAME                TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
web-app-clusterip   ClusterIP   10.96.93.110   <none>        80/TCP    12s
```

Endpoint output:

```text
NAME                ENDPOINTS                                   AGE
web-app-clusterip   10.244.1.8:80,10.244.1.9:80,10.244.2.9:80   38s
```

Test output:

```html
<h1>Welcome to nginx!</h1>
```

## Important points

- ClusterIP assigned: `10.96.93.110`
- Service endpoints matched the 3 Pod IPs.
- Nginx page was accessible from inside the cluster.
- ClusterIP is used for internal service-to-service communication.

---

# Task 3: Discover Services with DNS

## What is Kubernetes DNS?

Kubernetes automatically creates DNS records for Services.

Every Service gets a DNS name in this format:

```text
<service-name>.<namespace>.svc.cluster.local
```

For our Service:

```text
web-app-clusterip.default.svc.cluster.local
```

Because the Service name is:

```text
web-app-clusterip
```

And the namespace is:

```text
default
```

Inside the same namespace, we can also use the short name:

```text
web-app-clusterip
```

---

## Commands used

Run a temporary DNS test Pod:

```bash
kubectl run dns-test --image=busybox:latest --rm -it --restart=Never -- sh
```

Inside the Pod:

```sh
wget -qO- http://web-app-clusterip
wget -qO- http://web-app-clusterip.default.svc.cluster.local
nslookup web-app-clusterip
exit
```

---

## Actual verification

DNS lookup output:

```text
Server:         10.96.0.10
Address:        10.96.0.10:53

Name:   web-app-clusterip.default.svc.cluster.local
Address: 10.96.93.110
```

## Important points

- DNS resolved the Service name to the ClusterIP.
- Service DNS name resolved to `10.96.93.110`.
- This matched the ClusterIP of `web-app-clusterip`.
- Short Service name worked inside the same namespace.
- Full DNS name also worked.

## About NXDOMAIN messages

During `nslookup`, BusyBox showed some `NXDOMAIN` messages for names like:

```text
web-app-clusterip.cluster.local
web-app-clusterip.svc.cluster.local
```

This is not a problem.

BusyBox tries multiple DNS search paths. Some invalid names fail, but the correct full DNS record was resolved successfully:

```text
web-app-clusterip.default.svc.cluster.local
```

---

# Task 4: NodePort Service

## What is NodePort?

A `NodePort` Service exposes the application on a port on every Kubernetes node.

It allows external access using:

```text
<NodeIP>:<NodePort>
```

Example:

```text
<NodeIP>:30080
```

Traffic flow:

```text
External client
  |
  v
NodeIP:30080
  |
  v
NodePort Service
  |
  v
web-app Pods
```

NodePort range:

```text
30000 - 32767
```

---

## File: `nodeport-service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-app-nodeport
spec:
  type: NodePort
  selector:
    app: web-app
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080
```

---

## YAML explanation

### `type: NodePort`

This exposes the Service using a node-level port.

---

### `nodePort: 30080`

This opens port `30080` on each Kubernetes node.

Traffic flow:

```text
<NodeIP>:30080 -> Service port 80 -> Pod port 80
```

---

### `selector.app: web-app`

This connects the Service to all Pods with label:

```yaml
app: web-app
```

---

## Commands used

Create the Service:

```bash
kubectl apply -f nodeport-service.yaml
```

Check Services:

```bash
kubectl get svc
```

Check endpoints:

```bash
kubectl get endpoints web-app-nodeport
```

Check node IPs:

```bash
kubectl get nodes -o wide
```

Try accessing from host:

```bash
curl http://172.19.0.2:30080
curl http://172.19.0.3:30080
curl http://172.19.0.4:30080
```

Test from inside the cluster:

```bash
kubectl run nodeport-test --image=busybox:latest --rm -it --restart=Never -- sh
```

Inside the Pod:

```sh
wget -qO- http://web-app-nodeport
exit
```

---

## Actual verification

Service output:

```text
web-app-nodeport   NodePort   10.96.167.149   <none>   80:30080/TCP
```

Endpoint output:

```text
web-app-nodeport   10.244.1.8:80,10.244.1.9:80,10.244.2.9:80
```

Inside cluster test output:

```html
<h1>Welcome to nginx!</h1>
```

## Important points

- NodePort Service was created successfully.
- NodePort was `30080`.
- Service endpoints matched the 3 Pods.
- Service worked from inside the cluster.
- Direct access using Kind node internal IP did not work from Windows host.

---

## Important note: NodePort in Kind on Windows

In a normal Kubernetes environment, NodePort can be accessed using:

```text
<NodeIP>:<NodePort>
```

But in Kind on Windows, Kubernetes nodes run as Docker containers.

So node IPs like:

```text
172.19.0.2
172.19.0.3
172.19.0.4
```

may not be directly reachable from the Windows host.

That is why these commands timed out:

```bash
curl http://172.19.0.2:30080
curl http://172.19.0.3:30080
curl http://172.19.0.4:30080
```

This does not mean the Service is wrong.

The Service was correctly verified from inside the cluster.

---

## Temporary port forwarding for local testing

For local development and testing, we can use `kubectl port-forward`.

Command used:

```bash
kubectl port-forward service/web-app-nodeport 8080:80
```

Meaning:

```text
localhost:8080 -> web-app-nodeport Service port 80 -> Pod port 80
```

Then test from another terminal or browser:

```bash
curl http://localhost:8080
```

Browser:

```text
http://localhost:8080
```

Actual output:

```text
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
Handling connection for 8080
```

## Temporary vs permanent access

### Temporary access

`kubectl port-forward` is temporary.

It works only while the terminal is running.

Good for:

- Local testing
- Debugging
- Development

Not good for:

- Production
- Shared access
- Long-term access

---

### Permanent NodePort access in Kind

For permanent host access in Kind, the cluster should be created with `extraPortMappings`.

Example Kind config:

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  extraPortMappings:
  - containerPort: 30080
    hostPort: 30080
    protocol: TCP
- role: worker
- role: worker
```

Create cluster:

```bash
kind create cluster --name praveen-cluster --config kind-config.yaml
```

Then NodePort can be accessed from the host:

```text
http://localhost:30080
```

Important:

```text
extraPortMappings must be added when creating the Kind cluster.
Usually, we cannot add it to an already created cluster.
```

---

# Task 5: LoadBalancer Service

## What is LoadBalancer?

A `LoadBalancer` Service is used to expose an application externally using a cloud provider load balancer.

In cloud Kubernetes environments:

- AWS EKS creates an AWS Load Balancer
- Azure AKS creates an Azure Load Balancer
- GKE creates a Google Cloud Load Balancer

Traffic flow:

```text
Internet user
  |
  v
Cloud Load Balancer
  |
  v
Kubernetes LoadBalancer Service
  |
  v
Pods
```

---

## File: `loadbalancer-service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-app-loadbalancer
spec:
  type: LoadBalancer
  selector:
    app: web-app
  ports:
  - port: 80
    targetPort: 80
```

---

## YAML explanation

### `type: LoadBalancer`

This asks Kubernetes to create an external load balancer.

In a cloud cluster, the cloud provider creates a real external load balancer.

In Kind/local cluster, there is no cloud provider, so external IP remains pending.

---

### `selector.app: web-app`

This Service also targets the same 3 Nginx Pods.

---

### `port: 80` and `targetPort: 80`

Traffic goes from Service port 80 to Pod port 80.

---

## Commands used

Create the Service:

```bash
kubectl apply -f loadbalancer-service.yaml
```

Check Services:

```bash
kubectl get svc
```

Check endpoints:

```bash
kubectl get endpoints web-app-loadbalancer
```

Describe Service:

```bash
kubectl describe service web-app-loadbalancer
```

Optional temporary port-forward:

```bash
kubectl port-forward service/web-app-loadbalancer 8081:80
```

Test:

```bash
curl http://localhost:8081
```

---

## Actual verification

Service output:

```text
web-app-loadbalancer   LoadBalancer   10.96.236.164   <pending>   80:31535/TCP
```

Endpoint output:

```text
web-app-loadbalancer   10.244.1.8:80,10.244.1.9:80,10.244.2.9:80
```

Describe output:

```text
Type:                     LoadBalancer
IP:                       10.96.236.164
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  31535/TCP
Endpoints:                10.244.2.9:80,10.244.1.9:80,10.244.1.8:80
External Traffic Policy:  Cluster
Internal Traffic Policy:  Cluster
```

Port-forward output:

```text
Forwarding from 127.0.0.1:8081 -> 80
Forwarding from [::1]:8081 -> 80
Handling connection for 8081
```

## Important points

- LoadBalancer Service was created successfully.
- ClusterIP was assigned: `10.96.236.164`
- Auto NodePort was assigned: `31535`
- External IP was `<pending>` because this was a Kind local cluster.
- This is expected behavior.

---

## Why is EXTERNAL-IP pending in Kind?

Kind is a local Kubernetes cluster running inside Docker.

It does not have a cloud provider integration.

So when we create:

```yaml
type: LoadBalancer
```

Kubernetes accepts the Service, but no real external load balancer is created.

Therefore:

```text
EXTERNAL-IP = <pending>
```

In a real cloud cluster, this would become a public IP or load balancer hostname.

---

# Task 6: Understand Service Types Side by Side

## What we did

We compared all three Services:

```bash
kubectl get svc -o wide
```

Actual output:

```text
NAME                   TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE     SELECTOR
kubernetes             ClusterIP      10.96.0.1       <none>        443/TCP        11d     <none>
web-app-clusterip      ClusterIP      10.96.93.110    <none>        80/TCP         55m     app=web-app
web-app-loadbalancer   LoadBalancer   10.96.236.164   <pending>     80:31535/TCP   8m35s   app=web-app
web-app-nodeport       NodePort       10.96.167.149   <none>        80:30080/TCP   43m     app=web-app
```

---

## Service comparison

| Service Type | Accessible From | External IP | NodePort | Use Case |
|---|---|---|---|---|
| ClusterIP | Inside cluster only | No | No | Internal communication |
| NodePort | Node IP + NodePort | No | Yes | Development/testing/direct node access |
| LoadBalancer | Cloud load balancer | Yes in cloud, pending in Kind | Yes | Production external traffic |

---

## Important concept

Each Service type builds on the previous one.

```text
ClusterIP
  |
  v
NodePort = ClusterIP + NodePort
  |
  v
LoadBalancer = ClusterIP + NodePort + Cloud Load Balancer
```

So:

```text
ClusterIP gives internal access.
NodePort gives internal access plus node-level access.
LoadBalancer gives internal access plus node-level access plus cloud external access.
```

---

## Endpoints comparison

Commands used:

```bash
kubectl get endpoints web-app-clusterip
kubectl get endpoints web-app-nodeport
kubectl get endpoints web-app-loadbalancer
```

Actual output showed the same endpoints for all Services:

```text
10.244.1.8:80,10.244.1.9:80,10.244.2.9:80
```

This means all three Services were routing traffic to the same 3 Pods.

Why?

Because all three Services used the same selector:

```yaml
selector:
  app: web-app
```

---

## About Endpoints warning

The cluster showed this warning:

```text
Warning: v1 Endpoints is deprecated in v1.33+; use discovery.k8s.io/v1 EndpointSlice
```

This means newer Kubernetes versions recommend using EndpointSlice.

Modern command:

```bash
kubectl get endpointslices
```

Specific Service EndpointSlice:

```bash
kubectl get endpointslice -l kubernetes.io/service-name=web-app-clusterip
kubectl get endpointslice -l kubernetes.io/service-name=web-app-nodeport
kubectl get endpointslice -l kubernetes.io/service-name=web-app-loadbalancer
```

---

# Task 7: Clean Up

## What we did

After verification, we deleted all created resources.

Resources deleted:

- `web-app` Deployment
- `web-app-clusterip` Service
- `web-app-nodeport` Service
- `web-app-loadbalancer` Service

---

## Commands used

Delete Services:

```bash
kubectl delete -f clusterip-service.yaml
kubectl delete -f nodeport-service.yaml
kubectl delete -f loadbalancer-service.yaml
```

Delete Deployment:

```bash
kubectl delete -f app-deployment.yaml
```

Verify Pods:

```bash
kubectl get pods
```

Expected:

```text
No resources found in default namespace.
```

Verify Services:

```bash
kubectl get svc
```

Expected:

```text
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   ...
```

Verify Deployments:

```bash
kubectl get deploy
```

Expected:

```text
No resources found in default namespace.
```

---

# All YAML Files Summary

## `app-deployment.yaml`

Creates 3 Nginx Pods using a Deployment.

```bash
kubectl apply -f app-deployment.yaml
```

---

## `clusterip-service.yaml`

Creates an internal Service.

```bash
kubectl apply -f clusterip-service.yaml
```

---

## `nodeport-service.yaml`

Creates a Service exposed on node port `30080`.

```bash
kubectl apply -f nodeport-service.yaml
```

---

## `loadbalancer-service.yaml`

Creates a LoadBalancer Service.

```bash
kubectl apply -f loadbalancer-service.yaml
```

---

# Useful Commands

## Check Pods

```bash
kubectl get pods
kubectl get pods -o wide
```

## Check Deployments

```bash
kubectl get deploy
kubectl describe deployment web-app
```

## Check Services

```bash
kubectl get svc
kubectl get svc -o wide
kubectl describe service web-app-clusterip
kubectl describe service web-app-nodeport
kubectl describe service web-app-loadbalancer
```

## Check Endpoints

```bash
kubectl get endpoints web-app-clusterip
kubectl get endpoints web-app-nodeport
kubectl get endpoints web-app-loadbalancer
```

## Check EndpointSlices

```bash
kubectl get endpointslices
kubectl get endpointslice -l kubernetes.io/service-name=web-app-clusterip
```

## Test Service from inside cluster

```bash
kubectl run test-client --image=busybox:latest --rm -it --restart=Never -- sh
```

Inside Pod:

```sh
wget -qO- http://web-app-clusterip
exit
```

## Temporary port-forward

```bash
kubectl port-forward service/web-app-nodeport 8080:80
kubectl port-forward service/web-app-loadbalancer 8081:80
```

Test:

```bash
curl http://localhost:8080
curl http://localhost:8081
```

---

# Common Mistakes and Troubleshooting

## 1. Service has no endpoints

Check:

```bash
kubectl get endpoints <service-name>
```

If endpoints are empty, selector may not match Pod labels.

Check Pod labels:

```bash
kubectl get pods --show-labels
```

Check Service selector:

```bash
kubectl describe service <service-name>
```

---

## 2. ClusterIP not accessible from browser

This is expected.

ClusterIP is internal-only.

Test it from inside the cluster using a temporary Pod.

---

## 3. NodePort not accessible in Kind on Windows

This can happen because Kind nodes run inside Docker containers.

Use one of these options:

1. Test from inside the cluster
2. Use `kubectl port-forward`
3. Recreate Kind cluster with `extraPortMappings`

---

## 4. LoadBalancer EXTERNAL-IP is pending

This is expected in Kind/local clusters.

A real external IP is created only in cloud Kubernetes clusters or with a local load balancer solution like MetalLB.

---

# Interview Questions and Answers

## 1. What problem does a Kubernetes Service solve?

A Service provides a stable network endpoint for Pods.

Pods have dynamic IP addresses and can be recreated anytime. A Service gives a stable IP and DNS name and load balances traffic across matching Pods.

---

## 2. Why should we not directly use Pod IPs?

Pod IPs are not stable.

If a Pod restarts or is recreated, it may get a new IP. Directly using Pod IPs makes communication unreliable.

---

## 3. How does a Service know which Pods to send traffic to?

A Service uses selectors.

Example:

```yaml
selector:
  app: web-app
```

It sends traffic to Pods with matching labels:

```yaml
app: web-app
```

---

## 4. What is ClusterIP?

ClusterIP is the default Service type.

It exposes the Service only inside the cluster. It is mainly used for internal communication between applications.

---

## 5. What is NodePort?

NodePort exposes a Service on a static port on every Kubernetes node.

Traffic flow:

```text
<NodeIP>:<NodePort> -> Service -> Pod
```

NodePort range is usually:

```text
30000 - 32767
```

---

## 6. What is LoadBalancer?

LoadBalancer exposes a Service using an external load balancer.

In cloud Kubernetes, the cloud provider creates a real load balancer and assigns an external IP or DNS name.

---

## 7. Why is EXTERNAL-IP pending in Kind?

Kind is a local cluster and does not have a cloud provider load balancer.

So LoadBalancer Service is created, but no external IP is assigned.

That is why EXTERNAL-IP remains `<pending>`.

---

## 8. Does LoadBalancer also create NodePort and ClusterIP?

Yes.

A LoadBalancer Service also has:

- ClusterIP
- NodePort
- LoadBalancer configuration

Service hierarchy:

```text
LoadBalancer -> NodePort -> ClusterIP -> Pods
```

---

## 9. What is the difference between `port`, `targetPort`, and `nodePort`?

### `port`

The port exposed by the Service.

### `targetPort`

The port on the target Pod/container.

### `nodePort`

The port opened on every Kubernetes node for external access.

Example:

```yaml
ports:
- port: 80
  targetPort: 80
  nodePort: 30080
```

Meaning:

```text
NodeIP:30080 -> Service:80 -> Pod:80
```

---

## 10. What are Endpoints?

Endpoints are the actual Pod IPs and ports behind a Service.

Example:

```text
10.244.1.8:80,10.244.1.9:80,10.244.2.9:80
```

Command:

```bash
kubectl get endpoints <service-name>
```

---

## 11. What is EndpointSlice?

EndpointSlice is the modern replacement for Endpoints.

It scales better for large clusters and is recommended in newer Kubernetes versions.

Command:

```bash
kubectl get endpointslices
```

---

## 12. What is Kubernetes DNS?

Kubernetes automatically creates DNS records for Services.

Format:

```text
<service-name>.<namespace>.svc.cluster.local
```

Example:

```text
web-app-clusterip.default.svc.cluster.local
```

Inside the same namespace, we can use the short name:

```text
web-app-clusterip
```

---

## 13. What is `kubectl port-forward`?

`kubectl port-forward` creates a temporary tunnel from the local machine to a Pod or Service.

Example:

```bash
kubectl port-forward service/web-app-nodeport 8080:80
```

Meaning:

```text
localhost:8080 -> Service port 80 -> Pod port 80
```

It is temporary and used for local testing/debugging.

---

## 14. Is port-forwarding used in production?

No.

Port-forwarding is mainly for local testing and debugging.

For production, use:

- LoadBalancer
- Ingress
- API Gateway
- Cloud load balancer

---

## 15. What happens if Service selector is wrong?

The Service will be created, but it will not send traffic to any Pod.

Endpoints will be empty.

Check using:

```bash
kubectl get endpoints <service-name>
```

---

# Final Learning Summary

In this task, we learned that Pods are temporary and their IPs are not reliable for direct communication.

Kubernetes Services provide a stable way to access Pods.

We created:

- A Deployment with 3 Nginx Pods
- A ClusterIP Service for internal access
- A NodePort Service for node-level access
- A LoadBalancer Service for cloud-style external access

We also tested:

- Service endpoints
- Service DNS
- Internal communication using temporary Pods
- Temporary local access using port-forwarding
- Cleanup of all resources

Most important learning:

```text
Service selector connects Service to Pods.
If selector matches Pod labels, traffic works.
If selector does not match, endpoints are empty.
