---
tags: [windows-server, nps, radius, networking, security]
aliases: [NPS, RADIUS Server, Network Policy Server]
created: 2026-06-26
status: #complete
difficulty: #intermediate
cert-relevant: #none
---

> [!NOTE|color-blue]
> 🏢 **WINDOWS SERVER**

`#complete` `#intermediate` `#none`

# WS-17: NPS and RADIUS Server

> [!abstract] Overview
> Yeh note Network Policy Server (NPS) aur RADIUS protocol ke baare mein hai. NPS Windows Server mein Microsoft ka RADIUS server implementation hai jo central authentication, authorization, aur accounting (AAA) provide karta hai network access ke liye. Ek system administrator ke liye yeh samajhna zaroori hai kyunki VPN, Wi-Fi, aur wired switches pe secure access manage karne ke liye NPS use hota hai. Agar NPS fail ho jaye, toh naye users network se connect nahi kar payenge.

---
## 🧠 Concept Overview

- **What it is** — NPS (Network Policy Server) is a role in Windows Server that acts as a RADIUS (Remote Authentication Dial-In User Service) server and proxy. RADIUS is a networking protocol that provides centralized Authentication, Authorization, and Accounting (AAA) management.
- **Why it matters** — Iske bina, agar aapke paas 50 Wi-Fi access points aur 20 VPN servers hain, toh sabpe alag alag users create karne padenge. NPS ek central point ban jaata hai jahan se saare devices Active Directory credentials use karke users ko verify kar sakte hain. *Real job mein central management aur security ke liye yeh vital hai.*
- **Where you see this** — Corporate Wi-Fi (802.1X), VPN connections, Network switches, aur DirectAccess setups mein jahan users ko securely authenticate karna hota hai. IT environments mein BYOD (Bring Your Own Device) manage karne ke liye bhi NPS use hota hai.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic task — Check karna agar user network (Wi-Fi/VPN) se connect ho pa raha hai ya nahi. AD mein account lock toh nahi hai dekhna. Event Viewer mein basic errors check karna. |
| **L2** | Configure, fix — NPS server ke logs check karna, network policies ko adjust karna, aur failed authentications ka root cause find out karna. Naye RADIUS clients (switches/APs) add karna. |
| **L3** | Architecture, design — Multi-site NPS architecture design karna, RADIUS proxies set up karna, certificates (PKI) deploy karna secure EAP-TLS ke liye. High availability aur load balancing set karna RADIUS servers ka. |

> [!tip] Seedha Simple Mein
> *NPS ek security guard ki tarah hai jo ek register (Active Directory) leke baitha hai. Jab bhi koi Wi-Fi ya VPN se connect karne ki koshish karta hai, network device (jaise router ya access point) guard (NPS) se poochta hai ki "kya isko entry doon?". Guard register check karke "yes" ya "no" bolta hai, aur conditions dekhta hai (e.g. kya iske paas VPN_Users group ka pass hai?).*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **A VIP Nightclub Entry System** is like **NPS/RADIUS Server** because...
>
> - **The Bouncer (Network Device - AP/Switch):** Gate pe khada bouncer kisi ko pehchanta nahi hai, uske paas bas walkie-talkie hai. Wo apna dimaag nahi lagata, sirf instructions follow karta hai.
> - **The Manager inside with the VIP List (NPS Server with AD):** Bouncer walkie-talkie (RADIUS protocol) se andar manager ko naam batata hai. Manager Active directory ka use karta hai.
> - **The VIP List (Active Directory):** Manager list check karta hai ki user allowed hai ya nahi, kis area (authorization - like VIP lounge vs General) mein ja sakta hai, aur kab entry li (accounting). Bouncer bas manager ke YES ya NO ka wait karta hai.
> - **Shared Secret:** Manager aur bouncer ke beech ka ek secret code, taaki koi random aadmi manager ko nakli message na bhej sake.

---
## 🔬 Technical Deep Dive

### 1. The AAA Framework

> [!info] Key Concept
> AAA stands for Authentication, Authorization, and Accounting. It's the core of how RADIUS operates over UDP ports (usually 1812 and 1813).

