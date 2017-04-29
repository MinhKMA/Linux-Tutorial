## Volume Manager, a real example
Let to see, step-by-step, a real example of Volume Manager on a CentOS 7 setup. 
On the host caldera01 there is an additional disk ``/dev/sdb`` we want to use for store a mysql database.

```
[root@caldera01 ~]# df -Th
Filesystem             Type      Size  Used Avail Use% Mounted on
/dev/mapper/os-root    xfs        50G  2.1G   48G   5% /
devtmpfs               devtmpfs  3.8G     0  3.8G   0% /dev
tmpfs                  tmpfs     3.8G     0  3.8G   0% /dev/shm
tmpfs                  tmpfs     3.8G  8.6M  3.8G   1% /run
tmpfs                  tmpfs     3.8G     0  3.8G   0% /sys/fs/cgroup
/dev/mapper/os-data    xfs       175G  256M  175G   1% /data
/dev/sda1              xfs       497M  190M  308M  39% /boot

[root@caldera01 ~]# lsblk
NAME                         MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda                            8:0    0 232.9G  0 disk
├─sda1                         8:1    0   500M  0 part /boot
└─sda2                         8:2    0 232.4G  0 part
  ├─os-swap                  253:0    0   7.8G  0 lvm  [SWAP]
  ├─os-root                  253:1    0    50G  0 lvm  /
  └─os-data                  253:2    0 174.6G  0 lvm  /data
sdb                            8:16   0 232.9G  0 disk
└─sdb1                         8:17   0 232.9G  0 part

[root@caldera01 ~]# fdisk /dev/sdb
Disk /dev/sdb: 250.1 GB, 250059350016 bytes, 488397168 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x0003b431

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048   488397167   244197560   8e  Linux LVM
```

Create a LVM layout
```
[root@caldera01 ~]# pvcreate /dev/sdb1
[root@caldera01 ~]# vgcreate vgdb /dev/sdb1
[root@caldera01 ~]# lvcreate -l 100%FREE -n lvol1 vgdb
[root@caldera01 ~]# pvs
  PV         VG   Fmt  Attr PSize   PFree
  /dev/sda2  os   lvm2 a--  232.39g    0
  /dev/sdb1  vgdb lvm2 a--  232.88g    0
[root@caldera01 ~]# vgs
  VG   #PV #LV #SN Attr   VSize   VFree
  os     1   3   0 wz--n- 232.39g    0
  vgdb   1   1   0 wz--n- 232.88g    0
[root@caldera01 ~]# lvs
  LV    VG   Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  data  os   -wi-ao---- 174.63g
  root  os   -wi-ao----  50.00g
  swap  os   -wi-ao----   7.77g
  lvol1 vgdb -wi-ao---- 232.88g
```

Make an ``ext3`` file system on the logical volume and mount the partition under a ``/db`` directory 
```
[root@caldera01 ~]# mkfs -t ext3 /dev/vgdb/lvol1
[root@caldera01 ~]# mkdir /db
[root@caldera01 ~]# mount /dev/sdb1 /db
```

Install a mysql database on the new filesystem
```
[root@caldera01 ~]# sudo rpm -Uvh http://dev.mysql.com/get/mysql-community-release-el7-5.noarch.rpm
[root@caldera01 ~]# yum install -y mysql-server
[root@caldera01 ~]#  vi /etc/my.cnf
[mysqld]
...
datadir=/db/mysql
...
[root@caldera01 ~]# systemctl start mysqld
[root@caldera01 ~]# systemctl status mysqld
[root@caldera01 ~]# systemctl enable mysqld
```

From GitHub, install a sample database with an integrated test suite, used to test your applications and database servers
```
[root@caldera01 ~]# yum install -y git
[root@caldera01 ~]# git clone https://github.com/datacharmer/test_db.git
[root@caldera01 ~]# cd /db/test_db
```

Load the test database
```
[root@caldera01 ~]# mysql < employees.sql
```

If you want to install with two large partitioned tables, alternatively, run
```
[root@caldera01 ~]# mysql < employees_partitioned.sql
```

