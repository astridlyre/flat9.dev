---
layout: post
title: 'Hard Disk Layout'
date: 2021-03-19
categories: linux disk device hardware
---

In this post, I'm going to give a broad overview of storage devices on Linux. A _disk_ is
a storage device or "physical container" for data. For historical reasons, storage devices
are known as [disks](https://en.wikipedia.org/wiki/Hard_disk_drive) due to the original
_hard disks_ being actual rotating devices, kind of like a CD-ROM. Contemporary storage
devices are often _solid state_ ([SSD](https://en.wikipedia.org/wiki/Solid-state_drive)),
which means they use a non-volatile flash-type system, similar to a USB drive.

Before a disk can be used to store data, it must be [partitioned](https://en.wikipedia.org/wiki/Disk_partitioning).
A _partition_ is a logical subset of the physical disk, designed as a way to compartmentalize
information. Each disk must have at least one partition. Information about the partitions
is stored in the [_partition table_](https://en.wikipedia.org/wiki/Partition_table).
Once a disk has been partitioned, each partition must be formatted with a _filesystem_.
A [filesystem](https://en.wikipedia.org/wiki/File_system) describes the way the information
is stored on the disk, including how directories are organized, their relationships,
and where the data is for each file.

Partitions cannot span multiple disks, but a _Logical Volume Manager_ ([LVM](<https://en.wikipedia.org/wiki/Logical_Volume_Manager_(Linux)>))
can combine multiple partitions to form a single _logical volume_. A logical volume
abstracts the limitations of physical devices and permits the creation of "pools" of disk
space, which is more flexible than traditional partitions. This is useful when a user
needs to add more space to a partition without having to migrate their data to a larger
device.

## Mounting

To access the data on a storage device, the disk must be [_mounted_](https://en.wikipedia.org/wiki/Mount_%28computing%29) at some point in the
host filesystem. The is called a _mount point_. The contents of the storage device will be
made available at the mount point as files within a directory. Before mounting a device,
the mount point must already have been created, and if it already has files in it those
files will be _masked_ (ie. hidden but not deleted) when the storage device is mounted.

On Linux, the traditional mount point is `/mnt`. This is where external devices would be
mounted, for example: `/mnt/cdrom` or `/mnt/floppy`. However, modern systems prefer to
automatically mount devices under the `/media` directory, such as when a USB drive is
plugged in and a udev [_hotplug_](https://en.wikipedia.org/wiki/Hot_swapping) event is triggered. Still, if a user is manually mounting
a device, the traditional `/mnt` directory is a good place to use.

## Keeping Things Separated

Some directories in Linux may benefit from being on their own partition or disk:

- `/boot` ought to be on its own partition, to ensure the system is still bootable even if
  the root filesystem crashes.
- `/home` may be on a separate partition to make it easier to reinstall the system without
  the risk of accidentally changing user data.
- `/var` can be located on a different disk in order to keep data related to a database or
  web server separate, and in order to prevent the disk from growing so large it prevents
  the system from operating. This also enables more disk space to be added without the
  whole system being migrated.
- `/` may be kept on a solid state drive for speed reasons, while larger directories such
  as `/home` or `/var` can be kept on slower hard disks for cost savings.

## The Boot Partition

The `/boot` partition contains files used by the bootloader (GRUB2 or GRUB legacy). This
partition is mounted at `/boot` and the bootloader files are typically in `/boot/grub`.
Having a separate boot partition is not technically required, since GRUB can mount the
root partition and load the files from a separate `/boot` directory, but it is desired for
safety reasons, or if using a different root filesystem that the bootloader does not
support (or if the root filesystem is to be encrypted).

The boot partition is the **first partition** on the disk. This is due to the way the
original IBM PC-BIOS addressed disks using _cylinders_, _heads_ and _sectors_ (CHS). This
system had a maximum of 1024 cylinders, 256 heads and 63 sectors, resulting in a maximum
size of 528 MB. Anything larger would not be accessible on legacy systems, unless using
_logical block addressing_ ([LBA](https://en.wikipedia.org/wiki/Logical_block_addressing)).
A safe size for a modern boot partition is **300 MB**.

## The EFI System Partition (ESP)

Modern machines based on the _Unified Extensible Firmware Interface_ (UEFI) use the EFI
system partition to store boot loaders and kernel images. This partition is formatted with
a FAT-based filesystem. On a [GUID](https://en.wikipedia.org/wiki/Universally_unique_identifier)
partition table it would have a GUID of `C12A7328-F81F-11D2-BA4B-00A0C93EC93B` and on an
MBR partition scheme it would have a partition ID of `0xEF`. The ESP is mounted under `/boot/efi`.

## Other Partitions

While most "normal" users on a Linux system have their own `/home/<user>` directory, the
_superuser_ or _root_ account has its home in `/root`. Other accounts, such as system
services, may not have a home directory.
