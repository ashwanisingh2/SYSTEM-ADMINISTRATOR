---
tags: [desktop-support, itsm, ticketing, change-control, L2]
aliases: [change-management, cmdb-guide, asset-lifecycle]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #itil-v4
---

# Change and Asset Management

---

## Concept Overview
- **What it is**: **Change Control (Change Management)** is the ITIL practice of maximizing successful IT modifications by assessing risk, securing approvals, and scheduling deployments. **Asset Management** is the practice of tracking corporate hardware/software lifecycles. A **Configuration Management Database (CMDB)** stores these assets as **Configuration Items (CIs)** and maps their dependencies.
- **Why it matters for a support engineer**: Untested changes cause 80% of system outages. Support engineers must follow change control policies before modifying servers, and keep the CMDB updated when replacing laptop motherboards, deployment monitors, or provisioning software licenses to prevent data gaps.
- **Where you encounter this in real job**: Submitting Change Requests (CRs) for updates, attending Change Advisory Board (CAB) meetings, scanning barcode assets, updating CMDB records, and decommissioning retired storage drives securely.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Updates asset statuses (e.g., "Deployed" or "In Repair"), assigns laptops, and executes pre-approved Standard Changes.
  - **L2**: Drafts Normal Change requests, conducts risk assessments, updates CMDB CI relationships, and audits asset inventories.
  - **L3**: Chairs CAB reviews, designs rollback plans for enterprise upgrades, manages CMDB auto-discovery configurations, and oversees data destruction compliance.

---

## Technical Deep Dive

### 1. ITIL Change Control Classifications
Changes are grouped into three categories, defining their approval paths:
1. **Standard Change**: 
   - *Characteristics*: Low risk, pre-approved, recurring, and well-documented.
   - *Examples*: Resetting a password, provisioning a new hire PC, or running a monthly disk cleanup.
2. **Normal Change**:
   - *Characteristics*: Medium-to-high risk. Requires formal scheduling, technical review, and approval from the **Change Advisory Board (CAB)**.
   - *Examples*: Upgrading a core network switch, moving a database server, or changing domain password policies.
3. **Emergency Change**:
   - *Characteristics*: Critical modifications required immediately to resolve a major outage (P1) or patch an active security breach.
   - *Approval*: Bypasses standard CAB scheduling; approved quickly by the **Emergency CAB (ECAB)**.

### 2. Configuration Items (CIs) & CMDB Relationships
- **Configuration Item (CI)**: Any component that must be managed to deliver an IT Service (e.g., a virtual server, an operating system, a firewall, a database, or a GPO).
- **CMDB**: The centralized database hosting all CI records. It maps **Dependencies**:

```
[Web Service App] ---> (Depends on) ---> [MySQL Database CI]
                                                |
                                                v (Runs on)
                                         [VM Server Host CI]
                                                |
                                                v (Connected to)
                                         [Core Network Switch CI]
```
- *Why it matters*: If you plan to reboot `VM Server Host`, the CMDB tells you instantly that `MySQL Database` and `Web Service App` will crash, allowing you to notify the correct business owners.

### 3. IT Asset Lifecycle Stages
Hardware assets transition through five lifecycle phases:
`Procurement -> Deployment -> Maintenance/Operations -> Retirement -> Disposal`
- **Data Sanitization**: Prior to physical disposal, storage drives must undergo certified wiping (using DOD 5220.22-M standards) or physical destruction (shredding), logging a **Certificate of Destruction** to prevent data leaks.

---

## Commands & Syntax

Ticketing systems use SQL databases to link active Changes to Configuration Items.

### SQL (Change Risk Dependency Audit)
```sql
-- Query to identify all active Normal Changes scheduled in the next 7 days affecting database CIs
SELECT 
    cr.change_id,
    cr.short_description,
    cr.start_date,
    cr.risk_level,
    ci.name AS configuration_item,
    ci.class AS ci_class
FROM 
    change_request cr
JOIN 
    configuration_item ci ON cr.configuration_item_id = ci.ci_id
WHERE 
    cr.state = 'Scheduled'
    AND cr.start_date BETWEEN GETDATE() AND DATEADD(day, 7, GETDATE())
    AND ci.class LIKE '%Database%'
ORDER BY 
    cr.start_date ASC;
```

---

## Real-World Scenarios

### Scenario 1: Unapproved Normal Change Triggers Office-Wide Outage
**User Complaint:** The marketing department logs 15 tickets: *"We lost connection to our project tracking portal. The web page displays 'Gateway Timeout 504'."*
**Your First 3 Checks:**
1. Check the active network connection to the server.
2. Search the Change Registry for active or recently completed changes.
3. Check the rollback plan defined in the Change Request.
**Diagnosis Steps:**
1. Ping the server hosting the portal. It is online.
2. Open the Change Registry in ServiceNow.
   - Find: A change request (CHG-00945 - "Firewall Upgrade") was marked as **Implemented** 20 minutes ago.
   - *Verify the details*: The change was submitted by a network technician as a "Standard Change" to bypass CAB review, underestimating the risk.
