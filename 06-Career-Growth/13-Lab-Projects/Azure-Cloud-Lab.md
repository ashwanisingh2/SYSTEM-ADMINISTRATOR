---
tags: [desktop-support, lab-setup, azure, cloud, networking, entra-id, L1, L2, L3]
aliases: [azure-lab, cloud-lab-setup]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #az-104, #az-900
---

# Azure Cloud Lab Setup Guide

---

## Concept Overview
- **What it is**: A hands-on deployment guide for setting up a Microsoft Azure sandbox environment, configuring cloud networking, deploying VMs, and managing cloud identities in Entra ID.
- **Why it matters**: Hybrid and cloud infrastructure management are critical skills for L2 and L3 engineers. Understanding cloud resource groups, virtual networks, network security groups, and Entra security baselines is essential.
- **Where you encounter this in real job**: Provisioning cloud VMs, troubleshooting remote access ports, configuring cloud user MFA, and managing subscription cost-containment locks.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Creates Entra ID users, assigns licenses, resets passwords, and monitors basic VM status in the Azure portal.
  - **L2**: Builds Virtual Networks (VNets), configures Network Security Groups (NSGs), configures backup rules, and deploys standard VM sizes.
  - **L3**: Configures site-to-site VPN tunnels, implements conditional access policies, automates resource builds via ARM/Bicep, and manages cost centers.

---

## Technical Deep Dive

### Azure Lab Network Architecture
In this cloud lab, you will build a secure virtual network containing a Windows Server virtual machine, protected by a Network Security Group (NSG) firewall:
```
                              [Internet]
                                  |
                                  v  (Port 3389 Allowed from your Home IP)
                        [Virtual Network: VNet-Lab]
                                  |
                   [Subnet: Subnet-Core (10.0.1.0/24)]
                                  |
                    [Network Security Group: NSG-Lab]
                                  |
                    +-------------+-------------+
                    |                           |
               [VM-WinServer]             [VM-Linux]
             (AD Domain Server)       (Web Hosting Node)
             IP: 10.0.1.4             IP: 10.0.1.5
```

### Resource Organization & Control
- **Azure Resource Manager (ARM)**: The management framework that handles deployment, organization, and security of all Azure resources.
- **VNet (Virtual Network)**: The fundamental block for your private network in Azure, allowing VMs to communicate securely with each other, the internet, and on-premises networks.
- **NSG (Network Security Group)**: A stateful firewall containing a list of security rules that allow or deny inbound/outbound network traffic based on source/destination IP, port, and protocol.

---

## Commands & Syntax

### PowerShell: Deploying Azure Resources via Az Module
Ensure you have the `Az` module installed (`Install-Module -Name Az`). Run these commands to build the lab:
```powershell
# 1. Log in to your Azure Account
Connect-AzAccount

# 2. Create the Resource Group
$RgName = "RG-Cloud-Lab"
$Location = "EastUS"
New-AzResourceGroup -Name $RgName -Location $Location

# 3. Create the Virtual Network and Subnet
$Subnet = New-AzVirtualNetworkSubnetConfig -Name "Subnet-Core" -AddressPrefix "10.0.1.0/24"
$VNet = New-AzVirtualNetwork -Name "VNet-Lab" -ResourceGroupName $RgName -Location $Location -AddressPrefix "10.0.0.0/16" -Subnet $Subnet

# 4. Deploy a Windows Server VM inside the subnet
New-AzVM -ResourceGroupName $RgName -Name "VM-WinServer" -Location $Location -VirtualNetworkName "VNet-Lab" -SubnetName "Subnet-Core" -Credential (Get-Credential) -OpenPorts 3389
```

### Azure CLI
```bash
# Log in and create a Linux Virtual Machine
az login
az group create --name "RG-Cloud-Lab" --location "eastus"
az vm create \
  --resource-group "RG-Cloud-Lab" \
  --name "VM-Linux" \
  --image "Ubuntu2204" \
  --admin-username "azureuser" \
  --generate-ssh-keys \
  --vnet-name "VNet-Lab" \
  --subnet "Subnet-Core"
```

### GUI Path
> Azure Portal -> Create a resource -> Virtual Machine
> Azure Portal -> Microsoft Entra ID -> Users -> New user
> Azure Portal -> Virtual Networks -> select VNet -> Subnets -> Associate NSG

---

## Real-World Scenarios

