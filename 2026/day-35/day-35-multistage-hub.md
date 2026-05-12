# Day 35 – Multi-Stage Builds & Docker Hub

## Goal

Today's goal is to build optimized Docker images and share them using Docker Hub.

Main focus:

```text
1. Build a normal large Docker image
2. Build an optimized multi-stage Docker image
3. Compare image sizes
4. Push image to Docker Hub
5. Pull and verify image
6. Apply image best practices
```

Multi-stage builds are important because real teams ship images that are:

- Small
- Secure
- Fast to pull
- Easy to scan
- Free from unnecessary build tools
- Suitable for CI/CD and Kubernetes deployments

Docker Hub is a container image registry used to store and distribute Docker images.

---

# Topics Covered

- Problem with large Docker images
- Single-stage Dockerfile
- Multi-stage Dockerfile
- Builder stage and runtime stage
- Minimal base images
- Docker Hub login
- Docker image tagging
- Docker push and pull
- Docker Hub repository and tags
- Image versioning
- Non-root containers
- Alpine vs Ubuntu
- Dockerfile best practices
- Basic, scenario-based, and senior-level interview questions

---

# 1. Problem with Large Docker Images

Docker images become large when they contain:

```text
Build tools
Compilers
Package managers
Temporary files
Source code
Development dependencies
Cache files
Unused libraries
```

Example:

If we build a Go app with:

```dockerfile
FROM golang:1.22
```

the final image contains:

```text
Go compiler
Go tools
Build cache
Source code
Final binary
```

But production only needs:

```text
Final compiled binary
Runtime dependencies
```

## Problems caused by large images

| Problem | Impact |
|---|---|
| Slow pull | Slower deployments |
| More disk usage | More storage cost |
| Bigger attack surface | More security risk |
| More CVEs | More vulnerability findings |
| Slower CI/CD | Longer pipeline time |
| Unnecessary tools | Not production-friendly |

---

# 2. What is a Multi-Stage Build?

A multi-stage build uses multiple `FROM` statements in one Dockerfile.

Example:

```dockerfile
FROM golang:1.22-alpine AS builder
# build app here

FROM alpine:3.20
# copy only final binary here
```

The first stage builds the app.

The final stage contains only what is needed to run the app.

---

# 3. Why Multi-Stage Images Are Smaller

Single-stage image:

```text
Base image + compiler + source code + dependencies + final app
```

Multi-stage image:

```text
Minimal runtime base image + final built artifact only
```

So the final image does not contain:

```text
Compiler
Build cache
Source code
Development dependencies
Package manager cache
```

---

# 4. Docker Hub Basics

Docker Hub is a container registry.

It is used to store and share Docker images.

Common workflow:

```bash
docker login
docker tag local-image:tag username/repository:tag
docker push username/repository:tag
docker pull username/repository:tag
```

Example:

```bash
docker tag day35-go-multistage:v1 yourusername/day35-go-app:v1
docker push yourusername/day35-go-app:v1
docker pull yourusername/day35-go-app:v1
```

---

# 5. Docker Image Tags

Docker image format:

```text
username/repository:tag
```

Example:

```text
yourusername/day35-go-app:v1
yourusername/day35-go-app:latest
```

Tags are used for versioning:

```text
v1
v1.0.0
dev
staging
prod
latest
```

Important:

> `latest` is just a tag name. It does not always mean newest.

For production, use specific version tags like:

```text
v1.0.0
```

---

# Project Folder Structure

```text
2026/day-35/
│
├── go-app/
│   ├── main.go
│   ├── Dockerfile.single
│   ├── Dockerfile.multistage
│   └── Dockerfile.best
│
└── day-35-multistage-hub.md
```

---

# Task 1 – The Problem with Large Images

## Goal

Write a simple Go app, build it using a single-stage Dockerfile, and check image size.

---

## Step 1: Create Folder

```bash
mkdir -p ~/2026/day-35/go-app
cd ~/2026/day-35/go-app
```

---

## Step 2: Create Go App

Create `main.go`:

```bash
cat > main.go <<'EOF'
package main

import (
	"fmt"
	"net/http"
	"os"
)

func main() {
	port := os.Getenv("PORT")
	if port == "" {
		port = "8080"
	}

	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintln(w, "Hello from Day 35 Go Docker App!")
	})

	http.HandleFunc("/health", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintln(w, "OK")
	})

	fmt.Println("Server running on port", port)
	http.ListenAndServe(":"+port, nil)
}
EOF
```

