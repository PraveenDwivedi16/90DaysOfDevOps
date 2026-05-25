# Kind Kubernetes Practice Setup on Ubuntu 24.04

This guide helps you set up a local Kubernetes practice environment on **Ubuntu 24.04** using:

- Docker Engine
- kubectl
- kind
- A first test Kubernetes cluster
- A sample Nginx deployment

Official references:

- Docker Engine Ubuntu install: https://docs.docker.com/engine/install/ubuntu/
- kubectl Linux install: https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/
- kind quick start: https://kind.sigs.k8s.io/docs/user/quick-start/

## Step 1: Update Ubuntu System

Run:
# sudo apt update
# sudo apt upgrade -y

Install common required packages:

sudo apt install -y curl wget apt-transport-https ca-certificates gnupg lsb-release

Verify:

# lsb_release -a

Expected Ubuntu version:
Ubuntu 24.04


## Step 2: Remove Old Docker Packages

Before installing Docker from the official Docker repository, remove old or conflicting packages:

sudo apt remove -y docker.io docker-compose docker-compose-v2 docker-doc podman-docker containerd runc

This is safe if these packages are not installed.

## Step 3: Add Docker Official GPG Key

Create the keyrings directory:

sudo install -m 0755 -d /etc/apt/keyrings

Download Docker GPG key:

sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  -o /etc/apt/keyrings/docker.asc


Set permission:

sudo chmod a+r /etc/apt/keyrings/docker.asc

## Step 4: Add Docker Repository

Add Docker's official Ubuntu repository:

sudo tee /etc/apt/sources.list.d/docker.sources <<EOF_DOCKER
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF_DOCKER

Update package list:

sudo apt update

## Step 5: Install Docker Engine

Install Docker Engine, Docker CLI, containerd, Buildx, and Docker Compose plugin:

sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin


Enable and start Docker service:

sudo systemctl enable docker
sudo systemctl start docker

Verify Docker service:

sudo systemctl status docker

Press `q` to exit the status screen.

## Step 6: Test Docker Installation

Run Docker hello-world container:

sudo docker run hello-world

If Docker is installed correctly, you should see a success message.

Check Docker version:

sudo docker version

## Step 7: Run Docker Without sudo

Add your current user to the Docker group:

sudo usermod -aG docker $USER

Apply the new group without logout:

newgrp docker

Verify Docker without sudo:

docker ps

If this works without permission error, Docker is ready.

If you still get permission errors, logout and login again, then run:

docker ps

## Step 8: Install kubectl

Create Kubernetes keyring directory:

sudo mkdir -p -m 755 /etc/apt/keyrings

Download Kubernetes signing key:

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.36/deb/Release.key \
  | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg


Set key permission:

sudo chmod 644 /etc/apt/keyrings/kubernetes-apt-keyring.gpg


Add Kubernetes apt repository:

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.36/deb/ /' \
  | sudo tee /etc/apt/sources.list.d/kubernetes.list


Update package list:

sudo apt update

Install kubectl:

sudo apt install -y kubectl


Verify kubectl:

kubectl version --client

## Step 9: Install kind

For most Ubuntu laptops/desktops using Intel or AMD processor, use amd64:

curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.31.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind

For ARM64 machine only, use this instead:

curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.31.0/kind-linux-arm64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind

Verify kind:
kind version
## Step 10: Create First kind Cluster

Create a local Kubernetes cluster:

kind create cluster --name practice-cluster

# Verify cluster list:
kind get clusters

# Expected output:
practice-cluster

# Verify Kubernetes cluster:
kubectl cluster-info

# Check nodes:
kubectl get nodes

Expected output example:
NAME                             STATUS   ROLES           AGE   VERSION
practice-cluster-control-plane   Ready    control-plane   1m    v1.xx.x

## Step 11: Create a Test Nginx Deployment

Create an Nginx deployment:

kubectl create deployment nginx-demo --image=nginx

Check deployment:

kubectl get deployments

Check pods:
kubectl get pods
Expected pod status:
Running

## Step 12: Expose Nginx Service

Expose the deployment using NodePort:

kubectl expose deployment nginx-demo --type=NodePort --port=80

Check services:

kubectl get svc

## Step 13: Access Nginx in Browser

Use port-forwarding:

kubectl port-forward service/nginx-demo 8080:80


Keep this terminal running.

Open this URL in browser:
http://localhost:8080

You should see the Nginx welcome page.

## Step 14: Useful kubectl Commands for Practice

# List nodes:
kubectl get nodes

# List pods:
kubectl get pods

# List services:
kubectl get svc

# List deployments:
kubectl get deployments


# Describe a pod:
kubectl describe pod <pod-name>

# View pod logs:
kubectl logs <pod-name>

# Delete deployment:
kubectl delete deployment nginx-demo

# Delete service:
kubectl delete service nginx-demo

## Step 15: Useful kind Commands

# List kind clusters:
kind get clusters

# Delete the practice cluster:
kind delete cluster --name practice-cluster

# Create the cluster again:
kind create cluster --name practice-cluster


## Step 16: Optional Multi-Node kind Cluster

Create a config file:
vim  kind-multi-node.yaml

Add this content:
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker

Create cluster using config:
kind create cluster --name multi-node-cluster --config kind-multi-node.yaml

Verify nodes:

kubectl get nodes

Expected: 1 control-plane node and 2 worker nodes.

# Delete multi-node cluster:

kind delete cluster --name multi-node-cluster

## Step 17: Basic YAML Practice

Create a file:

vim  nginx-deployment.yaml

# Add this YAML:
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-yaml-demo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-yaml-demo
  template:
    metadata:
      labels:
        app: nginx-yaml-demo
    spec:
      containers:
        - name: nginx
          image: nginx
          ports:
            - containerPort: 80

Apply it:
kubectl apply -f nginx-deployment.yaml
Check pods:
kubectl get pods

Delete it:
kubectl delete -f nginx-deployment.yaml
