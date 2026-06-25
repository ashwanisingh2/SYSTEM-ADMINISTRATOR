---
tags: [desktop-support, active-directory, users-groups, L1]
aliases: [ad-accounts, ad-security-groups]
created: 2026-06-25
status: #complete
difficulty: #beginner
cert-relevant: #md-102
---

# Users and Groups

---

## Concept Overview
- **What it is**: Active Directory Users and Groups are logical directory database objects used to represent human identities (users) and collect them into management units (groups) for security access control and email distribution.
- **Why it matters for a support engineer**: A support engineer must know how to manage accounts, unlock locked users, reset passwords, change group memberships, and automate account creations.
- **Where you encounter this in real job**: Resetting user passwords, unlocking accounts (Event ID 4740), adding users to security groups, creating bulk accounts using PowerShell, and auditing group memberships.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Performs password resets, unlocks user accounts, updates user contact details, and adds members to existing groups.
  - **L2**: Creates bulk user accounts using scripts, manages group nesting configurations, and configures account delegation settings.
  - **L3**: Designs security group scope hierarchies (AGDLP model), implements Privileged Identity Management (PIM) policies, and audits access logs.

---

## Technical Deep Dive

### 1. Security Groups vs. Distribution Groups
- **Security Groups**: Used to assign access permissions to shared resources (e.g., file shares, printers). They can also be used as email distribution lists.
- **Distribution Groups**: Used exclusively for email distribution lists (e.g., Exchange distribution groups). They cannot be assigned access permissions because they lack a Security Identifier (SID).

### 2. Group Scopes
Group scopes determine where in the forest the group can be assigned permissions and what members it can contain:
- **Domain Local**: Can contain members from any domain in the forest. Can only be assigned permissions to resources within the *local domain* where the group resides.
- **Global**: Can only contain members from the *local domain* where the group resides. Can be assigned permissions to resources in *any domain* in the forest.
- **Universal**: Can contain members from any domain in the forest. Can be assigned permissions to resources in *any domain* in the forest. Membership data is cached in the Global Catalog, which requires replication overhead.

### 3. The AGDLP Nesting Model
Microsoft recommends the **AGDLP** design model to manage permissions at scale:
- **A (Accounts)**: Place user accounts into...
- **G (Global groups)**: ...representing department roles (e.g., `Sales-Dept`).
- **DL (Domain Local groups)**: Nest the global groups into domain local groups representing resource access (e.g., `DL-Share-ReadOnly`).
- **P (Permissions)**: Assign resource permissions directly to the Domain Local group.

```
[ User Account ] ---> [ Global Group (Sales-Dept) ] ---> [ Domain Local (DL-Share-RO) ] ---> [ File Share NTFS Permissions ]
```

---

## Commands & Syntax

### PowerShell
```powershell
# Reset a user's Active Directory password and force a change at next login
Set-ADAccountPassword -Identity "jdoe" -NewPassword (Read-Host -AsSecureString "Enter Password") -Reset $true

# Retrieve all disabled Active Directory user accounts in a specific OU
Get-ADUser -Filter "Enabled -eq `$false" -SearchBase "OU=Employees,DC=lab,DC=local" |
    Select-Object Name, SamAccountName, UserPrincipalName
