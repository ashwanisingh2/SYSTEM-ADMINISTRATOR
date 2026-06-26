---
tags: [desktop-support, itsm, ticketing, jira, L1]
aliases: [jira-service-desk, itsm-04]
created: 2026-06-25
status: #complete
difficulty: #beginner
cert-relevant: #none
---

# ITSM-04: Jira Service Desk

> [!note] Overview
> This note covers Jira Service Management (formerly Jira Service Desk). It details queues, Customer Request Types, SLA configurations, transitions, custom Jira automation rules, and the use of Jira Query Language (JQL) to search and organize support tickets.

---
## Concept Overview
- **What it is** — Jira Service Management (JSM) is a service desk platform built by Atlassian that enables IT and support teams to receive, track, and resolve service requests, incidents, problems, and changes.
- **Why it matters** — Many modern software development companies use Jira for project tracking. JSM integrates helpdesk tickets directly with engineering sprint boards, making it easy to link support bugs to actual developer task backlogs.
- **Real job encounter** — Managing ticket queues, using JQL to find all tickets assigned to a retired system, updating ticket status, and configuring basic automated email responses.
- **L1 vs L2 vs L3 responsibility split:**
  - **L1 Resolution**: Receive incoming customer emails converted to tickets, verify details, categorize request types, reply to users using canned responses, and resolve basic password or hardware access requests.
  - **Escalation Trigger**: Escalate if the ticket breaches response SLAs, if the user reports a service-wide outage, or if the request requires direct code or database modifications.
  - **L2 Resolution**: Link related customer requests to a master bug ticket, transition issues through custom workflows, and adjust reporter permissions.
  - **L3 Resolution**: Configure project schemas, design custom screens and fields, write advanced JSM automation rules (e.g., auto-routing tickets based on text analysis), and integrate JSM with Slack or external APIs.

*Seedha simple mein: Jira Service Management (Jira Service Desk) ek and ticketing system hai jo developers aur support teams ko connect karta hai. Jab koi user system request karta hai, toh ek ticket (issue) banta hai jise support team queues me handle karti hai. Isme tickets ko search karne ke liye JQL use kiya jata hai.*

---
## Technical Deep Dive

### 1. Jira Service Management Concepts
- **Issues**: The base unit of work in Jira (e.g., INC-12, REQ-45).
- **Queues**: Filtered views of issues that help support teams prioritize and process tickets (e.g., "Assigned to me", "Unassigned", "High Priority").
- **Customer Portal**: The interface where end-users submit requests without seeing the complex agent-side interface.
- **Request Type**: How a customer views the ticket (e.g., "Get help with Wi-Fi"), which maps internally to an ITIL **Issue Type** (e.g., Incident).

### 2. JQL (Jira Query Language)
JQL is a flexible search syntax used to find specific issues within Jira:
- `project = "ITSD" AND status = "In Progress"`
- `assignee = currentUser() AND created >= -1d`
- `text ~ "password reset"` (Searches text fields for "password reset").

---
## Step-by-Step Lab

> [!warning] Pre-requisites
> - Access to a Jira Service Management Cloud instance (Atlassian offers free tier plans for up to 3 agents).
> - Permissions to create and transition issues.

### Step 1: Accessing the Queues and Creating a Ticket
We will simulate managing incoming customer tickets.

1. Log in to Jira Service Management.
2. Select your project (e.g., **IT Service Desk**).
3. Click the **Raise a request** button on the top right (or go to the Customer Portal).
4. Select **Get IT help** and fill out the fields:
   - **Summary**: `VPN connection drops every 5 minutes`
   - **Description**: `Since updating the GlobalProtect VPN client yesterday, it disconnects constantly. Work is blocked.`
5. Click **Create**.
**Expected Result:** A ticket number (e.g., `ITSD-101`) is generated.

### Step 2: Transitioning a Ticket Through the Workflow
1. Navigate back to the **Agent View** -> **Queues** -> **All Open**.
2. Locate and open ticket `ITSD-101`.
3. Set the **Assignee** field to yourself.
4. Click on the status dropdown (currently `To Do` or `New`) and change it to **In Progress**.
5. Add an internal comment:
```text
Checking VPN firewall logs and client version incompatibility.
```
6. If you need details from the user, transition the status to **Waiting for customer** and write a public reply:
```text
Hi, could you please provide your current GlobalProtect client version? You can find this in the 'About' section of the VPN application.
```
7. Click **Save**.
**Expected Result:** The SLA clock pauses during the "Waiting for Customer" state.

### Step 3: Resolving the Ticket
1. Once the user replies and you diagnose the issue, apply the fix.
2. Transition the ticket status to **Resolved** or **Done**.
3. Fill out the resolution fields:
   - **Resolution**: `Fixed`
   - **Linked Issues**: `None`
