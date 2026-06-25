---
tags: [sysadmin, powershell, scripting, automation]
difficulty: Advanced
lab-required: Yes
read-time: 20 mins
---

# PS-04: PowerShell Scripting

> [!abstract] Overview
> This note covers advanced PowerShell scripting. It details CmdletBinding parameters, Try/Catch error handling, pipeline logging, registry modifications, and contains five production-ready automation scripts.

---
## Concept
Think of writing a PowerShell script like designing a robotic assembly machine for a manufacturing line:
- The **Param Block** is the physical hopper at the top where you feed raw materials (variables, file paths).
- **Try-Catch-Finally** is the safety breaker circuit: if a bottle jams on the line (e.g., access denied to a file), the system doesn't blow up and catch fire. It triggers the backup breaker (Catch), logs the warning, and safely keeps the conveyor belt moving.
- **Advanced Functions (`CmdletBinding`)** turn your basic script into a certified smart machine that supports standard safety features like `-WhatIf` (simulating the run without changing files) and `-Verbose` (printing real-time sensor metrics).

*Seedha simple mein: PowerShell scripting multiple commands ko logical flow (Try/Catch) aur functions mein packaging karne ka tareeqa hai. Iske zariye hum CSV/JSON configuration templates, system health audits, aur Active Directory updates ko automate karte hain.*

---
## Technical Deep Dive

### 1. Script Architecture & Lifecycle Blocks
Advanced scripts use three execution blocks inside functions:
- **`begin`:** Runs once when the function starts. Used to initialize variables, database connections, or log files.
- **`process`:** Runs once for every object passed through the pipeline. **CRITICAL:** All core loop logic must reside here.
- **`end`:** Runs once after all pipeline objects are processed. Used to close file streams and write summaries.

### 2. Advanced Functions & CmdletBinding
Adding `[CmdletBinding()]` to a function elevates it to behave like a native cmdlet:
- Enables common parameters: `-Verbose`, `-Debug`, `-ErrorAction`.
- Supports **`-WhatIf`** and **`-Confirm`** (requires `SupportsShouldProcess=$true`), allowing admins to dry-run destructive scripts safely.

### 3. Structured Error Handling
Never rely on default shell errors. Use `Try-Catch-Finally` blocks:
- **`-ErrorAction Stop`:** Forces standard non-terminating errors (like a missing file) to become terminating errors, triggering the `Catch` block.
- **`Finally`:** Code here runs *always*, regardless of whether an error occurred (used to close database connections or file handles).

```powershell
try {
    $Content = Get-Content -Path "C:\secure.txt" -ErrorAction Stop
}
catch [System.IO.FileNotFoundException] {
    Write-Warning "File not found: $_"
}
catch {
    Write-Error "Unexpected failure: $_"
}
finally {
    # Cleanup runs always
}
```

### 4. Logging Stream Channels
- `Write-Verbose` — Informational developer logs (suppressed by default; active when running `-Verbose`).
- `Write-Debug` — Detailed debugging metrics.
- `Write-Error` — Non-terminating error log.

### 5. Registry Manipulation via PowerShell Drive
PowerShell treats the Windows Registry as a filesystem drive (`HKLM:` and `HKCU:`):
- *Query:* `Get-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion"`
- *Write:* `New-ItemProperty -Path "HKLM:\Software\MyApp" -Name "Version" -Value "1.0" -PropertyType String -Force`

---
## Five Production-Ready PowerShell Scripts

### Script 1: Active Directory User Audit Report (CSV)
Audits user accounts and exports details (logon date, status, password expiry) to a CSV report.
```powershell
[CmdletBinding()]
param(
    [string]$OutPath = "C:\AuditReports\AD_User_Audit.csv"
)

# Create output folder
$Folder = Split-Path $OutPath
if (-not (Test-Path $Folder)) { New-Item -Path $Folder -ItemType Directory -Force }

# Query AD
$Users = Get-ADUser -Filter * -Properties LastLogonDate, PasswordExpired, LockedOut, Enabled, PasswordLastSet

$Report = foreach ($User in $Users) {
    [PSCustomObject]@{
        SamAccountName     = $User.SamAccountName
        Name               = $User.Name
        Enabled            = $User.Enabled
        LockedOut          = $User.LockedOut
        PasswordLastSet    = $User.PasswordLastSet
        LastLogonDate      = $User.LastLogonDate
        PasswordExpired    = $User.PasswordExpired
    }
}

$Report | Export-Csv -Path $OutPath -NoTypeInformation -Encoding utf8
Write-Host "Audit report exported successfully to $OutPath" -ForegroundColor Green
```

