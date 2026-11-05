# Experiment 10: Observing Virtual Memory Addresses Across Concurrent Processes in C

## Aim

To write a C program that prints the virtual memory address of a stack
variable and remains active, then run multiple instances concurrently to
compare their addresses and observe the effect of Address Space Layout
Randomization (ASLR).

## Objectives

-   Declare a local variable on the stack and print its virtual address
    using `%p`.
-   Keep each process alive using an infinite loop and `sleep(1)`.
-   Launch three or four instances of the program concurrently.
-   Compare the printed addresses across processes.
-   Understand how separate process address spaces and ASLR affect
    virtual addresses.

------------------------------------------------------------------------

## 1. Concept: Virtual Addresses and Process Isolation

Each Linux process has its own virtual address space. A pointer printed
by a process is a virtual address in that process's address space; it is
not necessarily a unique physical RAM address.

Because processes have separate address spaces, two different processes
can use the same numerical virtual address while referring to different
memory. This is not a collision in the sense of sharing the same
physical memory.

**Address Space Layout Randomization (ASLR)** is a security feature that
randomizes the locations of selected memory regions, such as the stack,
shared libraries, and memory mappings. This makes memory locations less
predictable and can make exploitation more difficult.

When comparing addresses from separate program executions, ASLR may
cause the stack variable to appear at different virtual addresses. The
exact result depends on system configuration, executable type, compiler
settings, and process launch conditions.

------------------------------------------------------------------------

## 2. C Program --- `address.c`

The program below declares a local stack variable, prints its address
and process ID, then remains active so that several instances can be
inspected at the same time.

``` c
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>

int main(void)
{
    int stack_variable = 100;

    printf("Process ID (PID): %ld\n", (long)getpid());
    printf("Stack variable value: %d\n", stack_variable);
    printf("Stack variable address: %p\n",
           (void *)&stack_variable);
    fflush(stdout);

    while (1)
    {
        sleep(1);
    }

    return 0;
}
```

### Explanation

-   `int stack_variable = 100;` declares a local variable whose storage
    is typically on the process stack.
-   `&stack_variable` obtains the variable's address.
-   `%p` is the correct `printf()` format specifier for a pointer. The
    pointer is cast to `void *` for portable use with `%p`.
-   `getpid()` returns the process ID, allowing each instance to be
    identified.
-   `fflush(stdout)` ensures that the output is displayed immediately,
    even when output is redirected.
-   `while (1)` keeps the process alive.
-   `sleep(1)` pauses execution for one second on each loop iteration,
    avoiding a busy loop that would consume unnecessary CPU.

------------------------------------------------------------------------

## 3. Compile the Program

Compile the source file using GCC:

``` bash
gcc -Wall -Wextra -o address address.c
```

Run one instance to verify the output:

``` bash
./address
```

Example output (the PID and address will differ on your system):

``` text
Process ID (PID): 5201
Stack variable value: 100
Stack variable address: 0x7ffd82a4b12c
```

Press `Ctrl+C` to stop this foreground instance.

------------------------------------------------------------------------

## 4. Launch Three or Four Concurrent Instances

Start multiple instances in the background:

``` bash
./address &
./address &
./address &
./address &
```

Each command starts a separate process. Because the program flushes its
output, the PID and stack address should appear promptly.

You can also save the process IDs in shell variables:

``` bash
./address &
PID1=$!

./address &
PID2=$!

./address &
PID3=$!

./address &
PID4=$!

echo "PIDs: $PID1 $PID2 $PID3 $PID4"
```

The special shell variable `$!` contains the PID of the most recently
started background job.

List the running instances:

``` bash
ps -o pid,comm,stat -p "$PID1","$PID2","$PID3","$PID4"
```

------------------------------------------------------------------------

## 5. Compare the Virtual Addresses

Record the address printed by each process and compare the values.

### Observation table

Fill in the table using the output from your own system.

  Instance      PID Stack variable address
  ----------- ----- ------------------------
  Process 1     --- ---
  Process 2     --- ---
  Process 3     --- ---
  Process 4     --- ---

### What to observe

-   Each process prints its own PID and the virtual address of its local
    variable.
