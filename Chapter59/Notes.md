# Notes From ***SOCKETS: INTERNET DOMAINS***

- [Notes From ***SOCKETS: INTERNET DOMAINS***](#notes-from-sockets-internet-domains)
  - [Internet Domain Sockets](#internet-domain-sockets)
  - [Network Byte Order](#network-byte-order)
  - [Data Representation](#data-representation)
  - [Internet Socket Addresses](#internet-socket-addresses)
    - [IPv4 socket addresses: `struct sockaddr_in`](#ipv4-socket-addresses-struct-sockaddr_in)
    - [IPv6 socket addresses: `struct sockaddr_in6`](#ipv6-socket-addresses-struct-sockaddr_in6)
      - [The `sockaddr_storage` structure](#the-sockaddr_storage-structure)
  - [Overview of Host and Service Conversion Functions](#overview-of-host-and-service-conversion-functions)
    - [Converting IPv4 addresses between binary and human-readable forms](#converting-ipv4-addresses-between-binary-and-human-readable-forms)
    - [Converting IPv4 and IPv6 addresses between binary and human-readable forms](#converting-ipv4-and-ipv6-addresses-between-binary-and-human-readable-forms)
    - [Converting host and service names to and from binary form (obsolete)](#converting-host-and-service-names-to-and-from-binary-form-obsolete)
    - [Converting host and service names to and from binary form (modern)](#converting-host-and-service-names-to-and-from-binary-form-modern)
  - [The `inet_pton()` and `inet_ntop()` Functions](#the-inet_pton-and-inet_ntop-functions)
  - [Client-Server Example (Datagram Sockets)](#client-server-example-datagram-sockets)
  - [Domain Name System (DNS)](#domain-name-system-dns)
  - [The `/etc/services` File](#the-etcservices-file)
  - [Protocol-Independent Host and Service Conversion](#protocol-independent-host-and-service-conversion)
    - [The `getaddrinfo()` Function](#the-getaddrinfo-function)

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

Because of differences in data representation, applications that exchange data between heterogeneous systems over a network must adopt some common convention for encoding that data.

The process of putting data into a standard format for transmission across a network is
referred to as **marshalling**.

---

## Internet Socket Addresses

### IPv4 socket addresses: `struct sockaddr_in`

An IPv4 socket address is stored in a `sockaddr_in` structure, defined in `<netinet/in.h>`

### IPv6 socket addresses: `struct sockaddr_in6`

Like an IPv4 address, an IPv6 socket address includes an IP address plus a port number. The difference is that an IPv6 address is **128 bits** instead of **32 bits**.

An IPv6 socket address is stored in a sockaddr_in6 structure, defined in `<netinet/in.h>`

Unlike their IPv4 counterparts, the IPv6 constant and variable initializers are in **network byte** order.

#### The `sockaddr_storage` structure

With the IPv6 sockets API, the new generic sockaddr_storage structure was introduced. This structure is defined to be large enough to hold any type of socket address

---

## Overview of Host and Service Conversion Functions

A **hostname** is the symbolic identifier for a system that is connected to a network
(possibly with multiple *IP addresses*).

A **service** name is the symbolic representation of a *port number*.

The following methods are available for representing host addresses and ports:

- A host address can be represented as a binary value, as a symbolic hostname, or in presentation format

- A port can be represented as a binary value or as a symbolic service name.

### Converting IPv4 addresses between binary and human-readable forms

The `inet_aton()` and `inet_ntoa()` functions convert an IPv4 address in dotted-decimal notation to binary and vice versa.

**Nowadays, they are obsolete.**

### Converting IPv4 and IPv6 addresses between binary and human-readable forms

The `inet_pton()` and `inet_ntop()` functions are like `inet_aton()` and `inet_ntoa()`, but differ in that they also handle *IPv6 addresses*.  
They convert binary IPv4 and IPv6 addresses to and from presentation format.
> that is, either dotted-decimal or hex-string notation.

### Converting host and service names to and from binary form (obsolete)

The `gethostbyname()` function returns the **binary IP** address(es) corresponding to a **hostname** and the `getservbyname()` function returns the **port number** corresponding
to a **service** name.

The reverse conversions are performed by `gethostbyaddr()` and `getservbyport()`.

### Converting host and service names to and from binary form (modern)

The `getaddrinfo()` function is the modern successor to both `gethostbyname()` and`getservbyname().`

The `getnameinfo()` function performs the reverse translation, converting an IP address and port number into the corresponding hostname and service name.

> We can also use `getaddrinfo()` and `getnameinfo()` to convert binary IP addresses to and from presentation format.

---

## The `inet_pton()` and `inet_ntop()` Functions

The `inet_pton()` and `inet_ntop()` functions allow conversion of both IPv4 and IPv6 addresses between binary form and dotted-decimal or hex-string notation.

```c
int inet_pton(int domain , const char * src_str , void * addrptr );

const char *inet_ntop(int domain , const void * addrptr , char * dst_str , size_t len );
```

---

## Client-Server Example (Datagram Sockets)

check pages *1207-1209* for an example of IPv6 application.

---

## Domain Name System (DNS)

Before the advent of DNS, mappings between hostnames and IP addresses were defined in a manually maintained local file, `/etc/hosts`.

check pages *1209-1212* for full explanation.

---

## The `/etc/services` File

Well-known port numbers are centrally registered by IANA. Each of these ports has a corresponding **service name**.

Because service numbers are centrally managed and are less volatile than IP addresses, an equivalent of the DNS server is usually not necessary. Instead, the port numbers and service names are recorded in the file `/etc/services`.

---

## Protocol-Independent Host and Service Conversion

The `getaddrinfo()` function converts host and service names to IP addresses and port numbers.
> It was defined in POSIX.1g as the (**reentrant**) successor to the obsolete `gethostbyname()` and `getservbyname()` functions.

The `getnameinfo()` function is the converse of `getaddrinfo()`. It translates a socket
address structure (either IPv4 or IPv6) to strings containing the corresponding **host** and **service** name.
> This function is the (**reentrant**) equivalent of the obsolete `gethostbyaddr()` and `getservbyport()` functions.

### The `getaddrinfo()` Function

Given a **host name** and a **service name**, `getaddrinfo()` returns a list of socket address structures, each of which contains an IP address and port number.

```c
int getaddrinfo(const char * host , const char * service ,
                const struct addrinfo * hints , struct addrinfo ** result );
```
