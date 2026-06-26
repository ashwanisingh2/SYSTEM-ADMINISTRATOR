---
tags: [desktop-support, automation, monitoring, prometheus-grafana, L2]
aliases: [prometheus-grafana, modern-monitoring-stack]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #none
---

# MON-03: Prometheus and Grafana (Modern Monitoring Stack)

> [!note] Overview
> This note covers the modern observability stack utilizing Prometheus and Grafana. It details the HTTP-based pull monitoring model, Time-Series Databases (TSDB), PromQL query logic, node exporter agents, and Grafana dashboard visualization integrations.

---
## Concept Overview
- **What it is** — Prometheus is a serverless, pull-based time-series database (TSDB) and alerting engine designed for monitoring cloud-native workloads, microservices, and containers. Grafana is an open-source analytics and visualization web console that connects to Prometheus to build interactive dashboards.
- **Why it matters** — Traditional monitoring engines struggle in modern environments where container workloads boot up and destroy in seconds. Prometheus uses a pull model to query lightweight HTTP metrics endpoints (exporters) in real time, and Grafana converts this raw data into graphs.
- **Real job encounter** — Setting up Node Exporter on Linux hosts to capture system stats, importing pre-built system dashboards in Grafana, writing alerting rules, and querying performance metrics using PromQL.
- **L1 vs L2 vs L3 responsibility split:**
  - **L1 Resolution**: Monitor Grafana dashboards for visual red warning alerts, view alerts inside Alertmanager, check target server ping connectivity, and report system availability.
  - **Escalation Trigger**: Escalate if Grafana panels display connection errors ("Data Source Unreachable"), or if Alertmanager fails to route notifications.
  - **L2 Resolution**: Install and configure metric exporters (Node Exporter / Windows Exporter), update scraping targets in the Prometheus configuration file, configure Grafana data sources, and import dashboards.
  - **L3 Resolution**: Architect clustered high-availability Prometheus deployments, configure Alertmanager routing and deduplication matrices, write advanced PromQL queries, and secure dashboard access using directory federation.

---
## Technical Deep Dive

### 1. Prometheus Pull Model vs. Traditional Push Models
- **Push Model (Nagios, Zabbix)**: Agents on target hosts must actively push their data packages to a central server port. This can saturate server ports during high loads and requires storing passwords on the agent.
- **Pull Model (Prometheus)**: The Prometheus server queries HTTP endpoints (scrapes) on target hosts (e.g., `http://target-ip:9100/metrics`) at regular intervals (e.g. every 15 seconds). The target host only needs to run a simple, lightweight web exporter script exposing metrics in a standard plaintext format.

### 2. Time-Series Database (TSDB)
Prometheus stores data as a sequence of values associated with timestamps. A metric is structured as:
```text
metric_name{label_key="label_value", label2="value2"} value timestamp
```
Example:
```text
node_cpu_seconds_total{cpu="0", mode="idle"} 358204.12 1719320400
```
Labels allow for powerful multidimensional queries, letting administrators filter metrics dynamically.

### 3. Exporters
Prometheus does not monitor systems directly; it relies on **Exporters** to translate OS/app metrics into plaintext formats:
- **Node Exporter**: Collects Linux OS statistics (CPU, memory, disk, network).
- **Windows Exporter**: Collects Windows OS statistics via WMI queries.
- **cAdvisor**: Collects container statistics (CPU/RAM resource allocations).

### 4. PromQL (Prometheus Query Language)
A query language designed for evaluating time-series data:
- `node_memory_Active_bytes / node_memory_MemTotal_bytes * 100`: Calculates current RAM utilization percentage.
- `rate(node_network_receive_bytes_total[5m])`: Calculates the average network transfer rate over the last 5 minutes.

---
## Step-by-Step Lab

> [!warning] Pre-requisites
> - A Rocky Linux 9 server (`SVR-MON02` with IP `192.168.10.60`) to host the monitoring stack.
> - A target Linux VM (`SVR-WEB01` with IP `192.168.10.30`) to monitor.
> - Root or sudo access on both machines.

