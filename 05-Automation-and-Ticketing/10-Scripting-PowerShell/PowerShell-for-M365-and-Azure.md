---
tags: [desktop-support, powershell, m365, azure, cloud-administration, L2]
aliases: [graph-powershell, az-powershell, exo-scripting]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #md-102
---

# PowerShell for M365 and Azure

---

## Concept Overview
- **What it is**: PowerShell for Microsoft 365 and Azure consists of three primary modules used to manage cloud identities and infrastructure: the **Microsoft Graph PowerShell SDK** (managing Entra ID), the **Exchange Online Management** module (managing mailboxes), and the **Az** module (managing Azure resources).
- **Why it matters for a support engineer**: Managing cloud portals via the GUI is slow and does not support bulk automation. Support engineers use cloud scripting to audit licensing costs, run mail traces, update user attributes across Entra ID, and provision virtual networks.
- **Where you encounter this in real job**: Connecting to Entra ID to reset licenses, querying mailbox folders in Exchange, starting Azure virtual machines, and checking cloud sign-in logs.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Connects to services, audits user licensing status, and adds users to Microsoft Teams/Exchange groups.
  - **L2**: Performs bulk user updates in Entra ID using CSVs, runs message traces, and provisions resources in Azure using Az cmdlets.
  - **L3**: Configures app registrations for automated PowerShell authentication (certificate-based), designs tenant-wide licensing scripts, and manages Azure automation runbooks.

---

## Technical Deep Dive

### 1. Microsoft Graph PowerShell SDK (API Scopes & Modern Authentication)
Microsoft deprecated the legacy `MSOnline` (MSOL) and `AzureAD` modules. All Entra ID administration now uses the **Microsoft Graph PowerShell SDK** (`Microsoft.Graph` module):
- **Graph API Wrapper**: The module maps PowerShell cmdlets directly to HTTP REST queries targeting the Microsoft Graph API.
- **Scopes**: Unlike older modules (which granted full tenant access on login), Graph requires you to specify **Scopes** (least privilege permissions) during connection.
  - *Example*: `Connect-MgGraph -Scopes "User.ReadWrite.All", "Group.ReadWrite.All"`
- **Authentication**: Supports interactive browser logins, MFA challenges, and non-interactive certificate-based logins for automation.

### 2. Exchange Online Management Module (EXO)
The `ExchangeOnlineManagement` module manages cloud-based messaging:
- **Remote PowerShell Sessions**: EXO cmdlets execute tasks on Microsoft's cloud servers.
- **Auto-Mapping**: EXO supports delegating permissions that map shared mailboxes to users' Outlook profiles automatically.

### 3. Azure PowerShell Module (`Az`)
The `Az` module replaces the deprecated `AzureRM` module. It manages Azure subscriptions, resource groups, storage, and VMs:
- **Contexts**: Azure commands execute within a specific **Context** (Subscription ID). You must set the context first if managing multiple subscriptions.

---

## Commands & Syntax

### PowerShell
```powershell
# 1. MICROSOFT GRAPH (ENTRA ID)
# Connect to Entra ID with targeted read/write scopes
Connect-MgGraph -Scopes "User.ReadWrite.All", "Directory.Read.All"

# Query all users in Entra ID whose department is 'Marketing'
Get-MgUser -Filter "Department eq 'Marketing'" -All | Select-Object DisplayName, UserPrincipalName, AccountEnabled

# Disable a user account in Entra ID
Update-MgUser -UserId "jdoe@company.com" -AccountEnabled:$false

# 2. EXCHANGE ONLINE
# Connect to Exchange Online using Modern Authentication
Connect-ExchangeOnline -UserPrincipalName admin@company.com

# Query mailbox storage usage details for all users
Get-Mailbox -ResultSize Unlimited | Get-MailboxStatistics |
    Select-Object DisplayName, TotalItemSize, ItemCount |
    Sort-Object TotalItemSize -Descending

# 3. AZURE AZ MODULE
# Connect to Azure account
Connect-AzAccount

# Set the active subscription context
Set-AzContext -SubscriptionId "11111111-2222-3333-4444-555555555555"

# Create a new Resource Group in East US region
New-AzResourceGroup -Name "RG-Prod-App" -Location "eastus"
```

