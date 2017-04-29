## Logical Volume Manager layout
Basically a Logical Volume Manager layout **LVM** looks like this:

* **Logical Volume(s)**: ``/dev/fileserver/share``, ``/dev/fileserver/backup``, ``/dev/fileserver/media``
* **Volume Group(s)**: ``fileserver``
* **Physical Volume(s)**: ``/dev/sdb1``, ``/dev/sdc1``, ``/dev/sdd1``, ``/dev/sdc1``

You have one or more physical volumes, and on these physical volumes you create one or more volume groups, and in each volume group you can create one or more logical volumes. If you use multiple physical volumes, each logical volume can be bigger than one of the underlying physical volumes (but of course the sum of the logical volumes cannot exceed the total space offered by the physical volumes). It is a good practice to not allocate the full space to logical volumes, but leave some space unused. That way you can enlarge one or more logical volumes later on if you feel the need for it.

With LVM, an hard drive or set of hard drives or different partitions of the same hard drive are allocated to one or more physical volumes. The physical volumes can be placed on other block devices which might span two or more disks. The physical volumes are combined into logical volumes, with the exception of the ``/boot`` partition. The ``/boot`` partition cannot be on a logical volume group because the boot loader cannot read it. If the root partition is on a logical volume, create a separate ``/boot`` partition which is not a part of a volume group. Since a physical volume cannot span over multiple drives, to span over more than one drive, create one or more physical volumes per drive.

The volume groups can be divided into logical volumes, which are assigned mount points, such as ``/home`` and root and file system types, such as **ext2** or **ext3**. When the partitions reach their full capacity, free space from the volume group can be added to the logical volume to increase the size of the partition. When a new hard drive is added to the system, it can be added to the volume group, and partitions that are logical volumes can be increased in size.

### Create a LVM layout
On my local CentOS machine, there is on additional hard drive ``/dev/sdb`` to use for LVM layout.
```
# fdisk /dev/sdb
Welcome to fdisk (util-linux 2.23.2).
Disk /dev/sdb: 250.1 GB, 250059350016 bytes, 488397168 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x0004da93

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048   488397167   244197560   8e  Linux LVM

Command (m for help):
```

The disk is already partitioned as Linux LVM, so no needs to do further. To create the LVM layout, first we need to create a physical volume
```
# pvcreate /dev/sdb1
  Physical volume "/dev/sdb1" successfully created
# pvs
  PV         VG   Fmt  Attr PSize   PFree
  /dev/sdb1       lvm2 ---  232.88g 232.88g
# pvscan
  PV
  /dev/sdb1           lvm2 [232.88 GiB]
  Total: 3 [465.28 GiB] / in use: 2 [232.39 GiB] / in no VG: 1 [232.88 GiB]
# pvdisplay
 "/dev/sdb1" is a new physical volume of "232.88 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/sdb1
  VG Name
  PV Size               232.88 GiB
  Allocatable           NO
  PE Size               0
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               ajGCMg-Y4cG-v4AD-Wxma-TaE5-zQig-XmnYAx
```

Now create the volume group
```
# vgcreate storage /dev/sdb1
  Volume group "storage" successfully created
# vgscan
  Reading all physical volumes.  This may take a while...
  Found volume group "storage" using metadata type lvm2
# vgs
  VG      #PV #LV #SN Attr   VSize   VFree
  storage   1   0   0 wz--n- 232.88g 232.88g
# vgdisplay
  --- Volume group ---
  VG Name               storage
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               232.88 GiB
  PE Size               4.00 MiB
  Total PE              59618
  Alloc PE / Size       0 / 0
  Free  PE / Size       59618 / 232.88 GiB
  VG UUID               nEcTxG-p5K6-npqD-OVeX-dRI1-aWP9-o4D1Z1

```
Now, everything is ready to create the logical volumes from the volume group
```
# lvcreate -L 20G -n db-area storage
  Logical volume "db-area" created.
# lvcreate -L 10G -n users-area storage
  Logical volume "users-area" created.
# lvcreate -L 60G -n staging-area storage
  Logical volume "staging-area" created.
# lvcreate -l 100%FREE -n spare storage
  Logical volume "spare" created.
# lvs
  LV           VG      Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  db-area      storage -wi-a-----  20.00g
  spare        storage -wi-a----- 142.88g
  staging-area storage -wi-a-----  60.00g
  users-area   storage -wi-a-----  10.00g
# lvscan
  ACTIVE            '/dev/storage/db-area' [20.00 GiB] inherit
  ACTIVE            '/dev/storage/users-area' [10.00 GiB] inherit
  ACTIVE            '/dev/storage/staging-area' [60.00 GiB] inherit
  ACTIVE            '/dev/storage/spare' [142.88 GiB] inherit
```
After creating the appropriate filesystem on the logical volumes, they become ready to use for the storage purpose
```
# mkfs.ext4 /dev/storage/db-area
# mkfs.ext4 /dev/storage/users-area
# mkfs.ext4 /dev/storage/staging-area
# mkfs.ext4 /dev/storage/spare

# mkdir /db
# mount /dev/storage/db-area /db
# mkdir /users
# mount /dev/storage/users-area /users
# mkdir /staging
# mount /dev/storage/staging-area /staging

```


