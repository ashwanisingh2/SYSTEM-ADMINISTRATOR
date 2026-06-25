---
tags: [desktop-support, m365, sharepoint, permissions, L2]
aliases: [sharepoint-guide, spo-permissions, hub-sites]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #md-102
---

# SharePoint Online

---

## Concept Overview
- **What it is**: SharePoint Online is Microsoft's cloud-based document management, collaboration, and intranet platform. It serves as the primary storage repository for Microsoft 365, powering files stored in Teams, OneDrive, and enterprise portals.
- **Why it matters for a support engineer**: A support engineer must understand SharePoint's complex permission models to troubleshoot document access blocks, site permission inheritance issues, external guest invitation failures, and synchronization locks.
- **Where you encounter this in real job**: Creating site collections, resolving "Access Denied" errors, setting up hub site links, breaking inheritance on directories, and recovering files from the site collection recycle bin.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Adds users to default SharePoint groups, grants folder access, restores files from the first-stage recycle bin, and helps users sync libraries locally.
  - **L2**: Manages site permission inheritance, configures site-level external sharing settings, registers site members, and handles hub site associations.
  - **L3**: Establishes tenant-wide sharing rules, configures Sensitivity Labels for data loss prevention (DLP), manages tenant storage allocations, and designs migration plans from local file shares to SharePoint using the SharePoint Migration Tool (SPMT).

---

## Technical Deep Dive

### 1. Site Collection Architecture
SharePoint Online is organized into **Site Collections** (independent containers holding document libraries, lists, and pages).

```
                            [SharePoint Tenant]
                        (company.sharepoint.com)
                                   |
         +-------------------------+-------------------------+
         |                                                   |
         v                                                   v
 [Communication Site]                                   [Team Site]
 - Intranet Portal (Read-heavy)               - Group Collaboration (Read/Write)
 - No M365 Group connection                   - Connected to M365 Group & Teams
 - Org-wide dissemination                     - Private or Public membership
```

- **Communication Sites**: Designed for broadcasting information (intranets). Typically public within the organization, with a few editors and many readers.
- **Team Sites**: Designed for active group collaboration. Always connected to a Microsoft 365 Group (which includes a shared mailbox, calendar, Planner, and a Microsoft Team).
- **Hub Sites**: Parent templates that connect and organize multiple Team or Communication sites together, providing shared navigation, uniform search, and consolidated news feeds.

### 2. SharePoint Permissions Model
SharePoint uses three default groups to manage access, though custom groups can be defined:
1. **Owners (Full Control)**: Can modify site settings, design layouts, delete the site, and manage permissions.
2. **Members (Edit)**: Can add, edit, and delete files, list items, and pages. By default, they *cannot* modify site-level permission structures.
3. **Visitors (Read)**: Can view files and pages but cannot add or modify content.

#### Permission Inheritance:
By default, subfolders and document libraries inherit permissions from their parent site. To restrict access to a specific folder:
- **Break Inheritance**: Wipes inherited permissions and creates a copy of active permissions on that specific folder, which can then be edited independently.
- *Warning*: Breaking inheritance excessively creates administrative overhead and makes permission auditing extremely difficult.

### 3. External Sharing Capabilities
Sharing settings can be adjusted at the tenant level and overridden at the site collection level:
- **Anyone (Anonymous)**: Links allow access without requiring login. Highly restricted or disabled in most corporate environments.
- **New and Existing Guests**: External users must sign in or provide a verification code to authenticate.
- **Existing Guests Only**: Only users already present in the Entra ID Guest directory can access sharing links.
- **Only People in Your Organization**: Disables all external sharing.

---

## Commands & Syntax

### PowerShell
SharePoint is administered using the `Microsoft.Online.SharePoint.PowerShell` module.
```powershell
# Install and connect to SharePoint Online Admin Center
Install-Module -Name Microsoft.Online.SharePoint.PowerShell -Force
Connect-SPOService -Url "https://company-admin.sharepoint.com"

# Create a new Communication Site Collection
New-SPOSite -Url "https://company.sharepoint.com/sites/Intranet-Marketing" -Owner "admin@company.com" -StorageQuota 1048576 -Title "Marketing Portal" -Template "SITEPAGEPUBLISHING#0"

# Set a site collection's external sharing policy to allow new and existing guests
Set-SPOSite -Identity "https://company.sharepoint.com/sites/Intranet-Marketing" -SharingCapability ExternalUserAndGuestSharing

# List all external guest users having access to the SharePoint site
Get-SPOExternalUser -SiteUrl "https://company.sharepoint.com/sites/Intranet-Marketing"
```

