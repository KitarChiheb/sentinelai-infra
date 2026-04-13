# Server Baseline - sentinelai-prod-01

## Document Metadata
* **Date:** 2026-04-12
* **Author:** Chiheb Kitar
* **Ticket Reference:** INFRA-18 (WSL2 Server Baseline Audit)
* **Status:** DRAFT (Awaiting Lead Review)

## 1. System Identity
* **OS:** Ubuntu 24.04.4 LTS (Noble Numbat)
* **Kernel:** 6.6.87.2-microsoft-standard-WSL2
* **Hostname:** DESKTOP-9T7GMVN
* **FQDN:** DESKTOP-9T7GMVN.localdomain
* **Virtualization Type:** WSL2 (Microsoft Hyper-V lightweight VM)

## 2. Hardware Profile
* **CPU:** Intel Core i7-1185G7 @ 3.00GHz — 4 physical cores, 8 logical CPUs (2 threads/core)
* **RAM:** 15GB
* **SWAP:** 4.0GB
* **Disk:** 1007GB (Virtual Disk Image)

## 3. User Accounts
*Full user list contains 28 entries. Table below documents accounts of operational significance.*
| Username | UID | Shell | Purpose | Interactive? |
| :--- | :--- | :--- | :--- | :--- |
| **root** | 0 | /bin/bash | System Administrator | Yes (**High Risk**: To be hardened in INFRA-23) |
| **daemon** | 1 | /usr/sbin/nologin | Standard system service | No |
| **www-data** | 33 | /usr/sbin/nologin | Web server (Apache/Nginx) | No |
| **backup** | 34 | /usr/sbin/nologin | System backup service | No |
| **nobody** | 65534 | /usr/sbin/nologin | Unprivileged tasks | No |
| **systemd-resolve** | 991 | /usr/sbin/nologin | Network Name Resolution service | No |
| **chihe** | 1000 | /bin/bash | Primary Admin (Human) | Yes |

## 4. Group Memberships
| Group | Members | Purpose | Risk Notes |
| :--- | :--- | :--- | :--- |
| **root** | *(none)* | Full system control | No human members allowed. |
| **adm** | syslog, **chihe** | Log File Access | High. Access to sensitive PII logs. |
| **sudo** | **chihe** | Root Privilege Access | **Critical.** Full system bypass. |
| **docker** | **chihe** | Docker Management | **Critical.** Privilege escalation risk. |
| **cdrom/dip** | **chihe** | Hardware/Legacy | **Low.** Desktop legacy; remove in INFRA-21. |
| **plugdev** | **chihe** | External Device Access | **Low.** Desktop legacy; remove in INFRA-21. |
| **users** | **chihe** | General User Group | Low. Shared resources. |

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

## 7. DNS Architecture
* The process queries the local **systemd-resolved** stub (`127.0.0.53`), which then forwards to the **Windows Host** resolver (`10.255.255.254`) for final internet resolution.

## 8. Disk Layout
| Mount Point | Filesystem | Size | Use% | Purpose |
| :--- | :--- | :--- | :--- | :--- |
| **/** | ext4 | 1.0T | 1% | **Main OS & Project Data** |
| **/mnt/c** | 9p | 953G | 27% | Windows Host Bridge |
| **/run** | tmpfs | 7.8G | 1% | Runtime volatile data |
| **/init** | rootfs | 7.8G | 1% | WSL2 Boot Init |

## 9. Key Findings & Risks
* **OS Discrepancy:** Running **Ubuntu 24.04 LTS**, contradicting the 22.04 requirement.
* **Privilege Creep:** User `chihe` holds unnecessary memberships in desktop-legacy groups (e.g., `cdrom`, `dip`, `plugdev`), documented in Section 4.
* **SSH Gap:** No `sshd` is running; remote access dependency for Sprint 1 is currently blocked.
* **Docker Risk:** `docker` group membership provides a direct path to root privilege escalation.
* **Unknown Surface:** UDP port `40121` is listening globally and requires forensic identification.

## 10. Pre-Change State Confirmation
* I, **Chiheb**, confirm that this document accurately captures the "Ground Truth" of the sentinelai-prod-01 environment as of 2026-04-12.
* This baseline serves as the official reference point before any configuration changes or hardening tasks are executed in Sprint 1.
