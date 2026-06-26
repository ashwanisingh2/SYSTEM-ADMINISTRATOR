---
tags: [linux, rhel, docker, containers, basics]
aliases: [docker-basics, linux-docker]
created: 2026-06-26
status: #complete
difficulty: #intermediate
cert-relevant: #none
---

> [!NOTE|color-green]
> 🐧 **LINUX / RHEL**

`#complete` `#intermediate` `#none`

# L-19: Docker Basics for Linux Admins

> [!abstract] Overview
> In today's IT infrastructure, containerization is essential. This note covers the fundamentals of Docker from a Linux System Administrator's perspective. *Docker ek aisi technology hai jisne application deployment ko bilkul aasan aur standardized bana diya hai.* A support engineer needs to understand Docker because many legacy applications are being migrated to containers, and troubleshooting containerized apps is a core daily task in modern operations.

---
## 🧠 Concept Overview

- **What it is** — Docker is an open-source platform that automates the deployment, scaling, and management of applications inside isolated environments called containers. *Docker aapki application aur uski saari dependencies ko ek chhote se container mein pack kar deta hai taaki wo kisi bhi machine par seamlessly chal sake.*
- **Why it matters** — It solves the classic "it works on my machine" problem. It ensures environment consistency across development, testing, staging, and production environments. *Real job mein ye zaroori hai taaki developers aur ops team ke beech environments ko lekar conflicts na hon.*
- **Where you see this** — Microservices architectures, CI/CD pipelines, modern web application deployments, and cloud-native application stacks. *Har badi company aajkal apne apps ko Docker containers ya Kubernetes (jo under the hood containers use karta hai) par run kar rahi hai.*

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic task — Check container status (`docker ps`), start/stop containers, and check basic logs. *Sirf ye dekhna ki container chal raha hai ya band ho gaya aur basic error messages capture karna.* |
| **L2** | Configure, fix, escalate — Build images from Dockerfiles, manage volumes, network troubleshooting, and writing basic docker-compose files. *Container kyu fail hua ye dhoondhna, ports map karna aur issues fix karna.* |
| **L3** | Architecture, design — Design container security policies, optimize Dockerfile builds, multi-host networking, and migration to orchestration tools like Kubernetes. *Poora enterprise architecture design karna aur resource limits optimize karna.* |

> [!tip] Seedha Simple Mein
> *Docker ek virtual machine ki tarah hai, par bahut lightweight. Isme poora operating system nahi hota, sirf wahi cheezein hoti hain jo app ko chalane ke liye chahiye. Isliye ye virtual machine ke seconds vs minutes mein start ho jata hai aur system resources kam khata hai.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Shipping Containers in a Port** are like **Docker Containers on a Server** because...
>
> - Pehle ships mein saaman alag-alag aakar ke baksiyo mein aata tha, jisse load/unload karna mushkil tha. Shipping industry ne sab kuch ek standard size ke box (container) mein daal diya. Docker ne software ke liye wahi kiya.
> - Aap container ke andar car rakhein, khana rakhein ya electronics, ship (server) ko fark nahi padta. Woh sirf standard container uthata hai. Waise hi, server ko fark nahi padta container ke andar Java app hai, Node.js hai ya Python app, wo bas usko standard tarike se run karta hai.
> - *Jaise shipping container ek jagah se doosri jagah safely transfer hota hai bina andar ka saaman toote, waise hi Docker container dev laptop se production server mein bina tute (without missing dependencies) chala jata hai.*

---
## 🔬 Technical Deep Dive

### 1. Docker Architecture

> [!info] Key Concept
> Docker follows a client-server architecture pattern. The Docker client talks to the Docker daemon, which does the heavy lifting of building, running, and distributing your Docker containers.

