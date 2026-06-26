---
tags: [desktop-support, azure, cloud, L2]
aliases: [azure-identity, azure-identity]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #az-104
---

# Azure Identity and Access Management

---

---
## Concept Overview
- **What it is**: Azure Identity and Access Management governs authentication and authorization for Microsoft Azure cloud resources. It leverages Microsoft Entra ID (Azure AD) for identities, and applies Role-Based Access Control (RBAC) to manage permissions on subscriptions, resource groups, and assets.
- **Why it matters for a support engineer**: Support engineers must understand the Azure permission boundaries to grant developers access to cloud resources, troubleshoot access blocks on VMs, audit subscription roles, and manage PIM access.
- **Where you encounter this in real job**: Assigning Owner/Contributor/Reader roles, managing subscription billing access, configuring custom roles, and activating role eligibility in Privileged Identity Management (PIM).
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Checks user role assignments, verifies subscription association, and guides users through PIM activation.
  - **L2**: Assigns RBAC roles at resource group levels, creates Azure custom roles, and troubleshoots access blocks using Activity Logs.
  - **L3**: Configures management group structures, designs tenant-wide PIM eligibility rules, audits global administrator assignments, and implements Azure Policy baselines.

---

---
## Technical Deep Dive
### 1. Azure Resource Hierarchy & Management Boundaries
Azure resources are organized hierarchically. Permissions flow downwards from the top level (Inheritance):

```
                        [Microsoft Entra ID Tenant]
                                    |
                        [Management Groups]
                        (Hierarchical folders for policies/billing)
                                    |
                            [Subscriptions]
                            (Billing and security boundaries)
                                    |
                            [Resource Groups]
                            (Logical groups for related resources)
                                    |
                              [Resources]
                            (VMs, Storage Accounts, VNets)
```

- **Inheritance**: Any RBAC role assigned at the Subscription level is automatically inherited by all Resource Groups and Resources inside it. If you are a Contributor on a Subscription, you are a Contributor on all its resources.

### 2. Azure RBAC Roles vs. Microsoft Entra ID Roles
A common point of confusion is the difference between Entra ID roles and Azure RBAC roles:

| Feature | Microsoft Entra ID Roles | Azure RBAC Roles |
|---|---|---|
| **What they secure** | Entra ID tenant objects (Users, Groups, domains) | Azure cloud resources (VMs, VNets, Storage) |
| **Examples** | Global Admin, User Admin, Helpdesk Admin | Owner, Contributor, Reader, Backup Contributor |
| **Control plane** | `https://entra.microsoft.com` | `https://portal.azure.com` |
| **Scope level** | Tenant-wide | Scoped to Management Group, Sub, RG, or Resource |

#### Key Azure RBAC Roles:
- **Owner**: Full access to all resources, including the ability to delegate access to others (assign RBAC roles).
- **Contributor**: Can create and manage all types of Azure resources, but *cannot* grant access to others.
- **Reader**: Can view existing Azure resources but cannot make any changes.
- **User Access Administrator**: Can manage user access to Azure resources (assign roles) but cannot create resources.

### 3. Privileged Identity Management (PIM)
PIM is an Entra ID P2 service that enforces **Just-In-Time (JIT)** administrative access:
- **Eligible Assignment**: Users are not active administrators permanently. Instead, they are marked as "Eligible".
- **Activation**: When they need to perform work, they must "Activate" the role in PIM, which requires them to complete MFA, provide a business justification, and wait for manager approval (optional).
- **Time-bound**: Once approved, the role is active for a set window (e.g., 4 hours) before automatically revoking.

---


## Real-World Scenarios

### Scenario 1: Developer Blocked from Creating Virtual Machines (Role Inheritance Issue)
**User Complaint:** A cloud developer reports: *"I am trying to deploy a new virtual machine in the 'RG-Dev-Testing' resource group, but the 'Create' button is greyed out. My manager says I have Contributor access."*
**Your First 3 Checks:**
1. Check the developer's effective RBAC role assignments on the 'RG-Dev-Testing' Resource Group.
2. Check if a subscription-level "Reader" role is overriding their permissions, or if there is a role mismatch.
3. Check the Azure Activity Log for the resource group to identify access denied events.
**Diagnosis Steps:**
1. Open Azure Portal -> Go to **Resource Groups** -> Select `RG-Dev-Testing`.
2. Click **Access Control (IAM)** -> Click **Check Access** -> Search for the developer.
   - Effective Role: `Reader` (inherited from the Subscription scope).
