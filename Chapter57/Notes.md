# Notes From ***SOCKETS: UNIX DOMAIN***

- [Notes From ***SOCKETS: UNIX DOMAIN***](#notes-from-sockets-unix-domain)
  - [UNIX Domain Socket Addresses: `struct sockaddr_un`](#unix-domain-socket-addresses-struct-sockaddr_un)
  - [Stream Sockets in the UNIX Domain](#stream-sockets-in-the-unix-domain)
  - [](#)

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

## 