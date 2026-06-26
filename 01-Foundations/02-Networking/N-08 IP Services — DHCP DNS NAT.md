---
tags: [sysadmin, networking, services, dhcp, dns, nat]
aliases: [N-08, DHCP, DNS, NAT]
created: 2026-06-26
status: #complete
difficulty: #intermediate
cert-relevant: #ccna
---

> [!NOTE|color-yellow]
> 🌐 **NETWORKING**

`#complete` `#intermediate` `#ccna`

# N-08: IP Services — DHCP DNS NAT

> [!abstract] Overview
> Yeh note key Layer 7 network helper services cover karta hai: DHCP address provisioning, DNS name resolution, aur NAT translation architectures. Yeh samajhna ek support engineer ke liye isliye zaroori hai taaki IP conflicts, name resolution errors, aur internet connectivity issues ko effectively troubleshoot kiya ja sake.

---
## 🧠 Concept Overview

- **What it is** — IP services woh background mechanisms hain jo devices ko network join karne (DHCP), names ko IPs mein translate karne (DNS), aur internet par private networks ko route karne (NAT) mein madad karte hain.
- **Why it matters** — Inke bina, har device ko manual configuration ki zaroorat padegi aur internet access impossible ho jayega.
- **Where you see this** — Har enterprise network, branch office, aur home router mein yeh services continuously run hoti hain.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic task — Check client IP configuration, ping test, and DNS resolution check. |
| **L2** | Configure DHCP scopes, update DNS records, and fix basic NAT routing issues. Escalate hardware/software failures. |
| **L3** | Architecture and design of enterprise DNS/DHCP infrastructure (IPAM), complex PAT/NAT policies, and HA deployments. |

> [!tip] Seedha Simple Mein
> *DHCP automatically systems ko IP address deta hai. DNS human-friendly website names ko IP addresses mein convert karta hai. NAT/PAT private networks ko single public IP share karke internet access karne ki permission deta hai.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **IP Services** are like **hotel infrastructure utilities** because...
>
> - **DHCP** is the reception desk. When you check in, the clerk assigns you a room number (IP), gives you a map (Default Gateway), and the info desk number (DNS server).
> - **DNS** is the phone book directory. To call the "Penthouse Suite", you look it up to find their actual extension.
> - **NAT / PAT** is the hotel switchboard. All rooms share one external phone number. The switchboard tracks your room number so replies reach you correctly.

---
## 🔬 Technical Deep Dive

### 1. DHCP Operation: The DORA Process

> [!info] Key Concept
> DHCP (Dynamic Host Configuration Protocol) operates at the application layer over UDP ports 67 (server) and 68 (client). DORA is Discover, Offer, Request, ACK.

- **Discover**: Client broadcasts "Is there a DHCP server?"
- **Offer**: Server replies "I have room 192.168.1.50 for you."
- **Request**: Client broadcasts "I accept that offer."
- **ACK**: Server confirms and provides options.

**Key Options:**
- Option 3: Default Gateway
- Option 6: DNS Servers
- Option 15: Domain Name Suffix

> [!danger] Common Mistake
> **Configuring overlapping DHCP scopes:** Creating identical DHCP scopes on two different unmanaged servers on the same local network leads to duplicate IP assignments. Always use DHCP failover or split scopes ($80/20$).

### 2. DNS Resolution Process

> [!info] Key Concept
> DNS translates hostnames to IP addresses over UDP/TCP port 53.

1. **Host Cache Check:** Client checks local hosts file and cache.
2. **Recursive Query:** Client asks its Local DNS Server.
3. **Iterative Queries:** Local DNS queries Root (`.`), TLD (`.com`), then Authoritative Server.
4. **Authoritative Response:** Server returns the IP.
5. **Caching:** Local DNS caches based on TTL.

**DNS Record Types:**
- **A Record:** IPv4 address.
- **AAAA Record:** IPv6 address.
- **CNAME:** Alias.
- **MX:** Mail Exchanger.
- **PTR:** Reverse DNS (IP to Name).

### 3. Network Address Translation (NAT) & PAT

> [!info] Key Concept
> NAT translates private RFC 1918 IPs to public routable IPs.

