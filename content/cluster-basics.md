## Cluster Basics
A cluster is two or more computers (cluster members) that work together to perform a task, for example, provide high availability of a given service. High availability clusters provide highly available services by eliminating single points of failure and by failing over services from one cluster member to another in case a node becomes inoperative.

Typically, services in a high availability cluster maintain data integrity as one cluster member takes over control of a service from another cluster member. Node failures in a high availability cluster are not visible from clients outside the cluster.

In the Linux world, there are many cluster tools to achieve High Availability of a resource. The most used is **Pacemaker**. A cluster configured with Pacemaker comprises separate component daemons that monitor cluster membership, scripts that manage the services, and resource management subsystems that monitor the resources. The following components form the Pacemaker architecture:

1. **Cluster Information Base**: the Pacemaker information daemon distributes and synchronizes the cluster configuration and status information from the Designated Coordinator (DC) of the cluster to all other cluster members. The DC is one cluster member designated to store the cluster state.

2. **Cluster Resource Management Daemon**: cluster resources managed by this component can be queried by client systems, moved, instantiated, and changed when needed. Each cluster node also includes a local resource manager daemon that acts as an interface between Cluster Resource Manager daemon and the resource itself. The local resource manager passes commands from Cluster Resource Manager to agents, such as starting and stopping and relaying resurce status information.

3. **Fencing Manager**: often deployed in conjunction with a power supply switch, this component acts as a cluster resource in Pacemaker that processes fence requests, forcefully powering down nodes and removing them from the cluster to ensure data integrity. Pacemaker use a fencing technique called **STONITH** (Shoot The Other Node In The Head) intended to prevent data corruption caused by faulty nodes in a cluster that are unresponsive but still accessing application data (the so called "Split Brain Scenario").

### Install a simple Cluster
Pacemaker requires a messaging layer daemon, called **Corosync** that provides a cluster membership and closed communication model for creating replicated state machines, on top of which Pacemaker can run. Corosync can be seen as the underlying system that connects the cluster nodes together, while Pacemaker monitors the cluster and takes action in the event of a failure. In addition, we are going to use **PCS**, a command line interface that interacts with both Corosync and Pacemaker.

This example will be also used to explain the basic concepts of Linux Clustering. 

                                      |
    +----------------------+          |          +----------------------+
    | Node01               |          |          | Node02               |
    | holly.noverit.com    +----------+----------+ benji.noverit.com    |
    | 10.10.10.22          |                     | 10.10.10.24          |
    +----------------------+                     +----------------------+

Install, start and enable Pacemaker and PCS on both the nodes. Because Corosync is a dependency to Pacemaker, it's usually a better idea to simply install Pacemaker and let the system decide which Corosync version should be installed.

    [root@holly ~]# yum -y install pacemaker
    [root@holly ~]# yum -y install pcs
    [root@holly ~]# systemctl start pcsd
    [root@holly ~]# systemctl enable pcsd

    [root@benji ~]# yum -y install pacemaker
    [root@benji ~]# yum -y install pcs
    [root@benji ~]# systemctl start pcsd
    [root@benji ~]# systemctl enable pcsd

Pacemaker need to communicate beween nodes, enable the port firewall on each node, which by default is 2224 over TCP. Otherwise, disable the firewall if you are working in a secure setup.

    [root@holly ~]# systemctl stop firewalld
    [root@holly ~]# systemctl disable firewalld
    [root@benji ~]# systemctl stop firewalld
    [root@benji ~]# systemctl disable firewalld

The PCS utility creates a user during installation, named ``hacluster``, with a disabled password. We need to define a password for this user on both servers. This will enable PCS to perform tasks such as synchronizing the Corosync configuration on multiple nodes, as well as starting and stopping the cluster.

    [root@holly ~]# passwd hacluster
    Changing password for user hacluster
    [root@benji ~]# passwd hacluster
    Changing password for user hacluster

Use the same password on both servers. We are going to use this password to configure the cluster in the next step. Please, note that the user ``hacluster`` has no interactive shell or home directory associated with its account, which means it's not possible to log into the server using its credentials.

