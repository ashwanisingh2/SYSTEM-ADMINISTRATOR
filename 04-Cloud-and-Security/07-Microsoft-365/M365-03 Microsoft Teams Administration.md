---
tags: [desktop-support, m365, collaboration, L1]
aliases: [m365-03-microsoft-teams-administration, m365-03]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #none
---

# M365-03: Microsoft Teams Administration

> [!abstract] Overview
> This note covers the architecture, administrative controls, and connectivity models of Microsoft Teams. It details how to govern collaboration, manage external and guest communication, and troubleshoot performance/logistical issues.

---

---
## Concept Overview
Think of Microsoft Teams as a modern office building. Each "Team" is a dedicated department floor. "Channels" are specialized conference rooms or bulletin boards on that floor (General, Projects, Budget). "Tabs" are the whiteboards and displays mounted in those rooms, while "Apps" are the tools and calculators brought into the rooms to help work get done.

---

---
## Technical Deep Dive
### 1. Teams Architecture & Storage
Teams is not a standalone storage engine; it is a unified interface built on top of multiple M365 services:
- **Team Creation**: Behind every Team is a hidden Microsoft 365 Group, which provisions a SharePoint Online Team Site, an Exchange Online group mailbox, and a OneNote notebook.
- **Files Storage**:
  - Files shared in a **Standard/Private Channel** are stored in a dedicated SharePoint document library.
  - Files shared in a **1:1 or Group Chat** are uploaded to the sender's personal OneDrive for Business (`/personal/username_domain_com/Documents/Microsoft Teams Chat Files`).
- **Chats & Messages**: Stored in a hidden folder in the Exchange mailbox of each user (for chats) or the group mailbox (for channel conversations) for eDiscovery compliance purposes.

### 2. Access Control: External Access vs. Guest Access
- **External Access (Federation)**: Allows Teams users in your organization to search, chat, call, and set up meetings with users in *other* M365 domains or Skype users. These external users cannot access your Teams channels, files, or resources.
- **Guest Access**: Allows users outside your organization to be added to a Team. Guests get access to Teams channels, shared files, chat histories, and can collaborate inside your tenant. Guest identities are managed as guest users in Microsoft Entra ID.

### 3. Policy Management (Messaging, Meeting, Calling)
Policies define what features are available to users. They can be applied globally (tenant-wide) or assigned to specific users/groups:
- **Messaging Policies**: Control features like chat, editing/deleting sent messages, read receipts, Giphy/meme permissions, and URL previews.
- **Meeting Policies**: Manage features available during Teams meetings, including screen-sharing permissions, bypass lobby settings, cloud recording options, and transcription enablement.
- **Calling Policies**: Define whether users can make private calls, use call forwarding, set up simultaneous ring, or delegate calls.

### 4. Teams Voice & Calling Architectures
- **Calling Plans**: Microsoft acts as the PSTN carrier. You purchase phone numbers and calling minutes directly from Microsoft.
- **Direct Routing**: Connects your on-premises or hosted Session Border Controller (SBC) to Microsoft Phone System. Allows using your local telecom carrier for dial tone.
- **Operator Connect**: A hybrid model where you select a participating third-party operator from the Teams Admin Center to manage PSTN calling over managed network pipelines.

---

## Common Mistakes
> [!warning] Avoid These
> Confusing External Access (Federation) with Guest Access. Disabling external access will prevent communication with foreign organizations, while guest access control is what governs outside vendors getting inside your teams.
> Allowing uncontrolled Team creation by all users. This leads to "Teams Sprawl," producing dozens of empty teams, duplicate SharePoint sites, and fragmented document storage.
> Deploying Teams without setting up Microsoft 365 Group naming policies, leading to confusing or duplicate names (e.g., "HR" vs "Human Resources Department").

---

## Pro Tips
> [!tip] Field Experience
> Restrict Microsoft 365 Group creation to a designated security group (e.g., "Group Creators"). Require users to submit a ticket/request form for new Teams. This keeps your SharePoint environment clean and organized.
> Use the **Call Quality Dashboard (CQD)** to monitor tenant-wide network health trends. It identifies whether call quality problems are localized to specific offices, Wi-Fi networks, or ISP routes.

---

---
## Step-by-Step Lab
> [!info] Lab Setup Needed
> Microsoft 365 Tenant with Teams Administrator permissions and Microsoft Teams PowerShell Module installed.

### Step 1: Install and Connect to Microsoft Teams PowerShell
Open PowerShell as an Administrator and authenticate:
```powershell
Install-Module -Name MicrosoftTeams -Force
Connect-MicrosoftTeams
```

### Step 2: Create a New Team and Configure Channel Structure
Create a Team for the Finance department and add channels:
```powershell
# Create the Team (this creates the underlying M365 Group)
$team = New-Team -DisplayName "Finance Department" -Visibility Private -Description "Collaboration space for Finance"

# Create a Private Channel (requires individual user membership)
New-TeamChannel -GroupId $team.GroupId -DisplayName "Tax Auditing" -MembershipType Private

# Create a Standard Channel
New-TeamChannel -GroupId $team.GroupId -DisplayName "General Budget" -MembershipType Standard
```

### Step 3: Configure Guest Access Settings via PowerShell
Ensure Guest Access is enabled globally in the tenant and block file sharing via guest accounts:
```powershell
# Enable Guest Access in the Tenant
Set-CsTeamsClientConfiguration -AllowGuestUser $true

# Restrict guest sharing properties
Set-CsTeamsMeetingConfiguration -AllowExternalUsersToPassLobby $false
```

