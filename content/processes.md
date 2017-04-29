### Linux processes
A **process** is simply an instance of one or more related tasks (**threads**) executing on the same machine. It is not the same as a program or a command; a single program may actually start several processes simultaneously. Some processes are independent of each other and others are related. A failure of one process may or may not affect the others running on the system. Processes use many system resources, such as memory, CPU cycles, and peripheral devices such as printers and displays. The operating system (especially the kernel) is responsible for allocating a proper share of these resources to each process and ensuring overall optimum utilization.

A terminal window, is a process that runs as long as needed. It allows users to execute programs and access resources in an interactive environment. You can also run programs in the background, which means they become detached from the shell. Processes can be of different types according to the task being performed. 

|Type|Description|
|--------|---------|
|Interactive |Need to be started by a user, either at a command line or through a graphical interface such as an icon or a menu selection.|
|Batch |Automatic processes which are scheduled from and then disconnected from the terminal. These tasks are queued and work on a FIFO (First In, First Out) basis.|
|Daemons|Server processes that run continuously. Many are launched during system startup and then wait for a user or system request indicating that their service is required.|
|Threads|Lightweight processes. These are tasks that run under the umbrella of a main process, sharing memory and other resources, but are scheduled and run by the system on an individual basis.|
|Kernel Threads|Kernel tasks that users neither start nor terminate and have little control over. These may perform actions like moving a thread from one CPU to another, or making sure input/output operations to disk are completed.|

When a process is in the **running state**, it means it is either currently executing instructions on a CPU, or is waiting for a share (or time slice) so it can run. A critical kernel routine called the **scheduler** constantly shifts processes in and out of the CPU, sharing time according to relative priority, how much time is needed and how much has already been granted to a task. All processes in this state reside on a run queue and on a computer with multiple CPUs there is a run queue on each. Sometimes processes go into the **sleep** state, generally when they are waiting for something to happen before they can resume, perhaps for the user to type something. In this condition a process is sitting in a wait queue. There are some other less frequent process states, especially when a process is terminating. Sometimes a child process completes but its parent process has not asked about its state. Such a process is said to be in a **zombie** state; it is not really alive but still shows up in the system's list of processes.

At any given time there are always multiple processes being executed. The operating system keeps track of them by assigning each a unique process ID or **PID** number. The PID is used to track process state, cpu usage, memory use, precisely where resources are located in memory, and other characteristics. New PIDs are usually assigned in ascending order as processes are born. Thus PID 1 denotes the **init** process (initialization process), and succeeding processes are gradually assigned higher numbers.

At any given time, many processes are running on the system. However, a **CPU** can actually accommodate only one task at a time, just like a car can have only one driver at a time. Some processes are more important than others so Linux allows you to set and manipulate process priority. Higher priority processes are granted more time on the processor. The **priority** for a process can be set by specifying a nice value, or **niceness**, for the process. The lower the nice value, the higher the priority. Low values are assigned to important processes, while high values are assigned to processes that can wait longer. A process with a high nice value simply allows other processes to be executed first. In Linux, a nice value of -20 represents the highest priority and 19 represents the lowest. You can also assign a real-time priority to time-sensitive tasks, such as controlling machines or collecting incoming data. This is just a very high priority and is not to be confused with what is called hard real time which is conceptually different, and has more to do with making sure a job gets completed within a very well-defined time window.

### Running processes
The ``ps`` command provides information about currently running processes, keyed by **PID**. If you want a repetitive update of this status, you can use the ``top`` command or commonly installed variants such as ``htop`` or ``atop`` from the command line. The ``ps`` command has many options for specifying exactly which tasks to examine, what information to display about them, and precisely what output format should be used.

Without options ``ps`` will display all processes running under the current shell. You can use the `` ps -u`` to display information of processes for a specified username. The command ``ps -ef`` displays all the processes in the system in full detail. The command ``ps -eLf`` goes one step further and displays one line of information for every thread (a process can contain multiple threads).

```
# ps -u adriano
  PID TTY          TIME CMD
  847 ?        00:00:00 sshd
  848 pts/2    00:00:00 bash
 1070 ?        00:00:00 sshd
 1071 pts/3    00:00:00 bash
 6475 pts/3    00:00:00 top
```

