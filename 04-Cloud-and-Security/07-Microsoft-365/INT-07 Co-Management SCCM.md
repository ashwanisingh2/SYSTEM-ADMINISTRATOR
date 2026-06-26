---
tags: [microsoft365, intune, sccm, co-management, mdm, advanced]
aliases: [Co-Management, SCCM-Intune, SCCM Co-Management, MECM Co-Management]
created: 2026-06-26
status: #complete
difficulty: #advanced
cert-relevant: #md-102
---

> [!NOTE|color-blue]
> ☁️ **MICROSOFT 365 / INTUNE**

`#complete` `#advanced` `#md-102`

# INT-07: Co-Management SCCM (MECM) & Intune

> [!abstract] Overview
> Yeh note SCCM (System Center Configuration Manager / MECM) aur Microsoft Intune ke Co-Management feature ko cover karta hai. Ek support engineer ya system admin ke liye yeh samajhna zaroori hai ki on-premise devices ko Cloud se kaise manage kiya jaata hai bina existing infrastructure ko tode. Is note mein architecture, workloads, logs aur interview questions shamil hain.

---
## 🧠 Concept Overview

- **What it is** — Co-Management allows you to concurrently manage Windows 10/11 devices by using both Configuration Manager (SCCM) and Microsoft Intune. It lets you attach your existing on-premises investment to the Microsoft 365 cloud, bringing modern management capabilities.
- **Why it matters** — Real job mein, organizations fully cloud pe overnight shift nahi ho sakti. On-premise servers, complex applications aur legacy systems ko handle karne ke liye SCCM zaroori hai. Cloud se conditional access aur modern deployment ke liye Intune zaroori hai. Co-Management ek bridge hai jo dono ki taaqat ko combine karta hai bina kisi ek ko replace kiye.
- **Where you see this** — Hybrid environments jahan corporate devices on-premise Active Directory se join hain but remote work (Work From Home) ke liye internet-based cloud management bhi chahiye.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic task — Check karna ki device Co-Managed state mein hai ya nahi (Configuration Manager applet se). Logs collect karna agar enrollment fail ho raha ho. |
| **L2** | Configure, fix, escalate kab karta hai — Client enrollment issues resolve karna, Intune sync errors troubleshoot karna. Workload policies verify karna client side pe. |
| **L3** | Architecture, design, enterprise-level — Co-Management setup design karna, workload sliders configure karna, Cloud Attach setup, aur PKI/CMG integration. |

> [!tip] Seedha Simple Mein
> *Maan lo SCCM ek purana, powerful desktop manager hai aur Intune ek naya cloud manager. Co-Management in dono ki dosti karwa deta hai taaki ek PC dono se order le sake. Pura control aapke haath mein hota hai ki kaun sa tool kis chiz (workload) ko manage karega. Agar aap chaho ki updates Intune sambhale, toh aap bs ek slider move kar dete ho.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **A Hybrid Car** is like **Co-Management** because...
>
> - **Petrol Engine (SCCM):** Reliable, powerful, heavy-duty tasks ke liye (OSD, complex legacy app deployments, deep system configurations).
> - **Electric Motor (Intune):** Modern, efficient, lightweight, aur remote access ke liye (Compliance, conditional access, remote wipe, mobile management).
> - **Driver (Admin):** Switch kar sakta hai ki kab petrol use karna hai aur kab electric (Workload shifting). Jab hybrid car start hoti hai (Provisioning), yeh automatically decide karti hai best engine as per configuration.

---
## 🔬 Technical Deep Dive

### 1. Prerequisites for Co-Management

> [!warning] Pre-requisites
> - Configuration Manager (Current Branch) version 1710 or later (Lekin latest version best hai).
> - Azure Active Directory (Azure AD / Entra ID) ya Hybrid Azure AD Joined devices. Device ka Entra ID mein dikhna zaroori hai.
> - Intune subscription and EMS licensing (E3/E5).
> - SCCM Client installed and healthy on the Windows devices.
> - Intune Auto-enrollment enabled in Entra ID (MDM User Scope set to 'All' or 'Some').
> - Cloud Management Gateway (CMG) recommended for internet-facing remote clients.

