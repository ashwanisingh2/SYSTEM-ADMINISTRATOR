---
tags: [desktop-support, lab, active-directory, azure, L3]
aliases: [hybrid-identity-lab, lab-06]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #az-104
---

# LAB-06: Hybrid Identity Lab

> [!note] Overview
> This lab guide covers setting up a Hybrid Identity lab. It walks through preparing local Active Directory (AD DS) objects using the IdFix tool, configuring custom User Principal Name (UPN) suffixes, installing Azure AD Connect (Entra Connect), configuring Password Hash Synchronization (PHS), and verifying directory sync in Microsoft Entra.

---
## Concept Overview
- **What it is** — A Hybrid Identity Lab is an environment that demonstrates how to synchronize local, on-premises Active Directory users, groups, and contacts to Microsoft Entra ID (formerly Azure Active Directory), creating a single identity footprint for cloud applications.
- **Why it matters** — Enterprise networks rarely exist solely in the cloud or on-prem. A hybrid identity model allows users to log into their local domain computers and access M365 email/SaaS tools using the same set of credentials.
- **Real job encounter** — Troubleshooting synchronization errors between on-prem AD and Entra ID (e.g. duplicate UPNs or mismatched attributes), running the IdFix utility, and forcing manual delta synchronization cycles.
- **L1 vs L2 vs L3 responsibility split:**
  - **L1 Resolution**: Verify sync status in Microsoft Entra Admin Center, locate user accounts in Active Directory, trigger manual syncs via PowerShell, and check sync reports.
  - **Escalation Trigger**: Escalate when synchronization fails globally, user password resets do not sync to the cloud, or when duplicate object errors prevent synchronization.
  - **L2 Resolution**: Resolve synchronization conflicts using IdFix, add custom UPN suffixes, edit attributes like `mail` or `proxyAddresses`, and run diagnostic troubleshooting commands.
  - **L3 Resolution**: Install and configure Azure AD Connect/Entra Connect sync engines, configure Pass-through Authentication (PTA) or ADFS federation, deploy staging mode backup servers, and configure Password Writeback.

*Seedha simple mein: Hybrid Identity Lab hume local Active Directory (On-Premises) aur Microsoft Entra ID (Cloud) ko aapas me sync karna seekhati hai. Azure AD Connect tool ki help se hum local users ko automatically Microsoft 365 portal me clone kar dete hain taaki user ek hi password se local PC aur online email dono access kar sake.*

---
## Technical Deep Dive

### 1. Synchronization Models
Choosing the right synchronization mechanism is critical:
- **Password Hash Synchronization (PHS)**: The simplest and most popular model. Replicas of the user's password hashes are securely synced from on-premises AD to Entra ID. Cloud authentication happens entirely within Entra ID.
- **Pass-through Authentication (PTA)**: Authentication requests are validated directly against on-premises Active Directory Domain Controllers via local agents, ensuring passwords never exist in the cloud.
- **Active Directory Federation Services (ADFS)**: Authentication is delegated to local servers. Users are redirected to an on-prem login page.

### 2. Matching Parameters and ImmutableID
- **User Principal Name (UPN)**: Must match the user's verified custom cloud domain (e.g. `ashwani@mydomain.com` instead of local `ashwani@corp.local`).
- **sourceAnchor (ImmutableID)**: The unique attribute (typically `objectGUID` converted to Base64) that links the on-premises AD object with its synced counterpart in Microsoft Entra ID.

---
## Step-by-Step Lab

> [!warning] Pre-requisites
> - A Windows Server running as an Active Directory Domain Controller (e.g., Domain `corp.local`).
> - A Microsoft Entra / M365 Tenant (Developer tenant is sufficient).
> - A verified custom domain added to the tenant (e.g., `mybrand.com`) to resolve matching UPN requirements.
> - An account in Entra ID with **Hybrid Identity Administrator** permissions.