Run the test suite
```
[root@caldera01 ~]# mysql  -t < test_employees_md5.sql
+----------------------+
| INFO                 |
+----------------------+
| TESTING INSTALLATION |
+----------------------+
+--------------+------------------+----------------------------------+
| table_name   | expected_records | expected_crc                     |
+--------------+------------------+----------------------------------+
| employees    |           300024 | 4ec56ab5ba37218d187cf6ab09ce1aa1 |
| departments  |                9 | d1af5e170d2d1591d776d5638d71fc5f |
| dept_manager |               24 | 8720e2f0853ac9096b689c14664f847e |
| dept_emp     |           331603 | ccf6fe516f990bdaa49713fc478701b7 |
| titles       |           443308 | bfa016c472df68e70a03facafa1bc0a8 |
| salaries     |          2844047 | fd220654e95aea1b169624ffe3fca934 |
+--------------+------------------+----------------------------------+
+--------------+------------------+----------------------------------+
| table_name   | found_records    | found_crc                        |
+--------------+------------------+----------------------------------+
| employees    |           300024 | 4ec56ab5ba37218d187cf6ab09ce1aa1 |
| departments  |                9 | d1af5e170d2d1591d776d5638d71fc5f |
| dept_manager |               24 | 8720e2f0853ac9096b689c14664f847e |
| dept_emp     |           331603 | ccf6fe516f990bdaa49713fc478701b7 |
| titles       |           443308 | bfa016c472df68e70a03facafa1bc0a8 |
| salaries     |          2844047 | fd220654e95aea1b169624ffe3fca934 |
+--------------+------------------+----------------------------------+
+--------------+---------------+-----------+
| table_name   | records_match | crc_match |
+--------------+---------------+-----------+
| employees    | OK            | ok        |
| departments  | OK            | ok        |
| dept_manager | OK            | ok        |
| dept_emp     | OK            | ok        |
| titles       | OK            | ok        |
| salaries     | OK            | ok        |
+--------------+---------------+-----------+
```

We want to add 2 additional LUNs via iSCSI protocol to the LVM layout. The iSCSI makes the system able to see the external LUNs as additional disks, called ``/dev/sdc`` and ``/dev/sdd``
Each additional disk is of 232.9G

```
[root@caldera01 ~]# lsblk
NAME                         MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda                            8:0    0 232.9G  0 disk
├─sda1                         8:1    0   500M  0 part /boot
└─sda2                         8:2    0 232.4G  0 part
  ├─os-swap                  253:0    0   7.8G  0 lvm  [SWAP]
  ├─os-root                  253:1    0    50G  0 lvm  /
  └─os-data                  253:2    0 174.6G  0 lvm  /data
sdb                            8:16   0 232.9G  0 disk
└─sdb1                         8:17   0 232.9G  0 part
  └─vgdb-lvol1               253:4    0 232.9G  0 lvm  /db
sdc                            8:32   0 232.9G  0 disk
sdd                            8:48   0 232.9G  0 disk
```

