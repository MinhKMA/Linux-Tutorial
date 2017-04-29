### The file streams
When commands are executed, by default there are three standard file streams or descriptors always open for use:

1. standard input or **stdin**
2. standard output or **stdout**
3. standard error or **stderr**

Usually, **stdin** is your keyboard, **stdout** and **stderr** are printed on your terminal; often **stderr** is redirected to an error logging file. The **stdin** is often supplied by directing input to come from a file or from the output of a previous command through a pipe. The **stdout** is also often redirected into a file. Since **stderr** is where error messages are written, often nothing will go there.

In Linux, all open files are represented internally by what are called file descriptors. Simply put, these are represented by numbers starting at zero. The **stdin** is file descriptor 0, **stdout** is file descriptor 1, and **stderr** is file descriptor 2. Typically, if other files are opened in addition to these three, which are opened by default, they will start at file descriptor 3 and increase from there.

We can redirect the three standard filestreams so that we can get input from either a file or another command instead of from our keyboard, and we can write output and errors to files or send them as input for subsequent commands. For example, having a program *called do_something* that reads from **stdin** and writes to **stdout** and **stderr**, we can change its input source:
```
$ do_something < input-file
```
If you want to send the output to a file, use the this as in:
```
$ do_something > output-file
```
We can pipe the output of one command or program into another as its input.
```
$ command1 | command2 | command3
```
The above represents what we often call a _pipeline_ and allows linux to combine the actions of several commands into one. 

### Search for files
The ``locate`` utility performs a search through a previously constructed database of files and directories on your system, matching all entries that contain a specified character string. The ``locate`` utilizes the database created by another program, ``updatedb``. Most Linux systems run this automatically once a day. However, you can update it at any time by just running ``updatedb`` from the command line as the root user.
```
# yum install -y mlocate
# updatedb
# locate zip
```
The result of ``locate`` utility can sometimes result in a very long list. To get a shorter more relevant list we can use the ``grep`` program as a filter. It will print only the lines that contain one or more specified strings as in: 
```
$ locate zip | grep bin
/usr/bin/gpg-zip
/usr/bin/gunzip
/usr/bin/gzip
/usr/bin/zipdetails
```
which will list all files and directories with both "zip" and "bin" in their name. 

Wildcards can be used in search for a filename containing specific characters.

|Wildcards|Result|
|---------|-----------|
|?     |Matches any single character|
|*     |Matches any string of characters|
|[set] |Matches any character not in the set of characters|
|[!set]|Matches any character not in the set of characters|

The ``find`` is extremely useful and often-used utility program in the daily life of a Linux system administrator. It recurses down the filesystem tree from any particular directory (or set of directories) and locates files that match specified conditions. The default <pathname> is always the present working directory.
```
$ find /var -name *.log
/var/log/audit/audit.log
/var/log/tuned/tuned.log
/var/log/anaconda/anaconda.log
/var/log/anaconda/anaconda.program.log
/var/log/anaconda/anaconda.packaging.log
/var/log/anaconda/anaconda.storage.log
```
When no arguments are given, ``find`` lists all files in the current directory and all of its subdirectories.

Searching for files and directories named "gcc":
```
$ find /usr -name gcc
```
Searching only for directories named "gcc":
```
$ find /usr -type d -name gcc
```
Searching only for regular files named "test1":
```
$ find /usr -type f -name test1
```
Another good use of ``find`` is being able to run commands on the files that match your search criteria. To find and remove all files that end with .swp:
```
$ find -name "*.swp" -exec rm {} ’;’
$ find -name "*.swp" -ok rm {} \;
```
The {} is a place holder that will be filled with all the file names that result from the find expression, and the preceding command will be run on each one individually. Note that you have to end the command with either ``‘;’`` or ``\;`` Both forms are fine. The second form behaves the same as the first one except that find will prompt you for permission before executing the command. This makes it a good way to test your results before blindly executing any potentially dangerous commands.

