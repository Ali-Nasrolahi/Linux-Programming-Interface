# Notes From ***PROCESS CREATION AND PROGRAM EXECUTION IN MORE DETAIL***

- [Notes From ***PROCESS CREATION AND PROGRAM EXECUTION IN MORE DETAIL***](#notes-from-process-creation-and-program-execution-in-more-detail)
  - [Process Accounting](#process-accounting)
    - [Enabling and disabling process accounting](#enabling-and-disabling-process-accounting)
    - [Process accounting records](#process-accounting-records)
    - [Process accounting Version 3 file format](#process-accounting-version-3-file-format)
  - [The `clone()` System Call](#the-clone-system-call)

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
