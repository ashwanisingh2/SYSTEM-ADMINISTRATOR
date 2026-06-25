---
tags: [desktop-support, active-directory, organizational-units, L1]
aliases: [ad-containers, ou-delegation]
created: 2026-06-25
status: #complete
difficulty: #beginner
cert-relevant: #md-102
---

# Organizational Units

---

## Concept Overview
- **What it is**: An Organizational Unit (OU) is a logical container object in an Active Directory domain used to organize users, computers, groups, and other objects. OUs act as boundaries for administrative delegation and Group Policy Object (GPO) application.
- **Why it matters for a support engineer**: OUs partition the Active Directory database. A support engineer must know how to navigate the OU structure, delegate permissions (like password resets) to helpdesk staff, and link GPOs to target objects.
- **Where you encounter this in real job**: Organizing new computers into corporate OUs, delegating local admin rights to branch technicians, troubleshooting GPO application issues, and protecting OUs from accidental deletion.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Navigates the OU tree, moves computer objects to target OUs during deployment, and checks basic delegation rights.
  - **L2**: Creates new OUs, configures delegation of control for helpdesk staff, and links GPOs to OUs.
  - **L3**: Designs the enterprise OU hierarchy, implements directory tiering models (e.g., Active Directory Administrative Tiering), and audits GPO inheritance loops.

---

## Technical Deep Dive

### 1. The Role of OUs
Unlike domains, OUs are not security boundaries. They are management containers:
- **Logical Organization**: Groups resources logically (e.g., by department, geography, or object type).
- **Delegation of Control**: Allows administrators to grant specific administrative permissions (e.g., resetting passwords) to users or groups over objects within a specific OU, without granting full Domain Admin rights.
- **Group Policy Application**: GPOs are linked directly to OUs, allowing target settings to apply to all users and computers within that container.

### 2. OU Design Best Practices
- **Logical Tiering**: Separate administrative accounts and servers from standard users and workstations (e.g., Tier 0: Domain Controllers, Tier 1: Servers, Tier 2: Workstations).
- **Flat Structure**: Avoid deep nesting (more than 3 to 4 levels) as it complicates Group Policy inheritance and slows down object queries.
- **Geographic vs. Functional**:
  - *Functional (Recommended)*: Structuring by department/role (e.g., `OU=Workstations`, `OU=Users`).
  - *Geographic*: Structuring by location (e.g., `OU=London`, `OU=NewYork`), useful for decentralized IT teams.

### 3. Accidental Deletion Protection
Windows Server 2008+ introduced the **Protect object from accidental deletion** flag:
- **Mechanism**: Adds a Deny permission ACE (Access Control Entry) for the "Everyone" group blocking Delete and Delete Subtree operations.
- If you attempt to delete the OU while this flag is active, Active Directory blocks the action, returning an "Access is denied" error.

---

## Commands & Syntax

### PowerShell
```powershell
# Create a new Organizational Unit with accidental deletion protection enabled
New-ADOrganizationalUnit -Name "SalesWorkstations" -Path "OU=Workstations,DC=lab,DC=local" -ProtectedFromAccidentalDeletion $true

# Disable accidental deletion protection on an OU to allow for modification
Set-ADOrganizationalUnit -Identity "OU=SalesWorkstations,OU=Workstations,DC=lab,DC=local" -ProtectedFromAccidentalDeletion $false
```

### CMD / Run Box
```cmd
REM Launch Active Directory Users and Computers console
dsa.msc
REM Move a computer object to a different OU from the command line
dsmove "CN=WORKSTATION01,CN=Computers,DC=lab,DC=local" -newparent "OU=Workstations,DC=lab,DC=local"
```

### GUI Path
> Server Manager -> Tools -> **Active Directory Users and Computers** -> Right-click Domain -> **New** -> **Organizational Unit**.
> *Note: To disable deletion protection in GUI: View -> Check **Advanced Features** -> Right-click OU -> Properties -> Object tab -> Uncheck "Protect object from accidental deletion".*

### Important Registry Paths
```
HKLM\SYSTEM\CurrentControlSet\Services\NTDS\Parameters\
```

### Key Event IDs
| Event ID | Meaning | Where to Check |
|----------|---------|----------------|
| 4741 | A computer account was created | Security Log |
| 4743 | A computer account was deleted | Security Log |
| 4662 | An operation was performed on an Active Directory object (Auditing required) | Security Log |

---

## Real-World Scenarios

