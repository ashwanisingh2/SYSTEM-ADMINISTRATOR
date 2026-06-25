---
tags: [desktop-support, powershell, active-directory, administration, L2]
aliases: [ad-powershell, get-aduser, bulk-provisioning]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #md-102
---

# PowerShell for Active Directory

---

## Concept Overview
- **What it is**: PowerShell for Active Directory utilizes the dedicated **ActiveDirectory** module to interface with Active Directory Domain Services (AD DS). It communicates with Domain Controllers (DCs) using Active Directory Web Services (ADWS) on TCP port 9389, allowing administrators to query, create, modify, and audit directory objects.
- **Why it matters for a support engineer**: Managing hundreds of user accounts, groups, and computers manually via the GUI (ADUC) is slow and prone to errors. Support engineers use the AD module to automate new hire provisioning, check lockout origins, audit stale computer accounts, and run security compliance reports.
- **Where you encounter this in real job**: Automating bulk user creation from HR spreadsheets, finding locked out accounts, auditing inactive machines, and modifying group memberships recursively.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Queries user properties (`Get-ADUser`), unlocks accounts, and adds users to security groups.
  - **L2**: Automates provisioning scripts, identifies stale accounts, moves objects between OUs, and cleans up group memberships.
  - **L3**: Script-automates forest-wide security audits, manages AD delegation permissions via code, configures fine-grained password policies (PSOs), and designs metadata cleanup scripts.

---

## Technical Deep Dive

### 1. Active Directory Module & AD Web Services (ADWS)
- **Module Source**: Part of the **Remote Server Administration Tools (RSAT)**. It requires the AD DS Tools feature enabled on the workstation.
- **Communication Protocol**: The module queries Domain Controllers using **Active Directory Web Services (ADWS)** on TCP Port `9389`. If ADWS is unavailable (on older Server 2008 DCs), it falls back to standard LDAP protocols.

### 2. Query Filtering: `-Filter` vs. `-LDAPFilter`
When querying large AD databases, filtering data on the Domain Controller is crucial to save network bandwidth:
- **`-Filter`**: Uses PowerShell-style syntax operators (e.g., `-eq`, `-like`, `-not`).
  - *Example*: `Get-ADUser -Filter "Department -eq 'Sales'"`
- **`-LDAPFilter`**: Uses RFC-compliant LDAP query strings (enclosed in parentheses). Faster for advanced directory audits.
  - *Example*: `Get-ADUser -LDAPFilter "(department=Sales)"`
- **`-SearchBase`**: Limits the query scope to a specific Organizational Unit (OU) path, preventing time-consuming domain-wide searches.

### 3. Bulk Management via CSV
To automate provisioning, support engineers use CSV files containing user parameters (e.g., Firstname, Lastname, Title, Department) and feed them into a loop:

```
[CSV Data File] ---> [Import-Csv] ---> [Foreach Loop] ---> [New-ADUser] ---> [Add-ADGroupMember]
```

---

## Commands & Syntax

### PowerShell
Ensure the `ActiveDirectory` module is installed and loaded.
```powershell
# Import the Active Directory module
Import-Module ActiveDirectory

# Query a user by SamAccountName, pulling properties not returned by default (Email, PasswordExpired)
Get-ADUser -Identity "jdoe" -Properties EmailAddress, PasswordExpired | Select-Object Name, UserPrincipalName, EmailAddress, PasswordExpired

# Find all locked out accounts in the domain (Search-ADAccount is a dedicated audit cmdlet)
Search-ADAccount -LockedOut | Select-Object Name, SamAccountName, DistinguishedName

# Query all users in a specific Sales OU using SearchBase
Get-ADUser -Filter * -SearchBase "OU=Sales,OU=HQ,DC=company,DC=local"

# Find computer accounts that have not logged on for the last 90 days (Stale Computers)
$90DaysAgo = (Get-Date).AddDays(-90)
Get-ADComputer -Filter "LastLogonDate -lt '$90DaysAgo'" -Properties LastLogonDate |
    Select-Object Name, LastLogonDate, Enabled |
    Sort-Object LastLogonDate

# Add a user to a specific security group
Add-ADGroupMember -Identity "G-Marketing-Team" -Members "jdoe"

# Retrieve all members of a group, including nested groups (RecursiveMatch OID rule)
Get-ADUser -Filter "MemberOf -recursiveMatch 'CN=G-HQ-Staff,OU=Groups,DC=company,DC=local'"
```

---

## Real-World Scenarios

