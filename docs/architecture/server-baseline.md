# Service Baseline - sentinelai-prod-01
## Document Metadata
* **Date:** 2026-04-12
* **Author:** Chiheb Kitar
* **Ticket Reference:** INFRA-10 (WSL2 Server Baseline Audit)
* **Status:** DRAFT (Awaiting Lead Review)
## 1. System Identity
* **OS:** Ubuntu 24.04.4 LTS
* **Kernel:** 6.6.87.2-microsoft-standard-WSL2
* **Hostname:** DESKTOP-9T7GMVN
* **FQDN:** DESKTOP-9T7GMVN.localdomain
* **Virtualization Type:** Full
## 2. Hardware Profile
* **CPU:** 8
* **RAM:** 15GB
* **SWAP:** 4.0GB
* **Disk:** 1007GB
## 3. User Accounts
| Username | UID | Shell | Purpose | Interactive? |
| :--- | :--- | :--- | :--- | :--- |
| **root** | 0 | /bin/bash | System Administrator (Superuser) | Yes |
| **daemon** | 1 | /usr/sbin/nologin | Standard system service executor | No |
| **www-data** | 33 | /usr/sbin/nologin | Web server (Apache/Nginx) user | No |
| **backup** | 34 | /usr/sbin/nologin | System backup service | No |
| **nobody** | 65534 | /usr/sbin/nologin | Unprivileged user for risky tasks | No |
| **systemd-resolve** | 991 | /usr/sbin/nologin | Network Name Resolution service | No |
| **chihe** | 1000 | /bin/bash | Primary Admin (Human Account) | Yes |
## 4. Group Memberships
| Group | Members | Purpose | Risk Notes |
| :--- | :--- | :--- | :--- |
| **root** | *(none)* | Full system control | Should never have human members. |
| **adm** | syslog, **chihe** | Access to `/var/log` | High. Members can read sensitive logs/PII. |
| **sudo** | **chihe** | Execute commands as root | **Critical.** "Keys to the kingdom" access. |
| **docker** | **chihe** | Manage Docker without sudo | **Critical.** Docker access = Root escape risk. |
| **cdrom/dip** | **chihe** | Hardware/Desktop legacy | **Low.** Desktop leftovers. |
| **users** | **chihe** | Standard human users | Low. General shared resources. |
## 5. Running Services
| Service | Purpose | Keep/Review/Remove |
| :--- | :--- | :--- |
| **cron.service** | Legacy task scheduler | **Review.** We prefer systemd timers. |
| **rsyslog.service** | System logging daemon | **Keep.** Essential for log pipeline. |
| **systemd-resolved** | DNS management | **Keep.** Core networking component. |
| **systemd-timesyncd** | NTP Time Sync | **Keep.** Essential for log timestamp accuracy. |
| **unattended-upgrades**| Auto-security patching | **Review.** Can break production if unmonitored. |
| **wsl-pro.service** | WSL2 helper | **Keep.** Required for the VM environment. |
| **user@1000.service** | User session manager | **Keep.** Manages your current login session. |
## 6. Network Configuration
(interfaces, IPs, listening ports)
* **Interfaces:** lo,eth0
* **IPs:** 127.0.0.1/10,172.19.90.180/20
* **Listening Ports:** 53,40121
## 7. DNS Architecture
* The process queries the local systemd-resolved stub (127.0.0.53), which then forwards to the Windows Host resolver (10.255.255.254) for final internet resolution.
## 8. Disk Layout
| Mount Point | Filesystem | Size | 
| :--- | :--- | :--- |
| **/** | ext4 | 1.0T | 
| **/mnt/c** | 9p | 953G | 
| **/run** | tmpfs | 7.8G | 
| **/init** | rootfs | 7.8G |
## 9. Key Findings & Risks
* OS Discrepancy: System is running Ubuntu 24.04 LTS (update), contradicting the initial requirement of 22.04.
* Privilege Creep: User chihe holds unnecessary memberships in desktop-legacy groups (cdrom, dip, plugdev).
* SSH Gap: No SSH daemon (sshd) is installed or running, creating a dependency for remote access tasks.
* Docker Risk: Membership in the docker group presents a direct path to root privilege escalation.
* Unknown Surface: UDP port 40121 is listening on 0.0.0.0 and requires investigation.
## 10. Pre-Change State Confirmation
* I, Chiheb, confirm that this document accurately captures the "Ground Truth" of the sentinelai-prod-01 environment as of 2026-04-12.
* This baseline serves as the official reference point before any configuration changes or hardening tasks are executed in Sprint 1.
