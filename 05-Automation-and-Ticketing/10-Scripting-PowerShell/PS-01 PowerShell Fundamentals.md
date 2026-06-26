---
tags: [powershell, scripting, CLI, fundamentals, automation, windows]
aliases: [PS-01, PowerShell Basics, PowerShell Intro]
created: 2026-06-26
status: "#complete"
difficulty: "#beginner"
cert-relevant: "#md-102"
---

> [!NOTE|color-purple]
> 📜 **POWERSHELL**

`#complete` `#beginner` `#md-102`

# PS-01: PowerShell Fundamentals

> [!abstract] Overview
> Yeh note PowerShell ke baseline administration ko cover karta hai — CMD aur PowerShell mein difference, PowerShell versions (5.1 vs 7.x), execution policies, help system, object pipeline, variables, operators, aur aliases. Agar tum ek L1 support engineer ho aur PowerShell seekhna chahte ho, toh yeh tumhara **Day-1 guide** hai. Iske baad tum commands likh paoge, scripts run kar paoge, aur objects ko pipeline mein manipulate kar paoge.

---

## 🧠 Concept Overview

- **What it is** — PowerShell ek **object-oriented task automation engine** aur scripting language hai jo Microsoft ne .NET Framework pe build kiya hai. Yeh CMD ka advanced, modern replacement hai.
- **Why it matters** — Aaj har Windows admin ka kaam PowerShell ke bina incomplete hai. Server management, Active Directory, Azure, Microsoft 365 — sab jagah PowerShell use hota hai. *Bina PowerShell ke tum ek haath se kaam kar rahe ho.*
- **Where you see this** — User account creation, bulk password resets, service restarts, log collection, scheduled tasks, deployment scripts, aur remote server management mein.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic cmdlets chalana (`Get-Service`, `Get-Process`), execution policy check karna, help system use karna |
| **L2** | Pipeline scripts likhna, execution policy configure karna, remote sessions banana, bulk operations |
| **L3** | Advanced scripting, DSC (Desired State Configuration), module development, CI/CD automation |

> [!tip] Seedha Simple Mein
> *PowerShell ek aisi language hai jo text nahi, **objects** return karti hai. Jaise CMD mein tum text parse karte ho, PowerShell mein tum directly object ke properties access karte ho — jaise ek Excel sheet ke columns. Pipeline (`|`) se ek command ka output dusre command ka input ban jaata hai. Execution policy decide karti hai ki scripts run hogi ya nahi.*

---

## 💡 Real-World Analogy

> [!info] Think of it like this...
> **CMD** is like an **old manual typewriter** — jab tum type karte ho, flat text lines aati hain paper pe. Agar tumhe doosri line ka teesra word chahiye, toh complex text filtering macros (`findstr`, `for /f`) likhne padte hain characters parse karne ke liye.
>
> **PowerShell** is like a **modern automated assembly line** handling **3D physical objects** — text nahi.
>
> - Jab tum `Get-Process` run karte ho, PowerShell text nahi deta — **object blocks** deta hai
> - Har block ke apne **labels** (Properties jaise `CPU`, `ID`, `ProcessName`) aur **actions** (Methods jaise `Kill()`, `Start()`) hote hain
> - Pipeline (`|`) ek **conveyor belt** hai — ek workstation se object dusre workstation tak jaata hai
> - Tumhe text parse karne ki zaroorat nahi — directly label reference karo, kaam ho gaya!

---

## 🔬 Technical Deep Dive

### 1. PowerShell vs. CMD

> [!info] Key Concept
> CMD text-based hai, PowerShell object-based hai. Yeh sabse fundamental difference hai jo poori philosophy change karta hai.

| 🔍 Feature | 🖥️ CMD (Command Prompt) | 📜 PowerShell |
|------------|------------------------|---------------|
| **Architecture** | Legacy MS-DOS based shell | .NET Framework based engine |
| **Output** | Flat text (string data) | Structured .NET objects |
| **Scripting** | Batch files (`.bat`, `.cmd`) | PowerShell scripts (`.ps1`) |
| **Pipeline** | Passes text between commands | Passes complete objects |
| **Remote Mgmt** | Limited (PsExec, etc.) | Built-in remoting (WinRM) |
| **Cross-Platform** | Windows only | Windows, Linux, macOS (7.x) |

