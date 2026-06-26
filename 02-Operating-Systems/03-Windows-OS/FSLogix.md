---
tags: [desktop-support, windows-os, profiles, FSLogix, L2]
aliases: [fslogix, fslogix-profiles]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #none
---

# FSLogix Profile Management

> [!note] Overview
> This note covers FSLogix Profile Container configuration and administration in enterprise virtual desktop (RDS/VDI/AVD) environments. It details profile redirection via VHD/VHDX containers, Cloud Cache implementation, storage permissions, and profile mount troubleshooting.

---
## Concept Overview
- **What it is** — FSLogix is a profile management solution designed for virtualization environments (VDI/RDS/Azure Virtual Desktop). It redirects user profiles to a remote network storage location by dynamically mounting a user-specific Virtual Hard Disk (VHD or VHDX) file during session logon.
- **Why it matters** — Traditional profile roaming copy mechanisms cause massive logon delays (known as "logon storms") and are prone to file corruption. FSLogix eliminates this by mounting the container in sub-seconds, making virtual desktops behave identically to physical PCs with local storage.
- **Real job encounter** — Resolving slow user logon tickets in Citrix, RDS, or Azure Virtual Desktop (AVD) host pools, managing Outlook caching, and dealing with temporary profile errors.
- **L1 vs L2 vs L3 responsibility split:**
  - **L1 Resolution**: Identify locked user profile containers, log off disconnected user sessions, and check network share disk space capacity.
  - **Escalation Trigger**: Escalate if FSLogix fails to mount for all users, or if user sessions default to temporary profile configurations due to registry/permission locks.
  - **L2 Resolution**: Install the FSLogix agent on master OS gold images, configure registry options, check file permissions on SMB shares, and clean up orphan locks.
  - **L3 Resolution**: Architect highly available profile storage (Azure Files, Azure NetApp Files, clustered storage), configure Cloud Cache configurations for disaster recovery, and design container sizing profiles.

---
## Technical Deep Dive

### 1. How FSLogix Profile Containers Work
Rather than copying the user profile files over the network during logon and copying them back during logoff:
1. When a user authenticates, the FSLogix driver (`frxdrv.sys`) queries the registry for file storage locations.
2. It locates the user's specific virtual disk (`Profile_username.vhd`).
3. The disk is dynamically mounted to the server and mapped to `C:\Users\username`.
4. The OS and applications process reads and writes directly to the mounted disk in real time.
5. On logoff, the disk is unmounted instantly, ensuring zero data loss or synchronization lags.

```
+---------------+              +------------------------------------------+
|  User Client  |              |        Virtual Host Server (RDS/VDI)     |
+---------------+              +------------------------------------------+
        |                                           |
        |---- Log In ------------------------------>| (FSLogix driver intercepts)
        |                                           |
        |                                           |-- Mounts VHDX file ----+
        |                                           |                        |
        |                                           |<-----------------------+
        |                                           |
        |                                           v
        |                                   [ C:\Users\Username ]
        |                               (Virtual mount points to SMB)
        |                                           |
        |                                           v
        |                                 +-----------------------+
        |                                 |   Network File Share  |
        |                                 | \\server\profiles     |
        |                                 | (Stores User.vhdx)    |
        |                                 +-----------------------+
```

### 2. Profile Container vs. Office Container
- **Profile Container**: Stores the core user profile metadata, app configurations, desktop files, registry hives (`NTUSER.DAT`), and user folders.
- **Office Container**: Stores non-critical, easily replaceable Outlook OST cache files, Teams data, OneDrive caches, and search indexing databases. This separation prevents the primary profile container from growing excessively.

### 3. Cloud Cache Configurations
Cloud Cache is an advanced FSLogix feature that writes profile updates to multiple storage providers simultaneously (e.g., local storage and Azure Files). If the primary cloud storage encounters an outage, the local cache handles requests and syncs data once connection is restored, ensuring offline resiliency.

---
## Step-by-Step Lab

