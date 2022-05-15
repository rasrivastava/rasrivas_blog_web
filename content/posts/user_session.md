---
title: "User Session Management in Linux"
date: 2022-05-15T14:03:54+05:30
draft: false
tags: ["linux", "session", "user_management", "cgroup"]
categories: ["linux", "operating system", "user_management", "cgroup"]
---

- `program` --> is executed by the operating system with the help of kernel
  - `OS (kernel)` (RAM + CPU) --> program converts into process and loads on the RAM
  - `RAM` <-- will the take all the information from the CPU

- If the entire RAM or CPU is asked by the program, by default the operating system assigns the resources but this can be not good practice for the other (crtical) resources.

- So, we have to tuned the system to avoid these, we can say a program to bind to only use the 'n' cpus and 'm' gb of memory, which means we can entirely system will not be used by this process.

### Current the logged user details

- once we login successfuly we get a session:

```
[root@rhel8-box ~]# w
 10:08:09 up 8 min,  1 user,  load average: 0.95, 1.17, 0.79
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
vagrant  pts/0    10.0.2.2         10:03    1.00s  0.08s  0.06s sshd: vagrant [priv]
[root@rhel8-box ~]#
```

```
[root@rhel8-box ~]# loginctl
SESSION  UID USER    SEAT TTY
      3 1000 vagrant

1 sessions listed.
[root@rhel8-box ~]#
```

- Now, loging to the system using also the same user but using other terminal.
  - Same user can login multiple times but all the times a new session will be created.

```
#vagrant ssh rhel8
Last login: Fri Apr 29 10:03:10 2022 from 10.0.2.2
[vagrant@rhel8-box ~]$
```

```
[vagrant@rhel8-box ~]$ w
 10:10:00 up 10 min,  2 users,  load average: 0.84, 1.02, 0.78
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
vagrant  pts/0    10.0.2.2         10:03    1:24   0.07s  0.06s sshd: vagrant [priv]
vagrant  pts/1    10.0.2.2         10:08    0.00s  0.03s  0.01s w
[vagrant@rhel8-box ~]$
```

- From the above command we can see two TTY are created i.e. `pts/0` and `pts/1` for the same user **vagrant**.

- Below command shows all the current active sessions using `loginctl`:

```
[root@rhel8-box ~]# loginctl
SESSION  UID USER    SEAT TTY
      3 1000 vagrant
      6 1000 vagrant <------- new sessoion is created

2 sessions listed.
[root@rhel8-box ~]#
```

- there is another comment to list all the session using `loginctl list-sessions`:

```
[root@rhel8-box ~]# loginctl list-sessions
SESSION  UID USER    SEAT TTY
      3 1000 vagrant
      6 1000 vagrant

2 sessions listed.
[root@rhel8-box ~]#
```

- To see the more details related to the each sessions:

```
[root@rhel8-box ~]# loginctl user-status
vagrant (1000)
           Since: Sat 2022-04-30 05:37:06 UTC; 4min 49s ago
           State: active
        Sessions: *1
          Linger: no
            Unit: user-1000.slice
                  ├─session-1.scope
                  │ ├─1796 sshd: vagrant [priv]
                  │ ├─1842 sshd: vagrant@pts/0
                  │ ├─1843 -bash
                  │ ├─2097 sudo su
                  │ ├─2105 su
                  │ ├─2110 bash
                  │ ├─3740 loginctl user-status
                  │ └─3741 less
                  └─user@1000.service
                    └─init.scope
                      ├─1823 /usr/lib/systemd/systemd --user
                      └─1832 (sd-pam)
[root@rhel8-box ~]#
```

### Command to check user details

- `id <user_name>` command

```
[root@rhel8-box ~]# id vagrant
uid=1000(vagrant) gid=1000(vagrant) groups=1000(vagrant)
[root@rhel8-box ~]#
```

### To check the more details about the logged session

```
[root@rhel8-box ~]# loginctl show-session 3
Id=3
User=1000 <-------------------------- `user ID`
Name=vagrant
Timestamp=Fri 2022-04-29 10:03:10 UTC
TimestampMonotonic=212028431
VTNr=0
Remote=yes <------------------------- `Remote Login`
RemoteHost=10.0.2.2
Service=sshd <----------------------- `using ssh`
Scope=session-3.scope <-------------- `scope of this session` --> `using this we can entirely control this session`
Leader=5332
Audit=3
Type=tty
Class=user
Active=yes
State=active
IdleHint=no
IdleSinceHint=0
IdleSinceHintMonotonic=0
LockedHint=no
[root@rhel8-box ~]#
```

