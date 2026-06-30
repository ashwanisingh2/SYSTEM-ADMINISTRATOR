---
tags: [desktop-support, interview-prep, m365, azure, cloud, L2, L3]
aliases: [m365-azure-interview, cloud-qa]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #az-104, #ms-102
---

# Top 20 Microsoft 365 and Azure Interview Questions and Answers

---

## Concept Overview
- **What it is**: A targeted technical interview bank containing the top 20 questions regarding Microsoft 365 tenant administration and Azure cloud operations, specifically designed for L1-L3 systems engineers.
- **Why it matters**: Cloud integration is standard in modern enterprise systems. Admins must understand hybrid identities, mail routing traces, resource group scope hierarchies, and CLI automation.
- **Where you encounter this in real job**: Managing hybrid user sync, querying Azure logs, tracing lost emails, and defending subscription costs against untagged resources.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Resolves client-side OneDrive sync and MFA status, assigns basic licenses, and escalates subscription blocks.
  - **L2**: Builds mail flow rules, conducts Exchange message traces, configures Entra ID Connect sync intervals, and assigns Azure RBAC roles.
  - **L3**: Architectures tenant-to-tenant migrations, sets up enterprise landing zones, configures conditional access trees, and builds automated deployment runbooks.

---

## Technical Deep Dive

### 1. Hybrid Identity Flow (Microsoft Entra Connect)
In a hybrid enterprise environment, user accounts are mastered in local Active Directory Domain Services (AD DS) and synced to Microsoft Entra ID (formerly Azure AD) using Entra Connect.
```
[Local Active Directory] ---> (Entra Connect Sync Engine) ---> [Microsoft Entra ID]
       (On-Premises)                (Express / Custom)              (Cloud Tenant)
```
- **Sync Schedules**: The default sync engine runs a delta sync every 30 minutes.
- **Password Hash Synchronization (PHS)**: Syncs a hash of the user's password hash from on-prem AD to Entra ID, allowing users to log in with the same credentials even if the on-prem network is offline.
- **Pass-Through Authentication (PTA)**: Validates passwords directly against local Active Directory Domain Controllers using lightweight agents installed on-premises.

### 2. Exchange Online Message Tracing
When emails are reported missing, delayed, or marked as false spam, the primary tool is the **Message Trace** in the Exchange Admin Center (EAC).
- Traces verify whether the email reached the M365 gateway, was filtered by Exchange Online Protection (EOP), or was delivered to a user's Junk folder/Inbox rule.
- Event logs contain details like `Receive`, `SpamDiagnostics`, `Submit`, `Deliver`, or `Fail`.

### 3. Azure Resource Hierarchy & Scopes
Azure organizes all resources hierarchically. Permissions and policies applied at a higher level are inherited downward:
```
[Management Groups] ---> [Subscriptions] ---> [Resource Groups] ---> [Resources]
```
- **Resource Groups (RGs)**: Logical containers hosting related resources for an application or department.
- **Resource Locks**: Applied at the subscription, resource group, or resource level to prevent accidental deletion (`CanNotDelete`) or unauthorized modifications (`ReadOnly`).

---

## Commands & Syntax

### PowerShell
```powershell
# Connect to Microsoft Graph and retrieve user MFA registration status
Connect-MgGraph -Scopes "User.Read.All", "UserAuthenticationMethod.Read.All"
Get-MgUser -All | Select-Object DisplayName, UserPrincipalName, Id

# Trigger an immediate manual delta sync on the Microsoft Entra Connect server
Start-ADSyncSyncCycle -PolicyType Delta

# Connect to Azure Resource Manager and check active resource locks
Connect-AzAccount
Get-AzResourceLock -ResourceGroupName "RG-Prod-Core"
```

### Azure CLI
```bash
# Log in to Azure account and set target subscription
az login
az account set --subscription "Production-Sub-01"

# Create a resource group and apply a delete lock
az group create --name "RG-Security-Logs" --location eastus
az lock create --name "PreventDeleteLogRG" --resource-group "RG-Security-Logs" --lock-type CanNotDelete
```

### GUI Path
> Exchange Admin Center -> Mail flow -> Message trace -> Start a trace
> Microsoft Entra Portal -> Identity -> Hybrid management -> Microsoft Entra Connect -> Sync status
> Azure Portal -> Resource Groups -> Select Group -> Locks -> Add

