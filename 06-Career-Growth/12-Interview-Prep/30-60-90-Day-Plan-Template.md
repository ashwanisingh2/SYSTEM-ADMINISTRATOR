---
tags: [desktop-support, interview-prep, onboarding, career, L1, L2, L3]
aliases: [onboarding-plan, thirty-sixty-ninety]
created: 2026-06-25
status: #complete
difficulty: #beginner
cert-relevant: #itil-v4
---

# 30-60-90 Day Plan Template for IT Support Engineers

---

## Concept Overview
- **What it is**: A career onboarding framework outlining how a Desktop Support or Systems Engineer will transition from a new hire (Day 1-30) to an independent contributor (Day 31-60) to a proactive systems optimizer (Day 61-90).
- **Why it matters**: Presenting a 30-60-90 day plan during interviews demonstrates foresight, extreme organization, and immediate value to hiring managers.
- **Where you encounter this in real job**: During the onboarding phase at any enterprise, MSP, or internal IT department, and during probation period reviews.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Focuses on learning ticketing queues, shadowing calls, mastering internal applications, and meeting basic SLAs.
  - **L2**: Focuses on understanding network topology, diagnosing escalation bottlenecks, and updating stale documentation.
  - **L3**: Focuses on auditing security policies, identifying automation opportunities, implementing backup routines, and standardizing deployments.

---

## Technical Deep Dive

### The 30-60-90 Day Phase Model

```
   [Days 1-30: Learn & Listen]     --->   [Days 31-60: Do & Contribute]  --->  [Days 61-90: Run & Optimize]
  - Shadow queues & procedures            - Run independent tickets             - Identify automation tasks
  - Map AD/Network architecture            - Handle L2 escalations               - Audit security policies
  - Define SLA milestones                 - Improve documentation               - Implement scripting projects
```

#### 1. Days 1-30: The Learning Phase (Absorption)
- **Primary Goal**: Understand the company's technical infrastructure, standard operating procedures (SOPs), and user landscape without breaking systems.
- **Milestones**: Map active Active Directory paths, learn the ITSM ticketing tool (ServiceNow/Jira), shadow senior engineers, and complete all mandatory security compliance training.

#### 2. Days 31-60: The Contributing Phase (Independence)
- **Primary Goal**: Handle core responsibilities with minimal supervision.
- **Milestones**: Own and resolve standard tickets, participate in rotational support schedules, resolve L2 escalations, and write or update at least three troubleshooting runbooks.

#### 3. Days 61-90: The Optimizing Phase (Proactive Value)
- **Primary Goal**: Drive efficiency and lead minor projects that optimize the IT environment.
- **Milestones**: Identify repetitive tickets and automate them using PowerShell, run security scans, propose cost-saving licensing changes, and lead junior onboarding sessions.

---

## Commands & Syntax

### PowerShell: Environment Discovery Commands (First 30 Days)
Run these commands on day one to understand the domain and environment you are support:
```powershell
# 1. Identify the Domain Controllers and Active Directory Forest info
Get-ADDomainController -Filter * | Select-Object Name, IPAddress, OperatingSystem

# 2. Query basic networking settings on the local server/workstation
Get-NetIPConfiguration | Select-Object InterfaceAlias, IPv4Address, IPv4DefaultGateway

# 3. Retrieve local group memberships to audit administrative permissions
Get-LocalGroupMember -Group "Administrators" | Select-Object Name, PrincipalSource
```

### CMD / Run Box
```cmd
REM Check GPO registry source path applied to client machines
gpresult /h C:\temp\gpreport.html
```

### GUI Path
> Active Directory Users & Computers -> View -> Advanced Features (to inspect system containers)
> Azure Portal -> Entra ID -> Sign-in logs (to check authentication paths)

---

## Real-World Scenarios

