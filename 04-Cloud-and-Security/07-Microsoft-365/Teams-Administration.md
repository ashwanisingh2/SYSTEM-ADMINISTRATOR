---
tags: [desktop-support, m365, teams-administration, collaboration, L2]
aliases: [teams-guide, teams-troubleshooting, teams-policies]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #md-102
---

# Teams Administration

---

## Concept Overview
- **What it is**: Microsoft Teams is a centralized communication and collaboration platform that integrates chat, video meetings, file storage, and application integration. It is built on top of existing Microsoft 365 services (SharePoint Online, OneDrive for Business, and Exchange Online).
- **Why it matters for a support engineer**: Teams is the primary workplace communication tool. Support engineers resolve desktop client crashes, audio/video call failures, sharing restriction blocks, and guest access issues daily.
- **Where you encounter this in real job**: Clearing corrupted Teams cache directories, modifying meeting policies in the Teams Admin Center, assigning phone numbers to users, and adjusting external guest access permissions.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Clears client cache, troubleshoots local headset/camera hardware, adjusts Teams status, and configures basic client settings.
  - **L2**: Manages Teams meeting, messaging, and app permission policies in the Teams Admin Center (TAC); troubleshoots external access issues, and audits basic call logs.
  - **L3**: Configures Teams Phone System (PSTN), manages Direct Routing/Operator Connect, monitors Call Quality Dashboard (CQD) telemetry, and sets global integration parameters.

---

## Technical Deep Dive

### 1. Underlying Architecture of Microsoft Teams
Teams is not a standalone storage system. It coordinates data across other Microsoft 365 services:

```
                  +--------------------------------+
                  |        Microsoft Teams         |
                  +--------------------------------+
                   /              |               \
                  /               |                \
                 v                v                 v
        [SharePoint Online]  [OneDrive for Business]  [Exchange Online]
        - Channel Files      - 1:1 Chat Files         - Calendar Sync
        - Team Sites         - Personal Storage       - Chat History (Hidden)
```

- **1:1 & Group Chat Files**: Stored in the sender's personal **OneDrive for Business** in a folder named `Microsoft Teams Chat Files`.
- **Channel Files**: Stored in the team's associated **SharePoint Online** site collection under the document library matching the channel name.
- **Chat & Channel Message History**: Stored in hidden folders inside the user/group **Exchange Online** mailboxes for compliance and discovery auditing.

### 2. Teams Policies vs. Organization-Wide Settings
TAC controls what users can see and do using a system of policies:
- **Messaging Policies**: Control chat features (e.g., allow deleting messages, sending Giphy, or using read receipts).
- **Meeting Policies**: Control meeting behavior (e.g., allow screen sharing, automatically admit guests, or enable transcription).
- **App Permission Policies**: Define which apps (Microsoft, third-party, or custom) are allowed in the tenant.
- **External Access (Federation)**: Allows Teams users to search, chat, and call users in *other* organizations using their corporate domain.
- **Guest Access**: Allows adding external users (via their personal email) to join specific Teams channels, view files, and participate in meetings as members of the team.

### 3. Voice & PSTN Calling Models
To connect Teams to the public telephone network (PSTN), organizations choose from three models:
1. **Microsoft Calling Plans**: Microsoft acts as the telecom carrier, providing phone numbers and minutes directly in the cloud. Easy to configure, but expensive at scale.
2. **Operator Connect**: Reuses existing contracts with major telecom operators (e.g., AT&T, Verizon) managed directly within the TAC.
3. **Direct Routing**: Connects an on-premises or cloud-based Session Border Controller (SBC) to Teams, allowing integration with legacy PBX systems and local carrier circuits.

---

## Commands & Syntax

### PowerShell
Teams management uses the `MicrosoftTeams` module.
```powershell
# Install and connect to Microsoft Teams PowerShell module
Install-Module -Name MicrosoftTeams -Force
Connect-MicrosoftTeams -Credential (Get-Credential)

# Retrieve a list of all Teams meeting policies
Get-CsTeamsMeetingPolicy | Select-Object Identity, AllowScreenSharing, AllowPrivateMeetingScheduling

# Create a custom messaging policy that disables Giphy and message deletion
New-CsTeamsMessagingPolicy -Identity "StrictMessagingPolicy" -AllowGifs $false -AllowUserDeleteMessage $false

# Assign the custom messaging policy to a specific user
Grant-CsTeamsMessagingPolicy -Identity "StrictMessagingPolicy" -PolicyName "StrictMessagingPolicy"
```

