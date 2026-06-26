---
tags: [m365, sharepoint, onedrive, cloud, beginner, troubleshooting]
aliases: [SharePoint and OneDrive, M365-03, SPO, ODFB]
created: 2026-06-26
status: #complete
difficulty: #beginner
cert-relevant: #none
---

> [!NOTE|color-blue]
> ☁️ **MICROSOFT 365 / INTUNE**

`#complete` `#beginner` `#none`

# M365-03: SharePoint and OneDrive

> [!abstract] Overview
> Yahan ek paragraph mein likho — SharePoint aur OneDrive Microsoft 365 ke core file storage and collaboration tools hain. Ek L1 support engineer ko pata hona chahiye ki permissions kaise manage karni hain, sync issues kaise fix karne hain, aur OneDrive setup kaise karna hai. Dono tools enterprise environment mein daily use hote hain aur kaafi saari tickets generate karte hain.

---
## 🧠 Concept Overview

- **What it is** — SharePoint is a cloud-based document management and collaboration platform. OneDrive is a personal cloud storage service for enterprise users, technically built on top of SharePoint but isolated per user.
- **Why it matters** — *Corporate environment mein sara data aur document sharing inhi platforms par hota hai. Agar user ka OneDrive sync ruk jaye ya SharePoint access na mile, toh unka kaam completely ruk sakta hai. Data security ke perspective se bhi yeh platforms critical hain.*
- **Where you see this** — *Daily tickets aati hain "OneDrive not syncing", "File access denied in SharePoint", "How to restore deleted file", "Share link is not working", "Missing documents".*

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic task — *OneDrive setup karna, sync issues resolve karna, file permissions check karna, basic file recovery from recycle bin, mapping document libraries.* |
| **L2** | Configure, fix — *SharePoint site creation, external sharing settings modify karna, complex permission issues fix karna, site storage limits extend karna.* |
| **L3** | Architecture — *Enterprise-wide data retention policies banana, SharePoint architecture design karna, migration from on-prem file servers to SharePoint.* |

> [!tip] Seedha Simple Mein
> *SharePoint ek company ka common drive (jaise purana on-premise file server ya NAS) hai jahan sab milke ek hi jagah kaam karte hain. Wahin dusri taraf, OneDrive user ka apna personal cloud folder hai (jaise My Documents) jise woh multiple devices (laptop, mobile) se safely access kar sakta hai.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **SharePoint** is like a **Corporate Conference Room** and **OneDrive** is like your **Personal Desk Drawer** because...
>
> - **Conference Room (SharePoint)** mein sab log aate hain, files dekhte hain aur milkar whiteboard par kaam karte hain. Usme alag-alag departments ke alag rooms (Sites) hote hain. Agar project khatam ho jaye, toh sab room chhod dete hain, but room (Site) wahi rehta hai.
> - **Personal Desk Drawer (OneDrive)** mein aapki khud ki personal aur draft files hoti hain. Jab tak aap kisi aur ko apni drawer ki chaabi (share link) nahi dete, koi aur us file ko nahi dekh sakta. Jab aap company chhodte hain, toh aapki drawer clear kar di jati hai.

---
## 🔬 Technical Deep Dive

### 1. OneDrive for Business (ODFB)

> [!info] Key Concept
> OneDrive for Business provides each user with 1 TB (or more, up to 5TB or 25TB depending on licensing) of personal cloud storage. It syncs local files to the cloud using the OneDrive Sync Client.

*OneDrive ka sabse bada fayda yeh hai ki user ka local data (Desktop, Documents, Pictures folders) automatically cloud par backup ho jata hai. Is process ko Known Folder Move (KFM) bolte hain. Agar laptop chori ho jaye ya kharab ho jaye, toh naya laptop de kar bas OneDrive sign-in karna hota hai aur saara data wapas aa jata hai.*

> [!danger] Common Mistake
> *Kai baar users 300,000 se zyada files sync karne lagte hain, jiski wajah se OneDrive client crash ya hang ho jata hai. Sync limits ka hamesha dhyan rakhna chahiye. SharePoint aur OneDrive ko mila kar ek client 300,000 files se zyada sync nahi karna chahiye.*

### 2. SharePoint Online (SPO)

> [!info] Key Concept
> SharePoint Online uses Sites and Document Libraries to store data. It is tightly integrated with Microsoft Teams. Every Team in MS Teams has a SharePoint site operating on the backend to store its files.

*SharePoint mein permissions inheritance chalti hai. Hierarchy aisi hoti hai: Site -> Library -> Folder -> File. Agar user ko Site par access hai, toh file par bhi automatically access hoga, unless permission manually break ki gayi ho (jise 'breaking inheritance' kehte hain).*

> [!danger] Common Mistake
> *External sharing ko bina strict policy ke open chhod dena ek bada security risk hai. Admins ko hamesha 'Anyone with the link' (Anonymous links) ko disable karna chahiye ya unpar 30-day expiry policy lagani chahiye.*

### 3. Version History & Co-authoring