Only on a node of the cluster, authenticate the cluster nodes

    [root@holly ~]# pcs cluster auth holly benji
    Username: hacluster
    Password:
    holly: Authorized
    benji: Authorized

From the same node, generate the Corosync configuration

    [root@holly ~]# pcs cluster setup --name mycluster holly benji
    Shutting down pacemaker/corosync services...
    Redirecting to /bin/systemctl stop  pacemaker.service
    Redirecting to /bin/systemctl stop  corosync.service
    Killing any remaining services...
    Removing all cluster configuration files...
    holly: Succeeded
    benji: Succeeded
    Synchronizing pcsd certificates on nodes holly, benji...
    benji: Success
    holly: Success
    Restaring pcsd on the nodes in order to reload the certificates...
    benji: Success
    holly: Success

This will generate a cluster configuration file (i.e. the cluster information base) located at ``/etc/corosync/corosync.conf`` based on the parameters provided to the cluster setup command:

    [root@holly ~]# cat /etc/corosync/corosync.conf
    totem {
        version: 2
        secauth: off
        cluster_name: mycluster
        transport: udpu
    }
    nodelist {
        node {
            ring0_addr: holly
            nodeid: 1
        }
        node {
            ring0_addr: benji
            nodeid: 2
        }
    }
    quorum {
        provider: corosync_votequorum
        two_node: 1
    }
    logging {
        to_logfile: yes
        logfile: /var/log/cluster/corosync.log
        to_syslog: yes
    }

Start and enable the cluster

    [root@holly ~]# pcs cluster start --all
    benji: Starting Cluster...
    holly: Starting Cluster...

    [root@holly ~]# pcs cluster enable --all
    holly: Cluster Enabled
    benji: Cluster Enabled


Check the status of the cluster

    [root@holly ~]# pcs status
    Cluster name: mycluster
    WARNING: no stonith devices and stonith-enabled is not false
    Last updated: Sat Jul 16 17:20:14 2016
    Stack: corosync
    Current DC: holly (version 1.1.13-10.el7_2.2-44eb2dd) - partition with quorum
    2 nodes and 0 resources configured
    Online: [ benji holly ]
    Full list of resources: -
    PCSD Status:
        holly: Online
        benji: Online
    Daemon Status:
        corosync: active/enabled
        pacemaker: active/enabled
        pcsd: active/enabled

Some interesting info:

  1. The Designated Coordinator (DC) is the node holly where from we configured the cluster
  2. There are only 2 nodes onlyne and no resurces
  3. The name of the cluster is "mycluster"
  4. All daemons: corosync, pacemaker and pcsd are active and enabled
  5. Fencing (stonith) is enabled but no fencing devices are configured

Confirm that both nodes joined the cluster by running the following command on any of the servers

    [root@holly ~]# pcs status corosync
    Membership information
    ----------------------
        Nodeid      Votes Name
             1          1 holly (local)
             2          1 benji
    [root@holly ~]#

Because our cluster does not manage shared data resources, there is no risk to have a Split Brain Scenario and so we are going to disable fencing

    [root@holly ~]# pcs property set stonith-enabled=false

Cluster quorum as a concept (see later) makes no sense in a two-node scenario, because you only have it when more than half the nodes are available, so we'll disable it too.

    [root@holly ~]# pcs property set no-quorum-policy=ignore

To see a recap of the Cluster properties

    [root@benji ~]# pcs property list
    Cluster Properties:
     cluster-infrastructure: corosync
     cluster-name: mycluster
     dc-version: 1.1.13-10.el7_2.2-44eb2dd
     have-watchdog: false
     no-quorum-policy: ignore
     stonith-enabled: false

Cluster nodes should not be halted as other standard nodes. It's always a best practice to shutdown the cluster first and then shutdown the system.

To stop the cluster on a signle node

    [root@benji ~]# pcs cluster stop
    Stopping Cluster (pacemaker)... Stopping Cluster (corosync)...
    [root@benji ~]# pcs cluster status
    Error: cluster is not currently running on this node

Or on all nodes of the cluster

    [root@holly ~]# pcs cluster stop --all
    holly: Stopping Cluster (pacemaker)...
    benji: Stopping Cluster (pacemaker)...
    benji: Stopping Cluster (corosync)...
    holly: Stopping Cluster (corosync)...
    [root@holly ~]#
  
