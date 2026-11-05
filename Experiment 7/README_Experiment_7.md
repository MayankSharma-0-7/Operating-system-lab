# Experiment 7: OS Scheduler Dynamics, CPU Utilization Profiles, and Process States

## Aim

To study how the Linux operating system schedules CPU-bound and
I/O-bound processes by running two contrasting C programs concurrently
and observing their CPU usage, process states, and scheduling
priorities.

## Objectives

-   Create a CPU-bound program that performs continuous arithmetic
    operations.
-   Create an I/O-bound program that writes to a file and sleeps between
    writes.
-   Compile and run both programs concurrently in the background.
-   Observe process states, CPU utilization, and scheduling information
    using `top`.
-   Understand the difference between a running/runnable process and a
    process in interruptible sleep.

------------------------------------------------------------------------

## 1. CPU-Bound Workload --- `cpu_bound.c`

### Concept

A CPU-bound process spends most of its time executing instructions and
competing for CPU time. A tight arithmetic loop is a simple way to keep
a CPU core busy.

### Example program

``` c
#include <stdio.h>
#include <stdint.h>
#include <signal.h>

int main(void)
{
    volatile uint64_t counter = 0;

    printf("CPU-bound process started. PID: %ld\n", (long)getpid());
    fflush(stdout);

    while (1)
    {
        counter++;

        /* Periodically print progress without printing every iteration. */
        if (counter % 1000000000ULL == 0)
        {
            printf("Counter: %llu\n", (unsigned long long)counter);
            fflush(stdout);
        }
    }

    return 0;
}
```

The program uses `getpid()`, so include the required header:

``` c
#include <unistd.h>
```

Add that include with the other headers when saving the final source
file.

### Compile

``` bash
gcc -Wall -Wextra -O2 cpu_bound.c -o cpu_bound
```

### Expected behavior

-   The loop continuously performs arithmetic.
-   While it is actively executing, the process is generally shown as
    `R` (running or runnable).
-   On a system with multiple CPU cores, a single-threaded process
    typically uses up to approximately one logical CPU core. In `top`'s
    default process view, this is commonly close to 100% CPU, though
    display conventions and system load can affect the reading.

------------------------------------------------------------------------

## 2. I/O-Bound Workload --- `io_bound.c`

### Concept

An I/O-bound process frequently waits for input/output operations or
other events rather than continuously using the CPU. This example writes
a short message to a file, flushes it, and then sleeps before repeating.

### Example program

``` c
#include <stdio.h>
#include <unistd.h>

int main(void)
{
    FILE *file = fopen("io_output.txt", "a");

    if (file == NULL)
    {
        perror("fopen");
        return 1;
    }

    printf("I/O-bound process started. PID: %ld\n", (long)getpid());
    fflush(stdout);

    while (1)
    {
        if (fprintf(file, "Writing data from I/O-bound process\n") < 0)
        {
            perror("fprintf");
            fclose(file);
            return 1;
        }

        if (fflush(file) != 0)
        {
            perror("fflush");
            fclose(file);
            return 1;
        }

        sleep(2);
    }

    fclose(file);
    return 0;
}
```

### Compile

``` bash
gcc -Wall -Wextra -O2 io_bound.c -o io_bound
```

### Expected behavior

-   The program writes a line to `io_output.txt`.
-   `fflush()` sends buffered output to the underlying file stream.
-   `sleep(2)` suspends the process for approximately two seconds.
-   During the sleep interval, the process is normally shown in state
    `S` (interruptible sleep).
-   Because it spends most of its time sleeping, its average CPU usage
    should be very low.

> **Note:** A file write may be served by the page cache and does not
> necessarily cause a physical disk write immediately. This program
> demonstrates a workload with periodic file operations and sleeping; it
> is not a guarantee of sustained physical-disk activity.

------------------------------------------------------------------------

## 3. Launch Both Processes in the Background

Run both programs from the directory containing the compiled
executables.

### Start the CPU-bound process

``` bash
./cpu_bound &
CPU_PID=$!
echo "CPU-bound PID: $CPU_PID"
```

### Start the I/O-bound process

``` bash
./io_bound &
IO_PID=$!
echo "I/O-bound PID: $IO_PID"
```

The `&` operator starts a command in the background. `$!` stores the
process ID of the most recently started background job.

Keep the terminal open so you can use the stored process IDs.

------------------------------------------------------------------------

## 4. Observe CPU Utilization and Process States with `top`

Use the process IDs to monitor both programs:

``` bash
top -p "$CPU_PID","$IO_PID"
```

In `top`, examine the following columns:

  Column    Meaning
  --------- ----------------------------------------
  `PID`     Process ID
  `USER`    Process owner
  `PR`      Scheduling priority displayed by `top`
  `NI`      Nice value
  `S`       Process state
  `%CPU`    CPU utilization
  `%MEM`    Percentage of physical memory used
  `TIME+`   Accumulated CPU time

### Expected observations

  -----------------------------------------------------------------------
  Process           Workload          Typical state     Expected CPU
                                                        usage
  ----------------- ----------------- ----------------- -----------------
  `cpu_bound`       Continuous        `R`               Approximately one
                    arithmetic loop                     logical CPU core
                                                        when scheduled

  `io_bound`        File writes       Usually `S`       Very low on
                    followed by       during sleep      average
                    `sleep(2)`                          
  -----------------------------------------------------------------------

