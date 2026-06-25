---
tags: [desktop-support, m365, licensing, administration, L2]
aliases: [licensing-guide, group-licensing, office-activation]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #md-102
---

# Microsoft 365 Licensing

---

## Concept Overview
- **What it is**: Microsoft 365 Licensing is the framework used to assign access permissions for cloud services and local software (like Outlook, Word, Teams, and SharePoint) to corporate user accounts. Licenses are managed via subscriptions (SKUs) purchased by the organization.
- **Why it matters for a support engineer**: License errors block user productivity immediately. Support engineers provision accounts, assign licensing tiers, resolve "Unlicensed Product" activation flags on desktops, and audit license availability to control IT costs.
- **Where you encounter this in real job**: Troubleshooting Office activation failures, converting licenses during employee promotions, configuring Group-Based licensing automated flows, and reclaiming licenses from offboarded accounts.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Assigns licenses to individual users in the M365 Admin Center, resets activation tokens on laptops, and checks license status.
  - **L2**: Troubleshoots Group-Based licensing synchronization errors, resolves overlapping SKU assignment conflicts, and configures Shared Computer Activation.
  - **L3**: Negotiates Microsoft Enterprise Agreements, audits global SKU consumption, configures Tenant-to-Tenant licensing migrations, and manages automated API license assignment scripts.

---

## Technical Deep Dive

### 1. Microsoft 365 SKU Tiers & Feature Matrix
Support engineers must understand the differences between the core business and enterprise SKUs:

| SKU Name | Target Audience | Key Features Included | Intune / Entra P1 Included |
|---|---|---|---|
| **M365 Business Basic** | Frontline/Web-only | Web apps only, Exchange Email, Teams, SharePoint | No |
| **M365 Business Standard** | Small/Mid Business | Desktop Office apps, Teams, Exchange, OneDrive | No |
| **M365 Business Premium** | SMB (Security Focus) | Desktop Office apps, Teams, Exchange, OneDrive, **Intune, Defender for Business, Entra ID P1** | **Yes** |
| **M365 Enterprise E3** | Large Enterprise | Advanced Office apps, Intune, Entra ID P1, basic compliance tools | **Yes** |
| **M365 Enterprise E5** | Enterprise (High Sec) | Everything in E3 + **Power BI Pro, Teams Phone System, Entra ID P2, Defender for Endpoint** | **Yes** |
| **M365 F3** | Frontline Workers | Web-only, lightweight mailboxes, basic security | Yes (Entra ID P1 + Intune) |

### 2. Group-Based Licensing
Manually assigning licenses is inefficient and error-prone. Enterprise environments use **Group-Based Licensing**:
1. Create a Security Group in Microsoft Entra ID (e.g., `G-License-E3-Users`).
2. Assign the Microsoft 365 E3 license SKU directly to the Group.
3. Add users to the group. Microsoft Entra ID automatically allocates the license to all group members.
4. If a user is removed from the group, Entra ID automatically reclaims the license.
- *Tip*: You can use **Dynamic Groups** (which add users automatically based on attributes like Department: `Sales`) to fully automate user licensing based on their job role.

### 3. Shared Computer Activation (SCA)
By default, a standard Office license allows a user to install Office on up to 5 personal devices. However, in shared environments (such as remote desktop RDS servers, or shared hospital terminals):
- **SCA Mode**: You install Office with SCA enabled. Instead of licensing the machine or deducting from the user's 5-device quota, Office checks the user's M365 identity on launch.
- **Token**: It downloads a temporary licensing token to the user's AppData profile. When the user logs out, the token is ignored, freeing up the license.

---

## Commands & Syntax

### PowerShell
Licensing management uses the `Microsoft.Graph.Identity.DirectoryManagement` and `Microsoft.Graph.Users` modules.
```powershell
# Connect to Microsoft Graph with License Administration scopes
Connect-MgGraph -Scopes "Directory.Read.All", "User.ReadWrite.All", "Organization.Read.All"

# Query all available License SKUs and active counts in the tenant
Get-MgSubscribedSku | Select-Object SkuId, SkuPartNumber, ActiveUnits, ConsumedUnits

# Assign an E3 License (e.g., SkuId: 6fd2c87f-b296-42f0-b197-1e91999b5316) to a user, and remove no licenses
Set-MgUserLicense -UserId "jdoe@company.com" -AddLicenses @{SkuId = "6fd2c87f-b296-42f0-b197-1e91999b5316"} -RemoveLicenses @()

# Check which licenses are directly assigned to a user
(Get-MgUser -UserId "jdoe@company.com" -Property LicenseAssignmentStates).LicenseAssignmentStates
```