### Step 1: Install and Run Node Exporter on Target Host (SVR-WEB01)
1. Log in to `SVR-WEB01`, download, and extract Node Exporter:
```bash
cd /tmp
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz
tar -xzf node_exporter-1.6.1.linux-amd64.tar.gz
sudo mv node_exporter-1.6.1.linux-amd64/node_exporter /usr/local/bin/
```
2. Create a systemd service unit:
```bash
sudo vi /etc/systemd/system/node_exporter.service
```
Insert the following configuration:
```text
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=nobody
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```
3. Reload systemd and start service:
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now node_exporter
```
**Expected Output:** Service is active. Query metrics locally: `curl http://localhost:9100/metrics`.

### Step 2: Install Prometheus on the Monitor Host (SVR-MON02)
1. Log in to `SVR-MON02` and download Prometheus:
```bash
cd /tmp
wget https://github.com/prometheus/prometheus/releases/download/v2.45.0/prometheus-2.45.0.linux-amd64.tar.gz
tar -xzf prometheus-2.45.0.linux-amd64.tar.gz
sudo mv prometheus-2.45.0.linux-amd64/prometheus /usr/local/bin/
sudo mv prometheus-2.45.0.linux-amd64/promtool /usr/local/bin/
```
2. Create directories and copy default configuration:
```bash
sudo mkdir -p /etc/prometheus /var/lib/prometheus
sudo cp prometheus-2.45.0.linux-amd64/prometheus.yml /etc/prometheus/
```

### Step 3: Configure Scraping Targets in Prometheus
1. Edit the main configuration file:
```bash
sudo vi /etc/prometheus/prometheus.yml
```
2. Add your target client node `SVR-WEB01` to the scrape configuration:
```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'rocky-servers'
    static_configs:
      - targets: ['192.168.10.30:9100']
```
3. Create systemd file `/etc/systemd/system/prometheus.service`:
```text
[Unit]
Description=Prometheus
After=network.target

[Service]
User=root
ExecStart=/usr/local/bin/prometheus --config.file=/etc/prometheus/prometheus.yml --storage.tsdb.path=/var/lib/prometheus/

[Install]
WantedBy=multi-user.target
```
4. Start Prometheus service:
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now prometheus
```

### Step 4: Install and Connect Grafana
1. Add the official Grafana repository:
```bash
sudo vi /etc/yum.repos.d/grafana.repo
```
Insert:
```text
[grafana]
name=grafana
baseurl=https://packages.grafana.com/oss/rpm
repo_gpgcheck=1
enabled=1
gpgcheck=1
gpgkey=https://packages.grafana.com/gpg.key
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
```
2. Install and start Grafana Server:
```bash
sudo dnf install -y grafana
sudo systemctl enable --now grafana-server
```
3. Open a browser and navigate to: `http://192.168.10.60:3000`.
4. Log in using default credentials:
   - Username: **`admin`**
   - Password: **`admin`** (Change on first login).
5. **Configure DataSource**: Go to Connections -> Data Sources -> Add data source -> Select **Prometheus**. Set URL to `http://localhost:9090`. Click **Save & test**.
6. **Import Dashboard**: Click `+` -> Import -> Enter ID **`1860`** (Node Exporter Full) and click **Load**. Select your Prometheus data source and click **Import**.

---
## Cheat Sheet / Quick Reference

| Term / Port | Purpose | Command / Target |
|---|---|---|
| **Port TCP 9090** | Prometheus Server Port | Default access URL for Prometheus console |
| **Port TCP 9100** | Node Exporter Port | Default port exposed by client system agents |
| **Port TCP 3000** | Grafana Web Port | Default URL for user dashboard interface |
| `prometheus.yml` | Main configuration file | `/etc/prometheus/prometheus.yml` |
| `promtool check config` | Validates Prometheus configuration syntax | `promtool check config /etc/prometheus/prometheus.yml` |
| **Rate query (PromQL)**| Calculates per-second rate of increase | `rate(node_cpu_seconds_total[5m])` |
| **Sum query (PromQL)** | Aggregates metric values across labels | `sum(rate(node_cpu_seconds_total[5m]))` |

