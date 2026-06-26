---
tags: [windows-server, sql-server, database-admin, advanced]
aliases: [SQL Admin, MSSQL Admin, SQL Server Admin]
created: 2026-06-26
status: #complete
difficulty: #advanced
cert-relevant: #none
---

> [!NOTE|color-blue]
> 🏢 **WINDOWS SERVER**

`#complete` `#advanced` `#none`

# WS-19: SQL Server Administration

> [!abstract] Overview
> This note covers the essentials of Microsoft SQL Server administration for System Administrators. SQL Server is a highly robust relational database management system developed by Microsoft. As an infrastructure or support engineer, knowing how to install, configure, backup, restore, and troubleshoot SQL databases is crucial because many enterprise applications rely on it. *Enterprise applications ka backend database zyada tar SQL Server par hota hai, aur agar DB down hua, toh poori application down ho jayegi. Yeh note aapko basic se advanced SQL server tasks manage karne mein help karega.*

---
## 🧠 Concept Overview

- **What it is** — MS SQL Server is an enterprise-class Relational Database Management System (RDBMS) that stores and retrieves data requested by other software applications. It manages data in tables using relationships.
- **Why it matters** — Almost all Microsoft-centric web applications, internal tools (like SCCM, SharePoint, SCOM), and ERP systems require SQL databases to store application data securely and reliably. It provides high availability and disaster recovery features.
- **Where you see this** — You will see SQL servers in multi-tier applications. Every time a user logs into an enterprise web app, processes a payroll transaction, or views a business report, SQL Server is working behind the scenes.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic task — Check SQL Server service status (`services.msc`), verify disk space on database drives, perform basic connectivity checks (ping, SSMS login), and report errors. |
| **L2** | Configure, fix, escalate — Perform DB backups/restores, manage SQL logins/permissions, check SQL error logs, expand disk volumes, and shrink transaction logs. |
| **L3** | Architecture, design, enterprise-level — Configure Always On Availability Groups, failover clustering, perform deep performance tuning, query optimization, and large-scale migrations. |

> [!tip] Seedha Simple Mein
> *SQL Server ek digital godown (warehouse) ki tarah hai jahan saara data tables aur rows ke form mein safe rakha jaata hai. Hamara kaam as an admin is godown ki security, backup aur speed maintain karna hai. Developers code likhte hain, hum environment sambhalte hain.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **A Bank Locker System** is like **SQL Server** because...
>
> - Bank mein lockers hote hain jahan alag-alag log apna samaan rakhte hain; *waise hi SQL server mein alag-alag databases hote hain.*
> - Bank manager decides karta hai kisko kis locker ki chaabi (keys) milegi; *waise hi SQL Admin (sysadmin) decide karta hai kaunsa user kis database ko read/write kar sakta hai.*
> - Agar bank manager har raat records ka backup copy banata hai toh disaster mein data loss nahi hoga; *waise hi SQL admin regular backups configure karta hai taaki server crash hone par data wapas laya ja sake.*
> - Bank mein security guards hote hain; *waise hi SQL mein firewalls aur AD authentication hoti hai.*

---
## 🔬 Technical Deep Dive

### 1. SQL Server Instances & Databases

> [!info] Key Concept
> An **Instance** is a single installation of SQL Server. You can have multiple instances on a single server machine to isolate environments (e.g., DEV and PROD). A **Database** resides within an instance and holds the actual tables, views, stored procedures, and data.

- **Default Instance:** Accessed via the server's hostname or IP address (e.g., `SERVER01`). It listens on TCP port 1433 by default. There can only be one default instance per machine.
- **Named Instance:** Accessed via `ServerName\InstanceName` (e.g., `SERVER01\SQLEXPRESS`). Listens on dynamic ports unless configured statically by the administrator.

### 2. System Databases
Every SQL Server installation includes several built-in system databases essential for its operation:
- **master:** *SQL server ka dimaag.* Records all system-level information (logins, linked servers, system configuration settings). If `master` is down, SQL Server cannot start.
- **model:** *Template hota hai.* Used as a template for creating new user databases. Any modifications made to the `model` database will be reflected in all databases created thereafter.
- **msdb:** *Task scheduler.* Used by SQL Server Agent for scheduling jobs, setting alerts, sending emails, and recording backup/restore history.
- **tempdb:** *Rough copy book.* A temporary workspace used to hold temporary tables, intermediate result sets, and sorting operations. Recreated fresh every time the SQL Server service restarts.

