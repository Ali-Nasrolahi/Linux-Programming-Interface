# Notes From ***SIGNALS: FUNDAMENTAL CONCEPTS***

- [Notes From ***SIGNALS: FUNDAMENTAL CONCEPTS***](#notes-from-signals-fundamental-concepts)
  - [Concepts and Overview](#concepts-and-overview)

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