This app exposes:

```text
/        → Hello message
/health  → OK
```

---

## Step 3: Create Single-Stage Dockerfile

Create `Dockerfile.single`:

```bash
cat > Dockerfile.single <<'EOF'
FROM golang:1.22

WORKDIR /app

COPY main.go .

RUN go build -o day35-app main.go

EXPOSE 8080

CMD ["./day35-app"]
EOF
```

---

## Step 4: Build Single-Stage Image

```bash
docker build -f Dockerfile.single -t day35-go-single:v1 .
```

---

## Step 5: Check Image Size

```bash
docker images | grep day35-go-single
```

Write the size in your notes:

```md
Single-stage image size: ______ MB
```

---

## Step 6: Run and Test

```bash
docker run -d --name day35-single -p 8087:8080 day35-go-single:v1
curl http://localhost:8087
curl http://localhost:8087/health
```

Expected:

```text
Hello from Day 35 Go Docker App!
OK
```

Cleanup:

```bash
docker rm -f day35-single
```

---

## Task 1 Result

The single-stage image works, but it is large because it contains the full Go build environment.

---

# Task 2 – Multi-Stage Build

## Goal

Rewrite the Dockerfile using multi-stage build.

```text
Stage 1: Build the app
Stage 2: Copy only final binary into minimal image
```

---

## Step 1: Create Multi-Stage Dockerfile

Create `Dockerfile.multistage`:

```bash
cat > Dockerfile.multistage <<'EOF'
FROM golang:1.22-alpine AS builder

WORKDIR /app

COPY main.go .

RUN go build -o day35-app main.go

FROM alpine:3.20

WORKDIR /app

COPY --from=builder /app/day35-app .

EXPOSE 8080

CMD ["./day35-app"]
EOF
```

---

## Explanation

### Builder stage

```dockerfile
FROM golang:1.22-alpine AS builder
```

This stage has the Go compiler.

```dockerfile
RUN go build -o day35-app main.go
```

This builds the binary.

### Runtime stage

```dockerfile
FROM alpine:3.20
```

This uses a minimal runtime base.

```dockerfile
COPY --from=builder /app/day35-app .
```

This copies only the compiled binary from the builder stage.

---

## Step 2: Build Multi-Stage Image

```bash
docker build -f Dockerfile.multistage -t day35-go-multistage:v1 .
```

---

## Step 3: Compare Image Sizes

```bash
docker images | grep day35-go
```

Fill actual sizes:

| Image | Dockerfile | Size |
|---|---|---|
| `day35-go-single:v1` | `Dockerfile.single` | ______ MB |
| `day35-go-multistage:v1` | `Dockerfile.multistage` | ______ MB |

---

## Step 4: Run and Test

```bash
docker run -d --name day35-multi -p 8088:8080 day35-go-multistage:v1
curl http://localhost:8088
curl http://localhost:8088/health
```

Expected:

```text
Hello from Day 35 Go Docker App!
OK
```

Cleanup:

```bash
docker rm -f day35-multi
```

---

## Task 2 Result

The multi-stage image is much smaller because the final image contains only:

```text
Alpine runtime image
Compiled Go binary
```

It does not contain:

```text
Go compiler
Build tools
Source code
Build cache
Development dependencies
```

---

# Task 3 – Push to Docker Hub

## Goal

Push optimized image to Docker Hub.

---

## Step 1: Create Docker Hub Account

Create a free Docker Hub account if you do not have one.

You need your Docker Hub username.

Example:

```text
yourusername
```

---

## Step 2: Login from Terminal

```bash
docker login
```

Enter:

```text
Docker Hub username
Docker Hub password or access token
```

Best practice:

> Use Docker Hub access token instead of your password.

---

## Step 3: Tag Image

Format:

```bash
docker tag local-image:tag username/repository:tag
```

Example:

```bash
docker tag day35-go-multistage:v1 YOUR_DOCKERHUB_USERNAME/day35-go-app:v1
docker tag day35-go-multistage:v1 YOUR_DOCKERHUB_USERNAME/day35-go-app:latest
```

---

## Step 4: Push Image

```bash
docker push YOUR_DOCKERHUB_USERNAME/day35-go-app:v1
docker push YOUR_DOCKERHUB_USERNAME/day35-go-app:latest
```

---

## Step 5: Verify Pull

Remove local image tags:

```bash
docker rmi YOUR_DOCKERHUB_USERNAME/day35-go-app:v1
docker rmi YOUR_DOCKERHUB_USERNAME/day35-go-app:latest
```

