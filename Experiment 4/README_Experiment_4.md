# Experiment 4: System Session Auditing, Unix Epoch Arithmetic, and Stream-Based Document Editing

## Aim

To practice auditing user sessions, calculating elapsed time with Unix
Epoch timestamps, extracting a range of lines from a file, and deleting
a target word from multiple files using Bash, `awk`, and `sed`.

## Objectives

-   Read login-session information from the system `who` database.
-   Use `awk` to extract relevant fields from command output.
-   Work with Unix Epoch time and calculate elapsed durations.
-   Print a selected range of lines with `sed`.
-   Process multiple filenames using positional parameters and `shift`.
-   Perform in-place text editing with `sed -i`.

------------------------------------------------------------------------

## 1. User Login Time and Session Duration --- `user_time.sh`

### Concept

The `who` command displays users currently logged in and information
about their login sessions. Its output can be processed with `awk` to
extract the username and login details.

Unix Epoch time represents the number of seconds elapsed since
**1970-01-01 00:00:00 UTC**. The command `date +%s` returns the current
time in Epoch seconds. Subtracting a login timestamp (also converted to
Epoch seconds) from the current Epoch time gives the elapsed duration.

### Example script

``` bash
#!/bin/bash

who | awk '{print $1, $3, $4}' |
while read -r username login_date login_time
do
    login_epoch=$(date -d "$login_date $login_time" +%s 2>/dev/null)

    if [ -n "$login_epoch" ]; then
        current_epoch=$(date +%s)
        duration=$((current_epoch - login_epoch))

        hours=$((duration / 3600))
        minutes=$(((duration % 3600) / 60))
        seconds=$((duration % 60))

        echo "User: $username"
        echo "Login: $login_date $login_time"
        echo "Active duration: ${hours}h ${minutes}m ${seconds}s"
        echo "-----------------------------"
    fi
done
```

> **Note:** The exact fields and date format produced by `who` can vary
> between Unix/Linux systems. The script assumes the date and time
> appear in fields 3 and 4 and that GNU `date` supports `-d`. Verify the
> output of `who` on your system before relying on this example. The
> calculation measures elapsed time since the reported login time; it
> does not necessarily represent uninterrupted keyboard or application
> activity.

### Run the script

``` bash
chmod +x user_time.sh
./user_time.sh
```

### Important commands

  -----------------------------------------------------------------------
  Command or expression               Purpose
  ----------------------------------- -----------------------------------
  `who`                               Displays logged-in user sessions

  `awk '{print $1, $3, $4}'`          Extracts selected fields from each
                                      line

  `date -d "DATE TIME" +%s`           Converts a date/time string to
                                      Epoch seconds (GNU `date`)

  `date +%s`                          Gets the current Epoch timestamp

  `$((a - b))`                        Performs integer arithmetic in Bash
  -----------------------------------------------------------------------

------------------------------------------------------------------------

## 2. Display a Range of Lines --- `display_range.sh`

### Concept

`sed` is a stream editor that can select and process lines from text.
The option `-n` suppresses automatic printing, while the `p` command
prints only the selected lines.

The address expression `${start},${end}p` selects lines from the
starting line through the ending line, inclusive.

### Example script

``` bash
#!/bin/bash

if [ "$#" -ne 3 ]; then
    echo "Usage: $0 <filename> <start_line> <end_line>"
    exit 1
fi

file=$1
start=$2
end=$3

if [ ! -f "$file" ]; then
    echo "Error: '$file' is not a regular file."
    exit 1
fi

if ! [[ "$start" =~ ^[0-9]+$ && "$end" =~ ^[0-9]+$ ]] ||
   [ "$start" -lt 1 ] || [ "$end" -lt "$start" ]; then
    echo "Error: Enter valid line numbers with end >= start."
    exit 1
fi

sed -n "${start},${end}p" "$file"
```

### Run the script

``` bash
chmod +x display_range.sh
./display_range.sh sample.txt 3 8
```

This prints lines 3 through 8 of `sample.txt`, including both boundary
lines.

