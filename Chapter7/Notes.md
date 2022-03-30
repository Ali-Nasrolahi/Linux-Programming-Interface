# Notes From ***MEMORY ALLOCATION***

- [Notes From ***MEMORY ALLOCATION***](#notes-from-memory-allocation)
  - [Allocating Memory on the Heap](#allocating-memory-on-the-heap)
    - [Adjusting the Program Break: `brk()` and `sbrk()`](#adjusting-the-program-break-brk-and-sbrk)
    - [Allocating Memory on the Heap: `malloc()` and `free()`](#allocating-memory-on-the-heap-malloc-and-free)
      - [To `free()` or not to `free()`?](#to-free-or-not-to-free)
    - [Implementation of `malloc()` and `free()`](#implementation-of-malloc-and-free)
      - [`malloc()` implementation](#malloc-implementation)
      - [`free()` implementation](#free-implementation)

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

When `free()` places a block of memory onto the free list,

**Q:** How does it know what size that block is?

Basically, each entries in free list contains 3 additional blocks to actual requested memory.

1. Length.
2. pointer to previous block.
3. pointer to next block.

which is set by `malloc()`.