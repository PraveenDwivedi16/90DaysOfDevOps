Day 56 - Kubernetes StatefulSets
Objective

Learn StatefulSets in Kubernetes and understand why they are used for databases and other stateful applications.

What is a StatefulSet?

A StatefulSet is a Kubernetes workload resource used to manage stateful applications.

Stateful applications need:

Stable Pod Names
Stable Storage
Stable Network Identity
Ordered Startup
Ordered Shutdown

Examples:

MySQL
PostgreSQL
MongoDB
Kafka
Elasticsearch
Why Not Deployment?

Deployment works well for stateless applications.

Examples:

Nginx
React Frontend
NodeJS API
Python API

Deployment Pods have random names.

Example:

nginx-deployment-56c45fd5ff-7lvw9
nginx-deployment-56c45fd5ff-jvrkd
nginx-deployment-56c45fd5ff-qmjcf

If a Pod is deleted:

nginx-deployment-56c45fd5ff-7lvw9

New Pod:

nginx-deployment-56c45fd5ff-5gpth

Name changes.

This is fine for web servers but not for databases.

Deployment vs StatefulSet
Feature	Deployment	StatefulSet
Pod Name	Random	Stable
Storage	Shared or Manual	Dedicated PVC per Pod
DNS	Not Stable	Stable
Startup	Parallel	Ordered
Shutdown	Random	Reverse Order
Database Support	No	Yes
Task 1 - Understand the Problem
Create Deployment

deployment.yaml

apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx-deployment

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
        image: nginx
YAML Explanation
apiVersion
apiVersion: apps/v1

Tells Kubernetes which API version to use.

kind
kind: Deployment

Creates a Deployment resource.

metadata
metadata:
  name: nginx-deployment

Deployment name.

replicas
replicas: 3

Create 3 Pods.

selector
selector:
  matchLabels:
    app: nginx

Deployment manages Pods with label:

app=nginx
template

Blueprint for future Pods.

containers
containers:
- name: nginx
  image: nginx

Creates nginx container.

Commands
kubectl apply -f deployment.yaml

kubectl get deploy

kubectl get pods

Delete one Pod:

kubectl delete pod POD_NAME

Observe:

A new Pod is created with a different random name.

Problem with Deployment

Databases require:

mysql-0
mysql-1
mysql-2

Stable names.

Deployment cannot provide this.

Task 2 - Create Headless Service
What is Headless Service?

A Headless Service is a Service without a Cluster IP.

Instead of load balancing traffic, it creates DNS records for individual Pods.

Headless Service YAML

headless-service.yaml

apiVersion: v1
kind: Service

metadata:
  name: nginx-headless

spec:
  clusterIP: None

  selector:
    app: web

  ports:
  - port: 80
YAML Explanation
clusterIP
clusterIP: None

Most important line.

Creates a Headless Service.

No virtual IP.

No load balancing.

Creates DNS records.

selector
selector:
  app: web

Find Pods with label:

app=web
ports
port: 80

Service Port.

Apply
kubectl apply -f headless-service.yaml

kubectl get svc

Output:

CLUSTER-IP = None
Task 3 - Create StatefulSet
StatefulSet YAML

statefulset.yaml

apiVersion: apps/v1
kind: StatefulSet

metadata:
  name: web

spec:
  serviceName: nginx-headless

  replicas: 3

  selector:
    matchLabels:
      app: web

  template:
    metadata:
      labels:
        app: web

    spec:
      containers:
      - name: nginx
        image: nginx

        volumeMounts:
        - name: web-data
          mountPath: /usr/share/nginx/html

  volumeClaimTemplates:
  - metadata:
      name: web-data

    spec:
      accessModes:
      - ReadWriteOnce

      resources:
        requests:
          storage: 100Mi
StatefulSet YAML Explanation
kind
kind: StatefulSet

Creates StatefulSet.

serviceName
serviceName: nginx-headless

Uses Headless Service for stable DNS.

replicas
replicas: 3

Creates:

web-0
web-1
web-2
volumeMounts
volumeMounts:

Mount storage inside container.

Path:

/usr/share/nginx/html
volumeClaimTemplates

Automatically creates PVCs.

No need to manually create PVCs.

accessModes
ReadWriteOnce

One node can read and write.

storage
storage: 100Mi

Each Pod gets 100Mi storage.

StatefulSet Pod Names

Pods:

web-0
web-1
web-2

Stable names.

PVC Names

PVC Naming Pattern:

<template-name>-<pod-name>

Example:

web-data-web-0
web-data-web-1
web-data-web-2
Ordered Pod Creation

