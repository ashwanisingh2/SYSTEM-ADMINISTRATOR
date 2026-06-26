---
tags: [azure, identity, governance, entra-id, rbac, policies, az-104]
aliases: [Azure AD, Entra ID, Azure Governance]
created: 2026-06-26
status: #complete
difficulty: #intermediate
cert-relevant: #az-104
---

> [!NOTE|color-blue]
> ☁️ **AZURE / CLOUD**

`#complete` `#intermediate` `#az-104`

# AZ104-01: Azure Identity and Governance

> [!abstract] Overview
> Yeh note Azure Identity (Microsoft Entra ID, pehle Azure AD) aur Azure Governance (RBAC, Policies, Tags, Resource Locks) ke baare mein hai. Ek cloud system admin ke liye yeh janna bahut zaroori hai ki resources ko securely manage kaise karein aur cost ya compliance ko strict control mein kaise rakhein.

---
## 🧠 Concept Overview

- **What it is** — Azure Identity deals with **who** you are (Authentication) and **what** you can do across Microsoft services. Azure Governance deals with ensuring your actual Azure resources comply with corporate standards, security baselines, and cost limits.
- **Why it matters** — Real job mein, unauthorized access rokna aur cloud bill ko control mein rakhna sabse badi priority hoti hai. *Agar governance nahi hogi, toh koi bhi junior admin galti se expensive GPU VMs bana dega aur bill lakho mein aayega.*
- **Where you see this** — New employee onboarding, resource deployment restrictions, enforcing MFA (Multi-Factor Authentication), password resets, and granting granular access to production databases.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic tasks — Password resets, user creation, adding users to security groups, tracking basic login failures. *Basic access issues check karna aur logs dekhna.* |
| **L2** | Configure, fix, escalate — RBAC roles assign karna, conditional access policies troubleshoot karna, Resource Locks lagana, PIM requests approve karna. |
| **L3** | Architecture, design — Designing enterprise-wide Management Groups, authoring custom Azure Policies, setting up Entra ID Connect hybrid architectures, and Identity Protection. |

> [!tip] Seedha Simple Mein
> *Identity ka matlab hai 'Kaun hai ye aadmi aur iska ID card valid hai ya nahi?' aur Governance ka matlab hai 'Isko building mein kahan jaane ki permission hai aur kis rule ke hisaab se?'.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Azure Environment** is like a **Highly Secure Corporate Office Building** because...
>
> - **Entra ID (Identity)** is the security guard at the main gate checking your company ID card before you enter (Authentication).
> - **RBAC (Role-Based Access Control)** is your smart access card deciding which floors, labs, or server rooms you can open (Authorization).
> - **Azure Policy (Governance)** is the strict building rulebook saying "No smoking inside" or "All visitors must wear badges" (Rules & Compliance).
> - **Resource Locks** are the locked glass cabinets housing critical switches that even floor managers cannot open without requesting a special master key (Preventing accidental deletion or modification).
> - **Tags** are the asset stickers on the back of monitors saying "Owned by HR Dept" (Cost allocation and billing).

---
## 🔬 Technical Deep Dive

### 1. Microsoft Entra ID (formerly Azure Active Directory)

> [!info] Key Concept
> Entra ID is Microsoft's cloud-based identity and access management (IAM) service. It helps employees sign in and access internal/external resources securely.

- **Users and Groups:** You can create cloud-only users directly in Azure, or sync them from your local on-premises Windows Server Active Directory using **Entra ID Connect**.
- **MFA (Multi-Factor Authentication):** *Password ke alawa ek aur security layer (e.g., OTP on phone, Microsoft Authenticator app).* Yeh phishing attacks se bachata hai.
- **Conditional Access:** "If-Then" statements for security. *Agar user office network se login kar raha hai, toh allow karo. Agar unknown location se hai, toh MFA mango. Agar risky IP se hai, toh block kardo.*
- **Entra ID Roles vs. Azure Roles:** Entra ID roles (like Global Admin, User Admin) manage directory resources (users, groups, billing). Azure roles (like Owner, Contributor) manage Azure resources (VMs, networks).

