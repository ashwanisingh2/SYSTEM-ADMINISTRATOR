---
tags: [desktop-support, azure, cloud, L2]
aliases: [az104-02-azure-storage-administration, az104-02]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #az-104
---

> [!NOTE|color-blue]
> ☁️ **AZURE CLOUD**

`#complete` `#advanced` `#az-104`

# AZ104-02: Azure Storage Administration

> [!abstract] Overview
> Yeh note Azure Storage accounts ke management par hai. Isme redundancy (RA-GRS), lifecycle management, SAS tokens, aur AzCopy commands shamil hain.

---
## 🧠 Concept Overview

- **What it is** — Advanced administration of Azure Storage (Blob, File, Queue, Table).
- **Why it matters** — Data security, availability, and cost optimization.
- **Where you see this** — Sharing files securely via SAS, moving huge data with AzCopy, or saving money with Lifecycle policies.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic task — Check storage account names, reset basic access. |
| **L2** | Configure, fix, escalate — Generate SAS tokens, run AzCopy commands. |
| **L3** | Architecture, design — Configure Lifecycle Management, Azure File Sync. |

> [!tip] Seedha Simple Mein
> *Seedha simple mein: Cloud mein data securely store karna, share karna aur purane data ko saste storage mein automatically move karna.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Azure Storage** is like **a high-security data safe** because...
>
> - **Access Keys** are the master combination.
> - **SAS Tokens** are temporary visitor keycards that only open one specific drawer for a few hours.

---
## 🔬 Technical Deep Dive

### 1. Storage Account Parameters & Redundancy
- **GRS**: Secondary region is passive.
- **RA-GRS**: Secondary region remains readable constantly for high-availability.

### 2. SAS and Keys
- **SAS**: Token appended to URL for restricted access (time, permissions).
- **Access Keys**: Full administrative control. Never share. Rotate periodically.

### 3. File Sync and Lifecycle
- **Azure File Sync**: Caches cloud files locally on Windows Server.
- **Lifecycle Management**: Auto-moves old blobs to Cool or Archive tiers to save money.

> [!danger] Common Mistake
> Sharing the master Access Key instead of generating a time-limited SAS token for external vendors.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Azure Storage Account
> - AzCopy CLI tool

### Step 1: Transfer Data using AzCopy

```bash
# Copy local folder recursively to Azure Blob Container
azcopy copy "C:\LocalData" "https://saalphadevlab01.blob.core.windows.net/images?[SAS_Token]" --recursive
```

> [!success] Expected Output
> ```
> Number of File Transfers Completed: 100
> ```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `azcopy login` | Logs into Azure AD for AzCopy | `azcopy login` |
| `azcopy copy` | Copies files to/from Azure Storage | `azcopy copy <source> <dest>` |
| `azcopy sync` | Syncs folders (only modified files) | `azcopy sync <source> <dest>` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| SAS Token access denied | Token expired or IPs restricted | Generate a new token with correct timeframe/IPs |
| Storage account unreachable | Storage Firewall blocking | Add client IP or VNet subnet to allowed list in Networking |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Large Data Upload

> [!example] Ticket
> "Need to upload 500GB of log files to Azure Storage securely."

**L1 Response:** Explain uploading via portal will fail/timeout.
**Escalation Trigger:** Requires AzCopy CLI and SAS token.
**L2 Resolution:** Generate Container SAS token with Write/List permissions, use AzCopy to upload.

---
## 🎤 Interview Questions

> [!question] Q1: What is RA-GRS?
> **Answer:** Read-Access Geo-Redundant Storage. It replicates data to a secondary region, and keeps the secondary region readable even if the primary region is up.

==**Exam Tip:** Account SAS grants access across multiple services (Blob, File), while Service SAS restricts to one container/share.==

---
## 🔗 Related Notes

- [[04-Cloud-and-Security/08-Azure/AZ9-04 Azure Storage and Networking|AZ9-04 Azure Storage and Networking]]
- [[02-Operating-Systems/04-Linux-RHEL/L-11 File Systems and Storage in Linux|L-11 File Systems and Storage in Linux]]
