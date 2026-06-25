---
tags: [sysadmin, linux, package-management, yum, dnf, rpm]
difficulty: Intermediate
lab-required: Yes
read-time: 12 mins
---

# L-10: Software Management — YUM/DNF

> [!abstract] Overview
> This note covers Red Hat package management, detailing RPM database querying, YUM/DNF dependency resolution, repository configuration, local repo creation, and package extraction techniques.

---
## Concept
Think of software management in Linux like ordering groceries for a recipe:
- An **RPM File** is a pre-packaged can of ingredients. It lists its contents, but if the recipe demands another can of sauce that you don't have, the manager stops and says, "Error: missing dependency. Go buy sauce yourself."
- **DNF (or YUM)** is an automated grocery delivery app. You say, "I want to cook NGINX." DNF checks the database, finds NGINX, checks the list of needed ingredients (dependencies), automatically adds the sauce, spices, and pots to your shopping cart, downloads them from the warehouse (Repositories), and cooks the meal for you.
- **`/etc/yum.repos.d/`** is your list of approved grocery stores (Repositories).

*Seedha simple mein: RPM low-level package controller hai jo individual files install karta hai lekin automatic dependencies download nahi karta. DNF modern high-level tool hai jo repositories se auto-dependency resolution ke sath packages download aur install karta hai.*

---
## Technical Deep Dive

### 1. RPM Package Format & Database
- **RPM (Red Hat Package Manager):** A low-level package manager. It installs, uninstalls, and queries `.rpm` software files.
- **RPM Database:** Stored in `/var/lib/rpm`. It tracks every file, dependency, and configuration installed on the system via RPM.
- **RPM Query Commands:**
  - `rpm -qa` — Query all installed packages.
  - `rpm -qi nginx` — Display detailed package info (version, release, description).
  - `rpm -ql nginx` — List all files installed by the package.
  - `rpm -qf /etc/nginx/nginx.conf` — Find which package owns a specific file.

### 2. YUM vs. DNF
- **YUM (Yellowdog Updater, Modified):** The default package manager in RHEL 5, 6, and 7. Based on Python 2.
- **DNF (Dandified YUM):** Replaced YUM in RHEL 8 and 9. 
  - **Advantages:** Runs on Python 3 and C, has a faster dependency resolution algorithm, consumes less memory, and supports structured API extensions. In RHEL 8+, `yum` is simply a symbolic link pointing to `dnf`.

### 3. Repository Configuration (`/etc/yum.repos.d/`)
Repositories are defined in files ending in `.repo` inside `/etc/yum.repos.d/`.
- **Repo File Structure:**
  ```ini
  [baseos]
  name=Rocky Linux $releasever - BaseOS
  baseurl=http://dl.rockylinux.org/pub/rocky/$releasever/BaseOS/$basearch/os/
  gpgcheck=1
  enabled=1
  gpgkey=file:///etc/pki/moc-gpg/RPM-GPG-KEY-rockyofficial
  ```
  - **`[baseos]`** — Unique repository ID.
  - **`baseurl`** — Path to the package database (can be HTTP, FTP, or local file system path `file://`).
  - **`gpgcheck`** — 1 enables GPG cryptographic signature checking to verify package authenticity.
  - **`enabled`** — 1 enables the repo; 0 disables it.

### 4. DNF Command Reference

```bash
dnf install nginx          # Install package and dependencies
dnf remove nginx           # Remove package and clean up unused dependencies
dnf update nginx           # Update package to latest version
dnf search webserver       # Search package names and summaries for keyword
dnf info nginx             # Display package details and dependencies

dnf group list             # List available package group profiles (e.g., "Development Tools")
dnf group install "Development Tools" # Install all packages in group

dnf repolist               # List all enabled repositories
dnf clean all              # Clear local metadata cache database
dnf makecache              # Download and build fresh repository metadata cache
```

### 5. Extracting RPM Files Without Installing
If you need to retrieve a single configuration file from an RPM package without installing the entire package, use `rpm2cpio`:
```bash
rpm2cpio package.rpm | cpio -idmv
```
- This extracts the internal archive directories into your current working directory.

---
## Lab — Step by Step
> [!info] Lab Setup Needed
> A running Rocky Linux VM with internet access and root privileges.

### Step 1: Query Existing Package Database
1. Log in as root. Find if Apache (`httpd`) is installed:
   ```bash
   rpm -q httpd
   ```
2. If it returns "package httpd is not installed," find which package provides the command `/usr/sbin/httpd`:
   ```bash
   dnf provides /usr/sbin/httpd
   ```
   **Verify:** Output should display `httpd-[version]`.

### Step 2: Configure a Custom Local Repository
To simulate an offline datacenter environment:
1. Create a directory to host packages:
   ```bash
   mkdir -p /opt/local_repo
   ```
2. Download a package (e.g., `wget`) to the directory without installing it:
   ```bash
   dnf download --destdir=/opt/local_repo wget
   ```
3. Initialize the directory as a repository database (requires `createrepo_c` package installed):
   ```bash
   createrepo_c /opt/local_repo
   ```
4. Create the repository configuration file:
   ```bash
   vim /etc/yum.repos.d/local.repo
   ```
5. Add configuration:
   ```ini
   [local-backup-repo]
   name=Local Offline Backup Repository
   baseurl=file:///opt/local_repo/
   enabled=1
   gpgcheck=0
   ```
6. Save and close (`:wq`).

### Step 3: Test Local Repo Installation
1. Disable all standard internet repositories temporarily:
   ```bash
   dnf config-manager --disable "*"
   dnf config-manager --enable "local-backup-repo"
   ```
2. Verify active repo list:
   ```bash
   dnf repolist
   ```
   **Verify:** Only `local-backup-repo` should be listed.
3. Install the package from your local repository:
   ```bash
   dnf install wget -y
   ```
4. Re-enable standard repositories when finished:
   ```bash
   dnf config-manager --enable "baseos,appstream"
   ```

---
## Related Notes
- [[02-Operating-Systems/04-Linux-RHEL/L-02 Command Line Basics|L-02 Command Line Basics]] — Basic execution syntaxes.
- [[02-Operating-Systems/04-Linux-RHEL/L-03 File System Management|L-03 File System Management]] — Navigating `/etc` and `/opt`.
- [[02-Operating-Systems/04-Linux-RHEL/L-08 Services and Systemd|L-08 Services and Systemd]] — Initializing services after package installation.