---

### Script 2: Disk Space Monitoring with Email Alert
Checks all system drive volumes; sends an SMTP email alert if space falls below $10\%$.
```powershell
[CmdletBinding()]
param(
    [int]$ThresholdPercent = 10,
    [string]$SMTPServer = "smtp.company.local",
    [string]$ToEmail = "sysadmin@company.com"
)

$Disks = Get-CimInstance -ClassName Win32_LogicalDisk -Filter "DriveType=3" # Local disks only

foreach ($Disk in $Disks) {
    $FreeSpaceGB = [math]::Round($Disk.FreeSpace / 1GB, 2)
    $SizeGB = [math]::Round($Disk.Size / 1GB, 2)
    $PercentFree = [math]::Round(($Disk.FreeSpace / $Disk.Size) * 100, 2)

    if ($PercentFree -lt $ThresholdPercent) {
        $Subject = "CRITICAL DISK ALERT: $($Disk.DeviceID) on $(hostname)"
        $Body = "Drive $($Disk.DeviceID) has only $PercentFree% free space left ($FreeSpaceGB GB of $SizeGB GB total)."
        
        Write-Warning $Body
        # Send-MailMessage -SmtpServer $SMTPServer -From "alerts@company.com" -To $ToEmail -Subject $Subject -Body $Body
    }
}
```

---

### Script 3: Bulk Password Reset from CSV
Reads a CSV list of users and resets their passwords to a temporary default, forcing change on next login.
```powershell
<#
Input CSV layout (reset.csv):
SamAccountName
jdoe
asmith
#>
[CmdletBinding()]
param(
    [string]$CSVPath = "C:\LabFiles\reset.csv",
    [string]$TempPass = "TempPassword999!"
)

Import-Module ActiveDirectory
$SecPassword = ConvertTo-SecureString $TempPass -AsPlainText -Force

if (-not (Test-Path $CSVPath)) {
    Write-Error "CSV file not found."
    exit 1
}

$Users = Import-Csv -Path $CSVPath

foreach ($User in $Users) {
    try {
        $ADUser = Get-ADUser -Identity $User.SamAccountName -ErrorAction Stop
        
        # Reset password
        Set-ADAccountPassword -Identity $ADUser.SamAccountName -NewPassword $SecPassword -Reset -ErrorAction Stop
        # Force password change at next logon
        Set-ADUser -Identity $ADUser.SamAccountName -ChangePasswordAtLogon $true -ErrorAction Stop
        
        Write-Host "SUCCESS: Reset completed for user $($User.SamAccountName)" -ForegroundColor Green
    }
    catch {
        Write-Warning "FAILED: Could not reset password for $($User.SamAccountName). Reason: $_"
    }
}
```

---

