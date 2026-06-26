---
tags: [desktop-support, azure, cloud, L1]
aliases: [az9-02-azure-global-infrastructure, az9-02]
created: 2026-06-25
status: #complete
difficulty: #beginner
cert-relevant: #none
---

# AZ9-02: Azure Global Infrastructure

> [!abstract] Overview
> This note covers the physical and logical organizational structure of Microsoft Azure. It details Regions, Region Pairs, Availability Zones, logical subscription hierarchies, and tagging standards.

---

---
## Concept Overview
Think of Azure's global infrastructure like a worldwide logistics and retail corporation:
- **Geographies** are the sovereign countries (USA, Germany, India) that have strict laws about where data must reside (Data Residency).
- **Regions** are major warehouse distribution hubs located in specific cities (e.g., East US in Virginia, West Europe in Amsterdam).
- **Availability Zones** are three separate, independent buildings inside the same warehouse hub, each running on its own power generators, water cooling, and network links. If one building floods, the other two keep shipping orders.
- **Resource Groups** are the shipping boxes: you group all items for a single order (a database, web server, and storage disk) in one box so they are easy to label, track, and bill.


---

---
## Technical Deep Dive
### 1. Physical Infrastructure Hierarchy

#### Geographies (Country Boundaries)
- A defined geographic boundary containing one or more Azure regions.
- **Data Residency Compliance:** Azure guarantees that customer data remains within the geography boundary for legal and data sovereignty reasons (e.g., European data stays in Europe).

#### Regions (City Scope)
- A set of datacenters deployed within a latency-defined perimeter and connected through a dedicated low-latency regional fiber network.
- **Choosing a Region:** Base your choice on:
  - **Latency:** Place resources close to your users.
  - **Service Availability:** Not all Azure services are available in all regions.
  - **Cost:** Pricing for the same VM size varies between regions based on local power and real estate costs.

#### Region Pairs (High Availability Backup)
- Every Azure region is paired with another region in the same geography located at least **300 miles away** (e.g., East US is paired with West US; North Europe is paired with West Europe).
- **Why they exist:** If a massive natural disaster (hurricane, earthquake) takes down an entire region, replication recovery services boot up in the paired region. Azure prioritizes updates to only one region per pair at a time to prevent concurrent outages.

#### Availability Zones (Datacenter Scope)
- Physically separate datacenters *within* the same Azure Region.
- **Redundancy:** Each zone has its own independent power, cooling, and networking.
- **SLA:** Deploying VMs across multiple Availability Zones increases your service SLA to **`99.99%`**.
- *Note:* Not all regions support Availability Zones (minimum 3 zones are present in regions that support them).

```
   [ Azure Geography (e.g., United States) ]
       |                              |
[ Region: East US ]            [ Region: West US ]  <--- Region Pair (300+ Miles)
  - Zone 1 (Datacenter A)
  - Zone 2 (Datacenter B)
  - Zone 3 (Datacenter C)
```

### 2. Azure Logical Hierarchy (Container Levels)
Azure organizes resources hierarchically to manage security (RBAC) and billing:

```
[ Microsoft Entra ID Tenant ]   <-- Global identity boundary
            |
    [ Management Groups ]        <-- Configure governance rules for multiple subscriptions
            |
      [ Subscriptions ]          <-- Billing boundary (creates invoices)
            |
      [ Resource Groups ]        <-- Logical grouping for lifecycle management
            |
        [ Resources ]            <-- The physical VM, Database, or VNet
```

- **Management Groups:** Manage compliance policies and access rights across multiple subscriptions (supports up to 6 levels of nesting).
- **Subscriptions:** The primary billing container. Every resource must belong to a subscription.
- **Resource Groups (RG):** A logical folder containing related resources for an Azure solution.
  - **Golden Rule:** A resource can only belong to **one** Resource Group at a time. An RG can contain resources from different regions.

### 3. Naming Conventions & Tagging Best Practices
- **Naming Conventions:** Standardize resource names using a prefix pattern:
  `[Resource Type]-[Application]-[Environment]-[Region]`
  - *Example:* `vm-salesweb-prod-eastus` (Virtual Machine for Sales Web app in Production in East US).
- **Tagging:** Tags are name-value metadata pairs applied to resources to track billing and management:
  - *Example tags:* `Department : Finance`, `Environment : Production`, `Owner : sysadmin@company.com`.

---

---
## Step-by-Step Lab
> [!info] Lab Setup Needed
> Access to the Azure Portal (`portal.azure.com`) and a free tier subscription.

### Step 1: Explore Regions and Services Availability
1. Open a web browser and go to: `https://azure.microsoft.com/en-us/explore/global-infrastructure/products-by-region/`
2. Select your target region (e.g., **East US**).
3. Search for a specific VM family (e.g., **D-Series v5**).
4. **Verify:** Confirm if the service is active in East US compared to a smaller region (e.g., West India). Note that service catalogs vary by region.

### Step 2: Create a Resource Group in Azure Portal
1. Log into the Azure Portal (`portal.azure.com`).
2. Search for and click **Resource groups**. Click **Create**.
3. In the creation wizard:
   - Subscription: Select your active billing subscription.
   - Resource group name: `rg-lab-infrastructure`.
   - Region: Select **(US) East US**.
4. Click Next: **Tags**. Add tags:
   - Name: `Project` | Value: `Sysadmin_Lab`
   - Name: `Env` | Value: `Development`
5. Click **Review + create**, then click **Create**.

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
*Seedha simple mein: Azure ke 60+ regions hote hain jo physical data centers ko group karte hain. Ek region ke andar 3 Availability Zones hote hain redundant power aur connectivity ke sath. Logical resources ko organize karne ke liye hum resource hierarchy (Management Groups, Subscriptions, Resource Groups) use karte hain.*

---
## Related Notes
- [[04-Cloud-and-Security/08-Azure/AZ9-01 Cloud Computing Fundamentals|AZ9-01 Cloud Computing Fundamentals]] — Base virtualization concepts.
- [[04-Cloud-and-Security/08-Azure/AZ104-01 Azure Identity and Governance|AZ104-01 Azure Identity and Governance]] — Managing subscriptions and resource locks.
