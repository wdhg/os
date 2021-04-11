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

### Terminating Threads

```c
void pthread_exit(void *value_ptr)
```

`pthread_exit` terminates the calling thread and makes `*value_ptr` available to any other successful join with the terminating thread. It is called implicitly when the thread's start routine returns, except when the intial thread which started `main`.

Note:

- If `main` terminates before any other threads call `pthread_exit`, the entire process is still terminated and the all threads with it.
- If `pthread_exit` is called in `main`, the process continues executing until the last thread is terminated (or `exit` is called).

An example of creating and terminating pthreads is:

```c
#include <pthread.h>
#include <stdio.h>
#include <unistd.h>

void *thread_work(void *thread_id) {
  long id = (long) thread_id;
  printf("Thread %ld\n", id);
  return NULL;
}

int main(int argc, char *argv[]) {
  pthread_t threads[5];
  for (long t = 0; t < 5; t++) {
    pthread_create(&threads[t], NULL, thread_work, (void *) t);
    sleep(1);
  }
  return 0;
}
```

```
$ gcc a.c && ./a.out
Thread 0
Thread 1
Thread 2
Thread 3
Thread 4
```

### Yielding the CPU

```c
int pthread_yield(void)
```

`pthread_yield` releases the CPU to let another thread run. Returns:

- `0` on success.
- an error code otherwise.

In linux, it always succeeds. A thread should yield if it has no need for CPU time, say it's waiting for a certain condition or event to occur.

### Joining Other Threads

```c
int pthread_join(pthread_t thread, void **value_ptr)
```

`pthread_join` blocks the calling thread until `thread` terminates. The `*value_ptr` passed to `pthead_exit` by the terminating thread will be stored at the joining threads `**value_ptr`. `**value_ptr` can be NULL if it is not needed.

An example of joining threads is:

```c
#include <pthread.h>
#include <stdio.h>

long a, b, c;

void *work1(void *x) {
  a = (long) x * (long) x;
  return NULL;
}

void *work2(void *y) {
  b = (long) y * (long) y;
  return NULL;
}

int main(int argc, char *argv[]) {
  pthread_t t1, t2;

  pthread_create(&t1, NULL, work1, (void*) 3);
  pthread_create(&t2, NULL, work1, (void*) 4);

  pthread_join(t1, NULL);
  pthread_join(t2, NULL);

  c = a + b;

  printf("3^2 + 4^2 = %ld\n", c);
}
```

```
$ gcc a.c && ./a.out
3^2 + 4^2 = 16
```


## Two Ways to Implement Threads

Either:

- **User-Level Threads**:
  - The kernel is not aware of the threads.
  - Each process manages its own threads
- **Kernel-Level Threads**:
  - Managed by the kernel

There exist trade-offs of each technique, and there exists various hybrid approaches.

### User-level Threads

From the perspective of the Kernel, it is managing processes only. Threads are implemented by a software library, and the process maintains its own thread table for scheduling.

Advantages:

- Better performance as:
  - Thread creation and termination are fast.
  - Thread switching is fast.
  - Thread synchronisation (e.g. via joining other threads) is fast.
  - All these operations don't require any kernel involvement.
- Each application can have its own scheduling algorithm (provides more flexibility).

Disadvantages:

- Blocking system calls stops all threads in a process, which denies one of the core motivations for using threads.
- Non-blocking I/O can be used (e.g. `select`), which can be harder to use and understand.
- During a page fault, the OS blocks the entire process even though other threads may be runnable.

### Kernel Threads

Advantages:

- Blocking system calls / page faults is easy, as if one thread blocks / causes a page fault the kernel can schedule a runnable thread from the same process

Disadvantages:

- Thread creation and termination is more expensive as they require systems calls (still much cheaper than processes), but this can be mitigated by recycling threads using thread pools.
- Thread synchronisation becomes more expensive as they require blocking system calls.
- Thread switching becomes more expensive as they require a system call (again, still much cheaper than process switches as they all exist in the same address space).
- Applications can't have their own scheduling algorithms.

### Hybrid Approaches

By using kernel threads and multiplexing a larger numebr of user-level threads onto some / all kernel threads. This provides the full true concurrency of kernel threads, but with the lightweight switching of user-level threads.