### Important commands

  Command or expression   Purpose
  ----------------------- -------------------------------------------------
  `sed -n`                Suppresses default output
  `${start},${end}p`      Prints the inclusive line range
  `$#`                    Number of command-line arguments
  `[[ ... =~ ... ]]`      Tests a value against a Bash regular expression
  `[ -f "$file" ]`        Checks whether a regular file exists

------------------------------------------------------------------------

## 3. Delete a Target Word from Multiple Files --- `delete_word.sh`

### Concept

A Bash script can accept a target word followed by one or more
filenames. The `shift` command removes the first positional parameter
and moves the remaining parameters down by one position.

The `sed` substitution command can delete matching text. With the global
flag `g`, all matches on each line are processed. The `-i` option edits
each file in place.

### Example script

``` bash
#!/bin/bash

if [ "$#" -lt 2 ]; then
    echo "Usage: $0 <word> <file1> [file2 ...]"
    exit 1
fi

word=$1
shift

for file in "$@"
do
    if [ ! -f "$file" ]; then
        echo "Skipping '$file': file not found."
        continue
    fi

    sed -i "s/${word}//g" "$file"
    echo "Processed: $file"
done
```

### Run the script

``` bash
chmod +x delete_word.sh
./delete_word.sh Linux file1.txt file2.txt file3.txt
```

This removes occurrences of `Linux` from the listed files.

> **Important:** This example treats the target as a basic `sed` regular
> expression, not necessarily as a literal string. Special characters
> such as `.`, `*`, `[`, and `/` can affect the pattern or command.
> Also, `sed -i` modifies files directly. Make backups before running it
> on important data. The exact `-i` syntax can vary between GNU and
> BSD/macOS `sed`.

### Important commands

  Command or expression       Purpose
  --------------------------- ---------------------------------------------------
  `shift`                     Removes the first positional parameter
  `"$@"`                      Expands the remaining arguments as separate words
  `for file in "$@"`          Iterates over each supplied filename
  `sed -i`                    Edits a file in place
  `s/pattern/replacement/g`   Replaces every match on each line
  `continue`                  Skips to the next loop iteration

------------------------------------------------------------------------

## 4. Practical Test Procedure

1.  Create the three scripts using a text editor.

2.  Save them as `user_time.sh`, `display_range.sh`, and
    `delete_word.sh`.

3.  Make the scripts executable:

    ``` bash
    chmod +x user_time.sh display_range.sh delete_word.sh
    ```

4.  Run `user_time.sh` and compare its output with the sessions shown by
    `who`.

5.  Create a sample text file and use `display_range.sh` to print a
    chosen range of lines.

6.  Create copies of sample files, then run `delete_word.sh` on those
    copies.

7.  Open the edited files and verify that the intended occurrences were
    removed.

## 5. Quick Reference

  Topic                              Command or syntax
  ---------------------------------- ----------------------------------
  Current login sessions             `who`
  Extract fields                     `awk '{print $1, $3, $4}'`
  Current Epoch seconds              `date +%s`
  Convert date/time to Epoch         `date -d "DATE TIME" +%s`
  Bash arithmetic                    `$((expression))`
  Print a line range                 `sed -n "${start},${end}p" file`
  Remove first argument              `shift`
  Iterate over remaining arguments   `for item in "$@"`
  In-place substitution              `sed -i 's/old/new/g' file`

## 6. Safety and Compatibility Notes

-   Check the actual output of `who` before assuming fixed field
    positions.
-   The `date -d` option is available in GNU `date`; systems using BSD
    `date` may require different syntax.
-   Session-duration calculations depend on correctly parsing the login
    date and time.
-   Validate filenames and line-number arguments before processing them.
-   Quote shell variables such as `"$file"` and `"$@"` to handle spaces
    in filenames safely.
-   Make backup copies before using `sed -i`, because it changes the
    original files.
-   The deletion example uses a regular-expression pattern. Escape
    special characters if you need literal matching.

## Conclusion

This experiment demonstrates three useful Linux shell techniques:
extracting and calculating login-session durations with `who`, `awk`,
and Epoch timestamps; selecting document line ranges with `sed`; and
processing multiple files with Bash positional parameters, `shift`, and
in-place stream editing. These techniques help automate routine system
auditing and text-processing tasks.
