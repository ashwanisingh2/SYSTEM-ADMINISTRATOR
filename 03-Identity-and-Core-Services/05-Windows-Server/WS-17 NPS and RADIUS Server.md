---
tags: [sysadmin, windows-server, nps, radius, security]
aliases: [nps-radius]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #none
---

> [!NOTE|color-blue]
> 🏢 **WINDOWS SERVER**

`#complete` `#intermediate` `#none`

# WS-17: NPS and RADIUS Server

> [!abstract] Overview
> Yeh note Network Policy Server (NPS) in Windows Server 2022 cover karta hai. Ek support engineer ke liye yeh janna bahut zaroori hai kyunki yeh RADIUS server/proxy ki tarah kaam karta hai aur enterprise networks (Wi-Fi, VPN, switches) ke liye Authentication, Authorization, aur Accounting (AAA) centralize karta hai.

---
## 🧠 Concept Overview

- **What it is** — NPS Microsoft ka RADIUS (Remote Authentication Dial-In User Service) server aur proxy hai. Yeh network access ke liye ek central gatekeeper ki tarah kaam karta hai.
- **Why it matters** — Enterprise networks mein har switch ya VPN par user credentials configure karna mushkil hai. NPS ise centralize karta hai taaki koi bhi device connect hone se pehle Active Directory se permission le.
- **Where you see this** — Corporate Wi-Fi authentication (802.1X), network switches (switch port security) ko secure karne, ya remote access VPN authentication control karne mein.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Check basic user network connectivity, verify AD accounts are not locked out, check group membership, aur client-side Wi-Fi/VPN error messages gather karna. |
| **L2** | Configure RADIUS Clients and Shared Secrets in NPS, inspect NPS logs (Event Viewer / IAS logs), aur test connection request policies. |
| **L3** | Design connection request policies and network policies, manage enterprise CA certificates for PEAP/EAP-TLS, aur RADIUS proxies configure karna. |

> [!tip] Seedha Simple Mein
> *NPS ek central security guard ki tarah hai. Jab bhi koi user corporate Wi-Fi, VPN, ya switch se connect hone ki koshish karta hai, toh woh device (RADIUS Client) user ke credentials check karne ke liye NPS ke paas bhejta hai. NPS Active Directory se pooch kar permission grant ya deny karta hai.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **NPS** is like **a nightclub bouncer** because...
>
> - The bouncer doesn't own the club, but checks the guest list (Active Directory).
> - You show your ID to the bouncer (Authentication), they check if you are VIP or regular (Authorization), and write down what time you entered (Accounting).

---
## 🔬 Technical Deep Dive

### 1. RADIUS (AAA) Architecture

> [!info] Key Concept
> RADIUS operates on the AAA model: Authentication, Authorization, and Accounting.

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

> [!danger] Common Mistake
> Configuring a different Shared Secret on the RADIUS client and the NPS server. Mismatched shared secrets are the #1 cause of RADIUS authentication failures!

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Active Directory Domain Controller (`DC-01.corp.local` with IP `192.168.10.10`).
> - A member server (`SVR-NPS01` with IP `192.168.10.15`) joined to the domain.
> - Administrative credentials (Domain Admin).

### Step 1: Install NPAS Role and Register in AD

```powershell
# Install NPS role and management tools
Install-WindowsFeature -Name NPAS -IncludeManagementTools
```

> [!success] Expected Output
> ```
> Success Restart Needed Exit Code Feature Result
> ------- -------------- --------- --------------
> True    No             Success   {Network Policy and Access Services...}
> ```

1. Open NPS Console (`nps.msc`).
2. Right-click **NPS (Local)** and select **Register server in Active Directory**.
3. Click **OK** to confirm. This adds the NPS server to the `RAS and IAS Servers` security group in Active Directory.

### Step 2: Configure a RADIUS Client

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
   - Select PEAP and click **Edit**. Ensure your Server Certificate (from AD CS) is selected.
   - Click Next.
6. Click **Finish**. Ensure this policy is moved to the top of the processing list.

### Step 4: Verify Local Accounting Logs

