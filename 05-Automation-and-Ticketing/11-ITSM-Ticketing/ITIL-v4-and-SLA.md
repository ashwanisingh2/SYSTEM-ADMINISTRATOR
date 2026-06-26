---
tags: [itsm, itil, ticketing, sla, ola, priority-matrix, service-management]
aliases: [itil-guide, sla-guide, priority-matrix, itil-v4]
created: 2026-06-25
status: #complete
difficulty: #beginner
cert-relevant: #none
---

> [!NOTE|color-blue]
> 📋 **ITSM / ITIL**

`#complete` `#beginner` `#none`

# ITSM-01: ITIL v4 and Service Level Agreements (SLA)

> [!abstract] Overview
> Yeh note cover karta hai **ITIL v4 framework** — jo IT service management ka gold standard hai — aur **SLAs (Service Level Agreements)** jo define karte hain ki IT team kitni fast response aur resolve karegi issues ko. Ek L1 support engineer ke liye yeh samajhna zaroori hai kyunki har ticket jo tum khologe usmein SLA clock tick kar raha hoga, aur agar tum priority galat set karo ya SLA breach ho jaaye toh company ko penalties bhi lag sakti hain. Yeh note tumhe sikhayega ki tickets ko kaise classify karo, priority kaise set karo (Impact × Urgency), aur SLA/OLA/UC mein kya fark hai.

---

## 🧠 Concept Overview

- **What it is** — **ITIL v4 (Information Technology Infrastructure Library)** is the industry-standard framework for managing IT services. A **Service Level Agreement (SLA)** is a committed agreement between an IT service provider and a customer that defines service standards — jaise system uptime, response times, aur resolution times.
- **Why it matters** — Ticketing systems (ServiceNow, Jira Service Desk) enforce strict SLA timers. Support engineers ko ITIL workflows samajhne padenge taaki tickets correctly classify ho, issues ko Impact aur Urgency ke basis pe prioritize kar sako, aur SLA breaches prevent ho.
- **Where you see this** — Incoming tickets ki triage, active incidents pe SLA clocks manage karna, internal team agreements (OLAs) use karke tasks hand off karna, aur weekly performance metrics report karna.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|-----------|-----------------|
| **L1** | Initial triage karta hai, ticket Urgency/Impact assign karta hai, passwords reset karta hai, aur standard incidents SLA response window ke andar resolve karta hai |
| **L2** | Escalated tickets ko SLA resolution windows ke andar resolve karta hai, cross-team hand-offs coordinate karta hai, aur workarounds document karta hai |
| **L3** | Business stakeholders ke saath global SLAs negotiate karta hai, Operational Level Agreements (OLAs) define karta hai, aur ticket trends review karke service availability improve karta hai |

> [!tip] Seedha Simple Mein
> *ITIL ek rulebook hai jo batata hai ki IT team ko kaise kaam karna chahiye — organized, measured, aur customer-focused. SLA ek promise hai — "Agar tera system down hoga toh hum itne time mein fix karenge." Agar yeh promise toota, company ki reputation aur paise dono jaate hain.*

---

## 💡 Real-World Analogy

> [!info] Think of it like this...
> **ITIL** is like a **restaurant's standard operating procedure (SOP)** aur **SLA** is like the **delivery time guarantee** printed on the menu.
>
> - Jaise Domino's ka promise hai "30 minutes ya free" — woh unka SLA hai customer ke saath
> - Kitchen staff ke beech ka rule "pizza 10 min mein oven se bahar aana chahiye" — woh unka OLA (internal agreement) hai
> - Flour supplier ke saath contract "aata 24 ghante mein deliver hona chahiye" — woh unka UC (Underpinning Contract) hai
> - Agar pizza late aaya toh customer ko free milta hai (SLA breach = penalty)
> - ITIL framework woh poori recipe book hai jo batati hai ki order lene se delivery tak sab kaise smooth chalega

---

## 🔬 Technical Deep Dive

### 1. ITIL v4 Key Frameworks

ITIL v4 focuses on **co-creating value**. *Matlab IT team sirf tickets close nahi karti, woh business ke saath milke value banati hai.* The two core frameworks are:

**The Four Dimensions of Service Management:**

| 🏗️ Dimension | 📝 Kya Hai | 💼 Example |
|-------------|-----------|-----------|
| **Organizations & People** | Roles, responsibilities, culture, communication | *Kaun L1 hai, kaun L2 hai, escalation chain kya hai* |
| **Information & Technology** | Software (ticketing system), databases, infrastructure | *ServiceNow, Jira, CMDB* |
| **Partners & Suppliers** | Cloud hosting, ISPs, third-party vendors | *AWS, Comcast, Microsoft* |
| **Value Streams & Processes** | Workflows to deliver value | *Incident → Triage → Resolve → Close* |

