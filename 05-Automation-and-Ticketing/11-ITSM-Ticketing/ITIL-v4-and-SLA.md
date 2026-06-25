---
tags: [desktop-support, itsm, ticketing, itil, L1]
aliases: [itil-guide, sla-guide, priority-matrix]
created: 2026-06-25
status: #complete
difficulty: #beginner
cert-relevant: #itil-v4
---

# ITIL v4 and Service Level Agreements (SLA)

---

## Concept Overview
- **What it is**: **ITIL v4 (Information Technology Infrastructure Library)** is the industry-standard framework for managing IT services. A **Service Level Agreement (SLA)** is a commitment between an IT service provider and a customer that defines service standards (e.g., system uptime, response times, and resolution times).
- **Why it matters for a support engineer**: Ticketing systems (ServiceNow, Jira Service Desk) enforce strict SLA timers. Support engineers must understand ITIL workflows to classify tickets correctly, prioritize issues based on Impact and Urgency, and prevent SLA breaches.
- **Where you encounter this in real job**: Triage of incoming tickets, managing SLA clocks on active incidents, handing off tasks using internal team agreements, and reporting weekly performance metrics.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Performs initial triage, assigns ticket Urgency/Impact, resets passwords, and resolves standard incidents within SLA response windows.
  - **L2**: Resolves escalated tickets within SLA resolution windows, coordinates cross-team hand-offs, and documents workarounds.
  - **L3**: Negotiates global SLAs with business stakeholders, defines Operational Level Agreements (OLAs), and reviews ticket trends to improve service availability.

---

## Technical Deep Dive

### 1. ITIL v4 Key Frameworks
ITIL v4 focuses on co-creating value. The two core frameworks are:
1. **The Four Dimensions of Service Management**:
   - *Organizations & People*: Roles, responsibilities, culture, and communication.
   - *Information & Technology*: The software (ticketing system), databases, and infrastructure.
   - *Partners & Suppliers*: Cloud hosting, ISPs, and third-party vendors.
   - *Value Streams & Processes*: Workflows to deliver value.
2. **The Service Value Chain**: The activities required to respond to demand (Plan, Improve, Engage, Design, Obtain/Build, Deliver/Support).

### 2. SLAs, OLAs, and Underpinning Contracts (UC)
To deliver service, ITIL defines three agreements:
- **Service Level Agreement (SLA)**: Agreement between **IT and the Business/Customer**. Defines targets for availability and resolution (e.g., "P1 incidents resolved in 2 hours").
- **Operational Level Agreement (OLA)**: Agreement between **Internal IT Teams** (e.g., a helpdesk OLA with the network team states: "Switch port requests configured within 24 hours").
- **Underpinning Contract (UC)**: Agreement between **IT and External Vendors/Suppliers** (e.g., an agreement with Comcast stating: "WAN link downtime restored within 4 hours").

```
  [Business/Customer]
         ^
         |  Service Level Agreement (SLA)
         v
     [IT Team] <--- Operational Level Agreement (OLA) ---> [Network Team]
         ^
         |  Underpinning Contract (UC)
         v
  [External ISP]
```

### 3. The Priority Matrix (Impact vs. Urgency)
Ticketing systems determine priority by combining two variables:
- **Impact**: The measure of how critical the issue is to the business (e.g., number of users or departments affected).
- **Urgency**: The measure of how long the business can tolerate the issue before suffering losses.
- **Priority = Impact x Urgency**:
  - **P1 (Critical)**: High Impact + High Urgency. (e.g., Main email server is down; entire company is blocked).
  - **P2 (High)**: Medium Impact + High Urgency. (e.g., VIP user cannot log in, or a single branch printer is down during payroll).
  - **P3 (Medium)**: Medium Impact + Medium Urgency. (e.g., Standard user cannot print).
  - **P4 (Low)**: Low Impact + Low Urgency. (e.g., User requests a software shortcut).

---

## Commands & Syntax

Ticketing systems (like ServiceNow or Jira) provide REST APIs to query and automate ticket data.

