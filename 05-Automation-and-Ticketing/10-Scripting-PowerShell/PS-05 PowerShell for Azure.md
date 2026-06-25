---
tags: [automation, powershell, azure, cloud, az-104]
aliases: [powershell-for-azure]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #az-104
---

# PS-05 PowerShell for Azure

> [!abstract] Overview
> PowerShell is a primary automation tool for Microsoft Azure cloud resources. By installing the `Az` PowerShell module, administrators can manage resources, automate deployments, and query environments directly via the Azure Resource Manager (ARM) API. This note covers the Az module lifecycle, cloud commands, Automation Runbooks, and Graph API concepts.

---

## Concept Overview
- **What it is** — The `Az` module is a collection of PowerShell cmdlets designed to interact with the Azure control plane to provision, manage, and configure resources.
- **Why it matters for a support engineer** — Clicking through the Azure Portal to manage virtual machines, storage accounts, or virtual networks is time-consuming. PowerShell scripting facilitates fast, reproducible administrative commands.
- **Where you encounter this in real job** — Automating VM shutdowns to optimize costs, running bulk configurations on user groups, and writing monitoring scripts for security auditing.
- **L1 vs L2 vs L3 responsibility split for this topic:**
  - ****L1 Resolution:**** Logs in to Azure via PowerShell (`Connect-AzAccount`), checks VM statuses, and starts/stops VM instances.
  - **Escalation Trigger:** Escalate to L2 if resource provisioning scripts fail with subscription authorization, API quota limits, or network path errors.
  - ****L2 Resolution:**** Installs and updates `Az` modules, writes resource provisioning scripts (for VMs, Storage, VNETs), and configures basic Azure AD resources.
  - ****L3 Resolution:**** Implements Azure Automation Runbooks, configure secure authentication using Managed Identities, and manages cloud identities via Graph API calls.

*Seedha simple mein: PowerShell for Azure (Az Module) ka use karke hum command-line se pure Azure cloud infrastructure ko control aur automate kar sakte hain. Iska use karke server creation, monitoring, aur directory services automate ho jati hain.*

---

## Technical Deep Dive

### 1. Azure PowerShell Architecture: The `Az` Module
The old module (`AzureRM`) is deprecated. The modern **`Az`** module is cross-platform (runs on Windows PowerShell, PowerShell Core, Linux, and macOS).
* **Connection Lifecycle**:
  * `Connect-AzAccount`: Interactive login via browser.
  * `Set-AzContext`: Selects the target subscription if multiple exist.
* **Module Grouping**: Commands are separated logically:
  * `Az.Compute` -> Virtual Machines, VM Scale Sets, Disks.
  * `Az.Storage` -> Storage Accounts, Blobs, File Shares.
  * `Az.Network` -> VNETs, Subnets, NSGs, Public IPs.

### 2. Azure Automation & Runbooks
Azure Automation Accounts execute PowerShell scripts directly in the cloud as **Runbooks**.
* **Managed Identities**: Runbooks run under a Managed Identity (system-assigned or user-assigned), which grants Azure permissions to the script without hardcoding passwords or secret client keys.

### 3. Microsoft Graph API via PowerShell
Azure Active Directory (Microsoft Entra ID) resources are accessed using the **Microsoft Graph API**. The `Microsoft.Graph` module enables managing users, groups, and directory roles directly via API endpoints.

---

## Step-by-Step Lab / Configuration

> [!warning] Pre-requisites
> - A client machine with internet connection.
> - An active Azure account and Subscription.
> - PowerShell 5.1 or PowerShell Core installed.

### Step 1: Install Az Module and Connect to Azure
We will install the required module from the PowerShell Gallery and authenticate.

1. Open PowerShell as Administrator.
2. Install the `Az` module:
```powershell
Install-Module -Name Az -AllowClobber -Force -Scope CurrentUser
```
3. Connect to your Azure cloud account:
```powershell
Connect-AzAccount
```
**Expected Output:** A web browser pops up asking for Microsoft credentials. On success, console lists your logged-in username, TenantId, and Subscription details.

---

### Step 2: Create a Resource Group and Virtual Network
We will write a script to build our base networking resources.

1. Define variables:
```powershell
$rgName = "RG-AzureAutomation"
$location = "EastUS"
$vnetName = "VNET-Prod-01"
```
2. Create Resource Group:
```powershell
New-AzResourceGroup -Name $rgName -Location $location
```
3. Define subnet configuration and create VNET:
```powershell
$subnet = New-AzVirtualNetworkSubnetConfig -Name "Subnet-01" -AddressPrefix "10.0.1.0/24"
New-AzVirtualNetwork -Name $vnetName -ResourceGroupName $rgName -Location $location -AddressPrefix "10.0.0.0/16" -Subnet $subnet
```
**Expected Output:** Console displays VNET parameters, showing provisioning status `Succeeded`.

