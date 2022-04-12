# Notes From ***THREADS: FURTHER DETAILS***

- [Notes From ***THREADS: FURTHER DETAILS***](#notes-from-threads-further-details)
  - [Thread Stacks](#thread-stacks)
  - [Threads and Signals](#threads-and-signals)
    - [How the UNIX Signal Model Maps to Threads](#how-the-unix-signal-model-maps-to-threads)
    - [Manipulating the Thread Signal Mask](#manipulating-the-thread-signal-mask)
    - [Sending a Signal to a Thread](#sending-a-signal-to-a-thread)
    - [Dealing with Asynchronous Signals Sanely](#dealing-with-asynchronous-signals-sanely)
    - [Threads and Process Control](#threads-and-process-control)

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
