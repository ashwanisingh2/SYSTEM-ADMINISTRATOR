---
tags: [desktop-support, azure, identity, hybrid, L2]
aliases: [entra-connect-hybrid, hybrid-identity]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #az-104
---

# AZ104-07: Azure AD Connect and Hybrid Identity

> [!note] Overview
> This note covers hybrid identity architecture in Microsoft Azure. It details the comparison between Entra Connect Sync and Entra Cloud Sync, configuring Hybrid Microsoft Entra Join for corporate endpoints, Seamless Single Sign-On (SSO), and hybrid identity diagnostics.

---
## Concept Overview
- **What it is** — Hybrid Identity links on-premises Active Directory Domain Services (AD DS) with Microsoft Entra ID in the cloud. It uses synchronization tools like Entra Connect Sync or Entra Cloud Sync to replicate identities and enables features like Seamless Single Sign-On (SSO) and Hybrid Device Join.
- **Why it matters** — Enterprise migration is rarely a 100% cloud-only move. Organizations operate legacy on-premises assets (fileservers, local print servers) alongside cloud resources. Hybrid identity guarantees users need only one identity to access both environments securely.
- **Real job encounter** — Configuring company laptops for Hybrid Entra Join, troubleshooting client Single Sign-On (SSO) failures, and monitoring directory synchronization health.
- **L1 vs L2 vs L3 responsibility split:**
  - **L1 Resolution**: Verify client machine domain connectivity, run local device join check commands (`dsregcmd /status`), and check user cloud login status.
  - **Escalation Trigger**: Escalate if multiple client machines fail to complete hybrid registration, or if users are repeatedly prompted for credentials despite successful domain login.
  - **L2 Resolution**: Force manual synchronization loops on the sync server, troubleshoot join errors using workstation event logs, and verify UPN mapping compatibility.
  - **L3 Resolution**: Configure Service Connection Points (SCP) in local AD, implement Seamless Single Sign-On, deploy Microsoft Entra Cloud Sync agents, and monitor sync status using Entra Connect Health.

---
## Technical Deep Dive

### 1. Entra Connect Sync vs. Entra Cloud Sync
Microsoft offers two synchronization engines for hybrid environments:

| Feature | Microsoft Entra Connect Sync | Microsoft Entra Cloud Sync |
|---|---|---|
| **Architecture** | Heavy local footprint; requires local database (SQL Express/Full) | Lightweight agent model; configuration processed in the cloud |
| **Multi-Forest** | Complex configuration | Simple setup; ideal for merged organizations |
| **Device Sync** | Supported (Hybrid Entra Join) | Not supported |
| **Writeback** | Passwords, Devices, Users | Passwords only |
| **Default Schedule**| 30-minute interval | Immediate delta syncing (near real-time) |

### 2. Hybrid Microsoft Entra Join
Hybrid Entra Join allows devices to be joined to on-premises AD and registered in Microsoft Entra ID.
- **Why it is used**: It enables Kerberos-based access to local shares/printers AND authorizes conditional access validation (compliance checks) in the cloud.
- **Discovery**: Clients discover the cloud tenant by querying the **Service Connection Point (SCP)** partition in the local AD configuration container, which contains the Entra tenant ID and verified domain.

### 3. Seamless Single Sign-On (Seamless SSO)
- Enables users inside the physical office network to log in to cloud applications automatically.
- When enabled, Entra ID creates a computer account named `AZUREADSSOACC` in the on-premises AD.
- During cloud login, the client is redirected to request a Kerberos ticket from local AD for this computer account, which is forwarded to Entra ID to authenticate the user without requiring password entry.

---
## Step-by-Step Lab

> [!warning] Pre-requisites
> - An on-premises Active Directory Domain Controller (`DC-01.corp.local`).
> - A member server (`SVR-ADSYNC01.corp.local`) running Entra Connect Sync.
> - A domain-joined Windows 10/11 client VM (`CLI-WIN10`).
> - Cloud Global Administrator credentials.

### Step 1: Verify Current Device Registration State on Client
1. Log in to `CLI-WIN10` using domain credentials.
2. Open Command Prompt and check device status:
```cmd
dsregcmd /status
```
**Expected Output:** Under Device State, verify that:
- `AzureAdJoined : NO`
- `DomainJoined : YES`

### Step 2: Configure Service Connection Point (SCP) in local AD
Clients cannot register in the cloud if they cannot locate the target tenant. We configure the SCP via the Entra Connect wizard on the sync server.

1. Log in to `SVR-ADSYNC01`. Open **Microsoft Entra Connect**.
2. Click **Configure** -> Select **Configure device options** -> Click Next.
3. Input cloud Global Admin credentials.
4. Select **Configure Hybrid Azure AD join**. Click Next.
5. On the SCP page, check the box next to your forest `corp.local`.
6. Select Authentication Service: **Microsoft Entra ID**.
7. Click **Add** and input Enterprise Admin credentials to write the SCP configuration to Active Directory. Click **Configure**.

### Step 3: Deploy Device Registration Group Policy
We will configure a GPO to instruct workstations to attempt registration.

1. On `DC-01`, open Group Policy Management (`gpmc.msc`).
2. Create and link a new GPO named `GPO-HybridJoin` to the computer OU.
3. Edit the GPO and navigate to:
`Computer Configuration > Policies > Administrative Templates > Windows Components > Device Registration`
4. Double-click **Register domain-joined computers as devices**. Set to **Enabled**. Click OK.

### Step 4: Trigger Client Sync and Verify Registration
1. On `CLI-WIN10`, open Command Prompt and execute a policy update:
```cmd
gpupdate /force
```
2. Reboot the client workstation.
3. Log back in. Open Command Prompt and run the registration check:
```cmd
dsregcmd /status
```
**Expected Output:** Under Device State, verify that:
- `AzureAdJoined : YES`
- `DomainJoined : YES`
*(Note: Under User State, verify that `WorkspaceJoined : NO` and `WAM Default Account : YES` indicating cloud SSO tokens are active).*