### Step 1: Add Custom UPN Suffix in Active Directory
By default, AD accounts end in `.local`. Entra ID cannot route login traffic for unverified domains. We must add the custom verified domain.

1. On the Domain Controller, open **Active Directory Domains and Trusts**.
2. Right-click the root node **Active Directory Domains and Trusts** and select **Properties**.
3. In the **Alternative UPN Suffixes** text box, type your verified domain name (e.g., `mybrand.com`).
4. Click **Add**, then click **Apply** and **OK**.
5. Open **Active Directory Users and Computers (ADUC)**.
6. Create an OU named `SyncOU`, and create a user `Ashwani Singh` inside it.
7. Open the user properties, navigate to the **Account** tab, select the UPN dropdown, change it from `@corp.local` to `@mybrand.com`, and click **OK**.

### Step 2: Clean Directory Using IdFix Tool
Before running the synchronization engine, we must verify that our local directory is free from attribute errors.

1. Download the **IdFix Identity Synchronization Error Tool** from Microsoft.
2. Run `IdFix.exe` on your Domain Controller.
3. Click **Query**.
**Expected Output:** Displays a list of users with invalid formatting, duplicate attributes, or invalid characters.
4. Correct any identified errors (e.g. remove trailing spaces or invalid characters from emails) and click **Apply**.

### Step 3: Install and Configure Azure AD Connect
1. Download **Microsoft Entra Connect** installer on your synchronization server.
2. Double-click the installer, accept license agreements, and choose **Customize** (or Express Settings for standard setups).
3. Connect your directory databases:
   - Enter your **Microsoft Entra Global Administrator** credentials.
   - Enter your **On-Premises AD enterprise administrator** credentials.
4. On the **Azure AD Sign-In Configuration** screen:
   - Verify that your custom UPN suffix (`mybrand.com`) shows as **Verified**.
   - Check the box: "Continue without matching all UPN suffixes to verified domains" (if testing).
5. On the **Filtering** screen, select **Synchronize selected OUs** and choose only **SyncOU**.
6. On the **Optional Features** screen, select **Password Hash Synchronization** and **Password Writeback** (if premium licenses are available).
7. Select **Start the synchronization process when configuration completes** and click **Install**.

### Step 4: Verify Synchronization
1. Open PowerShell on the sync server and run the synchronization status command:
```powershell
Get-ADSyncScheduler
```
**Expected Output:** Shows `SyncCycleEnabled : True` and `NextSyncCycleStartTimeInUTC`.

2. Log into the **Microsoft Entra Admin Center** (`entra.microsoft.com`).
3. Navigate to **Identity** -> **Users** -> **All Users**.
4. Search for `Ashwani Singh`.
**Expected Output:** User appears in the user list, and the **On-premises sync enabled** attribute shows as **Yes**.

---
## Cheat Sheet / Quick Reference

| PowerShell Command (ADSync Module) | Purpose | Example |
|---|---|---|
| `Start-ADSyncSyncCycle -PolicyType Delta` | Trigger a delta sync (changes only) | `Start-ADSyncSyncCycle -PolicyType Delta` |
| `Start-ADSyncSyncCycle -PolicyType Initial`| Trigger a full directory sync cycle | `Start-ADSyncSyncCycle -PolicyType Initial` |
| `Get-ADSyncScheduler` | Check sync intervals and schedule states | `Get-ADSyncScheduler` |
| `Set-ADSyncScheduler -SyncCycleEnabled $true`| Enable the background sync scheduler | `Set-ADSyncScheduler -SyncCycleEnabled $true` |
| `Get-ADSyncConnectorRunStatus` | View running status of connectors | `Get-ADSyncConnectorRunStatus` |

---
## Troubleshooting Table

