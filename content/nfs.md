## Network Filesystem
Using **NFS** (the Network File System) is one of the methods used for sharing data across physical systems. Many system administrators mount remote users' home directories on a server in order to give them access to the same files and configuration files across multiple client systems. This allows the users to log in to different machines yet still have access to the same files and resources.

On a generic Linux distribution, the NFS server daemon is typically started with the command ``service nfs start``. The file ``/etc/exports`` contains the directories and permissions that a host is willing to share with other systems over NFS. An entry in this file may look like ``/shared *(rw)``. This entry allows the directory ``/shared`` to be mounted using NFS with read and write (rw) permissions and shared with other hosts in the same domain. After modifying the ``/etc/exports`` file, you can use the ``exportfs -av`` command to notify Linux about the directories you are allowing to be remotely mounted using NFS.

On the client machine, if it is desired to have the remote filesystem mounted automatically upon system boot, the ``/etc/fstab`` file is modified to accomplish this. For example, an entry in the client's ``/etc/fsta``b file might look like ``<servername>:/shared /mnt/nfs/shared nfs defaults 0 0``. You can also mount the remote filesystem without a reboot or as a one-time mount by directly using the ``mount`` command. If ``/etc/fstab`` is not modified, this remote mount will not be present the next time the system is restarted.

On RedHat based distributions (CentOS-7) NFS server
```
# yum install -y nfs-utils
# mkdir /var/shared
```
Add an entry into the ``/etc/exports`` file
```
# vi /etc/exports
# /var/shared 10.10.10.0/24(no_root_squash,no_all_squash,rw,sync)
```
Where:
* ``/var/shared`` is the shared folder
* ``10.10.10.0/24`` is IP address range of clients
* ``rw`` is the permission to shared folder
* ``sync`` synchronizes shared folder
* ``root_squash`` disable the root privilege
* ``no_root_squash`` enables the root privilege
* ``no_all_squash`` enables the user’s authority

The ``no_root_squash`` option leaves root users on NFS clients to write files as root user on the NFS server. Default is ``root_squash``.

```
# systemctl start rpcbind
# systemctl start nfs-server
# systemctl enable rpcbind
# systemctl enable nfs-server
# systemctl status rpcbind
# systemctl status nfs-server
```

On the client machine mount the shared folder to a local folder
```
# mkdir -p /mnt/nfs
# mount 10.10.10.97:/var/shared /mnt/nfs
# cd /mnt/nfs
# touch filename.txt
```
**Note**: this is only for explanation. Please, do not use it in production systems. Check the NFS resources related to your distribution.

To run a NFS server behind the firewall, you should make some changes on the NFS configuration file, e.g. ``/etc/sysconfig/nfs`` on Red Hat/CentOS family, and then enable the ports on the firewall configuration. The reason is that NFS requires the ``rpcbind`` service, which dynamically assigns ports for RPC services and can cause problems for configuring firewall rules. See: [http://initrd.org/wiki/NFS_Setup](http://initrd.org/wiki/NFS_Setup)
