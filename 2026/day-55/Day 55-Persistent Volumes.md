Day-55 Persistent Volumes (PV) and Persistent Volume Claims
Part-1: Storage Fundamentals + Task-1 + Task-2
Chapter 1: Why Do We Need Storage?

Before learning PV and PVC, first understand the problem.

Suppose you have a Pod:

apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: app
    image: nginx

Pod starts.

Now enter inside Pod:

kubectl exec -it my-pod -- sh

Create a file:

echo "Hello Kubernetes" > test.txt

Check:

cat test.txt

Output:

Hello Kubernetes

Everything looks fine.

Now delete Pod:

kubectl delete pod my-pod

Create Pod again.

Enter Pod:

kubectl exec -it my-pod -- sh

Check file:

cat test.txt

Output:

No such file or directory

File disappeared.

Why Did Data Disappear?

Because container storage is:

Ephemeral Storage

Meaning:

Temporary Storage

Lifecycle:

Pod Created
     ↓
Container Created
     ↓
Data Written
     ↓
Pod Deleted
     ↓
Container Deleted
     ↓
Data Deleted
Real World Problem

Imagine:

MySQL Database

contains:

100000 Customers

Pod crashes.

Data gone.

Company gone 😂

That's why Kubernetes provides:

Persistent Storage
Kubernetes Storage Architecture

Remember this forever:

Application
     │
     ▼
    Pod
     │
     ▼
    PVC
     │
     ▼
     PV
     │
     ▼
 Physical Storage
Chapter 2: Volume Types in Kubernetes

Before PV/PVC, know important volume types.

1. emptyDir

Most basic volume.

Example:

volumes:
- name: temp-storage
  emptyDir: {}

Behavior:

Pod Start
   ↓
Volume Created
   ↓
Data Written
   ↓
Pod Deleted
   ↓
Volume Deleted
   ↓
Data Lost

Use Cases:

Cache
Temporary Files
Build Artifacts
2. hostPath

Mounts a folder from node.

Example:

hostPath:
  path: /tmp/data

Behavior:

Node Folder
     │
     ▼
Container

Use Cases:

Learning
Testing
Labs

Not recommended in production.

3. ConfigMap Volume

Used for configuration files.

Example:

nginx.conf
application.properties
4. Secret Volume

Used for:

Passwords
API Keys
Certificates
Tokens
5. NFS Volume

Shared storage.

Supports:

RWX
(ReadWriteMany)
6. CSI Storage

Production storage.

Examples:

AWS EBS
Azure Disk
GCP PD
Longhorn
Ceph
Chapter 3: Task-1 Data Loss Demonstration

Goal:

Prove:

Pod Delete
     ↓
Data Delete
Task-1 Architecture
Pod
 │
 ▼
emptyDir
Task-1 YAML

File:

vi ephemeral-pod.yaml

Paste:

apiVersion: v1
kind: Pod

metadata:
  name: ephemeral-pod

spec:
  containers:
  - name: app
    image: busybox

    command:
    - sh
    - -c
    - |
      while true
      do
        date >> /data/message.txt
        sleep 30
      done

    volumeMounts:
    - name: temp-storage
      mountPath: /data

  volumes:
  - name: temp-storage
    emptyDir: {}
Line-by-Line YAML Explanation
apiVersion
apiVersion: v1

Using Kubernetes core API.

kind
kind: Pod

Create Pod.

metadata
metadata:
  name: ephemeral-pod

Pod name.

image
image: busybox

Lightweight Linux image.

Used for testing.

command
command:

Runs automatically after container starts.

date Command
date >> /data/message.txt

Appends timestamp.

Example:

Tue Jun 2 10:00:00 UTC 2026
volumeMounts
volumeMounts:

Mount storage inside container.

mountPath
mountPath: /data

Storage becomes available at:

/data
volumes
volumes:

Defines storage.

emptyDir
emptyDir: {}

Temporary storage.

Deleted with Pod.

Important Concept

Many beginners confuse:

volumeMounts

and

volumes
volumes

Creates storage.

Example:

volumes:
- name: temp-storage
  emptyDir: {}

Meaning:

Create Storage
volumeMounts

Attaches storage.

Example:

volumeMounts:
- name: temp-storage
  mountPath: /data

Meaning:

Attach Storage

Diagram:

temp-storage
      │
      ▼
    /data
Task-1 Commands

Apply:

kubectl apply -f ephemeral-pod.yaml

Check:

kubectl get pods

Expected:

ephemeral-pod Running

Check file:

kubectl exec -it ephemeral-pod -- cat /data/message.txt

Example:

Tue Jun 2 10:00:00 UTC 2026

Delete Pod:

kubectl delete pod ephemeral-pod

Recreate:

kubectl apply -f ephemeral-pod.yaml

Check file again:

kubectl exec -it ephemeral-pod -- cat /data/message.txt

Example:

Tue Jun 2 10:15:00 UTC 2026

Old timestamp missing.

Verification

Question:

Is timestamp same?

Answer:

No

Why?

emptyDir deleted with Pod
Learning From Task-1

Remember:

emptyDir

means:

Temporary Storage

and

Pod Delete
    ↓
Data Delete
Chapter 4: Persistent Volume (PV)

Now we solve the problem.

What is Persistent Volume?

Definition:

Persistent Volume = Actual Storage

Simple:

PV = Disk

Examples:

1Gi
10Gi
100Gi
PV Characteristics

PV is:

Cluster Wide Resource

Not namespaced.

Check:

kubectl get pv
PV Lifecycle
Available
    ↓
Bound
    ↓
Released
Available

PV exists.

No PVC attached.

Bound

PV attached to PVC.

Released

PVC deleted.

PV still exists.

Task-2 Create Persistent Volume

Goal:

Create:

1Gi Persistent Storage
Step-1 Create Host Directory

Command:

mkdir -p /tmp/k8s-pv-data

Verify:

ls -ld /tmp/k8s-pv-data

Expected:

drwxr-xr-x ...
Why This Folder?

This folder becomes actual storage.

Diagram:

/tmp/k8s-pv-data
        ▲
        │
        │
        PV
Step-2 Create PV

File:

vi pv.yaml

Paste:

apiVersion: v1
kind: PersistentVolume

metadata:
  name: manual-pv

spec:
  capacity:
    storage: 1Gi

  accessModes:
  - ReadWriteOnce

  persistentVolumeReclaimPolicy: Retain

  hostPath:
    path: /tmp/k8s-pv-data
Deep YAML Explanation
metadata.name
name: manual-pv

PV name.

capacity
capacity:
  storage: 1Gi

Storage size.

Meaning:

1 Gigabyte
accessModes
accessModes:
- ReadWriteOnce

Meaning:

One Node
Read + Write
persistentVolumeReclaimPolicy
Retain

Meaning:

PVC Deleted
     ↓
Keep Data
hostPath
hostPath:
  path: /tmp/k8s-pv-data

Actual storage location.

Apply PV
kubectl apply -f pv.yaml

Expected:

persistentvolume/manual-pv created
Verification

Check:

kubectl get pv

Expected:

manual-pv   1Gi   RWO   Retain   Available

Describe:

kubectl describe pv manual-pv

Observe:

Capacity
Access Modes
Reclaim Policy
Status
Host Path
Why STATUS = Available?

Because:

PV Created

But:

PVC Not Created Yet

Diagram:

manual-pv

Available

Waiting For PVC
Task-2 Learning Summary

PV means:

Actual Storage

Current State:

Folder Created
     ↓
PV Created
     ↓
Waiting For PVC
Part-1 Revision Sheet

Task-1:

emptyDir

Result:

Pod Delete
     ↓
Data Delete

Task-2:

Persistent Volume

Result:

PV Created
     ↓
Available
Interview Questions
What is emptyDir?

Temporary storage deleted with Pod.

Difference between volumes and volumeMounts?

volumes:
Creates storage.

