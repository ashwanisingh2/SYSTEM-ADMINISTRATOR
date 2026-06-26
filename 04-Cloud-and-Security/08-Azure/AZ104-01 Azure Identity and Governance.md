---
tags: [desktop-support, azure, cloud, L2]
aliases: [az104-01-azure-identity-and-governance, az104-01]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #az-104
---

> [!NOTE|color-blue]
> ☁️ **AZURE CLOUD**

`#complete` `#advanced` `#az-104`

# AZ104-01: Azure Identity and Governance

> [!abstract] Overview
> Yeh note advanced Azure identity management aur resource governance ke baare mein hai. Isme B2B, B2C, Dynamic Groups, Administrative Units, custom RBAC, aur Azure Policy cover hota hai.

---
## 🧠 Concept Overview

- **What it is** — Advanced identity and governance tools in Azure to control access and compliance.
- **Why it matters** — Enterprise environments mein security aur access scope manage karna zaroori hai.
- **Where you see this** — Providing guest access, scoping admin roles, automating group memberships.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic task — Verify basic RBAC assignments, check user access. |
| **L2** | Configure, fix, escalate — Create Dynamic Groups, scope Administrative Units. |
| **L3** | Architecture, design — Design Custom RBAC JSON definitions, Policy Initiatives. |

> [!tip] Seedha Simple Mein
> *Seedha simple mein: Azure mein kaun kya kar sakta hai (RBAC) aur kis resource par kar sakta hai (Governance), iska pura control.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Azure Identity & Governance** is like **a multi-tenant skyscraper** because...
>
> - **Entra ID Tenant** is the entire building.
> - **B2B** is a contractor badge for guests.
> - **Dynamic Groups** are automatic door permissions based on your HR file.
> - **Administrative Units** limit a manager to only the 4th floor.

---
## 🔬 Technical Deep Dive

### 1. Entra ID Profiles
- **B2B**: Guest users from other organizations.
- **B2C**: Customer identity (social logins) for consumer apps.

### 2. Groups and Units
- **Dynamic Groups**: Automatically update membership based on user attributes (e.g., department).
- **Administrative Units**: Partition Entra ID to scope admin permissions locally (e.g., London Helpdesk).

### 3. Custom RBAC and Policies
- **Custom RBAC**: JSON definitions for specific access when built-in roles don't fit.
- **Policy Initiatives**: A collection of multiple Azure policies (e.g., PCI-DSS).

> [!danger] Common Mistake
> Assigning Owner or Contributor at the Management Group or Root scope instead of Resource Group, causing unintended global access.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Azure Portal Access (Owner)

### Step 1: Assign Custom RBAC Role

```azurecli
# Assign Contributor role to user at Resource Group scope
az role assignment create --assignee "asmith@company.com" \
                          --role "Contributor" \
                          --resource-group "rg-prod-web"
```

> [!success] Expected Output
> ```
> Role assigned successfully.
> ```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `New-MgUser` | Creates Entra ID user | `New-MgUser -DisplayName...` |
| `az role assignment create` | Assigns RBAC | `az role assignment create ...` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| Dynamic Group not updating | Syntax error in query | Check rule syntax and wait for background evaluation |
| Helpdesk admin can't reset password | Admin scope limited by AU | Ensure user is within the target Administrative Unit |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Guest Access Needed

> [!example] Ticket
> "Need to invite an external contractor to our Azure AD."

**L1 Response:** Verify request approval.
**Escalation Trigger:** Requires B2B invitation and specific RBAC scoping.
**L2 Resolution:** Invite user via Entra ID B2B, assign Reader role to specific Resource Group.

---
## 🎤 Interview Questions

> [!question] Q1: What is an Administrative Unit?
> **Answer:** It's a container in Entra ID that lets you restrict the scope of administrative roles, like letting a Helpdesk Admin only reset passwords for users in London.

==**Exam Tip:** Dynamic groups require Entra ID Premium P1/P2 licenses.==

---
## 🔗 Related Notes

- [[04-Cloud-and-Security/08-Azure/AZ9-02 Azure Global Infrastructure|AZ9-02 Azure Global Infrastructure]]
- [[04-Cloud-and-Security/08-Azure/AZ9-05 Azure Identity and Security|AZ9-05 Azure Identity and Security]]
