---
title: "IPv6 Over Bluetooth"
category: info

docname: draft-elkins-v6ops-ipv6blt-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Operations and Management"
workgroup: "IPv6 Operations"
keyword:
 - next generation
 - unicorn
 - sparkling distributed ledger
venue:
  group: "IPv6 Operations"
  type: "Working Group"
  mail: "v6ops@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/v6ops/"
  github: "mrakshith21/draft-ipv6-over-bluetooth"
  latest: "https://mrakshith21.github.io/draft-ipv6-over-bluetooth/draft-elkins-v6ops-ipv6blt.html"

author:
 -
    fullname: "Nalini Elkins"
    organization: Inside Products, Inc.
    email: "nalini.elkins@insidethestack.com"
 -
    fullname: "Mohit P. Tahiliani"
    organization: NITK Surathkal
    email: "tahiliani@nitk.edu.in"
 -
    fullname: "Rakshith Mohan"
    organization: NITK Surathkal
    email: "rakshith21mohan@gmail.com"
 -
    fullname: "Tejas Sankpal"
    organization: NITK Surathkal
    email: "tejas@gmail.com"
 -
    fullname: "Shrinidhi Ballal Nidamboor"
    organization: NITK Surathkal
    email: "ballalshrinidhi@gmail.com"


normative:

informative:

--- abstract

RFC 7668 and RFC 9159 describe IPv6 over Bluetooth Low Energy networks. However, the potential of this technology has not been fully explored. This document describes IPv6 over Bluetooth on Windows along with an implementation. It also discusses some use cases of this technology beyond BLE networks.

--- middle

# Introduction

Bluetooth is a low-power wireless technology designed for short-range control and monitoring applications. The IETF has produced specifications regarding IPv6 over Bluetooth Low Energy devices to improve the Internet of Things arena, such as the Internet Protocol Support Profile (IPSP). RFC 7668 {{?RFC7668}} was developed for star topology while RFC 9159 {{?RFC9159}} specifies mechanisms to enable IPv6 mesh over Bluetooth LE links. However, the potential of this technology for non-low powered devices has been unexplored. This document introduces IPv6 over Bluetooth with uncompressed (optional) frames as a potential medium for non low powered devices (Windows, Linux, Android OS). It  includes an implementation of IPv6 over Bluetooth on Windows as an example and discusses use cases of this technology.

## Background

### GATT

GATT (Generic ATTribute Profile) defines the way that two Bluetooth Low Energy devices messages transfer between each other. It uses a generic data protocol called the Attribute Protocol (ATT), which is used to store services, and characteristics in a simple lookup table with 16-bit IDs (UUIDs). GATT operates when a dedicated connection is established between two devices.

GATT transactions in BLE are based on high-level, nested objects called Profiles, Services, Characteristics and Control Points. Services are used to break data up into logical entities, and contain specific chunks of data called characteristics. A service can have one or more characteristics, and each service distinguishes itself from other services by means of a unique numeric ID called a UUID, which can be either 16-bit (for officially adopted BLE Services) or 128-bit (for custom services). The lowest level concept in GATT transactions is the Characteristic, which encapsulates a single data point. Each characteristic distinguishes itself via a pre-defined 16-bit or 128-bit UUID.

### IPv6 over Bluetooth networks and the IPSP

As per RFC 9159 {{?RFC9159}}, for IPv6 over BLE mesh networks, a multilink model is chosen. The network consists of nodes 6LN (node), 6LR (router), and 6LBR (border router). A 6LN is a peripheral node which connects to the network through a 6LR or a 6LBR. A 6LR connects 6LN’s to other 6LR’s or 6LBR. One or more 6LBRs are connected to the Internet. A router can manage connections with a number of peripheral devices, while a peripheral is usually connected only to a central node. A single global unicast prefix is used on the whole subnet.
The IPSP enables discovery of IP-enabled devices and the establishment of a link-layer connection for transporting IPv6 packets. The IPSP defines the node and router roles for devices that consume/originate IPv6 packets and for devices that can route IPv6 packets, respectively.

