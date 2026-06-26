---
tags: [sysadmin, windows-server, nps, radius, security]
aliases: [nps-radius]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #none
---

# WS-17: NPS and RADIUS Server

**Verification:**
- [ ] Install Network Policy and Access Services (NPAS) role on Windows Server
- [ ] Register NPS in Active Directory Domain Services
- [ ] Configure RADIUS client (e.g. Virtual Switch, Router, or VPN server)
- [ ] Verify NPS accounting and local log files configuration
- [ ] Test client connection and verify NPS Event Viewer logs (Event ID 6272/6273)

> [!abstract] Overview
> This note covers Network Policy Server (NPS) in Windows Server 2022. It details NPS as a RADIUS server/proxy, centralizing Authentication, Authorization, and Accounting (AAA), setting up RADIUS clients, designing connection request policies and network policies, and troubleshooting RADIUS access issues.

---
## Concept Overview
- **What it is** — NPS is Microsoft's implementation of a RADIUS (Remote Authentication Dial-In User Service) server and proxy. It acts as a central gatekeeper for network access.
- **Why it matters for a support engineer** — In enterprise networks, you don't want to configure user credentials on every switch, Wi-Fi access point, or VPN gateway. NPS centralizes this so that any device checks with Active Directory before letting a user connect.
- **Where you encounter this in real job** — Configuring corporate Wi-Fi authentication (802.1X), securing network switches (switch port security), or controlling remote access VPN authentication.
- **L1 vs L2 vs L3 responsibility split for this topic:**
  - **L1 Resolution**: Check basic user network connectivity, verify user accounts are not locked out in Active Directory, check user group membership in AD, and gather client-side Wi-Fi/VPN error messages.
  - **Escalation Trigger**: Escalate to L2 if client fails to connect despite correct credentials, indicating policies, shared secrets, or certificates are blocking authentication.
  - **L2 Resolution**: Configure RADIUS Clients and Shared Secrets in NPS, inspect NPS logs (Event Viewer Custom Views / IAS log files) to diagnose connection failures, and test connection request policies.
  - **L3 Resolution**: Design connection request policies and network policies, manage enterprise CA certificates for PEAP/EAP-TLS, configure RADIUS proxies for multi-forest routing, and implement SQL Server accounting.

*Seedha simple mein: NPS ek central security guard ki tarah hai. Jab bhi koi user corporate Wi-Fi, VPN, ya switch se connect hone ki koshish karta hai, toh woh device (RADIUS Client) user ke credentials check karne ke liye NPS ke paas bhejta hai. NPS Active Directory se pooch kar permission grant ya deny karta hai.*

---
## Technical Deep Dive

### 1. RADIUS (AAA) Architecture
RADIUS operates on the AAA model:
- **Authentication**: Verifying who the user is (username/password or certificate check against Active Directory).
- **Authorization**: Determining what they are allowed to do (based on policies like group membership, time of day).
- **Accounting**: Keeping a record of when they logged in, logged out, and how much data they used (saved in local log files or SQL Database).

### 2. RADIUS Client vs. RADIUS Server vs. RADIUS Proxy
- **RADIUS Client (Network Access Server - NAS)**: The physical or virtual network device (like a Cisco switch, Ruckus Wi-Fi Access Point, or Windows RRAS Server) that forwards connection requests to the NPS.
- **RADIUS Server**: The NPS itself, which processes authentication requests locally against Active Directory.
- **RADIUS Proxy**: NPS configured to forward authentication requests to another RADIUS server (useful in multi-forest or external vendor scenarios).

### 3. NPS Policies: Connection Request vs. Network Policies
NPS evaluates policies in a specific sequence:
1. **Connection Request Policies (CRP)**: Determine *where* the authentication happens. It asks: "Should I authenticate this request locally, or should I forward it to another RADIUS server (Proxy)?"
2. **Network Policies**: Determine *who* gets access and under what *conditions*. NPS evaluates these in order of priority. The first policy that matches the client's conditions is applied, and the access is either **Granted** or **Denied**.