3. During the rule update, port 8080 traffic was blocked, cutting the connection to the web application.
**Root Cause:** A medium-risk firewall change was misclassified as a low-risk Standard Change, bypassing CAB review and causing an outage.
**Fix:**
1. Contact the technician who applied the change. Instruct them to execute the **Rollback Plan** instantly to restore service.
2. The tech reverts the firewall rule to the previous baseline state.
3. Access is restored.
4. Flag the ticket for a **Post-Implementation Review (PIR)**. Force the technician to re-submit the change as a **Normal Change** with a scheduled maintenance window and CAB review.
**Prevention:** Audit standard change categories annually to remove items that modify core network security parameters.
**Ticket Close Note:** "Reverted firewall rules using the rollback plan. Restored portal access. Escalated to PIR for misclassification. Closed."

### Scenario 2: Decommissioning and Sanitizing Storage Assets for Disposal
**User Complaint:** The IT Manager submits a ticket: *"We are replacing 10 old file servers. Please decommission these devices, update the CMDB status, wipe the hard drives securely, and prepare them for recycling."*
**Your First 3 Checks:**
1. Retrieve the serial numbers and Asset IDs of the 10 servers.
2. Wipes the drives using approved DOD sanitization tools.
3. Update the CMDB states to "Disposed".
**Diagnosis Steps:**
1. Locate the physical servers in the server room. Verify the asset tag barcodes (e.g. `AST-009841` through `AST-009850`).
2. Boot the servers using a secure PXE network tool running **DBAN (Darik's Boot and Nuke)**.
3. Configure the tool to run a 3-pass DOD-standard wipe on all SAS drives.
4. Once completed, extract the secure wipe logs and print the **Certificate of Destruction**.
**Root Cause:** Hardware retirement compliance.
**Fix:**
1. Physical destroy the drives if required (or place them in the secure recycling bin).
2. Log in to the CMDB portal. Search for the server CIs.
3. Change the asset status from `Active` to `Retired/Disposed`.
4. Attach the Certificate of Destruction PDF to the asset logs.
**Prevention:** Do not allow hardware assets to be recycled without a signed disk sanitization log.
**Ticket Close Note:** "Completed 3-pass DOD wipe on 10 decommissioned servers. Updated CMDB status to Disposed. Attached certificates. Closed."

---

## Critical Points

> [!danger] Never Do This
> - Never execute a Normal Change without a documented and tested **Rollback Plan**.
> - If an upgrade fails (e.g. database schema upgrade hangs), and you do not have a tested rollback plan (like restoring a snapshot), the service will remain broken indefinitely, resulting in major business outages. Always plan the rollback before applying the change.

> [!warning] Common Trap
> - Failing to update the CMDB when performing hot-swaps during emergency incidents.
> - If a VM host fails and you migrate disks to a new host without updating the CMDB, the database relationships become inaccurate. The next time an administrator shuts down the "empty" old host for maintenance, they will cause a silent production outage.

> [!tip] Senior Engineer Tip
> - Build a **Pre-Approved Change Catalog** for standard helpdesk operations. Tasks like "Add 10GB to C: drive" or "Create AD Security Group" should be documented once, approved by the CAB as standard, and then executed by L1 techs instantly without ticketing delays.

> [!success] Verification Steps
> - Run a CMDB dependency check to verify that a target server shows all running software services.
> - Verify that all completed changes have a signed post-implementation log entry.

> [!question] Interview Alert
> - "What is the difference between a Standard Change and a Normal Change?"
> - Answer: "A **Standard Change** is a low-risk, recurring, pre-approved change that follows a documented procedure, such as a user password reset. A **Normal Change** is a medium-to-high risk change that has no pre-approval. It requires formal risk assessment, scheduling, approval from the Change Advisory Board (CAB), and a documented rollback plan before implementation."

---

## Common Mistakes & Fixes

| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| Bypassing CAB for complex changes by calling them "emergencies" | Trying to avoid scheduling wait times | Restrict Emergency changes strictly to active outages (P1) or active security threats. |
| Neglecting the rollback plan in change forms | Assuming the upgrade will never fail | Write detailed rollback steps, including backup verify steps, in every change request. |
| Recycling old laptops with data intact | Lack of data destruction policies | Perform a certified wipe and record the Certificate of Destruction before recycling hardware. |

---

## Lab Exercise

**Objective:** Draft a formal Change Request (CR) document for a simulated Domain Controller OS upgrade, specifying risk assessments, testing plans, implementation steps, and rollback plans.
**Time Required:** 30 minutes
**Environment Needed:** A text editor.
**Pre-requisites:** Knowledge of Change Control phases.

**Steps:**
1. Create a markdown file `Change-Request-DC.md` in the artifacts directory.
2. Structure the document with the following mandatory sections:
   - **Header**: Change ID, Requester, Title, Target CI, Scheduled Window.
   - **Business Justification**: Why the upgrade is needed.
   - **Risk Assessment**: Potential impact on users (e.g. AD authentication drops).
   - **Pre-Implementation Test Plan**: How you tested the upgrade in the lab.
   - **Step-by-Step Implementation Plan**: Detailed commands and steps.
   - **Post-Implementation Verification**: How to confirm success (`repadmin`, `netdom`).
   - **Rollback Plan**: Steps to restore service if the upgrade fails (restoring VM state).
3. Fill out the details completely without placeholders.
   - Target CI: `DC-HQ-01` (Windows Server 2019 to 2022).
   - Rollback: Revert to VMware Snapshot.

**Success Criteria:** The Change Request is written completely, incorporating risk assessments, detailed step-by-step instructions, and a rollback plan.
**Common Failures:** The form lacks specific rollback steps or omits the post-implementation verification checklist.

---

## Interview Questions & Answers

### Basic (L1 Level)
**Q: What is a Configuration Item (CI) in IT service management?**
A: A CI is any component or asset that needs to be managed to deliver an IT service. Examples include physical servers, routers, software licenses, operating systems, and even Group Policy Objects.

**Q: What is the main purpose of the Asset Management lifecycle?**
A: The purpose is to track IT assets (hardware and software) from the time they are purchased, deployed, and maintained, until they are retired and disposed of, ensuring cost control and security compliance.

### Intermediate (L2 Level)
**Q: Why is it critical to have a rollback plan in a Change Request?**
A: A rollback plan is critical because system upgrades or configurations can fail unexpectedly in production, even if they worked in a lab. If a failure occurs, the rollback plan provides a step-by-step guide to restore the service to its previous stable state, minimizing downtime.

**Q: What is a CMDB (Configuration Management Database) and how does it help troubleshoot incident outages?**
A: A CMDB is a centralized database that stores CI records and maps their relationships and dependencies. It helps troubleshoot outages by showing you what services run on a crashed server (e.g., showing that a crashed VM hosts our payroll database), allowing you to identify the impact and contact the correct support teams.

### Advanced (L3/Senior Level)
**Q: You are planning to upgrade the core firewall firmware over the weekend. The change is classified as Normal and has been approved by the CAB. Describe your preparation and execution steps.**
A:
- **Situation**: Executing an approved Normal Change to upgrade core firewall firmware.
- **Task**: Secure the connection and ensure a recovery path is active during the upgrade.
- **Action**: First, I back up the current firewall configuration database. I schedule the change during our standard maintenance window (Saturday 12:00 AM to 4:00 AM) to minimize business impact. I notify the business stakeholders in advance. During the upgrade, I document my progress. If the firmware fails or corrupts, I execute the rollback plan: reload the previous firmware version from the alternate boot partition and import the backup configuration.
- **Result**: The upgrade completed successfully, and the verification tests confirmed all VPN tunnels were restored.

### HR / Behavioral
**Q: Tell me about a time you had to reject a change request or suggest a modification to a colleague's plan during a review meeting.**
A: During a CAB meeting, a systems engineer submitted a Normal Change to upgrade our active directory schema on a Friday afternoon. There were no rollback steps listed other than "reboot server." I raised a concern, stating that schema upgrades cannot be easily rolled back and Friday changes risk weekend outages. I requested they reschedule the change to Tuesday morning, perform a full system state backup of the PDC emulator first, and document the metadata cleanup steps. The engineer agreed, and we executed the change safely next week.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **What**: The practices of managing IT modifications (Change Control) and tracking IT components (Asset Management / CMDB).
> **Why**: Prevents outages, maintains audit compliance, and maps resource dependencies.
> **How**: Classify changes (Standard, Normal, Emergency), secure CAB approvals, write rollback plans, and update CMDB CI statuses.
> **Command**: SQL Change audits / DBAN sanitization
> **Interview Answer Starter**: "To manage risk during IT modifications, I enforce strict Change Control processes—ensuring Normal changes undergo CAB reviews and contain rollback plans—while keeping the CMDB updated..."

**Key Numbers to Remember:**
- Hard drive sanitization pass standard: 3-pass DOD 5220.22-M
- Types of ITIL changes: `3` (Standard, Normal, Emergency)
- Change review meeting name: CAB (Change Advisory Board)
- Retention period for soft deleted backup data: 14 days

**3 Things Interviewer Wants to Hear:**
- Never run a Normal Change without a documented, tested rollback plan
- CMDB maps the dependencies between infrastructure (hardware) and applications (software)
- Retired storage drives must undergo certified wiping before recycling

---

## Related Notes
- [[05-Automation-and-Ticketing/11-ITSM-Ticketing/ITIL-v4-and-SLA|ITIL v4 and SLA]] — Focuses on the ticketing priority matrix.
- [[05-Automation-and-Ticketing/11-ITSM-Ticketing/Incident-and-Problem-Management|Incident and Problem Management]] — Details resolving the incidents caused by bad changes.
- [[04-Cloud-and-Security/08-Azure/Azure-Backup|Azure Backup]] — Covers the snapshot restoration backups used in rollback plans.

---

## Tags
#desktop-support #itsm #ticketing #change-control #L2 #interview-topic #lab-complete #daily-use

