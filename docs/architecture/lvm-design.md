# LVM Volume Architecture Design — sentinelai-prod-01

## Document Metadata
* **Date:** 2026-05-04
* **Author:** Chiheb Kitar
* **Status:** ACCEPTED / DESIGN
* **Ticket Reference:** INFRA-24/25

## 1. Design Overview
This architecture uses Logical Volume Management (LVM) to decouple physical 
storage from the filesystem hierarchy. By inserting an abstraction layer, 
we enable online volume resizing, point-in-time snapshots for backups, and 
critical isolation of log-heavy directories to prevent system-wide outages.

## 2. Physical Infrastructure Assumption
The design assumes a two-disk production server:
* **Disk 1 (/dev/sda - 100GB):** System Disk. 
    * `/dev/sda1` (1GB): `/boot` partition (Non-LVM).
    * `/dev/sda2` (4GB): `swap` partition (Non-LVM).
    * `/dev/sda3` (80GB): LVM Physical Volume for `vg_system`.
    * (Remaining ~15GB): Unallocated raw space.
* **Disk 2 (/dev/sdb - 500GB):** Data Disk. 
    * `/dev/sdb1` (500GB): LVM Physical Volume for `vg_data`.

## 3. LVM Layer Design

### 3.1 Physical Volumes (PV)
| Device    | Size  | Role                           |
| :-------- | :---- | :----------------------------- |
| /dev/sda3 | 80GB  | System PV (OS, Logs, Tmp)      |
| /dev/sdb1 | 500GB | Data PV (App, Models, Backups) |

### 3.2 Volume Groups (VG)
| VG Name       | PVs Included | Total Size | Purpose                    |
| :------------ | :----------- | :--------- | :------------------------- |
| **vg_system** | /dev/sda3    | 80GB       | System Stability Domain    |
| **vg_data**   | /dev/sdb1    | 500GB      | Application/Data Domain    |

### 3.3 Logical Volumes (LV)
| LV Name       | VG        | Size  | Mount Point      | FS   |
| :------------ | :-------- | :---- | :--------------- | :--- |
| **lv_root**   | vg_system | 20GB  | /                | ext4 |
| **lv_var**    | vg_system | 30GB  | /var             | ext4 |
| **lv_tmp**    | vg_system | 5GB   | /tmp             | ext4 |
| *(free)*      | vg_system | 25GB  | -                | -    |
| **lv_app**    | vg_data   | 100GB | /opt/sentinelai  | ext4 |
| **lv_backup** | vg_data   | 200GB | /backup          | ext4 |
| *(free)*      | vg_data   | 200GB | -                | -    |

## 4. Disk Layout Diagram
PHYSICAL DISKS         VOLUME GROUPS         LOGICAL VOLUMES           MOUNTS
[ /dev/sda3 ] -------> [ vg_system ] ------> [ lv_root ] ----------> /
                                     ------> [ lv_var  ] ----------> /var
                                     ------> [ lv_tmp  ] ----------> /tmp
                                     ------> [ 25GB Free Space ]

[ /dev/sdb1 ] -------> [  vg_data  ] ------> [ lv_app    ] --------> /opt/sentinelai
                                     ------> [ lv_backup ] --------> /backup
                                     ------> [ 200GB Free Space ]

## 5. Thin Provisioning Decision
Volumes in vg_system use Thick Provisioning for guaranteed allocation.
Thin Provisioning is used only for lv_app in vg_data to handle
unpredictable ML model growth.

Monitoring: Thin-pools must be expanded if they reach 90% utilization.

## 6. Snapshot Strategy
lv_root: Snapshot before apt upgrade. Retained 24h.

lv_app: Snapshot daily for backup consistency. Retained 48h.

lv_var: Daily snapshot during log rotation/backup. Retained 24h.

## 7. Sizing Justification
lv_root (20GB): Sufficient for Ubuntu baseline and utilities.

lv_var (30GB): Prevents log growth from impacting OS stability.

lv_tmp (5GB): Hard cap on world-writable scratch space.

lv_app (100GB): Accommodates large ML model binaries.

## 8. WSL2 Implementation Note (Bare-Metal)
On production hardware, implementation begins with these three commands:

# 1. Initialize Physical Volumes
pvcreate /dev/sda3 /dev/sdb1

# 2. Create the Volume Groups
vgcreate vg_system /dev/sda3
vgcreate vg_data /dev/sdb1

# 3. Create initial Logical Volumes
lvcreate -L 20G -n lv_root vg_system
lvcreate -L 100G -n lv_app vg_data
