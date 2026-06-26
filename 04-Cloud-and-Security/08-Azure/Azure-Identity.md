---
tags: [desktop-support, azure, cloud, L2]
aliases: [azure-identity]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #az-104
---

> [!NOTE|color-blue]
> ☁️ **AZURE CLOUD**

`#complete` `#intermediate` `#az-104`

# Azure Identity and Access Management

> [!abstract] Overview
> Azure Identity and Access Management governs authentication and authorization for Microsoft Azure cloud resources, using Microsoft Entra ID and RBAC. Ek support engineer ke liye permissions assign karna, VM access blocks troubleshoot karna aur PIM access manage karna bahut zaroori hai.

---
## 🧠 Concept Overview

- **What it is** — Azure Identity and Access Management governs authentication and authorization for Microsoft Azure cloud resources.
- **Why it matters** — Real job mein resources pe permissions assign karne, blocks troubleshoot karne aur subscription roles audit karne ke liye.
- **Where you see this** — Assigning Owner/Contributor/Reader roles, managing subscription billing access, aur PIM mein role eligibility activate karna.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Checks user role assignments, verifies subscription association, and guides users through PIM activation. |
| **L2** | Assigns RBAC roles at resource group levels, creates Azure custom roles, and troubleshoots access blocks using Activity Logs. |
| **L3** | Configures management group structures, designs tenant-wide PIM eligibility rules, audits global administrator assignments, and implements Azure Policy baselines. |

> [!tip] Seedha Simple Mein
> *Azure-Identity ke bare mein seekhta hai. Yeh azure infrastructure aur system settings ko properly implement karne aur support tickets ko runbooks ke help se standard templates me clear karne me help karta hai. Jaise koi dost samjha raha ho, yeh cloud mein kisko kya access milega, woh control karta hai.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Azure RBAC** is like **a building security system** because...
>
> - Just like a keycard gives you access to specific floors but not others, RBAC gives you access to specific resource groups.
> - Just like a security guard logs who enters and exits, Azure Activity Logs track who modifies resources.

---
## 🔬 Technical Deep Dive

### 1. Azure Resource Hierarchy & Management Boundaries

> [!info] Key Concept
> Permissions flow downwards from the top level (Inheritance).

```text
                        [Microsoft Entra ID Tenant]
                                    |
                        [Management Groups]
                                    |
                            [Subscriptions]
                                    |
                            [Resource Groups]
                                    |
                              [Resources]
```

- **Inheritance**: Any RBAC role assigned at the Subscription level is automatically inherited by all Resource Groups and Resources inside it.

### 2. Azure RBAC Roles vs. Microsoft Entra ID Roles

| Feature | Microsoft Entra ID Roles | Azure RBAC Roles |
|---|---|---|
| **What they secure** | Entra ID tenant objects (Users, Groups) | Azure cloud resources (VMs, Storage) |
| **Examples** | Global Admin, User Admin | Owner, Contributor, Reader |

> [!danger] Common Mistake
> - Assuming that a "Global Administrator" in Microsoft Entra ID automatically has access to view all Azure Subscriptions and Resource Groups. By default, they do not.

### 3. Privileged Identity Management (PIM)

> [!info] Key Concept
> PIM enforces Just-In-Time (JIT) administrative access.

- **Eligible Assignment**: Users are not active administrators permanently.
- **Activation**: Requires MFA, justification, and approval.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Azure Free Tier or Sandbox account.
> - PowerShell installed as Administrator.

### Step 1: Login and Set Context

```powershell
# Install Az module and log in
Install-Module -Name Az -AllowClobber -Force
Connect-AzAccount
Set-AzContext -SubscriptionName "MyTestSubscription"
```

> [!success] Expected Output
> ```
> Logs you in and sets the active subscription context.
> ```

### Step 2: Create Resource Group and Assign Role

```powershell
# Create RG and assign Reader role
New-AzResourceGroup -Name "RG-Lab-Identity" -Location "eastus"
New-AzRoleAssignment -SignInName "testuser@yourdomain.onmicrosoft.com" -RoleDefinitionName "Reader" -ResourceGroupName "RG-Lab-Identity"
```

