---
tags: [desktop-support, devops, iac, terraform, L2]
aliases: [terraform-basics, iac-basics]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #none
---

# DEV-03: Terraform Basics (Infrastructure as Code)

> [!note] Overview
> This note covers the fundamentals of Infrastructure as Code (IaC) using HashiCorp Terraform. It details the declarative syntax model, provider plugins, state file management, variables, outputs, and the standard initialization, planning, and deployment workflows.

---
## Concept Overview
- **What it is** — Terraform is an open-source Infrastructure as Code (IaC) tool. It enables administrators to provision, manage, and version-control physical and cloud infrastructure resources (virtual machines, networks, DNS records, databases) using a declarative configuration language called HashiCorp Configuration Language (HCL).
- **Why it matters** — Building cloud resources manually by clicking through the Azure Portal or AWS console is slow, error-prone, and undocumented. Terraform defines your entire datacenter in text files. This code can be version-controlled, reviewed via pull requests, and redeployed instantly, guaranteeing consistency across Dev, Test, and Prod environments.
- **Real job encounter** — Provisioning a new development network and VM cluster, checking for configuration drift, updating server sizes, and cleaning up test environments.
- **L1 vs L2 vs L3 responsibility split:**
  - **L1 Resolution**: Verify active Terraform states using CLI check commands, review planned execution logs (`terraform plan` printouts), and check system logs.
  - **Escalation Trigger**: Escalate if deployments fail due to state file locks, provider credential expirations, or code syntax errors.
  - **L2 Resolution**: Write standard `.tf` resource blocks to deploy infrastructure components, define variables, manage outputs, and run resource plans/deployments.
  - **L3 Resolution**: Implement remote backend storage (Azure Blob Storage / AWS S3) with lease locking, secure secrets using Key Vault integrations, design reusable Terraform Modules, and configure automated CI/CD deployment pipelines.

---
## Technical Deep Dive

### 1. Declarative IaC Paradigm
Terraform is **declarative**. You describe the *desired end-state* of your infrastructure, and Terraform handles the calculations to achieve it:
- **Imperative (How)**: Standard scripting (PowerShell/Bash). You write step-by-step instructions. E.g., "Check if VNet exists. If not, run command. Check if Subnet exists...".
- **Declarative (What)**: You declare the final state. E.g., "I want a VNet named VNet-01 with address space 10.0.0.0/16." Terraform queries the target cloud, compares it to your code, and only runs the necessary API calls to create it. If it already exists, Terraform does nothing.

### 2. Core Components of Terraform
- **Terraform CLI**: The local executable tool that parses your code, evaluates dependencies, and runs configurations.
- **Providers**: Plugins that translate Terraform HCL code into API calls for specific vendors (e.g., AzureRM, AWS, GCP, VMware, Active Directory).
- **State File (`terraform.tfstate`)**: A JSON file that maps the resources defined in your HCL code to real-world objects in your cloud subscription. **Protecting this file is critical**, as it contains plain-text passwords, keys, and IP addresses.

### 3. Core Operational Workflow
Every Terraform deployment follows four basic steps:

```
+-----------------------------------------------------------+
| 1. Write HCL Code (main.tf, variables.tf)                 |
+-----------------------------------------------------------+
                             |
+-----------------------------------------------------------+
| 2. terraform init (Downloads provider plugins)            |
+-----------------------------------------------------------+
                             |
+-----------------------------------------------------------+
| 3. terraform plan (Previews changes to be made)           |
+-----------------------------------------------------------+
                             |
+-----------------------------------------------------------+
| 4. terraform apply (Executes API calls and updates state) |
+-----------------------------------------------------------+
```

---
## Step-by-Step Lab

> [!warning] Pre-requisites
> - An Azure Subscription.
> - Azure CLI installed and authenticated (`az login`).
> - Terraform CLI installed on your system.
> - Access to a shell/terminal window.

### Step 1: Create a Workspace Directory
1. Open a terminal and create a folder for the lab:
```bash
mkdir ~/terraform-azure-lab && cd ~/terraform-azure-lab
```

### Step 2: Configure the Provider Configuration File
We will define which cloud provider Terraform must communicate with.

1. Create a file named `providers.tf`:
```hcl
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
}

provider "azurerm" {
  features {}
}
```

### Step 3: Define Infrastructure Resources
We will write code to deploy an Azure Resource Group and a Virtual Network.

1. Create a file named `main.tf`:
```hcl
# Create Resource Group
resource "azurerm_resource_group" "lab_rg" {
  name     = "RG-Terraform-Lab"
  location = "EastUS"
}

# Create Virtual Network inside the Resource Group
resource "azurerm_virtual_network" "lab_vnet" {
  name                = "VNET-Prod-01"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.lab_rg.location
  resource_group_name = azurerm_resource_group.lab_rg.name
}
```

### Step 4: Initialize the Directory
1. Run initialization to download the AzureRM provider plugin:
```bash
terraform init
```
**Expected Output:** Downloads plugins, displaying: `Terraform has been successfully initialized!`

### Step 5: Preview and Apply the Changes
1. Preview the planned changes:
```bash
terraform plan
```
**Expected Output:** Shows `Plan: 2 to add, 0 to change, 0 to destroy.`

2. Apply the changes to Azure:
```bash
terraform apply -auto-approve
```
**Expected Output:** Executes API calls. Ends with:
`Apply complete! Resources: 2 added, 0 changed, 0 destroyed.`
*(You can verify in the Azure portal that the resource group and VNet exist).*

