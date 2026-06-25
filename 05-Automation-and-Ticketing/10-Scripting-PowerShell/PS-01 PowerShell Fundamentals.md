---
tags: [sysadmin, powershell, CLI, scripting, fundamentals]
difficulty: Beginner
lab-required: Yes
read-time: 12 mins
---

# PS-01: PowerShell Fundamentals

> [!abstract] Overview
> This note covers PowerShell baseline administration. It details differences between CMD and PowerShell versions, execution policy scopes, the help system, object pipelines, variables, operators, and aliases.

---
## Concept
Think of Command Prompt (CMD) as an old manual typewriter. When you type a command, it spits out flat text lines onto the paper (the screen). If you want to grab the third word on the second line, you have to write complex text filtering macros (like `findstr` or `for /f`) to parse the characters.

PowerShell is an automated assembly line handling **3D physical objects** rather than flat text. 
When you query a process list (`Get-Process`), PowerShell does not output text characters. 

It outputs a collection of physical object blocks. Each block has built-in labels (Properties like `CPU`, `ID`, `ProcessName`) and built-in actions it can take (Methods like `Kill()`, `Start()`). 

If you want to sort, filter, or destroy these blocks, you pipe (`|`) them down the conveyor belt to the next workstation, where you reference their labels directly without parsing any text characters.

*Seedha simple mein: PowerShell cmdlets objects return karte hain, na ki simple text output. Iski execution policies script run authorization ko control karti hain. Pipeline (`|`) ke zariye ek cmdlet ka object output dusre cmdlet ka input banta hai.*

---
## Technical Deep Dive

### 1. PowerShell vs. CMD
- **CMD (Command Prompt):** Legacy shell based on MS-DOS. Outputs flat text (string data). Lacks built-in object-oriented capabilities or complex scripting logic.
- **PowerShell:** Object-oriented task automation engine and scripting language built on the .NET framework. Commands return structured objects.

### 2. PowerShell Version Evolution
- **PowerShell 5.1 (Windows PowerShell):** Built into the Windows operating system. Runs exclusively on the Windows-only .NET Framework.
- **PowerShell 7.x (PowerShell Core):** Cross-platform version running on .NET Core. Supports Windows, Linux, and macOS. The executable name is `pwsh` instead of `powershell`.

### 3. Execution Policies (Script Security Gates)
Execution policies are not safety fences protecting against hackers; they are guidelines preventing users from running unapproved scripts accidentally.
- **Restricted:** The default setting. No scripts can be executed. Only individual command inputs are allowed.
- **AllSigned:** Only scripts signed by a trusted publisher certificate can run.
- **RemoteSigned:** Locally written scripts can run without signatures, but scripts downloaded from the internet must be signed by a trusted publisher. Standard for admin workstations.
- **Unrestricted:** All scripts can run, but warning prompts display for files downloaded from the internet.
- **Bypass:** Nothing is blocked; no warning prompts display. Used in automated deployment agents.
- *Command:* `Set-ExecutionPolicy RemoteSigned -Scope LocalMachine`

### 4. Navigating the Command Engine
- **`Get-Command`:** Searches for installed cmdlets, functions, or aliases:
  `Get-Command *service*`
- **`Get-Help`:** The documentation reference system:
  `Get-Help Get-Service -Detailed` or `Get-Help Get-Service -Examples`.
  - *Update System:* Run `Update-Help` as Administrator to download the latest manual pages.
- **`Get-Member` (The Object Inspector):** Pipes a command to `Get-Member` to view all available Properties (labels) and Methods (actions) of the output object:
  `Get-Process -Name explorer | Get-Member`

### 5. The Object Pipeline
The pipeline (`|`) passes complete .NET objects between cmdlets:
- `Get-Service | Where-Object {$_.Status -eq "Running"} | Select-Object -Property Name, Status`
  - **`$_`** represents the current object passing down the conveyor belt.

### 6. Variables, Typing, and Operators
- **Variables:** Labeled with `$` prefix: `$MyVar = "Active"`.
- **Casting (Enforcing Type):** `[int]$Count = 5` or `[string]$IP = "10.0.0.1"`.
- **Comparison Operators:** PowerShell does not use standard symbols (like `>`, `<`) because they clash with redirection symbols.
  - `-eq` (equals), `-ne` (not equal), `-gt` (greater than), `-lt` (less than), `-like` (wildcard match, e.g., `-like "*svc*"`), `-match` (regex match).
- **Aliases:** Shortcuts for long cmdlets:
  - `dir` and `ls` are aliases for `Get-ChildItem`.
  - `select` is an alias for `Select-Object`.
  - Check aliases using `Get-Alias`.

---
## Lab — Step by Step
> [!info] Lab Setup Needed
> A Windows client workstation and PowerShell console access.

### Step 1: Help and Cmdlet Discovery
1. Open PowerShell.
2. Search for all cmdlets related to event logs:
   ```powershell
   Get-Command *EventLog*
   ```
3. Read help documentation and view examples for the `Get-Process` cmdlet:
   ```powershell
   Get-Help Get-Process -Examples
   ```

### Step 2: Object Inspection and Pipelines
1. Fetch process details for the shell itself:
   ```powershell
   Get-Process -Id $PID
   ```
2. Inspect the properties and methods of this process object:
   ```powershell
   Get-Process -Id $PID | Get-Member
   ```
   **Verify:** Locate the `Path` property and the `Kill` method in the output list.
3. Write a pipeline to list only running services, sorted by status:
   ```powershell
   Get-Service | Where-Object {$_.Status -eq "Running"} | Sort-Object DisplayName | Select-Object -Property Name, DisplayName
   ```

### Step 3: Configure Script Execution Policy
1. Check your current active execution policy:
   ```powershell
   Get-ExecutionPolicy -List
   ```
2. Modify the execution policy to allow local scripting:
   ```powershell
   Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser -Force
   ```
3. Re-run `Get-ExecutionPolicy` to confirm the change.

---
## Related Notes
- [[02-Operating-Systems/04-Linux-RHEL/L-02 Command Line Basics|L-02 Command Line Basics]] — Comparison to Linux CLI commands.
- [[05-Automation-and-Ticketing/10-Scripting-PowerShell/PS-02 PowerShell for Active Directory|PS-02 PowerShell for Active Directory]] — Applying pipeline commands to AD DS.
- [[05-Automation-and-Ticketing/10-Scripting-PowerShell/PS-03 PowerShell for System Administration|PS-03 PowerShell for System Administration]] — Running remote cmdlets.
- [[05-Automation-and-Ticketing/10-Scripting-PowerShell/PS-04 PowerShell Scripting|PS-04 PowerShell Scripting]] — Structuring advanced `.ps1` script blocks.

