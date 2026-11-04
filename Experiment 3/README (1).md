# Experiment 3: Bash Scripting, Conditionals & Loops

## Aim

To understand Bash scripting, command-line arguments, positional
parameters, conditional statements, loops, command substitution, and
basic file-system automation in Linux.

## 1. Introduction to Shell Scripts

A shell script is a text file containing commands that are executed by a
shell such as Bash. Scripts help automate repetitive tasks by running
multiple commands in sequence.

Example:

``` bash
#!/bin/bash
echo "Hello, Linux!"
date
pwd
```

The first line, `#!/bin/bash`, is called the **shebang**. It specifies
the Bash interpreter.

## 2. Command-Line Arguments

Command-line arguments are values passed to a script when it is
executed.

Example:

``` bash
./myscript.sh hello.txt notes folder1
```

### Positional Parameters

  Parameter   Purpose
  ----------- -------------------------------------
  `$0`        Name or path used to run the script
  `$1`        First argument
  `$2`        Second argument
  `$3`        Third argument
  `$#`        Total number of arguments
  `$@`        All arguments as separate items

## 3. Conditional Testing

Bash uses `if`, `elif`, and `else` to make decisions. Conditions are
commonly written inside `[ ]`.

### Numeric Operators

  Operator   Meaning
  ---------- --------------------------
  `-eq`      Equal
  `-ne`      Not equal
  `-lt`      Less than
  `-le`      Less than or equal to
  `-gt`      Greater than
  `-ge`      Greater than or equal to

### File Test Operators

  Operator   Meaning
  ---------- ---------------------------------
  `-e`       Path exists
  `-f`       Regular file exists
  `-d`       Directory exists
  `-s`       File exists and is not empty
  `-r`       Read permission is available
  `-w`       Write permission is available
  `-x`       Execute permission is available

## 4. Command Substitution

Command substitution stores the output of a command in a variable.

Syntax:

``` bash
variable=$(command)
```

Example:

``` bash
current_date=$(date)
echo "$current_date"
```

## 5. Filename Manipulation

The `tr` command can convert lowercase text to uppercase.

``` bash
echo "hello world" | tr '[:lower:]' '[:upper:]'
```

Output:

``` text
HELLO WORLD
```

The `mv` command renames or moves files.

``` bash
mv old.txt new.txt
```

## 6. Conditional Statements and Loops

### A. if Statement

``` bash
#!/bin/bash
if [ -f "notes.txt" ]
then
    echo "notes.txt is a regular file."
fi
```

The command runs only when `notes.txt` exists as a regular file.

### B. if-else Statement

``` bash
#!/bin/bash
if [ -f "notes.txt" ]
then
    echo "notes.txt is a regular file."
else
    echo "notes.txt is not a regular file."
fi
```

The `then` block runs when the condition is true; otherwise, the `else`
block runs. `fi` ends the conditional statement.

### C. for Loop

A `for` loop repeats commands for each value in a list.

``` bash
#!/bin/bash
for name in Ali Rahul Aman
do
    echo "Hello $name"
done
```

Output:

``` text
Hello Ali
Hello Rahul
Hello Aman
```

Other common `for` loop forms:

``` bash
for i in {1..5}
do
    echo "Count: $i"
done
```

``` bash
for (( i=1; i<=5; i++ ))
do
    echo "Number: $i"
done
```

``` bash
for file in *.sh
do
    echo "Found script: $file"
done
```

### D. while Loop

A `while` loop repeats commands as long as its condition is true.

``` bash
#!/bin/bash
count=1
while (( count <= 5 ))
do
    echo "$count"
    ((count++))
done
```

## 7. Creating and Running a Bash Script

1.  Create a script:

    ``` bash
    nano script_name.sh
    ```

2.  Write the Bash commands and save the file.

3.  Give it execute permission:

    ``` bash
    chmod +x script_name.sh
    ```

4.  Run the script:

    ``` bash
    ./script_name.sh
    ```

5.  To pass arguments:

    ``` bash
    ./script_name.sh argument1 argument2
    ```

## 8. Task 1 --- Check Whether an Argument Is a File or Directory

### Script

``` bash
#!/bin/bash
if [ $# -ne 1 ]; then
    echo "No Argument passed, Try Again with an Argument"
    exit 1
fi

target="$1"

if [ -f "$target" ]; then
    echo "$target is a regular file."
elif [ -d "$target" ]; then
    echo "$target is a directory."
elif [ -e "$target" ]; then
    echo "$target exists, but is neither a regular file nor a directory."
else
    echo "Error: $target does not exist."
fi
```

### Explanation

-   `$#` checks the number of arguments.
-   `-ne 1` checks whether the argument count is not equal to one.
-   `exit 1` stops the script when the input is invalid.
-   `$1` stores the first argument in `target`.
-   `-f` checks for a regular file.
-   `-d` checks for a directory.
-   `-e` checks whether the path exists.
-   `elif` checks another condition when the previous one is false.

### Run and Test

``` bash
chmod +x check_type.sh
./check_type.sh /etc/passwd
./check_type.sh /var/log
```

## 9. Task 2 --- Rename Files to Uppercase

### Script

``` bash
#!/bin/bash
for file in "$@"
do
    mv "$file" "$(echo "$file" | tr '[:lower:]' '[:upper:]')"
done
```

### Explanation

-   `"$@"` represents all command-line arguments.
-   The `for` loop processes each filename separately.
-   `echo "$file"` outputs the current filename.
-   The pipe (`|`) sends that output to `tr`.
-   `tr '[:lower:]' '[:upper:]'` converts lowercase letters to
    uppercase.
-   `$(...)` captures the converted filename.
-   `mv` renames the original file.

### Run and Test

``` bash
nano rename.sh
chmod +x rename.sh
touch notes.txt report.txt hello.txt
./rename.sh notes.txt report.txt hello.txt
```

The filenames become `NOTES.TXT`, `REPORT.TXT`, and `HELLO.TXT`. The
contents of the files do not change.

> **Note:** Check that the uppercase destination filenames do not
> already exist before running the rename script, because `mv` may
> replace an existing destination file.

## 10. Arithmetic Expansion

Bash supports arithmetic using `$(( ))`.

``` bash
a=10
b=3

echo "Addition: $((a + b))"
echo "Subtraction: $((a - b))"
echo "Multiplication: $((a * b))"
echo "Division: $((a / b))"
echo "Modulus: $((a % b))"
echo "Exponentiation: $((a ** b))"
```

For integer division, the fractional part is discarded. For example,
`10 / 3` gives `3`.

## 11. Important Commands and Concepts

  Command / Concept   Purpose
  ------------------- ------------------------------------------------
  `#!/bin/bash`       Selects Bash as the interpreter
  `echo`              Displays text or values
  `date`              Displays the current date and time
  `pwd`               Displays the current directory
  `nano`              Opens a text editor in the terminal
  `chmod +x`          Adds execute permission
  `touch`             Creates an empty file or updates its timestamp
  `mv`                Moves or renames files
  `tr`                Translates or converts characters
  `$(...)`            Performs command substitution
  `"$@"`              Expands all arguments separately
  `do ... done`       Defines the body of a loop
  `if ... fi`         Defines a conditional statement

## Conclusion

In this experiment, we learned the basics of Bash scripting,
command-line arguments, positional parameters, file tests, conditional
statements, and loops. We also practised command substitution,
arithmetic expansion, and automating file-system tasks such as checking
paths and renaming files.
