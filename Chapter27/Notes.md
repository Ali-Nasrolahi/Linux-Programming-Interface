# Notes From ***PROGRAM EXECUTION***

- [Notes From ***PROGRAM EXECUTION***](#notes-from-program-execution)
  - [Executing a New Program: `execve()`](#executing-a-new-program-execve)
  - [The `exec()` Library Functions](#the-exec-library-functions)
    - [The `PATH` Environment Variable](#the-path-environment-variable)
    - [Specifying Program Arguments as a List](#specifying-program-arguments-as-a-list)
    - [Passing the Caller’s Environment to the New Program](#passing-the-callers-environment-to-the-new-program)
    - [Executing a File Referred to by a Descriptor: `fexecve()`](#executing-a-file-referred-to-by-a-descriptor-fexecve)
  - [Interpreter Scripts](#interpreter-scripts)
    - [Execution of interpreter scripts](#execution-of-interpreter-scripts)
    - [Using the script `optional-arg`](#using-the-script-optional-arg)
    - [Executing scripts with `execlp()` and `execvp()`](#executing-scripts-with-execlp-and-execvp)
  - [File Descriptors and `exec()`](#file-descriptors-and-exec)

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

The value of `PATH` is a string consisting of colon-separated directory names called **path prefixes**.

e.g,

```bash
$ echo $PATH
/home/mtk/bin:/usr/local/bin:/usr/bin:/bin:.
```

The `PATH` value for a login shell is set by *system-wide* and *user-specific* shell startup scripts.

The directory pathnames specified in `PATH` can be either **absolute** (commencing with an initial `/` ) or **relative**.

> A relative pathname is interpreted with respect to the current working directory of the calling process.

### Specifying Program Arguments as a List

When we **know** the number of arguments for an `exec()` at the time we write a program, we can use `execle()`, `execlp(),` or `execl()` to specify the arguments as a *list* within the function call.

### Passing the Caller’s Environment to the New Program

For security reasons, it is sometimes preferable to ensure that a program is exceed with a known environment list.

### Executing a File Referred to by a Descriptor: `fexecve()`

Since version 2.3.2, glibc provides `fexecve()`, which behaves just like `execve()`, but specifies the file to be execed via the **open file descriptor** `fd`, rather than as a *pathname*.

```c
int fexecve(int fd , char *const argv [], char *const envp []);
```

---

## Interpreter Scripts

An **interpreter** is a program that reads commands in text form and executes them.

> This contrasts with a compiler, which translates input source code into a machine language that can then be executed on a real or virtual machine.

UNIX kernels allow interpreter scripts to be run in the same way as a binary program file, as long as two requirements are met.

1. First, **execute permission** must be enabled for the script file.

2. Second, the file must contain an initial line that specifies the *pathname of the interpreter* to be used to run the script.

This line has the following form:

```text
#! interpreter-path [ optional-arg ]
```

The `PATH` environment variable is **not used** in interpreting this pathname, so that an **absolute** pathname usually should be specified.

> A relative pathname is also possible, though unusual.

### Execution of interpreter scripts

If `execve()` detects that the file it has been given commences with the 2-byte sequence `#!` , then it extracts the remainder of the line, and execs the interpreter file with the following list of arguments:

```text
interpreter-path [ optional-arg ] script-path arg ...
```

### Using the script `optional-arg`

One use of the `optional-arg` in a script’s initial `#!` line is to specify command-line options for the **interpreter**. This feature is useful with certain interpreters, such as `awk`.

### Executing scripts with `execlp()` and `execvp()`

Normally, the absence of a `#!` line at the start of a script causes the `exec()` functions to fail. However, `execlp()` and `execvp()` do things somewhat **differently**.

If either of these functions finds a file that has *execute permission* turned on, but is **not a binary** executable and **does not** start with a `#!` line, then they exec the `shell` to interpret the file.

> On Linux, this means that such files are treated as though they started with a line containing the string `#!/bin/sh`.

---

## File Descriptors and `exec()`