1. Click on **NPS (Local)** -> **Accounting** -> **Configure Accounting**.
2. Choose **Log to a local file**.
3. Select log file directory: `C:\Windows\System32\LogFiles`.
4. Ensure **Log SQL-compatible format** is checked.

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `Install-WindowsFeature -Name NPAS` | Installs NPS role | `Install-WindowsFeature -Name NPAS -IncludeManagementTools` |
| `Get-NpsRadiusClient` | Lists all RADIUS clients | `Get-NpsRadiusClient` |
| `New-NpsRadiusClient` | Adds a new RADIUS client | `New-NpsRadiusClient -Name "AP1" -Address "192.168.10.50" -SharedSecret "P@ss123"` |
| `Set-NpsRadiusClient` | Updates RADIUS client details | `Set-NpsRadiusClient -Name "AP1" -SharedSecret "NewSecret!"` |
| `netsh nps export` | Exports NPS configuration | `netsh nps export filename="C:\npsconfig.xml" exportPrivateConfiguration=yes` |
| `netsh nps import` | Imports NPS configuration | `netsh nps import filename="C:\npsconfig.xml"` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| Connection fails with invalid credentials | Shared Secret mismatch between client and NPS | Re-enter the exact same secret key on both the NPS RADIUS Client and the switch/AP. |
| Event log: "NPS is not authorized to read user dial-in properties" | NPS server is not registered in Active Directory | Right-click NPS (Local) -> Register server in Active Directory, then restart service. |
| Event ID 6273, Reason 22 (Client certificate not trusted) | Client does not trust the Root CA certificate | Distribute AD Root CA certificate to all clients using a GPO. |
| RADIUS Access-Request timeout | Firewall blocking UDP ports 1812/1813 | Enable inbound rules for "Network Policy Server (RADIUS-In)" (UDP ports 1812, 1813). |
| Event ID 6273, Reason 66 (User dial-in properties deny access) | User dial-in permission is set to "Deny access" or no policy matches | Go to ADUC -> user properties -> Dial-in tab. Change to "Allow Access" or fix NPS Network Policy. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Corporate Wi-Fi Authentication Loops

> [!example] Ticket
> "Users reporting they cannot connect to Corp_WiFi. Laptops are prompting for username and password repeatedly, even after entering correct credentials."

**L1 Response:** Verify user account status in AD. Check if this happens on multiple devices. Gather client-side Windows WLAN logs.
**Escalation Trigger:** The issue occurs on all client devices, pointing to an untrusted or invalid server security certificate.
**L2 Resolution:** Remote into `SVR-NPS01`. Open NPS Console -> Policies -> Network Policies. Right-click the active wireless policy -> Properties. Go to Constraints -> Authentication Methods. Select PEAP and click Edit. Check if the SSL certificate is expired. If expired, select the active, renewed certificate, apply, and restart the NPS service.

### 🎫 Scenario 2: VPN Client Connection Fails with Access Denied

> [!example] Ticket
> "A new user, Alice, cannot connect to the company VPN. She gets an Access Denied error (Error 812)."

**L1 Response:** Check if Alice's user account is enabled and not locked out. Verify if other users are connecting successfully.
**Escalation Trigger:** Alice's account is active, but connection fails, indicating an NPS policy restriction.
**L2 Resolution:** Open ADUC. Check Alice's group membership. Verify if she is a member of `G_VPN_Users` configured in the NPS Network Policy. If missing, add her. If she is there, check NPS Event Viewer for Event ID 6273 to see which policy denied her.

---
## 🎤 Interview Questions

> [!question] Q1: What is the primary function of RADIUS, and which UDP ports does it use?
> **Answer:** RADIUS centralizes AAA management for network access. It uses **UDP Port 1812** for Authentication/Authorization and **UDP Port 1813** for Accounting.

==**Exam Tip:** Port 1812 and 1813 are crucial to remember for RADIUS authentication and accounting.==

> [!question] Q2: Explain the difference between Connection Request Policies and Network Policies in NPS.
> **Answer:** **Connection Request Policies (CRP)** determine *where* the request is authenticated (locally or forwarded to a proxy). **Network Policies** determine *who* gets access based on conditions like AD group membership, and are evaluated only if CRP decides to authenticate locally.

> [!question] Q3: You see Event ID 6273 with "The certificate chain was issued by an authority that is not trusted." Root cause?
> **Answer:** The client device does not trust the Certificate Authority that issued the NPS server's certificate. Resolve by distributing the Enterprise Root CA certificate to client laptops via GPO.

---
## 🔗 Related Notes

- [[03-Identity-and-Core-Services/05-Windows-Server/WS-14 VPN and Remote Access (RRAS)|WS-14 VPN and Remote Access (RRAS)]] — Integrates RRAS as a RADIUS client to NPS.
- [[03-Identity-and-Core-Services/06-Active-Directory/WS-13 Active Directory Certificate Services|WS-13 Active Directory Certificate Services]] — Issues server certificates required for secure PEAP/TLS RADIUS authentication.
- [[03-Identity-and-Core-Services/06-Active-Directory/WS-06 Active Directory — Users Groups OUs|WS-06 Active Directory — Users Groups OUs]] — Creating security groups used as conditions in NPS policies.
