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
  - [The `SIGCHLD` Signal](#the-sigchld-signal)
    - [Establishing a Handler for `SIGCHLD`](#establishing-a-handler-for-sigchld)
      - [Design issues for `SIGCHLD` handlers](#design-issues-for-sigchld-handlers)
    - [Delivery of `SIGCHLD` for Stopped Children](#delivery-of-sigchld-for-stopped-children)
    - [Ignoring Dead Child Processes](#ignoring-dead-child-processes)
      - [Deviations from SUSv3 in older Linux kernels](#deviations-from-susv3-in-older-linux-kernels)
      - [The `sigaction()` `SA_NOCLDWAIT` flag](#the-sigaction-sa_nocldwait-flag)
      - [The System V SIGCLD signal](#the-system-v-sigcld-signal)
  - [END](#end)

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

The lifetimes of parent and child processes are usually *not the same*.
> either the parent outlives the child or vice versa.

This raises two questions:

1. Who becomes the parent of an **orphaned** child?
     - The orphaned child is adopted by `init`, the ancestor of all processes, whose process ID is `1`.
     > In other words, after a child’s parent terminates, a call to `getppid()` will return the value 1.
2. What happens to a child that terminates **before** its parent has had a chance to perform a `wait()`?
    > The point here is that, although the child has finished its work, the parent should still be permitted to perform a `wait()` at some later time to determine how the child terminated.
    - The kernel deals with this situation by turning the child into a **zombie**. This means that **most of the resources** held by the child are released back to the system to be reused by other processes.
    >The only part of the process that remains is an entry in the kernel’s process table recording the child’s PID, termination status, and resource usage statistics.

When the parent does perform a `wait()`, the kernel **removes** the *zombie*, since the last remaining information about the child is no longer required.

On the other hand, if the parent terminates **without** doing a `wait()`, then the `init` process adopts the child and **automatically** performs a `wait()`, thus removing the zombie process from the system.

If a large number of **zombie children** are created, they will eventually fill the *kernel process table*, preventing the creation of new processes.

Since the zombies can’t be killed by a signal, the only way to remove them from the system is to **kill their parent**, at which time the zombies are adopted and waited on by `init`, and consequently removed from the system.

---

## The `SIGCHLD` Signal

The termination of a child process is an event that occurs **asynchronously**.
> A parent can’t predict when one of its child will terminate.

We have already seen that the parent should use `wait()` in order to prevent the accumulation of *zombie* children, and have looked at two ways in which this can be done:

- The parent can call `wait()`, without specifying the `WNOHANG` flag, in which case the call will **block** if a child has not already terminated.

- The parent can periodically perform a **nonblocking** check (a poll) for dead children via a call to `waitpid()` specifying the `WNOHANG` flag.

Both of these approaches can be **inconvenient**.

### Establishing a Handler for `SIGCHLD`

The `SIGCHLD` signal is sent to a parent process whenever one of its children **terminates**.
> By default, this signal is **ignored**, but we can catch it by installing a signal handler.

Within the signal handler, we can use `wait()` to reap the *zombie child*.

if a second and third child terminate in quick succession while a `SIGCHLD` handler is executing for an already terminated child, then, although `SIGCHLD` is generated twice, it is **queued only once** to the parent.

> As a result, if the parent’s `SIGCHLD` handler called `wait()` only once each time it was invoked, the handler might fail to reap some zombie children.

The solution is to loop inside the `SIGCHLD` handler, **repeatedly** calling `waitpid()` with the `WNOHANG` flag until there are no more dead children to be reaped.

> body of a `SIGCHLD` handler simply consists of the following code, which reaps any dead children without checking their status:

```c
while (waitpid(-1, NULL, WNOHANG) > 0)
  continue;
```

#### Design issues for `SIGCHLD` handlers

Suppose that, at the time we establish a handler for `SIGCHLD` , there is already a terminated child for this process.

- Does the kernel then immediately generate a `SIGCHLD` signal for the parent?
  > SUSv3 leaves this point unspecified.
  - Some System V–derived implementations do generate a `SIGCHLD` in these circumstances; other implementations, including Linux, do not.

A portable application can make this difference invisible by establishing the `SIGCHLD` handler **before creating any children**.

we noted that using a system call from within a signal handler **may change** the value of the global variable `errno`.

For this reason, it is sometimes necessary to code a `SIGCHLD` handler to **save** `errno` in a local variable on entry to the handler, and then **restore** the `errno` value just prior to returning.

### Delivery of `SIGCHLD` for Stopped Children

Just as `waitpid()` can be used to monitor stopped children, so is it possible for a parent process to receive the `SIGCHLD` signal when one of its children is **stopped** by a signal.

This behavior is controlled by the `SA_NOCLDSTOP` flag when using `sigaction()` to establish a handler for the `SIGCHLD` signal.

If this flag is **omitted**, a `SIGCHLD` signal is **delivered** to the parent when one of its children stops; if the flag is present, `SIGCHLD` is **not delivered** for stopped children

### Ignoring Dead Child Processes

Explicitly setting the disposition of `SIGCHLD` to `SIG_IGN` causes any child process that subsequently terminates to be **immediately removed** from the system instead of being converted into a **zombie**.

In this case, since the status of the child process is simply discarded, a subsequent call to `wait()` **can’t return** any information for the terminated child.

#### Deviations from SUSv3 in older Linux kernels

SUSv3 specifies that if the disposition of `SIGCHLD` is set to `SIG_IGN`, the resource usage information for the child should be **discarded** and not included in the totals
returned when the parent makes a call to `getrusage()` specifying the `RUSAGE_CHILDREN` flag.

However, on Linux versions before kernel 2.6.9, the CPU times and resources used by the child are recorded and are visible in calls to `getrusage()`.

#### The `sigaction()` `SA_NOCLDWAIT` flag

SUSv3 specifies the `SA_NOCLDWAIT` flag, which can be used when setting the disposition of the `SIGCHLD` signal using `sigaction()`. This flag produces behavior similar to
that when the disposition of `SIGCHLD` is set to `SIG_IG`.

he principal difference between setting the disposition of `SIGCHLD` to `SIG_IGN` and employing `SA_NOCLDWAIT` is that, when establishing a handler with `SA_NOCLDWAIT`, SUSv3 leaves it unspecified whether or not a `SIGCHLD` signal is sent to the parent
when a child terminates.

In some UNIX implementations, including Linux, the kernel **does generate** a `SIGCHLD` signal for the parent process. On other UNIX implementations, `SIGCHLD` is not generated.

#### The System V SIGCLD signal

On Linux, the name `SIGCLD` is provided as a synonym for the `SIGCHLD` signal.

The key difference between BSD `SIGCHLD` and System V `SIGCLD` lies in what happens when the disposition of the signal was set to `SIG_IGN`:

- On historical BSD implementations, the system **continues** to generate zombies for unwaited-for children, even when `SIGCHLD` is ignored.

- On System V, using `signal()` (but not `sigaction()`) to ignore `SIGCLD` has the result
that zombies are **not generated** when children died.

---

## END
