# Notes From ***THREADS: FURTHER DETAILS***

- [Notes From ***THREADS: FURTHER DETAILS***](#notes-from-threads-further-details)
  - [Thread Stacks](#thread-stacks)
  - [Threads and Signals](#threads-and-signals)
    - [How the UNIX Signal Model Maps to Threads](#how-the-unix-signal-model-maps-to-threads)
    - [Manipulating the Thread Signal Mask](#manipulating-the-thread-signal-mask)
    - [Sending a Signal to a Thread](#sending-a-signal-to-a-thread)
    - [Dealing with Asynchronous Signals Sanely](#dealing-with-asynchronous-signals-sanely)
    - [Threads and Process Control](#threads-and-process-control)
      - [Threads and `exec()`](#threads-and-exec)
      - [Threads and `fork()`](#threads-and-fork)
      - [Threads and `exit()`](#threads-and-exit)
  - [Thread Implementation Models](#thread-implementation-models)
    - [Many-to-one (M:1) implementations (user-level threads)](#many-to-one-m1-implementations-user-level-threads)
      - [Advantages](#advantages)
      - [Disadvantages of `M:1`](#disadvantages-of-m1)
    - [One-to-one (1:1) implementations (kernel-level threads)](#one-to-one-11-implementations-kernel-level-threads)
      - [Disadvantages of `1:1`](#disadvantages-of-11)
    - [Many-to-many (M:N) implementations (two-level model)](#many-to-many-mn-implementations-two-level-model)
  - [Linux Implementations of POSIX Threads](#linux-implementations-of-posix-threads)
    - [LinuxThreads](#linuxthreads)
      - [LinuxThreads deviations from specified behavior](#linuxthreads-deviations-from-specified-behavior)
      - [Other problems with LinuxThreads](#other-problems-with-linuxthreads)
    - [NPTL](#nptl)
    - [Which Threading Implementation?](#which-threading-implementation)
      - [Discovering the threading implementation](#discovering-the-threading-implementation)

## Thread Stacks

Each thread has its own stack whose size is **fixed** when the thread is created.

The related `pthread_attr_setstack()` function can be used to control both the *size* and the *location* of the stack.

---

## Threads and Signals

### How the UNIX Signal Model Maps to Threads

### Manipulating the Thread Signal Mask

Only difference between *process signal mask* and *thread* one is just being **thread-wide** or **process-wide**.
> check chapters 20-22 for full explaination of signals, including signal masks.

When a new thread is created, it inherits a copy of the signal mask of the thread that created it.

A thread can use `pthread_sigmask()` to change its signal mask, to retrieve the existing mask or both.

```c
int pthread_sigmask(int how , const sigset_t * set , sigset_t * oldset );
```

### Sending a Signal to a Thread

The `pthread_kill()` function sends the signal *sig* to another thread in the same process.

```c
int pthread_kill(pthread_t thread , int sig );
```

The Linux-specific `pthread_sigqueue()` function combines the functionality of `pthread_kill()` and `sigqueue()`:  
It sends a signal with accompanying data to another thread in the same process.

```c
int pthread_sigqueue(pthread_t thread , int sig , const union sigval value );
```

### Dealing with Asynchronous Signals Sanely

An approach to deal with Asynchronous Signals:

- All threads **block** all of the asynchronous signals that the process might receive

- Create a single dedicated thread that accepts incoming signals using `sigwaitinfo()`, `sigtimedwait()`, or`sigwait()`.

```c
int sigwait(const sigset_t * set , int * sig );
```

Check chapter 20-22 for more informations about signals.

### Threads and Process Control

#### Threads and `exec()`

When any thread calls one of the  `exec()` functions, the calling program is **completely** *replaced*.
> All threads, except the one that called  `exec()`, vanish immediately.

*None of* the threads executes *destructors* for thread-specific data or calls *cleanup handlers*.

After an  `exec()`, the *thread ID* of the remaining thread is **unspecified**.

#### Threads and `fork()`

When a multithreaded process calls  `fork()`, only the calling thread is replicated in the child process.

> All of the other threads vanish in the child; no thread-specific data destructors or cleanup handlers are executed for those threads.

The usual recommendation is that the only use of `fork()` in a multithreaded process should be one that is followed by an **immediate** `exec()`.

For programs that must use a `fork()` that is not followed by an `exec()`, the Pthreads API provides a mechanism for defining **fork handlers**.

Fork handlers are established using a `pthread_atfork()` call of the following form:

```c
pthread_atfork(prepare_func, parent_func, child_func);
```

#### Threads and `exit()`

If any thread calls `exit()` or, equivalently, the main thread does a **return** , all threads immediately vanish.
> no thread-specific data destructors or cleanup handlers are executed.

---

## Thread Implementation Models

The differences between these implementation models hinge on how threads are **mapped** onto **kernel scheduling entities** (*KSEs*). which are the units to which the kernel allocates the CPU and other system resources.

### Many-to-one (M:1) implementations (user-level threads)

In `M:1` threading implementations, all of the details of thread creation, *scheduling*, and *synchronization* are handled entirely within the process by a user-space threading library.

#### Advantages

- The greatest advantage is that many threading operations are **fast**.
  > for example, creating and terminating a thread, context switching between threads, and mutex and condition variable operations.  
  since a switch to kernel mode is *not required*.

- Since kernel support for the **threading library** is **not required**, an `M:1` implementation can be *relatively easily ported* from one system to another.

#### Disadvantages of `M:1`

- When a thread makes a system call such as `read(2)`, control passes from the user- space threading library to the kernel.
  > This means that if the `read()` call blocks, then **all threads** in the process are **blocked**.

- The kernel can’t *schedule* the threads of a process. Since the kernel is **unaware** of the existence of multiple threads within the process, it can’t schedule the separate threads to different processors on multiprocessor hardware.

### One-to-one (1:1) implementations (kernel-level threads)

In a `1:1` threading implementation, *each* thread maps onto a **separate** KSE. The
kernel handles each thread’s scheduling **separately**.

`1:1` implementations eliminate the disadvantages suffered by `M:1` implementations.

#### Disadvantages of `1:1`

- However, **operations** such as thread creation, context switching, and synchronization are **slower** on a `1:1` implementations, since a switch into *kernel mode* is **required**.

- Furthermore, the overhead required to maintain a **separate** KSE for **each** of the threads in an application that contains a *large number* of threads may place a significant load on the kernel scheduler, degrading overall system performance.

### Many-to-many (M:N) implementations (two-level model)

`M:N` implementations aim to combine the advantages of the `1:1` and `M:1` models, while eliminating their disadvantages.

In the M:N model, each process can have **multiple** associated *KSEs*, and **several** threads may map to each KSE.

> This design permits the kernel to distribute the threads of an application across multiple CPUs, while eliminating the possible scaling problems associated with applications that employ large numbers of threads.

The most significant disadvantage of the M:N model is **complexity**.

---

## Linux Implementations of POSIX Threads

Linux has two main implementations of the Pthreads API:

- **LinuxThreads**: This is the original Linux threading implementation, developed by *Xavier Leroy*.

- **NPTL** (*Native POSIX Threads Library*): This is the modern Linux threading implementation, developed by Ulrich Drepper and Ingo Molnar as a successor to LinuxThreads.
  > NPTL provides performance that is superior to LinuxThreads, and it adheres more closely to the SUSv3 specification for Pthreads.

### LinuxThreads

The essentials of the LinuxThreads implementation are as follows:

- Threads are created using a `clone()` call that specifies the following flags:

  ```c
  CLONE_VM | CLONE_FILES | CLONE_FS | CLONE_SIGHAND
  ```

- In addition to the threads created by the application, *LinuxThreads* creates an additional “**manager**” thread that **handles** thread creation and termination.

- The implementation uses signals for its internal operation.

#### LinuxThreads deviations from specified behavior

Check pages *690-691* for full list.

#### Other problems with LinuxThreads

The LinuxThreads implementation has the following problems, as well:

- If the manager thread is killed, then the remaining threads must be **manually** cleaned up.
- A *core dump* of a multithreaded program **may not include** all of the threads of the process.

- The nonstandard `ioctl()` `TIOCNOTTY` operation can remove the process’s association with a controlling terminal only when called from the main thread.

### NPTL

*NPTL* was designed to address most of the shortcomings of *LinuxThreads*.

check pages *692-694* for full explanation.

### Which Threading Implementation?

We may sometimes need to answer the following questions:

- Which threading implementation is available in a particular Linux distribution?

- On a Linux distribution that provides both LinuxThreads and NPTL, which implementation is used by default, and how can we explicitly select the implementation that is used by a program?

#### Discovering the threading implementation

We can use the following command to discover which threading implementation the system provides, or, if it provides both implementation, then which one is used by default:

```bash
$ getconf GNU_LIBPTHREAD_VERSION
```