Now let's to extend the LV ``lvol1`` by using the 2 additional LUNs
```
[root@caldera01 ~]# pvcreate -f /dev/sdc
  Wiping iso9660 signature on /dev/sdc.
  Wiping dos signature on /dev/sdc.
  Physical volume "/dev/sdc" successfully created

[root@caldera01 ~]# pvcreate -f /dev/sdd
  Wiping dos signature on /dev/sdd.
  Physical volume "/dev/sdd" successfully created

[root@caldera01 ~]# pvscan
  PV /dev/sda2   VG os     lvm2 [232.39 GiB / 0    free]
  PV /dev/sdb1   VG vgdb   lvm2 [232.88 GiB / 0    free]
  PV /dev/sdc              lvm2 [232.89 GiB]
  PV /dev/sdd              lvm2 [232.89 GiB]

[root@caldera01 ~]# vgextend vgdb /dev/sdc /dev/sdd
  Volume group "vgdb" successfully extended

[root@caldera01 ~]# vgscan
  Reading all physical volumes.  This may take a while...
  Found volume group "os" using metadata type lvm2
  Found volume group "vgdb" using metadata type lvm2

[root@caldera01 ~]# vgdisplay vgdb
  --- Volume group ---
  VG Name               vgdb
  System ID
  Format                lvm2
  Metadata Areas        3
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               1
  Max PV                0
  Cur PV                3
  Act PV                3
  VG Size               698.64 GiB
  PE Size               4.00 MiB
  Total PE              178852
  Alloc PE / Size       59618 / 232.88 GiB
  Free  PE / Size       119234 / 465.76 GiB
  VG UUID               O557zn-CcSI-1Ec4-LryC-uqi8-B42R-pYTHKU
  
[root@caldera01 ~]# lvscan
  ACTIVE            '/dev/os/root' [50.00 GiB] inherit
  ACTIVE            '/dev/os/data' [174.63 GiB] inherit
  ACTIVE            '/dev/os/swap' [7.77 GiB] inherit
  ACTIVE            '/dev/vgdb/lvol1' [232.88 GiB] inherit

[root@caldera01 ~]# lvextend -L +400G /dev/vgdb/lvol1
  Size of logical volume vgdb/lvol1 changed from 232.88 GiB (59618 extents) to 632.88 GiB (162018 extents).
  Logical volume lvol1 successfully resized
[root@caldera01 ~]#

[root@caldera01 ~]# lvscan
  ACTIVE            '/dev/os/root' [50.00 GiB] inherit
  ACTIVE            '/dev/os/data' [174.63 GiB] inherit
  ACTIVE            '/dev/os/swap' [7.77 GiB] inherit
  ACTIVE            '/dev/vgdb/lvol1' [632.88 GiB] inherit
```

Now the LV successfully increased its size by 632G but the file system still is 232G
```
[root@caldera01 ~]# df -Th
Filesystem             Type      Size  Used Avail Use% Mounted on
/dev/mapper/os-root    xfs        50G  2.1G   48G   5% /
devtmpfs               devtmpfs  3.8G     0  3.8G   0% /dev
tmpfs                  tmpfs     3.8G     0  3.8G   0% /dev/shm
tmpfs                  tmpfs     3.8G  8.6M  3.8G   1% /run
tmpfs                  tmpfs     3.8G     0  3.8G   0% /sys/fs/cgroup
/dev/mapper/os-data    xfs       175G  256M  175G   1% /data
/dev/sda1              xfs       497M  190M  308M  39% /boot
/dev/mapper/vgdb-lvol1 ext3      230G  642M  217G   1% /db
```

Extend the file system without umount it
```
[root@caldera01 ~]# resize2fs -p /dev/vgdb/lvol1
resize2fs 1.42.9 (28-Dec-2013)
Filesystem at /dev/vgdb/lvol1 is mounted on /db; on-line resizing required
old_desc_blocks = 15, new_desc_blocks = 40
The filesystem on /dev/vgdb/lvol1 is now 165906432 blocks long.
```

Let's check our data file
```
[root@caldera01 ~]# cd /db/test_db
[root@caldera01 ~]# mysql  -t < test_employees_md5.sql

+----------------------+
| INFO                 |
+----------------------+
| TESTING INSTALLATION |
+----------------------+
+--------------+------------------+----------------------------------+
| table_name   | expected_records | expected_crc                     |
+--------------+------------------+----------------------------------+
| employees    |           300024 | 4ec56ab5ba37218d187cf6ab09ce1aa1 |
| departments  |                9 | d1af5e170d2d1591d776d5638d71fc5f |
| dept_manager |               24 | 8720e2f0853ac9096b689c14664f847e |
| dept_emp     |           331603 | ccf6fe516f990bdaa49713fc478701b7 |
| titles       |           443308 | bfa016c472df68e70a03facafa1bc0a8 |
| salaries     |          2844047 | fd220654e95aea1b169624ffe3fca934 |
+--------------+------------------+----------------------------------+
+--------------+------------------+----------------------------------+
| table_name   | found_records    | found_crc                        |
+--------------+------------------+----------------------------------+
| employees    |           300024 | 4ec56ab5ba37218d187cf6ab09ce1aa1 |
| departments  |                9 | d1af5e170d2d1591d776d5638d71fc5f |
| dept_manager |               24 | 8720e2f0853ac9096b689c14664f847e |
| dept_emp     |           331603 | ccf6fe516f990bdaa49713fc478701b7 |
| titles       |           443308 | bfa016c472df68e70a03facafa1bc0a8 |
| salaries     |          2844047 | fd220654e95aea1b169624ffe3fca934 |
+--------------+------------------+----------------------------------+
+--------------+---------------+-----------+
| table_name   | records_match | crc_match |
+--------------+---------------+-----------+
| employees    | OK            | ok        |
| departments  | OK            | ok        |
| dept_manager | OK            | ok        |
| dept_emp     | OK            | ok        |
| titles       | OK            | ok        |
| salaries     | OK            | ok        |
+--------------+---------------+-----------+
```

