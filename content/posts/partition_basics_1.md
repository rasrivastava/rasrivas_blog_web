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

## Making a partition permanent

#### Listing the partition on the system

```
# lsblk 
NAME                MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sr0                  11:0    1  7.9G  0 rom  /dvd
nvme0n1             259:0    0  100G  0 disk 
├─nvme0n1p1         259:1    0    1G  0 part /boot
└─nvme0n1p2         259:2    0   99G  0 part 
  ├─rhel_192-root   253:0    0   50G  0 lvm  /
  ├─rhel_192-swap   253:1    0  4.2G  0 lvm  [SWAP]
  └─rhel_192-home   253:2    0 44.8G  0 lvm  /home
nvme0n2             259:3    0    5G  0 disk 
└─mydemovg-mydemolv 253:3    0    8G  0 lvm  /root/mydemodirectory
nvme0n3             259:4    0    5G  0 disk 
├─mydemovg-mydemolv 253:3    0    8G  0 lvm  /root/mydemodirectory
└─mydemovg-demolv2  253:4    0   12M  0 lvm
#
```

```
# df -hT
Filesystem                    Type      Size  Used Avail Use% Mounted on
devtmpfs                      devtmpfs  1.6G     0  1.6G   0% /dev
tmpfs                         tmpfs     1.6G  4.0K  1.6G   1% /dev/shm
tmpfs                         tmpfs     1.6G  9.7M  1.6G   1% /run
tmpfs                         tmpfs     1.6G     0  1.6G   0% /sys/fs/cgroup
/dev/mapper/rhel_192-root     xfs        50G  9.9G   41G  20% /
/dev/mapper/rhel_192-home     xfs        45G  352M   45G   1% /home
/dev/nvme0n1p1                xfs      1014M  211M  804M  21% /boot
/dev/sr0                      iso9660   7.9G  7.9G     0 100% /dvd
tmpfs                         tmpfs     316M  1.2M  315M   1% /run/user/42
tmpfs                         tmpfs     316M  4.0K  316M   1% /run/user/0
/dev/mapper/mydemovg-mydemolv ext4      7.9G   32M  7.4G   1% /root/mydemodirectory
```

### Listing the mount command

```
# mount | grep /dev/mapper/mydemovg-mydemolv
/dev/mapper/mydemovg-mydemolv on /root/mydemodirectory type ext4 (rw,relatime,seclabel)
```

### Listing the current /etc/fstab file

```
# cat /etc/fstab 

#
# /etc/fstab
# Created by anaconda on Sat May 30 07:17:47 2020
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
/dev/mapper/rhel_192-root /                       xfs     defaults        0 0
UUID=8b6ecb4a-2165-4726-a5f6-dfad4827aae3 /boot                   xfs     defaults        0 0
/dev/mapper/rhel_192-home /home                   xfs     defaults        0 0
/dev/mapper/rhel_192-swap swap                    swap    defaults        0 0
```

### Making entry in the fstab

- Here we have six fields
  - device_name
  - mount_point
  - format
  - options
  - diskcheck
  - disksync


- adding below line in the /etc/fstab
  ```
  /dev/mydemovg/mydemolv /root/mydemodirectory ext4 defaults 0 0
  ```
  
  ```
  # cat /etc/fstab 

  #
  # /etc/fstab
  # Created by anaconda on Sat May 30 07:17:47 2020  
  #
  # Accessible filesystems, by reference, are maintained under '/dev/disk/'.
  # See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
  #
  # After editing this file, run 'systemctl daemon-reload' to update systemd
  # units generated from this file. 
  #
  /dev/mapper/rhel_192-root /                       xfs     defaults        0 0
  UUID=8b6ecb4a-2165-4726-a5f6-dfad4827aae3 /boot                   xfs     defaults        0 0
  /dev/mapper/rhel_192-home /home                   xfs     defaults        0 0
  /dev/mapper/rhel_192-swap swap                    swap    defaults        0 0

  /dev/mydemovg/mydemolv /root/mydemodirectory ext4 defaults 0 0
  ```
  
- We need to verify that everything we have written in the file is correct, we can run below command to test it
  `# mount -a`


### we also have a command blkid to list all the partitions

- this shows us the **UUID** for the partiotion which is always unique
- sometime it happens the partition name changes OR if storage is coming from Network (**NAS**) but the **UUID** will never change

```
# blkid 
/dev/nvme0n1: PTUUID="78f0aa00" PTTYPE="dos"
/dev/nvme0n1p1: UUID="8b6ecb4a-2165-4726-a5f6-dfad4827aae3" TYPE="xfs" PARTUUID="78f0aa00-01"
/dev/nvme0n1p2: UUID="0o6PU9-2Igb-5GEl-hCYJ-ZkGb-ZngN-EIxCN2" TYPE="LVM2_member" PARTUUID="78f0aa00-02"
/dev/nvme0n2: UUID="G7hzUX-cqXS-A9v1-9VaS-k4Yl-ZEUd-Klj9RL" TYPE="LVM2_member"
/dev/nvme0n3: UUID="d4XYBS-3HMS-MURj-0EvE-NceI-24ns-lzoPIh" TYPE="LVM2_member"
/dev/sr0: UUID="2020-04-04-08-21-15-00" LABEL="RHEL-8-2-0-BaseOS-x86_64" TYPE="iso9660" PTUUID="47055c33" PTTYPE="dos"
/dev/mapper/rhel_192-root: UUID="81345e98-f211-4811-bf37-8cfd9f9b0330" TYPE="xfs"
/dev/mapper/rhel_192-swap: UUID="137d9a23-b533-4684-b489-7d4185478180" TYPE="swap"
/dev/mapper/rhel_192-home: UUID="ae05ea7b-ecd1-4062-8f18-0a84ea2f7350" TYPE="xfs"
/dev/mapper/mydemovg-mydemolv: UUID="fe2bf001-7323-4f9c-a00e-525f12ed6098" TYPE="ext4"
```

- making entry in fstab using UUID
  ```
  UUID=fe2bf001-7323-4f9c-a00e-525f12ed6098 /root/mydemodirectory ext4 defaults 0 0
  ```
  
  ```
  # cat /etc/fstab

  #
  # /etc/fstab
  # Created by anaconda on Sat May 30 07:17:47 2020
  #
  # Accessible filesystems, by reference, are maintained under '/dev/disk/'.
  # See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
  #
  # After editing this file, run 'systemctl daemon-reload' to update systemd
  # units generated from this file.
  #
  /dev/mapper/rhel_192-root /                       xfs     defaults        0 0
  UUID=8b6ecb4a-2165-4726-a5f6-dfad4827aae3 /boot                   xfs     defaults        0 0
  /dev/mapper/rhel_192-home /home                   xfs     defaults        0 0
  /dev/mapper/rhel_192-swap swap                    swap    defaults        0 0

  UUID=fe2bf001-7323-4f9c-a00e-525f12ed6098 /root/mydemodirectory ext4 defaults 0 0
  #
  ```
  
  ```
  # mount -a
  ```
