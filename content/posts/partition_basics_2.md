---
title: "Logical Volume Manager (LVM) in Linux"
date: 2022-05-08T18:00:41+05:30
draft: false
---

LVM is a tool for logical volume management which includes allocating disks, striping, mirroring and resizing logical volumes.

With LVM, a hard drive or set of hard drives is allocated to one or more physical volumes. LVM physical volumes can be placed on other block devices which might span two or more disks.

The physical volumes are combined into logical volumes, with the exception of the /boot partition. The /boot partition cannot be on a logical volume group because the boot loader cannot read it. If the root (/) partition is on a logical volume, create a separate /boot partition which is not a part of a volume group.

Since a physical volume cannot span over multiple drives, to span over more than one drive, create one or more physical volumes per drive.

The volume groups can be divided into logical volumes, which are assigned mount points, such as /home and / and file system types, such as ext2 or ext3. When "partitions" reach their full capacity, free space from the volume group can be added to the logical volume to increase the size of the partition. When a new hard drive is added to the system, it can be added to the volume group, and partitions that are logical volumes can be increased in size.

On the other hand, if a system is partitioned with the ext3 file system, the hard drive is divided into partitions of defined sizes. If a partition becomes full, it is not easy to expand the size of the partition. Even if the partition is moved to another hard drive, the original hard drive space has to be reallocated as a different partition or not used.


### Adding two new hard disk to the system
  - **nvme0n2** (2 G)
  - **nvme0n2** (3 G)

- Verifying it if its detechted by the system
  - ```
    # lsblk 
    NAME              MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
    sr0                11:0    1  7.9G  0 rom  /dvd
    nvme0n1           259:0    0  100G  0 disk 
    ├─nvme0n1p1       259:1    0    1G  0 part /boot
    └─nvme0n1p2       259:2    0   99G  0 part 
      ├─rhel_192-root 253:0    0   50G  0 lvm  /
      ├─rhel_192-swap 253:1    0  4.2G  0 lvm  [SWAP]
      └─rhel_192-home 253:2    0 44.8G  0 lvm  /home
    nvme0n2           259:3    0    2G  0 disk 
    nvme0n3           259:4    0    3G  0 disk #
    ```
  
