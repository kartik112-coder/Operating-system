# Experiment 5: String Operations in Shell Script & C

## Aim

To write a Bash shell script and an equivalent C program to: - Find the
length of a string entered by the user. - Extract a substring using a
starting position and a required number of characters. - Compare the
execution time of both programs using the Linux `time` command.

## Part A --- Shell Script (`exp7.sh`)

### Program

``` bash
#!/bin/bash
read -p "Enter a string: " str
echo "Length of string: ${#str}"
read -p "Enter starting position: " start
read -p "Enter number of characters: " length
echo "Substring: ${str:$start:$length}"
```

### Explanation

1.  `#!/bin/bash` specifies that Bash should run the script.
2.  `read -p "Enter a string: " str` displays a prompt and stores the
    input in `str`.
3.  `${#str}` returns the length of the string.
4.  `read -p "Enter starting position: " start` reads the starting
    index.
5.  `read -p "Enter number of characters: " length` reads how many
    characters to extract.
6.  `${str:$start:$length}` extracts the requested substring.

The general substring syntax is:

``` bash
${string:start:length}
```

Bash string positions start at index `0`.

### Example

Input:

``` text
Enter a string: OperatingSystem
Enter starting position: 9
Enter number of characters: 6
```

Output:

``` text
Length of string: 15
Substring: System
```

### Create and Run

``` bash
nano exp7.sh
chmod +x exp7.sh
time ./exp7.sh
```

`chmod +x` gives the script execute permission. `time` reports its
execution time.

## Part B --- C Program (`exp7.c`)

### Program

``` c
#include <stdio.h>

int main()
{
    char str[100];
    int start, length, i, count = 0;

    printf("Enter a string: ");
    fgets(str, 100, stdin);

    while (str[count] != '\0' && str[count] != '\n')
    {
        count++;
    }

    printf("Length of string: %d\n", count);

    printf("Enter starting position: ");
    scanf("%d", &start);

    printf("Enter number of characters: ");
    scanf("%d", &length);

    printf("Substring: ");

    for (i = start; i < start + length && i < count; i++)
    {
        printf("%c", str[i]);
    }

    printf("\n");
    return 0;
}
```

### Explanation

1.  `#include <stdio.h>` includes the standard input/output library for
    `printf()`, `scanf()`, and `fgets()`.
2.  `int main()` is the starting point of the C program.
3.  `char str[100]` declares a character array to store the input
    string.
4.  `start` stores the starting position; `length` stores the number of
    characters; `i` is the loop variable; `count` tracks the string
    length.
5.  `fgets(str, 100, stdin)` reads the string from standard input.
6.  The `while` loop counts characters until it reaches the newline or
    null character.
7.  `printf()` displays the string length.
8.  `scanf("%d", &start)` reads the starting position. `&start` gives
    the address where the input is stored.
9.  `scanf("%d", &length)` reads the number of characters.
10. The `for` loop prints characters from `start` up to the requested
    length, without going beyond `count`.
11. `%c` prints one character.
12. `return 0` indicates successful completion.

### Compile and Run

``` bash
gcc exp7.c -o exp7
./exp7
```

### Sample Output

``` text
Enter a string: OperatingSystem
Length of string: 15
Enter starting position: 9
Enter number of characters: 6
Substring: System
```

## Comparing Execution Time

Run both programs using `time`:

``` bash
time ./exp7
time ./exp7.sh
```

### Understanding the `time` Output

Example:

``` text
real    0m0.005s
user    0m0.001s
sys     0m0.004s
```

  -----------------------------------------------------------------------
  Field                               Meaning
  ----------------------------------- -----------------------------------
  `real`                              Total elapsed wall-clock time

  `user`                              CPU time spent executing user-level
                                      code

  `sys`                               CPU time spent in kernel/system
                                      calls on behalf of the program
  -----------------------------------------------------------------------

-   **real:** The time from program start to finish, like a stopwatch.
-   **user:** CPU time used for operations performed in user space, such
    as calculations and loops.
-   **sys:** CPU time used for kernel work, including system calls such
    as `read`, `write`, `open`, and `close`.

Execution times can vary between runs because of CPU scheduling, cache
state, terminal input/output, system load, and virtual-machine overhead.
Small variations are normal when running Ubuntu inside VirtualBox.

## Conclusion

In this experiment, we implemented string-length calculation and
substring extraction in both Bash and C. We used Bash parameter
expansion and a C character array with loops to perform the operations.
Finally, we used the Linux `time` command to observe and compare the
execution times of both programs.