- **Authentication:** *Yeh prove karna ki aap kaun hain.* (e.g., User ID aur password, ya certificate check karna). Client claims to be 'User1', server verifies this identity.
- **Authorization:** *Aap network pe kya kar sakte hain aur kis resources ka access hai.* (e.g., VLAN assign karna, VPN access allow/deny karna AD group ke basis pe). If 'User1' is authenticated, are they allowed to access the VPN?
- **Accounting:** *Aapne kab log in kiya, kitni der connected rahe, kitna data use kiya.* (Logs jo compliance, billing, aur auditing ke liye use hote hain). NPS can log these directly to a local text file or a SQL Server database.

### 2. NPS Components in Windows Server

- **RADIUS Clients:** Yeh actual network devices (like Cisco Switches, Aruba Wi-Fi Access Points, VPN gateways) hote hain jo NPS ko requests bhejte hain. *Client ka matlab yahan user ka laptop ya phone nahi, balki networking hardware hai.*
- **Connection Request Policies (CRP):** Yeh rules decide karte hain ki kaunsa RADIUS server is request ko process karega. In a large enterprise, ek NPS server local requests khud process kar sakta hai aur doosri requests ko kisi aur branch ke server pe forward (proxy) kar sakta hai.
- **Network Policies:** Agar CRP decide karta hai ki request local server process karega, tab Network Policy hit hoti hai. Yeh rules decide karte hain ki user ko access milna chahiye ya nahi, based on conditions like Windows Groups (e.g., `Domain Admins`), Time of day, ya authentication type (EAP, PEAP).

### 3. EAP and PEAP (Authentication Methods)

> [!info] Key Concept
> EAP (Extensible Authentication Protocol) provides the framework for multiple authentication methods. EAP types determine exactly HOW the authentication happens securely over an insecure network.

- **EAP-TLS:** Sabse secure method. User (client device) aur server dono ke paas digital certificates hone chahiye. *Isme username/password ki zarurat nahi hoti, sirf certificate.* Extremely secure but complex to set up because you need a PKI (Public Key Infrastructure).
- **PEAP-MSCHAPv2:** Protected EAP. Most commonly used for corporate Wi-Fi. Isme server ke paas certificate hota hai (apni identity prove karne ke liye taaki user nakli AP se na connect ho), aur user apna AD username/password enter karta hai. Encryption tunnel banne ke baad password check hota hai.

> [!danger] Common Mistake
> RADIUS clients ko NPS mein add karte time **Shared Secret** match hona bohot zaroori hai. Agar switch/AP aur NPS server pe shared secret mein ek bhi character alag hoga, toh communication silently fail ho jayega aur RADIUS client invalid mana jayega.

### 4. Accounting Options in NPS

Accounting is crucial for tracking who accessed what and when.
- **Local File Logging:** Logs text format mein `C:\Windows\System32\LogFiles` mein save hote hain. Yeh troubleshooting ke liye best hai.
- **SQL Server Logging:** Large enterprises mein jahan multiple NPS servers hain, saara data central SQL database mein bheja jaata hai for rich reporting and auditing.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Windows Server 2019/2022 with Active Directory Domain Services installed.
> - Domain Admin credentials.
> - A test client machine or a network switch/AP.

### Step 1: Install the NPS Role

Aap Server Manager se "Network Policy and Access Services" check kar sakte hain, ya PowerShell use karein.

```powershell
# Yeh command Server Manager ke bina NPS role install karti hai
Install-WindowsFeature NPAS -IncludeManagementTools
```

> [!success] Expected Output
> ```
> Success Restart Needed Exit Code      Feature Result
> ------- -------------- ---------      --------------
> True    No             Success        {Network Policy and Access Services...
> ```

### Step 2: Register NPS Server in Active Directory

Active Directory ko pata hona chahiye ki yeh server users ke passwords aur dial-in properties read kar sakta hai. *Bina register kiye, NPS ko AD mein user check karne ki permission nahi milegi.*

```powershell
# Register the NPS Server in AD
netsh nps add registeredserver
```

### Step 3: Add a RADIUS Client (e.g., Wi-Fi Access Point)

Humein NPS ko batana hoga ki kis IP se requests aayengi. Naye devices ko allow karne ke liye unhe as RADIUS clients add karein.

