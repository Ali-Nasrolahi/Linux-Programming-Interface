# Notes From ***PROCESS CREATION AND PROGRAM EXECUTION IN MORE DETAIL***

- [Notes From ***PROCESS CREATION AND PROGRAM EXECUTION IN MORE DETAIL***](#notes-from-process-creation-and-program-execution-in-more-detail)
  - [Process Accounting](#process-accounting)
    - [Enabling and disabling process accounting](#enabling-and-disabling-process-accounting)
    - [Process accounting records](#process-accounting-records)
    - [Process accounting Version 3 file format](#process-accounting-version-3-file-format)
  - [The `clone()` System Call](#the-clone-system-call)
    - [The `clone()` flags Argument](#the-clone-flags-argument)
    - [Extensions to `waitpid()` for Cloned Children](#extensions-to-waitpid-for-cloned-children)
  - [Speed of Process Creation](#speed-of-process-creation)
  - [Effect of `exec()` and `fork()` on Process Attributes](#effect-of-exec-and-fork-on-process-attributes)
  - [END](#end)

## Process Accounting

When **process accounting** is enabled, the kernel writes an *accounting record* to the system-wide *process accounting* file as each process terminates.

This accounting record contains various *information* maintained by the kernel **about the process**, including its **termination status** and how much **CPU time** it consumed.

Historically, the primary use of process accounting was to charge users for consumption of system resources on multiuser UNIX systems.

However, process accounting can also be useful for **obtaining information** about a process that was not otherwise monitored and reported on by **its parent**.

### Enabling and disabling process accounting

The `acct()` system call is used by a **privileged** process to enable and disable process accounting.

> This system call is rarely used in application programs.  
> Normally, process accounting is enabled at each system restart by placing appropriate commands in the system boot scripts.

```c
int acct(const char * acctfile );
```

A typical pathname for the accounting file is `/var/log/pacct` or `/usr/account/pacct`.

### Process accounting records

Once process accounting is enabled, an `acct` record is written to the accounting file as each process terminates. The `acct` structure is defined in `<sys/acct.h>`.

check pages *593-594* for `acct` structure.

### Process accounting Version 3 file format

Starting with kernel 2.6.8, Linux introduced an optional alternative version of the process accounting file that addresses some limitations of the traditional accounting file.

To use this alternative version, known as **Version 3**, the `CONFIG_BSD_PROCESS_ACCT_V3` kernel configuration option must be enabled before building the kernel.

check pages *598* for **version 3** `acct` structure.

---

## The `clone()` System Call

Like `fork()` and `vfork()`, the Linux-specific `clone()` system call creates a new process. It differs from the other two calls in allowing **finer control** over the steps that occur during process creation.

The main use of `clone()` is in the implementation of **threading** libraries.

```c
int clone(int (* func ) (void *), void * child_stack , int flags , void * func_arg , ...
        /* pid_t * ptid , struct user_desc * tls , pid_t * ctid */ );
```

Like `fork()`, a new process created with `clone()` is an **almost exact** duplicate of the parent.

Unlike `fork()`, the cloned child **doesn’t continue** from the point of the call, but instead commences by calling the function specified in the *func* argument; we’ll refer to this as the *child function*.

The `clone()` flags argument serves two purposes.

1. First, its lower byte specifies the child’s **termination signal**, which is the signal to be sent to the parent when the child terminates.

2. The remaining bytes of the flags argument hold a bit mask that controls the operation of `clone()`.

check page *600* for full list of bitmasks.

### The `clone()` flags Argument

The `clone()` flags argument is a combination (ORing) of the bit-mask values described in pages **603-609**.

**Kernel scheduling entity** (KSE): is used in some texts to refer to the objects that are dealt with by the **kernel scheduler**. 

Really, *threads* and *processes* are simply *KSEs* that provide for greater and lesser **degrees of sharing** of attributes with other *KSEs*.

> such attributes are (virtual memory, open file descriptors, signal dispositions, process ID, and so on)


### Extensions to `waitpid()` for Cloned Children

To wait for children produced by `clone()`, the additional flags explained in pages *609-610* values can be included in the *options* bit-mask argument for `waitpid()`, `wait3()`, and `wait4()`.

---

## Speed of Process Creation

check pages *610-612* for full comparison.

---

## Effect of `exec()` and `fork()` on Process Attributes

check pages *612-615* for table of inheritances of process' attributes by `fork()` and `exec()`.

---

## END
