---
tags: [desktop-support, m365, collaboration, L1]
aliases: [m365-04-sharepoint-online-administration, m365-04]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #none
---

# M365-04: SharePoint Online Administration

> [!abstract] Overview
> This note covers SharePoint Online architecture, site collection management, permission models, sharing controls, and integration with OneDrive. It details how to design secure file collaboration environments and troubleshoot synchronization failures.

---

---
## Concept Overview
Think of SharePoint Online as a digital city. The "Tenant" is the entire city map. "Site Collections" are individual corporate complexes (e.g., HR, Finance, Operations). "Sites" are rooms inside these buildings containing filing cabinets ("Document Libraries") that hold folders and files. "Hub Sites" act as transport rings or corridors linking these independent buildings together under a common theme and search index.

---

---
## Technical Deep Dive
### 1. SharePoint Architecture
- **Site Collections**: Top-level containers that act as security and management boundaries. Each site collection has its own storage quota, templates, and owners.
- **Modern Experience**: Fast, responsive, mobile-friendly layouts. Built around **Team Sites** (connected to Microsoft 365 Groups for team collaboration) and **Communication Sites** (broadcasting information to a wide audience without an underlying M365 group).
- **Classic Experience**: Legacy ASP.NET-style layouts with complex web parts, subsites, and custom master pages. Heavily deprecated.
- **Hub Sites**: Instead of nesting sites via classic hierarchal subsites (which makes permissions and structures rigid), modern SharePoint uses flat Site Collections associated with a central **Hub Site** to share search results, news, and navigation.

### 2. Permissions & Sharing Architecture
- **SharePoint Groups**: Default groups within a site collection:
  - **Owners (Full Control)**: Complete site administration rights.
  - **Members (Edit)**: Can add, modify, and delete files and lists.
  - **Visitors (Read)**: View files and pages, cannot modify.
- **Permission Inheritance**: By default, child items (document libraries, folders, files) inherit permissions from their parent site. Inheritance can be broken at any level to assign unique permissions.
- **External Sharing Tiers**: Can be restricted globally in the SharePoint Admin Center, or overridden at the site collection level (up to the global limit):
  - **Anyone**: Anonymous access links. High risk.
  - **New and Existing Guests**: Requires authentication; guests must sign in or provide a verification code.
  - **Existing Guests**: Only users already present in the directory as Guest objects.
  - **Only People in Your Organization**: Disables external sharing.

### 3. Document Library Controls
- **Versioning**: Saves snapshots of files as modifications occur. Keeps a minimum of 500 major versions. Prevents data loss during ransomware events.
- **Co-Authoring**: Real-time collaborative editing using Office Online or Office Desktop apps. Enabled by default via Office Open XML formats.
- **Check-Out/Check-In**: Forces a file to lock for editing by a single user, preventing others from modifying it until it is checked back in.

### 4. OneDrive for Business Integration
- OneDrive is essentially a personal SharePoint site collection allocated to each user (`/personal/username_domain_com`).
- Admin controls include mapping storage quotas (default 1 TB, expandable to 5 TB or 25 TB depending on license), setting default retention periods for deleted employee accounts (default 30 days), and configuring device access restrictions.

---

## Common Mistakes
> [!warning] Avoid These
> Creating complex hierarchies of nested subsites. Modern SharePoint design requires a flat structure (independent site collections connected by Hubs). Nested subsites complicate site migration and break governance policies.
> Allowing anonymous ("Anyone") sharing globally without setting an expiration date policy. This leads to leaked corporate files floating on public indexing sites forever.
> Breaking inheritance at the individual file level. This generates high performance overhead and makes tracking access rights a nightmare for compliance audits.

---

## Pro Tips
> [!tip] Field Experience
> Implement **Sensitivity Labels** to automate sharing policies. For example, label a document "Highly Confidential" to automatically restrict external sharing and enforce encryption, regardless of site settings.
> Configure **OneDrive Silent Account Config** via Group Policy or Intune. This automatically logs users into their OneDrive sync client using their Windows credentials without prompting them, increasing user adoption instantly.

---

---
## Step-by-Step Lab
> [!info] Lab Setup Needed
> Microsoft 365 Tenant with SharePoint Administrator permissions and SharePoint Online Management Shell installed.

### Step 1: Connect to SharePoint Online PowerShell
Open PowerShell as an Administrator and authenticate to your admin URL:
```powershell
Install-Module -Name Microsoft.Online.SharePoint.PowerShell -Force
Connect-SPOService -Url "https://yourtenant-admin.sharepoint.com"
```

### Step 2: Create a Site Collection
Create a modern Communication site for corporate announcements:
```powershell
New-SPOSite -Url "https://yourtenant.sharepoint.com/sites/CorpNews" -Owner "admin@yourtenant.onmicrosoft.com" -StorageQuota 1048576 -Title "Corporate Announcements" -Template "SITEPAGEPUBLISHING#0"
```

### Step 3: Configure Site External Sharing Settings
Restrict the newly created site's sharing policies so it only allows collaboration with existing directory guests (disallowing anonymous links):
```powershell
Set-SPOSite -Identity "https://yourtenant.sharepoint.com/sites/CorpNews" -SharingCapability ExistingExternalUserSharingOnly
```

