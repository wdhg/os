# User-Level and Kernel-Level Threads

There exist two main ways to implement threads:

- **User-Level Threads**:
  - The kernel is not aware of the threads.
  - Each process manages its own threads
- **Kernel-Level Threads**:
  - Managed by the kernel

There exist trade-offs of each technique, and there exists various hybrid approaches.

## User-level Threads

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

## Kernel Threads

Advantages:

- Blocking system calls / page faults is easy, as if one thread blocks / causes a page fault the kernel can schedule a runnable thread from the same process

Disadvantages:

- Thread creation and termination is more expensive as they require systems calls (still much cheaper than processes), but this can be mitigated by recycling threads using thread pools.
- Thread synchronisation becomes more expensive as they require blocking system calls.
- Thread switching becomes more expensive as they require a system call (again, still much cheaper than process switches as they all exist in the same address space).
- Applications can't have their own scheduling algorithms.

## Hybrid Approaches

By using kernel threads and multiplexing a larger numebr of user-level threads onto some / all kernel threads. This provides the full true concurrency of kernel threads, but with the lightweight switching of user-level threads.
