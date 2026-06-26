---
tags: [desktop-support, lab, containers, docker, L1]
aliases: [docker-home-lab, lab-04]
created: 2026-06-25
status: #complete
difficulty: #beginner
cert-relevant: #none
---

# LAB-04: Docker Home Lab

> [!note] Overview
> This lab guide walks through building a containerized home lab on a local machine or single server. It covers setting up Docker, deploying management tools (Portainer, Nginx Proxy Manager), running utility containers (Pi-hole), and configuring volume persistence and networks.

---
## Concept Overview
- **What it is** — A Docker Home Lab is a local testing environment where a support engineer runs multiple self-hosted services (like DNS, proxy managers, and dashboards) inside lightweight Docker containers.
- **Why it matters** — Running a home lab is the single most effective way to gain hands-on systems engineering experience. It provides exposure to Linux administration, networking protocols (DNS, routing, HTTP proxy), Docker containerization, and troubleshooting, all without risk to production environments.
- **Real job encounter** — Setting up local test servers, validating application network connections, testing container configurations, and practicing infrastructure orchestration.
- **L1 vs L2 vs L3 responsibility split:**
  - **L1 Resolution**: Deploy pre-configured docker-compose stacks, start/stop containers, view container resource graphs, and check system logs.
  - **Escalation Trigger**: Escalate if containers fail to start due to port conflicts on the host, if persistent volume mounts fail, or if virtual networks become unreachable.
  - **L2 Resolution**: Write custom compose files, map persistent volumes to local folders, resolve DNS queries via local resolvers, and bind reverse proxies.
  - **L3 Resolution**: Implement container resource limits, secure internal container endpoints, run local container registries, and set up Docker hosts.

*Seedha simple mein: Docker Home Lab ek personal learning system hai jise aap apne local computer ya server par banate hain. Is lab me aap Portainer (Web UI dashboard), Pi-hole (DNS server/ad-blocker), aur Nginx Proxy Manager ko ek sath chalana seekhte hain, taaki networking aur virtualization ke concepts strong ho sakein.*

---
## Technical Deep Dive

### 1. Lab Architecture
The lab uses a single Docker Host (Ubuntu/Rocky Linux or Windows/macOS running WSL2) and isolates services using custom Docker networks:
- **Portainer**: Provides a web-based GUI to manage containers, images, volumes, and networks.
- **Nginx Proxy Manager (NPM)**: Acts as a reverse proxy, routing incoming domain traffic to the correct container ports.
- **Pi-hole**: Serves as a local DNS resolver and network-wide ad blocker.

### 2. Container Networks and Port Conflicts
- **Bridge Network**: The default driver. Containers get internal IPs (e.g., `172.18.0.x`) and can communicate. Ports are mapped to the host (e.g., `-p 8080:80`).
- **Port Conflicts**: Only one service can bind to a specific port on the host IP address (e.g., port `80` cannot be shared by Apache and Nginx directly on the host interface).

---
## Step-by-Step Lab

> [!warning] Pre-requisites
> - A computer running Ubuntu Server 22.04 LTS, Rocky Linux, or Windows with WSL2.
> - Docker Engine and Docker Compose plugin installed.
> - Sudo administrative access on the host.

### Step 1: Prepare the Directory Structure
1. Log in to your server and create a directory hierarchy for persistent configuration files:
```bash
mkdir -p ~/homelab/portainer
mkdir -p ~/homelab/nginx-proxy-manager/data
mkdir -p ~/homelab/nginx-proxy-manager/letsencrypt
mkdir -p ~/homelab/pihole/config
mkdir -p ~/homelab/pihole/dnsmasq
cd ~/homelab
```

### Step 2: Write the Master docker-compose.yml File
We will combine our services into a single Compose file using a shared bridge network.

1. Create and open `docker-compose.yml`:
```yaml
version: '3.8'

networks:
  homelab-net:
    driver: bridge

services:
  # Portainer - Container Management Web UI
  portainer:
    image: portainer/portainer-ce:latest
    container_name: portainer
    restart: always
    security_opt:
      - no-new-privileges:true
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./portainer:/data
    ports:
      - "9000:9000"
    networks:
      - homelab-net

  # Nginx Proxy Manager - Reverse Proxy with SSL
  nginx-proxy:
    image: 'jc21/nginx-proxy-manager:latest'
    container_name: nginx-proxy
    restart: always
    ports:
      - '80:80'
      - '81:81'
      - '443:443'
    volumes:
      - ./nginx-proxy-manager/data:/data
      - ./nginx-proxy-manager/letsencrypt:/etc/letsencrypt
    networks:
      - homelab-net

  # Pi-hole - DNS and Ad Blocker
  pihole:
    image: pihole/pihole:latest
    container_name: pihole
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "8080:80/tcp"
    environment:
      TZ: 'UTC'
      WEBPASSWORD: 'SecureAdminPassword123'
    volumes:
      - ./pihole/config:/etc/pihole
      - ./pihole/dnsmasq:/etc/dnsmasq.d
    restart: always
    networks:
      - homelab-net
```

### Step 3: Deploy the Lab
1. Start all three services in detached background mode:
```bash
docker compose up -d
```
**Expected Output:** Downloads the images and creates the containers.
```text
[+] Running 4/4
 ⠿ Network homelab_homelab-net      Created
 ⠿ Container portainer              Started
 ⠿ Container nginx-proxy            Started
 ⠿ Container pihole                 Started
```