### CMD / Run Box (Teams Cache Cleaning)
For the Classic Teams client and the New Teams client (which runs on the Webview2/AppX framework):
```cmd
:: For New Teams Client (Reset via PowerShell/Appx)
powershell -Command "Get-AppxPackage *MSTeams* -AllUsers | Reset-AppxPackage"

:: For Classic Teams Client (Manual file clean - CMD)
taskkill /IM teams.exe /F
rmdir /S /Q "%appdata%\Microsoft\Teams"
```

### GUI Path
- **Teams Administration**: Go to **admin.teams.microsoft.com** (Teams Admin Center).
- **Teams Logs Collection**: Press `Ctrl` + `Alt` + `Shift` + `1` on a Windows client. It saves diagnostic TXT files directly to the user's `Downloads` folder.

### Important Registry Paths
- Teams installation and autostart registration:
  ```
  HKCU\Software\Microsoft\Windows\CurrentVersion\Run
  (Entry: "com.squirrel.Teams.Teams")
  ```

---

## Real-World Scenarios

### Scenario 1: Teams Client Stalls on Boot or Shows "Loading..." Infinite Loop
**User Complaint:** A project manager reports: *"My Teams desktop app is showing a blank white screen with a spinning wheel saying 'Loading Microsoft Teams' and never opens. I can access Teams fine in my web browser."*
**Your First 3 Checks:**
1. Check if the user has an active internet connection and can resolve M365 domains.
2. Terminate all background Teams processes using Task Manager.
3. Check the Teams client cache directories for corrupted local configurations.
**Diagnosis Steps:**
1. Run `tasklist` in Command Prompt to find if `Teams.exe` is hung in the background. Kill it: `taskkill /f /im Teams.exe`.
2. Inspect the local application cache. With the New Teams client built on the modern AppX runtime:
   - Go to `C:\Users\<username>\AppData\Local\Packages\MSTeams_8wekyb3d8bbwe`.
   - Clear the local state by running:
     `Get-AppxPackage *MSTeams* -AllUsers | Reset-AppxPackage` in PowerShell.
3. If they are still using Classic Teams, delete all files within `%appdata%\Microsoft\Teams` but preserve the folder structure.
4. Restart the Teams desktop app. The client forces the user to re-authenticate and pulls down clean profile metadata.
**Root Cause:** Corrupted local authentication tokens or cached layout databases in the AppData directory.
**Fix:** Isolate processes, clear client application cache (or run Appx reset), and force re-authentication.
**Prevention:** Educate users on using Web Teams if they face temporary app crashes, and ensure WebView2 runtimes are updated via Windows Update.
**Ticket Close Note:** "Killed hung Teams process, cleared client local app cache directory using Reset-AppxPackage. Relaunched client and user authenticated successfully. Closed."

### Scenario 2: External Vendors Unable to Share Screens in Teams Meetings
**User Complaint:** An HR Director hosting an interview reports: *"External candidates joining our interview meetings are unable to share their screens. The share tray icon is greyed out for them, even though I am the organizer."*
**Your First 3 Checks:**
1. Verify the client candidate is joining as a "Guest" or federated "External User".
2. Check the global Meeting Policies in the Teams Admin Center.
3. Check the meeting options configured for that specific calendar invite.
**Diagnosis Steps:**
1. Log into the Teams Admin Center (`admin.teams.microsoft.com`).
2. Go to **Meetings** -> **Meeting policies**. Identify the policy applied to the HR Director (typically the Global Org-wide default).
3. Scroll down to the **Content sharing** section.
   - Look at the setting: **Screen sharing mode**.
   - Current value: **Entire screen** (but with **Allow an external participant to give or request control** turned `Off`).
   - Check the **Who can present** setting. It was set to **Only people in my organization**.
4. Because the default presenter role was locked to internal staff, any external candidate joining was downgraded to an "Attendee", which blocks screen sharing permissions.
**Root Cause:** The global Teams meeting policy restricted presenter permissions for external users joining organization-scheduled meetings.
**Fix:**
1. In TAC meeting policies, verify the HR default policy allows attendees to share content.
2. Instruct the HR Director to click **Meeting options** inside the Outlook calendar invite or during the meeting.
3. Change "Who can present?" to **Everyone** or "People I choose", and toggle "Allow screen sharing for attendees" to **Yes**.
**Prevention:** Train staff hosting external meetings to review "Meeting options" before sending invites out.
**Ticket Close Note:** "Modified HR team meeting options to allow external presenters. Verified candidate could share their screen. Closed."

