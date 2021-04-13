# Race Conditions and Memory Models


## Race Conditions

A **race condition** occurs when any two processes read and write shared data without the protection of any synchronisation mechanism. The result depends on the execution order of each process (e.g. their exact interleaving).

### Thread Interleavings

The **thread interleavings** of two or more threads is the total set of all possible orderings of their statement execution.

For example, two threads A and B with statements `{A1; A2}` and `{B1; B2}` respectively have thread interleavings:

- `A1 -> A2 -> B1 -> B2`
- `A1 -> B1 -> A2 -> B2`
- `A1 -> B1 -> B2 -> A2`
- `B1 -> B2 -> A1 -> A2`
- `B1 -> A1 -> B2 -> A2`
- `B1 -> A1 -> A2 -> B2`

## Memory Models

A **memory model** describes the the interactions of threads through memory and shared use of data. There exist many memory models as they are dependant on hardware behaviour and compiler optimisations.

### Sequential Consistency

**Sequential Consistency**: The operations of each thread appear in program order, and the operations of all threads are executed in some sequential order atomically.

In this book, sequential consistency is assumed.

For example, under sequential consistency it is impossible for both threads to read `flag1 == 0` and `flag2 == 0` inside their if conditions:

```c
int flag1 = 0;
int flag2 = 0;

void thread_work_a() {
  flag1 = 1;
  if (flag2 == 0) {
    critical_section();
  }
}

void thread_work_b() {
  flag2 = 1;
  if (flag1 == 0) {
    critical_section();
  }
}
```

However under a **weak memory model**, it would be possible.

### Weak Memory Models

Under **weak memory models**, the hardware (or compiler) is permitted to reorder memory writes.

### Happens-Before Relationship

A **happens-before relationship** is a theoretical framework that is used reason about race conditions under concurrency. It works by introducing a partial order between events (e.g. instructions) in a trace, denoted by `a -> b` where `a` and `b` are events in a trace.

For example, consider `a` and `b` where `a` occurs before `b` in the trace:

- If `a` and `b` are in the same thread, then `a -> b`.
- If `a` is `release(lock)` and `b` is `acquire(lock)`, then `a -> b` (this can be generalised for other synchronisation mechanisms).

Some properties about `->`:

- **Irreflexive**: for all `a`, `a -/-> a` (i.e. `a -> a` is impossible).
- **Antisymmetric**: for all `a`, `b`, if `a -> b` then `b -/-> a`.
- **Transitive**: for all `a`, `b`, `c`, if `a -> b` and `b -> c` then `a -> c`.

A data race (race condition) occurs between `a` and `b` in the trace if and only if:

- they access the same memory location.
- at least one of the is a write.
- they are unordered accoring to happens-before.

### Data Race Examples

```c
int a, b;

void thread_work_a() {
  a = 1; // write
  b = 1; // write
}

void thread_work_b() {
  b = 2; // write
  a = 2; // write
}
```

The partial orders are: `a = 1 -> b = 1` and `b = 2 -> a = 2`. Since the instructions:

- access the same memory
- are all writes
- are unordered accoring to happens-before

there is an obvious data race.

```c
int a, b;
int a_b_lock;

void thread_work_a() {
  acquire(a_b_lock); // lock
  a = 1;             // write
  b = 1;             // write
  release(a_b_lock); // unlock
}

void thread_work_b() {
  acquire(a_b_lock); // lock
  b = 2;             // write
  a = 2;             // write
  release(a_b_lock); // unlock
}
```

The potential traces are: 

- `acquire(a_b_lock) -> a = 1 -> b = 1 -> release(a_b_lock) -> acquire(a_b_lock) -> b = 2 -> a = 2 -> release(a_b_lock)`
- `acquire(a_b_lock) -> b = 2 -> a = 2 -> release(a_b_lock) -> acquire(a_b_lock) -> a = 1 -> b = 1 -> release(a_b_lock)`

As they are ordered according to happens-before, there is no data race.

```c
int a, b;
int b_lock;

void thread_work_a() {
  a++;             // write
  acquire(b_lock); // lock
  b++;             // write
  release(b_lock); // unlock
}

void thread_work_b() {
  acquire(b_lock); // lock
  b++;             // write
  release(b_lock); // unlock
  a++;             // write
}
```

This example is interesting. If we assume thread A is scheduled first, the trace would look like:

`
a++ -> acquire(b_lock) -> b++ -> release(b_lock) -> acquire(b_lock) -> b++ -> release(b_lock) -> a++
`

which looks safe. However, if thread B is scheduled first the trace would look like:

```
acquire(b_lock) -> b++ -> release(b_lock) -> a++
                                |
                                V
                   a++ -> acquire(b_lock) -> b++ -> release(b_lock) 
```

Now both the `a++` instructions are unordered relative with each other, which is a data race (`a++` is **not** atomic).