> [!danger] Common Mistake
> Never put user data or application tables in system databases (`master`, `model`, `msdb`). *Agar aapne galti se master DB mein apni table bana di, aur system crash hua, toh recovery bahut complex ho jayegi.* Always create a dedicated User Database.

### 3. Authentication Modes
SQL Server supports two authentication modes for access control:
- **Windows Authentication Mode:** Uses Active Directory credentials. SQL Server trusts the Windows token provided by the user. *Sabse secure tarika hai kyunki passwords AD mein manage hote hain aur password policies automatically apply hoti hain.*
- **Mixed Mode (Windows Authentication and SQL Server Authentication):** Allows both AD accounts and local SQL Server accounts. Useful when non-Windows applications (like Linux-based Java apps) need to connect using a SQL-specific username and password (like the default `sa` account).

### 4. Recovery Models
The recovery model determines how transactions are logged and dictates the backup and restore capabilities:
- **Simple:** No transaction log backups are allowed. Space in the log file is automatically reclaimed after checkpoints. Good for development/test environments or data warehouses where data is read-only.
- **Full:** All transactions are strictly logged. Allows point-in-time recovery to the exact minute/second before a disaster. Requires administrators to perform both full database backups AND regular transaction log backups. *Production databases hamesha Full recovery mode mein hone chahiye.*
- **Bulk-Logged:** Similar to Full, but minimally logs bulk operations (like bulk inserts) to save space and improve performance during massive data loads.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - A Windows Server with MS SQL Server installed.
> - SQL Server Management Studio (SSMS) installed.
> - Admin access (`sysadmin` server role) to the SQL Instance.
> - Basic understanding of T-SQL syntax.

### Step 1: Taking a Full Database Backup via T-SQL

*Jab bhi koi critical activity ho (jaise application upgrade), sabse pehle database ka full backup lena zaruri hai.*

**Using T-SQL:**
```sql
-- Yeh command 'MyProductionDB' ka full backup legi aur specified location pe save karegi.
-- Hamesha verify karein ki backup drive mein enough space hai.
BACKUP DATABASE [MyProductionDB] 
TO DISK = N'D:\SQLBackups\MyProductionDB_Full.bak' 
WITH NOFORMAT, NOINIT,  
NAME = N'MyProductionDB-Full Database Backup', 
SKIP, NOREWIND, NOUNLOAD, STATS = 10;
GO
```

> [!success] Expected Output
> ```
> 10 percent processed.
> 20 percent processed.
> ...
> 100 percent processed.
> Processed 3450 pages for database 'MyProductionDB', file 'MyProductionDB' on file 1.
> Processed 20 pages for database 'MyProductionDB', file 'MyProductionDB_log' on file 1.
> BACKUP DATABASE successfully processed 3470 pages in 0.450 seconds (600.052 MB/sec).
> ```

### Step 2: Restoring a Database

*Agar data corrupt ho jaye ya user galat data delete kar de, toh hum backup file se database ko wapas restore karte hain.*

```sql
-- Yeh command backup file se database ko nayi location par restore karegi as a copy
RESTORE DATABASE [MyProductionDB_Restored] 
FROM DISK = N'D:\SQLBackups\MyProductionDB_Full.bak' 
WITH FILE = 1,  
MOVE N'MyProductionDB' TO N'E:\SQLData\MyProductionDB_Restored.mdf',  
MOVE N'MyProductionDB_log' TO N'F:\SQLLogs\MyProductionDB_Restored_log.ldf',  
NOUNLOAD,  REPLACE,  STATS = 5;
GO
```

### Step 3: Creating a SQL Login and Mapping to a Database

*Application ko backend database se connect karne ke liye specific credentials aur permissions chahiye hote hain.*