---

## Critical Points

> [!danger] Never Do This
> - Never disable **External Access (Federation)** under the impression that it blocks all spam, without consulting business units first.
> - Disabling federation blocks your entire workforce from communicating with external clients, vendors, and partners who use Teams, forcing them to use shadow IT communication channels.

> [!warning] Common Trap
> - Assuming that deleting the Teams desktop shortcut uninstall the app.
> - Teams Classic installs a secondary background application called **Teams Machine-Wide Installer**. If you do not uninstall this companion installer, Teams will reinstall itself for every user that logs into that computer during the next boot cycle.

> [!tip] Senior Engineer Tip
> - When diagnosing client network and performance lag during voice calls, press `Ctrl` + `Alt` + `Shift` + `1` on the client PC. Open the downloaded `MSTeams Diagnostics Log [Date].txt` file and search for `CallQualityTelemetry`. It displays packet loss, round-trip time (RTT), and jitter values directly from the client endpoint.

> [!success] Verification Steps
> - Run: `Get-CsTeamsMeetingPolicy -Identity "Global"` to verify that meetings allow screen sharing and external attendee participation.
> - Verify that changes made in the Teams Admin Center replicate to users (changes can take up to 2-4 hours to propagate globally).

> [!question] Interview Alert
> - "Where are files sent in a Teams 1:1 chat stored versus files sent in a Teams Channel?"
> - Answer: "Files shared in 1:1 or group chats are stored in the sender's personal OneDrive for Business library inside a folder called 'Microsoft Teams Chat Files'. Files shared in a Channel are stored in the document library of the SharePoint site collection associated with that Team."

---

## Common Mistakes & Fixes

| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| Deleting the main Teams folder while application is running | Locked files prevent deletion | Close the Teams app in Task Manager before trying to wipe the AppData cache. |
| Restricting Guest Access globally to fix single channel security | Lack of understanding of Teams permissions | Use Private/Shared Channels instead of shutting down guest collaboration for the entire tenant. |
| Forgetting to configure dynamic ports for Teams traffic | Network firewalls block high ports | Ensure firewall permits UDP ports 3478 through 3811 for real-time audio and video traffic. |

---

## Lab Exercise

**Objective:** Access the Teams Admin Center (or simulation/PowerShell), inspect the global messaging policy, build a custom policy blocking Giphy, and clear a test PC's Teams cache.
**Time Required:** 30 minutes
**Environment Needed:** A computer with internet access and a Microsoft 365 Developer tenant.
**Pre-requisites:** Administrative credentials to the Microsoft Teams Admin Center.

**Steps:**
1. Open a browser and navigate to the **Teams Admin Center** (`admin.teams.microsoft.com`).
2. Go to **Messaging policies** -> Click **Add** to create a new custom policy.
3. Name it `No-Giphy-Staff`. Toggle the **Giphys in conversations** setting to **Off**. Click **Save**.
4. Open PowerShell as Administrator and connect to Teams:
   ```powershell
   Connect-MicrosoftTeams
   ```
5. Assign this policy to a test user:
   ```powershell
   Grant-CsTeamsMessagingPolicy -Identity "testuser@yourdomain.com" -PolicyName "No-Giphy-Staff"
   ```
6. On the test user's computer, simulate cache corruption clearing:
   - Terminate the Teams process.
   - Open Run (`Win + R`), type `%localappdata%\Packages\MSTeams_8wekyb3d8bbwe` and delete the local cache folders, or reset the app package:
   ```powershell
   Get-AppxPackage *MSTeams* -AllUsers | Reset-AppxPackage
   ```

**Success Criteria:** The policy is applied to the test user, and their local Teams client cache is cleared, forcing a fresh download of the profile.
**Common Failures:** Policy updates in the Teams Admin Center can take up to 4 hours to take effect on the client desktop.

---

## Interview Questions & Answers

### Basic (L1 Level)
**Q: Where does Microsoft Teams store files shared in a 1:1 chat?**
A: Files shared in 1:1 or group chats are stored in the sender's OneDrive for Business account in a folder named "Microsoft Teams Chat Files". Only the members of the chat are granted permission to access that file.

