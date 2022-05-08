# Notes From ***MONITORING CHILD PROCESSES***

- [Notes From ***MONITORING CHILD PROCESSES***](#notes-from-monitoring-child-processes)
  - [Waiting on a Child Process](#waiting-on-a-child-process)
    - [The `wait()` System Call](#the-wait-system-call)
    - [The `waitpid()` System Call](#the-waitpid-system-call)
    - [The Wait Status Value](#the-wait-status-value)
    - [Process Termination from a Signal Handler](#process-termination-from-a-signal-handler)
    - [The `waitid()` System Call](#the-waitid-system-call)
    - [The `wait3()` and `wait4()` System Calls](#the-wait3-and-wait4-system-calls)
  - [Orphans and Zombies](#orphans-and-zombies)

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

The status value returned by `wait()` and `waitpid()` allows us to distinguish the following events for the child:

- The child terminated by calling `_exit()` (or `exit()`), specifying an integer **exit status**.

- The child was terminated by the delivery of an **unhandled signal**.

- The child was stopped by a signal, and `waitpid()` was called with the `WUNTRACED` flag.

- The child was resumed by a `SIGCONT` signal, and `waitpid()` was called with the `WCONTINUED` flag.

> (In the shell, we can obtain the termination status of the last command executed by examining the contents of the variable `$?`)

The `<sys/wait.h>` header file defines a standard set of *macros* that can be used to dissect a wait status value.

> When applied to a status value returned by `wait()` or ``waitpid(), only one of the macros in the list below will return true.

check page *546* for list of macros.

### Process Termination from a Signal Handler

In some circumstances, we may wish to have certain cleanup steps performed **before** a *process terminates*.  
> For this purpose, we can arrange to have a handler catch such signals, perform the cleanup steps, and then terminate the process.

If the child needs to inform the parent that it terminated because of a **signal**, then the child’s signal handler should first **disestablish** itself, and then **raise** the same signal once more, which this time will terminate the process.

### The `waitid()` System Call

Like `waitpid()`, `waitid()` returns the status of child processes. However, `waitid()` provides extra functionality that is unavailable with `waitpid()`.

```c
int waitid(idtype_t idtype , id_t id , siginfo_t * infop , int options );
```

The most significant difference between `waitpid()` and `waitid()` is that `waitid()` provides **more precise control** of the child events that should be waited for.

> We control this by ORing one or more of the flags in *options*

check pages *550-551* for full list of options.

### The `wait3()` and `wait4()` System Calls

The `wait3()` and `wait4()` system calls perform a similar task to `waitpid()`.

The principal semantic difference is that `wait3()` and `wait4()` return **resource usage** information about the terminated child in the structure pointed to by the *rusage* argument.

This information includes the amount of **CPU time** used by the process and **memory-management** statistics.

```c
pid_t wait3(int * status , int options , struct rusage * rusage );
pid_t wait4(pid_t pid , int * status , int options , struct rusage * rusage );
```

`wait3()` waits for any child, while `wait4()` can be used to select a specific child or children upon which to wait.

---

## Orphans and Zombies
