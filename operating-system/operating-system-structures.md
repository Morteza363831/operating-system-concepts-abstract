# Season 2: System Calls

## 1. Understanding System Calls
System calls are the essential mechanism through which user programs interact with the operating system (OS). They act as a controlled gateway, allowing applications to request services—like accessing hardware, managing files, or creating processes—that require kernel-level privileges. Without system calls, programs would lack a safe, standardized way to perform these tasks.

- **Definition**: A system call is a programmatic request to the OS kernel to execute a privileged operation on behalf of a user application.
- **Why They Matter**:
  - **Security**: Direct hardware access by applications could crash the system or breach security; system calls enforce controlled access.
  - **Abstraction**: Programs don’t need to understand hardware details—system calls provide a uniform interface (e.g., reading from a disk or network uses the same `read()` call).
- **Real-World Example**:
  - On Ubuntu, when you type `cat file.txt` in the terminal, `cat` uses the `open()` system call to access the file, `read()` to retrieve its contents, and `write()` to display them on the screen.
- **Learning Point**: System calls are invoked via a mode switch from **user mode** (unprivileged) to **kernel mode** (privileged), ensuring the OS retains control.
- **Ubuntu Tool**: Run `strace cat file.txt` to see the exact system calls executed, revealing the behind-the-scenes interaction with the kernel.

---

## 2. Categories of System Calls
System calls are grouped by the type of service they provide. Mastering these categories gives you a full picture of how an OS like Ubuntu manages resources and operations.

### 2.1 Process Control
These calls manage the lifecycle of processes—the executing instances of programs.

- **Key System Calls**:
  - **`fork()`**: Creates a duplicate of the calling process, resulting in a parent and child process.
    - *Example*: Launching a new terminal tab in Ubuntu uses `fork()` to spawn a new shell process.
  - **`exec()`**: Replaces the current process image with a new program.
    - *Example*: After `fork()`, the child might call `exec("/bin/bash")` to run a shell.
  - **`exit()`**: Terminates a process.
    - *Example*: Closing a terminal calls `exit()` to end the shell process.
  - **`wait()`**: Pauses the parent until a child process finishes.
    - *Example*: A script waiting for a background task to complete.
- **Learning Points**:
  - The parent and child processes after `fork()` share the same code but have separate memory spaces.
  - **Orphan processes** occur if a parent exits before its child, leaving the child adopted by `init` (PID 1).
  - **Zombie processes** linger if a parent doesn’t call `wait()` to clean up a terminated child.
- **Ubuntu Example**: Run `ps aux` to see active processes, then use `strace -f -e fork,execve bash` to trace process creation.

### 2.2 File Management
These calls handle operations on files, such as creating, reading, or deleting them.

- **Key System Calls**:
  - **`open(const char *pathname, int flags, mode_t mode)`**: Opens a file, returning a file descriptor.
    - *Example*: `int fd = open("notes.txt", O_RDWR | O_CREAT, 0644);` opens a file for reading and writing, creating it if it doesn’t exist.
  - **`read(int fd, void *buf, size_t count)`**: Reads data from a file descriptor into a buffer.
    - *Example*: `read(fd, buffer, 50);` reads 50 bytes.
  - **`write(int fd, const void *buf, size_t count)`**: Writes data to a file descriptor.
    - *Example*: `write(fd, "Hello!", 6);` writes "Hello!" to the file.
  - **`close(int fd)`**: Closes a file descriptor, freeing resources.
    - *Example*: `close(fd);`
- **Learning Points**:
  - **File descriptors** are small integers (e.g., 0 for stdin, 1 for stdout, 2 for stderr) that index open files in a process’s file table.
  - System calls return `-1` on error, with `errno` set to indicate the issue (e.g., `ENOENT` for "file not found").
- **Ubuntu Example**: Create a file with `touch test.txt`, then use `strace -e open,read,write cat test.txt` to observe file operations.

### 2.3 Device Management
These calls manage hardware devices, often treating them as files (a Unix philosophy).

- **Key System Calls**:
  - **`ioctl(int fd, unsigned long request, ...)`**: Performs device-specific operations.
    - *Example*: `ioctl(fd, TIOCSWINSZ, &winsize);` sets terminal window size.
  - **`read()` and `write()`**: Used for devices too.
    - *Example*: `read(fd, buffer, 10);` from `/dev/random` generates random bytes.
- **Learning Points**:
  - Devices appear as files in `/dev` (e.g., `/dev/sda` for a disk, `/dev/tty` for the terminal).
  - The "everything is a file" abstraction simplifies programming but requires device-specific handling via `ioctl()`.
- **Ubuntu Example**: Run `cat /dev/random | head -c 10` and trace it with `strace` to see device interaction.

### 2.4 Information Maintenance
These calls retrieve or set system or process information.

- **Key System Calls**:
  - **`getpid()`**: Returns the calling process’s ID.
    - *Example*: `pid_t pid = getpid();` might return 1234.
  - **`time(time_t *tloc)`**: Gets the current time in seconds since the Unix epoch.
    - *Example*: `time_t now = time(NULL);`
  - **`sysinfo(struct sysinfo *info)`**: Provides system stats (e.g., free memory).
- **Learning Points**:
  - Useful for debugging, logging, or time-based logic.
  - Combine with `getenv()` to access environment variables like `PATH`.
- **Ubuntu Example**: Write a script using `time()` to log timestamps, and check PIDs with `echo $$` in a terminal.

### 2.5 Communication
These calls enable interprocess communication (IPC) or networking.

