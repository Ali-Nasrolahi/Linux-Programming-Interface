# Notes From ***SOCKETS: INTERNET DOMAINS***

- [Notes From ***SOCKETS: INTERNET DOMAINS***](#notes-from-sockets-internet-domains)
  - [Internet Domain Sockets](#internet-domain-sockets)
  - [Network Byte Order](#network-byte-order)

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
