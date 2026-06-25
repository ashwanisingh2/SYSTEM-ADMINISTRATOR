---
tags: [desktop-support, career-roadmap, certification, microsoft, azure, cloud, az-104, L1, L2, L3]
aliases: [az-104-prep, azure-administrator-roadmap]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #az-104
---

# AZ-104: Microsoft Azure Administrator Certification Study Roadmap

---

## Concept Overview
- **What it is**: A comprehensive preparation guide, resource inventory checklist, and execution strategy for passing the Microsoft Azure Administrator Associate (AZ-104) certification.
- **Why it matters**: Cloud computing is standard in modern systems administration. The AZ-104 validates an engineer's ability to manage Azure identities, compute instances, storage accounts, virtual networks, and backup resources at scale.
- **Where you encounter this in real job**: Managing subscription budgets, configuring cross-region network peering, assigning Role-Based Access Control (RBAC) permissions, and automating VM provisioning.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Resets user passwords in Entra ID, checks VM statuses, registers MFA devices, and monitors backup task indicators.
  - **L2**: Provisions Azure storage containers, configures network security groups (NSGs), handles virtual network peering, and deploys Azure VMs.
  - **L3**: Architectures multi-region cloud networks, configures hybrid VPN tunnels, creates custom RBAC roles, and implements automation policies.

---

## Technical Deep Dive

### AZ-104 Exam Domains and Weights
The exam covers five functional domains:
```
  [1. Identity & Governance (15-20%)] ---> [2. Implement & Manage Storage (15-20%)]
                                                   |
                                                   v
  [3. Deploy & Manage Compute (20-25%)]  ---> [4. Config & Manage VNet (20-25%)]
                                                   |
                                                   v
                       [5. Monitor & Back Up Azure Resources (10-15%)]
```

### Identity and Access Governance
- **Azure AD / Microsoft Entra ID**: Managing cloud identities, licenses, and group membership structures.
- **Azure RBAC (Role-Based Access Control)**: Provides granular access management for Azure resources. Applied at four scopes: Management Group, Subscription, Resource Group, or Resource. Inherits downward.
- **Azure Policy**: Enforces corporate rules and compliance standards across your subscriptions (e.g. blocking the creation of high-cost VMs or restricting deployments to certain regions).

---

## Commands & Syntax

### PowerShell: Azure Administrator Command Operations
```powershell
# 1. Log in to Azure and select subscription target
Connect-AzAccount
Set-AzContext -SubscriptionId "sub-prod-01-guid"

# 2. Deploy a new Storage Account with LRS redundancy
New-AzStorageAccount -ResourceGroupName "RG-Prod-Core" `
    -Name "storeprodcore009" `
    -Location "EastUS" `
    -SkuName "Standard_LRS" `
    -Kind "StorageV2"

# 3. Create a Virtual Network Peering link
Add-AzVirtualNetworkPeering -Name "VNet1-to-VNet2" `
    -VirtualNetwork (Get-AzVirtualNetwork -Name "VNet1" -ResourceGroupName "RG-Core") `
    -RemoteVirtualNetworkId "/subscriptions/sub-id/resourceGroups/RG-Core/providers/Microsoft.Network/virtualNetworks/VNet2"
```

### Azure CLI
```bash
# Query all virtual machines in a resource group and output as a clean table
az vm list --resource-group "RG-Prod-Core" --output table

# Create a resource lock preventing deletion on a storage account
az lock create --name "PreventDeleteStorage" \
  --resource-group "RG-Prod-Core" \
  --resource-name "storeprodcore009" \
  --resource-type "Microsoft.Storage/storageAccounts" \
  --lock-type CanNotDelete
```

### GUI Path
> **Azure Portal**: Subscriptions -> Access Control (IAM) -> Add role assignment (RBAC config)
> **Azure Portal**: Monitor -> Metrics (to analyze CPU/IOPS utilization trends)

---

## Real-World Scenarios

