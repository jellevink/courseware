<!--in-progress, may split out/remove some/all vpn and onion stuff-->

# IP address

<!--TOC_START-->
### Contents
- [Overview](#overview)
- [Usage](#usage)
	- [IP addresses](#ip-addresses)
		- [IPv4](#ipv4)
		- [Reserved IPv4 addresses](#reserved-ipv4-addresses)
		- [IPv6](#ipv6)
		- [`localhost`](#localhost)
	- [ARP](#arp)
	- [Subnets](#subnets)
	- [Internal & External IPs](#internal--external-ips)
	- [Routing](#routing)
- [Viewing your IP address](#viewing-your-ip-address)
	- [Windows](#windows)
	- [Linux](#linux)
- [Changing IP address](#changing-ip-address)
	- [VPNs](#vpns)
		- [ISP tracking](#isp-tracking)
		- [Jurisdiction tracking](#jurisdiction-tracking)
		- [Enemies of the Internet](#enemies-of-the-internet)
	- [Onion routing](#onion-routing)
		- [Tor](#tor)
	- [Why?](#why)
- [Tasks](#tasks)
	- [Find your current IP address](#find-your-current-ip-address)
	- [Change your IP address](#change-your-ip-address)
		- [Installing Opera VPN](#installing-opera-vpn)

<!--TOC_END-->
## Overview
Nearly all networks are based on the **Internet Protocol Suite (TCP/IP)**, the set of protocols that allow for a device to gain access to the Internet.

Contained within this suite is the **Transmission Control Protocol**, which facilitates connections between nodes within a network, and the **Internet Protocol**, which has the task of delivering data from a source node to a destination node.

The biggest difference between the two is that while TCP is a connection protocol - in charge of ensuring that connections are made before data is transferred - IP is connectionless, and does concern itself with whether information it attempts to deliver is received or not.

IP also encapsulates data into **datagrams**. Irrespective of the content of the data that is being transmitted, IP splits it into one or more datagrams, each containing a **header** and a **payload**:

|Type|Information| 
|-|-|
|Datagram|basic transfer unit - contains header; payload|
|Header|contains source IP address; destination IP address; routing metadata|
|Payload|content of data to be transported|

IP routes these datagrams from a source host interface to a destination host interface, across one or more IP networks.

Essentially, if TCP is the postal service, IP is the postman. TCP makes sure that the connections are in place for deliveries to take place, but all IP does is delivers stuff, and doesn't care if that stuff ever gets where it's supposed to be or not.

## Usage
The Internet Protocol requires five specific things in order to work:
1. An **IP address**, assigned to whichever device is attempting to transfer information over the Internet
2. **ARP**, a mechanism for mapping hardware-specific MAC addresses to networked IP addresses
3. A clearly-defined local network (the `localhost`)
4. A gateway from the local **internal** network to **external** networks
5. A **routing** mechanism for moving data around both local and external networks

This section will discuss each of these characteristics.

### 1. IP addresses
In the same way that a postal system uses addresses for posting letters, IP makes use of addresses, which are assigned to every Internet-connected device, in order to send information from a source node to a destination node.

Each networked system must be identified as unique, so that when information is sent via IP, we can be sure of where we are sending that information - even if it is not received.

Addressing solves this issue by using a unique **IP address** which distinguishes one node from any other.

The **Internet Engineering Task Force (IETF)** regulates the usage of IP addresses.

#### IPv4
The current standard of IP addresses, **IPv4**, are made up of a 32-bit address, and can be expressed in four octets of binary:
```text
00001010.00001100.00111000.00010111
```

IPv4 addresses are designed to be human-readable, thus, they are usually written in dotted decimal:
```text
10.12.56.23
```

Allocating unique IP addresses is usually done automatically with the **Dyanmic Host Configuration Protocol (DHCP)**.

#### Reserved IPv4 addresses
As each section of an IPv4 address can only be numbered from 0 to 255, they are limited to 4.3 billion unique combinations - this may seem like a lot, but several restrictions apply: 

The following IPv4 ranges are designated for private purposes, namely local communications within a private network, and therefore cannot be used on the Internet:

|Range|
|-|
|`10.0.0.0 - 10.255.255.255`|
|`172.16.0.0 - 172.31.255.255`|
|`192.168.0.0 - 192.168.255.255`|

There are also other restrictions on IPv4 addresses - some important reserved address blocks are:

|Range|Usage|
|-|-|
|`0.x.x.x`|software use only|
|`127.x.x.x`|`localhost`|
|`224.0.0.0 - 239.255.255.255`|IP multicasting, e.g. for streaming media|
|`240.0.0.0 - 255.255.255.254`|reserved for future usage|
|`255.255.255.255`|local broadcast|

#### IPv6
Eventually, we will run out of IP addresses.

Given that the Internet has been in continuous usage for decades, the problem of IP address exhaustion has been prevalent since the late-1980s.

IPv6 was developed, and is currently in deployment, in preparation for the time when IPv4 addresses deplete entirely.

As a result of IPv4's depletion, IPv6 upgrades the amount of data stored in an IP address from IPv4's 32-bit system to 128-bit - allowing for 340.3 undecillion unique combinations.

This can be expressed in eight colonned groups of four hexadecimal digits, and allows for concatenation of zero values by double-colonning - thus, the same address can be written in multiple forms:
```text
2001:0db8:0000:0000:8a2e:0370:7334:0001

2001:db8:0:0:82ae:370:7734:1

2001:db8::8a2e:370:7334::1
```

IPv4 addresses can also be expressed in IPv6 format - the IPv4 address `10.12.56.23` becomes:
```text
0:0:0:0:0:ffff:a0c:3817

::ffff:a0c:3817
```

As 17% of IPv4 addresses are currently in-use (as of 2011), it is widely considered that IPv4 will be used alongside IPv6 for the forseeable future.

### 2. ARP
Every networked device contains at least one physical **Network Interface Card (NIC)**, such for Ethernet or wireless connections, providing a physical connection between that device and a network.

NICs can generate a **broadcast domain** between devices if they are all connected to the same system - the numbers in the below diagram signifies a different broadcast domain:

```text
                 1                  2
          ┌────────────[Router]────────────┐
       [Switch]           | 3           [Switch]
    ┌─────┴─────┐       [Node]       ┌─────┴─────┐
  [Node]      [Node]               [Node]      [Node]
```

Each NIC has a unique [MAC address](https://github.com/bob-crutchley/courseware/tree/master/topics/networking/modules/mac-address/) burned into it by its manufacturer, which are used by Ethernet and wireless connections to communicate between network devices.

As these NICs only understand MAC addresses, the **Address Resolution Protocol (ARP)** is used to resolve MAC to IP, or vice-versa.

ARP uses broadcast packets to all computers on the Ethernet segment to discover MAC addresses.

Here, **Node 1** and **Node 2** are both present on the same network:

```text
 ┌────────[Node 1]─────────┐						        			┌────────[Node 2]─────────┐
 │ MAC: 12:6e:eb:de:b3:ed  │────────────────────────────────────────────│ MAC: 12:6f:56:c0:c4:c1  │
 │ IP: 10.12.2.73          │											│ IP: 10.12.2.1           │
 └─────────────────────────┘							        		└─────────────────────────┘
```

If **Node 1** tries to find the MAC address associated with the IPv4 address `10.12.2.1`, it will broadcast an **ARP request** along the network until it finds the device it needs:

```text
 ┌────────[Node 1]─────────┐											┌────────[Node 2]─────────┐
 │ MAC: 12:6e:eb:de:b3:ed  │────────────>───────────────────>───────────│ MAC: 12:6f:56:c0:c4:c1  │
 │ IP: 10.12.2.73          │	   Source MAC: 12:6e:eb:de:b3:ed		│ IP: 10.12.2.1           │
 └─────────────────────────┘	   Source IP:  10.12.2.73  	       		└─────────────────────────┘
								   Target MAC: 00:00:00:00:00:00
								   Target IP:  10.12.2.1
```

Once **Node 2** receives this request, it will send an **ARP response** if the Target IP matches its IP address:

```text
 ┌────────[Node 1]─────────┐											┌────────[Node 2]─────────┐
 │ MAC: 12:6e:eb:de:b3:ed  │────────────<───────────────────<───────────│ MAC: 12:6f:56:c0:c4:c1  │
 │ IP: 10.12.2.73          │	   Source MAC: 12:6f:56:c0:c4:c1		│ IP: 10.12.2.1           │
 └─────────────────────────┘	   Source IP:  10.12.2.1	   	   		└─────────────────────────┘
								   Target MAC: 12:6e:eb:de:b3:ed
								   Target IP:  10.12.2.73
```

#### Subnets
Due to performance limitations of broadcast domains, networks are split into **subnets**.​

To facilitate this, IPv4 addresses are split into three sections: a *network ID*, a *host ID* and a *subnet ID*.

A 32-bit **subnet mask** is used to identify the network which a particular node is on - depending on the network, certain sections of the mask are either static or variable.

The IP address and subnet mask are used in tandem to determine the network which a given IP address is connected to.

#### Network Classes
The first IPv4 addresses were limited to 254 unique allocations. As these depleted, IPv4 was further subdivided through subnets into a **network class** structure.

Depending on the network class, these subnets could take different forms:
|Class|Octet (binary)|Network ID|Host ID|Network ID|# of networks|# of addresses|
|-|-|-|-|-|-|-|
|A|`0XXXXXXX`|`0-127`|`a`|`b.c.d`|2&#8311;|2&sup2;&#8308;|
|B|`10XXXXXX`|`128-191`|`a.b`|`c.d`|2&sup1;&#8308;|2&sup1;&#8310;|
|C|`110XXXXX`|`192-223`|`a.b.c`|`d`|2&sup2;&sup1;|2&#8312;|
|D|`1110XXXX`|`223-254`|`a.b.c.d`|`(experimental)`|2&sup2;&sup3;|2&#8313;|

For instance, the class B IPv4 address `10.0.0.0` would allow for 65,536 available hosts.

### CIDR
Network classes became mostly obsolete as IPv4 depletion rapidly increased with the dotcom bubble of the late-90s, and it was eventually subsumed and replaced by **Classless Inter-Domain Routing (CIDR)**.

IP addresses are 

#### 3. `localhost`
The `127.x.x.x` category of addresses is reserved for use by internal systems within a host node - when referring to a `localhost`, we mean *this computer*.

It is used to access network services that are running on the host computer via the **local loopback mechanism**, a network interface which bypasses the networking hardware of that computer entirely.

For instance, if we run a program which can be hosted on the Internet directly from a computer, such as a Java application which connects to the Internet, we will find that this program is hosted at `localhost:8080`.

In this instance, `8080` refers to the specific **port** which a program is using to run. We discuss ports in [this module]().

Because the local loopback mechanism can be used to run a network service on a host without requiring any physical network interface, it can even run on a machine which is not connected to any networks at all.

The name `localhost` normally resolves to the IPv4 loopback address `127.0.0.1`, and to the IPv6 loopback address `::1`.

### 4. Internal & External IPs
### 5. Routing

## Viewing your IP address
### Windows
### Linux

## Changing IP address
**IP-based geolocation** is the mapping of an issued IP address to the location in which the server that a particular node is connected to. 

While this is sometimes useful - such as for querying the location of a central web server which a particular node is connected to, or for **Global Positioning System (GPS)** location on mobile devices - there are cases in which changing a node's perceived location can be beneficial.

### VPNs
A **Virtual Private Network (VPN)** is used to extend a private network across a public network, enabling nodes to send and receive data on a public network as though they were directly connected to a private network.

Originally, VPNs were developed as a way to allow remote users, such as those working for large organisations, access to corporate applications and resources - eliminating the need to use a machine that is directly connected to an organisational network.

In terms of security, the connection to a private network is established through an encrypted layered **tunneling protocol**; the VPN user then authenticates themselves so as to gain access to the network.

In recent times, most individuals with a VPN use them for a number of reasons:
 - **Privacy** - it is commonly believed that **Internet Service Providers (ISPs)** are able to view the history of every website that was visited by each of its users; using a VPN arguably obfuscates this information.
 - **Security** - as a VPN extends a private network across a public system through an encrypted tunnel, they act as an extra measure when accessing sensitive information; some VPNs may themselves be encrypted.
 - **Anonymity** - a VPN can be used as a **proxy server**, which acts as an intermediary layer between a user's client and the servers they access. 
 - **Geo-spoofing/anti-censorship** - VPN companies usually host their servers in locations which have more lax Internet restrictions, such as for video- and music-streaming, than the countries they market their service to; these users connect to the VPN in order to spoof their location so as to appear as though they are accessing Web services from the location of these servers.

VPNs are less powerful than most of the general public believe, for a number of reasons. [This video explores some of the more common misconceptions surrounding VPNs.](https://www.youtube.com/watch?v=WVDQEoe6ZWY)

## Tasks
### View the ARP table
#### macOS, Linux, or Windows Subsystem for Linux
Open a terminal, and enter the following command:
```text
admin@ubuntu:~$ cat /proc/net/arp
```
You should see something like this:

```text
IP address		Hw type		Flags	HW address			Mask	Device
192.168.1.73	0x1			0x2		10:fe:ed:11:d4:ec	*		enp0s3
192.169.1.254	0x1			0x2		a0:39:ee:4e:5a:49	*		enp0s3
```
What might these represent?

#### Windows
Open a command prompt/Powershell, then enter the following command:
```text
 C:\WINDOWS\system32>arp -a
```
You should see one or more tables like this:
```text
Interface: 192.168.1.154 --- 0x18
  Internet Address      Physical Address      Type
  192.168.1.133         f4-30-b9-dc-fd-41     dynamic
  192.168.1.228         80-e8-2c-b3-d2-32     dynamic
  192.168.1.254         00-0c-29-3c-38-5f     dynamic
  192.168.1.255         ff-ff-ff-ff-ff-ff     static
  224.0.0.22            01-00-5e-00-00-16     static
  224.0.0.251           01-00-5e-00-00-fb     static
  224.0.0.252           01-00-5e-00-00-fc     static
  239.255.255.250       01-00-5e-7f-ff-fa     static
```
What might these represent?

### Find your current IP address
### Change your IP address
#### Installing Opera VPN
