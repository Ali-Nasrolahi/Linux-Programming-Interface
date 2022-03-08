# Notes from ***FUNDAMENTAL CONCEPTS***

## The Core Operating System: The Kernel

> The Kernel (in this context): refers to the **central** software that **manages** and **allocates** computer resources (i.e., the CPU, RAM, and devices).

> The Linux kernel executable typically resides at the pathname **/boot/vmlinuz** , or something similar.

### Kernel mode and user mode

When running in user mode, the CPU can access only memory that is marked as being in user space; attempts to *access memory* in *kernel space* result in a hardware **exception**.

Certain operations can be performed only while the processor is operating in
kernel mode.

> (for instance: executing the halt instruction to stop the system, accessing the memory-management hardware, and initiating device I/O operations.)

### Process versus kernel views of the system

- The kernel facilitates the running of *all* processes on the system.
- The kernel decides which *process* will next obtain access to the *CPU*, when it will do so, and for *how long*.

- The kernel **maintains** data structures containing **information** about all *running processes* and **updates** these structures as processes are created, **change state**, and **terminate**.

- The kernel **maintains** all of the low-level data structures that enable the **filenames** used by programs to be translated into **physical locations** on the disk.

- The kernel also maintains data structures that map the **virtual memory** of each process into the **physical memory** of the computer and the *swap area(s)* on disk

- All communication between processes is done via mechanisms provided by the kernel.
  
- In response to requests from processes, the kernel creates *new* processes and *terminates* existing processes.

- Lastly, the kernel (in particular, device drivers) performs **all direct communication** with *input* and *output* devices, transferring information to and from user processes as required

### The Shell

 A shell is a special-purpose program designed to *read* commands typed by a user and *execute* appropriate programs in response to those commands. (aka. *command interpreter*)

> login shell means the process that is created to run a shell when the user first logs in.

> On UNIX systems, the shell is a user process.

- *Bourne shell (sh)*: This is the oldest of the widely used shells, and was written by **Steve Bourne**. It was the standard shell for Seventh Edition UNIX.
  
- *C shell (csh)*: This shell was written by **Bill Joy** at the University of California at
Berkeley.

- *Korn shell (ksh)*: This shell was written as the successor to the *Bourne shell* by **David Korn** at AT&T Bell Laboratories.

- *Bourne again shell (bash)*: This shell is the **GNU** project’s reimplementation of
the *Bourne shell*. The principal authors of bash are **Brian Fox** and **Chet Ramey**. Bash is probably **the most** widely used shell on Linux.

### Users and Groups

#### Users

 Every user of the system has a **unique** login name (**username**) and a corresponding numeric **user ID** (UID).

 For each user, these are defined by a line in the system password file, */etc/passwd*.

 The **passwd** file includes;

- *Group ID*: the numeric group ID of the first of the groups of which the user is a member.
- *Home directory*: the initial directory into which the user is placed after logging in

- *Login shell*: the name of the program to be executed to interpret user commands.

For security reasons, the password is often stored in the separate **shadow password file**, which is readable only by privileged users.

#### Groups

For administrative purposes—in particular, for controlling access to files and other system resources—it is useful to organize users into groups.

The system group file, **/etc/group** , which includes the following information:

- *Group name*: the (unique) name of the group.
- *Group ID (GID)*: the numeric ID associated with this group.
- *User list*: a comma-separated list of **login names** of *users* who are members of this group (and who are not otherwise identified as members of the group by virtue of the group ID field of their password file record).

#### Superuser

- The superuser account has **UID 0**, and normally has the login name **root**.

### Single Directory Hierarchy, Directories, Links, and Files

The kernel maintains a single hierarchical directory structure to organize all files in the system.  

At the base of this hierarchy is the *root* directory, named **/** (slash). All files and directories are children or further removed descendants of the root directory.

#### File types

Within the file system, each file is marked with a *type*, indicating what kind of file it is.
These other file types other than *regular* or *plain* files; includes devices, pipes, sockets, directories, and symbolic links.

#### Directories and links

A **directory** is a special file whose contents take the form of a table of filenames coupled with references to the corresponding files.

This filename-plus-reference association is called a **link**.

#### Symbolic links

Like a normal link, a **symbolic link** provides an alternative name for a file.

But whereas a *normal link* is ai *filename-plus-pointer* entry in a directory list, a symbolic link is a specially *marked file* containing the *name* of **another file**.

> If a symbolic link refers to a file that doesn’t exist, it is said to be a *dangling* link.

> Often **hard link** and **soft link** are used as alternative terms for normal and symbolic links.

#### Filenames

- On most Linux file systems, filenames can be up to *255* characters(Bytes) long.

- Filenames may contain any characters except slashes ( **/** ) and null characters ( **\0** ).

- We should also avoid filenames beginning with a hyphen ( - ), since such filenames may be mistaken for options when specified in a shell command.

#### Pathnames

- An absolute pathname begins with a slash ( / ) and specifies the location of a file with respect to the root directory.

> Such as: /home/user/Wassup.txt

- A relative pathname specifies the location of a file relative to a process’s current working directory (see below), and is distinguished from an absolute pathname by the absence of an initial slash.

> Such as: ../root/RemoveMePls.dump

BTW: Don't forget that pathnames must stay in 4096 bytes limit. (this *includes* filenames)

#### File ownership and permissions

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

### File I/O Model

> Remember: UNIX systems **have no** *end-of-file* character; the end of a file is detected by a *read* that returns **no data**.

#### File descriptors

The I/O system calls refer to *open* files using a **file descriptor**, a (usually small) non-negative integer.

> A file descriptor is typically obtained by a call to open() (could be other syscall like socket(2) as well, which is based on open)

Normally, a process inherits three open file descriptors when it is started by
the shell:

- 0 which indicates **standard input**
- 1 which indicates **standard output**
- 2 which indicates **standard error**

***CAUTION***: These three FDs are intentionally stay open in child process, normally it's better that a child process **does not** inherit parent's FDs. (which is possible with **O_CLOEXEC** flag)

### Process

A process is an instance of an executing program.

States of a program:

1. When a program is executed, the kernel loads the code of the program into *virtual memory*,
2. Then Kernel allocates space for program variables.

3. And sets up kernel bookkeeping data structures to record various information about process
    > For instance: PID, termination status, UIDs, and GIDs

From a kernel point of view, processes are the entities among which the kernel must share the various resources of the computer.

For resources that are *limited*, such as memory:

 1. The kernel initially allocates some amount of the resource to the process

 2. And adjusts this allocation over the lifetime of the process in response to the demands of the process and the overall system demand for that resource.
