---
tags: [sysadmin, windows-server, active-directory, management]
difficulty: Intermediate
lab-required: Yes
read-time: 15 mins
---

# WS-06: Active Directory — Users, Groups, OUs

> [!abstract] Overview
> This note covers Active Directory administration, including OU layout design, object properties, bulk deployment scripting, Group Scope resolution, security auditing, and privilege delegation.

---
## Concept
Think of Active Directory objects like the structural organization of a high-security office building:
- **OUs** are the physical department rooms (Sales, Finance) labeled on the doors.
- **Users** are the individual employees who walk around the building.
- **Groups** are the colored security access cards:
  - **Global Groups** represent your job title (e.g., "Accounts Clerk card").
  - **Domain Local Groups** represent the electronic locks on specific file drawers (e.g., "Read-Write Lock on Finance Files").
  - You assign employees their job cards (Global Group), slot the job cards into the locks (Domain Local Group), and access is granted. This is the **AGDLP** rule.

*Seedha simple mein: AD OUs container hote hain jahan objects store hote hain. AD groups do type ke hote hain: Security (permissions ke liye) aur Distribution (email ke liye). AGDLP rule permissions manage karne ka best practice standards hai.*

---
## Technical Deep Dive

### 1. OU Design Best Practices
Organizational Units (OUs) should be designed to support **Group Policy application** and **Administrative Delegation**, not just replicate the company organizational chart.
- **Standard Structure (Hierarchical):**
  - **Root OU:** Corporate Name (e.g., `Corporate_Root`)
    - **Sub-OUs:**
      - `Admin_Accounts` (Isolated accounts with elevated rights)
      - `Workstations` (Member computers)
      - `Servers` (Internal server objects)
      - `Departments` (OUs for specific business units: `Sales`, `HR`, `Finance` containing their respective users and computers).

### 2. User Account Properties Explained
- **Member Of Tab:** Lists the groups the user belongs to. Primary Group defaults to Domain Users.
- **Logon Hours:** Restricts the specific hours and days the user can log into domain workstations (e.g., block logons outside 8 AM - 6 PM).
- **Log On To:** Restricts the user to authenticate only on specific workstation names.
- **User Principal Name (UPN):** The user's domain login format: `username@company.local` (or UPN suffix `company.com`).
- **sAMAccountName:** The legacy pre-Windows 2000 login format: `COMPANY\username`.

### 3. Group Types and Scopes

#### Group Types
- **Security Groups:** Used to assign permissions to resources (shares, folder security). Can also be used as distribution lists.
- **Distribution Groups:** Used exclusively for email distribution lists (Exchange). Cannot be used to assign permissions because they lack a Security Identifier (SID).

#### Group Scopes (The AGDLP Rule)
- **Domain Local (DL):** Used to assign permissions to resources. Can contain objects from any trusted domain, but only grants access to resources inside its own domain.
- **Global (G):** Used to group users with similar organizational roles. Can only contain objects from its own domain, but can be granted access to resources in any domain.
- **Universal (U):** Used to consolidate groups across multiple domains. Stored in the Global Catalog.
- **AGDLP Rule:** **A**ccounts go into **G**lobal Groups, which are nested into **D**omain **L**ocal Groups, which are assigned **P**ermissions on the resource.

### 4. Account Lockout Policies
Protects against brute-force attacks:
- **Account Lockout Threshold:** Number of failed login attempts before lockout occurs (e.g., 5 attempts).
- **Account Lockout Duration:** Time the account remains locked before auto-unlocking (e.g., 30 minutes).
- **Reset Account Lockout Counter After:** Time window between failed attempts before the counter resets (e.g., 15 minutes).

### 5. Delegation of Control
Delegation allows administrators to grant specific permissions (e.g., resetting passwords, modifying group memberships) to non-admin users (like a Helpdesk technician) on a specific OU without granting them full Domain Admin rights.

---
## Lab — Step by Step
> [!info] Lab Setup Needed
> A Domain Controller (`SVR-DC01`) with Active Directory Users and Computers (ADUC) installed.