---

## Real-World Scenarios

### Scenario 1: New Employee Missing in Entra ID (Hybrid Sync Issue)
**User Complaint**: An L1 helpdesk technician created a user account in the local Active Directory container, but the user cannot log in to Microsoft 365, and their account is missing in the Entra ID portal.
**Your First 3 Checks**:
1. Check the local AD OU location to verify it is within the synced scope of Entra Connect.
2. Check the Entra Connect Sync Service manager for active sync errors or stopped services.
3. Check the `UserPrincipalName` (UPN) attribute of the user in local AD to ensure it matches a verified domain.
**Diagnosis Steps**:
1. Open local Active Directory Users and Computers (ADUC) and confirm the user `john.doe@company.com` is in the `Synced-Users` OU.
2. Log into the Entra Connect server and check the Synchronization Service tool.
3. Locate the user UPN suffix. If it shows `john.doe@company.local`, it is unroutable and Entra ID will reject the sync or assign a default `.onmicrosoft.com` suffix.
**Root Cause**: The user was placed in the correct OU, but their UPN was configured with the local internal domain suffix (`.local`) instead of the public routable domain suffix (`.com`).
**Fix**:
1. Update the user UPN suffix in local AD to `@company.com`.
2. Open PowerShell on the Entra Connect sync server and force a manual delta synchronization cycle:
   ```powershell
   Start-ADSyncSyncCycle -PolicyType Delta
   ```
3. Confirm the user shows up in Microsoft Entra ID.
**Prevention**: Set up template validator scripts in PowerShell for the L1 provisioning desk to block account creation if the UPN suffix is not routable.

### Scenario 2: Urgent Email Missing in Mailbox
**User Complaint**: A VP claims they did not receive an urgent business contract from `vendor@partner.com` sent two hours ago. No bounce-back error was reported by the sender.
**Your First 3 Checks**:
1. Run a Message Trace for the sender address `vendor@partner.com` targeting the VP's email address.
2. Check the VP's Junk Email and Deleted Items folders.
3. Check if the user has any active Inbox rules that route external emails out of their primary inbox.
**Diagnosis Steps**:
1. Execute an Exchange Message Trace in the EAC for the past 24 hours.
2. The message trace status returns: `FilteredAsSpam` and routed to the default quarantine pool.
3. Inspect the quarantine log in the Security & Compliance portal to view the message headers.
**Root Cause**: The external partner domain's SPF record was misconfigured, causing Exchange Online Protection to mark the incoming contract email as high-confidence spam and send it to quarantine.
**Fix**:
1. Release the email from the quarantine portal to the VP's mailbox immediately.
2. Add the sender's domain to the tenant's temporary Safe Senders list in the anti-spam policies.
3. Contact the partner company's IT department to fix their SPF record.
**Prevention**: Implement weekly spam policy reviews and configure alerting notifications for high-impact sender quarantine events.

---

## Critical Points

> [!danger] Never Do This
> Do not delete an Active Directory object from local AD if it is synced to the cloud, unless you intend to delete it from both environments. Deletion on-prem synchronizes down to Entra ID and immediately purges their cloud license, mailbox contents, and OneDrive files.

> [!warning] Common Trap
> Assuming local AD schema additions sync immediately. Custom Active Directory attributes require re-running the Azure AD Connect installation wizard to refresh directory schema mappings before they show up in Entra ID.

> [!tip] Senior Engineer Tip
> When troubleshooting hybrid sync issues, always check the **ImmutableID** of the user. This anchor attribute links the on-prem user object's GUID to the cloud object. If this mismatches, you get duplicate users.

> [!success] Verification Steps
> To verify a successful hybrid sync execution:
> 1. Run `Get-ADUser -Identity "Username" -Properties ObjectGUID` on-premises.
> 2. Convert the GUID to Base64 format.
> 3. Verify it matches the `ImmutableId` output in the cloud user properties.

> [!question] Interview Alert
> "What is the difference between an Azure Active Directory (Entra ID) Security Group and an Office 365 Group?"
> - **Answer**: Security groups are used strictly for access control on files, applications, and folders. O365 Groups are collaborative units that provision a shared mailbox, calendar, SharePoint site, and Teams channel alongside membership.

---

## Common Mistakes & Fixes

| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| Storing Azure VMs in default Resource Groups | Provisioning via GUI without checking location | Enforce Azure Policies that restrict resource creation to specific tagged resource groups. |
| Deleting Azure Resource Groups with active locks | Forgetting lock inheritance | Delete or disable the resource lock first before initiating Resource Group deletions. |
| Troubleshooting mail flow using client Outlook | Confusing client-side synchronization with server routing | Perform server-side Message Traces in the Exchange Admin Center to isolate routing issues. |

---

## Lab Exercise

**Objective**: Verify the effect of an Azure Resource Lock on a Resource Group and manage it via PowerShell.
**Time Required**: 15 minutes
**Environment Needed**: An active Azure Subscription (Free Tier allowed) with PowerShell AZ module installed.

**Steps**:
1. Log into your Azure account using PowerShell:
   ```powershell
   Connect-AzAccount
   ```
2. Create a test resource group named `RG-Lab-Locks`:
   ```powershell
   New-AzResourceGroup -Name "RG-Lab-Locks" -Location "EastUS"
   ```
3. Apply a delete lock named `LabDeleteLock` on the resource group:
   ```powershell
   New-AzResourceLock -LockName "LabDeleteLock" -LockLevel "CanNotDelete" -ResourceGroupName "RG-Lab-Locks"
   ```
4. Attempt to delete the resource group using PowerShell to test the lock:
   ```powershell
   Remove-AzResourceGroup -Name "RG-Lab-Locks" -Force
   ```
   - *Expected Outcome*: An error stating the operation is blocked due to the lock `LabDeleteLock`.
5. Remove the lock to cleanup the lab:
   ```powershell
   Remove-AzResourceLock -LockName "LabDeleteLock" -ResourceGroupName "RG-Lab-Locks" -Force
   Remove-AzResourceGroup -Name "RG-Lab-Locks" -Force
   ```

**Pass Criteria**: The delete command fails when the lock is active, and succeeds only after the lock is removed.

---

## Interview Questions & Answers

### L1 Level

#### Q1: What is Microsoft Entra ID (formerly Azure AD) and how does it differ from active directory on-premises?
**A**: Local Active Directory (AD DS) runs on physical or virtual servers within a corporate network and uses protocols like Kerberos and LDAP to authenticate desktop clients. Microsoft Entra ID is a cloud-based identity and access management service that uses web-based protocols like OAuth 2.0, SAML, and OpenID Connect to authenticate cloud applications and mobile devices.

#### Q2: What is Microsoft 365 licensing? If a user has an F3 license, can they install desktop Office apps?
**A**: Microsoft 365 licensing assigns cloud service permissions to users. An **F3** (Firstline Worker) license is designed for web-only access. It does not allow users to install local desktop Office applications like Word or Excel; they must use the online web versions or mobile apps on screens under 10.1 inches.

#### Q3: A user is not getting their MFA push notifications on their Microsoft Authenticator app. How do you troubleshoot?
**A**:
1. Verify the client device has active internet connectivity (Wi-Fi or cellular).
2. Check the user's authentication registration status in the Entra portal and trigger a "Require re-register MFA" action.
3. Ensure the app has notification permissions enabled in the phone's OS settings.
4. Have the user open the app and manually pull down to check for pending approvals.

---

### L2 Level

#### Q4: Explain the differences between Exchange Hybrid classic deployment and modern agent deployment.
**A**: Classic Hybrid requires publishing local Exchange server ports (specifically port 443) directly to the internet and configuring custom SSL certificates for secure communication. Modern Hybrid uses the Microsoft Hybrid Agent, which installs on a local server and establishes an outbound connection to the cloud tenant. This bypasses the need for inbound external firewall port rules and public IP mappings.

#### Q5: A user needs temporary administrator access to modify billing properties in Azure. How do you implement this securely?
**A**: I use **Microsoft Entra Privileged Identity Management (PIM)**. Instead of assigning a permanent Billing Administrator role, I configure the user as "Eligible" for the role. The user must request activation through PIM, provide a business justification, pass MFA verification, and the elevation will automatically expire after a set time (e.g., 2 hours).

#### Q6: How do you configure a mail flow rule to block files containing specific extensions?
**A**: In the Exchange Admin Center, I create a new Mail Flow Rule. I set the condition to "Any attachment... file extension includes these words" and list restricted extensions (e.g., `.exe`, `.scr`, `.js`, `.zip`). I configure the action to "Reject the message with the explanation" and send a bounce-back alert to the sender.

