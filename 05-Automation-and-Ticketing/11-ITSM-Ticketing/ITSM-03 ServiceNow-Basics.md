---
tags: [desktop-support, itsm, ticketing, servicenow, L1]
aliases: [servicenow-basics, itsm-03]
created: 2026-06-25
status: #complete
difficulty: #beginner
cert-relevant: #itil-v4
---

# ITSM-03: ServiceNow Basics

> [!note] Overview
> This note covers the fundamentals of using ServiceNow (SNOW) as an enterprise IT Service Management (ITSM) tool. It details Incident, Problem, and Change modules, configuration item (CI) mapping, Service Level Agreements (SLAs), and the step-by-step lifecycle of an enterprise ticket.

---
## Concept Overview
- **What it is** — ServiceNow is a cloud-based ITSM platform designed to automate and manage IT infrastructure service workflows in compliance with the ITIL (Information Technology Infrastructure Library) framework.
- **Why it matters** — In enterprise IT, every issue, request, or system change must be logged for auditing, tracking, and capacity planning. ServiceNow acts as the single source of truth for all IT service operations.
- **Real job encounter** — Creating an Incident for a user experiencing a computer crash, assigning the ticket to the Desktop Support queue, categorizing the asset using the CMDB, and resolving the incident within the SLA timeline.
- **L1 vs L2 vs L3 responsibility split:**
  - **L1 Resolution**: Log user tickets, verify contact info, assign severity/priority, perform basic troubleshooting, search the Knowledge Base (KB), and assign tickets to L2 queues.
  - **Escalation Trigger**: Escalate if the ticket SLA is within 30 minutes of breaching, if the user is a VIP, or if the issue requires advanced system privileges (e.g. Server/Network outages).
  - **L2 Resolution**: Diagnose complex incidents, link multiple incidents to a single "Problem" ticket, update CMDB CI records, and resolve standard service requests.
  - **L3 Resolution**: Implement system-wide changes, approve Emergency Changes, customize ServiceNow workflow forms, build Service Catalog items, and perform root-cause analysis (RCA).

*Seedha simple mein: ServiceNow ek corporate complaint box aur records book hai. Agar kisi user ka computer kharab hota hai, toh support engineer ServiceNow par "Incident" create karta hai. Us ticket ko track kiya jata hai, and problem solve hone ke baad properly close kiya jata hai taaki company check kar sake ki issue kitni jaldi solve hua.*

---
## Technical Deep Dive

### 1. Incident vs. Problem vs. Change vs. Request
Understanding ticketing types under ITIL is vital:
- **Incident (INC)**: An unplanned interruption or reduction in the quality of an IT service (e.g., "Outlook not opening"). Goal: Restore normal service as quickly as possible.
- **Problem (PRB)**: The underlying cause of one or more incidents (e.g., "Exchange server memory exhaustion"). Goal: Prevent incidents from recurring.
- **Change (CHG)**: The addition, modification, or removal of anything that could affect IT services (e.g., "Upgrading firewall firmware"). Requires CAB (Change Advisory Board) approval.
- **Service Request (REQ/RITM)**: A formal user request for something to be provided (e.g., "Requesting a new laptop").

### 2. Core ServiceNow Terminology
- **CMDB (Configuration Management Database)**: A repository containing all details about IT assets (hardware, software, networks) known as **Configuration Items (CIs)**.
- **SLA (Service Level Agreement)**: A contract defining the expected resolution time (e.g., "P1 Critical - Resolve in 4 hours").
- **Knowledge Base (KB)**: A repository of articles containing troubleshooting steps and documentation used by L1/L2 agents to resolve common issues.
- **Sys_id**: A unique 32-character hexadecimal identifier assigned to every record in the database.

---
## Step-by-Step Lab

> [!warning] Pre-requisites
> - Access to a ServiceNow instance (or a ServiceNow Developer Instance - PDI).
> - Appropriate role permissions (e.g., `itil` role).

### Step 1: Create a New Incident
We will walk through the creation of a standard incident ticket.

1. Navigate to the filter navigator on the top left and type `Incident`.
2. Click on **Create New**.
3. Fill out the mandatory fields:
   - **Caller**: Type and select `Ashwani Singh`.
   - **Category**: Select `Hardware`.
   - **Subcategory**: Select `Computer`.
   - **Configuration Item (CI)**: Search and select the user's asset (e.g., `LAP-CORP-4521`).
   - **Short Description**: `User laptop is failing to boot - Stuck at BitLocker recovery screen`.
   - **Description**: `User reports that after the latest Windows updates, the laptop boots directly into the BitLocker Recovery key entry screen. User does not have their key.`
4. Set **Impact** to `3 - Low` and **Urgency** to `2 - Medium`.
**Expected Result:** The system automatically calculates **Priority** as `4 - Low` based on the impact-urgency matrix.

### Step 2: Assign and Investigate
1. Scroll down to the **Assignment Group** field and type `Desktop Support`.
2. Assign the ticket to yourself in the **Assigned to** field.
3. Click **Submit** or **Save**.
4. In the Work Notes section, add an update:
```text
Investigating the incident. Need to retrieve the BitLocker recovery key from Microsoft Entra / Active Directory.
```
5. Click **Post**.

### Step 3: Resolve the Incident
1. Once the recovery key is located and applied, update the ticket state.
2. Change **State** to `Resolved`.
3. Navigate to the **Resolution Information** tab:
   - **Resolution Code**: Select `Solved (Workaround)` or `Solved (Permanent)`.
   - **Resolution Notes**:
