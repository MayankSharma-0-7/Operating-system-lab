# Experiment 1: Linux Basics, Redirection & Process Management

A practical introduction to essential Linux commands for file-system
navigation, file and directory management, input/output redirection,
pipes, text searching, terminal editors, and process control.

## Objectives

By completing this experiment, you should be able to:

-   Navigate the Linux file system and manage files and directories.
-   Read file contents and select specific portions of text.
-   Understand standard input, standard output, and standard error.
-   Redirect and append command output.
-   Connect commands using pipes and common text-processing utilities.
-   Search text with `grep` and locate files with `find`.
-   Perform basic editing in `vim` and `nano`.
-   Inspect, control, and adjust the priority of running processes.

------------------------------------------------------------------------

## 1. Directory and File Operations

### 1.1 Navigate directories

  Command   Purpose                                 Example
  --------- --------------------------------------- -----------------
  `pwd`     Display the current working directory   `pwd`
  `cd`      Change the current directory            `cd /home/user`
  `cd ..`   Move to the parent directory            `cd ..`
  `cd ~`    Move to the home directory              `cd ~`

### 1.2 Create directories: `mkdir`

``` bash
mkdir projects
mkdir -p projects/linux/basics
mkdir -v reports
mkdir -m 755 secure_dir
```

-   `-p`: Create parent directories as needed; do not report an error if
    the directory already exists.
-   `-v`: Print a message for each directory created.
-   `-m`: Set the directory permissions at creation time.

### 1.3 Remove directories: `rmdir`

``` bash
rmdir empty_dir
rmdir -v empty_dir
rmdir -p projects/linux/basics
```

-   `-p`: Remove the specified directory and its empty parent
    directories.
-   `-v`: Print a message for each directory removed.

`rmdir` removes empty directories only. It does not remove directories
containing files.

### 1.4 List files and metadata: `ls`

``` bash
ls
ls -l
ls -a
ls -h
ls -lah
```

-   `-l`: Use long listing format, showing details such as permissions,
    owner, size, and modification time.
-   `-a`: Include hidden entries.
-   `-h`: Display file sizes in a human-readable format when used with a
    long listing.

### 1.5 Create or update timestamps: `touch`

``` bash
touch notes.txt
touch -c existing.txt
touch -t 202611051030 report.txt
touch -d "2026-11-05 10:30" report.txt
```

-   Default: Create an empty file if it does not exist, or update its
    access and modification timestamps if it does.
-   `-c`: Do not create the file if it does not exist.
-   `-t`: Set timestamps using a timestamp value.
-   `-d`: Set timestamps using a date/time string.

### 1.6 Copy files and directories: `cp`

``` bash
cp source.txt destination.txt
cp -r source_dir backup_dir
cp -i source.txt destination.txt
cp -p source.txt backup.txt
```

-   `-r`: Copy directories recursively.
-   `-i`: Prompt before overwriting a destination.
-   `-p`: Preserve mode, ownership, and timestamps where permitted.

### 1.7 Move or rename: `mv`

``` bash
mv old_name.txt new_name.txt
mv file.txt documents/
mv -i source.txt destination.txt
mv -n source.txt destination.txt
mv -v source.txt destination.txt
```

-   `-i`: Prompt before overwriting.
-   `-n`: Do not overwrite an existing destination.
-   `-v`: Display each move operation.

### 1.8 Delete files and directories: `rm`

``` bash
rm file.txt
rm -i file.txt
rm -r directory/
rm -rf directory/
```

-   `-i`: Prompt before each removal.
-   `-r`: Remove directories and their contents recursively.
-   `-f`: Force removal without prompting and ignore nonexistent files.

**Caution:** `rm -rf` can permanently delete large amounts of data.
Check the path carefully before running it.

### 1.9 View file contents: `cat`

``` bash
cat file.txt
cat -n file.txt
cat -b file.txt
```

-   `-n`: Number all output lines.
-   `-b`: Number non-empty output lines.

### 1.10 Display selected portions: `head` and `tail`

``` bash
head file.txt
head -n 5 file.txt
tail file.txt
tail -n 10 file.txt
```

-   `head -n 5`: Display the first five lines.
-   `tail -n 10`: Display the last ten lines.

------------------------------------------------------------------------

## 2. I/O Redirection and Streams

Linux commands commonly use three standard data streams:

  Stream                         File descriptor Purpose
  ---------------------------- ----------------- -------------------------------
  Standard input (`stdin`)                   `0` Data supplied to a command
  Standard output (`stdout`)                 `1` Normal command output
  Standard error (`stderr`)                  `2` Error and diagnostic messages

### 2.1 Overwrite output: `>`

``` bash
ls > files.txt
```

Redirects standard output to `files.txt`. If the file already exists,
its previous contents are overwritten.

### 2.2 Append output: `>>`

``` bash
date >> log.txt
```

Appends standard output to the end of `log.txt` without replacing its
existing contents.

