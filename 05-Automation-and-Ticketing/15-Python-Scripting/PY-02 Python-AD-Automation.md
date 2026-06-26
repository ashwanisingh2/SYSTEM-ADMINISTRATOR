---
tags: [desktop-support, automation, active-directory, python, L3]
aliases: [python-ad-automation, py-02]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #none
---

# PY-02: Python AD Automation

> [!note] Overview
> This note covers automating Active Directory (AD) administration using Python. It focuses on the `ldap3` library for directory authentication, searching directory objects, modifying user attributes, and executing bulk actions (creating, disabling, and resetting passwords for multiple users).

---
## Concept Overview
- **What it is** — Active Directory automation in Python involves communicating with AD Domain Controllers over LDAP/LDAPS (Lightweight Directory Access Protocol) to query, create, update, or delete directory services objects.
- **Why it matters** — Standard tools like ADUC (Active Directory Users and Computers) are manual, and PowerShell scripts can become hard to maintain in heterogeneous cross-platform environments. Python allows cross-platform, automated provisioning systems to connect directly to AD.
- **Real job encounter** — Integrating AD with an HR onboarding system to automatically provision new accounts, disabling inactive users in bulk, and performing scheduled directory audits.
- **L1 vs L2 vs L3 responsibility split:**
  - **L1 Resolution**: Execute pre-configured bulk upload scripts with a CSV file, check execution logs for errors, and verify user creation in ADUC.
  - **Escalation Trigger**: Escalate if connection errors occur (e.g. "invalid credentials" or "server not available"), or if the LDAP schema prevents new entries.
  - **L2 Resolution**: Write scripts to search AD for specific user attributes (e.g., expiry date), reset passwords programmatically, and parse raw CSV data to feed into provisioning routines.
  - **L3 Resolution**: Author full-cycle lifecycle management systems using secure LDAPS (port 636), handle schema extensions, manage Kerberos authentication, and configure security controls for service account permissions.

*Seedha simple mein: Active Directory ko Python ke sath manage karne ke liye hum LDAP protocol aur `ldap3` library ka use karte hain. Iska fayda ye hai ki hum single script se hazaron users ko bulk me create, delete, ya modify kar sakte hain bina Windows GUI open kiye.*

---
## Technical Deep Dive

### 1. Active Directory and LDAP
Active Directory uses LDAP as its core query protocol.
- **LDAP Port**: Default unencrypted traffic runs on port `389`.
- **LDAPS Port**: Secured SSL/TLS traffic runs on port `636`. *This is mandatory for modifying user passwords.*
- **Distinguished Name (DN)**: The unique path identifying an object (e.g., `CN=User1,OU=Employees,DC=corp,DC=local`).

### 2. The ldap3 Library
A pure Python LDAP client library that works on Windows, Linux, and macOS.
- **Server Object**: Configures connection parameters, target domain controller, port, and security requirements.
- **Connection Object**: Binds (authenticates) using service account credentials. Supports Simple, NTLM, and GSSAPI (Kerberos) authentication.
- **Search Filters**: standard LDAP search filters (e.g., `(&(objectClass=user)(sAMAccountName=ashwani))`).

---
## Step-by-Step Lab

> [!warning] Pre-requisites
> - An active Active Directory Domain Controller reachable on the network.
> - An administrative domain account (or service account with delegation permissions to manage users).
> - Python 3 installed.

### Step 1: Install ldap3 and Initialize Connection
1. Activate your virtual environment and install `ldap3`:
```bash
pip install ldap3
```
**Expected Output:** Successful installation of the library.

### Step 2: Write a Directory Query Script
We will write a script to search for a user by their username (sAMAccountName) and print their details.

