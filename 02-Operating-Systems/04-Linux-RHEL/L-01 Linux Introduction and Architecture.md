---
tags: [sysadmin, linux, architecture, introduction]
difficulty: Beginner
lab-required: Yes
read-time: 10 mins
---

# L-01: Linux Introduction and Architecture

> [!abstract] Overview
> This note introduces the Linux operating system, detailing its history, open-source licensing model, distribution families, kernel-space/user-space architecture, and terminal interfaces.

---
## Concept
Think of the operating system as a complete corporate hierarchy inside a major factory:
- **Hardware** is the physical building, machine tools, and engines (CPU, RAM, Disks).
- **The Kernel** is the general manager. They work in a secure private office (Kernel Space) and are the only one allowed to talk directly to physical machine controllers, deciding who gets to use what tool when.
- **The Shell** is the manager's secretary. They stand at the office door (User Space) taking orders from workers (Applications) in specific languages (bash, zsh), translating those orders, and passing them to the general manager.
- **Applications** are the factory workers who execute tasks (editing documents, serving web pages).

*Seedha simple mein: Linux ek open-source operating system hai jo Linus Torvalds ne 1991 mein create kiya tha. Iska main core 'Kernel' hota hai jo hardware aur applications ke beech mediator ka kaam karta hai, aur 'Shell' user command input lene ka platform hai.*

---
## Technical Deep Dive

### 1. Open Source & The GPL License
Linux is distributed under the **GPL (GNU General Public License)**.
- **Copyleft Concept:** Anyone can use, modify, and redistribute the source code for free, but they must release their modifications under the same GPL license. They cannot make modifications proprietary.
- **Community-Driven:** Thousands of developers globally review and patch source code, securing fast bug fixes and stability.

### 2. History of Linux
- **1969:** Bell Labs creates **Unix**, a multi-user, multi-tasking operating system. However, it was proprietary and expensive.
- **1983:** Richard Stallman starts the **GNU Project** to build a completely free Unix-like OS, creating tools (compiler, shell, libraries) but lacking a functional kernel.
- **1991:** Linus Torvalds, a Finnish student, writes a free kernel called **Linux** to run on his Intel 386 computer. He merges it with the GNU project tools, forming the **GNU/Linux** operating system.

### 3. Linux Distribution Families (Distros)
A distribution bundles the Linux kernel with GNU utilities, package managers, and graphical shells.

- **Red Hat Family (Enterprise Standard):**
  - **RHEL (Red Hat Enterprise Linux):** The commercial enterprise standard. Requires paid support subscriptions.
  - **Fedora:** Cutting-edge upstream testing ground for new Red Hat features.
  - **CentOS Stream / Rocky Linux / AlmaLinux:** Free, community-driven RHEL downstream rebuilds.
- **Debian Family (Stability & Web Hosting):**
  - **Debian:** Rock-solid stability, slow release cycles.
  - **Ubuntu:** Canonical-supported user-friendly desktop and server distro built on Debian.
- **Arch Family (Customization):**
  - Rolling release distro focusing on manual construction and lightweight footprints.

### 4. Linux Architecture Stack

```
+-------------------------------------------------------------+
| Applications (Web Server, Browser, Text Editor)             |  (User Space)
+-------------------------------------------------------------+
| Shell (Bash, Zsh) & System Libraries (glibc)                |
+-------------------------------------------------------------+
| SYSTEM CALLS (API layer bridging User and Kernel)           |
+-------------------------------------------------------------+
| Kernel (Process Scheduler, Memory Manager, Device Drivers)   |  (Kernel Space)
+-------------------------------------------------------------+
| Hardware (CPU, RAM, Storage, NIC)                          |  (Physical Layer)
+-------------------------------------------------------------+
```
- **Kernel Space:** High-privilege memory zone protected from user interference.
- **User Space:** Low-privilege memory zone where standard applications and shells execute, preventing app crashes from taking down the OS.

