---
tags: [sysadmin, networking, services, dhcp, dns, nat]
difficulty: Intermediate
lab-required: Yes
read-time: 15 mins
---

# N-08: IP Services — DHCP, DNS, NAT

> [!abstract] Overview
> This note covers key Layer 7 network helper services: DHCP address provisioning, DNS name resolution, and NAT translation architectures. It details DHCP DORA flows, standard DNS record types, and Port Address Translation (PAT) mechanisms.

---
## Concept
Think of these IP services as the infrastructure utilities of a hotel:
- **DHCP** is the reception desk. When you check in (boot up), the clerk assigns you a room number (IP address), gives you a map showing where the dining hall is (Default Gateway), and lists the hotel information desk number (DNS server).
- **DNS** is the phone book directory. If you want to call the room named "Penthouse Suit," you look it up in the book to find their actual telephone extension (IP address).
- **NAT / PAT** is the hotel main switchboard telephone operator. All guest rooms share a single external phone number. When you make an outbound call, the switchboard records your room number and assigns you a unique outbound port. The outside world only sees the hotel main number, and when they reply, the operator routes the call back to your room.

*Seedha simple mein: DHCP automatically systems ko IP address deta hai. DNS human-friendly website names ko IP addresses mein convert karta hai. NAT/PAT private networks ko single public IP share karke internet access karne ki permission deta hai.*

---
## Technical Deep Dive

### 1. DHCP Operation: The DORA Process
DHCP (Dynamic Host Configuration Protocol) operates at the application layer over UDP ports 67 (server) and 68 (client).

```
Client                                                  Server
  |                                                       |
  | ----------- DHCPDISCOVER (L2 Broadcast) ------------> | "Is there a DHCP server?"
  |                                                       |
  | <---------- DHCPOFFER (L2 Unicast/Broadcast) -------- | "I have room 192.168.1.50 for you."
  |                                                       |
  | ----------- DHCPREQUEST (L2 Broadcast) -------------> | "I accept that offer."
  |                                                       |
  | <---------- DHCPACK (L2 Unicast/Broadcast) ---------- | "Confirmed. Here are your options."
  |                                                       |
```
- **DHCP Options:**
  - **Option 3:** Default Gateway (Router IP).
  - **Option 6:** Domain Name System (DNS) Servers.
  - **Option 15:** Connection-specific Domain Name Suffix (e.g., `company.local`).
- **Scope Parameters:**
  - **Scope:** The range of IP addresses available to lease (e.g., `192.168.1.100` to `.200`).
  - **Exclusion:** Specific IPs inside the scope reserved for manual assignment (e.g., `.100` to `.105` for printers/servers) that DHCP will not assign.
  - **Reservation:** Binding a specific MAC address to a specific IP so that the client always receives the same address.

### 2. DNS Resolution Process
DNS (Domain Name System) translates hostnames to IP addresses over UDP/TCP port 53.

```
Client ---> Local DNS Server ---> Root Server (.) ---> TLD Server (.com) ---> Authoritative Server (google.com)
  ^                                                                                 |
  +------------------------------- Returns IP 142.250.190.46 <---------------------+
```
1. **Host Cache Check:** Client checks local hosts file and local cache.
2. **Recursive Query:** Client asks its Local DNS Server (recursive resolver) for `www.google.com`.
3. **Iterative Queries:**
   - Local DNS queries the **Root Name Server (`.`)**, which redirects to the `.com` Top-Level Domain (TLD) server.
   - Local DNS queries the **TLD Server**, which redirects to the **Authoritative Name Server** for `google.com`.
4. **Authoritative Response:** The authoritative server returns the IP address.
5. **Caching:** Local DNS caches the result based on the **TTL (Time to Live)** value and returns it to the client.

### 3. DNS Record Types
- **A Record:** Maps a hostname to an IPv4 address.
- **AAAA Record:** Maps a hostname to an IPv6 address.
- **CNAME (Canonical Name):** An alias record pointing one domain to another (e.g., `web.com` to `www.web.com`).
- **MX (Mail Exchanger):** Specifies mail servers responsible for receiving email for the domain.
- **PTR (Pointer):** Resolves an IP address to a hostname (Reverse DNS).
- **NS (Name Server):** Identifies the authoritative name servers for the zone.
- **SOA (Start of Authority):** Contains administrative data about the zone (refresh timers, contact email, serial number).