### Scenario 1: Group Policy Fails to Apply to New Laptops (Workstation in Default Container)
**User Complaint:** "We deployed 10 new laptops today. They joined the domain, but they are not receiving our corporate wallpaper, software packages, or security settings."
**Your First 3 Checks:**
1. Check where the new computer objects reside in the Active Directory tree.
2. Verify if the target GPOs are linked to that container.
3. Check the client group policy report using `gpresult`.
**Diagnosis Steps:**
1. Open ADUC (`dsa.msc`).
2. Search for the laptop hostnames. They are located in the default **Computers** container.
3. The default **Computers** container is a container, not an OU. GPOs cannot be linked to default containers.
**Root Cause:** The laptops were left in the default Computers container after domain join, preventing them from inheriting policies linked to the Workstations OU.
**Fix:** Move the computer objects to the `OU=Workstations,DC=lab,DC=local` OU and run `gpupdate /force` on the clients.
**Prevention:** Change the default computer landing container in Active Directory using the `redircmp` utility:
```cmd
redircmp "OU=Workstations,DC=lab,DC=local"
```
**Ticket Close Note:** "Moved computer objects from default Computers container to Workstations OU. Verified successful GPO application on clients. Redirected default landing container. Closing ticket."

### Scenario 2: Helpdesk Engineer Cannot Reset Passcards (Delegation Failure)
**User Complaint:** "I was assigned to the helpdesk group, but when I try to reset a user's password in ADUC, I get an access denied error."
**Your First 3 Checks:**
1. Check the helpdesk engineer's group memberships in ADUC.
2. Verify the delegation settings on the target OU.
3. Check if the target user account is a member of a privileged group (e.g., Domain Admins).
**Diagnosis Steps:**
1. Verify the engineer is a member of `Global-Helpdesk`.
2. Open ADUC -> Right-click the target user's OU -> **Properties** -> **Security** -> **Advanced**.
   - Confirm `Global-Helpdesk` has **Reset Password** and **Write lockoutTime** permissions, which is correct.
3. Check the target user account properties. The user is a member of **Domain Admins**.
   - Active Directory runs the **AdminSDHolder** thread every 60 minutes, which automatically strips custom delegation permissions from accounts in privileged groups, replacing them with inheritance protection.
**Root Cause:** The target user account was a member of a privileged group (Domain Admins). AD security policy (AdminSDHolder) blocks standard helpdesk delegation on administrative accounts.
**Fix:** Do not use delegated helpdesk accounts to manage administrative accounts. Domain Admins must have their passwords reset by other Domain Admins.
**Prevention:** Educate helpdesk staff on administrative account boundaries.
**Ticket Close Note:** "Target user was identified as a Domain Admin. Explained that helpdesk delegation does not apply to privileged administrative accounts due to AdminSDHolder protection. Reset performed by Domain Admin. Closing ticket."

---

## Critical Points

> [!danger] Never Do This
> - Keep the default computer landing container (`CN=Computers`) active without redirecting it or managing it.
> - Any machine that joins the domain will sit unconfigured without security GPOs applied, presenting a security vulnerability.
> - Redirect the landing container to a secure staging OU using the `redircmp` command.

> [!warning] Common Trap
> - Attempting to delete an OU and getting "Access is denied" because of accidental deletion protection, then blindly modifying security ACL permissions to force deletion.
> - This can corrupt the directory object's DACL parameters.
> - Follow the proper steps: Enable Advanced Features, uncheck the protection flag on the Object tab, and delete the OU normally.

> [!tip] Senior Engineer Tip
> - When designing OUs, align them with **GPO Boundaries**. If a group of machines requires a different security baseline (e.g., developers need local admin rights, while sales staff do not), place them in separate sub-OUs rather than trying to filter GPOs using complex WMI queries.

> [!success] Verification Steps
> - Run the command: `Get-ADOrganizationalUnit -Filter "Name -eq 'SalesWorkstations'"` to check properties.
> - Verify that the **ProtectedFromAccidentalDeletion** status reads `True`.

> [!question] Interview Alert
> - "What happens when you delete an OU containing objects while accidental deletion protection is active?"
> - Answer: "Active Directory blocks the deletion and returns an 'Access is denied' error. To delete it, I must enable Advanced Features in ADUC, open the OU's properties, navigate to the Object tab, uncheck 'Protect object from accidental deletion', and then delete the OU. This deletes the child objects if I confirm the delete subtree prompt."

---

## Common Mistakes & Fixes
| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| Deleting OUs accidentally | Deletion protection disabled during creation | Always check the box **Protect container from accidental deletion** when creating OUs. |
| Linking GPOs to default containers | Confusing containers with OUs | Only link GPOs to Organizational Units (OUs), as default containers (`CN=Computers`, `CN=Users`) do not support GPO links. |
| Deeply nested OU structures | Attempting to replicate org charts | Keep OU structures flat (under 3-4 levels) to optimize GPO processing and search times. |

---

## Lab Exercise

**Objective:** Create an OU structure, enable accidental deletion protection, delegate password reset permissions to a helpdesk group, and verify configuration.
**Time Required:** 30 minutes
**Environment Needed:** A Windows Server 2022 VM (Domain Controller).
**Pre-requisites:** Active Directory Domain Services role active.

**Steps:**
1. Open PowerShell as Administrator on the DC.
2. Create the Helpdesk Security Group:
   ```powershell
   New-ADGroup -Name "G-Helpdesk-Staff" -GroupScope Global -GroupCategory Security -Path "CN=Users,DC=lab,DC=local"
   ```
