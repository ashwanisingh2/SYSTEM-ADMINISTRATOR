---
tags: [sysadmin, windows-server, file-services, fsrm]
difficulty: Intermediate
lab-required: Yes
read-time: 15 mins
---

# WS-09: File Server and FSRM

> [!abstract] Overview
> This note covers Windows Server file service provisioning, including SMB sharing architecture, NTFS permissions evaluation, and File Server Resource Manager (FSRM) quotas and file screening.

---
## Concept
Think of a corporate File Server as a physical library locker facility. 
- **Share Permissions** are the security guard standing at the front entrance of the locker building. If your name is not on their list, you can't even get through the front door (Share Level).
- **NTFS Permissions** are the specific locks and keys on the individual drawers inside the building. Even if the front guard let you in, you can only open the specific drawers matching your keycard. If a conflict occurs, the most restrictive rule wins (e.g., if the front guard allowed Read-Write, but the drawer lock specifies Read-Only, you are restricted to Read-Only).
- **FSRM** is the automated inventory clerk who checks what you bring. If you try to dump contraband (unapproved files like `.mp3` or `.exe` movies/games) in the lockers, the clerk blocks you (File Screening) and restricts how much weight you can store (Quotas).

*Seedha simple mein: Share permissions network connection ko control karti hain aur NTFS permissions local files ko. Dono mein se jo sabse zyada restrictive hoti hai, wahi apply hoti hai. FSRM ke zariye hum storage limits (quotas) aur unapproved file types ko block (file screening) karte hain.*

---
## Technical Deep Dive

### 1. NTFS vs. Share Permissions Evaluation
When a user accesses a shared folder across a network:
- **Share Permissions:** Only apply when accessing the folder via the network path (e.g., `\\server\share`).
- **NTFS Permissions:** Apply both over the network and locally (if logging in direct to the server console).
- **The Calculation Rule:** **Most Restrictive Wins**. 
  - *Example:* If Share Permission is set to **Full Control** (Read/Write/Delete) but NTFS Permission is set to **Read**, the user's effective network permission is **Read**.

### 2. NTFS Permission Inheritance and Deny Rules
- **Explicit vs. Inherited:** Permissions assigned directly on a file/folder are **Explicit**. Permissions passed down from a parent folder are **Inherited**.
- **Explicit Deny overrides all:** If a user belongs to Group A (Allowed Read-Write) and Group B (Explicitly Denied Write), the user is blocked from writing. Explicit Deny overrides any allowed permissions.
- **Inheritance Block:** Disabling inheritance allows you to copy existing inherited permissions as explicit, or strip them entirely to secure the folder.

### 3. Effective Permissions
Because users belong to multiple nested security groups, calculating permissions manually can be error-prone. The **Effective Access** tab in the folder's Advanced Security properties queries Active Directory to calculate the exact, final access rights for a specific user or group on that file.

### 4. File Server Resource Manager (FSRM)
FSRM is a role service that enables storage auditing, management, and security:
- **Quota Management:** Limit the storage space allowed on a folder or volume.
  - **Hard Quota:** Prevents users from saving files once the limit is reached.
  - **Soft Quota:** Does not block file writes, but triggers notifications (email, event logs) when thresholds are crossed. Used for monitoring.
- **File Screening:** Blocks users from saving specific file formats (e.g., blocking audio/video formats to save disk space, or blocking executables to prevent malware).
  - **Active Screening:** Blocks file writes and alerts.
  - **Passive Screening:** Allows file writes but logs audit alerts.
- **File Groups:** Logical containers defining file extensions (e.g., Image Files = `*.jpg`, `*.png`).

---
## Windows Server CLI Configuration Commands

### Creating an SMB Share via PowerShell
```powershell
# Install File Server role services
Install-WindowsFeature -Name FS-FileServer -IncludeManagementTools

# Create a new folder and share it over the network
New-Item -Path "C:\Shares\Finance" -ItemType Directory
New-SmbShare -Name "Finance_Share" -Path "C:\Shares\Finance" -FullAccess "COMPANY\Domain Admins" -ChangeAccess "COMPANY\Finance_Staff" -ReadAccess "Everyone"
```

