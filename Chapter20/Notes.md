# Notes From ***SIGNALS: FUNDAMENTAL CONCEPTS***

- [Notes From ***SIGNALS: FUNDAMENTAL CONCEPTS***](#notes-from-signals-fundamental-concepts)
  - [Concepts and Overview](#concepts-and-overview)
  - [Signal Types and Default Actions](#signal-types-and-default-actions)
  - [Changing Signal Dispositions: `signal()`](#changing-signal-dispositions-signal)
  - [Introduction to Signal Handlers](#introduction-to-signal-handlers)
  - [Sending Signals: `kill()`](#sending-signals-kill)
  - [Checking for the Existence of a Process](#checking-for-the-existence-of-a-process)
  - [Other Ways of Sending Signals: `raise()` and `killpg()`](#other-ways-of-sending-signals-raise-and-killpg)
  - [Displaying Signal Descriptions](#displaying-signal-descriptions)
  - [Signal Sets](#signal-sets)

## Concepts and Overview

A **signal** is a notification to a process that an event has occurred. Signals are sometimes described as software interrupts.

Each signal is defined as a unique integer, starting sequentially from 1.
These integers are defined in <*signal.h*> with symbolic names of the form **SIGxxxx**.

Signals fall into two broad categories:

- **Traditional** or **Standard** signals, which are used by the kernel to notify processes of events.
- The **realtime** signals.

A signal is said to be **generated** by some event. Once generated, a signal is later **delivered** to a process.  
Between the time it is generated and the time it is delivered, a signal is said to be **pending**.

We can add a signal to the process’s **signal mask**—a set of signals whose delivery is currently **blocked**.
> If a signal is generated while it is blocked, it remains pending until it is later unblocked

Upon delivery of a signal, a process carries out one of the following default actions, depending on the signal:

- The signal is **ignored**.
    > i.e., it is discarded by the kernel and has no effect on the process.
- The process is **terminated** (killed).
- A core dump file is generated, and the process is terminated.
    > A core dump file contains an image of the virtual memory of the process, which can be loaded into a debugger in order to inspect the state of the process at the time that it terminated.
- The process is **stopped**—execution of the process is *suspended*.
- Execution of the process is **resumed** after previously being *stopped*.

**Disposition** of the signal means changing default action of a signal.

A program can set one of the following dispositions for a signal:

- The **default** action should occur.
- The signal is **ignored**.
- A **signal handler** is executed.

A **signal handler** is a function, written by the programmer, that performs appropriate tasks in response to the delivery of a signal.


## Signal Types and Default Actions

check `signal(7)` manual for full list.

---

## Changing Signal Dispositions: `signal()` 

UNIX systems provide two ways of changing the disposition of a signal: `signal()` and `sigaction()`. 
> use `sigaction()` instead of`signal()`. 

```c
void ( *signal(int sig , void (* handler )(int)) ) (int);
```

---

## Introduction to Signal Handlers

A **signal handler** (also called a *signal catcher*) is a function that is called when a specified signal is delivered to a process.

---

## Sending Signals: `kill()` 

One process can send a signal to another process using the `kill()` system call.

```c
int kill(pid_t pid , int sig );
```

A process needs appropriate *permissions* to be able send a signal to another process. The permission rules are as follows:

- A privileged ( `CAP_KILL` ) process may send a signal to any process.

- The `init` process is a special case. It can be sent only signals for which it has a handler installed.

- An unprivileged process can send a signal to another process if the real or effective user ID of the sending process matches the real user ID or saved set-user-ID of the receiving process

- The `SIGCONT` signal is treated specially. An unprivileged process may send this signal to any other process in the same session, regardless of user ID checks.

---

## Checking for the Existence of a Process

The `kill()` system call can serve another purpose.

If the *sig* argument is specified as *0* (the so-called `null signal`), then no signal is sent. Instead, `kill()` merely performs error checking to see if the process can be signaled.

Various other techniques can also be used to check whether a particular process is running, including the following:

- The `wait()` system calls. 
  > They can be employed only if the monitored process is a child of the caller.
- **Semaphores** and exclusive **file locks**.
  > if we can acquire the semaphore or lock, we know the process has terminated.
- *IPC channels* such as **pipes** and **FIFOs**:
  > We set up the monitored process so that it holds a file descriptor open for writing on the channel as long as it is alive.  Meanwhile, the monitoring process holds open a read descriptor for the channel, and it knows that the monitored process has terminated when the write end of the channel is closed.
 - The `/proc/PID` interface:

---

## Other Ways of Sending Signals: `raise()` and `killpg()`

Sometimes, it is useful for a process to send a signal to **itself**.The `raise()` function performs this task.

```c
int raise(int sig);
```

The `killpg()` function sends a signal to all of the members of a **process group**.

```c
int killpg(pid_t pgrp , int sig );
```

---

## Displaying Signal Descriptions

Each signal has an associated printable description. These descriptions are listed in
the array `sys_siglist`. 
> e.g., `sys_siglist[SIGPIPE]` to get the description for`SIGPIPE`. 

However, rather than using the `sys_siglist` array directly, the `strsignal()` function is preferable.

```c
#define _BSD_SOURCE
#include <signal.h>

extern const char *const sys_siglist[];

#define _GNU_SOURCE
#include <string.h>

char *strsignal(int sig);
```

The `psignal()` function displays (on *standard error*) the string given in its argument msg, followed by a colon, and then the signal description corresponding to *sig*.

```c
void psignal(int sig ,const char* msg);
```

---

## Signal Sets