### PowerShell (ServiceNow API Query)
```powershell
# Define ServiceNow API connection parameters
$User = "api_user"
$Pass = "SecurePassword123!"
$Base64Auth = [Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes("$($User):$($Pass)"))
$Headers = @{ Authorization = "Basic $Base64Auth" }

# Query active P1 tickets nearing SLA breach (Response time remaining < 15 minutes)
$URI = "https://company.service-now.com/api/now/table/incident?sysparm_query=active=true^priority=1^sla_dueRELATIVELE@minute@ahead@15"

try {
    $Response = Invoke-RestMethod -Uri $URI -Method Get -Headers $Headers -ContentType "application/json" -ErrorAction Stop
    foreach ($Incident in $Response.result) {
        [PSCustomObject]@{
            IncidentNumber = $Incident.number
            Description    = $Incident.short_description
            AssignedGroup  = $Incident.assignment_group
            SLADue         = $Incident.sla_due
        }
    }
} catch {
    Write-Error "Failed to query ServiceNow API. Reason: $_"
}
```

---

## Real-World Scenarios

### Scenario 1: VIP User Account Lockout (Triage & Priority Classification)
**User Complaint:** A VIP user (VP of Marketing) calls: *"I am trying to log in to our client presentation dashboard, but it says my account is locked out. The meeting starts in 10 minutes. I need this reset immediately."*
**Your First 3 Checks:**
1. Check the VIP status of the user (identifying Urgency).
2. Determine the number of users affected (identifying Impact).
3. Classify the ticket priority based on the Impact/Urgency matrix.
**Diagnosis Steps:**
1. Check the active directory user status: the account `vpmarketing` is indeed locked.
2. *Evaluate Impact*: Only 1 user is locked out. Standard Impact = Low.
3. *Evaluate Urgency*: The user is a VIP presenting to a client in 10 minutes. Urgency = High.
4. *Matrix Calculation*: Low Impact (1 user) + High Urgency (VIP/Meeting) = **P2 (High Priority)**.
5. *Why not P1?* P1 requires enterprise-wide service loss. Downgrading to P2 still flags the ticket for immediate action, but prevents triggering corporate paging alerts.
**Root Cause:** Bad password attempts on a secondary device locked the VIP account.
**Fix:**
1. Unlock the account in Active Directory and reset the password.
2. Verify the VP can log in.
3. Keep the ticket logged as a P2 to track VIP support trends.
**Prevention:** Enroll VIP users in Self-Service Password Reset (SSPR) to allow self-recovery.
**Ticket Close Note:** "Unlocked VP account. Classified as P2 due to VIP urgency. Verified access. Closed."

### Scenario 2: Active SLA Breach Warning on Network Outage Ticket
**User Complaint:** A monitoring alert triggers: *"SLA Breach warning: Incident INC-009845 (Core Switch Down in Warehouse) has 10 minutes remaining on the resolution SLA. Current state: Assigned to Network Group, unacknowledged."*
**Your First 3 Checks:**
1. Check the active ticket status and assignment log.
2. Contact the on-duty Network Engineer to verify they are working the issue.
3. Trigger OLA escalation procedures.
**Diagnosis Steps:**
1. Open the incident: INC-009845. The ticket was assigned to the `Network-Ops` group 1.5 hours ago but has not been updated or accepted.
2. *Check SLA details*: Resolution SLA = 2 hours. Elapsed time = 1 hour 50 minutes.
3. The OLA between Helpdesk and Network-Ops states: "Network-Ops must acknowledge P1 tickets within 15 minutes." The OLA is breached.
**Root Cause:** Communication breakdown; the Network team was working on the physical hardware switch but forgot to update the ticket, leaving the SLA clock running.
**Fix:**
1. Call the Network Operations team room immediately. Confirm they are on-site replacing the switch.
2. Update the ticket: "Confirmed Network team is on-site replacing hardware. Work is in progress."
3. Toggle the ticket state to **In Progress** or **Work in Progress** (depending on tool, this may pause or update the SLA status indicators).
4. Resolve the ticket once the switch is replaced.
**Prevention:** Configure automated SMS/PagerDuty alerts to page the Network Manager if a P1 ticket is unacknowledged for 15 minutes.
**Ticket Close Note:** "Escalated to Network team via phone. Switch replaced. Restored connectivity. Closed."

---

## Critical Points

> [!danger] Never Do This
> - Never abuse the priority system by marking every standard user ticket as "P1 (Critical)" to get it resolved faster.
> - Overusing P1 ratings leads to "alert fatigue" in senior engineering groups, causing them to ignore actual critical outages (such as ransomware attacks or core database failures). Reserve P1 strictly for system-wide outages.

