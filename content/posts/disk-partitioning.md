---
title: "Disk Partitioning"
date: 2021-06-18T16:30:28+05:30
draft: false
tags: ["linux", "storage", "basics", "TIL"]
---

## What's Partition?

*Disk Partitioning* allows you to divide hard drive into multiple logical sections. There are several reasons why we do it:
- Multi-boot OS
- Easier backup
- Different filesystems to be installed on different partitions
- Clear separation between operating system and user files

### Partitioning Schemes
`MBR` & `GPT` are two different schemes available to do partitions.

#### MBR
`MBR` supports maximum four primary partitions. On Linux, one can create more partitions having three primary & one extended partition which may contain multiple logical partitions.

The available size for block addresses and related information is limited to 32 bits. For hard disks with 512‑byte sectors, the MBR partition table entries allow a maximum size of `2 TiB` (2^32 × 512‑bytes) or `2.20 TB` (2.20 × 10¹² bytes).

In today's world, we have physical disks greater than 2TiB in size, so MBR scheme having a maximum disk & partition size of 2TiB is really big problem, which is solved by GPT.

#### GPT
`GPT` addresses many of the limitations introduced by MBR & is Forming a part of the Unified Extensible Firmware Interface.

`GPT` uses 64 bits for logical block addresses, allowing a maximum disk size of 2^64 sectors. For disks with 512‑byte sectors, the maximum size is `8 ZiB` (2^64 × 512‑bytes) or `9.44 ZB` (9.44 × 10^21 bytes).

### Creating Partitions
We can create/manage partitions using *`parted`*, *`fdisk`*, *`gdisk`*. I'll be using `parted` to play with partitions on USB drive.

#### Create GPT Partition
*Select disk to create partition*
```
# list devices
[dipak@localhost ~]$ lsblk 
NAME                                            MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINT
sdb                                               8:16   1  14.9G  0 disk  
nvme0n1                                         259:0    0 953.9G  0 disk  
├─nvme0n1p1                                     259:1    0   600M  0 part  /boot/efi
├─nvme0n1p2                                     259:2    0     1G  0 part  /boot
└─nvme0n1p3                                     259:3    0 952.3G  0 part  
  ├─fedora_localhost--live-root                 253:0    0    70G  0 lvm   
  │ └─luks-493fecbb-ff12-452a-914d-33f64d65e942 253:2    0    70G  0 crypt /
  ├─fedora_localhost--live-swap                 253:1    0  19.5G  0 lvm   
  │ └─luks-db8bdc6b-1e43-4074-89ca-1b412c35bc04 253:3    0  19.5G  0 crypt [SWAP]
  └─fedora_localhost--live-home                 253:4    0 862.8G  0 lvm   
    └─luks-c863fba7-ff0b-4896-9fcc-2e6144fccda4 253:5    0 862.8G  0 crypt /home

# check current partition table
[dipak@localhost ~]$ sudo parted /dev/sdb print
Error: /dev/sdb: unrecognised disk label
Model: Innostor Innostor (scsi)                                           
Disk /dev/sdb: 15.9GB
Sector size (logical/physical): 512B/512B
Partition Table: unknown
Disk Flags: 
```
As you can see from above output, we don't have any partition table on `sdb` device.

