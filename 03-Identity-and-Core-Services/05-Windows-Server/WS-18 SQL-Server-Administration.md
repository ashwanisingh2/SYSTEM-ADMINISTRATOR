---
tags: [desktop-support, windows-server, sql, databases, L2]
aliases: [sql-server-administration, sql-basics]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #none
---

# WS-18: SQL Server Administration

> [!note] Overview
> This note covers SQL Server administration fundamentals for systems administrators. It details database components (MDF/LDF), recovery models (Simple vs. Full), backup and restore procedures, authentication modes, SQL Agent automation, and common database troubleshooting scenarios.

---
## Concept Overview
- **What it is** — Microsoft SQL Server is a relational database management system (RDBMS) that hosts and stores database tables for enterprise services, including MECM/SCCM, WSUS, Active Directory Certificate Services, web portals, and custom corporate applications.
- **Why it matters** — A systems administrator does not need to be a professional Database Administrator (DBA), but they must understand database operations. When a database server runs out of disk space, experiences connection issues, or gets corrupted, it brings down all dependent IT infrastructure.
- **Real job encounter** — Restoring a backup database to test environments, shrinking transaction log files that filled up a server hard drive, configuring SQL permissions for domain users, and resolving connection timeouts.
- **L1 vs L2 vs L3 responsibility split:**
  - **L1 Resolution**: Verify SQL Server services are running, monitor database disk space allocation, restart failed SQL Agent service, and log client connection errors.
  - **Escalation Trigger**: Escalate if database operations timeout, if transaction logs exhaust storage drives, or if database integrity checks report corruption errors.
  - **L2 Resolution**: Perform database backups and restores, configure database users and SQL logins, create automated SQL Agent backup jobs, and shrink transaction logs.
  - **L3 Resolution**: Implement SQL Server clusters and AlwaysOn Availability Groups, optimize database memory limits, configure database transaction log shipping, and troubleshoot locking/deadlocking issues.

---
## Technical Deep Dive

### 1. Database File Architecture
A Microsoft SQL Server database consists of at least two physical operating system files:
- **Primary Data File (`.mdf`)**: Stores the actual data tables, schemas, indexes, views, and system catalog data.
- **Secondary Data Files (`.ndf`)**: Optional files used to spread data across multiple physical drives to optimize disk I/O performance.
- **Transaction Log File (`.ldf`)**: Logs all database modifications (inserts, updates, deletes) before they are written to the data file. This ensures database transaction integrity (ACID properties) and is critical for database recovery.

### 2. Database Recovery Models
The recovery model determines how transaction logs are managed and how the database can be restored:
- **Simple Recovery**: Transaction logs are truncated automatically at checkpoints. Log files remain small, but you can only restore the database to the exact time of the last full or differential backup.
- **Full Recovery**: Transactions are written to the log file and remain there permanently until a Transaction Log Backup is executed. This model permits restoring the database to a specific second in the past (point-in-time restore), but log files will grow indefinitely until backed up, risking disk exhaustion.
- **Bulk-Logged**: A hybrid model optimizing log space during bulk-import operations.

```
       [ Client Database Write ]
                   |
                   v
      [ Written to Log (.ldf) ]
                   |
         +---------+---------+
         |                   |
         v                   v
   [ Simple Model ]    [ Full Model ]
         |                   |
    Automatically        Log remains
    truncated on        until backed
    Checkpoints.         up via Log
    Logs stay small.    Backup. Allows
                        Point-in-Time.
```

### 3. Authentication Modes
- **Windows Authentication (Recommended)**: Integrates with Active Directory. Users and service accounts log in using their Windows credentials without exposing passwords in connection strings.
- **Mixed Mode**: Allows both Windows Authentication and **SQL Server Authentication**. SQL Server maintains its own local logins (like the default administrative `sa` account) inside the database engine.

---
## Step-by-Step Lab

> [!warning] Pre-requisites
> - A Windows Server 2022 instance running SQL Server (Express, Developer, or Standard Edition).
> - SQL Server Management Studio (SSMS) installed locally.
> - Administrative access to the SQL Server instance.

### Step 1: Manage SQL Services via PowerShell
Verify service health and configure startup types to survive reboots.

1. Open PowerShell as Administrator and query SQL service states:
```powershell
# Get status of SQL Database Engine and SQL Server Agent
Get-Service -Name MSSQLSERVER, SQLSERVERAGENT
```
**Expected Output:**
```text
Status   Name               DisplayName
------   ----               -----------
Running  MSSQLSERVER        SQL Server (MSSQLSERVER)
Stopped  SQLSERVERAGENT     SQL Server Agent (MSSQLSERVER)
```
2. Start the SQL Server Agent (required for automated backup jobs) and set it to Automatic:
```powershell
Set-Service -Name SQLSERVERAGENT -StartupType Automatic
Start-Service -Name SQLSERVERAGENT
```