3. Check the **Role assignments** tab at the Resource Group level.
   - The developer was assigned `Contributor` at a different Resource Group (`RG-Dev-Storage`) but not on the target `RG-Dev-Testing`.
4. Because the developer only had `Reader` inherited from the Subscription, they were blocked from creating resources in `RG-Dev-Testing`.
**Root Cause:** The developer was assigned the Contributor role at a specific resource group scope rather than the target resource group scope, leaving them with inherited Reader-only access.
**Fix:**
1. Go to **Resource Groups** -> `RG-Dev-Testing` -> **Access Control (IAM)**.
2. Click **Add** -> **Add role assignment**.
3. Select **Contributor**, click Next, select **Members** -> Add the developer's account, and click **Save**.
4. Ask the developer to refresh their portal browser tab. The "Create" button becomes active.
**Prevention:** Group developers into designated AD Security Groups (e.g., `G-Dev-Contributors`) and assign roles to the group rather than individual users.
**Ticket Close Note:** "Assigned Contributor role to the developer at the RG-Dev-Testing resource group scope. Confirmed they can now create VMs. Closed."

### Scenario 2: Eligible Administrator Blocked from Activating PIM Role
**User Complaint:** An database admin reports: *"I need to make a change on a database in our production subscription. I went to PIM to activate my 'Owner' role, but the 'Activate' link is disabled, and it says I do not have a strong authentication method registered."*
**Your First 3 Checks:**
1. Check if the user is using MFA to sign in to the Azure Portal.
2. Verify the configuration settings in the PIM Owner role activation policy.
3. Check the user's registered authentication methods in Microsoft Entra ID.
**Diagnosis Steps:**
1. In Entra Admin Center, locate the database admin. Go to **Authentication methods**.
2. Note that the user has SMS authentication registered, but not the Microsoft Authenticator app.
3. Open the **Microsoft Entra PIM** console -> **Azure resources** -> select the target subscription -> **Roles** -> click the **Owner** role -> **Role settings**.
   - Note the policy rule: `Require multifactor authentication on activation` is set to `True`.
   - The policy enforces a **Phishing-Resistant MFA or Microsoft Authenticator App** registration for role activation. SMS is not accepted.
**Root Cause:** The PIM activation policy required strong authentication (MFA app), which the user had not fully registered.
**Fix:**
1. Generate a Temporary Access Pass (TAP) or guide the user to `aka.ms/setup` to register their Microsoft Authenticator app.
2. Once registered, have them log out and log back in to the Azure Portal using the Authenticator app.
3. Navigate to PIM, select **My roles**, and click **Activate** on the Owner role. The request now proceeds to the MFA code verification screen.
**Prevention:** Configure onboarding policies that require all administrators to register the Authenticator app immediately.
**Ticket Close Note:** "Guided admin through registering Microsoft Authenticator. Verified they could successfully complete MFA and activate their Owner role in PIM. Closed."

---


## Critical Points

> [!danger] Never Do This
> - Never assign RBAC permissions directly to individual user accounts.
> - This makes permission auditing impossible over time. When a user leaves the company, finding and removing their individual access links across multiple subscriptions is difficult. Always assign roles to **Entra ID Security Groups** instead.

> [!warning] Common Trap
> - Assuming that a "Global Administrator" in Microsoft Entra ID automatically has access to view all Azure Subscriptions and Resource Groups.
> - By default, Global Admins cannot see subscriptions unless they explicitly toggle **Access management for Azure resources** to `Yes` in their Entra properties. This elevates their privileges to User Access Administrator on all subscriptions.

> [!tip] Senior Engineer Tip
> - When troubleshooting silent permission blocks, check if there is an active **Azure Lock** on the resource group or resource. A `ReadOnly` or `CanNotDelete` lock will block even subscription Owners from deleting or modifying resources, throwing deceptive permission error messages.

> [!success] Verification Steps
> - Run: `Get-AzRoleAssignment -SignInName "user@company.com"` to verify the active roles and scopes.
> - Check the Azure Activity Log to verify that resource modifications are logged under the user's caller UPN.

