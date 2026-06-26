---
tags: [desktop-support, azure, containers, aci, L2]
aliases: [azure-container-instances, aci-guide]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #az-104
---

> [!NOTE|color-blue]
> ☁️ **AZURE CLOUD**

`#complete` `#intermediate` `#az-104`

# AZ104-08: Azure Container Instances (ACI)

> [!abstract] Overview
> Yeh note Azure Container Instances (ACI) ke baare mein hai jo ek serverless container hosting solution hai. Isme architecture, Container Groups, private registry (ACR) integration, aur persistent storage (Azure Files) mounting cover kiya gaya hai. Isse fast, VM-less execution milta hai.

---
## 🧠 Concept Overview

- **What it is** — ACI ek serverless compute service hai jahan aap bina Virtual Machine ya orchestrator (like Kubernetes) manage kiye Docker containers run kar sakte hain.
- **Why it matters** — Sirf ek chota script ya test app chalane ke liye puri VM banana expensive aur slow hota hai. ACI seconds mein start hota hai aur per-second billing karta hai.
- **Where you see this** — Short-lived automation scripts chalane mein, test web apps host karne mein, ya background processing (ETL jobs) run karne mein.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Container status dekhna, CPU/RAM charts monitor karna, aur logs check karna (`az container logs`). |
| **L2** | Container instances create/delete karna, environment variables set karna, aur Azure File shares mount karna. |
| **L3** | VNet integration (private IPs) setup karna, Managed Identities configure karna aur ARM/Bicep templates likhna. |

> [!tip] Seedha Simple Mein
> *ACI me aap direct apne Docker code ko cloud me run kar sakte ho, bina OS update aur server ka tension liye. Jitni der container chalega, utne hi paise lagenge.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Azure Container Instances** is like renting a **Fully Equipped Meeting Room** by the minute because...
>
> - You don't buy or maintain the building (No VMs to manage).
> - You bring your own briefcase of tools (Docker Image).
> - You step in, do your work instantly, and leave, paying only for the exact minutes used (Serverless Billing).

---
## 🔬 Technical Deep Dive

### 1. ACI Serverless Architecture

> [!info] Key Concept
> No VM overhead. Microsoft manages the hypervisor. You just define CPU/RAM and Image. Hypervisor-level security isolates each container.

### 2. Container Groups

- Equivalent to a Pod in Kubernetes. A collection of containers on the same physical host.
- They share the same lifecycle, local network (`localhost`), and storage volumes.
- Used for **Sidecar patterns** (e.g. primary web app container + helper logging container).

### 3. Azure Container Registry (ACR) Integration

- Best practice is pulling images from private ACR using Admin Credentials (for testing) or Managed Identity / Service Principal (for production).

### 4. Storage Persistence

- Container storage is ephemeral (data lost on restart).
- Persistent storage is achieved by mounting an **Azure File Share** (SMB) directly as a directory path inside the container.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Active Azure Subscription and Azure CLI.

### Step 1: Deploy a Public-Facing Container Instance

```bash
az group create --name RG-Containers-Lab --location eastus

az container create --resource-group RG-Containers-Lab --name hello-aci --image mcr.microsoft.com/azuredocs/aci-helloworld --dns-name-label aci-demo-ashwani --ports 80 --cpu 1 --memory 1.5
```
*Wait for provisioning to succeed.*

### Step 2: Query the FQDN and Test

```bash
az container show --resource-group RG-Containers-Lab --name hello-aci --query ipAddress.fqdn --output tsv
```
> [!success] Expected Output
> `aci-demo-ashwani.eastus.azurecontainer.io`. Open in browser to see the hello-world page.

### Step 3: Mount a Persistent Azure File Share

1. Create a storage account and file share.
```bash
az storage account create -g RG-Containers-Lab -n storaci2026 -l eastus --sku Standard_LRS
export AZ_KEY=$(az storage account keys list -g RG-Containers-Lab -n storaci2026 --query "[0].value" -o tsv)
az storage share create --name acishare --account-name storaci2026 --account-key $AZ_KEY
```
2. Deploy a container that mounts this share to `/mnt/storage`.
```bash
az container create -g RG-Containers-Lab -n data-writer --image alpine --command-line "sh -c 'echo Data Saved at $(date) > /mnt/storage/log.txt'" --azure-file-volume-share-name acishare --azure-file-volume-account-name storaci2026 --azure-file-volume-account-key $AZ_KEY --azure-file-volume-mount-path /mnt/storage
```
Check the Azure File Share in the portal to see `log.txt`.

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `az container create` | Provisions a new container group | `az container create -g RG -n Web --image nginx` |
| `az container list` | Lists all containers | `az container list -o table` |
| `az container logs` | Fetches stdout logs from a container | `az container logs -g RG -n Web` |
| `az container exec` | Runs interactive shell inside container | `az container exec -g RG -n Web --exec-command "/bin/sh"` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| Container fails with `ImagePullBackOff` | Spelling error or ACR unauthorized | Check image tag and provide `--registry-login-server`, username, and password. |
| Container shows `Terminated` instantly | Process finished or crashed | Check `az container logs`. Ensure Docker image has a long-running foreground process. |
| Cannot mount Azure File Share | Key incorrect or SMB blocked | Verify storage key and ensure firewall/NSG allows port 445 (SMB) traffic. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: ACI App Crashing on Startup

> [!example] Ticket
> "My containerized python script is failing to start up in ACI. It says Terminated."

**L1 Response:** Use portal or CLI to check the container state and run `az container logs`.
**Escalation Trigger:** If logs show out-of-memory or complex code errors.
**L2 Resolution:** Read logs. If it's a memory issue, redeploy with higher memory limit (`--memory 4`). If it's a code issue, inform the developer of the exact traceback found in the stdout.

---
## 🎤 Interview Questions

> [!question] Q1: How do you view the output logs of a running container in ACI?
> **Answer:** Run `az container logs --resource-group <RG> --name <Container>` to view stdout/stderr.

> [!question] Q2: What resources do containers inside a Container Group share?
> **Answer:** They share the same physical host, lifecycle, local network (`localhost`), and storage volume mounts.

==**Exam Tip:** To use a Private IP, ACI must be deployed into an Azure Virtual Network (VNet) using `--vnet` and `--subnet`. This cannot be added after creation.==

---
## 🔗 Related Notes

- [[02-Operating-Systems/04-Linux-RHEL/L-19 Docker Basics for Linux Admins|L-19 Docker Basics for Linux Admins]] — General Docker container fundamentals.
- [[04-Cloud-and-Security/08-Azure/AZ104-02 Azure Storage Administration|AZ104-02 Azure Storage Administration]] — Formatting Azure File Shares for mounts.
- [[04-Cloud-and-Security/08-Azure/AZ104-04 Azure Virtual Networking|AZ104-04 Azure Virtual Networking]] — Integrating containers inside virtual subnets.