> [!warning] Pre-requisites
> - A Windows Server 2022 domain controller (`DC-01.corp.local`).
> - A member fileserver (`SVR-FS01`) hosting the profile directory.
> - A virtual host server (`SVR-VDI01`) running Windows Server or Windows 10/11 Enterprise.
> - Administrative credentials.

### Step 1: Configure NTFS Permissions on the Profile File Share
For FSLogix to operate securely, configure share permissions on `SVR-FS01` inside folder `D:\FSProfiles`:

1. Right-click the `FSProfiles` folder -> **Properties** -> **Security** -> **Advanced**.
2. Disable inheritance and convert permissions.
3. Configure the permissions table as follows:

| Group / User | Apply To | Permissions |
|---|---|---|
| **SYSTEM** | This folder, subfolders, and files | Full Control |
| **Administrators** | This folder, subfolders, and files | Full Control |
| **Domain Users** | This folder only | Read, Write, List folder contents |
| **CREATOR OWNER** | Subfolders and files only | Full Control |

### Step 2: Install FSLogix Apps Agent on Virtual Host
1. Log in to `SVR-VDI01`. Download the FSLogix installation zip from Microsoft.
2. Run `FSLogixAppsSetup.exe` as Administrator.
3. Accept licensing and click **Install**. No reboot is required.

### Step 3: Configure Registry Settings for Redirection
We will enable FSLogix and point it to the profile share via the registry.

1. Open PowerShell as Administrator on `SVR-VDI01` and execute:
```powershell
# Enable FSLogix Profiles
reg add "HKLM\Software\FSLogix\Profiles" /v Enabled /t REG_DWORD /d 1 /f

# Set network share path for VHD storage
reg add "HKLM\Software\FSLogix\Profiles" /v VHDLocations /t REG_SZ /d "\\SVR-FS01\FSProfiles" /f

# Enforce VHDX format (better stability than VHD)
reg add "HKLM\Software\FSLogix\Profiles" /v VolumeType /t REG_SZ /d "vhdx" /f

# Enable Profile dynamic shrink to save storage
reg add "HKLM\Software\FSLogix\Profiles" /v SizeInMBs /t REG_DWORD /d 30720 /f # 30 GB Limit
```

### Step 4: Verify Deployment and Container Mounting
1. Log in to `SVR-VDI01` as a standard test domain user (e.g. `jdoe`).
2. Open File Explorer on `SVR-FS01` and browse to `D:\FSProfiles`.
3. Verify that a folder `S-1-5-..._jdoe` has been created, containing `Profile_jdoe.vhdx`.
4. On `SVR-VDI01`, open Disk Management (`diskmgmt.msc`) and verify that a virtual disk is attached and pointing to the user profile path.

---
## Cheat Sheet / Quick Reference

| Registry Path / Tool | Purpose | Configuration Example |
|---|---|---|
| `HKLM\Software\FSLogix\Profiles` | Primary root key for general profile settings | Root Key |
| `Enabled (DWORD)` | Turns FSLogix profile redirection On (1) or Off (0) | `1` |
| `VHDLocations (REG_SZ)` | Set UNC paths to SMB target fileserver | `\\server\share` |
| `DeleteLocalProfileWhenVHDShouldApply` | Deletes local profile conflicts to prevent temp profiles | `1` |
| `frx.exe` | Command-line utility for managing FSLogix drivers | `frx list-redirects` |
| `frxcontext.exe` | Diagnoses context rules and app rules | `frxcontext -h` |
| **Local logs path** | System events path for mount diagnostics | `C:\ProgramData\FSLogix\Logs\Profile` |

---
## Troubleshooting Table

