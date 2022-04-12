# Notes From ***SIGNALS: SIGNAL HANDLERS***

- [Notes From ***SIGNALS: SIGNAL HANDLERS***](#notes-from-signals-signal-handlers)
  - [Designing Signal Handlers](#designing-signal-handlers)
    - [Signals Are Not Queued (Revisited)](#signals-are-not-queued-revisited)
    - [Reentrant and Async-Signal-Safe Functions](#reentrant-and-async-signal-safe-functions)
      - [Reentrant and nonreentrant functions](#reentrant-and-nonreentrant-functions)
      - [Standard async-signal-safe functions](#standard-async-signal-safe-functions)
      - [Use of `errno` inside signal handlers](#use-of-errno-inside-signal-handlers)
    - [Global Variables and the `sig_atomic_t` Data Type](#global-variables-and-the-sig_atomic_t-data-type)
  - [Other Methods of Terminating a Signal Handler](#other-methods-of-terminating-a-signal-handler)
    - [Performing a Nonlocal Goto from a Signal Handler](#performing-a-nonlocal-goto-from-a-signal-handler)
    - [Terminating a Process Abnormally: `abort()`](#terminating-a-process-abnormally-abort)
  - [Handling a Signal on an Alternate Stack: sigaltstack()](#handling-a-signal-on-an-alternate-stack-sigaltstack)
  - [The `SA_SIGINFO` Flag](#the-sa_siginfo-flag)

## Designing Signal Handlers

In general, it is preferable to write simple signal handlers.
> One important reason for this is to reduce the risk of creating race conditions.

Two common designs for signal handlers are the following:

- The signal handler sets a global flag and exits.

- The signal handler performs some type of cleanup and then either terminates the process or uses a nonlocal goto to unwind the stack and return control to a predetermined location in the main program.

### Signals Are Not Queued (Revisited)

### Reentrant and Async-Signal-Safe Functions

Not all system calls and library functions can be safely called from a signal handler.

To understand why we need to review 2 concepts:

- reentrant functions
- async-signal-safe functions.

#### Reentrant and nonreentrant functions

A function is said to be **reentrant** if it can *safely* be **simultaneously** executed by *multiple threads* of execution in the same process.
> “safe” means that the function achieves its expected result, regardless of the state of execution of any other thread of execution.

A function may be **nonreentrant** if it updates **global** or **static** data structures.

#### Standard async-signal-safe functions

An **async-signal-safe** function is one that the implementation guarantees to be safe when called from a **signal handler**.

> A function is **async-signal-safe** either because it is *reentrant* or because it is *not interruptible* by a signal handler.

#### Use of `errno` inside signal handlers

Because even async-signal-safe functions may update *errno*, we should save the old value of *errno* before doing any operation which could modify this value; and restore its original value at the end.

### Global Variables and the `sig_atomic_t` Data Type

Sharing global variables between signal handler and other threads (e.g., main thread) only could be used as a **checking value** and does associated actions with it.

In other words, only signal handler should modify the value and others such as main thread must just check the value.(or vice versa)

When global variables are accessed in this way from a signal handler, we should always declare them using the **volatile** attribute

Reading and writing global variables may involve more than one machine-language instruction, and a signal handler may **interrupt** the main program in the **middle** of such an instruction sequence.(accessing the variable is *nonatomic*)

Solution to this matter is ,`sig_atomic_t` data type for which reads and writes are guaranteed to be **atomic**.

For instance:

```c
volatile sig_atomic_t flag;
```

----

## Other Methods of Terminating a Signal Handler

Sometimes returning to main section after signal handling isn’t desirable, or in some cases, isn’t even useful.

There are various other ways of terminating a signal handler:

- Use `_exit()` to terminate the process.
- Use `kill()` or `raise()` to send a signal that kills the process
- Perform a **nonlocal goto** from the signal handler.
- Use the `abort()` function to terminate the process with a core dump.

### Performing a Nonlocal Goto from a Signal Handler

We could use `setjump()` and `longjump()` to perform nonlocal goto, however there is a problem with this approach which is handling process **signal mask**.

We noted earlier that, upon entry to the signal handler, the kernel **automatically adds** the invoking signal, as well as any signals specified in the `act.sa_mask field`, to the process signal mask, and then **removes** these signals from the mask when the handler does a *normal return*.

What happens to the *signal mask* if we exit the signal handler using`longjmp()`?

Under linux (based on System V semantics), `longjmp()` **doesn’t restore** the signal mask, so that blocked signals are not unblocked upon leaving the handler.

Because of this difference in the two main UNIX variants, *POSIX.1-1990* chose not to specify the handling of the signal mask by `setjmp()` and`longjmp()`.

Instead, it defined a pair of new functions, `sigsetjmp()` and `siglongjmp()`, that provide **explicit** control of the *signal mask* when performing a nonlocal goto.

```c
int sigsetjmp(sigjmp_buf env , int savesigs );

void siglongjmp(sigjmp_buf env , int val );
```

### Terminating a Process Abnormally: `abort()`

The abort() function terminates the calling process and causes it to produce a *core dump*.

```c
void abort(void);
```

----

## Handling a Signal on an Alternate Stack: sigaltstack()

Normally, when a signal handler is invoked, the kernel creates a frame for it on the **process stack**. However, this *may not be possible* if a process attempts to extend the stack beyond the maximum possible size.

If we need to ensure that the `SIGSEGV` signal is handled in these circumstances, we can do the following:

1. Allocate an area of memory, called an alternate signal stack, to be used for the stack frame of a *signal handler*.

2. Use the `sigaltstack()` system call to inform the kernel of the existence of the alternate signal stack.

3. When establishing the signal handler, specify the *SA_ONSTACK* flag, to tell the kernel that the frame for this handler should be created on the alternate stack.

The `sigaltstack()` system call both **establishes** an alternate signal stack and **returns information** about any alternate signal stack that is already established.

```c
int sigaltstack(const stack_t * sigstack , stack_t * old_sigstack );
```

----

## The `SA_SIGINFO` Flag