### Scenario 1: VM Connection Timeout via RDP Block
**User Complaint**: You deployed a Windows VM in Azure for a developer, but they cannot connect to it via Remote Desktop (RDP). The connection attempts time out.
**Your First 3 Checks**:
1. Check if the VM status is marked as Running in the Azure Portal.
2. Check the public IP address assigned to the virtual machine.
3. Review the inbound rules in the Network Security Group associated with the VM's network interface.
**Diagnosis Steps**:
1. Verify the VM status: VM is running, and has public IP `52.186.44.12`.
2. Open Network Security Group rules. Inbound Port 3389 is set to "Allow Any Any".
3. Check the developer's client machine network. The developer is working from an office network that blocks outbound port 3389.
**Root Cause**: The VM is configured correctly, but the developer's local corporate firewall blocks outbound connections on port 3389, preventing them from connecting.
**Fix**:
1. Change the RDP listening port on the Azure VM or deploy **Azure Bastion**.
2. Azure Bastion provides secure, browser-based RDP/SSH access over SSL (port 443), bypassing the blocked port 3389 on the developer's network.
3. Deploy Bastion inside `VNet-Lab` and have the developer log in via the Azure Portal.
**Prevention**: Avoid opening port 3389 directly to the public internet. Use Azure Bastion or a VPN Gateway for all administrative access.

### Scenario 2: Global Administrator locked out due to MFA Loss
**User Complaint**: An L3 Azure administrator lost their authentication device and cannot log in to the Azure Portal. They are the only Global Administrator account in the tenant.
**Your First 3 Checks**:
1. Check if there are other accounts with delegated administrator rights.
2. Check if a "Break-Glass" (Emergency Access) account was configured.
3. Open a support request with Microsoft Data Protection team.
**Diagnosis Steps**:
1. Confirm the user is locked out and has no backup MFA methods configured.
2. Search the Entra ID portal (using another account with Read permissions) for accounts matching `breakglass@domain.com`.
3. Locate the emergency account. It is configured to bypass all conditional access policies and has MFA disabled.
**Root Cause**: The primary administrator lost their phone, and because no alternative authentication options were configured, their login attempt is blocked.
**Fix**:
1. Log in to the Azure tenant using the pre-configured **Break-Glass** account.
2. Navigate to Microsoft Entra ID -> Users -> Select the locked Administrator -> Authentication methods.
3. Click **"Require re-register multi-factor authentication"** and clear their existing devices, allowing them to register their new phone on their next login.
**Prevention**: Always maintain at least two emergency access (break-glass) accounts in your Entra tenant, excluded from conditional access policies, with secure offline password storage.

---

## Critical Points

> [!danger] Never Do This
> Do not leave inbound RDP (port 3389) or SSH (port 22) open to "Any" source IP address in your NSG. Automated bots scan public cloud IP ranges constantly. Leaving these ports open will result in brute-force attacks within minutes.

> [!warning] Common Trap
> Leaving idle VMs running when they are not in use. Azure charges hourly for compute resources. Running an unneeded VM in your lab can consume your free trial credits quickly. Configure auto-shutdown rules for all lab VMs.

> [!tip] Senior Engineer Tip
> Use **Azure Service Tags** in your NSG rules instead of raw IP ranges. For example, use the `AzureActiveDirectory` service tag to allow traffic to Entra endpoints, or the `Internet` tag to control outbound web access.

> [!success] Verification Steps
> To verify a web server VM is reachable:
> 1. Set up an NSG rule allowing Inbound TCP Port 80.
> 2. Open a web browser on your host machine and type the VM's public IP address.
> 3. Verify the default IIS or Apache page loads.

> [!question] Interview Alert
> "What is the difference between an Azure Network Security Group (NSG) and Azure Firewall?"
> - **Answer**: NSGs are basic network/transport layer firewalls applied at the subnet or NIC level to filter traffic on ports/IPs. Azure Firewall is a managed, highly available Layer 3-7 security service that provides centralized traffic filtering, FQDN filtering, and threat intelligence integration across subscriptions.

---

## Common Mistakes & Fixes

| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| Public IP changes on reboot | Public IP allocation set to Dynamic | Change the Public IP assignment method from Dynamic to Static in the IP resource properties. |
| Resource deletion fails | Active resource locks applied | Delete the Resource Lock from the Azure Portal before initiating resource removal. |
| Incompatible subnets | Trying to link VPN gateways to standard subnets | Create a dedicated subnet named `GatewaySubnet` (specifically spelled) to host virtual gateways. |

---

## Lab Exercise

