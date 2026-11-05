# Shell Redirection and the `sed` Stream Editor

A beginner-friendly guide to shell output redirection and common `sed`
commands for viewing, deleting, replacing, and rearranging text.

## Table of Contents

-   [Shell redirection](#shell-redirection)
-   [What is sed?](#what-is-sed)
-   [Printing lines](#printing-lines)
-   [Deleting lines](#deleting-lines)
-   [Substituting text](#substituting-text)
-   [Editing files in place](#editing-files-in-place)
-   [Patterns and regular
    expressions](#patterns-and-regular-expressions)
-   [Capturing groups](#capturing-groups)
-   [Swapping the first two words](#swapping-the-first-two-words)
-   [Quick reference](#quick-reference)

## Shell redirection

You can run commands sequentially on one line with a semicolon (`;`) or
a logical AND (`&&`).

``` bash
date; who > logged_users.txt
```

-   `date` prints the current date and time to the terminal.
-   `;` runs the next command after the first command finishes,
    regardless of its exit status.
-   `who` lists users currently logged in.
-   `>` redirects the output of `who` into `logged_users.txt`, replacing
    the file's previous contents if it exists.

To run the second command only if the first succeeds, use:

``` bash
date && who > logged_users.txt
```

**Note:** The availability and usefulness of `who` depend on the system
and its configuration. Alternatives for checking the current account or
session information include `whoami` and `w`.

## What is `sed`?

`sed` stands for **Stream Editor**. It is a command-line utility that
reads input, applies editing instructions, and writes the result to
standard output. It is commonly used to:

-   Print selected lines
-   Delete lines from the output
-   Search for text
-   Replace text
-   Match patterns and transform text automatically

Unlike an interactive editor such as `nano`, `sed` normally processes
text without opening an interactive editing screen.

The examples below use a file named `Names.txt` containing:

``` text
John Doe
Alice Smith
Rahul Sharma
Aniket Singh
```

## Printing lines

### Print the contents of a file

``` bash
sed -n 'p' Names.txt
```

-   `sed` starts the stream editor.
-   `-n` suppresses automatic printing.
-   `p` explicitly prints the selected line(s).
-   `Names.txt` is the input file.

Because `p` is applied to every line, this prints the entire file once.

### Print one specific line

``` bash
sed -n '2p' Names.txt
```

Prints only line 2:

``` text
Alice Smith
```

### Print a range of lines

``` bash
sed -n '2,4p' Names.txt
```

Prints lines 2 through 4:

``` text
Alice Smith
Rahul Sharma
Aniket Singh
```

## Deleting lines

### Delete a specific line from the output

``` bash
sed '2d' Names.txt
```

`2d` means delete line 2 from the output produced by `sed`. The original
file is not changed.

### Delete the first or last line from the output

``` bash
sed '1d' Names.txt
sed '$d' Names.txt
```

-   `1d` deletes the first line from the output.
-   `$d` deletes the last line from the output. In `sed` addresses, `$`
    means the final line.

To make a deletion permanent, use in-place editing:

``` bash
sed -i '2d' Names.txt
```

## Substituting text

The general substitution syntax is:

``` text
s/old_text/new_text/
```

### Replace the first match on each line

``` bash
sed 's/John/Amit/' Names.txt
```

This changes the first occurrence of `John` on each line in the output.
By default, only the first match on each line is replaced.

### Replace every match on each line

``` bash
sed 's/John/Amit/g' Names.txt
```

The `g` flag means global: it replaces every matching occurrence on each
line.

For example, if the input is:

``` text
John Doe
John Smith
Rahul John
```

the command above produces:

``` text
Amit Doe
Amit Smith
Rahul Amit
```

### Replace text at the beginning of a line

``` bash
sed 's/^John/Amit/' Names.txt
```

`^` anchors the match to the beginning of a line. A `John` appearing
elsewhere on the line will not match.

### Replace text at the end of a line

``` bash
sed 's/John$/Amit/' Names.txt
```

`$` anchors the match to the end of a line.

### Replace spaces with underscores

Replace every space on each line:

``` bash
sed 's/ /_/g' Names.txt
```

Replace only the first space on each line:

``` bash
sed 's/ /_/' Names.txt
```

For example, `John Michael Doe` becomes `John_Michael Doe` with the
second command.

## Editing files in place

By default, `sed` prints the transformed text and leaves the original
file unchanged.

Use `-i` to write changes back to the file:

``` bash
sed -i 's/John/Amit/g' Names.txt
```

  ------------------------------------------------------------------------
  Command                              Effect
  ------------------------------------ -----------------------------------
  `sed 's/John/Amit/g' Names.txt`      Prints the modified output; leaves
                                       the file unchanged

  `sed -i 's/John/Amit/g' Names.txt`   Modifies the original file
  ------------------------------------------------------------------------

### Create a backup before editing

``` bash
sed -i.bak 's/John/Amit/g' Names.txt
```

This edits `Names.txt` and creates `Names.txt.bak` containing the
original version. A backup is useful when testing transformations.

## Patterns and regular expressions

### The dot (`.`)

In a regular expression, `.` matches any single character.

``` bash
sed 's/.at/XXX/' file.txt
```

For input:

``` text
cat
bat
hat
dog
```

the output is:

``` text
XXX
XXX
XXX
dog
```

The pattern `.at` matches any one character followed by `at`.

### A character class that excludes spaces

`[^ ]` matches one character that is not a space. The `^` inside square
brackets means "not"; this differs from `^` at the beginning of a
pattern, where it anchors a match to the start of a line.

`[^ ]*` matches zero or more non-space characters. It is often used to
match a word.

### Replace the first word on each line

``` bash
sed 's/[^ ]*/WORD/' Names.txt
```

For the sample names, the output is:

``` text
WORD Doe
WORD Smith
WORD Sharma
WORD Singh
```

## Capturing groups

A capturing group stores part of a match so it can be reused in the
replacement text. In basic `sed` regular expressions, groups are written
with escaped parentheses: `\(...\)`.

### Capture and reuse the first word

``` bash
sed 's/\([^ ]*\)/[\1]/' Names.txt
```

This wraps the first word in square brackets. For example:

``` text
[John] Doe
[Alice] Smith
[Rahul] Sharma
[Aniket] Singh
```

-   `\([^ ]*\)` captures the first word as Group 1.
-   `\1` inserts the text captured by Group 1.

### Capture two words and join them with a hyphen

``` bash
sed 's/\([^ ]*\) \([^ ]*\)/\1-\2/' Names.txt
```

The first group captures the first word; the second group captures the
second word. The replacement `\1-\2` joins them with a hyphen:

``` text
John-Doe
Alice-Smith
Rahul-Sharma
Aniket-Singh
```

## Swapping the first two words

To swap the first two words on every line:

``` bash
sed 's/^\([^ ]*\) \([^ ]*\)/\2 \1/' Names.txt
```

For the sample file, the output is:

``` text
Doe John
Smith Alice
Sharma Rahul
Singh Aniket
```

### How the pattern works

The search pattern is:

``` text
^\([^ ]*\) \([^ ]*\)
```

  Pattern part      Meaning
  ----------------- ------------------------------------
  `^`               Start at the beginning of the line
  `\([^ ]*\)`       Capture the first word as Group 1
  A literal space   Match the space between the words
  `\([^ ]*\)`       Capture the second word as Group 2

The replacement is:

``` text
\2 \1
```

It places Group 2 first, followed by a space and Group 1. Thus,
`John Doe` becomes `Doe John`.

### Complete working example

Create and edit the file:

``` bash
nano Names.txt
```

Enter:

``` text
John Doe
Alice Smith
Rahul Sharma
Aniket Singh
```

Save the file, then preview the transformation:

``` bash
sed 's/^\([^ ]*\) \([^ ]*\)/\2 \1/' Names.txt
```

To modify the original file:

``` bash
sed -i 's/^\([^ ]*\) \([^ ]*\)/\2 \1/' Names.txt
```

To preserve a backup first:

``` bash
sed -i.bak 's/^\([^ ]*\) \([^ ]*\)/\2 \1/' Names.txt
```

## Quick reference

  Symbol or command   Meaning
  ------------------- ---------------------------------------------------
  `sed`               Stream Editor
  `-n`                Suppress automatic printing
  `p`                 Print
  `d`                 Delete from the output
  `s`                 Substitute
  `g`                 Replace all matches on each line
  `-i`                Edit the original file in place
  `-i.bak`            Edit in place and create a `.bak` backup
  `^`                 Beginning of a line
  `$`                 End of a line / final line address
  `.`                 Any single character
  `*`                 Zero or more repetitions of the preceding pattern
  `[^ ]`              Any character except a space
  `\(...\)`           Capture a group
  `\1`                First captured group
  `\2`                Second captured group

## Key idea

`sed` first identifies text using an address or pattern, then applies an
instruction such as print (`p`), delete (`d`), or substitute (`s`).
Capturing groups let a replacement reuse matched text in a different
order.

For example:

``` bash
sed 's/^\([^ ]*\) \([^ ]*\)/\2 \1/' Names.txt
```

Finds the first word, the separating space, and the second word; stores
the words as Groups 1 and 2; and replaces them with Group 2 followed by
Group 1.

**Tip:** Preview a `sed` transformation without `-i` before applying it
to an important file. Use `-i.bak` when you want an original-file
backup.
