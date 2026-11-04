# Experiment 8: Observing Process Memory Maps Using `pmap`

This experiment examines the memory mappings of the CPU-bound and I/O-bound C programs from Experiment 7. It uses `pmap` and `pmap -x` to inspect virtual address regions, permissions, resident memory, dirty memory, and mapped files or libraries.

## 1. Aim

To examine and compare the memory mappings of the CPU-bound and I/O-bound programs using the Linux `pmap` command.

## 2. Requirements

- Linux
- GCC-compiled `io_bound` and `cpu_bound` programs from Experiment 7
- `pmap`, `pgrep`, and `kill` utilities
- Terminal access

## 3. Procedure

### Step 1: Run both programs

From the directory containing the compiled executables, run:

```bash
./io_bound &
./cpu_bound &
```

The `&` symbol starts each program in the background.

### Step 2: Find their process IDs (PIDs)

```bash
pgrep -af 'io_bound|cpu_bound'
```

Example output:

```text
2451 ./io_bound
2452 ./cpu_bound
```

These PIDs are examples only. Use the actual PIDs shown on your system.

### Step 3: Display the CPU-bound process memory map

```bash
pmap <CPU_PID>
```

For detailed output:

```bash
pmap -x <CPU_PID>
```

### Step 4: Display the I/O-bound process memory map

```bash
pmap <IO_PID>
```

For detailed output:

```bash
pmap -x <IO_PID>
```

Replace `<CPU_PID>` and `<IO_PID>` with the actual process IDs.

## 4. What to Expect in `pmap` Output

Both programs are C programs running under Linux, so their memory maps contain broadly similar types of regions, even though their instructions and runtime behavior differ.

| Memory region | Meaning |
|---|---|
| Executable code | Machine instructions generated from the C source |
| Data/BSS | Storage for applicable global or static variables |
| Stack | Function-call information and local variables, where stored on the stack |
| Heap | Dynamic memory allocations, if any |
| Shared libraries | Libraries required by the program, such as the C standard library |
| Other mappings | Additional regions used by the runtime and operating system |

In the CPU-bound program, `counter` is a local variable. In the I/O-bound program, `fp` is a local file pointer. Both programs may have stack mappings, although the compiler can choose how individual variables are stored.

Neither program explicitly calls `malloc()`, so neither needs a large dynamic allocation for its main task.

## 5. Understanding `pmap -x`

Run:

```bash
pmap -x <PID>
```

Each row describes one contiguous region of the process's virtual address space.

### 5.1 Column summary

| Column | Meaning |
|---|---|
| Address | Starting virtual address of the memory region |
| Kbytes | Total size of the region, in kilobytes |
| RSS | Resident Set Size: how much of the region is currently resident in RAM |
| Dirty | Amount of memory in the region reported as dirty |
| Mode | Access permissions for the region |
| Mapping | File, library, or memory region associated with the mapping |

### 5.2 Address

The `Address` column identifies where a memory region starts in the process's virtual address space. It is not necessarily a physical RAM address. Each row represents one mapping, and different regions can have different starting addresses.

### 5.3 Kbytes

`Kbytes` gives the total size of that particular mapping, not the total memory of the process. A large mapped size does not mean all of that memory is currently loaded into physical RAM.

### 5.4 RSS

RSS stands for **Resident Set Size**. It shows how much of the mapping is resident in physical RAM at the time of measurement.

For example:

```text
Address             Kbytes  RSS
000076ec1b228000    1632    1056
```

This mapping is 1632 KB in total, with 1056 KB resident in RAM at that moment. RSS is not the same as private memory because shared pages can also be counted.

### 5.5 Dirty

The `Dirty` column reports the amount of memory counted as dirty, associated with modified memory pages.

For example:

```text
Kbytes  RSS  Dirty
4       4    4
```

Dirty memory does not simply mean data that has not been saved to disk. Anonymous and file-backed dirty pages can behave differently. A value of `0` means no dirty memory was reported for that mapping.

### 5.6 Mode: memory permissions

The `Mode` column shows which operations are allowed on the region.

| Symbol | Meaning |
|---|---|
| `r` | Read permission |
| `w` | Write permission |
| `x` | Execute permission |
| `-` | Permission not granted |