### Add a resource to the Cluster
Lets add a cluster service, we'll choose one doesn't require too much configuration and works everywhere to make things easy.

Install and configure an HTTP Server on both the nodes. Note: not need to start/enable the service.

    [root@benji ~]# yum install -y httpd
    [root@benji ~]# echo "Hello Benji" > /var/www/html/index.html
    [root@holly ~]# yum install -y httpd
    [root@holly ~]# echo "Hello Holly" > /var/www/html/index.html

Add the HTTP Server as resource of the cluster

    [root@benji ~]# pcs resource create HTTPServer ocf:heartbeat:apache \
    > configfile=/etc/httpd/conf/httpd.conf \
    > op monitor interval=1min

The name of the resource is ``HTTPServer`` of type ``ocf:heartbeat:apache``. The type defined for a resource tell the cluster which script to use for the resource, the provider of the script and what standards it conforms to. In that case, the standard is **Open Cluster Framework**. The command tells also Pacemaker to check the health of this service every 60 seconds by calling the agent's monitor action.

Add a Virtual IP address as second resource of the cluster. This IP Address will be used by clients of the cluster to access the HTTP Server resource

    [root@benji ~]# pcs resource create VirtualIP ocf:heartbeat:IPaddr2 \
    > ip=10.10.10.23 \
    > cidr_netmask=24 \
    > op monitor interval=30s

The name of the resource is ``VirtualIP`` of type ``ocf:heartbeat:IPaddr2``. The command tells also Pacemaker to check the health of this service every 30 seconds by calling the agent's monitor action. The Virtual IP resource binds the IP address specified in the command above to the network interface of the node owning the Virtual IP resources. This Virtual IP is floating from one node to the other, depending on the status of the node itself:

    [root@benji ~]# ip addr show ens32
    2: ens32: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
        link/ether 00:0c:29:20:d2:dd brd ff:ff:ff:ff:ff:ff
        inet 10.10.10.24/24 brd 10.10.10.255 scope global ens32
           valid_lft forever preferred_lft forever
        inet 10.10.10.23/24 brd 10.10.10.255 scope global secondary ens32
           valid_lft forever preferred_lft forever
           
    [root@holly ~]# ip addr show ens32
    2: ens32: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
        link/ether 00:0c:29:77:68:56 brd ff:ff:ff:ff:ff:ff
        inet 10.10.10.22/24 brd 10.10.10.255 scope global ens32
           valid_lft forever preferred_lft forever

Set that HTTPServer and VirtualIP are always on a same node

    [root@benji ~]# pcs constraint colocation add HTTPServer with VirtualIP

Set that the order of starting is VirtualIP first and then HTTPServer. This is required to assure there is always an IP Address where to send client's requests

    [root@holly ~]# pcs constraint order VirtualIP then HTTPServer
    Adding VirtualIP HTTPServer (kind: Mandatory) (Options: first-action=start then-action=start)

See the status of both the resources

    [root@benji ~]# pcs status resources
     VirtualIP      (ocf::heartbeat:IPaddr2):       Started by benji
     HTTPServer     (ocf::heartbeat:apache):        Started by holly

and resources constraints

    [root@holly ~]# pcs constraint
    Location Constraints:
    Ordering Constraints:
      start VirtualIP then start HTTPServer (kind:Mandatory)
    Colocation Constraints:
      HTTPServer with VirtualIP (score:INFINITY)

Now we can access the HTTP Server from a web client by pointing to the Virtual IP Address 10.10.10.23

    [stack@director ~]$ curl http://10.10.10.23
    Hello Benji

To test Cluster failover, stop current active node manually 

    [root@benji html]# pcs cluster stop
    Stopping Cluster (pacemaker)... Stopping Cluster (corosync)...
    

and make sure resource will switch to the other node

    [stack@director ~]$ curl http://10.10.10.23
    Hello Holly

### Accessing the cluster management form a Web GUI
Cluster management is possible also via a Web GUI. Point the browser to the primary member node and login as the ``hacluster`` user

    https://<primary_node_ip>:2224
