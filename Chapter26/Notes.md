# Notes From ***MONITORING CHILD PROCESSES***

- [Notes From ***MONITORING CHILD PROCESSES***](#notes-from-monitoring-child-processes)
  - [Waiting on a Child Process](#waiting-on-a-child-process)
    - [The `wait()` System Call](#the-wait-system-call)
    - [The `waitpid()` System Call](#the-waitpid-system-call)
    - [The Wait Status Value](#the-wait-status-value)

## Waiting on a Child Process

In many applications where a parent creates child processes, it is useful for the parent to be able to monitor the children to find out when and how they terminate.

This facility is provided by `wait()` and a number of related system calls.

### The `wait()` System Call

The `wait()` system call waits for one of the children of the calling process to **terminate** and returns the termination status of that child in the buffer pointed to by *status*.

```c
pid_t wait(int * status );
```

### The `waitpid()` System Call

The `wait()` system call has a number of limitations, which `waitpid()` was designed to address:

- If a parent process has created multiple children, it is not possible to `wait()` for the completion of a **specific child**; we can only wait for the next child that terminates.

- If no child has yet terminated, `wait()` **always blocks**. Sometimes, it would be preferable to perform a nonblocking wait so that if no child has yet terminated, we obtain an immediate indication of this fact.

- Using `wait(),` we can find out only about children that have **terminated**. It is not possible to be notified when a child is stopped by a **signal**

```c
pid_t waitpid(pid_t pid , int * status , int options );
```

### The Wait Status Value
