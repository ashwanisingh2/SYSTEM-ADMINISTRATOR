---
tags: [sysadmin, linux, CLI, commands]
difficulty: Beginner
lab-required: Yes
read-time: 15 mins
---

# L-02: Command Line Basics

> [!abstract] Overview
> This note covers the basics of the Linux Command Line Interface (CLI). It details command syntax rules, active terminal shortcuts, and provides a reference matrix of fifty essential Linux commands.

---
## Concept
Think of the Command Line Interface (CLI) as a direct, written conversation with your computer. 
Using a Graphical User Interface (GUI) is like ordering food at a restaurant by pointing to pictures on a menu: it is easy, but you are limited to what the pictures show. 

Using the CLI is like talking directly to the head chef: you write out exactly what ingredients you want (options/flags) and what dish you want them applied to (arguments). It requires learning the recipe terms, but it allows you to customize and automate any dish imaginable.

*Seedha simple mein: CLI computer se direct keyboard ke zariye baat karne ka tool hai. Command execution ka basic format hota hai: `command [options] [arguments]`. Tab key aur history shortcuts work speed ko improve karte hain.*

---
## Technical Deep Dive

### 1. Command Syntax Structure
Linux commands follow a strict case-sensitive pattern:
$$\text{command} \quad \text{--options} \quad \text{arguments}$$
- **Command:** The binary executable or built-in tool name (e.g., `ls`).
- **Options (Flags):** Modifies how the command behaves. Usually prefixed with a single dash (`-`) for short options or double dashes (`--`) for long options (e.g., `-a` or `--all`).
- **Arguments:** The target of the command (e.g., a file path or directory name `/var/log`).

### 2. Tab Completion & History Shortcuts
- **Tab Completion:** Type the first few characters of a command or file path and press **Tab**. The shell auto-completes the name. Pressing Tab twice lists all matching options.
- **Command History:**
  - `history` — Lists all previously executed commands.
  - **`Ctrl + R`** — Launches reverse-i-search to dynamically search history as you type.
  - `!!` — Repeats the last command.
  - `!sudo` — Repeats the last command that started with `sudo`.

---
## 50 Essential Linux Commands Matrix

### Directory Navigation & Manipulation
1. `pwd` — Print working directory. E.g., `pwd` returns `/home/sysadmin`.
2. `ls -la` — List directory contents with long format (permissions, owner, size) and hidden files.
3. `cd /var/log` — Change directory to target path. `cd ~` goes home; `cd ..` goes up one level.
4. `mkdir -p /tmp/a/b/c` — Create directories. `-p` flag creates missing parent directories.
5. `rm -rf /tmp/test` — Remove files/folders. `-r` is recursive, `-f` forces deletion without prompt. **WARNING:** Use with caution.
6. `cp -r /source /dest` — Copy files/folders. `-r` is required to copy directories recursively.
7. `mv old.txt new.txt` — Move or rename files/folders.
8. `touch file.txt` — Create an empty file or update the access/modification timestamps of an existing file.

### File Inspection & Searching
9. `cat file.txt` — Concatenate and display the entire contents of a file.
10. `less /var/log/messages` — Open file in a scrollable, paging terminal window. (q to exit).
11. `more file.txt` — Older pagination viewer; scroll down only.
12. `head -n 10 file.txt` — Output the first 10 lines of a file.
13. `tail -f /var/log/nginx/error.log` — Output the last lines of a file. `-f` follows changes in real time.
14. `grep -r "error" /var/log` — Search for a text pattern inside files. `-r` searches recursively.
15. `find /etc -name "*.conf"` — Search the directory tree for files matching criteria.
16. `locate filename` — Search database index for files (faster than find, requires `updatedb`).
17. `which systemctl` — Display the absolute path of the executable command.
18. `whereis bash` — Locate the binary, source, and manual page files for a command.

### Help & Documentation
19. `man ls` — Open the manual page reference guide for a command (q to exit).
20. `command --help` — Display brief inline help documentation for the command options.

