# Notes From ***PROCESSES***

- [Notes From ***PROCESSES***](#notes-from-processes)
  - [Processes and Programs](#processes-and-programs)
  - [Process ID and Parent Process ID](#process-id-and-parent-process-id)
  - [Memory Layout of a Process](#memory-layout-of-a-process)

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

## Memory Layout of a Process
