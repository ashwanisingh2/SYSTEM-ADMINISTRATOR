---
tags: [desktop-support, networking, subnetting, L2]
aliases: [ip-subnetting, cidr-notation]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #ccna
---

# Subnetting

---

## Concept Overview
- **What it is**: Subnetting is the practice of dividing a single large physical network into multiple smaller, logical sub-networks (subnets) by borrowing host bits from the IP address to use as network bits.
- **Why it matters for a support engineer**: A support engineer must understand subnetting to diagnose IP address conflicts, identify routing issues, and configure network adapters with correct masks.
- **Where you encounter this in real job**: Designing IP ranges for new departments, troubleshooting connection drops caused by incorrect subnet masks, configuring router interfaces, and setting up DHCP scopes.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Verifies client IP address parameters, checks gateway connectivity, and checks static configurations for IP/mask errors.
  - **L2**: Configures subnets on switches and routers, assigns IP scopes to departments, and troubleshoots subnet boundary issues.
  - **L3**: Designs the organization's global IP addressing schema, implements Variable Length Subnet Masking (VLSM) policies, and manages global routing tables.

---

## Technical Deep Dive

### 1. IP Address Components & Subnet Masks
An IPv4 address consists of 32 bits, divided into four 8-bit octets. It has two parts:
- **Network ID**: Identifies the network segment.
- **Host ID**: Identifies the specific device on that segment.

The **Subnet Mask** determines where the network portion ends and the host portion begins. It uses binary `1`s for the network portion and `0`s for the host portion.

### 2. CIDR Notation & Common Subnets Reference
Classless Inter-Domain Routing (CIDR) represents the subnet mask by listing the count of active network bits (binary 1s) after a slash (`/`):

| CIDR Prefix | Subnet Mask | Borrowed Bits | Number of Subnets | Total Hosts | Usable Hosts ($2^H - 2$) |
|---|---|---|---|---|---|
| **`/24`** | `255.255.255.0` | 0 | 1 | 256 | **254** |
| **`/25`** | `255.255.255.128`| 1 | 2 | 128 | **126** |
| **`/26`** | `255.255.255.192`| 2 | 4 | 64 | **62** |
| **`/27`** | `255.255.255.224`| 3 | 8 | 32 | **30** |
| **`/28`** | `255.255.255.240`| 4 | 16 | 16 | **14** |
| **`/30`** | `255.255.255.252`| 6 | 64 | 4 | **2** (Used for point-to-point links) |

*Note: We subtract 2 from total hosts because the first address represents the Network ID and the last address represents the Broadcast ID.*

### 3. Step-by-Step Subnet Calculation Math
**Example Problem**: Subnet the network `192.168.10.0` using a `/26` prefix.
- **Step 1: Convert CIDR to Subnet Mask**:
  - `/26` means 26 active bits: `11111111.11111111.11111111.11000000` -> `255.255.255.192`.
- **Step 2: Calculate Subnet Block Size**:
  - Subtract the last non-zero octet from 256: $256 - 192 = 64$.
  - The subnets increment by 64.
- **Step 3: Define Subnet Ranges**:
  - **Subnet 1**:
    - Network ID: `192.168.10.0`
    - First Usable IP: `192.168.10.1`
    - Last Usable IP: `192.168.10.62`
    - Broadcast IP: `192.168.10.63`
  - **Subnet 2**:
    - Network ID: `192.168.10.64`
    - First Usable IP: `192.168.10.65`
    - Last Usable IP: `192.168.10.126`
    - Broadcast IP: `192.168.10.127`

---

## Commands & Syntax

### PowerShell
```powershell
# Retrieve subnet mask and prefix details of active network connections
Get-NetIPAddress -AddressFamily IPv4 |
    Select-Object IPAddress, InterfaceAlias, PrefixLength
```

### CMD / Run Box
```cmd
REM Display current IP details and subnet mask configurations
ipconfig
REM Display current local IP routing table
route print
```

### GUI Path
> Start -> Settings -> Network & Internet -> Advanced Network Settings -> More network adapter options -> Right-click adapter -> Properties -> Double-click **Internet Protocol Version 4 (TCP/IPv4)**.

### Important Registry Paths
```
HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\Interfaces\{GUID}\SubnetMask
HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\Interfaces\{GUID}\IPAddress
```

### Key Event IDs
| Event ID | Meaning | Where to Check |
|----------|---------|----------------|
| 4199 | Duplicate IP address detected on the network | System Log |
| 1001 | DHCP client failed to obtain an IP lease | System Log |

---

## Real-World Scenarios

### Scenario 1: Workstation Cannot Communicate with Devices on Same VLAN
**User Complaint:** "I cannot print to the office printer or access the local file share. My IP address is `192.168.10.75`, and the printer is at `192.168.10.90`. We are in the same room."
**Your First 3 Checks:**
1. Check the IP configuration and subnet mask on both the workstation and the printer.
2. Verify the routing connection to the default gateway.
3. Check the subnet range calculations.
**Diagnosis Steps:**
1. Run `ipconfig` on the client:
   - IP: `192.168.10.75` -> Subnet Mask: `255.255.255.192` (CIDR `/26`).
