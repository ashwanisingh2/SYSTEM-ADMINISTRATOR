---
tags: [powershell, automation, scripting, advanced]
aliases: [ps-automation, powershell-scripting]
created: 2026-06-26
status: #complete
difficulty: #advanced
cert-relevant: #none
---

> [!NOTE|color-purple]
> 📜 **POWERSHELL**

`#complete` `#advanced` `#none`

# PS-09: PowerShell Automation and Scripting

> [!abstract] Overview
> Yeh note advanced PowerShell scripting aur automation concepts cover karta hai, jisme loops, functions, error handling, aur scheduled tasks shamil hain. Ek support engineer ko apne daily tasks automate karne ke liye in advanced techniques ka aana zaroori hai.

---
## 🧠 Concept Overview

- **What it is** — PowerShell scripts likhna to automate repetitive tasks aur complex workflows ko streamline karna.
- **Why it matters** — Manual kaam me time waste hota hai aur mistakes ke chances badhte hain. Automation se speed, consistency, aur accuracy milti hai.
- **Where you see this** — User onboarding scripts, daily health checks, bulk AD updates, log cleanup scripts, aur server patching.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Bani banayi scripts ko run karna aur basic parameters pass karna. Script fail hone par error report karna. |
| **L2** | Existing scripts ko modify karna, loops aur basic error handling add karna, minor bugs fix karna. |
| **L3** | Complex modules design karna, API integrations, parallel processing, aur enterprise-level automation architecture banana. |

> [!tip] Seedha Simple Mein
> *Automation ka matlab hai ki jo kaam aap roz 10 bar manually karte ho, use ek baar code mein likh do. Fir woh kaam ek click me khud ho jayega.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **PowerShell Script** is like a **Robot Assistant in a factory** because...
>
> - Factory me ek hi product baar-baar banana hota hai (Repetitive task).
> - Robot ko ek baar instruct (script) kar diya, toh woh bina thake kaam karta rahega (Automation).
> - Agar raw material khatam ho gaya, toh robot alarm bajayega (Error Handling).

---
## 🔬 Technical Deep Dive

### 1. Variables and Data Types

> [!info] Key Concept
> Variables data store karne ke containers hote hain. PowerShell dynamically typed hai par strong typing bhi support karta hai.

```powershell
# Strong typing example
[int]$serverCount = 10
[string]$serverName = "DC01"
[array]$serverList = @("DC01", "EX01", "FS01")
[hashtable]$config = @{Name="DC01"; IP="192.168.1.10"}
```

### 2. Control Structures (If/Else, Switch)

> [!info] Key Concept
> Condition ke basis pe script ka flow control karna.

```powershell
$service = Get-Service -Name "wuauserv"
if ($service.Status -eq 'Running') {
    Write-Host "Windows Update is running." -ForegroundColor Green
} else {
    Write-Host "Service is stopped. Starting it..." -ForegroundColor Yellow
    Start-Service -Name "wuauserv"
}
```

### 3. Loops (Foreach, For, While)

> [!info] Key Concept
> Multiple items pe ek hi action perform karna (Bulk operations).

```powershell
$users = Import-Csv -Path "C:\temp\new_users.csv"
foreach ($user in $users) {
    # AD User create karne ka command
    Write-Host "Creating user: $($user.Name)"
}
```

### 4. Custom Functions

> [!info] Key Concept
> Code ke block ko reuse karne ke liye functions banaye jate hain. Advanced functions mein `[CmdletBinding()]` use hota hai.

```powershell
function Get-ServerUptime {
    [CmdletBinding()]
    param (
        [Parameter(Mandatory=$true)]
        [string]$ComputerName
    )
    $os = Get-WmiObject -Class Win32_OperatingSystem -ComputerName $ComputerName
    $uptime = (Get-Date) - $os.ConvertToDateTime($os.LastBootUpTime)
    return $uptime.Days
}
```

### 5. Error Handling (Try / Catch / Finally)

> [!info] Key Concept
> Script bich me break na ho aur errors ko gracefully handle kiya jaye.

```powershell
try {
    # ErrorAction Stop zaroori hai Try-Catch work karne ke liye
    Stop-Service -Name "NonExistentService" -ErrorAction Stop
} catch {
    Write-Warning "Failed to stop service: $($_.Exception.Message)"
} finally {
    Write-Host "Execution completed."
}
```

> [!danger] Common Mistake
> ErrorAction 'Stop' set nahi karna. Default behavior (Continue) mein catch block trigger nahi hota. *Try-Catch ka use tabhi hoga jab aap PowerShell ko bataoge ki error aane par ruko.*

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Administrator access to the machine.
> - PowerShell execution policy set to RemoteSigned or Unrestricted.

### Step 1: Set Execution Policy

```powershell
# Script run karne ke liye permission deni hogi
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser -Force
```

### Step 2: Create a Basic System Health Report Script

Create a script `HealthCheck.ps1`:

```powershell
$computer = $env:COMPUTERNAME
$date = Get-Date -Format "yyyy-MM-dd"
$reportPath = "C:\temp\HealthReport_$date.txt"

# 1. CPU Usage
$cpu = Get-WmiObject Win32_Processor | Measure-Object -Property LoadPercentage -Average | Select-Object -ExpandProperty Average

# 2. Disk Space (C: Drive)
$disk = Get-Volume -DriveLetter C
$freeSpaceGB = [math]::Round($disk.SizeRemaining / 1GB, 2)

# Generate Report
"System Health Report for $computer" | Out-File $reportPath
"---------------------------------" | Out-File $reportPath -Append
"CPU Usage: $cpu %" | Out-File $reportPath -Append
"C: Drive Free Space: $freeSpaceGB GB" | Out-File $reportPath -Append

Write-Host "Report saved to $reportPath" -ForegroundColor Green
```