> [!question] Interview Alert
> - "What is the difference between Contributor and Owner roles in Azure?"
> - Answer: "Both roles allow full management of all Azure resources, including creating, editing, and deleting virtual machines and networks. However, the Owner role allows the assignment of RBAC roles to other users (managing Access Control / IAM), whereas the Contributor role does not permit role delegation."

---


## Common Mistakes & Fixes

| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| Assigning roles at the individual resource level | Trying to isolate permissions quickly | Apply roles at the Resource Group scope, utilizing folders to separate access levels. |
| Forgetting to toggle "Access management" as Global Admin | Assuming Global Admin is root everywhere | Toggle 'Access management for Azure resources' to Yes in Entra properties to manage subscription billing. |
| Leaving administrative roles active 24/7 | Bypassing PIM to avoid MFA logins | Implement Privileged Identity Management (PIM) to enforce Just-in-Time administrative access. |

---


## Tags
#desktop-support #azure #identity #rbac #L2 #interview-topic #lab-complete #daily-use

---
## Step-by-Step Lab
**Objective:** Install the Az PowerShell module, login, check subscription access, create a resource group, assign a test Reader role, and verify scope inheritance.
**Time Required:** 30 minutes
**Environment Needed:** Azure Free Tier or Sandbox account.
**Pre-requisites:** PowerShell installed as Administrator.

**Steps:**
1. Open PowerShell. Install the Azure Az module if not present:
   ```powershell
   Install-Module -Name Az -AllowClobber -Force
   ```
2. Log in to your Azure environment:
   ```powershell
   Connect-AzAccount
   ```
3. List your active subscriptions and set the context:
   ```powershell
   Get-AzSubscription
   Set-AzContext -SubscriptionName "MyTestSubscription"
   ```
4. Create a test Resource Group:
   ```powershell
   New-AzResourceGroup -Name "RG-Lab-Identity" -Location "eastus"
   ```
5. Assign the **Reader** role to a test user scoped to your new Resource Group:
   ```powershell
   New-AzRoleAssignment -SignInName "testuser@yourdomain.onmicrosoft.com" -RoleDefinitionName "Reader" -ResourceGroupName "RG-Lab-Identity"
   ```
6. Verification: Check the active assignments on the Resource Group:
   ```powershell
   Get-AzRoleAssignment -ResourceGroupName "RG-Lab-Identity" | Select-Object DisplayName, RoleDefinitionName
   ```
   - Confirm the test user is listed with the Reader role.

**Success Criteria:** The Resource Group is created and the Reader role is assigned to the test user successfully, verified via PowerShell.
**Common Failures:** The command fails if your Azure account lacks the "User Access Administrator" or "Owner" role on the target subscription.

---

---
## Cheat Sheet / Quick Reference
### PowerShell
Azure administration uses the `Az` PowerShell module.
```powershell
# Log in to Azure account
Connect-AzAccount

# List all subscriptions accessible by your account
Get-AzSubscription

# Set the active subscription context
Set-AzContext -SubscriptionId "11111111-2222-3333-4444-555555555555"

# Create a new RBAC role assignment (Contributor) for a user at a Resource Group scope
New-AzRoleAssignment -SignInName "developer@company.com" -RoleDefinitionName "Contributor" -ResourceGroupName "RG-Prod-Web"

# List all active RBAC role assignments on a specific Resource Group
Get-AzRoleAssignment -ResourceGroupName "RG-Prod-Web" | Select-Object DisplayName, RoleDefinitionName, Scope
```

### Azure CLI
```bash
# Log in to Azure via browser session
az login

# List subscriptions in a table format
az account list --output table

# Create a Reader role assignment for a user scoped to a Resource Group
az role assignment create --assignee "auditor@company.com" --role "Reader" --resource-group "RG-Prod-Web"
```

### GUI Path
- Go to **portal.azure.com** -> Search for **Subscriptions** or **Resource Groups** -> Click target resource -> Select **Access Control (IAM)** -> **Role assignments**.
- For PIM: Search **Microsoft Entra PIM** -> **My roles** -> **Activate**.

---

