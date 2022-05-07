# Notes From ***PROCESS TERMINATION***

- [Notes From ***PROCESS TERMINATION***](#notes-from-process-termination)
  - [Terminating a Process: `_exit()` and `exit()`](#terminating-a-process-_exit-and-exit)
  - [Details of Process Termination](#details-of-process-termination)

## Terminating a Process: `_exit()` and `exit()`

A process may terminate in two general ways.

1. One of these is **abnormal** termination, caused by the **delivery of a signal** whose default action is to terminate the process.

2. Alternatively, a process can terminate **normally**, using the `_exit()` system call.

```c
void _exit(int status );
```

Although defined as an int, only the **bottom 8 bits** of status are actually made available to the parent.
> By convention, a termination status of 0 indicates that a process completed successfully and a nonzero status value indicates that the process terminated unsuccessfully,


Programs generally don’t call `_exit()` directly, but instead call the `exit()` library function, which performs various actions before calling`_exit()`. 

```c
void exit(int status );
```

The following actions are performed by `exit()`:

- Exit handlers are called, in reverse order of their registration.
- The *stdio* stream buffers are flushed.
- The `_exit()` system call is invoked, using the value supplied in *status*.

---

## Details of Process Termination