### Address Configuration
RFC 7668 specifies a stateless address autoconfiguration scheme for all nodes in the network. An IPv6 link-local address is assigned to the Bluetooth interface based on the 48-bit Bluetooth device address. A 64-bit address is generated from the Bluetooth address, which is prepended with fe80::/64. Mechanisms for registering a non link local address, and multiple addresses are also provided.

### Header Compression
Since BLE communication aims to conserve power, IPv6 over BLE involves header compression to reduce the size of IPv6 packets. This compression is based on the 6LoWPAN IPv6 header compression as specified in RFC 6282. However, since the document focuses mainly on IPv6 over Bluetooth for non low powered devices, header compression is optional.

# Implementation of IPv6 over Bluetooth on Windows

In this section, we describe an implementation of IPv6 over Bluetooth on Windows.

## Architecture
There are two primary components to this project:

- WFP callout driver
- Packet processing app

Additionally, there are four DLL libraries in this project to support functionality in the packet processing app:

- A custom driver interoperability library.
- A Bluetooth GATT library, also including device enumeration and discovery.
- A 6LoWPAN library.

The following diagram illustrates the system architecture for this solution:

<!-- ![image](https://github.com/mrakshith21/draft-ipv6-over-bluetooth/assets/78913321/d250a65d-272f-4127-a1e5-f9bf4523b23e) -->

<!-- Figure 2: IPv6 over Bluetooth Architecture -->

The componets described in the diagram are briefly described below:

~~~
                    +--------------+
                    |    6LoWPAN   |
			                 | library (DLL)|
                    +--------------+
				                       |
                           |
                           |
+----------------+	  +--------------+	 +----------------+
 | Driver interop |	 |    Packet    |	 | Bluetooth GATT |
 | library (DLL)  |--|  processing  |--| library (DLL)  |
 +----------------+	 |    UWP app   |	 +----------------+
          |          +--------------+		         |
          +------------------+			               |
User mode			                 |			               |
-----------------------------|------------------|----------
Kernel mode			               |			               |
				                         |			               |
 +--------------+		          |		         +--------------+
 |      UDP     |		          |		         | Bluetooth LE |
 +--------------+	  +---------------+    +--------------+
 |     IPv6     |	  |  WFP callout  |	   |   Bluetooth  |
 +--------------+	  |    driver     |	   |     L2CAP    |
 |Wi-Fi/Ethernet|   |(IPv6ToBle.sys)|	   +--------------+
 +--------------+   +---------------+
~~~
{: #fig-arch title="Architecture"}

- WFP Callout Driver (IPv6ToBle.sys): This driver acts as a bridge between the TCP/IP stack and the Bluetooth stack, filtering traffic destined for BLE devices and managing outbound traffic.

- Packet Processing App: This user-mode component completes the bridge between TCP/IP and Bluetooth LE stacks. It handles tasks like obtaining IPv6 addresses, scanning for compatible BLE devices, compressing IPv6 headers, and transferring packets over BLE.

- Driver Interop Library: Used by the packet processing app to communicate with the driver, supporting both synchronous and asynchronous commands.

- Bluetooth GATT Library: Encompasses GATT server and client functionality, advertising services and facilitating packet transfer between devices.

- 6LoWPAN Library: Implements header compression and stateless auto-configuration, for BLE communication.

## Components
The following are the components involved in the implementation of the IPv6 over Bluetooth in Windows operating system.

### Driver
The driver acts as a bridge between the Windows TCP/IP network stack and the Bluetooth stack. It filters outgoing IPv6 packets for the nodes running in the BLE network. These packets are then processed by the packet processing app. The driver can also filter incoming IPv6 packets and inject them into the TCP/IP stack.

### Packet Processing App
This is a core component that is a part of the bridge along with the driver. Firstly, it obtains a link-local address based on the Bluetooth address. It also scans for nearby devices and identifies compatible ones. Then, it subscribes and receives IPv6 packets from the driver. After compressing the header, it transfers the packet over to nearby compatible devices. The packet processing app on the other device receives the packet. If it is for that device, it sends it to the driver for inbound injection into the TCP/IP network stack, otherwise, sends the packet to each nearby device.

### GATT server
The GATT server is started by the packet processing app. It contains the Internet Protocol Support Service (ISPS), which indicates that the server supports IPv6 over Bluetooth. The server runs on all nodes, and nodes can act as both client and server depending on whether they receive or send the packet.

### Bluetooth packet layout

An IPv6 over BLE packet consists of an IPv6 packet embedded in a Bluetooth packet. It consists of the following:

<!-- ![image](./packet.png) -->
<!--
![image](https://github.com/mrakshith21/draft-ipv6-over-bluetooth/assets/78913321/60eda1f9-8be4-4a12-b6e5-98e6bf440a6e) -->

<!-- Figure 3: Packet Layout -->

~~~
+--------------------------+
|                          |
|       Packet Layout      |
|                          |
+--------------------------+

~~~
{: #fig-packet title="Packet layout"}

A custom lua dissector to dissect the IPv6 packet within the Bluetooth ATT Value field was designed. In the application, the IPv6 packet is written to a GATT characteristic with handle 0x0018. So, the custom dissector is invoked when the ATT handle’s value is 0x0018. The buffer is then passed to the in builtipv6 dissector which dissects the data in the ‘Value’ field.


### Header compression
Header compression is implemented as a library (named  6LoWPAN library), not as an operating system layer or module. The compression/decompression code was based on Contiki OS,  an open source operating system in which 6LoWPAn is implemented as an adaptation layer in the network stack. This is not possible on Windows because it is closed source. Therefore, the concept of an adaptation layer is spread across the driver and this module
Thus, the header compression code lies in the user space, as a library.

- While the packet processing app receives a packet from the driver, the 6LoWPAN library is called to compress the header, before sending it over Bluetooth.
- Similarly, when a packet is received over Bluetooth, the 6LoWPAN library is called to decompress the header.

# Comparison with RFC 7668

RFC 7668 {{?RFC7668}} specifies RFC 6282 {{?RFC6282}} for 6LoWPAN

- Header compression
- Fragmentation and reassembly
- Stateless auto configuration

The LoWPAN library of the project fulfills the first and third goals, the second goal is automatically performed at the Bluetooth L2CAP layer by Windows Bluetooth.
In particular, IPv6 header compression is implemented as per RFC 6282 {{?RFC6282}}, and IPv6 address compression is implemented as per RFC 7668 {{?RFC7668}}.

# Use cases

IPv6 over Bluetooth can be used as an alternate technology for connecting devices, while using the vast features of IP as well as a low powered interface using Bluetooth.

## Disaster recovery

In case of scenarios such as natural disasters, Internet failures or shutdown, the area loses its ability to connect to the world through IP. This may be the result due to cable cuts or mobile tower failures. However, a local network can indeed be formed using IPv6 over Bluetooth.

We make the assumption that devices are Bluetooth enabled and are in Bluetooth range, so as to form a network of Bluetooth devices. As described in RFC 9159 {{?RFC9159}} IPv6 over Bluetooth for Mesh Networks, devices can then take up the roles similar to a 6LN, or 6LR. In other words, some devices act as nodes, and some as routers as well. Note that routers forward packets only within the network. In the event that the Internet is accessible for some of the devices, they can act as 6LBR or border routers. With the IPv6 over Bluetooth layer available on the devices, applications can continue to use IP while the actual communication uses Bluetooth. This idea can be backed by the fact that Bluetooth is available not only on PCs but mobile phones as well, which leads to the formation of a network with more links.

There are some things to consider:

- Devices must enforce IPv6 over IPv4 since the technology is only designed to transmit IPv6 packets over Bluetooth.
- Selection of devices as routers, or border routers is a key challenge. The border router must ensure that devices in its domain have unique addresses. The network also needs to have an IPv6 prefix, which is to be determined.

## Near field chat

## Connecting IoT Gateways

# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Security Considerations

The security considerations specified in RFC 7668 {{?RFC7668}} apply for IPv6 over Bluetooth in non lower powered devices as well. RFC 7416 {{?RFC7416}} provides an overview of such threats. Particularly, the routing protocol poses opportunities for threats and attacks. Considering that devices are not resource constrained, the scale of threats can only increase.


# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments
{:numbered="false"}

The authors would like to acknowledge the developer of [IPv6OverBluetoothLowEnergyMesh](https://github.com/uwbiot/IPv6OverBluetoothLowEnergyMesh), for his contribution of a Windows implementation for IPv6 over Bluetooth.