### Step 2: Back up a Database using T-SQL
We will execute a full backup of database `ClientDB` to local disk.

1. Open SSMS, click **New Query**, and execute:
```sql
-- Ensure directory C:\SQLBackups exists before running
BACKUP DATABASE [ClientDB]
TO DISK = 'C:\SQLBackups\ClientDB.bak'
WITH FORMAT,
MEDIANAME = 'SQLBackups',
NAME = 'Full Backup of ClientDB';
```
**Expected Output:** `BACKUP DATABASE successfully processed...`

### Step 3: Restore a Database using T-SQL
1. To restore `ClientDB` from the backup file, run:
```sql
-- Restore database replacing existing database if present
USE [master];
ALTER DATABASE [ClientDB] SET SINGLE_USER WITH ROLLBACK IMMEDIATE;

RESTORE DATABASE [ClientDB]
FROM DISK = 'C:\SQLBackups\ClientDB.bak'
WITH REPLACE,
MULTI_USER;
```
*Note: We force the database to Single_User mode to terminate active connections before running the restore.*

### Step 4: Add Windows logins and Map Permissions
1. Create a database login for a domain user `corp\jdoe` and grant read/write access:
```sql
-- Create SQL login for Windows AD account
CREATE LOGIN [corp\jdoe] FROM WINDOWS;

-- Map login to database user
USE [ClientDB];
CREATE USER [jdoe] FOR LOGIN [corp\jdoe];

-- Grant read and write permissions to user
ALTER ROLE [db_datareader] ADD MEMBER [jdoe];
ALTER ROLE [db_datawriter] ADD MEMBER [jdoe];
```

### Step 5: Shrink a Runaway Transaction Log
If a database in Full Recovery mode lacks regular log backups, the `.ldf` file will expand and fill the drive. We will truncate and shrink it.

1. Query file names inside the database:
```sql
USE [ClientDB];
EXEC sp_helpfile;
```
*Note the logical name of the log file (e.g. `ClientDB_log`).*

2. Change recovery model, shrink log to 10MB, and restore recovery model:
```sql
-- Temporarily set recovery to simple to truncate active logs
ALTER DATABASE [ClientDB] SET RECOVERY SIMPLE;

-- Shrink the physical log file to 10 MB
DBCC SHRINKFILE (ClientDB_log, 10);

-- Restore full recovery model
ALTER DATABASE [ClientDB] SET RECOVERY FULL;
```
**Expected Output:** The transaction log file shrinks to 10MB, freeing up local disk space.

---
## Cheat Sheet / Quick Reference

| Action / Command | SQL Query | Purpose / Example |
|---|---|---|
| Query Active Connections | `sp_who2` | Returns active logins and system resource usage |
| Kill Hung Session | `KILL <SPID>` | Terminates a blocked or hung connection thread |
| Check Database Integrity | `DBCC CHECKDB ('Name')` | Validates file system integrity and checks for corruption |
| Query DB Size | `sp_spaceused` | Displays space occupied by tables and indexes |
| Get-SqlDatabase | `Get-SqlDatabase -ServerInstance "localhost"` | PowerShell cmdlet to query database list |
| Set recovery model | `ALTER DATABASE [DB] SET RECOVERY SIMPLE;` | Changes recovery options |
| **Port 1433 (TCP)** | Default SQL Server port | Port opened on local firewalls for external database access |
| **Port 1434 (UDP)** | SQL Browser service port | Port utilized to query dynamic SQL named instances |

---
## Troubleshooting Table

| Problem | Cause | Fix |
|---|---|---|
| Application fails to connect, returning error: "Login failed for user..." | Authentication credentials do not match, or SQL server authentication is disabled. | Open SSMS -> Right-click SQL Server instance -> Properties -> Security. Ensure Server Authentication is set to **SQL Server and Windows Authentication mode**. |
| Connection failed: "A network-related or instance-specific error occurred..." | SQL Server is not configured to accept remote connections, or port 1433 is blocked. | Open SQL Server Configuration Manager. Go to SQL Server Network Configuration -> Protocols. Enable **TCP/IP**. Open Windows Firewall port TCP 1433. |
| SQL Agent service fails to start. | The service account password expired, or service is disabled. | Open Services console. Verify the service is not disabled. Update service login credentials in properties using a domain service account. |
| Database shows status: `SUSPECT` or `RECOVERY PENDING`. | Database files (`.mdf`/`.ldf`) are missing, locked, or corrupted. | Check system Event logs. Restore database from backup, or run emergency repair: `ALTER DATABASE [DB] SET EMERGENCY; DBCC CHECKDB ([DB], REPAIR_ALLOW_DATA_LOSS);`. |
| Queries run extremely slow, causing application timeout alerts. | CPU/RAM saturation, missing tables indexes, or deadlocking. | Run `sp_who2` to check for blocked sessions (`BlkBy` column). Identify blocking SPID and run `KILL <SPID>`. Rebuild indexes. |