### 4. Authentication Methods (PEAP vs. EAP-TLS)
- **PEAP-MSCHAPv2**: Requires a certificate only on the NPS server. Users authenticate using their standard AD username and password. High convenience, medium security.
- **EAP-TLS**: Requires certificates on both the NPS server and the client machine/user. Extremely secure, zero passwords exchanged, but requires a functional PKI (AD CS).

---
## Step-by-Step Lab / Configuration

> [!warning] Pre-requisites
> - Active Directory Domain Controller (`DC-01.corp.local` with IP `192.168.10.10`).
> - A member server (`SVR-NPS01` with IP `192.168.10.15`) joined to the domain.
> - Administrative credentials (Domain Admin).

### Step 1: Install NPAS Role and Register in AD
1. Log in to `SVR-NPS01`. Open PowerShell as Administrator and run:
```powershell
Install-WindowsFeature -Name NPAS -IncludeManagementTools
```
**Expected Output:**
```text
Success Restart Needed Exit Code Feature Result
------- -------------- --------- --------------
True    No             Success   {Network Policy and Access Services...}
```

2. Open NPS Console (`nps.msc`).
3. Right-click **NPS (Local)** and select **Register server in Active Directory**.
4. Click **OK** to confirm. This adds the NPS server to the `RAS and IAS Servers` security group in Active Directory, allowing it to read user account properties.

### Step 2: Configure a RADIUS Client (e.g., Corporate Access Point)
1. In the NPS console, expand **RADIUS Clients and Servers** and right-click **RADIUS Clients** -> **New**.
2. Configure the client:
   - Friendly name: `Corp-AP-01`
   - Address (IP or DNS): `192.168.10.254`
   - Shared secret (Manual): `SuperSecureRadiusSecret123!`
   - Confirm shared secret: `SuperSecureRadiusSecret123!`
3. Click **OK**.

### Step 3: Configure Network Policy for Wi-Fi Access
1. Expand **Policies** and right-click **Network Policies** -> **New**.
2. Policy Name: `Allow-Corp-WiFi`. Network Access Server Type: `Unspecified`. Click Next.
3. Conditions: Click **Add**.
   - Select **User Groups** -> Add `corp\G_WiFi_Users`.
   - Select **NAS Port Type** -> Add `Wireless - IEEE 802.11` and `Wireless - Other`.
   - Click Next.
4. Access Permission: Select **Access granted**. Click Next.
5. Authentication Methods:
   - Click **Add** -> Select **Microsoft: Protected EAP (PEAP)** -> Click OK.
   - Select PEAP and click **Edit**. Under Certificate Issued, ensure your Server Certificate (from AD CS) is selected.
   - Under EAP Types, ensure **Secured password (EAP-MSCHAP v2)** is present.
   - Click Next.
6. Constraints: Leave defaults and click Next.
7. Configure Settings: Leave defaults and click Next.
8. Click **Finish**. Ensure this policy is moved to the top of the processing list.

### Step 4: Verify Local Accounting Logs
1. Click on **NPS (Local)** -> **Accounting** -> **Configure Accounting**.
2. Choose **Log to a local file**.
3. Select log file directory: `C:\Windows\System32\LogFiles`.
4. Ensure **Log SQL-compatible format** is checked to track user logon details.

---
## Cheat Sheet

| Command / Setting | Purpose | Example |
|---|---|---|
| `Install-WindowsFeature -Name NPAS -IncludeManagementTools` | Installs NPS role and console | `Install-WindowsFeature -Name NPAS -IncludeManagementTools` |
| `Get-NpsRadiusClient` | Lists all configured RADIUS clients | `Get-NpsRadiusClient` |
| `New-NpsRadiusClient` | Adds a new RADIUS client via PowerShell | `New-NpsRadiusClient -Name "AP-Floor1" -Address "192.168.10.50" -SharedSecret "P@ss123"` |
| `Set-NpsRadiusClient` | Updates details of an existing RADIUS client | `Set-NpsRadiusClient -Name "AP-Floor1" -SharedSecret "NewSecret456!"` |
| `netsh nps export filename="C:\npsconfig.xml" exportPrivateConfiguration=yes` | Exports entire NPS configuration including shared secrets | `netsh nps export filename="C:\npsconfig.xml" exportPrivateConfiguration=yes` |
| `netsh nps import filename="C:\npsconfig.xml"` | Imports NPS configuration from XML file | `netsh nps import filename="C:\npsconfig.xml"` |
| `RADIUS Port 1812 (UDP)` | Port used for authentication and authorization | Set in Firewall Rules / Router settings |
| `RADIUS Port 1813 (UDP)` | Port used for RADIUS Accounting | Set in Firewall Rules / Router settings |