### Step 4: Create and Assign a Custom Meeting Policy
Create a custom meeting policy that disables recording and forces transcription off for junior employees:
```powershell
# Create Policy
New-CsTeamsMeetingPolicy -Identity "RestrictedMeetingPolicy" -AllowCloudRecording $false -AllowTranscription $false

# Assign to a user
Grant-CsTeamsMeetingPolicy -Identity "user@yourtenant.onmicrosoft.com" -PolicyName "RestrictedMeetingPolicy"
```

---

---
## Cheat Sheet / Quick Reference
```powershell
Get-Team                                               # Lists all Teams in the tenant
Get-TeamUser -GroupId "your-group-id"                 # Lists members and owners of a Team
Add-TeamUser -GroupId "id" -User "upn" -Role Member    # Adds user to a Team
Remove-TeamUser -GroupId "id" -User "upn"              # Removes user from a Team
Set-Team -GroupId "id" -AllowCreateUpdateChannels $false # Disables channel creation for members
```

---
| # | Concept | One Line Summary |
|---|---------|-----------------|
| 1 | Standard Channel | Collaboration space visible and accessible to all members of the Team. |
| 2 | Private Channel | Restricted space inside a Team accessible only to a selected subset of members. |
| 3 | Guest Access | Enables external users to participate in chats, meetings, and co-author files. |
| 4 | External Access | Federation enabling text/call communication with external domains without tenant entry. |
| 5 | Call Quality Dashboard | Analytics portal aggregating call and network performance data across the organization. |

---

---
## Troubleshooting
**Scenario 1:**
- Problem: Users report that external guests are unable to collaborate on files inside a Teams channel, despite guest access being enabled. They see a "You don't have access to this file" error.
- Root Cause: SharePoint external sharing settings are set more restrictively than Teams guest access settings. SharePoint must allow sharing with external users for Teams file-sharing to work.
- Fix:
  1. Open the SharePoint Admin Center.
  2. Navigate to `Policies` -> `Sharing`.
  3. Ensure that the SharePoint and OneDrive external sharing settings are set to "New and existing guests" or "Anyone" (matching or exceeding the Teams guest configuration).
  4. Ensure that the site collection backing the specific Team does not have sharing disabled at the site level.

**Scenario 2:**
- Problem: A user complains that during calls their audio drops frequently, and they experience robotic voice effects and sluggish screen sharing.
- Root Cause: High network latency, jitter, or packet loss. Often caused by lack of Quality of Service (QoS) configurations or network traffic being routed through a VPN or security proxy instead of splitting off directly to Microsoft.
- Fix:
  1. Open the **Teams Admin Center** -> `Users` -> Manage Users -> Select the user -> `Meetings & Calls`.
  2. Scroll down to the call history, click on the specific bad call, and check the network metrics (Round trip time should be < 150ms, Jitter < 30ms, Packet loss < 1%).
  3. Implement **split-tunneling** in the corporate VPN to bypass routing Teams traffic (`*.lync.com`, `*.teams.microsoft.com`, and IP blocks defined in Office 365 URLs).
  4. Configure DSCP markings (EF for voice, AF41 for video) on switches and firewalls.

---

---
## Interview Questions
**Q1: Where does Teams store its files, chats, and meeting recordings? Explain the architectural components.**
A: Teams uses existing M365 infrastructure for storage. Files shared in channels are stored in the Team's SharePoint site document library. Files shared in 1:1 or group chats are stored in the sender's OneDrive for Business. Chat messages are written to hidden folders inside Exchange Online mailboxes for compliance indexing. Modern meeting recordings are stored directly in the SharePoint site folder (for channels) or the OneDrive "Recordings" folder of the user who initiated the recording.

**Q2: A client is deploying Teams Voice. How would you choose between Microsoft Calling Plans and Direct Routing?**
A:
- **Situation**: The client needed a PSTN connection strategy for 500 users, split between a US head office and a manufacturing plant in India.
- **Task**: I had to evaluate which model was cost-effective and legally compliant, given different local telecom laws.
- **Action**: I compared Microsoft Calling Plans against Direct Routing. For the US office, I recommended Microsoft Calling Plans for simplicity and low maintenance. For the India plant, due to strict local regulatory compliance (TRAI) and existing contracts with local telecom carriers, I configured Direct Routing by installing a certified Ribbon SBC connected to their local SIP trunks.
- **Result**: The hybrid deployment saved 35% in monthly calling costs compared to pure Calling Plans, and successfully bypassed legal compliance hurdles in international offices.

**Q3: How do you configure Quality of Service (QoS) for Microsoft Teams, and what ports must be prioritised on the firewall?**
A: QoS is configured by enabling DSCP markings in the Teams Admin Center under Meetings -> Meeting Settings -> Network. You must mark Audio traffic as DSCP EF (Expedited Forwarding), Video as AF41, and Application Sharing as AF31. On the local network firewalls and switches, you must prioritize UDP ports 3478 through 3481, ensuring they are not routed through deep packet inspection (DPI) proxies which introduce latency.

---

---
## Seedha Simple Mein
*Seedha simple mein: Microsoft Teams aapke organizational communication ka hub hai, jahan teams, channels aur custom messaging policies ke zariye admin access control aur integration settings set ki jaati hain.*

---
## Related Notes
- [[04-Cloud-and-Security/07-Microsoft-365/M365-01 Microsoft 365 Administration|M365-01 Microsoft 365 Administration]] — Manages tenant-wide licensing and basic user roles.
- [[04-Cloud-and-Security/07-Microsoft-365/M365-04 SharePoint Online Administration|M365-04 SharePoint Online Administration]] — Governs the storage layer underpinning Teams channels.
- [[04-Cloud-and-Security/07-Microsoft-365/M365-05 Security and Compliance|M365-05 Security and Compliance]] — Controls retention, DLP, and eDiscovery for Teams data.
