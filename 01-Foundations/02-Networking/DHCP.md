---
tags: [desktop-support, networking, dhcp, L1]
aliases: [dynamic-host-configuration, ip-assignment]
created: 2026-06-25
status: #complete
difficulty: #beginner
cert-relevant: #ccna
---

# DHCP

---

## Concept Overview
- **What it is**: The Dynamic Host Configuration Protocol (DHCP) is a network management protocol operating at the Application layer (OSI Layer 7) that automatically assigns IP addresses, subnet masks, default gateways, and DNS settings to network devices.
- **Why it matters for a support engineer**: Manual IP configuration on client workstations is inefficient and leads to IP conflict errors. DHCP automates IP configuration, making it critical for client connectivity and mobility.
- **Where you encounter this in real job**: Diagnosing self-assigned APIPA IP addresses (`169.254.x.x`), configuring IP reservations for network printers, managing DHCP scopes on routers or servers, and setting up PXE boot options.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Troubleshoots client-side IP lease errors, executes release/renew commands, and configures static IP reservations.
  - **L2**: Manages server DHCP scopes, configures exclusion ranges, and sets up scope options.
  - **L3**: Configures DHCP Relay Agents (IP Helper addresses) on routers, designs DHCP failover clusters, and implements security controls like DHCP Snooping.

---

## Technical Deep Dive

### 1. The DHCP DORA Process
When a device boots up with IP settings set to automatic, it runs the 4-step DORA handshake:
- **Discover (Broadcast)**: The client broadcasts a request packet to locate any active DHCP servers on the subnet.
- **Offer (Unicast/Broadcast)**: Available DHCP servers reply with an offered IP address, subnet mask, lease duration, and server IP.
- **Request (Broadcast)**: The client broadcasts a request accepting the offered IP configuration from the specific server, notifying other servers that their offers were not chosen.
- **Acknowledge (Unicast/Broadcast)**: The chosen DHCP server replies with an ACK packet confirming the lease registration and sending additional scope options (DNS, gateway).

```
Client                                     DHCP Server
  | ----------[ DISCOVER (Broadcast) ]------>  |
  | <---------[ OFFER (Unicast) ]-------------  |
  | ----------[ REQUEST (Broadcast) ]------->  |
  | <---------[ ACK (Unicast) ]---------------  |
```

### 2. DHCP Scopes, Exclusions & Reservations
- **DHCP Scope**: A consecutive range of IP addresses that a DHCP server is authorized to distribute to clients on a specific subnet (e.g., `192.168.10.100` to `192.168.10.200`).
- **Exclusion Range**: A sub-range of IPs within the scope that the server is blocked from distributing (e.g., `192.168.10.100` to `192.168.10.110`), reserved for static configurations (servers, printers).
- **IP Reservation**: A configuration that binds a specific IP address to a client's physical MAC address, ensuring that the client always receives the same IP address on lease renewal.

### 3. Core DHCP Options
Options deliver configuration parameters alongside the IP address:
- **Option 003 (Router)**: The IP address of the subnet's default gateway.
- **Option 006 (DNS Servers)**: The IP addresses of local or external DNS servers.
- **Option 015 (Domain Name)**: The connection suffix for the domain (e.g., `lab.local`).
- **Option 066 / 067 (Boot Server Host Name / Bootfile Name)**: Configures PXE network boot configurations.

### 4. DHCP Relay Agent (IP Helper)
DHCP Discover broadcasts cannot cross routers. To centralize DHCP servers, routers are configured with a **DHCP Relay Agent** (or Cisco `ip helper-address`). When the router receives a DHCP discover broadcast on a local interface, it converts it to a unicast packet and forwards it directly to the designated DHCP server on another subnet.

---

## Commands & Syntax

### PowerShell
```powershell
# Retrieve active IP configuration parameters including DHCP lease times
Get-NetIPAddress -InterfaceAlias "Ethernet" |
    Select-Object IPAddress, InterfaceAlias, PrefixLength

# Query active DHCP leases from the DHCP server (runs on DHCP Server)
Get-DhcpServerv4Lease -ScopeId 192.168.10.0
```

### CMD / Run Box
```cmd
REM Release the current DHCP leased IP address
ipconfig /release
REM Request a new IP address lease from the DHCP server
ipconfig /renew
```

### GUI Path
> Start -> Run -> type `dhcpmgmt.msc` (DHCP Server Management Console)
> Or: Network Connections -> Adapter Properties -> IPv4 Properties -> Check **Obtain an IP address automatically**

### Important Registry Paths
```
HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\Interfaces\{GUID}\
HKLM\SYSTEM\CurrentControlSet\Services\Dhcp\
```

### Key Event IDs
| Event ID | Meaning | Where to Check |
|----------|---------|----------------|
| 1001 | DHCP client failed to obtain an IP lease (APIPA fallback) | System Log |
| 1002 | DHCP client lost connection to the DHCP server | System Log |
| 1044 | DHCP server is unauthorized in Active Directory | System Log |

---

## Real-World Scenarios

