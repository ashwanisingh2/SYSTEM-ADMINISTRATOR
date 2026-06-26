---
tags: [desktop-support, azure, cloud, L2]
aliases: [az104-04-azure-virtual-networking, az104-04]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #az-104
---

> [!NOTE|color-blue]
> ☁️ **AZURE CLOUD**

`#complete` `#advanced` `#az-104`

# AZ104-04: Azure Virtual Networking

> [!abstract] Overview
> Yeh note advanced Azure networking ko cover karta hai. Isme VNet IP planning, subnets (aur Microsoft reserved IPs), Application Security Groups (ASG), UDR (User-Defined Routes), VNet Peering, Azure Bastion, aur Private/Service Endpoints ki jaankari hai. Ek support engineer ke liye network connectivity aur security troubleshoot karne mein yeh bahut zaroori hai.

---
## 🧠 Concept Overview

- **What it is** — Azure cloud mein aapka apna private network, jahan aap apne resources (VMs, DBs) ko secure aur isolate kar sakte hain.
- **Why it matters** — Bina secure virtual networking ke aapke cloud resources internet pe open ho sakte hain ya aapas mein baat nahi kar payenge.
- **Where you see this** — Jab do VMs aapas mein connect nahi ho rahe, ya on-premise se Azure secure connection chahiye.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic connectivity check karna (ping, telnet) aur Azure portal se status dekhna. |
| **L2** | NSG rules configure karna, VNet peering setup karna, aur routing issues fix karna. |
| **L3** | Enterprise level hub-and-spoke architecture design karna aur complex VPN/ExpressRoute integration karna. |

> [!tip] Seedha Simple Mein
> *Azure virtual networking ek badi office building ki tarah hai jahan alag-alag departments (subnets) hain. Peering corridor ka kaam karta hai, aur Bastion ek secure check-in desk hai bina bahar ki road (internet) use kiye.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Azure Virtual Networking** is like designing a secure **Multi-Department Corporate Campus** because...
>
> - **VNet Peering** is building a private corridor between the Sales building (VNet A) and HR building (VNet B) so employees can walk without going out on the public street.
> - **Azure Bastion** is a secure, armored visitor check-in desk at the entrance. Instead of public access, visitors use a secure monitored console.
> - **Private Endpoints** are running a dedicated private telephone line straight to a secure vault.
> - **ASG** is like labelling rooms as "Meeting Rooms" instead of listing room numbers 101 to 150.

---
## 🔬 Technical Deep Dive

### 1. VNet IP Address Planning & Subnet Sizing

> [!info] Key Concept
> IP Address space design karte waqt non-overlapping ranges use karein. Har subnet mein Azure **5 IP addresses reserve** karta hai.

- `.0` — Network address.
- `.1` — Default Gateway address.
- `.2` & `.3` — DNS mapping.
- `.255` — Broadcast address (not physically used but reserved).

> [!danger] Common Mistake
> Agar subnets ka IP range overlap ho gaya, toh aap baad mein VNet Peering establish nahi kar payenge. Hamesha future expansion ka dhyan rakhein.

### 2. Application Security Groups (ASGs) & NSG

> [!info] Key Concept
> ASGs allow you to configure network security rules based on application logic rather than IP addresses.

- **Use Case:** 10 Web VMs aur 5 DB VMs ke liye alag alag IPs ki jagah `asg-web` aur `asg-db` use karein.
- **Rule Example:** Allow inbound TCP 1433 from Source: `asg-web` to Destination: `asg-db`.

### 3. Routing: System Routes vs. UDR (User-Defined Routes)

- **System Routes:** Azure default routing between subnets, VNets, and internet.
- **UDR:** Custom route tables to override defaults. (e.g., Forced tunneling via NVA virtual firewall).

### 4. Secure Access: Azure Bastion

- Deploys in `AzureBastionSubnet` (minimum `/26`).
- Secure RDP/SSH access directly over SSL (port 443) via Azure Portal without assigning public IPs to VMs.

### 5. Private Endpoints vs. Service Endpoints

- **Service Endpoint:** Directs traffic to PaaS over Microsoft backbone. PaaS still has a public IP but firewall restricts access to your VNet.
- **Private Endpoint:** Creates a physical NIC with a private IP in your VNet linking to the PaaS resource. Completely removes public accessibility.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Access to Azure Portal
> - RG `rg-lab-infrastructure` created.

