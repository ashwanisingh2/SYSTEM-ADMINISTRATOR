---
tags: [desktop-support, linux, rhel, L2]
aliases: [l-19-docker-basics-for-linux-admins, l-19]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #rhcsa
---

# L-19: Docker Basics for Linux Admins

**Verification:**
- [ ] Configure the official Docker repository on a Linux host
- [ ] Install Docker CE and enable the docker systemd service
- [ ] Run a containerized application (e.g., NGINX) mapping host port to container port
- [ ] Demonstrate storage persistence using Docker volumes or bind mounts
- [ ] Inspect container resource usage and view stdout log streams

> [!abstract] Overview
> This note introduces containerization concepts using Docker on Linux. It explains the differences between Virtual Machines and containers, explores the Docker architecture, details essential container management lifecycles, covers volume mapping, networking, and includes practical administration commands.

---

---
## Concept Overview
- **What it is** — Docker is an open-source platform that automates the deployment of applications inside lightweight, portable, and self-contained environments called containers.
- **Why it matters for a support engineer** — Modern applications are rarely installed directly on server operating systems. They are packaged as Docker containers, which bundle the app along with all its libraries and dependencies. This ensures that the application runs identically on any environment.
- **Where you encounter this in real job** — Deploying web servers, database systems, testing software stacks, or running automation scripts in isolated environments without messing up host system packages.
- **L1 vs L2 vs L3 responsibility split for this topic:**
  - **L1**: Start/stop Docker containers, view running containers using `docker ps`, check basic container status, and gather container log outputs for escalation.
  - **L2**: Install and configure Docker engine, write simple Dockerfiles, manage container networking, clean up unused images/volumes, and configure port mappings.
  - **L3**: Set up persistent storage volumes, secure Docker daemon (TLS setup), optimize image sizes, set up resource limits, and orchestrate basic multi-container setups using Docker Compose.


---

---
## Technical Deep Dive
## Real-World Analogy
Think of a physical cargo ship:
- Before shipping containers, cargo was loose and loaded individually (barrels of oil next to boxes of electronics). If one leaked, it ruined the others. This is like installing multiple apps on a single Linux OS, leading to library conflicts (DLL/dependency hell).
- **Docker Containers** are standard shipping containers. Each container is sealed, isolated, and holds a specific item (app + dependencies). The cargo ship (Linux Kernel/Host OS) doesn't care what's inside the container; it can carry thousands of them efficiently.

---

### 1. Virtual Machines vs. Containers
- **Virtual Machines (VMs)**: Include a full Guest OS, virtual device drivers, and a hypervisor layer. They have high resource footprints (GBs of RAM/Disk) and take minutes to boot.
- **Containers**: Share the host Linux Kernel directly via kernel namespaces (for isolation) and control groups (cgroups, for resource limits). They are extremely lightweight (MBs of size), share host resources, and start in milliseconds.

```
+-----------------------------+     +-----------------------------+
|    App A   |    App B       |     |    App A   |    App B       |
+------------+------------+---+     +------------+------------+---+
|  Bin/Libs  |  Bin/Libs  |         | Container  | Container  |
+------------+------------+         +------------+------------+
|  Guest OS  |  Guest OS  |         |       Docker Engine         |
+------------+------------+         +-----------------------------+
|         Hypervisor          |     |          Host OS            |
+-----------------------------+     +-----------------------------+
|          Host OS            |     |        Host Kernel          |
+-----------------------------+     +-----------------------------+
|          Hardware           |     |          Hardware           |
+-----------------------------+     +-----------------------------+
|      Virtual Machines       |     |         Containers          |
+-----------------------------+     +-----------------------------+
```

### 2. Docker Core Components
- **Docker Daemon (`dockerd`)**: The background system service managing Docker objects (images, containers, networks, volumes).
- **Docker Client (`docker`)**: The CLI tool used by administrators to send commands to the daemon.
- **Docker Image**: A read-only template containing instructions to build a container (packaged operating system layers and application code).
- **Docker Container**: A runnable instance of a Docker image. It is the writeable layer added on top of the read-only image layers.
- **Docker Registry (e.g., Docker Hub)**: A central storage library where developers upload and download images.

### 3. Docker Networking Modes
- **Bridge (Default)**: Creates a private virtual network inside the host (usually `172.17.0.0/16`). Containers get internal IPs, and external traffic is mapped via NAT/IPTables port forwarding.
- **Host**: Removes isolation between the container and the host. The container binds directly to host IP and ports.
- **None**: Disables all networking interfaces except the loopback device (`127.0.0.1`), ensuring maximum network isolation.

### 4. Docker Storage Drivers & Persistence
By default, data created inside a container is ephemeral (gets destroyed when the container is deleted). To persist data:
- **Volumes**: Managed by Docker in a dedicated directory on the host (`/var/lib/docker/volumes/`). Safest and easiest way to persist data.
- **Bind Mounts**: Maps any directory or file on the host filesystem directly into the container (e.g., mapping host `/var/www/` to container `/usr/share/nginx/html`).