### Scenario 1: Workstation Obtains an APIPA IP Address (`169.254.100.45`)
**User Complaint:** "I cannot connect to the local intranet or the internet. My network adapter properties show the status as connected, but my IP address starts with 169."
**Your First 3 Checks:**
1. Check the client's current IP configuration using `ipconfig /all`.
2. Verify if the client can reach the gateway router or other devices in the office.
3. Check the DHCP server scope utilization status.
**Diagnosis Steps:**
1. Run `ipconfig` on the client:
   - IP: `169.254.100.45`, indicating an APIPA address.
2. Check if other machines are affected. Other users on the same subnet have working `192.168.10.x` addresses, ruling out network switch issues.
3. Open the DHCP Manager console (`dhcpmgmt.msc`) on the DHCP server:
   - Check scope `192.168.10.0` -> **Address Pool** and **Active Leases**.
   - The scope has `0` available addresses. Address pool utilization is at 100%.
**Root Cause:** The DHCP scope address pool was exhausted, causing new devices to fail the DORA process and fall back to APIPA.
**Fix:** Expand the scope range or reduce the lease time from 8 days to 24 hours to reclaim inactive IP allocations. Run `ipconfig /renew` on the client.
**Prevention:** Configure DHCP scope alerts to notify the team when address pool capacity reaches 85%.
**Ticket Close Note:** "Diagnosed scope exhaustion on DHCP server. Extended the scope IP range. Ran ipconfig /renew on the client, which leased `192.168.10.155`. Closing ticket."

### Scenario 2: Network Printer Receives a Different IP After Power Outage
**User Complaint:** "Our department printer is offline, and we cannot print documents. The printer's control panel shows a new IP address."
**Your First 3 Checks:**
1. Check the current IP address displayed on the printer panel.
2. Verify the configuration settings in the print queue properties.
3. Retrieve the physical MAC address of the printer.
**Diagnosis Steps:**
1. Check the print queue: It is configured to route traffic to static IP `192.168.10.50`.
2. Check the printer panel: The current IP is dynamically leased as `192.168.10.142`.
3. Locate the printer's MAC address (printed on the NIC label or shown in the printer settings).
**Root Cause:** The printer was configured with a dynamic IP instead of an IP reservation. After a power outage, the printer rebooted, requested a new lease, and was assigned a different IP.
**Fix:** Create a static IP reservation on the DHCP server binding `192.168.10.50` to the printer's MAC address. Reboot the printer.
**Prevention:** Always configure static IP reservations for shared network peripherals (printers, scanners).
**Ticket Close Note:** "Created static DHCP reservation for printer MAC `00:11:22:33:44:55` at IP `192.168.10.50`. Rebooted printer. Verified printing functionality. Closing ticket."

---

## Critical Points

> [!danger] Never Do This
> - Connect unauthorized consumer routers (such as a home Wi-Fi router) to corporate wall jacks.
> - These routers run active DHCP servers by default, acting as **Rogue DHCP Servers** that distribute incorrect IP configurations to local clients, causing network-wide connection drops.
> - Configure **DHCP Snooping** on corporate switches to block unauthorized DHCP offer messages.

> [!warning] Common Trap
> - Assuming that running `ipconfig /renew` will resolve connection issues when the network cable (Layer 1) or switch port configuration (Layer 2) is faulty.
> - If the client cannot communicate with the DHCP server, `ipconfig /renew` will hang and eventually time out, returning an APIPA address.
> - Verify physical link and VLAN configurations before running renewals.

> [!tip] Senior Engineer Tip
> - Set DHCP lease times based on the network environment: Set long lease times (e.g., 8 days) for static office desktops. Set short lease times (e.g., 2 to 4 hours) for guest Wi-Fi networks with high user turnover to prevent scope exhaustion.

> [!success] Verification Steps
> - Run: `ipconfig /all` on the client.
> - Verify the **DHCP Enabled** line displays **Yes**, and the lease start/expiry timestamps are correct.

> [!question] Interview Alert
> - "What is an APIPA IP address, and what does it tell you about a system's connection status?"
> - Answer: "APIPA (Automatic Private IP Addressing) is a feature in Windows that assigns an IP in the `169.254.0.0/16` range when a system set to obtain an IP automatically cannot locate a DHCP server. It tells me that the client has a working local network adapter, but cannot communicate with the DHCP server."

---

## Common Mistakes & Fixes
| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| Overlapping scopes across DHCP servers | Poor planning | Ensure scopes do not overlap to prevent IP address conflict errors. |
| Forgetting gateway IP in options | Configuring scope without Option 003 | Always define Option 003 (Router) to allow clients to route traffic outside their subnet. |
| Leaving unauthorized DCs unauthorized | Installing DHCP on DC without AD authorization | Authorize the DHCP server in Active Directory (Event ID 1044). |

---

## Lab Exercise

**Objective:** Configure a DHCP scope, set up an exclusion range, create a static IP reservation, and verify IP delivery on a client machine.
**Time Required:** 30 minutes
**Environment Needed:** A Windows Server 2022 VM (DHCP Server) and a Windows 11 VM.
**Pre-requisites:** DHCP Server role installed on the server VM, both VMs connected to VMnet1 (Host-only).

