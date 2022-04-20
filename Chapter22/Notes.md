# Notes From ***SIGNALS: ADVANCED FEATURES***

- [Notes From ***SIGNALS: ADVANCED FEATURES***](#notes-from-signals-advanced-features)
  - [Core Dump Files](#core-dump-files)
    - [Circumstances in which core dump files are not produced](#circumstances-in-which-core-dump-files-are-not-produced)
    - [Naming the core dump file: `/proc/sys/kernel/core_pattern`](#naming-the-core-dump-file-procsyskernelcore_pattern)
  - [Special Cases for Delivery, Disposition, and Handling](#special-cases-for-delivery-disposition-and-handling)
    - [`SIGKILL` and `SIGSTOP`](#sigkill-and-sigstop)
    - [`SIGCONT` and stop signals](#sigcont-and-stop-signals)
    - [Don’t change the disposition of ignored terminal-generated signals](#dont-change-the-disposition-of-ignored-terminal-generated-signals)
  - [Interruptible and Uninterruptible Process Sleep States](#interruptible-and-uninterruptible-process-sleep-states)
  - [Hardware-Generated Signals](#hardware-generated-signals)
  - [Synchronous and Asynchronous Signal Generation](#synchronous-and-asynchronous-signal-generation)
  - [Timing and Order of Signal Delivery](#timing-and-order-of-signal-delivery)
    - [When is a signal delivered?](#when-is-a-signal-delivered)
    - [Order of delivery of multiple unblocked signals](#order-of-delivery-of-multiple-unblocked-signals)
  - [Implementation and Portability of `signal()`](#implementation-and-portability-of-signal)
    - [Some *glibc* details](#some-glibc-details)
    - [`sigaction()` is the preferred API for establishing a signal handler](#sigaction-is-the-preferred-api-for-establishing-a-signal-handler)
  - [Realtime Signals](#realtime-signals)
    - [Limits on the number of queued realtime signals](#limits-on-the-number-of-queued-realtime-signals)
    - [Using realtime signals](#using-realtime-signals)
    - [Sending Realtime Signals](#sending-realtime-signals)
    - [Handling Realtime Signals](#handling-realtime-signals)
  - [Waiting for a Signal Using a Mask: `sigsuspend()`](#waiting-for-a-signal-using-a-mask-sigsuspend)
  - [Synchronously Waiting for a Signal](#synchronously-waiting-for-a-signal)
  - [Fetching Signals via a File Descriptor](#fetching-signals-via-a-file-descriptor)
  - [Interprocess Communication with Signals](#interprocess-communication-with-signals)
  - [Earlier Signal APIs (System V and BSD)](#earlier-signal-apis-system-v-and-bsd)
    - [The System V signal API](#the-system-v-signal-api)
    - [The BSD signal API](#the-bsd-signal-api)
  - [END](#end)

## Core Dump Files

Certain signals cause a process to create a *core dump* and terminate.

A **core dump** is a file containing a *memory image* of the process at the time it terminated.

> This memory image can be loaded into a debugger in order to examine the state of a program’s code and data at the moment when the signal arrived.

One way of causing a program to produce a core dump is to type the quit character (usually **Control-\\**), which causes the `SIGQUIT` signal to be generated.

The *core dump* file is created in the working directory of the process, with the name *core*.

### Circumstances in which core dump files are not produced

- The process doesn’t have permission to write the core dump file.

- A regular file with the same name **already** exists, and is writable, but there is **more** than one (hard) link to the file.

- The directory in which the core dump file is to be created doesn’t exist.

- The process resource limit on the size of a core dump file is set to 0.

- The process resource limit on the size of a file that may be produced by the process is set to 0.

- The binary executable file that the process is executing doesn’t have read permission enabled.

- The file system on which the current working directory resides is mounted read-only, is full, or has run out of i-nodes.

- Set-user-ID (set-group-ID) programs executed by a user other than the file owner (group owner) don’t generate core dumps.

Since kernel 2.6.23, the Linux-specific `/proc/ PID/coredump_filter` can be used on a per-process basis to determine which types of memory mappings are written to a core dump file.
> See the core(5) manual page for further details.

### Naming the core dump file: `/proc/sys/kernel/core_pattern`

The format string contained in the Linux-specific `/proc/sys/kernel/core_pattern` file controls the naming of all core dump files produced on the system.

A privileged user can define this file to include any of the format specifiers.

Check pages *449-450* for full format specifiers table.

Additionally, the string may include slashes **( / )**. In other words, we can control not just the name of the core file, but also the (absolute or relative) **directory** in which it is created.

> After all format specifiers have been replaced, the resulting pathname string is truncated to a maximum of 128 characters

If this file contains a string starting with the pipe symbol **( | )**, then the remaining characters in the file are interpreted as a **program** —with optional arguments that may include the *%* specifiers.

> that is to be executed when a process dumps core.

---

## Special Cases for Delivery, Disposition, and Handling

For certain signals, special rules apply regarding delivery, disposition, and handling,

### `SIGKILL` and `SIGSTOP`

It is not possible to change the default action for `SIGKILL` , which always *terminates* a process, and `SIGSTOP`, which always *stops* a process. Both `signal()` and `sigaction()` return an error on attempts to change the disposition of these signals.

### `SIGCONT` and stop signals

As noted earlier, the `SIGCONT` signal is used to continue a process previously stopped.

If a process is currently stopped, the arrival of a `SIGCONT` signal **always** causes the process to **resume**, even if the process is currently *blocking* or *ignoring* `SIGCONT`.

### Don’t change the disposition of ignored terminal-generated signals

If, at the time it was execed, a program finds that the disposition of a terminal- generated signals has been set to `SIG_IGN` (ignore), then generally the program **should not** attempt to change the disposition of the signal.

> his is not a rule enforced by the system, but rather a convention that should be followed when writing applications.

---

## Interruptible and Uninterruptible Process Sleep States

We need to add a proviso to our earlier statement that `SIGKILL` and `SIGSTOP` always act *immediately* on a process.

At various times, the kernel may put a process to sleep, and two sleep states are distinguished:

- `TASK_INTERRUPTIBLE`: The process is waiting for some event.

- `TASK_UNINTERRUPTIBLE`: The process is waiting on certain special classes of event, such as the completion of a disk I/O.

Because a process normally spends only very brief periods in the `TASK_UNINTERRUPTIBLE` state, the fact that a signal is delivered only when the process **leaves** this state is *invisible*.

> However, in rare circumstances, a process may remain hung in this state, perhaps as the result of a hardware failure, an NFS problem, or a kernel bug. (system should be restarted in this scenario.)

Linux adds a third state to address the hanging process problem just described:

- `TASK_KILLABLE`: This state is like `TASK_UNINTERRUPTIBLE`, but wakes the process if a fatal signal (i.e., one that would kill the process) is received.

---

## Hardware-Generated Signals

`SIGBUS`, `SIGFPE`, `SIGILL`, and `SIGSEGV` can be generated as a consequence of a **hardware exception** or, less usually, by being sent by`kill()`.

In the case of a hardware exception, SUSv3 specifies that the behavior of a process is undefined if it returns from a *handler* for the signal, or if it *ignores* or *blocks* the signal.

The reasons for this are as follows:

- *Returning from the signal handler*: Suppose that a machine-language instruction generates one of these signals, and a signal handler is consequently invoked. On normal return from the handler, the program attempts to **resume** execution at the point where it was interrupted.  
  > Results in infinite loop between signal and signal handler.

- *Ignoring the signal*: It makes little sense to ignore a hardware-generated signal,
as it is **unclear** how a program should *continue* execution.  
When one of these signals is generated as a consequence of a hardware exception, Linux **forces** its delivery, even if the program has requested that the signal be **ignored**.

- *Blocking the signal*: As with the previous case, it makes little sense to block a hardware-generated signal, as it is unclear how a program should then continue execution.

The correct way to deal with hardware-generated signals is either to accept their default action (process termination) or to write handlers that don’t perform a normal return.

---

## Synchronous and Asynchronous Signal Generation

We have already seen that a process generally can’t predict when it will receive a signal. We now need to qualify this observation by distinguishing between **synchronous** and **asynchronous** signal generation.

Generally we've been dealing with **asynchronous** signal generation in most of our scenarios.

However, in some cases, a signal is generated while the process itself is executing. We have already seen two examples of this:

- The hardware-generated signals.
- A process can use `raise()`, `kill()`, or `killpg()` to send a signal to itself.

In these cases, the generation of the signal is synchronous—the signal is delivered **immediately**.

In other words, the earlier statement about the **unpredictability** of the delivery of a signal **doesn't** apply.

---

## Timing and Order of Signal Delivery

### When is a signal delivered?

When a signal is generated asynchronously, there may be a (small) delay while the signal is pending between the time when it was generated and the time it is actually delivered, even if we have not blocked the signal.
> The reason for this is that the kernel delivers a pending signal to a process only at the next switch from kernel mode to user mode while executing that process.

This means signal delivered in these situations:

- When the process is rescheduled after it earlier timed out.

- At completion of a system call.

### Order of delivery of multiple unblocked signals

If a process has **multiple** pending signals that are unblocked using `sigprocmask()`, then all of these signals are **immediately** delivered to the process.

As currently implemented, the Linux kernel delivers the signals in **ascending** order.

> e.g. SIGINT (signal 2) and then SIGQUIT (signal 3).

---

## Implementation and Portability of `signal()`

Early implementations of signals were unreliable, meaning that:

- On entry to a signal handler, the disposition of the signal was reset to its default.
  > Another call to `signal()` must've been occured to register signal handler.
- Delivery of further occurrences of a signal was not blocked during execution of a signal handler.
  > i.e., singal handler could've been invoked recursively, which in numerous signal delivery results in *stack overflow*.

check page *455* for `signal()` implementation with `sigaction()`.

### Some *glibc* details

If we want to obtain unreliable signal semantics with modern versions of glibc, we can explicitly replace our calls to `signal()` with calls to the (nonstandard) `sysv_signal()` function.

```c
void ( *sysv_signal(int sig , void (* handler )(int)) ) (int);
```

### `sigaction()` is the preferred API for establishing a signal handler

Because of the System V versus BSD (and old versus recent glibc) portability issues described above, it is good practice always to use `sigaction()`, rather than `signal()`, to establish signal handlers.

---

## Realtime Signals

Realtime signals were defined in POSIX.1b to remedy a number of limitations of
standard signals.

Advantages over standard signals:

- Realtime signals provide an **increased range** of signals that can be used for application-defined purposes.

- Realtime signals are **queued**. If multiple instances of a realtime signal are sent to a process, then the signal is delivered multiple times.

- When sending a realtime signal, it is possible to **specify data** that accompanies the signal.

- The order of delivery of different realtime signals is **guaranteed**. If multiple different realtime signals are pending, then the lowest-numbered signal is delivered first.

The <*signal.h*> header file defines the constant `RTSIG_MAX` to indicate the number of available realtime signals, and the constants `SIGRTMIN` and `SIGRTMAX` to indicate the lowest and highest available realtime signal numbers.

### Limits on the number of queued realtime signals

SUSv3 allows an implementation to place an *upper limit* on the number of realtime signals (of all types) that may be queued to a process, and requires that this limit be at least`_POSIX_SIGQUEUE_MAX`

It can also make this information available through the following call:

```c
lim = sysconf(_SC_SIGQUEUE_MAX);
```

> On Linux, this call returns –1. The reason for this is that Linux employs a different model for limiting the number of realtime signals that may be queued to a process.

In Linux versions up to and including 2.6.7, this limit can be viewed and (with privilege) changed via the Linux-specific `/proc/sys/kernel/rtsig-max` file.

Starting with Linux 2.6.8, `/proc` interfaces were removed. Under the new model, the `RLIMIT_SIGPENDING` soft resource limit defines a limit on the number of signals that can be queued to all processes owned by a particular real user ID.

### Using realtime signals

In order for a pair of processes to send and receive realtime signals, SUSv3 requires the following:

- The sending process sends the signal plus its accompanying data using the `sigqueue()` system call.

- he receiving process establishes a handler for the signal using a call to `sigaction()` that specifies the `SA_SIGINFO` flag.

### Sending Realtime Signals

The `sigqueue()` system call sends the realtime signal specified by *sig* to the process specified by *pid*.

```c
int sigqueue(pid_t pid , int sig , const union sigval value );
```

### Handling Realtime Signals

We can handle realtime signals just like standard signals, using a normal (single- argument) signal handler.

Alternatively, we can handle a realtime signal using a three-argument signal handler established using the `SA_SIGINFO` flag.

---

## Waiting for a Signal Using a Mask: `sigsuspend()`

The `sigsuspend()` system call replaces the process signal mask by the signal set pointed to by *mask*, and then **suspends** execution of the process until a signal is
**caught** and its handler returns.

Once the handler returns, `sigsuspend()` restores the process signal mask to the value it had prior to the call.

```c
int sigsuspend(const sigset_t * mask );
```

Calling `sigsuspend()` is equivalent to **atomically** performing these operations:

```c
sigprocmask(SIG_SETMASK, &mask, &prevMask); /* Assign new mask */
pause();
sigprocmask(SIG_SETMASK, &prevMask, NULL); /* Restore old mask */
```

---

## Synchronously Waiting for a Signal

We can use the `sigwaitinfo()` system call to **synchronously** *accept* a signal.

```c
int sigwaitinfo(const sigset_t * set , siginfo_t * info );
```

The `sigtimedwait()` system call is a variation on `sigwaitinfo()`. The only difference is that `sigtimedwait()` allows us to specify a *time limit* for waiting.

```c
int sigtimedwait(const sigset_t * set , siginfo_t * info,
                 const struct timespec * timeout );
```

---

## Fetching Signals via a File Descriptor

`signalfd()` system call, which creates a special file descriptor from which signals directed to the caller can be read.

```c
int signalfd(int fd , const sigset_t * mask , int flags );
```

---

## Interprocess Communication with Signals

Disadvantages of singals for IPC purposes:

- The **asynchronous** nature of signals means that we face various problems, including reentrancy requirements, race conditions, and the correct handling of global variables from signal handlers.

- Standard signals are not queued. Even for realtime signals, there are upper limits on the number of signals that may be queued. This means that in order to avoid loss of information, the process receiving the signals must have a method of informing the sender that it is ready to receive another signal.

- A further problem is that signals carry only a limited amount of information.

---

## Earlier Signal APIs (System V and BSD)

### The System V signal API

As noted earlier, one important difference in the System V signal API is that when a handler is established with `signal()`, we get the older, **unreliable** signal semantics.

To establish a signal handler with **reliable** semantics, System V provided the `sigset()` call

```c
void (*sigset(int sig , void (* handler )(int)))(int);
```

- The `sighold()` function adds a signal to the process signal mask.
- The `sigrelse()` function removes a signal from the signal mask.
- The `sigignore()` function sets a signal’s disposition to ignore.
- The `sigpause()` function is similar to `sigsuspend(),` but removes just one signal from the process signal mask before suspending the process until the arrival of a signal.

```c
int sighold(int sig );
int sigrelse(int sig );
int sigignore(int sig );

int sigpause(int sig );
```

### The BSD signal API

The `sigvec()` function is analogous to`sigaction()`.

```c
int sigvec(int sig , struct sigvec * vec , struct sigvec * ovec );
```

- The `sigblock()` function adds a set of signals to the process signal mask. It is analogous to the `sigprocmask()` `SIG_BLOCK` operation.
  
- The `sigsetmask()` call specifies an absolute value for the signal mask. It is analogous to the `sigprocmask()` `SIG_SETMASK` operation.

- The `sigpause()` function is analogous to`sigsuspend()`.

```c
int sigblock(int mask );
int sigsetmask(int mask );

int sigpause(int sigmask );

int sigmask( sig );
```

---

## END
