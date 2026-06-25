---
tags: [sysadmin, powershell, active-directory, ad-module]
difficulty: Intermediate
lab-required: Yes
read-time: 15 mins
---

# PS-02: PowerShell for Active Directory

> [!abstract] Overview
> This note covers Active Directory administration using the ActiveDirectory PowerShell module. It details user/group/computer management cmdlets, pipeline filtering, and includes a production-ready bulk user creation script.

---
## Concept
Think of managing Active Directory via the GUI (ADUC) like using a manual cash register. If you need to check out 100 customers (create 100 users, reset their passwords, and add them to their groups), you have to type, click, open tabs, and close wizards 100 times, which takes hours.

PowerShell is an automated bar-code scanner and checkout lane. You feed it a spreadsheet (CSV list), write a single loop cmdlet, and in under 10 seconds, the scanner reads all lines, creates the users, generates random passwords, maps their departments to matching OUs, and sends them verification emails automatically.

*Seedha simple mein: ActiveDirectory module install karne ke baad hum direct command line se user, computer, and OU objects manage karte hain. `Get-ADUser` search filters and `ForEach-Object` bulk automation processes ko lead karte hain.*

---
## Technical Deep Dive

### 1. The Active Directory Module Installation
Before you can run Active Directory cmdlets, you must install the RSAT (Remote Server Administration Tools) AD module:
- **On Windows Server:** Installed automatically when promoting to a Domain Controller. On member servers, run:
  `Install-WindowsFeature RSAT-AD-PowerShell`
- **On Windows Client (Windows 10/11):** Enable via Optional Features:
  `Add-WindowsCapability -Online -Name "Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0"`

### 2. User Management Cmdlets (Reference)
Advanced content only — basics in [[Basic AD user management]]

Core Active Directory administration cmdlets:

- **`Get-ADUser`:** Queries user accounts.
  - *Filtering:* By default, `Get-ADUser` only returns 10 basic properties. Use `-Properties *` or specify attributes:
    `Get-ADUser -Filter "Department -eq 'Sales'" -Properties EmailAddress, Title`
- **`New-ADUser`:** Creates a user.
- **`Set-ADUser`:** Modifies user attributes.
- **`Remove-ADUser`:** Deletes user accounts.
- **`Enable-ADAccount` / `Disable-ADAccount`:** Toggles account state.
- **`Unlock-ADAccount`:** Unlocks a locked user account.
- **`Set-ADAccountPassword`:** Changes user password. Requires password as secure string.

### 3. Group, Computer, and OU Cmdlets
- **OU Management:**
  `New-ADOrganizationalUnit -Name "Sales" -Path "OU=Corp,DC=company,DC=local"`
- **Group Management:**
  - Create: `New-ADGroup -Name "G_Sales_Staff" -GroupScope Global -GroupCategory Security`
  - Add Member: `Add-ADGroupMember -Identity "G_Sales_Staff" -Members "jdoe", "asmith"`
  - Get Members: `Get-ADGroupMember -Identity "G_Sales_Staff"`
- **Computer Management:**
  `Get-ADComputer -Filter "OperatingSystem -like '*Server 2022*'"`

---
## Production Script: Bulk User Creation from CSV

Save this script as `BulkCreateUsers.ps1`. It imports user details from a CSV file, verifies if the target OU exists, creates the accounts, sets default passwords, and locks them to force password changes on first login.