### 2. The Seven Workloads (Kaun kya karega?)

> [!info] Key Concept
> **Workloads** determine karte hain ki SCCM ya Intune mein se kaun authority hold karta hai ek specific task ke liye. By default, sab SCCM ke paas hota hai.

Jab Co-Management enable hota hai, toh admin in workloads ko SCCM se Intune par shift kar sakta hai:
1. **Compliance Policies:** Check device health, password requirements, encryption status.
2. **Device Configuration:** Wi-Fi profiles, VPN, Certificates, Administrative Templates.
3. **Endpoint Protection:** Windows Defender Antivirus, Firewall, Application Guard.
4. **Resource Access Policies:** EAP-TLS profiles, SCEP certificates.
5. **Client Apps:** Application deployment (Intune Company Portal).
6. **Office Click-to-Run apps:** Microsoft 365 Apps deployment and updates.
7. **Windows Update policies:** Feature updates, Quality updates via Windows Update for Business (WUfB).

*Hindi translation:* *Workload slider ek volume button ki tarah hai SCCM console mein. Agar aap "Windows Update" ko Intune pe set kar doge, toh aage se OS updates SCCM Distribution Point se nahi balke seedha Microsoft Cloud (Intune) se aayenge.*

> [!danger] Common Mistake
> Ekdum se saare workloads Intune pe shift kar dena bina pilot group testing ke. Isse devices par policy conflict aa sakta hai aur devices non-compliant ho sakte hain. Hamesha Pilot collection use karein.

### 3. Client Enrollment States and Co-Management Capabilities

- **Provisioning:** SCCM client is getting policy from Management Point to enroll in Intune.
- **Enrolling:** Device is trying to register to Intune MDM via the Scheduled Task.
- **Co-managed:** Device is successfully managed by both SCCM and Intune.

### 4. Paths to Co-Management

There are two primary ways to reach a Co-Managed state:
- **Path 1: Existing SCCM Clients:** SCCM managed devices are automatically enrolled into Intune. This is the most common path for existing enterprises.
- **Path 2: Internet-first (Autopilot):** New devices are provisioned via Windows Autopilot, enrolled into Intune first, and then the SCCM client is pushed via Intune to make them co-managed.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - SCCM Console access with Full Administrator rights.
> - Intune Administrator and Global Admin rights in Azure portal.

### Step 1: Enable Co-Management in SCCM Console

```powershell
# While this is mostly a GUI task, here is the path in the SCCM Console:
# 1. Administration Workspace > Overview > Cloud Services > Cloud Attach
# 2. Right-click "Cloud Attach" and select "Configure Cloud Attach"
```

> [!success] Quick Win
> GUI wizard mein sign in karne ke baad, Azure AD Tenant automatically detect ho jayega. "Use Intune" check box tick karna na bhulein.

### Step 2: Configure Client Enrollment Options

Choose how clients enroll into Intune in the Enablement tab:
- **Pilot:** Only a specific Device Collection will auto-enroll into Intune. *(Best practice for rollout)*
- **All:** All eligible SCCM clients will auto-enroll into Intune.

*Hindi tip:* *Pilot ka matlab hai pehle IT team ki devices pe test karo, phir poore office mein lagao.*

### Step 3: Shift Workloads (Pilot to Intune)

Console Path: `Administration > Overview > Cloud Services > Cloud Attach > Properties > Workloads tab`.
- Slide the workload (e.g., Compliance Policies) to **Pilot Intune** or **Intune**.
- Agar Pilot Intune select kiya hai, toh Staging tab mein specify karein ki kaunsi collection is workload ke liye Intune use karegi.

### Step 4: Manually Trigger Co-Management Enrollment on Client (For Testing)

```powershell
# To manually trigger the Co-Management enrollment on a client via PowerShell (Run as Admin)
# Run the Configuration Manager client action trigger:
Invoke-WmiMethod -Namespace root\ccm -Class SMS_Client -Name TriggerSchedule -ArgumentList "{00000000-0000-0000-0000-000000000121}"
```