### Formatting these partiontions:
  - creating partition for **/dev/nvme0n2**
  ```
  # fdisk /dev/nvme0n2
  ```

 - Verifying it, we will see a partition **nvme0n2p1** for the **nvme0n2** hard disk
   ```
   # lsblk 
   NAME              MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
   sr0                11:0    1  7.9G  0 rom  /dvd
   nvme0n1           259:0    0  100G  0 disk 
   ├─nvme0n1p1       259:1    0    1G  0 part /boot
   └─nvme0n1p2       259:2    0   99G  0 part 
     ├─rhel_192-root 253:0    0   50G  0 lvm  /
     ├─rhel_192-swap 253:1    0  4.2G  0 lvm  [SWAP]
     └─rhel_192-home 253:2    0 44.8G  0 lvm  /home
   nvme0n2           259:3    0    2G  0 disk 
     └─nvme0n2p1       259:6    0    1K  0 part 
   nvme0n3           259:4    0    3G  0 disk 
   #
   ```
   
   - creating partition for **/dev/nvme0n3**
     ```
     # fdisk /dev/nvme0n3
     ```
   - verifying it, we will see a new partition **nvme0n3p1** for the **nvme0n3** hard disk
     ```
     # lsblk 
     NAME              MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
     sr0                11:0    1  7.9G  0 rom  /dvd
     nvme0n1           259:0    0  100G  0 disk 
     ├─nvme0n1p1       259:1    0    1G  0 part /boot
     └─nvme0n1p2       259:2    0   99G  0 part 
       ├─rhel_192-root 253:0    0   50G  0 lvm  /
       ├─rhel_192-swap 253:1    0  4.2G  0 lvm  [SWAP]
       └─rhel_192-home 253:2    0 44.8G  0 lvm  /home
     nvme0n2           259:3    0    2G  0 disk 
       └─nvme0n2p1       259:6    0    1K  0 part 
     nvme0n3           259:4    0    3G  0 disk 
       └─nvme0n3p1       259:7    0    1K  0 part 
     #

### We have created below partitions are demoing LVM concept

- We have created two partition
  - **nvme0n2** at ***/dev/nvme0n2*** location
  - **nvme0n3** at ***/dev/nvme0n2*** location
- ***nvme0n2*** and ***nvme0n3*** are called as **partions**


### Creating Physical Volume for ***/dev/nvme0n2*** and ***/dev/nvme0n2***

- Creating ***pvcreate*** for **/dev/nvme0n2**
  ```
  # pvcreate /dev/nvme0n2
  Physical volume "/dev/nvme0n2" successfully created.
  ```

- Listing the **/dev/nvme0n2** using ***pvdisplay***
  ```
  # pvdisplay /dev/nvme0n2
  "/dev/nvme0n2" is a new physical volume of "2.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/nvme0n2
  VG Name               
  PV Size               2.00 GiB
  Allocatable           NO  <<= no, its free
  PE Size               0   
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               TsbM4f-gbcC-vloN-K1uJ-hbMP-uhFq-TVDxc2
  # 
  ```

- Creating ***pvcreate*** for **/dev/nvme0n3**
  ```
  # pvcreate /dev/nvme0n3
  Physical volume "/dev/nvme0n3" successfully created.
  ```

- Listing the **/dev/nvme0n3** using ***pvdisplay***
  ```
  # pvdisplay /dev/nvme0n3 
  "/dev/nvme0n3" is a new physical volume of "3.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/nvme0n3
  VG Name               
  PV Size               3.00 GiB
  Allocatable           NO  <<= no, its free
  PE Size               0   
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               e3lw33-BKMf-5bXU-Uhz8-vgIP-BDfo-tdkjf9
  #
  ```

### Creating Volume Group for **/dev/nvme0n2** and **/dev/nvme0n3**

- Listing all the ***vgdisplay***
  ```
  # vgdisplay 
  --- Volume group ---
  VG Name               rhel_192
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  4
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                3
  Open LV               3
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <99.00 GiB
  PE Size               4.00 MiB
  Total PE              25343
  Alloc PE / Size       25343 / <99.00 GiB
  Free  PE / Size       0 / 0   
  VG UUID               IWXt27-SjOw-vKwc-0dHJ-GzgJ-TDLx-LcYE64
  #
  ```

- Creating ***vgcreate*** for **/dev/nvme0n2** and **/dev/nvme0n3** physical volume

  ```
  # vgcreate rasrivas_vg /dev/nvme0n2 /dev/nvme0n3
  Volume group "rasrivas_vg" successfully created
  #
  ```

- Listing ***vgdisplay*** for ***rasrivas_vg***
  ```
  # vgdisplay rasrivas_vg
  --- Volume group ---
  VG Name               rasrivas_vg
  System ID             
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               4.99 GiB
  PE Size               4.00 MiB
  Total PE              1278
  Alloc PE / Size       0 / 0   
  Free  PE / Size       1278 / 4.99 GiB
  VG UUID               G9mXOZ-fEwq-SqQ5-sGXN-Kfac-n7Zb-Tp9euu
  #
  ```
- Listing ***pvdisplay*** for **/dev/nvme0n3**
  - we will see **rasrivas_vg** volume group is attached to **/dev/nvme0n3** physical volume
    ```
    # pvdisplay /dev/nvme0n3 
    --- Physical volume ---
    PV Name               /dev/nvme0n3
    VG Name               rasrivas_vg <<= this pv is attached this
    PV Size               3.00 GiB / not usable 4.00 MiB
    Allocatable           yes 
    PE Size               4.00 MiB
    Total PE              767
    Free PE               767
    Allocated PE          0
    PV UUID               e3lw33-BKMf-5bXU-Uhz8-vgIP-BDfo-tdkjf9
    #
    ```
  - we will see **rasrivas_vg** volume group is attached to **/dev/nvme0n2** physical volume
    ```
    # pvdisplay /dev/nvme0n2 
    --- Physical volume ---
    PV Name               /dev/nvme0n2
    VG Name               rasrivas_vg  <<= this pv is attached this
    PV Size               2.00 GiB / not usable 4.00 MiB
    Allocatable           yes 
    PE Size               4.00 MiB
    Total PE              511
    Free PE               511
    Allocated PE          0
    PV UUID               TsbM4f-gbcC-vloN-K1uJ-hbMP-uhFq-TVDxc2
    #
    ```

### Creating Logical Volume for the "rasrivas_vg" Volume Group 

- Creating logical volume
  ```
  # lvcreate --size 4G --name rasrivas_lv rasrivas_vg
  Logical volume "rasrivas_lv" created.
  ``

- Listing the logical volume
  ```
  # lvdisplay  rasrivas_vg/rasrivas_lv
  --- Logical volume ---
  LV Path                /dev/rasrivas_vg/rasrivas_lv <<= partition name
  LV Name                rasrivas_lv
  VG Name                rasrivas_vg
  LV UUID                SPjPL2-hTsh-ytUD-GOwP-iwCz-YT1A-HtJjr8
  LV Write Access        read/write
  LV Creation host, time 192.168.53.143, 2020-05-31 22:16:52 +0530
  LV Status              available
  # open                 0
  LV Size                4.00 GiB
  Current LE             1024
  Segments               2
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:3
  #
  ```

### Formatting the /dev/rasrivas_vg/rasrivas_lv partition (logical volume)

- Formatting the partition
  ```
  # mkfs.ext4 /dev/rasrivas_vg/rasrivas_lv
  mke2fs 1.45.4 (23-Sep-2019)
  Creating filesystem with 1048576 4k blocks and 262144 inodes
  Filesystem UUID: 4573229e-20f4-4bf1-822d-2958d5eeaeb5
  Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736

  Allocating group tables: done                            
  Writing inode tables: done                            
  Creating journal (16384 blocks): done
  Writing superblocks and filesystem accounting information: done 
  ```
  