### CMD / Run Box (Office Activation Troubleshooting)
The `ospp.vbs` script manages local Office activation licenses on the computer.
```cmd
:: Navigate to the Office installation directory (x64 path)
cd "C:\Program Files\Microsoft Office\Office16"

:: Display the license status and the last 5 characters of active product keys
cscript ospp.vbs /dstatus

:: Uninstall a corrupted Office product key (Use the 5 characters returned by /dstatus)
cscript ospp.vbs /unpkey:VMFT9

:: Force Office to check the Microsoft activation servers immediately
cscript ospp.vbs /act
```

### GUI Path
- **Admin Portal**: Go to **admin.microsoft.com** -> **Billing** -> **Licenses**.
- **Group Licensing**: Go to **entra.microsoft.com** -> **Billing** -> **Licenses** -> **All products** -> Select Product -> **Licensed groups**.

### Important Registry Paths
- Enabling Shared Computer Activation for Office Click-to-Run:
  ```
  HKLM\SOFTWARE\Microsoft\Office\ClickToRun\Configuration
  (Create String Value "SharedComputerLicensing" set to "1")
  ```

---

## Real-World Scenarios

### Scenario 1: Office Shows "Unlicensed Product" Error on Shared Workstation
**User Complaint:** A nurse logs into a shared terminal in the clinic, opens Microsoft Word, and is blocked by a red banner stating: *"Unlicensed Product. Most features have been disabled."* They cannot edit their shift reports.
**Your First 3 Checks:**
1. Check if the user's account has an active Office license assigned (e.g., M365 Business Premium or E3).
2. Check if the computer has Shared Computer Activation (SCA) enabled.
3. Check for corrupted cached activation tokens in the user's profile.
**Diagnosis Steps:**
1. Log in to the M365 portal and confirm the nurse has an active E3 license.
2. Open Command Prompt on the client PC as Administrator and run the registry query:
   `reg query HKLM\SOFTWARE\Microsoft\Office\ClickToRun\Configuration /v SharedComputerLicensing`
   - Output: `SharedComputerLicensing REG_SZ 0` (or key not found).
3. *The client PC is running in standard user mode.* Because multiple nurses log in to this terminal, the device limit of 5 activations was breached, blocking new users.
**Root Cause:** The shared terminal was configured in standard device licensing mode instead of Shared Computer Activation (SCA) mode.
**Fix:**
1. Open Registry Editor on the shared PC. Go to:
   `HKLM\SOFTWARE\Microsoft\Office\ClickToRun\Configuration`
2. Add or modify the string value: `SharedComputerLicensing` and set it to `1`.
3. Clear the user's old activation token folders:
   Delete `%localappdata%\Microsoft\Office\16.0\Licensing`.
4. Relaunch Word. The application prompts the user to sign in, contacts Microsoft servers, downloads a temporary SCA token, and activates Word successfully.
**Prevention:** Deploy Office to shared terminals using the Office Deployment Tool (ODT) with `SharedComputerLicensing` enabled in the `configuration.xml` template.
**Ticket Close Note:** "Enabled Shared Computer Licensing in registry. Cleared cached licensing tokens. Confirmed user could activate Office on the shared terminal. Closed."

### Scenario 2: User Loses Access to Services Due to License Conflict
**User Complaint:** A newly promoted financial analyst reports: *"I was assigned access to Power BI Pro today, but now my Outlook email has stopped receiving messages, and it says my mailbox is almost full."*
**Your First 3 Checks:**
1. Verify the active licenses assigned to the analyst in the M365 Admin Center.
2. Check for "License Assignment Errors" in the Entra ID Portal.
3. Check the mailbox size and type inside the Exchange Admin Center.
**Diagnosis Steps:**
1. Open the user properties in Microsoft Entra Admin Center.
2. Click **Licenses**.
   - Notice a red warning: `License assignment failed: Conflict detected.`
   - Assigned Licenses: `M365 Business Premium` (assigned via Group) and `Office 365 E1` (assigned manually by support today to give them a service plan).
