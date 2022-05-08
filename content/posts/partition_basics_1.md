---
title: "Disk Partitioning in Linux"
date: 2022-05-08T17:05:08+05:30
draft: false
tags: ["linux", "filesysystem", "partition"]
categories: ["linux", "operating system"]
---

Disk Partitioning is the process of dividing a disk into one or more logical areas, often known as partitions, on which the user can work separately.

It is one step of disk formatting. If a partition is created, the disk will store the information about the location and size of partitions in the partition table.

With the partition table, each partition can appear to the operating system as a logical disk, and users can read and write data on those disks.

The main advantage of disk partitioning is that each partition can be managed separately.

- Create a partition
- Format a partition  
- Mount a partition
- Unmount a partition

### List the current partition using the `lsblk` command
```
# lsblk 
NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
xvda    202:0    0  10G  0 disk 
├─xvda1 202:1    0   1M  0 part 
└─xvda2 202:2    0  10G  0 part /
xvdb    202:16   0   1G  0 disk 
```

### Creating a new partition
```
# fdisk /dev/xvdb
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table
Building a new DOS disklabel with disk identifier 0x56f4b7d2.

Command (m for help): n
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p 
Partition number (1-4, default 1): 1
First sector (2048-2097151, default 2048): 
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-2097151, default 2097151): 
Using default value 2097151
Partition 1 of type Linux and of size 1023 MiB is set

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
```

### Verifying the new partition
```
# lsblk 
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
xvda    202:0    0   10G  0 disk 
├─xvda1 202:1    0    1M  0 part 
└─xvda2 202:2    0   10G  0 part /
xvdb    202:16   0    1G  0 disk 
└─xvdb1 202:17   0 1023M  0 part 
```

### Formating the new partition
```
# mkfs.ext4 /dev/xvdb1 
mke2fs 1.42.9 (28-Dec-2013)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
65536 inodes, 261888 blocks
13094 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=268435456
8 block groups
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done
```

### Mounting the new partition
```
# mount /dev/xvdb1 /mnt/rasrivas/
```

### verifying the new mount point
```
# lsblk 
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
xvda    202:0    0   10G  0 disk 
├─xvda1 202:1    0    1M  0 part 
└─xvda2 202:2    0   10G  0 part /
xvdb    202:16   0    1G  0 disk 
└─xvdb1 202:17   0 1023M  0 part /mnt/rasrivas
```

### Unmounting the partition
```
# umount /mnt/rasrivas/
```