### Scenario 1: Onboarding at a Disorganized MSP
**User Complaint**: You join a Managed Service Provider (MSP) as an L2 engineer and find the ticket queue has 200 open, unassigned tickets, and the client documentation is severely outdated.
**Your First 3 Checks**:
1. Check the oldest tickets in the queue to identify systemic outages.
2. Identify the most critical client accounts by SLA tier.
3. Review the available internal knowledge base for baseline login procedures.
**Diagnosis Steps**:
1. Run a query on the ticketing console: filter by `CreatedDate` ascending and `Status` = Open.
2. The oldest tickets are mostly password resets and slow client reports.
3. Locate the client contact list: "Client-A" represents 50% of the MSP's revenue but has 40 pending tickets.
**Root Cause**: Lack of structured queue management and missing documentation has caused L1 agents to ignore tickets they cannot solve.
**Fix (Using the 30-60-90 Day Framework)**:
- **First 30 Days**: Shadow the L1 queue. Map client environments. Clear the password reset backlogs to reduce queue noise.
- **Next 30 Days**: Draft documentation for Client-A's custom software configurations. Start resolving the intermediate escalations.
- **Last 30 Days**: Set up a self-service password reset portal for Client-A, reducing their ticket volume by 40% and preventing future backlog spikes.
**Prevention**: Establish weekly ticket triage sessions to prevent queues from exceeding manageable thresholds.

### Scenario 2: Sole L3 Engineer Inherits a Hybrid Cloud Network
**User Complaint**: The senior engineer resigned suddenly. You step in as the sole L3 administrator. The CEO complains that cloud operations are slow, and they do not know what resources are running in Azure.
**Your First 3 Checks**:
1. Run an Azure Resource inventory report.
2. Check the backup job statuses for all on-prem and cloud VMs.
3. Review global administrator permissions in Entra ID.
**Diagnosis Steps**:
1. Log into the Azure Portal and run resource queries.
2. Check Azure Backup vaults: 3 critical databases have not completed backup runs in 14 days.
3. Check Entra ID: 12 former employees still have active Global Administrator accounts.
**Root Cause**: The previous administrator did not configure alerting or perform identity cleanup before leaving.
**Fix (Using the 30-60-90 Day Framework)**:
- **First 30 Days**: Immediately disable access for all inactive Global Administrator accounts. Remediate the database backup failures and configure automated email alerts.
- **Next 30 Days**: Audit subscription spending. Run costs optimization assessments in Azure Advisor.
- **Last 30 Days**: Build a PowerShell VM automation script to auto-shutdown non-production instances overnight, saving 30% on compute billing.
**Prevention**: Implement monthly user access reviews (UARs) and regular backup validation drills.

---

## Critical Points

> [!danger] Never Do This
> Do not attempt to rewrite company infrastructure, policies, or scripts in your first 30 days. Making changes before understanding the business logic is a primary cause of client outages.

> [!warning] Common Trap
> Focusing only on technical tasks and ignoring relational networking. Use your first 30 days to build relationships with key department heads; they are your primary customers.

> [!tip] Senior Engineer Tip
> Document everything you learn during your first 30 days. If you find a gap in the onboarding wiki, update it. This immediately shows the team that you are a proactive contributor.

> [!success] Verification Steps
> To verify onboarding success at Day 30:
> 1. Can you explain the mail routing path?
> 2. Do you have correct permissions to access the required tools?
> 3. Have you met with your manager to align on key performance indicators (KPIs)?

> [!question] Interview Alert
> "What will you do in your first week on the job?"
> - **Answer**: "I will focus on listening and learning. I will set up my workspace, complete mandatory training, shadow colleagues to learn the ticketing workflow, and meet key team members to understand immediate priorities."

---

## Common Mistakes & Fixes

| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| Fearing to ask questions | Afraid of looking incompetent | Ask questions after doing initial research: "I checked the wiki for X but couldn't find the configuration key. Could you point me to it?" |
| Skipping documentation | Rushing to close tickets | Spend an extra 5 minutes adding detailed notes to every ticket. |
| Over-committing to projects | Trying to impress the team early | Focus on mastering the basics before proposing large system changes. |