> [!info] Key Concept
> **The Service Value Chain** — Yeh 6 activities hain jo demand ko value mein convert karti hain:
> **Plan → Improve → Engage → Design → Obtain/Build → Deliver/Support**
>
> *Socho ek pipeline jaise — ek end se customer ki demand aati hai, doosre end se resolved service nikalti hai.*

### 2. SLAs, OLAs, and Underpinning Contracts (UC)

ITIL defines **three types of agreements** to deliver service:

| 📄 Agreement | 🤝 Between | 📝 Example |
|-------------|-----------|-----------|
| **SLA** (Service Level Agreement) | IT ↔ Business/Customer | *"P1 incidents resolved in 2 hours"* |
| **OLA** (Operational Level Agreement) | Internal IT Team ↔ Another IT Team | *"Switch port requests configured within 24 hours"* |
| **UC** (Underpinning Contract) | IT ↔ External Vendor/Supplier | *"WAN link downtime restored within 4 hours by Comcast"* |

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

> [!danger] Common Mistake
> **SLA aur OLA confuse mat karo!**
> - ==**SLA** customer ke saath contract hai — breach hone pe **financial penalty** lagti hai==
> - **OLA** internal teams ke beech agreement hai — breach hone pe internal workflow slow hota hai, lekin direct customer penalty nahi lagti
> - **UC** external vendor ke saath legal contract hai — iske breach pe vendor ko penalty milti hai

### 3. The Priority Matrix (Impact vs. Urgency)

Ticketing systems **priority calculate karte hain** by combining two variables:

- **Impact** — *Kitne users ya systems affected hain?* (Business pe kitna asar padega)
- **Urgency** — *Business kitni der tolerate kar sakta hai?* (Kitni jaldi fix hona zaroori hai)
- ==**Priority = Impact × Urgency**==

| 🔴 Priority | ⚡ Impact + Urgency | 💼 Example |
|------------|-------------------|-----------|
| **P1 (Critical)** | High Impact + High Urgency | *Main email server down — poori company blocked* |
| **P2 (High)** | Medium Impact + High Urgency | *VIP user login nahi kar pa raha, ya payroll ke time branch printer down* |
| **P3 (Medium)** | Medium Impact + Medium Urgency | *Standard user print nahi kar pa raha* |
| **P4 (Low)** | Low Impact + Low Urgency | *User ne software shortcut request kiya* |

> [!danger] Common Mistake
> **Kabhi bhi har ticket ko P1 mat mark karo sirf isliye ki user gussa hai!** P1 overuse se "alert fatigue" hoti hai — senior engineers real critical outages (jaise ransomware ya core database failure) ko ignore karne lagte hain. ==P1 sirf system-wide outages ke liye reserve rakho.==

> [!tip] Pro Tip
> **"Pending Customer" state use karo!** Agar tum user ke reply ka wait kar rahe ho (email ya laptop laane ke liye), toh ticket state ko **Awaiting User Info** ya **Pending Customer** change karo. Isse SLA resolution clock **pause** ho jaata hai — tumhari team ki metrics pe penalty nahi lagti user ki delay ki wajah se.

---

## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - PowerShell installed (Windows pe already hota hai)
> - Basic scripting knowledge
> - Koi ticketing system access zaruri nahi — hum **mock data** use karenge

**Objective:** Audit ticket priorities using PowerShell — check karo ki tickets correctly prioritized hain ya nahi, based on simulated user counts aur VIP status.

**⏱️ Time Required:** 30 minutes

### Step 1: Open PowerShell and Create Mock Ticket Data

```powershell
# Step 1: Mock ticket data banao — jaise ServiceNow se export ki ho
$Tickets = @(
    [PSCustomObject]@{ID="INC-1"; UsersAffected=150; VIP=$false; Priority="Low"}
    [PSCustomObject]@{ID="INC-2"; UsersAffected=1;   VIP=$true;  Priority="Low"}
    [PSCustomObject]@{ID="INC-3"; UsersAffected=1;   VIP=$false; Priority="Low"}
)
```

### Step 2: Run Priority Audit Logic