```sql
-- 1. Server level par login banayein
CREATE LOGIN [AppUser] WITH PASSWORD=N'StrongPass123!';
GO

-- 2. Specific database context mein switch karein
USE [MyProductionDB];
GO

-- 3. Database mein user account banayein jo server login se mapped ho
CREATE USER [AppUser] FOR LOGIN [AppUser];
GO

-- 4. Database user ko Read aur Write roles assign karein
ALTER ROLE [db_datareader] ADD MEMBER [AppUser];
ALTER ROLE [db_datawriter] ADD MEMBER [AppUser];
GO
```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command (T-SQL) / PowerShell | 🛠️ Kya karta hai | 📝 Example |
|---------------------------------|-----------------|-----------|
| `sp_who2` | Shows all active user connections and what queries they are executing. Helpful to find blocked processes. | `EXEC sp_who2;` |
| `DBCC SQLPERF(LOGSPACE);` | Checks transaction log size and the percentage of space used for all databases. | `DBCC SQLPERF(LOGSPACE);` |
| `DBCC CHECKDB` | Checks logical and physical integrity of all objects in the specified database. Identifies corruption. | `DBCC CHECKDB ('MyProductionDB');` |
| `Get-Service *SQL*` | PowerShell command to list the status of all SQL-related services on the server. | `Get-Service *SQL* \| Format-Table` |
| `Restart-Service "MSSQLSERVER"` | Restarts the default SQL Server service via PowerShell. | `Restart-Service "MSSQLSERVER" -Force` |
| `SELECT @@VERSION;` | Returns the current installed version and edition of the SQL Server instance. | `SELECT @@VERSION;` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| **Login failed for user 'sa' (Error 18456)** | Incorrect password, or the SQL instance does not have Mixed Mode Authentication enabled. | Open Server Properties -> Security -> Change auth mode to Mixed Mode, restart SQL service, and reset 'sa' password. |
| **Transaction log is full (Error 9002)** | Database is in Full Recovery Mode and transaction log backups are not running, causing the `.ldf` file to fill the entire disk. | Run an immediate Log Backup: `BACKUP LOG [DB] TO DISK...`, or temporarily switch to Simple recovery mode, shrink log, and switch back to Full. Fix the backup jobs. |
| **Database is in "Recovery Pending" state** | SQL Server cannot open the underlying database files (MDF/LDF) due to missing files, disk unavailability, or severe corruption. | Check Windows Event Logs and SQL Error logs. Verify the SAN/disk is online. Run `DBCC CHECKDB` in emergency mode if files are corrupted. |
| **Cannot connect to SQL Server (Network error 40)** | SQL Service is stopped, TCP/IP protocol is disabled, or Windows Firewall is blocking the SQL listening port (1433). | Start the SQL Service. Enable TCP/IP in SQL Server Configuration Manager. Create an inbound rule in Windows Firewall to allow port 1433. |
| **High CPU or Memory usage by sqlservr.exe** | Poorly written queries scanning massive tables, missing indexes, or SQL Server dynamic memory allocation taking all OS RAM. | Cap the "Maximum server memory" setting in SQL Server properties (leave 4-8GB for OS). Use Activity Monitor or `sp_who2` to find and kill blocking queries. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Database Disk Full Issue

> [!example] Ticket
> "Critical Alert: Drive F: (SQL Logs) is at 99% capacity on SERVER-SQL-01. Applications are starting to fail and users cannot save data."

**L1 Response:** RDP into the server, check which folder is consuming space. Verify it is the `.ldf` (Transaction Log) files. Note down the exact database name causing it.
**Escalation Trigger:** If L1 doesn't have `sysadmin` permissions to run SQL commands or the disk cannot be extended immediately at the VM/SAN level.
**L2 Resolution:**
1. Connect to the instance via SSMS.
2. Check why logs are growing (*log backups fail ho gaye honge ya chal nahi rahe honge*).
3. Take an immediate transaction log backup: `BACKUP LOG [DBName] TO DISK='D:\TempBackup\EmergencyLog.trn'`.
4. Run a shrink operation to reclaim space at the OS level: `DBCC SHRINKFILE (N'DBName_Log', 1024)`.
5. Fix the broken SQL Server Agent job that is supposed to handle regular transaction log backups.

### 🎫 Scenario 2: Application Cannot Connect to SQL Server

> [!example] Ticket
> "Users are getting a yellow screen error on the HR Web Portal: 'A network-related or instance-specific error occurred while establishing a connection to SQL Server'."

**L1 Response:** Ping the SQL Server from the app server. Log into the SQL Server OS and check if the `MSSQLSERVER` service is running in `services.msc`.
**Escalation Trigger:** Service is running, server is reachable via ping, but the application connection still fails.
**L2 Resolution:**
1. Open **SQL Server Configuration Manager**.
2. Navigate to SQL Server Network Configuration -> Protocols. Ensure **TCP/IP** is Enabled.
3. Check Windows Firewall to ensure Port 1433 (Default) or the specific dynamic port is allowed for inbound traffic.
4. Verify port connectivity using PowerShell from the application server: `Test-NetConnection -ComputerName SERVER-SQL-01 -Port 1433`.
5. Check if the SQL Server browser service is running if using a named instance.