### Scenario 1: VNet Peering Disruption causing VM Isolation
**User Complaint**: VMs in `VNet-Development` can no longer communicate with the database servers hosted in `VNet-Production`, interrupting application tests.
**Your First 3 Checks**:
1. Check the provisioning state of the peering link between the two VNets in the portal.
2. Check if the address spaces of the peer VNets overlap.
3. Review the Network Security Group (NSG) rules on the database subnet interfaces.
**Diagnosis Steps**:
1. Open the Azure Portal and navigate to `VNet-Development` -> Peerings.
2. The peering connection state to `VNet-Production` is marked as: **Disconnected**.
3. Check the route tables assigned to the subnet: no route routes traffic to the peer network space.
**Root Cause**: A junior administrator added a new IP address space (`10.0.0.0/16`) to `VNet-Development` that overlapped with `VNet-Production`'s address range. When address spaces overlap, Azure disables peering links to prevent routing conflicts.
**Fix**:
1. Remove the overlapping address space configuration from `VNet-Development`.
2. Re-create the peering link on both VNets, ensuring "Allow Virtual Network Access" is enabled.
3. Verify the peering connection state transitions back to: **Connected**.
**Prevention**: Maintain an IP Address Management (IPAM) sheet tracking all subnet spaces to prevent overlapping ranges during network expansions.

### Scenario 2: Storage Account Access Key Compromise
**User Complaint**: An application log files storage account access key was mistakenly committed to a public GitHub repository. You must secure the storage account immediately.
**Your First 3 Checks**:
1. Identify if the compromised key is Key 1 or Key 2.
2. Confirm the active applications using the storage account connection string.
3. Prepare to rotate the access keys.
**Diagnosis Steps**:
1. The compromised string contains `key1` of storage account `storeprodlogs`.
2. Confirm the application connection configuration: it is currently configured to connect using `key1`.
**Root Cause**: Credential leakage in public source repositories exposes the storage account to unauthorized read/write access.
**Fix**:
1. Log into the Azure Portal and select the storage account -> Access Keys.
2. Configure active applications to switch their connection strings to use **Key 2** as a temporary backup.
3. Click **Regenerate Key 1** in the Azure portal. This revokes the compromised key instantly.
4. Update application parameters to use the newly generated Key 1 and verify connection health.
**Prevention**: Implement pre-commit Git hooks (like git-secrets) to scan code repositories for private keys and connection string formats before code commits.

---

## Critical Points

> [!danger] Never Do This
> Do not assign administrative roles (like Contributor or Owner) at the root Subscription level when assigning access to external vendors. Always apply permissions at the specific Resource Group or Resource level to follow the Principle of Least Privilege.

> [!warning] Common Trap
> Confusing Azure Active Directory (Entra ID) roles with Azure RBAC roles. Entra roles (like Global Admin) manage directory resources like users and domains. Azure RBAC roles (like Owner or Contributor) manage Azure infrastructure resources like VMs, databases, and networks.

> [!tip] Senior Engineer Tip
> Configure **Diagnostic Settings** on all critical storage accounts and virtual networks to route activity logs to a central Log Analytics Workspace. This allows you to run KQL queries to trace resource creation and deletion events.

> [!success] Verification Steps
> To verify VNet Peering routing:
> 1. Open the VM console inside `VNet-Development`.
> 2. Run `tnc 10.2.1.4 -port 3389` (Test-NetConnection) in PowerShell targeting the peer VM.
> 3. Verify `TcpTestSucceeded` returns `True`.

> [!question] Interview Alert
> "What is the difference between Local Redundant Storage (LRS) and Geo-Redundant Storage (GRS)?"
> - **Answer**: LRS replicates your data three times within a single data center in the primary region, protecting against local hardware failure. GRS replicates your data synchronously three times within the primary region, and then asynchronously to a secondary paired region hundreds of miles away, protecting against a regional disaster.

---

## Common Mistakes & Fixes

| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| Virtual machine resizing failure | Resizing to a VM SKU not supported in the active hardware cluster | Stop the VM first to deallocate the hardware node, then change the VM size to the new SKU. |
| Locked resource delete attempts fail | Forgetting active resource locks | Remove the Resource Lock (`CanNotDelete`) from the properties tab before initiating deletion. |
| Inbound network rule conflicts | Multiple NSG rules with overlapping priority numbers | Assign security rules unique priority levels (lower numbers have higher priority). |

---