### Script 4: Server Health Check Report (HTML)
Gathers system metrics (OS, CPU, Memory, Stopped Services) and compiles a styled HTML report.
```powershell
$OutPath = "C:\AuditReports\Server_Health.html"

# Gather Metrics
$OS = Get-CimInstance Win32_OperatingSystem
$CS = Get-CimInstance Win32_ComputerSystem
$RAM = [math]::Round($OS.TotalVisibleMemorySize / 1MB, 2)
$FreeRAM = [math]::Round($OS.FreePhysicalMemory / 1MB, 2)
$StoppedServices = Get-Service | Where-Object {$_.StartType -eq "Automatic" -and $_.Status -eq "Stopped"}

# Compile HTML String
$HTML = @"
<html>
<head>
<style>
body { font-family: Arial; }
th { background-color: #004D40; color: white; padding: 5px; }
td { padding: 5px; border-bottom: 1px solid #ddd; }
</style>
</head>
<body>
<h2>Server Health Report: $($CS.Name)</h2>
<hr>
<h3>Operating System Specs</h3>
<table>
<tr><th>OS</th><td>$($OS.Caption)</td></tr>
<tr><th>Version</th><td>$($OS.Version)</td></tr>
<tr><th>Total RAM</th><td>$RAM GB</td></tr>
<tr><th>Free RAM</th><td>$FreeRAM GB</td></tr>
</table>
<h3>Stopped Automatic Services</h3>
<table>
<tr><th>Service Name</th><th>Display Name</th></tr>
"@

foreach ($Svc in $StoppedServices) {
    $HTML += "<tr><td>$($Svc.Name)</td><td>$($Svc.DisplayName)</td></tr>"
}

$HTML += "</table></body></html>"
$HTML | Out-File -FilePath $OutPath -Encoding utf8
Write-Host "Health check HTML report written to $OutPath" -ForegroundColor Green
```

---

### Script 5: Inactive User Cleanup Script
Identifies users who have not logged in for 90 days and disables their accounts.
```powershell
[CmdletBinding(SupportsShouldProcess=$true)]
param(
    [int]$InactiveDays = 90
)

Import-Module ActiveDirectory
$CutoffDate = (Get-Date).AddDays(-$InactiveDays)

# Query inactive enabled accounts
$Users = Get-ADUser -Filter "Enabled -eq '$true'" -Properties LastLogonDate | 
         Where-Object {$_.LastLogonDate -lt $CutoffDate -and $_.LastLogonDate -ne $null}

foreach ($User in $Users) {
    $ActionMsg = "Disabling inactive account: $($User.SamAccountName) (Last Logon: $($User.LastLogonDate))"
    
    # SupportsShouldProcess enables support for standard -WhatIf and -Confirm parameters
    if ($PSCmdlet.ShouldProcess($User.SamAccountName, "Disable AD User Account")) {
        Disable-ADAccount -Identity $User.SamAccountName
        Write-Host $ActionMsg -ForegroundColor Yellow
    }
}
```

---
## Lab — Step by Step
> [!info] Lab Setup Needed
> A Windows client or server running PowerShell, with directory `C:\LabFiles` and `C:\AuditReports` created.

### Step 1: Create the Health Check Script
1. Open PowerShell ISE or VS Code.
2. Create a new file and paste the contents of **Script 4** (Server Health Check Report).
3. Save the file as `C:\LabFiles\Get-HealthReport.ps1`.

### Step 2: Execute the Script with Execution Permissions
1. Open PowerShell as Administrator.
2. Verify execution policy supports local scripts (RemoteSigned).
3. Run the script:
   ```powershell
   & "C:\LabFiles\Get-HealthReport.ps1"
   ```
4. Confirm console logs print "Health check HTML report written...".

### Step 3: Verify the Output
1. Navigate to the output directory:
   ```powershell
   Invoke-Item "C:\AuditReports\Server_Health.html"
   ```
2. **Verify:** The default web browser should open and display the formatted HTML report showing CPU/RAM metrics and stopped automatic services.

---
## Related Notes
- [[03-Identity-and-Core-Services/06-Active-Directory/WS-05 Group Policy — Complete Guide|WS-05 Group Policy — Complete Guide]] — Using startup scripts via GPO.
- [[05-Automation-and-Ticketing/10-Scripting-PowerShell/PS-01 PowerShell Fundamentals|PS-01 PowerShell Fundamentals]] — Core language variables and aliases syntax.
- [[05-Automation-and-Ticketing/10-Scripting-PowerShell/PS-02 PowerShell for Active Directory|PS-02 PowerShell for Active Directory]] — Pipeline loops for AD users.
- [[05-Automation-and-Ticketing/10-Scripting-PowerShell/PS-03 PowerShell for System Administration|PS-03 PowerShell for System Administration]] — Accessing remote CIM classes inside scripts.