### Controlling a session using CGroup

- Using the **scope** we can entrely control the session like `resources/tasks`

- Suppose we can run `which date` command, so this **command** becomes **process** when its loaded on the RAM and this **process** is also know as **task**.

- This feature is provided by **Cgroup**.

- The **cgroup** can control on `users` as well as `sessions`.

### Status of scope of a session

- scope for the **session-3.scope**

```
[root@rhel8-box ~]# systemctl status session-3.scope
● session-3.scope - Session 3 of user vagrant
   Loaded: loaded (/run/systemd/transient/session-3.scope; transient) <---- `transient`: as session stops, this scope will be deleted
Transient: yes
   Active: active (running) since Fri 2022-04-29 10:03:10 UTC; 30min ago
    Tasks: 8 <---- 8 tasks are already running on this session
   Memory: 9.2M <----  these 8 tasks are using 9.2M memory but these tasks want to on entire RAM or CPU, they can run becuase there are no contrainst set on this session
   CGroup: /user.slice/user-1000.slice/session-3.scope
           ├─5332 sshd: vagrant [priv]
           ├─5346 sshd: vagrant@pts/0
           ├─5347 -bash
           ├─5378 sudo su -
           ├─5380 su -
           ├─5381 -bash
           ├─5784 systemctl status session-3.scope
           └─5785 less

Warning: Journal has been rotated since unit was started. Log output is incomplete or unavailable.
[root@rhel8-box ~]#
```

- scope for the **session-6.scope**

```
[vagrant@rhel8-box ~]$ systemctl status session-6.scope
● session-6.scope - Session 6 of user vagrant
   Loaded: loaded (/run/systemd/transient/session-6.scope; transient)
Transient: yes
   Active: active (running) since Fri 2022-04-29 10:13:12 UTC; 23min ago
    Tasks: 5 <---- 5 tasks are already running on this session
   Memory: 4.0M
   CGroup: /user.slice/user-1000.slice/session-6.scope
           ├─5537 sshd: vagrant [priv]
           ├─5540 sshd: vagrant@pts/1
           ├─5541 -bash
           ├─5828 systemctl status session-6.scope
           └─5829 less
[vagrant@rhel8-box ~]$
```

### Sessions used by the user

```
[root@rhel8-box ~]# loginctl user-status vagrant
vagrant (1000)
           Since: Fri 2022-04-29 10:03:10 UTC; 38min ago
           State: active
        Sessions: 6 *3
          Linger: no
            Unit: user-1000.slice
                  ├─session-3.scope <--- first session 'session-3.scope'
                  │ ├─5332 sshd: vagrant [priv]
                  │ ├─5346 sshd: vagrant@pts/0
                  │ ├─5347 -bash
                  │ ├─5378 sudo su -
                  │ ├─5380 su -
                  │ ├─5381 -bash
                  │ ├─5889 loginctl user-status vagrant
                  │ └─5890 less
                  ├─session-6.scope <-- second session 'session-6.scope'
                  │ ├─5537 sshd: vagrant [priv]
                  │ ├─5540 sshd: vagrant@pts/1
                  │ └─5541 -bash
                  └─user@1000.service
                    └─init.scope
                      ├─5336 /usr/lib/systemd/systemd --user
                      └─5340 (sd-pam)
[root@rhel8-box ~]#
```

## 'loginctl show-session' command

```
[root@rhel8-box ~]# loginctl show-session
EnableWallMessages=no
NAutoVTs=6
KillUserProcesses=no
RebootToFirmwareSetup=no
IdleHint=no
IdleSinceHint=0
IdleSinceHintMonotonic=0
DelayInhibited=sleep
InhibitDelayMaxUSec=5s
HandlePowerKey=poweroff
HandleSuspendKey=suspend
HandleHibernateKey=hibernate
HandleLidSwitch=suspend
HandleLidSwitchDocked=ignore
HoldoffTimeoutUSec=30s
IdleAction=ignore
IdleActionUSec=30min
PreparingForShutdown=no
PreparingForSleep=no
Docked=yes
RemoveIPC=no
RuntimeDirectorySize=190287872
InhibitorsMax=8192
NCurrentInhibitors=1
SessionsMax=8192   <---- maximum number can login, it may be same or different session
NCurrentSessions=2 <---- current session
[root@rhel8-box ~]#
```

### Controlling the resources of the system on a particular session using UNIT