*Create gpt partition of around 10 GiB.*
```
# create new disklabel `gpt`.
[dipak@localhost ~]$ sudo parted /dev/sdb
GNU Parted 3.3
Using /dev/sdb
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) mklabel gpt

# verify partition table
(parted) print                                                            
Model: Innostor Innostor (scsi)
Disk /dev/sdb: 15.9GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags: 

Number  Start  End  Size  File system  Name  Flags

# create first `ext4` partition starting from 1MiB to 10GiB.
(parted) mkpart                                                
Partition name?  []? p1                                                   
File system type?  [ext2]? ext4                                           
Start? 1MiB                                                               
End? 10GiB                                                                

# check created partitions
(parted) print                                                            
Model: Innostor Innostor (scsi)
Disk /dev/sdb: 15.9GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags: 

Number  Start   End     Size    File system  Name  Flags
 1      1049kB  10.7GB  10.7GB  ext4         p1
```
*Create filesystem & Mount it*
```
# check if we have any filesystem created on partition
[dipak@localhost ~]$ lsblk /dev/sdb
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sdb      8:16   1 14.9G  0 disk 
└─sdb1   8:17   1   10G  0 part

# we don't have any filesystem on partition. Let's create ext4 filesystem on partition `/dev/sdb1`.
[dipak@localhost ~]$ sudo mkfs.ext4 /dev/sdb1
mke2fs 1.45.5 (07-Jan-2020)
Creating filesystem with 2621184 4k blocks and 655360 inodes
Filesystem UUID: 3b4f2bbd-2c3d-4a2a-a317-4859d22d5894
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done 

# verify `ext4` filesystem 
[dipak@localhost ~]$ lsblk --fs /dev/sdb
NAME   FSTYPE FSVER LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINT
sdb                                                                           
└─sdb1 ext4   1.0         3b4f2bbd-2c3d-4a2a-a317-4859d22d5894  

# mount partition & create some data in it.
sudo mkdir /mnt/data
sudo mount /dev/sdb1 /mnt/data

# check if partition is mounted
[dipak@localhost ~]$ lsblk /dev/sdb
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sdb      8:16   1 14.9G  0 disk 
└─sdb1   8:17   1   10G  0 part /mnt/data

# we can permanently mount filesystems by updating`/etc/fstab` entries. 
UUID=3b4f2bbd-2c3d-4a2a-a317-4859d22d5894  /mnt/data  ext4  defaults  0  2
```

#### Delete GPT Partition

We have out friend `wipefs` which helps us to wipe signature from a device
```
# check device
[dipak@localhost ~]$ lsblk /dev/sdb
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sdb      8:16   1 14.9G  0 disk 
└─sdb1   8:17   1   10G  0 part /mnt/data

# unmount filesystem
[dipak@localhost ~]$ sudo umount /dev/sdb1 

# check filesystem is unmounted
[dipak@localhost ~]$ lsblk /dev/sdb
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sdb      8:16   1 14.9G  0 disk 
└─sdb1   8:17   1   10G  0 part 

# wipe signature from device
[dipak@localhost ~]$ sudo wipefs -a /dev/sdb
/dev/sdb: 8 bytes were erased at offset 0x00000200 (gpt): 45 46 49 20 50 41 52 54
/dev/sdb: 8 bytes were erased at offset 0x3b5fffe00 (gpt): 45 46 49 20 50 41 52 54
/dev/sdb: 2 bytes were erased at offset 0x000001fe (PMBR): 55 aa
/dev/sdb: calling ioctl to re-read partition table: Success

# check device after wiping signature 
[dipak@localhost ~]$ lsblk /dev/sdb 
NAME MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sdb    8:16   1 14.9G  0 disk 

# check partition table
[dipak@localhost ~]$ sudo parted /dev/sdb print
Error: /dev/sdb: unrecognised disk label
Model: Innostor Innostor (scsi)                                           
Disk /dev/sdb: 15.9GB
Sector size (logical/physical): 512B/512B
Partition Table: unknown
Disk Flags: 
```

#### Create MBR Partition
*Create MBR primary partition of around 10 GiB.*
```
# create dos label
[dipak@localhost ~]$ sudo parted /dev/sdb mklabel msdos
Information: You may need to update /etc/fstab.

# create primary partition of 10GiB
[dipak@localhost ~]$ sudo parted /dev/sdb mkpart primary xfs 1MiB 10GiB
Information: You may need to update /etc/fstab.

# verify created partition
[dipak@localhost ~]$ sudo parted /dev/sdb print
Model: Innostor Innostor (scsi)
Disk /dev/sdb: 15.9GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags: 

Number  Start   End     Size    Type     File system  Flags
 1      1049kB  10.7GB  10.7GB  primary  ext4
```

#### Delete MBR Partition
*Let's not use `wipefs -a` this time.*
```
# check current partition table
[dipak@localhost ~]$ sudo parted /dev/sdb
GNU Parted 3.3
Using /dev/sdb
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) print                                                            
Model: Innostor Innostor (scsi)
Disk /dev/sdb: 15.9GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags: 

Number  Start   End     Size    Type     File system  Flags
 1      1049kB  10.7GB  10.7GB  primary  ext4

# remove partition 1
(parted) rm 1                                                             

# verify removed partition
(parted) print                                                            
Model: Innostor Innostor (scsi)
Disk /dev/sdb: 15.9GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags: 

Number  Start  End  Size  Type  File system  Flags
```