Examples:

- `r----` — read permission.
- `r-x--` — read and execute permissions.
- `rw---` — read and write permissions.

These permissions help protect memory regions from inappropriate access, such as accidentally overwriting program code.

### 5.7 Mapping

The `Mapping` column identifies the file, library, or memory type associated with a region.

| Mapping | Meaning |
|---|---|
| `exp7a` | Example name of a compiled executable's memory mapping |
| `libc.so.6` | C standard library mapping |
| `ld-linux-x86-64.so.2` | Dynamic linker/loader mapping |
| `[ anon ]` | Anonymous mapping, not directly associated with a named file |
| `[ stack ]` | Main thread's stack mapping |

Exact mapping names can differ between Linux systems.

## 6. Reading a Complete Row

Consider this illustrative row:

```text
000076ec1b228000 1632 1056 0 r-x-- libc.so.6
```

Read it from left to right:

| Column | Value | Meaning |
|---|---|---|
| Address | `000076ec1b228000` | Starting virtual address |
| Kbytes | `1632` | Total mapping size: 1632 KB |
| RSS | `1056` | Resident memory: 1056 KB |
| Dirty | `0` | No dirty memory reported |
| Mode | `r-x--` | Read and execute permissions |
| Mapping | `libc.so.6` | C standard library mapping |

## 7. Understanding the Total Row

An illustrative total row is:

```text
total kB 2772 1744 96
```

It means:

- `2772 KB` — total mapped size of all listed mappings.
- `1744 KB` — total reported RSS.
- `96 KB` — total reported dirty memory.

These are totals across the listed mappings, not the values of one individual mapping.

## 8. Illustrative Output

The following shortened example is for explanation only. Addresses, sizes, and mapping names will differ on your system.

```text
Address             Kbytes  RSS   Dirty  Mode   Mapping
0000563c1a2b4000    4       4     0      r----  cpu_bound
0000563c1a2b5000    4       4     0      r-x--  cpu_bound
0000563c1a2b7000    4       4     4      rw---  cpu_bound
00007f8e3c000000    1892    1124  0      r-x--  libc.so.6
00007ffd5e2a1000    132     12    12     rw---  [ stack ]
...
total kB             2500    1400  120
```

For the `r-x--` executable mapping, the example shows a 4 KB region, all 4 KB resident, no dirty memory reported, and read/execute permissions.

## 9. Observation Table

Run `pmap -x` for both processes and record your actual readings.

| Parameter | CPU-bound process | I/O-bound process |
|---|---|---|
| Executable mapping | Present | Present |
| Shared libraries | Typically present | Typically present |
| Stack | Typically present | Typically present |
| Total mapped memory | Record actual value | Record actual value |
| RSS | Record actual value | Record actual value |

Do not copy illustrative values as if they were measurements from your machine. The precise output depends on the Linux system and the process.

## 10. Analysis

The CPU-bound program performs repeated calculations, while the I/O-bound program writes to a file and sleeps. These behavioral differences primarily affect CPU utilization and waiting time, not necessarily the size of the memory map.

The `pmap` command lets you inspect how memory is mapped for each process. It does not directly measure CPU share; use `top` for CPU utilization and process activity.

## 11. Result

The memory maps of both programs were examined using `pmap` and `pmap -x`. Their executable mappings, stack, shared libraries, and memory statistics were observed and compared.

## 12. Cleanup

Identify the relevant PIDs:

```bash
pgrep -af 'io_bound|cpu_bound'
```

Terminate the processes using their actual PIDs:

```bash
kill <IO_PID> <CPU_PID>
```

This prevents the infinite loops from continuing to run after the experiment.

## Key Takeaway

`pmap -x` provides a detailed view of a process's virtual memory mappings:

- **Address** tells where a mapping starts.
- **Kbytes** tells how large the mapping is.
- **RSS** tells how much is resident in physical RAM.
- **Dirty** reports memory counted as dirty.
- **Mode** shows the mapping's permissions.
- **Mapping** identifies the executable, library, or memory region.

Use `pmap` to inspect memory layout and `top` to observe CPU utilization.