volumeMounts:
Attaches storage.

What is PV?

Actual storage resource.

Why is PV status Available?

Because no PVC is attached.

Q:- What is hostPath?
Node directory mounted as storage.

Q:- What is Retain policy?
Keep PV and data after PVC deletion.

Chapter 5 - Persistent Volume Claim (PVC)

Task-2 me humne PV banaya tha.

Current Architecture:

/tmp/k8s-pv-data
        ▲
        │
        │
     manual-pv

Ab problem ye hai:

Pod

direct:

PV

use nahi kar sakta.

Why Pod Cannot Directly Use PV?

New learners ka common question:

PV already hai

To Pod direct use kyu nahi kar sakta?

Good Question.

Wrong Architecture
Pod
 │
 ▼
PV

Kubernetes allow nahi karta.

Correct Architecture
Pod
 │
 ▼
PVC
 │
 ▼
PV
 │
 ▼
Storage
Real World Example

Hotel Example

PV:

Room 101
1Gi

Already available.

Developer:

Mujhe Storage Chahiye

PVC creates request.

Example:

Need 500Mi

Kubernetes compares:

PV

with

PVC

If matched:

Bound
What is PVC?

Definition:

PVC = Storage Request

Simple:

PV = Storage

PVC = Storage Request

Remember forever.

Why PVC Exists?

Without PVC:

Application must know:

Storage Type
Storage Location
Storage Details

Bad design.

With PVC:

Application only says:

Need 500Mi Storage

Kubernetes handles rest.

Task-3 Create PVC

Goal:

Create PVC.

Bind with:

manual-pv
PVC YAML

File:

vi pvc.yaml

Paste:

apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: my-pvc

spec:
  storageClassName: ""

  accessModes:
  - ReadWriteOnce

  resources:
    requests:
      storage: 500Mi
Deep YAML Explanation
apiVersion
apiVersion: v1

Core Kubernetes API.

kind
kind: PersistentVolumeClaim

Create PVC.

metadata
metadata:
  name: my-pvc

PVC name.

storageClassName
storageClassName: ""

Very Important.

Meaning:

Do NOT use Dynamic Provisioning

Use only existing PV.

Why We Added This?

Remember?

Initially PVC was:

Pending

Reason:

StorageClass = standard

But PV:

StorageClass = empty

Mismatch.

Adding:

storageClassName: ""

forces PVC to use static provisioning.

accessModes
accessModes:
- ReadWriteOnce

Must match PV.

PV:

RWO

PVC:

RWO

Match.

requests.storage
storage: 500Mi

Meaning:

Need 500Mi Storage
PVC Binding Algorithm

Very Important Interview Topic.

Kubernetes checks:

Check 1 Capacity

PV:

1Gi

PVC:

500Mi

Check:

1Gi >= 500Mi

PASS

Check 2 Access Mode

PV:

RWO

PVC:

RWO

PASS

Check 3 StorageClass

PV:

empty

PVC:

empty

PASS

Result:

Bound
Apply PVC
kubectl apply -f pvc.yaml

Expected:

persistentvolumeclaim/my-pvc created
Verify PVC
kubectl get pvc

Expected:

NAME     STATUS   VOLUME
my-pvc   Bound    manual-pv
Verify PV
kubectl get pv

Expected:

manual-pv Bound
Explain Output

Example:

my-pvc   Bound   manual-pv

Meaning:

PVC Attached To manual-pv
CLAIM Column

Example:

manual-pv
CLAIM:
default/my-pvc

Meaning:

manual-pv is attached to my-pvc
Common PVC Pending Reasons

Interview Favorite Question.

Reason 1

StorageClass mismatch.

Reason 2

Access Mode mismatch.

Example:

PV:

RWO

PVC:

RWX

Result:

Pending
Reason 3

Capacity mismatch.

PV:

1Gi

PVC:

2Gi

Result:

Pending
Reason 4

No PV available.

Result:

Pending
Task-3 Learning Summary

Current Architecture:

PVC
 │
 ▼
PV
 │
 ▼
Storage

Current Status:

PVC Bound
PV Bound
Chapter 6 - Task-4 Use PVC in Pod

Now actual magic begins.

Until now:

PV Created
PVC Created

But no Pod using storage.

Goal

Prove:

Pod Delete

does NOT mean

Data Delete
Architecture
Pod
 │
 ▼
PVC
 │
 ▼
PV
 │
 ▼
hostPath
 │
 ▼
/tmp/k8s-pv-data
Pod YAML

File:

vi persistent-pod.yaml

Paste:

apiVersion: v1
kind: Pod

metadata:
  name: persistent-pod

spec:
  containers:
  - name: app
    image: busybox

    command:
    - sh
    - -c
    - |
      echo "Pod Started at $(date)" >> /data/message.txt
      sleep 3600

    volumeMounts:
    - name: data-volume
      mountPath: /data

  volumes:
  - name: data-volume
    persistentVolumeClaim:
      claimName: my-pvc
Deep YAML Explanation
claimName
claimName: my-pvc

Meaning:

Use PVC named my-pvc
persistentVolumeClaim
persistentVolumeClaim:

Means:

Mount PVC Storage
Complete Flow

When Pod starts:

Pod
 │
 ▼
Uses my-pvc
 │
 ▼
my-pvc uses manual-pv
 │
 ▼
manual-pv uses hostPath
 │
 ▼
/tmp/k8s-pv-data
Apply Pod
kubectl apply -f persistent-pod.yaml

Expected:

pod/persistent-pod created
Verify Pod
kubectl get pods

Expected:

persistent-pod Running
Check Data
kubectl exec -it persistent-pod -- cat /data/message.txt

Example:

Pod Started at Tue Jun 2 ...
Important Discovery You Made

You checked:

ls /tmp/k8s-pv-data

Nothing visible.

Why?

Because you are using:

KIND Cluster
KIND Architecture

Very Important.

Your node is actually:

Docker Container

Not actual Linux machine.

Real Architecture
Ubuntu Server
      │
      ▼
Docker
      │
      ▼
Kind Node Container
      │
      ▼
/tmp/k8s-pv-data
Verify Actual Data Location

Check Pod Node:

kubectl get pod persistent-pod -o wide

Example:

NODE:
praveen-cluster-worker

Enter Node Container:

docker exec -it praveen-cluster-worker bash

Check:

cat /tmp/k8s-pv-data/message.txt

Output:

Pod Started at Tue Jun 2 ...
Persistence Test

Delete Pod:

kubectl delete pod persistent-pod

Recreate:

kubectl apply -f persistent-pod.yaml

Check File Again

kubectl exec -it persistent-pod -- cat /data/message.txt

Expected:

Pod Started at First Time
Pod Started at Second Time
Why Two Lines?

Because:

First Pod wrote:

Line 1

Second Pod wrote:

Line 2

Storage survived.

Compare Task-1 vs Task-4
Task-1
emptyDir

Result:

Pod Delete
 ↓
Data Delete
Task-4
PVC
 ↓
PV

Result:

Pod Delete
 ↓
Data Safe
Task-4 Learning Summary

You proved:

Persistent Storage Works

Flow:

Pod
 ↓
PVC
 ↓
PV
 ↓
hostPath

Data survives Pod deletion.

Part-2 Revision Sheet

PVC:

Storage Request

PV:

Actual Storage

Binding Requirements:

StorageClass Match
AccessMode Match
Capacity Match

Task-3 Result:

PVC Bound
PV Bound

Task-4 Result:

Pod Delete
 ↓
Data Safe
Interview Questions
What is PVC?

Storage request made by application.

Why PVC required?

Provides abstraction between application and storage.

Can Pod directly use PV?

No.

Correct architecture:

Pod
 ↓
PVC
 ↓
PV
Why was PVC Pending initially?

StorageClass mismatch.

How does PV-PVC binding happen?

Using:

Capacity
Access Mode
StorageClass

matching.

Why did data survive in Task-4?

Because data was stored in PV, not inside container filesystem.

Why was hostPath not visible on Ubuntu host?

Because KIND nodes run inside Docker containers.

What is StorageClass?

Simple Definition:

StorageClass = Storage Blueprint

ya

Storage Template
Real World Example

Hotel Example

Developer:

Mujhe Room Chahiye

PVC:

Need Storage

Storage Team:

Kaunsa Room?

Standard
Premium
Luxury

StorageClass exactly ye information store karti hai.

StorageClass Responsibilities

StorageClass define karti hai:

1. Provisioner
2. Reclaim Policy
3. Volume Binding Mode
4. Expansion Rules
Check StorageClass

Command:

kubectl get storageclass

Your Output:

standard (default)

Describe:

kubectl describe storageclass standard

Output:

Provisioner:
rancher.io/local-path

ReclaimPolicy:
Delete

VolumeBindingMode:
WaitForFirstConsumer
StorageClass YAML (Concept Only)

Example:

apiVersion: storage.k8s.io/v1
kind: StorageClass

metadata:
  name: standard

provisioner: rancher.io/local-path

reclaimPolicy: Delete

volumeBindingMode: WaitForFirstConsumer
Deep YAML Explanation
kind
kind: StorageClass

Create StorageClass.

metadata.name
name: standard

StorageClass name.

Used inside PVC.

Example:

storageClassName: standard
provisioner
provisioner: rancher.io/local-path

Most important field.

reclaimPolicy
Delete

Delete storage when PVC deleted.

volumeBindingMode
WaitForFirstConsumer

Wait until Pod created.

Provisioner Deep Dive

Most Important Concept.

What is Provisioner?

Provisioner is a plugin.

Responsible for:

Creating Storage
Creating PV
Managing Storage
Real World Example

Developer:

Need 500Mi Storage

PVC created.

PVC cannot create storage itself.

StorageClass calls:

Provisioner

Provisioner creates storage.

Examples of Provisioners

AWS:

ebs.csi.aws.com

Creates:

AWS EBS Volume

Azure:

disk.csi.azure.com

Creates:

Azure Managed Disk

GCP:

pd.csi.storage.gke.io

Creates:

Persistent Disk

Your Cluster:

rancher.io/local-path

Creates:

Local Node Storage
Provisioner Workflow
PVC Created
     │
     ▼
StorageClass
     │
     ▼
Provisioner
     │
     ▼
PV Created

Remember this.

Interview favorite.

Reclaim Policy Deep Dive

Your StorageClass:

Delete
What Happens?

PVC Deleted

↓

PV Deleted

↓

Storage Deleted

Diagram:

PVC Delete
     │
     ▼
PV Delete
     │
     ▼
Data Delete
Compare with Manual PV

Manual PV:

Retain

Behavior:

PVC Delete
     │
     ▼
PV Released
     │
     ▼
Data Safe
Reclaim Policies Summary
Retain

Keep data.

Best for:

Databases
Critical Data
Delete

Delete everything.

Best for:

Testing
Temporary Environments
Recycle

Deprecated.

No longer used.

Interview only.

WaitForFirstConsumer Deep Dive

This confused many learners.

You saw:

waiting for first consumer

inside PVC events.

What is Consumer?

Consumer means:

Pod
Deployment
StatefulSet
DaemonSet

Anything using PVC.

Why Wait?

Imagine:

worker-1
worker-2
worker-3

PVC created.

Kubernetes doesn't know:

Which node will run Pod

If PV created too early:

Wrong Node

may be selected.

Therefore:

WaitForFirstConsumer

means:

Wait for Pod
Then create volume
Workflow
PVC Created
     │
     ▼
Pending
     │
     ▼
Pod Created
     │
     ▼
Node Selected
     │
     ▼
PV Created
     │
     ▼
PVC Bound
Task-5 Learning Summary

StorageClass contains:

Provisioner
Reclaim Policy
Binding Mode