> [!info] Key Concept
> Both SharePoint and OneDrive support Version History. Every time a file is saved, a new version is created. Users can restore an older version at any time without admin help.

*Yeh ransomware attacks ya accidental overwrites se bachne ka sabse achha tarika hai. M365 by default pichle 500 versions tak store karke rakhta hai.*

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - M365 Business Basic or above license
> - Windows 10/11 or macOS with OneDrive client installed
> - Global Admin or SharePoint Admin rights (for backend configuration)

### Step 1: Resetting the OneDrive Sync Client

*Agar OneDrive bilkul sync nahi kar raha, processing changes par stuck hai, ya red X dikha raha hai, toh client ko reset karna sabse best troubleshooting trick hai. Isse local data ya cloud data delete nahi hota, bas sync engine ka internal database refresh hota hai.*

```powershell
# Open Run dialog (Win + R) or CMD/PowerShell and run this:
# Yeh command OneDrive ko reset karta hai
%localappdata%\Microsoft\OneDrive\onedrive.exe /reset
```

> [!success] Expected Output
> *OneDrive ka blue cloud icon system tray se gayab hoga aur 1-2 minute baad automatically wapas aayega. Agar wapas na aaye toh manually start karein:*
> ```powershell
> # Start OneDrive manually if it doesn't auto-start
> %localappdata%\Microsoft\OneDrive\onedrive.exe
> ```

### Step 2: Restoring Deleted Files from SharePoint

*Agar kisi user ne galti se SharePoint document library se file delete kar di hai, toh L1 engineer use easily recycle bin se restore kar sakta hai.*

1. Go to the specific SharePoint Site URL.
2. Click on the **Recycle bin** option on the left navigation pane.
3. Select the deleted file(s) or folder(s) using the checkbox.
4. Click on **Restore** at the top menu.

> [!tip] Pro Tip
> *First-stage recycle bin (accessible to users) holds files for 93 days. If a user deletes the file from there, it goes to the Second-stage recycle bin (accessible only to site admins) for the remainder of the 93-day period.*

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `onedrive.exe /reset` | *OneDrive sync engine ko puri tarah se reset karta hai bina data lose kiye.* | `%localappdata%\Microsoft\OneDrive\onedrive.exe /reset` |
| `Get-SPOSite` | *Tenant mein saari SharePoint sites ki details ki list dikhata hai.* | `Get-SPOSite -Limit All` |
| `Set-SPOSite` | *Kisi specific site ki properties change karne ke liye, jaise storage quota badhana.* | `Set-SPOSite -Identity https://contoso.sharepoint.com/sites/IT -StorageQuota 1024` |
| `Connect-SPOService` | *SharePoint Online management shell (PowerShell) se admin connect karne ke liye.* | `Connect-SPOService -Url https://contoso-admin.sharepoint.com` |
| `Get-SPOUser` | *Ek specific SharePoint site par kis user ke paas kya permissions hain, yeh check karna.* | `Get-SPOUser -Site https://contoso.sharepoint.com/sites/IT` |
| `Remove-SPOUser` | *Kisi user ko SharePoint site ke access list se hatana.* | `Remove-SPOUser -Site https://contoso.sharepoint.com/sites/IT -LoginName user@contoso.com` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| **OneDrive Sync is stuck or very slow** | *Bohot saari files ek saath sync ho rahi hain, ya path bohot lamba hai (255+ chars).* | *OneDrive client reset karein, lambe file/folder names ko rename karke chota karein.* |
| **User cannot access a SharePoint file** | *Permission explicitly revoke ho gayi hai, ya shared link expire ho gaya hai.* | *Site admin ya file owner se check karwayein, "Manage Access" panel mein jaakar direct access/permission grant karein.* |
| **"Name already exists" error in OneDrive** | *Cloud par aur local PC folder mein same naam ki file conflict kar rahi hai.* | *Local file ka naam thoda change karein aur phir se sync hone dein, phir baad mein merge karein.* |
| **External user unable to open shared link** | *Tenant level par ya site level par external sharing policy block ki hui hai.* | *SharePoint admin center se external sharing policies tenant aur site level par check aur modify karein.* |
| **Red 'X' mark on OneDrive files/folders** | *Sync error because file dusri application mein open hai ya naam mein invalid characters hain.* | *Check if the file is open (like Word/Excel). Rename file to remove characters like `* : < > ? / \ |`* |
| **"Storage full" message for OneDrive** | *User ka 1TB quota exhaust ho gaya hai, ya recycle bin mein huge files hain.* | *OneDrive ka Recycle bin empty karein. Agar real data hai toh M365 Admin portal se limit 5TB tak badha dein.* |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: OneDrive Sync Stuck on "Processing changes"

> [!example] Ticket
> "My OneDrive is stuck on 'Processing changes' for the last 3 days. I can't see my new files on my laptop and it's draining my battery."