### Extend a LVM layout
On the local CentOS machine, there are 2 hard drive ``/dev/sda`` and ``/dev/sdb``. The ``/dev/sda`` is partioned as follow
```
# fdisk -l /dev/sda

Disk /dev/sda: 250.1 GB, 250059350016 bytes, 488397168 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x000b78bc

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048     1026047      512000   83  Linux
/dev/sda2         1026048   155004927    76989440   8e  Linux LVM
/dev/sda3       155004928   488397167   166696120   83  Linux LVM
```

The ``/dev/sda1`` is for the ``/boot`` partition and is not into LVM layout. Both ``/dev/sda2`` and ``/dev/sda3`` partitions are part of the LVM layout. Note that both the partitions are part of the same physical disk. This is not so common in production but is possible to have. More common is the case of partitions belonging to different physical disks.
```
# lvmdiskscan
  /dev/os/swap [       3.89 GiB]
  /dev/sda1    [     500.00 MiB]
  /dev/os/root [      50.00 GiB]
  /dev/sda2    [      73.42 GiB] LVM physical volume
  /dev/os/data [     178.50 GiB]
  /dev/sda3    [     158.97 GiB] LVM physical volume
  /dev/sdb1    [     232.88 GiB]
  3 disks
  2 partitions
  0 LVM physical volume whole disks
  2 LVM physical volumes

# pvs
  PV         VG   Fmt  Attr PSize   PFree
  /dev/sda2  os   lvm2 a--   73.42g    0
  /dev/sda3  os   lvm2 a--  158.97g    0

# vgs
  VG   #PV #LV #SN Attr   VSize   VFree
  os     2   3   0 wz--n- 232.39g    0

# lvs
  LV   VG   Attr       LSize   Pool Origin Data%  Move Log Cpy%Sync Convert
  data os   -wi-ao---- 178.50g
  root os   -wi-ao----  50.00g
  swap os   -wi-ao----   3.89g
```
The two partitons are seen as two LVM physical volumes: ``/dev/sda2`` and ``/dev/sda3``. The two phisical volumes are part of the same volume group called ``os``. On top of this volume group there are three logical volumes: ``/root``, ``/data`` and ``/swap``.

We want to increase the space of the LVM layout with a new partition belonging to the second hard drive ``/dev/sdb``. The hard drive is partitioned as follow:
```
# fdisk -l /dev/sdb

Disk /dev/sdb: 250.1 GB, 250059350016 bytes, 488397168 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x0004da93

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048   488397167   244197560   83  Linux

```

The partition ``/dev/sdb1`` is Linux type. Change the partition type to Linux LVM
```
# fdisk /dev/sdb
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): p

Disk /dev/sdb: 250.1 GB, 250059350016 bytes, 488397168 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x0004da93

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048   488397167   244197560   83  Linux

Command (m for help): t
Selected partition 1
Hex code (type L to list all codes): 8e
Changed type of partition 'Linux' to 'Linux LVM'

Command (m for help): p

Disk /dev/sdb: 250.1 GB, 250059350016 bytes, 488397168 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x0004da93

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048   488397167   244197560   8e  Linux LVM

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.

WARNING: Re-reading the partition table failed with error 16: Device or resource busy.
The kernel still uses the old table. The new table will be used at
the next reboot or after you run partprobe(8) or kpartx(8)
Syncing disks.
```
A warning message which basically means in order to use the new table with the changes a system reboot is required. As workaround, run the ``partprobe -s`` to rescan the partitions.

```
# partprobe -s
/dev/sda: msdos partitions 1 2 3
/dev/sdb: msdos partitions 1
```

