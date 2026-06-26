---
tags: [desktop-support, m365, collaboration, L1]
aliases: [microsoft-entra-id, microsoft-entra-id]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #none
---

# Microsoft Entra ID

---

---
## Concept Overview
- **What it is**: Microsoft Entra ID (formerly Azure Active Directory) is a cloud-based identity and access management service that manages user sign-ins, credentials, devices, and access permissions for cloud applications like Microsoft 365, Microsoft Intune, and thousands of external SaaS applications.
- **Why it matters for a support engineer**: Support engineers spend significant time in Entra ID resetting user passwords, managing cloud licenses, troubleshooting MFA loops, unblocking accounts, and checking hybrid device status.
- **Where you encounter this in real job**: Offboarding and onboarding users, fixing UPN synchronization issues, checking sign-in logs to find blockages, and managing group memberships.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Resets cloud passwords, manages group memberships, updates user contact info, and registers authentication methods.
  - **L2**: Troubleshoots sync errors (using IDFix), verifies Hybrid Entra Join status on workstations, audits sign-in logs, and manages guest accounts.
  - **L3**: Configures Microsoft Entra Connect Sync servers, designs custom synchronization rules, configures Single Sign-On (SSO), and manages Entra Identity Protection risk policies.

---

---
## Technical Deep Dive
### 1. Microsoft Entra ID vs. Active Directory Domain Services (AD DS)
It is a common mistake to assume Entra ID is simply Active Directory running in the cloud. They are architecturally distinct:

| Feature | Active Directory Domain Services (AD DS) | Microsoft Entra ID |
|---|---|---|
| **Structure** | Hierarchical (Forests, Domains, OUs, Trees) | Flat structure (Tenants, Users, Groups) |
| **Access Protocol** | LDAP (Lightweight Directory Access Protocol) | Microsoft Graph API (HTTPS / REST) |
| **Authentication** | Kerberos, NTLM | modern protocols (SAML, OpenID Connect, OAuth 2.0) |
| **Device Management** | Group Policy Objects (GPO) | Mobile Device Management (MDM / Microsoft Intune) |
| **Security Boundary** | Forest level | Tenant level |

### 2. Hybrid Identity & Synchronization
For organizations transitioning from on-premises to the cloud, user accounts are synchronized from Active Directory DS to Microsoft Entra ID:
- **Microsoft Entra Connect Sync**: A sync engine installed on an on-premises Windows Server. It polls Active Directory changes (default every 30 minutes) and syncs them to Entra ID.
- **Microsoft Entra Cloud Sync**: A lightweight agent installed on-premises that offloads the sync configuration and processing to the Microsoft cloud, useful for multi-forest environments.

#### Authentication Models in Hybrid Environments:
1. **Password Hash Synchronization (PHS)**: A hash of the user's password hash is sent from AD to Entra ID. Users authenticate directly in the cloud. Best for high availability.
2. **Pass-Through Authentication (PTA)**: Users log in to Entra ID, but Entra ID passes the credentials to an on-premises agent, which validates them against the local AD. Good for enforcing on-premises security policies.
3. **Federated Authentication (AD FS)**: Users are redirected to an on-premises Active Directory Federation Services portal to sign in. The local domain handles all authentication requests.

---


## Real-World Scenarios

### Scenario 1: On-Premises Password Reset Not Syncing to Microsoft 365
**User Complaint:** "I changed my password on my office computer this morning. I can log in to my PC with the new password, but when I try to open my Outlook email on my phone or log in to office.com, it only accepts my old password."
**Your First 3 Checks:**
1. Check if the user is a hybrid user (synced from on-premises AD) or cloud-only user.
2. Check the on-premises Entra Connect sync health and the last synchronization time.
3. Check if Password Writeback is enabled if the user tried to reset via the cloud.
**Diagnosis Steps:**
1. In the M365 Admin Center, look up the user. Under "Sync status", verify it says "Synced from on-premises".
2. Remote desktop into the on-premises Entra Connect server.
3. Open **Synchronization Service Manager** (`miisclient.exe`).
4. Note that the last run of the `Delta` sync cycle failed with the error `stopped-connectivity-failure`.
5. Check network interfaces on the server; a local firewall change blocked HTTPS (Port 443) outbound traffic to Microsoft's registration endpoints.
**Root Cause:** The Entra Connect server's delta sync cycle was failing due to an outbound firewall block, preventing local password hash updates from reaching the cloud.
**Fix:**
1. Coordinate with the firewall team to allow outgoing HTTPS (Port 443) traffic to `*.msappproxy.net` and `*.microsoftonline.com`.
2. Run PowerShell on the sync server:
   `Start-ADSyncSyncCycle -PolicyType Delta`