> [!danger] Common Mistake
> Bahut se freshers sochte hain ki PowerShell sirf ek "fancy CMD" hai. **Galat!** PowerShell ek poori programming language hai objects ke saath. CMD mein `dir` text deta hai, PowerShell mein `Get-ChildItem` file objects deta hai with properties like `.Length`, `.LastWriteTime`, `.FullName`.

---

### 2. PowerShell Version Evolution

> [!info] Key Concept
> Do main versions hain — **Windows PowerShell 5.1** (built-in) aur **PowerShell 7.x** (cross-platform). Dono alag-alag executables hain.

| 📦 Version | 🏗️ Framework | 🖥️ Executable | 🌐 Platform |
|-----------|-------------|--------------|------------|
| **5.1** (Windows PowerShell) | .NET Framework | `powershell.exe` | Windows only |
| **7.x** (PowerShell Core) | .NET Core / .NET 6+ | `pwsh.exe` | Windows, Linux, macOS |

- PowerShell 5.1 har Windows 10/11/Server mein pre-installed aata hai
- PowerShell 7.x separately install karna padta hai — dono side-by-side run ho sakte hain
- *Production environments mein agar legacy modules use ho rahe hain toh 5.1 chalega, naye automation ke liye 7.x prefer karo*

==**Exam Tip:** MD-102 mein yeh zaroor poochha jaata hai — PowerShell 5.1 ka executable `powershell.exe` hai aur 7.x ka `pwsh.exe`. Dono alag-alag installed rehte hain.==

---

### 3. Execution Policies (Script Security Gates)

> [!info] Key Concept
> Execution policies **security boundaries nahi hain** — yeh **convenience features** hain jo accidental script execution rokti hain. Koi bhi user inhe bypass kar sakta hai.

| 🔒 Policy | 📋 Kya karta hai | 🏢 Use Case |
|-----------|-----------------|------------|
| **Restricted** | ❌ Koi script nahi chalegi, sirf individual commands | Default Windows setting |
| **AllSigned** | ✍️ Sirf trusted publisher certificate signed scripts | High-security environments |
| **RemoteSigned** | 📝 Local scripts bina sign ke, internet scripts signed honi chahiye | ==Standard admin workstation setting== |
| **Unrestricted** | ⚠️ Sab scripts chalegi, internet files pe warning | Development machines |
| **Bypass** | 🚫 Kuch block nahi, koi warning nahi | Automated deployment agents, CI/CD |

```powershell
# Execution policy check karo — sab scopes dikhega
Get-ExecutionPolicy -List

# RemoteSigned set karo machine level pe
Set-ExecutionPolicy RemoteSigned -Scope LocalMachine

# Sirf current user ke liye set karo (admin rights nahi chahiye)
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser -Force
```

> [!danger] Common Mistake
> Freshers aksar `Set-ExecutionPolicy Unrestricted` ya `Bypass` set kar dete hain aur bhool jaate hain. **Production mein yeh kabhi mat karo!** Hamesha `RemoteSigned` use karo. Exam mein bhi yahi recommended answer hai.

==**Exam Tip:** `RemoteSigned` sabse commonly recommended execution policy hai admin workstations ke liye. Interview aur MD-102 dono mein yeh yaad rakho.==

---

### 4. Navigating the Command Engine (The Holy Trinity)

> [!info] Key Concept
> PowerShell ki **3 sabse important discovery commands** hain — `Get-Command`, `Get-Help`, aur `Get-Member`. Inhe yaad karo toh tum kuch bhi khud discover kar sakte ho.

**`Get-Command`** — Installed cmdlets, functions, aliases search karo:

```powershell
# "Service" se related sab commands dhoondho
Get-Command *service*

# Sirf cmdlets dhoondho (functions nahi)
Get-Command -CommandType Cmdlet -Name *process*
```

**`Get-Help`** — Documentation reference system (man pages jaisa):

```powershell
# Detailed help dekho
Get-Help Get-Service -Detailed

# Sirf examples dekho — sabse useful!
Get-Help Get-Service -Examples

# Online documentation browser mein kholo
Get-Help Get-Service -Online
```

```powershell
# Help system update karo (Admin required) — pehli baar zaroor karo
Update-Help -Force -ErrorAction SilentlyContinue
```

**`Get-Member`** — Object Inspector (X-Ray machine):