```powershell
# Step 2: Business rules ke basis pe correct priority calculate karo
# Rule: UsersAffected > 100 -> Priority = "Critical" (P1)
# Rule: VIP = True -> Priority = "High" (P2)
# Rule: UsersAffected = 1 and VIP = False -> Priority = "Medium" (P3)

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

### Step 3: Print the Audit Report

```powershell
# Step 3: Audit report print karo
$AuditedTickets | Format-Table -AutoSize
```

> [!success] Expected Output
> ```
> ID    UsersAffected   VIP LoggedPriority AuditedPriority Misclassified
> --    -------------   --- -------------- --------------- -------------
> INC-1           150 False Low            Critical                 True
> INC-2             1  True Low            High                     True
> INC-3             1 False Low            Medium                   True
> ```
> INC-1 → Critical, INC-2 → High, INC-3 → Medium hona chahiye. Teeno misclassified the!

### Step 4: (Bonus) ServiceNow API Query — P1 SLA Breach Monitor

```powershell
# Step 4: ServiceNow REST API se P1 tickets query karo jo SLA breach ke kareeb hain
$User = "api_user"
$Pass = "SecurePassword123!"
$Base64Auth = [Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes("$($User):$($Pass)"))
$Headers = @{ Authorization = "Basic $Base64Auth" }

# Active P1 tickets jinka Response SLA 15 min se kam bacha hai
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

> [!warning] Caution
> Yeh Step 4 sirf tab kaam karega jab tumhare paas actual ServiceNow instance ka API access ho. Lab ke liye Steps 1-3 kaafi hain.

---

## ⌨️ Command Cheat Sheet

### ITIL Terminology Quick Reference

| 📋 Term | 🛠️ Kya Hai | 📝 Example |
|--------|-----------|-----------|
| `SLA` | IT ↔ Customer agreement with service targets | *"P1 resolve in 2 hours"* |
| `OLA` | Internal IT team ↔ IT team agreement | *"Network team acknowledge in 15 min"* |
| `UC` | IT ↔ External vendor legal contract | *"ISP restore WAN in 4 hours"* |
| `Impact` | Kitne users/systems affected hain | *High = 100+ users, Low = 1 user* |
| `Urgency` | Business kitni der tolerate kar sakta hai | *High = VIP/meeting in 10 min* |
| `Priority` | Impact × Urgency | *P1, P2, P3, P4* |
| `CMDB` | Configuration Management Database | *Sab IT assets ka record* |
| `Pending Customer` | Ticket state — user ka reply wait ho raha | *SLA clock PAUSE hota hai* |
| `Pending Vendor` | Ticket state — vendor ka reply wait ho raha | *SLA clock PAUSE hota hai* |
| `SVC` | Service Value Chain | *Plan → Improve → Engage → Design → Obtain/Build → Deliver/Support* |

### Priority Matrix Quick Reference

| ⚡ | 🟢 Low Urgency | 🟡 Medium Urgency | 🔴 High Urgency |
|----|---------------|-------------------|-----------------|
| **🟢 Low Impact** | P4 (Low) | P3 (Medium) | P3 (Medium) |
| **🟡 Medium Impact** | P3 (Medium) | P3 (Medium) | P2 (High) |
| **🔴 High Impact** | P2 (High) | P2 (High) | P1 (Critical) |

### Key Numbers to Remember

| 🔢 Number | 📋 Kya Hai |
|----------|-----------|
| `4` | ITIL Dimensions of Service Management |
| `6` | Service Value Chain Activities |
| `Impact × Urgency` | Priority calculation formula |
| `2-4 hours` | Typical P1 SLA resolution target |
| `15 minutes` | Typical P1 acknowledgment OLA |

---

## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| SLA breach ho gayi jabki team kaam kar rahi thi | Ticket update nahi kiya — SLA clock chalta raha | Ticket state ko **"In Progress"** ya **"Work in Progress"** set karo taaki SLA clock update ho |
| SLA clock user ke wait mein bhi chalti rahi | **"Pending Customer"** state set nahi kiya | Jab bhi user se reply wait ho, state **"Awaiting User Info"** pe change karo — clock pause hogi |
| Har ticket P1 mark ho raha hai | Users ya agents priority inflate kar rahe hain | Strictly **Impact × Urgency matrix** follow karo — P1 sirf system-wide outage ke liye |
| OLA breach ho gayi lekin kisi ko pata nahi chala | OLA monitoring/alerts configured nahi the | Ticketing system mein **automated OLA breach alerts** configure karo (SMS/PagerDuty) |
| Vendor response late aa raha, SLA breach hone wali | **"Pending Vendor"** state use nahi kiya | UC timeline check karo, ticket state **"Pending Vendor"** set karo, vendor ko escalate karo |
| Ticket galat team ko assign ho gaya, time waste hua | Wrong assignment group select kiya | Ticket categories aur assignment groups ki **mapping document** banao aur team ko train karo |
| Customer ne 7 din reply nahi diya, ticket stuck hai | Auto-close policy nahi hai | **Auto-follow-up emails** configure karo (Day 3, Day 5) aur **Day 7 pe auto-close** with note |