Your StorageClass:

standard

Provisioner:

rancher.io/local-path

Reclaim Policy:

Delete

Binding Mode:

WaitForFirstConsumer
Chapter 8 - Dynamic Provisioning

Now we will remove manual PV creation.

Static Provisioning

Admin:

Creates PV

Developer:

Creates PVC

Architecture:

Admin
 │
 ▼
PV
 │
 ▼
PVC
 │
 ▼
Pod
Dynamic Provisioning

Developer:

Creates PVC

StorageClass:

Creates PV Automatically

Architecture:

PVC
 │
 ▼
StorageClass
 │
 ▼
Provisioner
 │
 ▼
PV
 │
 ▼
Pod
Task-6 Create Dynamic PVC

File:

vi dynamic-pvc.yaml

Paste:

apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: dynamic-pvc

spec:
  storageClassName: standard

  accessModes:
  - ReadWriteOnce

  resources:
    requests:
      storage: 200Mi
YAML Explanation
storageClassName
storageClassName: standard

Meaning:

Use StorageClass standard
accessModes
ReadWriteOnce

One node read/write.

requests.storage
200Mi

Need 200Mi storage.

Apply PVC
kubectl apply -f dynamic-pvc.yaml
Check PVC
kubectl get pvc

Initially:

Pending

Why?

Because:

WaitForFirstConsumer
Verify Events
kubectl describe pvc dynamic-pvc

Output:

waiting for first consumer
Why PVC Pending?

No Pod using PVC yet.

Consumer missing.

Create Dynamic Pod

File:

vi dynamic-pod.yaml

Paste:

apiVersion: v1
kind: Pod

metadata:
  name: dynamic-pod

spec:
  containers:
  - name: app
    image: busybox

    command:
    - sh
    - -c
    - |
      echo "Dynamic PV Test $(date)" >> /data/test.txt
      sleep 3600

    volumeMounts:
    - name: dynamic-storage
      mountPath: /data

  volumes:
  - name: dynamic-storage
    persistentVolumeClaim:
      claimName: dynamic-pvc
YAML Explanation
claimName
claimName: dynamic-pvc

Use dynamic-pvc.

mountPath
/data

Storage available at:

/data
Apply Pod
kubectl apply -f dynamic-pod.yaml
What Happens Internally?

Step-1

Pod Created

Step-2

Consumer Found

Step-3

StorageClass Activated

Step-4

Provisioner Runs

Step-5

PV Created Automatically

Step-6

PVC Bound

Step-7

Pod Running
Verify

Check PVC:

kubectl get pvc

Expected:

dynamic-pvc Bound

Check PV:

kubectl get pv

Expected:

pvc-xxxxxxxx

Auto-created PV.

Why Strange Name?

Manual PV:

manual-pv

You named it.

Dynamic PV:

pvc-001159ed...

Kubernetes generated name.

Verify Data
kubectl exec -it dynamic-pod -- cat /data/test.txt

Example:

Dynamic PV Test Tue Jun 2 ...
Static vs Dynamic Provisioning
Feature	Static	Dynamic
PV Creation	Manual	Automatic
Admin Work	High	Low
StorageClass	Optional	Required
Provisioner	Not Needed	Needed
Scaling	Difficult	Easy
Production Usage	Rare	Common
Task-6 Learning Summary

You proved:

PVC
 ↓
StorageClass
 ↓
Provisioner
 ↓
PV Auto Created
Part-3 Revision Sheet

StorageClass:

Storage Blueprint

Provisioner:

Storage Plugin

WaitForFirstConsumer:

Wait For Pod
Then Create Volume

Dynamic Provisioning:

PVC
 ↓
PV Auto Created
Interview Questions
What is StorageClass?

Storage blueprint used for dynamic provisioning.

What is Provisioner?

Plugin that creates storage automatically.

Why was dynamic-pvc Pending?

WaitForFirstConsumer.

No Pod existed yet.

What is Consumer?

Pod, Deployment, StatefulSet or DaemonSet using PVC.

