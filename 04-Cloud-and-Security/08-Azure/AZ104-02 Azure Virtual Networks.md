---
tags: [azure, networking, vnet, az-104]
aliases: [Azure VNet, Virtual Network]
created: 2026-06-26
status: #complete
difficulty: #intermediate
cert-relevant: #az-104
---

> [!NOTE|color-blue]
> ☁️ **AZURE / CLOUD**

`#complete` `#intermediate` `#az-104`

# AZ104-02: Azure Virtual Networks (VNet)

> [!abstract] Overview
> Azure Virtual Network (VNet) is the fundamental building block for your private network in Azure. Yeh note detail mein cover karta hai ki VNet kaise kaam karta hai, subnets kya hote hain, aur routing aur security kaise configure ki jaati hai. Ek support engineer ya cloud admin ko Azure mein kisi bhi resource ko securely connect karne ke liye iska in-depth knowledge hona chahiye.

---
## 🧠 Concept Overview

- **What it is** — A logical isolation of the Azure cloud dedicated to your subscription. It allows Azure resources (like VMs) to securely communicate with each other, the internet, and on-premises networks.
- **Why it matters** — Bina VNet ke aap Azure mein securely machines deploy nahi kar sakte. Yeh foundation hai poore cloud infrastructure ka.
- **Where you see this** — Jab bhi ek VM banta hai, ya koi service private network pe deploy hoti hai, toh ek VNet aur Subnet ki zaroorat padti hai.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic task — Check karna ki VM ko IP mila hai ya nahi, aur NSG (Network Security Group) rules dekhna ki port open hai ya block. |
| **L2** | Configure, fix, escalate — VNet peering configure karna, subnet add karna, ya routing issues fix karna. |
| **L3** | Architecture, design — Hub-and-Spoke topology design karna, ExpressRoute ya VPN Gateway setup karna enterprise scale ke liye. |

> [!tip] Seedha Simple Mein
> *VNet ek virtual building ki tarah hai, aur subnets uske alag-alag floors ya rooms hain. Har room ka apna ek unique address (IP) hota hai jahan pe servers (resources) rehte hain.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Azure VNet** is like **a Gated Community (Society)** because...
>
> - **VNet (Society Boundry):** Baahar ke log direct andar nahi aa sakte bina permission ke.
> - **Subnets (Blocks/Towers):** Society ke andar alag-alag blocks hote hain (A Block, B Block). Aise hi VNet mein subnets hote hain (Web Subnet, DB Subnet).
> - **NSG / Firewall (Security Guard):** Yeh decide karte hain ki kisko aane dena hai aur kisko nahi.
> - **IP Address (House Number):** Har ghar ka ek address hota hai, waise hi har VM ka ek IP hota hai.

---
## 🔬 Technical Deep Dive

### 1. Address Spaces & Subnets

> [!info] Key Concept
> **Address Space (CIDR):** The range of IP addresses assigned to the VNet. Example: `10.0.0.0/16`.
> **Subnet:** A smaller segment of the VNet address space. Example: `10.0.1.0/24`.

Azure reserves 5 IP addresses in every subnet:
1. `x.x.x.0`: Network address
2. `x.x.x.1`: Azure default gateway
3. `x.x.x.2`: Azure DNS mapping
4. `x.x.x.3`: Azure DNS mapping (future use)
5. `x.x.x.255`: Network broadcast address (Azure doesn't support broadcast, but it's reserved)

*Iska matlab agar aapko `/24` (256 IPs) subnet milta hai, toh actual usable IPs sirf 251 hote hain.*

> [!danger] Common Mistake
> Kabhi bhi VNet ka address space on-premises ya doosre connected VNets ke saath overlap nahi hona chahiye. Agar IP address range same hui, toh VPN ya Peering configure nahi ho payegi aur routing conflict aayega.

### 2. VNet Peering

> [!info] Key Concept
> Connecting two VNets together so resources in either VNet can communicate with each other using private IP addresses.
> *Yeh do alag societies ke beech ek direct bridge banane jaisa hai.*

