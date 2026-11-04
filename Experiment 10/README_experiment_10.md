# Experiment 10: Observing Virtual Memory Addresses in C

**Model Institute of Engineering & Technology (Autonomous)**  
**Course:** COM-312 Operating System Lab

## 1. Aim and Objectives

**Aim:** To write, compile, and execute a C program that prints the memory address of a local variable inside an infinite loop, run 3–4 instances concurrently in the background, and analyze the resulting addresses with Address Space Layout Randomization (ASLR) enabled and disabled.

### Objectives

1. Understand that pointers printed by C programs are virtual addresses, not raw physical DRAM addresses.
2. Run multiple independent processes concurrently using the shell background operator (`&`) and monitor them using `pgrep`.
3. Observe how ASLR randomizes stack addresses between program executions.
4. Disable ASLR for the test using `setarch $(uname -m) -R` and observe repeated virtual address values.
5. Understand how the CPU Memory Management Unit (MMU) and per-process page tables translate virtual addresses and isolate process memory.

## 2. Theoretical Foundation

### 2.1 Virtual Address Space

In Linux, a process does not directly access physical RAM. Each process receives its own virtual address space, which provides an isolated view of memory.

A process's virtual address space includes areas such as:

- Program code (`.text`)
- Initialized data (`.data`)
- Uninitialized data (`.bss`)
- Dynamic heap memory
- Shared libraries (`.so`)
- Execution stack

Two independent processes can use the same numerical virtual address without interfering with each other. The operating system and hardware memory-management mechanisms maintain the isolation.

### 2.2 Virtual Addresses vs. Physical Addresses

A C pointer, such as `&var`, contains a virtual memory address. When the CPU accesses memory, its Memory Management Unit (MMU) translates the virtual address through the active process's page tables to a physical memory frame.

**Core concept:** Standard user-space C code works with virtual memory addresses. It does not directly expose or manipulate raw physical RAM addresses.

### 2.3 Address Space Layout Randomization (ASLR)

ASLR is an operating-system security feature that makes memory locations less predictable. It randomizes the starting locations of areas such as the stack, heap, and shared memory mappings when a program is launched.

With ASLR enabled, repeated executions can print different addresses for the same stack variable. Disabling ASLR for the experiment makes the virtual memory layout more predictable.

## 3. C Program: `virtual_addr.c`

The program declares a local integer variable, prints the process ID and the variable's address, and then remains running in an infinite loop. It sleeps for one second on each iteration so it does not continuously consume a CPU core.

```c
/*
 * Filename: virtual_addr.c
 * Description: Demonstrating process virtual address space isolation in Linux.
 */

#include <stdio.h>
#include <unistd.h>

int main() {
    // Allocate an integer variable on the process stack frame
    int var = 42;

    // Print the Process ID (PID) and the hexadecimal address of 'var'
    printf("PID: %d | Address: %p\n", getpid(), (void*)&var);
    fflush(stdout);

    // Infinite loop keeps the process active
    while (1) {
        sleep(1); // Yield CPU cycles each second
    }

    return 0;
}
```

## 4. Step-by-Step Execution Procedure

### Step 1: Compile the program

```bash
gcc virtual_addr.c -o virtual_addr
```

### Step 2: Start four concurrent background instances with ASLR active

Run the executable four times using the shell background operator (`&`):

```bash
./virtual_addr &
./virtual_addr &
./virtual_addr &
./virtual_addr &
```

### Sample output with ASLR active

```text
PID: 5101 | Address: 0x7ffd8a3e41ac
PID: 5102 | Address: 0x7ffc1b9028dc
PID: 5103 | Address: 0x7ffe44d1953c
PID: 5104 | Address: 0x7ffdf012b76c
```

The PIDs and addresses above are illustrative values from the lab material. Your output will differ.

### Step 3: Verify the background processes

Use `pgrep` to check that the instances are running:

```bash
pgrep -a virtual_addr
```

Example:

```text
5101 ./virtual_addr
5102 ./virtual_addr
5103 ./virtual_addr
5104 ./virtual_addr
```

