---
tags: [desktop-support, azure, cloud, L1]
aliases: [az9-05-azure-identity-and-security, az9-05]
created: 2026-06-25
status: #complete
difficulty: #beginner
cert-relevant: #none
---

> [!NOTE|color-blue]
> ☁️ **AZURE CLOUD**

`#complete` `#beginner` `#az-900`

# AZ9-05: Azure Identity and Security

> [!abstract] Overview
> Yeh note identity governance aur threat protection cover karta hai Microsoft Azure mein. Ek support engineer ke liye Entra ID, Authentication vs Authorization, aur Conditional Access jaanna zaroori hai taaki resources secure rahen.

---
## 🧠 Concept Overview

- **What it is** — Identity and security principles for cloud resources in Azure.
- **Why it matters** — Proper security configuration prevents unauthorized access and data breaches.
- **Where you see this** — Managing user access, troubleshooting sign-ins, and configuring role assignments.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Check user identity status, perform password resets, and verify basic access rights. |
| **L2** | Configure RBAC permissions, troubleshoot Conditional Access blocks, and enforce MFA. |
| **L3** | Design Zero Trust architecture, configure Sentinel SIEM, and implement enterprise-level identity governance. |

> [!tip] Seedha Simple Mein
> *Seedha simple mein: Microsoft Entra ID (pehle Azure AD) cloud identity manager hai. Authentication identity verify karta hai aur Authorization (RBAC) permissions check karta hai. Secrets aur keys safe rakhne ke liye hum Key Vault use karte hain.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Corporate Research Facility** is like **Azure Identity and Security** because...
>
> - **Microsoft Entra ID** is the central employee badge registration office. It records who you are, what department you belong to, and issues your master photo ID.
> - **Authentication** is verifying your face matches the photo on the ID badge when you walk through the gate.
> - **Authorization** is verifying if your badge has authorization clearance (RBAC) to open the door to Laboratory 3.
> - **Conditional Access** is the smart check: "Even if your ID is valid, if you are attempting entry at 2:00 AM (Time condition) from an external unknown IP address (Location condition), you must supply a second security key (MFA) before the door unlocks."
> - **Azure Key Vault** is the high-security steel safe in the basement where the office keys and passwords are locked away.

---
## 🔬 Technical Deep Dive

### 1. Microsoft Entra ID (Formerly Azure Active Directory)

> [!info] Key Concept
> Entra ID is Microsoft's cloud-based identity and access management service.

- **Differs from legacy AD DS:** Entra ID does not run on Domain Controllers using Kerberos/NTLM authentication. Instead, it is a flat, cloud-native identity service running web-friendly authentication protocols: **OAuth 2.0**, **SAML**, and **OpenID Connect**.
- **Tenant:** A dedicated instance of Entra ID representing a single organization.

### 2. Authentication vs. Authorization

- **Authentication (AuthN):** The process of verifying the identity of a user or service (proving "Who you are"). Methods include passwords, SMS codes, biometrics, or security keys.
- **Authorization (AuthZ):** The process of verifying what resources the authenticated user is allowed to access (proving "What you can do"). Checked via RBAC permissions.

> [!danger] Common Mistake
> Confusing AuthN with AuthZ. AuthN = Identity (Who are you?), AuthZ = Access (What can you do?).

### 3. Multifactor Authentication (MFA) & Conditional Access

- **MFA:** Demands two or more credentials from different categories:
  - *Something you know:* Password or PIN.
  - *Something you have:* Mobile authenticator app token or USB security key.
  - *Something you are:* Fingerprint or face ID.

- **Conditional Access (Entra ID Premium):** A policy engine that evaluates real-time signals (User/group, IP Location, Device state, Application target) to enforce security rules:
  - *Rule example:* "If a user attempts to log into the Azure Portal from outside the country, Block access, or Require MFA."

### 4. Role-Based Access Control (RBAC)

RBAC manages authorization by assigning roles to security principals at specific scopes.
- **Core Built-in Roles:**
  - **Owner:** Full access to all resources, including the ability to delegate access to others (assign RBAC roles).
  - **Contributor:** Can create and manage all types of Azure resources, but cannot delegate access to others.
  - **Reader:** Can view existing Azure resources but cannot make modifications.
  - **User Access Administrator:** Can only manage user access and role assignments (cannot create resources).

### 5. Zero Trust Security Model