Create a new physical volume from the new partition
```
# pvcreate /dev/sdb1
WARNING: xfs signature detected on /dev/sdb1 at offset 0. Wipe it? [y/n] y
  Wiping xfs signature on /dev/sdb1.
  Physical volume "/dev/sdb1" successfully created
```

Check the new physical volume just created by ``pvdisplay`` command
```
# pvdisplay
   "/dev/sdb1" is a new physical volume of "232.88 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/sdb1
  VG Name
  PV Size               232.88 GiB
  Allocatable           NO
  PE Size               0
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               qtRwhD-Pxcv-JQlD-u7xu-lNi0-CiBv-F9XUoO
```

Now extend the ``os`` volume group by adding in the new physical volume which we created earlier
```
# vgextend os /dev/sdb1
  Volume group "os" successfully extended
```

Using the ``pvscan`` command we scan all disks for physical volumes, this should confirm the new created physical volume ``/dev/sdb1`` and along with the old volumes
```
# pvscan
  PV /dev/sda2   VG os   lvm2 [73.42 GiB / 0    free]
  PV /dev/sda3   VG os   lvm2 [158.97 GiB / 0    free]
  PV /dev/sdb1   VG os   lvm2 [232.88 GiB / 232.88 GiB free]
  Total: 3 [465.28 GiB] / in use: 3 [465.28 GiB] / in no VG: 0 [0   ]
```

Next want to increase the logical volume ``/dev/os/data`` which basically means we will be taking our original logical volume ``/dev/os/data`` and extending it over the new physical volume ``/dev/sdb1`` just created.

```
# lvscan
  ACTIVE            '/dev/os/root' [50.00 GiB] inherit
  ACTIVE            '/dev/os/swap' [3.89 GiB] inherit
  ACTIVE            '/dev/os/data' [178.50 GiB] inherit
# lvextend /dev/os/data /dev/sdb1
  Extending logical volume data to 411.39 GiB
  Logical volume data successfully resized
# lvscan
  ACTIVE            '/dev/os/root' [50.00 GiB] inherit
  ACTIVE            '/dev/os/swap' [3.89 GiB] inherit
  ACTIVE            '/dev/os/data' [411.39 GiB] inherit
```

Note the size of the logical volume ``/dev/os/data`` increased from 178.50 GiB to 411.39 GiB. Howewer, the size of the ``/data`` file system is still 179G
```
# df -h
Filesystem           Size  Used Avail Use% Mounted on
/dev/mapper/os-root   50G  2.0G   48G   4% /
devtmpfs             1.8G     0  1.8G   0% /dev
tmpfs                1.9G     0  1.9G   0% /dev/shm
tmpfs                1.9G  8.6M  1.8G   1% /run
tmpfs                1.9G     0  1.9G   0% /sys/fs/cgroup
/dev/mapper/os-data  179G   33M  179G   1% /data
/dev/sda1            497M  183M  315M  37% /boot
```

There is then one final step which is to resize the file system so that it can take advantage of this additional space, this is done using the ``resize2fs`` command. In our case, the command will fail since we are using *xfs* file system. We need to use the ``xfs_growfs`` to have the same effect.
```
# xfs_growfs /dev/os/data
meta-data=/dev/mapper/os-data    isize=256    agcount=4, agsize=11698432 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=0
data     =                       bsize=4096   blocks=46793728, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=0
log      =internal               bsize=4096   blocks=22848, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 46793728 to 107842560

# df -h
Filesystem           Size  Used Avail Use% Mounted on
/dev/mapper/os-root   50G  2.0G   48G   4% /
devtmpfs             1.8G     0  1.8G   0% /dev
tmpfs                1.9G     0  1.9G   0% /dev/shm
tmpfs                1.9G  8.6M  1.8G   1% /run
tmpfs                1.9G     0  1.9G   0% /sys/fs/cgroup
/dev/mapper/os-data  412G   33M  412G   1% /data
/dev/sda1            497M  183M  315M  37% /boot
```

### Reduce the LVM layout
Unfortunally we can NOT make a XFS partition smaller online. The only way to shrink is to do a complete dump of data, make a new smaller volume group and restore the data.

```
# mkdir /dump
# mv /data/* /dump/
# umount /data
```

