# Notes From ***PROCESS TERMINATION***

- [Notes From ***PROCESS TERMINATION***](#notes-from-process-termination)
  - [Terminating a Process: `_exit()` and `exit()`](#terminating-a-process-_exit-and-exit)
  - [Details of Process Termination](#details-of-process-termination)
  - [Exit Handlers](#exit-handlers)
    - [Registering exit handlers](#registering-exit-handlers)
  - [Interactions Between `fork()`, `stdio` Buffers, and `_exit()`](#interactions-between-fork-stdio-buffers-and-_exit)
  - [END](#end)

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

During both normal and abnormal termination of a process, the following actions
occur:

- Open file descriptors, directory streams, message catalog descriptors and conversion descriptors are closed.

- As a consequence of closing file descriptors, any file locks held by this process are released.

- Any attached System V shared memory segments are detached, and the `shm_nattch` counter corresponding to each segment is decremented by one.

- For each System V semaphore for which a `semadj` value has been set by the process, that `semadj` value is added to the semaphore value.

- If this is the controlling process for a controlling terminal, then the `SIGHUP` signal is sent to each process in the controlling terminal’s foreground process
group, and the terminal is disassociated from the session.

- Any POSIX named semaphores that are open in the calling process are closed as though `sem_close()` were called.

- Any POSIX message queues that are open in the calling process are closed as though `mq_close()` were called.

- If, as a consequence of this process exiting, a process group becomes orphaned and there are any stopped processes in that group, then all processes in the
group are sent a `SIGHUP` signal followed by a `SIGCONT` signal.

- Any memory locks established by this process using `mlock()` or`mlockall()`.

- Any memory mappings established by this process using `mmap()` are unmapped.

---

## Exit Handlers

An exit handler is a programmer-supplied function that is registered at some point during the life of the process and is then automatically called during normal process termination via `exit()`.
> Exit handlers are not called if a program calls`_exit()` directly or if the process is terminated abnormally by a signal.

### Registering exit handlers

The GNU C library provides two ways of registering exit handlers. The first method, specified in SUSv3, is to use the `atexit()` function.

```c
int atexit(void (* func )(void));
```

When the program invokes exit(), these functions are called in **reverse order** of registration.

> This ordering is logical because, typically, functions that are registered earlier are those that carry out more fundamental types of cleanups that may need to be performed after later-registered functions.

However, if one of the exit handlers fails to return then the remaining exit handlers **are not called**.

SUSv3 requires that an implementation allow a process to be able to register at least **32 exit handlers**.
> Using the call `sysconf(_SC_ATEXIT_MAX)`, a program can determine the implementation-defined upper limit on the number of exit handlers that can be registered.

A child process created via `fork()` inherits a copy of its parent’s exit handler registrations.

Exit handlers registered with `atexit()` suffer a couple of limitations.

- The first is that when called, an exit handler doesn’t know what status was passed to`exit()`.

- The second limitation is that we can’t specify an argument to the exit handler when it is called.

To address these limitations, glibc provides a (nonstandard) alternative method of registering exit handlers: `on_exit()`.

```c
int on_exit(void (* func )(int, void *), void * arg );
```

Functions registered using `atexit()` and `on_exit()` are placed on the same list. If both methods are used in the same program, then the exit handlers are called in reverse
order of their registration using the two methods.

---

## Interactions Between `fork()`, `stdio` Buffers, and `_exit()`

We can prevent duplicated `stdio` output ,by forked child ,from occurring in one of the following ways:

- As a specific solution to the stdio buffering issue, we can use `fflush()` to flush the `stdio` buffer prior to a `fork()` call. Alternatively, we could use `setvbuf()` or `setbuf()` to **disable buffering** on the `stdio` stream.

- Instead of calling `exit()`, the child can call `_exit()`, so that it doesn’t flush `stdio` buffers.

---

## END