> [!success] Expected Output
> Client immediately checks in with the SCCM Management point to get the Co-Management policy and triggers the scheduled task for MDM enrollment.

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command / Registry Path | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `control smscfgrc` | Opens Configuration Manager Applet on Windows client | Run dialog: `control smscfgrc` |
| `C:\Windows\CCM\Logs\CoManagementHandler.log` | Core log for Co-Management configuration and enrollment status | `cmtrace CoManagementHandler.log` |
| `C:\Windows\CCM\Logs\CIAgent.log` | Log for Compliance and policy application | `cmtrace CIAgent.log` |
| `dsregcmd /status` | Checks Azure AD join status (crucial for Co-Mgmt to work) | `dsregcmd /status` |
| `Get-ItemProperty HKLM:\SOFTWARE\Microsoft\CCM\CoManagement` | Checks the Co-Management Flags on the client registry | Check the `CoManagementFlags` value |
| `Event Viewer > App & Services > Microsoft > Windows > DeviceManagement-Enterprise-Diagnostic-Provider > Admin` | MDM Enrollment event logs to verify Intune registration | Event ID 75 or 76 indicates sync status |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| **Device not enrolling to Intune** | Device is not Hybrid Azure AD Joined or AD Connect is failing | Run `dsregcmd /status`. If not joined, check AAD Connect sync or SCCM Client Settings for Hybrid Join. |
| **CoManagementHandler.log shows 'Failed to enroll'** | MDM User Scope is set to 'None' in Azure AD portal | Go to Entra ID > Mobility (MDM and MAM) > Microsoft Intune > Set MDM user scope to "Some" or "All". |
| **Workload not applying from Intune** | Device is not in the correct Pilot Collection in SCCM | Verify in SCCM Console if the device is part of the collection selected for the Pilot workload in Staging tab. |
| **Conflict between GPO and Intune Policy** | SCCM/Intune policies conflict with on-prem Active Directory GPOs | Ensure `MDMWinsOverGPO` policy is enabled in Intune or remove conflicting on-prem AD GPOs. |
| **Auto-enrollment Scheduled Task not starting** | User does not have Intune/EMS License assigned | Verify user licenses in M365 Admin Center. Co-Management requires an active Intune license. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Device Shows "Co-managed" as NO in Intune Console

> [!example] Ticket
> "L1 Helpdesk: User's laptop is missing from Intune devices list, but SCCM shows the client is active. We need to deploy an Intune-only app to this user immediately."

**L1 Response:** Pehle client PC par check karo ki Co-Management enable hua hai ya nahi. `control smscfgrc` open karke "General" tab mein "Co-management: Enabled" aana chahiye. Aur `dsregcmd /status` check karo ki hybrid join working hai.
**Escalation Trigger:** Agar `CoManagementHandler.log` mein authentication ya MDM enrollment errors hain jo resolve nahi ho rahe.
**L2 Resolution:** L2 admin logs check karta hai. Pata chalta hai ki user account ke paas Intune/EMS license missing hai. User ko Azure portal mein license assign karne ke baad, device automatically kuch der mein enroll ho jata hai.

### 🎫 Scenario 2: Windows Updates Failing After Workload Shift

> [!example] Ticket
> "Security Team: Several Windows 11 devices in the Sales department are not receiving the latest zero-day patch via SCCM Software Center."

**L1 Response:** Check SCCM Client logs `WUAHandler.log` aur `UpdatesDeployment.log`. Dekho ki "Windows Update" workload kahan point kar raha hai.
**Escalation Trigger:** Agar workload SCCM pe set hai par updates nahi aa rahe, ya workload Intune pe shift ho chuka hai galti se without policies.
**L2 Resolution:** SCCM Console mein check kiya toh "Windows Update policies" workload galti se Intune pe shift kar diya gaya tha bina proper Update Rings configure kiye. Workload wapas SCCM pe laya gaya (slider move karke) temporary fix ke liye, jab tak Intune policies test na ho jaye.

### 🎫 Scenario 3: Device Stuck in "Provisioning" State

> [!example] Ticket
> "SysAdmin: Multiple devices in the new remote office are stuck in 'Provisioning' state for Co-Management for the last 3 days. They never reach the Enrolled state."

