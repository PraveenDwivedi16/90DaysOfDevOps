Day 55 - Persistent Volumes (PV) and Persistent Volume Claims (PVC)
Objective
Learn why containers need persistent storage and how Kubernetes provides it using:
Persistent Volumes (PV)
Persistent Volume Claims (PVC)
StorageClasses
Static Provisioning
Dynamic Provisioning
---
Why Containers Need Persistent Storage
Containers are ephemeral.
Example:
```text
Pod Deleted
    ↓
Container Deleted
    ↓
Data Deleted
```
This is a problem for:
MySQL
PostgreSQL
MongoDB
Redis persistence
Business applications
Solution:
```text
Pod
 ↓
PVC
 ↓
PV
 ↓
Storage
```
---
Kubernetes Storage Architecture
```text
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
 Actual Storage
```
---
Important Volume Types
emptyDir
Temporary storage.
```yaml
volumes:
- name: temp-storage
  emptyDir: {}
```
Behavior:
```text
Pod Delete
   ↓
Data Delete
```
Use Cases:
Cache
Temporary files
Scratch space
---
hostPath
Mounts a folder from the Kubernetes node.
```yaml
hostPath:
  path: /tmp/k8s-pv-data
```
Good for learning.
Not recommended in production.
---
ConfigMap Volume
Used to mount configuration files.
Examples:
nginx.conf
application.properties
---
Secret Volume
Used to mount:
Passwords
API Keys
Tokens
Certificates
---
NFS Volume
Shared storage.
Supports:
```text
RWX
(ReadWriteMany)
```
---
CSI Volumes
Production storage.
Examples:
AWS EBS
Azure Disk
GCP Persistent Disk
Longhorn
Ceph
---
Task 1 - Data Loss Demonstration
Goal
Show that emptyDir loses data after Pod recreation.
Manifest
```yaml
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
```
volumes vs volumeMounts
volumes
Creates storage.
```yaml
volumes:
- name: temp-storage
  emptyDir: {}
```
volumeMounts
Attaches storage inside container.
```yaml
volumeMounts:
- name: temp-storage
  mountPath: /data
```
Flow:
```text
temp-storage
      │
      ▼
    /data
```
Result:
```text
Pod Delete
   ↓
emptyDir Delete
   ↓
Data Lost
```
---
Persistent Volume (PV)
Definition
PV = Actual Storage
Cluster-wide resource.
Example:
```text
1Gi
10Gi
100Gi
```
Check:
```bash
kubectl get pv
```
---
PV Lifecycle
```text
Available
   ↓
Bound
   ↓
Released
```
Available
No PVC attached.
Bound
PVC attached.
Released
PVC deleted but PV still exists.
---
Task 2 - Create a PV
Create directory:
```bash
mkdir -p /tmp/k8s-pv-data
```
PV Manifest:
```yaml
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
```
Verify:
```bash
kubectl get pv
kubectl describe pv manual-pv
```
Expected:
```text
STATUS = Available
```
---
Access Modes
ReadWriteOnce (RWO)
One node can read and write.
Most common.
Used by:
MySQL
PostgreSQL
MongoDB
---
ReadOnlyMany (ROX)
Many nodes can read.
Read-only.
---
ReadWriteMany (RWX)
Many nodes can read and write.
Used by:
NFS
Shared storage
---
Reclaim Policies
Retain
```yaml
persistentVolumeReclaimPolicy: Retain
```
Behavior:
```text
PVC Delete
   ↓
PV Released
   ↓
Data Safe
```
Use For:
Databases
Critical business data
---
Delete
```yaml
persistentVolumeReclaimPolicy: Delete
```
Behavior:
```text
PVC Delete
   ↓
PV Delete
   ↓
Data Delete
```
Use For:
Testing
Temporary environments
---
Recycle (Deprecated)
Old reclaim policy.
No longer recommended.
---
Persistent Volume Claim (PVC)
Definition
PVC = Request for Storage
Example:
```text
Need 500Mi Storage
```
---
Task 3 - Create PVC
```yaml
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
```
Why storageClassName: "" ?
```text
Disable Dynamic Provisioning
Use existing PV only
```
Verify:
```bash
kubectl get pvc
kubectl get pv
```
Expected:
```text
PVC = Bound
PV  = Bound
```
---
How Binding Works
PV:
```text
1Gi
RWO
```
PVC:
```text
500Mi
RWO
```
Matching:
```text
1Gi >= 500Mi
RWO = RWO
```
Result:
```text
Bound
```
---
Task 4 - Use PVC Inside Pod
```yaml
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
```
Architecture:
```text
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
```
Result:
```text
Pod Delete
   ↓
Data Safe
```
---
KIND Cluster Important Note
In KIND:
```text
hostPath
```
Exists inside node container.
Not directly on host machine.
Example:
```bash
docker exec -it praveen-cluster-worker bash
```
Then:
```bash
cat /tmp/k8s-pv-data/message.txt
```
---
StorageClass
Definition
StorageClass = Storage Blueprint / Template
Defines:
Provisioner
Reclaim Policy
Volume Binding Mode
Expansion Rules
---
Your StorageClass
```text
standard (default)
```
Provisioner:
```text
rancher.io/local-path
```
Reclaim Policy:
```text
Delete
```
VolumeBindingMode:
```text
WaitForFirstConsumer
```
---
Provisioner
Provisioner is a storage plugin.
Creates storage automatically.
Examples:
AWS:
```text
ebs.csi.aws.com
```
Azure:
```text
disk.csi.azure.com
```
Rancher:
```text
rancher.io/local-path
```
---
WaitForFirstConsumer
Meaning:
```text
Do not create PV immediately.
Wait for Pod scheduling first.
```
Flow:
```text
PVC Created
     ↓
Wait
     ↓
Pod Created
     ↓
PV Created
```
---
Static Provisioning
Admin creates PV.
Developer creates PVC.
```text
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
```
---
Dynamic Provisioning
Developer creates PVC only.
StorageClass creates PV automatically.
```text
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
```
---
Task 6 - Dynamic Provisioning
PVC:
```yaml
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
```
Pod:
```yaml
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
```
Result:
```text
PVC Created
   ↓
PV Auto Created
```
---
Static vs Dynamic Provisioning
Feature	Static	Dynamic
PV Creation	Manual	Automatic
Admin Work	High	Low
StorageClass	Optional	Required
Scaling	Difficult	Easy
Production	Less Common	Common
---
Task 7 - Cleanup
Delete Pods:
```bash
kubectl delete pod persistent-pod
kubectl delete pod dynamic-pod
```
Delete PVCs:
```bash
kubectl delete pvc my-pvc
kubectl delete pvc dynamic-pvc
```
Check PVs:
```bash
kubectl get pv
```
Expected:
Manual PV:
```text
Released
```
Dynamic PV:
```text
Deleted Automatically
```
---
Common PVC Pending Reasons
StorageClass mismatch
Capacity mismatch
AccessMode mismatch
No matching PV
WaitForFirstConsumer
---
Production Recommendations
Avoid:
hostPath
Prefer:
AWS EBS
Azure Disk
GCP PD
Longhorn
Ceph
NFS
---
Interview Questions and Answers
What is PV?
Actual storage resource in Kubernetes.
---
What is PVC?
Storage request made by applications.
---
Difference between PV and PVC?
PV = Storage
PVC = Request for Storage
---
What is StorageClass?
Template used for dynamic provisioning.
---
What is Provisioner?
Plugin that creates storage automatically.
---
What is RWO?
One node can read and write.
---
What is RWX?
Many nodes can read and write.
---
What is ROX?
Many nodes can read only.
---
What is Retain Policy?
Keep PV and data after PVC deletion.
---
What is Delete Policy?
Delete PV and data after PVC deletion.
---
What is WaitForFirstConsumer?
Wait until Pod is scheduled before creating/binding volume.
---
Why was PVC Pending in the lab?
StorageClass mismatch and later WaitForFirstConsumer behavior.
---
Why use PVC instead of directly using PV?
PVC decouples applications from storage implementation.
---
Why is hostPath not recommended in production?
Data is tied to a single node.

What are PV States?
Available
Bound
Released