### NTFS Permission Management (icacls)
```cmd
:: Windows Command Prompt
:: Grant Read/Write access on folder to Finance group
icacls "C:\Shares\Finance" /grant "COMPANY\Finance_Staff":(OI)(CI)RXW

:: (OI) = Object Inherit (files inherit permissions)
:: (CI) = Container Inherit (subfolders inherit permissions)
:: RXW = Read, Execute, Write permissions
```

---
## Lab — Step by Step
> [!info] Lab Setup Needed
> A Windows Server 2022 VM (`SVR-FS01`), an OU containing a test user `jdoe` who is a member of the group `G_Finance`.

### Step 1: Create Folder with Custom NTFS Permissions
1. On `SVR-FS01`, create folder `C:\CompanyData\Finance`.
2. Right-click `Finance` -> **Properties** -> **Security** tab.
3. Click **Advanced**. Click **Disable inheritance**.
4. Select **Convert inherited permissions into explicit permissions on this object** (retains default rules for editing).
5. Select the **Users** group and click **Remove** (removes general domain user access).
6. Click **Add**. Add the group **`G_Finance`**. Grant them: **Modify**, **Read & execute**, **List folder contents**, **Read**, and **Write**.
7. Click OK and Apply.

### Step 2: Share the Folder over SMB
1. Go to the **Sharing** tab of the `Finance` folder properties. Click **Advanced Sharing**.
2. Check **Share this folder**. Share name: `Finance$`. (Appending `$` makes the share hidden from network browsing).
3. Click **Permissions**.
4. Select **Everyone**. Change permissions to **Full Control**. (Note: We set Full Control at the share level, allowing NTFS permissions to handle actual security filtering. This is industry standard).
5. Click OK and Apply.

### Step 3: Install FSRM and Apply Quota
1. Open Server Manager -> Add Roles and Features. Install **File Server Resource Manager** (located under File and Storage Services -> File and iSCSI Services).
2. Open Tools -> **File Server Resource Manager**.
3. Expand **Quota Management** -> right-click **Quotas** -> **Create Quota**.
4. Path: Browse to `C:\CompanyData\Finance`.
5. Select: **Derive properties from quota template**. Choose **100 MB Limit**.
6. Click **Create**.
7. To verify, log into a client machine as `jdoe`, open `\\SVR-FS01\Finance$`, and attempt to copy a 150MB file. The write will fail with an "Out of Space" warning.

---
## Troubleshooting Scenarios

**Scenario 1:**
- **Problem:** A user in the `G_Finance` group cannot open a PDF inside the `Finance$` share, receiving "Access Denied." Running `gpresult` shows they belong to the correct group.
- **Root Cause:** A permission conflict. An administrator explicitly denied access to the "Everyone" group on the folder's NTFS ACL, or the user is also a member of a group (like `Temp_Staff`) that has an explicit Deny write rule on the file.
- **Fix:**
  1. Open the folder Properties -> Security -> **Advanced**.
  2. Go to the **Effective Access** tab.
  3. Select the failing user account and click **View effective access**.
  4. Look for red **X** marks indicating where access is blocked.
  5. Remove any explicit Deny rules assigned to groups the user belongs to, replacing them with clean permission scopes.

**Scenario 2:**
- **Problem:** A user attempts to save an Excel sheet named `SalesReport.xlsx` in their department folder but receives: "You do not have permission to save files to this location." Pinging the server and opening other files works.
- **Root Cause:** An active FSRM File Screen is blocking Microsoft Office spreadsheet formats from being saved, or a Quota limit has been reached on the folder.
- **Fix:**
  1. Log into the File Server. Open FSRM.
  2. Check **File Screening Management** -> **File Screens**.
  3. Verify if a screen is active on the target path (e.g., blocking Office files, or blocking media files).
  4. If a block is active, modify the screen properties to exclude `.xlsx` or move the user's file format to an approved location.
  5. Check if the folder has exceeded a Hard Quota threshold. If yes, expand the quota limit.

