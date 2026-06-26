---
tags: [desktop-support, security, penetration-testing, sysadmin-awareness, L2]
aliases: [penetration-testing-basics, pentesting-awareness]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #none
---

# SEC-02: Penetration Testing Basics

> [!note] Overview
> This note covers the fundamentals of penetration testing from a systems administrator's defense perspective. It details pentesting phases, black/gray/white box models, common attack tools (Nmap, Responder, Mimikatz), and sysadmin security hardening practices.

---
## Concept Overview
- **What it is** — Penetration testing is an authorized, simulated cyberattack designed to evaluate the security of an organization's network, systems, and applications. It aims to identify exploitable weaknesses and test the effectiveness of defensive controls.
- **Why it matters** — Sysadmins must understand basic pentesting concepts to defend their networks effectively. By learning how attackers scan ports, poison local names, and extract credentials from memory, administrators can proactively harden configurations and monitor for indicator signatures.
- **Real job encounter** — Assisting external audit teams during annual security testing, investigating intrusion alerts triggered by pentesting activity, and patch-remediating findings.
- **L1 vs L2 vs L3 responsibility split:**
  - **L1 Resolution**: Identify and report rapid connection failures indicating active network port scans, disable user accounts during suspected compromises, and document basic alert logs.
  - **Escalation Trigger**: Escalate if active credential harvesting, lateral movement, or domain controller mapping patterns are detected.
  - **L2 Resolution**: Configure firewall blocks against scanning source IPs, audit group policy password complexity, disable insecure legacy protocols, and run internal security scans.
  - **L3 Resolution**: Define scope requirements for external pentests, configure Endpoint Detection & Response (EDR) blocking policies, analyze post-pentest reports to orchestrate system hardening, and manage security zones.

---
## Technical Deep Dive

### 1. The 5 Phases of a Penetration Test
Security auditors follow a structured phase model:
1. **Reconnaissance (Information Gathering)**: Gathering OSINT (Open Source Intelligence), DNS mapping, and email address harvesting.
2. **Scanning & Enumeration**: Probing target IPs to discover active systems, open ports, and running service versions (usually using **Nmap**).
3. **Gaining Access (Exploitation)**: Exploiting identified vulnerabilities (e.g. running buffer overflows via **Metasploit**) or performing credential spraying.
4. **Maintaining Access**: Establishing persistent control backdoors (installing webshells, modifying scheduled tasks, or creating hidden admin accounts).
5. **Analysis & Reporting**: Documenting the entry path, listing vulnerabilities exploited, and presenting remediation guidelines to the customer.

### 2. Testing Methodologies
- **Black Box**: The tester has zero prior knowledge of the target network. Simulates an external rogue hacker.
- **Gray Box**: The tester has limited information, such as low-privilege domain user credentials. Simulates a malicious insider or a compromised employee endpoint.
- **White Box**: The tester has full access, including network diagrams, source code, and administrator credentials. Allows for deep, thorough audits.

### 3. Key Attack Vectors & SysAdmin Hardening Targets
Sysadmins must harden their environments against three common attack vectors:
- **LLMNR / NetBIOS Name Poisoning**:
  - *The Attack*: In Windows networks, if a client cannot resolve a hostname via DNS, it broadcasts LLMNR/NBT-NS queries. Attackers use tools like **Responder** to answer these requests with fake responses, forcing the client to send their Windows password hashes.
  - *Hardening*: Disable LLMNR and NetBIOS via Group Policy.
- **Credential Dumping (LSASS Memory)**:
  - *The Attack*: Attackers who gain administrative access run tools like **Mimikatz** to extract cleartext passwords or NTLM hashes directly from the Local Security Authority Subsystem Service (LSASS) memory.
  - *Hardening*: Enable Windows Credential Guard and restrict debug privileges.
- **SMB Relay Attacks**:
  - *The Attack*: Attackers intercept NTLM authentication requests on the local network and relay them to another server to gain administrative access without cracking the password.
  - *Hardening*: Enforce **SMB Signing** on all systems.

---
## Step-by-Step Lab

> [!warning] Pre-requisites
> - A security testing VM running Kali Linux (`192.168.10.99`).
> - A target Rocky Linux 9 server (`SVR-PROD01` with IP `192.168.10.30`).
> - Sudo or root privileges on the target server.

### Step 1: Perform Service and Operating System Scanning using Nmap
From your testing machine (Kali Linux), scan the target server to identify open network doors.

1. Execute a service detection and OS fingerprinting scan:
```bash
# Scan target for open ports, versions, and OS signatures
nmap -sV -O 192.168.10.30
```
**Expected Output:**
```text
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.7 (protocol 2.0)
80/tcp open  http    Apache httpd 2.4.51 ((Rocky Linux))
Device type: general purpose
Running: Linux 5.X
```
*Takeaway:* The scanner identified that ports 22 (SSH) and 80 (HTTP) are open, and fingerprinting detected Rocky Linux running Apache.

### Step 2: Audit Logs for Scanning Indicators (Sysadmin Task)
On the target server, verify that the scanning activity was recorded in the system logs.

1. Log in to `SVR-PROD01` and check SSH connection attempts:
```bash
sudo grep "Failed password" /var/log/secure | tail -n 10
```
**Expected Output:** Displays failed password log entries if the scanner performed brute-force password checks.

2. Check Apache web logs to identify web directory scanning:
```bash
sudo tail -n 20 /var/log/httpd/access_log
```
**Expected Output:** Multiple rapid HTTP GET requests targeting common admin paths (e.g. `/admin`, `/phpmyadmin`).

### Step 3: Mitigation Hardening (Disable LLMNR on Windows Server client)
For multi-OS networks, secure Windows clients against Responder name poisoning.

