# Notes From ***THREADS: THREAD SYNCHRONIZATION***

- [Notes From ***THREADS: THREAD SYNCHRONIZATION***](#notes-from-threads-thread-synchronization)
  - [Protecting Accesses to Shared Variables: Mutexes](#protecting-accesses-to-shared-variables-mutexes)
    - [Statically Allocated Mutexes](#statically-allocated-mutexes)
    - [Locking and Unlocking a Mutex](#locking-and-unlocking-a-mutex)
      - [`pthread_mutex_trylock()` and `pthread_mutex_timedlock()`](#pthread_mutex_trylock-and-pthread_mutex_timedlock)
    - [Performance of Mutexes](#performance-of-mutexes)
    - [Mutex Deadlocks](#mutex-deadlocks)
    - [Dynamically Initializing a Mutex](#dynamically-initializing-a-mutex)
    - [Mutex Attributes](#mutex-attributes)
    - [Mutex Types](#mutex-types)
  - [Signaling Changes of State: Condition Variables](#signaling-changes-of-state-condition-variables)
    - [Statically Allocated Condition Variables](#statically-allocated-condition-variables)
    - [Signaling and Waiting on Condition Variables](#signaling-and-waiting-on-condition-variables)
    - [Dynamically Allocated Condition Variables](#dynamically-allocated-condition-variables)
  - [END](#end)

## Protecting Accesses to Shared Variables: Mutexes

The term **critical section** is used to refer to a section of code that accesses a shared resource and whose execution should be **atomic**.
> i.e., its execution should not be interrupted by another thread that simultaneously accesses the same shared resource.

To avoid the problems that can occur when threads try to **update** a shared variable, we must use a `mutex` (short for *mutual exclusion*) to ensure that only **one** thread at a time can *access* the variable.

A mutex has two states: **locked** and **unlocked**.  
At any moment, at most **one** thread may hold the lock on a mutex. Attempting to lock a mutex that is already locked either *blocks* or *fails* with an error.

In general, we employ a different mutex for each shared resource and each thread employs the following protocol for *accessing* a resource:

- lock the mutex for the shared resource
- access the shared resource; and
- unlock the mutex.

### Statically Allocated Mutexes

A mutex can either be allocated as a **static** variable or be created **dynamically** at run time.

A mutex is a variable of the type `pthread_mutex_t`. Before it can be used, a mutex **must always** be initialized.

For a *statically* allocated mutex, we can do this by assigning it the value `PTHREAD_MUTEX_INITIALIZER`

```c
pthread_mutex_t mtx = PTHREAD_MUTEX_INITIALIZER;
```

### Locking and Unlocking a Mutex

To lock and unlock a mutex, we use the `pthread_mutex_lock()` and `pthread_mutex_unlock()` functions.

```c
int pthread_mutex_lock(pthread_mutex_t * mutex );
int pthread_mutex_unlock(pthread_mutex_t * mutex );
```

If the calling thread itself has **already** locked the mutex given to `pthread_mutex_lock()`, then, for the default type of mutex, one of two implementation-defined possibilities may result:

Either the thread **deadlocks**, blocked trying to lock a mutex that it already owns, or the call **fails**, returning the error `EDEADLK`.

#### `pthread_mutex_trylock()` and `pthread_mutex_timedlock()`

The `pthread_mutex_trylock()` function is the same as `pthread_mutex_lock()`, except that if the mutex is currently locked, `pthread_mutex_trylock()` fails, returning the error `EBUSY` .

The `pthread_mutex_timedlock()` function is the same as `pthread_mutex_lock()`, except that the caller can specify an additional argument, *abstime*, which is timeout for locking.

### Performance of Mutexes

### Mutex Deadlocks

Sometimes, a thread needs to simultaneously access *two or more* different shared resources, each of which is governed by a *separate* mutex. When more than one thread is locking the *same* set of mutexes, **deadlock** situations can arise.

The simplest way to avoid such deadlocks is to define a mutex **hierarchy**. When threads can lock the same set of mutexes, they should always lock them in the **same order**.

### Dynamically Initializing a Mutex

We must **dynamically** initialize the mutex using `pthread_mutex_init()`.

```c
int pthread_mutex_init(pthread_mutex_t * mutex , const pthread_mutexattr_t * attr );
```

Among the cases where we must use `pthread_mutex_init()` rather than a static initializer are the following:

- The mutex was dynamically allocated on the heap.
- The mutex is an automatic variable allocated on the stack.
- We want to initialize a statically allocated mutex with attributes other than the defaults.

Dynamically allocated mutex should be destroyed using `pthread_mutex_destroy()`.

```c
int pthread_mutex_destroy(pthread_mutex_t * mutex );
```

### Mutex Attributes

### Mutex Types

---

## Signaling Changes of State: Condition Variables

A **condition variable** allows one thread to inform other threads about *changes* in the state of a shared variable and allows the other threads to wait for such notification.

### Statically Allocated Condition Variables

A condition variable has the type `pthread_cond_t`. As with a mutex, a condition variable must be initialized before use.

```c
pthread_cond_t cond = PTHREAD_COND_INITIALIZER;
```

### Signaling and Waiting on Condition Variables

The principal condition variable operations are **signal** and **wait**.

- **Signal**: operation is a notification to one or more waiting threads that a shared variable’s state has changed.

- **Wait**: operation is the means of blocking until such a notification is received.

The `pthread_cond_signal()` and `pthread_cond_broadcast()` functions both signal the condition variable.

The `pthread_cond_wait()` function blocks a thread until the condition variable is signaled.

```c
int pthread_cond_signal(pthread_cond_t * cond );
int pthread_cond_broadcast(pthread_cond_t * cond );
int pthread_cond_wait(pthread_cond_t * cond , pthread_mutex_t * mutex );
```

With `pthread_cond_signal()`, we are simply guaranteed that *at least one* of the blocked threads is woken up; with `pthread_cond_broadcast()`, **all** blocked threads are woken up.

The `pthread_cond_timedwait()` function is the same as `pthread_cond_wait()`,
though with timeout specification.

```c
int pthread_cond_timedwait(pthread_cond_t * cond , pthread_mutex_t * mutex ,
const struct timespec * abstime );
```

### Dynamically Allocated Condition Variables

The `pthread_cond_init()` function is used to dynamically initialize a condition variable

> The circumstances in which we need to use `pthread_cond_init()` are analogous to those where `pthread_mutex_init()` is needed to dynamically initialize a mutex.

```c
int pthread_cond_init(pthread_cond_t * cond , const pthread_condattr_t * attr );
```

When an automatically or dynamically allocated condition variable is no longer required, then it should be **destroyed** using `pthread_cond_destroy()`.

```c
int pthread_cond_destroy(pthread_cond_t * cond );
```

---

## END
