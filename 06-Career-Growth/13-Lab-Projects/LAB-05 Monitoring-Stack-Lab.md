---
tags: [desktop-support, lab, monitoring, prometheus, L2]
aliases: [monitoring-stack-lab, lab-05]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #none
---

# LAB-05: Monitoring Stack Lab

> [!note] Overview
> This lab guide covers deploying a modern monitoring and visualization stack using Docker Compose. It installs Prometheus (metrics database), Node Exporter (system metrics collector), and Grafana (visualization dashboards), linking them to monitor host server performance.

---
## Concept Overview
- **What it is** — A Monitoring Stack Lab is a self-contained infrastructure observability environment that collects metrics (CPU, memory, disk, network) from targets, stores them in a time-series database, and visualizes them on real-time dashboards.
- **Why it matters** — Sysadmins cannot manage what they do not measure. Observability is key to detecting performance bottlenecks, predicting disk capacity runouts, and diagnosing service downtime before users report it. 
- **Real job encounter** — Creating Grafana dashboards to monitor server health, investigating alerts triggered by CPU spikes, and tracking network bandwidth consumption metrics.
- **L1 vs L2 vs L3 responsibility split:**
  - **L1 Resolution**: Monitor Grafana screen dashboards for red/amber alert states, verify that alerts are logged in ticketing systems, and notify L2 on call during outages.
  - **Escalation Trigger**: Escalate if Grafana panels return datasource query errors, or if alerting pipelines (Slack, email alerts) fail to notify.
  - **L2 Resolution**: Add new servers as monitoring targets, install metric collection agents (like Node Exporter or Zabbix Agent), and build custom dashboard views.
  - **L3 Resolution**: Author custom PromQL alert rules, deploy high-availability cluster monitoring configurations, and manage metric retention policies.

*Seedha simple mein: Monitoring Stack Lab hume servers ki "health status" ko track karna seekhati hai. Hum Prometheus ke jariye memory/CPU usage ka data collect karte hain aur use Grafana ke dashboard par graph ke form me dekhte hain. Isse server crash hone se pehle alerts set karna easy ho jata hai.*

---
## Technical Deep Dive

### 1. Prometheus and Grafana Architecture
Understanding metrics pipelines:
- **Metrics Source (Node Exporter)**: A lightweight agent running on the target machine that reads local OS metrics (CPU, RAM, Disks) and exposes them on a web endpoint (`http://localhost:9100/metrics`) in a plain-text key-value format.
- **Time Series Database (Prometheus)**: A database optimized for time-stamped numeric data. It queries ("scrapes") the metrics source at a defined interval (e.g. every 15 seconds) and stores the data.
- **Visualization (Grafana)**: Connects to Prometheus as a data source, queries the metrics using PromQL (Prometheus Query Language), and renders interactive graphs.

### 2. Monitoring Pull vs. Push Models
- **Pull Model (Prometheus)**: The central server reaches out to targets to fetch metrics. It is easy to see target state (if target is offline, pull fails and flags "DOWN").
- **Push Model (Zabbix/Nagios)**: Agents send metrics to the central monitoring server. Better suited for dynamic autoscale instances or strict outbound firewall rules.

---
## Step-by-Step Lab

> [!warning] Pre-requisites
> - A Linux server or local machine running Docker and Docker Compose.
> - Outbound network access to download Docker images.

### Step 1: Create the Project Folders and Configuration
We will set up the workspace and write the Prometheus configuration.

1. Create directory structures:
```bash
mkdir -p ~/monitoring-lab/prometheus
cd ~/monitoring-lab
```
2. Create the Prometheus configuration file `prometheus/prometheus.yml`:
```yaml
global:
  scrape_interval: 15s # How often to poll metrics

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100'] # Docker service name resolves automatically
```

### Step 2: Write the docker-compose.yml File
We will combine Prometheus, Node Exporter, and Grafana in one compose stack.

1. Create `docker-compose.yml` in `~/monitoring-lab`:
```yaml
version: '3.8'

networks:
  monitor-net:
    driver: bridge

services:
  # Prometheus Time-Series Database
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
    ports:
      - "9090:9090"
    networks:
      - monitor-net
    restart: always

  # Node Exporter - System metrics agent
  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($|/)'
    ports:
      - "9100:9100"
    networks:
      - monitor-net
    restart: always

  # Grafana - Data visualization UI
  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    volumes:
      - grafana_data:/var/lib/grafana
    networks:
      - monitor-net
    restart: always

volumes:
  prometheus_data:
  grafana_data:
```

### Step 3: Deploy and Test the Stack
1. Start the containers:
```bash
docker compose up -d
```
2. Verify all containers are running:
```bash
docker compose ps
```
**Expected Output:** Prometheus, node-exporter, and Grafana show `Up` state.

