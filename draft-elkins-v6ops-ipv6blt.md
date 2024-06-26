---
title: "IPv6 Over Bluetooth for Power Affluent Devices"
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
 - power affluent
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

IPv6 over Bluetooth enables devices to communicate using the IPv6 protocol stack over Bluetooth wireless connections, expanding the range of networking options available to devices with Bluetooth capability. RFC 7668 and RFC 9159  describe IPv6 over Bluetooth for Bluetooth Low Energy devices. However, the potential of this technology has yet to be fully explored. This document proposes using this technology for power affluent devices with optional header compression and discusses some use cases.

--- middle

# Introduction

Bluetooth is a low-power wireless technology designed for short-range control and monitoring applications.  The IETF has developed specifications for IPv6 over BLE devices to enhance IoT capabilities, outlined in RFC 7668 {{?RFC7668}}. This RFC primarily addresses star topology, while RFC 9159 {{?RFC9159}} extends IPv6 mesh networking capabilities over BLE links. Despite its focus on low-power devices, IPv6 over BLE can be extended to power affluent platforms like Windows, Linux, Android OS, and iPhone running on personal computers and mobile phones.

By adhering to the Internet Protocol Support Profile (IPSP) published by the Bluetooth SIG, devices such as personal computers and smartphones can harness IPv6 over Bluetooth, presenting an alternative connectivity solution. This integration amalgamates the benefits of the robust addressing of IPv6 and routing capabilities with Bluetooth.

Adding the IPv6 over Bluetooth functionality depends on the design and flexibility of the operating system. However, the adaptation layer need not be built from scratch since the Bluetooth L2CAP Protocol supports fragmentation and reassembly. This document discusses a few use cases of this technology in disaster recovery, near-field chat, and IoT gateways. It also presents an implementation of IPv6 over Bluetooth on Windows.

# Terminology

Throughout this document, these terms have specific meanings:

- The term Bluetooth refers to Bluetooth Low Energy unless mentioned. However, a few devices (older versions) do not support Bluetooth Low Energy, which would mean Bluetooth Classic.
- The term power affluent describes devices that are not resource-constrained. A few examples are smartphones, laptops, personal computers, etc.
- The term node refers to a power-affluent device part of the IPv6 over Bluetooth network.

# Use cases

IPv6 over Bluetooth is an alternate technology for connecting devices while using the vast features of IP as well as the low-powered interface of Bluetooth. By developing implementations of the IPv6 over Bluetooth stack on different operating systems, we can facilitate the use of this technology for power-affluent devices and explore its potential. This section discusses some use cases.


## Disaster recovery

In a disaster area where traditional means of communication like the Internet and cables are damaged or unavailable, establishing connectivity between devices can be critical for coordination, rescue efforts, and communication with survivors. This also applies to cases where the Internet is shut down. In such a situation, IPv6 over Bluetooth can form a network of connected devices.

We assume that devices are Bluetooth enabled and in Bluetooth range to form a network of Bluetooth devices. As described in RFC 9159 {{?RFC9159}} IPv6 over Bluetooth for Mesh Networks, devices can then take up roles similar to a 6LN or 6LR. In other words, some devices act as nodes, and some as routers. Note that routers forward packets only within the network. If the Internet is accessible for some devices, they can act as 6LBR or border routers. With the IPv6 over Bluetooth layer available on the devices, applications can continue to use IP while the actual communication uses Bluetooth. This idea can be backed by the fact that Bluetooth is available not only on PCs but also on mobile phones, which leads to a network with more links.

There are some things to consider.
## Chat application

Bluetooth chat gained prominence through Bridgefy during the Hong Kong protests when the Internet was shut down. IPv6 over Bluetooth offers an enhanced alternative, allowing any network application, including chat apps, to operate similarly as on IPv6. Chat applications can be used to establish local networks via IPv6 over Bluetooth in remote areas or emergencies where Internet access is unavailable.

Devices need to advertise and register themselves in the network to be discovered. Sending a message to a nearby device would take a single hop. However, if a message is to be sent to a farther device, it would include multiple hops across routers. Users can participate in public and private chat. This can be particularly useful in relaying emergency information or in crowded places with a poor Internet connection to communicate within a group.

