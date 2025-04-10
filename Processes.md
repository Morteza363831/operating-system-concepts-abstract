# Season 3: Processes

## 1. What is a Process?
A **process** is the cornerstone of any operating system, defined as a program in execution. It’s not just the static code (called the *text section*), but an active entity that includes the current state of execution—tracked by the program counter, CPU registers, stack, and data. Each process operates in its own isolated memory space, managed by the operating system, which ensures security and prevents interference between processes.

- **Formal Definition**: A process encompasses the program code, its current execution state (program counter, registers), and allocated resources (memory, files, I/O devices).
- **Key Roles**:
  - **Multitasking**: Enables running multiple applications simultaneously (e.g., editing a document in LibreOffice while streaming music on Ubuntu).
  - **Resource Management**: The OS allocates CPU time, memory, and I/O to processes efficiently.
  - **Isolation**: Keeps processes separate, protecting system stability.
- **Practical Example**:
  - On Ubuntu, launch **Firefox**. This creates a process. Check it with `ps aux | grep firefox` to see its details (PID, memory usage, etc.).
- **Learning Point**: The OS kernel oversees processes, managing their creation, scheduling, and termination.
- **Ubuntu Tool**: Use `top` or `htop` to observe running processes, their CPU/memory usage, and states in real-time.

---

## 2. Process States and Lifecycle
A process doesn’t stay static—it transitions through distinct **states** based on its current activity, managed by the operating system.

- **Core States**:
  - **New**: The process is being initialized.
  - **Ready**: The process is prepared to run and awaiting CPU allocation.
  - **Running**: The process is actively executing on the CPU.
  - **Waiting**: The process is paused, waiting for an event (e.g., I/O completion or a signal).
  - **Terminated**: The process has completed execution and is being cleaned up.
- **State Transitions**:
  - **New → Ready**: Process creation completes, and it’s queued for execution.
  - **Ready → Running**: The scheduler assigns it to the CPU.
  - **Running → Waiting**: The process requests I/O or another blocking operation.
  - **Waiting → Ready**: The event completes, and the process is ready again.
  - **Running → Terminated**: The process finishes or is killed.
- **Learning Points**:
  - The OS tracks all processes in a **process table**, recording their states and metadata.
  - **Context Switching**: When the CPU switches processes, it saves the current process’s state and loads the next one’s—crucial for multitasking.
- **Ubuntu Example**:
  - Run `ps -e -o pid,state,comm` to see PIDs, states (`R` for running, `S` for sleeping/waiting), and command names.
- **Visual Example**: Imagine a process like `gedit` (text editor):
  - **New**: `gedit` is launched.
  - **Ready**: It’s queued.
  - **Running**: You’re typing.
  - **Waiting**: It’s saving a file to disk.
  - **Terminated**: You close it.

---

## 3. Process Control Block (PCB)
Every process is represented by a **Process Control Block (PCB)**, a kernel data structure that acts as the process’s “identity card.” It stores all critical information needed to manage the process.

- **Key Components**:
  - **Process ID (PID)**: A unique number identifying the process.
  - **Process State**: Current state (e.g., running, waiting).
  - **Program Counter**: Points to the next instruction to execute.
  - **CPU Registers**: Saved values (e.g., stack pointer, general-purpose registers) for context switching.
  - **Memory Info**: Details like page tables or memory limits.
  - **I/O Status**: Open files, pending I/O operations.
  - **Scheduling Data**: Priority level, pointers to scheduling queues.
- **Why It Matters**:
  - The PCB enables the OS to pause a process, save its state, and resume it later—vital for multitasking.
- **Ubuntu Example**:
  - Run `cat /proc/$$/status` (where `$$` is your shell’s PID) to view details like state, memory usage, and more.
- **Learning Point**:
  - On Ubuntu, the `/proc` filesystem provides a window into PCBs (e.g., `/proc/1234/` for PID 1234).
- **Practical Insight**: Killing a process (`kill -9 <PID>`) clears its PCB, freeing resources.

---

## 4. Process Scheduling
The OS uses a **scheduler** to decide which process runs on the CPU, balancing efficiency, responsiveness, and fairness.

- **Scheduling Queues**:
  - **Job Queue**: All processes in the system.
  - **Ready Queue**: Processes ready to execute.
  - **Device Queues**: Processes waiting for specific I/O devices.
- **Common Algorithms**:
  - **First-Come, First-Served (FCFS)**: Runs processes in arrival order—simple but can lead to delays.
  - **Shortest Job First (SJF)**: Prioritizes the process with the shortest next CPU burst—optimal but hard to predict.
  - **Round-Robin (RR)**: Gives each process a time slice (quantum), ideal for time-sharing systems like Ubuntu.
- **Types**:
  - **Non-Preemptive**: A process runs until it yields the CPU.
  - **Preemptive**: The OS can interrupt a process to run another—used in modern OSes like Ubuntu.
- **Learning Points**:
  - Ubuntu’s Linux kernel uses the **Completely Fair Scheduler (CFS)**, a sophisticated RR variant.
  - Priority affects scheduling—higher-priority processes run sooner.
- **Ubuntu Example**:
  - Adjust priority with `nice` (e.g., `nice -n 10 firefox` runs Firefox with lower priority) or `renice` for running processes.

---

