---
tags: [desktop-support, azure, cloud, L1]
aliases: [az9-06-azure-cost-and-governance, az9-06]
created: 2026-06-25
status: #complete
difficulty: #beginner
cert-relevant: #none
---

> [!NOTE|color-blue]
> ☁️ **AZURE CLOUD**

`#complete` `#beginner` `#none`

# AZ9-06: Azure Cost and Governance

> [!abstract] Overview
> Yeh note Azure mein cost estimation, billing controls, aur architectural governance ko cover karta hai. Ek support engineer ke liye yeh jaanna bahot zaroori hai taaki accidental high billing ko roka ja sake aur resources company ki policies ke hisaab se deploy ho sakein. Isme hum Pricing Calculators, Cost Management, Azure Advisor, Azure Policy, aur Resource Locks seekhenge.

---
## 🧠 Concept Overview

- **What it is** — Azure Governance resources aur unke cost ko control aur monitor karne ka ek system hai, jisse rules set kiye jaate hain.
- **Why it matters** — Cloud mein pay-as-you-go model hota hai; agar restrictions nahi lagaye, toh users expensive resources create kar sakte hain jisse company ko bahot bada bill aa sakta hai.
- **Where you see this** — Jab developers by mistake 10 high-end VMs create karne ki koshish karte hain, tab Policy unhe rokti hai. Ya phir jab hum budgets set karte hain taaki spend ek limit ke bahar na jaye.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Locks check karna, basic cost estimation calculators use karna, Azure Advisor ki recommendations report karna. |
| **L2** | Azure Policies aur Management Groups configure karna, Cost alerts aur budgets set karna. |
| **L3** | Enterprise-wide governance framework design karna, complex composite SLA calculations karna. |

> [!tip] Seedha Simple Mein
> *Azure Cost Management budget limits aur spending alerts setup karta hai. Azure Policy resources ke rules and boundaries enforce karti hai. Resource Locks (Delete, ReadOnly) critical files ko accidental deletion se protect karte hain.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Azure Governance** is like **managing a corporate credit card and policy handbook for a company** because...
>
> - **Pricing Calculator** menu prices check karne jaisa hai pehle order karne se.
> - **Cost Management Budgets** credit card limits hain, jisse manager ko alert milta hai agar limit cross hone wali ho.
> - **Azure Policy** company ka rulebook hai jo employee ko expensive ya unapproved items kharidne se rokti hai.
> - **Resource Locks** safe pe lage locks hain jo accidental delete hone se bachate hain.

---
## 🔬 Technical Deep Dive

### 1. Cost Estimation Tools

> [!info] Key Concept
> Estimate costs before deploying resources.

- **Azure Pricing Calculator:** A web tool used to estimate the monthly consumption costs of Azure resources before they are deployed. You configure specific service sizes, regions, storage capacity, and support tiers to generate quotes.
- **TCO (Total Cost of Ownership) Calculator:** Used to compare the financial costs of running workloads in your on-premises datacenter against migrating to Azure over a 1 to 5-year window.

### 2. Cost Management + Billing

- **Budgets:** Set spending limits based on cost or utilization.
- **Alerts:** Triggers emails or webhooks when spending reaches configured percentages (e.g., alert at $50\%, 80\%, 100\%$ of budget).

### 3. Azure Advisor

An automated optimization engine that evaluates your deployed resources and provides recommendations in five categories:
- **Cost:** Identifies underutilized VMs that can be shut down or resized to save money.
- **Security:** Evaluates compliance against security baselines (recommends MFA, NSG rules).
- **Reliability:** Recommends Availability Sets or zones to prevent outages.
- **Performance:** Identifies performance bottlenecks.
- **Operational Excellence:** Best practices for deployment.

### 4. Azure Policy & Management Groups

- **Azure Policy:** Enforces organizational standards and compliance rules on resources.
  - *Example policy:* "Block deployment of any VM size except the B-series," or "Only allow deployments in the East US region."
- **Management Groups:** Container structures used to apply policies and RBAC roles across multiple subscriptions simultaneously.

### 5. Resource Locks

Locks prevent accidental modifications or deletions of critical resources. Applied at the Subscription, Resource Group, or individual Resource level.

> [!danger] Common Mistake
> Forgetting that a lock applied to a Resource Group is inherited by all resources inside it.

- **Delete Lock (CanNotDelete):** Users can read and modify the resource, but cannot delete it.
- **Read-Only Lock (ReadOnly):** Users can only read the resource settings; they cannot modify or delete it (overrides all Contributor permissions).

### 6. SLA (Service Level Agreement) Calculations

SLAs define Microsoft's commitment to service uptime.