**L1 Response:** *Pehle check karein ki kya user ne koi bohot badi file (jaise Outlook ka old .pst file ya ISO file) OneDrive mein daali hai. Check for invalid characters in file names. Try pausing and resuming the sync.*
**Escalation Trigger:** *Agar basic reset aur pause/resume ke baad bhi issue persist kare aur log files mein sync engine ke complex errors aayen.*
**L2 Resolution:** *Registry (`HKCU\Software\Microsoft\OneDrive`) mein jake OneDrive credentials clear karna, aur OneDrive client ko cleanly uninstall karke latest version install karna.*

### 🎫 Scenario 2: Accidentally Deleted a Critical SharePoint Folder

> [!example] Ticket
> "I accidentally deleted the entire 'Q3 Finance Reports' folder from the Accounting SharePoint site while trying to move it. Please help, my manager needs it now!"

**L1 Response:** *User ko calm karein aur turant us SharePoint site ke Recycle Bin mein jayein. Us folder ko dhoond kar 'Restore' par click karein. Data wapas original location par chala jayega.*
**Escalation Trigger:** *Agar user ne panic hoke Recycle Bin se bhi folder clear kar diya (purge kar diya) hai.*
**L2 Resolution:** *SharePoint Admin center mein jaakar ya site collection settings mein jaakar Second-stage recycle bin access karna aur wahan se data admin rights use karke restore karna.*

### 🎫 Scenario 3: User cannot share file with External Vendor

> [!example] Ticket
> "I am trying to share a critical contract document with a vendor outside the company, but the 'Anyone with the link' option is completely greyed out and unclickable."

**L1 Response:** *Check karein ki user kis site se share kar raha hai. Yeh confirm karein ki baki SharePoint sites par bhi same issue hai ya sirf us ek specific site par. Screenshot lijiye error ka.*
**Escalation Trigger:** *Hamesha L2 ya Security team ko escalate karein kyunki yahan external sharing policy bypass ya modification required ho sakti hai.*
**L2 Resolution:** *SharePoint admin center mein check karna ki tenant-level par sharing allowed hai ya nahi. Phir us specific site ki sharing settings (Active sites -> Sharing) update karna agar vendor communication authorised hai.*

### 🎫 Scenario 4: User left company, Manager wants OneDrive Data

> [!example] Ticket
> "John Doe left the company yesterday. I am his manager, I need access to his OneDrive to retrieve ongoing project files before his account gets deleted."

**L1 Response:** *Check karein account ka status (disabled hai ya deleted). Agar deleted nahi hai, toh M365 Admin Center mein user ke profile mein jaakar 'OneDrive' tab par click karein aur 'Create link to files' generate karein.*
**Escalation Trigger:** *Agar user account delete huye 30 din se zyada ho gaye hain.*
**L2 Resolution:** *Agar account hard delete ho gaya hai aur retention policy apply nahi thi, toh L2 check karega backup solution (jaise Veeam ya Datto) se recover karne ke options.*

---
## 🎤 Interview Questions

> [!question] Q1: What is the main difference between SharePoint Online and OneDrive for Business?
> **Answer:** SharePoint is designed for team collaboration and sharing documents across the entire organization (like a modern network drive). OneDrive for Business is personal cloud storage meant for an individual user's draft and personal work files. OneDrive is actually built on top of the SharePoint architecture.

==**Exam Tip:** *Interview mein hamesha "Conference Room vs Personal Drawer" wala analogy use karein, it shows clear and practical understanding.*==

> [!question] Q2: How long are deleted files kept in SharePoint Online and OneDrive?
> **Answer:** Files are kept for a total of 93 days in both platforms. They first go to the user/site recycle bin (first-stage). If deleted or purged from there, they move to the site collection recycle bin (second-stage) for the remaining days of that 93-day period.

> [!question] Q3: What should you do if a user complains their OneDrive isn't syncing files and shows a red cross icon?
> **Answer:** I would first check the OneDrive system tray icon to see what exactly the red cross indicates. Usually, it's a file path that is too long, invalid characters (`* < > ? / \ |`) in the file name, or the file is open in another app. If nothing is obvious, I would restart the app, and if that fails, perform a `onedrive.exe /reset` command.

> [!question] Q4: How do permissions and inheritance work in SharePoint Document Libraries?
> **Answer:** SharePoint uses inherited permissions from top to bottom. By default, permissions assigned at the Parent Site level flow down to the document libraries, then to folders, and finally to individual files. However, an admin can "break inheritance" at any folder or file level to assign unique, restricted access.

> [!question] Q5: Can you explain what "Version History" is and how it helps?
> **Answer:** Version History automatically saves different versions of a file every time it is modified and saved in SPO or OneDrive. It helps users easily revert back to a previous state if they made a mistake, overwrote data accidentally, or if the file got corrupted by ransomware, without needing IT admin intervention.

---
## 🔗 Related Notes

- [[M365-01 Office 365 Basics|M365 Basics]] — *Core M365 licensing and admin concepts*
- [[M365-02 Exchange Online|Exchange Online]] — *Email architecture and mailboxes*
- [[Windows-10-Troubleshooting|Windows Desktop Issues]] — *Related to local sync and network issues affecting OneDrive*
