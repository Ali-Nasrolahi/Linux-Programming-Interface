# Notes From ***SOCKETS: INTERNET DOMAINS***

- [Notes From ***SOCKETS: INTERNET DOMAINS***](#notes-from-sockets-internet-domains)
  - [Internet Domain Sockets](#internet-domain-sockets)
  - [Network Byte Order](#network-byte-order)
  - [Data Representation](#data-representation)

## Internet Domain Sockets

Internet domain **stream sockets** are implemented on top of **TCP**.
> They provide a reliable, bidirectional, byte-stream communication channel.

Internet domain **datagram sockets** are implemented on top of **UDP**.  
UDP sockets are similar to their UNIX domain counterparts, but note the following differences:

- UNIX domain datagram sockets are reliable, but UDP sockets are not.

- Sending on a UNIX domain datagram socket will block if the queue of data for the receiving socket is full.
    > By contrast, with UDP, if the incoming datagram would overflow the receiver’s queue, then the datagram is silently dropped.

---

## Network Byte Order

One problem we encounter when passing these values across a network is that different hardware architectures store the bytes of a multibyte integer in different orders.

**big endian**: Stores integers with the **most** significant byte first.

**little endian**: those that store the **least** significant byte first.

The byte ordering used on a particular machine is called the **host byte order**.

Since port numbers and IP addresses must be transmitted between, and understood by, all hosts on a network, a standard ordering must be used.

This ordering is called **network byte order**, and happens to be **big endian**.

The `htons()`, `htonl()`, `ntohs()`, and `ntohl()` functions are defined (typically as macros) for converting integers in either direction between host and network byte order.

```c
uint16_t htons(uint16_t host_uint16 );

uint32_t htonl(uint32_t host_uint32 );

uint16_t ntohs(uint16_t net_uint16 );

uint32_t ntohl(uint32_t net_uint32 );
```

---

## Data Representation
