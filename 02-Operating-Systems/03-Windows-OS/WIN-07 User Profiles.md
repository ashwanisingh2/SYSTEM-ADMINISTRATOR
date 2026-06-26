---
tags: [desktop-support, windows-os, user-profiles, L1]
aliases: [profile-corruption, profile-management]
created: 2026-06-25
status: #complete
difficulty: #beginner
cert-relevant: #md-102
---

> [!NOTE|color-blue]
> 🏢 **WINDOWS**

`#complete` `#beginner` `#md-102`

# WIN-07: User Profiles

> [!abstract] Overview
> Yeh note User Profiles ko cover karta hai, jo Windows mein user-specific settings aur files store karte hain. Ek support engineer ko iska aana bahot zaroori hai taaki temporary profile errors fix kar sake, corrupt profiles repair kar sake aur user data restore kar sake.

---
## 🧠 Concept Overview

- **What it is** — A Windows User Profile is a collection of user-specific settings, configurations, and files (stored in `C:\Users\<username>`) that define the user's desktop environment, application data, and registry hive (`NTUSER.DAT`).
- **Why it matters** — Profile corruption and configuration mismatches cause login delays, application errors, and data loss. A support engineer must know how to repair corrupt profiles, manage profile types, and configure folder redirection.
- **Where you see this** — Fixing "Logged in with a temporary profile" errors, migrating user data to new machines, resetting corrupt application settings, and managing shared hot-desk terminals.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Troubleshoots basic login errors, fixes temporary profile loops, and backs up user desktop/documents data. |
| **L2** | Performs profile migrations (using USMT or PCMover), cleans up registry profile lists, and configures local profile deletion policies. |
| **L3** | Designs roaming profile storage structures, configures FSLogix profiles for virtual desktop environments (VDI), and defines folder redirection group policies. |

> [!tip] Seedha Simple Mein
> *User Profile aapka digital kamra hai. Jaise aap apne kamre ko sajate ho (desktop wallpaper, shortcuts), waisa hi data profile mein save hota hai. Agar profile corrupt ho jaye toh Windows aapko ek 'Temporary' room de deta hai.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **User Profiles** are like **hotel rooms** because...
> 
> - **Local Profile**: A permanent room assigned to you.
> - **Roaming Profile**: Your luggage moves with you whichever room you check into.
> - **Temporary Profile**: A spare room given when your original room key is broken. All changes are cleared when you checkout.

---
## 🔬 Technical Deep Dive

### 1. Types of User Profiles

> [!info] Key Concept
> Windows supports several profile structures to accommodate different work environments: Local, Roaming, Mandatory, and Temporary.

- **Local User Profile**: Stored on the local storage drive of a specific computer. Settings are unique to that machine.
- **Roaming User Profile**: Stored on a centralized network share. The profile downloads to the local machine during login and syncs changes back to the share during logout.
- **Mandatory User Profile**: A read-only profile template (`NTUSER.MAN`). Changes made by the user are discarded upon logout. Used for public kiosks or student labs.
- **Temporary User Profile (`TEMP`)**: Loaded by Windows when the system fails to load the user's registry hive (`NTUSER.DAT`). All changes are deleted at logout.

> [!danger] Common Mistake
> Saving files on the Desktop when logged into a temporary profile, resulting in complete data loss at logout.

### 2. The Profile Registration Registry Structure
Windows tracks user profiles in the registry:
- **Path**: `HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\ProfileList`
- Each profile has a subkey named after the user's Security Identifier (SID) (e.g., `S-1-5-21-...`).
- **Key Values**:
  - `ProfileImagePath`: Points to the folder path (e.g., `C:\Users\jdoe`).
  - `RefCount`: Monitors active connections to the profile.
  - `State`: Flags the current profile configuration state.

```
ProfileList
  |---> [S-1-5-21...-1001]      <-- Active User profile configuration
  |---> [S-1-5-21...-1001.bak]  <-- Corrupt profile backup (forces TEMP login)
```

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - A Windows 11 VM with local administrator rights.
> - Back up the registry before editing.

### Step 1: Query User Profiles

```powershell
# Retrieve all user profiles on the system, including SID and folder path details
Get-CimInstance -ClassName Win32_UserProfile | Select-Object LocalPath, SID, Special, Loaded
```

> [!success] Expected Output
> ```
> List of user profile paths and associated Security Identifiers.
> ```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `whoami /user` | Display current user and SID | `whoami /user` |
| `systempropertiesadvanced` | Open Advanced System Properties to manage profiles | `systempropertiesadvanced` |
| `logoff [SessionID]` | Force logoff of a specific user session | `logoff 2` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| Temporary Profile Login | `NTUSER.DAT` locked by AV/service, registry key renamed to `.bak` | Open regedit, delete temporary SID key without `.bak`, remove `.bak` from original key, reset `State`/`RefCount` to 0. |
| Migration Utility Access Denied | Disconnected user session is locking the profile files | Force logoff disconnected session and grant Administrator Full Control on `C:\Users\<username>`. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: User Logged in with a Temporary Profile

> [!example] Ticket
> "I logged into my computer this morning, and all my desktop shortcuts, files, and browser bookmarks are missing. A warning bubble popped up at the bottom saying I'm using a temporary profile."

**L1 Response:** Check active user folders in `C:\Users`. Tell user not to save files.
**Escalation Trigger:** Requires registry editing for permanent fix.
**L2 Resolution:** Checked `ProfileList` in registry. Found `S-1-5...-1001` (TEMP) and `S-1-5...-1001.bak` (Original). Deleted the TEMP key, removed `.bak` from original, set State=0, RefCount=0. Rebooted and user logged in successfully.

---
## 🎤 Interview Questions

> [!question] Q1: How do you resolve a "Logged in with a temporary profile" error in Windows?
> **Answer:** I log in as local administrator, open the registry, navigate to the `ProfileList` path, delete the temporary SID key, remove the `.bak` suffix from the original user's SID key, reset the state and refcount values to 0, and reboot the system to restore the original profile.

==**Exam Tip:** Never delete a user's `C:\Users\<username>` folder directly via File Explorer without deleting the corresponding `ProfileList` registry key, or it creates a persistent temporary profile loop.==

---
## 🔗 Related Notes

- [[02-Operating-Systems/03-Windows-OS/Registry|Registry]] — Details NTUSER.DAT registry hives.
- [[02-Operating-Systems/03-Windows-OS/Event-Viewer|Event Viewer]] — Details Event IDs used to diagnose profile failures.
- [[04-Cloud-and-Security/07-Microsoft-365/OneDrive-Administration|OneDrive Administration]] — Details folder backup synchronization.