### System Verification & Hardware
21. `uptime` — Display how long the system has been running, active users, and load averages.
22. `whoami` — Output the username of the active terminal session owner.
23. `id` — Print active user and group IDs (UID, GID).
24. `hostname` — View or set the system host name.
25. `uname -a` — Print complete system and kernel details.
26. `df -h` — Display disk space utilization of all mounted filesystems in human-readable format.
27. `du -sh /var` — Display total disk usage of a specific folder in summarized human-readable format.
28. `free -m` — Display total, used, and free system RAM memory in Megabytes.
29. `date` — Display or set the system date and time.
30. `cal` — Output a calendar grid of the current month.

### Process Management
31. `top` — Launch dynamic, real-time process monitoring list.
32. `ps aux` — Output snapshot list of all active running processes.
33. `kill 1234` — Terminate process using its Process ID (PID). Defaults to SIGTERM (15).
34. `killall nginx` — Terminate all active processes matching the name.

### File Permissions & Archive
35. `chmod 755 file.sh` — Modify file access permissions.
36. `chown root:root file.txt` — Modify file owner and group assignments.
37. `ln -s target link` — Create a soft symbolic link pointing to a target file.
38. `tar -czf backup.tar.gz /data` — Create a compressed gzip archive.
39. `zip backup.zip file.txt` — Create standard zip archive.
40. `unzip backup.zip` — Extract a zip archive.

### Network Testing & Downloads
41. `wget http://site.com/file.zip` — Download files directly from the web.
42. `curl -I https://google.com` — Send HTTP requests. `-I` returns header data only.
43. `ping -c 4 8.8.8.8` — Send ICMP pings. Linux pings infinitely unless limited by `-c` count.
44. `ss -tuln` — View active listening socket connections and ports. Replaces deprecated `netstat`.
45. `ip addr` — View all network interface configurations and IP addresses.
46. `netstat` — Legacy network socket connection diagnostic tool.

### Environment & Shell Control
47. `clear` — Clear terminal screen.
48. `echo "Hello World"` — Print text to terminal output or redirect to files.
49. `history` — Output the command history log file contents.
50. `exit` — Close the active terminal session.

---
## Lab — Step by Step
> [!info] Lab Setup Needed
> A running Rocky Linux VM and access to the terminal console interface.

### Step 1: Directory Exercises
1. Log in. Open terminal.
2. Verify your current folder location:
   ```bash
   pwd
   ```
3. Create a test nested folder:
   ```bash
   mkdir -p ~/lab_temp/sub_folder
   ```
4. Change directory to the new subfolder:
   ```bash
   cd ~/lab_temp/sub_folder
   ```
5. Verify location: `pwd` should return `/home/[user]/lab_temp/sub_folder`.

### Step 2: File Creation and Viewing
1. Create an empty file:
   ```bash
   touch my_note.txt
   ```
2. Write text into the file using output redirection:
   ```bash
   echo "This is line 1 of my test note." > my_note.txt
   echo "This is line 2 of my test note." >> my_note.txt
   ```
   *(Note: `>` overwrites; `>>` appends).*
3. Verify file contents:
   ```bash
   cat my_note.txt
   ```

### Step 3: Search and Clean up
1. Search for the word "line" inside your note:
   ```bash
   grep "line" my_note.txt
   ```
2. Go back home: `cd ~`.
3. Remove the entire test directory recursively:
   ```bash
   rm -rf ~/lab_temp
   ```
4. Verify deletion: `ls ~/lab_temp` should return an error stating no such directory exists.

---
## Related Notes
- [[02-Operating-Systems/04-Linux-RHEL/L-01 Linux Introduction and Architecture|L-01 Linux Introduction and Architecture]] — Operating system base architectures.
- [[02-Operating-Systems/04-Linux-RHEL/L-03 File System Management|L-03 File System Management]] — FHS and directories layout specifications.
- [[02-Operating-Systems/04-Linux-RHEL/L-04 Text Editors and File Viewing|L-04 Text Editors and File Viewing]] — Editing files inside terminal CLI.

