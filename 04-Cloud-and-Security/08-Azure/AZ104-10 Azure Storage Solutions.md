---
tags: [azure, cloud, storage, az-104, infrastructure]
aliases: [Azure Storage, Storage Accounts, AZ104 Storage]
created: 2026-06-26
status: #complete
difficulty: #intermediate
cert-relevant: #az-104
---

> [!NOTE|color-blue]
> ☁️ **AZURE / CLOUD**

`#complete` `#intermediate` `#az-104`

# AZ104-10: Azure Storage Solutions

> [!abstract] Overview
> Azure Storage is Microsoft's cloud storage solution for modern data storage scenarios. It offers highly available, massively scalable, durable, and secure storage for a variety of data objects in the cloud. Ek support engineer ko yeh aana chahiye kyunki har application ko data store karne ke liye storage lagti hai, aur storage ke bina cloud infra ban hi nahi sakta. Storage management is a core day-to-day activity.

---
## 🧠 Concept Overview

- **What it is** — Azure Storage Accounts are the basic container that gives you access to the Azure Storage services: Blob, File, Queue, and Table storage. It provides a unique namespace for your Azure Storage data that's accessible from anywhere in the world over HTTP or HTTPS.
- **Why it matters** — Real job mein data storage, backup, virtual machine disks, log aggregation, sab kuch Azure Storage pe nirbhar karta hai. *Agar storage down ya slow hui, toh poora infrastructure aur application down ho sakta hai.*
- **Where you see this** — You'll see this when managing corporate file shares (Azure Files), backing up databases (Blobs), hosting static websites (Blobs), queuing application messages (Queues), or attaching managed disks to Azure VMs (Page Blobs).

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic task — Check storage account capacity, verify access keys, check file share access, basic troubleshooting of mapped drives. *Basics check karte hain ki storage full toh nahi ho gayi ya user ko access kyun nahi mil raha.* |
| **L2** | Configure storage replication (LRS/GRS), manage RBAC permissions, create private endpoints, configure lifecycle management rules, generate SAS tokens securely. *Issues ko deep-dive karte hain aur security aur performance optimize karte hain.* |
| **L3** | Architecture, design enterprise-level storage solutions, disaster recovery planning, integration with on-prem networks, designing data lake storage for big data analytics. *Bade decisions lete hain ki poora data structure kaisa hoga aur compliance meet ho rahi hai ya nahi.* |

> [!tip] Seedha Simple Mein
> *Azure storage ek bohot bada, infinite size ka hard drive hai cloud mein jiske alag-alag hisse hain - files (documents) ke liye alag, unstructured data (photos/videos/backups) ke liye alag, aur applications ke communication messages ke liye alag. Aap bas rent dete ho jitna storage aap consume karte ho.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Azure Storage Account** is like an **Enterprise Godown (Warehouse)** because...
>
> - **Blob Storage** is like an open floor area where you can dump large unorganized boxes of any shape and size (unstructured data like images, logs, backups). No shelves, just space.
> - **File Storage** is like the organized filing cabinets where multiple employees can open drawers and read files at the same time, using standard keys (SMB protocol).
> - **Queue Storage** is like the reception desk where customer requests are queued up one by one for the workers to process asynchronously.
> - **Table Storage** is like a simple register book keeping key-value records (NoSQL), not a complex relational database with multiple tables linked together.

---
## 🔬 Technical Deep Dive

### 1. Storage Services Overview

> [!info] Key Concept
> Azure offers four primary storage services under a single Storage Account namespace. Every service has its own endpoint URL.

- **Azure Blob Storage**: Object storage solution. Ideal for serving images or documents directly to a browser, storing data for backup and restore, disaster recovery, and archiving. It scales massively. Endpoints look like `https://<account>.blob.core.windows.net`.
- **Azure Files**: Highly available network file shares that can be accessed by using the standard Server Message Block (SMB) protocol or Network File System (NFS) protocol. You can mount these shares concurrently by cloud or on-premises deployments of Windows, Linux, and macOS. *Jaise humare office mein shared drive (Z: drive) hoti hai, waisa hi cloud version.* Endpoints look like `https://<account>.file.core.windows.net`.
- **Azure Queue Storage**: A service for storing large numbers of messages. You access messages from anywhere in the world via authenticated calls using HTTP or HTTPS. Useful for decoupling application components. Endpoints look like `https://<account>.queue.core.windows.net`.
- **Azure Table Storage**: A service that stores non-relational structured data (also known as structured NoSQL data) in the cloud, providing a key/attribute store with a schemaless design. Endpoints look like `https://<account>.table.core.windows.net`.