-   The addresses may differ between processes because their stack
    locations can be randomized.
-   If two processes print the same numerical virtual address, that does
    not mean they share the same variable or physical memory. Each
    address is interpreted within its own process address space.
-   Re-running the program can produce different addresses when ASLR is
    enabled.

Do not assume that every run must produce a different address. Address
placement depends on the operating system, executable configuration, and
runtime environment.

------------------------------------------------------------------------

## 6. Check ASLR Configuration on Linux

Linux commonly exposes its ASLR setting through:

``` bash
cat /proc/sys/kernel/randomize_va_space
```

Typical values are:

    Value Meaning
  ------- -------------------------------------------
      `0` Address-space randomization disabled
      `1` Conservative randomization
      `2` Full randomization (commonly the default)

This setting describes the system's ASLR policy. It does not by itself
prove that every mapping in every process is randomized.

You can inspect a process's memory mappings with:

``` bash
cat /proc/<PID>/maps
```

Replace `<PID>` with one of the process IDs. The mappings file shows
virtual address ranges and permissions for that process.

------------------------------------------------------------------------

## 7. Stop the Background Processes

Stop the instances using the saved PIDs:

``` bash
kill "$PID1" "$PID2" "$PID3" "$PID4"
```

Check whether they have stopped:

``` bash
ps -p "$PID1","$PID2","$PID3","$PID4"
```

If fewer than four instances were launched, use only the PID variables
that were assigned.

You can also list matching processes before stopping them:

``` bash
pgrep -a address
```

Avoid killing unrelated processes that happen to have a similar name.

------------------------------------------------------------------------

## 8. Practical Test Procedure

1.  Create a source file named `address.c`.

2.  Enter the C program shown above.

3.  Compile it:

    ``` bash
    gcc -Wall -Wextra -o address address.c
    ```

4.  Run one instance and verify that it prints a PID and stack variable
    address.

5.  Launch three or four instances in the background.

6.  Record each PID and printed virtual address.

7.  Compare the numerical addresses across the processes.

8.  Check the system ASLR setting:

    ``` bash
    cat /proc/sys/kernel/randomize_va_space
    ```

9.  Optionally inspect one process's memory mappings:

    ``` bash
    cat /proc/<PID>/maps
    ```

10. Stop all test instances using their saved PIDs.

## 9. Quick Reference

  -------------------------------------------------------------------------------
  Command or expression                       Purpose
  ------------------------------------------- -----------------------------------
  `gcc -Wall -Wextra -o address address.c`    Compile the C program

  `./address`                                 Run the program

  `./address &`                               Start an instance in the background

  `$!`                                        Get the most recently started
                                              background process ID

  `getpid()`                                  Retrieve the current process ID

  `&stack_variable`                           Obtain the variable's address

  `printf("%p", (void *)ptr)`                 Print a pointer value

  `sleep(1)`                                  Pause for approximately one second

  `cat /proc/sys/kernel/randomize_va_space`   Read the system ASLR setting

  `cat /proc/<PID>/maps`                      Inspect a process's virtual memory
                                              mappings

  `kill PID`                                  Send a termination signal to a
                                              process
  -------------------------------------------------------------------------------

## 10. Safety and Notes

-   The program intentionally runs indefinitely. Stop all instances
    after completing the observations.
-   The `sleep(1)` call prevents the program from continuously consuming
    CPU while it waits.
-   A virtual address is meaningful within the address space of the
    process that produced it. Do not interpret equal pointer values
    across processes as proof of shared physical memory.
-   ASLR can change address placement between executions, but observed
    values depend on kernel configuration, executable format, compiler
    options, and runtime conditions.
-   Avoid changing the system-wide ASLR setting on a shared or
    production machine for this experiment.
-   The address of a local variable is typically in the stack region,
    but compiler and ABI details determine the precise layout.

## Conclusion

This experiment demonstrates how separate Linux processes can print
virtual addresses for local stack variables while running concurrently.
Comparing these addresses provides a practical way to discuss process
address-space isolation and Address Space Layout Randomization. The
experiment also shows that identical numerical virtual addresses in
different processes do not imply that the processes share the same
physical memory.