> [!success] Expected Output
> ```
> Resource Group is created and Reader role is assigned to the test user successfully.
> ```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `Connect-AzAccount` | Log in to Azure account | `Connect-AzAccount` |
| `Get-AzSubscription` | List active subscriptions | `Get-AzSubscription` |
| `New-AzRoleAssignment` | Assign RBAC role | `New-AzRoleAssignment -SignInName "user@domain.com" -RoleDefinitionName "Contributor" -ResourceGroupName "RG-Prod"` |
| `Get-AzRoleAssignment` | View role assignments | `Get-AzRoleAssignment -ResourceGroupName "RG-Prod"` |
| `az role assignment create` | CLI: Assign RBAC role | `az role assignment create --assignee "user@domain.com" --role "Reader" --resource-group "RG-Prod"` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| Create button greyed out | User has inherited Reader role but lacks Contributor on target scope | Assign Contributor role at the specific Resource Group level |
| Can't activate PIM role | User hasn't registered required MFA app | Guide user to aka.ms/setup to register Microsoft Authenticator |
| Access Denied on VM delete | Active Delete or ReadOnly Lock on Resource Group | Remove the Azure Lock from the Resource Group or Resource |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Developer Blocked from Creating Virtual Machines

> [!example] Ticket
> "I am trying to deploy a new virtual machine in the 'RG-Dev-Testing' resource group, but the 'Create' button is greyed out. My manager says I have Contributor access."

**L1 Response:** Check the developer's effective RBAC role assignments on the 'RG-Dev-Testing' Resource Group. Verify if a subscription-level "Reader" role is overriding their permissions.
**Escalation Trigger:** If the role assignment appears correct but the issue persists (e.g. Azure Policy blocking it).
**L2 Resolution:** Assign the Contributor role at the specific 'RG-Dev-Testing' resource group scope instead of relying on inherited access from elsewhere. Verify creation works.

### 🎫 Scenario 2: Eligible Administrator Blocked from Activating PIM Role

> [!example] Ticket
> "I went to PIM to activate my 'Owner' role, but the 'Activate' link is disabled, and it says I do not have a strong authentication method registered."

**L1 Response:** Check the user's registered authentication methods in Microsoft Entra ID. Note if they are only using SMS.
**Escalation Trigger:** If MFA registration fails or policy prevents them from setting it up.
**L2 Resolution:** Review the PIM Owner role activation policy. Guide the user to register the Microsoft Authenticator app (aka.ms/setup) since SMS is not accepted.

---
## 🎤 Interview Questions

> [!question] Q1: What is the difference between Contributor and Owner roles in Azure?
> **Answer:** Both roles allow full management of all Azure resources, including creating, editing, and deleting virtual machines and networks. However, the Owner role allows the assignment of RBAC roles to other users, whereas the Contributor role does not permit role delegation.

==**Exam Tip:** Owner can assign roles, Contributor cannot. Both can manage all resources.==

> [!question] Q2: A developer complains that they cannot delete a Virtual Machine in a Resource Group, even though they have the Owner role on the Subscription. How do you troubleshoot this permission issue?
> **Answer:** I check the Locks settings under the VM blade and parent Resource Group. An active `Delete Lock` or `ReadOnly Lock` will block even subscription Owners. Next, I check if an Azure Policy is active on the subscription blocking deletions.

==**Exam Tip:** Locks always override RBAC permissions.==

---
## 🔗 Related Notes

- [[04-Cloud-and-Security/07-Microsoft-365/Microsoft-Entra-ID|Microsoft Entra ID]] — The directory providing identity lookup for Azure.
- [[04-Cloud-and-Security/09-Security/Access-Management|Access Management]] — Discusses privilege management theory and design.
- [[04-Cloud-and-Security/08-Azure/Azure-VMs|Azure VMs]] — Explains virtual machines managed via RBAC.