### 2. Redundancy (Replication) Options

Data is always replicated to ensure durability, high availability, and protection against hardware failures.
- **Locally Redundant Storage (LRS)**: Replicates data three times within a single physical location in the primary region. Protects against server rack and drive failures. *Sabse sasta, par agar data center doob gaya toh data gaya.*
- **Zone-Redundant Storage (ZRS)**: Replicates data synchronously across three Azure availability zones in the primary region. *Data center doobne se bacha lega, par agar poora region fail hua toh problem hai.*
- **Geo-Redundant Storage (GRS)**: Replicates data synchronously three times in the primary region (like LRS), then replicates data asynchronously to the secondary paired region. *Disaster recovery ke liye best.*
- **Geo-Zone-Redundant Storage (GZRS)**: Combines high availability of ZRS with protection from regional outages of GRS. Synchronously across zones, asynchronously to secondary region.

### 3. Access Tiers for Blob Storage

Azure provides different tiers to optimize costs based on how frequently data is accessed.
- **Hot**: For frequently accessed data. Highest storage costs, lowest access/transaction costs. *Jo data roz lagta hai aur jaldi load hona chahiye.*
- **Cool**: For infrequently accessed data stored for at least 30 days. Lower storage costs, higher access costs. *Purane logs jo shayad mahine mein ek baar chahiye.*
- **Cold**: For data stored for at least 90 days. Even lower storage costs.
- **Archive**: For rarely accessed data stored for at least 180 days. Lowest storage cost, highest access cost, takes hours to retrieve (rehydrate) the data before you can read it. *Data jo saalon tak nahi dekhna par legal/compliance ke liye rakhna zaroori hai.*

> [!danger] Common Mistake
> Putting highly active, frequently read/written data in the Cool or Archive tier. *Paisa bachane ke chakkar mein read/write operations (transactions) pe itna zyada bill aa jayega ki budget cross ho jayega! Hamesha use-case ke hisaab se tier choose karo.*

### 4. Security Features

- **Encryption at Rest**: All data is encrypted automatically using Storage Service Encryption (SSE) with Microsoft-managed keys (or Customer-managed keys).
- **Secure Transfer Required**: Forces all connections to use HTTPS.
- **Firewalls and Virtual Networks**: You can restrict access to the storage account from specific IP addresses, ranges, or Azure Virtual Networks.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - An active Azure Subscription
> - Azure CLI installed or Azure Cloud Shell access
> - A Resource group created (e.g., `rg-storage-lab`)

### Step 1: Create a Storage Account

```bash
# Yeh command naya storage account banati hai LRS redundancy ke saath
az storage account create \
  --name mystoragelab12345678 \
  --resource-group rg-storage-lab \
  --location eastus \
  --sku Standard_LRS \
  --kind StorageV2
```

> [!success] Expected Output
> ```json
> {
>   "accessTier": "Hot",
>   "creationTime": "2026-06-26T10:00:00.000000+00:00",
>   "id": "/subscriptions/.../resourceGroups/rg-storage-lab/providers/Microsoft.Storage/storageAccounts/mystoragelab12345678",
>   "location": "eastus",
>   "name": "mystoragelab12345678",
>   "primaryEndpoints": {
>     "blob": "https://mystoragelab12345678.blob.core.windows.net/",
>     "file": "https://mystoragelab12345678.file.core.windows.net/"
>   },
>   "provisioningState": "Succeeded"
> }
> ```

### Step 2: Retrieve Storage Account Keys

