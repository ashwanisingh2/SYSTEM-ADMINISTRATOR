---
tags: [desktop-support, itsm, ticketing, incident-management, L1]
aliases: [problem-management, rca-guide, workarounds]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #itil-v4
---

# Incident and Problem Management

---

## Concept Overview
- **What it is**: **Incident Management** is the ITIL practice of restoring normal service operation as quickly as possible following an unplanned interruption, minimizing business impact. **Problem Management** is the practice of identifying and analyzing the root cause of recurrent incidents to prevent them from happening again.
- **Why it matters for a support engineer**: A support engineer must know the difference between an incident and a problem. Fixing a single user's crashed Outlook is Incident Management. Investigating why 50 users' Outlook clients crashed after a patch update is Problem Management.
- **Where you encounter this in real job**: Resolving support tickets, creating workaround entries in the Known Error Database (KEDB), participating in post-incident reviews, and writing Root Cause Analysis (RCA) logs.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Identifies recurring issues, logs incidents, applies documented workarounds, and escalates to L2.
  - **L2**: Investigates complex incident histories, applies deep-level workarounds, and submits Problem tickets to initiate root cause investigations.
  - **L3**: Conducts root cause analysis (RCA) meetings, designs permanent system fixes, audits the KEDB database, and writes post-mortem reports.

---

## Technical Deep Dive

### 1. Incident Management Lifecycle
The structured lifecycle of an incident consists of the following phases:

```
[Incident Logged] ---> [Categorized & Prioritized] ---> [Initial Diagnosis]
                                                             |
   [Closure] <--- [Resolution & Recovery] <--- [Escalated (If needed to L2/L3)]
```

1. **Identification**: Alert triggers or user contacts helpdesk.
2. **Logging**: Record incident metadata (User, device, short description).
3. **Categorization**: Route to correct group (e.g., Network, Hardware).
4. **Prioritization**: Determine Impact x Urgency.
5. **Initial Diagnosis**: L1 runs troubleshooting checklists.
6. **Escalation**: Functional (to L2/L3 tech groups) or Hierarchical (notifying managers for P1s).
7. **Resolution**: Apply fix or workaround.
8. **Closure**: Confirm with user that service is restored.