---

## Lab Exercise

**Objective**: Customize the 30-60-90 Day Plan for a Specific Desktop Support Role.
**Time Required**: 15 minutes
**Environment Needed**: Markdown editor.

**Steps**:
1. Open a new file named `My-Onboarding-Plan.md`.
2. Fill out the target goals for your target role:
   - **Target Company Name**: [Insert Company]
   - **Ticketing Tool Used**: [Jira / ServiceNow / SysAid]
   - **Week 1 Objective**: Get credentialed, configure local workstation, map basic printer queues.
   - **Day 30 KPI**: Independently close 15 tickets per day with 90% SLA compliance.
   - **Day 60 KPI**: Create 2 runbooks for client-side VPN and BitLocker resets.
   - **Day 90 KPI**: Script a automated computer cleanup task using PowerShell.

**Pass Criteria**: The customized document has realistic, time-bound technical deliverables.

---

## Interview Questions & Answers

### Basic / L1 Level

#### Q1: If hired, what is your plan for the first 30 days?
**A**: "In the first 30 days, my goal is to absorb as much information as possible. I will focus on understanding the ticketing system, shadowing senior technicians, and mastering your specific hardware configurations. I want to become independent on basic tickets like password resets and software deployments by the end of week three, minimizing the load on the rest of the team."

#### Q2: How do you know if your onboarding was successful?
**A**: "I measure onboarding success through performance metrics and team feedback. Successful onboarding means I am meeting my daily SLA targets, resolving tickets without constant escalations, and my documentation updates are helping other team members solve tickets faster."

---

### Intermediate / L2 Level

#### Q3: How do you plan to handle the transition from our legacy ticketing system to ServiceNow in your second month?
**A**: "During my second month (the 60-day phase), I will use my experience with ITSM transitions to ease the team's shift. I will focus on mapping our old category workflows to ServiceNow, documenting the new escalation paths, and helping L1 technicians get comfortable with the new interface, ensuring our average ticket resolution time does not spike during migration."

#### Q4: How would you prioritize updating our knowledge base during your first 60 days?
**A**: "I will prioritize based on ticket volume. I'll query the ticket history to find the top three issues that generate the most support calls (e.g., VPN drops, printer connectivity). I will update those runbooks first with clear screenshots and commands, as resolving these tickets faster provides the highest immediate return for the team."

---

### Advanced / L3 Level

#### Q5: Describe your 90-day plan for reducing our infrastructure licensing costs.
**A**:
- **Situation**: You are hired to optimize a growing M365 tenant containing unmanaged licenses.
- **Task**: Identify and remove stale cloud accounts to reduce monthly licensing fees.
- **Action**: In the 90-day phase, I will write a PowerShell script to audit user sign-in logs and flag accounts that have not logged in for 90 days. I will cross-reference this with HR databases to find terminated workers and convert their mailboxes to free shared mailboxes.
- **Result**: Proactively reclaiming unused E5 licenses, saving the organization thousands of dollars in annual software expenses.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **30 Days**: Listen, learn, map AD/Network, handle basic L1 tickets, shadow team.
> **60 Days**: Contribute independently, resolve L2 escalations, write runbooks, support migrations.
> **90 Days**: Optimize systems, deploy automation scripts, audit costs/licensing, lead training.
> **Key Action**: Ask for feedback at each 30-day mark to align expectations.

---

## Related Notes
- [[06-Career-Growth/12-Interview-Prep/Top-10-Behavioral-and-HR-Questions|Top 10 Behavioral and HR Questions]]
- [[05-Automation-and-Ticketing/11-ITSM-Ticketing/ITIL-v4-and-SLA|ITIL v4 and SLA]]
- [[00-MOC/Master-Index|Master Index]]

---

## Tags
#desktop-support #onboarding #onboarding-plan #career-development #job-ready #management #L1 #L2 #L3
