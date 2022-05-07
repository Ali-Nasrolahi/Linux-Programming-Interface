# Notes From ***PROCESS CREATION***

- [Notes From ***PROCESS CREATION***](#notes-from-process-creation)
  - [Overview of `fork()`, `exit()`, `wait()`, and `execve()`](#overview-of-fork-exit-wait-and-execve)

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