```powershell
# Add a RADIUS client with a shared secret
New-NpsRadiusClient -Name "HQ-WiFi-AP1" -Address "192.168.10.50" -SharedSecret "MyS3cr3tKey123!"
```

### Step 4: Create a Network Policy for VPN/Wi-Fi Users

UI method (`nps.msc`) is highly recommended because the wizard simplifies complex configurations.

1. Run `nps.msc` from Run prompt.
2. Under "Standard Configuration" choose "RADIUS server for 802.1X Wireless or Wired Connections".
3. Click "Configure 802.1X".
4. Add your RADIUS clients (if not already done).
5. Choose an Authentication Method (e.g., Microsoft: Protected EAP (PEAP)). *Note: Iske liye ek server certificate chahiye.*
6. Specify User Groups (e.g., `CORP\Wireless_Users`).
7. Complete wizard. Ab `Wireless_Users` group ke log AP ke through connect kar payenge.

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `Install-WindowsFeature NPAS` | Install NPS Role on Windows Server | `Install-WindowsFeature NPAS -IncludeManagementTools` |
| `nps.msc` | Opens the Network Policy Server GUI console | `nps.msc` |
| `netsh nps add registeredserver` | AD mein NPS ko register karta hai taaki wo user properties read kar sake | `netsh nps add registeredserver` |
| `New-NpsRadiusClient` | Naya RADIUS client (Switch/AP) add karta hai | `New-NpsRadiusClient -Name "Switch1" -Address "10.0.0.5" -SharedSecret "Key"` |
| `Get-NpsRadiusClient` | List all configured RADIUS clients on the server | `Get-NpsRadiusClient` |
| `Get-NpsNetworkPolicy` | Server ki saari Network Policies list karta hai | `Get-NpsNetworkPolicy` |
| `Export-NpsConfiguration` | NPS ki poori configuration ka XML backup lene ke liye | `Export-NpsConfiguration -Path "C:\NpsBackup.xml"` |
| `Import-NpsConfiguration` | XML backup se config restore karne ke liye | `Import-NpsConfiguration -Path "C:\NpsBackup.xml"` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| Users cannot connect to Wi-Fi/VPN | NPS server is not registered in AD | Run `netsh nps add registeredserver` ya GUI se server name pe right click karke "Register server in AD" karein. |
| Event ID 13 or 18 (Access Denied) | Wrong shared secret between NPS and AP | Dono taraf (Switch/AP aur NPS Client config) mein Shared Secret exactly re-enter aur verify karein. |
| Event ID 22 (Client not configured) | NPS doesn't recognize the IP sending the request | Verify the IP address of the switch/AP in NPS under RADIUS Clients. Ensure firewall isn't NATting the IP. |
| Authentication fails, Event ID 6273 | User does not meet Network Policy conditions | User ko sahi AD group (e.g., VPN Users) mein add karein, ya policy ki conditions (time of day, valid groups) verify karein. |
| Certificate Error on Client Device | Server certificate expired or untrusted | NPS server ka certificate renew karein aur client ke Trusted Root CAs mein check karein. Policy mein naya cert select karein. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: User Cannot Connect to Corporate Wi-Fi After Password Change

> [!example] Ticket
> "I changed my Windows password this morning and now my phone and laptop won't connect to the 'Corp-Secure' Wi-Fi network."

**L1 Response:** *Pehle check karein agar user AD mein locked out toh nahi hai naye password aur phone mein save purane cached credentials conflict ki wajah se.*
**Escalation Trigger:** Agar account unlocked hai aur AD mein credentials sahi hain, par sirf is user ko issue aa raha hai aur baaki users perfectly connect ho rahe hain, further logs check karne ke liye ticket L2 ko bhejein.
**L2 Resolution:** NPS server ke Event Viewer (Custom Views -> Server Roles -> Network Policy and Access Services) mein check kiya. Agar error Reason Code 16 (Authentication failed due to user credentials mismatch) hai, toh user ko bataya gaya ki apne device pe Wi-Fi network ko "Forget" karein aur dobara naya password daal ke connect karein.

### 🎫 Scenario 2: New Branch Office Switch Cannot Authenticate Any Users

> [!example] Ticket
> "Network team just installed a new 48-port switch at the London Branch office, but nobody connecting to it can authenticate their PCs via 802.1X."