| Problem | Cause | Fix |
|---|---|---|
| User logs in and gets a temporary profile (`TEMP`). | Network share permissions are incorrect, or the user already has a local profile folder blocking the redirect. | Re-verify NTFS permissions matching the creator-owner configuration. Set registry key `DeleteLocalProfileWhenVHDShouldApply` to `1`. |
| Login fails with error: "The user profile failed to attach. Please contact support." | The user's VHDX file is locked on the fileserver by a previous disconnected session. | Open Computer Management on `SVR-FS01` -> **System Tools** -> **Shared Folders** -> **Open Files**. Locate the user's folder/VHDX lock, right-click, and select **Close Open File**. |
| User profile container is growing rapidly and running out of disk space. | Large cache directories (Teams, Spotify, browser caches) are filling the container. | Implement FSLogix Redirections XML (`redirections.xml`) configuration to exclude application cache folders from saving to the VHDX. |
| Slow logons during peak office hours ("Logon Storms"). | Fileserver storage throughput (IOPS) is saturated. | Move profiles to high-speed SSD/NVMe storage pools, or configure FSLogix Cloud Cache to offload initial read operations to local disks. |
| User files are missing after logging in from a different session host. | The second host has FSLogix disabled, or is pointing to a different VHDLocations share. | Ensure the FSLogix configurations are consistently deployed to all host pool members using a Group Policy Object (GPO). |

---
## Interview Questions

> [!question] L1 Question
> **Q:** How do you resolve a ticket where a user's desktop login hangs with a message saying FSLogix is waiting for the user profile container to attach?
> **A:** This usually happens when the user's VHDX file is locked by a previous session host (often due to an improper logoff or crash). I will log in to the profile fileserver, open **Computer Management**, go to **Shared Folders > Open Files**, search for the user's username, right-click the active file handle lock, and select **Close Open File** to release the container.

> [!question] L2 Question
> **Q:** What is the difference between Profile Containers and Office Containers in FSLogix? Why would you separate them?
> **A:** Profile Containers store user settings, application data, registry configs, and documents. Office Containers store replaceable Microsoft Office cache data, such as Outlook OST files, Teams cache, and OneDrive local caches. We separate them because Office data changes rapidly and expands container sizes. Separating them keeps the core profile container small and fast to back up, while allowing the Office container to be deleted and rebuilt without losing settings.

> [!question] L3/Scenario Question
> **Q:** A customer complains that when their primary Azure datacenter encounters an outage, users lose access to virtual desktops because FSLogix profiles cannot be read. How do you design a disaster recovery solution for FSLogix profile storage?
> **A:** 
> - **Situation:** Cloud profile storage outage causes complete virtual desktop downtime.
> - **Task:** Design a highly resilient profile storage system.
> - **Action:** I will implement **FSLogix Cloud Cache**. Instead of configuring standard `VHDLocations`, I will configure `CCDLocations` (Cloud Cache Directory). I will define two storage locations: 1) A high-speed local SMB share in the primary site, and 2) An Azure Files storage account in a secondary DR region.
> - **Result:** The Cloud Cache driver writes profile changes to both locations in the background. If the primary site fails, the driver automatically redirects read/write operations to the secondary Azure Files instance, ensuring zero user disruption.

---
## Seedha Simple Mein
*Seedha simple mein: FSLogix ek aisi technology hai jo user profiles ko network par ek Virtual Disk (VHDX file) ke roop mein store karti hai. Jab user login karta hai, toh yeh disk instant mount ho jati hai. Roaming profiles ke mukable isse login speed bohot fast ho jati hai aur profiles corrupt nahi hotin.*

---
## Related Notes
- [[02-Operating-Systems/03-Windows-OS/User-Profiles|User-Profiles]] — Traditional profile configurations in Windows.
- [[03-Identity-and-Core-Services/05-Windows-Server/WS-09 File Server and FSRM|WS-09 File Server and FSRM]] — Hosting file shares and configuring quotas.
- [[04-Cloud-and-Security/08-Azure/AZ104-02 Azure Storage Administration|AZ104-02 Azure Storage Administration]] — Utilizing Azure Files for profile hosting.

---
*Tags: #desktop-support #windows-os #profiles #FSLogix #L2*
