---
title: "SWAP partition in Linux"
date: 2022-05-08T18:36:43+05:30
draft: false
tags: ["linux", "filesysystem", "partition", "swap"]
categories: ["linux", "operating system"]
---

- Swap space in Linux is used when the amount of physical memory (RAM) is full.
- If the system needs more memory resources and the RAM is full, inactive pages - in memory are moved to the swap space.
- While swap space can help machines with a small amount of RAM, it should not be considered a replacement for more RAM.
- Swap space is located on hard drives, which have a slower access time than physical memory.
- Swap space can be a dedicated swap partition (recommended), a swap file, or a combination of swap partitions and swap files.
- Virtual memory is not same as SWAP partition

### Listing the available hard disk
  ```
  $ lsblk 
  NAME                MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
  sr0                  11:0    1  7.9G  0 rom  /dvd
  nvme0n1             259:0    0  100G  0 disk 
  ├─nvme0n1p1         259:1    0    1G  0 part /boot
  └─nvme0n1p2         259:2    0   99G  0 part 
    ├─rhel_192-root   253:0    0   50G  0 lvm  /
    ├─rhel_192-swap   253:1    0  4.2G  0 lvm  [SWAP]
    └─rhel_192-home   253:2    0 44.8G  0 lvm  /home
  nvme0n2             259:3    0    5G  0 disk 
  └─mydemovg-mydemolv 253:3    0    8G  0 lvm  
  nvme0n3             259:4    0    5G  0 disk 
  ├─mydemovg-mydemolv 253:3    0    8G  0 lvm  
  └─mydemovg-demolv2  253:4    0   12M  0 lvm  
  nvme0n4             259:5    0   20G  0 disk
  ```

  - We can see **nvme0n4** if available

#### Creating a 5 GB primary partition for the nvme0n4
  ```
  # fdisk /dev/nvme0n4

  Welcome to fdisk (util-linux 2.32.1).
  Changes will remain in memory only, until you decide to write them.
  Be careful before using the write command.

  Device does not contain a recognized partition table.
  Created a new DOS disklabel with disk identifier 0x2e5bfd47.

  Command (m for help): n
  Partition type
     p   primary (0 primary, 0 extended, 4 free)
     e   extended (container for logical partitions)
  First sector (2048-41943039, default 2048): 
  Last sector, +sectors or +size{K,M,G,T,P} (2048-41943039, default 41943039): +5G

  Created a new partition 1 of type 'Linux' and of size 5 GiB.

  Command (m for help): w
  The partition table has been altered.
  Calling ioctl() to re-read partition table.
  Syncing disks.
  #
  ```