---
## Troubleshooting

| Problem | Cause | Fix |
|---|---|---|
| Connection fails with error: Shared Secret mismatch or invalid credentials. | The Shared Secret key configured on the RADIUS client (Switch/AP) does not match the Shared Secret in NPS. | Re-enter the exact same secret key on both the NPS RADIUS Client properties and the network switch/AP configuration. Avoid copying trailing spaces. |
| NPS does not process requests, Event log shows "NPS is not authorized to read user dial-in properties". | NPS server has not been registered in Active Directory. | Right-click **NPS (Local)** in the console and click **Register server in Active Directory** (adds server to `RAS and IAS Servers` group). Restart the NPS service. |
| Clients fail to connect, Event Viewer shows Event ID 6273 with Reason Code 22 (The client certificate is not trusted). | The client machine does not trust the Root CA certificate that issued the NPS server's certificate. | Distribute the Active Directory Root CA certificate to all client machines using a Group Policy Object (GPO) in the Trusted Root Certification Authorities store. |
| RADIUS Access-Request timeout on client, but NPS is running. | Perimeter or local Windows Defender Firewall is blocking UDP ports 1812 and 1813. | Open Windows Defender Firewall and ensure the inbound rules for "Network Policy Server (RADIUS-In)" (UDP ports 1812, 1813, 1645, 1646) are enabled. |
| User is rejected despite typing correct credentials, Event ID 6273 shows "Reason Code 66: The user's account dial-in properties deny access". | User dial-in permission in Active Directory (ADUC) is set to "Deny access" or "Control access through NPS Policy" but no matching grant policy is found. | Go to ADUC -> user properties -> Dial-in tab. Change access to "Allow Access" or ensure the NPS Network Policy is configured to explicitly match and grant access. |

---
## Real-World Ticket Scenarios

### Scenario 1: Corporate Wi-Fi Authentication Loops for all users (Certificate Expired)
**Ticket:** "Users reporting they cannot connect to Corp_WiFi. Laptops are prompting for username and password repeatedly, even after entering correct credentials."
**L1 Resolution:** Verify user account status in Active Directory. Check if this is happening on multiple devices. Gather client-side Windows WLAN logs (`netsh wlan show interfaces` and Event Viewer `WLAN-AutoConfig` logs).
**Escalation Trigger:** The issue occurs on all client devices, and the event logs point to an untrusted or invalid server security certificate.
**L2 Resolution:** Remote into the NPS server `SVR-NPS01`. Open NPS Console (`nps.msc`) -> Policies -> Network Policies. Right-click the active wireless policy and select Properties. Go to Constraints -> Authentication Methods. Select Protected EAP (PEAP) and click Edit. Check if the SSL certificate bound to PEAP is expired or invalid. If expired, select the active, renewed certificate issued by the Active Directory Certificate Services (AD CS) Root CA, click Apply, and restart the NPS service.

---

### Scenario 2: VPN Client Connection Fails with Access Denied (AD Group Membership Missing)
**Ticket:** "A new user, Alice, cannot connect to the company VPN. She gets an Access Denied error (Error 812: The connection was prevented due to a policy configured on your RAS/VPN server)."
**L1 Resolution:** Check if Alice's user account is enabled and not locked out in Active Directory. Verify if other users are connecting successfully.
**Escalation Trigger:** Alice's account is active, but the connection continues to fail, indicating an NPS policy restriction.
**L2 Resolution:** Open the Active Directory Users and Computers (ADUC) console. Check Alice's group membership. Verify if she is a member of the security group `G_VPN_Users` configured in the NPS Network Policy conditions. If missing, add her to the group. If she is in the group, open NPS console, inspect Event Viewer -> Custom Views -> Server Roles -> Network Policy and Access Services. Look for Event ID 6273 (Audit Failure) for Alice. Verify if the matched policy or the user account dial-in tab properties is configured to deny access. Resolve by updating the AD group membership and validating compliance.