**L1 Response:** Pings check karo SCCM management point aur domain controller tak. Check `LocationServices.log` on client to see if it can reach the server.
**Escalation Trigger:** Network connectivity theek hai par phir bhi provisioning aage nahi badh rahi aur MDM logs empty hain.
**L2 Resolution:** Remote office devices VPN pe nahi hain aur unke paas Cloud Management Gateway (CMG) assign nahi hai. Internet se SCCM server reach nahi ho raha. SCCM boundary groups fix kiye gaye aur CMG configuration ko un clients tak push kiya gaya taaki internet se communicate karke enrollment policy download kar sakein.

### 🎫 Scenario 4: Autopilot Device Failing to get SCCM Client

> [!example] Ticket
> "Desktop Support: New laptops built with Autopilot are enrolled in Intune perfectly, but they are not installing the SCCM client. They cannot access legacy on-prem apps."

**L1 Response:** Intune portal mein device check karo aur SCCM Client deployment app status dekho.
**Escalation Trigger:** App installation stuck at "Waiting for install status".
**L2 Resolution:** SCCM Client installation properties in Intune LOB app mein command line parameters galat the. `CCMSETUPCMD="CCMHOSTNAME=CMG.CONTOSO.COM/CCM_Proxy_MutualAuth/72... SMSSiteCode=PS1"` theek se update kiya gaya so that internet clients can find the CMG and download the SCCM agent.

---
## 🎤 Interview Questions

> [!question] Q1: What is the primary difference between SCCM and Intune, and why use Co-Management?
> **Answer:** SCCM is an on-premise tool best for OS deployment, deep server management, and heavy legacy application deployment. Intune is a cloud-native MDM/MAM solution best for modern Windows devices, conditional access, and mobile device management. Co-Management allows organizations to use both simultaneously on a single Windows device, migrating workloads to the cloud at their own pace without disruption.

> [!question] Q2: How does a device know it is Co-Managed? Which log do you check on the client?
> **Answer:** The SCCM Client receives the co-management policy from the Management Point. You can visually check the Configuration Manager applet in Control Panel (General tab -> Co-management state). For deep technical troubleshooting, the main log to check is `C:\Windows\CCM\Logs\CoManagementHandler.log`.

> [!question] Q3: If the "Compliance Policies" workload is set to Intune, but there is an active SCCM compliance baseline applied, which one wins?
> **Answer:** When the workload is shifted to Intune, Intune becomes the authority for compliance policies. The SCCM client will automatically stop enforcing its compliance baselines for those specific workloads on that client to avoid conflicts.

> [!question] Q4: What are the two main paths to set up Co-Management?
> **Answer:**
> 1. **Auto-enroll existing SCCM clients:** You configure SCCM to automatically enroll already SCCM-managed domain-joined devices into Intune. (Path 1)
> 2. **Bootstrap SCCM client via Intune:** A device is initially provisioned via Windows Autopilot and managed by Intune, and then the SCCM client is installed onto it as an application via Intune to make it co-managed. (Path 2)

> [!question] Q5: Can you shift a workload back to SCCM if Intune deployment fails?
> **Answer:** Yes, the workload slider in the SCCM console allows you to shift workloads back and forth. If you shift a workload back to Configuration Manager, the SCCM client will resume evaluating and applying policies for that specific workload.

==**Exam Tip:** Hamesha yaad rakhna ki MDM User Scope in Azure AD (Entra ID) must be configured to allow SCCM to auto-enroll devices into Intune. Iske bina auto-enrollment hamesha fail hoga! The SCCM Client must also be able to reach Azure endpoints.==

---
## 🔗 Related Notes

- [[INT-01 Introduction to Intune|INT-01 Introduction to Intune]] — Basics of Intune
- [[INT-02 Windows Autopilot|INT-02 Windows Autopilot]] — Provisioning devices from the cloud
- [[SCCM-01 Overview|SCCM-01 Overview]] — Basics of Configuration Manager
- [[AZ-05 Azure AD Join vs Hybrid|AZ-05 Azure AD Join vs Hybrid]] — Prerequisites for Co-Management