### Scenario 1: Provisioning 50 New Hires from an HR CSV File
**User Complaint:** HR submits a ticket containing a spreadsheet: *"We have 50 new contractors starting next Monday. Please create their Active Directory accounts, set temporary passwords, assign their departments, and enable the accounts."*
**Your First 3 Checks:**
1. Verify the CSV file headers and formatting.
2. Determine the target OU Distinguished Name where the users should be placed.
3. Check the default password policy rules (complexity/length) of the domain.
**Diagnosis Steps:**
1. Open the CSV file in Notepad. Confirm the headers are: `Firstname,Lastname,UPN,Department,Title`.
2. Target OU: `OU=Contractors,OU=HQ,DC=company,DC=local`.
3. Construct a script that:
   - Imports the CSV.
   - Generates a unique SamAccountName (e.g., `first initial + last name`).
   - Generates a secure, temporary, complex password.
   - Creates the account, sets password-change-on-logon, and enables it.
**Root Cause:** Requirement to automate bulk account provisioning to avoid manual GUI errors.
**Fix:**
Run the following script:
```powershell
$Users = Import-Csv -Path "C:\Temp\new_hires.csv"
$OU = "OU=Contractors,OU=HQ,DC=company,DC=local"

foreach ($User in $Users) {
    # Generate SamAccountName
    $Sam = ($User.Firstname[0] + $User.Lastname).ToLower()
    
    # Generate Secure Password
    $SecPass = ConvertTo-SecureString "TempPass2026!" -AsPlainText -Force
    
    # Creation Parameter HashTable
    $Params = @{
        Name = "$($User.Firstname) $($User.Lastname)"
        SamAccountName = $Sam
        UserPrincipalName = $User.UPN
        Department = $User.Department
        Title = $User.Title
        Path = $OU
        AccountPassword = $SecPass
        ChangePasswordAtLogon = $true
        Enabled = $true
        ErrorAction = "Stop"
    }
    
    try {
        New-ADUser @Params
        Write-Host "Created user: $Sam" -ForegroundColor Green
    } catch {
        Write-Warning "Failed to create user $Sam. Reason: $_"
    }
}
```
**Prevention:** Always use script validation sweeps to ensure no duplicate SamAccountNames or UPNs are created.
**Ticket Close Note:** "Executed automated bulk provisioning script. Created 50 accounts in the Contractors OU. Closed."

### Scenario 2: Auditing and Disabling Stale Computer Accounts
**User Complaint:** A security auditor requests: *"Please run a compliance scan. We need a list of all active Windows computer accounts that have not logged into the domain for over 180 days. These must be disabled and moved to the 'Stale Devices' OU."*
**Your First 3 Checks:**
1. Calculate the cutoff timestamp (180 days ago).
2. Query computer objects where the `LastLogonDate` is less than the cutoff.
3. Verify the target "Stale Devices" OU path.
**Diagnosis Steps:**
1. Set the target date:
   `$Cutoff = (Get-Date).AddDays(-180)`
2. Query AD computers. Filter for enabled machines that haven't authenticated:
   `Get-ADComputer -Filter "Enabled -eq 'True' -and LastLogonDate -lt '$Cutoff'" -Properties LastLogonDate`
3. Target destination OU: `OU=Stale Devices,DC=company,DC=local`.
4. Construct a pipeline script that finds, disables, and moves the computers.
**Root Cause:** Compliance requirement to secure stale directory computer objects.
**Fix:**
Run the following pipeline:
```powershell
$Cutoff = (Get-Date).AddDays(-180)
$StaleOU = "OU=Stale Devices,DC=company,DC=local"

# Find and process stale computers
Get-ADComputer -Filter "Enabled -eq 'True' -and LastLogonDate -lt '$Cutoff'" -Properties LastLogonDate |
    Foreach-Object {
        $Name = $_.Name
        # Disable the account
        Set-ADComputer -Identity $_.DistinguishedName -Enabled $false
        # Move to Stale OU
        Move-ADObject -Identity $_.DistinguishedName -TargetPath $StaleOU
        Write-Host "Disabled and moved computer: $Name" -ForegroundColor Yellow
    }
```
**Prevention:** Schedule this cleanup script to run monthly via Task Scheduler.
**Ticket Close Note:** "Audited computers inactive for 180+ days. Disabled and migrated 12 computer accounts to the Stale Devices OU. Closed."

---

## Critical Points

> [!danger] Never Do This
> - Never run `Get-ADUser -Filter *` in a large enterprise domain (e.g., 50,000+ users) without specifying a `-SearchBase` or restricting output limits (like `-ResultSetSize 100`).
> - Querying the entire domain forces the Domain Controller to search, compile, and transmit megabytes of LDAP database objects, which can consume 100% DC CPU, freeze Active Directory Web Services, and cause network latency for other users logging in.