---
## Interview Questions

> [!question] L1 Question
> **Q:** What is the difference between an `.mdf` file and an `.ldf` file in Microsoft SQL Server?
> **A:** The `.mdf` file is the primary data file that stores all the database tables, records, schemas, indexes, and stored procedures. The `.ldf` file is the transaction log file that tracks all data changes before they are committed to the `.mdf` file. This log ensures data consistency and transaction safety during sudden shutdowns or crashes.

> [!question] L2 Question
> **Q:** What is the difference between Simple and Full Recovery models in SQL Server? How does this affect log file growth?
> **A:** In **Simple Recovery**, transaction logs are automatically truncated during checkpoints, reclaiming log space and preventing log files from growing large. However, you can only restore data up to the time of your last full backup. In **Full Recovery**, transactions are stored in the log file permanently until a Transaction Log Backup is run. This allows point-in-time restores (down to the second), but the log file will grow indefinitely and fill the disk if log backups are not scheduled regularly.

> [!question] L3/Scenario Question
> **Q:** You are troubleshooting a corporate application crash. The application logs display a timeout error: "Transaction was deadlocked on lock resources with another process." What is a deadlock, and how do you resolve it?
> **A:** 
> - **Situation:** Application crash due to SQL database deadlocking.
> - **Task:** Identify the conflicting queries and resolve the lock block.
> - **Action:** 
>   1. **Understand Deadlock**: A deadlock occurs when Process A holds a lock on Table 1 and requests a lock on Table 2, while Process B holds a lock on Table 2 and requests a lock on Table 1. Neither process can proceed, creating an infinite block.
>   2. **Identify Blockers**: Open SSMS and run `sp_who2` or execute `EXEC sp_lock` to trace active locks. Alternatively, inspect the SQL Server Error Log or set up an Extended Events session to capture the deadlock graph.
>   3. **Resolve Lock**: Identify the blocking session process ID (SPID) and terminate it using the command `KILL <SPID>` to free resources.
>   4. **Optimize Code**: Request developers add indexes to the tables to speed up queries, or modify connection transactions to read database using the `NOLOCK` hint (if dirty reads are acceptable).
> - **Result:** The database releases the locks, allowing the application to resume queries.

---
## Seedha Simple Mein
*Seedha simple mein: SQL Server ek database engine hai jo data ko files (tables) mein store karta hai. MDF main data file hoti hai aur LDF transaction log file hoti hai. Agar SQL server slow chal raha hai ya connect nahi ho raha, toh service status check karna, remote TCP/IP connection enable karna aur port 1433 open karna priority troubleshooting steps hain.*

---

---
## Real-World Scenario: File Server Crash (500+ Users Affected)
Agar company ka main file server (500+ users connected) crash ho jaye, toh as a Senior Sysadmin aapka immediate action plan yeh hoga:
1. **Identify & Isolate**: Check if the server is pingable. Verify hardware status via Dell iDRAC / HPE iLO or check virtual machine status in vCenter/Hyper-V.
2. **Access Check & Lock**: If the storage volume is corrupt or unmapped, stop DFS Namespace referrals to the crashed target to redirect users to secondary replica servers if active.
3. **Trigger Disaster Recovery (DR)**: Start restoring the crashed system drive or volumes from the latest VSS snapshot or Azure Backup RSV restore point.
4. **Communication**: Broadcast status updates to the Incident Commander and helpdesk teams to manage incoming support volume.

### PowerShell Automation Snippet: Verify File Shares and Access Permissions
```powershell
# Get all active file shares on the server
Get-SmbShare | Format-Table -AutoSize

# Test local DFS Namespace server connection health
Test-DFSNamespaceTarget -Path "\\corp.local\DFSRoot\Public"
```

---
## Related Notes
- [[03-Identity-and-Core-Services/05-Windows-Server/WS-09 File Server and FSRM|WS-09 File Server and FSRM]] — Managing storage volumes hosting databases.
- [[03-Identity-and-Core-Services/05-Windows-Server/WS-17 SCCM-MECM|WS-17 SCCM-MECM]] — Utilizing SQL Server as backend database storage.
- [[05-Automation-and-Ticketing/10-Scripting-PowerShell/PS-03 PowerShell for System Administration|PS-03 PowerShell for System Administration]] — Running database queries via PowerShell.

---
*Tags: #desktop-support #windows-server #sql #databases #L2*
