---
title: "IPv6 Over Bluetooth for Non Low Powered Devices"
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
 - ipv6
 - bluetooth
 - non low powered devices
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
    fullname: "Shrinidhi Ballal Nidamboor"
    organization: NITK Surathkal
    email: "ballalshrinidhi@gmail.com"
 -
    fullname: "Tejas Sankpal"
    organization: NITK Surathkal
    email: "tejas@gmail.com"


normative:

informative:

--- abstract

IPv6 over Bluetooth enables devices to communicate using the IPv6 protocol stack over Bluetooth wireless connections, expanding the range of networking options available to devices with Bluetooth capability. RFC 7668 and RFC 9159  describe IPv6 over Bluetooth for Bluetooth Low Energy devices. However, the potential of this technology has not been fully explored. This document proposes using this technology for non-low-powered devices with optional header compression and discusses some use cases.

--- middle

# Introduction

Bluetooth is a low-power wireless technology designed for short-range control and monitoring applications.  The IETF has developed specifications for IPv6 over BLE devices to enhance IoT capabilities, outlined in RFC 7668 {{?RFC7668}}. This RFC primarily addresses star topology, while RFC 9159 {{?RFC9159}} extends IPv6 mesh networking capabilities over BLE links. Despite its focus on low-power devices, IPv6 over BLE can be extended to non-low-powered platforms like Windows, Linux, Android OS, and iPhone running on personal computers and mobile phones.

By adhering to the Internet Protocol Support Profile (IPSP), published by the Bluetooth SIG, devices such as personal computers and smartphones can harness IPv6 over Bluetooth, presenting an alternative connectivity solution. This integration amalgamates the benefits of the robust addressing of IPv6 and routing capabilities with Bluetooth.

Adding the IPv6 over Bluetooth functionality depends on the design and flexibility of the operating system. However, the adaptation layer need not be built from scratch since the Bluetooth L2CAP Protocol supports fragmentation and reassembly. Furthermore, in the case of unconstrained devices, header compression is not necessary (but can be kept optional if the Bluetooth MTU size is small or if low-powered devices are also involved). This document discusses a few use cases of this technology in disaster recovery, near-field chat, and IoT gateways. It also presents an implementation of IPv6 over Bluetooth on Windows.

# Use cases

IPv6 over Bluetooth can be used as an alternate technology for connecting devices while using the vast features of IP as well as the low-powered interface of Bluetooth. By developing implementations of the IPv6 over Bluetooth stack on different operating systems, we can facilitate the use of this technology for non-low-powered devices and explore its potential. This section discusses some use cases.

## Disaster recovery

In a disaster area where traditional means of communication like the Internet and cables are damaged or unavailable, establishing connectivity between devices can be critical for coordination, rescue efforts, and communication with survivors. This also applies to cases where the Internet is shut down. In such a situation, IPv6 over Bluetooth can potentially form a network of connected devices.

We assume that devices are Bluetooth enabled and are in Bluetooth range, to form a network of Bluetooth devices. As described in RFC 9159 {{?RFC9159}} IPv6 over Bluetooth for Mesh Networks, devices can then take up the roles similar to a 6LN, or 6LR. In other words, some devices act as nodes, and some as routers as well. Note that routers forward packets only within the network. If the Internet is accessible for some of the devices, they can act as 6LBR or border routers. With the IPv6 over Bluetooth layer available on the devices, applications can continue to use IP while the actual communication uses Bluetooth. This idea can be backed by the fact that Bluetooth is available not only on PCs but mobile phones as well, which leads to the formation of a network with more links.

There are some things to consider.

- Devices must enforce IPv6 over IPv4 since the technology is only designed to transmit IPv6 packets over Bluetooth.
- Selection of devices as routers, or border routers is a key challenge. The border router must ensure that devices in its domain have unique addresses. The network also needs to have an IPv6 prefix, which is to be determined.

## Chat application

