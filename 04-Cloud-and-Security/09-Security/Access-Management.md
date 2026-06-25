---
tags: [desktop-support, security, access-management, rbac, L3]
aliases: [pim-guide, rbac-guide, jit-access]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #az-104
---

# Access Management (RBAC, PIM, and JIT)

---

## Concept Overview
- **What it is**: Access Management is the framework of security policies and technologies that controls user privileges. Key components include **Role-Based Access Control (RBAC)** (permissions linked to job roles), **Privileged Identity Management (PIM)** (time-bound administrative access), and **Just-in-Time (JIT)** access (opening management ports dynamically).
- **Why it matters for a support engineer**: Administrators are prime targets for cyberattacks. Support engineers manage access delegation, troubleshoot PIM role activation issues, configure JIT VM rules, and ensure that the principle of least privilege is maintained.
- **Where you encounter this in real job**: Activating admin roles in PIM, assigning custom RBAC roles to development teams, requesting JIT VM RDP access, and maintaining emergency "Break-Glass" credentials.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Adds users to scoped security groups, assists with PIM login prompts, and verifies basic user rights.
  - **L2**: Assigns RBAC roles at resource group scopes, configures JIT rules on virtual machines, and audits active role assignments.
  - **L3**: Designs the enterprise RBAC directory structure, configures global PIM approval workflows, implements Privileged Access Workstations (PAWs), and audits emergency break-glass account check-outs.

---

## Technical Deep Dive

### 1. Role-Based Access Control (RBAC) Design
RBAC groups access rights into roles. Instead of assigning individual permissions to users, you assign **Roles** (like Reader, Contributor, or custom roles) to **Security Groups**:
- **Scope Hierarchy**: Roles can be scoped at four levels: Management Group -> Subscription -> Resource Group -> Resource.
- **Inheritance**: Roles assigned at a higher scope flow down automatically. A Reader at the subscription level has read access to all virtual machines and networks inside it.

### 2. Privileged Identity Management (PIM)
PIM eliminates "standing access" (permanent administrator privileges) in favor of **Eligible Access**:

```
[Eligible User] --[Requests Activation]--> [PIM Engine]
                                                | (Evaluates Policy)
                                                v
[Access Granted (Time-Bound)] <--- (Approver Approved) <--- [MFA Check + Justification]
```

- **Active Assignment**: The user has the role permissions 24/7 (standing access - insecure).
- **Eligible Assignment**: The user lacks the role permissions by default. When they need to perform work:
  1. They open PIM and click **Activate**.
  2. They must satisfy an MFA challenge.
  3. They input a business ticket justification.
  4. The role is active for a set window (e.g., 4 hours) and then automatically deactivates.

### 3. Just-in-Time (JIT) VM Access
Leaving management ports (TCP `3389` for RDP, TCP `22` for SSH) open to the internet invites brute-force attacks.
- **How JIT works**: A security policy blocks these ports by default. When an administrator needs to connect:
  1. They request access via the Azure Portal or PowerShell.
  2. Azure verifies their RBAC permissions.
  3. The network security group (NSG) dynamically creates an inbound rule allowing traffic **only** from the administrator's current public IP address on the specified port.
  4. The rule automatically expires and is deleted after a set duration (e.g., 2 hours).

### 4. Privileged Access Workstations (PAWs) & Break-Glass Accounts
- **PAW**: A highly hardened physical workstation used *only* for administrative tasks (e.g., editing AD, managing Azure). It has no email access and no web browsing capability, reducing malware compromise risks.
- **Emergency Access (Break-Glass) Accounts**: Dedicated accounts used to regain tenant access if standard authentication fails (e.g., during an Entra ID MFA outage).
  - *Design*: Excluded from all Conditional Access policies, do not use phone-based MFA (often use physical FIDO2 keys stored in a physical safe), and trigger immediate high-priority alerts when logged in.

---

## Commands & Syntax

### PowerShell
Access management uses the `Az.Resources` and `Microsoft.Graph.Identity.Governance` modules.
```powershell
# Connect to Azure
Connect-AzAccount

# Create a new RBAC role assignment scoped to a specific Resource Group
New-AzRoleAssignment -SignInName "admin@company.com" -RoleDefinitionName "User Access Administrator" -ResourceGroupName "RG-Prod-Services"

# Request JIT VM Access for a virtual machine via PowerShell (opens port 3389 for 2 hours)
Start-AzNetworkWatcherResourceIPFlowVerify -TargetResourceId "/subscriptions/sub-id/resourceGroups/RG-Prod/providers/Microsoft.Compute/virtualMachines/VM-App" ... # (or use MDE JIT cmdlets)

# Connect to Microsoft Graph with PIM scopes
Connect-MgGraph -Scopes "RoleAssignmentSchedule.ReadWrite.Directory"

# Query active PIM role schedules for Entra ID roles
Get-MgRoleManagementDirectoryRoleAssignmentSchedule | Select-Object PrincipalId, RoleDefinitionId, Status
```