**Core Components:**
1. **Docker Daemon (`dockerd`)**: The background service running on the host OS that manages building, running, and distributing Docker containers. It listens for Docker API requests. *Yeh main engine hai jo server ke background mein chal kar sab control karta hai.*
2. **Docker Client (`docker`)**: The command-line tool that allows users to interact with the daemon. When you type `docker run`, the client sends this command to `dockerd`. *Yeh wo CLI tool hai jiske through hum (admins) commands dete hain.*
3. **Docker Images**: Read-only templates with instructions for creating a Docker container. Often based on another image, with some additional customization. *Yeh ek blueprint ya CD-ROM ki tarah hai, jisse container banta hai.*
4. **Docker Containers**: A runnable instance of an image. You can create, start, stop, move, or delete a container using the Docker API or CLI. *Yeh image ka memory mein chalta hua (live) form hai.*
5. **Docker Registry**: Stores Docker images (e.g., Docker Hub, AWS ECR). *Yeh ek app store ki tarah hai jahan se custom ya public images download (pull) ya upload (push) ki jati hain.*

> [!danger] Common Mistake
> Confusing an Image with a Container. An Image is the blueprint (not running), while a Container is the running instance of that blueprint. *Image ek recipe (vidhi) hai, aur container us recipe se bana hua khana (dish) hai.*

### 2. Images vs. Containers

- **Image**: Static, immutable (cannot be changed). Made up of multiple read-only filesystem layers.
- **Container**: Dynamic. Takes an image and adds a thin read-write layer on top of the image layers. Any modifications made while the container is running are written to this thin top layer.

### 3. What is a Dockerfile?

A `Dockerfile` is a simple text document that contains all the commands a user could call on the command line to assemble an image.
*Dockerfile ek script ya instructions ki list hoti hai jisme hum likhte hain ki image step-by-step kaise banani hai.*

Example instructions:
- `FROM ubuntu:22.04` — Defines the Base OS
- `RUN apt-get update -y` — Runs a command during the image build process
- `COPY . /app` — Copies source code files from host to the image
- `CMD ["python3", "app.py"]` — The default command to run when the container starts

### 4. Docker Volumes & Data Persistence

By default, data created inside a container is stored in the container's writable layer. When the container is deleted, that data is permanently lost. To persist data (like database records), we use **Volumes**.
- *Agar aap MySQL database Docker par run kar rahe hain, toh volume map karna zaroori hai warna container delete hone par aapka saara company data ud jayega.*

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - A Linux server (RHEL 8/9, Rocky Linux, or Ubuntu)
> - Root or sudo privileges
> - Active internet connection to pull images

### Step 1: Install Docker CE (RHEL / Rocky Linux)

```bash
# Update the system packages
sudo dnf update -y

# Add the official Docker repository
sudo dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo

# Install Docker Community Edition and dependencies
sudo dnf install docker-ce docker-ce-cli containerd.io -y

# Start and enable the Docker daemon to run on boot
sudo systemctl start docker
sudo systemctl enable docker
```

> [!success] Expected Output
> ```
> Created symlink /etc/systemd/system/multi-user.target.wants/docker.service → /usr/lib/systemd/system/docker.service.
> ```

### Step 2: Run Your First Web Container

```bash
# Yeh command nginx web server ka container run karta hai
# -d: Detached mode (background mein chalega, terminal freeze nahi hoga)
# -p 8080:80: Host ka port 8080 container ke port 80 se map (bind) karega
# --name: Container ko ek readable naam dega
sudo docker run -d -p 8080:80 --name my-nginx-server nginx:latest
```

> [!success] Expected Output
> ```
> Unable to find image 'nginx:latest' locally
> latest: Pulling from library/nginx
> [...]
> Digest: sha256: e209ac2f37c70c1e0e9873a5f7231e91dce8472011ebfc47d...
> Status: Downloaded newer image for nginx:latest
> a1b2c3d4e5f6g7h8i9j0... (This is the full Container ID)
> ```

### Step 3: Verify the Container is Running

```bash
# Check running containers
sudo docker ps
```

*Ab aap apne system ke browser mein `http://<server-ip>:8080` kholenge toh aapko Nginx ka default welcome page dikhai dega.*

### Step 4: Interacting with the Container

