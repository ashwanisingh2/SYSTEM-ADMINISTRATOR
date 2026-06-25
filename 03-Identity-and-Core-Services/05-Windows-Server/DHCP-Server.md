---
tags: [desktop-support, windows-server, dhcp, L2]
aliases: [dhcp-server-role, windows-dhcp]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #md-102
---

# DHCP Server

---

## Concept Overview
- **What it is**: The Windows Server DHCP Server is a network service role that dynamically manages and automates the allocation of IP addresses and configuration parameters (DNS, default gateway) to client devices on the network.
- **Why it matters for a support engineer**: A support engineer must know how to configure and manage DHCP scopes, create static reservations, set up failover redundancy, and restore the DHCP database to prevent network-wide outages.
- **Where you encounter this in real job**: Creating new IP subnet pools for department expansions, configuring high-availability failover configurations between servers, setting up network printer IP reservations, and recovering corrupted DHCP databases.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Creates static IP reservations, checks active client lease logs, and runs basic renewals.
  - **L2**: Creates new DHCP scopes, configures scope options, and manages failover replication.
  - **L3**: Designs subnet routing scopes, configures DHCP failover clusters, and manages database backup/restore operations during server migrations.

---

## Technical Deep Dive

### 1. Active Directory Authorization
Before a Windows Server DHCP server can distribute IP addresses, it must be **Authorized in Active Directory**. 
- **Mechanism**: The DHCP service queries AD DS during startup. If the server's IP is not on the authorized list, the DHCP service automatically stops (Event ID 1044). This prevents unauthorized "rogue" DHCP servers from disrupting the network.

### 2. DHCP Scopes & Database Architecture
- **DHCP Database**: Stored as a Microsoft Jet database engine file (`dhcp.mdb`) in the `%systemroot%\System32\dhcp\` directory. Transaction logs are written to `.log` files to track database changes before writing them to the database.
- **Scope Parameters**:
  - **Address Pool**: The range of distributable IP addresses.
  - **Exclusion Range**: IPs within the pool that the server is blocked from distributing (e.g., reserving `.1` to `.20` for routers and switches).
  - **Lease Duration**: The time a client holds an IP address before it must renew. Defaults to 8 days.

### 3. DHCP Failover Configurations
Windows Server 2012+ features native DHCP Failover to support high availability:
- **Load Balance Mode (Active-Active)**: Both servers actively lease IP addresses from the same scope, splitting the load based on a configurable ratio (e.g., 50/50).
- **Hot Standby Mode (Active-Passive)**: One server acts as the primary lease handler, while the standby server monitors status. If the primary server goes offline, the standby server takes over leasing for the subnet.
- **MCLT (Maximum Client Lead Time)**: The temporary lease duration granted by the standby server to new clients before database replication is restored.

---

## Commands & Syntax

### PowerShell
```powershell
# Create a new DHCP scope on Windows Server
Add-DhcpServerv4Scope -Name "Finance Scope" -StartRange 192.168.20.100 -EndRange 192.168.20.200 -SubnetMask 255.255.255.0 -State Active