### CMD / Run Box
```cmd
:: Validate tenant DNS resolution to ensure name lookup works
nslookup company.sharepoint.com
```

### GUI Path
- **Admin Center**: Go to **admin.microsoft.com** -> **SharePoint** (SharePoint Admin Center) -> **Active Sites**.
- **Site Permissions**: Go to target Site Home Page -> Click **Settings** (Gear icon) -> **Site Permissions** -> **Advanced site settings**.

---

## Real-World Scenarios

### Scenario 1: External Client Receives "Access Denied" When Accessing Folder
**User Complaint:** A project manager reports: *"I shared a folder in our client site with an external vendor (vendor@external.com). When they click the link in their email, they get an 'Access Denied' screen. We need them to access this file immediately."*
**Your First 3 Checks:**
1. Check the tenant-wide sharing setting for SharePoint.
2. Check the specific Site Collection's sharing capabilities.
3. Verify if the vendor accepted the external invitation using their correct account.
**Diagnosis Steps:**
1. Log in to the SharePoint Admin Center. Select the site: `https://company.sharepoint.com/sites/ProjectAlpha`.
2. Check the Site details.
   - Sharing status: `Only people in your organization`.
   - *Even though the user generated a share link, the site-level security policy overrode and blocked it.*
3. Open SharePoint PowerShell and verify:
   `Get-SPOSite -Identity "https://company.sharepoint.com/sites/ProjectAlpha" | Select SharingCapability`
   - Output: `Disabled`.
**Root Cause:** The SharePoint site collection's external sharing permission was set to block all external access, overriding user-created guest sharing links.
**Fix:**
1. Change the site sharing capability using PowerShell:
   `Set-SPOSite -Identity "https://company.sharepoint.com/sites/ProjectAlpha" -SharingCapability ExternalUserAndGuestSharing`
2. Instruct the project manager to re-send the folder sharing invite.
3. Have the vendor open the link in an Incognito window, select "Send Code" to verify their identity, and access the folder.
**Prevention:** Establish a process where site collections intended for external client collaboration are explicitly created with external sharing enabled during provisioning.
**Ticket Close Note:** "Modified site-level sharing capability to ExternalUserAndGuestSharing. Vendor successfully authenticated and accessed the folder. Closed."

### Scenario 2: Unlocking a File Checked Out by a Terminated User
**User Complaint:** A department coordinator complains: *"We are trying to edit the 'Q3-Budget.xlsx' file, but Excel says it is locked for editing by 'Jane Doe', who was terminated yesterday. We cannot check it back in or edit it."*
**Your First 3 Checks:**
1. Determine if the document library has "Require check-out" enabled.
2. Verify if you have Site Collection Administrator permissions on the site.
3. Check the check-out status of the file in the browser.
**Diagnosis Steps:**
1. Open the document library in SharePoint Web.
2. Find the 'Q3-Budget.xlsx' file. Notice the little green arrow icon pointing away from the file, indicating it is checked out.
3. Hovering shows: `Checked out to Jane Doe`.
4. Because Jane's account is disabled and her session terminated, the lock remains on the database, preventing other editors from modifying it.
**Root Cause:** A file was left checked out by a user whose account was disabled, blocking editing permissions for other team members.
**Fix:**
1. Log in with a Site Collection Administrator account.
2. Navigate to the document library -> Hover over the file -> Click the three vertical dots (Show actions) -> **More** -> **Discard check-out** (or **Check in**).
3. Select **Discard check-out** to release the lock immediately (Warning: any unsaved local edits Jane made prior to exit will be lost).
4. Verify that the file lock is removed and other members can edit.
**Prevention:** Turn off "Require check-out" in document library settings unless version control is strictly required by business compliance.
**Ticket Close Note:** "Discarded check-out session for terminated user. Lock released. Verified file is editable by other department members. Closed."

---

