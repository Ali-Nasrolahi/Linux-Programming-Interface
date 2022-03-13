# Notes from ***FUNDAMENTAL CONCEPTS***

- [Notes from ***FUNDAMENTAL CONCEPTS***](#notes-from-fundamental-concepts)
  - [The Core Operating System: The Kernel](#the-core-operating-system-the-kernel)
  - [Kernel mode and user mode](#kernel-mode-and-user-mode)
    - [Process vs kernel views of the system](#process-vs-kernel-views-of-the-system)
  - [The Shell](#the-shell)
  - [Users and Groups](#users-and-groups)
    - [Users](#users)
    - [Groups](#groups)
    - [Superuser](#superuser)
  - [Single Directory Hierarchy, Directories, Links, and Files](#single-directory-hierarchy-directories-links-and-files)
    - [File types](#file-types)
    - [Directories and links](#directories-and-links)
    - [Symbolic links](#symbolic-links)
    - [Filenames](#filenames)
    - [Pathnames](#pathnames)
    - [File ownership and permissions](#file-ownership-and-permissions)
  - [File I/O Model](#file-io-model)
    - [File descriptors](#file-descriptors)
  - [Process](#process)
    - [Process memory layout](#process-memory-layout)
    - [Process creation and program execution](#process-creation-and-program-execution)
    - [Process termination and termination status](#process-termination-and-termination-status)
    - [Process user and group identifiers (credentials)](#process-user-and-group-identifiers-credentials)
    - [Privileged processes](#privileged-processes)
    - [The init process](#the-init-process)
    - [Daemon processes](#daemon-processes)
    - [Resource limits](#resource-limits)
  - [Memory Mappings](#memory-mappings)
    - [Application of memory mapping](#application-of-memory-mapping)
  - [Static and Shared Libraries](#static-and-shared-libraries)
    - [Static library](#static-library)
      - [Cons of using Static Libraries](#cons-of-using-static-libraries)
    - [Shared libraries](#shared-libraries)
      - [Pros of shared libraries](#pros-of-shared-libraries)
  - [Interprocess Communication and Synchronization](#interprocess-communication-and-synchronization)
  - [Signals](#signals)
  - [Threads](#threads)
    - [Features of threads](#features-of-threads)
    - [Advantages](#advantages)
  - [Process Groups and Shell Job Control](#process-groups-and-shell-job-control)
  - [Sessions, Controlling Terminals, and Controlling Processes](#sessions-controlling-terminals-and-controlling-processes)
    - [Controlling Terminal](#controlling-terminal)
  - [Pseudoterminals](#pseudoterminals)
    - [How does it works?](#how-does-it-works)
    - [Applications](#applications)
  - [Date and Time](#date-and-time)
  - [Client-Server Architecture](#client-server-architecture)
    - [Why do we use Client-Server apps](#why-do-we-use-client-server-apps)
  - [Realtime](#realtime)
  - [The `/proc` File System](#the-proc-file-system)
  - [END](#end)

## The Core Operating System: The Kernel

> The Kernel (in this context): refers to the **central** software that **manages** and **allocates** computer resources (i.e., the CPU, RAM, and devices).  
> The Linux kernel executable typically resides at the pathname **/boot/vmlinuz** , or something similar.

## Kernel mode and user mode

When running in user mode, the CPU can access only memory that is marked as being in user space; attempts to *access memory* in *kernel space* result in a hardware **exception**.

Certain operations can be performed only while the processor is operating in
kernel mode.

> (for instance: executing the halt instruction to stop the system, accessing the memory-management hardware, and initiating device I/O operations.)

### Process vs kernel views of the system

- The kernel facilitates the running of *all* processes on the system.
- The kernel decides which *process* will next obtain access to the *CPU*, when it will do so, and for *how long*.

- The kernel **maintains** data structures containing **information** about all *running processes* and **updates** these structures as processes are created, **change state**, and **terminate**.

- The kernel **maintains** all of the low-level data structures that enable the **filenames** used by programs to be translated into **physical locations** on the disk.

- The kernel also maintains data structures that map the **virtual memory** of each process into the **physical memory** of the computer and the *swap area(s)* on disk

- All communication between processes is done via mechanisms provided by the kernel.
  
- In response to requests from processes, the kernel creates *new* processes and *terminates* existing processes.

- Lastly, the kernel (in particular, device drivers) performs **all direct communication** with *input* and *output* devices, transferring information to and from user processes as required

---

## The Shell

 A shell is a special-purpose program designed to *read* commands typed by a user and *execute* appropriate programs in response to those commands. (aka. *command interpreter*)

> login shell means the process that is created to run a shell when the user first logs in.  
> On UNIX systems, the shell is a user process.

- *Bourne shell (sh)*: This is the oldest of the widely used shells, and was written by **Steve Bourne**. It was the standard shell for Seventh Edition UNIX.
  
- *C shell (csh)*: This shell was written by **Bill Joy** at the University of California at
Berkeley.

- *Korn shell (ksh)*: This shell was written as the successor to the *Bourne shell* by **David Korn** at AT&T Bell Laboratories.

- *Bourne again shell (bash)*: This shell is the **GNU** project’s reimplementation of
the *Bourne shell*. The principal authors of bash are **Brian Fox** and **Chet Ramey**. Bash is probably **the most** widely used shell on Linux.

---

## Users and Groups

### Users

 Every user of the system has a **unique** login name (**username**) and a corresponding numeric **user ID** (UID).

 For each user, these are defined by a line in the system password file, */etc/passwd*.

 The **passwd** file includes;

- *Group ID*: the numeric group ID of the first of the groups of which the user is a member.
- *Home directory*: the initial directory into which the user is placed after logging in

- *Login shell*: the name of the program to be executed to interpret user commands.

For security reasons, the password is often stored in the separate **shadow password file**, which is readable only by privileged users.

### Groups

For administrative purposes—in particular, for controlling access to files and other system resources—it is useful to organize users into groups.

The system group file, **/etc/group** , which includes the following information:

- *Group name*: the (unique) name of the group.
- *Group ID (GID)*: the numeric ID associated with this group.
- *User list*: a comma-separated list of **login names** of *users* who are members of this group (and who are not otherwise identified as members of the group by virtue of the group ID field of their password file record).

### Superuser

- The superuser account has **UID 0**, and normally has the login name **root**.

---

## Single Directory Hierarchy, Directories, Links, and Files

The kernel maintains a single hierarchical directory structure to organize all files in the system.  

At the base of this hierarchy is the *root* directory, named **/** (slash). All files and directories are children or further removed descendants of the root directory.

### File types

Within the file system, each file is marked with a *type*, indicating what kind of file it is.
These other file types other than *regular* or *plain* files; includes devices, pipes, sockets, directories, and symbolic links.

### Directories and links

A **directory** is a special file whose contents take the form of a table of filenames coupled with references to the corresponding files.

This filename-plus-reference association is called a **link**.

### Symbolic links

Like a normal link, a **symbolic link** provides an alternative name for a file.

But whereas a *normal link* is ai *filename-plus-pointer* entry in a directory list, a symbolic link is a specially *marked file* containing the *name* of **another file**.

> If a symbolic link refers to a file that doesn’t exist, it is said to be a *dangling* link.  
> Often **hard link** and **soft link** are used as alternative terms for normal and symbolic links.

### Filenames

- On most Linux file systems, filenames can be up to *255* characters(Bytes) long.

- Filenames may contain any characters except slashes ( **/** ) and null characters ( **\0** ).

- We should also avoid filenames beginning with a hyphen ( - ), since such filenames may be mistaken for options when specified in a shell command.

### Pathnames

- An absolute pathname begins with a slash ( / ) and specifies the location of a file with respect to the root directory.

> Such as: /home/user/Wassup.txt

- A relative pathname specifies the location of a file relative to a process’s current working directory, and is distinguished from an absolute pathname by the absence of an initial slash.

> Such as: ../root/RemoveMePls.dump

BTW: Don't forget that pathnames must stay in 4096 bytes limit. (this *includes* filenames)

### File ownership and permissions

 Each file has an associated user ID and group ID that define the owner of the file and the group to which it belongs.

For the purpose of accessing a file, the system divides users into three categories:

- The owner of the file (*user*)

- Users who are members of the group matching the file’s group ID (*group*)

- The rest of the world (*other*)

**Three** permission bits may be set for each of these categories of use (total of 9 bits)

- **Read permission**: allows the contents of the file to be read.

- **Write permission**: allows modification of the contents of the file;

- **Execute permission**: allows execution of the file.

Meaning of these are slightly different for directories

- **Read permission**: allows the contents of (i.e., the filenames in) the directory to be *listed*.

- **Write permission**: allows the contents of the directory to be changed (i.e., filenames can be added, removed, and changed).

- **Execute permission**: allows access to files within the directory (subject to the permissions on the files themselves).

---

## File I/O Model

> Remember: UNIX systems **have no** *end-of-file* character; the end of a file is detected by a *read* that returns **no data**.

### File descriptors

The I/O system calls refer to *open* files using a **file descriptor**, a (usually small) non-negative integer.

> A file descriptor is typically obtained by a call to `open()` (could be other syscall like `socket(2)` as well, which is based on open)

Normally, a process inherits three open file descriptors when it is started by
the shell:

- 0 which indicates **standard input**
- 1 which indicates **standard output**
- 2 which indicates **standard error**

***CAUTION***: These three FDs are intentionally stay open in child process, normally it's better that a child process **does not** inherit parent's FDs. (which is possible with **O_CLOEXEC** flag)

---

## Process

A process is an instance of an executing program.

States of a program:

1. When a program is executed, the kernel loads the code of the program into *virtual memory*,
2. Then Kernel allocates space for program variables.

3. And sets up kernel bookkeeping data structures to record various information about process
    > For instance: PID, termination status, UIDs, and GIDs

From a kernel point of view, processes are the entities among which the kernel must share the various resources of the computer.:

For resources that are *limited*, such as memory:

 1. The kernel initially allocates some amount of the resource to the process

 2. And adjusts this allocation over the lifetime of the process in response to the demands of the process and the overall system demand for that resource.

### Process memory layout

*segments*: are parts of each process which logically divided to. Divisions are as follows:

- **Text**: the instructions of the program.
- **Data**: the static variables used by the program.
- **Heap**: an area from which programs can dynamically allocate extra memory.
- **Stack**: a piece of memory that grows and shrinks as functions are called and return.

### Process creation and program execution

> A parent process can create a child process using `fork(2)`; the kernel creates the child process by making a duplicate of the parent.  
> And the child inherits copies of the parent’s *data*, *stack*, and *heap* segments, which it may then modify **independently** of the parent’s copies.

**NOTE:** The program text segment, which is placed in memory marked as **read-only**, is **shared** by the two processes.

### Process termination and termination status

A process can terminate in one of two ways:

1. By *requesting* its own **termination** using the `_exit()` system call (or the related `exit()` library function)

2. Or by being killed by the delivery of a **signal**.

**Remember:** System calls and C library functions are 2 different things.
> `_exit()` the syscall and `exit()` in C lib ----->> **NOT SAME**.  
> Usually C libs' functions ,that works like a syscall, are only wrappers which call actual syscall.

*termination status*: is a small nonnegative integer value which will be examined by parent process using `wait()` syscall.

> termination status by `_exit()`: the process explicitly specifies its own termination status  
> termination status by a signal: status sets according to type of signal which caused termination.

### Process user and group identifiers (credentials)

Each process has a number of associated UIDs and GIDs.

Which includes:

- *Real* UID and *real* GID: These identify the user and group to which the process belongs.
  - > A new process inherits these IDs from its parent.
  - > A login shell gets its real user ID and real group ID from the corresponding fields in the system password file.

- *Effective* UID and *effective* GID: These two IDs (in conjunction with supplementary GIDs) are used in determining the permissions that the process has when accessing protected resources.
  - > Typically, the process’s effective IDs have the same values as the corresponding real IDs.
  - > Changing the effective IDs is a mechanism that allows a process to assume the privileges of another user or group.

- *Supplementary* GIDs: These IDs identify additional groups to which a process belongs.
  - > A new process inherits its supplementary group IDs from its parent.
  - > A login shell gets its supplementary group IDs from the system group file.

### Privileged processes

Traditionally, on UNIX systems, a privileged process is one whose *effective* UID is 0 (superuser).

Another way a process may become privileged is via the set-user-ID mechanism.

- > which allows a process to assume an effective user ID that is the same as the user ID of the program file that it is executing.

### The init process

When booting the system, the kernel creates a special process called init, the “**parent of all processes**”.

init features are as follows:

- init process derived from */sbin/init*
- All processes on the system are created either by *init* or by one of *its descendants*.
- The init process always has the PID 1 and runs with superuser privileges.

- The init process **can’t be killed** (not even by the superuser), and it terminates *only* when the system is *shut down*.

- The main task of init is to **create** and **monitor** a range of processes required by a running system.

### Daemon processes

A daemon process has these attributes:

1. It is long-lived: A daemon process is often started at **system boot** and *remains* in existence until the system is *shut down*.

2. It runs in the background, and has **no controlling terminal** from which it can read input or to which it can write output.

### Resource limits

Using the `setrlimit()` system call, a process can establish upper limits on its consumption of various resources

- Soft Limit: limits the amount of the resource that the process may consume.
- Hard Limit: is a ceiling on the value to which the soft limit may be adjusted.

---

## Memory Mappings

The `mmap()` system call creates a new *memory mapping* in the calling process’s virtual address space.

Mappings devides into:

1. **File mapping**: maps a region of a file into the calling process’s virtual memory.  
    > Once mapped, the file’s contents can be accessed by operations on the bytes in the corresponding memory region.  
    > The pages of the mapping are automatically loaded from the file as required.

2. **Anonymous mapping**: doesn’t have a corresponding file. Instead, the pages of the mapping are initialized to 0.

The memory in one process’s mapping may be shared with mappings in other processes.

In two ways:

1. 2 processes map same region of a file.

2. A child process created by `fork()` inherits a mapping from its parent.

When multiple processes share the same pages, each process may see the changes made by *other* processes dependi*ng on whether the mapping is created as **private** or **shared**.

### Application of memory mapping

Some of memory mapping are as follows:

- Initialization of a *process’s* **text segment** from the corresponding segment of an *executable* file.
- Allocation of new (zero-filled) memory.
- File I/O (memory-mapped I/O).
- IPC (interprocess communication) via a shared mapping.

---

## Static and Shared Libraries

**Object library**: is a file containing the compiled object code for a set of functions that may be called from application programs.

Modern UNIX systems provide two types of object libraries: *static* and *shared* libraries.

### Static library

A static library is essentially a structured bundle of compiled object modules.

> Note: static lib (aka. *archives*) were the only type of library on early UNIX systems.

To use functions from a static library, we specify that library in the *link* command used to build a program. (must be added for linker)

**Question**: How does linker deal with static libs?

After resolving the various function references from the main program to the modules in the static library, the linker extracts *copies* of the required object modules from the library and *copies these into* the resulting **executable file** (such program called ***staticlly linked***)

> **Note**: Objects will be added to executable file.

#### Cons of using Static Libraries

The fact that each statically linked program includes its **own copy** of the object modules required from the library creates a number of disadvantage.

1. **Duplication** of object code in different executable files wastes disk space.
    > A corresponding waste of memory occurs when statically linked programs using the same library function are executed at the same time; each program requires its own copy of the function to reside in memory.

2. In case of library **modification**, requirement of *recompiling* and *relinking* all programs, is unwise and costly.

### Shared libraries

> Shared libraries were designed to address the problems with static libraries.

**Question**: How does linker deals with shared libs?

Instead of copying object modules from the library into the executable,the linker just *writes* a record into the executable to indicate that at *run time* the executable needs to use that *shared library*.

When the executable is loaded into memory at run time, a program called the **dynamic linker** ensures that the shared libraries required by the executable are found and loaded into memory.  
And performs **run-time** linking to resolve the function calls in the executable to the corresponding definitions in the shared libraries.

#### Pros of shared libraries

1. At run time, only **a single** copy of the code of the shared library needs to be resident in memory.

2. All **running programs** can use that copy.

3. The fact that a shared library contains the sole compiled version of a function *saves* disk space.

4. It also greatly eases the job of ensuring that programs employ the **newest version** of a function.

5. Simply *rebuilding* the shared library with the new function definition causes existing programs to **automatically** use the **new definition** when they are *next* executed.

---

## Interprocess Communication and Synchronization

Some processes, however, cooperate to achieve their intended purposes, and these processes need methods of **communicating** with one another(IPC) and **synchronizing** their actions.

One way for processes to communicate is by reading and writing information in disk files. However, for many applications, this is *too slow* and *inflexible*.

Some of approaches for IPC in modern UNIX implementation:

- **Signals**: are used to indicate that an event has occurred.

- **Pipes**: and *FIFOs*, which can be used to transfer data between processes.

- **Sockets**: can be used to transfer data from one process to another, either on the *same* host computer or on **different** hosts connected by a network.

- **File locking**: allows a process to *lock regions* of a *file* in order to prevent other processes from **reading** or **updating** the file contents.

- **Message queues**: are used to exchange messages (packets of data) between processes.

- **Semaphores**: are used to synchronize the actions of processes.

- **Shared memory**: allows two or more processes to share a piece of memory.  
  > When one process changes the contents of the shared memory, all of the other processes can immediately see the changes.

---

## Signals

Signals are often described as “**software interrupts**.”

> The arrival of a signal informs a process that some event or exceptional condition has occurred.

Each signal type is identified by a different integer, defined with symbolic names of the form **SIGxxxx**. And has specific **meaning** and **consequences**.

Signals are sent to a process by the kernel, by **another process** (with suitable permissions), or by the process **itself**.

Within the shell, the `kill` command can be used to send a signal to a process. The `kill()` system call provides the same facility within programs.

On receiving a signal, one of the below action can happens:

- Process ignores the signal;
- it is killed by the signal; or
- it is suspended until later being resumed by receipt of a special-purpose signal.

For most signal types, instead of accepting the default signal action, a program can choose to ignore the signal or to establish a **signal handler**.

In the interval between the time it is generated and the time it is delivered, a signal is said to be **pending** for a *process*.

Normally, a pending signal is delivered **as soon as** the receiving process is *next scheduled* to run, or *immediately* if the process is already running.

It is also possible to **block** a signal by adding it to the process’s *signal mask*  
If a signal is generated while it is blocked, it *remains* pending until it is later *unblocked* (i.e., removed from the signal mask).

---

## Threads

### Features of threads

1. Threads are a set of processes that share the same **virtual memory**, as well as a range of other attributes.
    > Threads and Process are different. google `Threads vs Process` to find out more.

2. Each thread is executing the same program code and *shares* the same data area and **heap**.

3. Each thread has it own **stack** containing *local variables* and *function call* linkage information.

**Question**: How threads talk to each other?

1. Via the **global** variables that they share.

2. The threading API provides **condition variables** and **mutexes**.
    > which are primitives that enable the threads of a process to communicate and synchronize their actions, in particular, their use of shared variables.

3. Threads can also communicate with one another using the IPC and synchronization mechanisms.

### Advantages

Primary **advantages** of using threads are:

- hey make it easy to share data (via global variables)

- A multithreaded application can transparently take advantage of the possibilities for parallel processing on multiprocessor hardware.

---

## Process Groups and Shell Job Control

Each program executed by the shell is started in a new process.

**Job Control**: allows the user to simultaneously execute and manipulate multiple commands or pipelines.

In job-control shells, all of the processes in a **pipeline** are placed in a new *process group* or *job*.

Each process in a process group has the **same** integer process *group identifier*, which is the same as the *process ID* of one of the processes in the group, termed **the process group leader**.

> The kernel allows for various actions, notably the **delivery of signals**, to be performed on all members of a process group

---

## Sessions, Controlling Terminals, and Controlling Processes

**session**: is a collection of process groups (*jobs*)

All of the processes in a *session* have the same **session identifier**.

**session leader**: is the process that created the session, and its *process ID* becomes the *session ID*.

### Controlling Terminal

The **controlling terminal** is established when the *session leader* process first opens a terminal device.

> For a session created by an interactive shell, this is the terminal at which the user logged in.

As a consequence of opening the controlling terminal, the session leader becomes the **controlling process** for the terminal.  
The controlling process receives a `SIGHUP` signal if a terminal disconnect occurs

At any point in time, **one** process group in a session is the *foreground process group* (foreground job), which may *read input* from the terminal and *send output* to it.

> If the user types the interrupt character (usually Control-C) or the suspend character (usually Control-Z) on the controlling terminal, then the terminal driver sends a signal that kills or suspends (i.e., stops) the foreground process group.

## Pseudoterminals

**pseudoterminal**: is a pair of connected *virtual* devices, known as the *master* and *slave*.

This device pair provides an **IPC channel** allowing data to be transferred in **both** directions between the two devices.

### How does it works?

- **slave** device provides an interface that behaves like *a terminal*, which makes it possible to connect a *terminal-oriented program* to the slave device and then use *another program* connected to the **master** device to drive the terminal-oriented program.

- *Output* written by the **driver program** undergoes the usual input processing performed by the terminal driver.

- And is then passed as *input* to the terminal-oriented program connected to the **slave**.

- Anything that the terminal-oriented program writes to the slave is passed as *input* to the *driver program*.

### Applications

Most notably in the implementation of terminal windows provided under an X Window System login and in applications providing network login services,
such as **telnet** and **ssh**.

---

## Date and Time

Two types of time are of interest to a process:

1. **Real time**: is measured either from some standard point (*calendar* time) or from some fixed point, typically the start, in the life of a process (*elapsed* or *wall* clock time).
    > On UNIX systems, *calendar* time is measured in **seconds** since midnight on the morning of *January 1, 1970 UTC*.  
    > This date, which is close to the birth of the UNIX system, is referred to as the ***Epoch***.

2. **Process time** (aka. CPU time): s the total amount of CPU time that a process has used since starting.
     - **system CPU time**: the time spent executing code in **kernel** mode.
     - **user CPU time**: the time spent executing code in **user** mode.

---

## Client-Server Architecture

A *client-server* application is one that is broken into two component processes:

- **client**, which asks the server to carry out some service by sending it a request message.

- **server**, which examines the client’s request, performs appropriate actions, and then sends a response message back to the client.

### Why do we use Client-Server apps

- *Efficiency*: It may be cheaper to provide one instance of a resource (e.g., a printer) that is managed by a server than to provide the same resource locally on every computer.

- *Control, coordination, and security*: By holding a resource at a single location, the server can coordinate access to the resource or secure it so that it is made available to only selected clients.

- *Operation in a heterogeneous environment*: In a network, the various clients, and the server, can be running on different hardware and operating system platforms.

--- 

## Realtime

**Realtime** applications are those that need to respond in a timely fashion to input.

Although many realtime applications require *rapid* responses to input, the defining factor is that the response is guaranteed to be delivered within a certain **deadline** time after the *triggering event*.

*POSIX.1b* defined a number of extensions to *POSIX.1* for the support of realtime applications.

> These include asynchronous I/O, shared memory, memory-mapped files, memory locking, realtime clocks and timers, alternative scheduling policies, realtime signals, message queues, and semaphores.

--- 

## The `/proc` File System

**/proc** file system: is a **virtual** file system that provides an interface to *kernel* **data structures** in a form that looks like *files* and *directories* on a file system.

In addition, a set of directories with names of the form `/proc/PID` allows us to view information about **each process** *running* on the system.

---
## END

