---
tags: [sysadmin, linux, vim, text-editing, piping]
difficulty: Beginner
lab-required: Yes
read-time: 12 mins
---

# L-04: Text Editors and File Viewing

> [!abstract] Overview
> This note covers Linux text editing and stream processing utilities. It provides a complete reference guide for Vi/Vim modes, file viewing commands, `grep` search filters, `sed` stream editing, `awk` log processing, and I/O redirection.

---
## Concept
Think of text processing in Linux as a physical assembly line in a bottling factory:
- **Redirection (`>`)** is routing the output of a machine into a specific bottle.
- **Piping (`|`)** is connecting two conveyor belts together: the output of the first machine (e.g., list files) feeds directly into the intake of the next machine (e.g., search text) without dropping anything on the floor.
- **Vim** is a high-speed, keyboard-driven workstation where the operator doesn't waste time reaching for a mouse. Instead, they use precise key shortcuts (modes) to cut, paste, edit, and save files.
- **sed** and **awk** are automated robotic filters that scan passing text lines, replacing words (sed) or counting and reformatting database columns (awk) at lightning speeds.

*Seedha simple mein: Vim ek command-line text editor hai jo modes par chalta hai. Command line par outputs ko manage karne ke liye pipes (`|`) aur redirection (`>`) ka use hota hai. sed aur awk scripts ke zariye hum log files se custom data extract karte hain.*

---
## Technical Deep Dive

### 1. The Vi/Vim Master Reference Guide
Vim is modal: keys behave differently depending on the active mode.

```
                  +-----------------------+
                  |  Normal Mode (ESC)    | <-------+
                  +-----------------------+         |
                    /        |          \           | ESC
         i / a / o /         | :         \ v        |
                  /          v            \         |
      +-------------+  +---------------+  +-------------+
      | Insert Mode |  | Command Mode  |  | Visual Mode |
      +-------------+  +---------------+  +-------------+
```

- **Normal Mode (Default):** Used for navigation and deletion.
  - **Navigation:** `h` (left), `j` (down), `k` (up), `l` (right).
  - `gg` — Jump to first line.
  - `G` — Jump to last line.
  - `:N` — Jump to line number $N$ (e.g., `:25`).
  - **Editing Triggers:**
    - `i` — Enter Insert mode at cursor.
    - `a` — Enter Insert mode after cursor.
    - `o` — Open new line below cursor.
    - `I` — Insert at beginning of line.
    - `A` — Insert at end of line.
  - **Deletion:**
    - `x` — Delete character under cursor.
    - `dd` — Delete (cut) current line.
    - `dw` — Delete from cursor to start of next word.
    - `d$` — Delete from cursor to end of line.
  - **Copy & Paste:**
    - `yy` — Yank (copy) current line.
    - `p` — Paste copied buffer *after* cursor.
    - `P` — Paste copied buffer *before* cursor.
  - **Undo & Redo:**
    - `u` — Undo last action.
    - `Ctrl + r` — Redo undone action.
  - **Search:**
    - `/pattern` — Search forward for pattern.
    - `n` — Jump to next match.
    - `N` — Jump to previous match.
- **Command Mode (Type `:` from Normal Mode):**
  - `:%s/old/new/g` — Search and replace all instances of "old" with "new" globally.
  - `:w` — Write (save) changes.
  - `:q` — Quit editor.
  - `:wq` (or `:x` or `ZZ` in normal mode) — Save and quit.
  - `:q!` — Force quit, discarding all unsaved changes.
  - `:wqa` — Save and close all open tabs.
- **Nano:** A simpler, modeless editor. Basic shortcuts are displayed at the bottom of the screen (e.g., `Ctrl + O` to write out/save, `Ctrl + X` to exit).

### 2. File Viewing Utilities
- `cat` — Outputs entire file content to terminal. Unsuited for large files.
- `less` — Memory-efficient pagination viewer. Only loads the viewed screen segment. Supports searching (`/pattern`) and scrolling.
- `head` / `tail` — Show file start/end.
  - `tail -f` — Intercepts new inputs appended to logs (essential for live diagnostics).