Each process prints its address once and then sleeps repeatedly, remaining active until terminated.

### Step 4: Terminate the ASLR-enabled instances

```bash
killall virtual_addr
```

### Step 5: Compile and execute with ASLR disabled

Compile the program as a non-PIE executable, then launch four instances using `setarch` with the randomization flag:

```bash
gcc -no-pie virtual_addr.c -o virtual_addr_fixed

setarch $(uname -m) -R ./virtual_addr_fixed &
setarch $(uname -m) -R ./virtual_addr_fixed &
setarch $(uname -m) -R ./virtual_addr_fixed &
setarch $(uname -m) -R ./virtual_addr_fixed &
```

### Sample output with ASLR disabled

```text
PID: 5210 | Address: 0x7fffffffde4c
PID: 5211 | Address: 0x7fffffffde4c
PID: 5212 | Address: 0x7fffffffde4c
PID: 5213 | Address: 0x7fffffffde4c
```

In the supplied observation, all four independent processes print the same virtual address when ASLR is disabled.

**Note:** Disabling ASLR with `setarch` may be restricted by the environment or system security policy. The exact address and whether the command is permitted depend on the system configuration.

### Step 6: Clean up the background instances

```bash
killall virtual_addr_fixed
```

## 5. Line-by-Line Code Breakdown

| Code statement | Operating system purpose |
|---|---|
| `#include <stdio.h>` | Provides standard I/O declarations such as `printf()` and `fflush()`. |
| `#include <unistd.h>` | Provides Unix interfaces such as `getpid()` and `sleep()`. |
| `int var = 42;` | Declares an automatic local integer variable in the process's stack frame. |
| `(void*)&var` | Obtains the variable's address and casts it to `void *` for use with `%p`. |
| `getpid()` | Retrieves the PID of the current process. |
| `printf(..., getpid(), (void*)&var)` | Prints the PID and the variable's virtual address. |
| `fflush(stdout);` | Flushes standard output so the line appears before the program sleeps. |
| `while (1)` | Keeps the process running indefinitely. |
| `sleep(1);` | Suspends execution for one second on each loop iteration. |
| `return 0;` | Normal exit statement; it is not reached while the infinite loop continues. |

## 6. In-Depth Observation and Analysis

### 6.1 ASLR Enabled

The supplied experiment records these addresses:

| PID | Printed virtual address |
|---:|---|
| 5101 | `0x7ffd8a3e41ac` |
| 5102 | `0x7ffc1b9028dc` |
| 5103 | `0x7ffe44d1953c` |
| 5104 | `0x7ffdf012b76c` |

The addresses share a high-order prefix but differ in their other hexadecimal digits. This illustrates how ASLR can vary stack locations between program launches.

### 6.2 ASLR Disabled

The supplied experiment records the same address for all four instances:

| PID | Printed virtual address |
|---:|---|
| 5210 | `0x7fffffffde4c` |
| 5211 | `0x7fffffffde4c` |
| 5212 | `0x7fffffffde4c` |
| 5213 | `0x7fffffffde4c` |

The identical pointer values do not mean the processes share the same physical memory. Each process has its own virtual address space and page-table mappings.

## 7. Why Processes Do Not Collide: MMU and Page Tables

The Linux kernel and the CPU's Memory Management Unit (MMU) work together to keep process memory isolated.

### 7.1 Per-process page tables

The operating system maintains page-table mappings for each process. When a process accesses a virtual address, the MMU translates it using the page tables active for that process.

Thus, the same virtual address in two different processes can map to different physical memory frames.

### 7.2 Address-space switching

During a context switch, the operating system changes the active address-space context so that subsequent memory accesses use the next process's mappings. On x86-64, this involves the CPU's page-table root register, CR3.

The lab material illustrates the concept as follows:

```text
Process 1 [PID 5210]: Virtual 0x7fffffffde4c
    -> Page Table 5210 -> Physical DRAM Frame 0x0A2B000

Process 2 [PID 5211]: Virtual 0x7fffffffde4c
    -> Page Table 5211 -> Physical DRAM Frame 0x1F4C000

Process 3 [PID 5212]: Virtual 0x7fffffffde4c
    -> Page Table 5212 -> Physical DRAM Frame 0x2E8D000

Process 4 [PID 5213]: Virtual 0x7fffffffde4c
    -> Page Table 5213 -> Physical DRAM Frame 0x3A01000
```

These physical frame values are illustrative examples from the lab material, not measurements made by the C program.

### 7.3 Hardware memory protection

Each process is restricted to the memory mappings available in its own address space. An invalid or unauthorized memory access can cause a hardware exception that the operating system may report as a segmentation fault (`SIGSEGV`).

## 8. Quick Summary for the Whiteboard

1. **Virtual is not physical:** C pointers contain virtual addresses, not physical DRAM addresses.
2. **ASLR:** Randomizes memory layout to make addresses harder to predict.
3. **Disabling ASLR:** `setarch $(uname -m) -R` requests execution without address-space randomization.
4. **Memory protection:** The MMU translates virtual addresses using page tables, maintaining isolation between processes even when their virtual addresses are identical.

## 9. Observation Table

The following table reproduces the experiment's illustrative observations.

| Instance | ASLR mode | PID | Printed virtual address | Physical DRAM allocation (illustrative label) | Memory collision |
|---|---|---:|---|---|---|
| Process 1 | Active (default) | 5101 | `0x7ffd8a3e41ac` | DRAM Frame A | None (protected) |
| Process 2 | Active (default) | 5102 | `0x7ffc1b9028dc` | DRAM Frame B | None (protected) |
| Process 3 | Active (default) | 5103 | `0x7ffe44d1953c` | DRAM Frame C | None (protected) |
| Process 4 | Active (default) | 5104 | `0x7ffdf012b76c` | DRAM Frame D | None (protected) |
| Process 1 | Disabled (`-R`) | 5210 | `0x7fffffffde4c` | DRAM Frame W | None (protected) |
| Process 2 | Disabled (`-R`) | 5211 | `0x7fffffffde4c` | DRAM Frame X | None (protected) |
| Process 3 | Disabled (`-R`) | 5212 | `0x7fffffffde4c` | DRAM Frame Y | None (protected) |
| Process 4 | Disabled (`-R`) | 5213 | `0x7fffffffde4c` | DRAM Frame Z | None (protected) |

## 10. Result

The `virtual_addr.c` program demonstrates how to print the PID and virtual address of a local variable while keeping a process running. The supplied observations show varying stack addresses with ASLR enabled and identical virtual address values across concurrent instances in the ASLR-disabled test. The experiment illustrates that virtual addresses are translated through per-process page tables by the MMU, allowing separate processes to use the same numerical virtual address without colliding in physical memory.

## 11. Files Used

| Filename | Description |
|---|---|
| `virtual_addr.c` | Prints the process ID and virtual address of a local variable, then remains active. |
| `virtual_addr` | Executable built for the ASLR-enabled test. |
| `virtual_addr_fixed` | Non-PIE executable used for the ASLR-disabled test. |

## 12. Complete Command Reference

```bash
# Compile
gcc virtual_addr.c -o virtual_addr

# Start four instances with ASLR active
./virtual_addr &
./virtual_addr &
./virtual_addr &
./virtual_addr &

# List active instances
pgrep -a virtual_addr

# Stop ASLR-enabled instances
killall virtual_addr

# Compile a non-PIE executable
gcc -no-pie virtual_addr.c -o virtual_addr_fixed

# Start four instances with ASLR disabled
setarch $(uname -m) -R ./virtual_addr_fixed &
setarch $(uname -m) -R ./virtual_addr_fixed &
setarch $(uname -m) -R ./virtual_addr_fixed &
setarch $(uname -m) -R ./virtual_addr_fixed &

# Stop ASLR-disabled instances
killall virtual_addr_fixed
```
