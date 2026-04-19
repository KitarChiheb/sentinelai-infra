# Server Baseline - sentinelai-prod-01

## Document Metadata
* **Date:** 2026-04-12
* **Author:** Chiheb Kitar
* **Ticket Reference:** INFRA-18 (created) | INFRA-20 (hostname update) | INFRA-21 (user model update) | INFRA-22 (sudo policy update) | INFRA-23 (SSH Hardening update)
* **Status:** APPROVED — Updated INFRA-23

## 1. System Identity
* **OS:** Ubuntu 24.04.4 LTS (Noble Numbat)
* **Kernel:** 6.6.87.2-microsoft-standard-WSL2
* **Hostname:** sentinelai-prod-01
* **FQDN:** sentinelai-prod-01.localdomain
* **Virtualization Type:** WSL2 (Microsoft Hyper-V lightweight VM)

## 2. Hardware Profile
* **CPU:** Intel Core i7-1185G7 @ 3.00GHz — 4 physical cores, 8 logical CPUs (2 threads/core)
* **RAM:** 15GB
* **SWAP:** 4.0GB
* **Disk:** 1007GB (Virtual Disk Image)

## 3. User Accounts
*Full user list contains 30 entries. Table below documents accounts of operational significance.*
*As of INFRA-21, production user model implemented. chihe account retained temporarily. Full transition deferred to Sprint 2.*
| Username | UID | Shell | Purpose | Interactive? |
| :--- | :--- | :--- | :--- | :--- |
| **root** | 0 | /bin/bash | System Administrator | Yes (**High Risk**: To be hardened in INFRA-23) |
| **daemon** | 1 | /usr/sbin/nologin | Standard system service | No |
| **www-data** | 33 | /usr/sbin/nologin | Web server (Apache/Nginx) | No |
| **backup** | 34 | /usr/sbin/nologin | System backup service | No |
| **nobody** | 65534 | /usr/sbin/nologin | Unprivileged tasks | No |
| **systemd-resolve** | 991 | /usr/sbin/nologin | Network Name Resolution service | No |
| **chihe** | 1000 | /bin/bash | Primary Admin (cleaned group memberships) | Yes |
| **chiheb** | 1001 | /bin/bash | Primary Sysadmin | Yes |
| **sentinelai-app** | 999 | /usr/sbin/nologin | SentinelAI Application Service Account | No |

## 4. Group Memberships
| Group | Members | Purpose | Risk Notes |
| :--- | :--- | :--- | :--- |
| **root** | *(none)* | Full system control | No human members allowed. |
| **adm** | syslog, **chihe**, **chiheb** | Log File Access | High. Access to sensitive PII logs. |
| **sudo** | **chihe** | Root Privilege Access | **Critical.** Full system bypass. |
| **docker** | **chihe** | Docker Management | **Critical.** Privilege escalation risk. |
| **cdrom/dip** | *(none)* | Hardware/Legacy | **Low.** Desktop legacy; Removed in INFRA-21. |
| **plugdev** | *(none)* | External Device Access | **Low.** Desktop legacy; Removed in INFRA-21. |
| **users** | **chihe** | General User Group | **Low.** Shared resources. |
| **sysadmin** | **chiheb** | Primary Sysadmin Group | **Critical.** Full sudo — INFRA-22 implemented. |
| **appteam** | *(none)* | Application Operators Group | **Low.** Scoped sudo for service operations — INFRA-22 implemented. sentinelai-app primary group. Primary group members do not appear in /etc/group member list. |
| **readonly** | *(none)* | Monitoring and Debugging Group | **Low.** No sudo privileges by design. Access via file permissions only. |
## 5. Running Services
| Service | Purpose | Keep/Review/Remove |
| :--- | :--- | :--- |
| **cron.service** | Legacy task scheduler | **Review.** Prefer systemd timers. |
| **rsyslog.service** | System logging daemon | **Keep.** Essential for logs. |
| **systemd-resolved** | DNS management | **Keep.** Core networking. |
| **systemd-timesyncd** | NTP Time Sync | **Keep.** Required for log accuracy. |
| **unattended-upgrades**| Auto-security patching | **Review.** Monitor to prevent downtime. |
| **wsl-pro.service** | WSL2 helper | **Keep.** Required for environment. |
| **user@1000.service** | User session manager | **Keep.** Manages admin session. |
| **ssh.service** | Remote access manager | **Keep.** ssh.socket disabled intentionally. |

## 6. Network Configuration

### 6.1 Interfaces & Addressing
| Interface | IP Address | Prefix | Scope | Notes |
| :--- | :--- | :--- | :--- | :--- |
| **lo** | 127.0.0.1 | /8 | Host | Standard IPv4 Loopback. |
| **lo** | 10.255.255.254 | /32 | Host | **WSL2 Internal**: DNS Listener. |
| **eth0** | 172.19.90.180 | /20 | Global | Primary vNIC; Untrusted scope. |

### 6.2 Listening Ports
| Protocol | Port | Service | Bound To | Risk Level |
| :--- | :--- | :--- | :--- | :--- |
| **UDP/TCP** | 53 | systemd-resolved | 127.0.0.53/54 | Low (Local) |
| **UDP/TCP** | 53 | DNS Resolver | 10.255.255.254 | Medium (WSL Bridge) |
| **UDP** | 40121 | Unknown | 0.0.0.0 | **Review Required (External)** |
| **TCP** | 2222 | ssh.service | 0.0.0.0 / [::] | Low (Hardened) |
## 7. DNS Architecture
* The process queries the local systemd-resolved stub (127.0.0.53), which then forwards to the Windows Host resolver (10.255.255.254) for final internet resolution.

## 8. Disk Layout
| Mount Point | Filesystem | Size | Use% | Purpose |
| :--- | :--- | :--- | :--- | :--- |
| **/** | ext4 | 1.0T | 1% | **Main OS & Project Data** |
| **/mnt/c** | 9p | 953G | 27% | Windows Host Bridge |
| **/run** | tmpfs | 7.8G | 1% | Runtime volatile data |
| **/init** | rootfs | 7.8G | 1% | WSL2 Boot Init |

## 9. Key Findings & Risks
* **OS Discrepancy:** Running **Ubuntu 24.04 LTS**, contradicting the 22.04 requirement.
* **Privilege Creep:** Resolved in INFRA-21: Legacy group memberships (cdrom, dip, plugdev) removed from chihe.
* **SSH Gap:** Resolved in INFRA-23.
* **Docker Risk:** `docker` group membership provides a direct path to root privilege escalation.
* **Unknown Surface:** UDP port `40121` is listening globally and requires forensic identification.

## 10. Pre-Change State Confirmation
* I, **Chiheb**, confirm that this document accurately captures the "Ground Truth" of the sentinelai-prod-01 environment as of 2026-04-12.
* This baseline serves as the official reference point before any configuration changes or hardening tasks are executed in Sprint 1.
