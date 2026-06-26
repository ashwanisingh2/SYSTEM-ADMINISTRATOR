---
tags: [desktop-support, microsoft-365, identity, ad-connect, L2]
aliases: [azure-ad-connect, entra-connect]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #az-104
---

# M365-07: Azure AD Connect (Microsoft Entra Connect)

> [!note] Overview
> This note covers the deployment, configuration, and administration of Microsoft Entra Connect (formerly Azure AD Connect). It details hybrid identity authentication methods (PHS, PTA, ADFS), directory synchronization loops, password writeback, and synchronization troubleshooting.

---
## Concept Overview
- **What it is** — Microsoft Entra Connect is an on-premises synchronization server that replicates user objects, groups, contacts, and password hashes from a local Active Directory Domain Services (AD DS) environment to Microsoft Entra ID (Azure AD) in the cloud.
- **Why it matters** — Operating separate user credentials for local office logging and cloud M365 accounts confuses users and increases administrative overhead. Entra Connect integrates both into a single hybrid identity system, allowing users to log in everywhere using the same credentials.
- **Real job encounter** — Provisioning new hybrid users, force-syncing directories after local AD edits, resolving duplicate UPN collision errors, and configuring self-service password reset (SSPR) writeback.
- **L1 vs L2 vs L3 responsibility split:**
  - **L1 Resolution**: Verify user sync status in the Microsoft 365 Admin Center, verify local AD account attributes (UPN, email), and handle basic user password resets.
  - **Escalation Trigger**: Escalate if directory synchronization fails globally, if changes fail to sync after 2+ hours, or if Entra Connect service crashes on the sync host.
  - **L2 Resolution**: Force manual delta or full directory synchronizations using PowerShell, troubleshoot sync warnings using the Synchronization Service Manager console, and resolve duplicate UPN conflicts.
  - **L3 Resolution**: Install and upgrade Entra Connect, configure advanced attribute mapping rules, deploy Password Writeback, manage high-availability Staging servers, and configure hybrid authentication models.

---
## Technical Deep Dive

### 1. Hybrid Authentication Models
Entra Connect supports three primary methods for authenticating hybrid users:
- **Password Hash Synchronization (PHS) (Most Common)**:
  - Entra Connect extracts the user's password hash from on-premises AD, hashes it 1,000 times for security, and uploads it to Entra ID.
  - Authentication happens directly in the cloud. Extremely simple to deploy and highly resilient (if on-premises servers go offline, cloud logins still work).
- **Pass-through Authentication (PTA)**:
  - Passwords are not stored in the cloud.
  - When a user logs in, Entra ID sends the password to an on-premises PTA connector agent via a secure queue. The agent validates it against local AD and returns the result.
  - Best for organizations needing local security policy enforcement (like login hours) instantly.
- **Federation (ADFS)**:
  - Authentication is redirected to on-premises Active Directory Federation Services (ADFS) servers.
  - Offers complex controls (like smart card logins) but introduces significant local server infrastructure overhead.

```
       [ Client Logs in to M365 ]
                   |
         +---------+---------+
         |                   |
         v                   v
   [ PHS Model ]       [ PTA Model ]
         |                   |
   Authenticated       Forwarded to 
   directly in the     On-Premises PTA
   cloud using synced   Connector agent.
   password hashes.    Local AD validates.
```

### 2. The Synchronization Loop
Entra Connect processes objects through three virtual storage zones:
1. **Connector Space (CS) (On-Premises)**: The mirror database of local Active Directory.
2. **Metaverse (MV)**: The central correlation database. It takes objects from the local CS, applies join/projection rules, and combines duplicates.
3. **Connector Space (CS) (Cloud)**: The mirror database of Microsoft Entra ID.
- **Sync Stages**: **Import** (loads data to CS) -> **Synchronization** (processes changes in MV) -> **Export** (writes changes to target directory).
- **Sync Schedule**: Runs a delta sync every **30 minutes** by default.

### 3. Password Writeback
- Essential for **Self-Service Password Reset (SSPR)**.
- When a user resets their password using the M365 cloud portal, Entra ID encrypts the new password and sends it via TLS to the on-premises Entra Connect agent, which immediately updates the local Active Directory domain controller.

---
## Step-by-Step Lab