Remove the logical volume ``/dev/os/data``
```
# lvremove /dev/os/data
Do you really want to remove active logical volume data? [y/n]: y
  Logical volume "data" successfully removed
# lvscan
  ACTIVE            '/dev/os/root' [50.00 GiB] inherit
  ACTIVE            '/dev/os/swap' [3.89 GiB] inherit
```
Detouch the physical volume from the volume group. To accomplish this task, use the ``vgreduce`` command. This command shrinks a volume group's capacity by removing one or more physical volumes. This frees the physical volumes to be used in other volume groups or to be removed from the system.

```
# vgreduce os /dev/sdb1
  Removed "/dev/sdb1" from volume group "os"
# pvs
  PV         VG   Fmt  Attr PSize   PFree
  /dev/sda2  os   lvm2 a--   73.42g  19.53g
  /dev/sda3  os   lvm2 a--  158.97g 158.97g
  /dev/sdb1       lvm2 a--  232.88g 232.88g
```

Since we do not need anymore for the physical volume ``/dev/sdb1`` , remove it from the LVM layout
```
# pvremove /dev/sdb1
  Labels on physical volume "/dev/sdb1" successfully wiped
# pvs
  PV         VG   Fmt  Attr PSize   PFree
  /dev/sda2  os   lvm2 a--   73.42g  19.53g
  /dev/sda3  os   lvm2 a--  158.97g 158.97g
```

Recreate the ``/dev/os/data`` logical volume with the remaining space in the ``os`` volume group
```
# lvcreate -l 100%FREE -n data os
WARNING: xfs signature detected on /dev/os/data at offset 0. Wipe it? [y/n] y
  Wiping xfs signature on /dev/os/data.
  Logical volume "data" created
# lvscan
  ACTIVE            '/dev/os/root' [50.00 GiB] inherit
  ACTIVE            '/dev/os/swap' [3.89 GiB] inherit
  ACTIVE            '/dev/os/data' [178.50 GiB] inherit
```

Format the volume group just created, mount it as ``/data`` fyle system and restore the data
```
# mkfs.xfs -f /dev/os/data
meta-data=/dev/os/data           isize=256    agcount=4, agsize=11698432 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=0
data     =                       bsize=4096   blocks=46793728, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=0
log      =internal log           bsize=4096   blocks=22848, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0

# mount -a
# 
# df -Th
Filesystem          Type      Size  Used Avail Use% Mounted on
/dev/mapper/os-root xfs        50G  2.0G   48G   4% /
devtmpfs            devtmpfs  1.8G     0  1.8G   0% /dev
tmpfs               tmpfs     1.9G     0  1.9G   0% /dev/shm
tmpfs               tmpfs     1.9G  8.5M  1.8G   1% /run
tmpfs               tmpfs     1.9G     0  1.9G   0% /sys/fs/cgroup
/dev/sda1           xfs       497M  183M  315M  37% /boot
/dev/mapper/os-data xfs       179G   33M  179G   1% /data
# 
# mv /dump/* /data/
```

Finally, change the partition type of ``/dev/sdb1`` back to Linux (no LVM), format as XFS and mount it as a standard physical partition
```
# fdisk /dev/sdb
Welcome to fdisk (util-linux 2.23.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Command (m for help): p

Disk /dev/sdb: 250.1 GB, 250059350016 bytes, 488397168 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x0004da93

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048   488397167   244197560   8e  Linux LVM

Command (m for help): t
Selected partition 1
Hex code (type L to list all codes): 83
Changed type of partition 'Linux LVM' to 'Linux'

Command (m for help): p

Disk /dev/sdb: 250.1 GB, 250059350016 bytes, 488397168 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x0004da93

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048   488397167   244197560   83  Linux

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
# mkfs.xfs -f /dev/sdb1
meta-data=/dev/sdb1              isize=256    agcount=4, agsize=15262348 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=0
data     =                       bsize=4096   blocks=61049390, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=0
log      =internal log           bsize=4096   blocks=29809, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0

# mount -a
# df -Th
Filesystem          Type      Size  Used Avail Use% Mounted on
/dev/mapper/os-root xfs        50G  2.0G   48G   4% /
devtmpfs            devtmpfs  1.8G     0  1.8G   0% /dev
tmpfs               tmpfs     1.9G     0  1.9G   0% /dev/shm
tmpfs               tmpfs     1.9G  8.5M  1.8G   1% /run
tmpfs               tmpfs     1.9G     0  1.9G   0% /sys/fs/cgroup
/dev/sda1           xfs       497M  183M  315M  37% /boot
/dev/mapper/os-data xfs       179G   33M  179G   1% /data
/dev/sdb1           xfs       233G   33M  233G   1% /cinder-volumes
#
```