> [!warning] Common Trap
> - Confusing SLAs with OLAs.
> - An SLA is your contract with the customer. If you breach an SLA, the company may owe financial penalties. An OLA is the internal agreement between your support teams. If you breach an OLA, it means your internal workflows are slow, but it does not violate the customer contract directly.

> [!tip] Senior Engineer Tip
> - Set up "Pending Customer" states in your ticketing system. If you are waiting for a user to reply to your email or bring in their laptop, change the ticket state to **Awaiting User Info** or **Pending Customer**. This pauses the SLA resolution clock, preventing your team's metrics from being penalized for user delays.

> [!success] Verification Steps
> - Verify that ticket priority matches the corporate Impact/Urgency matrix guidelines.
> - Check that the SLA status indicator displays `In Progress` or `Paused` appropriately.

> [!question] Interview Alert
> - "Explain the difference between SLA, OLA, and Underpinning Contract (UC)."
> - Answer: "An **SLA** is a service agreement between IT and the customer defining system targets. An **OLA** is an internal agreement between different IT departments (e.g., helpdesk and database team) defining how they support each other. An **Underpinning Contract** is a legal agreement with an external vendor (e.g., Microsoft or Comcast) supporting our internal SLAs."

---

## Common Mistakes & Fixes

| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| Setting ticket priority based on user volume/anger | Users demand high priority | Apply the objective Impact vs. Urgency matrix to determine priority. |
| Letting the SLA clock run while waiting for a user response | Forgetting to pause state | Change ticket state to 'Pending Customer' to pause the SLA clock. |
| Escalating to L3 without completing L1 checks | Trying to close tickets quickly | Complete the L1 diagnostic checklist (reboot, network check) before escalating. |

---

## Lab Exercise

**Objective:** Audit ticket priorities using PowerShell, check if tickets are correctly prioritized based on simulated user counts and VIP status, and output an escalation report.
**Time Required:** 30 minutes
**Environment Needed:** A computer running PowerShell.
**Pre-requisites:** Basic scripting knowledge.

**Steps:**
1. Open PowerShell.
2. Create a mock ticket data list:
   ```powershell
   $Tickets = @(
       [PSCustomObject]@{ID="INC-1"; UsersAffected=150; VIP=$false; Priority="Low"}
       [PSCustomObject]@{ID="INC-2"; UsersAffected=1;   VIP=$true;  Priority="Low"}
       [PSCustomObject]@{ID="INC-3"; UsersAffected=1;   VIP=$false; Priority="Low"}
   )
   ```
3. Run an audit loop to correct the priorities based on business rules:
   - *Rule*: UsersAffected > 100 -> Priority = "Critical" (P1).
   - *Rule*: VIP is True -> Priority = "High" (P2).
   - *Rule*: UsersAffected = 1 and VIP is False -> Priority = "Medium" (P3).
   ```powershell
   $AuditedTickets = foreach ($Ticket in $Tickets) {
       $CorrectPriority = "Low"
       if ($Ticket.UsersAffected -gt 100) {
           $CorrectPriority = "Critical"
       }
       elseif ($Ticket.VIP -eq $true) {
           $CorrectPriority = "High"
       }
       else {
           $CorrectPriority = "Medium"
       }
       
       [PSCustomObject]@{
           ID              = $Ticket.ID
           UsersAffected   = $Ticket.UsersAffected
           VIP             = $Ticket.VIP
           LoggedPriority  = $Ticket.Priority
           AuditedPriority = $CorrectPriority
           Misclassified   = ($Ticket.Priority -ne $CorrectPriority)
       }
   }
   ```
4. Verification: Print the audited report:
   ```powershell
   $AuditedTickets | Format-Table -AutoSize
   ```
   - Confirm that INC-1 is classified as Critical, INC-2 as High, and INC-3 as Medium.

**Success Criteria:** The mock tickets are correctly evaluated and classified according to impact and urgency rules, outputting a table format.
**Common Failures:** Script logic errors if conditional statements (`if/elseif/else`) are ordered incorrectly.

---

## Interview Questions & Answers