### 4. Network Address Translation (NAT) & PAT
NAT translates private RFC 1918 IPs to public routable IPs.
- **Static NAT:** One-to-one mapping. A single private IP maps to a single public IP. Used for hosting internal web/mail servers.
- **Dynamic NAT:** Many-to-many mapping. Private hosts share a pool of public IP addresses on a first-come, first-served basis.
- **PAT (Port Address Translation / NAT Overload):** Many-to-one mapping. Multiple private hosts share a single public IP address by multiplexing TCP/UDP port numbers.

```
NAT Translation Table Example:
+---------------------+-------------------+----------------------+-------------------+
| Inside Local IP     | Inside Local Port | Inside Global IP     | Inside Global Port|
+---------------------+-------------------+----------------------+-------------------+
| 192.168.1.10 (PC1)  | 4125              | 203.0.113.5 (Public) | 10001             |
| 192.168.1.20 (PC2)  | 4125              | 203.0.113.5 (Public) | 10002             |
+---------------------+-------------------+----------------------+-------------------+
```

---
## Lab — Step by Step
> [!info] Lab Setup Needed
> A Windows Server (2019/2022) virtual machine running in Hyper-V or VMware.

### Step 1: Install DHCP Role on Windows Server
1. Open **Server Manager**. Click **Add roles and features**.
2. Select **Role-based or feature-based installation**. Click Next.
3. Select the local server, check **DHCP Server**, click **Add Features**, and complete installation.

### Step 2: Configure a DHCP Scope
1. Open Server Manager -> Tools -> **DHCP**.
2. Expand the server node, right-click **IPv4**, and select **New Scope**.
3. Name: `Client_Scope`. Click Next.
4. Set IP Address Range:
   - Start IP: `192.168.10.100`
   - End IP: `192.168.10.200`
   - Subnet Mask: `255.255.255.0` (/24). Click Next.
5. Add Exclusions: Add `192.168.10.100` to `192.168.10.110` (reserved for local devices). Click Next.
6. Lease Duration: Leave at default (8 days). Click Next.

### Step 3: Configure Scope Options
1. Select **Yes, I want to configure these options now**. Click Next.
2. **Router (Option 003):** Enter `192.168.10.1` (the default gateway). Click Add. Click Next.
3. **DNS Server (Option 006):** Enter `192.168.10.10` (your DNS server IP). Click Add. Click Next.
4. **WINS Server (Option 044):** Leave blank. Click Next.
5. Select **Yes, I want to activate this scope now**. Click Finish.

### Step 4: Verify Client Lease
1. Boot a Windows client VM connected to the same virtual switch.
2. Open CMD on the client. Run:
   ```cmd
   ipconfig /renew
   ```
3. **Verify:** Run `ipconfig /all`. Confirm the IP address falls between `192.168.10.111` and `192.168.10.200`, and options `003` and `006` are configured correctly.

---
## Commands Reference
Advanced content only — basics in [[Basic networking commands]]

Execute remote resolution and interface diagnostic tests:

```cmd
:: Windows Command Prompt
nslookup www.google.com                            :: Check A-record resolution
nslookup -type=mx google.com                       :: Query MX records for google.com
nslookup -type=ptr 8.8.8.8                         :: Perform reverse DNS lookup
ipconfig /displaydns                               :: View current local DNS cache
ipconfig /flushdns                                 :: Clear local DNS cache resolver entries
```

---
## Troubleshooting Scenarios

**Scenario 1:**
- **Problem:** Client workstations on a remote VLAN are not obtaining IP addresses from the central DHCP server. They default to APIPA addresses (`169.254.x.x`).
- **Root Cause:** DHCP Discover broadcasts are blocked by default on routers. The gateway interface lacks a helper address to forward requests to the DHCP server.
- **Fix:**
  1. Log into the router gateway interface for the client VLAN.
  2. Configure the IP helper address targeting the remote DHCP server:
     ```text
     Router(config)# interface gigabitethernet 0/0.10
     Router(config-if)# ip helper-address 10.0.0.50
     ```
  3. Verify that clients now successfully negotiate DHCP leases across the router.

