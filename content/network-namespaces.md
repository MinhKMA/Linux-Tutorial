### Linux Network Namespaces
Dedicated networking devices use Virtual Routing and Forwarding (VRF), meaning that more than one virtual router (Layer 3 forwarding instance) can be run on the same physical device. In the Linux virtual networking space, the network namespaces allow separate instances of network interfaces and routing tables to operate independent of each other.

#### Basic operations on namespaces
In Linux, yust be root for all operations which change the configuration of the network stack.

Creating a network namespace

    [root@centos-01 ~]# ip netns add Blue
    [root@centos-01 ~]# ip netns list
    Blue
    [root@centos-01 network-scripts]# ll /var/run/netns/
    total 0
    -r--r--r-- 1 root root 0 Feb 11 16:29 Blue

Each network namespace has its own loopback interface, its own routing table and its own iptables setup providing nat and filtering. 

    [root@centos-01 ~]# ip netns exec Blue ip addr list
    1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00

Make sure to bring up that interface before to operate with the network namespace

    [root@centos-01 ~]# ip netns exec Blue ip link set dev lo up
    [root@centos-01 ~]# ip netns exec Blue ifconfig
    lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
            inet 127.0.0.1  netmask 255.0.0.0
            inet6 ::1  prefixlen 128  scopeid 0x10<host>
            loop  txqueuelen 0  (Local Loopback)
            RX packets 0  bytes 0 (0.0 B)
            RX errors 0  dropped 0  overruns 0  frame 0
            TX packets 0  bytes 0 (0.0 B)
            TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

Network namespaces offer in addition the capability to run processes within the network namespace. For example, run a bash session in the Blue namespace

    [root@centos-01 ~]# ip netns exec Blue bash
    [root@centos-01 ~]# ifconfig
    lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
            inet 127.0.0.1  netmask 255.0.0.0
            inet6 ::1  prefixlen 128  scopeid 0x10<host>
            loop  txqueuelen 0  (Local Loopback)
            RX packets 0  bytes 0 (0.0 B)
            RX errors 0  dropped 0  overruns 0  frame 0
            TX packets 0  bytes 0 (0.0 B)
            TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
    [root@centos-01 ~]# netstat -nr
    Kernel IP routing table
    Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
    [root@centos-01 ~]# exit

Delete the namespace

    [root@centos-01 ~]# ip netns add Yellow
    [root@centos-01 ~]# ip netns list
    Yellow
    Blue
    [root@centos-01 ~]# ip netns delete Yellow
    [root@centos-01 ~]# ip netns list
    Blue

#### Add interfaces to network namespaces
To connect a network namespace to the outside world, attach a virtual interface to the “default” or “global” namespace where physical interfaces exist. To accomplish this, let's to create a couple of virtual interfaces, called ``vetha`` and ``vethb``

    [root@centos-01 ~]# ip link add vetha type veth peer name vethb

Attach ``vethb`` to the Blue namespace 

    [root@centos-01 ~]# ip link set vethb netns Blue
    [root@centos-01 ~]# ip netns exec Blue ip link set dev vethb up
    [root@centos-01 ~]# ip netns exec Blue ifconfig
    lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
            inet 127.0.0.1  netmask 255.0.0.0
            inet6 ::1  prefixlen 128  scopeid 0x10<host>
            loop  txqueuelen 0  (Local Loopback)
    vethb: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
            ether 7e:e4:29:bc:9c:67  txqueuelen 1000  (Ethernet)

Virtual network interface ``vetha`` remain attacched to the global namespace

    [root@centos-01 ~]# ip link set dev vetha up
    [root@centos-01 ~]# ifconfig
    ens32: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
            inet 10.10.10.21  netmask 255.255.255.0  broadcast 10.10.10.255
            inet6 fe80::20c:29ff:fe1e:6bf1  prefixlen 64  scopeid 0x20<link>
            ether 00:0c:29:1e:6b:f1  txqueuelen 1000  (Ethernet)
    lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
            inet 127.0.0.1  netmask 255.0.0.0
            inet6 ::1  prefixlen 128  scopeid 0x10<host>
            loop  txqueuelen 0  (Local Loopback)
    vetha: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
            inet6 fe80::e899:ceff:fef6:3010  prefixlen 64  scopeid 0x20<link>
            ether ea:99:ce:f6:30:10  txqueuelen 1000  (Ethernet)

Configure the virtual interface in global network namespace 

    [root@centos-01 ~]# ip addr add 192.168.100.1/24 dev vetha
    [root@centos-01 ~]# route
    Kernel IP routing table
    Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
    default         gateway         0.0.0.0         UG    100    0        0 ens32
    10.10.10.0      0.0.0.0         255.255.255.0   U     100    0        0 ens32
    192.168.100.0   0.0.0.0         255.255.255.0   U     0      0        0 vetha
    [root@centos-01 ~]#
    
and in the Blue network namespace

    [root@centos-01 ~]# ip netns exec Blue ip addr add 192.168.100.2/24 dev vethb
    [root@centos-01 ~]# ip netns exec Blue route
    Kernel IP routing table
    Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
    192.168.100.0   0.0.0.0         255.255.255.0   U     0      0        0 vethb
    [root@centos-01 ~]#

Both the namespaces, Blue and global are now reachable each other via virtual network interfaces

    [root@centos-01 ~]# ping 192.168.100.2
    PING 192.168.100.2 (192.168.100.2) 56(84) bytes of data.
    64 bytes from 192.168.100.2: icmp_seq=1 ttl=64 time=0.041 ms
    64 bytes from 192.168.100.2: icmp_seq=2 ttl=64 time=0.029 ms
    ^C
    [root@centos-01 ~]# ip netns exec Blue ping 192.168.100.1
    PING 192.168.100.1 (192.168.100.1) 56(84) bytes of data.
    64 bytes from 192.168.100.1: icmp_seq=1 ttl=64 time=0.034 ms
    64 bytes from 192.168.100.1: icmp_seq=2 ttl=64 time=0.039 ms
    ^C

But they are completly separated routing entities

    [root@centos-01 ~]# ip netns exec Blue ping 10.10.10.1
    connect: Network is unreachable
    [root@centos-01 ~]#
    [root@centos-01 ~]# ping 10.10.10.1
    PING 10.10.10.1 (10.10.10.1) 56(84) bytes of data.
    64 bytes from 10.10.10.1: icmp_seq=1 ttl=64 time=0.545 ms
    64 bytes from 10.10.10.1: icmp_seq=2 ttl=64 time=0.369 ms
    ^C