### Step 6: Clean Up (Destroy)
1. Delete all resources created in this lab to avoid charges:
```bash
terraform destroy -auto-approve
```
**Expected Output:** Deletes VNet and Resource Group. Ends with:
`Destroy complete! Resources: 2 destroyed.`

---
## Cheat Sheet / Quick Reference

| Command / Option | Purpose | Example |
|---|---|---|
| `terraform init` | Initializes directories and downloads providers | `terraform init` |
| `terraform plan` | Previews planned changes against current state | `terraform plan` |
| `terraform apply` | Applies configurations to the target environment | `terraform apply -auto-approve` |
| `terraform destroy` | Deletes all resources managed by the state file | `terraform destroy -auto-approve` |
| `terraform validate` | Validates HCL code syntax correctness | `terraform validate` |
| `terraform fmt` | Rewrites config files to match standard spacing | `terraform fmt` |
| `terraform show` | Prints human-readable details of current state file | `terraform show` |
| `terraform state list` | Lists all resources tracked in the state file | `terraform state list` |

---
## Troubleshooting Table

| Problem | Cause | Fix |
|---|---|---|
| `terraform init` fails with: "Error querying database/provider..." | Network block prevents access to HashiCorp Registry, or provider version is invalid. | Check internet connection. Verify provider block syntax and version mapping in `providers.tf`. |
| `terraform apply` fails: "Error: ResourceAlreadyExists." | A resource with the same name already exists in the cloud, but is not tracked in the state file. | Delete the existing resource in the cloud, or import it into the state file: `terraform import <resource_type>.<name> <resource_id>`. |
| Error: "Resource group not found" during deployment. | Dependency mismatch; Terraform tried to create the VNet before the Resource Group was created. | Use references (e.g. `resource_group_name = azurerm_resource_group.rg.name`) to build implicit dependencies, forcing sequential creation. |
| Error: "State file is locked." | Another process or administrator is currently running a terraform apply on the same state file. | Wait for the other process to finish. If a run crashed, unlock manually: `terraform force-unlock <lock_id>` (requires caution). |
| Outbound changes are not detected after manual edits in the cloud portal. | The local state file is out of sync with real-world infrastructure. | Run `terraform refresh` (or run `terraform plan` which automatically runs a refresh) to update the state file metadata. |

---
## Interview Questions

> [!question] L1 Question
> **Q:** What is the purpose of the `terraform.tfstate` file, and why should you never modify it manually?
> **A:** The `terraform.tfstate` file is a JSON file that acts as Terraform's memory. It maps the resources defined in your HCL code to the actual physical assets in your cloud provider. You should never modify it manually because any structural formatting error or ID mismatch will corrupt the mapping database, causing Terraform to lose track of resources, deploy duplicates, or delete production assets.

> [!question] L2 Question
> **Q:** What is the difference between `terraform plan` and `terraform apply` commands?
> **A:** `terraform plan` queries the target cloud provider, compares the active state with your HCL code, and previews the exact additions, deletions, or modifications that will be made, without writing any changes. `terraform apply` compiles the plan and executes the actual API calls to provision the resources, updating the state file upon completion.

> [!question] L3/Scenario Question
> **Q:** You are setting up a Terraform pipeline for a team of 5 cloud engineers who need to manage the same Azure subscription. If they run deployments locally, they overwrite each other's state files. How do you design a secure, collaborative environment?
> **A:** 
> - **Situation:** Multi-engineer team conflicts over local state file management.
> - **Task:** Design a secure, shared state architecture with locking.
> - **Action:** 
>   1. **Implement Remote Backend**: Configure a remote backend in `providers.tf` pointing to an **Azure Blob Storage Account** (or AWS S3).
>   2. **Configure State Locking**: Configure the backend to use Azure Blob storage native leasing mechanisms. When an engineer runs `terraform apply`, Terraform locks the blob file.
>   3. **Secure the State**: Enable HTTPS, require TLS 1.2, and restrict storage account access using Azure RBAC roles.
>   4. **CI/CD Integration**: Move the execution pipeline to a CI/CD platform (like GitHub Actions or Azure Pipelines) where Terraform runs under a service connection, eliminating local deployments.
> - **Result:** State files are centralized, modifications are serialized via locking, and credentials are protected.

---
## Seedha Simple Mein
*Seedha simple mein: Terraform ek Infrastructure as Code (IaC) tool hai jo cloud platforms (Azure, AWS) par servers, networks aur database setups ko ek program code ke zariye deploy karne ki facility deta hai. Manual configuration ke opposite, isme script (`.tf` file) compile karke `terraform apply` run kiya jata hai jo automatically resources provision kar deta hai.*

---
## Related Notes
- [[04-Cloud-and-Security/08-Azure/AZ104-04 Azure Virtual Networking|AZ104-04 Azure Virtual Networking]] — Networking targets managed by Terraform.
- [[05-Automation-and-Ticketing/14-DevOps-Basics/DEV-01 Docker-Fundamentals|DEV-01 Docker-Fundamentals]] — Local container testing compared to cloud provisioning.
- [[05-Automation-and-Ticketing/12-Ansible/ANS-01 Ansible for Windows Admins|ANS-01 Ansible for Windows Admins]] — Comparison of IaC tools vs configuration managers.

---
*Tags: #desktop-support #devops #iac #terraform #L2*