### Mounting the /dev/rasrivas_vg/rasrivas_lv partition (logical volume)
- Creating a directroy where to be mounted
  `# mkdir rasrivas_mp`

- Mounting the partition
  ```
  # mount /dev/rasrivas_vg/rasrivas_lv rasrivas_mp
  #
  ```

- Verifying the mounted partition
  ```
  # lsblk 
  NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
  sr0                        11:0    1  7.9G  0 rom  
  nvme0n1                   259:0    0  100G  0 disk 
  ├─nvme0n1p1               259:1    0    1G  0 part /boot
  └─nvme0n1p2               259:2    0   99G  0 part 
    ├─rhel_192-root         253:0    0   50G  0 lvm  /
    ├─rhel_192-swap         253:1    0  4.2G  0 lvm  [SWAP]
    └─rhel_192-home         253:2    0 44.8G  0 lvm  /home
  nvme0n2                   259:3    0    2G  0 disk 
  └─rasrivas_vg-rasrivas_lv 253:3    0    4G  0 lvm  /root/rasrivas_mp
  nvme0n3                   259:4    0    3G  0 disk 
  └─rasrivas_vg-rasrivas_lv 253:3    0    4G  0 lvm  /root/rasrivas_mp
  # 
  ```
  
  ```
  # df -h
  Filesystem                           Size  Used Avail Use% Mounted on
  devtmpfs                             2.1G     0  2.1G   0% /dev
  tmpfs                                2.1G     0  2.1G   0% /dev/shm
  tmpfs                                2.1G  9.7M  2.1G   1% /run
  tmpfs                                2.1G     0  2.1G   0% /sys/fs/cgroup
  /dev/mapper/rhel_192-root             50G  4.6G   46G  10% /
  /dev/mapper/rhel_192-home             45G  352M   45G   1% /home 
  /dev/nvme0n1p1                      1014M  211M  804M  21% /boot
  tmpfs                                420M  1.2M  419M   1% /run/user/42
  tmpfs                                420M  4.0K  420M   1% /run/user/0
  /dev/mapper/rasrivas_vg-rasrivas_lv  3.9G   16M  3.7G   1% /root/rasrivas_mp
  # 
  ```
  
  ```
  # ls -l /dev/mapper/rasrivas_vg-rasrivas_lv
  lrwxrwxrwx. 1 root root 7 May 31 22:19 /dev/mapper/rasrivas_vg-rasrivas_lv -> ../dm-3
  # 


### If we want to increase/decrease storage for the entire partition then we can ask Logical Group

- **NOTE**
  - We can only increase the size of the ****Logical Group*** if **Volume Group** has space available
    - otherwise we need tp follow below steps:
      - ***Firstly** we need to increase the size of the **Volume Group** by adding more **Physical Volome**
      - ***Secondly*** then we can increse the size of the ****Logical Group***

- Checking the current size for the **/dev/mapper/rasrivas_vg-rasrivas_lv** filesystem using **df -hT** command
  - It will show its having **3.9G** capacity (**Maximum Size**)
    ```
    # df -hT
    Filesystem                          Type      Size  Used Avail Use% Mounted on
    devtmpfs                            devtmpfs  2.1G     0  2.1G   0% /dev
    tmpfs                               tmpfs     2.1G     0  2.1G   0% /dev/shm
    tmpfs                               tmpfs     2.1G  9.7M  2.1G   1% /run
    tmpfs                               tmpfs     2.1G     0  2.1G   0% /sys/fs/cgroup
    /dev/mapper/rhel_192-root           xfs        50G  4.6G   46G  10% /
    /dev/mapper/rhel_192-home           xfs        45G  352M   45G   1% /home
    /dev/nvme0n1p1                      xfs      1014M  211M  804M  21% /boot
    tmpfs                               tmpfs     420M  1.2M  419M   1% /run/user/42
    tmpfs                               tmpfs     420M  4.0K  420M   1% /run/user/0
    /dev/mapper/rasrivas_vg-rasrivas_lv ext4      3.9G   16M  3.7G   1% /root/rasrivas_mp
    #
    ```

- Now, Increasing the partition size to 500M
  ```
  # lvextend --size +500M /dev/mapper/rasrivas_vg-rasrivas_lv
  Size of logical volume rasrivas_vg/rasrivas_lv changed from 4.00 GiB (1024 extents) to <4.49 GiB (1149 extents).
  Logical volume rasrivas_vg/rasrivas_lv successfully resized.
  # 
  ```
  
- If we again see the for the **/dev/mapper/rasrivas_vg-rasrivas_lv** filesystem using **df -hT** command
  - Again, it will show its having **3.9G** capacity (**Maximum Size**) because we need to ***format*** the partition again.
    - **Reasons**
      - ***df*** command only shows the size which is formated
    ```
    # df -hT
    Filesystem                          Type      Size  Used Avail Use% Mounted on
    devtmpfs                            devtmpfs  2.1G     0  2.1G   0% /dev
    tmpfs                               tmpfs     2.1G     0  2.1G   0% /dev/shm
    tmpfs                               tmpfs     2.1G  9.7M  2.1G   1% /run
    tmpfs                               tmpfs     2.1G     0  2.1G   0% /sys/fs/cgroup
    /dev/mapper/rhel_192-root           xfs        50G  4.6G   46G  10% /
    /dev/mapper/rhel_192-home           xfs        45G  352M   45G   1% /home
    /dev/nvme0n1p1                      xfs      1014M  211M  804M  21% /boot
    tmpfs                               tmpfs     420M  1.2M  419M   1% /run/user/42
    tmpfs                               tmpfs     420M  4.0K  420M   1% /run/user/0
    /dev/mapper/rasrivas_vg-rasrivas_lv ext4      3.9G   16M  3.7G   1% /root/rasrivas_mp
    #
    ```
    
### Formatting the partition
  - If we use below command, it will fisrt delete the data in this partition and then it will format
    `# mkfs.ext4 /dev/rasrivas_vg/rasrivas_lv`
  - We dont want to loose our old data, to overcome this situation, we use ***resize2fs*** command
    - ****resize2fs**** command is used to format the remaining without loosing the old data of same partition
  - Formatting the partition using ***resize2fs*** command
    ```
    # resize2fs /dev/rasrivas_vg/rasrivas_lv
    resize2fs 1.45.4 (23-Sep-2019)
    Filesystem at /dev/rasrivas_vg/rasrivas_lv is mounted on /root/rasrivas_mp; on-line resizing required
    old_desc_blocks = 1, new_desc_blocks = 1
    The filesystem on /dev/rasrivas_vg/rasrivas_lv is now 1176576 (4k) blocks long.
    ```
  - Now, if we check the size for the **/dev/rasrivas_vg/rasrivas_lv** partition, it will be increased to **500M** and will be **4.4G**
    ```
    # df -hT
    Filesystem                          Type      Size  Used Avail Use% Mounted on
    devtmpfs                            devtmpfs  2.1G     0  2.1G   0% /dev
    tmpfs                               tmpfs     2.1G     0  2.1G   0% /dev/shm
    tmpfs                               tmpfs     2.1G  9.7M  2.1G   1% /run
    tmpfs                               tmpfs     2.1G     0  2.1G   0% /sys/fs/cgroup
    /dev/mapper/rhel_192-root           xfs        50G  4.6G   46G  10% /
    /dev/mapper/rhel_192-home           xfs        45G  352M   45G   1% /home
    /dev/nvme0n1p1                      xfs      1014M  211M  804M  21% /boot
    tmpfs                               tmpfs     420M  1.2M  419M   1% /run/user/42
    tmpfs                               tmpfs     420M  4.0K  420M   1% /run/user/0
    /dev/mapper/rasrivas_vg-rasrivas_lv ext4      4.4G   16M  4.2G   1% /root/rasrivas_mp
    #

### Increating the size for the Volume Group

- We can only increase the size of the ****Logical Group*** if **Volume Group** has space available
    - otherwise we need tp follow below steps:
      - ***Firstly** we need to increase the size of the **Volume Group** by adding more **Physical Volome**
      - ***Secondly*** then we can increse the size of the ****Logical Group***

- Creating ***Physical Volume*** for **/dev/nvme0n4** partition which if of **2Gib** size
  ```
  # pvcreate /dev/nvme0n4 
  Physical volume "/dev/nvme0n4" successfully created.
  #
  ```

- Verifying if **/dev/nvme0n4** is created
  ```
  # pvdisplay /dev/nvme0n4
  "/dev/nvme0n4" is a new physical volume of "2.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/nvme0n4
  VG Name               
  PV Size               2.00 GiB
  Allocatable           NO
  PE Size               0   
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               8EXbjv-q8SP-ROG3-RqHq-y1ES-D16q-a3dR3I
  #
  ```

- Now, we will increase the ***Volume Group** size by using **vgextend** command
  ```
  # vgextend rasrivas_vg /dev/nvme0n4
  Volume group "rasrivas_vg" successfully extended
  #
  ```

- Verifying if the size for the **rasrivas_vg**, it should be **4.99 + 2 = 6.99 GiB
  ```
  # vgdisplay rasrivas_vg
  --- Volume group ---
  VG Name               rasrivas_vg
  System ID             
  Format                lvm2
  Metadata Areas        3
  Metadata Sequence No  4
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               0
  Max PV                0
  Cur PV                3
  Act PV                3
  VG Size               <6.99 GiB  <<<== this has been extended to 2 Gib as it was having size of 2 Gib
  PE Size               4.00 MiB
  Total PE              1789
  Alloc PE / Size       1149 / <4.49 GiB
  Free  PE / Size       640 / 2.50 GiB
  VG UUID               G9mXOZ-fEwq-SqQ5-sGXN-Kfac-n7Zb-Tp9euu
  #
  ```

### Decreasing the size of the Logical Volume but first we create the Logical Volume from the scratch

- Adding two new hard disk of size
  - **nvme0n2** (5 G)
  - **nvme0n2** (5 G)
  
  - ```
    # lsblk 
    NAME              MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
    sr0                11:0    1  7.9G  0 rom  /dvd
    nvme0n1           259:0    0  100G  0 disk 
    ├─nvme0n1p1       259:1    0    1G  0 part /boot
    └─nvme0n1p2       259:2    0   99G  0 part 
      ├─rhel_192-root 253:0    0   50G  0 lvm  /
      ├─rhel_192-swap 253:1    0  4.2G  0 lvm  [SWAP]
      └─rhel_192-home 253:2    0 44.8G  0 lvm  /home
    nvme0n2           259:3    0    5G  0 disk 
    nvme0n3           259:4    0    5G  0 disk #
    ```
  
- Now, formatting these partiontions:
  - creating partition for **/dev/nvme0n2**
  ```
  [root@rasrivas ~]# fdisk /dev/nvme0n2

  Welcome to fdisk (util-linux 2.32.1).
  Changes will remain in memory only, until you decide to write them.
  Be careful before using the write command.

  Device does not contain a recognized partition table.
  Created a new DOS disklabel with disk identifier 0x193c523e.

  Command (m for help): n
  Partition type
     p   primary (0 primary, 0 extended, 4 free)
     e   extended (container for logical partitions)
  Select (default p): e
  Partition number (1-4, default 1): 
  First sector (2048-10485759, default 2048): 
  Last sector, +sectors or +size{K,M,G,T,P} (2048-10485759, default 10485759): 

  Created a new partition 1 of type 'Extended' and of size 5 GiB.

  Command (m for help): w
  The partition table has been altered.
  Calling ioctl() to re-read partition table.
  Syncing disks.
  #
  ```
 - Verifying it, we will see a partition **nvme0n2p1** for the **nvme0n2** hard disk
   ```
   [root@rasrivas ~]# lsblk 
   NAME              MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
   sr0                11:0    1  7.9G  0 rom  /dvd
   nvme0n1           259:0    0  100G  0 disk 
   ├─nvme0n1p1       259:1    0    1G  0 part /boot
   └─nvme0n1p2       259:2    0   99G  0 part 
     ├─rhel_192-root 253:0    0   50G  0 lvm  /
     ├─rhel_192-swap 253:1    0  4.2G  0 lvm  [SWAP]
     └─rhel_192-home 253:2    0 44.8G  0 lvm  /home
   nvme0n2           259:3    0    5G  0 disk 
     └─nvme0n2p1       259:6    0    1K  0 part 
   nvme0n3           259:4    0    5G  0 disk 
   #
   ```
   
   - creating partition for **/dev/nvme0n3**
     ```
     # fdisk /dev/nvme0n3

     Welcome to fdisk (util-linux 2.32.1).
     Changes will remain in memory only, until you decide to write them.
     Be careful before using the write command.

     Device does not contain a recognized partition table.
     Created a new DOS disklabel with disk identifier 0xe5ec6e43.

     Command (m for help): n
     Partition type
        p   primary (0 primary, 0 extended, 4 free)
        e   extended (container for logical partitions)
     Select (default p): e
     Partition number (1-4, default 1): 
     First sector (2048-10485759, default 2048): 
     Last sector, +sectors or +size{K,M,G,T,P} (2048-10485759, default 10485759): 

     Created a new partition 1 of type 'Extended' and of size 5 GiB.

     Command (m for help): w
     The partition table has been altered.
     Calling ioctl() to re-read partition table.
     Syncing disks.
     #
     ```
   - verifying it, we will see a new partition **nvme0n3p1** for the **nvme0n3** hard disk
     ```
     [root@rasrivas ~]# lsblk 
     NAME              MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
     sr0                11:0    1  7.9G  0 rom  /dvd
     nvme0n1           259:0    0  100G  0 disk 
     ├─nvme0n1p1       259:1    0    1G  0 part /boot
     └─nvme0n1p2       259:2    0   99G  0 part 
       ├─rhel_192-root 253:0    0   50G  0 lvm  /
       ├─rhel_192-swap 253:1    0  4.2G  0 lvm  [SWAP]
       └─rhel_192-home 253:2    0 44.8G  0 lvm  /home
     nvme0n2           259:3    0    5G  0 disk 
       └─nvme0n2p1       259:6    0    1K  0 part 
     nvme0n3           259:4    0    5G  0 disk 
       └─nvme0n3p1       259:7    0    1K  0 part 
     #
     ```
 
-  Creating **Physical Volume** for the **/dev/nvme0n2** and **/dev/nvme0n3**
   ```
   [root@rasrivas ~]# pvcreate /dev/nvme0n2
   WARNING: dos signature detected on /dev/nvme0n2 at offset 510. Wipe it? [y/n]: y
   Wiping dos signature on /dev/nvme0n2.
   Physical volume "/dev/nvme0n2" successfully created.
   ```
   
   ```
   [root@rasrivas ~]# pvcreate /dev/nvme0n3
   WARNING: dos signature detected on /dev/nvme0n3 at offset 510. Wipe it? [y/n]: y
   Wiping dos signature on /dev/nvme0n3.
   Physical volume "/dev/nvme0n3" successfully created.
   #
   ```
   
   - verifying it using ***pvdisplay***
     ```
     # pvdisplay /dev/nvme0n2
     "/dev/nvme0n2" is a new physical volume of "5.00 GiB"
     --- NEW Physical volume ---
     PV Name               /dev/nvme0n2
     VG Name               
     PV Size               5.00 GiB
     Allocatable           NO
     PE Size               0   
     Total PE              0
     Free PE               0
     Allocated PE          0
     PV UUID               G7hzUX-cqXS-A9v1-9VaS-k4Yl-ZEUd-Klj9RL
     ```
     
     ```
     # pvdisplay /dev/nvme0n3
     "/dev/nvme0n3" is a new physical volume of "5.00 GiB"
     --- NEW Physical volume ---
     PV Name               /dev/nvme0n3
     VG Name               
     PV Size               5.00 GiB
     Allocatable           NO
     PE Size               0   
     Total PE              0
     Free PE               0
     Allocated PE          0
     PV UUID               d4XYBS-3HMS-MURj-0EvE-NceI-24ns-lzoPIh
     #
     ```
     
 - Creating **Volume Group** for the **/dev/nvme0n2** and **/dev/nvme0n3**
 
   - creating **Volume Group**
     ```
     # vgcreate mydemovg /dev/nvme0n2 /dev/nvme0n3
     Volume group "mydemovg" successfully created
     #
     ```
   
   - verifying ***volume group*** for the **mydemovg**, it should around **10 GB** space and available ***PV count*** should be **2**
     ```
     [root@rasrivas ~]# vgdisplay mydemovg
     --- Volume group ---
     VG Name               mydemovg
     System ID             
     Format                lvm2
     Metadata Areas        2
     Metadata Sequence No  1
     VG Access             read/write
     VG Status             resizable
     MAX LV                0
     Cur LV                0
     Open LV               0
     Max PV                0
     Cur PV                2
     Act PV                2
     VG Size               9.99 GiB
     PE Size               4.00 MiB
     Total PE              2558
     Alloc PE / Size       0 / 0   
     Free  PE / Size       2558 / 9.99 GiB
     VG UUID               KlvYYf-tlSb-Jd0p-d6sr-ORAO-MEPD-q0QM5G
     #
     ```
 
- Now, now finally creating ***logical volume***
  - creating a LV of 7 GB size
    ```
    # lvcreate --size 7G --name mydemolv mydemovg
    WARNING: dos signature detected on /dev/mydemovg/mydemolv at offset 510. Wipe it? [y/n]: y
    Wiping dos signature on /dev/mydemovg/mydemolv.
    Logical volume "mydemolv" created.
    #
    ```
    
 - verifying it 
   ```
   # lvdisplay 
   --- Logical volume ---
   LV Path                /dev/mydemovg/mydemolv
   LV Name                mydemolv
   VG Name                mydemovg
   LV UUID                aT4ihP-15Vb-2nDl-jJaq-vduP-q7Xs-CwN0CG
   LV Write Access        read/write
   LV Creation host, time rasrivas.example.com, 2020-07-11 16:37:40 +0530
   LV Status              available
   # open                 0
   LV Size                7.00 GiB
   Current LE             1792
   Segments               2
   Allocation             inherit
   Read ahead sectors     auto
   - currently set to     8192
   Block device           253:3
   ```

- Now, we will format this partition
  ```
  [root@rasrivas ~]# mkfs.ext4 /dev/mydemovg/mydemolv
  mke2fs 1.45.4 (23-Sep-2019)
  Creating filesystem with 1835008 4k blocks and 458752 inodes
  Filesystem UUID: fe2bf001-7323-4f9c-a00e-525f12ed6098
  Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632

  Allocating group tables: done                            
  Writing inode tables: done                            
  Creating journal (16384 blocks): done
  Writing superblocks and filesystem accounting information: done 
  #
  ```

- Now, mounting this partition
  ```
  # mkdir mydemodirectory
  # ls -la mydemodirectory
  total 4
  drwxr-xr-x.  2 root root    6 Jul 11 16:41 .
  dr-xr-x---. 26 root root 4096 Jul 11 16:41 ..
  #
  ```
  
  ```
  # mount /dev/mydemovg/mydemolv mydemodirectory
  #
  ```
  
  - verfiying it
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
  └─mydemovg-mydemolv 253:3    0    7G  0 lvm  /root/mydemodirectory
  nvme0n3             259:4    0    5G  0 disk 
  └─mydemovg-mydemolv 253:3    0    7G  0 lvm  /root/mydemodirectory
  #
  ```
  