StatefulSet creates Pods in order:

web-0
↓
web-1
↓
web-2

Not parallel.

Task 4 - Stable Network Identity
DNS Format
<pod-name>.<service-name>.<namespace>.svc.cluster.local

Example:

web-0.nginx-headless.default.svc.cluster.local
Create BusyBox Pod
kubectl run dns-test \
--image=busybox:1.35 \
--restart=Never \
-it -- sh
DNS Test
nslookup web-0.nginx-headless.default.svc.cluster.local

nslookup web-1.nginx-headless.default.svc.cluster.local

nslookup web-2.nginx-headless.default.svc.cluster.local

Verify:

kubectl get pods -o wide

DNS IP must match Pod IP.

Why Stable DNS?

Database replicas connect using hostnames.

Example:

mysql-0.mysql-headless.default.svc.cluster.local

No need to remember IPs.

Task 5 - Stable Storage
Write Data
kubectl exec web-0 -- sh -c "echo 'Data from web-0' > /usr/share/nginx/html/index.html"
Verify Data
kubectl exec web-0 -- cat /usr/share/nginx/html/index.html

Output:

Data from web-0
Delete Pod
kubectl delete pod web-0
Verify Again
kubectl exec web-0 -- cat /usr/share/nginx/html/index.html

Output:

Data from web-0
Why Data Survived?

Because:

Pod Deleted
PVC Survived
Same PVC Reattached
Storage Mapping
web-0
  |
  ▼
web-data-web-0

web-1
  |
  ▼
web-data-web-1

web-2
  |
  ▼
web-data-web-2
Task 6 - Ordered Scaling
Scale Up
kubectl scale statefulset web --replicas=5

New Pods:

web-3
web-4

New PVCs:

web-data-web-3
web-data-web-4
Scale Down
kubectl scale statefulset web --replicas=3

Deleted:

web-4
web-3

Reverse order.

Important Observation

PVCs remain:

web-data-web-0
web-data-web-1
web-data-web-2
web-data-web-3
web-data-web-4

Data is preserved.

Task 7 - Cleanup
Delete StatefulSet
kubectl delete statefulset web
Delete Headless Service
kubectl delete svc nginx-headless
Verify PVCs
kubectl get pvc

PVCs still exist.

Delete PVCs
kubectl delete pvc --all
Important Notes
StatefulSet Guarantees
Stable Pod Name
Stable Storage
Stable DNS
Ordered Startup
Ordered Shutdown
Pod Naming
web-0
web-1
web-2
DNS Format
<pod-name>.<service-name>.<namespace>.svc.cluster.local
PVC Naming
<template-name>-<pod-name>
Scale Down

Pods deleted.

PVCs stay.

Delete StatefulSet

Pods deleted.

PVCs stay.

Real World StatefulSet Examples
MySQL
mysql-0
mysql-1
mysql-2
PostgreSQL
postgres-0
postgres-1
postgres-2
MongoDB
mongo-0
mongo-1
mongo-2
Kafka
broker-0
broker-1
broker-2
Elasticsearch
es-0
es-1
es-2
Interview Questions and Answers
1. What is StatefulSet?

StatefulSet is a Kubernetes workload used for stateful applications.

2. Why use StatefulSet?

For stable identity and persistent storage.

3. What is Headless Service?

A Service with:

clusterIP: None
4. Why is Headless Service required?

To create stable DNS records.

5. StatefulSet pod naming pattern?
web-0
web-1
web-2
6. PVC naming pattern?
<template-name>-<pod-name>

Example:

web-data-web-0
7. Difference between Deployment and StatefulSet?

Deployment = Stateless

StatefulSet = Stateful

8. What happens if StatefulSet Pod is deleted?

Same Pod name is recreated.

9. Does StatefulSet preserve storage?

Yes.

10. What is volumeClaimTemplates?

Creates PVCs automatically for each Pod.

11. What is ReadWriteOnce?

One node can read/write the volume.

12. What is serviceName in StatefulSet?

Headless Service name.

13. Startup order in StatefulSet?
0 → 1 → 2
14. Shutdown order in StatefulSet?
2 → 1 → 0
15. Does scaling down delete PVCs?

No.

16. Does deleting StatefulSet delete PVCs?

No.

17. Why databases need StatefulSet?

Because they need stable identity and storage.

18. What DNS format does StatefulSet use?
<pod-name>.<service-name>.<namespace>.svc.cluster.local
19. Can StatefulSet work without PVC?

Yes, but data persistence is lost.

20. Which applications commonly use StatefulSets?

MySQL, PostgreSQL, MongoDB, Kafka, Elasticsearch.