A security framework designed on three core principles:
1. **Verify Explicitly:** Always authenticate and authorize based on all available data points (never trust blindly).
2. **Use Least Privilege Access:** Limit user access to only the resources they need for their immediate task (Just-In-Time and Just-Enough-Access).
3. **Assume Breach:** Design networks assuming attackers are already inside the perimeter. Use segmentation to prevent lateral movement.

### 6. Azure Key Vault & Microsoft Sentinel

- **Azure Key Vault:** A secure cloud service for storing and safeguarding cryptographic keys, secrets (passwords, connection strings), and certificates.
- **Microsoft Sentinel (SIEM):** A scalable, cloud-native **SIEM (Security Information and Event Management)** and **SOAR (Security Orchestration, Automated Response)** system. It aggregates security logs, uses AI to detect anomalies, and executes automated scripts to block threats.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Administrative access to target systems.
> - Target Azure Subscription.

### Step 1: Execute Verification

```bash
# Yeh command verification complete hone ka status print karta hai
echo "Verification Check Completed"
```

> [!success] Expected Output
> ```
> Verification Check Completed
> ```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Concept / Term | 🛠️ Kya karta hai | 📝 Example / Note |
|-----------|-----------------|-----------|
| `Entra ID` | Cloud-based identity management service replacing legacy AD DS. | Uses OAuth 2.0, SAML |
| `AuthN vs AuthZ` | Authentication verifies who you are; Authorization verifies what you have access to do. | Identity vs Access |
| `Conditional Access` | Dynamic policy engine checking device state and location rules before granting entry. | IP-based blocking / MFA prompt |
| `Owner Role` | Premium RBAC role granting full resource control and authorization delegation rights. | Can assign roles |
| `Key Vault` | Secure cloud storage box isolating passwords and certificates from application code files. | Secret management |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| Service connection timeout | Network firewall or routing blocking traffic | Check network route and enable target ports on firewall |
| Access Denied error | User account lacks permissions or invalid credentials | Verify account access permissions or reset password |
| Resource not found | Object or path is misspelled or deleted | Verify spelling of target path or query active objects |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: User Cannot Access Azure Portal

> [!example] Ticket
> "I am getting an Access Denied error when trying to restart my VM in Azure Portal."

**L1 Response:** Verify user account status in Entra ID and check if credentials are correct.
**Escalation Trigger:** Account is active but user still lacks access to specific resources.
**L2 Resolution:** Check IAM (RBAC) roles on the VM's Resource Group. Assign `Virtual Machine Contributor` role if approved.

---
## 🎤 Interview Questions

> [!question] Q1: What is the difference between Microsoft Entra ID and legacy Active Directory Domain Services (AD DS)?
> **Answer:** **AD DS** is an on-premises identity service designed for local network infrastructures. It uses a hierarchical database structure (forests, domains, OUs), requires Domain Controllers, and authenticates clients using Kerberos and NTLM protocols. **Microsoft Entra ID** is a flat, cloud-native identity service designed for internet-based resource management. It does not use OUs or group policies natively. It manages users and devices over HTTP/HTTPS, authenticating connections using modern web protocols like SAML, OpenID Connect, and OAuth 2.0.

> [!question] Q2: An administrator needs to grant a developer the ability to restart and manage Virtual Machines in a Resource Group, but wants to prevent them from deleting the VMs or modifying network switches. How do you configure this?
> **Answer:** Assign the built-in **Virtual Machine Contributor** role to the developer's Entra ID user account at the Resource Group scope. They can start, stop, restart, and edit VMs inside that specific Resource Group, but are blocked from deleting VMs, modifying VNet interfaces, or accessing other resource groups.

> [!question] Q3: Describe the three core pillars of the Zero Trust security model.
> **Answer:** The three core pillars of Zero Trust are: 1) **Verify Explicitly:** Authenticate and authorize every access request based on user identity, location, device health. 2) **Use Least Privilege Access:** Grant users only the minimum access rights required. 3) **Assume Breach:** Assume the system has already been compromised. Segment networks, encrypt all data streams.

==**Exam Tip:** Conditional Access requires Entra ID Premium P1 or P2. AuthN is Identity, AuthZ is Permissions!==

---
## 🔗 Related Notes

- [[04-Cloud-and-Security/08-Azure/AZ9-01 Cloud Computing Fundamentals|AZ9-01 Cloud Computing Fundamentals]] — Shared responsibility model.
- [[04-Cloud-and-Security/08-Azure/AZ104-01 Azure Identity and Governance|AZ104-01 Azure Identity and Governance]] — Advanced Entra ID configuration, group management, and custom roles.