---

## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: VIP User Account Lockout (Triage & Priority Classification)

> [!example] Ticket
> "I am trying to log in to our client presentation dashboard, but it says my account is locked out. The meeting starts in 10 minutes. I need this reset immediately." — *VP of Marketing*

**L1 Response:**
1. Check VIP status of the user → **Urgency identify karo**
2. Determine number of users affected → **Impact identify karo**
3. **Impact Assessment:** Only 1 user locked out → Impact = **Low**
4. **Urgency Assessment:** VIP user, client meeting in 10 min → Urgency = **High**
5. **Matrix Calculation:** Low Impact + High Urgency = ==**P2 (High Priority)**==
6. *Why not P1?* P1 requires enterprise-wide service loss. P2 still flags for immediate action without triggering corporate paging alerts.
7. Unlock account in Active Directory, reset password, verify login

**Escalation Trigger:** Agar AD mein unlock karne ke baad bhi login fail ho raha — toh L2 ko escalate karo (possible Azure AD sync issue)

**L2 Resolution:** Azure AD Connect sync check, conditional access policies review

**Root Cause:** Bad password attempts on a secondary device locked the VIP account.

**Prevention:** Enroll VIP users in **Self-Service Password Reset (SSPR)** to allow self-recovery.

**Ticket Close Note:** *"Unlocked VP account via AD. Classified as P2 due to VIP urgency. Verified login access. Recommended SSPR enrollment. Closed."*

---

### 🎫 Scenario 2: Active SLA Breach Warning on Network Outage

> [!example] Ticket
> "SLA Breach Warning: Incident INC-009845 (Core Switch Down in Warehouse) has 10 minutes remaining on the resolution SLA. Current state: Assigned to Network Group, unacknowledged." — *Monitoring System Alert*

**L1 Response:**
1. Open incident INC-009845 — check status aur assignment log
2. Contact on-duty Network Engineer immediately via phone
3. Trigger OLA escalation procedures

**Diagnosis:**
- Ticket `Network-Ops` group ko 1.5 hours pehle assign hua tha — but **no update, no acknowledgment**
- **SLA Details:** Resolution SLA = 2 hours. Elapsed = 1 hour 50 min. ==Sirf 10 min baaki!==
- **OLA Check:** Helpdesk ↔ Network-Ops OLA states "P1 tickets acknowledged within 15 min" — **OLA already breached**

**Escalation Trigger:** OLA breach + SLA about to breach → immediately phone call to Network Manager

**L2 Resolution:**
1. Call Network Operations team — confirm they are on-site replacing the switch
2. Update ticket: *"Confirmed Network team on-site replacing hardware. Work in progress."*
3. Toggle ticket state to **"In Progress"** (this updates SLA status indicators)
4. Resolve ticket once switch is replaced

**Root Cause:** Communication breakdown — Network team was working on physical hardware but forgot to update the ticket, leaving the SLA clock running.

**Prevention:** Configure **automated SMS/PagerDuty alerts** to page the Network Manager if a P1 ticket is unacknowledged for 15 minutes.

**Ticket Close Note:** *"Escalated to Network team via phone. Core switch replaced. Warehouse connectivity restored. OLA acknowledgment breach documented. Closed."*

---

### 🎫 Scenario 3: 20% SLA Breaches Due to Customer Inactivity

> [!example] Ticket
> "Monthly SLA report shows 20% of incidents breached resolution SLA. Analysis reveals majority were stuck in 'Wait for Customer' state without SLA pause configured." — *Service Delivery Manager*

**L1 Response:** Yeh L1 ke scope se bahar hai — immediately L3/Service Delivery Manager ko escalate karo with data

**Escalation Trigger:** Systemic SLA failure pattern — requires process change, not individual fix

**L2 Resolution:** L2 helps identify which ticket categories are most affected, pulls report data

**L3 Resolution:**
1. Configure ticketing system to **automatically pause SLA clock** when ticket is set to "Awaiting User Info"
2. Set up **automated email follow-ups**:
   - Day 3: First reminder email to user
   - Day 5: Second reminder with warning
   - Day 7: **Auto-close ticket** with note — *"Closed due to inactivity"*
3. **Result:** SLA breach rates dropped by 18%, technicians spend less time chasing unresponsive users

**Ticket Close Note:** *"Implemented auto-pause SLA on 'Awaiting User' state. Configured auto-reminder at Day 3, 5. Auto-close at Day 7. SLA breach rate improved by 18%. Closed."*

---

## 🎤 Interview Questions