### 3. Grep Filter Arguments
- `grep -i "error" file.log` — Case-insensitive search.
- `grep -r "critical" /var/log` — Recursive search through directories.
- `grep -n "failed" auth.log` — Display line numbers of matches.
- `grep -v "info" application.log` — Invert match (show lines *not* containing pattern).
- `grep -l "segfault" /var/log/*` — Output only the filenames containing the match.

### 4. Sed & Awk Basics
- **sed (Stream Editor):** Modifies text streams on the fly:
  - *Command:* `sed -i 's/port=80/port=8080/g' config.txt` (Modifies the file in-place).
- **awk:** Pattern scanning and processing language. Standard use: extracting specific columns from table logs (default delimiter is whitespace):
  - *Command:* `awk '{print $1, $4}' /var/log/httpd/access_log` (Prints IP address and date fields).

### 5. Pipes and Redirection Channels
Linux processes communicate via three standard I/O channels:
- `0` (stdin) — Keyboard input.
- `1` (stdout) — Standard output (terminal screen).
- `2` (stderr) — Error output messages.
- **Operators:**
  - `command > file.txt` — Redirect stdout, overwriting file.
  - `command >> file.txt` — Redirect stdout, appending to file.
  - `command < file.txt` — Feed file contents into command stdin.
  - `command 2> error.log` — Redirect stderr only.
  - `command > output.log 2>&1` — Redirect both stdout and stderr to the same file.
  - `command1 | command2` — Feeds stdout of command1 as stdin to command2.

---
## Lab — Step by Step
> [!info] Lab Setup Needed
> A Linux terminal console session.

### Step 1: Vim Configuration Exercise
1. Open a new file in vim:
   ```bash
   vim test_config.conf
   ```
2. Enter Insert mode by pressing `i`.
3. Type:
   ```text
   server_port=80
   server_name=localhost
   enable_ssl=false
   ```
4. Press `ESC` to return to Normal Mode.
5. Move to the first line using `gg`. Position cursor on `80`.
6. Press `a` and change it to `8080`.
7. Exit to normal mode (`ESC`). Save and quit by typing:
   ```text
   :wq
   ```

### Step 2: Stream Processing using Pipes and Grep
1. Output the active system process list:
   ```bash
   ps aux
   ```
2. Pipe the output to filter only processes running under the `root` user:
   ```bash
   ps aux | grep "^root"
   ```
3. Count the number of processes running under the root user:
   ```bash
   ps aux | grep -c "^root"
   ```

### Step 3: Extract Column Fields using Awk
1. Open the local password index file: `/etc/passwd`.
2. Extract only the system usernames (first column field separated by colon `:`):
   ```bash
   awk -F: '{print $1}' /etc/passwd
   ```
   *(Note: `-F:` tells awk to split fields using the colon character).*

---
## Commands Reference
```bash
# Search for active system log entries containing "denied" ignoring case
grep -i "denied" /var/log/secure

# Replace all instances of "127.0.0.1" with "0.0.0.0" in config.txt
sed -i 's/127.0.0.1/0.0.0.0/g' config.txt

# Extract and print user home directory paths (column 6) from /etc/passwd
awk -F: '{print $1 " -> " $6}' /etc/passwd
```

---
## Troubleshooting Scenarios

**Scenario 1:**
- **Problem:** When attempting to run a script, it fails throwing syntax errors. The script was edited on a Windows machine before being copied to the Linux server.
- **Root Cause:** Windows uses `CRLF` (Carriage Return + Line Feed, `\r\n`) line endings, while Linux uses `LF` (`\n`). The invisible `\r` carriage return characters corrupt shell scripts.
- **Fix:**
  1. Open the script in Vim: `vim run_script.sh`.
  2. Enter Command Mode and check file format:
     ```text
     :set ff?
     ```
     If it returns `fileformat=dos`, convert it to unix format:
     ```text
     :set ff=unix
     :wq
     ```
  3. Alternatively, convert using the command line utility:
     ```bash
     sed -i 's/\r//' run_script.sh
     ```

