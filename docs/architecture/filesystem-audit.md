# Filesystem Audit — sentinelai-prod-01

## Document Metadata
* **Date:** 2026-04-20
* **Author:** Chiheb Kitar
* **Ticket Reference:** INFRA-24 (Filesystem Audit)
* **Status:** APPROVED — INFRA-24

## 1. Audit Overview
* **Tools Used:** `findmnt`, `lsblk`, `df -i`, `cat`
* **Scope:** Full system mount table, block device inventory, and security option analysis. This audit serves as the baseline for the LVM design in INFRA-25.

## 2. Block Device Inventory
| Device | UUID | FSType | Size | Mount Point | Notes |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **sda** | N/A | ext4 | 384M | [None] | Internal WSL2 virtual disk |
| **sdb** | N/A | ext4 | 2G | [None] | Internal WSL2 virtual disk |
| **sdc** | f665a148... | swap | 4G | [SWAP] | System swap partition |
| **sdd** | f2c3e4e9... | ext4 | 1T | / | Primary system storage (VHDX) |

## 3. Mount Point Analysis
| Mount Point | Source | FSType | Key Options | Security Assessment |
| :--- | :--- | :--- | :--- | :--- |
| **/** | /dev/sdd | ext4 | rw, relatime | **Insecure:** Missing `nodev` |
| **/mnt/c** | C:\ | 9p | rw, noatime | **Operational:** Windows Host bridge |
| **/run** | none | tmpfs | rw, nosuid, nodev | **Hardened:** Standard kernel defaults |

**Note on /tmp:** `/tmp` is not independently mounted; it is currently a directory on the root filesystem inheriting all root options. This lack of isolation is a primary security gap.

## 4. /etc/fstab Analysis
The file `/etc/fstab` contains the header `# UNCONFIGURED FSTAB FOR BASE SYSTEM`.
* **Finding:** In WSL2, the filesystem table is empty by default because the init process manages mounts dynamically.
* **Architectural Impact:** Traditional persistent hardening via `/etc/fstab` is not viable for system-managed mounts. We must utilize Systemd `.mount` units or `wsl.conf` settings for persistence.

## 5. Inode Usage
| Filesystem | Inodes Total | Inodes Used | Use% | Status |
| :--- | :--- | :--- | :--- | :--- |
| **/dev/sdd** | 67,108,864 | ~129,000 | 1% | **Healthy** |
| **/run** | 2,028,598 | 652 | <1% | **Healthy** |
| **/run/user/1000** | 405,970 | 25 | <1% | **Healthy** |
| **9p (drvfs)** | 10,000,000 | -999,001 | N/A | **Reporting Artifact** |

## 6. Security Gap Analysis
| Mount Point | Missing Options | Risk Level | Fixable in WSL2? |
| :--- | :--- | :--- | :--- |
| **/** | `nodev` | Medium | No (WSL2 Managed) |
| **/tmp** | `noexec`, `nosuid`, `nodev` | **High** | Yes (Via systemd unit) |
| **/run/shm** | `noexec` | Medium | Yes |

* **Risk Note:** The lack of `noexec` on `/tmp` is a critical gap. It allows the execution of arbitrary scripts or binaries in a world-writable directory, facilitating local privilege escalation (LPE) attacks.

## 7. WSL2 Constraints
* **Root Mount Persistence:** The `/` mount is established by the WSL2 init process. Adding `nodev` to `/` is currently restricted by the environment's architecture.
* **FSTAB Management:** Because `/etc/fstab` is unmanaged, any persistent mount configuration for directories like `/tmp` must be implemented as a Systemd `.mount` unit. This method ensures custom options are applied after the WSL2 init sequence.
* **Inode Inconsistency:** Inode reporting on `9p` (Windows bridge) is unreliable due to the translation layer between Linux and NTFS.

## 8. Recommendations
1.  **INFRA-27:** Hardening of `/tmp` via a Systemd `.mount` unit.
2.  **INFRA-25:** Design LVM architecture leveraging the current 1TB virtual disk capacity.
3.  **Ongoing:** Documentation of all non-fstab persistent mounts in the central baseline.
