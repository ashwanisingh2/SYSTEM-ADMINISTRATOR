---
tags: [sysadmin, windows-server, dns, name-resolution]
difficulty: Intermediate
lab-required: Yes
read-time: 15 mins
---

# WS-03: DNS Server — Install and Configure

> [!abstract] Overview
> This note covers DNS configuration inside a Windows Server environment. It details forward/reverse zone creation, AD-integrated security, forwarder types, record hygiene (scavenging), and diagnostic methodologies.

---
## Concept
Think of DNS as the digital switchboard and GPS directory of your company. 
When a workstation wants to log a user into the domain, it doesn't know which physical server is the Domain Controller. It queries the DNS server: "Show me the service records (`_msdcs`) pointing to the LDAP servers in this domain." 

The DNS server checks its directory database, finds the active record pointing to the DC's IP address, and hands it back. If the DNS server goes offline or has corrupted records, workstations cannot find the Domain Controller, users cannot log in, and all domain activity stops.

*Seedha simple mein: Active Directory bina DNS ke nahi chal sakta. DNS domain records (SRV records) store karta hai jisse devices ek dusre ko find kar sakein. Forward lookup zone host names ko IP mein badalta hai, aur Reverse lookup zone IP ko host names mein.*

---
## Technical Deep Dive

### 1. Active Directory Integration
When installing DNS on a Domain Controller, zones should be configured as **Active Directory-Integrated Zones**:
- **Multi-Master Replication:** DNS data is stored directly in the Active Directory database (`ntds.dit`) and replicated automatically to all other DCs, eliminating the single point of failure of a primary/secondary zone file transfer.
- **Secure Dynamic Updates:** Only domain-joined computers can register their IP addresses dynamically, preventing malicious host spoofing.

### 2. Forward vs. Reverse Lookup Zones
- **Forward Lookup Zone (FLZ):** Translates user-friendly hostnames (e.g., `srv-file.company.local`) to IP addresses (`192.168.10.20`).
- **Reverse Lookup Zone (RLZ):** Translates IP addresses back to hostnames. Uses the special domain suffix `in-addr.arpa`. Used for security validation, mail server verification, and system logging.

### 3. Record Types (Standard Windows Configuration)
- **A & AAAA:** Basic host-to-IP pointers.
- **CNAME:** Canonical name (alias).
- **MX:** Mail exchanger records.
- **PTR:** Pointer records located in the RLZ.
- **SRV (Service Location Record):** Used by Active Directory to locate services (e.g., LDAP on port 389, Kerberos on port 88). Location prefix: `_service._protocol.domain`.
- **SOA & NS:** Define zone authority parameters and name server paths.

### 4. Resolving External Queries: Forwarders vs. Root Hints
If a client queries the DNS server for an external website (e.g., `google.com`):
- **DNS Forwarders:** The DNS server forwards all external queries directly to a upstream recursive resolver (e.g., `8.8.8.8` or your ISP DNS). This is fast and caches results locally.
- **Conditional Forwarders:** Redirects queries for a specific domain name prefix to a specific target DNS server. Essential in forest trusts and multi-domain companies (e.g., forward queries for `partner.com` to `10.50.0.10`).
- **Root Hints:** If no forwarders are configured, the DNS server queries the 13 public internet root servers directly, resolving iteratively.

### 5. DNS Scavenging (Record Hygiene)
Over time, dynamic hosts (like DHCP laptops) disconnect from the network permanently, leaving "stale" records in the DNS database.
- **Scavenging:** Automatically deletes stale dynamic DNS records after a designated aging period.
  - **No-refresh interval:** Time during which the client cannot refresh its timestamp (prevents database write replication overhead). Default: 7 days.
  - **Refresh interval:** Time during which the client is expected to update its timestamp. Default: 7 days.
  - **Scavenging Period:** Once the dynamic age exceeds 14 days, the record is flagged as stale and removed.

---
## Lab — Step by Step
> [!info] Lab Setup Needed
> A Windows Server 2022 VM configured as a Domain Controller (which installs DNS by default) or a member server.

