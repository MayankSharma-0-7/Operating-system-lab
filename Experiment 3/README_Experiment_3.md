# Experiment 3: Shell Automation, Command-Line Parameter Handling, and Branching Logic

A practical introduction to Bash scripting, positional parameters,
conditional tests, loops, and two utility scripts for inspecting and
renaming files.

## Objectives

By completing this experiment, you should be able to:

-   Write and run basic Bash scripts.
-   Access command-line arguments using positional parameters such as
    `$1`, `$#`, and `$@`.
-   Evaluate file and directory conditions using `-f`, `-d`, and `-s`.
-   Compare integer values using `-eq`.
-   Use `for` loops to process multiple inputs.
-   Create a script that identifies files and directories.
-   Create a script that renames input filenames to uppercase.

------------------------------------------------------------------------

## 1. Bash Scripts

A Bash script is a text file containing commands that the Bash shell
executes in sequence.

A script typically begins with a shebang:

``` bash
#!/bin/bash
```

Save commands in a file with the `.sh` extension, for example:

``` bash
nano script.sh
```

Make the script executable and run it:

``` bash
chmod +x script.sh
./script.sh
```

Alternatively, run it directly with Bash:

``` bash
bash script.sh
```

## 2. Command-Line Positional Parameters

Bash provides special parameters for accessing arguments passed to a
script.

  -----------------------------------------------------------------------
  Parameter                           Meaning
  ----------------------------------- -----------------------------------
  `$1`                                First command-line argument

  `$2`                                Second command-line argument

  `$#`                                Number of arguments supplied

  `$@`                                All positional arguments,
                                      individually preserved when quoted
                                      as `"$@"`
  -----------------------------------------------------------------------

### Example

Create `parameters.sh`:

``` bash
#!/bin/bash

echo "First argument: $1"
echo "Second argument: $2"
echo "Number of arguments: $#"
echo "All arguments: $@"
```

Run it:

``` bash
chmod +x parameters.sh
./parameters.sh apple banana orange
```

Example output:

``` text
First argument: apple
Second argument: banana
Number of arguments: 3
All arguments: apple banana orange
```

**Tip:** Use `"$@"` when passing arguments through to another command or
iterating over them. It preserves each argument as a separate item,
including filenames containing spaces.

## 3. Conditional Evaluation

Bash conditional statements allow a script to make decisions based on
tests.

### Common test operators

  Operator    Meaning
  ----------- ----------------------------------------------------------
  `-f path`   True if the path is a regular file
  `-d path`   True if the path is a directory
  `-s path`   True if the path exists and has a size greater than zero
  `-eq`       Integer equality comparison

### Example: Check a path

``` bash
#!/bin/bash

path="$1"

if [ -f "$path" ]; then
    echo "It is a regular file."
elif [ -d "$path" ]; then
    echo "It is a directory."
else
    echo "The path does not exist or is neither a regular file nor a directory."
fi
```

Run:

``` bash
bash check_path.sh notes.txt
```

The `if`, `elif`, and `else` branches select which message is printed
based on the test result.

### Example: Integer comparison

``` bash
#!/bin/bash

a=10
b=10

if [ "$a" -eq "$b" ]; then
    echo "The numbers are equal."
else
    echo "The numbers are not equal."
fi
```

## 4. Iterating with a `for` Loop

A `for` loop repeats a set of commands for each item in a list.

### Example

``` bash
#!/bin/bash

for item in "$@"; do
    echo "Processing: $item"
done
```

Run:

``` bash
bash loop.sh file1.txt file2.txt file3.txt
```

The loop processes each supplied argument separately.

------------------------------------------------------------------------

## 5. Utility Script 1: `check_type.sh`

### Purpose

`check_type.sh` checks each path supplied as a command-line argument:

-   If it is a regular file, the script reports its size.
-   If it is a directory, the script lists its contents.
-   If it is neither, the script reports that the path was not found or
    is not a regular file or directory.

### Script

Create the file:

``` bash
nano check_type.sh
```

Add:

``` bash
#!/bin/bash

if [ "$#" -eq 0 ]; then
    echo "Usage: $0 path1 [path2 ...]"
    exit 1
fi

for path in "$@"; do
    if [ -f "$path" ]; then
        echo "Regular file: $path"
        if [ -s "$path" ]; then
            wc -c < "$path" | {
                read -r size
                echo "Size: $size bytes"
            }
        else
            echo "Size: 0 bytes"
        fi

    elif [ -d "$path" ]; then
        echo "Directory: $path"
        echo "Contents:"
        ls -lah -- "$path"

    else
        echo "Not found or unsupported path: $path"
    fi

    echo "-------------------------"
done
```

Make it executable:

``` bash
chmod +x check_type.sh
```

Run it with a file and a directory:

``` bash
./check_type.sh notes.txt Documents
```

You can also provide several paths:

``` bash
./check_type.sh notes.txt Documents image.png missing.txt
```

### How it works

