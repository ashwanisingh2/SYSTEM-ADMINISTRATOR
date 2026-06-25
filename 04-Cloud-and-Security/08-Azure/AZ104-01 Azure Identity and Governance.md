---
tags: [sysadmin, azure, az-104, identity, governance]
difficulty: Advanced
lab-required: Yes
read-time: 15 mins
---

# AZ104-01: Azure Identity and Governance

> [!abstract] Overview
> This note covers advanced Azure identity management and resource governance. It details Entra ID profiles (B2B, B2C), Dynamic Group membership rules, Administrative Units, custom RBAC definitions, policy initiatives, and management group hierarchies.

---
## Concept
Think of Azure Identity and Governance like managing access and rules in a massive multi-tenant skyscraper:
- **Entra ID Tenant** is the entire building dedicated to your company.
- **B2B (Business-to-Business)** is giving a temporary contractor badge to a guest from another company (they use their own company's credentials to log in, but can enter your rooms).
- **B2C (Business-to-Consumer)** is setting up a public restaurant on the ground floor: customers sign up using their personal accounts (Google, Facebook) to buy food, completely isolated from corporate office floors.
- **Dynamic Groups** are automatic door permissions: "If your file says your department is Sales (User attribute), you are automatically assigned to the Sales Group and given access cards."
- **Administrative Units (AUs)** are branch management desks: instead of making someone a Global Manager of the entire building, you make them the Branch Manager of the 4th Floor only (scoping admin permissions).

*Seedha simple mein: AZ-104 level par identity governance secure RBAC scopes (Subscription to Resource) aur dynamic groups par chalta hai. Administrative Units helpdesk teams ko localized control dete hain bina global access diye.*

---
## Technical Deep Dive

### 1. Entra ID Profiles: B2B vs. B2C
- **Entra ID B2B (Collaboration):** Allows you to invite guest users from other organizations to access your internal resources (teams, folders, apps). Authentication is handled by the guest's home identity provider (e.g., their own Entra ID or Google).
- **Entra ID B2C (Customer Identity):** A separate directory infrastructure designed for customer-facing web applications. Allows users to sign up and create profiles using personal emails or social logins (Facebook, Google, Microsoft accounts). Completely isolated from your internal corporate directory.

### 2. Assigned vs. Dynamic Groups
- **Assigned Groups:** Members are added manually by administrators or owners.
- **Dynamic Groups (Premium P1/P2):** Members are added or removed automatically based on user attributes matching defined rules.
  - *Dynamic Group Rule Syntax (Advanced query):*
    `user.department -eq "Sales" -and user.jobTitle -contains "Manager"`
    - If a user's department changes to "Marketing," the Entra ID engine automatically removes them from the group within minutes.

### 3. Administrative Units (AUs)
AUs partition your Entra ID directory into smaller logical containers.
- **Use Case:** A global organization has offices in London and New York. Instead of making a local London support tech a Helpdesk Administrator for the entire tenant, you create a "London_AU" container, place London users inside, and delegate Helpdesk Admin rights scoped *only* to that AU. The tech cannot reset passwords for New York users.

### 4. RBAC: Custom Roles and Scopes
If built-in roles (Owner, Contributor, Reader) do not fit requirements, create a **Custom RBAC Role** using JSON:

```json
{
  "Name": "Virtual Machine Operator",
  "Description": "Can start, stop, and restart virtual machines.",
  "Actions": [
    "Microsoft.Compute/*/read",
    "Microsoft.Compute/virtualMachines/start/action",
    "Microsoft.Compute/virtualMachines/restart/action"
  ],
  "NotActions": [],
  "AssignableScopes": [
    "/subscriptions/xxxx-xxxx-xxxx-xxxx"
  ]
}
```
- **Scope Levels:** Roles are inherited downward:
  `Management Group > Subscription > Resource Group > Resource`
  - If you assign Contributor at the Subscription scope, the user has Contributor rights on every Resource Group and Resource inside that subscription.

### 5. Azure Policy Initiatives & Remediation
- **Policy Definition:** A single compliance rule (e.g., "MFA must be enabled").
- **Initiative Definition:** A collection of multiple related policies grouped together (e.g., "PCI-DSS Compliance Initiative" containing 50 individual security policies).
- **Remediation Task:** When a policy evaluates a resource as non-compliant, a remediation task can run automatically to fix the configuration (e.g., installing missing monitoring agents using DeployIfNotExist).

---
## PowerShell Administration Commands

### Creating Users via PowerShell
```powershell
# Authenticate to Entra ID
Connect-MgGraph

# Create a new user account
$PasswordProfile = @{ Password = "UserP@ssword123!" }
New-MgUser -DisplayName "Alice Smith" `
           -UserPrincipalName "asmith@company.onmicrosoft.com" `
           -MailNickName "asmith" `
           -AccountEnabled `
           -PasswordProfile $PasswordProfile
```

### Assigning RBAC Roles via CLI
```azurecli
# Assign Contributor role to user at Resource Group scope
az role assignment create --assignee "asmith@company.com" \
                          --role "Contributor" \
                          --resource-group "rg-prod-web"
```

---
## Lab — Step by Step
> [!info] Lab Setup Needed
> Access to the Azure Portal with Owner permissions on a subscription.

### Step 1: Create a Dynamic Group
1. In the Azure Portal, open **Microsoft Entra ID**.
2. Click **Groups** -> **New group**.
3. Configuration:
   - Group type: **Security**
   - Group name: `dyn-sales-employees`
   - Membership type: **Dynamic User**
4. Click **Add dynamic query**.
5. Build Rule:
   - Property: `department` | Operator: `Equals` | Value: `Sales`
6. Click **Save**, then click **Create**.

### Step 2: Create Custom RBAC Role
1. Search for and open your **Subscription**. Click **Access control (IAM)**.
2. Click **+ Add** -> **Add custom role**.
3. Custom role name: `Support Backup Reader`.
4. Go to the **Permissions** tab. Click **Add permissions**.
5. Search for `Microsoft.Storage/storageAccounts/read` and check it.
6. Click **Review + create** to publish the custom role.

### Step 3: Assign and Verify Custom Role
1. Go to resource group `rg-lab-infrastructure`.
2. Click **Access control (IAM)** -> **Add role assignment**.
3. Select your custom role `Support Backup Reader`. Click Next.
4. Members: Select a test user account. Click Next, then **Review + assign**.
5. The user can now view storage accounts within that resource group but has no other administrative rights.

---
## Related Notes
- [[04-Cloud-and-Security/08-Azure/AZ9-02 Azure Global Infrastructure|AZ9-02 Azure Global Infrastructure]] — Resource group containers and tenant models.
- [[04-Cloud-and-Security/08-Azure/AZ9-05 Azure Identity and Security|AZ9-05 Azure Identity and Security]] — Base RBAC roles and Entra ID basics.
- [[04-Cloud-and-Security/08-Azure/AZ104-02 Azure Storage Administration|AZ104-02 Azure Storage Administration]] — Granting permissions to storage keys.