**Q: How do you clear the Teams cache on a Windows computer where Teams is failing to launch?**
A: I close Teams in Task Manager. If it's the modern Teams client, I run the PowerShell command `Get-AppxPackage *MSTeams* -AllUsers | Reset-AppxPackage` to reset it. For classic Teams, I go to Run, type `%appdata%\Microsoft\Teams`, delete the files inside, and relaunch Teams.

### Intermediate (L2 Level)
**Q: What is the difference between "External Access" and "Guest Access" in Teams?**
A: External Access (Federation) allows your users to search, chat, and call users in other organizations without leaving their own tenant. Guest Access allows you to add an external user (using their email) to your team as a member, giving them access to channels, shared files, and group conversations.

**Q: What ports and protocols must be open on a firewall for Microsoft Teams audio and video calls to work properly?**
A: Teams requires outbound HTTPS port 443 TCP for web traffic, and UDP ports 3478 through 3811 for real-time media (audio, video, screen sharing). Blocking these UDP ports forces Teams to fall back to TCP 443, resulting in poor call quality and lag.

### Advanced (L3/Senior Level)
**Q: A user reports poor call quality and frequent drops during Teams meetings in our remote office. How do you analyze this issue?**
A:
- **Situation**: User experiencing audio drops and lag during corporate calls.
- **Task**: Identify the network bottle-neck causing poor call quality.
- **Action**: I log into the Teams Admin Center, find the user's account, and check their call history. I select the problematic call to view the session telemetry. I check the **Call Quality Dashboard** (CQD) for packet loss (must be < 1%), latency (must be < 150ms), and jitter (must be < 30ms). If packet loss is high, I trace the client network connection, checking if they are using Wi-Fi instead of wired Ethernet, or if the traffic is being routed through an unnecessary VPN tunnel.
- **Result**: I discovered the office VPN was routing Teams traffic to the headquarters before going to Microsoft. Setting up VPN split-tunneling for Teams endpoints resolved the call drops.

### HR / Behavioral
**Q: Tell me about a time you had to support a critical business event where technology failure was not an option. How did you prepare?**
A: Our company held an online AGM over Teams with external board members. To prepare, I set up a dedicated testing meeting with the presenters 2 days prior to test their cameras, microphones, and screen sharing permissions. During the live event, I monitored the Teams Admin Center real-time call logs. When one presenter faced audio lag, I guided them to switch to a phone call dial-in backup option, which I had prepared and sent in advance, ensuring the meeting proceeded smoothly.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **What**: Microsoft's central hub for collaboration, integration, meetings, and voice calls.
> **Why**: Critical for maintaining company communications. Relies on SharePoint, OneDrive, and Exchange Online.
> **How**: Manage client software cache issues, set messaging/meeting policies in TAC, and verify firewall port configurations.
> **Command**: `Get-CsTeamsMeetingPolicy` / `Reset-AppxPackage` / `Ctrl+Alt+Shift+1` (diagnostics logs)
> **Interview Answer Starter**: "Microsoft Teams is a wrapper service that stores files in SharePoint and OneDrive. To troubleshoot calling quality, I check port configurations and CQD telemetry..."

**Key Numbers to Remember:**
- Audio/Video UDP Ports: 3478 - 3811
- Target packet loss for calls: < 1%
- Target round-trip latency: < 150 ms
- Target jitter: < 30 ms

**3 Things Interviewer Wants to Hear:**
- Files shared in chats go to OneDrive, files in channels go to SharePoint
- Clearing Teams app cache via Appx Package Reset or clearing AppData folders
- Checking Call Quality Dashboard (CQD) for network packet loss/jitter

---

## Related Notes
- [[04-Cloud-and-Security/07-Microsoft-365/OneDrive-Administration|OneDrive Administration]] — Where chat-based files are saved.
- [[04-Cloud-and-Security/07-Microsoft-365/SharePoint-Online|SharePoint Online]] — The storage foundation for teams channels.
- [[01-Foundations/02-Networking/TCP-IP-and-Ports|TCP/IP and Ports]] — Critical for understanding Teams voice media traffic routing.

---

## Tags
#desktop-support #m365 #teams-administration #collaboration #L2 #interview-topic #lab-complete #daily-use

