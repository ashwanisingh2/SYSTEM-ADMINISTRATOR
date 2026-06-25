---
tags: [desktop-support, networking, dns, L2]
aliases: [domain-name-system, name-resolution]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #ccna
---

# DNS

---

## Concept Overview
- **What it is**: The Domain Name System (DNS) is a hierarchical, decentralized naming system that translates human-readable domain names (e.g., `google.com`) into numerical IP addresses (e.g., `142.250.190.46`) required to route network traffic.
- **Why it matters for a support engineer**: DNS is the foundation of network connectivity and directory services (Active Directory). When DNS fails, users cannot access websites, connect to local file shares, or log into the domain.
- **Where you encounter this in real job**: Diagnosing "Server not found" errors, adding mail server MX records, troubleshooting Active Directory domain controller discovery errors, and configuring network interfaces.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Clears local client DNS caches, checks local DNS server IP configurations, and runs basic nslookup queries.
  - **L2**: Manages local DNS zone records, troubleshoots name resolution timeouts, and configures forwarders.
  - **L3**: Designs public DNS routing architectures, manages DNS zone transfers, and configures DNSSEC security validation.

---

## Technical Deep Dive

### 1. DNS Record Types
DNS databases store data in Resource Records (RRs). Key records include:
- **A Record**: Maps a domain name to a 32-bit IPv4 address (e.g., `web.company.com -> 192.168.10.25`).
- **AAAA Record**: Maps a domain name to a 128-bit IPv6 address.
- **CNAME (Canonical Name)**: Creates an alias pointing to another domain name (e.g., `www.company.com -> company.com`).
- **MX (Mail Exchanger)**: Specifies the mail servers responsible for receiving email for the domain, with priority values.
- **PTR (Pointer)**: Maps an IP address to a domain name (used in Reverse Lookup zones).
- **SRV (Service Record)**: Identifies the host and port for specific services (critical for Active Directory domain controller discovery).
- **TXT (Text)**: Stores text metadata, commonly used for email security validation (SPF, DKIM, DMARC records).

### 2. DNS Resolution Flow
When a client requests a domain name, the system queries through the following steps:
1. **Local Cache & Hosts File**: Client checks its local memory cache and the local `%systemroot%\System32\drivers\etc\hosts` file.
2. **Recursive Resolver**: If not found, the client queries its configured local DNS server (Resolver).
3. **Root Servers (`.`)**: If the resolver cannot answer, it queries the internet Root Name Servers.
4. **TLD Servers**: The Root directs the resolver to the Top-Level Domain (TLD) servers (e.g., `.com` or `.net`).
5. **Authoritative Server**: The TLD directs the resolver to the domain's Authoritative Name Server, which returns the IP address.
6. **Result Cached**: The resolver caches the IP for a duration defined by the record's TTL (Time-To-Live) and returns the IP to the client.

```
Client -> [Local Cache/Hosts] -> [Local DNS Resolver] -> [Root "."] -> [TLD ".com"] -> [Authoritative NS]
```

