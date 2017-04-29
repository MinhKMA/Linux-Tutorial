## Shared storage on the network with iSCSI
Many ways to share storage on a network exist. The iSCSI protocol defines a way to see a remote blocks device as a local disk. A remote device on the network is called iSCSI Target, a client which connects to iSCSI Target is called iSCSI Initiator.

### iSCSI Target Setup
Install admin tools first, configure target to persistantly start at boot time and then start it
```
# yum -y install targetcli
# systemctl enable target
# systemctl start target
```
To start using ``targetcli``, run it and to get a layout of the tree interface, run ls
```
# targetcli
targetcli shell version 2.1.fb37
Copyright 2011-2013 by Datera, Inc and others.
For help on commands, type 'help'.

/> ls
o- / .............................................................................................................. [...]
  o- backstores ................................................................................................... [...]
  | o- block ....................................................................................... [Storage Objects: 0]
  | o- fileio ...................................................................................... [Storage Objects: 0]
  | o- pscsi ....................................................................................... [Storage Objects: 0]
  | o- ramdisk ..................................................................................... [Storage Objects: 0]
  o- iscsi ................................................................................................. [Targets: 0]
  o- loopback .............................................................................................. [Targets: 0]
/>
```
#### Create a Backstore
Backstores enable support for different methods of storing an object on the local machine. Creating a storage object defines the resources the backstore will use. The supported backstores are: block devices, files, pscsi and ramdisks. Block devices are in our case.
```
/> /backstores/block create name=block_storage dev=/dev/sdb1
Generating a wwn serial.
Created block storage object block_backend using /dev/sdb1.
```
#### Create an iSCSI Target
Create an iSCSI target using a specified name
```
/> iscsi/ create iqn.2015-05.com.noverit.caldara02:3260
Created target iqn.2015-05.com.noverit.caldara02:3260.
Created TPG 1.
```
#### Configure an iSCSI Portal
An iSCSI Portal is an object specifying the IP address and port where the iSCSI target listen to incoming connections
```
/> /iscsi/iqn.2015-05.com.noverit.caldara02:3260/tpg1/portals/ create
Using default IP port 3260
Binding to INADDR_ANY (0.0.0.0)
Created network portal 0.0.0.0:3260
```
By default, a portal is created when the iSCSI Target is created listening on all IP addresses (0.0.0.0) and the default iSCSI port 3260. Make sure that the 3260 is not used by another application, else specify a different port.

#### Configure Access List
Create an Access List for each initiator that will be connecting to the target. This enforces authentication when that initiator connects, allowing only LUNs to be exposed to each initiator. Usually each initator has exclusive access to a LUN. All initiators have unique identifying names IQN. The initiator's unique name IQN must be known to configure ACLs. For open-iscsi initiators, this can be found in the ``/etc/iscsi/initiatorname.iscsi`` file.
```
# cat /etc/iscsi/initiatorname.iscsi
InitiatorName=iqn.1994-05.com.redhat:2268c31791
```
If required, use this IQN to enforce authentication by creating the ACLs.

#### Configure the LUNs
A Logical Unit Number (LUN) is a number used to identify a logical unit, which is a device addressed by the standard SCSI protocol or Storage Area Network protocols which encapsulate SCSI, such as Fibre Channel or iSCSI itself. 
To configure LUNs, create LUNs of already created storage objects.
```
/> /iscsi/iqn.2015-05.com.noverit.caldara02:3260/tpg1/luns/ create /backstores/block/block_storage
Created LUN 0.
```
At the end of configuration, the iSCSI target envinronment should look like the following
```
/> ls
o- / ........................................................................................................... [...]
  o- backstores ................................................................................................ [...]
  | o- block .................................................................................... [Storage Objects: 2]
  | | o- ana-storage ...................................................... [/dev/sdb1 (20.0GiB) write-thru activated]
  | | o- oracle-storage .................................................. [/dev/sdb2 (120.0GiB) write-thru activated]
  | o- fileio ................................................................................... [Storage Objects: 0]
  | o- pscsi .................................................................................... [Storage Objects: 0]
  | o- ramdisk .................................................................................. [Storage Objects: 0]
  o- iscsi .............................................................................................. [Targets: 1]
  | o- iqn.2015-05.com.noverit.caldara02:3260 .............................................................. [TPGs: 1]
  |   o- tpg1 .................................................................................... [gen-acls, no-auth]
  |     o- acls ............................................................................................ [ACLs: 0]
  |     o- luns ............................................................................................ [LUNs: 2]
  |     | o- lun0 .................................................................... [block/ana-storage (/dev/sdb1)]
  |     | o- lun1 ................................................................. [block/oracle-storage (/dev/sdb2)]
  |     o- portals ...................................................................................... [Portals: 1]
  |       o- 10.10.10.98:3260 ................................................................................... [OK]
  o- loopback ........................................................................................... [Targets: 0]
/>
/> exit
Global pref auto_save_on_exit=true
Last 10 configs saved in /etc/target/backup.
Configuration saved to /etc/target/saveconfig.json
```
The ``/etc/target/saveconfig.json`` file contains the above configuration.

