## Linux Virtual Networking
Linux virtual networking is made easy by libvirt and linux virtual bridge. See [libvirt Virtual Networking](http://wiki.libvirt.org/page/VirtualNetworking).

Install, start and enable the libvirt
```
# yum install libvirt
# systemctl start libvirtd
# systemctl enable libvirtd
```

The libvirt supports following type of virtual networking:

1. Network Address Translation mode
2. Routed mode
3. Isolated mode
4. Bridged Mode

#### Virtual Networking in NAT Mode
When the libvirt daemon is first installed on a server, it comes with an initial default virtual network switch configuration. This default virtual switch is in NAT mode, and is used by installed Virtual Machines for communication to the outside network through the host physical machine. The configuration is stored in the XML file ``/etc/libvirt/qemu/networks/default.xml``. A ``virbr0`` interface is also created on the host machine.
```
# cat /etc/libvirt/qemu/networks/default.xml
<network>
  <name>default</name>
  <uuid>f6514662-26e5-431e-8544-fd216bed9312</uuid>
  <forward mode='nat'/>
  <bridge name='virbr0' stp='on' delay='0'/>
  <mac address='52:54:00:53:d4:ee'/>
  <ip address='192.168.122.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.122.2' end='192.168.122.254'/>
    </dhcp>
  </ip>
</network>

# ifconfig virbr0
virbr0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 192.168.122.1  netmask 255.255.255.0  broadcast 192.168.122.255
        ether 52:54:00:53:d4:ee  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

# ping 192.168.122.1
PING 192.168.122.1 (192.168.122.1) 56(84) bytes of data.
64 bytes from 192.168.122.1: icmp_seq=1 ttl=64 time=0.044 ms
64 bytes from 192.168.122.1: icmp_seq=2 ttl=64 time=0.035 ms
^C
```
The ``virsh`` command is used to configure and check the virtual networking
```
# virsh net-list
 Name                 State      Autostart     Persistent
----------------------------------------------------------
 default              active     yes           yes
```
If it is missing, then the example XML config can be reloaded & activated
```
# virsh net-list
 Name                 State      Autostart     Persistent
----------------------------------------------------------

# virsh net-define /etc/libvirt/qemu/networks/default.xml
Network default defined from /etc/libvirt/qemu/networks/default.xml
# virsh net-start default
Network default started
# virsh net-list
 Name                 State      Autostart     Persistent
----------------------------------------------------------
 default              active     yes           yes
```

When the libvirt default network is running, you will see an isolated bridge device. This device explicitly does *NOT* have any physical interfaces added, since it uses NAT + forwarding to connect to outside world. Do not add interfaces to this bridge
```
# brctl show
bridge name     bridge id               STP enabled     interfaces
virbr0          8000.52540053d4ee       yes             virbr0-nic
```

The libvirt will add iptables rules to allow traffic to/from guests attached to the virtual interface. It will also attempt to enable ip_forward. Some other applications may disable it, so the best option is to add the following to check before to procede
```
# cat /etc/sysctl.conf
...
net.ipv4.ip_forward=1
...
```

Create a Virtual Machine (KVM) and attach it to the virtual network
```
# yum install virt-install
# virt-install \
--name VM1 \
--ram 2048 \
--vcpu 2 \
--network network=default \
--graphics none \
--os-type linux \
--disk path=/data/vm-images/vm1.img,size=10 \
--location /tmp/ubuntu-14.04.1-server-amd64.iso \
--extra-args 'console=ttyS0,115200n8 serial'
```

On the host machine, check that a new interface ``vnet0`` has been set and this interface is part of the linux bridge previously defined
```
#ifconfig vnet0
vnet0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 fe80::fc54:ff:fe6d:3355  prefixlen 64  scopeid 0x20<link>
        ether fe:54:00:6d:33:55  txqueuelen 500  (Ethernet)
        RX packets 117  bytes 55089 (53.7 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 6958  bytes 372399 (363.6 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

# brctl show
bridge name     bridge id               STP enabled     interfaces
virbr0          8000.52540053d4ee       yes             virbr0-nic
                                                        vnet0

```

Login to the VM machine and check the network access to external networks
```
# virsh list --all
 Id    Name                           State
----------------------------------------------------
 4     VM1                            running

# virsh console VM1
Connected to domain VM1
Ubuntu 14.04.2 LTS VM1 ttyS0
VM1 login: caldera
Password:

caldera@VM1:~$ ifconfig eth0
eth0      Link encap:Ethernet  HWaddr 52:54:00:6d:33:55
          inet addr:192.168.122.91  Bcast:192.168.122.255  Mask:255.255.255.0
          inet6 addr: fe80::5054:ff:fe6d:3355/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:94 errors:0 dropped:0 overruns:0 frame:0
          TX packets:119 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:14491 (14.4 KB)  TX bytes:55473 (55.4 KB)

caldera@VM1:~$ netstat -nr
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
0.0.0.0         192.168.122.1   0.0.0.0         UG        0 0          0 eth0
192.168.122.0   0.0.0.0         255.255.255.0   U         0 0          0 eth0
caldera@VM1:~$

caldera@VM1:~$ ping github.com
PING github.com (192.30.252.128) 56(84) bytes of data.
64 bytes from github.com (192.30.252.128): icmp_seq=1 ttl=50 time=100 ms
64 bytes from github.com (192.30.252.128): icmp_seq=1 ttl=50 time=100 ms
...
```

Create a second VM and check the second interface ``vnet1`` has been set and this interface is part of the linux bridge previously defined

```
# virt-install \
--name VM2 \
--ram 2048 \
--vcpu 2 \
--network network=default \
--graphics none \
--os-type linux \
--disk path=/data/vm-images/vm1.img,size=10 \
--location /tmp/ubuntu-14.04.1-server-amd64.iso \
--extra-args 'console=ttyS0,115200n8 serial'

# virsh list --all
 Id    Name                           State
----------------------------------------------------
 2     VM1                            running
 3     VM2                            running

# ifconfig vnet1
vnet1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 fe80::fc54:ff:feb7:1ab9  prefixlen 64  scopeid 0x20<link>
        ether fe:54:00:b7:1a:b9  txqueuelen 500  (Ethernet)
        RX packets 48  bytes 7264 (7.0 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 367  bytes 22146 (21.6 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

# brctl show
bridge name     bridge id               STP enabled     interfaces
virbr0          8000.52540053d4ee       yes             virbr0-nic
                                                        vnet0
                                                        vnet1

```

Login to the VM2 and check the network communication between the VMs and outwards
```
# virsh console VM2
Connected to domain VM2
VM1 login: caldera
caldera@VM2:~$
caldera@VM2:~$ netstat -nr
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
0.0.0.0         192.168.122.1   0.0.0.0         UG        0 0          0 eth0
192.168.122.0   0.0.0.0         255.255.255.0   U         0 0          0 eth0
caldera@VM2:~$ tracepath cisco.com
 1?: [LOCALHOST]                                         pmtu 1500
 1:  192.168.122.1                                         0.265ms
 1:  192.168.122.1                                         0.255ms
 2:  10.10.10.1                                            1.952ms
     ...

caldera@VM2:~$ tracepath  192.168.122.91 (VM1)
 1?: [LOCALHOST]                                         pmtu 1500
 1:  VM1                                                   0.660ms reached
 1:  VM1                                                   0.396ms reached
     Resume: pmtu 1500 hops 1 back 1
caldera@VM2:~$

```

On the host machine, check the bridge status

```
# brctl show
bridge name     bridge id               STP enabled     interfaces
virbr0          8000.52540053d4ee       yes             virbr0-nic
                                                        vnet0
                                                        vnet1
```

### Virtual Networking in Bridged Mode
Now let to implement the Bridge mode as a custom configuration of the virtual networking based on ``libvirt``.
First of all, disable the default network based on NAT model
```
# virsh net-destroy default
# virsh net-list
 Name                 State      Autostart     Persistent
----------------------------------------------------------
```

Create an XML file for the new custom bridge network with the bridge name set to mybridge0
```
# cp /etc/libvirt/qemu/networks/default.xml /etc/libvirt/qemu/networks/bridged_network.xml
# virsh net-define /etc/libvirt/qemu/networks/bridged_network.xml
Network bridged_network defined from /etc/libvirt/qemu/networks/bridged_network.xml
# virsh net-edit bridged_network
# cat /etc/libvirt/qemu/networks/bridged_network.xml
<network>
  <name>bridged_network</name>
  <uuid>6579db1f-ca49-4e51-a116-c1342cd5d47a</uuid>
  <forward mode='bridge'/>
  <bridge name='mybridge0'/>
</network>

# virsh net-start bridged_network
Network bridged_network started

# virsh net-list
 Name                 State      Autostart     Persistent
----------------------------------------------------------
 bridged_network      active     yes           yes
```

Create a new bridge called mybridge0
```
# brctl addbr mybridge0
# brctl show
bridge name     bridge id               STP enabled     interfaces
mybridge0       8000.0024810d318d       no
```

Create a network interface file script for the bridge above. The IP Address and the other network parameters are, usually, taken from the real network interface.
```
# vi /etc/sysconfig/network-scripts/ifcfg-mybridge0
DEVICE=mybridge0
TYPE=Bridge
BOOTPROTO=none
IPADDR=10.10.10.98
PREFIX=24
GATEWAY=10.10.10.1
DNS1=8.8.8.8
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
ONBOOT=yes
DELAY=0
NM_CONTROLLED=no
```

Change the real network interface parameters, in order to be an unnumbered interface
```
# vi /etc/sysconfig/network-scripts/ifcfg-enp0s25
TYPE=Ethernet
BOOTPROTO=none
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=no
NAME=enp0s25
UUID=48d129a3-89df-4f8b-9b99-5e3518edc111
ONBOOT=yes
HWADDR=00:24:81:0D:3F:8D
BRIDGE=mybridge0
NM_CONTROLLED=no
IPV4_FAILURE_FATAL=no
```

Then, assign the real network card to the bridge and restart the network service
```
# brctl addif mybridge0 enp0s25
# brctl show
bridge name     bridge id               STP enabled     interfaces
mybridge0       8000.0024810d318d       no              enp0s25

# systemctl restart network
# ifconfig mybridge0
mybridge0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.10.10.98  netmask 255.255.255.0  broadcast 10.10.10.255
        inet6 fe80::224:81ff:fe0d:318d  prefixlen 64  scopeid 0x20<link>
        ether 00:24:81:0d:3f:8d  txqueuelen 0  (Ethernet)
        RX packets 232483  bytes 612684572 (584.3 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 212561  bytes 17767763 (16.9 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

```

Create a new virtual machine to use the bridged network

```
# virt-install \
--name VM3 \
--ram 2048 \
--vcpu 2 \
--network network=bridged_network \
--graphics none \
--os-type linux \
--disk path=/data/vm-images/vm3.img,size=10 \
--location /tmp/ubuntu-14.04.1-server-amd64.iso \
--extra-args 'console=ttyS0,115200n8 serial'

# virsh list-all
 Id    Name                           State
----------------------------------------------------
 6     VM3                            running
 -     VM1                            shut off
 -     VM2                            shut off

```

Login to the new VM and check that it is getting connectivity by the same network infrastructure as the host
```
ubuntu login: caldera
Password:
Welcome to Ubuntu 14.04.2 LTS (GNU/Linux 3.13.0-55-generic x86_64)
...
caldera@ubuntu:~$ ifconfig eth0
eth0      Link encap:Ethernet  HWaddr 52:54:00:33:cc:01
          inet addr:10.10.10.106  Bcast:10.10.10.255  Mask:255.255.255.0
          inet6 addr: fe80::5054:ff:fe33:cc01/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:81 errors:0 dropped:0 overruns:0 frame:0
          TX packets:30 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:8247 (8.2 KB)  TX bytes:3057 (3.0 KB)

caldera@ubuntu:~$ ping github.com
PING github.com (192.30.252.131) 56(84) bytes of data.
64 bytes from github.com (192.30.252.131): icmp_seq=1 ttl=50 time=101 ms
64 bytes from github.com (192.30.252.131): icmp_seq=2 ttl=50 time=100 ms
^C
caldera@ubuntu:~$ netstat -nr
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
0.0.0.0         10.10.10.1      0.0.0.0         UG        0 0          0 eth0
10.10.10.0      0.0.0.0         255.255.255.0   U         0 0          0 eth0
```

Finally, check that a new host interface called ``vnet0`` is present and is part of the bridge
```
# ifconfig vnet0
vnet0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 fe80::fc54:ff:fe33:cc01  prefixlen 64  scopeid 0x20<link>
        ether fe:54:00:33:cc:01  txqueuelen 500  (Ethernet)
        RX packets 1556  bytes 147144 (143.6 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 3892  bytes 3763246 (3.5 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

# brctl show
bridge name     bridge id               STP enabled     interfaces
mybridge0       8000.0024810d318d       no              enp0s25
                                                        vnet0
```

Please, note that different virtual networks types can be share the same physical host, e.g. NAT and Bridged mode.
Enable the default virtual network as defined previously in NAT mode

```
# virsh net-start default
Network default started
# virsh net-list
 Name                 State      Autostart     Persistent
----------------------------------------------------------
 bridged_network      active     yes           yes
 default              active     yes           yes
 
# virsh start VM1
Domain VM1 started

# virsh start VM2
Domain VM2 started

# virsh list --all
 Id    Name                           State
----------------------------------------------------
 6     VM3                            running
 7     VM1                            running
 8     VM2                            running
```

### Virtual Networking in Routed Mode
Routed mode is a variation of the virtual networking with NAT mode. In routing mode, the ``virbr0`` interface acts as routing interface without NAT. The DHCP can be enabled or not, depending on the requirements.

Create the envinronment for the routed virtual networking

```
# cat /etc/libvirt/qemu/networks/router.xml
<network>
  <name>router</name>
  <uuid>186ffd11-ce2d-452a-9970-baf07b723de8</uuid>
  <forward mode='route'/>
  <bridge name='virbr1' stp='on' delay='0'/>
  <mac address='52:54:00:56:e6:21'/>
  <ip address='192.168.132.1' netmask='255.255.255.0'>
  </ip>
</network>

# virsh net-define /etc/libvirt/qemu/networks/router.xml
Network router defined from /etc/libvirt/qemu/networks/router.xml

# virsh net-edit router
Network router XML configuration not changed.

# virsh net-start router
Network router started

# virsh net-autostart router
Network router marked as autostarted

# virsh net-list
 Name                 State      Autostart     Persistent
----------------------------------------------------------
 default              active     yes           yes
 router               active     yes           yes

# ifconfig virbr1
virbr1: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 192.168.132.1  netmask 255.255.255.0  broadcast 192.168.132.255
        ether 52:54:00:46:e6:21  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

# brctl show virbr1
bridge name     bridge id               STP enabled     interfaces
virbr1          8000.52540046e621       yes             virbr1-nic

# netstat -nr
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
0.0.0.0         10.10.10.1      0.0.0.0         UG        0 0          0 enp0s25
10.10.10.0      0.0.0.0         255.255.255.0   U         0 0          0 enp0s25
192.168.122.0   0.0.0.0         255.255.255.0   U         0 0          0 virbr0
192.168.132.0   0.0.0.0         255.255.255.0   U         0 0          0 virbr1
```

The VMs must be created with static IP address in the same network of the ``virbr1`` and default gateway the same interface. Since no NAT is performed by the virtual networking, make sure the network env is aware of the new network (e.g. put static routes).

### Spawning multiple hosts
Linux virtual networking can spawn multiple hosts. Let to build a virtual networking setup of two hosts whith several VMs running on both. Only one host has a network interface connected to internet, and so it acts as a NAT gateway for both the hosts. The two hosts are bridged togeter using a secondary network interface.

```
 caldera02            caldera03
----------            ----------
|   VM1  |            |   VM1  |
|   VM2  |============|   VM2  |
----------            ----------
    ||
 Internet
``` 

On the gateway host, caldera02, enable the default virtual networking in NAT mode
``` 
# virsh net-list
 Name                 State      Autostart     Persistent
----------------------------------------------------------
 default              active     yes           yes

# cat /etc/libvirt/qemu/networks/default.xml
<network>
  <name>default</name>
  <uuid>f6514662-26e5-431e-8544-fd216bed9312</uuid>
  <forward mode='nat'/>
  <bridge name='virbr0' stp='on' delay='0'/>
  <mac address='52:54:00:53:d4:ee'/>
  <ip address='192.168.122.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.122.2' end='192.168.122.254'/>
    </dhcp>
  </ip>
</network>
``` 

Create and start two VMs and check the bridge status
```
# virsh start VM1
# virsh start VM2
# virsh list --all
 Id    Name                           State
----------------------------------------------------
 16    VM1                            running
 17    VM2                            running
 
# brctl show
bridge name     bridge id               STP enabled     interfaces
virbr0          8000.52540053d4ee       yes             virbr0-nic
                                                        vnet2
                                                        vnet3
``` 

This is a simple virtual networking in NAT mode where virtual interfaces of the virtual machines, ``vnet2`` and ``vnet3`` are bridged togeter with the bridge interface ``virbr0-nic``.

On the second host, caldera03, enable the default virtual networking in isolated mode, i.e. the virtual machines can only talk to each other.

```
# virsh net-list
 Name                 State      Autostart     Persistent
----------------------------------------------------------
 default              active     yes           yes

cat /etc/libvirt/qemu/networks/default.xml
<network>
  <name>default</name>
  <uuid>e8849d67-243a-4b0f-83ea-bc2b447580cf</uuid>
  <bridge name='virbr0' stp='on' delay='0'/>
  <mac address='52:54:00:22:1b:5f'/>
</network>
```

The missing ``<forward mode=''/>`` tag means the virtual networking is in isolated mode. Also, the missing ``<ip address='' netmask=''>`` tag means the virtual networking interface is unnumbered and the VMs will not receive any IP address.

Create and start two VMs and check the bridge
```
# virsh start VM1
# virsh start VM2
# virsh list --all
 Id    Name                           State
----------------------------------------------------
 16    VM1                            running
 17    VM2                            running

# brctl show
bridge name     bridge id               STP enabled     interfaces
virbr0          8000.c46e1f0099cd       yes             virbr0-nic
                                                        vnet0
                                                        vnet1
 
```

Now we have to bridge the two hosts with a dedicated network interface ``ens2``.
On the gateway host, caldera02, make ``ens2`` interface unnumbered and add it to the bridge
```
# cat /etc/sysconfig/network-scripts/ifcfg-ens2
TYPE=Ethernet
BOOTPROTO=none
IPV4_FAILURE_FATAL=no
IPV6INIT=no
NAME=ens2
UUID=452ee132-35a6-4c36-915b-91b85902f2b9
ONBOOT=yes
NM_CONTROLLED=no

#  brctl addif virbr0 ens2
#  brctl show
bridge name     bridge id               STP enabled     interfaces
virbr0          8000.52540053d4ee       yes             ens2
                                                        virbr0-nic
                                                        vnet2
                                                        vnet3
```

Do the same on the second host, caldera03
```
# cat /etc/sysconfig/network-scripts/ifcfg-ens2
TYPE=Ethernet
BOOTPROTO=none
IPV4_FAILURE_FATAL=no
IPV6INIT=no
NAME=ens2
UUID=1de92d8b-1c83-4bf5-be22-171f65b1b66b
ONBOOT=yes
NM_CONTROLLED=no

#  brctl addif virbr0 ens2
# brctl show
bridge name     bridge id               STP enabled     interfaces
virbr0          8000.c46e1f0099cd       yes             ens2
                                                        vnet0
                                                        vnet1
```

On the host caldera03, login to VM and check the virtual machine is getting the IP address from the gateway host, caldera02. Then check the tracepath to the internet
```
# virsh console VM1
Connected to domain VM1
Escape character is ^]

caldera@ubuntu:~$ ifconfig eth0
eth0      Link encap:Ethernet  HWaddr 52:54:00:36:5a:aa
          inet addr:192.168.122.101  Bcast:192.168.122.255  Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:4975 errors:0 dropped:0 overruns:0 frame:0
          TX packets:3031 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:6741689 (6.7 MB)  TX bytes:368948 (368.9 KB)

caldera@ubuntu:~$ tracepath cisco.com
 1?: [LOCALHOST]                                         pmtu 1500
 1:  192.168.122.1                                         0.686ms
 1:  192.168.122.1                                         0.670ms
 2:  10.10.10.1                                            4.425ms

```