> [!danger] Common Mistake
> Giving the `Global Administrator` role to everyone in the IT team. *Sirf 2-3 senior logo ke paas Global Admin hona chahiye, baaki sab ko strictly least privilege principle ke hisaab se access dena chahiye.*

### 2. Azure RBAC (Role-Based Access Control)

> [!info] Key Concept
> RBAC manages who has access to Azure resources, what they can do with those resources, and exactly what areas they have access to.

RBAC uses three main components:
1. **Security Principal:** Who gets the access? (This can be a User, a Security Group, a Service Principal, or a Managed Identity).
2. **Role Definition:** What can they do? (Owner, Contributor, Reader, or Custom Roles).
3. **Scope:** Where does the access apply? (Hierarchy: Management Group -> Subscription -> Resource Group -> Specific Resource).

*Difference between Owner and Contributor:* An Owner can do everything PLUS manage access (assign roles to other users). A Contributor can create and manage resources but CANNOT manage access for others.

### 3. Azure Governance (Policies, Locks, and Tags)

#### Azure Policy
Azure Policy evaluates your resources for non-compliance against specific rules. It can audit, deny, or automatically remediate resources.
*Example: Tum rule bana sakte ho ki koi bhi Virtual Machine sirf 'Central India' ya 'South India' region mein banega, baaki kahin nahi. Agar koi US region mein banayega, toh Azure usko block kar dega.*

#### Resource Locks
Resource Locks prevent accidental deletion or modification of resources, regardless of RBAC permissions.
- **CanNotDelete:** Authorized users can read and modify the resource, but they cannot delete it.
- **ReadOnly:** Authorized users can only read the resource. They cannot modify or delete it. *Yeh critical production databases ya network hubs pe zaroor lagate hain taaki koi configuration change na kar de.*

#### Tags
Tags are simple name/value pairs applied to resources to logically organize them.
*Billing department ko pata chalta hai ki kaunsa resource kis department (HR, IT, Finance) ka hai, taaki cost track ho sake aur chargeback model work kare.*

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - An active Azure Subscription
> - Sufficient permissions (User Access Administrator or Owner at the Resource Group level)
> - Azure CLI installed locally or using Azure Cloud Shell

### Step 1: Create a Resource Group and Apply a Lock

We will create a Resource Group and apply a `CanNotDelete` lock to prevent accidental deletion by any admin.

```bash
# Yeh command naya resource group banata hai 'Central India' region mein
az group create --name "rg-prod-app-01" --location "centralindia"

# Yeh command us resource group pe Delete lock lagata hai
az lock create --name "LockProdRG" --lock-type CanNotDelete --resource-group "rg-prod-app-01"
```

> [!success] Expected Output
> ```json
> {
>   "id": "/subscriptions/xxx/resourceGroups/rg-prod-app-01/providers/Microsoft.Authorization/locks/LockProdRG",
>   "level": "CanNotDelete",
>   "name": "LockProdRG",
>   "type": "Microsoft.Authorization/locks"
> }
> ```

### Step 2: Assign an RBAC Role

Now we will assign the `Reader` role to a user on this newly created Resource Group.

```bash
# Yeh command user ko Reader role assign karta hai specific resource group pe
az role assignment create \
  --assignee "user@yourdomain.com" \
  --role "Reader" \
  --resource-group "rg-prod-app-01"
```

> [!success] Expected Output
> ```json
> {
>   "principalId": "xxxx-xxxx-xxxx-xxxx",
>   "principalType": "User",
>   "roleDefinitionId": "/subscriptions/xxx/providers/Microsoft.Authorization/roleDefinitions/xxx",
>   "scope": "/subscriptions/xxx/resourceGroups/rg-prod-app-01"
> }
> ```

### Step 3: Test Resource Deletion (Lock Verification)