> [!warning] Common Trap
> - Assuming that `LastLogon` and `LastLogonDate` (or `LastLogonTimestamp`) are the same attribute.
> - `LastLogon` is **not replicated** between Domain Controllers. It only records login on the specific DC that authenticated the session. To check domain-wide activity, use `LastLogonDate` (which maps to `LastLogonTimestamp` and replicates between DCs, though it has a default 9-to-14-day replication lag).

> [!tip] Senior Engineer Tip
> - When writing scripts that modify multiple user attributes, use **Splatting**. Instead of writing a single command that wraps across 3 lines on screen (which is hard to read and debug), place parameters in a clean HashTable (`$Params = @{...}`) and pass it to the cmdlet using `@Params`.

> [!success] Verification Steps
> - Run `Get-ADUser -Identity "username"` and verify that modified attributes (like Department, Title, or Enabled) reflect the targeted changes.
> - Check in ADUC that disabled objects display a small downward arrow, indicating they are inactive.

> [!question] Interview Alert
> - "How do you query all AD users who have been inactive for over 90 days?"
> - Answer: "I use the `Search-ADAccount` cmdlet with the `-AccountInactive` and `-TimeSpan` parameters. The syntax is:
>   `Search-ADAccount -AccountInactive -TimeSpan 90.00:00:00 -UsersOnly`
>   This cmdlet is highly optimized for finding stale directory objects compared to constructing manual filter loops."

---

## Common Mistakes & Fixes

| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| Querying the entire domain for single OU audits | Omitting the `-SearchBase` parameter | Always define the specific OU path in `-SearchBase` to limit LDAP search scopes. |
| Using the un-replicated `LastLogon` attribute | Misunderstanding replication schemas | Query `LastLogonDate` or `LastLogonTimestamp` to get accurate domain-wide user activity. |
| Hardcoding administrator credentials in scripts | Seeking to bypass login prompts | Use `Get-Credential` or run the script using secure service accounts via Task Scheduler. |

---

## Lab Exercise

**Objective:** Create a test OU, import test users from a local CSV string, add them to a security group, query locked out accounts, and perform cleanup.
**Time Required:** 30 minutes
**Environment Needed:** A Windows Server VM configured as a Domain Controller.
**Pre-requisites:** Administrative credentials.

**Steps:**
1. Open PowerShell as Administrator on your DC.
2. Create a test Organizational Unit:
   ```powershell
   New-ADOrganizationalUnit -Name "Lab-Testing" -Path "DC=lab,DC=local"
   ```
3. Simulate an HR CSV string:
   ```powershell
   $CSVData = @"
   Firstname,Lastname,UPN,Department
   Albert,Einstein,aeinstein@lab.local,Research
   Marie,Curie,mcurie@lab.local,Research
   "@
   $CSVData | Out-File -FilePath "C:\Windows\Temp\lab_users.csv"
   ```
4. Create a test security group inside the new OU:
   ```powershell
   New-ADGroup -Name "G-Lab-Researchers" -GroupScope Global -GroupCategory Security -Path "OU=Lab-Testing,DC=lab,DC=local"
   ```
5. Import users from the CSV and add them to the group:
   ```powershell
   $Users = Import-Csv -Path "C:\Windows\Temp\lab_users.csv"
   foreach ($User in $Users) {
       $Sam = ($User.Firstname[0] + $User.Lastname).ToLower()
       New-ADUser -Name "$($User.Firstname) $($User.Lastname)" -SamAccountName $Sam -UserPrincipalName $User.UPN -Department $User.Department -Path "OU=Lab-Testing,DC=lab,DC=local" -Enabled $true
       Add-ADGroupMember -Identity "G-Lab-Researchers" -Members $Sam
   }
   ```
6. Verification: Query the members of the security group:
   ```powershell
   Get-ADGroupMember -Identity "G-Lab-Researchers" | Select-Object Name, SamAccountName
   ```
   - Confirm Albert and Marie are listed.

**Success Criteria:** The test OU, group, and users are created, group membership assigned, and verified via PowerShell.
**Common Failures:** The command fails if the Active Directory PowerShell module is not imported or domain paths are incorrect.

---

## Interview Questions & Answers

### Basic (L1 Level)
**Q: Which PowerShell cmdlet do you run to unlock a user's account?**
A: I run the command: `Unlock-ADAccount -Identity "username"`. I can verify if it's locked first by running `Get-ADUser -Identity "username" -Properties LockedOut`.

