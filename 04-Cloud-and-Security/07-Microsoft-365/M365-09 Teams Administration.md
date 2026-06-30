---
tags: [microsoft-365, teams, administration, intermediate]
aliases: [teams-admin, teams-management]
created: 2026-06-26
status: #complete
difficulty: #intermediate
cert-relevant: #none
---

> [!NOTE|color-blue]
> ☁️ **MICROSOFT 365 / INTUNE**

`#complete` `#intermediate` `#none`

# M365-09: Teams Administration

> [!abstract] Overview
> Teams Administration is the backbone of modern corporate communication. It involves managing user access, meeting policies, chat features, and voice routing within Microsoft Teams. As an L1/L2 support engineer, understanding Teams Admin Center (TAC) is critical because Teams is the most frequently used app in the enterprise. *Yeh note aapko sikhayega ki Teams Admin Center se calls, meetings aur chat settings ko kaise control aur troubleshoot kiya jaata hai.*

---
## 🧠 Concept Overview

- **What it is** — Teams Administration is a centralized portal (admin.teams.microsoft.com) to govern all aspects of Microsoft Teams, including messaging, meetings, voice, and device management.
- **Why it matters** — In a remote or hybrid work setup, any issue with Teams stops the entire business. Admins need to ensure secure and seamless communication. *Agar Teams down hai, toh poori company ka kaam ruk jaata hai.*
- **Where you see this** — Managing guest access, fixing call drops, creating meeting policies for executives, and setting up auto-attendants.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic task — Check user licenses, reset Teams cache, collect debug logs, check basic network connectivity. *User complaints sunna aur basic troubleshooting karna.* |
| **L2** | Configure meeting policies, troubleshoot call quality using Call Quality Dashboard (CQD), manage guest/external access. *Policies assign karna aur call drop ke deeper logs check karna.* |
| **L3** | Architecture, Direct Routing (SBC) configuration, enterprise voice setup, integrating third-party compliance apps. *Complex voice routing aur system design karna.* |

> [!tip] Seedha Simple Mein
> *Teams Admin Center ek universal remote ki tarah hai jisse aap control kar sakte ho ki kaun kisse chat kar sakta hai, kaun file share kar sakta hai aur meeting mein kaun present kar sakta hai.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Microsoft Teams Admin Center** is like the **Control Room of a Corporate Office Building** because...
>
> - Just like a control room decides who gets entry passes (Guest vs External access).
> - It determines which meeting rooms have projectors or whiteboards (Meeting Policies and features).
> - It monitors the quality of intercom lines across all floors (Call Quality Dashboard).
> - It decides if conversations can be recorded (Meeting Policies).

---
## 🔬 Technical Deep Dive

### 1. Teams Admin Center (TAC) Core Areas

> [!info] Key Concept
> TAC is divided into several workloads: Teams, Users, Meetings, Messaging, Voice, and Analytics & reports. It's a role-based access control (RBAC) environment.

- **Teams and Channels:** Manage lifecycle, naming policies, and channel creation.
- **Users:** Assign specific policies to specific users (e.g., stopping a user from deleting chat messages).
- **Meetings:** Meeting policies, Live events, and Webinars. You can enforce lobbies so unauthorized people don't jump into calls.
- **Messaging:** Chat settings, GIF restrictions, and read receipts. You can turn off memes for corporate seriousness.
- **Voice:** Phone system, call queues, auto attendants.
- **Teams Apps:** Controlling which third-party apps (like Trello, Zoom) can be integrated into Teams channels.

### 2. Guest Access vs External Access (Federation)

> [!danger] Common Mistake
> Confusing Guest Access with External Access is the #1 mistake new admins make! Exam mein bhi yahi question aata hai.

- **External Access:** Allows users in your organization to find, call, and chat with users in another external organization (e.g., `@contoso.com` chatting with `@fabrikam.com`). They don't join your teams. *Baahar ki company walo ke saath sirf chat aur call karna, without joining the environment.*
- **Guest Access:** Allows you to invite people from outside your organization to join a specific Team, access files, and participate in channel conversations. *Baahar ke logo ko apni company ke andar kisi specific project folder mein invite karna.*

### 3. Call Quality Dashboard (CQD) & Call Analytics

