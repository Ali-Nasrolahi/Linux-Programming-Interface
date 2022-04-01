# Notes From ***MEMORY ALLOCATION***

- [Notes From ***MEMORY ALLOCATION***](#notes-from-memory-allocation)
  - [Allocating Memory on the Heap](#allocating-memory-on-the-heap)
    - [Adjusting the Program Break: `brk()` and `sbrk()`](#adjusting-the-program-break-brk-and-sbrk)
    - [Allocating Memory on the Heap: `malloc()` and `free()`](#allocating-memory-on-the-heap-malloc-and-free)
      - [To `free()` or not to `free()`?](#to-free-or-not-to-free)
    - [Implementation of `malloc()` and `free()`](#implementation-of-malloc-and-free)
      - [`malloc()` implementation](#malloc-implementation)
      - [`free()` implementation](#free-implementation)
      - [Tools and libraries for `malloc` debugging](#tools-and-libraries-for-malloc-debugging)
    - [Other Methods of Allocating Memory on the Heap](#other-methods-of-allocating-memory-on-the-heap)
      - [Allocating aligned memory: `memalign()` and`posix_memalign()`](#allocating-aligned-memory-memalign-andposix_memalign)
  - [Allocating Memory on the Stack: `alloca()`](#allocating-memory-on-the-stack-alloca)

## Allocating Memory on the Heap

The current limit of the heap is referred to as the *program break*.

### Adjusting the Program Break: `brk()` and `sbrk()`

Initially, the program break lies just past the end of the *uninitialized data* segment.

After the program break is increased, the program may access any address in the newly allocated area, but **no physical memory pages** are *allocated* yet.
> The kernel automatically allocates new physical pages on the first attempt by the process to access addresses in those pages.

The UNIX system has provided two system calls for manipulating the program break, `brk()` and `sbrk()`.

```c
int brk(void * end_data_segment);

void *sbrk(intptr_t increment);
```

The call `sbrk(0)` returns the **current** setting of the program break **without** changing it.
> This can be useful if we want to track the size of the heap.

### Allocating Memory on the Heap: `malloc()` and `free()`

```c
void *malloc(size_t size );

void free(void * ptr );
```

> In general, `free()` **doesn’t** lower the program break, but instead **adds** the block of memory to a list of *free blocks* that are recycled by future calls to `malloc()`

#### To `free()` or not to `free()`?

When a process terminates, all of its memory is returned to the system, including heap memory allocated by functions in the malloc package.

Although relying on process termination to *automatically* free memory is acceptable for many programs, there are a couple of reasons why it can be desirable to **explicitly** free all allocated memory:

- Explicitly calling `free()` may make the program more *readable* and *maintainable* in the face of future modifications.

- If we are using a *malloc* debugging library too find memory leaks in a program, then any memory that is not **explicitly** freed will be reported as a memory leak. This can complicate the task of finding **real** memory leaks.

### Implementation of `malloc()` and `free()`

#### `malloc()` implementation

The implementation of `malloc()` is straightforward.

1. It first scans the list of memory blocks **previously** released by `free()` in order to find one whose size is *larger* than or *equal* to its requirements.  
If no block on the free list is large enough

2. Then `malloc()` calls `sbrk()` to allocate more memory.

#### `free()` implementation

When `free()` places a block of memory onto the free list.
> free list is a doubly linked list

**Q:** How does it know what size that block is?

Basically, each entries in free list contains 3 additional blocks to actual requested memory.

1. Length.
    > length of the requested block of memory (i.e., 4 btyes)
2. pointer to previous block.
3. pointer to next block.

which is set by `malloc()`.

#### Tools and libraries for `malloc` debugging

- The `mtrace()` and `muntrace()` functions allow a program to turn tracing of memory allocation calls *on* and *off*
    > These functions are used in conjunction with the ``MALLOC_TRACE environment variable, which should be defined to contain the name of a file to which tracing information should be written.

- The `mcheck()` and `mprobe()` functions allow a program to perform consistency checks on blocks of allocated memory.
    > for example, catching errors such as attempting to write to a location past the end of a block of allocated memory.  
    Programs that employ these functions must be linked with the `mcheck` library using the `cc –lmcheck` option.
- The `MALLOC_CHECK_` environment variable serves a similar purpose to `mcheck()` and`mprobe().` By setting this variable to *different* integer values, we can control how a program responds to memory allocation errors.
  > One notable difference between the two techniques is that using `MALLOC_CHECK_` doesn’t require modification and recompilation of the program.

**NOTE**: Further information about all of the above features can be found in the `glibc` manual.

### Other Methods of Allocating Memory on the Heap

The `calloc()` function allocates memory for an array of identical items.

```c
void *calloc(size_t numitems , size_t size );
```

> Unlike `malloc(),` `calloc()` **initializes** the allocated memory to **0**.

The `realloc()` function is used to resize a block of memory previously allocated by one of the functions in the *malloc* package.

```c
void *realloc(void * ptr , size_t size );
```

#### Allocating aligned memory: `memalign()` and`posix_memalign()`

The `memalign()` and `posix_memalign()` functions are designed to allocate memory starting at an address aligned at a specified *power-of-two* boundary,

```c
void *memalign(size_t boundary , size_t size );
```

```c
int posix_memalign(void ** memptr , size_t alignment , size_t size );
```

---

## Allocating Memory on the Stack: `alloca()` 