```

### CMD / Run Box
```cmd
REM Launch the Active Directory Users and Computers GUI console
dsa.msc
REM Query current domain account details for a specific user from cmd
net user jdoe /domain
```

### GUI Path
> Server Manager -> Tools -> **Active Directory Users and Computers** (ADUC)
> Or: Start -> search `Active Directory Administrative Center` (ADAC)

### Important Registry Paths
```
HKLM\SYSTEM\CurrentControlSet\Services\NTDS\Parameters\
HKLM\SYSTEM\CurrentControlSet\Control\Lsa\
```

### Key Event IDs
Active Directory security events are logged on Domain Controllers:

| Event ID | Meaning | Where to Check |
|----------|---------|----------------|
| 4720 | A user account was created | Security Log |
| 4724 | An attempt was made to reset an account's password | Security Log |
| 4740 | A user account was locked out (includes source computer) | Security Log |
| 4728 | A member was added to a security-enabled global group | Security Log |

---

## Real-World Scenarios

### Scenario 1: User Account Locked Out Repeatedly (Active Directory Lockout Triage)
**User Complaint:** "My password is correct, but my account keeps locking out every 10 minutes. I cannot log into my PC or open my emails."
**Your First 3 Checks:**
1. Check the user account status in the ADUC console.
2. Query the Domain Controller Security log for Event ID 4740.
3. Identify the source machine sending the incorrect credentials.
**Diagnosis Steps:**
1. Open PowerShell on a Domain Controller and check the lockout status:
   `Get-ADUser -Identity "jdoe" -Properties LockedOut, BadLogonCount`
   - Output: `LockedOut: True, BadLogonCount: 5`
2. Search the Security log for the lockout source:
   `Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4740} | Where-Object {$_.Message -match "jdoe"}`
   - Output: `A member account was locked out. Caller Computer Name: PHONE-JDOE.`
3. The lockout source is the user's mobile phone `PHONE-JDOE`.
**Root Cause:** The user recently changed their Active Directory password, but their mobile phone had cached the expired password for the corporate Wi-Fi or mail sync, sending bad credentials repeatedly and triggering the lockout threshold.
**Fix:** Unlock the user account in ADUC, and update the Wi-Fi/mail password on `PHONE-JDOE` or disconnect it from the network temporarily.
**Prevention:** Configure corporate Wi-Fi profiles to use certificate-based authentication instead of static user passwords.
**Ticket Close Note:** "Identified lockout source as PHONE-JDOE using Event ID 4740 on the DC. Updated cached credentials on the device and unlocked the AD account. Closing ticket."

### Scenario 2: Marketing Employee Cannot Access Shared Finance Drive (NTFS vs. Group Membership)
**User Complaint:** "I was recently transferred from Marketing to Finance. I can access the building, but when I try to open the shared Finance drive `F:`, I get an access denied error."
**Your First 3 Checks:**
1. Check the user's active group memberships in ADUC.
2. Check the NTFS folder permissions on the physical shared directory.
3. Verify if the client workstation has updated its security access token.
**Diagnosis Steps:**
1. Open ADUC and search the user's properties -> **Member Of** tab.
   - The group `Global-Finance` is listed, confirming group membership is updated.
2. Check the shared folder permissions:
   - The Domain Local group `DL-Finance-ReadWrite` has NTFS permissions.
   - The Global group `Global-Finance` is nested inside `DL-Finance-ReadWrite`.
3. Run `whoami /groups` on the client workstation.
   - Output: `Global-Marketing` is listed, but `Global-Finance` is missing from the active groups list.
**Root Cause:** The user had not logged off and back on since being added to the new security group, meaning their local security access token lacked the new group SID.
**Fix:** Instruct the user to log off their workstation and log back in to generate a fresh security token.
**Prevention:** Instruct the helpdesk to advise users to restart or log off their machines after group membership changes.
**Ticket Close Note:** "User logged off and logged back in to refresh their security access token. Confirmed access to the shared Finance drive is restored. Closing ticket."

---

## Critical Points

> [!danger] Never Do This
> - Copy an existing employee's Active Directory account to create a new user without auditing their group memberships.
> - Copying accounts duplicates all group memberships, which can result in the new user inheriting privileged access rights (e.g., access to HR or Finance folders) they should not have (Privilege Creep).
> - Always create a clean account or use standard role-based template accounts.

> [!warning] Common Trap
> - Attempting to nest a Domain Local group inside a Global group.
> - Active Directory group scope rules block this configuration. Global groups can only contain accounts or other global groups from the local domain.
> - Follow the AGDLP model: nest Global groups inside Domain Local groups.

> [!tip] Senior Engineer Tip
> - If a user account keeps locking out and you cannot find the source, run the Microsoft tool **LockoutStatus.exe**. It queries all Domain Controllers in the domain to find the exact DC that locked the account and displays the bad password count in real-time.

> [!success] Verification Steps
> - Run: `whoami /groups` on the client workstation.
> - Verify that the new group name is listed in the output table.

> [!question] Interview Alert
> - "Explain the AGDLP nesting model and why it is recommended."
> - Answer: "AGDLP stands for Accounts, Global Groups, Domain Local Groups, Permissions. We place user Accounts into Global groups representing roles, nest those into Domain Local groups representing resource access, and assign Permissions to the Domain Local groups. This model simplifies access management, as changing user access only requires modifying group membership, without changing folder NTFS permissions."

---

## Common Mistakes & Fixes
| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| Orphaned user accounts | Decommissioning users without disabling | Disable the user account immediately when they leave the organization, move it to a "Terminated" OU, and delete it after 30 days. |
| Exposing administrative rights | Adding standard users to Domain Admins | Use delegation of control to grant specific rights to helpdesk staff instead of adding them to admin groups. |
| Inconsistent naming conventions | Creating groups ad-hoc | Implement a naming standard (e.g., `G-Dept-Role` or `DL-Resource-Access`) to keep the directory clean. |

---

## Lab Exercise

**Objective:** Create an Organizational Unit, create a user account, create global and domain local groups, nest the groups using the AGDLP model, and verify membership.
**Time Required:** 30 minutes
**Environment Needed:** A Windows Server 2022 VM (Domain Controller).
**Pre-requisites:** Active Directory Domain Services role active.

**Steps:**
1. Open PowerShell as Administrator on the DC.
2. Create the Organizational Unit:
   ```powershell
   New-ADOrganizationalUnit -Name "LabUsers" -Path "DC=lab,DC=local"
   ```
3. Create the User Account:
   ```powershell
   $Password = ConvertTo-SecureString "Pass1234Secure!" -AsPlainText -Force
   New-ADUser -Name "Test User" -SamAccountName "tuser" -UserPrincipalName "tuser@lab.local" -Path "OU=LabUsers,DC=lab,DC=local" -AccountPassword $Password -Enabled $true -ChangePasswordAtLogon $false
   ```
4. Create the Global Group:
   ```powershell
   New-ADGroup -Name "G-Lab-Staff" -GroupScope Global -GroupCategory Security -Path "OU=LabUsers,DC=lab,DC=local"
   ```
5. Create the Domain Local Group:
   ```powershell
   New-ADGroup -Name "DL-LabShare-RW" -GroupScope DomainLocal -GroupCategory Security -Path "OU=LabUsers,DC=lab,DC=local"
   ```
6. Nest the Groups:
   - Add the user to the Global group:
     ```powershell
     Add-ADGroupMember -Identity "G-Lab-Staff" -Members "tuser"
     ```
   - Add the Global group to the Domain Local group:
     ```powershell
     Add-ADGroupMember -Identity "DL-LabShare-RW" -Members "G-Lab-Staff"
     ```
7. Verification: Run the verification command.
   ```powershell
   Get-ADGroupMember -Identity "DL-LabShare-RW" | Select-Object Name, objectClass
   ```
   - Expected output: Name `G-Lab-Staff` is listed, confirming group nesting.

**Success Criteria:** The user account is nested inside the Global group, which is nested inside the Domain Local group, matching the AGDLP design.
**Common Failures:** Command failure if group names contain spelling errors.

---

## Interview Questions & Answers

### Basic (L1 Level)
**Q: What is the difference between a security group and a distribution group?**
A: A security group is used to assign access permissions to shared resources (like folders or printers) and can also be used for email distribution. A distribution group is used exclusively for email distribution lists and cannot be assigned access permissions.

**Q: How do you unlock a user's Active Directory account?**
A: I open the Active Directory Users and Computers console, locate the user's account, double-click to open their properties, navigate to the **Account** tab, check the box **Unlock account. This account is currently locked out on this Active Directory Domain Controller**, and click Apply.

### Intermediate (L2 Level)
**Q: Explain the difference between Domain Local and Global group scopes.**
A: A Domain Local group can contain members from any domain in the forest, but can only be assigned permissions to resources within the local domain where the group resides. A Global group can only contain members from its local domain, but can be assigned permissions to resources in any domain within the forest.

### Advanced (L3/Senior Level)
**Q: A helpdesk engineer needs to be able to reset user passwords and unlock accounts, but must not have domain administrator rights. Explain how you would configure this.**
A:
- **Situation**: A helpdesk engineer required account reset rights without administrative privileges.
- **Task**: I needed to delegate specific permissions in Active Directory.
- **Action**: I opened the Active Directory Users and Computers console, right-clicked the target User OU (e.g., `OU=Employees`), and selected **Delegate Control**. In the wizard, I selected the helpdesk security group, chose the task **Reset user passwords and force password change at next logon**, and completed the delegation.
- **Result**: The helpdesk engineers successfully reset passwords and unlocked accounts within the OU without holding domain admin privileges.

### HR / Behavioral
**Q: Tell me about a time you audited user access rights and identified a security risk. How did you resolve it?**
A: During an access audit, I found that several terminated employees still had active AD accounts. I immediately disabled the accounts, updated the HR termination workflow, and implemented automated PowerShell scripts to disable inactive accounts after 30 days.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **What**: The Active Directory objects representing human identities (users) and permission containers (groups).
> **Why**: Critical for managing network access, resetting passwords, and securing corporate resources.
> **How**: Use ADUC, ADAC, or PowerShell to manage accounts, and implement the AGDLP nesting model.
> **Command**: `Set-ADAccountPassword`
> **Interview Answer Starter**: "In my experience, user and group administration requires following the AGDLP nesting model to simplify permission structures..."

**Key Numbers to Remember:**
- Default account lockout event ID: 4740
- Active Directory default group type: Security Group
- Maximum dynamic group nesting levels: No hard limit (recommend under 3 for performance)

**3 Things Interviewer Wants to Hear:**
1. Implementing the AGDLP nesting model for resource access
2. Identifying account lockout sources using Event ID 4740
3. Delegating OU permissions to prevent domain admin sprawl

---

## Related Notes
- [[02-Operating-Systems/03-Windows-OS/User-Profiles|User Profiles]] — Details client-side profile directories.
- [[03-Identity-and-Core-Services/06-Active-Directory/Organizational-Units|Organizational Units]] — Details OU containers and delegation.
- [[03-Identity-and-Core-Services/06-Active-Directory/Group-Policy|Group Policy]] — Details GPO deployments to users and computers.

---

## Tags
#desktop-support #active-directory #users-groups #L1 #interview-topic #lab-complete #daily-use

