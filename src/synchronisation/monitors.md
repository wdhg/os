# Monitors

A **monitor** is a higher-level synchronisation primitive that allows threads to have both mutual exclusion and the ability to await a signal / condition to become true or false.

A monitor consists of:

- shared data.
- entry procedures (must be called to **enter the monitor**).
- internal procedures (can only be called by a process **inside the monitor**).
- an (implicit) monitor lock.
- one or more condition variables.

A process being **inside the monitor** means that it has mutual exclusion over all of the shared data. Processes can call an entry procedure to enter the monitor and gain access to the internal data / procedures.

**Only one process can be inside the monitor at any given time**.



## Condition Variables

A condition variable is associated with some high-level condition, for example (in the context of producers / consumers):

- "some space has become available in the buffer".
- "some data has arrived in the buffer".

A condition variable has the following operations:

- `wait(c)`: releases the monitor lock and waits for the condition variable `c` to be signalled.
- `signal(c)`: wakes up one process waiting for `c` to be signalled.
- `broadcast(c)`: wakes up all processes waiting for `c` to be signalled.

Signals on a condition variable do not accumulate. If a condition variable is signalled but no process is waiting for it, the signal is lost.

### What Happens on a Signal?

There are two different implementations of what to do on a signal, one defined by [Hoare](https://en.wikipedia.org/wiki/Tony_Hoare) and the other by [Lampson](https://en.wikipedia.org/wiki/Butler_Lampson).

**Hoare**: A process waiting for a signal is immeditately scheduled.

Advantages:

- Easy to reason about.

Disadvantages:

- Inefficient. The process which sends the signal is switched out even if it has not finished with the monitor.
- It places extra constraints on the scheduler as it is harder to implement.

**Lampson**: Sending a signal and waking up from a wait is not atomic.

Advantages:

- More efficient as there are no constraints on the scheduler.
- More tolerant of errors: if the condition being signalled is wrong, it is simply discarded when rechecked.

Disadvantages:

- More difficult to understand as `wait` and `signal` are no longer atomic, so the programmer must be careful when waking up from a `wait`.

Most of the time, Lampson's implemention is used.

## Pseudo Example of Monitors (Producer / Consumer)

```
monitor ProducerConsumer {
  condition not_full, not_empty;
  integer count = 0;

  entry procedure insert(item) {
    if (count == N) {
      wait(not_full);
    }
    insert_item(item);
    count++;
    signal(not_empty);
  }

  entry procedure remove(item) {
    if (count == 0) {
      wait(not_empty);
    }
    remove_item(item);
    count--;
    signal(not_full);
  }
}
```

However under a Lampson model, the `if` statements before `wait` must be replaced with `while` statements:

```
monitor ProducerConsumer {
  condition not_full, not_empty;
  integer count = 0;

  entry procedure insert(item) {
    while (count == N) {
      wait(not_full);
    }
    insert_item(item);
    count++;
    signal(not_empty);
  }

  entry procedure remove(item) {
    wghile (count == 0) {
      wait(not_empty);
    }
    remove_item(item);
    count--;
    signal(not_full);
  }
}
```

## Monitors are a Language Construct

Monitors are a **language construct**. This means that unlike semaphores which are provided by the OS, monitors are built on top of the provided primitives. Monitors do not exist C, but they exist in many other languages. For example, in Java:

- every object and class is logically associated with a monitor.
- synchronized blocks / methods use these monitors to create monitor regions.
- Java doesn't have explicit condition variables, but it does have `wait()` and `notify()`, which act the same as `wait()` and `signal()` on a traditional condition variable.

[Source](https://www.programcreek.com/2011/12/monitors-java-synchronization-mechanism/)