---

## Real-World Scenarios

### Scenario 1: Auditing Unlicensed Entra ID Users to Reduce IT Costs
**User Complaint:** The billing department sends an alert: *"Our Microsoft 365 licensing bill increased. We suspect there are active employee accounts that have been terminated but still have E5 licenses assigned. We need a list of all unlicensed users to audit."*
**Your First 3 Checks:**
1. Connect to Microsoft Graph with Directory and licensing scopes.
2. Query all users and retrieve their licensing properties.
3. Filter out users who have no active licenses assigned, generating a CSV report.
**Diagnosis Steps:**
1. Connect to Microsoft Graph:
   `Connect-MgGraph -Scopes "User.Read.All", "Organization.Read.All"`
2. Fetch users and check the `AssignedLicenses` property.
3. *Construct the pipeline*:
   - By default, `AssignedLicenses` returns a collection of SKU IDs. If the collection is empty, the user is unlicensed.
**Root Cause:** Terminated or inactive accounts remain active in Entra ID with assigned licenses, wasting IT budget.
**Fix:**
Run the following script to generate the audit report:
```powershell
# Connect to Microsoft Graph
Connect-MgGraph -Scopes "User.Read.All"

# Retrieve all users and audit licensing
$Users = Get-MgUser -All -Property AssignedLicenses, AccountEnabled

$Report = foreach ($User in $Users) {
    # If AssignedLicenses is empty/null, the user is unlicensed
    if ($User.AssignedLicenses.Count -eq 0 -and $User.AccountEnabled -eq $true) {
        [PSCustomObject]@{
            DisplayName       = $User.DisplayName
            UserPrincipalName = $User.UserPrincipalName
            UserId            = $User.Id
            AccountEnabled    = $User.AccountEnabled
        }
    }
}

# Export the report
$Report | Export-Csv -Path "C:\Temp\UnlicensedActiveUsers.csv" -NoTypeInformation
Write-Host "Report exported to C:\Temp\UnlicensedActiveUsers.csv" -ForegroundColor Green
```
**Prevention:** Configure automated group-based licensing rules; removing a user from the group automatically reclaims their license.
**Ticket Close Note:** "Generated unlicensed active user report using Graph PowerShell. Identified 8 accounts. Closed."

### Scenario 2: Deploying a Virtual Machine in Azure via Scripted Automation
**User Complaint:** A developer submits a request: *"We need a test virtual machine deployed in Azure East US region named 'VM-Test-01' inside a new resource group 'RG-Lab-Test' for project validation."*
**Your First 3 Checks:**
1. Verify subscription access using `Get-AzSubscription`.
2. Plan the Resource Group name and location parameters.
3. Define the virtual machine size and networking parameters.
**Diagnosis Steps:**
1. Connect to Azure: `Connect-AzAccount`.
2. Verify you are in the correct subscription context.
3. Construct the deployment script using the `New-AzVM` cmdlet, which automates creating the Virtual Network, Subnet, Public IP, and Network Security Group in a single task.
**Root Cause:** Automated cloud environment provisioning request.
**Fix:**
Run the following PowerShell script:
```powershell
# Connect to Azure
Connect-AzAccount

# Create the Resource Group
$RGName = "RG-Lab-Test"
$Loc = "eastus"
New-AzResourceGroup -Name $RGName -Location $Loc

# Define VM Parameters
$VMName = "VM-Test-01"
$Cred = Get-Credential  # Prompts for VM local admin username/password

# Deploy VM with default networking and standard B2s sizing
New-AzVM -ResourceGroupName $RGName -Name $VMName -Location $Loc -Size "Standard_B2s" -Credential $Cred -OpenPorts 3389
```
**Prevention:** Save this template script as a standard deployment template in Azure DevOps.
**Ticket Close Note:** "Created resource group RG-Lab-Test and deployed VM-Test-01 using PowerShell Az module. Closed."

---

## Critical Points

