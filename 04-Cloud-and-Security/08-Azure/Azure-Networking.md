---
tags: [desktop-support, azure, networking, nsg, L2]
aliases: [azure-networking-guide, vnet-guide, nsg-rules]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #az-104
---

# Azure Networking

---

## Concept Overview
- **What it is**: Azure Networking provides the virtualized infrastructure to connect Azure resources (like VMs, databases, and App Services) to each other, to on-premises office networks, and to the internet. The core building blocks are Virtual Networks (VNets), Subnets, and Network Security Groups (NSGs).
- **Why it matters for a support engineer**: A server cannot function without network connectivity. Support engineers configure network interfaces, troubleshoot firewall blocks on specific ports (e.g., HTTP 80, RDP 3389, SQL 1433), manage VNet peering, and set up remote developer VPNs.
- **Where you encounter this in real job**: Opening port rules in NSGs to allow database connections, setting up Point-to-Site VPNs for remote users, verifying VNet routing, and using Network Watcher to trace connection drops.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Verifies a VM's public/private IP addresses, checks basic port connectivity, and guides users on connecting to Azure VPN clients.
  - **L2**: Manages NSG inbound and outbound rules, creates virtual subnets, establishes VNet Peering, and checks route tables.
  - **L3**: Designs hybrid cloud networking topologies (Hub-and-Spoke), configures ExpressRoute circuits, manages Azure Firewalls, and designs user-defined routing (UDR) paths.

---

## Technical Deep Dive

### 1. Virtual Networks (VNets) & Subnets
- **VNet**: A representation of your own network in the cloud. It is a logical isolation of the Azure cloud dedicated to your subscription. You define the private IP address space using CIDR notation (e.g., `10.0.0.0/16`).
- **Subnets**: Subdivisions of a VNet (e.g., `10.0.1.0/24`). Subnets allow you to group related resources (like a web tier and a database tier) and isolate them using network security boundaries.

### 2. Network Security Groups (NSG) & Application Security Groups (ASG)
- **NSG**: A basic stateful packet filtering firewall. It contains a list of Security Rules that allow or deny network traffic.
  - **Priority Numbers**: Rules are evaluated in order from lowest priority number (e.g., 100) to highest (e.g., 65000). The first rule that matches the traffic pattern is applied.
  - **Default Rules**: Every NSG contains default rules that block all inbound traffic and allow all outbound traffic (with exceptions for VNet communication and internet outbound).
- **ASG**: Group virtual network interfaces (NICs) together. Instead of writing separate NSG rules for 10 individual web server IPs, you group those VMs into an ASG named `ASG-WebServers` and write a single NSG rule targeting the ASG.

### 3. VNet Peering
VNet Peering connects two separate Virtual Networks seamlessly:
- **Local Peering**: Connects VNets in the same Azure region.
- **Global Peering**: Connects VNets across different physical Azure regions.
- **No Overlapping IPs**: **CRITICAL**: You cannot peer two VNets if their configured IP address spaces overlap (e.g., both VNets cannot use `10.0.0.0/16`).