Let's try to delete the resource group to see if our lock works.

```bash
# Resource group ko delete karne ki koshish karte hain
az group delete --name "rg-prod-app-01" --yes
```

> [!bug] Expected Error
> The client 'user@yourdomain.com' with object id 'xxx' has permission to perform action 'Microsoft.Resources/subscriptions/resourceGroups/delete' on scope '/subscriptions/xxx/resourceGroups/rg-prod-app-01'; however, the access is denied because of the deny assignment with name 'LockProdRG' and Id 'xxx'.

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `az ad user create` | Entra ID mein naya user banata hai | `az ad user create --display-name "John Doe" --user-principal-name john@domain.com --password "P@ssw0rd123"` |
| `az ad group create` | Naya security group banata hai | `az ad group create --display-name "IT-Admins" --mail-nickname "itadmins"` |
| `az group create` | Naya Resource Group banata hai | `az group create --name "MyRG" --location "eastus"` |
| `az role assignment create` | Kisi user ko RBAC role assign karta hai | `az role assignment create --assignee user@domain.com --role "Contributor" -g "MyRG"` |
| `az lock create` | Resource pe lock lagata hai taaki delete na ho | `az lock create --name "NoDelete" --lock-type CanNotDelete -g "MyRG"` |
| `az policy assignment create` | Policy rule apply karta hai (jaise region restrict karna) | `az policy assignment create --policy "policy-id" -g "MyRG"` |
| `az tag create` | Resource pe tag lagata hai | `az tag create --resource-id "/subscriptions/xxx/resourceGroups/MyRG" --tags Dept=IT` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| User cannot create a VM in Resource Group | User ke paas sirf 'Reader' role hai, Contributor nahi | RBAC (IAM tab) mein jaakar user ko **Contributor** ya **Virtual Machine Contributor** role assign karein. |
| Cannot delete a Resource Group or VM | Resource ya Resource Group pe `CanNotDelete` lock laga hua hai | Pehle lock ko delete karein portal se ya CLI se (`az lock delete`), uske baad resource ko delete karein. |
| VM deployment fails with "PolicyViolation" error | Azure Policy ne us specific VM size, OS, ya region ko block kiya hua hai | Error details mein Azure Policy check karein aur allowed size/region use karein, ya admin se policy ko modify karwayein. |
| User cannot log in to Azure Portal or O365 | Conditional Access policy block kar rahi hai ya password expire ho gaya hai | Entra ID sign-in logs check karein. Agar location blocked hai, toh VPN disconnect karke try karein. Agar password issue hai, toh reset karein. |
| Cannot assign an RBAC role to someone else | User is a 'Contributor' but not an 'Owner' or 'User Access Administrator' | Role assignment permissions ke liye user ko **Owner** role ya **Role Based Access Control Administrator** role chahiye hota hai. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Accidental Deletion Prevention

> [!example] Ticket
> "We had a Sev-1 incident where a junior admin accidentally deleted a production SQL database. We need to ensure this doesn't happen again to any production resource."

**L1 Response:** Check if the resource is currently running and verify which user performed the action via Azure Activity Logs.
**Escalation Trigger:** Pass to L2 if an enterprise-wide lock strategy needs to be implemented.
**L2 Resolution:** Apply a `CanNotDelete` Resource Lock at the Subscription level or on specific high-value Resource Groups for all production workloads. *Production environment mein humesha Delete lock lagana industry standard practice hai.*

### 🎫 Scenario 2: Access Denied to Resource Group

> [!example] Ticket
> "I am a new backend developer, and I can see the Resource Group 'rg-dev-backend', but the 'Create' button for making a new Azure Storage Account is greyed out. Please fix my access."

**L1 Response:** Azure Portal mein Resource Group level pe IAM (Access Control) tab mein user ka "Check Access" use karein. *Most likely user ke paas sirf Reader access hoga.*
**Escalation Trigger:** If the group owner needs to formally approve the access elevation.
**L2 Resolution:** Escalate via PIM (Privileged Identity Management) for just-in-time access, or manually assign the `Contributor` role to the developer on that specific Resource Group after getting their manager's documented approval.

