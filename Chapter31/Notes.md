# Notes From ***THREADS: THREAD SAFETY AND PER-THREAD STORAGE***

- [Notes From ***THREADS: THREAD SAFETY AND PER-THREAD STORAGE***](#notes-from-threads-thread-safety-and-per-thread-storage)
  - [Thread Safety (and Reentrancy Revisited)](#thread-safety-and-reentrancy-revisited)
    - [Reentrant and nonreentrant functions](#reentrant-and-nonreentrant-functions)
  - [One-Time Initialization](#one-time-initialization)

## Thread Safety (and Reentrancy Revisited)

A function is said to be **thread-safe** if it can safely be invoked by multiple threads at the same time.

One of solutions for thread-safety is using mutex.

### Reentrant and nonreentrant functions

A **reentrant function** achieves thread safety **without** the use of *mutexes*. It does this by avoiding the use of *global* and *static* variables.

---

## One-Time Initialization

