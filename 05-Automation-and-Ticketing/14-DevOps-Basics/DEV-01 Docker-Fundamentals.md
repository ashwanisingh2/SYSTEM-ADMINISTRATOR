---
tags: [desktop-support, devops, containers, docker, L2]
aliases: [docker-fundamentals, dev-01]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #none
---

# DEV-01: Docker Fundamentals

> [note] Overview
> This note covers the fundamentals of Docker containerization within a DevOps workflow. It details the difference between VMs and containers, SPEC file-like Dockerfile authoring, multi-stage image optimization, Docker volumes, custom networks, and multi-container orchestration using Docker Compose.

---
## Concept Overview
- **What it is** — Docker is an open-source containerization platform that packages application code, libraries, dependencies, and system tools into a single, immutable package called a Docker Image. This image runs in a lightweight, isolated user-space environment called a Container.
- **Why it matters** — A common challenge in software deployment is configuration drift between test and production servers. Docker eliminates the "it worked on my machine" problem by ensuring the application runs identically in any environment because it carries its runtime environment with it.
- **Real job encounter** — Containerizing custom python or node applications, deploying multi-container stacks (like a web app connected to a database), pushing internal images to private registries, and optimizing host disk usage.
- **L1 vs L2 vs L3 responsibility split:**
  - **L1 Resolution**: Verify active containers using `docker ps`, restart failed containers (`docker restart`), inspect system log files (`docker logs`), and monitor host disk space.
  - **Escalation Trigger**: Escalate if container updates result in data loss on persistent volumes, or if containers experience port conflicts on the host.
  - **L2 Resolution**: Write standard `Dockerfiles` to package applications, compile images using `docker build`, tag and push images to registries, and map network port access.
  - **L3 Resolution**: Author complex `docker-compose.yml` configurations for multi-service environments, configure overlay networks, implement multi-stage builds to shrink image footprints, and secure Docker daemon processes.

---
## Technical Deep Dive

### 1. Virtual Machines vs. Containers
Understanding the resource boundary is essential for systems engineering:
- **Virtual Machines**: Abstract physical hardware. Each VM includes a virtual BIOS, virtual drivers, and a full Guest Operating System (consuming gigabytes of space and taking minutes to boot).
- **Containers**: Abstract the operating system kernel. They share the host OS kernel directly using Linux **Namespaces** (for isolation) and **Control Groups / cgroups** (for resource limitation). Containers start in milliseconds and have megabyte footprints.

### 2. Anatomy of a Dockerfile
A Dockerfile is the text configuration script describing the step-by-step assembly of a Docker Image:
- **`FROM`**: Sets the base parent image (e.g. `alpine`, `ubuntu`, `node:18`).
- **`WORKDIR`**: Creates and sets the active working directory inside the container.
- **`COPY` / `ADD`**: Copies files from the host machine directory into the container filesystem.
- **`RUN`**: Executes shell commands during the build phase (e.g. `npm install` or `apt-get install`).
- **`EXPOSE`**: Documents the container network port that the application listens on.
- **`ENV`**: Injects environment variables.
- **`CMD` / `ENTRYPOINT`**: Specifies the default executable command run when the container boots.

### 3. Multi-Stage Builds
In devops pipelines, building lightweight images is critical. A standard build includes build compilers (like JDK or Go SDK) which are not needed at runtime.
- **Multi-Stage Builds** use multiple `FROM` statements in a single Dockerfile.
- Stage 1 compiles the binary using heavy development tools.
- Stage 2 copies *only* the compiled binary into a lightweight runtime image (e.g., `alpine`), reducing the final production image size by up to 90%.

### 4. Docker Compose
A tool used to define and orchestrate multi-container applications. Instead of running multiple individual `docker run` commands, you define the entire application stack (services, databases, networks, volumes) in a single **`docker-compose.yml`** file and start it using `docker-compose up`.

---
## Step-by-Step Lab

> [!warning] Pre-requisites
> - A Linux server (Rocky Linux or Ubuntu) with Docker Engine installed.
> - A standard user account added to the `docker` group.
> - Outbound internet access to pull base images.

### Step 1: Create a Simple Web Application
We will create a basic Node.js application to containerize.

1. Create a project folder and navigate inside:
```bash
mkdir ~/node-project && cd ~/node-project
```
2. Create the file `server.js`:
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
We will write a Dockerfile to package this Node application.

1. Create `Dockerfile` inside the project root:
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
1. Build the image and tag it as `devops-web:1.0`:
```bash
docker build -t devops-web:1.0 .
```
**Expected Output:** Builds each layer. Ends with:
`Successfully built [Image-ID-Hash]`
`Successfully tagged devops-web:1.0`

2. Verify image exists locally:
```bash
docker images
```

### Step 4: Run and Test the Container
1. Launch the container in the background, mapping host port `8080` to container port `3000`:
```bash
docker run -d --name my-web-app -p 8080:3000 devops-web:1.0
```
2. Verify container state:
```bash
docker ps
```
3. Test web connection:
```bash
curl http://localhost:8080
```
**Expected Output:** `<h1>Docker DevOps Lab Complete!</h1>`