---
## Troubleshooting Table

| Problem | Cause | Fix |
|---|---|---|
| Prometheus Target Status shows `DOWN` with error: "Connection refused." | Node Exporter service is stopped on the target server, or firewalls block TCP port 9100. | Check service on target: `systemctl status node_exporter`. Open inbound port TCP 9100 on target firewall configurations. |
| Grafana dashboard panel displays: "No data." | The datasource name is wrong, or the target host metrics are not matching dashboard queries. | Verify Prometheus connection status in Grafana Settings. Ensure target job name matches the variables expected by the imported dashboard. |
| Prometheus service fails to start: "Failed to lock storage." | Another instance of Prometheus is already running and locking the TSDB database directory. | Find running process: `ps aux \| grep prometheus`. Kill orphaned processes and restart service. |
| Host logs return: "Too many open files" during Prometheus scraping. | The Prometheus server has run out of file handles due to high target counts. | Increase host server limits in `/etc/security/limits.conf` and adjust `fs.file-max` in sysctl. |
| Grafana Web login returns: "Access denied / locked." | Administrator password has been forgotten. | Reset admin password via CLI: `grafana-cli admin reset-admin-password newpassword`. |

---
## Interview Questions

> [!question] L1 Question
> **Q:** What are the default network ports used by Prometheus, Node Exporter, and Grafana?
> **A:** Prometheus server uses **TCP Port 9090** for its web console and API. Node Exporter runs on **TCP Port 9100** to expose system metrics. Grafana dashboard interface uses **TCP Port 3000** for client browser access.

> [!question] L2 Question
> **Q:** Explain how the Prometheus Pull Model works. How does it locate targets to monitor?
> **A:** Prometheus uses a pull model where the server actively requests metrics from target hosts over HTTP. It locates targets through static configurations defined in the `prometheus.yml` file (where hosts and ports are explicitly listed) or through **Service Discovery** (automatically querying cloud APIs like Azure, AWS, or Kubernetes to dynamically detect new instances).

> [!question] L3/Scenario Question
> **Q:** You need to monitor a group of microservices where containers are created and destroyed dynamically every few minutes. Why is Prometheus better suited for this compared to Nagios, and how would you implement target discovery?
> **A:** 
> - **Situation:** Dynamic container workloads require monitoring without manual endpoint updates.
> - **Task:** Design a self-updating target discovery pipeline.
> - **Action:** 
>   - **Comparison**: Nagios requires static file configurations for each host, causing alerts when containers are destroyed. Prometheus is designed for dynamic environments because it uses **Service Discovery (SD)**.
>   - **Implementation**: I will configure Prometheus with **Kubernetes Service Discovery (kubernetes_sd_configs)** or **DNS Service Discovery (dns_sd_configs)**. In the `prometheus.yml` scrape configuration, I will define discovery queries that watch the container orchestrator's API.
> - **Result:** As new container nodes boot up, they expose metrics at `/metrics`. Prometheus automatically detects them via API changes, begins scraping immediately, and drops them from the target list when they terminate, avoiding false alarms.

---
## Seedha Simple Mein
*Seedha simple mein: Prometheus aur Grafana ek modern observability stack hain. Prometheus ek database (TSDB) hai jo servers se data fetch (pull) karta hai, aur Grafana us metrics ko dashboard charts me show karta hai. Rocky server par node exporter setup karke hum metrics ko port 9100 par release karte hain aur use Grafana panel (port 3000) me visualize karte hain.*

---
## Related Notes
- [[05-Automation-and-Ticketing/13-Monitoring/MON-02 Zabbix-Complete-Guide|MON-02 Zabbix-Complete-Guide]] — Enterprise database-backed monitoring comparisons.
- [[02-Operating-Systems/04-Linux-RHEL/L-18 Linux Performance Monitoring|L-18 Linux Performance Monitoring]] — Performance check metrics.
- [[02-Operating-Systems/04-Linux-RHEL/L-08 Services and Systemd|L-08 Services and Systemd]] — Systemd unit configurations.

---
*Tags: #desktop-support #automation #monitoring #prometheus-grafana #L2*
