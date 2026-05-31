# Day 54 – Kubernetes ConfigMaps and Secrets

## Goal of the Day

In real applications, we should not hardcode configuration values inside the application code or Docker image.

Examples of configuration values:

```text
APP_ENV=production
APP_DEBUG=false
APP_PORT=8080
DB_USER=admin
DB_PASSWORD=s3cureP@ssw0rd
API_KEY=abcd1234
FEATURE_FLAG=true
```

If these values are hardcoded inside the container image, then every time a value changes, we need to rebuild and redeploy the image.

Kubernetes solves this using:

```text
ConfigMap -> for non-sensitive configuration
Secret    -> for sensitive configuration
```

---

## What is a ConfigMap?

A ConfigMap is a Kubernetes object used to store **non-sensitive configuration data**.

Use ConfigMap for:

```text
APP_ENV
APP_DEBUG
APP_PORT
LOG_LEVEL
FEATURE_FLAGS
Nginx configuration files
Application configuration files
```

### Important

ConfigMap values are stored as plain text.

So do not store passwords, tokens, API keys, or private credentials inside a ConfigMap.

Bad example:

```yaml
data:
  DB_PASSWORD: "my-secret-password"
```

Good example:

```yaml
data:
  APP_ENV: "production"
  APP_PORT: "8080"
```

---

## What is a Secret?

A Secret is a Kubernetes object used to store **sensitive configuration data**.

Use Secret for:

```text
DB_USER
DB_PASSWORD
API_KEY
JWT_SECRET
TLS certificates
Docker registry credentials
```

### Very Important

Kubernetes Secrets are base64 encoded by default.

Base64 is **not encryption**.

Anyone who has permission to read the Secret can decode the value.

Example:

```bash
echo 'czNjdXJlUEBzc3cwcmQ=' | base64 --decode
```

Output:

```text
s3cureP@ssw0rd
```

So Secret security depends on:

```text
RBAC permissions
Encryption at rest
Who can run kubectl get secret
Who can exec into Pods
Secret management process
```

---

## ConfigMap vs Secret

| Topic | ConfigMap | Secret |
|---|---|---|
| Purpose | Non-sensitive configuration | Sensitive data |
| Example | APP_ENV, APP_PORT | DB_PASSWORD, API_KEY |
| Stored format | Plain text | Base64 encoded |
| Encrypted by default? | No | No, base64 only |
| Good for GitHub? | Usually yes | No, not in public repositories |
| Pod usage | Env vars or volume | Env vars or volume |

---

## How Pods Consume ConfigMaps and Secrets

Pods can consume ConfigMaps and Secrets in two common ways.

---

### 1. As Environment Variables

Use this for simple key-value settings.

Example:

```yaml
envFrom:
- configMapRef:
    name: app-config
```

Or for a single Secret key:

```yaml
env:
- name: DB_USER
  valueFrom:
    secretKeyRef:
      name: db-credentials
      key: DB_USER
```

### Important

Environment variables are set only when the Pod starts.

If the ConfigMap or Secret is updated later, the environment variables inside the running Pod do **not** update automatically.

To get updated env values, we must restart or recreate the Pod.

---

### 2. As Volume Mounts

Use this when you need full configuration files or secret files.

Example:

```yaml
volumeMounts:
- name: nginx-config-volume
  mountPath: /etc/nginx/conf.d

volumes:
- name: nginx-config-volume
  configMap:
    name: nginx-config
```

When a ConfigMap or Secret is mounted as a volume, each key becomes a file.

Example:

```text
ConfigMap key: default.conf
Mounted file: /etc/nginx/conf.d/default.conf
```

### Important

Volume-mounted ConfigMaps and Secrets can update automatically after some delay.

Usually it can take 30-60 seconds.

---

# Task 1 – Create a ConfigMap from Literals

## Goal

Create a ConfigMap named:

```text
app-config
```

With these values:

```text
APP_ENV=production
APP_DEBUG=false
APP_PORT=8080
```