1. Open Group Policy Management console.
2. Navigate to: `Computer Configuration > Policies > Administrative Templates > Network > DNS Client`.
3. Set **Turn off Multicast Name Resolution** to **Enabled**.
4. Run `gpupdate /force` on client workstations to block LLMNR broadcasts.

---
## Cheat Sheet / Quick Reference

| Tool / Parameter | Primary Security Purpose | Sysadmin Mitigation Action |
|---|---|---|
| `nmap -sS` | Performs a stealthy TCP SYN port scan | Block scanning source IP on local firewall |
| `nmap -p-` | Scans all 65,535 TCP ports | Close unused listening ports |
| **Responder** | Poisons LLMNR/NBT-NS network queries | Disable LLMNR / NetBIOS via GPO |
| **Mimikatz** | Extracts cleartext passwords from LSASS memory | Enable LSA RunAsPPL and Credential Guard |
| **Metasploit** | Automated exploit framework | Keep software and OS patches updated |
| **Hydra** | High-speed network logon brute-forcer | Enforce Account Lockout thresholds |
| `dsregcmd /status` | Checks if workstation is hybrid joined | Verify device state in Active Directory |

---
## Troubleshooting Table

| Problem | Cause | Fix |
|---|---|---|
| Internal security scanner reports false positive vulnerabilities on a host. | Scanner parsed software banners without verifying actual service settings. | Configure "Credentialed Scan" mapping in the scanner, or write a local registry exclusion rule to suppress the alert. |
| Port scan from testing VM times out and returns zero open ports. | Host firewall is drop-filtering all incoming SYN packets. | Ensure local firewalls (`iptables` / Windows Defender Firewall) are configured to drop traffic rather than reject it, which masks host existence. |
| User accounts locked out repeatedly during security testing. | Automated brute-force or credential spray testing is running. | Identify source IP in Event ID 4625 (Logon Failure) logs. Temporarily block the testing host IP at the switch port or firewall. |
| Active directory login fails after disabling LLMNR on Windows clients. | Domain DNS resolution is offline, and clients were relying on LLMNR fallback. | Verify DC DNS service is running. Restore proper DNS IP parameters on client network cards. |
| Mimikatz extracts password hashes even though LSA Protection is enabled. | LSA protection was set but registry key did not enforce UEFI lock. | Enable Credential Guard through Group Policy to isolate credentials inside virtualization-based security (VBS). |

---
## Interview Questions

> [!question] L1 Question
> **Q:** What is LLMNR name poisoning, and how can a sysadmin prevent it?
> **A:** LLMNR (Link-Local Multicast Name Resolution) is a fallback protocol Windows uses when DNS resolution fails. An attacker on the local network running a tool like **Responder** can listen for LLMNR broadcasts and reply with fake name resolutions, forcing the client to send their NTLM credentials. A sysadmin can prevent this by disabling LLMNR globally using Group Policy under `Network > DNS Client > Turn off Multicast Name Resolution`.

> [!question] L2 Question
> **Q:** What is the difference between a stateful Security Group and a stateless Network ACL?
> **A:** A **Security Group** acts as a host firewall for virtual machines. It is stateful; if inbound traffic is allowed on a port (e.g. 443), the outbound return traffic is automatically permitted. A **Network ACL (Access Control List)** acts as a subnet firewall. It is stateless; you must write separate rules for both inbound and outbound traffic paths.

> [!question] L3/Scenario Question
> **Q:** During a penetration test, the auditor gains local admin access to a workstation and immediately executes lateral movement to compromise the entire active directory domain. How do you restrict lateral movement at the workstation level?
> **A:** 
> - **Situation:** Attacker compromises a single workstation and moves laterally to take over the domain.
> - **Task:** Enforce local workstation security to block lateral movement vectors.
> - **Action:** 
>   1. **Disable Local Admin Reuse**: Implement LAPS (Local Administrator Password Solution) to ensure every workstation has a unique, random local administrator password, preventing password reuse.
>   2. **Configure Windows Firewall**: Deploy GPOs to block inbound traffic on ports TCP 139/445 (SMB) and TCP 3389 (RDP) between workstations, preventing workstation-to-workstation connections.
>   3. **Restricting Debug Rights**: Configure GPO to restrict "Act as part of the operating system" and debug privileges to prevent Mimikatz from dumping LSASS memory.
>   4. **Restricting Domain Admin Logins**: Prohibit Domain Administrators from logging in to standard workstations. They should only log in to secure Domain Controllers or Dedicated Admin Workstations (PAWs).
> - **Result:** The attacker is contained on the single compromised workstation, preventing lateral expansion.

---
## Seedha Simple Mein
*Seedha simple mein: Penetration Testing ka matlab hai security experts ke zariye network par hack hone ki check simulation karna. Sysadmins ko basic attack tools (jaise Nmap, Responder) ki working samajhni hoti hai taaki wo insecure legacy protocols (jaise LLMNR) ko block kar sakein aur corporate passwords ko leak hone se bacha sakein.*

---
## Related Notes
- [[04-Cloud-and-Security/09-Security/SEC-01 Vulnerability-Scanning|SEC-01 Vulnerability-Scanning]] — Proactive scanning systems.
- [[03-Identity-and-Core-Services/06-Active-Directory/WS-05 Group Policy — Complete Guide|WS-05 Group Policy — Complete Guide]] — Deploying security templates globally.
- [[04-Cloud-and-Security/09-Security/Incident-Response-Playbook|Incident-Response-Playbook]] — Incident handling workflows when attacks occur.

---
*Tags: #desktop-support #security #penetration-testing #sysadmin-awareness #L2*
