## System Services
Systemd is the new init system for modern Linux distributions replacing the old init based on ``/etc/init.d/script``. It provides many powerful features for starting, stopping and managing processes. Here is an example to create a MineCraft service for systemd. MainCraft is a Java based game from Mojang.

First, install the game and its envinronment.
```
# yum install java-1.8.0-openjdk.x86_64
# which java
/bin/java
# mkdir /root/Minecraft
# cd /root/Minecraft
# wget -O minecraft_server.jar https://s3.amazonaws.com/Minecraft.Download/versions/1.8.6/minecraft_server.1.8.6.jar
# ls -lrt
-rw-r--r--. 1 root root 9780573 May 25 11:47 minecraft_server.jar
-rw-r--r--. 1 root root       2 Jun  1 11:48 whitelist.json
-rw-r--r--. 1 root root     180 Jun  1 12:01 eula.txt
drwxr-xr-x. 2 root root    4096 Jun  1 16:09 logs
-rw-r--r--. 1 root root     785 Jun  1 16:09 server.properties
-rw-r--r--. 1 root root       2 Jun  1 16:09 banned-players.json
-rw-r--r--. 1 root root       2 Jun  1 16:09 banned-ips.json
-rw-r--r--. 1 root root       2 Jun  1 16:09 ops.json
-rw-r--r--. 1 root root     109 Jun  1 16:10 usercache.json
drwxr-xr-x. 8 root root    4096 Jun  1 16:37 world
```

The MineCraft server can be started at command line, by issuing the following command
```
# java -Xmx1024M -Xms1024M -jar minecraft_server.jar nogui
```

Alternately, a systemd configuration file can be created to start, stop, and check the status of the server as a standard system service by using the ``systemctl`` utility
```
# vi /lib/systemd/system/minecraftd.service
[Unit]
Description=Minecraft Server
After=syslog.target network.target

[Service]
Type=simple
WorkingDirectory=/root/Minecraft
ExecStart=/bin/java -Xmx1024M -Xms1024M -jar minecraft_server.jar nogui
SuccessExitStatus=143
Restart=on-failure

[Install]
WantedBy=multi-user.target

# systemctl start minecraftd
# systemctl status minecraftd
minecraftd.service - Minecraft Server
   Loaded: loaded (/usr/lib/systemd/system/minecraftd.service; disabled)
   Active: active (running) since Mon 2015-06-01 16:00:12 UTC; 18s ago
 Main PID: 20975 (java)
   CGroup: /system.slice/minecraftd.service
           └─20975 /bin/java -Xmx1024M -Xms1024M -jar minecraft_server.jar nogui

# systemctl stop minecraftd
```
Note: the ``SuccessExitStatus=143`` is required when a process does not handle the exit signal properly. This is almost always due to programming errors, and is pretty common with Java applications of all types. To avoid a failed status of the MainCraft when stopping the service, the exit code 143 needs to be added into the unit file as a "success" exit status.

The ``systemctl`` utility can be used to enable/disable the service at startup
```
# systemctl enable minecraftd
ln -s '/usr/lib/systemd/system/minecraftd.service' '/etc/systemd/system/multi-user.target.wants/minecraftd.service'
# systemctl is-enabled minecraftd
enabled
# systemctl disable minecraftd
```

Here another example
```
# cat /etc/systemd/system/redmined.service
[Unit]
Description=Redmine Server
After=syslog.target network.target

[Service]
Type=simple
PermissionsStartOnly=true
WorkingDirectory=/home/redmine/redmine
ExecStartPre=/usr/sbin/iptables -A PREROUTING -t nat -i eth0 -p tcp --dport 80 -j REDIRECT --to-port 8080
ExecStart=/usr/bin/ruby bin/rails server -b 0.0.0.0 -p 8080 webrick -e production
User=redmine
Group=redmine
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=redmined
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target

```