#### Q7: What is the difference between a Soft-Deleted mailbox and a Hard-Deleted mailbox in M365?
**A**: A **Soft-Deleted** mailbox occurs when the user license or object is removed, but remains in the Exchange recycle bin for 30 days, allowing restoration. A **Hard-Deleted** mailbox is permanently deleted from the database and cannot be recovered unless a Litigation Hold or Retention Policy was applied before deletion.

---

### L3 Level

#### Q8: How does Microsoft Entra ID Connect handle object match conflict (soft match vs hard match)?
**A**:
- **Soft Match**: Compares the on-premise user's `mail` or `proxyAddresses` attribute with the cloud user's `UserPrincipalName`. If they match, Entra Connect binds the objects and updates the cloud anchor.
- **Hard Match**: Compares the on-premise user's `ObjectGUID` (converted to Base64) with the cloud user's `ImmutableId`. This is manually configured using PowerShell to force matching when email domains differ.

#### Q9: Explain how you would implement Conditional Access policies to enforce compliance.
**A**:
- **Situation**: The company needed to prevent data leakage from unmanaged personal devices accessing cloud files.
- **Task**: Enforce MFA and device compliance checks for all external logins.
- **Action**: I created an Entra Conditional Access policy targeting all users. Under Cloud Apps, I selected "All cloud apps". I set the conditions to include only iOS/Android/Windows OS types. Under access controls, I selected "Grant access" but required both "Require multi-factor authentication" and "Require device to be marked as compliant" via Intune.
- **Result**: Blocked all connections from unmanaged external devices, ensuring corporate M365 data stays within the secure network boundary.

#### Q10: How do you query Azure Resource Log Analytics workspaces to list failed virtual machine start operations?
**A**: I write a Kusto Query Language (KQL) query targeting the `AzureActivity` log table:
```kusto
AzureActivity
| where OperationNameValue == "Microsoft.Compute/virtualMachines/start/action"
| where ActivityStatusValue == "Failed"
| project TimeGenerated, ResourceGroup, Resource, Caller, Properties
```
This isolates the timestamp, resource details, and caller identity responsible for the startup failures.

---

### Scenario / HR Level

#### Q11: Tell me about a time you solved a critical billing escalation in Azure.
**A**:
- **Situation**: An automated test script failed to destroy environment resources, generating a $5,000 daily run charge over a weekend.
- **Task**: Isolate the rogue resources, terminate them immediately, and recover the budget from Azure Support.
- **Action**: I used Azure Cost Management logs to identify the culprit Resource Group containing high-sku GPU VM clusters. I terminated the resources, applied temporary resource locks to prevent re-creation, and configured Azure Budgets alerts at 80% threshold. I compiled logs detailing the script loop error and opened a billing support ticket with Microsoft.
- **Result**: Microsoft agreed to waive 80% of the accidental charges, and my cost-alert configuration saved the project from future overruns.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **Identity**: AD Connect matches objects via **ImmutableID** (Base64 of local `ObjectGUID`). Delta sync runs every 30 mins.
> **Mail Routing**: Use Message Trace in the Exchange Admin Center to locate lost, delayed, or quarantined mail.
> **Azure Governance**: Resource locks inherit down. `CanNotDelete` blocks deletion; `ReadOnly` blocks deletion and modifications.
> **Command**: `Start-ADSyncSyncCycle -PolicyType Delta`

**Key M365 Portals**:
- `admin.microsoft.com` (Tenant Admin)
- `admin.exchange.microsoft.com` (Exchange Admin)
- `entra.microsoft.com` (Identity & MFA)
- `portal.azure.com` (Cloud Services)

---

## Related Notes
- [[04-Cloud-and-Security/07-Microsoft-365/Microsoft-Entra-ID|Microsoft Entra ID]]
- [[04-Cloud-and-Security/07-Microsoft-365/M365-02 Exchange Online|M365-02 Exchange Online]]
- [[04-Cloud-and-Security/08-Azure/Azure-Identity|Azure Identity]]
- [[05-Automation-and-Ticketing/10-Scripting-PowerShell/PowerShell-for-M365-and-Azure|PowerShell for M365 and Azure]]

---

## Tags
#desktop-support #m365 #azure #hybrid-identity #message-trace #resource-locks #L2 #L3