```powershell
# Process object ke sab Properties aur Methods dekho
Get-Process -Name explorer | Get-Member

# Sirf properties dekho
Get-Process -Name explorer | Get-Member -MemberType Property
```

> [!tip] Pro Tip
> Jab bhi koi naya cmdlet milta hai, hamesha pehle `Get-Help <cmdlet> -Examples` run karo, phir output ko `Get-Member` mein pipe karo. **Yeh do commands tumhare best friends hain.**

---

### 5. The Object Pipeline

> [!info] Key Concept
> Pipeline (`|`) complete .NET objects pass karta hai — text nahi. `$_` current object ko represent karta hai jo conveyor belt pe chal raha hai.

```powershell
# Sirf running services dikhao, sorted by name
Get-Service | Where-Object {$_.Status -eq "Running"} | Select-Object -Property Name, Status

# Top 5 CPU consuming processes
Get-Process | Sort-Object CPU -Descending | Select-Object -First 5 Name, CPU

# Services export karo CSV mein
Get-Service | Where-Object {$_.Status -eq "Running"} | Export-Csv -Path "C:\Temp\RunningServices.csv" -NoTypeInformation
```

- **`$_`** — Current pipeline object (*jo abhi conveyor belt pe hai*)
- **`Where-Object`** — Filter karo (SQL ke `WHERE` jaisa)
- **`Select-Object`** — Sirf specific columns/properties lo (SQL ke `SELECT` jaisa)
- **`Sort-Object`** — Sorting karo kisi bhi property pe
- **`ForEach-Object`** — Har object pe ek action perform karo

---

### 6. Variables, Typing, and Operators

**Variables** — `$` prefix se define hoti hain:

```powershell
# Simple variable
$MyVar = "Active"

# Type casting — enforce specific data type
[int]$Count = 5
[string]$IP = "10.0.0.1"
[datetime]$Today = Get-Date

# Array
$Servers = @("DC01", "FS01", "WEB01")
```

**Comparison Operators** — PowerShell standard symbols (`>`, `<`) use nahi karta kyunki woh redirection symbols hain:

| 🔣 Operator | 📋 Meaning | 📝 Example |
|------------|-----------|-----------|
| `-eq` | Equals | `$x -eq 5` |
| `-ne` | Not Equal | `$x -ne 0` |
| `-gt` | Greater Than | `$x -gt 10` |
| `-lt` | Less Than | `$x -lt 100` |
| `-ge` | Greater or Equal | `$x -ge 1` |
| `-le` | Less or Equal | `$x -le 50` |
| `-like` | Wildcard match | `$name -like "*svc*"` |
| `-match` | Regex match | `$ip -match "^\d{1,3}\."` |
| `-contains` | Array contains | `$arr -contains "DC01"` |

==**Exam Tip:** PowerShell mein `>` redirection ke liye use hota hai (file mein output bhejne ke liye), comparison ke liye nahi. Hamesha `-eq`, `-gt`, etc. use karo.==

---

### 7. Aliases — Shortcuts for Long Cmdlets

> [!info] Key Concept
> Aliases short names hain long cmdlets ke liye. Linux aur CMD users ke liye familiar commands bhi kaam karte hain — lekin internally PowerShell cmdlet hi chalti hai.

| 🔤 Alias | 📜 Full Cmdlet | 🐧 Linux Equivalent |
|---------|---------------|---------------------|
| `dir` | `Get-ChildItem` | `ls` |
| `ls` | `Get-ChildItem` | `ls` |
| `cd` | `Set-Location` | `cd` |
| `cls` | `Clear-Host` | `clear` |
| `cat` | `Get-Content` | `cat` |
| `select` | `Select-Object` | — |
| `where` | `Where-Object` | — |
| `foreach` | `ForEach-Object` | — |
| `sort` | `Sort-Object` | `sort` |

```powershell
# Kisi bhi alias ka full cmdlet name check karo
Get-Alias dir

# Sab aliases ki list
Get-Alias | Sort-Object Name
```

> [!danger] Common Mistake
> Scripts mein aliases use mat karo! `dir` ya `ls` readability kharab karta hai aur cross-platform compatibility tod sakta hai. **Production scripts mein hamesha full cmdlet names likho** — `Get-ChildItem`, `Select-Object`, etc.

---

## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Windows 10/11 ya Windows Server with PowerShell 5.1+
> - PowerShell console access (Run as Administrator for Step 3)
> - Internet connection (for `Update-Help`)

### Step 1: Help System aur Cmdlet Discovery

```powershell
# PowerShell kholo aur version check karo
$PSVersionTable

# Event log se related sab commands dhoondho
Get-Command *EventLog*

# Get-Process ke examples dekho
Get-Help Get-Process -Examples
```

> [!success] Expected Output
> ```
> CommandType     Name                     Version    Source
> -----------     ----                     -------    ------
> Cmdlet          Clear-EventLog           3.1.0.0    Microsoft.PowerShell.Management
> Cmdlet          Get-EventLog             3.1.0.0    Microsoft.PowerShell.Management
> Cmdlet          Write-EventLog           3.1.0.0    Microsoft.PowerShell.Management
> ...
> ```

### Step 2: Object Inspection aur Pipelines

```powershell
# Current PowerShell process ka detail dekho
Get-Process -Id $PID

# Object ke andar kya-kya hai — Properties aur Methods
Get-Process -Id $PID | Get-Member
```

> [!success] Expected Output
> ```
>    TypeName: System.Diagnostics.Process
>
> Name        MemberType     Definition
> ----        ----------     ----------
> Kill        Method         void Kill(), void Kill(bool entireProcessTree)
> Start       Method         bool Start()
> CPU         Property       double CPU {get;}
> Id          Property       int Id {get;}
> Path        Property       string Path {get;}
> ProcessName Property       string ProcessName {get;}
> ...
> ```

**✅ Verify:** Output mein `Path` property aur `Kill` method dhoondho — yeh confirm karta hai ki PowerShell objects return karta hai, text nahi.

```powershell
# Pipeline practice — sirf running services, sorted by display name
Get-Service | Where-Object {$_.Status -eq "Running"} | Sort-Object DisplayName | Select-Object -Property Name, DisplayName
```

### Step 3: Execution Policy Configure Karo

```powershell
# Current policy check karo — sab scopes dikhenge
Get-ExecutionPolicy -List

# RemoteSigned set karo current user ke liye
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser -Force

# Confirm karo — change laga ya nahi
Get-ExecutionPolicy
```

> [!success] Expected Output
> ```
>         Scope ExecutionPolicy
>         ----- ---------------
> MachinePolicy       Undefined
>    UserPolicy       Undefined
>       Process       Undefined
>   CurrentUser    RemoteSigned
>  LocalMachine       Undefined
> ```

### Step 4: Variable aur Pipeline Combo Practice

```powershell
# Variable mein services store karo
$StoppedSvc = Get-Service | Where-Object {$_.Status -eq "Stopped"}

# Count check karo
$StoppedSvc.Count

# Specific property access karo
$StoppedSvc | Select-Object -First 3 Name, Status, StartType
```

---

## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `Get-Command` | Cmdlets/functions search karo | `Get-Command *service*` |
| `Get-Help` | Documentation dekho | `Get-Help Get-Process -Examples` |
| `Update-Help` | Help files download/update karo | `Update-Help -Force` |
| `Get-Member` | Object ke properties/methods dekho | `Get-Service \| Get-Member` |
| `Get-ExecutionPolicy` | Current execution policy check karo | `Get-ExecutionPolicy -List` |
| `Set-ExecutionPolicy` | Execution policy change karo | `Set-ExecutionPolicy RemoteSigned -Scope CurrentUser` |
| `Get-Process` | Running processes ki list | `Get-Process -Name explorer` |
| `Get-Service` | Windows services ki list | `Get-Service -Name wuauserv` |
| `Where-Object` | Pipeline mein filter lagao | `Get-Service \| Where-Object {$_.Status -eq "Running"}` |
| `Select-Object` | Specific properties select karo | `Get-Service \| Select-Object Name, Status` |
| `Sort-Object` | Output ko sort karo | `Get-Process \| Sort-Object CPU -Descending` |
| `Get-Alias` | Alias ka full cmdlet name dekho | `Get-Alias dir` |
| `$PSVersionTable` | PowerShell version info | `$PSVersionTable.PSVersion` |

---

## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| Script run nahi ho rahi: *"cannot be loaded because running scripts is disabled"* | Execution policy `Restricted` hai | `Set-ExecutionPolicy RemoteSigned -Scope CurrentUser -Force` |
| `Get-Help` koi useful info nahi de raha | Help files download nahi hue | `Update-Help -Force -ErrorAction SilentlyContinue` (Admin mein) |
| `pwsh` command not found | PowerShell 7.x installed nahi hai | Microsoft Store ya GitHub se PowerShell 7.x install karo |
| `The term 'xyz' is not recognized` | Cmdlet installed nahi ya module load nahi | `Get-Command *xyz*` se verify karo, `Import-Module` try karo |
| Pipeline output blank aa raha hai | Filter condition galat hai ya koi match nahi | `Where-Object` ke andar `$_.PropertyName` verify karo `Get-Member` se |
| `Set-ExecutionPolicy` access denied | Admin rights nahi hain | PowerShell as Administrator run karo ya `-Scope CurrentUser` use karo |
| Variable mein data nahi aa raha | Assignment `=` ki jagah `-eq` use kiya | `$x = value` (assignment) vs `$x -eq value` (comparison) — dono alag hain |

---

## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Script Execution Blocked

> [!example] Ticket
> "Hi Team, I downloaded a PowerShell script from our SharePoint to automate user onboarding but when I try to run it, I get an error: *'File cannot be loaded because running scripts is disabled on this system.'* Please help!"

**L1 Response:**
1. User se poochho ki kya woh PowerShell as Administrator run kar rahe hain
2. `Get-ExecutionPolicy -List` run karke current policy check karo
3. Agar `Restricted` hai toh — `Set-ExecutionPolicy RemoteSigned -Scope CurrentUser -Force`
4. Agar phir bhi error aaye toh file pe right-click → Properties → "Unblock" checkbox check karo

**Escalation Trigger:** Agar Group Policy se execution policy locked hai (`MachinePolicy` scope mein set hai), toh L2 ko escalate karo.

**L2 Resolution:**
1. `Get-ExecutionPolicy -List` se confirm karo ki `MachinePolicy` scope kya set hai
2. Group Policy Object (GPO) mein jaake policy change karo: `Computer Configuration → Administrative Templates → Windows Components → Windows PowerShell → Turn on Script Execution`
3. GPO update karo: `gpupdate /force`

---

### 🎫 Scenario 2: Bulk Service Status Check

> [!example] Ticket
> "We need a report of all stopped services on 5 servers — DC01, DC02, FS01, FS02, WEB01. Can you generate a CSV by EOD?"

**L1 Response:**
1. PowerShell open karo aur yeh one-liner run karo:

```powershell
# Sab servers ke stopped services ka CSV report
$Servers = @("DC01", "DC02", "FS01", "FS02", "WEB01")
$Servers | ForEach-Object {
    Get-Service -ComputerName $_ | Where-Object {$_.Status -eq "Stopped"}
} | Select-Object MachineName, Name, DisplayName, Status |
Export-Csv -Path "C:\Reports\StoppedServices.csv" -NoTypeInformation
```

2. CSV file user ko email karo

**Escalation Trigger:** Agar kisi server pe WinRM nahi chal raha aur connection fail ho raha hai.

**L2 Resolution:** Target server pe `Enable-PSRemoting -Force` run karo, firewall rule check karo for WinRM (TCP 5985/5986).

---

### 🎫 Scenario 3: User PowerShell Version Confusion

> [!example] Ticket
> "I installed PowerShell 7 but when I open PowerShell from Start Menu, it still shows version 5.1. Is my installation broken?"

**L1 Response:**
1. Samjhao ki Windows PowerShell 5.1 aur PowerShell 7.x **dono alag-alag installed rehte hain**
2. PowerShell 7 ke liye Start Menu mein "PowerShell 7" ya "pwsh" search karo
3. Version verify karo: `$PSVersionTable.PSVersion`
4. Shortcut: Terminal mein `pwsh` type karo — yeh PowerShell 7 launch karega

**Escalation Trigger:** Koi escalation nahi — yeh informational resolution hai.

---

## 🎤 Interview Questions

> [!question] Q1: PowerShell aur CMD mein kya fundamental difference hai?
> **Answer:** CMD text-based output deta hai — flat strings jo screen pe print hoti hain. PowerShell **structured .NET objects** return karta hai jisme properties (labels) aur methods (actions) hote hain. Iska matlab hai ki PowerShell mein text parsing ki zaroorat nahi — directly object properties access kar sakte ho. PowerShell .NET Framework pe built hai aur ek full scripting language hai with loops, functions, error handling, aur remoting capabilities.

