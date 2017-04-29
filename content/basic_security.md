## Linux basic security
By default, Linux has several account types in order to isolate processes and workloads:

1. **root**
2. **system**
2. **normal**
3. **network**

For a safe environment, it is advised to grant the minimum privileges possible and necessary to accounts, and remove inactive accounts. The ``last`` command, which shows the last time each user logged into the system, can be used to help identify potentially inactive accounts which are candidates for system removal.
```
# last
adriano  pts/4        10.10.10.113     Thu Feb 19 16:50   still logged in
mina     pts/2        10.10.10.113     Thu Feb 19 16:39   still logged in
root     pts/1        10.10.10.113     Thu Feb 19 16:25 - 16:25  (00:00)
root     pts/0        10.10.10.113     Thu Feb 19 15:42   still logged in
adriano  pts/3        10.10.10.246     Wed Feb 18 17:53 - 18:44  (00:51)
root     pts/2        10.10.10.99      Wed Feb 18 17:14 - 18:44  (01:30)
adriano  pts/1        10.10.10.246     Wed Feb 18 16:57 - 19:19  (02:22)
root     pts/0        10.10.10.246     Wed Feb 18 16:25 - 19:19  (02:53)
root     pts/0        10.10.10.246     Tue Feb 17 13:29 - 19:29  (06:00)
reboot   system boot  3.10.0-123.20.1. Tue Feb 17 13:28 - 17:20 (2+03:51)
```

The **root** account is the most privileged account on a Linux/UNIX system. This account has the ability to carry out all facets of system administration, including adding accounts, changing user passwords, examining log files, installing software, etc. 

A regular account user can perform some operations requiring special permissions; however, the system configuration must allow such abilities to be exercised. Running a network client or sharing a file over the network are operations that do not require a root account.

In Linux you can use either ``su`` or ``sudo`` commands to temporarily grant root access to a normal user; these methods are actually quite different. When using the ``su`` command:

* to elevate the privilege, you need to enter the root password. Giving the root password to a normal user should never, ever be done
* once a user elevates to the root account, the normal user can do anything that the root user can do for as long as the user wants, without being asked again for a password
* there are limited logging features

When using the ``sudo`` command:

* you need to enter the user’s password and not the root password
* what the user is allowed to do can be precisely controlled and limited; by default the user will either always have to keep giving their password to do further operations with ``sudo``, or can avoid doing so for a configurable time interval
* detailed logging features are available

### The sudo command
Granting privileges using the ``sudo`` command is less dangerous than ``su`` and it should be preferred. By default, ``sudo`` must be enabled on a per-user basis. However, some distributions (such as Ubuntu) enable it by default for at least one main user, or give this as an installation option. To execute just one command with root privilege type ``sudo <command>``. When the command is complete you will return to being a normal unprivileged user. The ``sudo`` configuration files are stored in the ``/etc/sudoers`` file and in the ``/etc/sudoers.d/`` directory. By default, that directory is empty.

The ``sudo`` command has the ability to keep track of unsuccessful attempts at gaining root access. An authentication failure message would appear in the ``/var/log/secure`` log file  when trying to execute sudo bash without successfully authenticating the user

```
# tail /var/log/secure
authentication failure; logname=op uid=0 euid=0 tty=/dev/pts/6 ruser=op rhost= user=op
conversation failed
auth could not identify password for [op]
op : 1 incorrect password attempt ;
TTY=pts/6 ; PWD=/var/log ; USER=root ; COMMAND=/bin/bash
```

Whenever the ``sudo`` command is invoked, a trigger will look at ``/etc/sudoers`` and the files in ``/etc/sudoers.d`` to determine if the user has the right to use ``sudo`` and what the scope of their privilege is. Unknown user requests and requests to do operations not allowed to the user even with ``sudo`` are reported. You can edit the sudoers file by using the ``visudo`` command, which ensures that only one person is editing the file at a time, has the proper permissions, and refuses to write out the file and exit if there is an error in the changes made.

The basic structure of an entry is:
> who where = (as_whom) what

