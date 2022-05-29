# Notes From ***SOCKETS: INTRODUCTION***

- [Notes From ***SOCKETS: INTRODUCTION***](#notes-from-sockets-introduction)
  - [Overview](#overview)
    - [Communication domains](#communication-domains)
    - [Socket types](#socket-types)
    - [Socket system calls](#socket-system-calls)
  - [Creating a Socket: `socket()`](#creating-a-socket-socket)
  - [Binding a Socket to an Address: `bind()`](#binding-a-socket-to-an-address-bind)
  - [Generic Socket Address Structures: `struct sockaddr`](#generic-socket-address-structures-struct-sockaddr)
  - [Stream Sockets](#stream-sockets)
    - [Active and passive sockets](#active-and-passive-sockets)
    - [Listening for Incoming Connections: `listen()`](#listening-for-incoming-connections-listen)
    - [Accepting a Connection: `accept()`](#accepting-a-connection-accept)
    - [Connecting to a Peer Socket: `connect()`](#connecting-to-a-peer-socket-connect)
    - [I/O on Stream Sockets](#io-on-stream-sockets)
    - [Connection Termination: `close()`](#connection-termination-close)
  - [Datagram Sockets](#datagram-sockets)
    - [Exchanging Datagrams: `recvfrom()` and `sendto()`](#exchanging-datagrams-recvfrom-and-sendto)
    - [Using `connect()` with Datagram Sockets](#using-connect-with-datagram-sockets)
  - [END](#end)

**Sockets** are a method of IPC that allow data to be exchanged between applications, either on the same host (computer) or on different hosts connected by a network.

## Overview

In a typical client-server scenario, applications communicate using sockets as follows:

- Each application creates a socket.
    > A socket is the “apparatus” that allows communication, and both applications require one.
- The server binds its socket to a well-known address (name) so that clients can locate it.

A socket is created using the `socket()` system call, which returns a *file descriptor* used to refer to the socket in subsequent system calls:

```c
fd = socket(domain, type, protocol);
```

### Communication domains

Sockets exist in a *communication domain*, which determines:

- the method of identifying a socket
- he range of communication

Modern operating systems support at least the following domains:

- The UNIX ( `AF_UNIX` ) domain allows communication between applications on the same host.

- The IPv4 ( `AF_INET` ) domain allows communication between applications running on hosts connected via an Internet Protocol version 4 (IPv4) network.

- The IPv6 ( `AF_INET6` ) domain allows communication between applications running on hosts connected via an Internet Protocol version 6 (IPv6) network.

### Socket types

Every sockets implementation provides at least two types of sockets: **stream** and **datagram**.

**Stream** sockets ( `SOCK_STREAM` ) provide a *reliable*, *bidirectional*, *byte-stream* communication channel.

By the terms in this description, we mean the following:

- **Reliable** means that we are guaranteed that either the transmitted data will arrive intact at the receiving application, exactly as it was transmitted by the sender, or that we’ll receive notification of a probable failure in transmission.

- **Bidirectional** means that data may be transmitted in either direction between two sockets.

- **Byte-stream** means that, as with pipes, there is no concept of message boundaries.

Stream sockets operate in connected **pairs**. For this reason, stream sockets are described as **connection-oriented**.

- The term **peer** socket refers to the socket at the other end of a connection;
- *peer address* denotes the address of that socket;
- and *peer application* denotes the application utilizing the peer socket.
     > Sometimes, the term **remote** (or foreign) is used synonymously with peer. And **local** as this end of connection.

**Datagram sockets** ( `SOCK_DGRAM` ) allow data to be exchanged in the form of messages called datagrams.

> With datagram sockets, message boundaries are preserved, but data transmission is not reliable.  
> Datagram sockets are an example of the more generic concept of a **connectionless** socket. A datagram socket **doesn’t** need to be *connected* to another socket in order to be used.

In the Internet domain, datagram sockets employ the *User Datagram Protocol* (UDP), and stream sockets (usually) employ the *Transmission Control Protocol* (TCP).

### Socket system calls

The key socket system calls are the following:

- The `socket()` system call creates a new socket.

- The `bind()` system call binds a socket to an address. Usually, a server employs this call to bind its socket to a well-known address so that clients can locate the socket.

- The `listen()` system call allows a stream socket to accept incoming connections from other sockets.

- The `accept()` system call accepts a connection from a peer application on a listening stream socket, and optionally returns the address of the peer socket.

- The `connect()` system call establishes a connection with another socket.

---

## Creating a Socket: `socket()`

The `socket()` system call creates a new socket.

```c
int socket(int domain , int type , int protocol );
```

---

## Binding a Socket to an Address: `bind()`

The `bind()` system call binds a socket to an address.

```c
int bind(int sockfd , const struct sockaddr * addr , socklen_t addrlen );
```

---

## Generic Socket Address Structures: `struct sockaddr`

The *addr* and *addrlen* arguments to `bind()` require some further explanation. Each socket domain uses a different address format

However, because system calls such as `bind()` are **generic** to all socket domains, they must be able to accept address structures of any type.
In order to permit this, the sockets API defines a generic address structure, `struct sockaddr`.

The only purpose for this type is to cast the various domain-specific address structures to a single type for use as arguments in the socket system calls.

---

## Stream Sockets

The operation of stream sockets can be explained by analogy with the telephone system:

1. The `socket()` system call, which creates a socket, is the equivalent of installing a telephone.
    > In order for two applications to communicate, each of them must create a socket.
2. Communication via a stream socket is analogous to a telephone call. One application must connect its socket to another application’s socket before communication can take place.
    - One application calls `bind()` in order to bind the socket to a well-known address, and then calls `listen()` to notify the kernel of its willingness to accept incoming connections.
        > having a known telephone number and ensuring that our telephone is turned on so that people can call us.

    - The other application establishes the connection by calling `connect()`, specifying the address of the socket to which the connection is to be made.
        > dialing someone’s telephone number.

    - The application that called `listen()` then accepts the connection using `accept()`.
        > picking up the telephone when it rings

3. Once a connection has been established, data can be transmitted in both directions between the applications (analogous to a two-way telephone conversation) until one of them closes the connection using `close()`.

### Active and passive sockets

Stream sockets are often distinguished as being either active or passive:

- By default, a socket that has been created using `socket()` is **active**.
    > An active socket can be used in a `connect()` call to establish a connection to a passive socket. This is referred to as performing an *active open*.
- A **passive** socket (also called a *listening socket*) is one that has been marked to allow incoming connections by calling`listen()`.
    > Accepting an incoming connection is referred to as performing a passive open.

In most applications that employ stream sockets, the **server** performs the *passive open*, and the **client** performs the *active open*.

### Listening for Incoming Connections: `listen()`

The `listen()` system call marks the stream socket referred to by the file descriptor *sockfd* as *passive*.

```c
int listen(int sockfd , int backlog );
```

### Accepting a Connection: `accept()`

The `accept()` system call accepts an incoming connection on the listening stream socket referred to by the file descriptor *sockfd*.

```c
int accept(int sockfd , struct sockaddr * addr , socklen_t * addrlen );
```

### Connecting to a Peer Socket: `connect()`

The `connect()` system call connects the active socket referred to by the file descriptor sockfd to the listening socket whose address is specified by *addr* and *addrlen*.

```c
int connect(int sockfd , const struct sockaddr * addr , socklen_t addrlen );
```

### I/O on Stream Sockets

A pair of connected stream sockets provides a bidirectional communication channel between the two endpoints.

The semantics of I/O on connected stream sockets are similar to those for pipes:

- To perform I/O, we use the `read()` and `write()` system calls (or the socket-specific `send()` and `recv()`). Since sockets are bidirectional, both calls may be used on each end of the connection.

- A socket may be closed using the `close()` system call or as a consequence of the application terminating.

### Connection Termination: `close()`

The usual way of terminating a stream socket connection is to call`close()`.

```c
int close(int fd);
```

---

## Datagram Sockets

UDP connections are smiliar enough to TCP ones, therefore only differences are mentioned here.

check pages *1159-1160* for full explanation.

- To send a datagram, an application calls `sendto()`, which takes as one of its arguments the **address** of the socket to which the datagram is to be sent.

- In order to receive a datagram, an application calls `recvfrom()`, which may block if no datagram has yet arrived. Because `recvfrom()` allows us to obtain the address of the sender, we can send a reply if desired.

### Exchanging Datagrams: `recvfrom()` and `sendto()`

The `recvfrom()` and `sendto()` system calls receive and send datagrams on a datagram socket.

```c
ssize_t recvfrom(int sockfd , void * buffer , size_t length , int flags ,
    struct sockaddr * src_addr , socklen_t * addrlen );

ssize_t sendto(int sockfd , const void * buffer , size_t length , int flags ,
    const struct sockaddr * dest_addr , socklen_t addrlen );
```

### Using `connect()` with Datagram Sockets

Even though datagram sockets are **connectionless**, the `connect()` system call serves a purpose when applied to datagram sockets.

Calling `connect()` on a datagram socket causes the kernel to record a particular address as this socket’s peer.
The term **connected datagram** socket is applied to such a socket. The term **unconnected datagram** socket is applied to a datagram socket on which `connect()` has *not* been called.

After a datagram socket has been connected:

- Datagrams can be sent through the socket using `write()` (or `send()`) and are automatically sent to the same peer socket.

- Only datagrams sent by the peer socket may be read on the socket.

The obvious advantage of setting the peer for a datagram socket is that we can use simpler I/O system calls when transmitting data on the socket.

---

## END