- **$99.9\%$ SLA:** $\approx 43.8$ minutes downtime allowed per month.
- **$99.95\%$ SLA:** $\approx 21.9$ minutes downtime allowed per month.
- **$99.99\%$ SLA:** $\approx 4.38$ minutes downtime allowed per month.
- **Composite SLA:** If an application uses an App Service ($99.95\%$) and an Azure SQL Database ($99.99\%$), the composite SLA is:
  $$\text{Composite SLA} = 0.9995 \times 0.9999 = 0.9994 \implies \mathbf{99.94\%}$$

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Access to the Azure Portal with Owner or User Access Administrator rights.
> - An existing Resource Group named `rg-lab-infrastructure`.

### Step 1: Apply a Delete Lock to a Resource Group

1. Log into the Azure Portal.
2. Go to **Resource groups** -> Select `rg-lab-infrastructure`.
3. In the left panel under Settings, click **Locks**. Click **+ Add**.
4. Configure lock parameters:
   - Lock name: `Prevent_Accidental_Delete`
   - Lock type: **Delete**
   - Notes: "Must remove lock before deleting this lab resource group."
5. Click OK.

```bash
# Alternative: Apply lock using Azure CLI
az lock create --name Prevent_Accidental_Delete --resource-group rg-lab-infrastructure --lock-type CanNotDelete
```

> [!success] Expected Output
> ```
> The lock is created and listed under the Locks section for the resource group.
> ```

### Step 2: Test the Lock Protection

1. Go to the Overview page of `rg-lab-infrastructure`.
2. Click **Delete resource group**.
3. Type the resource group name to confirm, and click **Delete**.

> [!success] Expected Output
> ```
> Deletion will fail, returning an error notification stating that the resource group is locked.
> ```

### Step 3: Clean up (Remove Lock)

1. Go back to Locks tab of the resource group.
2. Click the **Delete** icon next to the `Prevent_Accidental_Delete` lock to remove it.
3. You can now delete the resource group to clean up your lab.

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `az lock create` | Resource par naya lock lagata hai | `az lock create --name MyLock --lock-type CanNotDelete --resource-group MyRG` |
| `az lock list` | Sabhi locks ki list dikhata hai | `az lock list --resource-group MyRG` |
| `az lock delete` | Ek existing lock ko hata deta hai | `az lock delete --name MyLock --resource-group MyRG` |
| `az policy assignment create` | Nayi Azure policy assign karta hai | `az policy assignment create --policy <policy-id>` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| Cannot delete a Resource Group | Uspe Delete lock (CanNotDelete) apply hua hai | Pehle resource group settings me jaake lock ko remove karein, phir delete try karein |
| VM Deployment Blocked | Azure Policy restrict kar rahi hai (e.g. wrong region ya expensive size) | Policy rules check karein aur allowed region/size select karke dubara deploy karein |
| Cannot modify a resource settings | Us resource ya uske parent RG par ReadOnly lock laga hai | ReadOnly lock ko delete karein ya modify operations allow karne wali permissions check karein |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Accidental Deletion Blocked

> [!example] Ticket
> "User is unable to delete a stale storage account inside `rg-lab-infrastructure`. Getting an error related to scope lock."

**L1 Response:** Azure Portal mein Storage Account ya uske Resource Group ki 'Locks' setting check karein ki koi CanNotDelete lock toh nahi laga hai.
**Escalation Trigger:** Agar lock management ne compliance reasons ke liye lagaya hai aur user ko sach mein delete karna hai, toh ticket L2/Manager ko approve karne ke liye bhej dein.
**L2 Resolution:** Approval aane ke baad lock ko temporarily remove karein, storage account delete karein, aur zarurat padne par wapas baki resources ke liye lock laga dein.

---
## 🎤 Interview Questions

> [!question] Q1: What is the difference between Azure Policy and Role-Based Access Control (RBAC)?
> **Answer:** RBAC controls *who* can perform actions (e.g., Contributor vs. Reader), while Azure Policy controls *what* can be done, regardless of who is doing it (e.g., even an Owner cannot deploy a VM in an unapproved region if blocked by Policy).

> [!question] Q2: How do you calculate a Composite SLA?
> **Answer:** You multiply the individual SLAs of all the connected components. For example, $99.95\%$ and $99.99\%$ become $0.9995 \times 0.9999 = 99.94\%$.

==**Exam Tip:** The composite SLA is ALWAYS lower than the individual SLAs of the components, because combining multiple services increases the chance of failure.==

---
## 🔗 Related Notes

- [[04-Cloud-and-Security/08-Azure/AZ9-02 Azure Global Infrastructure|AZ9-02 Azure Global Infrastructure]] — Subscriptions and Management Group containers.
- [[04-Cloud-and-Security/08-Azure/AZ104-01 Azure Identity and Governance|AZ104-01 Azure Identity and Governance]] — Advanced Azure Policy implementation and governance.