**Scenario 2:**
- **Problem:** An administrator edits the SSH configuration file `/etc/ssh/sshd_config` using Vim, but when they try to save (`:wq`), the editor throws: `E45: 'readonly' option is set (add ! to override)`.
- **Root Cause:** The file was opened without root privileges. Standard users cannot write to configuration files in `/etc`.
- **Fix:**
  1. Do not quit yet. Save the modifications using a sudo bypass command inside Vim:
     ```text
     :w !sudo tee %
     ```
  2. Press Enter to confirm, type the sudo password, then quit Vim using `:q!`.
  3. Verify that the changes were successfully saved.

---
## Common Mistakes
> [!warning] Avoid These
> **Using cat to read multi-gigabyte log files:** Running `cat /var/log/httpd/access_log` on a busy web server. This dumps millions of lines of text to the terminal buffer, consuming CPU resources, slowing down the console session, and making navigation impossible.
> **Correct approach:** Use `less` or `tail -n 100` to read log files in manageable segments.

---
## Pro Tips
> [!tip] Field Experience
> When running a long-running background command over SSH, always redirect both stdout and stderr to a file and run the command with `nohup` or inside a `screen`/`tmux` session. This prevents the process from being terminated if your SSH network connection drops.
> E.g.: `nohup ./backup_run.sh > backup.log 2>&1 &`

---
## Quick Revision Table
| # | Concept | One Line Summary |
|---|---------|-----------------|
| 1 | Vim Normal Mode | The default navigation mode; use keys `h j k l` to navigate and `i` to enter edit mode. |
| 2 | Vim Substitute | command `:%s/old/new/g` substitutes all instances of a string in a file globally. |
| 3 | Piping | Using the `|` operator to feed the standard output of one command as the input of another. |
| 4 | tail -f | Follow option that outputs new additions to a file in real-time; crucial for log monitoring. |
| 5 | sed -i | Stream editor flag that modifies target file contents in-place without generating a new file. |

---
## Interview Q&A

**Q1: How do you search for all occurrences of the word "denied" in the directory `/var/log` recursively, outputting only the filenames and line numbers?**
A: I will use the `grep` command with recursive (`-r`), line number (`-n`), and ignore-case (`-i`) options:
```bash
sudo grep -rin "denied" /var/log
```
If I only want the list of filenames without the matching content lines, I would substitute the options to include `-l`:
```bash
sudo grep -ril "denied" /var/log
```

**Q2: A web server log has lines formatted as: `[IP] - - [Date] "Request" [Status] [Bytes]`. Explain how you would use awk to calculate the total bandwidth served.**
A: 
- **Situation:** Web server logs must be parsed to sum the bytes column.
- **Task:** Write an awk script to isolate the bytes column and print the sum.
- **Action:** First, I identify the column index. The bytes transferred is usually the last column (e.g., column 10). I will use awk to loop through the lines, add the value of column 10 to a running sum variable, and print the total in the `END` block.
  ```bash
  awk '{sum+=$10} END {print "Total Bandwidth Served: " sum/1024/1024 " MB"}' /var/log/httpd/access_log
  ```
- **Result:** The script calculates the cumulative sum and prints it in Megabytes.

**Q3: Explain the difference between `command > file.txt` and `command 2>&1 > file.txt` and `command > file.txt 2>&1`.**
A: 
- `command > file.txt` redirects standard output (stdout/1) to the file, but standard error (stderr/2) still outputs to the screen.
- `command 2>&1 > file.txt` first redirects standard error (2) to the current target of standard output (which is still the terminal screen), then redirects standard output (1) to the file. Result: stderr goes to screen; stdout goes to file.
- `command > file.txt 2>&1` first redirects standard output (1) to the file, then redirects standard error (2) to the current location of standard output (which is the file). Result: both stdout and stderr are routed to the file.

---
## Related Notes
- [[02-Operating-Systems/04-Linux-RHEL/L-02 Command Line Basics|L-02 Command Line Basics]] — Basic navigation and command syntaxes.
- [[02-Operating-Systems/04-Linux-RHEL/L-03 File System Management|L-03 File System Management]] — FHS directory layouts.
- [[02-Operating-Systems/04-Linux-RHEL/L-13 Bash Scripting for Sysadmins|L-13 Bash Scripting for Sysadmins]] — Automating log processing inside scripts.