---
## Interview Questions

> [!question] L1 Question
> **Q:** What is the primary function of RADIUS in an enterprise network, and which UDP ports does it use?
> **A:** RADIUS (Remote Authentication Dial-In User Service) is a network protocol that centralizes AAA (Authentication, Authorization, and Accounting) management for users connecting to a network. Instead of managing logins locally on dozens of switches or Access Points, they query a central RADIUS server (like NPS). RADIUS uses **UDP Port 1812** for Authentication/Authorization and **UDP Port 1813** for Accounting. (Note: Legacy implementations used UDP 1645 and 1646).

> [!question] L2 Question
> **Q:** Explain the difference between Connection Request Policies and Network Policies in NPS. How are they evaluated?
> **A:** **Connection Request Policies (CRP)** are evaluated first. They determine *where* the request should be authenticated (either locally on the NPS server itself, or forwarded to a remote RADIUS server group as a proxy). **Network Policies** are evaluated second, only if the CRP decides to process the request locally. Network Policies determine *who* is allowed to access the network. They evaluate conditions such as Active Directory group membership, allowed authentication protocols (e.g., PEAP or EAP-TLS), and device type (e.g., Wireless 802.11) to decide whether to Grant or Deny the connection.

> [!question] L3/Scenario Question
> **Q:** You are troubleshooting a Wi-Fi connection failure where corporate laptops cannot connect via 802.1X (PEAP). In the NPS Server Event Viewer, you find Event ID 6273 with failure reason: "The certificate chain was issued by an authority that is not trusted." What is the root cause, and how would you resolve it?
> **A:** This error occurs because the client device does not trust the Certificate Authority (CA) that issued the NPS server's certificate used for PEAP encryption. To resolve this:
> 1. Verify that the NPS server has a valid computer certificate installed from the enterprise CA.
> 2. Ensure that the root CA certificate is installed in the "Trusted Root Certification Authorities" certificate store on the client laptops.
> 3. Use Group Policy (GPO) under `Computer Configuration > Policies > Windows Settings > Security Settings > Public Key Policies` to automatically distribute the Enterprise Root CA certificate to all domain-joined laptops.
> 4. Ensure that the wireless network profile deployed to clients is configured to validate the server certificate and trusts the specific Root CA.

---

---
## Real-World Scenario: File Server Crash (500+ Users Affected)
Agar company ka main file server (500+ users connected) crash ho jaye, toh as a Senior Sysadmin aapka immediate action plan yeh hoga:
1. **Identify & Isolate**: Check if the server is pingable. Verify hardware status via Dell iDRAC / HPE iLO or check virtual machine status in vCenter/Hyper-V.
2. **Access Check & Lock**: If the storage volume is corrupt or unmapped, stop DFS Namespace referrals to the crashed target to redirect users to secondary replica servers if active.
3. **Trigger Disaster Recovery (DR)**: Start restoring the crashed system drive or volumes from the latest VSS snapshot or Azure Backup RSV restore point.
4. **Communication**: Broadcast status updates to the Incident Commander and helpdesk teams to manage incoming support volume.

### PowerShell Automation Snippet: Verify File Shares and Access Permissions
```powershell
# Get all active file shares on the server
Get-SmbShare | Format-Table -AutoSize

# Test local DFS Namespace server connection health
Test-DFSNamespaceTarget -Path "\\corp.local\DFSRoot\Public"
```

---
## Related Notes
- [[03-Identity-and-Core-Services/05-Windows-Server/WS-14 VPN and Remote Access (RRAS)|WS-14 VPN and Remote Access (RRAS)]] — Integrates RRAS as a RADIUS client to NPS.
- [[03-Identity-and-Core-Services/06-Active-Directory/WS-13 Active Directory Certificate Services|WS-13 Active Directory Certificate Services]] — Issues server certificates required for secure PEAP/TLS RADIUS authentication.
- [[03-Identity-and-Core-Services/06-Active-Directory/WS-06 Active Directory — Users Groups OUs|WS-06 Active Directory — Users Groups OUs]] — Creating security groups used as conditions in NPS policies.

---
*Tags: #sysadmin #windows-server #nps #radius #security #cert-none*
