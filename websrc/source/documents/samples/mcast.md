---
layout: "documents"
page_title: "Examples"
sidebar_current: "samples-mcast"
description: |-
  Examples
---

# Multicast Application Examples
This page illustrates how to validate multicast in a Contiv network.

## Multicast Between Containers
In this demo you run two containers, one on each VM. You create a VLAN network
and run a sender and a receiver multicast application. The demo uses a
multicast application from `https://github.com/leslie-wang/py-multicast-example`.

### Step 1: Download the Example
Pull the Contiv Network workspace from GitHub:

```
$ git clone https://github.com/contiv/netplugin
$ cd netplugin
```

### Step 2: Create Demo VMs and Network

The following commands start the VMs, log in to Node 1, and create a multicast-enabled network:

```
$ make demo
$ vagrant ssh netplugin-node1
$ netctl net create contiv-net --encap=vlan --subnet=20.1.1.0/24 --gateway=20.1.1.254 --pkt-tag=1010
```

### Step 3: Start the Sender
The following commands run a Docker container in the network and start a multicast sender application:

```
$ docker pull qiwang/centos7-mcast
$ docker run -it --name=msender --net=contiv-net qiwang/centos7-mcast /bin/bash
root@9f4e7fd418c5:/# cd /root
root@9f4e7fd418c5:/# ./mcast.py -s -i eth0
```

### Step 4: Run the Receiver

The following commands log in to Node 2, start a Docker container in the network, and launch the multicast receiver:

```
$ vagrant ssh netplugin-node2
$ docker pull qiwang/centos7-mcast
$ docker run -it --name=mreceiver --net=contiv-net qiwang/centos7-mcast /bin/bash
root@564f7f4424c1:/# cd /root
root@564f7f4424c1:/# ./mcast.py -i eth0

('20.1.1.3', 35624)  '1453881422.973572'
('20.1.1.3', 35624)  '1453881423.977554'
('20.1.1.3', 35624)  '1453881424.978941'
```

(In the output from the receiver, `20.1.1.3` is the IP of the `msender` container.)


## Multicast between a Container and a VM

In this demo you run a sender and a receiver multicast application on a host VM and a container respectively.

### Step 1: Create the VMs
The following commands start the VMs, log in to Node 1, and create a multicast-enabled network:

```
$ make demo
$ vagrant ssh netplugin-node1
$ netctl net create contiv-net --encap=vlan --subnet=20.1.1.0/24 --gateway=20.1.1.254 --pkt-tag=1010
```

### Step 2: Create a Port 
The following commands create a port on the OVS using the network tag of the new network:

```
$ sudo ovs-vsctl add-port contivVlanBridge inb01 -- set interface inb01 type=internal
$ sudo ovs-vsctl set port inb01 tag=1010
$ sudo ifconfig inb01 30.1.1.8/24
```

### Step 3: Start the Sender
The following command starts a multicast sender application:

```
$ ./mcast.py -s -i inb01
```

### Step 4: Run the Receiver
The following commands log in to Node 2, start a docker container in the new network, and launch a container-based multicast receiver:

```
$ vagrant ssh netplugin-node2
$ docker pull qiwang/centos7-mcast
$ docker run -it --name=mreceiver --net=contiv-net qiwang/centos7-mcast /bin/bash
root@426b8cdbf5f8:/# cd /root
root@426b8cdbf5f8:/# ./mcast.py -i eth0

('30.1.1.8', 35678)  '1453882966.102203'
('30.1.1.8', 35678)  '1453882967.120764'
('30.1.1.8', 35678)  '1453882968.12215'
```
