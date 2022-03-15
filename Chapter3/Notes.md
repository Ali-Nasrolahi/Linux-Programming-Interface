# Notes from ***SYSTEM PROGRAMMING CONCEPTS***

- [Notes from ***SYSTEM PROGRAMMING CONCEPTS***](#notes-from-system-programming-concepts)
  - [System Calls](#system-calls)
    - [How syscalls work?](#how-syscalls-work)
  - [Library Functions](#library-functions)
  - [The Standard C Library; The GNU C Library ( glibc )](#the-standard-c-library-the-gnu-c-library--glibc-)
  - [Handling Errors from System Calls and Library Functions](#handling-errors-from-system-calls-and-library-functions)
    - [Handling system call errors](#handling-system-call-errors)
  - [Notes on the Example Programs in This Book](#notes-on-the-example-programs-in-this-book)
  - [Portability Issues](#portability-issues)
    - [Feature Test Macros](#feature-test-macros)
    - [System Data Types](#system-data-types)
    - [Miscellaneous Portability Issues](#miscellaneous-portability-issues)
      - [Initializing and using structures](#initializing-and-using-structures)
      - [Using macros that may not be present on all implementations](#using-macros-that-may-not-be-present-on-all-implementations)
  - [END](#end)

## System Calls

**system call** is a controlled entry point into the kernel, allowing a process to **request** that the *kernel* perform some action on the process’s *behalf*.

checkout all syscalls in `syscall(2)` manual page.

Some notes about syscalls:

- A system call changes the processor state from **user mode** to **kernel mode**, so that the CPU can access protected kernel memory.

- The set of system calls is **fixed**. Each system call is identified by a **unique** number.

- Each system call may have a set of arguments that specify information to be transferred from user space to kernel space and vice versa.

### How syscalls work?

**NOTE**: Explanation of syscalls' functionality could be vague ,with ahead description,
therefore checkout pages 44 & 45 of the book to understand delicately.

1. The application program makes a system call by invoking a wrapper function in the C library.

2. The wrapper function must make all of the system call arguments available to the system call trap-handling routine. The wrapper function copies the arguments to specified registers.

3. The wrapper function copies the system call number into a specific CPU register ( %eax ) so kernel could identify each syscalls.

4. The wrapper function executes a **trap** machine instruction ( int 0x80 ), which causes the processor to switch from *user mode* to *kernel mode* and execute code pointed to by location **0x80** (128 decimal) of the system’s trap vector.

5. In response to the trap to location 0x80 , the kernel invokes its `system_call()` routine to handle the trap.

6. If the return value of the system call service routine indicated an error, the wrapper function sets the global variable **errno** using this value. The wrapper function then returns to the caller, providing an integer return value indicating the success or failure of the system call.

---

## Library Functions

A library function is simply one of the multitude of functions that constitutes the standard C library.

Many library functions don’t make any use of system calls.

Often, library functions are designed to provide a more caller-friendly interface than the underlying system call.

## The Standard C Library; The GNU C Library ( glibc )

```c
const char *gnu_get_libc_version(void);
```

> Returns pointer to null-terminated, statically allocated string containing GNU C library version number"such as 2.12."

## Handling Errors from System Calls and Library Functions

Almost every system call and library function returns status value, this value should **always** be checked to see whether the call succeeded.

> A few system calls never fail. For example, `getpid()`, `_exit()`

### Handling system call errors

Usually, an error is indicated by a return of –1. Check manual page for certainty.

When a system call fails, it sets the global integer variable **errno** to a positive value that identifies the specific error.

**NOTE:**

Successful system calls and library functions **never** reset *errno* to "0".

> So this variable may have a nonzero value as a consequence of an error from a previous call.

**TIP:**

A few system calls (e.g., `getpriority()`) can legitimately return –1 on **success**.

To determine whether an error occurs in such calls, we set errno to 0 **before** the call, and then **check** it *afterward*.

> If the call returns –1 and errno is nonzero, an error occurred.

The `perror()` function prints the string pointed to by its msg argument, followed by a message corresponding to the current value of errno.

```c
#include <stdio.h>
void perror(const char * msg );
```

The `strerror()` function returns the error string corresponding to the error number
given in its `errnum`argument.

```c
#include <string.h>
char *strerror(int errnum);
```

---

## Notes on the Example Programs in This Book

> This section talks about its own policies for ahead sample codes, therefore **NO NOTE**.

---

## Portability Issues

### Feature Test Macros

Various standards govern the behavior of the system call and library function APIs.

> Some of these standards are defined by standards bodies such SUS, while others are defined by the two historically important UNIX implementations: BSD and System V Release 4.

Sometimes, when writing a portable application, we may want the various header files to expose only the definitions that follow a particular **standard**.

To do this, we define one or more of the **feature test macros** when compiling a program.

Methods to do so are:

```c
#define _BSD_SOURCE 1
```

And

```bash
cc -D_BSD_SOURCE prog.c 
# or gcc and many other compiler, 
# nevertheless checkout ,whatever compiler you're using, manuals just to make sure beforehand.
```

> To check each standard's definition of macros and prototype, remember pages 61 to 63 of the book.

### System Data Types

Various implementation data types are represented using standard C types.

Although it would be possible to use the C fundamental types such as *int* and *long* to declare variables storing such information, this **reduces** portability across UNIX systems, for the following reasons:

1. The sizes of these fundamental types **vary** across UNIX implementations (e.g., a long may be *4 bytes* on one system and *8 bytes* on another)
    > or sometimes even in different compilation environments on the same implementation.

2. Even on a single UNIX implementation, the types used to represent information may **differ** between **releases** of the implementation.
    > Notable examples on Linux are user and group IDs. On Linux 2.2 and earlier, these values were represented in 16 bits. On Linux 2.4 and later, they are 32-bit values

To avoid such portability problems, SUSv3 specifies various standard system data types, and requires an implementation to define and use these types appropriately.

Each of these types is defined using the C `typedef` feature.

### Miscellaneous Portability Issues

#### Initializing and using structures

Don't initialize struct like this :

```c
struct sembuf s = { 3, -1, SEM_UNDO }; // Not necessarily portable
```

Instead use:

```c
struct sembuf s;
s.sem_num = 3;
s.sem_op = -1;
s.sem_flg = SEM_UNDO;
```

Or if you're using C99, try this then:

```c
struct sembuf s = { .sem_num = 3, .sem_op = -1, .sem_flg = SEM_UNDO };
```

#### Using macros that may not be present on all implementations

In some cases, a macro may be not be defined on all UNIX implementations

So to make sure go with:

```c
#ifdef WCOREDUMP
    /* Use WCOREDUMP() macro */
#endif
```

---

## END