**Steps:**
1. Open DHCP Manager: Open Server Manager -> `Tools` -> **DHCP**.
2. Create Scope: Right-click IPv4 -> Select **New Scope**.
   - Name: `Client Scope`
   - IP Address Range: `192.168.10.100` to `192.168.10.200`
   - Subnet Mask: `255.255.255.0`
3. Add Exclusions: Enter `192.168.10.100` to `192.168.10.110` -> Click **Add** -> Click **Next**.
4. Configure Options: Set Option 003 (Router) to `192.168.10.1`. Set Option 006 (DNS) to `192.168.10.10`.
5. Create Reservation: Expand the new scope -> Right-click **Reservations** -> Select **New Reservation**.
   - Name: `Test Client`
   - IP Address: `192.168.10.150`
   - MAC Address: Enter the MAC address of the Windows 11 VM -> Click **Add**.
6. Test Client: Log into your Windows 11 VM. Open a command prompt and run:
   ```cmd
   ipconfig /release
   ipconfig /renew
   ```
7. Verification: Confirm the leased IP is exactly `192.168.10.150`.
   ```powershell
   Get-NetIPConfiguration
   ```

**Success Criteria:** The client successfully leases IP `192.168.10.150`, and receives the configured gateway and DNS server settings.
**Common Failures:** Lease failure if the client's MAC address is input incorrectly in the reservation details.

---

## Interview Questions & Answers

### Basic (L1 Level)
**Q: Explain the DORA process in DHCP.**
A: The DORA process is the 4-step handshake used to assign IP addresses: **Discover** (client broadcasts to find a server), **Offer** (server offers an IP address configuration), **Request** (client requests to lease the offered IP), and **Acknowledge** (server confirms the lease registration and sends scope options).

**Q: What is the default port used by DHCP?**
A: DHCP operates on UDP transport ports. Clients send query packets on source port **68** to target destination port **67** on the DHCP server.

### Intermediate (L2 Level)
**Q: What is the purpose of a DHCP Relay Agent, and when is it required?**
A: A DHCP Relay Agent (or IP Helper) is a router configuration that forwards local DHCP broadcast packets across subnets as unicast packets to a centralized DHCP server. It is required when the DHCP server is located on a different subnet than the client devices, as standard routers block broadcast traffic.

### Advanced (L3/Senior Level)
**Q: You are migrating a department subnet to a new VLAN. The network is configured with a centralized DHCP server. Explain the configuration steps required to ensure clients on the new VLAN receive IP addresses.**
A:
- **Situation**: A new department VLAN was created, requiring dynamic IP distribution from a centralized DHCP server.
- **Task**: I needed to enable DHCP routing across the subnets.
- **Action**: I logged into the core network switch and configured an `ip helper-address` pointing to our DHCP server IP (`192.168.10.10`) on the new VLAN interface. Next, I opened the DHCP console, created a new scope matching the new subnet IP range (e.g., `192.168.20.0/24`), configured the scope options (Router Option 003 set to `192.168.20.1`), and activated the scope.
- **Result**: The core switch successfully relayed DHCP Discover broadcasts from the new VLAN to the DHCP server, which matched the incoming interface IP to the correct scope and distributed IP addresses.

### HR / Behavioral
**Q: Tell me about a time you resolved an IP conflict on a critical production server. How did you resolve the issue?**
A: A production server went offline with an IP address conflict warning. I logged into the router, ran an ARP table search to locate the MAC address of the conflicting device, and traced it to an unmanaged testing workstation that was statically configured with the server's IP. I reconfigured the workstation to use DHCP, resolving the conflict.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **What**: The dynamic IP configuration protocol that automates IP address distribution.
> **Why**: Prevents manual configuration errors and simplifies client network onboarding.
> **How**: Configure scopes, exclusions, reservations, and options, and manage client lease renewals.
> **Command**: `ipconfig /renew`
> **Interview Answer Starter**: "In my experience, DHCP troubleshooting starts by verifying client connectivity and running ipconfig to check for APIPA fallback states..."

**Key Numbers to Remember:**
- APIPA IP range: 169.254.0.0/16
- DHCP client port: 68 (UDP) / server port: 67 (UDP)
- DHCP scope default lease duration: 8 days (Windows Server)

**3 Things Interviewer Wants to Hear:**
1. Complete DORA handshake sequence (Discover -> Offer -> Request -> Acknowledge)
2. Difference between Exclusions (manual static blocks) and Reservations (MAC-bound IPs)
3. Role of IP Helper / DHCP Relay Agent in routing configurations

---

## Related Notes
- [[01-Foundations/02-Networking/OSI-Model|OSI Model]] — Details Application layer context.
- [[01-Foundations/02-Networking/DNS|DNS]] — Distributed via Option 006.
- [[03-Identity-and-Core-Services/05-Windows-Server/DHCP-Server|DHCP Server]] — Deep dive into Windows Server DHCP setups.

---

## Tags
#desktop-support #networking #dhcp #L1 #interview-topic #lab-complete #daily-use

