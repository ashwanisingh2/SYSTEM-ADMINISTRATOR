---
tags: [powershell, scripting, variables, logic, conditionals, automation]
aliases: [ps-variables-logic, powershell-logic]
created: 2026-06-26
status: #complete
difficulty: #beginner
cert-relevant: #none
---

> [!NOTE|color-purple]
> 📜 **POWERSHELL**

`#complete` `#beginner` `#none`

# PS-02: Variables and Logic

> [!abstract] Overview
> *PowerShell mein variables data store karne ke liye use hote hain, aur logic (if/else, loops) scripts ko decision lene mein madad karte hain. Ek system administrator ko yeh aana chahiye taaki woh repetitive tasks ko automate kar sake aur dynamic scripts likh sake.* This note covers variable declaration, data types, comparison operators, and logical constructs in PowerShell.

---
## 🧠 Concept Overview

- **What it is** — Variables are containers for storing data values. Logic refers to the conditional statements (`if`, `else`, `switch`) and loops (`for`, `foreach`, `while`) that control the flow of a script.
- **Why it matters** — Without variables, scripts are static. Without logic, scripts can't react to different situations (e.g., *agar service band hai toh start karo, warna chhod do*). 
- **Where you see this** — Used in almost every automation script: reading user inputs, processing lists of servers, checking service status, and generating reports.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Run existing scripts and pass simple variable values (like username or server name). *Basic errors check karna.* |
| **L2** | Modify scripts, add basic `if/else` logic to handle specific errors, update variable values to adapt to new requirements. |
| **L3** | Write complex modules, use advanced logic arrays and hash tables, implement robust error handling and logging. |

> [!tip] Seedha Simple Mein
> *Variables matlab dabbe jisme tum data rakhte ho. Logic matlab dimaag jo decide karta hai ki uss data ke saath kya karna hai.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **A Restaurant Order System** is like **Variables and Logic** because...
>
> - **Variables:** The order ticket (Table 5, 2 Burgers, 1 Coke). *Yeh data store kar raha hai ki kisne kya order kiya.*
> - **Logic (If/Else):** The chef checking the ticket. "If burger is ordered, start grilling patty. If Coke is ordered, pour drink." *Conditions check ho rahi hain.*
> - **Loops (Foreach):** The waiter delivering food to multiple tables. "For each table with food ready, deliver the food." *Repetitive kaam ek list (tables) par perform karna.*

---
## 🔬 Technical Deep Dive

### 1. Variables in PowerShell

> [!info] Key Concept
> Variables in PowerShell start with a dollar sign `$`. They can hold strings, integers, arrays, objects, or even the output of commands.

```powershell
# Basic variables
$Username = "admin_user"
$Port = 8080
$IsActive = $true

# Storing command output in a variable
$Services = Get-Service
```

> [!danger] Common Mistake
> Forgetting the `$` sign when calling the variable later in the script. *Variable declare karte waqt aur use karte waqt dono time `$` lagana zaroori hai.* `Write-Host Username` will just print the word "Username", not the value. It should be `Write-Host $Username`.

### 2. Comparison Operators

> [!info] Key Concept
> PowerShell uses text-based comparison operators instead of traditional mathematical symbols (`<`, `>`, `==`).

- `-eq` : Equal to
- `-ne` : Not equal to
- `-gt` : Greater than
- `-lt` : Less than
- `-ge` : Greater than or equal to
- `-le` : Less than or equal to
- `-like` : Wildcard comparison (e.g., `*test*`)
- `-match` : Regular expression comparison

```powershell
if ($Port -eq 80) {
    Write-Host "HTTP Port is selected."
}
```

### 3. Conditional Logic (If / ElseIf / Else)

Used to execute code only if a certain condition is met.