### Now, we will decrease the size the **Logical Volume**

- There are five steps:
  - Offline (Umonut )the partition
  - Clean/Scan the partition --> 
  - Reformat
  - LV reduce
  - Online (Mount)

**Step1:** Offline (Umonut) the partition
  
  - Listing the existing partitions **/dev/mapper/mydemovg-mydemolv** which is mounted at **/root/mydemodirectory** location
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
    /dev/mapper/mydemovg-mydemolv ext4      6.9G   32M  6.5G   1% /root/mydemodirectory
    #
    ```
    
  - Unmonting the partitons, which is also called making this partition as **OFFLINE**
    ```
    # umount /root/mydemodirectory
    #
    ```
    
  - Verifying it
    
    ```
    # df -hT
    Filesystem                Type      Size  Used Avail Use% Mounted on
    devtmpfs                  devtmpfs  1.6G     0  1.6G   0% /dev
    tmpfs                     tmpfs     1.6G  4.0K  1.6G   1% /dev/shm
    tmpfs                     tmpfs     1.6G  9.7M  1.6G   1% /run
    tmpfs                     tmpfs     1.6G     0  1.6G   0% /sys/fs/cgroup
    /dev/mapper/rhel_192-root xfs        50G  9.9G   41G  20% /
    /dev/mapper/rhel_192-home xfs        45G  352M   45G   1% /home
    /dev/nvme0n1p1            xfs      1014M  211M  804M  21% /boot
    /dev/sr0                  iso9660   7.9G  7.9G     0 100% /dvd
    tmpfs                     tmpfs     316M  1.2M  315M   1% /run/user/42
    tmpfs                     tmpfs     316M  4.0K  316M   1% /run/user/0
    # 
    ```
  
**Step2:** Scan and Clean the Filesystem

  - Scanning and cleaning the **/dev/mapper/mydemovg-mydemolv** Filesystem using **e2fsck** command
    - This will check all the bad/ corrupt sectors for this Filesystem
   
    ```
    # e2fsck -f /dev/mapper/mydemovg-mydemolv
    e2fsck 1.45.4 (23-Sep-2019)
    Pass 1: Checking inodes, blocks, and sizes
    Pass 2: Checking directory structure
    Pass 3: Checking directory connectivity
    Pass 4: Checking reference counts
    Pass 5: Checking group summary information
    /dev/mapper/mydemovg-mydemolv: 11/458752 files (0.0% non-contiguous), 53247/1835008 blocks
    #
    ```

**Step3:** Reformat the Filesystem to reduce the size

- Now, we can want to reduce the size to 6G
- For this we need to online format it

- Reormatting it to reduce the size to 6G
  ```
  # resize2fs /dev/mapper/mydemovg-mydemolv 6G
  resize2fs 1.45.4 (23-Sep-2019)
  Resizing the filesystem on /dev/mapper/mydemovg-mydemolv to 1572864 (4k) blocks.
  The filesystem on /dev/mapper/mydemovg-mydemolv is now 1572864 (4k) blocks long.
  #
  ```
- Now, we check the partition size it wil still show 7G
  ```
  # lvdisplay 
  --- Logical volume ---
  LV Path                /dev/mydemovg/mydemolv
  LV Name                mydemolv
  VG Name                mydemovg
  LV UUID                aT4ihP-15Vb-2nDl-jJaq-vduP-q7Xs-CwN0CG
  LV Write Access        read/write
  LV Creation host, time rasrivas.example.com, 2020-07-11 16:37:40 +0530
  LV Status              available
  # open                 0
  LV Size                7.00 GiB
  Current LE             1792
  Segments               2
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:3
  ```

**Step4:** Reduce the size

- Reducing the size for the LV
  - it can be using by two commands
    - lvreduce --size -1G /dev/mydemovg/mydemolv
    - lvreduce --size 6G /dev/mydemovg/mydemolv
  - Reducing it
    ```
    # lvreduce --size 6G /dev/mydemovg/mydemolv
    WARNING: Reducing active logical volume to 6.00 GiB.
    THIS MAY DESTROY YOUR DATA (filesystem etc.)
    Do you really want to reduce mydemovg/mydemolv? [y/n]: y
    Size of logical volume mydemovg/mydemolv changed from 7.00 GiB (1792 extents) to 6.00 GiB (1536 extents).
    Logical volume mydemovg/mydemolv successfully resized.
    #
    ```

- verifying it, it should be now **6.00 GiB**
  ```
  # lvdisplay 
  --- Logical volume ---
  LV Path                /dev/mydemovg/mydemolv
  LV Name                mydemolv
  VG Name                mydemovg
  LV UUID                aT4ihP-15Vb-2nDl-jJaq-vduP-q7Xs-CwN0CG
  LV Write Access        read/write
  LV Creation host, time rasrivas.example.com, 2020-07-11 16:37:40 +0530
  LV Status              available
  # open                 0
  LV Size                6.00 GiB
  Current LE             1536
  Segments               2
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:3
  ```

- now, we can see around **3.99 GiB** is free in the VG
  ```
  # vgdisplay mydemovg
  --- Volume group ---
  VG Name               mydemovg
  System ID             
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               0
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               9.99 GiB
  PE Size               4.00 MiB
  Total PE              2558
  Alloc PE / Size       1536 / 6.00 GiB
  Free  PE / Size       1022 / 3.99 GiB
  VG UUID               KlvYYf-tlSb-Jd0p-d6sr-ORAO-MEPD-q0QM5G
  #
  ```

**Step5:** mounting the partition

- mounting it
  ```
  # mount /dev/mydemovg/mydemolv mydemodirectory
  ```

- we can verify it
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
  /dev/mapper/mydemovg-mydemolv ext4      5.9G   28M  5.6G   1% /root/mydemodirectory
  ```