**Scenario 2:**
- **Problem:** Users complain they cannot open websites, but internal tools and direct IP communication work.
- **Root Cause:** DNS server service is down, or DNS configurations on the clients are invalid.
- **Fix:**
  1. Ping a public IP from a client: `ping 8.8.8.8`. If successful, the path is healthy.
  2. Attempt resolution: `nslookup google.com`. If it returns timeout, query an external DNS directly:
     ```cmd
     nslookup google.com 8.8.8.8
     ```
  3. If the direct query succeeds, the local DNS server is offline. Restarts the DNS Server service on the domain controller or update the client DNS configuration to point to a redundant local DNS server.

---
## Common Mistakes
> [!warning] Avoid These
> **Configuring overlapping DHCP scopes:** Creating identical DHCP scopes on two different unmanaged servers on the same local network. This leads to duplicate IP assignments, causing random network drops and IP conflict warnings across the network.
> **Correct approach:** Use DHCP failover configurations (active/passive) or split scopes ($80/20$ rule) where scope pools do not overlap.

---
## Pro Tips
> [!tip] Field Experience
> When hosting external services behind a PAT gateway, never rely on standard ISP DNS dynamic IPs. Use a Dynamic DNS (DDNS) provider or allocate a dedicated static public IP block from your ISP to prevent connection breaks when WAN interfaces reboot and obtain new public IPs.

---
## Quick Revision Table
| # | Concept | One Line Summary |
|---|---------|-----------------|
| 1 | DORA | Discover (Broadcast), Offer (Unicast), Request (Broadcast), ACK (Unicast). |
| 2 | DHCP Helper | Router command (`ip helper-address`) that relays DHCP broadcasts across networks. |
| 3 | A vs CNAME | A record resolves to a physical IP; CNAME resolves to another domain name (alias). |
| 4 | PTR Record | Reverse DNS record that resolves an IP address back to its FQDN domain name. |
| 5 | PAT / Overload| Mapping thousands of private IPs to one public IP by tracking unique TCP/UDP ports. |

---
## Interview Q&A

**Q1: Describe the step-by-step process of how a router uses PAT to route a connection from an internal host to an external web server.**
A: 
1. An internal host (`192.168.1.15:52300`) sends a packet to web server `142.250.190.46:443`.
2. The packet arrives at the router's inside interface. The router recognizes that the destination requires internet access.
3. The router intercepts the packet and checks its NAT translation table. It translates the private source IP to its public WAN interface IP (`203.0.113.5`).
4. It swaps the source port `52300` with an available unique port (e.g., `10005`) from its pool, saving the mapping: `192.168.1.15:52300 <-> 203.0.113.5:10005`.
5. The router forwards the packet. When the web server replies, it sends packets back to `203.0.113.5:10005`.
6. The router receives the reply, looks up port `10005` in its NAT table, translates the destination back to `192.168.1.15:52300`, and forwards it inside.

**Q2: What is the difference between a Recursive DNS query and an Iterative DNS query?**
A: A **Recursive query** is sent by the client to the local DNS server. The client demands a complete answer (either the IP address or an error). The client does not want referrals. An **Iterative query** is sent by the local DNS server to external name servers. The queried server returns the best answer it has based on its local zone files or cache (e.g., a referral to a TLD server), but does not query other servers itself.

**Q3: Explain what DHCP starvation and DHCP spoofing attacks are, and how you mitigate them on switches.**
A: **DHCP Starvation** is an attack where an attacker broadcasts thousands of DHCP request frames with fake spoofed MAC addresses, exhausting the DHCP scope IP pool so legitimate users cannot obtain leases. **DHCP Spoofing** is when an attacker runs a rogue DHCP server on the LAN, issuing fake gateway and DNS configurations to capture client traffic (Man-in-the-Middle). Both are mitigated by enabling **DHCP Snooping** on the switch. This configuration classifies switch ports as trusted (leading to the real DHCP server) or untrusted (client ports). The switch drops incoming DHCP offers/acks on untrusted ports, and rate-limits DHCP requests to prevent pool starvation.

---
## Related Notes
- [[01-Foundations/02-Networking/N-04 IPv4 Addressing Complete Guide|N-04 IPv4 Addressing Complete Guide]] — Subnet limits and private scope definitions.
- [[03-Identity-and-Core-Services/05-Windows-Server/WS-03 DNS Server — Install and Configure|WS-03 DNS Server — Install and Configure]] — Enterprise DNS setups.
- [[03-Identity-and-Core-Services/05-Windows-Server/WS-04 DHCP Server — Install and Configure|WS-04 DHCP Server — Install and Configure]] — Active Directory DHCP integration and configuration.

