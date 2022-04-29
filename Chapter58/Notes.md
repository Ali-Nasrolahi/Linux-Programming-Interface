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
    - [IPv6 addresses](#ipv6-addresses)
  - [The Transport Layer](#the-transport-layer)
    - [Port Numbers](#port-numbers)
      - [Well-known, registered, and privileged ports](#well-known-registered-and-privileged-ports)
      - [Ephemeral ports](#ephemeral-ports)
    - [User Datagram Protocol (UDP)](#user-datagram-protocol-udp)
      - [Selecting a UDP datagram size to avoid IP fragmentation](#selecting-a-udp-datagram-size-to-avoid-ip-fragmentation)
    - [Transmission Control Protocol (TCP)](#transmission-control-protocol-tcp)
      - [Connection establishment](#connection-establishment)
      - [Packaging of data in segments](#packaging-of-data-in-segments)
      - [Acknowledgements, retransmissions, and timeouts](#acknowledgements-retransmissions-and-timeouts)
      - [Sequencing](#sequencing)
      - [Flow control](#flow-control)
      - [Congestion control: slow-start and congestion-avoidance algorithms](#congestion-control-slow-start-and-congestion-avoidance-algorithms)
  - [Requests for Comments (RFCs)](#requests-for-comments-rfcs)
  - [END](#end)

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

When an organization applies for a range of IPv4 addresses for its hosts, it receives a 32-bit **network address** and a corresponding 32-bit **network mask**.

> In binary form, this mask consists of a sequence of 1s in the leftmost bits, followed by a sequence of 0s to fill out the remainder of the mask.  
> 1s indicate which part of the address contains the assigned network ID, while the 0s indicate which part of the address is available to the organization to assign as unique host IDs on its network.

Two addresses can’t be assigned.

- One of these is the address whose host ID is all **0 bits**, which is used to
identify the *network* itself.
  > like 192.168.1.0 or 172.16.60.0
- The other is the address whose host ID is all **1 bits** which is *subnet broadcast address*.
  > like 192.168.1.255

The special address `127.0.0.1` is normally defined as the **loopback** address.
> A datagram sent to this address never actually reaches the network, but instead automatically loops back to become input to the sending host.

For use in a C program, the integer constant `INADDR_LOOPBACK` is defined for this address.

The constant `INADDR_ANY` is the so-called IPv4 **wildcard address**.

The wildcard IP address is useful for applications that bind Internet domain sockets on **multihomed hosts**.

> If an application on a multihomed host binds a socket to just one of its host’s IP addresses, then that socket can receive only UDP datagrams or TCP connection requests sent to that IP address.

Typically, IPv4 addresses are **subnetted**.

**Subnetting** divides the host ID part of an IPv4 address into two parts:

- a *subnet ID*
- and a *host ID*

### IPv6 addresses

The key difference is that IPv6 addresses consist of **128 bits**, and the first few bits of the address are a *format prefix*, indicating the address type.

IPv6 addresses are typically written as a series of *16-bit* hexadecimal numbers separated by *colons*, as in the following:

> like F000:0:0:0:0:0:A:1

IPv6 addresses often include a sequence of zeros and, as a notational convenience, two colons ( `::` ) can be employed to indicate such a sequence.

> such as F000::A:1

IPv6 also provides equivalents of the IPv4’s loopback address (127 zeros, followed by a one, thus `::1` )  
and wildcard address (all zeros, written as either `0::0` or `::` ).

---

## The Transport Layer

There are two widely used transport-layer protocols in the TCP/IP suite:

- User Datagram Protocol (**UDP**) is the protocol used for datagram sockets.
- Transmission Control Protocol (**TCP**) is the protocol used for stream sockets.

### Port Numbers

The task of the transport protocol is to provide an end-to-end communication service to applications residing on different hosts (or sometimes on the same host).

In order to do this, the transport layer requires a method of differentiating the applications on a host.

In TCP and UDP, this differentiation is provided by a *16-bit* **port number**.

#### Well-known, registered, and privileged ports

Some **well-known** port numbers are **permanently** assigned to specific applications (also known as *services*).
> the ssh daemon uses the well-known port 22, and HTTP uses the well-known port 80.

Well-known ports are assigned numbers in the range *0 to 1023* by a central authority, the Internet Assigned Numbers Authority.

IANA also records **registered** ports, which are allocated to application developers on a less stringent basis.  
The range of IANA registered ports is *1024 to 41951*. (Not all port numbers in this range are registered.)

In most TCP/IP implementations (including Linux), the port numbers in the range *0 to 1023* are also **privileged**, meaning that only privileged processes may bind to these ports.

> Sometimes, privileged ports are referred to as **reserved** ports

#### Ephemeral ports

If an application doesn’t select a particular port (i.e., in sockets terminology, it doesn’t `bind()` its socket to a particular port), then TCP and UDP assign a unique
**ephemeral** port (i.e., short-lived) number to the socket.

> TCP and UDP also assign an ephemeral port number if we bind a socket to port 0.

IANA specifies the ports in the range 49152 to 65535 as **dynamic** or **private**, with the intention that these ports can be used by local applications and assigned as ephemeral ports.

On Linux, the range of ephemeral port is defined by (and can be modified via) two numbers contained in the file `/proc/sys/net/ipv4/ip_local_port_range`.

### User Datagram Protocol (UDP)

UDP adds just two features to IP:

- port numbers and
- a data checksum to allow the detection of errors in the transmitted data.

> Like IP, UDP is connectionless. Since it adds no reliability to IP, UDP is likewise unreliable.

#### Selecting a UDP datagram size to avoid IP fragmentation

While TCP contains mechanisms for avoiding IP fragmentation, UDP does not.

UDP-based applications that aim to avoid IP fragmentation typically adopt a conservative approach, which is to ensure that the transmitted IP datagram is less than the IPv4 minimum reassembly buffer size of **576 bytes**.

In practice, many UDP-based applications opt for a still lower limit of 512 bytes for their datagrams

### Transmission Control Protocol (TCP)

TCP provides a reliable, connection-oriented, bidirectional, byte-stream communication channel between two endpoints.

#### Connection establishment

Before communication can commence, TCP establishes a communication channel between the two endpoints. During connection establishment, the sender and receiver can exchange options to advertise parameters for the connection.

#### Packaging of data in segments

Data is broken into segments, each of which contains a checksum to allow the detection of end-to-end transmission errors. Each segment is transmitted in a single IP datagram.

#### Acknowledgements, retransmissions, and timeouts

When a TCP segment arrives at its destination without errors, the receiving TCP sends a positive acknowledgement to the sender, informing it of the successfully delivered data.

If a segment arrives with errors, then it is discarded, and no acknowledgement is sent.

To handle the possibility of segments that never arrive or are discarded, the sender starts a timer when each segment is transmitted. If an acknowledgement is not received before the timer expires, the segment is retransmitted.

#### Sequencing

Each byte that is transmitted over a TCP connection is assigned a logical sequence number. This number indicates the position of that byte in the data stream for the connection.

Attaching sequence numbers to each segment serves a variety of purposes:

- The sequence number allows TCP segments to be assembled in the correct order at the destination, and then passed as a byte stream to the application layer.

- The acknowledgement message passed from the receiver back to the sender can use the sequence number to identify which TCP segment was received.

- The receiver can use the sequence number to eliminate duplicate segments.

#### Flow control

Flow control prevents a **fast sender** from overwhelming a **slow receiver**.

To implement flow control, the receiving TCP maintains a **buffer** for incoming data.

The TCP flow-control algorithm employs a so-called **sliding window** algorithm, which allows unacknowledged segments containing a total of up `N` (the offered window size) bytes to be in transit between the sender and receiver.
> If a receiving TCP’s incoming data buffer fills completely, then the window is said to be closed, and the sending TCP stops transmitting.

#### Congestion control: slow-start and congestion-avoidance algorithms

TCP’s congestion-control algorithms are designed to prevent a **fast sender** from overwhelming a network.  
If a sending TCP transmits packets faster than they can be relayed by an intervening router, that router will start dropping packets.

> This could lead to high rates of packet loss and, consequently, serious performance degradation, if the sending TCP kept retransmitting these dropped segments at the same rate.

TCP’s congestion-control algorithms are important in two circumstances:

- *After connection establishment*: At this time, the sender could start by immediately injecting as many segments into the network as would be permitted by the window size advertised by the receiver.  
The problem here is that if the network can’t handle this flood of segments, the sender risks overwhelming the network immediately.

- *When congestion is detected*: If the sending TCP detects that congestion is occurring, then it must reduce its transmission rate.  
TCP detects that congestion is occurring based on the assumption that segment loss because of transmission errors is very low; thus, if a packet is lost, the cause is assumed to be congestion.

TCP’s congestion-control strategy employs two algorithms in combination:

- slow start and
- congestion avoidance.

The **slow-start** algorithm causes the sending TCP to initially transmit segments at a slow rate, but allows it to exponentially increase the rate as these segments are acknowledged by the receiving TCP.

However, if unrestrained, slow start’s exponential increase in the transmission rate could mean that the sender would soon *overwhelm the network*.

TCP’s **congestion-avoidance** algorithm prevents this, by placing a **governor** on the *rate increase*.

In congestion-avoidance algorithm, as the sender receives acknowledgements from the peer
TCP, the congestion window initially grows **exponentially**.

However, once the congestion window reaches a certain threshold believed to be close to the transmission capacity of the network, its growth becomes **linear**, rather than exponential.

---

## Requests for Comments (RFCs)

Each of the Internet protocols that we discuss is defined in an RFC document—a formal protocol specification.

---

## END