### Step 4: Register a Hub Site and Associate Site Collections
Turn `CorpNews` into a Hub site and register another site to inherit its structure:
```powershell
# Register the Hub Site
Register-SPOHubSite -Site "https://yourtenant.sharepoint.com/sites/CorpNews"

# Associate another site to the Hub
Add-SPOHubSiteAssociation -Site "https://yourtenant.sharepoint.com/sites/Finance" -HubSite "https://yourtenant.sharepoint.com/sites/CorpNews"
```

---

---
## Cheat Sheet / Quick Reference
```powershell
Get-SPOSite -Limit All                                # Lists all site collections
Set-SPOSite -Identity "site-url" -LockState NoAccess  # Locks a site (Read-only/No Access)
Remove-SPOSite -Identity "site-url" -NoTrash          # Permanently deletes a site bypassing recycle bin
Get-SPOUser -Site "site-url"                          # Lists users and permissions inside a site
Set-SPOTenant -SharingCapability Disabled             # Disables external sharing tenant-wide
```

---
| # | Concept | One Line Summary |
|---|---------|-----------------|
| 1 | Team Site | SharePoint site associated with a Microsoft 365 Group, designed for internal collaboration. |
| 2 | Communication Site | Standalone site designed to publish news and announcements to the entire org. |
| 3 | Hub Site | Orchestrates navigation, branding, and search results across multiple flat sites. |
| 4 | External Sharing | Site-level options determining if files can be shared outside the tenant. |
| 5 | Versioning | Built-in history recovery mechanism allowing users to roll back file changes. |

---

---
## Troubleshooting
**Scenario 1:**
- Problem: Users report that the OneDrive Sync Client is showing a red X icon and stating "You are syncing too many files" or "Sync blocked due to invalid characters".
- Root Cause: The sync client has hit limits (e.g., syncing more than 300,000 files across library attachments causes performance issues), or file names contain unsupported characters (`" * : < > ? / \ |`) or exceed the 260-character path limit.
- Fix:
  1. Click the blue cloud icon in the system tray, select `Settings` -> `Account`, and click `Stop Sync` on problematic libraries.
  2. Run a PowerShell script on the client machine to rename files with illegal characters or shorten long folder paths.
  3. Encourage users to use the **"Add Shortcut to OneDrive"** feature instead of syncing large top-level folders locally.

**Scenario 2:**
- Problem: A site owner accidentally broke permission inheritance, deleted standard groups, and now no users can access the document library except the site owners.
- Root Cause: Manual permission configuration mistakes. Inherited access was deleted, destroying access rules.
- Fix:
  1. Go to the Document Library -> `Library Settings` -> `More Library Settings`.
  2. Click on `Permissions for this document library`.
  3. On the ribbon, click **"Inherit Permissions"** to restore original settings and delete the custom permissions.
  4. If custom permissions are required, recreate the group mappings manually using `Grant Permissions`.

---

---
## Interview Questions
**Q1: How does modern SharePoint differ from classic SharePoint architecture, and why is a flat site structure recommended?**
A: Classic SharePoint relied on a single site collection containing nested subsites. This structure caused issues when a department split or moved, as files could not easily be separated, and permissions became overly complex. Modern SharePoint uses a flat structure where every site is a separate site collection, and connections are managed using Hub Sites. This allows for flexible restructuring, simpler permissions management, and prevents single-site quotas from affecting the entire tenant.

**Q2: A company wants to prevent users from downloading confidential files to personal devices while still allowing web editing. How would you solve this?**
A:
- **Situation**: The client needed to restrict download privileges for security compliance, while retaining online editing.
- **Task**: I had to deploy a policy that allowed browser-based work on files without allowing local file downloads.
- **Action**: I used SharePoint Online PowerShell to enable Conditional Access limited access policies. I ran `Set-SPTOenant -ConditionalAccessPolicy AllowLimitedAccess` globally, then configured Microsoft Entra Conditional Access policies targeting unmanaged devices to enforce browser-only access.
- **Result**: Users on non-enrolled personal devices could view and edit Excel/Word documents in Microsoft 365 Web App but found the "Download", "Print", and "Sync" options grayed out, preventing data leakage.

**Q3: Explain the difference between "Add shortcut to OneDrive" and "Sync" options in SharePoint libraries.**
A: The "Sync" option maps the library folder directly to the local Windows File Explorer using the OneDrive sync client, creating a separate sync folder path. "Add shortcut to OneDrive" inserts a pointer link inside the user's personal OneDrive directory, making the folder accessible across all devices (web, mobile, desktop) within their personal folder tree. Microsoft recommends shortcuts over Sync to avoid conflicts and reduce synchronization overhead.

---

---
## Seedha Simple Mein
*Seedha simple mein: SharePoint Online data store karne, organize karne aur design karne ki basic utility hai. Har team aur department ke liye alag site collections hoti hain jinhe common navigation, access control aur sync options ke sath manage kiya jata hai.*

---
## Related Notes
- [[04-Cloud-and-Security/07-Microsoft-365/M365-01 Microsoft 365 Administration|M365-01 Microsoft 365 Administration]] — Controls tenant identities and subscription boundaries.
- [[04-Cloud-and-Security/07-Microsoft-365/M365-03 Microsoft Teams Administration|M365-03 Microsoft Teams Administration]] — Relies on SharePoint document libraries to store channel files.
- [[04-Cloud-and-Security/07-Microsoft-365/INT-04 Configuration Profiles|INT-04 Configuration Profiles]] — Deploys auto-configuration policies for the OneDrive sync engine.