> [!question] Q1: What is an SLA (Service Level Agreement) in a helpdesk environment?
> **Answer:** An SLA is a committed target agreement between IT and the customer that defines service delivery standards — jaise ek ticket ko 1 hour mein respond karna aur 4 hours mein resolve karna. It ensures consistent support delivery aur dono parties ko expectations clear hoti hain.

> [!question] Q2: How do you determine the priority of a newly submitted ticket?
> **Answer:** Priority determine karne ke liye main do factors combine karta hoon: **Impact** (kitne users ya systems affected hain) aur **Urgency** (business ko kitni jaldi fix chahiye). Corporate **Priority Matrix (Impact × Urgency)** use karke P1, P2, P3, ya P4 assign karta hoon.

> [!question] Q3: Explain the difference between SLA, OLA, and Underpinning Contract (UC).
> **Answer:** ==**SLA** IT aur customer ke beech ka agreement hai jo service targets define karta hai. **OLA** internal IT departments ke beech ka agreement hai (e.g., helpdesk aur network team) jo define karta hai ki woh ek doosre ko kaise support karenge. **UC** ek external vendor ke saath legal contract hai (e.g., Microsoft ya Comcast) jo humari internal SLAs support karta hai.==

> [!question] Q4: What do you do if a ticket is about to breach its SLA, but you are waiting for a vendor to respond?
> **Answer:** Main ticket status ko **"Pending Vendor"** ya **"Awaiting Vendor"** change karta hoon — isse most ticketing systems mein SLA timer pause hota hai. Vendor contact reference ticket logs mein document karta hoon aur Underpinning Contract ki timeline ke according follow up karta hoon.

> [!question] Q5: A VIP Director demands you classify their personal laptop issue as P1, bypassing 20 other tickets. How do you handle this?
> **Answer:** Main politely validate karta hoon unki urgency. Phir explain karta hoon: *"Main isko P1 classify nahi kar sakta kyunki office network chal raha hai, lekin main ise P2 High Priority VIP ticket ke taur pe log kar raha hoon. Main ise directly next available tech ko assign kar raha hoon aur woh aapko 15 minutes mein contact karenge."* Isse priority database accurate rehta hai aur VIP customer service bhi excellent rehti hai.

> [!question] Q6: You are auditing ticket queues and notice 20% SLA breaches due to 'Wait for Customer' delays. How do you fix this?
> **Answer:** (**STAR format mein:**)
> - **Situation:** Helpdesk metrics SLA targets miss kar rahi hain customer responsiveness ki wajah se
> - **Task:** Follow-ups automate karna aur SLA penalties rokna
> - **Action:** Ticketing system configure karta hoon taaki "Awaiting User Info" pe SLA clock auto-pause ho. Automated email system set up karta hoon — Day 3 aur Day 5 pe reminder, Day 7 pe auto-close with note
> - **Result:** Team ki SLA breach rates 18% drop hui, technicians ka time save hua

> [!question] Q7: What are the Four Dimensions of ITIL v4 Service Management?
> **Answer:** The four dimensions are: **(1) Organizations & People** — roles, culture, communication; **(2) Information & Technology** — tools, databases, infrastructure; **(3) Partners & Suppliers** — external vendors; **(4) Value Streams & Processes** — workflows to deliver value. Chaaron dimensions balanced hone chahiye effective service management ke liye.

> [!question] Q8: What happens if an SLA is breached? What about an OLA breach?
> **Answer:** ==Agar **SLA breach** hoti hai toh company ko **financial penalties** milti hain aur customer trust damage hota hai — kyunki yeh customer ke saath direct contract hai. Agar **OLA breach** hoti hai toh internal workflows slow hote hain — yeh directly customer contract violate nahi karta, but future mein SLA breaches ka risk badh jaata hai.==

==**Exam Tip:** Interviewer hamesha SLA vs OLA ka fark poochta hai. Yaad rakho — SLA = external (customer ke saath), OLA = internal (teams ke beech), UC = vendor ke saath. Teeno ka direction yaad karo!==

---

## 🔗 Related Notes

- [[05-Automation-and-Ticketing/11-ITSM-Ticketing/Incident-and-Problem-Management|Incident and Problem Management]] — Focuses on ticket resolution workflows
- [[05-Automation-and-Ticketing/11-ITSM-Ticketing/Change-and-Asset-Management|Change and Asset Management]] — Details change control processes
- [[06-Career-Growth/12-Interview-Prep/Top-10-Behavioral-Questions|Top 10 Behavioral Questions]] — Covers customer handling behavioral skills

---

#desktop-support #itsm #ticketing #itil #L1 #interview-topic #lab-complete #daily-use
