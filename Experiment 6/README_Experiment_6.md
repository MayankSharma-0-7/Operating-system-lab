# Experiment 6: Virtual Address Space, Demand Paging, and Physical RAM Consumption

## Aim

To observe the difference between virtual memory allocation and physical
RAM usage by dynamically allocating memory in a C program, accessing the
allocated memory page by page, and monitoring system memory statistics.

## Objectives

-   Allocate a user-specified amount of memory dynamically using
    `malloc()`.
-   Understand the difference between virtual address space size (VSZ)
    and resident set size (RSS).
-   Write to allocated memory at 4096-byte intervals.
-   Observe how accessing previously untouched memory causes page faults
    and can increase physical memory consumption.
-   Monitor system memory using `free -h`, `watch`, and
    process-monitoring commands.

------------------------------------------------------------------------

## 1. Concept: Virtual Memory and Demand Paging

When a process requests memory using `malloc()`, the C library requests
memory from the operating system. The allocation may reserve or map
virtual address space without immediately bringing every corresponding
page into physical RAM.

This behavior is associated with **demand paging**:

1.  The process requests a memory region.
2.  The region becomes part of the process's virtual address space.
3.  Physical pages may not be assigned to every part of the region
    immediately.
4.  When the program accesses an untouched page, a page fault occurs.
5.  The operating system handles the fault and makes a physical page
    available to back the accessed memory.

Two useful process memory measurements are:

-   **VSZ (Virtual Set Size):** the total virtual address space
    associated with a process.
-   **RSS (Resident Set Size):** the amount of the process's memory
    currently resident in physical RAM.

The exact behavior of allocation and memory accounting depends on the
operating system, C library, allocator, and system configuration. Linux
may also use zero-filled pages and other optimizations.

------------------------------------------------------------------------

## 2. C Program --- `memory.c`

The program below accepts the requested allocation size in megabytes,
allocates that amount of memory using `malloc()`, and writes one byte at
every 4096-byte interval. It then waits so that memory usage can be
inspected from another terminal.

``` c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

#define PAGE_SIZE 4096UL
#define BYTES_PER_MB (1024UL * 1024UL)

int main(int argc, char *argv[])
{
    if (argc != 2) {
        fprintf(stderr, "Usage: %s <memory_in_MB>\n", argv[0]);
        return 1;
    }

    char *endptr;
    unsigned long mb = strtoul(argv[1], &endptr, 10);

    if (*argv[1] == '\0' || *endptr != '\0' || mb == 0 ||
        mb > (unsigned long)-1 / BYTES_PER_MB) {
        fprintf(stderr, "Please enter a valid positive MB value.\n");
        return 1;
    }

    size_t bytes = (size_t)mb * BYTES_PER_MB;

    printf("Requesting %lu MB (%zu bytes)...\n", mb, bytes);

    unsigned char *memory = malloc(bytes);

    if (memory == NULL) {
        perror("malloc failed");
        return 1;
    }

    printf("Allocation completed at address %p.\n", (void *)memory);
    printf("Touching memory in %lu-byte increments...\n", PAGE_SIZE);

    for (size_t i = 0; i < bytes; i += PAGE_SIZE) {
        memory[i] = 1;
    }

    /* Also touch the final byte if the size is not page-aligned. */
    if (bytes % PAGE_SIZE != 0) {
        memory[bytes - 1] = 1;
    }

    printf("Memory pages touched. PID: %ld\n", (long)getpid());
    printf("Press Enter to release the memory and exit.\n");

    getchar();

    free(memory);
    printf("Memory released. Exiting.\n");

    return 0;
}
```

### Explanation

-   `malloc(bytes)` requests a memory block of the specified size.
-   The null check verifies that the allocation succeeded.
-   `PAGE_SIZE` is set to 4096 bytes for this experiment.
-   The loop writes one byte at each 4096-byte offset. This causes the
    program to access each corresponding page in the allocation.
-   The final-byte check touches the last page when the allocation size
    is not an exact multiple of 4096.
-   `getchar()` keeps the process alive while memory usage is observed.
-   `free(memory)` releases the allocation when the user presses Enter.

> **Page-size note:** 4096 bytes is common on Linux systems, but it is
> not universal. The actual base page size can be checked with
> `getconf PAGE_SIZE`. If your system reports a different page size,
> update `PAGE_SIZE` in the program accordingly.

------------------------------------------------------------------------

## 3. Compile and Run

Compile the program using GCC:

``` bash
gcc -Wall -Wextra -O2 memory.c -o memory
```

Run it with a memory allocation size, for example 200 MB:

``` bash
./memory 200
```

The program allocates the requested memory, touches it page by page,
prints its process ID, and waits for Enter.

You can try different sizes, such as:

``` bash
./memory 50
./memory 100
./memory 200
```

Choose a size that is safe for the available RAM and any memory limits
imposed on your environment.

------------------------------------------------------------------------

## 4. Monitor Memory Usage Using Two Terminals

Use two terminal windows so that the program can remain running while
memory statistics are monitored.

### Terminal 1: Run the program

``` bash
./memory 200
```