3. Create the target OU with deletion protection enabled:
   ```powershell
   New-ADOrganizationalUnit -Name "CorpEmployees" -Path "DC=lab,DC=local" -ProtectedFromAccidentalDeletion $true
   ```
4. Delegate Password Reset Rights: Open ADUC (`dsa.msc`).
5. Right-click `CorpEmployees` OU -> Select **Delegate Control** -> Click **Next**.
6. Add group `G-Helpdesk-Staff` -> Click **Next**.
7. Select **Reset user passwords and force password change at next logon** -> Click **Next** -> Click **Finish**.
8. Verify Accidental Deletion Protection: Try to delete the OU via PowerShell:
   ```powershell
   Remove-ADOrganizationalUnit -Identity "OU=CorpEmployees,DC=lab,DC=local" -Confirm:$false
   ```
   - Expected output: `Access is denied` error.
9. Disable protection and delete:
   ```powershell
   Set-ADOrganizationalUnit -Identity "OU=CorpEmployees,DC=lab,DC=local" -ProtectedFromAccidentalDeletion $false
   Remove-ADOrganizationalUnit -Identity "OU=CorpEmployees,DC=lab,DC=local" -Confirm:$false
   ```
   - Expected output: Command completes successfully.

**Success Criteria:** The OU is protected from accidental deletion, delegates password reset rights, and is deleted only after the protection flag is removed.
**Common Failures:** Delegation fails if the helpdesk group name is misspelled.

---

## Interview Questions & Answers

### Basic (L1 Level)
**Q: What is an Organizational Unit (OU) and how does it differ from a default container?**
A: An OU is a logical container in Active Directory used to organize users, computers, and groups. It differs from default containers (like `CN=Computers`) because OUs support Group Policy Object (GPO) links and administrative delegation, while default containers do not.

**Q: How do you protect an OU from accidental deletion?**
A: When creating an OU, I check the box **Protect container from accidental deletion**. If the OU already exists, I enable Advanced Features in ADUC, open the OU's properties, navigate to the Object tab, and check the protection box.

### Intermediate (L2 Level)
**Q: Explain how Delegation of Control works on an OU.**
A: Delegation of Control allows domain administrators to assign specific permissions (e.g., resetting passwords, modifying group memberships, or joining computers to the domain) over objects within a specific OU to a user or security group. This allows delegation of tasks to helpdesk staff without granting them full domain administrator privileges.

### Advanced (L3/Senior Level)
**Q: A helpdesk engineer reports they delegated password reset permissions on a parent OU, but the settings are not applying to users in child OUs. Explain your diagnostic and resolution strategy.**
A:
- **Situation**: Delegated permissions on a parent OU failed to apply to child OUs.
- **Task**: I needed to identify why permission inheritance was blocked.
- **Action**: I opened ADUC, enabled Advanced Features, right-clicked the child OU, selected **Properties** -> **Security** -> **Advanced**. I observed that the checkbox **Enable inheritance** was unchecked on the child OU, blocking permissions from the parent. I enabled inheritance on the child OU.
- **Result**: The delegated permissions successfully propagated down the OU tree.

### HR / Behavioral
**Q: Tell me about a time you had to restructure an Active Directory OU layout. How did you plan the migration?**
A: We had to migrate from a geographic to a functional tiering OU layout to simplify GPO management. I mapped our active group policies, created the new OU hierarchy, moved objects in batches during maintenance windows, and verified policy applications, completing the migration with zero user impact.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **What**: The logical containers in Active Directory used to organize directory objects and define GPO and delegation boundaries.
> **Why**: Critical for organizing resources, delegating task management, and target policy applications.
> **How**: Create OUs with accidental deletion protection, delegate specific tasks using the wizard, and link target GPOs.
> **Command**: `New-ADOrganizationalUnit`
> **Interview Answer Starter**: "In my experience, OU design must focus on GPO linking requirements and administrative delegation scopes to maintain a clean structure..."

**Key Numbers to Remember:**
- Default computer landing container: `CN=Computers`
- Max nesting depth recommendation: Under 3-4 levels
- AdminSDHolder execution interval: 60 minutes

**3 Things Interviewer Wants to Hear:**
1. Using OUs as boundaries for GPO linking and administrative delegation
2. Enabling Accidental Deletion Protection on all OUs
3. Troubleshooting delegation failures caused by blocked inheritance or AdminSDHolder policies

---

## Related Notes
- [[03-Identity-and-Core-Services/06-Active-Directory/Users-and-Groups|Users and Groups]] — Details objects stored inside OUs.
- [[03-Identity-and-Core-Services/06-Active-Directory/Group-Policy|Group Policy]] — Details policies linked to OUs.
- [[03-Identity-and-Core-Services/06-Active-Directory/Domain-Join|Domain Join]] — Details computer placement in target OUs.

---

## Tags
#desktop-support #active-directory #organizational-units #L1 #interview-topic #lab-complete #daily-use