### Step 1: Create a Reverse Lookup Zone
1. Open Server Manager -> Tools -> **DNS**.
2. Expand the server node, right-click **Reverse Lookup Zones**, and select **New Zone**.
3. In the wizard, select **Primary Zone**. Check **Store the zone in Active Directory** (if on a DC). Click Next.
4. Active Directory Zone Replication Scope: Select **To all DNS servers running on domain controllers in this domain**. Click Next.
5. Select **IPv4 Reverse Lookup Zone**. Click Next.
6. Network ID: Enter the first three octets of your local subnet: `192.168.10`. Click Next.
7. Dynamic Update: Select **Allow only secure dynamic updates** (recommended for AD). Click Next, then **Finish**.

### Step 2: Create a Host Record and PTR Record
1. Expand **Forward Lookup Zones** -> `company.local`.
2. Right-click in the empty right pane and select **New Host (A or AAAA)**.
3. Name: `srv-backup`.
4. IP Address: `192.168.10.50`.
5. Check **Create associated pointer (PTR) record**.
6. Click **Add Host**.
7. **Verify:** Open the Reverse Lookup Zone (`10.168.192.in-addr.arpa`). Click Refresh. Verify that the PTR record for `192.168.10.50` pointing to `srv-backup.company.local` is present.

### Step 3: Configure Upstream Forwarders
1. In DNS Manager, right-click the root Server Name (e.g., `SVR-DC01`) and select **Properties**.
2. Go to the **Forwarders** tab. Click **Edit**.
3. Add IP: `8.8.8.8` (Google public DNS) and `1.1.1.1` (Cloudflare).
4. Click OK, then click Apply. The server can now resolve external web domains.

---
## Diagnostic and Troubleshooting commands
Advanced content only — basics in [[Basic networking commands]]

```cmd
:: Windows Command Prompt
:: Query local DNS server to resolve srv-backup record
nslookup srv-backup.company.local

:: Direct query targeting a specific DNS server IP
nslookup srv-backup.company.local 192.168.10.10

:: Verify SRV records for Active Directory LDAP service
nslookup -type=srv _ldap._tcp.dc._msdcs.company.local

:: Clear local workstation DNS cache
ipconfig /flushdns

:: Force a client machine to reregister its host records on the DNS server
ipconfig /registerdns
```

---
## Troubleshooting Scenarios

**Scenario 1:**
- **Problem:** Workstations cannot join the Active Directory domain. The error states: "An active directory domain controller for the domain company.local could not be contacted. DNS name does not exist."
- **Root Cause:** The workstation is using an external DNS server or the DNS server has missing SRV records in the `_msdcs` zone.
- **Fix:**
  1. From the client workstation, run:
     ```cmd
     nslookup -type=srv _ldap._tcp.dc._msdcs.company.local
     ```
  2. If the query returns a timeout or failure, log into the DNS server.
  3. Verify that the zone `_msdcs.company.local` exists. If missing, restart the Netlogon service on the Domain Controller to force SRV reregistration:
     ```powershell
     Restart-Service netlogon
     ```
  4. Change the workstation's IPv4 configuration to point exclusively to the local DNS server IP.

**Scenario 2:**
- **Problem:** The DNS server database is bloated with thousands of records pointing to dead IPs, causing slow name resolutions and replication lag.
- **Root Cause:** DNS Scavenging is disabled on the server and zones.
- **Fix:**
  1. Open DNS Manager. Right-click the Server name and select **Properties**.
  2. Go to the **Advanced** tab. Check **Enable automatic scavenging of stale records**. Set the interval to `7 days`.
  3. Right-click the zone `company.local` -> **Properties**.
  4. On the General tab, click **Aging**.
  5. Check **Scavenge stale resource records**. Set No-refresh and Refresh intervals to `7 days`.
  6. To force a cleanup run immediately, right-click the server node -> **Scavenge Stale Resource Records**.