Keep this terminal open when the program displays the message asking you
to press Enter.

Note the PID printed by the program.

### Terminal 2: Monitor overall memory

Run:

``` bash
watch -n 1 free -h
```

This refreshes the system memory summary every second.

Useful columns include:

-   **total:** total recognized physical memory.
-   **used:** memory currently used, according to the tool's accounting.
-   **free:** completely unused memory.
-   **available:** an estimate of memory available for new applications
    without swapping.

`free -h` reports system-wide memory, not the VSZ and RSS of one
specific process. Other processes and Linux's memory cache can affect
these values.

### Monitor the program's VSZ and RSS

In Terminal 2, use the PID printed by the program:

``` bash
watch -n 1 "ps -o pid,comm,vsz,rss,%mem -p <PID>"
```

Replace `<PID>` with the actual numeric process ID. For example:

``` bash
watch -n 1 "ps -o pid,comm,vsz,rss,%mem -p 12345"
```

The `ps` output includes:

  Column   Meaning
  -------- ---------------------------------------------------
  `PID`    Process ID
  `COMM`   Command name
  `VSZ`    Virtual memory size, generally shown in KiB
  `RSS`    Resident memory size, generally shown in KiB
  `%MEM`   Percentage of physical memory used by the process

You can also inspect the process memory summary directly:

``` bash
grep -E 'VmSize|VmRSS' /proc/<PID>/status
```

Replace `<PID>` with the program's process ID.

------------------------------------------------------------------------

## 5. Expected Observation

Observe the process before and after it touches the allocated memory, if
you modify the program to pause immediately after `malloc()` and before
the page-touching loop.

Typical behavior on a demand-paging Linux system is:

1.  After `malloc()`, VSZ may increase to reflect the requested virtual
    allocation.
2.  RSS may remain comparatively low because the program has not yet
    accessed all of the allocated pages.
3.  As the program writes to one location in each page, page faults
    cause pages to become resident, and RSS generally increases.
4.  When the program calls `free()` and exits, its process memory is
    released back to the operating system.

The supplied program pauses after touching memory, so it is useful for
observing the post-touch RSS. To compare the pre-touch and post-touch
states directly, add a pause immediately after the successful `malloc()`
and before the loop, then remove that pause after recording the first
measurement.

### Observation table

Record values from your own system. Values will vary by machine and
operating-system configuration.

  -------------------------------------------------------------------------
  Stage                            VSZ                  RSS Observation
  --------------- -------------------- -------------------- ---------------
  Before                           ---                  --- Record baseline
  allocation                                                process memory

  After                            ---                  --- Virtual
  `malloc()`,                                               allocation may
  before touching                                           be visible
  pages                                                     

  After writing                    ---                  --- RSS generally
  at 4096-byte                                              increases
  intervals                                                 

  After `free()`                   ---                  --- Allocation is
                                                            released;
                                                            allocator
                                                            behavior may
                                                            affect
                                                            immediate
                                                            process
                                                            statistics
  -------------------------------------------------------------------------

------------------------------------------------------------------------

## 6. Important Commands

  --------------------------------------------------------------------------------
  Command                                      Purpose
  -------------------------------------------- -----------------------------------
  `gcc memory.c -o memory`                     Compiles the C program

  `./memory 200`                               Runs the program with a 200 MB
                                               allocation

  `getconf PAGE_SIZE`                          Displays the system's base page
                                               size

  `free -h`                                    Displays system memory in
                                               human-readable units

  `watch -n 1 free -h`                         Refreshes memory statistics every
                                               second

  `ps -o pid,comm,vsz,rss,%mem -p PID`         Displays selected process memory
                                               statistics

  `grep -E 'VmSize\|VmRSS' /proc/PID/status`   Displays virtual and resident
                                               memory fields for a process
  --------------------------------------------------------------------------------

------------------------------------------------------------------------

## 7. Safety and Notes

-   Do not request an excessively large allocation. It may cause heavy
    memory pressure, swapping, or termination by the system's
    out-of-memory manager.
-   `malloc()` returning a non-NULL pointer does not guarantee that all
    requested physical memory is already resident.
-   The operating system may use memory overcommit, so a large
    allocation can succeed initially and still fail to be fully backed
    later.
-   RSS is not always exactly equal to the amount of memory requested by
    the program. It includes other resident process memory, such as
    code, libraries, and stack.
-   `free -h` is a system-wide view. Use `ps` or `/proc/<PID>/status` to
    inspect a particular process.
-   The 4096-byte interval is a common base-page size, not a universal
    value.
-   The program writes one byte per page rather than filling every byte.
    This is sufficient to access each page in the allocation for the
    experiment.

## Conclusion

This experiment demonstrates the difference between a process's virtual
address space and its resident physical memory. A call to `malloc()` can
establish a virtual allocation without immediately making every page
resident. Writing to memory at page-sized intervals triggers page faults
and generally increases RSS as pages are brought into physical memory.
Using `free -h`, `watch`, `ps`, and `/proc` makes it possible to observe
system-wide and per-process memory behavior on Linux.