**Objective**: Create an Azure Resource Group, deploy an Ubuntu Web Server, and configure an NSG to allow HTTP access.
**Time Required**: 20 minutes
**Environment Needed**: Active Azure Subscription and Azure Cloud Shell.

**Steps**:
1. Log in to the Azure Portal and open **Cloud Shell** (bash mode).
2. Create a Resource Group:
   ```bash
   az group create --name "RG-Web-Lab" --location "eastus"
   ```
3. Deploy an Ubuntu VM and auto-install Apache:
   ```bash
   az vm create \
     --resource-group "RG-Web-Lab" \
     --name "VM-Apache" \
     --image "Ubuntu2204" \
     --admin-username "webadmin" \
     --generate-ssh-keys \
     --custom-data "#!/bin/bash\napt-get update\napt-get install -y apache2"
   ```
4. Open port 80 (HTTP) in the Network Security Group:
   ```bash
   az vm open-port --resource-group "RG-Web-Lab" --name "VM-Apache" --port 80
   ```
5. Retrieve the VM's public IP:
   ```bash
   az vm list-ip-addresses --resource-group "RG-Web-Lab" --name "VM-Apache" --output table
   ```
6. Open your browser and navigate to the public IP. Verify the default Apache page loads.
7. Clean up resources to prevent charges:
   ```bash
   az group delete --name "RG-Web-Lab" --no-wait --yes
   ```

**Pass Criteria**: The web page loads successfully over public HTTP, and the resource group is deleted.

---

## Interview Questions & Answers

### L1 Level

#### Q1: What is a Resource Group in Azure and how is it used?
**A**: A Resource Group is a logical container in Azure that holds related resources for an application, environment, or department. It is used to group resources together for easier deployment, monitoring, access management (RBAC), and consolidated billing.

#### Q2: How can you prevent a critical resource (like a production VM) from being deleted by accident?
**A**: I can apply an **Azure Resource Lock** to the resource or its resource group. By setting the lock type to `CanNotDelete` (Delete Lock), authorized users can read and modify the resource, but cannot delete it until the lock is explicitly removed.

---

### L2 Level

#### Q3: What is the difference between a stateful and stateless firewall? Is an Azure NSG stateful?
**A**:
- **Stateless Firewall**: Evaluates each packet of data independently. If you allow inbound traffic, you must also write a corresponding outbound rule for the return traffic to pass.
- **Stateful Firewall**: Tracks the state of active network connections. If an inbound connection is allowed, the return outbound traffic is automatically permitted.
- **Azure NSGs are stateful firewalls**.

#### Q4: Explain the difference between Azure Virtual Network Peering and a VPN Gateway connection.
**A**: VNet Peering links two virtual networks in the same or different regions directly over the Microsoft backbone network, providing high bandwidth and extremely low latency. A VPN Gateway connects VNets or on-premises networks using encrypted tunnels over the public internet, which has lower bandwidth and higher latency, but is required for hybrid connections.

---

### L3 Level

#### Q5: Explain how you would implement a security baseline for a brand new Azure subscription.
**A**:
- **Situation**: The company purchased a new Azure subscription with no active security configurations.
- **Task**: Secure the environment against identity threats, unauthorized resource creation, and accidental cost overruns.
- **Action**: I configured Entra ID Security Defaults to enforce MFA for all users. I set up two Break-Glass administrative accounts and excluded them from conditional policies. I configured **Azure Budgets** with alerts set at 50%, 80%, and 100% of our monthly cost target. Finally, I deployed an **Azure Policy** to restrict resource deployment regions only to `EastUS` and `WestUS`.
- **Result**: Established a secure cloud environment, blocking unauthorized logins and preventing accidental billing spikes.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **ARM Structure**: Management Groups -> Subscriptions -> Resource Groups -> Resources.
> **Security**: Never expose ports 3389 or 22 to the public internet; use Azure Bastion or VPN tunnels.
> **Stateful Networking**: NSGs are stateful. Traffic allowed inbound is automatically allowed outbound.
> **Identity**: Enforce MFA in Entra ID, configure a Break-Glass account, and use PIM for temporary admin elevations.

---

## Related Notes
- [[04-Cloud-and-Security/08-Azure/Azure-Identity|Azure Identity]]
- [[04-Cloud-and-Security/08-Azure/Azure-VMs|Azure VMs]]
- [[04-Cloud-and-Security/07-Microsoft-365/Microsoft-Entra-ID|Microsoft Entra ID]]

---

## Tags
#desktop-support #azure-cloud #virtual-networks #network-security #entra-id #cloud-firewall #lab-complete #L2 #L3

