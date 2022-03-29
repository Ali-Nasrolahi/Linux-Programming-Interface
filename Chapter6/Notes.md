# Notes From ***PROCESSES***

- [Notes From ***PROCESSES***](#notes-from-processes)
  - [Processes and Programs](#processes-and-programs)
  - [Process ID and Parent Process ID](#process-id-and-parent-process-id)
  - [Memory Layout of a Process](#memory-layout-of-a-process)
  - [Virtual Memory Management](#virtual-memory-management)
  - [The Stack and Stack Frames](#the-stack-and-stack-frames)
  - [Command-Line Arguments (`argc` , `argv`)](#command-line-arguments-argc--argv)
  - [Environment List](#environment-list)
    - [Accessing the environment from a program](#accessing-the-environment-from-a-program)
    - [Modifying the environment](#modifying-the-environment)
  - [Performing a Nonlocal Goto: `setjmp()` and `longjmp()`](#performing-a-nonlocal-goto-setjmp-and-longjmp)
    - [TODO](#todo)
  - [END](#end)

## Processes and Programs

A **process** is an instance of an executing *program*.

A **program** is a file containing a range of information that describes how to construct a *process* at run time.

This information includes the following:

- **Binary format identification**: Each program file includes metainformation describing the format of the executable file. This enables the kernel to interpret the
remaining information in the file.

  - Two widely used formats for UNIX executable files were the original **a.out**
    > (“assembler output”)
  - and the later, more sophisticated **COFF**
    > (Common Object File Format)
  - Nowadays, most UNIX implementations employ the **ELF**
    > (Executable and Linking Format)
- **Machine-language instructions**: These encode the algorithm of the program.

- **Program entry-point address**: This identifies the location of the instruction at which execution of the program should commence.

- **Data**:The program file contains values used to initialize variables and also literal constants used by the program.

- **Symbol and relocation tables**: These describe the locations and names of functions and variables within the program.

- **Shared-library and dynamic-linking information**: The program file includes fields listing the shared libraries that the program needs to use at run time and the pathname of the dynamic linker that should be used to load these libraries.

- Other information: The program file contains various other information that
describes how to construct a process.

Another definition of a **process**:

A **process** is an abstract entity, defined by the kernel, to which system *resources* are **allocated** in order to execute a **program**.

From the kernel’s point of view, a process consists of:

- *user-space* memory containing program code and variables
- A range of kernel *data structures* that **maintain** information about the **state of the process**
  - includes various IDs associated with the process
  - *virtual memory tables*
  - the table of open *file descriptors*
  - information relating to **signal** delivery and handling
  - process resource usages and **limits**
  - the current working directory and etc.

---

## Process ID and Parent Process ID

Each process has a process ID (**PID**).
> a positive integer that uniquely identifies the process on the system

`pid_t getpid(void)`:

**Always successfully** returns process ID of caller.

> This syscall is one of unique sycalls which **always** returns **successfully**.

Each process has a parent—the process that created it.

`pid_t getppid(void)`

**Always successfully** returns process ID of parent of caller.

If a child process becomes orphaned because its “*birth*” parent terminates, then the child is adopted by the **init** process, and subsequent calls to `getppid()` in the child return 1.

---

## Memory Layout of a Process

The memory allocated to each process is composed of a number of parts, usually referred to as **segments**:

- *Text segment*: contains the machine-language instructions of the program run by the process.
- *Initialized data segment*: contains global and static variables that *are* explicitly **initialized**.
- *uninitialized data segment*: contains global and static variables that *are not* explicitly **initialized**.
- *stack*: is a dynamically growing and shrinking segment containing stack frames.
    > A frame stores the function’s local variables (so-called automatic variables), arguments, and return value.
- *heap*: is an area from which memory (for variables) can be dynamically allocated at run time.
    > The top end of the heap is called the **program break**.

`size(1)`: displays the size of the text, initialized data, and uninitialized data (*bss*) segments of a binary executable.

---

## Virtual Memory Management

**virtual memory management**: is to make efficient use of both the CPU and RAM (physical memory) by exploiting a property by name of **locality of reference**.

Two kinds of locality:

- **Spatial locality**: is the tendency of a program to reference memory addresses that are near those that were recently accessed.
    > because of sequential processing of instructions, and, sometimes, sequential processing of data structures

- **Temporal locality**: is the tendency of a program to access the same memory addresses in the near future that it accessed in the recent past
    > because of loops

**Page**: A virtual memory scheme splits the memory used by each program into small, fixed-size units called *pages*.

Correspondingly, RAM is divided into a series of **page frames** of the same size.

**Resident set**: resident pages of program in page frame (RAM).  
Copies of the unused pages of a program are maintained in the **swap area**.

**Swap area**: a reserved area of disk space used to supplement the computer’s RAM.  
and loaded into physical memory only as required

**Page table**: describes the location of each page in the process’s *virtual address space*.
> Each entry in the page table either indicates the location of a virtual page in RAM or indicates that it currently resides on disk.

If a process tries to access an address for which there is **no** corresponding *page-table entry*, it receives a **SIGSEGV** signal.
> Oh, yes; our old friend **Segmentation fault**.

---

## The Stack and Stack Frames

The `stack` grows and shrinks *linearly* as functions are called and return.
> A special-purpose register, the *stack pointer*, tracks the current top of the stack.  
> Each time a function is called, an additional frame is allocated on the stack, and this frame is removed when the function returns.

Each (user) stack frame contains the following information:

- **Function arguments and local variables**: In C these are referred to as `automatic variables`, since they are automatically created when a function is called. These
variables also automatically disappear when the function returns.
- **Call linkage information**: Each function uses certain CPU registers counter, Each time one function calls another, a *copy* of these registers is saved in the called function’s *stack frame*.
  > so that when the function returns, the appropriate register values can be restored for the calling function.

---

## Command-Line Arguments (`argc` , `argv`)

`int argc`: indicates how many command-line arguments there are.
  > `argv[0]`, is (conventionally) the name of the program itself.

`char *argv[]`: is an array of pointers to the command-line arguments.

To portably make the command-line arguments available in other functions, we must either pass `argv` as an *argument* to those functions or set a *global* variable pointing to `argv`.

**Nonportable** methods of accessing part or all of this information from anywhere in a program:

- The command-line arguments of any process can be read via the Linux-specific `/proc/ PID /cmdline` file.
    > A program can access its own command-line arguments via `/proc/self/cmdline`.
- The GNU C library provides two global variables that may be used anywhere in a program in order to obtain the name used to invoke the program.
  1. `program_invocation_name`: provides the complete pathname used to invoke the program.
  2. `program_invocation_short_name`: provides a version of this name with any directory prefix stripped off.
  > can be obtained from `<errno.h>` by defining the macro `_GNU_SOURCE`.

---

## Environment List

Each process has an associated array of strings called the **environment list**, or simply the *environment*.
> Each of these strings is a definition of the form `name=value`.

When a new process is created, it *inherits* a copy of its **parent’s environment**.

This is a primitive but frequently used form of **IPC**.

The environment list of any process can be examined via the Linux-specific `/proc/PID/environ` file, with each `NAME=value` pair being terminated by a null byte.

### Accessing the environment from a program

Within a C program, the environment list can be accessed using the global variable `char **environ`.

```c
extern char **environ;
```

An alternative method of accessing the environment list is to declare a third argument to the `main()` function:

```c
int main(int argc, char *argv[], char *envp[])
```

**NOTE**: This method should be avoided since, in addition to the scope limitation, it is not specified in SUSv3.

Instead Use:

The `getenv()` function retrieves individual values from the process environment.

Note the following portability considerations when using `getenv()`:

- SUSv3 stipulates that an application should not modify the string returned by
`getenv()`. This is because (in most implementations) this string is actually part of the environment.

- SUSv3 permits an implementation of `getenv()` to return its result using a statically allocated buffer that may be **overwritten** by subsequent calls to `getenv()`, `setenv()`, `putenv()`, or `unsetenv().`

```c
char *getenv(const char * name );
```

### Modifying the environment

Use

```c
int putenv(char * string );
```

to modify environment.

The `setenv()` function is an alternative to `putenv()` for adding a variable to the environment.

```c
int setenv(const char * name , const char * value , int overwrite );
```

The `unsetenv()` function removes the variable identified by *name* from the environment.

```c
int unsetenv(const char * name );
```

On occasion, it is useful to erase the entire environment, and then rebuild it with
selected values

We can erase the environment by assigning `NULL` to `environ`.

```c
environ = NULL;
```

This is exactly the step performed by the `clearenv()` library function.

```c
int clearenv(void)
```

---

## Performing a Nonlocal Goto: `setjmp()` and `longjmp()`

The term **nonlocal** refers to the fact that the target of the *goto* is a location somewhere **outside** the currently executing function.

```c
int setjmp(jmp_buf env );
void longjmp(jmp_buf env , int val );
```

**NOTE**: Avoid `setjmp()` and `longjmp()` where possible.

### TODO

This section includes many advanced concepts which I wouldn't be able to fully absorb, therefore I left this part for a better time to be completed. :)

---

## END