To create a normal user account and give it sudo access, login as root user and edit the ``/etc/sudoers`` file with the ``visudo`` command. Find the lines in the file that grant ``sudo`` access to users in the group ``wheel`` when enabled.
```
## Allows people in group wheel to run all commands
# %wheel        ALL=(ALL)       ALL
```
Remove the comment character at the start of the second line. This enables the configuration option. Save your changes. Add the user you created to the ``wheel`` group.
```
# usermod -aG wheel adriano
# su adriano -
$ groups
adriano wheel
$ sudo whoami
We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for adriano:
root
```
If sudo is configured correctly the last line value will be ``root``.

Some Linux distributions prefer you add a file in the directory ``/etc/sudoers.d`` with a name the same as the user. This file contains the individual user's sudo configuration, and one should leave the master configuration file untouched except for changes that affect all users.

### The process isolation
Linux is considered to be more secure than many other operating systems because processes are naturally isolated from each other. One process normally cannot access the resources of another process, even when that process is running with the same user privileges. Additional security mechanisms that have been recently introduced in order to make risks even smaller are:

1. **Control Groups**: allows system administrators to group processes and associate finite resources to each group (**cgroup**).
2. **Linux Containers**: makes it possible to run multiple isolated Linux systems containers on a single system.
3. **Virtualization**: hardware is emulated in such a way that not only processes can be isolated, but entire systems are run simultaneously as isolated and insulated guests (**virtual machines**) on one physical host.

### Password encryption
Protecting passwords has become a crucial element of security. Most Linux distributions rely on a modern password encryption algorithm called SHA-512 (Secure Hashing Algorithm 512 bits), developed by the U.S. National Security Agency (NSA) to encrypt passwords. The SHA-512 algorithm is widely used for security applications and protocols. These security applications and protocols include TLS, SSL, PHP, SSH, S/MIME and IPSec. SHA-512 is one of the most tested hashing algorithms.

### Password aging
The password aging is a method to ensure that users get prompts that remind them to create a new password after a specific period. This can ensure that passwords, if cracked, will only be usable for a limited amount of time. This feature is implemented using the ``chage`` command, which configures the password expiry information for a user.
```
# chage --list adriano
Last password change                                    : Feb 18, 2015
Password expires                                        : never
Password inactive                                       : never
Account expires                                         : never
Minimum number of days between password change          : 0
Maximum number of days between password change          : 99999
Number of days of warning before password expires       : 7
```

### Public/Private Keys for Authentication
Using encrypted keys for authentication offers two main benefits. Firstly, it is convenient as you no longer need to enter a password if you use public/private keys. Secondly, once public/private key pair authentication has been set up on the server, you can disable password authentication completely meaning that without an authorized key you can't gain access.

Create a private key for client and a public key for server to do it
```
# ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.

# cd /root/.ssh
# ll
total 8
-rw------- 1 root root    0 May 30 11:17 authorized_keys
-rw------- 1 root root 1675 May 30 11:17 id_rsa
-rw-r--r-- 1 root root  396 May 30 11:17 id_rsa.pub
-rw-r--r-- 1 root root    0 May 30 11:07 known_hosts
# chmod 700 ~/.ssh
# chmod 600 ~/.ssh/id_rsa
```

This will create two files in your hidden ssh directory called: ``id_rsa`` and ``id_rsa.pub`` The first is your private key and the other is your public key. Install the public key to the authorized keys list and then remove it from the server
```
# cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
# rm -rf ~/.ssh/id_rsa.pub
```
Please, note that the same public key can be installed to many servers, just copy it on that server and install to the authorized keys list.

Copy the private key on the client that you will use to connect to the server and then remove it from the server
```
# scp ~/.ssh/id_rsa root@clientmachine:root/.ssh/
# rm -rf ~/.ssh/id_rsa
```

On Linux and Unix client, use the private key to login to the server
```
# ssh -i ~/.ssh/id_rsa root@servermachine
```

On Windows client, use the puttygen tool to make the key in a suitable format and use the Putty application to login to the server. Please, note that each user that want to login must have his own key pair.
