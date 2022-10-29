# Notes From ***INTERPROCESS COMMUNICATION OVERVIEW***

- [Notes From ***INTERPROCESS COMMUNICATION OVERVIEW***](#notes-from-interprocess-communication-overview)
  - [A Taxonomy of IPC Facilities](#a-taxonomy-of-ipc-facilities)
  - [Communication Facilities](#communication-facilities)
    - [Data transfer](#data-transfer)
    - [Shared memory](#shared-memory)
  - [Synchronization Facilities](#synchronization-facilities)
  - [Comparing IPC Facilities](#comparing-ipc-facilities)
    - [Functionality](#functionality)
    - [Network communication](#network-communication)
  - [END](#end)

## A Taxonomy of IPC Facilities

- **Communication**: These facilities are concerned with exchanging data between processes.

- **Synchronization**: These facilities are concerned with synchronizing the actions of processes or threads.

- **Signals**: Although signals are intended primarily for other purposes, they can be used as a synchronization technique in certain circumstances. More rarely, signals can be used as a communication technique: the signal number itself is a form of information, and realtime signals can be accompanied by associated data (an integer or a pointer).

## Communication Facilities

We can break the communication facilities into two categories:

- **Data-transfer facilities**: The key factor distinguishing these facilities is the notion of writing and reading. In order to communicate, one process writes data to the IPC facility, and another process reads the data.
  - These facilities require two data transfers between user memory and kernel memory: one transfer from user memory to kernel memory during writing, and another transfer from kernel memory to user memory during reading.

- **Shared memory**: Shared memory allows processes to exchange information by placing it in a region of memory that is shared between the processes.
  - A process can make data available to other processes by placing it in the shared memory region. Because communication doesn’t require system calls or data transfer between user memory and kernel memory, shared memory can provide very **fast communication**.

### Data transfer

We can further break data-transfer facilities into the following subcategories:

- **Byte stream**: The data exchanged via *pipes*, *FIFOs*, and *datagram sockets* is an undelimited byte stream. Each read operation may read an arbitrary number of bytes from the IPC facility, regardless of the size of blocks written by the writer.

- **Message**: The data exchanged via *System V message queues*, *POSIX message queues*, and *datagram sockets* takes the form of delimited messages. Each read operation reads a whole message, as written by the writer process. It is not possible to read part of a message, leaving the remainder on the IPC facility; nor is it possible to read multiple messages in a single read operation.

- **Pseudoterminals**: A pseudoterminal is a communication facility intended for use in specialized situations.

A few general features distinguish data-transfer facilities from shared memory:

- Although a data-transfer facility may have multiple readers, reads are destructive. A read operation consumes data, and that data is not available to any other process.

- Synchronization between the reader and writer processes is automatic. If a reader attempts to fetch data from a data-transfer facility that currently has no data, then (by default) the read operation will block until some process writes data to the facility.

### Shared memory

Most modern UNIX systems provide three flavors of shared memory: System V shared memory, POSIX shared memory, and memory mappings.

- Although shared memory provides fast communication, this speed advantage is offset by the need to synchronize operations on the shared memory.
  - > For example, one process should not attempt to access a data structure in the shared memory while another process is updating it. A semaphore is the usual synchronization method used with shared memory.

- Data placed in shared memory is visible to all of the processes that share that memory.
  - > (This contrasts with the destructive read semantics described above for data-transfer facilities.)

## Synchronization Facilities

**Synchronization** allows processes to avoid doing things such as *simultaneously* updating a shared memory region or the same part of a file.
> Without synchronization, such simultaneous updates could cause an application to produce incorrect results.

UNIX systems provide the following synchronization facilities:

- **Semaphores**: A semaphore is a kernel-maintained integer whose value is never permitted to fall below `0`. A process can *decrease* or *increase* the value of a semaphore. If an attempt is made to decrease the value of the semaphore below `0`, then the kernel blocks the operation until the semaphore’s value increases to a level that permits the operation to be performed.

- **File locks**: File locks are a synchronization method explicitly designed to coordinate the actions of multiple processes operating on the same file. They can also be used to coordinate access to other shared resources. File locks come in two flavors: *read (shared)* locks and *write (exclusive)* locks. Any number of processes can hold a read lock on the same file (or region of a file). However, when one process holds a write lock on a file (or file region), other processes are prevented from holding either read or write locks on that file (or file region).

- **Mutexes and condition variables**: These synchronization facilities are normally used with POSIX threads When performing interprocess synchronization, our choice of facility is typically determined by the functional requirements. When coordinating access to a file, file record locking is usually the best choice. Semaphores are often the better choice for coordinating access to other types of shared resource.

## Comparing IPC Facilities

### Functionality

There are functional differences between the various IPC facilities that can be relevant in determining which facility to use. We begin by summarizing the differences between data-transfer facilities and shared memory:

- **Data-transfer facilities** involve read and write operations, with transferred data being consumable by just one reader process.

- Other application designs more naturally suit a **shared-memory model**. Shared memory allows one process to make data visible to any number of other processes sharing the same memory region.  Communication “operations” are simple—a process can access data in shared memory in the same manner as it accesses any other memory in its virtual address space. On the other hand, the need to handle synchronization (and perhaps flow control) can add to the complexity of a shared-memory design. This model fits well with application designs that need to maintain shared state

With respect to the various data-transfer facilities, the following points are worth noting

- Some data-transfer facilities transfer data as a **byte stream** (pipes, FIFOs, and stream sockets); others are **message-oriented** (message queues and datagram sockets). Which approach is preferable depends on the application.

- A distinctive feature of System V and POSIX message queues, compared with other data-transfer facilities, is the ability to assign a numeric type or priority to a message, so that messages can be delivered in a different order from that in which they were sent.

- Pipes, FIFOs, and sockets are implemented using file descriptors. These IPC facilities all support a range of alternative I/O models: I/O multiplexing (the `select()` and `poll()` system calls), signaldriven I/O, and the Linux-specific `epoll` API. The primary benefit of these techniques is that they allow an application to simultaneously monitor multiple file descriptors to see whether I/O is possible on any of them.

- POSIX message queues provide a notification facility that can send a signal to a process, or instantiate a new thread, when a message arrives on a previously empty queue.

- UNIX domain sockets provide a feature that allows a file descriptor to be passed from one process to another.

- UDP (Internet domain datagram) sockets allow a sender to broadcast or multicast a message to multiple recipients.

### Network communication

Of all of the IPC methods shown, only **sockets** permit processes to communicate over a network. Sockets are generally used in one of two domains: the UNIX domain, which allows communication between processes on the same system, and the Internet domain, which allows communication between processes on different hosts connected via a TCP/IP network.

## END
