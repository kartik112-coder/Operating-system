# Experiment 2: Combined Redirection and Stream Editor (`sed`)

## Aim

To practise command chaining, output redirection, and text processing
using the Linux stream editor `sed`.

## 1. Command Chaining and Redirection

Multiple commands can be executed sequentially using a semicolon (`;`).

### Example: Display the date and save logged-in users

``` bash
date; who > logged_users.txt
```

**Explanation:** - `date` displays the current system date and time. -
`;` runs the next command after the first command finishes. - `who`
displays users currently logged in. - `>` redirects the output of `who`
into `logged_users.txt`, replacing its previous contents if the file
exists.

View the saved output:

``` bash
cat logged_users.txt
```

## 2. Introduction to `sed`

`sed` stands for **Stream Editor**. It is a non-interactive text editor
that reads input line by line, applies editing commands, and writes the
result to standard output by default.

**Basic syntax:**

``` bash
sed [OPTIONS] 'COMMAND' filename
```

### Important concepts

-   `-n`: Suppresses automatic printing of input lines.
-   `p`: Prints the selected line or lines.
-   `d`: Deletes selected lines from the output stream.
-   `s`: Substitutes matching text.

## 3. Printing Specific Lines

### Print a particular line

``` bash
sed -n '3p' file.txt
```

Displays only line 3.

### Print a range of lines

``` bash
sed -n '2,5p' file.txt
```

Displays lines 2 through 5.

**Remember:** `-n` prevents automatic printing, while `p` prints only
the selected lines.

## 4. Deleting Lines

The `d` command deletes selected lines from the output stream. It does
not modify the original file unless in-place editing is used.

  Command                 Purpose
  ----------------------- -------------------------------------------
  `sed '3d' file.txt`     Deletes line 3 from the displayed output.
  `sed '1d' file.txt`     Deletes the first line.
  `sed '$d' file.txt`     Deletes the last line.
  `sed '2,4d' file.txt`   Deletes lines 2 through 4.

## 5. Text Substitution

The substitution command uses this format:

``` bash
sed 's/old/new/' file.txt
```

It replaces the first occurrence of `old` with `new` on each line.

### First occurrence

``` bash
sed 's/apple/mango/' fruits.txt
```

Replaces only the first matching occurrence on each line.

### All occurrences in each line

``` bash
sed 's/apple/mango/g' fruits.txt
```

The `g` flag means global: replace every matching occurrence on each
line.

### Modify the original file

``` bash
sed -i 's/apple/mango/g' fruits.txt
```

The `-i` option edits the file directly.

### Create a backup before modifying

``` bash
sed -i.bak 's/apple/mango/g' fruits.txt
```

This modifies `fruits.txt` and creates a backup named `fruits.txt.bak`.

**Important:** Without `-i`, `sed` normally prints the edited result but
leaves the original file unchanged.

## 6. Regular Expressions and Anchors

Regular expressions help `sed` identify text patterns.

  -------------------------------------------------------------------------------
  Symbol / Pattern        Meaning                 Example
  ----------------------- ----------------------- -------------------------------
  `^`                     Beginning of a line     `sed -n '/^Hello/p' file.txt`

  `$`                     End of a line           `sed -n '/world$/p' file.txt`

  `.`                     Any single character    `sed -n '/c.t/p' file.txt`

  `[^ ]`                  Any character except a  Matches one non-space
                          space                   character.

  `*`                     Zero or more            `a*` matches zero or more `a`
                          repetitions of the      characters.
                          preceding expression    
  -------------------------------------------------------------------------------

### Examples

**Find lines beginning with `Hello`:**

``` bash
sed -n '/^Hello/p' file.txt
```

**Find lines ending with `world`:**

``` bash
sed -n '/world$/p' file.txt
```

**Find a pattern with any one character between `c` and `t`:**

``` bash
sed -n '/c.t/p' file.txt
```

**Note:** In basic regular expressions, `[^ ]*` means zero or more
non-space characters. It can be used to match a sequence of non-space
characters.

## 7. Capture Groups and Backreferences

Capture groups allow part of a matched pattern to be saved and reused.

In `sed` basic regular expressions: - `\(...\)` creates a capture
group. - `\1` refers to the first captured group. - `\2` refers to the
second captured group.

### Example: Swap two words

``` bash
echo "Linux Ubuntu" | sed 's/\(Linux\) \(Ubuntu\)/\2 \1/'
```

**Output:**

``` text
Ubuntu Linux
```

**Explanation:** 1. `\(Linux\)` captures the first word as group 1. 2.
`\(Ubuntu\)` captures the second word as group 2. 3. `\2 \1` prints the
second captured word followed by the first.

## 8. Quick Revision

  -----------------------------------------------------------------------
  Concept                 Command / Symbol        Use
  ----------------------- ----------------------- -----------------------
  Sequential commands     `;`                     Runs commands one after
                                                  another.

  Output redirection      `>`                     Saves output to a file.

  Suppress automatic      `-n`                    Prevents default line
  printing                                        output.

  Print selected lines    `p`                     Prints matching lines.

  Delete lines            `d`                     Removes selected lines
                                                  from output.

  Substitute text         `s/old/new/`            Replaces the first
                                                  match on each line.

  Global substitution     `s/old/new/g`           Replaces all matches on
                                                  each line.

  In-place editing        `-i`                    Modifies the original
                                                  file.

  Backup while editing    `-i.bak`                Saves a backup before
                                                  modification.

  Line beginning          `^`                     Matches the start of a
                                                  line.

  Line ending             `$`                     Matches the end of a
                                                  line.

  Any character           `.`                     Matches one character.

  Capture group           `\(...\)`               Stores a matched text
                                                  segment.

  Backreference           `\1`, `\2`              Reuses captured text.
  -----------------------------------------------------------------------

## Conclusion

This experiment introduces command chaining, output redirection, and the
`sed` stream editor. It demonstrates how to print and delete selected
lines, replace text, edit files safely with backups, and use regular
expressions and backreferences for text manipulation.
