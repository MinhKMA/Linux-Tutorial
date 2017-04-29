### File permissions
In Linux and other UNIX operating systems, every file is associated with a user who is the owner. Every file is also associated with a group which has an interest in the file and certain rights, or permissions: read, write, and execute.

|Command|Result|
|-------|-----------|
|chown|Used to change user ownership of a file or directory|
|chgrp|Used to change group ownership|
|chmod|Used to change the permissions on the file|

Files have three kinds of permissions: read (**r**), write (**w**), execute (**x**). These are generally represented as in the following order **rwx**. These permissions affect three groups of owners: user (**u**), group (**g**), and others (**o**). As a result, you have the following three groups of three permissions:

|rwx:|rwx:|rwx|
|----|----|---|
|u:|g:|o|

There are a number of different ways to use the ``chmod`` command. For instance, to give the owner execute permission:

```
$ ls -l test1
-rw-rw-r-- 1 joy caldera 1601 Mar 9 15:04 test1
$ chmod u+x test1
$ ls -l test1
-rwxrw-r-- 1 joy caldera 1601 Mar 9 15:04 test1
```

This kind of syntax can be difficult to type and remember, so one often uses a shorthand which lets you set all the permissions in one step. This is done with a simple algorithm, and a single digit suffices to specify all three permission bits for each entity. This digit is the sum of:

* 4 if read permission is desired.
* 2 if write permission is desired.
* 1 if execute permission is desired.

Thus 7 means read+write+execute, 6 means read+write, and 5 means read+execute.

When you apply this to the ``chmod`` command you have to give three digits for each degree of freedom, such as in
```
$ chmod 755 test1
$ ls -l test1
-rwxr-xr-x 1 joy caldera 1601 Mar 9 15:04 test1
```
The group ownership is changed by using the ``chgrp`` command
```
# ll /home/mina/myfile.txt
-rw-rw-r--. 1 mina caldera 679 Feb 19 16:51 /home/mina/myfile.txt
# chgrp root /home/mina/myfile.txt
# ll /home/mina/myfile.txt
-rw-rw-r--. 1 mina root 679 Feb 19 16:51 /home/mina/myfile.txt
```