Call Analytics is a per-user troubleshooting tool to see why a specific call was bad (Jitter, Packet Loss, Latency).
CQD is a tenant-wide view to identify macro network issues (e.g., all users in the Mumbai office have bad calls). Teams requires UDP traffic for optimal voice/video performance; if firewalls force TCP fallback, performance degrades.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Global Admin or Teams Administrator role.
> - PowerShell module for Microsoft Teams installed.

### Step 1: Install and Connect to Teams PowerShell

To automate tasks, PowerShell is essential.

```powershell
# Yeh command Teams PowerShell module install karta hai
Install-Module -Name MicrosoftTeams -Force -AllowClobber

# Yeh command Teams backend se connect karta hai
Connect-MicrosoftTeams
```

> [!success] Expected Output
> ```text
> Account                Environment Tenant                               TenantId
> -------                ----------- ------                               --------
> admin@contoso.onmicrosoft.com AzureCloud  contoso.onmicrosoft.com       a1b2c3d4...
> ```

### Step 2: Create a Custom Meeting Policy

Suppose HR wants a policy where recordings automatically expire after 30 days.

```powershell
# Yeh command ek nayi meeting policy banata hai aur recording expiration set karta hai
New-CsTeamsMeetingPolicy -Identity "HR_Meeting_Policy" -AllowCloudRecording $true -NewMeetingRecordingExpirationDays 30
```

### Step 3: Assign the Policy to a User

```powershell
# Assign policy to a specific HR user
Grant-CsTeamsMeetingPolicy -Identity "hr_manager@contoso.com" -PolicyName "HR_Meeting_Policy"
```

### Step 4: Clear Teams Cache (Windows)

This is the most common L1 task when Teams freezes or sync issues occur.

```cmd
# Step 1: Quit Teams. Step 2: Delete cache files.
taskkill /f /im msteams.exe
rmdir /q /s "%appdata%\Microsoft\Teams"
```

> [!tip] Pro Tip
> *Naye Teams (Teams V2) ka cache clear karne ke liye `Windows Settings -> Apps -> Installed Apps -> Microsoft Teams -> Advanced Options -> Reset` use karna padta hai.*

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `Connect-MicrosoftTeams` | Connects to Teams admin center via PS | `Connect-MicrosoftTeams` |
| `Get-Team` | Lists all Teams in the tenant | `Get-Team` |
| `Add-TeamUser` | Adds a user to a specific Team | `Add-TeamUser -GroupId <ID> -User user@domain.com` |
| `Set-CsTeamsMeetingPolicy` | Modifies an existing meeting policy | `Set-CsTeamsMeetingPolicy -Identity Global -AllowCloudRecording $true` |
| `Get-CsOnlineUser` | Gets details of voice/teams routing for a user | `Get-CsOnlineUser -Identity user@domain.com` |
| `Grant-CsTeamsMessagingPolicy` | Assigns messaging policy to a user | `Grant-CsTeamsMessagingPolicy -Identity user1 -PolicyName NoDelete` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| **User can't chat with external vendor** | External access is disabled or domain is blocked. | Go to TAC -> Users -> External Access. Allow the specific domain. |
| **"You're missing out" error on login** | Corrupt Teams Cache or outdated client. | Clear Teams Cache (AppData), update client, or use Web Teams to verify. |
| **Call drops exactly at 15 mins** | SIP Session Timer issues or Firewall blocking UDP ports. | Check Firewall rules for UDP 3478-3481. *Firewall UDP traffic drop kar raha hoga.* |
| **User cannot record meetings** | Meeting policy or lack of OneDrive/SharePoint storage. | Check TAC Meeting Policies (`AllowCloudRecording`). Ensure user has OneDrive license. |
| **Poor Call Quality (Robotic Voice)** | High packet loss or latency on user's network. | Open TAC -> Users -> Search User -> Meetings & Calls. Check Call Analytics for Jitter > 30ms or Packet Loss > 1%. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: External Access / Federation Issue

> [!example] Ticket
> "Hi IT, I am trying to message John from ABC Corp (john@abccorp.com) but it says 'We can't set up the conversation'."

