# Experiment 4: User Session Tracking, Line Ranges & Text Processing

## Aim

To understand Linux user-session tracking, Unix timestamps, line-range
extraction using `sed`, command-line argument shifting, and batch
deletion of lines from multiple files.

## Experiment 1 --- Determine User Working Period

### Complete Code

``` bash
#!/bin/bash
read -p "Enter username: " user

login_time=$(who | awk -v u="$user" '$1 == u {print $3, $4; exit}')

if [ -z "$login_time" ]
then
    echo "User is not currently logged in."
else
    echo "Login time: $login_time"

    login_seconds=$(date -d "$login_time" +%s)
    current_seconds=$(date +%s)
    total_seconds=$((current_seconds - login_seconds))

    hours=$((total_seconds / 3600))
    minutes=$(((total_seconds % 3600) / 60))

    echo "Total working time: $hours hours $minutes minutes"
fi
```

### Main Concepts

-   `read -p` displays a prompt and stores the entered username in
    `user`.
-   `who` displays currently logged-in users.
-   `awk` processes the output of `who` and selects the row matching the
    username.
-   `-v u="$user"` passes the Bash username variable to AWK as `u`.
-   `$1`, `$3`, and `$4` refer to AWK fields: username, date, and time.
-   `$(...)` runs a command and captures its output.
-   `-z "$login_time"` checks whether the variable is empty.

### Unix Timestamp and Time Calculation

A Unix timestamp is the number of seconds elapsed since January 1, 1970,
00:00:00 UTC (the Unix epoch).

``` bash
login_seconds=$(date -d "$login_time" +%s)
current_seconds=$(date +%s)
total_seconds=$((current_seconds - login_seconds))

hours=$((total_seconds / 3600))
minutes=$(((total_seconds % 3600) / 60))
```

-   `date -d` converts the supplied login date and time.
-   `date +%s` gives the current Unix timestamp.
-   Subtracting the timestamps gives the elapsed time in seconds.
-   Dividing by `3600` gives hours; using `% 3600` and dividing by `60`
    gives the remaining minutes.

## Experiment 2 --- Display Lines Between Start and End Line Numbers

### Complete Shell Script

``` bash
#!/bin/bash
if [ "$#" -ne 3 ]
then
    echo "Usage: $0 filename start_line end_line"
    exit 1
fi

file="$1"
start="$2"
end="$3"

sed -n "${start},${end}p" "$file"
```

### How to Run

``` bash
chmod +x script.sh
./script.sh names.txt 3 6
```

If `names.txt` contains:

``` text
Amit
Rahul
Aniket
Rohit
Neha
Priya
Karan
Aman
```

Output:

``` text
Aniket
Rohit
Neha
Priya
```

### Explanation

-   `$#` stores the total number of command-line arguments.
-   `-ne 3` checks that exactly three arguments were supplied.
-   `$1` is the filename, `$2` is the starting line, and `$3` is the
    ending line.
-   `sed -n` suppresses automatic printing.
-   `"${start},${end}p"` tells `sed` to print only the selected line
    range.

## Experiment 3 --- Delete Lines Containing a Specified Word

### Complete Code

``` bash
#!/bin/bash
if [ "$#" -lt 2 ]
then
    echo "Usage: $0 word file1 [file2 ...]"
    exit 1
fi

word="$1"
shift

for file in "$@"
do
    sed -i "/$word/d" "$file"
done
```

### Example

``` bash
./script.sh Aniket names.txt students.txt teachers.txt
```

The first argument (`Aniket`) is stored in `word`. The `shift` command
removes that first argument, leaving the filenames in `"$@"`.

### Explanation

-   `$#` checks the number of arguments.
-   `-lt 2` checks whether fewer than two arguments were supplied.
-   `word="$1"` stores the word to search for.
-   `shift` removes the first positional argument.
-   `for file in "$@"` processes each remaining filename.
-   `sed` is a stream editor.
-   `-i` edits the file directly.
-   `/Aniket/d` tells `sed` to delete lines matching the word `Aniket`.
-   `"$file"` is the current filename.

**Note:** This command edits the files directly. Keep a backup if the
original contents need to be preserved.

## Important Commands and Concepts

  Command / Concept   Purpose
  ------------------- -----------------------------------------------
  `who`               Shows currently logged-in users
  `awk`               Processes and selects text fields
  `date +%s`          Displays the current Unix timestamp
  `date -d`           Converts a supplied date/time
  `$(...)`            Captures command output
  `$((...))`          Performs arithmetic
  `sed -n`            Suppresses automatic output
  `sed -i`            Edits a file directly
  `shift`             Removes the first positional argument
  `"$@"`              Represents all remaining arguments separately
  `exit 1`            Ends the script with an error status

## Conclusion

In this experiment, we learned how to check a user's login session and
calculate the time spent logged in using Unix timestamps. We also used
`sed` to display a selected range of lines and delete lines containing a
specified word from multiple files. The experiment demonstrated how Bash
arguments, `shift`, loops, `awk`, and `sed` can automate common Linux
text-processing tasks.