## Critical Points

> [!danger] Never Do This
> - Never grant users "Full Control" permissions to resolve a simple file access issue.
> - Giving Full Control allows users to delete the site collection, alter metadata schemas, and add other users without security approval. Always follow the principle of least privilege, using "Edit" or "Read" groups.

> [!warning] Common Trap
> - Confusing "SharePoint Groups" with "Microsoft 365 Groups".
> - SharePoint Groups exist strictly inside the SharePoint database schema (Owners, Members, Visitors). Microsoft 365 Groups exist in Entra ID and control access to multiple tools (Teams, Exchange, Planner). Modifying members in a Teams app edits the M365 Group, which syncs to the SharePoint site Members group.

> [!tip] Senior Engineer Tip
> - When users complain that SharePoint files synced to their computer via OneDrive are running slowly or showing constant sync conflicts, check the library limit. Microsoft officially supports syncing up to **300,000 files** across all synced libraries. Syncing more than 300,000 files will overload the local OneDrive client database and cause sync failures.

> [!success] Verification Steps
> - Run: `Get-SPOSite -Identity "https://site-url" | Select SharingCapability` to confirm the target sharing configuration is active.
> - Verify in the browser site permissions page that the correct user accounts are listed under site group memberships.

> [!question] Interview Alert
> - "What happens to a subfolder's permissions when you select 'Inherit Permissions' after breaking inheritance?"
> - Answer: "Selecting 'Inherit Permissions' deletes all custom/unique permission structures assigned to that subfolder and replaces them with a copy of the parent folder's permissions. Any custom user access granted to that subfolder is lost."

---

## Common Mistakes & Fixes

| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| Breaking permission inheritance on individual files | Trying to secure specific files quickly | Create a separate document library or site collection for sensitive files instead of breaking inheritance on files. |
| Exceeding the character limit for file paths | Nested folders and long file names | Keep folder nesting minimal and use SharePoint metadata fields for filtering instead. |
| Deleting a SharePoint site to clear disk space | Unaware of Teams integration | Deleting a Team Site deletes the connected Microsoft Team, shared mailbox, and files. |

---

## Lab Exercise

**Objective:** Access the SharePoint Online Admin Center (or PowerShell), build a new Site Collection, break permission inheritance on a sub-folder, and configure external guest sharing limits.
**Time Required:** 30 minutes
**Environment Needed:** A computer with internet access and a Microsoft 365 Developer tenant.
**Pre-requisites:** Global Admin or SharePoint Admin credentials.

**Steps:**
1. Open PowerShell and connect to your SharePoint Admin Center:
   ```powershell
   Import-Module Microsoft.Online.SharePoint.PowerShell -ErrorAction SilentlyContinue
   Connect-SPOService -Url "https://yourtesttenant-admin.sharepoint.com"
   ```
2. Create a test Team Site collection:
   ```powershell
   New-SPOSite -Url "https://yourtesttenant.sharepoint.com/sites/Lab-Collage" -Owner "admin@yourtesttenant.onmicrosoft.com" -StorageQuota 1048576 -Title "Lab Collaboration"
   ```
3. Set the sharing capability to restrict sharing only to external users who exist in the directory:
   ```powershell
   Set-SPOSite -Identity "https://yourtesttenant.sharepoint.com/sites/Lab-Collage" -SharingCapability ExistingExternalUserSharing
   ```
4. Verify the settings:
   ```powershell
   Get-SPOSite -Identity "https://yourtesttenant.sharepoint.com/sites/Lab-Collage" | Select SharingCapability
   ```
5. Navigate to the site in your browser. Open the "Documents" library, create a folder named `Confidential`, open its access settings, and click **Stop Inheriting Permissions** to break inheritance.

**Success Criteria:** The site is provisioned, its sharing configuration restricted, and permission inheritance successfully broken on the subfolder.
**Common Failures:** The `New-SPOSite` command fails if the site path suffix matches an existing active site (site names must be unique).

---

## Interview Questions & Answers

### Basic (L1 Level)
**Q: What are the three default permission groups created with a new SharePoint site, and what are their rights?**
A: The three default groups are Owners (who have Full Control to design and administer the site), Members (who have Edit access to add, modify, and delete files), and Visitors (who have Read-only access to view pages and download documents).