```bash
# View the live logs of the container
sudo docker logs my-nginx-server

# Go inside the running container using an interactive bash shell
sudo docker exec -it my-nginx-server /bin/bash

# (Inside container) check the OS version
cat /etc/os-release
# (Inside container) exit to return to host
exit
```

### Step 5: Clean up (Important for Housekeeping)

```bash
# Stop the running container gracefully
sudo docker stop my-nginx-server

# Remove the stopped container
sudo docker rm my-nginx-server

# Remove unused data (images, stopped containers, unused networks)
sudo docker system prune -f
```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `docker run` | Naya container create aur start karta hai ek command mein. | `docker run -d nginx` |
| `docker ps` | Sirf chalte hue (running) containers ki list dikhata hai. | `docker ps` |
| `docker ps -a` | Saare containers dikhata hai (stopped, exited, aur running). | `docker ps -a` |
| `docker stop` | Chalte hue container ko gracefully (SIGTERM bhej kar) band karta hai. | `docker stop my-web` |
| `docker rm` | Band container ko server se delete karta hai. | `docker rm my-web` |
| `docker rmi` | Docker image ko host disk se delete karta hai. | `docker rmi nginx:latest` |
| `docker images` | Host par downloaded saari images list karta hai sizes ke sath. | `docker images` |
| `docker logs` | Container ke stdout/stderr logs check karne ke liye. | `docker logs container_id` |
| `docker exec -it` | Chalte hue container ke andar shell (CLI) access lene ke liye. | `docker exec -it web bash` |
| `docker build` | Dockerfile ko read karke nayi custom image banata hai. | `docker build -t myapp:v1 .` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| `Cannot connect to the Docker daemon` | Docker service run nahi kar rahi ya current user ko permissions nahi hai. | `sudo systemctl start docker` ya user ko `docker` group mein add karein. |
| `Bind for 0.0.0.0:80 failed: port is already allocated` | Host OS par wo port pehle se koi aur application (jaise Apache HTTPD) use kar raha hai. | Doosra free port map karein: `-p 8080:80` ya running conflicting app ko stop karein. |
| `Container starts and immediately stops` | Container ki main foreground process (PID 1) khatam ho gayi ya fail ho gayi. | `docker logs <container_id>` se error check karein. Background tasks directly nahi chalte. |
| `No space left on device` | Docker images/containers/logs ne `/var/lib/docker` ka disk partition poora bhar diya hai. | Unused data aur dangling images delete karein: `docker system prune -a --volumes` |
| `exec: "bash": executable file not found in $PATH` | Container ke andar bash shell install hi nahi hai (e.g. Alpine linux based image). | `bash` ki jagah `/bin/sh` try karein: `docker exec -it <id> sh` |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Developer can't access application inside container

> [!example] Ticket
> "Hi IT Support, I started my Node.js application container successfully, but I cannot access it from my browser on port 3000."

**L1 Response:** Check if the container is actually running using `docker ps`. Verify if the port mapping was done correctly.
**Escalation Trigger:** If the container is running and port mapping is visibly correct in `docker ps`, but it's still inaccessible over network, pass to L2.
**L2 Resolution:**
1. Run `docker ps` and check the ports column for `-p 3000:3000`. If it just says `3000/tcp` without the host mapping, the developer forgot the `-p` flag during `docker run`.
2. Check container application logs: `docker logs <container_id>`. The app inside might be crashing.
3. *Aksar log `-p` flag lagana bhool jate hain, ya host ki local firewall (firewalld/iptables) us port ko block kar rahi hoti hai.* Run `netstat -tulnp | grep 3000` to verify the host is actively listening.

### 🎫 Scenario 2: Container keeps crashing (CrashLoopBackOff pattern)

> [!example] Ticket
> "The production backend API container keeps restarting every few seconds. Our service monitor is alerting us to an outage."

