---
tags: [desktop-support, devops, containers, docker, L2]
aliases: [docker-fundamentals, dev-01]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #none
---

> [!NOTE|color-blue]
> ☁️ **DEVOPS / CLOUD**

`#complete` `#intermediate` `#none`

# DEV-01: Docker Fundamentals

> [!abstract] Overview
> This note covers the fundamentals of Docker containerization within a DevOps workflow. It details the difference between VMs and containers, SPEC file-like Dockerfile authoring, multi-stage image optimization, Docker volumes, custom networks, and multi-container orchestration using Docker Compose.

---
## 🧠 Concept Overview

- **What it is** — Docker is an open-source containerization platform that packages application code, libraries, dependencies, and system tools into a single, immutable package called a Docker Image. This image runs in a lightweight, isolated user-space environment called a Container.
- **Why it matters** — A common challenge in software deployment is configuration drift between test and production servers. Docker eliminates the "it worked on my machine" problem by ensuring the application runs identically in any environment because it carries its runtime environment with it.
- **Where you see this** — Containerizing custom python or node applications, deploying multi-container stacks (like a web app connected to a database), pushing internal images to private registries, and optimizing host disk usage.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Verify active containers using `docker ps`, restart failed containers, inspect logs (`docker logs`), monitor host disk space |
| **L2** | Write standard `Dockerfiles` to package applications, compile images using `docker build`, tag/push images, map network port access |
| **L3** | Author complex `docker-compose.yml` configurations, configure overlay networks, implement multi-stage builds, secure Docker daemon |

> [!tip] Seedha Simple Mein
> *Docker ek software packaging tool hai jo application code aur dependencies ko ek image me convert kar deta hai. VM ke opposite, containers direct host kernel share karte hain, jisse ye fast aur lightweight ho jate hain. Multi-container applications ko single command se run karne ke liye `docker-compose` use kiya jata hai.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Shipping Containers** are like **Docker Containers** because...
>
> - Before shipping containers, goods were packed differently (bags, boxes, crates) and ships had to adapt. Now, everything goes into standard steel containers that fit any ship, train, or truck.
> - Similarly, before Docker, code was deployed differently on every server. With Docker, you pack your app into a standard container that runs exactly the same on your laptop, the testing server, or in the cloud.

---
## 🔬 Technical Deep Dive

### 1. Virtual Machines vs. Containers

> [!info] Key Concept
> Understanding the resource boundary is essential for systems engineering.

- **Virtual Machines**: Abstract physical hardware. Each VM includes a virtual BIOS, virtual drivers, and a full Guest Operating System. Consumes gigabytes of space and takes minutes to boot.
- **Containers**: Abstract the operating system kernel. They share the host OS kernel directly using Linux **Namespaces** (for isolation) and **Control Groups / cgroups** (for resource limitation). Containers start in milliseconds and have megabyte footprints.

### 2. Anatomy of a Dockerfile

A Dockerfile is the text configuration script describing the step-by-step assembly of a Docker Image:
- `FROM`: Sets the base parent image (e.g. `alpine`, `ubuntu`, `node:18`).
- `WORKDIR`: Creates and sets the active working directory inside the container.
- `COPY` / `ADD`: Copies files from the host machine into the container.
- `RUN`: Executes shell commands during the build phase (e.g. `npm install`).
- `EXPOSE`: Documents the container network port.
- `ENV`: Injects environment variables.
- `CMD` / `ENTRYPOINT`: Specifies the default executable command run when the container boots.

### 3. Multi-Stage Builds

> [!tip] Pro Tip
> In DevOps pipelines, building lightweight images is critical. Multi-stage builds reduce the final production image size by up to 90%.

- **Stage 1** compiles the binary using heavy development tools (like JDK or Go SDK).
- **Stage 2** copies *only* the compiled binary into a lightweight runtime image (e.g., `alpine`), leaving the heavy compilers behind.

### 4. Docker Compose

> [!info] Key Concept
> Instead of running multiple individual `docker run` commands, you define the entire application stack (services, databases, networks, volumes) in a single `docker-compose.yml` file and start it using `docker-compose up`.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - A Linux server (Rocky Linux or Ubuntu) with Docker Engine installed.
> - A standard user account added to the `docker` group.
> - Outbound internet access to pull base images.

### Step 1: Create a Simple Web Application

```bash
mkdir ~/node-project && cd ~/node-project
```

Create `server.js`:
```javascript
const http = require('http');
const port = 3000;

const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/html');
  res.end('<h1>Docker DevOps Lab Complete!</h1>\n');
});

server.listen(port, () => {
  console.log(`Server running on port ${port}`);
});
```

### Step 2: Write the Dockerfile

Create `Dockerfile`:
```dockerfile
# Use lightweight Node runtime as base
FROM node:18-alpine

# Set working path
WORKDIR /app

# Copy application script
COPY server.js .

# Document listening port
EXPOSE 3000

# Execute server on startup
CMD ["node", "server.js"]
```