- **Key System Calls**:
  - **`pipe(int pipefd[2])`**: Creates a pipe for one-way communication.
    - *Example*: `pipe(pipefd);`—write to `pipefd[1]`, read from `pipefd[0]`.
  - **`socket(int domain, int type, int protocol)`**: Sets up network communication.
    - *Example*: `socket(AF_INET, SOCK_STREAM, 0);` for a TCP socket.
  - **`shmget(key_t key, size_t size, int shmflg)`**: Allocates shared memory.
- **Learning Points**:
  - Pipes are ideal for parent-child communication post-`fork()`.
  - Sockets power networked applications like web servers.
- **Ubuntu Example**: Use `nc -l 12345` (netcat) and trace it to see `socket()` in action.

### 2.6 Protection
These calls manage access control and security.

- **Key System Calls**:
  - **`chmod(const char *pathname, mode_t mode)`**: Changes file permissions.
    - *Example*: `chmod("script.sh", 0755);` makes it executable.
  - **`chown(const char *pathname, uid_t owner, gid_t group)`**: Changes ownership.
    - *Example*: `chown("file.txt", 1000, 1000);`
  - **`umask(mode_t mask)`**: Sets default permissions for new files.
- **Learning Points**:
  - Permissions use octal notation (e.g., 0644 = read/write for owner, read for others).
  - **Setuid** bits allow programs to run with elevated privileges.
- **Ubuntu Example**: Run `chmod +x script.sh` and verify with `ls -l`, then trace it with `strace`.

---

## 3. How System Calls Work
System calls involve a switch from user mode to kernel mode, orchestrated by the OS.

- **Process**:
  1. A program invokes a system call (e.g., via a library like `libc`).
  2. A special CPU instruction (e.g., `syscall` on x86-64) triggers an interrupt.
  3. The kernel handles the request and returns a result.
- **Technical Details**:
  - System calls have unique numbers (e.g., `open()` is 2 on Linux x86-64—see `/usr/include/asm/unistd_64.h`).
  - Modern CPUs use fast instructions like `syscall`/`sysret` to reduce overhead.
- **Learning Point**: This mode switch incurs a small performance cost, so efficient programs minimize system calls.
- **Ubuntu Example**: Use `strace -T ls` to see the time spent on each call, highlighting the overhead.

---

## 4. Detailed Examples on Ubuntu
Let’s explore practical examples for each category.

### 4.1 Process Control
```c
#include <stdio.h>
#include <unistd.h>
int main() {
    pid_t pid = fork();
    if (pid == 0) printf("Child PID: %d\n", getpid());
    else if (pid > 0) printf("Parent PID: %d\n", getpid());
    else perror("fork failed");
    return 0;
}
```
- **Output**: Shows parent and child PIDs.
- **Trace**: `strace -e fork ./a.out`

### 4.2 File Management
```c
#include <fcntl.h>
#include <unistd.h>
int main() {
    int fd = open("test.txt", O_WRONLY | O_CREAT, 0644);
    write(fd, "Ubuntu rocks!", 13);
    close(fd);
    return 0;
}
```
- **Output**: Creates "test.txt" with the text.
- **Trace**: `strace -e open,write,close ./a.out`

### 4.3 Device Management
```c
#include <fcntl.h>
#include <unistd.h>
int main() {
    int fd = open("/dev/random", O_RDONLY);
    char buf[5];
    read(fd, buf, 5);
    close(fd);
    return 0;
}
```
- **Output**: Reads 5 random bytes.
- **Trace**: `strace -e read ./a.out`

### 4.4 Information Maintenance
```c
#include <stdio.h>
#include <unistd.h>
#include <time.h>
int main() {
    printf("PID: %d, Time: %ld\n", getpid(), time(NULL));
    return 0;
}
```
- **Output**: Displays PID and epoch time.
- **Trace**: `strace -e getpid,time ./a.out`

### 4.5 Communication
```c
#include <unistd.h>
int main() {
    int pipefd[2];
    pipe(pipefd);
    write(pipefd[1], "Hi", 2);
    char buf[2];
    read(pipefd[0], buf, 2);
    write(1, buf, 2); // Print to stdout
    return 0;
}
```
- **Output**: Prints "Hi".
- **Trace**: `strace -e pipe,read,write ./a.out`

### 4.6 Protection
```c
#include <sys/stat.h>
int main() {
    chmod("test.txt", 0600); // Owner read/write only
    return 0;
}
```
- **Output**: Changes permissions (verify with `ls -l`).
- **Trace**: `strace -e chmod ./a.out`

---

## 5. Practice Exercises
1. **Define a system call with an example.**
   - A system call requests a kernel service. Example: `write(1, "Hi", 2);` prints "Hi".
2. **List three system calls per category.**
   - Process: `fork()`, `exec()`, `wait()`.
   - File: `open()`, `read()`, `close()`.
   - Communication: `pipe()`, `socket()`, `shmget()`.
3. **Write a program to append text to a file.**
   - Use `open()` with `O_APPEND`, `write()`, and `close()`.
4. **Explain `fork()` vs. `exec()`.**
   - `fork()` duplicates a process; `exec()` loads a new program into it.
5. **Trace a command.**
   - Run `strace -o trace.txt ls` and analyze the output.

---

## 6. Extra Learning Tips
- **Experiment**: Write and trace C programs for each system call.
- **Read**: Use `man 2 <call>` (e.g., `man 2 fork`) for details.
- **Visualize**: Diagram the user-to-kernel transition for a call like `read()`.

This abstraction equips you with a deep, practical understanding of Season 2, tailored for Ubuntu. Let me know if you’d like more examples or clarification!