> [!danger] Never Do This
> - Never store administrative credentials (passwords or client secrets) in plaintext inside your PowerShell script files.
> - Anyone who gains read access to the script can steal the credentials and gain full administrative access to your Microsoft 365 or Azure tenant. Use **Azure Key Vault** or **Certificate-Based Authentication** (CBA) to secure script access.

> [!warning] Common Trap
> - Assuming that `Get-ADUser` from the Active Directory module can query Microsoft Entra ID users.
> - The local Active Directory module *cannot* query cloud Entra ID accounts. You must connect using `Connect-MgGraph` and run `Get-MgUser`. They are separate directories and use different modules.

> [!tip] Senior Engineer Tip
> - When using the Microsoft Graph SDK, remember that `Get-MgUser` does not return all user properties by default to optimize API performance. To retrieve properties like `Department`, `Manager`, or `AssignedLicenses`, you must explicitly request them using the `-Property` parameter.

> [!success] Verification Steps
> - Run `Get-MgUser -UserId "user@company.com"` to verify changes in Entra ID.
> - Run `Get-AzVM -ResourceGroupName "RG-Name"` to confirm the virtual machine is running in the resource group.

> [!question] Interview Alert
> - "Why did Microsoft deprecate the AzureAD and MSOnline PowerShell modules, and what replaces them?"
> - Answer: "Microsoft deprecated the older modules because they relied on the legacy Azure AD Graph API, which is retired. They are replaced by the **Microsoft Graph PowerShell SDK** (`Microsoft.Graph`), which interfaces with the modern Microsoft Graph REST API, supporting granular scopes and token-based modern authentication."

---

## Common Mistakes & Fixes

| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| Storing raw passwords in script files | Seeking automated task run ease | Use Certificate-Based Authentication (CBA) or Managed Identities in Azure. |
| Using deprecated `Get-AzureADUser` commands | Using outdated documentation resources | Migrate scripts to use the Graph SDK cmdlet: `Get-MgUser`. |
| Queries returning incomplete user properties | Get-MgUser limits default outputs | Specify required properties explicitly using the `-Property` parameter in Graph queries. |

---

## Lab Exercise

**Objective:** Install Microsoft Graph module, connect, retrieve user listing, check active license states, connect to Azure, create a Resource Group, and clean up.
**Time Required:** 30 minutes
**Environment Needed:** A computer with internet access and a Microsoft 365 Developer/Azure Sandbox tenant.
**Pre-requisites:** PowerShell run as Administrator.

**Steps:**
1. Open PowerShell. Install the Microsoft Graph SDK module:
   ```powershell
   Install-Module Microsoft.Graph -Force
   ```
2. Connect to Microsoft Entra ID:
   ```powershell
   Connect-MgGraph -Scopes "User.Read.All"
   ```
3. Retrieve all users, listing their UPN and account status:
   ```powershell
   Get-MgUser -Top 5 | Select-Object UserPrincipalName, AccountEnabled
   ```
4. Connect to your Azure sandbox account:
   ```powershell
   Connect-AzAccount
   ```
5. Create a test Resource Group:
   ```powershell
   New-AzResourceGroup -Name "RG-Lab-PowerShell" -Location "westus"
   ```
6. Verification: List the resource groups to confirm creation:
   ```powershell
   Get-AzResourceGroup -Name "RG-Lab-PowerShell"
   ```
7. Cleanup: Delete the resource group:
   ```powershell
   Remove-AzResourceGroup -Name "RG-Lab-PowerShell" -Force
   ```

**Success Criteria:** Graph and Az connections are established, Entra users listed, Azure resource group created and successfully deleted via PowerShell.
**Common Failures:** The command fails if the user account lacks the Global Admin or subscription Contributor permissions.

---

## Interview Questions & Answers

### Basic (L1 Level)
**Q: Which PowerShell module do you use to manage Microsoft Entra ID (Azure AD) user accounts today?**
A: I use the **Microsoft Graph PowerShell SDK** (the `Microsoft.Graph` module) because the legacy `AzureAD` and `MSOnline` modules have been deprecated by Microsoft.

**Q: How do you log in to your Azure subscription via PowerShell to manage virtual machines?**
A: I open PowerShell, run the command `Connect-AzAccount`, which prompts a web browser window to authenticate with my corporate credentials, and then set the context using `Set-AzContext`.

