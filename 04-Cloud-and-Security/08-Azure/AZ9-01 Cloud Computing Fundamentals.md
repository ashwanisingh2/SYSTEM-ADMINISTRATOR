---
tags: [sysadmin, azure, az-900, cloud-computing]
difficulty: Beginner
lab-required: No
read-time: 10 mins
---

# AZ9-01: Cloud Computing Fundamentals

> [!abstract] Overview
> This note covers foundational cloud computing concepts, including cloud service benefits, CapEx/OpEx financial profiles, IaaS/PaaS/SaaS service models, cloud deployment types (Public/Private/Hybrid), and the Shared Responsibility Model.

---
## Concept
Think of cloud computing like the city municipal electricity grid:
- **On-Premises Infrastructure** is like buying your own giant coal-fired power generator, building a storage shed for it in your backyard, hiring full-time mechanics, and running your own cables. You pay a massive upfront fee (CapEx) regardless of whether you turn on one lightbulb or run a factory.
- **Cloud Computing** is turning off your backyard generator and plugging your power cord into the wall outlet. The power company (Azure) manages the massive power generators, security, and repairs. You only pay for the exact amount of electricity you consume per minute (OpEx). If you host a party and need more power (Scale), you draw it instantly; when the party ends, you scale back down, paying only for what you used.

*Seedha simple mein: Cloud computing internet ke zariye servers, storage aur databases ko rent par lene ki facility hai. Isme hume physical hardware buy nahi karna padta (OpEx model) aur hum resources ko demand ke basis par instantly auto-scale kar sakte hain.*

---
## Technical Deep Dive

### 1. Cloud Infrastructure Benefits
- **High Availability (HA):** Guarantees resource availability and minimal downtime through SLA bindings and redundant datacenter clustering.
- **Scalability:** The ability to add resources (CPU/RAM/storage) when load increases.
  - *Vertical Scaling (Scale Up):* Adding more power to an existing server (e.g., upgrading a VM from 4GB to 16GB RAM).
  - *Horizontal Scaling (Scale Out):* Adding more virtual machine instances to share the workload (e.g., scaling from 1 VM to 5 identical VMs).
- **Elasticity:** The ability to **auto-scale** resources dynamically: adding VMs during load spikes (like Black Friday sales) and shutting them down automatically when traffic drops to save costs.
- **Reliability:** Fault tolerance built into hardware. Data is replicated across multiple disks/datacenters automatically.
- **Predictability:** Cost and performance predictability using cloud monitoring tools.
- **Security & Governance:** Built-in physical security, firewalls, and automated patch compliance policies.

### 2. Financial Models: CapEx vs. OpEx
- **CapEx (Capital Expenditure):** Upfront spending on physical infrastructure (buying physical servers, network switches, cooling units, real estate). You must pay the full cost upfront, and amortize the value over several years.
- **OpEx (Operational Expenditure):** Ongoing spending on operations and services. You pay as you go, with no upfront infrastructure costs. You can deduct this expense in the tax year it occurs.
  - *Example:* On-prem SAN hardware purchase ($50,000) = **CapEx**. Azure Blob Storage monthly subscription ($150/month) = **OpEx**.

### 3. Cloud Service Models: IaaS, PaaS, SaaS
The cloud categorizes services based on the level of management the customer retains:

- **IaaS (Infrastructure as a Service):**
  - **Description:** Renting physical servers, storage, and networks. The cloud provider manages the hypervisor and hardware; **you** configure and manage the Operating System, updates, middleware, and applications.
  - **Azure Example:** **Azure Virtual Machines (VMs)**, Virtual Networks (VNets).
- **PaaS (Platform as a Service):**
  - **Description:** The cloud provider manages the physical hardware, virtualization, and the Operating System. **You** only manage the application code and databases.
  - **Azure Example:** **Azure App Services**, Azure SQL Database.
- **SaaS (Software as a Service):**
  - **Description:** Fully managed software applications accessed over the web. You do not manage hardware, OS, or application code; **you** only configure user access settings and consume the service.
  - **Microsoft Example:** **Microsoft 365 (Office 365)**, Microsoft Teams.

