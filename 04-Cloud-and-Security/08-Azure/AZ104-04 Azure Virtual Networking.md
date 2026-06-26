---
tags: [desktop-support, azure, cloud, L2]
aliases: [az104-04-azure-virtual-networking, az104-04]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #az-104
---

# AZ104-04: Azure Virtual Networking

> [!abstract] Overview
> This note covers advanced Azure networking. It details VNet IP planning, subnet allocations (including Microsoft reserved IPs), Application Security Groups (ASG), User Defined Routes (UDR), VNet Peering transit, Azure Bastion, Private/Service Endpoints, and Private DNS zones.

---

---
## Concept Overview
Think of Azure virtual networking as designing a secure multi-department corporate office campus:
- **VNet Peering** is building a private corridor between the Sales building (VNet A) and the HR building (VNet B) so employees can walk back and forth without going out onto the public street (the Internet).
- **Azure Bastion** is a secure, armored visitor check-in desk at the entrance. Instead of letting remote administrators RDP directly to servers over public ports (a major security risk), administrators log into a web browser SSL portal, and Bastion projects a secure console inside the browser.
- **Private Endpoints** are running a private, dedicated telephone line straight to a secure remote cloud database (PaaS), completely bypassing public network routing.
- **ASG (Application Security Groups)** allow you to group servers logically: instead of writing NSG rules for 50 different IP addresses, you label the servers "WebServers" and write one rule for the group.


---

---
## Technical Deep Dive
### 1. VNet IP Address Planning & Subnet Sizing
- **VNet Allocation:** Choose non-overlapping address spaces (e.g., `10.1.0.0/16`). If subnets overlap, you cannot establish VNet Peering later.
- **Reserved IP Addresses:** For every subnet created, **Azure reserves 5 IP addresses** automatically:
  - `.0` — Network address.
  - `.1` — Default Gateway address.
  - `.2` & `.3` — DNS mapping.
  - `.255` — Broadcast address (not physically used but reserved).
  - *Example:* A `/28` subnet normally has 16 IPs, but in Azure, only $16-5=\mathbf{11}$ usable IP addresses are available for host VMs.

### 2. Application Security Groups (ASGs)
ASGs allow you to configure network security rules based on application logic rather than IP addresses.
- **Use Case:** You have 10 Web VMs and 5 Database VMs. Instead of updating an NSG with 15 different IP addresses, you create two ASGs: `asg-web` and `asg-db`.
- **NSG Rule:** "Allow inbound TCP 1433 from Source: `asg-web` to Destination: `asg-db`."

### 3. Routing: System Routes vs. UDR (User-Defined Routes)
By default, Azure automatically creates system route tables to route traffic between subnets, VNets, and the internet.
- **UDR (Custom Route Tables):** Allows administrators to override default system routing.
  - *Example:* Force all outbound internet traffic from the Database subnet to route through a virtual firewall appliance (NVA) first (called "forced tunneling").
  - *Configuration:* Create a Route Table -> Add route `0.0.0.0/0` -> Next hop type: **Virtual Appliance** -> IP: `10.1.1.4` (the NVA IP).

### 4. VNet Peering & Transit
- **VNet Peering:** Connects two VNets. Traffic runs over Microsoft's private network.
- **Global Peering:** Connects VNets located in different physical Azure regions.
- **Gateway Transit:** Allows peered spoke VNets to share a single VPN Gateway located in the Hub VNet, saving infrastructure costs.

### 5. Secure Access: Azure Bastion
Provides secure RDP/SSH access to VMs directly over SSL (port 443) via the Azure Portal.
- **How it works:** The Bastion host is deployed in a dedicated subnet named **`AzureBastionSubnet`** (requires minimum `/26` size). It assigns a public IP, but the target VMs do not need public IPs. The administrator connects to Bastion over HTTPS, and Bastion proxies the RDP/SSH connection inside the VNet.

### 6. Private Endpoints vs. Service Endpoints
- **Service Endpoint:** Directs traffic from your VNet to Azure PaaS services (like Storage or SQL) over the Microsoft backbone. The PaaS service still uses a public IP address, but firewalls restrict access exclusively to your VNet subnet.
- **Private Endpoint:** Creates a physical network interface (NIC) with a **private IP address** from your subnet inside the VNet, linking it directly to the PaaS resource. The PaaS resource behaves as a local VM, completely removing public IP accessibility.

---

---
## Step-by-Step Lab
> [!info] Lab Setup Needed
> Access to the Azure Portal.

