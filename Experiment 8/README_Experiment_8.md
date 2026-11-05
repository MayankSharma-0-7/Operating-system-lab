# Experiment 8: Virtual Memory Structure, Shared Memory, and Page Permissions

## Aim

To inspect the virtual memory layout of active Linux processes using
`pmap -x`, identify important memory regions, understand page
permissions, and examine the relationship between mapping size, resident
memory, and dirty pages.

## Objectives

-   Inspect the memory mappings of an active process using `pmap -x`.

-   Understand the six columns: Address, Kbytes, RSS, Dirty, Mode, and
    Mapping.

-   Identify executable text segments, shared libraries, heap, and
    stack.

-   Interpret memory permission strings such as `r-xp` and `rw-p`.

-   Check the memory accounting relationship:

    \[ `\text{Dirty}`{=tex} `\leq `{=tex}`\text{RSS}`{=tex}
    `\leq `{=tex}`\text{Kbytes}`{=tex} \]

------------------------------------------------------------------------

## 1. Concept: Virtual Memory Mapping

Linux gives each process its own virtual address space. This address
space is divided into memory mappings that may represent program code,
data, dynamically allocated memory, stacks, shared libraries,
memory-mapped files, and other regions.

A mapping does not necessarily occupy an equal amount of physical RAM.
Some of its pages may not have been accessed, may be shared with other
processes, or may be backed by files.

The `pmap` utility displays a process's memory map. Its extended mode,
`-x`, provides additional memory-usage information.

## 2. Select an Active Process

First, list running processes and identify a process ID (PID):

``` bash
ps aux
```

Alternatively, search for a specific process:

``` bash
pgrep -a bash
```

Choose a process that remains active while you inspect it. Record its
PID.

> Use a process that you own or are permitted to inspect. Some process
> details may be restricted by system permissions.

------------------------------------------------------------------------

## 3. Display the Extended Memory Map

Run:

``` bash
pmap -x <PID>
```

Replace `<PID>` with the actual numeric process ID. For example:

``` bash
pmap -x 1234
```

The output typically contains a table with these six columns:

  -----------------------------------------------------------------------
  Column                              Meaning
  ----------------------------------- -----------------------------------
  `Address`                           Starting virtual address of the
                                      mapping

  `Kbytes`                            Size of the mapping in kilobytes

  `RSS`                               Resident Set Size: memory from the
                                      mapping currently resident in RAM,
                                      in kilobytes

  `Dirty`                             Dirty resident memory associated
                                      with the mapping, in kilobytes

  `Mode`                              Mapping permissions and
                                      sharing/private indicator

  `Mapping`                           Name or description of the mapped
                                      object, such as a library, file,
                                      heap, or stack
  -----------------------------------------------------------------------

A final `total` row may summarize the process's mappings. The exact
output and labels can vary slightly by Linux distribution and `procps`
version.

------------------------------------------------------------------------

## 4. Identify Important Memory Regions

### A. Text segment (program code)

The text segment contains executable program instructions. It is
commonly mapped with permissions such as:

``` text
r-xp
```

-   `r`: readable
-   `-`: not writable
-   `x`: executable
-   `p`: private mapping

The `r-xp` permission pattern means the mapping is readable and
executable, but not writable, and is privately mapped.

### B. Shared libraries

Shared libraries provide reusable code used by programs. Examples
include the C standard library, often shown with a name similar to:

``` text
/usr/lib/.../libc.so.6
```

A shared library can appear in several mappings with different
permissions. For example:

-   `r--p`: read-only data or metadata
-   `r-xp`: executable code
-   `rw-p`: writable data

The same library file may therefore occupy multiple rows in the memory
map.

### C. Heap

The heap is used for dynamic memory allocation, for example through
`malloc()` in C.

It may appear with a mapping label such as:

``` text
[heap]
```

The heap is commonly writable and non-executable, often with permissions
like:

``` text
rw-p
```

Not all dynamically allocated memory must appear in the traditional
`[heap]` mapping. Large allocations may be served through separate
anonymous mappings, depending on the allocator and allocation size.

### D. Stack

The stack stores function-call information, local variables, and related
execution state. It may appear as:

``` text
[stack]
```

The stack is typically readable and writable, commonly with permissions
like:

``` text
rw-p
```

Threads can have additional stack mappings, and their names or
presentation may differ by kernel and tool version.

------------------------------------------------------------------------

## 5. Understand the Mode / Permission Field

The `Mode` column describes the access permissions and mapping type.

  Character   Meaning
  ----------- ---------------------------------
  `r`         Read permission
  `w`         Write permission
  `x`         Execute permission
  `-`         Permission not granted
  `p`         Private (copy-on-write) mapping
  `s`         Shared mapping

Examples:

  Mode     Interpretation
  -------- ---------------------------------------------
  `r-xp`   Readable, executable, private; not writable
  `rw-p`   Readable, writable, private; not executable
  `r--p`   Read-only, private
  `rw-s`   Readable, writable, shared