#### Partition is created for nvme0n4 as nvme0n4p1
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
  └─mydemovg-mydemolv 253:3    0    8G  0 lvm  
  nvme0n3             259:4    0    5G  0 disk 
  ├─mydemovg-mydemolv 253:3    0    8G  0 lvm  
  └─mydemovg-demolv2  253:4    0   12M  0 lvm  
  nvme0n4             259:5    0   20G  0 disk 
  └─nvme0n4p1         259:7    0    5G  0 part
  ```

####  Update udevadm
   ```
   # udevadm settle
   #
   ```
  - After the kernel boots, udevd is used to create device nodes for all detected devices.
  - That is a relatively time consuming task that has to be completed for the boot process to continue, otherwise there is a risk of services failing due to missing device nodes.
  - `udevadm settle` waits for udevd to process the device creation events for all hardware devices, thus ensuring that any device nodes have been created successfully before proceeding.

#### Creating SWAP
  ```
  # mkswap /dev/nvme0n4p1
  Setting up swapspace version 1, size = 5 GiB (5368705024 bytes)
  no label, UUID=9f42d441-5934-4cde-ac9b-380aa4c81912
  ```
 
#### we can verfiy it, it doesn't show the new swap
  ```
  # swapon -s
  Filename				Type		Size	Used	Priority
  /dev/dm-1                              	partition	4431868	36096	-2
  ```

#### also, if we see using **free** command, we already had `4.2Gi` total
  ```
  # free -h
              total        used        free      shared  buff/cache   available
  Mem:          3.1Gi       1.8Gi       822Mi       5.0Mi       501Mi       1.1Gi
  Swap:         4.2Gi        35Mi       4.2Gi
  ```

#### To show this, We have to active it
  ```
  # swapon /dev/nvme0n4p1
  ```

#### If we now verify it, we see the new partition added
  ```
  # swapon -s
  Filename				Type		Size	Used	Priority
  /dev/dm-1                              	partition	4431868	36096	-2
  /dev/nvme0n4p1                         	partition	5242876	0	-3
  ```

#### Now, we can see total swap m/m as **9.2Gi**
  ```
  # free -h
              total        used        free      shared  buff/cache   available
  Mem:          3.1Gi       1.8Gi       817Mi       5.0Mi       501Mi       1.1Gi
  Swap:         9.2Gi        35Mi       9.2Gi
  ```

#### Again this is tempoary in nature, to make it permanent, we need to make an entry in fstab
  ```
  /dev/nvme0n4p1 swap swap defaults 0 0
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

  /dev/nvme0n4p1 swap swap defaults 0 0
  ```
  
  - we can verify if everything entired is correct
    ```
    # mount -a
    #
    ```

### How much SWAP we can create

- Any amount of SWAP memory we can create but only depends upon your use case how much RAM is being used in fixing months according to which we can decide the SWAP memory.

### Use case of SWAP memory

#### Lets create a system which has
  - **2 GiB** of **SWAP** memory
  - **4 GiB** of real **RAM** memory

- Now, lets suppose the real **RAM** memory is consumed by **3.5 GiB** of memory and only **500 MiB** of memory is left
- And we have a image of **1 GiB** which has to loaded as user wants to view the picture
- But, actually we have only **500 MiB** of memory is left
- Now, what happens there will be call between **SWAP** memory and **RAM** memory
  - **SWAP** memory will be say to **RAM** memory, I have a **1 GiB** image which needs to viewed by user (for this it should actually load on RAM only because while CPU processing, it will only look for real **RAM** memory not **SWAP** memory)
  - So, in this case, what **RAM** will do, it will look into its internal processes/programs which is already loaded on it.
  - and will search for the last viewed processes/programs by the CPU, these processes/programs will be moved to **SWAP** memory which is logically on physical hard disk (this is called as **SWIPE OUT**) and the image which has to be viewed by user will be loaded on real **RAM** and **CPU** can load it and view to the user. 
  - If the again the processes/programs which is loaded on **SWAP** memory is called by user (CPU), then again these processes/programs can be moved back on real **RAM** memory (this is called as **SWIPE OUT**)
- **Note:** In all the ways the real **RAM** memory will be only **4 GiB** and CPU will (can) process processes/programs running on real **RAM** memory
  
- we have a another command to view more stats on SWAP m/m
  ```
  # vmstat 
  procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
  r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
  0  0  36096 829532     40 520520    1    4    71    24  105  218  1  1 98  0  0
  ```

#### Another use case of SWAP memory

- https://opensource.com/article/18/9/swap-space-linux-systems

- There are two basic types of memory in a typical computer.
- The first type, random access memory (RAM), is used to store data and programs while they are being actively used by the computer.
- Programs and data cannot be used by the computer unless they are stored in RAM.
- RAM is volatile memory; that is, the data stored in RAM is lost if the computer is turned off.

- Hard drives are magnetic media used for long-term storage of data and programs.
- Magnetic media is nonvolatile; the data stored on a disk remains even when power is removed from the computer.
- The CPU (central processing unit) cannot directly access the programs and data on the hard drive; it must be copied into RAM first, and that is where the CPU can access its programming instructions and the data to be operated on by those instructions.
- During the boot process, a computer copies specific operating system programs, such as the kernel and init or systemd, and data from the hard drive into RAM, where it is accessed directly by the computer’s processor, the CPU.

### Swap space
  - The primary function of swap space is to substitute disk space for RAM memory when real RAM fills up and more space is needed.
  - For example, assume you have a computer system with **8GB of RAM**.
  - If you start up programs that don’t fill that RAM, everything is fine and no swapping is required.
  - But suppose the spreadsheet you are working on grows when you add more rows, and that, plus everything else that's running, now fills all of RAM.
  - **Without swap space available**, you would have to stop working on the spreadsheet until you could free up some of your limited RAM by **closing down some other programs**.
  - The **kernel** uses a memory management program that detects ***blocks, aka pages, of memory*** in which the **contents have not been used recently**.
  - The ***memory management program*** **swaps enough of these relatively infrequently used pages of memory out*** to a special partition on the hard drive specifically designated for **paging** or **swapping**.
  - This ***frees up RAM and makes room for more data*** to be entered into your spreadsheet.
  - Those pages of memory **swapped out** to the hard drive are tracked by the kernel’s memory management code and can be **paged back into RAM if they are needed**.
  - The **total amount of memory** in a Linux computer is the **RAM plus swap space** and is referred to as ***virtual memory***.

### Types of Linux swap

- Linux provides for two types of swap space.
- By default, most Linux installations create a **swap partition**, but it is also possible to use a specially configured file as a swap file.
- A **swap partition** is just what its name implies—a standard disk partition that is designated as swap space by the **mkswap** command.

- A **swap file** can be used if there is no free disk space in which to create a new swap partition or space in a volume group where a logical volume can be created for swap space.
- This is just a regular file that is created and preallocated to a specified size.
- Then the **mkswap command** is run to configure it as swap space.
- I don’t recommend using a file for swap space unless absolutely necessary.

### Thrashing

- **Thrashing** can occur when **total virtual memory**, ***both RAM and swap space***, become **nearly full**.
- The system spends so much time paging blocks of memory between swap space and RAM and back that little time is left for real work.
- The typical symptoms of this are obvious:
  - The system becomes slow or completely unresponsive
  - and the hard drive activity light is on almost constantly.

- If you can manage to issue a command like free that shows CPU load and memory usage, you will see that the CPU load is very high, perhaps as much as 30 to 40 times the number of CPU cores in the system.
- Another symptom is that both RAM and swap space are almost completely allocated.

- After the fact, looking at SAR (system activity report) data can also show these symptoms.
- I install SAR on every system I work on and use it for post-repair forensic analysis.
