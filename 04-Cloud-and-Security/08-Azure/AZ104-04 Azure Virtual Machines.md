---
tags: [azure, virtual-machines, compute, #intermediate]
aliases: [Azure VMs, AZ-104 VMs, Virtual Machines]
created: 2026-06-26
status: #complete
difficulty: #intermediate
cert-relevant: #az-104
---

> [!NOTE|color-blue]
> ☁️ **AZURE / CLOUD**

`#complete` `#intermediate` `#az-104`

# AZ104-04: Azure Virtual Machines

> [!abstract] Overview
> Azure Virtual Machines (VMs) provide on-demand, scalable computing resources in the cloud. *Yeh note cover karta hai Azure VMs kaise kaam karte hain, inko kaise deploy aur manage karte hain, aur ek L1/L2 support engineer ko inke baare mein kya janna zaroori hai.* You'll learn about VM sizes, availability, storage, networking aspects, and common troubleshooting steps required in an enterprise environment.

---
## 🧠 Concept Overview

- **What it is** — A virtualized computer running in Microsoft Azure data centers. *Cloud mein ek virtual server (Windows ya Linux) jismein aap apni requirement ke hisaab se CPU, RAM, aur Storage choose karte hain.*
- **Why it matters** — Allows you to run custom software, host applications, and manage workloads without buying physical hardware. *Company ko apne data center mein physical server kharidne aur maintain karne ki zaroorat nahi padti, sab kuch cloud se manage hota hai.*
- **Where you see this** — Hosting web apps, Active Directory domain controllers, databases, or legacy applications that require OS-level access. *Jahan bhi aapko server par poora control (RDP/SSH) chahiye hota hai, wahan VMs use hote hain.*

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic task — Check VM status, restart VMs, reset local admin passwords, basic connectivity checks. *VM chal raha hai ya nahi, aur user login kar pa raha hai ya nahi.* |
| **L2** | Configure, fix, escalate — Resize VMs, configure backups, attach/detach data disks, fix RDP/SSH issues, troubleshoot performance. *Disk space badhana ya boot diagnostics check karna.* |
| **L3** | Architecture, design — Design VM architecture, Availability Zones/Sets, disaster recovery planning, large-scale deployments using ARM templates or Terraform. *Poore enterprise ka infrastructure design karna.* |

> [!tip] Seedha Simple Mein
> *Azure VM aapka ek remote computer hai jo Microsoft ke data center mein rakha hai. Aap isme apne hisaab se power (CPU/RAM) aur space (Disk) laga sakte ho aur internet ya VPN ke through (RDP ya SSH) access kar sakte ho.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Renting a fully furnished office space** is like deploying an **Azure Virtual Machine** because...
>
> - Just like you can choose a small office (startup) or a large floor (enterprise), you choose a VM size (A-series, D-series, E-series).
> - You pay rent only for the time you use the office (Pay-as-you-go pricing model).
> - If you need an extra room to store files, you can rent it (attaching a Data Disk).
> - If you stop paying rent, you lose access to the office, but you can keep your furniture in storage (Deallocated VM vs. deleting the VM entirely).

---
## 🔬 Technical Deep Dive

### 1. VM Compute Sizes (Series)

> [!info] Key Concept
> Azure offers different VM sizes tailored for specific workloads. *Har type ka workload alag VM series demand karta hai.*

- **A-Series / B-Series:** Entry-level / Burstable. Great for dev/test environments. *Saste aur basic testing ke liye best hain.*
- **D-Series:** General Purpose. Good balance of CPU and memory. *Web servers aur normal applications ke liye.*
- **E-Series:** Memory Optimized. High RAM to CPU ratio. *In-memory databases (like SAP HANA, SQL) ke liye jahan RAM zyada chahiye.*
- **F-Series:** Compute Optimized. High CPU to RAM ratio. *Batch processing aur web analytics ke liye jahan processing power zyada chahiye.*
- **N-Series:** GPU Enabled. *Video rendering, AI, aur deep learning workloads ke liye.*

### 2. VM Storage (Disks)

Azure VMs use Managed Disks.
- **OS Disk:** Contains the operating system (Windows/Linux). Max 4TB. Drive `C:` on Windows, `/dev/sda` on Linux. *Jismein OS install hota hai.*
- **Temporary Disk:** Ephemeral storage. Data is lost upon reboot or deallocation. Drive `D:` on Windows, `/dev/sdb` on Linux. *Isme kabhi critical data mat rakho, yeh temporary hota hai.*
- **Data Disks:** Attached separately for application data. *Aapka actual user data ya database inme store hona chahiye. Inki performance OS disk se alag rehti hai.*

> [!danger] Common Mistake
> Storing important files or database backups on the **Temporary Disk (Drive D:)**. *Agar VM restart ya stop (deallocate) hua, toh temporary disk ka data permanently delete ho jayega!*

### 3. High Availability Options

- **Availability Set (99.95% SLA):** Protects against hardware failures within a single data center by placing VMs in different Fault Domains (FD) and Update Domains (UD). *Ek hi building mein alag-alag racks par VMs rakhna taaki power fail ho toh doosra rack chalta rahe.*
- **Availability Zone (99.99% SLA):** Protects against entire data center failures. Places VMs in separate data centers within the same Azure region. *Alag-alag buildings mein VMs rakhna (power, cooling, network sab independent).*

### 4. Virtual Machine Scale Sets (VMSS)

- **Auto-scaling:** VMSS allows you to create and manage a group of load-balanced VMs. *Ek saath bohot saare identical VMs banane aur manage karne ka feature.*
- **Elasticity:** The number of VM instances can automatically increase or decrease in response to demand or a defined schedule. *Agar website pe achanak traffic badhta hai toh naye VMs automatically add ho jate hain, aur traffic kam hone par delete ho jate hain.*

### 5. VM Extensions

- **Post-Deployment Configurations:** Small applications that provide post-deployment configuration and automation tasks on Azure VMs. *VM banne ke baad usme automatically monitoring agents ya scripts install karna.*
- **Examples:** Custom Script Extension, Azure Monitor Dependency agent, Microsoft Antimalware.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - An active Azure Subscription
> - Azure CLI installed or Azure Cloud Shell access
> - Resource Group already created (e.g., `RG-Prod`)

### Step 1: Create an Azure Virtual Machine via CLI

```bash
# Yeh command naya Ubuntu VM banata hai
az vm create \
  --resource-group RG-Prod \
  --name WebServerVM1 \
  --image Ubuntu2204 \
  --admin-username azureuser \
  --generate-ssh-keys \
  --size Standard_B2s
```

> [!success] Expected Output
> ```json
> {
>   "fqdns": "",
>   "id": "/subscriptions/.../resourceGroups/RG-Prod/providers/Microsoft.Compute/virtualMachines/WebServerVM1",
>   "location": "eastus",
>   "macAddress": "00-0D-3A-XX-XX-XX",
>   "powerState": "VM running",
>   "privateIpAddress": "10.0.0.4",
>   "publicIpAddress": "20.123.45.67",
>   "resourceGroup": "RG-Prod",
>   "zones": ""
> }
> ```

### Step 2: Open Port 80 for Web Traffic

```bash
# Yeh command NSG mein rule add karta hai HTTP traffic allow karne ke liye
az vm open-port --port 80 --resource-group RG-Prod --name WebServerVM1
```

### Step 3: Connect to the VM (Linux)

```bash
# SSH command se server par securely login karo
ssh azureuser@20.123.45.67
```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `az vm list` | Lists all VMs in the subscription. *Saare VMs ki list dikhata hai.* | `az vm list -o table` |
| `az vm show` | Shows details of a specific VM. *Ek VM ki poori information (IP, size, status) nikalta hai.* | `az vm show -g RG-Prod -n WebServerVM1` |
| `az vm start` | Starts a stopped VM. *VM ko chalu karta hai.* | `az vm start -g RG-Prod -n WebServerVM1` |
| `az vm stop` | Stops and deallocates a VM. *VM ko band karta hai aur billing rokta hai (compute charges).* | `az vm stop -g RG-Prod -n WebServerVM1` |
| `az vm restart` | Restarts the VM. *Reboot karta hai OS level se.* | `az vm restart -g RG-Prod -n WebServerVM1` |
| `az vm deallocate` | Fully releases hardware so compute billing stops. *Hard stop, hardware free kar deta hai.* | `az vm deallocate -g RG-Prod -n WebVM` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| **Cannot RDP/SSH to VM** | Network Security Group (NSG) is blocking port 3389/22. *Firewall block kar raha hai.* | Check Effective Security Rules on VM's NIC and add allow rule. Also verify if VM has a Public IP. |
| **VM is slow / High CPU** | Under-provisioned VM size or high process usage. *Size chota hai ya koi andar ki process resource kha rahi hai.* | Check Azure Monitor metrics, use `top`/Task Manager, or resize the VM to a larger D-series size. |
| **Boot loop / Fails to boot** | Bad OS update, corrupted boot sector, or custom script failure. *OS crash ho gaya hai ya VM start nahi ho paa raha.* | Use **Boot Diagnostics** to view the screenshot/serial log. If needed, attach OS disk to another VM to fix filesystem. |
| **Data lost on Drive D:** | User saved files on the Temporary Storage. *Temporary disk restart hone par wipe ho jati hai.* | Explain that this is by design. Data cannot be recovered unless backed up elsewhere. Always attach a Data Disk for persistent storage. |
| **No Internet Access from VM** | Missing NAT gateway, Public IP, or routing issue. *VM ke andar se bahar internet nahi chal raha.* | Check if VM is behind a standard Load Balancer without outbound rules, or check User Defined Routes (UDR). |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: User Cannot Connect via RDP

> [!example] Ticket
> "I am trying to RDP into the Application server (AppVM-01) but it says connection timed out. It was working fine yesterday."

**L1 Response:** Check if the VM is in a "Running" state in the Azure Portal. Verify the public/private IP address. Check if the user is connected to the corporate VPN (if Private IP is used). *Check karo VM on hai ya nahi.*
**Escalation Trigger:** If VM is running, IP is correct, VPN is connected, but RDP still fails.
**L2 Resolution:** Review the Network Security Group (NSG) attached to the subnet and the NIC. Use the **IP Flow Verify** tool in Azure Network Watcher to test port 3389 connectivity. Reset the RDP configuration using "Reset Password" > "Reset configuration only" blade if NSG is fine.

### 🎫 Scenario 2: VM Compute Cost Spike

> [!example] Ticket
> "Our monthly Azure bill has spiked, and it looks like it's coming from DevVM-05. Why is a stopped VM still charging us?"

**L1 Response:** Look at the VM status. Is it "Stopped" or "Stopped (Deallocated)"? *Check karo ki VM deallocate hua hai ya sirf OS se shutdown kiya gaya hai.*
**Escalation Trigger:** N/A - Usually an L1/L2 educational fix.
**L2 Resolution:** The user probably shut down the VM from within the OS (Start -> Power -> Shut Down in Windows). This leaves the VM in a "Stopped" state, which keeps the hardware reserved, so compute charges continue. You must Stop it from the Azure Portal or CLI so it becomes "Stopped (Deallocated)". Configure an Auto-shutdown schedule for Dev VMs to prevent this issue in the future.

### 🎫 Scenario 3: Disk Space Full Alert

> [!example] Ticket
> "Alert: OS Disk on SQL-VM-Prod is at 99% capacity. The Application might crash if not fixed immediately."

**L1 Response:** Log into the VM if possible to see what is taking up space (IIS logs, Windows temp files). Clear basic temp files to get breathing room. *Disk cleanup chalake dekho agar thodi space milti hai.*
**Escalation Trigger:** If clearing temp files doesn't help and a disk resize is required.
**L2 Resolution:** Stop the VM (Deallocate), go to the Disks blade, select the OS Disk, update the size from (e.g.) 128GB to 256GB. Start the VM. Log into Windows, open Disk Management, and Extend the Volume to utilize the new space.

### 🎫 Scenario 4: Custom Script Extension Failure

> [!example] Ticket
> "The automated VM deployment failed at the Custom Script Extension step. The VM is running but the required software isn't installed."

**L1 Response:** Check the deployment logs in the Resource Group's Activity Log. Verify the VM status in the portal. *Dekho ki script fail kyu hua Azure Portal ke error details mein.*
**Escalation Trigger:** The script failed with a specific exit code and the actual script logic needs to be analyzed.
**L2 Resolution:** Navigate to the VM -> Extensions -> click on the Custom Script Extension to view the detailed status and error message. Log into the VM directly and look at the extension logs located at `C:\WindowsAzure\Logs\Plugins` (Windows) or `/var/log/azure/` (Linux).

---
## 🎤 Interview Questions

> [!question] Q1: What is the difference between a Stopped VM and a Deallocated VM?
> **Answer:** A "Stopped" VM is shut down from within the guest OS; you are still billed for compute because the hardware is reserved on the Azure host. A "Deallocated" VM is stopped via the Azure control plane (Portal/CLI); the hardware is released, and compute billing stops (you only pay for the storage disks).

==**Exam Tip:** Always remember: Deallocated = No compute charges. Stopped = Compute charges apply.==

> [!question] Q2: How can you protect your VMs from a complete data center failure in a single region?
> **Answer:** By deploying VMs across multiple **Availability Zones (AZs)**. Each AZ represents a physically separate data center with independent power, cooling, and networking within the same region. This ensures high availability.

> [!question] Q3: What is the purpose of the Temporary Disk (Drive D: in Windows / /dev/sdb in Linux) on an Azure VM?
> **Answer:** It provides short-term storage for applications and processes, such as page files or swap files. Data stored here is not persistent and is lost during a reboot, resize, or deallocation. Important data should never be kept here.

> [!question] Q4: Your Linux VM is failing to boot, and you cannot SSH into it. How can you troubleshoot this from the Azure side?
> **Answer:** You can use **Boot Diagnostics** to view the console serial log and boot screenshot. If you still cannot fix it, you can use the **Serial Console** to interactively troubleshoot, or detach the OS disk and attach it to a recovery VM to repair the filesystem or configuration files.

> [!question] Q5: Can you change the size of an Azure VM after it is created? Is there any downtime?
> **Answer:** Yes, you can resize a VM. However, doing so requires a reboot, which means there will be downtime. The new size must also be available in the underlying hardware cluster where the VM currently resides.

---
## 🔗 Related Notes

- [[AZ104-05 Azure Virtual Networks (VNet)|Azure Virtual Networks]] — *VMs ko network se kaise connect karte hain.*
- [[AZ104-06 Azure Storage|Azure Storage and Managed Disks]] — *VM ke disks ke baare mein detailed jankari.*
- [[AZ104-10 Azure Load Balancer|Azure Load Balancer]] — *Multiple VMs pe traffic distribute karne ka tarika.*
- [[Windows Server Troubleshooting|Windows Server Troubleshooting]] — *Windows OS level issues resolve karne ke liye.*
- [[Linux Basic Commands|Linux Basic Commands]] — *Linux VMs handle karne ke liye command reference.*