### 2.3 Redirect errors: `2>`

``` bash
ls missing_file 2> errors.txt
```

Redirects standard error (file descriptor `2`) to `errors.txt`.

### 2.4 Redirect output and errors together: `&>`

``` bash
command &> output_and_errors.txt
```

Redirects both standard output and standard error to the same file. This
is supported by Bash.

A portable shell alternative is:

``` bash
command > output_and_errors.txt 2>&1
```

### 2.5 Read input from a file: `<`

``` bash
sort < names.txt
```

Uses `names.txt` as the command's standard input.

------------------------------------------------------------------------

## 3. Pipes and Utilities

A pipe (`|`) sends the standard output of one command to the standard
input of another.

### 3.1 Chain commands with pipes

``` bash
cat names.txt | sort
```

This sends the contents of `names.txt` to `sort`.

### 3.2 Sort lines: `sort`

``` bash
sort names.txt
sort -r names.txt
```

-   Default: Sort lines alphabetically according to the active locale.
-   `-r`: Reverse the sort order.

### 3.3 Remove adjacent duplicate lines: `uniq`

``` bash
uniq names.txt
sort names.txt | uniq
sort names.txt | uniq -c
```

`uniq` removes adjacent duplicate lines. Sorting first is a common way
to group identical lines together.

-   `-c`: Prefix each output line with its number of occurrences.

### 3.4 Display and save output: `tee`

``` bash
ls -lah | tee listing.txt
```

Displays the output on the terminal and writes a copy to `listing.txt`.

To append instead of overwrite:

``` bash
ls -lah | tee -a listing.txt
```

### 3.5 Convert input into command arguments: `xargs`

``` bash
cat files_to_remove.txt | xargs rm
```

`xargs` reads items from standard input and passes them as arguments to
another command.

Use it carefully when filenames may contain spaces or unusual
characters. For safer handling of arbitrary filenames, use
null-delimited input where the producing command supports it:

``` bash
find . -type f -print0 | xargs -0 ls -l
```

------------------------------------------------------------------------

## 4. Pattern Searching and Finding Files

### 4.1 Search text with `grep`

General syntax:

``` bash
grep [options] "pattern" filename
```

Examples:

``` bash
grep "Linux" notes.txt
grep -i "linux" notes.txt
grep -v "error" log.txt
grep -c "success" log.txt
grep -w "cat" words.txt
grep -n "main" program.c
```

  Option   Purpose
  -------- ------------------------------
  `-i`     Ignore case distinctions
  `-v`     Show lines that do not match
  `-c`     Count matching lines
  `-w`     Match whole words
  `-n`     Show line numbers

### 4.2 Find files and directories with `find`

General syntax:

``` bash
find [starting_path] [conditions]
```

Examples:

``` bash
find /home -name "report.txt"
find . -iname "*.txt"
find / -size +100M
find /home -user student
find . -empty
```

  Option or test   Purpose
  ---------------- ---------------------------------------
  `-name`          Match a name, case-sensitively
  `-iname`         Match a name, ignoring case
  `-size`          Filter by file size
  `-user`          Match files owned by a specified user
  `-empty`         Match empty files and directories

Useful size examples:

``` bash
find . -size +10M
find . -size -1k
find . -size 0
```

-   `+10M`: More than 10 MiB.
-   `-1k`: Less than 1 KiB.
-   `0`: Empty-size files.

Searching from `/` may produce permission errors or take a long time.
Use an appropriate starting directory and elevated privileges only when
necessary.

------------------------------------------------------------------------

## 5. Terminal Text Editors

Linux commonly provides both modal and modeless terminal editors.

### 5.1 `vim` (modal editor)

Start editing a file:

``` bash
vim notes.txt
```

Common modes and commands:

  Mode or key               Purpose
  ------------------------- ------------------------------------------
  Normal mode               Navigate and issue editing commands
  Insert mode (`i`)         Insert text before the cursor
  Append mode (`a`)         Insert text after the cursor
  Command-line mode (`:`)   Enter commands such as save and quit
  `Esc`                     Return to Normal mode
  `/word`                   Search forward for `word`
  `n`                       Move to the next search match
  `dd`                      Delete the current line
  `yy`                      Copy (yank) the current line
  `p`                       Paste after the cursor or below the line
  `u`                       Undo the previous change
  `:w`                      Save the file
  `:q`                      Quit
  `:wq`                     Save and quit
  `:q!`                     Quit without saving changes

### 5.2 `nano` (modeless editor)

Start editing a file:

``` bash
nano notes.txt
```

Common operations:

  Shortcut     Purpose
  ------------ ----------------------------------
  `Ctrl + O`   Write (save) the file
  `Enter`      Confirm the filename when saving
  `Ctrl + X`   Exit
  `Ctrl + W`   Search
  `Ctrl + K`   Cut the current line
  `Ctrl + U`   Paste the cut text
  `Ctrl + G`   Open help