---
## Common Mistakes
> [!warning] Avoid These
> **Setting highly restrictive Share permissions alongside NTFS permissions:** Configuring Read-Only share permissions for everyone, and then configuring Modify permissions on NTFS. This leads to configuration confusion: users will be blocked from writing over the network, but can write if they log in locally, creating inconsistent access behaviors.
> **Correct approach:** Set Share permissions to "Everyone - Full Control", and manage all granular folder security rules exclusively on the NTFS permissions tab.

---
## Pro Tips
> [!tip] Field Experience
> Always enable **Access-Based Enumeration (ABE)** on shared folders. When ABE is active, the server hides files and folders from users who do not have read permissions for them. This keeps the share interface clean and prevents users from requesting access to folders they shouldn't even know exist.

---
## Quick Revision Table
| # | Concept | One Line Summary |
|---|---------|-----------------|
| 1 | Share vs NTFS | Share rules govern network path access; NTFS governs local and network files. Most restrictive wins. |
| 2 | Explicit Deny | A permission setting that overrides any other allowed group permissions for that user. |
| 3 | ABE | Access-Based Enumeration; hides folders and files from users who lack read access. |
| 4 | Hard Quota | Storage limit inside FSRM that physically blocks file writes once the threshold is crossed. |
| 5 | File Screening | FSRM service that blocks specific file formats (e.g., `.mp3`, `.exe`) from being saved. |

---
## Interview Q&A

**Q1: Explain how Windows Server calculates the final effective permissions for a user accessing a file over the network, taking into account multiple group memberships.**
A: Windows calculates effective permissions using a two-step process:
1. **NTFS Permission Calculation:** The server gathers the user's account SID and all security group SIDs. It evaluates the NTFS ACLs. Allowed permissions are cumulative (if Group A grants Read, and Group B grants Write, the user has Read-Write). However, any explicit Deny rule instantly overrides allowed permissions.
2. **Share Permission Calculation:** The server evaluates the Share ACLs in the same cumulative manner.
3. **The Comparison:** The server compares the final NTFS permission against the final Share permission and enforces the **most restrictive** of the two.

**Q2: A client demands a quota system on user home folders. Each user should get a 5GB storage limit, but they want alerts sent to the IT ticketing system when a user hits $85\%$ capacity. Explain how you would configure this.**
A: 
- **Situation:** Home folders need a 5GB hard limit and an $85\%$ capacity alert trigger.
- **Task:** Create and deploy an FSRM Quota Template matching the criteria.
- **Action:** First, I will open File Server Resource Manager (FSRM). Second, I will create a new Quota Template named `5GB_HomeFolder_Limit`. I will set a **Hard Quota** limit of 5 GB. Third, I will add a **Notification Threshold** at $85\%$ capacity. Under the threshold properties, I will configure an email alert to send a ticket template to the IT helpdesk email address, and write an event log entry.
- **Result:** I will apply this quota template to the parent home directory path as an auto-apply quota, generating the limits dynamically for all existing and new subfolders.

**Q3: What is the difference between copying a file and moving a file across different volumes in terms of NTFS permissions?**
A: 
- **Copying a file:** Whether copying within the same volume or to a different volume, the file is treated as a new object. It discards its original permissions and **inherits** the NTFS permissions of the destination parent folder.
- **Moving a file within the same NTFS volume:** The file retains its original explicit permissions and does not inherit the destination parent's permissions.
- **Moving a file to a different NTFS volume:** The file behaves as a copy-and-delete operation. It discards its original permissions and **inherits** the destination parent's permissions.

---
## Related Notes
- [[03-Identity-and-Core-Services/06-Active-Directory/WS-05 Group Policy — Complete Guide|WS-05 Group Policy — Complete Guide]] — Deploying drive maps via policy preferences.
- [[03-Identity-and-Core-Services/05-Windows-Server/WS-10 DFS — Distributed File System|WS-10 DFS — Distributed File System]] — Creating virtualized namespaces for shares.
- [[03-Identity-and-Core-Services/05-Windows-Server/WS-11 Storage — Disk Management and RAID|WS-11 Storage — Disk Management and RAID]] — Formatting volumes with NTFS.