Pull again:

```bash
docker pull YOUR_DOCKERHUB_USERNAME/day35-go-app:v1
```

Run pulled image:

```bash
docker run -d --name day35-hub-test -p 8089:8080 YOUR_DOCKERHUB_USERNAME/day35-go-app:v1
curl http://localhost:8089
docker rm -f day35-hub-test
```

Expected:

```text
Hello from Day 35 Go Docker App!
```

---

# Task 4 – Docker Hub Repository

## Goal

Check pushed image on Docker Hub and understand repository tags.

---

## Steps

1. Open Docker Hub.
2. Go to your repository:

```text
YOUR_DOCKERHUB_USERNAME/day35-go-app
```

3. Add repository description.
4. Open Tags tab.
5. Check tags:

```text
v1
latest
```

---

## Pull Specific Tag vs Latest

Specific tag:

```bash
docker pull YOUR_DOCKERHUB_USERNAME/day35-go-app:v1
```

Latest tag:

```bash
docker pull YOUR_DOCKERHUB_USERNAME/day35-go-app:latest
```

No tag:

```bash
docker pull YOUR_DOCKERHUB_USERNAME/day35-go-app
```

Docker uses:

```text
latest
```

Important:

> `latest` is not automatically the newest. It is just a tag.

---

# Task 5 – Image Best Practices

## Goal

Apply production image best practices:

```text
Use minimal base image
Do not run as root
Combine commands
Use specific tags
Check image size before and after
```

---

## Best Practice 1: Minimal Base Image

Avoid large base images when not needed.

Bad:

```dockerfile
FROM ubuntu
```

Better:

```dockerfile
FROM alpine:3.20
```

For Go apps, possible minimal bases:

```text
alpine
scratch
distroless
```

---

## Best Practice 2: Do Not Run as Root

Running as root is risky.

Create a non-root user:

```dockerfile
RUN adduser -D appuser
USER appuser
```

---

## Best Practice 3: Use Specific Tags

Avoid:

```dockerfile
FROM alpine:latest
```

Use:

```dockerfile
FROM alpine:3.20
```

Specific tags make builds predictable.

---

## Best Practice 4: Combine RUN Commands

Instead of:

```dockerfile
RUN apk update
RUN apk add curl
RUN rm -rf /var/cache/apk/*
```

Use:

```dockerfile
RUN apk add --no-cache curl
```

This avoids extra cache and unnecessary layers.

---

## Best Practice 5: Use `.dockerignore`

Ignore:

```text
.git
node_modules
.env
*.md
logs
coverage
```

---

## Best-Practice Dockerfile

Create `Dockerfile.best`:

```bash
cat > Dockerfile.best <<'EOF'
FROM golang:1.22-alpine AS builder

WORKDIR /app

COPY main.go .

RUN CGO_ENABLED=0 GOOS=linux go build -o day35-app main.go

FROM alpine:3.20

WORKDIR /app

RUN adduser -D appuser

COPY --from=builder /app/day35-app .

RUN chown appuser:appuser /app/day35-app

USER appuser

EXPOSE 8080

CMD ["./day35-app"]
EOF
```

---

## Build Best-Practice Image

```bash
docker build -f Dockerfile.best -t day35-go-best:v1 .
```

Check size:

```bash
docker images | grep day35-go
```

Run:

```bash
docker run -d --name day35-best -p 8090:8080 day35-go-best:v1
curl http://localhost:8090
docker exec day35-best whoami
docker rm -f day35-best
```

Expected:

```text
Hello from Day 35 Go Docker App!
appuser
```

---

# Final Dockerfiles

## Dockerfile.single

```dockerfile
FROM golang:1.22

WORKDIR /app

COPY main.go .

RUN go build -o day35-app main.go

EXPOSE 8080

CMD ["./day35-app"]
```

---

## Dockerfile.multistage

```dockerfile
FROM golang:1.22-alpine AS builder

WORKDIR /app

COPY main.go .

RUN go build -o day35-app main.go

FROM alpine:3.20

WORKDIR /app

COPY --from=builder /app/day35-app .

EXPOSE 8080

CMD ["./day35-app"]
```

---

## Dockerfile.best

```dockerfile
FROM golang:1.22-alpine AS builder

WORKDIR /app

COPY main.go .

RUN CGO_ENABLED=0 GOOS=linux go build -o day35-app main.go

FROM alpine:3.20

WORKDIR /app

RUN adduser -D appuser

COPY --from=builder /app/day35-app .

RUN chown appuser:appuser /app/day35-app

USER appuser

EXPOSE 8080

CMD ["./day35-app"]
```

