# Synchronisation

Key concepts:

- Critical Sections.
- Mutual Exclusion.
- Atomic Operations.
- Race Conditions.
- Deadlocks.
- Starvation.
- Synchronisation Mechanism (e.g. locks, semaphores, monitors, etc).

These concepts are relevant to **both** processes and threads, but to keep things simple they will both be refered to as processes throughout this chapter unless stated otherwise.

## Critical Sections and Mutual Exclusion

A **critical section / region** is a section of code in which processes access a shared resource.

**Mutual exclusion** ensure that if one process is executing a critical section, no other process can be executing it (processes must request permission to enter critical sections).

A **synchronisation mechanism** is used at the entry / exit of a critical section, and is what processes use to request permission to enter it.

The requirements for mutual exclusion are:

- no two processes may be simulatneously inside the same critical section.
- no process running outside the critical section may prevent other processes from entering the critical section. This means when no process is currently inside a critical section, any process that requests permission to enter must be allowed to do so immeditately.
- no process requiring access to its critical section can be delayed forever (i.e. all processes requesting entry must be granted it at some point).
- no assumptions are to be made about the relative speed of processes i.e a process cannot assume that another process will exit its critical section in a given amount of time.

## Examples Methods of Gaining Mutual Exclusion

### Disabling Interrupts

```c
CLI() // clears interrupt flag (IF = 0), disabling interrupts
STI() // set interrupt flag (IF = 1), enabling interrupts
```

By using `CLI` and `STI` to disable / enable interrupts, mutual exclusion can be achieved as other processes won't be able to be scheduled:

```c
int valueMap[10];

// ...

void safe_set_value(int index, int value) {
  CLI();
  valueMap[index] = value;
  STI();
}
```

There are a few issues with this method:

- it only works on single-processor systems. Other processes on other processors will still be running.
- misbehaving / buggy processes may not release the CPU.

Because of this, disabling interrupts should only used by kernel code that requires the fine level of control / performance, and it should be used as little as possible in the kernel code as well.

### Strict Alternation

Strict alternation relies on a variable, say `turn`, to control which process can run at any moment in time. At the end of its critical section, a process will update the `turn` variable to indicate it is done.

For example, assume there are two processes A and B which have an overlapping critical section:

```c
// ...

char turn = 'A';

void process_work_a() {
  while (true) {
    while (turn != 'A') {}    // empty loop to await turn
    critical_section();       // enter critical_section
    turn = 'B';               // leave critical_section
    non_critical_section_a(); // non-critical section
  }
}

void process_work_b()(void *arg) {
  while (true) {
    while (turn != 'B') {}
    critical_section();
    turn = 'A';
    non_critical_section_b();
  }
}
```

This fixes the problem of having to turn off interrupts, but it also introduces new problems:

- if it is process A's turn, but it is in its non-critical section, it can take as long as it wants / never enter its critical section again. B will be stuck waiting for A, which breaks two rules:
  - that no process should be able to prevent another process from entering its critical section
  - that if no process is in its critical section, and process attempting to enter its critical section should be allowed to do so immeditately.
  - depending on the behaviour of `non_critical_section_a`, it could potentially break a third rule: that no process should be waiting to enter its critical section forever.
- process B is unable to enter its critical section twice in a row as it is forced to wait for process A to update `turn`.

### Busy Waiting

Strict alternation uses a mechanism called **busy waiting**, which means a loop that constantly tests a value until it meets a certain condition. This is bad as it wastes valuable CPU time. As a result, busy waiting should only ever be used when the wait is expected to be short.