## 5. Process Operations
The OS provides mechanisms (system calls) to create, manage, and terminate processes.

### 5.1 Process Creation
- **System Call**: `fork()`
  - Creates a child process as a copy of the parent, with a new PID and PCB.
  - Both parent and child continue execution from the `fork()` call.
- **Post-Fork**: The child often uses `exec()` (e.g., `execlp()`) to load a new program.
- **Example**:
  - In Ubuntu, type `bash` in a terminal—it forks a new shell process.
- **Code Example**:
  ```c
  #include <unistd.h>
  #include <stdio.h>
  int main() {
      pid_t pid = fork();
      if (pid == 0) printf("Child PID: %d\n", getpid());
      else printf("Parent PID: %d, Child PID: %d\n", getpid(), pid);
      return 0;
  }
  ```
  - **Output**: Shows both parent and child PIDs.
- **Ubuntu Tool**: Use `pstree -p` to see the process hierarchy.

### 5.2 Process Termination
- **System Call**: `exit()`
  - Ends a process, freeing its resources (memory, files).
- **Parent Role**: Uses `wait()` or `waitpid()` to collect the child’s exit status.
- **Learning Point**:
  - If a parent exits first, its children become **orphans**, adopted by `init` (PID 1).
- **Ubuntu Example**:
  - Terminate a process with `kill 1234` (sends `SIGTERM`) or `kill -9 1234` (forceful `SIGKILL`).

### 5.3 Process Hierarchy
- Processes form a tree, starting with `init` or `systemd` (PID 1 on Ubuntu).
- **Example**: `systemd` spawns `sshd`, which forks a shell for your login.
- **Learning Point**: Killing a parent may cascade to children unless they’re detached.

---

## 6. Interprocess Communication (IPC)
Processes often need to exchange data or coordinate. **IPC** mechanisms enable this while preserving isolation.

### 6.1 Shared Memory
- **Concept**: Processes share a memory region for fast data exchange.
- **System Calls**: `shmget()` (allocate), `shmat()` (attach), `shmdt()` (detach).
- **Pros/Cons**: Fast, but requires synchronization (e.g., semaphores) to avoid conflicts.
- **Ubuntu Example**:
  - Run `ipcs -m` to list shared memory segments.
- **Code Example**:
  ```c
  #include <sys/shm.h>
  #include <stdio.h>
  int main() {
      int shmid = shmget(IPC_PRIVATE, 1024, 0666 | IPC_CREAT);
      char *data = shmat(shmid, NULL, 0);
      sprintf(data, "Hello from shared memory!");
      printf("Wrote: %s\n", data);
      shmdt(data);
      return 0;
  }
  ```

### 6.2 Message Passing
- **Concept**: Processes send/receive messages via kernel-managed channels.
- **Mechanisms**:
  - **Pipes**: Unidirectional (e.g., `ls | wc` in a shell).
  - **Message Queues**: More flexible, using `msgget()`, `msgsnd()`, `msgrcv()`.
- **Pros/Cons**: Slower than shared memory but inherently synchronized.
- **Ubuntu Example**:
  - Create a named pipe: `mkfifo mypipe`, then `echo "test" > mypipe` and `cat mypipe`.
- **Code Example**:
  ```c
  #include <unistd.h>
  #include <stdio.h>
  int main() {
      int fd[2];
      pipe(fd);
      if (fork() == 0) {
          close(fd[0]);
          write(fd[1], "Hi!", 4);
      } else {
          close(fd[1]);
          char buf[4];
          read(fd[0], buf, 4);
          printf("Received: %s\n", buf);
      }
      return 0;
  }
  ```

### 6.3 Sockets
- **Concept**: Communication over networks or locally (e.g., TCP/IP, UNIX domain sockets).
- **Use Case**: Client-server apps (e.g., a web browser and server).
- **Ubuntu Example**: Run `netstat -tuln` to see listening sockets.

---

## 7. Practical Ubuntu Examples
- **Process Details**: Find a process with `pgrep firefox`, then inspect `/proc/<PID>/status`.
- **Scheduling**: Run a script with `chrt -r -p 10 <PID>` for real-time priority.
- **IPC**: Use `ipcs` to monitor shared memory/pipes, `ipcrm` to remove them.

---

## 8. Exercises to Master Season 3
1. **Define the PCB and its role.**
   - Answer: A kernel structure storing PID, state, registers, etc., used for process management and context switching.
2. **List process states with examples.**
   - New (process starting), Ready (queued), Running (executing), Waiting (I/O), Terminated (done).
3. **Compare shared memory and pipes.**
   - Shared memory: fast, needs synchronization; pipes: slower, built-in order.
4. **Write a program using `fork()` and a pipe.**
   - See the pipe example above.
5. **Trace a process’s lifecycle on Ubuntu.**
   - Launch `gedit`, use `ps` and `/proc`, then `kill` it—observe state changes.

---

## 9. Additional Learning Tips
- **Hands-On**: Write C programs with `fork()`, `exec()`, and IPC.
- **Visualize**: Sketch state transitions or process trees.
- **Explore**: Read `man 2 fork`, `man 2 pipe`, etc., on Ubuntu for detailed docs.

This abstraction equips you with a deep, practical understanding of Season 3: Processes, enriched with examples and Ubuntu-specific insights. Let me know if you’d like more depth or examples!