The ``pstree`` command displays the processes running on the system in the form of a tree diagram showing the relationship between a process and its parent process and any other processes that it created. Repeated entries of a process are not displayed, and threads are displayed in curly braces.
```
# yum install -y psmisc
# pstree
# systemd─┬─agetty
          ├─auditd───{auditd}
          ├─avahi-daemon───avahi-daemon
          ├─crond
          ├─dbus-daemon───{dbus-daemon}
          ├─firewalld───{firewalld}
          ├─iprdump
          ├─iprinit
          ├─iprupdate
          ├─lvmetad
          ├─master─┬─pickup
          │        └─qmgr
          ├─polkitd───5*[{polkitd}]
          ├─rsyslogd───2*[{rsyslogd}]
          ├─sshd───sshd───bash───pstree
          ├─systemd-journal
          ├─systemd-logind
          ├─systemd-udevd
          └─tuned───4*[{tuned}]
```

To terminate a process you can type ``kill -SIGKILL <pid>`` or ``kill -9 <pid>``. Note however, you can only kill your own processes: those belonging to another user are off limits unless you are root.

While a static view of what the system is doing is useful, monitoring the system performance live over time is also valuable. One option would be to run the ``ps`` command at regular intervals. A better alternative is to use ``top`` to get constant real-time updates (every two seconds by default). The ``top`` command clearly highlights which processes are consuming the most CPU cycles and memory.
```
top - 15:40:31 up 4 days,  2:13,  1 user,  load average: 0.77, 0.66, 0.45
Tasks: 244 total,   2 running, 241 sleeping,   0 stopped,   1 zombie
%Cpu(s):  6.5 us,  1.3 sy,  0.0 ni, 88.3 id,  3.7 wa,  0.0 hi,  0.2 si,  0.0 st
KiB Mem:   3801380 total,  3642652 used,   158728 free,       24 buffers
KiB Swap:  4079612 total,     3072 used,  4076540 free.   326620 cached Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
 1367 glance    20   0  351800  54996   5928 S   1.3  1.4  64:32.28 glance-api
 1373 nova      20   0  383444  73304   6768 S   1.3  1.9  68:25.51 nova-api
 1365 keystone  20   0  353108  58340   6192 S   1.0  1.5  67:16.01 keystone-all
 1369 cinder    20   0  365404  60120   6632 S   1.0  1.6  68:38.74 cinder-api
 1371 cinder    20   0  287924  30584   4924 S   1.0  0.8  68:08.01 cinder-volume
 1380 nova      20   0  348120  46568   6580 S   1.0  1.2  68:19.02 nova-conductor
 1408 ceilome+  20   0  254312  20560   4260 R   1.0  0.5  64:28.69 ceilometer-agen
...
```
The first line of the ``top`` output displays a quick summary of what is happening in the system including:

1. How long the system has been up
2. How many users are logged on
3. What is the load average

The load average determines how busy the system is. A load average of 1.00 per CPU indicates a fully subscribed, but not overloaded, system. If the load average goes above this value, it indicates that processes are competing for CPU time. If the load average is very high, it might indicate that the system is having a problem, such as a **runaway** process (a process in a non-responding state).

The second line of the ``top`` output displays the total number of processes, the number of running, sleeping, stopped and zombie processes. Comparing the number of running processes with the load average helps determine if the system has reached its capacity or perhaps a particular user is running too many processes. The stopped processes should be examined to see if everything is running correctly.

The third line of the ``top`` output indicates how the CPU time is being divided between the users (**us**) and the kernel (**sy**) by displaying the percentage of CPU time used for each. The percentage of user jobs running at a lower priority (**ni**) is then listed. Idle mode (**id**) should be low if the load average is high, and vice versa. The percentage of jobs waiting (**wa**) for I/O is listed. Interrupts include the percentage of hardware (**hi**) vs. software interrupts (**si**). Steal time (**st**) is generally used with virtual machines, which has some of its idle CPU time taken for other uses.

The fourth and fifth lines of the ``top`` output indicate memory usage, which is divided in two categories:

1. Physical memory (RAM) – displayed on line 4.
2. Swap space – displayed on line 5.
3. Both categories display total memory, used memory, and free space.

You need to monitor memory usage very carefully to ensure good system performance. Once the physical memory is exhausted, the system starts using swap space as an extended memory pool, and since accessing disk is much slower than accessing memory, this will negatively affect system performance. If the system starts using swap often, you can add more swap space. However, adding more physical memory should also be considered.

Each line in the process list of the ``top`` output displays information about a process. By default, processes are ordered by highest CPU usage. The following information about each process is displayed:

* Process Identification Number (PID)
* Process owner (USER)
* Priority (PR) and nice values (NI)
* Virtual (VIRT), physical (RES), and shared memory (SHR)
* Status (S)
* Percentage of CPU (%CPU) and memory (%MEM) used
* Execution time (TIME+)
* Command (COMMAND)

To control the healt of a system, the average load of the system should be checked first. Assuming our system is a single-CPU system, the 0.25 means that for the past minute, on average, the system has been 25% utilized. 0.12 in the next position means that over the past 5 minutes, on average, the system has been 12% utilized; and 0.15 in the final position means that over the past 15 minutes, on average, the system has been 15% utilized. If we saw a value of 1.00 in the second position, that would imply that the single-CPU system was 100% utilized, on average, over the past 5 minutes; this is good if we want to fully use a system. A value over 1.00 for a single-CPU system implies that the system was over-utilized: there were more processes needing CPU than CPU was available. If we had more than one CPU, say a quad-CPU system, we would divide the load average numbers by the number of CPUs. In this case, for example, seeing a 1 minute load average of 4.00 implies that the system as a whole was 100% (4.00/4) utilized during the last minute. Short term increases are usually not a problem. A high peak you see is likely a burst of activity, not a new level. For example, at start up, many processes start and then activity settles down. If a high peak is seen in the 5 and 15 minute load averages, it would may be cause for concern.

### Background and foreground processes
Linux supports **background** and **foreground** job processing. Foreground jobs run directly from the shell, and when one foreground job is running, other jobs need to wait for shell access until it is completed. This is fine when jobs complete quickly. But this can have an adverse effect if the current job is going to take a long time to complete. In such cases, you can run the job in the background and free the shell for other tasks. The background job will be executed at lower priority, which, in turn, will allow smooth execution of the interactive tasks, and you can type other commands in the terminal window while the background job is running. By default all jobs are executed in the foreground. This You can put a job in the background:

```
# updatedb &
[1] 7437
# jobs
[1]+  Done                    updatedb
#
```

### Scheduling processes
The ``at`` utility program is used to execute any non-interactive command at a specified time. The ``at`` jobs is picked by the ``atd`` service.
```
# yum install -y at
# systemctl start atd
# systemctl enable atd
# at now + 5 minutes
at> pstree
at> <EOT>
job 9 at Sat Feb 21 16:28:00 2015
```

The ``atq`` command is used to list the scheduled jobs by the ``at`` command.
```
# atq
9       Sat Feb 21 16:28:00 2015 a root
```

The ``cron`` utility is a time-based scheduling utility program. It can launch routine background jobs at specific times and or days on an on-going basis. cron is driven by a configuration file called ``/etc/crontab`` which contains the various shell commands that need to be run at the properly scheduled times. There are both system-wide crontab files and individual user-based ones. Each line of a crontab file represents a job, and is composed of an expression, followed by a shell command to execute. The ``crontab -e`` command will open the crontab editor to edit existing jobs or to create new jobs. Each line of the crontab file will contain 6 fields:

1. MIN	Minutes	0 to 59
2. HOUR	Hour field	0 to 23
3. DOM	Day of Month	1-31
4. MON	Month field	1-12
5. DOW	Day Of Week	0-6 (0 = Sunday)
6. CMD	Command	Any command to be executed

For example, the entry
```
* * * * * /usr/local/bin/execute/this/script.sh
```
will schedule a job to execute the script every minute of every hour of every day of the month, and every month and every day in the week. The entry
```
30 08 10 06 * /home/sysadmin/full-backup
```
will schedule a full-backup at 8.30am, 10-June irrespective of the day of the week.

### Delaying processes
Sometimes a command or job must be delayed or suspended. Suppose, for example, an application has read and processed the contents of a data file and then needs to save a report on a backup system. If the backup system is currently busy or not available, the application can be made to sleep until it can complete its work. Such a delay might be to mount the backup device and prepare it for writing. The ``sleep`` command suspends execution for at least the specified period of time, which can be given as the number of seconds (the default), minutes, hours or days. After that time has passed, the execution will resume.

```
# vi script.sh
#!/bin/bash
echo "The system will go to sleep fo 30 seconds ..."
sleep 15
echo "The system is awaked"
# chmod u+x script.sh
# ./script.sh
The system will go to sleep fo 30 seconds ...
The system is awaked
#
```
