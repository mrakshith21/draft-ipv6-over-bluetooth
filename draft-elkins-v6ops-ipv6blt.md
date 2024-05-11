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
    email: "rakshith21mohan@gmail.com"


normative:

informative:

--- abstract

RFC 7668 and RFC 9159 describe IPv6 over Bluetooth Low Energy networks. However, the potential of this technology has not been fully explored. This document desribes IPv6 over Bluetooth on Windows along with an implementation. It also discusses some use cases of this technology beyond BLE networks.

--- middle

# Introduction

Bluetooth devices can only connect to the Internet using the smartphone or a central server as a gateway. However, it is also possible for such devices to operate autonomously using IPv6 over Bluetooth. It is an addition of an IPv6 layer to the Bluetooth stack. The reason IPv6 is chosen, not IPv4, is because it provides significantly more individual IP addresses for the anticipated billions of bluetooth devices that will make up the Internet of Things in the future.

## Background

RFC 7668 specifies the standards and guidelines for transmitting IPv6 packets over Bluetooth Low Energy (BLE) networks.

### Network Topology
IPv6 over BLE networks form a topology with nodes of two roles, central and peripheral. A central node can manage connections with a number of peripheral devices, while a peripheral is usually connected only to a central node. A central node is assumed to be less resource-constrained and acts as a border router. In a typical scenario, the BLE network is connected to the Internet through the border router, which can forward packets between the nodes and to and from the Internet.
The peripheral nodes are connected to each other by IPv6 over Bluetooth. The central node is connected to every peripheral node by IPv6 over Bluetooth.

### Address Configuration
Every node in the topology has to be distinguished from each other. This is done by assigning each of the nodes with a unique ipv6 link-local address. All nodes in the network use their MAC address for generating the unique IPv6 link local address.
IPv6 link-local addresses are 128 bits long. The address consists of two sections as shown in the figure-1, the first 64 bits is used indicating that the IPv6 address is a link-local address and is always set to FE80::/64. The second half is called the interface ID. The generation of this section is different for different operating systems.

The device MAC address is used to generate the Interface ID of the IPv6 link-local address using EUI-64 (Extended Unique Identifier). The steps followed are given below:
- The MAC addresses are 48 bits long. This address is split into two equal halves.
- The bits FF FE are added in between the two halves, hence giving the 64 bit address.
- Next the 7th bit from the left is flipped, giving the final 64 bit Interface ID.

### Header Compression
Since BLE communication aims to conserve power, IPv6 over BLE involves header compression to reduce the size of IPv6 packets. This compression is based on the 6LoWPAN IPv6 header compression as specified in RFC 6282.

# Implementation of IPv6 over Bluetooth on Windows

## Architecture
There are two primary components to this project:
- WFP callout driver (IPv6ToBle.sys).
- Packet processing app.
Additionally, there are four DLL libraries in this project to support functionality in the packet processing app:
- A custom driver interoperability library.
- A Bluetooth GATT library, also including device enumeration and discovery.
- A 6LoWPAN library.

The following diagram illustrates the system architecture for this solution:

![image](https://github.com/mrakshith21/draft-ipv6-over-bluetooth/assets/78913321/d250a65d-272f-4127-a1e5-f9bf4523b23e)

The following are the components involved in the implementation of the IPv6 over Bluetooth in Windows operating system.

## Driver
The driver acts as a bridge between the Windows TCP/IP network stack and the Bluetooth stack. It filters outgoing IPv6 packets for the nodes running in the BLE network. These packets are then processed by the packet processing app. The driver can also filter incoming IPv6 packets and inject them into the TCP/IP stack.

## Packet Processing App
This is a core component that is a part of the bridge along with the driver. Firstly, it obtains a link-local address based on the Bluetooth address. It also scans for nearby devices and identifies compatible ones. Then, it subscribes and receives IPv6 packets from the driver. After compressing the header, it transfers the packet over to nearby compatible devices. The packet processing app on the other device receives the packet. If it is for that device, it sends it to the driver for inbound injection into the TCP/IP network stack, otherwise, sends the packet to each nearby device.

## GATT server
The GATT server is started by the packet processing app. It contains the Internet Protocol Support Service (ISPS), which indicates that the server supports IPv6 over Bluetooth. The server runs on all nodes, and nodes can act as both client and server depending on whether they receive or send the packet.

# Bluetooth packet layout

An IPv6 over BLE packet consists of an IPv6 packet embedded in a Bluetooth packet. It consists of the following:

<!-- | Bluetooth L2CAP Protocol |
| Bluetooth ATT Protocol |
|     Opcode |
|     Handle |
|     Value  |
|         IPv6 packet| -->



+------------------------------+<br />
| -- Bluetooth L2CAP Protocol &nbsp; &nbsp; &nbsp;|<br />
| -- Bluetooth ATT Protocol &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;|<br />
| ---- Opcode &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;|<br />
| ---- Handle &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; |<br />
| ---- Value &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;  &nbsp; &nbsp;  &nbsp;|<br />
| -------- IPv6 Packet  &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;|<br />
+------------------------------+<br />



# Where is header compression done?
Header compression is implemented as a library (named  6LoWPAN library), not as an operating system layer or module. The compression/decompression code was based on Contiki OS,  an open source operating system in which 6LoWPAn is implemented as an adaptation layer in the network stack. This is not possible on Windows because it is closed source. Therefore, the concept of an adaptation layer is spread across the driver and this module.

Thus, the header compression code lies in the user space, as a library.

- While the packet processing app receives a packet from the driver, the 6LoWPAN library is called to compress the header, before sending it over Bluetooth.
- Similarly, when a packet is received over Bluetooth, the 6LoWPAN library is called to decompress the header.


# Comparison with RFC 7668

RFC 7668 specifies RFC 6282 for 6LoWPAN
- Header compression
- Fragmentation and reassembly
- Stateless auto configuration

The LoWPAN library of the project fulfills the first and third goals, the second goal is automatically performed at the Bluetooth L2CAP layer by Windows Bluetooth.
In particular, IPv6 header compression is implemented as per RFC 6282, and IPv6 address compression is implemented as per RFC 7668.


# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
