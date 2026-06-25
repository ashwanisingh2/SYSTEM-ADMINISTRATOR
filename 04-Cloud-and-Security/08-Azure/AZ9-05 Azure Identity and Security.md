---
tags: [sysadmin, azure, az-900, security, identity]
difficulty: Beginner
lab-required: No
read-time: 10 mins
---

# AZ9-05: Azure Identity and Security

> [!abstract] Overview
> This note covers identity governance and threat protection in Microsoft Azure. It details Microsoft Entra ID, Authentication vs. Authorization, Conditional Access, Role-Based Access Control (RBAC), Zero Trust architecture, Key Vault security, and Sentinel SIEM.

---
## Concept
Think of identity and security in Azure like securing a high-tech corporate research facility:
- **Microsoft Entra ID** is the central employee badge registration office. It records who you are, what department you belong to, and issues your master photo ID.
- **Authentication** is verifying your face matches the photo on the ID badge when you walk through the gate.
- **Authorization** is verifying if your badge has authorization clearance (RBAC) to open the door to Laboratory 3.
- **Conditional Access** is the smart check: "Even if your ID is valid, if you are attempting entry at 2:00 AM (Time condition) from an external unknown IP address (Location condition), you must supply a second security key (MFA) before the door unlocks."
- **Azure Key Vault** is the high-security steel safe in the basement where the office keys and passwords are locked away.

*Seedha simple mein: Microsoft Entra ID (pehle Azure AD) cloud identity manager hai. Authentication identity verify karta hai aur Authorization (RBAC) permissions check karta hai. Secrets aur keys safe rakhne ke liye hum Key Vault use karte hain.*

---
## Technical Deep Dive

### 1. Microsoft Entra ID (Formerly Azure Active Directory)
Entra ID is Microsoft's cloud-based identity and access management service.
- **Differs from legacy AD DS:** Entra ID does not run on Domain Controllers using Kerberos/NTLM authentication. Instead, it is a flat, cloud-native identity service running web-friendly authentication protocols: **OAuth 2.0**, **SAML**, and **OpenID Connect**.
- **Tenant:** A dedicated instance of Entra ID representing a single organization.

### 2. Authentication vs. Authorization
- **Authentication (AuthN):** The process of verifying the identity of a user or service (proving "Who you are"). Methods include passwords, SMS codes, biometrics, or security keys.
- **Authorization (AuthZ):** The process of verifying what resources the authenticated user is allowed to access (proving "What you can do"). Checked via RBAC permissions.

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

### 6. Azure Key Vault
A secure cloud service for storing and safeguarding cryptographic keys, secrets (passwords, connection strings), and certificates.
- **Isolation:** Prevents developers from writing plain-text passwords in application code. The code queries Key Vault dynamically to retrieve credentials.

### 7. Microsoft Sentinel (SIEM)
A scalable, cloud-native **SIEM (Security Information and Event Management)** and **SOAR (Security Orchestration, Automated Response)** system. It aggregates security logs from all servers, devices, and cloud hosts, uses AI to detect anomalies, and executes automated scripts to block threats.

---
## Quick Revision Table
| # | Concept | One Line Summary |
|---|---------|-----------------|
| 1 | Entra ID | Cloud-based identity management service replacing legacy AD DS Kerberos protocols. |
| 2 | AuthN vs AuthZ | Authentication verifies who you are; Authorization verifies what you have access to do. |
| 3 | Conditional Access| Dynamic policy engine checking device state and location rules before granting entry. |
| 4 | Owner Role | Premium RBAC role granting full resource control and authorization delegation rights. |
| 5 | Key Vault | Secure cloud storage box isolating passwords and certificates from application code files. |

---
## Interview Q&A

**Q1: What is the difference between Microsoft Entra ID and legacy Active Directory Domain Services (AD DS)?**
A: **AD DS** is an on-premises identity service designed for local network infrastructures. It uses a hierarchical database structure (forests, domains, OUs), requires Domain Controllers, and authenticates clients using Kerberos and NTLM protocols. **Microsoft Entra ID** is a flat, cloud-native identity service designed for internet-based resource management. It does not use OUs or group policies natively. It manages users and devices over HTTP/HTTPS, authenticating connections using modern web protocols like SAML, OpenID Connect, and OAuth 2.0.

**Q2: An administrator needs to grant a developer the ability to restart and manage Virtual Machines in a Resource Group, but wants to prevent them from deleting the VMs or modifying network switches. How do you configure this?**
A: 
- **Situation:** A developer needs restricted VM management permissions on a Resource Group scope.
- **Task:** Select and apply the appropriate RBAC role and scope.
- **Action:** I will navigate to the target Resource Group in the Azure Portal. Under **Access Control (IAM)**, I will click Add Role Assignment. I will select the built-in **Virtual Machine Contributor** role, and assign it to the developer's Entra ID user account.
- **Result:** The developer can start, stop, restart, and edit VMs inside that specific Resource Group, but is blocked from deleting VMs, modifying the VNet interfaces, or accessing other resource groups.

**Q3: Describe the three core pillars of the Zero Trust security model.**
A: The three core pillars of Zero Trust are: 
1) **Verify Explicitly:** Authenticate and authorize every access request based on user identity, location, device health, and anomalies, rather than assuming trust because the device is inside the network.
2) **Use Least Privilege Access:** Grant users only the minimum access rights required for their specific job role using JIT (Just-In-Time) and JEA (Just-Enough-Access) configurations to protect sensitive data.
3) **Assume Breach:** Assume the system has already been compromised. Segment networks, encrypt all data streams, and use analytics to monitor for threats to minimize blast radius.

---
## Related Notes
- [[04-Cloud-and-Security/08-Azure/AZ9-01 Cloud Computing Fundamentals|AZ9-01 Cloud Computing Fundamentals]] - Shared responsibility model.
- [[04-Cloud-and-Security/08-Azure/AZ104-01 Azure Identity and Governance|AZ104-01 Azure Identity and Governance]] - Advanced Entra ID configuration, group management, and custom roles.

