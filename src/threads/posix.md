# PThreads (POSIX Threads)

PThreads are implemented by most UNIX systems, and are defined by IEEE standard 1003.1c

```c
#include <pthread.h>
#include <sys/types.h>

pthread_t      // type representing a thread
pthread_attr_t // type representing the attributes of a thread
```

## Creating Threads

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

## Terminating Threads

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

## Yielding the CPU

```c
int pthread_yield(void)
```

`pthread_yield` releases the CPU to let another thread run. Returns:

- `0` on success.
- an error code otherwise.

In linux, it always succeeds. A thread should yield if it has no need for CPU time, say it's waiting for a certain condition or event to occur.

## Joining Other Threads

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
  long tmp = *((long*) x);
  a = tmp * tmp;
  return NULL;
}

void *work2(void *y) {
  long tmp = *((long*) y);
  b = tmp * tmp;
  return NULL;
}

int main(int argc, char *argv[]) {
  pthread_t t1, t2;

  long x = 3, y = 4;

  pthread_create(&t1, NULL, work1, (void*) &x);
  pthread_create(&t2, NULL, work2, (void*) &y);

  pthread_join(t1, NULL);
  pthread_join(t2, NULL);

  c = a + b;

  printf("3^2 + 4^2 = %ld\n", c);
}
```

```
$ gcc a.c && ./a.out
3^2 + 4^2 = 25
```