> [!question] Q2: Execution Policy kya hai? Kya yeh ek security feature hai?
> **Answer:** Execution Policy ek **convenience feature** hai jo accidental script execution rokti hai — ==yeh security boundary nahi hai.== Koi bhi user ise bypass kar sakta hai (e.g., `powershell -ExecutionPolicy Bypass -File script.ps1`). Main policies hain: **Restricted** (default, no scripts), **AllSigned** (signed scripts only), **RemoteSigned** (local scripts ok, internet scripts must be signed), **Unrestricted** (all scripts with warning), **Bypass** (no restrictions). Admin workstations ke liye **RemoteSigned** recommended hai.

> [!question] Q3: `Get-Member` ka kya use hai?
> **Answer:** `Get-Member` kisi bhi object ke **Properties** (data fields) aur **Methods** (actions) dikhata hai. Yeh PowerShell ka X-ray machine hai. Jab bhi koi naya cmdlet use karo, uska output `Get-Member` mein pipe karo — tumhe pata chal jaayega ki output ke andar kya-kya available hai. Example: `Get-Process | Get-Member` se tum dekhoge ki process object mein `CPU`, `Id`, `Path`, `Kill()` etc. available hain.

> [!question] Q4: Pipeline mein `$_` kya represent karta hai?
> **Answer:** `$_` (ya `$PSItem`) **current pipeline object** hai — *jo object abhi conveyor belt pe chal raha hai.* Jab tum `Where-Object {$_.Status -eq "Running"}` likhte ho, toh `$_` har service object ko ek-ek karke represent karta hai while filter apply ho raha hai. Yeh PowerShell 3.0+ mein `$PSItem` se bhi access ho sakta hai.

> [!question] Q5: PowerShell 5.1 aur 7.x mein kya difference hai?
> **Answer:** PowerShell 5.1 (Windows PowerShell) **.NET Framework** pe built hai aur sirf Windows pe chalta hai — executable `powershell.exe` hai. PowerShell 7.x **.NET Core** pe built hai, **cross-platform** hai (Windows, Linux, macOS) — executable `pwsh.exe` hai. Dono side-by-side install ho sakte hain. 7.x mein naye features hain jaise ternary operator, pipeline chain operators (`&&`, `||`), aur better error handling.

> [!question] Q6: Production scripts mein aliases kyun use nahi karne chahiye?
> **Answer:** Aliases readability kharab karte hain — `gci` ya `ls` dekhke naye team member ko samajh nahi aayega ki kya ho raha hai. Cross-platform compatibility bhi tod sakte hain — `ls` Linux pe native command hai, PowerShell mein alias hai. **Best practice:** Scripts mein hamesha full cmdlet names likho (`Get-ChildItem`, `Select-Object`) taaki code self-documenting aur portable rahe.

> [!question] Q7: Kisi cmdlet ke baare mein kuch nahi pata — kaise seekhoge bina Google kiye?
> **Answer:** PowerShell ke built-in **discovery trio** use karo:
> 1. `Get-Command *keyword*` — related cmdlets dhoondho
> 2. `Get-Help <cmdlet> -Examples` — usage examples dekho
> 3. `<cmdlet> | Get-Member` — output object ke andar kya hai dekho
>
> ==Yeh teeno commands yaad karo — interview mein bonus points milenge.==

---

## 🔗 Related Notes

- [[02-Operating-Systems/04-Linux-RHEL/L-02 Command Line Basics|L-02 Command Line Basics]] — Linux CLI commands se comparison, bash vs PowerShell
- [[05-Automation-and-Ticketing/10-Scripting-PowerShell/PS-02 PowerShell for Active Directory|PS-02 PowerShell for Active Directory]] — AD DS mein pipeline commands apply karna
- [[05-Automation-and-Ticketing/10-Scripting-PowerShell/PS-03 PowerShell for System Administration|PS-03 PowerShell for System Administration]] — Remote cmdlets aur server management
- [[05-Automation-and-Ticketing/10-Scripting-PowerShell/PS-04 PowerShell Scripting|PS-04 PowerShell Scripting]] — Advanced `.ps1` script blocks aur structured scripting