- **Local Peering:** Same Azure region mein VNets connect karna.
- **Global Peering:** Alag-alag Azure regions mein VNets connect karna.

### 3. Network Security Groups (NSG)

> [!info] Key Concept
> Ek virtual firewall jo network traffic ko allow ya deny karti hai. Isme Inbound aur Outbound rules hote hain. NSG ya toh Subnet pe lagta hai, ya seedha NIC (Network Interface) pe.

*Best practice yeh hai ki NSG ko subnet level pe lagao, taaki us subnet ke andar aane waale sabhi VMs pe ek saath rules apply ho jayein.*

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - An active Azure Subscription.
> - Sufficient permissions (Network Contributor ya Contributor role).

### Step 1: Create a Resource Group

```bash
# Yeh command ek naya Resource Group banata hai East US region mein
az group create --name RG-Network-Lab --location eastus
```

> [!success] Expected Output
> ```json
> {
>   "id": "/subscriptions/.../resourceGroups/RG-Network-Lab",
>   "location": "eastus",
>   "name": "RG-Network-Lab",
>   "properties": {
>     "provisioningState": "Succeeded"
>   }
> }
> ```

### Step 2: Create a Virtual Network and Subnet

```bash
# Yeh command VNet banata hai aur sath mein ek default subnet bhi
az network vnet create \
  --resource-group RG-Network-Lab \
  --name VNet-Prod \
  --address-prefix 10.0.0.0/16 \
  --subnet-name Subnet-Web \
  --subnet-prefix 10.0.1.0/24
```

> [!success] Expected Output
> ```json
> {
>   "newVNet": {
>     "addressSpace": {
>       "addressPrefixes": [
>         "10.0.0.0/16"
>       ]
>     },
>     "subnets": [
>       {
>         "addressPrefix": "10.0.1.0/24",
>         "name": "Subnet-Web",
>         "provisioningState": "Succeeded"
>       }
>     ],
>     "provisioningState": "Succeeded"
>   }
> }
> ```

### Step 3: Create a Network Security Group (NSG)

```bash
# NSG create karna
az network nsg create \
  --resource-group RG-Network-Lab \
  --name NSG-Web
```

### Step 4: Associate NSG with Subnet

```bash
# NSG ko subnet pe link karna
az network vnet subnet update \
  --resource-group RG-Network-Lab \
  --vnet-name VNet-Prod \
  --name Subnet-Web \
  --network-security-group NSG-Web
```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `az network vnet list -o table` | List all VNets in the subscription. | `az network vnet list --resource-group RG-01` |
| `az network vnet create` | Naya Virtual Network banata hai. | `az network vnet create -g RG -n vnet1` |
| `az network vnet subnet create` | Existing VNet mein naya subnet add karta hai. | `az network vnet subnet create -g RG --vnet-name vnet1 -n sub1` |
| `az network vnet peering create` | Do VNets ke beech peering setup karta hai. | `az network vnet peering create -n Peer1to2 -g RG --vnet-name v1 ...` |
| `az network nsg rule create` | NSG ke andar naya port allow/deny rule add karta hai. | `az network nsg rule create -g RG --nsg-name nsg1 -n allow-ssh ...` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| Cannot connect to VM via RDP/SSH. | NSG mein port 3389 (RDP) ya 22 (SSH) open nahi hai. | NSG inbound rules check karo aur RDP/SSH ko apni IP ke liye allow karo. |
| Can't create VNet Peering. | Address space overlapping ho raha hai dono VNets ke beech. | Do VNets jinki IP range same hai, unhe peer nahi kar sakte. Alag IP range use karni padegi. |
| VM is not getting a public IP. | Public IP address resource attach nahi kiya gaya VM ke NIC pe. | Portal se NIC settings mein jao aur ek Public IP assign karo. |
| Subnet cannot be deleted. | Subnet abhi bhi kisi resource (like VM ya App Gateway) se attached hai. | Pehle subnet ke andar saare resources ko delete ya move karo, phir subnet delete hoga. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Unable to reach web server from the internet

> [!example] Ticket
> "Hi IT Support, we deployed a new web server on Azure VM but nobody can access the website from outside."

