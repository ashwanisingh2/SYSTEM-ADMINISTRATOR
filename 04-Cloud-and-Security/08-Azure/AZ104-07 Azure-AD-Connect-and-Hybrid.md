---
tags: [desktop-support, azure, identity, hybrid, L2]
aliases: [entra-connect-hybrid, hybrid-identity]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #az-104
---

> [!NOTE|color-blue]
> ☁️ **AZURE CLOUD**

`#complete` `#advanced` `#az-104`

# AZ104-07: Azure AD Connect and Hybrid Identity

> [!abstract] Overview
> Yeh note Microsoft Azure hybrid identity architecture par focus karta hai. Isme Entra Connect Sync vs Cloud Sync, Hybrid Microsoft Entra Join, Seamless SSO aur synchronization troubleshooting detail mein cover kiye gaye hain. Ek system administrator ko hybrid logins aur device sync issue fix karne mein yeh madad karega.

---
## 🧠 Concept Overview

- **What it is** — On-premises Active Directory ko cloud Microsoft Entra ID se connect karke synchronize karne ka process, taaki ek hi identity dono jagah chale.
- **Why it matters** — Users ko on-prem file servers aur M365 apps dono ke liye alag alag passwords yaad rakhne ki zaroorat nahi padti (Seamless SSO).
- **Where you see this** — Jab kisi user ka laptop Entra join nahi ho raha ho ya cloud mein naya password sync na ho raha ho.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Client machine par `dsregcmd /status` check karna aur cloud login test karna. |
| **L2** | Manual sync cycles chalana, event logs check karna aur workstation join errors fix karna. |
| **L3** | Service Connection Points (SCP) configure karna, Entra Cloud Sync agents deploy karna. |

> [!tip] Seedha Simple Mein
> *Hybrid Identity se hamare local office systems aur Microsoft cloud aapas me jud jaate hain. Clients "Hybrid Entra Joined" hote hain jisse users bina double logins ke fileservers aur cloud dono use karte hain.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Hybrid Identity** is like having a **Universal Keycard** for two different campuses because...
>
> - The old local office (On-Prem AD) and the new skyscraper (Azure Cloud) have different security systems.
> - **Entra Connect** is the background system that duplicates your old badge data to the new skyscraper.
> - **Seamless SSO** means you flash your badge once at the front gate, and all internal doors open automatically without asking again.

---
## 🔬 Technical Deep Dive

### 1. Entra Connect Sync vs. Entra Cloud Sync

> [!info] Key Concept
> Microsoft provides two engines. Connect Sync is heavy and feature-rich (supports device sync), whereas Cloud Sync is a lightweight agent approach.

- **Connect Sync:** Heavy local footprint (requires DB), supports Device Sync, writebacks for Passwords, Devices, Users. Default schedule: 30 minutes.
- **Cloud Sync:** Lightweight agent, configuration processed in cloud, NO Device Sync, writeback for passwords only. Near real-time syncing.

### 2. Hybrid Microsoft Entra Join

- **Purpose:** Devices join to on-prem AD AND register in Entra ID. Enables Kerberos access to local shares PLUS conditional access validation in the cloud.
- **Discovery:** Clients find the cloud tenant by querying the **Service Connection Point (SCP)** in local AD.

### 3. Seamless Single Sign-On (SSO)

- Enabled by a computer account named `AZUREADSSOACC` in local AD.
- Client gets a Kerberos ticket for this account, which is forwarded to Entra ID to silently authenticate the user.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - On-prem AD Domain Controller.
> - Entra Connect Sync server.
> - Windows 10/11 client VM.

### Step 1: Verify Current Device Registration State

```cmd
dsregcmd /status
```
> [!success] Expected Output
> `AzureAdJoined : NO` and `DomainJoined : YES` before configuration.

### Step 2: Configure Service Connection Point (SCP)

1. Open **Microsoft Entra Connect** on sync server.
2. Click **Configure** > **Configure device options**.
3. Provide Global Admin credentials.
4. Select **Configure Hybrid Azure AD join**.
5. Add SCP configuration for your forest `corp.local` using Enterprise Admin credentials.

### Step 3: Deploy Device Registration GPO

1. Open GPMC on DC. Create `GPO-HybridJoin` linked to computer OU.
2. Edit: Computer Configuration > Policies > Admin Templates > Windows Components > Device Registration.
3. Enable **Register domain-joined computers as devices**.

### Step 4: Trigger Client Sync and Verify

```cmd
gpupdate /force
# Reboot, log back in, then:
dsregcmd /status
```
> [!success] Expected Output
> `AzureAdJoined : YES` and `DomainJoined : YES`.

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `dsregcmd /status` | Displays device join and SSO status | `dsregcmd /status` |
| `dsregcmd /join` | Forces manual device registration | `dsregcmd /join` (Run as System) |
| `dsregcmd /leave` | Unregisters device from Entra ID | `dsregcmd /leave` |
| `Start-ADSyncSyncCycle` | Forces a manual sync of Entra Connect | `Start-ADSyncSyncCycle -PolicyType Delta` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| `AzureAdJoined : NO` | Client cannot reach registration endpoint | Check firewall allows TCP 443 outbound to `https://enterpriseregistration.windows.net`. |
| Automatic registration failed Code: 0x801c03f2 | Computer object not synced yet | Ensure OU is selected in Entra Connect and run `Start-ADSyncSyncCycle -PolicyType Delta`. |
| Prompted for password despite SSO | Browser/App not configured for Kerberos pass-through | Add `https://autologon.microsoftazuread-sso.com` to Intranet Zone in Internet Options via GPO. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: User Prompted for Cloud Passwords

> [!example] Ticket
> "I keep getting asked for my password every time I open Teams, even though I'm logged into my office PC."

**L1 Response:** Verify network connection and check `dsregcmd /status`.
**Escalation Trigger:** If `AzureAdPrt : NO`.
**L2 Resolution:** Run `gpupdate /force`, have user log off and log back in on the corporate network to get a new Primary Refresh Token (PRT). Check Intranet Zone settings for SSO URL.

---
## 🎤 Interview Questions

> [!question] Q1: Which CLI command do you run to verify if a computer has completed Hybrid Microsoft Entra Join?
> **Answer:** `dsregcmd /status`. Both `AzureAdJoined` and `DomainJoined` should be YES.

> [!question] Q2: What is a Primary Refresh Token (PRT)?
> **Answer:** A JWT issued by Entra ID during user logon. It proves authentication for the device and user, enabling SSO across M365 desktop apps and web services.

==**Exam Tip:** The `AZUREADSSOACC` computer account in AD is strictly required for Seamless SSO to function.==

---
## 🔗 Related Notes

- [[04-Cloud-and-Security/07-Microsoft-365/M365-07 Azure-AD-Connect|M365-07 Azure-AD-Connect]] — Deep dive into the Entra Connect sync engine configuration.
- [[04-Cloud-and-Security/07-Microsoft-365/Microsoft-Entra-ID|Microsoft-Entra-ID]] — Cloud identity directory structures.
- [[04-Cloud-and-Security/08-Azure/AZ104-04 Azure Virtual Networking|AZ104-04 Azure Virtual Networking]] — Hybrid connectivity.
