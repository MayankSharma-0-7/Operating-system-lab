# Experiment 9: Child Process Creation, Synchronization, and PID Retrieval in C

## Aim

To write a C program that creates a child process using `fork()`,
retrieves process IDs using `getpid()`, waits for the child process to
terminate using `wait(NULL)`, and displays the parent and child process
IDs.

## Objectives

-   Retrieve the current process ID using `getpid()`.
-   Create a child process using `fork()`.
-   Understand the return values of `fork()` in the parent and child
    processes.
-   Synchronize parent and child execution using `wait(NULL)`.
-   Prevent zombie processes by collecting the child's termination
    status.
-   Understand process lifecycle states and the possibility of race
    conditions.

------------------------------------------------------------------------

## 1. Process Creation with `fork()`

In Unix/Linux, `fork()` creates a new process by duplicating the calling
process. The newly created process is called the **child process**,
while the original process is the **parent process**.

The `fork()` system call returns different values depending on where it
executes:

  -----------------------------------------------------------------------
  Return value                        Meaning
  ----------------------------------- -----------------------------------
  Negative (`-1`)                     Process creation failed

  `0`                                 The code is executing in the child
                                      process

  Positive value                      The code is executing in the
                                      parent; the value is the child's
                                      PID
  -----------------------------------------------------------------------

Both parent and child continue execution from the statement immediately
following the successful `fork()` call.

## 2. Retrieve Process IDs with `getpid()`

The `getpid()` function returns the process ID (PID) of the process that
calls it.

-   In the parent process, `getpid()` returns the parent's PID.
-   In the child process, `getpid()` returns the child's PID.

The `getppid()` function can also be used by the child to retrieve its
parent's PID.

------------------------------------------------------------------------

## 3. C Program --- `process.c`

The following program creates a child process, displays the process IDs,
and makes the parent wait for the child to finish.

``` c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>

int main(void)
{
    pid_t pid;

    printf("Parent process started.\n");
    printf("Parent PID: %ld\n", (long)getpid());

    pid = fork();

    if (pid < 0)
    {
        perror("fork failed");
        return EXIT_FAILURE;
    }
    else if (pid == 0)
    {
        /* Child process */
        printf("\nChild process created.\n");
        printf("Child PID: %ld\n", (long)getpid());
        printf("Parent PID from child: %ld\n", (long)getppid());

        printf("Child process is terminating.\n");
        _exit(EXIT_SUCCESS);
    }
    else
    {
        /* Parent process */
        printf("\nParent is waiting for child PID: %ld\n", (long)pid);

        if (wait(NULL) == -1)
        {
            perror("wait failed");
            return EXIT_FAILURE;
        }

        printf("Child process has terminated.\n");
        printf("Parent process PID: %ld\n", (long)getpid());
    }

    return EXIT_SUCCESS;
}
```

### Why `_exit()` is used in the child

The example uses `_exit()` to terminate the child without flushing or
duplicating any stdio buffers inherited from the parent at `fork()`. If
you replace it with `exit()`, be aware that buffered output created
before `fork()` can be flushed by both processes if it was not flushed
before forking.

------------------------------------------------------------------------

## 4. Compile and Run the Program

Compile using GCC:

``` bash
gcc -Wall -Wextra -o process process.c
```

Run the executable:

``` bash
./process
```

### Example output

The exact PID values and the order of some messages will vary:

``` text
Parent process started.
Parent PID: 4200

Parent is waiting for child PID: 4201

Child process created.
Child PID: 4201
Parent PID from child: 4200
Child process is terminating.
Child process has terminated.
Parent process PID: 4200
```

The child may print its messages before or after the parent's waiting
message. The important synchronization guarantee is that the parent
prints `Child process has terminated.` only after `wait(NULL)` returns
successfully.

------------------------------------------------------------------------

## 5. Process Synchronization Using `wait(NULL)`

After `fork()`, the parent and child are independent processes. The
operating system scheduler determines when each process gets CPU time,
so their instructions can interleave.

The `wait(NULL)` system call makes the parent wait until one of its
child processes terminates. It also collects the child's termination
status.

### Why is `wait()` important?

-   It synchronizes the parent with child-process termination.
-   It prevents the parent from proceeding past the wait point before
    the child has terminated.
-   It allows the parent to collect the child's exit status.
-   It prevents a terminated child from remaining as an unreaped zombie
    after the parent has collected its status.