**L1 Response:** *Pehle NSG check karega.* Check if NSG associated with the Subnet/NIC has an inbound rule allowing port 80/443. Check if the VM has a Public IP assigned.
**Escalation Trigger:** Agar NSG theek hai, public IP bhi hai, but phir bhi site nahi khul rahi. *Kab L2 ko pass kare* -> Routing issues ya internal firewall suspected ho.
**L2 Resolution:** Check OS level firewall (Windows Firewall or Linux iptables). Run `Network Watcher -> IP Flow Verify` to confirm if traffic is blocked by any specific rule. Ensure IIS/Apache service is actually running on the VM.

### 🎫 Scenario 2: Two VMs in different VNets cannot communicate

> [!example] Ticket
> "Database VM in VNet-DB cannot be reached by Web VM in VNet-Web via private IP."

**L1 Response:** Confirm if both VMs are running. Check if they have private IPs.
**Escalation Trigger:** Basic checks pass. Requires networking configuration.
**L2 Resolution:** Configure VNet Peering between VNet-Web and VNet-DB. *Dono taraf se peering link establish karna zaroori hai (VNet-Web -> VNet-DB AND VNet-DB -> VNet-Web).* Check NSG rules on both sides to ensure cross-VNet traffic is allowed.

### 🎫 Scenario 3: Unable to add a new subnet to VNet

> [!example] Ticket
> "I am trying to add a new subnet for AKS but the portal says invalid address space."

**L1 Response:** Identify what address space user is trying to add. Compare with existing VNet address space.
**Escalation Trigger:** User is confused about CIDR and networking logic.
**L2 Resolution:** Calculate subnet size. If VNet is `10.0.0.0/24` and it's already full, you cannot add a subnet of `/24`. *Fix:* Expand the VNet address space (add another CIDR block like `10.0.1.0/24` to VNet) and then create the subnet. Note: You can add address space dynamically without downtime now.

---
## 🎤 Interview Questions

> [!question] Q1: How many IP addresses does Azure reserve in a subnet and why?
> **Answer:** Azure reserves 5 IP addresses per subnet. First 3 (`.0` network, `.1` gateway, `.2` DNS mapping) and last 1 (`.255` broadcast). Additionally `.3` is reserved for future use. 

==**Exam Tip:** Agar interview mein poochein ki /24 subnet mein kitne usable IP hote hain, toh answer hamesha 251 dena (256 - 5).==

> [!question] Q2: What is the difference between VNet Peering and VPN Gateway?
> **Answer:** VNet Peering is a direct, high-bandwidth, low-latency connection over Microsoft's backbone network used to connect VNets. VPN Gateway is used to connect Azure VNets to on-premises networks over the public internet using encrypted tunnels. 

==**Exam Tip:** VNet peering doesn't require a virtual network gateway and has no bandwidth restrictions other than VM capabilities.==

> [!question] Q3: Can you peer two VNets with overlapping IP address ranges?
> **Answer:** No. VNet Peering requires the address spaces of the connected VNets to be non-overlapping. Agar overlap hoga, toh network packets confuse ho jayenge ki traffic kis network pe bhejna hai.

> [!question] Q4: What is the default routing behavior in a VNet?
> **Answer:** By default, resources within the same VNet can communicate with each other automatically. Azure provides default system routes. To override default routing (e.g., to send all traffic to a firewall appliance), you must create and apply a User Defined Route (UDR).

> [!question] Q5: Can I apply an NSG to a VNet?
> **Answer:** No. NSGs can only be applied to a Subnet or directly to a Network Interface Card (NIC), not at the VNet level.

---
## 🔗 Related Notes

- [[04-Cloud-and-Security/08-Azure/AZ104-03 Network Security Groups|Network Security Groups (NSG)]]
- [[04-Cloud-and-Security/08-Azure/AZ104-01 Azure Virtual Machines|Azure Virtual Machines]]
- [[04-Cloud-and-Security/08-Azure/AZ104-04 Azure VPN and ExpressRoute|Azure VPN and ExpressRoute]]