> [!info] 60-Second Summary
> **What**: The authorization system managing access to Microsoft Azure cloud resources.
> **Why**: Critical for enforcing least privilege, billing separation, and secure cloud resource administration.
> **How**: Configure resource hierarchy, apply RBAC roles (Owner, Contributor, Reader) at the appropriate scope, and use PIM for JIT access.
> **Command**: `New-AzRoleAssignment` / `az role assignment create`
> **Interview Answer Starter**: "To manage cloud security, I implement Azure RBAC to delegate resource control at the Resource Group or Subscription level, combining it with Privileged Identity Management..."

**Key Numbers to Remember:**
- Default duration of PIM role activations: 1 - 8 hours
- Levels in Azure hierarchy: 4 (Management Groups, Subscriptions, Resource Groups, Resources)
- Port required for portal access: TCP 443 (HTTPS)
- PowerShell module for Azure: `Az`

**3 Things Interviewer Wants to Hear:**
- The difference between Entra roles (identity control) and Azure RBAC (resource control)
- Using Group-based RBAC instead of individual user assignments
- Checking for Resource Locks (Delete/ReadOnly) during permission troubleshooting

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
**Q: What is Azure RBAC and how does it help protect cloud resources?**
A: Azure RBAC (Role-Based Access Control) is a system that manages authorization to Azure resources. It allows you to assign specific permissions—like Owner, Contributor, or Reader—to users at different scope levels, ensuring employees only have the access needed to perform their jobs.

**Q: How do you assign a user permissions to manage a single Virtual Machine?**
A: I navigate to the Virtual Machine in the Azure Portal, click Access Control (IAM), click Add Role Assignment, select the Contributor role, select the user's account, and click Save.

### Intermediate (L2 Level)
**Q: A developer has Contributor access on a Subscription. Can they add a database administrator to a Resource Group inside that subscription?**
A: No. The Contributor role allows a user to create and manage resources, but it does not permit role delegation or management of access control. To assign roles to other users, they would need the User Access Administrator or Owner role on that subscription or resource group.

**Q: What is Azure PIM (Privileged Identity Management) and what is its primary benefit?**
A: Azure PIM is a service that manages and monitors privileged access. Its primary benefit is Just-in-Time (JIT) access, which allows eligible administrators to activate their roles only when needed for a set duration, reducing the risk of administrative credentials being compromised.

### Advanced (L3/Senior Level)
**Q: A developer complains that they cannot delete a Virtual Machine in a Resource Group, even though they have the Owner role on the Subscription. How do you troubleshoot this permission issue?**
A:
- **Situation**: Owner blocked from deleting a virtual machine.
- **Task**: Identify the security control overriding the Owner role permissions.
- **Action**: First, I log into the Azure Portal and check the Virtual Machine resource. I go to the **Locks** settings under the VM blade and parent Resource Group. I check if there is an active `Delete Lock` or `ReadOnly Lock`. Next, I check if an **Azure Policy** is active on the subscription that blocks VM deletions.
- **Result**: I located a `Delete Lock` configured on the parent Resource Group to prevent accidental deletions. Removing the lock allowed the developer to delete the VM successfully.

### HR / Behavioral
**Q: Tell me about a time you had to perform an audit of access rights. What did you discover and how did you resolve it?**
A: I was asked to audit administrative permissions for our Azure subscription. I found that 15 developers had permanent Owner access to our production environment. I met with their lead and explained the security risks. I built a transition plan: I revoked their permanent access, created Entra groups for their roles, and set them up as "Eligible" for Contributor access in Privileged Identity Management (PIM) with mandatory MFA and justification. This reduced our attack surface significantly while still allowing developers to work efficiently.

---

---
## Seedha Simple Mein
*Seedha simple mein: Azure-Identity ke bare mein seekhta hai. Yeh azure infrastructure aur system settings ko properly implement karne aur support tickets ko runbooks ke help se standard templates me clear karne me help karta hai.*

---
## Related Notes
- [[04-Cloud-and-Security/07-Microsoft-365/Microsoft-Entra-ID|Microsoft Entra ID]] — The directory providing identity lookup for Azure.
- [[04-Cloud-and-Security/09-Security/Access-Management|Access Management]] — Discusses privilege management theory and design.
- [[04-Cloud-and-Security/08-Azure/Azure-VMs|Azure VMs]] — Explains virtual machines managed via RBAC.

---