# Add a DNS server option to the newly created scope (Option 006)
Set-DhcpServerv4OptionValue -ScopeId 192.168.20.0 -OptionId 6 -Value "192.168.10.10"
```

### CMD / Run Box
```cmd
REM Launch the DHCP Management console directly
dhcpmgmt.msc
REM Stop the DHCP Server service from the command prompt
net stop dhcpserver
```

### GUI Path
> Server Manager -> Tools -> **DHCP** -> Expand Server name -> Right-click **IPv4** -> Select **New Scope**.

### Important Registry Paths
```
HKLM\SYSTEM\CurrentControlSet\Services\DHCPServer\Parameters\
HKLM\SYSTEM\CurrentControlSet\Services\DHCPServer\Configuration\
```

### Key Event IDs
DHCP audits events in the log path: `%systemroot%\System32\dhcp\dhcpSrvLog-*`

| Event ID | Meaning | Where to Check |
|----------|---------|----------------|
| 1044 | DHCP server is unauthorized in Active Directory | System Log |
| 1010 | The DHCP service encountered a database corruption error | System Log |
| 37 | DHCP Failover connection lost with partner server | System Log |

---

## Real-World Scenarios

### Scenario 1: DHCP Service Stops on Startup (AD Authorization Failure)
**User Complaint:** "I installed the DHCP role on our new backup domain controller. I started the service, but it stops immediately after launching."
**Your First 3 Checks:**
1. Check the Event Viewer System log for Event ID 1044.
2. Verify if the server is authorized in the DHCP console.
3. Check the Active Directory domain connection status.
**Diagnosis Steps:**
1. Open Event Viewer -> System Log -> Filter by Event ID `1044`.
   - Output: `The DHCP/BINL service on the local computer has determined that it is not authorized to start.`
2. The server is not authorized to lease IPs.
3. Open the DHCP console. The server status indicator shows a red down arrow next to the hostname.
**Root Cause:** The new DHCP server was not authorized in Active Directory, causing the DHCP service to shut down automatically for security compliance.
**Fix:** Right-click the DHCP server node in the DHCP console -> Select **Authorize**. Alternatively, use PowerShell:
```powershell
Add-DhcpServerInDC -DnsName "backup-dc.lab.local" -IPAddress 192.168.10.11
```
**Prevention:** Always authorize new DHCP server installations in Active Directory before activating scopes.
**Ticket Close Note:** "Authorized the new DHCP server in Active Directory. Verified the DHCP service started successfully and is active. Closing ticket."

### Scenario 2: Corrupted DHCP Database Recovery
**User Complaint:** "The DHCP console is empty, the service is stopped, and Event Viewer shows a database corruption error. None of the clients are receiving IP leases."
**Your First 3 Checks:**
1. Check the Event Viewer for database error codes (Event ID 1010).
2. Check the backup folder status at `%systemroot%\System32\dhcp\backup`.
3. Stop the DHCP service before attempting recovery.
**Diagnosis Steps:**
1. Open Event Viewer. Event ID 1010 is logged: `The DHCP service failed to load the database.`
2. Stop the service: `net stop dhcpserver`.
3. Check the backup directory:
   - The backup folder contains a copy of `dhcp.mdb` from the automated 60-minute backup task.
**Root Cause:** A system crash caused database write errors, corrupting the active `dhcp.mdb` file.
**Fix: Restore the Database from Backup**
1. Navigate to `%systemroot%\System32\dhcp\`.
2. Rename the corrupt `dhcp.mdb` to `dhcp.mdb.old`.
3. Copy the healthy `dhcp.mdb` from `%systemroot%\System32\dhcp\backup\Jet\new\` to the active `dhcp` folder.
4. Restart the service:
   ```cmd
   net start dhcpserver
   ```
**Prevention:** Ensure the server is connected to a UPS to prevent write errors during power outages.
**Ticket Close Note:** "Restored the DHCP database from the local backup folder. Restarted the DHCP service. Verified scope records and active leases. Closing ticket."

---

## Critical Points

> [!danger] Never Do This
> - Deploy overlapping IP scopes on two separate DHCP servers without configuring a DHCP Failover relationship.
> - Overlapping scopes without coordination cause both servers to lease the same IP address to different devices, resulting in IP address conflict errors.
> - Use the built-in DHCP Failover wizard to sync scopes.

> [!warning] Common Trap
> - Forgetting to configure Option 003 (Router) and Option 006 (DNS) when creating a new DHCP scope.
> - Clients will lease an IP address but will not receive default gateway or DNS settings, leaving them unable to access external networks.
> - Verify scope options are active after scope creation.

> [!tip] Senior Engineer Tip
> - Windows Server DHCP performs automatic database cleanups and backups every 60 minutes. If you suspect database issues, check the `dhcpSrvLog` files located in the `dhcp` folder to view detailed audit logs of lease transactions.

> [!success] Verification Steps
> - Run the command: `Get-DhcpServerv4Scope` on the server.
> - Confirm the status displays **Active** and active lease statistics are recorded.

> [!question] Interview Alert
> - "Explain the difference between Hot Standby and Load Balance modes in DHCP Failover."
> - Answer: "In Hot Standby mode, one server acts as the primary handler for leases, while the standby server takes over only if the primary fails. In Load Balance mode, both servers actively distribute IP addresses from the same pool simultaneously, sharing the load based on a configurable ratio."

---

## Common Mistakes & Fixes
| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| Exclusions within pool ranges | Failing to configure exclusions | Define exclusion ranges for all static IP assignments before activating scopes. |
| Incompatible lease times on Wi-Fi | Leaving default 8-day lease active | Configure short lease times (e.g., 2 to 4 hours) for guest wireless subnets. |
| Unsynchronized failover scopes | Modifying scopes on one node only | Always right-click the modified scope and select **Replicate Scope** to sync changes to the failover partner. |

---

## Lab Exercise

**Objective:** Install the DHCP server role, create a scope with exclusions, set up a failover partner configuration, and verify lease delivery.
**Time Required:** 30 minutes
**Environment Needed:** Two Windows Server 2022 VMs (DC01 and DC02) and a Windows 11 VM.
**Pre-requisites:** Active Directory domain configured, both servers authorized.

**Steps:**
1. Open PowerShell on `DC01` as Administrator. Install the DHCP role:
   ```powershell
   Install-WindowsFeature -Name "DHCP" -IncludeManagementTools
   ```
2. Authorize the server in AD:
   ```powershell
   Add-DhcpServerInDC -DnsName "dc01.lab.local" -IPAddress 192.168.10.10
   ```
3. Create the DHCP Scope:
   ```powershell
   Add-DhcpServerv4Scope -Name "Workstation Scope" -StartRange 192.168.10.100 -EndRange 192.168.10.200 -SubnetMask 255.255.255.0 -State Active
   ```
4. Configure Scope Options:
   ```powershell
   Set-DhcpServerv4OptionValue -ScopeId 192.168.10.0 -OptionId 3 -Value "192.168.10.1"
   Set-DhcpServerv4OptionValue -ScopeId 192.168.10.0 -OptionId 6 -Value "192.168.10.10"
   ```
5. Configure DHCP Failover: On `DC01`, run command to set up failover with `DC02` in Hot Standby:
   ```powershell
   Add-DhcpServerv4Failover -Name "DC-Failover" -PartnerServer "dc02.lab.local" -ScopeId 192.168.10.0 -Mode HotStandby -Role Active -AutoStateTransition $true
   ```
6. Log into the Windows 11 client VM. Run `ipconfig /renew`.
7. Verification: Run verification command on `DC01` to check the client lease:
   ```powershell
   Get-DhcpServerv4Lease -ScopeId 192.168.10.0
   ```

**Success Criteria:** The DHCP failover relationship is established, and the client successfully leases an IP address.
**Common Failures:** Failover setup fails if the partner server's DHCP role is not installed or configured.

---

## Interview Questions & Answers

### Basic (L1 Level)
**Q: Why must a DHCP server be authorized in Active Directory?**
A: A DHCP server must be authorized to prevent unauthorized "rogue" DHCP servers from joining the network and distributing incorrect IP configurations to clients, which causes connection drops.

**Q: Where are the DHCP database and backup files stored by default?**
A: The active database is stored at `%systemroot%\System32\dhcp\dhcp.mdb`. Automated backups are written every 60 minutes to the `%systemroot%\System32\dhcp\backup\` folder.

### Intermediate (L2 Level)
**Q: How does DHCP Failover Hot Standby mode work, and what is the Maximum Client Lead Time?**
A: In Hot Standby mode, the primary server handles all IP leases, and the standby server takes over only if the primary goes offline. The Maximum Client Lead Time (MCLT) is a temporary lease duration (typically 1 hour) granted by the standby server to new clients before it assumes full control of the scope.

### Advanced (L3/Senior Level)
**Q: You need to migrate the DHCP role from an old Windows Server 2012 DC to a new Server 2022 DC. Explain your migration process.**
A:
- **Situation**: A legacy Server 2012 DC hosting active DHCP scopes needed to be decommissioned.
- **Task**: I needed to migrate the DHCP configuration and active lease database to the new Server 2022 DC with minimum downtime.
- **Action**: I opened PowerShell on the old server and exported the configuration:
   `Export-DhcpServer -File C:\dhcp.xml -Leases`.
   I copied `dhcp.xml` to the new server. I installed the DHCP role on the new server, authorized it, and ran the import command:
   `Import-DhcpServer -File C:\dhcp.xml -BackupPath C:\dhcpbackup\ -Leases -Force`.
   I then disabled the scopes on the old server and activated them on the new server.
- **Result**: The scopes and lease databases were migrated successfully, preserving client leases and preventing conflicts.

### HR / Behavioral
**Q: Tell me about a time you resolved a major network outage. How did you identify the DHCP server as the source of the issue?**
A: Users could not log into their computers after a power failure. I checked the DHCP service status, found the database was corrupt, restored the `dhcp.mdb` file from the backup folder, restarted the service, and restored connection.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **What**: The Windows Server role that manages dynamic IP address allocation and configuration options.
> **Why**: Critical for automating network configurations and preventing duplicate IP errors.
> **How**: Configure scopes, set exclusions and reservations, and configure DHCP Failover for high availability.
> **Command**: `Get-DhcpServerv4Scope`
> **Interview Answer Starter**: "In my experience, Windows DHCP management requires configuring scopes with proper exclusions and setting up Active-Passive failover for redundancy..."

**Key Numbers to Remember:**
- Default lease duration: 8 days
- Database path: `%systemroot%\System32\dhcp\dhcp.mdb`
- DHCP Failover ports: TCP 647 (Failover communication)

**3 Things Interviewer Wants to Hear:**
1. Restoring a corrupt DHCP database from the Jet backup directory
2. How DHCP Failover manages high availability
3. Authorizing DHCP servers in Active Directory to prevent rogue servers

---

## Related Notes
- [[01-Foundations/02-Networking/DHCP|DHCP]] — Details client-side DHCP protocols.
- [[03-Identity-and-Core-Services/05-Windows-Server/Roles|Roles]] — Details server role deployment settings.
- [[03-Identity-and-Core-Services/06-Active-Directory/Users-and-Groups|Users and Groups]] — Details service account integrations.

---

## Tags
#desktop-support #windows-server #dhcp #L2 #interview-topic #lab-complete #daily-use

