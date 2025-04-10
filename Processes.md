# Season 3: Processes

## 1. What is a Process?
A **process** is a program in execution—an active entity that includes not just the code but also its current state, memory, and resources. It’s the fundamental unit of work in an operating system (OS), enabling multitasking, resource allocation, and isolation.

- **Formal Definition**: A process comprises:
  - **Text Section**: The executable code (e.g., compiled binary).
  - **Data Section**: Global and static variables.
  - **Heap**: Dynamically allocated memory during runtime.
  - **Stack**: Temporary data like function call frames and local variables.
  - **Program Counter (PC)**: Points to the next instruction.
  - **CPU Registers**: Hold current computation values.
- **Process vs. Thread**: A process is a standalone entity with its own memory space, while threads are lightweight sub-processes sharing the same memory within a process. Example: Firefox runs as a process with multiple threads for tabs.
- **Examples**:
  - **Ubuntu**: Launch **gedit** (text editor). Use `ps aux | grep gedit` to see its PID, memory usage, and status.
  - **Python**: `os.fork()` duplicates the current process:
    ```python
    import os
    pid = os.fork()
    if pid == 0:
        print(f"Child process: {os.getpid()}")
    else:
        print(f"Parent process: {os.getpid()}, Child: {pid}")
    ```
  - **C**: Using `fork()` to create a process:
    ```c
    #include <unistd.h>
    #include <stdio.h>
    int main() {
        if (fork() == 0) printf("Child PID: %d\n", getpid());
        else printf("Parent PID: %d\n", getpid());
        return 0;
    }
    ```
- **Memory Layout Insight**: Use `pmap <PID>` on Ubuntu (e.g., `pmap $(pgrep gedit)`) to inspect memory regions (text, stack, heap).
- **Learning Point**: The OS kernel manages process isolation via virtual memory, ensuring each process operates independently.

---

## 2. Process States and Lifecycle
A process transitions through various **states**, reflecting its current activity and managed by the OS.

- **Detailed States**:
  - **New**: Being created (e.g., running `./myprogram`).
  - **Ready**: Waiting for CPU time (e.g., multiple apps open, queued).
  - **Running**: Executing on the CPU (e.g., typing in **gedit**).
  - **Waiting (Blocked)**: Awaiting an event like I/O (e.g., **wget** downloading a file).
  - **Terminated**: Finished or killed (e.g., closing **Firefox**).
- **State Transitions** (with Examples):
  - **New → Ready**: `gcc -o myprog myprog.c` completes compilation, and `./myprog` is queued.
  - **Ready → Running**: The scheduler assigns CPU to `./myprog`.
  - **Running → Waiting**: `./myprog` reads a file with `fread()`.
  - **Waiting → Ready**: File read completes, `./myprog` is queued again.
  - **Running → Terminated**: `./myprog` calls `exit(0)` or is killed with `Ctrl+C`.
- **Context Switching**: The OS saves the current process’s state (via PCB) and loads another’s, enabling multitasking. Example: Switching between **Firefox** and **VLC**.
- **Real-World Analogy**: Processes are like chefs in a kitchen:
  - New: Chef hired.
  - Ready: Chef waiting for an oven.
  - Running: Chef cooking.
  - Waiting: Chef waiting for dough to rise.
  - Terminated: Chef finishes shift.
- **Ubuntu Tools**:
  - `ps -e -o pid,stat,comm` shows states (`R`=running, `S`=sleeping/waiting, `Z`=zombie).
  - `top` or `htop` provides a live view of process states.

---

## 3. Process Control Block (PCB)
The **PCB** is the kernel’s data structure for each process, storing all information needed to manage and resume it.

- **Comprehensive Fields**:
  - **PID**: Unique identifier (e.g., 1234).
  - **State**: Current state (e.g., running).
  - **Program Counter**: Next instruction address.
  - **Registers**: CPU state (e.g., `%rax`, `%rsp` on x86_64).
  - **Memory Management**: Page tables, memory limits.
  - **File Descriptors**: Open files (e.g., `/dev/tty`).
  - **I/O Status**: Pending operations.
  - **Priority**: Scheduling priority (e.g., nice value).
  - **Parent PID (PPID)**: Parent process ID.
  - **Signals**: Pending signals (e.g., `SIGTERM`).
- **Role**: Enables context switching by saving/restoring process state.
- **Ubuntu Examples**:
  - `cat /proc/1234/status` shows PCB details for PID 1234.
  - `ls /proc/$$/fd` lists open file descriptors for your shell.
