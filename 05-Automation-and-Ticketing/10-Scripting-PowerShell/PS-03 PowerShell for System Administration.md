---
tags: [sysadmin, powershell, administration, remote-management]
difficulty: Intermediate
lab-required: Yes
read-time: 15 mins
---

# PS-03: PowerShell for System Administration

> [!abstract] Overview
> This note covers Windows system administration using PowerShell. It details process and service management, event log analysis (`Get-WinEvent`), remote execution (`Invoke-Command`/WinRM), and CIM hardware inventory queries.

---
## Concept
Think of system administration using PowerShell like having a central command center and teleportation pad. 
Instead of a technician walking down the hallway to sit at a user's desk (RDP) to check their task manager, stop a service, or read their system logs, the administrator sits at their own desk. 

- **Invoke-Command** is like teleporting a command payload directly into the RAM of 50 remote servers simultaneously, executing the task, and gathering the results in a unified table.
- **Enter-PSSession** is like opening a secure holographic portal (interactive remote shell) to work inside a remote server's console directly without RDP desktop overhead.

*Seedha simple mein: PowerShell system administration ke zariye hum local and remote machines ko manage karte hain. WinRM (Windows Remote Management) enable karke hum `Invoke-Command` aur `Enter-PSSession` run karte hain. `Get-WinEvent` log analysis ke liye use hota hai.*

---
## Technical Deep Dive

### 1. Process and Service Management
- **Processes:**
  - Start: `Start-Process notepad.exe`
  - Stop: `Stop-Process -Name notepad -Force`
- **Services:**
  - Get: `Get-Service -Name wuauserv` (Windows Update service)
  - Control: `Start-Service`, `Stop-Service`, `Restart-Service`.

### 2. Log Auditing: Get-EventLog vs. Get-WinEvent
- **`Get-EventLog` (Deprecated):** Legacy cmdlet. Only queries classic Windows event logs (Application, Security, System). Slow performance.
- **`Get-WinEvent` (Modern Standard):** Highly optimized, queries both classic logs and modern XML-structured application/service log channels (e.g., AppLocker, Sysmon, Windows Defender).
  - *Query Filter XML:* Uses structured hash tables for high performance, reducing filter overhead on the server:
    `Get-WinEvent -FilterHashtable @{LogName='System'; Id=1074}` (Finds shutdown/reboot events).

### 3. Remote Administration: WinRM & WS-Management
PowerShell Remoting uses the **WS-Management (WS-Man)** protocol running over **WinRM (Windows Remote Management)**.
- **Ports:** HTTP: **5985** | HTTPS: **5986**.
- **`Enter-PSSession`:** Establishes a one-to-one interactive remote CLI console session:
  `Enter-PSSession -ComputerName SVR-FS01`
- **`Invoke-Command` (One-to-Many Execution):** Transmits script blocks to multiple remote hosts in parallel:
  `Invoke-Command -ComputerName SVR-DC01, SVR-FS01 -ScriptBlock {Restart-Service spooler}`

### 4. Hardware Inventory: CIM vs. WMI
- **WMI (`Get-WmiObject` - Deprecated):** Legacy DCOM-based protocol. Slow, firewalls-unfriendly.
- **CIM (`Get-CimInstance` - Modern):** Uses standard WS-Man protocols. Fast, runs over standard port 5985.
  - *Example:* `Get-CimInstance -ClassName Win32_OperatingSystem | Select-Object Caption, Version, OSArchitecture`

---
## Windows PowerShell Remoting Setup Commands

### Enabling WinRM on Target Server
Must be run as Administrator on the target machine:
```powershell
# Enable PowerShell Remoting and configure firewall exceptions
Enable-PSRemoting -Force
```

### Remote Execution Syntax
```powershell
# Run a system diagnostic command on multiple remote hosts
Invoke-Command -ComputerName "SVR-FS01", "SVR-DHCP02" -ScriptBlock {
    Get-Service -Name "spooler" | Select-Object MachineName, Name, Status
}
```

---
## Lab — Step by Step
> [!info] Lab Setup Needed
> Two Windows Server VMs (`SVR-DC01` as administrator console, and `SVR-DHCP02` as target node) on the same domain, with WinRM enabled on both.

### Step 1: Open an Interactive Remote Session
1. On `SVR-DC01`, open PowerShell as Administrator.
2. Connect to the secondary server `SVR-DHCP02`:
   ```powershell
   Enter-PSSession -ComputerName "SVR-DHCP02"
   ```
3. **Verify:** Check your shell prompt. It should change to `[SVR-DHCP02]: PS C:\Users\Administrator\Documents>`. Any command you run now executes inside the RAM of the remote server.
4. Exit the session:
   ```powershell
   Exit-PSSession
   ```

### Step 2: Query Remote Hardware Details using CIM
1. Execute a query to retrieve remote system motherboard and processor details:
   ```powershell
   Invoke-Command -ComputerName "SVR-DHCP02" -ScriptBlock {
       Get-CimInstance -ClassName Win32_Processor | Select-Object DeviceID, Name, NumberOfCores
   }
   ```

### Step 3: Audit System Reboots using Get-WinEvent
1. Check when the target server was last rebooted, and identify which user triggered the action (Event ID 1074):
   ```powershell
   Invoke-Command -ComputerName "SVR-DHCP02" -ScriptBlock {
       Get-WinEvent -FilterHashtable @{LogName='System'; Id=1074} -MaxEvents 5 | 
       Select-Object TimeCreated, Message | Format-Table -Wrap
   }
   ```

---
## Related Notes
- [[03-Identity-and-Core-Services/05-Windows-Server/WS-01 Windows Server 2022 Introduction|WS-01 Windows Server 2022 Introduction]] — Configuring initial host parameters.
- [[03-Identity-and-Core-Services/06-Active-Directory/WS-05 Group Policy — Complete Guide|WS-05 Group Policy — Complete Guide]] — Deploying WinRM enable settings via GPO.
- [[05-Automation-and-Ticketing/10-Scripting-PowerShell/PS-01 PowerShell Fundamentals|PS-01 PowerShell Fundamentals]] — Pipelines and object filtering.
- [[05-Automation-and-Ticketing/10-Scripting-PowerShell/PS-04 PowerShell Scripting|PS-04 PowerShell Scripting]] — Handling remote execution errors.

