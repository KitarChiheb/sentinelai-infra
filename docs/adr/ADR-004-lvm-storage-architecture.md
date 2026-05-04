# ADR-004: LVM Storage Architecture

## Status
ACCEPTED

## Context
Standard static partitioning is fragile. If a directory like `/var` fills 
completely, the system fails. We need a way to resize volumes online and 
isolate system stability from application data growth.

## Decision
We will implement a dual-Volume Group (VG) LVM strategy:
* **vg_system** for OS stability (Thick Provisioned).
* **vg_data** for application scale (Thin Provisioned for `lv_app`).
This separation ensures application data corruption or exhaustion does not 
prevent the core OS from booting or logging.

## Consequences
* **Positive:** Online volume expansion without downtime.
* **Positive:** Point-in-time snapshots for consistent live backups.
* **Negative:** ~1% performance overhead and increased `initramfs` complexity.

## Alternatives Considered
1. **Flat Partitions:** Rejected. Requires offline resizing and risks outages.
2. **ZFS/BTRFS:** Rejected. High RAM overhead and unnecessary complexity.
3. **Single Volume Group:** Rejected. Creates "Failure Coupling" where a 
data disk failure could prevent the system disk from booting due to shared 
LVM metadata risks.
