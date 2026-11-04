# Experiment 9: Process Creation and Termination Using C

**Model Institute of Engineering & Technology (Autonomous)**  
**Course:** COM-312 Operating System Lab

## 1. Aim and Objectives

**Aim:** To write and execute C programs that create a child process, wait for its termination, and display its Process ID (PID) under Linux.

### Objectives

1. Understand what a Process ID (PID) is and retrieve it using `getpid()`.
2. Demonstrate process creation using `fork()` and observe its different return values in the parent and child.
3. Synchronize parent and child processes using `wait(NULL)` and display the terminated child's PID.

## 2. Step 1: Understanding a Process ID (PID)

Whenever a C program runs, the operating system assigns the running process a unique identification number called a **Process ID (PID)**. It is similar to a student roll number.

The `getpid()` function, declared in `<unistd.h>`, returns the PID of the currently running process.

### Source Code: `step1_pid.c`

```c
#include <stdio.h>
#include <unistd.h>

int main() {
    // getpid() returns the PID of the currently running process
    printf("I am a simple process. My PID is: %d\n", getpid());
    return 0;
}
```

### Compilation and Execution

```bash
gcc step1_pid.c -o step1_pid
./step1_pid
```

### Sample Output

```text
I am a simple process. My PID is: 3412
```

**Key teaching takeaway:** One running program instance corresponds to one active process with its own unique PID. The displayed PID will vary between executions.

## 3. Step 2: Creating a Child Process with `fork()`

Calling `fork()` creates a child process by duplicating the calling process. After the call, both parent and child continue executing from the point immediately after `fork()`.

The return value distinguishes the two processes:

- **Child process:** `fork()` returns `0`.
- **Parent process:** `fork()` returns the child's positive PID.

### Source Code: `step2_fork.c`

```c
#include <stdio.h>
#include <unistd.h>

int main() {
    pid_t p = fork();

    // From this point onward, both processes execute the same code
    if (p == 0) {
        // Only the child executes this branch
        printf("I am the CHILD process! My own PID: %d\n", getpid());
    } else {
        // Only the parent executes this branch
        printf("I am the PARENT process! fork() gave me child's PID: %d\n", p);
    }

    return 0;
}
```

### Compilation and Execution

```bash
gcc step2_fork.c -o step2_fork
./step2_fork
```

### Sample Output

```text
I am the PARENT process! fork() gave me child's PID: 3450
I am the CHILD process! My own PID: 3450
```

**Observation:** Parent and child execute independently, so the order of their output is not fixed. The two lines may appear in either order or be interleaved. The child PID shown by `getpid()` should match the positive PID returned by `fork()` to the parent.

> **Note:** The sample output is illustrative. Actual PIDs and output order vary. A robust program should also check whether `fork()` returned `-1`, which indicates failure.

## 4. Step 3: Synchronizing with `wait()` to Display the Child's PID

The `wait()` function, declared in `<sys/wait.h>`, allows the parent to wait for a child process to terminate.

Calling `wait(NULL)`:

1. Blocks the parent until one of its child processes terminates (if a waitable child exists).
2. Returns the PID of the child whose termination status was collected.

### Main Lab Source Code: `process_creation_wait.c`

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>

int main() {
    pid_t p = fork();

    if (p == 0) {
        // --- CHILD BRANCH ---
        printf("[Child] Work started. My PID: %d\n", getpid());
        sleep(2); // Pause so students can clearly observe the parent waiting
        printf("[Child] Work finished! Exiting now.\n");
        exit(0);
    } else {
        // --- PARENT BRANCH ---
        printf("[Parent] Child started. Now waiting for it to finish...\n");

        // wait(NULL) blocks the parent and returns the terminated child's PID
        pid_t terminated_pid = wait(NULL);
        printf("[Parent] Child with PID %d has terminated!\n", terminated_pid);
    }

    return 0;
}
```

### Compilation and Execution

```bash
gcc process_creation_wait.c -o exp9
./exp9
```

### Sample Output

```text
[Parent] Child started. Now waiting for it to finish...
[Child] Work started. My PID: 3450
[Child] Work finished! Exiting now.
[Parent] Child with PID 3450 has terminated!
```

**Observation:** The child sleeps for two seconds while the parent is blocked in `wait(NULL)`. Once the child exits, the parent resumes and prints the PID returned by `wait()`.

## 5. Line-by-Line Code Breakdown

| Code statement | Operating system explanation |
|---|---|
| `pid_t p = fork();` | Creates a child process. Returns `0` to the child and the child's PID to the parent; returns `-1` on failure. |
| `if (p == 0)` | Evaluates true in the child process. |
| `getpid()` | Retrieves the PID of the process that calls it. |
| `sleep(2);` | Pauses the child for two seconds so the parent waiting can be observed. |
| `exit(0);` | Terminates the child with a successful exit status. |
| `else { ... }` | In the successful-fork case, this branch runs in the parent, where `p` contains the child's PID. |
| `pid_t terminated_pid = wait(NULL);` | Blocks until a child terminates, then returns the PID of the child whose status was collected. |

## 6. Quick Summary for the Whiteboard

- **`getpid()`**: Retrieves the current process's own PID.
- **`fork()`**: Creates a child process. It returns `0` in the child and the child's PID in the parent.
- **`wait(NULL)`**: Makes the parent wait for a child to terminate and returns the terminated child's PID.

## 7. Observation Table

The following values are examples from an illustrative run. Actual PIDs differ on each execution.

| Execution step | Observed process PID | Return value recorded |
|---|---|---|
| Step 1 (`getpid`) | Single process: `3412` | `getpid() = 3412` |
| Step 2 (`fork`) | Parent: `3449`; Child: `3450` | `fork() = 0` in child; `3450` in parent |
| Step 3 (`wait`) | Parent: `3449`; Child: `3450` | `wait(NULL) = 3450` |

## 8. Result

The three C programs demonstrate process ID inspection using `getpid()`, process creation using `fork()`, and parent-child synchronization using `wait(NULL)`. The experiment illustrates how Linux identifies processes, creates a child process, and coordinates orderly process termination.

## 9. Files in This Experiment

| Filename | Purpose |
|---|---|
| `step1_pid.c` | Displays the PID of the current process. |
| `step2_fork.c` | Creates a child process and displays the different `fork()` return values. |
| `process_creation_wait.c` | Waits for the child to finish and displays its terminated PID. |

## 10. Running the Experiment

Compile and run each program from the directory containing its source file:

```bash
gcc step1_pid.c -o step1_pid
./step1_pid

gcc step2_fork.c -o step2_fork
./step2_fork

gcc process_creation_wait.c -o exp9
./exp9
```

To remove the generated executables:

```bash
rm -f step1_pid step2_fork exp9
```
