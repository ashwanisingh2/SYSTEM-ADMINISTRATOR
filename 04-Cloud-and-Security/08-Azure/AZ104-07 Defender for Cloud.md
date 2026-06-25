---
tags: [cloud, azure, security, defender, az-104]
aliases: [defender-for-cloud]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #az-104
---

# AZ104-07: Microsoft Defender for Cloud

**Verification:**
- [ ] Enable Microsoft Defender for Cloud on an Azure subscription
- [ ] View and interpret the subscription's Secure Score and recommendations
- [ ] Configure Just-in-Time (JIT) VM Access on an Azure Virtual Machine
- [ ] Onboard a hybrid or multi-cloud server using Azure Arc connection strings
- [ ] Verify automatic alert notifications and link remediation Playbooks (Logic Apps)

> [!abstract] Overview
> Microsoft Defender for Cloud is a cloud-native application protection platform (CNAPP) designed to manage and secure hybrid and multi-cloud environments. This note covers Cloud Security Posture Management (CSPM), Cloud Workload Protection (CWPP), Secure Score evaluation, Just-in-Time (JIT) VM Access, and automated remediation playbooks.

---
## Concept Overview
- **What it is** — Microsoft Defender for Cloud is a central security management console that continuously audits the security state of your cloud resources, calculates a "Secure Score," and alerts you to active threats across Azure, AWS, GCP, and on-premises physical or virtual servers.
- **Why it matters for a support engineer** — In the cloud, minor configuration mistakes (like leaving an SSH port wide open or hosting a public storage bucket) can lead to data breaches. Defender for Cloud acts as an automated security auditor, providing clear remediation guidelines.
- **Where you encounter this in real job** — Auditing compliance standards, closing public network exposure on servers, enabling threat alerts on Azure SQL databases, and setting up secure temporary administrative access to cloud VMs.
- **L1 vs L2 vs L3 responsibility split for this topic:**
  - **L1 Resolution**: Monitor the Secure Score dashboard, review low-severity recommendations, and escalate active high-severity security alerts (e.g. SQL injection attempts).
  - **Escalation Trigger**: Escalate to L2 if applying a recommendation requires changing production virtual network configurations, network security groups, or altering storage access policies.
  - **L2 Resolution**: Implement standard security recommendations (enabling secure transmission, restricting port rules), configure Just-in-Time VM Access, and troubleshoot log analytics agent deployment errors.
  - **L3 Resolution**: Design multi-cloud connectors (AWS/GCP), build automated alert response workflows using Azure Logic Apps (Playbooks), define custom enterprise policy initiatives, and generate compliance reports for auditors.

*Seedha simple mein: Microsoft Defender for Cloud cloud assets ka security guard hai. Yeh pure environment ko check karta rehta hai aur batata hai ki kahan security leaks hain (jaise data storage open reh jana). Yeh hume ek security percentage score deta hai aur use badhane ke tips deta hai.*

---
## Real-World Analogy
Think of a **smart security company** protecting your office building:
- **CSPM (Cloud Security Posture Management)**: The auditor who walks around during the day saying: "Hey, window 3 is unlocked, you need to install a vault lock on the back door." It gives you a checklist of recommendations.
- **CWPP (Cloud Workload Protection)**: The armed guard sitting in the control room who immediately rings the alarm if someone tries to break a window or upload a virus in real time.
- **Multi-Cloud**: This service checks not just your head office (Azure), but also your rented warehouses (AWS/GCP) and regional branch offices (on-premises servers) from a single control room.

---
## Technical Deep Dive

### 1. CSPM vs. CWPP
Microsoft Defender for Cloud is divided into two primary pillars:
- **Cloud Security Posture Management (CSPM)**:
  - *Included in Free Tier (Foundational CSPM)*.
  - Focuses on assessment, policies, and compliance.
  - Generates the **Secure Score** based on resource configuration audits.
  - Identifies security gaps and suggests manual or automated remediations.
- **Cloud Workload Protection Platform (CWPP)**:
  - *Requires paid Defender plans* (e.g., Defender for Servers, Storage, Databases).
  - Focuses on real-time threat detection and security analytics.
  - Generates **Security Alerts** when suspicious activity occurs (e.g., brute force attacks, cryptomining, suspicious process spawning).