---

---

### Enterprise RHEL Service & Network Configurations

#### 1. Custom Systemd Service Creation
Create a custom systemd service configuration file `/etc/systemd/system/myapp.service`:
```ini
[Unit]
Description=My Custom Enterprise Application
After=network.target

[Service]
Type=simple
User=sysadmin
ExecStart=/usr/bin/python3 /opt/myapp/server.py
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
Enable and start the service:
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now myapp.service
```

#### 2. Log Rotation & Rsyslog Configuration
Configure log rotation rule in `/etc/logrotate.d/myapp` for automatic log cleaning:
```text
/var/log/myapp/*.log {
    daily
    rotate 7
    compress
    delaycompress
    missingok
    notifempty
    create 0660 sysadmin sysadmin
}
```
Define Rsyslog rule in `/etc/rsyslog.d/50-myapp.conf` to redirect application logs to a dedicated file:
```text
if $programname == 'myapp' then /var/log/myapp/syslog.log
& stop
```
Restart Rsyslog service:
```bash
sudo systemctl restart rsyslog
```

#### 3. Network Bonding & Teaming (LACP Link Aggregation)
Create a network team interface config file `/etc/sysconfig/network-scripts/ifcfg-team0` (RHEL standard):
```text
DEVICE=team0
DEVICETYPE=Team
BOOTPROTO=none
IPADDR=192.168.1.100
PREFIX=24
GATEWAY=192.168.1.1
ONBOOT=yes
TEAM_CONFIG='{"runner": {"name": "lacp"}}'
```
Bind slave physical interfaces (e.g., `eth1`) to the team interface:
```text
# /etc/sysconfig/network-scripts/ifcfg-eth1
DEVICE=eth1
ONBOOT=yes
TEAM_MASTER=team0
DEVICETYPE=TeamPort
```

---
## Step-by-Step Lab
> [!warning] Pre-requisites
> - A Rocky Linux 9 or RHEL 9 virtual machine (`Rocky-App01` with IP `192.168.10.30`).
> - Sudo or root privileges.
> - Outbound Internet connectivity to download repository metadata and images.

### Step 1: Remove Legacy Container Engines
RHEL/Rocky systems come preinstalled with `podman` or legacy Docker packages. Remove them first:
```bash
sudo dnf remove -y docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-engine podman runc
```
**Expected Output:** `[Packages removed or No match for argument]`

### Step 2: Configure Official Docker Repository and Install Engine
1. Install utility packages:
```bash
sudo dnf install -y yum-utils
```
2. Add the official Docker CE stable repository:
```bash
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```
3. Install Docker Engine, CLI, and containerd:
```bash
sudo dnf install -y docker-ce docker-ce-cli containerd.io
```
**Expected Output:** Successful installation of docker-ce packages.

### Step 3: Start and Enable Docker Service
1. Enable and start the systemd service:
```bash
sudo systemctl enable --now docker
```
2. Verify service status:
```bash
sudo systemctl status docker --no-pager
```
**Expected Output:** `Active: active (running)...`

3. (Optional) Allow non-root users to run docker commands:
```bash
sudo usermod -aG docker $USER
# Log out and log back in for changes to take effect
```

### Step 4: Run an NGINX Web Server Container with Persistent Storage
1. Create a directory on the host for static web pages (Bind Mount source):
```bash
mkdir -p ~/web_data
echo "<h1>Hello from Docker Container!</h1>" > ~/web_data/index.html
```
2. Run NGINX container mapping port 8080 on the host to port 80 in the container:
```bash
docker run -d --name my-web-server -p 8080:80 -v ~/web_data:/usr/share/nginx/html:ro nginx:latest
```
**Expected Output:**
```text
Unable to find image 'nginx:latest' locally
latest: Pulling from library/nginx
...
Digest: sha256:...
Status: Downloaded newer image for nginx:latest
[Container-ID-Hash]
```

3. Verify running containers:
```bash
docker ps
```
**Expected Output:** Shows `my-web-server` running with port mapping `0.0.0.0:8080->80/tcp`.

4. Test connectivity from host command line:
```bash
curl http://localhost:8080
```
**Expected Output:** `<h1>Hello from Docker Container!</h1>`

### Step 5: Clean Up Resources
1. Stop and remove the container:
```bash
docker stop my-web-server
docker rm my-web-server
```
2. Remove downloaded images to free space:
```bash
docker rmi nginx:latest
```

---