2. Run calculations for a `/26` mask on the `192.168.10.0` network:
   - Subnet 1 range: `192.168.10.0` to `192.168.10.63`.
   - Subnet 2 range: `192.168.10.64` to `192.168.10.127`.
   - The workstation (`192.168.10.75`) resides in Subnet 2.
3. Check the printer IP (`192.168.10.90`) and its mask. The printer has mask `255.255.255.0` (/24).
**Root Cause:** The client has a `/26` subnet mask, while the printer has a `/24` subnet mask. The client flags `192.168.10.90` as belonging to a different subnet, attempting to route traffic through the default gateway, which blocks or misroutes the packets.
**Fix:** Reconfigure the client's network adapter with the correct subnet mask matching the subnet configuration (`255.255.255.0`).
**Prevention:** Avoid static IP configurations on client machines; enforce DHCP to distribute correct subnet settings.
**Ticket Close Note:** "Corrected the client's subnet mask to match the subnet configuration (changed from 255.255.255.192 to 255.255.255.0). Verified print access. Closing ticket."

### Scenario 2: Duplicate IP Conflict Warning on Production Server
**User Complaint:** "The server console is showing a warning stating duplicate IP address detected on the network, and the server is losing connection intermittently."
**Your First 3 Checks:**
1. Check the server's MAC address and IP configuration.
2. Search the network ARP table to find the MAC address of the conflicting device.
3. Check the active leases on the DHCP server.
**Diagnosis Steps:**
1. Open the command prompt on the server and check its MAC address:
   `getmac` -> Output: `00-11-22-AA-BB-CC`.
2. Locate the conflicting MAC address from the router ARP table:
   `arp -a` -> Output: `192.168.10.10` maps to `00-11-22-DD-EE-FF` (indicating another device is active at that IP).
