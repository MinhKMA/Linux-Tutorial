### Users and Groups
Linux is a multiuser operating system where more than one user can log on at the same time. The ``who`` command lists the currently logged-on users. To identify the current user, use the ``whoami`` command.

```
# who -a
           system boot  2015-02-17 13:28
LOGIN      tty1         2015-02-17 13:28               761 id=tty1
root     + pts/0        2015-02-17 13:29   .         12379 (10.10.10.246)
           run-level 3  2015-02-17 13:29
root     + pts/1        2015-02-17 17:37   .         18762 (10.10.10.246)
```
Linux uses groups for organizing users. Groups are collections of accounts with certain shared permissions. Control of group membership is administered through the ``/etc/group`` file, which shows a list of groups and their members. By default, every user belongs to a default or primary group. When a user logs in, the group membership is set for their primary group and all the members enjoy the same level of access and privilege. Permissions on various files and directories can be modified at the group level.

All Linux users are assigned a unique user ID, the **uid**, which is just an integer, as well as one or more group ID’s, the  **gid**, including a default one which is the same as the user ID. Historically, RedHat based distros start uid's at 500. Other distributions begin at 1000. These numbers are associated with names through the files ``/etc/passwd`` and ``/etc/group``. Groups are used to establish a set of users who have common interests for the purposes of access rights, privileges, and security considerations. Access rights to files and devices are granted on the basis of the user and the group they belong to.

Only the root user can add and remove users and groups. Adding a new user is done with the ``useradd`` command and removing an existing user is done with the ``userdel`` command. In the simplest form an account for the new user adriano would be done with:
```
# useradd adriano
# cat /etc/passwd | grep adriano
adriano:x:1000:1000::/home/adriano:/bin/bash
# ls -lrta /home/adriano/
total 16
-rw-r--r--. 1 adriano adriano 231 Sep 26 03:53 .bashrc
-rw-r--r--. 1 adriano adriano 193 Sep 26 03:53 .bash_profile
-rw-r--r--. 1 adriano adriano  18 Sep 26 03:53 .bash_logout
drwxr-xr-x. 3 root    root     20 Feb 17 17:48 ..
-rw-------. 1 adriano adriano   9 Feb 17 17:49 .bash_history
drwx------. 2 adriano adriano  79 Feb 17 17:49 .
```
which by default sets the his home directory to ``/home/adriano``, populates it with some basic files and sets the default shell to ``/bin/bash``.

Remove the user account by typing:
```
# userdel adriano
# cat /etc/passwd | grep adriano
# ls -lrta /home/adriano/
total 16
-rw-r--r--. 1 1000 1000 231 Sep 26 03:53 .bashrc
-rw-r--r--. 1 1000 1000 193 Sep 26 03:53 .bash_profile
-rw-r--r--. 1 1000 1000  18 Sep 26 03:53 .bash_logout
drwxr-xr-x. 3 root root  20 Feb 17 17:48 ..
-rw-------. 1 1000 1000   9 Feb 17 17:49 .bash_history
drwx------. 2 1000 1000  79 Feb 17 17:49 .
```
However, this will leave the home directory intact. This might be useful if it is a temporary inactivation. To remove the home directory while removing the account one needs to use the related option.
```
# userdel -r adriano
# cat /etc/passwd | grep adriano
# ls -lrta /home/adriano/
ls: cannot access /home/adriano/: No such file or directory
```
The command ``id`` with no argument gives information about the current user. If given the name of another user as an argument, id will report information about that other user.
```
# id
uid=0(root) gid=0(root) groups=0(root)
# id adriano
uid=1000(adriano) gid=1000(adriano) groups=1000(adriano)
```
Use the ``passwd`` command to change the password for the new user
```
# passwd adriano
Changing password for user adriano.
New password:
Retype new password:
passwd: all authentication tokens updated successfully.
```

Adding a new group is done with the ``groupadd`` command and removed with the ``groupdel`` command.
```
# groupadd newgroup
# groupdel newgroup
```
Adding a user to an already existing group is done with the ``usermod`` command. Removing a user from the group is a somewhat trickier.

```
# groupadd newgroup
# usermod -G newgroup adriano
# groups adriano
adriano : adriano newgroup
# usermod -g newgroup adriano
# groups adriano
adriano : newgroup
#
```
All these commands update the ``/etc/group`` as necessary. The ``groupmod`` command can be used to change the group properties such as the Group ID or the name
```
# groupmod newgroup -n newgoupname
# groups adriano
adriano : newgoupname
```

### The root user
The **root** account is very powerful and has full access to the system. Other operating systems often call this the administrator account; in Linux it is often called the **superuser** account. You must be extremely cautious before granting full root access to a user; it is rarely if ever justified. External attacks often consist of tricks used to elevate to the root account. However, you can use the sudo feature to assign more limited privileges to standard user accounts:

1. on only a temporary basis.
2. only for a specific subset of commands.

When assigning elevated privileges, you can use the command ``su`` (switch user) to launch a new shell running as another user (you must type the password of the user you are becoming). Most often this other user is root, and the new shell allows the use of elevated privileges until it is exited. It is almost always a bad (dangerous for both security and stability) practice to use ``su`` to become root. Resulting errors can include deletion of vital files from the system and security breaches.