- **Windows Insight**: Uses **EPROCESS** structure, accessible via tools like Process Explorer.
- **Practical Use**: `lsof -p <PID>` lists a process’s open files, reflecting PCB data.

---

## 4. Process Scheduling
The **scheduler** allocates CPU time to processes, optimizing throughput, fairness, and responsiveness.

- **Scheduling Queues**:
  - **Job Queue**: All system processes.
  - **Ready Queue**: Processes awaiting CPU.
  - **Device Queues**: Processes waiting for I/O (e.g., disk).
- **Algorithms** (with Examples):
  - **First-Come, First-Served (FCFS)**:
    - Order: P1 (5ms), P2 (2ms), P3 (3ms) → P1, P2, P3.
    - Issue: Long waits (convoy effect).
  - **Shortest Job First (SJF)**:
    - Order: P2 (2ms), P3 (3ms), P1 (5ms) → Optimal but needs burst time prediction.
  - **Round-Robin (RR)**:
    - Quantum=2ms: P1 (2ms), P2 (2ms), P3 (2ms), P1 (3ms) → Fair, used in Ubuntu.
  - **Priority Scheduling**:
    - P1 (priority 5), P2 (10), P3 (2) → P2, P1, P3. Risk: Starvation of low-priority tasks.
  - **Multilevel Queue**:
    - System processes (high priority) vs. user processes (low priority).
- **Real-Time Scheduling**:
  - Types: Hard (e.g., avionics) vs. Soft (e.g., video playback).
  - Algorithms: Rate Monotonic, Earliest Deadline First.
- **Ubuntu’s CFS**: The Completely Fair Scheduler uses a red-black tree for fairness.
- **Examples**:
  - `nice -n 10 python script.py` runs with lower priority.
  - `chrt -r -p 20 <PID>` sets real-time scheduling.

---

## 5. Process Operations
The OS provides system calls to create, manage, and terminate processes.

### 5.1 Process Creation
- **System Call**: `fork()`
  - Duplicates the parent process, returning 0 to the child and the child’s PID to the parent.
- **Follow-Up**: `exec()` variants (e.g., `execlp()`) replace the child’s program.
- **Code Examples**:
  - **C with Fork and Exec**:
    ```c
    #include <unistd.h>
    #include <stdio.h>
    int main() {
        pid_t pid = fork();
        if (pid == 0) {
            execlp("ls", "ls", "-l", NULL);
            perror("exec failed");
        } else {
            printf("Parent waiting...\n");
            wait(NULL);
        }
        return 0;
    }
    ```
  - **Python**:
    ```python
    import os
    pid = os.fork()
    if pid == 0:
        os.execlp("ls", "ls", "-l")
    else:
        os.wait()
        print("Parent done")
    ```
- **Ubuntu Tool**: `strace -e fork,execve ./a.out` traces creation.

### 5.2 Process Termination
- **System Call**: `exit(status)`
  - Returns `status` to the parent, frees resources.
- **Types**:
  - **Normal**: `exit(0)` (success).
  - **Error**: `exit(1)` (failure).
  - **Signal**: `SIGKILL` (e.g., `kill -9 <PID>`).
- **Special Cases**:
  - **Zombies**: Terminated but unclaimed by parent (`ps -e | grep Z`).
  - **Orphans**: Adopted by `init` (PID 1) if parent exits.
- **Example**: `kill -SIGTERM $(pgrep gedit)` terminates **gedit**.

### 5.3 Process Hierarchy
- **Tree Structure**: Starts with `init` or `systemd` (PID 1).
- **Example**: `systemd` → `sshd` → `bash` → `python script.py`.
- **Ubuntu Tool**: `pstree -p` visualizes the hierarchy.

---

## 6. Interprocess Communication (IPC)
IPC enables processes to share data or synchronize actions.

### 6.1 Shared Memory
- **Concept**: Fastest IPC; processes access a common memory region.
- **System Calls**: `shmget()`, `shmat()`, `shmdt()`, `shmctl()`.
- **Code Example**:
  ```c
  #include <sys/shm.h>
  #include <stdio.h>
  int main() {
      int shmid = shmget(IPC_PRIVATE, 1024, 0666);
      char *shm = shmat(shmid, NULL, 0);
      if (fork() == 0) {
          sprintf(shm, "Child wrote this");
          shmdt(shm);
      } else {
          wait(NULL);
          printf("Parent reads: %s\n", shm);
          shmdt(shm);
          shmctl(shmid, IPC_RMID, NULL);
      }
      return 0;
  }
  ```
