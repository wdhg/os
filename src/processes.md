# Processes

A process is an instance of a program being executed (a running program). They allow for a single processor to run multiple programs "simulatneously". Each process runs on a virtual CPU, and the real CPU can use this to run any single process. Effectively, the single CPU has been divided into multiple virtual CPUs.

## Why have Processes?

They:

- provide an illusion of concurrency (more on this later).
- provide isolation between programs (each process has its own address space).
- simplify programming (programs don't have to worry about other programs).
- allow for better utilisation of resources (different processes require different resources at certain times).

## Time-Slicing for Concurrency

An OS can switch the current running process every few milliseconds such that every process can have CPU time. This is done using a CPU scheduler, and it can be used to provide pseudo concurrency.

## Types of Concurrency

- **Pseudo Concurrency**: a single processor switches between processes by interleaving their runtimes. Overtime this can give the illusion of concurrent execution.
- **Real Concurrency**: multiple processors or CPU cores are used to run multiple processes in parallel.

## Fairness

Fairness is the concept that the CPU scheduler switches fairly between processes. This means that every process should have equal CPU time, or more generally they should have appropriate CPU times determined by each processes priority and workload.

## Context Switches

A context switch is when a processor switches from executing one process to another.

An OS may switch either periodically or in reponse to certain events / interrupts (such as I/O completion). Context switches are not pre-determined because the events causing them are non-deterministic.

Since processes will need to be restarted safely when being switched to, all relevant information (e.g. register values) must be stored when they are switched away from. This is done by storing the data in a process descriptor or a process control block (PCB), which is kept inside a process table.

Context switches are expensive. The process state needs to be saved / restored and CPU caches (including translation lookaside buffer (TLB)) will be lost. It is important for an OS to avoid unnecessary context switches.

## Process Control Block (PCB)

Each process has it's own virtual machine: They have their own virtual CPU, address space (stack, heap, text, data, etc), open file descriptors, etc. Enough data must be stored such that all of this remains preserved after a context switch.

The information that gets stored on a context switch is:

- **Registers**: program counter (PC), page table register, stack pointer, etc.
- **Process management info**: process ID (PID), parent process, process group, priority, CPU used, etc.
- **File management info**: root directory, working directory, open file descriptors, etc.

## Process Creation

There are two types of processes:

- **Foreground**: processes that interact with users.
- **Background**: daemons (e.g. handling incoming mail, printing requests, etc).

Processes are created on:

- system initialisation.
- user request.
- system calls by a running process.

Processes terminated can be caused by:

- **Normal Completion**: when the process completes the execution of its body.
- **System Calls**: e.g. `exit()` in UNIX, `ExitProcess()` in Windows.
- **Abnormal Exit**: when the process runs into an error or unhandled exception.
- **Aborted**: when another process overrules its execution (e.g. killed from terminall).
- **Never**: some processes run in an endless loop and never terminate unless an error occurs (e.g. most web servers).

## Process Hierarchies

UNIX allows for processes to form hierarchies (e.g. parent, child, child's child, etc).

Windows has no notion of process hierarchy. Instead when a process is created its parent is given a handle token to control it. This handle can be passed around to other processes.

## UNIX

### Creating Processes

```c
int fork(void)
```

`fork` creates a new child process by making an exact copy of the parent process image. The child process inherits its parent's resources and will begin to execute concurrently with it.

`fork` returns twice, once in the parent, and once in the child:

- **Parent**: return value is the process ID of the child.
- **Child**: return value is `0`.
- **Error**: if fork is unable to create a child, `-1` is returned to the parent. This can happen if there isn't enough memory to create the child process in.

An example of some code using `fork`:

```c
#include <unistd.h>
#include <stdio.h>

int main() {
  if (fork() != 0) {
    printf("Parent process\n");
  } else {
    printf("Child process\n");
  }

  printf("Common code\n");
}
```

### Executing Processes

```c
int execve(
  const char *path,
  char *const argv[],
  char *const envp[]
)
```

`execve` executes a command. Instead of copying the parent process, a completely new process is made.

Its arguments are:

- `path`: the full pathname of the program to be executed.
- `argv`: the arguments passed to this program.
- `envp`: a list of environment variables.

`execve` has many useful wrappers (`execl`, `execle`, `execvp`, `execv`, etc). Read `man execve` for more.

### Waiting for Process Termination

```c
int waitpid(
  int pid,
  int* stat,
  int options
)
```

`waitpid` suspends execution of the calling process until the child process with the PID terminates normally or it receives a signal.

`waitpid` can be used to wait for multiple children:

- `pid = -1`: wait for any child.
- `pid = 0` wait for any child in the same process group as caller.
- `pid = -gid` wait for any child with process group `gid`.

Its return value can be:

- the `pid` of the terminated child process.
- `0` if `WNOHANG` is set in options. This option "is used to indicate that the call should not block if there are no processes that wish to report status" (from `man waitpid`).
- `-1` on error, and `errno` is set to indicate the error (see `man errno`).

### Process Termination

```c
void exit(int status)
```

`exit` terminates the calling process, returning the exit `status` to the parent process (i.e. through `int *stat` in `waitpid`).

```c
void kill(int pid, int sid)
```
`kill` sends signal `sig` to process with PID `pid`.

An example of `exit`:

```c
#include <unistd.h>
#include <sys/wait.h>
#include <stdio.h>
#include <stdlib.h>

int main() {
  int pid = fork();

  if (pid != 0) {
    // parent process
    int *stat;
    waitpid(pid, stat, 0);
    printf("Child exit status is: %d\n", WEXITSTATUS(*stat));
  } else {
    // child process
    exit(4);
  }
}
```

```
$ gcc a.c && ./a.out
Child exit status is: 4
```


## Windows

Windows use `CreateProcess`, which is the equivalent of the UNIX `fork` and `execve`. `CreateProcess` is an alias of `CreateProcessA`:

```c++
BOOL CreateProcessA(
  LPCSTR                lpApplicationName,
  LPSTR                 lpCommandLine,
  LPSECURITY_ATTRIBUTES lpProcessAttributes,
  LPSECURITY_ATTRIBUTES lpThreadAttributes,
  BOOL                  bInheritHandles,
  DWORD                 dwCreationFlags,
  LPVOID                lpEnvironment,
  LPCSTR                lpCurrentDirectory,
  LPSTARTUPINFOA        lpStartupInfo,
  LPPROCESS_INFORMATION lpProcessInformation
);
```

[Source](https://docs.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-createprocessa)


## Processes Communication

Processes can communicate via a variety of ways:

- Files.
- Signals (UNIX).
- Events, Exceptions (Windows).
- Pipes.
- Message Queues (UNIX).
- Mailslots (Windows).
- Sockets.
- Shared memory.
- Semaphores.

## UNIX Signals

Signals are a Inter-Process Communication (IPC) mechanism. They are used to notify a process when an event has occured. Signal delivery is similar to the delivery of hardware interrupts.

A proceess can only send signals to another process if it has the correct permissions to do so:

- The real or effective user ID of the receiving process must match that of the sending process, or the user must have the appropriate privileges.
- The kernel can send signals to any process.

Signals are generated:

- when an exeception occurs (e.g. division by zero = `SIGFPE`, segment violation = `SIGSEGV`).
- when the kernel wants to notify the process of an event that has occured (e.g. if the process writes to a closed pipe = `SIGPIPE`).
- when the user enters certain key combinations (e.g. CTRL+C = `SIGINT`).
- when using the `kill` system call.

All of the UNIX signals are as follows (from `man signal`):

1.   `SIGHUP`:     terminal line hangup
2.   `SIGINT`:     interrupt program
3.   `SIGQUIT`:    quit program
4.   `SIGILL`:     illegal instruction
5.   `SIGTRAP`:    trace trap
6.   `SIGABRT`:    abort program (formerly SIGIOT)
7.   `SIGEMT`:     emulate instruction executed
8.   `SIGFPE`:     floating-point exception
9.   `SIGKILL`:    kill program
10.  `SIGBUS`:     bus error
11.  `SIGSEGV`:    segmentation violation
12.  `SIGSYS`:     non-existent system call invoked
13.  `SIGPIPE`:    write on a pipe with no reader
14.  `SIGALRM`:    real-time timer expired
15.  `SIGTERM`:    software termination signal
16.  `SIGURG`:     urgent condition present on socket
17.  `SIGSTOP`:    stop (cannot be caught or ignored)
18.  `SIGTSTP`:    stop signal generated from keyboard
19.  `SIGCONT`:    continue after stop
20.  `SIGCHLD`:    child status has changed
21.  `SIGTTIN`:    background read attempted from control terminal
22.  `SIGTTOU`:    background write attempted to control terminal
23.  `SIGIO`:      I/O is possible on a descriptor (see fcntl(2))
24.  `SIGXCPU`:    cpu time limit exceeded (see setrlimit(2))
25.  `SIGXFSZ`:    file size limit exceeded (see setrlimit(2))
26.  `SIGVTALRM`:  virtual time alarm (see setitimer(2))
27.  `SIGPROF`:    profiling timer alarm (see setitimer(2))
28.  `SIGWINCH`:   Window size change
29.  `SIGINFO`:    status request from keyboard
30.  `SIGUSR1`:    User defined signal 1
31.  `SIGUSR2`:    User defined signal 2

The default action for most signals is to terminal the process, however the receiving process can choose to:

- ignore the signal.
- handle the signal by installing a signal handler.

The two exceptions are `SIGKILL` and `SIGSTOP`, which cannot be ignored or handled.

An example signal handler is:

```c
#include <signal.h>
#include <stdio.h>

void sigint_handler(int sig) {
  fprintf(stderr, " => SIGINT caught!\n");
}

int main(int argc, char *argv[]) {
  signal(SIGINT, sigint_handler); // installs sigint_handler
  while (1) {}
}
```

```
$ gcc a.c && ./a.out || pkill a.out
^C => SIGINT caught!
^Z
[1]  + 16252 suspended  ./a.out
[1]  + 16252 terminated  ./a.out
```


## Unix Pipes

A pipe connects the standard output of one process to the standard input of another process. This allows for a one-way communication channel between processes.

They are extremely useful in the command line / shell scripts:

```bash
ls | less
ps aux | grep my_prog.out
```

There exist two types of pipes:

- **Unnamed**: created, used, and destroyed within the life of the process.
- **Named (FIFOs)**: persistent pipes that can outlive the process that created them. They are stored on the file system, and any process can open it like a regular file. They are more efficient than using files as they only exist in memory.

### Unnamed Pipes

```c
int pipe(int fd[2])
```

`pipe` returns two file descriptors:

- `fd[0]`: the read end of the pipe (sender should close).
- `fd[1]`: the write end of the pipe (receiver should close).

If a receiver reads from an empty pipe, it blocks until data is written at the other end. If the sender writes to a full pipe, it blocks until the data is read at the other end and the pipe is cleared.

An example of `pipe` is:

```c
#include <assert.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>
#include <string.h>

int main(int argc, char *argv[]) {
  int fd[2];
  char buf;

  assert(argc == 2);

  if (pipe(fd) == -1) {
    exit(1);
  }

  if (fork() != 0) {
    close(fd[0]);
    write(fd[1], argv[1], strlen(argv[1]));
    close(fd[1]);
    waitpid(-1, NULL, 0);
  } else {
    close(fd[1]);
    while(read(fd[0], &buf, 1) > 0) {
      printf("%c", buf);
    }
    printf("\n");
    close(fd[0]);
  }
}
```

```
$ gcc a.c && ./a.out "hello world"
hello world
```

### Named Pipes (FIFOs)

```bash
NAME
     mkfifo -- make fifos

SYNOPSIS
     mkfifo [-m mode] fifo_name ...
```

(from `man mkfifo`)

`mkfifo` allows for the creation of named pipes.

An example of `mkfifo` is:

```
$ mkfifo /tmp/abc
$ echo ABC > /tmp/abc
```

```
$ cat /tmp/abc
ABC
```
