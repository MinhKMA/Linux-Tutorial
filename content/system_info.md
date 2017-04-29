### Linux release and system info
Linux System Administrators need to get info from the system. Here some useful commands.

Linux release and distribution
```
# cat /etc/*release
CentOS Linux release 7.0.1406 (Core)
NAME="CentOS Linux"
VERSION="7 (Core)"
ID="centos"
ID_LIKE="rhel fedora"
VERSION_ID="7"
PRETTY_NAME="CentOS Linux 7 (Core)"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:centos:centos:7"
HOME_URL="https://www.centos.org/"
BUG_REPORT_URL="https://bugs.centos.org/"
CentOS Linux release 7.0.1406 (Core)
```
Kernel version
```
# uname -r
3.10.0-123.13.2.el7.x86_64
```
Memory Info
```
# head /proc/meminfo
MemTotal:        3776748 kB
MemFree:         2230496 kB
MemAvailable:    2782088 kB
Buffers:            1452 kB
Cached:           652196 kB
SwapCached:            0 kB
Active:          1069616 kB
Inactive:         193056 kB
Active(anon):     609504 kB
Inactive(anon):     8304 kB
```
File system
```
# df -h
Filesystem         Dimens. Usati Disp. Uso% Montato su
/dev/sda1          12G     6,2G  4,9G  56%      /
/dev/small-db02    5,9G    2,6G  3,0G  46%      /db02
/dev/small-db01    5,0G    3,6G  1,2G  77%      /db01
/dev/small-db05    7,8G    1,2G  6,2G  17%      /db05
/dev/small-db03    39G     5,4G   32G  15%      /db03                      
/dev/small-db04    30G     2,5G   26G   9%      /db04
```

Count the number of CPU
```
# cat /proc/cpuinfo | grep model | uniq -c
      2 model name      : Intel(R) Core(TM)2 Duo CPU     E8500  @ 3.16GHz
```

### The proc Filesystem
The ``/proc`` filesystem contains virtual files that exist only in memory. This filesystem contains files and directories that mimic kernel structures and configuration information. It doesn't contain real files but runtime system information (e.g. system memory, devices mounted, hardware configuration, etc). Some important files in ``/proc`` are:

```
/proc/cpuinfo
/proc/interrupts
/proc/meminfo
/proc/mounts
/proc/partitions
/proc/version
/proc/<process-id-#>
/proc/sys
```
The ``/proc`` filesystem is very useful because the information it reports is gathered only as needed and never needs storage on disk.

### Hostname
The hostname identifies the machine within the domain.
```
# cat /etc/hostname
```
Set a new host name
```
# hostname NEW_NAME
```
This will set the hostname of the system to NEW_NAME. This is active right away and will remain like that until the system will be rebooted. On **Debian** based systems, use the file ``/etc/hostname`` to read the hostname of the system at boot time and set it up using the init script ``/etc/init.d/hostname.sh``. The hostname saved in the file ``/etc/hostname`` will be preserved on system reboot and will be set using the same script we used.

On **RedHat** based systems, use the ``hostnamectl`` utility to get and set the hostname.

```
# hostnamectl status
   Static hostname: caldera01
         Icon name: computer-desktop
           Chassis: desktop
        Machine ID: <machineId>
           Boot ID: <bootId>
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-123.13.2.el7.x86_64
      Architecture: x86_64
```
