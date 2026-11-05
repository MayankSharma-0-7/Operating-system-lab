# Experiment 5: Dual-Paradigm String Manipulation and Performance Profiling

## Aim

To implement string length calculation and substring extraction using
both Bash string operations and C character-array operations, then
compare their execution performance using the Linux `time` utility.

## Objectives

-   Calculate string length in Bash using `${#str}`.
-   Extract a substring in Bash using `${str:start:length}`.
-   Calculate string length in C by traversing characters until the null
    terminator (`'\0'`).
-   Copy a substring into a destination buffer in C.
-   Compile and execute a C program using GCC.
-   Measure elapsed time and CPU time using `time`.

------------------------------------------------------------------------

## 1. String Manipulation in Bash

Bash provides built-in string operations, so basic length calculation
and substring extraction do not require an external command.

### Example script --- `string_bash.sh`

``` bash
#!/bin/bash

str="Linux String Manipulation"

# Calculate string length
length=${#str}

# Extract a substring
# Bash substring positions are zero-based.
start=6
count=6
substring=${str:start:count}

echo "Original string: $str"
echo "String length: $length"
echo "Substring: $substring"
```

### Run the script

``` bash
chmod +x string_bash.sh
./string_bash.sh
```

### Key Bash expressions

  -----------------------------------------------------------------------
  Expression                          Purpose
  ----------------------------------- -----------------------------------
  `${#str}`                           Returns the length of the string

  `${str:start:length}`               Extracts `length` characters
                                      beginning at zero-based position
                                      `start`

  `start=6`                           Sets the substring starting
                                      position

  `count=6`                           Sets the number of characters to
                                      extract
  -----------------------------------------------------------------------

**Note:** Bash string length and substring positions are affected by the
current locale and character encoding. For simple ASCII input, each
character occupies one position.

------------------------------------------------------------------------

## 2. String Manipulation in C

In C, strings are character arrays terminated by the null character
`'\0'`. A string-length loop increments a counter until it reaches this
terminator.

To extract a substring, the program copies the requested characters into
a destination buffer and appends `'\0'` so that the result is a valid C
string.

### Example program --- `string_c.c`

``` c
#include <stdio.h>

int string_length(const char str[])
{
    int length = 0;

    while (str[length] != '\0')
    {
        length++;
    }

    return length;
}

void extract_substring(
    const char source[],
    char destination[],
    int start,
    int count
)
{
    int i;

    for (i = 0; i < count && source[start + i] != '\0'; i++)
    {
        destination[i] = source[start + i];
    }

    destination[i] = '\0';
}

int main(void)
{
    const char str[] = "Linux String Manipulation";
    char substring[7];

    int length = string_length(str);

    extract_substring(str, substring, 6, 6);

    printf("Original string: %s\n", str);
    printf("String length: %d\n", length);
    printf("Substring: %s\n", substring);

    return 0;
}
```

### Compile and run

``` bash
gcc -Wall -Wextra -O2 string_c.c -o string_c
./string_c
```

### Explanation

1.  `string_length()` starts a counter at zero.
2.  The loop checks each character until `'\0'` is found.
3.  `extract_substring()` copies characters beginning at the requested
    index.
4.  The copy stops when the requested count is reached or the source
    string ends.
5.  The destination is terminated with `'\0'`.
6.  The destination array must be large enough for the copied characters
    plus the terminating null character.

For the example, the substring buffer has seven bytes: six for the
requested characters and one for `'\0'`.

------------------------------------------------------------------------

## 3. Benchmarking with the Linux `time` Utility

### Concept

The `time` utility measures how long a command takes to execute. Its
common measurements are:

-   **real:** Wall-clock time from start to finish.
-   **user:** CPU time spent executing the program's code in user mode.
-   **sys:** CPU time spent by the operating system kernel on behalf of
    the program.

### Run each implementation

``` bash
time ./string_bash.sh
time ./string_c
```

The output commonly resembles:

``` text
real    0m0.003s
user    0m0.001s
sys     0m0.002s
```

These are illustrative values only. Actual measurements depend on the
machine, operating system, workload, and system activity.

### Use GNU `/usr/bin/time`

On Linux systems with GNU `time`, the following command prints the three
measurements in a compact format:

``` bash
/usr/bin/time -f "real=%e s user=%U s sys=%S s" ./string_bash.sh
/usr/bin/time -f "real=%e s user=%U s sys=%S s" ./string_c
```

Format specifiers:

  Specifier   Meaning
  ----------- -------------------------------------
  `%e`        Elapsed real time in seconds
  `%U`        CPU time in user mode, in seconds
  `%S`        CPU time in system mode, in seconds

**Important:** `time` may be provided as a shell keyword or as an
external program. The format option `-f` is supported by GNU
`/usr/bin/time`; it may not be supported by every shell's built-in
`time`.

------------------------------------------------------------------------

## 4. Improving the Benchmark

The sample programs perform very little work. A single run may finish
too quickly for reliable comparison, and the reported time may be
rounded to zero.

For a more useful experiment:

1.  Run each implementation several times.
2.  Record the `real`, `user`, and `sys` values.
3.  Compare repeated measurements rather than relying on one run.
4.  Keep the input and amount of work equivalent between
    implementations.
5.  Avoid running other CPU-intensive tasks during measurement.

For more measurable timings, repeat the same string operation many times
in each implementation. Ensure that the compiler cannot remove the C
work as unused and that both implementations perform equivalent
operations. A benchmark that only repeats an operation on a fixed, tiny
string may mostly measure interpreter or process-startup overhead rather
than the cost of string manipulation itself.

### Results table

Fill in this table with measurements from your own Linux system.

  Implementation     Real time (s)   User time (s)   System time (s)
  ---------------- --------------- --------------- -----------------
  Bash                         ---             ---               ---
  C                            ---             ---               ---

### Observations

-   Bash provides concise built-in syntax for common string operations.
-   C requires explicit traversal and copying when implementing these
    operations manually.
-   A compiled C program may have lower runtime overhead for repeated
    low-level operations, but the result depends on the workload and
    measurement method.
-   For tiny programs, process startup and measurement resolution can
    dominate the result.

------------------------------------------------------------------------

## 5. Bash and C Comparison

  -----------------------------------------------------------------------
  Feature                 Bash                    C
  ----------------------- ----------------------- -----------------------
  String representation   Shell string            Null-terminated
                                                  character array

  Length calculation      `${#str}`               Loop until `'\0'`

  Substring extraction    `${str:start:length}`   Copy characters into a
                                                  destination buffer

  Memory handling         Managed by the shell    Programmer manages
                                                  destination-buffer
                                                  capacity

  Compilation             Interpreted by Bash     Compiled, for example,
                                                  with GCC

  Typical use             Shell scripting and     Systems programming and
                          automation              low-level operations
  -----------------------------------------------------------------------

------------------------------------------------------------------------

## 6. Practical Test Procedure

1.  Create `string_bash.sh` and enter the Bash example.

2.  Create `string_c.c` and enter the C program.

3.  Make the Bash script executable:

    ``` bash
    chmod +x string_bash.sh
    ```

4.  Compile the C source:

    ``` bash
    gcc -Wall -Wextra -O2 string_c.c -o string_c
    ```

5.  Run both programs and compare their displayed string lengths and
    substrings.

6.  Measure each program using `time` or `/usr/bin/time`.

7.  Repeat the measurements and record the results in the table.

8.  Compare the real, user, and sys times, keeping in mind the
    limitations of very short benchmarks.

## 7. Quick Reference

  -------------------------------------------------------------------------------------------
  Task                                Command or syntax
  ----------------------------------- -------------------------------------------------------
  Bash string length                  `${#str}`

  Bash substring                      `${str:start:length}`

  C string terminator                 `'\0'`

  Compile C program                   `gcc -Wall -Wextra -O2 file.c -o program`

  Run Bash script                     `./string_bash.sh`

  Run C program                       `./string_c`

  Measure execution                   `time ./program`

  GNU time with formatted output      `/usr/bin/time -f "real=%e user=%U sys=%S" ./program`
  -------------------------------------------------------------------------------------------

## Conclusion

This experiment demonstrates two approaches to string manipulation. Bash
offers compact built-in expressions for calculating string length and
extracting substrings, while C uses null-terminated character arrays,
loops, and explicit buffer copying. The Linux `time` utility provides
real, user, and system timing measurements that can be used to compare
program execution, provided the workload is large enough to produce
meaningful results.
