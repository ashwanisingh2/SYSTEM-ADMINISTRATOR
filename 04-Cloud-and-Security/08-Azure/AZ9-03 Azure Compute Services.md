---
tags: [sysadmin, azure, az-900, compute]
difficulty: Beginner
lab-required: Yes
read-time: 12 mins
---

# AZ9-03: Azure Compute Services

> [!abstract] Overview
> This note covers the Azure compute service portfolio. It details Virtual Machine sizing profiles, Availability Sets, Scale Sets, Azure App Services, serverless Functions, Container Instances (ACI), and Kubernetes (AKS).

---
## Concept
Think of Azure compute options like choosing a commercial vehicle fleet for a business:
- **Azure Virtual Machines (IaaS)** is like buying a cargo van. You must purchase the chassis, maintain the engine, change the oil (OS patching), and drive it yourself, but you have complete control over what sits in the back.
- **Azure App Service (PaaS)** is hiring a taxi. You don't care about the taxi's engine or maintenance; you just give the driver your destination (your code/website files) and pay for the ride.
- **Azure Functions (Serverless)** is renting an automated delivery drone for one single package. The drone is asleep; it wakes up when a delivery request arrives (trigger), flies the package, lands, and turns off. You only pay for the exact seconds the drone was flying (consumption model).

*Seedha simple mein: Azure compute services dynamic scaling and hosting solutions provide karti hain. VMs full OS control (IaaS) deti hain. App Services code hosting (PaaS) ke liye hain, aur Azure Functions serverless triggers execute karne ke liye compile kiye jaate hain.*

---
## Technical Deep Dive

### 1. Azure Virtual Machines (VMs)
Azure VMs provide on-demand, virtualized compute resources.
- **VM Size Series:**
  - **B-Series (Burstable):** Low-cost, designed for workloads that run idle but need occasional high CPU bursts (e.g., test environments, small databases).
  - **D-Series (General Purpose):** Balanced CPU-to-memory ratios. Best for web servers and standard business apps.
  - **E-Series (Memory Optimized):** High memory-to-CPU ratios. Best for databases, cache servers (Redis), and in-memory analytics.
  - **F-Series (Compute Optimized):** High CPU-to-memory ratios. Best for batch processing, web servers, and scientific modeling.
- **VM Sizing Naming Syntax:** E.g., `Standard_D4s_v5`.
  - `D` = Series family.
  - `4` = Number of virtual CPU cores.
  - `s` = Supports premium SSD storage.
  - `v5` = Hardware version iteration.

### 2. High Availability: Fault vs. Update Domains
To protect VMs from hardware failures, deploy them inside an **Availability Set**:
- **Fault Domain (FD):** Represents physical racks sharing a common power supply and network switch. VMs in an Availability Set are separated across different FDs (usually 2 or 3) so a power failure on one rack does not take down all VMs.
- **Update Domain (UD):** Represents logical groups of hardware that can be rebooted during Microsoft system updates. Only one UD is updated at a time, keeping your application online.

```
       [ Availability Set ]
   +-----------+-----------+
   |           |           |
[ Rack 1 ]  [ Rack 2 ]  [ Rack 3 ]   <--- Fault Domains (FD 0, FD 1, FD 2)
  - VM01      - VM02      - VM03
  (UD 0)      (UD 1)      (UD 2)     <--- Update Domains (UD 0, UD 1, UD 2)
```

### 3. VM Scale Sets (VMSS)
VMSS allows you to deploy and manage a load-balanced group of identical VMs.
- **Auto-Scaling:** Automatically increases or decreases the number of VM instances based on CPU utilization, memory usage, or scheduled times (horizontal scaling).

### 4. Azure App Service (PaaS Web Hosting)
A fully managed hosting platform for web applications, APIs, and mobile backends.
- **Support:** Native support for .NET, Java, Node.js, Python, PHP, and Docker containers.
- **Tiers:** Free/Shared (testing), Basic (low traffic), Standard (production, auto-scale, SSL), and Premium (high performance).

### 5. Serverless Compute: Azure Functions
Executes code on-demand in response to events (triggers) without managing virtual servers.
- **Consumption Plan:** Automatically scales down to zero when inactive. You pay only for the execution time (measured in gigabyte-seconds) and the number of runs.

### 6. Container Services: ACI and AKS
- **Azure Container Instances (ACI):** The fastest and simplest way to run a single isolated Docker container in Azure without managing VMs or orchestration layers.
- **Azure Kubernetes Service (AKS):** A fully managed Kubernetes service used to orchestrate, scale, and manage complex, multi-container microservice deployments.

---
## Compute Services Decision Table

| Business Requirement | Recommended Azure Service | Service Model |
|---|---|---|
| Lift-and-shift legacy app requiring direct OS registry edits | **Azure Virtual Machines** | IaaS |
| Host a public Python web application, automate patching and scale | **Azure App Services** | PaaS |
| Execute a image-resizing script whenever a user uploads a photo | **Azure Functions** | Serverless |
| Launch a single test container script for 15 minutes | **Azure Container Instances** | PaaS |
| Deploy a microservice application with 50 load-balanced containers | **Azure Kubernetes Service** | PaaS |

---
## Lab — Step by Step
> [!info] Lab Setup Needed
> Access to the Azure Portal and an active subscription.

### Step 1: Create a Linux Virtual Machine
1. Log into the Azure Portal. Click **Create a resource** -> **Virtual machine**.
2. Project Details:
   - Subscription: Select yours.
   - Resource Group: Choose `rg-lab-infrastructure` (created in AZ9-02).
3. Instance Details:
   - VM Name: `vm-web-dev`
   - Region: **East US**
   - Availability options: **No infrastructure redundancy required** (standard testing).
   - Image: **Ubuntu Server 22.04 LTS - x64 Gen2**.
   - Size: Select **Standard_B1s** (1 vCPU, 1 GiB memory - lowest cost).
4. Administrator Account:
   - Authentication type: **SSH public key**.
   - Username: `azureuser`.
   - Key source: **Generate new key pair**.
5. Inbound Port Rules:
   - Public inbound ports: Select **Allow selected ports**.
   - Select ports: **SSH (22)** and **HTTP (80)**.
6. Click **Review + create**, then click **Create**.
7. A prompt will ask to download the private key. Click **Download private key and create resource**. Save the `.pem` key file to your host machine.

### Step 2: Verify Compute Deployment
1. Once deployment completes, click **Go to resource**.
2. Locate the **Public IP address** of the VM.
3. Use a terminal on your host machine to verify SSH access:
   ```bash
   ssh -i ~/Downloads/vm-web-dev_key.pem azureuser@[Your_VM_Public_IP]
   ```
4. Confirm you can connect successfully to the Ubuntu shell.

---
## Related Notes
- [[04-Cloud-and-Security/08-Azure/AZ9-01 Cloud Computing Fundamentals|AZ9-01 Cloud Computing Fundamentals]] — Service model comparisons (IaaS vs PaaS).
- [[04-Cloud-and-Security/08-Azure/AZ9-04 Azure Storage and Networking|AZ9-04 Azure Storage and Networking]] — Virtual networks for VM interfaces.
- [[04-Cloud-and-Security/08-Azure/AZ104-03 Azure Virtual Machines|AZ104-03 Azure Virtual Machines]] — Advanced sizing and ARM template deployments.

