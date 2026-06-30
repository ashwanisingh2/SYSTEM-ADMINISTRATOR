---
tags: [desktop-support, m365, onedrive, file-storage, L1]
aliases: [onedrive-administration, onedrive-administration]
created: 2026-06-25
status: #complete
difficulty: #beginner
cert-relevant: #none
---

> [!NOTE|color-blue]
> ☁️ **CLOUD & MICROSOFT 365**

`#complete` `#beginner` `#none`

# M365-10: OneDrive Administration

> [!abstract] Overview
> Yeh note OneDrive for Business ko manage karne aur troubleshoot karne pe focus karta hai. Client sync issues, files on-demand, aur Known Folder Move (KFM) sab cover hote hain.

---
## 🧠 Concept Overview

- **What it is** — Cloud-based personal file storage and sync service for M365 users.
- **Why it matters** — Protects user files from local hard drive crashes and enables collaboration.
- **Where you see this** — Red "X" sync errors, storage limits, accidental deletions.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic task — Resolve local sync conflicts, clear cache, restore deleted files. |
| **L2** | Configure, fix, escalate — Configure KFM, audit sharing links, adjust storage limits. |
| **L3** | Architecture, design — Set global tenant sharing policies, configure data retention. |

> [!tip] Seedha Simple Mein
> *Seedha simple mein: Local PC ke files ko cloud par backup aur sync karne ka tool. Agar laptop kharab ho jaye, toh naye pc pe OneDrive login karte hi sab wapas aa jata hai.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **OneDrive Files On-Demand** is like **a library catalog** because...
>
> - The cloud outline icon is just the catalog index card (file name is there, but book isn't on shelf).
> - When you double-click it, you request the book, and it's downloaded (green check).

---
## 🔬 Technical Deep Dive

### 1. Files On-Demand Icons

> [!info] Key Concept
> OneDrive shows icons to indicate if a file takes up local disk space or is streaming.

- **Cloud**: Cloud-only. Takes 0 bytes locally.
- **Green Outline**: Locally available. Can be offloaded automatically.
- **Solid Green**: Always keep on this device. Pinned for offline access.
- **Red X**: Sync error (usually illegal characters).

### 2. Known Folder Move (KFM)
Automatically redirects Desktop, Documents, and Pictures to OneDrive. Essential for transparent backups.

### 3. OneDrive Lifecycle
When a user is deleted, their manager gets access. Retention is 30 days default. Then goes to recycle bin for 93 days.

> [!danger] Common Mistake
> Running a manual deletion script on the OneDrive appdata folder instead of using `/reset`. This corrupts the local SQL database and duplicates files!

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Windows client with OneDrive

### Step 1: Reset the OneDrive Sync Client

```cmd
# Kill and reset the OneDrive client cache
%localappdata%\Microsoft\OneDrive\onedrive.exe /reset

# Restart OneDrive
start %localappdata%\Microsoft\OneDrive\onedrive.exe
```

> [!success] Expected Output
> ```
> Taskbar cloud icon disappears, then restarts cleanly and re-scans files.
> ```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `onedrive.exe /reset` | Resets the local sync database cache | `%localappdata%\Microsoft\OneDrive\onedrive.exe /reset` |
| `Set-SPOUser -StorageQuota` | Increases OneDrive storage | `Set-SPOUser -Site ... -StorageQuota 5242880` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| Red X "File name contains characters" | File path has illegal characters (`< > ? *`) | Rename file in File Explorer to remove characters |
| Ransomware encrypts local synced files | Virus synced to cloud | Use "Restore your OneDrive" feature on web to rollback by 24 hrs |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Sync Stuck

> [!example] Ticket
> "OneDrive shows 'Processing changes' for 3 days and isn't uploading my new files."

**L1 Response:** Check for illegal characters or huge files (like PSTs).
**Escalation Trigger:** No illegal files, client just hung.
**L2 Resolution:** Run `onedrive.exe /reset`. Rebuilds index, sync resumes.

---
## 🎤 Interview Questions

> [!question] Q1: How long is a deleted OneDrive file kept?
> **Answer:** 93 days in the recycle bin. Can be restored from the web portal.

==**Exam Tip:** Storing Outlook PST files or active databases in OneDrive blocks syncing and must be avoided.==

---
## 🔗 Related Notes

- [[04-Cloud-and-Security/07-Microsoft-365/M365-04 SharePoint Online Administration|M365-04 SharePoint Online Administration]]
- [[02-Operating-Systems/03-Windows-OS/WIN-07 User Profiles|WIN-07 User Profiles]]