### Basic (L1 Level)
**Q: What is an SLA (Service Level Agreement) in a helpdesk environment?**
A: An SLA is a committed target agreement between IT and the customer that defines service delivery standards, such as responding to a ticket within 1 hour and resolving it within 4 hours. It helps ensure consistent support delivery.

**Q: How do you determine the priority of a newly submitted ticket?**
A: I determine priority by combining two factors: **Impact** (how many users or systems are affected) and **Urgency** (how fast the business needs the issue resolved). I use the corporate Priority Matrix (Impact x Urgency) to assign P1, P2, P3, or P4.

### Intermediate (L2 Level)
**Q: What is an OLA (Operational Level Agreement) and how does it help a support team?**
A: An OLA is an agreement between internal IT groups that defines support responsibilities and timelines (e.g. Helpdesk team SLA is supported by Network team OLA to fix switch ports in 24 hours). It helps teams coordinate work and prevents tickets from being delayed during cross-team hand-offs.

**Q: What do you do if a ticket is about to breach its SLA, but you are waiting for a vendor to respond?**
A: I change the ticket status to **Pending Vendor** or **Awaiting Vendor** (which pauses the SLA timer in most ticketing systems, as we are waiting for an external contract partner to deliver). I document the vendor contact reference in the ticket logs and follow up based on our Underpinning Contract timeline.

### Advanced (L3/Senior Level)
**Q: You are auditing our incident ticket queues and notice that 20% of tickets breach their SLAs due to 'Wait for Customer' delays. How do you resolve this operational issue?**
A:
- **Situation**: Helpdesk metrics are failing SLA targets due to customer responsiveness.
- **Task**: Implement a system to automate follow-ups and stop SLA penalties.
- **Action**: First, I configure our ticketing system to automatically pause the SLA clock when a ticket is set to 'Awaiting User Info'. I configure an automated email system: when a ticket enters this state, the user receives an alert. The system sends reminder emails on days 3 and 5. On day 7, if the user has not replied, the system automatically closes the ticket with a note: "Closed due to inactivity."
- **Result**: The team's SLA breach rates dropped by 18%, and technicians spent less time chasing unresponsive users.

### HR / Behavioral
**Q: Tell me about a time you had to deal with a VIP user who demanded you violate our standard ticketing and priority rules. How did you handle it?**
A: A Director called demanding I fix their personal laptop instantly, bypassing 20 user tickets, and classification as a P1. I validated their urgency. I explained politely: "I cannot classify this as a P1 because the office network is running, but I am logging it as a P2 High Priority VIP ticket. I am assigning it directly to our next available tech, and we will contact you in 15 minutes." This kept our priority database accurate while providing excellent VIP customer service.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **What**: The framework (ITIL v4) and agreements (SLA, OLA, UC) managing IT service delivery.
> **Why**: Ensures consistent support speeds, defines priorities, and tracks helpdesk performance.
> **How**: Matrix impact vs. urgency, coordinate OLA hand-offs, and use paused states (Pending Customer) to preserve metrics.
> **Command**: ServiceNow REST API endpoints
> **Interview Answer Starter**: "To manage ticketing operations effectively, I classify incidents based on the priority matrix, monitoring SLA clocks and leveraging OLAs..."

**Key Numbers to Remember:**
- ITIL dimensions: `4`
- Service Value Chain activities: `6`
- Priority calculations: `Impact x Urgency`
- Default SLA target window for P1: 2 - 4 hours

**3 Things Interviewer Wants to Hear:**
- The difference between SLAs (customer) and OLAs (internal)
- How to pause the SLA clock using 'Pending Customer' states
- Maintaining objective priority matrix metrics rather than user-anger priorities

---

## Related Notes
- [[05-Automation-and-Ticketing/11-ITSM-Ticketing/Incident-and-Problem-Management|Incident and Problem Management]] — Focuses on resolving tickets.
- [[05-Automation-and-Ticketing/11-ITSM-Ticketing/Change-and-Asset-Management|Change and Asset Management]] — Details change controls.
- [[06-Career-Growth/12-Interview-Prep/Top-10-Behavioral-Questions|Top 10 Behavioral Questions]] — Covers customer handling behavioral skills.

---

## Tags
#desktop-support #itsm #ticketing #itil #L1 #interview-topic #lab-complete #daily-use
