# Experiment 6: Memory Allocation & Virtual Memory in C

A Linux OS lab demonstrating the difference between **requesting virtual memory** and **actually using physical memory**. The program allocates a user-specified amount of memory, touches one byte in each 4096-byte page, keeps the allocation alive for inspection, and releases it when you press `Ctrl+C`.

## Objectives

- Read a memory size from the command line using `argc` and `argv`.
- Convert the supplied string to a number.
- Allocate heap memory with `malloc()`.
- Touch each 4096-byte page to trigger demand paging.
- Observe memory usage with `free -h` and `watch`.
- Handle `SIGINT` (`Ctrl+C`) and demonstrate memory cleanup.

## Requirements

- Linux
- GCC
- A terminal

## Files

- `memory.c` — the C program shown below.

## Build

Compile the program:

```bash
gcc memory.c -o memory
```

## Run

Pass the requested allocation size in mebibytes (the lab informally calls these MB):

```bash
./memory 500
```

The argument `500` requests:

```text
500 × 1024 × 1024 = 524,288,000 bytes
```

The program stays running so the allocation remains available for observation.

## Monitor memory usage

Open a **second terminal** while the program is running:

```bash
free -h
```

To refresh the display every second:

```bash
watch -n 1 free -h
```

Stop the program with `Ctrl+C`.

## How it works

1. **Command-line arguments:** `argc` counts command-line words, including the program name. `argv[1]` contains the requested size as text.
2. **Input conversion:** `atoi(argv[1])` converts that text to an integer.
3. **Allocation:** `malloc()` requests a block of heap memory and returns its starting address.
4. **Allocation check:** if `malloc()` returns `NULL`, the program reports the failure and exits.
5. **Demand paging:** the loop writes one byte every 4096 bytes. This touches one byte in each 4 KB page, causing the operating system to make the corresponding pages resident as needed.
6. **Observation:** the infinite loop with `sleep(1)` keeps the process alive without busy-spinning, allowing memory usage to be inspected from another terminal.
7. **Cleanup:** pressing `Ctrl+C` sends `SIGINT`. The signal-handling version below responds by releasing the allocation and exiting.

### Why touch every 4096 bytes?

A successful `malloc()` request reserves virtual address space; it does not necessarily mean every page is immediately backed by physical RAM. Writing to one location in each page forces the pages to be accessed, making demand paging visible in the system's memory statistics.

The example uses a 4096-byte page size, which is common on Linux systems. Actual page sizes can vary by architecture and system configuration.

## Complete program (`memory.c`)

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <signal.h>

char *memory;

void handle_sigint(int sig)
{
    free(memory);
    printf("\\nMemory released.\\n");
    exit(0);
}

int main(int argc, char *argv[])
{
    int mb;

    if (argc != 2)
    {
        printf("Usage: %s <memory_in_MB>\\n", argv[0]);
        return 1;
    }

    signal(SIGINT, handle_sigint);

    mb = atoi(argv[1]);
    size_t size = (size_t)mb * 1024 * 1024;

    memory = malloc(size);
    if (memory == NULL)
    {
        printf("Memory allocation failed.\\n");
        return 1;
    }

    for (size_t i = 0; i < size; i += 4096)
    {
        memory[i] = 1;
    }

    printf("Memory allocated successfully.\\n");
    printf("Press Ctrl+C to release memory and exit.\\n");

    while (1)
    {
        sleep(1);
    }

    return 0;
}
```

## Example session

Terminal 1:

```bash
gcc memory.c -o memory
./memory 500
```

Expected program output:

```text
Memory allocated successfully.
Press Ctrl+C to release memory and exit.
```

Terminal 2:

```bash
free -h
watch -n 1 free -h
```

Return to Terminal 1 and press `Ctrl+C`. The program prints:

```text
Memory released.
```

## Important notes

- **MB vs MiB:** `1024 × 1024` bytes is technically one mebibyte (MiB), not one SI megabyte (MB). The lab uses the familiar MB label.
- **Memory is not permanently pinned:** the operating system may reclaim, evict, or swap pages under memory pressure. Touching a page does not grant permanent ownership of a physical RAM frame.
- **Signal-handler safety:** this is a teaching example. Calling `free()`, `printf()`, and `exit()` directly from a signal handler is not strictly async-signal-safe. Production code should generally have the handler set a `volatile sig_atomic_t` flag and perform cleanup in the normal flow of the program.
- **Input validation:** the compact lab example uses `atoi()` and assumes a sensible positive integer. A production program should validate the input and check for conversion errors and size overflow.

## Key takeaway

**Requesting memory, using memory, and releasing memory are separate steps.** `malloc()` requests virtual address space, touching each page causes it to be backed as needed, and cleanup returns the allocation to the allocator. The `free -h` command helps make the effects observable.
