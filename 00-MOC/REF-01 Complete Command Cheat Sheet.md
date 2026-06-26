---
tags: [desktop-support, systems, commands, L2]
aliases: [complete-command-cheat-sheet, ref-01]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #none
---

# REF-01: Complete Command Cheat Sheet

> [!abstract] Overview
> This reference note compiles essential Windows CMD, PowerShell, Linux Bash, and network administration commands. It serves as a lookup guide for configuration, diagnostics, automation, and infrastructure maintenance.

---
## Concept
Think of this cheat sheet as a sysadmin's multi-tool pocketknife. You don't need to forge a new blade every time you need to cut a wire or open a box; instead, you pull out the exact tool (command) designed for the task.
*Seedha simple mein: Yeh cheat sheet ek complete sysadmin toolkit hai, jisme Linux, Windows, Active Directory aur network configuration ke saare major commands aur unke practical examples ek hi jagah par hain.*

---
## Technical Deep Dive

### 1. Networking Diagnostics (Cross-Platform)
Commands used to troubleshoot connectivity, DNS resolution, routing, and open ports:

#### Ping
Tests network connectivity.
- CMD/PS: `ping -t 8.8.8.8` (Continuous ping)
- Linux: `ping -c 4 8.8.8.8` (Limit to 4 packets)

#### Traceroute
Traces the route packets take to a destination.
- Windows: `tracert 8.8.8.8`
- Linux: `traceroute 8.8.8.8`

#### NSLookup
Queries DNS servers for records.
- Windows/Linux: `nslookup -type=mx google.com`

#### Netstat / SS
Displays network connections and listening ports.
- Windows: `netstat -ano | findstr :80`
- Linux: `ss -tulnp | grep :80`

#### IP Configuration
Displays network adapter settings.
- Windows: `ipconfig /all`
- Linux: `ip addr show` or `ip link`

---

### 2. Linux System Administration (RHCSA Essentials)
Essential commands for managing files, permissions, processes, and storage in RHEL/CentOS/Ubuntu:

#### File Management
- `ls -la` (List all files with details)
- `mkdir -p /var/www/html` (Create nested directories)
- `rm -rf /tmp/scratch` (Force remove directory)
- `cp -rp src/ dest/` (Copy preserving permissions)
- `find /var/log -name "*.log" -mtime -7` (Find logs modified in last 7 days)

#### Permissions & Ownership
- `chmod 755 script.sh` (Set rwxr-xr-x)
- `chown -R apache:apache /var/www` (Change owner/group recursively)
- `setfacl -m u:jdoe:rwx file.txt` (Grant specific user access)

#### Process & Service Management
- `ps aux | grep nginx` (List active nginx processes)
- `kill -9 <PID>` (Force kill a process)
- `systemctl restart sshd` (Restart SSH service)
- `journalctl -u sshd --since "1 hour ago"` (Read SSH service logs)

#### Storage & Filesystems
- `df -h` (Display disk space usage)
- `du -sh /var/log` (Display directory size)
- `mount -a` (Mount all filesystems in /etc/fstab)
- `pvcreate /dev/sdb`, `vgcreate web_vg /dev/sdb`, `lvcreate -n web_lv -L 10G web_vg` (LVM creation)

---

### 3. Windows Command Prompt (CMD) & Utility Tools
Essential command-line utilities for local Windows maintenance:

#### File & Disk System Check
Advanced content only — basics in [[Basic Windows troubleshooting]]
- `sfc /scannow` (Scan system file integrity)
- `dism /online /cleanup-image /restorehealth` (Restore system image files)
- `chkdsk c: /f /r` (Scan and repair file system errors)

#### Security & Policy Enforcement
- `gpupdate /force` (Force Group Policy update)
- `gpresult /r` (Display Applied GPOs summary)

#### Identity & Authentication
- `whoami /groups` (List current user group memberships)
- `net user administrator /active:yes` (Enable local administrator account)

---

### 4. PowerShell Administration (Sysadmin & AD)
Cmdlets used to automate administrative tasks and query Active Directory:

#### Active Directory User Management
Advanced content only — basics in [[Basic AD user management]]
- `Get-ADUser -Filter "Enabled -eq `$false" -Properties Name, LastLogonDate` (Find disabled accounts)
- `New-ADUser -Name "John Doe" -UserPrincipalName "jdoe@company.com" -Enabled $true` (Create user)
- `Unlock-ADAccount -Identity "jdoe"` (Unlock account)
- `Add-ADGroupMember -Identity "Domain Admins" -Members "jdoe"` (Add user to group)