```powershell
$CpuUsage = 85

if ($CpuUsage -gt 90) {
    Write-Host "Critical: CPU is too high!"
} elseif ($CpuUsage -gt 70) {
    Write-Host "Warning: CPU is getting high."
} else {
    Write-Host "Normal: CPU is fine."
}
```

### 4. Loops (Foreach, For, While)

> [!info] Key Concept
> Loops allow you to run the same block of code multiple times. `Foreach` is the most common in admin scripts.

```powershell
$Servers = @("Server01", "Server02", "Server03")

foreach ($Server in $Servers) {
    Write-Host "Checking status of $Server..."
    # Yahan ping ya koi aur check command laga sakte hain
}
```

> [!danger] Common Mistake
> Creating an infinite loop with `while`. *Agar condition kabhi false nahi hogi, toh script chalti hi rahegi jab tak system crash na ho.* Always ensure the loop has an exit condition.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Windows OS with PowerShell 5.1 or later.
> - Administrator privileges (optional, but good for some service checks).

### Step 1: Create and Output Variables

```powershell
# Create variables
$FirstName = "Ashwani"
$LastName = "Singh"
$FullName = "$FirstName $LastName"

# Output
Write-Host "Admin logged in: $FullName"
```

> [!success] Expected Output
> ```
> Admin logged in: Ashwani Singh
> ```

### Step 2: Use If/Else Logic to Check a Service

```powershell
# Check if Spooler service is running
$ServiceName = "Spooler"
$ServiceStatus = (Get-Service -Name $ServiceName).Status

if ($ServiceStatus -eq "Running") {
    Write-Host "$ServiceName is already running. No action needed." -ForegroundColor Green
} else {
    Write-Host "$ServiceName is stopped. Starting it now..." -ForegroundColor Yellow
    Start-Service -Name $ServiceName
    Write-Host "$ServiceName started successfully." -ForegroundColor Green
}
```

> [!success] Expected Output
> *(Depending on current status)*
> ```
> Spooler is already running. No action needed.
> ```

### Step 3: Iterate Through an Array with Foreach

```powershell
# List of processes to check
$ProcessList = @("explorer", "chrome", "notepad")

foreach ($Proc in $ProcessList) {
    $IsRunning = Get-Process -Name $Proc -ErrorAction SilentlyContinue
    
    if ($IsRunning) {
        Write-Host "Process $Proc is RUNNING." -ForegroundColor Cyan
    } else {
        Write-Host "Process $Proc is NOT RUNNING." -ForegroundColor Red
    }
}
```

> [!success] Expected Output
> ```
> Process explorer is RUNNING.
> Process chrome is RUNNING.
> Process notepad is NOT RUNNING.
> ```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `$Var = "Value"` | Declares a variable | `$Server = "DC01"` |
| `Read-Host` | Prompts user for input | `$User = Read-Host "Enter username"` |
| `-eq`, `-ne` | Equal, Not Equal operators | `if ($A -eq $B)` |
| `-gt`, `-lt` | Greater Than, Less Than | `if ($CPU -gt 80)` |
| `if () {} else {}` | Conditional execution | `if ($true) { Write-Host "Yes" }` |
| `foreach ($x in $y)` | Loop through a collection | `foreach ($File in $Files) {}` |
| `switch ($var) {}` | Multiple condition checking | `switch ($Status) { "Running" {} }` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| Variable not outputting correctly | Single quotes (`'`) used instead of double quotes (`"`) around variables | Use double quotes: `"Hello $Name"` *Single quotes variable ko text maan lete hain.* |
| `if ($var = 5)` error | Used `=` (assignment) instead of `-eq` (comparison) | Change to `if ($var -eq 5)`. `=` sign sirf value assign karne ke liye hai. |
| Object vs String properties | Trying to compare an object instead of its property | Use dot notation: `if ($Service.Status -eq "Running")` |
| Variable not found | Typo in variable name or scope issue | Check spelling. Ensure variable is declared before it's called. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Bulk User Password Reset

