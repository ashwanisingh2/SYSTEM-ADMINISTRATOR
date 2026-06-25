---
tags: [sysadmin, azure, az-104, storage, azcopy]
difficulty: Advanced
lab-required: Yes
read-time: 15 mins
---

# AZ104-02: Azure Storage Administration

> [!abstract] Overview
> This note covers advanced Azure Storage engineering. It details account options, geo-redundancy profiles (RA-GRS), lifecycle management, mounting Azure Files, File Sync configurations, Shared Access Signatures (SAS), and `AzCopy` CLI data transfer.

---
## Concept
Think of Azure storage administration as managing secure data safes:
- **Storage Access Keys** are the master combination keys to the entire building. If a developer gets a copy of this key, they can open every safe, delete databases, and steal files.
- **Shared Access Signatures (SAS)** are temporary visitor keycards: you configure a card that only opens Safe 3 (Blob container) for exactly 4 hours, and only permits reading files, not deleting them.
- **Azure File Sync** is a continuous delivery truck syncing your local office file servers with the master cloud safe.
- **AzCopy** is a high-speed packing conveyor belt built to shift millions of files in and out of the storage safes at multi-gigabit speeds.

*Seedha simple mein: AZ-104 storage admin security aur performance par focus karta hai. Master keys share karne ki jagah hum SAS tokens use karte hain access limit karne ke liye. local server sync ke liye File Sync use hota hai aur mass data migration ke liye AzCopy tool use hota hai.*

---
## Technical Deep Dive

### 1. Storage Account Parameters & Redundancy Profiles
When creating a storage account:
- **Performance Tiers:** Standard (magnetic/Standard SSD backplanes) vs. Premium (Ultra-low latency solid-state drives).
- **Geo-Redundancy Options:**
  - **GRS (Geo-Redundant Storage):** Secondary region is completely passive. You cannot read data from the secondary region unless Microsoft initiates a failover.
  - **RA-GRS (Read-Access Geo-Redundant Storage):** The secondary replication region remains **readable** constantly. If the primary region has a read outage, applications can failover to read from the secondary location immediately.

### 2. Blob Containers & Lifecycle Management
Blob storage organizes unstructured data. To save costs, configure **Lifecycle Management Policies** using rule conditions:
- *Rule example:* If a block blob has not been modified for 30 days, move it to the **Cool Tier**. If not modified for 90 days, move it to the **Archive Tier**. If not modified for 365 days, delete the blob permanently.

### 3. Azure File Sync Architecture
Azure File Sync caches a copy of your Azure File shares locally on an on-premises Windows Server.
- **Cloud Tiering:** A core feature. The local server only caches recently accessed files. Older, unused files are tiered up to the cloud, leaving "pointer files" on the local disk. This allows a local 500GB server disk to present a 10TB cloud file share.

### 4. Shared Access Signatures (SAS)
A SAS is a URI token appended to a storage URL that grants restricted access rights.
- **Scope Levels:**
  - **Account SAS:** Grants access to resources across multiple services (Blob, File, Queue, Table).
  - **Service SAS:** Grants access to a specific resource container (e.g., a single blob container).
  - **User Delegation SAS:** Secured using Microsoft Entra ID credentials rather than the storage account master keys (recommended for high security).
- **Parameters configured:** Allowed permissions (Read, Write, Delete, List), Start and Expiry times, allowed IP addresses, and allowed protocols (HTTPS only).

### 5. Storage Security: Access Keys and Firewalls
- **Storage Access Keys:** Every account has two master keys (Key1 and Key2). Used for full administrative access.
  - *Best Practice:* Rotate keys periodically. Use Key2 while regenerating Key1 to prevent service downtime.
- **Storage Firewall:** Restricts access to the storage account so it is only reachable from specific Azure Virtual Networks (subnets) or designated public IP blocks.

---
## AzCopy Command Reference
`AzCopy` is a command-line utility optimized for copying data to and from storage accounts.

```bash
# Login to Azure using active credentials
azcopy login

# Copy local folder recursively to Azure Blob Container
azcopy copy "C:\LocalData" "https://saalphadevlab01.blob.core.windows.net/images?[SAS_Token]" --recursive

# Sync local folder with Container (only copies modified files)
azcopy sync "C:\LocalData" "https://saalphadevlab01.blob.core.windows.net/images?[SAS_Token]"

# Copy data between two Azure storage containers directly in the cloud
azcopy copy "https://source.blob.core.windows.net/container1?[SAS]" "https://dest.blob.core.windows.net/container2?[SAS]" --recursive
```

---
## Lab — Step by Step
> [!info] Lab Setup Needed
> An active Azure Storage Account (`saalphadevlab01`), a local file `C:\LabFiles\data.txt` on your workstation, and the AzCopy executable downloaded.

### Step 1: Generate a SAS Token in the Portal
1. Open the Azure Portal. Select your storage account `saalphadevlab01`.
2. In the left panel, click **Shared access signature** (under Security + networking).
3. Allowed services: Check **Blob**.
4. Allowed resource types: Check **Container** and **Object**.
5. Allowed permissions: Check **Read**, **Write**, and **List** (Uncheck Delete).
6. Allowed protocols: Select **HTTPS only**.
7. Click **Generate SAS and connection string**.
8. Copy the **SAS token** string (starts with `?sv=`).

### Step 2: Transfer Data using AzCopy
1. Open Command Prompt on your host machine. Navigate to your AzCopy directory.
2. Run the copy command to upload a file:
   ```cmd
   azcopy copy "C:\LabFiles\data.txt" "https://saalphadevlab01.blob.core.windows.net/images/data.txt[Paste_SAS_Token_Here]"
   ```
3. Verify that the upload completes successfully.

### Step 3: Mount Azure Files on Windows
1. In the Azure Portal, open the storage account -> click **File shares** -> click **+ File share**.
2. Name: `sharedfiles`. Tier: Transaction optimized. Click Create.
3. Click on the new share `sharedfiles` -> click **Connect**.
4. Select **Windows**. Copy the generated PowerShell script block.
5. Open PowerShell on a local server/PC. Paste and run the script.
6. **Verify:** Open File Explorer. Confirm that the drive `Z:` is mounted and points to the Azure Cloud File Share.

---
## Related Notes
- [[04-Cloud-and-Security/08-Azure/AZ9-04 Azure Storage and Networking|AZ9-04 Azure Storage and Networking]] — Base storage account redundancy types.
- [[04-Cloud-and-Security/08-Azure/AZ104-01 Azure Identity and Governance|AZ104-01 Azure Identity and Governance]] — RBAC assignment for storage access.
- [[02-Operating-Systems/04-Linux-RHEL/L-11 File Systems and Storage in Linux|L-11 File Systems and Storage in Linux]] — Mounting network shares (NFS/SMB) in Linux.

