<!--in-progress, may split out/remove some/all vpn and onion stuff-->

# IP address

<!--TOC_START-->
### Contents
- [Overview](#overview)
	- [Internet Protocol](#internet-protocol)
	- [Usage](#usage)
	- [Relevance](#relevance)
- [Viewing your IP address](#viewing-your-ip-address)
	- [Windows](#windows)
	- [Linux](#linux)
- [Format](#format)
	- [IP Standards](#ip-standards)
	- [IPv4](#ipv4)
	- [IPv6](#ipv6)
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
IP requires a number of different things to work:
1. An IP address, assigned to whichever device is attempting to transfer information over the Internet
2. A mechanism for mapping hardware-specific MAC addresses to networked IP addresses
3. A defined local network
4. A gateway from a the local network to external networks
5. A mechanism for moving data around both local and external networks

This section will discuss each of these characteristics.

### IP addresses
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
2001:0db8:0000:0000:8a2e:0370:7334:0000

2001:db8:0:0:82ae:370:7734:0

2001:db8::8a2e:370:7334::
```

IPv4 addresses can also be expressed in IPv6 format - the IPv4 address `10.12.56.23` becomes:
```text
0:0:0:0:0:ffff:a0c:3817

::ffff:a0c:3817
```

As 17% of IPv4 addresses are currently in-use (as of 2011), it is widely considered that IPv4 will be used alongside IPv6 for the forseeable future.

#### `localhost`
The `127.x.x.x` category of addresses is reserved for use by internal systems within a host node - when referring to a `localhost`, we mean *this computer*.

It is used to access network services that are running on the host computer via the **local loopback mechanism**, a network interface which bypasses the networking hardware of that computer entirely.

For instance, if we run a program which can be hosted on the Internet directly from a computer, such as a Java application which connects to the Internet, we will find that this program is hosted at `localhost:8080`.

In this instance, `8080` refers to the specific **port** which a program is using to run. We discuss ports in [this module]().

Because the local loopback mechanism can be used to run a network service on a host without requiring any physical network interface, it can even run on a machine which is not connected to any networks at all.

The name `localhost` normally resolves to the IPv4 loopback address `127.0.0.1`, and to the IPv6 loopback address `::1`.

### ARP
Every networked device contains at least one physical **Network Interface Card (NIC)**, such for Ethernet or wireless connections, providing a physical connection between that device and a network.

NICs can generate a **broadcast domain** between devices if they are all connected to the same system - such as a local area network inside an office building:



Each NIC has a unique [MAC address](https://github.com/bob-crutchley/courseware/tree/master/topics/networking/modules/mac-address/) burned into it by its manufacturer, which are used by Ethernet and wireless connections to communicate between network devices.

As these NICs only understand MAC addresses, the **Address Resolution Protocol (ARP)** is used to resolve MAC to IP, or vice-versa.

ARP uses broadcast packets to all computers on the Ethernet segment to discover MAC addresses.


### Subnets
### Internal & External IPs
### Routing

## Viewing your IP address
### Windows
### Linux

## Changing IP address
### VPNs
#### ISP tracking
#### Jurisdiction tracking
#### Enemies of the Internet
### Onion routing
#### Tor
### Why?

## Tasks
### Find your current IP address
### Change your IP address
#### Installing Opera VPN
