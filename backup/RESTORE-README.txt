=====================================================================
 OpenMediaVault - System Backup - HOW TO RESTORE
=====================================================================

Generated: 2026-09-24T00:51:25+0300
This folder holds openmediavault-backup system backups and the restore
tool (omv-restore-helper).

---------------------------------------------------------------------
WHY YOU CANNOT RESTORE FROM WITHIN OPENMEDIAVAULT
---------------------------------------------------------------------
A restore overwrites, byte for byte, the disk OpenMediaVault boots from.
You cannot do that while the system is running and that disk is mounted -
for the same reason you cannot replace a car's engine while driving it.
The restore must run from OUTSIDE the installed system: boot a live USB /
rescue environment, then restore from there.

---------------------------------------------------------------------
WHAT YOU NEED
---------------------------------------------------------------------
  * A live/rescue Linux USB (e.g. SystemRescue on x86; see SBC note below)
  * Access to this backup folder from that environment
  * Tools: zstd, fdisk, blkid, lsblk, parted (+ fsarchiver/borg if used)

This machine, as captured in the backup:
  Hostname     : nas
  Backup date  : 2026-09-24_00-00-01
  Method       : dd
  Architecture : x86_64
  Board        : n/a
  Root disk    : /dev/sda
  Layout       : dos, root=/dev/sda1, boot=none, esp=#0

---------------------------------------------------------------------
EASIEST WAY - GUIDED AUTOMATIC RESTORE
---------------------------------------------------------------------
From the rescue environment, in this folder, run:

    sudo ./omv-restore-helper auto

It lists your backups, lets you pick the target disk, asks you to confirm,
and runs every step. Preview it first without changing anything:

    sudo ./omv-restore-helper --dry-run auto

---------------------------------------------------------------------
MANUAL RESTORE (reference) - method: dd
---------------------------------------------------------------------
1) Restore the partition table:
     sfdisk <TARGET_DISK> < backup-omv-2026-09-24_00-00-01.sfdisk                 # MBR

2) Restore the root partition (partition 1):
     zstd -d -c backup-omv-2026-09-24_00-00-01.dd.zst | dd of=<TARGET_DISK>1 bs=4M status=progress

   sync

Partition naming: SATA/USB disks use sdb1, sdb2, ...; NVMe/SD/eMMC need a
'p', e.g. nvme0n1p1, mmcblk0p1. The guided 'auto' mode handles this for you.

After a partition/file-level restore you may need to reinstall GRUB from
a chroot - see the Quick Reference in ./omv-restore-helper.

---------------------------------------------------------------------
RASPBERRY PI / SBC (ARM) NOTE
---------------------------------------------------------------------
There is no universal ARM rescue USB (each board boots differently). The
simplest recovery is to write a full-disk (ddfull) image onto a fresh
SD card / USB from ANY computer (the copy is architecture-independent):

    zstd -d -c backup-omv-2026-09-24_00-00-01.ddfull.zst | sudo dd of=<TARGET_DISK> bs=4M status=progress

u-boot is contained in that image, so the card boots with no extra steps.
=====================================================================