### 2. Understanding Secure Score
- Secure Score is represented as a percentage.
- The score is calculated by grouping security recommendations into **Security Controls** (e.g., "Enable MFA", "Restrict Unauthorized Network Access").
- Each control has a maximum point value. You must complete *all* recommendations within a control to get the points associated with it, ensuring thorough security coverage.

### 3. Just-in-Time (JIT) VM Access
Leaving administrative ports (TCP 3389 for RDP and TCP 22 for SSH) open to the internet exposes VMs to constant automated brute-force attacks.
- **How JIT works**: When enabled, Defender for Cloud blocks all inbound RDP/SSH traffic by creating a deny rule in the Network Security Group (NSG).
- When a user needs access, they request it through the Azure portal or CLI.
- If approved, Defender temporarily modifies the NSG rule to allow traffic from the user's specific source IP for a limited time (e.g., 3 hours).
- Once time expires, Defender automatically restores the block rule.

```
                  +-----------------------------------+
                  |      User Requests Access         |
                  +-----------------------------------+
                                    |
                  +-----------------------------------+
                  |  Is Request Approved by Policies? |
                  +-----------------------------------+
                        /                       \
                     Yes                         No
                     /                             \
+-----------------------------------+     +-------------------------+
| Temporary NSG Rule Added:         |     | Request Denied          |
| Source IP allowed on port 3389/22 |     +-------------------------+
| Time window begins (e.g. 3 hours) |
+-----------------------------------+
                     |
+-----------------------------------+
| Time Window Expires:             |
| Rule removed, port blocked again |
+-----------------------------------+
```

---
## Step-by-Step Lab / Configuration

> [!warning] Pre-requisites
> - An active Azure Subscription.
> - A target virtual machine (`VM-PROD-WEB`) configured with a Network Security Group.
> - `Owner` or `Contributor` role permissions on the target resource group.

### Step 1: Enable Defender for Cloud Plans
1. Log in to the Azure Portal.
2. Search for and open **Microsoft Defender for Cloud**.
3. Under **Management** in the left menu, select **Environment settings**.
4. Select your subscription name.
5. In the **Defender plans** tab, toggle **Defender for Servers** to **On**.
6. Click **Save** at the top.