Bluetooth chat gained prominence through Bridgefy during the Hong Kong protests when the Internet was shut down. IPv6 over Bluetooth offers an enhanced alternative, allowing any network application, including chat apps, to operate similarly as on IPv6. In remote areas or emergencies where Internet access is unavailable, local networks can be established using IPv6 over Bluetooth.

Devices need to advertise and register themselves in the network to be discovered. Sending a message to a nearby device would take a single hop. However, if a message is to be sent to a farther device, it would include multiple hops across routers. Users can participate in public and private chat. If some devices can connect to the Internet, they can act as border routers, enabling the other devices to connect to the Internet. This can be particularly useful in relaying emergency information or in crowded places with a poor Internet connection to communicate within a group.

One of the advantages of this mechanism over plain Bluetooth is that devices acting as routers can forward packets using established routing protocols. Otherwise, the application would need to manage this, resulting in an increased burden on the application and higher delays. IPv6 over Bluetooth thus provides a more efficient and scalable solution for local communication during Internet disruptions.

## IoT Gateways

Use cases in building management, healthcare, workplaces, manufacturing, logistics, and hospitality have introduced low-power IoT devices into their environments. These devices typically do not support IP-based interfaces. Hence, gateway functions need to allow these devices to communicate with the applications that manage them. IoT gateways provide various functionalities depending on the kind of devices they are connected to and their functionality. For example, authentication and authorization of application clients that will access the device, interfaces that allow for bi-directional communication to non-IP devices, and one or more channels to process requests, responses, and asymmetric communications with the non-IP radio resources (access points) in the system, and so on. IoT networks use customized gateways that communicate with each other and external networks to fulfill the required functionality.

IPv6 over Bluetooth can be used in IoT networks wherein multiple IoT gateways communicate with each other and a central router using IPv6 over Bluetooth Low Energy (BLE). The topology consists of IoT devices connected to local gateways, which relay data to the central router for external connectivity.

~~~
               Iot Gateway 1
                     |
                     |
IoT Gateway 2 --- Border Router ------------- Internet
                     |
                     |
               IoT Gateway 3
