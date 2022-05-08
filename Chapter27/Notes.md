# Notes From ***PROGRAM EXECUTION***

- [Notes From ***PROGRAM EXECUTION***](#notes-from-program-execution)
  - [Executing a New Program: `execve()`](#executing-a-new-program-execve)
  - [The `exec()` Library Functions](#the-exec-library-functions)
    - [The `PATH` Environment Variable](#the-path-environment-variable)

## Executing a New Program: `execve()`

The `execve()` system call loads a new program into a process’s memory.

During this operation, the old program is discarded, and the process’s stack, data, and heap are **replaced** by those of the new program.

```c
int execve(const char * pathname , char *const argv [], char *const envp []);
```

Since `execve()` replaces the program that called it, a **successful** `execve()` **never returns**.  We never need to check the return value from `execve()`; it will always be `–1`.

---

## The `exec()` Library Functions

The library functions described in this section provide alternative APIs for performing an `exec()`.

All of these functions are layered on top of execve(), and they differ from one another and from `execve()` only in the way in which the **program name**, **argument list**, and **environment** of the new program are specified.

```c
int execle(const char * pathname , const char * arg , ...
            /* , (char *) NULL, char *const envp [] */ );
int execlp(const char * filename , const char * arg , ...
            /* , (char *) NULL */);
int execvp(const char * filename , char *const argv []);
int execv(const char * pathname , char *const argv []);
int execl(const char * pathname , const char * arg , ...
            /* , (char *) NULL */);
```

The final letters in the names of these functions provide a clue to the differences between them. These differences are summarized detailed in the following list:

- Most of the `exec()` functions expect a pathname as the specification of the new program to be loaded. However, `execlp()` and `execvp()` allow the program to be specified using just a filename.

- Instead of using *an array* to specify the `argv` list for the new program, `execle()`, `execlp()`, and `execl()` require the programmer to specify the arguments as a *list of strings* within the call.

- The `execve()` and `execle()` functions allow the programmer to **explicitly** specify the environment for the new program using `envp`, a NULL-terminated array of pointers to character strings.

### The `PATH` Environment Variable