> [!example] Ticket
> "Need to reset passwords for a list of 50 temp users and set them to require a password change at next logon."

**L1 Response:** Manually resetting via ADUC GUI (Very slow, prone to errors). *Ek ek karke AD mein change karna padega.*
**Escalation Trigger:** Too many users, takes too long. Escalate to L2 for automation.
**L2 Resolution:** Writes a PowerShell script using `foreach` loop.
```powershell
$Users = Get-Content "C:\Temp\users.txt"
foreach ($User in $Users) {
    Set-ADAccountPassword $User -Reset
    Set-ADUser -Identity $User -ChangePasswordAtLogon $true
}
```

### 🎫 Scenario 2: Disk Space Alerting

> [!example] Ticket
> "Monitoring team needs a daily report of any C: drives under 10% free space across our critical servers."

**L1 Response:** Manually logging into servers to check C: drive properties.
**Escalation Trigger:** Cannot do this manually every day.
**L2 Resolution:** Uses an array of servers, a `foreach` loop, and `if` logic to check space.
```powershell
$Servers = @("SRV01", "SRV02", "SRV03")
foreach ($Server in $Servers) {
    $Disk = Get-WmiObject Win32_LogicalDisk -ComputerName $Server -Filter "DeviceID='C:'"
    $FreePct = ($Disk.FreeSpace / $Disk.Size) * 100
    if ($FreePct -lt 10) {
        Write-Warning "$Server C: Drive is critically low!"
    }
}
```

### 🎫 Scenario 3: Service Auto-Restarter

> [!example] Ticket
> "The legacy inventory app service crashes randomly. Can we check it every hour and restart it if it's down?"

**L1 Response:** Restart service manually when users complain. *Jab user call karega tab service restart karenge.*
**Escalation Trigger:** Users are getting angry waiting for L1 to fix it. Needs automated remediation.
**L2 Resolution:** Sets up a Scheduled Task running a PowerShell script with `if` logic.
```powershell
$Service = Get-Service "LegacyApp"
if ($Service.Status -ne "Running") {
    Start-Service "LegacyApp"
    Send-MailMessage -To "admin@company.com" -Subject "LegacyApp Restarted"
}
```

---
## 🎤 Interview Questions

> [!question] Q1: How do you declare a variable in PowerShell?
> **Answer:** You prefix the variable name with a dollar sign `$`. For example, `$MyVariable = "Value"`.

> [!question] Q2: What is the difference between single quotes (`'`) and double quotes (`"`) when dealing with variables?
> **Answer:** Double quotes expand variables (evaluate their values). Single quotes treat the content as literal strings. For example, `"$Name"` outputs the value of `$Name`, while `'$Name'` just outputs the text `$Name`.
> ==**Exam Tip:** Double quotes "expand", Single quotes are "literal".==

> [!question] Q3: Why does `if ($number > 5)` fail in PowerShell?
> **Answer:** PowerShell uses `-gt` for "greater than", not the mathematical `>`. The `>` symbol in PowerShell is used for output redirection to a file. The correct syntax is `if ($number -gt 5)`.

> [!question] Q4: How do you loop through an array of server names in a script?
> **Answer:** By using a `foreach` loop. For example: `foreach ($Server in $ServerList) { ... }`. This executes the code block once for every item in the array.

> [!question] Q5: What is the purpose of `$null`?
> **Answer:** `$null` is an automatic variable in PowerShell that represents nothing or no value. It is often used in `if` statements to check if a command returned any output (e.g., `if ($User -eq $null)`).

---
## 🔗 Related Notes

- [[PS-01 Introduction to PowerShell|PS-01: Introduction to PowerShell]] — Basics of cmdlets and pipeline.
- [[PS-03 Objects and Pipelines|PS-03: Objects and Pipelines]] — How variables interact with objects.
- [[PS-04 Functions and Modules|PS-04: Functions and Modules]] — Wrapping your logic into reusable commands.