### Step 4: Configure Grafana Dashboard
1. Open a browser and go to `http://[SERVER-IP]:3000`. Log in using default credentials `admin` / `admin`. (Change password on first login).
2. Go to **Connections** -> **Data Sources** -> **Add data source**. Select **Prometheus**.
3. Set the URL to `http://prometheus:9090` (internal Docker hostname resolution) and click **Save & test**.
**Expected Output:** Green success banner: "Data source is working".
4. To avoid building a dashboard from scratch, go to **Dashboards** -> **New** -> **Import**.
5. Enter ID `1860` (the community standard Node Exporter Full dashboard) and click **Load**.
6. Select your Prometheus data source in the dropdown and click **Import**.
**Expected Result:** A dashboard showing CPU load, RAM usage, disk IO, and network traffic for your host.

---
## Cheat Sheet / Quick Reference

| Command / PromQL | Scope | Purpose / Example |
|---|---|---|
| `docker compose up -d` | CLI | Start the monitoring stack |
| `up{job="node-exporter"}` | PromQL | Checks if target host node-exporter is up (returns 1) or down (0) |
| `node_memory_Active_bytes` | PromQL | Query active RAM usage on target |
| `100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)`| PromQL | Query average CPU utilization over 5 mins |
| `docker compose logs -f grafana`| CLI | Stream logs from Grafana for troubleshooting |

---
## Troubleshooting Table

| Problem | Cause | Fix | Command |
|---|---|---|---|
| Grafana shows "No Data" or blank graphs on panels. | The time window filter on the top right is set incorrectly (e.g., set to future or wrong time). | Adjust the time picker dropdown (top-right) in Grafana to "Last 5 minutes" or "Last 1 hour." | N/A |
| Prometheus console target page shows Node Exporter is "DOWN." | Prometheus configuration URL target is wrong, or service is blocked. | Verify spelling in `prometheus.yml` targets; check docker network connectivity between containers. | `docker network inspect monitor-net` |
| Prometheus metrics fail to persist across restarts. | Volume binding for `/prometheus` data path is missing. | Verify named volume `prometheus_data` is correctly configured in `docker-compose.yml`. | N/A |
| Grafana fails with: "database disk image is malformed." | Grafana SQLite database got corrupted due to improper host shutdown. | Stop Grafana, delete/rename the local `.db` file in grafana_data, and restart container to rebuild. | N/A |

---
## Interview Questions

> [!question] L1 Question
> **Q:** What is Node Exporter, and what port does it default to?
> **A:** **Node Exporter** is a lightweight metrics agent developed by Prometheus for Unix/Linux systems. It queries system APIs and reads kernel stats to collect hardware and OS metrics (like CPU load, memory, disk, and network stats). It exposes these metrics as plain text on default TCP port `9100` under the `/metrics` path.

> [!question] L2 Question
> **Q:** In Prometheus, what is the difference between "Scrape Interval" and "Evaluation Interval"?
> **A:** **Scrape Interval** is how frequently Prometheus queries target exporter endpoints (e.g., Node Exporter) to fetch new metric values and store them in the database. **Evaluation Interval** is how often Prometheus evaluates configured alerting rules (e.g. checking if CPU usage has exceeded 90% for a sustained period) to decide if an alert should be triggered.

> [!question] L3/Scenario Question
> **Q:** You need to monitor disk usage across 50 production virtual machines. You notice Prometheus is using substantial disk space and disk writes are slowing down host performance. How do you optimize this?
> **A:** 
> - **Situation:** High disk write I/O and large TSDB storage footprints on Prometheus host.
> - **Task:** Shrink database storage footprint and reduce disk writes.
> - **Action:** 
>   1. **Increase Scrape Interval**: Change the default scrape interval in `prometheus.yml` from `15s` to `60s`. This cuts write frequencies by 75%.
>   2. **Retention Policy**: Reduce metric retention time from default 15 days to 7 days by passing the flag `--storage.tsdb.retention.time=7d` to Prometheus command args.
>   3. **Scrape Filtering (Metric Relabeling)**: Use `metric_relabel_configs` to drop useless or high-cardinality metrics (like loop mounts or virtual network interfaces) before saving.
>   4. **Target Allocation**: Separate metric storage from host disks by mounting a dedicated SSD volume to `/prometheus`.
> - **Result:** Disk wear is minimized, database storage footprint shrinks, and system performance is restored.

---
## Seedha Simple Mein
*Seedha simple mein: Servers ko monitor karne ke liye Node Exporter OS statistics nikalta hai, Prometheus us metrics ko regular intervals (scrape) par collect karke store karta hai, aur Grafana us raw numbers ko design dashboard me convert kar deta hai taaki server parameters ko dynamic graphs me visual track kiya ja sake.*

---
## Related Notes
- [[05-Automation-and-Ticketing/13-Monitoring/MON-03 Prometheus-and-Grafana|MON-03 Prometheus-and-Grafana]] — Advanced metrics configurations.
- [[06-Career-Growth/13-Lab-Projects/LAB-04 Docker-Home-Lab|LAB-04 Docker-Home-Lab]] — Container deployment foundation.
- [[05-Automation-and-Ticketing/13-Monitoring/MON-01 Nagios-Setup|MON-01 Nagios-Setup]] — Standard status check systems.

---
*Tags: #desktop-support #lab #monitoring #prometheus #L2*
