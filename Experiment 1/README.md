# Linux System Administration & Process Management

A practical reference guide to essential Linux commands for directory
and file operations, input/output redirection, pipes, text searching,
file discovery, text editors, and process management.

## Table of Contents

1.  [Directory Operations](#1-directory-operations)
2.  [File Operations](#2-file-operations)
3.  [I/O Redirection](#3-io-redirection)
4.  [Pipes and Command-Line
    Utilities](#4-pipes-and-command-line-utilities)
5.  [Grep: Searching Text](#5-grep-searching-text)
6.  [Find: Searching for Files](#6-find-searching-for-files)
7.  [Text Editors: Vim and Nano](#7-text-editors-vim-and-nano)
8.  [Linux Process Management](#8-linux-process-management)
9.  [Quick Revision](#9-quick-revision)

------------------------------------------------------------------------

## 1. Directory Operations

  ----------------------------------------------------------------------------------
  Command                 Purpose                 Example
  ----------------------- ----------------------- ----------------------------------
  `pwd`                   Displays the current    `pwd`
                          working directory.      

  `cd DIR`                Changes to another      `cd /etc/netplan`
                          directory.              

  `cd ..`                 Moves one directory up. `cd ..`

  `cd -`                  Returns to the previous `cd -`
                          directory.              

  `cd ~` or `cd`          Moves to the home       `cd ~`
                          directory.              

  `mkdir DIR`             Creates a directory.    `mkdir ~/scripts`

  `rmdir DIR`             Removes an empty        `rmdir ~/Templates`
                          directory.              

  `chmod MODE DIR`        Changes directory       `chmod 755 ~/Documents/projects`
                          permissions.            
  ----------------------------------------------------------------------------------

### Useful `mkdir` options

-   `-p`: Creates parent directories if they do not exist.

    ``` bash
    mkdir -p ~/Documents/projects/app/src
    ```

-   `-v`: Displays a message for each directory created.

    ``` bash
    mkdir -v ~/Downloads/iso
    ```

-   `-m`: Sets permissions while creating the directory.

    ``` bash
    mkdir -m 700 ~/.ssh
    ```

### Useful `rmdir` options

-   `-p`: Removes empty parent directories as well.

    ``` bash
    rmdir -p ~/Documents/projects/app/src
    ```

-   `-v`: Displays confirmation.

    ``` bash
    rmdir -v ~/Downloads/iso
    ```

> **Note:** `rmdir` removes only empty directories. Use `rm -r` only
> when you intentionally need to remove a directory and its contents.

## 2. File Operations

  --------------------------------------------------------------------------
  Command                 Purpose                 Example
  ----------------------- ----------------------- --------------------------
  `ls`                    Lists directory         `ls`
                          contents.               

  `touch FILE`            Creates an empty file   `touch notes.txt`
                          or updates its          
                          timestamp.              

  `cp SRC DEST`           Copies a file.          `cp file.txt backup.txt`

  `mv SRC DEST`           Moves or renames a      `mv old.txt new.txt`
                          file.                   

  `rm FILE`               Removes a file.         `rm old.txt`

  `cat FILE`              Displays file contents. `cat /etc/os-release`

  `head FILE`             Displays the first 10   `head -n 5 file.txt`
                          lines by default.       

  `tail FILE`             Displays the last 10    `tail -n 5 file.txt`
                          lines by default.       
  --------------------------------------------------------------------------

### Useful `ls` options

-   `-l`: Long listing with permissions, owner, size, and date.
-   `-a`: Includes hidden files.
-   `-h`: Displays human-readable file sizes.
-   `-t`: Sorts by modification time, newest first.
-   `-R`: Lists subdirectories recursively.

Example:

``` bash
ls -lah ~
```

### Useful `touch` options

-   `-c`: Does not create a file if it does not already exist.
-   `-t`: Sets a timestamp in the specified numeric format.
-   `-d`: Sets a timestamp using a date string.

Examples:

``` bash
touch -c ~/.bashrc
touch -t 202601011200 ~/install.log
touch -d "2026-04-23" ~/release_notes.txt
```

### Useful `cp` options

-   `-r` or `-R`: Copies directories recursively.
-   `-i`: Asks before overwriting.
-   `-p`: Preserves file mode, ownership, and timestamps where
    permitted.

Examples:

``` bash
cp -r /etc/netplan ~/backup_netplan
cp -i /etc/hosts ~/hosts
cp -p /etc/timezone ~/timezone_backup
```

### Useful `mv` options

-   `-i`: Asks before overwriting.
-   `-n`: Does not overwrite an existing destination.
-   `-v`: Displays the names of moved items.

Examples:

``` bash
mv -i test.conf ~/Documents/
mv -n ~/.bashrc.bak ~/.bashrc
mv -v ~/Downloads/*.deb ~/Packages/
```

### Useful `rm` options

-   `-r`: Removes directories and their contents recursively.
-   `-f`: Forces removal without prompting.
-   `-i`: Asks before each removal.

Examples:

``` bash
rm -i old_file.txt
rm -r ~/Downloads/temp_setup/
```

**Safety:** Be very careful with `rm -r` and especially `rm -rf`. Check
the path before pressing Enter; deletion may be permanent.

### `cat`, `head`, and `tail` examples

``` bash
cat -n /etc/shells       # Number all output lines
cat -b /etc/issue        # Number non-empty lines
head -n 10 /etc/passwd   # First 10 lines
head -n 5 /etc/fstab     # First 5 lines
tail -n 25 /var/log/auth.log  # Last 25 lines
tail -n 50 /var/log/dpkg.log  # Last 50 lines
```

## 3. I/O Redirection

Redirection changes where a command receives input or sends output.

### Standard file descriptors

  Descriptor   Name     Meaning
  ------------ -------- -----------------
  `0`          stdin    Standard input
  `1`          stdout   Standard output
  `2`          stderr   Standard error

### Common redirection operators

  --------------------------------------------------------------------------
  Operator                Purpose                 Example
  ----------------------- ----------------------- --------------------------
  `>`                     Sends output to a file, `echo "hello" > myfile`
                          overwriting it.         

  `>>`                    Appends output to a     `echo "buddy" >> myfile`
                          file.                   

  `<`                     Takes input from a      `cat < data1`
                          file.                   

  `2>`                    Sends error output to a `command 2> error.txt`
                          file.                   

  `&>`                    Sends standard output   `command &> output.txt`
                          and errors to one file  
                          in Bash.                
  --------------------------------------------------------------------------

### Examples

**Combine two files into a third file:**

``` bash
cat data1 data2 > data3
```

**Save matching records to a file:**

``` bash
grep "Doctor" users.csv > doctor_data
```

**Overwrite versus append:**

``` bash
echo "hello" > myfile
echo "buddy" >> myfile
```

The first command replaces existing content. The second adds a new line
at the end.

**Redirect errors:**

``` bash
cd /root/ 2> error.txt
cat error.txt
```

The error message is written to `error.txt` instead of being displayed
on the terminal.

**Redirect both output and errors (Bash):**

``` bash
command &> output.txt
```

**Take input from a file:**

``` bash
cat < data1
```

## 4. Pipes and Command-Line Utilities

A pipe (`|`) sends the output of one command as the input of another
command.

### Common examples

**Combine and sort two files:**

``` bash
cat names.txt countries.txt | sort
```

**Remove duplicate lines:**

``` bash
cat names.txt | sort | uniq
```

`uniq` removes adjacent duplicate lines. Sorting first brings matching
lines together.

**Display lines 31 to 37:**

``` bash
head -n 37 data.txt | tail -n 7
```

**Use `tee` to display and save output:**

``` bash
ls | tee test.txt
```

`tee` sends the output to the terminal and writes it to `test.txt` at
the same time.

**Use `xargs` to turn input into command arguments:**

``` bash
cat file_names.txt | xargs touch
```

This passes names from the file as arguments to `touch`.

## 5. Grep: Searching Text

`grep` searches files or input for lines matching a pattern. It is
similar to the Find feature in a text editor.

  Option   Purpose                           Example
  -------- --------------------------------- ------------------------------
  `-i`     Ignores letter case.              `grep -i "error" logs.txt`
  `-v`     Shows lines that do not match.    `grep -v "error" logs.txt`
  `-c`     Counts matching lines.            `grep -c "Doctor" users.csv`
  `-w`     Matches a whole word.             `grep -w "Carly" users.csv`
  `-n`     Displays matching line numbers.   `grep -n "Carly" users.csv`

## 6. Find: Searching for Files

The `find` command searches for files and directories.

**Syntax:**

``` bash
find [path] [expression]
```

A dot (`.`) means the current directory.

  ------------------------------------------------------------------------
  Option                  Purpose                 Example
  ----------------------- ----------------------- ------------------------
  `-size`                 Searches by file size.  `find . -size 4M`

  `-name`                 Searches by exact,      `find . -name "a123"`
                          case-sensitive name or  
                          pattern.                

  `-iname`                Searches by name        `find . -iname "a123"`
                          without case            
                          sensitivity.            

  `-user`                 Searches for files      `find . -user root`
                          owned by a user.        

  `-empty`                Finds empty files and   `find . -empty`
                          directories.            
  ------------------------------------------------------------------------

**Note:** `find . -size 4M` matches files of a size of 4 MiB according
to `find`'s size rules; use `+4M` or `-4M` for greater-than or less-than
searches.

## 7. Text Editors: Vim and Nano

### Vim

Vim is a modal text editor. Its main modes include Normal mode, Insert
mode, and Command-line mode.

Open a file:

``` bash
vim filename.txt
```

  Key or command   Action
  ---------------- --------------------------------------------
  `i`              Enters Insert mode.
  `Esc`            Returns to Normal mode.
  `:w`             Saves the file.
  `:q`             Quits.
  `:wq`            Saves and quits.
  `:q!`            Quits without saving.
  `dd`             Deletes the current line.
  `yy`             Copies the current line.
  `p`              Pastes below the cursor.
  `u`              Undoes the previous change.
  `/pattern`       Searches forward for a pattern.
  `n` / `N`        Moves to the next / previous search match.

### Nano

Nano is a simple, modeless terminal text editor. Its shortcuts are shown
at the bottom of the screen.

Open a file:

``` bash
nano filename.txt
```

  Shortcut     Action
  ------------ --------------------------------------------
  `Ctrl + O`   Saves (Write Out); press Enter to confirm.
  `Ctrl + X`   Exits Nano.
  `Ctrl + W`   Searches for text.
  `Ctrl + K`   Cuts the current line.
  `Ctrl + U`   Pastes the cut text.
  `Ctrl + _`   Goes to a line number.

## 8. Linux Process Management

A **process** is a program that is currently running. Linux provides
commands to inspect, control, prioritize, and monitor processes.

### A. Job control

These commands manage jobs started from the current shell.

  Command   Purpose                                    Example
  --------- ------------------------------------------ -----------------
  `jobs`    Lists jobs managed by the current shell.   `jobs`
  `bg`      Resumes a stopped job in the background.   `bg` or `bg %2`
  `fg`      Brings a job to the foreground.            `fg` or `fg %2`

Example:

``` bash
sleep 100 &
jobs
```

To stop a foreground job temporarily, press `Ctrl + Z`. Then use:

``` bash
jobs
bg
```

To bring it back to the foreground:

``` bash
fg
```

**Job ID versus PID:** A job ID (such as `%1`) is used by shell
job-control commands. A PID is the system-wide process ID.

### B. Process inspection

  -----------------------------------------------------------------------
  Command                             Purpose
  ----------------------------------- -----------------------------------
  `ps`                                Displays processes associated with
                                      the current terminal/session.

  `ps -e`                             Displays all processes.

  `ps -ef`                            Displays all processes in
                                      full-format detail, including PID
                                      and PPID.

  `ps -p PID`                         Displays a selected process.

  `ps -p PID -f`                      Displays detailed information for
                                      one selected process.

  `ps -o pid,ni,cmd`                  Selects the PID, nice value, and
                                      command columns.

  `ps aux`                            Displays processes with CPU and
                                      memory usage information.
  -----------------------------------------------------------------------

Important columns:

-   `PID`: Process ID.
-   `PPID`: Parent Process ID.
-   `UID` / `USER`: Process owner.
-   `TTY`: Associated terminal.
-   `TIME`: CPU time used.
-   `CMD` / `COMMAND`: Command that started the process.
-   `%CPU`: CPU usage.
-   `%MEM`: Memory usage.
-   `NI`: Nice value.

**Difference:** - `ps -ef`: Useful for process details and parent-child
relationships. - `ps aux`: Useful for viewing CPU and memory usage.

### C. Sending signals with `kill`

`kill` sends a signal to a process identified by its PID.

  Command            Meaning
  ------------------ -----------------------------------------
  `kill PID`         Sends SIGTERM (normally signal 15).
  `kill -15 PID`     Requests graceful termination.
  `kill -9 PID`      Sends SIGKILL for forceful termination.
  `kill -l`          Lists available signal names/numbers.
  `kill -STOP PID`   Stops/suspends a process.
  `kill -CONT PID`   Continues a stopped process.

**Important signals:**

  Signal      Number Meaning
  --------- -------- ------------------------------
  SIGHUP           1 Hangup
  SIGINT           2 Interrupt
  SIGKILL          9 Force termination
  SIGTERM         15 Graceful termination request
  SIGCONT         18 Continue a stopped process
  SIGSTOP         19 Stop a process

**Recommended approach:** Try `kill PID` first. Use `kill -9 PID` only
if the process does not respond to normal termination. SIGKILL does not
allow the process to perform its own cleanup.

### D. Process priority: `nice` and `renice`

The nice value influences CPU scheduling priority.

-   Range: `-20` to `+19`.
-   Lower nice value: more favorable scheduling priority.
-   Higher nice value: less favorable scheduling priority.
-   Default nice value is usually `0`.

**Start a process with a chosen nice value:**

``` bash
nice -n 10 sleep 300 &
```

**Change the nice value of an existing process:**

``` bash
renice 10 -p PID
```

  Command    Purpose
  ---------- ----------------------------------------------
  `nice`     Sets the nice value when starting a process.
  `renice`   Changes the nice value of a running process.

A nice value of 10 does **not** mean the process uses 10% CPU. It
affects scheduling priority.

### E. Live monitoring: `top`

`top` displays a continuously updating view of processes and system
resource usage.

Start:

``` bash
top
```

Useful keys while `top` is running:

  Key   Action
  ----- --------------------------------------------
  `q`   Quits `top`.
  `k`   Prompts for a PID and signal to send.
  `c`   Toggles command display format.
  `u`   Filters by username.
  `f`   Manages displayed fields.
  `z`   Toggles color display.
  `b`   Toggles highlighting of running processes.

**Remember:** `ps` gives a snapshot; `top` provides live, continuously
refreshed monitoring.

## 9. Quick Revision

### Directory and file commands

-   `pwd` --- show current directory.
-   `cd` --- change directory.
-   `mkdir` --- create directory.
-   `rmdir` --- remove an empty directory.
-   `ls` --- list contents.
-   `touch` --- create file/update timestamp.
-   `cp` --- copy.
-   `mv` --- move or rename.
-   `rm` --- remove.
-   `cat` --- display or combine file contents.
-   `head` / `tail` --- show beginning / end of a file.

### Text and data commands

-   `grep` --- search matching text.
-   `find` --- search for files and directories.
-   `sort` --- sort lines.
-   `uniq` --- remove adjacent duplicate lines.
-   `tee` --- display and save output.
-   `xargs` --- convert input into command arguments.
-   `|` --- pass output to another command.

### Process-management commands

-   `jobs` --- list current shell jobs.
-   `bg` --- continue a stopped job in the background.
-   `fg` --- bring a job to the foreground.
-   `ps` --- inspect processes.
-   `kill` --- send a signal to a process.
-   `nice` --- set priority when starting a process.
-   `renice` --- change priority of a running process.
-   `top` --- monitor processes live.

------------------------------------------------------------------------

**Study tip:** Practise commands in a test directory or virtual machine.
Be especially careful with commands that delete files or affect system
processes.
