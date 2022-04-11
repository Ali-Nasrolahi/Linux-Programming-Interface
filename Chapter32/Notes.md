# Notes From ***THREADS: THREAD CANCELLATION***

- [Notes From ***THREADS: THREAD CANCELLATION***](#notes-from-threads-thread-cancellation)
  - [Canceling a Thread](#canceling-a-thread)
  - [Cancellation State and Type](#cancellation-state-and-type)
  - [Cancellation Points](#cancellation-points)
  - [Testing for Thread Cancellation](#testing-for-thread-cancellation)
  - [Cleanup Handlers](#cleanup-handlers)
  - [Asynchronous Cancelability](#asynchronous-cancelability)
  - [END](#end)

## Canceling a Thread

The `pthread_cancel()` function sends a cancellation request to the specified thread.

```c
int pthread_cancel(pthread_t thread );
```

---

## Cancellation State and Type

The `pthread_setcancelstate()` and `pthread_setcanceltype()` functions set flags that allow a thread to control how it responds to a cancellation request.

```c
int pthread_setcancelstate(int state , int * oldstate );
int pthread_setcanceltype(int type , int * oldtype );
```

---

## Cancellation Points

When cancelability is enabled and deferred, a cancellation request is acted upon only when a thread next reaches a **cancellation point**.

A cancellation point is a call to one of a set of functions defined by the implementation.

check page *673* for full table.

---

## Testing for Thread Cancellation

The purpose of `pthread_testcancel()` is simply to be a cancellation point. If a cancellation is pending when this function is called, then the calling thread is terminated.

```c
void pthread_testcancel(void);
```

## Cleanup Handlers

A thread can establish one or more **cleanup handlers**—functions that are automatically executed if the thread is canceled.

The `pthread_cleanup_push()` and `pthread_cleanup_pop()` functions respectively add and remove handlers on the calling thread’s stack of cleanup handlers.

---

## Asynchronous Cancelability

When a thread is made asynchronously cancelable (cancelability type `PTHREAD_CANCEL_ASYNCHRONOUS`), it may be canceled at any time.
> (i.e., at any machine-language instruction)

This type of cancelability is used in rare cases and should be use with caution.

---

## END
