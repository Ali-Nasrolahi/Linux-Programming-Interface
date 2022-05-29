# Notes from ***HISTORY AND STANDARDS***

- [Notes from ***HISTORY AND STANDARDS***](#notes-from-history-and-standards)
  - [A Brief History of UNIX and C](#a-brief-history-of-unix-and-c)
    - [Dates 1969-1973](#dates-1969-1973)
  - [The birth of BSD and System V](#the-birth-of-bsd-and-system-v)
    - [Dates 1979-1983](#dates-1979-1983)
  - [A Brief History of Linux](#a-brief-history-of-linux)
    - [The GNU Project](#the-gnu-project)
    - [Linux](#linux)
  - [Standardization](#standardization)
    - [The First POSIX Standards](#the-first-posix-standards)
    - [SUSVx](#susvx)
      - [SUSv3 and POSIX.1-2001](#susv3-and-posix1-2001)
      - [SUSv3](#susv3)
      - [SUSv4 and POSIX.1-2008](#susv4-and-posix1-2008)
  - [END](#end)

## A Brief History of UNIX and C

### Dates 1969-1973

- 1969 born of **UNIX** by **Ken Thompson** and **Dennis Ritchie** an early collaborator on UNIX.

- C programming language was developed in *1972* by Dennis Ritchie at bell laboratories of AT&T

- Third Edition, February *1973*: This edition included a C compiler and the first implementation of pipes

- Fourth Edition, November *1973*: This was the first version to be almost totally written in C. (*almost a year after 3rd edition*)

## The birth of BSD and System V

### Dates 1979-1983

- January *1979* UNIX 7th and **beginning** of BSD and SysV

> Note that *3BSD* was released in December *1979* (not the first release but important one)

- In *1983*, the Computer Systems Research Group at the University of California at Berkeley released *4.2BSD*.

> 4.2BSD is significant because it contained a complete TCP/IP implementation, including the sockets application programming interface (API) and a variety of networking tools.  
> Other significant BSD releases were 4.3BSD, in 1986, and the final release, 4.4BSD, in 1993

- The first release of **System V** (five) followed in *1983*, and a series of releases led to the *definitive System V Release 4* (SVR4) in *1989*

## A Brief History of Linux

### The GNU Project

- In *1984*, **Richard Stallman** set to work on creating a **"free"** UNIX implementation

  - > **Extreme Respect** to one of the most influential individual, I know.

### Linux

- In *1991*, **Linus Torvalds** was inspired to write an operating system for his Intel 80386 PC.

- *Minix* a small UNIX-like operating system kernel developed in the *mid-1980s* by **Andrew Tanenbaum**. it was designed to be largely independent of the hardware architecture, and it did not take full advantage of the 386 processor’s capabilities.

- *October 5, 1991*, Torvalds requested the help of other programmers for development of linux kernel

- Early 1990s. **Bill and Lynne Jolitz** had developed a port of the already mature BSD system for the x86-32, known as *386/BSD*.

## Standardization

### The First POSIX Standards

POSIX: Portable Operating System Interface

- refers to a group of standards developed under the auspices of *IEEE*

### SUSVx

- *XPG4* version 2 was subsequently repackaged as the *Single UNIX Specification* (SUS, or sometimes SUSv1)

- Version 2 of the Single UNIX Specification (*SUSv2*) appeared in *1997*

#### SUSv3 and POSIX.1-2001

- Beginning in *1999*, the *IEEE*, *The Open Group*, and the *ISO/IEC Joint Technical Committee 1* collaborated in the Austin Common Standards Revision Group (CSRG) POSIX and SUS standards .

- *POSIX 1003.1-2001* replaces SUSv2, POSIX.1, POSIX.2. This standard is also known as the *SUSv3*.

#### SUSv3

The SUSv3 base specifications divided into the following four parts:

- > Base Definitions (XBD): This part contains definitions, terms, concepts, and specifications of the contents of header files. A total of 84 header file specifications are provided.

- > System Interfaces (XSH): This part begins with various useful background information. Its bulk consists of the specification of various functions (which are implemented as either system calls or library functions on specific UNIX implementations). A total of 1123 system interfaces are included in this part.

- > Shell and Utilities (XCU): This specifies the operation of the shell and various UNIX commands. A total of 160 utilities are specified in this part.

- > Rationale (XRAT): This part includes informative text and justifications relating to the earlier parts.

#### SUSv4 and POSIX.1-2008

- In *2008*, the Austin group completed a revision of the combined POSIX.1 and Single UNIX Specification.

---

## END