#### Remoting & Management
- `Enter-PSSession -ComputerName "Server01"` (Start interactive session)
- `Invoke-Command -ComputerName "Server01", "Server02" -ScriptBlock { Get-Service | Where-Object {$_.Status -eq "Running"} }` (Run scripts remotely)
- `Get-CimInstance -ClassName Win32_OperatingSystem | Select-Object Caption, Version` (Retrieve OS information)

---

### 5. Azure PowerShell & CLI (AZ-104 Reference)
Cloud resource management commands:

#### Azure CLI
- `az login` (Authenticate to Azure)
- `az vm list -g "Prod-RG" -o table` (List VMs in a resource group)
- `az storage blob upload -c "logs" -f "app.log" -n "app.log" --account-name "storprod"` (Upload file to container)

#### Azure PowerShell
- `Connect-AzAccount` (Log in)
- `Get-AzVM -ResourceGroupName "Prod-RG" | Start-AzVM` (Start all VMs in group)
- `New-AzResourceGroup -Name "Dev-RG" -Location "EastUS"` (Create resource group)
---
### 6. DevOps, Monitoring, & Automation Essentials

#### Docker Commands
- `docker run -d -p 8080:80 --name my-web nginx:alpine` (Run container in background)
- `docker ps -a` (List all local containers, including stopped ones)
- `docker logs -f my-web` (Stream container logs)
- `docker exec -it my-web sh` (Open interactive shell inside container)
- `docker system prune -a --volumes -f` (Clean up unused container resources)

#### Terraform Commands
- `terraform init` (Initialize project and download provider plugins)
- `terraform plan -out=tfplan` (Generate and save execution plan)
- `terraform apply tfplan` (Apply plan to provision resources)
- `terraform destroy --auto-approve` (Tear down all managed infrastructure)

#### Python One-Liners for Sysadmins
- `python3 -m http.server 8000` (Start quick local HTTP web server)
- `python3 -c "import json, sys; print(json.load(sys.stdin)['ip'])"` (Parse JSON from pipe and print a key)
- `python3 -c "import urllib.request; print(urllib.request.urlopen('https://api.ipify.org').read().decode())"` (Get public IP)

#### Monitoring Commands
- `curl http://localhost:9100/metrics` (Check Node Exporter metrics endpoint)
- `promtool check config /etc/prometheus/prometheus.yml` (Validate Prometheus config syntax)
- `zabbix_get -s 192.168.1.50 -k system.cpu.load` (Query Zabbix agent metrics from manager)

#### CI/CD Pipeline Execution
- `act -j build-and-test` (Run GitHub Actions workflows locally using Docker)
- `jenkins-cli.jar -s http://localhost:8080/ build Test-Pipeline -s -v` (Trigger Jenkins build via CLI)

---
## Step-by-Step Lab
> [!info] Lab Setup Needed
> A workstation running Windows 11 with PowerShell installed, and a local or virtual Linux host running SSH.

### Step 1: Combine CMD and Network Diagnostics to Verify Local Port Listening
On your Windows machine, locate which process is listening on Port 135:
```cmd
netstat -ano | findstr LISTENING | findstr :135
```

### Step 2: Use PowerShell to Collect System Diagnostics and Export to CSV
Write a PowerShell command to collect service status and output it:
```powershell
Get-Service | Where-Object {$_.Status -eq "Stopped" -and $_.StartType -eq "Automatic"} | Select-Object Name, DisplayName, StartType | Export-Csv -Path "$Home\StoppedAutoServices.csv" -NoTypeInformation
```

### Step 3: Run Linux Command Sequence to Archive Logs and Check Disk Space
Log into your Linux machine via SSH and run this sequence:
```bash
tar -czf /tmp/logs-backup-$(date +%F).tar.gz /var/log/*.log
df -h /tmp
```

### Step 4: Verify Command Execution
Check that the output files (`StoppedAutoServices.csv` on Windows, and the `.tar.gz` archive on Linux) exist in their target folders.

---
## Commands Reference
```bash
# General Command Syntax Help
man ls                   # Linux manual lookup
get-help Get-Service -examples # PowerShell help with examples
netstat /?               # CMD command syntax documentation
```

---
## Troubleshooting Scenarios

**Scenario 1:**
- Problem: Running a PowerShell cmdlet (like `Get-ADUser`) returns the error "The term 'Get-ADUser' is not recognized as the name of a cmdlet".
- Root Cause: The Active Directory PowerShell module is not installed on the system, or Remote Server Administration Tools (RSAT) are missing.
- Fix:
  1. Open PowerShell as Administrator.
  2. Install RSAT Active Directory tools:
     `Add-WindowsCapability -Online -Name "Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0"`
  3. Import the module: `Import-Module ActiveDirectory` and rerun the command.