3. The analyst was manually assigned an O365 E1 license, which conflicts with their existing Business Premium SKU.
4. During the conflict state, the user's Exchange license features were paused, downgrading their mailbox to the basic 2GB limit instead of the 50GB limit.
**Root Cause:** A manual licensing assignment created a conflict with group-based licenses, suspending the user's advanced mailbox features.
**Fix:**
1. In the Licenses tab, remove the manually assigned `Office 365 E1` license.
2. Ensure the user remains a member of the group provisioning `Business Premium`.
3. Add the Power BI Pro license independently.
4. Save changes and click **Reprocess** to re-evaluate the licensing state.
5. In Exchange Admin Center, verify the user's mailbox capacity restores to 50GB.
**Prevention:** Avoid assigning conflicting base licenses manually; use Group-Based licensing to manage all profile changes.
**Ticket Close Note:** "Removed conflicting E1 license. Reprocessed licensing state. Confirmed mailbox limit restored to 50GB and Power BI Pro is active. Closed."

---

## Critical Points

> [!danger] Never Do This
> - Never delete a user's license immediately after termination if you need to run an email archive migration or set up mail forwarding.
> - Removing the license immediately deletes the user's mailbox and OneDrive data after a 30-day grace period. Always convert the mailbox to a shared mailbox first, delegate permissions, and then safely strip the license.

> [!warning] Common Trap
> - Assuming that assigning a license instantly activates all features for a user.
> - While license assignment is fast, it can take up to 24 hours for services like SharePoint or Exchange to fully provision the user database partitions in the backend. Warn users of potential 15-minute to 1-hour propagation delays.

> [!tip] Senior Engineer Tip
> - When troubleshooting persistent Office activation errors where clearing the AppData folder fails, use the official Microsoft script **Sign-in troubleshooting tool** or manually run `ospp.vbs /unpkey` to wipe all registration keys. This completely resets Office's security interface, forcing it to prompt for fresh credentials.

> [!success] Verification Steps
> - Run `cscript ospp.vbs /dstatus` on a client PC to check that the product is licensed.
> - Look for `LICENSE STATUS: --- LICENSED ---` in the command output.

> [!question] Interview Alert
> - "What happens to a user's data when you remove their Microsoft 365 license?"
> - Answer: "When a license is removed, the user's account enters a 30-day grace period. During these 30 days, all data (emails in Exchange, files in OneDrive) remains accessible and can be recovered by re-assigning the license. After 30 days, the data is permanently deleted from primary directories, though SharePoint and Exchange retention policies may preserve it in hidden archives."

---

## Common Mistakes & Fixes

| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| Manually assigning individual licenses in large organizations | Lack of knowledge of Group-Based licensing | Configure Group-Based licensing using Microsoft Entra Security Groups. |
| Using standard licensing on virtual RDS servers | Users exceed the 5-device activation limit | Enable Shared Computer Activation (SCA) in the Office deployment XML settings. |
| Ignoring license conflict warnings in Entra Portal | Overlooking sync alerts | Monitor the billing and licensing alert dashboard to resolve conflict flags. |

---

## Lab Exercise

**Objective:** Query available licensing SKUs via PowerShell, configure a group-based license rule, and verify the licensing state on a test user.
**Time Required:** 30 minutes
**Environment Needed:** A computer with internet access and a Microsoft 365 test tenant.
**Pre-requisites:** The Microsoft Graph PowerShell modules installed.

**Steps:**
1. Open PowerShell and connect to your M365 tenant:
   ```powershell
   Connect-MgGraph -Scopes "User.ReadWrite.All", "Directory.ReadWrite.All", "Organization.Read.All"
   ```
2. Retrieve the active SKU list and copy a SkuId (e.g. for M365 Business Premium):
   ```powershell
   Get-MgSubscribedSku | Select-Object SkuId, SkuPartNumber, ActiveUnits
   ```
3. Create a test security group:
   ```powershell
   New-MgGroup -DisplayName "G-Lic-BusinessPremium" -MailEnabled:$false -SecurityEnabled:$true -MailNickname "bp_lic"
   ```
4. Navigate to the Entra ID Portal (`entra.microsoft.com`). Go to **Licenses** -> **All Products** -> Select the product -> **Groups** -> Assign the license to the newly created `G-Lic-BusinessPremium` group.
5. Create a test user:
   ```powershell
   New-MgUser -DisplayName "Lic Test" -PasswordProfile @{Password="Pass1234!"; ForceChangePasswordNextSignIn=$true} -AccountEnabled:$true -MailNickname "lictest" -UserPrincipalName "lictest@yourdomain.onmicrosoft.com"
   ```
6. Add the test user to the security group:
   - Run in PowerShell or add in the GUI.
7. Verification: Run the licensing check:
   ```powershell
   (Get-MgUser -UserId "lictest@yourdomain.onmicrosoft.com" -Property LicenseAssignmentStates).LicenseAssignmentStates
   ```
   - Verify that the SkuId matches, and the assignment type shows "Inherited".

