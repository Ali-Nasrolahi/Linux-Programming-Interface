# Notes From ***FILE I/O: FURTHER DETAILS***

- [Notes From ***FILE I/O: FURTHER DETAILS***](#notes-from-file-io-further-details)
  - [Atomicity and Race Conditions](#atomicity-and-race-conditions)
  - [File Control Operations: `fcntl()`](#file-control-operations-fcntl)
  - [Open File Status Flags](#open-file-status-flags)
  - [Relationship Between File Descriptors and Open Files](#relationship-between-file-descriptors-and-open-files)
    - [*Per-process* file descriptor](#per-process-file-descriptor)
    - [*System-wide* table of all open file descriptions](#system-wide-table-of-all-open-file-descriptions)
    - [i-node table](#i-node-table)
  - [Duplicating File Descriptors](#duplicating-file-descriptors)
  - [File I/O at a Specified Offset: `pread()` and `pwrite()`](#file-io-at-a-specified-offset-pread-and-pwrite)
  - [Scatter-Gather I/O: `readv()` and `writev()`](#scatter-gather-io-readv-and-writev)
    - [Scatter input](#scatter-input)
    - [Gather output](#gather-output)
      - [Question](#question)
      - [Answer](#answer)
    - [Performing scatter-gather I/O at a specified offset](#performing-scatter-gather-io-at-a-specified-offset)
  - [Truncating a File: `truncate()`and `ftruncate()`](#truncating-a-file-truncateand-ftruncate)
  - [Nonblocking I/O](#nonblocking-io)
  - [I/O on Large Files](#io-on-large-files)
    - [The transitional LFS API](#the-transitional-lfs-api)
    - [The `_FILE_OFFSET_BITS` macro](#the-_file_offset_bits-macro)
    - [Passing `off_t` values to `printf()`](#passing-off_t-values-to-printf)
  - [The `/dev/fd` Directory](#the-devfd-directory)
  - [Creating Temporary Files](#creating-temporary-files)
  - [END](#end)

## Atomicity and Race Conditions

**atomicity**:  the notion that the actions performed by a *system call* are executed as a
**single** *uninterruptible* step.

*All* system calls are executed **atomically**.
> By this, we mean that the kernel **guarantees** that all of the steps in a *system call* are completed as a **single** operation, without being *interrupted* by another *process* or *thread*.

**race conditions** (aka. *race hazards*):

is a situation where the result produced by two processes (or threads) operating on *shared resources* depends in an unexpected way on the **relative** *order* in which the processes gain access to the CPU(s).

**Note:** this section has 2 examples of race conditions which might guide you to better understanding.
> To do so check out pages 91 to 92.

---

## File Control Operations: `fcntl()`

Checkout `fnctl(2)` manual for its usage.

---

## Open File Status Flags

One use of `fcntl()` is to **retrieve** or **modify** the *access mode* and *open file status* flags of an open file.

Checking the access mode of the file is slightly more complex.

> ince the O_RDONLY (0), O_WRONLY (1), and O_RDWR (2) constants don’t correspond to single bits in the open file status flags.

Therefore, to make this check, we mask the *flags* (returned by `fnctl(fd, F_GETFL)`) value with the constant `O_ACCMODE`, and then test for equality with one of the constants.

Using `fcntl()` to modify open file status flags is particularly useful in the following cases:

1. The file was not opened by the calling program, so that it had no control over the flags used in the `open()` call.

2. The file descriptor was obtained from a system call other than `open()`.

---

## Relationship Between File Descriptors and Open Files

It is possible — and useful— to have multiple descriptors referring to the same open file.
> These file descriptors may be open in the same process or in different processes.

To understand what is going on, we need to examine three data structures maintained by the kernel:

- the per-process file descriptor table;
- the system-wide table of open file descriptions;
- the file system i-node table.

### *Per-process* file descriptor

Each entry in this table records information about a single file descriptor, including:

- a set of flags controlling the operation of the file descriptor.
    > there is just one such flag, the close-on-exec flag.
- a reference to the open file description.

### *System-wide* table of all open file descriptions

> This table is sometimes referred to as the *open file table*, and its entries are sometimes called *open file handles*.

An open file description stores all information relating to an open file, including:

- The current file offset.
- Status flags specified when opening the file.
- The file access mode.
- Settings relating to signal-driven I/O.
- Reference to the i-node object for this file.

### i-node table

Each file system has a table of *i-nodes* for all files residing in the file system.

The i-node for each file includes the following information:

- file type (e.g., regular file, socket, or FIFO) and permissions;
- a pointer to a list of locks held on this file;
- various properties of the file, including its size and timestamps relating to different types of file operations.

---

## Duplicating File Descriptors

The `dup()` call takes *oldfd*, an open file descriptor, and returns a new descriptor that refers to the **same** open file description.

The new descriptor is guaranteed to be the **lowest** unused file descriptor.

he `dup2()` system call makes a duplicate of the file descriptor given in *oldfd* using the descriptor number supplied in *newfd*.

If the file descriptor specified in *newfd* is already open, `dup2()` **closes** it first.

A further interface that provides some extra flexibility for duplicating file descriptors is the `fcntl()` **F_DUPFD** operation.
> This call makes a duplicate of *oldfd* by using the **lowest** unused file descriptor *greater* than or *equal* to **startfd**.  
> This is useful if we want a guarantee that the new descriptor (newfd) falls in a certain *range of values*.

**Note**:

The new file descriptor has its own set of file descriptor flags, and its close-on-exec flag (`FD_CLOEXEC`) is always **turned off**.

The `dup3()` system call performs the same task as `dup2()`, but adds an additional
argument, *flags*, that is a bit mask that **modifies** the behavior of the system call.

Currently, `dup3()` supports one flag, `O_CLOEXEC`.

> The `dup3()` system call is new in Linux 2.6.27.  
> Since Linux 2.6.24, Linux also supports an additional `fcntl()` operation for duplicating file descriptors: `F_DUPFD_CLOEXEC`.

---

## File I/O at a Specified Offset: `pread()` and `pwrite()`

The `pread()`and `pwrite()`system calls operate just like `read()`and `write()`, except the file I/O is performed at the location specified by **offset**, rather than at the **current** file offset.

> The file offset is left **unchanged** by these calls.

These system calls can be particularly useful in **multithreaded** applications.

All of the threads in a process share the same file descriptor table. This means that the file offset for each open file is **global** to all threads.

Using `pread()`or `pwrite()`, multiple threads can **simultaneously** perform I/O on the same file descriptor *without* being affected by changes made to the file *offset* by other threads.

> If we attempted to use `lseek()` plus `read()` (or `write()`) instead, then we would create a **race condition**.

---

## Scatter-Gather I/O: `readv()` and `writev()`

The `readv()` and `writev()` system calls perform **scatter-gather** I/O.

Instead of accepting a **single** buffer of data to be *read* or *written*, these functions transfer **multiple** buffers of data in a single system call.

### Scatter input

The `readv()` system call performs *scatter* input.

It reads a **contiguous** sequence of bytes from the file referred to by the file descriptor `fd` and places (“*scatters*”) these bytes into the buffers specified by `iov`.

Each of the buffers, starting with the one defined by `iov[0]`, is **completely** filled before `readv()` proceeds to the *next* buffer.

An important property of `readv()` is that it completes **atomically**; the kernel performs a *single* data transfer between the file referred to by `fd` and *user memory*.

> This means, for example, that when reading from a file, we can be sure that the range of bytes read is **contiguous**, even if another process (or thread) sharing the same file *offset* attempts to manipulate the *offset* at the **same time** as the `readv()` call.

### Gather output

he `writev()` system call performs **gather output**.

It concatenates (“*gathers*”) data from all of the buffers specified by `iov` and writes them as a sequence of **contiguous** bytes to the file referred to by the file descriptor `fd`.

> The buffers are gathered in array order, starting with the buffer defined by `iov[0]`.

Like `readv()`, `writev()` completes **atomically**, with all data being transferred in a *single* operation from user memory to the file referred to by `fd`.

The primary advantages of `readv()` and `writev()` are *convenience* and *speed*.

#### Question

Why should we use scatter-gather I/O? (i.e. `writev()`)

For example, we could replace a call to `writev()` by either:

- code that allocates a single **large** buffer, *copies* the data to be written from other locations in the process’s address space into that buffer, and *then* calls `write()` to output the buffer;

- a **series** of `write()` calls that output the buffers *individually*.

#### Answer

The first of these options, while semantically equivalent to using`writev()`, leaves us with the **inconvenience** (and inefficiency) of *allocating* buffers and *copying* data in user space.

The second option is *not* semantically equivalent to a single call to `writev(),` since the `write()` calls are *not* performed **atomically**.

> Furthermore, performing a **single** `writev()` system call is **cheaper** than performing **multiple** `write()` calls

### Performing scatter-gather I/O at a specified offset

Linux 2.6.30 adds two new system calls that combine scatter-gather I/O functionality with the ability to perform the I/O at a specified offset:

`preadv()` and`pwritev()`.

> These system calls are nonstandard, but are also available on the modern BSDs.

These system calls are useful for applications (e.g., **multithreaded** applications) that want to *combine* the benefits of `scatter-gather I/O` with the ability to *perform I/O* at a location that is **independent** of the current file `offset`.

---

## Truncating a File: `truncate()`and `ftruncate()`

The `truncate()` and `ftruncate()` system calls set the *size* of a file to the value specified by`length`.

- If the file is *longer* than `length`, the excess data is **lost**.
- If the file is currently *shorter* than `length`, it is extended by padding with a sequence of *null bytes* or a *hole*.

The difference between the two system calls lies in how the file is specified.

With `truncate(),` the file, which must be **accessible** and **writable**, is specified as a `pathname` string.

The `ftruncate()` system call takes a *descriptor* for a file that has been opened for writing.

> It doesn’t change the file offset for the file.  
> The `truncate()` system call is unique in being the **only** system call that can change the **contents** of a file without first obtaining a descriptor for the file via `open()`.

---

## Nonblocking I/O

Specifying the `O_NONBLOCK` flag when opening a file serves two purposes:

- If the file can’t be opened immediately, then `open()` returns an error instead of **blocking**.

- After a successful `open(),` subsequent I/O operations are also **nonblocking**.
    > If an I/O system call can’t complete **immediately**, then either a **partial** data transfer is performed or the system call **fails** with one of the errors `EAGAIN` or `EWOULDBLOCK`.

---

## I/O on Large Files

We can write applications requiring **LFS** (*Large File Summit*) functionality in one of two ways:

- Use an alternative API that supports large files.
    > This API was designed by the LFS as a “transitional extension” to the Single UNIX Specification. Thus, this API is not required to be present on systems conforming to SUSv2 or SUSv3, but many conforming systems do provide it.  
    > This approach is now **obsolete**.
- Define the `_FILE_OFFSET_BITS` macro with the value **64** when compiling our programs.
  > This is the preferred approach, because it allows conforming applications to obtain LFS functionality without making any source code changes.

### The transitional LFS API

To use the transitional LFS API, we must define the `_LARGEFILE64_SOURCE` feature test macro.

This API provides functions capable of handling 64-bit **file sizes** and **offsets**.

> These functions have the same names as their 32-bit counterparts, but have the suffix 64 appended to the function name.  
> Among these functions are `fopen64()`, `open64()`, `lseek64()`, `truncate64()`, `stat64()`, `mmap64()`, and `setrlimit64()`.

In addition to the aforementioned functions, the transitional LFS API adds some new data types, including:

- **struct stat64**: an analog of the `stat` structure allowing for large file sizes.

- **off64_t:** a 64-bit type for representing file offsets.

### The `_FILE_OFFSET_BITS` macro

The recommended method of obtaining LFS functionality is to define the macro `_FILE_OFFSET_BITS` with the value `64` when compiling a program.

This automatically converts all of the relevant 32-bit functions and data types into their 64-bit counterparts.

### Passing `off_t` values to `printf()`

One problem that the LFS extensions don’t solve for us is how to pass `off_t` values to `printf()` calls.

> In n Section 3.6.2, we noted that the portable method of displaying values of one of the predefined system data types (e.g., **pid_t** or **uid_t**) was to cast that value to `long`, and use the `%ld` `printf()` specifier.

However, if we are employing the LFS extensions, then this is often not sufficient for the `off_t` data type.

Therefore, to display a value of type `off_t`, we cast it to long long and use the `%lld` `printf()` specifier.

---

## The `/dev/fd` Directory

For each **process**, the kernel provides the special virtual directory `/dev/fd`.
This directory contains filenames of the form `/dev/fd/n`, where `n` is a number corresponding to one of the *open file descriptors* for the process.

---

## Creating Temporary Files

Some programs need to create temporary files that are used only while the program is running, and these files should be removed when the program terminates.

The `mkstemp()` function generates a **unique** filename based on a `template` supplied by the caller and opens the file, returning a *file descriptor* that can be used with I/O system calls.

Typically, a temporary file is **unlinked** (deleted) soon after it is opened, using the `unlink()` system call.

The `tmpfile()` function creates a uniquely named temporary file that is opened for reading and writing.

---

## END
