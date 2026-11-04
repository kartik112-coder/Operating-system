# Experiment 7: CPU-Bound vs I/O-Bound C Programs Using `top`

This experiment compares a CPU-bound C program with an I/O-bound C program. It uses Linux process and job-control commands together with `top` to observe CPU utilization, process states, and accumulated CPU time.

## 1. Aim

To write and run a CPU-bound C program and an I/O-bound C program, then compare their CPU utilization, process states, and accumulated CPU time using the `top` command.

## 2. Requirements

- Linux or another POSIX-compatible environment
- GCC compiler
- Terminal access
- `top` and `pgrep` utilities

## 3. Program 1: I/O-Bound Program

**Filename:** `io_bound.c`

```c
#include <stdio.h>
#include <unistd.h>

int main()
{
    FILE *fp;

    printf("Starting I/O task...\n");

    while (1)
    {
        fp = fopen("dummy.txt", "w");

        if (fp != NULL)
        {
            fprintf(fp, "Writing disk data\n");
            fclose(fp);
            printf("File operation done\n");
        }

        sleep(1);
    }

    return 0;
}
```

### How it works

- `stdio.h` provides standard input/output and file-handling functions such as `printf()`, `fopen()`, `fprintf()`, and `fclose()`.
- `unistd.h` provides the POSIX `sleep()` function.
- `FILE *fp` declares a file pointer for the stream returned by `fopen()`.
- `fopen("dummy.txt", "w")` opens the file for writing, creating it if needed and truncating its previous contents.
- The `fp != NULL` check ensures the program only writes when the file was opened successfully.
- `fprintf()` writes text to the file stream.
- `fclose()` closes the stream and flushes the C library's buffered output. It does not guarantee that the data has physically reached the storage device.
- `sleep(1)` suspends the process for approximately one second, so it spends much of its time waiting rather than actively using the CPU.

### Expected output

```text
Starting I/O task...
File operation done
File operation done
File operation done
...
```

The confirmation message normally appears about once per second. Actual timing can vary.

## 4. Program 2: CPU-Bound Program

**Filename:** `cpu_bound.c`

```c
#include <stdio.h>

int main()
{
    long long counter = 0;

    while (1)
    {
        counter++;

        if (counter % 100000000 == 0)
        {
            printf("CPU task running: %lld\n", counter);
        }
    }

    return 0;
}
```

### How it works

- `stdio.h` provides `printf()`.
- `long long counter = 0;` declares and initializes a large integer counter.
- `while (1)` creates an infinite loop.
- `counter++` repeatedly increments the counter.
- `counter % 100000000 == 0` becomes true at each multiple of 100 million.
- `printf()` displays the counter value at those intervals. The `%lld` format specifier prints a `long long` value.

### Expected output

```text
CPU task running: 100000000
CPU task running: 200000000
CPU task running: 300000000
...
```

Most of this program's work is repeated counting, so it generally keeps a CPU core busy.

## 5. Compile the Programs

Run these commands in the directory containing both source files:

```bash
gcc io_bound.c -o io_bound
gcc cpu_bound.c -o cpu_bound
```

## 6. Run Both Programs in the Background

```bash
./io_bound &
./cpu_bound &
```

The `&` symbol starts each process in the background, returning control to the terminal.

## 7. Find the Process IDs

```bash
pgrep -af 'io_bound|cpu_bound'
```

Example output:

```text
2451 ./io_bound
2452 ./cpu_bound
```

The numbers shown are example PIDs. Use the actual PIDs reported on your system.

## 8. Observe CPU Usage with `top`

| Purpose | Command or action | Notes |
|---|---|---|
| Display processes | `top` | Press `P` to sort by CPU utilization. |
| Monitor the two processes | `top -p 2451,2452` | Replace the example PIDs with your actual PIDs. |
| Refresh every second | `top -d 1` | Updates the display once per second. |
| Collect five snapshots | `top -b -n 5 -d 1` | Batch mode is useful for recording readings. |
| View individual CPU cores | Run `top`, then press `1` | Shows utilization for each logical CPU. |

### Expected observations

- `cpu_bound` should generally show high `%CPU`.
- `io_bound` should generally show low `%CPU` because it sleeps for one second after each file operation.
- The CPU-bound process generally accumulates CPU time faster.
- The CPU-bound process is commonly in the running state (`R`) when observed.
- The I/O-bound process is often sleeping (`S`) while waiting between operations.

A single-threaded process may use approximately 100% of one logical CPU. Exact readings depend on system load and `top` display settings.

## 9. Observation Table

Record your actual readings while both programs are running.

| Parameter | CPU-bound program | I/O-bound program |
|---|---|---|
| CPU utilization (`%CPU`) | Generally high | Generally low |
| Process state | Usually `R` when observed | Often `S` during sleep |
| CPU time (`TIME+`) | Increases rapidly | Increases more slowly |
| Main activity | Repeated counter increments | File writing and sleeping |

## 10. Stop the Programs

Find the current PIDs:

```bash
pgrep -af 'io_bound|cpu_bound'
```

Stop each process using its actual PID:

```bash
kill <io_bound_PID> <cpu_bound_PID>
```

Alternatively, if the processes are still controlled by the current shell, use the shell's job-control commands such as `jobs` and `kill %<job_number>`.

## 11. Result

Both C programs can be executed and compared using the `top` command. The CPU-bound program generally consumes more CPU time because it continuously performs calculations. The I/O-bound program generally uses less CPU time because it performs file operations and spends much of its time sleeping.

## Key Takeaway

A **CPU-bound** process spends most of its time performing computations, while an **I/O-bound** process spends much of its time waiting for input/output or other events. The `top` command helps compare their CPU utilization, process states, and accumulated CPU time.