> [!warning] Pre-requisites
> - An on-premises Domain Controller (`DC-01.corp.local`).
> - A dedicated member server (`SVR-ADSYNC01`) running Windows Server 2022.
> - An active Microsoft 365 Tenant with a custom domain verified (e.g., `company.com`).
> - Local Enterprise Admin and Cloud Global Administrator credentials.

### Step 1: Add Public UPN Suffix to On-Premises Active Directory
Local domains often use internal names (e.g., `corp.local`). Microsoft Entra ID cannot verify or route internal suffixes. We must add the public domain suffix.

1. Log in to `DC-01`. Open **Active Directory Domains and Trusts**.
2. Right-click the root node -> **Properties**.
3. In **Alternative UPN Suffixes**, add your verified public domain: `company.com`. Click **Add** -> **Apply**.
4. Open **Active Directory Users and Computers** (`dsa.msc`).
5. Open properties for user `jdoe`. Set their UPN suffix to `jdoe@company.com`.

### Step 2: Install Microsoft Entra Connect
1. Log in to `SVR-ADSYNC01`. Download the Microsoft Entra Connect installer.
2. Run `AzureADConnect.msi` as Administrator.
3. Agree to license terms and click **Customize**.
4. Connect to Microsoft Entra ID: Enter your Cloud Global Admin credentials.
5. Connect your Directories:
   - Directory Type: **Active Directory**.
   - Forest: Select `corp.local`.
   - Click **Add Directory** and input local Enterprise Admin credentials.
6. Azure AD Sign-In Configuration: Select `userPrincipalName` as the mapping attribute. Ensure your verified domain `company.com` shows status **Verified**.
7. Domain and OU Filtering: Select **Sync selected domains and OUs** and check only target production OUs (e.g. `OU=Users,OU=Prod`).
8. Optional Features: Check **Password Hash Synchronization** and **Password writeback**.
9. Click **Install**. Clear the box "Start the synchronization process when configuration completes" if you want to verify first.

### Step 3: Trigger Manual Directory Synchronization
To force updates immediately without waiting for the 30-minute interval, run a manual sync cycle.

1. On `SVR-ADSYNC01`, open PowerShell as Administrator.
2. Trigger a **Delta Sync** (replicates only new modifications):
```powershell
# Run delta sync cycle
Import-Module ADSync
Start-ADSyncSyncCycle -PolicyType Delta
```
**Expected Output:**
```text
Result
------
Success
```

3. To force a **Full Sync** (scans all objects, required when adding new OUs or changing attributes mapping):
```powershell
Start-ADSyncSyncCycle -PolicyType Initial
```

### Step 4: Verify Synchronization Logs
1. Open the **Synchronization Service** console from the Start Menu on `SVR-ADSYNC01`.
2. Inspect the **Operations** tab.
3. Verify that you see tasks: `Active Directory Import`, `Metaverse Sync`, and `Windows Azure Active Directory Export` displaying status `success`.
4. Log in to `admin.microsoft.com`. Navigate to **Users** -> **Active Users**. Verify that user accounts display sync status: **Synced from on-premises**.

---
## Cheat Sheet / Quick Reference

| Command / Setting | Purpose | Example / Details |
|---|---|---|
| `Start-ADSyncSyncCycle -PolicyType Delta` | Forces immediate sync of changed objects | PowerShell command |
| `Start-ADSyncSyncCycle -PolicyType Initial` | Forces a complete full sync of all directory objects | PowerShell command |
| `Get-ADSyncScheduler` | Displays sync schedules, intervals, and active status | `Get-ADSyncScheduler` |
| `Set-ADSyncScheduler` | Modifies synchronization engine intervals | `Set-ADSyncScheduler -CustomInterval 01:00:00` |
| `Get-ADSyncConnectorRunStatus` | Queries current running state of the sync engine | `Get-ADSyncConnectorRunStatus` |
| `IdFix.exe` | Microsoft directory parsing tool checking for invalid characters | Pre-migration cleanup utility |
| **Default Sync Interval** | Time window between automated synchronization cycles | `30 Minutes` |
| **Sync Service Manager** | Graphical interface console for tracking runs | `miisclient.exe` |

---
## Troubleshooting Table