### Step 2: Configure Just-in-Time (JIT) VM Access
1. In the Defender for Cloud menu under **Cloud Security**, select **Workload protections**.
2. Click **Just-in-Time VM access** tile.
3. Select the **Configured** or **Not Configured** tab, select `VM-PROD-WEB` and click **Enable JIT on 1 VM**.
4. Set port rules:
   - Select port **3389** (RDP).
   - Set **Allowed source IPs** to `Per-request IP` (restricts connection to the requester's IP only).
   - Set **Max request time** to `3 hours`.
5. Click **Save**.
6. **Verify operation**: Try to RDP to the VM before requesting access (connection should fail). Go to the JIT console, select the VM, click **Request Access**, choose your port, and click **Open ports**. Try RDP again (connection succeeds).

### Step 3: Remediate an Open Storage Account Recommendation
1. In Defender for Cloud, click on **Recommendations**.
2. Expand the control **Secure management ports** or **Encrypt data in transit**.
3. Locate the recommendation: *"Storage accounts should restrict network access using virtual network rules"*.
4. Click on the recommendation. You will see a list of non-compliant storage accounts.
5. Select your target storage account.
6. Click the **Remediate** button or follow the manual steps provided:
   - Go to the Storage Account -> **Networking**.
   - Change "Allow access from" to **Enabled from selected virtual networks and IP addresses**.
   - Click **Save**.
7. Within 30-60 minutes, check the recommendation status. It will update to **Compliant**, increasing your Secure Score.

---
## Cheat Sheet

| Command / Setting | Purpose | Example |
|---|---|---|
| `az security location list` | Lists locations configured under security settings | `az security location list` |
| `az security alert list` | Lists all security alerts generated on the subscription | `az security alert list --state Active` |
| `az security pricing create` | Configures the pricing tier plan (Free/Standard) | `az security pricing create -n Servers --tier Standard` |
| `az security task list` | Retrieves active recommendations/tasks | `az security task list` |
| `ConfigurationMode` | Metaconfiguration setting defining agent behavior | Configured in Log Analytics agent settings |
| **Secure Score** | Overall security rating percentage of cloud assets | Viewable on the main Defender dashboard |

---
## Troubleshooting

| Problem | Cause | Fix |
|---|---|---|
| Secure Score percentage does not increase immediately after remediating an issue. | Defender for Cloud runs assessments on schedules (typically every 12 to 24 hours). | Wait for the next evaluation cycle or check the recommendation's "Last scanned" stamp in the console. |
| Cannot configure JIT VM access, error: "Missing write permissions on network security groups." | User does not have RBAC role permissions on the VM's Network Security Group. | Assign the user the `Network Contributor` role or a custom role with `Microsoft.Network/networkSecurityGroups/write` permissions. |
| Hybrid / On-premises server is not reporting security logs to Defender for Cloud. | The Azure Arc agent or Log Analytics workspace agent is disconnected or lacks outbound internet access. | Verify the system's local `himds` (Azure Arc) service is running. Confirm port 443 outbound to `*.his.arc.azure.com` is open. |
| Security alerts are not generating emails, even though high-severity alarms exist. | Email notifications have not been configured under Environment Settings. | Go to Defender console -> Environment Settings -> Email notifications. Enable emails for Owner/Contributor roles and set severity threshold to High. |
| JIT access button is greyed out for a virtual machine in the console. | The VM does not have an attached Network Security Group (NSG) at either the NIC or Subnet level. | Associate an NSG with the VM's network interface card (NIC) or parent subnet to enable dynamic port filtering. |

---
## Interview Questions

> [!question] L1 Question
> **Q:** What is the difference between Security Recommendations and Security Alerts in Defender for Cloud?
> **A:** **Security Recommendations** are part of the free Cloud Security Posture Management (CSPM). They are proactive suggestions identifying security weaknesses (like missing MFA or unencrypted disks) that help prevent future attacks. **Security Alerts** are part of the paid Cloud Workload Protection (CWPP) plans. They are reactive notifications triggered when active threats, attacks, or anomalous activities are detected on your running systems (such as a database brute-force attack).

> [!question] L2 Question
> **Q:** Explain how Just-in-Time (JIT) VM Access secures a virtual machine, and identify the firewall resource it manipulates.
> **A:** JIT VM Access blocks administrative inbound ports (like 22 for SSH and 3389 for RDP) on target VMs by default. It manipulates the **Network Security Group (NSG)** rules associated with the VM's network interface or subnet. When a user requests access and is authorized, Defender temporarily injects an inbound allow rule in the NSG restricting access to the user's specific source IP address for a pre-defined time window, automatically deleting the rule once the window expires.

> [!question] L3/Scenario Question
> **Q:** You are tasked with designing a security posture framework for an enterprise environment spanning Azure resources, AWS EC2 instances, and on-premises physical servers. How would you design a centralized management plane using Microsoft Defender for Cloud?
> **A:** 
> - **Situation:** Centrally securing a multi-cloud (Azure + AWS) and on-premises infrastructure.
> - **Task:** Design a unified security posture management framework.
> - **Action:** I will implement Microsoft Defender for Cloud as the central pane.
>   1. **For Azure**: Enable Microsoft Defender plans natively.
>   2. **For AWS**: Configure the **AWS Connector** in Defender for Cloud. This uses AWS IAM Roles and CloudFormation templates to automatically monitor configuration metadata and stream security logs.
>   3. **For On-Premises**: Deploy **Azure Arc** on the local physical/virtual servers. Azure Arc registers the local machines as hybrid resources in Azure.
>   4. **Agent Integration**: Once connected via Arc, deploy the Log Analytics / Azure Monitor agent to stream OS-level events and configure Microsoft Defender for Servers plans on those nodes.
> - **Result:** All recommendations, Secure Scores, and real-time threat alerts from Azure, AWS, and local servers are aggregated on a single console, allowing automated responses via Logic App playbooks globally.

---
## Related Notes
- [[04-Cloud-and-Security/08-Azure/AZ104-03 Azure Virtual Machines|AZ104-03 Azure Virtual Machines]] — Creating and managing Azure VMs JIT applies to.
- [[04-Cloud-and-Security/08-Azure/AZ104-04 Azure Virtual Networking|AZ104-04 Azure Virtual Networking]] — Detailing Network Security Groups (NSGs) manipulated by JIT rules.
- [[04-Cloud-and-Security/09-Security/Microsoft-Sentinel-SIEM|Microsoft-Sentinel-SIEM]] — Integrating Defender alerts into Sentinel for SOC operations.

---
*Tags: #cloud #azure #security #defender #az-104*