### Azure CLI
```bash
# Create a Contributor role assignment for a group scoped to a subscription
az role assignment create --assignee-object-id "group-guid" --role "Contributor" --scope "/subscriptions/11111111-2222-3333-4444-555555555555"

# Request JIT VM connection access using Azure CLI
az security jit-policy request --location eastus --name "default" --resource-group RG-Prod --vm-name VM-App --port 3389 --allowed-ips "203.0.113.10"
```

### GUI Path
- **Azure RBAC**: Azure Portal -> Resource Group -> **Access Control (IAM)** -> **Role assignments**.
- **Entra PIM**: Entra Admin Center -> **Protection** -> **Privileged Identity Management**.
- **JIT VM Access**: Azure Portal -> **Microsoft Defender for Cloud** -> **Cloud Security** -> **Just-in-time VM access**.

---

## Real-World Scenarios

### Scenario 1: Setting Up Just-in-Time (JIT) VM Access for a Remote Developer
**User Complaint:** A contractor reports: *"I am trying to log in to the application VM 'VM-Dev-09' via RDP to review log files, but the connection times out. The network team says the RDP port is closed. I have Contributor access on the resource group."*
**Your First 3 Checks:**
1. Check the Network Security Group (NSG) associated with `VM-Dev-09`.
2. Verify if Just-in-Time VM Access is enabled for this server in Defender for Cloud.
3. Check the developer's public IP address.
**Diagnosis Steps:**
1. Open Azure Portal -> Navigate to `VM-Dev-09` -> Click **Networking**.
2. Notice the inbound rules: a rule named `Microsoft.Security/JIT/...` is actively blocking port 3389.
3. The VM is protected by a JIT policy. To connect, the developer must trigger a JIT request to open the port.
4. *Decision*: Do not open port 3389 permanently. Instruct the developer to request JIT activation.
**Root Cause:** The virtual machine is protected by a Just-in-Time access policy, blocking standing RDP connections.
**Fix:**
1. Instruct the developer to go to the Azure Portal -> **Virtual Machines** -> select `VM-Dev-09`.
2. Click **Connect** -> Select **RDP** -> Click **Request access** (or run `az security jit-policy request`).
3. Under the hood: Azure queries the developer's public egress IP (`203.0.113.50`), modifies the NSG to add a temporary rule allowing `203.0.113.50` to access port `3389`, and starts a 2-hour timer.
4. Instruct the developer to connect. RDP opens successfully.
**Prevention:** Configure JIT policy templates to automatically apply to all newly provisioned developer VMs.
**Ticket Close Note:** "JIT RDP access requested and granted for developer IP 203.0.113.50. Port 3389 opened for 2 hours. Closed."

### Scenario 2: Emergency Tenant Recovery Using a Break-Glass Account
**User Complaint:** A critical incident ticket is raised: *"An authentication routing failure in Microsoft Entra ID has caused a complete outage of our phone-based MFA services. No administrators can log in to the portal because the MFA prompt never arrives on their phones. We need to log in to bypass MFA and configure emergency settings."*
**Your First 3 Checks:**
1. Locate the secure physical safe containing the Break-Glass account credentials.
2. Confirm the account exists in Microsoft Entra ID and is excluded from CA.
3. Execute the emergency logon and track the session activity.
**Diagnosis Steps:**
1. Retrieve the sealed envelope containing the credentials for our emergency account `emergency-admin@company.onmicrosoft.com` from the IT Director's safe.
2. Go to the Azure Portal. Enter the username and password.
3. *Evaluate CA policy*: The account is excluded from the 'Enforce-MFA-All-Admins' Conditional Access policy. It bypasses the MFA prompt and logs in directly.
4. *Action*: Once logged in, navigate to Conditional Access policies. Temporarily disable the MFA requirement for our primary administrators or switch the policy to accept security keys while the telephony issue is resolved by Microsoft.
**Root Cause:** A tenant-wide MFA infrastructure outage blocked standard admin logons.
**Fix:**
1. Log in using the emergency Break-Glass account (which bypasses MFA).
2. Adjust Conditional Access policies to restore administrative service access.
3. Log out of the emergency account. Return the credentials to the secure safe, reset the password, and log the incident.
**Prevention:** Regularly test the Break-Glass account logon process quarterly, ensuring it triggers immediate security alerts.
**Ticket Close Note:** "Logged in using emergency Break-Glass account. Bypassed CA to disable MFA rules temporarily during Azure MFA outage. Reset emergency password. Closed."

