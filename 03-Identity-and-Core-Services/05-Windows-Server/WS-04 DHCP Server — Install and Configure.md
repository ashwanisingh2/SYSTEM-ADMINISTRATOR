---
tags: [sysadmin, windows-server, dhcp, ip-management]
difficulty: Intermediate
lab-required: Yes
read-time: 15 mins
---

# WS-04: DHCP Server — Install and Configure

> [!abstract] Overview
> This note covers Windows Server DHCP deployment, scope management, reservations, failover clustering, Active Directory authorization, and auditing logs.

---
## Concept
Think of a Windows Server DHCP server as the automated front-desk keycard programmer at a massive hotel. 

Instead of having a security officer (sysadmin) manually write down and assign a specific room number (static IP) to every incoming guest (device), the front desk manages a dynamic key pool (Scope). When a guest walks in (boots up), they ask for a key (DORA process). The programmer issues a temporary keycard (lease) that works for a set time (e.g., 8 days). 

For VIP guests (like network printers and servers), the programmer reserves a permanent key tied directly to their passport ID (MAC Address Reservation) so they always get the same room every time they visit.

*Seedha simple mein: DHCP Server network par sabhi device ko dynamically IP address distribute karta hai. AD environment mein unauthorized DHCP servers ko prevent karne ke liye authorization zaroori hoti hai. High availability ke liye hum DHCP Failover set karte hain.*

---
## Technical Deep Dive

### 1. Active Directory Authorization
In an Active Directory environment, a standalone DHCP server can cause conflicts by distributing incorrect IP addresses or gateways (Rogue DHCP). 
- **Authorization Requirement:** Before a Windows DHCP server can lease IP addresses, it must be authorized in Active Directory.
- **How it works:** When the DHCP service starts, it queries the Domain Controller. If it is not listed in the AD authorized servers list, the service automatically disables itself and logs an error in the Event Viewer. Only Enterprise Admins can authorize a DHCP server.

### 2. Scope Configuration & Key Options
A DHCP Scope is the range of IP addresses on a subnet that the DHCP server can distribute.
- **Scope Parameters:**
  - **Start & End IP:** The lease boundaries (e.g., `192.168.10.100` to `192.168.10.254`).
  - **Exclusion Range:** Addresses *within* the scope that are blocked from dynamic distribution (e.g., exclude `192.168.10.100` to `192.168.10.120` because they are static printer/router IPs).
  - **Lease Duration:** Time a client holds the IP before needing to renew. Default is 8 days for wired clients; set shorter (e.g., 8 hours) for guest Wi-Fi networks with high user turnover.
- **Standard DHCP Options:**
  - **Option 003 Router:** The Default Gateway IP.
  - **Option 006 DNS Servers:** Target IPs for name resolution.
  - **Option 015 Domain Name:** The domain suffix (e.g., `company.local`).
  - **Option 060 PXE Client:** Used for network boot installation (e.g., WDS).

### 3. Server Options vs. Scope Options Hierarchy
- **Server Options:** Configured globally at the server root. Applies to *all* scopes on that DHCP server.
- **Scope Options:** Configured inside a specific scope. Applies only to clients in that subnet.
- **Reservation Options:** Configured directly on a single client reservation.
- **Hiearchy rule:** **Specific overrides general**. Reservation Options override Scope Options, and Scope Options override Server Options.

### 4. DHCP Reservations
A reservation binds a specific MAC address to a specific IP address.
- **Mechanism:** When the client sends a `DHCPDISCOVER`, the server reads the client's MAC address, matches it against its reservation database, and leases the assigned IP instead of a dynamic one.
- **Use Case:** Crucial for network printers, storage arrays (NAS), and management interfaces that need static addressing but benefit from centralized DHCP management (e.g., pushing DNS changes centrally).

### 5. DHCP Failover Modes
Windows Server 2012 and newer features native DHCP Failover, securing high availability for scopes between two servers without complex clustering.
- **Active-Passive (Hot Standby) Mode:** One server is the primary active lease provider. The secondary server remains silent, replicating the lease database. If the primary goes offline, the standby takes over after a configured interval. Best for multi-site hubs.
- **Active-Active (Load Balance) Mode:** Both servers distribute IPs from the same scope simultaneously based on a load-sharing percentage (e.g., $50/50$). Best for single-site datacenters.