**Q: What is RSAT and why is it required to manage Active Directory via PowerShell?**
A: RSAT stands for Remote Server Administration Tools. It contains the Active Directory PowerShell module. Without RSAT installed on my Windows client PC, the shell will not recognize AD cmdlets like `Get-ADUser` or `Set-ADUser`.

### Intermediate (L2 Level)
**Q: What is the difference between filtering users using the '-Filter' parameter versus piping to 'Where-Object'?**
A: Using the `-Filter` parameter performs the filtering directly on the Domain Controller (Server-side filtering), sending only matching users over the network. Piping to `Where-Object` retrieves all users from the DC first and filters them locally (Client-side filtering), which is slow and consumes network bandwidth. Server-side filtering is the best practice.

**Q: Why should you use the 'LastLogonDate' attribute instead of 'LastLogon' when auditing old user accounts?**
A: The `LastLogon` attribute is not replicated between Domain Controllers. If you query it, you only get the logon time from the specific DC you are connected to. `LastLogonDate` (which is a human-readable conversion of `LastLogonTimestamp`) is replicated across all DCs, providing an accurate, domain-wide view of the user's last activity.

### Advanced (L3/Senior Level)
**Q: You are writing a user termination script. You need to disable the user, remove them from all security groups (to revoke access), move them to a 'Disabled Users' OU, and set their description. How do you handle this?**
A:
- **Situation**: Automating user offboarding workflows in Active Directory.
- **Task**: Securely disable access, strip group memberships, and relocate the object.
- **Action**: First, I retrieve the user's details and check their group memberships. Since we cannot remove their primary group (usually Domain Users), I filter it out. I run a loop to strip other memberships using `Remove-ADGroupMember`. I disable the account, change the description to include the date, and move the object to the target OU:
  ```powershell
  Set-ADUser -Identity $User -Enabled $false -Description "Disabled by script $(Get-Date)"
  $Groups = (Get-ADUser $User -Properties MemberOf).MemberOf
  foreach ($Group in $Groups) {
      Remove-ADGroupMember -Identity $Group -Members $User -Confirm:$false
  }
  Move-ADObject -Identity (Get-ADUser $User).DistinguishedName -TargetPath "OU=Disabled Users,DC=corp,DC=local"
  ```
- **Result**: The user account is disabled, stripped of privileges, and moved to the archive folder automatically.

### HR / Behavioral
**Q: Tell me about a time you made a mistake while running an automation script. What happened and how did you resolve it?**
A: I was running a script to update the department attribute for our marketing users. I made a typo in the filter, running `Set-ADUser` against the entire domain instead of just the Marketing OU. It started updating all users to "Marketing". I aborted the script instantly (`Ctrl+C`). I looked at the script log. I had modified 15 accounts. I used our active backup system to pull the previous night's AD database state, retrieved the original department values for those 15 users, and wrote a quick script to restore them. I reported the incident to my manager, and now I always test scripts using a `-WhatIf` flag first.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **What**: The PowerShell module used to manage Active Directory objects (users, groups, OUs, computers).
> **Why**: Automates bulk provisioning, audits stale accounts, and manages security group configurations.
> **How**: Import the `ActiveDirectory` module, use server-side filters, specify search bases, and use splatting for parameters.
> **Command**: `Get-ADUser` / `Search-ADAccount` / `Unlock-ADAccount`
> **Interview Answer Starter**: "To manage Active Directory resources, I leverage the AD PowerShell module, ensuring queries utilize server-side filters and SearchBase scopes to optimize performance..."

**Key Numbers to Remember:**
- AD Web Services port: TCP 9389
- Default replication lag for LastLogonTimestamp: 9 - 14 days
- Output limit parameter: `-ResultSetSize`
- Wildcard character: `*`

**3 Things Interviewer Wants to Hear:**
- Server-side filtering (`-Filter`) is faster than client-side filtering (`Where-Object`)
- `LastLogon` is un-replicated; always use `LastLogonDate` for stale account audits
- Using `-WhatIf` to test destructive AD scripts before running them

---

## Related Notes
- [[03-Identity-and-Core-Services/06-Active-Directory/Users-and-Groups|AD Users and Groups]] — Core directory objects managed via cmdlets.
- [[03-Identity-and-Core-Services/06-Active-Directory/Organizational-Units|AD OUs]] — The target locations configured by `-SearchBase`.
- [[05-Automation-and-Ticketing/10-Scripting-PowerShell/PowerShell-Fundamentals|PowerShell Fundamentals]] — Outlines the core pipeline and object mechanisms.

---

## Tags
#desktop-support #powershell #active-directory #administration #L2 #interview-topic #lab-complete #daily-use