**Scenario 2:**
- Problem: Attempting to run a Linux storage command like `pvcreate` returns "Permission denied" or "Command not found", despite logged in as a normal user with sudo privileges.
- Root Cause: Standard user path limits do not include administrative directories (`/sbin` or `/usr/sbin`), or command was executed without administrative elevation (`sudo`).
- Fix:
  1. Prefix the command with `sudo`: `sudo pvcreate /dev/sdb`.
  2. If still not found, check the path: `sudo /sbin/pvcreate /dev/sdb`.

---
## Common Mistakes
> [!warning] Avoid These
> Running commands like `rm -rf /` or `Get-Process | Stop-Process` without verified filtering parameters, risking system corruption.
> Using obsolete tools like `nslookup` for scripting instead of `Resolve-DnsName` in PowerShell, which handles structured object properties better.
> Storing cleartext passwords inside script arguments instead of using secure credential inputs (e.g., `Get-Credential` or Key Vault references).

---
## Pro Tips
> [!tip] Field Experience
> Use `Ctrl + R` in both Windows PowerShell and Linux Bash terminals. It triggers a search of your command history, allowing you to quickly locate complex commands you executed previously.
> When running risky commands (like deleting files or stopping services) in PowerShell, append the `-WhatIf` flag. This displays what actions the command would take without actually executing them.

---
## Quick Revision Table
| # | Command Class | Command | One Line Summary |
|---|---------------|---------|-----------------|
| 1 | Networking | `nslookup -type=mx` | Queries the MX (Mail Exchange) record for a target domain. |
| 2 | Linux | `journalctl -u <svc> -f` | Follows the log output of a specific systemd service in real-time. |
| 3 | PowerShell | `Invoke-Command` | Runs a script or cmdlet on remote computers simultaneously. |
| 4 | Windows CMD | `gpupdate /force` | Re-evaluates and pulls down active GPOs from the Domain Controller. |
| 5 | Azure CLI | `az vm list -o table` | Formats and displays Azure Virtual Machine properties in a table. |

---
## Interview Q&A

**Q1: Explain the difference between running netstat and ss commands, and when you would use each.**
A: Both tools check network socket statistics. `netstat` is a legacy cross-platform utility available on both Windows and Linux. `ss` is a modern Linux utility that retrieves socket statistics directly from the kernel space, making it significantly faster and capable of displaying more detailed protocol information than `netstat`. I use `ss` on modern Linux hosts, and `netstat` on Windows systems.

**Q2: A server is slow, and you suspect a network port conflict. How would you identify the process holding a specific port?**
A:
- **Situation**: A web server failed to start because its target port was already in use.
- **Task**: I needed to identify and terminate the process binding the port.
- **Action**: On Windows, I ran `netstat -ano | findstr :80` to find the process ID (PID). On Linux, I ran `ss -lptn 'sport = :80'` to get the process name and PID.
- **Result**: I identified that a rogue testing service was holding the port. I terminated it using `kill -9 <PID>` on Linux (or `taskkill /F /PID <PID>` on Windows) and restarted the web server.

**Q3: Why is Get-WinEvent preferred over Get-EventLog in modern PowerShell scripting?**
A: `Get-WinEvent` is preferred because it uses the modern Windows Event Log API, which allows it to query both classic event logs and newer event provider logs. It is also faster than `Get-EventLog` because it can filter events on the remote host before transferring data over the network, using XML-based queries.

---
## Seedha Simple Mein
*Seedha simple mein: Yeh cheat sheet sysadmin ke daily use wale networking, Linux, Windows, PowerShell, Azure, Docker, and automation commands ka ek primary center collection hai. Iske commands ko direct system terminal me copy-paste karke dynamic results aur service configurations verify kiye ja sakte hain.*

---
## Related Notes
- [[02-Operating-Systems/04-Linux-RHEL/L-02 Command Line Basics|L-02 Command Line Basics]] — Covers Linux CLI syntax and core tools.
- [[05-Automation-and-Ticketing/10-Scripting-PowerShell/PS-01 PowerShell Fundamentals|PS-01 PowerShell Fundamentals]] — Details PowerShell pipeline architecture.
- [[00-MOC/REF-03 Troubleshooting Decision Trees|REF-03 Troubleshooting Decision Trees]] — Implements these commands in diagnostic flows.