**L1 Response:** Identify the exact container name/ID. Capture the recent logs using `docker logs --tail 50 <container_id>` and attach to the ticket. Send to L2.
**Escalation Trigger:** Immediate escalation to L2/L3 as this is a production application outage.
**L2 Resolution:**
1. Inspect the captured logs. E.g., `docker logs backend-api`.
2. Found error: `Error: ENOSPC: no space left on device` or perhaps a database connection timeout.
3. Check host disk space: `df -h`. `/var/lib/docker` partition is 100% full.
4. Run `docker system prune` (with extreme caution in prod!) to clear old cache, or expand the LVM disk volume to fix the issue. *Disk space full hona Linux admins ke liye sabse common issue hai production Docker environments mein.*

### 🎫 Scenario 3: Lost database data after container replacement

> [!example] Ticket
> "I accidentally removed the old MySQL container and created a new one with the same image. Now all my database tables and user data are completely gone! Please restore it ASAP."

**L1 Response:** Calm the user down. Verify what exact commands the user ran from their bash history. Ask if they explicitly used a Docker Volume for the database storage.
**Escalation Trigger:** Data loss issues always go directly to L2/L3 for immediate recovery attempts.
**L2/L3 Resolution:**
1. Check if a volume was mapped in the original `docker run` command history.
2. If NO volume was mapped (`-v` flag missing), the data was being written directly to the container's ephemeral read-write layer. When `docker rm` was executed, that layer was permanently destroyed. *Agar volume map nahi kiya gaya tha, toh system se data recover karna lagbhag namumkin hai.*
3. If they have a separate backup (e.g. AWS EBS snapshot or a manual `mysqldump`), restore from that backup. Educate the developer strongly on always using Docker Volumes for persistent stateful data.

---
## 🎤 Interview Questions

> [!question] Q1: What is the primary difference between a Docker Image and a Docker Container?
> **Answer:** An Image is a read-only, immutable template containing the application code, runtime, system tools, libraries, and dependencies. A Container is a running, interactive instance of that image, which has a thin read-write layer added on top by the Docker engine.
> ==**Exam Tip:** Image = The static Blueprint/Class, Container = The running House/Object.==

> [!question] Q2: How does Docker fundamentally differ from traditional Virtual Machines (VMs)?
> **Answer:** VMs virtualize the underlying hardware and require a full, heavy Guest OS for each running instance. Docker, on the other hand, virtualizes the OS kernel; multiple containers share the exact same host OS kernel. This makes Docker containers extremely lightweight, incredibly fast to boot (seconds instead of minutes), and far less resource-intensive regarding CPU/RAM.
> ==**Exam Tip:** Docker shares the OS kernel; VMs bring their own separate and bulky Guest OS.==

> [!question] Q3: What happens to the data stored inside a container when the container stops or is deleted?
> **Answer:** If the container is merely stopped (`docker stop`), the data remains intact in its internal read-write layer. However, if the container is removed (`docker rm`), all data not stored in an external Docker Volume is permanently deleted. *Hamesha persistent data (like logs, databases) ke liye external volumes map karna chahiye.*
> ==**Exam Tip:** To persist data permanently independent of the container's lifecycle, you must use Docker Volumes or Bind Mounts.==

> [!question] Q4: How do you enter the shell of an already running Docker container for live troubleshooting?
> **Answer:** You use the `docker exec` command. For example, `docker exec -it <container_name_or_id> /bin/bash` (or `/bin/sh` if it's an Alpine-based image where bash is not available). The `-i` flag stands for interactive, and `-t` allocates a pseudo-TTY.
> ==**Exam Tip:** Remember that the `-it` flags are strictly required together to get an interactive, usable terminal session inside the container.==

---
## 🔗 Related Notes

- [[L-18 Linux Networking Basics|L-18 Linux Networking Basics]] — Essential for understanding how Docker bridging and routing works under the hood.
- [[L-20 Advanced Docker and Docker Compose|L-20 Advanced Docker and Docker Compose]] — The next logical step after mastering these fundamentals.
- [[L-15 Linux Storage and LVM|L-15 Linux Storage and LVM]] — Very useful when dealing with Docker Volumes and managing host disk space issues.
- [[L-10 Linux Process Management|L-10 Linux Process Management]] — Helps understand how container processes are just isolated host processes.