```bash
# Storage account ki keys nikaalne ke liye taaki aage ke commands run kar sakein
ACCOUNT_KEY=$(az storage account keys list \
  --resource-group rg-storage-lab \
  --account-name mystoragelab12345678 \
  --query "[0].value" \
  --output tsv)
```

### Step 3: Create a Blob Container

```bash
# Naya container (folder) banana blob storage ke andar
az storage container create \
  --name my-first-blob-container \
  --account-name mystoragelab12345678 \
  --account-key $ACCOUNT_KEY \
  --public-access off
```

### Step 4: Upload a File to Blob Storage

```bash
# Ek local file create karke usko blob mein upload karte hain
echo "Hello Azure Storage! This is a test file for our lab." > testfile.txt

az storage blob upload \
  --container-name my-first-blob-container \
  --file testfile.txt \
  --name sampleblob.txt \
  --account-name mystoragelab12345678 \
  --account-key $ACCOUNT_KEY
```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `az storage account create` | Creates a new storage account | `az storage account create -n mystorage -g myrg -l eastus --sku Standard_LRS` |
| `az storage account show-connection-string` | Gets connection strings for app integration | `az storage account show-connection-string -g myrg -n mystorage` |
| `az storage container create` | Creates a blob container | `az storage container create -n mycontainer --account-name mystorage` |
| `az storage blob upload` | Uploads a file to a blob | `az storage blob upload -c mycontainer -f file.txt -n blob.txt --account-name mystorage` |
| `az storage share create` | Creates an Azure File share | `az storage share create -n myshare --account-name mystorage` |
| `az storage account keys list` | Retrieves storage access keys | `az storage account keys list -g myrg -n mystorage` |
| `az storage account update` | Updates account properties (like TLS versions) | `az storage account update -n mystorage -g myrg --min-tls-version TLS1_2` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| Cannot access Blob URL publicly from browser | Container access level is set to Private. | Change container access level to Blob or Container (if public access is intended), or generate and use a Shared Access Signature (SAS) token in the URL. |
| Port 445 blocked error when mounting Azure Files | ISP or local corporate firewall is blocking SMB port 445. | Use an Azure Point-to-Site VPN, Azure ExpressRoute, or test from an Azure VM instead of on-prem network. |
| Application gets "Authorization failure (403)" | Access keys rolled over, expired SAS token, or incorrect RBAC role. | Verify connection string, regenerate keys if necessary, generate a new SAS token, or assign `Storage Blob Data Contributor` role if using Entra ID authentication. |
| Storage account name not accepted during creation | Name is already taken globally across all of Azure or has uppercase/special chars. | Use a globally unique name with ONLY lowercase letters and numbers, length between 3 and 24 characters. |
| Unexpectedly high billing for storage account | Storing data in hot tier that should be archive, or too many List/Read transactions on cool tier. | Implement Lifecycle Management rules to move older blobs to Cool or Archive tiers automatically. Optimize app code to reduce unnecessary API calls. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Developer needs access to specific Blob Container

> [!example] Ticket
> "Hi IT Support, I need to upload web assets to the 'app-assets' blob container in the production storage account, but I don't have access. Please provide the connection string."

**L1 Response:** Check user's current permissions. Request approval from manager. *Developer ko poore account ki keys (connection string) Dena bilkul safe nahi hai kyunki usse saare containers ka access mil jayega.*
**Escalation Trigger:** If L1 doesn't have permissions to generate SAS tokens or make RBAC assignments in production.
**L2 Resolution:** Instead of giving the Master Access Key, create a time-bound **Shared Access Signature (SAS)** token with Read, Write, List permissions ONLY for the 'app-assets' container. Better yet, assign the "Storage Blob Data Contributor" Azure AD (Entra ID) role to the developer for that specific container scope so they can authenticate using their Azure AD credentials.

### 🎫 Scenario 2: Legacy App Cannot Connect via SMB to Azure Files

> [!example] Ticket
> "Our legacy CRM application cannot mount the new Azure File Share. We are getting a connection timeout and error code 53."