Difference between Static and Dynamic Provisioning?

Static:

Admin creates PV.

Dynamic:

StorageClass creates PV automatically.

Why did dynamic PV have a random name?

Because Kubernetes generated it automatically.

What was your default StorageClass?
standard
What was the provisioner?
rancher.io/local-path
What was the reclaim policy?
Delete
What was the volume binding mode?
WaitForFirstConsumer

Step-1 Delete Pods

Commands:

kubectl delete pod persistent-pod
kubectl delete pod dynamic-pod

Verify:

kubectl get pods

Expected:

No resources found
Why Delete Pods First?

Because:

Pod

is currently using:

PVC

Good practice:

Delete Consumers First

Then:

Delete Storage
Step-2 Delete PVCs

Commands:

kubectl delete pvc my-pvc
kubectl delete pvc dynamic-pvc

Expected:

persistentvolumeclaim "my-pvc" deleted
persistentvolumeclaim "dynamic-pvc" deleted
Step-3 Observe PVs

Command:

kubectl get pv
Expected Result

Manual PV:

manual-pv
Released

Dynamic PV:

Deleted Automatically

May not appear at all.

Why Manual PV Survived?

Because:

persistentVolumeReclaimPolicy: Retain

Behavior:

PVC Deleted
     ↓
PV Remains
     ↓
Data Remains
Why Dynamic PV Disappeared?

StorageClass:

ReclaimPolicy: Delete

Behavior:

PVC Deleted
     ↓
PV Deleted
     ↓
Storage Deleted
Visual Comparison
Retain
PVC Deleted
     │
     ▼
PV Released
     │
     ▼
Data Safe
Delete
PVC Deleted
     │
     ▼
PV Deleted
     │
     ▼
Data Deleted
Step-4 Delete Remaining PV

Command:

kubectl delete pv manual-pv

Verify:

kubectl get pv

Expected:

No resources found
Important PV States
Available

Meaning:

PV Exists
No PVC Attached

Example:

manual-pv
Available
Bound

Meaning:

PV Attached To PVC

Example:

manual-pv
Bound
Released

Meaning:

PVC Deleted
PV Still Exists

Example:

manual-pv
Released
Failed

Rare state.

Something went wrong during provisioning.

Chapter 10 - Most Common Storage Problems

Very Important for Interviews.

Problem-1 PVC Pending

Command:

kubectl get pvc

Output:

Pending
Cause-1 StorageClass Mismatch

PV:

StorageClass=""

PVC:

StorageClass=standard

Result:

Pending
Cause-2 Capacity Mismatch

PV:

1Gi

PVC:

2Gi

Result:

Pending
Cause-3 Access Mode Mismatch

PV:

RWO

PVC:

RWX

Result:

Pending
Cause-4 No Matching PV

No suitable PV exists.

Result:

Pending
Cause-5 WaitForFirstConsumer

PVC exists.

Pod does not exist.

Result:

Pending
Troubleshooting Command

Always run:

kubectl describe pvc <pvc-name>

Example:

kubectl describe pvc dynamic-pvc
Problem-2 Pod Stuck Pending

Check:

kubectl describe pod <pod-name>

Look at:

Events Section
Problem-3 Data Not Visible

Check:

kubectl exec -it pod-name -- ls /data

Check mount:

kubectl describe pod pod-name
Chapter 11 - Production Notes
Why hostPath is Bad for Production?

Example:

worker-1

contains:

/tmp/k8s-pv-data

Pod moves to:

worker-2

Data unavailable.

Production Storage Options

AWS:

EBS
EFS

Azure:

Azure Disk
Azure Files

GCP:

Persistent Disk
Filestore

On-Premise:

NFS
Ceph
Longhorn
OpenEBS
Why Databases Usually Use Retain?

Imagine:

Production PostgreSQL

PVC accidentally deleted.

With:

Delete

Everything lost.

With:

Retain

Storage survives.

Recovery possible.

Stateful Applications

Common Examples:

MySQL
PostgreSQL
MongoDB
Elasticsearch
Redis
Kafka

All need persistent storage.

Chapter 12 - Static vs Dynamic Provisioning
Static Provisioning

Admin creates PV.

Developer creates PVC.

Architecture:

Admin
 │
 ▼
PV
 │
 ▼
PVC
 │
 ▼
Pod

Advantages:

Simple
Full Control

Disadvantages:

Manual Work
Not Scalable
Dynamic Provisioning

Developer creates PVC.

StorageClass creates PV automatically.

Architecture:

PVC
 │
 ▼
StorageClass
 │
 ▼
Provisioner
 │
 ▼
PV

Advantages:

Automatic
Scalable
Production Friendly

Disadvantages:

Requires StorageClass
Requires Provisioner
Complete Flow Revision
Task-1
Pod
 ↓
emptyDir

Result:

Pod Delete
 ↓
Data Delete
Task-2
Create PV

Result:

Available
Task-3
Create PVC

Result:

Bound
Task-4
Pod
 ↓
PVC
 ↓
PV

Result:

Data Survives
Task-5

Learned:

StorageClass
Provisioner
WaitForFirstConsumer
Task-6

Learned:

Dynamic Provisioning

Result:

PVC
 ↓
PV Auto Created
Task-7

Learned:

Retain
vs
Delete
30 Detailed Interview Questions & Answers
1. What is a Volume?

Storage attached to a Pod.

2. What is emptyDir?

Temporary volume deleted with Pod.

3. What is hostPath?

Node directory mounted into Pod.

4. What is PV?

Cluster-wide storage resource.

5. What is PVC?

Storage request made by application.

6. Difference between PV and PVC?

PV = Actual storage.

PVC = Request for storage.

7. Why PVC required?

Provides abstraction between application and storage.

8. Can Pod directly use PV?

No.

Must use PVC.

9. What is StorageClass?

Storage blueprint used for dynamic provisioning.

10. What is Provisioner?

Plugin that creates storage automatically.

11. What is Dynamic Provisioning?

Automatic PV creation using StorageClass.

12. What is Static Provisioning?

Manual PV creation by admin.

13. What is RWO?

ReadWriteOnce.

One node read/write.

14. What is ROX?

ReadOnlyMany.

Many nodes read-only.

15. What is RWX?

ReadWriteMany.

Many nodes read/write.

16. What is Retain?

Keep PV and data after PVC deletion.

17. What is Delete?

Delete PV and data after PVC deletion.

18. What is Recycle?

Deprecated reclaim policy.

19. What is WaitForFirstConsumer?

Wait for Pod scheduling before volume provisioning.

20. Why was dynamic PVC Pending?

No consumer existed yet.

21. What is Consumer?

Pod/Deployment/StatefulSet using PVC.

22. Why was PVC Pending in Task-3?

StorageClass mismatch.

23. What causes PVC Pending?

Capacity mismatch, access mode mismatch, StorageClass mismatch.

24. What are PV States?

Available, Bound, Released.

25. Why use Retain for databases?

Protects data from accidental deletion.

26. Why avoid hostPath in production?

Node-dependent storage.

27. What happens if Pod restarts with PVC?

Data survives.

28. What happens if Pod restarts with emptyDir?

Data lost.

29. What is the default StorageClass in your cluster?
standard
30. What provisioner did your cluster use?
rancher.io/local-path
Final Revision Sheet (Read Before Interview)

Remember:

PV = Actual Storage
PVC = Storage Request
StorageClass = Storage Blueprint
Provisioner = Storage Plugin
emptyDir

Result:

Pod Delete
 ↓
Data Delete
PVC + PV

Result:

Pod Delete
 ↓
Data Safe

PV Lifecycle:

Available
   ↓
Bound
   ↓
Released

Dynamic Provisioning:

PVC
 ↓
StorageClass
 ↓
Provisioner
 ↓
PV

Reclaim Policies:

Retain = Keep Data

Delete = Remove Data