- **Static NAT:** 1-to-1 mapping.
- **Dynamic NAT:** Many-to-many from a pool.
- **PAT (NAT Overload):** Many-to-one using unique TCP/UDP ports.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - A Windows Server (2019/2022) virtual machine running in Hyper-V or VMware.

### Step 1: Install DHCP Role on Windows Server

```powershell
# Open Server Manager and add DHCP Server role
Install-WindowsFeature -Name DHCP -IncludeManagementTools
```

> [!success] Expected Output
> ```
> Success Restart Needed Exit Code      Feature Result
> ------- -------------- ---------      --------------
> True    No             Success        {DHCP Server, RSAT-DHCP}
> ```

### Step 2: Configure a DHCP Scope

1. Open Server Manager -> Tools -> **DHCP**.
2. Right-click **IPv4**, select **New Scope** (Name: `Client_Scope`).
3. Set Range: `192.168.10.100` to `.200`, Subnet Mask: `255.255.255.0`.
4. Add Exclusions: `192.168.10.100` to `.110`.

### Step 3: Verify Client Lease

```cmd
# Run on the client to refresh IP
ipconfig /renew
```

> [!success] Expected Output
> Client receives an IP between 192.168.10.111 and .200.

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `ipconfig /renew` | Renews the DHCP lease for the interface | `ipconfig /renew` |
| `ipconfig /flushdns` | Clears local DNS cache resolver entries | `ipconfig /flushdns` |
| `nslookup` | Checks A-record resolution | `nslookup www.google.com` |
| `nslookup -type=mx` | Queries MX records for a domain | `nslookup -type=mx google.com` |
| `nslookup -type=ptr` | Performs reverse DNS lookup | `nslookup -type=ptr 8.8.8.8` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| APIPA Address (`169.254.x.x`) | DHCP Discover blocked by router | Add `ip helper-address <DHCP_IP>` on router gateway interface |
| Cannot open websites by name | DNS server offline or wrong config | Test with `nslookup google.com 8.8.8.8`. Restart local DNS or fix client config |
| Duplicate IPs on network | Overlapping DHCP scopes | Use failover or split scope design |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Workstations Not Getting IP

> [!example] Ticket
> "Users on VLAN 10 report no network access and have 169.254.x.x IPs."

**L1 Response:** Confirm APIPA on clients. Check if clients can ping other clients on the same VLAN. Verify physical connectivity.
**Escalation Trigger:** Issue affects whole VLAN and basic connectivity is fine.
**L2 Resolution:** Login to gateway router and configure DHCP relay: `ip helper-address 10.0.0.50` on the VLAN interface.

---
## 🎤 Interview Questions

> [!question] Q1: Describe how a router uses PAT to route a connection from an internal host to an external server.
> **Answer:** 
> 1. Host (`192.168.1.15:52300`) sends packet to `142.250.190.46:443`.
> 2. Router intercepts, translates private IP to its public IP, and swaps source port to a unique port (e.g., `10005`).
> 3. Router saves mapping in NAT table. When reply comes to `10005`, router translates it back to `192.168.1.15:52300`.

==**Exam Tip:** PAT is also known as NAT Overload.==

> [!question] Q2: What is the difference between a Recursive and an Iterative DNS query?
> **Answer:** Recursive is from client to local DNS (demands full answer). Iterative is from local DNS to external servers (accepts referrals).

> [!question] Q3: What are DHCP Starvation and Spoofing?
> **Answer:** Starvation exhausts the IP pool via fake MACs. Spoofing is a rogue server giving fake gateways (MitM). 
> **Mitigation:** Enable DHCP Snooping on switches.

---
## 🔗 Related Notes

- [[01-Foundations/02-Networking/N-04 IPv4 Addressing Complete Guide|N-04 IPv4 Addressing Complete Guide]] — Subnet limits and private scope definitions.
- [[03-Identity-and-Core-Services/05-Windows-Server/WS-03 DNS Server — Install and Configure|WS-03 DNS Server — Install and Configure]] — Enterprise DNS setups.
- [[03-Identity-and-Core-Services/05-Windows-Server/WS-04 DHCP Server — Install and Configure|WS-04 DHCP Server — Install and Configure]] — Active Directory DHCP integration.