Our data is safe! Now we want to resize the logical volume to use only 1 LUN instead of 2 LUNs as we made before. Firt of all, shutdown any service or application that is using the logical volume we want to resize. In our case, we have to stop the MySQL database running on the volume
```
[root@caldera01 test_db]# systemctl stop mysql
[root@caldera01 test_db]# systemctl status mysql
mysqld.service - MySQL Community Server
   Loaded: loaded (/usr/lib/systemd/system/mysqld.service; enabled)
   Active: inactive (dead) since Thu 2015-10-22 12:13:28 CEST; 2min 12s ago
 Main PID: 15388 (code=exited, status=0/SUCCESS)
```

Make a backup of ALL yuor data present on the volume
```
[root@caldera01 test_db]# tar cvf data_backup.tar *
[root@caldera01 test_db]# gzip data_backup.tar
[root@caldera01 test_db]# mv data_backup.tar /data_backup_folder
```

Umount the LV
```
[root@caldera01 ~]# umount /db
```

Then check for the file-system error (Must pass in every 5 steps of file-system check if not there might be some issue with your file-system)

```
[root@caldera01 /]# e2fsck -ff /dev/vgdb/lvol1
e2fsck 1.42.9 (28-Dec-2013)
Pass 1: Checking inodes, blocks, and sizes
Pass 2: Checking directory structure
Pass 3: Checking directory connectivity
Pass 4: Checking reference counts
Pass 5: Checking group summary information
/dev/vgdb/lvol1: 403/41484288 files (10.9% non-contiguous), 2803302/165906432 blocks
```

Now let's to resize the file system. Since we want to remove the last LUN, we have to calculate the final size of the file system by summing its original size ``/dev/sdb1`` to the size of the first added LUN ``/dev/sdc``. We have ``/dev/sdb1`` Total PE 59618 + ``/dev/sdc`` Total PE 59617 = Total PE 119235. Total size, expressed in Megabyte are 4 x 119235 = 476940 MB

```
[root@caldera01 ~]# pvdisplay
--- Physical volume ---
  PV Name               /dev/sdb1
  VG Name               vgdb
  PV Size               232.88 GiB / not usable 2.18 MiB
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              59618
  Free PE               0
  Allocated PE          59618
  PV UUID               mymhL6-Ta6m-cx3V-LjKM-Zo2T-K7Xj-MI1jb3

  --- Physical volume ---
  PV Name               /dev/sdc
  VG Name               vgdb
  PV Size               232.89 GiB / not usable 7.18 MiB
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              59617
  Free PE               0
  Allocated PE          59617
  PV UUID               0dIQWO-jjaa-zgl3-1o8E-8kGR-4xjc-BxPmwO

  --- Physical volume ---
  PV Name               /dev/sdd
  VG Name               vgdb
  PV Size               232.89 GiB / not usable 7.18 MiB
  Allocatable           yes
  PE Size               4.00 MiB
  Total PE              59617
  Free PE               16834
  Allocated PE          42783
  PV UUID               Df0M7R-6KDB-TUeZ-l1Gw-2r5J-ci4c-MsyNw4
  
[root@caldera01 ~]# fsadm -e -y resize /dev/vgdb/lvol1 476940M
resize2fs 1.42.9 (28-Dec-2013)
Resizing the filesystem on /dev/mapper/vgdb-lvol1 to 122096640 (4k) blocks.
The filesystem on /dev/mapper/vgdb-lvol1 is now 122096640 blocks long.
```

