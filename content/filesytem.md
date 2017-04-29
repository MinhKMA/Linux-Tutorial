### Filesystem Structure
On many systems, including Linux, the **filesystem** is structured like a tree. The tree is usually portrayed as inverted, and starts at what is most often called the **root** directory, which marks the beginning of the hierarchical filesystem and is also denoted by **/**.

The Filesystem Hierarchy Standard (**FHS**) grew out of historical standards from early versions of UNIX. The FHS provides Linux developers and system administrators with a standard directory structure for the filesystem, which provides consistency between systems and distributions. Linux supports various filesystem types created for Linux, along with compatible filesystems from other operating systems. Many older, legacy filesystems are supported. Some examples of filesystem types that Linux supports are:

1. **ext3**, **ext4**, **btrfs**, **xfs** (native Linux filesystems)
2. **vfat**, **ntfs**, **hfs** (filesystems from other operating systems)

Each filesystem resides on a hard disk **partition**. Partitions help to organize the contents of disks according to the kind of data contained and how it is used. For example, important programs required to run the system are often kept on a separate partition than the one that contains files owned by regular users. In addition, temporary files created and destroyed during the normal operation of Linux are often located on a separate partition; in this way, using all available space on a particular partition may not fatally affect the normal operation of the system.

Before you can start using a filesystem, you need to mount it to the filesystem tree at a **mountpoint**. This is simply a directory (which may or may not be empty) where the filesystem is to be attached (mounted). Sometimes you may need to create the directory if it doesn't already exist. If you mount a filesystem on a non-empty directory, the former contents of that directory are covered-up and not accessible until the filesystem is unmounted. Thus mount points are usually empty directories.

The ``mount`` command is used to attach a filesystem somewhere within the filesystem tree. Arguments include the device node and mount point.
```
$ mount /dev/sda5 /mnt
```
This will attach the filesystem contained in the disk partition associated with the ``/dev/sda5`` device node, into the filesystem tree at the ``/mnt`` mount point. Note that unless the system is otherwise configured only the root user has permission to run mount. If you want it to be automatically available every time the system starts up, you need to edit the file ``/etc/fstab`` accordingly. The name is short for Filesystem Table. Looking at this file will show you the configuration of all pre-configured filesystems.

The ``umount`` command is used to detach the filesystem from the mount point.
```
$ umount /mnt
```

The command ``df -Th`` (it stands for disk-free) will display information about mounted filesystems including type and usage statistics about currently used and available space.

```
# df -Th
Filesystem                 Type      Size  Used Avail Use% Mounted on
/dev/mapper/os-root        xfs        50G  2.0G   48G   4% /
devtmpfs                   devtmpfs  1.8G     0  1.8G   0% /dev
tmpfs                      tmpfs     1.9G  4.0K  1.9G   1% /dev/shm
tmpfs                      tmpfs     1.9G  8.6M  1.8G   1% /run
tmpfs                      tmpfs     1.9G     0  1.9G   0% /sys/fs/cgroup
/dev/mapper/swift01-zone01 xfs        49G   33M   49G   1% /srv/node/z1d1
/dev/mapper/swift02-zone02 xfs        49G   33M   49G   1% /srv/node/z2d1
/dev/sda1                  xfs       497M  167M  331M  34% /boot
/dev/mapper/os-data        xfs        20G  261M   20G   2% /data
```

### The home directories
In any UNIX system, each user has his own home directory, usually placed under ``/home``. The ``/root`` directory on modern Linux systems is no more than the root user's home directory. The ``/home`` directory is often mounted as a separate filesystem on its own partition, or even exported remotely on a network through NFS.

### The binary directories
The ``/bin`` directory contains executable binaries, essential commands used in single-user mode, and essential commands required by all system users, such as ``ps``, ``ls``, ``cp``. Commands that are not essential for the system in single-user mode are placed in the ``/usr/bin`` directory, while the ``/sbin`` directory is used for essential binaries related to system administration, such as ``ifconfig`` and ``shutdown``. There is also a ``/usr/sbin`` directory for less essential system administration programs. All the binary directories are under the root partition. Sometimes ``/usr`` is a separate filesystem that may not be available in single-user mode. This was why essential commands were separated from non-essential commands. However, in some of the most modern Linux systems this distinction is considered obsolete, and ``/usr/bin`` and ``/bin`` are actually just linked together as are ``/usr/sbin`` and ``/sbin``.

### The device directory
The ``/dev`` directory contains device nodes, a type of pseudo-file used by most hardware and software devices, except for network devices. This directory is empty on the disk partition when it is not mounted but it contains entries which are created by the ``udev`` system, which creates and manages device nodes on Linux, creating them dynamically when devices are found. The ``/dev`` directory contains items such as:
```
/dev/sda1
/dev/lp1
/dev/dvd1
```

### The variable directory
The ``/var`` directory contains files that are expected to change in size and content as the system is running (var stands for variable) such as the entries in the following directories:

* System log files: ``/var/log``
* Packages files: ``/var/lib``
* Print queues: ``/var/spool``
* Temp files: ``/var/tmp``
* FTP home directory: ``/var/ftp``
* Web Server directory: ``/var/www``

The ``/var`` directory may be put in its own partition so that growth of the files can be accommodated and the file sizes do not fatally affect the system.

### The system configuration directory
The ``/etc`` directory is the home for system configuration files. It contains no binary programs, although there are some executable scripts. For example, the file ``resolv.conf`` tells the system where to go on the network to obtain host name to IP address mappings (DNS). Files like ``passwd``, ``shadow`` and ``group`` for managing user accounts are found in the ``/etc`` directory. System run level scripts are found in subdirectories of ``/etc``. For example, ``/etc/rc2.d`` contains links to scripts for entering and leaving run level 2. Some Linux distributions extend the contents of ``/etc``. For example, **Red Hat** adds the ``/etc/sysconfig`` subdirectory that contains more configuration files.

### The boot directory
The ``/boot`` directory contains the few essential files needed to boot the system. For every alternative kernel installed on the system there are four files:

* ``vmlinuz`` is the compressed Linux kernel, required for booting
* ``initramfs`` is the initial ram filesystem, required for booting
* ``config is`` the kernel configuration file, only used for debugging
* ``System.map`` contains the kernel symbol table, only used for debugging

Each of these files has a kernel version appended to its name.

### The libraries directory
The ``/lib`` contains libraries (common code shared by applications and needed for them to run) for the essential programs in ``/bin`` and ``/sbin`` folders. Most of these are what are known as dynamically loaded libraries (also known as shared libraries or Shared Objects (SO)). On some Linux distributions there exists a ``/lib64`` directory containing 64-bit libraries, while ``/lib`` contains 32-bit versions. Kernel modules (kernel code, often device drivers, that can be loaded and unloaded without re-starting the system) are located in ``/lib/modules/``.

### Additional directories

|Directory|Usage|
|---------|-----|
| /opt | Optional application software packages |
| /sys | Virtual pseudo-filesystem giving information about the system and the hardware. Can be used to alter system parameters and for debugging purposes. |
| /srv | Site-specific data served up by the system. Seldom used. |
| /tmp | Temporary files; on some distributions these files are erased across a reboot |
| /media | It is typically located where removable media, such as CDs, DVDs and USB drives are mounted. Unless configuration prohibits it, Linux automatically mounts the removable media in this directory when they are detected. |
| /usr | Multi-user applications, utilities and data |
| /usr/include | Header files used to compile applications |
| /usr/lib | Libraries for binary programs |
| /usr/lib64 | 64bit Libraries for binary programs |
| /usr/share | Shared data used by applications, generally architecture-independent |
| /usr/src | Source code, usually for the Linux kernel |
| /usr/local | Data and programs specific to the local machine. |

### File System Table
for details on the file system table, i.e. the ``/etc/fstab`` file, please see [fstab (Italian)](https://wiki.archlinux.org/index.php/Fstab_%28Italiano%29#Dischi_esterni)