```
[root@rhel8-box ~]# systemctl cat session-3.scope
# /run/systemd/transient/session-3.scope
# This is a transient unit file, created programmatically via the systemd API. Do not edit.
[Scope]
Slice=user-1000.slice

[Unit]
Description=Session 3 of user vagrant
After=systemd-logind.service
After=systemd-user-sessions.service

[Scope]  <---- `this helps in setting the scope`
SendSIGHUP=yes
TasksMax=infinity <-- `by default its set to infinity i.e. any amount of resources any used`
[root@rhel8-box ~]#
```

- Editing this unit file --> it will overwrite the `session-3.scope` file

```
[root@rhel8-box ~]# systemctl edit session-1.scope
[Scope]
SendSIGHUP=yes
TasksMax=infinity <---- TasksMax=10, instead of infinity it will run max 10 tasks
[root@rhel8-box ~]#
```

```
[root@rhel8-box ~]# systemctl cat session-3.scope
# /run/systemd/transient/session-3.scope
# This is a transient unit file, created programmatically via the systemd API. Do not edit.
[Scope]
Slice=user-1000.slice

[Unit]
Description=Session 3 of user vagrant
After=systemd-logind.service
After=systemd-user-sessions.service

[Scope]
SendSIGHUP=yes
TasksMax=infinity

# /etc/systemd/system/session-3.scope.d/override.conf
[Scope]
SendSIGHUP=yes
TasksMax=10 <--- `max of 10 is set using previous command`
[root@rhel8-box ~]#
```

- verify

```
[root@rhel8-box ~]# systemctl status session-1.scope
● session-1.scope - Session 1 of user vagrant
   Loaded: loaded (/run/systemd/transient/session-1.scope; transient)
Transient: yes
  Drop-In: /etc/systemd/system/session-1.scope.d
           └─override.conf
   Active: active (running) since Sat 2022-04-30 05:37:06 UTC; 8min ago
    Tasks: 8 (limit: 10) <---------- `max/limit of 10 is applied`
   Memory: 13.3M
   CGroup: /user.slice/user-1000.slice/session-1.scope
           ├─1796 sshd: vagrant [priv]
           ├─1842 sshd: vagrant@pts/0
           ├─1843 -bash
           ├─2097 sudo su
           ├─2105 su
           ├─2110 bash
           ├─3909 systemctl status session-1.scope
           └─3910 systemctl status session-1.scope

Warning: Journal has been rotated since unit was started. Log output is incomplete or unavailable.
[root@rhel8-box ~]#
```

### Verify the resource limit

- Currently limit is 10 is set, now lets create some new tasks

```
[root@rhel8-box ~]# sleep 100 &
[3] 3927
[root@rhel8-box ~]# sleep 100 &
[4] 3930
[root@rhel8-box ~]#
```

```
[root@rhel8-box ~]# systemctl status session-1.scope
Failed to fork: Resource temporarily unavailable
● session-1.scope - Session 1 of user vagrant
   Loaded: loaded (/run/systemd/transient/session-1.scope; transient)
Transient: yes
  Drop-In: /etc/systemd/system/session-1.scope.d
           └─override.conf
   Active: active (running) since Sat 2022-04-30 05:37:06 UTC; 14min ago
    Tasks: 10 (limit: 10) <---- `all the 10 used now`
   Memory: 13.8M
   CGroup: /user.slice/user-1000.slice/session-1.scope
           ├─1796 sshd: vagrant [priv]
           ├─1842 sshd: vagrant@pts/0
           ├─1843 -bash
           ├─2097 sudo su
           ├─2105 su
           ├─2110 bash
           ├─3982 sleep 100
           ├─3986 sleep 100
           ├─3989 sleep 100
           └─3991 systemctl status session-1.scope

Warning: Journal has been rotated since unit was started. Log output is incomplete or unavailable.
[root@rhel8-box ~]#
```

- It shows `bash: fork: retry: Resource temporarily unavailable` error as the tasks are full
```
[root@rhel8-box ~]# sleep 100 &
bash: fork: retry: Resource temporarily unavailable
bash: fork: retry: Resource temporarily unavailable
bash: fork: retry: Resource temporarily unavailable
^Cbash: fork: Interrupted system call

[root@rhel8-box ~]# date
bash: fork: retry: Resource temporarily unavailable
bash: fork: retry: Resource temporarily unavailable
bash: fork: retry: Resource temporarily unavailable
^Cbash: fork: Interrupted system call

[root@rhel8-box ~]#
```

