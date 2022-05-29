# Notes From ***FILE I/O: THE UNIVERSAL I/O MODEL***

- [Notes From ***FILE I/O: THE UNIVERSAL I/O MODEL***](#notes-from-file-io-the-universal-io-model)
  - [Overview](#overview)
  - [Universality of I/O](#universality-of-io)
  - [Opening a File: `open()`](#opening-a-file-open)
    - [The `open()` flags Argument](#the-open-flags-argument)
    - [Errors from `open()`](#errors-from-open)
    - [The `creat()` System Call](#the-creat-system-call)
  - [Reading from a File: `read()`](#reading-from-a-file-read)
  - [Writing to a File: `write()`](#writing-to-a-file-write)
  - [Closing a File: `close()`](#closing-a-file-close)
  - [Changing the File Offset: `lseek()`](#changing-the-file-offset-lseek)
    - [File holes](#file-holes)
  - [Operations Outside the Universal I/O Model: `ioctl()`](#operations-outside-the-universal-io-model-ioctl)
  - [END](#end)

## Overview

All system calls for performing I/O refer to open files using a **file descriptor**, a (usually small) *nonnegative* **integer**.

File descriptors are used to refer to *all types* of open files.

> including pipes, FIFOs, sockets, terminals, devices, and regular files. Each process has its own set of file descriptors.

By convention, most programs expect to be able to use the three standard file descriptors.

These three descriptors are opened on the program’sbehalf by the shell, before the program is started.

> Or, more precisely, the program inherits copies of the shell’s file descriptors, and the shell normally operates with these three file descriptors always open.

| FD        | Posix name    |
| --------- | ------------- |
| 0 -stdin  | STDIN_FILENO  |
| 1 -stdout | STDOUT_FILENO |
| 3 -stderr | STDERR_FILENO |

The four key system calls for performing file I/O:

1. `fd = open(pathname, flags, mode);`
    > opens the file identified by *pathname*, returning a file descriptor used to refer to the open file in subsequent calls.  
    > The *mode* argument specifies the permissions to be placed on the file if it is **created** by this call.

2. `numread = read(fd, buffer, count);`
    > reads at most *count* bytes from the open file referred to by *fd* and stores them in *buffer*.

3. `numwritten = write(fd, buffer, count)`
    > writes up to *count* bytes from *buffer* to the open file referred to by *fd*.

4. `status = close(fd)`
    > called after all I/O has been completed, in order to release the file descriptor *fd* and its associated kernel resources.

---

## Universality of I/O

**Universality of I/O**: means that the same four system calls , `open()`, `read()`, `write()`, and `close()`,are used to perform I/O on **all** types of *files*.

---

## Opening a File: `open()`

**NOTE**: Checkout manual page of `open(2)`.

SUSv3 specifies that if open() succeeds, it is guaranteed to use the **lowest-numbered** **unused** file descriptor for the process.

### The `open()` flags Argument

**NOTE**: Checkout manual page of `open(2)` or pages 74-77 for definition of each flag.

### Errors from `open()`

**NOTE**: Checkout manual page of `open(2)` or pages 77-78 for definition of each error.

### The `creat()` System Call

In **early** UNIX implementations, `open()` had only **two** arguments and could not be used to *create* a new file. Instead, the `creat()` system call was used to create and open a new file.

Because the `open()` flags argument provides **greater** control over how the file is opened creat() is now **obsolete**

---

## Reading from a File: `read()`

**NOTE**: Checkout manual page of `read(2)`.

`read()` doesn’t place a terminating null byte at the end of the string.

> since `read()` can be used to read any sequence of bytes from a file.  
> In some cases, this input might be text, but in other cases, the input might be binary integers or C structures in binary form.

If a terminating null byte is **required** at the end of the input buffer, we must put it there **explicitly**.

> Because the terminating null byte requires a byte of memory, the size of buffer must be at least one greater than the largest string we expect to read.

---

## Writing to a File: `write()`

**NOTE**: Checkout manual page of `write(2)`.

---

## Closing a File: `close()`

**NOTE**: Checkout manual page of `close(2)`.

> especially note section of close. **VERY IMPORTANT**. I messed up big time for not reading this part. :smile:

File descriptors are a **consumable** resource, so *failure* to close a file descriptor could result in a process *running out* of descriptors.

---

## Changing the File Offset: `lseek()`

For each open file, the kernel records a **file offset**, sometimes also called the *read- write offset* or *pointer*.

This is the location in the file at which the next `read()` or `write()` will commence.

The file offset is expressed as an ordinal byte position **relative** to the *start* of the file.

The `lseek()` system call adjusts the file offset of the open file.

1. **SEEK_SET**: The file offset is set offset bytes from the **beginning** of the file.

2. **SEEK_CUR**: The file offset is adjusted by offset bytes relative to the **current** file offset.

3. **SEEK_END**: The file offset is set to the **size** of the file **plus offset**.
    > In other words, offset is interpreted with respect to the next byte after the last byte of the file.

### File holes

**Question**: What happens if a program seeks past the *end-of-file*, and then performs I/O?

A call to `read()` will return 0, indicating end-of-file.

Somewhat surprisingly, it is **possible** to *write* bytes at an arbitrary point past the end of the file.

**File hole**: is the space in between the previous end of the file and the newly written bytes.

File holes **do not**, however, take up any *disk space*

The main **advantage** of file holes is that a sparsely populated file consumes **less** disk space than would otherwise be required if the null bytes actually needed to be allocated in disk blocks.

> Core dump files are common examples of files that contain large holes.

---

## Operations Outside the Universal I/O Model: `ioctl()`

The `ioctl()` system call is a general-purpose mechanism for performing file and device operations that fall outside the universal I/O model

---

## END
