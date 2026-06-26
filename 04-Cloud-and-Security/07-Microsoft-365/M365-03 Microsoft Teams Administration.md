---
tags: [desktop-support, m365, collaboration, L1]
aliases: [m365-03-microsoft-teams-administration, m365-03]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #none
---

> [!NOTE|color-blue]
> ☁️ **CLOUD & MICROSOFT 365**

`#complete` `#intermediate` `#none`

# M365-03: Microsoft Teams Administration

> [!abstract] Overview
> Yeh note Microsoft Teams architecture, administrative controls, aur connectivity models cover karta hai. Ek support engineer ke roop mein aapko external/guest access govern karna, policies manage karna aur call quality issues troubleshoot karna aana chahiye.

---
## 🧠 Concept Overview

- **What it is** — Teams sirf chat app nahi hai, ye unified interface hai jo M365 groups, SharePoint aur Exchange ko joddta hai.
- **Why it matters** — Galat access settings ki wajah se external logon ko company data dikh sakta hai, aur sprawl se data management mushkil ho jati hai.
- **Where you see this** — Jab kisi user ko kisi external client ko chat par invite karna ho, ya low call quality troubleshoot karni ho.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Users ko add/remove karna teams me, basic login issues fix karna. |
| **L2** | Teams aur Channel creation policies manage karna, Guest access configure karna, aur Call Quality Dashboard check karna. |
| **L3** | Direct Routing aur Voice configurations karna, QoS network policies define karna. |

> [!tip] Seedha Simple Mein
> *Microsoft Teams aapke organizational communication ka hub hai, jahan teams, channels aur custom messaging policies ke zariye admin access control aur integration settings set ki jaati hain.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Microsoft Teams** is like **a modern office building** because...
>
> - **Team**: Dedicate department floor.
> - **Channels**: Specialized conference rooms on that floor.
> - **Tabs**: Whiteboards mounted in those rooms.
> - **Apps**: Tools and calculators brought into the rooms.

---
## 🔬 Technical Deep Dive

### 1. Teams Architecture & Storage

> [!info] Key Concept
> Teams apna data alag se store nahi karta, wo SharePoint, OneDrive aur Exchange par dependent hai.

- **Files in Standard/Private Channels**: SharePoint document library mein store hoti hain.
- **Files in 1:1/Group Chat**: Sender ke OneDrive mein store hoti hain.
- **Chat Messages**: Exchange mailboxes (hidden folder) mein store hote hain compliance ke liye.

### 2. External Access vs. Guest Access

- **External Access (Federation)**: Doosre tenants (jaise vendors) se chat aur call kar sakte hain. Unhe teams/files ka access nahi milta.
- **Guest Access**: Outside users ko Teams mein add karna jisse unhe files aur channel chat ka access milta hai (Entra ID mein guest ban jata hai).

> [!danger] Common Mistake
> Allowing uncontrolled Team creation by all users. Isse "Teams Sprawl" hota hai aur bohot saare khali aur duplicate SharePoint sites ban jate hain. Hamesha Group creation ko ek admin group tak limit karein.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Teams Administrator permissions
> - Microsoft Teams PowerShell Module

### Step 1: Create a Custom Meeting Policy

```powershell
# Authenticate
Install-Module -Name MicrosoftTeams -Force
Connect-MicrosoftTeams

# Create Policy disabling cloud recording
New-CsTeamsMeetingPolicy -Identity "RestrictedMeetingPolicy" -AllowCloudRecording $false -AllowTranscription $false

# Assign to a user
Grant-CsTeamsMeetingPolicy -Identity "user@yourtenant.com" -PolicyName "RestrictedMeetingPolicy"
```

> [!success] Expected Output
> ```
> Custom meeting policy created and successfully assigned to the user.
> ```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `Get-Team` | Tenant me saare Teams list karta hai | `Get-Team` |
| `Add-TeamUser` | Kisi user ko Team mein add karta hai | `Add-TeamUser -GroupId "id" -User "upn" -Role Member` |
| `Set-CsTeamsClientConfiguration` | Global client settings (like Guest access) configure karta hai | `Set-CsTeamsClientConfiguration -AllowGuestUser $true` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| External guests cannot access files in Teams | SharePoint external sharing restrict kar raha hai | SharePoint admin center mein sharing ko "New and existing guests" set karein. |
| Call dropping, robotic voice, lag | Network latency, jitter, packet loss | Call Quality Dashboard check karein, aur ensure karein ki VPN me split-tunneling on ho Teams traffic ke liye. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Audio Issues on Calls

> [!example] Ticket
> "I was on a call with a client and my voice kept cutting out. It made me look very unprofessional."

**L1 Response:** User ke network aur headset settings check karna.
**Escalation Trigger:** Agar user home internet pe nahi hai balki office VPN pe hai.
**L2 Resolution:** Teams Admin Center -> Users -> Meetings & Calls history check karna. Jitter (>30ms) aur packet loss (>1%) dekh kar VPN split-tunneling network team se theek karwana.

---
## 🎤 Interview Questions

> [!question] Q1: Where does Teams store its files, chats, and meeting recordings?
> **Answer:** Channel files SharePoint pe, 1:1 chat files OneDrive pe, chat messages hidden Exchange folder me, aur meeting recordings SharePoint (channels) ya OneDrive (private meetings) mein store hoti hain.

==**Exam Tip:** External access federation enable karta hai sirf chat/call ke liye. Guest access se full team collaboration allow hota hai.==

---
## 🔗 Related Notes

- [[04-Cloud-and-Security/07-Microsoft-365/M365-01 Microsoft 365 Administration|M365-01 Microsoft 365 Administration]] — Manages tenant-wide licensing.
- [[04-Cloud-and-Security/07-Microsoft-365/M365-04 SharePoint Online Administration|M365-04 SharePoint Online Administration]] — Governs the storage layer underpinning Teams channels.
- [[04-Cloud-and-Security/07-Microsoft-365/M365-05 Security and Compliance|M365-05 Security and Compliance]] — Controls retention, DLP, and eDiscovery for Teams data.