### Again if we want to increase the size of the LV

```
# lvextend --size +2G /dev/mydemovg/mydemolv
Size of logical volume mydemovg/mydemolv changed from 6.00 GiB (1536 extents) to 8.00 GiB (2048 extents).
Logical volume mydemovg/mydemolv successfully resized.
```

- now, we can see the size will be incresed by 2GB and total will be 8 GB
```
# lvdisplay 
  --- Logical volume ---
  LV Path                /dev/mydemovg/mydemolv
  LV Name                mydemolv
  VG Name                mydemovg
  LV UUID                aT4ihP-15Vb-2nDl-jJaq-vduP-q7Xs-CwN0CG
  LV Write Access        read/write
  LV Creation host, time rasrivas.example.com, 2020-07-11 16:37:40 +0530
  LV Status              available
  # open                 1
  LV Size                8.00 GiB
  Current LE             2048
  Segments               2
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:3
```

- remount it
  ```
  # resize2fs /dev/mydemovg/mydemolv
  resize2fs 1.45.4 (23-Sep-2019)
  Filesystem at /dev/mydemovg/mydemolv is mounted on /root/mydemodirectory; on-line resizing required
  old_desc_blocks = 1, new_desc_blocks = 1
  The filesystem on /dev/mydemovg/mydemolv is now 2097152 (4k) blocks long.
  ```
  