---

## Critical Points

> [!danger] Never Do This
> - Never grant any user permanent (standing) Global Administrator or Subscription Owner access.
> - Standing admin access is a high security risk. If a user's account is compromised, the attacker has instant, permanent control of the tenant. Always configure these roles as **Eligible** in PIM, requiring MFA and approval to activate.

> [!warning] Common Trap
> - Forgetting to monitor the activation and checkout logs of emergency Break-Glass accounts.
> - Because Break-Glass accounts bypass MFA and security rules, they are highly targeted. If a Break-Glass account logs in and you do not have an automated high-priority alert (sent via SMS/Email to the security team), an attacker could control your tenant undetected.

> [!tip] Senior Engineer Tip
> - When designing custom RBAC roles, use the wildcard `*` with caution. Instead of granting `Microsoft.Compute/*` (which allows all actions including deleting VMs), specify only the actions needed, such as `Microsoft.Compute/virtualMachines/read` and `Microsoft.Compute/virtualMachines/start/action`.

> [!success] Verification Steps
> - Run: `Get-MgRoleManagementDirectoryRoleAssignmentSchedule` to verify that administrative roles are configured as 'Eligible' rather than 'Active'.
> - Verify that JIT connection rules in the NSG disappear automatically after the expiration timeout.

> [!question] Interview Alert
> - "What is the difference between an 'Eligible' role assignment and an 'Active' role assignment in PIM?"
> - Answer: "An Active assignment means the user has the role permissions 24/7. An Eligible assignment means the user does not have the permissions by default, but is authorized to request them. When they need to perform work, they must activate the role in PIM by completing MFA and providing a justification, which grants them the permissions for a limited duration."

---

## Common Mistakes & Fixes

| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| Granting permanent Owner access to developers | Seeking to avoid PIM activation alerts | Configure the developers as Eligible Contributors in PIM, keeping security high. |
| Leaving management ports (3389/22) open to Any source | Quick VM setup habits | Enforce Just-in-Time (JIT) VM Access in Defender for Cloud. |
| Including Break-Glass accounts in global CA block policies | Overlooking exclusions list | Always exclude the emergency account from all Conditional Access and MFA policies. |

---

## Lab Exercise

**Objective:** Configure a resource group, create a custom RBAC role using PowerShell, assign it to a test user, and verify scope inheritance.
**Time Required:** 30 minutes
**Environment Needed:** Azure Free Tier or Sandbox account.
**Pre-requisites:** Azure Az PowerShell module logged in.

**Steps:**
1. Open PowerShell. Create the Resource Group:
   ```powershell
   New-AzResourceGroup -Name "RG-Lab-Access" -Location "eastus"
   ```
2. Define a custom RBAC role in a JSON format that only permits starting and stopping virtual machines:
   Create a local file `CustomRole.json`:
   ```json
   {
     "Name": "Virtual Machine Operator Lab",
     "Id": null,
     "IsCustom": true,
     "Description": "Can start and stop virtual machines.",
     "Actions": [
       "Microsoft.Storage/*/read",
       "Microsoft.Network/*/read",
       "Microsoft.Compute/*/read",
       "Microsoft.Compute/virtualMachines/start/action",
       "Microsoft.Compute/virtualMachines/powerOff/action"
     ],
     "NotActions": [],
     "AssignableScopes": [
       "/subscriptions/your-subscription-id"
     ]
   }
   ```
3. Create the custom role in Azure:
   ```powershell
   New-AzRoleDefinition -InputFile "C:\Temp\CustomRole.json"
   ```
4. Assign the new custom role to a test user scoped to your Resource Group:
   ```powershell
   New-AzRoleAssignment -SignInName "testuser@yourdomain.com" -RoleDefinitionName "Virtual Machine Operator Lab" -ResourceGroupName "RG-Lab-Access"
   ```
5. Verification: Check the active assignments:
   ```powershell
   Get-AzRoleAssignment -ResourceGroupName "RG-Lab-Access" | Select-Object DisplayName, RoleDefinitionName
   ```
   - Confirm the test user is listed with the custom role.

**Success Criteria:** The custom role is created, registered in the subscription, and assigned to the test user successfully, verified via PowerShell.
**Common Failures:** The command fails if the subscription ID in the JSON file's AssignableScopes does not match your active subscription.