The state may change while you observe it. For example, the I/O-bound
program can briefly become runnable while executing a write or other
instruction, then return to `S` when it sleeps.

### Understanding process states

-   **R --- Running or runnable:** The process is executing on a CPU or
    is ready to run and waiting for CPU time.
-   **S --- Interruptible sleep:** The process is waiting for an event
    and can be woken by an appropriate signal or event.
-   **D --- Uninterruptible sleep:** The process is waiting in a state
    that generally cannot be interrupted by signals, often during
    certain I/O waits. This is not the expected steady state for the
    example's `sleep(2)` call.

------------------------------------------------------------------------

## 5. Compare Scheduling Priorities

Linux scheduling behavior can be inspected using `top`, `ps`, and the
`nice` value.

Check the two processes:

``` bash
ps -o pid,comm,ni,pri,stat,%cpu -p "$CPU_PID","$IO_PID"
```

The `NI` column displays the nice value. For ordinary processes, a lower
nice value generally means a more favorable scheduling weight, while a
higher nice value means a lower scheduling weight.

### Optional: Start processes with different nice values

You can launch the CPU-bound workload with a higher nice value:

``` bash
nice -n 10 ./cpu_bound &
CPU_PID=$!
```

Launch the I/O-bound workload with the default nice value:

``` bash
./io_bound &
IO_PID=$!
```

Then inspect them with:

``` bash
top -p "$CPU_PID","$IO_PID"
```

A nice value affects scheduling preference among normal tasks; it does
not guarantee a fixed CPU share. Actual CPU time depends on runnable
workloads, CPU affinity, scheduler policy, system load, and other
factors.

------------------------------------------------------------------------

## 6. Stop the Background Processes

Stop both programs when the experiment is complete:

``` bash
kill "$CPU_PID" "$IO_PID"
```

Check whether they have exited:

``` bash
ps -p "$CPU_PID","$IO_PID"
```

If necessary, press Enter in the terminal and use `wait` to collect the
background jobs:

``` bash
wait "$CPU_PID" "$IO_PID" 2>/dev/null
```

Remove the generated output file if it is no longer needed:

``` bash
rm -f io_output.txt
```

------------------------------------------------------------------------

## 7. Practical Test Procedure

1.  Create `cpu_bound.c` and `io_bound.c` using the examples above.
    Ensure `cpu_bound.c` includes `<unistd.h>` for `getpid()`.

2.  Compile both programs:

    ``` bash
    gcc -Wall -Wextra -O2 cpu_bound.c -o cpu_bound
    gcc -Wall -Wextra -O2 io_bound.c -o io_bound
    ```

3.  Start each program in the background and record its PID.

4.  Run `top -p "$CPU_PID","$IO_PID"` to monitor both processes.

5.  Observe the `S`, `%CPU`, `PR`, and `NI` columns.

6.  Run the `ps` command to capture process state and scheduling
    information.

7.  Optionally repeat the test with different nice values.

8.  Stop both processes and inspect or remove `io_output.txt`.

## 8. Observation Table

Record the values observed on your Linux system.

  Process         PID State (`S`)     Nice (`NI`)   CPU usage (%) Remarks
  ------------- ----- ------------- ------------- --------------- --------------------------------
  `cpu_bound`     --- ---                     ---             --- Continuous arithmetic workload
  `io_bound`      --- ---                     ---             --- Periodic file writes and sleep

## 9. Quick Reference

  ------------------------------------------------------------------------------------
  Command                                          Purpose
  ------------------------------------------------ -----------------------------------
  `gcc -Wall -Wextra -O2 file.c -o program`        Compile a C program

  `./program &`                                    Start a process in the background

  `$!`                                             Get the most recently started
                                                   background process ID

  `top -p PID1,PID2`                               Monitor selected processes

  `ps -o pid,comm,ni,pri,stat,%cpu -p PID1,PID2`   Display process and scheduling
                                                   information

  `nice -n 10 ./program &`                         Start a process with a specified
                                                   nice value

  `kill PID`                                       Send the default termination signal

  `wait PID`                                       Wait for a background process to
                                                   finish
  ------------------------------------------------------------------------------------

## 10. Safety and Notes

-   The CPU-bound program intentionally runs forever and can consume a
    full logical CPU core. Stop it after the observation.
-   The I/O-bound program appends to `io_output.txt` indefinitely. Stop
    it to prevent unnecessary file growth.
-   Run the programs only in a directory where you have permission to
    create and modify files.
-   A process in state `R` may be running or waiting in the runnable
    queue; it is not necessarily executing at the exact instant `top`
    samples it.
-   A process using `sleep()` is normally in state `S` while sleeping,
    but its state changes when it wakes up.
-   CPU percentages depend on `top`'s display mode and the number of CPU
    cores. On a multi-core machine, one fully utilized core is commonly
    shown as approximately 100% in the default process view.
-   Nice values influence scheduling preference but do not directly
    assign fixed CPU percentages.

## Conclusion

This experiment demonstrates how workload characteristics affect CPU
utilization and process states in Linux. The CPU-bound program
continuously competes for processor time and is typically observed in
state `R`, while the I/O-oriented program periodically writes to a file
and spends most of its time in interruptible sleep (`S`). Tools such as
`top`, `ps`, `nice`, and `kill` help inspect process scheduling behavior
and manage the test workloads.