A **zombie process** is a child that has terminated but whose parent has
not yet collected its termination status. A short-lived zombie entry can
exist between the child's termination and the parent's successful
`wait()` call. Calling `wait()` reaps the child.

------------------------------------------------------------------------

## 6. Race Conditions and Process Lifecycle

### Race condition

A race condition can occur when program behavior depends on which
process the scheduler runs first.

For example, without synchronization, the parent might print a message
or finish before the child prints its output. This can make output order
unpredictable.

Using `wait(NULL)` ensures that the parent waits for child termination,
but it does not force the child's output to appear before every parent
message printed before the wait call.

### Basic process lifecycle

1.  **Creation:** The parent calls `fork()` and the operating system
    creates the child.
2.  **Ready / running:** The child becomes eligible for scheduling and
    executes when given CPU time.
3.  **Termination:** The child exits or is otherwise terminated.
4.  **Zombie state:** The terminated child may temporarily remain in the
    process table until its parent collects its status.
5.  **Reaping:** The parent calls `wait()` and the kernel releases the
    child's remaining process-table entry.

------------------------------------------------------------------------

## 7. Important System Calls and Functions

  -----------------------------------------------------------------------
  Function                            Purpose
  ----------------------------------- -----------------------------------
  `fork()`                            Creates a child process

  `getpid()`                          Returns the PID of the calling
                                      process

  `getppid()`                         Returns the parent PID of the
                                      calling process

  `wait(NULL)`                        Waits for a child to terminate and
                                      collects its status

  `_exit(status)`                     Terminates a process without
                                      running standard C exit handlers

  `perror()`                          Prints an error message associated
                                      with the current `errno`
  -----------------------------------------------------------------------

### Header files

  -----------------------------------------------------------------------
  Header                              Purpose
  ----------------------------------- -----------------------------------
  `<stdio.h>`                         Provides `printf()` and `perror()`

  `<stdlib.h>`                        Provides `EXIT_SUCCESS` and
                                      `EXIT_FAILURE`

  `<unistd.h>`                        Declares `fork()`, `getpid()`,
                                      `getppid()`, and `_exit()`

  `<sys/types.h>`                     Defines process-related types such
                                      as `pid_t`

  `<sys/wait.h>`                      Declares `wait()` and related
                                      functions
  -----------------------------------------------------------------------

------------------------------------------------------------------------

## 8. Practical Test Procedure

1.  Create a source file named `process.c`.

2.  Enter the C program shown above.

3.  Compile the program:

    ``` bash
    gcc -Wall -Wextra -o process process.c
    ```

4.  Execute it:

    ``` bash
    ./process
    ```

5.  Note the parent PID printed at the beginning.

6.  Note the child PID returned by `fork()` to the parent and printed by
    the child using `getpid()`.

7.  Compare the child's `getppid()` result with the parent's PID.

8.  Observe that the parent reports child termination only after
    `wait(NULL)` returns.

9.  Run the program several times and note that PIDs and output
    interleaving may change.

## 9. Quick Reference

  Task                    Function / command
  ----------------------- ------------------------------------------
  Compile the C program   `gcc -Wall -Wextra -o process process.c`
  Run the program         `./process`
  Create a child          `fork()`
  Get current PID         `getpid()`
  Get parent PID          `getppid()`
  Wait for a child        `wait(NULL)`
  Terminate child         `_exit(EXIT_SUCCESS)`

## 10. Safety and Notes

-   Always check whether `fork()` returned `-1` before using its result.
-   The child and parent execute independently after a successful
    `fork()`.
-   Do not assume a fixed order for messages printed by the parent and
    child.
-   Check the return value of `wait()` to detect errors.
-   `wait(NULL)` waits for any one child. If a program creates multiple
    children, it may need to call `wait()` repeatedly or use `waitpid()`
    to wait for a particular child.
-   The child's parent PID can change if the original parent exits
    before the child; in this experiment, the parent waits, so it
    remains alive until the child terminates.
-   A zombie is not a running process; it has terminated but has not yet
    been reaped by its parent.

## Conclusion

This experiment demonstrates basic Unix/Linux process creation and
synchronization in C. The `fork()` system call creates a child process,
`getpid()` and `getppid()` identify the processes, and `wait(NULL)`
allows the parent to synchronize with child termination and collect its
status. These mechanisms help explain process scheduling, race
conditions, and zombie-process prevention.