**L1 Response:** Verify if the user is typing the correct email address. Ask them to test on Teams Web app.
**Escalation Trigger:** If the issue persists on Web app and only for this specific external domain.
**L2 Resolution:** Open Teams Admin Center > Users > External Access. Ensure "abccorp.com" is not in the blocked domains list, or add it to the allowed domains list if the organization uses a strict allow-list policy. *Dono side ke admins ko external access enable karna padta hai.*

### 🎫 Scenario 2: Call Quality Complaints

> [!example] Ticket
> "My calls are dropping frequently and participants complain my voice sounds like a robot."

**L1 Response:** Ask the user to switch from Wi-Fi to a wired LAN connection and re-test. Clear Teams cache.
**Escalation Trigger:** If LAN connection doesn't fix the issue and multiple users in the same office face it.
**L2 Resolution:** Use Call Quality Dashboard (CQD) and Call Analytics in TAC. Search for the user and review the latest call. Look for Red markers indicating High Network Jitter or High Packet Loss. If it's a site-wide issue, escalate to the Network team.

### 🎫 Scenario 3: Missing Meeting Recording

> [!example] Ticket
> "I recorded yesterday's all-hands meeting but I cannot find the recording link anywhere."

**L1 Response:** Ask the user to check their OneDrive "Recordings" folder or the specific Channel's "Files > Recordings" folder if it was a channel meeting.
**Escalation Trigger:** If the recording is missing from OneDrive/SharePoint and the chat window says "Recording failed".
**L2 Resolution:** Check Teams Meeting Policy applied to the user. Ensure `AllowCloudRecording` is enabled. Also, check M365 Admin Center to ensure the user's OneDrive has sufficient quota space. *Agar OneDrive full hoga, toh recording save nahi hogi.*

### 🎫 Scenario 4: User Cannot Create New Teams

> [!example] Ticket
> "I need to create a new Team for Project Alpha, but the 'Create Team' button is missing from my Teams app."

**L1 Response:** Verify that the user is not just looking in the wrong place. Give them web app instructions.
**Escalation Trigger:** If it's globally missing for the user.
**L2 Resolution:** The organization might have restricted M365 Group creation to a specific security group via Azure AD PowerShell. Check if the user is a member of the allowed security group. *Companies aksar restrict kar deti hain taaki Teams spam na ho jaaye.* Add the user to the "M365 Group Creators" security group.

---
## 🎤 Interview Questions

> [!question] Q1: What is the difference between External Access and Guest Access in Teams?
> **Answer:** External Access (Federation) allows chat and calls with users from other organizations without them switching tenants. Guest Access allows you to invite an external user into your Team/Channel, giving them access to files, chats, and resources.

> [!question] Q2: Where are Teams meeting recordings stored?
> **Answer:** Since 2021, Teams meeting recordings are stored in OneDrive for Business (for ad-hoc or private meetings) and in SharePoint Online (for Channel meetings). They are no longer stored in Microsoft Stream classic.

> [!question] Q3: A user complains about poor audio quality. How do you troubleshoot this as an admin?
> **Answer:** I would go to the Teams Admin Center, navigate to the user's profile, and click on "Meetings & Calls" to access Call Analytics. I would look at the specific bad call to check metrics like Packet Loss, Jitter, and Round Trip Time (Latency) to determine if it's a local network, ISP, or device issue.

> [!question] Q4: How do you prevent users from deleting sent chat messages?
> **Answer:** I would create a custom Messaging Policy in the Teams Admin Center, set "Delete sent messages" to OFF, and assign this policy to the specific users or groups.

> [!question] Q5: What network ports and protocols are most important for Teams Media Quality?
> **Answer:** Microsoft Teams heavily relies on UDP for media (Voice and Video). The key ports are UDP 3478 through 3481. If UDP is blocked, it falls back to TCP 443, which causes poor performance and delays.

==**Exam Tip:** Always remember the default ports for Teams media traffic are UDP 3478 through 3481. Exam scenarios often test your knowledge on firewall port openings for voice traffic!==

---
## 🔗 Related Notes

- [[M365-01 Introduction to M365|M365 Overview]] — Understanding M365 licensing
- [[M365-05 SharePoint Admin|SharePoint Administration]] — Where Teams files are stored
- [[Net-03 TCP and UDP|TCP vs UDP Protocols]] — Important for voice/video troubleshooting
