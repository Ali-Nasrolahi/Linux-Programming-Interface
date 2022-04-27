# Notes From ***SOCKETS: UNIX DOMAIN***

- [Notes From ***SOCKETS: UNIX DOMAIN***](#notes-from-sockets-unix-domain)
  - [UNIX Domain Socket Addresses: `struct sockaddr_un`](#unix-domain-socket-addresses-struct-sockaddr_un)
  - [Stream Sockets in the UNIX Domain](#stream-sockets-in-the-unix-domain)
  - [Datagram Sockets in the UNIX Domain](#datagram-sockets-in-the-unix-domain)
    - [Maximum datagram size for UNIX domain datagram sockets](#maximum-datagram-size-for-unix-domain-datagram-sockets)
  - [UNIX Domain Socket Permissions](#unix-domain-socket-permissions)
  - [Creating a Connected Socket Pair: `socketpair()`](#creating-a-connected-socket-pair-socketpair)
  - [The Linux Abstract Socket Namespace](#the-linux-abstract-socket-namespace)
  - [END](#end)

## UNIX Domain Socket Addresses: `struct sockaddr_un`

In the UNIX domain, a socket address takes the form of a pathname.

In order to bind a UNIX domain socket to an address, we initialize a `sockaddr_un` structure, and then pass a (cast) pointer to this structure as the `addr` argument to `bind()`, and specify `addrlen` as the size of the structure.

When used to bind a UNIX domain socket, `bind()` creates an entry in the file system.
> Thus, a directory specified as part of the socket pathname needs to be accessible and writable

The ownership of the file is determined according to the usual rules for file creation. The file is marked as a **socket**.

The following points are worth noting about binding a UNIX domain socket:

- We can’t bind a socket to an existing pathname.

- It is usual to bind a socket to an absolute pathname, so that the socket resides at a fixed address in the file system.

- A socket may be bound to **only one** pathname; conversely, a pathname can be bound to only one socket.

- We can’t use `open()` to open a socket.

- When the socket is no longer required, its pathname entry can be removed using `unlink()` (or`remove()`).

---

## Stream Sockets in the UNIX Domain

Check pages *1167-1170* for an example of server-client communication in UNIX domain.

## Datagram Sockets in the UNIX Domain

We stated that communication using datagram sockets is **unreliable**. This is the case for datagrams transferred over a **network**.  
However, for *UNIX domain* sockets, datagram transmission is carried out within the kernel, and is **reliable**.

> All messages are delivered in order and unduplicated.

### Maximum datagram size for UNIX domain datagram sockets

SUSv3 doesn’t specify a maximum size for datagrams sent via a UNIX domain socket. On Linux, we can send quite large datagrams.

The limits are controlled via the `SO_SNDBUF` socket option and various `/proc` files, as described in the `socket(7)` manual page.

check pages *1171-1174* for an example datagram type of transmission in UNIX domain.

---

## UNIX Domain Socket Permissions

The ownership and permissions of the socket file determine which processes are able to communicate with that socket:

- To connect to a UNIX domain stream socket, **write** permission is required on the socket file.

- To send a datagram to a UNIX domain datagram socket, **write** permission is required on the socket file.

In addition, **execute** (search) permission is required on each of the directories in the socket pathname.

> by default, a socket is created (by bind()) with all permissions granted to owner (user), group, and other.  
> To change this, we can precede the call to bind() with a call to `umask()` to disable the permissions that we do not wish to grant.

---

## Creating a Connected Socket Pair: `socketpair()`

Sometimes, it is useful for a single process to create a pair of sockets and connect them together.

```c
int socketpair(int domain , int type , int protocol , int sockfd [2]);
```

This `socketpair()` system call can be used only in the **UNIX domain**; that is, *domain* must be specified as `AF_UNIX`.

Typically, a socket pair is used in a similar fashion to a pipe. After the `socketpair()` call, the process then creates a child via `fork()`.  
The child inherits copies of the parent’s file descriptors, including the descriptors referring to the socket pair.
> Thus, the parent and child can use the socket pair for IPC.

One way in which the use of `socketpair()` differs from creating a pair of connected sockets manually is that the sockets are **not bound** to any address.
> This can help us avoid a whole class of security vulnerabilities.

---

## The Linux Abstract Socket Namespace

The so-called **abstract namespace** is a Linux-specific feature that allows us to bind a UNIX domain socket to a name **without** that name being **created** in the file system.

This provides a few potential advantages:

- We don’t need to worry about possible collisions with existing names in the file system.

- It is not necessary to unlink the socket pathname when we have finished using the socket.

- We don’t need to create a file-system pathname for the socket.

To create an abstract binding, we specify the first byte of the `sun_path` field as a null byte (`\0` ).

The remaining bytes of the `sun_path` field then define the abstract name for the socket.
> These bytes are interpreted in their entirety, rather than as a null-terminated string.

---

## END