3. Search for the MAC address `00-11-22-DD-EE-FF` in the DHCP console or MAC vendor database. It matches a user's personal smartphone connected to the corporate Wi-Fi.
**Root Cause:** A user statically configured their personal device with IP `192.168.10.10` (the server's IP), creating a duplicate IP conflict.
**Fix:** Block the conflicting device MAC address on the wireless controller, and reconfigure the server with an IP reservation outside the active DHCP scope.
**Prevention:** Exclude all static server IP ranges from the active DHCP address pools.
**Ticket Close Note:** "Isolated the duplicate IP conflict. Removed the unauthorized static device from the network. Reconfigured the server IP range exclusions. Closing ticket."

---

## Critical Points

> [!danger] Never Do This
> - Assign the Network ID (first address in a subnet range) or the Broadcast ID (last address in a subnet range) to a client workstation or server interface.
> - These IP addresses are reserved for network routing and broadcast functions; assigning them causes routing loops and network connection drops.
> - Always allocate IPs within the usable host range.

> [!warning] Common Trap
> - Assuming that devices on different subnets (e.g., `192.168.10.35/26` and `192.168.10.85/26`) can communicate directly through a switch without a router.
> - Even if they are connected to the same physical switch, different subnets require a Layer 3 routing device to forward packets between them.
> - Configure Inter-VLAN routing (Router-on-a-Stick) to connect different subnets.

> [!tip] Senior Engineer Tip
> - When subnetting, remember the formula for usable hosts: $2^H - 2$, where $H$ is the number of host bits (binary 0s). The 2 represents the Network ID and the Broadcast ID.

> [!success] Verification Steps
> - Run: `Test-NetConnection -ComputerName "192.168.10.10"` to verify routing.
> - Confirm the connection test succeeds.

> [!question] Interview Alert
> - "How many usable host IP addresses are available in a `/27` subnet?"
> - Answer: "A `/27` subnet has 27 network bits, leaving 5 host bits ($32 - 27 = 5$). The total hosts count is $2^5 = 32$. Subtracting the network and broadcast addresses ($32 - 2$, we get **30** usable host IP addresses."

---

## Common Mistakes & Fixes
| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| Incorrect subnet mask configuration | Typing `255.255.255.0` by habit | Check the network design specifications before entering masks. |
| Using the broadcast IP for a gateway | Confusing host ranges | Set the gateway to the first usable host IP (e.g., `.1` or `.254`) in the range. |
| IP scopes overlapping | Overlapping DHCP ranges | Ensure subnet IP ranges do not overlap across DHCP scopes. |

---

## Lab Exercise

**Objective:** Calculate subnet parameters for a given subnet mask, configure a secondary IP address on a network interface, and verify routing boundaries.
**Time Required:** 20 minutes
**Environment Needed:** A Windows 11 VM or physical machine with administrative rights.
**Pre-requisites:** Working network adapter.

**Steps:**
1. Calculate the subnets for the IP network `192.168.5.0` with mask `/26` (`255.255.255.192`):
   - Block size: $256 - 192 = 64$.
   - Subnet 1: `192.168.5.0` -> Usable range: `192.168.5.1` to `192.168.5.62`.
   - Subnet 2: `192.168.5.64` -> Usable range: `192.168.5.65` to `192.168.5.126`.
2. Configure a static IP on the workstation interface: Open network adapter settings -> IPv4 Properties.
3. Configure settings:
   - IP Address: `192.168.5.10`
   - Subnet Mask: `255.255.255.192`
   - Gateway: Leave blank.
4. Try to ping IP `192.168.5.70` (belongs to Subnet 2).
   - Expected output: Ping fails with "Request timed out" or "Destination host unreachable," confirming subnet isolation.
5. Verification: Run the verification command.
   ```powershell
   Get-NetIPAddress -IPAddress 192.168.5.10 | Select-Object IPAddress, PrefixLength
   ```
   - Expected output: `PrefixLength : 26`

**Success Criteria:** The IP is configured with the correct `/26` prefix, and traffic to secondary subnets is blocked without a router.
**Common Failures:** Communication succeeds if the host adapter is configured with a default gateway that routes traffic between the subnets.

---

## Interview Questions & Answers

### Basic (L1 Level)
**Q: What is a default gateway and why is it required?**
A: A default gateway is the router interface that local hosts send traffic to when the destination IP address is outside the local subnet. It is required to route packets between different networks.

**Q: What does CIDR stand for and what does the slash notation mean?**
A: CIDR stands for Classless Inter-Domain Routing. The slash notation (e.g., `/24`) indicates the count of active network bits (binary 1s) in the subnet mask. For example, `/24` represents mask `255.255.255.0`.

### Intermediate (L2 Level)
**Q: Explain Variable Length Subnet Masking (VLSM) and why it is used.**
A: VLSM is the practice of dividing a network space into subnets of varying sizes using different subnet masks. It is used to prevent IP address waste by matching subnet allocations to the actual host requirements of each department (e.g., assigning a `/30` for point-to-point links and a `/24` for a large user segment).

### Advanced (L3/Senior Level)
**Q: A branch office needs to accommodate 4 departments: Sales (100 hosts), HR (14 hosts), IT (25 hosts), and Finance (50 hosts). You are allocated network `172.16.10.0/24`. Design the VLSM subnet allocation.**
A:
- **Situation**: A branch office required subnet allocations for 4 departments using a single `/24` network pool.
- **Task**: I needed to allocate subnet ranges using VLSM to optimize address space.
- **Action**: I sorted the subnets by size, from largest to smallest, to allocate ranges:
  1. **Sales** (100 hosts): Requires a `/25` subnet (126 usable hosts).
     - Range: `172.16.10.0/25` (IPs: `.1` to `.126`).
  2. **Finance** (50 hosts): Requires a `/26` subnet (62 usable hosts).
     - Range: `172.16.10.128/26` (IPs: `.129` to `.190`).
  3. **IT** (25 hosts): Requires a `/27` subnet (30 usable hosts).
     - Range: `172.16.10.192/27` (IPs: `.193` to `.222`).
  4. **HR** (14 hosts): Requires a `/28` subnet (14 usable hosts).
     - Range: `172.16.10.224/28` (IPs: `.225` to `.238`).
- **Result**: All departments were allocated subnets with room for growth, leaving IP range `172.16.10.240/28` unallocated for future subnets.

### HR / Behavioral
**Q: Tell me about a time you had to design or reconfigure a network addressing scheme for a department migration.**
A: I designed a new IP addressing scheme for a relocated development group. I calculated their host requirements, configured a `/26` VLAN subnet pool, updated the DHCP scope options, and verified routing paths, completing the migration with zero downtime.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **What**: The practice of dividing a physical network into smaller logical subnets to optimize routing.
> **Why**: Conserves IP space, improves security boundaries, and minimizes broadcast traffic.
> **How**: Borrow host bits to create network subnets, and configure adapters with corresponding masks.
> **Command**: `Get-NetIPAddress`
> **Interview Answer Starter**: "In my experience, subnet calculations start by identifying the host count requirements and matching them to the closest power of 2..."

**Key Numbers to Remember:**
- Usable hosts formula: $2^H - 2$
- Point-to-point subnet prefix: `/30` (2 usable hosts)
- Standard Class C network prefix: `/24` (254 usable hosts)

**3 Things Interviewer Wants to Hear:**
1. How to calculate usable host counts ($2^H - 2$)
2. Understanding of VLAN subnet boundaries and routing requirements
3. Experience with VLSM allocations

---

## Related Notes
- [[01-Foundations/02-Networking/OSI-Model|OSI Model]] — Details Network layer routing context.
- [[01-Foundations/02-Networking/NAT-and-PAT|NAT and PAT]] — Details translation boundaries.
- [[01-Foundations/02-Networking/Troubleshooting-Tools|Troubleshooting Tools]] — Diagnostic tools for subnet routing checks.

---

## Tags
#desktop-support #networking #subnetting #L2 #interview-topic #lab-complete #daily-use
