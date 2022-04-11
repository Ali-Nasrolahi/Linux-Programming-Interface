# Notes From ***THREADS: INTRODUCTION***

- [Notes From ***THREADS: INTRODUCTION***](#notes-from-threads-introduction)
  - [Overview](#overview)
    - [Multiprocess VS Multithread](#multiprocess-vs-multithread)
  - [Background Details of the Pthreads API](#background-details-of-the-pthreads-api)
    - [Pthreads data types](#pthreads-data-types)
    - [Threads and `errno`](#threads-and-errno)
    - [Return value from Pthreads functions](#return-value-from-pthreads-functions)
  - [Thread Creation](#thread-creation)
  - [Thread Termination](#thread-termination)
  - [Thread IDs](#thread-ids)
  - [Joining with a Terminated Thread](#joining-with-a-terminated-thread)
  - [Detaching a Thread](#detaching-a-thread)

## Overview

Like processes, `threads` are a mechanism that permits an application to perform multiple tasks *concurrently*.
> A single process can contain multiple threads.

All of these threads are independently executing the same program, and they all share the same **global memory**, including the *initialized data*, *uninitialized data*, and *heap segments*.

### Multiprocess VS Multithread

Although using multiprocess program to handle tasks simultaneously works well in many scenarios, theses limitations makes is controversial:

- It is difficult to **share** information between **processes**. Since the parent and child *don’t* share memory.
    > we must use some form of IPC to exchange data.
- Process creation with `fork()` is relatively **expensive**. Even with the *copy-on-write* technique.

Threads address both of these problems:

- **Sharing** information between threads is easy and fast. It is just a matter of copying data into shared (global or heap) variables.

- Thread creation is **faster** than process creation—typically, ten times faster or better.  
Thread creation is faster because many of the attributes that must be **duplicated** in a child created by `fork()` are instead **shared** between threads.

---

## Background Details of the Pthreads API

### Pthreads data types

Checkout Pthreads data types in `pthreads.h` and page 620 for explanation.

### Threads and `errno`

In the traditional UNIX API, `errno` is a **global** integer variable.

If a thread made a function call that returned an error in a global *errno* variable, then this would confuse other threads that might also be making function calls and checking *errno*. In other words, **race conditions** would result.

Therefore, in threaded programs, each thread has its own *errno* value.

### Return value from Pthreads functions

The traditional method of returning status from system calls and some library functions is to return 0 on success and –1 on error, with *errno* being set to indicate the error.

However,

All Pthreads functions return *0* on **success** or a *positive* value on **failure**.
> The failure value is one of the same values that can be placed in `errno` by traditional UNIX system calls.

---

## Thread Creation

The `pthread_create()` function creates a new thread.

```c
int pthread_create(pthread_t * thread,
const pthread_attr_t * attr , void *(* start )(void *), void * arg );
```

---

## Thread Termination

The execution of a thread terminates in one of the following ways:

- The thread’s start function performs a **return** specifying a return value for the thread.
- The thread calls`pthread_exit()`.
- The thread is canceled using`pthread_cancel()`.
- Any of the threads calls `exit()`, or the main thread performs a *return*, which causes all threads in the process to **terminate** immediately.

The `pthread_exit()` function **terminates** the calling thread, and specifies a *return value* that can be obtained in another thread by calling `pthread_join()`.

```c
void pthread_exit(void * retval);
```

---

## Thread IDs

Each thread within a process is **uniquely** identified by a *thread ID*. This ID is returned to the caller of `pthread_create()`, and a thread can obtain its **own** ID using `pthread_self()`.

```c
pthread_t pthread_self(void);
```

Thread IDs are useful within applications for the following reasons:

- Various Pthreads functions use *thread IDs* to identify the thread on which they are to act.
    > e.g. `pthread_join()`, `pthread_detach()`, `pthread_cancel()`, and `pthread_kill()`.
- In some applications, it can be useful to tag **dynamic** data structures with the ID of a particular thread.

The `pthread_equal()` function allows us check whether two thread IDs are the same.

```c
int pthread_equal(pthread_t t1 , pthread_t t2 );
```

The `pthread_equal()` function is needed because the `pthread_t` data type must be treated as **opaque** data.

---

## Joining with a Terminated Thread

The `pthread_join()` function waits for the thread identified by thread to terminate.

```c
int pthread_join(pthread_t thread , void ** retval );
```

If a thread is not **detached**, then we must join with it using`pthread_join()`.  
If we fail to do this, then, when the thread terminates, it produces the thread equivalent of a **zombie** process

The task that `pthread_join()` performs for threads is similar to that performed by `waitpid()` for processes. However, there are some notable differences:

- Threads are peers. Any thread in a process can use pthread_join() to join with any other thread in the process. This differs from the **hierarchical** relationship between processes.
    > When a parent process creates a child using `fork(),` it is the only process that can `wait()` on that child.
- There is no way of saying “*join with any thread*”. nor is there a way to do a nonblocking joinThere are ways to achieve similar functionality using **condition variables**.

> `waitpid(–1, &status, options)` and `waitpid()` with **WNOHANG** flag.

---

## Detaching a Thread

Sometimes, we don’t care about the thread’s *return status*; we simply want the system to automatically clean up and remove the thread when it terminates.

In this case, we can mark the thread as **detached**, by making a call to `pthread_detach()` specifying the thread’s identifier in *thread*.

```c
int pthread_detach(pthread_t thread );
```
