---
tags: [sysadmin, azure, az-900, governance, cost-management]
difficulty: Beginner
lab-required: Yes
read-time: 10 mins
---

# AZ9-06: Azure Cost and Governance

> [!abstract] Overview
> This note covers cost estimation, billing controls, and architectural governance in Microsoft Azure. It details the Pricing/TCO Calculators, Cost Management budgets, Azure Advisor, Azure Policy, Resource Locks, and SLA calculations.

---
## Concept
Think of Azure governance and cost controls like managing a corporate credit card and policy handbook for a company:
- **Pricing Calculator** is checking the menu prices before you order a catered meal to estimate the bill.
- **TCO (Total Cost of Ownership) Calculator** is comparing the cost of cooking meals at home (buying pots, stoves, paying utility bills - On-Prem) vs. ordering takeout every day (Cloud).
- **Cost Management Budgets** are the credit card spending limits: if a developer spends $80\%$ of their limit, the system alerts the manager automatically.
- **Azure Policy** is the corporate rulebook: it prevents employees from ordering expensive lobster (deploying high-cost VM sizes) or ordering from unapproved cities (deploying resources in unapproved regions).
- **Resource Locks** are the safety locks on critical vaults: they prevent someone from accidentally deleting the main storage safe.

*Seedha simple mein: Azure Cost Management budget limits aur spending alerts setup karta hai. Azure Policy resources ke rules and boundaries enforce karti hai. Resource Locks (Delete, ReadOnly) critical files ko accidental deletion se protect karte hain.*

---
## Technical Deep Dive

### 1. Cost Estimation Tools
- **Azure Pricing Calculator:** A web tool used to estimate the monthly consumption costs of Azure resources before they are deployed. You configure specific service sizes, regions, storage capacity, and support tiers to generate quotes.
- **TCO (Total Cost of Ownership) Calculator:** Used to compare the financial costs of running workloads in your on-premises datacenter (includes hardware, power, cooling, network bandwidth, IT labor, real estate) against migrating to Azure over a 1 to 5-year window.

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
  - *Example policy:* "Block deployment of any VM size except the B-series," or "Only allow deployments in the East US region." If a user attempts to create a resources that violates the policy, the deployment is blocked.
- **Management Groups:** Container structures used to apply policies and RBAC roles across multiple subscriptions simultaneously.

### 5. Resource Locks
Locks prevent accidental modifications or deletions of critical resources. Applied at the Subscription, Resource Group, or individual Resource level.
- **Delete Lock (CanNotDelete):** Users can read and modify the resource, but cannot delete it.
- **Read-Only Lock (ReadOnly):** Users can only read the resource settings; they cannot modify or delete it (overrides all Contributor permissions).

### 6. SLA (Service Level Agreement) Calculations
SLAs define Microsoft's commitment to service uptime. Down time is calculated monthly:
- **$99.9\%$ SLA:** $\approx 43.8$ minutes downtime allowed per month.
- **$99.95\%$ SLA:** $\approx 21.9$ minutes downtime allowed per month.
- **$99.99\%$ SLA:** $\approx 4.38$ minutes downtime allowed per month.
- **Composite SLA:** If an application uses an App Service ($99.95\%$) and an Azure SQL Database ($99.99\%$), the composite SLA is:
  $$\text{Composite SLA} = 0.9995 \times 0.9999 = 0.9994 \implies \mathbf{99.94\%}$$
  - *Rule:* The composite SLA is always **lower** than the individual SLAs because adding components increases potential failure points.

---
## Lab — Step by Step
> [!info] Lab Setup Needed
> Access to the Azure Portal.

### Step 1: Apply a Delete Lock to a Resource Group
1. Log into the Azure Portal.
2. Go to **Resource groups** -> Select `rg-lab-infrastructure`.
3. In the left panel under Settings, click **Locks**. Click **+ Add**.
4. Configure lock parameters:
   - Lock name: `Prevent_Accidental_Delete`
   - Lock type: **Delete**
   - Notes: "Must remove lock before deleting this lab resource group."
5. Click OK.

### Step 2: Test the Lock Protection
1. Go to the Overview page of `rg-lab-infrastructure`.
2. Click **Delete resource group**.
3. Type the resource group name to confirm, and click **Delete**.
4. **Verify:** The deletion will fail, returning an error notification stating that the resource group is locked.

### Step 3: Clean up (Remove Lock)
1. Go back to Locks tab of the resource group.
2. Click the **Delete** icon next to the `Prevent_Accidental_Delete` lock to remove it.
3. You can now delete the resource group to clean up your lab.

---
## Related Notes
- [[04-Cloud-and-Security/08-Azure/AZ9-02 Azure Global Infrastructure|AZ9-02 Azure Global Infrastructure]] — Subscriptions and Management Group containers.
- [[04-Cloud-and-Security/08-Azure/AZ104-01 Azure Identity and Governance|AZ104-01 Azure Identity and Governance]] — Advanced Azure Policy implementation and governance.

