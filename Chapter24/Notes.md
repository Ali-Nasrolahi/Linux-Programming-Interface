# Notes From ***PROCESS CREATION***

- [Notes From ***PROCESS CREATION***](#notes-from-process-creation)
  - [Overview of `fork()`, `exit()`, `wait()`, and `execve()`](#overview-of-fork-exit-wait-and-execve)
  - [Creating a New Process: `fork()`](#creating-a-new-process-fork)
    - [File Sharing Between Parent and Child](#file-sharing-between-parent-and-child)
    - [Memory Semantics of `fork()`](#memory-semantics-of-fork)
    - [Controlling a process’s memory footprint](#controlling-a-processs-memory-footprint)

## Overview of `fork()`, `exit()`, `wait()`, and `execve()`

we provide an overview of these four system calls and how they are typically used together.

- The `fork()` system call allows one process, the parent, to create a new process, the child. This is done by making the new child process an (almost) exact duplicate of the parent.
    > he child obtains copies of the parent’s stack, data, heap, and text segments.

- The `exit(status)` library function terminates a process, making all resources
(memory, open file descriptors, and so on) used by the process available for
subsequent reallocation by the kernel.
   > The **status** argument is an integer that determines the termination status for the process. Using the `wait()` system call, the parent can retrieve this status.
- The `wait(&status)` system call has two purposes:
  - First, if a child of this process has not yet terminated by calling `exit()`, then `wait()` **suspends** execution of the process until one of its children has terminated.
  - Second, the *termination status* of the child is returned in the status argument of`wait()`.

- The `execve(pathname, argv, envp)` system call loads a new program (`pathname`, with argument list `argv`, and environment list `envp`) into a process’s memory.

Some other operating systems combine the functionality of `fork()` and `exec()` into a single operation—a so-called **spawn**—that *creates a new process* that then *executes a specified program*.

---

## Creating a New Process: `fork()`

The `fork()` system call creates a new process, the **child**, which is an almost exact duplicate of the calling process, the **parent**.

```c
pid_t fork(void);
```

The key point to understanding `fork()` is to realize that after it has completed its work, two processes exist, and, in each process, execution *continues* from **the point where `fork()` returns**.

> The two processes are executing the same program text, but they have separate copies of the stack, data, and heap segments. The child’s stack, data, and heap segments are initially exact duplicates of the corresponding parts the parent’s memory.  After the `fork()`.

Within the code of a program, we can distinguish the two processes via the value returned from `fork()`.

- For the parent, `fork()` returns the **process ID** of the newly created child.

- For the child, `fork()` **returns 0**.
    > If necessary, the child can obtain its own process ID using `getpid()`, and the process ID of its parent using `getppid()`.

### File Sharing Between Parent and Child

When a `fork()` is performed, the child receives duplicates of all of the parent’s file descriptors.

Sharing of open file attributes between the parent and child processes is frequently useful.
> For example, if the parent and child are both writing to a file, sharing the file offset ensures that the two processes don’t overwrite each other’s output.  
> It does not, however, prevent the output of the two processes from being randomly intermingled. If this is not desired, then some form of process **synchronization** is required.

### Memory Semantics of `fork()`

Conceptually, we can consider `fork()` as creating copies of the parent’s *text*, *data*, *heap*, and *stack* segments.

However, actually performing a simple copy of the parent’s virtual memory pages into the new child process would be wasteful for a number of reasons.

> one being that a fork() is often followed by an immediate `exec()`, which replaces the process’s text with a new program.

Most modern UNIX implementations, including Linux, use two techniques to avoid such wasteful copying:

- The kernel marks the *text* segment of each process as **read-only**, so that a process can’t modify its own code.

- For the pages in the data, heap, and stack segments of the parent process, the kernel employs a technique known as **copy-on-write**.

> Think of it like this: read-only at first, however when any changes is needed on these segments then a copy of memroy is taken and changes apply to that copy.  
> In other words, **copy only when write is needed**.

### Controlling a process’s memory footprint

We can combine the use of `fork()` and `wait()` to control the memory footprint of a process.