### Step 1: Create an OU Structure
1. Open Server Manager -> Tools -> **Active Directory Users and Computers**.
2. Right-click the domain `company.local` -> **New** -> **Organizational Unit**.
3. Name: `Corp_Root`. Check **Protect container from accidental deletion** (Standard safety baseline). Click OK.
4. Right-click `Corp_Root` -> **New** -> **Organizational Unit**. Create sub-OUs: `Corp_Users` and `Corp_Computers`.

### Step 2: Create Users via PowerShell (Bulk Import)
1. Open PowerShell ISE as Administrator on the DC.
2. Run the following script to bulk create 5 users from a memory array (or CSV structure):
   ```powershell
   # Import AD module
   Import-Module ActiveDirectory

   # Define user array
   $Users = @(
       [PSCustomObject]@{Sam="jdoe"; First="John"; Last="Doe"; OU="OU=Corp_Users,OU=Corp_Root,DC=company,DC=local"}
       [PSCustomObject]@{Sam="asmith"; First="Alice"; Last="Smith"; OU="OU=Corp_Users,OU=Corp_Root,DC=company,DC=local"}
       [PSCustomObject]@{Sam="bjones"; First="Bob"; Last="Jones"; OU="OU=Corp_Users,OU=Corp_Root,DC=company,DC=local"}
   )

   # Default password config
   $SecPassword = ConvertTo-SecureString "UserP@ssword123!" -AsPlainText -Force

   foreach ($User in $Users) {
       New-ADUser -Name "$($User.First) $($User.Last)" `
                  -SamAccountName $User.Sam `
                  -UserPrincipalName "$($User.Sam)@company.local" `
                  -GivenName $User.First `
                  -Surname $User.Last `
                  -Path $User.OU `
                  -AccountPassword $SecPassword `
                  -Enabled $true `
                  -ChangePasswordAtLogon $true
   }
   Write-Output "Bulk creation complete."
   ```
3. Refresh ADUC. Verify that the users `jdoe`, `asmith`, and `bjones` are created inside the `Corp_Users` OU.

### Step 3: Delegate Helpdesk Password Reset Rights
1. Create a security group named `Helpdesk_Staff` in ADUC. Add a test user to this group.
2. Right-click the OU `Corp_Users` and select **Delegate Control**. Click Next.
3. Add the group `Helpdesk_Staff`. Click Next.
4. Tasks to Delegate: Check **Reset user passwords and force password change at next logon**. Click Next, then **Finish**.
5. Test: Log into a workstation using a Helpdesk user account. Verify that they can reset passwords for users inside `Corp_Users`, but receive access denied if attempting to reset Domain Admin passwords.

---
## Commands and Diagnostics Reference
Advanced content only — basics in [[Basic AD user management]]

Execute dynamic account query and unlocking scripts:

```powershell
# Windows PowerShell
# Find all locked-out accounts in the domain
Search-ADAccount -LockedOut | Select-Object Name, SamAccountName

# Unlock a specific user account
Unlock-ADAccount -Identity "jdoe"

# Force password change on next login for a user
Set-ADUser -Identity "jdoe" -ChangePasswordAtLogon $true

# Get detailed group membership listings for a user
Get-ADPrincipalGroupMembership -Identity "jdoe" | Select-Object Name, GroupScope, GroupCategory
```

---
## Troubleshooting Scenarios

**Scenario 1:**
- **Problem:** A Helpdesk tech delegates control to reset passwords on the `Corp_Users` OU. However, when they attempt to reset the password for a manager whose account resides in that OU, they receive "Access Denied."
- **Root Cause:** The manager user object is a member of a protected administrative group (e.g., Domain Admins). Active Directory runs a background service called **AdminSDHolder** every 60 minutes, which resets ACL permissions on all protected accounts, stripping custom delegated OU permissions to secure administrative integrity.
- **Fix:**
  1. Open ADUC. Verify if the manager belongs to Domain Admins or Account Operators.
  2. If they do not require administrative rights, remove them from the protected group.
  3. If they do require administrative rights, they must be managed exclusively by Domain Administrators; delegation at the OU level cannot override AdminSDHolder protections.

**Scenario 2:**
- **Problem:** An administrator tries to delete an empty OU named `Old_Computers` in ADUC, but receives error: "You do not have sufficient privileges to delete Old_Computers, or this object is protected from accidental deletion."
- **Root Cause:** The "Protect container from accidental deletion" flag is active on the OU, blocking L3/L4 system deletes.
- **Fix:**
  1. In ADUC, click **View** -> **Advanced Features**.
  2. Locate the OU `Old_Computers`. Right-click it and select **Properties**.
  3. Go to the **Object** tab.
  4. Uncheck **Protect object from accidental deletion**. Click Apply and OK.
  5. Right-click the OU and click **Delete**; it will now delete successfully.

