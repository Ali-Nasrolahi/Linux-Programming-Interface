# Notes From ***SOCKETS: FUNDAMENTALS OF TCP/IP NETWORKS***

- [Notes From ***SOCKETS: FUNDAMENTALS OF TCP/IP NETWORKS***](#notes-from-sockets-fundamentals-of-tcpip-networks)
  - [Internets](#internets)
  - [Networking Protocols and Layers](#networking-protocols-and-layers)
    - [Encapsulation](#encapsulation)
  - [The Data-Link Layer](#the-data-link-layer)
  - [The Network Layer: IP](#the-network-layer-ip)
    - [IP transmits datagrams](#ip-transmits-datagrams)
    - [IP is connectionless and unreliable](#ip-is-connectionless-and-unreliable)
    - [IP may fragment datagrams](#ip-may-fragment-datagrams)
  - [IP Addresses](#ip-addresses)
    - [IPv4 addresses](#ipv4-addresses)

## Internets

An *internetwork* or, more commonly, **internet** (with a lowercase i), connects different computer networks, allowing hosts on all of the networks to communicate with one another.

The term *subnetwork*, or **subnet**, is used to refer to one of the networks composing an internet.

The term **Internet** (with an uppercase I) is used to refer to the TCP/IP internet that connects millions of computers globally.

A router, a computer whose function is to connect one *subnetwork* to another, transferring data between them.  
As well as understanding the internet protocol being used, a router must also understand the (possibly) different data-link-layer protocols used on each of the subnets that it connects.

A router has multiple network interfaces, one for each of the subnets to which it is connected. The more general term **multihomed host** is used for any host—not necessarily a router—with **multiple** network interfaces.

---

## Networking Protocols and Layers

A **networking protocol** is a set of rules defining how information is to be transmitted across a network.

Networking protocols are generally organized as a series of **layers**.
> with each layer building on the layer below it to add features that are made available to higher layers.

The **TCP/IP protocol** suite is a layered networking protocol.

One of the notions that lends great power and flexibility to protocol layering is **transparency**— each protocol layer shields higher layers from the operation and complexity of lower layers.
> e.g., an application making use of TCP only needs to use the standard sockets API and to know that it is employing a reliable, byte-stream transport service.

### Encapsulation

The key idea of **encapsulation** is that the information (e.g., application data, a TCP segment, or an IP datagram) passed from a higher layer to a lower layer is treated as **opaque** data by the lower layer.

> In other words, the lower layer makes no attempt to interpret information sent from the upper layer, but merely places that information inside whatever type of packet is used in the lower layer and adds its own layer-specific header before passing the packet down to the next lower layer.

---

## The Data-Link Layer

**Data-link layer**, consists of the *device driver* and the *hardware interface* (network card) to the underlying physical medium (e.g., a telephone line, a coaxial cable, or a fiber-optic cable).

To transfer data, the data-link layer encapsulates *datagrams* from the network layer into units called **frames**.

Each frame includes a header containing, for example, the destination address and frame size. The data-link layer transmits the frames across the physical link and handles acknowledgements from the receiver.

This layer may perform *error detection*, *retransmission*, and *flow control*.
>Some data-link layers also split large network packets into multiple frames and reassemble them at the receiver.

One characteristic of the data-link layer that is important for our discussion of IP is the *maximum transmission unit* (**MTU**).

> A data-link layer’s MTU is the upper limit that the layer places on the size of a frame.

---

## The Network Layer: IP

Above the *data-link layer* is the **network layer**, which is concerned with delivering packets (data) from the source host to the destination host.

This layer performs a variety of tasks, including:

- breaking data into fragments small enough for transmission via the data-link layer (if necessary);

- routing data across the internet; and

- providing services to the transport layer.

In the TCP/IP protocol suite, the principal protocol in the network layer is **IP**.

### IP transmits datagrams

IP transmits data in the form of **datagrams** (**packets**).  
Each datagram sent between two hosts travels *independently* across the network, possibly taking a different route.

An IP datagram includes a header, which ranges in size from 20 to 60 bytes,

### IP is connectionless and unreliable

IP is described as a **connectionless** protocol, since it doesn’t provide the notion of a *virtual circuit* connecting two hosts.

IP is also an **unreliable** protocol: it makes a “best effort” to transmit datagrams from the sender to the receiver, but *doesn’t guarantee* that packets will arrive in the order they were transmitted, that they won’t be duplicated, or even that they will arrive at all.

Nor does IP provide error recovery (packets with header errors are silently discarded).

> Reliability must be provided either by using a reliable transport-layer protocol (e.g., TCP) or within the application itself.

### IP may fragment datagrams

When an IP datagram is larger than the MTU, IP fragments (breaks up) the datagram into suitably sized units for transmission across the network. These fragments are then reassembled at the final destination to re-create the original datagram.

IP fragmentation occurs transparently to higher protocol layers, but nevertheless is generally considered **undesirable**.

The problem is that, because IP doesn’t perform retransmission, and a datagram can be reassembled at the destination only if all fragments arrive, the entire datagram is unusable if any fragment is lost or contains transmission errors.

Modern TCP implementations employ algorithms to determine the MTU of a path between hosts, and accordingly break up the data they pass to IP, so that IP is *not asked* to transmit datagrams that exceed this size.

---

## IP Addresses

An IP address consists of two parts: a **network ID**, which specifies the *network* on which a host resides, and a **host ID**, which identifies the *host* within that network.

### IPv4 addresses

An IPv4 address consists of 32 bits.