- we can verify it
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

### Formatting the partition for "xfs" format type

- **resize2fs** only works for ***ext4*** format type
- For **xfs** format type, we have to use
  
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
  #
  ```
- below is command for it
  ```
  # xfs_growfs /dev/mapper/rhel_192-home
  meta-data=/dev/mapper/rhel_192-home isize=512    agcount=4, agsize=2934016 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1
  data     =                       bsize=4096   blocks=11736064, imaxpct=25
         =                       sunit=0      swidth=0 blks
  naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
  log      =internal log           bsize=4096   blocks=5730, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
  realtime =none                   extsz=4096   blocks=0, rtextents=0
  ```
  
### Extending and reducing the "xfs" type format

- **xfs** format type which uses **xfs_growfs** command does not support for reduce of LVM (it only supports for extend)


### Size type provided by PV and VG

- Size type provided by **PV** and **VG** are not in **byte** or **sector** type
- Its in form of ***PE Size*** for **PV** and **LV** and ***PE Size*** size is by default **4.00 MiB**

- For example:
  ```
  # pvdisplay /dev/nvme0n2
  --- Physical volume ---
  PV Name               /dev/nvme0n2
  VG Name               mydemovg
  PV Size               5.00 GiB / not usable 4.00 MiB
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              1279
  Free PE               0
  Allocated PE          1279
  PV UUID               G7hzUX-cqXS-A9v1-9VaS-k4Yl-ZEUd-Klj9RL
  ```
- here **PE Size** is **4.00 MiB** and total **PE size** is **1279**
- **4.00 MiB** * **1279** = **5116 MiB** OR **5.00 GiB**

- This we are discussing because whenever we lv for which is not mulitple of 4 either will increse the size or decrease the size of its own
- In below example, we are going to create **lv** of **10M**, as its ***not multiple of 4***, either will create **8M** or **12M**
  ```
  # lvcreate --size 10M --name demolv2 mydemovg
  Rounding up size to full physical extent 12.00 MiB
  Logical volume "demolv2" created.
  ```

- If we want to change the size of **PE Size**, we can use `$ vgcreate -s` OR `$ vgcreate -physicalextentsize` command
  ```
  $ man vgcreate
   -s|--physicalextentsize Size[m|UNIT]
              Sets the physical extent size of PVs in the VG.  The value must be either a power of 2 of at least 1
              sector (where the sector size is the largest sector size of the PVs currently used in the VG), or at
              least 128KiB.  Once this value has been set, it is difficult to change without recreating the VG,
              unless no extents need moving.
  ```