- **Ubuntu Tool**: `ipcs -m` lists segments; `ipcrm -m <shmid>` removes them.

### 6.2 Message Passing
- **Mechanisms**:
  - **Pipes**: Unidirectional (e.g., `ls | wc -l`).
  - **Named Pipes (FIFOs)**: `mkfifo myfifo; echo "data" > myfifo & cat myfifo`.
  - **Message Queues**: Structured messages via `msgget()`, `msgsnd()`, `msgrcv()`.
- **Code Example (Pipe)**:
  ```c
  #include <unistd.h>
  #include <stdio.h>
  int main() {
      int fd[2];
      pipe(fd);
      if (fork() == 0) {
          close(fd[0]);
          write(fd[1], "Hello", 6);
          close(fd[1]);
      } else {
          close(fd[1]);
          char buf[6];
          read(fd[0], buf, 6);
          printf("Received: %s\n", buf);
          close(fd[0]);
      }
      return 0;
  }
  ```

### 6.3 Sockets
- **Concept**: Network or local communication (e.g., TCP, UDP).
- **Example**: `nc -l 1234` (server), `telnet localhost 1234` (client).

### 6.4 Signals
- **Concept**: Asynchronous notifications (e.g., `SIGINT` via `Ctrl+C`).
- **Code Example**:
  ```c
  #include <signal.h>
  #include <stdio.h>
  void handler(int sig) { printf("Caught signal %d\n", sig); }
  int main() {
      signal(SIGINT, handler);
      while (1) sleep(1);
      return 0;
  }
  ```
- **Ubuntu**: `kill -SIGUSR1 <PID>` sends a custom signal.

### 6.5 Semaphores
- **Concept**: Synchronization counters.
- **Example**: `semget()`, `semop()` to manage access to shared resources.

---

## 7. Process Synchronization
Processes accessing shared resources need **synchronization** to avoid **race conditions**.

- **Critical Section**: Code accessing shared data.
- **Primitives**:
  - **Mutexes**: Binary locks (e.g., `pthread_mutex_lock()`).
  - **Semaphores**: General counters.
- **Example**: Two processes incrementing a shared counter without a mutex could yield incorrect results.

---

## 8. Deadlocks
A **deadlock** is when processes wait indefinitely for each other’s resources.

- **Conditions**:
  - **Mutual Exclusion**: Resources held exclusively.
  - **Hold and Wait**: Holding one resource, waiting for another.
  - **No Preemption**: Resources can’t be taken away.
  - **Circular Wait**: Cyclic dependency.
- **Example**: P1 holds R1, wants R2; P2 holds R2, wants R1.
- **Prevention**: Break one condition (e.g., resource ordering).
- **Ubuntu Tool**: `ps` and `lsof` can help detect resource contention.

---

## 9. Process Management in Modern OSes
- **Ubuntu/Linux**: Uses CFS, managed by `systemd`.
- **Windows**: Priority-based scheduler, Task Manager.
- **macOS**: BSD-based, uses **XNU** kernel and Grand Central Dispatch.

---

## 10. Advanced Topics
- **Threads**: Share memory within a process (e.g., `pthread_create()`).
- **Multiprocessing**: Multiple CPUs (e.g., `taskset` on Ubuntu).
- **Distributed Systems**: Processes across machines (e.g., MPI).

---

## 11. Ubuntu-Specific Tools
- **Monitoring**: `ps aux`, `top`, `htop`, `pstree -p`.
- **Tracing**: `strace -p <PID>` (system calls), `ltrace` (library calls).
- **Priority**: `nice -n 5 <cmd>`, `renice 5 -p <PID>`.
- **Affinity**: `taskset -c 0,1 <cmd>`.
- **Resources**: `ulimit -u` (max processes), `/proc/<PID>/`.

---

## 12. Exercises
1. **Write a C program with two child processes using shared memory and semaphores.**
2. **Simulate FCFS and RR scheduling with sample processes in Python.**
3. **Trace a process’s lifecycle with `strace` and `ps`.**
4. **Create a deadlock with pipes, then resolve it.**
5. **Explore `/proc` for five processes, noting their memory and state.**

---

This abstraction is now richer with examples, tools, and topics like synchronization and deadlocks, ensuring a complete mastery of Season 3: Processes. Let me know if you need more!