**L1 Response:** Verify network connectivity. Ping the storage account endpoint. Check if the user is running the command as Administrator. *Check karte hain ki traffic ja bhi raha hai ya network level pe blocked hai.*
**Escalation Trigger:** If ICMP is blocked but Telnet to port 445 fails, meaning it's a port block.
**L2 Resolution:** Identify that the ISP or corporate firewall is blocking outbound port 445 (SMB). Set up an Azure Point-to-Site VPN or ExpressRoute to route traffic securely over a private tunnel. Alternatively, check if the legacy app requires SMB 2.1 and the Azure Storage account requires "Secure transfer required" (which forces SMB 3.0). If they are connecting from outside the Azure region, SMB 3.0 is mandatory.

### 🎫 Scenario 3: Accidental Deletion of Critical Blob File

> [!example] Ticket
> "URGENT: I accidentally deleted a blob containing last month's financial reports from the accounting container. Can we recover it?"

**L1 Response:** Check if Soft Delete was enabled on the storage account properties. *Jaldi se portal mein check karna ki soft delete on hai ya nahi.*
**Escalation Trigger:** If L1 doesn't know how to navigate the portal to undelete or if soft delete wasn't previously enabled.
**L2 Resolution:** If Blob Soft Delete was enabled, go to the specific container, toggle "Show deleted blobs" in the view, select the deleted file, and click "Undelete". If soft delete was NOT enabled, the data is permanently lost unless a separate backup mechanism (like Azure Backup) was in place. *As a post-incident action, enable Soft Delete and point-in-time restore for future protection immediately.*

---
## 🎤 Interview Questions

> [!question] Q1: What is the main difference between Storage Account Access Keys and Shared Access Signatures (SAS)?
> **Answer:** Access Keys provide root-level administrative access to the entire storage account and all its services. A Shared Access Signature (SAS) provides restricted, granular, and delegated access to specific resources (like a single blob, container, or file share) with specific permissions (e.g., read-only, write) for a specific time duration.

> [!question] Q2: How do you enforce that all traffic to your Storage Account is encrypted in transit?
> **Answer:** You enable the "Secure transfer required" property on the storage account settings. This forces all incoming requests to use HTTPS and immediately rejects any unencrypted HTTP requests.

> [!question] Q3: If an Azure region goes down entirely (e.g., East US is offline), what redundancy option ensures your application can still READ storage data from another region?
> **Answer:** Read-Access Geo-Redundant Storage (RA-GRS) or Read-Access Geo-Zone-Redundant Storage (RA-GZRS). While standard GRS replicates data to the secondary region, that secondary data is not readable until Microsoft initiates a failover. RA-GRS makes the secondary region endpoint continuously readable at all times, even without a failover.

> [!question] Q4: How do you automate the process of moving files older than 30 days to the Cool tier to save costs?
> **Answer:** You configure "Lifecycle Management" rules in the storage account. You can create a policy rule that evaluates daily: if a blob hasn't been modified in 30 days, move it to the Cool tier, and if it hasn't been modified in 180 days, move it to the Archive tier.

> [!question] Q5: Can you change a storage account from Locally Redundant Storage (LRS) to Geo-Redundant Storage (GRS) after it has been created?
> **Answer:** Yes, you can change the replication type of an existing storage account from LRS to GRS/RA-GRS through the Azure Portal or CLI without any downtime. However, additional charges will apply for the cross-region replication bandwidth and the higher storage cost of GRS.

==**Exam Tip:** For the AZ-104 exam, heavily memorize the differences between Blob access tiers (Hot, Cool, Archive) and the redundancy types (LRS, ZRS, GRS). Always choose the most cost-effective option that meets the scenario requirements. Know that Archive tier takes up to 15 hours for rehydration!==

---
## 🔗 Related Notes

- [[AZ104-01 Azure Subscriptions and RBAC|AZ104-01 Azure Subscriptions and RBAC]] — For managing Storage Account permissions via Entra ID.
- [[AZ104-09 Azure Virtual Networks|AZ104-09 Azure Virtual Networks]] — For Private Endpoints, Service Endpoints, and Storage Firewalls.
- [[Storage Account Security|Storage Account Security Best Practices]] — Deep dive into SAS tokens, encryption, and threat protection.