---
## Lab — Step by Step
> [!info] Lab Setup Needed
> Two Windows Server 2022 VMs (`SVR-DC01` and `SVR-DHCP02`) on the same subnet, both authorized in AD.

### Step 1: Create a Scope and Exclusion
1. On `SVR-DC01`, open Tools -> **DHCP**.
2. Right-click **IPv4** -> **New Scope**.
3. Name: `Corporate_LAN`. Range: `192.168.10.1` to `192.168.10.254`. Mask: `/24`.
4. Add Exclusions: Add `192.168.10.1` to `192.168.10.50` (reserved for servers and gateways).
5. Set lease duration to `8 days`.
6. Configure Options:
   - Router (003): `192.168.10.1`
   - DNS (006): `192.168.10.10`
7. Click Finish. Right-click the server name and click **Authorize** (if not already completed).

### Step 2: Create a Client Reservation
1. In the DHCP console, expand **Corporate_LAN** -> **Reservations**.
2. Right-click **Reservations** and select **New Reservation**.
3. Reservation Name: `PRINTER-ACCOUNTS`.
4. IP Address: `192.168.10.60`.
5. MAC Address: Enter the 12-digit MAC of the printer NIC (e.g., `001122aabbcc`).
6. Supported Types: **Both** (DHCP and BOOTP). Click **Add**.

### Step 3: Configure DHCP Failover (Hot Standby)
1. Install the DHCP role on `SVR-DHCP02` and authorize it.
2. Go back to `SVR-DC01`. In DHCP Manager, right-click the scope `Corporate_LAN` and select **Configure Failover**.
3. Select the target scope. Click Next.
4. Partner Server: Enter `SVR-DHCP02.company.local`. Click Next.
5. Failover Configuration:
   - Relationship Name: `DC01-DHCP02-Failover`
   - Mode: **Hot Standby**
   - Role of Partner Server: **Standby**
   - Addresses reserved for standby: $5\%$ (reserved for emergency leases during transition).
   - State Switchover Interval: Check and set to `60 minutes` (automatically declares partner dead after 1 hour).
   - Shared Secret: Enter a replication password (e.g., `SyncSecret123`).
6. Click Next, then **Finish**. Verify that synchronization completes successfully.

---
## Diagnostic and Troubleshooting Commands
Advanced content only — basics in [[Basic networking commands]]

```cmd
:: Windows Command Prompt
:: Release current IP lease on client adapter
ipconfig /release

:: Request a new IP lease from the active DHCP server
ipconfig /renew

:: View current DHCP lease source server details
ipconfig /all | findstr /i "DHCP Server"
```