## Lab Exercise

**Objective**: Configure Azure Backup for a Virtual Machine and run a Backup Job manually.
**Time Required**: 25 minutes
**Environment Needed**: Active Azure Subscription and a test Windows/Linux VM.

**Steps**:
1. Log in to the Azure Portal.
2. Search for and select **Recovery Services Vaults**.
3. Click **Create** to deploy a vault:
   - Resource Group: `RG-Cloud-Backup`
   - Vault Name: `Vault-Prod-Backup`
   - Location: Same as your VM (e.g. EastUS)
4. Open the created Vault and go to Get Started -> **Backup**.
5. Select Workload: **Azure** and resource type: **Virtual Machine**. Click Configure Backup.
6. Select Backup Policy: Create a new daily backup policy or select default.
7. Select Virtual Machine: Choose your test VM. Click Enable Backup.
8. Under Backup Items -> Azure Virtual Machine, select your VM and click **Backup Now**.

**Pass Criteria**: The backup deployment completes and the manual backup status transitions to "Completed" inside the vault dashboard.

---

## Interview Questions & Answers

### L1 Level

#### Q1: What is Azure Cloud Shell?
**A**: Azure Cloud Shell is an interactive, browser-accessible command-line shell used for managing Azure resources. It supports both Bash and PowerShell modules, integrates automatically with your account credentials, and provides pre-installed tools like the Azure CLI, kubectl, and Git.

#### Q2: What is the purpose of tags in Azure?
**A**: Tags are key-value pairs applied directly to Azure resources and resource groups. They are used to organize resources for consolidated billing, identify cost centers, define environment types (e.g. Production vs Development), and automate management scripts.

---

### L2 Level

#### Q3: Explain how Azure Load Balancer differs from Azure Application Gateway.
**A**:
- **Azure Load Balancer**: Operates at Layer 4 (Transport Layer - TCP/UDP) and routes incoming traffic based on IP addresses and ports to backend pools.
- **Application Gateway**: Operates at Layer 7 (Application Layer - HTTP/HTTPS) and provides advanced routing features like URL-path routing, cookie-based affinity, SSL termination, and Web Application Firewall (WAF) services.

#### Q4: What is Azure AD Join and how does it help manage corporate laptops?
**A**: Azure AD Join registers and configures corporate laptops directly in Microsoft Entra ID. It allows users to log in to their Windows devices using their corporate M365 email accounts, registers the device in Microsoft Intune for configuration policies, and enables Single Sign-On (SSO) to cloud apps.

---

### L3 Level

#### Q5: Explain how you would recover a virtual machine if its primary disk becomes corrupted.
**A**:
- **Situation**: A production VM failed to boot, showing a blue screen boot loop, indicating file system corruption.
- **Task**: Restore virtual machine operations as quickly as possible.
- **Action**: I used Azure PowerShell to stop the VM and detach the corrupted OS disk. I created a snapshot of the disk and provisioned a temporary VM. I attached the snapshot disk as a data disk to the temporary VM. I ran repair commands (`chkdsk`, restored registry files) on the disk. Once repaired, I detached it, replaced the original OS disk on the production VM, and booted the VM.
- **Result**: Successfully repaired the VM's file system, avoiding a full restoration write cycle and reducing MTTR by two hours.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **RBAC Scopes**: Management Groups -> Subscriptions -> Resource Groups -> Resources. Permissions inherit down.
> **Networking**: Peering is non-transitive (VNet A peered to B, and B peered to C, does not allow A to talk to C).
> **Storage**: Use Shared Access Signatures (SAS) to grant temporary, restricted access to storage files.
> **Command**: `az vm resize --resource-group RG-Core --name VM-Prod --size Standard_D2s_v3`

---

## Related Notes
- [[04-Cloud-and-Security/08-Azure/Azure-VMs|Azure VMs]]
- [[04-Cloud-and-Security/08-Azure/Azure-Networking|Azure Networking]]
- [[04-Cloud-and-Security/08-Azure/Azure-Identity|Azure Identity]]

---

## Tags
#desktop-support #azure #az-104 #cloud-administrator #virtual-networks #rbac-security #certification-roadmap #L2 #L3