### 3. Forwarders vs. Conditional Forwarders
- **Standard Forwarder**: Configures the local DNS server to forward all queries it cannot resolve internally to an external DNS server (e.g., `8.8.8.8`).
- **Conditional Forwarder**: Configures the DNS server to forward queries for specific domain names (e.g., `partner.local`) to designated IP addresses (e.g., the partner's internal DNS server).

---

## Commands & Syntax

### PowerShell
```powershell
# Query DNS records for a domain using PowerShell
Resolve-DnsName -Name "google.com" -Type MX

# Clear the local client DNS resolver cache
Clear-DnsClientCache
```

### CMD / Run Box
```cmd
REM Flush the DNS resolver cache
ipconfig /flushdns
REM Query DNS records using the interactive nslookup tool
nslookup -type=srv _ldap._tcp.dc._msdcs.company.local
```

### GUI Path
> Start -> Windows Tools -> **DNS** (DNS Manager console on Domain Controller)
> Or: Control Panel -> Network and Sharing Center -> Adapter Properties -> IPv4 Properties -> **Preferred DNS Server**

### Important Registry Paths
```
HKLM\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters\
HKLM\SYSTEM\CurrentControlSet\Services\DNS\Parameters\
```

### Key Event IDs
| Event ID | Meaning | Where to Check |
|----------|---------|----------------|
| 1014 | DNS name resolution timeout | System Log |
| 4013 | DNS server failed to load Active Directory integrated zones | Directory Service |

---

## Real-World Scenarios

### Scenario 1: Users Cannot Access Local Web App After Server Migration
**User Complaint:** "The internal timesheet portal is down. I get a page cannot be displayed error in my browser."
**Your First 3 Checks:**
1. Check if the server is powered on and reachable by its IP address.
2. Check if the client can resolve the portal's domain name using `nslookup`.
3. Check the DNS record details in the DNS Manager console.
**Diagnosis Steps:**
1. Ping the portal IP directly. Ping succeeds, confirming network reachability.
2. Run a DNS lookup on the client: `nslookup timesheet.company.local`
   - Output: `Name: timesheet.company.local, Address: 192.168.10.88`
3. Verify the actual server IP. The server was migrated to a new subnet and its IP changed to `192.168.10.99`.
   - The DNS record still points to the old IP (`192.168.10.88`).
**Root Cause:** The DNS A record was not updated in the local DNS zone database after the server migration.
**Fix:** Open the DNS Manager console on the DC, locate the A record for `timesheet.company.local`, and update the IP to `192.168.10.99`. Run `ipconfig /flushdns` on client machines.
**Prevention:** Integrate IP address management (IPAM) tools to automate DNS record updates during VM migrations.
**Ticket Close Note:** "Updated DNS A record to point to the new IP (192.168.10.99). Flushed DNS cache on client workstations. Verified access. Closing ticket."

### Scenario 2: Active Directory Workstation Logins Fail (DNS Timeout)
**User Complaint:** "I cannot log into my computer. I get an error stating the security database on the server does not have a computer account for this workstation trust relationship."
**Your First 3 Checks:**
1. Verify if the client workstation can ping the Domain Controller IP.
2. Check the DNS server IP configuration on the client's network adapter.
3. Test if the client can resolve Active Directory service location (SRV) records.
**Diagnosis Steps:**
1. Ping the Domain Controller IP. Ping succeeds, indicating Layer 3 connection.
2. Execute `ipconfig /all` on the client.
   - The Preferred DNS server is set to `8.8.8.8` (Google public DNS) instead of the local Domain Controller IP.
3. Run: `nslookup _ldap._tcp.dc._msdcs.lab.local`
   - Output: `*** google.com can't find _ldap._tcp.dc._msdcs.lab.local: Non-existent domain`
**Root Cause:** The client's DNS was configured with public DNS servers, preventing it from resolving internal Active Directory SRV records required for domain controller discovery and authentication.
**Fix:** Reconfigure the client network adapter properties to use the local DC IP (`192.168.10.10`) as the Preferred DNS server.
**Prevention:** Configure DHCP options (Option 6 - DNS Servers) to enforce local DNS distribution across all subnets.
**Ticket Close Note:** "Reconfigured client Preferred DNS server to point to the Domain Controller IP (192.168.10.10). Verified successful domain login. Closing ticket."

---

## Critical Points

> [!danger] Never Do This
> - Deploy public DNS servers (like `8.8.8.8` or `1.1.1.1`) directly in the network adapter configurations of domain-joined Active Directory workstations.
> - Public DNS servers cannot resolve your internal domain name (`company.local`), which breaks domain logins, group policy applications, and access to internal shares.
> - Set preferred DNS to your local DCs, and configure public DNS as Forwarders on the DC instead.

> [!warning] Common Trap
> - Modifying the local hosts file to quickly bypass a DNS issue without documenting it.
> - The hosts file takes priority over DNS. If the server IP changes in the future, the hosts file entry will continue to route traffic to the old IP, causing connection failures that are difficult to diagnose.
> - Always fix the actual DNS record in the DNS server database.

> [!tip] Senior Engineer Tip
> - If you suspect a DNS server is having performance issues, check the Event Viewer Directory Service logs for Event ID 4013. This event indicates the DNS server cannot load Active Directory integrated zones, which is usually caused by replication delays during DC reboots.

> [!success] Verification Steps
> - Run: `nslookup -type=srv _ldap._tcp.dc._msdcs.company.local` to verify Active Directory SRV records.
> - Confirm the output displays the correct Domain Controller hostnames and IPs.

> [!question] Interview Alert
> - "What are SRV records and why are they critical for Active Directory?"
> - Answer: "SRV (Service Location) records are DNS resource records that map specific service names (like LDAP or Kerberos) to the hostnames and ports running those services. They are critical for Active Directory because clients use them to locate Domain Controllers during domain join and authentication."

---

## Common Mistakes & Fixes
| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| Stale DNS records in cache | DNS TTL is too high | Flushed local cache with `ipconfig /flushdns` and configure lower TTL values on dynamic zones. |
| Forgetting Reverse Lookup zones | Assuming Forward lookup is sufficient | Create Reverse Lookup zones and check **Create associated PTR record** on A records. |
| Duplicate A records for one host | Overlapping static reservations | Delete outdated records to prevent round-robin routing conflicts. |

---

## Lab Exercise

**Objective:** Configure a forward lookup zone record in DNS Manager, configure a conditional forwarder, and verify name resolution on a client machine.
**Time Required:** 20 minutes
**Environment Needed:** A Windows Server 2022 VM (Domain Controller) and a Windows 11 VM.
**Pre-requisites:** AD DS and DNS server roles active on the DC.

**Steps:**
1. Open DNS Manager: Open Server Manager -> `Tools` -> **DNS**.
2. Create A Record: Expand `Forward Lookup Zones` -> Right-click your domain -> Select **New Host (A or AAAA)**.
   - Name: `intranet`
   - IP Address: `192.168.10.75`
   - Check **Create associated pointer (PTR) record**.
   - Click **Add Host** -> Click **OK**.
3. Create Conditional Forwarder: Right-click **Conditional Forwarders** -> Select **New Conditional Forwarder**.
   - DNS Domain: `partner.local`
   - IP Addresses: `192.168.20.10` -> Click **OK**.
4. Test Client: Log into your Windows 11 VM. Open a command prompt.
5. Flush cache and test resolution:
   ```cmd
   ipconfig /flushdns
   nslookup intranet.lab.local
   ```
   - Expected output: `Address: 192.168.10.75`
6. Verification: Run the verification command.
   ```powershell
   Resolve-DnsName -Name "intranet.lab.local"
   ```

**Success Criteria:** The DNS server resolves the new A record and PTR record, and the client successfully queries `intranet.lab.local`.
**Common Failures:** Failed PTR resolution if the corresponding Reverse Lookup zone does not exist.

---

## Interview Questions & Answers

### Basic (L1 Level)
**Q: What is the difference between a forward DNS lookup and a reverse DNS lookup?**
A: A forward lookup translates a host domain name into an IP address (e.g., `web.com -> 192.168.10.10`) using an A record. A reverse lookup translates an IP address into a host domain name (e.g., `192.168.10.10 -> web.com`) using a PTR record in a reverse lookup zone.

**Q: How do you clear the DNS cache on a Windows client?**
A: I open a command prompt and run the command `ipconfig /flushdns`. In PowerShell, I can run `Clear-DnsClientCache`. This clears the local DNS resolver cache, forcing the OS to query the DNS server for fresh records.

### Intermediate (L2 Level)
**Q: What is the difference between a recursive DNS query and an iterative DNS query?**
A: In a recursive query, the client requests the DNS resolver to return the final answer (the resolved IP) or an error, and the resolver handles the search. In an iterative query, the resolver queries other name servers (Root, TLD, Authoritative) step-by-step; each server returns the IP of the next server to query, until the resolver finds the authoritative answer.

### Advanced (L3/Senior Level)
**Q: A user complains they cannot connect to resources in a newly established partner organization's Active Directory domain (`partner.local`). An ExpressRoute is active. Explain your resolution strategy.**
A:
- **Situation**: Users could not access resources in a partner domain (`partner.local`) despite network routing being active.
- **Task**: I needed to establish name resolution support for the partner domain.
- **Action**: I verified that the network path was open. I opened DNS Manager on our Domain Controller, created a **Conditional Forwarder** targeting `partner.local`, and pointed the query target to the partner's Domain Controller IP (`10.50.10.10`).
- **Result**: DNS queries for `partner.local` were forwarded directly to the partner DNS server, establishing name resolution and resource access.

### HR / Behavioral
**Q: Tell me about a time you resolved a major network service outage. How did you identify DNS as the root cause?**
A: Users reported they could not access our local web application. I pinged the application server IP directly and got a response, but pinging the domain name failed. I checked the DNS server console, found the DNS service was stopped, restarted it, and verified name resolution returned to service.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **What**: The naming directory system translating domain names into IP addresses.
> **Why**: Critical for domain logins, web application access, and directory service routing.
> **How**: Configure Forwarders, check A/SRV/PTR records, and manage resolver caches.
> **Command**: `nslookup`
> **Interview Answer Starter**: "In my experience, DNS troubleshooting starts by verifying that the client adapter points to the correct local DNS servers..."

**Key Numbers to Remember:**
- DNS Port: 53 (UDP for standard queries, TCP for zone transfers)
- Windows host file path: `%systemroot%\System32\drivers\etc\hosts`
- DNS event timeout: Event ID 1014

**3 Things Interviewer Wants to Hear:**
1. DNS settings on AD client adapters must point to Domain Controllers
2. Role of SRV records in Active Directory operation
3. Difference between Forwarders and Conditional Forwarders

---

## Related Notes
- [[01-Foundations/02-Networking/OSI-Model|OSI Model]] — Details Application layer protocols.
- [[01-Foundations/02-Networking/DHCP|DHCP]] — Configures automatic IP and DNS distributions.
- [[03-Identity-and-Core-Services/05-Windows-Server/DNS-Server|DNS Server]] — Deep dive into Windows Server DNS configuration.

---

## Tags
#desktop-support #networking #dns #L2 #interview-topic #lab-complete #daily-use