~~~
{: #fig-gateways title="Multiple IoT gateways connected through a border router"}

# Relevant Technology

## GATT

GATT (Generic ATTribute Profile) defines how two Bluetooth Low Energy devices transfer messages between each other. It uses a generic data protocol called the Attribute Protocol (ATT), which is used to store services and characteristics in a simple lookup table with 16-bit IDs (UUIDs). GATT transactions in BLE are based on high-level, nested objects called Profiles, Services, Characteristics, and Control Points. A service can have one or more characteristics, and each service distinguishes itself from other services using a unique numeric ID called a UUID, which can be either 16-bit (for officially adopted BLE Services) or 128-bit (for custom services). The lowest level concept in GATT transactions is the Characteristic, which encapsulates a single data point. Each characteristic distinguishes itself via a pre-defined 16-bit or 128-bit UUID.

## IPv6 over Bluetooth networks and the IPSP

As per RFC 9159 {{?RFC9159}}, for IPv6 over BLE mesh networks, a multilink model is chosen. The network consists of nodes 6LN (node), 6LR (router), and 6LBR (border router). A 6LN is a peripheral node that connects to the network through a 6LR or a 6LBR. A 6LR connects 6LNs to other 6LRs or 6LBR. One or more 6LBRs are connected to the Internet. A router can manage connections with a number of peripheral devices, while a peripheral is usually connected only to a central node. A single global unicast prefix is used on the whole subnet.
The IPSP enables the discovery of IP-enabled devices and the establishment of a link-layer connection for transporting IPv6 packets. The IPSP defines the node and router roles for devices that consume/originate IPv6 packets and for devices that can route IPv6 packets.


## Address Configuration
RFC 7668 {{?RFC7668}} specifies a stateless address autoconfiguration scheme for all nodes in the network. An IPv6 link-local address is assigned to the Bluetooth interface based on the 48-bit Bluetooth device address. A 64-bit address is generated from the Bluetooth address, which is prepended with fe80::/64. Mechanisms for registering a non-link local address and multiple addresses are also provided.

## Header Compression
Since BLE communication aims to conserve power, IPv6 over BLE involves header compression to reduce the size of IPv6 packets. This compression is based on the 6LoWPAN IPv6 header compression specified in RFC 6282 {{?RFC6282}}. However, since the document focuses mainly on IPv6 over Bluetooth for non-low powered devices, header compression is optional, though it is provided in the implementation.

## Shortwave radio

Amateur or shortwave radio is also being used as a viable alternative for long-range communication, particularly in remote or disaster-affected areas. These radios utilize radio waves that can travel vast distances by bouncing off the ionosphere. The distance covered by these signals can vary based on the frequency used. Various frequencies are designated for emergency communication, including those used by medical relief operations, state police, fire departments, and maritime weather alerts. However, these frequencies can differ from country to country.

Setting up a personal computer as a shortwave radio transmitter or receiver is feasible. A possible solution involves the draft "A Method for Deriving Stable IPv6 Interface Identifiers from Amateur Radio Callsigns." by E. Pratten, which outlines a method to generate IPv6 addresses for amateur radio nodes or stations. By configuring devices with these designated IPv6 addresses to communicate over shortwave radio and leveraging local connectivity via IPv6 over Bluetooth, it is possible to extend Internet connectivity to remote areas.

# Implementation of IPv6 over Bluetooth on Windows

In this section, we describe an implementation of IPv6 over Bluetooth on Windows.

## Architecture
There are two primary components to this implementation:

- WFP callout driver
- Packet processing app

Additionally, there are four libraries in this project to support functionality in the Packet Processing App:

- A custom driver interoperability library.
- A Bluetooth GATT library, which also includes device enumeration and discovery.
- A 6LoWPAN library.

{{fig-arch}} illustrates the system architecture for this solution.

~~~
                    +--------------+
                    |    6LoWPAN   |
                    | library (DLL)|
                    +--------------+
                           |
                           |
                           |
+----------------+  +--------------+  +----------------+
| Driver interop |  |    Packet    |  | Bluetooth GATT |
| library        |--|  processing  |--| library        |
+----------------+  |    UWP app   |  +----------------+
          |         +--------------+	      |
          +------------------+		      |
User mode	             |		      |
-----------------------------|----------------|-------------
Kernel mode		     |		      |
                             |		      |
 +--------------+	     |		   +--------------+
 |      UDP     |	     |		   | Bluetooth LE |
 +--------------+     +---------------+    +--------------+
 |     IPv6     |     |  WFP callout  |	   |   Bluetooth  |
 +--------------+     |    driver     |	   |     L2CAP    |
 |Wi-Fi/Ethernet|     +---------------+	   +--------------+
 +--------------+
~~~
{: #fig-arch title="Architecture"}

The architecture consists of components operating in the user and kernel modes.

The kernel mode involves low-level operations such as filtering packets through network interfaces and establishing Bluetooth links. The driver is a core component that filters outgoing IPv6 packets and can also filter incoming ones, forwarding them appropriately. It can also push a given IPv6 packet embedded in a Bluetooth packet into the network stack for an appropriate application to consume.

Components operating in the user mode are as follows. The Packet Processing App, working with the driver, is an important component that bridges the transition of the packet from the IP layer to Bluetooth and vice versa. It obtains Bluetooth-based link-local addresses, scans for nearby Bluetooth devices, compresses IPv6 packet headers, and initiates the transfer of Bluetooth packets or receipts. It uses libraries, such as a 6LowPAN library for address and header compression and a driver interface library to communicate with the driver. The GATT server, initiated by the Packet Processing App, provides IPSP, supports IPv6 over Bluetooth, and operates on all nodes, allowing them to act as clients and servers as needed.

## Components
The following are the components involved in the implementation of IPv6 over Bluetooth in the Windows operating system.

### Driver
A WFP driver (Windows Filtering Platform) in Windows is a network traffic processing component that provides a programmable interface for filtering and examining network packets at various TCP/IP protocol stack layers.
The driver acts as a bridge between the Windows TCP/IP network stack and the Bluetooth stack. It filters outgoing IPv6 packets for the nodes running in the BLE network. These packets are then processed by the packet processing app. The driver can filter incoming IPv6 packets and inject them into the TCP/IP stack.

### Packet Processing App
This is a core component that is a part of the bridge and the driver. Firstly, it obtains a link-local address based on the Bluetooth address. It also scans for nearby devices and identifies compatible ones. Then, it subscribes and receives IPv6 packets from the driver. After compressing the header, the packet is transferred to nearby compatible devices. The packet processing app on the other device receives the packet. If it is for that device, it sends it to the driver for inbound injection into the TCP/IP network stack; otherwise, sends the packet to each nearby device.

### GATT server
The GATT server is started by the packet processing app. It contains the Internet Protocol Support Service (IPSP), which indicates that the server supports IPv6 over Bluetooth. A node must write to the characteristics of the IPSP to send a packet.

### DriverTest
This is a sample application that generates UDP packets with hard-coded data. The driver will intercept these packets and send them to the Packet Processing App for further action.

## Bluetooth packet layout

An IPv6 over BLE packet consists of an IPv6 packet embedded in a Bluetooth packet. A sample packet captured from the application is shown in {{fig-packet}}

~~~
+--------------------------------------------+
| Bluetooth L2CAP Protocol                   |
+--------------------------------------------+
| Bluetooth Attribute Protocol               |
+--------------------------------------------+
|  >  Opcode: Write Command (0x52)           |
+--------------------------------------------+
|  >  Handle: 0x0018                         |
+--------------------------------------------+
|  >  Value: IPv6 Packet                     |
+--------------------------------------------+
|          > IPv6 Header                     |
+--------------------------------------------+
|          > UDP Header                      |
+--------------------------------------------+
|          > Data: Hello World               |
+--------------------------------------------+

~~~
{: #fig-packet title="Packet layout"}

A custom lua dissector was designed to dissect the IPv6 packet within the Bluetooth ATT Value field. The IPv6 packet is written to a GATT characteristic with handle 0x0018. So, the custom dissector is invoked when the ATT handle value is 0x0018. The buffer is then passed to the in-built IPv6 dissector, which dissects the data in the 'Value' field.

## Header compression
Header compression is implemented as a library (named  6LoWPAN library), not as an operating system layer or module. The compression/decompression code was based on Contiki OS,  an open-source operating system in which 6LoWPAn is implemented as an adaptation layer in the network stack. This is not possible on Windows because it is a closed source. Therefore, the concept of an adaptation layer is spread across the driver, and this module
Thus, the header compression code lies in the user space as a library.

- While the packet processing app receives a packet from the driver, the 6LoWPAN library is called to compress the header before sending it over Bluetooth.
- Similarly, when a packet is received over Bluetooth, the 6LoWPAN library is called to decompress the header.

## Comparison with RFC 7668

RFC 7668 {{?RFC7668}} specifies RFC 6282 {{?RFC6282}} for 6LoWPAN

- Header compression
- Fragmentation and reassembly
- Stateless auto configuration

The 6LoWPAN library of the project fulfills the first and third goals, the second goal is automatically performed at the Bluetooth L2CAP layer by Windows Bluetooth.
In particular, IPv6 header compression is implemented as per RFC 6282 {{?RFC6282}}, and IPv6 address compression is implemented as per RFC 7668 {{?RFC7668}}.

# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Security Considerations

The security considerations specified in RFC 7668 {{?RFC7668}} also apply for IPv6 over Bluetooth in non-low powered devices. RFC 7416 {{?RFC7416}} provides an overview of such threats. Particularly, the routing protocol poses opportunities for threats and attacks. Considering that devices are not resource constrained, the scale of threats can only increase.


# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments
{:numbered="false"}

The authors would like to acknowledge the developer of [IPv6OverBluetoothLowEnergyMesh](https://github.com/uwbiot/IPv6OverBluetoothLowEnergyMesh), for his contribution of a Windows implementation for IPv6 over Bluetooth.