### DHCP Audit Logs Analysis
DHCP servers maintain detailed audit logs in `%systemroot%\System32\dhcp\`.
- Filename format: `DhcpSrvLog-xxx.log` (split by day of the week, e.g., `DhcpSrvLog-Mon.log`).
- **Key Event Codes to parse:**
  - `10` — A new IP lease was dynamically issued to a client.
  - `11` — A lease renewal request was acknowledged.
  - `13` — IP address conflict detected on the network (very critical).
  - `20` — An unauthorized DHCP server was detected.

---
## Troubleshooting Scenarios

**Scenario 1:**
- **Problem:** The DHCP Server service starts up, but immediately shuts down, throwing error in Event Viewer: "The DHCP service on this computer has shut down because it is not authorized in Active Directory."
- **Root Cause:** A non-authorized DC or member server running the DHCP role.
- **Fix:**
  1. Log into a Domain Controller using Enterprise Administrator credentials.
  2. Open the **DHCP** management console.
  3. Right-click the root server name that is failing. Select **Authorize**.
  4. Alternatively, use PowerShell:
     ```powershell
     Add-DhcpServerInDC -DnsName SVR-DHCP02.company.local -IPAddress 192.168.10.11
     ```
  5. Start the DHCP Server service; it will now remain running.

**Scenario 2:**
- **Problem:** Workstation users complain they are receiving dynamic IP addresses, but they have no Internet access. Running `ipconfig /all` reveals their gateway (Option 003) is listed as `192.168.10.254` instead of the correct gateway `192.168.10.1`.
- **Root Cause:** A rogue DHCP server is active on the network segment (e.g., a home router plugged in backward, or a misconfigured virtualization host).
- **Fix:**
  1. Open a command prompt on an affected client. Run `ipconfig /all` and write down the IP of the DHCP Server leasing the bad configuration.
  2. Open the active DHCP audit logs on the real DC. Look for event code `20` alerts listing rogue IPs.
  3. Locate the physical port of the rogue IP using the switch MAC table (`show mac address-table` on the core switch).
  4. Shut down the switch port to disconnect the rogue server. Enable **DHCP Snooping** on the switch to prevent future occurrences.

---
## Common Mistakes
> [!warning] Avoid These
> **Configuring dynamic reservations inside the active pool without exclusions:** Creating a reservation for an IP address that has already been dynamically leased to another active client. This instantly triggers an IP conflict, disconnecting both devices.
> **Correct approach:** Place reservations on IP addresses that reside outside the dynamic pool range or within the configured exclusion block of the scope.

---
## Pro Tips
> [!tip] Field Experience
> When configuring DHCP failover, always check the system clocks on both servers. DHCP failover uses timestamped replication packages; if the time drift between the two servers exceeds 60 seconds, synchronization will fail and drop the failover relationship status into a degraded state.

---
## Quick Revision Table
| # | Concept | One Line Summary |
|---|---------|-----------------|
| 1 | AD Authorization | Security check preventing unauthorized rogue DHCP services from launching in a domain. |
| 2 | DHCP Exclusions | Subnet blocks within a scope blocked from dynamic distribution to reserve for static hosts. |
| 3 | DHCP Reservations| Binding a client MAC address to a specific IP address to guarantee a static lease. |
| 4 | Hot Standby | Active-Passive failover mode where the partner server takes over only if the primary goes down. |
| 5 | Event Code 13 | DHCP log event indicating that an IP address conflict has occurred on the network. |

---
## Interview Q&A

**Q1: Explain the difference between Scope Options and Server Options in a Windows DHCP server. Which takes priority?**
A: **Server Options** are global configurations applied at the root level of the DHCP server. They apply to all scopes hosted on that server (e.g., setting global DNS servers). **Scope Options** are subnet-specific configurations applied only to clients within that specific scope (e.g., setting the specific default gateway for that VLAN). Scope Options take priority. If an option (like DNS Server Option 006) is configured on both the Server level and the Scope level, the client will receive the Scope-level setting because it is more specific.

**Q2: A client's DHCP scope is $98\%$ full, and new devices are failing to connect. You cannot expand the physical subnet. How do you resolve this?**
A: 
- **Situation:** The DHCP pool is exhausted, and subnet resizing is not possible.
- **Task:** Reclaim inactive leases and optimize scope efficiency.
- **Action:** First, I will look at the Lease Duration. If it is set to the default 8 days, but the subnet has high device turnover (like a guest Wi-Fi network), I will lower the lease duration to 4 or 8 hours. This allows the server to reclaim expired leases much faster. Second, I will check for stale inactive leases in the lease pool and delete them manually. Third, I can configure a **Superscope** to group multiple logical subnets together if another IP range can be routed on the same physical link.
- **Result:** Lowering the lease time dynamically frees up IPs, resolving the allocation failures.

**Q3: What is the DHCP client transition state process when an IP lease reaches $50\%$ and $87.5\%$ of its duration?**
A: 
1. **Renewal State (T1 at $50\%$ lease time):** The client attempts to renew its lease. It sends a unicast `DHCPREQUEST` directly to the DHCP server that originally issued the IP. If the server acknowledges (`DHCPACK`), the lease timer resets.
2. **Rebinding State (T2 at $87.5\%$ lease time):** If the original server does not respond (e.g., it is offline), the client waits until $87.5\%$ of the lease has elapsed. It then enters the rebinding state and sends a broadcast `DHCPREQUEST` to *any* available DHCP server on the network. If a server responds, the lease is renewed. If the lease expires completely ($100\%$), the client drops the IP and defaults to APIPA.

---
## Related Notes
- [[01-Foundations/02-Networking/N-08 IP Services — DHCP DNS NAT|N-08 IP Services — DHCP DNS NAT]] — The core network services protocol flows.
- [[03-Identity-and-Core-Services/06-Active-Directory/WS-02 Active Directory Domain Services|WS-02 Active Directory Domain Services]] — AD DS authorization processes.
- [[03-Identity-and-Core-Services/05-Windows-Server/WS-03 DNS Server — Install and Configure|WS-03 DNS Server — Install and Configure]] — Mapping dynamic DNS records.