---
## Common Mistakes
> [!warning] Avoid These
> **Forgetting to configure Forwarders when using Root Hints on locked down networks:** Relying on default Root Hints when port 53 (DNS) outbound is blocked on the enterprise firewall. This results in the DNS server failing to resolve any internet addresses.
> **Correct approach:** Configure forwarders targeting approved corporate upstream DNS servers, and ensure the firewall permits outbound port 53 traffic from those specific forwarder IPs.

---
## Pro Tips
> [!tip] Field Experience
> When configuring multi-site Active Directory domains, always keep DNS servers on every local DC. Configure local clients to use their local DC as the preferred DNS server, and the nearest remote DC as the secondary. This keeps local directory queries within the LAN, preventing slow logons caused by DNS traffic crossing WAN links.

---
## Quick Revision Table
| # | Concept | One Line Summary |
|---|---------|-----------------|
| 1 | AD-Integrated Zone| DNS zone stored in the AD database (`ntds.dit`), utilizing multi-master replication. |
| 2 | SRV Record | Directory pointer that identifies host IP and port locations for LDAP and Kerberos services. |
| 3 | PTR Record | Pointer record residing in the Reverse Lookup Zone resolving IPs back to domain names. |
| 4 | DNS Scavenging | Automated housekeeping service that deletes stale dynamic DNS host timestamps. |
| 5 | Conditional Forward| Upstream redirect rule targeting a specific DNS server based on domain name suffix. |

---
## Interview Q&A

**Q1: Why is DNS critical for Active Directory Domain Services operation?**
A: Active Directory relies entirely on DNS to locate its logical resources. When a client workstation attempts to log in, join the domain, or locate services (like LDAP or Kerberos), it does not use broadcasts. Instead, it queries DNS for specific Service Location (SRV) records (e.g., `_ldap._tcp.dc._msdcs.company.local`). If the DNS server is down, misconfigured, or has stale entries, clients cannot locate the Domain Controllers, causing authentication, Group Policy application, and replication processes to fail.

**Q2: A client reports that reverse DNS lookup fails for half their servers. Explain how you would troubleshoot this.**
A: 
- **Situation:** Reverse name resolution (PTR lookups) is failing on a subset of corporate servers.
- **Task:** Verify Reverse Lookup Zone existence and record auto-registration status.
- **Action:** First, I will check if a Reverse Lookup Zone (RLZ) matching the IP subnet of those servers exists in DNS Manager. If the subnet is `192.168.20.0/24`, there must be a `20.168.192.in-addr.arpa` zone. Second, I will check the network adapter settings on the failing servers. I need to verify that "Register this connection's addresses in DNS" is enabled in the Advanced TCP/IP settings. Third, I will check if the zone allows secure dynamic updates.
- **Result:** Creating the missing RLZ and running `ipconfig /registerdns` on the servers registers the PTR records and resolves the lookup failures.

**Q3: Explain the difference between the No-Refresh Interval and the Refresh Interval in DNS aging and scavenging.**
A: In DNS scavenging, the **No-Refresh Interval** is the period of time (default 7 days) after a record timestamp is created or updated during which the DNS server will reject any update requests from the client for that same record. This prevents unnecessary Active Directory replication write traffic. The **Refresh Interval** is the period (default 7 days) immediately following the no-refresh interval during which the DNS server accepts timestamp updates from the client. If the refresh interval expires without the client updating its timestamp, the record is flagged as stale.

---
## Related Notes
- [[03-Identity-and-Core-Services/06-Active-Directory/WS-02 Active Directory Domain Services|WS-02 Active Directory Domain Services]] — The AD DS role installation process.
- [[03-Identity-and-Core-Services/05-Windows-Server/WS-04 DHCP Server — Install and Configure|WS-04 DHCP Server — Install and Configure]] — Automating dynamic DNS client updates.
- [[03-Identity-and-Core-Services/06-Active-Directory/WS-05 Group Policy — Complete Guide|WS-05 Group Policy — Complete Guide]] — Deploying DNS options via policies.