### 4. Hybrid Connectivity Options
- **Point-to-Site (P2S) VPN**: Connects an individual client computer (developer's laptop) to the VNet using secure VPN client protocols.
- **Site-to-Site (S2S) VPN**: Connects an entire on-premises office network to the Azure VNet over an encrypted IPsec/IKE VPN tunnel.
- **ExpressRoute**: A dedicated, private, high-speed fiber connection from your office network to Azure, bypassing the public internet entirely.

---

## Commands & Syntax

### PowerShell
Networking management uses the `Az.Network` module.
```powershell
# Connect to Azure
Connect-AzAccount

# Create a new Virtual Network and a default subnet
New-AzVirtualNetwork -ResourceGroupName "RG-Prod-Network" -Name "VNet-EastUS" -AddressPrefix "10.10.0.0/16" -Location "eastus"
Add-AzVirtualNetworkSubnetConfig -Name "Subnet-Web" -AddressPrefix "10.10.1.0/24" -VirtualNetwork (Get-AzVirtualNetwork -ResourceGroupName "RG-Prod-Network" -Name "VNet-EastUS")

# Add an inbound security rule (Allow RDP Port 3389) to an existing NSG
$NSG = Get-AzNetworkSecurityGroup -ResourceGroupName "RG-Prod-Network" -Name "NSG-Web-Rules"
Add-AzNetworkSecurityRuleConfig -Name "Allow-RDP-Inbound" -NetworkSecurityGroup $NSG -Protocol TCP -Direction Inbound -Priority 100 -SourceAddressPrefix "*" -SourcePortRange "*" -DestinationAddressPrefix "*" -DestinationPortRange 3389 -Access Allow
Set-AzNetworkSecurityGroup -NetworkSecurityGroup $NSG
```

### Azure CLI
```bash
# Create a Network Security Group (NSG)
az network nsg create --resource-group RG-Prod-Network --name NSG-Web-Rules

# Create an inbound rule to allow HTTP (Port 80) traffic
az network nsg rule create --resource-group RG-Prod-Network --nsg-name NSG-Web-Rules --name Allow-HTTP-Inbound --priority 110 --source-address-prefixes '*' --destination-port-ranges 80 --direction Inbound --access Allow --protocol Tcp
```

### GUI Path
- **VNet/Subnets**: Azure Portal -> Search **Virtual Networks** -> Click target network -> Select **Subnets**.
- **NSG Rules**: Search **Network security groups** -> Click target NSG -> Select **Inbound security rules** or **Outbound security rules**.

---

## Real-World Scenarios

### Scenario 1: Newly Deployed VM Cannot Access Database (NSG Port Block)
**User Complaint:** A developer deploys a web server VM (`VM-Web-Prod`) in a new subnet. They report: *"My web application cannot connect to our database server (IP: 10.10.2.4) running on port 1433. The database is active, and ping works, but connection fails."*
**Your First 3 Checks:**
1. Check the subnet configurations of both the web server and the database server.
2. Verify the Inbound security rules on the database server's NSG.
3. Verify the Outbound security rules on the web server's NSG.
**Diagnosis Steps:**
1. Log in to the Azure Portal. Go to the Database VM -> Click **Networking**.
2. Note the attached Network Security Group: `NSG-DB-Prod`.
3. Check the **Inbound port rules**:
   - Rule 100: `Allow-VNet-Traffic (Any to Any)`.
   - Rule 110: `Block-All-Inbound (Any to Any)`.
   - *Wait, if VNet traffic is allowed, why is the web server blocked?*
4. Check the Web Server's subnet: `VNet-Prod/Subnet-Web` (IP: `10.20.1.15`).
5. Check the Database Server's subnet: `VNet-Prod/Subnet-DB` (IP: `10.10.2.4`).
6. *The web server is on a different VNet!* The VNet peering between `VNet-Prod` and `VNet-DB` was created, but the database's NSG rule only allowed traffic from its *local* VNet. Traffic from the peered web VNet was blocked by the default rules.
**Root Cause:** The database NSG did not contain a rule allowing inbound traffic on port 1433 from the peered Web VNet IP range.
**Fix:**
1. Go to `NSG-DB-Prod` -> **Inbound security rules** -> Click **Add**.
2. Source: **IP Addresses**. Source IP: `10.20.1.0/24` (Web Subnet range).
3. Protocol: **TCP**. Destination Port: `1433`.
4. Action: **Allow**. Priority: `120`. Name: `Allow-WebSubnet-to-SQL`. Click **Add**.
5. Test database connectivity from the web server using PowerShell:
   `Test-NetConnection -ComputerName 10.10.2.4 -Port 1433`
   - Output: `TcpTestSucceeded : True`.
**Prevention:** Use Application Security Groups (ASGs) to group resources logically, and write rules based on ASG identities rather than hardcoded subnets.
**Ticket Close Note:** "Added inbound rule to NSG-DB-Prod allowing SQL port 1433 from the web server subnet range. Verified TCP test succeeds. Closed."

### Scenario 2: VNet Peering Setup Fails (Overlapping IP Space)
**User Complaint:** A network admin trying to link a newly acquired branch office VNet (`VNet-Branch`) to our main hub VNet (`VNet-Hub`) reports: *"I am trying to establish VNet Peering, but the peering status keeps showing 'Failed' with a notification saying IP address ranges overlap."*
**Your First 3 Checks:**
1. Check the IP Address space configured for `VNet-Hub`.
2. Check the IP Address space configured for `VNet-Branch`.
3. Verify if there is any overlapping CIDR range.
**Diagnosis Steps:**
1. Open the Azure Portal. Navigate to `VNet-Hub`.
   - Address Space: `10.0.0.0/16` (IP range `10.0.0.0` - `10.0.255.255`).
2. Navigate to `VNet-Branch`.
   - Address Space: `10.0.5.0/24` (IP range `10.0.5.0` - `10.0.5.255`).
3. Compare the ranges.
   - The Branch network range (`10.0.5.0/24`) is completely contained inside the Hub network range (`10.0.0.0/16`).
4. Azure routing cannot resolve destinations if the same IP block exists in both VNets.
**Root Cause:** The two virtual networks contain overlapping IP address spaces, making VNet Peering routing impossible.
**Fix:**
1. *You cannot modify the IP space of a VNet if it has active resources.* Since `VNet-Branch` is new and has no VMs, we can delete it and rebuild it with a different address space.
2. Delete `VNet-Branch`.
3. Create a new virtual network named `VNet-Branch-New` with Address Space: `10.1.0.0/16` (non-overlapping).
4. Create the Peering on `VNet-Hub` pointing to `VNet-Branch-New`.
5. Create the reciprocal Peering on `VNet-Branch-New` pointing to `VNet-Hub`.
6. Verify both peering statuses show `Connected`.
**Prevention:** Establish a corporate IP Address Management (IPAM) registry to track and allocate unique IP blocks to all cloud and local networks.
**Ticket Close Note:** "Rebuilt the branch VNet with a non-overlapping IP space (10.1.0.0/16). Established VNet Peering. Status is now Connected. Closed."

---

## Critical Points

> [!danger] Never Do This
> - Never create an inbound NSG rule allowing any source (`*`) to connect to port `3389` (RDP) or `22` (SSH) on public IP addresses.
> - This leaves your servers open to brute-force attacks from anywhere in the world. Always restrict management ports to specific corporate public IP addresses or use Azure Bastion.

> [!warning] Common Trap
> - Assuming that VNet Peering automatically routes traffic through a third "transit" VNet in a Hub-and-Spoke network model.
> - VNet Peering is non-transitive. If VNet A is peered to VNet B, and VNet B is peered to VNet C, VNet A *cannot* communicate with VNet C. You must peer A directly to C, or deploy an Azure Firewall/NVA in VNet B and configure User-Defined Routes (UDRs).

> [!tip] Senior Engineer Tip
> - When troubleshooting network connectivity, do not manually check every NSG rule. Use **Azure Network Watcher** and select **IP Flow Verify**. You input the source and destination IP/port, and Network Watcher tells you instantly if the connection is allowed or blocked, listing the exact NSG rule name responsible.

> [!success] Verification Steps
> - Run: `Test-NetConnection -ComputerName [Target-IP] -Port [Port]` from a VM to check port status.
> - Confirm the Peering Status shows `Connected` on both peered VNets in the portal.

> [!question] Interview Alert
> - "Explain what happens when two peered VNets contain overlapping IP address spaces."
> - Answer: "VNet Peering will fail to connect. Azure routing tables cannot resolve IP destinations if identical address ranges exist on both networks. To resolve this, one of the virtual networks must be deleted and recreated using a unique, non-overlapping IP address space."

---

## Common Mistakes & Fixes

| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| Exposing RDP (3389) publicly on the internet | Seeking quick remote server access | Use Azure Bastion or a Point-to-Site VPN client to secure administrative connections. |
| Neglecting to configure mutual VNet peering | Assuming one-way setup is sufficient | VNet peering must be created in both directions (VNet A to B, and B to A) to establish connection. |
| Writing overlapping priority numbers in NSG rules | Unplanned security configurations | Standardize priority increments (e.g., 100, 110, 120) to leave space for future rules. |

---

## Lab Exercise

**Objective:** Use PowerShell to create two virtual networks with non-overlapping IP ranges, configure VNet Peering between them, and verify the peering status.
**Time Required:** 30 minutes
**Environment Needed:** Azure Free Tier or Sandbox account.
**Pre-requisites:** Azure Az PowerShell module logged in.

**Steps:**
1. Open PowerShell. Create the Resource Group:
   ```powershell
   New-AzResourceGroup -Name "RG-Lab-Networking" -Location "eastus"
   ```
2. Create Virtual Network A:
   ```powershell
   $vnetA = New-AzVirtualNetwork -ResourceGroupName "RG-Lab-Networking" -Name "VNet-Lab-A" -AddressPrefix "10.100.0.0/16" -Location "eastus"
   ```
3. Create Virtual Network B:
   ```powershell
   $vnetB = New-AzVirtualNetwork -ResourceGroupName "RG-Lab-Networking" -Name "VNet-Lab-B" -AddressPrefix "10.200.0.0/16" -Location "eastus"
   ```
4. Create Peering from VNet A to VNet B:
   ```powershell
   Add-AzVirtualNetworkPeering -Name "Link-A-to-B" -VirtualNetwork $vnetA -RemoteVirtualNetworkId $vnetB.Id
   ```
5. Create Peering from VNet B to VNet A:
   ```powershell
   Add-AzVirtualNetworkPeering -Name "Link-B-to-A" -VirtualNetwork $vnetB -RemoteVirtualNetworkId $vnetA.Id
   ```
6. Verification: Check the peering status on VNet A:
   ```powershell
   Get-AzVirtualNetworkPeering -VirtualNetworkName "VNet-Lab-A" -ResourceGroupName "RG-Lab-Networking" | Select-Object Name, PeeringState
   ```
   - Expected Output: `PeeringState : Connected`.

**Success Criteria:** Both Virtual Networks are created, peered in both directions, and the PeeringState returns `Connected`.
**Common Failures:** The command fails if the Resource IDs are not retrieved correctly, or if there is an IP address collision.

---

## Interview Questions & Answers

### Basic (L1 Level)
**Q: What is a Network Security Group (NSG) in Azure?**
A: An NSG is a virtual firewall that controls inbound and outbound network traffic to Azure network interfaces (NICs) or subnets. It contains security rules where you specify source, destination, port, and protocol to allow or deny traffic.

**Q: Which port does Remote Desktop Protocol (RDP) use, and why should it be restricted in Azure?**
A: RDP uses port 3389. It should be restricted because leaving port 3389 open to the public internet allows automated bots to scan and execute brute-force password attacks on your virtual servers, risking a breach.

### Intermediate (L2 Level)
**Q: What are the requirements to connect two Azure Virtual Networks using VNet Peering?**
A: To peer two VNets, they must be in active subscriptions, have non-overlapping IP address spaces, and the peering must be configured in both directions (VNet A to B, and B to A). Once connected, VMs in both networks can communicate using private IP addresses.

**Q: What is the difference between an NSG and an ASG?**
A: An NSG is the virtual firewall containing the security rules. An ASG (Application Security Group) is a logical grouping of VM network interfaces. You use ASGs inside NSG rules as source or destination targets, allowing you to apply security rules to groups of VMs without hardcoding individual IP addresses.

### Advanced (L3/Senior Level)
**Q: You have a Hub-and-Spoke VNet topology. Spoke A and Spoke B are both peered to the Hub VNet. How do you enable Spoke A to communicate with Spoke B?**
A:
- **Situation**: Peer communication blocked in a Hub-and-Spoke Azure network.
- **Task**: Route traffic transitively through the Hub VNet.
- **Action**: VNet Peering is non-transitive, so Spoke A cannot talk to Spoke B directly through peering. First, I deploy a Network Virtual Appliance (NVA) or Azure Firewall in the Hub VNet. Next, I configure User-Defined Routes (UDRs) in a route table, and link it to the subnets of Spoke A and Spoke B. The route rule states: "To reach the other spoke's IP range, route traffic to the Hub Firewall's private IP." Finally, in the VNet Peering settings, I check the box "Use remote gateways" or "Allow gateway transit".
- **Result**: The traffic routes successfully from Spoke A through the Hub Firewall to Spoke B.

### HR / Behavioral
**Q: Describe a time you had to troubleshoot a network connectivity issue under pressure. How did you isolate the problem?**
A: During a client product demo, our web application server suddenly lost connection to the backend database VM. The developers were arguing about whose code broke. I stepped in, opened Azure Network Watcher, and ran **IP Flow Verify** from the web VM to the database IP on port 1433. The tool reported the connection was blocked by a specific NSG rule. I checked the NSG and found a junior tech had added a rule to block SQL ports to fix a security warning, forgetting it would break the web app. I disabled the rule, restoring connection in under 5 minutes.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **What**: The network infrastructure providing virtual routers, subnets, and firewalls in Azure.
> **Why**: Connects VMs, secures ports, enables hybrid local network connections, and routes cloud traffic.
> **How**: Define non-overlapping VNets/Subnets, manage NSG rule priorities, establish VNet Peering, and use Network Watcher.
> **Command**: `Add-AzNetworkSecurityRuleConfig` / `az network nsg rule create`
> **Interview Answer Starter**: "To manage Azure networks, I define private subnets, apply stateful NSG firewalls, and use VNet Peering for internal routing, ensuring no IP address ranges overlap..."

**Key Numbers to Remember:**
- Default RDP Port: 3389
- Default SSH Port: 22
- NSG priority number range: 100 - 65000
- Maximum peered VNets: VNet peering is non-transitive

**3 Things Interviewer Wants to Hear:**
- VNet Peering requires non-overlapping IP spaces
- Network Watcher IP Flow Verify simplifies NSG troubleshooting
- Disabling public RDP/SSH access in favor of Azure Bastion/VPN

---

## Related Notes
- [[01-Foundations/02-Networking/Subnetting|Subnetting]] — Core IP subnet planning theory.
- [[04-Cloud-and-Security/08-Azure/Azure-VMs|Azure VMs]] — Explains resources that connect to VNets.
- [[04-Cloud-and-Security/08-Azure/Azure-Identity|Azure Identity]] — Manages access to network security group configurations.

---

## Tags
#desktop-support #azure #networking #nsg #L2 #interview-topic #lab-complete #daily-use