### 5. Core Kernel Functions
- **Process Management:** The scheduler decides which CPU cores run which process queues.
- **Memory Management:** Maps virtual memory blocks to physical RAM chips.
- **Device Management:** Employs device drivers to write/read data to/from hardware.
- **System Calls:** The API boundary allowing user-space apps to request kernel services (e.g., `open()`, `read()`, `write()`).

### 6. Shell Classifications
The shell reads command inputs, evaluates them, and invokes execution.
- **sh (Bourne Shell):** The legacy Unix shell standard.
- **bash (Bourne Again Shell):** Default shell on most Linux distributions. Supports tab completion, command history, and scripting.
- **zsh (Z Shell):** Built on bash with advanced auto-suggestions and themes. Default shell on macOS and modern Kali Linux.
- **fish (Friendly Interactive Shell):** Modern, highly interactive shell focusing on visual highlights and out-of-the-box configurations.

### 7. Console vs. Terminal vs. Shell
- **Console:** The physical monitor and keyboard connected directly to the server motherboard.
- **Terminal:** The software interface wrapper that displays text inputs/outputs (e.g., GNOME Terminal, PuTTY, Command Prompt).
- **Shell:** The command interpreter application executing *inside* the terminal window.

---
## Lab — Step by Step
> [!info] Lab Setup Needed
> A workstation running VirtualBox or Hyper-V, and a CentOS Stream or Rocky Linux ISO file.

### Step 1: Create a VM in VirtualBox
1. Open VirtualBox. Click **New**.
2. Name: `RockyLinux_Lab_01`. Type: **Linux**. Version: **Red Hat (64-bit)**.
3. RAM: Allocate `2048 MB` (minimum).
4. Hard Disk: Select "Create a virtual hard disk now." Size: `20 GB`. Click Create.
5. Go to VM Settings -> Storage -> select the empty Optical Drive -> click CD icon -> browse and attach the Rocky Linux ISO.

### Step 2: OS Installation
1. Start the VM. Select **Install Rocky Linux**.
2. Select English language. Click Continue.
3. In the Installation Summary dashboard:
   - **Software Selection:** Choose **Server with GUI** (or **Minimal Install** for Server Core experience).
   - **Installation Destination:** Select the 20GB virtual disk. Click Done.
   - **Network & Host Name:** Turn on the Ethernet adapter connection. Set hostname to `rocky-srv01.local`.
   - **Root Password:** Set a secure password.
   - **User Creation:** Create a standard admin user (e.g., `sysadmin`) and check **Make this user administrator**.
4. Click **Begin Installation**. Once complete, reboot the VM.

### Step 3: Verify Boot and Identify Shell
1. Log in. Open a Terminal window.
2. Verify you are running the bash shell:
   ```bash
   echo $SHELL
   ```
   It should return `/bin/bash`.
3. Check the active kernel version:
   ```bash
   uname -r
   ```

---
## Commands Reference
Query system details and interface scopes:

```bash
# Print system information (OS name, network node hostname, kernel release details)
uname -a

# View operating system release identification details
cat /etc/os-release

# Print active system uptime and load average statistics
uptime
```

---
## Troubleshooting Scenarios

**Scenario 1:**
- **Problem:** When attempting to run a custom administrative application, the process crashes, throwing: "Kernel Panic - not syncing: Attempted to kill init! exitcode=0x00000007."
- **Root Cause:** A critical kernel space crash. The `init` system process (PID 1), which parent-controls all user-space processes, was killed or corrupted.
- **Fix:**
  1. Force reboot the system.
  2. At the GRUB boot menu, select the recovery kernel (rescue target).
  3. If missing, boot using Rocky Linux installation media and select **Troubleshooting > Rescue a Rocky Linux system**.
  4. Mount the local system partition, run `chroot /mnt/sysimage`, and verify `/sbin/init` binary integrity or check disk errors using `fsck`.