### 🎫 Scenario 3: Restricting VM Deployments to Save Costs

> [!example] Ticket
> "To save costs this quarter, IT management wants to ensure that developers in the Dev Subscription can only deploy B-Series (Basic/Burstable) Virtual Machines in the 'Central India' region."

**L1 Response:** Understand the exact requirement, note the allowed SKUs and locations, and confirm the scope (Dev subscription).
**Escalation Trigger:** Pass to L2/L3 cloud engineers to author and assign the policy.
**L2 Resolution:** Create an **Azure Policy** using the built-in definitions "Allowed virtual machine size SKUs" and "Allowed locations". Set the parameters to only allow B-series and Central India. Assign this policy to the Developer Subscription scope. *Azure Policy compliance ensure karne ke liye best preventive tool hai.*

---
## 🎤 Interview Questions

> [!question] Q1: What is the primary difference between Azure AD (Entra ID) roles and Azure RBAC roles?
> **Answer:** Entra ID roles (like Global Administrator) handle authentication and management of the identity directory itself, as well as access to Microsoft 365 applications. Azure RBAC roles (like Owner or Contributor) handle authorization specifically for Azure infrastructure resources (like VMs, Storage Accounts, VNETs).

==**Exam Tip:** RBAC is for Azure *resources*. Entra ID is for *identity* and directory-level configurations.==

> [!question] Q2: If a user has the 'Reader' role at the Subscription level, but the 'Contributor' role at the Resource Group level, what permissions do they have on the resources inside that Resource Group?
> **Answer:** They will have full Contributor permissions on that specific Resource Group and its contents. Azure RBAC is an additive model, and permissions flow down the hierarchy. The most permissive role at the specific scope applies.

==**Exam Tip:** Role assignments are inherited from top to bottom (Management Group -> Subscription -> Resource Group -> Resource). They do not flow upwards.==

> [!question] Q3: How can you prevent a Global Administrator or a Subscription Owner from accidentally deleting a critical production Virtual Network?
> **Answer:** By applying a `CanNotDelete` Resource Lock directly on the Virtual Network or its parent Resource Group. Even a Global Admin or an Owner cannot delete the resource until they explicitly remove the lock first.

==**Exam Tip:** Resource Locks override ALL RBAC permissions. A lock applies to EVERYONE, regardless of their role.==

> [!question] Q4: A company wants to group multiple subscriptions together to apply a single set of security policies and RBAC roles across all of them efficiently. What feature should they use?
> **Answer:** Management Groups. Management Groups allow you to organize multiple subscriptions into a hierarchical structure and apply governance conditions (Azure Policies, RBAC) that all child subscriptions automatically inherit.

==**Exam Tip:** The hierarchy is: Root Management Group -> Custom Management Groups -> Subscriptions -> Resource Groups -> Resources.==

> [!question] Q5: You want to ensure that all resources deployed in a specific Resource Group automatically have a tag `Environment: Production`. What is the best way to enforce this?
> **Answer:** Use Azure Policy. You can assign a built-in Azure Policy like "Append a tag and its value to resources" at the Resource Group scope. When anyone deploys a new resource, Azure will automatically append the `Environment: Production` tag to it.

==**Exam Tip:** Azure Policy can do more than just audit or deny; it can *Modify* or *Append* configurations (like tags) during deployment.==

---
## 🔗 Related Notes

- [[AZ104-02 Virtual Networking|AZ104-02 Virtual Networking]] — Networking concept after Identity
- [[Windows Server Active Directory|Windows Server AD]] — On-premises Identity fundamentals
- [[Azure PowerShell Basics|Azure PowerShell]] — For managing Azure via CLI scripts
- [[Azure Cost Management|Azure Cost Management]] — Tracking billing based on Tags