### Step 3: Compile the Custom Docker Image

```bash
docker build -t devops-web:1.0 .
```

> [!success] Expected Output
> `Successfully built [Image-ID-Hash]`
> `Successfully tagged devops-web:1.0`

### Step 4: Run and Test the Container

```bash
# Launch the container mapping host port 8080 to container port 3000
docker run -d --name my-web-app -p 8080:3000 devops-web:1.0

# Verify container state
docker ps

# Test web connection
curl http://localhost:8080
```

> [!success] Expected Output
> `<h1>Docker DevOps Lab Complete!</h1>`

### Step 5: Orchestrate with Docker Compose

Create a `docker-compose.yml` file:
```yaml
version: '3.8'

services:
  web:
    image: devops-web:1.0
    ports:
      - "8080:3000"
    networks:
      - app-net
    depends_on:
      - cache

  cache:
    image: redis:alpine
    networks:
      - app-net

networks:
  app-net:
    driver: bridge
```

Start the stack:
```bash
docker-compose up -d
```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|---------|-------------|---------|
| `docker build -t <tag> .` | Compile image | `docker build -t web:latest .` |
| `docker run -d -p 80:80 <img>` | Run detached | `docker run -d -p 8080:3000 my-image` |
| `docker logs <container>` | View app logs | `docker logs web-server` |
| `docker exec -it <name> sh` | Enter container shell | `docker exec -it web-server sh` |
| `docker-compose up -d` | Start multi-container stack | `docker-compose up -d` |
| `docker-compose down` | Stop & remove stack | `docker-compose down` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|---------|-------------|-----|
| Docker commands fail with "Permission denied" | User not in `docker` group | Add user: `sudo usermod -aG docker $USER`. Re-login. |
| Container starts but exits immediately | No foreground process | Ensure app runs in foreground (no daemon scripts in CMD) |
| `docker-compose: command not found` | Compose not installed | `sudo dnf install -y docker-compose-plugin` |
| Updates on host don't reflect in container | Files were copied, not mounted | Mount volume: `-v /host:/container` or use Compose volumes |
| Host disk is 100% full | Unused containers/images | Run `docker system prune -a --volumes -f` |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Disk Space Exhaustion

> [!example] Ticket
> "Our DevOps build server is out of disk space and all pipelines are failing. Please check if Docker is eating up the drive."

**L1 Response:** Check disk space using `df -h`. Check Docker usage with `docker system df`.
**Escalation Trigger:** If clearing cache deletes required active images for other pipelines.
**L2 Resolution:**
- Run `docker system prune -a --volumes -f` to clear dangling images, unused containers, and build cache.
- Setup a cron job to routinely prune unused images.

### 🎫 Scenario 2: Container Port Conflict

> [!example] Ticket
> "I can't start the new web container. I'm getting a 'Bind for 0.0.0.0:8080 failed: port is already allocated' error."

**L1 Response:** Use `docker ps` or `netstat -tulpn | grep 8080` to find what is using the port.
**Escalation Trigger:** If the conflicting service is critical and cannot be stopped.
**L2 Resolution:** 
- Change the host port mapping in `docker run` or `docker-compose.yml`. E.g., change `-p 8080:3000` to `-p 8081:3000`.

---
## 🎤 Interview Questions

> [!question] Q1: What is the difference between a Docker Image and a Docker Container?
> **Answer:** A **Docker Image** is a read-only, immutable template containing the application code, libraries, and runtime instructions needed to build a container. A **Docker Container** is a running, writeable instance of that image. You can run multiple independent containers from a single image.

> [!question] Q2: Why would you use Multi-stage builds in a Dockerfile?
> **Answer:** Multi-stage builds shrink the size of the final production image. You use intermediate images with heavy build tools (compilers, SDKs) to compile the code, then copy ONLY the compiled binary into a clean, minimal runtime base image (like Alpine), excluding all unnecessary compiler overhead.

> [!question] Q3: How do you share persistent configuration data between three separate containers?
> **Answer:** Create a named Docker volume (`docker volume create shared-config-vol`) and mount it to the required paths in all three containers using Docker Compose. For safety, some containers can mount it as read-only (`ro`).

==**Exam Tip:** Understanding the difference between `COPY` (build-time) and `-v` volume mounts (run-time) is critical for Docker troubleshooting.==

---
## 🔗 Related Notes

- [[02-Operating-Systems/04-Linux-RHEL/L-19 Docker Basics for Linux Admins|L-19 Docker Basics for Linux Admins]] — OS installation steps on Rocky/RHEL
- [[05-Automation-and-Ticketing/14-DevOps-Basics/DEV-02 Kubernetes-Basics|DEV-02 Kubernetes-Basics]] — Container orchestration scaling models
- [[05-Automation-and-Ticketing/12-Ansible/ANS-02 Ansible for Linux Admins|ANS-02 Ansible for Linux Admins]] — Automating Docker host deployments