It is sometimes the case that you wish to find files according to attributes such as when they were created, last used, etc, or based on their size. Both are easy to accomplish.

Finding based on time:
```
$ find / -ctime 3
```

Here, _-ctime_ is when the inode meta-data (i.e., file ownership, permissions, etc) last changed; it is often, but not necessarily when the file was first created. You can also search for accessed/last read _-atime_ or modified/last written _-mtime_ times. The number is the number of days and can be expressed as either a number (n) that means exactly that value, +n which means greater than that number, or -n which means less than that number.

Finding based on sizes:
```
$ find / -size +10M
```
To find files greater than 10 MB in size.

### Manage files
Use the following utilities to view files:

|Command|Usage|
|-------|-----------|
|cat  |Used for viewing files that are not very long|
|tac  |Used to look at a file backwards, starting with the last line|
|less |Used to view larger files because it is a paging program; it pauses at each screenful of text, provides scroll-back capabilities, and lets you search and navigate within the file.|
|tail |Used to print the last 10 lines of a file by default. You can change the number of lines by doing -n 15 or just -15 if you wanted to look at the last 15 lines instead of the default|
|head |The opposite of tail; by default it prints the first 10 lines of a file|

The ``touch`` command is often used to set or update the access, change, and modify times of files. By default it resets a file's time stamp to match the current time.

However, you can also create an empty file using touch:
```
$ touch <filename>
```
This is normally done to create an empty file as a placeholder for a later purpose.
The -t option allows you to set the date and time stamp of the file.
To set the time stamp to a specific time:
```
$ touch -t 03201600 <filename>
```
This sets the file, myfile's, time stamp to 4 p.m., March 20th (03 20 1600).

The ``mkdir`` command is used to create a directory. Removing a directory is simply done with ``rmdir`` command. The directory must be empty or it will fail.
```
# mkdir ./test
# rmdir ./test
# 
# mkdir ./test
# mkdir ./test/inside
# rmdir ./test
rmdir: failed to remove ‘test’: Directory not empty
# rm -rf ./test
# ls ./test
ls: cannot access ./test: No such file or directory
```

### Compare files
The ``diff`` command is used to compare files and directories. 

```
$ cat file1.txt
Amor, ch'a nullo amato amar perdona,
Mi prese del costui piacer si forte,
Che, come vedi, ancor non m'abbandona.
$ 
$  cat file2.txt
amor, ch'a nullo amato amar perdona,
mi prese del costui piacer si forte,
che, come vedi, ancor non m'abbandona.
$ 
$ diff  file1.txt file2.txt
< Amor, ch'a nullo amato amar perdona,
< Mi prese del costui piacer si forte,
< Che, come vedi, ancor non m'abbandona.
---
> amor, ch'a nullo amato amar perdona,
> mi prese del costui piacer si forte,
> che, come vedi, ancor non m'abbandona.
$ 
$ diff -c file1.txt file2.txt
*** file1.txt   2015-02-17 16:10:03.781804799 +0100
--- file2.txt   2015-02-17 16:13:41.059088459 +0100
***************
! Amor, ch'a nullo amato amar perdona,
! Mi prese del costui piacer si forte,
! Che, come vedi, ancor non m'abbandona.
--- 1,3 ----
! amor, ch'a nullo amato amar perdona,
! mi prese del costui piacer si forte,
! che, come vedi, ancor non m'abbandona.
$ 
$  diff -i file1.txt file2.txt
$ 
```
### The file utility
In Linux, a file's extension often does not categorize it the way it might in other operating systems. One can not assume that a file named ``file.txt`` is a text file and not an executable program. In Linux a file name is generally more meaningful to the user of the system than the system itself; in fact most applications directly examine a file's contents to see what kind of object it is rather than relying on an extension. The real nature of a file can be ascertained by using the ``file`` utility. For the file names given as arguments, it examines the contents and certain characteristics to determine whether the files are plain text, shared libraries, executable programs, scripts, or something else.

```
$ file /etc/resolv.conf
/etc/resolv.conf: ASCII text
```