### Step 4: Verify Web Dashboards
1. **Portainer**: Open your web browser and go to `http://[YOUR-SERVER-IP]:9000`. Set up your admin account.
2. **Nginx Proxy Manager**: Go to `http://[YOUR-SERVER-IP]:81`.
   - Default login: `admin@example.com` / `changeme`. Modify these immediately.
3. **Pi-hole**: Go to `http://[YOUR-SERVER-IP]:8080/admin`. Log in using `SecureAdminPassword123`.

---
## Cheat Sheet / Quick Reference

| Action / Command | Scope | Purpose / Example |
|---|---|---|
| `docker compose up -d` | CLI | Start all containers in the background |
| `docker compose down` | CLI | Stop and remove all containers in stack |
| `docker compose ps` | CLI | List status of all active containers |
| `docker compose logs -f`| CLI | Stream runtime logs from all services |
| `docker system df` | CLI | Display local Docker disk usage stats |
| `docker system prune -f`| CLI | Remove all unused container data |

---
## Troubleshooting Table

| Problem | Cause | Fix | Command |
|---|---|---|---|
| Pi-hole fails to start: "port 53 already in use." | The system-resolved service on Ubuntu is already using port 53. | Disable systemd-resolved DNS resolver or edit its configuration to bind to a different local loopback address. | `sudo systemctl stop systemd-resolved` / `sudo systemctl disable systemd-resolved` |
| NPM portal page returns "502 Bad Gateway." | NPM is trying to reach a container that is down or not on the same network. | Verify both containers are connected to the `homelab-net` network using docker inspect. | `docker network inspect homelab-net` |
| Volume modifications result in permission denied. | The local directories created have root ownership, preventing container access. | Change file ownership of directory folders to match user ID permissions. | `sudo chown -R $USER:$USER ~/homelab` |
| Portainer cannot reach Docker socket. | Docker socket file permissions are restricted or Docker service is stopped. | Ensure Docker service is running; verify `/var/run/docker.sock` volume mapping. | `sudo systemctl status docker` |

---
## Interview Questions

> [!question] L1 Question
> **Q:** Why did we map `/var/run/docker.sock` inside the Portainer container?
> **A:** The `/var/run/docker.sock` file is the Unix socket that the Docker daemon listens to for API calls. Mapping this file inside Portainer gives it the ability to communicate directly with the host's Docker daemon, allowing it to start, stop, monitor, and configure other containers on the host.

> [!question] L2 Question
> **Q:** What is the purpose of Nginx Proxy Manager in a home lab, and how does it relate to reverse proxies?
> **A:** Nginx Proxy Manager serves as a **reverse proxy**. Instead of opening multiple high-number ports (like 9000, 8080, 81) on the firewall, NPM listens on standard HTTP (80) and HTTPS (443) ports. It reads the incoming request domain name (e.g. `portainer.local` or `pihole.local`) and routes the traffic internally to the correct container IP and port. It also manages SSL certificates.

> [!question] L3/Scenario Question
> **Q:** You want to run a web app container in your home lab, but you also want it to have its own separate IP address on your home router network, instead of sharing the Docker host's IP. How would you configure this?
> **A:** 
> - **Situation:** Container needs a dedicated IP address on the physical LAN.
> - **Task:** Configure a Docker network driver that bridges directly to the physical interface.
> - **Action:** 
>   1. **Macvlan Network**: I will create a Docker network using the **macvlan** driver. This allows assigning a MAC address to a container, making it appear as a physical device on the network.
>   2. **Create Network**:
>      ```bash
>      docker network create -d macvlan \
>        --subnet=192.168.1.0/24 \
>        --gateway=192.168.1.1 \
>        -o parent=eth0 macvlan-net
>      ```
>   3. **Assign IP**: In my docker-compose, configure the service to use `macvlan-net` and specify a static IP:
>      ```yaml
>      networks:
>        macvlan-net:
>          ipv4_address: 192.168.1.150
>      ```
> - **Result:** The container obtains an IP directly from the physical router network and is directly queryable.

---
## Seedha Simple Mein
*Seedha simple mein: Docker Home Lab ek local environment hai jahan hum containers run karna seekhte hain. Ek single configuration file (`docker-compose.yml`) ka use karke hum Portainer (GUI dashboard), Pi-hole (DNS Manager), aur Nginx Reverse Proxy ko ek sath configure kar sakte hain, jisse physical networking concepts ko clear kiya ja sake.*

---
## Related Notes
- [[05-Automation-and-Ticketing/14-DevOps-Basics/DEV-01 Docker-Fundamentals|DEV-01 Docker-Fundamentals]] — Core Docker concepts.
- [[02-Operating-Systems/04-Linux-RHEL/L-19 Docker Basics for Linux Admins|L-19 Docker Basics for Linux Admins]] — OS package configurations.
- [[06-Career-Growth/13-Lab-Projects/LAB-05 Monitoring-Stack-Lab|LAB-05 Monitoring-Stack-Lab]] — Set up monitoring stack on Docker.

---
*Tags: #desktop-support #lab #containers #docker #L1*