### Intermediate (L2 Level)
**Q: What is the purpose of the '-Scopes' parameter when connecting to Microsoft Graph?**
A: The `-Scopes` parameter specifies the exact permissions the PowerShell session is requesting from the Microsoft Graph API. This follows the security principle of least privilege, ensuring the admin session only has access to the services specified (e.g. `User.ReadWrite.All` to modify users, without access to exchange billing).

**Q: A script running `Get-MgUser` does not return the user's Department or Title. How do you fix this?**
A: To optimize API performance, the Microsoft Graph SDK only returns a default set of basic properties (like DisplayName and ID). To retrieve other properties, I must request them using the `-Property` parameter:
`Get-MgUser -UserId "user@company.com" -Property Department, JobTitle | Select-Object Department, JobTitle`

### Advanced (L3/Senior Level)
**Q: You need to schedule a PowerShell script to run daily in the cloud to audit our Exchange mailboxes. It must run automatically without user interaction. How do you configure the authentication?**
A:
- **Situation**: Scheduling non-interactive daily email audits in the cloud.
- **Task**: Configure secure, passwordless authentication for the script.
- **Action**: I register an application in Microsoft Entra ID (App Registration) and assign it the required Exchange API permissions (e.g. `Mailbox.Read.All`). I generate a self-signed digital certificate and upload the public key to the App Registration. I install the private certificate on our secure execution server. In the PowerShell script, I connect using certificate thumbprint credentials:
  `Connect-ExchangeOnline -AppID [App-ID] -CertificateThumbprint [Thumbprint] -Organization [Tenant]`
- **Result**: The script authenticates securely without passwords, allowing automated runs.

### HR / Behavioral
**Q: Tell me about a time you had to learn a new programming or scripting tool to solve a problem. How did you approach it?**
A: When Microsoft deprecated the `MSOnline` module, our user provisioning scripts broke. I had to learn the Microsoft Graph SDK quickly. I studied Microsoft's migration guides, built a testing sandbox, and mapped our old commands (like `Get-MsolUser`) to their new Graph equivalents (like `Get-MgUser`). I updated all our scripts and tested them in the sandbox. This preparation allowed us to switch to the new module with zero disruption to our daily ticketing operations.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **What**: The cloud management modules: Graph (Entra ID), EXO (Exchange), and Az (Azure Resources).
> **Why**: Automates user management, mail auditing, and resource provisioning in cloud environments.
> **How**: Specify scopes on connection, request non-default properties explicitly, and manage subscription contexts.
> **Command**: `Connect-MgGraph -Scopes` / `Connect-ExchangeOnline` / `Connect-AzAccount`
> **Interview Answer Starter**: "To manage Microsoft 365 and Azure environments, I utilize Graph SDK, Exchange Online, and Az modules, ensuring sessions request explicit scopes under modern authentication..."

**Key Numbers to Remember:**
- Legacy AD API replacement: Microsoft Graph REST API
- HTTPS connection port: TCP 443
- Subscription context setter: `Set-AzContext`
- User properties query parameter: `-Property`

**3 Things Interviewer Wants to Hear:**
- Old `AzureAD`/`MSOnline` modules are deprecated; use `Microsoft.Graph` instead
- Graph requires specifying permissions (scopes) on connection (least privilege)
- Never store plaintext passwords in scripts; use certificate-based authentication instead

---

## Related Notes
- [[04-Cloud-and-Security/07-Microsoft-365/Microsoft-Entra-ID|Microsoft Entra ID]] — The directory managed via Graph cmdlets.
- [[04-Cloud-and-Security/08-Azure/Azure-Identity|Azure Identity]] — Outlines the RBAC permissions required for Az scripts.
- [[05-Automation-and-Ticketing/10-Scripting-PowerShell/PowerShell-Fundamentals|PowerShell Fundamentals]] — Detail the core pipeline and object mechanisms.

---

## Tags
#desktop-support #powershell #m365 #azure #cloud-administration #L2 #interview-topic #lab-complete #daily-use