### 4. Cloud Deployment Types
- **Public Cloud:** Services are hosted by a third-party cloud provider (Azure, AWS) and shared across multiple tenants over the public internet.
- **Private Cloud:** Cloud infrastructure dedicated exclusively to a single organization. Hosted either in their own on-prem datacenter or by a third party. High security but high CapEx.
- **Hybrid Cloud:** Combines public and private clouds. Allows organizations to host legacy databases in their private cloud for security, while utilizing public cloud compute power for web servers.
- **Multi-Cloud:** Utilizing multiple public cloud providers simultaneously (e.g., using Azure for AD, and AWS for hosting databases) to prevent vendor lock-in.

### 5. Shared Responsibility Model
Who manages what in the cloud depends on the service model:

```
+--------------------------+--------+--------+--------+--------+
| Feature                  | On-Prem|  IaaS  |  PaaS  |  SaaS  |
+--------------------------+--------+--------+--------+--------+
| Data & Access            | Cust   | Cust   | Cust   | Cust   |
| Applications             | Cust   | Cust   | Cust   | Cloud  |
| Operating System (OS)    | Cust   | Cust   | Cloud  | Cloud  |
| Virtualization/Hypervisor| Cust   | Cloud  | Cloud  | Cloud  |
| Physical Servers/Network | Cust   | Cloud  | Cloud  | Cloud  |
| Datacenter Security      | Cust   | Cloud  | Cloud  | Cloud  |
+--------------------------+--------+--------+--------+--------+
```
*(Cust = Customer responsibility | Cloud = Cloud provider responsibility)*
- **THE GOLDEN RULE:** Regardless of the cloud service model, **the customer is ALWAYS responsible for their own Data and Access Identity permissions.**

---
## Quick Revision Table
| # | Concept | One Line Summary |
|---|---------|-----------------|
| 1 | Elasticity | The ability of a cloud network to scale resources up or down automatically on-demand. |
| 2 | OpEx | Pay-as-you-go financial model with no upfront costs; deduction occurs in the active billing year. |
| 3 | IaaS | Infrastructure model where the provider hosts hardware, and the customer manages the OS up. |
| 4 | PaaS | Platform model where the provider manages hardware and OS; customer only hosts their code. |
| 5 | Shared Respons| The boundary agreement defining what parts of the stack the customer secures versus the cloud. |

---
## Interview Q&A

**Q1: What is the difference between Scalability and Elasticity in cloud computing?**
A: **Scalability** is the ability of an infrastructure system to handle increased load by adding resources, either vertically (adding more CPU/RAM to a single VM) or horizontally (adding more VM instances to a web cluster). It is a measure of capacity limit. **Elasticity** is the ability to automate this process. An elastic system uses monitoring metrics (like CPU crossing $80\%$) to trigger auto-scaling: adding VMs dynamically when load spikes, and shutting them down when traffic drops to prevent billing waste.

**Q2: A company wants to migrate its on-premises SQL database to Azure. They want to eliminate the need to apply operating system security patches but need full control over database engine settings. What service model do you recommend?**
A: 
- **Situation:** A database must be migrated, removing OS patching requirements while retaining database controls.
- **Task:** Recommend the correct cloud service model (IaaS, PaaS, SaaS).
- **Action:** I will recommend **PaaS (Platform as a Service)**, specifically **Azure SQL Database** or **Azure SQL Managed Instance**.
- **Result:** Under PaaS, Microsoft manages the physical server hardware, virtualization layer, and the underlying Windows/Linux operating system, including all security patching. The client only manages the database schemas, tables, and access configurations.

**Q3: Who is responsible for data security and user access permissions in a Software as a Service (SaaS) application like Microsoft 365?**
A: Under the Shared Responsibility Model, the **Customer** is always $100\%$ responsible for securing their own data and managing user access permissions (identities), regardless of the service model. While Microsoft secures the physical datacenters, network routing, and application servers hosting Microsoft 365, the customer must configure password policies, MFA, conditional access rules, and data classification labels to prevent unauthorized logins or data leaks.

---
## Related Notes
- [[04-Cloud-and-Security/08-Azure/AZ9-02 Azure Global Infrastructure|AZ9-02 Azure Global Infrastructure]] — Regions and Availability Zones.
- [[04-Cloud-and-Security/08-Azure/AZ9-03 Azure Compute Services|AZ9-03 Azure Compute Services]] — Azure VM (IaaS) and App Service (PaaS) profiles.
- [[04-Cloud-and-Security/08-Azure/AZ9-06 Azure Cost and Governance|AZ9-06 Azure Cost and Governance]] — Cloud billing and pricing models.