3. Confirm in `miisclient.exe` that the sync run completed successfully with `success`.
4. Ask the user to attempt login to office.com with the new password.
**Prevention:** Configure automated monitoring alerts for Entra Connect sync health using Microsoft Entra Connect Health agent.
**Ticket Close Note:** "Firewall rule blocking port 443 outbound from sync server resolved. Triggered manual delta synchronization cycle. User confirmed M365 login is successful with new password. Closed."

### Scenario 2: New Hybrid User Fails to Sync Due to UPN Prefix Mismatch
**User Complaint:** "We hired a new developer, 'Jane Miller' (jmiller@company.com), but she does not show up in the Outlook directory and cannot sign in to her Microsoft 365 account."
**Your First 3 Checks:**
1. Check if Jane's account exists in the local Active Directory.
2. Check if the local account has a User Principal Name (UPN) suffix matching the public domain (`company.com`).
3. Check the on-premises sync server Event Viewer for synchronization errors.
**Diagnosis Steps:**
1. Open Active Directory Users and Computers. Locate `jmiller`.
2. Right-click user -> Properties -> Account tab.
   - User logon name: `jmiller@company.local` (instead of `jmiller@company.com`).
3. Because the domain suffix `@company.local` is non-routable in the cloud, Microsoft Entra Connect synchronizes the user but assigns the default tenant UPN: `jmiller@company.onmicrosoft.com`. The developer was trying to log in using `company.com` and was blocked.
**Root Cause:** The on-premises user account was created with a local, non-routable UPN suffix (`.local`) instead of the registered public suffix (`.com`).
**Fix:**
1. On the local Domain Controller, open Active Directory Users and Computers.
2. Edit Jane Miller's properties -> Account tab. Select `@company.com` from the UPN suffix dropdown.
3. Run the synchronization delta on the sync server:
   `Start-ADSyncSyncCycle -PolicyType Delta`
4. Confirm the account in Entra ID has changed from `jmiller@company.onmicrosoft.com` to `jmiller@company.com`.
5. Instruct the developer to log in using the correct public email address.
**Prevention:** Build user provisioning templates that restrict selection to routable UPN domains.
**Ticket Close Note:** "Changed the user UPN suffix in local AD from local to public company.com. Forced delta synchronization. User confirmed successful logon. Closed."

---


## Critical Points

> [!danger] Never Do This
> - Never delete a user in on-premises AD and attempt to recover them in the Entra ID recycle bin.
> - Since the source of authority is on-premises, any cloud-side recovery will be overwritten and deleted again during the next 30-minute sync cycle. Always restore the user account on-premises (using the AD Recycle Bin) and let it sync up to the cloud.

> [!warning] Common Trap
> - Creating a cloud-only user with the exact same email address as an on-premises user before syncing them.
> - This leads to synchronization conflicts (specifically duplicate UPN/SMTP errors). Entra Connect will fail to match the accounts unless SMTP matching (soft-matching) is configured correctly, resulting in duplicate user entries or sync blockages.

> [!tip] Senior Engineer Tip
> - Use the **IDFix** tool (`IdFix.exe`) provided by Microsoft prior to installing or configuring synchronization. IDFix scans your entire local Active Directory schema for syntax errors, spaces, and duplicate attributes that would cause synchronization failures in Microsoft Entra.

> [!success] Verification Steps
> - Run `dsregcmd /status` on a client laptop.
> - Look for `AzureAdJoined : YES` and `DomainJoined : YES` to verify that Hybrid Entra ID join worked correctly.

> [!question] Interview Alert
> - "What is the difference between soft-match and hard-match in Entra ID Connect?"
> - Answer: "Soft-match matches an on-premises user account to a cloud account based on the SMTP email address (`mail`) or UPN. Hard-match matches them based on a unique binary identifier called `sourceAnchor` (derived from the on-premises `objectGUID`). Hard-matching is more secure and is used when SMTP addresses might change."

---


## Common Mistakes & Fixes

| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| Manually editing user profiles in M365 Admin Center for synced users | Forgetting source of authority is local AD | Always edit synced users in the on-premises Active Directory. |
| Syncing non-routable UPN suffixes (`.local`) | Default AD configuration | Add public UPN suffixes in Domains and Trusts and apply them to AD users. |
| Using deprecated `AzureAD` PowerShell cmdlets | Not keeping up with Microsoft SDK updates | Migrate to `Microsoft.Graph` module for all PowerShell scripts. |

---


## Tags
#desktop-support #m365 #entra-id #hybrid-identity #L2 #interview-topic #lab-complete #daily-use

---
## Step-by-Step Lab
**Objective:** Inspect hybrid sync status, execute a delta sync cycle, and verify that a newly created on-premises user replicates to Microsoft Entra ID.
**Time Required:** 30 minutes
**Environment Needed:** A local Domain Controller with Entra Connect installed, and access to a Microsoft 365 Developer tenant.
**Pre-requisites:** Entra Connect Sync active between local DC and cloud tenant.

**Steps:**
1. Open PowerShell on your Domain Controller.
2. Create a test user with a routable UPN suffix:
   ```powershell
   New-ADUser -Name "Sync Test" -SamAccountName "synctest" -UserPrincipalName "synctest@company.com" -Path "CN=Users,DC=company,DC=local" -Enabled $true
   ```
3. Force a delta sync to push the user to the cloud immediately:
   ```powershell
   Import-Module ADSync
   Start-ADSyncSyncCycle -PolicyType Delta
   ```
4. Verification: Connect to Entra ID in PowerShell:
   ```powershell
   Connect-MgGraph -Scopes "User.Read.All"
   Get-MgUser -Filter "UserPrincipalName eq 'synctest@company.com'"
   ```
5. Confirm the query returns the user object and ID from Microsoft Entra ID.

**Success Criteria:** The user account created in local AD appears in the Entra ID Portal within 5 minutes.
**Common Failures:** The sync fails if the UPN suffix (`company.com`) is not registered as a custom verified domain in the M365 tenant.

---

---
## Cheat Sheet / Quick Reference
### PowerShell
Modern administration of Entra ID uses the `Microsoft.Graph` PowerShell module, which replaces the deprecated `AzureAD` and `MSOnline` modules.
```powershell
# Connect to Microsoft Entra ID tenant (declares scopes needed)
Connect-MgGraph -Scopes "User.ReadWrite.All", "Group.ReadWrite.All"

# Query a user in Entra ID by User Principal Name (UPN)
Get-MgUser -UserId "jdoe@company.com" | Select-Object Id, DisplayName, UserPrincipalName, AccountEnabled

# Force a manual delta synchronization cycle (Run on the on-premises Entra Connect server)
Import-Module ADSync
Start-ADSyncSyncCycle -PolicyType Delta

# Force a full synchronization cycle (Syncs all changes and properties)
Start-ADSyncSyncCycle -PolicyType Initial
```

### CMD / Run Box
```cmd
:: Check if the local client computer is registered in Entra ID
dsregcmd /status
```

### GUI Path
> Open browser -> Go to **admin.microsoft.com** (M365 Admin Center) -> **Microsoft Entra** (Microsoft Entra Admin Center) -> **Identity** -> **Users** -> **All Users**.

### Important Registry Paths
- Workstation Entra ID registration enrollment keys:
  ```
  HKLM\SOFTWARE\Microsoft\Enrollments
  ```
- Entra join workstation endpoint status:
  ```
  HKLM\SYSTEM\CurrentControlSet\Control\CloudDomainJoin
  ```

### Key Event IDs
On the client workstation, look in Event Viewer under:
`Applications and Services Logs` -> `Microsoft` -> `Windows` -> `User Device Registration` -> `Admin`.

| Event ID | Meaning | Troubleshooting |
|----------|---------|-----------------|
| 306 | Device was successfully registered in Entra ID | Success event; client authentication token obtained. |
| 304 | Device registration failed | Check network connectivity to cloud endpoints and verify MDM scope. |
| 1097 | Client machine failed to authenticate to Entra ID | Clean local registration using `dsregcmd /leave` and reboot. |

---

> [!info] 60-Second Summary
> **What**: Microsoft's cloud-based identity and access management directory service.
> **Why**: Critical for authenticating cloud resources (M365, SharePoint, OneDrive, Intune) and SaaS tools.
> **How**: Synchronizes local AD users to the cloud via Entra Connect Sync.
> **Command**: `Start-ADSyncSyncCycle -PolicyType Delta` / `dsregcmd /status`
> **Interview Answer Starter**: "To manage modern cloud identity, Microsoft Entra ID serves as the centralized authorization system, integrating with on-premises directories through..."

