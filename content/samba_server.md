## Samba server and Windows file sharing
Samba is an open source implementation of the SMB/CIFS protocol. It allows the networking of Microsoft Windows®, Linux, UNIX, and other operating systems. Samba allows a Linux/Unix server to appear as a Windows server to Windows clients.

With Samba, an administrator can do:

1. Serve directory trees and printers to Linux, UNIX, and Windows clients
2. Assist in network browsing with or without NetBIOS
3. Authenticate Windows domain logins
4. Provide WINS name server resolution

Samba is comprised of **smb**, **nmb**, and **winbind** services.

The ``smbd`` server daemon provides file sharing and printing services to Windows clients. In addition, it is responsible for user authentication, resource locking, and data sharing through the SMB protocol. The default ports on which the server listens for SMB traffic are TCP ports 139 and 445.

The ``nmbd`` server daemon understands and replies to NetBIOS name service requests produced by SMB in Windows-based systems. The default port that the server listens to for NMB traffic is UDP port 137.

The ``winbindd`` service resolves user and group information received from a server running Windows. This makes Windows user and group information understandable by Linux and UNIX platforms. This allows Windows domain users to appear and operate as Linux and UNIX users on a Linux or UNIX machine. Both ``winbindd`` and ``smbd`` are bundled with the Samba distribution, but the ``winbindd`` service is controlled separately from the ``smbd`` service.

#### Setup a Samba server
We'll setup a Samba server to make Linux file sharing available to Windows clients. Install the Samba package, enable and start the ``smbd`` and ``nmbd`` services

```
# yum install samba
# systemctl enable smb
# systemctl enable nmb
# systemctl start smb
# systemctl start nmb
```

Samba uses ``/etc/samba/smb.conf`` as its configuration file. 

```
# mv /etc/samba/smb.conf /etc/samba/smb.conf.orig
# vi /etc/samba/smb.conf

# =============== Global configuration ===============
[global]
; Windows workgroup name and server description
workgroup = WORKGROUP
server string = My SMB Server %v
; NetBIOS name as the Linux machine will appear in Windows clients
netbios name = MYSMBSERVER
; interfaces where the service is listening: localhost and ens32 interfaces
interfaces = lo ens32
; users passwords database backend and location
passdb backend = smbpasswd
smb passwd file = /etc/samba/smbpasswd
; permitted hosts to use the Samba server: localhost and all host belonging to 10.10.10.0/24 subnet
hosts allow = 127. 10.10.10.
; protocol version
max protocol = SMB3
; type of security
security = user
; no printing services
printing = bsd
printcap name = /dev/null

# =============== Shares configuration ===============
[share1]
comment = Private Documents
; path of files to share
path = /samba/admin/data
; users admitted to use the file sharing service
valid users = admin
invalid users = user2 user3
; no guest user is admitted
guest ok = no
; make the share writable as Samba make it as readonly by default
writable = yes
; make the share visible as shared folder
browsable = yes

[share2]
comment = Public Documents
path = /samba/user2/data
valid users = user2 admin
guest ok = no
writable = yes
browsable = yes

[share3]
comment = Public Documents
path = /samba/user3/data
valid users = user3 admin
guest ok = no
writable = yes
browsable = yes
```

The Samba configuration file can be checked by the ``testparm`` command
```
# testparm /etc/samba/smb.conf
Load smb config files from /etc/samba/smb.conf
rlimit_max: increasing rlimit_max (1024) to minimum Windows limit (16384)
Processing section "[homes]"
Processing section "[admin]"
Processing section "[guest]"
Loaded services file OK.
Server role: ROLE_STANDALONE
Press enter to see a dump of your service definitions
```

#### User access
More than one user can be admitted to access the same share. In the case above, the share1 is only accesible to the "admin" user. The share2 is accessible to "admin" and "user2" users but not "user3". The share3 is accessible to "admin" and "user3" to "user2".

**Note:** the connection to shares by the same Windows client needs to use the same user name. In our case, a Windows client can access all the shares above as "admin" but cannot access to share2 as "user2" AND access to share3 as "user3". If the Windows client needs to access with different users, it needs to logout from the previous user and then login again with a different user. Since Windows caches the login user, it needs to force the logout by issuing the command: ``net use * /delete`` from the Windows command shell

```
Microsoft Windows [Versione 10.0.10240]
(c) 2015 Microsoft Corporation. Tutti i diritti sono riservati.
C:\Users\Adriano>net use * /delete
Connessioni remote presenti:
                    \\10.10.10.12\IPC$
Continuando si annulleranno le connessioni.
Continuare questa operazione? (S/N) [N]: S
Esecuzione comando riuscita.
```
Samba uses different type of security. In the case above, the method is based on user level (default). With this method, each share is assigned specific users that can access it. When a user requests a connection to a share, Samba authenticates by validating the given username and password with the authorized users in the configuration file and the passwords in the password database of the Samba server.