## IoT Gateways

Use cases in building management, healthcare, workplaces, manufacturing, logistics, and hospitality have introduced low-power IoT devices into their environments. These devices typically do not support IP-based interfaces. Hence, gateway functions need to allow these devices to communicate with the applications that manage them. IoT gateways provide various functionalities depending on the kind of devices they are connected to and their functionality. For example, authentication and authorization of application clients that will access the device, interfaces that allow for bi-directional communication to non-IP devices, and one or more channels to process requests, responses, and asymmetric communications with the non-IP radio resources (access points) in the system, and so on. IoT networks use customized gateways that communicate with each other and external networks to fulfill the required functionality.

IPv6 over Bluetooth can be used in IoT networks wherein multiple IoT gateways communicate with each other and a central router using IPv6 over Bluetooth Low Energy (BLE). The topology consists of IoT devices connected to local gateways, which relay data to the central router for external connectivity.

~~~
               IoT Gateway 1
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

As per RFC 9159 {{?RFC9159}}, for IPv6 over BLE mesh networks, a multilink model is chosen. The network consists of nodes 6LN (node), 6LR (router), and 6LBR (border router). A 6LN is a peripheral node that connects to the network through a 6LR or a 6LBR. A 6LR connects 6LNs to other 6LRs or 6LBR. One or more 6LBRs are connected to the Internet. A router can manage connections with several peripheral devices, while a peripheral is usually connected only to a central node. A single global unicast prefix is used on the whole subnet.
The IPSP enables the discovery of IP-enabled devices and the establishment of a link-layer connection for transporting IPv6 packets. The IPSP defines the node and router roles for devices that consume/originate IPv6 packets and for devices that can route IPv6 packets.

## Address Configuration
RFC 7668 {{?RFC7668}} specifies a stateless address autoconfiguration scheme for all nodes in the network. An IPv6 link-local address is assigned to the Bluetooth interface based on the 48-bit Bluetooth device address. A 64-bit address is generated from the Bluetooth address, which is prepended with fe80::/64. Mechanisms for registering a non-link local address and multiple addresses are also provided.

## Header Compression
Since BLE communication aims to conserve power, IPv6 over BLE involves header compression to reduce the size of IPv6 packets. This compression is based on the 6LoWPAN IPv6 header compression specified in RFC 6282 {{?RFC6282}}. However, since the document focuses mainly on IPv6 over Bluetooth for power-affluent devices, header compression is optional, though it is provided in the implementation.

## Shortwave radio

Amateur or shortwave radio is a viable alternative for long-range communication, particularly in remote or disaster-affected areas. These radios utilize radio waves that can travel vast distances by bouncing off the ionosphere. The distance covered by these signals can vary based on the frequency used. Various frequencies are designated for emergency communication, including those used by medical relief operations, state police, fire departments, and maritime weather alerts. However, these frequencies can differ from country to country.

Setting up a personal computer as a shortwave radio transmitter or receiver is feasible. A possible solution involves the draft "A Method for Deriving Stable IPv6 Interface Identifiers from Amateur Radio Callsigns." by E. Pratten, which outlines a method to generate IPv6 addresses for amateur radio nodes or stations. By configuring devices with these designated IPv6 addresses to communicate over shortwave radio and leveraging local connectivity via IPv6 over Bluetooth, it is possible to extend Internet connectivity to remote areas.

# Implementation Considerations

The following describes a list of points to consider while developing an IPv6 over Bluetooth implementation:

- Header compression can be optional and determined by the application using IPv6 over Bluetooth. Correspondingly, the application can make use of the 6LowPAN library if needed. If an Internet-based application uses IPv6 over Bluetooth for communication, header compression may be avoided.
- Traditionally, Bluetooth MAC addresses has been used to generate IPv6 addresses. However, this method has not been promoted recently. Thus, any relevant method can be used. The address generation is beyond the scope of this draft.
- Devices must enforce IPv6 over IPv4 since the technology only transmits IPv6 packets over Bluetooth.
- Selection of devices as routers or border routers is a crucial challenge. The border router must ensure that devices in its domain have unique addresses. The network must also have an IPv6 prefix, which is to be determined.
- Packet delivery cannot be guaranteed since devices may sometimes go out of Bluetooth range.