4. Add a public comment:
```text
Downgraded GlobalProtect VPN client to v6.1.2 as a workaround for the bug in v6.2.0. Verified connection is stable.
```
5. Click **Save**.
**Expected Result:** Ticket status moves to `Resolved` and the SLA clock stops.

---
## Cheat Sheet / Quick Reference

| Command / Query (JQL) | Scope | Purpose / Example |
|---|---|---|
| `project = "ITS" AND status = "To Do"` | JQL | Find all unstarted tickets in IT Service project |
| `assignee = currentUser()` | JQL | Filter tickets assigned to the active user |
| `priority = High OR priority = Critical` | JQL | Filter critical tickets |
| `updated >= -4h` | JQL | Find tickets updated in the last 4 hours |
| `resolution = Unresolved` | JQL | Find all open/active issues |
| `issuekey = "ITSD-105"` | JQL | Open a specific ticket directly |

---
## Troubleshooting Table

| Problem | Cause | Fix | Command |
|---|---|---|---|
| SLA timer did not stop after resolving the ticket. | The resolution field was not set, or the status transition was mismapped. | Ensure the status is set to the final "Done" category, and verify the Resolution field is populated. | N/A |
| "You do not have permission to view this issue." | The agent is not part of the Service Desk Team role, or issue security is restricted. | Request Project Admin to add you to the Service Desk Team role in Project Settings. | N/A |
| Jira search is missing relevant tickets. | Search is matching exact keywords, or project scope is wrong. | Switch to JQL mode and use wildcard search: `text ~ "network*"` to match network, networks, etc. | N/A |
| Customer did not receive the updates. | The comment was marked as "Internal comment" (yellow background) instead of "Reply to customer". | Copy the internal comment content and post it using the "Reply to customer" tab. | N/A |
| Automated routing rule is not executing. | The trigger conditions in Jira Automation rules are configured incorrectly. | Open Project Settings -> Automation, check the audit log for the rule to identify where it failed. | N/A |

---
## Interview Questions

> [!question] L1 Question
> **Q:** What is the difference between a Comment and an Internal Note in a Jira Service Management ticket?
> **A:** An **Internal Note** (marked with a yellow background) is only visible to agents and admins working inside Jira. It is used to collaborate internally without spamming the customer. A **Reply to Customer** (Standard Comment) is visible to the customer on the Portal and is sent to them via email notifications.

> [!question] L2 Question
> **Q:** Explain what JQL is and write a query to find all unresolved hardware tickets.
> **A:** **JQL (Jira Query Language)** is a query language used to search for issues in Jira. To find unresolved hardware tickets, I would write:
> `project = "ITSD" AND Component = "Hardware" AND resolution = Unresolved`
> (where `Component` maps to the hardware category).

> [!question] L3/Scenario Question
> **Q:** A developer complains that they cannot see helpdesk tickets assigned to them on their software sprint board. JSM is active. How do you integrate these?
> **A:** 
> - **Situation:** Helpdesk tickets isolated from the software development board.
> - **Task:** Connect JSM tickets with Jira Software Scrum/Kanban boards.
> - **Action:** 
>   1. **Issue Linking**: When L2 logs a bug, they can use the "Link Issue" feature to link JSM tickets directly to the developers' software project board as a block.
>   2. **JQL Board Filter**: I will edit the Scrum/Kanban board's saved filter query. By default, the board filter query is `project = DEV`. I will change it to:
>      `project = DEV OR (project = ITSD AND labels = "Dev-Fix")`
>   3. **Automate Transition**: Configure Atlassian Automation so that when the developer closes the linked ticket on the DEV board, JSM automatically transitions the user's ticket to "Resolved" and emails the customer.
> - **Result:** Seamless team collaboration, real-time ticket tracking, and reduced manual updates.

---
## Seedha Simple Mein
*Seedha simple mein: Jira Service Management support tickets ko handle karne ka software hai jo development and IT teams ke liye best hai. Har issue (ticket) ka unique ID hota hai (jaise ITSD-101). Hum customer ko visible "Reply" bhej sakte hain ya system admins ke aapas ke discuss ke liye "Internal Comment" likh sakte hain. Search karne ke liye JQL framework use kiya jata hai.*

---
## Related Notes
- [[05-Automation-and-Ticketing/11-ITSM-Ticketing/ITSM-03 ServiceNow-Basics|ITSM-03 ServiceNow-Basics]] — Comparison with ServiceNow ITSM.
- [[05-Automation-and-Ticketing/11-ITSM-Ticketing/Incident-and-Problem-Management|Incident-and-Problem-Management]] — Processing ticket frameworks.
- [[05-Automation-and-Ticketing/11-ITSM-Ticketing/Change-and-Asset-Management|Change-and-Asset-Management]] — Change tickets details.

---
*Tags: #desktop-support #itsm #ticketing #jira #L1*