### Step 1: Create a Hub-Spoke VNet Topology

```bash
# Example CLI command for VNet creation (conceptual for portal steps)
az network vnet create --name vnet-hub --resource-group rg-lab-infrastructure --address-prefixes 10.1.0.0/16
```

1. Azure Portal > Virtual networks > **+ Create**.
2. **Hub VNet:** IP: `10.1.0.0/16`. Subnets: `default` (`10.1.0.0/24`) and `GatewaySubnet` (`10.1.254.0/24`).
3. **Spoke VNet:** Name: `vnet-spoke01`, IP: `10.2.0.0/16`. Subnet: `web` (`10.2.1.0/24`).

### Step 2: Establish VNet Peering

1. Go to **vnet-hub** > **Peerings** > **+ Add**.
2. Name (Hub to Spoke): `hub-to-spoke01`. Select partner VNet `vnet-spoke01`.
3. Name (Spoke to Hub): `spoke01-to-hub`.
4. Click Add.

> [!success] Expected Output
> Peering status will change to **Connected**, allowing VMs to communicate via private IPs.

### Step 3: Deploy Azure Bastion

1. Add subnet named `AzureBastionSubnet` (`10.1.2.0/26`) to `vnet-hub`.
2. Create Bastion > Select VNet `vnet-hub`.
3. Create New Public IP. Click **Review + create**.

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `Test-NetConnection` | Check network path & port via PowerShell | `Test-NetConnection 10.2.0.4 -Port 3389` |
| `nc -zv` | Check open ports via Linux netcat | `nc -zv 10.2.0.4 22` |
| `az network vnet peering list` | List Azure peerings via CLI | `az network vnet peering list -g RG1 --vnet-name VNet1` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| VMs in peered VNets cannot communicate | Peering status is "Initiated" or "Disconnected" | Ensure peering is created on BOTH sides (bidirectional). |
| Bastion deployment fails | Subnet name incorrect or too small | Create exactly `AzureBastionSubnet` with `/26` minimum. |
| Cannot access PaaS via Private Endpoint | DNS resolution failure | Configure Azure Private DNS Zone and link it to the VNet. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: VM Connectivity Issue

> [!example] Ticket
> "I cannot RDP to the newly deployed application server from my local machine."

**L1 Response:** Verify VM is running and Bastion/Public IP is correctly assigned. Guide user to use Bastion.
**Escalation Trigger:** If Bastion fails or NSG blocks traffic despite correct rules.
**L2 Resolution:** Check NSG Effective Rules. Ensure inbound port 443 is allowed for Bastion and internal RDP port 3389 is allowed from Bastion Subnet.

---
## 🎤 Interview Questions

> [!question] Q1: How many IP addresses are reserved by Azure in a subnet, and what are they used for?
> **Answer:** Azure reserves 5 IPs: `.0` (network), `.1` (gateway), `.2` and `.3` (DNS), and `.255` (broadcast).

> [!question] Q2: What is the main difference between Service Endpoints and Private Endpoints?
> **Answer:** Service Endpoint routes traffic over MS backbone but the PaaS service retains a public IP. Private Endpoint assigns a private IP from your VNet directly to the PaaS resource, disabling public access.

==**Exam Tip:** For Bastion, the subnet must be named EXACTLY `AzureBastionSubnet` and must be `/26` or larger.==

---
## 🔗 Related Notes

- [[01-Foundations/02-Networking/N-04 IPv4 Addressing Complete Guide|N-04 IPv4 Addressing Complete Guide]] — Dynamic subnetting calculations.
- [[04-Cloud-and-Security/08-Azure/AZ9-04 Azure Storage and Networking|AZ9-04 Azure Storage and Networking]] — Base Virtual Network basics.
- [[04-Cloud-and-Security/08-Azure/AZ104-03 Azure Virtual Machines|AZ104-03 Azure Virtual Machines]] — Connecting VMs to subnet interfaces.
- [[04-Cloud-and-Security/08-Azure/AZ104-05 Azure Load Balancing|AZ104-05 Azure Load Balancing]] — Integrating Load Balancers into VNets.