1. Create `ad_search.py`:
```python
from ldap3 import Server, Connection, ALL

# Configure Active Directory DC details
LDAP_SERVER = "192.168.1.10" # Replace with Domain Controller IP or FQDN
LDAP_USER = "CORP\\Administrator" # Domain\Admin account
LDAP_PASSWORD = "SecretPassword123" # Admin password
SEARCH_BASE = "DC=corp,DC=local" # Root search directory

# Define connection
server = Server(LDAP_SERVER, get_info=ALL)

print("Connecting to Active Directory...")
with Connection(server, user=LDAP_USER, password=LDAP_PASSWORD, auto_bind=True) as conn:
    print("Authentication successful!")
    
    # Define LDAP search filter
    search_filter = "(&(objectClass=user)(sAMAccountName=ashwani))"
    
    # Search directory
    conn.search(
        search_base=SEARCH_BASE,
        search_filter=search_filter,
        attributes=['cn', 'mail', 'sAMAccountName', 'memberOf']
    )
    
    # Print entries
    for entry in conn.entries:
        print("--------------------------------------------------")
        print(f"Name: {entry.cn}")
        print(f"Email: {entry.mail}")
        print(f"Username: {entry.sAMAccountName}")
        print(f"Groups: {entry.memberOf}")
        print("--------------------------------------------------")
```
2. Execute search:
```bash
python ad_search.py
```

### Step 3: Write a Bulk Account Creation Script
We will write a script that reads a CSV list of users and provisions them inside an OU named `Employees`.

1. Create `ad_bulk_provision.py`:
```python
import csv
from ldap3 import Server, Connection, ALL, MODIFY_REPLACE

LDAP_SERVER = "192.168.1.10"
LDAP_USER = "CORP\\Administrator"
LDAP_PASSWORD = "SecretPassword123"

# CSV Data Setup
csv_data = """firstname,lastname,username,email
Amit,Kumar,akumar,akumar@corp.local
Priya,Sharma,psharma,psharma@corp.local"""

with open("users.csv", "w") as f:
    f.write(csv_data.strip())

def provision_users():
    server = Server(LDAP_SERVER, get_info=ALL)
    conn = Connection(server, user=LDAP_USER, password=LDAP_PASSWORD, auto_bind=True)
    
    with open("users.csv", mode='r') as file:
        reader = csv.DictReader(file)
        for row in reader:
            # Construct Distinguished Name
            user_dn = f"CN={row['firstname']} {row['lastname']},OU=Employees,DC=corp,DC=local"
            
            # Attributes for user creation
            attrs = {
                'objectClass': ['top', 'person', 'organizationalPerson', 'user'],
                'sAMAccountName': row['username'],
                'userPrincipalName': row['email'],
                'givenName': row['firstname'],
                'sn': row['lastname'],
                'displayName': f"{row['firstname']} {row['lastname']}",
                'mail': row['email']
            }
            
            print(f"Creating user: {row['username']}...")
            try:
                conn.add(user_dn, attributes=attrs)
                if conn.result['description'] == 'success':
                    print(f"User {row['username']} created successfully.")
                else:
                    print(f"Failed to create user {row['username']}: {conn.result['description']}")
            except Exception as e:
                print(f"Error provisioning {row['username']}: {e}")
                
    conn.unbind()

if __name__ == "__main__":
    provision_users()
```
2. Run script:
```bash
python ad_bulk_provision.py
```

---
## Cheat Sheet / Quick Reference

| Method / Library | Scope | Purpose / Syntax Example |
|---|---|---|
| `Server(ip, get_info=ALL)` | Config | Instantiates Domain Controller settings |
| `Connection(srv, user, pwd)`| Connect | Creates binding connection object |
| `conn.bind()` | Action | Performs simple LDAP authentication check |
| `conn.search(...)` | Action | Searches directories matching filters |
| `conn.add(dn, attributes)` | Modify | Creates new Active Directory object |
| `conn.modify(dn, changes)` | Modify | Updates values of an existing AD object |
| `conn.delete(dn)` | Modify | Removes specified AD object from hierarchy |

---
## Troubleshooting Table