### Step 1: Create a Hub-Spoke VNet Topology
1. Log into the Azure Portal. Search for and click **Virtual networks**. Click **+ Create**.
2. **Hub VNet:**
   - Resource Group: `rg-lab-infrastructure`.
   - Name: `vnet-hub`. Region: **East US**.
   - IP Address: `10.1.0.0/16`.
   - Subnets: Create `default` subnet (`10.1.0.0/24`) and `GatewaySubnet` (`10.1.254.0/24` - reserved for VPN gateways).
   - Click Create.
3. **Spoke VNet:** Click Create again.
   - Resource Group: `rg-lab-infrastructure`.
   - Name: `vnet-spoke01`. Region: **East US**.
   - IP Address: `10.2.0.0/16` (non-overlapping).
   - Subnets: Create `web` subnet (`10.2.1.0/24`).
   - Click Create.

### Step 2: Establish VNet Peering
1. Go to the **vnet-hub** page. In the left panel, click **Peerings** (under Settings). Click **+ Add**.
2. Configuration:
   - Peering link name (Hub to Spoke): `hub-to-spoke01`.
   - Partner Virtual Network: Select `vnet-spoke01`.
   - Peering link name (Spoke to Hub): `spoke01-to-hub`.
   - Traffic settings: Allow traffic forwarding, leave Gateway Transit unchecked for this lab.
3. Click **Add**.
4. **Verify:** Check the peering status on both VNets. Once it changes to **Connected**, VMs in `vnet-hub` can ping VMs in `vnet-spoke01` directly using private IPs.

### Step 3: Deploy Azure Bastion
1. Go to the `vnet-hub` page -> click **Subnets** -> click **+ Subnet**.
2. Name: **Must be exact:** `AzureBastionSubnet`.
3. Range: `10.1.2.0/26`. Click Save.
4. Search for **Bastions** in the portal search. Click **Create**.
5. Project Details: RG `rg-lab-infrastructure`. Name `bastion-hub`.
6. Virtual Network: Select `vnet-hub`.
7. Subnet: It auto-selects `AzureBastionSubnet`.
8. Public IP: Create new. Click **Review + create**, then **Create** (Deployment takes 5-10 mins). You can now RDP/SSH securely to any VM in the VNet using the Bastion option in the portal.

---

---
## Cheat Sheet / Quick Reference
| Command / Configuration | Scope | Purpose / Example |
|---|---|---|
| `systemctl status <service>` | Linux | Check status of system service |
| `ip address show` | Linux | Display local interface network details |
| `Get-Service` | PowerShell | Verify service status on Windows hosts |
| `Test-NetConnection` | PowerShell | Check network path connectivity to target ports |

---
## Troubleshooting
| Problem | Cause | Fix | Command |
|---|---|---|---|
| Service connection timeout | Network firewall or routing blocking traffic | Check network route and enable target ports on firewall | `ping -c 4 <ip>` / `nc -zv <ip> <port>` |
| Access Denied error | User account lacks permissions or invalid credentials | Verify account access permissions or reset password | N/A |
| Resource not found | Object or path is misspelled or deleted | Verify spelling of target path or query active objects | N/A |

---
## Interview Questions
> [!question] L1 Question
> **Q:** How do you verify if the target service is running?
> **A:** On Linux, I would execute `systemctl status <service-name>`. On Windows, I would run `Get-Service <service-name>` in PowerShell or check Services.msc.

> [!question] L2 Question
> **Q:** Explain how you would troubleshoot a network connectivity issue to a remote server.
> **A:** I would verify local IP configuration, test routing gateway using `ping`, trace hops using `traceroute` or `tracert`, and check port accessibility using `telnet` or `Test-NetConnection` on target port.

---
## Seedha Simple Mein
*Seedha simple mein: AZ-104 networking Hub-Spoke topology, NSG/ASG security filtering, VNet Peering, and Private Endpoints par based hota hai. Subnets design karte waqt yaad rakhna chahiye ki Azure har subnet mein 5 IPs reserve karta hai.*

---
## Related Notes
- [[01-Foundations/02-Networking/N-04 IPv4 Addressing Complete Guide|N-04 IPv4 Addressing Complete Guide]] — Dynamic subnetting calculations.
- [[04-Cloud-and-Security/08-Azure/AZ9-04 Azure Storage and Networking|AZ9-04 Azure Storage and Networking]] — Base Virtual Network basics.
- [[04-Cloud-and-Security/08-Azure/AZ104-03 Azure Virtual Machines|AZ104-03 Azure Virtual Machines]] — Connecting VMs to subnet interfaces.
- [[04-Cloud-and-Security/08-Azure/AZ104-05 Azure Load Balancing|AZ104-05 Azure Load Balancing]] — Integrating Load Balancers into VNets.
