---
tags: [desktop-support, azure, cloud, L2]
aliases: [az104-03-azure-virtual-machines, az104-03]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #az-104
---

# AZ104-03: Azure Virtual Machines

> [!abstract] Overview
> This note covers advanced Azure Virtual Machine engineering. It details disk SKU categories, proximity placement configurations, custom extensions, backup integration, metrics collection, and deployment automation using ARM templates.

---

---
## Concept Overview
Think of Azure Virtual Machine administration as deploying servers inside a massive global data center:
- **Managed Disks** are the physical server hard drives: you choose between slow magnetic disks for archives (Standard HDD) or high-speed solid-state drives for database engines (Premium/Ultra SSD).
- **Proximity Placement Groups** are like reserving desks in the same room for a team: you tell the manager (Azure scheduler) to place your VMs physically close to each other inside the data center to reduce network latency.
- **Custom Script Extensions** are the automated installation crews: the moment the VM finishes booting, they run your custom scripts to install tools, configure firewalls, and prepare the OS.
- **ARM Templates** are the architectural blueprints: instead of manually clicking buttons in the portal, you write a JSON code file describing the VM, and the platform deploys the exact system automatically on-demand.


---

---
## Technical Deep Dive
### 1. VM Sizing & Managed Disk SKUs
Azure VMs support different managed disk types based on size and performance requirements:

- **Standard HDD:** Magnetic drive media. Low cost. Best for backup storage.
- **Standard SSD:** Entry-level solid-state storage. Balanced performance for test environments.
- **Premium SSD (v1/v2):** High performance, high IOPS, low latency. Standard for production databases and mission-critical apps. Disk size determines performance (e.g., a **P10** disk is 128GB, providing 500 IOPS; a **P30** disk is 1TB, providing 5000 IOPS).
- **Ultra Disk:** Sub-millisecond latency. Allows independent scaling of capacity, IOPS, and throughput without resizing the disk. Cannot be backed up using standard Azure Backup.

### 2. Proximity Placement Groups (PPG)
While Availability Zones separate VMs physically to prevent concurrent outages, some applications (like high-frequency trading or database clusters) require the lowest possible latency between VMs.
- **PPG:** A logical grouping configuration that forces Azure to physically deploy associated VMs within the same hardware datacenter facility, minimizing network latency.

### 3. Virtual Machine Scale Sets (VMSS) & Autoscale
A VMSS deploys groups of identical, load-balanced VMs.
- **Scaling Rules:**
  - **Manual:** Administrator manually sets instance count.
  - **Autoscale (Metric-based):** Scale rules evaluate metrics (e.g., if average CPU usage exceeds $75\%$ for 10 minutes, scale out by adding 2 VMs; if CPU drops below $30\%$, scale in by deleting 1 VM).

### 4. Custom Script Extension
Allows administrators to execute custom PowerShell or Bash scripts inside the guest OS automatically after the VM finishes provisioning.
- **Use Case:** Automating software installations, domain joining, or configuration baselines without needing manual login or SSH access.

### 5. VM Backup: Recovery Services Vault
Azure Backup manages VM backups securely without VM agents.
- **Recovery Services Vault (RSV):** The cloud storage container hosting backup recovery points.
- **Backup Policies:** Define backup frequency (daily, weekly) and retention schedules (keep daily backups for 30 days, monthly for 12 months). Supports instant restore points using disk snapshots.

### 6. ARM (Azure Resource Manager) Templates
Infrastructure-as-Code (IaC) templates written in JSON.
- **Structure:**
  - `parameters` — Values provided during deployment (e.g., VM name, admin password).
  - `variables` — Internal values constructed within the template.
  - `resources` — The actual Azure objects being deployed (VMs, NICs, VNets).
  - `outputs` — Values returned after deployment (e.g., VM Public IP).

---

---
## Step-by-Step Lab
> [!info] Lab Setup Needed
> Access to the Azure Portal and an active subscription.

### Step 1: Create a Recovery Services Vault
1. Log into the Azure Portal. Search for and click **Recovery Services vaults**. Click **+ Create**.
2. Project Details:
   - Resource Group: Select `rg-lab-infrastructure`.
   - Vault name: `rsv-lab-backup`.
   - Region: **East US**.
3. Click **Review + create**, then click **Create**.

### Step 2: Configure VM Backup
1. Go to the newly created vault `rsv-lab-backup`.
2. Under Getting started, click **Backup**.
3. Where is your workload running? **Azure**.
4. What do you want to backup? **Virtual machine**. Click Backup.
5. Backup Policy: Select **Standard** (Daily backup, 30 days retention).
6. Virtual Machines: Click **Add**.
7. Select your lab VM `vm-web-dev`. Click OK.
8. Click **Enable Backup**. The backup engine configures the initial restore point snapshot schedule.

### Step 3: Deploy a Custom Script Extension
To install the NGINX web server on `vm-web-dev` automatically:
1. Navigate to the Virtual Machine page for `vm-web-dev`.
2. Under Settings in the left panel, click **Extensions + applications**. Click **+ Add**.
3. Select **Custom Script Extension**. Click Next.
4. Upload a script file `install_web.sh` (contents: `#!/bin/bash \n apt-get update && apt-get install -y nginx`).
5. Click **Create**.
6. Once execution completes, open a browser and navigate to the VM's public IP address.
7. **Verify:** Confirm that the default NGINX welcome page displays, proving the script ran successfully.

---

---
## Cheat Sheet / Quick Reference
| Command / Configuration | Scope | Purpose / Example |
|---|---|---|
| `systemctl status <service>` | Linux | Check status of system service |
| `ip address show` | Linux | Display local interface network details |
| `Get-Service` | PowerShell | Verify service status on Windows hosts |
| `Test-NetConnection` | PowerShell | Check network path connectivity to target ports |

---
## Troubleshooting
| Problem | Cause | Fix | Command |
|---|---|---|---|
| Service connection timeout | Network firewall or routing blocking traffic | Check network route and enable target ports on firewall | `ping -c 4 <ip>` / `nc -zv <ip> <port>` |
| Access Denied error | User account lacks permissions or invalid credentials | Verify account access permissions or reset password | N/A |
| Resource not found | Object or path is misspelled or deleted | Verify spelling of target path or query active objects | N/A |

---
## Interview Questions
> [!question] L1 Question
> **Q:** How do you verify if the target service is running?
> **A:** On Linux, I would execute `systemctl status <service-name>`. On Windows, I would run `Get-Service <service-name>` in PowerShell or check Services.msc.

> [!question] L2 Question
> **Q:** Explain how you would troubleshoot a network connectivity issue to a remote server.
> **A:** I would verify local IP configuration, test routing gateway using `ping`, trace hops using `traceroute` or `tracert`, and check port accessibility using `telnet` or `Test-NetConnection` on target port.

---
## Seedha Simple Mein
*Seedha simple mein: AZ-104 VMs managed disks (Premium SSD, Standard SSD), high-availability (Availability Sets, Zones), monitoring (diagnostic metrics), and deployment automation (ARM/JSON templates) par run hoti hain.*

---
## Related Notes
- [[04-Cloud-and-Security/08-Azure/AZ9-03 Azure Compute Services|AZ9-03 Azure Compute Services]] — Base compute profiles and fault domain basics.
- [[04-Cloud-and-Security/08-Azure/AZ104-01 Azure Identity and Governance|AZ104-01 Azure Identity and Governance]] — RBAC controls for virtual machine contributor.
- [[04-Cloud-and-Security/08-Azure/AZ104-06 Azure Monitor and Backup|AZ104-06 Azure Monitor and Backup]] — Advanced backup recovery options and metric alerts.