```powershell
<#
.SYNOPSIS
    Bulk imports Active Directory users from a CSV file.
.DESCRIPTION
    Input CSV structure (users.csv):
    SamAccountName,FirstName,LastName,Department,Title,OUPath
    jdoe,John,Doe,Sales,Manager,"OU=Sales,OU=Corp,DC=company,DC=local"
#>

Import-Module ActiveDirectory

$CSVPath = "C:\LabFiles\users.csv"
$DefaultPassword = "UserP@ssword123!"
$SecPassword = ConvertTo-SecureString $DefaultPassword -AsPlainText -Force

if (-not (Test-Path $CSVPath)) {
    Write-Error "CSV File not found at $CSVPath"
    exit 1
}

$Users = Import-Csv -Path $CSVPath

foreach ($User in $Users) {
    # Check if user already exists
    $ExistingUser = Get-ADUser -Filter "SamAccountName -eq '$($User.SamAccountName)'"
    if ($ExistingUser) {
        Write-Warning "User $($User.SamAccountName) already exists. Skipping."
        continue
    }

    # Verify target OU exists
    if (-not (Get-ADOrganizationalUnit -Identity $User.OUPath -ErrorAction SilentlyContinue)) {
        Write-Warning "OU path $($User.OUPath) is invalid. Creating user in default Users container."
        $TargetOU = "CN=Users,DC=company,DC=local"
    } else {
        $TargetOU = $User.OUPath
    }

    # Create the AD User Object
    try {
        New-ADUser -Name "$($User.FirstName) $($User.LastName)" `
                   -SamAccountName $User.SamAccountName `
                   -UserPrincipalName "$($User.SamAccountName)@company.local" `
                   -GivenName $User.FirstName `
                   -Surname $User.LastName `
                   -Department $User.Department `
                   -Title $User.Title `
                   -Path $TargetOU `
                   -AccountPassword $SecPassword `
                   -Enabled $true `
                   -ChangePasswordAtLogon $true `
                   -ErrorAction Stop

        Write-Host "SUCCESS: Created user $($User.SamAccountName)" -ForegroundColor Green
    }
    catch {
        Write-Error "FAILED: Cannot create user $($User.SamAccountName). Reason: $_"
    }
}
```

---
## Lab — Step by Step
> [!info] Lab Setup Needed
> A Domain Controller (`SVR-DC01`) with Active Directory PowerShell module installed, and directory `C:\LabFiles` created.

### Step 1: Create the Source CSV File
1. Open PowerShell on the DC. Run the following to generate the `users.csv` file:
   ```powershell
   $CSVContent = @"
   SamAccountName,FirstName,LastName,Department,Title,OUPath
   asmith,Alice,Smith,Sales,Rep,"OU=Corp_Users,OU=Corp_Root,DC=company,DC=local"
   bjones,Bob,Jones,HR,Coordinator,"OU=Corp_Users,OU=Corp_Root,DC=company,DC=local"
   charris,Chris,Harris,IT,Admin,"OU=Corp_Users,OU=Corp_Root,DC=company,DC=local"
   "@
   New-Item -Path "C:\LabFiles" -ItemType Directory -Force
   $CSVContent | Out-File -FilePath "C:\LabFiles\users.csv" -Encoding utf8
   ```

### Step 2: Run the Bulk Creation Script
1. Open PowerShell ISE or VS Code as Administrator on the DC.
2. Load the `BulkCreateUsers.ps1` script (pasted in Section 3 above).
3. Run the script. Verify output logs show success.

### Step 3: Verify and Query AD using PowerShell
1. Query the newly created users to verify their description and password parameters:
   ```powershell
   Get-ADUser -Filter "SamAccountName -eq 'asmith'" -Properties Department, Title, CannotChangePassword
   ```
2. Verify that they are enabled and forced to change passwords:
   ```powershell
   Get-ADUser -Filter "Department -eq 'Sales'" | Select-Object Name, UserPrincipalName, Enabled
   ```

---
## Related Notes
- [[03-Identity-and-Core-Services/06-Active-Directory/WS-02 Active Directory Domain Services|WS-02 Active Directory Domain Services]] — Core AD containers and schemas.
- [[03-Identity-and-Core-Services/06-Active-Directory/WS-06 Active Directory — Users Groups OUs|WS-06 Active Directory — Users Groups OUs]] — Manual AD objects administration rules.
- [[05-Automation-and-Ticketing/10-Scripting-PowerShell/PS-01 PowerShell Fundamentals|PS-01 PowerShell Fundamentals]] — Pipelines and comparison operators basics.
- [[05-Automation-and-Ticketing/10-Scripting-PowerShell/PS-04 PowerShell Scripting|PS-04 PowerShell Scripting]] — Advanced parameter coding standards.