A private (`p`) mapping generally uses copy-on-write semantics: changes
made by a process are private to that process once a page is copied. A
shared (`s`) mapping can be visible to other processes mapping the same
shared region.

------------------------------------------------------------------------

## 6. Validate the Memory Accounting Invariant

For each mapping row, compare the following quantities:

-   `Kbytes`: total virtual size of the mapping.
-   `RSS`: resident portion of that mapping.
-   `Dirty`: dirty portion of resident memory.

The expected accounting relationship for the reported values is:

\[ `\boxed{\text{Dirty} \leq \text{RSS} \leq \text{Kbytes}}`{=tex} \]

### Why this relationship is expected

1.  A mapping's RSS represents the portion of its virtual size currently
    resident in physical memory, so RSS should not exceed the mapping's
    Kbytes.
2.  Dirty memory is a subset of resident memory, so Dirty should not
    exceed RSS.

Check the values in individual mapping rows and, where applicable, the
summary row. The values are reported in kilobytes and may be rounded.
Interpret the relationship using the figures reported by your system;
accounting details can vary across kernel and tool versions.

### Example observation

The following values are illustrative only:

  Mapping             Kbytes   RSS   Dirty Check
  ----------------- -------- ----- ------- -----------------
  Executable text        120    80       0 0 ≤ 80 ≤ 120
  Shared library        1800   900      20 20 ≤ 900 ≤ 1800
  Heap                   512   256     256 256 ≤ 256 ≤ 512
  Stack                  132    64       8 8 ≤ 64 ≤ 132

Use the actual values from your `pmap -x` output for the experiment
report.

------------------------------------------------------------------------

## 7. Useful Commands

  -----------------------------------------------------------------------
  Command                             Purpose
  ----------------------------------- -----------------------------------
  `ps aux`                            Lists running processes

  `pgrep -a process_name`             Finds process IDs by name

  `pmap <PID>`                        Displays a process memory map

  `pmap -x <PID>`                     Displays extended memory mapping
                                      information

  `cat /proc/<PID>/maps`              Shows the process's virtual memory
                                      mappings and permissions

  `cat /proc/<PID>/status`            Shows process status and memory
                                      summary fields

  `watch -n 1 "pmap -x <PID>"`        Refreshes the extended map every
                                      second
  -----------------------------------------------------------------------

For example, to inspect a process repeatedly:

``` bash
watch -n 1 "pmap -x 1234"
```

Replace `1234` with the PID you selected.

------------------------------------------------------------------------

## 8. Practical Test Procedure

1.  Open a terminal and identify an active process using `ps aux` or
    `pgrep -a`.

2.  Record the process ID.

3.  Display its extended memory map:

    ``` bash
    pmap -x <PID>
    ```

4.  Identify rows for the executable, shared libraries, `[heap]`, and
    `[stack]`, where present.

5.  Inspect the `Mode` column and interpret permissions such as `r-xp`
    and `rw-p`.

6.  Record the `Kbytes`, `RSS`, and `Dirty` values for selected
    mappings.

7.  Verify the relationship:

    ``` text
    Dirty <= RSS <= Kbytes
    ```

8.  Optionally compare the output with `/proc/<PID>/maps` and
    `/proc/<PID>/status`.

9.  Repeat the inspection for another active process and compare the
    layouts.

## 9. Observation Table

Fill in this table using values from your own `pmap -x` output.

  Memory region / mapping         Address     Kbytes   RSS   Dirty Mode
  ------------------------------- --------- -------- ----- ------- ------
  Executable text                 ---            ---   ---     --- ---
  Shared library (`libc.so`)      ---            ---   ---     --- ---
  Heap (`[heap]`, if present)     ---            ---   ---     --- ---
  Stack (`[stack]`, if present)   ---            ---   ---     --- ---

**Invariant check:** For each selected row, confirm that
`Dirty ≤ RSS ≤ Kbytes`.

## 10. Safety and Notes

-   Replace `<PID>` with a real process ID; do not include the angle
    brackets in the command.
-   Some processes may restrict access to their memory information. Try
    a process owned by your user if permission is denied.
-   Memory mappings can change while a process is running, so repeated
    `pmap` output may differ.
-   A library may be represented by multiple mappings with different
    permissions.
-   Not every process will display a traditional `[heap]` mapping, and
    thread stacks may be represented differently.
-   The `p` and `s` indicators describe private and shared mapping
    behavior; they are not access permissions.
-   `pmap` reports memory accounting information, not a complete
    description of how much physical memory is exclusively owned by a
    process. Shared resident pages may be counted in the RSS of multiple
    processes.

## Conclusion

This experiment uses `pmap -x` to inspect the virtual memory layout of
Linux processes. By examining the Address, Kbytes, RSS, Dirty, Mode, and
Mapping columns, it is possible to identify code, shared libraries,
heap, stack, and page permissions. Comparing Dirty, RSS, and Kbytes
demonstrates the memory accounting relationship `Dirty ≤ RSS ≤ Kbytes`
for the reported mappings and provides a practical introduction to
virtual memory and shared memory behavior.
