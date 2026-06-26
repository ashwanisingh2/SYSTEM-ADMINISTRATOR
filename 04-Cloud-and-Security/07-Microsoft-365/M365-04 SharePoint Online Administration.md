---
tags: [desktop-support, m365, collaboration, L1]
aliases: [m365-04-sharepoint-online-administration, m365-04]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #none
---

> [!NOTE|color-blue]
> ☁️ **CLOUD & MICROSOFT 365**

`#complete` `#intermediate` `#none`

# M365-04: SharePoint Online Administration

> [!abstract] Overview
> Yeh note SharePoint Online architecture, site collection management, permission models, aur OneDrive integration par focus karta hai. Secure file collaboration environment banana isme sikhaya gaya hai.

---
## 🧠 Concept Overview

- **What it is** — Microsoft ka cloud-based document management aur intranet platform.
- **Why it matters** — Sari company ka data yahi store hota hai. Galat permission se confidential data leak ho sakta hai.
- **Where you see this** — Intranet portal banana, documents share karna, ya OneDrive sync client issues solve karna.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Users ko site access dena, OneDrive sync issues fix karna aur version history se file recover karna. |
| **L2** | Site Collections banana, permission inheritance manage karna aur external sharing settings adjust karna. |
| **L3** | Hub sites architecture design karna, Sensitivity labels aur data loss prevention policies implement karna. |

> [!tip] Seedha Simple Mein
> *SharePoint Online data store karne aur organize karne ki service hai. Har team ke liye alag site collections hoti hain jinhe common access control aur sync options ke sath manage kiya jata hai.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **SharePoint Online** is like **a digital city** because...
>
> - **Tenant**: Entire city map.
> - **Site Collections**: Individual corporate complexes (HR, Finance).
> - **Sites/Libraries**: Rooms with filing cabinets.
> - **Hub Sites**: Transport rings linking independent buildings under a common theme.

---
## 🔬 Technical Deep Dive

### 1. SharePoint Architecture

> [!info] Key Concept
> Modern SharePoint flat architecture follow karta hai. Nested subsites ki jagah independent Site Collections aur Hub sites use hote hain.

- **Team Site**: M365 group se connected hota hai, internal collaboration ke liye.
- **Communication Site**: Pori company mein news broadcast karne ke liye (No underlying M365 group).

### 2. Permissions & Sharing Architecture

- **Groups**: Owners (Full Control), Members (Edit), Visitors (Read).
- **Inheritance**: Default me child files/folders parent site ki permission inherit karte hain. Isse manually thoda/break kiya ja sakta hai.
- **External Sharing**: Globally ya site-level pe restrict hoti hai (Anyone, Existing Guests, Only Organization).

> [!danger] Common Mistake
> Breaking inheritance at the individual file level. Isse system slow hota hai aur audits mein permission track karna bohot mushkil ho jata hai. Folders ya libraries pe inheritance break karein, files pe nahi.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - SharePoint Admin Center permissions
> - SharePoint Online Management Shell

### Step 1: Create Site Collection and Restrict Sharing

```powershell
# Connect
Install-Module -Name Microsoft.Online.SharePoint.PowerShell -Force
Connect-SPOService -Url "https://yourtenant-admin.sharepoint.com"

# Create Communication Site
New-SPOSite -Url "https://yourtenant.sharepoint.com/sites/CorpNews" -Owner "admin@yourtenant.com" -StorageQuota 1048576 -Title "Corp Announcements" -Template "SITEPAGEPUBLISHING#0"

# Restrict Sharing to existing guests only
Set-SPOSite -Identity "https://yourtenant.sharepoint.com/sites/CorpNews" -SharingCapability ExistingExternalUserSharingOnly
```

> [!success] Expected Output
> ```
> Site collection created successfully and sharing restricted.
> ```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `Get-SPOSite` | Saare site collections list karta hai | `Get-SPOSite -Limit All` |
| `Set-SPOSite -LockState NoAccess` | Kisi site ko temporarily access-blocked kar deta hai | `Set-SPOSite -Identity "site-url" -LockState NoAccess` |
| `Remove-SPOSite -NoTrash` | Site ko permanently delete karta hai | `Remove-SPOSite -Identity "site-url" -NoTrash` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| OneDrive Sync Client shows Red X "Sync blocked" | File name mein invalid characters hain ya path 260 limit cross kar raha hai | Rename file/folder to remove `< > ? / \ | * "` or shorten path. |
| Nobody can access the document library except owners | Site owner accidentally broke permission inheritance | Library Settings -> Permissions for this document library -> "Inherit Permissions" click karein. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Recovering Overwritten Files

> [!example] Ticket
> "I accidentally saved over a massive Excel spreadsheet on the finance SharePoint site and lost all the original formulas."

**L1 Response:** Go to the file in SharePoint web view.
**Escalation Trigger:** Agar file Version History me nahi mil rahi.
**L2 Resolution:** File ke bagal mein 3 dots click karke "Version History" me jana. Wahan pichle 500 versions me se purana time/date wala version select karke "Restore" click karna. (No data loss).

---
## 🎤 Interview Questions

> [!question] Q1: How does modern SharePoint differ from classic SharePoint architecture?
> **Answer:** Classic me ek site collection ke andar nested subsites hote the jisse permissions complex ho jati thi. Modern architecture flat hai (har site apne aap mein site collection hai) aur unhe Hub sites ke through join kiya jata hai jisse restructuring easy rehti hai.

==**Exam Tip:** "Sync" option OneDrive me local path banata hai, jabki "Add shortcut to OneDrive" ek pointer banata hai jo har device par visible hota hai. Microsoft shortcuts prefer karta hai.==

---
## 🔗 Related Notes

- [[04-Cloud-and-Security/07-Microsoft-365/M365-01 Microsoft 365 Administration|M365-01 Microsoft 365 Administration]] — Controls tenant identities.
- [[04-Cloud-and-Security/07-Microsoft-365/M365-03 Microsoft Teams Administration|M365-03 Microsoft Teams Administration]] — Relies on SharePoint to store channel files.
- [[04-Cloud-and-Security/07-Microsoft-365/INT-04 Configuration Profiles|INT-04 Configuration Profiles]] — Deploys auto-configuration policies for OneDrive.