---
## Cheat Sheet / Quick Reference

| Command / Configuration | Purpose | Target Path / Example |
|---|---|---|
| `dsregcmd /status` | Displays device join and SSO authentication status | Local CLI |
| `dsregcmd /join` | Forces manual device registration checks | Local CLI (run under System context) |
| `dsregcmd /leave` | Unregisters the device from Microsoft Entra ID | `dsregcmd /leave` |
| **`AZUREADSSOACC`** | Computer account created in local AD to facilitate SSO | Active Directory computer object |
| **`dsregcmd` Log location** | Event viewer path for join diagnostics | Applications and Services Logs > Microsoft > Windows > User Device Registration |
| **SCP AD Partition** | Registry configuration path in Active Directory | `CN=62a0ff2e-...,CN=Services,CN=Configuration,DC=...` |

---
## Troubleshooting Table

| Problem | Cause | Fix |
|---|---|---|
| `dsregcmd /status` shows `AzureAdJoined : NO` and `DomainJoined : YES`. | Client is unable to communicate with the Entra ID device registration endpoints. | Check Event Viewer -> User Device Registration log. Ensure ports TCP 443 outbound to `https://enterpriseregistration.windows.net` are allowed by local firewalls. |
| User is prompted for password when launching M365 apps, despite SSO configuration. | The browser or application is not configured to pass Kerberos tickets to the cloud. | Deploy Group Policy to add `https://autologon.microsoftazuread-sso.com` to the user's Intranet Zone site list in Internet Options. |
| Event logs show error: "Automatic registration failed. Code: 0x801c03f2." | The computer object has not been synchronized to Microsoft Entra ID by Entra Connect yet. | Ensure the computer object's OU is selected for synchronization in Entra Connect, and run `Start-ADSyncSyncCycle -PolicyType Delta`. |
| Workstation displays status: `AzureAdJoined : YES` but user state has `AzureAdPrt : NO`. | Primary Refresh Token (PRT) was not issued due to authentication policy mismatch. | Run `gpupdate /force`. Have the user log off and log in again over a direct connection to the corporate network (or local VPN). |
| Multi-forest client machines cannot discover the cloud tenant. | The SCP is missing in one of the Active Directory forests. | Re-run the Entra Connect device options configuration wizard and add the SCP config for all active forests. |

---
## Interview Questions

> [!question] L1 Question
> **Q:** Which Windows CLI command do you run to verify if a user's computer has completed Hybrid Microsoft Entra Join?
> **A:** I run the command **`dsregcmd /status`** in the command prompt. I check the values for `AzureAdJoined` and `DomainJoined`. For a successful hybrid join, both parameters must display **`YES`**.

> [!question] L2 Question
> **Q:** What is a Primary Refresh Token (PRT), and why is it important for Hybrid Azure AD joined machines?
> **A:** A Primary Refresh Token (PRT) is a key JSON Web Token (JWT) issued by Microsoft Entra ID to Windows 10/11 devices during user logon. It acts as a proof of authentication for the device and user. It is critical because it enables Single Sign-On (SSO) across M365 desktop apps and web services, allowing users to authenticate without being prompted for credentials.

> [!question] L3/Scenario Question
> **Q:** You are troubleshooting a hybrid join failure. In the User Device Registration event logs, you see error: "The device object was not found in the tenant. Code: 0x801c03f2." Describe the root cause and how you resolve it.
> **A:** 
> - **Situation:** Workstation registration fails with error code `0x801c03f2` (Device object not found).
> - **Task:** Resolve the synchronization delay between local Active Directory and Entra ID.
> - **Action:** 
>   1. **Identify Cause**: When a client computer attempts a hybrid join, it contacts Entra ID to register. Entra ID searches for the computer's synchronized AD object. If Entra Connect has not synchronized the computer object yet, the join fails with this error.
>   2. **Verify Sync Scope**: Open the Entra Connect wizard and ensure the OU containing the computer object is selected in the OU filtering page.
>   3. **Force Sync**: Open PowerShell on the sync server and execute `Start-ADSyncSyncCycle -PolicyType Delta` to push the computer object immediately.
>   4. **Trigger Client Registration**: On the client, run `gpupdate /force`, restart, or execute `dsregcmd /join` under the System context.
> - **Result:** The device object synchronizes, allowing Entra ID to complete registration and set `AzureAdJoined : YES`.

---
## Seedha Simple Mein
*Seedha simple mein: Hybrid Identity se hamare local office systems (Active Directory) aur Microsoft cloud (Entra ID) aapas me jud jaate hain. Iske baad client laptops "Hybrid Entra Joined" ho jaate hain, jisse users bina double logins ke fileservers aur cloud apps dono seamlessly access kar sakte hain. Iska status check karne ke liye workstation par `dsregcmd /status` run kiya jata hai.*

---
## Related Notes
- [[04-Cloud-and-Security/07-Microsoft-365/M365-07 Azure-AD-Connect|M365-07 Azure-AD-Connect]] — Deep dive into the Entra Connect sync engine configuration.
- [[04-Cloud-and-Security/07-Microsoft-365/Microsoft-Entra-ID|Microsoft-Entra-ID]] — Cloud identity directory structures.
- [[04-Cloud-and-Security/08-Azure/AZ104-04 Azure Virtual Networking|AZ104-04 Azure Virtual Networking]] — Detailing hybrid connectivity (VPN, ExpressRoute) supporting sync.

---
*Tags: #desktop-support #azure #identity #hybrid #L2 #cert-az-104*
