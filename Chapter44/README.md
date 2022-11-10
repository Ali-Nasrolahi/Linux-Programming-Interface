# Notes From ***PIPES AND FIFOS***

- [Notes From ***PIPES AND FIFOS***](#notes-from-pipes-and-fifos)
  - [Overview](#overview)
    - [A pipe is a byte stream](#a-pipe-is-a-byte-stream)
    - [Reading from a pipe](#reading-from-a-pipe)
    - [Pipes are unidirectional](#pipes-are-unidirectional)
    - [Writes of up to `PIPE_BUF` bytes are guaranteed to be atomic](#writes-of-up-to-pipe_buf-bytes-are-guaranteed-to-be-atomic)
  - [Creating and Using Pipes](#creating-and-using-pipes)
    - [Pipes allow communication between related processes](#pipes-allow-communication-between-related-processes)
    - [Closing unused pipe file descriptors](#closing-unused-pipe-file-descriptors)
  - [Pipes as a Method of Process Synchronization](#pipes-as-a-method-of-process-synchronization)
  - [Using Pipes to Connect Filters](#using-pipes-to-connect-filters)
  - [Talking to a Shell Command via a Pipe: `popen()`](#talking-to-a-shell-command-via-a-pipe-popen)
  - [Pipes and stdio Buffering](#pipes-and-stdio-buffering)
  - [FIFOs](#fifos)
  - [A Client-Server Application Using FIFOs](#a-client-server-application-using-fifos)
  - [Nonblocking I/O](#nonblocking-io)
  - [Semantics of `read()` and `write()` on Pipes and FIFOs](#semantics-of-read-and-write-on-pipes-and-fifos)
  - [END](#end)

## Overview

Two processes are unaware of the existence of the pipe; they just read from and write to the standard file descriptors. The shell must do some work in order to set things up in this way.

### A pipe is a byte stream

When we say that a pipe is a byte stream, we mean that there is no concept of messages or message boundaries when using a pipe. The process reading from a pipe can read blocks of data of any size, regardless of the size of blocks written by the writing process. Furthermore, the data passes through the pipe sequentially—bytes are read from a pipe in exactly the order they were written. It is not possible to randomly access the data in a pipe using `lseek()`.

### Reading from a pipe

Attempts to read from a pipe that is currently empty block until at least one byte has been written to the pipe. If the write end of a pipe is closed, then a process reading from the pipe will see **end-of-file**.

### Pipes are unidirectional

Data can travel only in one direction through a pipe. One end of the pipe is used for writing, and the other end is used for reading.

### Writes of up to `PIPE_BUF` bytes are guaranteed to be atomic

If multiple processes are writing to a single pipe, then it is guaranteed that their data won’t be intermingled if they write no more than `PIPE_BUF` bytes at a time.

The `PIPE_BUF` limit affects exactly when data is transferred to the pipe. When writing up to `PIPE_BUF` bytes, `write()` will block if necessary until sufficient space is available in the pipe so that it can complete the operation atomically. When more than `PIPE_BUF` bytes are being written, write() transfers as much data as possible to fill the pipe, and then blocks until data has been removed from the pipe by some reading process.

A pipe is simply a buffer maintained in kernel memory. This buffer has a maximum capacity. Once a pipe is full, further writes to the pipe block until the reader removes some data from the pipe.

---

## Creating and Using Pipes

The `pipe()` system call creates a new pipe:

```c
int pipe(int filedes[2]);
```

A pipe has few uses within a *single process*.  Normally, we use a pipe to allow communication between two processes. To connect two processes using a pipe, we follow the `pipe()` call with a call to `fork()`. During a `fork(),` the child process **inherits** copies of its parent’s file descriptors. While it is possible for the parent and child to both read from and write to the pipe, this is not usual. Therefore, immediately after the `fork()`, one process **closes** its descriptor for the write end of the pipe, and the other closes its descriptor for the read end.

### Pipes allow communication between related processes

Pipes can be used for communication between any two (or more) **related processes**, as long as the pipe was created by a **common ancestor** before the series of `fork()` calls that led to the existence of
the processes.

### Closing unused pipe file descriptors

Closing unused pipe file descriptors is more than a matter of ensuring that a process doesn’t exhaust its limited set of file descriptors—it is essential to the correct use of pipes. We now consider why the unused file descriptors for both the read and write ends of the pipe must be closed.

The process reading from the pipe closes its write descriptor for the pipe, so that, when the other process completes its output and closes its write descriptor, the reader sees end-of-file.

When a process tries to write to a pipe for which no process has an open read descriptor, the kernel sends the `SIGPIPE` signal to the writing process. By default, this signal kills a process. A process can instead arrange to catch or ignore this signal, in which case the write() on the pipe fails with the error `EPIPE` (broken pipe).

One final reason for closing unused file descriptors is that it is only after all file descriptors in all processes that refer to a pipe are closed that the pipe is destroyed and its resources released for reuse by other processes. At this point, any unread data in the pipe is lost.

---

## Pipes as a Method of Process Synchronization

To perform the synchronization,

1. the parent builds a pipe before creating the child processes.
2. Each child inherits a file descriptor for the write end of the pipe and closes this descriptor once it has completed its action.
3. After all of the children have closed their file descriptors for the write end of the pipe, the parent’s `read()`
4. from the pipe will complete, returning end-of-file (0). At this point, the parent is free to carry on to do other work.

---

## Using Pipes to Connect Filters

---

## Talking to a Shell Command via a Pipe: `popen()`

A common use for pipes is to execute a shell command and either read its output or send it some input. The `popen()` and `pclose()` functions are provided to simplify this task.

```c
FILE *popen(const char *command, const char *mode);

int pclose(FILE *stream);
```

The `popen()` function creates a pipe, and then forks a child process that execs a shell, which in turn creates a child process to execute the string given in command.
he `mode` argument is a string that determines whether the calling process will read from the pipe (mode is r) or write to it (mode is w).

After the `popen()` call, the calling process uses the pipe to read the output of **command** or to send input to it. Just as with pipes created using `pipe()`, when reading from the pipe, the calling process encounters end-of-file once command has closed the write end of the pipe; when writing to the pipe, it is sent a `SIGPIPE` signal, and gets the `EPIPE` error, if command has closed the read end of the pipe.

Once I/O is complete, the `pclose()` function is used to close the pipe and wait for the child shell to terminate.

When performing a wait to obtain the status of the child shell, SUSv3 requires that `pclose(),` like `system()`, should automatically restart the internal call that it makes to `waitpid()` if that call is interrupted by a signal handler.

As with `system()`, `popen()` **should never** be used from privileged programs.

---

## Pipes and stdio Buffering

---

## FIFOs

Semantically, a FIFO is similar to a pipe. The principal difference is that a FIFO has a name within the file system and is opened in the same way as a regular file. This allows a FIFO to be used for communication between unrelated processes.

Once a FIFO has been opened, we use the same I/O system calls as are used with pipes and other files (i.e., `read()`, `write()`, and `close())`. Just as with pipes, a FIFO has a write end and a read end, and data is read from the pipe in the same order as it is written. This fact gives FIFOs their name: first in, first out. FIFOs are also sometimes known as **named pipes**.

The `mkfifo()` function creates a new FIFO with the given pathname

```c
int mkfifo(const char *pathname, mode_t mode);

```

Opening a FIFO has somewhat unusual semantics. Generally, the only sensible use of a FIFO is to have a reading process and a writing process on each end.  Therefore, by default, opening a FIFO for reading (the open() `O_RDONLY` flag) blocks until another process opens the FIFO for writing (the `open()` `O_WRONLY` flag). Conversely, opening the FIFO for writing blocks until another process opens the FIFO for reading.  In other words, opening a FIFO synchronizes the reading and writing processes. If the opposite end of a FIFO is already open (perhaps because a pair of processes have already opened each end of the FIFO), then `open()` succeeds immediately.

---

## A Client-Server Application Using FIFOs

---

## Nonblocking I/O

As noted earlier, when a process opens one end of a FIFO, it **blocks** if the other end of the FIFO has not yet been opened. Sometimes, it is desirable *not to block*, and for this purpose, the `O_NONBLOCK` flag can be specified when calling `open()`.

The `O_NONBLOCK` flag changes things only if the other end of the FIFO is not yet open, and the effect depends on whether we are opening the FIFO for reading or writing:

- If the FIFO is being opened for reading, and no process currently has the write end of the FIFO open, then the `open()` call succeeds immediately ( just as though the other end of the FIFO was already open).

- If the FIFO is being opened FIFO for writing, and the other end of the FIFO is not already open for reading, then `open()` fails, setting errno to `ENXIO`.

Using the `O_NONBLOCK` flag when opening a FIFO serves two main purposes:

- It allows a single process to open both ends of a FIFO. The process first opens the FIFO for reading specifying `O_NONBLOCK`, and then opens the FIFO for writing.

- It prevents deadlocks between processes opening two FIFOs.

---

## Semantics of `read()` and `write()` on Pipes and FIFOs

The only difference between blocking and nonblocking reads occurs when no data is present and the write end is open. In this case, a normal `read()` blocks, while a nonblocking `read()` fails with the error `EAGAIN`.

The `O_NONBLOCK` flag causes a `write()` on a pipe or FIFO to fail (with the error `EAGAIN`) in any case where data can’t be transferred immediately. This means that if we are writing up to `PIPE_BUF` bytes, then the `write()` will fail if there is not sufficient space in the pipe or FIFO, because the kernel can’t complete the operation immediately and can’t perform a partial write, since that would break the requirement that writes of up to `PIPE_BUF` bytes are **atomic**.

---

## END