| Problem | Cause | Fix | Command |
|---|---|---|---|
| Users sync but show `@tenant.onmicrosoft.com` instead of custom UPN domain. | The custom domain UPN suffix was not assigned to the user object in on-prem AD, or is unverified in Entra. | Assign the alternative UPN suffix to the user account in ADUC; verify custom domain status in Entra. | N/A |
| Sync status shows error: `AttributeValueMustBeUnique`. | Two on-premises objects have the same UPN, Email, or proxyAddresses attribute. | Open IdFix, identify the duplicate attributes, and assign unique values to the objects. | N/A |
| Changes in AD do not reflect in Entra ID immediately. | The sync cycle has not run (default schedule is every 30 minutes). | Manually force a delta sync cycle using PowerShell on the sync server. | `Start-ADSyncSyncCycle -PolicyType Delta` |
| Sync engine reports: `Stopped-Server-Down`. | The Domain Controller is offline, or the ADSync service on the sync host is stopped. | Verify DC network connectivity; restart the Microsoft Azure AD Sync service. | `Get-Service ADSync` / `Start-Service ADSync` |

---
## Interview Questions

> [!question] L1 Question
> **Q:** What is the default synchronization frequency for Azure AD Connect, and how do you force a synchronization cycle manually?
> **A:** The default synchronization interval is **30 minutes**. To force a synchronization cycle immediately for new changes, you open PowerShell on the sync server and run:
> `Start-ADSyncSyncCycle -PolicyType Delta`

> [!question] L2 Question
> **Q:** What is the purpose of the Microsoft IdFix tool?
> **A:** The **IdFix** tool is used to perform a search and identify potential compliance issues in an on-premises Active Directory environment before starting directory synchronization to Microsoft Entra. It flags formatting errors, duplicate attributes, invalid characters, and empty UPNs that would cause synchronization failures in the cloud.

> [!question] L3/Scenario Question
> **Q:** A synced hybrid user changes their name and email in on-premises AD, but the changes fail to update in Entra ID, returning a "SymmetricKeyError." How do you troubleshoot and fix this synchronization failure?
> **A:** 
> - **Situation:** Synced object modification fails with directory-level security or attribute conflicts.
> - **Task:** Re-align matching keys and fix the synchronization block.
> - **Action:** 
>   1. **Locate Error Logs**: Open the **Synchronization Service Manager** (`miisclient.exe`) on the AD Connect server, check the "Operations" log, and select the failed sync run to isolate the conflicting attribute.
>   2. **Verify ImmutableID**: Extract the user's local ObjectGUID, convert it to Base64 (ImmutableID), and compare it with the ImmutableID of the object in Entra ID using `Get-MgUser` (Microsoft Graph PowerShell).
>   3. **Fix conflicts**: If there's a mismatch, disconnect the object by setting UPN to a cloud-only status, clear the Entra ImmutableID, and then force a synchronization cycle to re-link using the correct key:
>      ```powershell
>      Set-MgUser -UserId <user-id> -OnPremisesImmutableId ""
>      ```
>   4. **Sync Run**: Run delta sync to establish soft-matching or hard-matching.
> - **Result:** The user object updates successfully, and synchronization integrity is restored.

---
## Seedha Simple Mein
*Seedha simple mein: Hybrid Identity Lab local AD users ko cloud (Microsoft Entra ID) me copy/sync karne ke process ko dikhati hai. Isme Azure AD Connect software download karke sync configuration run ki jati hai. Agar kisi user ka email structure match nahi karta, toh IdFix tool use error clean karne ke liye suggest karta hai.*

---
## Related Notes
- [[04-Cloud-and-Security/07-Microsoft-365/M365-07 Azure-AD-Connect|M365-07 Azure-AD-Connect]] — Sync infrastructure deep dive.
- [[03-Identity-and-Core-Services/06-Active-Directory/WS-02 AD DS Installation|WS-02 AD DS Installation]] — Local directory foundations.
- [[04-Cloud-and-Security/08-Azure/AZ104-15 Azure-AD-Connect-and-Hybrid|AZ104-15 Azure-AD-Connect-and-Hybrid]] — Entra Hybrid identity models.

---
*Tags: #desktop-support #lab #active-directory #azure #L3*