### 🎫 Scenario 3: Accidental Data Deletion (Point-in-Time Restore)

> [!example] Ticket
> "Urgent: A junior developer accidentally dropped a critical configuration table in the DEV database at 2:15 PM today. We need to restore the database to how it was exactly at 2:10 PM."

**L1 Response:** Acknowledge the ticket immediately. Escalate to L2/L3 immediately as this requires DBA-level access and point-in-time restore procedures. Do NOT attempt without experience.
**Escalation Trigger:** Data loss incident requires immediate and precise database restore.
**L2 Resolution:**
1. Verify the database is in Full Recovery mode.
2. Take a "Tail-Log" backup to capture all transactions up to the current point to ensure no other data is lost.
3. Restore the last Full Backup `WITH NORECOVERY`.
4. Restore the latest Differential backup (if any) `WITH NORECOVERY`.
5. Restore Transaction Log backups in sequence, using the `STOPAT` clause: `RESTORE LOG [DB] FROM DISK='...' WITH STOPAT = '2026-06-26 14:10:00', RECOVERY`.
6. Verify with the developer that the table is back and data is consistent.

---
## 🎤 Interview Questions

> [!question] Q1: What is the exact difference between Simple and Full recovery models in SQL Server?
> **Answer:** In the Simple recovery model, transaction logs are automatically truncated to save disk space, but you cannot restore to a specific minute in time (you can only restore to the last full/diff backup). In the Full recovery model, all transactions are kept in the log until a log backup is explicitly taken. This allows for precise point-in-time recovery, which is essential for enterprise production databases.
==**Exam Tip:** Production databases = Full Recovery. Development/Test = Simple Recovery.==

> [!question] Q2: How do you fix a transaction log that has grown too large and filled up the disk drive?
> **Answer:** First, you must take a Transaction Log Backup to truncate the inactive portions of the log file. Then, you use the `DBCC SHRINKFILE` command to shrink the physical `.ldf` file and reclaim the disk space for the operating system. *Agar disk space bilkul zero hai, toh pehle disk space add karo ya temporary files delete karo taaki log backup chal sake, phir shrink karo.*
==**Exam Tip:** Never just delete the .ldf file from Windows Explorer! It will immediately corrupt the database.==

> [!question] Q3: What is the primary purpose of the `tempdb` system database?
> **Answer:** `tempdb` acts as a global workspace for SQL Server. It holds temporary tables, intermediate query results, temporary stored procedures, and handles internal sorting operations. It is completely wiped and recreated fresh every time the SQL Server service restarts.
==**Exam Tip:** Performance bottlenecks in SQL often track back to `tempdb` contention. Always try to keep `tempdb` data files on the fastest available disks (NVMe/SSD).==

> [!question] Q4: What is the difference between a SQL Login and a Database User?
> **Answer:** A SQL Login provides access at the server/instance level (Authentication). It determines who can log into the SQL Server itself. A Database User is mapped to that login and provides access and permissions within one specific database (Authorization). *Login se aap building (server) mein ghuste hain, User ID se aap specific room (database) mein ghuste hain.*
==**Exam Tip:** Logins live in the `master` system database; Users live in the specific user database they are granted access to.==

> [!question] Q5: What does the command `DBCC CHECKDB` do, and when should you run it?
> **Answer:** `DBCC CHECKDB` checks the logical and physical integrity of all objects in the specified database. It ensures that the database files are not corrupt and the data pages are consistent. It should be run as part of regular maintenance tasks (e.g., weekly) to catch corruption early before taking backups.
==**Exam Tip:** Running `DBCC CHECKDB` on a large database can be very resource-intensive, so schedule it during off-peak hours.==

---
## 🔗 Related Notes

- [[WS-01 Windows Server Basics|Windows Server Basics]] — For underlying OS concepts, services, and disk management.
- [[AD-01 Active Directory Overview|Active Directory Overview]] — Active Directory concepts related to Windows Authentication and service accounts in SQL.
- [[PS-01 PowerShell Basics|PowerShell Basics]] — Managing SQL Services and automating admin tasks via PowerShell.
- [[AZ-04 Azure SQL Fundamentals|Azure SQL Fundamentals]] — Cloud equivalent (PaaS) of on-prem SQL server.
