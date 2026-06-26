---
tags: [desktop-support, azure, networking, nsg, L2, interview-topic, lab-complete, daily-use]
aliases: [azure-networking]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #az-104
---

> [!NOTE|color-blue]
> ☁️ **AZURE CLOUD**

`#complete` `#intermediate` `#az-104`

# Azure Networking

> [!abstract] Overview
> Azure Networking provides the virtualized infrastructure to connect Azure resources (like VMs, databases, and App Services) to each other, to on-premises office networks, and to the internet. A support engineer must know this because a server cannot function without network connectivity. Support engineers configure network interfaces, troubleshoot firewall blocks on specific ports, manage VNet peering, and set up remote developer VPNs.

---
## 🧠 Concept Overview

- **What it is** — Azure Networking provides the virtualized infrastructure to connect Azure resources to each other and the internet. The core building blocks are Virtual Networks (VNets), Subnets, and Network Security Groups (NSGs).
- **Why it matters** — A server cannot function without network connectivity. Support engineers configure network interfaces, troubleshoot firewall blocks on specific ports (e.g., HTTP 80, RDP 3389, SQL 1433), manage VNet peering, and set up remote developer VPNs.
- **Where you see this** — Opening port rules in NSGs to allow database connections, setting up Point-to-Site VPNs for remote users, verifying VNet routing, and using Network Watcher to trace connection drops.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Verifies a VM's public/private IP addresses, checks basic port connectivity, and guides users on connecting to Azure VPN clients. |
| **L2** | Manages NSG inbound and outbound rules, creates virtual subnets, establishes VNet Peering, and checks route tables. |
| **L3** | Designs hybrid cloud networking topologies (Hub-and-Spoke), configures ExpressRoute circuits, manages Azure Firewalls, and designs user-defined routing (UDR) paths. |

> [!tip] Seedha Simple Mein
> *Azure-Networking ke bare mein seekhta hai. Yeh azure infrastructure aur system settings ko properly implement karne aur support tickets ko runbooks ke help se standard templates me clear karne me help karta hai.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Azure Networking** is like **a city's road and traffic system** because...
>
> - VNets and Subnets are the different roads and neighborhoods.
> - NSGs are the security checkpoints that allow or deny traffic based on rules.

---
## 🔬 Technical Deep Dive

### 1. Virtual Networks (VNets) & Subnets

> [!info] Key Concept
> A logical isolation of the Azure cloud dedicated to your subscription.

- **VNet**: A representation of your own network in the cloud. You define the private IP address space using CIDR notation (e.g., `10.0.0.0/16`).
- **Subnets**: Subdivisions of a VNet (e.g., `10.0.1.0/24`). Subnets allow you to group related resources and isolate them using network security boundaries.

### 2. Network Security Groups (NSG) & Application Security Groups (ASG)

> [!info] Key Concept
> Stateful packet filtering firewall containing a list of Security Rules that allow or deny network traffic.

- **NSG Priority Numbers**: Rules are evaluated in order from lowest priority number (e.g., 100) to highest (e.g., 65000). The first rule that matches the traffic pattern is applied.
- **ASG**: Group virtual network interfaces (NICs) together to apply single NSG rules to multiple resources simultaneously.

> [!danger] Common Mistake
> Exposing RDP (3389) publicly on the internet. Never create an inbound NSG rule allowing any source (`*`) to connect to port `3389` (RDP) or `22` (SSH) on public IP addresses.

### 3. VNet Peering

VNet Peering connects two separate Virtual Networks seamlessly:
- **Local Peering**: Connects VNets in the same Azure region.
- **Global Peering**: Connects VNets across different physical Azure regions.
- **No Overlapping IPs**: You cannot peer two VNets if their configured IP address spaces overlap.

### 4. Hybrid Connectivity Options

- **Point-to-Site (P2S) VPN**: Connects an individual client computer to the VNet using secure VPN client protocols.
- **Site-to-Site (S2S) VPN**: Connects an entire on-premises office network to the Azure VNet over an encrypted tunnel.
- **ExpressRoute**: A dedicated, private, high-speed fiber connection from your office network to Azure.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Azure Az PowerShell module logged in.
> - Azure Free Tier or Sandbox account.

### Step 1: Create Resource Group and VNets

```powershell
# Create RG
New-AzResourceGroup -Name "RG-Lab-Networking" -Location "eastus"

# Create Virtual Network A
$vnetA = New-AzVirtualNetwork -ResourceGroupName "RG-Lab-Networking" -Name "VNet-Lab-A" -AddressPrefix "10.100.0.0/16" -Location "eastus"

# Create Virtual Network B
$vnetB = New-AzVirtualNetwork -ResourceGroupName "RG-Lab-Networking" -Name "VNet-Lab-B" -AddressPrefix "10.200.0.0/16" -Location "eastus"
```

### Step 2: Establish Peering