---

## Interview Questions & Answers

### Basic (L1 Level)
**Q: What is the principle of Least Privilege?**
A: The principle of least privilege is a security rule stating that users should only be granted the minimum access levels (permissions) necessary to complete their job tasks. This limits the potential damage if an account is compromised.

**Q: What is a Break-Glass account in Microsoft 365?**
A: A Break-Glass account is an emergency-access global administrator account that is excluded from all Conditional Access and MFA policies. It is kept as a backup to allow access to the tenant if all standard admin logins are blocked (e.g., during an MFA outage).

### Intermediate (L2 Level)
**Q: How does Just-in-Time (JIT) VM Access protect cloud virtual machines?**
A: JIT VM Access blocks management ports like TCP 3389 (RDP) and TCP 22 (SSH) by default. When an administrator needs to connect, they request access. Azure verifies their permissions and dynamically opens the port in the Network Security Group, allowing traffic *only* from the admin's specific public IP address for a limited time (e.g., 2 hours), closing it automatically afterward.

**Q: What is the difference between Azure AD Roles and Azure RBAC Roles?**
A: Azure AD (Entra) roles control permissions for cloud directory objects (like users, groups, domains, and licenses). Examples include Global Administrator and User Administrator. Azure RBAC roles control permissions for Azure cloud resources (like virtual machines, storage accounts, and virtual networks). Examples include Owner, Contributor, and Reader.

### Advanced (L3/Senior Level)
**Q: You are configuring Privileged Identity Management (PIM) for a group of security engineers. The compliance team requires that any activation of the 'Security Administrator' role must require approval from the IT Director and expire after 2 hours. Describe your configuration steps.**
A:
- **Situation**: Implementing role activation policies in PIM.
- **Task**: Configure approval and time-bound constraints for a high-privilege role.
- **Action**: In the Entra Admin Center, I open the **Microsoft Entra PIM** console. I navigate to Microsoft Entra roles -> Roles -> select **Security Administrator** -> click **Settings** (Role settings). I edit the activation policy: under 'Maximum activation duration', I set the value to `2` hours. I check the box 'Require MFA on activation'. Under 'Require approval to activate', I toggle it to 'Yes', and select the IT Director's account as the designated approver.
- **Result**: The security engineers are configured as eligible; their role activations require director approval and expire automatically after 2 hours.

### HR / Behavioral
**Q: Tell me about a time you had to audit user permissions. What did you discover, and how did you communicate your findings to the business?**
A: I conducted an audit of our Azure storage permissions and found that our developers had assigned the "Contributor" role to several external contractor accounts, allowing them to download database backups. I documented these access paths and compiled a risk report. I met with the development lead, explaining how this violates data compliance rules. I helped them transition the contractors to a custom "Reader" role that only allowed read access to specific non-production folders, securing our data while allowing the contractors to finish their testing.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **What**: The security framework managing user privileges, utilizing RBAC for job roles, PIM for JIT admin access, and JIT VM port controls.
> **Why**: Critical for preventing credential abuse, stopping lateral movement, and enforcing least privilege.
> **How**: Scoping roles to groups (IAM), configuring eligible roles in PIM, and securing RDP/SSH ports via JIT policies.
> **Command**: `New-AzRoleAssignment` / `az security jit-policy request`
> **Interview Answer Starter**: "To implement access management, I enforce Role-Based Access Control to eliminate direct user assignments, combining it with PIM for JIT admin access..."

**Key Numbers to Remember:**
- Default duration for JIT port access: 2 - 3 hours
- Levels in RBAC hierarchy: 4
- Standard port for RDP: TCP 3389 / SSH: TCP 22
- Target Emergency Accounts: Minimum of 2 Break-Glass accounts recommended

**3 Things Interviewer Wants to Hear:**
- Standing administrative access is a major security risk and should be replaced by PIM
- Using JIT VM Access blocks public RDP ports (3389) from constant scanner attacks
- The setup and testing process for emergency Break-Glass accounts

---

## Related Notes
- [[04-Cloud-and-Security/08-Azure/Azure-Identity|Azure Identity]] — Details resource scopes and IAM bindings.
- [[04-Cloud-and-Security/07-Microsoft-365/Microsoft-Entra-ID|Microsoft Entra ID]] — The identity system managing administrative users.
- [[04-Cloud-and-Security/09-Security/CIA-Triad-and-Zero-Trust|CIA Triad and Zero Trust]] — Covers the Least Privilege concept.

---

## Tags
#desktop-support #security #access-management #rbac #L3 #interview-topic #lab-complete #daily-use