**L1 Response:** *Confirm karein if it's a single user issue or global at the branch. Since it's global on a new switch, it's an infrastructure configuration issue.*
**Escalation Trigger:** Pass to L2/L3 immediately as it involves adding infrastructure devices to the NPS server configuration.
**L2 Resolution:** Check NPS Event Viewer logs for Event ID 22 or 13. Pata chala ki naye switch ka IP address (10.50.1.5) NPS mein as a "RADIUS Client" add hi nahi kiya gaya tha. `New-NpsRadiusClient` use karke naya switch configure kiya aur shared secret network team ko diya. Uske baad switch se sabhi PC authenticate hone lage.

### 🎫 Scenario 3: Certificate Expiry Causing Global Outage

> [!example] Ticket
> "MASSIVE OUTAGE: No one can connect to the AnyConnect VPN or Corporate Wi-Fi. Everyone is getting an error saying certificate validation failed."

**L1 Response:** *Agar sabhi ko ek saath same error aaye, this is a P1/Sev 1 incident.* L1 turant major incident bridge call trigger karta hai aur Server Admin on-call ko page karta hai.
**Escalation Trigger:** Directly escalate to L3 / Server Admin team.
**L3 Resolution:** NPS uses a digital certificate to prove its identity during PEAP/EAP-TLS authentication. L3 admin ne server pe check kiya, certificate ki validity kal raat expire ho gayi thi. Internal PKI/CA server se naya certificate issue karwaya, uske baad NPS console (nps.msc) mein jaakar Network Policies ko edit kiya aur naya valid certificate drop-down se select kiya. NPS service restart ki aur connection restore ho gaya.

---
## 🎤 Interview Questions

> [!question] Q1: What is the primary role of NPS in a Windows Server environment?
> **Answer:** NPS (Network Policy Server) acts as a RADIUS server and proxy. It provides centralized Authentication, Authorization, and Accounting (AAA) for network access devices like VPN servers, wireless access points, and 802.1X capable switches by validating credentials against Active Directory.

> [!question] Q2: Why must an NPS server be registered in Active Directory?
> **Answer:** *AD mein register hona zaroori hai taaki* the NPS server gets the necessary permissions to read user accounts' dial-in properties and group memberships. Without this registration, it cannot authorize users properly based on AD data.

> [!question] Q3: What is a RADIUS Client in the context of NPS? Is it the end-user's laptop?
> **Answer:** No, the RADIUS client is NOT the end-user's device (laptop or phone). The RADIUS client is the network access server (NAS) — such as a wireless access point, VPN gateway, or a managed network switch — that forwards the user's authentication request to the NPS server.

> [!question] Q4: What is the difference between Authentication and Authorization in RADIUS?
> **Answer:** Authentication is verifying *who the user is* (e.g., checking password or certificate validity). Authorization is determining *what the user is allowed to do or access* (e.g., assigning a specific VLAN, or allowing VPN dial-in based on AD group membership after successful authentication).

> [!question] Q5: Which protocol and port numbers does RADIUS commonly use?
> **Answer:** RADIUS uses the **UDP** protocol. Default ports are **1812** for Authentication and Authorization, and **1813** for Accounting. (Historically, UDP ports 1645 and 1646 were also used and you may still see them in older legacy devices).

> [!question] Q6: What is a Shared Secret in NPS?
> **Answer:** It's a text string that serves as a password between the RADIUS client (like a Wi-Fi AP) and the RADIUS Server (NPS). *Dono side match hona chahiye taaki inke beech ka communication secure rahe.*

==**Exam Tip:** Hamesha yaad rakhna ki RADIUS Client = Network Device (Switch/AP), NOT the user's PC. Aur Shared Secret mismatch is the #1 cause of silent authentication failures.==

---
## 🔗 Related Notes

- [[WS-01 Active Directory Domain Services]] — The backend database NPS uses for authentication.
- [[WS-16 Certificate Authority (AD CS)]] — Used to issue the certificates required for PEAP and EAP-TLS.
- [[VPN Concepts]] — VPN gateways often use NPS for authenticating dial-in users.
- [[Network Access Control (NAC)]] — NPS plays a crucial role in NAC architectures.
