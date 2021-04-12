# Process Communication

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

This chapter will focus on UNIX process communication.

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


## UNIX Pipes

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
