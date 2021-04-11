# Threads

A thread is an execution stream that share the same address space. When multithreading is used, each process can contain one or more threads.

A break down of per process and per thread items is:

Per process items:

- Address space.
- Global variables.
- Open files.
- Child processes.
- Signals.

Per thread items:

- Program counter (PC).
- Registers.
- Stack.

## Why Threads?

Many applications require running multiple activities, some of which need to:

- execute in parallel.
- access and process the same data.
- potentially block.

## Why not Processes?

Threads are much lighter than processes. Processes have a lot more state associated with them (e.g. their own address spaces and other kernel state). They also:

- are difficult to communitcate between as they have different address spaces.
- may block, causing the entire application to be switched out.
- require expensive context switches as we need to change the address space that is currently visible to the executing program.
- are expensive to create / destroy.


## Problems / Concerns with Threads

Threads have a shared address space. This means that they don't have any protection from reading / writing to each others stack like processes have. This can cause memory corruption issues and other concurrency bugs due to concurrent access to shared data (e.g. through global variables).

Forking inside a thread is another concern. Does the fork create a new process with the same number of threads, or with a single thread?

Handling signals also becomes an issue. Which thread should handle the signal?


## PThreads (POSIX Threads)

PThreads are implemented by most UNIX systems, and are defined by IEEE standard 1003.1c

```c
#include <pthread.h>
#include <sys/types.h>

pthread_t      // type representing a thread
pthread_attr_t // type representing the attributes of a thread
```

### Creating Threads

```c
int pthread_create(
  pthread_t *thread,
  const pthread_attr_t *attr,
  void *(*start_routine) (void*),
  void *arg
)
```

`pthread_create` creates a new thread, stored in `*thread`. The function returns:

- `0` if a the thread was successfully created.
- an error code otherwise.

It arguments are:

- `*thread`: the variable to store the thread in.
- `*attr`: specifies the thread attributes (e.g. minimum stack size, guard size, detached / joinable, etc). Can be `NULL` for default attributes.
- `*start_routine`: the C function the thread will start executing upon creation.
- `*arg`: argument to be passed to the `start_routine`. Can be `NULL` if no arguments are needed.