### Step 5: Orchestrate Multi-Container Stack using Docker Compose
We will spin up our web app alongside a Redis cache container.

1. Stop and remove the old container:
```bash
docker stop my-web-app && docker rm my-web-app
```
2. Create a `docker-compose.yml` file:
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
3. Start the entire application stack:
```bash
docker-compose up -d
```
**Expected Output:** Creates network `app-net`, pulls Redis, and starts both containers.
4. Verify running stack:
```bash
docker-compose ps
```

---
## Cheat Sheet / Quick Reference

| Command / Instruction | Scope | Purpose / Example |
|---|---|---|
| `docker build -t <tag> .` | CLI | Compiles an image using the local Dockerfile |
| `docker run -d -p 80:80 <img>`| CLI | Runs container in detached background mode mapping ports |
| `docker logs <container>` | CLI | Dumps application standard output logs |
| `docker exec -it <name> sh` | CLI | Opens an interactive terminal shell inside container |
| `docker-compose up -d` | Compose | Starts all services in `docker-compose.yml` |
| `docker-compose down` | Compose | Stops and removes all containers, networks, and volumes |
| `FROM` | Dockerfile | Sets parent base image for subsequent layers |
| `COPY` | Dockerfile | Copies local host files to container filesystem |
| `WORKDIR` | Dockerfile | Sets execution directory inside container |

---
## Troubleshooting Table

| Problem | Cause | Fix |
|---|---|---|
| Docker commands fail with: "Permission denied." | Current user is not in the `docker` security group. | Add user to group: `sudo usermod -aG docker $USER`. Log out and log back in to apply group changes. |
| Container starts but exits immediately. | The container has no active foreground process running (it completed its CMD). | Ensure the application runs in the foreground (e.g. do not run scripts in daemon mode inside CMD). |
| Docker Compose fails: "docker-compose: command not found." | The Docker Compose binary is not installed on the system. | Download compose binary: `sudo dnf install -y docker-compose-plugin` or download binary from GitHub. |
| Host files change but updates do not reflect inside the running container. | The files were copied during build, not mounted dynamically. | Mount the host directory as a volume during run: `-v /host/path:/container/path` or configure volumes in Compose. |
| Host disk is 100% full due to Docker files. | Accumulated unused containers, dangling images, and build cache. | Clean up local disk: `docker system prune -a --volumes -f`. |

---
## Interview Questions

> [!question] L1 Question
> **Q:** What is the difference between a Docker Image and a Docker Container?
> **A:** A **Docker Image** is a read-only, immutable template containing the application code, libraries, and runtime instructions needed to build a container. A **Docker Container** is a running, writeable instance of that image. You can run multiple independent containers from a single image.

> [!question] L2 Question
> **Q:** Why would you use Multi-stage builds in a Dockerfile?
> **A:** Multi-stage builds are used to optimize and shrink the size of the final production Docker image. They allow you to use temporary intermediate images containing heavy build tools (like compilers, SDKs, and build scripts) to compile the code. The final stage copies only the compiled production binary into a clean, minimal runtime base image (like Alpine), excluding all unnecessary compiler overhead.

> [!question] L3/Scenario Question
> **Q:** You are deploying a containerized application that needs to share persistent configuration data between three separate containers on the same host. How do you configure this?
> **A:** 
> - **Situation:** Sharing persistent configurations across multiple local containers.
> - **Task:** Design a shared volume access architecture.
> - **Action:** 
>   1. **Create Named Volume**: I will create a named Docker volume managed by the daemon: `docker volume create shared-config-vol`.
>   2. **Configure Volume Mounts**: When launching the containers, I will mount this single named volume to the required path in all three containers. In Docker Compose:
>      ```yaml
>      services:
>        app1:
>          image: myapp:1.0
>          volumes:
>            - shared-config-vol:/etc/app/config
>        app2:
>          image: myhelper:1.0
>          volumes:
>            - shared-config-vol:/data/config:ro # Read-only mount for safety
>      volumes:
>        shared-config-vol:
>      ```
> - **Result:** All three containers access the same folder structure, changes are persisted if containers restart, and read-only limits can protect data integrity.

---
## Seedha Simple Mein
*Seedha simple mein: Docker ek software packaging tool hai jo application code aur dependencies ko ek image me convert kar deta hai. VM ke opposite, containers direct host kernel share karte hain, jisse ye fast aur lightweight ho jate hain. Multi-container applications ko single command se run karne ke liye `docker-compose` use kiya jata hai.*

---
## Related Notes
- [[02-Operating-Systems/04-Linux-RHEL/L-19 Docker Basics for Linux Admins|L-19 Docker Basics for Linux Admins]] — OS installation steps on Rocky/RHEL.
- [[05-Automation-and-Ticketing/14-DevOps-Basics/DEV-02 Kubernetes-Basics|DEV-02 Kubernetes-Basics]] — Container orchestration scaling models.
- [[05-Automation-and-Ticketing/12-Ansible/ANS-02 Ansible for Linux Admins|ANS-02 Ansible for Linux Admins]] — Automating Docker host deployments.

---
*Tags: #desktop-support #devops #containers #docker #L2*