The task mentions `kubectl create configmap --from-literal`, but for real DevOps and GitHub practice, YAML is better.

We used YAML-first approach.

---

## File: `app-config.yaml`

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: "production"
  APP_DEBUG: "false"
  APP_PORT: "8080"
```

---

## YAML Explanation

### `apiVersion: v1`

```yaml
apiVersion: v1
```

ConfigMap is a core Kubernetes resource, so it uses API version `v1`.

### `kind: ConfigMap`

```yaml
kind: ConfigMap
```

This tells Kubernetes that we are creating a ConfigMap.

### `metadata.name`

```yaml
metadata:
  name: app-config
```

The ConfigMap name is `app-config`.

Pods will refer to this ConfigMap using this name.

### `data`

```yaml
data:
  APP_ENV: "production"
  APP_DEBUG: "false"
  APP_PORT: "8080"
```

This is the actual configuration data.

All values are stored as strings.

### Important

ConfigMap values are plain text.

They are not base64 encoded and not encrypted.

---

## Commands

```bash
kubectl apply -f app-config.yaml
```

Verify:

```bash
kubectl describe configmap app-config
```

```bash
kubectl get configmap app-config -o yaml
```

---

## Expected Output

```text
APP_DEBUG:
----
false

APP_ENV:
----
production

APP_PORT:
----
8080
```

YAML output:

```yaml
data:
  APP_DEBUG: "false"
  APP_ENV: production
  APP_PORT: "8080"
```

---

## Second Option: Command-Based Method

This also works:

```bash
kubectl create configmap app-config \
  --from-literal=APP_ENV=production \
  --from-literal=APP_DEBUG=false \
  --from-literal=APP_PORT=8080
```

But this is an imperative method.

For GitHub and CI/CD, YAML is better.

---

# Task 2 – Create a ConfigMap from a File

## Goal

Create a ConfigMap named:

```text
nginx-config
```

It should contain an Nginx config file named:

```text
default.conf
```

The Nginx config should expose a `/health` endpoint returning:

```text
healthy
```

---

## Why File-Based ConfigMap?

Task 1 used simple key-value settings.

Task 2 stores a full configuration file inside a ConfigMap.

Example:

```yaml
data:
  default.conf: |
    server {
        listen 80;

        location /health {
            return 200 "healthy\n";
        }
    }