| Problem | Cause | Fix | Command |
|---|---|---|---|
| LDAP connection fails: `LDAPBindError: invalidCredentials`. | The username/password is wrong, or user format is incorrect. | Ensure sAMAccountName includes domain prefix (`CORP\user`) or use UPN (`user@corp.local`). | N/A |
| User modification fails when resetting passwords. | Active Directory requires secure connections (`LDAPS`) for password updates. | Enable LDAPS certificate on DC and run connection on port 636 with SSL: `use_ssl=True`. | N/A |
| Connection times out or host unreachable. | Firewall is blocking ports `389` / `636` between host and Domain Controller. | Test connectivity using test-netconnection on port 389. | `test-netconnection <DC-IP> -port 389` |
| Creation fails with: `LDAPAttributeOrValueExistsResult`. | The object DN or sAMAccountName already exists in AD. | Implement check logic: Query username via `conn.search()` before running `conn.add()`. | N/A |
| Object creation fails with: `noSuchObject`. | The parent Organization Unit (OU) in DN path does not exist. | Create the target OU in ADUC first, or check spelling in DN path. | N/A |

---
## Interview Questions

> [!question] L1 Question
> **Q:** What is the difference between LDAP and LDAPS, and why is it important when managing users?
> **A:** **LDAP** communicates in plain text over port `389`, meaning anyone sniffing network traffic can read directory queries and passwords. **LDAPS** runs over port `636` using SSL/TLS encryption. Secure communication (LDAPS) is mandatory for password-related actions (like resetting passwords or setting passwords on new users) to prevent data intercept.

> [!question] L2 Question
> **Q:** How do you formulate an LDAP filter to find all disabled user accounts in a domain?
> **A:** To find disabled accounts, we query the `userAccountControl` attribute. The bitmask code `2` indicates a disabled account. The filter is:
> `(&(objectCategory=person)(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=2))`
> This filter matches objects that are users and uses the LDAP Matching Rule `1.2.840.113556.1.4.803` (LDAP_MATCHING_RULE_BIT_AND) to check if the disabled bit (2) is set.

> [!question] L3/Scenario Question
> **Q:** You are tasked with migrating user accounts from a legacy AD domain to a new one. A Python script copies attributes, but password hashes cannot be extracted from Active Directory database directly. How would you solve user password allocation on the new domain?
> **A:** 
> - **Situation:** AD password migration cannot extract raw hashes due to NTDS.dit security configurations.
> - **Task:** Assign secure, temporary passwords to migrated users and force changes on first login.
> - **Action:** 
>   1. **Generate Temporary Passwords**: I will generate a unique, cryptographically secure random password for each user using Python's `secrets` module.
>   2. **Secure Provisioning**: Connect to the destination domain controller over LDAPS (port 636) and set the user's password.
>   3. **Force Change Flag**: Update the user's `pwdLastSet` attribute to `0`. This forces AD to prompt the user to change their password immediately upon their first logon.
>   4. **Delivery**: Securely deliver the temporary passwords to users via an encrypted channel (like SMS, manager email, or password vault).
> - **Result:** Accounts are migrated securely, identity integrity is preserved, and compliance standards are met.

---
## Seedha Simple Mein
*Seedha simple mein: Python scripts Active Directory se direct communication ke liye `ldap3` library use karti hain. Hum port 389 unencrypted kaam ke liye aur password change jaise secure jobs ke liye port 636 (LDAPS) use karte hain. Script user ki configuration check karti hai, new accounts configure karti hai, aur bulk tasks ko easy banati hai.*

---
## Related Notes
- [[03-Identity-and-Core-Services/06-Active-Directory/WS-02 AD DS Installation|WS-02 AD DS Installation]] — Configuring directory server roles.
- [[05-Automation-and-Ticketing/10-Scripting-PowerShell/PS-02 PowerShell for Active Directory|PS-02 PowerShell for Active Directory]] — Standard PowerShell command options.
- [[05-Automation-and-Ticketing/15-Python-Scripting/PY-01 Python-for-Sysadmins|PY-01 Python-for-Sysadmins]] — Core Python systems libraries.

---
*Tags: #desktop-support #automation #active-directory #python #L3*