Samba uses different database backends for storing users passwords. The simplest is store the password in a file called ``smbpasswd`` similar to the ``/etc/passwd`` file. Usually this file is located under ``/var/lib/samba/private/smbpasswd`` but location can be changed.

Add the user and set password in the Samba user database

```
# smbpasswd -a admin
New SMB password:
Retype new SMB password:
#
```
The ``pdbedit`` command lists the Samba users database

```
# pdbedit -L
admin:1000:
user1:1001:
user2:1002:
user3:1003:
```

Other security methods: domain and server level security are deprecated in latest Samba.

With smbpasswd database backend, a Samba user should exist as valid user in the Linux machine. To secure the Linux machine preventing login from Samba users, you should disable the login from these users
```
# useradd -d /samba/share user1
# usermod -s /bin/false user1
# cat /etc/passwd | grep user1
user1:x:1003:1002::/samba/share:/sbin/nologin
#
# ssh user1@localhost
user1@localhost's password:
Last login: Tue Sep 15 11:50:08 2015
This account is currently not available.
Connection to localhost closed.
#
# sftp user1@localhost
user1@localhost's password:
subsystem request failed on channel 0
Couldn't read packet: Connection reset by peer
```
Alternatively, you can leave the ssh but should chroot the user's home directory.

#### File permissions and attributes
In our example above, we are going to share Linux files and folders to Windows clients. Since Windows and Linux use different approach to file permissions and attributes, Samba will take care of mapping the two approaches.

All Linux files have read, write, and execute bits for three classifications of users: owner (u), group (g), and rest of the world (o). Windows, on the other hand, has four principal bits that it uses with any file: read-only, system, hidden, and archive:

1. Read-only. The file's contents can be read by a user but cannot be written to.
2. System. This file has a specific purpose required by the operating system.
3. Hidden. This file has been marked to be invisible to the user, unless the operating systems is explicitly set to show it.
4. Archive. This file has been touched since the last backup was performed on it.

There is no bit to specify that a file is executable since Windows identifies executable files by looking at the file extension. Windows files stored on a Linux Samba share have their own attributes that need to be preserved. Samba preserves these bits by reusing the Linux executable permission bits of the file, if it is instructed to do so. Mapping these bits, however, has a side-effect: if a Windows user stores a file in a Samba share, at Linux side, some of the executable bits are set.

The Samba options deciding the mapping
```
[share]
...
	store dos attributes = yes
	map archive = yes ;default is yes
	map system = yes  ;default is no
	map hidden = yes  ;default is no
```
The last three options map the archive, system, and hidden attributes to the owner, group, and world execute bits of the file, respectively. In the example above, the options are used on a per-share basis. Setting them globally makes them the default for all shares. The first option also makes sure that Samba does not change the Windows permission bits.

**Note:** These options can be used if the Linux file system supports the extended attributes, and those attributes are enabled, usually via the ``user_xattr`` mount option in the ``/etc/fstab`` file. Unlike _ext3_ and _ext4_, the _xfs_ file system enables the ``user_xattr`` option by default.

Samba has the ``create mask`` and the ``directory mask`` options to help with files and folders creation. The creation masks help to define the permissions a file or directory at the time it is created. On the Linux side, you can control what permissions a file or directory have when it is created. On the Windows side, you can disable the read-only, archive, system, and hidden attributes of a file as well.

```
[share]
...
	store dos attributes = yes
	map archive = yes            ;default is yes
	map system = yes             ;default is no
	map hidden = yes             ;default is no
	create mask = 0744           ;default is 0744
	directory mask = 0755        ;default is 0755
```

On the Linux side, new files and folders will look like

```
# ll /samba/share/user1
total 0
-rwxr--r-- 1 user1 samba 0 Sep 15 13:00 mydocument.txt
drwxr-xr-x 2 user1 samba 6 Sep 15 13:00 myfolder
```

It is possible force various bits with the ``force create mode`` and ``force directory mode`` options. With the ``create mask`` and ``create directory mask`` options, the administrator allow the permission bits to be set by the requested user. On the other side, the ``force create mode`` and ``force directory mode`` will force a particular bit to be set, even if it wasn’t requested by the user.

At the same time, it is possible to force the Linux user and group attributes of a file that is created on the Windows side by the ``force user`` and the ``force group`` options.

```
[share]
...
	store dos attributes = yes
	map archive = yes            ;default is yes
	map system = yes             ;default is no
	map hidden = yes             ;default is no
	create mask = 0744           ;default is 0744
	directory mask = 0755        ;default is 0755
	force create mode = 0000     ;default is 0000
	force directory mode = 0000  ;default is 0000
	force user = user1
	force group samba
```