```

Here:

```text
default.conf = key name
server { ... } = file content
```

When this ConfigMap is mounted as a volume, Kubernetes creates a file using the key name.

```text
ConfigMap key: default.conf
Mounted file: /etc/nginx/conf.d/default.conf
```

---

## Why `/etc/nginx/conf.d`?

The official Nginx Docker image reads config files from:

```text
/etc/nginx/conf.d/*.conf
```

So we mount our ConfigMap at:

```text
/etc/nginx/conf.d
```

Then Kubernetes creates:

```text
/etc/nginx/conf.d/default.conf
```

Nginx reads this config and serves `/health`.

---

## File: `nginx-config.yaml`

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  default.conf: |
    server {
        listen 80;

        location / {
            return 200 "Welcome from custom nginx config\n";
        }

        location /health {
            return 200 "healthy\n";
        }
    }
```

---

## YAML Explanation

### `metadata.name`

```yaml
metadata:
  name: nginx-config
```

ConfigMap name is `nginx-config`.

### `default.conf: |`

```yaml
default.conf: |
```

This is the most important line.

`default.conf` is the key.

When mounted into a Pod, this key becomes a file.

The `|` means multiline text block in YAML.

### Nginx Root Location

```nginx
location / {
    return 200 "Welcome from custom nginx config\n";
}
```

This confirms that the custom Nginx config is loaded.

### Nginx Health Endpoint

```nginx
location /health {
    return 200 "healthy\n";
}
```

This returns:

```text
healthy
```

This is useful for:

```text
Load balancer health checks
Kubernetes probes
Monitoring tools
```

---

## Commands

```bash
kubectl apply -f nginx-config.yaml
```

Verify:

```bash
kubectl get configmap nginx-config -o yaml
```

```bash
kubectl describe configmap nginx-config
```

---

## Expected Output

```text
default.conf:
----
server {
    listen 80;

    location / {
        return 200 "Welcome from custom nginx config\n";
    }

    location /health {
        return 200 "healthy\n";
    }
}
```

---

## Second Option: Command-Based Method

First create a local file:

```bash
cat > custom-nginx.conf <<'EOF'
server {
    listen 80;

    location / {
        return 200 "Welcome from custom nginx config\n";
    }

    location /health {
        return 200 "healthy\n";
    }
}
EOF
```

Then create the ConfigMap:

```bash
kubectl create configmap nginx-config --from-file=default.conf=custom-nginx.conf
```

Meaning:

```text
default.conf = key name inside ConfigMap
custom-nginx.conf = local file name
```

If the ConfigMap already exists, this command fails.

```text
configmaps "nginx-config" already exists
```

That is why `kubectl apply -f nginx-config.yaml` is better.

---

# Task 3 – Use ConfigMaps in Pods

Task 3 has two parts.

```text
Task 3A: Use app-config as environment variables
Task 3B: Mount nginx-config as a volume
```

---

# Task 3A – ConfigMap as Environment Variables

## Goal

Create a BusyBox Pod that reads all keys from `app-config` as environment variables.

Expected values inside Pod:

```text
APP_ENV=production
APP_DEBUG=false
APP_PORT=8080
```

---

## File: `config-env-pod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-env-pod
spec:
  restartPolicy: Never
  containers:
  - name: busybox
    image: busybox:latest
    command:
    - sh
    - -c
    - |
      echo "APP_ENV=$APP_ENV"
      echo "APP_DEBUG=$APP_DEBUG"
      echo "APP_PORT=$APP_PORT"
    envFrom:
    - configMapRef:
        name: app-config
```

---

## YAML Explanation

### `restartPolicy: Never`

```yaml
restartPolicy: Never
```

The Pod runs the command once and exits.

This is why the Pod status becomes `Completed`.

### `envFrom`

```yaml
envFrom:
- configMapRef:
    name: app-config
```

This injects all keys from `app-config` as environment variables.

So:

```text
APP_ENV -> environment variable
APP_DEBUG -> environment variable
APP_PORT -> environment variable
```

### Command

```yaml
command:
- sh
- -c
- |
  echo "APP_ENV=$APP_ENV"
  echo "APP_DEBUG=$APP_DEBUG"
  echo "APP_PORT=$APP_PORT"
```

This prints the environment variable values.

---

## Commands

```bash
kubectl apply -f config-env-pod.yaml
```

Check Pod:

```bash
kubectl get pod config-env-pod
```

Check logs:

```bash
kubectl logs config-env-pod
```

---

## Expected Output

```text
APP_ENV=production
APP_DEBUG=false
APP_PORT=8080
```

### Note

If logs are checked too quickly, this may appear:

```text
container "busybox" in pod "config-env-pod" is waiting to start: ContainerCreating
```

This is not an issue.

Wait a few seconds and run logs again.

---

# Task 3B – ConfigMap as Volume Mount

## Goal

Create an Nginx Pod and mount `nginx-config` as a volume at:

```text
/etc/nginx/conf.d
```

Then verify:

```text
/health -> healthy
```

---

## File: `nginx-config-pod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-config-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.25
    ports:
    - containerPort: 80
    volumeMounts:
    - name: nginx-config-volume
      mountPath: /etc/nginx/conf.d
  volumes:
  - name: nginx-config-volume
    configMap:
      name: nginx-config
```

---

## YAML Explanation

### Nginx Container

```yaml
containers:
- name: nginx
  image: nginx:1.25
```

Runs an Nginx container.

### Container Port

```yaml
ports:
- containerPort: 80
```

Nginx listens on port 80.

### Volume Mount

```yaml
volumeMounts:
- name: nginx-config-volume
  mountPath: /etc/nginx/conf.d
```

Mounts the ConfigMap volume inside the container at:

```text
/etc/nginx/conf.d
```

### Volume Source

```yaml
volumes:
- name: nginx-config-volume
  configMap:
    name: nginx-config
```

The volume source is the `nginx-config` ConfigMap.

### Important

These names must match:

```yaml
volumeMounts:
- name: nginx-config-volume
```

```yaml
volumes:
- name: nginx-config-volume
```

If the names do not match, the Pod will fail.

---

## Commands

```bash
kubectl apply -f nginx-config-pod.yaml
```

Check Pod:

```bash
kubectl get pod nginx-config-pod
```

Expected:

```text
nginx-config-pod   1/1   Running
```

---

## Verify Mounted File

In Git Bash on Windows, use `MSYS_NO_PATHCONV=1` to avoid path conversion issues.

```bash
MSYS_NO_PATHCONV=1 kubectl exec nginx-config-pod -- ls -l /etc/nginx/conf.d
```

Expected output:

```text
default.conf -> ..data/default.conf
```

This symlink is normal.

Kubernetes mounts ConfigMap files using symlinks.

Now check file content:

```bash
MSYS_NO_PATHCONV=1 kubectl exec nginx-config-pod -- cat /etc/nginx/conf.d/default.conf
```

Expected:

```nginx
location /health {
    return 200 "healthy\n";
}
```

---

## Git Bash Path Conversion Issue

This command may fail in Git Bash:

```bash
kubectl exec nginx-config-pod -- ls -l /etc/nginx/conf.d
```

Error:

```text
ls: cannot access 'C:/Program Files/Git/etc/nginx/conf.d': No such file or directory
```

Reason:

Git Bash converts Linux paths to Windows paths.

Fix:

```bash
MSYS_NO_PATHCONV=1 kubectl exec nginx-config-pod -- ls -l /etc/nginx/conf.d
```

Use this rule:

```text
When using kubectl exec with Linux paths in Git Bash, use MSYS_NO_PATHCONV=1.
```

---

## Test `/health` Endpoint

The task says:

```bash
kubectl exec <pod> -- curl -s http://localhost/health
```

But the official Nginx image may not include `curl` or `wget`.

In our practice, `wget` was missing inside the Nginx container.

Error:

```text
exec: "wget": executable file not found in $PATH
```

So we tested using a temporary BusyBox Pod.

First get the Nginx Pod IP:

```bash
kubectl get pod nginx-config-pod -o wide
```

Example output:

```text
nginx-config-pod   1/1   Running   10.244.1.3
```

Then test:

```bash
kubectl run health-test --image=busybox:latest --rm -it --restart=Never -- wget -qO- http://10.244.1.3/health
```

Expected output:

```text
healthy
```

This proves the mounted Nginx config is working.

---

# Task 4 – Create a Secret

## Goal

Create a Secret named:

```text
db-credentials
```

With values:

```text
DB_USER=admin
DB_PASSWORD=s3cureP@ssw0rd
```

---

## YAML Secret Options

There are two ways to write Secret YAML.

### Option 1: `data`

Values must be base64 encoded.

```yaml
data:
  DB_USER: YWRtaW4=
  DB_PASSWORD: czNjdXJlUEBzc3cwcmQ=
```

### Option 2: `stringData`

Values can be written in plain text.

```yaml
stringData:
  DB_USER: admin
  DB_PASSWORD: s3cureP@ssw0rd
```

Kubernetes converts `stringData` into base64 encoded `data`.

For learning, we used `stringData`.

---

## File: `db-secret.yaml`

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
stringData:
  DB_USER: admin
  DB_PASSWORD: s3cureP@ssw0rd
```

---

## YAML Explanation

### `kind: Secret`

```yaml
kind: Secret
```

This creates a Kubernetes Secret.

### `type: Opaque`

```yaml
type: Opaque
```

`Opaque` means a generic key-value Secret.

Use it for:

```text
DB_USER
DB_PASSWORD
API_KEY
JWT_SECRET
```

### `stringData`

```yaml
stringData:
  DB_USER: admin
  DB_PASSWORD: s3cureP@ssw0rd
```

`stringData` lets us write plaintext values.

Kubernetes stores them in encoded form under `data`.

### Production Warning

Do not commit real secrets to public GitHub repositories.

For production, use:

```text
External Secrets Operator
Sealed Secrets
SOPS
Vault
AWS Secrets Manager
Azure Key Vault
Google Secret Manager
CI/CD secret injection
```

---

## Commands

```bash
kubectl apply -f db-secret.yaml
```

Inspect Secret:

```bash
kubectl get secret db-credentials -o yaml
```

Expected:

```yaml
data:
  DB_PASSWORD: czNjdXJlUEBzc3cwcmQ=
  DB_USER: YWRtaW4=
```

Decode password:

```bash
kubectl get secret db-credentials -o jsonpath='{.data.DB_PASSWORD}' | base64 --decode; echo
```

Expected:

```text
s3cureP@ssw0rd
```

Decode user:

```bash
kubectl get secret db-credentials -o jsonpath='{.data.DB_USER}' | base64 --decode; echo
```

Expected:

```text
admin
```

---

## Important

Base64 is encoding, not encryption.

If someone can read this:

```text
czNjdXJlUEBzc3cwcmQ=
```

they can decode it back to:

```text
s3cureP@ssw0rd
```

---

# Task 5 – Use Secrets in a Pod

## Goal

Create a Pod that:

```text
1. Injects DB_USER as an environment variable using secretKeyRef
2. Mounts the full db-credentials Secret as a volume at /etc/db-credentials
3. Reads the mounted files
4. Verifies mounted file values are plaintext, not base64
```

---

## File: `secret-pod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-pod
spec:
  restartPolicy: Never
  containers:
  - name: busybox
    image: busybox:latest
    command:
    - sh
    - -c
    - |
      echo "Environment variable DB_USER=$DB_USER"
      echo "Mounted Secret files:"
      ls -l /etc/db-credentials
      echo "DB_USER file value:"
      cat /etc/db-credentials/DB_USER
      echo
      echo "DB_PASSWORD file value:"
      cat /etc/db-credentials/DB_PASSWORD
      echo
    env:
    - name: DB_USER
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: DB_USER
    volumeMounts:
    - name: db-credentials-volume
      mountPath: /etc/db-credentials
      readOnly: true
  volumes:
  - name: db-credentials-volume
    secret:
      secretName: db-credentials
```

---

## YAML Explanation

### Secret as Environment Variable

```yaml
env:
- name: DB_USER
  valueFrom:
    secretKeyRef:
      name: db-credentials
      key: DB_USER
```

This injects only the `DB_USER` key as an environment variable.

Inside the Pod:

```bash
echo $DB_USER
```

Output:

```text
admin
```

### Secret as Volume

```yaml
volumeMounts:
- name: db-credentials-volume
  mountPath: /etc/db-credentials
  readOnly: true
```

Mounts the Secret at:

```text
/etc/db-credentials
```

### Volume Source

```yaml
volumes:
- name: db-credentials-volume
  secret:
    secretName: db-credentials
```

The source Secret is:

```text
db-credentials
```

### Result Inside Pod

Each Secret key becomes a file:

```text
/etc/db-credentials/DB_USER
/etc/db-credentials/DB_PASSWORD
```

File content is decoded plaintext:

```text
DB_USER      -> admin
DB_PASSWORD  -> s3cureP@ssw0rd
```

---

## Commands

```bash
kubectl apply -f secret-pod.yaml
```

Check Pod:

```bash
kubectl get pod secret-pod
```

Check logs:

```bash
kubectl logs secret-pod
```

---

## Expected Output

```text
Environment variable DB_USER=admin
Mounted Secret files:
DB_PASSWORD -> ..data/DB_PASSWORD
DB_USER -> ..data/DB_USER
DB_USER file value:
admin
DB_PASSWORD file value:
s3cureP@ssw0rd
```

### Important

`kubectl get secret -o yaml` shows base64 encoded values.

But Secret mounted inside a Pod gives plaintext decoded values.

---

# Task 6 – Update a ConfigMap and Observe Propagation

## Goal

Prove that:

```text
Volume-mounted ConfigMap updates automatically after some delay.
Environment variable ConfigMap values do not update automatically.
```

---

## Step 1: Create live-config ConfigMap

## File: `live-config.yaml`

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: live-config
data:
  message: "hello"
```

Apply:

```bash
kubectl apply -f live-config.yaml
```

---

## Step 2: Create a Pod that Reads ConfigMap File in Loop

## File: `live-config-pod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: live-config-pod
spec:
  containers:
  - name: busybox
    image: busybox:latest
    command:
    - sh
    - -c
    - |
      while true; do
        echo "$(date) - message=$(cat /etc/live-config/message)"
        sleep 5
      done
    volumeMounts:
    - name: live-config-volume
      mountPath: /etc/live-config
  volumes:
  - name: live-config-volume
    configMap:
      name: live-config
```

---

## YAML Explanation

### Loop Command

```yaml
while true; do
  echo "$(date) - message=$(cat /etc/live-config/message)"
  sleep 5
done
```

This reads:

```text
/etc/live-config/message
```

every 5 seconds.

Initially file content is:

```text
hello
```

After ConfigMap update, it changes to:

```text
world
```

### Volume Mount

```yaml
volumeMounts:
- name: live-config-volume
  mountPath: /etc/live-config
```

Mounts the ConfigMap as files.

### ConfigMap Volume

```yaml
volumes:
- name: live-config-volume
  configMap:
    name: live-config
```

The key `message` becomes:

```text
/etc/live-config/message
```

---

## Commands

Apply Pod:

```bash
kubectl apply -f live-config-pod.yaml
```

Check logs:

```bash
kubectl logs live-config-pod --tail=5
```

Expected:

```text
message=hello
message=hello
message=hello
```

---

## Patch ConfigMap

```bash
kubectl patch configmap live-config --type merge -p '{"data":{"message":"world"}}'
```

Expected:

```text
configmap/live-config patched
```

Wait 30-60 seconds.

Then check logs:

```bash
kubectl logs live-config-pod --tail=20
```

Expected:

```text
message=hello
message=hello
message=world
message=world
```

Check Pod restart count:
kubectl get pod live-config-pod
Expected:
RESTARTS   0


## Actual Observation
Logs showed:
Sun May 31 12:53:17 UTC 2026 - message=hello
Sun May 31 12:53:22 UTC 2026 - message=hello
Sun May 31 12:53:27 UTC 2026 - message=hello
Sun May 31 12:54:17 UTC 2026 - message=world
Sun May 31 12:54:22 UTC 2026 - message=world

Pod status:
live-config-pod   1/1   Running   0

This proves:
ConfigMap volume updated without restarting the Pod.

## Very Important Difference

| Usage Type | Updates Automatically? | Restart Needed? |
|---|---:|---:|
| ConfigMap as env var | No | Yes |
| ConfigMap as volume | Yes, after delay | No |
| Secret as env var | No | Yes |
| Secret as volume | Usually yes, after delay | No |

# Task 7 – Clean Up

## Created Resources
Pods:
config-env-pod
nginx-config-pod
secret-pod
live-config-pod

ConfigMaps:
app-config
nginx-config
live-config

Secret:
db-credentials

## Delete Pods
kubectl delete pod config-env-pod nginx-config-pod secret-pod live-config-pod


## Delete ConfigMaps
kubectl delete configmap app-config nginx-config live-config

## Delete Secret
kubectl delete secret db-credentials

## Verify Cleanup
kubectl get pods
Expected:
No resources found in default namespace.

Check ConfigMaps:
kubectl get configmaps
Expected only default ConfigMap:
kube-root-ca.crt
Check Secrets:
kubectl get secrets

Expected:
No resources found in default namespace.
or only default/service-account related secrets depending on cluster version.

# Important Commands Summary

## ConfigMaps
kubectl get configmaps
kubectl describe configmap app-config
kubectl get configmap app-config -o yaml
kubectl get configmap nginx-config -o yaml

## Secrets
kubectl get secrets
kubectl get secret db-credentials -o yaml
kubectl get secret db-credentials -o jsonpath='{.data.DB_PASSWORD}' | base64 --decode; echo

## Pods
kubectl get pods
kubectl logs config-env-pod
kubectl logs secret-pod
kubectl logs live-config-pod --tail=20

## Git Bash Path Conversion Fix
MSYS_NO_PATHCONV=1 kubectl exec nginx-config-pod -- ls -l /etc/nginx/conf.d


# Common Issues and Fixes

## Issue 1: ConfigMap already exists
Error:
configmaps "nginx-config" already exists
Reason:
`kubectl create` creates a new resource and fails if it already exists.
Fix:
Use YAML and apply:
kubectl apply -f nginx-config.yaml

Or delete and recreate:
kubectl delete configmap nginx-config
kubectl create configmap nginx-config --from-file=default.conf=custom-nginx.conf

## Issue 2: Logs checked too early
Error:
container is waiting to start: ContainerCreating

Fix:
Wait a few seconds and run:
kubectl logs <pod-name>

## Issue 3: Git Bash converts Linux paths
Error:
ls: cannot access 'C:/Program Files/Git/etc/nginx/conf.d'

Fix:
MSYS_NO_PATHCONV=1 kubectl exec nginx-config-pod -- ls -l /etc/nginx/conf.d

## Issue 4: Nginx image does not have curl/wget
Error:
exec: "wget": executable file not found in $PATH

Fix:
Use a temporary BusyBox Pod:
kubectl get pod nginx-config-pod -o wide
kubectl run health-test --image=busybox:latest --rm -it --restart=Never -- wget -qO- http://<POD-IP>/health


# Final Folder Structure
Recommended GitHub structure:
2026/day-54/
├── app-config.yaml
├── nginx-config.yaml
├── config-env-pod.yaml
├── nginx-config-pod.yaml
├── db-secret.yaml
├── secret-pod.yaml
├── live-config.yaml
├── live-config-pod.yaml
└── day-54-configmaps-secrets.md

# Interview Questions and Answers

## 1. What is a ConfigMap in Kubernetes?
A ConfigMap is a Kubernetes object used to store non-sensitive configuration data as key-value pairs or configuration files.
Example:
APP_ENV=production
APP_PORT=8080
It helps separate configuration from container images.


## 2. Why should we not hardcode configuration inside Docker images?
Because if configuration changes, we need to rebuild the Docker image.
Using ConfigMaps and Secrets allows us to change configuration without rebuilding the image.


## 3. What is a Secret in Kubernetes?
A Secret is a Kubernetes object used to store sensitive information such as passwords, API keys, tokens, and certificates.
Example:
DB_PASSWORD
API_KEY
JWT_SECRET


## 4. Are Kubernetes Secrets encrypted by default?
No.
Secrets are base64 encoded by default, not encrypted.
Base64 is only encoding and can be decoded easily.

## 5. What is the difference between ConfigMap and Secret?
ConfigMap is for non-sensitive data.
Secret is for sensitive data.
ConfigMap stores values as plain text.
Secret stores values as base64 encoded data.

## 6. What is `stringData` in a Kubernetes Secret?
`stringData` allows us to write Secret values in plain text inside YAML.
Kubernetes automatically converts `stringData` into base64 encoded `data`.

Example:
stringData:
  DB_USER: admin


## 7. What is `data` in a Kubernetes Secret?
`data` stores base64 encoded values.
Example:
data:
  DB_USER: YWRtaW4=

## 8. What is the difference between `envFrom` and `env`?
`envFrom` injects all keys from a ConfigMap or Secret as environment variables.

Example:
envFrom:
- configMapRef:
    name: app-config
  
`env` with `valueFrom` injects a specific key.

Example:
env:
- name: DB_USER
  valueFrom:
    secretKeyRef:
      name: db-credentials
      key: DB_USER

## 9. When should we use ConfigMap as environment variables?
Use environment variables for simple key-value settings.

Examples:
APP_ENV
APP_PORT
LOG_LEVEL
FEATURE_FLAG
## 10. When should we use ConfigMap as volume?
Use volume mounts for full configuration files.
Examples:
Nginx config
Application YAML config
JSON config
Properties files

## 11. Do ConfigMap environment variables update automatically?
No
Environment variables are set when the Pod starts.
If the ConfigMap changes later, the running Pod does not get updated env values.
The Pod must be restarted or recreated.

## 12. Do volume-mounted ConfigMaps update automatically?
Yes, usually after a delay.
In our task, `message=hello` changed to `message=world` without Pod restart.


## 13. Why does ConfigMap volume update take time?
Kubernetes updates mounted ConfigMap files eventually.
It may take around 30-60 seconds depending on kubelet sync behavior.

## 14. What happens when a ConfigMap is mounted as a volume?
Each key becomes a file.
Example:
ConfigMap key: message
Mounted file: /etc/live-config/message

## 15. What happens when a Secret is mounted as a volume?
Each Secret key becomes a file.
The file content is decoded plaintext, not base64.
Example:
/etc/db-credentials/DB_USER -> admin
/etc/db-credentials/DB_PASSWORD -> s3cureP@ssw0rd

## 16. Why did `kubectl exec nginx-config-pod -- wget` fail?
Because the official `nginx:1.25` image does not include `wget` or `curl`.
We used a temporary BusyBox Pod to test the `/health` endpoint.


## 17. Why did Git Bash convert `/etc/nginx/conf.d` to `C:/Program Files/Git/etc/nginx/conf.d`?
Git Bash automatically converts Linux-style paths to Windows paths.
To disable this behavior, use:
MSYS_NO_PATHCONV=1 kubectl exec nginx-config-pod -- ls -l /etc/nginx/conf.d

## 18. What is the difference between `kubectl create` and `kubectl apply`?
`kubectl create` creates a resource and fails if it already exists.
`kubectl apply` creates the resource if it does not exist and updates it if it already exists.
For GitHub and CI/CD, `kubectl apply -f file.yaml` is preferred.

## 19. Should Secret YAML files be committed to GitHub?
Not if they contain real sensitive values.
For learning it is okay, but in production use:
External Secrets Operator
Sealed Secrets
SOPS
Vault
Cloud Secret Manager
CI/CD secret injection

## 20. How do you decode a Secret value?
Use:
kubectl get secret db-credentials -o jsonpath='{.data.DB_PASSWORD}' | base64 --decode; echo

## 21. What is `readOnly: true` in a Secret volume mount?
It makes the mounted Secret files read-only inside the container.
This is recommended because applications should read Secrets, not modify them.

## 22. Can ConfigMaps and Secrets be used in Deployments?
Yes.
In real applications, we usually use ConfigMaps and Secrets inside Deployment Pod templates instead of standalone Pods.
Example:
spec:
  template:
    spec:
      containers:
      - name: app
        envFrom:
        - configMapRef:
            name: app-config

## 23. Which is better for Nginx config: environment variable or volume mount?
Volume mount is better because Nginx expects config files.
Example:
/etc/nginx/conf.d/default.conf

## 24. Which is better for APP_PORT: environment variable or volume mount?

Environment variable is better because `APP_PORT` is a simple key-value setting.
