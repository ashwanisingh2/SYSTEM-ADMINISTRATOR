---
tags: [desktop-support, linux, rhel, L1]
aliases: [l-06-file-permissions-and-ownership, l-06]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #rhcsa
---

# L-06: File Permissions and Ownership

> [!abstract] Overview
> This note covers the Linux file permission model, detailing standard rwx permissions for files and directories, octal/symbolic notation, default umask calculations, special permissions (SUID, SGID, Sticky Bit), and Access Control Lists (ACLs).

---

---
## Concept Overview
Think of file permissions in Linux like a keycard security gate:
- Every file has three groups of users: the **Owner** (you), the **Group** (your team), and **Others** (guests).
- For **Files**, the rules are simple: **Read** allows you to open and view the document, **Write** allows you to edit it, and **Execute** allows you to run it (if it's a program).
- For **Directories (Folders)**, the keys change: 
  - **Read (`r`)** is the directory list (allows you to run `ls` to see what folders are inside).
  - **Write (`w`)** is the authority to add or delete files *inside* the folder.
  - **Execute (`x`)** is the authority to enter the folder (run `cd` to make it your current path). Without `x`, you cannot read files inside even if you have read keys to the files themselves.


---

---
## Technical Deep Dive
### 1. Permissions: Files vs. Directories
The standard permissions (Read, Write, Execute) have different meanings depending on whether they are applied to files or directories:

| Permission | Applied to File | Applied to Directory |
|---|---|---|
| **Read (r / 4)** | View file contents (e.g., using `cat`, `less`). | List files inside directory (runs `ls`). |
| **Write (w / 2)** | Modify file contents. | Create, delete, or rename files inside directory. |
| **Execute (x / 1)**| Run the file as an executable program/script. | Enter the directory (runs `cd`), access file metadata. |

*Warning: If a user has Write (`w`) access on a directory, they can delete files inside that directory even if they have zero permissions on the files themselves.*

### 2. Numeric (Octal) vs. Symbolic Permissions
Permissions are represented as three octal digits (0-7) calculated by summing the binary values of the active permission bits:
- **`r` (Read) = 4**
- **`w` (Write) = 2**
- **`x` (Execute) = 1**
- **`-` (No Permission) = 0**

- **Common Permissions:**
  - **`755`** (`rwxr-xr-x`) — Owner has full access (`4+2+1=7`); Group and Others have read and execute access (`4+1=5`). Standard for directories.
  - **`644`** (`rw-r--r--`) — Owner has read/write (`4+2=6`); Group and Others have read-only (`4`). Standard for regular files.
  - **`777`** (`rwxrwxrwx`) — Everyone has full read/write/execute rights. Security hazard.
- **chmod Usage:**
  - *Numeric:* `chmod 755 script.sh`
  - *Symbolic:* `chmod u+x,g-w,o=r script.sh` (User add execute, Group remove write, Others set to read-only).

### 3. Ownership Management
- **`chown` (Change Owner):**
  - `chown sysadmin file.txt` — Change owner user.
  - `chown sysadmin:developers file.txt` — Change owner user and owner group simultaneously.
  - `chown -R sysadmin:developers /data` — Change ownership recursively on all subfolders.
- **`chgrp` (Change Group):**
  - `chgrp developers file.txt` — Change owner group only.

### 4. Default Permissions: umask
When a new file or directory is created, it is assigned default permissions filtered by the system's **`umask` (user mask)** value.
- **System Max Defaults:** Files default to `666` (no execute by default); Directories default to `777`.
- **The Calculation:** $\text{Default Permission} = \text{Max Default} - \text{umask}$.
  - *Example:* If system umask is **`022`**:
    - Directory default: $777 - 022 = \mathbf{755}$ (`rwxr-xr-x`).
    - File default: $666 - 022 = \mathbf{644}$ (`rw-r--r--`).
  - *Example:* If system umask is **`007`** (restrictive):
    - Directory default: $777 - 007 = \mathbf{770}$ (`rwxrwx---`).

### 5. Special Permissions

- **SUID (Set User ID - Octal Value 4000):**
  - **Represented by:** `s` in owner's execute field (e.g., `-rwsr-xr-x`).
  - **Effect:** The file runs with the privileges of the file **owner** rather than the user executing the file.
  - *Example:* The `/usr/bin/passwd` command has SUID set. Standard users can run it to modify `/etc/shadow` (which is root-only) because the process runs with root privileges.
- **SGID (Set Group ID - Octal Value 2000):**
  - **Represented by:** `s` in group's execute field (e.g., `drwxrwsr-x`).
  - **Effect (on Directories):** Any new files created inside the directory automatically inherit the **owner group** of the directory, rather than the primary group of the creating user. Crucial for shared team folders.
- **Sticky Bit (Octal Value 1000):**
  - **Represented by:** `t` in others' execute field (e.g., `drwxrwxrwt`).
  - **Effect (on Directories):** Only the file owner, directory owner, or root can delete or rename files inside the folder.
  - *Example:* Applied to `/tmp` so users cannot delete each other's temporary files.

### 6. Access Control Lists (ACLs)
Standard Linux permissions are limited to Owner-Group-Others. If a file needs custom permissions for a *fourth* user or group, use ACLs.
- **`getfacl file.txt`** — View active ACL permissions.
- **`setfacl -m u:jdoe:rx file.txt`** — Modify (`-m`) user (`u:`) `jdoe` access to read-execute.
- **`setfacl -x g:marketing file.txt`** — Remove (`-x`) group (`g:`) marketing permissions.
- **`setfacl -b file.txt`** — Remove all custom ACLs (returns to standard permissions).
- *Visual Indicator:* A file with active ACLs displays a `+` symbol at the end of the permissions string in `ls -l` (e.g., `-rw-r--r--+`).

---

## Common Mistakes
> [!warning] Avoid These
> **Setting 777 permissions to solve access errors:** Running `chmod -R 777 /var/www` to fix application read errors. This allows any local user or compromised service to modify, overwrite, or inject malicious code into your web application files.
> **Correct approach:** Identify the specific service user (e.g., `nginx`), chown the directory owner to that user, and set secure permissions (`755` for directories, `644` for files).

---

## Pro Tips
> [!tip] Field Experience
> When configuring SGID on shared folders, always set the directory's **default ACL** as well. While SGID ensures correct group ownership, it does not guarantee file write permissions for group members if a user's umask defaults to `022`. Setting a default ACL ensures all new files get explicit `rw` access for group members: `setfacl -d -m g:marketing:rw /shared_marketing`.

---

---

### Enterprise RHEL Service & Network Configurations

#### 1. Custom Systemd Service Creation
Create a custom systemd service configuration file `/etc/systemd/system/myapp.service`:
```ini
[Unit]
Description=My Custom Enterprise Application
After=network.target

[Service]
Type=simple
User=sysadmin
ExecStart=/usr/bin/python3 /opt/myapp/server.py
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
Enable and start the service:
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now myapp.service
```

#### 2. Log Rotation & Rsyslog Configuration
Configure log rotation rule in `/etc/logrotate.d/myapp` for automatic log cleaning:
```text
/var/log/myapp/*.log {
    daily
    rotate 7
    compress
    delaycompress
    missingok
    notifempty
    create 0660 sysadmin sysadmin
}
```
Define Rsyslog rule in `/etc/rsyslog.d/50-myapp.conf` to redirect application logs to a dedicated file:
```text
if $programname == 'myapp' then /var/log/myapp/syslog.log
& stop
```
Restart Rsyslog service:
```bash
sudo systemctl restart rsyslog
```

#### 3. Network Bonding & Teaming (LACP Link Aggregation)
Create a network team interface config file `/etc/sysconfig/network-scripts/ifcfg-team0` (RHEL standard):
```text
DEVICE=team0
DEVICETYPE=Team
BOOTPROTO=none
IPADDR=192.168.1.100
PREFIX=24
GATEWAY=192.168.1.1
ONBOOT=yes
TEAM_CONFIG='{"runner": {"name": "lacp"}}'
```
Bind slave physical interfaces (e.g., `eth1`) to the team interface:
```text
# /etc/sysconfig/network-scripts/ifcfg-eth1
DEVICE=eth1
ONBOOT=yes
TEAM_MASTER=team0
DEVICETYPE=TeamPort
```

---
## Step-by-Step Lab
> [!info] Lab Setup Needed
> A running Linux VM with root access, and a standard user `sysadmin` and group `marketing` created.

### Step 1: Create a Shared Directory with SGID
1. Log in as root. Create folder `/shared_marketing`:
   ```bash
   mkdir /shared_marketing
   ```
2. Change group owner to `marketing`:
   ```bash
   chown :marketing /shared_marketing
   ```
3. Set permissions: Owner root has full access, group marketing has read-write-execute, others have no access. Set **SGID** so files inherit the `marketing` group:
   ```bash
   chmod 2770 /shared_marketing
   ```
4. Verify permissions:
   ```bash
   ls -ld /shared_marketing
   ```
   **Verify:** Output should display `drwxrws---` (note the `s` in the group block).

### Step 2: Test SGID Inheritance
1. Switch to user `sysadmin` (ensure they are in the `marketing` group).
2. Create a test file inside the shared folder:
   ```bash
   touch /shared_marketing/test_file.txt
   ```
3. Check permissions of the new file:
   ```bash
   ls -l /shared_marketing/test_file.txt
   ```
   **Verify:** Even though `sysadmin` created the file, the group owner is automatically set to **`marketing`** instead of the user's primary group, allowing team access.

### Step 3: Configure Advanced ACLs
1. Supposed a third user `jdoe` (not in the marketing group) needs read-only access to `/shared_marketing/test_file.txt`.
2. Configure ACL:
   ```bash
   setfacl -m u:jdoe:r /shared_marketing/test_file.txt
   ```
3. Verify the ACL:
   ```bash
   getfacl /shared_marketing/test_file.txt
   ```
4. Confirm `user:jdoe:r--` is listed. Note that `ls -l` now displays a `+` sign.

---

---
## Cheat Sheet / Quick Reference
```bash
# Set SUID on script to run with owner privileges
chmod u+s script.sh

# Set Sticky Bit on shared directory
chmod +t /shared_data

# Remove all ACL overrides from configuration file
setfacl -b config.txt
```

---
| # | Concept | One Line Summary |
|---|---------|-----------------|
| 1 | Directory Exec | The execute (`x`) permission on a directory is required to enter it and read metadata. |
| 2 | SGID on Dir | Forces new files created inside to inherit the parent directory's group owner automatically. |
| 3 | Sticky Bit | Restricts file deletion/renaming inside a folder to the file owner, directory owner, or root. |
| 4 | umask | System mask subtracted from default permissions (777/666) to determine new file permissions. |
| 5 | setfacl | Configuration utility used to assign granular user/group permissions outside standard owners. |

---

---
## Troubleshooting
**Scenario 1:**
- **Problem:** A web server (Apache/NGINX) throws `HTTP 403 Forbidden` errors when clients try to open web pages. The index file `/var/www/html/index.html` has permissions `644` (`rw-r--r--`) and is owned by `root`.
- **Root Cause:** Directory permission failure. The web server process runs under the `apache` or `nginx` user. While the user has read access to `index.html` itself, one of the parent directories (e.g., `/var/www/html`) lacks execute (`x`) permissions for Others, blocking the service from entering the directory to read the file.
- **Fix:**
  1. Verify directory permissions:
     ```bash
     namei -om /var/www/html/index.html
     ```
  2. If any parent directory (e.g., `/var/www` or `/html`) displays `drwxrwx---`, modify permissions to grant execute access to Others:
     ```bash
     chmod o+x /var/www
     chmod o+x /var/www/html
     ```
  3. Re-test web page loading.

**Scenario 2:**
- **Problem:** A user attempts to create a file inside `/shared_data` but receives: `Permission Denied`. Running `ls -ld /shared_data` displays `drwxrwxr-x` and group owner is `developers`. Running `id` confirms the user is a member of the `developers` group.
- **Root Cause:** Group membership token cache drift. The user was added to the group during the active login session, and the active terminal process has not updated its token database.
- **Fix:**
  1. Run `id` inside the active shell. If the group does not show, run:
     ```bash
     exec su - $USER
     ```
  2. This forces a shell re-login, re-reading the `/etc/group` database and updating group tokens.
  3. Alternatively, use the `newgrp` tool to change active group scope: `newgrp developers`.

---

---
## Interview Questions
**Q1: What does it mean if a directory has permissions drwxrwxrwt? Explain the significance of the 't' bit.**
A: The `t` at the end indicates that the directory has **Sticky Bit** enabled alongside full read-write-execute permissions for everyone (such as `/tmp`). The sticky bit is a security mechanism that restricts deletion and renaming of files inside the directory. Even if others have write (`w`) access on the directory (which normally allows deleting any file inside), they can only delete or rename files that they physically own. This prevents users from deleting each other's temporary files.

**Q2: A developer needs read-write access to a configuration file owned by root:root. You cannot add them to the root group. How do you implement this safely?**
A: 
- **Situation:** A specific user needs access to a root-owned file without group addition.
- **Task:** Configure a custom access rule without changing primary file owners.
- **Action:** I will use Access Control Lists (ACLs). I will run the `setfacl` command to grant the specific developer user (e.g., `dev_user`) read-write (`rw`) access to the target configuration file.
  ```bash
  setfacl -m u:dev_user:rw /etc/app/config.conf
  ```
- **Result:** The developer obtains access, and I can verify it using `getfacl /etc/app/config.conf`.

**Q3: How do you calculate the default permissions of a new file and a new directory if the umask is set to 077?**
A: 
1. The maximum default permissions are: `666` for files (no execute by default) and `777` for directories.
2. The umask is `077`.
3. For a Directory: $777 - 077 = \mathbf{700}$ (Owner has full `rwx` access; Group and Others have no access `---`).
4. For a File: $666 - 077 = \mathbf{600}$ (Owner has read-write `rw-` access; Group and Others have no access `---`).
This is a highly restrictive umask configuration typical in high-security environments.

---

---
## Seedha Simple Mein
*Seedha simple mein: Linux mein har file and folder par User, Group, aur Others ke liye Read (4), Write (2), aur Execute (1) permissions set hoti hain. Special permissions jaise SUID/SGID/Sticky Bit aur Advanced ACLs (setfacl) complex permission requirements ko manage karte hain.*

---
## Related Notes
- [[02-Operating-Systems/04-Linux-RHEL/L-03 File System Management|L-03 File System Management]] — Directory structures and `ls -l` metadata reading.
- [[02-Operating-Systems/04-Linux-RHEL/L-05 User and Group Management|L-05 User and Group Management]] — Managing UIDs and GIDs.
- [[02-Operating-Systems/04-Linux-RHEL/L-11 File Systems and Storage in Linux|L-11 File Systems and Storage in Linux]] — Formatting and mounting volumes.
