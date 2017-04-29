## Text commands
Linux provides utilities for file and text manipulation:

1. Display contents using ``cat`` and ``echo``. 
2. Edit file contents using ``sed`` and ``awk``.
3. Search for patterns using ``grep``.

### Display contents
The ``cat`` is short for concatenate and is often used to read and print files as well as for simply viewing file contents, while the ``tac`` command prints the lines of a file in reverse order.
```
$ cat > myfile.txt
Mario Rossi
Antonio Esposito
Michele Laforca
Ctrl-D
$ cat myfile.txt
Mario Rossi
Antonio Esposito
Michele Laforca
$
$ tac myfile.txt
Michele Laforca
Antonio Esposito
Mario Rossi
```
The ``echo`` simply displays text.
```
$ echo myfile.txt
myfile.txt
]$echo HOME
HOME
$ echo $HOME
/home/ec2-user
```
### Edit file content
The command ``sed`` is a powerful text processing tool. Its name is an abbreviation for stream editor. It filters text as well as perform substitutions in data streams. Data from an input source/file (or stream) is taken and moved to a working space. The entire list of operations/modifications is applied over the data in the working space and the final contents are moved to the standard output space (or stream).
```
$ sed s/Mario/Saverio/ myfile.txt
Saverio Rossi
Antonio Esposito
Michele Laforca
$ cat myfile.txt
Mario Rossi
Antonio Esposito
Michele Laforca
$ sed s/Mario/Saverio/ myfile.txt >  myfile2.txt
$ cat myfile2.txt
Saverio Rossi
Antonio Esposito
Michele Laforca
$ sed -i s/Mario/Saverio/ myfile.txt
$ cat myfile.txt
Saverio Rossi
Antonio Esposito
Michele Laforca
```
For example, to convert 01/02/… to JAN/FEB/…
```
sed -e 's/01/JAN/' -e 's/02/FEB/' -e 's/03/MAR/' -e 's/04/APR/' -e 's/05/MAY/' \ 
-e 's/06/JUN/' -e 's/07/JUL/' -e 's/08/AUG/' -e 's/09/SEP/' -e 's/10/OCT/' \
-e 's/11/NOV/' -e 's/12/DEC/'
```
The ``awk`` command is used to extract and then print specific contents of a file and is often used to construct reports. It is a powerful utility and interpreted programming language, used to manipulate data files, retrieving, and processing text.
It works well with fields (containing a single piece of data, essentially a column) and records (a collection of fields, essentially a line in a file).

```
$ awk '{ print $0 }' myfile.txt
Saverio Rossi
Antonio Esposito
Michele Laforca
$ awk '{ print $1 }' myfile.txt
Saverio
Antonio
Michele
$ awk '{ print $2 }' myfile.txt
Rossi
Esposito
Laforca
```
Please, check the man pages for the ``awk`` and ``sed`` commands for futher details.

### File manipulation
The ``sort`` command is used to rearrange the lines of a text file either in ascending or descending order, according to a sort key.
```
# cat myfile.txt
Mario Rossi
Antonio Esposito
Michele Laforca
# sort myfile.txt
Antonio Esposito
Mario Rossi
Michele Laforca
# sort -r myfile.txt
Michele Laforca
Mario Rossi
Antonio Esposito
```
The ``uniq`` is used to remove duplicate lines in a text file and is useful for simplifying text display. It requires that the duplicate entries to be removed are consecutive.

```
# cat myfile.txt
Mario Rossi
Antonio Esposito
Michele Laforca
Antonio Esposito
# sort myfile.txt | uniq
Antonio Esposito
Mario Rossi
Michele Laforca
# sort myfile.txt | uniq -c
      2 Antonio Esposito
      1 Mario Rossi
      1 Michele Laforca
```

The ``paste`` command is used to combine fields from different files

```
# cat names.txt
Mario Rossi
Antonio Esposito
Michele Laforca
Antonio Esposito
[root@caldera01 ~]# cat ages.txt
34
46
29
46
[root@caldera01 ~]# paste names.txt ages.txt
Mario Rossi     34
Antonio Esposito        46
Michele Laforca 29
Antonio Esposito        46
```

The ``join`` command combines two files on a common field

```
# cat names.txt
01 Mario Rossi
02 Antonio Esposito
03 Michele Laforca
04 Antonio Esposito
# cat ages.txt
01 34
02 46
03 29
04 46
# join names.txt ages.txt
01 Mario Rossi 34
02 Antonio Esposito 46
03 Michele Laforca 29
04 Antonio Esposito 46
```

The ``grep`` comand is extensively used as a primary text searching tool. It scans files for specified patterns and can be used with regular expressions.
```
# grep Ant* names.txt
02 Antonio Esposito
04 Antonio Esposito
```
The ``tr`` utility is used to **tr**anslate specified characters into other characters or to delete them.
```
# cat names.txt
01 Mario Rossi
02 Antonio Esposito
03 Michele Laforca
04 Antonio Esposito
# cat names.txt | tr a-z A-Z
01 MARIO ROSSI
02 ANTONIO ESPOSITO
03 MICHELE LAFORCA
04 ANTONIO ESPOSITO
```
The ``tee`` command takes the output from any command, and while sending it to standard output, it also saves it to a file.
```
# ls -l | tee list.txt
total 32
-rw-r--r--. 1 root root   24 Mar  3 14:42 ages.txt
-rw-------. 1 root root 1883 Jan 21 20:53 anaconda-ks.cfg
-rw-r--r--. 1 root root   74 Mar  3 14:42 names.txt
-rwxr--r--. 1 root root  102 Feb 21 16:47 script.sh
-rw-r--r--. 1 root root   74 Mar  3 14:52 tr
[root@caldera01 ~]# cat list.txt
total 32
-rw-r--r--. 1 root root   24 Mar  3 14:42 ages.txt
-rw-------. 1 root root 1883 Jan 21 20:53 anaconda-ks.cfg
-rw-r--r--. 1 root root   74 Mar  3 14:42 names.txt
-rwxr--r--. 1 root root  102 Feb 21 16:47 script.sh
-rw-r--r--. 1 root root   74 Mar  3 14:52 tr
```

The ``wc`` (word count) counts the number of lines, words, and characters in a file or list of files. 
```
# cat names.txt
01 Mario Rossi
02 Antonio Esposito
03 Michele Laforca
04 Antonio Esposito
[root@caldera01 ~]# wc -l names.txt
4 names.txt
[root@caldera01 ~]# wc -c names.txt
74 names.txt
[root@caldera01 ~]# wc -w names.txt
12 names.txt
```
The ``cut`` command is used for manipulating column-based files and is designed to extract specific columns. The default column separator is the tab character. A different delimiter can be given as a command option.
```
# cut -d" " -f1 names.txt
01
02
03
04
# cut -d" " -f2 names.txt
Mario
Antonio
Michele
Antonio
# cut -d" " -f3 names.txt
Rossi
Esposito
Laforca
Esposito
```

The ``head`` reads the first few lines of each named file (10 by default) and displays it on standard output.
```
# head -n 2 names.txt
01 Mario Rossi
02 Antonio Esposito
```
The ``tail`` prints the last few lines of each named file and displays it on standard output. By default, it displays the last 10 lines.
```
# tail -n 2 names.txt
03 Michele Laforca
04 Antonio Esposito
#
# tail -f -n3 /var/log/messages
Mar  3 14:38:59 caldera01 systemd: Started Session 35 of user root.
Mar  3 15:01:01 caldera01 systemd: Starting Session 36 of user root.
Mar  3 15:01:01 caldera01 systemd: Started Session 36 of user root.
```
