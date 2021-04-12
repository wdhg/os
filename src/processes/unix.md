# UNIX Processes

## Creating Processes

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

## Executing Processes

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

## Waiting for Process Termination

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

## Process Termination

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