**Key Numbers to Remember:**
- Default sync cycle frequency: 30 minutes
- Outbound sync port requirement: TCP 443 (HTTPS)
- Client registration status tool: `dsregcmd /status`
- Deprecated AD modules replacement: `Microsoft.Graph` PowerShell module

**3 Things Interviewer Wants to Hear:**
- Entra ID is a flat structure, not OU-hierarchical like AD DS
- Active Directory uses LDAP/Kerberos, Entra ID uses Graph API/SAML/OAuth
- How to troubleshoot synchronization errors using IDFix

---

---
## Troubleshooting
| Problem | Cause | Fix | Command |
|---|---|---|---|
| Service connection timeout | Network firewall or routing blocking traffic | Check network route and enable target ports on firewall | `ping -c 4 <ip>` / `nc -zv <ip> <port>` |
| Access Denied error | User account lacks permissions or invalid credentials | Verify account access permissions or reset password | N/A |
| Resource not found | Object or path is misspelled or deleted | Verify spelling of target path or query active objects | N/A |

---
## Interview Questions
### Basic (L1 Level)
**Q: What is the main difference between Active Directory (on-prem) and Microsoft Entra ID (cloud)?**
A: Active Directory is designed to run locally on a company network using hierarchical domains and protocols like Kerberos to control access to computers and file shares. Microsoft Entra ID is a flat, cloud-based service that uses web-friendly protocols like SAML and OAuth to manage access to cloud software like M365, Teams, and SaaS apps.

**Q: Where do you reset a user's password if their account is synchronized from the local office AD?**
A: Because the account's source of authority is the local Active Directory, I must reset the password in the on-premises AD (e.g., via ADUC). The change will replicate to the cloud within 30 minutes.

### Intermediate (L2 Level)
**Q: How does Password Hash Synchronization (PHS) work, and how does it benefit a business?**
A: PHS works by taking the password hash on-premises, running it through a secondary encryption algorithm, and synchronizing that result to Entra ID. It benefits businesses by allowing users to use the same username and password for local and cloud systems, while acting as a recovery authentication method if the on-premises AD network goes down.

**Q: What command-line tool do you use to check if a computer is successfully registered with Microsoft Entra ID?**
A: I use the command `dsregcmd /status` in CMD. I check the output flags, specifically looking for `AzureAdJoined : YES` and `DomainJoined : YES` to confirm that hybrid registration is functioning.

### Advanced (L3/Senior Level)
**Q: A hybrid sync server is reporting synchronization errors for a single user (Error: `AttributeValueMustBeUnique`). How do you diagnose and resolve this conflict?**
A:
- **Situation**: An on-premises user failed to sync due to duplicate attribute values.
- **Task**: Identify and fix the conflicting attribute.
- **Action**: I run the **IDFix** tool to scan Active Directory for duplicate values. Alternatively, I run PowerShell `Get-ADUser -Filter * -Properties UserPrincipalName, proxyAddresses` to find which accounts contain overlapping values. Once found, I locate the conflicting proxyAddress or UPN, remove the duplicate entry from the incorrect account, and force a delta sync cycle.
- **Result**: The sync completes without errors, and the user's M365 mailbox is created successfully.

### HR / Behavioral
**Q: Tell me about a time you had to adapt to a major technological shift at work. How did you handle it?**
A: When our company decided to migrate all local mail systems to Microsoft 365, I spent my personal time building a lab with a trial tenant. I practiced setting up hybrid synchronization, resetting accounts, and managing licenses. This preparation allowed me to step up during the migration, assisting our users with minimal disruption and training L1 staff on the new admin interface.

---

---
## Seedha Simple Mein
*Seedha simple mein: Microsoft-Entra-ID ke bare mein seekhta hai. Yeh m365 infrastructure aur system settings ko properly implement karne aur support tickets ko runbooks ke help se standard templates me clear karne me help karta hai.*

---
## Related Notes
- [[03-Identity-and-Core-Services/06-Active-Directory/Users-and-Groups|Users and Groups]] — Focuses on the local source of synced AD objects.
- [[04-Cloud-and-Security/07-Microsoft-365/MFA|MFA]] — Explains multi-factor authentication policies inside Entra.
- [[04-Cloud-and-Security/07-Microsoft-365/Conditional-Access|Conditional Access]] — Covers security policies applied to Entra identities.

---