---
## Cheat Sheet / Quick Reference
| Command / Setting | Purpose | Example |
|---|---|---|
| `docker run` | Creates and starts a container from an image | `docker run -d --name web -p 80:80 nginx` |
| `docker ps` | Lists running containers | `docker ps` |
| `docker ps -a` | Lists all containers (running and stopped) | `docker ps -a` |
| `docker stop` | Gracefully stops a running container | `docker stop my-container` |
| `docker start` | Starts a stopped container | `docker start my-container` |
| `docker rm` | Deletes a stopped container | `docker rm my-container` |
| `docker images` | Lists all downloaded Docker images | `docker images` |
| `docker rmi` | Deletes a Docker image from host | `docker rmi nginx:latest` |
| `docker logs` | Fetches container log outputs | `docker logs my-container` |
| `docker exec -it` | Executes an interactive shell inside a running container | `docker exec -it my-container /bin/bash` |
| `docker inspect` | Returns detailed low-level info about objects | `docker inspect my-container` |
| `docker system prune -a` | Cleans up unused containers, networks, and images | `docker system prune -a --volumes` |

---

---
## Troubleshooting
| Problem | Cause | Fix |
|---|---|---|
| Error: "Cannot connect to the Docker daemon. Is the docker daemon running?" | The docker systemd service is stopped or not enabled. | Run `sudo systemctl start docker` to start the daemon, and `sudo systemctl enable docker` for boot persistence. |
| Permission denied when running docker commands without sudo. | Current user is not a member of the `docker` security group. | Add user to group: `sudo usermod -aG docker $USER`. Close terminal and log in again to apply groups. |
| Error: "Bind for 0.0.0.0:80 failed: port is already allocated". | Another process (e.g., Apache/httpd) is already listening on the requested host port. | Find process: `sudo ss -tulpn \| grep :80`. Stop conflicting process or change host port mapping (e.g., `-p 8080:80`). |
| Container exits immediately after starting. | The container has no foreground process running, or application inside encountered a fatal error. | Inspect exit logs using `docker logs <container-id>` to locate the configuration/code error. |
| Host files are updated, but the change does not reflect inside the running container's bind mount. | SELinux is blocking container access to host directory files. | Append `:z` flag to allow shared SELinux labels: `-v /host/path:/container/path:z`. |

---

---
## Interview Questions
> [!question] L1 Question
> **Q:** What is the difference between `docker run` and `docker start` commands?
> **A:** `docker run` is used to create and start a **new** container from a specified image. It downloads the image if not present locally, creates the writeable container layer, and starts it. `docker start` is used to start an **existing, stopped** container that was already created using `docker run` or `docker create`. It does not overwrite files or modify configuration.

> [!question] L2 Question
> **Q:** Explain Docker volume mapping. What is the difference between a volume and a bind mount?
> **A:** Both are used to persist data outside the lifecycle of containers.
> - **Volumes** are created and managed completely by Docker. They are stored in `/var/lib/docker/volumes/` on the host, and non-Docker processes should not modify this directory.
> - **Bind Mounts** map any arbitrary directory or file on the host system to a path inside the container. This is useful for sharing development files or configurations, but makes containers dependent on host filesystem layouts.

> [!question] L3/Scenario Question
> **Q:** A containerized database application on a Rocky Linux host is experiencing extremely slow read/write performance. Host CPU and RAM are normal. How would you investigate and optimize storage performance for this container?
> **A:** 
> 1. **Check Storage Type**: Ensure the database is using **Docker Volumes** or **Bind Mounts** to persist data. If the app writes data inside the container's writeable layer (using storage drivers like overlay2), it incurs a severe performance overhead due to the copy-on-write mechanism.
> 2. **Monitor I/O Bottlenecks**: Run `iostat -xz 1 5` on the host to monitor disk utilization (`%util` and `await`). Run `docker stats` to verify if the container is hitting any disk constraints.
> 3. **SELinux Auditing**: If SELinux is set to Enforcing, check `/var/log/audit/audit.log` for denials, and ensure the volume mount contains the correct context labels (`:z` or `:Z`).
> 4. **Hardware Bottlenecks**: Ensure volumes are hosted on high-performance storage media (e.g., NVMe SSDs rather than slow HDDs) and mount directories with the `noatime` option on the host filesystem.

---

---
## Seedha Simple Mein
*Seedha simple mein: Docker ek software hai jo applications ko unki sabhi dependencies ke saath ek packet (container) mein band kar deta hai. Yeh container kisi bhi Linux server par bina kisi compatibility issue ke turant chal jata hai, bilkul waise hi jaise ek packaged box ko kahin bhi transport kiya ja sake.*

---
## Related Notes
- [[02-Operating-Systems/04-Linux-RHEL/L-08 Services and Systemd|L-08 Services and Systemd]] — Managing the Docker daemon systemd service unit.
- [[02-Operating-Systems/04-Linux-RHEL/L-12 Network Configuration in Linux|L-12 Network Configuration in Linux]] — Explains IP routing and bridging interfaces utilized by Docker.
- [[02-Operating-Systems/04-Linux-RHEL/L-14 Linux Security Hardening|L-14 Linux Security Hardening]] — SELinux configuration and policy management.

---
*Tags: #linux #rhel #docker #containers #devops #cert-rhcsa*