**Scenario 2:**
- **Problem:** When the server boots up, it drops directly into a black text CLI interface. The administrator cannot load the desktop GUI environment.
- **Root Cause:** The default systemd target is set to command-line runlevel (`multi-user.target`) instead of graphical runlevel (`graphical.target`).
- **Fix:**
  1. Log in as root.
  2. Verify the current default target:
     ```bash
     systemctl get-default
     ```
  3. Change the default target to graphical:
     ```bash
     systemctl set-default graphical.target
     ```
  4. Start the GUI manually: `systemctl isolate graphical.target` or reboot.

---
## Common Mistakes
> [!warning] Avoid These
> **Running commands as root by default:** Logging in as the `root` superuser for daily system administration. A simple typo in a delete command (e.g., running `rm -rf /` instead of `rm -rf ./`) as root will wipe the entire operating system instantly without warning prompts.
> **Correct approach:** Log in as a standard user and use the `sudo` command prefix to elevate privileges only when executing specific administrative commands.

---
## Pro Tips
> [!tip] Field Experience
> When choosing Linux distributions for enterprise servers, align with either RHEL (Rocky/Alma) or Ubuntu LTS. These releases provide stable long-term support (typically 5-10 years of security updates), preventing application breaks caused by major package changes on rolling-release distros.

---
## Quick Revision Table
| # | Concept | One Line Summary |
|---|---------|-----------------|
| 1 | Kernel | The core operating system manager handling hardware, memory, CPU scheduling, and drivers. |
| 2 | Shell | The command interpreter that reads user inputs, translates them, and interacts with the kernel. |
| 3 | GPL | Copyleft license guaranteeing code freedom; modifications must remain open source. |
| 4 | RHEL | Red Hat Enterprise Linux; the commercial industry standard distribution family. |
| 5 | System Call | The API bridge allowing user-space applications to request services from the kernel. |

---
## Interview Q&A

**Q1: What is the difference between User Space and Kernel Space in Linux architecture?**
A: **Kernel Space** is the protected memory zone where the core operating system kernel executes. It has direct access to all physical hardware, CPU registers, and memory controllers. **User Space** is the restricted memory zone where standard user applications, databases, and shells run. User space applications cannot talk directly to hardware; they must request kernel services using system calls. This separation ensures stability: if an application in user space crashes, it cannot corrupt kernel memory or crash the hardware.

**Q2: A client wants to deploy a free, stable version of Red Hat for their database hosts. What distribution do you recommend and why?**
A: 
- **Situation:** A database server requires a free, stable enterprise-grade Linux distribution compatible with RHEL.
- **Task:** Select and justify the correct RHEL downstream rebuild.
- **Action:** I will recommend **Rocky Linux** or **AlmaLinux**. Both are community-driven, $1:1$ binary-compatible downstreams of Red Hat Enterprise Linux. They compile source code directly from RHEL git repositories, providing identical enterprise stability and update packages without RHEL licensing costs.
- **Result:** The database runs on a RHEL-like operating system, ensuring enterprise support, driver compatibility, and stability.

**Q3: Describe the exact difference between a Terminal and a Shell.**
A: A **Terminal** is the display and input wrapper program (such as GNOME Terminal or PuTTY) that renders the text window and interfaces with the desktop GUI. It captures keystrokes and displays output. A **Shell** is the command interpreter engine (such as bash or zsh) running *inside* that terminal window. The shell reads the text input received from the terminal, evaluates variables, executes scripts, runs commands, and returns the resulting output to the terminal for display.

---
## Related Notes
- [[02-Operating-Systems/04-Linux-RHEL/L-02 Command Line Basics|L-02 Command Line Basics]] — Basic CLI execution commands.
- [[02-Operating-Systems/04-Linux-RHEL/L-03 File System Management|L-03 File System Management]] — Navigating directories.
- [[02-Operating-Systems/04-Linux-RHEL/L-08 Services and Systemd|L-08 Services and Systemd]] — Systemd process runlevels and targets.

