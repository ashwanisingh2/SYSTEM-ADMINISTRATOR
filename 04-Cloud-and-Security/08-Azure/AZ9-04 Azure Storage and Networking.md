---
tags: [sysadmin, azure, az-900, storage, networking]
difficulty: Beginner
lab-required: Yes
read-time: 12 mins
---

# AZ9-04: Azure Storage and Networking

> [!abstract] Overview
> This note covers the Azure storage and networking models. It details storage replication scopes (LRS/ZRS/GRS), blob access tiers, Virtual Network (VNet) design, Network Security Groups (NSG), VNet Peering, and hybrid connectivity (VPN Gateway/ExpressRoute).

---
## Concept
Think of Azure networking and storage like organizing a private secure corporate campus:
- **VNet (Virtual Network)** is the fenced physical property border of the campus.
- **Subnets** are the specific rooms inside the office building (e.g., separating the database servers room from the public reception desk).
- **NSG (Network Security Group)** is the security badge reader on each door: it checks who is entering from which room on what port and blocks unapproved traffic.
- **LRS/ZRS/GRS** are the backup filing systems: you can keep three copies in the same room (LRS), copy them to three different buildings on campus (ZRS), or load them in a shipping container to another city (GRS).

*Seedha simple mein: Azure storage account different storage types (Blob, File share) provide karta hai. VNet private network connectivity provide karta hai jise NSG firewalls regulate karte hain. Hybrid systems ko connect karne ke liye hum VPN Gateway ya ExpressRoute use karte hain.*

---
## Technical Deep Dive

### 1. Azure Storage Account Replication Options
All data stored in an Azure Storage Account is duplicated to prevent loss.

- **LRS (Locally Redundant Storage):** Replicates data **three times** within a single physical facility in a single region. Protects against hardware rack failures, but not datacenter disasters. Low cost.
- **ZRS (Zone-Redundant Storage):** Replicates data across **three Availability Zones** in the primary region. Protects against datacenter outages.
- **GRS (Geo-Redundant Storage):** Replicates data to a paired region located hundreds of miles away (creating six total copies). Protects against total regional disasters.
- **GZRS (Geo-Zone-Redundant Storage):** Combines ZRS efficiency in the local region with GRS replication to the paired region. Maximum safety.

### 2. Blob Access Tiers (Hot, Cool, Archive)
Blob (Binary Large Object) storage is unstructured data storage (files, images, videos).
- **Hot Tier:** Optimized for storing data that is accessed frequently. High storage costs, low transaction/read costs.
- **Cool Tier:** Optimized for storing data that is accessed infrequently (stored for at least 30 days). Lower storage costs, higher read transaction costs.
- **Archive Tier:** Optimized for data that is rarely accessed (stored for at least 180 days). Lowest storage costs, highest read costs. Data is offline and requires "rehydration" (latency of hours) to read.

### 3. Azure Files
Presents fully managed file shares in the cloud accessible via the industry-standard **SMB (Server Message Block)** or NFS protocols. Allows on-premises or cloud servers to mount the share as a standard network drive.

### 4. Azure Virtual Networks (VNets)
A VNet is a logical representation of your own isolated network in the cloud.
- **Address Space:** Defined using CIDR blocks (e.g., `10.0.0.0/16`).
- **Subnets:** Segments the VNet address space into smaller logical networks (e.g., `10.0.1.0/24` for Web, `10.0.2.0/24` for Database).
- **NSG (Network Security Group):** A basic stateful firewall applied to subnets or individual network interfaces. Contains rules matching: Source/Destination IP, Port, and Protocol. Rules use priorities (100-65000); lowest number is evaluated first.

### 5. Inter-Network Connectivity
- **VNet Peering:** Connects two VNets directly through Microsoft's private backbone network, making resources inside behave as if they belong to the same network. Latency is extremely low.
- **VPN Gateway:** Establishes an encrypted Site-to-Site tunnel over the public internet, connecting on-prem corporate LANs to the Azure VNet.
- **ExpressRoute:** Establishes a dedicated, private, high-speed physical fiber connection from your office directly to Azure (bypassing the public internet entirely). Offers maximum speed, reliability, and security but high cost.

---
## Lab — Step by Step
> [!info] Lab Setup Needed
> Access to the Azure Portal.

### Step 1: Create a Storage Account and Blob Container
1. Log into the Azure Portal. Click **Storage accounts** -> **Create**.
2. Project Details:
   - Subscription: Select yours.
   - Resource Group: Choose `rg-lab-infrastructure`.
3. Instance Details:
   - Storage account name: `saalphadevlab01` (Must be globally unique, 3-24 characters, numbers and lowercase letters only).
   - Region: **East US**.
   - Primary service: **Standard**.
   - Redundancy: Select **Locally-redundant storage (LRS)** (saves lab cost).
4. Click **Review + create**, then click **Create**.

### Step 2: Upload a File to Container
1. Once deployed, go to the Storage Account page.
2. In the left panel, click **Containers** (under Data storage). Click **+ Container**.
3. Name: `images`. Anonymous access level: **Private (no anonymous access)**. Click Create.
4. Click on the new `images` container. Click **Upload**.
5. Browse and select a dummy text or image file on your host machine. Click Upload.
6. The file is now hosted securely in Azure Blob Storage.

### Step 3: Configure an NSG Rule
1. Search for and click **Network security groups**.
2. Select the NSG associated with your lab VM (`vm-web-dev-nsg`).
3. Click **Inbound security rules** in the left panel. Click **+ Add**.
4. Configure rule to permit custom port 8080 traffic:
   - Source: **Any** | Source port: **\***
   - Destination: **Any** | Destination port: **8080**
   - Protocol: **TCP** | Action: **Allow**
   - Priority: **150**
   - Name: `Allow_Custom_8080`
5. Click **Add**. Verify that the rule compiles successfully.

---
## Related Notes
- [[04-Cloud-and-Security/08-Azure/AZ9-02 Azure Global Infrastructure|AZ9-02 Azure Global Infrastructure]] — Availability Zone designs.
- [[04-Cloud-and-Security/08-Azure/AZ9-03 Azure Compute Services|AZ9-03 Azure Compute Services]] — Mounting network disks to VMs.
- [[04-Cloud-and-Security/08-Azure/AZ104-02 Azure Storage Administration|AZ104-02 Azure Storage Administration]] — Advanced storage sync and lifecycle policies.
- [[04-Cloud-and-Security/08-Azure/AZ104-04 Azure Virtual Networking|AZ104-04 Azure Virtual Networking]] — Advanced hub-spoke VNet peering structures.