| Problem | Cause | Fix |
|---|---|---|
| User account does not sync to cloud, Synchronization Service reports `AttributeValueMustBeUnique`. | Two local AD accounts are sharing the same User Principal Name (UPN) or ProxyAddresses (SMTP) email address. | Open the Synchronization Service console. Identify the conflicting attribute. Open ADUC and update the duplicate attribute to ensure uniqueness. |
| User password updates do not sync to Entra ID, but other attribute changes sync fine. | The ADSync service account lacks "Replicating Directory Changes" permissions in local AD. | Open Active Directory Users and Computers. Right-click domain root -> Properties -> Security. Ensure the ADSync user account has **Replicating Directory Changes** checked. |
| User resets password in Office 365 portal, but password fails to write back to local AD. | Password Writeback is disabled in Entra Connect, or the agent lacks permissions on local user objects. | Re-run the Entra Connect installer on the sync server. Go to Optional Features and check **Password Writeback**. Verify agent account has write permissions on user objects. |
| Client sync fails with error: `InvalidSoftMatch`. | An on-premises user principal name matches a cloud-only user account domain, but the security anchors do not match. | Match the local user's UPN/Email with the cloud user. In PowerShell, set the cloud user's `ImmutableID` to null to allow matching or delete the cloud user and let the sync create it. |
| Synchronization Service console shows status: `stopped-extension-dll-exception`. | Database access issues or service account credential expiration. | Check Windows Event Logs. Verify that the SQL database hosting ADSync is running, and that the ADSync service account password has not expired. |

---
## Interview Questions

> [!question] L1 Question
> **Q:** How do you force a directory sync on an Azure AD Connect server using PowerShell?
> **A:** I will log in to the synchronization server, open PowerShell as Administrator, import the ADSync module, and run the command: **`Start-ADSyncSyncCycle -PolicyType Delta`**. This triggers an immediate sync of changes made since the last run.

> [!question] L2 Question
> **Q:** What is the purpose of running the `IdFix` tool before deploying Azure AD Connect?
> **A:** `IdFix` is a Microsoft utility used to scan the local Active Directory database prior to synchronization. It identifies validation errors in object attributes, such as invalid characters (like slashes or spaces), duplicate UPN suffixes, or invalid SMTP formatting, which would block synchronization.

> [!question] L3/Scenario Question
> **Q:** You are preparing to upgrade the hardware of your Azure AD Connect server. How do you transition the sync engine to a new member server without causing duplicate objects or directory sync downtime?
> **A:** 
> - **Situation:** Upgrading Azure AD Connect server hardware without causing synchronization downtime.
> - **Task:** Install a redundant sync instance and execute a safe transition.
> - **Action:** 
>   1. **Staging Mode**: Install Azure AD Connect on the new server (`SVR-ADSYNC02`). During configuration, select the option to enable **Staging Mode**.
>   2. **Verify Staging**: In staging mode, the new server imports changes and processes policies in its database, but does *not* export or write any changes to either local AD or Entra ID. I will verify that the operations run successfully in the console.
>   3. **Transition**: Disable sync on the old server: `Set-ADSyncScheduler -SyncCycleEnabled $false`.
>   4. **Activate New Server**: Re-run the configuration wizard on the new server (`SVR-ADSYNC02`) and disable Staging Mode.
>   5. **Decommission**: Uninstall Azure AD Connect from the old hardware once the new server completes its first export run.
> - **Result:** The transition occurs with zero disruption, and staging mode prevents configuration collisions.

---
## Seedha Simple Mein
*Seedha simple mein: Azure AD Connect ek aisi technology hai jo office ke local servers (Active Directory) se users aur passwords ko Microsoft 365 cloud (Entra ID) ke sath sync karti hai. Isse users ko ek hi password se cloud aur local office systems dono access karne ko milte hain. Agar koi user sync nahi ho raha, toh `miisclient.exe` console me status check karte hain.*

---
## Related Notes
- [[03-Identity-and-Core-Services/06-Active-Directory/WS-02 Active Directory Domain Services|WS-02 Active Directory Domain Services]] — The on-premises source identity database.
- [[04-Cloud-and-Security/07-Microsoft-365/M365-01 Microsoft 365 Administration|M365-01 Microsoft 365 Administration]] — Cloud identity portal operations.
- [[04-Cloud-and-Security/07-Microsoft-365/SSPR|SSPR]] — Explains Self-Service Password Reset dependent on Writeback.

---
*Tags: #desktop-support #microsoft-365 #identity #ad-connect #L2 #cert-az-104*