Restart the target service 
```
# service target restart
Redirecting to /bin/systemctl restart  target.service
```
### iSCSI Initiator Setup
After configuring the iSCSI on the target machine, move to setup the iSCSI initiator machine.
Install admin tools first

```
# yum -y install iscsi-initiator-utils
```
The iSCSI initiator is composed by two services, iscsi and iscsid, start both and enable to start at system startup
```
# service iscsid start
# service iscsi start
# service iscsid status
# service iscsi status
# chkconfig iscsi on
# chkconfig iscsid on
# chkconfig --list | grep iscsi
iscsi           0:off   1:off   2:off   3:on    4:on    5:on    6:off
iscsid          0:off   1:off   2:on    3:on    4:on    5:on    6:off
```

To connect the target, first discover the published iSCSI resouces and then login
```
# iscsiadm --mode discovery --type sendtargets --portal caldara02:3260 --discover
10.10.10.98:3260,1 iqn.2015-05.com.noverit.caldara02:3260
# iscsiadm --mode node --targetname iqn.2015-05.com.noverit.caldara02:3260 --portal caldara02:3260 --login
Logging in to [iface: default, target: iqn.2015-05.com.noverit.caldara02:3260, portal: 10.10.10.98,3260] (multiple)
Login to [iface: default, target: iqn.2015-05.com.noverit.caldara02:3260, portal: 10.10.10.98,3260] successful.
#
```
Since no authentication has been set, no user and password are required.
Check the storage block devices.
```
[root@caldara01 ~]# lsblk
NAME                      MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda                         8:0    0 232.9G  0 disk
├─sda1                      8:1    0   500M  0 part /boot
├─sda2                      8:2    0  73.4G  0 part
│ ├─os-swap               253:0    0   3.9G  0 lvm  [SWAP]
│ ├─os-root               253:1    0    50G  0 lvm  /
│ └─os-data               253:2    0 178.5G  0 lvm  /data
└─sda3                      8:3    0   159G  0 part
  └─os-data               253:2    0 178.5G  0 lvm  /data
sdc                         8:32   0    20G  0 disk
sdd                         8:48   0   120G  0 disk
```
The two disks ``/dev/sdc`` and ``/dev/sdd`` are the remote iSCSI block devices exported by the target. They are seen as local block devices in the initiator machine. The disks can be used as standard local disks commands and configurations, including ``fdisk``, ``mkfs``, ``e2label``, etc.

```
# e2label /dev/sdc ANA
# e2label /dev/sdd ORACLE
# mkdir /ana
# mkdir /oracle
# mount -L ANA /ana
# mount -L ORACLE /oracle
# df -h
Filesystem           Size  Used Avail Use% Mounted on
/dev/mapper/os-root   50G  2.8G   48G   6% /
devtmpfs             3.8G     0  3.8G   0% /dev
tmpfs                3.8G     0  3.8G   0% /dev/shm
tmpfs                3.8G  370M  3.4G  10% /run
tmpfs                3.8G     0  3.8G   0% /sys/fs/cgroup
/dev/mapper/os-data  179G   22G  158G  12% /data
/dev/sda1            497M  228M  270M  46% /boot
/dev/sdc              20G   45M   19G   1% /ana
/dev/sdd             118G   60M  112G   1% /oracle
```
To disconnect the remote devices, umount and logout
```
# umount /ana
# umount /oracle
#
#  iscsiadm --mode node --targetname iqn.2015-05.com.noverit.caldara02:3260 --portal 10.10.10.98 --logout
Logging out of session [sid: 10, target: iqn.2015-05.com.noverit.caldara02:3260, portal: 10.10.10.98,3260]
Logout of [sid: 10, target: iqn.2015-05.com.noverit.caldara02:3260, portal: 10.10.10.98,3260] successful.
#
```

Stop and then disable the services at startup, if required
```
# service iscsid status
iscsid (pid  1184) is running...
# service iscsi status
No active sessions
# service iscsid stop
Stopping iscsid:                      [  OK  ]
# service iscsi stop
Stopping iscsi:                       [  OK  ]
# chkconfig iscsid off
# chkconfig iscsi off
# chkconfig --list | grep iscsi
iscsi           0:off   1:off   2:off   3:off   4:off   5:off   6:off
iscsid          0:off   1:off   2:off   3:off   4:off   5:off   6:off
```