```text
Retrieved the BitLocker recovery key from Microsoft Entra portal. Provided the key to the user over a secure voice call. User successfully bypassed the recovery screen and verified Windows booted to login page.
```
4. Click **Resolve**.
**Expected Result:** Ticket status moves to `Resolved`. After 5 business days (standard policy), the ticket will auto-close.

---
## Cheat Sheet / Quick Reference

| Field / Action | Navigation / Code | Purpose |
|---|---|---|
| **Create Incident** | `incident.do` (in address bar) | Shortcut to open new incident form |
| **P1 Incident** | Critical Impact + Critical Urgency | High-priority ticket, pages on-call staff |
| **Watch List** | Field on form | Allows CC'ing additional users to email updates |
| **Work Notes** | Internal | Visible ONLY to IT staff (not the caller) |
| **Additional Comments**| Customer-facing | Email updates sent directly to the user |
| **CMDB Lookup** | `cmdb_ci.list` (in address bar) | View all active Configuration Items |
| **KB Search** | `kb_knowledge.list` | Open Knowledge Base article search |

---
## Troubleshooting Table

| Problem | Cause | Fix | Command |
|---|---|---|---|
| Ticket cannot be saved: "Required fields are empty." | Mandatory fields marked with red asterisks (or UI policies) are blank. | Complete all highlighted fields (e.g., Caller, Category, Description) before saving. | N/A |
| "Record Locked" alert at the top of the form. | Another support engineer is actively editing the same record. | Wait for the lock icon to clear, or contact the colleague to avoid overwriting updates. | N/A |
| ServiceNow page loads slow or freezes. | Local browser cache is bloated or outdated ServiceNow cookies are corrupt. | Clear browser cache, use an incognito window, or append `cache.do` to the instance URL (if admin). | N/A |
| SLA is running but ticket was resolved. | The ticket state was not changed to "Resolved" in the system. | Update the State field to "Resolved" to stop the SLA clock; "In Progress" continues SLA tracking. | N/A |
| Missing assignees or queue lists. | The logged-in user lacks the `itil` role or is not mapped to the group. | Escalate to ServiceNow Admin team to verify group memberships. | N/A |

---
## Interview Questions

> [!question] L1 Question
> **Q:** What is the difference between "Work Notes" and "Additional Comments" in a ServiceNow incident form?
> **A:** **Work Notes** are internal communications visible only to IT fulfillment staff (agents with the `itil` role). They are used to document diagnostic steps and collaborate. **Additional Comments** are customer-facing communications. Any text entered here is emailed directly to the caller and is visible to them in the Employee Service Portal.

> [!question] L2 Question
> **Q:** What is a CMDB in ServiceNow, and how does it help a support engineer during an outage?
> **A:** The **CMDB (Configuration Management Database)** is a database that stores Configuration Items (CIs)—such as servers, applications, laptops, and networking devices—along with their structural dependencies. During an outage, the CMDB helps engineers trace dependencies (e.g., which database server supports which financial application), allowing them to perform fast impact analysis and locate the root cause.

> [!question] L3/Scenario Question
> **Q:** A VIP customer calls L1 complaining that they cannot access a business-critical database. You notice a Change Request is active on that database. How would you handle this incident?
> **A:** 
> - **Situation:** VIP user reporting database access loss during an active Change window.
> - **Task:** Connect the Incident to the Change ticket and manage communication.
> - **Action:** 
>   1. **Investigate Change**: I will look up the Configuration Item (database) in ServiceNow and locate active Change Requests (CHG).
>   2. **Identify Status**: Check if the Change is "In Progress". If the scheduled window is active, the outage might be expected maintenance.
>   3. **Validate Incident Priority**: Since it's a VIP user, elevate the incident urgency and flag the dependency.
>   4. **Contact Change Owner**: Reach out to the implementer listed on the Change ticket to confirm if the service is down and check the expected return time.
>   5. **Link tickets**: Link the Incident to the Change record in ServiceNow.
>   6. **Communicate**: Inform the VIP caller about the scheduled maintenance window and provide the ETA.
> - **Result:** Duplicate investigations are avoided, cross-team visibility is maintained, and professional customer service is delivered.

---
## Seedha Simple Mein
*Seedha simple mein: ServiceNow ek digital ticketing software hai jisse companies me software, hardware, aur networking issues ko manage kiya jata hai. Incident ka matlab hota hai koi emergency problem (jaise printer stop hona), aur CMDB database batata hai ki kaun sa hardware kis user se connected hai. Incident ko start, work progress, aur finish hone par resolve state me dalna zaroori hai.*

---
## Related Notes
- [[05-Automation-and-Ticketing/11-ITSM-Ticketing/Incident-and-Problem-Management|Incident-and-Problem-Management]] — Processing ticket frameworks.
- [[05-Automation-and-Ticketing/11-ITSM-Ticketing/ITIL-v4-and-SLA|ITIL-v4-and-SLA]] — SLA management and service desk rules.
- [[05-Automation-and-Ticketing/11-ITSM-Ticketing/ITSM-04 Jira-Service-Desk|ITSM-04 Jira-Service-Desk]] — Alternative ticketing systems.

---
*Tags: #desktop-support #itsm #ticketing #servicenow #L1*