### Startup Files
In Linux, the command shell program, generally **bash** uses one or more startup files to configure the environment. Files in the ``/etc`` directory define global settings for all users while the initialization files in the user's home directory can include and or override the global settings. The startup files can do anything the user would like to do in every command shell, such as:

* Customizing the user's prompt
* Defining command-line shortcuts and aliases
* Setting the default text editor
* Setting the path for where to find executable programs

When you first login to Linux, the  ``/etc/profile`` file is read and evaluated, after which the following files are searched in the listed order:

1. ``~/.bash_profile``
2. ``~/.bash_login``
3. ``~/.profile``

The Linux login shell evaluates whatever startup file that it comes across first and ignores the rest. This means that if it finds ``~/.bash_profile``, it ignores the rest. Different distributions may use different startup files. However, every time you create a new shell, or terminal window, etc., you do not perform a full system login; only the ``~/.bashrc`` file is read and evaluated. Although this file is not read and evaluated along with the login shell, most distributions and/or users include the ``~/.bashrc`` file from within one of the three user-owned startup files. In the Ubuntu, openSuse, and CentOS distros, the user must make appropriate changes in the ``~/.bash_profile`` file to include the ``~/.bashrc`` file. The ``~/.bash_profile`` will have certain extra lines, which in turn will collect the required customization parameters from ``~/.bashrc``.

### Environment variables
The environment variables are simply named quantities that have specific values and are understood by the command shell, such as **bash**. Some of these are pre-set by the system, and others are set by the user either at the command line or within startup and other scripts. An environment variable is actually no more than a character string that contains information used by one or more applications. There are a number of ways to view the values of currently set environment variables. All the ``set``, ``env``, or ``export`` commands display the environment variables.

By default, variables created within a script are only available to the current shell. All the child processes (sub-shells) will not have access to values that have been set or modified. Allowing child processes to see the values, requires use of the ``export`` command.

|Task|Command|
|----|-------|
|Show the value of a specific variable|echo $SHELL|
|Export a new variable value|export VAR=value|
|Add a variable permanently|Add the line export VAR=value to ~/.bashrc|

The **HOME** is an environment variable that represents the home or login directory of the user. The ``cd`` command without arguments will change the current working directory to the value of HOME. Note the tilde character (~) is often used as an abbreviation for $HOME.

The **PATH** environment variable is an ordered list of directories which is scanned when a command is given to find the appropriate program or script to run. Each directory in the path is separated by colons. An empty directory name indicates the current directory at any given time.

```
$ export PATH=$HOME/bin:$PATH
$ echo $PATH
/home/me/bin:/usr/local/bin:/usr/bin:/bin/usr
```

The **PS** environment variable (Prompt Statement) is used to customize your prompt string in your terminal windows to display the information you want. PS1 is the primary prompt variable which controls what your command line prompt looks like. The following special characters can be included in PS1 :

|Character|Usage|
|---------|-----|
|\u|User name| 
|\h|Host name|
|\w|Current working directory| 
|\!|History number of this command|
|\d|Date|

They must be surrounded in single quotes when they are used
```
# export PS1='\u@\h:\w$ '
root@caldera01:~$
root@caldera01:~$ export PS1='\d-\u@\h:\w$ '
Wed Feb 18-root@caldera01:~$
```
The **SHELL** environment variable points to the user's default command shell (the program that is handling whatever you type in a command window, usually bash) and contains the full pathname to the shell
```
$ echo $SHELL
/bin/bash
$
```

### Command history
The bash keeps track of previously entered commands and statements in a history buffer; you can recall previously used commands simply by using the Up and Down cursor keys. To view the list of previously executed commands, you can use the ``history`` at the command line. The list of commands is displayed with the most recent command appearing last in the list. This information is stored in ``~/.bash_history`` file. Several associated environment variables can be used to get information about the history file. 

|Variable|Usage|
|--------|-----|
|HISTFILE|stores the location of the history file|
|HISTFILESIZE|stores the maximum number of lines in the history file|
|HISTSIZE|stores the maximum number of lines in the history file for the current session|

The table below shows the syntax used to execute previously used commands

|Syntax|Usage|
|------|-----|
|!!|Execute the previous command|
|!|Start a history substitution|
|!$|Refer to the last argument in a line|
|!n|Refer to the n-th command line|
|!string|Refer to the most recent command starting with string|

### Creating Aliases
Customized commands can be created to modify the behavior of already existing ones by creating aliases. Most often these aliases are placed in your ``~/.bashrc`` file so they are available to any command shells you create. The ``alias`` command with no arguments will list currently defined aliases.

```
$ alias
alias cp='cp -i'
alias egrep='egrep --color=auto'
alias fgrep='fgrep --color=auto'
alias grep='grep --color=auto'
alias l.='ls -d .* --color=auto'
alias ll='ls -l --color=auto'
alias ls='ls --color=auto'
alias mv='mv -i'
alias rm='rm -i'
```