### 2. Incident vs. Problem Management
A fundamental ITIL distinction:
- **Incident**: An unplanned interruption to an IT service (focuses on **Speed**).
  - *Goal*: Restore service fast (even if it's a temporary workaround).
- **Problem**: The underlying cause of one or more incidents (focuses on **Quality & Root Cause**).
  - *Goal*: Find the root cause and implement a permanent fix.

### 3. Workarounds, KEDB, and RCA
- **Workaround**: A temporary fix that reduces or eliminates the impact of an incident without resolving the root cause (e.g., restarting a service).
- **Known Error Database (KEDB)**: A repository storing documented workarounds and known software bugs. Managed by Problem Management.
- **Root Cause Analysis (RCA) Techniques**:
  - **5 Whys**: Asking "Why?" recursively (usually 5 times) to drill down from symptoms to the root cause.
  - **Ishikawa (Fishbone) Diagram**: Visualizing potential causes across categories (People, Process, Technology, Environment).

---

## Commands & Syntax

Ticketing metrics are stored in SQL relational databases. Support leads run SQL queries to track recurring incidents.

### SQL (Incident Audit Query)
```sql
-- Query to identify recurring incidents by category to initiate Problem tickets
SELECT 
    category,
    short_description,
    COUNT(incident_id) AS incident_count,
    MIN(sys_created_on) AS first_seen,
    MAX(sys_created_on) AS last_seen
FROM 
    incident
WHERE 
    active = false 
    AND sys_created_on >= DATEADD(day, -30, GETDATE())
GROUP BY 
    category, 
    short_description
HAVING 
    COUNT(incident_id) >= 5
ORDER BY 
    incident_count DESC;
```

---

## Real-World Scenarios

### Scenario 1: Recurrent Print Spooler Crashes (Incident to Problem Transition)
**User Complaint:** Helpdesk receives 12 separate tickets in 2 hours: *"My local printer isn't printing, and when I check, my Windows Print Spooler service is stopped."*
**Your First 3 Checks:**
1. Check if the issues originate from a single office location.
2. Search the Known Error Database (KEDB) for existing spooler workarounds.
3. Link the incidents to a new Master Problem Ticket.
**Diagnosis Steps:**
1. L1 technicians start the Spooler service for each user (Incident Management - restores service).
2. Within 30 minutes, the Spooler services crash again.
3. *Identify the Problem*: Multiple identical incidents indicating a systemic root cause.
4. L2 creates a **Problem ticket** (PRB-00845) in ServiceNow. All 12 user incidents are linked to this Problem ticket.
5. In the KEDB, the team finds a workaround: "If Spooler crashes on v3 HP drivers, change driver to MS Class Driver." Helpdesk applies this workaround to restore service immediately.
**Root Cause:** A newly installed network printer driver conflicted with the local Windows OS patch version.
**Fix:**
1. L3 investigates the printer server logs (RCA).
2. The team deploys a signed v4 print driver to the print server, replacing the conflicting driver.
3. The Problem ticket is marked as resolved, which automatically resolves and closes all 12 linked user incidents.
**Prevention:** Test print drivers in a lab environment before deploying to production servers.
**Ticket Close Note:** "Linked incidents to PRB-00845. Updated print drivers on server. Auto-closed all linked child incidents. Closed."

### Scenario 2: Executing "5 Whys" Root Cause Analysis for a Database Outage
**User Complaint:** A major database server went offline, blocking the billing department. The Incident team restored service by rebooting the VM. You are assigned to complete the RCA report.
**Your First 3 Checks:**
1. Collect server logs surrounding the exact timestamp of the crash.
2. Interview the DBA and System Admin on duty.
3. Apply the "5 Whys" methodology to determine the root cause.
**Diagnosis Steps (5 Whys Analysis):**
1. *Why did the database crash?* 
   - Answer: The database engine ran out of memory.
2. *Why did the engine run out of memory?*
   - Answer: A massive, unoptimized reporting query was executed.
3. *Why was an unoptimized reporting query executed?*
   - Answer: A new marketing script was scheduled to run during peak billing hours.
4. *Why was the script scheduled during peak billing hours?*
   - Answer: The developer who wrote the script did not coordinate with the database administrator.
5. *Why did the developer not coordinate with the database administrator?*
   - Answer: There is no formal Change Control process requiring DBA approval for database script scheduling.
**Root Cause:** Absence of a Change Control process requiring DBA approval for scheduling developer scripts on production databases.
**Fix:**
1. Update the database settings to limit memory consumption per query.
2. Establish a mandatory Change Control policy: all SQL scripts run on production databases must undergo DBA review and be scheduled during maintenance windows.
**Prevention:** Run daily SQL query performance audits.
**Ticket Close Note:** "Completed 5 Whys analysis. Root cause identified as lack of script scheduling Change Control. Policy updated. Closed."

---

## Critical Points

> [!danger] Never Do This
> - Never close an Incident ticket with a workaround (e.g., "Rebooted server") without investigating *why* the server crashed if it happens repeatedly.
> - Rebooting handles the incident but ignores the problem. If you ignore the problem, the server will crash again, leading to recurrent downtime and SLA breaches. Always open a Problem ticket.

> [!warning] Common Trap
> - Assuming that L1 helpdesk technicians should spend hours investigating the code or root cause of an incident.
> - Incident Management is about **Restoring Service Fast**. If a workaround works (e.g., restarting a service or setting up a temp profile), L1 should apply it, resolve the incident, and escalate the root cause to Problem Management.

> [!tip] Senior Engineer Tip
> - Build a **Known Error Database (KEDB)** directly inside your ticketing platform. Link the KB articles to incident templates. When a technician opens a ticket and types "Outlook crash", the matching workaround from the KEDB should pop up automatically, reducing resolution time (MTTR).

> [!success] Verification Steps
> - Verify that all related incidents are linked to the master Problem ticket in the ticketing console.
> - Audit the closed Incident ticket notes to verify the resolution specifies whether a permanent fix or workaround was applied.

> [!question] Interview Alert
> - "Explain the difference between Incident Management and Problem Management."
> - Answer: "Incident Management focuses on speed: restoring normal service operations as quickly as possible using workarounds or quick fixes. Problem Management focuses on quality: investigating the root cause of recurring incidents to implement permanent fixes and prevent future occurrences."

---

## Common Mistakes & Fixes

| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| Leaving recurrent incidents unlinked | Lack of ticketing training | Link identical incidents to a single Master Problem ticket to track trends. |
| Spent too long investigating root cause during an incident | Technical curiosity over service priority | Apply a workaround to get the user working, then research the root cause later. |
| Forgetting to document workarounds in the KEDB | Assuming team members know the fix | Document all verified workarounds in the KEDB to speed up future resolutions. |

---

## Lab Exercise

**Objective:** Construct a structured Post-Mortem / Root Cause Analysis (RCA) report document for a simulated major network outage, outlining timelines, root causes, permanent actions, and prevention steps.
**Time Required:** 30 minutes
**Environment Needed:** A text editor or markdown file.
**Pre-requisites:** Knowledge of incident lifecycle.

**Steps:**
1. Open a markdown file `RCA-Report.md` in the artifacts directory.
2. Structure the report with the following mandatory sections:
   - **Header**: Incident ID, Date, Priority, System Affected.
   - **Executive Summary**: Brief description of the outage and business impact.
   - **Timeline**: Chronological logs from alert trigger to service recovery.
   - **5 Whys Analysis**: Step-by-step drilling to the root cause.
   - **Permanent Corrective Actions**: What was fixed.
   - **Preventative Measures**: Actions to stop it happening again.
3. Write a mock incident scenario:
   - Outage: Corporate Active Directory replication failure locked out remote users.
   - Root Cause: A network engineer changed firewall rules, blocking port 9389.
4. Draft the corrective actions:
   - Revert firewall change (Immediate).
   - Implement change control audit checks (Permanent).

**Success Criteria:** The RCA report is written completely, incorporating the 5 Whys analysis, timeline logs, and permanent fixes without placeholders.
**Common Failures:** The report lacks detail or lists vague root causes (like "Network issue" instead of "Firewall port 9389 blocked").

---

## Interview Questions & Answers

### Basic (L1 Level)
**Q: What is the primary goal of Incident Management?**
A: The primary goal of Incident Management is to restore normal service operations as quickly as possible following an outage or issue, minimizing the impact on business productivity.

**Q: What is a workaround, and can you give an example?**
A: A workaround is a temporary fix that gets a user working again without resolving the underlying root cause. An example is restarting a stopped print spooler service to allow a user to print, even though we haven't fixed the corrupted print driver causing the crash.

### Intermediate (L2 Level)
**Q: What is a Known Error Database (KEDB) and who maintains it?**
A: The KEDB is a database containing records of all known bugs, configuration issues, and verified workarounds. It is created and maintained by the Problem Management team to help helpdesk technicians resolve recurring incidents faster.

**Q: A user reports a recurring issue that has occurred three times this week. How do you handle this from an ITSM standpoint?**
A: I resolve the user's immediate incident using the documented workaround to get them back to work. Then, I search the ticketing system to see if other users have reported the same issue. If so, I open a new **Problem ticket**, link all the related incidents to it, and assign it to the L3 systems team to perform a root cause analysis.

### Advanced (L3/Senior Level)
**Q: Explain how you would lead a Root Cause Analysis (RCA) meeting after a major P1 database outage.**
A:
- **Situation**: Conducting an RCA after a critical production database failure.
- **Task**: Identify the true root cause and prevent future occurrences.
- **Action**: I gather the systems administrator, network engineer, and database administrator. I review the incident timeline from logging to resolution. I use the **5 Whys** technique to drill down past the immediate symptoms (e.g., VM crashed) to the system cause (e.g., memory limits exceeded) and the organizational cause (e.g., lack of change control). I document these findings in an RCA report.
- **Result**: We assign corrective actions (with owners and due dates) to update memory limits and implement change controls, preventing a recurrence of the outage.

### HR / Behavioral
**Q: Tell me about a time you had to deal with a system outage that was caused by a mistake you made. How did you handle it?**
A: I was configuring a GPO update and misconfigured a setting that blocked our sales team from opening their ERP software. Within 5 minutes, tickets flooded the helpdesk. I realized my change had caused the issue. I did not hide it; I immediately notified my manager: "I made a mistake in the GPO. I am reverting the change now." I forced a policy update on the workstations, restoring access within 10 minutes. I updated the incident logs, wrote a post-mortem report, and recommended a policy requiring all GPO changes be tested on a pilot group first.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **What**: The core ITSM practices: Incident (restore service fast) and Problem (find the root cause).
> **Why**: Prevents service downtime, resolves recurring tickets, and maintains service quality.
> **How**: Lifecycle logging, link incidents to problems, use KEDB workarounds, and complete 5 Whys RCAs.
> **Command**: SQL Incident audits / KEDB search
> **Interview Answer Starter**: "To manage support operations, I run Incident lifecycle steps to restore user access quickly, while systematically escalating recurring tickets to Problem Management..."

**Key Numbers to Remember:**
- Default number of "Whys" in RCA: 5
- System table for incident records in SQL: `incident`
- Days held in first-stage recycle bin: 93 days (AD/SharePoint equivalent)
- Target time to resolve a P1 Incident: 2 - 4 hours

**3 Things Interviewer Wants to Hear:**
- Incident focuses on speed (workarounds); Problem focuses on cause (permanent fixes)
- Linking child incidents to a Master Problem consolidates resolution tasks
- RCA must find systemic process gaps, not just blame individual users

---

## Related Notes
- [[05-Automation-and-Ticketing/11-ITSM-Ticketing/ITIL-v4-and-SLA|ITIL v4 and SLA]] — Details the SLA frameworks backing incident response.
- [[05-Automation-and-Ticketing/11-ITSM-Ticketing/Change-and-Asset-Management|Change and Asset Management]] — Outlines the change controls that prevent problems.
- [[06-Career-Growth/12-Interview-Prep/Top-20-Scenario-Based-Questions|Top 20 Scenario-Based Questions]] — Preps for troubleshooting case studies.

---

## Tags
#desktop-support #itsm #ticketing #incident-management #L1 #interview-topic #lab-complete #daily-use
