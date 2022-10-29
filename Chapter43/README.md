# Notes From ***INTERPROCESS COMMUNICATION OVERVIEW***

- [Notes From ***INTERPROCESS COMMUNICATION OVERVIEW***](#notes-from-interprocess-communication-overview)
  - [A Taxonomy of IPC Facilities](#a-taxonomy-of-ipc-facilities)
  - [Communication Facilities](#communication-facilities)
    - [Data transfer](#data-transfer)
    - [Shared memory](#shared-memory)
  - [Synchronization Facilities](#synchronization-facilities)


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