---
## Common Mistakes
> [!warning] Avoid These
> **Using Domain Admins for daily non-admin work:** Adding IT support staff accounts directly into the `Domain Admins` group to allow them to reset user passwords or join computers to the domain. This exposes the entire forest to credential harvesting if their workstation is compromised.
> **Correct approach:** Use Delegation of Control to grant minimal permissions (least privilege) on specific OUs for daily administrative tasks, keeping Domain Admin group membership strictly limited.

---
## Pro Tips
> [!tip] Field Experience
> When designing naming conventions for sAMAccountNames, always plan for collisions. If using `firstinitial+lastname` (e.g., `jsmith` for John Smith), you will eventually encounter a Jane Smith. Standardize on unique identifiers or use middle initials (e.g., `jasmith`) in your creation scripts.

---
## Quick Revision Table
| # | Concept | One Line Summary |
|---|---------|-----------------|
| 1 | AGDLP | Accounts (Users) -> Global (Job Role) -> Domain Local (Access Lock) -> Permissions. |
| 2 | Security Group | Group type containing a SID, used to assign resource permissions and rights. |
| 3 | Distribution Group| Email distribution group; lacks a SID, cannot be used for ACL permissions. |
| 4 | AdminSDHolder | Security process that automatically resets ACL overrides on protected admin objects. |
| 5 | Delegation | Granting restricted administrative rights on an OU scope to specific non-admin groups. |

---
## Interview Q&A

**Q1: Explain the AGDLP nesting rule and why it is considered best practice in Active Directory design.**
A: **AGDLP** stands for: **A**ccounts go into **G**lobal groups, nested into **D**omain **L**ocal groups, which are assigned **P**ermissions on a resource. 
- Accounts represent users.
- Global Groups represent their job role (e.g., `G_Finance_Clerks`).
- Domain Local Groups represent access to a specific resource (e.g., `DL_Finance_Share_ReadWrite`).
- Permissions are assigned on the folder/share to the Domain Local group.
This is best practice because it separates user management from resource permission management. If a finance clerk leaves, you simply remove them from the Global Group. If permissions on the resource change, you modify the Domain Local Group. You never need to modify individual ACLs on folders, reducing administrative overhead.

**Q2: A user account is locked out repeatedly. Walk through how you would locate the source of the lockout.**
A: 
- **Situation:** A domain user account is experiencing continuous lockouts, indicating a stale credential loop.
- **Task:** Trace the authentication traffic to identify the source device generating the failed attempts.
- **Action:** First, I will run the Microsoft **LockoutStatus.exe** tool or query the domain controllers' security event logs (specifically event ID `4740` - "A user account was locked out"). Second, I will look at the event details to find the "Caller Computer Name." This tells me which workstation sent the bad credentials. Third, I will check that workstation for cached credentials, such as a mapped network drive with old passwords, a stale Outlook credential, or a running service/scheduled task running under that user account.
- **Result:** Clearing the cached credential or updating the scheduled task password on the client workstation stops the lockouts.

**Q3: What is the difference between a Global Group and a Domain Local Group scope?**
A: A **Global Group** can only contain objects (users, computers) from its own domain, but can be added to resources and other groups in any trusted domain in the forest. A **Domain Local Group** can contain objects from any trusted domain (including global groups and users from other domains), but it can only be granted permissions to resources located inside its own domain.

---
## Related Notes
- [[03-Identity-and-Core-Services/06-Active-Directory/WS-02 Active Directory Domain Services|WS-02 Active Directory Domain Services]] — Logical structures and NTDS layouts.
- [[03-Identity-and-Core-Services/06-Active-Directory/WS-05 Group Policy — Complete Guide|WS-05 Group Policy — Complete Guide]] — Linking policies to OU scopes.
- [[05-Automation-and-Ticketing/10-Scripting-PowerShell/PS-02 PowerShell for Active Directory|PS-02 PowerShell for Active Directory]] — Automating object management via scripting.