---

# Important Commands Summary

## Build Images

```bash
docker build -f Dockerfile.single -t day35-go-single:v1 .
docker build -f Dockerfile.multistage -t day35-go-multistage:v1 .
docker build -f Dockerfile.best -t day35-go-best:v1 .
```

## Check Image Size

```bash
docker images | grep day35-go
```

## Run Images

```bash
docker run -d --name day35-single -p 8087:8080 day35-go-single:v1
docker run -d --name day35-multi -p 8088:8080 day35-go-multistage:v1
docker run -d --name day35-best -p 8090:8080 day35-go-best:v1
```

## Test

```bash
curl http://localhost:8087
curl http://localhost:8088
curl http://localhost:8090
```

## Docker Hub

```bash
docker login
docker tag day35-go-multistage:v1 YOUR_DOCKERHUB_USERNAME/day35-go-app:v1
docker tag day35-go-multistage:v1 YOUR_DOCKERHUB_USERNAME/day35-go-app:latest
docker push YOUR_DOCKERHUB_USERNAME/day35-go-app:v1
docker push YOUR_DOCKERHUB_USERNAME/day35-go-app:latest
docker pull YOUR_DOCKERHUB_USERNAME/day35-go-app:v1
```

---

# Screenshots to Attach

Attach screenshots for:

1. `main.go`
2. `Dockerfile.single`
3. Single-stage build
4. Single-stage image size
5. Running single-stage image
6. `Dockerfile.multistage`
7. Multi-stage build
8. Multi-stage image size
9. Size comparison
10. Docker Hub login
11. Docker tag command
12. Docker push command
13. Docker Hub repository page
14. Docker Hub tags page
15. Pull image from Docker Hub
16. Run pulled image
17. `Dockerfile.best`
18. Non-root user verification with `whoami`

---

# Troubleshooting Notes

## docker login fails

Use Docker Hub username and access token.

Create token:

```text
Docker Hub → Account Settings → Personal access tokens
```

---

## push denied

Error:

```text
denied: requested access to the resource is denied
```

Check:

```text
Correct Docker Hub username
Logged in successfully
Repository name is correct
Image tag is correct
```

---

## image not found while push

Check local images:

```bash
docker images
```

Then tag correctly:

```bash
docker tag day35-go-multistage:v1 yourusername/day35-go-app:v1
```

---

## port already allocated

Check running containers:

```bash
docker ps
```

Remove old one:

```bash
docker rm -f container-name
```

---

## permission denied with non-root user

Make sure ownership is correct:

```dockerfile
RUN chown appuser:appuser /app/day35-app
USER appuser
```

---

# Practice Questions

## Basic Questions

1. What is a multi-stage Docker build?
2. Why are single-stage images often large?
3. What does `FROM ... AS builder` mean?
4. What does `COPY --from=builder` do?
5. What is Docker Hub?
6. What is a Docker image tag?
7. What does `docker login` do?
8. What does `docker push` do?
9. What does `docker pull` do?
10. Why should we avoid using only `latest` in production?

## Scenario-Based Questions

1. Your Docker image is 900MB. How can you reduce it?
2. Your final image contains source code and compiler. What strategy should you use?
3. Docker push fails with access denied. What will you check?
4. Your app works locally but fails after pulling from Docker Hub. What will you verify?
5. You want to deploy version 1.0.0 exactly. Which tag should you use?
6. Your image runs as root. Why is this risky?
7. Your base image uses `latest`. Why is this a problem?
8. You want a very small Go image. Which base image can you use?
9. You want to share an image with your team. Where can you push it?
10. You changed source code but image still behaves old. What should you do?

---

# Interview Questions and Answers

## Q1. What is a multi-stage Docker build?

A multi-stage Docker build uses multiple `FROM` statements in one Dockerfile. One stage builds the app, and another stage runs it with only the required artifact.

---

## Q2. Why use multi-stage builds?

They reduce image size and improve security by excluding build tools, compilers, source code, and development dependencies from the final image.

---

## Q3. What is the builder stage?

The builder stage is where the application is compiled or built.

Example:

```dockerfile
FROM golang:1.22-alpine AS builder
```

---

## Q4. What is the runtime stage?

The runtime stage is the final production image that contains only runtime dependencies and the built artifact.

---

## Q5. What does `COPY --from=builder` do?