- After some time, testing back, now it works fine

```
[root@rhel8-box ~]# date
Sat Apr 30 05:49:15 UTC 2022
[1]   Done                    sleep 30
[2]   Done                    sleep 30
[3]-  Done                    sleep 30
[4]+  Done                    sleep 100
[root@rhel8-box ~]#
[root@rhel8-box ~]# date
Sat Apr 30 05:49:20 UTC 2022
[root@rhel8-box ~]#
```

## View all the CGROUP

```
[vagrant@rhel8-box cgroup]$ pwd
/sys/fs/cgroup
[vagrant@rhel8-box cgroup]$
```

```
[vagrant@rhel8-box cgroup]$ ls
blkio  cpu,cpuacct  cpuset   freezer  memory   net_cls,net_prio  perf_event  rdma
cpu    cpuacct      devices  hugetlb  net_cls  net_prio          pids        systemd
[vagrant@rhel8-box cgroup]$
```

```
[vagrant@rhel8-box user.slice]$ pwd
/sys/fs/cgroup/pids/user.slice
[vagrant@rhel8-box user.slice]$
```

```
[vagrant@rhel8-box user.slice]$ ls
cgroup.clone_children  notify_on_release  pids.events  tasks
cgroup.procs           pids.current       pids.max     user-1000.slice
[vagrant@rhel8-box user.slice]$
```

```
[vagrant@rhel8-box user-1000.slice]$ pwd
/sys/fs/cgroup/pids/user.slice/user-1000.slice
```

```
[vagrant@rhel8-box user-1000.slice]$ ls
cgroup.clone_children  notify_on_release  pids.events  session-1.scope  user@1000.service
cgroup.procs           pids.current       pids.max     tasks
[vagrant@rhel8-box user-1000.slice]$
```

- **above we can see there is one session created `session-1.scope`, lets create a new session**

```
vagrant ssh rhel8
Last login: Wed May  4 14:02:47 2022 from 10.0.2.2
[vagrant@rhel8-box ~]$
```

```
[vagrant@rhel8-box user-1000.slice]$ ls
cgroup.clone_children  notify_on_release  pids.events  session-1.scope  tasks
cgroup.procs           pids.current       pids.max     session-3.scope  user@1000.service
[vagrant@rhel8-box user-1000.slice]$
```

- **here we can see `session-3.scope` is created**

```
[vagrant@rhel8-box ~]$ loginctl
SESSION  UID USER    SEAT TTY
      1 1000 vagrant
      3 1000 vagrant

2 sessions listed.
[vagrant@rhel8-box ~]$

[vagrant@rhel8-box ~]$
```
- **we have set the hard limit as 10, lets verify it**
```
[vagrant@rhel8-box session-1.scope]$ pwd
/sys/fs/cgroup/pids/user.slice/user-1000.slice/session-1.scope
[vagrant@rhel8-box session-1.scope]$
[vagrant@rhel8-box session-1.scope]$ ls
cgroup.clone_children  cgroup.procs  notify_on_release  pids.current  pids.events  pids.max  tasks
[vagrant@rhel8-box session-1.scope]$
[vagrant@rhel8-box session-1.scope]$ cat pids.max
10 <------------------------------------------------- its added here
[vagrant@rhel8-box session-1.scope]$
```

- **check all the current jobs**

```
[vagrant@rhel8-box session-1.scope]$ cat pids.current
4
[vagrant@rhel8-box session-1.scope]$
```

## Setting the memory limit

```
# systemctl edit session-1.scope
[Scope]
TasksMax=10
MemoryLimit=1G
```

```
# systemctl status session-1.scope
● session-1.scope - Session 1 of user vagrant
   Loaded: loaded (/run/systemd/transient/session-1.scope; transient)
Transient: yes
  Drop-In: /etc/systemd/system/session-1.scope.d
           └─override.conf
   Active: active (running) since Wed 2022-05-04 14:02:47 UTC; 20min ago
    Tasks: 8 (limit: 10)
   Memory: 12.3M (limit: 1.0G) <------------------------------------------ 1 G is set here
   CGroup: /user.slice/user-1000.slice/session-1.scope
           ├─2382 sshd: vagrant [priv]
           ├─2396 sshd: vagrant@pts/0
           ├─2397 -bash
           ├─3993 sudo su
           ├─3995 su
           ├─3996 bash
           ├─4133 systemctl status session-1.scope
           └─4134 systemctl status session-1.scope
```