```powershell
# Peer A to B
Add-AzVirtualNetworkPeering -Name "Link-A-to-B" -VirtualNetwork $vnetA -RemoteVirtualNetworkId $vnetB.Id

# Peer B to A
Add-AzVirtualNetworkPeering -Name "Link-B-to-A" -VirtualNetwork $vnetB -RemoteVirtualNetworkId $vnetA.Id
```

### Step 3: Verify Peering

```powershell
# Check peering status
Get-AzVirtualNetworkPeering -VirtualNetworkName "VNet-Lab-A" -ResourceGroupName "RG-Lab-Networking" | Select-Object Name, PeeringState
```

> [!success] Expected Output
> ```
> Name        PeeringState
> ----        ------------
> Link-A-to-B Connected
> ```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `New-AzVirtualNetwork` | Creates a new VNet | `New-AzVirtualNetwork -Name "VNet1" -AddressPrefix "10.10.0.0/16"` |
| `Add-AzVirtualNetworkPeering` | Peers two VNets | `Add-AzVirtualNetworkPeering -VirtualNetwork $vnetA -RemoteVirtualNetworkId $vnetB.Id` |
| `Get-AzNetworkSecurityGroup` | Retrieves NSG info | `Get-AzNetworkSecurityGroup -Name "NSG-Web"` |
| `az network nsg create` | Creates NSG in CLI | `az network nsg create --name NSG-Web-Rules` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| Service connection timeout | Network firewall or routing blocking traffic | Check network route and enable target ports on firewall |
| Access Denied error | User account lacks permissions or invalid credentials | Verify account access permissions or reset password |
| Resource not found | Object or path is misspelled or deleted | Verify spelling of target path or query active objects |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Newly Deployed VM Cannot Access Database (NSG Port Block)

> [!example] Ticket
> "My web application cannot connect to our database server (IP: 10.10.2.4) running on port 1433. The database is active, and ping works, but connection fails."

**L1 Response:** Verify VM public/private IPs, perform basic ping test, guide user to Azure Network Watcher.
**Escalation Trigger:** Passes to L2 if ping works but port is blocked by NSG.
**L2 Resolution:** Add inbound rule to `NSG-DB-Prod` allowing SQL port 1433 from the web server subnet range (`10.20.1.0/24`). Test TCP connection using `Test-NetConnection`.

### 🎫 Scenario 2: VNet Peering Setup Fails (Overlapping IP Space)

> [!example] Ticket
> "I am trying to establish VNet Peering, but the peering status keeps showing 'Failed' with a notification saying IP address ranges overlap."

**L1 Response:** Checks both VNet IP spaces to confirm overlap.
**Escalation Trigger:** Overlap requires architecture change or recreation.
**L2 Resolution:** Rebuild the branch VNet with a non-overlapping IP space. Re-establish peering.

---
## 🎤 Interview Questions

> [!question] Q1: What is a Network Security Group (NSG) in Azure?
> **Answer:** An NSG is a virtual firewall that controls inbound and outbound network traffic to Azure network interfaces (NICs) or subnets. It contains security rules where you specify source, destination, port, and protocol to allow or deny traffic.

> [!question] Q2: Which port does Remote Desktop Protocol (RDP) use, and why should it be restricted in Azure?
> **Answer:** RDP uses port 3389. It should be restricted because leaving port 3389 open to the public internet allows automated bots to scan and execute brute-force password attacks on your virtual servers, risking a breach.

> [!question] Q3: What are the requirements to connect two Azure Virtual Networks using VNet Peering?
> **Answer:** To peer two VNets, they must be in active subscriptions, have non-overlapping IP address spaces, and the peering must be configured in both directions (VNet A to B, and B to A).

> [!question] Q4: What is the difference between an NSG and an ASG?
> **Answer:** An NSG is the virtual firewall containing the security rules. An ASG (Application Security Group) is a logical grouping of VM network interfaces used as source or destination targets inside NSG rules.

> [!question] Q5: You have a Hub-and-Spoke VNet topology. Spoke A and Spoke B are both peered to the Hub VNet. How do you enable Spoke A to communicate with Spoke B?
> **Answer:** VNet Peering is non-transitive, so Spoke A cannot talk to Spoke B directly through peering. You must deploy a Network Virtual Appliance (NVA) or Azure Firewall in the Hub VNet, configure User-Defined Routes (UDRs), and enable "Use remote gateways" or "Allow gateway transit" on the peering settings.

==**Exam Tip:** VNet Peering requires non-overlapping IP spaces, and peering is non-transitive.==

---
## 🔗 Related Notes

- [[01-Foundations/02-Networking/Subnetting|Subnetting]] — Core IP subnet planning theory.
- [[04-Cloud-and-Security/08-Azure/Azure-VMs|Azure VMs]] — Explains resources that connect to VNets.
- [[04-Cloud-and-Security/08-Azure/Azure-Identity|Azure Identity]] — Manages access to network security group configurations.