It copies files from a previous stage into the current stage.

Example:

```dockerfile
COPY --from=builder /app/day35-app .
```

---

## Q6. Why is a multi-stage image smaller?

Because the final stage contains only the built artifact and minimal runtime base, not compiler, source code, build tools, or cache.

---

## Q7. What is Docker Hub?

Docker Hub is a container registry where Docker images can be stored, shared, pushed, and pulled.

---

## Q8. How do you push an image to Docker Hub?

```bash
docker login
docker tag local-image:tag username/repository:tag
docker push username/repository:tag
```

---

## Q9. What is the difference between `docker tag` and `docker push`?

`docker tag` gives a local image a repository name and tag.

`docker push` uploads that tagged image to a registry.

---

## Q10. What is the difference between `latest` and version tags?

`latest` is just a tag name. It is not guaranteed to be the newest or safest image.

Version tags like `v1.0.0` are specific and predictable.

---

# Senior-Level Interview Questions and Answers

## Q11. How would you optimize a Docker image for production?

Use:

- Multi-stage builds
- Minimal base image
- `.dockerignore`
- Non-root user
- Specific base image tags
- Locked dependency versions
- Fewer unnecessary layers
- Vulnerability scanning
- No secrets in image

---

## Q12. Why is running containers as root risky?

If the app is compromised, root access inside the container increases the impact. It may allow attackers to access sensitive files, abuse mounted volumes, or exploit misconfigurations.

Running as non-root reduces risk.

---

## Q13. What are distroless images?

Distroless images contain only the application and required runtime libraries. They do not include shell, package manager, or unnecessary OS tools.

They improve security and reduce size, but debugging is harder.

---

## Q14. When would you use `scratch`?

`scratch` is an empty base image. It is useful for statically compiled binaries like Go apps.

It creates extremely small images but has no shell or debugging tools.

---

## Q15. What is the downside of Alpine?

Alpine is small but uses `musl libc` instead of `glibc`. Some applications or dependencies may behave differently or require fixes.

For some workloads, Debian slim or distroless may be safer.

---

## Q16. How do you handle secrets during Docker builds?

Do not copy secrets into images.

Use:

- Docker BuildKit secrets
- Runtime environment variables
- Docker secrets
- Kubernetes secrets
- CI/CD secret variables
- Cloud secret managers

---

## Q17. How do you make Docker builds reproducible?

Use:

- Specific base image tags
- Locked dependency versions
- Immutable app tags
- `.dockerignore`
- CI/CD build pipeline
- Avoid relying only on `latest`

---

## Q18. How do layers affect image size?

Each Dockerfile instruction can create a layer. If files are added in one layer and removed in another, the data may still exist in previous layers.

Clean up in the same `RUN` command where files are created.

---

## Q19. Why should build tools not be in final image?

Build tools increase size and security risk. They are not required at runtime.

Multi-stage builds keep them in the builder stage only.

---

## Q20. What Docker image release strategy would you recommend?

Use immutable semantic version tags:

```text
v1.0.0
v1.0.1
v1.1.0
```

Optionally also maintain:

```text
latest
dev
staging
prod
```

Production deployments should use exact version tags, not only `latest`.

---

# Real-World DevOps Notes

## CI/CD Image Flow

```text
Developer pushes code
        ↓
CI pipeline runs tests
        ↓
Docker image is built
        ↓
Image is tagged with version or commit SHA
        ↓
Image is pushed to registry
        ↓
Deployment pulls exact image tag
```

Example tags:

```text
myapp:v1.0.0
myapp:git-abc123
myapp:2026-05-12
```

---

# Production Image Checklist

Before shipping an image:

```text
Use multi-stage build
Use minimal runtime image
Run as non-root
Use specific base image tags
Do not include secrets
Do not include unnecessary source code
Use .dockerignore
Scan image
Tag properly
Push to registry
Test pull and run
```

---

# Final Revision Notes

## Key Points

- Single-stage images are often large.
- Multi-stage builds separate build and runtime.
- Builder stage contains compilers and build tools.
- Runtime stage contains only the final artifact.
- `COPY --from=builder` copies files from another stage.
- Minimal images are smaller and safer.
- Docker Hub stores and distributes images.
- `docker tag` prepares image name for registry.
- `docker push` uploads image.
- `docker pull` downloads image.
- Avoid relying only on `latest`.
- Run containers as non-root.
- Use specific base image tags.
- Use `.dockerignore`.
- Scan images before production.