> [!success] Expected Output
> ```
> Report saved to C:\temp\HealthReport_2026-06-26.txt
> ```

### Step 3: Run the Script
```powershell
.\HealthCheck.ps1
```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `Write-Host` | Console par text print karta hai (not for output streams) | `Write-Host "Done" -ForegroundColor Green` |
| `Write-Output` | Pipeline me data bhejta hai | `Write-Output $data` |
| `Import-Csv` | CSV file ko PowerShell objects me convert karta hai | `$data = Import-Csv -Path file.csv` |
| `Export-Csv` | PowerShell objects ko CSV me save karta hai | `$data \| Export-Csv -Path out.csv -NoTypeInformation` |
| `Start-Transcript` | Console output ka record rakhta hai (logging) | `Start-Transcript -Path log.txt` |
| `Stop-Transcript` | Transcript logging ko stop karta hai | `Stop-Transcript` |
| `Invoke-Command` | Remote computer par command/script run karta hai | `Invoke-Command -ComputerName Srv1 -ScriptBlock { Get-Process }` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| `cannot be loaded because running scripts is disabled` | Execution policy restricted hai. | `Set-ExecutionPolicy RemoteSigned -Scope CurrentUser` run karein. |
| Catch block is not executing | Error non-terminating tha. | Cmdlet ke aage `-ErrorAction Stop` lagayein taaki try-catch trigger ho. |
| `File cannot be found` in Import-Csv | Path galat hai ya file exist nahi karti. | Full absolute path use karein ya `Test-Path` se file check karein. |
| Variables are retaining old values | Loop ke andar variable overwrite nahi ho raha properly. | Loop ke start mein variable ko `$null` set kar dein. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: New Employee Onboarding

> [!example] Ticket
> "HR has sent a CSV with 50 new hires. Please create their AD accounts, set default passwords, and add them to the AllStaff group."

**L1 Response:** Script run karega aur CSV input dega. Success/fail logs check karega.
**Escalation Trigger:** Agar kuch accounts ban nahi rahe ya password policy error aa raha hai.
**L2 Resolution:** Script check karega ki attributes (UPN, SAMAccountName) theek format mein hain. `-ErrorAction Stop` use karke exact error debug karega.

### 🎫 Scenario 2: Service Stopped on Multiple Servers

> [!example] Ticket
> "The Print Spooler service has crashed on 20 print servers. Please restart it across all of them."

**L1 Response:** Manually RDP karke ya Server Manager se restart karne ki koshish karega (time-consuming).
**Escalation Trigger:** Agar 20 servers hain aur jaldi karna hai.
**L2 Resolution:** PowerShell ka use karke bulk restart karega:
`Invoke-Command -ComputerName (Get-Content servers.txt) -ScriptBlock { Restart-Service Spooler -Force }`

### 🎫 Scenario 3: Disk Space Alert Automation

> [!example] Ticket
> "We are getting too many alerts for C: drive space. Please write a script to automatically clean up IIS logs older than 30 days."

**L1 Response:** Manually C:\inetpub\logs me jaa kar delete karega.
**Escalation Trigger:** L1 wants a permanent automated fix.
**L2 Resolution:** Ek script likhega aur Task Scheduler me set karega:
`Get-ChildItem -Path "C:\inetpub\logs\LogFiles" -Recurse \| Where-Object { $_.LastWriteTime -lt (Get-Date).AddDays(-30) } \| Remove-Item -Force`

---
## 🎤 Interview Questions

> [!question] Q1: What is the difference between Write-Host and Write-Output?
> **Answer:** `Write-Host` sirf console screen par text show karne ke liye hota hai (colors waghera ke sath), yeh pipeline me data aage nahi bhejta. `Write-Output` pipeline me object bhejta hai, jisko doosre cmdlets process kar sakte hain (jaise Export-Csv).

==**Exam Tip:** Hamesha pipeline output ke liye `Write-Output` ya implicit output use karein, `Write-Host` sirf user messages/logging ke liye.==

> [!question] Q2: How do you handle errors in a PowerShell script?
> **Answer:** `try { } catch { } finally { }` block use karke. Yeh zaroori hai ki hum terminating errors generate karein, jiske liye command ke sath `-ErrorAction Stop` ya script ke top pe `$ErrorActionPreference = 'Stop'` define karna hota hai.

> [!question] Q3: What is `$PSItem` or `$_` in PowerShell?
> **Answer:** Yeh ek automatic variable hai jo pipeline mein current object ko represent karta hai. For example, `Get-Process \| Where-Object { $_.CPU -gt 100 }` mein `$_` pipeline ke ek-ek process ko refer karta hai.

> [!question] Q4: What does `[CmdletBinding()]` do at the top of a function?
> **Answer:** Yeh ek simple function ko "Advanced Function" bana deta hai. Isse function built-in parameters jaise `-Verbose`, `-Debug`, aur `-ErrorAction` support karne lagta hai, jaise standard PowerShell cmdlets karte hain.

---
## 🔗 Related Notes

- [[PS-01 Introduction to PowerShell|PS-01 Introduction to PowerShell]] — Basics of PowerShell.
- [[AD-04 Active Directory Bulk User Creation|AD-04 AD Bulk User Creation]] — Scripting applied to Active Directory.
- [[WIN-07 Windows Task Scheduler|WIN-07 Windows Task Scheduler]] — Automating script execution on a schedule.
