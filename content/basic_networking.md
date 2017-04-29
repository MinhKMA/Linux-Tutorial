## Network interfaces
Network interfaces are a connection channel between a device and a network. Physically, network interfaces can proceed through a network interface card (**NIC**) or can be more abstractly implemented as software. You can have multiple network interfaces operating at once. Specific interfaces can be brought up (activated) or brought down (de-activated) at any time. A list of currently active network interfaces is reported by the ``ifconfig`` utility. Network configuration files are essential to ensure that interfaces function correctly.

For **Debian** family configuration, the basic network configuration file is ``/etc/network/interfaces``. For **RedHat** family system configuration, the routing and host information is contained in ``/etc/sysconfig/network``. The network interface configuration script for the ``eth0`` interface is located at ``/etc/sysconfig/network-scripts/ifcfg-eth0``. For **SUSE** family system configuration, the routing and host information and network interface configuration scripts are contained in the ``/etc/sysconfig/network`` directory.

```
# cat /etc/sysconfig/network-scripts/ifcfg-enp0s25
TYPE=Ethernet
BOOTPROTO=none
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=no
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
NAME=enp0s25
UUID=d9315bd4-159b-4871-95f5-98f2fbcc5a06
ONBOOT=yes
HWADDR=00:24:81:0F:EC:DE
IPADDR=10.10.10.97
PREFIX=24
GATEWAY=10.10.10.1
DNS=8.8.8.8
```

The ``ip`` is a very powerful program that can do many things.
```
# ip addr show
# ip route show
```

### Routing table
The ``route`` command is used to view or change the IP routing table. You may want to change the IP routing table to add, delete or modify static routes to specific hosts or networks.

```
# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.10.10.1      0.0.0.0         UG    0      0        0 enp0s25
10.10.10.0      0.0.0.0         255.255.255.0   U     0      0        0 enp0s25
169.254.0.0     0.0.0.0         255.255.0.0     U     1002   0        0 enp48s0
169.254.0.0     0.0.0.0         255.255.0.0     U     1003   0        0 enp0s25
172.25.101.0    0.0.0.0         255.255.255.0   U     0      0        0 enp48s0
# 
# route add 10.58.47.235 gw 172.25.101.1
route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.10.10.1      0.0.0.0         UG    0      0        0 enp0s25
10.10.10.0      0.0.0.0         255.255.255.0   U     0      0        0 enp0s25
10.58.47.235    172.25.101.1    255.255.255.255 UGH   0      0        0 enp48s0
169.254.0.0     0.0.0.0         255.255.0.0     U     1002   0        0 enp48s0
169.254.0.0     0.0.0.0         255.255.0.0     U     1003   0        0 enp0s25
172.25.101.0    0.0.0.0         255.255.255.0   U     0      0        0 enp48s0
# 
# route delete 10.58.47.235 gw 172.25.101.1
# route add -net 10.0.0.0 netmask 255.0.0.0 gw 10.10.10.1 enp0s25
# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.10.10.1      0.0.0.0         UG    0      0        0 enp0s25
10.0.0.0        10.10.10.1      255.0.0.0       UG    0      0        0 enp0s25
10.10.10.0      0.0.0.0         255.255.255.0   U     0      0        0 enp0s25
169.254.0.0     0.0.0.0         255.255.0.0     U     1002   0        0 enp48s0
169.254.0.0     0.0.0.0         255.255.0.0     U     1003   0        0 enp0s25
172.25.101.0    0.0.0.0         255.255.255.0   U     0      0        0 enp48s0
# route delete -net 10.0.0.0 netmask 255.0.0.0 gw 10.10.10.1 enp0s25
```