Nano displays many shortcuts at the bottom of the terminal. In its
shortcut notation, `^` represents the `Ctrl` key.

------------------------------------------------------------------------

## 6. Process Management

### 6.1 Shell job control

``` bash
jobs
bg
bg %1
fg
fg %1
```

-   `jobs`: List jobs started by the current shell.
-   `bg`: Resume a stopped job in the background.
-   `bg %1`: Resume job number 1 in the background.
-   `fg`: Bring a job to the foreground.
-   `fg %1`: Bring job number 1 to the foreground.

A running foreground command can commonly be suspended with `Ctrl + Z`,
then resumed in the background with `bg`.

### 6.2 Inspect processes with `ps`

``` bash
ps
ps -e
ps -ef
ps -p 1234
ps aux
```

  -----------------------------------------------------------------------
  Command                             Purpose
  ----------------------------------- -----------------------------------
  `ps`                                Show processes associated with the
                                      current terminal

  `ps -e`                             Show all processes

  `ps -ef`                            Show a full-format listing of all
                                      processes

  `ps -p 1234`                        Show the process with PID `1234`

  `ps aux`                            Show processes in BSD-style format,
                                      including processes without a
                                      controlling terminal
  -----------------------------------------------------------------------

### 6.3 Send signals with `kill`

`kill` sends a signal to a process, usually identified by its process ID
(PID).

``` bash
kill -15 1234
kill -9 1234
kill -STOP 1234
kill -CONT 1234
```

  -----------------------------------------------------------------------
  Signal                  Common name             Purpose
  ----------------------- ----------------------- -----------------------
  `-15`                   `SIGTERM`               Request that a process
                                                  terminate gracefully

  `-9`                    `SIGKILL`               Force termination; the
                                                  process cannot catch or
                                                  ignore it

  `-STOP`                 `SIGSTOP`               Stop a process

  `-CONT`                 `SIGCONT`               Continue a stopped
                                                  process
  -----------------------------------------------------------------------

Use `SIGTERM` first when possible. Reserve `SIGKILL` for cases where a
process does not respond to a normal termination request.

### 6.4 Adjust CPU scheduling priority: `nice` and `renice`

``` bash
nice -n 10 command
renice 5 -p 1234
```

-   `nice`: Start a command with an adjusted niceness value.
-   `renice`: Change the niceness value of an existing process.
-   A higher niceness value generally means lower scheduling priority.
-   Permission requirements apply when changing priority, especially
    when attempting to increase a process's priority.

### 6.5 Monitor processes: `top`

``` bash
top
```

`top` provides a live, periodically refreshed view of system activity,
including:

-   Running processes
-   CPU usage
-   Memory usage
-   Process IDs and owners
-   Load and system resource information

Press `q` to quit `top`.

------------------------------------------------------------------------

## 7. Quick Command Reference

  -----------------------------------------------------------------------
  Category                Command                 Main use
  ----------------------- ----------------------- -----------------------
  Navigation              `pwd`, `cd`             Show or change the
                                                  working directory

  Directories             `mkdir`, `rmdir`        Create or remove empty
                                                  directories

  Listing                 `ls -lah`               List files, including
                                                  hidden files, with
                                                  details

  Files                   `touch`, `cp`, `mv`,    Create/update, copy,
                          `rm`                    move, and delete files

  Viewing                 `cat`, `head`, `tail`   Display file contents

  Redirection             `>`, `>>`, `2>`, `&>`,  Redirect input and
                          `<`                     output streams

  Pipes                   `|`                     Pass output between
                                                  commands

  Text utilities          `sort`, `uniq`, `tee`,  Process and route text
                          `xargs`                 streams

  Searching               `grep`, `find`          Search text and locate
                                                  files

  Editors                 `vim`, `nano`           Edit text files in the
                                                  terminal

  Job control             `jobs`, `bg`, `fg`      Manage shell jobs

  Processes               `ps`, `kill`, `top`     Inspect, signal, and
                                                  monitor processes

  Priority                `nice`, `renice`        Adjust process
                                                  scheduling priority
  -----------------------------------------------------------------------

------------------------------------------------------------------------

## 8. Safety Notes

-   Check your current directory with `pwd` before running
    file-management commands.
-   Be especially careful with `rm -rf`; verify the target path before
    execution.
-   Use `cp -i` and `mv -i` when you want an overwrite confirmation.
-   Preview text transformations before using `sed -i` or other in-place
    operations.
-   Prefer `SIGTERM` before `SIGKILL` when stopping processes.
-   Use `find` with a specific starting directory when possible to avoid
    unnecessarily broad searches.

## Conclusion

This experiment introduces the Linux commands and concepts needed for
everyday command-line work. It covers file-system operations, standard
streams and redirection, pipelines, searching, terminal editing, and
process management. Practising the commands with sample files and
harmless test processes will help build confidence using the Linux
shell.