# Common Implementation Details of IPv6 over Bluetooth

In this section, we describe common implementation details of IPv6 over Bluetooth.

**Note**: Exact implementation details may vary across different operating systems. This section only describes a general architecture and its components.

## Architecture
The following components are necessary for an implementation:

- A driver or kernel-level module
- A packet processing application

Additionally, the following libraries will be needed to support the functionality of the packet-processing app:

- A driver interoperability library
- A Bluetooth GATT library
- A 6LoWPAN library

{{fig-arch}} illustrates the system architecture for this solution.

~~~
                    +--------------+
                    |    6LoWPAN   |
                    |    library   |
                    +--------------+
                           |
                           |
                           |
+----------------+  +---------------+   +----------------+
| Driver interop |  |    Packet     |   | Bluetooth GATT |
| library        |--|  processing   |---| library        |
+----------------+  |  application  |   +----------------+
          |         +---------------+	            |
          +------------------+		            |
User mode	             |		            |
-----------------------------|----------------------|-------------
Kernel mode		     |		            |
                             |		            |
 +--------------+	     |		         +--------------+
 |      UDP     |	     |		         | Bluetooth LE |
 +--------------+     +-----------------+        +--------------+
 |     IPv6     |     | Driver or kernel|	 |   Bluetooth  |
 +--------------+     |    module       |	 |     L2CAP    |
 |Wi-Fi/Ethernet|     +-----------------+	 +--------------+
 +--------------+

~~~
{: #fig-arch title="Architecture"}

The architecture consists of components operating in the user and kernel modes.

## Components
The following describes the functionalities of the components specified in the architecture in detail.

### Driver
The driver is the component that operates in the kernel mode. The following operations are to be supported by the driver
- Filters outgoing IPv6 packets.
- Push a given IPv6 packet embedded in a Bluetooth packet into the network stack for an appropriate application to consume.

### Packet Processing Application
The packet processing application, working with the driver, is an essential component that bridges the transition of the packet from the IP layer to Bluetooth and vice versa. The following operations are to be supported by the packet processing application:

- Obtain IPv6 address to be used in the IPv6 over Bluetooth network
- Scan for nearby Bluetooth devices
- Communicate with the driver to receive intercepted IPv6 packets and send them over Bluetooth. Send IPv6 packets received over Bluetooth to the driver for inbound injection. If an inbound packet is not for the device, it may be forwarded it to the router application (if running) for further transmission.
- Use libraries, such as a 6LowPAN library for address and header compression and a driver interface library to communicate with the driver.

### Libraries

- A driver interoperability library: This is an interface used by the packet processing application to communicate with the driver.
- A Bluetooth GATT library: The packet processing application initiates a GATT application, which acts as a server or client. It supports the Internet Protocol Support Service (IPSP). A node communicates with another node by writing an IPv6 packet to a characteristic of the IPSP in the GATT application.
- A 6LoWPAN library: This is used for header compression and decompression.

### Bluetooth packet layout

An IPv6 over BLE packet consists of an IPv6 packet embedded in a Bluetooth packet. A sample packet captured from a Windows implementation is shown in {{fig-packet}}. Only the data within the Bluetooth Attribute Protocol is shown in detail for this discussion.

~~~
+--------------------------------------------+
| Bluetooth HCI                              |
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

# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Security Considerations

The security considerations specified in RFC 7668 {{?RFC7668}} also applies for IPv6 over Bluetooth in power-affluent devices. RFC 7416 {{?RFC7416}} provides an overview of such threats. Notably, the routing protocol poses opportunities for threats and attacks. Considering that devices are not resource constrained, the scale of threats can only increase.


# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments
{:numbered="false"}

The authors would like to acknowledge the developer of [IPv6OverBluetoothLowEnergyMesh](https://github.com/uwbiot/IPv6OverBluetoothLowEnergyMesh), Duncan MacMichael, for his contribution of a Windows implementation for IPv6 over Bluetooth.