1.  `$#` checks how many arguments were supplied.
2.  `-eq 0` tests whether the argument count is zero.
3.  `for path in "$@"` processes every supplied path.
4.  `-f` checks whether a path is a regular file.
5.  `-s` checks whether the file has a nonzero size.
6.  `wc -c` counts the file's bytes.
7.  `-d` checks whether a path is a directory.
8.  `ls -lah` lists the directory contents in a detailed, human-readable
    format.

------------------------------------------------------------------------

## 6. Utility Script 2: `rename_upper.sh`

### Purpose

`rename_upper.sh` batch-renames input filenames to uppercase.

For example:

``` text
report.txt  -> REPORT.TXT
notes.pdf   -> NOTES.PDF
image.png   -> IMAGE.PNG
```

### Script

Create the file:

``` bash
nano rename_upper.sh
```

Add:

``` bash
#!/bin/bash

if [ "$#" -eq 0 ]; then
    echo "Usage: $0 filename1 [filename2 ...]"
    exit 1
fi

for file in "$@"; do
    if [ -e "$file" ]; then
        directory=$(dirname -- "$file")
        filename=$(basename -- "$file")
        upper_name=$(printf '%s' "$filename" | tr '[:lower:]' '[:upper:]')
        target="$directory/$upper_name"

        if [ "$file" = "$target" ]; then
            echo "Already uppercase: $file"

        elif [ -e "$target" ]; then
            echo "Skipped: target already exists: $target"

        else
            mv -i -- "$file" "$target"
            echo "Renamed: $file -> $target"
        fi
    else
        echo "File or directory not found: $file"
    fi
done
```

Make it executable:

``` bash
chmod +x rename_upper.sh
```

Run it by supplying filenames:

``` bash
./rename_upper.sh report.txt notes.pdf image.png
```

### How it works

1.  `$#` checks whether any filenames were provided.
2.  `for file in "$@"` processes each input separately.
3.  `-e` checks whether the input path exists.
4.  `dirname` obtains the parent directory.
5.  `basename` obtains the filename.
6.  `tr '[:lower:]' '[:upper:]'` converts lowercase letters to
    uppercase.
7.  The script constructs the target path in the same directory.
8.  It checks whether the target already exists to avoid overwriting it.
9.  `mv -i` performs the rename with an overwrite confirmation if
    needed.

**Important:** Renaming changes the actual filenames. Test the script on
sample files first. The script skips a rename if the uppercase target
already exists.

------------------------------------------------------------------------

## 7. Suggested Practical Test

Create a test directory and sample files:

``` bash
mkdir -p experiment3_test/docs
touch experiment3_test/report.txt
echo "Bash scripting experiment" > experiment3_test/notes.txt
touch experiment3_test/docs/example.pdf
```

Inspect the paths:

``` bash
./check_type.sh experiment3_test/report.txt experiment3_test experiment3_test/docs
```

Rename sample files:

``` bash
./rename_upper.sh experiment3_test/report.txt experiment3_test/notes.txt experiment3_test/docs/example.pdf
```

Check the results:

``` bash
find experiment3_test -print
```

Expected renamed paths include:

``` text
experiment3_test/REPORT.TXT
experiment3_test/NOTES.TXT
experiment3_test/docs/EXAMPLE.PDF
```

------------------------------------------------------------------------

## 8. Quick Reference

  ---------------------------------------------------------------------------------
  Concept                 Syntax                            Purpose
  ----------------------- --------------------------------- -----------------------
  Shebang                 `#!/bin/bash`                     Select Bash as the
                                                            script interpreter

  First argument          `$1`                              Read the first argument

  Argument count          `$#`                              Count supplied
                                                            arguments

  All arguments           `"$@"`                            Iterate over arguments
                                                            separately

  Regular file test       `[ -f "$path" ]`                  Check for a regular
                                                            file

  Directory test          `[ -d "$path" ]`                  Check for a directory

  Non-empty test          `[ -s "$path" ]`                  Check for a non-empty
                                                            file

  Integer equality        `[ "$a" -eq "$b" ]`               Compare integers

  Loop                    `for item in "$@"; do ... done`   Process multiple
                                                            arguments

  Uppercase conversion    `tr '[:lower:]' '[:upper:]'`      Convert lowercase
                                                            letters to uppercase

  Rename                  `mv -- source target`             Move or rename a path
  ---------------------------------------------------------------------------------

## 9. Safety Notes

-   Test scripts using sample files and directories before using
    important data.
-   Quote variables such as `"$file"` and use `"$@"` to handle spaces in
    filenames.
-   Check whether a target filename already exists before renaming.
-   Do not run batch-renaming scripts on important directories without a
    backup.
-   Review the script and the paths supplied to it before execution.

## Conclusion

This experiment introduces Bash automation and command-line parameter
handling. It demonstrates how to use positional parameters, conditional
tests, and `for` loops to create two practical utilities:
`check_type.sh` for identifying files and directories, and
`rename_upper.sh` for batch-renaming filenames to uppercase.