---

### Step 3: Deploy an Azure VM via PowerShell
We will provision a simple Windows Server VM in our new VNET.

1. Execute the creation command (requires entering username/password for VM):
```powershell
$cred = Get-Credential
New-AzVM -ResourceGroupName "RG-AzureAutomation" -Name "VM-Web-01" -Location "EastUS" -VirtualNetworkName "VNET-Prod-01" -SubnetName "Subnet-01" -SecurityGroupName "NSG-Prod-01" -PublicIpAddressName "PIP-Web-01" -Credential $cred -OpenPorts 80, 3389
```
**Expected Output:** Azure creates dependencies and starts deployment. Status completes with VM specifications.

---

### Step 4: Automate VM Control using Runbooks (Azure Automation)
We will write a block that would run inside an Automation Runbook to stop our VM.

1. Define target VM details:
```powershell
$vmName = "VM-Web-01"
$rg = "RG-AzureAutomation"
```
2. Stop the VM (forcing allocation release to save billing cost):
```powershell
Stop-AzVM -Name $vmName -ResourceGroupName $rg -Force -NoWait
```
**Expected Output:** Command executes asynchronously. The VM changes status in the Portal to `Stopped (deallocated)`.

---

## Common Commands / Cheat Sheet

| Command | Description | Example |
|---------|-------------|---------|
| `Connect-AzAccount` | Login to Azure context | `Connect-AzAccount` |
| `Get-AzSubscription` | List subscriptions linked to account | `Get-AzSubscription` |
| `Get-AzVM` | List details of VMs in subscription | `Get-AzVM -ResourceGroupName "RG"` |
| `Start-AzVM` | Turn on a stopped VM | `Start-AzVM -Name "VMName" -ResourceGroupName "RG"` |
| `Get-AzStorageAccount` | List storage accounts | `Get-AzStorageAccount` |
| `Connect-MgGraph` | Authenticate to Microsoft Graph API | `Connect-MgGraph -Scopes "User.ReadWrite.All"` |

---

## Troubleshooting

| Problem | Likely Cause | Fix |
|---------|-------------|-----|
| **`Connect-AzAccount` fails behind corporate proxy** | Proxy blocks SSL connections to Microsoft endpoints. | Configure local system proxy environment variables inside PowerShell before authenticating. |
| **Command fails with SubscriptionNotFound** | Session targetting wrong active Azure Subscription. | Check subscription ID. Run `Select-AzSubscription -SubscriptionId "[id]"` to swap environments. |
| **`New-AzVM` fails with Quota Exceeded** | Azure region has run out of CPU Core allocations for subscription. | Request a quota increase in the Azure Portal or change the VM deployment region (e.g., EastUS to WestUS). |

---

## Interview Questions

**Q1: What is the difference between the legacy `AzureRM` module and the modern `Az` module?**
> A: The legacy `AzureRM` module was designed only for Windows PowerShell (v5.1 or below). The modern `Az` module is cross-platform, built on .NET Standard, and runs on PowerShell Core (v6.0+) across Windows, Linux, and macOS. It receives active updates and supports newer Azure features.

**Q2: How does an Azure Automation Runbook authenticate securely without passwords?**
> A: It uses a **Managed Identity** (System-assigned or User-assigned). The identity registers as an application object in Entra ID. RBAC permissions are assigned directly to this identity, allowing the script to authenticate automatically inside Azure using local API calls without hardcoding login credentials.

**Q3: How do you list all virtual machines in a specific Resource Group that are currently running?**
> A: Use the following cmdlet filter command:
> `Get-AzVM -ResourceGroupName "RG-Prod" -Status | Where-Object {$_.PowerState -eq "VM running"}`

**Q4: Which cmdlet is used to switch active target subscriptions in Azure PowerShell?**
> A: The `Set-AzContext -SubscriptionId "your_subscription_id"` (or `Select-AzSubscription`) cmdlet is used to switch target contexts.

---

## Related Notes
- [[04-Cloud-and-Security/08-Azure/AZ104-03 Azure Virtual Machines]]
- [[05-Automation-and-Ticketing/10-Scripting-PowerShell/PowerShell-Fundamentals]]
- [[04-Cloud-and-Security/08-Azure/BAK-02 Azure Backup and Site Recovery]]