Reduce the Logical Volume and remove the last LUN from the Volume Group
```
[root@caldera01 ~]# lvreduce -L 476940M /dev/vgdb/lvol1
  WARNING: Reducing active logical volume to 465.76 GiB
  THIS MAY DESTROY YOUR DATA (filesystem etc.)
Do you really want to reduce lvol1? [y/n]: y
  Size of logical volume vgdb/lvol1 changed from 632.88 GiB (162018 extents) to 465.76 GiB (119235 extents).
  Logical volume lvol1 successfully resized
  
[root@caldera01 ~]# vgreduce vgdb /dev/sdd
  Removed "/dev/sdd" from volume group "vgdb"
[root@caldera01 ~]#
```

Remove the physical volume
```
[root@caldera01 ~]# pvremove /dev/sdd
  Labels on physical volume "/dev/sdd" successfully wiped
[root@caldera01 ~]#
```

And check again the file system and mount back it
```
[root@caldera01 ~]# e2fsck -ff /dev/vgdb/lvol1
e2fsck 1.42.9 (28-Dec-2013)
Pass 1: Checking inodes, blocks, and sizes
Pass 2: Checking directory structure
Pass 3: Checking directory connectivity
Pass 4: Checking reference counts
Pass 5: Checking group summary information
/dev/vgdb/lvol1: 403/30531584 files (10.9% non-contiguous), 2116084/122096640 blocks
[root@caldera01 ~]#

[root@caldera01 ~]# mount /dev/vgdb/lvol1 /db
[root@caldera01 ~]# df -Th /db
Filesystem             Type  Size  Used Avail Use% Mounted on
/dev/mapper/vgdb-lvol1 ext3  459G  653M  439G   1% /db
[root@caldera01 ~]#
```

Start the mysql service and check for data integrity
```
[root@caldera01 ~]# systemctl start mysqld
[root@caldera01 ~]# cd /db/test_db/
[root@caldera01 test_db]# mysql  -t < test_employees_md5.sql
+----------------------+
| INFO                 |
+----------------------+
| TESTING INSTALLATION |
+----------------------+
+--------------+------------------+----------------------------------+
| table_name   | expected_records | expected_crc                     |
+--------------+------------------+----------------------------------+
| employees    |           300024 | 4ec56ab5ba37218d187cf6ab09ce1aa1 |
| departments  |                9 | d1af5e170d2d1591d776d5638d71fc5f |
| dept_manager |               24 | 8720e2f0853ac9096b689c14664f847e |
| dept_emp     |           331603 | ccf6fe516f990bdaa49713fc478701b7 |
| titles       |           443308 | bfa016c472df68e70a03facafa1bc0a8 |
| salaries     |          2844047 | fd220654e95aea1b169624ffe3fca934 |
+--------------+------------------+----------------------------------+
+--------------+------------------+----------------------------------+
| table_name   | found_records    | found_crc                        |
+--------------+------------------+----------------------------------+
| employees    |           300024 | 4ec56ab5ba37218d187cf6ab09ce1aa1 |
| departments  |                9 | d1af5e170d2d1591d776d5638d71fc5f |
| dept_manager |               24 | 8720e2f0853ac9096b689c14664f847e |
| dept_emp     |           331603 | ccf6fe516f990bdaa49713fc478701b7 |
| titles       |           443308 | bfa016c472df68e70a03facafa1bc0a8 |
| salaries     |          2844047 | fd220654e95aea1b169624ffe3fca934 |
+--------------+------------------+----------------------------------+
+--------------+---------------+-----------+
| table_name   | records_match | crc_match |
+--------------+---------------+-----------+
| employees    | OK            | ok        |
| departments  | OK            | ok        |
| dept_manager | OK            | ok        |
| dept_emp     | OK            | ok        |
| titles       | OK            | ok        |
| salaries     | OK            | ok        |
+--------------+---------------+-----------+
+------------------+
| computation_time |
+------------------+
| 00:00:14         |
+------------------+
+---------+--------+
| summary | result |
+---------+--------+
| CRC     | OK     |
| count   | OK     |
+---------+--------+

```

