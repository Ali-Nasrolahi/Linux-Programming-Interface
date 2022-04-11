# Notes From ***THREADS: THREAD SAFETY AND PER-THREAD STORAGE***

- [Notes From ***THREADS: THREAD SAFETY AND PER-THREAD STORAGE***](#notes-from-threads-thread-safety-and-per-thread-storage)
  - [Thread Safety (and Reentrancy Revisited)](#thread-safety-and-reentrancy-revisited)
    - [Reentrant and nonreentrant functions](#reentrant-and-nonreentrant-functions)
  - [One-Time Initialization](#one-time-initialization)
  - [Thread-Specific Data](#thread-specific-data)
    - [Thread-Specific Data from the Library Function’s Perspective](#thread-specific-data-from-the-library-functions-perspective)
    - [Overview of the Thread-Specific Data API](#overview-of-the-thread-specific-data-api)
    - [Details of the Thread-Specific Data API](#details-of-the-thread-specific-data-api)
  - [Thread-Local Storage](#thread-local-storage)
  - [END](#end)

## Thread Safety (and Reentrancy Revisited)

A function is said to be **thread-safe** if it can safely be invoked by multiple threads at the same time.

One of solutions for thread-safety is using mutex.

### Reentrant and nonreentrant functions

A **reentrant function** achieves thread safety **without** the use of *mutexes*. It does this by avoiding the use of *global* and *static* variables.

---

## One-Time Initialization

A library function can perform one-time initialization using the `pthread_once()` function.

```c
int pthread_once(pthread_once_t * once_control , void (* init )(void));
```

---

## Thread-Specific Data

**NOTE**: This section contains some advanced subjects which I could not fully absorb, Therefore I'd be back later and complete it.

The most efficient way of making a function thread-safe is to make it *reentrant*.

**Thread-specific data** is a technique for making an existing function thread-safe without changing its interface.

Thread-specific data allows a function to maintain a **separate** copy of a variable for **each** thread that calls the function

### Thread-Specific Data from the Library Function’s Perspective

### Overview of the Thread-Specific Data API

### Details of the Thread-Specific Data API

Calling `pthread_key_create()` creates a new thread-specific data key that is returned to the caller in the buffer.

```c
int pthread_key_create(pthread_key_t * key , void (* destructor )(void *));
```

The `pthread_setspecific()` function requests the Pthreads API to save a *copy of value* in a data structure that associates it with the calling thread and with *key*.

The `pthread_getspecific()` function performs the converse operation, returning the value that was previously associated with the given *key* for this thread.

```c
int pthread_setspecific(pthread_key_t key , const void * value );
void *pthread_getspecific(pthread_key_t key );
```

---

## Thread-Local Storage

Like thread-specific data, thread-local storage provides persistent per-thread storage.

To create a thread-local variable, we simply include the `__thread` specifier in the declaration of a global or static variable:

```c
static __thread buf[MAX_ERROR_LEN];
```

---

## END