**Success Criteria:** The user is added to the group and receives the assigned license automatically, verified via PowerShell.
**Common Failures:** The sync can take up to 5-10 minutes. If the check returns empty immediately, wait and retry.

---

## Interview Questions & Answers

### Basic (L1 Level)
**Q: What is Microsoft 365 Shared Computer Activation (SCA) and why is it used?**
A: Shared Computer Activation is a licensing setting used when deploying Office to shared devices, such as hospital terminals or remote desktop servers. It allows multiple users to log into the same computer and use Office without subtracting from their standard 5-device personal license limit.

**Q: A user's Office client says "Unlicensed Product." What are the first three checks you perform?**
A: First, I verify the user has an active license containing Office desktop apps in the M365 Admin Center. Second, I check if the computer has an active internet connection to contact activation servers. Third, I check the system date and time, as time drifts block SSL activation traffic.

### Intermediate (L2 Level)
**Q: How does Group-Based licensing work, and what happens when a user is removed from the licensing group?**
A: Group-Based licensing allows admins to link M365 licenses to a security group. When a user joins the group, Entra ID assigns them the licenses. When the user is removed from the group, Entra ID automatically revokes the license. Any user data (e.g., mailboxes) enters a 30-day grace period, during which it can be recovered if the license is restored.

**Q: Which command line utility do you use to troubleshoot Office activation licenses, and how do you display the current activation state?**
A: I use the VBScript script named `ospp.vbs` located in the Microsoft Office installation folder. I open Command Prompt as Administrator, navigate to the folder, and run:
`cscript ospp.vbs /dstatus`
This displays the current license status and the last 5 characters of any active product keys.

### Advanced (L3/Senior Level)
**Q: Explain how you would manage the licensing and data retention workflow during a company merger where 500 users are migrating to a new Microsoft 365 tenant.**
A:
- **Situation**: Migrating 500 users and their licenses to a consolidated tenant during a company merger.
- **Task**: Prevent service disruption, ensure correct SKU assignment, and manage data preservation.
- **Action**: First, I catalog the source tenant's active SKUs and match them to available licensing packages in the target tenant. I configure Group-Based licensing in the target tenant using dynamic groups targeted by department tags. To prevent data loss, I ensure the source licenses remain active during the data migration phase (using tools like BitTitan). Once data sync is complete, I block access to the source tenant, assign target licenses to the users, and wait 30 days before letting the source licenses expire.
- **Result**: The users migrated to the new tenant with identical license feature sets, and zero data was lost.

### HR / Behavioral
**Q: Tell me about a time you identified an inefficiency in your team's workflow and automated it.**
A: Our helpdesk spent hours manually assigning licenses to new hires, often missing specific add-ons like Teams Phone licenses. I proposed and built a Group-Based licensing workflow in Microsoft Entra ID. I created security groups for our standard user roles and linked the matching licenses. Now, when HR creates an account and assigns them to their department group, the correct licenses are assigned automatically. This eliminated manual ticketing errors and saved the team 3-4 hours of work per week.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **What**: The mechanism for managing access permissions for M365 services and desktop Office software.
> **Why**: Critical for billing control, compliance, and user access. Managed via SKUs.
> **How**: Assign licenses to Entra Groups to automate allocation, and use `ospp.vbs` to fix client activation issues.
> **Command**: `cscript ospp.vbs /dstatus` / `Get-MgSubscribedSku`
> **Interview Answer Starter**: "To manage enterprise licensing efficiently, I utilize Group-Based licensing in Microsoft Entra ID to align license SKUs with user departments, while using ospp.vbs..."

**Key Numbers to Remember:**
- Days in license removal grace period: 30 days
- Max personal device activations per user license: 5 devices
- Registry key value for Shared Computer Activation: `1`
- PowerShell module for licensing: `Microsoft.Graph.Users`

**3 Things Interviewer Wants to Hear:**
- Group-Based licensing prevents manual assignment mistakes
- Shared Computer Activation (SCA) for RDS/shared environments
- How removing a license initiates the 30-day deletion grace period

---

## Related Notes
- [[04-Cloud-and-Security/07-Microsoft-365/Microsoft-Entra-ID|Microsoft Entra ID]] — The directory where groups and users are licensed.
- [[04-Cloud-and-Security/07-Microsoft-365/Exchange-Online|Exchange Online]] — Relies on licenses to provision mailbox quotas.
- [[06-Career-Growth/14-Certifications/MD-102|MD-102 Roadmap]] — Covers licensing objectives for modern endpoints.

---

## Tags
#desktop-support #m365 #licensing #administration #L2 #interview-topic #lab-complete #daily-use