**Q: Where do you go to recover a file that was deleted from a SharePoint library 60 days ago?**
A: SharePoint uses a two-stage recycle bin. When a user deletes a file, it goes to the site's Recycle Bin for 93 days. If it's deleted from there, it moves to the second-stage (Site Collection) Recycle Bin for the remainder of the 93 days. I can log in as a site administrator, open the site Recycle Bin, click the Second-Stage link, select the file, and restore it.

### Intermediate (L2 Level)
**Q: What does "Breaking Inheritance" mean in SharePoint permissions, and when would you do it?**
A: Breaking inheritance means separating a document library, list, or folder from the parent site's permissions. Once broken, the parent site's permissions no longer flow down to the folder. You do this when you have a folder containing sensitive data that must only be viewed by a small group of users, while keeping the rest of the site open to everyone.

**Q: Why does deleting an M365 Group-connected SharePoint site cause an outage in Microsoft Teams?**
A: When a Team is created in Microsoft Teams, a Microsoft 365 Group is generated, and a SharePoint Team site is provisioned to store all the channel files. If you delete the SharePoint site collection, you delete the files and folders linked to the Team. This breaks file access for the Team, causing failures for all users.

### Advanced (L3/Senior Level)
**Q: A user reports that when they attempt to sync a SharePoint document library to their PC, the OneDrive client stalls, throws sync errors, and slows down their laptop. How do you resolve this?**
A:
- **Situation**: User's sync client is failing during a large SharePoint library synchronization.
- **Task**: Identify and fix the sync stagnation and client resource overload.
- **Action**: First, I check the size of the SharePoint library. If the library exceeds 300,000 files, I advise the user that syncing this library will overload the local client's database. Instead, I instruct them to add a shortcut to the library in OneDrive ("Add shortcut to My Files") or use the web interface. If the file count is below the limit, I run `onedrive.exe /reset` to clean the database state, and ensure Windows Files On-Demand is enabled to prevent downloading the entire library structure.
- **Result**: The file count was 450,000. Replacing the sync link with a OneDrive shortcut resolved the local database locking, and the client resumed normal operations.

### HR / Behavioral
**Q: Describe a time you had to explain a complex technical limitation to a non-technical manager who was frustrated by it.**
A: A marketing manager was frustrated because they couldn't upload a folder containing subfolders and files with names like `A&B_Partners/Update?.pdf` to SharePoint. I explained that SharePoint uses a database backend, and characters like `&` and `?` are used in web URLs, which causes database lookup errors. I helped them write a quick PowerShell script to rename the files, removing the illegal characters. They appreciated the explanation and the solution to automate the rename process.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **What**: The core document management and collaboration platform powering files in M365.
> **Why**: Critical for file storage, intranet sites, and Teams collaboration structures.
> **How**: Provision sites, configure permission levels (Owner, Member, Visitor), manage inheritance, and set external sharing settings.
> **Command**: `Set-SPOSite -SharingCapability` / `New-SPOSite`
> **Interview Answer Starter**: "SharePoint Online structures data into Site Collections, managing permissions through inherited groups. To troubleshoot external access..."

**Key Numbers to Remember:**
- Default retention for deleted files: 93 days (combined first/second stage)
- Recommended file sync limit for OneDrive client: 300,000 files
- Maximum URL character path length: 400 characters
- Port required for connection: TCP 443 (HTTPS)

**3 Things Interviewer Wants to Hear:**
- SharePoint stores files for Teams and OneDrive
- Breaking inheritance allows unique permissions on folders/libraries
- Why to avoid granting Full Control (Owners) permissions to standard users

---

## Related Notes
- [[04-Cloud-and-Security/07-Microsoft-365/OneDrive-Administration|OneDrive Administration]] — Detail how personal folders link to SharePoint.
- [[04-Cloud-and-Security/07-Microsoft-365/Teams-Administration|Teams Administration]] — Focuses on how Teams files are stored in site folders.
- [[04-Cloud-and-Security/09-Security/Access-Management|Access Management]] — Explains RBAC and permissions design structures.

---

## Tags
#desktop-support #m365 #sharepoint #permissions #L2 #interview-topic #lab-complete #daily-use

